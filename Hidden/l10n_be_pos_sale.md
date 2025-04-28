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
        'point_of_sale.assets': [
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
            intracom_fpos = self.env.ref(f"l10n_be.{self.env.company.id}_fiscal_position_template_3", False)
            if intracom_fpos:
                loaded_data['intracom_tax_ids'] = intracom_fpos.tax_ids.tax_dest_id.ids
        return res

```

## File: models\__init__.py

```python
from . import pos_session

```

## File: static\src\js\models.js

```javascript
odoo.define("l10n_be_pos_sale.models", function (require) {
    "use strict";

    var { PosGlobalState } = require("point_of_sale.models");
    const Registries = require("point_of_sale.Registries");

    const PoSSaleBeGlobalState = (PosGlobalState) =>
        class PoSSaleBeGlobalState extends PosGlobalState {
            async _processData(loadedData) {
                await super._processData(...arguments);
                if (this.company.country && this.company.country.code == "BE") {
                    this.intracom_tax_ids = loadedData["intracom_tax_ids"];
                }
            }
        };
    Registries.Model.extend(PosGlobalState, PoSSaleBeGlobalState);
});

```

## File: static\src\js\PaymentScreen.js

```javascript
/** @odoo-module **/

import PaymentScreen from 'point_of_sale.PaymentScreen';
import Registries from 'point_of_sale.Registries';

export const PoSSaleBePaymentScreen = (PaymentScreen) =>
    class extends PaymentScreen {
        toggleIsToInvoice() {
            const orderLines = this.currentOrder.get_orderlines();
            const has_origin_order = orderLines.some((line) => line.sale_order_origin_id);
            const has_intracom_taxes = orderLines.some((line) =>
                line.tax_ids && this.env.pos.intracom_tax_ids && line.tax_ids.some((tax) => this.env.pos.intracom_tax_ids.includes(tax))
            );
            if (
                this.currentOrder.is_to_invoice() &&
                this.env.pos.company.country.code === "BE" &&
                has_origin_order &&
                has_intracom_taxes
            ) {
                this.showPopup('ErrorPopup', {
                    title: this.env._t('This order needs to be invoiced'),
                    body: this.env._t('If you do not invoice imported orders containing intra-community taxes you will encounter issues in your accounting. Especially in the EC Sales List report'),
                });
            }
            else{
                super.toggleIsToInvoice();
            }
        }
    };

Registries.Component.extend(PaymentScreen, PoSSaleBePaymentScreen);

```

## File: static\src\js\ProductScreen.js

```javascript
/** @odoo-module **/

import ProductScreen from 'point_of_sale.ProductScreen';
import Registries from 'point_of_sale.Registries';

export const PoSSaleBeProductScreen = (ProductScreen) =>
    class extends ProductScreen {
        async _onClickPay() {
            const orderLines = this.currentOrder.get_orderlines();
            const has_origin_order = orderLines.some(line => line.sale_order_origin_id);
            const has_intracom_taxes = orderLines.some(
                (line) =>
                    line.tax_ids &&
                    this.env.pos.intracom_tax_ids &&
                    line.tax_ids.some((tax) => this.env.pos.intracom_tax_ids.includes(tax))
            );
            if (
                this.env.pos.company.country &&
                this.env.pos.company.country.code === "BE" &&
                has_origin_order &&
                has_intracom_taxes
            ) {
                this.currentOrder.to_invoice = true;
            }
            return super._onClickPay(...arguments);
        }
    };

Registries.Component.extend(ProductScreen, PoSSaleBeProductScreen);

```

