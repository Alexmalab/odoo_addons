# Odoo Module: stock_sms

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

from odoo import api, SUPERUSER_ID


def _assign_default_sms_template_picking_id(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    company_ids_without_default_sms_template_id = env['res.company'].search([
        ('stock_sms_confirmation_template_id', '=', False)
    ])
    default_sms_template_id = env.ref('stock_sms.sms_template_data_stock_delivery', raise_if_not_found=False)
    if default_sms_template_id:
        company_ids_without_default_sms_template_id.write({
            'stock_sms_confirmation_template_id': default_sms_template_id.id,
        })

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Stock - SMS",
    'summary': 'Send text messages when final stock move',
    'description': "Send text messages when final stock move",
    'category': 'Hidden',
    'version': '1.0',
    'depends': ['stock', 'sms'],
    'data': [
        'data/sms_data.xml',
        'views/res_config_settings_views.xml',
        'wizard/confirm_stock_sms_views.xml',
        'security/ir.model.access.csv',
        'security/sms_security.xml',
    ],
    'application': False,
    'auto_install': True,
    'post_init_hook': '_assign_default_sms_template_picking_id',
    'license': 'LGPL-3',
}

```

## File: data\sms_data.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <data noupdate="1">
        <record id="sms_template_data_stock_delivery" model="sms.template">
            <field name="name">Delivery: Send by SMS Text Message</field>
            <field name="model_id" ref="stock.model_stock_picking"/>
            <field name="body">
                %if object.origin:
                    ${object.company_id.name}: We are glad to inform you that your order n° ${object.origin} has been shipped.
                %else:
                    ${object.company_id.name}: We are glad to inform you that your order has been shipped.
                %endif
                %if object.carrier_tracking_ref:
                    Your tracking reference is ${object.carrier_tracking_ref}.
                %endif
            </field>
        </record>
    </data>
</odoo>

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Company(models.Model):
    _inherit = "res.company"

    def _default_confirmation_sms_picking_template(self):
        try:
            return self.env.ref('stock_sms.sms_template_data_stock_delivery').id
        except ValueError:
            return False

    stock_move_sms_validation = fields.Boolean("SMS Confirmation", default=True)
    stock_sms_confirmation_template_id = fields.Many2one(
        'sms.template', string="SMS Template",
        domain="[('model', '=', 'stock.picking')]",
        default=_default_confirmation_sms_picking_template,
        help="SMS sent to the customer once the order is done.")
    has_received_warning_stock_sms = fields.Boolean()

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    stock_move_sms_validation = fields.Boolean(
        related='company_id.stock_move_sms_validation',
        string='SMS Validation with stock move', readonly=False)
    stock_sms_confirmation_template_id = fields.Many2one(
        related='company_id.stock_sms_confirmation_template_id', readonly=False)

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _

import threading


class Picking(models.Model):
    _inherit = 'stock.picking'

    def _sms_get_number_fields(self):
        """ This method returns the fields to use to find the number to use to
        send an SMS on a record. """
        return ['mobile', 'phone']

    def _check_sms_confirmation_popup(self):
        is_delivery = self.company_id.stock_move_sms_validation \
                and self.picking_type_id.code == 'outgoing' \
                and (self.partner_id.mobile or self.partner_id.phone)
        if is_delivery and not getattr(threading.currentThread(), 'testing', False) \
                and not self.env.registry.in_test_mode() \
                and not self.company_id.has_received_warning_stock_sms \
                and self.company_id.stock_move_sms_validation:
            view = self.env.ref('stock_sms.view_confirm_stock_sms')
            wiz = self.env['confirm.stock.sms'].create({'picking_id': self.id})
            return {
                'name': _('SMS'),
                'type': 'ir.actions.act_window',
                'view_mode': 'form',
                'res_model': 'confirm.stock.sms',
                'views': [(view.id, 'form')],
                'view_id': view.id,
                'target': 'new',
                'res_id': wiz.id,
                'context': self.env.context,
            }
        return False

    def _send_confirmation_email(self):
        super(Picking, self)._send_confirmation_email()
        if not getattr(threading.currentThread(), 'testing', False) and not self.env.registry.in_test_mode():
            pickings = self.filtered(lambda p: p.company_id.stock_move_sms_validation and p.picking_type_id.code == 'outgoing' and (p.partner_id.mobile or p.partner_id.phone))
            for picking in pickings:
                # Sudo as the user has not always the right to read this sms template.
                template = picking.company_id.sudo().stock_sms_confirmation_template_id
                picking._message_sms_with_template(
                    template=template,
                    partner_ids=picking.partner_id.ids,
                    put_in_queue=False
                )

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_company
from . import res_config_settings
from . import stock_picking

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_sms_template_stock_manager,access.sms.template.stock.manager,sms.model_sms_template,stock.group_stock_manager,1,1,1,1

```

## File: security\sms_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_rule_sms_template_stock_manager" model="ir.rule">
        <field name="name">SMS Template: stock manager CUD on stock picking templates</field>
        <field name="model_id" ref="sms.model_sms_template"/>
        <field name="groups" eval="[(4, ref('stock.group_stock_manager'))]"/>
        <field name="domain_force">[('model_id.model', '=', 'stock.picking')]</field>
        <field name="perm_read" eval="False"/>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form_stock" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.delivery.stock</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="stock.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='module_stock_sms']" position="replace">
                <field name="stock_move_sms_validation"/>
            </xpath>
            <xpath expr="//div[@id='stock_confirmation_sms']" position="replace">
                <div class="row mt16" attrs="{'invisible': [('stock_move_sms_validation', '=', False)]}">
                    <label for="stock_sms_confirmation_template_id" string="SMS Template" class="col-lg-4 o_light_label"/>
                    <field name="stock_sms_confirmation_template_id" class="oe_inline" attrs="{'required': [('stock_move_sms_validation', '=', True)]}" context="{'default_model': 'stock.picking'}"/>
                </div>
                <widget name="iap_buy_more_credits" service_name="sms" attrs="{'invisible': [('stock_move_sms_validation', '=', False)]}"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: wizard\confirm_stock_sms.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ConfirmStockSms(models.TransientModel):
    _name = 'confirm.stock.sms'
    _description = 'Confirm Stock SMS'

    picking_id = fields.Many2one('stock.picking', required=True)
    company_id = fields.Many2one('res.company', string='Company', required=True, related='picking_id.company_id')

    def send_sms(self):
        self.ensure_one()
        if not self.company_id.has_received_warning_stock_sms:
            self.company_id.sudo().write({'has_received_warning_stock_sms': True})
        return self.picking_id.button_validate()

    def dont_send_sms(self):
        self.ensure_one()
        if not self.company_id.has_received_warning_stock_sms:
            self.company_id.sudo().write({
                'has_received_warning_stock_sms': True,
                'stock_move_sms_validation': False,
            })
        return self.picking_id.button_validate()

```

## File: wizard\confirm_stock_sms_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_confirm_stock_sms" model="ir.ui.view">
        <field name="name">stock_confirm_sms</field>
        <field name="model">confirm.stock.sms</field>
        <field name="arch" type="xml">
            <form string="SMS">
                You are about to confirm this Delivery Order by SMS Text Message.<br/>
                This feature can easily be disabled from the Settings of Inventory or by clicking on "Disable SMS".<br/>
                <group invisible="1">
                    <field name="picking_id" invisible="1"/>
                    <field name="company_id" invisible="1"/>
                </group>
                <footer>
                    <button name="send_sms" type="object"
                            string="Confirm" class="oe_highlight"/>
                    <button name="dont_send_sms" type="object"
                            string="Disable SMS" class="btn btn-secondary"/>
                    <button special="cancel" string="Cancel"/>
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import confirm_stock_sms

```

