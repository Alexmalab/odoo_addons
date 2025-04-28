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

    def _pos_data_process(self, loaded_data):
        res = super()._pos_data_process(loaded_data)
        if self.company_id.country_code == 'BE':
            intracom_fpos = self.env["account.chart.template"].with_company(
                self.company_id).ref("fiscal_position_template_3", False)
            loaded_data['intracom_tax_ids'] = intracom_fpos.tax_ids.tax_dest_id.ids if intracom_fpos else []
        return res

```

## File: models\__init__.py

```python
from . import pos_session

```

## File: static\src\js\models.js

```javascript
/** @odoo-module **/

import { patch } from "@web/core/utils/patch";
import { Order } from "@point_of_sale/app/store/models";
import { PosStore } from "@point_of_sale/app/store/pos_store";

patch(Order.prototype, {
    async pay() {
        const orderLines = this.get_orderlines();
        const has_origin_order = orderLines.some((line) => line.sale_order_origin_id);
        const has_intracom_taxes = orderLines.some(
            (line) =>
                line.tax_ids &&
                this.pos.intracom_tax_ids &&
                line.tax_ids.some((tax) => this.pos.intracom_tax_ids.includes(tax))
        );
        if (
            this.pos.company.country &&
            this.pos.company.country.code === "BE" &&
            has_origin_order &&
            has_intracom_taxes
        ) {
            this.to_invoice = true;
        }
        return super.pay(...arguments);
    }
});

patch(PosStore.prototype, {
    async _processData(loadedData) {
        await super._processData(...arguments);
        if (this.company.country?.code == "BE") {
            this.intracom_tax_ids = loadedData["intracom_tax_ids"];
        }
    },
});

```

## File: static\src\js\PaymentScreen.js

```javascript
/** @odoo-module **/

import { ErrorPopup } from "@point_of_sale/app/errors/popups/error_popup";
import { PaymentScreen } from "@point_of_sale/app/screens/payment_screen/payment_screen";
import { patch } from "@web/core/utils/patch";
import { _t } from "@web/core/l10n/translation";

patch(PaymentScreen.prototype, {
    async toggleIsToInvoice() {
        const orderLines = this.currentOrder.get_orderlines();
        const has_origin_order = orderLines.some(line => line.sale_order_origin_id);
        const has_intracom_taxes = orderLines.some(line=>line.tax_ids?.some(tax=>this.pos.intracom_tax_ids?.includes(tax)));
        if(this.currentOrder.is_to_invoice() && this.pos.company.country?.code === "BE" && has_origin_order && has_intracom_taxes){
            this.popup.add(ErrorPopup, {
                title: _t('This order needs to be invoiced'),
                body: _t('If you do not invoice imported orders containing intra-community taxes you will encounter issues in your accounting. Especially in the EC Sales List report'),
            });
        }
        else{
            super.toggleIsToInvoice(...arguments);
        }
    }
});

```

