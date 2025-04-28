# Odoo Module: l10n_es_pos

Category: Accounting/Localizations/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Spain - Point of Sale',
    'category': 'Accounting/Localizations/Point of Sale',
    'summary': """Spanish localization for Point of Sale""",
    'depends': ['point_of_sale', 'l10n_es'],
    'license': 'LGPL-3',
    'data': [
        'views/res_config_settings_views.xml',
        'views/pos_order_views.xml',
    ],
    'assets': {
        'point_of_sale._assets_pos': [
            'l10n_es_pos/static/src/**/*',
        ],
        'web.assets_tests': [
            'l10n_es_pos/static/tests/**/*',
        ],
    },
}

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _compute_l10n_es_is_simplified(self):
        super()._compute_l10n_es_is_simplified()
        for move in self:
            if move.pos_order_ids:
                move.l10n_es_is_simplified = move.pos_order_ids[0].is_l10n_es_simplified_invoice

```

## File: models\pos_config.py

```python
from odoo import api, fields, models


class PosConfig(models.Model):
    _inherit = "pos.config"

    is_spanish = fields.Boolean(string="Company located in Spain", compute="_compute_is_spanish")
    l10n_es_simplified_invoice_journal_id = fields.Many2one(
        comodel_name='account.journal',
        domain=[('type', '=', 'sale')],
        check_company=True,
    )
    simplified_partner_id = fields.Many2one(
        comodel_name="res.partner",
        string="Simplified invoice partner",
        compute="_compute_simplified_partner_id",
    )

    @api.depends("company_id")
    def _compute_is_spanish(self):
        for pos in self:
            pos.is_spanish = pos.company_id.country_code == "ES" and pos.l10n_es_simplified_invoice_journal_id

    def _compute_simplified_partner_id(self):
        for config in self:
            config.simplified_partner_id = self.env.ref("l10n_es.partner_simplified").id

    def get_limited_partners_loading(self):
        # this function normally returns 100 partners, but we have to make sure that
        # the simplified partner is also loaded
        res = super().get_limited_partners_loading()
        if (self.simplified_partner_id.id,) not in res:
            res.append((self.simplified_partner_id.id,))
        return res

```

## File: models\pos_order.py

```python
from odoo import api, fields, models

class PosOrder(models.Model):
    _inherit = "pos.order"

    is_l10n_es_simplified_invoice = fields.Boolean("Simplified invoice")
    l10n_es_simplified_invoice_number = fields.Char("Simplified invoice number", compute="_compute_l10n_es_simplified_invoice_number")

    @api.depends("account_move")
    def _compute_l10n_es_simplified_invoice_number(self):
        for order in self:
            if order.is_l10n_es_simplified_invoice:
                order.l10n_es_simplified_invoice_number = order.account_move.name
            else:
                order.l10n_es_simplified_invoice_number = False

    def _prepare_invoice_vals(self):
        res = super()._prepare_invoice_vals()
        if self.config_id.is_spanish and self.is_l10n_es_simplified_invoice:
            res["journal_id"] = self.config_id.l10n_es_simplified_invoice_journal_id.id
        return res

    def get_invoice_name(self):
        return self.account_move.name

```

## File: models\res_company.py

```python
from odoo import api, models


class ResCompany(models.Model):
    _inherit = "res.company"

    @api.model
    def _load_pos_data_fields(self, config_id):
        params = super()._load_pos_data_fields(config_id)
        if self.env.company.country_id.code == "ES":
            params += ["street", "city", "zip", "l10n_es_simplified_invoice_limit"]
        return params

```

## File: models\res_config_settings.py

```python
from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = "res.config.settings"

    pos_l10n_es_simplified_invoice_journal_id = fields.Many2one(
        related="pos_config_id.l10n_es_simplified_invoice_journal_id",
        readonly=False,
    )

```

## File: models\__init__.py

```python
from . import pos_config
from . import pos_order
from . import res_config_settings
from . import account_move
from . import res_company

```

## File: static\src\overrides\components\payment_screen\payment_screen.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { patch } from "@web/core/utils/patch";
import { AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { PaymentScreen } from "@point_of_sale/app/screens/payment_screen/payment_screen";

patch(PaymentScreen.prototype, {
    async validateOrder(isForceValidate) {
        if (this.pos.config.is_spanish && !this.skipAutomaticInvoicing()) {
            const order = this.currentOrder;
            order.is_l10n_es_simplified_invoice =
                order.canBeSimplifiedInvoiced() && !order.to_invoice;
            if (!order.is_l10n_es_simplified_invoice && !order.to_invoice) {
                this.env.services.dialog.add(AlertDialog, {
                    title: _t("Error"),
                    body: _t(
                        "Order amount is too large for a simplified invoice, use an invoice instead."
                    ),
                });
                return false;
            }
            if (order.is_l10n_es_simplified_invoice) {
                order.to_invoice = Boolean(
                    this.pos.config.raw.l10n_es_simplified_invoice_journal_id
                );
                if ((await this._askForCustomerIfRequired()) === false) {
                    return false;
                }
                if (!order.partner_id) {
                    order.set_partner(this.pos.config.simplified_partner_id);
                }
            }
        }
        return await super.validateOrder(...arguments);
    },
    skipAutomaticInvoicing() {
        const order = this.currentOrder;
        if (
            this.pos.config.is_spanish &&
            order.is_settling_account &&
            order.lines.length === 0 &&
            !order.to_invoice
        ) {
            return true;
        }
        return false;
    },
    shouldDownloadInvoice() {
        return this.pos.config.is_spanish
            ? !this.currentOrder.is_l10n_es_simplified_invoice
            : super.shouldDownloadInvoice();
    },
    async _postPushOrderResolve(order, order_server_ids) {
        if (this.pos.config.is_spanish) {
            const invoiceName = await this.pos.data.call("pos.order", "get_invoice_name", [
                order_server_ids,
            ]);
            order.invoice_name = invoiceName;
        }
        return super._postPushOrderResolve(...arguments);
    },
});

```

## File: static\src\overrides\components\receipt_header\receipt_header.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates id="template" xml:space="preserve">
    <t t-name="ReceiptHeader" t-inherit="point_of_sale.ReceiptHeader" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('pos-receipt-contact')]//div" position="before">
            <t t-set="isSimplifiedInvoice" t-value="props.data.is_l10n_es_simplified_invoice"/>
            <t t-if="props.data.is_spanish">
                <t t-if="isSimplifiedInvoice">
                    <div>Simplified invoice</div>
                    <div class="simplified-invoice-number" t-esc="props.data.invoice_name" />
                </t>
                <div t-if="props.data.company.street" t-esc="props.data.company.street" />
                <div t-if="props.data.company.zip" t-esc="props.data.company.zip" />
                <div t-if="props.data.company.city" t-esc="props.data.company.city" />
                <div t-if="props.data.company.state_id">(<t t-esc="props.data.company.state_id[1]"/>)</div>
            </t>
        </xpath>
        <xpath expr="//div[hasclass('pos-receipt-contact')]" position="inside">
            <t t-set="partner" t-value="props.data.partner"/>
            <t t-if="props.data.is_spanish and partner and partner.id !== props.data.simplified_partner_id">
                <div>Customer: <t t-esc="partner.name" /></div>
                <div t-if="partner.vat"><t t-esc="props.data.company.country_id?.vat_label || 'Tax ID'"/>: <t t-esc="partner.vat" /></div>
                <div t-if="partner.address" t-esc="partner.address"/>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\models\pos_order.js

```javascript
import { PosOrder } from "@point_of_sale/app/models/pos_order";
import { patch } from "@web/core/utils/patch";
import { roundCurrency } from "@point_of_sale/app/models/utils/currency";

patch(PosOrder.prototype, {
    canBeSimplifiedInvoiced() {
        return (
            this.config.is_spanish &&
            roundCurrency(this.get_total_with_tax(), this.currency) <
                this.company.l10n_es_simplified_invoice_limit
        );
    },
    wait_for_push_order() {
        return this.config.is_spanish ? true : super.wait_for_push_order(...arguments);
    },
});

```

## File: static\src\overrides\models\pos_store.js

```javascript
import { patch } from "@web/core/utils/patch";
import { PosStore } from "@point_of_sale/app/store/pos_store";

patch(PosStore.prototype, {
    getReceiptHeaderData(order) {
        const result = super.getReceiptHeaderData(...arguments);
        result.is_spanish = this.config.is_spanish;
        result.simplified_partner_id = this.config.simplified_partner_id.id;
        if (order) {
            result.is_l10n_es_simplified_invoice = order.is_l10n_es_simplified_invoice;
            result.partner = order.get_partner();
            result.invoice_name = order.invoice_name;
        }
        return result;
    },
    _getCreateOrderContext(orders, options) {
        let context = super._getCreateOrderContext(...arguments);
        if (this.config.is_spanish) {
            const noOrderRequiresInvoicePrinting = orders.every(
                (order) => !order.to_invoice && order.data.is_l10n_es_simplified_invoice
            );
            if (noOrderRequiresInvoicePrinting) {
                context = { ...context, generate_pdf: false };
            }
        }
        return context;
    },
});

```

## File: views\pos_order_views.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <!-- POS order form -->
    <record id="view_pos_pos_form_simplified_invoice" model="ir.ui.view">
        <field name="name">pos.order.form</field>
        <field name="model">pos.order</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_pos_form" />
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="l10n_es_simplified_invoice_number" readonly="1" invisible="not l10n_es_simplified_invoice_number" />
            </field>
        </field>
    </record>
    <!-- POS order tree -->
    <record id="view_pos_order_tree" model="ir.ui.view">
        <field name="model">pos.order</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_order_tree" />
        <field name="arch" type="xml">
            <field name="pos_reference" position="before">
                <field name="l10n_es_simplified_invoice_number"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="point_of_sale.res_config_settings_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//setting[@id='pos_default_journals']" position="inside">
                <div class="row" invisible="country_code != 'ES'">
                    <label string="Simplified Invoice" for="pos_l10n_es_simplified_invoice_journal_id" class="col-lg-3 o_light_label"/>
                    <field name="pos_l10n_es_simplified_invoice_journal_id" context="{'default_company_id': company_id, 'default_type': 'sale', 'default_code': 'SINV'}"/>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

