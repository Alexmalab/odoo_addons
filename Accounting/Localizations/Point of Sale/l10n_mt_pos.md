# Odoo Module: l10n_mt_pos

Category: Accounting/Localizations/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import wizards
from . import reports

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    "name": "Malta - Point of Sale",
    "version": "1.0",
    "category": "Accounting/Localizations/Point of Sale",
    "description": """Malta Compliance Letter for EXO Number""",
    "countries": ["mt"],
    "depends": [
        "point_of_sale",
    ],
    "data": [
        'security/ir.model.access.csv',
        'wizards/compliance_letter_view.xml',
        'reports/compliance_letter_report.xml',
    ],
    "installable": True,
    "auto_install": True,
    "license": "LGPL-3",
}

```

## File: reports\compliance_letter_report.xml

```xml
<odoo>
    <template id="report_compliance_letter_template">
        <t t-call="web.basic_layout">
            <div class="page px-4">
                <h3 class="text-center mb-4">
                    Odoo Point Of Sale application, version
                    <t t-esc="version"/>
                </h3>
                <div class="mb-1"><t t-esc="date"/></div>
                <div class="mb-1"><t t-esc="name"/></div>
                <div class="mb-1"><t t-esc="address"/></div>
                <div class="mb-4"><t t-esc="vat"/></div>
                <div class="mb-3">Dear Sir/Madam,</div>
                <div class="ps-3 mb-4" style="line-height: 1.5;">
                    Odoo S.A., (Belgium) is a registered company with the Trade and Companies Register of Nivelles with VAT number BE0477472701, having its registered office at Chaussée de Namur, 40, 1367 Grand-Rosière, Belgium ("Odoo"). Odoo provides Enterprise Resource Planning (ERP) cloud and on-premise applications to customers worldwide, incorporating a browser-based Point-of-Sale (POS) application.
                </div>
                <div class="ps-3 mb-3" style="line-height: 1.5;">
                    Odoo hereby declares that the POS application in the version
                    <t t-esc="version"/> provides the following functions and controls:
                </div>
                <ol class="ps-5 py-1 mb-4">
                    <li class="mb-2">Monitor cash register adjustments and easily verify cash contents at the end of the day</li>
                    <li class="mb-2">Keep track of daily sales and totals for every payment type</li>
                    <li class="mb-2">View all past orders as well as search by customer, product, cashier, or date</li>
                    <li class="mb-2">Advertise your current promotions, hours of operation, and upcoming events on your printed receipts</li>
                    <li class="mb-2">Set customer prices or offer percentage-based discounts on either a single product or the entire order</li>
                    <li class="mb-2">Payments are directly integrated into Odoo Accounting, making bookkeeping simple and reliable</li>
                    <li class="mb-2">Generate and print invoices for your business customers</li>
                    <li class="mb-2">Cash, checks, and credit card payment methods are available. New types of payments can also be added</li>
                    <li class="mb-2">Quickly find products by their name or barcode with the built-in search</li>
                    <li class="mb-2">Register customers' VAT numbers and apply them to invoices</li>
                    <li class="mb-2">Point-of-sale transactions cannot be tampered with through the application</li>
                </ol>
                <div class="ps-3 mb-4">Signed this day: <t t-esc="date"/></div>
                <div class="text-center mb-2">Odoo SA, BE0477472701</div>
                <div class="text-center">40 Chaussée de Namur, 1367 Grand-Rosière, Belgium</div>
            </div>
        </t>
    </template>
    <record id="report_compliance_letter" model="ir.actions.report">
        <field name="name">Compliance Letter</field>
        <field name="model">res.company</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">l10n_mt_pos.report_compliance_letter_template</field>
        <field name="report_file">l10n_mt_pos.report_compliance_letter_template</field>
        <field name="binding_model_id" ref="web.model_res_company"/>
        <field name="binding_type">report</field>
        <field name="print_report_name">'Compliance Letter'</field>
    </record>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_compliance_letter_user,compliance.letter user,model_compliance_letter_wizard,base.group_user,1,1,1,1
access_compliance_letter_admin,compliance.letter admin,model_compliance_letter_wizard,base.group_system,1,1,1,1

```

## File: wizards\compliance_letter.py

```python
from odoo import models, fields, release, _
from datetime import datetime
from odoo.exceptions import UserError

class ComplianceLetter(models.TransientModel):
    _name = 'compliance.letter.wizard'
    _description = 'Compliance Letter for EXO Number'

    company_id = fields.Many2one('res.company', string='Company', required=True, default=lambda self: self.env.company)

    def generate_letter(self):
        if self.company_id.country_id.code != 'MT':
            raise UserError(_("Compliance letters can only be created for companies registered in Malta. Please ensure the company's country is set to Malta."))

        data = {
            "version": self._get_odoo_version(),
            "date": self._get_formatted_date(),
            "name": self.company_id.name,
            "vat": self.company_id.vat,
            "address": self.company_id.partner_id.contact_address,
        }
        return self.env.ref('l10n_mt_pos.report_compliance_letter').report_action([], data=data)

    def _get_formatted_date(self):
        """Returns the formatted date as 'Date (Month, xxth, 20XX)'."""
        date_obj = datetime.strptime(str(fields.Date.today()), '%Y-%m-%d')
        day = date_obj.day
        day_suffix = 'th' if 11 <= day <= 13 else {1: 'st', 2: 'nd', 3: 'rd'}.get(day % 10, 'th')
        formatted_date = date_obj.strftime(f"%B {day}{day_suffix}, %Y")
        return formatted_date

    def _get_odoo_version(self):
        return release.major_version

```

## File: wizards\compliance_letter_view.xml

```xml
<odoo>
   <record id="view_generate_compliance_letter" model="ir.ui.view">
        <field name="name">compliance.letter.wizard.form</field>
        <field name="model">compliance.letter.wizard</field>
        <field name="arch" type="xml">
            <form string="Compliance Letter">
                <label string="Company" for="company_id"/>
                <field name="company_id"/>
                <footer>
                    <button name="generate_letter" string="Print" type="object" class="btn-primary"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" />
                </footer>
            </form>
        </field>
    </record>
    <record id="action_generate_compliance_letter" model="ir.actions.act_window">
        <field name="name">Compliance Letter</field>
        <field name="res_model">compliance.letter.wizard</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
    <menuitem id="pos_mt_statements_menu" name="Malta EXO" parent="point_of_sale.menu_point_rep" sequence="6" />
    <menuitem id="menu_compliance_letter" name="Compliance Letter"
              action="l10n_mt_pos.action_generate_compliance_letter"
              parent="l10n_mt_pos.pos_mt_statements_menu" sequence="10"/>
</odoo>

```

## File: wizards\__init__.py

```python
from . import compliance_letter

```

