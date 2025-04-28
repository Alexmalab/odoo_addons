# Odoo Module: account_fleet

Category: Accounting/Accounting

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
    'name': 'Accounting/Fleet bridge',
    'category': 'Accounting/Accounting',
    'summary': 'Manage accounting with fleets',
    'description': "",
    'version': '1.0',
    'depends': ['fleet', 'account'],
    'data': [
        'data/fleet_service_type_data.xml',
        'views/account_move_views.xml',
        'views/fleet_vehicle_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'application': False,
    'license': 'LGPL-3',
}

```

## File: data\fleet_service_type_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="data_fleet_service_type_vendor_bill" model="fleet.service.type">
        <field name="name">Vendor Bill</field>
        <field name="category">service</field>
    </record>
</odoo>

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, _


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _post(self, soft=True):
        vendor_bill_service = self.env.ref('account_fleet.data_fleet_service_type_vendor_bill', raise_if_not_found=False)
        if not vendor_bill_service:
            return super()._post(soft)

        val_list = []
        log_list = []
        not_posted_before = self.filtered(lambda r: not r.posted_before)
        posted = super()._post(soft)  # We need the move name to be set, but we also need to know which move are posted for the first time.
        for line in (not_posted_before & posted).line_ids.filtered(lambda ml: ml.vehicle_id and ml.move_id.move_type == 'in_invoice'):
            val = line._prepare_fleet_log_service()
            log = _('Service Vendor Bill: <a href=# data-oe-model=account.move data-oe-id={move_id}>{move_name}</a>').format(
                move_id=line.move_id.id,
                move_name=line.move_id.name,
            )
            val_list.append(val)
            log_list.append(log)
        log_service_ids = self.env['fleet.vehicle.log.services'].create(val_list)
        for log_service_id, log in zip(log_service_ids, log_list):
            log_service_id.message_post(body=log)
        return posted


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    vehicle_id = fields.Many2one('fleet.vehicle', string='Vehicle', index=True)
    need_vehicle = fields.Boolean(compute='_compute_need_vehicle',
        help="Technical field to decide whether the vehicle_id field is editable")

    def _compute_need_vehicle(self):
        self.need_vehicle = False

    def _prepare_fleet_log_service(self):
        vendor_bill_service = self.env.ref('account_fleet.data_fleet_service_type_vendor_bill', raise_if_not_found=False)
        return {
            'service_type_id': vendor_bill_service.id,
            'vehicle_id': self.vehicle_id.id,
            'amount': self.price_subtotal,
            'vendor_id': self.partner_id.id,
            'description': self.name,
        }

```

## File: models\fleet_vehicle.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import Command, models, fields


class FleetVehicle(models.Model):
    _inherit = 'fleet.vehicle'

    bill_count = fields.Integer(compute='_compute_move_ids', string="Bills Count")
    account_move_ids = fields.One2many('account.move', compute='_compute_move_ids')

    def _compute_move_ids(self):
        if not self.env.user.has_group('account.group_account_readonly'):
            self.account_move_ids = False
            self.bill_count = 0
            return

        moves = self.env['account.move.line'].read_group(
            domain=[
                ('vehicle_id', 'in', self.ids),
                ('parent_state', '!=', 'cancel'),
                ('move_id.move_type', 'in', self.env['account.move'].get_purchase_types())
            ],
            fields=['vehicle_id', 'move_id:array_agg'],
            groupby=['vehicle_id'],
        )
        vehicle_move_mapping = {move['vehicle_id'][0]: set(move['move_id']) for move in moves}
        for vehicle in self:
            vehicle.account_move_ids = [Command.set(vehicle_move_mapping.get(vehicle.id, []))]
            vehicle.bill_count = len(vehicle.account_move_ids)

    def action_view_bills(self):
        self.ensure_one()

        form_view_ref = self.env.ref('account.view_move_form', False)
        tree_view_ref = self.env.ref('account_fleet.account_move_view_tree', False)

        result = self.env['ir.actions.act_window']._for_xml_id('account.action_move_in_invoice_type')
        result.update({
            'domain': [('id', 'in', self.account_move_ids.ids)],
            'views': [(tree_view_ref.id, 'tree'), (form_view_ref.id, 'form')],
        })
        return result

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import fleet_vehicle

```

## File: views\account_move_views.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>

    <record id="view_move_form" model="ir.ui.view">
        <field name="name">account.move.form</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='line_ids']//field[@name='name']" position="after">
                <field name='need_vehicle' invisible='1'/>
                <field name='vehicle_id' attrs="{'required': [('need_vehicle', '=', True), ('parent.move_type', 'in', ('in_invoice', 'in_refund'))], 'column_invisible': [('parent.move_type', 'not in', ('in_invoice', 'in_refund'))]}" optional='hidden'/>
            </xpath>
            <xpath expr="//field[@name='invoice_line_ids']//field[@name='name']" position="after">
                <field name='need_vehicle' invisible='1'/>
                <field name='vehicle_id' attrs="{'required': [('need_vehicle', '=', True), ('parent.move_type', 'in', ('in_invoice', 'in_refund'))], 'column_invisible': [('parent.move_type', 'not in', ('in_invoice', 'in_refund'))]}" optional='hidden'/>
            </xpath>
        </field>
    </record>

    <record id="account_move_view_tree" model="ir.ui.view">
        <field name="name">account.move.tree.inherit.fleet</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_tree"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='date']" position="attributes">
                <attribute name="string">Creation Date</attribute>
            </xpath>
            <xpath expr="//field[@name='date']" position="after">
                <field name="invoice_date" optional="show"/>
            </xpath>
        </field>
    </record>

    <record id="view_move_line_tree_fleet" model="ir.ui.view">
        <field name="name">view.move.line.tree.fleet</field>
        <field name="model">account.move.line</field>
        <field name="inherit_id" ref="account.view_move_line_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='name']" position='after'>
                <field name="vehicle_id" optional='hidden'/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\fleet_vehicle_views.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <record id="fleet_vehicle_view_form" model="ir.ui.view">
        <field name="name">fleet.vehicle.form</field>
        <field name="model">fleet.vehicle</field>
        <field name="inherit_id" ref="fleet.fleet_vehicle_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='open_assignation_logs']" position='before'>
                <button name="action_view_bills"
                    type="object"
                    class="oe_stat_button"
                    icon="fa-pencil-square-o"
                    attrs="{'invisible': [('bill_count', '=', 0)]}"
                    help="show the vendor bills for this vehicle">
                    <field name="bill_count" widget="statinfo" string="Bills"/>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\account_automatic_entry_wizard.py

```python
from odoo import api, fields, models, _

class AutomaticEntryWizard(models.TransientModel):
    _inherit = 'account.automatic.entry.wizard'

    def _get_move_line_dict_vals_change_period(self, aml, date):
        res = super()._get_move_line_dict_vals_change_period(aml, date)
        if aml.vehicle_id:
            for move_line_data in res:
                if move_line_data[2]['account_id'] == aml.account_id.id:
                    move_line_data[2]['vehicle_id'] = aml.vehicle_id.id
        return res

```

## File: wizard\__init__.py

```python
from . import account_automatic_entry_wizard

```

