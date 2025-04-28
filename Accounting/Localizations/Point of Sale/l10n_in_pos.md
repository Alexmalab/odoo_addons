# Odoo Module: l10n_in_pos

Category: Accounting/Localizations/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Indian - Point of Sale',
    'version': '1.0',
    'description': """GST Point of Sale""",
    'category': 'Accounting/Localizations/Point of Sale',
    'depends': [
        'l10n_in',
        'point_of_sale'
    ],
    'data': [
        'views/pos_order_line_views.xml',
        'views/res_config_settings_views.xml',
        'data/pos_bill_data.xml',
    ],
    'demo': [
        'data/product_demo.xml',
    ],
    'auto_install': True,
    'assets': {
        'point_of_sale._assets_pos': [
            'l10n_in/static/src/helpers/hsn_summary.js',
            'l10n_in_pos/static/src/**/*',
        ],
        'web.assets_tests': [
            'l10n_in_pos/static/tests/tours/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\pos_bill_data.xml

```xml
<odoo>
    <data noupdate="1">
        <record model="pos.bill" id="500_00" forcecreate="0">
            <field name="name">500.00</field>
            <field name="value">500.00</field>
        </record>

        <record model="pos.bill" id="point_of_sale.0_05" forcecreate="0">
            <field name="for_all_config">False</field>
        </record>

        <record model="pos.bill" id="point_of_sale.0_10" forcecreate="0">
            <field name="for_all_config">False</field>
        </record>

        <record model="pos.bill" id="point_of_sale.0_20" forcecreate="0">
            <field name="for_all_config">False</field>
        </record>

        <record model="pos.bill" id="point_of_sale.0_25" forcecreate="0">
            <field name="for_all_config">False</field>
        </record>

        <record model="pos.bill" id="point_of_sale.0_50" forcecreate="0">
            <field name="for_all_config">False</field>
        </record>
    </data>
</odoo>

```

## File: data\product_demo.xml

```xml
<odoo>
    <data noupdate="1">
        <record id="product.desk_organizer" model="product.product">
            <field name="l10n_in_hsn_code">9403</field>
        </record>
        <record id="product.desk_pad" model="product.product">
            <field name="l10n_in_hsn_code">9403</field>
        </record>
        <record id="point_of_sale.led_lamp" model="product.product">
            <field name="l10n_in_hsn_code">85395000</field>
        </record>
        <record id="point_of_sale.letter_tray" model="product.product">
            <field name="l10n_in_hsn_code">48196000</field>
        </record>
        <record id="point_of_sale.magnetic_board" model="product.product">
            <field name="l10n_in_hsn_code">39219099</field>
        </record>
        <record id="product.monitor_stand" model="product.product">
            <field name="l10n_in_hsn_code">9403</field>
        </record>
        <record id="point_of_sale.newspaper_rack" model="product.product">
            <field name="l10n_in_hsn_code">94031090</field>
        </record>
        <record id="point_of_sale.small_shelf" model="product.product">
            <field name="l10n_in_hsn_code">94031090</field>
        </record>
        <record id="point_of_sale.product_product_tip" model="product.product">
            <field name="l10n_in_hsn_code">82090090</field>
        </record>
        <record id="point_of_sale.wall_shelf" model="product.product">
            <field name="l10n_in_hsn_code">94031090</field>
        </record>
        <record id="point_of_sale.whiteboard" model="product.product">
            <field name="l10n_in_hsn_code">39261099</field>
        </record>
        <record id="point_of_sale.whiteboard_pen" model="product.product">
            <field name="l10n_in_hsn_code">9608</field>
        </record>
    </data>
</odoo>

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    @api.depends('pos_session_ids')
    def _compute_l10n_in_state_id(self):
        res = super()._compute_l10n_in_state_id()
        to_compute = self.filtered(lambda m: m.country_code == 'IN' and not m.l10n_in_state_id and m.journal_id.type == 'general' and m.pos_session_ids)
        for move in to_compute:
            move.l10n_in_state_id = move.company_id.state_id
        return res

```

## File: models\account_tax.py

```python
from odoo import api, models


class AccountTax(models.Model):
    _inherit = 'account.tax'

    @api.model
    def _load_pos_data_fields(self, config_id):
        fields = super()._load_pos_data_fields(config_id)
        fields += ['l10n_in_tax_type']
        return fields

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class PosOrder(models.Model):
    _inherit = 'pos.order'

    def _prepare_invoice_vals(self):
        vals = super()._prepare_invoice_vals()
        if self.session_id.company_id.country_id.code == 'IN':
            partner = self.partner_id
            l10n_in_gst_treatment = partner.l10n_in_gst_treatment
            if not l10n_in_gst_treatment and partner.country_id and partner.country_id.code != 'IN':
                l10n_in_gst_treatment = 'overseas'
            if not l10n_in_gst_treatment:
                l10n_in_gst_treatment = partner.vat and 'regular' or 'consumer'
            vals['l10n_in_gst_treatment'] = l10n_in_gst_treatment
        return vals

```

## File: models\pos_order_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class PosOrderLine(models.Model):
    _inherit = "pos.order.line"

    l10n_in_hsn_code = fields.Char(string="HSN/SAC Code", compute="_compute_l10n_in_hsn_code", store=True, readonly=False, copy=False)

    @api.depends('product_id')
    def _compute_l10n_in_hsn_code(self):
        indian_lines = self.filtered(lambda line: line.company_id.account_fiscal_country_id.code == 'IN')
        (self - indian_lines).l10n_in_hsn_code = False
        for line in indian_lines:
            if line.product_id:
                line.l10n_in_hsn_code = line.product_id.l10n_in_hsn_code

    @api.model
    def _load_pos_data_fields(self, config_id):
        params = super()._load_pos_data_fields(config_id)
        if self.env.company.country_id.code == 'IN':
            params += ['l10n_in_hsn_code']
        return params

```

## File: models\product_product.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ProductProduct(models.Model):
    _inherit = "product.product"

    @api.model
    def _load_pos_data_fields(self, config_id):
        fields = super()._load_pos_data_fields(config_id)
        if self.env.company.country_id.code == 'IN':
            fields += ['l10n_in_hsn_code']
        return fields

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_order
from . import pos_order_line
from . import product_product
from . import account_move
from . import account_tax

```

## File: static\src\company_state_dialog\company_state_dialog.js

```javascript
/** @odoo-module */

import { Dialog } from "@web/core/dialog/dialog";
import { Component } from "@odoo/owl";
import { usePos } from "@point_of_sale/app/store/pos_hook";

export class companyStateDialog extends Component {
    static components = { Dialog };
    static template = "l10n_in_pos.companyStateDialog";
    static props = {
        close: Function,
    };

    setup() {
        this.pos = usePos();
    }

    redirect() {
        window.location = "/odoo/companies/" + this.pos.company.id;
    }

    onClose() {
        this.props.close();
    }
}

```

## File: static\src\company_state_dialog\company_state_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="l10n_in_pos.companyStateDialog">
        <Dialog size="'md'" title.translate="Company address is missing">
            <div>
                <p>Your company <span><t t-esc="this.pos.company.name"/></span> needs to have a correct address in order validate the invoice.</p>
                <p>Set the address of your company (Don't forget the State field)</p>
            </div>
            <t t-set-slot="footer">
                <div class="modal-footer-right d-flex gap-2">
                    <button class="button icon btn btn-lg btn-primary" t-on-click="redirect">
                            Go to company configuration
                    </button>
                    <button class="button btn btn-secondary" t-on-click="onClose">
                            Ok
                    </button>
                </div>
            </t>
        </Dialog>
    </t>

</templates>

```

## File: static\src\overrides\components\pos_receipt.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="l10n_in_pos.ReceiptHeader" t-inherit="point_of_sale.ReceiptHeader" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('pos-receipt-contact')]" position="after">
            <t t-if="props.data.partner and props.data.company?.country_id?.code == 'IN'">
                <div class="pos-receipt-center-align">
                    <div><t t-out="props.data.partner.name" /></div>
                    <t t-if="props.data.partner.phone">
                        <div>
                            <span>Phone: </span>
                            <t t-out="props.data.partner.phone" />
                        </div>
                    </t>
                    <br />
                </div>
            </t>
        </xpath>
    </t>

    <t t-name="l10n_in_pos.OrderReceipt" t-inherit="point_of_sale.OrderReceipt" t-inherit-mode="extension">
        <xpath expr="//Orderline" position="inside">
            <t t-if="line.l10n_in_hsn_code and props.data.headerData.company.country_id?.code === 'IN'">
                <div class="pos-receipt-left-padding">
                    <span>HSN Code: </span>
                    <t t-out="line.l10n_in_hsn_code"/>
                </div>
            </t>
        </xpath>
        <xpath expr="//div[@class='before-footer']" position="after">
            <br/>
            <t t-set="l10n_in_hsn_summary" t-value="props.data?.l10n_in_hsn_summary"/>
            <table class="l10n_in_hsn_summary_table"
                   t-if="l10n_in_hsn_summary and props.data.headerData.company.country_id?.code === 'IN' and l10n_in_hsn_summary.items.length > 0" style="width:100%;">
              <tr>
                    <th class="text-center fw-bolder" colspan="6">HSN Summary</th>
                </tr>
                <tr>
                    <th class="text-center">HSN Code</th>
                    <th class="text-center">Rate%</th>
                    <th class="text-center">CGST</th>
                    <th class="text-center">SGST</th>
                    <th class="text-center" t-if="l10n_in_hsn_summary.has_igst">IGST</th>
                    <th class="text-center" t-if="l10n_in_hsn_summary.has_cess">CESS</th>
                </tr>
                <tr t-foreach="l10n_in_hsn_summary.items" t-as="item" t-key="item_index">
                    <td class="text-center" t-out="item.l10n_in_hsn_code"/>
                    <td class="text-center"><t t-out="item.rate"/> %</td>
                    <td class="text-center" t-out="item.tax_amount_cgst"/>
                    <td class="text-center" t-out="item.tax_amount_sgst"/>
                    <td class="text-center" t-if="l10n_in_hsn_summary.has_igst" t-out="item.tax_amount_igst"/>
                    <td class="text-center" t-if="l10n_in_hsn_summary.has_cess" t-out="item.tax_amount_cess"/>
                </tr>
            </table>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\models\cash_move_popup.js

```javascript
/** @odoo-module */

import { CashMovePopup } from "@point_of_sale/app/navbar/cash_move_popup/cash_move_popup";

import { patch } from "@web/core/utils/patch";
import { companyStateDialog } from "@l10n_in_pos/company_state_dialog/company_state_dialog";

patch(CashMovePopup.prototype, {
    async confirm() {
        if (this.pos.company.country_id?.code === "IN" && !this.pos.company.state_id) {
            this.dialog.add(companyStateDialog);
            return;
        }
        return await super.confirm();
    },
});

```

## File: static\src\overrides\models\closing_session.js

```javascript
/** @odoo-module */

import { ClosePosPopup } from "@point_of_sale/app/navbar/closing_popup/closing_popup";

import { patch } from "@web/core/utils/patch";
import { companyStateDialog } from "@l10n_in_pos/company_state_dialog/company_state_dialog";

patch(ClosePosPopup.prototype, {
    async confirm() {
        if (this.pos.company.country_id?.code === "IN" && !this.pos.company.state_id) {
            this.dialog.add(companyStateDialog);
            return;
        }
        return await super.confirm();
    },
});

```

## File: static\src\overrides\models\invoice_button.js

```javascript
/** @odoo-module */

import { InvoiceButton } from "@point_of_sale/app/screens/ticket_screen/invoice_button/invoice_button";

import { patch } from "@web/core/utils/patch";
import { companyStateDialog } from "@l10n_in_pos/company_state_dialog/company_state_dialog";

patch(InvoiceButton.prototype, {
    click() {
        if (this.pos.company.country_id?.code === "IN" && !this.pos.company.state_id) {
            this.dialog.add(companyStateDialog);
            return;
        }
        return super.click();
    },
});

```

## File: static\src\overrides\models\payment_screen.js

```javascript
/** @odoo-module */

import { PaymentScreen } from "@point_of_sale/app/screens/payment_screen/payment_screen";

import { patch } from "@web/core/utils/patch";
import { companyStateDialog } from "@l10n_in_pos/company_state_dialog/company_state_dialog";

patch(PaymentScreen.prototype, {
    toggleIsToInvoice() {
        if (this.pos.company.country_id?.code === "IN" && !this.pos.company.state_id) {
            this.dialog.add(companyStateDialog);
            return;
        }
        return super.toggleIsToInvoice();
    },
});

```

## File: static\src\overrides\models\pos_order.js

```javascript
import { PosOrder } from "@point_of_sale/app/models/pos_order";
import { patch } from "@web/core/utils/patch";
import { accountTaxHelpers } from "@account/helpers/account_tax";
import { formatCurrency } from "@point_of_sale/app/models/utils/currency";
import { lt } from "@point_of_sale/utils";

patch(PosOrder.prototype, {
    export_for_printing(baseUrl, headerData) {
        const result = super.export_for_printing(...arguments);
        if (this.company.country_id?.code === "IN") {
            result.l10n_in_hsn_summary = this._prepareL10nInHsnSummary();
        }
        return result;
    },
    _prepareL10nInHsnSummary() {
        const currency = this.config.currency_id;
        const company = this.company;
        const orderLines = this.lines;

        // If each line is negative, we assume it's a refund order.
        // It's a normal order if it doesn't contain a line (useful for pos_settle_due).
        // TODO: Properly differentiate refund orders from normal ones.
        const documentSign =
            this.lines.length === 0 ||
            !this.lines.every((l) => lt(l.qty, 0, { decimals: currency.decimal_places }))
                ? 1
                : -1;

        const baseLines = orderLines.map((line) => {
            return accountTaxHelpers.prepare_base_line_for_taxes_computation(
                line,
                line.prepareBaseLineForTaxesComputationExtraValues({
                    quantity: documentSign * line.qty,
                })
            );
        });
        accountTaxHelpers.add_tax_details_in_base_lines(baseLines, company);
        accountTaxHelpers.round_base_lines_tax_details(baseLines, company);
        const hsnSummary = accountTaxHelpers.l10n_in_get_hsn_summary_table(baseLines, false);
        if (hsnSummary) {
            for (const item of hsnSummary.items) {
                for (const key of [
                    "tax_amount_igst",
                    "tax_amount_cgst",
                    "tax_amount_sgst",
                    "tax_amount_cess",
                ]) {
                    item[key] = formatCurrency(item[key], this.currency);
                }
            }
        }
        return hsnSummary;
    },
});

```

## File: static\src\overrides\models\pos_order_line.js

```javascript
import { PosOrderline } from "@point_of_sale/app/models/pos_order_line";
import { Orderline } from "@point_of_sale/app/generic_components/orderline/orderline";
import { patch } from "@web/core/utils/patch";

patch(PosOrderline.prototype, {
    setup(vals) {
        this.l10n_in_hsn_code = this.product_id.l10n_in_hsn_code || "";
        return super.setup(...arguments);
    },
    getDisplayData() {
        return {
            ...super.getDisplayData(),
            l10n_in_hsn_code: this.get_product().l10n_in_hsn_code || "",
        };
    },

    // EXTENDS 'point_of_sale'
    prepareBaseLineForTaxesComputationExtraValues(customValues = {}) {
        const extraValues = super.prepareBaseLineForTaxesComputationExtraValues(customValues);
        extraValues.l10n_in_hsn_code = this.product_id.l10n_in_hsn_code;
        return extraValues;
    },
});

patch(Orderline, {
    props: {
        ...Orderline.props,
        line: {
            ...Orderline.props.line,
            shape: {
                ...Orderline.props.line.shape,
                l10n_in_hsn_code: { type: String, optional: true },
            },
        },
    },
});

```

## File: static\src\overrides\store\pos_store.js

```javascript
/** @odoo-module */

import { PosStore } from "@point_of_sale/app/store/pos_store";
import { patch } from "@web/core/utils/patch";

patch(PosStore.prototype, {
    getReceiptHeaderData() {
        return {
            ...super.getReceiptHeaderData(...arguments),
            partner: this.selectedOrder.partner_id,
        };
    },
});

```

## File: views\pos_order_line_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="view_pos_pos_form_inherit" model="ir.ui.view">
        <field name="name">pos.order.form.inherit</field>
        <field name="model">pos.order</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_pos_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='lines']/list/field[@name='full_product_name']" position="after">
                <field name="l10n_in_hsn_code" optional="hide"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="res_config_settings_view_form_l10n_in_pos_inherit" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.l10n_in_pos.view</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="point_of_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='available_payment_terminal']" position="attributes">
                <attribute name="invisible">not is_kiosk_mode or country_code == 'IN'</attribute>
            </xpath>
            <xpath expr="//div[@id='available_payment_terminal']" position="after">
                <div class="o_notification_alert alert alert-warning" role="alert" invisible="not is_kiosk_mode or country_code != 'IN'">
                    <span>Please note that the kiosk for INR currency only works with Razorpay terminal</span>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

