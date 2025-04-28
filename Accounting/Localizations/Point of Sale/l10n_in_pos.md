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
    'icon': '/l10n_in/static/description/icon.png',
    'version': '1.0',
    'description': """GST Point of Sale""",
    'category': 'Accounting/Localizations/Point of Sale',
    'depends': [
        'l10n_in',
        'point_of_sale'
    ],
    'demo': [
        'data/product_demo.xml',
    ],
    'auto_install': True,
    'assets': {
        'point_of_sale.assets': [
            'l10n_in_pos/static/src/js/**/*',
            'l10n_in_pos/static/src/xml/**/*',
        ],
    },
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

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.exceptions import RedirectWarning
from odoo.tools.translate import _


class PosConfig(models.Model):
    _inherit = 'pos.config'

    def open_ui(self):
        for config in self:
            if config.company_id.country_id.code == 'IN' and not config.company_id.state_id:
                msg = _("Your company %s needs to have a correct address in order to open the session.\n"
                "Set the address of your company (Don't forget the State field)") % (config.company_id.name)
                action = {
                    "view_mode": "form",
                    "res_model": "res.company",
                    "type": "ir.actions.act_window",
                    "res_id" : config.company_id.id,
                    "views": [[self.env.ref("base.view_company_form").id, "form"]],
                }
                raise RedirectWarning(msg, action, _('Go to Company configuration'))
        return super(PosConfig, self).open_ui()

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

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class PosSession(models.Model):
    _inherit = 'pos.session'

    def _loader_params_product_product(self):
        result = super()._loader_params_product_product()
        result['search_params']['fields'].append('l10n_in_hsn_code')
        return result

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_order
from . import pos_session
from . import pos_config

```

## File: static\src\js\receipt.js

```javascript
odoo.define('l10n_in_pos.receipt', function (require) {
"use strict";

var { Orderline } = require('point_of_sale.models');
const Registries = require('point_of_sale.Registries');


const L10nInOrderline = (Orderline) => class L10nInOrderline extends Orderline {
    export_for_printing() {
        var line = super.export_for_printing(...arguments);
        line.l10n_in_hsn_code = this.get_product().l10n_in_hsn_code;
        return line;
    }
}
Registries.Model.extend(Orderline, L10nInOrderline);

});

```

## File: static\src\xml\pos_receipt.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="OrderReceipt" t-inherit="point_of_sale.OrderReceipt" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[hasclass('orderlines')]" position="before">
            <t t-if="receipt.partner and env.pos.company.country and env.pos.company.country.code == 'IN'">
                <div class="pos-receipt-center-align">
                    <div><t t-esc="receipt.partner.name" /></div>
                    <t t-if="receipt.partner.phone">
                        <div>
                            <span>Phone: </span>
                            <t t-esc="receipt.partner.phone" />
                        </div>
                    </t>
                    <br />
                </div>
            </t>
        </xpath>
    </t>

    <t t-name="OrderLinesReceipt" t-inherit="point_of_sale.OrderLinesReceipt" t-inherit-mode="extension" owl="1">
        <xpath expr="//t[@t-foreach='receipt.orderlines']" position="inside">
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

