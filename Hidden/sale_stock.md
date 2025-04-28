# Odoo Module: sale_stock

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import report
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Sales and Warehouse Management',
    'version': '1.0',
    'category': 'Hidden',
    'summary': 'Quotation, Sales Orders, Delivery & Invoicing Control',
    'description': """
Manage sales quotations and orders
==================================

This module makes the link between the sales and warehouses management applications.

Preferences
-----------
* Shipping: Choice of delivery at once or partial delivery
* Invoicing: choose how invoices will be paid
* Incoterms: International Commercial terms

""",
    'depends': ['sale', 'stock_account'],
    'data': [
        'security/sale_stock_security.xml',
        'security/ir.model.access.csv',
        'views/sale_order_views.xml',
        'views/stock_views.xml',
        'views/res_config_settings_views.xml',
        'views/sale_stock_portal_template.xml',
        'views/stock_production_lot_views.xml',
        'views/report_invoice.xml',
        'report/sale_order_report_templates.xml',
        'report/stock_report_deliveryslip.xml',
        'data/sale_stock_data.xml',
        'wizard/stock_rules_report_views.xml',
    ],
    'demo': ['data/sale_order_demo.xml'],
    'qweb': ['static/src/xml/qty_at_date.xml'],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import exceptions
from odoo.addons.sale.controllers.portal import CustomerPortal
from odoo.http import request, route
from odoo.tools import consteq


class SaleStockPortal(CustomerPortal):

    def _stock_picking_check_access(self, picking_id, access_token=None):
        picking = request.env['stock.picking'].browse([picking_id])
        picking_sudo = picking.sudo()
        try:
            picking.check_access_rights('read')
            picking.check_access_rule('read')
        except exceptions.AccessError:
            if not access_token or not consteq(picking_sudo.sale_id.access_token, access_token):
                raise
        return picking_sudo

    @route(['/my/picking/pdf/<int:picking_id>'], type='http', auth="public", website=True)
    def portal_my_picking_report(self, picking_id, access_token=None, **kw):
        """ Print delivery slip for customer, using either access rights or access token
        to be sure customer has access """
        try:
            picking_sudo = self._stock_picking_check_access(picking_id, access_token=access_token)
        except exceptions.AccessError:
            return request.redirect('/my')

        # print report as sudo, since it require access to product, taxes, payment term etc.. and portal does not have those access rights.
        pdf = request.env.ref('stock.action_report_delivery').sudo().render_qweb_pdf([picking_sudo.id])[0]
        pdfhttpheaders = [
            ('Content-Type', 'application/pdf'),
            ('Content-Length', len(pdf)),
        ]
        return request.make_response(pdf, headers=pdfhttpheaders)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\sale_order_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
    
        <record id="sale.sale_order_1" model="sale.order">
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>
        
        <record id="sale.sale_order_2" model="sale.order">
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>
        
        <record id="sale.sale_order_3" model="sale.order">
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>

        <record id="sale.sale_order_5" model="sale.order">
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>

        <record id="sale.sale_order_6" model="sale.order">
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>

        <record id="sale.sale_order_8" model="sale.order">
            <field name="warehouse_id" ref="stock.warehouse0"/>
        </record>

        <record id="stock_inventory_customizable_desk_update" model="stock.inventory">
            <field name="name">Inventory for new Customizable Desks</field>
        </record>

        <record id="stock_inventory_line_7e" model="stock.inventory.line">
            <field name="product_id" ref="sale.product_product_4e"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="inventory_id" ref="stock_inventory_customizable_desk_update"/>
            <field name="product_qty">65.0</field>
            <field name="location_id" ref="stock.stock_location_components"/>
        </record>
        <record id="stock_inventory_line_7f" model="stock.inventory.line">
            <field name="product_id" ref="sale.product_product_4f"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="inventory_id" ref="stock_inventory_customizable_desk_update"/>
            <field name="product_qty">70.0</field>
            <field name="location_id" ref="stock.stock_location_components"/>
        </record>

        <function model="stock.inventory" name="action_start">
            <function eval="[[('state','=','draft'),('id', '=', ref('stock_inventory_customizable_desk_update'))]]" model="stock.inventory" name="search"/>
        </function>
        <function model="stock.inventory" name="action_validate">
            <function eval="[[('state','=','confirm'),('id', '=', ref('stock_inventory_customizable_desk_update'))]]" model="stock.inventory" name="search"/>
        </function>
    </data>
</odoo>

```

## File: data\sale_stock_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
    	<record id="stock.route_warehouse0_mto" model="stock.location.route">
            <field name="sale_selectable">True</field>
        </record>
    </data>
</odoo>

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import fields, models
from odoo.tools import float_is_zero, float_compare
from odoo.tools.misc import formatLang


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _stock_account_get_last_step_stock_moves(self):
        """ Overridden from stock_account.
        Returns the stock moves associated to this invoice."""
        rslt = super(AccountMove, self)._stock_account_get_last_step_stock_moves()
        for invoice in self.filtered(lambda x: x.type == 'out_invoice'):
            rslt += invoice.mapped('invoice_line_ids.sale_line_ids.move_ids').filtered(lambda x: x.state == 'done' and x.location_dest_id.usage == 'customer')
        for invoice in self.filtered(lambda x: x.type == 'out_refund'):
            rslt += invoice.mapped('reversed_entry_id.invoice_line_ids.sale_line_ids.move_ids').filtered(lambda x: x.state == 'done' and x.location_id.usage == 'customer')
            # Add refunds generated from the SO
            rslt += invoice.mapped('invoice_line_ids.sale_line_ids.move_ids').filtered(lambda x: x.state == 'done' and x.location_id.usage == 'customer')
        return rslt

    def _get_invoiced_lot_values(self):
        """ Get and prepare data to show a table of invoiced lot on the invoice's report. """
        self.ensure_one()
        if self.state == 'draft' or not self.invoice_date or self.type not in ('out_invoice', 'out_refund'):
            return []

        current_invoice_amls = self.invoice_line_ids.filtered(lambda aml: not aml.display_type and aml.product_id and aml.product_id.type in ('consu', 'product') and aml.quantity)
        all_invoices_amls = current_invoice_amls.sale_line_ids.invoice_lines.filtered(lambda aml: aml.move_id.state == 'posted').sorted(lambda aml: (aml.date, aml.move_name, aml.id))
        index = all_invoices_amls.ids.index(current_invoice_amls[:1].id) if current_invoice_amls[:1] in all_invoices_amls else 0
        previous_amls = all_invoices_amls[:index]

        previous_qties_invoiced = previous_amls._get_invoiced_qty_per_product()
        invoiced_qties = current_invoice_amls._get_invoiced_qty_per_product()
        invoiced_products = invoiced_qties.keys()

        qties_per_lot = defaultdict(float)
        previous_qties_delivered = defaultdict(float)
        stock_move_lines = current_invoice_amls.sale_line_ids.move_ids.move_line_ids.filtered(lambda sml: sml.state == 'done' and sml.lot_id).sorted(lambda sml: (sml.date, sml.id))
        for sml in stock_move_lines:
            if sml.product_id not in invoiced_products or 'customer' not in {sml.location_id.usage, sml.location_dest_id.usage}:
                continue
            product = sml.product_id
            product_uom = product.uom_id
            qty_done = sml.product_uom_id._compute_quantity(sml.qty_done, product_uom)

            if sml.location_id.usage == 'customer':
                returned_qty = min(qties_per_lot[sml.lot_id], qty_done)
                qties_per_lot[sml.lot_id] -= returned_qty
                qty_done = returned_qty - qty_done

            previous_qty_invoiced = previous_qties_invoiced[product]
            previous_qty_delivered = previous_qties_delivered[product]
            # If we return more than currently delivered (i.e., qty_done < 0), we remove the surplus
            # from the previously delivered (and qty_done becomes zero). If it's a delivery, we first
            # try to reach the previous_qty_invoiced
            if float_compare(qty_done, 0, precision_rounding=product_uom.rounding) < 0 or \
                    float_compare(previous_qty_delivered, previous_qty_invoiced, precision_rounding=product_uom.rounding) < 0:
                previously_done = qty_done if sml.location_id.usage == 'customer' else min(previous_qty_invoiced - previous_qty_delivered, qty_done)
                previous_qties_delivered[product] += previously_done
                qty_done -= previously_done

            qties_per_lot[sml.lot_id] += qty_done

        lot_values = []
        for lot, qty in qties_per_lot.items():
            lot_sudo = lot.sudo()
            if float_is_zero(invoiced_qties[lot_sudo.product_id], precision_rounding=lot_sudo.product_uom_id.rounding) \
                    or float_compare(qty, 0, precision_rounding=lot_sudo.product_uom_id.rounding) <= 0:
                continue
            invoiced_lot_qty = min(qty, invoiced_qties[lot_sudo.product_id])
            invoiced_qties[lot_sudo.product_id] -= invoiced_lot_qty
            lot_values.append({
                'product_name': lot_sudo.product_id.display_name,
                'quantity': formatLang(self.env, invoiced_lot_qty, dp='Product Unit of Measure'),
                'uom_name': lot_sudo.product_uom_id.name,
                'lot_name': lot_sudo.name,
                # The lot id is needed by localizations to inherit the method and add custom fields on the invoice's report.
                'lot_id': lot_sudo.id,
            })

        return lot_values


class AccountMoveLine(models.Model):
    _inherit = "account.move.line"

    def _sale_can_be_reinvoice(self):
        self.ensure_one()
        return not self.is_anglo_saxon_line and super(AccountMoveLine, self)._sale_can_be_reinvoice()


    def _stock_account_get_anglo_saxon_price_unit(self):
        self.ensure_one()
        price_unit = super(AccountMoveLine, self)._stock_account_get_anglo_saxon_price_unit()

        so_line = self.sale_line_ids and self.sale_line_ids[-1] or False
        if so_line:
            is_line_reversing = bool(self.move_id.reversed_entry_id)
            qty_to_invoice = self.product_uom_id._compute_quantity(self.quantity, self.product_id.uom_id)
            posted_invoice_lines = so_line.invoice_lines.filtered(lambda l: l.move_id.state == 'posted' and bool(l.move_id.reversed_entry_id) == is_line_reversing)
            qty_invoiced = sum([x.product_uom_id._compute_quantity(x.quantity, x.product_id.uom_id) for x in posted_invoice_lines])

            average_price_unit = self.product_id.with_context(force_company=self.company_id.id, is_returned=is_line_reversing)._compute_average_price(qty_invoiced, qty_to_invoice, so_line.move_ids)
            price_unit = average_price_unit or price_unit
            price_unit = self.product_id.uom_id.with_context(force_company=self.company_id.id)._compute_price(price_unit, self.product_uom_id)
        return price_unit

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    @api.onchange('type')
    def _onchange_type(self):
        """ We want to prevent storable product to be expensed, since it make no sense as when confirm
            expenses, the product is already out of our stock.
        """
        res = super(ProductTemplate, self)._onchange_type()
        if self.type == 'product':
            self.expense_policy = 'no'
            self.service_type = 'manual'
        return res

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class company(models.Model):
    _inherit = 'res.company'

    security_lead = fields.Float(
        'Sales Safety Days', default=0.0, required=True,
        help="Margin of error for dates promised to customers. "
             "Products will be scheduled for procurement and delivery "
             "that many days earlier than the actual promised date, to "
             "cope with unexpected delays in the supply chain.")

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    security_lead = fields.Float(related='company_id.security_lead', string="Security Lead Time", readonly=False)
    group_display_incoterm = fields.Boolean("Incoterms", implied_group='sale_stock.group_display_incoterm')
    group_lot_on_invoice = fields.Boolean("Display Lots & Serial Numbers on Invoices",
        implied_group='sale_stock.group_lot_on_invoice')
    use_security_lead = fields.Boolean(
        string="Security Lead Time for Sales",
        config_parameter='sale_stock.use_security_lead',
        help="Margin of error for dates promised to customers. Products will be scheduled for delivery that many days earlier than the actual promised date, to cope with unexpected delays in the supply chain.")
    default_picking_policy = fields.Selection([
        ('direct', 'Ship products as soon as available, with back orders'),
        ('one', 'Ship all products at once')
        ], "Picking Policy", default='direct', default_model="sale.order", required=True)

    @api.onchange('use_security_lead')
    def _onchange_use_security_lead(self):
        if not self.use_security_lead:
            self.security_lead = 0.0

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, timedelta
from collections import defaultdict

from odoo import api, fields, models, _
from odoo.tools import DEFAULT_SERVER_DATETIME_FORMAT, float_compare, float_round
from odoo.exceptions import UserError


class SaleOrder(models.Model):
    _inherit = "sale.order"

    @api.model
    def _default_warehouse_id(self):
        company = self.env.company.id
        warehouse_ids = self.env['stock.warehouse'].search([('company_id', '=', company)], limit=1)
        return warehouse_ids

    incoterm = fields.Many2one(
        'account.incoterms', 'Incoterm',
        help="International Commercial Terms are a series of predefined commercial terms used in international transactions.")
    picking_policy = fields.Selection([
        ('direct', 'As soon as possible'),
        ('one', 'When all products are ready')],
        string='Shipping Policy', required=True, readonly=True, default='direct',
        states={'draft': [('readonly', False)], 'sent': [('readonly', False)]}
        ,help="If you deliver all products at once, the delivery order will be scheduled based on the greatest "
        "product lead time. Otherwise, it will be based on the shortest.")
    warehouse_id = fields.Many2one(
        'stock.warehouse', string='Warehouse',
        required=True, readonly=True, states={'draft': [('readonly', False)], 'sent': [('readonly', False)]},
        default=_default_warehouse_id, check_company=True)
    picking_ids = fields.One2many('stock.picking', 'sale_id', string='Transfers')
    delivery_count = fields.Integer(string='Delivery Orders', compute='_compute_picking_ids')
    procurement_group_id = fields.Many2one('procurement.group', 'Procurement Group', copy=False)
    effective_date = fields.Date("Effective Date", compute='_compute_effective_date', store=True, help="Completion date of the first delivery order.")
    expected_date = fields.Datetime( help="Delivery date you can promise to the customer, computed from the minimum lead time of "
                                          "the order lines in case of Service products. In case of shipping, the shipping policy of "
                                          "the order will be taken into account to either use the minimum or maximum lead time of "
                                          "the order lines.")

    @api.depends('picking_ids.date_done')
    def _compute_effective_date(self):
        for order in self:
            pickings = order.picking_ids.filtered(lambda x: x.state == 'done' and x.location_dest_id.usage == 'customer')
            dates_list = [date for date in pickings.mapped('date_done') if date]
            order.effective_date = fields.Date.context_today(order, min(dates_list)) if dates_list else False

    @api.depends('picking_policy')
    def _compute_expected_date(self):
        super(SaleOrder, self)._compute_expected_date()
        for order in self:
            dates_list = []
            for line in order.order_line.filtered(lambda x: x.state != 'cancel' and not x._is_delivery() and not x.display_type):
                dt = line._expected_date()
                dates_list.append(dt)
            if dates_list:
                expected_date = min(dates_list) if order.picking_policy == 'direct' else max(dates_list)
                order.expected_date = fields.Datetime.to_string(expected_date)

    @api.model
    def create(self, vals):
        if 'warehouse_id' not in vals and 'company_id' in vals and vals.get('company_id') != self.env.company.id:
            vals['warehouse_id'] = self.env['stock.warehouse'].search([('company_id', '=', vals.get('company_id'))], limit=1).id
        return super(SaleOrder, self).create(vals)

    def write(self, values):
        if values.get('order_line') and self.state == 'sale':
            for order in self:
                pre_order_line_qty = {order_line: order_line.product_uom_qty for order_line in order.mapped('order_line') if not order_line.is_expense}

        if values.get('partner_shipping_id'):
            new_partner = self.env['res.partner'].browse(values.get('partner_shipping_id'))
            for record in self:
                picking = record.mapped('picking_ids').filtered(lambda x: x.state not in ('done', 'cancel'))
                addresses = (record.partner_shipping_id.display_name, new_partner.display_name)
                message = _("""The delivery address has been changed on the Sales Order<br/>
                        From <strong>"%s"</strong> To <strong>"%s"</strong>,
                        You should probably update the partner on this document.""") % addresses
                picking.activity_schedule('mail.mail_activity_data_warning', note=message, user_id=self.env.user.id)

        res = super(SaleOrder, self).write(values)
        if values.get('order_line') and self.state == 'sale':
            rounding = self.env['decimal.precision'].precision_get('Product Unit of Measure')
            for order in self:
                to_log = {}
                for order_line in order.order_line:
                    if order_line.display_type:
                        continue
                    if float_compare(order_line.product_uom_qty, pre_order_line_qty.get(order_line, 0.0), precision_rounding=order_line.product_uom.rounding or rounding) < 0:
                        to_log[order_line] = (order_line.product_uom_qty, pre_order_line_qty.get(order_line, 0.0))
                if to_log:
                    documents = self.env['stock.picking']._log_activity_get_documents(to_log, 'move_ids', 'UP')
                    documents = {k:v for k, v in documents.items() if k[0].state != 'cancel'}
                    order._log_decrease_ordered_quantity(documents)
        return res

    def _action_confirm(self):
        self.order_line._action_launch_stock_rule()
        return super(SaleOrder, self)._action_confirm()

    @api.depends('picking_ids')
    def _compute_picking_ids(self):
        for order in self:
            order.delivery_count = len(order.picking_ids)

    @api.onchange('company_id')
    def _onchange_company_id(self):
        if self.company_id:
            warehouse_id = self.env['ir.default'].get_model_defaults('sale.order').get('warehouse_id')
            self.warehouse_id = warehouse_id or self.env['stock.warehouse'].search([('company_id', '=', self.company_id.id)], limit=1)

    @api.onchange('partner_shipping_id')
    def _onchange_partner_shipping_id(self):
        res = {}
        pickings = self.picking_ids.filtered(
            lambda p: p.state not in ['done', 'cancel'] and p.partner_id != self.partner_shipping_id
        )
        if pickings:
            res['warning'] = {
                'title': _('Warning!'),
                'message': _(
                    'Do not forget to change the partner on the following delivery orders: %s'
                ) % (','.join(pickings.mapped('name')))
            }
        return res

    def action_view_delivery(self):
        '''
        This function returns an action that display existing delivery orders
        of given sales order ids. It can either be a in a list or in a form
        view, if there is only one delivery order to show.
        '''
        action = self.env.ref('stock.action_picking_tree_all').read()[0]

        pickings = self.mapped('picking_ids')
        if len(pickings) > 1:
            action['domain'] = [('id', 'in', pickings.ids)]
        elif pickings:
            form_view = [(self.env.ref('stock.view_picking_form').id, 'form')]
            if 'views' in action:
                action['views'] = form_view + [(state,view) for state,view in action['views'] if view != 'form']
            else:
                action['views'] = form_view
            action['res_id'] = pickings.id
        # Prepare the context.
        picking_id = pickings.filtered(lambda l: l.picking_type_id.code == 'outgoing')
        if picking_id:
            picking_id = picking_id[0]
        else:
            picking_id = pickings[0]
        action['context'] = dict(self._context, default_partner_id=self.partner_id.id, default_picking_type_id=picking_id.picking_type_id.id, default_origin=self.name, default_group_id=picking_id.group_id.id)
        return action

    def action_cancel(self):
        documents = None
        for sale_order in self:
            if sale_order.state == 'sale' and sale_order.order_line:
                sale_order_lines_quantities = {order_line: (order_line.product_uom_qty, 0) for order_line in sale_order.order_line}
                documents = self.env['stock.picking']._log_activity_get_documents(sale_order_lines_quantities, 'move_ids', 'UP')
        self.mapped('picking_ids').filtered(lambda p: p.state != 'done').action_cancel()
        if documents:
            filtered_documents = {}
            for (parent, responsible), rendering_context in documents.items():
                if parent._name == 'stock.picking':
                    if parent.state == 'cancel':
                        continue
                filtered_documents[(parent, responsible)] = rendering_context
            self._log_decrease_ordered_quantity(filtered_documents, cancel=True)
        return super(SaleOrder, self).action_cancel()

    def _prepare_invoice(self):
        invoice_vals = super(SaleOrder, self)._prepare_invoice()
        invoice_vals['invoice_incoterm_id'] = self.incoterm.id
        return invoice_vals

    @api.model
    def _get_customer_lead(self, product_tmpl_id):
        super(SaleOrder, self)._get_customer_lead(product_tmpl_id)
        return product_tmpl_id.sale_delay

    def _log_decrease_ordered_quantity(self, documents, cancel=False):

        def _render_note_exception_quantity_so(rendering_context):
            order_exceptions, visited_moves = rendering_context
            visited_moves = list(visited_moves)
            visited_moves = self.env[visited_moves[0]._name].concat(*visited_moves)
            order_line_ids = self.env['sale.order.line'].browse([order_line.id for order in order_exceptions.values() for order_line in order[0]])
            sale_order_ids = order_line_ids.mapped('order_id')
            impacted_pickings = visited_moves.filtered(lambda m: m.state not in ('done', 'cancel')).mapped('picking_id')
            values = {
                'sale_order_ids': sale_order_ids,
                'order_exceptions': order_exceptions.values(),
                'impacted_pickings': impacted_pickings,
                'cancel': cancel
            }
            return self.env.ref('sale_stock.exception_on_so').render(values=values)

        self.env['stock.picking']._log_activity(_render_note_exception_quantity_so, documents)


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    qty_delivered_method = fields.Selection(selection_add=[('stock_move', 'Stock Moves')])
    product_packaging = fields.Many2one( 'product.packaging', string='Package', default=False, check_company=True)
    route_id = fields.Many2one('stock.location.route', string='Route', domain=[('sale_selectable', '=', True)], ondelete='restrict', check_company=True)
    move_ids = fields.One2many('stock.move', 'sale_line_id', string='Stock Moves')
    product_type = fields.Selection(related='product_id.type')
    virtual_available_at_date = fields.Float(compute='_compute_qty_at_date')
    scheduled_date = fields.Datetime(compute='_compute_qty_at_date')
    free_qty_today = fields.Float(compute='_compute_qty_at_date')
    qty_available_today = fields.Float(compute='_compute_qty_at_date')
    warehouse_id = fields.Many2one('stock.warehouse', compute='_compute_qty_at_date')
    qty_to_deliver = fields.Float(compute='_compute_qty_to_deliver')
    is_mto = fields.Boolean(compute='_compute_is_mto')
    display_qty_widget = fields.Boolean(compute='_compute_qty_to_deliver')

    @api.depends('product_id', 'product_uom_qty', 'qty_delivered', 'state', 'product_uom')
    def _compute_qty_to_deliver(self):
        """Compute the visibility of the inventory widget."""
        for line in self:
            line.qty_to_deliver = line.product_uom_qty - line.qty_delivered
            if line.state == 'draft' and line.product_type == 'product' and line.product_uom and line.qty_to_deliver > 0:
                line.display_qty_widget = True
            else:
                line.display_qty_widget = False

    @api.depends('product_id', 'customer_lead', 'product_uom_qty', 'product_uom', 'order_id.warehouse_id', 'order_id.commitment_date')
    def _compute_qty_at_date(self):
        """ Compute the quantity forecasted of product at delivery date. There are
        two cases:
         1. The quotation has a commitment_date, we take it as delivery date
         2. The quotation hasn't commitment_date, we compute the estimated delivery
            date based on lead time"""
        qty_processed_per_product = defaultdict(lambda: 0)
        grouped_lines = defaultdict(lambda: self.env['sale.order.line'])
        # We first loop over the SO lines to group them by warehouse and schedule
        # date in order to batch the read of the quantities computed field.
        for line in self:
            if not (line.product_id and line.display_qty_widget):
                continue
            line.warehouse_id = line.order_id.warehouse_id
            if line.order_id.commitment_date:
                date = line.order_id.commitment_date
            else:
                date = line._expected_date()
            grouped_lines[(line.warehouse_id.id, date)] |= line

        treated = self.browse()
        for (warehouse, scheduled_date), lines in grouped_lines.items():
            product_qties = lines.mapped('product_id').with_context(to_date=scheduled_date, warehouse=warehouse).read([
                'qty_available',
                'free_qty',
                'virtual_available',
            ])
            qties_per_product = {
                product['id']: (product['qty_available'], product['free_qty'], product['virtual_available'])
                for product in product_qties
            }
            for line in lines:
                line.scheduled_date = scheduled_date
                qty_available_today, free_qty_today, virtual_available_at_date = qties_per_product[line.product_id.id]
                line.qty_available_today = qty_available_today - qty_processed_per_product[line.product_id.id]
                line.free_qty_today = free_qty_today - qty_processed_per_product[line.product_id.id]
                line.virtual_available_at_date = virtual_available_at_date - qty_processed_per_product[line.product_id.id]
                if line.product_uom and line.product_id.uom_id and line.product_uom != line.product_id.uom_id:
                    line.qty_available_today = line.product_id.uom_id._compute_quantity(line.qty_available_today, line.product_uom)
                    line.free_qty_today = line.product_id.uom_id._compute_quantity(line.free_qty_today, line.product_uom)
                    line.virtual_available_at_date = line.product_id.uom_id._compute_quantity(line.virtual_available_at_date, line.product_uom)
                qty_processed_per_product[line.product_id.id] += line.product_uom_qty
            treated |= lines
        remaining = (self - treated)
        remaining.virtual_available_at_date = False
        remaining.scheduled_date = False
        remaining.free_qty_today = False
        remaining.qty_available_today = False
        remaining.warehouse_id = False

    @api.depends('product_id', 'route_id', 'order_id.warehouse_id', 'product_id.route_ids')
    def _compute_is_mto(self):
        """ Verify the route of the product based on the warehouse
            set 'is_available' at True if the product availibility in stock does
            not need to be verified, which is the case in MTO, Cross-Dock or Drop-Shipping
        """
        self.is_mto = False
        for line in self:
            if not line.display_qty_widget:
                continue
            product = line.product_id
            product_routes = line.route_id or (product.route_ids + product.categ_id.total_route_ids)

            # Check MTO
            mto_route = line.order_id.warehouse_id.mto_pull_id.route_id
            if not mto_route:
                try:
                    mto_route = self.env['stock.warehouse']._find_global_route('stock.route_warehouse0_mto', _('Make To Order'))
                except UserError:
                    # if route MTO not found in ir_model_data, we treat the product as in MTS
                    pass

            if mto_route and mto_route in product_routes:
                line.is_mto = True
            else:
                line.is_mto = False

    @api.depends('product_id')
    def _compute_qty_delivered_method(self):
        """ Stock module compute delivered qty for product [('type', 'in', ['consu', 'product'])]
            For SO line coming from expense, no picking should be generate: we don't manage stock for
            thoses lines, even if the product is a storable.
        """
        super(SaleOrderLine, self)._compute_qty_delivered_method()

        for line in self:
            if not line.is_expense and line.product_id.type in ['consu', 'product']:
                line.qty_delivered_method = 'stock_move'

    @api.depends('move_ids.state', 'move_ids.scrapped', 'move_ids.product_uom_qty', 'move_ids.product_uom')
    def _compute_qty_delivered(self):
        super(SaleOrderLine, self)._compute_qty_delivered()

        for line in self:  # TODO: maybe one day, this should be done in SQL for performance sake
            if line.qty_delivered_method == 'stock_move':
                qty = 0.0
                outgoing_moves, incoming_moves = line._get_outgoing_incoming_moves()
                for move in outgoing_moves:
                    if move.state != 'done':
                        continue
                    qty += move.product_uom._compute_quantity(move.product_uom_qty, line.product_uom, rounding_method='HALF-UP')
                for move in incoming_moves:
                    if move.state != 'done':
                        continue
                    qty -= move.product_uom._compute_quantity(move.product_uom_qty, line.product_uom, rounding_method='HALF-UP')
                line.qty_delivered = qty

    @api.model_create_multi
    def create(self, vals_list):
        lines = super(SaleOrderLine, self).create(vals_list)
        lines.filtered(lambda line: line.state == 'sale')._action_launch_stock_rule()
        return lines

    def write(self, values):
        lines = self.env['sale.order.line']
        if 'product_uom_qty' in values:
            precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
            lines = self.filtered(
                lambda r: r.state == 'sale' and not r.is_expense and float_compare(r.product_uom_qty, values['product_uom_qty'], precision_digits=precision) == -1)
        previous_product_uom_qty = {line.id: line.product_uom_qty for line in lines}
        res = super(SaleOrderLine, self).write(values)
        if lines:
            lines._action_launch_stock_rule(previous_product_uom_qty)
        return res

    @api.depends('order_id.state')
    def _compute_invoice_status(self):
        def check_moves_state(moves):
            # All moves states are either 'done' or 'cancel', and there is at least one 'done'
            at_least_one_done = False
            for move in moves:
                if move.state not in ['done', 'cancel']:
                    return False
                at_least_one_done = at_least_one_done or move.state == 'done'
            return at_least_one_done
        super(SaleOrderLine, self)._compute_invoice_status()
        for line in self:
            # We handle the following specific situation: a physical product is partially delivered,
            # but we would like to set its invoice status to 'Fully Invoiced'. The use case is for
            # products sold by weight, where the delivered quantity rarely matches exactly the
            # quantity ordered.
            if line.order_id.state == 'done'\
                    and line.invoice_status == 'no'\
                    and line.product_id.type in ['consu', 'product']\
                    and line.product_id.invoice_policy == 'delivery'\
                    and line.move_ids \
                    and check_moves_state(line.move_ids):
                line.invoice_status = 'invoiced'

    @api.depends('move_ids')
    def _compute_product_updatable(self):
        for line in self:
            if not line.move_ids.filtered(lambda m: m.state != 'cancel'):
                super(SaleOrderLine, line)._compute_product_updatable()
            else:
                line.product_updatable = False

    @api.onchange('product_id')
    def _onchange_product_id_set_customer_lead(self):
        self.customer_lead = self.product_id.sale_delay

    @api.onchange('product_packaging')
    def _onchange_product_packaging(self):
        if self.product_packaging:
            return self._check_package()

    @api.onchange('product_uom_qty')
    def _onchange_product_uom_qty(self):
        # When modifying a one2many, _origin doesn't guarantee that its values will be the ones
        # in database. Hence, we need to explicitly read them from there.
        if self._origin:
            product_uom_qty_origin = self._origin.read(["product_uom_qty"])[0]["product_uom_qty"]
        else:
            product_uom_qty_origin = 0

        if self.state == 'sale' and self.product_id.type in ['product', 'consu'] and self.product_uom_qty < product_uom_qty_origin:
            # Do not display this warning if the new quantity is below the delivered
            # one; the `write` will raise an `UserError` anyway.
            if self.product_uom_qty < self.qty_delivered:
                return {}
            warning_mess = {
                'title': _('Ordered quantity decreased!'),
                'message' : _('You are decreasing the ordered quantity! Do not forget to manually update the delivery order if needed.'),
            }
            return {'warning': warning_mess}
        if self.product_packaging:
            return self._check_package()
        return {}

    def _prepare_procurement_values(self, group_id=False):
        """ Prepare specific key for moves or other components that will be created from a stock rule
        comming from a sale order line. This method could be override in order to add other custom key that could
        be used in move/po creation.
        """
        values = super(SaleOrderLine, self)._prepare_procurement_values(group_id)
        self.ensure_one()
        date_planned = self.order_id.date_order\
            + timedelta(days=self.customer_lead or 0.0) - timedelta(days=self.order_id.company_id.security_lead)
        values.update({
            'group_id': group_id,
            'sale_line_id': self.id,
            'date_planned': date_planned,
            'route_ids': self.route_id,
            'warehouse_id': self.order_id.warehouse_id or False,
            'partner_id': self.order_id.partner_shipping_id.id,
            'company_id': self.order_id.company_id,
            'sequence': self.sequence,
        })
        for line in self.filtered("order_id.commitment_date"):
            date_planned = fields.Datetime.from_string(line.order_id.commitment_date) - timedelta(days=line.order_id.company_id.security_lead)
            values.update({
                'date_planned': fields.Datetime.to_string(date_planned),
            })
        return values

    def _get_qty_procurement(self, previous_product_uom_qty=False):
        self.ensure_one()
        qty = 0.0
        outgoing_moves, incoming_moves = self._get_outgoing_incoming_moves()
        for move in outgoing_moves:
            qty += move.product_uom._compute_quantity(move.product_uom_qty, self.product_uom, rounding_method='HALF-UP')
        for move in incoming_moves:
            qty -= move.product_uom._compute_quantity(move.product_uom_qty, self.product_uom, rounding_method='HALF-UP')
        return qty

    def _get_outgoing_incoming_moves(self):
        outgoing_moves = self.env['stock.move']
        incoming_moves = self.env['stock.move']

        for move in self.move_ids.filtered(lambda r: r.state != 'cancel' and not r.scrapped and self.product_id == r.product_id):
            if move.location_dest_id.usage == "customer":
                if not move.origin_returned_move_id or (move.origin_returned_move_id and move.to_refund):
                    outgoing_moves |= move
            elif move.location_dest_id.usage != "customer" and move.to_refund:
                incoming_moves |= move

        return outgoing_moves, incoming_moves

    def _get_procurement_group(self):
        return self.order_id.procurement_group_id

    def _prepare_procurement_group_vals(self):
        return {
            'name': self.order_id.name,
            'move_type': self.order_id.picking_policy,
            'sale_id': self.order_id.id,
            'partner_id': self.order_id.partner_shipping_id.id,
        }

    def _action_launch_stock_rule(self, previous_product_uom_qty=False):
        """
        Launch procurement group run method with required/custom fields genrated by a
        sale order line. procurement group will launch '_run_pull', '_run_buy' or '_run_manufacture'
        depending on the sale order line product rule.
        """
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        procurements = []
        for line in self:
            if line.state != 'sale' or not line.product_id.type in ('consu','product'):
                continue
            qty = line._get_qty_procurement(previous_product_uom_qty)
            if float_compare(qty, line.product_uom_qty, precision_digits=precision) >= 0:
                continue

            group_id = line._get_procurement_group()
            if not group_id:
                group_id = self.env['procurement.group'].create(line._prepare_procurement_group_vals())
                line.order_id.procurement_group_id = group_id
            else:
                # In case the procurement group is already created and the order was
                # cancelled, we need to update certain values of the group.
                updated_vals = {}
                if group_id.partner_id != line.order_id.partner_shipping_id:
                    updated_vals.update({'partner_id': line.order_id.partner_shipping_id.id})
                if group_id.move_type != line.order_id.picking_policy:
                    updated_vals.update({'move_type': line.order_id.picking_policy})
                if updated_vals:
                    group_id.write(updated_vals)

            values = line._prepare_procurement_values(group_id=group_id)
            product_qty = line.product_uom_qty - qty

            line_uom = line.product_uom
            quant_uom = line.product_id.uom_id
            product_qty, procurement_uom = line_uom._adjust_uom_quantities(product_qty, quant_uom)
            procurements.append(self.env['procurement.group'].Procurement(
                line.product_id, product_qty, procurement_uom,
                line.order_id.partner_shipping_id.property_stock_customer,
                line.name, line.order_id.name, line.order_id.company_id, values))
        if procurements:
            self.env['procurement.group'].run(procurements)
        return True

    def _check_package(self):
        default_uom = self.product_id.uom_id
        pack = self.product_packaging
        qty = self.product_uom_qty
        q = default_uom._compute_quantity(pack.qty, self.product_uom)
        # We do not use the modulo operator to check if qty is a mltiple of q. Indeed the quantity
        # per package might be a float, leading to incorrect results. For example:
        # 8 % 1.6 = 1.5999999999999996
        # 5.4 % 1.8 = 2.220446049250313e-16
        if (
            qty
            and q
            and float_compare(
                qty / q, float_round(qty / q, precision_rounding=1.0), precision_rounding=0.001
            )
            != 0
        ):
            newqty = qty - (qty % q) + q
            return {
                'warning': {
                    'title': _('Warning'),
                    'message': _("This product is packaged by %.2f %s. You should sell %.2f %s.") % (pack.qty, default_uom.name, newqty, self.product_uom.name),
                },
            }
        return {}

    def _update_line_quantity(self, values):
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        line_products = self.filtered(lambda l: l.product_id.type in ['product', 'consu'])
        if line_products.mapped('qty_delivered') and float_compare(values['product_uom_qty'], max(line_products.mapped('qty_delivered')), precision_digits=precision) == -1:
            raise UserError(_('You cannot decrease the ordered quantity below the delivered quantity.\n'
                              'Create a return first.'))
        super(SaleOrderLine, self)._update_line_quantity(values)

```

## File: models\stock.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import api, fields, models, _
from odoo.tools.sql import column_exists, create_column


class StockLocationRoute(models.Model):
    _inherit = "stock.location.route"
    sale_selectable = fields.Boolean("Selectable on Sales Order Line")


class StockMove(models.Model):
    _inherit = "stock.move"
    sale_line_id = fields.Many2one('sale.order.line', 'Sale Line', index=True)

    @api.model
    def _prepare_merge_moves_distinct_fields(self):
        distinct_fields = super(StockMove, self)._prepare_merge_moves_distinct_fields()
        distinct_fields.append('sale_line_id')
        return distinct_fields

    @api.model
    def _prepare_merge_move_sort_method(self, move):
        move.ensure_one()
        keys_sorted = super(StockMove, self)._prepare_merge_move_sort_method(move)
        keys_sorted.append(move.sale_line_id.id)
        return keys_sorted

    def _get_related_invoices(self):
        """ Overridden from stock_account to return the customer invoices
        related to this stock move.
        """
        rslt = super(StockMove, self)._get_related_invoices()
        invoices = self.mapped('picking_id.sale_id.invoice_ids').filtered(lambda x: x.state == 'posted')
        rslt += invoices
        #rslt += invoices.mapped('reverse_entry_ids')
        return rslt

    def _assign_picking_post_process(self, new=False):
        super(StockMove, self)._assign_picking_post_process(new=new)
        if new:
            picking_id = self.mapped('picking_id')
            sale_order_ids = self.mapped('sale_line_id.order_id')
            for sale_order_id in sale_order_ids:
                picking_id.message_post_with_view(
                    'mail.message_origin_link',
                    values={'self': picking_id, 'origin': sale_order_id},
                    subtype_id=self.env.ref('mail.mt_note').id)


class ProcurementGroup(models.Model):
    _inherit = 'procurement.group'

    sale_id = fields.Many2one('sale.order', 'Sale Order')


class StockRule(models.Model):
    _inherit = 'stock.rule'

    def _get_custom_move_fields(self):
        fields = super(StockRule, self)._get_custom_move_fields()
        fields += ['sale_line_id', 'partner_id', 'sequence']
        return fields


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    sale_id = fields.Many2one(related="group_id.sale_id", string="Sales Order", store=True, readonly=False)

    def _auto_init(self):
        """
        Create related field here, too slow
        when computing it afterwards through _compute_related.

        Since group_id.sale_id is created in this module,
        no need for an UPDATE statement.
        """
        if not column_exists(self.env.cr, 'stock_picking', 'sale_id'):
            create_column(self.env.cr, 'stock_picking', 'sale_id', 'int4')
        return super()._auto_init()

    def _log_less_quantities_than_expected(self, moves):
        """ Log an activity on sale order that are linked to moves. The
        note summarize the real proccessed quantity and promote a
        manual action.

        :param dict moves: a dict with a move as key and tuple with
        new and old quantity as value. eg: {move_1 : (4, 5)}
        """

        def _keys_in_sorted(sale_line):
            """ sort by order_id and the sale_person on the order """
            return (sale_line.order_id.id, sale_line.order_id.user_id.id)

        def _keys_in_groupby(sale_line):
            """ group by order_id and the sale_person on the order """
            return (sale_line.order_id, sale_line.order_id.user_id)

        def _render_note_exception_quantity(moves_information):
            """ Generate a note with the picking on which the action
            occurred and a summary on impacted quantity that are
            related to the sale order where the note will be logged.

            :param moves_information dict:
            {'move_id': ['sale_order_line_id', (new_qty, old_qty)], ..}

            :return: an html string with all the information encoded.
            :rtype: str
            """
            origin_moves = self.env['stock.move'].browse([move.id for move_orig in moves_information.values() for move in move_orig[0]])
            origin_picking = origin_moves.mapped('picking_id')
            values = {
                'origin_moves': origin_moves,
                'origin_picking': origin_picking,
                'moves_information': moves_information.values(),
            }
            return self.env.ref('sale_stock.exception_on_picking').render(values=values)

        documents = self._log_activity_get_documents(moves, 'sale_line_id', 'DOWN', _keys_in_sorted, _keys_in_groupby)
        self._log_activity(_render_note_exception_quantity, documents)

        return super(StockPicking, self)._log_less_quantities_than_expected(moves)

    def _needs_automatic_assign(self):
        self.ensure_one()
        if self.sale_id:
            return False
        return super()._needs_automatic_assign()


class ProductionLot(models.Model):
    _inherit = 'stock.production.lot'

    sale_order_ids = fields.Many2many('sale.order', string="Sales Orders", compute='_compute_sale_order_ids')
    sale_order_count = fields.Integer('Sale order count', compute='_compute_sale_order_ids')

    @api.depends('name')
    def _compute_sale_order_ids(self):
        sale_orders = defaultdict(lambda: self.env['sale.order'])
        for move_line in self.env['stock.move.line'].search([('lot_id', 'in', self.ids), ('state', '=', 'done')]):
            move = move_line.move_id
            if move.picking_id.location_dest_id.usage == 'customer' and move.sudo().sale_line_id.order_id:
                sale_orders[move_line.lot_id.id] |= move.sudo().sale_line_id.order_id
        for lot in self:
            lot.sale_order_ids = sale_orders[lot.id]
            lot.sale_order_count = len(lot.sale_order_ids)

    def action_view_so(self):
        self.ensure_one()
        action = self.env.ref('sale.action_orders').read()[0]
        action['domain'] = [('id', 'in', self.mapped('sale_order_ids.id'))]
        action['context'] = dict(self._context, create=False)
        return action

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import res_company
from . import sale_order
from . import res_config_settings
from . import stock
from . import product_template

```

## File: report\report_stock_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ReportStockRule(models.AbstractModel):
    _inherit = 'report.stock.report_stock_rule'

    @api.model
    def _get_routes(self, data):
        res = super(ReportStockRule, self)._get_routes(data)
        if data.get('so_route_ids'):
            res = self.env['stock.location.route'].browse(data['so_route_ids']) | res
        return res

```

## File: report\sale_order_report_templates.xml

```xml
<?xml version="1.0"?>
<odoo>
    <template id="report_saleorder_document_inherit_sale_stock" inherit_id="sale.report_saleorder_document">
        <xpath expr="//div[@name='expiration_date']" position="after">
            <div class="col-3" t-if="doc.incoterm" groups="sale_stock.group_display_incoterm">
                <strong>Incoterm:</strong>
                <p t-field="doc.incoterm.code"/>
            </div>
        </xpath>
    </template>

    <template id="report_invoice_document_inherit_sale_stock" inherit_id="account.report_invoice_document">
        <xpath expr="//div[@name='reference']" position="after">
            <div class="col-auto mw-100 mb-2" t-if="o.invoice_incoterm_id" groups="sale_stock.group_display_incoterm" name="invoice_incoterm_id">
                <strong>Incoterm:</strong>
                <p class="m-0" t-field="o.invoice_incoterm_id.code"/>
            </div>
        </xpath>
    </template>
</odoo>

```

## File: report\sale_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class SaleReport(models.Model):
    _inherit = "sale.report"

    warehouse_id = fields.Many2one('stock.warehouse', 'Warehouse', readonly=True)

    def _query(self, with_clause='', fields={}, groupby='', from_clause=''):
        fields['warehouse_id'] = ", s.warehouse_id as warehouse_id"
        groupby += ', s.warehouse_id'
        return super(SaleReport, self)._query(with_clause, fields, groupby, from_clause)

```

## File: report\stock_report_deliveryslip.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_delivery_document_inherit_sale_stock" inherit_id="stock.report_delivery_document">
        <xpath expr="//div[@name='div_sched_date']" position="after">
            <div class="row justify-content-end" t-if="o.sudo().sale_id.client_order_ref">
                <div class="col-auto">
                    <strong>Customer Reference:</strong>
                    <p t-field="o.sudo().sale_id.client_order_ref"/>
                </div>
            </div>
        </xpath>
    </template>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale_report
from . import report_stock_rule

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_stock_picking_salesman,stock_picking salesman,stock.model_stock_picking,sales_team.group_sale_salesman,1,1,1,0
access_stock_move_salesman,stock_move salesman,stock.model_stock_move,sales_team.group_sale_salesman,1,1,1,0
access_stock_move_manager,stock_move manager,stock.model_stock_move,sales_team.group_sale_manager,1,1,1,1
access_sale_order_stock_worker,sale.order stock worker,model_sale_order,stock.group_stock_user,1,1,0,0
access_sale_order_line_stock_worker,sale.order.line stock worker,model_sale_order_line,stock.group_stock_user,1,1,0,0
access_stock_picking_sales,stock.picking.sales,stock.model_stock_picking,sales_team.group_sale_manager,1,1,1,1
access_product_packaging_user,product.packaging.user,product.model_product_packaging,sales_team.group_sale_salesman,1,1,1,0
access_product_packaging_manager,product.packaging.manager,product.model_product_packaging,sales_team.group_sale_manager,1,0,0,0
access_stock_warehouse_user,stock.warehouse.user,stock.model_stock_warehouse,sales_team.group_sale_salesman,1,0,0,0
access_stock_location_user,stock.location.user,stock.model_stock_location,sales_team.group_sale_salesman,1,0,0,0
access_product_packaging_sale_manager,product.packaging salemanager,product.model_product_packaging,sales_team.group_sale_manager,1,1,1,1
access_stock_warehouse_orderpoint_sale_salesman,stock.warehouse.orderpoint,stock.model_stock_warehouse_orderpoint,sales_team.group_sale_salesman,1,0,0,0
access_account_partial_reconcile,account.partial.reconcile,account.model_account_partial_reconcile,stock.group_stock_manager,1,1,1,1
access_account_journal,account.journal,account.model_account_journal,stock.group_stock_manager,1,1,0,0
access_stock_location_sale_manager,stock.location sale manager,stock.model_stock_location,sales_team.group_sale_manager,1,0,0,0
access_stock_rule_salemanager,stock_rule salemanager,stock.model_stock_rule,sales_team.group_sale_manager,1,1,1,1
access_stock_rule,stock.rule.flow,stock.model_stock_rule,sales_team.group_sale_salesman,1,0,0,0

```

## File: security\sale_stock_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">
        <record id="group_display_incoterm" model="res.groups">
            <field name="name">Display incoterms on Sales Order and related invoices</field>
            <field name="category_id" ref="base.module_category_hidden"/>
        </record>

        <record id="group_lot_on_invoice" model="res.groups">
            <field name="name">Display Serial &amp; Lot Number on Invoices</field>
            <field name="category_id" ref="base.module_category_hidden"/>
        </record>

        <!-- Stock Portal Access Rules -->
        <record id="stock_picking_rule_portal" model="ir.rule">
            <field name="name">Portal Follower Transfers</field>
            <field name="model_id" ref="stock.model_stock_picking"/>
            <field name="domain_force">['|', '|', ('message_partner_ids', 'in', [user.partner_id.id]), ('partner_id', '=', user.partner_id.id), ('sale_id.partner_id', '=', user.partner_id.id)]</field>
            <field name="groups" eval="[(4, ref('base.group_portal'))]"/>
        </record>
    </data>
</odoo>

```

## File: static\src\js\qty_at_date_widget.js

```javascript
odoo.define('sale_stock.QtyAtDateWidget', function (require) {
"use strict";

var core = require('web.core');
var QWeb = core.qweb;

var Widget = require('web.Widget');
var Context = require('web.Context');
var data_manager = require('web.data_manager');
var widget_registry = require('web.widget_registry');
var config = require('web.config');

var _t = core._t;
var time = require('web.time');

var QtyAtDateWidget = Widget.extend({
    template: 'sale_stock.qtyAtDate',
    events: _.extend({}, Widget.prototype.events, {
        'click .fa-info-circle': '_onClickButton',
    }),

    /**
     * @override
     * @param {Widget|null} parent
     * @param {Object} params
     */
    init: function (parent, params) {
        this.data = params.data;
        this._super(parent);
    },

    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            self._setPopOver();
        });
    },

    updateState: function (state) {
        this.$el.popover('dispose');
        var candidate = state.data[this.getParent().currentRow];
        if (candidate) {
            this.data = candidate.data;
            this.renderElement();
            this._setPopOver();
        }
    },
    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------
    /**
     * Set a bootstrap popover on the current QtyAtDate widget that display available
     * quantity.
     */
    _setPopOver: function () {
        var self = this;
        if (!this.data.scheduled_date) {
            return;
        }
        this.data.delivery_date = this.data.scheduled_date.clone().add(this.getSession().getTZOffset(this.data.scheduled_date), 'minutes').format(time.getLangDateFormat());
        // The grid view need a specific date format that could be different than
        // the user one.
        this.data.delivery_date_grid = this.data.scheduled_date.clone().add(this.getSession().getTZOffset(this.data.scheduled_date), 'minutes').format('YYYY-MM-DD');
        this.data.debug = config.isDebug();
        var $content = $(QWeb.render('sale_stock.QtyDetailPopOver', {
            data: this.data,
        }));
        var $forecastButton = $content.find('.action_open_forecast');
        $forecastButton.on('click', function(ev) {
            ev.stopPropagation();
            data_manager.load_action('stock.report_stock_quantity_action_product').then(function (action) {
                // Change action context to choose a specific date and product(s)
                // As grid_anchor is set to now() by default in the data, we need
                // to load the action first, change the context then launch it via do_action
                // additional_context cannot replace a context value, only add new
                //
                // in case of kit product, the forecast view show the kit's components
                self._rpc({
                    model: 'product.product',
                    method: 'get_components',
                    args: [[self.data.product_id.data.id]]
                }).then(function (res) {
                    var additional_context = {};
                    additional_context.grid_anchor = self.data.delivery_date_grid;
                    additional_context.search_default_warehouse_id = [self.data.warehouse_id.data.id];
                    action.context = new Context(action.context, additional_context);
                    action.domain = [
                        ['product_id', 'in', res]
                    ];
                    self.do_action(action);
                });
            });
        });
        var options = {
            content: $content,
            html: true,
            placement: 'left',
            title: _t('Availability'),
            trigger: 'focus',
            delay: {'show': 0, 'hide': 100 },
        };
        this.$el.popover(options);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------
    _onClickButton: function () {
        // We add the property special click on the widget link.
        // This hack allows us to trigger the popover (see _setPopOver) without
        // triggering the _onRowClicked that opens the order line form view.
        this.$el.find('.fa-info-circle').prop('special_click', true);
    },
});

widget_registry.add('qty_at_date_widget', QtyAtDateWidget);

return QtyAtDateWidget;
});

```

## File: static\src\xml\qty_at_date.xml

```xml
<templates>
    <div t-name="sale_stock.qtyAtDate">
        <div t-att-class="!widget.data.display_qty_widget ? 'invisible' : ''">
            <t t-if="widget.data.virtual_available_at_date &lt; widget.data.qty_to_deliver and !widget.data.is_mto">
                <a tabindex="0" class="fa fa-info-circle text-danger"/>
            </t>
            <t t-else="">
                <a tabindex="0" class="fa fa-info-circle text-primary"/>
            </t>
        </div>
    </div>

    <div t-name="sale_stock.QtyDetailPopOver">
        <table>
            <tbody>
                <t t-if="!data.is_mto">
                    <tr>
                        <td><strong>Forecasted Stock</strong><br /><small>On <span t-esc="data.delivery_date"/></small></td>
                        <td><t t-esc='data.virtual_available_at_date'/>
                        <t t-esc='data.product_uom.data.display_name'/></td>
                    </tr>
                    <tr>
                        <td><strong>Available</strong><br /><small>All planned operations included</small></td>
                        <td><t t-esc='data.free_qty_today'/>
                        <t t-esc='data.product_uom.data.display_name'/></td>
                    </tr>
                </t>
                <t t-else="">
                    <tr>
                        <td><strong>Expected Delivery</strong></td>
                        <td class="oe-right"><span t-esc="data.delivery_date"/></td>
                    </tr>
                    <tr>
                        <p>This product is replenished on demand.</p>
                    </tr>
                </t>
            </tbody>
        </table>
        <button t-if="!data.is_mto" class="text-left btn btn-link action_open_forecast"
            type="button">
            <i class="fa fa-fw o_button_icon fa-arrow-right"></i>
            View Forecast
        </button>
    </div>
</templates>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="sale_stock_report_invoice_document" inherit_id="account.report_invoice_document">
        <xpath expr="//div[@id='total']" position="after">
          <t groups="sale_stock.group_lot_on_invoice">
            <t t-set="lot_values" t-value="o._get_invoiced_lot_values()"/>
            <t t-if="lot_values">
                <br/>
                <table class="table table-sm" style="width: 50%;" name="invoice_snln_table">
                    <thead>
                        <tr>
                            <th><span>Product</span></th>
                            <th class="text-right"><span>Quantity</span></th>
                            <th class="text-right"><span>SN/LN</span></th>
                        </tr>
                    </thead>
                    <tbody>
                        <t t-foreach="lot_values" t-as="snln_line">
                            <tr>
                                <td><t t-esc="snln_line['product_name']"/></td>
                                <td class="text-right">
                                    <t t-esc="snln_line['quantity']"/>
                                    <t t-esc="snln_line['uom_name']" groups="uom.group_uom"/>
                                </td>
                                <td class="text-right"><t t-esc="snln_line['lot_name']"/></td>
                            </tr>
                        </t>
                    </tbody>
                </table>
            </t>
          </t>
        </xpath>
    </template>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form_sale" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.sale.stock.sale</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='ups']" position="after">
                <div class="col-12 col-lg-6 o_setting_box">
                    <div class="o_setting_left_pane">
                        <field name="group_display_incoterm"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="group_display_incoterm"/>
                        <div class="text-muted">
                            Display incoterms on orders &amp; invoices
                        </div>
                        <div class="content-group" attrs="{'invisible': [('group_display_incoterm','=',False)]}">
                            <div class="mt8">
                                <button name="%(account.action_incoterms_tree)d" icon="fa-arrow-right" type="action" string="Incoterms" class="btn-link"/>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

    <record id="res_config_settings_view_form_stock" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.sale.stock.stock</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="stock.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="production_lot_info" position="inside">
                <div class="col-12 col-lg-6 o_setting_box" attrs="{'invisible': [('group_stock_production_lot', '=', False)]}" id="group_lot_on_invoice">
                    <div class="o_setting_left_pane">
                        <field name="group_lot_on_invoice"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="group_lot_on_invoice"/>
                        <div class="text-muted">
                            Lots &amp; Serial numbers will appear on the invoice
                        </div>
                    </div>
                </div>
            </div>
            <div id="warning_info" position="after">
                <div class="col-12 col-lg-6 o_setting_box" title="Reserving products manually in delivery orders or by running the scheduler is advised to better manage priorities in case of long customer lead times or/and frequent stock-outs. By default, the scheduler runs automatically every 24 hours.">
                    <div class="o_setting_right_pane">
                        <label for="module_procurement_jit"/>
                        <div class="text-muted">
                            When to reserve sold products
                        </div>
                        <div class="content-group">
                            <div class="mt16">
                                <field name="module_procurement_jit" class="o_light_label" widget="radio"/>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="col-12 col-lg-6 o_setting_box">
                    <div class="o_setting_right_pane">
                        <label for="default_picking_policy"/>
                        <div class="text-muted">
                            When to start shipping
                        </div>
                        <div class="content-group">
                            <div class="mt16">
                                <field name="default_picking_policy" class="o_light_label" widget="selection"/>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <xpath expr="//h2[@id='schedule_info']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <div id="sale_security_lead" position="replace">
                <div class="col-12 col-lg-6 o_setting_box" title="Margin of error for dates promised to customers. Products will be scheduled for procurement and delivery that many days earlier than the actual promised date, to cope with unexpected delays in the supply chain.">
                    <div class="o_setting_left_pane">
                        <field name="use_security_lead"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="use_security_lead"/>
                        <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." role="img" aria-label="Values set here are company-specific." groups="base.group_multi_company"/>
                        <div class="text-muted">
                            Schedule deliveries earlier to avoid delays
                        </div>
                        <div class="content-group">
                            <div class="mt16" attrs="{'invisible': [('use_security_lead','=',False)]}">
                                <span>Move forward expected delivery dates by <field name="security_lead" class="oe_inline"/> days</span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </field>
    </record>

</odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <record id="view_order_form_inherit_sale_stock" model="ir.ui.view">
            <field name="name">sale.order.form.sale.stock</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="sale.view_order_form"/>
            <field name="arch" type="xml">
                <xpath expr="//button[@name='action_view_invoice']" position="before">
                    <field name="picking_ids" invisible="1"/>
                    <button type="object"
                        name="action_view_delivery"
                        class="oe_stat_button"
                        icon="fa-truck"
                        attrs="{'invisible': [('delivery_count', '=', 0)]}" groups="base.group_user">
                        <field name="delivery_count" widget="statinfo" string="Delivery"/>
                    </button>
                </xpath>
                <xpath expr="//group[@name='sale_shipping']" position="attributes">
                    <attribute name="groups"></attribute><!-- Remove the res.group on the group and set it on the field directly-->
                    <attribute name="string">Delivery</attribute>
                </xpath>
                <xpath expr="//label[@for='commitment_date']" position="before">
                    <field name="warehouse_id" options="{'no_create': True}" groups="stock.group_stock_multi_warehouses" force_save="1"/>
                    <field name="incoterm" widget="selection" groups="sale_stock.group_display_incoterm"/>
                    <field name="picking_policy" required="True"/>
                </xpath>
                <xpath expr="//group[@name='sale_shipping']" position="inside">
                    <field name="effective_date" attrs="{'invisible': [('effective_date', '=', False)]}"/>
                </xpath>
                <xpath expr="//page/field[@name='order_line']/form/group/group/field[@name='price_unit']" position="before">
                    <field name="product_packaging" attrs="{'invisible': [('product_id', '=', False)]}" context="{'default_product_id': product_id, 'tree_view_ref':'product.product_packaging_tree_view', 'form_view_ref':'product.product_packaging_form_view'}" domain="[('product_id','=',product_id)]" groups="product.group_stock_packaging" />
                </xpath>
                <xpath expr="//page/field[@name='order_line']/tree/field[@name='price_unit']" position="before">
                    <field name="product_packaging" attrs="{'invisible': [('product_id', '=', False)]}" context="{'default_product_id': product_id, 'tree_view_ref':'product.product_packaging_tree_view', 'form_view_ref':'product.product_packaging_form_view'}" domain="[('product_id','=',product_id)]" groups="product.group_stock_packaging" optional="show"/>
                </xpath>
                <xpath expr="//field[@name='order_line']/form/group/group/field[@name='analytic_tag_ids']" position="before">
                    <field name="route_id" groups="stock.group_adv_location" options="{'no_create': True}"/>
                </xpath>
                <xpath expr="//field[@name='order_line']/tree/field[@name='analytic_tag_ids']" position="after">
                    <field name="route_id" groups="stock.group_adv_location" options="{'no_create': True}" optional="hide"/>
                </xpath>
           </field>
        </record>

        <record id="view_quotation_tree" model="ir.ui.view">
            <field name="name">sale.order.tree.inherit.sale.stock</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="sale.view_quotation_tree"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='team_id']" position="after">
                    <field name="warehouse_id" options="{'no_create': True}" groups="stock.group_stock_multi_warehouses" optional="hide" />
                </xpath>
            </field>
        </record>

        <record id="view_order_tree" model="ir.ui.view">
            <field name="name">sale.order.tree.inherit.sale.stock</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="sale.view_order_tree"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='team_id']" position="after">
                    <field name="warehouse_id" options="{'no_create': True}" groups="stock.group_stock_multi_warehouses" optional="hide"/>
                </xpath>
            </field>
        </record>

        <record id="view_order_line_tree_inherit_sale_stock" model="ir.ui.view">
            <field name="name">sale.order.line.tree.sale.stock.location</field>
            <field name="inherit_id" ref="sale.view_order_line_tree"/>
            <field name="model">sale.order.line</field>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='price_subtotal']" position="before">
                    <field name="route_id" groups="stock.group_adv_location" options="{'no_create': True}"/>
                </xpath>
            </field>
        </record>

        <record id="view_order_form_inherit_sale_stock_qty" model="ir.ui.view">
            <field name="name">sale.order.line.tree.sale.stock.qty</field>
            <field name="inherit_id" ref="sale.view_order_form"/>
            <field name="model">sale.order</field>
            <field name="arch" type="xml">
                <xpath expr="//page/field[@name='order_line']/tree/field[@name='product_uom_qty']" position="after">
                    <!-- below fields are used in the widget qty_at_date_widget -->
                    <field name="product_type" invisible="1"/>
                    <field name="virtual_available_at_date" invisible="1"/>
                    <field name="qty_available_today" invisible="1"/>
                    <field name="free_qty_today" invisible="1"/>
                    <field name="scheduled_date" invisible="1"/>
                    <field name="warehouse_id" invisible="1"/>
                    <field name="qty_to_deliver" invisible="1"/>
                    <field name="is_mto" invisible="1"/>
                    <field name="display_qty_widget" invisible="1"/>
                    <widget name="qty_at_date_widget" width="20px"/>
                </xpath>
            </field>
        </record>

        <template id="sale_order_line_view_list" name="sale.order.line.view.list" inherit_id="web.assets_backend">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/sale_stock/static/src/js/qty_at_date_widget.js"></script>
            </xpath>
        </template>
    </data>
</odoo>

```

## File: views\sale_stock_portal_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="sale_order_portal_content_inherit_sale_stock" name="Orders Shipping Followup" inherit_id="sale.sale_order_portal_content">
        <xpath expr="//div[@id='so_date']" position="after">
            <div class="row" t-if="sale_order.incoterm">
                <div class="mb-3 col-6 ml-auto">
                    <strong>Incoterm: </strong> <span t-field="sale_order.incoterm"/>
                </div>
            </div>
        </xpath>
        <xpath expr="//div[@id='informations']" position="inside">
            <t t-if="sale_order.picking_ids">
                <div>
                    <strong>Delivery Orders</strong>
                </div>
                <div>
                    <t t-foreach="sale_order.picking_ids.filtered(lambda picking: picking.picking_type_id.code != 'internal')" t-as="i">
                        <t t-set="delivery_report_url" t-value="'/my/picking/pdf/%s?%s' % (i.id, keep_query())"/>
                        <div class="d-flex flex-wrap align-items-center justify-content-between o_sale_stock_picking">
                            <div>
                                <a t-att-href="delivery_report_url">
                                    <span t-esc="i.name"/>
                                </a>
                                <div class="small d-lg-inline-block ml-3">Date:
                                    <span t-if="i.state == 'done'" class="text-muted" t-field="i.date_done" t-options="{'date_only': True}"/>
                                    <span t-if="i.state != 'done'" class="text-muted" t-field="i.scheduled_date" t-options="{'date_only': True}"/>
                                </div>
                            </div>
                            <span t-if="i.state == 'done'" class="small badge badge-success orders_label_text_align"><i class="fa fa-fw fa-truck"/> <b>Shipped</b></span>
                            <span t-if="i.state == 'cancel'" class="small badge badge-danger orders_label_text_align"><i class="fa fa-fw fa-times"/> <b>Cancelled</b></span>
                            <span t-if="i.state in ['draft', 'waiting', 'confirmed', 'assigned']" class="small badge badge-info orders_label_text_align"><i class="fa fa-fw fa-clock-o"/> <b>Preparation</b></span>
                        </div>
                    </t>
                </div>
            </t>
        </xpath>
  </template>

</odoo>

```

## File: views\stock_production_lot_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="stock_production_lot_view_form" model="ir.ui.view">
        <field name="name">stock.production.lot.view.form</field>
        <field name="model">stock.production.lot</field>
        <field name="inherit_id" ref="stock.view_production_lot_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_button_box')]/button" position="before">
                <button class="oe_stat_button" name="action_view_so"
                        type="object" icon="fa-pencil-square-o" help="Sale Orders"
                        attrs="{'invisible': ['|', ('sale_order_count', '=', 0), ('display_complete', '=', False)]}">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="sale_order_count" widget="statinfo" nolabel="1" class="mr4"/>
                        </span>
                        <span class="o_stat_text">Sales</span>
                    </div>
                </button>
            </xpath>
            <xpath expr="//group[@name='main_group']" position="after">
                <group>
                    <field name="sale_order_ids" widget="many2many" readonly="True"
                           attrs="{'invisible': ['|', ('sale_order_ids', '=', []), ('display_complete', '=', True)]}">
                        <tree>
                            <field name="name"/>
                            <field name="partner_id"/>
                            <field name="date_order"/>
                            <field name="state" invisible="1"/>
                        </tree>
                    </field>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\stock_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="exception_on_so">
            <div>
                Exception(s) occurred on the sale order(s):
                <t t-foreach="sale_order_ids" t-as="sale_order">
                    <a href="#" data-oe-model="sale.order" t-att-data-oe-id="sale_order.id"><t t-esc="sale_order.name"/></a>.
                </t>
                Manual actions may be needed.
                <div class="mt16">
                    <p>Exception(s):</p>
                    <ul t-foreach="order_exceptions" t-as="exception">
                        <li>
                            <t t-set="order_line" t-value="exception[0]"/>
                            <t t-set="new_qty" t-value="exception[1][0]"/>
                            <t t-set="old_qty" t-value="exception[1][1]"/>
                                <a href="#" data-oe-model="sale.order" t-att-data-oe-id="order_line.order_id.id"><t t-esc="order_line.order_id.name"/></a>:
                                <t t-esc="new_qty"/> <t t-esc="order_line.product_uom.name"/> of <t t-esc="order_line.product_id.name"/>
                                <t t-if="cancel">
                                    cancelled
                                </t>
                                <t t-if="not cancel">
                                    ordered instead of <t t-esc="old_qty"/> <t t-esc="order_line.product_uom.name"/>
                                </t>
                          </li>
                    </ul>
                </div>
                <div class="mt16" t-if="not cancel and impacted_pickings">
                    <p>Impacted Transfer(s):</p>
                    <ul t-foreach="impacted_pickings" t-as="picking">
                        <li><a href="#" data-oe-model="stock.picking" t-att-data-oe-id="picking.id"><t t-esc="picking.name"/></a></li>
                    </ul>
                </div>
            </div>
        </template>

        <template id="exception_on_picking">
            <div>
                Exception(s) occurred on the picking:
                <a href="#" data-oe-model="stock.picking" t-att-data-oe-id="origin_picking.id"><t t-esc="origin_picking.name"/></a>.
                Manual actions may be needed.
                <div class="mt16">
                    <p>Exception(s):</p>
                    <ul t-foreach="moves_information" t-as="exception">
                        <t t-set="move" t-value="exception[0]"/>
                        <t t-set="new_qty" t-value="exception[1][0]"/>
                        <t t-set="old_qty" t-value="exception[1][1]"/>
                        <li>
                        <t t-esc="new_qty"/> <t t-esc="move.product_uom.name"/>
                        of <t t-esc="move.product_id.display_name"/> processed instead of <t t-esc="old_qty"/> <t t-esc="move.product_uom.name"/></li>
                    </ul>
                </div>
            </div>
        </template>

        <menuitem id="menu_aftersale" name="After-Sale"
            groups="sales_team.group_sale_salesman"
            parent="sale.sale_menu_root" sequence="5" />
        <menuitem id="menu_invoiced" name="Invoicing" parent="menu_aftersale" sequence="1"/>

        <record id="stock_location_route_view_form_inherit_sale_stock" model="ir.ui.view">
            <field name="name">stock.location.route.form</field>
            <field name="inherit_id" ref="stock.stock_location_route_form_view"/>
            <field name="model">stock.location.route</field>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='warehouse_ids']" position="after">
                    <br/><field name="sale_selectable" string="Sales Order Lines"/>
                </xpath>
            </field>
        </record>

        <record id="product_template_view_form_inherit_sale" model="ir.ui.view">
            <field name="name">product.template.inherit.form</field>
            <field name="inherit_id" ref="sale.product_template_form_view_sale_order_button"/>
            <field name="model">product.template</field>
            <field name="arch" type="xml">
                <xpath expr="//button[@name='action_view_sales']" position="replace" />
            </field>
        </record>

        <record id="product_template_view_form_inherit_stock" model="ir.ui.view">
            <field name="name">product.template.inherit.form</field>
            <field name="inherit_id" ref="stock.product_template_form_view_procurement_button"/>
            <field name="model">product.template</field>
            <field name="arch" type="xml">
                <button name="action_view_orderpoints" position="after">
                    <button class="oe_stat_button" name="action_view_sales"
                        type="object" icon="fa-signal" attrs="{'invisible': [('sale_ok', '=', False)]}"
                        groups="sales_team.group_sale_salesman" help="Sold in the last 365 days">
                        <div class="o_field_widget o_stat_info">
                            <span class="o_stat_value">
                                <field name="sales_count" widget="statinfo" nolabel="1" class="mr4"/>
                                <field name="uom_name"/>
                            </span>
                            <span class="o_stat_text">Sold</span>
                        </div>
                    </button>
                </button>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\stock_rules_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class StockRulesReport(models.TransientModel):
    _inherit = 'stock.rules.report'

    so_route_ids = fields.Many2many('stock.location.route', string='Apply specific routes',
        domain="[('sale_selectable', '=', True)]", help="Choose to apply SO lines specific routes.")

    def _prepare_report_data(self):
        data = super(StockRulesReport, self)._prepare_report_data()
        data['so_route_ids'] = self.so_route_ids.ids
        return data

```

## File: wizard\stock_rules_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_stock_rules_report_sale" model="ir.ui.view">
        <field name="name">Stock Rules Report Sale</field>
        <field name="model">stock.rules.report</field>
        <field name="inherit_id" ref="stock.view_stock_rules_report"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='warehouse_ids']" position="after">
                <field name="so_route_ids" widget="many2many_tags" groups="stock.group_adv_location"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import stock_rules_report

```

