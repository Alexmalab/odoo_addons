# Odoo Module: base_iban

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
    'name': 'IBAN Bank Accounts',
    'category': 'Hidden/Tools',
    'description': """
This module installs the base for IBAN (International Bank Account Number) bank accounts and checks for it's validity.
======================================================================================================================

The ability to extract the correctly represented local accounts from IBAN accounts
with a single statement.
    """,
    'depends': ['account', 'web'],
    'data': [
        'views/partner_view.xml',
        'views/setup_wizards_view.xml'
    ],
    'demo': ['data/res_partner_bank_demo.xml'],
    'assets': {
        'web.assets_backend': [
            'base_iban/static/src/components/**/*',
        ],
        'web.qunit_suite_tests': [
            'base_iban/static/src/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\res_partner_bank_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="bank_iban_asustek" model="res.partner.bank">
            <field name="acc_type">iban</field>
            <field name="acc_number">BE39103123456719</field>
            <field name="bank_id" ref="base.bank_bnp" />
            <field name="partner_id" ref="base.res_partner_1"/>
        </record>

        <record id="bank_iban_china_export" model="res.partner.bank">
            <field name="acc_type">iban</field>
            <field name="acc_number">SI56191000000123438</field>
            <field name="bank_id" ref="base.bank_bnp" />
            <field name="partner_id" ref="base.res_partner_3"/>
        </record>

        <record id="bank_iban_main_partner" model="res.partner.bank">
            <field name="acc_type">iban</field>
            <field name="acc_number">BE61310126985517</field>
            <field name="bank_id" ref="base.bank_ing" />
            <field name="partner_id" ref="base.res_partner_4"/>
        </record>

    </data>
</odoo>

```

## File: models\res_partner_bank.py

```python
# -*- coding: utf-8 -*-

import re

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError


def normalize_iban(iban):
    return re.sub(r'[\W_]', '', iban or '')

def pretty_iban(iban):
    """ return iban in groups of four characters separated by a single space """
    try:
        validate_iban(iban)
        iban = ' '.join([iban[i:i + 4] for i in range(0, len(iban), 4)])
    except ValidationError:
        pass
    return iban

def get_bban_from_iban(iban):
    """ Returns the basic bank account number corresponding to an IBAN.
        Note : the BBAN is not the same as the domestic bank account number !
        The relation between IBAN, BBAN and domestic can be found here : http://www.ecbs.org/iban.htm
    """
    return normalize_iban(iban)[4:]

def validate_iban(iban):
    iban = normalize_iban(iban)
    if not iban:
        raise ValidationError(_("There is no IBAN code."))

    country_code = iban[:2].lower()
    if country_code not in _map_iban_template:
        raise ValidationError(_("The IBAN is invalid, it should begin with the country code"))

    iban_template = _map_iban_template[country_code]
    if len(iban) != len(iban_template.replace(' ', '')) or not re.fullmatch("[a-zA-Z0-9]+", iban):
        raise ValidationError(_("The IBAN does not seem to be correct. You should have entered something like this %s\n"
            "Where B = National bank code, S = Branch code, C = Account No, k = Check digit", iban_template))

    check_chars = iban[4:] + iban[:4]
    digits = int(''.join(str(int(char, 36)) for char in check_chars))  # BASE 36: 0..9,A..Z -> 0..35
    if digits % 97 != 1:
        raise ValidationError(_("This IBAN does not pass the validation check, please verify it."))


class ResPartnerBank(models.Model):
    _inherit = "res.partner.bank"

    @api.model
    def _get_supported_account_types(self):
        rslt = super(ResPartnerBank, self)._get_supported_account_types()
        rslt.append(('iban', _('IBAN')))
        return rslt

    @api.model
    def retrieve_acc_type(self, acc_number):
        try:
            validate_iban(acc_number)
            return 'iban'
        except ValidationError:
            return super(ResPartnerBank, self).retrieve_acc_type(acc_number)

    def get_bban(self):
        if self.acc_type != 'iban':
            raise UserError(_("Cannot compute the BBAN because the account number is not an IBAN."))
        return get_bban_from_iban(self.acc_number)

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if vals.get('acc_number'):
                try:
                    validate_iban(vals['acc_number'])
                    vals['acc_number'] = pretty_iban(normalize_iban(vals['acc_number']))
                except ValidationError:
                    pass
        return super(ResPartnerBank, self).create(vals_list)

    def write(self, vals):
        if vals.get('acc_number'):
            try:
                validate_iban(vals['acc_number'])
                vals['acc_number'] = pretty_iban(normalize_iban(vals['acc_number']))
            except ValidationError:
                pass
        return super(ResPartnerBank, self).write(vals)

    @api.constrains('acc_number')
    def _check_iban(self):
        for bank in self:
            if bank.acc_type == 'iban':
                validate_iban(bank.acc_number)

    def check_iban(self, iban=''):
        try:
            validate_iban(iban)
            return True
        except ValidationError:
            return False

# Map ISO 3166-1 -> IBAN template, as described here :
# http://en.wikipedia.org/wiki/International_Bank_Account_Number#IBAN_formats_by_country
_map_iban_template = {
    'ad': 'ADkk BBBB SSSS CCCC CCCC CCCC',  # Andorra
    'ae': 'AEkk BBBC CCCC CCCC CCCC CCC',  # United Arab Emirates
    'al': 'ALkk BBBS SSSK CCCC CCCC CCCC CCCC',  # Albania
    'at': 'ATkk BBBB BCCC CCCC CCCC',  # Austria
    'az': 'AZkk BBBB CCCC CCCC CCCC CCCC CCCC',  # Azerbaijan
    'ba': 'BAkk BBBS SSCC CCCC CCKK',  # Bosnia and Herzegovina
    'be': 'BEkk BBBC CCCC CCXX',  # Belgium
    'bg': 'BGkk BBBB SSSS DDCC CCCC CC',  # Bulgaria
    'bh': 'BHkk BBBB CCCC CCCC CCCC CC',  # Bahrain
    'br': 'BRkk BBBB BBBB SSSS SCCC CCCC CCCT N',  # Brazil
    'by': 'BYkk BBBB AAAA CCCC CCCC CCCC CCCC',  # Belarus
    'ch': 'CHkk BBBB BCCC CCCC CCCC C',  # Switzerland
    'cr': 'CRkk BBBC CCCC CCCC CCCC CC',  # Costa Rica
    'cy': 'CYkk BBBS SSSS CCCC CCCC CCCC CCCC',  # Cyprus
    'cz': 'CZkk BBBB SSSS SSCC CCCC CCCC',  # Czech Republic
    'de': 'DEkk BBBB BBBB CCCC CCCC CC',  # Germany
    'dk': 'DKkk BBBB CCCC CCCC CC',  # Denmark
    'do': 'DOkk BBBB CCCC CCCC CCCC CCCC CCCC',  # Dominican Republic
    'ee': 'EEkk BBSS CCCC CCCC CCCK',  # Estonia
    'es': 'ESkk BBBB SSSS KKCC CCCC CCCC',  # Spain
    'fi': 'FIkk BBBB BBCC CCCC CK',  # Finland
    'fo': 'FOkk CCCC CCCC CCCC CC',  # Faroe Islands
    'fr': 'FRkk BBBB BGGG GGCC CCCC CCCC CKK',  # France
    'gb': 'GBkk BBBB SSSS SSCC CCCC CC',  # United Kingdom
    'ge': 'GEkk BBCC CCCC CCCC CCCC CC',  # Georgia
    'gi': 'GIkk BBBB CCCC CCCC CCCC CCC',  # Gibraltar
    'gl': 'GLkk BBBB CCCC CCCC CC',  # Greenland
    'gr': 'GRkk BBBS SSSC CCCC CCCC CCCC CCC',  # Greece
    'gt': 'GTkk BBBB MMTT CCCC CCCC CCCC CCCC',  # Guatemala
    'hr': 'HRkk BBBB BBBC CCCC CCCC C',  # Croatia
    'hu': 'HUkk BBBS SSSC CCCC CCCC CCCC CCCC',  # Hungary
    'ie': 'IEkk BBBB SSSS SSCC CCCC CC',  # Ireland
    'il': 'ILkk BBBS SSCC CCCC CCCC CCC',  # Israel
    'is': 'ISkk BBBB SSCC CCCC XXXX XXXX XX',  # Iceland
    'it': 'ITkk KBBB BBSS SSSC CCCC CCCC CCC',  # Italy
    'jo': 'JOkk BBBB NNNN CCCC CCCC CCCC CCCC CC',  # Jordan
    'kw': 'KWkk BBBB CCCC CCCC CCCC CCCC CCCC CC',  # Kuwait
    'kz': 'KZkk BBBC CCCC CCCC CCCC',  # Kazakhstan
    'lb': 'LBkk BBBB CCCC CCCC CCCC CCCC CCCC',  # Lebanon
    'li': 'LIkk BBBB BCCC CCCC CCCC C',  # Liechtenstein
    'lt': 'LTkk BBBB BCCC CCCC CCCC',  # Lithuania
    'lu': 'LUkk BBBC CCCC CCCC CCCC',  # Luxembourg
    'lv': 'LVkk BBBB CCCC CCCC CCCC C',  # Latvia
    'mc': 'MCkk BBBB BGGG GGCC CCCC CCCC CKK',  # Monaco
    'md': 'MDkk BBCC CCCC CCCC CCCC CCCC',  # Moldova
    'me': 'MEkk BBBC CCCC CCCC CCCC KK',  # Montenegro
    'mk': 'MKkk BBBC CCCC CCCC CKK',  # Macedonia
    'mr': 'MRkk BBBB BSSS SSCC CCCC CCCC CKK',  # Mauritania
    'mt': 'MTkk BBBB SSSS SCCC CCCC CCCC CCCC CCC',  # Malta
    'mu': 'MUkk BBBB BBSS CCCC CCCC CCCC CCCC CC',  # Mauritius
    'nl': 'NLkk BBBB CCCC CCCC CC',  # Netherlands
    'no': 'NOkk BBBB CCCC CCK',  # Norway
    'pk': 'PKkk BBBB CCCC CCCC CCCC CCCC',  # Pakistan
    'pl': 'PLkk BBBS SSSK CCCC CCCC CCCC CCCC',  # Poland
    'ps': 'PSkk BBBB XXXX XXXX XCCC CCCC CCCC C',  # Palestinian
    'pt': 'PTkk BBBB SSSS CCCC CCCC CCCK K',  # Portugal
    'qa': 'QAkk BBBB CCCC CCCC CCCC CCCC CCCC C',  # Qatar
    'ro': 'ROkk BBBB CCCC CCCC CCCC CCCC',  # Romania
    'rs': 'RSkk BBBC CCCC CCCC CCCC KK',  # Serbia
    'sa': 'SAkk BBCC CCCC CCCC CCCC CCCC',  # Saudi Arabia
    'se': 'SEkk BBBB CCCC CCCC CCCC CCCC',  # Sweden
    'si': 'SIkk BBSS SCCC CCCC CKK',  # Slovenia
    'sk': 'SKkk BBBB SSSS SSCC CCCC CCCC',  # Slovakia
    'sm': 'SMkk KBBB BBSS SSSC CCCC CCCC CCC',  # San Marino
    'tn': 'TNkk BBSS SCCC CCCC CCCC CCCC',  # Tunisia
    'tr': 'TRkk BBBB BRCC CCCC CCCC CCCC CC',  # Turkey
    'ua': 'UAkk BBBB BBCC CCCC CCCC CCCC CCCC C',  # Ukraine
    'vg': 'VGkk BBBB CCCC CCCC CCCC CCCC',  # Virgin Islands
    'xk': 'XKkk BBBB CCCC CCCC CCCC',  # Kosovo
}

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_partner_bank
```

## File: static\src\components\iban_widget\iban_widget.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { CharField, charField } from "@web/views/fields/char/char_field";
import { useDebounced } from "@web/core/utils/timing";
import { useService } from "@web/core/utils/hooks";
import { useState } from "@odoo/owl";

export const DELAY = 400;

export class IbanWidget extends CharField {
    setup() {
        super.setup();
        this.state = useState({ isValidIBAN: null });
        this.orm = useService("orm");
        this.validateIbanDebounced = useDebounced(async (ev) => {
            const iban = ev.target.value;
            if (!iban) {
                this.state.isValidIBAN = null;
            } else if (!/[A-Za-z]{2}.{3,}/.test(iban)) {
                this.state.isValidIBAN = false;
            } else {
                this.state.isValidIBAN = await this.orm.call("res.partner.bank", "check_iban", [[], iban]);
            }
        }, DELAY);
    }
}
IbanWidget.template = "base_iban.iban";

export const ibanWidget = {
    ...charField,
    component: IbanWidget,
};

registry.category("fields").add("iban", ibanWidget);

```

## File: static\src\components\iban_widget\iban_widget.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="base_iban.iban" t-inherit="web.CharField">
        <xpath expr="//input" position="attributes">
            <attribute name="class" add="o_iban_input_with_validator" separator=" "/>
            <attribute name="t-on-input">validateIbanDebounced</attribute>
            <attribute name="t-on-focus">validateIbanDebounced</attribute>
            <attribute name="t-on-blur">(ev) => { this.state.isValidIBAN = null; }</attribute>
        </xpath>
        <xpath expr="//input" position="after">
            <t t-if="this.state.isValidIBAN === true">
                <i class="fa fa-check o_iban text-success"/>
            </t>
            <t t-elif="this.state.isValidIBAN === false">
                <i class="fa fa-times o_iban text-danger o_iban_fail" title="Account isn't a valid IBAN"/>
            </t>
        </xpath>
    </t>
</templates>

```

## File: views\partner_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_partner_property_iban_form" model="ir.ui.view">
        <field name="name">res.partner.bank.form.inherit</field>
        <field name="model">res.partner.bank</field>
        <field name="inherit_id" ref="account.view_partner_bank_form_inherit_account"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='acc_number']" position="attributes">
                <attribute name="widget">iban</attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\setup_wizards_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="setup_bank_account_iban_wizard" model="ir.ui.view">
        <field name="name">account.online.sync.res.partner.bank.setup.form.inherit</field>
        <field name="model">account.setup.bank.manual.config</field>
        <field name="inherit_id" ref="account.setup_bank_account_wizard"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='acc_number']" position="attributes">
                <attribute name="widget">iban</attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

