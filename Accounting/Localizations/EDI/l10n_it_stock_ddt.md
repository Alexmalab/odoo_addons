# Odoo Module: l10n_it_stock_ddt

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models
from odoo import api, SUPERUSER_ID


def _create_picking_seq(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    ptypes = env['stock.picking.type'].search([('code', '=', 'outgoing'), ('warehouse_id', '!=', False)])
    for ptype in ptypes:
        wh = ptype.warehouse_id
        ptype.l10n_it_ddt_sequence_id = env['ir.sequence'].create({
        'name': wh.name + ' ' + ' Sequence ' + ' ' + ptype.sequence_code,
        'prefix': wh.code + '/' + ptype.sequence_code + '/DDT', 'padding': 5,
        'company_id': wh.company_id.id,
        'implementation': 'no_gap',
    }).id
```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': "l10n_it_stock_ddt",
    'website': 'https://www.odoo.com',
    'category': 'Accounting/Localizations/EDI',
    'version': '0.1',
    'description': """
Documento di Trasporto (DDT)

Whenever goods are transferred between A and B, the DDT serves
as a legitimation e.g. when the police would stop you. 

When you want to print an outgoing picking in an Italian company, 
it will print you the DDT instead.  It is like the delivery 
slip, but it also contains the value of the product, 
the transportation reason, the carrier, ... which make it a DDT.  

We also use a separate sequence for the DDT as the number should not 
have any gaps and should only be applied at the moment the goods are sent. 

When invoices are related to their sale order and the sale order with the 
delivery, the system will automatically calculate the linked DDTs for every 
invoice line to export in the FatturaPA XML.   
    """,
    'depends': ['l10n_it_edi', 'delivery'],
    'data': [
        'report/l10n_it_ddt_report.xml',
        'views/stock_picking_views.xml',
        'views/account_invoice_views.xml',
        'data/l10n_it_ddt_template.xml',
    ],
    'auto_install': True,
    'post_init_hook': '_create_picking_seq',
    'license': 'LGPL-3',
}

```

## File: data\l10n_it_ddt_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
     <template id="my_view_name" inherit_id="l10n_it_edi.account_invoice_it_FatturaPA_export">
            <xpath expr='//DatiDDT' position="after">
                <t t-if="ddt_dict and not record.l10n_it_ddt_id">
                    <t t-foreach="ddt_dict" t-as="picking">
                        <DatiDDT>
                            <NumeroDDT t-esc="format_alphanumeric(picking.l10n_it_ddt_number[-20:])"/>
                            <DataDDT t-esc="format_date(picking.date_done)"/>
                            <t t-if="len(ddt_dict) > 1">
                                <t t-foreach="ddt_dict[picking]" t-as="line_ref">
                                    <RiferimentoNumeroLinea t-esc="line_ref"/>
                                </t>
                            </t>
                        </DatiDDT>
                    </t>
                </t>
            </xpath>
        </template>
</odoo>

```

## File: models\account_invoice.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api, fields, _
from odoo.tools.float_utils import float_compare


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_it_ddt_ids = fields.Many2many('stock.picking', compute="_compute_ddt_ids")
    l10n_it_ddt_count = fields.Integer(compute="_compute_ddt_ids")

    def _get_ddt_values(self):
        """
        We calculate the link between the invoice lines and the deliveries related to the invoice through the
        links with the sale order(s).  We assume that the first picking was invoiced first. (FIFO)
        :return: a dictionary with as key the picking and value the invoice line numbers (by counting)
        """
        self.ensure_one()
        # We don't consider returns/credit notes as we suppose they will lead to more deliveries/invoices as well
        if self.move_type != "out_invoice" or self.state != 'posted':
            return {}
        line_count = 0
        invoice_line_pickings = {}
        for line in self.invoice_line_ids.filtered(lambda l: not l.display_type):
            line_count += 1
            done_moves_related = line.sale_line_ids.mapped('move_ids').filtered(lambda m: m.state == 'done' and m.location_dest_id.usage == 'customer')
            if len(done_moves_related) <= 1:
                if done_moves_related and line_count not in invoice_line_pickings.get(done_moves_related.picking_id, []):
                    invoice_line_pickings.setdefault(done_moves_related.picking_id, []).append(line_count)
            else:
                total_invoices = done_moves_related.mapped('sale_line_id.invoice_lines').filtered(
                    lambda l: l.move_id.state == 'posted' and l.move_id.move_type == 'out_invoice').sorted(lambda l: (l.move_id.invoice_date, l.move_id.id))
                total_invs = [(i.product_uom_id._compute_quantity(i.quantity, i.product_id.uom_id), i) for i in total_invoices]
                inv = total_invs.pop(0)
                # Match all moves and related invoice lines FIFO looking for when the matched invoice_line matches line
                for move in done_moves_related.sorted(lambda m: (m.date, m.id)):
                    rounding = move.product_uom.rounding
                    move_qty = move.product_qty
                    while (float_compare(move_qty, 0, precision_rounding=rounding) > 0):
                        if float_compare(inv[0], move_qty, precision_rounding=rounding) > 0:
                            inv = (inv[0] - move_qty, inv[1])
                            invoice_line = inv[1]
                            move_qty = 0
                        if float_compare(inv[0], move_qty, precision_rounding=rounding) <= 0:
                            move_qty -= inv[0]
                            invoice_line = inv[1]
                            if total_invs:
                                inv = total_invs.pop(0)
                            else:
                                move_qty = 0 #abort when not enough matched invoices
                        # If in our FIFO iteration we stumble upon the line we were checking
                        if invoice_line == line and line_count not in invoice_line_pickings.get(move.picking_id, []):
                            invoice_line_pickings.setdefault(move.picking_id, []).append(line_count)
        return invoice_line_pickings

    @api.depends('invoice_line_ids', 'invoice_line_ids.sale_line_ids')
    def _compute_ddt_ids(self):
        it_out_invoices = self.filtered(lambda i: i.move_type == 'out_invoice' and i.company_id.country_id.code == 'IT')
        for invoice in it_out_invoices:
            invoice_line_pickings = invoice._get_ddt_values()
            pickings = self.env['stock.picking']
            for picking in invoice_line_pickings:
                pickings |= picking
            invoice.l10n_it_ddt_ids = pickings
            invoice.l10n_it_ddt_count = len(pickings)
        for invoice in self - it_out_invoices:
            invoice.l10n_it_ddt_ids = self.env['stock.picking']
            invoice.l10n_it_ddt_count = 0

    def get_linked_ddts(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'view_mode': 'tree,form',
            'name': _("Linked deliveries"),
            'res_model': 'stock.picking',
            'domain': [('id', 'in', self.l10n_it_ddt_ids.ids)],
        }

    def _prepare_fatturapa_export_values(self):
        template_values = super()._prepare_fatturapa_export_values()
        template_values['ddt_dict'] = self._get_ddt_values()
        return template_values

```

## File: models\stock_picking.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _


class StockPicking(models.Model):
    _inherit = "stock.picking"

    l10n_it_transport_reason = fields.Selection([('sale', 'Sale'),
                                                 ('outsourcing', 'Outsourcing'),
                                                 ('evaluation', 'Evaluation'),
                                                 ('gift', 'Gift'),
                                                 ('transfer', 'Transfer'),
                                                 ('substitution', 'Substitution'),
                                                 ('attemped_sale', 'Attempted Sale'),
                                                 ('loaned_use', 'Loaned for Use'),
                                                 ('repair', 'Repair')], default="sale", tracking=True, string='Transport Reason')
    l10n_it_transport_method = fields.Selection([('sender', 'Sender'), ('recipient', 'Recipient'), ('courier', 'Courier service')],
                                                default="sender", string='Transport Method')
    l10n_it_transport_method_details = fields.Char('Transport Note')
    l10n_it_parcels = fields.Integer(string="Parcels", default=1)
    l10n_it_country_code = fields.Char(related="company_id.country_id.code")
    l10n_it_ddt_number = fields.Char('DDT Number', readonly=True)
    l10n_it_show_print_ddt_button = fields.Boolean(compute="_compute_l10n_it_show_print_ddt_button")

    @api.depends('l10n_it_country_code',
                 'picking_type_code',
                 'state',
                 'is_locked',
                 'move_ids_without_package',
                 'move_ids_without_package.partner_id',
                 'location_id',
                 'location_dest_id')
    def _compute_l10n_it_show_print_ddt_button(self):
        # Enable printing the DDT for done outgoing shipments
        # or dropshipping (picking going from supplier to customer)
        for picking in self:
            picking.l10n_it_show_print_ddt_button = (
                picking.l10n_it_country_code == 'IT'
                and picking.state == 'done'
                and picking.is_locked
                and (picking.picking_type_code == 'outgoing'
                     or (
                         picking.move_ids_without_package
                         and picking.move_ids_without_package[0].partner_id
                         and picking.location_id.usage == 'supplier'
                         and picking.location_dest_id.usage == 'customer'
                         )
                     )
                )

    def _action_done(self):
        super(StockPicking, self)._action_done()
        for picking in self.filtered(lambda p: p.picking_type_id.l10n_it_ddt_sequence_id):
            picking.l10n_it_ddt_number = picking.picking_type_id.l10n_it_ddt_sequence_id.next_by_id()


class StockPickingType(models.Model):
    _inherit = 'stock.picking.type'

    l10n_it_ddt_sequence_id = fields.Many2one('ir.sequence')

    def _get_dtt_ir_seq_vals(self, warehouse_id, sequence_code):
        if warehouse_id:
            wh = self.env['stock.warehouse'].browse(warehouse_id)
            ir_seq_name = wh.name + ' ' + _('Sequence') + ' ' + sequence_code
            ir_seq_prefix = wh.code + '/' + sequence_code + '/DDT'
        else:
            ir_seq_name = _('Sequence') + ' ' + sequence_code
            ir_seq_prefix = sequence_code + '/DDT'
        return ir_seq_name, ir_seq_prefix

    @api.model
    def create(self, vals):
        company = self.env['res.company'].browse(vals.get('company_id', False)) or self.env.company
        if company.country_id.code == 'IT' and vals['code'] == 'outgoing' and ('l10n_it_ddt_sequence_id' not in vals or not vals['l10n_it_ddt_sequence_id']):
            ir_seq_name, ir_seq_prefix = self._get_dtt_ir_seq_vals(vals.get('warehouse_id'), vals['sequence_code'])
            vals['l10n_it_ddt_sequence_id'] = self.env['ir.sequence'].create({
                    'name': ir_seq_name,
                    'prefix': ir_seq_prefix,
                    'padding': 5,
                    'company_id': company.id,
                    'implementation': 'no_gap',
                }).id
        return super(StockPickingType, self).create(vals)

    def write(self, vals):
        if 'sequence_code' in vals:
            for picking_type in self.filtered(lambda p: p.l10n_it_ddt_sequence_id):
                warehouse = picking_type.warehouse_id.id if 'warehouse_id' not in vals else vals['warehouse_ids']
                ir_seq_name, ir_seq_prefix = self._get_dtt_ir_seq_vals(warehouse, vals['sequence_code'])
                picking_type.l10n_it_ddt_sequence_id.write({
                        'name': ir_seq_name,
                        'prefix': ir_seq_prefix,
                    })
        return super(StockPickingType, self).write(vals)
```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_picking
from . import account_invoice
```

## File: report\l10n_it_ddt_report.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <template id="report_ddt_view">
        <t t-call="web.external_layout">
            <t t-set="o" t-value="o.with_context(lang=lang)"/>
            <t t-if="o.move_ids_without_package and o.move_ids_without_package[0].partner_id and o.location_dest_id.usage == 'customer' and o.location_id.usage == 'supplier'">
              <t t-set="delivery_from" t-value="o.partner_id"/>
              <t t-set="delivery_to" t-value="o.move_ids_without_package[0].partner_id"/>
            </t>
            <t t-elif="o.picking_type_id.warehouse_id.partner_id">
              <t t-set="delivery_from" t-value="o.picking_type_id.warehouse_id.partner_id"/>
              <t t-set="delivery_to" t-value="o.partner_id"/>
            </t>
            <t t-else="">
              <t t-set="delivery_from" t-value="o.company_id.partner_id"/>
              <t t-set="delivery_to" t-value="o.partner_id"/>
            </t>
            <div class="page">
                <div class="row">
                    <div class="col-6">
                       <span><strong>Warehouse Address:</strong></span>
                        <div t-esc="delivery_from"
                        t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True, "phone_icons": True}'/>
                        <p t-if="delivery_from.vat">VAT: <span t-field="delivery_from.vat"/></p>
                    </div>
                    <div class="col-5 offset-1">
                        <div>
                            <span><strong>Customer Address:</strong></span>
                            <div t-esc="delivery_to"
                                   t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True, "phone_icons": True}'/>
                            <p t-if="delivery_to.vat">
                                <t t-set="default_vat_label">VAT</t>
                                <t t-esc="delivery_to.country_id.vat_label or default_vat_label"/>: <span t-esc="delivery_to.vat"/>
                            </p>
                        </div>
                    </div>
                </div>
                <div class="mt16"/>
                <div class="mt64"/>
                <div>
                    <h1>Documento di Trasporto <span t-esc="o.l10n_it_ddt_number"/></h1>
                </div>
                <div class="clearfix"/>
                <div class="mb32"/>
                <div class="row">
                    <div class="col-6">
                        <table class="table table-bordered">
                            <tbody>
                                <tr>
                                    <td>Transportation Reason</td>
                                    <td><span t-field="o.l10n_it_transport_reason"/></td>
                                </tr>
                                <tr>
                                    <td>Transportation Method</td>
                                    <td><span t-field="o.l10n_it_transport_method"/></td>
                                </tr>
                                <tr>
                                    <td>Carrier Condition</td>
                                    <td><span t-field="o.sale_id.incoterm.name"/></td>
                                </tr>
                                <tr>
                                    <td>Carrier</td>
                                    <td><span t-field="o.carrier_id"/></td>
                                </tr>
                            </tbody>
                        </table>
                        <div t-if="o.l10n_it_transport_method_details">
                            <b>Transportation Method Details: </b>
                            <span t-field="o.l10n_it_transport_method_details"/>
                        </div>
                    </div>
                    <div class="col-5 offset-1">
                        <table class="table table-bordered">
                            <tbody>
                                <tr>
                                    <td>Order</td>
                                    <td><span t-field="o.origin"/></td>
                                </tr>
                                <tr>
                                    <td>Picking Number</td>
                                    <td><span t-field="o.name"/></td>
                                </tr>
                                <tr>
                                    <td>Shipping Date</td>
                                    <td><span t-field="o.date_done"/></td>
                                </tr>
                                <tr>
                                    <td>Gross Weight (kg)</td>
                                    <td><span t-field="o.shipping_weight"/></td>
                                </tr>
                                <tr>
                                    <td>Parcels</td>
                                    <td><span t-field="o.l10n_it_parcels"/></td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>

                <div class="mt64"/>
                <div t-if="o.note"><b>Note:</b> <span t-field="o.note"/></div>

                <div class="mt64"/>
                <div class="mt64"/>

                <table class="table table-sm" name="document_details">
                    <thead>
                        <tr>
                            <th><strong>Product</strong></th>
                            <th><strong>Quantity</strong></th>
                            <th><strong>Total Value</strong></th>
                        </tr>
                    </thead>
                    <tbody>
                        <t t-set="total_value" t-value="0"/>
                        <t t-foreach="o.move_lines" t-as="move">
                            <t t-if="move.quantity_done">
                                <tr>
                                    <td>
                                        <span t-field="move.product_id"/>
                                    </td>
                                    <td>
                                        <span t-field="move.quantity_done"/>
                                        <span t-field="move.product_uom" groups="uom.group_uom"/>
                                    </td>
                                    <td>
                                        <t t-if="move.sale_line_id">
                                            <t t-set="lst_price" t-value="move.sale_line_id.price_unit * move.quantity_done"/>
                                        </t>
                                        <t t-else="">
                                            <t t-set="lst_price" t-value="move.product_id.lst_price * move.product_uom._compute_quantity(move.quantity_done, move.product_id.uom_id)"/>
                                        </t>
                                        <span t-esc="lst_price"  t-options='{"widget": "monetary", "display_currency": o.company_id.currency_id}'/>
                                        <t t-set="total_value" t-value="total_value + lst_price"/>
                                    </td>
                                </tr>
                            </t>
                        </t>
                        <tr>
                            <td>
                            </td>
                            <td style="text-align:right">
                                <b>Total:</b>
                            </td>
                            <td>
                                <span t-esc="total_value"  t-options='{"widget": "monetary", "display_currency": o.company_id.currency_id}'/>
                            </td>
                        </tr>
                    </tbody>
                </table>
                <div class="mt64"/>
                <div class="mt64"/>
                <table class="table table-sm">
                    <thead>
                        <tr>
                            <th><div class="row"><span class="fa fa-pencil mt4"></span><div class="ml4"/><strong>Company Signature</strong></div></th>
                            <th><div class="row"><span class="fa fa-pencil mt4"></span><div class="ml4"/><strong>Carrier Signature</strong></div></th>
                            <th><div class="row"><span class="fa fa-pencil mt4"></span><div class="ml4"/><strong>Customer Signature</strong></div></th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td>
                                <div class="col">
                                </div>
                            </td>
                            <td>
                                <div class="col">
                                </div>
                            </td>
                            <td>
                                <div class="col">
                                </div>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </t>
    </template>

    <template id="report_ddt">
        <t t-call="web.html_container">
            <t t-foreach="docs" t-as="o">
                <t t-set="lang" t-value="o._get_report_lang()"/>
                <t t-call="l10n_it_stock_ddt.report_ddt_view" t-lang="lang"/>
            </t>
        </t>
    </template>

    <record id="action_report_ddt" model="ir.actions.report">
        <field name="name">DDT report</field>
        <field name="model">stock.picking</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">l10n_it_stock_ddt.report_ddt</field>
        <field name="report_file">report_ddt</field>
        <field name="print_report_name">'DDT - %s - %s' % (object.partner_id.name or '', object.l10n_it_ddt_number)</field>
    </record>
</odoo>

```

## File: views\account_invoice_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="account_invoice_view_form_inherit_ddt" model="ir.ui.view">
        <field name="name">account.invoice.form.inherit.ddt</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <button name="get_linked_ddts" type="object" class="oe_stat_button" icon="fa-calendar" attrs="{'invisible':[('l10n_it_ddt_count','=', 0)]}">
                    <field name="l10n_it_ddt_count" widget="statinfo" string="DDTs"/>
                </button>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\stock_picking_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
   <record id="view_picking_form_inherit_l10n_it_ddt" model="ir.ui.view">
        <field name="name">stock.picking.form.l10n.it.ddt</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='do_print_picking']" position="after">
                <field name="l10n_it_country_code" invisible="1"/>
                <field name="l10n_it_show_print_ddt_button" invisible="1"/>
                <button name="%(l10n_it_stock_ddt.action_report_ddt)d" type="action" string="Print"
                        attrs="{'invisible': [('l10n_it_show_print_ddt_button', '=', False)]}"
                        groups="base.group_user"/>
            </xpath>
            <xpath expr="//button[@name='%(stock.action_report_delivery)d']" position="attributes">
                <attribute name="attrs">{'invisible': ['|', ('l10n_it_show_print_ddt_button', '=', True), '&amp;', ('picking_type_code', '=', 'outgoing'), ('l10n_it_country_code', '=', 'IT')]}</attribute>
            </xpath>
            <group name='carrier_data' position="after">
                <group string="DDT Information" attrs="{'invisible': ['|', ('l10n_it_country_code', '!=', 'IT'), ('picking_type_code', '!=', 'outgoing')]}">
                    <field name="l10n_it_country_code" invisible="1"/>
                    <field name="l10n_it_ddt_number"/>
                    <field name="l10n_it_transport_reason"/>
                    <field name="l10n_it_transport_method"/>
                    <field name="l10n_it_transport_method_details"/>
                    <field name="l10n_it_parcels"/>
                </group>
            </group>
        </field>
   </record>

    <record id="view_picking_search_inherit_l10n_it_ddt" model="ir.ui.view">
        <field name="name">stock.picking.search.l10n.it.ddt</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.view_picking_internal_search"/>
        <field name="arch" type="xml">
            <field name="origin" position="after">
                <field name="l10n_it_ddt_number"/>
            </field>
        </field>
    </record>

    <record id="view_picking_tree_inherit_l10n_it_ddt" model="ir.ui.view">
        <field name="name">stock.picking.tree.l10n.it.ddt</field>
        <field name="model">stock.picking</field>
        <field name="inherit_id" ref="stock.vpicktree"/>
        <field name="arch" type="xml">
            <field name="origin" position="after">
                <field name="l10n_it_ddt_number" optional="hide"/>
            </field>
        </field>
    </record>
</odoo>

```

