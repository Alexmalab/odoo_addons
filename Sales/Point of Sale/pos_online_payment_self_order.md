# Odoo Module: pos_online_payment_self_order

Category: Sales/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'POS Self-Order / Online Payment',
    'category': 'Sales/Point of Sale',
    'summary': 'Support online payment in self-order',
    'version': '1.0',
    'depends': ['pos_online_payment', 'pos_self_order'],
    'data': [
        'views/res_config_settings_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'assets': {
        'pos_self_order.assets': [
            'pos_online_payment_self_order/static/src/**/*',
            'web/static/lib/zxing-library/zxing-library.js',
        ],
        'web.assets_tests': [
            'pos_online_payment_self_order/static/tests/tours/pos_online_payment_multi_table_order.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\payment_portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request
from odoo.addons.pos_online_payment.controllers.payment_portal import PaymentPortal

class PaymentPortalSelfOrder(PaymentPortal):
    @http.route()
    def pos_order_pay(self, pos_order_id, access_token=None, exit_route=None):
        self._send_notification_payment_status(pos_order_id, 'progress')
        return super().pos_order_pay(pos_order_id, access_token=access_token, exit_route=exit_route)

    @http.route()
    def pos_order_pay_confirmation(self, pos_order_id, tx_id=None, access_token=None, exit_route=None, **kwargs):
        result = super().pos_order_pay_confirmation(pos_order_id, tx_id=tx_id, access_token=access_token, exit_route=exit_route, **kwargs)
        tx_sudo = request.env['payment.transaction'].sudo().search([('id', '=', tx_id)])

        if tx_sudo.state not in ('authorized', 'done'):
            self._send_notification_payment_status(pos_order_id, 'fail')
        else:
            self._send_notification_payment_status(pos_order_id, 'success')

        return result

    def _send_notification_payment_status(self, pos_order_id, status):
        pos_order = request.env['pos.order'].sudo().browse(pos_order_id)
        pos_order.config_id._notify("ONLINE_PAYMENT_STATUS", {
            'status': status, # progress, success, fail
            'data': {
                'pos.order': pos_order.read(pos_order._load_pos_self_data_fields(pos_order.config_id.id), load=False)
            }
        })

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import payment_portal

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from typing import Dict
from odoo import models, fields, api, _
from odoo.exceptions import ValidationError


class PosConfig(models.Model):
    _inherit = 'pos.config'

    self_order_online_payment_method_id = fields.Many2one('pos.payment.method', string='Self Online Payment', help="The online payment method to use when a customer pays a self-order online.", domain=[('is_online_payment', '=', True)], store=True, readonly=False)

    @api.constrains('self_order_online_payment_method_id')
    def _check_self_order_online_payment_method_id(self):
        for config in self:
            if config.self_ordering_mode == 'mobile' and config.self_ordering_service_mode == 'each' and config.self_order_online_payment_method_id and not config.self_order_online_payment_method_id._get_online_payment_providers(config.id, error_if_invalid=True):
                raise ValidationError(_("The online payment method used for self-order in a POS config must have at least one published payment provider supporting the currency of that POS config."))

    def _get_self_ordering_data(self):
        res = super()._get_self_ordering_data()
        payment_methods = self._get_self_ordering_payment_methods_data(self.self_order_online_payment_method_id)
        res['pos_payment_methods'] += payment_methods
        return res

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, tools
from odoo.osv import expression


class PosOrder(models.Model):
    _inherit = 'pos.order'

    use_self_order_online_payment = fields.Boolean(compute='_compute_use_self_order_online_payment', store=True, readonly=True)

    @api.depends('config_id.self_order_online_payment_method_id')
    def _compute_use_self_order_online_payment(self):
        for order in self:
            order.use_self_order_online_payment = bool(order.config_id.self_order_online_payment_method_id)

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if 'use_self_order_online_payment' not in vals or vals['use_self_order_online_payment']:
                session = self.env['pos.session'].browse(vals['session_id'])
                config = session.config_id
                vals['use_self_order_online_payment'] = bool(config.self_order_online_payment_method_id)
        return super().create(vals_list)

    def write(self, vals):
        # Because use_self_order_online_payment is not intended to be changed manually,
        # avoid to raise an error.
        if 'use_self_order_online_payment' not in vals:
            return super().write(vals)

        can_change_self_order_domain = [('state', '=', 'draft')]
        if vals['use_self_order_online_payment']:
            can_change_self_order_domain = expression.AND([can_change_self_order_domain, [('config_id.self_order_online_payment_method_id', '!=', False)]])

        can_change_self_order_orders = self.filtered_domain(can_change_self_order_domain)
        cannot_change_self_order_orders = self - can_change_self_order_orders

        res = True
        if can_change_self_order_orders:
            res = super(PosOrder, can_change_self_order_orders).write(vals) and res
        if cannot_change_self_order_orders:
            clean_vals = vals.copy()
            clean_vals.pop('use_self_order_online_payment', None)
            res = super(PosOrder, cannot_change_self_order_orders).write(clean_vals) and res

        return res

    @api.depends('use_self_order_online_payment', 'config_id.self_order_online_payment_method_id', 'config_id.payment_method_ids')
    def _compute_online_payment_method_id(self):
        for order in self:
            if order.use_self_order_online_payment:
                # It is expected to use the self order online payment method.
                # If for any reason it is not defined, then the online payment
                # of the order is set to null to make the problem noticeable.
                order.online_payment_method_id = order.config_id.self_order_online_payment_method_id
            else:
                super(PosOrder, order)._compute_online_payment_method_id()

    def get_and_set_online_payments_data(self, next_online_payment_amount=False):
        res = super().get_and_set_online_payments_data(next_online_payment_amount)
        if 'paid_order' not in res and not res.get('deleted', False) and not isinstance(next_online_payment_amount, bool):
            # This method is only called in the POS frontend flow, not self order.
            # If the next online payment is 0, then the online payment of the frontend
            # flow is cancelled, and the default flow is self order if it is configured.
            self.use_self_order_online_payment = tools.float_is_zero(next_online_payment_amount, precision_rounding=self.currency_id.rounding) and self.config_id.self_order_online_payment_method_id
        return res

```

## File: models\pos_payment_method.py

```python
from odoo import models, api
from odoo.osv import expression


class PosPaymentMethod(models.Model):
    _inherit = "pos.payment.method"

    @api.model
    def _load_pos_self_data_domain(self, data):
        if data['pos.config']['data'][0]['self_ordering_mode'] == 'kiosk':
            domain = super()._load_pos_self_data_domain(data)
            domain = expression.OR([[('is_online_payment', '=', True)], domain])
            return domain
        else:
            return [('is_online_payment', '=', True)]

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    pos_self_order_online_payment_method_id = fields.Many2one(related='pos_config_id.self_order_online_payment_method_id', readonly=False)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_config
from . import pos_order
from . import res_config_settings
from . import pos_payment_method

```

## File: static\src\self_order_service.js

```javascript
import { patch } from "@web/core/utils/patch";
import { SelfOrder } from "@pos_self_order/app/self_order_service";
import { session } from "@web/session";

patch(SelfOrder.prototype, {
    async setup(...args) {
        await super.setup(...args);
        this.onlinePaymentStatus = null;
        this.data.connectWebSocket("ONLINE_PAYMENT_STATUS", ({ status, data }) => {
            this.models.loadData(data, [], false);
            this.onlinePaymentStatus = status;
            this.paymentError = status === "fail";

            const order = this.models["pos.order"].find(
                (o) => o.access_token === data["pos.order"][0].access_token
            );
            if (status === "success" && !this.currentOrder.access_token && order) {
                this.confirmationPage("order", this.config.self_ordering_mode, order.access_token);
            }
        });
    },
    getOnlinePaymentUrl(
        { id: order_id, access_token: order_access_token, config_id: order_pos_config_id },
        exitRoute = true
    ) {
        const baseUrl = session.base_url;
        const order = this.currentOrder;
        let exitRouteUrl = baseUrl;

        if (exitRoute) {
            let table = "";
            exitRouteUrl += `/pos-self/${order_pos_config_id.id}`;

            if (this.config.self_ordering_pay_after === "each") {
                exitRouteUrl += `/confirmation/${order.access_token}/order`;
            }

            if (this.currentTable) {
                table = `&table_identifier=${this.currentTable.identifier}`;
            }

            exitRouteUrl += `?access_token=${this.access_token}${table}`;
        }

        const exit = encodeURIComponent(exitRouteUrl);
        return `${baseUrl}/pos/pay/${order_id}?access_token=${order_access_token}&exit_route=${exit}`;
    },
    filterPaymentMethods(pms) {
        const pm = super.filterPaymentMethods(...arguments);
        const online_pms = pms.filter(
            (rec) =>
                rec.is_online_payment &&
                (this.config.self_order_online_payment_method_id?.id === rec.id ||
                    (this.config.self_ordering_mode === "kiosk" &&
                        this.config.payment_method_ids.includes(rec.id)))
        );
        return [...new Set([...pm, ...online_pms])];
    },
});

```

## File: static\src\components\order_widget\order_widget.js

```javascript
import { patch } from "@web/core/utils/patch";
import { _t } from "@web/core/l10n/translation";
import { OrderWidget } from "@pos_self_order/app/components/order_widget/order_widget";

patch(OrderWidget.prototype, {
    get buttonToShow() {
        const buttonName = this.router.activeSlot === "product_list" ? _t("Order") : _t("Pay");
        const type = this.selfOrder.config.self_ordering_mode;
        const mode = this.selfOrder.config.self_ordering_pay_after;
        const isOnlinePayment = this.selfOrder.models["pos.payment.method"].find(
            (p) => p.is_online_payment
        );
        const order = this.selfOrder.currentOrder;
        const takeAway = order.takeaway;
        const service = this.selfOrder.config.self_ordering_service_mode;
        const isNoLine = order.lines.length === 0;

        if (type === "kiosk") {
            return super.buttonToShow;
        }

        if (order.amount_total === 0 && !isNoLine) {
            return { label: _t("Order"), disabled: false };
        }

        if (mode === "each") {
            return { label: buttonName, disabled: isNoLine };
        } else if (mode === "meal") {
            const order = this.selfOrder.currentOrder;

            if (!order) {
                return { label: "", disabled: true };
            }

            if (Object.keys(order.changes).length > 0) {
                return { label: _t("Order"), disabled: false };
            } else {
                if (isOnlinePayment) {
                    return { label: buttonName, disabled: false };
                } else {
                    if (this.router.activeSlot === "product_list") {
                        return {
                            label: _t("Order"),
                            disabled: true,
                        };
                    } else {
                        const label =
                            takeAway || service === "counter" ? _t("Pay at cashier") : _t("Pay");
                        return {
                            label: label,
                            disabled: false,
                        };
                    }
                }
            }
        } else {
            return super.buttonToShow;
        }
    },
});

```

## File: static\src\pages\payment_page\payment_page.js

```javascript
import { patch } from "@web/core/utils/patch";
import { PaymentPage } from "@pos_self_order/app/pages/payment_page/payment_page";
import { _t } from "@web/core/l10n/translation";

patch(PaymentPage.prototype, {
    async startPayment() {
        let order = this.selfOrder.currentOrder;
        const pm = this.selectedPaymentMethod;
        const device = this.selfOrder.config.self_ordering_mode;

        if (!pm || !pm.is_online_payment) {
            return super.startPayment(...arguments);
        } else {
            order = await this.selfOrder.sendDraftOrderToServer();
        }

        if (device === "kiosk") {
            const url = this.selfOrder.getOnlinePaymentUrl(order, false);
            this.generateQrcodeImg(url);
        } else {
            this.checkAndOpenPaymentPage(order);
        }
    },
    get selectedPaymentIsOnline() {
        const paymentMethods = this.selectedPaymentMethod;
        return paymentMethods && paymentMethods.is_online_payment;
    },
    generateQrcodeImg(url) {
        const codeWriter = new window.ZXing.BrowserQRCodeSvgWriter();
        const qr_code_svg = new XMLSerializer().serializeToString(codeWriter.write(url, 150, 150));
        this.state.qrImage = "data:image/svg+xml;base64," + window.btoa(qr_code_svg);
    },
    async checkAndOpenPaymentPage(order) {
        if (order.state === "draft") {
            const onlinePaymentUrl = this.selfOrder.getOnlinePaymentUrl(order, true);
            window.open(onlinePaymentUrl, "_self");
        } else {
            this.selfOrder.notification.add(
                _t("The current order cannot be paid (maybe it is already paid)."),
                { type: "danger" }
            );
        }
    },
});

```

## File: static\src\pages\payment_page\payment_page.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_online_payment_self_order.PaymentPage" t-inherit="pos_self_order.PaymentPage" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('payment-state-container')]" position="after">
            <div t-if="this.selectedPaymentIsOnline and !state.selection and selfOrder.config.self_ordering_mode === 'kiosk'" class="d-flex justify-content-center align-items-center flex-column h-100 px-3 text-center">
                <h1 class="mb-4">Scan the QR code to pay</h1>
                <div class="d-inline-flex flex-column border rounded p-4 bg-view mb-3">
                    <img t-att-src="state.qrImage" />
                </div>
                <h3 t-if="selfOrder.onlinePaymentStatus === 'progress'">Payment in progress</h3>
            </div>
        </xpath>
        <xpath expr="//div[hasclass('payment-state-container')]" position="attributes">
            <attribute name="t-if">!this.selectedPaymentIsOnline and !state.selection</attribute>
        </xpath>
    </t>
</templates>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="res_config_settings_view_form_menu" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.pos_online_payment.view</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="point_of_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='self-payment-after']" position="after">
                <div class="content-group row" invisible="not pos_self_ordering_mode == 'mobile'">
                    <label string="Online Payment" for="pos_self_order_online_payment_method_id" class="col-lg-4" />
                    <field name="pos_self_order_online_payment_method_id" placeholder="Pay at cashier if empty" />
                </div>
                <div class="d-flex flex-column align-items-start w-50" invisible="not pos_self_ordering_mode == 'mobile' and not pos_self_ordering_pay_after == 'each'">
                    <button name="%(point_of_sale.action_payment_methods_tree)d" icon="oi-arrow-right" type="action" string="Payment Methods" class="btn-link" />
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

