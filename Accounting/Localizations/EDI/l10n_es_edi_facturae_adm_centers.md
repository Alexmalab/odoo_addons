# Odoo Module: l10n_es_edi_facturae_adm_centers

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'Spain - Facturae EDI - Administrative Centers Patch',
    'version': '1.0',
    'description': """
    Patch module to fix the missing Administrative Centers in the Facturae EDI.
    """,
    'license': 'LGPL-3',
    'category': 'Accounting/Localizations/EDI',
    'depends': [
        'l10n_es_edi_facturae',
    ],
    'data': [
        'data/l10n_es_edi_facturae_adm_centers.ac_role_type.csv',
        'data/facturae_templates.xml',

        'security/ir.model.access.csv',

        'views/res_partner_views.xml',
    ],
    'demo': [
        'demo/l10n_es_edi_facturae_demo.xml',
    ],
    'auto_install': True,
}

```

## File: data\facturae_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Sub-template used for every instance of AdministrativeCentresType -->
        <template id="administrative_centers_type">
            <AdministrativeCentres>
                <AdministrativeCentre t-foreach="administrative_centers" t-as="ac">
                    <CentreCode t-out="(ac.get('center_code') or '')[:10]"/>
                    <RoleTypeCode t-out="ac.get('role_type_code')"/>
                    <Name t-out="(ac.get('name') or '')[:40]"/>
                    <t t-call="l10n_es_edi_facturae.address_type">
                        <t t-set="partner" t-value="ac.get('partner')"/>
                        <t t-set="partner_country_code" t-value="ac.get('partner_country_code')"/>
                    </t>
                    <t t-call="l10n_es_edi_facturae.contact_details_type">
                        <t t-set="partner" t-value="ac.get('partner')"/>
                        <t t-set="partner_phone" t-value="ac.get('partner_phone')"/>
                    </t>
                    <PhysicalGLN t-out="(ac.get('physical_gln') or '')[:14]"/>
                    <LogicalOperationalPoint t-out="(ac.get('logical_operational_point') or '')[:14]"/>
                </AdministrativeCentre>
            </AdministrativeCentres>
        </template>

        <template id="business_type" inherit_id="l10n_es_edi_facturae.business_type">
            <xpath expr="//t[@t-call='l10n_es_edi_facturae.tax_identification_type']" position="after">
                <t t-call="l10n_es_edi_facturae_adm_centers.administrative_centers_type"/>
            </xpath>
        </template>

        <template id="account_invoice_facturae_export" inherit_id="l10n_es_edi_facturae.account_invoice_facturae_export">
            <xpath expr="//SellerParty//t[@t-set='partner_name']" position="after">
                <t t-set="administrative_centers" t-value="self_party_administrative_centers if is_outstanding else other_party_administrative_centers"/>
            </xpath>
            <xpath expr="//BuyerParty//t[@t-set='partner_name']" position="after">
                <t t-set="administrative_centers" t-value="other_party_administrative_centers if is_outstanding else self_party_administrative_centers"/>
            </xpath>
        </template>
    </data>
</odoo>

```

## File: data\l10n_es_edi_facturae_adm_centers.ac_role_type.csv

```csv
"id","code","name"
"ac_role_type_01","01","Fiscal"
"ac_role_type_02","02","Receiver"
"ac_role_type_03","03","Payer"
"ac_role_type_04","04","Buyer"
"ac_role_type_05","05","Collector"
"ac_role_type_06","06","Seller"
"ac_role_type_07","07","Payment Receiver"
"ac_role_type_08","08","Collection Receiver"
"ac_role_type_09","09","Issuer"

```

## File: models\account_move.py

```python
from odoo import models
from odoo.addons.l10n_es_edi_facturae.models.account_move import COUNTRY_CODE_MAP, PHONE_CLEAN_TABLE


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _l10n_es_edi_facturae_get_administrative_centers(self, partner):
        self.ensure_one()
        administrative_centers = []
        for ac in partner.child_ids.filtered(lambda p: p.type == 'facturae_ac'):
            ac_template = {
                'center_code': ac.l10n_es_edi_facturae_ac_center_code,
                'name': ac.name,
                'partner': ac,
                'partner_country_code': COUNTRY_CODE_MAP[ac.country_code],
                'partner_phone': ac.phone.translate(PHONE_CLEAN_TABLE) if ac.phone else False,
                'physical_gln': ac.l10n_es_edi_facturae_ac_physical_gln,
                'logical_operational_point': ac.l10n_es_edi_facturae_ac_logical_operational_point,
            }
            # An administrative center can have multiple roles, each of which should be reported separately.
            for role in ac.l10n_es_edi_facturae_ac_role_type_ids or [self.env['l10n_es_edi_facturae_adm_centers.ac_role_type']]:
                administrative_centers.append({
                    **ac_template,
                    'role_type_code': role.code,
                })
        return administrative_centers

    def _l10n_es_edi_facturae_export_facturae(self):
        template_values, signature_values = super()._l10n_es_edi_facturae_export_facturae()
        template_values['self_party_administrative_centers'] = self._l10n_es_edi_facturae_get_administrative_centers(
            template_values.get('self_party')
        )
        template_values['other_party_administrative_centers'] = self._l10n_es_edi_facturae_get_administrative_centers(
            template_values.get('other_party')
        )
        return template_values, signature_values

```

## File: models\res_partner.py

```python
from odoo import _, api, fields, models
from odoo.exceptions import ValidationError
from odoo.tools import check_barcode_encoding


class AcRoleType(models.Model):
    _name = 'l10n_es_edi_facturae_adm_centers.ac_role_type'
    _description = 'Administrative Center Role Type'

    code = fields.Char(required=True)
    name = fields.Char(required=True, translate=True)


class Partner(models.Model):
    _inherit = 'res.partner'

    type = fields.Selection(selection_add=[('facturae_ac', 'FACe Center'), ('other',)])
    l10n_es_edi_facturae_ac_center_code = fields.Char(string='Code', size=10, help="Code of the issuing department.")
    l10n_es_edi_facturae_ac_role_type_ids = fields.Many2many(
        string='Roles',
        comodel_name='l10n_es_edi_facturae_adm_centers.ac_role_type',
        help="It indicates the role played by the Operational Point defined as a Workplace/Department.\n"
             "These functions are:\n"
             "- Receiver: Workplace associated to the recipient's tax identification number where the invoice will be received.\n"
             "- Payer: Workplace associated to the recipient's tax identification number responsible for paying the invoice.\n"
             "- Buyer: Workplace associated to the recipient's tax identification number who issued the purchase order.\n"
             "- Collector: Workplace associated to  the issuer's tax identification number responsible for handling the collection.\n"
             "- Fiscal: Workplace associated to the recipient's tax identification number, where an Operational Point mailbox is shared "
             "by different client companies with different tax identification numbers and it is necessary to differentiate between "
             "where the message is received (shared letterbox) and the workplace where it must be stored (recipient company).",
    )
    l10n_es_edi_facturae_ac_physical_gln = fields.Char(
        string='Physical GLN',
        size=14,
        help="Identification of the connection point to the VAN EDI (Global Location Number). Barcode of 13 standard positions. "
        "Codes are registered in Spain by AECOC. The code is made up of the country code (2 positions) Spain is '84' "
        "+ Company code (5 positions) + the remaining positions. The last one is the product + check digit."
    )
    l10n_es_edi_facturae_ac_logical_operational_point = fields.Char(
        string='Logical Operational Point',
        size=14,
        help="Code identifying the company. Barcode of 13 standard positions. Codes are registered in Spain by AECOC. "
        "The code is made up of the country code (2 positions) Spain is '84' + Company code (5 positions) + the remaining positions. "
        "The last one is the product + check digit.",
    )

    @api.constrains('l10n_es_edi_facturae_ac_physical_gln')
    def _validate_l10n_es_edi_facturae_ac_physical_gln(self):
        for p in self:
            if not p.l10n_es_edi_facturae_ac_physical_gln:
                continue
            if not check_barcode_encoding(p.l10n_es_edi_facturae_ac_physical_gln, 'ean13'):
                raise ValidationError(_('The Physical GLN entered is not valid.'))

    @api.constrains('l10n_es_edi_facturae_ac_logical_operational_point')
    def _validate_l10n_es_edi_facturae_ac_logical_operational_point(self):
        for p in self:
            if not p.l10n_es_edi_facturae_ac_logical_operational_point:
                continue
            if not check_barcode_encoding(p.l10n_es_edi_facturae_ac_logical_operational_point, 'ean13'):
                raise ValidationError(_('The Logical Operational Point entered is not valid.'))

```

## File: models\__init__.py

```python
from . import account_move
from . import res_partner

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
l10n_es_edi_facturae_adm_centers.access_l10n_es_edi_facturae_ac_role_type_invoice,access_l10n_es_edi_facturae_ac_role_type,l10n_es_edi_facturae_adm_centers.model_l10n_es_edi_facturae_adm_centers_ac_role_type,account.group_account_invoice,1,0,0,0
l10n_es_edi_facturae_adm_centers.access_l10n_es_edi_facturae_ac_role_type_readonly,access_l10n_es_edi_facturae_ac_role_type,l10n_es_edi_facturae_adm_centers.model_l10n_es_edi_facturae_adm_centers_ac_role_type,account.group_account_readonly,1,0,0,0

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_partner_form_inherit_l10n_es_edi_facturae" model="ir.ui.view">
        <field name="name">res.partner.form.inherit.l10n_es_edi_facturae</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='child_ids']//div[hasclass('oe_edit_only')]" position="inside">
                <p class="mb-0" invisible="type != 'facturae_ac'">
                    <span>Administrative Center for Spain Public Administrations. Used in Spanish electronic invoices.</span>
                </p>
            </xpath>
            <xpath expr="//field[@name='child_ids']//form//field[@name='name']" position="after">
                <field name="l10n_es_edi_facturae_ac_center_code" invisible="type != 'facturae_ac'"/>
                <field name="l10n_es_edi_facturae_ac_role_type_ids" widget="many2many_tags" invisible="type != 'facturae_ac'"/>
                <field name="l10n_es_edi_facturae_ac_physical_gln" invisible="type != 'facturae_ac'"/>
                <field name="l10n_es_edi_facturae_ac_logical_operational_point" invisible="type != 'facturae_ac'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

