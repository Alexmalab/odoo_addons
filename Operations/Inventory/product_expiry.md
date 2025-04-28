# Odoo Module: product_expiry

Category: Operations/Inventory

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Products Expiration Date',
    'category': 'Operations/Inventory',
    'depends': ['stock'],
    'demo': [],
    'description': """
Track different dates on products and production lots.
======================================================

Following dates can be tracked:
-------------------------------
    - end of life
    - best before date
    - removal date
    - alert date

Also implements the removal strategy First Expiry First Out (FEFO) widely used, for example, in food industries.
""",
    'data': ['views/production_lot_views.xml',
             'views/product_template_views.xml',
             'views/stock_quant_views.xml',
             'data/product_expiry_data.xml'],
    'license': 'LGPL-3',
}

```

## File: data\product_expiry_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="removal_fefo" model="product.removal">
            <field name="name">First Expiry First Out (FEFO)</field>
            <field name="method">fefo</field>
        </record>
        <record id="mail_activity_type_alert_date_reached" model="mail.activity.type">
            <field name="name">Alert Date Reached</field>
            <field name="category">default</field>
            <field name="res_model_id" ref="stock.model_stock_production_lot"/>
            <field name="icon">fa-tasks</field>
            <field name="delay_count">0</field>
        </record>
</odoo>


```

## File: models\production_lot.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import datetime
from odoo import api, fields, models, SUPERUSER_ID, _


class StockProductionLot(models.Model):
    _inherit = 'stock.production.lot'

    life_date = fields.Datetime(string='End of Life Date',
        help='This is the date on which the goods with this Serial Number may become dangerous and must not be consumed.')
    use_date = fields.Datetime(string='Best before Date',
        help='This is the date on which the goods with this Serial Number start deteriorating, without being dangerous yet.')
    removal_date = fields.Datetime(string='Removal Date',
        help='This is the date on which the goods with this Serial Number should be removed from the stock. This date will be used in FEFO removal strategy.')
    alert_date = fields.Datetime(string='Alert Date',
        help='Date to determine the expired lots and serial numbers using the filter "Expiration Alerts".')
    product_expiry_alert = fields.Boolean(compute='_compute_product_expiry_alert', help="The Alert Date has been reached.")
    product_expiry_reminded = fields.Boolean(string="Expiry has been reminded")

    @api.depends('alert_date')
    def _compute_product_expiry_alert(self):
        current_date = fields.Datetime.now()
        lots = self.filtered(lambda l: l.alert_date)
        for lot in lots:
            lot.product_expiry_alert = lot.alert_date <= current_date
        (self - lots).product_expiry_alert = False

    def _get_dates(self, product_id=None):
        """Returns dates based on number of days configured in current lot's product."""
        mapped_fields = {
            'life_date': 'life_time',
            'use_date': 'use_time',
            'removal_date': 'removal_time',
            'alert_date': 'alert_time'
        }
        res = dict.fromkeys(mapped_fields, False)
        product = self.env['product.product'].browse(product_id) or self.product_id
        if product:
            for field in mapped_fields:
                duration = getattr(product, mapped_fields[field])
                if duration:
                    date = datetime.datetime.now() + datetime.timedelta(days=duration)
                    res[field] = fields.Datetime.to_string(date)
        return res

    # Assign dates according to products data
    @api.model
    def create(self, vals):
        dates = self._get_dates(vals.get('product_id') or self.env.context.get('default_product_id'))
        for d in dates:
            if not vals.get(d):
                vals[d] = dates[d]
        return super(StockProductionLot, self).create(vals)

    @api.onchange('product_id')
    def _onchange_product(self):
        dates_dict = self._get_dates()
        for field, value in dates_dict.items():
            setattr(self, field, value)

    @api.model
    def _alert_date_exceeded(self):
        """Log an activity on internally stored lots whose alert_date has been reached.

        No further activity will be generated on lots whose alert_date
        has already been reached (even if the alert_date is changed).
        """
        alert_lots = self.env['stock.production.lot'].search([
            ('alert_date', '<=', fields.Date.today()),
            ('product_expiry_reminded', '=', False)])

        lot_stock_quants = self.env['stock.quant'].search([
            ('lot_id', 'in', alert_lots.ids),
            ('quantity', '>', 0),
            ('location_id.usage', '=', 'internal')])
        alert_lots = lot_stock_quants.mapped('lot_id')

        for lot in alert_lots:
            lot.activity_schedule(
                'product_expiry.mail_activity_type_alert_date_reached',
                user_id=lot.product_id.responsible_id.id or SUPERUSER_ID,
                note=_("The alert date has been reached for this lot/serial number")
            )
        alert_lots.write({
            'product_expiry_reminded': True
        })


class ProcurementGroup(models.Model):
    _inherit = 'procurement.group'

    @api.model
    def _run_scheduler_tasks(self, use_new_cursor=False, company_id=False):
        super(ProcurementGroup, self)._run_scheduler_tasks(use_new_cursor=use_new_cursor, company_id=company_id)
        self.env['stock.production.lot']._alert_date_exceeded()
        if use_new_cursor:
            self.env.cr.commit()

```

## File: models\product_product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    life_time = fields.Integer(string='Product Life Time',
        help='Number of days before the goods may become dangerous and must not be consumed. It will be computed on the lot/serial number.')
    use_time = fields.Integer(string='Product Use Time',
        help='Number of days before the goods starts deteriorating, without being dangerous yet. It will be computed using the lot/serial number.')
    removal_time = fields.Integer(string='Product Removal Time',
        help='Number of days before the goods should be removed from the stock. It will be computed on the lot/serial number.')
    alert_time = fields.Integer(string='Product Alert Time',
        help='Number of days before an alert should be raised on the lot/serial number.')

```

## File: models\stock_quant.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class StockQuant(models.Model):
    _inherit = 'stock.quant'

    removal_date = fields.Datetime(related='lot_id.removal_date', store=True, readonly=False)

    @api.model
    def _get_removal_strategy_order(self, removal_strategy):
        if removal_strategy == 'fefo':
            return 'removal_date, in_date, id'
        return super(StockQuant, self)._get_removal_strategy_order(removal_strategy)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import production_lot
from . import product_product
from . import stock_quant

```

## File: views\production_lot_views.xml

```xml
<?xml version="1.0" encoding='UTF-8'?>
<odoo>
    <record id="view_move_form_expiry" model="ir.ui.view">
        <field name="name">stock.production.lot.inherit.form</field>
        <field name="model">stock.production.lot</field>
        <field name="inherit_id" ref="stock.view_production_lot_form" />
        <field name="arch" type="xml">
        <xpath expr="//page[@name='description']" position="before">
            <page string="Dates">
                <group>
                    <group>
                        <field name="use_date" />
                        <field name="removal_date" />
                    </group>
                    <group>
                        <field name="life_date" />
                        <field name="alert_date" />
                    </group>
                </group>
            </page>
        </xpath>
        <xpath expr="//div[hasclass('oe_title')]" position="inside">
            <field name="product_expiry_alert" invisible="1"/>
            <span class="badge badge-danger" attrs="{'invisible': [('product_expiry_alert', '=', False)]}">Expiration Alert</span>
        </xpath>
        </field>
    </record>

    <record id="search_product_lot_filter_inherit_product_expiry" model="ir.ui.view">
        <field name="name">stock.production.lot.search.inherit</field>
        <field name="model">stock.production.lot</field>
        <field name="inherit_id" ref="stock.search_product_lot_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='product_id']" position="after">
                <filter string="Expiration Alerts" name="expiration_alerts" domain="[('alert_date', '&lt;=', time.strftime('%Y-%m-%d %H:%M:%S'))]"/>
            </xpath>
        </field>
     </record>

    <record id="view_production_lot_view_tree" model="ir.ui.view">
        <field name="name">stock.production.lot.tree.inherit.product.expiry</field>
        <field name="model">stock.production.lot</field>
        <field name="inherit_id" ref="stock.view_production_lot_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='create_date']" position="after">
                <field name="use_date" optional="hide"/>
                <field name="removal_date" optional="hide"/>
                <field name="life_date" optional="hide"/>
                <field name="alert_date" optional="hide"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\product_template_views.xml

```xml
<?xml version="1.0" encoding='UTF-8'?>
<odoo>
    <record id="view_product_form_expiry" model="ir.ui.view">
            <field name="name">product.template.inherit.form</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="stock.view_template_property_form" />
            <field name="arch" type="xml">
                <group name="traceability" position="after">
                    <group string="Dates" name="expiry_and_lots" groups="stock.group_production_lot" attrs="{'invisible': [('tracking', '=','none')]}">
                        <label for="use_time"/>
                        <div>
                            <field name="use_time" class="oe_inline"/>
                            <span> days</span>
                        </div>
                        <label for="life_time"/>
                        <div>
                            <field name="life_time" class="oe_inline"/>
                            <span> days</span>
                        </div>
                        <label for="removal_time"/>
                        <div>
                            <field name="removal_time" class="oe_inline"/>
                            <span> days</span>
                        </div>
                        <label for="alert_time"/>
                        <div>
                            <field name="alert_time" class="oe_inline"/>
                            <span> days</span>
                        </div>
                    </group>
                </group>
            </field>
        </record>
</odoo>

```

## File: views\stock_quant_views.xml

```xml
<?xml version="1.0" encoding='UTF-8'?>
<odoo>
        <record id="view_quant_form_expiry" model="ir.ui.view">
            <field name="name">stock.quant.inherit.form</field>
            <field name="model">stock.quant</field>
            <field name="inherit_id" ref="stock.view_stock_quant_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='lot_id']" position="after" >
                    <field name="removal_date"/>
                </xpath>
            </field>
        </record>
</odoo>

```

