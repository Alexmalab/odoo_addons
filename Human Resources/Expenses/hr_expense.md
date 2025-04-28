# Odoo Module: hr_expense

Category: Human Resources/Expenses

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'Expenses',
    'version': '2.0',
    'category': 'Human Resources/Expenses',
    'sequence': 70,
    'summary': 'Submit, validate and reinvoice employee expenses',
    'description': """
Manage expenses by Employees
============================

This application allows you to manage your employees' daily expenses. It gives you access to your employees’ fee notes and give you the right to complete and validate or refuse the notes. After validation it creates an invoice for the employee.
Employee can encode their own expenses and the validation flow puts it automatically in the accounting after validation by managers.


The whole flow is implemented as:
---------------------------------
* Draft expense
* Submitted by the employee to his manager
* Approved by his manager
* Validation by the accountant and accounting entries creation

This module also uses analytic accounting and is compatible with the invoice on timesheet module so that you are able to automatically re-invoice your customers' expenses if your work by project.
    """,
    'website': 'https://www.odoo.com/app/expenses',
    'depends': ['account', 'web_tour', 'hr'],
    'data': [
        'security/hr_expense_security.xml',
        'security/ir.model.access.csv',
        'data/digest_data.xml',
        'data/mail_activity_type_data.xml',
        'data/mail_alias_data.xml',
        'data/mail_message_subtype_data.xml',
        'data/mail_templates.xml',
        'data/hr_expense_sequence.xml',
        'data/hr_expense_data.xml',
        'wizard/hr_expense_refuse_reason_views.xml',
        'wizard/hr_expense_approve_duplicate_views.xml',
        'wizard/hr_expense_split_wizard_views.xml',
        'views/hr_expense_views.xml',
        'views/mail_activity_views.xml',
        'security/ir_rule.xml',
        'report/hr_expense_report.xml',
        'views/account_move_views.xml',
        'views/account_payment_views.xml',
        'views/hr_department_views.xml',
        'views/res_config_settings_views.xml',
        'views/account_journal_dashboard.xml',
    ],
    'demo': ['data/hr_expense_demo.xml'],
    'installable': True,
    'application': True,
    'assets': {
        'web.assets_backend': [
            'hr_expense/static/src/components/*.js',
            'hr_expense/static/src/components/*.xml',
            'hr_expense/static/src/mixins/*.js',
            'hr_expense/static/src/views/*.js',
            'hr_expense/static/src/views/*.xml',
            'hr_expense/static/src/scss/hr_expense.scss',
            'hr_expense/static/src/js/tours/*.js',
        ],
        'web.assets_tests': [
            'hr_expense/static/tests/tours/expense_upload_tours.js',
            'hr_expense/static/tests/tours/expense_form_tours.js',
        ],
        'web.report_assets_common': [
            'hr_expense/static/src/scss/hr_expense.scss',
        ],
        'web.qunit_suite_tests': [
            'hr_expense/static/tests/**/*.js',
            ('remove', 'hr_expense/static/tests/mobile/**/*.js'),
        ],
        'web.qunit_mobile_suite_tests': [
            'hr_expense/static/tests/mobile/**/*.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\digest_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <data>
        <record id="digest_tip_hr_expense_0" model="digest.tip">
            <field name="name">Tip: Snap pictures of your receipts with the remote app</field>
            <field name="sequence">1100</field>
            <field name="group_id" ref="base.group_user" />
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Snap pictures of your receipts with the remote app</p>
    <p class="tip_content">Do not keep your expense tickets in your pockets any longer. Just snap a picture of your receipt and let Odoo digitalizes it for you. The OCR and Artificial Intelligence will fill the data automatically.</p>
    <img src="https://download.odoocdn.com/digests/hr_expense/static/src/img/milk-expense.png" width="540" class="illustration_border" />
</div>
            </field>
        </record>
    </data>
</odoo>

```

## File: data\hr_expense_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- sgv note - icons will be replaced. Design task is ongoing -->
        <record id="expense_product_meal" model="product.product">
            <field name="name">Meals</field>
            <field name="description">Restaurants, business lunches, etc.</field>
            <field name="standard_price">0.0</field>
            <field name="type">service</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">FOOD</field>
            <field name="can_be_expensed" eval="True"/>
            <field name="sale_ok" eval="False"/>
            <field name="purchase_ok" eval="False"/>
            <field name="image_1920" type="base64" file="hr_expense/static/img/food.svg"/>
        </record>
        <record id="expense_product_travel_accommodation" model="product.product">
            <field name="name">Travel &amp; Accommodation</field>
            <field name="description">Hotel, plane ticket, taxi, etc.</field>
            <field name="standard_price">0.0</field>
            <field name="type">service</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="default_code">TRANS &amp; ACC</field>
            <field name="can_be_expensed" eval="True"/>
            <field name="sale_ok" eval="False"/>
            <field name="purchase_ok" eval="False"/>
            <field name="image_1920" type="base64" file="hr_expense/static/img/transport.svg"/>
        </record>
        <record id="expense_product_mileage" model="product.product">
            <field name="name">Mileage</field>
            <field name="standard_price">1.0</field>
            <field name="type">service</field>
            <field name="uom_id" ref="uom.product_uom_km"/>
            <field name="uom_po_id" ref="uom.product_uom_km"/>
            <field name="default_code">MIL</field>
            <field name="can_be_expensed" eval="True"/>
            <field name="sale_ok" eval="False"/>
            <field name="purchase_ok" eval="False"/>
            <field name="image_1920" type="base64" file="hr_expense/static/img/mileage.svg"/>
        </record>
        <record id="expense_product_gift" model="product.product">
            <field name="name">Gifts</field>
            <field name="description">Gifts to customers or vendors</field>
            <field name="standard_price">0.0</field>
            <field name="type">service</field>
            <field name="uom_id" ref="uom.product_uom_km"/>
            <field name="uom_po_id" ref="uom.product_uom_km"/>
            <field name="default_code">GIFT</field>
            <field name="can_be_expensed" eval="True"/>
            <field name="sale_ok" eval="False"/>
            <field name="purchase_ok" eval="False"/>
            <field name="image_1920" type="base64" file="hr_expense/static/img/gift.svg"/>
        </record>
        <record id="expense_product_communication" model="product.product">
            <field name="name">Communication</field>
            <field name="description">Phone bills, postage, etc.</field>
            <field name="standard_price">0.0</field>
            <field name="type">service</field>
            <field name="uom_id" ref="uom.product_uom_km"/>
            <field name="uom_po_id" ref="uom.product_uom_km"/>
            <field name="default_code">COMM</field>
            <field name="can_be_expensed" eval="True"/>
            <field name="sale_ok" eval="False"/>
            <field name="purchase_ok" eval="False"/>
            <field name="image_1920" type="base64" file="hr_expense/static/img/communication.svg"/>
        </record>
        <record id="product_product_no_cost" model="product.product">
            <field name="name">Others</field>
            <field name="standard_price">0.0</field>
            <field name="type">service</field>
            <field name="default_code">EXP_GEN</field>
            <field name="categ_id" ref="product.cat_expense"/>
            <field name="can_be_expensed" eval="True"/>
            <field name="image_1920" type="base64" file="hr_expense/static/img/other.svg"/>
        </record>
    </data>
    <data>
        <function model="ir.config_parameter" name="set_param" eval="('hr_expense.use_mailgateway', True)"/>
    </data>
</odoo>

```

## File: data\hr_expense_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="base.user_demo" model="res.users">
            <field name="groups_id" eval="[(3, ref('hr_expense.group_hr_expense_manager'))]"/>
        </record>

        <record id="hr.employee_mit" model="hr.employee">
            <field name="private_email">douglas.fletcher51@example.com</field>
            <field name="private_phone">(132)-553-7242</field>
        </record>

        <record id="hr_expense_account_journal" model="account.journal">
            <field name="name">Expense</field>
            <field name="code">EXP</field>
            <field name="type">purchase</field>
            <!-- avoid being selected as default journal -->
            <field name="sequence">99</field>
            <field name="alias_name">purchase_expense</field>
        </record>

        <record id="hr.employee_fme" model="hr.employee">
            <field name="private_street">Chaussée de Namur, 40</field>
            <field name="private_zip">1367</field>
            <field name="private_city">Grand-Rosière-Hottomont</field>
            <field name="private_country_id" ref="base.be"/>
        </record>

        <record id="hr.employee_al" model="hr.employee">
            <field name="private_street">Chaussée de Namur, 40</field>
            <field name="private_zip">1367</field>
            <field name="private_city">Grand-Rosière-Hottomont</field>
            <field name="private_country_id" ref="base.be"/>
        </record>

        <!-- ++++++++++++++ Expense sheet for Admin ++++++++++++++-->
        <record id="screen_expense" model="hr.expense">
            <field name="name">Screen</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="analytic_distribution" eval="{ref('analytic.analytic_our_super_product'): 100}"/>
            <field name="product_id" ref="product_product_no_cost"/>
            <field eval="289.0" name="total_amount_currency"/>
            <field name="date" eval="time.strftime('%Y')+'-04-03'"/>
        </record>

        <record id="laptop_expense" model="hr.expense">
            <field name="name">Laptop</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="analytic_distribution" eval="{ref('analytic.analytic_our_super_product'): 100}"/>
            <field name="product_id" ref="product_product_no_cost"/>
            <field eval="889.0" name="total_amount_currency"/>
            <field name="date" eval="time.strftime('%Y')+'-04-03'"/>
        </record>

        <record id="travel_admin_by_car_expense" model="hr.expense">
            <field name="name">Travel by car</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="product_id" ref="expense_product_mileage"/>
            <field name="product_uom_id" ref="uom.product_uom_km"/>
            <field eval="108.84" name="quantity"/>
        </record>

        <record id="breakfast_admin_expense" model="hr.expense">
            <field name="name">BreakFast</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="product_id" ref="expense_product_meal"/>
            <field eval="20" name="total_amount_currency"/>
        </record>

        <record id="travel_ny_sheet" model="hr.expense.sheet">
            <field name="name">Commercial Travel at New York</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="state">approve</field>
        </record>

        <record id="travel_by_air_expense" model="hr.expense">
            <field name="name">Travel by Air</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="analytic_distribution" eval="{ref('analytic.analytic_our_super_product'): 100}"/>
            <field name="product_id" ref="expense_product_travel_accommodation"/>
            <field eval="700.0" name="total_amount_currency"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="date" eval="time.strftime('%Y-%m')+'-12'"/>
            <field name="sheet_id" ref="travel_ny_sheet"/>
        </record>

        <record id="hotel_bill_expense" model="hr.expense">
            <field name="name">Hotel Expenses</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="analytic_distribution" eval="{ref('analytic.analytic_nebula'): 100}"/>
            <field name="product_id" ref="expense_product_travel_accommodation"/>
            <field eval="2000.0" name="total_amount_currency"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="date" eval="time.strftime('%Y-%m')+'-17'"/>
            <field name="sheet_id" ref="travel_ny_sheet"/>
        </record>

        <record id="lunch_customer_bill_expense" model="hr.expense">
            <field name="name">Lunch with Customer</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="analytic_distribution" eval="{ref('analytic.analytic_nebula'): 100}"/>
            <field name="product_id" ref="expense_product_meal"/>
            <field eval="152.8" name="total_amount_currency"/>
            <field name="date" eval="time.strftime('%Y-%m')+'-13'"/>
            <field name="sheet_id" ref="travel_ny_sheet"/>
        </record>

        <record id="lunch_bill_expense" model="hr.expense">
            <field name="name">Lunch</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="analytic_distribution" eval="{ref('analytic.analytic_nebula'): 100}"/>
            <field name="product_id" ref="expense_product_meal"/>
            <field name="date" eval="time.strftime('%Y-%m')+'-15'"/>
            <field eval="56.8" name="total_amount_currency"/>
            <field name="sheet_id" ref="travel_ny_sheet"/>
        </record>

        <!-- ++++++++++++++ Expense sheet for Demo ++++++++++++++-->
        <record id="customer_meeting_sheet" model="hr.expense.sheet">
            <field name="name">Customer meeting</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="state">submit</field>
        </record>

        <record id="travel_demo_by_car_expense" model="hr.expense">
            <field name="name">Travel by Car</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="analytic_distribution" eval="{ref('analytic.analytic_our_super_product'): 100}"/>
            <field name="product_id" ref="expense_product_mileage"/>
            <field name="product_uom_id" ref="uom.product_uom_km"/>
            <field name="date" eval="time.strftime('%Y')+'-01-15'"/>
            <field eval="120.85" name="quantity"/>
            <field name="sheet_id" ref="customer_meeting_sheet"/>
        </record>

        <record id="lunch_demo_customer_bill_expense" model="hr.expense">
            <field name="name">Lunch with Customer</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="analytic_distribution" eval="{ref('analytic.analytic_nebula'): 100}"/>
            <field name="product_id" ref="product_product_no_cost"/>
            <field name="date" eval="time.strftime('%Y')+'-01-15'"/>
            <field eval="152.8" name="total_amount_currency"/>
            <field name="sheet_id" ref="customer_meeting_sheet"/>
        </record>

        <!-- ++++++++++++++ Expense sheet for Keith Byrd ++++++++++++++-->
        <record id="team_building_sheet" model="hr.expense.sheet">
            <field name="name">Team Building</field>
            <field name="employee_id" ref="hr.employee_fme"/>
            <field name="state">submit</field>
        </record>

        <record id="pizzas_bill_expense" model="hr.expense">
            <field name="name">Pizzas</field>
            <field name="employee_id" ref="hr.employee_fme"/>
            <field name="analytic_distribution" eval="{ref('analytic.analytic_nebula'): 100}"/>
            <field name="product_id" ref="expense_product_meal"/>
            <field name="date" eval="time.strftime('%Y-%m')+'-05'"/>
            <field eval="154" name="total_amount_currency"/>
            <field name="sheet_id" ref="team_building_sheet"/>
        </record>

        <record id="drinks_bill_expense" model="hr.expense">
            <field name="name">Drinks</field>
            <field name="employee_id" ref="hr.employee_fme"/>
            <field name="analytic_distribution" eval="{ref('analytic.analytic_nebula'): 100}"/>
            <field name="product_id" ref="expense_product_meal"/>
            <field name="date" eval="time.strftime('%Y-%m')+'-05'"/>
            <field eval="42.5" name="total_amount_currency"/>
            <field name="sheet_id" ref="team_building_sheet"/>
        </record>

        <record id="paintball_bill_expense" model="hr.expense">
            <field name="name">Paintball</field>
            <field name="employee_id" ref="hr.employee_fme"/>
            <field name="analytic_distribution" eval="{ref('analytic.analytic_nebula'): 100}"/>
            <field name="product_id" ref="product_product_no_cost"/>
            <field name="date" eval="time.strftime('%Y-%m')+'-05'"/>
            <field eval="25" name="total_amount_currency"/>
            <field name="sheet_id" ref="team_building_sheet"/>
        </record>

        <!-- ++++++++++++++ Expense sheet for Ronnie Hart ++++++++++++++-->
        <record id="office_furniture_sheet" model="hr.expense.sheet">
            <field name="name">Office furniture</field>
            <field name="employee_id" ref="hr.employee_al"/>
            <field name="state">submit</field>
        </record>

        <record id="chair_bill_expense" model="hr.expense">
            <field name="name">Chairs</field>
            <field name="employee_id" ref="hr.employee_al"/>
            <field name="analytic_distribution" eval="{ref('analytic.analytic_nebula'): 100}"/>
            <field name="product_id" ref="product_product_no_cost"/>
            <field name="date" eval="time.strftime('%Y')+'-06-02'"/>
            <field eval="55.75" name="total_amount_currency"/>
            <field name="sheet_id" ref="office_furniture_sheet"/>
        </record>

        <record id="lamp_bill_expense" model="hr.expense">
            <field name="name">Lamp</field>
            <field name="employee_id" ref="hr.employee_al"/>
            <field name="analytic_distribution" eval="{ref('analytic.analytic_nebula'): 100}"/>
            <field name="product_id" ref="product_product_no_cost"/>
            <field name="date" eval="time.strftime('%Y')+'-06-02'"/>
            <field eval="28.99" name="total_amount_currency"/>
            <field name="sheet_id" ref="office_furniture_sheet"/>
        </record>

        <!-- ++++++++++++++ Expense for Randall Lewis ++++++++++++++-->
        <record id="afterwork_bill_expense" model="hr.expense">
            <field name="name">Car tyres</field>
            <field name="employee_id" ref="hr.employee_stw"/>
            <field name="analytic_distribution" eval="{ref('analytic.analytic_nebula'): 100}"/>
            <field name="product_id" ref="product_product_no_cost"/>
            <field name="date" eval="time.strftime('%Y')+'-03-15'"/>
            <field eval="450.58" name="total_amount_currency"/>
        </record>

    </data>
</odoo>

```

## File: data\hr_expense_sequence.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="seq_hr_expense_invoice" model="ir.sequence">
            <field name="name">Expense invoice</field>
            <field name="code">hr.expense.invoice</field>
            <field name="prefix">EXP/</field>
            <field name="padding">3</field>
        </record>

    </data>
</odoo>

```

## File: data\mail_activity_type_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mail_act_expense_approval" model="mail.activity.type">
            <field name="name">Expense Approval</field>
            <field name="icon">fa-dollar</field>
            <field name="res_model">hr.expense.sheet</field>
        </record>
    </data>
</odoo>

```

## File: data\mail_alias_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- default alias for expenses -->
        <record id="mail_alias_expense" model="mail.alias">
            <field name="alias_name">expense</field>
            <field name="alias_model_id" ref="model_hr_expense"/>
            <field name="alias_contact">employees</field>
        </record>
    </data>
</odoo>

```

## File: data\mail_message_subtype_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Expense-related subtypes for messaging / Chatter -->
        <record id="mt_expense_approved" model="mail.message.subtype">
            <field name="name">Approved</field>
            <field name="res_model">hr.expense.sheet</field>
            <field name="default" eval="True"/>
            <field name="description">Expense report approved</field>
        </record>
        <record id="mt_expense_refused" model="mail.message.subtype">
            <field name="name">Refused</field>
            <field name="res_model">hr.expense.sheet</field>
            <field name="default" eval="True"/>
            <field name="description">Expense report refused</field>
        </record>
        <record id="mt_expense_paid" model="mail.message.subtype">
            <field name="name">Paid</field>
            <field name="res_model">hr.expense.sheet</field>
            <field name="description">Expense report paid</field>
            <field name="default" eval="True"/>
        </record>
        <record id="mt_expense_reset" model="mail.message.subtype">
            <field name="name">Draft</field>
            <field name="res_model">hr.expense.sheet</field>
            <field name="default" eval="True"/>
            <field name="description">Expense report reset to Draft</field>
        </record>
        <record id="mt_expense_entry_delete" model="mail.message.subtype">
            <field name="name">Journal Entry Deleted</field>
            <field name="res_model">hr.expense.sheet</field>
            <field name="default" eval="True"/>
            <field name="description">Journal entry deleted</field>
        </record>
    </data>
</odoo>

```

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <template id="hr_expense_template_refuse_reason">
            <p>Your Expense Report <t t-out="name"/> has been refused</p>
            <ul class="o_timeline_tracking_value_list">
                <li>Reason: <t t-out="reason"/></li>
            </ul>
        </template>

        <template id="hr_expense_template_register">
            <p>Dear <t t-out="expense.employee_id.name"/>,</p>
            <p>
                Your expense has been successfully registered.
                <t t-if="expense.employee_id.user_id">
                    You can now submit it to the manager from the following link.
                </t>
            </p>
            <p t-if="expense.product_id">
                Category: <t t-out="expense.product_id.name"/>
            </p>
            <div t-else="">
                <p>Category: not found</p>
                <p>The first word of the email subject did not correspond to any category code. You'll have to set the category manually on the expense.</p>
            </div>
            <p>
                Price: <t t-out="expense.price_unit"/><t t-out="expense.currency_id.symbol"/>
            </p>
            <p t-if="expense.employee_id.user_id">
                <br/>
                <a t-att-href="'/web#id=%s&amp;model=hr.expense&amp;view_type=form' % (expense.id)" style="background-color: #9E588B; margin-top: 10px; padding: 10px; text-decoration: none; color: #fff; border-radius: 5px; font-size: 16px;">View Expense</a>
            </p>
        </template>

        <template id="hr_expense_template_register_no_user">
            <div style="background:#F0F0F0;color:#515166;padding:10px 0px;font-family:Arial,Helvetica,sans-serif;font-size:14px;">
                <table style="width:600px;margin:5px auto;" t-if="not expense.employee_id.company_id.uses_default_logo">
                    <tbody>
                        <tr>
                            <td><a href="/"><img t-attf-src="/logo.png?company={{ expense.employee_id.company_id.id }}" style="vertical-align:baseline;max-width:100px;max-height:50px;" /></a></td>
                        </tr>
                    </tbody>
                </table>
                <table style="width:600px;margin:0px auto;background:white;border:1px solid #e1e1e1;">
                    <tbody>
                        <tr>
                            <td style="padding:15px 20px 10px 20px;">
                                <t t-call="hr_expense.hr_expense_template_register"/>
                                <p style="color:#9E588B;">Powered by <a target="_blank" href="https://www.odoo.com">Odoo</a>.</p>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </template>
    </data>
</odoo>

```

## File: models\account_journal_dashboard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.addons.account.models.account_journal_dashboard import group_by_journal


class AccountJournal(models.Model):
    _inherit = "account.journal"

    def _prepare_expense_sheet_data_domain(self):
        return [
            ('journal_id', 'in', self.ids),
            '|',
            ('state', '=', 'post'),
            '&',
            ('state', '=', 'done'),
            ('payment_state', '=', 'partial'),
        ]

    def _get_expense_to_pay_query(self):
        return self.env['hr.expense.sheet']._where_calc(self._prepare_expense_sheet_data_domain())

    def _fill_sale_purchase_dashboard_data(self, dashboard_data):
        super(AccountJournal, self)._fill_sale_purchase_dashboard_data(dashboard_data)
        sale_purchase_journals = self.filtered(lambda journal: journal.type in ('sale', 'purchase'))
        if not sale_purchase_journals:
            return
        field_list = [
            "hr_expense_sheet.journal_id",
            # todo master: "hr_expense_sheet.amount_residual AS amount_total_company",
            "hr_expense_sheet.total_amount AS amount_total_company",
            "hr_expense_sheet.currency_id AS currency",
        ]
        query, params = sale_purchase_journals._get_expense_to_pay_query().select(*field_list)
        self.env.cr.execute(query, params)
        query_results_to_pay = group_by_journal(self.env.cr.dictfetchall())
        for journal in sale_purchase_journals:
            currency = journal.currency_id or journal.company_id.currency_id
            (number_expenses_to_pay, sum_expenses_to_pay) = self._count_results_and_sum_amounts(query_results_to_pay[journal.id], currency)
            dashboard_data[journal.id].update({
                'number_expenses_to_pay': number_expenses_to_pay,
                'sum_expenses_to_pay': currency.format(sum_expenses_to_pay),
            })

    def open_expenses_action(self):
        action = self.env['ir.actions.act_window']._for_xml_id('hr_expense.action_hr_expense_sheet_all_all')
        action['context'] = {
            'search_default_approved': 1,
            'search_default_to_post': 1,
            'search_default_journal_id': self.id,
            'default_journal_id': self.id,
        }
        action['view_mode'] = 'tree,form'
        action['views'] = [(k,v) for k,v in action['views'] if v in ['tree', 'form']]
        action['domain'] = self._prepare_expense_sheet_data_domain()
        return action

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, _
from odoo.api import ondelete
from odoo.exceptions import UserError
from odoo.tools.misc import frozendict


class AccountMove(models.Model):
    _inherit = "account.move"

    expense_sheet_id = fields.Many2one(comodel_name='hr.expense.sheet', ondelete='set null', copy=False, index='btree_not_null')
    show_commercial_partner_warning = fields.Boolean(compute='_compute_show_commercial_partner_warning')

    @api.depends('partner_id', 'expense_sheet_id', 'company_id')
    def _compute_commercial_partner_id(self):
        own_expense_moves = self.filtered(lambda move: move.sudo().expense_sheet_id.payment_mode == 'own_account')
        for move in own_expense_moves:
            if move.expense_sheet_id.payment_mode == 'own_account':
                move.commercial_partner_id = (
                    move.partner_id.commercial_partner_id
                    if move.partner_id.commercial_partner_id != move.company_id.partner_id
                    else move.partner_id
                )
        super(AccountMove, self - own_expense_moves)._compute_commercial_partner_id()

    def action_open_expense_report(self):
        self.ensure_one()
        return {
            'name': self.expense_sheet_id.name,
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'views': [(False, 'form')],
            'res_model': 'hr.expense.sheet',
            'res_id': self.expense_sheet_id.id
        }

    @api.depends('commercial_partner_id')
    def _compute_show_commercial_partner_warning(self):
        for move in self:
            move.show_commercial_partner_warning = (
                    move.commercial_partner_id == self.env.company.partner_id
                    and move.move_type == 'in_invoice'
                    and move.partner_id.sudo().employee_ids
            )

    # Expenses can be written on journal other than purchase, hence don't include them in the constraint check
    def _check_journal_move_type(self):
        return super(AccountMove, self.filtered(lambda x: not x.expense_sheet_id))._check_journal_move_type()

    def _creation_message(self):
        if self.expense_sheet_id:
            return _("Expense entry created from: %s", self.expense_sheet_id._get_html_link())
        return super()._creation_message()

    @api.depends('expense_sheet_id')
    def _compute_needed_terms(self):
        # EXTENDS account
        # We want to set the account destination based on the 'payment_mode'.
        super()._compute_needed_terms()
        for move in self:
            if move.expense_sheet_id and move.expense_sheet_id.payment_mode == 'company_account':
                term_lines = move.line_ids.filtered(lambda l: l.display_type != 'payment_term')
                move.needed_terms = {
                    frozendict(
                        {
                            "move_id": move.id,
                            "date_maturity": move.expense_sheet_id.accounting_date or fields.Date.context_today(move.expense_sheet_id),
                        }
                    ): {
                        "balance": -sum(term_lines.mapped("balance")),
                        "amount_currency": -sum(term_lines.mapped("amount_currency")),
                        "name": "",
                        "account_id": move.expense_sheet_id._get_expense_account_destination(),
                    }
                }

    def _reverse_moves(self, default_values_list=None, cancel=False):
        own_expense_moves = self.filtered(lambda move: move.expense_sheet_id.payment_mode == 'own_account')
        own_expense_moves.write({'expense_sheet_id': False, 'ref': False})
        # else, when restarting the expense flow we get duplicate issue on vendor.bill
        return super()._reverse_moves(default_values_list=default_values_list, cancel=cancel)

    @ondelete(at_uninstall=True)
    def _must_delete_all_expense_entries(self):
        if self.expense_sheet_id and self.expense_sheet_id.account_move_ids - self:  # If not all the payments are to be deleted
            raise UserError(_("You cannot delete only some entries linked to an expense report. All entries must be deleted at the same time."))

```

## File: models\account_move_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.tools.misc import frozendict


class AccountMoveLine(models.Model):
    _inherit = "account.move.line"

    expense_id = fields.Many2one('hr.expense', string='Expense', copy=True) # copy=True, else we don't know price is tax incl.

    @api.constrains('account_id', 'display_type')
    def _check_payable_receivable(self):
        super(AccountMoveLine, self.filtered(lambda line: line.move_id.expense_sheet_id.payment_mode != 'company_account'))._check_payable_receivable()

    def _get_attachment_domains(self):
        attachment_domains = super(AccountMoveLine, self)._get_attachment_domains()
        if self.expense_id:
            attachment_domains.append([('res_model', '=', 'hr.expense'), ('res_id', '=', self.expense_id.id)])
        return attachment_domains

    def _compute_tax_key(self):
        super()._compute_tax_key()
        for line in self:
            if line.expense_id:
                line.tax_key = frozendict(**line.tax_key, expense_id=line.expense_id.id)

    def _compute_all_tax(self):
        expense_lines = self.filtered('expense_id')
        super(AccountMoveLine, expense_lines.with_context(force_price_include=True))._compute_all_tax()
        super(AccountMoveLine, self - expense_lines)._compute_all_tax()
        for line in expense_lines:
            for key in list(line.compute_all_tax.keys()):
                new_key = frozendict(**key, expense_id=line.expense_id.id)
                line.compute_all_tax[new_key] = line.compute_all_tax.pop(key)

    def _compute_totals(self):
        expenses = self.filtered('expense_id')
        super(AccountMoveLine, expenses.with_context(force_price_include=True))._compute_totals()
        super(AccountMoveLine, self - expenses)._compute_totals()

    def _convert_to_tax_base_line_dict(self):
        result = super()._convert_to_tax_base_line_dict()
        if self.expense_id:
            result.setdefault('extra_context', {})
            result['extra_context']['force_price_include'] = True
        return result

    def _get_extra_query_base_tax_line_mapping(self):
        return ' AND (base_line.expense_id IS NULL OR account_move_line.expense_id = base_line.expense_id)'

```

## File: models\account_payment.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _
from odoo.api import ondelete
from odoo.exceptions import UserError


class AccountPayment(models.Model):
    _inherit = "account.payment"

    def action_open_expense_report(self):
        self.ensure_one()
        return {
            'name': self.expense_sheet_id.name,
            'type': 'ir.actions.act_window',
            'view_type': 'form',
            'view_mode': 'form',
            'views': [(False, 'form')],
            'res_model': 'hr.expense.sheet',
            'res_id': self.expense_sheet_id.id
        }

    def _synchronize_from_moves(self, changed_fields):
        # EXTENDS account
        if self.expense_sheet_id:
            # Constraints bypass when entry is linked to an expense.
            # Context is not enough, as we want to be able to delete
            # and update those entries later on.
            return
        super()._synchronize_from_moves(changed_fields)

    def _synchronize_to_moves(self, changed_fields):
        # EXTENDS account
        trigger_fields = set(self._get_trigger_fields_to_synchronize()) | {'ref', 'expense_sheet_id', 'payment_method_line_id'}
        if self.expense_sheet_id and any(field_name in trigger_fields for field_name in changed_fields):
            raise UserError(_("You cannot do this modification since the payment is linked to an expense report."))
        return super()._synchronize_to_moves(changed_fields)

    def _creation_message(self):
        # EXTENDS mail
        self.ensure_one()
        if self.move_id.expense_sheet_id:
            return _("Payment created for: %s", self.move_id.expense_sheet_id._get_html_link())
        return super()._creation_message()

    @ondelete(at_uninstall=True)
    def _must_delete_all_expense_payments(self):
        if self.expense_sheet_id and self.expense_sheet_id.account_move_ids.payment_ids - self:  # If not all the payments are to be deleted
            raise UserError(_("You cannot delete only some payments linked to an expense report. All payments must be deleted at the same time."))

```

## File: models\account_tax.py

```python
# -*- coding: utf-8 -*-

from odoo import models


class AccountTax(models.Model):
    _inherit = "account.tax"

    def _hook_compute_is_used(self, taxes_to_compute):
        # OVERRIDE in order to fetch taxes used in expenses

        used_taxes = super()._hook_compute_is_used(taxes_to_compute)
        taxes_to_compute -= used_taxes

        if taxes_to_compute:
            self.env['hr.expense'].flush_model(['tax_ids'])
            self.env.cr.execute("""
                SELECT id
                FROM account_tax
                WHERE EXISTS(
                    SELECT 1
                    FROM expense_tax AS exp
                    WHERE tax_id IN %s
                    AND account_tax.id = exp.tax_id
                )
            """, [tuple(taxes_to_compute)])

            used_taxes.update([tax[0] for tax in self.env.cr.fetchall()])

        return used_taxes

```

## File: models\analytic.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools import SQL
from odoo.exceptions import UserError


class AccountAnalyticApplicability(models.Model):
    _inherit = 'account.analytic.applicability'
    _description = "Analytic Plan's Applicabilities"

    business_domain = fields.Selection(
        selection_add=[
            ('expense', 'Expense'),
        ],
        ondelete={'expense': 'cascade'},
    )

    @api.depends('business_domain')
    def _compute_display_account_prefix(self):
        super()._compute_display_account_prefix()
        for applicability in self.filtered(lambda rec: rec.business_domain == 'expense'):
            applicability.display_account_prefix = True


class AccountAnalyticAccount(models.Model):
    _inherit = 'account.analytic.account'

    @api.ondelete(at_uninstall=False)
    def _unlink_except_account_in_analytic_distribution(self):
        self.env.cr.execute(
            SQL(
                r"""
                SELECT id FROM hr_expense
                    WHERE %s && %s
                LIMIT 1
                """,
                [str(id) for id in self.ids],
                self.env['hr.expense']._query_analytic_accounts(),
            )
        )
        expense_ids = self.env.cr.fetchall()
        if expense_ids:
            raise UserError(_("You cannot delete an analytic account that is used in an expense."))

```

## File: models\hr_department.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class HrDepartment(models.Model):
    _inherit = 'hr.department'

    def _compute_expense_sheets_to_approve(self):
        expense_sheet_data = self.env['hr.expense.sheet']._read_group([('department_id', 'in', self.ids), ('state', '=', 'submit')], ['department_id'], ['__count'])
        result = {department.id: count for department, count in expense_sheet_data}
        for department in self:
            department.expense_sheets_to_approve_count = result.get(department.id, 0)

    expense_sheets_to_approve_count = fields.Integer(compute='_compute_expense_sheets_to_approve', string='Expenses Reports to Approve')

```

## File: models\hr_employee.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class EmployeeBase(models.AbstractModel):
    _inherit = 'hr.employee.base'

    filter_for_expense = fields.Boolean(store=False, search='_search_filter_for_expense')

    def _search_filter_for_expense(self, operator, value):
        assert operator == '=' and value, "Operation not supported"

        res = [('id', '=', 0)]  # Nothing accepted by domain, by default
        user = self.env.user
        employee = user.employee_id
        if self.user_has_groups('hr_expense.group_hr_expense_user') or self.user_has_groups('account.group_account_user'):
            res = ['|', ('company_id', '=', False), ('company_id', 'child_of', self.env.company.root_id.id)]  # Then, domain accepts everything
        elif self.user_has_groups('hr_expense.group_hr_expense_team_approver') and user.employee_ids:
            res = [
                '|', '|', '|',
                ('department_id.manager_id', '=', employee.id),
                ('parent_id', '=', employee.id),
                ('id', '=', employee.id),
                ('expense_manager_id', '=', user.id),
                '|', ('company_id', '=', False), ('company_id', '=', employee.company_id.id),
            ]
        elif user.employee_id:
            res = [('id', '=', employee.id), '|', ('company_id', '=', False), ('company_id', '=', employee.company_id.id)]
        return res


class Employee(models.Model):
    _inherit = 'hr.employee'

    def _group_hr_expense_user_domain(self):
        # We return the domain only if the group exists for the following reason:
        # When a group is created (at module installation), the `res.users` form view is
        # automatically modified to add application accesses. When modifying the view, it
        # reads the related field `expense_manager_id` of `res.users` and retrieve its domain.
        # This is a problem because the `group_hr_expense_user` record has already been created but
        # not its associated `ir.model.data` which makes `self.env.ref(...)` fail.
        group = self.env.ref('hr_expense.group_hr_expense_team_approver', raise_if_not_found=False)
        return [('groups_id', 'in', group.ids)] if group else []

    expense_manager_id = fields.Many2one(
        comodel_name='res.users',
        string='Expense',
        compute='_compute_expense_manager', store=True, readonly=False,
        domain=_group_hr_expense_user_domain,
        help='Select the user responsible for approving "Expenses" of this employee.\n'
             'If empty, the approval is done by an Administrator or Approver (determined in settings/users).',
    )

    @api.depends('parent_id')
    def _compute_expense_manager(self):
        for employee in self:
            previous_manager = employee._origin.parent_id.user_id
            manager = employee.parent_id.user_id
            if manager and manager.has_group('hr_expense.group_hr_expense_user') \
                    and (employee.expense_manager_id == previous_manager or not employee.expense_manager_id):
                employee.expense_manager_id = manager
            elif not employee.expense_manager_id:
                employee.expense_manager_id = False

    def _get_user_m2o_to_empty_on_archived_employees(self):
        return super()._get_user_m2o_to_empty_on_archived_employees() + ['expense_manager_id']


class EmployeePublic(models.Model):
    _inherit = 'hr.employee.public'

    expense_manager_id = fields.Many2one('res.users', readonly=True)


class User(models.Model):
    _inherit = ['res.users']

    expense_manager_id = fields.Many2one(related='employee_id.expense_manager_id', readonly=False)

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + ['expense_manager_id']

```

## File: models\hr_expense.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from markupsafe import Markup
import werkzeug

from odoo import api, fields, Command, models, _
from odoo.exceptions import UserError, ValidationError
from odoo.tools.misc import format_date
from odoo.tools import email_split, float_repr, float_round, is_html_empty


class HrExpense(models.Model):
    _name = "hr.expense"
    _inherit = ['mail.thread.main.attachment', 'mail.activity.mixin', 'analytic.mixin']
    _description = "Expense"
    _order = "date desc, id desc"
    _check_company_auto = True

    @api.model
    def _default_employee_id(self):
        employee = self.env.user.employee_id
        if not employee and not self.env.user.has_group('hr_expense.group_hr_expense_team_approver'):
            raise ValidationError(_('The current user has no related employee. Please, create one.'))
        return employee

    name = fields.Char(
        string="Description",
        compute='_compute_name', precompute=True, store=True, readonly=False,
        required=True,
        copy=True,
    )
    date = fields.Date(string="Expense Date", default=fields.Date.context_today)
    employee_id = fields.Many2one(
        comodel_name='hr.employee',
        string="Employee",
        compute='_compute_employee_id', precompute=True, store=True, readonly=False,
        required=True,
        default=_default_employee_id,
        check_company=True,
        domain=[('filter_for_expense', '=', True)],
        tracking=True,
    )
    company_id = fields.Many2one(
        comodel_name='res.company',
        string="Company",
        required=True,
        readonly=True,
        default=lambda self: self.env.company,
    )
    # product_id is not required to allow to create an expense without product via mail alias, but should be required on the view.
    product_id = fields.Many2one(
        comodel_name='product.product',
        string="Category",
        tracking=True,
        check_company=True,
        domain=[('can_be_expensed', '=', True)],
        ondelete='restrict',
    )
    product_description = fields.Html(compute='_compute_product_description')
    product_uom_id = fields.Many2one(
        comodel_name='uom.uom',
        string="Unit of Measure",
        compute='_compute_uom_id', precompute=True, store=True,
        domain="[('category_id', '=', product_uom_category_id)]",
        copy=True,
    )
    product_uom_category_id = fields.Many2one(
        comodel_name='uom.category',
        string="UoM Category",
        related='product_id.uom_id.category_id',
        readonly=True,
    )
    product_has_cost = fields.Boolean(compute='_compute_from_product')  # Whether the product has a cost (standard_price) or not
    product_has_tax = fields.Boolean(string="Whether tax is defined on a selected product", compute='_compute_from_product')
    quantity = fields.Float(required=True, digits='Product Unit of Measure', default=1)
    description = fields.Text(string="Internal Notes")
    message_main_attachment_checksum = fields.Char(related='message_main_attachment_id.checksum')
    nb_attachment = fields.Integer(string="Number of Attachments", compute='_compute_nb_attachment')
    attachment_ids = fields.One2many(
        comodel_name='ir.attachment',
        inverse_name='res_id',
        domain="[('res_model', '=', 'hr.expense')]",
        string="Attachments",
    )
    state = fields.Selection(
        selection=[
            ('draft', 'To Report'),
            ('reported', 'To Submit'),
            ('submitted', 'Submitted'),
            ('approved', 'Approved'),
            ('done', 'Done'),
            ('refused', 'Refused')
        ],
        string="Status",
        compute='_compute_state', store=True, readonly=True,
        index=True,
        copy=False,
        default='draft',
    )
    sheet_id = fields.Many2one(
        comodel_name='hr.expense.sheet',
        string="Expense Report",
        domain="[('employee_id', '=', employee_id), ('company_id', '=', company_id)]",
        readonly=True,
        copy=False,
    )
    approved_by = fields.Many2one(comodel_name='res.users', string="Approved By", related='sheet_id.user_id', tracking=False)
    approved_on = fields.Datetime(string="Approved On", related='sheet_id.approval_date')
    duplicate_expense_ids = fields.Many2many(comodel_name='hr.expense', compute='_compute_duplicate_expense_ids')  # Used to trigger warnings

    # Amount fields
    tax_amount_currency = fields.Monetary(
        string="Tax amount in Currency",
        currency_field='currency_id',
        compute='_compute_tax_amount_currency', precompute=True, store=True,
        help="Tax amount in currency",
    )
    tax_amount = fields.Monetary(
        string="Tax amount",
        currency_field='company_currency_id',
        compute='_compute_tax_amount', precompute=True, store=True,
        help="Tax amount in company currency",
    )
    total_amount_currency = fields.Monetary(
        string="Total In Currency",
        currency_field='currency_id',
        compute='_compute_total_amount_currency', precompute=True, store=True, readonly=False,
        tracking=True,
    )
    untaxed_amount_currency = fields.Monetary(
        string="Total Untaxed Amount In Currency",
        currency_field='currency_id',
        compute='_compute_tax_amount_currency', precompute=True, store=True,
    )
    total_amount = fields.Monetary(
        string="Total",
        currency_field='company_currency_id',
        compute='_compute_total_amount', inverse='_inverse_total_amount', precompute=True, store=True, readonly=False,
        tracking=True,
    )
    price_unit = fields.Float(
        string="Unit Price",
        compute='_compute_price_unit', precompute=True, store=True, required=True, readonly=True,
        copy=True,
        digits='Product Price',
    )
    currency_id = fields.Many2one(
        comodel_name='res.currency',
        string="Currency",
        compute='_compute_currency_id', precompute=True, store=True, readonly=False,
        required=True,
        default=lambda self: self.env.company.currency_id,
    )
    company_currency_id = fields.Many2one(
        comodel_name='res.currency',
        related='company_id.currency_id',
        string="Report Company Currency",
        readonly=True,
    )
    is_multiple_currency = fields.Boolean(
        string="Is currency_id different from the company_currency_id",
        compute='_compute_is_multiple_currency',
    )
    currency_rate = fields.Float(compute='_compute_currency_rate', digits=(16, 9), readonly=True, tracking=True)
    label_currency_rate = fields.Char(compute='_compute_currency_rate', readonly=True)

    # Account fields
    payment_mode = fields.Selection(
        selection=[
            ('own_account', "Employee (to reimburse)"),
            ('company_account', "Company")
        ],
        string="Paid By",
        default='own_account',
        tracking=True,
    )
    account_id = fields.Many2one(
        comodel_name='account.account',
        string="Account",
        compute='_compute_account_id', precompute=True, store=True, readonly=False,
        check_company=True,
        domain="[('account_type', 'not in', ('asset_receivable', 'liability_payable', 'asset_cash', 'liability_credit_card'))]",
        help="An expense account is expected",
    )
    tax_ids = fields.Many2many(
        comodel_name='account.tax',
        relation='expense_tax',
        column1='expense_id',
        column2='tax_id',
        string="Included taxes",
        compute='_compute_tax_ids', precompute=True, store=True, readonly=False,
        domain="[('type_tax_use', '=', 'purchase')]",
        check_company=True,
        help="Both price-included and price-excluded taxes will behave as price-included taxes for expenses.",
    )
    accounting_date = fields.Date(  # The date used for the accounting entries or the one we'd like to use if not yet posted
        related='sheet_id.accounting_date',
        string="Accounting Date",
        store=True,
        groups='account.group_account_invoice,account.group_account_readonly',
    )

    # Security fields
    is_editable = fields.Boolean(string="Is Editable By Current User", compute='_compute_is_editable')

    @api.depends('product_has_cost')
    def _compute_currency_id(self):
        for expense in self:
            if expense.product_has_cost and expense.state in {'draft', 'reported'}:
                expense.currency_id = expense.company_currency_id

    @api.depends('sheet_id.is_editable')
    def _compute_is_editable(self):
        for expense in self:
            if expense.sheet_id:
                expense.is_editable = expense.sheet_id.is_editable
            else:
                expense.is_editable = True

    @api.onchange('product_has_cost')
    def _onchange_product_has_cost(self):
        """ Reset quantity to 1, in case of 0-cost product. To make sure switching non-0-cost to 0-cost doesn't keep the quantity."""
        if not self.product_has_cost and self.state in {'draft', 'reported'}:
            self.quantity = 1

    @api.depends_context('lang')
    @api.depends('product_id')
    def _compute_product_description(self):
        for expense in self:
            expense.product_description = not is_html_empty(expense.product_id.description) and expense.product_id.description

    @api.depends('product_id')
    def _compute_name(self):
        for expense in self:
            expense.name = expense.name or expense.product_id.display_name

    @api.depends('currency_id', 'total_amount_currency', 'date')
    def _compute_currency_rate(self):
        """
            We want the default odoo rate when the following change:
            - the currency of the expense
            - the total amount in foreign currency
            - the date of the expense
            this will cause the rate to be recomputed twice with possible changes but we don't have the required fields
            to store the override state in stable
        """
        date_today = fields.Date.context_today(self)
        for expense in self:
            if expense.is_multiple_currency:
                if (
                        expense.currency_id != expense._origin.currency_id
                        or expense.total_amount_currency != expense._origin.total_amount_currency
                        or expense.date != expense._origin.date
                ):
                    expense.currency_rate = self.env['res.currency']._get_conversion_rate(
                        from_currency=expense.currency_id,
                        to_currency=expense.company_currency_id,
                        company=expense.company_id,
                        date=expense.date or date_today,
                    )
                else:
                    expense.currency_rate = expense.total_amount / expense.total_amount_currency if expense.total_amount_currency else 1.0
            else:  # Mono-currency case computation shortcut, no need for the label if there is no conversion
                expense.currency_rate = 1.0
                expense.label_currency_rate = False
                continue

            expense.label_currency_rate = _(
                '1 %(exp_cur)s = %(rate)s %(comp_cur)s',
                exp_cur=expense.currency_id.name,
                rate=float_repr(expense.currency_rate, 6),
                comp_cur=expense.company_currency_id.name,
            )

    @api.depends('currency_id', 'company_currency_id')
    def _compute_is_multiple_currency(self):
        for expense in self:
            expense.is_multiple_currency = expense.currency_id != expense.company_currency_id

    @api.depends('product_id.standard_price')
    def _compute_from_product(self):
        for expense in self:
            expense.product_has_cost = expense.product_id and not expense.company_currency_id.is_zero(expense.product_id.standard_price)
            tax_ids = expense.product_id.supplier_taxes_id.filtered_domain(self.env['account.tax']._check_company_domain(expense.company_id))
            expense.product_has_tax = bool(tax_ids)
            if not expense.product_has_cost and expense.state in {'draft', 'reported'} and expense.quantity != 1:
                expense.quantity = 1

    @api.depends('product_id.uom_id')
    def _compute_uom_id(self):
        for expense in self:
            expense.product_uom_id = expense.product_id.uom_id

    @api.depends('sheet_id', 'sheet_id.account_move_ids', 'sheet_id.state')
    def _compute_state(self):
        for expense in self:
            if not expense.sheet_id:
                expense.state = 'draft'
            elif expense.sheet_id.state == 'draft':
                expense.state = 'reported'
            elif expense.sheet_id.state == 'cancel':
                expense.state = 'refused'
            elif expense.sheet_id.state in {'approve', 'post'}:
                expense.state = 'approved'
            elif not expense.sheet_id.account_move_ids:
                expense.state = 'submitted'
            else:
                expense.state = 'done'

    @api.depends('quantity', 'price_unit', 'tax_ids')
    def _compute_total_amount_currency(self):
        for expense in self.filtered('product_has_cost'):
            base_lines = [expense._convert_to_tax_base_line_dict(price_unit=expense.price_unit, quantity=expense.quantity)]
            taxes_totals = self.env['account.tax']._compute_taxes(base_lines)['totals'][expense.currency_id]
            expense.total_amount_currency = taxes_totals['amount_untaxed'] + taxes_totals['amount_tax']

    @api.onchange('total_amount_currency')
    def _inverse_total_amount_currency(self):
        for expense in self:
            if not expense.is_editable:
                raise UserError(_('You are not authorized to edit this expense.'))
            expense.price_unit = (expense.total_amount / expense.quantity) if expense.quantity != 0 else 0.

    @api.depends(
        'date',
        'company_id',
        'currency_id',
        'company_currency_id',
        'is_multiple_currency',
        'total_amount_currency',
        'product_id',
        'employee_id.user_id.partner_id',
        'quantity',
    )
    def _compute_total_amount(self):
        for expense in self:
            if expense.is_multiple_currency:
                base_lines = [expense._convert_to_tax_base_line_dict(
                    price_unit=expense.total_amount_currency * expense.currency_rate,
                    currency=expense.company_currency_id,
                )]
                taxes_totals = self.env['account.tax']._compute_taxes(base_lines)['totals'][expense.company_currency_id]
                expense.total_amount = taxes_totals['amount_untaxed'] + taxes_totals['amount_tax']
            else:  # Mono-currency case computation shortcut
                expense.total_amount = expense.total_amount_currency

    def _inverse_total_amount(self):
        """ Allows to set a custom rate on the expense, and avoid the override when it makes no sense """
        for expense in self:
            if expense.is_multiple_currency:
                base_lines = [expense._convert_to_tax_base_line_dict(
                    price_unit=expense.total_amount,
                    currency=expense.company_currency_id,
                )]
                taxes_totals = self.env['account.tax']._compute_taxes(base_lines)['totals'][expense.company_currency_id]
                expense.tax_amount = taxes_totals['amount_tax']
            else:
                expense.total_amount_currency = expense.total_amount
                expense.tax_amount = expense.tax_amount_currency
            expense.currency_rate = expense.total_amount / expense.total_amount_currency if expense.total_amount_currency else 1.0
            expense.price_unit = expense.total_amount / expense.quantity if expense.quantity else expense.total_amount

    @api.depends('product_id', 'company_id')
    def _compute_tax_ids(self):
        for _expense in self:
            expense = _expense.with_company(_expense.company_id)
            # taxes only from the same company
            expense.tax_ids = expense.product_id.supplier_taxes_id.filtered_domain(self.env['account.tax']._check_company_domain(expense.company_id))

    @api.depends('total_amount_currency', 'tax_ids')
    def _compute_tax_amount_currency(self):
        """
             Note: as total_amount_currency can be set directly by the user (for product without cost)
             or needs to be computed (for product with cost), `untaxed_amount_currency` can't be computed in the same method as `total_amount_currency`.
        """
        for expense in self:
            base_lines = [expense._convert_to_tax_base_line_dict(price_unit=expense.total_amount_currency)]
            taxes_totals = self.env['account.tax']._compute_taxes(base_lines)['totals'][expense.currency_id]
            expense.tax_amount_currency = taxes_totals['amount_tax']
            expense.untaxed_amount_currency = taxes_totals['amount_untaxed']

    @api.depends('total_amount', 'currency_rate', 'tax_ids', 'is_multiple_currency')
    def _compute_tax_amount(self):
        """
             Note: as total_amount can be set directly by the user when the currency_rate is overriden,
             the tax must be computed after the total_amount.
        """
        for expense in self:
            if expense.is_multiple_currency:
                base_lines = [expense._convert_to_tax_base_line_dict(
                    price_unit=expense.total_amount,
                    currency=expense.company_currency_id,
                )]
                taxes_totals = self.env['account.tax']._compute_taxes(base_lines)['totals'][expense.company_currency_id]
                expense.tax_amount = taxes_totals['amount_tax']
            else:  # Mono-currency case computation shortcut
                expense.tax_amount = expense.tax_amount_currency

    @api.depends('total_amount', 'total_amount_currency')
    def _compute_price_unit(self):
        """
           The price_unit is the unit price of the product if no product is set and no attachment overrides it.
           Otherwise it is always computed from the total_amount and the quantity else it would break the vendor bill
           when edited after creation.
        """
        for expense in self:
            if expense.state not in {'draft', 'reported'}:
                continue
            product_id = expense.product_id
            if expense._needs_product_price_computation():
                expense.price_unit = product_id._price_compute(
                    'standard_price',
                    uom=expense.product_uom_id,
                    company=expense.company_id,
                )[product_id.id]
            else:
                expense.price_unit = expense.company_currency_id.round(expense.total_amount / expense.quantity) if expense.quantity else 0.

    def _needs_product_price_computation(self):
        # Hook to be overridden.
        self.ensure_one()
        return self.product_has_cost

    @api.depends('product_id', 'company_id')
    def _compute_account_id(self):
        for _expense in self:
            expense = _expense.with_company(_expense.company_id)
            if not expense.product_id:
                expense.account_id = self.env['ir.property']._get('property_account_expense_categ_id', 'product.category')
                continue
            account = expense.product_id.product_tmpl_id._get_product_accounts()['expense']
            if account:
                expense.account_id = account

    @api.depends('company_id')
    def _compute_employee_id(self):
        if not self.env.context.get('default_employee_id'):
            for expense in self:
                expense.employee_id = self.env.user.with_company(expense.company_id).employee_id

    @api.depends('employee_id', 'product_id', 'total_amount_currency')
    def _compute_duplicate_expense_ids(self):
        self.duplicate_expense_ids = [Command.clear()]

        expenses = self.filtered(lambda expense: expense.employee_id and expense.product_id and expense.total_amount_currency)
        if expenses.ids:
            duplicates_query = """
              SELECT ARRAY_AGG(DISTINCT he.id)
                FROM hr_expense AS he
                JOIN hr_expense AS ex ON he.employee_id = ex.employee_id
                                     AND he.product_id = ex.product_id
                                     AND he.date = ex.date
                                     AND he.total_amount_currency = ex.total_amount_currency
                                     AND he.company_id = ex.company_id
                                     AND he.currency_id = ex.currency_id
               WHERE ex.id in %(expense_ids)s
               GROUP BY he.employee_id, he.product_id, he.date, he.total_amount_currency, he.company_id, he.currency_id
              HAVING COUNT(he.id) > 1
            """
            self.env.cr.execute(duplicates_query, {'expense_ids': tuple(expenses.ids)})

            for duplicates_ids in (x[0] for x in self.env.cr.fetchall()):
                expenses_duplicates = expenses.filtered(lambda expense: expense.id in duplicates_ids)
                expenses_duplicates.duplicate_expense_ids = [Command.set(duplicates_ids)]
                expenses = expenses - expenses_duplicates

    @api.depends('product_id', 'account_id')
    def _compute_analytic_distribution(self):
        for expense in self:
            distribution = self.env['account.analytic.distribution.model']._get_distribution({
                'product_id': expense.product_id.id,
                'product_categ_id': expense.product_id.categ_id.id,
                'account_prefix': expense.account_id.code,
                'company_id': expense.company_id.id,
            })
            expense.analytic_distribution = distribution or expense.analytic_distribution

    def _compute_nb_attachment(self):
        attachment_data = self.env['ir.attachment']._read_group(
            [('res_model', '=', 'hr.expense'), ('res_id', 'in', self.ids)],
            ['res_id'],
            ['__count'],
        )
        attachment = dict(attachment_data)
        for expense in self:
            expense.nb_attachment = attachment.get(expense._origin.id, 0)

    @api.constrains('payment_mode')
    def _check_payment_mode(self):
        self.sheet_id._check_payment_mode()

    def _convert_to_tax_base_line_dict(self, base_line=None, currency=None, price_unit=None, quantity=None):
        self.ensure_one()
        return self.env['account.tax']._convert_to_tax_base_line_dict(
            base_line,
            currency=currency or self.currency_id,
            product=self.product_id,
            taxes=self.tax_ids,
            price_unit=price_unit or self.total_amount,
            quantity=quantity if quantity is not None else 1,
            account=self.account_id,
            analytic_distribution=self.analytic_distribution,
            extra_context={'force_price_include': True},
        )

    def attach_document(self, **kwargs):
        """When an attachment is uploaded as a receipt, set it as the main attachment."""
        self.message_main_attachment_id = kwargs['attachment_ids'][-1]

    def create_expense_from_attachments(self, attachment_ids=None, view_type='list'):
        """
            Create the expenses from files.

            :return: An action redirecting to hr.expense tree view.
        """
        if not attachment_ids:
            raise UserError(_("No attachment was provided"))
        attachments = self.env['ir.attachment'].browse(attachment_ids)
        expenses = self.env['hr.expense']

        if any(attachment.res_id or attachment.res_model != 'hr.expense' for attachment in attachments):
            raise UserError(_("Invalid attachments!"))

        product = self.env['product.product'].search([('can_be_expensed', '=', True)])
        if product:
            product = product.filtered(lambda p: p.default_code == "EXP_GEN")[:1] or product[0]
        else:
            raise UserError(_("You need to have at least one category that can be expensed in your database to proceed!"))

        for attachment in attachments:
            attachment_name = '.'.join(attachment.name.split('.')[:-1])
            vals = {
                'name': attachment_name,
                'price_unit': 0,
                'product_id': self.env.company.expense_product_id.id or product.id,
            }
            if product.property_account_expense_id:
                vals['account_id'] = product.property_account_expense_id.id
            expense = self.env['hr.expense'].create(vals)
            attachment.write({'res_model': 'hr.expense', 'res_id': expense.id})

            attachment.register_as_main_attachment()
            expenses += expense
        return {
            'name': _('Generate Expenses'),
            'res_model': 'hr.expense',
            'type': 'ir.actions.act_window',
            'views': [[False, view_type], [False, "form"]],
            'context': {'search_default_my_expenses': 1, 'search_default_no_report': 1},
        }

    # ----------------------------------------
    # ORM Overrides
    # ----------------------------------------

    @api.ondelete(at_uninstall=False)
    def _unlink_except_posted_or_approved(self):
        for expense in self:
            if expense.state in {'done', 'approved'}:
                raise UserError(_('You cannot delete a posted or approved expense.'))

    def write(self, vals):
        if (
                'state' in vals
                and vals['state'] not in ('draft', 'submitted')
                and not self.user_has_groups('hr_expense.group_hr_expense_manager')
                and any(state == 'draft' for state in self.mapped('state'))
        ):
            raise UserError(_("You don't have the rights to bypass the validation process of this expense."))
        expense_to_previous_sheet = {}
        if 'sheet_id' in vals:
            self.env['hr.expense.sheet'].browse(vals['sheet_id']).check_access_rule('write')
            for expense in self:
                expense_to_previous_sheet[expense] = expense.sheet_id
        if 'tax_ids' in vals or 'analytic_distribution' in vals or 'account_id' in vals:
            if any(not expense.is_editable for expense in self):
                raise UserError(_('You are not authorized to edit this expense report.'))
        res = super().write(vals)

        if 'employee_id' in vals:
            # In case expense has sheet which has only one expense_line_ids,
            # then changing the expense.employee_id triggers changing the sheet.employee_id too.
            # Otherwise we unlink the expense line from sheet, (so that the user can create a new report).
            if self.sheet_id:
                employees = self.sheet_id.expense_line_ids.mapped('employee_id')
                if len(employees) == 1:
                    self.sheet_id.write({'employee_id': vals['employee_id']})
                elif len(employees) > 1:
                    self.sheet_id = False
        if 'sheet_id' in vals:
            # The sheet_id has been modified, either by an explicit write on sheet_id of the expense,
            # or by processing a command on the sheet's expense_line_ids.
            # We need to delete the attachments on the previous sheet coming from the expenses that were modified,
            # and copy the attachments of the expenses to the new sheet,
            # if it's a no-op (writing same sheet_id as the current sheet_id of the expense),
            # nothing should be done (no unlink then copy of the same attachments)
            attachments_to_unlink = self.env['ir.attachment']
            for expense in self:
                previous_sheet = expense_to_previous_sheet[expense]
                checksums = set((expense.attachment_ids - previous_sheet.expense_line_ids.attachment_ids).mapped('checksum'))
                attachments_to_unlink += previous_sheet.attachment_ids.filtered(lambda att: att.checksum in checksums)
                if vals['sheet_id'] and expense.sheet_id != previous_sheet:
                    for attachment in expense.attachment_ids.with_context(sync_attachment=False):
                        attachment.copy({
                            'res_model': 'hr.expense.sheet',
                            'res_id': vals['sheet_id'],
                        })
            attachments_to_unlink.with_context(sync_attachment=False).unlink()
        return res

    def unlink(self):
        attachments_to_unlink = self.env['ir.attachment']
        for sheet in self.sheet_id:
            checksums = set((sheet.expense_line_ids.attachment_ids & self.attachment_ids).mapped('checksum'))
            attachments_to_unlink += sheet.attachment_ids.filtered(lambda att: att.checksum in checksums)
        attachments_to_unlink.with_context(sync_attachment=False).unlink()
        return super().unlink()

    @api.model
    def get_empty_list_help(self, help_message):
        return super().get_empty_list_help((help_message or '') + self._get_empty_list_mail_alias())

    @api.model
    def _get_empty_list_mail_alias(self):
        use_mailgateway = self.env['ir.config_parameter'].sudo().get_param('hr_expense.use_mailgateway')
        expense_alias = self.env.ref('hr_expense.mail_alias_expense', raise_if_not_found=False) if use_mailgateway else False
        if expense_alias and expense_alias.alias_domain and expense_alias.alias_name:
            # encode, but force %20 encoding for space instead of a + (URL / mailto difference)
            params = werkzeug.urls.url_encode({'subject': _("Lunch with customer $12.32")}).replace('+', '%20')
            return Markup(
                """<p>%(send_string)s <a href="mailto:%(alias_email)s?%(params)s">%(alias_email)s</a></p>"""
            ) % {
                'alias_email': expense_alias.display_name,
                'params': params,
                'send_string': _("Or send your receipts at"),
            }
        return ""

    # ----------------------------------------
    # Actions
    # ----------------------------------------

    def action_view_sheet(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'views': [[False, "form"]],
            'res_model': 'hr.expense.sheet',
            'target': 'current',
            'res_id': self.sheet_id.id
        }

    def _get_default_expense_sheet_values(self):
        # If there is an expense with total_amount == 0, it means that expense has not been processed by OCR yet
        expenses_with_amount = self.filtered(lambda expense: not (
            expense.currency_id.is_zero(expense.total_amount_currency)
            or expense.company_currency_id.is_zero(expense.total_amount)
            or (expense.product_id and not float_round(expense.quantity, precision_rounding=expense.product_uom_id.rounding))
        ))

        if any(expense.state != 'draft' or expense.sheet_id for expense in expenses_with_amount):
            raise UserError(_("You cannot report twice the same line!"))
        if not expenses_with_amount:
            raise UserError(_("You cannot report the expenses without amount!"))
        if len(expenses_with_amount.mapped('employee_id')) != 1:
            raise UserError(_("You cannot report expenses for different employees in the same report."))
        if any(not expense.product_id for expense in expenses_with_amount):
            raise UserError(_("You can not create report without category."))
        if len(self.company_id) != 1:
            raise UserError(_("You cannot report expenses for different companies in the same report."))

        # Check if two reports should be created
        own_expenses = expenses_with_amount.filtered(lambda x: x.payment_mode == 'own_account')
        company_expenses = expenses_with_amount - own_expenses
        create_two_reports = own_expenses and company_expenses

        sheets = (own_expenses, company_expenses) if create_two_reports else (expenses_with_amount,)
        values = []

        # We use a fallback name only when several expense sheets are created,
        # else we use the form view required name to force the user to set a name
        for todo in sheets:
            paid_by = 'company' if todo[0].payment_mode == 'company_account' else 'employee'
            sheet_name = _("New Expense Report, paid by %(paid_by)s", paid_by=paid_by) if len(sheets) > 1 else False
            if len(todo) == 1:
                sheet_name = todo.name
            else:
                dates = todo.mapped('date')
                if False not in dates:  # If at least one date isn't set, we don't set a default name
                    min_date = format_date(self.env, min(dates))
                    max_date = format_date(self.env, max(dates))
                    if min_date == max_date:
                        sheet_name = min_date
                    else:
                        sheet_name = _("%(date_from)s - %(date_to)s", date_from=min_date, date_to=max_date)

            values.append({
                'company_id': self.company_id.id,
                'employee_id': self[0].employee_id.id,
                'name': sheet_name,
                'expense_line_ids': [Command.set(todo.ids)],
                'state': 'draft',
            })
        return values

    def get_expenses_to_submit(self):
        # if there ere no records selected, then select all draft expenses for the user
        if self:
            expenses = self.filtered(lambda expense: expense.state == 'draft' and not expense.sheet_id and expense.is_editable)
        else:
            expenses = self.env['hr.expense'].search([
                ('state', '=', 'draft'),
                ('sheet_id', '=', False),
                ('employee_id', '=', self.env.user.employee_id.id),
            ]).filtered(lambda expense: expense.is_editable)

        if not expenses:
            raise UserError(_('You have no expense to report'))
        return expenses.action_submit_expenses()

    def action_submit_expenses(self):
        if self.filtered(lambda expense: not expense.is_editable):
            raise UserError(_('You are not authorized to edit this expense.'))
        sheets = self.env['hr.expense.sheet'].create(self._get_default_expense_sheet_values())
        return {
            'name': _('New Expense Reports'),
            'type': 'ir.actions.act_window',
            'res_model': 'hr.expense.sheet',
            'context': self.env.context,
            'views': [[False, "list"], [False, "form"]] if len(sheets) > 1 else [[False, "form"]],
            'domain': [('id', 'in', sheets.ids)],
            'res_id': sheets.id if len(sheets) == 1 else False,
        }

    def action_get_attachment_view(self):
        self.ensure_one()
        res = self.env['ir.actions.act_window']._for_xml_id('base.action_attachment')
        res.update({
            'domain': [('res_model', '=', 'hr.expense'), ('res_id', 'in', self.ids)],
            'context': {'default_res_model': 'hr.expense', 'default_res_id': self.id},
        })
        return res

    def action_approve_duplicates(self):
        root = self.env['ir.model.data']._xmlid_to_res_id("base.partner_root")
        for expense in self.duplicate_expense_ids:
            expense.message_post(
                body=_('%(user)s confirms this expense is not a duplicate with similar expense.', user=self.env.user.name),
                author_id=root,
            )

    def _get_split_values(self):
        self.ensure_one()
        half_price = self.total_amount_currency / 2
        price_round_up = float_round(half_price, precision_digits=self.currency_id.decimal_places, rounding_method='UP')
        price_round_down = float_round(half_price, precision_digits=self.currency_id.decimal_places, rounding_method='DOWN')

        return [{
            'name': self.name,
            'product_id': self.product_id.id,
            'total_amount_currency': price,
            'tax_ids': self.tax_ids.ids,
            'currency_id': self.currency_id.id,
            'company_id': self.company_id.id,
            'analytic_distribution': self.analytic_distribution,
            'employee_id': self.employee_id.id,
            'expense_id': self.id,
        } for price in (price_round_up, price_round_down)]

    def action_split_wizard(self):
        self.ensure_one()
        splits = self.env['hr.expense.split'].create(self._get_split_values())

        wizard = self.env['hr.expense.split.wizard'].create({
            'expense_split_line_ids': splits.ids,
            'expense_id': self.id,
        })
        return {
            'name': _('Expense split'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'views': [[False, "form"]],
            'res_model': 'hr.expense.split.wizard',
            'res_id': wizard.id,
            'target': 'new',
            'context': self.env.context,
        }

    # ----------------------------------------
    # Business
    # ----------------------------------------

    def _prepare_payments_vals(self):
        self.ensure_one()

        journal = self.sheet_id.journal_id
        payment_method_line = self.sheet_id.payment_method_line_id
        if not payment_method_line:
            raise UserError(_("You need to add a manual payment method on the journal (%s)", journal.name))
        move_lines = []
        tax_data = self.env['account.tax']._compute_taxes(
            [self._convert_to_tax_base_line_dict(price_unit=self.total_amount_currency, currency=self.currency_id)],
            include_caba_tags=(self.payment_mode == 'company_account')
        )
        rate = abs(self.total_amount_currency / self.total_amount) if self.total_amount else 1.0
        base_line_data, to_update = tax_data['base_lines_to_update'][0]  # Add base line
        amount_currency = to_update['price_subtotal']
        expense_name = self.name.split("\n")[0][:64]
        base_move_line = {
            'name': f'{self.employee_id.name}: {expense_name}',
            'account_id': base_line_data['account'].id,
            'product_id': base_line_data['product'].id,
            'analytic_distribution': base_line_data['analytic_distribution'],
            'expense_id': self.id,
            'tax_ids': [Command.set(self.tax_ids.ids)],
            'tax_tag_ids': to_update['tax_tag_ids'],
            'amount_currency': amount_currency,
            'currency_id': self.currency_id.id,
            'quantity': self.quantity,
        }
        move_lines.append(base_move_line)
        total_tax_line_balance = 0.0
        for tax_line_data in tax_data['tax_lines_to_add']:  # Add tax lines
            tax_line_balance = self.company_currency_id.round(tax_line_data['tax_amount'] / rate)
            total_tax_line_balance += tax_line_balance
            tax_line = {
                'name': self.env['account.tax'].browse(tax_line_data['tax_id']).name,
                'account_id': tax_line_data['account_id'],
                'analytic_distribution': tax_line_data['analytic_distribution'],
                'expense_id': self.id,
                'tax_tag_ids': tax_line_data['tax_tag_ids'],
                'balance': tax_line_balance,
                'amount_currency': tax_line_data['tax_amount'],
                'tax_base_amount': self.company_currency_id.round(tax_line_data['base_amount'] / rate),
                'currency_id': self.currency_id.id,
                'tax_repartition_line_id': tax_line_data['tax_repartition_line_id'],
            }
            move_lines.append(tax_line)
        base_move_line['balance'] = self.total_amount - total_tax_line_balance
        expense_name = self.name.split("\n")[0][:64]
        move_lines.append({  # Add outstanding payment line
            'name': f'{self.employee_id.name}: {expense_name}',
            'account_id': self.sheet_id._get_expense_account_destination(),
            'balance': -self.total_amount,
            'amount_currency': self.currency_id.round(-self.total_amount_currency),
            'currency_id': self.currency_id.id,
        })
        return {
            **self.sheet_id._prepare_move_vals(),
            'date': self.date,  # Overidden from self.sheet_id._prepare_move_vals() so we can use the expense date for the account move date
            'ref': self.name,
            'journal_id': journal.id,
            'move_type': 'entry',
            'amount': self.total_amount_currency,
            'payment_type': 'outbound',
            'partner_type': 'supplier',
            'payment_method_line_id': payment_method_line.id,
            'currency_id': self.currency_id.id,
            'line_ids': [Command.create(line) for line in move_lines],
            'attachment_ids': [
                Command.create(attachment.copy_data({'res_model': 'account.move', 'res_id': False, 'raw': attachment.raw})[0])
                for attachment in self.message_main_attachment_id]
        }

    def _prepare_move_lines_vals(self):
        self.ensure_one()
        account = self.account_id
        if not account:
            # We need to do this as the installation process may delete the original account, and it doesn't recompute properly after.
            # This forces the default values if none is found
            if self.product_id:
                account = self.product_id.product_tmpl_id._get_product_accounts()['expense']
            else:
                account = self.env['ir.property']._get('property_account_expense_categ_id', 'product.category')
        expense_name = self.name.split('\n')[0][:64]
        return {
            'name': f'{self.employee_id.name}: {expense_name}',
            'account_id': account.id,
            'quantity': self.quantity or 1,
            'price_unit': self.price_unit,
            'product_id': self.product_id.id,
            'product_uom_id': self.product_uom_id.id,
            'analytic_distribution': self.analytic_distribution,
            'expense_id': self.id,
            'partner_id': False if self.payment_mode == 'company_account' else self.employee_id.sudo().work_contact_id.id,
            'tax_ids': [Command.set(self.tax_ids.ids)],
        }

    @api.model
    def get_expense_dashboard(self):
        expense_state = {
            'to_submit': {
                'description': _('to submit'),
                'amount': 0.0,
                'tooltip': _("Expenses that need to be submitted to the approver."),
                'currency': self.env.company.currency_id.id,
            },
            'submitted': {
                'description': _('under validation'),
                'amount': 0.0,
                'tooltip': _("Expenses from which the report has been submitted to the approver and is waiting for approval."),
                'currency': self.env.company.currency_id.id,
            },
            'approved': {
                'description': _('to be reimbursed'),
                'amount': 0.0,
                'tooltip': _("Expenses paid by employee that are approved but not paid yet."),
                'currency': self.env.company.currency_id.id,
            }
        }
        if not self.env.user.employee_ids:
            return expense_state
        target_currency = self.env.company.currency_id
        # Counting the expenses to display in the dashboard:
        # - To submit: contains the expenses paid either by the employee or by the company, and that are draft or reported
        # - Under validation: contains expenses paid by the employee or paid by the company, and that have been submitted but still need to be approved/refused
        # - To be reimbursed: contains ONLY expenses paid by the employee that are approved, the payment has not yet been made
        expenses = self._read_group(
            [
                ('employee_id', 'in', self.env.user.employee_ids.ids),
                '|', '&', ('payment_mode', 'in', ('own_account', 'company_account')), ('state', 'in', ('draft', 'reported', 'submitted')),
                     '&', ('payment_mode', '=', 'own_account'), ('state', '=', 'approved')
            ], ['state'], ['total_amount:sum'])
        for state, total_amount_sum in expenses:
            if state in {'draft', 'reported'}:  # Fuse the two states into only one "To Submit" state
                state = 'to_submit'
            expense_state[state]['amount'] += total_amount_sum
        return expense_state

    # ----------------------------------------
    # Mail Thread
    # ----------------------------------------

    @api.model
    def message_new(self, msg_dict, custom_values=None):
        email_address = email_split(msg_dict.get('email_from', False))[0]
        employee = self._get_employee_from_email(email_address)

        if not employee:
            return super().message_new(msg_dict, custom_values=custom_values)

        expense_description = msg_dict.get('subject', '')

        if employee.user_id:
            company = employee.user_id.company_id
            currencies = company.currency_id | employee.user_id.company_ids.mapped('currency_id')
        else:
            company = employee.company_id
            currencies = company.currency_id

        if not company:  # ultimate fallback, since company_id is required on expense
            company = self.env.company

        # The expenses alias is the same for all companies, we need to set the proper context
        # To select the product account
        self = self.with_company(company)

        product, price, currency_id, expense_description = self._parse_expense_subject(expense_description, currencies)
        vals = {
            'employee_id': employee.id,
            'name': expense_description,
            'total_amount_currency': price,
            'product_id': product.id if product else None,
            'product_uom_id': product.uom_id.id,
            'tax_ids': [Command.set(product.supplier_taxes_id.filtered(lambda r: r.company_id == company).ids)],
            'quantity': 1,
            'company_id': company.id,
            'currency_id': currency_id.id
        }

        account = product.product_tmpl_id._get_product_accounts()['expense']
        if account:
            vals['account_id'] = account.id

        expense = super().message_new(msg_dict, dict(custom_values or {}, **vals))
        self._send_expense_success_mail(msg_dict, expense)
        return expense

    @api.model
    def _get_employee_from_email(self, email_address):
        employee = self.env['hr.employee'].search([
            ('user_id', '!=', False),
            '|',
            ('work_email', 'ilike', email_address),
            ('user_id.email', 'ilike', email_address),
        ])

        if len(employee) > 1:
            # Several employees can be linked to the same user.
            # In that case, we only keep the employee that matched the user's company.
            return employee.filtered(lambda e: e.company_id == e.user_id.company_id)

        if not employee:
            # An employee does not always have a user.
            return self.env['hr.employee'].search([
                ('user_id', '=', False),
                ('work_email', 'ilike', email_address),
            ], limit=1)

        return employee

    @api.model
    def _parse_product(self, expense_description):
        """
            Parse the subject to find the product.
            Product code should be the first word of expense_description
            Return product.product and updated description
        """
        product_code = expense_description.split(' ')[0]
        product = self.env['product.product'].search([('can_be_expensed', '=', True), ('default_code', '=ilike', product_code)], limit=1)
        if product:
            expense_description = expense_description.replace(product_code, '', 1)

        return product, expense_description

    @api.model
    def _parse_price(self, expense_description, currencies):
        """ Return price, currency and updated description """
        symbols, symbols_pattern, float_pattern = [], '', r'[+-]?(\d+[.,]?\d*)'
        price = 0.0
        for currency in currencies:
            symbols += [re.escape(currency.symbol), re.escape(currency.name)]
        symbols_pattern = '|'.join(symbols)
        price_pattern = f'(({symbols_pattern})?\\s?{float_pattern}\\s?({symbols_pattern})?)'
        matches = re.findall(price_pattern, expense_description)
        currency = currencies[:1]
        if matches:
            match = max(matches, key=lambda match: len([group for group in match if group]))
            # get the longest match. e.g. "2 chairs 120$" -> the price is 120$, not 2
            full_str = match[0]
            currency_str = match[1] or match[3]
            price = match[2].replace(',', '.')

            if currency_str and currencies:
                currencies = currencies.filtered(lambda c: currency_str in [c.symbol, c.name])
                currency = currencies[:1] or currency
            expense_description = expense_description.replace(full_str, ' ')  # remove price from description
            expense_description = re.sub(' +', ' ', expense_description.strip())

        return float(price), currency, expense_description

    @api.model
    def _parse_expense_subject(self, expense_description, currencies):
        """
            Fetch product, price and currency info from mail subject.

            Product can be identified based on product name or product code.
            It can be passed between [] or it can be placed at start.

            When parsing, only consider currencies passed as parameter.
            This will fetch currency in symbol($) or ISO name (USD).

            Some valid examples:
                Travel by Air [TICKET] USD 1205.91
                TICKET $1205.91 Travel by Air
                Extra expenses 29.10EUR [EXTRA]
        """
        product, expense_description = self._parse_product(expense_description)
        price, currency_id, expense_description = self._parse_price(expense_description, currencies)

        return product, price, currency_id, expense_description

    def _send_expense_success_mail(self, msg_dict, expense):
        if expense.employee_id.user_id:
            mail_template_id = 'hr_expense.hr_expense_template_register'
        else:
            mail_template_id = 'hr_expense.hr_expense_template_register_no_user'
        rendered_body = self.env['ir.qweb']._render(mail_template_id, {'expense': expense})
        body = self.env['mail.render.mixin']._replace_local_links(rendered_body)
        if expense.employee_id.user_id.partner_id:
            expense.message_post(
                body=body,
                email_layout_xmlid='mail.mail_notification_light',
                partner_ids=expense.employee_id.user_id.partner_id.ids,
                subject=f'Re: {msg_dict.get("subject", "")}',
                subtype_xmlid='mail.mt_note',
            )
        else:
            self.env['mail.mail'].sudo().create({
                'author_id': self.env.user.partner_id.id,
                'auto_delete': True,
                'body_html': body,
                'email_from': self.env.user.email_formatted,
                'email_to': msg_dict.get('email_from', False),
                'references': msg_dict.get('message_id'),
                'subject': f'Re: {msg_dict.get("subject", "")}',
            }).send()

```

## File: models\hr_expense_sheet.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, Command, models, _
from odoo.exceptions import UserError, ValidationError, RedirectWarning
from odoo.tools.misc import clean_context


class HrExpenseSheet(models.Model):
    """
        Here are the rights associated with the expense flow

        Action       Group                   Restriction
        =================================================================================
        Submit      Employee                Only his own
                    Officer                 If he is expense manager of the employee, manager of the employee
                                             or the employee is in the department managed by the officer
                    Manager                 Always
        Approve     Officer                 Not his own and he is expense manager of the employee, manager of the employee
                                             or the employee is in the department managed by the officer
                    Manager                 Always
        Post        Anybody                 State = approve and journal_id defined
        Done        Anybody                 State = approve and journal_id defined
        Cancel      Officer                 Not his own and he is expense manager of the employee, manager of the employee
                                             or the employee is in the department managed by the officer
                    Manager                 Always
        =================================================================================
    """
    _name = "hr.expense.sheet"
    _inherit = ['mail.thread.main.attachment', 'mail.activity.mixin']
    _description = "Expense Report"
    _order = "accounting_date desc, id desc"
    _check_company_auto = True

    @api.model
    def _default_employee_id(self):
        return self.env.user.employee_id

    @api.model
    def _default_journal_id(self):
        """
             The journal is determining the company of the accounting entries generated from expense.
             We need to force journal company and expense sheet company to be the same.
        """
        company_journal_id = self.env.company.expense_journal_id
        if company_journal_id:
            return company_journal_id.id
        default_company_id = self.default_get(['company_id'])['company_id']
        journal = self.env['account.journal'].search([
            *self.env['account.journal']._check_company_domain(default_company_id),
            ('type', '=', 'purchase'),
        ], limit=1)
        return journal.id

    name = fields.Char(string="Expense Report Summary", required=True, tracking=True)
    expense_line_ids = fields.One2many(
        comodel_name='hr.expense', inverse_name='sheet_id',
        string="Expense Lines",
        copy=False,
    )
    nb_expense = fields.Integer(compute='_compute_nb_expense', string="Number of Expenses")
    state = fields.Selection(
        selection=[
            ('draft', 'To Submit'),
            ('submit', 'Submitted'),
            ('approve', 'Approved'),
            ('post', 'Posted'),
            ('done', 'Done'),
            ('cancel', 'Refused')
        ],
        string="Status",
        compute='_compute_state', store=True, readonly=True,
        index=True,
        required=True,
        default='draft',
        tracking=True,
        copy=False,
    )
    approval_state = fields.Selection(
        selection=[
            ('submit', 'Submitted'),
            ('approve', 'Approved'),
            ('cancel', 'Refused'),
        ],
        copy=False,
    )
    approval_date = fields.Datetime(string="Approval Date", readonly=True)
    company_id = fields.Many2one(
        comodel_name='res.company',
        string="Company",
        required=True,
        readonly=True,
        default=lambda self: self.env.company,
    )
    employee_id = fields.Many2one(
        comodel_name='hr.employee',
        string="Employee",
        required=True,
        readonly=True,
        default=_default_employee_id,
        domain=[('filter_for_expense', '=', True)],
        check_company=True,
        tracking=True,
    )

    department_id = fields.Many2one(
        comodel_name='hr.department',
        related='employee_id.department_id',
        string="Department",
        store=True,
        copy=False,
    )
    user_id = fields.Many2one(
        comodel_name='res.users',
        string="Manager",
        compute='_compute_from_employee_id', store=True, readonly=True,
        domain=lambda self: [('groups_id', 'in', self.env.ref('hr_expense.group_hr_expense_team_approver').id)],
        copy=False,
        tracking=True,
    )
    product_ids = fields.Many2many(
        comodel_name='product.product',
        string="Categories",
        compute='_compute_product_ids',
        search='_search_product_ids',
        check_company=True,
    )

    # === Amount fields === #
    total_amount = fields.Monetary(
        string="Total",
        currency_field='company_currency_id',
        compute='_compute_amount', store=True, readonly=True,
        tracking=True,
    )
    untaxed_amount = fields.Monetary(
        string="Untaxed Amount",
        currency_field='company_currency_id',
        compute='_compute_amount', store=True, readonly=True,
    )
    total_tax_amount = fields.Monetary(
        string="Taxes",
        currency_field='company_currency_id',
        compute='_compute_amount', store=True, readonly=True,
    )
    amount_residual = fields.Monetary(
        string="Amount Due",
        currency_field='company_currency_id',
        compute='_compute_from_account_move_ids', store=True, readonly=True,
    )
    currency_id = fields.Many2one(
        comodel_name='res.currency',
        string="Currency",
        compute='_compute_currency_id', store=True, readonly=True,
    )
    company_currency_id = fields.Many2one(
        comodel_name='res.currency',
        related='company_id.currency_id',
        string="Report Company Currency"
    )
    is_multiple_currency = fields.Boolean(
        string="Handle lines with different currencies",
        compute='_compute_is_multiple_currency',
    )

    # === Account fields === #
    payment_state = fields.Selection(
        selection=lambda self: self.env["account.move"]._fields["payment_state"]._description_selection(self.env),
        string="Payment Status",
        compute='_compute_from_account_move_ids', store=True, readonly=True,
        copy=False,
        tracking=True,
    )
    payment_mode = fields.Selection(
        related='expense_line_ids.payment_mode',
        string="Paid By",
        tracking=True,
        readonly=True,
    )
    employee_journal_id = fields.Many2one(
        comodel_name='account.journal',
        string="Journal",
        default=_default_journal_id,
        check_company=True,
        domain=[('type', '=', 'purchase')],
        help="The journal used when the expense is paid by employee.",
    )
    selectable_payment_method_line_ids = fields.Many2many(
        comodel_name='account.payment.method.line',
        compute='_compute_selectable_payment_method_line_ids',
    )
    payment_method_line_id = fields.Many2one(
        comodel_name='account.payment.method.line',
        string="Payment Method",
        compute='_compute_payment_method_line_id', store=True, readonly=False,
        domain="[('id', 'in', selectable_payment_method_line_ids)]",
        help="The payment method used when the expense is paid by the company.",
    )
    attachment_ids = fields.One2many(
        comodel_name='ir.attachment',
        inverse_name='res_id',
        domain="[('res_model', '=', 'hr.expense.sheet')]",
        string='Attachments of expenses',
    )
    message_main_attachment_id = fields.Many2one(compute='_compute_main_attachment', store=True)
    accounting_date = fields.Date(string="Accounting Date", compute='_compute_accounting_date', store=True)
    account_move_ids = fields.One2many(
        string="Journal Entries",
        comodel_name='account.move', inverse_name='expense_sheet_id', readonly=True,
    )
    nb_account_move = fields.Integer(string="Number of Journal Entries", compute='_compute_nb_account_move')
    journal_id = fields.Many2one(
        comodel_name='account.journal',
        string="Expense Journal",
        compute='_compute_journal_id', store=True,
        check_company=True,
    )

    # === Security fields === #
    can_reset = fields.Boolean(string='Can Reset', compute='_compute_can_reset')
    can_approve = fields.Boolean(string='Can Approve', compute='_compute_can_approve')
    cannot_approve_reason = fields.Char(string='Cannot Approve Reason', compute='_compute_can_approve')
    is_editable = fields.Boolean(string="Expense Lines Are Editable By Current User", compute='_compute_is_editable')

    _sql_constraints = [(
        'journal_id_required_posted',
        "CHECK((state IN ('post', 'done') AND journal_id IS NOT NULL) OR (state NOT IN ('post', 'done')))",
        'The journal must be set on posted expense'
    )]

    @api.depends('expense_line_ids.total_amount', 'expense_line_ids.tax_amount')
    def _compute_amount(self):
        for sheet in self:
            sheet.total_amount = sum(sheet.expense_line_ids.mapped('total_amount'))
            sheet.total_tax_amount = sum(sheet.expense_line_ids.mapped('tax_amount'))
            sheet.untaxed_amount = sheet.total_amount - sheet.total_tax_amount

    @api.depends('account_move_ids.payment_state', 'account_move_ids.amount_residual')
    def _compute_from_account_move_ids(self):
        for sheet in self:
            if sheet.payment_mode == 'company_account':
                if sheet.account_move_ids:
                    # when the sheet is paid by the company, the state/amount of the related account_move_ids are not relevant
                    # unless all moves have been reversed
                    sheet.amount_residual = 0.
                    if sheet.account_move_ids - sheet.account_move_ids.filtered('reversal_move_id'):
                        sheet.payment_state = 'paid'
                    else:
                        sheet.payment_state = 'reversed'
                else:
                    sheet.amount_residual = sum(sheet.account_move_ids.mapped('amount_residual'))
                    payment_states = set(sheet.account_move_ids.mapped('payment_state'))
                    if len(payment_states) <= 1:  # If only 1 move or only one state
                        sheet.payment_state = payment_states.pop() if payment_states else 'not_paid'
                    elif 'partial' in payment_states or 'paid' in payment_states:  # else if any are (partially) paid
                        sheet.payment_state = 'partial'
                    else:
                        sheet.payment_state = 'not_paid'
            else:
                # Only one move is created when the expenses are paid by the employee
                if sheet.account_move_ids:
                    sheet.amount_residual = sum(sheet.account_move_ids.mapped('amount_residual'))
                    sheet.payment_state = sheet.account_move_ids[:1].payment_state
                else:
                    sheet.amount_residual = 0.0
                    sheet.payment_state = 'not_paid'

    @api.depends('selectable_payment_method_line_ids')
    def _compute_payment_method_line_id(self):
        for sheet in self:
            sheet.payment_method_line_id = sheet.selectable_payment_method_line_ids[:1]

    @api.depends('employee_journal_id', 'payment_method_line_id')
    def _compute_journal_id(self):
        for sheet in self:
            if sheet.payment_mode == 'company_account':
                sheet.journal_id = sheet.payment_method_line_id.journal_id
            else:
                sheet.journal_id = sheet.employee_journal_id

    @api.depends('company_id')
    def _compute_selectable_payment_method_line_ids(self):
        for sheet in self:
            allowed_method_line_ids = sheet.company_id.company_expense_allowed_payment_method_line_ids
            if allowed_method_line_ids:
                sheet.selectable_payment_method_line_ids = allowed_method_line_ids
            else:
                sheet.selectable_payment_method_line_ids = self.env['account.payment.method.line'].search([
                    ('payment_type', '=', 'outbound'),
                    ('company_id', 'parent_of', sheet.company_id.id)
                ])

    @api.depends('account_move_ids', 'payment_state', 'approval_state')
    def _compute_state(self):
        for sheet in self:
            if sheet.payment_state != 'not_paid':
                sheet.state = 'done'
            elif sheet.account_move_ids:
                sheet.state = 'post'
            elif sheet.approval_state:
                sheet.state = sheet.approval_state
            else:
                sheet.state = 'draft'

    @api.depends('expense_line_ids.attachment_ids')
    def _compute_main_attachment(self):
        for sheet in self:
            attachments = sheet.attachment_ids
            if not sheet.message_main_attachment_id or sheet.message_main_attachment_id not in attachments:
                expenses = sheet.expense_line_ids
                expenses_mma_checksums = expenses.message_main_attachment_id.mapped('checksum')
                sheet.message_main_attachment_id = attachments.filtered(
                    lambda att: att.checksum in expenses_mma_checksums
                )[:1] or attachments[:1]

    @api.depends('expense_line_ids.currency_id', 'company_currency_id')
    def _compute_currency_id(self):
        for sheet in self:
            if not sheet.expense_line_ids or sheet.is_multiple_currency or sheet.payment_mode == 'own_account':
                sheet.currency_id = sheet.company_currency_id
            else:
                sheet.currency_id = sheet.expense_line_ids[:1].currency_id

    @api.depends('expense_line_ids.currency_id')
    def _compute_is_multiple_currency(self):
        for sheet in self:
            sheet.is_multiple_currency = any(sheet.expense_line_ids.mapped('is_multiple_currency')) \
                                         or len(sheet.expense_line_ids.mapped('currency_id')) > 1

    @api.depends('employee_id')
    def _compute_can_reset(self):
        is_expense_user = self.user_has_groups('hr_expense.group_hr_expense_team_approver')
        for sheet in self:
            sheet.can_reset = is_expense_user if is_expense_user else sheet.employee_id.user_id == self.env.user

    @api.depends_context('uid')
    @api.depends('employee_id')
    def _compute_can_approve(self):
        is_team_approver = self.user_has_groups('hr_expense.group_hr_expense_team_approver') or self.env.su
        is_approver = self.user_has_groups('hr_expense.group_hr_expense_user') or self.env.su
        is_hr_admin = self.user_has_groups('hr_expense.group_hr_expense_manager') or self.env.su

        for sheet in self:
            reason = False
            if not is_team_approver:
                reason = _("%s: Your are not a Manager or HR Officer", sheet.name)

            elif not is_hr_admin:
                sheet_employee = sheet.employee_id
                current_managers = sheet_employee.expense_manager_id \
                                   | sheet_employee.parent_id.user_id \
                                   | sheet_employee.department_id.manager_id.user_id \
                                   | sheet.user_id

                if sheet_employee.user_id == self.env.user:
                    reason = _("%s: It is your own expense", sheet.name)

                elif self.env.user not in current_managers and not is_approver and sheet_employee.expense_manager_id.id != self.env.user.id:
                    reason = _("%s: It is not from your department", sheet.name)

            sheet.can_approve = not reason
            sheet.cannot_approve_reason = reason

    @api.depends('expense_line_ids')
    def _compute_nb_expense(self):
        for sheet in self:
            sheet.nb_expense = len(sheet.expense_line_ids)

    @api.depends('account_move_ids')
    def _compute_nb_account_move(self):
        for sheet in self:
            sheet.nb_account_move = len(sheet.account_move_ids)

    @api.depends('account_move_ids.date')
    def _compute_accounting_date(self):
        for sheet in self.filtered('account_move_ids'):
            sheet.accounting_date = sheet.account_move_ids[:1].date

    @api.depends('employee_id', 'employee_id.department_id')
    def _compute_from_employee_id(self):
        for sheet in self:
            sheet.department_id = sheet.employee_id.department_id
            sheet.user_id = sheet.employee_id.expense_manager_id or sheet.employee_id.parent_id.user_id

    @api.depends_context('uid')
    @api.depends('employee_id', 'user_id', 'state')
    def _compute_is_editable(self):
        is_hr_admin = self.user_has_groups('hr_expense.group_hr_expense_manager,base.group_system')
        is_approver = self.user_has_groups('hr_expense.group_hr_expense_user')
        for sheet in self:
            if sheet.state not in {'draft', 'submit', 'approve'}:
                # Not editable
                sheet.is_editable = False
                continue

            if is_hr_admin or self.env.su:
                # Administrator-level users are not restricted
                sheet.is_editable = True
                continue

            employee = sheet.employee_id

            is_own_sheet = employee.user_id == self.env.user
            if is_own_sheet and sheet.state == 'draft':
                # Anyone can edit their own draft sheet
                sheet.is_editable = True
                continue

            managers = employee.expense_manager_id | employee.parent_id.user_id | employee.department_id.manager_id.user_id
            if is_approver:
                managers |= self.env.user
            if not is_own_sheet and self.env.user in managers:
                # If Approver-level or designated manager, can edit other people sheet
                sheet.is_editable = True
                continue
            sheet.is_editable = False

    @api.constrains('expense_line_ids')
    def _check_payment_mode(self):
        for sheet in self:
            expense_lines = sheet.mapped('expense_line_ids')
            if expense_lines and any(expense.payment_mode != expense_lines[:1].payment_mode for expense in expense_lines):
                raise ValidationError(_("All expenses in an expense report must have the same \"paid by\" criteria."))

    @api.depends('expense_line_ids')
    def _compute_product_ids(self):
        for sheet in self:
            sheet.product_ids = sheet.expense_line_ids.mapped('product_id')

    @api.constrains('expense_line_ids', 'employee_id')
    def _check_employee(self):
        for sheet in self:
            if sheet.expense_line_ids.employee_id - sheet.employee_id:
                raise ValidationError(_('You cannot add expenses of another employee.'))

    @api.constrains('expense_line_ids', 'company_id')
    def _check_expense_lines_company(self):
        for sheet in self:
            if sheet.expense_line_ids.company_id - sheet.company_id:
                raise ValidationError(_('An expense report must contain only lines from the same company.'))

    @api.model
    def _search_product_ids(self, operator, value):
        if operator == 'in' and not isinstance(value, list):
            value = [value]
        return [('expense_line_ids.product_id', operator, value)]

    # ----------------------------------------
    # ORM Overrides
    # ----------------------------------------

    def _read_format(self, fnames, load='_classic_read'):
        # setting the context in the field on the view is not enough
        self = self.with_context(show_payment_journal_id=True)
        return super()._read_format(fnames, load)

    @api.model_create_multi
    def create(self, vals_list):
        context = clean_context(self.env.context)
        context.update({
            'mail_create_nosubscribe': True,
            'mail_auto_subscribe_no_notify': True,
        })
        sheets = super(HrExpenseSheet, self.with_context(context)).create(vals_list)
        sheets.activity_update()
        return sheets

    def write(self, vals):
        if 'state' in vals or 'approval_state' in vals:
            # Avoid user with write access on expense sheet in draft state to bypass the validation process
            valid_states = {'submit', None}
            if (
                    not self.user_has_groups('hr_expense.group_hr_expense_manager')
                    and any(state == 'draft' for state in self.mapped('state'))
                    and (vals.get('state') not in valid_states or vals.get('approval_state') not in valid_states)
            ):
                raise UserError(_("You don't have the rights to bypass the validation process of this expense report."))
            elif vals.get('state') == 'approve' or vals.get('approval_state') == 'approve':
                self._check_can_approve()
            elif vals.get('state') == 'cancel' or vals.get('approval_state') == 'cancel':
                self._check_can_refuse()
        return super().write(vals)

    @api.ondelete(at_uninstall=False)
    def _unlink_except_posted_or_paid(self):
        for expense in self:
            if expense.state in {'post', 'done'}:
                raise UserError(_('You cannot delete a posted or paid expense.'))

    # --------------------------------------------
    # Mail Thread
    # --------------------------------------------

    def _track_subtype(self, init_values):
        self.ensure_one()
        if 'state' in init_values and self.state == 'draft':
            return self.env.ref('hr_expense.mt_expense_reset')
        if 'state' in init_values and self.state == 'approve':
            if init_values['state'] in {'post', 'done'}:
                return self.env.ref('hr_expense.mt_expense_entry_delete')
            return self.env.ref('hr_expense.mt_expense_approved')
        if 'state' in init_values and self.state == 'cancel':
            return self.env.ref('hr_expense.mt_expense_refused')
        if 'state' in init_values and self.state == 'done':
            return self.env.ref('hr_expense.mt_expense_paid')
        return super()._track_subtype(init_values)

    def _message_auto_subscribe_followers(self, updated_values, subtype_ids):
        res = super()._message_auto_subscribe_followers(updated_values, subtype_ids)
        if updated_values.get('employee_id'):
            employee_user = self.env['hr.employee'].browse(updated_values['employee_id']).user_id
            if employee_user:
                res.append((employee_user.partner_id.id, subtype_ids, False))
        return res

    def activity_update(self):
        reports_requiring_feedback = self.env['hr.expense.sheet']
        reports_activity_unlink = self.env['hr.expense.sheet']
        for expense_report in self:
            if expense_report.state == 'submit':
                expense_report.activity_schedule(
                    'hr_expense.mail_act_expense_approval',
                    user_id=expense_report.sudo()._get_responsible_for_approval().id or self.env.user.id)
            elif expense_report.state == 'approve':
                reports_requiring_feedback |= expense_report
            elif expense_report.state in {'draft', 'cancel'}:
                reports_activity_unlink |= expense_report
        if reports_requiring_feedback:
            reports_requiring_feedback.activity_feedback(['hr_expense.mail_act_expense_approval'])
        if reports_activity_unlink:
            reports_activity_unlink.activity_unlink(['hr_expense.mail_act_expense_approval'])

    # --------------------------------------------
    # Actions
    # --------------------------------------------

    def action_submit_sheet(self):
        self._do_submit()

    def action_approve_expense_sheets(self):
        self._check_can_approve()
        self._validate_analytic_distribution()
        duplicates = self.expense_line_ids.duplicate_expense_ids.filtered(lambda exp: exp.state in {'approved', 'done'})
        if duplicates:
            action = self.env["ir.actions.act_window"]._for_xml_id('hr_expense.hr_expense_approve_duplicate_action')
            action['context'] = {'default_sheet_ids': self.ids, 'default_expense_ids': duplicates.ids}
            return action
        self._do_approve()

    def action_refuse_expense_sheets(self):
        self._check_can_refuse()
        return self.env["ir.actions.act_window"]._for_xml_id('hr_expense.hr_expense_refuse_wizard_action')

    def action_reset_approval_expense_sheets(self):
        self._check_can_reset_approval()
        self._do_reset_approval()

    def action_sheet_move_create(self):
        self._check_can_create_move()
        self._do_create_moves()

    def action_reset_expense_sheets(self):
        self._do_reverse_moves()
        self._do_reset_approval()

    def action_register_payment(self):
        ''' Open the account.payment.register wizard to pay the selected journal entries.
        There can be more than one bank_account_id in the expense sheet when registering payment for multiple expenses.
        The default_partner_bank_id is set only if there is one available, if more than one the field is left empty.
        :return: An action opening the account.payment.register wizard.
        '''
        return self.account_move_ids.with_context(default_partner_bank_id=(
            self.account_move_ids.partner_bank_id.id if len(self.account_move_ids.partner_bank_id.ids) <= 1 else None
        )).action_register_payment()

    def action_open_expense_view(self):
        self.ensure_one()
        if self.nb_expense == 1:
            return {
                'type': 'ir.actions.act_window',
                'view_mode': 'form',
                'res_model': 'hr.expense',
                'res_id': self.expense_line_ids.id,
            }
        return {
            'name': _('Expenses'),
            'type': 'ir.actions.act_window',
            'view_mode': 'list,form',
            'views': [[False, "list"], [False, "form"]],
            'res_model': 'hr.expense',
            'domain': [('id', 'in', self.expense_line_ids.ids)],
        }

    def action_open_account_moves(self):
        self.ensure_one()
        if self.payment_mode == 'own_account':
            res_model = 'account.move'
            record_ids = self.account_move_ids
        else:
            res_model = 'account.payment'
            record_ids = self.account_move_ids.mapped('payment_id')

        action = {'type': 'ir.actions.act_window', 'res_model': res_model}
        if len(self.account_move_ids) == 1:
            action.update({
                'name': record_ids.name,
                'view_mode': 'form',
                'res_id': record_ids.id,
                'views': [(False, 'form')],
            })
        else:
            action.update({
                'name': _("Journal entries"),
                'view_mode': 'list',
                'domain': [('id', 'in', record_ids.ids)],
                'views': [(False, 'list'), (False, 'form')],
            })
        return action

    # --------------------------------------------
    # Business
    # --------------------------------------------

    def set_to_paid(self):
        # hook used in other modules to bypass payment registration
        self.write({'state': 'done'})

    def set_to_posted(self):
        # hook used in other modules to bypass move creation
        self.write({'state': 'post'})

    def _check_can_approve(self):
        if not all(self.mapped('can_approve')):
            reasons = _("You cannot approve:\n %s", "\n".join(self.mapped('cannot_approve_reason')))
            raise UserError(reasons)

    def _check_can_refuse(self):
        if not all(self.mapped('can_approve')):
            reasons = _("You cannot refuse:\n %s", "\n".join(self.mapped('cannot_approve_reason')))
            raise UserError(reasons)

    def _check_can_reset_approval(self):
        if not all(self.mapped('can_reset')):
            raise UserError(_("Only HR Officers or the concerned employee can reset to draft."))

    def _check_can_create_move(self):
        if any(sheet.state != 'approve' for sheet in self):
            raise UserError(_("You can only generate accounting entry for approved expense(s)."))

        if any(not sheet.journal_id for sheet in self):
            raise UserError(_("Please specify an expense journal in order to generate accounting entries."))

        missing_email_employees = self.filtered(lambda sheet: not sheet.employee_id.work_email).employee_id
        if missing_email_employees:
            action = self.env['ir.actions.actions']._for_xml_id('hr.open_view_employee_tree')
            action['domain'] = [('id', 'in', missing_email_employees.ids)]
            raise RedirectWarning(_("The work email of some employees is missing. Please add it on the employee form"), action, _("Show missing work email employees"))

    def _do_submit(self):
        self.write({'approval_state': 'submit'})
        self.sudo().activity_update()

    def _do_approve(self):
        for sheet in self.filtered(lambda s: s.state in {'submit', 'draft'}):
            sheet.write({
                'approval_state': 'approve',
                'user_id': sheet.user_id.id or self.env.user.id,
                'approval_date': fields.Date.context_today(sheet),
            })
        self.activity_update()

    def _do_reset_approval(self):
        self.sudo().write({'approval_state': False, 'accounting_date': False})
        self.activity_update()

    def _do_refuse(self, reason):
        if self.account_move_ids:  # Todo: in 17.3+, edit it to allow draft entries
            raise UserError(_("You cannot cancel an expense sheet linked to a journal entry"))
        self.approval_state = 'cancel'
        subtype_id = self.env['ir.model.data']._xmlid_to_res_id('mail.mt_comment')
        for sheet in self:
            sheet.message_post_with_source(
                'hr_expense.hr_expense_template_refuse_reason',
                subtype_id=subtype_id,
                render_values={'reason': reason, 'name': sheet.name},
            )
        self.activity_update()

    def _do_create_moves(self):
        self = self.with_context(clean_context(self.env.context))  # remove default_*
        skip_context = {
            'skip_invoice_sync': True,
            'skip_invoice_line_sync': True,
            'skip_account_move_synchronization': True,
        }
        own_account_sheets = self.filtered(lambda sheet: sheet.payment_mode == 'own_account')
        company_account_sheets = self - own_account_sheets

        moves = self.env['account.move'].create([sheet._prepare_bills_vals() for sheet in own_account_sheets])
        # Set the main attachment on the moves directly to avoid recomputing the
        # `register_as_main_attachment` on the moves which triggers the OCR again
        for move in moves:
            move.message_main_attachment_id = move.attachment_ids[0] if move.attachment_ids else None
        payments = self.env['account.payment'].with_context(**skip_context).create([
            expense._prepare_payments_vals() for expense in company_account_sheets.expense_line_ids
        ])
        moves |= payments.move_id
        moves.action_post()
        self.activity_update()

        return moves

    def _do_reverse_moves(self):
        self = self.with_context(clean_context(self.env.context))
        moves = self.account_move_ids
        if moves:
            moves_sudo = self.sudo().account_move_ids
            draft_moves_sudo = moves_sudo.filtered(lambda m: m.state == 'draft')
            non_draft_moves_sudo = moves_sudo - draft_moves_sudo
            non_draft_moves_sudo._reverse_moves(
                default_values_list=[{'invoice_date': fields.Date.context_today(move), 'ref': False} for move in non_draft_moves_sudo],
                cancel=True
            )
            draft_moves_sudo.unlink()

    def _prepare_bills_vals(self):
        self.ensure_one()
        move_vals = self._prepare_move_vals()
        if self.employee_id.sudo().bank_account_id:
            move_vals['partner_bank_id'] = self.employee_id.sudo().bank_account_id.id
        return {
            **move_vals,
            'invoice_date': self.accounting_date or fields.Date.context_today(self),
            'journal_id': self.journal_id.id,
            'ref': self.name,
            'move_type': 'in_invoice',
            'partner_id': self.employee_id.sudo().work_contact_id.id,
            'commercial_partner_id': self.employee_id.user_partner_id.id,
            'currency_id': self.currency_id.id,
            'line_ids': [Command.create(expense._prepare_move_lines_vals()) for expense in self.expense_line_ids],
            'attachment_ids': [
                Command.create(attachment.copy_data({'res_model': 'account.move', 'res_id': False, 'raw': attachment.raw})[0])
                for attachment in self.expense_line_ids.message_main_attachment_id
            ],
        }

    def _prepare_move_vals(self):
        self.ensure_one()
        return {
            # force the name to the default value, to avoid an eventual 'default_name' in the context
            # to set it to '' which cause no number to be given to the account.move when posted.
            'name': '/',
            'date': self.accounting_date or max(self.expense_line_ids.filtered(lambda exp: exp.date).mapped('date'), default=fields.Date.context_today(self)),
            'expense_sheet_id': self.id,
        }

    def _validate_analytic_distribution(self):
        for line in self.expense_line_ids:
            line._validate_distribution(account=line.account_id.id, product=line.product_id.id, business_domain='expense', company_id=line.company_id.id)

    def _get_responsible_for_approval(self):
        if self.user_id:
            return self.user_id
        if self.employee_id.parent_id.user_id:
            return self.employee_id.parent_id.user_id
        if self.employee_id.department_id.manager_id.user_id:
            return self.employee_id.department_id.manager_id.user_id
        return self.env['res.users']

    def _get_expense_account_destination(self):
        self.ensure_one()
        if self.payment_mode == 'company_account':
            journal = self.payment_method_line_id.journal_id
            account_dest = (
                self.payment_method_line_id.payment_account_id
                or journal.company_id.account_journal_payment_credit_account_id
            )
        else:
            if not self.employee_id.sudo().work_contact_id:
                raise UserError(_("No work contact found for the employee %s, please configure one.", self.employee_id.name))
            partner = self.employee_id.sudo().work_contact_id.with_company(self.company_id)
            account_dest = partner.property_account_payable_id or partner.parent_id.property_account_payable_id
        return account_dest.id

```

## File: models\ir_actions_report.py

```python
import io
from odoo import models
from odoo.tools import pdf
from odoo.tools.pdf import OdooPdfFileReader, OdooPdfFileWriter


class IrActionsReport(models.Model):
    _inherit = 'ir.actions.report'

    def _render_qweb_pdf_prepare_streams(self, report_ref, data, res_ids=None):
        # OVERRIDE
        res = super()._render_qweb_pdf_prepare_streams(report_ref, data, res_ids)
        if not res_ids:
            return res
        report = self._get_report(report_ref)
        if report.report_name == 'hr_expense.report_expense_sheet':
            expense_sheets = self.env['hr.expense.sheet'].browse(res_ids)
            for expense_sheet in expense_sheets:
                # Will contains the expense report
                stream_list = []
                stream = res[expense_sheet.id]['stream']
                stream_list.append(stream)
                attachments = self.env['ir.attachment'].search([('res_id', 'in', expense_sheet.expense_line_ids.ids), ('res_model', '=', 'hr.expense')])
                expense_report = OdooPdfFileReader(stream, strict=False)
                output_pdf = OdooPdfFileWriter()
                output_pdf.appendPagesFromReader(expense_report)
                for attachment in attachments:
                    if attachment.mimetype == 'application/pdf':
                        attachment_stream = pdf.to_pdf_stream(attachment)
                    else:
                        # In case the attachment is not a pdf we will create a new PDF from the template "report_expense_sheet_img"
                        # And then append to the stream. By doing so, the attachment is put on a new page with the name of the expense
                        # associated to the attachment
                        data['attachment'] = attachment
                        attachment_prep_stream = self._render_qweb_pdf_prepare_streams('hr_expense.report_expense_sheet_img', data, res_ids=res_ids)
                        attachment_stream = attachment_prep_stream[expense_sheet.id]['stream']
                    attachment_reader = OdooPdfFileReader(attachment_stream, strict=False)
                    output_pdf.appendPagesFromReader(attachment_reader)
                    stream_list.append(attachment_stream)

                new_pdf_stream = io.BytesIO()
                output_pdf.write(new_pdf_stream)
                res[expense_sheet.id]['stream'] = new_pdf_stream

                for stream in stream_list:
                    stream.close()
        return res

```

## File: models\ir_attachment.py

```python
from odoo import models, api


class IrAttachment(models.Model):
    _inherit = 'ir.attachment'

    @api.model_create_multi
    def create(self, vals_list):
        attachments = super().create(vals_list)
        if self.env.context.get('sync_attachment', True):
            expenses_attachments = attachments.filtered(lambda att: att.res_model == 'hr.expense')
            if expenses_attachments:
                expenses = self.env['hr.expense'].browse(expenses_attachments.mapped('res_id'))
                for expense in expenses.filtered('sheet_id'):
                    checksums = set(expense.sheet_id.attachment_ids.mapped('checksum'))
                    for attachment in expense.attachment_ids.filtered(lambda att: att.checksum not in checksums):
                        attachment.copy({
                            'res_model': 'hr.expense.sheet',
                            'res_id': expense.sheet_id.id,
                        })
        return attachments

    def unlink(self):
        if self.env.context.get('sync_attachment', True):
            attachments_to_unlink = self.env['ir.attachment']
            expenses_attachments = self.filtered(lambda att: att.res_model == 'hr.expense')
            if expenses_attachments:
                expenses = self.env['hr.expense'].browse(expenses_attachments.mapped('res_id'))
                for expense in expenses.exists().filtered('sheet_id'):
                    checksums = set(expense.attachment_ids.mapped('checksum'))
                    attachments_to_unlink += expense.sheet_id.attachment_ids.filtered(lambda att: att.checksum in checksums)
            sheets_attachments = self.filtered(lambda att: att.res_model == 'hr.expense.sheet')
            if sheets_attachments:
                sheets = self.env['hr.expense.sheet'].browse(sheets_attachments.mapped('res_id'))
                for sheet in sheets.exists():
                    checksums = set((sheet.attachment_ids & sheets_attachments).mapped('checksum'))
                    attachments_to_unlink += sheet.expense_line_ids.attachment_ids.filtered(lambda att: att.checksum in checksums)
            super(IrAttachment, attachments_to_unlink).unlink()
        return super().unlink()

```

## File: models\product_product.py

```python
from odoo import api, fields, models, _


class ProductProduct(models.Model):
    _inherit = "product.product"

    standard_price_update_warning = fields.Char(compute="_compute_standard_price_update_warning")

    @api.onchange('standard_price')
    def _compute_standard_price_update_warning(self):
        undone_expenses = self.env['hr.expense']._read_group(
            domain=[('state', 'in', ['draft', 'reported']), ('product_id', 'in', self.ids)],
            groupby=['price_unit'],
            )
        # The following list is composed of all the unit_amounts of expenses that use this product and should NOT trigger a warning.
        # Those are the amounts of any undone expense using this product and 0.0 which is the default unit_amount.
        unit_amounts_no_warning = [self.env.company.currency_id.round(row[0]) for row in undone_expenses]
        for product in self:
            product.standard_price_update_warning = False
            if undone_expenses:
                rounded_price = self.env.company.currency_id.round(product.standard_price)
                if rounded_price and (len(unit_amounts_no_warning) > 1 or (len(unit_amounts_no_warning) == 1 and rounded_price not in unit_amounts_no_warning)):
                    product.standard_price_update_warning = _(
                            "There are unsubmitted expenses linked to this category. Updating the category cost will change expense amounts. "
                            "Make sure it is what you want to do."
                        )

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.tools.sql import column_exists, create_column


class ProductTemplate(models.Model):
    _inherit = "product.template"

    @api.model
    def default_get(self, fields):
        result = super(ProductTemplate, self).default_get(fields)
        if self.env.context.get('default_can_be_expensed'):
            result['supplier_taxes_id'] = False
        return result

    can_be_expensed = fields.Boolean(string="Can be Expensed", compute='_compute_can_be_expensed',
        store=True, readonly=False, help="Specify whether the product can be selected in an expense.")

    def _auto_init(self):
        if not column_exists(self.env.cr, "product_template", "can_be_expensed"):
            create_column(self.env.cr, "product_template", "can_be_expensed", "boolean")
            self.env.cr.execute(
                """
                UPDATE product_template
                SET can_be_expensed = false
                WHERE type NOT IN ('consu', 'service')
                """
            )
        return super()._auto_init()

    @api.depends('type')
    def _compute_can_be_expensed(self):
        self.filtered(lambda p: p.type not in ['consu', 'service']).update({'can_be_expensed': False})

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = "res.company"

    expense_product_id = fields.Many2one(
        "product.product",
        string="Default Expense Category",
        check_company=True,
        domain="[('can_be_expensed', '=', True)]",
    )
    expense_journal_id = fields.Many2one(
        "account.journal",
        string="Default Expense Journal",
        check_company=True,
        domain="[('type', '=', 'purchase')]",
        help="The company's default journal used when an employee expense is created.",
    )
    company_expense_allowed_payment_method_line_ids = fields.Many2many(
        "account.payment.method.line",
        string="Payment methods available for expenses paid by company",
        check_company=True,
        domain="[('payment_type', '=', 'outbound'), ('journal_id', '!=', False)]",
    )

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    hr_expense_alias_prefix = fields.Char(
        'Default Alias Name for Expenses',
        compute='_compute_hr_expense_alias_prefix',
        store=True,
        readonly=False)
    hr_expense_alias_domain_id = fields.Many2one(
        comodel_name='mail.alias.domain',
        compute='_compute_hr_expense_alias_domain_id',
        inverse='_inverse_hr_expense_alias_domain_id',
        readonly=False)
    hr_expense_use_mailgateway = fields.Boolean(string='Let your employees record expenses by email',
                                             config_parameter='hr_expense.use_mailgateway')
    module_hr_payroll_expense = fields.Boolean(string='Reimburse Expenses in Payslip')
    module_hr_expense_extract = fields.Boolean(string='Send bills to OCR to generate expenses')
    expense_product_id = fields.Many2one('product.product', related='company_id.expense_product_id', readonly=False, check_company=True)
    expense_journal_id = fields.Many2one('account.journal', related='company_id.expense_journal_id', readonly=False, check_company=True)
    company_expense_allowed_payment_method_line_ids = fields.Many2many(
        comodel_name='account.payment.method.line',
        check_company=True,
        related='company_id.company_expense_allowed_payment_method_line_ids',
        readonly=False,
    )

    @api.model
    def get_values(self):
        res = super(ResConfigSettings, self).get_values()
        expense_alias = self.env.ref('hr_expense.mail_alias_expense', raise_if_not_found=False)
        res.update(
            hr_expense_alias_prefix=expense_alias.alias_name if expense_alias else False,
            hr_expense_alias_domain_id=expense_alias.alias_domain_id if expense_alias else False,
        )
        return res

    def set_values(self):
        super().set_values()
        expense_alias = self.env.ref('hr_expense.mail_alias_expense', raise_if_not_found=False)
        if not expense_alias and self.hr_expense_alias_prefix:
            # create data again
            alias = self.env['mail.alias'].sudo().create({
                'alias_contact': 'employees',
                'alias_domain_id': self.env.company.alias_domain_id.id,
                'alias_model_id': self.env['ir.model']._get_id('hr.expense'),
                'alias_name': self.hr_expense_alias_prefix,
            })
            self.env['ir.model.data'].sudo().create({
                'name': 'mail_alias_expense',
                'module': 'hr_expense',
                'model': 'mail.alias',
                'noupdate': True,
                'res_id': alias.id,
            })
        elif expense_alias and expense_alias.alias_name != self.hr_expense_alias_prefix:
            expense_alias.alias_name = self.hr_expense_alias_prefix

    @api.depends('hr_expense_use_mailgateway')
    def _compute_hr_expense_alias_prefix(self):
        self.filtered(lambda w: not w.hr_expense_use_mailgateway).hr_expense_alias_prefix = False

    @api.depends('hr_expense_use_mailgateway')
    def _compute_hr_expense_alias_domain_id(self):
        self.filtered(lambda w: not w.hr_expense_use_mailgateway).hr_expense_alias_domain_id = False

    def _inverse_hr_expense_alias_domain_id(self):
        expense_alias = self.env.ref('hr_expense.mail_alias_expense', raise_if_not_found=False)
        for record in self:
            if expense_alias and expense_alias.alias_domain_id != record.hr_expense_alias_domain_id:
                expense_alias.alias_domain_id = record.hr_expense_alias_domain_id

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_employee
from . import account_move
from . import account_move_line
from . import account_payment
from . import account_tax
from . import hr_department
from . import hr_expense
from . import hr_expense_sheet
from . import ir_attachment
from . import product_product
from . import product_template
from . import res_config_settings
from . import account_journal_dashboard
from . import res_company
from . import analytic
from . import ir_actions_report

```

## File: report\hr_expense_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_expense_sheet">
        <t t-call="web.html_container">
            <t t-foreach="docs" t-as="o">
                <t t-call="web.external_layout">
                    <div class="page o_content_pdf">
                        <div class="oe_structure"></div>
                        <h2>Expenses Report</h2>
                        <h3><span t-out="o.name">Business Trip</span></h3>
                        <div class="row o_header">
                            <div class="col-6">
                                <div class="row">
                                    <div class="col-3 fw-bold"><span>Employee:</span> </div>
                                    <div class="col-9 text-muted"><span t-field="o.employee_id.name">Marc Demo</span></div>
                                </div>
                                <div class="row" t-if="o.accounting_date">
                                    <div class="col-3 fw-bold"><span>Date:</span></div>
                                    <div class="col-9 text-muted"><span t-field="o.accounting_date">2023-08-11</span></div>
                                </div>
                            </div>
                            <div class="col-6">
                                <div class="row" t-if="o.user_id.name">
                                    <div class="col-3 fw-bold"><span>Manager:</span></div>
                                    <div class="col-9 text-muted"><span t-field="o.user_id">Mitchell Admin</span></div>
                                </div>
                                <div class="row">
                                    <div class="col-3 fw-bold"><span>Paid by:</span></div>
                                    <div class="col-9 text-muted"><span t-field="o.payment_mode">Credit Card</span></div>
                                </div>
                            </div>
                        </div>
                        <div class="oe_structure"></div>

                        <table class="o_table">
                            <thead>
                                <tr>
                                    <th class="text-start">Date</th>
                                    <th class="text-start">Name</th>
                                    <th class="text-end">Unit Price</th>
                                    <th class="text-end">Quantity</th>
                                    <th class="text-end">Taxes</th>
                                    <t t-set="foreign_currencies" t-value="o.expense_line_ids.currency_id - o.company_currency_id"/>
                                    <th t-if="foreign_currencies" class="text-end">Subtotal in currency</th>
                                    <th class="text-end">Subtotal</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr t-foreach="o.expense_line_ids" t-as="line">
                                    <td class="text-start"><span t-field="line.date"></span></td>
                                    <td t-att-class="'text-start' + (' o_overflow' if len(line.name) > 30 else '')">
                                        <span t-field="line.name">Flight Ticket</span>
                                    </td>
                                    <td class="text-end"><span t-field="line.price_unit">$100.00</span></td>
                                    <td class="text-end"><span t-field="line.quantity">1</span></td>
                                    <t t-set="taxes" t-value="', '.join([(tax.invoice_label or tax.name) for tax in line.tax_ids])"/>
                                    <td t-attf-class="text-end {{ 'text-nowrap' if len(taxes) &lt; 10 else '' }}">
                                        <span t-out="taxes" id="line_tax_ids">Tax 15%</span>
                                    </td>
                                    <td t-if="foreign_currencies" class="text-end">
                                        <span t-field="line.total_amount_currency" t-options='{"widget": "monetary", "display_currency": line.currency_id}'>$120.00</span>
                                    </td>
                                    <td class="text-end">
                                        <span t-field="line.total_amount" t-options='{"widget": "monetary", "display_currency": o.company_currency_id}'>$100.00</span>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                        <div class="oe_structure"></div>
                        <div class="row justify-content-end o_total">
                            <div class="col-4">
                                <div class="oe_structure"></div>
                                <table class="o_subtable">
                                    <tbody>
                                        <tr>
                                            <td>Untaxed Amount</td>
                                            <td class="text-end"><span t-field="o.untaxed_amount" t-options='{"widget": "monetary", "display_currency": o.currency_id}'>$500.00</span></td>
                                        </tr>
                                        <tr>
                                            <td>Taxes</td>
                                            <td class="text-end"><span t-field="o.total_tax_amount" t-options='{"widget": "monetary", "display_currency": o.currency_id}'>$100.00</span></td>
                                        </tr>
                                        <tr class="fw-bold">
                                            <td>Total</td>
                                            <td class="text-end">
                                                <span t-field="o.total_amount" t-options='{"widget": "monetary", "display_currency": o.currency_id}'>$600.00</span>
                                            </td>
                                        </tr>
                                    </tbody>
                                </table>
                                <div class="oe_structure"></div>
                            </div>
                        </div>
                    </div>
                </t>
            </t>
        </t>
    </template>

    <template id="report_expense_sheet_img">
        <t t-call="web.html_container">
            <t t-foreach="docs" t-as="o">
                <t t-call="web.basic_layout">
                    <div class="oe_structure"></div>
                    <div t-if="attachment and attachment.mimetype != 'application/pdf'">
                        <h3> <span t-out="attachment.res_name">Attachment Name</span> </h3>
                        <img t-att-src="attachment.image_src" class="o_attachment_pdf"/>
                    </div>
                </t>
            </t>
        </t>
    </template>

    <record id="action_report_hr_expense_sheet" model="ir.actions.report">
        <field name="name">Expenses Report</field>
        <field name="model">hr.expense.sheet</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">hr_expense.report_expense_sheet</field>
        <field name="report_file">hr_expense.report_expense_sheet</field>
        <field name="print_report_name">'Expenses - %s - %s' % (object.employee_id.name, (object.name).replace('/', ''))</field>
        <field name="binding_model_id" ref="model_hr_expense_sheet"/>
        <field name="binding_type">report</field>
    </record>

    <record id="action_report_expense_sheet_img" model="ir.actions.report">
        <field name="name">Expense Report Image</field>
        <field name="model">hr.expense.sheet</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">hr_expense.report_expense_sheet_img</field>
        <field name="report_file">hr_expense.report_expense_sheet_img</field>
    </record>
</odoo>

```

## File: security\hr_expense_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.module.category" id="base.module_category_human_resources_expenses">
        <field name="description">Helps you manage your expenses.</field>
        <field name="sequence">12</field>
    </record>

    <record id="group_hr_expense_team_approver" model="res.groups">
        <field name="name">Team Approver</field>
        <field name="category_id" ref="base.module_category_human_resources_expenses"/>
        <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <record id="group_hr_expense_user" model="res.groups">
        <field name="name">All Approver</field>
        <field name="category_id" ref="base.module_category_human_resources_expenses"/>
        <field name="implied_ids" eval="[(4, ref('hr_expense.group_hr_expense_team_approver'))]"/>
    </record>

    <record id="group_hr_expense_manager" model="res.groups">
        <field name="name">Administrator</field>
        <field name="category_id" ref="base.module_category_human_resources_expenses"/>
        <field name="implied_ids" eval="[(4, ref('hr_expense.group_hr_expense_user'))]"/>
        <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
    </record>

    <record id="base.default_user" model="res.users">
        <field name="groups_id" eval="[(4,ref('hr_expense.group_hr_expense_manager'))]"/>
    </record>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_expense_employee,hr.expense.employee,model_hr_expense,base.group_user,1,1,1,1
access_hr_expense_sheet_employee,hr.expense.sheet.employee,model_hr_expense_sheet,base.group_user,1,1,1,1
access_hr_expense_user,hr.expense.user,model_hr_expense,hr_expense.group_hr_expense_team_approver,1,1,1,1
access_hr_expense_sheet_user,hr.expense.sheet.user,model_hr_expense_sheet,hr_expense.group_hr_expense_team_approver,1,1,1,1
access_hr_expense_manager,hr.expense.manager,model_hr_expense,hr_expense.group_hr_expense_manager,1,1,1,1
access_hr_expense_sheet_manager,hr.expense.sheet.manager,model_hr_expense_sheet,hr_expense.group_hr_expense_manager,1,1,1,1
access_product_product_hr_expense_user,product.product.hr.expense.user,product.model_product_product,hr_expense.group_hr_expense_manager,1,1,1,1
access_product_template_hr_expense_user,product.template.hr.expense.user,product.model_product_template,hr_expense.group_hr_expense_manager,1,1,1,1
access_uom_uom_hr_expense_user,uom.uom.hr.expense.user,uom.model_uom_uom,hr_expense.group_hr_expense_team_approver,1,1,1,1
access_account_journal_user,account.journal.user,account.model_account_journal,hr_expense.group_hr_expense_team_approver,1,0,0,0
access_account_journal_employee,account.journal.employee,account.model_account_journal,base.group_user,1,0,0,0
access_account_move_user,account.move.hr.expense.approver,account.model_account_move,hr_expense.group_hr_expense_team_approver,1,0,0,0
access_account_move_line_user,account.move.line.hr.expense.approver,account.model_account_move_line,hr_expense.group_hr_expense_team_approver,1,0,0,0
access_account_analytic_line_user,account.analytic.line.user,account.model_account_analytic_line,hr_expense.group_hr_expense_team_approver,1,1,1,1
access_mail_activity_type_expense_user,mail.activity.type.expense.user,mail.model_mail_activity_type,hr_expense.group_hr_expense_manager,1,1,1,1
access_hr_expense_refuse_wizard,access.hr.expense.refuse.wizard,model_hr_expense_refuse_wizard,hr_expense.group_hr_expense_team_approver,1,1,1,0
access_hr_expense_approve_duplicate,access.hr.expense.approve.duplicate,model_hr_expense_approve_duplicate,hr_expense.group_hr_expense_team_approver,1,1,1,0
access_hr_expense_split_wizard_manager,access.hr.expense.split.wizard.manager,model_hr_expense_split_wizard,base.group_user,1,1,1,0
access_hr_expense_split_manager,access.hr.expense.split.manager,model_hr_expense_split,base.group_user,1,1,1,1

```

## File: security\ir_rule.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

        <record id="ir_rule_hr_expense_manager" model="ir.rule">
            <field name="name">Manager Expense</field>
            <field name="model_id" ref="model_hr_expense"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[
                (4, ref('account.group_account_user')),
                (4, ref('hr_expense.group_hr_expense_user'))]"/>
        </record>
        <record id="ir_rule_hr_expense_approver" model="ir.rule">
            <field name="name">Team Approver Expense</field>
            <field name="model_id" ref="model_hr_expense"/>
            <field name="domain_force">['|', '|', '|', '|',
                ('employee_id.user_id', '=', user.id),
                ('employee_id.department_id.manager_id.user_id', '=', user.id),
                ('employee_id.parent_id.user_id', '=', user.id),
                ('employee_id.expense_manager_id', '=', user.id),
                ('sheet_id.user_id', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('hr_expense.group_hr_expense_team_approver'))]"/>
        </record>
        <record id="ir_rule_hr_expense_employee" model="ir.rule">
            <field name="name">Employee Expense</field>
            <field name="model_id" ref="model_hr_expense"/>
            <field name="domain_force">[('employee_id.user_id', '=', user.id), ('state', 'in', ('draft', 'reported'))]</field>
            <field name="groups" eval="[(4, ref('base.group_user'))]"/>
        </record>
        <record id="ir_rule_hr_expense_employee_not_draft" model="ir.rule">
            <field name="name">Employee can't modify expense that is not in draft state</field>
            <field name="model_id" ref="model_hr_expense"/>
            <field name="domain_force">[('employee_id.user_id', '=', user.id), ('state', 'not in', ('draft', 'reported'))]</field>
            <field name="groups" eval="[(4, ref('base.group_user'))]"/>
            <field name="perm_create" eval="False"/>
            <field name="perm_write" eval="False"/>
            <field name="perm_unlink" eval="False"/>
        </record>

        <record id="ir_rule_hr_expense_sheet_manager" model="ir.rule">
            <field name="name">Manager Expense Sheet</field>
            <field name="model_id" ref="model_hr_expense_sheet"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="groups" eval="[
                (4, ref('account.group_account_user')),
                (4, ref('hr_expense.group_hr_expense_user'))]"/>
        </record>
        <record id="ir_rule_hr_expense_sheet_approver" model="ir.rule">
            <field name="name">Approver Expense Sheet</field>
            <field name="model_id" ref="model_hr_expense_sheet"/>
            <field name="domain_force">['|', '|', '|', '|',
                ('employee_id.user_id', '=', user.id),
                ('employee_id.department_id.manager_id.user_id', '=', user.id),
                ('employee_id.parent_id.user_id', '=', user.id),
                ('employee_id.expense_manager_id', '=', user.id),
                ('user_id', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('hr_expense.group_hr_expense_team_approver'))]"/>
        </record>
        <record id="ir_rule_hr_expense_sheet_employee" model="ir.rule">
            <field name="name">Employee Expense Sheet</field>
            <field name="model_id" ref="model_hr_expense_sheet"/>
            <field name="domain_force">[('employee_id.user_id', '=', user.id), ('state', '=', 'draft')]</field>
            <field name="groups" eval="[(4, ref('base.group_user'))]"/>
        </record>
        <record id="ir_rule_hr_expense_sheet_employee_not_draft" model="ir.rule">
            <field name="name">Employee can't modify expense sheet that is not in draft state</field>
            <field name="model_id" ref="model_hr_expense_sheet"/>
            <field name="domain_force">[('employee_id.user_id', '=', user.id), ('state', '!=', 'draft')]</field>
            <field name="groups" eval="[(4, ref('base.group_user'))]"/>
            <field name="perm_create" eval="False"/>
            <field name="perm_write" eval="False"/>
            <field name="perm_unlink" eval="False"/>
        </record>

        <record id="hr_expense_comp_rule" model="ir.rule">
            <field name="name">Expense multi company rule</field>
            <field name="model_id" ref="model_hr_expense"/>
            <field eval="True" name="global"/>
            <field name="domain_force">[('company_id', 'in', company_ids)]</field>
        </record>
        <record id="hr_expense_report_comp_rule" model="ir.rule">
            <field name="name">Expense Report multi company rule</field>
            <field name="model_id" ref="model_hr_expense_sheet"/>
            <field eval="True" name="global"/>
            <field name="domain_force">[('company_id', 'in', company_ids)]</field>
        </record>

        <record id="hr_expense_team_approver_account_move_rule" model="ir.rule">
            <field name="name">Expense Team Approver Account Move</field>
            <field name="model_id" ref="account.model_account_move"/>
            <field name="domain_force">[('line_ids.expense_id', '!=', False)]</field>
            <field name="groups" eval="[(4, ref('hr_expense.group_hr_expense_team_approver'))]"/>
        </record>
        <record id="hr_expense_team_approver_account_move_line_rule" model="ir.rule">
            <field name="name">Expense Team Approver Account Move Line</field>
            <field name="model_id" ref="account.model_account_move_line"/>
            <field name="domain_force">[('expense_id', '!=', False)]</field>
            <field name="groups" eval="[(4, ref('hr_expense.group_hr_expense_team_approver'))]"/>
        </record>

</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M28 16c0 5.523-4.477 10-10 10S8 21.523 8 16 12.477 6 18 6s10 4.477 10 10Z" fill="#2EBCFA"/><path d="M43 24a5 5 0 1 1-10 0 5 5 0 0 1 10 0Zm-27 7h26a4 4 0 0 1 4 4v5a4 4 0 0 1-4 4H20a4 4 0 0 1-4-4v-9Z" fill="#144496"/><path d="M4 31h16c7.18 0 13 5.82 13 13H17C9.82 44 4 38.18 4 31Z" fill="#2EBCFA"/><path d="M17.617 22h.787v-1.138C20.558 20.73 22 19.668 22 17.905v-.014c0-1.554-.997-2.332-2.962-2.762l-.634-.132v-2.013c.69.111 1.143.493 1.24 1.139l.008.014 2.188-.007.007-.007c-.077-1.687-1.338-2.811-3.443-2.957V10h-.787v1.16c-2.05.096-3.52 1.151-3.52 2.886v.014c0 1.562 1.032 2.402 2.893 2.811l.627.132v2.054c-.816-.076-1.29-.416-1.401-1.027l-.007-.014-2.195.007-.014.007c.056 1.777 1.491 2.755 3.617 2.839V22Zm-1.185-8.127v-.014c0-.5.404-.82 1.185-.889v1.847c-.809-.216-1.185-.486-1.185-.944Zm3.234 4.282v.014c0 .534-.454.819-1.262.888v-1.88c.906.228 1.262.464 1.262.978Z" fill="#fff"/></svg>

```

## File: static\img\communication.svg

```svg
<svg width="70" height="70" viewBox="0 0 70 70" fill="none" xmlns="http://www.w3.org/2000/svg">
<rect x="17" y="5" width="36" height="60" rx="1" stroke="#875B7B" stroke-width="2"/>
<rect x="18" y="13" width="35" height="2" fill="#875B7B"/>
<rect x="18" y="54" width="35" height="2" fill="#875B7B"/>
<circle cx="35" cy="60" r="2" stroke="#C0A5BD" stroke-width="2"/>
<rect x="31" y="9" width="6" height="2" rx="1" fill="#C0A5BD"/>
<rect x="38" y="9" width="2" height="2" rx="1" fill="#C0A5BD"/>
<path d="M53 16H55V20H53V16Z" fill="#875B7B"/>
<path d="M53 21H55V27H53V21Z" fill="#875B7B"/>
<path d="M34.7313 43.5949C33.4391 43.5736 32.1821 43.4782 30.9604 43.3084C29.7386 43.1175 28.7518 42.8629 28 42.5447V39.8397C28.7988 40.1791 29.8209 40.4761 31.0661 40.7307C32.3113 40.9853 33.533 41.1232 34.7313 41.1444V34.716C33.58 34.419 32.5698 34.1008 31.7004 33.7613C30.8546 33.4007 30.1615 32.9976 29.6211 32.552C29.0808 32.1065 28.6696 31.5973 28.3877 31.0245C28.1292 30.4304 28 29.7515 28 28.9878C28 27.9482 28.2702 27.0571 28.8106 26.3146C29.3744 25.572 30.1615 24.9886 31.1718 24.5643C32.1821 24.1187 33.3686 23.8641 34.7313 23.8005V21H36.9868V23.7687C38.232 23.7899 39.3598 23.9172 40.37 24.1506C41.4038 24.3627 42.3436 24.6279 43.1894 24.9461L42.2379 27.3011C41.486 27.0253 40.652 26.7919 39.7357 26.601C38.8429 26.3888 37.9266 26.2509 36.9868 26.1873V32.5838C38.5374 32.9869 39.8297 33.4219 40.8634 33.8886C41.8972 34.3341 42.6725 34.8964 43.1894 35.5753C43.7298 36.233 44 37.0922 44 38.153C44 39.6381 43.3891 40.8474 42.1674 41.7809C40.9457 42.6932 39.2188 43.2554 36.9868 43.4676V47H34.7313V43.5949ZM36.9868 40.9853C38.373 40.858 39.3833 40.5716 40.0176 40.1261C40.652 39.6593 40.9692 39.0653 40.9692 38.3439C40.9692 37.8135 40.8517 37.3786 40.6167 37.0392C40.3818 36.6785 39.9706 36.3709 39.3833 36.1163C38.8194 35.8617 38.0206 35.6177 36.9868 35.3843V40.9853ZM34.7313 26.2509C33.8855 26.2934 33.1924 26.4313 32.652 26.6646C32.1116 26.8768 31.7004 27.1632 31.4185 27.5239C31.1601 27.8845 31.0308 28.2982 31.0308 28.765C31.0308 29.3166 31.1366 29.794 31.348 30.1971C31.583 30.5789 31.9706 30.9078 32.511 31.1836C33.0514 31.4382 33.7915 31.6716 34.7313 31.8837V26.2509Z" fill="#C0A5BD"/>
</svg>

```

## File: static\img\food.svg

```svg
<svg width="70" height="70" viewBox="0 0 70 70" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M43 31H62L61.1219 63.0274C61.1071 63.5688 60.6639 64 60.1223 64H44.8777C44.3361 64 43.8929 63.5688 43.8781 63.0274L43 31Z" fill="white" stroke="#875B7B" stroke-width="2"/>
<path d="M41 23C41 22.4477 41.4477 22 42 22H63C63.5523 22 64 22.4477 64 23V31H41V23Z" fill="white" stroke="#875B7B" stroke-width="2"/>
<path d="M52.7567 6.64888C52.903 6.25857 53.2762 6 53.693 6H55.557C56.2552 6 56.7385 6.69737 56.4933 7.35112L54.0637 13.8302C54.0216 13.9425 54 14.0614 54 14.1813V21C54 21.5523 53.5523 22 53 22H51C50.4477 22 50 21.5523 50 21V14.1813C50 14.0614 50.0216 13.9425 50.0637 13.8302L52.7567 6.64888Z" fill="#C0A5BD"/>
<path d="M50.0637 13.8302L51 14.1813L50.0637 13.8302ZM54.0637 13.8302L55 14.1813L54.0637 13.8302ZM53.693 7H55.557V5H53.693V7ZM53 21H51V23H53V21ZM55.557 7L53.1273 13.4791L55 14.1813L57.4297 7.70225L55.557 7ZM53 14.1813V21H55V14.1813H53ZM51 21V14.1813H49V21H51ZM51 14.1813L53.693 7L51.8203 6.29775L49.1273 13.4791L51 14.1813ZM51 14.1813L51 14.1813L49.1273 13.4791C49.0431 13.7036 49 13.9415 49 14.1813H51ZM53.1273 13.4791C53.0431 13.7036 53 13.9415 53 14.1813H55L55 14.1813L53.1273 13.4791ZM51 21V21H49C49 22.1046 49.8954 23 51 23V21ZM53 23C54.1046 23 55 22.1046 55 21H53V21V23ZM55.557 7L55.557 7L57.4297 7.70225C57.92 6.39475 56.9534 5 55.557 5V7ZM53.693 5C52.8593 5 52.1131 5.51714 51.8203 6.29775L53.693 7L53.693 7V5Z" fill="#875B7B"/>
<path d="M8 59H35V63C35 63.5523 34.5523 64 34 64H9C8.44771 64 8 63.5523 8 63V59Z" fill="white" stroke="#875B7B" stroke-width="2"/>
<rect x="8" y="55" width="27" height="4" rx="1" fill="#C0A5BD" stroke="#875B7B" stroke-width="2"/>
<path d="M21.5 36C28.8348 36 35 37.7845 35 46H8C8 37.7845 14.1652 36 21.5 36Z" fill="white" stroke="#875B7B" stroke-width="2"/>
<path d="M5 53L6.63826 50.9522C7.43891 49.9514 8.96109 49.9514 9.76174 50.9522L9.83826 51.0478C10.6389 52.0486 12.1611 52.0486 12.9617 51.0478L13.0383 50.9522C13.8389 49.9514 15.3611 49.9514 16.1617 50.9522L16.2383 51.0478C17.0389 52.0486 18.5611 52.0486 19.3617 51.0478L19.4383 50.9522C20.2389 49.9514 21.7611 49.9514 22.5617 50.9522L22.6383 51.0478C23.4389 52.0486 24.9611 52.0486 25.7617 51.0478L25.8383 50.9522C26.6389 49.9514 28.1611 49.9514 28.9617 50.9522L29.0383 51.0478C29.8389 52.0486 31.3611 52.0486 32.1617 51.0478L32.2383 50.9522C33.0389 49.9514 34.5611 49.9514 35.3617 50.9522L37 53" stroke="#875B7B" stroke-width="2" stroke-linecap="square"/>
</svg>

```

## File: static\img\gift.svg

```svg
<svg width="70" height="70" viewBox="0 0 70 70" fill="none" xmlns="http://www.w3.org/2000/svg">
<rect x="11" y="33" width="48" height="30" rx="1" fill="white" stroke="#875B7B" stroke-width="2"/>
<rect x="9" y="21" width="52" height="12" rx="1" fill="white" stroke="#875B7B" stroke-width="2"/>
<rect x="32" y="33" width="6" height="29" fill="#C0A5BD" stroke="#875B7B" stroke-width="2"/>
<rect x="31" y="21" width="8" height="12" fill="#C0A5BD" stroke="#875B7B" stroke-width="2"/>
<path d="M37 14C37 10.134 40.134 7 44 7V7C47.866 7 51 10.134 51 14V14C51 17.866 47.866 21 44 21H37V14Z" fill="white" stroke="#875B7B" stroke-width="2"/>
<path d="M33 14C33 10.134 29.866 7 26 7V7C22.134 7 19 10.134 19 14V14C19 17.866 22.134 21 26 21H33V14Z" fill="white" stroke="#875B7B" stroke-width="2"/>
<path d="M50.1214 44.7782C49.9339 44.5907 49.8285 44.3363 49.8285 44.0711L49.8285 39.8284C49.8285 39.2762 50.2762 38.8284 50.8285 38.8284L55.0711 38.8284C55.3364 38.8284 55.5907 38.9338 55.7783 39.1213L61.8493 45.1924C62.2398 45.5829 62.2398 46.2161 61.8493 46.6066L57.6067 50.8493C57.2162 51.2398 56.583 51.2398 56.1925 50.8493L50.1214 44.7782Z" fill="white"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M48.8284 39.8284C48.8284 38.7239 49.7238 37.8284 50.8284 37.8284L55.0711 37.8284C55.6015 37.8284 56.1102 38.0392 56.4853 38.4142L62.5563 44.4853C63.3374 45.2664 63.3374 46.5327 62.5563 47.3137L58.3137 51.5564C57.5326 52.3374 56.2663 52.3374 55.4853 51.5564L49.4142 45.4853C49.0391 45.1102 48.8284 44.6015 48.8284 44.0711L48.8284 39.8284ZM50.8284 39.8284L50.8284 44.0711L56.8995 50.1422L61.1421 45.8995L55.0711 39.8284L50.8284 39.8284Z" fill="#875B7B"/>
<path d="M43.179 33.5932L44.5932 32.179L53.7856 41.3714L52.3713 42.7856L43.179 33.5932Z" fill="#875B7B"/>
</svg>

```

## File: static\img\mileage.svg

```svg
<svg width="70" height="70" viewBox="0 0 70 70" fill="none" xmlns="http://www.w3.org/2000/svg">
<circle cx="35" cy="35" r="29" fill="white" stroke="#875B7B" stroke-width="2"/>
<path d="M34 24C34 23.4477 34.4477 23 35 23V23C35.5523 23 36 23.4477 36 24V35C36 35.5523 35.5523 36 35 36V36C34.4477 36 34 35.5523 34 35V24Z" fill="#875B7B"/>
<circle cx="35" cy="39" r="3" stroke="#875B7B" stroke-width="2"/>
<path d="M25 49C25 48.4477 25.4477 48 26 48H44C44.5523 48 45 48.4477 45 49V53C45 53.5523 44.5523 54 44 54H26C25.4477 54 25 53.5523 25 53V49Z" fill="#C0A5BD" stroke="#875B7B" stroke-width="2"/>
<path d="M34 14.0234C28.1407 14.2981 22.9093 16.974 19.2678 21.0893L21.1247 22.9463L19.9462 24.1248L18.2083 22.3869C15.5661 25.8989 14 30.2666 14 35H17V37H12V35C12 22.2975 22.2975 12 35 12C47.7025 12 58 22.2975 58 35V37H53V35H56C56 30.0425 54.2822 25.4863 51.4093 21.894L49.1785 24.1248L48 22.9463L50.3149 20.6314C46.6965 16.7762 41.6391 14.2878 36 14.0234V17H34V14.0234Z" fill="#875B7B"/>
</svg>

```

## File: static\img\other.svg

```svg
<svg width="70" height="70" viewBox="0 0 70 70" fill="none" xmlns="http://www.w3.org/2000/svg">
<g clip-path="url(#clip0_42_408)">
<path d="M40.4601 57.92V53.56C40.4601 53.56 44.8201 50.91 44.8201 45.92V26.27L27.0001 24.08L20.0001 29.78C19.2314 30.4959 18.6188 31.3628 18.2006 32.3264C17.7825 33.29 17.5678 34.3296 17.5701 35.38V57.92H40.4601Z" fill="white" stroke="#875B7B" stroke-width="2" stroke-miterlimit="10"/>
<path d="M41.5501 57.92H15.3501V65.56H41.5501V57.92Z" fill="white"/>
<path d="M15.3501 65.57V57.92H41.5501V65.57" fill="#C0A5BD"/>
<path d="M15.3501 65.57V57.92H41.5501V65.57" stroke="#875B7B" stroke-width="2" stroke-miterlimit="10"/>
<path d="M18.6299 62.29H21.8999" stroke="#875B7B" stroke-width="2" stroke-miterlimit="10"/>
<path d="M52.4701 4.42999H29.5401C28.961 4.43264 28.4065 4.66454 27.998 5.07495C27.5895 5.48537 27.3601 6.0409 27.3601 6.61999V39.37L30.6301 42.64H52.4701C53.0475 42.6374 53.6004 42.4068 54.0087 41.9986C54.417 41.5903 54.6475 41.0374 54.6501 40.46V6.61999C54.6501 6.0409 54.4208 5.48537 54.0122 5.07495C53.6037 4.66454 53.0492 4.43264 52.4701 4.42999V4.42999Z" fill="white" stroke="#875B7B" stroke-miterlimit="10"/>
<path d="M30.6301 42.64H52.4701C53.0475 42.6374 53.6004 42.4068 54.0087 41.9986C54.417 41.5903 54.6475 41.0374 54.6501 40.46V6.61999C54.6501 6.0409 54.4208 5.48537 54.0122 5.07495C53.6037 4.66454 53.0492 4.43264 52.4701 4.42999H29.5401C28.961 4.43264 28.4065 4.66454 27.998 5.07495C27.5895 5.48537 27.3601 6.0409 27.3601 6.61999V39.37" fill="white"/>
<path d="M30.6301 42.64H52.4701C53.0475 42.6374 53.6004 42.4068 54.0087 41.9986C54.417 41.5903 54.6475 41.0374 54.6501 40.46V6.61999C54.6501 6.0409 54.4208 5.48537 54.0122 5.07495C53.6037 4.66454 53.0492 4.43264 52.4701 4.42999H29.5401C28.961 4.43264 28.4065 4.66454 27.998 5.07495C27.5895 5.48537 27.3601 6.0409 27.3601 6.61999V39.37" stroke="#875B7B" stroke-width="2" stroke-miterlimit="10"/>
<path d="M49.1899 4.42999H42.6399V42.64H49.1899V4.42999Z" fill="#C0A5BD" stroke="#875B7B" stroke-width="2" stroke-miterlimit="10"/>
<path d="M38.27 7.70999V12.08" stroke="#875B7B" stroke-width="2" stroke-miterlimit="10"/>
<path d="M38.27 14.26V18.63" stroke="#875B7B" stroke-width="2" stroke-miterlimit="10"/>
<path d="M35.3399 26.53C32.8099 29.07 22.9199 38.74 22.9199 38.74L31.6399 50.13C32.7046 48.7314 33.3794 47.0755 33.5955 45.3311C33.8116 43.5866 33.5612 41.8161 32.8699 40.2C32.8699 40.2 37.2799 35.26 38.3599 33.77C40.7699 30.2 37.8799 24 35.3399 26.53Z" fill="white"/>
<path d="M22.9199 38.74C22.9199 38.74 32.8099 29.07 35.3399 26.53C37.8699 23.99 40.7699 30.2 38.3399 33.77C37.2599 35.26 32.8499 40.2 32.8499 40.2C33.5412 41.8161 33.7916 43.5866 33.5755 45.331C33.3594 47.0755 32.6846 48.7314 31.6199 50.13" fill="white"/>
<path d="M22.9199 38.74C22.9199 38.74 32.8099 29.07 35.3399 26.53C37.8699 23.99 40.7699 30.2 38.3399 33.77C37.2599 35.26 32.8499 40.2 32.8499 40.2C33.5412 41.8161 33.7916 43.5866 33.5755 45.331C33.3594 47.0755 32.6846 48.7314 31.6199 50.13" stroke="#875B7B" stroke-width="2" stroke-miterlimit="10"/>
</g>
<defs>
<clipPath id="clip0_42_408">
<rect width="70" height="70" fill="white"/>
</clipPath>
</defs>
</svg>

```

## File: static\img\transport.svg

```svg
<svg width="70" height="70" viewBox="0 0 70 70" fill="none" xmlns="http://www.w3.org/2000/svg">
<g clip-path="url(#clip0_42_450)">
<path d="M8.46992 15.2L8.41992 18.65L22.9499 26.39L30.5799 34.02L42.3699 22.05L8.46992 15.2Z" fill="#C0A5BD" stroke="#875B7B" stroke-width="2" stroke-miterlimit="10"/>
<path d="M47.95 27.63L35.98 39.42L43.61 47.05L51.35 61.58L54.8 61.52L47.95 27.63Z" fill="#C0A5BD" stroke="#875B7B" stroke-width="2" stroke-miterlimit="10"/>
<path d="M27.0899 49.7L28.6599 61.79L25.3899 63.28L19.6599 53.73" fill="white"/>
<path d="M27.0899 49.7L28.6599 61.79L25.3899 63.28L19.6599 53.73" stroke="#875B7B" stroke-width="2" stroke-miterlimit="10"/>
<path d="M20.3 42.91L8.20997 41.34L6.71997 44.61L16.27 50.34" fill="white"/>
<path d="M20.3 42.91L8.20997 41.34L6.71997 44.61L16.27 50.34" stroke="#875B7B" stroke-width="2" stroke-miterlimit="10"/>
<path d="M62.7301 7.27001C60.8701 5.40001 54.4201 8.79001 52.5501 10.66L22.3301 40.88C20.3301 42.88 11.5001 54.47 13.5101 56.49C15.5201 58.51 27.1001 49.69 29.1201 47.67L59.3401 17.45C61.2101 15.58 64.6001 9.13001 62.7301 7.27001Z" fill="white" stroke="#875B7B" stroke-width="2" stroke-miterlimit="10"/>
</g>
<defs>
<clipPath id="clip0_42_450">
<rect width="70" height="70" fill="white"/>
</clipPath>
</defs>
</svg>

```

## File: static\src\components\attachment_view.js

```javascript
/** @odoo-module **/

import { patch } from "@web/core/utils/patch";
import { AttachmentView } from "@mail/core/common/attachment_view";

patch(AttachmentView.prototype, {
    get displayName() {
        if (this.state.thread.model === 'hr.expense.sheet') {
            return (this.state.thread.mainAttachment.res_name || this.state.thread.name);
        }
        return super.displayName;
    }
});

```

## File: static\src\components\expense_dashboard.js

```javascript
/** @odoo-module */

import { useService } from '@web/core/utils/hooks';
import { formatMonetary } from "@web/views/fields/formatters";
import { Component, onWillStart, useState } from "@odoo/owl";

export class ExpenseDashboard extends Component {

    setup() {
        super.setup();
        this.orm = useService('orm');

        this.state = useState({
            expenses: {}
        });

        onWillStart(async () => {
            const expense_states = await this.orm.call("hr.expense", 'get_expense_dashboard', []);
            this.state.expenses = expense_states;
        });
    }

    renderMonetaryField(value, currency_id) {
        return formatMonetary(value, { currencyId: currency_id});;
    }
}
ExpenseDashboard.template = 'hr_expense.ExpenseDashboard';

```

## File: static\src\components\expense_dashboard.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="hr_expense.ExpenseDashboard">
        <div class="o_expense_container position-sticky start-0 d-flex o_form_statusbar">
            <t t-foreach="Object.entries(state.expenses)" t-as="expense" t-key="expense[0]">
                <t t-set="name" t-value="expense[0]"/>
                <t t-set="data" t-value="expense[1]"/>
                <div t-attf-class="o_expense_card o_arrow_button flex-grow-1 d-flex flex-column p-3 border-bottom text-center"
                     t-att-data-tooltip="data['tooltip']">
                    <span t-out="renderMonetaryField(data['amount'], data['currency'])" class="h2 m-0 text-primary"/>
                    <b class="mx-2" t-out="data['description']"/>
                </div>
                <div t-if="name !== 'approved'" t-attf-class="o_expense_card o_arrow_button flex-grow-1 d-flex flex-column p-3 border-bottom text-center">
                    <i class="fa fa-angle-right fa-3x"/>
                </div>
            </t>
            <div class="fa fa-question-circle-o flex-grow-0.5 d-flex flex-column p-3 border-bottom text-center" t-if="env.debug" data-tooltip="Numbers computed from your personal expenses."/>
        </div>
    </t>
</templates>

```

## File: static\src\components\nb_attachment.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { Component } from "@odoo/owl";

class AttachmentNumber extends Component {

    setup() {
        super.setup();
        this.nb_attachment = this.props.record.data.nb_attachment
    }
    static template = "hr_expense.AttachmentNumber"
}

registry.category("fields").add("nb_attachment", {component: AttachmentNumber});

```

## File: static\src\components\nb_attachment.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="hr_expense.AttachmentNumber">
        <div>
            <span class="fa fa-paperclip pe-1 align-middle"/>
            <t t-if="nb_attachment > 0" class="text-center" t-out="nb_attachment"/>
        </div>
    </t>
</templates>

```

## File: static\src\components\qrcode_action.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { Component } from "@odoo/owl";
import { sprintf } from "@web/core/utils/strings";


const actionRegistry = registry.category("actions");

class QRModalComponent extends Component {
    setup() {
        this.url = sprintf(
            "/report/barcode/?barcode_type=QR&value=%s&width=256&height=256&humanreadable=1",
            this.props.action.params.url);
    }
}

QRModalComponent.template = "hr_expense.QRModalComponent"

actionRegistry.add("expense_qr_code_modal", QRModalComponent);

```

## File: static\src\components\qrcode_action.xml

```xml
<?xml version="1.0"?>
<templates>
    <t t-name="hr_expense.QRModalComponent" owl="1">
        <div style="text-align:center;" class="o_expense_modal">
            <t t-if="url">
                <h3>Scan this QR code to get the Odoo app:</h3><br/><br/>
                <img class="border border-dark rounded" t-att-src="url"/>
            </t>
        </div>
    </t>
</templates>

```

## File: static\src\js\tours\hr_expense.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { stepUtils } from "@web_tour/tour_service/tour_utils";

import { markup } from "@odoo/owl";

registry.category("web_tour.tours").add('hr_expense_tour' , {
    url: "/web",
    rainbowManMessage: _t("There you go - expense management in a nutshell!"),
    steps: () => [stepUtils.showAppsMenuItem(), {
    trigger: '.o_app[data-menu-xmlid="hr_expense.menu_hr_expense_root"]',
    content: _t("Wasting time recording your receipts? Let’s try a better way."),
    position: 'right',
    edition: 'community'
}, {
    trigger: '.o_app[data-menu-xmlid="hr_expense.menu_hr_expense_root"]',
    content: _t("Wasting time recording your receipts? Let’s try a better way."),
    position: 'bottom',
    edition: 'enterprise'
}, {
    trigger: '.o_list_button_add',
    extra_trigger: '.o_button_upload_expense',
    content: _t("It all begins here - let's go!"),
    position: 'bottom',
    mobile: false,
}, {
    trigger: '.o-kanban-button-new',
    extra_trigger: '.o_button_upload_expense',
    content: _t("It all begins here - let's go!"),
    position: 'bottom',
    mobile: true,
}, {
    trigger: '.o_field_widget[name="product_id"] .o_input_dropdown',
    extra_trigger: '.o_hr_expense_form_view_view',
    content: _t("Enter a name then choose a category and configure the amount of your expense."),
    position: 'bottom',
}, {
    trigger: '.o_form_status_indicator_dirty .o_form_button_save',
    extra_trigger: '.o_hr_expense_form_view_view',
    content: markup(_t("Ready? You can save it manually or discard modifications from here. You don't <em>need to save</em> - Odoo will save eveyrthing for you when you navigate.")),
    position: 'bottom',
}, ...stepUtils.statusbarButtonsSteps(_t("Attach Receipt"), _t("Attach a receipt - usually an image or a PDF file.")),
...stepUtils.statusbarButtonsSteps(_t("Create Report"), _t("Create a report to submit one or more expenses to your manager.")),
...stepUtils.statusbarButtonsSteps(_t("Submit to Manager"), markup(_t('Once your <b>Expense Report</b> is ready, you can submit it to your manager and wait for approval.'))),
...stepUtils.goBackBreadcrumbsMobile(
    _t("Use the breadcrumbs to go back to the list of expenses."),
    undefined,
    ".o_hr_expense_form_view_view",
),
{
    trigger: '.breadcrumb > li.breadcrumb-item:first',
    extra_trigger: ".o_hr_expense_form_view_view",
    content: _t("Let's go back to your expenses."),
    position: 'bottom',
    mobile: false,
}, {
    trigger: '.o_expense_container',
    content: _t("The status of all your current expenses is visible from here."),
    position: 'bottom',
},
stepUtils.openBurgerMenu(),
{
    trigger: "[data-menu-xmlid='hr_expense.menu_hr_expense_report']",
    extra_trigger: '.o_main_navbar',
    content: _t("Let's check out where you can manage all your employees expenses"),
    position: "bottom"
}, {
    trigger: '.o_list_renderer tbody tr[data-id]',
    content: _t('Managers can inspect all expenses from here.'),
    position: 'bottom',
    mobile: false,
}, {
    trigger: '.o_kanban_renderer .oe_kanban_card',
    content: _t('Managers can inspect all expenses from here.'),
    position: 'bottom',
    mobile: true,
},
...stepUtils.statusbarButtonsSteps(_t("Approve"), _t("Managers can approve the report here, then an accountant can post the accounting entries.")),
]});

```

## File: static\src\js\tours\show_expense_receipt_tour.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { stepUtils } from "@web_tour/tour_service/tour_utils";

registry.category("web_tour.tours").add('show_expense_receipt_tour', {
    test: true,
    url: '/web',
    steps: () => [
        ...stepUtils.goToAppSteps('hr_expense.menu_hr_expense_root', "Go to the Expenses app"),
        {
            content: "Go to Expense Reports",
            trigger: '.dropdown-item[data-menu-xmlid="hr_expense.menu_hr_expense_report"]',
            position:'bottom',
        },
        {
            content: "Go to a report",
            trigger: '.o_data_row .o_data_cell[name="payment_state"]',
        },
        {
            content: "Click on an expense line 2",
            trigger: '.o_data_row .o_data_cell[data-tooltip="expense_2"]',
        },
        {
            content: "Check attachment",
            trigger: ".o_attachment_preview .o-mail-Attachment-imgContainer .img[src*='test_file_2.png']",
        },
        {
            content: "Click on an expense line 3",
            trigger: '.o_data_row .o_data_cell[data-tooltip="expense_3"]',
        },
        {
            content: "Check attachment",
            trigger: ".o_attachment_preview .o-mail-Attachment-imgContainer .img[src*='test_file_3.png']",
        },
        {
            content: "Click on an expense line 1",
            trigger: '.o_data_row .o_data_cell[data-tooltip="expense_1"]',
        },
        {
            content: "Check attachment",
            trigger: ".o_attachment_preview .o-mail-Attachment-imgContainer .img[src*='test_file_1.png']",
        },
    ],
});

```

## File: static\src\mixins\document_upload.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { useBus, useService } from '@web/core/utils/hooks';
import { useRef, useEffect, useState } from "@odoo/owl";

export const ExpenseDocumentDropZone = (T) => class ExpenseDocumentDropZone extends T {
    setup() {
        super.setup();
        this.dragState = useState({
            showDragZone: false,
        });
        this.root = useRef("root");

        useEffect(
            (el) => {
                if (!el) {
                    return;
                }
                const highlight = this.highlight.bind(this);
                const unhighlight = this.unhighlight.bind(this);
                const drop = this.onDrop.bind(this);
                el.addEventListener("dragover", highlight);
                el.addEventListener("dragleave", unhighlight);
                el.addEventListener("drop", drop);
                return () => {
                    el.removeEventListener("dragover", highlight);
                    el.removeEventListener("dragleave", unhighlight);
                    el.removeEventListener("drop", drop);
                };
            },
            () => [document.querySelector('.o_content')]
        );
    }

    highlight(ev) {
        ev.stopPropagation();
        ev.preventDefault();
        this.dragState.showDragZone = true;
    }

    unhighlight(ev) {
        ev.stopPropagation();
        ev.preventDefault();
        this.dragState.showDragZone = false;
    }

    async onDrop(ev) {
        ev.preventDefault();
        await this.env.bus.trigger("change_file_input", {
            files: ev.dataTransfer.files,
        });        
    }
};

export const ExpenseDocumentUpload = (T) => class ExpenseDocumentUpload extends T {
    setup() {
        super.setup();
        this.actionService = useService('action');
        this.notification = useService('notification');
        this.orm = useService('orm');
        this.http = useService('http');
        this.fileInput = useRef('fileInput');
        this.root = useRef("root");
        this.isExpenseSheet = this.model.config.resModel === "hr.expense.sheet";

        useBus(this.env.bus, "change_file_input", async (ev) => {
            this.fileInput.el.files = ev.detail.files;
            await this.onChangeFileInput();
        });
    }

    displayCreateReport() {
        const records = this.model.root.selection;
        return !this.isExpenseSheet && (records.length === 0 || records.some(record => record.data.state === "draft"))
    }

    async action_show_expenses_to_submit () {
        const records = this.model.root.selection;
        const res = await this.orm.call(this.model.config.resModel, 'get_expenses_to_submit', [records.map((record) => record.resId)]);
        if (res) {
            await this.actionService.doAction(res, {});
        }
    }

    uploadDocument() {
        this.fileInput.el.click();
    }

    async onChangeFileInput() {
        const params = {
            csrf_token: odoo.csrf_token,
            ufile: [...this.fileInput.el.files],
            model: 'hr.expense',
            id: 0,
        };

        const fileData = await this.http.post('/web/binary/upload_attachment', params, "text");
        const attachments = JSON.parse(fileData);
        if (attachments.error) {
            throw new Error(attachments.error);
        }
        this.onUpload(attachments);
    }

    async onUpload(attachments) {
        const attachmentIds = attachments.map((a) => a.id);
        if (!attachmentIds.length) {
            this.notification.add(
                _t('An error occurred during the upload')
            );
            return;
        }

        const action = await this.orm.call('hr.expense', 'create_expense_from_attachments', ["", attachmentIds]);
        this.actionService.doAction(action);
    }
};

```

## File: static\src\mixins\qrcode.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { useService } from "@web/core/utils/hooks";
import { onMounted, onPatched, useRef } from "@odoo/owl";

export const ExpenseMobileQRCode = (T) => class ExpenseMobileQRCode extends T {
    setup() {
        super.setup();
        this.root = useRef('root');
        this.actionService = useService('action');

        onMounted(this.bindAppsIcons);
        onPatched(this.bindAppsIcons);
    }

    bindAppsIcons() {
        const apps = this.root.el.querySelectorAll('.o_expense_mobile_app');
        if (!apps) {
            return;
        }

        const handler = this.handleClick.bind(this);
        for (const app of apps) {
            app.addEventListener('click', handler);
        }
    }

    handleClick(ev) {
        ev.preventDefault();
        ev.stopPropagation();

        const url = ev.currentTarget && ev.currentTarget.href;
        if (!this.env.isSmall) {
            this.actionService.doAction({
                name: _t("Download our App"),
                type: "ir.actions.client",
                tag: 'expense_qr_code_modal',
                target: "new",
                params: { url },
            });
        } else {
            this.actionService.doAction({ type: "ir.actions.act_url", url });
        }
    }
};

```

## File: static\src\views\expense_form_view.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { useService } from "@web/core/utils/hooks";
import { registry } from "@web/core/registry";
import { FormController } from "@web/views/form/form_controller";
import { formView } from "@web/views/form/form_view";

import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";

export class ExpenseFormController extends FormController {
    setup() {
        super.setup();
        this.dialogService = useService("dialog");
        this.orm = useService("orm");
    }

    /**
     * @override
     */
    async beforeExecuteActionButton(clickParams) {
        const record = this.model.root;
        if (
            clickParams.name === "action_submit_expenses" &&
            record.data.duplicate_expense_ids.count
        ) {
            return new Promise((resolve) => {
                this.dialogService.add(ConfirmationDialog, {
                    body: _t("An expense of same category, amount and date already exists."),
                    confirm: async () => {
                        await this.orm.call("hr.expense", "action_approve_duplicates", [record.resId]);
                        resolve(true);
                    },
                }, {
                    onClose: resolve.bind(null, false),
                });
            });
        }
        return super.beforeExecuteActionButton(...arguments);
    }
}

export const ExpenseFormView = {
    ...formView,
    Controller: ExpenseFormController,
};

registry.category("views").add("hr_expense_form_view", ExpenseFormView);

```

## File: static\src\views\expense_line_widget.js

```javascript
/** @odoo-module */

import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";

import { ListRenderer } from "@web/views/list/list_renderer";
import { X2ManyField, x2ManyField } from "@web/views/fields/x2many/x2many_field";

export class ExpenseLinesListRenderer extends ListRenderer {
    setup() {
        super.setup();
        this.threadService = useService("mail.thread");

        this.sheetId = this.env.model.root.resId;
        this.sheetThread = this.threadService.getThread('hr.expense.sheet', this.sheetId);
    }

    /** @override **/
    async onCellClicked(record, column, ev) {
        const attachmentChecksum = record.data.message_main_attachment_checksum;

        if (attachmentChecksum && this.sheetThread.mainAttachment?.checksum !== attachmentChecksum) {
            this.sheetThread.update({ mainAttachment: this.sheetThread.attachments.find((attachment) => attachment.checksum === attachmentChecksum) });
        }
        super.onCellClicked(record, column, ev);
    }
}

export class ExpenseLinesWidget extends X2ManyField {
    static components = {
        ...X2ManyField.components,
        ListRenderer: ExpenseLinesListRenderer,
    };

    setup() {
        super.setup();
        this.canOpenRecord = false;
    }

    get isMany2Many() {
        // The field is used like a many2many to allow for adding existing lines to the sheet.
        return true;
    }
}

export const expenseLinesWidget = {
    ...x2ManyField,
    component: ExpenseLinesWidget,
    relatedFields: [{ name: "message_main_attachment_checksum", type: "char" }],
    additionalClasses: ["o_field_many2many"],
};

registry.category("fields").add("expense_lines_widget", expenseLinesWidget);

```

## File: static\src\views\kanban.js

```javascript
/** @odoo-module */

import { registry } from '@web/core/registry';

import { ExpenseDashboard } from '../components/expense_dashboard';
import { ExpenseMobileQRCode } from '../mixins/qrcode';
import { ExpenseDocumentUpload, ExpenseDocumentDropZone } from '../mixins/document_upload';

import { kanbanView } from '@web/views/kanban/kanban_view';
import { KanbanController } from '@web/views/kanban/kanban_controller';
import { KanbanRenderer } from '@web/views/kanban/kanban_renderer';

export class ExpenseKanbanController extends ExpenseDocumentUpload(KanbanController) {}

export class ExpenseKanbanRenderer extends ExpenseDocumentDropZone(ExpenseMobileQRCode(KanbanRenderer)) {}
ExpenseKanbanRenderer.template = 'hr_expense.KanbanRenderer';

export class ExpenseDashboardKanbanRenderer extends ExpenseKanbanRenderer {}
ExpenseDashboardKanbanRenderer.components = { ...ExpenseDashboardKanbanRenderer.components, ExpenseDashboard};
ExpenseDashboardKanbanRenderer.template = 'hr_expense.DashboardKanbanRenderer';

registry.category('views').add('hr_expense_kanban', {
    ...kanbanView,
    buttonTemplate: 'hr_expense.KanbanButtons',
    Controller: ExpenseKanbanController,
    Renderer: ExpenseKanbanRenderer,
});

registry.category('views').add('hr_expense_dashboard_kanban', {
    ...kanbanView,
    buttonTemplate: 'hr_expense.KanbanButtons',
    Controller: ExpenseKanbanController,
    Renderer: ExpenseDashboardKanbanRenderer,
});

```

## File: static\src\views\kanban.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="hr_expense.KanbanRenderer" t-inherit="web.KanbanRenderer" t-inherit-mode="primary">
        <xpath expr="//div[hasclass('o_kanban_renderer')]" position="attributes">
            <attribute name="class" add="position-relative h-auto" separator=" "/>
        </xpath>
        <xpath expr="//div[hasclass('o_kanban_renderer')]" position="before">
            <div t-if="dragState.showDragZone" class="o_dropzone">
                <i class="fa fa-upload fa-10x"></i>
            </div>
        </xpath>
    </t>

    <t t-name="hr_expense.DashboardKanbanRenderer" t-inherit="hr_expense.KanbanRenderer" t-inherit-mode="primary">
        <xpath expr="//div[hasclass('o_kanban_renderer')]" position="before">
            <ExpenseDashboard/>
        </xpath>
    </t>

    <t t-name="hr_expense.KanbanButtons" t-inherit="web.KanbanView.Buttons" t-inherit-mode="primary">
        <xpath expr="//div[hasclass('o_cp_buttons')]" position="attributes">
            <attribute name="class" remove="d-flex" separator=" "/>
            <attribute name="class" add="d-block d-xl-flex" separator=" "/>
        </xpath>
        <xpath expr="//div[hasclass('o_cp_buttons')]" position="inside">
            <input type="file" name="ufile" class="d-none" t-ref="fileInput" multiple="1" accept="*" t-on-change="onChangeFileInput" />
            <button type="button" class="d-none d-md-inline o_button_upload_expense btn btn-primary" t-on-click.prevent="uploadDocument">
                Upload
            </button>
            <button type="button" class="d-inline d-md-none o_button_upload_expense btn btn-primary" t-on-click.prevent="uploadDocument">
                Scan
            </button>
            <button t-if="displayCreateReport()" class="btn btn-secondary" t-on-click="() => this.action_show_expenses_to_submit()">
                Create Report
            </button>
        </xpath>
    </t>
</templates>

```

## File: static\src\views\list.js

```javascript
/** @odoo-module */

import { ExpenseDashboard } from '../components/expense_dashboard';
import { ExpenseMobileQRCode } from '../mixins/qrcode';
import { ExpenseDocumentUpload, ExpenseDocumentDropZone } from '../mixins/document_upload';

import { registry } from '@web/core/registry';
import { useService } from '@web/core/utils/hooks';
import { listView } from "@web/views/list/list_view";

import { ListController } from "@web/views/list/list_controller";
import { ListRenderer } from "@web/views/list/list_renderer";
import { onWillStart } from "@odoo/owl";

export class ExpenseListController extends ExpenseDocumentUpload(ListController) {
    setup() {
        super.setup();
        this.orm = useService('orm');
        this.actionService = useService('action');
        this.rpc = useService("rpc");
        this.user = useService("user");
        this.isExpenseSheet = this.model.config.resModel === "hr.expense.sheet";

        onWillStart(async () => {
            this.userIsExpenseTeamApprover = await this.user.hasGroup("hr_expense.group_hr_expense_team_approver");
            this.userIsAccountInvoicing = await this.user.hasGroup("account.group_account_invoice");
        });
    }

    displaySubmit() {
        const records = this.model.root.selection;
        return records.length && records.every(record => record.data.state === 'draft') && this.isExpenseSheet;
    }

    displayApprove() {
        const records = this.model.root.selection;
        return this.userIsExpenseTeamApprover && records.length && records.every(record => record.data.state === 'submit') && this.isExpenseSheet;
    }

    displayPost() {
        const records = this.model.root.selection;
        return this.userIsAccountInvoicing && records.length && records.every(record => record.data.state === 'approve') && this.isExpenseSheet;
    }

    displayPayment() {
        const records = this.model.root.selection;
        return this.userIsAccountInvoicing && records.length && records.every(record => record.data.state === 'post' && record.data.payment_state === 'not_paid') && this.isExpenseSheet;
    }

    async onClick (action) {
        const records = this.model.root.selection;
        const recordIds = records.map((a) => a.resId);
        const model = this.model.config.resModel;
        const context = {};
        if (action === 'action_approve_expense_sheets') {
            context['validate_analytic'] = true;
        }
        const res = await this.orm.call(model, action, [recordIds], {context: context});
        if (res) {
            await this.actionService.doAction(res, {
                additionalContext: {
                    dont_redirect_to_payments: 1,
                },
                onClose: async () => {
                    await this.model.root.load();
                    this.render(true);
                }
            });
        }
        await this.model.root.load();
    }
}

export class ExpenseListRenderer extends ExpenseDocumentDropZone(ExpenseMobileQRCode(ListRenderer)) {}
ExpenseListRenderer.template = 'hr_expense.ListRenderer';

export class ExpenseDashboardListRenderer extends ExpenseListRenderer {}

ExpenseDashboardListRenderer.components = { ...ExpenseDashboardListRenderer.components, ExpenseDashboard};
ExpenseDashboardListRenderer.template = 'hr_expense.DashboardListRenderer';

registry.category('views').add('hr_expense_tree', {
    ...listView,
    buttonTemplate: 'hr_expense.ListButtons',
    Controller: ExpenseListController,
    Renderer: ExpenseListRenderer,
});

registry.category('views').add('hr_expense_dashboard_tree', {
    ...listView,
    buttonTemplate: 'hr_expense.ListButtons',
    Controller: ExpenseListController,
    Renderer: ExpenseDashboardListRenderer,
});

```

## File: static\src\views\list.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="hr_expense.ListButtons" t-inherit="web.ListView.Buttons" t-inherit-mode="primary">

       <!-- hr.expense and hr.expense.sheet -->
        <xpath expr="//div[hasclass('o_list_buttons')]" position="attributes">
            <attribute name="class" remove="d-flex" separator=" "/>
            <attribute name="class" add="d-block d-xl-flex" separator=" "/>
        </xpath>
        <xpath expr="//div[hasclass('o_list_buttons')]" position="inside">
            <input type="file" name="ufile" class="d-none" t-ref="fileInput" multiple="1" accept="*" t-on-change="onChangeFileInput"/>
            <button type="button" class="d-none d-md-inline o_button_upload_expense btn btn-primary" t-on-click.prevent="uploadDocument">
                Upload
            </button>
           <button type="button" class="d-inline d-md-none o_button_upload_expense btn btn-primary" t-on-click.prevent="uploadDocument">
                Scan
            </button>
            <button t-if="displayCreateReport()" class="btn btn-secondary" t-on-click="() => this.action_show_expenses_to_submit()">
                Create Report
            </button>
            <button t-if="displaySubmit()" class="d-none d-md-block btn btn-secondary" t-on-click="() => this.onClick('action_submit_sheet')">
                Submit
            </button>
            <button t-if="displayApprove()" class="d-none d-md-block btn btn-secondary" t-on-click="() => this.onClick('action_approve_expense_sheets')">
                Approve Report
            </button>
            <button t-if="displayPost()" class="d-none d-md-block btn btn-secondary" t-on-click="() => this.onClick('action_sheet_move_create')">
                Post Entries
            </button>
            <button t-if="displayPayment()" class="d-none d-md-block btn btn-secondary" t-on-click="() => this.onClick('action_register_payment')">
                Register Payment
            </button>
        </xpath>
    </t>

    <t t-name="hr_expense.ListRenderer" t-inherit="web.ListRenderer" t-inherit-mode="primary">
        <xpath expr="//div[hasclass('o_list_renderer')]" position="before">
            <div t-if="dragState.showDragZone" class="o_dropzone">
                <i class="fa fa-upload fa-10x"></i>
            </div>
        </xpath>
        <xpath expr="//div[hasclass('o_list_renderer')]" position="attributes">
            <attribute name="class">o_list_renderer hr_expense h-auto o_forbidden_tooltip_parent</attribute>
        </xpath>
    </t>


    <t t-name="hr_expense.DashboardListRenderer" t-inherit="hr_expense.ListRenderer" t-inherit-mode="primary">
        <xpath expr="//div[hasclass('o_list_renderer')]" position="before">
            <ExpenseDashboard/>
        </xpath>
    </t>
</templates>

```

## File: views\account_journal_dashboard.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="account_journal_dashboard_kanban_view_inherit_hr_expense" model="ir.ui.view">
        <field name="name">account.journal.dashboard.kanban</field>
        <field name="model">account.journal</field>
        <field name="inherit_id" ref="account.account_journal_dashboard_kanban_view" />
        <field name="arch" type="xml">
            <xpath expr="//t[@id='account.JournalBodySalePurchase']//div[hasclass('o_kanban_primary_right')]" position="inside">
                <div class="row" t-if="dashboard.number_expenses_to_pay">
                    <div class="col overflow-hidden text-start">
                        <a type="object" t-if="journal_type == 'purchase'" name="open_expenses_action">
                            <t t-out="dashboard.number_expenses_to_pay"/> Expenses to Process
                        </a>
                    </div>
                    <div class="col-auto text-end">
                        <span t-if="journal_type == 'purchase'"><t t-out="dashboard.sum_expenses_to_pay"/></span>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_move_form_inherit_expense" model="ir.ui.view">
            <field name="name">account.move.form.inherit</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_move_form"/>
            <field name="arch" type="xml">
                <xpath expr="//div[@name='button_box']" position="inside">
                    <field name="expense_sheet_id" invisible="1"/>
                    <button name="action_open_expense_report"
                            class="oe_stat_button"
                            icon="fa-file-text-o"
                            type="object"
                            invisible="not expense_sheet_id">
                            <div class="o_stat_info">
                                <span class="o_stat_text">Expense Report</span>
                            </div>
                    </button>
                </xpath>

                <xpath expr="//sheet" position="before">
                    <field name="show_commercial_partner_warning" invisible="1"/>
                    <div class="alert alert-warning" role="alert"
                         invisible="not show_commercial_partner_warning">
                        Do you really want to invoice your own company? Remove the "Company Name" from the partner to fix the configuration. Cancel this invoice and start again.
                    </div>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\account_payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_payment_form_inherit_expense" model="ir.ui.view">
            <field name="name">account.payment.form.inherit</field>
            <field name="model">account.payment</field>
            <field name="inherit_id" ref="account.view_account_payment_form"/>
            <field name="arch" type="xml">
                <xpath expr="//div[@name='button_box']" position="inside">
                    <field name="expense_sheet_id" invisible="1"/>
                    <button name="action_open_expense_report"
                            class="oe_stat_button"
                            icon="fa-file-text-o"
                            type="object"
                            invisible="not expense_sheet_id">
                            <div class="o_stat_info">
                                <span class="o_stat_text">Expense Report</span>
                            </div>
                    </button>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\hr_department_views.xml

```xml
<odoo>
    <!--Hr Department Inherit Kanban view-->
    <record id="hr_department_view_kanban" model="ir.ui.view">
        <field name="name">hr.department.kanban.inherit</field>
        <field name="model">hr.department</field>
        <field name="inherit_id" ref="hr.hr_department_view_kanban"/>
        <field name="arch" type="xml">
            <data>
                <xpath expr="//templates" position="before">
                    <field name="expense_sheets_to_approve_count" groups="hr_expense.group_hr_expense_team_approver"/>
                </xpath>

                <xpath expr="//div[hasclass('o_kanban_primary_right')]" position="inside">
                    <div t-if="record.expense_sheets_to_approve_count.raw_value > 0" class="row ml16" groups="hr_expense.group_hr_expense_team_approver">
                        <div class="col">
                            <a name="%(action_hr_expense_sheet_department_to_approve)d" type="action">
                                <t t-out="record.expense_sheets_to_approve_count.raw_value"/> Expense Reports
                            </a>
                        </div>
                    </div>
                </xpath>

                <xpath expr="//div[hasclass('o_kanban_manage_reports')]" position="inside">
                    <a role="menuitem" class="dropdown-item" name="%(action_hr_expense_sheet_department_filtered)d"
                       type="action" groups="hr_expense.group_hr_expense_team_approver">
                        Expenses
                    </a>
                </xpath>
            </data>
        </field>
    </record>
</odoo>

```

## File: views\hr_expense_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record id="hr_employee_view_form_inherit_expense" model="ir.ui.view">
            <field name="name">hr.employee.view.form.expense</field>
            <field name="model">hr.employee</field>
            <field name="inherit_id" ref="hr.view_employee_form"/>
            <field name="arch" type="xml">
                <xpath expr="//group[@name='managers']" position="inside">
                    <field name="expense_manager_id" context="{'default_company_id': company_id}" widget="many2one_avatar_user"/>
                </xpath>
                 <xpath expr="//group[@name='managers']" position="attributes">
                    <attribute name="invisible">0</attribute>
                </xpath>
            </field>
        </record>

        <record id="view_employee_tree_inherit_expense" model="ir.ui.view">
            <field name="name">hr.employee.tree.expense</field>
            <field name="model">hr.employee</field>
            <field name="inherit_id" ref="hr.view_employee_tree"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='work_location_id']" position="after">
                    <field name="expense_manager_id" optional="hide" string="Expense Approver" widget="many2one_avatar_user"/>
                </xpath>
            </field>
        </record>

        <record id="res_users_view_form_preferences" model="ir.ui.view">
            <field name="name">hr.user.preferences.form.inherit.hr.expense</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="hr.res_users_view_form_profile" />
            <field name="arch" type="xml">
                <xpath expr="//group[@name='managers']" position="inside">
                    <field name="expense_manager_id" readonly="not can_edit" context="{'default_company_id': company_id}"/>
                </xpath>
                 <xpath expr="//group[@name='managers']" position="attributes">
                    <attribute name="invisible">0</attribute>
                </xpath>
            </field>
        </record>

        <record id="hr_expense_view_expenses_analysis_tree" model="ir.ui.view">
            <field name="name">hr.expense.tree</field>
            <field name="model">hr.expense</field>
            <field name="arch" type="xml">
                <tree string="Expenses" multi_edit="1" sample="1" js_class="hr_expense_tree" decoration-info="state == 'draft'">
                    <field name="is_editable" column_invisible="True"/>
                    <field name="company_id" column_invisible="True"/>
                    <field name="company_currency_id" column_invisible="True"/>
                    <field name="nb_attachment" column_invisible="True"/>
                    <field name="is_multiple_currency" column_invisible="True"/>
                    <field name="product_has_cost" column_invisible="True"/>
                    <field name="date" optional="show" readonly="not is_editable"/>
                    <field name="product_id" optional="hide" readonly="not is_editable"/>
                    <field name="name" readonly="not is_editable"/>
                    <field name="employee_id" widget="many2one_avatar_user" readonly="not is_editable"/>
                    <field name="sheet_id" optional="show" readonly="True" column_invisible="not context.get('show_report', False)"/>
                    <field name="payment_mode" optional="show" readonly="not is_editable"/>
                    <field name="activity_ids" widget="list_activity" optional="show"/>
                    <field name="accounting_date" optional="hide" groups="account.group_account_invoice,account.group_account_readonly"
                           readonly="not is_editable"/>
                    <field name="analytic_distribution" widget="analytic_distribution"
                           optional="show"
                           groups="analytic.group_analytic_accounting"
                           readonly="not is_editable"
                           options="{'product_field': 'product_id', 'business_domain': 'expense'}"/>
                    <field name="account_id" optional="hide" groups="account.group_account_readonly"
                           readonly="not is_editable"/>
                    <field name="company_id" optional="show" groups="base.group_multi_company" readonly="True"/>
                    <field name="price_unit" string="Unit Price" optional="hide" widget="monetary"
                           options="{'currency_field': 'company_currency_id', 'field_digits': True}" readonly="True"/>
                    <field name="quantity" optional="hide" readonly="not is_editable or not product_has_cost"/>
                    <field name="tax_ids" optional="hide" widget="many2many_tags"
                           groups="account.group_account_invoice,account.group_account_readonly"
                           readonly="not is_editable or not product_has_cost"/>
                    <field name="tax_amount" sum="Total Taxes" readonly="True"
                           optional="hide" groups="account.group_account_invoice,account.group_account_readonly"/>
                    <field name="nb_attachment" widget="nb_attachment" nolabel="1" readonly="True"/>
                    <field name="total_amount" sum="Total Amount" widget='monetary'
                           readonly="not is_editable or product_has_cost"
                           options="{'currency_field': 'company_currency_id'}" decoration-bf="1"/>
                    <field name="total_amount_currency" widget='monetary'
                           readonly="not is_editable or not is_multiple_currency or product_has_cost"
                           options="{'currency_field': 'currency_id'}" optional="hide" decoration-bf="1"
                           groups="base.group_multi_currency"/>
                    <field name="currency_id" optional="hide" readonly="True" groups="base.group_multi_currency"/>
                    <field name="state" optional="show" readonly="True" decoration-info="state in ['draft', 'reported']"
                           decoration-success="state in ['approved', 'done']"
                           decoration-warning="state == 'submitted'" decoration-danger="state == 'refused'" widget="badge"/>
                </tree>
            </field>
        </record>

        <record id="view_expenses_tree" model="ir.ui.view">
            <field name="name">hr.expense.tree</field>
            <field name="model">hr.expense</field>
            <field name="inherit_id" ref="hr_expense_view_expenses_analysis_tree"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//tree" position="attributes">
                    <!-- Display the tree dashboard view with the header -->
                    <attribute name="js_class">hr_expense_dashboard_tree</attribute>
                </xpath>
            </field>
        </record>

        <record id="view_my_expenses_tree" model="ir.ui.view">
            <field name="name">hr.expense.tree</field>
            <field name="model">hr.expense</field>
            <field name="priority">20</field>
            <field name="inherit_id" ref="hr_expense.view_expenses_tree"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//tree" position="attributes">
                    <!-- Display the tree dashboard view with the header -->
                    <attribute name="js_class">hr_expense_dashboard_tree</attribute>
                </xpath>
            </field>
        </record>

        <record id="hr_expense_view_form" model="ir.ui.view">
            <field name="name">hr.expense.view.form</field>
            <field name="model">hr.expense</field>
            <field name="arch" type="xml">
                <form string="Expenses" js_class="hr_expense_form_view">
                <header>
                  <button name="action_submit_expenses" string="Create Report" type="object"
                          class="oe_highlight o_expense_submit" invisible="nb_attachment &lt;= 0 or sheet_id" data-hotkey="v"/>
                  <button name="action_view_sheet" type="object" string="View Report" class="oe_highlight" invisible="not sheet_id or nb_attachment &lt; 1" data-hotkey="w"/>
                  <widget name="attach_document" string="Attach Receipt" action="attach_document" invisible="nb_attachment &lt; 1"/>
                  <widget name="attach_document" string="Attach Receipt" action="attach_document" highlight="1" invisible="nb_attachment &gt;= 1"/>
                  <button name="action_submit_expenses" string="Create Report" type="object" class="o_expense_submit"
                          invisible="nb_attachment &gt;= 1 or sheet_id" data-hotkey="v"/>
                  <field name="state" widget="statusbar" statusbar_visible="draft,reported,submitted,approved,done"
                         invisible="state == 'refused'"/>
                  <field name="state" widget="statusbar" statusbar_visible="draft,reported,submitted,refused"
                         invisible="state != 'refused'"/>
                  <button name="action_view_sheet" type="object" string="View Report" class="oe_highlight" invisible="not sheet_id or nb_attachment &gt;= 1" data-hotkey="w"/>
                  <button name="action_split_wizard" string="Split Expense" type="object" invisible="sheet_id or product_has_cost"/>
                </header>
                <sheet>
                    <div class="oe_title">
                        <label for="name"/>
                        <h1>
                            <field name="name" placeholder="e.g. Lunch with Customer" readonly="not is_editable"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="sheet_id" invisible="1"/>
                            <field name="product_has_cost" invisible="1"/>
                            <field name="product_has_tax" invisible="1"/>
                            <field name="is_multiple_currency" invisible="1"/>
                            <field name="is_editable" invisible="1"/>
                            <field name="currency_id" invisible="1"/>
                            <field name="company_id" invisible="1"/>
                            <field name="company_currency_id" invisible="1"/>
                            <field name="tax_amount" invisible="1"/>
                            <field name="price_unit" invisible="1"/>
                            <field name="nb_attachment" invisible="1"/>
                            <field name="total_amount" invisible="1"/>
                            <field name="duplicate_expense_ids" invisible="1"/>
                            <field name="currency_rate" invisible="1"/>
                            <field name="product_uom_category_id" invisible="1"/>

                            <label for="product_id"/>
                            <div>
                                <field name="product_id" required="1" readonly="not is_editable"
                                       context="{'default_detailed_type': 'service', 'default_can_be_expensed': 1, 'tree_view_ref': 'hr_expense.product_product_expense_tree_view', 'form_view_ref': 'hr_expense.product_product_expense_form_view'}"
                                       class="w-100"/>
                                <div class="fst-italic" invisible="not is_editable or not product_description or not product_id">
                                    <field name="product_description"/>
                                </div>
                            </div>

                            <!-- CASE: product has a cost defined -> user input qty (always in company currency) -->
                            <field name="price_unit" required="1" widget="monetary"
                                   options="{'currency_field': 'currency_id', 'field_digits': True}"
                                   invisible="not product_has_cost" readonly="True"/>
                            <label for="quantity" invisible="not product_has_cost"/>
                            <div invisible="not product_has_cost">
                                <div class="o_row">
                                    <field name="quantity" class="oe_inline" readonly="not is_editable"/>
                                    <field name="product_uom_id" required="1" force_save="1"
                                           options="{'no_open': True, 'no_create': True}" groups="uom.group_uom"
                                           readonly="not is_editable"/>
                                </div>
                            </div>

                            <!-- CASE: product has no cost defined -> user input amount (in other currency if multi-currency) -->
                            <label for="total_amount_currency" string="Total" invisible="product_has_cost" readonly="not is_editable"/>
                            <div class="o_row" invisible="product_has_cost">
                                <field name="total_amount_currency" widget='monetary' options="{'currency_field': 'currency_id'}"
                                       readonly="not is_editable" class="oe_inline mw-50 me-0"/>
                                <field name="currency_id" class="mw-25 ms-0" groups="base.group_multi_currency"
                                       options="{'no_create': True}"
                                       readonly="not is_editable"/>
                            </div>

                            <!-- CASE: converter when currency is different than the company one -->
                            <label name="total_amount_product_cost_label" for="total_amount" string="Total" invisible="is_multiple_currency or not product_has_cost"/>
                            <label name="total_amount_multicurrency_label" for="total_amount" string="" invisible="not is_multiple_currency"/>
                            <div class="o_row" invisible="not is_multiple_currency and not product_has_cost">
                                <field name="total_amount" widget='monetary' options="{'currency_field': 'company_currency_id'}"
                                       force_save="1" readonly="not is_editable or product_has_cost" class="oe_inline"/>
                                <field name="label_currency_rate" class="ps-0"/>
                            </div>

                            <label for="tax_ids"/>
                            <div class="o_row">
                                <field name="tax_ids"
                                       force_save="1"
                                       widget="many2many_tags"
                                       readonly="not is_editable"
                                       options="{'no_create': True}"/>
                                <field name="tax_amount_currency"/>
                            </div>
                            <t groups="hr_expense.group_hr_expense_team_approver">
                                <field name="employee_id" groups="!hr.group_hr_user"
                                       context="{'default_company_id': company_id}" widget="many2one_avatar_employee"
                                       options="{'relation': 'hr.employee.public', 'no_create': True}"
                                       readonly="not is_editable"/>
                                <field name="employee_id" groups="hr.group_hr_user"
                                       context="{'default_company_id': company_id}" widget="many2one_avatar_employee"
                                       options="{'relation': 'hr.employee', 'no_create': True}"
                                   readonly="not is_editable"/>
                            </t>
                            <label id="lo" for="payment_mode"/>
                            <div id="payment_mode">
                                <field name="payment_mode" widget="radio" readonly="sheet_id"/>
                            </div>
                        </group>
                        <group>
                            <field name="date" readonly="not is_editable"/>
                            <field name="accounting_date"
                                   invisible="not accounting_date or state not in ['approved', 'done']"
                                   readonly="not is_editable"/>
                            <field name="account_id" options="{'no_create': True}"
                                   domain="[('account_type', 'not in', ('asset_receivable','liability_payable','asset_cash','liability_credit_card')), ('deprecated', '=', False)]"
                                   groups="account.group_account_readonly" readonly="not is_editable"
                                   context="{'default_company_id': company_id}"/>
                            <field name="analytic_distribution" widget="analytic_distribution"
                                groups="analytic.group_analytic_accounting"
                                options="{'product_field': 'product_id', 'account_field': 'account_id', 'business_domain': 'expense'}"
                                readonly="not is_editable"/>
                            <field name="company_id" groups="base.group_multi_company" readonly="state != 'draft'"/>
                        </group>
                    </group>
                    <div>
                        <field name="description" placeholder="Notes..." readonly="not is_editable"/>
                    </div>
                </sheet>
                <div class="o_attachment_preview o_center_attachment"/>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="activity_ids"/>
                    <field name="message_ids"/>
                </div>
                </form>
            </field>
        </record>

        <record id="hr_expense_view_form_without_header" model="ir.ui.view">
            <field name="name">hr.expense.view.form</field>
            <field name="model">hr.expense</field>
            <field name="inherit_id" ref="hr_expense.hr_expense_view_form"/>
            <field eval="35" name="priority"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="/form/header" position="attributes">
                    <attribute name="invisible">1</attribute>
                </xpath>
                <field name="employee_id" position="attributes">
                    <attribute name="readonly">1</attribute>
                </field>
                <field name="company_id" position="attributes">
                    <attribute name="readonly">1</attribute>
                </field>
            </field>
        </record>

        <record id="hr_expense_view_expenses_analysis_kanban" model="ir.ui.view">
            <field name="name">hr.expense.kanban</field>
            <field name="model">hr.expense</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile hr_expense" sample="1" js_class="hr_expense_kanban" quick_create="false">
                    <field name="name"/>
                    <field name="employee_id"/>
                    <field name="total_amount_currency"/>
                    <field name="date"/>
                    <field name="state"/>
                    <field name="activity_state"/>
                    <field name="currency_id"/>
                    <progressbar field="activity_state" colors='{"planned": "success", "today": "warning", "overdue": "danger"}'/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                                <div class="row">
                                    <div class="col-12">
                                        <strong class="o_kanban_record_title"><span><t t-out="record.name.value"/></span></strong>
                                        <strong class="o_kanban_record_subtitle float-end"><span class="text-end">
                                            <field name="total_amount_currency" widget="monetary"/></span>
                                        </strong>
                                    </div>
                                </div>
                                <div class="row mt8">
                                    <div class="col-6 text-muted">
                                        <field name="employee_id" widget="many2one_avatar_user"
                                               options="{'display_avatar_name': True}"
                                               readonly="True"/><br/>
                                        <t t-out="record.date.value"/>
                                    </div>
                                    <div class="col-6">
                                        <span class="float-end text-end">
                                            <field name="state" widget="label_selection"
                                                   options="{'classes': {'draft': 'default', 'reported': 'primary',
                                                                         'submitted': 'warning', 'refused': 'danger',
                                                                         'done': 'warning', 'approved': 'success'}}"/>
                                        </span>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <!-- Kanban view without header -->
        <record id="hr_expense_kanban_view" model="ir.ui.view">
            <field name="name">hr.expense.kanban</field>
            <field name="model">hr.expense</field>
            <field name="inherit_id" ref="hr_expense_view_expenses_analysis_kanban"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//kanban" position="attributes">
                    <attribute name="js_class">hr_expense_kanban</attribute>
                </xpath>
            </field>
        </record>

        <!-- Kanban view with header. Used in "My Expenses -->
        <record id="hr_expense_kanban_view_header" model="ir.ui.view">
            <field name="name">hr.expense.kanban</field>
            <field name="model">hr.expense</field>
            <field name="inherit_id" ref="hr_expense_view_expenses_analysis_kanban"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//kanban" position="attributes">
                    <attribute name="js_class">hr_expense_dashboard_kanban</attribute>
                </xpath>
            </field>
        </record>

        <record id="hr_expense_view_pivot" model="ir.ui.view">
            <field name="name">hr.expense.pivot</field>
            <field name="model">hr.expense</field>
            <field name="arch" type="xml">
                <pivot string="Expenses Analysis" sample="1">
                    <field name="employee_id" type="row"/>
                    <field name="date" interval="month" type="col"/>
                    <field name="total_amount_currency" type="measure"/>
                </pivot>
            </field>
        </record>

        <record id="hr_expense_view_graph" model="ir.ui.view">
            <field name="name">hr.expense.graph</field>
            <field name="model">hr.expense</field>
            <field name="arch" type="xml">
                <graph string="Expenses Analysis" sample="1">
                    <field name="price_unit" invisible="1"/>
                    <field name="quantity" invisible="1"/>
                    <field name="date"/>
                    <field name="employee_id"/>
                    <field name="total_amount" type="measure" />
                    <field name="tax_amount"/>
                </graph>
            </field>
        </record>

        <record id="hr_expense_view_search" model="ir.ui.view">
            <field name="name">hr.expense.view.search</field>
            <field name="model">hr.expense</field>
            <field name="arch" type="xml">
                <search string="Expense">
                    <field string="Expense" name="name"
                           filter_domain="['|', '|', ('employee_id', 'ilike', self), ('name', 'ilike', self), ('product_id', 'ilike', self)]"/>
                    <field name="date"/>
                    <field name="employee_id"/>
                    <filter string="My Expenses" name="my_expenses" domain="[('employee_id.user_id', '=', uid)]"/>
                    <filter string="My Team" name="my_team_expenses" domain="[('employee_id.parent_id.user_id', '=', uid)]"
                            groups="hr_expense.group_hr_expense_team_approver" help="Expenses of Your Team Member"/>
                    <separator />
                    <filter string="To Report" name="no_report" domain="[('sheet_id', '=', False)]"/>
                    <filter string="Refused" name="refused" domain="[('state', '=', 'refused')]" help="Refused Expenses"/>
                    <separator />
                    <filter string="Expense Date" name="date" date="date"/>
                    <separator />
                    <filter string="Former Employees" name="inactive" domain="[('employee_id.active', '=', False)]"
                            groups="hr_expense.group_hr_expense_user,hr_expense.group_hr_expense_manager"/>
                    <separator/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))]"/>
                    <group expand="0" string="Group By">
                        <filter string="Employee" name="employee" domain="[]" context="{'group_by': 'employee_id'}"/>
                        <filter string="Category" name="product" domain="[]" context="{'group_by': 'product_id'}"/>
                        <filter string="Status" name="status" domain="[]" context="{'group_by': 'state'}"/>
                        <filter string="Expense Date" name="expensesmonth" domain="[]" context="{'group_by': 'date'}"
                                help="Expense Date"/>
                        <filter string="Company" name="company" domain="[]" context="{'group_by': 'company_id'}"
                                groups="base.group_multi_company"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="hr_expense_view_activity" model="ir.ui.view">
            <field name="name">hr.expense.activity</field>
            <field name="model">hr.expense</field>
            <field name="arch" type="xml">
                <activity string="Expenses">
                    <field name="employee_id"/>
                    <field name="currency_id"/>
                    <templates>
                        <div t-name="activity-box">
                            <img class="rounded"
                                 t-att-src="activity_image('hr.employee', 'avatar_128', record.employee_id.raw_value)"
                                 t-att-title="record.employee_id.value" t-att-alt="record.employee_id.value"/>
                            <div class="ms-2">
                                <field name="name" display="full" class="o_text_block"/>
                                <field name="total_amount_currency" widget="monetary" muted="1" display="full"/>
                            </div>
                        </div>
                    </templates>
                </activity>
            </field>
        </record>

        <record id="hr_expense_actions_all" model="ir.actions.act_window">
            <field name="name">Expenses Analysis</field>
            <field name="res_model">hr.expense</field>
            <field name="view_mode">graph,pivot,tree,form</field>
            <field name="search_view_id" ref="hr_expense_view_search"/>
            <field name="help" type="html">
                <p class="o_view_nocontent_empty_folder">
                    No data yet!
                </p><p>
                    Create new expenses to get statistics.
                </p>
            </field>
        </record>

        <record id="hr_expense_actions_all_graph" model="ir.actions.act_window.view">
            <field name="view_mode">graph</field>
            <field name="view_id" ref="hr_expense.hr_expense_view_graph"/>
            <field name="act_window_id" ref="hr_expense_actions_all"/>
        </record>

        <record id="hr_expense_actions_all_pivot" model="ir.actions.act_window.view">
            <field name="view_mode">pivot</field>
            <field name="view_id" ref="hr_expense.hr_expense_view_pivot"/>
            <field name="act_window_id" ref="hr_expense_actions_all"/>
        </record>

        <record id="hr_expense_actions_all_tree" model="ir.actions.act_window.view">
            <field name="view_mode">tree</field>
            <field name="view_id" ref="hr_expense_view_expenses_analysis_tree"/>
            <field name="act_window_id" ref="hr_expense_actions_all"/>
        </record>

        <record id="hr_expense_actions_my_all" model="ir.actions.act_window">
            <field name="name">My Expenses</field>
            <field name="res_model">hr.expense</field>
            <field name="view_mode">tree,kanban,form,graph,pivot,activity</field>
            <field name="search_view_id" ref="hr_expense_view_search"/>
            <field name="context">{'search_default_my_expenses': 1}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_expense_receipt">
                    <h2 class="d-none d-md-block">
                        Drag and drop files to create expenses
                    </h2>
                    <p class="d-none d-md-block">
                        Or
                    </p>
                    <h2 class="d-none d-md-block">
                        Did you try the mobile app?
                    </h2>
                </p>
                <p>Snap pictures of your receipts and let Odoo<br/> automatically create expenses for you.</p>
                <p class="d-none d-md-block">
                    <a href="https://apps.apple.com/be/app/odoo/id1272543640" target="_blank" class="o_expense_mobile_app">
                        <img alt="Apple App Store" class="img img-fluid h-100 o_expense_apple_store"
                             src="/hr_expense/static/img/app_store.png"/>
                    </a>
                    <a href="https://play.google.com/store/apps/details?id=com.odoo.mobile" target="_blank"
                       class="o_expense_mobile_app">
                        <img alt="Google Play Store" class="img img-fluid h-100 o_expense_google_store"
                             src="/hr_expense/static/img/play_store.png"/>
                    </a>
                </p>
            </field>
        </record>

        <!-- Tree & Kanban view for "All My Expenses" with header -->
        <record id="hr_expense_actions_my_all_tree" model="ir.actions.act_window.view">
            <field name="view_mode">tree</field>
            <field name="view_id" ref="view_my_expenses_tree"/>
            <field name="act_window_id" ref="hr_expense_actions_my_all"/>
        </record>

        <record id="hr_expense_actions_my_all_kanban" model="ir.actions.act_window.view">
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="hr_expense_kanban_view_header"/>
            <field name="act_window_id" ref="hr_expense_actions_my_all"/>
        </record>

        <record id="view_product_hr_expense_form" model="ir.ui.view">
            <field name="name">product.template.expense.form</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="product.product_template_form_view"/>
            <field name="arch" type="xml">
                <div name="options" position="inside">
                    <span class="d-inline-flex">
                        <field name="can_be_expensed"/>
                        <label for="can_be_expensed"/>
                    </span>
                </div>
            </field>
        </record>

        <record id="product_template_search_view_inherit_hr_expense" model="ir.ui.view">
            <field name="name">product.template.search.view.inherit.hr_expense</field>
            <field name="model">product.template</field>
            <field name="inherit_id" ref="product.product_template_search_view"/>
            <field name="arch" type="xml">
                <filter name="filter_to_purchase" position="after">
                    <filter string="Can be Expensed" name="filter_to_expense" domain="[('can_be_expensed', '=', True)]"/>
                </filter>
            </field>
        </record>

        <record id="product_product_expense_form_view" model="ir.ui.view">
            <field name="name">product.product.expense.form</field>
            <field name="model">product.product</field>
            <field name="arch" type="xml">
                <form string="Expense Categories">
                    <div class="alert alert-warning" role="alert" invisible="not standard_price_update_warning">
                        <field name="standard_price_update_warning"/>
                    </div>
                    <sheet>
                        <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                        <field name='product_variant_count' invisible='1'/>
                        <field name="id" invisible="1"/>
                        <field name="image_1920" widget="image" class="oe_avatar" options="{'image_preview': 'image_128'}"/>
                        <field name="detailed_type" invisible="1"/>
                        <div class="oe_title">
                            <label for="name" string="Product Name"/>
                            <h1><field name="name" placeholder="e.g. Lunch"/></h1>
                            <div name="options" groups="base.group_user" invisible="1">
                                <div>
                                    <field name="can_be_expensed"/>
                                    <label for="can_be_expensed"/>
                                </div>
                            </div>
                        </div>
                        <group name="product_details">
                            <group string="General Information">
                                <field name="active" invisible="1"/>
                                <field name="type" invisible="1"/>
                                <field name="standard_price" class="w-25"
                                       help="When the cost of an expense product is different than 0, then the user
                                        using this product won't be able to change the amount of the expense,
                                        only the quantity. Use a cost different than 0 for expense categories funded by
                                        the company at fixed cost like allowances for mileage, per diem, accommodation
                                        or meal."/>
                                <field name="uom_id" class="w-25" groups="uom.group_uom" options="{'no_create': True}"/>
                                <field name="uom_po_id" invisible="1"/>
                                <label for="default_code"/>
                                <div>
                                    <field name="default_code" class="w-50"/>
                                    <span class="d-inline-block">
                                        <i class="text-muted">Use this reference as a subject prefix when submitting by email.</i>
                                    </span>
                                </div>
                                <field name="categ_id" class="w-50"/>
                                <field name="company_id" class="w-50" groups="base.group_multi_company"/>
                            </group>
                            <group string="Accounting">
                                <field name="property_account_expense_id" class="w-50" groups="account.group_account_readonly"/>
                                <field name="supplier_taxes_id" class="w-50" widget="many2many_tags"
                                       context="{'default_type_tax_use':'purchase', 'default_price_include': 1}"
                                       options="{'no_quick_create': True}"/>
                            </group>
                        </group>
                        <field name="description" class="mt-5"
                               placeholder="This note will be shown to users when they select this expense product."/>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="product_product_expense_kanban_view" model="ir.ui.view">
            <field name="name">product.product.kanban.expense</field>
            <field name="inherit_id" ref="product.product_kanban_view"/>
            <field name="mode">primary</field>
            <field name="model">product.product</field>
            <field name="arch" type="xml">
                <xpath expr="//div[@name='product_lst_price']" position="after">
                    <div name="product_standard_price" class="mt-1">
                        Cost: <field name="standard_price"/>
                    </div>
                </xpath>
            </field>
        </record>

        <record id="product_product_expense_tree_view" model="ir.ui.view">
            <field name="name">product.product.expense.tree</field>
            <field name="model">product.product</field>
            <field eval="50" name="priority"/>
            <field name="arch" type="xml">
                <tree string="Product Variants">
                    <field name="default_code"/>
                    <field name="name"/>
                    <field name="product_template_attribute_value_ids" widget="many2many_tags"
                           groups="product.group_product_variant"/>
                    <field name="standard_price"/>
                    <field name="uom_id" options="{'no_open': True, 'no_create': True}" groups="uom.group_uom"/>
                    <field name="barcode"/>
                </tree>
            </field>
        </record>

        <record model="ir.ui.view" id="product_product_expense_categories_tree_view">
            <field name="name">product.product.expense.categories.tree.view</field>
            <field name="model">product.product</field>
            <field name="arch" type="xml">
                <tree class="o_expense_categories">
                    <field name="name" readonly="1"/>
                    <field name="default_code" optional="show" readonly="1"/>
                    <field name="description" widget="html" string="Internal Note" optional="show" readonly="1"/>
                    <field name="lst_price" optional="show" string="Sales Price"/>
                    <field name="standard_price" optional="show"/>
                    <field name="supplier_taxes_id" widget="many2many_tags" optional="show"/>
                </tree>
            </field>
        </record>

        <record id="hr_expense_product" model="ir.actions.act_window">
            <field name="name">Expense Categories</field>
            <field name="res_model">product.product</field>
            <field name="view_mode">tree,kanban,form</field>
            <field name="search_view_id" ref="product.product_search_form_view"/>
            <field name="context">{"default_can_be_expensed": 1, 'default_detailed_type': 'service'}</field>
            <field name="domain">[('can_be_expensed', '=', True)]</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No expense categories found. Let's create one!
              </p><p>
                Expense categories can be reinvoiced to your customers.
              </p>
            </field>
        </record>

        <record id="hr_expense_product_tree" model="ir.actions.act_window.view">
            <field name="sequence" eval="1"/>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="product_product_expense_categories_tree_view"/>
            <field name="act_window_id" ref="hr_expense_product"/>
        </record>

        <record id="hr_expense_product_kanban" model="ir.actions.act_window.view">
            <field name="sequence" eval="2"/>
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="product_product_expense_kanban_view"/>
            <field name="act_window_id" ref="hr_expense_product"/>
        </record>

        <record id="hr_expense_product_form" model="ir.actions.act_window.view">
            <field name="sequence" eval="3"/>
            <field name="view_mode">form</field>
            <field name="view_id" ref="product_product_expense_form_view"/>
            <field name="act_window_id" ref="hr_expense_product"/>
        </record>

        <record id="view_hr_expense_sheet_tree" model="ir.ui.view">
            <field name="name">hr.expense.sheet.tree</field>
            <field name="model">hr.expense.sheet</field>
            <field name="arch" type="xml">
                <tree string="Expense Reports" multi_edit="1" js_class="hr_expense_tree" sample="1" decoration-info="state == 'draft'">
                    <field name="product_ids" column_invisible="True"/>
                    <field name="currency_id" column_invisible="True"/>
                    <field name="company_currency_id" column_invisible="True"/>
                    <field name="is_editable" column_invisible="True"/>
                    <field name="employee_id" widget="many2one_avatar_user" readonly="state != 'draft'"/>
                    <field name="accounting_date" optional="hide" groups="account.group_account_manager" readonly="not is_editable"/>
                    <field name="create_date" optional="hide"/>
                    <field name="name" string="Expense Report" readonly="not is_editable"/>
                    <field name="payment_mode" optional="hide"/>
                    <field name="user_id" optional="hide" widget="many2one_avatar_user" readonly="state != 'draft'"/>
                    <field name="company_id" optional="show" groups="base.group_multi_company" readonly="state != 'draft'"/>
                    <field name="activity_ids" widget="list_activity" optional="show" readonly="True"/>
                    <field name="journal_id" optional="hide" readonly="not is_editable"/>
                    <field name="total_amount" sum="Total Amount" decoration-bf="1" widget="monetary"/>
                    <field name="state" optional="show"
                           decoration-info="state == 'draft'"
                           decoration-success="state in ['approve', 'post', 'done']"
                           decoration-warning="state == 'submit'"
                           decoration-danger="state == 'cancel'"
                           widget="badge"/>
                    <field name="payment_state" optional="show"
                           decoration-info="payment_state in ('partial','in_payment')"
                           decoration-success="payment_state == 'paid'"
                           decoration-danger="payment_state in ('reversed','not_paid')"
                           widget="badge" invisible="state in ['draft', 'submit', 'cancel']"/>
                </tree>
            </field>
        </record>

        <!-- Tree view for "My Reports" with header -->
        <record id="view_hr_expense_sheet_dashboard_tree_header" model="ir.ui.view">
            <field name="name">hr.expense.sheet.dashboard.tree</field>
            <field name="model">hr.expense.sheet</field>
            <field name="priority">20</field>
            <field name="inherit_id" ref="hr_expense.view_hr_expense_sheet_tree"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//tree" position="attributes">
                    <attribute name="js_class">hr_expense_dashboard_tree</attribute>
                </xpath>
            </field>
        </record>

        <record id="view_hr_expense_sheet_form" model="ir.ui.view">
            <field name="name">hr.expense.sheet.form</field>
            <field name="model">hr.expense.sheet</field>
            <field eval="25" name="priority"/>
            <field name="arch" type="xml">
                <form string="Expense Reports">
                <field name="can_reset" invisible="1"/>
                <field name="can_approve" invisible="1"/>
                <field name="company_id" invisible="1"/>
                <field name="payment_state" invisible="1"/>
                <field name="is_editable" invisible="1"/>
                <field name="currency_id" invisible="1"/>
                <field name="company_currency_id" invisible="1"/>
                <header>
                    <button name="action_submit_sheet"
                            string="Submit to Manager"
                            invisible="state != 'draft'"
                            type="object"
                            class="oe_highlight o_expense_sheet_submit"
                            data-hotkey="l"/>
                    <button name="action_approve_expense_sheets"
                            string="Approve"
                            type="object"
                            data-hotkey="q"
                            context="{'validate_analytic': True}"
                            invisible="not can_approve or state != 'submit'"
                            class="oe_highlight o_expense_sheet_approve"/>
                    <button name="action_sheet_move_create"
                            string="Post Journal Entries"
                            type="object"
                            data-hotkey="y"
                            class="oe_highlight o_expense_sheet_post"
                            invisible="state != 'approve'"
                            groups="account.group_account_invoice"/>
                    <button name="action_register_payment"
                            string="Register Payment"
                            type="object"
                            data-hotkey="w"
                            class="oe_highlight o_expense_sheet_pay"
                            context="{'dont_redirect_to_payments': True}"
                            invisible="payment_mode == 'company_account' or state not in ('post', 'done') or payment_state in ('paid', 'in_payment')"
                            groups="account.group_account_invoice"/>
                    <button name="action_refuse_expense_sheets"
                            string="Refuse"
                            invisible="state not in ('submit', 'approve')"
                            type="object"
                            groups="hr_expense.group_hr_expense_team_approver"
                            data-hotkey="x"/>
                    <button name="action_reset_approval_expense_sheets"
                            string="Reset to Draft"
                            type="object"
                            invisible="not can_reset or state not in ('submit', 'cancel', 'approve')"
                            data-hotkey="k"/>
                    <button name="action_reset_expense_sheets"
                            string="Reset to Draft"
                            type="object"
                            data-hotkey="c"
                            invisible="state != 'post'"
                            groups="account.group_account_readonly,account.group_account_invoice"/>
                    <field name="state" widget="statusbar" statusbar_visible="draft,submit,approve,post,done"
                           force_save="1" invisible="state == 'cancel'"/>
                    <field name="state" widget="statusbar" statusbar_visible="draft,submit,cancel"
                           force_save="1" invisible="state != 'cancel'"/>

                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_open_account_moves"
                            class="oe_stat_button"
                            icon="fa-file-text-o"
                            type="object"
                            invisible="state not in ['post', 'done'] or nb_account_move == 0"
                            groups="account.group_account_invoice">
                            <div class="o_stat_info">
                                <field name="nb_account_move" class="o_stat_value"/>
                                <span class="o_stat_text">Journal Entry</span>
                            </div>
                        </button>
                        <button name="action_open_expense_view"
                            class="oe_stat_button"
                            icon="fa-file-text-o"
                            type="object"
                            invisible="nb_expense == 0">
                            <field name="nb_expense" widget="statinfo" string="Expenses"/>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Paid" bg_color="text-bg-success" invisible="payment_state != 'paid'"/>
                    <widget name="web_ribbon" title="Partial" bg_color="text-bg-info" invisible="payment_state != 'partial'"/>
                    <widget name="web_ribbon" title="In Payment" invisible="payment_state != 'in_payment'"/>
                    <div class="oe_title">
                        <label for="name" class="oe_edit_only"/>
                        <h1>
                            <field name="name" placeholder="e.g. Trip to NY" readonly="not is_editable" force_save="1"/>
                        </h1>
                    </div>
                    <group>
                        <group name="employee_details">
                            <field name="employee_id" context="{'default_company_id': company_id}"
                                   widget="many2one_avatar_user" readonly="state != 'draft'"/>
                            <field name="payment_mode"/>
                            <field name="employee_journal_id"
                                   groups="account.group_account_invoice,account.group_account_readonly"
                                   options="{'no_open': True, 'no_create': True}"
                                   invisible="payment_mode != 'own_account'" readonly="not is_editable"
                                   context="{'default_company_id': company_id}"/>
                            <field name="selectable_payment_method_line_ids" invisible="1"/>
                            <field name="payment_method_line_id"
                                   context="{'show_payment_journal_id': 1}"
                                   options="{'no_open': True, 'no_create': True}"
                                   invisible="payment_mode != 'company_account'"
                                   readonly="not is_editable"
                                   required="payment_mode == 'company_account'"/>
                            <field name="department_id" invisible="1" readonly="not is_editable"
                                   context="{'default_company_id': company_id}"/>
                        </group>
                        <group>
                            <field name="company_id" groups="base.group_multi_company" readonly="state != 'draft'"/>
                            <field name="user_id" widget="many2one_avatar_user" readonly="state != 'draft'"/>
                            <field name="accounting_date"
                                   groups="account.group_account_invoice,account.group_account_readonly"
                                   invisible="state not in ['approve', 'post', 'done']"
                                   readonly="not is_editable"/>
                        </group>
                    </group>
                     <notebook>
                        <page name="expenses" string="Expense">
                            <field name="expense_line_ids"
                                    nolabel="1"
                                    widget="expense_lines_widget"
                                    mode="tree,kanban"
                                    domain="[
                                        ('sheet_id', '=', False),
                                        ('employee_id', '=', employee_id),
                                        ('company_id', '=', company_id),
                                        ('payment_mode', '=?', payment_mode),
                                    ]"
                                    options="{'reload_on_button': True}"
                                    context="{
                                        'form_view_ref' : 'hr_expense.hr_expense_view_form_without_header',
                                        'default_company_id': company_id,
                                        'default_employee_id': employee_id,
                                        'default_payment_mode': payment_mode or 'own_account',
                                    }"
                                    readonly="not is_editable"
                                    force_save="1">
                                <tree editable="bottom">
                                    <field name="employee_id" column_invisible="True"/>
                                    <field name="state" column_invisible="True"/>
                                    <field name="nb_attachment" column_invisible="True"/>
                                    <field name="message_main_attachment_id" column_invisible="True"/>
                                    <field name="company_id" column_invisible="True"/>
                                    <field name="company_currency_id" column_invisible="True"/>
                                    <field name="is_multiple_currency" column_invisible="True"/>
                                    <field name="product_has_cost" column_invisible="True"/>
                                    <field name="date" optional="show"/>
                                    <field name="product_id"/>
                                    <field name="name"/>
                                    <field name="description" optional="hide"/>
                                    <button name="action_get_attachment_view" type="object" icon="fa-paperclip"
                                            aria-label="View Attachments" title="View Attachments" class="float-end pe-0"
                                            readonly="True" invisible="nb_attachment == 0"/>
                                    <field name="analytic_distribution" widget="analytic_distribution"
                                           groups="analytic.group_analytic_accounting"
                                           optional="show"
                                           options="{'product_field': 'product_id', 'account_field': 'account_id', 'business_domain': 'expense'}"/>
                                    <field name="account_id" optional="hide" groups="account.group_account_readonly"/>
                                    <field name="price_unit" optional="hide" widget="monetary" options="{'currency_field': 'company_currency_id', 'field_digits': True}" readonly="True"/>
                                    <field name="currency_id" optional="hide" readonly="True" groups="base.group_multi_currency"/>
                                    <field name="quantity" optional="hide" readonly="True"/>
                                    <field name="tax_ids" string="Taxes" optional="show" widget="many2many_tags"
                                           context="{'default_company_id': company_id}" readonly="True"/>
                                    <field name="tax_amount_currency" optional="hide"  options="{'currency_field': 'currency_id'}"
                                           context="{'default_company_id': company_id}" readonly="True" groups="base.group_multi_currency"/>
                                    <field name="tax_amount" optional="hide" readonly="True"/>
                                    <field name="total_amount_currency" options="{'currency_field': 'currency_id'}"
                                           string="Subtotal In Currency " optional="show"
                                           readonly="true" groups="base.group_multi_currency"/>
                                    <field name="total_amount" string="Subtotal" readonly="True"/>
                                </tree>
                            </field>
                            <group class="oe_subtotal_footer" colspan="2" name="expense_total">
                                <field name="untaxed_amount"/>
                                <div class="oe_inline o_td_label">
                                    <label for="total_tax_amount"/>
                                </div>
                                <field name="total_tax_amount" nolabel="1"/>
                                <div class="oe_inline o_td_label">
                                    <label for="total_amount"/>
                                </div>
                                <field name="total_amount" nolabel="1" class="oe_subtotal_footer_separator"/>
                                <field name="amount_residual"
                                    class="oe_subtotal_footer_separator"
                                    invisible="state not in ('post', 'done')"/>
                            </group>
                        </page>
                     </notebook>
                </sheet>
                <div class="o_attachment_preview o_center_attachment"/>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="activity_ids"/>
                    <field name="message_ids"/>
                </div>
                </form>
            </field>
        </record>

        <record id="view_hr_expense_sheet_kanban" model="ir.ui.view">
            <field name="name">hr.expense.sheet.kanban</field>
            <field name="model">hr.expense.sheet</field>
            <field name="arch" type="xml">
                <kanban class="o_kanban_mobile" sample="1">
                    <field name="name"/>
                    <field name="employee_id"/>
                    <field name="total_amount"/>
                    <field name="accounting_date"/>
                    <field name="state"/>
                    <field name="currency_id"/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="oe_kanban_card oe_kanban_global_click">
                                <div class="row">
                                    <div class="col-12">
                                        <strong class="o_kanban_record_title"><span><t t-out="record.name.value"/></span></strong>
                                        <strong class="o_kanban_record_subtitle float-end">
                                            <span class="text-end"><field name="total_amount" widget="monetary"/></span>
                                        </strong>
                                    </div>
                                </div>
                                <div class="row mt8">
                                    <div class="col-6 text-muted">
                                        <field name="employee_id" widget="many2one_avatar_user"  options="{'display_avatar_name': True}" readonly="state != 'draft'"/><t t-out="record.accounting_date.value"/>
                                    </div>
                                    <div class="col-6">
                                        <span class="float-end text-end">
                                            <field name="state" widget="label_selection"
                                                   options="{'classes': {'draft': 'default', 'submit': 'default',
                                                                         'cancel': 'danger', 'post': 'warning',
                                                                         'done': 'success'}}"/>
                                        </span>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <!-- Kanban view without header -->
        <record id="view_hr_expense_sheet_kanban_no_header" model="ir.ui.view">
            <field name="name">hr.expense.sheet.kanban</field>
            <field name="model">hr.expense.sheet</field>
            <field name="inherit_id" ref="view_hr_expense_sheet_kanban"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//kanban" position="attributes">
                    <attribute name="js_class">hr_expense_kanban</attribute>
                </xpath>
            </field>
        </record>

        <!-- Kanban view with header -->
        <record id="view_hr_expense_sheet_kanban_header" model="ir.ui.view">
            <field name="name">hr.expense.sheet.kanban</field>
            <field name="model">hr.expense.sheet</field>
            <field name="inherit_id" ref="view_hr_expense_sheet_kanban"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//kanban" position="attributes">
                    <attribute name="js_class">hr_expense_dashboard_kanban</attribute>
                </xpath>
            </field>
        </record>

        <record id="view_hr_expense_sheet_pivot" model="ir.ui.view">
            <field name="name">hr.expense.sheet.pivot</field>
            <field name="model">hr.expense.sheet</field>
            <field name="arch" type="xml">
                <pivot string="Expenses Analysis" sample="1">
                    <field name="employee_id" type="row"/>
                    <field name="accounting_date" interval="month" type="col"/>
                    <field name="total_amount" type="measure"/>
                </pivot>
            </field>
        </record>

        <record id="view_hr_expense_sheet_graph" model="ir.ui.view">
            <field name="name">hr.expense.sheet.graph</field>
            <field name="model">hr.expense.sheet</field>
            <field name="arch" type="xml">
                <graph string="Expenses Analysis" sample="1">
                    <field name="employee_id"/>
                    <field name="accounting_date" interval="month"/>
                    <field name="total_amount" type="measure"/>
                </graph>
            </field>
        </record>


        <record id="hr_expense_sheet_view_search" model="ir.ui.view">
            <field name="name">hr.expense.sheet.view.search</field>
            <field name="model">hr.expense.sheet</field>
            <field name="arch" type="xml">
                <search string="Expense">
                    <field string="Expense Report" name="name"/>
                    <field name="accounting_date"/>
                    <separator />
                    <field name="employee_id"/>
                    <field string="Department" name="department_id" operator="child_of"/>
                    <field string="Journal" name="journal_id"/>
                    <filter string="My Reports" name="my_reports" domain="[('employee_id.user_id', '=', uid)]"/>
                    <filter string="My Team" name="my_team_reports"
                            domain="[('employee_id.parent_id.user_id', '=', uid)]"
                            groups="hr_expense.group_hr_expense_manager" help="Expenses of Your Team Member"/>
                    <separator invisible="1"/>
                    <filter string="Not Refused" name="not_refused_reports"
                            domain="[('employee_id.user_id', '=', uid), ('state', '!=', 'cancel')]" invisible="1"/>
                    <separator />
                    <filter string="Date" name="filter_accounting_date" date="accounting_date"/>
                    <separator/>
                    <filter domain="[('employee_id.active', '=', False)]" string="Former Employees"
                            name="inactive" groups="hr_expense.group_hr_expense_user,hr_expense.group_hr_expense_manager"/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                            domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))
                            ]"/>
                    <group expand="0" string="Group By" name="group_filters">
                        <filter string="Employee" name="employee" domain="[]" context="{'group_by': 'employee_id'}"/>
                        <filter string="Department" name="department" domain="[]" context="{'group_by': 'department_id'}"/>
                        <filter string="Company" name="company" domain="[]" context="{'group_by': 'company_id'}"
                                groups="base.group_multi_company"/>
                        <filter string="Date" name="expenses_month" domain="[]" context="{'group_by': 'accounting_date'}"
                                help="Expenses by Date"/>
                        <filter string="Status" domain="[]" context="{'group_by': 'state'}" name="state"/>
                    </group>
                </search>
            </field>
        </record>

        <record id="hr_expense_sheet_view_search_with_panel" model="ir.ui.view">
            <field name="name">hr.expense.sheet.view.search.with.panel</field>
            <field name="model">hr.expense.sheet</field>
            <field name="inherit_id" ref="hr_expense_sheet_view_search"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//group[@name='group_filters']" position='after'>
                    <searchpanel>
                        <field name="state" expand="1" select="multi" icon="fa-check-square-o" enable_counters="1"/>
                        <field name="employee_id"  limit="20" hierarchize="0" select="one" icon="fa-users"/>
                        <field name="company_id" expand="1" icon="fa-building" groups="base.group_multi_company"/>
                    </searchpanel>
                </xpath>
            </field>
        </record>

        <record id="hr_expense_sheet_view_activity" model="ir.ui.view">
            <field name="name">hr.expense.sheet.activity</field>
            <field name="model">hr.expense.sheet</field>
            <field name="arch" type="xml">
                <activity string="Expenses">
                    <field name="employee_id"/>
                    <field name="currency_id"/>
                    <templates>
                        <div t-name="activity-box">
                            <img class="rounded"
                                 t-att-src="activity_image('hr.employee', 'avatar_128', record.employee_id.raw_value)"
                                 t-att-title="record.employee_id.value" t-att-alt="record.employee_id.value"/>
                            <div class="ms-2">
                                <field name="name" display="full" class="o_text_block"/>
                                <field name="total_amount" widget="monetary" muted="1" display="full"/>
                                <field name="state" display="right"/>
                            </div>
                        </div>
                    </templates>
                </activity>
            </field>
        </record>

        <record id="action_hr_expense_sheet_my_all" model="ir.actions.act_window">
            <field name="name">My Reports</field>
            <field name="res_model">hr.expense.sheet</field>
            <field name="view_mode">tree,kanban,form,pivot,graph,activity</field>
            <field name="search_view_id" ref="hr_expense_sheet_view_search"/>
            <field name="context">{'search_default_my_reports': 1, 'search_default_not_refused_reports': 1}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No expense report found. Let's create one!
              </p><p>
                Once you have created your expense, submit it to your manager who will validate it.
              </p>
            </field>
        </record>

        <!-- Tree & Kanban view for "My Reports" with header -->
        <record id="action_hr_expense_sheet_my_all_tree" model="ir.actions.act_window.view">
            <field name="view_mode">tree</field>
            <field name="view_id" ref="view_hr_expense_sheet_dashboard_tree_header"/>
            <field name="act_window_id" ref="action_hr_expense_sheet_my_all"/>
        </record>

        <record id="action_hr_expense_sheet_my_all_kanban" model="ir.actions.act_window.view">
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="view_hr_expense_sheet_kanban_header"/>
            <field name="act_window_id" ref="action_hr_expense_sheet_my_all"/>
        </record>

        <record id="action_hr_expense_sheet_all" model="ir.actions.act_window">
            <field name="name">All Reports</field>
            <field name="res_model">hr.expense.sheet</field>
            <field name="view_mode">tree,kanban,form,pivot,graph</field>
            <field name="search_view_id" ref="hr_expense_sheet_view_search_with_panel"/>
            <field name="domain">[]</field>
            <field name="context">{ 'searchpanel_default_state': ["draft", "submit", "approve", "post", "done"] }</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No expense reports found. Let's create one!
                </p><p>
                    Expense reports regroup all the expenses incurred during a specific event.
                </p>
            </field>
        </record>

        <record id="action_hr_expense_account" model="ir.actions.act_window">
            <field name="name">Employee Expenses</field>
            <field name="res_model">hr.expense.sheet</field>
            <field name="view_mode">tree,kanban,form,pivot,graph</field>
            <field name="search_view_id" ref="hr_expense_sheet_view_search"/>
            <field name="view_id" ref="view_hr_expense_sheet_tree"/>
            <field name="domain">[]</field>
            <field name="context">{
                'search_default_approved': 1,
                'search_default_to_post': 1,
            }
            </field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new expense report
              </p><p>
                Once you have created your expense, submit it to your manager who will validate it.
              </p>
            </field>
        </record>

        <record id="action_hr_expense_sheet_all_all" model="ir.actions.act_window">
            <field name="name">All Expense Reports</field>
            <field name="res_model">hr.expense.sheet</field>
            <field name="view_mode">graph,pivot,tree,kanban,form</field>
            <field name="search_view_id" ref="hr_expense_sheet_view_search"/>
            <field name="domain">[]</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new expense report
              </p><p>
                Once you have created your expense, submit it to your manager who will validate it.
              </p>
            </field>
        </record>

        <record id="action_hr_expense_sheet_department_to_approve" model="ir.actions.act_window">
            <field name="name">Expense Reports to Approve</field>
            <field name="res_model">hr.expense.sheet</field>
            <field name="view_mode">tree,kanban,form,pivot,graph</field>
            <field name="search_view_id" ref="hr_expense_sheet_view_search_with_panel"/>
            <field name="context">{ 'searchpanel_default_state': ["submit"] }</field>
        </record>

        <record id="action_hr_expense_sheet_department_filtered" model="ir.actions.act_window">
            <field name="name">Expense Reports Analysis</field>
            <field name="res_model">hr.expense.sheet</field>
            <field name="view_mode">graph,pivot</field>
            <field name="context">{
                'search_default_department_id': [active_id],
                'default_department_id': active_id}
            </field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No data yet!
                </p>
            </field>
        </record>

        <menuitem id="menu_hr_expense_root" name="Expenses" sequence="230" web_icon="hr_expense,static/description/icon.png"/>

        <menuitem id="menu_hr_expense_my_expenses" name="My Expenses" sequence="1" parent="menu_hr_expense_root" groups="base.group_user"/>
        <menuitem id="menu_hr_expense_my_expenses_all" sequence="1" parent="menu_hr_expense_my_expenses"
                  action="hr_expense_actions_my_all" name="My Expenses"/>
        <menuitem id="menu_hr_expense_sheet_my_reports" sequence="2" parent="menu_hr_expense_my_expenses"
                  action="action_hr_expense_sheet_my_all" name="My Reports"/>

        <menuitem id="menu_hr_expense_report" name="Expense Reports" sequence="2" parent="menu_hr_expense_root"
                   action="action_hr_expense_sheet_all"
                   groups="account.group_account_user,hr_expense.group_hr_expense_team_approver"/>

        <menuitem id="menu_hr_expense_reports" name="Reporting" sequence="4" parent="menu_hr_expense_root"
                  groups="hr_expense.group_hr_expense_manager"/>
        <menuitem id="menu_hr_expense_all_expenses" name="Expenses Analysis" sequence="0"
                  parent="menu_hr_expense_reports" action="hr_expense_actions_all"/>

        <menuitem id="menu_hr_expense_configuration" name="Configuration" parent="menu_hr_expense_root"
            sequence="100"/>
        <menuitem id="menu_hr_product" name="Expense Categories" parent="menu_hr_expense_configuration"
            action="hr_expense_product" groups="hr_expense.group_hr_expense_manager" sequence="10"/>

        <menuitem id="menu_hr_expense_account_employee_expenses" name="Employee Expenses" sequence="22"
                  parent="account.menu_finance_payables" groups="hr_expense.group_hr_expense_user"
                  action="action_hr_expense_account"/>
</odoo>

```

## File: views\mail_activity_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <!-- Activity types config -->
    <record id="mail_activity_type_action_config_hr_expense" model="ir.actions.act_window">
        <field name="name">Activity Types</field>
        <field name="res_model">mail.activity.type</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">['|', ('res_model', '=', False), ('res_model', '=', 'hr.expense.sheet')]</field>
        <field name="context">{'default_res_model': 'hr.expense.sheet'}</field>
    </record>
    <menuitem id="hr_expense_menu_config_activity_type"
        action="mail_activity_type_action_config_hr_expense"
        parent="menu_hr_expense_configuration"
        groups="base.group_no_one"/>
</odoo>
```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.hr.expense</field>
            <field name="model">res.config.settings</field>
            <field name="priority" eval="85"/>
            <field name="inherit_id" ref="base.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//form" position="inside">
                    <app data-string="Expenses" string="Expenses" name="hr_expense" groups="hr_expense.group_hr_expense_manager">
                        <block title="Expenses" name="expenses_setting_container">
                            <setting id="create_expense_setting" string="Incoming Emails"
                                     help="Create expenses from incoming emails"
                                     title="Send an email to this email alias with the receipt in attachment to create an expense in one click. If the first word of the mail subject contains the category's internal reference or the category name, the corresponding category will automatically be set. Type the expense amount in the mail subject to set it on the expense too.">
                                <field name="hr_expense_use_mailgateway"/>
                                <div class="content-group" invisible="not hr_expense_use_mailgateway or not alias_domain_id">
                                    <div class="mt16 d-flex align-items-start" dir="ltr">
                                        <label for="hr_expense_alias_prefix" string="Alias" class="o_light_label"/>
                                        <field name="hr_expense_alias_prefix" class="oe_inline ps-2"/>
                                        <span>@</span>
                                        <field name="hr_expense_alias_domain_id" class="oe_inline" placeholder="e.g. domain.com"
                                               options="{'no_create': True, 'no_open': True}"/>
                                    </div>
                                </div>
                                <div class="content-group" invisible="not hr_expense_use_mailgateway or alias_domain_id">
                                    <div class="mt16">
                                        <button type="action" name="base_setup.action_general_configuration"
                                                icon="oi-arrow-right" string="Setup your alias domain" class="btn-link"/>
                                    </div>
                                </div>
                            </setting>
                            <setting string="Reimburse in Payslip" help="Reimburse expenses in payslips" id="hr_payroll_accountant">
                                <field name="module_hr_payroll_expense" widget="upgrade_boolean"/>
                            </setting>
                            <setting id="expense_extract_settings" string="Expense Digitalization (OCR)" company_dependent="1"
                                     help="Digitalize your receipts with OCR and Artificial Intelligence"
                                     title="use OCR to fill data from a picture of the bill">
                                <field name="module_hr_expense_extract" widget="upgrade_boolean"/>
                            </setting>
                            <setting company_dependent="1" string="Default Category"
                                     help="Default expense categories for uploaded expenses"
                                     title="Enable users to choose default category for automatically generated expenses.">
                                <field name="expense_product_id"/>
                            </setting>
                        </block>
                        <block title="Accounting">
                            <setting company_dependent="1" help="Default accounting journal for expenses paid by employees."
                                     string="Employee Expense Journal">
                                <field name="expense_journal_id"/>
                            </setting>
                            <setting company_dependent="1" string="Payment methods"
                                     help="Payment method allowed for expenses paid by company.">
                                <field name="company_expense_allowed_payment_method_line_ids" widget="many2many_tags"
                                       placeholder="All payment methods allowed" options="{'no_create': True}"
                                       context="{'show_payment_journal_id': 1}"/>
                            </setting>
                        </block>
                    </app>
                </xpath>
            </field>
        </record>

        <record id="action_hr_expense_configuration" model="ir.actions.act_window">
            <field name="name">Settings</field>
            <field name="res_model">res.config.settings</field>
            <field name="view_mode">form</field>
            <field name="target">inline</field>
            <field name="context">{'module' : 'hr_expense', 'bin_size': False}</field>
        </record>

        <menuitem id="menu_hr_expense_global_settings" name="Settings"
            parent="menu_hr_expense_configuration" sequence="0" action="action_hr_expense_configuration" groups="base.group_system"/>
    </data>
</odoo>

```

## File: wizard\account_payment_register.py

```python
# -*- coding: utf-8 -*-

from odoo import models, fields, api, _


class AccountPaymentRegister(models.TransientModel):
    _inherit = 'account.payment.register'

    # -------------------------------------------------------------------------
    # BUSINESS METHODS
    # -------------------------------------------------------------------------

    @api.model
    def _get_line_batch_key(self, line):
        # OVERRIDE to set the bank account defined on the employee
        res = super()._get_line_batch_key(line)
        expense_sheet = line.move_id.expense_sheet_id.filtered(lambda sheet: sheet and sheet.payment_mode == 'own_account')
        if expense_sheet and not line.move_id.partner_bank_id:
            res['partner_bank_id'] = expense_sheet.employee_id.sudo().bank_account_id.id \
                                     or line.partner_id.bank_ids  \
                                     and line.partner_id.bank_ids.ids[0]
        return res

    def _init_payments(self, to_process, edit_mode=False):
        # OVERRIDE
        payments = super()._init_payments(to_process, edit_mode=edit_mode)
        for payment, vals in zip(payments, to_process):
            expenses = vals['batch']['lines'].expense_id
            if expenses:
                payment.line_ids.write({'expense_id': expenses[0].id})
        return payments

    def _reconcile_payments(self, to_process, edit_mode=False):
        # OVERRIDE
        res = super()._reconcile_payments(to_process, edit_mode=edit_mode)
        for vals in to_process:
            expense_sheets = vals['batch']['lines'].expense_id.sheet_id
            for expense_sheet in expense_sheets:
                if expense_sheet.currency_id.is_zero(expense_sheet.amount_residual):
                    expense_sheet.state = 'done'
        return res

```

## File: wizard\hr_expense_approve_duplicate.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class HrExpenseApproveDuplicate(models.TransientModel):
    """
    This wizard is shown whenever an approved expense is similar to one being
    approved. The user has the opportunity to still validate it or decline.
    """

    _name = "hr.expense.approve.duplicate"
    _description = "Expense Approve Duplicate"

    sheet_ids = fields.Many2many('hr.expense.sheet')
    expense_ids = fields.Many2many('hr.expense', readonly=True)

    @api.model
    def default_get(self, fields):
        res = super().default_get(fields)

        if 'sheet_ids' in fields:
            res['sheet_ids'] = [(6, 0, self.env.context.get('default_sheet_ids', []))]
        if 'duplicate_expense_ids' in fields:
            res['expense_ids'] = [(6, 0, self.env.context.get('default_expense_ids', []))]

        return res

    def action_approve(self):
        self.sheet_ids._do_approve()
        return {'type': 'ir.actions.act_window_close'}

    def action_refuse(self):
        self.sheet_ids._do_refuse(_('Duplicate Expense'))
        return {'type': 'ir.actions.act_window_close'}

```

## File: wizard\hr_expense_approve_duplicate_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_expense_approve_duplicate_view_form" model="ir.ui.view">
        <field name="model">hr.expense.approve.duplicate</field>
        <field name="arch" type="xml">
            <form string="Expense Validate Duplicate">
                <field name="sheet_ids" invisible="1" />
                <p>The following approved expenses have similar employee, amount and category than some expenses of this report. Please verify this report does not contain duplicates.</p>
                <field name="expense_ids" nolabel="1">
                    <tree>
                        <field name="date" readonly="1" />
                        <field name="employee_id" readonly="1" widget="many2one_avatar_user"/>
                        <field name="product_id" readonly="1" />
                        <field name="total_amount" readonly="1" />
                        <field name="name" readonly="1" />
                        <field name="approved_by" readonly="1" widget="many2one_avatar_user"/>
                        <field name="approved_on" readonly="1" />
                    </tree>
                </field>
                <footer>
                    <button string="Refuse" class="btn-primary" name="action_refuse" type="object" invisible="not sheet_ids" data-hotkey="q" />
                    <button string="Approve" class="btn-secondary" name="action_approve" type="object" invisible="not sheet_ids" data-hotkey="w" />
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x"/>
                </footer>
           </form>
        </field>
    </record>

    <record id="hr_expense_approve_duplicate_action" model="ir.actions.act_window">
        <field name="name">Validate Duplicate Expenses</field>
        <field name="res_model">hr.expense.approve.duplicate</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="hr_expense_approve_duplicate_view_form"/>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\hr_expense_refuse_reason.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class HrExpenseRefuseWizard(models.TransientModel):
    """ Wizard to specify reason on expense sheet refusal """

    _name = "hr.expense.refuse.wizard"
    _description = "Expense Refuse Reason Wizard"

    reason = fields.Char(string='Reason', required=True)
    sheet_ids = fields.Many2many('hr.expense.sheet')

    @api.model
    def default_get(self, fields):
        res = super().default_get(fields)
        if 'sheet_ids' in fields:
            res['sheet_ids'] = self.env.context.get('active_ids', [])
        return res

    def action_refuse(self):
        self.sheet_ids._do_refuse(self.reason)
        return {'type': 'ir.actions.act_window_close'}

```

## File: wizard\hr_expense_refuse_reason_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_expense_refuse_wizard_view_form" model="ir.ui.view">
        <field name="name">hr.expense.refuse.wizard.form</field>
        <field name="model">hr.expense.refuse.wizard</field>
        <field name="arch" type="xml">
            <form string="Expense refuse reason">
                <separator string="Reason to refuse Expense"/>
                <field name="sheet_ids" invisible="1"/>
                <field name="reason" class="w-100"/>
                <footer>
                    <button string='Refuse' name="action_refuse" type="object" class="oe_highlight" data-hotkey="q"/>
                    <button string="Cancel" class="oe_link" special="cancel" data-hotkey="x"/>
                </footer>
           </form>
        </field>
    </record>

    <record id="hr_expense_refuse_wizard_action" model="ir.actions.act_window">
        <field name="name">Refuse Expense</field>
        <field name="res_model">hr.expense.refuse.wizard</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="hr_expense_refuse_wizard_view_form"/>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\hr_expense_split.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from copy import deepcopy

from odoo import fields, models, api, Command
from odoo.tools import float_compare


class HrExpenseSplit(models.TransientModel):
    _name = 'hr.expense.split'
    _inherit = ['analytic.mixin']
    _description = 'Expense Split'
    _check_company_auto = True

    def default_get(self, fields):
        result = super().default_get(fields)
        if 'expense_id' in result:
            expense = self.env['hr.expense'].browse(result['expense_id'])
            result['total_amount_currency'] = 0.0
            result['name'] = expense.name
            result['tax_ids'] = expense.tax_ids
            result['product_id'] = expense.product_id
            result['company_id'] = expense.company_id
            result['analytic_distribution'] = deepcopy(expense.analytic_distribution) or {}
            result['employee_id'] = expense.employee_id
            result['currency_id'] = expense.currency_id
        return result

    name = fields.Char(string='Description', required=True)
    wizard_id = fields.Many2one(comodel_name='hr.expense.split.wizard')
    expense_id = fields.Many2one(comodel_name='hr.expense', string='Expense')
    product_id = fields.Many2one(comodel_name='product.product', string='Product', required=True, check_company=True, domain=[('can_be_expensed', '=', True)],)
    tax_ids = fields.Many2many(
        comodel_name='account.tax',
        check_company=True,
        domain="[('type_tax_use', '=', 'purchase')]",
    )
    total_amount_currency = fields.Monetary(
        string="Total In Currency",
        required=True,
        compute='_compute_from_product_id', store=True, readonly=False,
    )
    tax_amount_currency = fields.Monetary(string='Tax amount in Currency', compute='_compute_tax_amount_currency')
    employee_id = fields.Many2one(comodel_name='hr.employee', string="Employee", required=True)
    company_id = fields.Many2one(comodel_name='res.company')
    currency_id = fields.Many2one(comodel_name='res.currency')
    product_has_tax = fields.Boolean(
        string="Whether tax is defined on a selected product",
        compute='_compute_product_has_tax',
    )
    product_has_cost = fields.Boolean(
        string="Is product with non zero cost selected",
        compute='_compute_from_product_id', store=True,
    )

    @api.depends('total_amount_currency', 'tax_ids')
    def _compute_tax_amount_currency(self):
        for split in self:
            taxes = split.tax_ids.with_context(force_price_include=True).compute_all(
                price_unit=split.total_amount_currency,
                currency=split.currency_id,
                quantity=1,
                product=split.product_id
            )
            split.tax_amount_currency = taxes['total_included'] - taxes['total_excluded']

    @api.depends('product_id')
    def _compute_from_product_id(self):
        for split in self:
            split.product_has_cost = split.product_id and (float_compare(split.product_id.standard_price, 0.0, precision_digits=2) != 0)
            if split.product_has_cost:
                split.total_amount_currency = split.product_id._price_compute('standard_price', currency=split.currency_id)[split.product_id.id]

    @api.onchange('product_id')
    def _onchange_product_id(self):
        """
        In case we switch to the product without taxes defined on it, taxes should be removed.
        Computed method won't be good for this purpose, as we don't want to recompute and reset taxes in case they are removed on purpose during splitting.
        """
        if self.product_has_tax and self.tax_ids:
            self.tax_ids = self.tax_ids
        else:
            self.tax_ids = self.product_id.supplier_taxes_id.filtered_domain(self.env['account.tax']._check_company_domain(self.company_id))

    @api.depends('product_id')
    def _compute_product_has_tax(self):
        for split in self:
            split.product_has_tax = split.product_id and split.product_id.supplier_taxes_id.filtered_domain(self.env['account.tax']._check_company_domain(split.company_id))

    def _get_values(self):
        self.ensure_one()
        vals = {
            'name': self.name,
            'product_id': self.product_id.id,
            'total_amount_currency': self.total_amount_currency,
            'total_amount': self.expense_id.currency_id.round(self.expense_id.currency_rate * self.total_amount_currency),
            'tax_ids': [Command.set(self.tax_ids.ids)],
            'analytic_distribution': self.analytic_distribution,
            'employee_id': self.employee_id.id,
            'product_uom_id': self.product_id.uom_id.id,
        }

        account = self.product_id.product_tmpl_id._get_product_accounts()['expense']
        if account:
            vals['account_id'] = account.id
        return vals

```

## File: wizard\hr_expense_split_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _
from odoo.tools import float_compare


class HrExpenseSplitWizard(models.TransientModel):
    _name = 'hr.expense.split.wizard'
    _description = 'Expense Split Wizard'

    expense_id = fields.Many2one(comodel_name='hr.expense', string='Expense', required=True)
    expense_split_line_ids = fields.One2many(comodel_name='hr.expense.split', inverse_name='wizard_id')
    total_amount_currency = fields.Monetary(string='Total Amount', compute='_compute_total_amount_currency', currency_field='currency_id')
    total_amount_currency_original = fields.Monetary(
        string='Total amount original', related='expense_id.total_amount_currency',
        currency_field='currency_id',
        help='Total amount of the original Expense that we are splitting',
    )
    tax_amount_currency = fields.Monetary(
        string='Taxes',
        currency_field='currency_id',
        compute='_compute_tax_amount_currency',
    )
    split_possible = fields.Boolean(help='The sum of after split shut remain the same', compute='_compute_split_possible')
    currency_id = fields.Many2one(comodel_name='res.currency', related='expense_id.currency_id')

    @api.depends('expense_split_line_ids.total_amount_currency')
    def _compute_total_amount_currency(self):
        for wizard in self:
            wizard.total_amount_currency = sum(wizard.expense_split_line_ids.mapped('total_amount_currency'))

    @api.depends('expense_split_line_ids.tax_amount_currency')
    def _compute_tax_amount_currency(self):
        for wizard in self:
            wizard.tax_amount_currency = sum(wizard.expense_split_line_ids.mapped('tax_amount_currency'))

    @api.depends('total_amount_currency_original', 'total_amount_currency')
    def _compute_split_possible(self):
        for wizard in self:
            wizard.split_possible = wizard.total_amount_currency_original \
                    and wizard.currency_id.compare_amounts(wizard.total_amount_currency_original, wizard.total_amount_currency) == 0

    def action_split_expense(self):
        self.ensure_one()
        expense_split = self.expense_split_line_ids[0]
        copied_expenses = self.env["hr.expense"]
        if expense_split:
            self.expense_id.write(expense_split._get_values())

            self.expense_split_line_ids -= expense_split
            if self.expense_split_line_ids:
                for split in self.expense_split_line_ids:
                    copied_expenses |= self.expense_id.copy(split._get_values())

                attachment_ids = self.env['ir.attachment'].search([
                    ('res_model', '=', 'hr.expense'),
                    ('res_id', '=', self.expense_id.id)
                ])

                for copied_expense in copied_expenses:
                    for attachment in attachment_ids:
                        attachment.copy({'res_model': 'hr.expense', 'res_id': copied_expense.id})

        return {
            'type': 'ir.actions.act_window',
            'res_model': 'hr.expense',
            'name': _('Split Expenses'),
            'view_mode': 'tree,form',
            'target': 'current',
            'domain': [('id', 'in', (copied_expenses | self.expense_split_line_ids.expense_id).ids)],
        }

```

## File: wizard\hr_expense_split_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="hr_expense_split" model="ir.ui.view">
            <field name="name">Expense split</field>
            <field name="model">hr.expense.split.wizard</field>
            <field name="arch" type="xml">
                <form>
                    <field name="total_amount_currency_original" invisible="1"/>
                    <field name="expense_id" invisible="1"/>
                    <field name="expense_split_line_ids" widget="one2many" context="{'default_expense_id': expense_id}">
                        <tree editable="bottom">
                            <field name="currency_id" column_invisible="True"/>
                            <field name="expense_id" column_invisible="True"/>
                            <field name="company_id" column_invisible="True"/>
                            <field name="product_has_tax" column_invisible="True"/>
                            <field name="product_has_cost" column_invisible="True"/>
                            <field name="name"/>
                            <field name="product_id"
                                context="{'default_detailed_type': 'service', 'default_can_be_expensed': 1, 'tree_view_ref': 'hr_expense.product_product_expense_tree_view', 'form_view_ref': 'hr_expense.product_product_expense_form_view'}"
                            />
                            <field name="total_amount_currency" force_save="1" readonly="product_has_cost"/>
                            <field name="tax_ids" widget="many2many_tags"/>
                            <field name="tax_amount_currency"/>
                            <field name="analytic_distribution" widget="analytic_distribution"
                                optional="show"
                                groups="analytic.group_analytic_accounting"/>
                            <field name="employee_id" widget="many2one_avatar_user"/>
                        </tree>
                    </field>
                    <field name="currency_id" invisible="1"/>
                    <group class="oe_subtotal_footer" colspan="2" name="expense_total">
                        <label for="total_amount_currency" invisible="split_possible"/>
                        <field name="total_amount_currency" nolabel="1" class="text-danger" invisible="split_possible"/>
                        <field name="total_amount_currency" invisible="not split_possible"/>
                        <field name="total_amount_currency_original" widget='monetary' string="Original Amount"/>
                        <field name="tax_amount_currency" widget='monetary' string="Taxes"/>
                    </group>
                    <field name="split_possible" invisible="1"/>
                    <footer>
                        <button name="action_split_expense" invisible="split_possible" string="Split Expense" type="object" class="oe_highlight" disabled="disabled"  data-hotkey="q"/>
                        <button name="action_split_expense" string="Split Expense" invisible="not split_possible" type="object" class="oe_highlight"  data-hotkey="q"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x"/>
                    </footer>
                </form>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-

from . import hr_expense_refuse_reason
from . import account_payment_register
from . import hr_expense_approve_duplicate
from . import hr_expense_split_wizard
from . import hr_expense_split

```

