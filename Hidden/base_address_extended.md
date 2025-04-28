# Odoo Module: base_address_extended

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from odoo import api, SUPERUSER_ID


def _update_street_format(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    specific_countries = env['res.country'].search([('street_format', '!=', '%(street_number)s/%(street_number2)s %(street_name)s')])
    env['res.partner'].search([('country_id', 'in', specific_countries.ids)])._split_street()

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Extended Addresses',
    'summary': 'Add extra fields on addresses',
    'sequence': '19',
    'category': 'Hidden',
    'complexity': 'easy',
    'description': """
Extended Addresses Management
=============================

This module holds all extra fields one may need to manage accurately addresses.

For example, in legal reports, some countries need to split the street into several fields,
with the street name, the house number, and room number.
        """,
    'data': [
        'views/base_address_extended.xml',
        'data/base_address_extended_data.xml',
    ],
    'depends': ['base'],
    'post_init_hook': '_update_street_format',
    'license': 'LGPL-3',
}

```

## File: data\base_address_extended_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="base.de" model="res.country">
            <field eval="'%(street_name)s %(street_number)s/%(street_number2)s'" name="street_format" />
        </record>
        <record id="base.nl" model="res.country">
            <field eval="'%(street_name)s %(street_number)s/%(street_number2)s'" name="street_format" />
        </record>
        <record id="base.mx" model="res.country">
            <field eval="'%(street_name)s %(street_number)s/%(street_number2)s'" name="street_format" />
        </record>
        <record id="base.hr" model="res.country">
            <field eval="'%(street_name)s %(street_number)s %(street_number2)s'" name="street_format" />
        </record>
    </data>
</odoo>

```

## File: models\base_address_extended.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

from odoo import api, fields, models, _
from odoo.exceptions import UserError

STREET_FIELDS = ['street_name', 'street_number', 'street_number2']


class ResCountry(models.Model):
    _inherit = 'res.country'

    street_format = fields.Text(
        help="Format to use for streets belonging to this country.\n\n"
             "You can use the python-style string pattern with all the fields of the street "
             "(for example, use '%(street_name)s, %(street_number)s' if you want to display "
             "the street name, followed by a comma and the house number)"
             "\n%(street_name)s: the name of the street"
             "\n%(street_number)s: the house number"
             "\n%(street_number2)s: the door number",
        default='%(street_number)s/%(street_number2)s %(street_name)s', required=True)

class Partner(models.Model):
    _inherit = ['res.partner']
    _name = 'res.partner'

    street_name = fields.Char('Street Name', compute='_split_street',
                              inverse='_set_street', store=True)
    street_number = fields.Char('House', compute='_split_street', help="House Number",
                                inverse='_set_street', store=True)
    street_number2 = fields.Char('Door', compute='_split_street', help="Door Number",
                                 inverse='_set_street', store=True)

    def _formatting_address_fields(self):
        """Returns the list of address fields usable to format addresses."""
        return super(Partner, self)._formatting_address_fields() + self.get_street_fields()

    def get_street_fields(self):
        """Returns the fields that can be used in a street format.
        Overwrite this function if you want to add your own fields."""
        return STREET_FIELDS

    def _set_street(self):
        """Updates the street field.
        Writes the `street` field on the partners when one of the sub-fields in STREET_FIELDS
        has been touched"""
        street_fields = self.get_street_fields()
        for partner in self:
            street_format = (partner.country_id.street_format or
                '%(street_number)s/%(street_number2)s %(street_name)s')
            previous_field = None
            previous_pos = 0
            street_value = ""
            separator = ""
            # iter on fields in street_format, detected as '%(<field_name>)s'
            for re_match in re.finditer(r'%\(\w+\)s', street_format):
                # [2:-2] is used to remove the extra chars '%(' and ')s'
                field_name = re_match.group()[2:-2]
                field_pos = re_match.start()
                if field_name not in street_fields:
                    raise UserError(_("Unrecognized field %s in street format.") % field_name)
                if not previous_field:
                    # first iteration: add heading chars in street_format
                    if partner[field_name]:
                        street_value += street_format[0:field_pos] + partner[field_name]
                else:
                    # get the substring between 2 fields, to be used as separator
                    separator = street_format[previous_pos:field_pos]
                    if street_value and partner[field_name]:
                        street_value += separator
                    if partner[field_name]:
                        street_value += partner[field_name]
                previous_field = field_name
                previous_pos = re_match.end()

            # add trailing chars in street_format
            street_value += street_format[previous_pos:]
            partner.street = street_value

    def _split_street_with_params(self, street_raw, street_format):
        street_fields = self.get_street_fields()
        vals = {}
        previous_pos = 0
        field_name = None
        # iter on fields in street_format, detected as '%(<field_name>)s'
        for re_match in re.finditer(r'%\(\w+\)s', street_format):
            field_pos = re_match.start()
            if not field_name:
                #first iteration: remove the heading chars
                street_raw = street_raw[field_pos:]

            # get the substring between 2 fields, to be used as separator
            separator = street_format[previous_pos:field_pos]
            field_value = None
            if separator and field_name:
                #maxsplit set to 1 to unpack only the first element and let the rest untouched
                tmp = street_raw.split(separator, 1)
                if previous_greedy in vals:
                    # attach part before space to preceding greedy field
                    append_previous, sep, tmp[0] = tmp[0].rpartition(' ')
                    street_raw = separator.join(tmp)
                    vals[previous_greedy] += sep + append_previous
                if len(tmp) == 2:
                    field_value, street_raw = tmp
                    vals[field_name] = field_value
            if field_value or not field_name:
                previous_greedy = None
                if field_name == 'street_name' and separator == ' ':
                    previous_greedy = field_name
                # select next field to find (first pass OR field found)
                # [2:-2] is used to remove the extra chars '%(' and ')s'
                field_name = re_match.group()[2:-2]
            else:
                # value not found: keep looking for the same field
                pass
            if field_name not in street_fields:
                raise UserError(_("Unrecognized field %s in street format.") % field_name)
            previous_pos = re_match.end()

        # last field value is what remains in street_raw minus trailing chars in street_format
        trailing_chars = street_format[previous_pos:]
        if trailing_chars and street_raw.endswith(trailing_chars):
            vals[field_name] = street_raw[:-len(trailing_chars)]
        else:
            vals[field_name] = street_raw
        return vals


    @api.depends('street')
    def _split_street(self):
        """Splits street value into sub-fields.
        Recomputes the fields of STREET_FIELDS when `street` of a partner is updated"""
        street_fields = self.get_street_fields()
        for partner in self:
            if not partner.street:
                for field in street_fields:
                    partner[field] = None
                continue

            street_format = (partner.country_id.street_format or
                '%(street_number)s/%(street_number2)s %(street_name)s')
            street_raw = partner.street
            vals = self._split_street_with_params(street_raw, street_format)
            # assign the values to the fields
            for k, v in vals.items():
                partner[k] = v
            for k in set(street_fields) - set(vals):
                partner[k] = None

    def write(self, vals):
        res = super(Partner, self).write(vals)
        if 'country_id' in vals and 'street' not in vals:
            self._set_street()
        return res


class Company(models.Model):
    _inherit = 'res.company'

    street_name = fields.Char('Street Name', compute='_compute_address',
                              inverse='_inverse_street_name')
    street_number = fields.Char('House Number', compute='_compute_address',
                                inverse='_inverse_street_number')
    street_number2 = fields.Char('Door Number', compute='_compute_address',
                                 inverse='_inverse_street_number2')

    def _get_company_address_fields(self, partner):
        address_fields = super(Company, self)._get_company_address_fields(partner)
        address_fields.update({
            'street_name': partner.street_name,
            'street_number': partner.street_number,
            'street_number2': partner.street_number2,
        })
        return address_fields

    def _inverse_street_name(self):
        for company in self:
            company.partner_id.street_name = company.street_name

    def _inverse_street_number(self):
        for company in self:
            company.partner_id.street_number = company.street_number

    def _inverse_street_number2(self):
        for company in self:
            company.partner_id.street_number2 = company.street_number2

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import base_address_extended

```

## File: views\base_address_extended.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo>

    <record id="view_res_country_extended_form" model="ir.ui.view">
        <field name="name">view_res_country_extended_form</field>
        <field name="model">res.country</field>
        <field name="inherit_id" ref="base.view_country_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='div_address_format']" position="after">
                <field name="street_format" groups="base.group_no_one" placeholder="Street format..."/>
                <div colspan="2" class="text-muted">Change how the system computes the full street field based on the different street subfields</div>
            </xpath>
        </field>
    </record>

    <record id="view_partner_structured_form" model="ir.ui.view">
        <field name="name">view_partner_structured_form</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='street']" position="replace">
                <div>
                    <field name="street" class="oe_read_only"/>
                </div>
                <field name="street_name" placeholder="Street Name..." attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}" class="oe_edit_only"/>
                <div class="oe_edit_only o_row">
                    <label for="street_number"/>
                    <span> </span>
                    <field name="street_number" attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <label for="street_number2"/>
                    <field name="street_number2" attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                </div>
            </xpath>
        </field>
    </record>

    <record id="view_partner_address_structured_form" model="ir.ui.view">
        <field name="name">view_partner_address_structured_form</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_address_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='street']" position="replace">
                <field name="street" invisible="1"/>
                <field name="street_name" placeholder="Street Name..." attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                <div class="o_row">
                    <label for="street_number" class="oe_edit_only"/>
                    <span> </span>
                    <field name="street_number" attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <label for="street_number2" class="oe_edit_only"/>
                    <field name="street_number2" attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                </div>
            </xpath>
        </field>
    </record>

    <record id="view_res_company_extended_form" model="ir.ui.view">
        <field name="name">view_res_company_extended_form</field>
        <field name="model">res.company</field>
        <field name="inherit_id" ref="base.view_company_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='street']" position="replace">
                <field name="street" invisible="1"/>
                <field name="street_name" placeholder="Street Name..."/>
                <div class="o_row">
                    <label for="street_number" class="oe_edit_only"/>
                    <span> </span>
                    <field name="street_number"/>
                    <label for="street_number2" class="oe_edit_only"/>
                    <field name="street_number2"/>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

