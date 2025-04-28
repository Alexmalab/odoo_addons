# Odoo Module: l10n_id_pos

Category: Accounting/Localizations/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Indonesia - Point of Sale',
    'version': '1.0',
    'description': """Indonesian Point of Sale""",
    'category': 'Accounting/Localizations/Point of Sale',
    'depends': [
        'l10n_id',
        'point_of_sale'
    ],
    'auto_install': True,
    'assets': {
        'point_of_sale._assets_pos': [
            'l10n_id_pos/static/src/**/*',
        ],
        'web.assets_tests': [
            'l10n_id_pos/static/tests/**/*'
        ]
    },
    'license': 'LGPL-3',
}

```

## File: models\pos_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class PosOrder(models.Model):
    _inherit = "pos.order"

    # referenced in l10n_id/models/res_bank.py where we will link QRIS transactions
    # to the record that initiates the payment flow
    l10n_id_qris_transaction_ids = fields.Many2many('l10n_id.qris.transaction')

```

## File: models\pos_payment_method.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.exceptions import UserError
from odoo import _, models


class PosPaymentMethod(models.Model):
    _inherit = "pos.payment.method"

    def l10n_id_verify_qris_status(self, trx_uuid):
        """ Verify qris payment status from the provided transaction UUID

        For all qris_invoice_details linked to the transaction, check the payment status
        """
        if self.payment_method_type != 'qr_code' or self.qr_code_method != 'id_qr':
            return True
        trx = self.env['l10n_id.qris.transaction']._get_latest_transaction('pos.order', trx_uuid)
        if not trx:
            raise UserError(_("No QRIS transaction record is found based on this order"))

        result = trx._l10n_id_get_qris_qr_statuses()
        return result['paid']

```

## File: models\qris_transaction.py

```python
from odoo import models


class QRISTransaction(models.Model):
    _inherit = "l10n_id.qris.transaction"

    def _get_supported_models(self):
        return super()._get_supported_models() + ['pos.order']

    def _get_record(self):
        # Override
        # add it for pos.order
        if self.model == 'pos.order':
            return self.env[self.model].search([('uuid', '=', self.model_id)])
        return super()._get_record()

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import pos_payment_method
from . import qris_transaction
from . import pos_order

```

## File: static\src\js\pos_store.js

```javascript
import { PosStore } from "@point_of_sale/app/store/pos_store";
import { patch } from "@web/core/utils/patch";
import { user } from "@web/core/user";

patch(PosStore.prototype, {
    async showQR(payment) {
        // Add context to signal backend it's coming from PoS
        user.updateContext({ qris_model: "pos.order", qris_model_id: payment.pos_order_id.uuid });
        return await super.showQR(payment);
    },
});

```

## File: static\src\js\qr_code_popup.js

```javascript
import { QRPopup } from "@point_of_sale/app/utils/qr_code_popup/qr_code_popup";
import { patch } from "@web/core/utils/patch";
import { AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { useService } from "@web/core/utils/hooks";
import { _t } from "@web/core/l10n/translation";

patch(QRPopup.prototype, {
    setup() {
        super.setup(...arguments);
        this.orm = useService("orm");
    },

    async _confirm() {
        // Verify whether the payment has been recieved by QRIS

        this.setButtonsDisabled(true);

        const pm_line = this.props.line;
        let result;

        try {
            result = await this.orm.call("pos.payment.method", "l10n_id_verify_qris_status", [
                [pm_line.payment_method_id.id],
                pm_line.pos_order_id.uuid,
            ]);
        } catch {
            this.env.services.dialog.add(AlertDialog, {
                title: _t("Failure"),
                body: _t("Failure to verify QRIS payment status"),
            });
            this.setButtonsDisabled(false);
            return false;
        }

        if (!result) {
            this.env.services.dialog.add(AlertDialog, {
                title: _t("Payment Status Update"),
                body: _t("Payment Status returns unpaid"),
            });
            this.setButtonsDisabled(false);
            return false;
        }
        this.setButtonsDisabled(false);
        return super._confirm();
    },
});

```

