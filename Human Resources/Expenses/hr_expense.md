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
    'website': 'https://www.odoo.com/page/expenses',
    'depends': ['hr_contract', 'account', 'web_tour'],
    'data': [
        'security/hr_expense_security.xml',
        'security/ir.model.access.csv',
        'data/digest_data.xml',
        'data/mail_data.xml',
        'data/hr_expense_sequence.xml',
        'data/hr_expense_data.xml',
        'wizard/hr_expense_refuse_reason_views.xml',
        'views/hr_expense_views.xml',
        'views/mail_activity_views.xml',
        'security/ir_rule.xml',
        'report/hr_expense_report.xml',
        'views/hr_department_views.xml',
        'views/assets.xml',
        'views/res_config_settings_views.xml',
        'views/account_journal_dashboard.xml',
    ],
    'demo': ['data/hr_expense_demo.xml'],
    'qweb': [
        "static/src/xml/documents_upload_views.xml",
        "static/src/xml/expense_dashboard.xml",
        "static/src/xml/expense_qr_modal_template.xml",
    ],
    'installable': True,
    'application': True,
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
    <img src="/hr_expense/static/src/img/expense.png" class="illustration_border" />
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
        <record id="product_product_fixed_cost" model="product.product">
            <field name="name">Expenses</field>
            <field name="list_price">0.0</field>
            <field name="standard_price">1.0</field>
            <field name="type">service</field>
            <field name="categ_id" ref="product.cat_expense"/>
            <field name="can_be_expensed" eval="True"/>
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

        <record id="hr.employee_mit" model="hr.employee">
            <field name="address_home_id" ref="base.res_partner_address_3"/>
        </record>

        <record id="hr_expense_account_journal" model="account.journal">
            <field name="name">Expense</field>
            <field name="code">EXP</field>
            <field name="type">purchase</field>
            <!-- avoid being selected as default journal -->
            <field name="sequence">99</field>
            <field name="alias_name">purchase_expense</field>
        </record>

        <record id="res_partner_address_fp" model="res.partner">
            <field name="name">Pieter Parter's Farm</field>
            <field name="parent_id" ref="base.partner_root"/>
            <field name="street">Chaussée de Namur, 40</field>
            <field name="zip">1367</field>
            <field name="city">Grand-Rosière-Hottomont</field>
            <field name="country_id" ref="base.be"/>
            <field name="type">private</field>
        </record>

        <record id="hr.employee_fme" model="hr.employee">
            <field name="address_home_id" ref="res_partner_address_fp"/>
        </record>

        <record id="hr.employee_al" model="hr.employee">
            <field name="address_home_id" ref="res_partner_address_fp"/>
        </record>

        <record id="car_travel" model="product.product">
            <field name="list_price">0.50</field>
            <field name="standard_price">0.32</field>
            <field name="type">service</field>
            <field name="name">Car Travel Expenses</field>
            <field name="uom_id" ref="uom.product_uom_km"/>
            <field name="uom_po_id" ref="uom.product_uom_km"/>
            <field name="categ_id" ref="product.cat_expense"/>
            <field name="can_be_expensed" eval="True" />
            <field name="image_1920" type="base64" file="hr_expense/static/img/car_travel-image.jpg"/>
        </record>

        <record id="air_ticket" model="product.product">
            <field name="list_price">700.0</field>
            <field name="standard_price">700.0</field>
            <field name="type">service</field>
            <field name="name">Air Flight</field>
            <field name="categ_id" ref="product.cat_expense"/>
            <field name="can_be_expensed" eval="True" />
            <field name="image_1920" type="base64" file="hr_expense/static/img/air_ticket-image.jpg"/>
        </record>

        <record id="product.expense_hotel" model="product.product">
            <field name="can_be_expensed" eval="True" />
        </record>

        <record id="product.product_delivery_01" model="product.product">
            <field name="can_be_expensed" eval="True" />
        </record>

        <record id="product.product_delivery_02" model="product.product">
            <field name="can_be_expensed" eval="True" />
        </record>

        <!-- ++++++++++++++ Expense sheet for Admin ++++++++++++++-->

        <record id="screen_expense" model="hr.expense">
            <field name="name">Screen</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="analytic_account_id" ref="analytic.analytic_our_super_product"/>
            <field name="product_id" ref="product_product_fixed_cost"/>
            <field eval="289.0" name="unit_amount"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field eval="1.0" name="quantity"/>
            <field name="date" eval="time.strftime('%Y')+'-04-03'"/>
        </record>

        <record id="laptop_expense" model="hr.expense">
            <field name="name">Laptop</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="analytic_account_id" ref="analytic.analytic_our_super_product"/>
            <field name="product_id" ref="product_product_fixed_cost"/>
            <field eval="889.0" name="unit_amount"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field eval="1.0" name="quantity"/>
            <field name="date" eval="time.strftime('%Y')+'-04-03'"/>
        </record>

        <record id="travel_ny_sheet" model="hr.expense.sheet">
            <field name="name">Commercial Travel at New York</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="state">approve</field>
        </record>

        <record id="travel_by_air_expense" model="hr.expense">
            <field name="name">Travel by Air</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="analytic_account_id" ref="analytic.analytic_our_super_product"/>
            <field name="product_id" ref="air_ticket"/>
            <field eval="700.0" name="unit_amount"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field eval="1.0" name="quantity"/>
            <field name="date" eval="time.strftime('%Y-%m')+'-12'"/>
            <field name="sheet_id" ref="travel_ny_sheet"/>
        </record>

        <record id="hotel_bill_expense" model="hr.expense">
            <field name="name">Hotel Expenses</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="analytic_account_id" ref="analytic.analytic_nebula"/>
            <field name="product_id" ref="product.expense_hotel"/>
            <field eval="400.0" name="unit_amount"/>
            <field name="product_uom_id" ref="uom.product_uom_day"/>
            <field eval="5.0" name="quantity"/>
            <field name="date" eval="time.strftime('%Y-%m')+'-17'"/>
            <field name="sheet_id" ref="travel_ny_sheet"/>
        </record>

        <record id="lunch_customer_bill_expense" model="hr.expense">
            <field name="name">Lunch with Customer</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="analytic_account_id" ref="analytic.analytic_nebula"/>
            <field name="product_id" ref="product_product_fixed_cost"/>
            <field eval="152.8" name="unit_amount"/>
            <field name="date" eval="time.strftime('%Y-%m')+'-13'"/>
            <field eval="1.0" name="quantity"/>
            <field name="sheet_id" ref="travel_ny_sheet"/>
        </record>

        <record id="lunch_bill_expense" model="hr.expense">
            <field name="name">Lunch</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="analytic_account_id" ref="analytic.analytic_nebula"/>
            <field name="product_id" ref="product_product_fixed_cost"/>
            <field name="date" eval="time.strftime('%Y-%m')+'-15'"/>
            <field eval="56.8" name="unit_amount"/>
            <field eval="1.0" name="quantity"/>
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
            <field name="analytic_account_id" ref="analytic.analytic_our_super_product"/>
            <field name="product_id" ref="car_travel"/>
            <field eval="0.52" name="unit_amount"/>
            <field name="product_uom_id" ref="uom.product_uom_km"/>
            <field name="date" eval="time.strftime('%Y')+'-01-15'"/>
            <field eval="152.0" name="quantity"/>
            <field name="sheet_id" ref="customer_meeting_sheet"/>
        </record>

        <record id="lunch_demo_customer_bill_expense" model="hr.expense">
            <field name="name">Lunch with Customer</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="analytic_account_id" ref="analytic.analytic_nebula"/>
            <field name="product_id" ref="product_product_fixed_cost"/>
            <field name="date" eval="time.strftime('%Y')+'-01-15'"/>
            <field eval="152.8" name="unit_amount"/>
            <field eval="1.0" name="quantity"/>
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
            <field name="analytic_account_id" ref="analytic.analytic_nebula"/>
            <field name="product_id" ref="product_product_fixed_cost"/>
            <field name="date" eval="time.strftime('%Y-%m')+'-05'"/>
            <field eval="12.5" name="unit_amount"/>
            <field eval="12.0" name="quantity"/>
            <field name="sheet_id" ref="team_building_sheet"/>
        </record>

        <record id="drinks_bill_expense" model="hr.expense">
            <field name="name">Drinks</field>
            <field name="employee_id" ref="hr.employee_fme"/>
            <field name="analytic_account_id" ref="analytic.analytic_nebula"/>
            <field name="product_id" ref="product_product_fixed_cost"/>
            <field name="date" eval="time.strftime('%Y-%m')+'-05'"/>
            <field eval="2.5" name="unit_amount"/>
            <field eval="17.0" name="quantity"/>
            <field name="sheet_id" ref="team_building_sheet"/>
        </record>

        <record id="paintball_bill_expense" model="hr.expense">
            <field name="name">Paintball</field>
            <field name="employee_id" ref="hr.employee_fme"/>
            <field name="analytic_account_id" ref="analytic.analytic_nebula"/>
            <field name="product_id" ref="product_product_fixed_cost"/>
            <field name="date" eval="time.strftime('%Y-%m')+'-05'"/>
            <field eval="25" name="unit_amount"/>
            <field eval="12.0" name="quantity"/>
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
            <field name="analytic_account_id" ref="analytic.analytic_nebula"/>
            <field name="product_id" ref="product.product_delivery_01"/>
            <field name="date" eval="time.strftime('%Y')+'-06-02'"/>
            <field eval="55.75" name="unit_amount"/>
            <field eval="6.0" name="quantity"/>
            <field name="sheet_id" ref="office_furniture_sheet"/>
        </record>

        <record id="lamp_bill_expense" model="hr.expense">
            <field name="name">Lamp</field>
            <field name="employee_id" ref="hr.employee_al"/>
            <field name="analytic_account_id" ref="analytic.analytic_nebula"/>
            <field name="product_id" ref="product.product_delivery_02"/>
            <field name="date" eval="time.strftime('%Y')+'-06-02'"/>
            <field eval="28.99" name="unit_amount"/>
            <field eval="1.0" name="quantity"/>
            <field name="sheet_id" ref="office_furniture_sheet"/>
        </record>

        <!-- ++++++++++++++ Expense for Randall Lewis ++++++++++++++-->
        <record id="afterwork_bill_expense" model="hr.expense">
            <field name="name">Car tyres</field>
            <field name="employee_id" ref="hr.employee_stw"/>
            <field name="analytic_account_id" ref="analytic.analytic_nebula"/>
            <field name="product_id" ref="product_product_fixed_cost"/>
            <field name="date" eval="time.strftime('%Y')+'-03-15'"/>
            <field eval="112.58" name="unit_amount"/>
            <field eval="4.0" name="quantity"/>
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

## File: data\mail_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="mail_act_expense_approval" model="mail.activity.type">
            <field name="name">Expense Approval</field>
            <field name="icon">fa-dollar</field>
            <field name="res_model_id" ref="hr_expense.model_hr_expense_sheet"/>
        </record>

        <!-- default alias for expenses -->
        <record id="mail_alias_expense" model="mail.alias">
            <field name="alias_name">expense</field>
            <field name="alias_model_id" ref="model_hr_expense"/>
            <field name="alias_user_id" ref="base.user_admin"/>
            <field name="alias_contact">employees</field>
        </record>

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

        <!-- Email Templates -->
        <template id="hr_expense_template_refuse_reason">
            <p>Your Expense <t t-if="is_sheet">Report </t><t t-esc="name"/> has been refused</p>
            <ul class="o_timeline_tracking_value_list">
                <li>Reason : <t t-esc="reason"/></li>
            </ul>
        </template>

        <template id="hr_expense_template_register">
            <p>Dear <t t-esc="expense.employee_id.name"/>,</p>
            <p>
                Your expense has been successfully registered.
                <t t-if="expense.employee_id.user_id">
                    You can now submit it to the manager from the following link.
                </t>
            </p>
            <p t-if="expense.product_id">
                Product: <t t-esc="expense.product_id.name"/>
            </p>
            <div t-else="">
                <p>Product: not found</p>
                <p>The first word of the email subject did not correspond to any product code. You'll have to set the product manually on the expense.</p>
            </div>
            <p>
                Price: <t t-esc="expense.unit_amount"/><t t-esc="expense.currency_id.symbol"/>
            </p>
            <p t-if="expense.employee_id.user_id">
                <br/>
                <a t-att-href="'/web#id=%s&amp;model=hr.expense&amp;view_type=form' % (expense.id)" style="background-color: #9E588B; margin-top: 10px; padding: 10px; text-decoration: none; color: #fff; border-radius: 5px; font-size: 16px;">View Expense</a>
            </p>
        </template>

        <template id="hr_expense_template_register_no_user">
            <div style="background:#F0F0F0;color:#515166;padding:10px 0px;font-family:Arial,Helvetica,sans-serif;font-size:14px;">
                <table style="width:600px;margin:5px auto;">
                    <tbody>
                        <tr>
                            <td><a href="/"><img src="/web/binary/company_logo" style="vertical-align:baseline;max-width:100px;" /></a></td>
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
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models
from odoo.tools.misc import formatLang


class AccountJournal(models.Model):
    _inherit = "account.journal"

    def _get_expenses_to_pay_query(self):
        """
        Returns a tuple containing as it's first element the SQL query used to
        gather the expenses in reported state data, and the arguments
        dictionary to use to run it as it's second.
        """
        query = """SELECT total_amount as amount_total, currency_id AS currency
                  FROM hr_expense_sheet
                  WHERE state IN ('approve', 'post')
                  and journal_id = %(journal_id)s"""
        return (query, {'journal_id': self.id})

    def get_journal_dashboard_datas(self):
        res = super(AccountJournal, self).get_journal_dashboard_datas()
        #add the number and sum of expenses to pay to the json defining the accounting dashboard data
        (query, query_args) = self._get_expenses_to_pay_query()
        self.env.cr.execute(query, query_args)
        query_results_to_pay = self.env.cr.dictfetchall()
        (number_to_pay, sum_to_pay) = self._count_results_and_sum_amounts(query_results_to_pay, self.company_id.currency_id)
        res['number_expenses_to_pay'] = number_to_pay
        res['sum_expenses_to_pay'] = formatLang(self.env, sum_to_pay or 0.0, currency_obj=self.currency_id or self.company_id.currency_id)
        return res

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
        return action

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _


class AccountMove(models.Model):
    _inherit = "account.move"

    def button_cancel(self):
        for l in self.line_ids:
            if l.expense_id:
                l.expense_id.refuse_expense(reason=_("Payment Cancelled"))
        return super().button_cancel()

    def button_draft(self):
        for line in self.line_ids:
            if line.expense_id:
                line.expense_id.sheet_id.write({'state': 'post'})
        return super().button_draft()

```

## File: models\account_move_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class AccountMoveLine(models.Model):
    _inherit = "account.move.line"

    expense_id = fields.Many2one('hr.expense', string='Expense', copy=False, help="Expense where the move line come from")

    def reconcile(self):
        # OVERRIDE
        not_paid_expenses = self.expense_id.filtered(lambda expense: expense.state != 'done')
        not_paid_expense_sheets = not_paid_expenses.sheet_id
        res = super().reconcile()
        paid_expenses = not_paid_expenses.filtered(lambda expense: expense.currency_id.is_zero(expense.amount_residual))
        paid_expenses.write({'state': 'done'})
        not_paid_expense_sheets.filtered(lambda sheet: all(expense.state == 'done' for expense in sheet.expense_line_ids)).set_to_paid()
        return res

    def _get_attachment_domains(self):
        attachment_domains = super(AccountMoveLine, self)._get_attachment_domains()
        if self.expense_id:
            attachment_domains.append([('res_model', '=', 'hr.expense'), ('res_id', '=', self.expense_id.id)])
        return attachment_domains

```

## File: models\hr_department.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class HrDepartment(models.Model):
    _inherit = 'hr.department'

    def _compute_expense_sheets_to_approve(self):
        expense_sheet_data = self.env['hr.expense.sheet'].read_group([('department_id', 'in', self.ids), ('state', '=', 'submit')], ['department_id'], ['department_id'])
        result = dict((data['department_id'][0], data['department_id_count']) for data in expense_sheet_data)
        for department in self:
            department.expense_sheets_to_approve_count = result.get(department.id, 0)

    expense_sheets_to_approve_count = fields.Integer(compute='_compute_expense_sheets_to_approve', string='Expenses Reports to Approve')

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class Employee(models.Model):
    _inherit = 'hr.employee'

    def _group_hr_expense_user_domain(self):
        # We return the domain only if the group exists for the following reason:
        # When a group is created (at module installation), the `res.users` form view is
        # automatically modifiedto add application accesses. When modifiying the view, it
        # reads the related field `expense_manager_id` of `res.users` and retrieve its domain.
        # This is a problem because the `group_hr_expense_user` record has already been created but
        # not its associated `ir.model.data` which makes `self.env.ref(...)` fail.
        group = self.env.ref('hr_expense.group_hr_expense_team_approver', raise_if_not_found=False)
        return [('groups_id', 'in', group.ids)] if group else []

    expense_manager_id = fields.Many2one(
        'res.users', string='Expense',
        domain=_group_hr_expense_user_domain,
        compute='_compute_expense_manager', store=True, readonly=False,
        help='Select the user responsible for approving "Expenses" of this employee.\n'
             'If empty, the approval is done by an Administrator or Approver (determined in settings/users).')

    @api.depends('parent_id')
    def _compute_expense_manager(self):
        for employee in self:
            previous_manager = employee._origin.parent_id.user_id
            manager = employee.parent_id.user_id
            if manager and manager.has_group('hr_expense.group_hr_expense_user') and (employee.expense_manager_id == previous_manager or not employee.expense_manager_id):
                employee.expense_manager_id = manager
            elif not employee.expense_manager_id:
                employee.expense_manager_id = False


class EmployeePublic(models.Model):
    _inherit = 'hr.employee.public'

    expense_manager_id = fields.Many2one('res.users', readonly=True)


class User(models.Model):
    _inherit = ['res.users']

    expense_manager_id = fields.Many2one(related='employee_id.expense_manager_id', readonly=False)

    def __init__(self, pool, cr):
        """ Override of __init__ to add access rights.
            Access rights are disabled by default, but allowed
            on some specific fields defined in self.SELF_{READ/WRITE}ABLE_FIELDS.
        """
        init_res = super(User, self).__init__(pool, cr)
        # duplicate list to avoid modifying the original reference
        pool[self._name].SELF_READABLE_FIELDS = pool[self._name].SELF_READABLE_FIELDS + ['expense_manager_id']
        return init_res

```

## File: models\hr_expense.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError
from odoo.tools import email_split, float_is_zero


class HrExpense(models.Model):

    _name = "hr.expense"
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _description = "Expense"
    _order = "date desc, id desc"
    _check_company_auto = True

    @api.model
    def _default_employee_id(self):
        employee = self.env.user.employee_id
        if not employee and not self.env.user.has_group('hr_expense.group_hr_expense_team_approver'):
            raise ValidationError(_('The current user has no related employee. Please, create one.'))
        return employee

    @api.model
    def _default_product_uom_id(self):
        return self.env['uom.uom'].search([], limit=1, order='id')

    @api.model
    def _default_account_id(self):
        return self.env['ir.property']._get('property_account_expense_categ_id', 'product.category')

    @api.model
    def _get_employee_id_domain(self):
        res = [('id', '=', 0)] # Nothing accepted by domain, by default
        if self.user_has_groups('hr_expense.group_hr_expense_user') or self.user_has_groups('account.group_account_user'):
            res = "['|', ('company_id', '=', False), ('company_id', '=', company_id)]"  # Then, domain accepts everything
        elif self.user_has_groups('hr_expense.group_hr_expense_team_approver') and self.env.user.employee_ids:
            user = self.env.user
            employee = self.env.user.employee_id
            res = [
                '|', '|', '|',
                ('department_id.manager_id', '=', employee.id),
                ('parent_id', '=', employee.id),
                ('id', '=', employee.id),
                ('expense_manager_id', '=', user.id),
                '|', ('company_id', '=', False), ('company_id', '=', employee.company_id.id),
            ]
        elif self.env.user.employee_id:
            employee = self.env.user.employee_id
            res = [('id', '=', employee.id), '|', ('company_id', '=', False), ('company_id', '=', employee.company_id.id)]
        return res

    name = fields.Char('Description', compute='_compute_from_product_id_company_id', store=True, required=True, copy=True,
        states={'draft': [('readonly', False)], 'reported': [('readonly', False)], 'refused': [('readonly', False)]})
    date = fields.Date(readonly=True, states={'draft': [('readonly', False)], 'reported': [('readonly', False)], 'refused': [('readonly', False)]}, default=fields.Date.context_today, string="Expense Date")
    accounting_date = fields.Date(string="Accounting Date", related='sheet_id.accounting_date', store=True, groups='account.group_account_invoice,account.group_account_readonly')
    employee_id = fields.Many2one('hr.employee', compute='_compute_employee_id', string="Employee",
        store=True, required=True, readonly=False, tracking=True,
        states={'approved': [('readonly', True)], 'done': [('readonly', True)]},
        default=_default_employee_id, domain=lambda self: self._get_employee_id_domain(), check_company=True)
    # product_id not required to allow create an expense without product via mail alias, but should be required on the view.
    product_id = fields.Many2one('product.product', string='Product', readonly=True, tracking=True, states={'draft': [('readonly', False)], 'reported': [('readonly', False)], 'refused': [('readonly', False)]}, domain="[('can_be_expensed', '=', True), '|', ('company_id', '=', False), ('company_id', '=', company_id)]", ondelete='restrict')
    product_uom_id = fields.Many2one('uom.uom', string='Unit of Measure', compute='_compute_from_product_id_company_id',
        store=True, copy=True, states={'draft': [('readonly', False)], 'refused': [('readonly', False)]},
        default=_default_product_uom_id, domain="[('category_id', '=', product_uom_category_id)]")
    product_uom_category_id = fields.Many2one(related='product_id.uom_id.category_id', readonly=True)
    unit_amount = fields.Float("Unit Price", compute='_compute_from_product_id_company_id', store=True, required=True, copy=True,
        states={'draft': [('readonly', False)], 'reported': [('readonly', False)], 'refused': [('readonly', False)]}, digits='Product Price')
    quantity = fields.Float(required=True, readonly=True, states={'draft': [('readonly', False)], 'reported': [('readonly', False)], 'refused': [('readonly', False)]}, digits='Product Unit of Measure', default=1)
    tax_ids = fields.Many2many('account.tax', 'expense_tax', 'expense_id', 'tax_id',
        compute='_compute_from_product_id_company_id', store=True, readonly=False,
        domain="[('company_id', '=', company_id), ('type_tax_use', '=', 'purchase')]", string='Taxes')
    untaxed_amount = fields.Float("Subtotal", store=True, compute='_compute_amount', digits='Account')
    total_amount = fields.Monetary("Total", compute='_compute_amount', store=True, currency_field='currency_id', tracking=True)
    amount_residual = fields.Monetary(string='Amount Due', compute='_compute_amount_residual', compute_sudo=True)
    company_currency_id = fields.Many2one('res.currency', string="Report Company Currency", related='sheet_id.currency_id', store=True, readonly=False)
    total_amount_company = fields.Monetary("Total (Company Currency)", compute='_compute_total_amount_company', store=True, currency_field='company_currency_id')
    company_id = fields.Many2one('res.company', string='Company', required=True, readonly=True, states={'draft': [('readonly', False)], 'refused': [('readonly', False)]}, default=lambda self: self.env.company)
    # TODO make required in master (sgv)
    currency_id = fields.Many2one('res.currency', string='Currency', readonly=True, states={'draft': [('readonly', False)], 'refused': [('readonly', False)]}, default=lambda self: self.env.company.currency_id)
    analytic_account_id = fields.Many2one('account.analytic.account', string='Analytic Account', check_company=True)
    analytic_tag_ids = fields.Many2many('account.analytic.tag', string='Analytic Tags', states={'post': [('readonly', True)], 'done': [('readonly', True)]}, domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    account_id = fields.Many2one('account.account', compute='_compute_from_product_id_company_id', store=True, readonly=False, string='Account',
        default=_default_account_id, domain="[('internal_type', '=', 'other'), ('company_id', '=', company_id)]", help="An expense account is expected")
    description = fields.Text('Notes...', readonly=True, states={'draft': [('readonly', False)], 'reported': [('readonly', False)], 'refused': [('readonly', False)]})
    payment_mode = fields.Selection([
        ("own_account", "Employee (to reimburse)"),
        ("company_account", "Company")
    ], default='own_account', tracking=True, states={'done': [('readonly', True)], 'approved': [('readonly', True)], 'reported': [('readonly', True)]}, string="Paid By")
    attachment_number = fields.Integer('Number of Attachments', compute='_compute_attachment_number')
    state = fields.Selection([
        ('draft', 'To Submit'),
        ('reported', 'Submitted'),
        ('approved', 'Approved'),
        ('done', 'Paid'),
        ('refused', 'Refused')
    ], compute='_compute_state', string='Status', copy=False, index=True, readonly=True, store=True, default='draft', help="Status of the expense.")
    sheet_id = fields.Many2one('hr.expense.sheet', string="Expense Report", domain="[('employee_id', '=', employee_id), ('company_id', '=', company_id)]", readonly=True, copy=False)
    reference = fields.Char("Bill Reference")
    is_refused = fields.Boolean("Explicitly Refused by manager or accountant", readonly=True, copy=False)

    is_editable = fields.Boolean("Is Editable By Current User", compute='_compute_is_editable')
    is_ref_editable = fields.Boolean("Reference Is Editable By Current User", compute='_compute_is_ref_editable')

    sample = fields.Boolean()

    @api.depends('sheet_id', 'sheet_id.account_move_id', 'sheet_id.state')
    def _compute_state(self):
        for expense in self:
            if not expense.sheet_id or expense.sheet_id.state == 'draft':
                expense.state = "draft"
            elif expense.sheet_id.state == "cancel":
                expense.state = "refused"
            elif expense.sheet_id.state == "approve" or expense.sheet_id.state == "post":
                expense.state = "approved"
            elif not expense.sheet_id.account_move_id:
                expense.state = "reported"
            else:
                expense.state = "done"

    @api.depends('quantity', 'unit_amount', 'tax_ids', 'currency_id')
    def _compute_amount(self):
        for expense in self:
            expense.untaxed_amount = expense.unit_amount * expense.quantity
            taxes = expense.tax_ids.compute_all(expense.unit_amount, expense.currency_id, expense.quantity, expense.product_id, expense.employee_id.user_id.partner_id)
            expense.total_amount = taxes.get('total_included')

    @api.depends("sheet_id.account_move_id.line_ids")
    def _compute_amount_residual(self):
        for expense in self:
            if not expense.sheet_id:
                expense.amount_residual = expense.total_amount
                continue
            if not expense.currency_id or expense.currency_id == expense.company_id.currency_id:
                residual_field = 'amount_residual'
            else:
                residual_field = 'amount_residual_currency'
            payment_term_lines = expense.sheet_id.account_move_id.line_ids \
                .filtered(lambda line: line.expense_id == expense and line.account_internal_type in ('receivable', 'payable'))
            expense.amount_residual = -sum(payment_term_lines.mapped(residual_field))

    @api.depends('date', 'total_amount', 'company_currency_id')
    def _compute_total_amount_company(self):
        for expense in self:
            amount = 0
            if expense.company_currency_id:
                date_expense = expense.date
                amount = expense.currency_id._convert(
                    expense.total_amount, expense.company_currency_id,
                    expense.company_id, date_expense or fields.Date.today())
            expense.total_amount_company = amount

    def _compute_attachment_number(self):
        attachment_data = self.env['ir.attachment'].read_group([('res_model', '=', 'hr.expense'), ('res_id', 'in', self.ids)], ['res_id'], ['res_id'])
        attachment = dict((data['res_id'], data['res_id_count']) for data in attachment_data)
        for expense in self:
            expense.attachment_number = attachment.get(expense.id, 0)

    @api.depends('employee_id')
    def _compute_is_editable(self):
        is_account_manager = self.env.user.has_group('account.group_account_user') or self.env.user.has_group('account.group_account_manager')
        for expense in self:
            if expense.state == 'draft' or expense.sheet_id.state in ['draft', 'submit']:
                expense.is_editable = True
            elif expense.sheet_id.state == 'approve':
                expense.is_editable = is_account_manager
            else:
                expense.is_editable = False

    @api.depends('employee_id')
    def _compute_is_ref_editable(self):
        is_account_manager = self.env.user.has_group('account.group_account_user') or self.env.user.has_group('account.group_account_manager')
        for expense in self:
            if expense.state == 'draft' or expense.sheet_id.state in ['draft', 'submit']:
                expense.is_ref_editable = True
            else:
                expense.is_ref_editable = is_account_manager

    @api.depends('product_id', 'company_id')
    def _compute_from_product_id_company_id(self):
        for expense in self.filtered('product_id'):
            expense = expense.with_company(expense.company_id)
            expense.name = expense.name or expense.product_id.display_name
            if not expense.attachment_number or (expense.attachment_number and not expense.unit_amount):
                expense.unit_amount = expense.product_id.price_compute('standard_price')[expense.product_id.id]
            expense.product_uom_id = expense.product_id.uom_id
            expense.tax_ids = expense.product_id.supplier_taxes_id.filtered(lambda tax: tax.company_id == expense.company_id)  # taxes only from the same company
            account = expense.product_id.product_tmpl_id._get_product_accounts()['expense']
            if account:
                expense.account_id = account

    @api.depends('company_id')
    def _compute_employee_id(self):
        if not self.env.context.get('default_employee_id'):
            for expense in self:
                expense.employee_id = self.env.user.with_company(expense.company_id).employee_id

    @api.onchange('product_id', 'date', 'account_id')
    def _onchange_product_id_date_account_id(self):
        rec = self.env['account.analytic.default'].sudo().account_get(
            product_id=self.product_id.id,
            account_id=self.account_id.id,
            company_id=self.company_id.id,
            date=self.date
        )
        self.analytic_account_id = self.analytic_account_id or rec.analytic_id.id
        self.analytic_tag_ids = self.analytic_tag_ids or rec.analytic_tag_ids.ids

    @api.constrains('payment_mode')
    def _check_payment_mode(self):
        self.sheet_id._check_payment_mode()

    @api.constrains('product_id', 'product_uom_id')
    def _check_product_uom_category(self):
        if self.product_id and self.product_uom_id.category_id != self.product_id.uom_id.category_id:
            raise UserError(_('Selected Unit of Measure does not belong to the same category as the product Unit of Measure.'))

    def create_expense_from_attachments(self, attachment_ids=None, view_type='tree'):
        ''' Create the expenses from files.
         :return: An action redirecting to hr.expense tree/form view.
        '''
        if attachment_ids is None:
            attachment_ids = []
        attachments = self.env['ir.attachment'].browse(attachment_ids)
        if not attachments:
            raise UserError(_("No attachment was provided"))
        expenses = self.env['hr.expense']

        if any(attachment.res_id or attachment.res_model != 'hr.expense' for attachment in attachments):
            raise UserError(_("Invalid attachments!"))

        product = self.env['product.product'].search([('can_be_expensed', '=', True)])
        if product:
            product = product.filtered(lambda p: p.default_code == "EXP_GEN") or product[0]
        else:
            raise UserError(_("You need to have at least one product that can be expensed in your database to proceed!"))

        for attachment in attachments:
            vals = {
                'name': attachment.name.split('.')[0],
                'unit_amount': 0,
                'product_id': product.id
            }
            if product.property_account_expense_id:
                vals['account_id'] = product.property_account_expense_id.id

            expense = self.env['hr.expense'].create(vals)
            expense.message_post(body=_('Uploaded Attachment'))
            attachment.write({
                'res_model': 'hr.expense',
                'res_id': expense.id,
            })
            attachment.register_as_main_attachment()
            expenses += expense
        if len(expenses) == 1:
            return {
                'name': _('Generated Expense'),
                'view_mode': 'form',
                'res_model': 'hr.expense',
                'type': 'ir.actions.act_window',
                'views': [[False, 'form']],
                'res_id': expenses[0].id,
            }
        return {
            'name': _('Generated Expenses'),
            'domain': [('id', 'in', expenses.ids)],
            'res_model': 'hr.expense',
            'type': 'ir.actions.act_window',
            'views': [[False, view_type], [False, "form"]],
        }

    # ----------------------------------------
    # ORM Overrides
    # ----------------------------------------

    def unlink(self):
        for expense in self:
            if expense.state in ['done', 'approved']:
                raise UserError(_('You cannot delete a posted or approved expense.'))
        return super(HrExpense, self).unlink()

    def write(self, vals):
        if 'sheet_id' in vals:
            self.env['hr.expense.sheet'].browse(vals['sheet_id']).check_access_rule('write')
        if 'tax_ids' in vals or 'analytic_account_id' in vals or 'account_id' in vals:
            if any(not expense.is_editable for expense in self):
                raise UserError(_('You are not authorized to edit this expense report.'))
        if 'reference' in vals:
            if any(not expense.is_ref_editable for expense in self):
                raise UserError(_('You are not authorized to edit the reference of this expense report.'))
        return super(HrExpense, self).write(vals)

    @api.model
    def get_empty_list_help(self, help_message):
        return super(HrExpense, self).get_empty_list_help(help_message or '' + self._get_empty_list_mail_alias())

    @api.model
    def _get_empty_list_mail_alias(self):
        use_mailgateway = self.env['ir.config_parameter'].sudo().get_param('hr_expense.use_mailgateway')
        alias_record = use_mailgateway and self.env.ref('hr_expense.mail_alias_expense') or False
        if alias_record and alias_record.alias_domain and alias_record.alias_name:
            return """
<p>
Or send your receipts at <a href="mailto:%(email)s?subject=Lunch%%20with%%20customer%%3A%%20%%2412.32">%(email)s</a>.
</p>""" % {'email': '%s@%s' % (alias_record.alias_name, alias_record.alias_domain)}
        return ""

    # ----------------------------------------
    # Actions
    # ----------------------------------------

    def action_view_sheet(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'hr.expense.sheet',
            'target': 'current',
            'res_id': self.sheet_id.id
        }

    def _create_sheet_from_expenses(self):
        if any(expense.state != 'draft' or expense.sheet_id for expense in self):
            raise UserError(_("You cannot report twice the same line!"))
        if len(self.mapped('employee_id')) != 1:
            raise UserError(_("You cannot report expenses for different employees in the same report."))
        if any(not expense.product_id for expense in self):
            raise UserError(_("You can not create report without product."))
        if len(self.company_id) != 1:
            raise UserError(_("You cannot report expenses for different companies in the same report."))

        todo = self.filtered(lambda x: x.payment_mode=='own_account') or self.filtered(lambda x: x.payment_mode=='company_account')
        sheet = self.env['hr.expense.sheet'].create({
            'company_id': self.company_id.id,
            'employee_id': self[0].employee_id.id,
            'name': todo[0].name if len(todo) == 1 else '',
            'expense_line_ids': [(6, 0, todo.ids)]
        })
        return sheet

    def action_submit_expenses(self):
        sheet = self._create_sheet_from_expenses()
        return {
            'name': _('New Expense Report'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'hr.expense.sheet',
            'target': 'current',
            'res_id': sheet.id,
        }

    def action_get_attachment_view(self):
        self.ensure_one()
        res = self.env['ir.actions.act_window']._for_xml_id('base.action_attachment')
        res['domain'] = [('res_model', '=', 'hr.expense'), ('res_id', 'in', self.ids)]
        res['context'] = {'default_res_model': 'hr.expense', 'default_res_id': self.id}
        return res

    # ----------------------------------------
    # Business
    # ----------------------------------------

    def _prepare_move_values(self):
        """
        This function prepares move values related to an expense
        """
        self.ensure_one()
        journal = self.sheet_id.bank_journal_id if self.payment_mode == 'company_account' else self.sheet_id.journal_id
        account_date = self.sheet_id.accounting_date or self.date
        move_values = {
            'journal_id': journal.id,
            'company_id': self.sheet_id.company_id.id,
            'date': account_date,
            'ref': self.sheet_id.name,
            # force the name to the default value, to avoid an eventual 'default_name' in the context
            # to set it to '' which cause no number to be given to the account.move when posted.
            'name': '/',
        }
        return move_values

    def _get_account_move_by_sheet(self):
        """ Return a mapping between the expense sheet of current expense and its account move
            :returns dict where key is a sheet id, and value is an account move record
        """
        move_grouped_by_sheet = {}
        for expense in self:
            # create the move that will contain the accounting entries
            if expense.sheet_id.id not in move_grouped_by_sheet:
                move_vals = expense._prepare_move_values()
                move = self.env['account.move'].with_context(default_journal_id=move_vals['journal_id']).create(move_vals)
                move_grouped_by_sheet[expense.sheet_id.id] = move
            else:
                move = move_grouped_by_sheet[expense.sheet_id.id]
        return move_grouped_by_sheet

    def _get_expense_account_source(self):
        self.ensure_one()
        if self.account_id:
            account = self.account_id
        elif self.product_id:
            account = self.product_id.product_tmpl_id.with_company(self.company_id)._get_product_accounts()['expense']
            if not account:
                raise UserError(
                    _("No Expense account found for the product %s (or for its category), please configure one.") % (self.product_id.name))
        else:
            account = self.env['ir.property'].with_company(self.company_id)._get('property_account_expense_categ_id', 'product.category')
            if not account:
                raise UserError(_('Please configure Default Expense account for Product expense: `property_account_expense_categ_id`.'))
        return account

    def _get_expense_account_destination(self):
        self.ensure_one()
        account_dest = self.env['account.account']
        if self.payment_mode == 'company_account':
            if not self.sheet_id.bank_journal_id.payment_credit_account_id:
                raise UserError(_("No Outstanding Payments Account found for the %s journal, please configure one.") % (self.sheet_id.bank_journal_id.name))
            account_dest = self.sheet_id.bank_journal_id.payment_credit_account_id.id
        else:
            if not self.employee_id.sudo().address_home_id:
                raise UserError(_("No Home Address found for the employee %s, please configure one.") % (self.employee_id.name))
            partner = self.employee_id.sudo().address_home_id.with_company(self.company_id)
            account_dest = partner.property_account_payable_id.id or partner.parent_id.property_account_payable_id.id
        return account_dest

    def _get_account_move_line_values(self):
        move_line_values_by_expense = {}
        for expense in self:
            move_line_name = expense.employee_id.name + ': ' + expense.name.split('\n')[0][:64]
            account_src = expense._get_expense_account_source()
            account_dst = expense._get_expense_account_destination()
            account_date = expense.date or expense.sheet_id.accounting_date or fields.Date.context_today(expense)

            company_currency = expense.company_id.currency_id

            move_line_values = []
            taxes = expense.tax_ids.with_context(round=True).compute_all(expense.unit_amount, expense.currency_id, expense.quantity, expense.product_id)
            total_amount = 0.0
            total_amount_currency = 0.0
            partner_id = expense.employee_id.sudo().address_home_id.commercial_partner_id.id

            # source move line
            balance = expense.currency_id._convert(taxes['total_excluded'], company_currency, expense.company_id, account_date)
            amount_currency = taxes['total_excluded']
            move_line_src = {
                'name': move_line_name,
                'quantity': expense.quantity or 1,
                'debit': balance if balance > 0 else 0,
                'credit': -balance if balance < 0 else 0,
                'amount_currency': amount_currency,
                'account_id': account_src.id,
                'product_id': expense.product_id.id,
                'product_uom_id': expense.product_uom_id.id,
                'analytic_account_id': expense.analytic_account_id.id,
                'analytic_tag_ids': [(6, 0, expense.analytic_tag_ids.ids)],
                'expense_id': expense.id,
                'partner_id': partner_id,
                'tax_ids': [(6, 0, expense.tax_ids.ids)],
                'tax_tag_ids': [(6, 0, taxes['base_tags'])],
                'currency_id': expense.currency_id.id,
            }
            move_line_values.append(move_line_src)
            total_amount -= balance
            total_amount_currency -= move_line_src['amount_currency']

            # taxes move lines
            for tax in taxes['taxes']:
                balance = expense.currency_id._convert(tax['amount'], company_currency, expense.company_id, account_date)
                amount_currency = tax['amount']

                if tax['tax_repartition_line_id']:
                    rep_ln = self.env['account.tax.repartition.line'].browse(tax['tax_repartition_line_id'])
                    base_amount = self.env['account.move']._get_base_amount_to_display(tax['base'], rep_ln)
                    base_amount = expense.currency_id._convert(base_amount, company_currency, expense.company_id, account_date)
                else:
                    base_amount = None

                move_line_tax_values = {
                    'name': tax['name'],
                    'quantity': 1,
                    'debit': balance if balance > 0 else 0,
                    'credit': -balance if balance < 0 else 0,
                    'amount_currency': amount_currency,
                    'account_id': tax['account_id'] or move_line_src['account_id'],
                    'tax_repartition_line_id': tax['tax_repartition_line_id'],
                    'tax_tag_ids': tax['tag_ids'],
                    'tax_base_amount': base_amount,
                    'expense_id': expense.id,
                    'partner_id': partner_id,
                    'currency_id': expense.currency_id.id,
                    'analytic_account_id': expense.analytic_account_id.id if tax['analytic'] else False,
                    'analytic_tag_ids': [(6, 0, expense.analytic_tag_ids.ids)] if tax['analytic'] else False,
                }
                total_amount -= balance
                total_amount_currency -= move_line_tax_values['amount_currency']
                move_line_values.append(move_line_tax_values)

            # destination move line
            move_line_dst = {
                'name': move_line_name,
                'debit': total_amount > 0 and total_amount,
                'credit': total_amount < 0 and -total_amount,
                'account_id': account_dst,
                'date_maturity': account_date,
                'amount_currency': total_amount_currency,
                'currency_id': expense.currency_id.id,
                'expense_id': expense.id,
                'partner_id': partner_id,
            }
            move_line_values.append(move_line_dst)

            move_line_values_by_expense[expense.id] = move_line_values
        return move_line_values_by_expense

    def action_move_create(self):
        '''
        main function that is called when trying to create the accounting entries related to an expense
        '''
        move_group_by_sheet = self._get_account_move_by_sheet()

        move_line_values_by_expense = self._get_account_move_line_values()

        for expense in self:
            # get the account move of the related sheet
            move = move_group_by_sheet[expense.sheet_id.id]

            # get move line values
            move_line_values = move_line_values_by_expense.get(expense.id)

            # link move lines to move, and move to expense sheet
            move.write({'line_ids': [(0, 0, line) for line in move_line_values]})
            expense.sheet_id.write({'account_move_id': move.id})

            if expense.payment_mode == 'company_account':
                expense.sheet_id.paid_expense_sheets()

        # post the moves
        for move in move_group_by_sheet.values():
            move._post()

        return move_group_by_sheet

    def refuse_expense(self, reason):
        self.write({'is_refused': True})
        self.sheet_id.write({'state': 'cancel'})
        self.sheet_id.message_post_with_view('hr_expense.hr_expense_template_refuse_reason',
                                             values={'reason': reason, 'is_sheet': False, 'name': self.name})

    @api.model
    def get_expense_dashboard(self):
        expense_state = {
            'draft': {
                'description': _('to report'),
                'amount': 0.0,
                'currency': self.env.company.currency_id.id,
            },
            'reported': {
                'description': _('under validation'),
                'amount': 0.0,
                'currency': self.env.company.currency_id.id,
            },
            'approved': {
                'description': _('to be reimbursed'),
                'amount': 0.0,
                'currency': self.env.company.currency_id.id,
            }
        }
        if not self.env.user.employee_ids:
            return expense_state
        target_currency = self.env.company.currency_id
        expenses = self.read_group(
            [
                ('employee_id', 'in', self.env.user.employee_ids.ids),
                ('payment_mode', '=', 'own_account'),
                ('state', 'in', ['draft', 'reported', 'approved'])
            ], ['total_amount', 'currency_id', 'state'], ['state', 'currency_id'], lazy=False)
        for expense in expenses:
            state = expense['state']
            currency = self.env['res.currency'].browse(expense['currency_id'][0]) if expense['currency_id'] else target_currency
            amount = currency._convert(
                    expense['total_amount'], target_currency, self.env.company, fields.Date.today())
            expense_state[state]['amount'] += amount
        return expense_state

    # ----------------------------------------
    # Mail Thread
    # ----------------------------------------

    @api.model
    def message_new(self, msg_dict, custom_values=None):
        email_address = email_split(msg_dict.get('email_from', False))[0]

        employee = self.env['hr.employee'].search([
            '|',
            ('work_email', 'ilike', email_address),
            ('user_id.email', 'ilike', email_address)
        ], limit=1)

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
            'unit_amount': price,
            'product_id': product.id if product else None,
            'product_uom_id': product.uom_id.id,
            'tax_ids': [(4, tax.id, False) for tax in product.supplier_taxes_id.filtered(lambda r: r.company_id == company)],
            'quantity': 1,
            'company_id': company.id,
            'currency_id': currency_id.id
        }

        account = product.product_tmpl_id._get_product_accounts()['expense']
        if account:
            vals['account_id'] = account.id

        expense = super(HrExpense, self).message_new(msg_dict, dict(custom_values or {}, **vals))
        self._send_expense_success_mail(msg_dict, expense)
        return expense

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
        symbols, symbols_pattern, float_pattern = [], '', '[+-]?(\d+[.,]?\d*)'
        price = 0.0
        for currency in currencies:
            symbols.append(re.escape(currency.symbol))
            symbols.append(re.escape(currency.name))
        symbols_pattern = '|'.join(symbols)
        price_pattern = "((%s)?\s?%s\s?(%s)?)" % (symbols_pattern, float_pattern, symbols_pattern)
        matches = re.findall(price_pattern, expense_description)
        currency = currencies and currencies[0]
        if matches:
            match = max(matches, key=lambda match: len([group for group in match if group])) # get the longuest match. e.g. "2 chairs 120$" -> the price is 120$, not 2
            full_str = match[0]
            currency_str = match[1] or match[3]
            price = match[2].replace(',', '.')

            if currency_str and currencies:
                currencies = currencies.filtered(lambda c: currency_str in [c.symbol, c.name])
                currency = (currencies and currencies[0]) or currency
            expense_description = expense_description.replace(full_str, ' ') # remove price from description
            expense_description = re.sub(' +', ' ', expense_description.strip())

        price = float(price)
        return price, currency, expense_description

    @api.model
    def _parse_expense_subject(self, expense_description, currencies):
        """ Fetch product, price and currency info from mail subject.

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

    # TODO: Make api.multi
    def _send_expense_success_mail(self, msg_dict, expense):
        mail_template_id = 'hr_expense.hr_expense_template_register' if expense.employee_id.user_id else 'hr_expense.hr_expense_template_register_no_user'
        expense_template = self.env.ref(mail_template_id)
        rendered_body = expense_template._render({'expense': expense}, engine='ir.qweb')
        body = self.env['mail.render.mixin']._replace_local_links(rendered_body)
        # TDE TODO: seems louche, check to use notify
        if expense.employee_id.user_id.partner_id:
            expense.message_post(
                partner_ids=expense.employee_id.user_id.partner_id.ids,
                subject='Re: %s' % msg_dict.get('subject', ''),
                body=body,
                subtype_id=self.env.ref('mail.mt_note').id,
                email_layout_xmlid='mail.mail_notification_light',
            )
        else:
            self.env['mail.mail'].sudo().create({
                'email_from': self.env.user.email_formatted,
                'author_id': self.env.user.partner_id.id,
                'body_html': body,
                'subject': 'Re: %s' % msg_dict.get('subject', ''),
                'email_to': msg_dict.get('email_from', False),
                'auto_delete': True,
                'references': msg_dict.get('message_id'),
            }).send()


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
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _description = "Expense Report"
    _order = "accounting_date desc, id desc"
    _check_company_auto = True

    @api.model
    def _default_employee_id(self):
        return self.env.user.employee_id

    @api.model
    def _default_journal_id(self):
        """ The journal is determining the company of the accounting entries generated from expense. We need to force journal company and expense sheet company to be the same. """
        default_company_id = self.default_get(['company_id'])['company_id']
        journal = self.env['account.journal'].search([('type', '=', 'purchase'), ('company_id', '=', default_company_id)], limit=1)
        return journal.id

    @api.model
    def _default_bank_journal_id(self):
        default_company_id = self.default_get(['company_id'])['company_id']
        return self.env['account.journal'].search([('type', 'in', ['cash', 'bank']), ('company_id', '=', default_company_id)], limit=1)

    name = fields.Char('Expense Report Summary', required=True, tracking=True)
    expense_line_ids = fields.One2many(
        comodel_name='hr.expense',
        inverse_name='sheet_id',
        string='Expense Lines',
        copy=False,
        states={'post': [('readonly', True)], 'done': [('readonly', True)], 'cancel': [('readonly', True)]}
    )
    state = fields.Selection([
        ('draft', 'Draft'),
        ('submit', 'Submitted'),
        ('approve', 'Approved'),
        ('post', 'Posted'),
        ('done', 'Paid'),
        ('cancel', 'Refused')
    ], string='Status', index=True, readonly=True, tracking=True, copy=False, default='draft', required=True, help='Expense Report State')
    employee_id = fields.Many2one('hr.employee', string="Employee", required=True, readonly=True, tracking=True, states={'draft': [('readonly', False)]}, default=_default_employee_id, check_company=True, domain= lambda self: self.env['hr.expense']._get_employee_id_domain())
    address_id = fields.Many2one('res.partner', compute='_compute_from_employee_id', store=True, readonly=False, copy=True, string="Employee Home Address", check_company=True)
    payment_mode = fields.Selection(related='expense_line_ids.payment_mode', default='own_account', readonly=True, string="Paid By", tracking=True)
    user_id = fields.Many2one('res.users', 'Manager', compute='_compute_from_employee_id', store=True, readonly=True, copy=False, states={'draft': [('readonly', False)]}, tracking=True, domain=lambda self: [('groups_id', 'in', self.env.ref('hr_expense.group_hr_expense_team_approver').id)])
    total_amount = fields.Monetary('Total Amount', currency_field='currency_id', compute='_compute_amount', store=True, tracking=True)
    amount_residual = fields.Monetary(
        string="Amount Due", store=True,
        currency_field='currency_id',
        compute='_compute_amount_residual')
    company_id = fields.Many2one('res.company', string='Company', required=True, readonly=True, states={'draft': [('readonly', False)]}, default=lambda self: self.env.company)
    currency_id = fields.Many2one('res.currency', string='Currency', readonly=True, states={'draft': [('readonly', False)]}, default=lambda self: self.env.company.currency_id)
    attachment_number = fields.Integer(compute='_compute_attachment_number', string='Number of Attachments')
    journal_id = fields.Many2one('account.journal', string='Expense Journal', states={'done': [('readonly', True)], 'post': [('readonly', True)]}, check_company=True, domain="[('type', '=', 'purchase'), ('company_id', '=', company_id)]",
        default=_default_journal_id, help="The journal used when the expense is done.")
    bank_journal_id = fields.Many2one('account.journal', string='Bank Journal', states={'done': [('readonly', True)], 'post': [('readonly', True)]}, check_company=True, domain="[('type', 'in', ['cash', 'bank']), ('company_id', '=', company_id)]",
        default=_default_bank_journal_id, help="The payment method used when the expense is paid by the company.")
    accounting_date = fields.Date("Accounting Date")
    account_move_id = fields.Many2one('account.move', string='Journal Entry', ondelete='restrict', copy=False, readonly=True)
    department_id = fields.Many2one('hr.department', compute='_compute_from_employee_id', store=True, readonly=False, copy=False, string='Department', states={'post': [('readonly', True)], 'done': [('readonly', True)]})
    is_multiple_currency = fields.Boolean("Handle lines with different currencies", compute='_compute_is_multiple_currency')
    can_reset = fields.Boolean('Can Reset', compute='_compute_can_reset')

    _sql_constraints = [
        ('journal_id_required_posted', "CHECK((state IN ('post', 'done') AND journal_id IS NOT NULL) OR (state NOT IN ('post', 'done')))", 'The journal must be set on posted expense'),
    ]

    @api.depends('expense_line_ids.total_amount_company')
    def _compute_amount(self):
        for sheet in self:
            sheet.total_amount = sum(sheet.expense_line_ids.mapped('total_amount_company'))

    @api.depends(
        'currency_id',
        'account_move_id.line_ids.amount_residual',
        'account_move_id.line_ids.amount_residual_currency',
        'account_move_id.line_ids.account_internal_type',)
    def _compute_amount_residual(self):
        for sheet in self:
            if sheet.currency_id == sheet.company_id.currency_id:
                residual_field = 'amount_residual'
            else:
                residual_field = 'amount_residual_currency'

            payment_term_lines = sheet.account_move_id.line_ids\
                .filtered(lambda line: line.account_internal_type in ('receivable', 'payable'))
            sheet.amount_residual = -sum(payment_term_lines.mapped(residual_field))

    def _compute_attachment_number(self):
        for sheet in self:
            sheet.attachment_number = sum(sheet.expense_line_ids.mapped('attachment_number'))

    @api.depends('expense_line_ids.currency_id')
    def _compute_is_multiple_currency(self):
        for sheet in self:
            sheet.is_multiple_currency = len(sheet.expense_line_ids.mapped('currency_id')) > 1

    def _compute_can_reset(self):
        is_expense_user = self.user_has_groups('hr_expense.group_hr_expense_team_approver')
        for sheet in self:
            sheet.can_reset = is_expense_user if is_expense_user else sheet.employee_id.user_id == self.env.user

    @api.depends('employee_id')
    def _compute_from_employee_id(self):
        for sheet in self:
            sheet.address_id = sheet.employee_id.sudo().address_home_id
            sheet.department_id = sheet.employee_id.department_id
            sheet.user_id = sheet.employee_id.expense_manager_id or sheet.employee_id.parent_id.user_id

    @api.constrains('expense_line_ids')
    def _check_payment_mode(self):
        for sheet in self:
            expense_lines = sheet.mapped('expense_line_ids')
            if expense_lines and any(expense.payment_mode != expense_lines[0].payment_mode for expense in expense_lines):
                raise ValidationError(_("Expenses must be paid by the same entity (Company or employee)."))

    @api.constrains('expense_line_ids', 'employee_id')
    def _check_employee(self):
        for sheet in self:
            employee_ids = sheet.expense_line_ids.mapped('employee_id')
            if len(employee_ids) > 1 or (len(employee_ids) == 1 and employee_ids != sheet.employee_id):
                raise ValidationError(_('You cannot add expenses of another employee.'))

    @api.constrains('expense_line_ids', 'company_id')
    def _check_expense_lines_company(self):
        for sheet in self:
            if any(expense.company_id != sheet.company_id for expense in sheet.expense_line_ids):
                raise ValidationError(_('An expense report must contain only lines from the same company.'))

    @api.model
    def create(self, vals):
        sheet = super(HrExpenseSheet, self.with_context(mail_create_nosubscribe=True, mail_auto_subscribe_no_notify=True)).create(vals)
        sheet.activity_update()
        return sheet

    def unlink(self):
        for expense in self:
            if expense.state in ['post', 'done']:
                raise UserError(_('You cannot delete a posted or paid expense.'))
        super(HrExpenseSheet, self).unlink()

    # --------------------------------------------
    # Mail Thread
    # --------------------------------------------

    def _track_subtype(self, init_values):
        self.ensure_one()
        if 'state' in init_values and self.state == 'approve':
            return self.env.ref('hr_expense.mt_expense_approved')
        elif 'state' in init_values and self.state == 'cancel':
            return self.env.ref('hr_expense.mt_expense_refused')
        elif 'state' in init_values and self.state == 'done':
            return self.env.ref('hr_expense.mt_expense_paid')
        return super(HrExpenseSheet, self)._track_subtype(init_values)

    def _message_auto_subscribe_followers(self, updated_values, subtype_ids):
        res = super(HrExpenseSheet, self)._message_auto_subscribe_followers(updated_values, subtype_ids)
        if updated_values.get('employee_id'):
            employee = self.env['hr.employee'].browse(updated_values['employee_id'])
            if employee.user_id:
                res.append((employee.user_id.partner_id.id, subtype_ids, False))
        return res

    # --------------------------------------------
    # Actions
    # --------------------------------------------

    def action_sheet_move_create(self):
        samples = self.mapped('expense_line_ids.sample')
        if samples.count(True):
            if samples.count(False):
                raise UserError(_("You can't mix sample expenses and regular ones"))
            self.write({'state': 'post'})
            return

        if any(sheet.state != 'approve' for sheet in self):
            raise UserError(_("You can only generate accounting entry for approved expense(s)."))

        if any(not sheet.journal_id for sheet in self):
            raise UserError(_("Expenses must have an expense journal specified to generate accounting entries."))

        expense_line_ids = self.mapped('expense_line_ids')\
            .filtered(lambda r: not float_is_zero(r.total_amount, precision_rounding=(r.currency_id or self.env.company.currency_id).rounding))
        res = expense_line_ids.action_move_create()
        for sheet in self.filtered(lambda s: not s.accounting_date):
            sheet.accounting_date = sheet.account_move_id.date
        to_post = self.filtered(lambda sheet: sheet.payment_mode == 'own_account' and sheet.expense_line_ids)
        to_post.write({'state': 'post'})
        (self - to_post).write({'state': 'done'})
        self.activity_update()
        return res

    def action_get_attachment_view(self):
        res = self.env['ir.actions.act_window']._for_xml_id('base.action_attachment')
        res['domain'] = [('res_model', '=', 'hr.expense'), ('res_id', 'in', self.expense_line_ids.ids)]
        res['context'] = {
            'default_res_model': 'hr.expense.sheet',
            'default_res_id': self.id,
            'create': False,
            'edit': False,
        }
        return res

    # --------------------------------------------
    # Business
    # --------------------------------------------

    def set_to_paid(self):
        self.write({'state': 'done'})

    def action_submit_sheet(self):
        self.write({'state': 'submit'})
        self.activity_update()

    def approve_expense_sheets(self):
        if not self.user_has_groups('hr_expense.group_hr_expense_team_approver'):
            raise UserError(_("Only Managers and HR Officers can approve expenses"))
        elif not self.user_has_groups('hr_expense.group_hr_expense_manager'):
            current_managers = self.employee_id.expense_manager_id | self.employee_id.parent_id.user_id | self.employee_id.department_id.manager_id.user_id

            if self.employee_id.user_id == self.env.user:
                raise UserError(_("You cannot approve your own expenses"))

            if not self.env.user in current_managers and not self.user_has_groups('hr_expense.group_hr_expense_user') and self.employee_id.expense_manager_id != self.env.user:
                raise UserError(_("You can only approve your department expenses"))
        
        notification = {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'title': _('There are no expense reports to approve.'),
                'type': 'warning',
                'sticky': False,  #True/False will display for few seconds if false
            },
        }
        filtered_sheet = self.filtered(lambda s: s.state in ['submit', 'draft'])
        if not filtered_sheet:
            return notification
        for sheet in filtered_sheet:
            sheet.write({'state': 'approve', 'user_id': sheet.user_id.id or self.env.user.id})
        notification['params'].update({
            'title': _('The expense reports were successfully approved.'),
            'type': 'success',
            'next': {'type': 'ir.actions.act_window_close'},
        })
            
        self.activity_update()
        return notification

    def paid_expense_sheets(self):
        self.write({'state': 'done'})

    def refuse_sheet(self, reason):
        if not self.user_has_groups('hr_expense.group_hr_expense_team_approver'):
            raise UserError(_("Only Managers and HR Officers can approve expenses"))
        elif not self.user_has_groups('hr_expense.group_hr_expense_manager'):
            current_managers = self.employee_id.expense_manager_id | self.employee_id.parent_id.user_id | self.employee_id.department_id.manager_id.user_id

            if self.employee_id.user_id == self.env.user:
                raise UserError(_("You cannot refuse your own expenses"))

            if not self.env.user in current_managers and not self.user_has_groups('hr_expense.group_hr_expense_user') and self.employee_id.expense_manager_id != self.env.user:
                raise UserError(_("You can only refuse your department expenses"))

        self.write({'state': 'cancel'})
        for sheet in self:
            sheet.message_post_with_view('hr_expense.hr_expense_template_refuse_reason', values={'reason': reason, 'is_sheet': True, 'name': sheet.name})
        self.activity_update()

    def reset_expense_sheets(self):
        if not self.can_reset:
            raise UserError(_("Only HR Officers or the concerned employee can reset to draft."))
        self.mapped('expense_line_ids').write({'is_refused': False})
        self.write({'state': 'draft'})
        self.activity_update()
        return True

    def _get_responsible_for_approval(self):
        if self.user_id:
            return self.user_id
        elif self.employee_id.parent_id.user_id:
            return self.employee_id.parent_id.user_id
        elif self.employee_id.department_id.manager_id.user_id:
            return self.employee_id.department_id.manager_id.user_id
        return self.env['res.users']

    def activity_update(self):
        for expense_report in self.filtered(lambda hol: hol.state == 'submit'):
            self.activity_schedule(
                'hr_expense.mail_act_expense_approval',
                user_id=expense_report.sudo()._get_responsible_for_approval().id or self.env.user.id)
        self.filtered(lambda hol: hol.state == 'approve').activity_feedback(['hr_expense.mail_act_expense_approval'])
        self.filtered(lambda hol: hol.state in ('draft', 'cancel')).activity_unlink(['hr_expense.mail_act_expense_approval'])

    def action_register_payment(self):
        ''' Open the account.payment.register wizard to pay the selected journal entries.
        :return: An action opening the account.payment.register wizard.
        '''
        return {
            'name': _('Register Payment'),
            'res_model': 'account.payment.register',
            'view_mode': 'form',
            'context': {
                'active_model': 'account.move',
                'active_ids': self.account_move_id.ids,
            },
            'target': 'new',
            'type': 'ir.actions.act_window',
        }

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.tools.sql import column_exists, create_column


class ProductTemplate(models.Model):
    _inherit = "product.template"

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

    @api.model
    def default_get(self, fields):
        result = super(ProductTemplate, self).default_get(fields)
        if self.env.context.get('default_can_be_expensed'):
            result['supplier_taxes_id'] = False
        return result

    @api.depends('type')
    def _compute_can_be_expensed(self):
        self.filtered(lambda p: p.type not in ['consu', 'service']).update({'can_be_expensed': False})

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    expense_alias_prefix = fields.Char('Default Alias Name for Expenses', compute='_compute_expense_alias_prefix',
        store=True, readonly=False)
    use_mailgateway = fields.Boolean(string='Let your employees record expenses by email',
                                     config_parameter='hr_expense.use_mailgateway')

    module_hr_payroll_expense = fields.Boolean(string='Reimburse Expenses in Payslip')
    module_hr_expense_extract = fields.Boolean(string='Send bills to OCR to generate expenses')


    @api.model
    def get_values(self):
        res = super(ResConfigSettings, self).get_values()
        res.update(
            expense_alias_prefix=self.env.ref('hr_expense.mail_alias_expense').alias_name,
        )
        return res

    def set_values(self):
        super(ResConfigSettings, self).set_values()
        self.env.ref('hr_expense.mail_alias_expense').write({'alias_name': self.expense_alias_prefix})

    @api.depends('use_mailgateway')
    def _compute_expense_alias_prefix(self):
        self.filtered(lambda w: not w.use_mailgateway).update({'expense_alias_prefix': False})

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import hr_employee
from . import account_move
from . import account_move_line
from . import hr_department
from . import hr_expense
from . import product_template
from . import res_config_settings
from . import account_journal_dashboard

```

## File: report\hr_expense_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_expense_sheet">
        <t t-call="web.html_container">
            <t t-foreach="docs" t-as="o">
                <t t-call="web.external_layout">
                    <div class="page">
                        <h2>Expenses Report</h2>

                        <div class="row mt32 mb32">
                            <div class="col-2">
                                <strong>Employee:</strong>
                                <p t-field="o.employee_id.name"/>
                            </div>
                            <div class="col-2">
                                <strong>Date:</strong>
                                <p t-field="o.accounting_date"/>
                            </div>
                            <div class="col-3">
                                <strong>Description:</strong>
                                <p t-field="o.name"/>
                            </div>
                            <div class="col-2">
                                <strong>Validated By:</strong>
                                <p t-field="o.user_id"/>
                            </div>
                            <div class="col-3">
                                <strong>Payment By:</strong>
                                <p t-field="o.payment_mode"/>
                            </div>
                        </div>

                        <table class="table table-sm">
                            <thead>
                                <tr>
                                    <th>Date</th>
                                    <th>Name</th>
                                    <th class="text-center">Ref.</th>
                                    <th>Unit Price</th>
                                    <th>Taxes</th>
                                    <th class="text-center">Qty</th>
                                    <th class="text-right">Price</th>
                                    <th t-if="o.is_multiple_currency" class="text-right">Price in Company Currency</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr t-foreach="o.expense_line_ids" t-as="line">
                                    <td><span t-field="line.date"/></td>
                                    <td>
                                        <span t-field="line.name"/>
                                        <span t-field="line.description"/><br/>
                                        <span t-field="line.analytic_account_id.name"/>
                                    </td>
                                    <td style="text-center">
                                        <span t-field="line.reference"/>
                                    </td>
                                    <td>
                                        <span t-field="line.unit_amount"/>
                                    </td>
                                    <td>
                                        <t t-foreach="line.tax_ids" t-as="tax">
                                          <t t-if="tax.description">
                                            <span t-field="tax.description"/>
                                          </t>
                                          <t t-if="not tax.description">
                                            <span t-field="tax.name"/>
                                          </t>
                                        </t>
                                    </td>
                                    <td class="text-center">
                                        <span t-field="line.quantity"/>
                                    </td>
                                    <td class="text-right">
                                        <span t-field="line.total_amount"
                                            t-options='{"widget": "monetary", "display_currency": line.currency_id}'/>
                                    </td>
                                    <td t-if="o.is_multiple_currency" class="text-right">
                                        <span t-field="line.total_amount_company"/>
                                    </td>
                                </tr>
                            </tbody>
                        </table>

                        <div class="row justify-content-end">
                            <div class="col-4">
                                <table class="table table-sm">
                                    <tr class="border-black">
                                        <td><strong>Total</strong></td>
                                        <td class="text-right">
                                            <span t-field="o.total_amount"
                                                t-options='{"widget": "monetary", "display_currency": o.currency_id}'/>
                                        </td>
                                    </tr>
                                </table>
                            </div>
                        </div>
                        <p>Certified honest and conform,<br/>(Date and signature).<br/><br/></p>
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
access_account_journal_user,account.journal.user,account.model_account_journal,hr_expense.group_hr_expense_team_approver,1,1,1,1
access_account_journal_employee,account.journal.employee,account.model_account_journal,base.group_user,1,0,0,0
access_account_move_user,account.move.hr.expense.approver,account.model_account_move,hr_expense.group_hr_expense_team_approver,1,0,0,0
access_account_move_line_user,account.move.line.hr.expense.approver,account.model_account_move_line,hr_expense.group_hr_expense_team_approver,1,0,0,0
access_account_analytic_line_user,account.analytic.line.user,account.model_account_analytic_line,hr_expense.group_hr_expense_team_approver,1,1,1,1
access_mail_activity_type_expense_user,mail.activity.type.expense.user,mail.model_mail_activity_type,hr_expense.group_hr_expense_manager,1,1,1,1
access_hr_expense_refuse_wizard,access.hr.expense.refuse.wizard,model_hr_expense_refuse_wizard,hr_expense.group_hr_expense_team_approver,1,1,1,0

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
            <field name="domain_force">['|', '|', '|',
                ('employee_id.user_id', '=', user.id),
                ('employee_id.department_id.manager_id.user_id', '=', user.id),
                ('employee_id.parent_id.user_id', '=', user.id),
                ('employee_id.expense_manager_id', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('hr_expense.group_hr_expense_team_approver'))]"/>
        </record>
        <record id="ir_rule_hr_expense_employee" model="ir.rule">
            <field name="name">Employee Expense</field>
            <field name="model_id" ref="model_hr_expense"/>
            <field name="domain_force">[('employee_id.user_id', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('base.group_user'))]"/>
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
            <field name="domain_force">['|', '|', '|',
                ('employee_id.user_id', '=', user.id),
                ('employee_id.department_id.manager_id.user_id', '=', user.id),
                ('employee_id.parent_id.user_id', '=', user.id),
                ('employee_id.expense_manager_id', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('hr_expense.group_hr_expense_team_approver'))]"/>
        </record>
        <record id="ir_rule_hr_expense_sheet_employee" model="ir.rule">
            <field name="name">Employee Expense Sheet</field>
            <field name="model_id" ref="model_hr_expense_sheet"/>
            <field name="domain_force">[('employee_id.user_id', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('base.group_user'))]"/>
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
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70">
    <defs>
        <path id="icon-a" d="M4,5.35309892e-14 C36.4160122,9.87060235e-15 58.0836068,-3.97961823e-14 65,5.07020818e-14 C69,6.733808e-14 70,1 70,5 C70,43.0488877 70,62.4235458 70,65 C70,69 69,70 65,70 C61,70 9,70 4,70 C1,70 7.10542736e-15,69 7.10542736e-15,65 C7.25721566e-15,62.4676575 3.83358709e-14,41.8005206 3.60818146e-14,5 C-1.13686838e-13,1 1,5.75716207e-14 4,5.35309892e-14 Z"/>
        <linearGradient id="icon-c" x1="100%" x2="0%" y1="0%" y2="98.616%">
            <stop offset="0%" stop-color="#797C79"/>
            <stop offset="100%" stop-color="#545554"/>
        </linearGradient>
        <path id="icon-d" d="M29.2777778,17.2348485 C34.4714536,17.2348485 38.681713,21.506348 38.681713,26.7755682 C38.681713,32.0447884 34.4714536,36.3162879 29.2777778,36.3162879 C24.0841019,36.3162879 19.8738426,32.0447884 19.8738426,26.7755682 C19.8738426,21.506348 24.0841019,17.2348485 29.2777778,17.2348485 Z M42.7933813,37.84729 C42.7933813,41.2291455 53.1142387,40.370791 53.1141828,46.7993457 C53.1141828,49.8785938 50.8543359,52.5088721 47.1832578,53.0575293 L47.1832578,55.6054688 C47.1832578,55.9614111 46.8831254,56.25 46.5129453,56.25 L44.2785703,56.25 C43.9083902,56.25 43.6082578,55.9614111 43.6082578,55.6054688 L43.6082578,53.0156885 C41.4017008,52.6548584 39.4354508,51.6754395 38.0920887,50.4390137 C37.8475922,50.2139648 37.8167578,49.8485693 38.0193598,49.5878564 L39.7168703,47.4035937 C39.944609,47.1106006 40.3787481,47.0596289 40.6753613,47.2886523 C42.0671535,48.3632471 43.8647641,49.2161768 45.5824957,49.2161768 C47.5829875,49.2161768 48.4941098,48.0702539 48.4941098,47.005542 C48.4941098,43.8578662 38.1732523,44.5412305 38.1732523,37.9061572 C38.1732523,35.0744092 40.3334461,32.784873 43.6083137,32.0710547 L43.6083137,29.3945312 C43.6083137,29.0385889 43.9084461,28.75 44.2786262,28.75 L46.5130012,28.75 C46.8831813,28.75 47.1833137,29.0385889 47.1833137,29.3945312 L47.1833137,31.9317822 C48.979416,32.1313721 50.9263387,32.8088281 52.2826602,33.9623779 C52.5156496,34.1605176 52.5744695,34.4877783 52.4264981,34.7508545 L51.1108981,37.0899121 C50.9184625,37.4321045 50.4576227,37.5328125 50.1291695,37.3056689 C48.8809918,36.4425342 47.3510035,35.783877 45.8581059,35.783877 C43.9963688,35.783877 42.7933813,36.5938379 42.7933813,37.84729 Z M35.2962963,36.7272727 C35.2962963,36.3141183 26.1175854,38.5736818 22.6967864,36.0773525 L18.5053937,37.1404272 C15.9936026,37.7775087 14.2314815,40.0671622 14.2314815,42.6939608 L14.2314815,44.9029356 C14.2314815,46.4837136 15.4945475,47.7651515 17.052662,47.7651515 C29.017554,47.7651515 35,47.7651515 35,47.7651515 C33.1481481,41.4242424 35.2962963,37.1404272 35.2962963,36.7272727 Z"/>
        <path id="icon-e" d="M29.2777778,15.2348485 C34.4714536,15.2348485 38.681713,19.506348 38.681713,24.7755682 C38.681713,30.0447884 34.4714536,34.3162879 29.2777778,34.3162879 C24.0841019,34.3162879 19.8738426,30.0447884 19.8738426,24.7755682 C19.8738426,19.506348 24.0841019,15.2348485 29.2777778,15.2348485 Z M42.7933813,35.84729 C42.7933813,39.2291455 53.1142387,38.370791 53.1141828,44.7993457 C53.1141828,47.8785938 50.8543359,50.5088721 47.1832578,51.0575293 L47.1832578,53.6054688 C47.1832578,53.9614111 46.8831254,54.25 46.5129453,54.25 L44.2785703,54.25 C43.9083902,54.25 43.6082578,53.9614111 43.6082578,53.6054688 L43.6082578,51.0156885 C41.4017008,50.6548584 39.4354508,49.6754395 38.0920887,48.4390137 C37.8475922,48.2139648 37.8167578,47.8485693 38.0193598,47.5878564 L39.7168703,45.4035937 C39.944609,45.1106006 40.3787481,45.0596289 40.6753613,45.2886523 C42.0671535,46.3632471 43.8647641,47.2161768 45.5824957,47.2161768 C47.5829875,47.2161768 48.4941098,46.0702539 48.4941098,45.005542 C48.4941098,41.8578662 38.1732523,42.5412305 38.1732523,35.9061572 C38.1732523,33.0744092 40.3334461,30.784873 43.6083137,30.0710547 L43.6083137,27.3945312 C43.6083137,27.0385889 43.9084461,26.75 44.2786262,26.75 L46.5130012,26.75 C46.8831813,26.75 47.1833137,27.0385889 47.1833137,27.3945312 L47.1833137,29.9317822 C48.979416,30.1313721 50.9263387,30.8088281 52.2826602,31.9623779 C52.5156496,32.1605176 52.5744695,32.4877783 52.4264981,32.7508545 L51.1108981,35.0899121 C50.9184625,35.4321045 50.4576227,35.5328125 50.1291695,35.3056689 C48.8809918,34.4425342 47.3510035,33.783877 45.8581059,33.783877 C43.9963688,33.783877 42.7933813,34.5938379 42.7933813,35.84729 Z M35.2962963,34.7272727 C35.2962963,34.3141183 26.1175854,36.5736818 22.6967864,34.0773525 L18.5053937,35.1404272 C15.9936026,35.7775087 14.2314815,38.0671622 14.2314815,40.6939608 L14.2314815,42.9029356 C14.2314815,44.4837136 15.4945475,45.7651515 17.052662,45.7651515 C29.017554,45.7651515 35,45.7651515 35,45.7651515 C33.1481481,39.4242424 35.2962963,35.1404272 35.2962963,34.7272727 Z"/>
    </defs>
    <g fill="none" fill-rule="evenodd">
        <mask id="icon-b" fill="#fff">
            <use xlink:href="#icon-a"/>
        </mask>
        <g mask="url(#icon-b)">
            <rect width="70" height="70" fill="url(#icon-c)"/>
            <path fill="#FFF" fill-opacity=".383" d="M4,1.8 L65,1.8 C67.6666667,1.8 69.3333333,1.13333333 70,-0.2 C70,2.46666667 70,3.46666667 70,2.8 L1.10547097e-14,2.8 C-1.65952376e-14,3.46666667 -2.9161925e-14,2.46666667 -2.66453526e-14,-0.2 C0.666666667,1.13333333 2,1.8 4,1.8 Z" transform="matrix(1 0 0 -1 0 2.8)"/>
            <path fill="#393939" d="M40.2790698,48 L4,48 C2,48 -7.10542736e-15,47.8556793 0,43.9590217 L2.0734159e-16,21.6246135 L20.6658658,0.0963982452 L37.783693,7.00604553 L34.121375,14.2273604 L43.799671,6.01962237 L50.8814479,14.2273604 L46.6827849,19.2273136 L52.6062041,25.9680455 L40.2790698,48 Z" opacity=".324" transform="translate(0 21)"/>
            <path fill="#000" fill-opacity=".383" d="M4,4 L65,4 C67.6666667,4 69.3333333,3 70,1 C70,3.66666667 70,5 70,5 L1.77635684e-15,5 C1.77635684e-15,5 1.77635684e-15,3.66666667 1.77635684e-15,1 C0.666666667,3 2,4 4,4 Z" transform="translate(0 65)"/>
            <use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#icon-d"/>
            <use fill="#FFF" fill-rule="nonzero" xlink:href="#icon-e"/>
        </g>
    </g>
</svg>

```

## File: static\src\js\expense_qr_code_action.js

```javascript
odoo.define('hr_expense.qr_code_action', function (require) {
"use strict";

const AbstractAction = require('web.AbstractAction');
const core = require('web.core');
const config = require('web.config');

const QRModalAction = AbstractAction.extend({
    template: 'hr_expense_qr_code',
    xmlDependencies: ['/hr_expense/static/src/xml/expense_qr_modal_template.xml'],

    init: function(parent, action){
        this._super.apply(this, arguments);
        this.url = _.str.sprintf("/report/barcode/?type=QR&value=%s&width=256&height=256&humanreadable=1", action.params.url);
    },
});

core.action_registry.add('expense_qr_code_modal', QRModalAction);
});

```

## File: static\src\js\expense_views.js

```javascript
odoo.define('hr_expense.expenses.tree', function (require) {
"use strict";
    var DocumentUploadMixin = require('hr_expense.documents.upload.mixin');
    var KanbanController = require('web.KanbanController');
    var KanbanView = require('web.KanbanView');
    var PivotView = require('web.PivotView');
    var ListController = require('web.ListController');
    var ListView = require('web.ListView');
    var viewRegistry = require('web.view_registry');
    var core = require('web.core');
    var ListRenderer = require('web.ListRenderer');
    var KanbanRenderer = require('web.KanbanRenderer');
    var PivotRenderer = require('web.PivotRenderer');
    var session = require('web.session');
    const config = require('web.config');

    var QWeb = core.qweb;

    var ExpensesListController = ListController.extend(DocumentUploadMixin, {
        buttons_template: 'ExpensesListView.buttons',
        events: _.extend({}, ListController.prototype.events, {
            'click .o_button_upload_expense': '_onUpload',
            'change .o_expense_documents_upload .o_form_binary_form': '_onAddAttachment',
        }),
    });

    const ExpenseQRCodeMixin = {
        async _renderView() {
            const self = this;
            await this._super(...arguments);
            const google_url = "https://play.google.com/store/apps/details?id=com.odoo.mobile";
            const apple_url = "https://apps.apple.com/be/app/odoo/id1272543640";
            const action_desktop = {
                name: 'Download our App',
                type: 'ir.actions.client',
                tag: 'expense_qr_code_modal',
                params: {'url': "https://apps.apple.com/be/app/odoo/id1272543640"},
                target: 'new',
            };
            this.$el.find('img.o_expense_apple_store').on('click', function(event) {
                event.preventDefault();
                if (!config.device.isMobileDevice) {
                    self.do_action(_.extend(action_desktop, {params: {'url': apple_url}}));
                } else {
                    self.do_action({type: 'ir.actions.act_url', url: apple_url});
                }
            });
            this.$el.find('img.o_expense_google_store').on('click', function(event) {
                event.preventDefault();
                if (!config.device.isMobileDevice) {
                    self.do_action(_.extend(action_desktop, {params: {'url': google_url}}));
                } else {
                    self.do_action({type: 'ir.actions.act_url', url: google_url});
                }
            });
        },
    };

    const ExpenseDashboardMixin = {
        _render: async function () {
            var self = this;
            await this._super(...arguments);
            const result = await this._rpc({
                model: 'hr.expense',
                method: 'get_expense_dashboard',
                context: this.context,
            });

            self.$el.parent().find('.o_expense_container').remove();
            const elem = QWeb.render('hr_expense.dashboard_list_header', {
                expenses: result,
                render_monetary_field: self.render_monetary_field,
            });
            self.$el.before(elem);
        },
        render_monetary_field: function (value, currency_id) {
            value = value.toFixed(2);
            var currency = session.get_currency(currency_id);
            if (currency) {
                if (currency.position === "after") {
                    value += currency.symbol;
                } else {
                    value = currency.symbol + value;
                }
            }
            return value;
        }
    };

    // Expense List Renderer
    var ExpenseListRenderer = ListRenderer.extend(ExpenseQRCodeMixin);

    // Expense List Renderer with the Header
    // Used in "My Expenses to Report", "All My Expenses" & "My Reports"
    var ExpenseListRendererHeader = ExpenseListRenderer.extend(ExpenseDashboardMixin);

    var ExpensesListViewDashboardUpload = ListView.extend({
        config: _.extend({}, ListView.prototype.config, {
            Renderer: ExpenseListRenderer,
            Controller: ExpensesListController,
        }),
    });

    // Used in "My Expenses to Report" & "All My Expenses"
    var ExpensesListViewDashboardUploadHeader = ExpensesListViewDashboardUpload.extend({
        config: _.extend({}, ExpensesListViewDashboardUpload.prototype.config, {
            Renderer: ExpenseListRendererHeader,
        }),
    });

    // The dashboard view of the expense module
    var ExpensesListViewDashboard = ListView.extend({
        config: _.extend({}, ListView.prototype.config, {
            Renderer: ExpenseListRenderer,
            Controller: ExpensesListController,
        }),
    });

    // The dashboard view of the expense module with an header
    // Used in "My Expenses"
    var ExpensesListViewDashboardHeader = ExpensesListViewDashboard.extend({
        config: _.extend({}, ExpensesListViewDashboard.prototype.config, {
            Renderer: ExpenseListRendererHeader,
        })
    });

    var ExpensesKanbanController = KanbanController.extend(DocumentUploadMixin, {
        buttons_template: 'ExpensesKanbanView.buttons',
        events: _.extend({}, KanbanController.prototype.events, {
            'click .o_button_upload_expense': '_onUpload',
            'change .o_expense_documents_upload .o_form_binary_form': '_onAddAttachment',
        }),
    });

    var ExpenseKanbanRenderer = KanbanRenderer.extend(ExpenseQRCodeMixin);

    var ExpenseKanbanRendererHeader = ExpenseKanbanRenderer.extend(ExpenseDashboardMixin);

    // The kanban view
    var ExpensesKanbanView = KanbanView.extend({
        config: _.extend({}, KanbanView.prototype.config, {
            Controller: ExpensesKanbanController,
            Renderer: ExpenseKanbanRenderer,
        }),
    });

    // The kanban view with the Header
    // Used in "My Expenses to Report", "All My Expenses" & "My Repo
    var ExpensesKanbanViewHeader = ExpensesKanbanView.extend({
        config: _.extend({}, ExpensesKanbanView.prototype.config, {
            Renderer: ExpenseKanbanRendererHeader,
        })
    });

    viewRegistry.add('hr_expense_tree_dashboard_upload', ExpensesListViewDashboardUpload);
    // Tree view with the header.
    // Used in "My Expenses to Report" & "All My Expenses"
    viewRegistry.add('hr_expense_tree_dashboard_upload_header', ExpensesListViewDashboardUploadHeader);
    viewRegistry.add('hr_expense_tree_dashboard', ExpensesListViewDashboard);
    // Tree view with the header.
    // Used in "My Reports"
    viewRegistry.add('hr_expense_tree_dashboard_header', ExpensesListViewDashboardHeader);
    viewRegistry.add('hr_expense_kanban', ExpensesKanbanView);
    // Kanban view with the header.
    // Used in "My Expenses to Report", "All My Expenses" & "My Reports"
    viewRegistry.add('hr_expense_kanban_header', ExpensesKanbanViewHeader);
});

```

## File: static\src\js\upload_mixin.js

```javascript
odoo.define('hr_expense.documents.upload.mixin', function (require) {
"use strict";

var core = require('web.core');
var config = require('web.config');
var _t = core._t;
var qweb = core.qweb;

/**
* Mixin for uploading single or multiple documents.
*/
var DocumentUploadMixin = {
    start: function () {
        // define a unique uploadId and a callback method
        this.fileUploadID = _.uniqueId('hr_expense_document_upload');
        $(window).on(this.fileUploadID, this._onFileUploaded.bind(this));
        return this._super.apply(this, arguments);
    },
    /**
     * @private
     */
    _onAddAttachment: function (ev) {
        // Auto submit form once we've selected an attachment
        var $input = $(ev.currentTarget).find('input.o_input_file');
        if ($input.val() !== '') {
            var $binaryForm = this.$('.o_expense_documents_upload form.o_form_binary_form');
            $binaryForm.submit();
        }
    },
    /**
     * @private
     */
    _onFileUploaded: function () {
        // Callback once attachment have been created, create an expense with attachment ids
        var self = this;
        var attachments = Array.prototype.slice.call(arguments, 1);
        // Get id from result
        var attachent_ids = attachments.reduce(function(filtered, record) {
            if (record.id) {
                filtered.push(record.id);
            } 
            return filtered;
        }, []);
        if (!attachent_ids.length) {
            return self.do_notify(false, _t("An error occurred during the upload"));
        }
        var myContext =  this.initialState.context
        myContext['isMobile'] =  config.device.isMobile
        return this._rpc({
            model: 'hr.expense',
            method: 'create_expense_from_attachments',
            args: ["", attachent_ids, this.viewType],
            context: myContext,
        }).then(function(result) {
            self.do_action(result);
        });
    },
    /**
     * @private
     * @param {Event} event
     */
    _onUpload: function (event) {
        var self = this;
        // If hidden upload form don't exists, create it
        var $formContainer = this.$('.o_content').find('.o_expense_documents_upload');
        if (!$formContainer.length) {
            $formContainer = $(qweb.render('hr.expense.DocumentsHiddenUploadForm', {widget: this}));
            $formContainer.appendTo(this.$('.o_content'));
        }
        // Trigger the input to select a file
        this.$('.o_expense_documents_upload .o_input_file').click();
    },
};

return DocumentUploadMixin;

});

```

## File: static\src\js\tours\hr_expense.js

```javascript
odoo.define('hr_expense.tour', function(require) {
"use strict";

var core = require('web.core');
var tour = require('web_tour.tour');

var _t = core._t;

tour.register('hr_expense_tour' , {
    url: "/web"
}, [tour.stepUtils.showAppsMenuItem(), {
    trigger: '.o_app[data-menu-xmlid="hr_expense.menu_hr_expense_root"]',
    content: _t("Want to manage your expenses? It starts here."),
    position: 'right',
    edition: 'community'
}, {
    trigger: '.o_app[data-menu-xmlid="hr_expense.menu_hr_expense_root"]',
    content: _t("Want to manage your expenses? It starts here."),
    position: 'bottom',
    edition: 'enterprise'
}, {
    trigger: '.o_form_button_save',
    content: _t("<p>Once your <b> Expense </b> is ready, you can save it.</p>"),
    position: 'bottom',
}, {
    trigger: '.o_attach_document',
    content: _t("Attach your receipt here."),
    position: 'bottom',
}, {
    trigger: '.o_expense_submit',
    extra_triggger: ".o_expense_form",
    content: _t('<p>Click on <b> Create Report </b> to create the report.</p>'),
    position: 'right',
}, {
    trigger: '.o_expense_tree input[type=checkbox]',
    content: _t('<p>Select expenses to submit them to your manager</p>'),
    position: 'bottom'
}, {
    trigger: '.o_dropdown_toggler_btn',
    extra_trigger: ".o_expense_tree",
    content: _t('<p>Click on <b> Action Create Report </b> to submit selected expenses to your manager</p>'),
    position: 'right',
},  {
    trigger: '.o_expense_sheet_submit',
    content: _t('Once your <b>Expense report</b> is ready, you can submit it to your manager and wait for the approval from your manager.'),
    position: 'bottom',
}, {
    trigger: '.o_expense_sheet_approve',
    content: _t("<p>Approve the report here.</p><p>Tip: if you refuse, don’t forget to give the reason thanks to the hereunder message tool</p>"),
    position: 'bottom',
}, {
    trigger: '.o_expense_sheet_post',
    content: _t("<p>The accountant receive approved expense reports.</p><p>He can post journal entries in one click if taxes and accounts are right.</p>"),
    position: 'bottom',
}, {
    trigger: '.o_expense_sheet_pay',
    content: _t("The accountant can register a payment to reimburse the employee directly."),
    position: 'bottom',
}, {
    trigger: 'li a[data-menu-xmlid="hr_expense.menu_hr_expense_sheet_my_all"], div[data-menu-xmlid="hr_expense.menu_hr_expense_sheet_my_all"]',
    content: _t("Managers can get all reports to approve from this menu."),
    position: 'bottom',
}]);

});

```

## File: static\src\xml\documents_upload_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>
    <t t-name="hr.expense.DocumentsHiddenUploadForm">
        <div class="d-none o_expense_documents_upload">
            <t t-call="HiddenInputFile">
                <t t-set="multi_upload" t-value="true"/>
                <t t-set="fileupload_id" t-value="widget.fileUploadID"/>
                <t t-set="fileupload_action" t-translation="off">/web/binary/upload_attachment</t>
                <input type="hidden" name="model" t-att-value="'hr.expense'"/>
                <input type="hidden" name="id" t-att-value="0"/>
            </t>
        </div>
    </t>

    <t t-extend="ListView.buttons" t-name="ExpensesListView.buttons">
        <t t-jquery="button.o_list_button_add" t-operation="after">
            <button type="button" class="btn btn-primary o_button_upload_expense">
                Upload
            </button>
        </t>
    </t>

    <t t-extend="KanbanView.buttons" t-name="ExpensesKanbanView.buttons">
        <t t-jquery="button" t-operation="after">
            <button type="button" class="btn btn-primary o_button_upload_expense">
                Upload
            </button>
        </t>
    </t>
</templates>

```

## File: static\src\xml\expense_dashboard.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates id="template">
    <t t-name="hr_expense.dashboard_list_header">
        <div class="o_expense_container d-flex o_form_statusbar">
            <t t-foreach="expenses" t-as="expense">
                <div t-attf-class="o_expense_card o_arrow_button flex-grow-1 d-flex flex-column border-right-0">
                    <div class="content_center">
                        <div>
                            <span t-esc="render_monetary_field(expenses[expense]['amount'], expenses[expense]['currency'])" class="h2 o_expense_purple"/>
                        </div>
                        <b class="m-2"><span t-esc="expenses[expense]['description']"/></b>
                    </div>
                </div>
                <div t-if="expense !== 'approved'" t-attf-class="o_expense_card o_arrow_button flex-grow-1 d-flex flex-column border-right-0">
                    <div class="content_center">
                        <span class="fa fa-angle-right fa-3x"/>
                    </div>
                </div>
            </t>
        </div>
    </t>
</templates>

```

## File: static\src\xml\expense_qr_modal_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
<t t-name="hr_expense_qr_code">
    <div style="text-align:center;" class="o_expense_modal">
        <t t-if="widget.url">
            <h3>Scan this QR code to get the Odoo app:</h3><br/><br/>
            <img class="border border-dark rounded" t-att-src="widget.url"/>
        </t>
    </div>
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
                    <div class="col overflow-hidden text-left">
                        <a type="object" t-if="journal_type == 'purchase'" name="open_expenses_action">
                            <t t-esc="dashboard.number_expenses_to_pay"/> Expenses to Process
                        </a>
                    </div>
                    <div class="col-auto text-right">
                        <span t-if="journal_type == 'purchase'"><t t-esc="dashboard.sum_expenses_to_pay"/></span>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="assets_backend" name="HR Expense Assets assets" inherit_id="web.assets_backend">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/hr_expense/static/src/js/expense_views.js"/>
                <script type="text/javascript" src="/hr_expense/static/src/js/expense_qr_code_action.js"/>
                <script type="text/javascript" src="/hr_expense/static/src/js/upload_mixin.js"/>
            </xpath>
            <xpath expr="link[last()]" position="after">
                <link rel="stylesheet" type="text/scss" href="/hr_expense/static/src/scss/hr_expense.scss"/>
            </xpath>
        </template>
        <template id="assets_tests" name="HR Expense Assets Tests" inherit_id="web.assets_tests">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/hr_expense/static/src/js/tours/hr_expense.js"></script>
                <script type="text/javascript" src="/hr_expense/static/tests/tours/expense_upload_tours.js"></script>
            </xpath>
        </template>
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
        <field name="groups_id" eval="[(4,ref('hr_expense.group_hr_expense_team_approver'))]"/>
        <field name="arch" type="xml">
            <data>
                <xpath expr="//templates" position="before">
                    <field name="expense_sheets_to_approve_count"/>
                </xpath>

                <xpath expr="//div[hasclass('o_kanban_primary_right')]" position="inside">
                    <div t-if="record.expense_sheets_to_approve_count.raw_value > 0" class="row ml16">
                        <div class="col-9">
                            <a name="%(action_hr_expense_sheet_department_to_approve)d" type="action">
                                Expense Reports
                            </a>
                        </div>
                        <div class="col-3 text-right">
                            <t t-esc="record.expense_sheets_to_approve_count.raw_value"/>
                        </div>
                    </div>
                </xpath>

                <xpath expr="//div[hasclass('o_kanban_manage_reports')]" position="inside">
                    <a role="menuitem" class="dropdown-item" name="%(action_hr_expense_sheet_department_filtered)d" type="action">
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
                    <field name="expense_manager_id" context="{'default_company_id': company_id}"/>
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
                <xpath expr="//field[@name='work_location']" position="after">
                    <field name="expense_manager_id" optional="hide" string="Expense Approver"/>
                </xpath>
            </field>
        </record>

        <record id="res_users_view_form_preferences" model="ir.ui.view">
            <field name="name">hr.user.preferences.form.inherit.hr.expense</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="hr.res_users_view_form_profile" />
            <field name="arch" type="xml">
                <xpath expr="//group[@name='managers']" position="inside">
                    <field name="expense_manager_id" attrs="{'readonly': [('can_edit', '=', False)]}" context="{'default_company_id': company_id}"/>
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
                <tree string="Expenses" multi_edit="1" sample="1" js_class="hr_expense_tree_dashboard_upload">
                    <header>
                        <button name="action_submit_expenses" type="object" string="Create Report"/>
                    </header>
                    <field name="attachment_number" invisible="True"/>
                    <field name="date" optional="show"/>
                    <field name="accounting_date" optional="hide" groups="account.group_account_invoice,account.group_account_readonly"/>
                    <field name="name"/>
                    <field name="employee_id" widget="many2one_avatar_employee"/>
                    <field name="sheet_id" optional="show" invisible="not context.get('show_report', False)" readonly="1"/>
                    <field name="payment_mode" optional="show"/>
                    <field name="reference" optional="hide"/>
                    <field name="analytic_account_id" optional="show" groups="analytic.group_analytic_accounting"/>
                    <field name="analytic_tag_ids" optional="hide" widget="many2many_tags" groups="analytic.group_analytic_tags"/>
                    <field name="account_id" optional="hide"/>
                    <field name="company_id" optional="show" groups="base.group_multi_company" readonly="1"/>
                    <field name="unit_amount" optional="hide" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                    <field name="currency_id" optional="hide"/>
                    <field name="quantity" optional="hide"/>
                    <field name="activity_ids" widget="list_activity" optional="show"/>
                    <field name="tax_ids" optional="show" widget="many2many_tags" groups="account.group_account_user"/>
                    <field name="total_amount" optional="show" sum="Total Amount" widget="monetary" options="{'currency_field': 'currency_id'}" decoration-bf="True"/>
                    <button name="action_get_attachment_view" string="Attachments" type="object" icon="fa-paperclip" attrs="{'invisible': [('attachment_number', '=', 0)]}"/>
                    <field name="message_unread" invisible="1"/>
                    <field name="state" optional="show" readonly="1" decoration-info="state == 'draft'" decoration-success="state in ['reported', 'approved', 'done']" decoration-danger="state in 'refused'" widget="badge"/>
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
                    <attribute name="js_class">hr_expense_tree_dashboard_upload_header</attribute>
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
                    <attribute name="js_class">hr_expense_tree_dashboard_header</attribute>
                    <attribute name="class">hr_expense</attribute>
                </xpath>
            </field>
        </record>

        <record id="hr_expense_view_form" model="ir.ui.view">
            <field name="name">hr.expense.view.form</field>
            <field name="model">hr.expense</field>
            <field name="arch" type="xml">
                <form string="Expenses" class="o_expense_form">
                <header>
                  <button name="action_submit_expenses" string="Create Report" type="object" class="oe_highlight o_expense_submit" attrs="{'invisible': ['|', ('attachment_number', '&lt;=', 0), ('sheet_id', '!=', False)]}"/>
                  <widget name="attach_document" string="Attach Receipt" action="message_post" attrs="{'invisible': ['|', ('attachment_number', '&lt;', 1), ('id','=',False)]}"/>
                  <widget name="attach_document" string="Attach Receipt" action="message_post" highlight="1" attrs="{'invisible': ['|',('attachment_number', '&gt;=', 1), ('id','=',False)]}"/>
                  <button name="action_submit_expenses" string="Create Report" type="object" class="o_expense_submit" attrs="{'invisible': ['|', ('attachment_number', '&gt;=', 1), ('sheet_id', '!=', False)]}"/>
                  <field name="state" widget="statusbar" statusbar_visible="draft,reported,approved,done,refused"/>
                  <button name="action_view_sheet" type="object" string="View Report" class="oe_highlight" attrs="{'invisible': [('sheet_id', '=', False)]}"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_get_attachment_view"
                            class="oe_stat_button"
                            icon="fa-file-text-o"
                            type="object">
                            <field name="attachment_number" widget="statinfo" string="Receipts" options="{'reload_on_button': true}"/>
                        </button>
                    </div>
                    <div class="oe_title">
                        <label for="name" class="oe_edit_only"/>
                        <h1>
                            <field name="name" placeholder="e.g. Lunch with Customer"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="is_editable" invisible="1"/>
                            <field name="is_ref_editable" invisible="1"/>
                            <field name="product_id" required="1" context="{'default_can_be_expensed': 1, 'tree_view_ref': 'hr_expense.product_product_expense_tree_view'}"
                                   widget="many2one_barcode"
                            />
                            <field name="unit_amount" required="1" widget="monetary" options="{'currency_field': 'currency_id', 'field_digits': True}"/>
                            <field name="product_uom_category_id" invisible="1"/>
                            <label for="quantity"/>
                            <div class="o_row">
                                <field name="quantity" class="oe_inline"/>
                                <field name="product_uom_id" required="1" widget="selection" class="oe_inline" groups="uom.group_uom"/>
                            </div>
                            <field name="tax_ids" widget="many2many_tags" groups="account.group_account_readonly" attrs="{'readonly': [('is_editable', '=', False)]}" context="{'default_company_id': company_id}"/>
                            <field name="total_amount" widget='monetary' options="{'currency_field': 'currency_id'}"/>
                            <field name="amount_residual" widget='monetary' options="{'currency_field': 'currency_id'}"/>
                        </group><group>
                            <field name="reference" attrs="{'readonly': [('is_ref_editable', '=', False)]}"/>
                            <field name="date"/>
                            <field name="accounting_date" attrs="{'invisible': ['|', ('accounting_date', '=', False), ('state', 'not in', ['approved', 'done'])]}" />
                            <field name="account_id" options="{'no_create': True}" domain="[('internal_type', '=', 'other'), ('company_id', '=', company_id)]" groups="account.group_account_readonly" attrs="{'readonly': [('is_editable', '=', False)]}" context="{'default_company_id': company_id}"/>
                            <field name="employee_id" groups="hr_expense.group_hr_expense_team_approver" context="{'default_company_id': company_id}"/>
                            <field name="sheet_id" invisible="1"/>
                            <field name="currency_id" groups="base.group_multi_currency"/>
                            <field name="analytic_account_id" domain="['|', ('company_id', '=', company_id), ('company_id', '=', False)]" groups="analytic.group_analytic_accounting" attrs="{'readonly': [('is_editable', '=', False)]}"/>
                            <field name="analytic_tag_ids" widget="many2many_tags" groups="analytic.group_analytic_tags" attrs="{'readonly': [('is_editable', '=', False)]}"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group><group>
                            <label for="payment_mode"/>
                            <div>
                                <field name="payment_mode" widget="radio"/>
                            </div>
                        </group>
                    </group>
                    <div>
                        <field name="description" class="oe_inline" placeholder="Notes..."/>
                    </div>
                </sheet>
                <div class="o_attachment_preview"/>
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
                <xpath expr="/form/sheet/div[hasclass('oe_button_box')]" position="attributes">
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
                    <field name="total_amount"/>
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
                                        <strong class="o_kanban_record_title"><span><t t-esc="record.name.value"/></span></strong>
                                        <strong class="o_kanban_record_subtitle float-right"><span class="text-right"><field name="total_amount" widget="monetary"/></span></strong>
                                    </div>
                                </div>
                                <div class="row mt8">
                                    <div class="col-6 text-muted">
                                        <span><t t-esc="record.employee_id.value"/> <t t-esc="record.date.value"/></span>
                                    </div>
                                    <div class="col-6">
                                        <span class="float-right text-right">
                                            <field name="state" widget="label_selection" options="{'classes': {'draft': 'default', 'reported': 'primary', 'refused': 'danger', 'done': 'warning',
                                            'approved': 'success'}}"/>
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

        <!-- Kanban view with header. Used in "All My Expenses -->
        <record id="hr_expense_kanban_view_header" model="ir.ui.view">
            <field name="name">hr.expense.kanban</field>
            <field name="model">hr.expense</field>
            <field name="inherit_id" ref="hr_expense_view_expenses_analysis_kanban"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//kanban" position="attributes">
                    <attribute name="js_class">hr_expense_kanban_header</attribute>
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
                    <field name="total_amount" type="measure"/>
                </pivot>
            </field>
        </record>

        <record id="hr_expense_view_graph" model="ir.ui.view">
            <field name="name">hr.expense.graph</field>
            <field name="model">hr.expense</field>
            <field name="arch" type="xml">
                <graph string="Expenses Analysis" sample="1">
                    <field name="date" type="col"/>
                    <field name="employee_id" type="row"/>
                    <field name="total_amount" type="measure"/>
                </graph>
            </field>
        </record>

        <record id="hr_expense_view_search" model="ir.ui.view">
            <field name="name">hr.expense.view.search</field>
            <field name="model">hr.expense</field>
            <field name="arch" type="xml">
                <search string="Expense">
                    <field string="Expense" name="name" filter_domain="['|', '|', ('employee_id', 'ilike', self), ('name', 'ilike', self), ('product_id', 'ilike', self)]"/>
                    <field name="date"/>
                    <field name="employee_id"/>
                    <field name="analytic_account_id" groups="analytic.group_analytic_accounting"/>
                    <filter string="My Expenses" name="my_expenses" domain="[('employee_id.user_id', '=', uid)]"/>
                    <filter string="My Team" name="my_team_expenses" domain="[('employee_id.parent_id.user_id', '=', uid)]" groups="hr_expense.group_hr_expense_team_approver" help="Expenses of Your Team Member"/>
                    <separator />
                    <filter string="To Report" name="no_report" domain="[('sheet_id', '=', False)]"/>
                    <filter string="Refused" name="refused" domain="[('state', '=', 'refused')]" help="Refused Expenses"/>
                    <separator />
                    <filter string="Expense Date" name="date" date="date"/>
                    <separator />
                    <filter string="Former Employees" name="inactive" domain="[('employee_id.active', '=', False)]" groups="hr_expense.group_hr_expense_user,hr_expense.group_hr_expense_manager"/>
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
                        <filter string="Product" name="product" domain="[]" context="{'group_by': 'product_id'}"/>
                        <filter string="Analytic Account" name="analyticacc" domain="[]" context="{'group_by': 'analytic_account_id'}" groups="analytic.group_analytic_accounting"/>
                        <filter string="Status" name="status" domain="[]" context="{'group_by': 'state'}"/>
                        <filter string="Expense Date" name="expensesmonth" domain="[]" context="{'group_by': 'date'}" help="Expense Date"/>
                        <filter string="Company" name="company" domain="[]" context="{'group_by': 'company_id'}" groups="base.group_multi_company"/>
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
                            <img t-att-src="activity_image('hr.employee', 'image_128', record.employee_id.raw_value)" t-att-title="record.employee_id.value" t-att-alt="record.employee_id.value"/>
                            <div>
                                <field name="name" display="full"/>
                                <field name="total_amount" widget="monetary" muted="1" display="full"/>
                            </div>
                        </div>
                    </templates>
                </activity>
            </field>
        </record>

        <record id="hr_expense_actions_all" model="ir.actions.act_window">
            <field name="name">Expenses Analysis</field>
            <field name="res_model">hr.expense</field>
            <field name="view_mode">graph,pivot,tree,kanban,form</field>
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

        <record id="hr_expense_actions_all_kanban" model="ir.actions.act_window.view">
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="hr_expense_view_expenses_analysis_kanban"/>
            <field name="act_window_id" ref="hr_expense_actions_all"/>
        </record>

        <record id="hr_expense_actions_my_unsubmitted" model="ir.actions.act_window">
            <field name="name">My Expenses to Report</field>
            <field name="res_model">hr.expense</field>
            <field name="view_mode">tree,kanban,form,graph,pivot,activity</field>
            <field name="search_view_id" ref="hr_expense_view_search"/>
            <field name="context">{'search_default_my_expenses': 1, 'search_default_no_report': 1}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_expense_receipt">
                    Did you try the mobile app?
                </p>
                <p>Snap pictures of your receipts and let Odoo<br/> automatically create expenses for you.</p>
                <p>
                    <a href="https://apps.apple.com/be/app/odoo/id1272543640" target="_blank">
                        <img alt="Apple App Store" class="img img-fluid h-100 o_expense_apple_store" src="/hr_expense/static/img/app_store.png"/>
                    </a>
                    <a href="https://play.google.com/store/apps/details?id=com.odoo.mobile" target="_blank" class="o_expense_google_store">
                        <img alt="Google Play Store" class="img img-fluid h-100 o_expense_google_store" src="/hr_expense/static/img/play_store.png"/>
                    </a>
                </p>
            </field>
        </record>

        <!-- Tree & Kanban view for "My Expenses to Report" with the header -->
        <record id="hr_expense_actions_my_unsubmitted_tree" model="ir.actions.act_window.view">
            <field name="view_mode">tree</field>
            <field name="view_id" ref="view_my_expenses_tree"/>
            <field name="act_window_id" ref="hr_expense_actions_my_unsubmitted"/>
        </record>

        <record id="hr_expense_actions_my_unsubmitted_kanban" model="ir.actions.act_window.view">
            <field name="view_mode">kanban</field>
            <field name="view_id" ref="hr_expense_kanban_view_header"/>
            <field name="act_window_id" ref="hr_expense_actions_my_unsubmitted"/>
        </record>

        <record id="hr_expense_actions_my_all" model="ir.actions.act_window">
            <field name="name">All My Expenses</field>
            <field name="res_model">hr.expense</field>
            <field name="view_mode">tree,kanban,form,graph,pivot,activity</field>
            <field name="search_view_id" ref="hr_expense_view_search"/>
            <field name="context">{'search_default_my_expenses': 1}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_expense_receipt">
                    Did you try the mobile app?
                </p>
                <p>Snap pictures of your receipts and let Odoo<br/> automatically create expenses for you.</p>
                <p>
                    <a href="https://apps.apple.com/be/app/odoo/id1272543640" target="_blank">
                        <img alt="Apple App Store" class="img img-fluid h-100 o_expense_apple_store" src="/hr_expense/static/img/app_store.png"/>
                    </a>
                    <a href="https://play.google.com/store/apps/details?id=com.odoo.mobile" target="_blank" class="o_expense_google_store">
                        <img alt="Google Play Store" class="img img-fluid h-100 o_expense_google_store" src="/hr_expense/static/img/play_store.png"/>
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
                    <div>
                        <field name="can_be_expensed"/>
                        <label for="can_be_expensed"/>
                    </div>
                </div>
                <xpath expr="//page[@name='general_information']//field[@name='taxes_id']" position="after">
                    <field name="supplier_taxes_id" widget="many2many_tags" attrs="{'invisible':['|',('can_be_expensed','=',False),('purchase_ok','=',True)]}" context="{'default_type_tax_use':'purchase', 'search_default_purchase': 1, 'search_default_service': type == 'service', 'search_default_goods': type == 'consu'}"/>
                </xpath>
            </field>
        </record>

        <record id="product_product_expense_form_view" model="ir.ui.view">
            <field name="name">product.product.expense.form</field>
            <field name="model">product.product</field>
            <field name="arch" type="xml">
                <form string="Expense Products">
                    <sheet>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <field name='product_variant_count' invisible='1'/>
                        <field name="id" invisible="True"/>
                        <field name="image_1920" widget="image" class="oe_avatar" options="{'image_preview': 'image_128'}"/>
                        <div class="oe_title">
                            <label class="oe_edit_only" for="name" string="Product Name"/>
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
                                <field name="list_price"/>
                                <field name="standard_price"/>
                                <field name="uom_id" groups="uom.group_uom" options="{'no_create': True}"/>
                                <field name="uom_po_id" invisible="1"/>
                                <label for="default_code"/>
                                <div>
                                    <field name="default_code"/>
                                    <i class="text-muted oe_edit_only">Use this reference as a subject prefix when submitting by email.</i>
                                </div>
                                <field name="company_id" groups="base.group_multi_company"/>
                            </group>
                            <group string="Accounting">
                                <field name="property_account_expense_id" groups="account.group_account_readonly"/>
                                <field name="supplier_taxes_id" widget="many2many_tags" context="{'default_type_tax_use':'purchase'}"/>
                                <field name="taxes_id" widget="many2many_tags" context="{'default_type_tax_use':'sale'}"/>
                            </group>
                        </group>
                    </sheet>
                </form>
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
                    <field name="product_template_attribute_value_ids" widget="many2many_tags" groups="product.group_product_variant"/>
                    <field name="standard_price"/>
                    <field name="uom_id" options="{'no_open': True, 'no_create': True}" groups="uom.group_uom"/>
                    <field name="barcode"/>
                </tree>
            </field>
        </record>

        <record id="hr_expense_product" model="ir.actions.act_window">
            <field name="name">Expense Products</field>
            <field name="res_model">product.product</field>
            <field name="view_mode">kanban,tree,form</field>
            <field name="search_view_id" ref="product.product_search_form_view"/>
            <field name="context">{"default_can_be_expensed": 1, 'default_type': 'service'}</field>
            <field name="domain">[('can_be_expensed', '=', True)]</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No expense products found. Let's create one!
              </p><p>
                Expense products can be reinvoiced to your customers.
              </p>
            </field>
        </record>

        <record id="hr_expense_product_tree" model="ir.actions.act_window.view">
            <field name="sequence" eval="2"/>
            <field name="view_mode">tree</field>
            <field name="act_window_id" ref="hr_expense_product"/>
        </record>

        <record id="hr_expense_product_kanban" model="ir.actions.act_window.view">
            <field name="sequence" eval="1"/>
            <field name="view_mode">kanban</field>
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
                <tree string="Expense Reports" js_class="hr_expense_tree_dashboard_upload" sample="1">
                    <field name="employee_id" widget="many2one_avatar_employee"/>
                    <field name="accounting_date" optional="show" groups="account.group_account_manager"/>
                    <field name="create_date" optional="hide"/>
                    <field name="name" string="Expense Report"/>
                    <field name="user_id" optional="hide" widget="many2one_avatar_user"/>
                    <field name="company_id" optional="show" groups="base.group_multi_company"/>
                    <field name="activity_ids" widget="list_activity" optional="show"/>
                    <field name="total_amount" optional="show" sum="Total Amount" decoration-bf="True"/>
                    <field name="currency_id" optional="hide"/>
                    <field name="journal_id" optional="hide"/>
                    <field name="state" optional="show" decoration-info="state == 'draft'" decoration-success="state in ['submit', 'approve', 'post', 'done']" decoration-danger="state == 'cancel'" widget="badge"/>
                    <field name="message_unread" invisible="1"/>
                </tree>
            </field>
        </record>

        <record id="view_hr_expense_sheet_dashboard_tree" model="ir.ui.view">
            <field name="name">hr.expense.sheet.dashboard.tree</field>
            <field name="model">hr.expense.sheet</field>
            <field name="priority">20</field>
            <field name="inherit_id" ref="hr_expense.view_hr_expense_sheet_tree"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//tree" position="attributes">
                    <attribute name="js_class">hr_expense_tree_dashboard</attribute>
                </xpath>
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
                    <attribute name="js_class">hr_expense_tree_dashboard_header</attribute>
                </xpath>
            </field>
        </record>

        <record id="view_hr_expense_sheet_form" model="ir.ui.view">
            <field name="name">hr.expense.sheet.form</field>
            <field name="model">hr.expense.sheet</field>
            <field eval="25" name="priority"/>
            <field name="arch" type="xml">
                <form string="Expense Reports" class="o_expense_sheet">
                <field name="can_reset" invisible="1"/>
                 <header>
                    <button name="action_submit_sheet" states="draft" string="Submit to Manager" type="object" class="oe_highlight o_expense_sheet_submit"/>
                    <button name="approve_expense_sheets" states="submit" string="Approve" type="object" groups="hr_expense.group_hr_expense_team_approver" class="oe_highlight o_expense_sheet_approve"/>
                    <button name="action_sheet_move_create"
                            string="Post Journal Entries"
                            type="object"
                            class="oe_highlight o_expense_sheet_post"
                            attrs="{'invisible': [('state', '!=', 'approve')]}"
                            groups="account.group_account_invoice"/>
                    <button name="action_register_payment"
                            type="object"
                            class="oe_highlight o_expense_sheet_pay"
                            attrs="{'invisible': [('state', '!=', 'post')]}"
                            context="{'dont_redirect_to_payments': True}"
                            string="Register Payment"
                            groups="account.group_account_invoice"/>
                    <button name="%(hr_expense.hr_expense_refuse_wizard_action)d" states="submit,approve" context="{'hr_expense_refuse_model':'hr.expense.sheet'}" string="Refuse" type="action" groups="hr_expense.group_hr_expense_team_approver" />
                    <button name="reset_expense_sheets" string="Reset to Draft" type="object" attrs="{'invisible': ['|', ('can_reset', '=', False), ('state', 'not in', ['submit', 'cancel'])]}"/>
                    <field name="state" widget="statusbar" statusbar_visible="draft,submit,approve,post,done"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_get_attachment_view"
                            class="oe_stat_button"
                            icon="fa-file-text-o"
                            type="object">
                            <field name="attachment_number" widget="statinfo" string="Documents"/>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Paid" bg_color="bg-success" attrs="{'invisible': [('state', '!=', 'done')]}"/>
                    <div class="oe_title">
                        <label for="name" class="oe_edit_only"/>
                        <h1>
                            <field name="name" placeholder="e.g. Trip to NY"/>
                        </h1>
                    </div>
                    <group>
                        <group name="employee_details">
                            <field name="employee_id" context="{'default_company_id': company_id}" widget="many2one_avatar_employee"/>
                            <field name="user_id" domain="[('share', '=', False)]" widget="many2one_avatar_user"/>
                            <field name="payment_mode"/>
                            <field name="address_id" invisible="1" context="{'default_company_id': company_id}"/>
                            <field name="department_id" invisible="1" context="{'default_company_id': company_id}"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                    </group>
                     <notebook>
                        <page name="expenses" string="Expense">
                        <field name="expense_line_ids" nolabel="1" widget="many2many" domain="[('state', '=', 'draft'), ('employee_id', '=', employee_id), ('company_id', '=', company_id)]" options="{'reload_on_button': True}" context="{'form_view_ref' : 'hr_expense.hr_expense_view_form_without_header', 'default_company_id': company_id, 'default_employee_id': employee_id}">
                            <tree decoration-danger="is_refused" editable="bottom">
                                <field name="date" optional="show"/>
                                <field name="name"/>
                                <field name="employee_id" invisible="1"/>
                                <field name="state" invisible="1"/>
                                <field name="reference" optional="hide"/>
                                <field name="analytic_account_id" optional="show" domain="['|', ('company_id', '=', parent.company_id), ('company_id', '=', False)]" groups="analytic.group_analytic_accounting"/>
                                <field name="analytic_tag_ids" optional="hide" widget="many2many_tags" groups="analytic.group_analytic_tags"/>
                                <field name="account_id" optional="hide"/>
                                <field name="message_unread" invisible="1"/>
                                <field name="attachment_number" string=" "/>
                                <button name="action_get_attachment_view"  string="View Attachments" type="object" icon="fa-paperclip"/>
                                <field name="unit_amount" optional="hide" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                                <field name="currency_id" optional="hide"/>
                                <field name="quantity" optional="hide"/>
                                <field name="company_id" invisible="1"/>
                                <field name="tax_ids" optional="show" widget="many2many_tags" groups="account.group_account_readonly" context="{'default_company_id': company_id}"/>
                                <field name="total_amount" optional="show"/>
                                <field name="company_currency_id" invisible="1"/>
                                <field name="total_amount_company" optional="show" groups="base.group_multi_currency"/>
                                <field name="is_refused" invisible="True"/>
                           </tree>
                        </field>
                        <field name="currency_id" invisible="1"/>
                        <group class="oe_subtotal_footer oe_right" colspan="2" name="expense_total">
                                <div class="oe_subtotal_footer_separator oe_inline o_td_label">
                                    <label for="total_amount"/>
                                </div>
                                <field name="total_amount" nolabel="1" class="oe_subtotal_footer_separator"/>
                                <field name="amount_residual"
                                       class="oe_subtotal_footer_separator"
                                       attrs="{'invisible': [('state', 'not in', ('post', 'done'))]}"/>
                            </group>
                        </page>
                        <page name="other_info" string="Other Info">
                            <group>
                                <group>
                                    <field name="journal_id" options="{'no_open': True, 'no_create': True}" attrs="{'invisible': [('payment_mode', '!=', 'own_account')]}" context="{'default_company_id': company_id}"/>
                                    <field name="bank_journal_id" groups="account.group_account_invoice,account.group_account_readonly" options="{'no_open': True, 'no_create': True}" attrs="{'invisible': [('payment_mode', '!=', 'company_account')]}" context="{'default_company_id': company_id}"/>
                                    <field name="accounting_date" groups="account.group_account_invoice,account.group_account_readonly" attrs="{'invisible': [('state', 'not in', ['approve', 'post', 'done'])]}"/>
                                </group>
                                <group>
                                    <field name="account_move_id" groups="account.group_account_invoice,account.group_account_readonly" attrs="{'invisible': [('state', 'not in', ['post', 'done'])]}" readonly="1"/>
                                </group>
                            </group>
                        </page>
                     </notebook>
                </sheet>
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
                                        <strong class="o_kanban_record_title"><span><t t-esc="record.name.value"/></span></strong>
                                        <strong class="o_kanban_record_subtitle float-right"><span class="text-right"><field name="total_amount" widget="monetary"/></span></strong>
                                    </div>
                                </div>
                                <div class="row mt8">
                                    <div class="col-6 text-muted">
                                        <span><t t-esc="record.employee_id.value"/> <t t-esc="record.accounting_date.value"/></span>
                                    </div>
                                    <div class="col-6">
                                        <span class="float-right text-right">
                                            <field name="state" widget="label_selection" options="{'classes': {'draft': 'default', 'submit': 'default', 'cancel': 'danger', 'post': 'warning',
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
                    <attribute name="js_class">hr_expense_kanban_header</attribute>
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
                    <field name="employee_id" type="col"/>
                    <field name="accounting_date" interval="month" type="row"/>
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
                    <filter string="My Team" name="my_team_reports" domain="[('employee_id.parent_id.user_id', '=', uid)]" groups="hr_expense.group_hr_expense_manager" help="Expenses of Your Team Member"/>
                    <separator />
                    <filter domain="[('state', '=', 'submit')]" string="To Approve" name="submitted" help="Confirmed Expenses"/>
                    <filter domain="[('state', '=', 'approve')]" string="To Post" name="to_post" help="Approved Expenses"/>
                    <filter domain="[('state', '=', 'post')]" string="To Pay" name="approved" help="Expenses to Invoice"/>
                    <filter domain="[('state', '=', 'cancel')]" string="Refused" name="canceled"/>
                    <separator/>
                    <filter string="Date" name="filter_accounting_date" date="accounting_date"/>
                    <separator/>
                    <filter domain="[('employee_id.active', '=', False)]" string="Former Employees" name="inactive" groups="hr_expense.group_hr_expense_user,hr_expense.group_hr_expense_manager"/>
                    <filter invisible="1" string="Late Activities" name="activities_overdue"
                        domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                        help="Show all records which has next action date is before today"/>
                    <filter invisible="1" string="Today Activities" name="activities_today"
                        domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                    <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                            domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))
                            ]"/>
                    <group expand="0" string="Group By">
                        <filter string="Employee" name="employee" domain="[]" context="{'group_by': 'employee_id'}"/>
                        <filter string="Department" name="department" domain="[]" context="{'group_by': 'department_id'}"/>
                        <filter string="Company" name="company" domain="[]" context="{'group_by': 'company_id'}" groups="base.group_multi_company"/>
                        <filter string="Date" name="expenses_month" domain="[]" context="{'group_by': 'accounting_date'}" help="Expenses by Date"/>
                        <filter string="Status" domain="[]" context="{'group_by': 'state'}" name="state"/>
                    </group>
                </search>
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
                            <img t-att-src="activity_image('hr.employee', 'image_128', record.employee_id.raw_value)" t-att-title="record.employee_id.value" t-att-alt="record.employee_id.value"/>
                            <div>
                                <field name="name" display="full"/>
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
            <field name="domain">[('state', '!=', 'cancel')]</field>
            <field name="context">{'search_default_my_reports': 1}</field>
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

        <record id="action_hr_expense_sheet_all_to_approve" model="ir.actions.act_window">
            <field name="name">Expense Reports to Approve</field>
            <field name="res_model">hr.expense.sheet</field>
            <field name="view_mode">tree,kanban,form,pivot,graph,activity</field>
            <field name="search_view_id" ref="hr_expense_sheet_view_search"/>
            <field name="domain">[]</field>
            <field name="context">{'search_default_submitted': 1}</field>
            <field name="view_id" ref="view_hr_expense_sheet_tree"/>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No expense reports found
              </p><p>
                Approve the new expense reports submitted by the employees you manage.
              </p>
            </field>
        </record>

        <record id="action_hr_expense_sheet_all_to_post" model="ir.actions.act_window">
            <field name="name">Expense Reports To Post</field>
            <field name="res_model">hr.expense.sheet</field>
            <field name="view_mode">tree,kanban,form,pivot,graph</field>
            <field name="search_view_id" ref="hr_expense_sheet_view_search"/>
            <field name="domain">[]</field>
            <field name="context">{'search_default_to_post': 1}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No expense reports found
              </p><p>
                Post the journal entries of the new expense reports approved by the employees' manager.
              </p>
            </field>
        </record>

        <record id="action_hr_expense_sheet_all_to_pay" model="ir.actions.act_window">
            <field name="name">Expense Reports To Pay</field>
            <field name="res_model">hr.expense.sheet</field>
            <field name="view_mode">tree,kanban,form,pivot,graph</field>
            <field name="search_view_id" ref="hr_expense_sheet_view_search"/>
            <field name="domain">[]</field>
            <field name="context">{'search_default_approved': 1}</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                No expense reports found
              </p><p>
                Reimburse the employees who incurred these costs or simply register the corresponding payments.
              </p>
            </field>
        </record>

        <record id="action_hr_expense_sheet_all" model="ir.actions.act_window">
            <field name="name">All Reports</field>
            <field name="res_model">hr.expense.sheet</field>
            <field name="view_mode">tree,kanban,form,pivot,graph</field>
            <field name="search_view_id" ref="hr_expense_sheet_view_search"/>
            <field name="domain">[]</field>
            <field name="context">{}</field>
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
            <field name="context">{}</field>
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
            <field name="search_view_id" ref="hr_expense_sheet_view_search"/>
            <field name="context">{
                'search_default_submitted': 1,
                'search_default_department_id': [active_id],
                'default_department_id': active_id
                }
            </field>
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

        <record id="hr_expense_submit_action_server" model="ir.actions.server">
            <field name="name">Create Report</field>
            <field name="type">ir.actions.server</field>
            <field name="model_id" ref="model_hr_expense"/>
            <field name="binding_model_id" ref="model_hr_expense"/>
            <field name="binding_view_types">list</field>
            <field name="state">code</field>
            <field name="code">
if records:
    action = records.action_submit_expenses()
            </field>
        </record>

        <record id="hr_expense_sheet_approve_action_server" model="ir.actions.server">
            <field name="name">Approve Report</field>
            <field name="type">ir.actions.server</field>
            <field name="model_id" ref="model_hr_expense_sheet"/>
            <field name="binding_model_id" ref="model_hr_expense_sheet"/>
            <field name="binding_view_types">list</field>
            <field name="groups_id" eval="[(4,ref('hr_expense.group_hr_expense_team_approver'))]"/>
            <field name="state">code</field>
            <field name="code">
if records:
    action = records.approve_expense_sheets()
            </field>
        </record>

        <record id="hr_expense_sheet_post_action_server" model="ir.actions.server">
            <field name="name">Post Entries</field>
            <field name="type">ir.actions.server</field>
            <field name="model_id" ref="model_hr_expense_sheet"/>
            <field name="binding_model_id" ref="model_hr_expense_sheet"/>
            <field name="binding_view_types">list</field>
            <field name="groups_id" eval="[(4,ref('hr_expense.group_hr_expense_team_approver'))]"/>
            <field name="state">code</field>
            <field name="code">
if records:
    action = records.action_sheet_move_create()
            </field>
        </record>

        <record id="action_expense_sheet_register_payment" model="ir.actions.server">
            <field name="name">Register Payment</field>
            <field name="groups_id" eval="[(4, ref('account.group_account_invoice'))]"/>
            <field name="model_id" ref="hr_expense.model_hr_expense_sheet"/>
            <field name="binding_model_id" ref="hr_expense.model_hr_expense_sheet"/>
            <field name="binding_view_types">list</field>
            <field name="state">code</field>
            <field name="code">
                if records:
                    action = records.action_register_payment()
            </field>
        </record>

        <menuitem id="menu_hr_expense_root" name="Expenses" sequence="100" web_icon="hr_expense,static/description/icon.png"/>

        <menuitem id="menu_hr_expense_my_expenses" name="My Expenses" sequence="1" parent="menu_hr_expense_root" groups="base.group_user"/>
        <menuitem id="menu_hr_expense_my_expenses_to_submit" sequence="1" parent="menu_hr_expense_my_expenses" action="hr_expense_actions_my_unsubmitted" name="My Expenses to Report"/>
        <menuitem id="menu_hr_expense_my_expenses_all" sequence="2" parent="menu_hr_expense_my_expenses" action="hr_expense_actions_my_all" name="All My Expenses"/>
        <menuitem id="menu_hr_expense_sheet_my_reports" sequence="3" parent="menu_hr_expense_my_expenses" action="action_hr_expense_sheet_my_all" name="My Reports"/>

        <menuitem id="menu_hr_expense_report" name="Expense Reports" sequence="2" parent="menu_hr_expense_root"/>
        <menuitem id="menu_hr_expense_sheet_all_to_approve"
                  name="Reports to Approve" sequence="1" parent="menu_hr_expense_report"
                  action="action_hr_expense_sheet_all_to_approve"
                  groups="hr_expense.group_hr_expense_team_approver"/>
        <menuitem id="menu_hr_expense_sheet_all_to_post"
                  name="Reports to Post" sequence="2" parent="menu_hr_expense_report"
                  action="action_hr_expense_sheet_all_to_post"
                  groups="account.group_account_invoice,hr_expense.group_hr_expense_manager"/>
        <menuitem id="menu_hr_expense_sheet_all_to_pay"
                  name="Reports to Pay" sequence="3" parent="menu_hr_expense_report"
                  action="action_hr_expense_sheet_all_to_pay"
                  groups="account.group_account_invoice,hr_expense.group_hr_expense_manager"/>
        <menuitem id="menu_hr_expense_sheet_all"
                  name="All Reports" sequence="4" parent="menu_hr_expense_report"
                  action="action_hr_expense_sheet_all"
                  groups="account.group_account_invoice,hr_expense.group_hr_expense_manager"/>

        <menuitem id="menu_hr_expense_reports" name="Reporting" sequence="4" parent="menu_hr_expense_root" groups="hr_expense.group_hr_expense_manager"/>
        <menuitem id="menu_hr_expense_all_expenses" name="Expenses Analysis" sequence="0" parent="menu_hr_expense_reports" action="hr_expense_actions_all"/>

        <menuitem id="menu_hr_expense_configuration" name="Configuration" parent="menu_hr_expense_root"
            sequence="100"/>
        <menuitem id="menu_hr_product" name="Expense Products" parent="menu_hr_expense_configuration"
            action="hr_expense_product" groups="hr_expense.group_hr_expense_manager" sequence="10"/>

        <menuitem id="menu_hr_expense_account_employee_expenses" name="Employee Expenses" sequence="22" parent="account.menu_finance_payables" groups="hr_expense.group_hr_expense_user" action="action_hr_expense_account"/>
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
        <field name="domain">['|', ('res_model_id', '=', False), ('res_model_id.model', '=', 'hr.expense.sheet')]</field>
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
                <xpath expr="//div[hasclass('settings')]" position="inside">
                    <div class="app_settings_block" data-string="Expenses" string="Expenses" data-key="hr_expense" groups="hr_expense.group_hr_expense_manager">
                        <h2>Expenses</h2>
                        <div class="row mt16 o_settings_container" name="expenses_setting_container">
                            <div class="col-xs-12 col-md-6 o_setting_box"
                                id="create_expense_setting"
                                title="Send an email to this email alias with the receipt in attachment to create an expense in one click. If the first word of the mail subject contains the product's internal reference or the product name, the corresponding product will automatically be set. Type the expense amount in the mail subject to set it on the expense too.">
                                <div class="o_setting_left_pane">
                                    <field name="use_mailgateway"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label string="Incoming Emails" for="use_mailgateway"/>
                                    <div class="text-muted">
                                        Create expenses from incoming emails
                                    </div>
                                    <div class="content-group" attrs="{'invisible': ['|', ('use_mailgateway', '=',  False), ('alias_domain', 'in', ['localhost', '', False])]}">
                                        <div class="mt16">
                                            <label for="expense_alias_prefix" string="Alias" class="o_light_label"/>
                                            <field name="expense_alias_prefix" class="oe_inline"/>
                                            <span>@</span>
                                            <field name="alias_domain" class="oe_inline" readonly="1" force_save="1"/>
                                        </div>
                                    </div>
                                    <div class="content-group" attrs="{'invisible': ['|', ('use_mailgateway', '=',  False), ('alias_domain', 'not in', ['localhost', '', False])]}">
                                        <div class="mt16">
                                            <button type="action" name="base_setup.action_general_configuration" icon="fa-arrow-right" string="Setup your domain alias" class="btn-link"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-lg-6 col-12 o_setting_box" id="hr_payroll_accountant">
                                <div class="o_setting_left_pane">
                                    <field name="module_hr_payroll_expense" widget="upgrade_boolean"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="module_hr_payroll_expense" string="Reimburse in Payslip"/>
                                    <div class="text-muted">
                                        Reimburse expenses in payslips
                                    </div>
                                </div>
                            </div>
                            <div class="col-xs-12 col-md-6 o_setting_box" title="use OCR to fill data from a picture of the bill">
                                <div class="o_setting_left_pane">
                                    <field name="module_hr_expense_extract" widget="upgrade_boolean"/>
                                </div>
                                <div class="o_setting_right_pane" id="expense_extract_settings">
                                    <label string="Expense Digitalization (OCR)" for="module_hr_expense_extract"/>
                                    <span class="fa fa-lg fa-building-o" title="Values set here are company-specific."/>
                                    <div class="text-muted">
                                        Digitalize your receipts with OCR and Artificial Intelligence
                                    </div>
                                </div>   
                            </div>
                        </div>
                    </div>
                </xpath>
            </field>
        </record>

        <record id="action_hr_expense_configuration" model="ir.actions.act_window">
            <field name="name">Settings</field>
            <field name="type">ir.actions.act_window</field>
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
        expense_sheet = self.env['hr.expense.sheet'].search([('payment_mode', '=', 'own_account'), ('account_move_id', 'in', line.move_id.ids)])
        if expense_sheet and not line.move_id.partner_bank_id:
            res['partner_bank_id'] = expense_sheet.employee_id.sudo().bank_account_id.id or line.partner_id.bank_ids and line.partner_id.bank_ids.ids[0]
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

## File: wizard\hr_expense_refuse_reason.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class HrExpenseRefuseWizard(models.TransientModel):
    """This wizard can be launched from an he.expense (an expense line)
    or from an hr.expense.sheet (En expense report)
    'hr_expense_refuse_model' must be passed in the context to differentiate
    the right model to use.
    """

    _name = "hr.expense.refuse.wizard"
    _description = "Expense Refuse Reason Wizard"

    reason = fields.Char(string='Reason', required=True)
    hr_expense_ids = fields.Many2many('hr.expense')
    hr_expense_sheet_id = fields.Many2one('hr.expense.sheet')

    @api.model
    def default_get(self, fields):
        res = super(HrExpenseRefuseWizard, self).default_get(fields)
        active_ids = self.env.context.get('active_ids', [])
        refuse_model = self.env.context.get('hr_expense_refuse_model')
        if refuse_model == 'hr.expense':
            res.update({
                'hr_expense_ids': active_ids,
                'hr_expense_sheet_id': False,
            })
        elif refuse_model == 'hr.expense.sheet':
            res.update({
                'hr_expense_sheet_id': active_ids[0] if active_ids else False,
                'hr_expense_ids': [],
            })
        return res

    def expense_refuse_reason(self):
        self.ensure_one()
        if self.hr_expense_ids:
            self.hr_expense_ids.refuse_expense(self.reason)
        if self.hr_expense_sheet_id:
            self.hr_expense_sheet_id.refuse_sheet(self.reason)

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
                <field name="hr_expense_ids" invisible="1"/>
                <field name="hr_expense_sheet_id" invisible="1"/>
                <field name="reason"/>
                <footer>
                    <button string='Refuse' name="expense_refuse_reason" type="object" class="oe_highlight"/>
                    <button string="Cancel" class="oe_link" special="cancel"/>
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

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-

from . import hr_expense_refuse_reason
from . import account_payment_register

```

