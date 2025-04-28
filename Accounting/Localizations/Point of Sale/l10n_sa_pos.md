# Odoo Module: l10n_sa_pos

Category: Accounting/Localizations/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Saudi Arabia - Point of Sale',
    'author': 'Odoo S.A',
    'category': 'Accounting/Localizations/Point of Sale',
    'description': """
K.S.A. POS Localization
=======================================================
    """,
    'license': 'LGPL-3',
    'depends': ['l10n_gcc_pos', 'l10n_sa_invoice'],
    'data': [
    ],
    'assets': {
        'web.assets_qweb': [
            'l10n_sa_pos/static/src/xml/OrderReceipt.xml',
        ],
        'point_of_sale.assets': [
            'web/static/lib/zxing-library/zxing-library.js',
            'l10n_sa_pos/static/src/js/models.js',
        ]
    },
    'auto_install': True,
}

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.exceptions import UserError
from odoo.tools.translate import _


class pos_config(models.Model):
    _inherit = 'pos.config'

    def open_ui(self):
        for config in self:
            if not config.company_id.country_id:
                raise UserError(_("You have to set a country in your company setting."))
        return super(pos_config, self).open_ui()

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import base64

from odoo import fields, api, models


class POSOrder(models.Model):
    _inherit = 'pos.order'

    def _prepare_invoice_vals(self):
        vals = super()._prepare_invoice_vals()
        if self.company_id.country_id.code == 'SA':
            vals.update({'l10n_sa_confirmation_datetime': self.date_order})
        return vals

```

## File: models\__init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_order
from . import pos_config

```

## File: static\src\js\models.js

```javascript
odoo.define('l10n_sa_pos.pos', function (require) {
"use strict";

const { Gui } = require('point_of_sale.Gui');
var models = require('point_of_sale.models');
var rpc = require('web.rpc');
var session = require('web.session');
var core = require('web.core');
var utils = require('web.utils');

var _t = core._t;
var round_di = utils.round_decimals;


var _super_order = models.Order.prototype;
models.Order = models.Order.extend({
    export_for_printing: function() {
      var result = _super_order.export_for_printing.apply(this,arguments);
      if (this.pos.company.country && this.pos.company.country.code === 'SA') {
          result.is_settlement = this.is_settlement();
          if (!result.is_settlement) {
              const codeWriter = new window.ZXing.BrowserQRCodeSvgWriter()
              let qr_values = this.compute_sa_qr_code(result.company.name, result.company.vat, result.date.isostring, result.total_with_tax, result.total_tax);
              let qr_code_svg = new XMLSerializer().serializeToString(codeWriter.write(qr_values, 150, 150));
              result.qr_code = "data:image/svg+xml;base64," + window.btoa(qr_code_svg);
          }
      }
      return result;
    },
    /**
     * If the order is empty (there are no products)
     * and all "pay_later" payments are negative,
     * we are settling a customer's account.
     * If the module pos_settle_due is not installed,
     * the function always returns false (since "pay_later" doesn't exist)
     * @returns {boolean} true if the current order is a settlement, else false
     */
    is_settlement: function() {
        return this.is_empty() &&
            !!this.paymentlines.filter(paymentline => paymentline.payment_method.type === "pay_later" && paymentline.amount < 0).length;
    },
    compute_sa_qr_code(name, vat, date_isostring, amount_total, amount_tax) {
        /* Generate the qr code for Saudi e-invoicing. Specs are available at the following link at page 23
        https://zatca.gov.sa/ar/E-Invoicing/SystemsDevelopers/Documents/20210528_ZATCA_Electronic_Invoice_Security_Features_Implementation_Standards_vShared.pdf
        */
        const seller_name_enc = this._compute_qr_code_field(1, name);
        const company_vat_enc = this._compute_qr_code_field(2, vat);
        const timestamp_enc = this._compute_qr_code_field(3, date_isostring);
        const invoice_total_enc = this._compute_qr_code_field(4, amount_total.toString());
        const total_vat_enc = this._compute_qr_code_field(5, amount_tax.toString());

        const str_to_encode = seller_name_enc.concat(company_vat_enc, timestamp_enc, invoice_total_enc, total_vat_enc);

        let binary = '';
        for (let i = 0; i < str_to_encode.length; i++) {
            binary += String.fromCharCode(str_to_encode[i]);
        }
        return btoa(binary);
    },

    _compute_qr_code_field(tag, field) {
        const textEncoder = new TextEncoder();
        const name_byte_array = Array.from(textEncoder.encode(field));
        const name_tag_encoding = [tag];
        const name_length_encoding = [name_byte_array.length];
        return name_tag_encoding.concat(name_length_encoding, name_byte_array);
    },

});

});

```

## File: static\src\xml\OrderReceipt.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-inherit="point_of_sale.OrderReceipt" t-inherit-mode="extension" owl="1">
        <xpath expr="//img[hasclass('pos-receipt-logo')]" position="after">
            <t t-if="receipt.is_gcc_country and !receipt.is_settlement">
                <img t-if="receipt.qr_code" id="qrcode" t-att-src="receipt.qr_code" class="pos-receipt-logo"/>
                <br/>
            </t>
        </xpath>

        <xpath expr="//span[@id='title_english']" position="replace">
            <t t-if="!receipt.is_settlement">
                <span id="title_english">Simplified Tax Invoice</span>
            </t>
        </xpath>

        <xpath expr="//span[@id='title_arabic']" position="replace">
            <t t-if="!receipt.is_settlement">
                <span id="title_arabic">فاتورة ضريبية مبسطة</span>
            </t>
        </xpath>
    </t>
</templates>

```

