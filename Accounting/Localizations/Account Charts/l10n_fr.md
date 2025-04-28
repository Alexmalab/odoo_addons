# Odoo Module: l10n_fr

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'France - Localizations',
    'icon': '/account/static/description/l10n.png',
    'countries': ['fr'],
    'version': '2.1',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
""",
    'depends': [
        'base',
    ],
    'data': [
        'data/res_country_data.xml',
        'views/res_company_views.xml',
        'views/res_partner_views.xml',
    ],
    'demo': [
        'data/l10n_fr_demo.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\l10n_fr_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="base.partner_demo_company_fr" model="res.partner" forcecreate="1">
        <field name="name">FR Company</field>
        <field name="vat">FR91746948785</field>
        <field name="street">Rue Abbé Huet</field>
        <field name="city">Rennes</field>
        <field name="country_id" ref="base.fr"/>
        <field name="siret">96851575905808</field>

        <field name="zip">35043</field>
        <field name="phone">+33 6 12 34 56 78</field>
        <field name="email">info@company.frexample.com</field>
        <field name="website">www.frexample.com</field>
        <field name="is_company" eval="True"/>
    </record>

    <record id="base.demo_company_fr" model="res.company" forcecreate="1">
        <field name="name">FR Company</field>
        <field name="partner_id" ref="base.partner_demo_company_fr"/>
    </record>

    <record id="base.demo_bank_fr" model="res.partner.bank" forcecreate="1">
        <field name="acc_number">FR5730003000507963949549B56</field>
        <field name="partner_id" ref="base.partner_demo_company_fr"/>
        <field name="company_id" ref="base.demo_company_fr"/>
    </record>

    <function model="res.company" name="_onchange_country_id">
        <value eval="[ref('base.demo_company_fr')]"/>
    </function>

    <function model="res.users" name="write">
        <value eval="[ref('base.user_root'), ref('base.user_admin'), ref('base.user_demo')]"/>
        <value eval="{'company_ids': [(4, ref('base.demo_company_fr'))]}"/>
    </function>
</odoo>

```

## File: data\res_country_data.xml

```xml
<odoo>
    <record id="dom-tom" model="res.country.group">
             <field name="name">DOM-TOM</field>
             <field name="country_ids" eval="[(6,0,[
                                                ref('base.yt'),
                                                ref('base.gp'),
                                                ref('base.mq'),
                                                ref('base.gf'),
                                                ref('base.re'),
                                                ref('base.pf'),
                                                ref('base.pm'),
                                                ref('base.mf'),
                                                ref('base.bl'),
                                                ref('base.nc')])]"/>
       </record>

    <record id="fr_and_mc" model="res.country.group">
        <field name="name">France and Monaco</field>
        <field name="country_ids" eval="[Command.set([ref('base.fr'), ref('base.mc')])]"/>
    </record>
</odoo>

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_fr_closing_sequence_id = fields.Many2one('ir.sequence', 'Sequence to use to build sale closings', readonly=True)
    siret = fields.Char(related='partner_id.siret', string='SIRET', size=14, readonly=False)
    ape = fields.Char(string='APE')
    is_france_country = fields.Boolean(
        compute="_compute_is_france_country",
        string="Is Part of DOM-TOM",
    )

    @api.depends('country_code')
    def _compute_is_france_country(self):
        for company in self:
            company.is_france_country = company.country_code in self._get_france_country_codes()

    @api.model
    def _get_france_country_codes(self):
        """Returns every country code that can be used to represent France
        """
        return ['FR', 'MF', 'MQ', 'NC', 'PF', 'RE', 'GF', 'GP', 'TF', 'BL', 'PM', 'YT', 'WF']  # These codes correspond to France and DOM-TOM.

    def _is_accounting_unalterable(self):
        if not self.vat and not self.country_id:
            return False
        return self.country_id and self.country_id.code in self._get_france_country_codes()

    @api.model_create_multi
    def create(self, vals_list):
        companies = super().create(vals_list)
        for company in companies:
            #when creating a new french company, create the securisation sequence as well
            if company._is_accounting_unalterable():
                sequence_fields = ['l10n_fr_closing_sequence_id']
                company._create_secure_sequence(sequence_fields)
        return companies

    def write(self, vals):
        res = super(ResCompany, self).write(vals)
        #if country changed to fr, create the securisation sequence
        for company in self:
            if company._is_accounting_unalterable():
                sequence_fields = ['l10n_fr_closing_sequence_id']
                company._create_secure_sequence(sequence_fields)
        return res

    def _create_secure_sequence(self, sequence_fields):
        """This function creates a no_gap sequence on each company in self that will ensure
        a unique number is given to all posted account.move in such a way that we can always
        find the previous move of a journal entry on a specific journal.
        """
        for company in self:
            vals_write = {}
            for seq_field in sequence_fields:
                if not company[seq_field]:
                    vals = {
                        'name': _('Securisation of %(field)s - %(company)s', field=seq_field, company=company.name),
                        'code': 'FRSECURE%s-%s' % (company.id, seq_field),
                        'implementation': 'no_gap',
                        'prefix': '',
                        'suffix': '',
                        'padding': 0,
                        'company_id': company.id}
                    seq = self.env['ir.sequence'].create(vals)
                    vals_write[seq_field] = seq.id
            if vals_write:
                company.write(vals_write)

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    siret = fields.Char(string='SIRET', size=14)

    def _deduce_country_code(self):
        if self.siret:
            return 'FR'
        return super()._deduce_country_code()

    def _peppol_eas_endpoint_depends(self):
        # extends account_edi_ubl_cii
        return super()._peppol_eas_endpoint_depends() + ['siret']

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_partner
from . import res_company

```

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_company_form_l10n_fr" model="ir.ui.view">
        <field name="name">res.company.form.l10n.fr</field>
        <field name="model">res.company</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="base.view_company_form"/>
        <field name="arch" type="xml">
        <data>
             <xpath expr="//field[@name='company_registry']" position="after">
                 <field name="siret" invisible="not is_france_country"/>
                 <field name="ape" invisible="not is_france_country"/>
             </xpath>
        </data>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_partner_form_l10n_fr" model="ir.ui.view">
        <field name="name">res.partner.form.l10n.fr</field>
        <field name="model">res.partner</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
        <data>
             <xpath expr="//field[@name='ref']" position="after">
                <field name="siret" invisible="not is_company"/>
             </xpath>
        </data>
        </field>
    </record>
</odoo>

```

