# Odoo Module: purchase_requisition_stock

Category: Inventory/Purchase

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Purchase Requisition Stock',
    'version': '1.2',
    'category': 'Inventory/Purchase',
    'sequence': 70,
    'summary': '',
    'description': "",
    'depends': ['purchase_requisition', 'purchase_stock'],
    'demo': [
        'data/purchase_requisition_stock_demo.xml'
        ],
    'data': [
        'security/ir.model.access.csv',
        'data/purchase_requisition_stock_data.xml',
        'views/purchase_requisition_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\purchase_requisition_stock_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
         <function
            model="ir.default" name="set"
            eval="('purchase.requisition', 'warehouse_id', ref('stock.warehouse0'))"/>
    </data>
</odoo>

```

## File: data\purchase_requisition_stock_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="purchase_requisition.requisition1" model="purchase.requisition">
            <field name="warehouse_id" ref="stock.stock_warehouse_shop0"/>
        </record>
    </data>
</odoo>

```

## File: models\purchase.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class PurchaseOrder(models.Model):
    _inherit = 'purchase.order'

    @api.onchange('requisition_id')
    def _onchange_requisition_id(self):
        super(PurchaseOrder, self)._onchange_requisition_id()
        if self.requisition_id:
            self.picking_type_id = self.requisition_id.picking_type_id.id

```

## File: models\purchase_requisition.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class PurchaseRequisition(models.Model):
    _inherit = 'purchase.requisition'

    def _get_picking_in(self):
        pick_in = self.env.ref('stock.picking_type_in', raise_if_not_found=False)
        company = self.env.company
        if not pick_in or not pick_in.sudo().active or pick_in.sudo().warehouse_id.company_id.id != company.id:
            pick_in = self.env['stock.picking.type'].search(
                [('warehouse_id.company_id', '=', company.id), ('code', '=', 'incoming')],
                limit=1,
            )
        return pick_in

    warehouse_id = fields.Many2one('stock.warehouse', string='Warehouse', domain="[('company_id', '=', company_id)]")
    picking_type_id = fields.Many2one('stock.picking.type', 'Operation Type', required=True, default=_get_picking_in, domain="['|',('warehouse_id', '=', False), ('warehouse_id.company_id', '=', company_id)]")
    procurement_group_id = fields.Many2one('procurement.group', 'Procurement Group')

    def _prepare_tender_values(self, product_id, product_qty, product_uom, location_id, name, origin, company_id, values):
        return {
            'origin': origin,
            'date_end': values['date_planned'],
            'user_id': False,
            'warehouse_id': values.get('warehouse_id') and values['warehouse_id'].id or False,
            'procurement_group_id': values.get('group_id') and values['group_id'].id or False,
            'company_id': company_id.id,
            'line_ids': [(0, 0, {
                'product_id': product_id.id,
                'product_uom_id': product_uom.id,
                'product_qty': product_qty,
                'product_description_variants': values.get('product_description_variants'),
                'move_dest_id': values.get('move_dest_ids') and values['move_dest_ids'][0].id or False
            })],
        }


class PurchaseRequisitionLine(models.Model):
    _inherit = "purchase.requisition.line"

    move_dest_id = fields.Many2one('stock.move', 'Downstream Move')

    def _prepare_purchase_order_line(self, name, product_qty=0.0, price_unit=0.0, taxes_ids=False):
        res = super(PurchaseRequisitionLine, self)._prepare_purchase_order_line(name, product_qty, price_unit, taxes_ids)
        res['move_dest_ids'] = self.move_dest_id and [(4, self.move_dest_id.id)] or []
        return res

```

## File: models\stock.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import api, fields, models


class StockRule(models.Model):
    _inherit = 'stock.rule'

    @api.model
    def _run_buy(self, procurements):
        requisitions_values_by_company = defaultdict(list)
        other_procurements = []
        for procurement, rule in procurements:
            if procurement.product_id.purchase_requisition == 'tenders':
                values = self.env['purchase.requisition']._prepare_tender_values(*procurement)
                values['picking_type_id'] = rule.picking_type_id.id
                requisitions_values_by_company[procurement.company_id.id].append(values)
            else:
                other_procurements.append((procurement, rule))
        for company_id, requisitions_values in requisitions_values_by_company.items():
            self.env['purchase.requisition'].sudo().with_company(company_id).create(requisitions_values)
        return super(StockRule, self)._run_buy(other_procurements)

    def _prepare_purchase_order(self, company_id, origins, values):
        res = super(StockRule, self)._prepare_purchase_order(company_id, origins, values)
        values = values[0]
        res['partner_ref'] = values['supplier'].purchase_requisition_id.name
        res['requisition_id'] = values['supplier'].purchase_requisition_id.id
        if values['supplier'].purchase_requisition_id.currency_id:
            res['currency_id'] = values['supplier'].purchase_requisition_id.currency_id.id
        return res

    def _make_po_get_domain(self, company_id, values, partner):
        domain = super(StockRule, self)._make_po_get_domain(company_id, values, partner)
        if 'supplier' in values and values['supplier'].purchase_requisition_id:
            domain += (
                ('requisition_id', '=', values['supplier'].purchase_requisition_id.id),
            )
        return domain


class StockMove(models.Model):
    _inherit = 'stock.move'

    requisition_line_ids = fields.One2many('purchase.requisition.line', 'move_dest_id')

    def _get_upstream_documents_and_responsibles(self, visited):
        # People without purchase rights should be able to do this operation
        requisition_lines_sudo = self.sudo().requisition_line_ids
        if requisition_lines_sudo:
            return [(requisition_line.requisition_id, requisition_line.requisition_id.user_id, visited) for requisition_line in requisition_lines_sudo if requisition_line.requisition_id.state not in ('done', 'cancel')]
        else:
            return super(StockMove, self)._get_upstream_documents_and_responsibles(visited)


class Orderpoint(models.Model):
    _inherit = "stock.warehouse.orderpoint"

    def _quantity_in_progress(self):
        res = super(Orderpoint, self)._quantity_in_progress()
        for op in self:
            for pr in self.env['purchase.requisition'].search([('state', '=', 'draft'), ('origin', '=', op.name)]):
                for prline in pr.line_ids.filtered(lambda l: l.product_id.id == op.product_id.id and not l.move_dest_id):
                    res[op.id] += prline.product_uom_id._compute_quantity(prline.product_qty, op.product_uom, round=False)
        return res

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import purchase
from . import stock
from . import purchase_requisition

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_purchase_requisition_stock_manager,purchase.requisition,purchase_requisition.model_purchase_requisition,stock.group_stock_manager,1,0,1,0
access_purchase_requisition_line_stock_manager,purchase.requisition.line,purchase_requisition.model_purchase_requisition_line,stock.group_stock_manager,1,0,1,0


```

## File: views\purchase_requisition_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_purchase_requisition_form_inherit" model="ir.ui.view">
            <field name="name">purchase.requisition.form.inherit</field>
            <field name="model">purchase.requisition</field>
            <field name="inherit_id" ref="purchase_requisition.view_purchase_requisition_form" />
            <field name="arch" type="xml">
                <field name="origin" position="after">
                    <field name="picking_type_id" options="{'no_open': True, 'no_create': True}" groups="stock.group_adv_location" attrs="{'readonly': [('state', '!=', 'draft')]}"/>
                </field>
            </field>
        </record>
    </data>
</odoo>

```

