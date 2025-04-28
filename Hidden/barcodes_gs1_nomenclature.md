# Odoo Module: barcodes_gs1_nomenclature

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Barcode - GS1 Nomenclature',
    'version': '1.0',
    'category': 'Hidden',
    'summary': 'Parse barcodes according to the GS1-128 specifications',
    'depends': ['barcodes', 'uom'],
    'data': [
        'data/barcodes_gs1_rules.xml',
        'views/barcodes_view.xml',
    ],
    'installable': True,
    'assets': {
        'web.assets_backend': [
            'barcodes_gs1_nomenclature/static/src/js/barcode_parser.js',
            'barcodes_gs1_nomenclature/static/src/js/barcode_service.js',
        ],
        'web.qunit_suite_tests': [
            'barcodes_gs1_nomenclature/static/src/js/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\barcodes_gs1_rules.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="default_gs1_nomenclature" model="barcode.nomenclature">
            <field name="name">Default GS1 Nomenclature</field>
            <field name="is_gs1_nomenclature">true</field>
        </record>

        <!-- Identifier Barcodes -->
        <record id="barcode_rule_gs1_00" model="barcode.rule">
            <field name="name">Serial Shipping Container Code</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">100</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(00)(\d{18})</field>
            <field name="type">package</field>
            <field name="gs1_content_type">identifier</field>
        </record>

        <record id="barcode_rule_gs1_01" model="barcode.rule">
            <field name="name">Global Trade Item Number (GTIN)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">101</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(01)(\d{14})</field>
            <field name="type">product</field>
            <field name="gs1_content_type">identifier</field>
        </record>

        <record id="barcode_rule_gs1_02" model="barcode.rule">
            <field name="name">GTIN of contained trade items</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">102</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(02)(\d{14})</field>
            <field name="type">product</field>
            <field name="gs1_content_type">identifier</field>
        </record>

        <record id="barcode_rule_gs1_410" model="barcode.rule">
            <field name="name">Ship to / Deliver to Global Location Number (GLN)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">110</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(410)(\d{13})</field>
            <field name="type">location_dest</field>
            <field name="gs1_content_type">identifier</field>
        </record>

        <record id="barcode_rule_gs1_413" model="barcode.rule">
            <field name="name">Ship for / Deliver for - Forward to Global Location Number (GLN)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">113</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(413)(\d{13})</field>
            <field name="type">location_dest</field>
            <field name="gs1_content_type">identifier</field>
        </record>

        <record id="barcode_rule_gs1_414" model="barcode.rule">
            <field name="name">Identification of a physical location - Global Location Number (GLN)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">114</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(414)(\d{13})</field>
            <field name="type">location</field>
            <field name="gs1_content_type">identifier</field>
        </record>

        <!-- Alphanumeric Barcodes -->
        <record id="barcode_rule_gs1_10" model="barcode.rule">
            <field name="name">Batch or lot number</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">125</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(10)([!"%-/0-9:-?A-Z_a-z]{0,20})</field>
            <field name="type">lot</field>
            <field name="gs1_content_type">alpha</field>
        </record>

        <record id="barcode_rule_gs1_21" model="barcode.rule">
            <field name="name">Serial number</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">126</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(21)([!"%-/0-9:-?A-Z_a-z]{0,20})</field>
            <field name="type">lot</field>
            <field name="gs1_content_type">alpha</field>
        </record>

        <!-- Date Barcodes -->
        <record id="barcode_rule_gs1_13" model="barcode.rule">
            <field name="name">Pack date (YYMMDD)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">137</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(13)(\d{6})</field>
            <field name="type">pack_date</field>
            <field name="gs1_content_type">date</field>
        </record>

        <record id="barcode_rule_gs1_15" model="barcode.rule">
            <field name="name">Best before date (YYMMDD)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">138</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(15)(\d{6})</field>
            <field name="type">use_date</field>
            <field name="gs1_content_type">date</field>
        </record>

        <record id="barcode_rule_gs1_17" model="barcode.rule">
            <field name="name">Expiration date (YYMMDD)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">139</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(17)(\d{6})</field>
            <field name="type">expiration_date</field>
            <field name="gs1_content_type">date</field>
        </record>

        <!-- Quantity/Measure Barcode -->
        <record id="barcode_rule_gs1_30" model="barcode.rule">
            <field name="name">Variable count of items (variable measure trade item)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">300</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(30)(\d{0,8})</field>
            <field name="associated_uom_id" ref="uom.product_uom_unit"/>
            <field name="type">quantity</field>
            <field name="gs1_content_type">measure</field>
            <field name="gs1_decimal_usage">False</field>
        </record>

        <record id="barcode_rule_gs1_37" model="barcode.rule">
            <field name="name">Count of trade items or trade item pieces contained in a logistic unit</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">305</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(37)(\d{0,8})</field>
            <field name="associated_uom_id" ref="uom.product_uom_unit"/>
            <field name="type">quantity</field>
            <field name="gs1_content_type">measure</field>
            <field name="gs1_decimal_usage">False</field>
        </record>

        <record id="barcode_rule_gs1_310y" model="barcode.rule">
            <field name="name">Net weight, kilograms (variable measure trade item)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">310</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(310[0-5])(\d{6})</field>
            <field name="associated_uom_id" ref="uom.product_uom_kgm"/>
            <field name="type">quantity</field>
            <field name="gs1_content_type">measure</field>
            <field name="gs1_decimal_usage">True</field>
        </record>

        <record id="barcode_rule_gs1_311y" model="barcode.rule">
            <field name="name">Length or first dimension, metres (variable measure trade item)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">311</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(311[0-5])(\d{6})</field>
            <field name="associated_uom_id" ref="uom.product_uom_meter"/>
            <field name="type">quantity</field>
            <field name="gs1_content_type">measure</field>
            <field name="gs1_decimal_usage">True</field>
        </record>

        <record id="barcode_rule_gs1_314y" model="barcode.rule">
            <field name="name">Area, square meters (variable measure trade item)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">314</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(314[0-5])(\d{6})</field>
            <field name="associated_uom_id" ref="uom.uom_square_meter"/>
            <field name="type">quantity</field>
            <field name="gs1_content_type">measure</field>
            <field name="gs1_decimal_usage">True</field>
        </record>

        <record id="barcode_rule_gs1_315y" model="barcode.rule">
            <field name="name">Net volume, litres (variable measure trade item)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">315</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(315[0-5])(\d{6})</field>
            <field name="associated_uom_id" ref="uom.product_uom_litre"/>
            <field name="type">quantity</field>
            <field name="gs1_content_type">measure</field>
            <field name="gs1_decimal_usage">True</field>
        </record>

        <record id="barcode_rule_gs1_316y" model="barcode.rule">
            <field name="name">Net volume, cubic metres (variable measure trade item)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">316</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(316[0-5])(\d{6})</field>
            <field name="associated_uom_id" ref="uom.product_uom_cubic_meter"/>
            <field name="type">quantity</field>
            <field name="gs1_content_type">measure</field>
            <field name="gs1_decimal_usage">True</field>
        </record>

        <record id="barcode_rule_gs1_320y" model="barcode.rule">
            <field name="name">Net weight, pounds (variable measure trade item)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">320</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(320[0-5])(\d{6})</field>
            <field name="associated_uom_id" ref="uom.product_uom_lb"/>
            <field name="type">quantity</field>
            <field name="gs1_content_type">measure</field>
            <field name="gs1_decimal_usage">True</field>
        </record>

        <record id="barcode_rule_gs1_321y" model="barcode.rule">
            <field name="name">Length or first dimension, inches (variable measure trade item)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">321</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(321[0-5])(\d{6})</field>
            <field name="associated_uom_id" ref="uom.product_uom_inch"/>
            <field name="type">quantity</field>
            <field name="gs1_content_type">measure</field>
            <field name="gs1_decimal_usage">True</field>
        </record>

        <record id="barcode_rule_gs1_322y" model="barcode.rule">
            <field name="name">Length or first dimension, feet (variable measure trade item)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">322</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(322[0-5])(\d{6})</field>
            <field name="associated_uom_id" ref="uom.product_uom_foot"/>
            <field name="type">quantity</field>
            <field name="gs1_content_type">measure</field>
            <field name="gs1_decimal_usage">True</field>
        </record>

        <record id="barcode_rule_gs1_323y" model="barcode.rule">
            <field name="name">Length or first dimension, yards (variable measure trade item)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">323</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(322[0-5])(\d{6})</field>
            <field name="associated_uom_id" ref="uom.product_uom_yard"/>
            <field name="type">quantity</field>
            <field name="gs1_content_type">measure</field>
            <field name="gs1_decimal_usage">True</field>
        </record>

        <record id="barcode_rule_gs1_351y" model="barcode.rule">
            <field name="name">Area, square feet (variable measure trade item)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">351</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(351[0-5])(\d{6})</field>
            <field name="associated_uom_id" ref="uom.uom_square_foot"/>
            <field name="type">quantity</field>
            <field name="gs1_content_type">measure</field>
            <field name="gs1_decimal_usage">True</field>
        </record>

        <record id="barcode_rule_gs1_357y" model="barcode.rule">
            <field name="name">Net weight (or volume), ounces (variable measure trade item)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">357</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(357[0-5])(\d{6})</field>
            <field name="associated_uom_id" ref="uom.product_uom_oz"/>
            <field name="type">quantity</field>
            <field name="gs1_content_type">measure</field>
            <field name="gs1_decimal_usage">True</field>
        </record>

        <record id="barcode_rule_gs1_360y" model="barcode.rule">
            <field name="name">Net volume, quarts (variable measure trade item)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">360</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(360[0-5])(\d{6})</field>
            <field name="associated_uom_id" ref="uom.product_uom_qt"/>
            <field name="type">quantity</field>
            <field name="gs1_content_type">measure</field>
            <field name="gs1_decimal_usage">True</field>
        </record>

        <record id="barcode_rule_gs1_361y" model="barcode.rule">
            <field name="name">Net volume, gallons U.S. (variable measure trade item)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">361</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(361[0-5])(\d{6})</field>
            <field name="associated_uom_id" ref="uom.product_uom_gal"/>
            <field name="type">quantity</field>
            <field name="gs1_content_type">measure</field>
            <field name="gs1_decimal_usage">True</field>
        </record>

        <record id="barcode_rule_gs1_364y" model="barcode.rule">
            <field name="name">Net volume, cubic inches (variable measure trade item)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">364</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(364[0-5])(\d{6})</field>
            <field name="associated_uom_id" ref="uom.product_uom_cubic_inch"/>
            <field name="type">quantity</field>
            <field name="gs1_content_type">measure</field>
            <field name="gs1_decimal_usage">True</field>
        </record>

        <record id="barcode_rule_gs1_365y" model="barcode.rule">
            <field name="name">Net volume, cubic feet (variable measure trade item)</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">365</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(365[0-5])(\d{6})</field>
            <field name="associated_uom_id" ref="uom.product_uom_cubic_foot"/>
            <field name="type">quantity</field>
            <field name="gs1_content_type">measure</field>
            <field name="gs1_decimal_usage">True</field>
        </record>

        <!-- Company internal information (91 to 99): Custom rules  -->
        <record id="barcode_rule_gs1_91" model="barcode.rule">
            <field name="name">Package type</field>
            <field name="barcode_nomenclature_id" ref="default_gs1_nomenclature"/>
            <field name="sequence">500</field>
            <field name="encoding">gs1-128</field>
            <field name="pattern">(91)([!"%-/0-9:-?A-Z_a-z]{0,90})</field>
            <field name="type">package_type</field>
            <field name="gs1_content_type">alpha</field>
        </record>
    </data>
</odoo>

```

## File: models\barcode_nomenclature.py

```python
import re
import datetime
import calendar

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError
from odoo.tools import get_barcode_check_digit

FNC1_CHAR = '\x1D'


class BarcodeNomenclature(models.Model):
    _inherit = 'barcode.nomenclature'

    is_gs1_nomenclature = fields.Boolean(
        string="Is GS1 Nomenclature",
        help="This Nomenclature use the GS1 specification, only GS1-128 encoding rules is accepted is this kind of nomenclature.")
    gs1_separator_fnc1 = fields.Char(
        string="FNC1 Separator", trim=False, default=r'(Alt029|#|\x1D)',
        help="Alternative regex delimiter for the FNC1. The separator must not match the begin/end of any related rules pattern.")

    @api.constrains('gs1_separator_fnc1')
    def _check_pattern(self):
        for nom in self:
            if nom.is_gs1_nomenclature and nom.gs1_separator_fnc1:
                try:
                    re.compile("(?:%s)?" % nom.gs1_separator_fnc1)
                except re.error as error:
                    raise ValidationError(_("The FNC1 Separator Alternative is not a valid Regex: ") + str(error))

    @api.model
    def gs1_date_to_date(self, gs1_date):
        """ Converts a GS1 date into a datetime.date.

        :param gs1_date: A year formated as yymmdd
        :type gs1_date: str
        :return: converted date
        :rtype: datetime.date
        """
        # See 7.12 Determination of century in dates:
        # https://www.gs1.org/sites/default/files/docs/barcodes/GS1_General_Specifications.pdf
        now = datetime.date.today()
        current_century = now.year // 100
        substract_year = int(gs1_date[0:2]) - (now.year % 100)
        century = (51 <= substract_year <= 99 and current_century - 1) or\
            (-99 <= substract_year <= -50 and current_century + 1) or\
            current_century
        year = century * 100 + int(gs1_date[0:2])

        if gs1_date[-2:] == '00':  # Day is not mandatory, when not set -> last day of the month
            date = datetime.datetime.strptime(str(year) + gs1_date[2:4], '%Y%m')
            date = date.replace(day=calendar.monthrange(year, int(gs1_date[2:4]))[1])
        else:
            date = datetime.datetime.strptime(str(year) + gs1_date[2:], '%Y%m%d')
        return date.date()

    def parse_gs1_rule_pattern(self, match, rule):
        result = {
            'rule': rule,
            'ai': match.group(1),
            'string_value': match.group(2),
        }
        if rule.gs1_content_type == 'measure':
            try:
                decimal_position = 0  # Decimal position begins at the end, 0 means no decimal.
                if rule.gs1_decimal_usage:
                    decimal_position = int(match.group(1)[-1])
                if decimal_position > 0:
                    result['value'] = float(match.group(2)[:-decimal_position] + "." + match.group(2)[-decimal_position:])
                else:
                    result['value'] = int(match.group(2))
            except Exception:
                raise ValidationError(_(
                    "There is something wrong with the barcode rule \"%s\" pattern.\n"
                    "If this rule uses decimal, check it can't get sometime else than a digit as last char for the Application Identifier.\n"
                    "Check also the possible matched values can only be digits, otherwise the value can't be casted as a measure.",
                    rule.name))
        elif rule.gs1_content_type == 'identifier':
            # Check digit and remove it of the value
            if match.group(2)[-1] != str(get_barcode_check_digit("0" * (18 - len(match.group(2))) + match.group(2))):
                return None
            result['value'] = match.group(2)
        elif rule.gs1_content_type == 'date':
            if len(match.group(2)) != 6:
                return None
            result['value'] = self.gs1_date_to_date(match.group(2))
        else:  # when gs1_content_type == 'alpha':
            result['value'] = match.group(2)
        return result

    def gs1_decompose_extanded(self, barcode):
        """Try to decompose the gs1 extanded barcode into several unit of information using gs1 rules.

        Return a ordered list of dict
        """
        self.ensure_one()
        separator_group = FNC1_CHAR + "?"
        if self.gs1_separator_fnc1:
            separator_group = "(?:%s)?" % self.gs1_separator_fnc1
        # zxing-library patch, removing GS1 identifiers
        for identifier in [']C1', ']e0', ']d2', ']Q3', ']J1', FNC1_CHAR]:
            if barcode.startswith(identifier):
                barcode = barcode.replace(identifier, '', 1)
                break
        results = []
        gs1_rules = self.rule_ids.filtered(lambda r: r.encoding == 'gs1-128')

        def find_next_rule(remaining_barcode):
            for rule in gs1_rules:
                match = re.search("^" + rule.pattern + separator_group, remaining_barcode)
                # If match and contains 2 groups at minimun, the first one need to be the AI and the second the value
                # We can't use regex nammed group because in JS, it is not the same regex syntax (and not compatible in all browser)
                if match and len(match.groups()) >= 2:
                    res = self.parse_gs1_rule_pattern(match, rule)
                    if res:
                        return res, remaining_barcode[match.end():]
            return None

        while len(barcode) > 0:
            res_bar = find_next_rule(barcode)
            # Cannot continue -> Fail to decompose gs1 and return
            if not res_bar or res_bar[1] == barcode:
                return None
            barcode = res_bar[1]
            results.append(res_bar[0])

        return results

    def parse_barcode(self, barcode):
        if self.is_gs1_nomenclature:
            return self.gs1_decompose_extanded(barcode)
        return super().parse_barcode(barcode)

    @api.model
    def _preprocess_gs1_search_args(self, args, barcode_types, field='barcode'):
        """Helper method to preprocess 'args' in _search method to add support to
        search with GS1 barcode result.
        Cut off the padding if using GS1 and searching on barcode. If the barcode
        is only digits to keep the original barcode part only.
        """
        nomenclature = self.env.company.nomenclature_id
        if nomenclature.is_gs1_nomenclature:
            for i, arg in enumerate(args):
                if not isinstance(arg, (list, tuple)) or len(arg) != 3:
                    continue
                field_name, operator, value = arg
                if field_name != field or operator not in ['ilike', 'not ilike', '=', '!='] or value is False:
                    continue

                parsed_data = []
                try:
                    parsed_data += nomenclature.parse_barcode(value) or []
                except (ValidationError, ValueError):
                    pass

                replacing_operator = 'ilike' if operator in ['ilike', '='] else 'not ilike'
                for data in parsed_data:
                    data_type = data['rule'].type
                    value = data['value']
                    if data_type in barcode_types:
                        if data_type == 'lot':
                            args[i] = (field_name, operator, value)
                            break
                        match = re.match('0*([0-9]+)$', str(value))
                        if match:
                            unpadded_barcode = match.groups()[0]
                            args[i] = (field_name, replacing_operator, unpadded_barcode)
                        break

                # The barcode isn't a valid GS1 barcode, checks if it can be unpadded.
                if not parsed_data:
                    match = re.match('0+([0-9]+)$', value)
                    if match:
                        args[i] = (field_name, replacing_operator, match.groups()[0])
        return args

```

## File: models\barcode_rule.py

```python
import re

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class BarcodeRule(models.Model):
    _inherit = 'barcode.rule'

    def _default_encoding(self):
        return 'gs1-128' if self.env.context.get('is_gs1') else 'any'

    encoding = fields.Selection(
        selection_add=[('gs1-128', 'GS1-128')], default=_default_encoding,
        ondelete={'gs1-128': 'set default'})
    type = fields.Selection(
        selection_add=[
            ('quantity', 'Quantity'),
            ('location', 'Location'),
            ('location_dest', 'Destination location'),
            ('lot', 'Lot number'),
            ('package', 'Package'),
            ('use_date', 'Best before Date'),
            ('expiration_date', 'Expiration Date'),
            ('package_type', 'Package Type'),
            ('pack_date', 'Pack Date'),
        ], ondelete={
            'quantity': 'set default',
            'location': 'set default',
            'location_dest': 'set default',
            'lot': 'set default',
            'package': 'set default',
            'use_date': 'set default',
            'expiration_date': 'set default',
            'package_type': 'set default',
            'pack_date': 'set default',
        })
    is_gs1_nomenclature = fields.Boolean(related="barcode_nomenclature_id.is_gs1_nomenclature")
    gs1_content_type = fields.Selection([
        ('date', 'Date'),
        ('measure', 'Measure'),
        ('identifier', 'Numeric Identifier'),
        ('alpha', 'Alpha-Numeric Name'),
    ], string="GS1 Content Type",
        help="The GS1 content type defines what kind of data the rule will process the barcode as:\
        * Date: the barcode will be converted into a Odoo datetime;\
        * Measure: the barcode's value is related to a specific UoM;\
        * Numeric Identifier: fixed length barcode following a specific encoding;\
        * Alpha-Numeric Name: variable length barcode.")
    gs1_decimal_usage = fields.Boolean('Decimal', help="If True, use the last digit of AI to determine where the first decimal is")
    associated_uom_id = fields.Many2one('uom.uom')

    @api.constrains('pattern')
    def _check_pattern(self):
        gs1_rules = self.filtered(lambda rule: rule.encoding == 'gs1-128')
        for rule in gs1_rules:
            try:
                re.compile(rule.pattern)
            except re.error as error:
                raise ValidationError(_("The rule pattern \"%s\" is not a valid Regex: ", rule.name) + str(error))
            groups = re.findall(r'\([^)]*\)', rule.pattern)
            if len(groups) != 2:
                raise ValidationError(_(
                    "The rule pattern \"%s\" is not valid, it needs two groups:"
                    "\n\t- A first one for the Application Identifier (usually 2 to 4 digits);"
                    "\n\t- A second one to catch the value.",
                    rule.name))

        super(BarcodeRule, (self - gs1_rules))._check_pattern()

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    def session_info(self):
        res = super().session_info()
        nomenclature = self.env.company.sudo().nomenclature_id
        if not nomenclature.is_gs1_nomenclature:
            return res
        res['gs1_group_separator_encodings'] = nomenclature.gs1_separator_fnc1
        return res

```

## File: models\__init__.py

```python
from . import barcode_nomenclature
from . import barcode_rule
from . import ir_http

```

## File: static\src\js\barcode_parser.js

```javascript
/** @odoo-module **/

import { patch } from "@web/core/utils/patch";
import { BarcodeParser } from "@barcodes/js/barcode_parser";
import { _t } from "@web/core/l10n/translation";
export class GS1BarcodeError extends Error {};

export const FNC1_CHAR = String.fromCharCode(29);

patch(BarcodeParser, {
    barcodeNomenclatureFields: [
        ...BarcodeParser.barcodeNomenclatureFields,
        "is_gs1_nomenclature",
        "gs1_separator_fnc1",
    ],
    barcodeRuleFields: [
        ...BarcodeParser.barcodeRuleFields,
        "gs1_content_type",
        "gs1_decimal_usage",
        "associated_uom_id",
    ],
});

patch(BarcodeParser.prototype, {
    setup(attributes) {
        super.setup(...arguments);
        // Use the nomenclature's separaor regex, else use an impossible one.
        const nomenclatureSeparator = this.nomenclature && this.nomenclature.gs1_separator_fnc1;
        this.gs1SeparatorRegex = new RegExp(nomenclatureSeparator || '.^', 'g');
    },

    /**
     * Convert YYMMDD GS1 date into a Date object
     *
     * @param {string} gs1Date YYMMDD string date, length must be 6
     * @returns {Date}
     */
    gs1_date_to_date(gs1Date) {
        // See 7.12 Determination of century in dates:
        // https://www.gs1.org/sites/default/files/docs/barcodes/GS1_General_Specifications.pdfDetermination of century
        const now = new Date();
        const substractYear = parseInt(gs1Date.slice(0, 2)) - (now.getFullYear() % 100);
        let century = Math.floor(now.getFullYear() / 100);
        if (51 <= substractYear && substractYear <= 99) {
            century--;
        } else if (-99 <= substractYear && substractYear <= -50) {
            century++;
        }
        const year = century * 100 + parseInt(gs1Date.slice(0, 2));
        const date = new Date(year, parseInt(gs1Date.slice(2, 4) - 1));

        if (gs1Date.slice(-2) === '00'){
            // Day is not mandatory, when not set -> last day of the month
            date.setDate(new Date(year, parseInt(gs1Date.slice(2, 4)), 0).getDate());
        } else {
            date.setDate(parseInt(gs1Date.slice(-2)));
        }
        return date;
    },

    /**
     * Perform interpretation of the barcode value depending of the rule.gs1_content_type
     *
     * @param {Array} match Result of a regex match with atmost 2 groups (ia and value)
     * @param {Object} rule Matched Barcode Rule
     * @returns {Object|null}
     */
    parse_gs1_rule_pattern(match, rule) {
        const result = {
            rule: Object.assign({}, rule),
            ai: match[1],
            string_value: match[2],
            code: match[2],
            base_code: match[2],
            type: rule.type
        };
        if (rule.gs1_content_type === 'measure'){
            let decimalPosition = 0; // Decimal position begin at the end, 0 means no decimal
            if (rule.gs1_decimal_usage){
                decimalPosition = parseInt(match[1][match[1].length - 1]);
            }
            if (decimalPosition > 0) {
                const integral = match[2].slice(0, match[2].length - decimalPosition);
                const decimal = match[2].slice(match[2].length - decimalPosition);
                result.value = parseFloat( integral + "." + decimal);
            } else {
                result.value = parseInt(match[2]);
            }
        } else if (rule.gs1_content_type === 'identifier'){
            if (parseInt(match[2][match[2].length - 1]) !== this.get_barcode_check_digit("0".repeat(18 - match[2].length) + match[2])){
                throw new Error(_t("Invalid barcode: the check digit is incorrect"));
                // return {error: _t("Invalid barcode: the check digit is incorrect")};
            }
            result.value = match[2];
        } else if (rule.gs1_content_type === 'date'){
            if (match[2].length !== 6){
                throw new Error(_t("Invalid barcode: can't be formated as date"));
                // return {error: _t("Invalid barcode: can't be formated as date")};
            }
            result.value = this.gs1_date_to_date(match[2]);
        } else {
            result.value = match[2];
        }
        return result;
    },

    /**
     * Try to decompose the gs1 extanded barcode into several unit of information using gs1 rules.
     *
     * @param {string} barcode
     * @returns {Array} Array of object
     */
    gs1_decompose_extanded(barcode) {
        const results = [];
        const rules = this.nomenclature.rules.filter(rule => rule.encoding === 'gs1-128');
        const separatorReg = `(?:${FNC1_CHAR}+)?`;
        barcode = this._convertGS1Separators(barcode);
        barcode = this.cleanBarcode(barcode);

        while (barcode.length > 0) {
            const barcodeLength = barcode.length;
            for (const rule of rules) {
                const match = barcode.match("^" + rule.pattern + separatorReg);
                if (match && match.length >= 3) {
                    const res = this.parse_gs1_rule_pattern(match, rule);
                    if (res) {
                        barcode = barcode.slice(match.index + match[0].length);
                        results.push(res);
                        if (barcode.length === 0) {
                            return results; // Barcode completly parsed, no need to keep looping.
                        }
                    } else {
                        throw new GS1BarcodeError(_t("This barcode can't be parsed by any barcode rules."));
                    }
                }
            }
            if (barcodeLength === barcode.length) {
                throw new GS1BarcodeError(_t("This barcode can't be partially or fully parsed."));
            }
        }

        return results;
    },

    /**
     * @override
     * @returns {Object|Array|null} If nomenclature is GS1, returns an array or null
     */
    parse_barcode(barcode) {
        if (this.nomenclature && this.nomenclature.is_gs1_nomenclature) {
            return this.gs1_decompose_extanded(barcode);
        }
        return super.parse_barcode(...arguments);
    },

    /**
     * Makes all needed operations to clean and prepare the barcode.
     * @param {string} barcode
     * @returns {string}
     */
    cleanBarcode(barcode) {
        if (barcode[0] === FNC1_CHAR) {
            // If first character is the separator, remove it to be able to parse the barcode.
            barcode = barcode.slice(1);
        }
        return barcode;
    },

    /**
     * The FNC1 is the default GS1 separator character, but through the field `gs1_separator_fnc1`,
     * the user has the possibility to define one or multiple characters to use as separator as
     * a regex. This method replaces all of the matches in the given barcode by the FNC1.
     *
     * @param {string} barcode
     * @returns {string}
     */
    _convertGS1Separators: function (barcode) {
        barcode = barcode.replace(this.gs1SeparatorRegex, FNC1_CHAR);
        return barcode;
    },
});

```

## File: static\src\js\barcode_service.js

```javascript
/** @odoo-module **/

import { session } from "@web/session";
import { patch } from "@web/core/utils/patch";
import { barcodeService } from '@barcodes/barcode_service';

import { FNC1_CHAR } from "@barcodes_gs1_nomenclature/js/barcode_parser";


patch(barcodeService, {
    // Use the regex given by the session, else use an impossible one
    gs1SeparatorRegex: new RegExp(session.gs1_group_separator_encodings || '.^', 'g'),

    cleanBarcode(barcode) {
        barcode = barcode.replace(barcodeService.gs1SeparatorRegex, FNC1_CHAR);
        return super.cleanBarcode(barcode);
    },
});

```

## File: views\barcodes_view.xml

```xml
<?xml version="1.0"?>
<odoo>
        <record id="view_barcode_gs1_nomenclature_form" model="ir.ui.view">
            <field name="name">Barcode Nomenclatures</field>
            <field name="model">barcode.nomenclature</field>
            <field name="inherit_id" ref="barcodes.view_barcode_nomenclature_form"/>
            <field name="arch" type="xml">
                <!-- Nomenclature form -->
                <xpath expr="//field[@name='upc_ean_conv']" position="attributes">
                    <attribute name="invisible">is_gs1_nomenclature</attribute>
                </xpath>
                <xpath expr="//group[@name='general_attributes']" position="after">
                    <group name="gs1_attributes" col="4">
                        <field name="is_gs1_nomenclature"/>
                        <field name="gs1_separator_fnc1" invisible="not is_gs1_nomenclature"/>
                    </group>
                </xpath>
                <!-- Rules table -->
                <xpath expr="//field[@name='rule_ids']" position="attributes">
                    <attribute name="context">{'is_gs1': is_gs1_nomenclature}</attribute>
                </xpath>
                <xpath expr="//field[@name='encoding']" position="attributes">
                    <attribute name="column_invisible">parent.is_gs1_nomenclature</attribute>
                </xpath>
                <xpath expr="//field[@name='pattern']" position="after">
                    <field name="gs1_content_type" column_invisible="not parent.is_gs1_nomenclature"/>
                    <field name="gs1_decimal_usage" column_invisible="not parent.is_gs1_nomenclature" invisible="gs1_content_type != 'measure'"/>
                    <field name="associated_uom_id" column_invisible="not parent.is_gs1_nomenclature" invisible="gs1_content_type != 'measure'"/>
                </xpath>
            </field>
        </record>

        <record id="view_barcode_gs1_nomenclature_tree" model="ir.ui.view">
            <field name="name">Barcode Nomenclatures</field>
            <field name="model">barcode.nomenclature</field>
            <field name="inherit_id" ref="barcodes.view_barcode_nomenclature_tree"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='name']" position="after">
                    <field name="is_gs1_nomenclature"/>
                </xpath>
            </field>
        </record>

        <record id="view_barcode_gs1_rule_form" model="ir.ui.view">
            <field name="name">Barcode Rule</field>
            <field name="model">barcode.rule</field>
            <field name="inherit_id" ref="barcodes.view_barcode_rule_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='encoding']" position="attributes">
                    <attribute name="invisible">parent.is_gs1_nomenclature or type == 'alias'</attribute>
                </xpath>
                <xpath expr="//field[@name='alias']" position="after">
                    <field name="gs1_content_type" invisible="not parent.is_gs1_nomenclature"/>
                    <field name="gs1_decimal_usage" invisible="not parent.is_gs1_nomenclature or gs1_content_type != 'measure'"/>
                    <field name="associated_uom_id" invisible="not parent.is_gs1_nomenclature or gs1_content_type != 'measure'"/>
                    <field name="is_gs1_nomenclature" invisible="1"/>
                </xpath>
            </field>
        </record>
</odoo>

```

