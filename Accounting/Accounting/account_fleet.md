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
    'version': '1.0',
    'depends': ['fleet', 'account'],
    'data': [
        'data/fleet_service_type_data.xml',
        'views/account_move_views.xml',
        'views/fleet_vehicle_views.xml',
        'views/fleet_vehicle_log_services_views.xml'
    ],
    'installable': True,
    'auto_install': True,
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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _post(self, soft=True):
        vendor_bill_service = self.env.ref('account_fleet.data_fleet_service_type_vendor_bill', raise_if_not_found=False)
        if not vendor_bill_service:
            return super()._post(soft)

        val_list = []
        log_list = []
        posted = super()._post(soft)  # We need the move name to be set, but we also need to know which move are posted for the first time.
        for line in posted.line_ids:
            if not line.vehicle_id or line.vehicle_log_service_ids\
                    or line.move_id.move_type != 'in_invoice'\
                    or line.display_type != 'product':
                continue
            val = line._prepare_fleet_log_service()
            log = _('Service Vendor Bill: %s', line.move_id._get_html_link())
            val_list.append(val)
            log_list.append(log)
        log_service_ids = self.env['fleet.vehicle.log.services'].create(val_list)
        for log_service_id, log in zip(log_service_ids, log_list):
            log_service_id.message_post(body=log)
        return posted


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    vehicle_id = fields.Many2one('fleet.vehicle', string='Vehicle', index='btree_not_null')
    # used to decide whether the vehicle_id field is editable
    need_vehicle = fields.Boolean(compute='_compute_need_vehicle')
    vehicle_log_service_ids = fields.One2many(export_string_translation=False,
        comodel_name='fleet.vehicle.log.services', inverse_name='account_move_line_id')  # One2one

    def _compute_need_vehicle(self):
        self.need_vehicle = False

    def _prepare_fleet_log_service(self):
        vendor_bill_service = self.env.ref('account_fleet.data_fleet_service_type_vendor_bill', raise_if_not_found=False)
        return {
            'service_type_id': vendor_bill_service.id,
            'vehicle_id': self.vehicle_id.id,
            'vendor_id': self.partner_id.id,
            'description': self.name,
            'account_move_line_id': self.id,
        }

    def write(self, vals):
        if 'vehicle_id' in vals and not vals['vehicle_id']:
            self.sudo().vehicle_log_service_ids.with_context(ignore_linked_bill_constraint=True).unlink()
        return super().write(vals)

    def unlink(self):
        self.sudo().vehicle_log_service_ids.with_context(ignore_linked_bill_constraint=True).unlink()
        return super().unlink()

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

        moves = self.env['account.move.line']._read_group(
            domain=[
                ('vehicle_id', 'in', self.ids),
                ('parent_state', '!=', 'cancel'),
                ('move_id.move_type', 'in', self.env['account.move'].get_purchase_types())
            ],
            groupby=['vehicle_id'],
            aggregates=['move_id:array_agg'],
        )
        vehicle_move_mapping = {vehicle.id: set(move_ids) for vehicle, move_ids in moves}
        for vehicle in self:
            vehicle.account_move_ids = [Command.set(vehicle_move_mapping.get(vehicle.id, []))]
            vehicle.bill_count = len(vehicle.account_move_ids)

    def action_view_bills(self):
        self.ensure_one()

        form_view_ref = self.env.ref('account.view_move_form', False)
        list_view_ref = self.env.ref('account_fleet.account_move_view_tree', False)

        result = self.env['ir.actions.act_window']._for_xml_id('account.action_move_in_invoice_type')
        result.update({
            'domain': [('id', 'in', self.account_move_ids.ids)],
            'views': [(list_view_ref.id, 'list'), (form_view_ref.id, 'form')],
        })
        return result

```

## File: models\fleet_vehicle_log_services.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError

class FleetVehicleLogServices(models.Model):
    _inherit = 'fleet.vehicle.log.services'

    account_move_line_id = fields.Many2one(comodel_name='account.move.line')  # One2one
    account_move_state = fields.Selection(related='account_move_line_id.parent_state')
    amount = fields.Monetary(string='Cost', compute="_compute_amount", inverse="_inverse_amount",
        readonly=False, store=True, tracking=True)
    vehicle_id = fields.Many2one(comodel_name='fleet.vehicle', string='Vehicle',
        compute="_compute_vehicle_id", store=True, readonly=False, required=True)

    @api.depends('account_move_line_id.vehicle_id')
    def _compute_vehicle_id(self):
        for service in self:
            # We avoid emptying the vehicle_id as it is a required field
            if not service.account_move_line_id.vehicle_id:
                continue
            service.vehicle_id = service.account_move_line_id.vehicle_id

    def _inverse_amount(self):
        if any(service.account_move_line_id for service in self):
            raise UserError(_("You cannot modify amount of services linked to an account move line. Do it on the related accounting entry instead."))

    @api.depends('account_move_line_id.price_subtotal')
    def _compute_amount(self):
        for log_service in self:
            log_service.amount = log_service.account_move_line_id.debit

    def action_open_account_move(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'account.move',
            'target': 'current',
            'name': _('Bill'),
            'res_id': self.account_move_line_id.move_id.id,
        }

    @api.ondelete(at_uninstall=False)
    def _unlink_if_no_linked_bill(self):
        if self.env.context.get('ignore_linked_bill_constraint'):
            return
        if any(log_service.account_move_line_id for log_service in self):
            raise UserError(_("You cannot delete log services records because one or more of them were bill created."))

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import fleet_vehicle
from . import fleet_vehicle_log_services

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
            <xpath expr="//field[@name='line_ids']//field[@name='account_id']" position="after">
                <field name='need_vehicle' column_invisible="True"/>
                <field name='vehicle_id' column_invisible="parent.move_type not in ('in_invoice', 'in_refund')" required="need_vehicle and parent.move_type in ('in_invoice', 'in_refund')" optional='hidden'/>
            </xpath>
            <xpath expr="//field[@name='invoice_line_ids']//field[@name='account_id']" position="after">
                <field name='need_vehicle' column_invisible="True"/>
                <field name='vehicle_id' column_invisible="parent.move_type not in ('in_invoice', 'in_refund')" required="need_vehicle and parent.move_type in ('in_invoice', 'in_refund')" optional='hidden'/>
            </xpath>
        </field>
    </record>

    <record id="account_move_view_tree" model="ir.ui.view">
        <field name="name">account.move.list.inherit.fleet</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_tree"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='date']" position="attributes">
                <attribute name="string">Creation Date</attribute>
            </xpath>
            <xpath expr="//field[@name='date']" position="after">
                <field name="invoice_date" optional="show" readonly="state != 'draft'"/>
            </xpath>
        </field>
    </record>

    <record id="view_move_line_tree_fleet" model="ir.ui.view">
        <field name="name">view.move.line.list.fleet</field>
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

## File: views\fleet_vehicle_log_services_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id='fleet_vehicle_log_services_view_form' model='ir.ui.view'>
        <field name="name">fleet.vehicle.log.services.form.inherit.account</field>
        <field name="model">fleet.vehicle.log.services</field>
        <field name="inherit_id" ref="fleet.fleet_vehicle_log_services_view_form"/>
        <field name="priority" eval="1" />
        <field name="arch" type="xml">
            <xpath expr="//sheet" position="inside">
                <div class="oe_button_box" name="button_box">
                    <button
                        name="action_open_account_move"
                        type="object"
                        class="oe_stat_button"
                        icon="fa-pencil-square-o"
                        invisible="not account_move_line_id">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_text text-success"
                                invisible="account_move_state != 'posted'"
                                title="Service's Bill">Service's Bill</span>
                            <span class="o_stat_text text-warning"
                                invisible="account_move_state == 'posted'"
                                title="Service's Bill">Service's Bill</span>
                        </div>
                    </button>
                </div>
            </xpath>
            <xpath expr="//field[@name='amount']" position="attributes">
                <attribute name="readonly">account_move_line_id</attribute>
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
                    invisible="bill_count == 0"
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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_automatic_entry_wizard

```

