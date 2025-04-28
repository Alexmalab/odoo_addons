# Odoo Module: l10n_in_edi_ewaybill

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import demo

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    "name": """Indian - E-waybill""",
    "countries": ["in"],
    "version": "1.03.00",
    "category": "Accounting/Localizations/EDI",
    "depends": [
        "l10n_in_edi",
    ],
    "description": """
Indian - E-waybill
====================================
To submit E-waybill through API to the government.
We use "Tera Software Limited" as GSP

Step 1: First you need to create an API username and password in the E-waybill portal.
Step 2: Switch to company related to that GST number
Step 3: Set that username and password in Odoo (Goto: Invoicing/Accounting -> Configration -> Settings -> Indian Electronic WayBill or find "E-waybill" in search bar)
Step 4: Repeat steps 1,2,3 for all GSTIN you have in odoo. If you have a multi-company with the same GST number then perform step 1 for the first company only.
    """,
    "data": [
        "security/ir.model.access.csv",
        "data/account_edi_data.xml",
        "data/ewaybill_type_data.xml",
        "views/account_move_views.xml",
        "views/edi_pdf_report.xml",
        "views/res_config_settings_views.xml",
    ],
    "demo": [
        "demo/demo_company.xml",
    ],
    "installable": True,
    # not auto_install because the company can be related to the service industry
    "license": "LGPL-3",
}

```

## File: data\account_edi_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="edi_in_ewaybill_json_1_03" model="account.edi.format">
        <field name="name">E-waybill (IN)</field>
        <field name="code">in_ewaybill_1_03</field>
    </record>
</odoo>

```

## File: data\ewaybill_type_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Document Type -->
    <record id="type_tax_invoice_sub_type_supply" model="l10n.in.ewaybill.type">
        <field name="name">Tax Invoice</field>
        <field name="code">INV</field>
        <field name="sub_type">Supply</field>
        <field name="sub_type_code">1</field>
        <field name="allowed_supply_type">both</field>
        <field name="active" eval="True"/>
    </record>

    <record id="type_tax_invoice_sub_type_export" model="l10n.in.ewaybill.type">
        <field name="name">Tax Invoice</field>
        <field name="code">INV</field>
        <field name="sub_type">Export</field>
        <field name="sub_type_code">3</field>
        <field name="allowed_supply_type">out</field>
        <field name="active" eval="True"/>
    </record>

    <record id="type_tax_invoice_sub_type_skd_ckd_lots" model="l10n.in.ewaybill.type">
        <field name="name">Tax Invoice</field>
        <field name="code">INV</field>
        <field name="sub_type">SKD/CKD/Lots</field>
        <field name="sub_type_code">9</field>
        <field name="allowed_supply_type">both</field>
        <field name="active" eval="True"/>
    </record>

    <record id="type_bill_of_supply_sub_type_supply" model="l10n.in.ewaybill.type">
        <field name="name">Bill of Supply</field>
        <field name="code">BIL</field>
        <field name="sub_type">Supply</field>
        <field name="sub_type_code">1</field>
        <field name="allowed_supply_type">both</field>
        <field name="active" eval="True"/>
    </record>

    <record id="type_bill_of_supply_sub_type_export" model="l10n.in.ewaybill.type">
        <field name="name">Bill of Supply</field>
        <field name="code">BIL</field>
        <field name="sub_type">Export</field>
        <field name="sub_type_code">3</field>
        <field name="allowed_supply_type">out</field>
        <field name="active" eval="True"/>
    </record>

    <record id="type_bill_of_supply_sub_type_skd_ckd_lots" model="l10n.in.ewaybill.type">
        <field name="name">Bill of Supply</field>
        <field name="code">BIL</field>
        <field name="sub_type">SKD/CKD/Lots</field>
        <field name="sub_type_code">9</field>
        <field name="allowed_supply_type">both</field>
        <field name="active" eval="True"/>
    </record>

    <record id="type_bill_of_entry_sub_type_import" model="l10n.in.ewaybill.type">
        <field name="name">Bill of Entry</field>
        <field name="code">BOE</field>
        <field name="sub_type">Import</field>
        <field name="sub_type_code">2</field>
        <field name="allowed_supply_type">in</field>
        <field name="active" eval="True"/>
    </record>
    <record id="type_bill_of_entry_sub_skd_ckd_lots" model="l10n.in.ewaybill.type">
        <field name="name">Bill of Entry</field>
        <field name="code">BOE</field>
        <field name="sub_type">SKD/CKD/Lots</field>
        <field name="sub_type_code">9</field>
        <field name="allowed_supply_type">in</field>
        <field name="active" eval="True"/>
    </record>

</odoo>

```

## File: data\neutralize.sql

```sql
-- disable l10n_in_edi_ewaybill integration
UPDATE res_company
   SET l10n_in_edi_ewaybill_username = NULL,
       l10n_in_edi_ewaybill_password = NULL,
       l10n_in_edi_ewaybill_auth_validity = NULL;

```

## File: models\account_edi_format.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
import json
from datetime import timedelta
from markupsafe import Markup

from odoo import models, fields, api, _
from odoo.tools import html_escape
from odoo.exceptions import AccessError

from .error_codes import ERROR_CODES

import logging

_logger = logging.getLogger(__name__)


class AccountEdiFormat(models.Model):
    _inherit = "account.edi.format"

    def _l10n_in_edi_ewaybill_base_irn_or_direct(self, move):
        """
            There is two type of api call to create E-waybill
            1. base on IRN, IRN is number created when we do E-invoice
            2. direct call, when E-invoice not aplicable or it"s credit note or debit note
        """
        if move.move_type == "out_refund" or move.debit_origin_id:
            return "direct"
        einvoice_in_edi_format = move.journal_id.edi_format_ids.filtered(lambda f: f.code == "in_einvoice_1_03")
        return einvoice_in_edi_format and einvoice_in_edi_format._get_move_applicability(move) and "irn" or "direct"

    def _is_compatible_with_journal(self, journal):
        if self.code == "in_ewaybill_1_03":
            # In the Invoice we have a button to send Ewaybill so not required to send it automatically.
            return False
        return super()._is_compatible_with_journal(journal)

    def _is_enabled_by_default_on_journal(self, journal):
        """
            It's sent with a button action on the invoice so it's disabled by default
        """
        self.ensure_one()
        if self.code == "in_ewaybill_1_03":
            return False
        return super()._is_enabled_by_default_on_journal(journal)

    def _get_move_applicability(self, invoice):
        self.ensure_one()
        if self.code != 'in_ewaybill_1_03':
            return super()._get_move_applicability(invoice)

        if invoice.is_invoice() and invoice.country_code == 'IN':
            res = {
                'post': self._l10n_in_edi_ewaybill_post_invoice_edi,
                'cancel': self._l10n_in_edi_ewaybill_cancel_invoice,
                'edi_content': self._l10n_in_edi_ewaybill_json_invoice_content,
            }
            base = self._l10n_in_edi_ewaybill_base_irn_or_direct(invoice)
            if base == 'irn':
                res.update({
                    'post': self._l10n_in_edi_ewaybill_irn_post_invoice_edi,
                    'edi_content': self._l10n_in_edi_ewaybill_irn_json_invoice_content,
                    })
            return res

    def _needs_web_services(self):
        self.ensure_one()
        return self.code == "in_ewaybill_1_03" or super()._needs_web_services()

    def _l10n_in_edi_ewaybill_irn_json_invoice_content(self, move):
        return json.dumps(self._l10n_in_edi_irn_ewaybill_generate_json(move)).encode()

    def _l10n_in_edi_ewaybill_json_invoice_content(self, move):
        return json.dumps(self._l10n_in_edi_ewaybill_generate_json(move)).encode()

    def _check_move_configuration(self, move):
        if self.code != "in_ewaybill_1_03":
            return super()._check_move_configuration(move)
        error_message = []
        base = self._l10n_in_edi_ewaybill_base_irn_or_direct(move)
        if not move.l10n_in_type_id and base == "direct":
            error_message.append(_("- Document Type"))
        if not move.l10n_in_mode:
            error_message.append(_("- Transportation Mode"))
        elif move.l10n_in_mode == "0" and not move.l10n_in_transporter_id:
            error_message.append(_("- Transporter is required when E-waybill is managed by transporter"))
        elif move.l10n_in_mode == "0" and move.l10n_in_transporter_id and not move.l10n_in_transporter_id.vat:
            error_message.append(_("- Selected Transporter is missing GSTIN"))
        elif move.l10n_in_mode == "1":
            if not move.l10n_in_vehicle_no and move.l10n_in_vehicle_type:
                error_message.append(_("- Vehicle Number and Type is required when Transportation Mode is By Road"))
        elif move.l10n_in_mode in ("2", "3", "4"):
            if not move.l10n_in_transportation_doc_no and move.l10n_in_transportation_doc_date:
                error_message.append(_("- Transport document number and date is required when Transportation Mode is Rail,Air or Ship"))
        if error_message:
            error_message.insert(0, _("The following information are missing on the invoice (see eWayBill tab):"))
        goods_lines = move.invoice_line_ids.filtered(lambda line: not (line.display_type in ('line_section', 'line_note', 'rounding') or line.product_id.type == "service"))
        if not goods_lines:
            error_message.append(_('You need at least one product having "Product Type" as stockable or consumable.'))
        if base == "irn":
            # already checked by E-invoice (l10n_in_edi) so no need to check
            return error_message
        is_purchase = move.is_purchase_document(include_receipts=True)
        error_message += self._l10n_in_validate_partner(move.partner_id)
        error_message += self._l10n_in_validate_partner(move.company_id.partner_id, is_company=True)
        if not re.match("^.{1,16}$", is_purchase and move.ref or move.name):
            error_message.append(_("%s number should be set and not more than 16 characters",
                (is_purchase and "Bill Reference" or "Invoice")))
        for line in goods_lines:
            if line.display_type == 'product':
                hsn_code = self._l10n_in_edi_extract_digits(line.l10n_in_hsn_code)
                if not hsn_code:
                    error_message.append(_("HSN code is not set in product line %s", line.name))
                elif not re.match(r'^\d{4}$|^\d{6}$|^\d{8}$', hsn_code):
                    error_message.append(_(
                        "Invalid HSN Code (%(hsn_code)s) in product line %(product_line)s") % {
                        'hsn_code': hsn_code,
                        'product_line': line.product_id.name or line.name
                    })
        if error_message:
            error_message.insert(0, _("Impossible to send the Ewaybill."))
        return error_message

    def _l10n_in_edi_ewaybill_cancel_invoice(self, invoices):
        if self.code != "in_ewaybill_1_03":
            return super()._cancel_invoice_edi(invoices)
        response = {}
        res = {}
        ewaybill_response_json = invoices._get_l10n_in_edi_ewaybill_response_json()
        cancel_json = {
            "ewbNo": ewaybill_response_json.get("ewayBillNo") or ewaybill_response_json.get("EwbNo"),
            "cancelRsnCode": int(invoices.l10n_in_edi_cancel_reason),
            "CnlRem": invoices.l10n_in_edi_cancel_remarks,
        }
        response = self._l10n_in_edi_ewaybill_cancel(invoices.company_id, cancel_json)
        if response.get("error"):
            error = response["error"]
            error_codes = [e.get("code") for e in error]
            if "238" in error_codes:
                # Invalid token eror then create new token and send generate request again.
                # This happen when authenticate called from another odoo instance with same credentials (like. Demo/Test)
                authenticate_response = self._l10n_in_edi_ewaybill_authenticate(invoices.company_id)
                if not authenticate_response.get("error"):
                    error = []
                    response = self._l10n_in_edi_ewaybill_cancel(invoices.company_id, cancel_json)
                    if response.get("error"):
                        error = response["error"]
                        error_codes = [e.get("code") for e in error]
            if "312" in error_codes:
                # E-waybill is already canceled
                # this happens when timeout from the Government portal but IRN is generated
                error_message = "<br/>".join([html_escape("[%s] %s" % (e.get("code"), e.get("message"))) for e in error])
                error = []
                response = {"data": ""}
                odoobot = self.env.ref("base.partner_root")
                invoices.message_post(author_id=odoobot.id, body=
                    Markup("%s<br/>%s:<br/>%s") %(
                        _("Somehow this E-waybill has been cancelled in the government portal before. You can verify by checking the details into the government (https://ewaybillgst.gov.in/Others/EBPrintnew.aspx)"),
                        _("Error"),
                        error_message
                    )
                )
            if "no-credit" in error_codes:
                res[invoices] = {
                    "success": False,
                    "error": self._l10n_in_edi_get_iap_buy_credits_message(invoices.company_id),
                    "blocking_level": "error",
                }
            elif error:
                error_message = "<br/>".join([html_escape("[%s] %s" % (e.get("code"), e.get("message"))) for e in error])
                blocking_level = "error"
                if "404" in error_codes:
                    blocking_level = "warning"
                res[invoices] = {
                    "success": False,
                    "error": error_message,
                    "blocking_level": blocking_level,
                }
        if not response.get("error"):
            json_dump = json.dumps(response.get("data"))
            json_name = "%s_ewaybill_cancel.json" % (invoices.name.replace("/", "_"))
            attachment = self.env["ir.attachment"].create({
                "name": json_name,
                "raw": json_dump.encode(),
                "res_model": "account.move",
                "res_id": invoices.id,
                "mimetype": "application/json",
            })
            inv_res = {"success": True, "attachment": attachment}
            res[invoices] = inv_res
        return res

    def _l10n_in_edi_ewaybill_handle_zero_distance_alert_if_present(self, invoice, response):
        if invoice.l10n_in_distance == 0 and (alert := response.get("data", {}).get('alert')):
            pattern = r"Distance between these two pincodes is (\d+)"
            if (match := re.search(pattern, alert)) and (distance := int(match.group(1))) > 0:
                invoice.l10n_in_distance = distance

    def _l10n_in_edi_ewaybill_irn_post_invoice_edi(self, invoices):
        response = {}
        res = {}
        generate_json = self._l10n_in_edi_irn_ewaybill_generate_json(invoices)
        response = self._l10n_in_edi_irn_ewaybill_generate(invoices.company_id, generate_json)
        if response.get("error"):
            error = response["error"]
            error_codes = [e.get("code") for e in error]
            if "1005" in error_codes:
                # Invalid token eror then create new token and send generate request again.
                # This happen when authenticate called from another odoo instance with same credentials (like. Demo/Test)
                authenticate_response = self._l10n_in_edi_authenticate(invoices.company_id)
                if not authenticate_response.get("error"):
                    error = []
                    response = self._l10n_in_edi_irn_ewaybill_generate(invoices.company_id, generate_json)
                    if response.get("error"):
                        error = response["error"]
                        error_codes = [e.get("code") for e in error]
            if "4002" in error_codes or "4026" in error_codes:
                # Get E-waybill by details in case of IRN is already generated
                # this happens when timeout from the Government portal but E-waybill is generated
                response = self._l10n_in_edi_irn_ewaybill_get(invoices.company_id, generate_json.get("Irn"))
                if not response.get("error"):
                    error = []
                    odoobot = self.env.ref("base.partner_root")
                    invoices.message_post(author_id=odoobot.id, body=
                        _("Somehow this E-waybill has been generated in the government portal before. You can verify by checking the invoice details into the government (https://ewaybillgst.gov.in/Others/EBPrintnew.aspx)")
                    )

            if "no-credit" in error_codes:
                res[invoices] = {
                    "success": False,
                    "error": self._l10n_in_edi_get_iap_buy_credits_message(invoices.company_id),
                    "blocking_level": "error",
                }
            elif error:
                error_message = "<br/>".join([html_escape("[%s] %s" % (e.get("code"), e.get("message"))) for e in error])
                blocking_level = "error"
                if "404" in error_codes or "waiting" in error_codes:
                    blocking_level = "warning"
                res[invoices] = {
                    "success": False,
                    "error": error_message,
                    "blocking_level": blocking_level,
                }
        if not response.get("error"):
            json_dump = json.dumps(response.get("data"))
            json_name = "%s_irn_ewaybill.json" % (invoices.name.replace("/", "_"))
            attachment = self.env["ir.attachment"].create({
                "name": json_name,
                "raw": json_dump.encode(),
                "res_model": "account.move",
                "res_id": invoices.id,
                "mimetype": "application/json",
            })
            inv_res = {"success": True, "attachment": attachment}
            res[invoices] = inv_res
            body = Markup("""
                <b>{}</b><br/>
                <ul>
                    <li>{}<i class="o-mail-Message-trackingSeparator fa fa-long-arrow-right mx-1 text-600"></i>{}</li>
                    <li>{}<i class="o-mail-Message-trackingSeparator fa fa-long-arrow-right mx-1 text-600"></i>{}</li>
                </ul>
            """).format(
                _('E-wayBill Sent'),
                _('Number'),
                str(response.get("data", {}).get('EwbNo')),
                _('Validity'),
                str(response.get("data", {}).get('EwbValidTill'))
            )

            invoices.message_post(body=body)
            self._l10n_in_edi_ewaybill_handle_zero_distance_alert_if_present(invoices, response)
        return res

    def _l10n_in_edi_irn_ewaybill_generate_json(self, invoice):
        json_payload = {
            "Irn": invoice._get_l10n_in_edi_response_json().get("Irn"),
            "Distance": invoice.l10n_in_distance,
        }
        if invoice.l10n_in_mode == "0":
            json_payload.update({
                "TransId": invoice.l10n_in_transporter_id.vat,
                "TransName": invoice.l10n_in_transporter_id.name,
            })
        elif invoice.l10n_in_mode == "1":
            json_payload.update({
                "TransMode": invoice.l10n_in_mode,
                "VehNo": invoice.l10n_in_vehicle_no,
                "VehType": invoice.l10n_in_vehicle_type,
            })
        elif invoice.l10n_in_mode in ("2", "3", "4"):
            doc_date = invoice.l10n_in_transportation_doc_date
            json_payload.update({
                "TransMode": invoice.l10n_in_mode,
                "TransDocDt": doc_date and doc_date.strftime("%d/%m/%Y") or False,
                "TransDocNo": invoice.l10n_in_transportation_doc_no,
            })
        return json_payload

    def _l10n_in_edi_ewaybill_post_invoice_edi(self, invoices):
        response = {}
        res = {}
        generate_json = self._l10n_in_edi_ewaybill_generate_json(invoices)
        response = self._l10n_in_edi_ewaybill_generate(invoices.company_id, generate_json)
        if response.get("error"):
            error = response["error"]
            error_codes = [e.get("code") for e in error]
            if "238" in error_codes:
                # Invalid token eror then create new token and send generate request again.
                # This happen when authenticate called from another odoo instance with same credentials (like. Demo/Test)
                authenticate_response = self._l10n_in_edi_ewaybill_authenticate(invoices.company_id)
                if not authenticate_response.get("error"):
                    error = []
                    response = self._l10n_in_edi_ewaybill_generate(invoices.company_id, generate_json)
                    if response.get("error"):
                        error = response["error"]
                        error_codes = [e.get("code") for e in error]
            if "604" in error_codes:
                # Get E-waybill by details in case of E-waybill is already generated
                # this happens when timeout from the Government portal but E-waybill is generated
                response = self._l10n_in_edi_ewaybill_get_by_consigner(
                    invoices.company_id, generate_json.get("docType"), generate_json.get("docNo"))
                if not response.get("error"):
                    error = []
                    odoobot = self.env.ref("base.partner_root")
                    invoices.message_post(author_id=odoobot.id, body=
                        _("Somehow this E-waybill has been generated in the government portal before. You can verify by checking the invoice details into the government (https://ewaybillgst.gov.in/Others/EBPrintnew.aspx)")
                    )
            if "no-credit" in error_codes:
                res[invoices] = {
                    "success": False,
                    "error": self._l10n_in_edi_get_iap_buy_credits_message(invoices.company_id),
                    "blocking_level": "error",
                }
            elif error:
                error_message = "<br/>".join([html_escape("[%s] %s" % (e.get("code"), e.get("message"))) for e in error])
                blocking_level = "error"
                if "404" in error_codes:
                    blocking_level = "warning"
                res[invoices] = {
                    "success": False,
                    "error": error_message,
                    "blocking_level": blocking_level,
                }
        if not response.get("error"):
            json_dump = json.dumps(response.get("data"))
            json_name = "%s_ewaybill.json" % (invoices.name.replace("/", "_"))
            attachment = self.env["ir.attachment"].create({
                "name": json_name,
                "raw": json_dump.encode(),
                "res_model": "account.move",
                "res_id": invoices.id,
                "mimetype": "application/json",
            })
            inv_res = {"success": True, "attachment": attachment}
            res[invoices] = inv_res
            body = Markup("""
                <b>{}</b><br/>
                <ul>
                    <li>{}<i class="o-mail-Message-trackingSeparator fa fa-long-arrow-right mx-1 text-600"></i>{}</li>
                    <li>{}<i class="o-mail-Message-trackingSeparator fa fa-long-arrow-right mx-1 text-600"></i>{}</li>
                </ul>
            """).format(
                _('E-wayBill Sent'),
                _('Number'),
                str(response.get("data", {}).get('ewayBillNo')),
                _('Validity'),
                str(response.get("data", {}).get('validUpto'))
            )
            invoices.message_post(body=body)
            self._l10n_in_edi_ewaybill_handle_zero_distance_alert_if_present(invoices, response)
        return res

    def _l10n_in_edi_ewaybill_get_error_message(self, code):
        error_message = self.env._(ERROR_CODES.get(code, ''))  # pylint: disable=gettext-variable
        return error_message or _("We don't know the error message for this error code. Please contact support.")

    def _get_l10n_in_edi_saler_buyer_party(self, move):
        res = super()._get_l10n_in_edi_saler_buyer_party(move)
        if move.is_outbound() and self.code == 'in_ewaybill_1_03':
            res = {
                "seller_details":  move.partner_id,
                "dispatch_details": move.partner_shipping_id or move.partner_id,
                "buyer_details": move.company_id.partner_id,
                "ship_to_details": move._l10n_in_get_warehouse_address() or move.company_id.partner_id,
            }
        return res

    def _l10n_in_edi_ewaybill_generate_json(self, invoices):
        def get_transaction_type(seller_details, dispatch_details, buyer_details, ship_to_details):
            """
                1 - Regular
                2 - Bill To - Ship To
                3 - Bill From - Dispatch From
                4 - Combination of 2 and 3
            """
            if seller_details != dispatch_details and buyer_details != ship_to_details:
                return 4
            elif seller_details != dispatch_details:
                return 3
            elif buyer_details != ship_to_details:
                return 2
            else:
                return 1

        saler_buyer = self._get_l10n_in_edi_saler_buyer_party(invoices)
        seller_details = saler_buyer.get("seller_details")
        dispatch_details = saler_buyer.get("dispatch_details")
        buyer_details = saler_buyer.get("buyer_details")
        ship_to_details = saler_buyer.get("ship_to_details")
        sign = invoices.is_inbound() and -1 or 1
        extract_digits = self._l10n_in_edi_extract_digits
        tax_details = self._l10n_in_prepare_edi_tax_details(invoices)
        tax_details_by_code = self._get_l10n_in_tax_details_by_line_code(tax_details.get("tax_details", {}))
        invoice_line_tax_details = tax_details.get("tax_details_per_record")
        rounding_amount = sum(line.balance for line in invoices.line_ids if line.display_type == 'rounding') * sign
        json_payload = {
            # Note:
            # Customer Invoice, Sales Receipt and Vendor Credit Note are Outgoing
            # Vendor Bill, Purchase Receipt, and Customer Credit Note are Incoming
            "supplyType": invoices.is_outbound() and "I" or "O",
            "subSupplyType": invoices.l10n_in_type_id.sub_type_code,
            "docType": invoices.l10n_in_type_id.code,
            "transactionType": get_transaction_type(seller_details, dispatch_details, buyer_details, ship_to_details),
            "transDistance": str(invoices.l10n_in_distance),
            "docNo": invoices.is_purchase_document(include_receipts=True) and invoices.ref or invoices.name,
            "docDate": invoices.date.strftime("%d/%m/%Y"),
            "fromGstin": seller_details.commercial_partner_id.vat or "URP",
            "fromTrdName": seller_details.commercial_partner_id.name,
            "fromAddr1": dispatch_details.street or "",
            "fromAddr2": dispatch_details.street2 or "",
            "fromPlace": dispatch_details.city or "",
            "fromPincode": dispatch_details.country_id.code == "IN" and int(extract_digits(dispatch_details.zip)) or "",
            "fromStateCode": int(seller_details.state_id.l10n_in_tin) or "",
            "actFromStateCode": dispatch_details.state_id.l10n_in_tin and int(dispatch_details.state_id.l10n_in_tin) or "",
            "toGstin": buyer_details.commercial_partner_id.vat or "URP",
            "toTrdName": buyer_details.commercial_partner_id.name,
            "toAddr1": ship_to_details.street or "",
            "toAddr2": ship_to_details.street2 or "",
            "toPlace": ship_to_details.city or "",
            "toPincode": int(extract_digits(ship_to_details.zip)),
            "actToStateCode": int(ship_to_details.state_id.l10n_in_tin),
            "toStateCode": invoices.l10n_in_state_id.l10n_in_tin and int(invoices.l10n_in_state_id.l10n_in_tin) or (
                buyer_details.state_id.l10n_in_tin or int(buyer_details.state_id.l10n_in_tin) or ""
            ),
            "itemList": [
                self._get_l10n_in_edi_ewaybill_line_details(line, line_tax_details, sign)
                for line, line_tax_details in invoice_line_tax_details.items()
            ],
            "totalValue": self._l10n_in_round_value(tax_details.get("base_amount")),
            "cgstValue": self._l10n_in_round_value(tax_details_by_code.get("cgst_amount", 0.00)),
            "sgstValue": self._l10n_in_round_value(tax_details_by_code.get("sgst_amount", 0.00)),
            "igstValue": self._l10n_in_round_value(tax_details_by_code.get("igst_amount", 0.00)),
            "cessValue": self._l10n_in_round_value(tax_details_by_code.get("cess_amount", 0.00)),
            "cessNonAdvolValue": self._l10n_in_round_value(tax_details_by_code.get("cess_non_advol_amount", 0.00)),
            "otherValue": self._l10n_in_round_value(tax_details_by_code.get("other_amount", 0.00) + rounding_amount),
            "totInvValue": self._l10n_in_round_value(tax_details.get("base_amount") + tax_details.get("tax_amount") + rounding_amount),
        }
        is_overseas = invoices.l10n_in_gst_treatment in ("overseas", "special_economic_zone")
        if invoices.is_outbound():
            if is_overseas:
                json_payload.update({"fromStateCode": 99})
            if is_overseas and dispatch_details.state_id.country_id.code != "IN":
                json_payload.update({
                    "actFromStateCode": 99,
                    "fromPincode": 999999,
                })
            else:
                json_payload.update({
                    "actFromStateCode": dispatch_details.state_id.l10n_in_tin and int(dispatch_details.state_id.l10n_in_tin) or "",
                    "fromPincode": int(extract_digits(dispatch_details.zip)),
                })
        else:
            if is_overseas:
                json_payload.update({"toStateCode": 99})
            if is_overseas and ship_to_details.state_id.country_id.code != "IN":
                json_payload.update({
                    "actToStateCode": 99,
                    "toPincode": 999999,
                })
            else:
                json_payload.update({
                    "actToStateCode": int(ship_to_details.state_id.l10n_in_tin),
                    "toPincode": int(extract_digits(ship_to_details.zip)),
                })

        if invoices.l10n_in_mode == "0":
            json_payload.update({
                "transporterId": invoices.l10n_in_transporter_id.vat or "",
                "transporterName": invoices.l10n_in_transporter_id.name or "",
            })
        if invoices.l10n_in_mode in ("2", "3", "4"):
            json_payload.update({
                "transMode": invoices.l10n_in_mode,
                "transDocNo": invoices.l10n_in_transportation_doc_no or "",
                "transDocDate": invoices.l10n_in_transportation_doc_date and
                    invoices.l10n_in_transportation_doc_date.strftime("%d/%m/%Y") or "",
            })
        if invoices.l10n_in_mode == "1":
            json_payload.update({
                "transMode": invoices.l10n_in_mode,
                "vehicleNo": invoices.l10n_in_vehicle_no or "",
                "vehicleType": invoices.l10n_in_vehicle_type or "",
            })
        return json_payload

    def _get_l10n_in_edi_ewaybill_line_details(self, line, line_tax_details, sign):
        extract_digits = self._l10n_in_edi_extract_digits
        tax_details_by_code = self._get_l10n_in_tax_details_by_line_code(line_tax_details.get("tax_details", {}))
        line_details = {
            "productName": line.product_id.name,
            "hsnCode": extract_digits(line.l10n_in_hsn_code),
            "productDesc": line.name,
            "quantity": line.quantity,
            "qtyUnit": line.product_uom_id.l10n_in_code and line.product_uom_id.l10n_in_code.split("-")[0] or "OTH",
            "taxableAmount": self._l10n_in_round_value(line.balance * sign),
        }
        gst_types = {'cgst', 'sgst', 'igst'}
        gst_tax_rates = {
            f"{gst_type}Rate": self._l10n_in_round_value(gst_tax_rate)
            for gst_type in gst_types
            if (gst_tax_rate := tax_details_by_code.get(f"{gst_type}_rate"))
        }
        line_details.update(
            gst_tax_rates or dict.fromkeys({f"{gst_type}Rate" for gst_type in gst_types}, 0.00)
        )
        if tax_details_by_code.get("cess_rate"):
            line_details.update({"cessRate": self._l10n_in_round_value(tax_details_by_code.get("cess_rate"))})
        return line_details

    #================================ E-invoice API methods ===========================

    @api.model
    def _l10n_in_edi_irn_ewaybill_generate(self, company, json_payload):
        # IRN is created by E-invoice API call so waiting for it.
        if not json_payload.get("Irn"):
            return {"error": [{
                "code": "waiting",
                "message": _("waiting For IRN generation To create E-waybill")}
            ]}
        token = self._l10n_in_edi_get_token(company)
        if not token:
            return self._l10n_in_edi_no_config_response()
        params = {
            "auth_token": token,
            "json_payload": json_payload,
        }
        return self._l10n_in_edi_connect_to_server(company, url_path="/iap/l10n_in_edi/1/generate_ewaybill_by_irn", params=params)

    @api.model
    def _l10n_in_edi_irn_ewaybill_get(self, company, irn):
        token = self._l10n_in_edi_get_token(company)
        if not token:
            return self._l10n_in_edi_no_config_response()
        params = {
            "auth_token": token,
            "irn": irn,
        }
        return self._l10n_in_edi_connect_to_server(company, url_path="/iap/l10n_in_edi/1/get_ewaybill_by_irn", params=params)

    #=============================== E-waybill API methods ===================================

    @api.model
    def _l10n_in_edi_ewaybill_no_config_response(self):
        return {"error": [{
            "code": "0",
            "message": _(
                "Unable to send E-waybill."
                "Create an API user in NIC portal, and set it using the top menu: Configuration > Settings."
            )}
        ]}

    @api.model
    def _l10n_in_edi_ewaybill_check_authentication(self, company):
        sudo_company = company.sudo()
        if sudo_company.l10n_in_edi_ewaybill_username and sudo_company._l10n_in_edi_ewaybill_token_is_valid():
            return True
        elif sudo_company.l10n_in_edi_ewaybill_username and sudo_company.l10n_in_edi_ewaybill_password:
            authenticate_response = self._l10n_in_edi_ewaybill_authenticate(company)
            if not authenticate_response.get("error"):
                return True
        return False

    def _l10n_in_set_missing_error_message(self, response):
        for error in response.get('error', []):
            if error.get('code') and not error.get('message'):
                error['message'] = self._l10n_in_edi_ewaybill_get_error_message(error.get('code'))
        return response

    @api.model
    def _l10n_in_edi_ewaybill_connect_to_server(self, company, url_path, params):
        params.update({
            "username": company.sudo().l10n_in_edi_ewaybill_username,
            "gstin": company.vat,
        })
        try:
            response = self.env['iap.account']._l10n_in_connect_to_server(
                is_production=company.sudo().l10n_in_edi_production_env,
                params=params,
                url_path=url_path,
                config_parameter="l10n_in_edi_ewaybill.endpoint",
                timeout=70
            )
            return self._l10n_in_set_missing_error_message(response)
        except AccessError as e:
            _logger.warning("Connection error: %s", e.args[0])
            return {
                "error": [{
                    "code": "access_error",
                    "message": _("Unable to connect to the E-WayBill service."
                        "The web service may be temporary down. Please try again in a moment.")
                }]
            }

    @api.model
    def _l10n_in_edi_ewaybill_authenticate(self, company):
        params = {"password": company.sudo().l10n_in_edi_ewaybill_password}
        response = self._l10n_in_edi_ewaybill_connect_to_server(
            company, url_path="/iap/l10n_in_edi_ewaybill/1/authenticate", params=params
        )
        if response and response.get("status_cd") == "1":
            company.sudo().l10n_in_edi_ewaybill_auth_validity = fields.Datetime.now() + timedelta(
                hours=6, minutes=00, seconds=00)
        return response

    @api.model
    def _l10n_in_edi_ewaybill_generate(self, company, json_payload):
        is_authenticated = self._l10n_in_edi_ewaybill_check_authentication(company)
        if not is_authenticated:
            return self._l10n_in_edi_ewaybill_no_config_response()
        params = {"json_payload": json_payload}
        return self._l10n_in_edi_ewaybill_connect_to_server(
            company, url_path="/iap/l10n_in_edi_ewaybill/1/generate", params=params
        )

    @api.model
    def _l10n_in_edi_ewaybill_cancel(self, company, json_payload):
        is_authenticated = self._l10n_in_edi_ewaybill_check_authentication(company)
        if not is_authenticated:
            return self._l10n_in_edi_ewaybill_no_config_response()
        params = {"json_payload": json_payload}
        return self._l10n_in_edi_ewaybill_connect_to_server(
            company, url_path="/iap/l10n_in_edi_ewaybill/1/cancel", params=params
        )

    @api.model
    def _l10n_in_edi_ewaybill_get_by_consigner(self, company, document_type, document_number):
        is_authenticated = self._l10n_in_edi_ewaybill_check_authentication(company)
        if not is_authenticated:
            return self._l10n_in_edi_ewaybill_no_config_response()
        params = {"document_type": document_type, "document_number": document_number}
        return self._l10n_in_edi_ewaybill_connect_to_server(
            company, url_path="/iap/l10n_in_edi_ewaybill/1/getewaybillgeneratedbyconsigner", params=params
        )

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
from odoo import fields, models, api, _
from odoo.exceptions import UserError


class AccountMove(models.Model):
    _inherit = "account.move"

    # Transaction Details
    l10n_in_type_id = fields.Many2one("l10n.in.ewaybill.type", "E-waybill Document Type", tracking=True)

    # transportation details
    l10n_in_distance = fields.Integer("Distance", tracking=True)
    l10n_in_mode = fields.Selection([
        ("0", "Managed by Transporter"),
        ("1", "Road"),
        ("2", "Rail"),
        ("3", "Air"),
        ("4", "Ship")],
        string="Transportation Mode", copy=False, tracking=True)

    # Vehicle Number and Type required when transportation mode is By Road.
    l10n_in_vehicle_no = fields.Char("Vehicle Number", copy=False, tracking=True)
    l10n_in_vehicle_type = fields.Selection([
        ("R", "Regular"),
        ("O", "Over Dimensional Cargo")],
        string="Vehicle Type", copy=False, tracking=True)

    # Document number and date required in case of transportation mode is Rail, Air or Ship.
    l10n_in_transportation_doc_no = fields.Char(
        string="E-waybill Document Number",
        help="""Transport document number. If it is more than 15 chars, last 15 chars may be entered""",
        copy=False, tracking=True)
    l10n_in_transportation_doc_date = fields.Date(
        string="Document Date",
        help="Date on the transporter document",
        copy=False,
        tracking=True)

    # transporter id required when transportation done by other party.
    l10n_in_transporter_id = fields.Many2one("res.partner", "Transporter", copy=False, tracking=True)
    # show and hide fields base on this
    l10n_in_edi_ewaybill_direct_api = fields.Boolean(string="E-waybill(IN) direct API", compute="_compute_l10n_in_edi_ewaybill_direct")
    l10n_in_edi_ewaybill_show_send_button = fields.Boolean(string="Show Send E-waybill Button", compute="_compute_l10n_in_edi_ewaybill_show_send_button")

    @api.depends('state', 'edi_document_ids', 'edi_document_ids.state')
    def _compute_l10n_in_edi_ewaybill_show_send_button(self):
        edi_format = self.env.ref('l10n_in_edi_ewaybill.edi_in_ewaybill_json_1_03', raise_if_not_found=False)
        if not edi_format:
            self.l10n_in_edi_ewaybill_show_send_button = False
            return
        posted_moves = self.filtered(lambda x: x.move_type in ('out_invoice', 'in_invoice', 'in_refund') and x.state == 'posted' and x.country_code == "IN")
        for move in posted_moves:
            already_sent = move.edi_document_ids.filtered(lambda x: x.edi_format_id == edi_format and x.state in ('sent', 'to_cancel', 'to_send'))
            if already_sent:
                move.l10n_in_edi_ewaybill_show_send_button = False
            else:
                move.l10n_in_edi_ewaybill_show_send_button = True
        (self - posted_moves).l10n_in_edi_ewaybill_show_send_button = False

    @api.depends("l10n_in_gst_treatment")
    def _compute_l10n_in_edi_ewaybill_direct(self):
        for move in self:
            base = self.env["account.edi.format"]._l10n_in_edi_ewaybill_base_irn_or_direct(move)
            move.l10n_in_edi_ewaybill_direct_api = base == "direct"

    @api.depends("edi_document_ids")
    def _compute_l10n_in_edi_show_cancel(self):
        super()._compute_l10n_in_edi_show_cancel()
        for invoice in self:
            if invoice.edi_document_ids.filtered(lambda i: i.edi_format_id.code == "in_ewaybill_1_03" and i.state in ("sent", "to_cancel", "cancelled")):
                invoice.l10n_in_edi_show_cancel = True

    def _get_l10n_in_edi_ewaybill_response_json(self):
        self.ensure_one()
        l10n_in_edi = self.edi_document_ids.filtered(lambda i: i.edi_format_id.code == "in_ewaybill_1_03"
            and i.state in ("sent", "to_cancel"))
        if l10n_in_edi and l10n_in_edi.sudo().attachment_id:
            return json.loads(l10n_in_edi.sudo().attachment_id.raw.decode("utf-8"))
        else:
            return {}

    def button_cancel_posted_moves(self):
        """Mark the edi.document related to this move to be canceled."""
        reason_and_remarks_not_set = self.env["account.move"]
        for move in self:
            send_l10n_in_edi_ewaybill = move.edi_document_ids.filtered(lambda doc: doc.edi_format_id.code == "in_ewaybill_1_03")
            # check submitted E-waybill does not have reason and remarks
            # because it's needed to cancel E-waybill
            if send_l10n_in_edi_ewaybill and (not move.l10n_in_edi_cancel_reason or not move.l10n_in_edi_cancel_remarks):
                reason_and_remarks_not_set += move
        if reason_and_remarks_not_set:
            raise UserError(_(
                "To cancel E-waybill set cancel reason and remarks at E-waybill tab in: \n%s",
                ("\n".join(reason_and_remarks_not_set.mapped("name"))),
            ))
        return super().button_cancel_posted_moves()

    def l10n_in_edi_ewaybill_send(self):
        edi_format = self.env.ref('l10n_in_edi_ewaybill.edi_in_ewaybill_json_1_03')
        edi_document_vals_list = []
        for move in self:
            if move.state != 'posted':
                raise UserError(_("You can only create E-waybill from posted invoice"))
            errors = edi_format._check_move_configuration(move)
            if errors:
                raise UserError(_("Invalid invoice configuration:\n\n%s", '\n'.join(errors)))
            existing_edi_document = move.edi_document_ids.filtered(lambda x: x.edi_format_id == edi_format)
            if existing_edi_document:
                if existing_edi_document.state in ('sent', 'to_cancel'):
                    raise UserError(_("E-waybill is already created"))
                existing_edi_document.sudo().write({
                    'state': 'to_send',
                    'attachment_id': False,
                })
            else:
                edi_document_vals_list.append({
                    'edi_format_id': edi_format.id,
                    'move_id': move.id,
                    'state': 'to_send',
                })
        self.env['account.edi.document'].create(edi_document_vals_list)
        self.env.ref('account_edi.ir_cron_edi_network')._trigger()

    def _can_force_cancel(self):
        # OVERRIDE
        self.ensure_one()
        return any(document.edi_format_id.code == 'in_ewaybill_1_03' and document.state == 'to_cancel' for document in self.edi_document_ids) or super()._can_force_cancel()

```

## File: models\error_codes.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.tools.translate import LazyTranslate

_lt = LazyTranslate(__name__)

ERROR_CODES = {
    "100": _lt("Invalid json"),
    "101": _lt("Invalid Username"),
    "102": _lt("Invalid Password"),
    "103": _lt("Invalid Client -Id"),
    "104": _lt("Invalid Client -Id"),
    "105": _lt("Invalid Token"),
    "106": _lt("Token Expired"),
    "107": _lt("Authentication failed. Pls. inform the helpdesk"),
    "108": _lt("Invalid login credentials."),
    "109": _lt("Decryption of data failed"),
    "110": _lt("Invalid Client-ID/Client-Secret"),
    "111": _lt("GSTIN is not registerd to this GSP"),
    "112": _lt("IMEI does not belong to the user"),
    "113": _lt("operating-system-type is mandatory in header"),
    "114": _lt("Invalid operating-system-type parameter value"),
    "201": _lt("Invalid Supply Type"),
    "202": _lt("Invalid Sub-supply Type"),
    "203": _lt("Sub-transaction type does not belongs to transaction type"),
    "204": _lt("Invalid Document type"),
    "205": _lt("Document type does not match with transaction & Sub trans type"),
    "206": _lt("Invaild Invoice Number"),
    "207": _lt("Invalid Invoice Date"),
    "208": _lt("Invalid Supplier GSTIN"),
    "209": _lt("Blank Supplier Address"),
    "210": _lt("Invalid or Blank Supplier PIN Code"),
    "211": _lt("Invalid or Blank Supplier state Code"),
    "212": _lt("Invalid Consignee GSTIN"),
    "213": _lt("Invalid Consignee Address"),
    "214": _lt("Invalid Consignee PIN Code"),
    "215": _lt("Invalid Consignee State Code"),
    "216": _lt("Invalid HSN Code"),
    "217": _lt("Invalid UQC Code"),
    "218": _lt("Invalid Tax Rate for Intra State Transaction"),
    "219": _lt("Invalid Tax Rate for Inter State Transaction"),
    "220": _lt("Invalid Trans mode"),
    "221": _lt("Invalid Approximate Distance"),
    "222": _lt("Invalid Transporter Id"),
    "223": _lt("Invalid Transaction Document Number"),
    "224": _lt("Invalid Transaction Date"),
    "225": _lt("Invalid Vehicle Number Format"),
    "226": _lt("Both Transaction and Vehicle Number Blank"),
    "227": _lt("User Gstin cannot be blank"),
    "228": _lt("User id cannot be blank"),
    "229": _lt("Supplier name is required"),
    "230": _lt("Supplier place is required"),
    "231": _lt("Consignee name is required"),
    "232": _lt("Consignee place is required"),
    "233": _lt("Eway bill does not contains any items"),
    "234": _lt("Total amount/Taxable amout is mandatory"),
    "235": _lt("Tax rates for Intra state transaction is blank"),
    "236": _lt("Tax rates for Inter state transaction is blank"),
    "237": _lt("Invalid client -Id/client-secret"),
    "238": _lt("Invalid auth token"),
    "239": _lt("Invalid action"),
    "240": _lt("Could not generate eway bill, pls contact helpdesk"),
    "242": _lt("Invalid or Blank Officer StateCode"),
    "243": _lt("Invalid or Blank IR Number"),
    "244": _lt("Invalid or Blank Actual Vehicle Number Format"),
    "245": _lt("Invalid Verification Date Format"),
    "246": _lt("Invalid Vehicle Release Date Format"),
    "247": _lt("Invalid Verification Time Format"),
    "248": _lt("Invalid Vehicle Release Date Format"),
    "249": _lt("Actual Value cannot be less than or equal to zero"),
    "250": _lt("Invalid Vehicle Release Date Format"),
    "251": _lt("CGST nad SGST TaxRate should be same"),
    "252": _lt("Invalid CGST Tax Rate"),
    "253": _lt("Invalid SGST Tax Rate"),
    "254": _lt("Invalid IGST Tax Rate"),
    "255": _lt("Invalid CESS Rate"),
    "256": _lt("Invalid Cess Non Advol value"),
    "278": _lt("User Gstin does not match with Transporter Id"),
    "280": _lt("Status is not ACTIVE"),
    "281": _lt("Eway Bill is already expired hence update transporter is not allowed."),
    "301": _lt("Invalid eway bill number"),
    "302": _lt("Invalid transporter mode"),
    "303": _lt("Vehicle number is required"),
    "304": _lt("Invalid vehicle format"),
    "305": _lt("Place from is required"),
    "306": _lt("Invalid from state"),
    "307": _lt("Invalid reason"),
    "308": _lt("Invalid remarks"),
    "309": _lt("Could not update vehicle details, pl contact helpdesk"),
    "311": _lt("Validity period lapsed, you cannot update vehicle details"),
    "312": _lt("This eway bill is either not generated by you or cancelled"),
    "313": _lt("Error in validating ewaybill for vehicle updation"),
    "315": _lt("Validity period lapsed, you cannot cancel this eway bill"),
    "316": _lt("Eway bill is already verified, you cannot cancel it"),
    "317": _lt("Could not cancel eway bill, please contact helpdesk"),
    "320": _lt("Invalid state to"),
    "321": _lt("Invalid place to"),
    "322": _lt("Could not generate consolidated eway bill"),
    "325": _lt("Could not retrieve data"),
    "326": _lt("Could not retrieve GSTIN details for the given GSTIN number"),
    "327": _lt("Could not retrieve data from hsn"),
    "328": _lt("Could not retrieve transporter details from gstin"),
    "329": _lt("Could not retrieve States List"),
    "330": _lt("Could not retrieve UQC list"),
    "331": _lt("Could not retrieve Error code"),
    "334": _lt("Could not retrieve user details by userid "),
    "336": _lt("Could not retrieve transporter data by gstin "),
    "337": _lt("Could not retrieve HSN details for the given HSN number"),
    "338": _lt("You cannot update transporter details, as the current tranporter is already entered Part B details of the eway bill"),
    "339": _lt("You are not assigned to update the tranporter details of this eway bill"),
    "341": _lt("This e-way bill is generated by you and hence you cannot reject it"),
    "342": _lt("You cannot reject this e-way bill as you are not the other party to do so"),
    "343": _lt("This e-way bill is cancelled"),
    "344": _lt("Invalid eway bill number"),
    "345": _lt("Validity period lapsed, you cannot reject the e-way bill"),
    "346": _lt("You can reject the e-way bill only within 72 hours from generated time"),
    "347": _lt("Validation of eway bill number failed, while rejecting ewaybill"),
    "348": _lt("Part-B is not generated for this e-way bill, hence rejection is not allowed."),
    "350": _lt("Could not generate consolidated eway bill"),
    "351": _lt("Invalid state code"),
    "352": _lt("Invalid rfid date"),
    "353": _lt("Invalid location code"),
    "354": _lt("Invalid rfid number"),
    "355": _lt("Invalid Vehicle Number Format"),
    "356": _lt("Invalid wt on bridge"),
    "357": _lt("Could not retrieve eway bill details, pl. contact helpdesk"),
    "358": _lt("GSTIN passed in request header is not matching with the user gstin mentioned in payload JSON"),
    "359": _lt("User GSTIN should match to GSTIN(from) for outward transactions"),
    "360": _lt("User GSTIN should match to GSTIN(to) for inward transactions"),
    "361": _lt("Invalid Vehicle Type"),
    "362": _lt("Transporter document date cannot be earlier than the invoice date"),
    "363": _lt("E-way bill is not enabled for intra state movement for you state"),
    "364": _lt("Error in verifying eway bill"),
    "365": _lt("Error in verifying consolidated eway bill"),
    "366": _lt("You will not get the ewaybills generated today, howerver you cann access the ewaybills of yester days"),
    "367": _lt("Could not retrieve data for officer login"),
    "368": _lt("Could not update transporter"),
    "369": _lt("GSTIN/Transin passed in request header should match with the transported Id mentioned in payload JSON"),
    "370": _lt("GSTIN/Transin passed in request header should not be the same as supplier(fromGSTIN) or recepient(toGSTIN)"),
    "371": _lt("Invalid or Blank Supplier Ship-to State Code"),
    "372": _lt("Invalid or Blank Consignee Ship-to State Code"),
    "373": _lt("The Supplier ship-to state code should be Other Country for Sub Supply Type- Export"),
    "374": _lt("The Consignee pin code should be 999999 for Sub Supply Type- Export"),
    "375": _lt("The Supplier ship-from state code should be Other Country for Sub Supply Type- Import"),
    "376": _lt("The Supplier pin code should be 999999 for Sub Supply Type- Import"),
    "377": _lt("Sub Supply Type is mentioned as Others, the description for that is mandatory"),
    "378": _lt("The supplier or conginee belong to SEZ, Inter state tax rates are applicable here"),
    "379": _lt("Eway Bill can not be extended.. Already Cancelled"),
    "380": _lt("Eway Bill Can not be Extended. Not in Active State"),
    "381": _lt("There is No PART-B/Vehicle Entry.. So Please Update Vehicle Information.."),
    "382": _lt("You Cannot Extend as EWB can be Extended only 8 hour before or after w.r.t Validity of EWB..!!"),
    "383": _lt("Error While Extending..Please Contact Helpdesk. "),
    "384": _lt("You are not current transporter or Generator of the ewayBill, with no transporter details."),
    "385": _lt("For Rail/Ship/Air transDocDate is mandatory"),
    "386": _lt("Reason Code, Remarks is mandatory."),
    "387": _lt("No Record Found for Entered consolidated eWay bill."),
    "388": _lt("Exception in regenration of consolidated eWayBill!!Please Contact helpdesk"),
    "389": _lt("Remaining Distance Required"),
    "390": _lt("Remaining Distance Can not be greater than Actual Distance."),
    "391": _lt("No eway bill of specified tripsheet, neither  ACTIVE nor not Valid."),
    "392": _lt("Tripsheet is already cancelled, Hence Regeration is not possible"),
    "393": _lt("Invalid GSTIN"),
    "394": _lt("For other than Road Transport, TransDoc number is required"),
    "395": _lt("Eway Bill Number should be numeric only"),
    "396": _lt("Either Eway Bill Number Or Consolidated Eway Bill Number is required for Verification"),
    "397": _lt("Error in Multi Vehicle Movement Initiation"),
    "398": _lt("Eway Bill Item List is Empty"),
    "399": _lt("Unit Code is not matching with any of the Unit Code from eway bill ItemList"),
    "400": _lt("total quantity is exceeding from multi vehicle movement initiation quantity"),
    "401": _lt("Error in inserting multi vehicle details"),
    "402": _lt("total quantity can not be less than or equal to zero"),
    "403": _lt("Error in multi vehicle details"),
    "405": _lt("No record found for multi vehicle update with specified ewbNo groupNo and old vehicleNo/transDocNo with status as ACT"),
    "406": _lt("Group number cannot be empty or zero"),
    "407": _lt("Invalid old vehicle number format"),
    "408": _lt("Invalid new vehicle number format"),
    "409": _lt("Invalid old transDoc number"),
    "410": _lt("Invalid new transDoc number"),
    "411": _lt("Multi Vehicle Initiation data is not there for specified ewayBill and group No"),
    "412": _lt("Multi Vehicle movement is already Initiated,hence PART B updation not allowed"),
    "413": _lt("Unit Code is not matching with unit code of first initiaton"),
    "415": _lt("Error in fetching in verification data for officer"),
    "416": _lt("Date range is exceeding allowed date range "),
    "417": _lt("No verification data found for officer "),
    "418": _lt("No record found"),
    "419": _lt("Error in fetching search result for taxpayer/transporter"),
    "420": _lt("Minimum six character required for Tradename/legalname search"),
    "421": _lt("Invalid pincode"),
    "422": _lt("Invalid mobile number"),
    "423": _lt("Error in fetching ewaybill list by vehicle number"),
    "424": _lt("Invalid PAN number"),
    "425": _lt("Error in fetching Part A data by IR Number"),
    "426": _lt("For Vehicle Released vehicle release date and time is mandatory"),
    "427": _lt("Error in saving Part-A verification Report"),
    "428": _lt("For Goods Detained,Vehicle Released feild is mandatory"),
    "429": _lt("Error in saving Part-B verification Report"),
    "430": _lt("Goods Detained Field required."),
    "431": _lt("Part-A for this ewaybill is already generated by you."),
    "432": _lt("invalid vehicle released value"),
    "433": _lt("invalid goods detained parameter value"),
    "434": _lt("invalid ewbNoAvailable parameter value"),
    "435": _lt("Part B is already updated,hence updation is not allowed"),
    "436": _lt("Invalid Consignee ship to State Code for the given pincode"),
    "437": _lt("Invalid Supplier ship from State Code for the given pincode"),
    "438": _lt("Invalid Latitude"),
    "439": _lt("Invalid Longitude"),
    "440": _lt("Error in inserting in verification data"),
    "441": _lt("Invalid verification type"),
    "442": _lt("Error in inserting verification details"),
    "443": _lt("invalid invoice available value"),
    "600": _lt("Invalid category"),
    "601": _lt("Invalid date format"),
    "602": _lt("Invalid File Number"),
    "603": _lt("For file details file number is required"),
    "604": _lt("E-way bill(s) are already generated for the same document number, you cannot generate again on same document number"),
    "607": _lt("dispatch from gstin is mandatary "),
    "608": _lt("ship to from gstin is mandatary"),
    "609": _lt(" invalid ship to from gstin "),
    "610": _lt("invalid dispatch from gstin "),
    "611": _lt("invalid document type for the given supply type "),
    "612": _lt("Invalid transaction type"),
    "613": _lt("Exception in getting Officer Role"),
    "614": _lt("Transaction type is mandatory"),
    "615": _lt("Dispatch From GSTIN cannot be sent as the transaction type selected is Regular"),
    "616": _lt("Ship to GSTIN cannot be sent as the transaction type selected is Regular"),
    "617": _lt("Bill-from and dispatch-from gstin should not be same for this transaction type"),
    "618": _lt("Bill-to and ship-to gstin should not be same for this transaction type"),
    "619": _lt("Transporter Id is mandatory for generation of Part A slip"),
    "620": _lt("Total invoice value cannot be less than the sum of total assessible value and tax values"),
    "621": _lt("Transport mode is mandatory since vehicle number is present"),
    "622": _lt("Transport mode is mandatory since transport document number is present"),
    "623": _lt("IGST value is not applicable for Intra State Transaction"),
    "624": _lt("CGST/SGST value is not applicable for Inter State Transaction"),
    "627": _lt("Total value should not be negative"),
    "628": _lt("Total invoice value should not be negative"),
    "629": _lt("IGST value should not be negative"),
    "630": _lt("CGST value should not be negative"),
    "631": _lt("SGST value should not be negative"),
    "632": _lt("Cess value should not be negative"),
    "633": _lt("Cess non advol should not be negative"),
    "634": _lt("Vehicle type should not be ODC when transmode is other than road"),
    "635": _lt("You cannot update part B, as the current tranporter is already entered Part B details of the eway bill"),
    "636": _lt("You are not assigned to update part B"),
    "637": _lt("You cannot extend ewaybill, as the current tranporter is already entered Part B details of the ewaybill"),
    "638": _lt("Transport mode is mandatory as Vehicle Number/Transport Document Number is given"),
    "640": _lt("Tolal Invoice value is mandatory"),
    "641": _lt("For outward CKD/SKD/Lots supply type, Bill To state should be as Other Country, since the  Bill To GSTIN given is of SEZ unit"),
    "642": _lt("For inward CKD/SKD/Lots supply type, Bill From state should be as Other Country, since the  Bill From GSTIN given is of SEZ unit"),
    "643": _lt("For regular transaction, Bill from state code and Dispatch from state code should be same"),
    "644": _lt("For regular transaction, Bill to state code and Ship to state code should be same"),
    "645": _lt("You cannot do multivehicle movement, as the current tranporter is already entered Part B details of the ewaybill"),
    "646": _lt("You are not assigned to do MultiVehicle Movement"),
    "647": _lt("Could not insert RFID data, pl. contact helpdisk"),
    "648": _lt("Multi Vehicle movement is already Initiated,hence generation of consolidated eway bill is not allowed"),
    "649": _lt("You cannot generate consolidated eway bill , as the current tranporter is already entered Part B details of the eway bill"),
    "650": _lt("You are not assigned to generate consolidated ewaybill"),
    "651": _lt("For Category Part-A or Part-B ewbdt is mandatory"),
    "652": _lt("For Category EWB03 procdt is mandatory"),
    "654": _lt("This GSTIN has generated a common Enrolment Number. Hence you are not allowed to generate Eway bill"),
    "655": _lt("This GSTIN has generated a common Enrolment Number. Hence you cannot mention it as a tranporter"),
    "656": _lt("This Eway Bill does not belongs to your state"),
    "657": _lt("Eway Bill Category wise details will be available after 4 days only"),
    "658": _lt("You are blocked for accesing this API as the allowed number of requests has been exceeded for this duration"),
    "659": _lt("Remarks is mandatory"),
    "670": _lt("Invalid Month Parameter"),
    "671": _lt("Invalid Year Parameter"),
    "672": _lt("User Id is mandatory"),
    "673": _lt("Error in getting officer dashboard"),
    "675": _lt("Error in getting EWB03 details by acknowledgement date range"),
    "676": _lt("Error in getting EWB Not Available List by entered date range"),
    "677": _lt("Error in getting EWB Not Available List by closed date range"),
    "678": _lt("Invalid Uniq No"),
    "679": _lt("Invalid EWB03 Ack No"),
    "680": _lt("Invalid Close Reason"),
    "681": _lt("Error in Closing EWB  Verification Data"),
    "682": _lt("No Record available to Close"),
    "683": _lt("Error in fetching WatchList Data"),
    "685": _lt("Exception in fetching dashboard data"),
    "700": _lt("You are not assigned to extend e-waybill"),
    "701": _lt("Invalid Vehicle Direction"),
    "702": _lt("The distance between the pincodes given is too high"),
    "703": _lt("Since the consignor is Composite Taxpayer, inter state transactions are not allowed"),
    "704": _lt("Since the consignor is Composite Taxpayer, Tax rates should be zero"),
    "705": _lt("Invalid transit type"),
    "706": _lt("Address Line1 is mandatory"),
    "707": _lt("Address Line2 is mandatory"),
    "708": _lt("Address Line3 is mandatory"),
    "709": _lt("Pin to pin distance is not available for the given pin codes"),
    "710": _lt("Invalid state code for the given pincode"),
    "711": _lt("Invalid consignment status for the given transmode"),
    "712": _lt("Transit Type is not required as the goods are in movement"),
    "713": _lt("Transit Address is not required as the goods are in movement"),
    "714": _lt("Document type - Tax Invoice is not allowed for composite tax payer"),
    "715": _lt("The Consignor GSTIN is blocked from e-waybill generation as Return is not filed for past 2 months"),
    "716": _lt("The Consignee GSTIN is blocked from e-waybill generation as Return is not filed for past 2 months"),
    "717": _lt("The Transporter GSTIN is blocked from e-waybill generation as Return is not filed for past 2 months"),
    "718": _lt("The User GSTIN is blocked from Transporter Updation as Return is not filed for past 2 months"),
    "719": _lt("The Transporter GSTIN is blocked from Transporter Updation as Return is not filed for past 2 months"),
    "720": _lt("E Way Bill should be generated as part of IRN generation or with reference to IRN in E Invoice System, Since Supplier is enabled for E Invoice."),
    "721": _lt("The distance between the given pincodes are not available in the system. Please provide distance."),
    "722": _lt("Consignee GSTIN is cancelled and document date is later than the  De-Registration date"),
    "724": _lt("HSN code of at least one item should be of goods to generate e-Way Bill"),
    "726": _lt("Vehicle type can not be regular when transportation mode is ship"),
    "727": _lt("This e-Way Bill does not have Oxygen items"),
    "728": _lt("You can cancel the ewaybill within 24 hours from Part B entry"),
    "801": _lt("Transporter id is not required for ewaybill for gold"),
    "802": _lt("Transporter name is not required for ewaybill for gold"),
    "803": _lt("TransDocNo is not required for ewaybill for gold"),
    "804": _lt("TransDocDate is not required for ewaybill for gold"),
    "805": _lt("Vehicle No is not required for ewaybill for gold"),
    "806": _lt("Vehicle Type is not required for ewaybill for gold"),
    "807": _lt("Transmode is mandatory for ewaybill for gold"),
    "808": _lt("Inter-State ewaybill is not allowed for gold"),
    "809": _lt("Other items are not allowed with eway bill for gold"),
    "810": _lt("Transport can not be updated for EwayBill For Gold"),
    "811": _lt("Vehicle can not be updated for EwayBill For Gold"),
    "812": _lt("ConsolidatedEWB cannot be generated for EwayBill For Gold "),
    "813": _lt("Duplicate request at the same time"),
    "814": _lt("MultiVehicleMovement cannot be initiated for EWay Bill For Gold"),
    "815": _lt("Only trans mode road is allowed for Eway Bill For Gold"),
    "816": _lt("Only transmode road is allowed for extending ewaybill for gold"),
    "817": _lt("MultiVehicleMovement cannot be initiated.Eway Bill is not in Active State"),
    "818": _lt("Validity period lapsed.Cannot generate consolidated Eway Bill"),
    "819": _lt("Ewaybill cannot be generated for the document date which is prior to 01/07/2017"),
}

```

## File: models\ewaybill_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class EWayBillType(models.Model):
    _name = "l10n.in.ewaybill.type"
    _description = "E-Waybill Document Type"

    name = fields.Char("Type")
    code = fields.Char("Type Code")
    sub_type = fields.Char("Sub-type")
    sub_type_code = fields.Char("Sub-type Code")
    allowed_supply_type = fields.Selection(
        [
            ("both", "Incoming and Outgoing"),
            ("out", "Outgoing"),
            ("in", "Incoming"),
        ],
        string="Allowed for supply type",
    )
    active = fields.Boolean("Active", default=True)

    @api.depends('sub_type')
    def _compute_display_name(self):
        """Show name and sub_type in name"""
        for ewaybill_type in self:
            ewaybill_type.display_name = _("%(name)s (Sub-Type: %(type)s)", name=ewaybill_type.name, type=ewaybill_type.sub_type)

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = "res.company"

    l10n_in_edi_ewaybill_username = fields.Char("E-Waybill (IN) Username", groups="base.group_system")
    l10n_in_edi_ewaybill_password = fields.Char("E-Waybill (IN) Password", groups="base.group_system")
    l10n_in_edi_ewaybill_auth_validity = fields.Datetime("E-Waybill (IN) Valid Until", groups="base.group_system")

    def _l10n_in_edi_ewaybill_token_is_valid(self):
        self.ensure_one()
        if self.l10n_in_edi_ewaybill_auth_validity and self.l10n_in_edi_ewaybill_auth_validity > fields.Datetime.now():
            return True
        return False

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, _
from odoo.exceptions import UserError
from odoo.tools import html_escape


class ResConfigSettings(models.TransientModel):
    _inherit = "res.config.settings"

    l10n_in_edi_ewaybill_username = fields.Char("Indian EDI Stock username",
        related="company_id.l10n_in_edi_ewaybill_username", readonly=False)
    l10n_in_edi_ewaybill_password = fields.Char("Indian EDI Stock password",
        related="company_id.l10n_in_edi_ewaybill_password", readonly=False)

    def l10n_in_edi_ewaybill_test(self):
        self.l10n_in_check_gst_number()
        response = self.env["account.edi.format"]._l10n_in_edi_ewaybill_authenticate(self.company_id)
        if response.get("error") or not self.company_id.sudo()._l10n_in_edi_ewaybill_token_is_valid():
            error_message = _("Incorrect username or password, or the GST number on company does not match.")
            if response.get("error"):
                error_message = "\n".join([html_escape("[%s] %s" % (e.get("code"), e.get("message"))) for e in response["error"]])
            raise UserError(error_message)
        return {
              'type': 'ir.actions.client',
              'tag': 'display_notification',
              'params': {
                  'type': 'info',
                  'sticky': False,
                  'message': _("API credentials validated successfully"),
              }
          }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ewaybill_type
from . import res_company
from . import res_config_settings
from . import account_move
from . import account_edi_format

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
l10n_in_edi_ewaybill.access_l10n_in_ewaybill_type,access_l10n_in_ewaybill_type,l10n_in_edi_ewaybill.model_l10n_in_ewaybill_type,base.group_user,1,0,0,0

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="invoice_form_inherit_l10n_in_edi_ewaybill" model="ir.ui.view">
        <field name="name">account.move.form.inherit.l10n.in.ewaybill</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='button_set_checked']" position="after">
                <button name="l10n_in_edi_ewaybill_send" string="Send E-waybill" class="oe_highlight" type="object" groups="account.group_account_invoice" invisible="not l10n_in_edi_ewaybill_show_send_button" data-hotkey="e"/>
            </xpath>
            <xpath expr="//notebook/page[@name='other_info']" position="before">
                <page string="eWayBill" name="l10n_in_edi_ewaybill_page"
                    invisible="move_type in ('entry', 'out_refund') or country_code != 'IN'">
                    <group name="ewaybill_group">
                        <group string="Transaction Details" name="Transaction_group" 
                            invisible="not l10n_in_edi_ewaybill_direct_api">
                            <field name="l10n_in_type_id" domain="[('code', '!=', 'CHL'), ('allowed_supply_type', 'in', ('in', 'both'))] if move_type in ('in_invoice', 'out_refund', 'in_receipt') else [('code', '!=', 'CHL'), ('allowed_supply_type', 'in', ('out', 'both'))]"/>
                        </group>
                        <group string="Transportation Details" name="transportation_group">
                            <field name="l10n_in_mode" widget="radio" options="{'horizontal': True}"/>
                            <label for="l10n_in_distance"/>
                            <div class="o_row" name="l10n_in_distance">
                                <field name="l10n_in_distance" class="oe_inline"/>
                                <span>km</span>
                            </div>
                            <field name="l10n_in_vehicle_type" 
                                invisible="l10n_in_mode != '1'"
                                required="l10n_in_mode == '1'"
                                widget="radio"
                                options="{'horizontal': True}"/>
                            <field name="l10n_in_vehicle_no" 
                                invisible="l10n_in_mode != '1'"
                                required="l10n_in_mode == '1'"/>
                            <field name="l10n_in_transporter_id"
                                domain="[('vat','not in', ['', False]), ('country_id.code','=','IN')]"
                                invisible="l10n_in_mode != '0'"
                                required="l10n_in_mode == '0'"/>
                            <field name="l10n_in_transportation_doc_no" 
                                invisible="l10n_in_mode not in ('2', '3', '4')"
                                required="l10n_in_mode in ('2', '3', '4')"
                                string="Transporter Document Number"/>
                            <field name="l10n_in_transportation_doc_date" 
                                invisible="l10n_in_mode not in ('2', '3', '4')"
                                required="l10n_in_mode in ('2', '3', '4')"/>
                        </group>
                        <group string="Cancel Reason" invisible="country_code != 'IN' or state != 'posted' or not l10n_in_edi_show_cancel">
                            <field name="l10n_in_edi_cancel_reason"/>
                            <field name="l10n_in_edi_cancel_remarks"/>
                        </group>
                    </group>
                </page>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\edi_pdf_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="l10n_in_einvoice_report_invoice_document_inherit" inherit_id="account.report_invoice_document">
        <xpath expr="//div[@id='informations']" position="inside">
            <t t-set="l10n_in_ewaybill_json" t-value="o._get_l10n_in_edi_ewaybill_response_json()"/>
            <div id="l10n_in_ewaybill_informations" class="col" t-if="l10n_in_ewaybill_json" name="ewaybill_number">
                <strong>E-waybill:</strong>
                <p t-if="'EwbNo' in l10n_in_ewaybill_json" class="m-0" t-esc="l10n_in_ewaybill_json['EwbNo']"/>
                <p t-if="'ewayBillNo' in l10n_in_ewaybill_json" class="m-0" t-esc="l10n_in_ewaybill_json['ewayBillNo']"/>
                Until: <t t-if="'EwbValidTill' in l10n_in_ewaybill_json" class="m-0" t-esc="l10n_in_ewaybill_json['EwbValidTill']"/>
                <t t-if="'validUpto' in l10n_in_ewaybill_json" class="m-0" t-esc="l10n_in_ewaybill_json['validUpto']"/>
            </div>
        </xpath>
    </template>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form_inherit_l10n_in_edi_ewaybill" model="ir.ui.view">
        <field name="name">res.config.settings.form.inherit.l10n_in_edi_ewaybill</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//setting[@name='electronic_waybill_in']" position="inside">
                <div class="content-group" invisible="not module_l10n_in_edi_ewaybill or not module_l10n_in_edi">
                    <div class="mt16 row">
                        <label for="l10n_in_edi_ewaybill_username" string="Username" class="col-3 col-lg-3 o_light_label"/>
                        <field name="l10n_in_edi_ewaybill_username" nolabel="1"/>
                    </div>
                    <div class="row">
                        <label for="l10n_in_edi_ewaybill_password" string="Password" class="col-3 col-lg-3 o_light_label"/>
                        <field name="l10n_in_edi_ewaybill_password" password="True" nolabel="1"/>
                    </div>
                </div>
                <div class='mt8' invisible="not module_l10n_in_edi_ewaybill or not module_l10n_in_edi">
                    <button name="l10n_in_edi_ewaybill_test" icon="oi-arrow-right" type="object" string="Verify Username and Password" class="btn-link"/>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

