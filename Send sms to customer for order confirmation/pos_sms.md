# Odoo Module: pos_sms

Category: Send sms to customer for order confirmation

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'POS - SMS',
    'category': 'Send sms to customer for order confirmation',
    'description': """This module integrates the Point of Sale with SMS""",
    'depends': ['point_of_sale', 'sms'],
    'data': [
        'data/sms_data.xml',
        'views/res_config_settings_views.xml',
        'data/point_of_sale_data.xml',
    ],
    'assets': {
        'point_of_sale._assets_pos': [
            'pos_sms/static/src/**/*',
        ],
    },
    'license': 'LGPL-3',
    'auto_install': True
}

```

## File: data\point_of_sale_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record model="pos.config" id="point_of_sale.pos_config_main" forcecreate="0">
        <field name="sms_receipt_template_id" ref="sms_template_data_point_of_sale"/>
    </record>
</odoo>

```

## File: data\sms_data.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <data noupdate="1">
        <record id="sms_template_data_point_of_sale" model="sms.template">
            <field name="name">POS: Sent Order Confirmation via Text</field>
            <field name="model_id" ref="point_of_sale.model_pos_order"/>
            <field name="body">
                {{ object.company_id.name }} : Your order with reference: {{ object.pos_reference }} was processed succesfully with amount {{  object.currency_id.format(object.amount_total) }}. Use {{ object.pos_reference }}  for further reference
            </field>
        </record>
    </data>
</odoo>

```

## File: models\pos_config.py

```python
from odoo import fields, models


class PosConfig(models.Model):
    _inherit = 'pos.config'

    sms_receipt_template_id = fields.Many2one('sms.template', string="Sms Receipt template", domain=[('model', '=', 'pos.order')], help="SMS will be sent to the customer based on this template")

```

## File: models\pos_order.py

```python
from odoo import models


class PosOrder(models.Model):
    _inherit = 'pos.order'

    def action_sent_message_on_sms(self, phone, _, basic_image=False):
        if not (self and self.config_id.module_pos_sms and self.config_id.sms_receipt_template_id and phone):
            return
        self.ensure_one()
        sms_composer = self.env['sms.composer'].with_context(active_id=self.id).create(
            {
                'composition_mode': 'comment',
                'numbers': phone,
                'recipient_single_number_itf': phone,
                'template_id': self.config_id.sms_receipt_template_id.id,
                'res_model': 'pos.order'
            }
        )
        self.mobile = phone
        sms_composer.action_send_sms()

```

## File: models\res_config_settings.py

```python
from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    pos_sms_receipt_template_id = fields.Many2one('sms.template', related='pos_config_id.sms_receipt_template_id', readonly=False)

```

## File: models\__init__.py

```python
from . import pos_config
from . import pos_order
from . import res_config_settings

```

## File: static\src\overrides\partner_list.js

```javascript
import { patch } from "@web/core/utils/patch";
import { PartnerList } from "@point_of_sale/app/screens/partner_list/partner_list";

patch(PartnerList.prototype, {
    getPhoneSearchTerms() {
        return ["phone_mobile_search"];
    },
});

```

## File: static\src\overrides\receipt_screen.js

```javascript
import { patch } from "@web/core/utils/patch";
import { ReceiptScreen } from "@point_of_sale/app/screens/receipt_screen/receipt_screen";

patch(ReceiptScreen.prototype, {
    setup() {
        super.setup(...arguments);
    },
    showPhoneInput() {
        return super.showPhoneInput() || this.pos.config.module_pos_sms;
    },
    actionSendReceiptOnSMS() {
        this.sendReceipt.call({
            action: "action_sent_message_on_sms",
            destination: this.state.phone,
            name: "SMS",
        });
    },
});

```

## File: static\src\overrides\receipt_screen.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="ReceiptScreen" t-inherit="point_of_sale.ReceiptScreen" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('sending-receipt-management')]" position="inside">
            <button t-if="pos.config.module_pos_sms" t-att-style="`width: ${this.ui.isSmall ? '4rem' : '8rem'}`"  class="btn btn-primary h-100" t-att-disabled="!isValidPhone" t-on-click="() => this.actionSendReceiptOnSMS()">
                <i t-attf-class="fa {{sendReceipt.status === 'loading' and sendReceipt.lastArgs?.[0]?.name === 'SMS' ? 'fa-fw fa-spin fa-circle-o-notch' : 'fa-lg fa-mobile'}}" />
            </button>
        </xpath>
    </t>
</templates>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_sms_res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.sms.pos</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="95"/>
        <field name="inherit_id" ref="point_of_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="warning_text_pos_sms" position="replace">
                <div class="row">
                    <label string="Receipt template" for="pos_sms_receipt_template_id" class="col-lg-3 o_light_label"/>
                    <field name="pos_sms_receipt_template_id" required="pos_module_pos_sms"/>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

