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
        'views/point_of_sale.xml',
    ],
    'demo': [
        'data/product_demo.xml',
    ],
    'qweb': [
        'static/src/xml/pos_receipt.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\product_demo.xml

```xml
<odoo>
    <data noupdate="1">
        <record id="point_of_sale.desk_organizer" model="product.product">
            <field name="l10n_in_hsn_code">9403</field>
            <field name="l10n_in_hsn_description">Other furniture and parts thereof.</field>
        </record>
        <record id="point_of_sale.desk_pad" model="product.product">
            <field name="l10n_in_hsn_code">9403</field>
            <field name="l10n_in_hsn_description">Other furniture and parts thereof.</field>
        </record>
        <record id="point_of_sale.led_lamp" model="product.product">
            <field name="l10n_in_hsn_code">8539.50.00</field>
            <field name="l10n_in_hsn_description">Light-emitting diode (LED) lamps</field>
        </record>
        <record id="point_of_sale.letter_tray" model="product.product">
            <field name="l10n_in_hsn_code">4819.60.00</field>
            <field name="l10n_in_hsn_description">Box files, letter trays, storage boxes and similar articles, of a kind used in offices, shops or the like</field>
        </record>
        <record id="point_of_sale.magnetic_board" model="product.product">
            <field name="l10n_in_hsn_code">3921.90.99</field>
            <field name="l10n_in_hsn_description">Other plates, sheets film , foil and strip, of plastics</field>
        </record>
        <record id="point_of_sale.product_product_consumable" model="product.product">
            <field name="l10n_in_hsn_code">8443.32.90</field>
            <field name="l10n_in_hsn_description">Other, capable of connecting to an automatic data processing machine or to a network</field>
        </record>
        <record id="point_of_sale.monitor_stand" model="product.product">
            <field name="l10n_in_hsn_code">9403</field>
            <field name="l10n_in_hsn_description">Other furniture and parts thereof.</field>
        </record>
        <record id="point_of_sale.newspaper_rack" model="product.product">
            <field name="l10n_in_hsn_code">9403.10.90</field>
            <field name="l10n_in_hsn_description">Metal furniture of a kind used in offices</field>
        </record>
        <record id="point_of_sale.small_shelf" model="product.product">
            <field name="l10n_in_hsn_code">9403.10.90</field>
            <field name="l10n_in_hsn_description">Metal furniture of a kind used in offices</field>
        </record>
        <record id="point_of_sale.product_product_tip" model="product.product">
            <field name="l10n_in_hsn_code">8209.00.90</field>
            <field name="l10n_in_hsn_description">Plates, sticks, tips and the like for tools, unmounted, of cermets.</field>
        </record>
        <record id="point_of_sale.wall_shelf" model="product.product">
            <field name="l10n_in_hsn_code">9403.10.90</field>
            <field name="l10n_in_hsn_description">Metal furniture of a kind used in offices</field>
        </record>
        <record id="point_of_sale.whiteboard" model="product.product">
            <field name="l10n_in_hsn_code">3926.10.99</field>
            <field name="l10n_in_hsn_description">Office supplies of a kind classified as stationary other than pins,clips, and writing instruments</field>
        </record>
        <record id="point_of_sale.whiteboard_pen" model="product.product">
            <field name="l10n_in_hsn_code">9608</field>
            <field name="l10n_in_hsn_description">Ball point pens; felt tipped and other porous-tipped pens and markers; fountain pens, stylograph pens and other pens; duplicating stylos; propelling or sliding pencils; pen-holders, pencilholders and similar holders; parts (including caps and clips) of the foregoing articles, othe than those of heading 9609
            </field>
        </record>
    </data>
</odoo>

```

## File: models\pos_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class PosOrder(models.Model):
    _inherit = 'pos.order'
    #TODO: remove in master
    @api.model
    def _get_account_move_line_group_data_type_key(self, data_type, values, options={}):
        res = super(PosOrder, self)._get_account_move_line_group_data_type_key(data_type, values, options)
        if data_type == 'tax' and res:
            if self.env['account.tax'].browse(values['tax_line_id']).company_id.country_id.code == 'IN':
                return res + (values['product_uom_id'], values['product_id'])
        return res
    #TODO: remove in master
    def _prepare_account_move_line(self, line, partner_id, current_company, currency_id, rounding_method):
        res = super(PosOrder, self)._prepare_account_move_line(line, partner_id, current_company, currency_id, rounding_method)
        for line_values in res:
            if line_values.get('data_type') in ['tax','product']:
                line_values['values'].update({
                    'product_id': line.product_id.id,
                    'product_uom_id': line.product_id.uom_id.id
                    })
        return res

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

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_order

```

## File: static\src\js\receipt.js

```javascript
odoo.define('l10n_in_pos.receipt', function (require) {
"use strict";

var models = require('point_of_sale.models');

models.load_fields('product.product', 'l10n_in_hsn_code');

var _super_orderline = models.Orderline.prototype;
models.Orderline = models.Orderline.extend({
    export_for_printing: function() {
        var line = _super_orderline.export_for_printing.apply(this,arguments);
        line.l10n_in_hsn_code = this.get_product().l10n_in_hsn_code;
        return line;
    },
});

});

```

## File: static\src\xml\pos_receipt.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="OrderReceipt" t-inherit="point_of_sale.OrderReceipt" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[hasclass('orderlines')]" position="before">
            <t t-if="receipt.client and env.pos.company.country and env.pos.company.country.code == 'IN'">
                <div class="pos-receipt-center-align">
                    <div><t t-esc="receipt.client.name" /></div>
                    <t t-if="receipt.client.phone">
                        <div>
                            <span>Phone: </span>
                            <t t-esc="receipt.client.phone" />
                        </div>
                    </t>
                    <br />
                </div>
            </t>
        </xpath>
        <xpath expr="//WrappedProductNameLines" position="after">
            <t t-if="line.l10n_in_hsn_code and env.pos.company.country and env.pos.company.country.code == 'IN'">
                <div class="pos-receipt-left-padding">
                    <span>HSN Code: </span>
                    <t t-esc="line.l10n_in_hsn_code"/>
                </div>
            </t>
        </xpath>
    </t>
</templates>

```

## File: views\point_of_sale.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets" inherit_id="point_of_sale.assets">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/l10n_in_pos/static/src/js/receipt.js"></script>
        </xpath>
    </template>
</odoo>

```

