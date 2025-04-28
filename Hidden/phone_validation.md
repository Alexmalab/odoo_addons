# Odoo Module: phone_validation

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import lib
from . import tools
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Phone Numbers Validation',
    'version': '2.0',
    'summary': 'Validate and format phone numbers',
    'sequence': '9999',
    'category': 'Hidden',
    'description': """
Phone Numbers Validation
========================

This module adds the feature of validation and formatting phone numbers
according to a destination country.

It also adds phone blacklist management through a specific model storing
blacklisted phone numbers.

It adds two mixins :

  * phone.validation.mixin: parsing / formatting helpers on records, to be
    used for example in number fields onchange;
  * mail.thread.phone: handle sanitation and blacklist of records numbers;
""",
    'data': [
        'security/ir.model.access.csv',
        'views/phone_blacklist_views.xml',
    ],
    'depends': [
        'base',
        'mail',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: lib\phonemetadata.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

try:
    # import for usage in phonenumbers_patch/region_*.py files
    from phonenumbers.phonemetadata import NumberFormat, PhoneNumberDesc, PhoneMetadata # pylint: disable=unused-import
except ImportError:
    pass

```

## File: lib\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import phonenumbers_patch

```

## File: lib\phonenumbers_patch\region_CI.py

```python
"""Auto-generated file, do not edit by hand. CI metadata"""
from ..phonemetadata import NumberFormat, PhoneNumberDesc, PhoneMetadata

PHONE_METADATA_CI = PhoneMetadata(id='CI', country_code=225, international_prefix='00',
    general_desc=PhoneNumberDesc(national_number_pattern='[02]\\d{9}', possible_length=(10,)),
    fixed_line=PhoneNumberDesc(national_number_pattern='2(?:[15]\\d{3}|7(?:2(?:0[23]|1[2357]|[23][45]|4[3-5])|3(?:06|1[69]|[2-6]7)))\\d{5}', example_number='2123456789', possible_length=(10,)),
    mobile=PhoneNumberDesc(national_number_pattern='0704[0-7]\\d{5}|0(?:[15]\\d\\d|7(?:0[0-37-9]|[4-9][7-9]))\\d{6}', example_number='0123456789', possible_length=(10,)),
    number_format=[NumberFormat(pattern='(\\d{2})(\\d{2})(\\d)(\\d{5})', format='\\1 \\2 \\3 \\4', leading_digits_pattern=['2']),
        NumberFormat(pattern='(\\d{2})(\\d{2})(\\d{2})(\\d{4})', format='\\1 \\2 \\3 \\4', leading_digits_pattern=['0'])])

```

## File: lib\phonenumbers_patch\__init__.py

```python
# -*- coding: utf-8 -*-
# Copyright (C) 2009 The Libphonenumber Authors

# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at

# http://www.apache.org/licenses/LICENSE-2.0

# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# https://github.com/google/libphonenumber

from odoo.tools.parse_version import parse_version

try:
    import phonenumbers
    # MONKEY PATCHING phonemetadata of Ivory Coast if phonenumbers is too old
    if parse_version('7.6.1') <= parse_version(phonenumbers.__version__) < parse_version('8.12.32'):
        def _local_load_region(code):
            __import__("region_%s" % code, globals(), locals(),
                fromlist=["PHONE_METADATA_%s" % code], level=1)
        # loading updated region_CI.py from current directory
        # https://github.com/daviddrysdale/python-phonenumbers/blob/v8.12.32/python/phonenumbers/data/region_CI.py
        phonenumbers.phonemetadata.PhoneMetadata.register_region_loader('CI', _local_load_region)
except ImportError:
    pass

```

## File: models\mail_thread_phone.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.addons.phone_validation.tools import phone_validation
from odoo.exceptions import UserError


class PhoneMixin(models.AbstractModel):
    """ Purpose of this mixin is to offer two services

      * compute a sanitized phone number based on ´´_sms_get_number_fields´´.
        It takes first sanitized value, trying each field returned by the
        method (see ``MailThread._sms_get_number_fields()´´ for more details
        about the usage of this method);
      * compute blacklist state of records. It is based on phone.blacklist
        model and give an easy-to-use field and API to manipulate blacklisted
        records;

    Main API methods

      * ``_phone_set_blacklisted``: set recordset as blacklisted;
      * ``_phone_reset_blacklisted``: reactivate recordset (even if not blacklisted
        this method can be called safely);
    """
    _name = 'mail.thread.phone'
    _description = 'Phone Blacklist Mixin'
    _inherit = ['mail.thread']

    phone_sanitized = fields.Char(
        string='Sanitized Number', compute="_compute_phone_sanitized", compute_sudo=True, store=True,
        help="Field used to store sanitized phone number. Helps speeding up searches and comparisons.")
    phone_blacklisted = fields.Boolean(
        string='Phone Blacklisted', compute="_compute_phone_blacklisted", compute_sudo=True, store=False,
        search="_search_phone_blacklisted", groups="base.group_user",
        help="If the email address is on the blacklist, the contact won't receive mass mailing anymore, from any list")

    @api.depends(lambda self: self._phone_get_number_fields())
    def _compute_phone_sanitized(self):
        self._assert_phone_field()
        number_fields = self._phone_get_number_fields()
        for record in self:
            for fname in number_fields:
                sanitized = record.phone_get_sanitized_number(number_fname=fname)
                if sanitized:
                    break
            record.phone_sanitized = sanitized

    @api.depends('phone_sanitized')
    def _compute_phone_blacklisted(self):
        # TODO : Should remove the sudo as compute_sudo defined on methods.
        # But if user doesn't have access to mail.blacklist, doen't work without sudo().
        blacklist = set(self.env['phone.blacklist'].sudo().search([
            ('number', 'in', self.mapped('phone_sanitized'))]).mapped('number'))
        for record in self:
            record.phone_blacklisted = record.phone_sanitized in blacklist

    @api.model
    def _search_phone_blacklisted(self, operator, value):
        # Assumes operator is '=' or '!=' and value is True or False
        self._assert_phone_field()
        if operator != '=':
            if operator == '!=' and isinstance(value, bool):
                value = not value
            else:
                raise NotImplementedError()

        if value:
            query = """
                SELECT m.id
                    FROM phone_blacklist bl
                    JOIN %s m
                    ON m.phone_sanitized = bl.number AND bl.active
            """
        else:
            query = """
                SELECT m.id
                    FROM %s m
                    LEFT JOIN phone_blacklist bl
                    ON m.phone_sanitized = bl.number AND bl.active
                    WHERE bl.id IS NULL
            """
        self._cr.execute(query % self._table)
        res = self._cr.fetchall()
        if not res:
            return [(0, '=', 1)]
        return [('id', 'in', [r[0] for r in res])]

    def _assert_phone_field(self):
        if not hasattr(self, "_phone_get_number_fields"):
            raise UserError(_('Invalid primary phone field on model %s') % self._name)
        if not any(fname in self and self._fields[fname].type == 'char' for fname in self._phone_get_number_fields()):
            raise UserError(_('Invalid primary phone field on model %s') % self._name)

    def _phone_get_number_fields(self):
        """ This method returns the fields to use to find the number to use to
        send an SMS on a record. """
        return []

    def _phone_get_country_field(self):
        if 'country_id' in self:
            return 'country_id'
        return False

    def phone_get_sanitized_numbers(self, number_fname='mobile', force_format='E164'):
        res = dict.fromkeys(self.ids, False)
        country_fname = self._phone_get_country_field()
        for record in self:
            number = record[number_fname]
            res[record.id] = phone_validation.phone_sanitize_numbers_w_record([number], record, record_country_fname=country_fname, force_format=force_format)[number]['sanitized']
        return res

    def phone_get_sanitized_number(self, number_fname='mobile', force_format='E164'):
        self.ensure_one()
        country_fname = self._phone_get_country_field()
        number = self[number_fname]
        return phone_validation.phone_sanitize_numbers_w_record([number], self, record_country_fname=country_fname, force_format=force_format)[number]['sanitized']

    def _phone_set_blacklisted(self):
        return self.env['phone.blacklist'].sudo()._add([r.phone_sanitized for r in self])

    def _phone_reset_blacklisted(self):
        return self.env['phone.blacklist'].sudo()._remove([r.phone_sanitized for r in self])

```

## File: models\phone_blacklist.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, fields, models, _
from odoo.addons.phone_validation.tools import phone_validation
from odoo.exceptions import UserError

_logger = logging.getLogger(__name__)


class PhoneBlackList(models.Model):
    """ Blacklist of phone numbers. Used to avoid sending unwanted messages to people. """
    _name = 'phone.blacklist'
    _inherit = ['mail.thread']
    _description = 'Phone Blacklist'
    _rec_name = 'number'

    number = fields.Char(string='Phone Number', required=True, index=True, tracking=True, help='Number should be E164 formatted')
    active = fields.Boolean(default=True, tracking=True)

    _sql_constraints = [
        ('unique_number', 'unique (number)', 'Number already exists')
    ]

    @api.model_create_multi
    def create(self, values):
        # First of all, extract values to ensure emails are really unique (and don't modify values in place)
        to_create = []
        done = set()
        for value in values:
            number = value['number']
            sanitized = phone_validation.phone_sanitize_numbers_w_record([number], self.env.user)[number]['sanitized']
            if not sanitized:
                raise UserError(_('Invalid number %s') % number)
            if sanitized in done:
                continue
            done.add(sanitized)
            to_create.append(dict(value, number=sanitized))

        """ To avoid crash during import due to unique email, return the existing records if any """
        sql = '''SELECT number, id FROM phone_blacklist WHERE number = ANY(%s)'''
        numbers = [v['number'] for v in to_create]
        self._cr.execute(sql, (numbers,))
        bl_entries = dict(self._cr.fetchall())
        to_create = [v for v in to_create if v['number'] not in bl_entries]

        results = super(PhoneBlackList, self).create(to_create)
        return self.env['phone.blacklist'].browse(bl_entries.values()) | results

    def write(self, values):
        if 'number' in values:
            number = values['number']
            sanitized = phone_validation.phone_sanitize_numbers_w_record([number], self.env.user)[number]['sanitized']
            if not sanitized:
                raise UserError(_('Invalid number %s') % number)
            values['number'] = sanitized
        return super(PhoneBlackList, self).write(values)

    def _search(self, args, offset=0, limit=None, order=None, count=False, access_rights_uid=None):
        """ Override _search in order to grep search on sanitized number field """
        if args:
            new_args = []
            for arg in args:
                if isinstance(arg, (list, tuple)) and arg[0] == 'number' and isinstance(arg[2], str):
                    number = arg[2]
                    sanitized = phone_validation.phone_sanitize_numbers_w_record([number], self.env.user)[number]['sanitized']
                    if sanitized:
                        new_args.append([arg[0], arg[1], sanitized])
                    else:
                        new_args.append(arg)
                else:
                    new_args.append(arg)
        else:
            new_args = args
        return super(PhoneBlackList, self)._search(new_args, offset=offset, limit=limit, order=order, count=count, access_rights_uid=access_rights_uid)

    def add(self, number):
        sanitized = phone_validation.phone_sanitize_numbers_w_record([number], self.env.user)[number]['sanitized']
        return self._add([sanitized])

    def _add(self, numbers):
        """ Add or re activate a phone blacklist entry.

        :param numbers: list of sanitized numbers """
        records = self.env["phone.blacklist"].with_context(active_test=False).search([('number', 'in', numbers)])
        todo = [n for n in numbers if n not in records.mapped('number')]
        if records:
            records.write({'active': True})
        if todo:
            records += self.create([{'number': n} for n in todo])
        return records

    def remove(self, number):
        sanitized = phone_validation.phone_sanitize_numbers_w_record([number], self.env.user)[number]['sanitized']
        return self._remove([sanitized])

    def _remove(self, numbers):
        """ Add de-activated or de-activate a phone blacklist entry.

        :param numbers: list of sanitized numbers """
        records = self.env["phone.blacklist"].with_context(active_test=False).search([('number', 'in', numbers)])
        todo = [n for n in numbers if n not in records.mapped('number')]
        if records:
            records.write({'active': False})
        if todo:
            records += self.create([{'number': n, 'active': False} for n in todo])
        return records

```

## File: models\phone_validation_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.addons.phone_validation.tools import phone_validation


class PhoneValidationMixin(models.AbstractModel):
    _name = 'phone.validation.mixin'
    _description = 'Phone Validation Mixin'

    def _phone_get_country(self):
        if 'country_id' in self and self.country_id:
            return self.country_id
        return self.env.company.country_id

    def phone_format(self, number, country=None, company=None):
        country = country or self._phone_get_country()
        if not country:
            return number
        return phone_validation.phone_format(
            number,
            country.code if country else None,
            country.phone_code if country else None,
            force_format='INTERNATIONAL',
            raise_exception=False
        )

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class Partner(models.Model):
    _name = 'res.partner'
    _inherit = ['res.partner', 'phone.validation.mixin']

    @api.onchange('phone', 'country_id', 'company_id')
    def _onchange_phone_validation(self):
        if self.phone:
            self.phone = self.phone_format(self.phone)

    @api.onchange('mobile', 'country_id', 'company_id')
    def _onchange_mobile_validation(self):
        if self.mobile:
            self.mobile = self.phone_format(self.mobile)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import phone_blacklist
from . import phone_validation_mixin
from . import mail_thread_phone
from . import res_partner

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_phone_blacklist_all,access.phone.blacklist.all,model_phone_blacklist,,0,0,0,0
access_phone_blacklist_system,access.phone.blacklist.system,model_phone_blacklist,base.group_system,1,1,1,1

```

## File: tools\phone_validation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _
from odoo.exceptions import UserError

import logging

_logger = logging.getLogger(__name__)
_phonenumbers_lib_warning = False


try:
    import phonenumbers

    def phone_parse(number, country_code):
        try:
            phone_nbr = phonenumbers.parse(number, region=country_code, keep_raw_input=True)
        except phonenumbers.phonenumberutil.NumberParseException as e:
            raise UserError(_('Unable to parse %s: %s') % (number, str(e)))

        if not phonenumbers.is_possible_number(phone_nbr):
            raise UserError(_('Impossible number %s: probably invalid number of digits') % number)
        if not phonenumbers.is_valid_number(phone_nbr):
            raise UserError(_('Invalid number %s: probably incorrect prefix') % number)

        return phone_nbr

    def phone_format(number, country_code, country_phone_code, force_format='INTERNATIONAL', raise_exception=True):
        """ Format the given phone number according to the localisation and international options.
        :param number: number to convert
        :param country_code: the ISO country code in two chars
        :type country_code: str
        :param country_phone_code: country dial in codes, defined by the ITU-T (Ex: 32 for Belgium)
        :type country_phone_code: int
        :param force_format: stringified version of format globals (see
          https://github.com/daviddrysdale/python-phonenumbers/blob/dev/python/phonenumbers/phonenumberutil.py)
            'E164' = 0
            'INTERNATIONAL' = 1
            'NATIONAL' = 2
            'RFC3966' = 3
        :type force_format: str
        :rtype: str
        """
        try:
            phone_nbr = phone_parse(number, country_code)
        except (phonenumbers.phonenumberutil.NumberParseException, UserError) as e:
            if raise_exception:
                raise
            else:
                return number
        if force_format == 'E164':
            phone_fmt = phonenumbers.PhoneNumberFormat.E164
        elif force_format == 'RFC3966':
            phone_fmt = phonenumbers.PhoneNumberFormat.RFC3966
        elif force_format == 'INTERNATIONAL' or phone_nbr.country_code != country_phone_code:
            phone_fmt = phonenumbers.PhoneNumberFormat.INTERNATIONAL
        else:
            phone_fmt = phonenumbers.PhoneNumberFormat.NATIONAL
        return phonenumbers.format_number(phone_nbr, phone_fmt)

except ImportError:

    def phone_parse(number, country_code):
        return False

    def phone_format(number, country_code, country_phone_code, force_format='INTERNATIONAL', raise_exception=True):
        global _phonenumbers_lib_warning
        if not _phonenumbers_lib_warning:
            _logger.info(
                "The `phonenumbers` Python module is not installed, contact numbers will not be "
                "verified. Please install the `phonenumbers` Python module."
            )
            _phonenumbers_lib_warning = True
        return number


def phone_sanitize_numbers(numbers, country_code, country_phone_code, force_format='E164'):
    """ Given a list of numbers, return parsezd and sanitized information

    :return dict: {number: {
        'sanitized': sanitized and formated number or False (if cannot format)
        'code': 'empty' (number was a void string), 'invalid' (error) or False (sanitize ok)
        'msg': error message when 'invalid'
    }}
    """
    if not isinstance(numbers, (list)):
        raise NotImplementedError()
    result = dict.fromkeys(numbers, False)
    for number in numbers:
        if not number:
            result[number] = {'sanitized': False, 'code': 'empty', 'msg': False}
            continue
        try:
            stripped = number.strip()
            sanitized = phone_format(
                stripped, country_code, country_phone_code,
                force_format=force_format, raise_exception=True)
        except Exception as e:
            result[number] = {'sanitized': False, 'code': 'invalid', 'msg': str(e)}
        else:
            result[number] = {'sanitized': sanitized, 'code': False, 'msg': False}
    return result


def phone_sanitize_numbers_w_record(numbers, record, country=False, record_country_fname='country_id', force_format='E164'):
    if not isinstance(numbers, (list)):
        raise NotImplementedError()
    if not country:
        if record and record_country_fname and hasattr(record, record_country_fname) and record[record_country_fname]:
            country = record[record_country_fname]
        elif record:
            country = record.env.company.country_id
    country_code = country.code if country else None
    country_phone_code = country.phone_code if country else None
    return phone_sanitize_numbers(numbers, country_code, country_phone_code, force_format=force_format)

```

## File: tools\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import phone_validation

```

## File: views\phone_blacklist_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="phone_blacklist_view_tree" model="ir.ui.view">
        <field name="name">phone.blacklist.view.tree</field>
        <field name="model">phone.blacklist</field>
        <field name="arch" type="xml">
            <tree string="Phone Blacklist">
                <field name="create_date" string="Blacklist Date"/>
                <field name="number"/>
            </tree>
        </field>
    </record>

    <record id="phone_blacklist_view_form" model="ir.ui.view">
        <field name="name">phone.blacklist.view.form</field>
        <field name="model">phone.blacklist</field>
        <field name="arch" type="xml">
            <form string="Phone Blacklist" duplicate="false">
                <sheet>
                    <group>
                        <group>
                            <field name="number"/>
                            <field name="active"/>
                        </group>
                    </group>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids" widget="mail_followers"/>
                    <field name="message_ids" widget="mail_thread"/>
                </div>
            </form>
        </field>
    </record>

    <record id="phone_blacklist_view_search" model="ir.ui.view">
        <field name="name">phone.blacklist.view.search</field>
        <field name="model">phone.blacklist</field>
        <field name="arch" type="xml">
            <search>
                <field name="number"/>
                <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
            </search>
        </field>
    </record>

    <record id="phone_blacklist_action" model="ir.actions.act_window">
        <field name="name">Phone Blacklist</field>
        <field name="res_model">phone.blacklist</field>
        <field name="view_id" ref="phone_blacklist_view_tree"/>
        <field name="search_view_id" ref="phone_blacklist_view_search"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Add a phone number in the blacklist
            </p><p>
                Blacklisted phone numbers means that the recipient won't receive SMS Marketing anymore.
            </p>
        </field>
    </record>

    <!-- Technical Menu -->
    <menuitem id="phone_menu_main"
        name="Phone / SMS"
        parent="base.menu_custom"
        sequence="2"/>

    <menuitem id="phone_blacklist_menu"
        name="Phone Blacklist"
        parent="phone_validation.phone_menu_main"
        sequence="3"
        action="phone_blacklist_action"/>

</odoo>

```

