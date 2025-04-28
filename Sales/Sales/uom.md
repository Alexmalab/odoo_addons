# Odoo Module: uom

Category: Sales/Sales

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Units of measure',
    'version': '1.0',
    'category': 'Sales/Sales',
    'depends': ['base'],
    'description': """
This is the base module for managing Units of measure.
========================================================================
    """,
    'data': [
        'data/uom_data.xml',
        'security/uom_security.xml',
        'security/ir.model.access.csv',
        'views/uom_uom_views.xml',
    ],
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: data\uom_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!--
        Resource: uom.category
        -->
        <record id="product_uom_categ_unit" model="uom.category">
            <field name="name">Unit</field>
            <field name="measure_type">unit</field>
        </record>
        <record id="product_uom_categ_kgm" model="uom.category">
            <field name="name">Weight</field>
            <field name="measure_type">weight</field>
        </record>
        <record id="uom_categ_wtime" model="uom.category">
            <field name="name">Working Time</field>
            <field name="measure_type">working_time</field>
        </record>
        <record id="uom_categ_length" model="uom.category">
            <field name="name">Length / Distance</field>
            <field name="measure_type">length</field>
        </record>
        <record id="product_uom_categ_vol" model="uom.category">
            <field name="name">Volume</field>
            <field name="measure_type">volume</field>
        </record>
       <!--
            Resource: uom.uom
        -->
        <record id="product_uom_unit" model="uom.uom">
            <field name="category_id" ref="product_uom_categ_unit"/>
            <field name="name">Units</field>
            <field name="factor" eval="1.0"/>
            <field name="rounding" eval="0.001"/>
        </record>
       <record id="product_uom_dozen" model="uom.uom">
            <field name="category_id" ref="uom.product_uom_categ_unit"/>
            <field name="name">Dozens</field>
            <field name="factor_inv" eval="12"/>
            <field name="uom_type">bigger</field>
        </record>

        <record id="product_uom_kgm" model="uom.uom">
            <field name="category_id" ref="product_uom_categ_kgm"/>
            <field name="name">kg</field>
            <field name="factor" eval="1"/>
            <field name="rounding" eval="0.001"/>
        </record>
        <record id="product_uom_gram" model="uom.uom">
            <field name="category_id" ref="product_uom_categ_kgm"/>
            <field name="name">g</field>
            <field name="factor" eval="1000"/>
            <field name="uom_type">smaller</field>
        </record>

        <record id="product_uom_day" model="uom.uom">
            <field name="name">Days</field>
            <field eval="uom_categ_wtime" name="category_id"/>
            <field name="factor" eval="1.0"/>
        </record>
        <record id="product_uom_hour" model="uom.uom">
            <field name="name">Hours</field>
            <field eval="uom_categ_wtime" name="category_id"/>
            <field name="factor" eval="8.0"/>
            <field name="uom_type">smaller</field>
        </record>
        <record id="product_uom_ton" model="uom.uom">
            <field name="category_id" ref="product_uom_categ_kgm"/>
            <!-- 'tonne' is the most common spelling in english-speaking countries,
                 the alternative is 'metric ton' in the US, abbreviated as 'mt' -->
            <field name="name">t</field>
            <field name="factor_inv" eval="1000"/>
            <field name="uom_type">bigger</field>
        </record>
        <record id="product_uom_meter" model="uom.uom">
            <field name="category_id" ref="uom_categ_length"/>
            <field name="name">m</field>
            <field name="factor" eval="1.0"/>
        </record>
        <record id="product_uom_km" model="uom.uom">
            <field name="category_id" ref="uom_categ_length"/>
            <field name="name">km</field>
            <field name="factor_inv" eval="1000"/>
            <field name="uom_type">bigger</field>
        </record>
        <record id="product_uom_cm" model="uom.uom">
            <field name="category_id" ref="uom_categ_length"/>
            <field name="name">cm</field>
            <field name="factor" eval="100"/>
            <field name="uom_type">smaller</field>
        </record>
        <record id="product_uom_litre" model="uom.uom">
            <field name="name">Liters</field>
            <field name="category_id" ref="product_uom_categ_vol"/>
            <field name="factor">1.0</field>
        </record>

        <!--Americanization of units of measure-->
        <record id="product_uom_lb" model="uom.uom">
            <field name="name">lbs</field>
            <field name="category_id" ref="product_uom_categ_kgm"/>
            <field name="factor">2.20462</field>
            <field name="uom_type">smaller</field>
        </record>
        <record id="product_uom_oz" model="uom.uom">
            <field name="name">ozs</field>
            <field name="category_id" ref="product_uom_categ_kgm"/>
            <field name="factor">35.274</field>
            <field name="uom_type">smaller</field>
        </record>
        <record id="product_uom_inch" model="uom.uom">
            <field name="name">inches</field>
            <field name="category_id" ref="uom_categ_length"/>
            <field name="factor">39.3701</field>
            <field name="uom_type">smaller</field>
        </record>
        <record id="product_uom_foot" model="uom.uom">
            <field name="name">foot(ft)</field>
            <field name="category_id" ref="uom_categ_length"/>
            <field name="factor">3.28084</field>
            <field name="uom_type">smaller</field>
        </record>
        <record id="product_uom_mile" model="uom.uom">
            <field name="name">miles</field>
            <field name="category_id" ref="uom_categ_length"/>
            <field name="factor_inv" eval="1609.34"/>
            <field name="uom_type">bigger</field>
        </record>
        <record id="product_uom_floz" model="uom.uom">
            <field name="name">fl oz</field>
            <field name="category_id" ref="product_uom_categ_vol"/>
            <field name="factor">33.814</field>
            <field name="uom_type">smaller</field>
        </record>
        <record id="product_uom_qt" model="uom.uom">
            <field name="name">qt</field>
            <field name="category_id" ref="product_uom_categ_vol"/>
            <field name="factor">1.05669</field>
            <field name="uom_type">smaller</field>
        </record>
        <record id="product_uom_gal" model="uom.uom">
            <field name="name">gals</field>
            <field name="category_id" ref="product_uom_categ_vol"/>
            <field name="factor_inv" eval="3.78541"/>
            <field name="uom_type">bigger</field>
        </record>
    </data>
</odoo>

```

## File: models\uom_uom.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, tools, models, _
from odoo.exceptions import UserError, ValidationError


class UoMCategory(models.Model):
    _name = 'uom.category'
    _description = 'Product UoM Categories'

    name = fields.Char('Unit of Measure Category', required=True, translate=True)
    measure_type = fields.Selection([
        ('unit', 'Default Units'),
        ('weight', 'Default Weight'),
        ('working_time', 'Default Working Time'),
        ('length', 'Default Length'),
        ('volume', 'Default Volume'),
    ], string="Type of Measure")

    _sql_constraints = [
        ('uom_category_unique_type', 'UNIQUE(measure_type)', 'You can have only one category per measurement type.'),
    ]

    def unlink(self):
        if self.filtered(lambda categ: categ.measure_type == 'working_time'):
            raise UserError(_("You cannot delete this UoM Category as it is used by the system."))
        return super(UoMCategory, self).unlink()


class UoM(models.Model):
    _name = 'uom.uom'
    _description = 'Product Unit of Measure'
    _order = "name"

    name = fields.Char('Unit of Measure', required=True, translate=True)
    category_id = fields.Many2one(
        'uom.category', 'Category', required=True, ondelete='cascade',
        help="Conversion between Units of Measure can only occur if they belong to the same category. The conversion will be made based on the ratios.")
    factor = fields.Float(
        'Ratio', default=1.0, digits=0, required=True,  # force NUMERIC with unlimited precision
        help='How much bigger or smaller this unit is compared to the reference Unit of Measure for this category: 1 * (reference unit) = ratio * (this unit)')
    factor_inv = fields.Float(
        'Bigger Ratio', compute='_compute_factor_inv', digits=0,  # force NUMERIC with unlimited precision
        readonly=True, required=True,
        help='How many times this Unit of Measure is bigger than the reference Unit of Measure in this category: 1 * (this unit) = ratio * (reference unit)')
    rounding = fields.Float(
        'Rounding Precision', default=0.01, digits=0, required=True,
        help="The computed quantity will be a multiple of this value. "
             "Use 1.0 for a Unit of Measure that cannot be further split, such as a piece.")
    active = fields.Boolean('Active', default=True, help="Uncheck the active field to disable a unit of measure without deleting it.")
    uom_type = fields.Selection([
        ('bigger', 'Bigger than the reference Unit of Measure'),
        ('reference', 'Reference Unit of Measure for this category'),
        ('smaller', 'Smaller than the reference Unit of Measure')], 'Type',
        default='reference', required=1)
    measure_type = fields.Selection(string="Type of measurement category", related='category_id.measure_type', store=True, readonly=True)

    _sql_constraints = [
        ('factor_gt_zero', 'CHECK (factor!=0)', 'The conversion ratio for a unit of measure cannot be 0!'),
        ('rounding_gt_zero', 'CHECK (rounding>0)', 'The rounding precision must be strictly positive.'),
        ('factor_reference_is_one', "CHECK((uom_type = 'reference' AND factor = 1.0) OR (uom_type != 'reference'))", "The reference unit must have a conversion factor equal to 1.")
    ]

    @api.depends('factor')
    def _compute_factor_inv(self):
        for uom in self:
            uom.factor_inv = uom.factor and (1.0 / uom.factor) or 0.0

    @api.onchange('uom_type')
    def _onchange_uom_type(self):
        if self.uom_type == 'reference':
            self.factor = 1

    @api.constrains('category_id', 'uom_type', 'active')
    def _check_category_reference_uniqueness(self):
        """ Force the existence of only one UoM reference per category
            NOTE: this is a constraint on the all table. This might not be a good practice, but this is
            not possible to do it in SQL directly.
        """
        category_ids = self.mapped('category_id').ids
        self.env['uom.uom'].flush(['category_id', 'uom_type', 'active'])
        self._cr.execute("""
            SELECT C.id AS category_id, count(U.id) AS uom_count
            FROM uom_category C
            LEFT JOIN uom_uom U ON C.id = U.category_id AND uom_type = 'reference' AND U.active = 't'
            WHERE C.id IN %s
            GROUP BY C.id
        """, (tuple(category_ids),))
        for uom_data in self._cr.dictfetchall():
            if uom_data['uom_count'] == 0:
                raise ValidationError(_("UoM category %s should have a reference unit of measure. If you just created a new category, please record the 'reference' unit first.") % (self.env['uom.category'].browse(uom_data['category_id']).name,))
            if uom_data['uom_count'] > 1:
                raise ValidationError(_("UoM category %s should only have one reference unit of measure.") % (self.env['uom.category'].browse(uom_data['category_id']).name,))

    @api.model_create_multi
    def create(self, vals_list):
        for values in vals_list:
            if 'factor_inv' in values:
                factor_inv = values.pop('factor_inv')
                values['factor'] = factor_inv and (1.0 / factor_inv) or 0.0
        return super(UoM, self).create(vals_list)

    def write(self, values):
        if 'factor_inv' in values:
            factor_inv = values.pop('factor_inv')
            values['factor'] = factor_inv and (1.0 / factor_inv) or 0.0
        return super(UoM, self).write(values)

    def unlink(self):
        if self.filtered(lambda uom: uom.measure_type == 'working_time'):
            raise UserError(_("You cannot delete this UoM as it is used by the system. You should rather archive it."))
        return super(UoM, self).unlink()

    @api.model
    def name_create(self, name):
        """ The UoM category and factor are required, so we'll have to add temporary values
        for imported UoMs """
        values = {
            self._rec_name: name,
            'factor': 1
        }
        # look for the category based on the english name, i.e. no context on purpose!
        # TODO: should find a way to have it translated but not created until actually used
        if not self._context.get('default_category_id'):
            EnglishUoMCateg = self.env['uom.category'].with_context({})
            misc_category = EnglishUoMCateg.search([('name', '=', 'Unsorted/Imported Units')])
            if misc_category:
                values['category_id'] = misc_category.id
            else:
                values['category_id'] = EnglishUoMCateg.name_create('Unsorted/Imported Units')[0]
        new_uom = self.create(values)
        return new_uom.name_get()[0]

    def _compute_quantity(self, qty, to_unit, round=True, rounding_method='UP', raise_if_failure=True):
        """ Convert the given quantity from the current UoM `self` into a given one
            :param qty: the quantity to convert
            :param to_unit: the destination UoM record (uom.uom)
            :param raise_if_failure: only if the conversion is not possible
                - if true, raise an exception if the conversion is not possible (different UoM category),
                - otherwise, return the initial quantity
        """
        if not self or not qty:
            return qty
        self.ensure_one()

        if self != to_unit and self.category_id.id != to_unit.category_id.id:
            if raise_if_failure:
                raise UserError(_('The unit of measure %s defined on the order line doesn\'t belong to the same category than the unit of measure %s defined on the product. Please correct the unit of measure defined on the order line or on the product, they should belong to the same category.') % (self.name, to_unit.name))
            else:
                return qty

        if self == to_unit:
            amount = qty
        else:
            amount = qty / self.factor
            if to_unit:
                amount = amount * to_unit.factor

        if to_unit and round:
            amount = tools.float_round(amount, precision_rounding=to_unit.rounding, rounding_method=rounding_method)

        return amount

    def _compute_price(self, price, to_unit):
        self.ensure_one()
        if not self or not price or not to_unit or self == to_unit:
            return price
        if self.category_id.id != to_unit.category_id.id:
            return price
        amount = price * self.factor
        if to_unit:
            amount = amount / to_unit.factor
        return amount

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import uom_uom

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_uom_category_manager,uom.category.manager,model_uom_category,base.group_system,1,1,1,1
access_uom_category_user,uom.category.user,model_uom_category,base.group_user,1,0,0,0
access_uom_uom_manager,uom.uom.manager,model_uom_uom,base.group_system,1,1,1,1
access_uom_uom_user,uom.uom.user,model_uom_uom,base.group_user,1,0,0,0

```

## File: security\uom_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="0">
    <record id="group_uom" model="res.groups">
        <field name="name">Manage Multiple Units of Measure</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>
</data>
</odoo>

```

## File: views\uom_uom_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_uom_tree_view" model="ir.ui.view">
        <field name="name">uom.uom.tree</field>
        <field name="model">uom.uom</field>
        <field name="arch" type="xml">
            <tree string="Units of Measure">
                <field name="name"/>
                <field name="category_id"/>
                <field name="uom_type"/>
            </tree>
        </field>
    </record>

    <record id="product_uom_form_view" model="ir.ui.view">
        <field name="name">uom.uom.form</field>
        <field name="model">uom.uom</field>
        <field name="arch" type="xml">
            <form string="Units of Measure">
                <sheet>
                    <group>
                         <group name="uom_details">
                             <field name="name"/>
                             <field name="category_id"/>
                             <field name="uom_type"/>
                             <label for="factor"
                                attrs="{'invisible':[('uom_type','!=','smaller')]}"/>
                            <div attrs="{'invisible':[('uom_type','!=','smaller')]}" class="o_row">
                                <field name="factor"
                                    digits="[42,5]"
                                    attrs="{'readonly':[('uom_type','=','bigger')]}"/>
                                <span class="oe_grey">
                                    e.g: 1*(reference unit)=ratio*(this unit)
                                </span>
                            </div>
                            <label for="factor_inv" 
                                attrs="{'invisible':[('uom_type','!=','bigger')]}"/>                           
                            <div attrs="{'invisible':[('uom_type','!=','bigger')]}" class="o_row">
                                <field name="factor_inv"
                                    digits="[42,5]"
                                    attrs="{'readonly':[('uom_type','!=','bigger')]}"/>
                                <span class="oe_grey">
                                    e.g: 1*(this unit)=ratio*(reference unit)
                                </span>
                            </div>
                         </group>
                         <group name="active_rounding">
                             <field name="active"/>
                             <field name="rounding" digits="[42, 5]"/>
                         </group>
                     </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="uom_uom_view_search" model="ir.ui.view">
        <field name="name">uom.uom.view.search</field>
        <field name="model">uom.uom</field>
        <field name="arch" type="xml">
            <search string="Search UOM">
                <field name="name"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <record id="product_uom_form_action" model="ir.actions.act_window">
        <field name="name">Units of Measure</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">uom.uom</field>
        <field name="view_id" ref="product_uom_tree_view"/>
        <field name="search_view_id" ref="uom_uom_view_search"/>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Add a new unit of measure
          </p><p>
            You must define a conversion rate between several Units of
            Measure within the same category.
          </p>
        </field>
    </record>

    <record id="product_uom_categ_form_view" model="ir.ui.view">
        <field name="name">uom.category.form</field>
        <field name="model">uom.category</field>
        <field name="arch" type="xml">
            <form string="Units of Measure categories">
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="measure_type"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="product_uom_categ_tree_view" model="ir.ui.view">
        <field name="name">uom.category.tree</field>
        <field name="model">uom.category</field>
        <field name="arch" type="xml">
            <tree string="Units of Measure categories" editable="bottom">
                <field name="name"/>
                <field name="measure_type" groups="base.group_no_one"/>
            </tree>
        </field>
    </record>

    <record id="product_uom_categ_form_action" model="ir.actions.act_window">
        <field name="name">Units of Measure Categories</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">uom.category</field>
        <field name="view_mode">tree,form</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Add a new unit of measure category
          </p><p>
            Units of measure belonging to the same category can be
            converted between each others. For example, in the category
            <i>'Time'</i>, you will have the following units of measure:
            Hours, Days.
          </p>
        </field>
    </record>
</odoo>

```

