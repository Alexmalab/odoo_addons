# Odoo Module: account_payment_invoice_online_payment_patch

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizards

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Payment - Account / Invoice Online Payment Patch",
    'category': 'Accounting/Accounting',
    'depends': ['account_payment'],
    'auto_install': True,
    'data': [
        'data/ir_config_parameter.xml',

        'views/account_portal_templates.xml',

        'wizards/res_config_settings_views.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\ir_config_parameter.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo noupdate="1">

    <record id="enable_portal_payment" model="ir.config_parameter" forcecreate="0">
        <field name="key">account_payment.enable_portal_payment</field>
        <field name="value">True</field>
    </record>

</odoo>

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools import str2bool


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _has_to_be_paid(self):
        enabled_feature = str2bool(
            self.env['ir.config_parameter'].sudo().get_param(
                'account_payment.enable_portal_payment'
            )
        )
        return enabled_feature and super()._has_to_be_paid()

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move

```

## File: views\account_portal_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <template id="portal_my_invoices_payment" inherit_id="account_payment.portal_my_invoices_payment">
        <xpath expr="//t[@t-foreach='invoices']/tr/td[count(t)=2]/t[@t-set='pending_manual_txs']/following-sibling::a[i]" position="attributes">
            <attribute name="t-if">
                invoice._has_to_be_paid()
            </attribute>
        </xpath>
        <xpath expr="//t[@t-foreach='invoices']/tr/td[hasclass('tx_status')]" position="replace">
            <td class="tx_status text-center">
                <t t-if="last_tx">
                    <!-- c/p of account_payment -->
                    <t t-if="invoice.state == 'posted' and invoice.payment_state in ('not_paid', 'partial') and (last_tx.state not in ['pending', 'authorized', 'done', 'cancel'] or (last_tx.state == 'pending' and last_tx.provider_code in ('none', 'custom')))">
                        <span class="badge rounded-pill text-bg-info"><i class="fa fa-fw fa-clock-o"></i><span class="d-none d-md-inline"> Waiting for Payment</span></span>
                    </t>
                    <t t-if="invoice.state == 'posted' and last_tx.state == 'authorized'">
                        <span class="badge rounded-pill text-bg-primary"><i class="fa fa-fw fa-check"/><span class="d-none d-md-inline"> Authorized</span></span>
                    </t>
                    <t t-if="invoice.state == 'posted' and last_tx.state == 'pending' and last_tx.provider_code not in ('none', 'custom')">
                        <span class="badge rounded-pill text-bg-warning"><span class="d-none d-md-inline"> Pending</span></span>
                    </t>
                    <t t-if="invoice.state == 'posted' and invoice.payment_state in ('paid', 'in_payment') or last_tx.state == 'done'">
                        <span class="badge rounded-pill text-bg-success"><i class="fa fa-fw fa-check"></i><span class="d-none d-md-inline"> Paid</span></span>
                    </t>
                    <t t-if="invoice.state == 'posted' and invoice.payment_state == 'reversed'">
                        <span class="badge rounded-pill text-bg-success"><i class="fa fa-fw fa-check"></i><span class="d-none d-md-inline"> Reversed</span></span>
                    </t>
                    <t t-if="invoice.state == 'cancel'">
                        <span class="badge rounded-pill text-bg-danger"><i class="fa fa-fw fa-remove"></i><span class="d-none d-md-inline"> Cancelled</span></span>
                    </t>
                </t>
                <t t-else="">
                    <!-- c/p of account -->
                    <t t-if="invoice.state == 'posted' and invoice.payment_state not in ('in_payment', 'paid', 'reversed')">
                        <span class="badge rounded-pill text-bg-info"><i class="fa fa-fw fa-clock-o" aria-label="Opened" title="Opened" role="img"></i><span class="d-none d-md-inline"> Waiting for Payment</span></span>
                    </t>
                    <t t-if="invoice.state == 'posted' and invoice.payment_state in ('paid', 'in_payment')">
                        <span class="badge rounded-pill text-bg-success"><i class="fa fa-fw fa-check" aria-label="Paid" title="Paid" role="img"></i><span class="d-none d-md-inline"> Paid</span></span>
                    </t>
                    <t t-if="invoice.state == 'posted' and invoice.payment_state == 'reversed'">
                        <span class="badge rounded-pill text-bg-success"><i class="fa fa-fw fa-check" aria-label="Reversed" title="Reversed" role="img"></i><span class="d-none d-md-inline"> Reversed</span></span>
                    </t>
                    <t t-if="invoice.state == 'cancel'">
                        <span class="badge rounded-pill text-bg-warning"><i class="fa fa-fw fa-remove" aria-label="Cancelled" title="Cancelled" role="img"></i><span class="d-none d-md-inline"> Cancelled</span></span>
                    </t>
                </t>
            </td>
        </xpath>
    </template>

    <template id="portal_invoice_page_inherit_payment" inherit_id="account_payment.portal_invoice_page_inherit_payment">
        <xpath expr="//div[@id='portal_pay']" position="attributes">
            <attribute name="t-if">
                invoice._has_to_be_paid()
            </attribute>
        </xpath>
        <xpath expr="//t[@t-call='portal.portal_record_sidebar']//div[hasclass('d-grid')]//a[starts-with(@href, '#')][//i]" position="attributes">
            <attribute name="t-if">
                invoice._has_to_be_paid()
            </attribute>
        </xpath>
    </template>

</odoo>

```

## File: wizards\res_config_settings.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    pay_invoices_online = fields.Boolean(config_parameter='account_payment.enable_portal_payment')

```

## File: wizards\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.account</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <field name="module_account_payment" position="replace">
                <field name="pay_invoices_online"/>
            </field>

            <xpath expr="//label[@for='module_account_payment']" position="replace">
                <label for="pay_invoices_online" string="Invoice Online Payment"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: wizards\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_config_settings

```

