# Odoo Module: base_address_extended

Category: Hidden

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
    'name': 'Extended Addresses',
    'summary': 'Add extra fields on addresses',
    'sequence': 19,
    'version': '1.1',
    'category': 'Hidden',
    'description': """
Extended Addresses Management
=============================

This module provides the ability to choose a city from a list (in specific countries).

It is primarily used for EDIs that might need a special city code.
        """,
    'data': [
        'security/ir.model.access.csv',
        'views/base_address_extended.xml',
        'views/res_city_view.xml',
        'views/res_country_view.xml',
    ],
    'depends': ['base', 'contacts'],
    'license': 'LGPL-3',
}

```

## File: models\res_city.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.osv import expression

class City(models.Model):
    _name = 'res.city'
    _description = 'City'
    _order = 'name'
    _rec_names_search = ['name', 'zipcode']

    name = fields.Char("Name", required=True, translate=True)
    zipcode = fields.Char("Zip")
    country_id = fields.Many2one(comodel_name='res.country', string='Country', required=True)
    state_id = fields.Many2one(comodel_name='res.country.state', string='State', domain="[('country_id', '=', country_id)]")

    @api.depends('zipcode')
    def _compute_display_name(self):
        for city in self:
            name = city.name if not city.zipcode else f'{city.name} ({city.zipcode})'
            city.display_name = name

```

## File: models\res_country.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Country(models.Model):
    _inherit = 'res.country'

    enforce_cities = fields.Boolean(
        string='Enforce Cities',
        help="Check this box to ensure every address created in that country has a 'City' chosen "
             "in the list of the country's cities.")

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools

class Partner(models.Model):
    _inherit = ['res.partner']

    street_name = fields.Char(
        'Street Name', compute='_compute_street_data', inverse='_inverse_street_data', store=True)
    street_number = fields.Char(
        'House', compute='_compute_street_data', inverse='_inverse_street_data', store=True)
    street_number2 = fields.Char(
        'Door', compute='_compute_street_data', inverse='_inverse_street_data', store=True)

    city_id = fields.Many2one(comodel_name='res.city', string='City ID')
    country_enforce_cities = fields.Boolean(related='country_id.enforce_cities')

    @api.model
    def _address_fields(self):
        return super()._address_fields() + ['city_id']

    def _inverse_street_data(self):
        """ update self.street based on street_name, street_number and street_number2 """
        for partner in self:
            street = ((partner.street_name or '') + " " + (partner.street_number or '')).strip()
            if partner.street_number2:
                street = street + " - " + partner.street_number2
            partner.street = street

    @api.depends('street')
    def _compute_street_data(self):
        """Splits street value into sub-fields.
        Recomputes the fields of STREET_FIELDS when `street` of a partner is updated"""
        for partner in self:
            partner.update(tools.street_split(partner.street))

    def _get_street_split(self):
        self.ensure_one()
        return {
            'street_name': self.street_name,
            'street_number': self.street_number,
            'street_number2': self.street_number2
        }

    @api.onchange('city_id')
    def _onchange_city_id(self):
        if self.city_id:
            self.city = self.city_id.name
            self.zip = self.city_id.zipcode
            self.state_id = self.city_id.state_id
        elif self._origin:
            self.city = False
            self.zip = False
            self.state_id = False

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_city
from . import res_country
from . import res_partner

```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
"access_res_city_group_user","res_city group_user","model_res_city","base.group_partner_manager",1,1,1,1
"access_res_city_group_all","res_city group_user_all","model_res_city","base.group_user",1,0,0,0

```

## File: views\base_address_extended.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo>

    <record id="address_street_extended_form" model="ir.ui.view">
        <field name="name">partner.form.address.extended</field>
        <field name="model">res.partner</field>
        <field name="priority" eval="900"/>
        <field name="arch" type="xml">
            <form>
                <div class="o_address_format">
                    <field name="country_enforce_cities" invisible="1"/>
                    <field name="parent_id" invisible="1"/>
                    <field name="type" invisible="1"/>
                    <field name="street" placeholder="Street..." class="o_address_street oe_read_only"
                           readonly="type == 'contact' and parent_id"/>
                    <div class="oe_edit_only o_row">
                        <field name="street_name" placeholder="Street" style="flex: 3 1 auto"
                               readonly="type == 'contact' and parent_id"/>
                        <span> </span>
                        <field name="street_number" placeholder="House #" style="flex: 1 1 auto"
                               readonly="type == 'contact' and parent_id"/>
                        <span> - </span>
                        <field name="street_number2" placeholder="Door #" style="flex: 1 1 auto"
                               readonly="type == 'contact' and parent_id"/>
                    </div>
                    <field name="street2" placeholder="Street 2..." class="o_address_street"
                           readonly="type == 'contact' and parent_id"/>
                    <field name="city_id"
                           placeholder="City"
                           class="o_address_city"
                           domain="[('country_id', '=', country_id)]"
                           invisible="not country_enforce_cities"
                           readonly="type == 'contact' and parent_id"
                           context="{'default_country_id': country_id, 'default_state_id': state_id, 'default_zipcode': zip}"/>
                    <field name="city"
                           placeholder="City"
                           class="o_address_city"
                           invisible="country_enforce_cities and (city_id or city in ('', False))"
                           readonly="type == 'contact' and parent_id"/>
                    <field name="state_id"
                           class="o_address_state"
                           placeholder="State"
                           readonly="type == 'contact' and parent_id"
                           options="{'no_open': True, 'no_quick_create': True}"
                           context="{'default_country_id': country_id}"/>
                    <field name="zip" placeholder="ZIP" class="o_address_zip"
                           readonly="type == 'contact' and parent_id"/>
                    <field name="country_id"
                           placeholder="Country"
                           class="o_address_country"
                           readonly="type == 'contact' and parent_id"
                           options="{'no_open': True, 'no_create': True}"/>
                </div>
            </form>
        </field>
    </record>

    <record id="address_street_extended_city_form" model="ir.ui.view">
        <field name="name">partner.form.address.extended.city_id</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <field name="city" position="attributes">
                <attribute name="invisible">country_enforce_cities and (city_id or city in ('', False))</attribute>
                <attribute name="readonly">type == 'contact' and parent_id</attribute>
            </field>
            <field name="city" position="before">
                <field name="country_enforce_cities" invisible="1"/>
                <field name="city_id"
                       placeholder="City"
                       class="o_address_city"
                       domain="[('country_id', '=', country_id)]"
                       invisible="not country_enforce_cities"
                       readonly="type == 'contact' and parent_id"
                       context="{'default_country_id': country_id, 'default_state_id': state_id, 'default_zipcode': zip}"/>
            </field>

            <xpath expr="//field[@name='child_ids']//form//field[@name='city']" position="attributes">
                <attribute name="invisible">country_enforce_cities and (city_id or city in ('', False))</attribute>
                <attribute name="readonly">type == 'contact' and parent_id</attribute>
            </xpath>
            <xpath expr="//field[@name='child_ids']//form//field[@name='city']" position="before">
                <field name="country_enforce_cities" invisible="1"/>
                <field name="city_id"
                       placeholder="City"
                       class="o_address_city"
                       domain="[('country_id', '=', country_id)]"
                       invisible="not country_enforce_cities"
                       readonly="type == 'contact' and parent_id"
                       context="{'default_country_id': country_id, 'default_state_id': state_id, 'default_zipcode': zip}"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\res_city_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_city_tree" model="ir.ui.view">
            <field name="name">res.city.tree</field>
            <field name="model">res.city</field>
            <field name="arch" type="xml">
                <tree string="City" editable="top">
                    <field name="name"/>
                    <field name="zipcode"/>
                    <field name="country_id" options="{'no_open': True, 'no_create': True}"/>
                    <field name="state_id" context="{'default_country_id': country_id}"/>
                </tree>
            </field>
        </record>
        <record id="view_city_filter" model="ir.ui.view">
            <field name="model">res.city</field>
            <field name="arch" type="xml">
                <search string="Search City">
                    <field name="name" filter_domain="['|', ('name','ilike',self), ('zipcode','ilike',self)]"
                           string="City"/>
                    <separator/>
                    <field name="country_id"/>
                </search>
            </field>
        </record>

        <record id="action_res_city_tree" model="ir.actions.act_window">
            <field name="name">Cities</field>
            <field name="res_model">res.city</field>
            <field name="view_mode">tree</field>
            <field name="help">
                Display and manage the list of all cities that can be assigned to
                your partner records. Note that an option can be set on each country separately
                to enforce any address of it to have a city in this list.
            </field>
        </record>

        <menuitem id="menu_res_city"
            action="action_res_city_tree"
            parent="contacts.menu_localisation"
            sequence="2"/>

    </data>
</odoo>

```

## File: views\res_country_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
    <record id="view_res_country_city_extended_form" model="ir.ui.view">
        <field name="model">res.country</field>
        <field name="inherit_id" ref="base.view_country_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
                <button name="%(action_res_city_tree)d"
                    class="oe_stat_button"
                    icon="fa-globe"
                    type="action"
                    context="{'default_country_id': id, 'search_default_country_id': id}">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_text">Cities</span>
                    </div>
                </button>
            </xpath>
            <xpath expr="//field[@name='phone_code']" position="after">
                <field name="enforce_cities"/>
            </xpath>
        </field>
    </record>
    </data>
</odoo>

```

