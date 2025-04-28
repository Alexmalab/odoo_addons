# Odoo Module: l10n_es_pos_tbai

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
    'name': "Spain - POS + TicketBAI",
    'version': '1.0',
    'category': 'Accounting/Localizations/Point of Sale',
    'depends': [
        'l10n_es_edi_tbai',
        'l10n_es_pos',
    ],
    'assets': {
        'point_of_sale._assets_pos': [
            'l10n_es_pos_tbai/static/src/**/*',
        ],
        'web.assets_tests': [
            'l10n_es_pos_tbai/static/tests/**/*',
        ],
    },
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    @api.model
    def get_refund_reason_list(self):
        return self.env['account.move']._fields['l10n_es_tbai_refund_reason']._description_selection(self.env)

```

## File: models\pos_order.py

```python
from odoo import models


class PosOrder(models.Model):
    _inherit = 'pos.order'

    def get_l10n_es_pos_tbai_qrurl(self):
        """ This function manually triggers the account.edi post CRON and synchronously
        wait for the process to finish, so that we can retrieve the generated QR code
        from the post response and transfer it to JS and eventually the Order Receipt XML. """
        self.ensure_one()
        if 'es_tbai' in self.account_move.edi_document_ids.edi_format_id.mapped('code'):
            tbai_documents_to_send = self.account_move.edi_document_ids.filtered(
                lambda d: d.edi_format_id.code == 'es_tbai' and d.state == 'to_send')
            tbai_documents_to_send._process_documents_web_services(job_count=1)
            return self.account_move._get_l10n_es_tbai_qr()

    def _generate_pos_order_invoice(self):
        # OVERRIDES 'point_of_sale'
        """ We need to make sure that the account.edi CRON does not run on TicketBai Invoices,
        because we plan to manually trigger it in our custom function above so that we can
        synchronously wait for the process to finish. """
        journal = self.config_id.l10n_es_simplified_invoice_journal_id \
            if self.is_l10n_es_simplified_invoice else self.config_id.invoice_journal_id

        if 'es_tbai' in journal.edi_format_ids.mapped('code'):
            return super(PosOrder, self.with_context(skip_account_edi_cron_trigger=True))._generate_pos_order_invoice()
        else:
            return super()._generate_pos_order_invoice()

    def _process_order(self, order, draft, existing_order):
        if'l10n_es_tbai_refund_reason' in order['data']:
            return super(PosOrder, self.with_context(l10n_es_tbai_refund_reason=order['data']['l10n_es_tbai_refund_reason']))._process_order(order, draft, existing_order)
        else:
            return super()._process_order(order, draft, existing_order)

    def _prepare_invoice_vals(self):
        res = super()._prepare_invoice_vals()
        if self.env.context.get('l10n_es_tbai_refund_reason'):
            res['l10n_es_tbai_refund_reason'] = self.env.context.get('l10n_es_tbai_refund_reason')
        return res

```

## File: models\__init__.py

```python
from . import pos_order
from . import account_move

```

## File: static\src\overrides\components\order_receipt\order_receipt.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="l10n_es_pos_tbai.OrderReceipt" t-inherit="point_of_sale.OrderReceipt" t-inherit-mode="extension">
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

import {patch} from "@web/core/utils/patch";
import {PaymentScreen} from "@point_of_sale/app/screens/payment_screen/payment_screen";
import {qrCodeSrc} from "@point_of_sale/utils";

patch(PaymentScreen.prototype, {
    async _postPushOrderResolve(order, order_server_ids) {
        if (this.pos.config.is_spanish) {
            const l10n_es_pos_tbai_qrurl = await this.orm.call(
                "pos.order",
                "get_l10n_es_pos_tbai_qrurl",
                [order_server_ids],
                {}
            );
            order.l10n_es_pos_tbai_qrsrc = l10n_es_pos_tbai_qrurl ? qrCodeSrc(l10n_es_pos_tbai_qrurl) : undefined;
        }
        return super._postPushOrderResolve(...arguments);
    },
});

```

## File: static\src\overrides\components\ticket_screen\ticket_screen.js

```javascript
/** @odoo-module */

import { patch } from "@web/core/utils/patch";
import { TicketScreen } from "@point_of_sale/app/screens/ticket_screen/ticket_screen";
import { SelectionPopup } from "@point_of_sale/app/utils/input_popups/selection_popup";
import { _t } from "@web/core/l10n/translation";

patch(TicketScreen.prototype, {
    //@override
    async addAdditionalRefundInfo(order, destinationOrder) {
        if (this.pos.config.is_spanish && order.state == "invoiced") {
            let selectionList = await this.orm.call("account.move", "get_refund_reason_list", []);
            selectionList = selectionList.map((el) => {
                return { 'id': el[0], 'label': el[1], 'item': el[0]}
            })
            const { confirmed, payload } = await this.popup.add(SelectionPopup, {
                title: _t("Select the refund reason"),
                list: selectionList,
            });
            if (payload && confirmed) {
                destinationOrder.l10n_es_tbai_refund_reason = payload;
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

import { Order } from "@point_of_sale/app/store/models";
import { patch } from "@web/core/utils/patch";

patch(Order.prototype, {
    //@override
    export_for_printing() {
        return {
            ...super.export_for_printing(...arguments),
            l10n_es_pos_tbai_qrsrc: this.l10n_es_pos_tbai_qrsrc,
        };
    },
    export_as_JSON() {
        const json = super.export_as_JSON(...arguments);
        if (this.pos.config.is_spanish) {
            json["l10n_es_tbai_refund_reason"] = this.l10n_es_tbai_refund_reason
        }
        return json;
    },
});

```

