# Odoo Module: l10n_es_edi_tbai_pos

Category: Accounting/Localizations/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': "Spain - Point of Sale + TicketBAI",
    'version': '1.0',
    'category': 'Accounting/Localizations/Point of Sale',
    'depends': [
        'l10n_es_edi_tbai',
        'point_of_sale',
    ],
    'data': [
        'views/pos_order_view.xml',
    ],
    'assets': {
        'point_of_sale._assets_pos': [
            'l10n_es_edi_tbai_pos/static/src/**/*',
        ],
    },
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\pos_order.py

```python
from odoo import api, fields, models
from odoo.addons.l10n_es_edi_tbai.models.account_move import TBAI_REFUND_REASONS
from odoo.exceptions import UserError


class PosOrder(models.Model):
    _inherit = 'pos.order'

    l10n_es_tbai_state = fields.Selection([
            ('to_send', 'To Send'),
            ('sent', 'Sent'),
        ],
        string='TicketBAI status',
        compute='_compute_l10n_es_tbai_state',
    )
    l10n_es_tbai_chain_index = fields.Integer(
        string="TicketBAI chain index",
        help="Invoice index in chain, set if and only if an in-chain XML was submitted and did not error",
        related='l10n_es_tbai_post_document_id.chain_index',
    )

    l10n_es_tbai_post_document_id = fields.Many2one(
        comodel_name='l10n_es_edi_tbai.document',
        copy=False,
    )

    l10n_es_tbai_post_file = fields.Binary(
        string="TicketBAI Post File",
        related='l10n_es_tbai_post_document_id.xml_attachment_id.datas',
    )
    l10n_es_tbai_post_file_name = fields.Char(
        string="TicketBAI Post Attachment Name",
        related="l10n_es_tbai_post_document_id.xml_attachment_id.name",
    )

    l10n_es_tbai_is_required = fields.Boolean(
        string="TicketBAI required",
        related="company_id.l10n_es_tbai_is_enabled",
    )

    l10n_es_tbai_refund_reason = fields.Selection(
        selection=TBAI_REFUND_REASONS,
        string="Invoice Refund Reason Code (TicketBai)",
        help="BOE-A-1992-28740. Ley 37/1992, de 28 de diciembre, del Impuesto sobre el "
        "Valor Añadido. Artículo 80. Modificación de la base imponible.",
        copy=False,
    )

    # -------------------------------------------------------------------------
    # COMPUTE METHODS
    # -------------------------------------------------------------------------

    @api.depends('l10n_es_tbai_post_document_id.state')
    def _compute_l10n_es_tbai_state(self):
        for order in self:
            state = 'to_send' if order.l10n_es_tbai_is_required and not order.account_move else None
            if order.l10n_es_tbai_post_document_id and order.l10n_es_tbai_post_document_id.state == 'accepted':
                state = 'sent'

            order.l10n_es_tbai_state = state

    # -------------------------------------------------------------------------
    # OVERRIDES
    # -------------------------------------------------------------------------

    def _process_saved_order(self, draft):
        if not self.l10n_es_tbai_is_required:
            return super()._process_saved_order(draft)

        self.ensure_one()

        if not self.to_invoice and self.amount_total > self.company_id.l10n_es_simplified_invoice_limit:
            raise UserError(self.env._("Please create an invoice for an amount over %s.", self.company_id.l10n_es_simplified_invoice_limit))

        if self.refunded_order_id:
            if self.to_invoice and self.refunded_order_id.state != 'invoiced':
                raise UserError(self.env._("You cannot invoice a refund whose linked order hasn't been invoiced."))
            if not self.to_invoice and self.refunded_order_id.state == 'invoiced':
                raise UserError(self.env._("Please invoice the refund as the linked order has been invoiced."))

        return super()._process_saved_order(draft)

    def action_pos_order_paid(self):
        res = super().action_pos_order_paid()

        if self.l10n_es_tbai_is_required and not self.to_invoice:
            self._l10n_es_tbai_post()

        return res

    def _prepare_invoice_vals(self):
        vals = super()._prepare_invoice_vals()

        if self.l10n_es_tbai_is_required:
            vals['l10n_es_tbai_refund_reason'] = self.l10n_es_tbai_refund_reason

        return vals

    # -------------------------------------------------------------------------
    # PUBLIC METHODS
    # -------------------------------------------------------------------------

    def get_l10n_es_pos_tbai_qrurl(self):
        """ Retrieve the QR Code from the related ticketbai document . """
        self.ensure_one()

        edi_document = self.account_move.l10n_es_tbai_post_document_id or self.l10n_es_tbai_post_document_id
        if edi_document and edi_document.state == 'accepted':
            return edi_document._get_tbai_qr()

    # -------------------------------------------------------------------------
    # WEB SERVICE CALL
    # -------------------------------------------------------------------------

    def l10n_es_tbai_retry_post(self):
        error = self._l10n_es_tbai_post()
        if error:
            raise UserError(error)

    def _l10n_es_tbai_post(self):
        self.ensure_one()

        if self.l10n_es_tbai_post_document_id and self.l10n_es_tbai_post_document_id.state == 'rejected':
            self.l10n_es_tbai_post_document_id.sudo().unlink()

        if not self.l10n_es_tbai_post_document_id:
            self.l10n_es_tbai_post_document_id = self._l10n_es_tbai_create_edi_document()

        edi_document = self.l10n_es_tbai_post_document_id

        error = edi_document._post_to_web_service(self._l10n_es_tbai_get_values())
        if error:
            return error

        if edi_document.state == 'accepted':
            return

        # Return the error message if the xml document was not accepted
        return edi_document.response_message

    def _l10n_es_tbai_create_edi_document(self, cancel=False):
        return self.sudo().env['l10n_es_edi_tbai.document'].create({
            'name': self.name,
            'company_id': self.company_id.id,
            'is_cancel': False,
            'date': self.date_order,
        })

    # -------------------------------------------------------------------------
    # XML VALUES
    # -------------------------------------------------------------------------

    def _l10n_es_tbai_get_values(self):
        self.ensure_one()

        base_lines = self.lines._prepare_tax_base_line_values()
        for base_line in base_lines:
            base_line['name'] = base_line['record'].name
        self.env['l10n_es_edi_tbai.document']._add_base_lines_tax_amounts(base_lines, self.company_id)

        return {
            'is_sale': True,
            'partner': self.partner_id,
            'is_simplified': True,
            'delivery_date': None,
            **self._l10n_es_tbai_get_attachment_values(),
            **self._l10n_es_tbai_get_credit_note_values(),
            'invoice_origin': False,
            'taxes': self.lines.tax_ids,
            'rate': self.currency_rate,
            'base_lines': base_lines,
        }

    def _l10n_es_tbai_get_attachment_values(self):
        return {
            'attachment_name': self.name + '_post.xml',
            'res_model': 'pos.order',
            'res_id': self.id,
        }

    def _l10n_es_tbai_get_credit_note_values(self):
        return {
            'is_refund': bool(self.refunded_order_id),
            'refund_reason': 'R5',
            'refunded_doc': self.refunded_order_id.l10n_es_tbai_post_document_id,
            'refunded_doc_invoice_date': self.refunded_order_id.date_order if self.refunded_order_id else False,
        }

```

## File: models\pos_session.py

```python
from odoo import models


class PosSession(models.Model):
    _inherit = 'pos.session'

    def _load_pos_data(self, models_to_load):
        data = super()._load_pos_data(models_to_load)

        tbai_refund_reason_field = self.env['ir.model.fields']._get('account.move', 'l10n_es_tbai_refund_reason')
        data['data'][0]['_tbai_refund_reasons'] = [
            {'value': refund_reason.value, 'name': refund_reason.name}
            for refund_reason in tbai_refund_reason_field.selection_ids
            if refund_reason.value != 'R5'  # R5 is for simplified invoice
        ]

        return data

```

## File: models\res_company.py

```python
from odoo import api, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    @api.model
    def _load_pos_data_fields(self, config_id):
        return super()._load_pos_data_fields(config_id) + ['l10n_es_tbai_is_enabled']

```

## File: models\__init__.py

```python
from . import pos_order
from . import pos_session
from . import res_company

```

## File: static\src\app\add_tbai_refund_reason_popup\add_tbai_refund_reason_popup.js

```javascript
import { Dialog } from "@web/core/dialog/dialog";
import { usePos } from "@point_of_sale/app/store/pos_hook";
import { Component, useState } from "@odoo/owl";

export class AddTbaiRefundReasonPopup extends Component {
    static template = "l10n_es_edi_tbai_pos.AddTbaiRefundReasonPopup";
    static components = { Dialog };

    setup() {
        this.pos = usePos();
        this.state = useState({
            l10n_es_tbai_refund_reason: this.props.order.l10n_es_tbai_refund_reason || "R1",
        });
    }
    confirm() {
        this.props.getPayload(this.state);
        this.props.close();
    }
}

```

## File: static\src\app\add_tbai_refund_reason_popup\add_tbai_refund_reason_popup.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="l10n_es_edi_tbai_pos.AddTbaiRefundReasonPopup">
        <Dialog title.translate="Additional Refund Information">
            <div class="mb-3">
                <label for="tbai_refund_reason" class="form-label">TicketBAI Refund Reason: </label>
                <select class="detail form-select" id="tbai_refund_reason" name="l10n_es_tbai_refund_reason" t-model="state.l10n_es_tbai_refund_reason">
                    <t t-foreach="pos.session._tbai_refund_reasons" t-as="l10n_es_tbai_refund_reason" t-key="l10n_es_tbai_refund_reason.value">
                        <option t-att-value="l10n_es_tbai_refund_reason.value"
                                t-att-selected="l10n_es_tbai_refund_reason.value === state.l10n_es_tbai_refund_reason ? 'selected' : undefined">
                            <t t-out="l10n_es_tbai_refund_reason.name"/>
                        </option>
                    </t>
                </select>
            </div>
            <t t-set-slot="footer">
                <button class="btn btn-primary o-default-button" t-on-click="confirm">Ok</button>
            </t>
        </Dialog>
    </t>

</templates>

```

## File: static\src\overrides\components\order_receipt\order_receipt.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="l10n_es_edi_tbai_pos.OrderReceipt" t-inherit="point_of_sale.OrderReceipt" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('before-footer')]" position="after">
            <t t-if="props.data.l10n_es_pos_tbai_qrsrc">
                <br/><br/>
                <div class="pos-receipt-order-data mb-2">TicketBai QR Code</div>
                <img t-att-src="props.data.l10n_es_pos_tbai_qrsrc" class="pos-receipt-qrcode"/>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\components\payment_screen\payment_screen.js

```javascript
/** @odoo-module */

import { patch } from "@web/core/utils/patch";
import { PaymentScreen } from "@point_of_sale/app/screens/payment_screen/payment_screen";
import { qrCodeSrc } from "@point_of_sale/utils";

patch(PaymentScreen.prototype, {
    async _postPushOrderResolve(order, order_server_ids) {
        if (this.pos.company.l10n_es_tbai_is_enabled) {
            const l10n_es_pos_tbai_qrurl = await this.pos.data.call(
                "pos.order",
                "get_l10n_es_pos_tbai_qrurl",
                [order.id]
            );
            order.l10n_es_pos_tbai_qrsrc = l10n_es_pos_tbai_qrurl
                ? qrCodeSrc(l10n_es_pos_tbai_qrurl)
                : undefined;
        }
        return super._postPushOrderResolve(...arguments);
    },
});

```

## File: static\src\overrides\components\ticket_screen\ticket_screen.js

```javascript
import { patch } from "@web/core/utils/patch";
import { TicketScreen } from "@point_of_sale/app/screens/ticket_screen/ticket_screen";
import { makeAwaitable } from "@point_of_sale/app/store/make_awaitable_dialog";
import { AddTbaiRefundReasonPopup } from "@l10n_es_edi_tbai_pos/app/add_tbai_refund_reason_popup/add_tbai_refund_reason_popup";

patch(TicketScreen.prototype, {
    async addAdditionalRefundInfo(order, destinationOrder) {
        if (this.pos.company.l10n_es_tbai_is_enabled && order.state == "invoiced") {
            const payload = await makeAwaitable(this.dialog, AddTbaiRefundReasonPopup, {
                order: destinationOrder,
            });
            if (payload) {
                destinationOrder.l10n_es_tbai_refund_reason = payload.l10n_es_tbai_refund_reason;
                destinationOrder.to_invoice = true;
            }
        }
        super.addAdditionalRefundInfo(...arguments);
    },
});

```

## File: static\src\overrides\models\order.js

```javascript
/** @odoo-module */

import { PosOrder } from "@point_of_sale/app/models/pos_order";
import { patch } from "@web/core/utils/patch";

patch(PosOrder.prototype, {
    //@override
    export_for_printing(baseUrl, headerData) {
        const result = super.export_for_printing(...arguments);
        result.l10n_es_pos_tbai_qrsrc = this.l10n_es_pos_tbai_qrsrc;
        return result;
    },
    wait_for_push_order() {
        return this.company.l10n_es_tbai_is_enabled
            ? true
            : super.wait_for_push_order(...arguments);
    },
});

```

## File: views\pos_order_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_pos_order_form_inherit_l10n_es_pos_tbai" model="ir.ui.view">
            <field name="name">pos.order.form.inherit.l10n_es_edi_tbai_pos</field>
            <field name="model">pos.order</field>
            <field name="inherit_id" ref="point_of_sale.view_pos_pos_form"/>
            <field name="arch" type="xml">
                <xpath expr="//button[@name='action_pos_order_invoice']" position="attributes">
                    <field name="l10n_es_tbai_post_document_id" invisible="1"/>
                    <attribute name="invisible" add="l10n_es_tbai_post_document_id" separator="or"></attribute>
                </xpath>
                <xpath expr="//header" position="inside">
                    <button name="l10n_es_tbai_retry_post" string="Send to TicketBAI" type="object" invisible="l10n_es_tbai_state != 'to_send'"/>
                </xpath>
                <xpath expr="//notebook" position="inside">
                    <page
                        id="ticketbai_tab"
                        string="TicketBAI"
                        invisible="not l10n_es_tbai_is_required or not l10n_es_tbai_post_document_id"
                    >
                        <group>
                            <field name="l10n_es_tbai_state"/>
                            <field name="l10n_es_tbai_chain_index" groups="base.group_no_one"/>
                            <field name="l10n_es_tbai_post_file_name" invisible="1"/>
                            <field name="l10n_es_tbai_post_file" widget="binary" filename="l10n_es_tbai_post_file_name"/>
                        </group>
                    </page>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

