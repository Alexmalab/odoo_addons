# Odoo Module: l10n_latam_base

Category: Accounting/Localizations

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models
from odoo import api, SUPERUSER_ID


def _set_default_identification_type(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    env.cr.execute(
        """
            UPDATE res_partner
               SET l10n_latam_identification_type_id = %s
        """,
        [env.ref('l10n_latam_base.it_vat').id]
    )

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'LATAM Localization Base',
    'version': '1.0',
    'category': 'Accounting/Localizations',
    'sequence': 14,
    'author': 'Odoo, ADHOC SA',
    'summary': 'LATAM Identification Types',
    'description': """
Add a new model named "Identification Type" that extend the vat field functionality in the partner and let the user to identify (an eventually invoice) to contacts not only with their fiscal tax ID (VAT) but with other types of identifications like national document, passport, foreign ID, etc. With this module installed you will see now in the partner form view two fields:

* Identification Type
* Identification Number

This behavior is a common requirement for some latam countries like Argentina and Chile. If your localization has this requirements then you need to depend on this module and define in your localization module the identifications types that are used in your country. Generally these types of identifications are defined by the government authorities that regulate the fiscal operations. For example:

* AFIP in Argentina defines DNI, CUIT (vat for legal entities), CUIL (vat for natural person), and another 80 valid identification types.

Each identification holds this information:

* name: short name of the identification
* description: could be the same short name or a long name
* country_id: the country where this identification belongs
* is_vat: identify this record as the corresponding VAT for the specific country.
* sequence: let us to sort the identification types depending on the ones that are most used.
* active: we can activate/inactivate identifications to make it easier to our customers

In order to make this module compatible for multi-company environments where we have companies that does not need/support this requirement, we have added generic identification types and generic rules to manage the contact information and make it transparent for the user when only use the VAT as we formerly know.

Generic Identifications:

* VAT: The Fiscal Tax Identification or VAT number, by default will be selected as identification type so the user will only need to add the related vat number.
* Passport
* Foreign ID (Foreign National Document)

Rules when creating a new partner: We will only see the identification types that are meaningful, taking into account these rules:

* If the partner have not country address set: Will show the generic identification types plus the ones defined in the partner's related company country (If the partner has not specific company then will show the identification types related to the current user company)

* If the partner has country address : will show the generic identification types plus the ones defined for the country of the partner.

When creating a new company, will set to the related partner always the related country is_vat identification type.

All the defined identification types can be reviewed and activate/deactivate in "Contacts / Configuration / Identification Type" menu.

This module is compatible with base_vat module in order to be able to validate VAT numbers for each country that have or not have the possibility to manage multiple identification types.
""",
    'depends': [
        'contacts',
        'base_vat',
    ],
    'data': [
        'data/l10n_latam.identification.type.csv',
        'views/res_partner_view.xml',
        'views/l10n_latam_identification_type_view.xml',
        'security/ir.model.access.csv',
    ],
    'installable': True,
    'auto_install': False,
    'application': False,
    'post_init_hook': '_set_default_identification_type',
    'license': 'LGPL-3',
}

```

## File: data\l10n_latam.identification.type.csv

```csv
id,name,sequence,is_vat
l10n_latam_base.it_vat,VAT,80,TRUE
l10n_latam_base.it_pass,Passport,90,
l10n_latam_base.it_fid,Foreign ID,100,

```

## File: models\l10n_latam_identification_type.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api
from odoo.osv import expression


class L10nLatamIdentificationType(models.Model):
    _name = 'l10n_latam.identification.type'
    _description = "Identification Types"
    _order = 'sequence'

    sequence = fields.Integer(default=10)
    name = fields.Char(translate=True, required=True,)
    description = fields.Char()
    active = fields.Boolean(default=True)
    is_vat = fields.Boolean()
    country_id = fields.Many2one('res.country')

    def name_get(self):
        multi_localization = len(self.search([]).mapped('country_id')) > 1
        return [(rec.id, '%s%s' % (
            rec.name, multi_localization and rec.country_id and ' (%s)' % rec.country_id.code or '')) for rec in self]

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api


class ResCompany(models.Model):
    _inherit = 'res.company'

    @api.model
    def create(self, vals):
        """ If exists, use specific vat identification.type for the country of the company """
        country_id = vals.get('country_id')
        if country_id:
            country_vat_type = self.env['l10n_latam.identification.type'].search(
                [('is_vat', '=', True), ('country_id', '=', country_id)], limit=1)
            if country_vat_type:
                self = self.with_context(default_l10n_latam_identification_type_id=country_vat_type.id)
        return super().create(vals)

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_latam_identification_type_id = fields.Many2one('l10n_latam.identification.type',
        string="Identification Type", index=True, auto_join=True,
        default=lambda self: self.env.ref('l10n_latam_base.it_vat', raise_if_not_found=False),
        help="The type of identification")
    vat = fields.Char(string='Identification Number', help="Identification Number for selected type")

    @api.model
    def _commercial_fields(self):
        return super()._commercial_fields() + ['l10n_latam_identification_type_id']

    @api.constrains('vat', 'l10n_latam_identification_type_id')
    def check_vat(self):
        with_vat = self.filtered(lambda x: x.l10n_latam_identification_type_id.is_vat)
        return super(ResPartner, with_vat).check_vat()

    @api.onchange('country_id')
    def _onchange_country(self):
        country = self.country_id or self.company_id.account_fiscal_country_id or self.env.company.account_fiscal_country_id
        identification_type = self.l10n_latam_identification_type_id
        if not identification_type or (identification_type.country_id != country):
            self.l10n_latam_identification_type_id = self.env['l10n_latam.identification.type'].search(
                [('country_id', '=', country.id), ('is_vat', '=', True)], limit=1) or self.env.ref(
                    'l10n_latam_base.it_vat', raise_if_not_found=False)

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import l10n_latam_identification_type
from . import res_partner
from . import res_company

```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
"access_latam_identification_type_all","latam id type all","model_l10n_latam_identification_type",,1,0,0,0
"access_latam_identification_type_manager","latam id type manager","model_l10n_latam_identification_type","base.group_partner_manager",1,1,0,0

```

## File: views\l10n_latam_identification_type_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="view_l10n_latam_identification_type_tree" model="ir.ui.view">
        <field name="name">l10n_latam.identification.type.tree</field>
        <field name="model">l10n_latam.identification.type</field>
        <field name="type">tree</field>
        <field name="arch" type="xml">
            <tree decoration-muted="(not active)" create="0" edit="0">
                <field name="name"/>
                <field name="description"/>
                <field name="country_id"/>
                <field name="active" widget="boolean_toggle"/>
            </tree>
        </field>
    </record>

    <record id="view_l10n_latam_identification_type_search" model="ir.ui.view">
        <field name="name">l10n_latam.identification.type.search</field>
        <field name="model">l10n_latam.identification.type</field>
        <field name="type">search</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <field name="description"/>
                <field name="country_id"/>
                <filter name="active" string="Active" domain="[('active','=',True)]" help="Show active identification types"/>
                <filter name="inactive" string="Archived" domain="[('active','=',False)]" help="Show archived identification types"/>
            </search>
        </field>
    </record>

    <record id="action_l10n_latam_identification_type" model="ir.actions.act_window">
        <field name="name">Identification Type</field>
        <field name="res_model">l10n_latam.identification.type</field>
        <field name="view_mode">tree</field>
        <field name="search_view_id" ref="view_l10n_latam_identification_type_search"/>
        <field name="domain">['|', ('active', '=', True), ('active', '=', False)]</field>
        <field name="context">{"search_default_active":1}</field>
    </record>

    <menuitem action="action_l10n_latam_identification_type"
            id="menu_l10n_latam_identification_type"
            parent="contacts.res_partner_menu_config"/>

</odoo>

```

## File: views\res_partner_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="view_partner_latam_form" model="ir.ui.view">
        <field name="name">view_partner_latam_form</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="model">res.partner</field>
        <field name="priority">100</field>
        <field type="xml" name="arch">
            <field name="vat" position="attributes">
                <attribute name="invisible">1</attribute>
            </field>
            <field name="vat" position="after">
                <label for="l10n_latam_identification_type_id" string="Identification Number"/>
                <div>
                    <field name="l10n_latam_identification_type_id" options="{'no_open': True, 'no_create': True}" placeholder="Type" attrs="{'readonly': [('parent_id','!=',False)]}" class="oe_inline" domain="country_id and ['|', ('country_id', '=', False), ('country_id', '=', country_id)] or []" required="True"/>
                    <span class="oe_read_only"> - </span>
                    <field name="vat" placeholder="Number" class="oe_inline" attrs="{'readonly': [('parent_id','!=',False)]}"/>
                </div>
            </field>
        </field>
    </record>

</odoo>

```

