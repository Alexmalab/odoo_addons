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
    'countries': ['in'],
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
        'point_of_sale._assets_pos': [
            'l10n_in_pos/static/src/**/*',
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
            <field name="l10n_in_hsn_code">85395000</field>
            <field name="l10n_in_hsn_description">Light-emitting diode (LED) lamps</field>
        </record>
        <record id="point_of_sale.letter_tray" model="product.product">
            <field name="l10n_in_hsn_code">48196000</field>
            <field name="l10n_in_hsn_description">Box files, letter trays, storage boxes and similar articles, of a kind used in offices, shops or the like</field>
        </record>
        <record id="point_of_sale.magnetic_board" model="product.product">
            <field name="l10n_in_hsn_code">39219099</field>
            <field name="l10n_in_hsn_description">Other plates, sheets film , foil and strip, of plastics</field>
        </record>
        <record id="point_of_sale.product_product_consumable" model="product.product">
            <field name="l10n_in_hsn_code">84433290</field>
            <field name="l10n_in_hsn_description">Other, capable of connecting to an automatic data processing machine or to a network</field>
        </record>
        <record id="point_of_sale.monitor_stand" model="product.product">
            <field name="l10n_in_hsn_code">9403</field>
            <field name="l10n_in_hsn_description">Other furniture and parts thereof.</field>
        </record>
        <record id="point_of_sale.newspaper_rack" model="product.product">
            <field name="l10n_in_hsn_code">94031090</field>
            <field name="l10n_in_hsn_description">Metal furniture of a kind used in offices</field>
        </record>
        <record id="point_of_sale.small_shelf" model="product.product">
            <field name="l10n_in_hsn_code">94031090</field>
            <field name="l10n_in_hsn_description">Metal furniture of a kind used in offices</field>
        </record>
        <record id="point_of_sale.product_product_tip" model="product.product">
            <field name="l10n_in_hsn_code">82090090</field>
            <field name="l10n_in_hsn_description">Plates, sticks, tips and the like for tools, unmounted, of cermets.</field>
        </record>
        <record id="point_of_sale.wall_shelf" model="product.product">
            <field name="l10n_in_hsn_code">9403.10.90</field>
            <field name="l10n_in_hsn_description">Metal furniture of a kind used in offices</field>
        </record>
        <record id="point_of_sale.whiteboard" model="product.product">
            <field name="l10n_in_hsn_code">39261099</field>
            <field name="l10n_in_hsn_description">Office supplies of a kind classified as stationary other than pins,clips, and writing instruments</field>
        </record>
        <record id="point_of_sale.whiteboard_pen" model="product.product">
            <field name="l10n_in_hsn_code">9608</field>
            <field name="l10n_in_hsn_description">Ball point pens; felt tipped and other porous-tipped pens and markers; fountain pens, stylograph pens and other pens; duplicating stylos; propelling or sliding pencils; pen-holders, pencilholders and similar holders; parts (including caps and clips) of the foregoing articles, othe than those of heading 9609
            </field>
        </record>
        <record model="pos.bill" id="500_00" forcecreate="0">
            <field name="name">500.00</field>
            <field name="value">500.00</field>
        </record>
    </data>
</odoo>

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, fields, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_in_pos_session_ids = fields.One2many("pos.session", "move_id", "POS Sessions")

    @api.depends('l10n_in_pos_session_ids')
    def _compute_l10n_in_state_id(self):
        res = super()._compute_l10n_in_state_id()
        to_compute = self.filtered(lambda m: m.country_code == 'IN' and not m.l10n_in_state_id and m.journal_id.type == 'general' and m.l10n_in_pos_session_ids)
        for move in to_compute:
            move.l10n_in_state_id = move.company_id.state_id
        return res

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

from odoo import models, api


class PosSession(models.Model):
    _inherit = 'pos.session'

    def _loader_params_product_product(self):
        result = super()._loader_params_product_product()
        result['search_params']['fields'].append('l10n_in_hsn_code')
        return result

    def _loader_params_account_tax(self):
        account_tax_params = super()._loader_params_account_tax()
        account_tax_params['search_params']['fields'].append('tax_group_id')
        return account_tax_params

    @api.model
    def _load_onboarding_main_config_data(self, shop_config):
        if shop_config.company_id.country_code == 'IN' and not shop_config.company_id.state_id:
            return

        super()._load_onboarding_main_config_data(shop_config)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_order
from . import pos_session
from . import pos_config
from . import account_move

```

## File: static\src\overrides\components\pos_receipt.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="l10n_in_pos.ReceiptHeader" t-inherit="point_of_sale.ReceiptHeader" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('pos-receipt-contact')]" position="after">
            <t t-if="props.data.partner and props.data.company?.country?.code == 'IN'">
                <div class="pos-receipt-center-align">
                    <div><t t-esc="props.data.partner.name" /></div>
                    <t t-if="props.data.partner.phone">
                        <div>
                            <span>Phone: </span>
                            <t t-esc="props.data.partner.phone" />
                        </div>
                    </t>
                    <br />
                </div>
            </t>
        </xpath>
    </t>

    <t t-name="l10n_in_pos.OrderReceipt" t-inherit="point_of_sale.OrderReceipt" t-inherit-mode="extension">
        <xpath expr="//Orderline" position="inside">
            <t t-if="line.l10n_in_hsn_code and props.data.headerData.company.country?.code === 'IN'">
                <div class="pos-receipt-left-padding">
                    <span>HSN Code: </span>
                    <t t-esc="line.l10n_in_hsn_code"/>
                </div>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\overrides\models\order.js

```javascript
/** @odoo-module */

import { Order } from "@point_of_sale/app/store/models";
import { patch } from "@web/core/utils/patch";

patch(Order.prototype, {
    export_for_printing() {
        const result = super.export_for_printing(...arguments);
        if (this.pos.company.country.code === 'IN') {
            result.tax_details.forEach((tax) => {
                tax.tax.letter = tax.tax.tax_group_id[1]
            })
        }
        return result;
    },
});

```

## File: static\src\overrides\models\orderline.js

```javascript
/** @odoo-module */

import { Orderline } from "@point_of_sale/app/store/models";
import { patch } from "@web/core/utils/patch";

patch(Orderline.prototype, {
    getDisplayData() {
        return {
            ...super.getDisplayData(),
            l10n_in_hsn_code: this.get_product().l10n_in_hsn_code,
        };
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
            partner: this.selectedOrder.partner
        }
    },
});

```

