# Odoo Module: l10n_be_pos_sale

Category: Hidden

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
    'name': 'l10n_be_pos_sale',
    'version': '1.0',
    'category': 'Hidden',
    'sequence': 6,
    'summary': 'Link module between pos_sale and l10n_be',
    'depends': ['pos_sale', 'l10n_be'],
    'auto_install': True,
    'assets': {
        'point_of_sale._assets_pos': [
            'l10n_be_pos_sale/static/src/js/**/*',
        ],
        'web.assets_tests': [
            'l10n_be_pos_sale/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\pos_session.py

```python
from odoo import models


class PosSession(models.Model):
    _inherit = 'pos.session'

    def _load_pos_data(self, data):
        data = super()._load_pos_data(data)
        if self.env.company.country_code == 'BE':
            intracom_fpos = self.env["account.chart.template"].with_company(self.company_id.root_id).ref("fiscal_position_template_3", False)
            if intracom_fpos:
                data['data'][0]['_intracom_tax_ids'] = intracom_fpos.tax_ids.tax_dest_id.ids
        return data

```

## File: models\__init__.py

```python
from . import pos_session

```

## File: static\src\js\PaymentScreen.js

```javascript
/** @odoo-module **/

import { AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { PaymentScreen } from "@point_of_sale/app/screens/payment_screen/payment_screen";
import { patch } from "@web/core/utils/patch";
import { _t } from "@web/core/l10n/translation";
import { onMounted } from "@odoo/owl";

patch(PaymentScreen.prototype, {
    setup() {
        super.setup(...arguments);
        onMounted(() => {
            if (this.checkIsToInvoice()) {
                this.currentOrder.set_to_invoice(true);
            }
        });
    },
    toggleIsToInvoice() {
        if (this.checkIsToInvoice()) {
            this.dialog.add(AlertDialog, {
                title: _t("This order needs to be invoiced"),
                body: _t(
                    "If you do not invoice imported orders containing intra-community taxes you will encounter issues in your accounting. Especially in the EC Sales List report"
                ),
            });
        } else {
            super.toggleIsToInvoice(...arguments);
        }
    },
    checkIsToInvoice() {
        const orderLines = this.currentOrder.get_orderlines();
        const has_origin_order = orderLines.some((line) => line.sale_order_origin_id);
        const has_intracom_taxes = orderLines.some((line) =>
            line.tax_ids?.some((tax) => this.pos.session._intracom_tax_ids?.includes(tax.id))
        );
        if (
            this.pos.company.country_id &&
            this.pos.company.country_id.code === "BE" &&
            has_origin_order &&
            has_intracom_taxes
        ) {
            return true;
        }
    },
});

```

