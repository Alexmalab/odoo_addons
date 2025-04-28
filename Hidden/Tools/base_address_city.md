# Odoo Module: base_address_city

Category: Hidden/Tools

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
    'name': 'City Addresses',
    'summary': 'Add a many2one field city on addresses',
    'sequence': '19',
    'category': 'Hidden/Tools',
    'complexity': 'easy',
    'description': """
City Management in Addresses
============================

This module allows to enforce users to choose the city of a partner inside a given list instead of a free text field.
        """,
    'data': [
        'security/ir.model.access.csv',
        'views/res_city_view.xml',
        'views/res_country_view.xml',
    ],
    'depends': ['base'],
    'license': 'LGPL-3',
}

```

## File: models\res_city.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class City(models.Model):
    _name = 'res.city'
    _description = 'City'
    _order = 'name'

    name = fields.Char("Name", required=True, translate=True)
    zipcode = fields.Char("Zip")
    country_id = fields.Many2one('res.country', string='Country', required=True)
    state_id = fields.Many2one(
        'res.country.state', 'State', domain="[('country_id', '=', country_id)]")

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

from lxml import etree

from odoo import api, models, fields
from odoo.tools.translate import _


class Partner(models.Model):
    _inherit = 'res.partner'

    country_enforce_cities = fields.Boolean(related='country_id.enforce_cities', readonly=True)
    city_id = fields.Many2one('res.city', string='City of Address')

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

    @api.model
    def _address_fields(self):
        """Returns the list of address fields that are synced from the parent."""
        return super(Partner, self)._address_fields() + ['city_id',]

    @api.model
    def _fields_view_get_address(self, arch):
        arch = super(Partner, self)._fields_view_get_address(arch)
        if self.env.context.get('no_address_format'):
            return arch
        # render the partner address accordingly to address_view_id
        doc = etree.fromstring(arch)
        if doc.xpath("//field[@name='city_id']"):
            return arch

        replacement_xml = """
            <div>
                <field name="country_enforce_cities" invisible="1"/>
                <field name="type" invisible="1"/>
                <field name="parent_id" invisible="1"/>
                <field name='city' placeholder="%(placeholder)s" class="o_address_city"
                    attrs="{
                        'invisible': [('country_enforce_cities', '=', True), '|', ('city_id', '!=', False), ('city', 'in', ['', False ])],
                        'readonly': [('type', '=', 'contact')%(parent_condition)s]
                    }"
                />
                <field name='city_id' placeholder="%(placeholder)s" string="%(placeholder)s" class="o_address_city"
                    context="{'default_country_id': country_id,
                              'default_name': city,
                              'default_zipcode': zip,
                              'default_state_id': state_id}"
                    domain="[('country_id', '=', country_id)]"
                    attrs="{
                        'invisible': [('country_enforce_cities', '=', False)],
                        'readonly': [('type', '=', 'contact')%(parent_condition)s]
                    }"
                />
            </div>
        """

        replacement_data = {
            'placeholder': _('City'),
        }

        def _arch_location(node):
            in_subview = False
            view_type = False
            parent = node.getparent()
            while parent is not None and (not view_type or not in_subview):
                if parent.tag == 'field':
                    in_subview = True
                elif parent.tag in ['list', 'tree', 'kanban', 'form']:
                    view_type = parent.tag
                parent = parent.getparent()
            return {
                'view_type': view_type,
                'in_subview': in_subview,
            }

        for city_node in doc.xpath("//field[@name='city']"):
            location = _arch_location(city_node)
            replacement_data['parent_condition'] = ''
            if location['view_type'] == 'form' or not location['in_subview']:
                replacement_data['parent_condition'] = ", ('parent_id', '!=', False)"

            replacement_formatted = replacement_xml % replacement_data
            for replace_node in etree.fromstring(replacement_formatted).getchildren():
                city_node.addprevious(replace_node)
            parent = city_node.getparent()
            parent.remove(city_node)

        arch = etree.tostring(doc, encoding='unicode')
        return arch

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
"access_res_city_group_all","res_city group_user_all","model_res_city",,1,0,0,0

```

## File: views\res_city_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_city_tree" model="ir.ui.view">
            <field name="model">res.city</field>
            <field name="arch" type="xml">
                <tree string="City" editable="top">
                    <field name="name"/>
                    <field name="zipcode"/>
                    <field name="country_id"/>
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
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">res.city</field>
            <field name="view_mode">tree</field>
            <field name="help">
                Display and manage the list of all cities that can be assigned to
                your partner records. Note that an option can be set on each country separately
                to enforce any address of it to have a city in this list.
            </field>
        </record>
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
                    context="{'default_country_id': active_id, 'search_default_country_id': active_id}"
                    string="Cities">
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

