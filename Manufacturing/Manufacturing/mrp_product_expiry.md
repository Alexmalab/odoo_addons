# Odoo Module: mrp_product_expiry

Category: Manufacturing/Manufacturing

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Manufacturing Expiry',
    'version': '1.0',
    'category': 'Manufacturing/Manufacturing',
    'summary': 'Manufacturing Expiry',
    'description': """
Technical module.
    """,
    'depends': ['mrp', 'product_expiry'],
    'data': [
        'wizard/confirm_expiry_view.xml',
    ],
    'installable': True,
    'auto_install': True,
    'application': False,
    'license': 'LGPL-3',
}

```

## File: models\mrp_production.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _


class MrpWorkorder(models.Model):
    _inherit = 'mrp.production'

    def _pre_button_mark_done(self):
        confirm_expired_lots = self._check_expired_lots()
        if confirm_expired_lots:
            return confirm_expired_lots
        return super()._pre_button_mark_done()

    def _check_expired_lots(self):
        # We use the 'skip_expired' context key to avoid to make the check when
        # user already confirmed the wizard about using expired lots.
        if self.env.context.get('skip_expired'):
            return False
        expired_lot_ids = self.move_raw_ids.move_line_ids.filtered(lambda ml: ml.lot_id.product_expiry_alert).lot_id.ids
        if expired_lot_ids:
            return {
                'name': _('Confirmation'),
                'type': 'ir.actions.act_window',
                'res_model': 'expiry.picking.confirmation',
                'view_mode': 'form',
                'target': 'new',
                'context': self._get_expired_context(expired_lot_ids),
            }

    def _get_expired_context(self, expired_lot_ids):
        context = dict(self.env.context)
        context.update({
            'default_lot_ids': [(6, 0, expired_lot_ids)],
            'default_production_ids': self.ids,
        })
        return context

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mrp_production


```

## File: wizard\confirm_expiry.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class ConfirmExpiry(models.TransientModel):
    _inherit = 'expiry.picking.confirmation'

    production_ids = fields.Many2many('mrp.production', readonly=True)
    workorder_id = fields.Many2one('mrp.workorder', readonly=True)

    @api.depends('lot_ids')
    def _compute_descriptive_fields(self):
        if self.production_ids or self.workorder_id:
            # Shows expired lots only if we are more than one expired lot.
            self.show_lots = len(self.lot_ids) > 1
            if self.show_lots:
                # For multiple expired lots, they are listed in the wizard view.
                self.description = _(
                    "You are going to use some expired components."
                    "\nDo you confirm you want to proceed ?"
                )
            else:
                # For one expired lot, its name is written in the wizard message.
                self.description = _(
                    "You are going to use the component %(product_name)s, %(lot_name)s which is expired."
                    "\nDo you confirm you want to proceed ?",
                    product_name=self.lot_ids.product_id.display_name,
                    lot_name=self.lot_ids.name,
                )
        else:
            super(ConfirmExpiry, self)._compute_descriptive_fields()

    def confirm_produce(self):
        ctx = dict(self._context, skip_expired=True)
        ctx.pop('default_lot_ids')
        return self.production_ids.with_context(ctx).button_mark_done()

    def confirm_workorder(self):
        ctx = dict(self._context, skip_expired=True)
        ctx.pop('default_lot_ids')
        return self.workorder_id.with_context(ctx).record_production()


```

## File: wizard\confirm_expiry_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="confirm_expiry_view_mrp_inherit" model="ir.ui.view">
        <field name="name">Confirm</field>
        <field name="model">expiry.picking.confirmation</field>
        <field name="inherit_id" ref="product_expiry.confirm_expiry_view"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='description']" position="after">
                <field name="picking_ids" invisible="1"/>
                <field name="production_ids" invisible="1"/>
                <field name="workorder_id" invisible="1"/>
            </xpath>
            <xpath expr="//button[@name='process']" position="attributes">
                <attribute name="attrs">{'invisible': [('picking_ids', '=', [])]}</attribute>
            </xpath>
            <xpath expr="//button[@name='process_no_expired']" position="attributes">
                <attribute name="attrs">{'invisible': [('picking_ids', '=', [])]}</attribute>
            </xpath>
            <xpath expr="//button[@special='cancel']" position="attributes">
                <attribute name="attrs">{'invisible': [('production_ids', '!=', [])]}</attribute>
            </xpath>
            <xpath expr="//button[@name='process']" position="after">
                <!-- From Produce Product wizard -->
                <button name="confirm_produce"
                    string="Confirm"
                    type="object"
                    attrs="{'invisible': [('production_ids', '=', [])]}"
                    class="btn-primary"/>
                <button special="cancel"
                    string="Discard"
                    attrs="{'invisible': [('production_ids', '=', [])]}"
                    class="btn-secondary"/>
                <!-- From a Workorder -->
                <button name="confirm_workorder"
                    string="Confirm"
                    type="object"
                    attrs="{'invisible': [('workorder_id', '=', False)]}"
                    class="btn-primary"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import confirm_expiry

```

