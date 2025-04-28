# Odoo Module: sale_timesheet

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import controllers
from . import models
from . import wizard
from . import report

from odoo import api, SUPERUSER_ID

def uninstall_hook(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})

    env.ref("account.account_analytic_line_rule_billing_user").write({'domain_force': "[(1, '=', 1)]"})

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Sales Timesheet',
    'category': 'Hidden',
    'summary': 'Sell based on timesheets',
    'description': """
Allows to sell timesheets in your sales order
=============================================

This module set the right product on all timesheet lines
according to the order/contract you work on. This allows to
have real delivered quantities in sales orders.
""",
    'depends': ['sale_project', 'hr_timesheet'],
    'data': [
        'data/sale_service_data.xml',
        'security/ir.model.access.csv',
        'security/sale_timesheet_security.xml',
        'views/account_invoice_views.xml',
        'views/sale_order_views.xml',
        'views/product_views.xml',
        'views/project_task_views.xml',
        'views/hr_timesheet_views.xml',
        'views/res_config_settings_views.xml',
        'views/hr_timesheet_templates.xml',
        'views/report_invoice.xml',
        'views/sale_timesheet_portal_templates.xml',
        'report/project_profitability_report_analysis_views.xml',
        'data/sale_timesheet_filters.xml',
        'wizard/project_create_sale_order_views.xml',
        'wizard/project_create_invoice_views.xml',
        'wizard/sale_make_invoice_advance_views.xml',
        'wizard/project_task_create_sale_order_views.xml'
    ],
    'qweb': [
        'static/src/xml/sale_project_templates.xml',
    ],
    'demo': [
        'data/sale_service_demo.xml',
    ],
    'auto_install': True,
    'uninstall_hook': 'uninstall_hook',
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http, _
from odoo.http import request
from odoo.osv import expression

from odoo.addons.account.controllers import portal
from odoo.addons.hr_timesheet.controllers.portal import TimesheetCustomerPortal


class PortalAccount(portal.PortalAccount):

    def _invoice_get_page_view_values(self, invoice, access_token, **kwargs):
        values = super(PortalAccount, self)._invoice_get_page_view_values(invoice, access_token, **kwargs)
        domain = request.env['account.analytic.line']._timesheet_get_portal_domain()
        domain = expression.AND([
            domain,
            request.env['account.analytic.line']._timesheet_get_sale_domain(
                invoice.mapped('line_ids.sale_line_ids'),
                request.env['account.move'].browse([invoice.id])
            )
        ])
        values['timesheets'] = request.env['account.analytic.line'].sudo().search(domain)
        values['is_uom_day'] = request.env['account.analytic.line'].sudo()._is_timesheet_encode_uom_day()
        return values


class CustomerPortal(portal.CustomerPortal):
    def _order_get_page_view_values(self, order, access_token, **kwargs):
        values = super(CustomerPortal, self)._order_get_page_view_values(order, access_token, **kwargs)
        domain = request.env['account.analytic.line']._timesheet_get_portal_domain()
        domain = expression.AND([
            domain,
            request.env['account.analytic.line']._timesheet_get_sale_domain(
                order.mapped('order_line'),
                order.invoice_ids
            )
        ])
        values['timesheets'] = request.env['account.analytic.line'].sudo().search(domain)
        values['is_uom_day'] = request.env['account.analytic.line'].sudo()._is_timesheet_encode_uom_day()
        return values


class SaleTimesheetCustomerPortal(TimesheetCustomerPortal):

    def _get_searchbar_inputs(self):
        searchbar_inputs = super()._get_searchbar_inputs()
        searchbar_inputs.update(
            sol={'input': 'sol', 'label': _('Search in Sales Order Item')},
            sol_id={'input': 'sol_id', 'label': _('Search in Sales Order Item ID')},
            invoice={'input': 'invoice_id', 'label': _('Search in Invoice ID')})
        return searchbar_inputs

    def _get_searchbar_groupby(self):
        searchbar_groupby = super()._get_searchbar_groupby()
        searchbar_groupby.update(sol={'input': 'sol', 'label': _('Sales Order Item')})
        return searchbar_groupby

    def _get_search_domain(self, search_in, search):
        search_domain = super()._get_search_domain(search_in, search)
        if search_in in ('sol', 'all'):
            search_domain = expression.OR([search_domain, [('so_line', 'ilike', search)]])
        if search_in in ('sol_id', 'invoice_id'):
            search = int(search) if search.isdigit() else 0
        if search_in == 'sol_id':
            search_domain = expression.OR([search_domain, [('so_line.id', '=', search)]])
        if search_in == 'invoice_id':
            invoice = request.env['account.move'].browse(search)
            domain = request.env['account.analytic.line']._timesheet_get_sale_domain(invoice.mapped('invoice_line_ids.sale_line_ids'), invoice)
            search_domain = expression.OR([search_domain, domain])
        return search_domain

    def _get_groupby_mapping(self):
        groupby_mapping = super()._get_groupby_mapping()
        groupby_mapping.update(sol='so_line')
        return groupby_mapping

    @http.route(['/my/timesheets', '/my/timesheets/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_timesheets(self, page=1, sortby=None, filterby=None, search=None, search_in='all', groupby='sol', **kw):
        if search and search_in and search_in in ('sol_id', 'invoice_id') and not search.isdigit():
            search = '0'
        return super().portal_my_timesheets(page, sortby, filterby, search, search_in, groupby, **kw)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\sale_service_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="time_product" model="product.product">
            <field name="name">Service on Timesheet</field>
            <field name="type">service</field>
            <field name="list_price">40</field>
            <field name="uom_id" ref="uom.product_uom_hour"/>
            <field name="uom_po_id" ref="uom.product_uom_hour"/>
            <field name="service_policy">delivered_timesheet</field>
            <field name="image_1920" type="base64" file="sale_timesheet/static/img/product_product_time_product.png"/>
        </record>
    </data>
    <data>
        <record model="res.groups" id="base.group_user">
            <field name="implied_ids" eval="[(4, ref('uom.group_uom'))]"/>
        </record>
    </data>
</odoo>

```

## File: data\sale_service_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="sale_line_services" model="sale.order.line">
            <field name="order_id" ref="sale.sale_order_3"/>
            <field name="name" model="sale.order.line" eval="obj().env.ref('sale.advance_product_0').get_product_multiline_description_sale()"/>
            <field name="product_id" ref="sale.advance_product_0"/>
            <field name="product_uom" ref="uom.product_uom_unit"/>
            <field name="price_unit">150.0</field>
            <field name="product_uom_qty">5.0</field>
        </record>

        <!-- Projects and Analytic Account -->
        <record id="account_analytic_account_project_support" model="account.analytic.account">
            <field name="name">After-Sales Services</field>
            <field name="code">INT</field>
            <field name="active" eval="True"/>
        </record>

        <record id="project_support" model="project.project">
            <field name="date_start" eval="time.strftime('%Y-%m-01 10:00:00')"/>
            <field name="name">After-Sales Services</field>
            <field name="analytic_account_id" ref="account_analytic_account_project_support"/>
            <field name="allow_billable" eval="True" />
            <field name="type_ids" eval="[(4, ref('project.project_stage_0')), (4, ref('project.project_stage_1')), (4, ref('project.project_stage_2'))]"/>
        </record>

        <!-- Project Task -->
        <record id="project_task_internal" model="project.task">
            <field name="name">Internal training</field>
            <field name="user_id" ref="base.user_admin"/>
            <field name="project_id" ref="project.project_project_1"/>
        </record>

        <!-- Products -->
        <record id="product.product_product_2" model="product.product">
            <field name="service_type">timesheet</field>
            <field name="service_tracking">project_only</field>
        </record>

        <record id="product.product_product_1" model="product.product">
            <field name="service_type">timesheet</field>
            <field name="service_tracking">task_global_project</field>
        </record>

        <record id="product_service_order_timesheet" model="product.product">
            <field name="name">Customer Care (Prepaid Hours)</field>
            <field name="categ_id" ref="product.product_category_3"/>
            <field name="type">service</field>
            <field name="list_price">250.00</field>
            <field name="standard_price">190.00</field>
            <field name="uom_id" ref="uom.product_uom_hour"/>
            <field name="uom_po_id" ref="uom.product_uom_hour"/>
            <field name="service_policy">ordered_timesheet</field>
            <field name="service_tracking">task_global_project</field>
            <field name="project_id" ref="project_support"/>
        </record>

        <record id="product_service_deliver_timesheet_1" model="product.product">
            <field name="name">Senior Architect (Invoice on Timesheets)</field>
            <field name="categ_id" ref="product.product_category_3"/>
            <field name="list_price">200.00</field>
            <field name="standard_price">150.00</field>
            <field name="type">service</field>
            <field name="uom_id" ref="uom.product_uom_hour"/>
            <field name="uom_po_id" ref="uom.product_uom_hour"/>
            <field name="service_policy">delivered_timesheet</field>
            <field name="service_tracking">task_in_project</field>
        </record>

        <record id="product_service_deliver_timesheet_2" model="product.product">
            <field name="name">Junior Architect (Invoice on Timesheets)</field>
            <field name="categ_id" ref="product.product_category_3"/>
            <field name="list_price">100.00</field>
            <field name="standard_price">85.00</field>
            <field name="type">service</field>
            <field name="uom_id" ref="uom.product_uom_hour"/>
            <field name="uom_po_id" ref="uom.product_uom_hour"/>
            <field name="service_policy">delivered_timesheet</field>
            <field name="service_tracking">task_in_project</field>
        </record>

        <record id="product_service_deliver_manual" model="product.product">
            <field name="name">Kitchen Assembly (Milestones)</field>
            <field name="categ_id" ref="product.product_category_3"/>
            <field name="list_price">500</field>
            <field name="standard_price">420.00</field>
            <field name="type">service</field>
            <field name="uom_id" ref="uom.product_uom_unit"/>
            <field name="uom_po_id" ref="uom.product_uom_unit"/>
            <field name="service_policy">delivered_manual</field>
            <field name="service_tracking">no</field>
        </record>

        <!-- Sales orders -->
        <record id="sale_order_1" model="sale.order">
            <field name="partner_id" ref="base.res_partner_2"/>
            <field name="client_order_ref">AGR</field>
            <field name="user_id" ref="base.user_admin"/>
            <field name="tag_ids" eval="[(4, ref('sales_team.categ_oppor6'))]"/>
        </record>

        <record id="sale_line_11" model="sale.order.line">
            <field name="order_id" ref="sale_order_1"/>
            <field name="sequence" eval="1"/>
            <field name="product_id" ref="product_service_order_timesheet"/>
            <field name="product_uom_qty">20</field>
        </record>
        <record id="sale_line_12" model="sale.order.line">
            <field name="order_id" ref="sale_order_1"/>
            <field name="sequence" eval="3"/>
            <field name="product_id" ref="product_service_deliver_manual"/>
            <field name="product_uom_qty">4</field>
        </record>
        <record id="sale_line_13" model="sale.order.line">
            <field name="order_id" ref="sale_timesheet.sale_order_1"/>
            <field name="product_id" ref="product_service_deliver_timesheet_1"/>
            <field name="sequence" eval="2"/>
            <field name="discount">10</field>
            <field name="product_uom_qty">50</field>
        </record>

        <!-- Sale Order 'sale_order_2' (Delta PC) -->
        <record id="sale_order_2" model="sale.order">
            <field name="partner_id" ref="base.res_partner_4"/>
            <field name="client_order_ref">DPC</field>
            <field name="user_id" ref="base.user_admin"/>
            <field name="tag_ids" eval="[(4, ref('sales_team.categ_oppor4')), (4, ref('sales_team.categ_oppor7'))]"/>
        </record>

        <record id="sale_line_21" model="sale.order.line">
            <field name="order_id" ref="sale_order_2"/>
            <field name="sequence" eval="1"/>
            <field name="product_id" ref="product_service_order_timesheet"/>
            <field name="product_uom_qty">150</field>
        </record>
        <record id="sale_line_22" model="sale.order.line">
            <field name="order_id" ref="sale_timesheet.sale_order_2"/>
            <field name="sequence" eval="2"/>
            <field name="product_id" ref="product_service_deliver_timesheet_2"/>
            <field name="product_uom_qty">10</field>
        </record>

        <!-- Activity of sales order -->
        <record id="sale_timesheet_activity_1" model="mail.activity">
            <field name="res_id" ref="sale_timesheet.sale_order_1"/>
            <field name="res_model_id" ref="sale.model_sale_order"/>
            <field name="activity_type_id" ref="mail.mail_activity_data_call"/>
            <field name="date_deadline" eval="(DateTime.today() + relativedelta(days=5)).strftime('%Y-%m-%d %H:%M')"/>
            <field name="summary">Call to follow-up</field>
            <field name="create_uid" ref="base.user_admin"/>
            <field name="user_id" ref="base.user_admin"/>
        </record>

        <!-- Confirm Sale Orders -->
        <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_1')]]"/>
        <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_2')]]"/>

        <!-- Add project to favorite list of admin -->
        <function model="project.project" name="write">
            <value model="project.project" eval="obj().search([('sale_line_id', '=', ref('sale_line_13'))]).ids"/>
            <value eval="{'favorite_user_ids': [(4, ref('base.user_admin'))]}"/>
        </function>

        <!-- Assign sale order's task to admin -->
        <function model="project.task" name="write">
            <value model="project.task" eval="obj().search([('sale_line_id', '=', ref('sale_line_13'))]).ids"/>
            <value eval="{'user_id': ref('base.user_admin')}"/>
        </function>

        <!-- Timesheets on sale_order_1 -->
        <record id="timesheet_1" model="account.analytic.line">
            <field name="name">Design</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=3,weeks=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">5.00</field>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_13'))]"/>
            <field name="project_id" search="[('sale_line_id', '=', ref('sale_line_13'))]"/>
        </record>
        <record id="timesheet_2" model="account.analytic.line">
            <field name="name">Fine tuning</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=2,weeks=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">5.00</field>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_13'))]"/>
            <field name="project_id" search="[('sale_line_id', '=', ref('sale_line_13'))]"/>
        </record>
        <record id="timesheet_3" model="account.analytic.line">
            <field name="name">Assembling</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=0)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">5.00</field>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_13'))]"/>
            <field name="project_id" search="[('sale_line_id', '=', ref('sale_line_13'))]"/>
        </record>
        <record id="timesheet_4" model="account.analytic.line">
            <field name="name">Delivery</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=1,weeks=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">5.00</field>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_13'))]"/>
            <field name="project_id" search="[('sale_line_id', '=', ref('sale_line_13'))]"/>
        </record>

        <record id="timesheet_5" model="account.analytic.line">
            <field name="name">Requirements analysis</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=0,weeks=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1.00</field>
            <field name="project_id" ref="project_support"/>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_11'))]"/>
        </record>
        <record id="timesheet_6" model="account.analytic.line">
            <field name="name">Client meeting</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=1,weeks=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1.00</field>
            <field name="project_id" ref="project_support"/>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_11'))]"/>
        </record>
        <record id="timesheet_7" model="account.analytic.line">
            <field name="name">Requirements check</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=2,weeks=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1.00</field>
            <field name="project_id" ref="project_support"/>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_11'))]"/>
        </record>
        <record id="timesheet_8" model="account.analytic.line">
            <field name="name">Requirements analysis</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=3,weeks=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1.00</field>
            <field name="project_id" ref="project_support"/>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_11'))]"/>
        </record>
        <record id="timesheet_9" model="account.analytic.line">
            <field name="name">Building</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=4,weeks=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1.00</field>
            <field name="project_id" ref="project_support"/>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_11'))]"/>
        </record>
        <record id="timesheet_10" model="account.analytic.line">
            <field name="name">Research</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=0,weeks=-3)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1.00</field>
            <field name="project_id" ref="project_support"/>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_11'))]"/>
        </record>
        <record id="timesheet_11" model="account.analytic.line">
            <field name="name">Assembling</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=1,weeks=-3)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1.00</field>
            <field name="project_id" ref="project_support"/>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_11'))]"/>
        </record>
        <record id="timesheet_12" model="account.analytic.line">
            <field name="name">Quality  check</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=2,weeks=-3)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1.00</field>
            <field name="project_id" ref="project_support"/>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_11'))]"/>
        </record>
        <record id="timesheet_13" model="account.analytic.line">
            <field name="name">Assembling</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=3,weeks=-3)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1.00</field>
            <field name="project_id" ref="project_support"/>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_11'))]"/>
        </record>
        <record id="timesheet_14" model="account.analytic.line">
            <field name="name">Wood chopping</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=4,weeks=-3)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1.00</field>
            <field name="project_id" ref="project_support"/>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_11'))]"/>
        </record>

        <!-- Timesheets on sale_order_2 -->
        <record id="timesheet_15" model="account.analytic.line">
            <field name="name">Research and Development</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=1,weeks=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">8.00</field>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_22'))]"/>
            <field name="project_id" search="[('sale_line_id', '=', ref('sale_line_22'))]"/>
        </record>
        <record id="timesheet_16" model="account.analytic.line">
            <field name="name">Quality analysis</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=2,weeks=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">8.00</field>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_22'))]"/>
            <field name="project_id" search="[('sale_line_id', '=', ref('sale_line_22'))]"/>
        </record>
        <record id="timesheet_17" model="account.analytic.line">
            <field name="name">Repair</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=0)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">8.00</field>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_22'))]"/>
            <field name="project_id" search="[('sale_line_id', '=', ref('sale_line_22'))]"/>
        </record>
        <record id="timesheet_18" model="account.analytic.line">
            <field name="name">Initial design improvement</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=4,weeks=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">8.00</field>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_22'))]"/>
            <field name="project_id" search="[('sale_line_id', '=', ref('sale_line_22'))]"/>
        </record>

        <record id="timesheet_19" model="account.analytic.line">
            <field name="name">Knowledge transfer</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=0,weeks=-5)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">4.00</field>
            <field name="project_id" ref="project_support"/>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_21'))]"/>
        </record>
        <record id="timesheet_20" model="account.analytic.line">
            <field name="name">Document analysis</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=0,weeks=-4)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">4.00</field>
            <field name="project_id" ref="project_support"/>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_21'))]"/>
        </record>
        <record id="timesheet_21" model="account.analytic.line">
            <field name="name">Design analysis</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=0,weeks=-3)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">4.00</field>
            <field name="project_id" ref="project_support"/>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_21'))]"/>
        </record>
        <record id="timesheet_22" model="account.analytic.line">
            <field name="name">Requirements meeting</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="date" eval="(DateTime.now() + relativedelta(weekday=0,weeks=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">4.00</field>
            <field name="project_id" ref="project_support"/>
            <field name="task_id" search="[('sale_line_id', '=', ref('sale_line_21'))]"/>
        </record>

        <!-- Non billable Timesheets in project_support -->
        <record id="timesheet_23" model="account.analytic.line">
            <field name="name">Technical training</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(months=-4, days=-10)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">8.00</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="task_id" ref="project_task_internal"/>
        </record>
        <record id="timesheet_24" model="account.analytic.line">
            <field name="name">Internal training</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(months=-4, days=-12)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">8.00</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="task_id" ref="project_task_internal"/>
        </record>
        <record id="timesheet_25" model="account.analytic.line">
            <field name="name">Internal discussion</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(months=-4, days=-13)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">8.00</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="task_id" ref="project_task_internal"/>
        </record>
        <record id="timesheet_26" model="account.analytic.line">
            <field name="name">Details improvement</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="date" eval="(DateTime.now() + relativedelta(months=-4, days=-11)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">8.00</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="task_id" ref="project_task_internal"/>
        </record>

        <!-- Vendor bill for sale_order_1 -->
        <record id="account_analytic_line_inv_1" model="account.analytic.line">
            <field name="name" model="account.analytic.line" eval="obj().env.ref('product.product_product_3').get_product_multiline_description_sale()"/>
            <field name="account_id" search="[('partner_id', '=', ref('base.res_partner_2'))]"/>
            <field name="partner_id" ref="base.partner_root"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=0)).strftime('%Y-%m-%d')"/>
            <field name="amount">-300.00</field>
            <field name="product_id" ref="product.product_product_3"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="unit_amount">10.00</field>
        </record>

        <!-- Expense bill for sale_order_1 -->
        <record id="account_analytic_line_exp_1" model="account.analytic.line">
            <field name="name" model="account.analytic.line" eval="obj().env.ref('product.expense_product').get_product_multiline_description_sale()"/>
            <field name="account_id" search="[('partner_id', '=', ref('base.res_partner_2'))]"/>
            <field name="partner_id" ref="base.partner_root"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=0)).strftime('%Y-%m-%d')"/>
            <field name="amount">-100.00</field>
            <field name="product_id" ref="product.expense_product"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="unit_amount">1.00</field>
        </record>

        <!-- Vendor bill for sale_order_2 -->
        <record id="account_analytic_line_inv_2" model="account.analytic.line">
            <field name="name" model="account.analytic.line" eval="obj().env.ref('product.product_product_3').get_product_multiline_description_sale()"/>
            <field name="account_id" search="[('partner_id', '=', ref('base.res_partner_4'))]"/>
            <field name="partner_id" ref="base.partner_root"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=0)).strftime('%Y-%m-%d')"/>
            <field name="amount">-400.00</field>
            <field name="product_id" ref="product.product_product_3"/>
            <field name="product_uom_id" ref="uom.product_uom_unit"/>
            <field name="unit_amount">15.00</field>
        </record>

        <!-- Expense bill for sale_order_2 -->
        <record id="account_analytic_line_exp_2" model="account.analytic.line">
            <field name="name" model="account.analytic.line" eval="obj().env.ref('product.expense_hotel').get_product_multiline_description_sale()"/>
            <field name="account_id" search="[('partner_id', '=', ref('base.res_partner_4'))]"/>
            <field name="partner_id" ref="base.partner_demo"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=0)).strftime('%Y-%m-%d')"/>
            <field name="amount">-75.00</field>
            <field name="product_id" ref="product.expense_hotel"/>
            <field name="product_uom_id" ref="uom.product_uom_day"/>
            <field name="unit_amount">1.00</field>
        </record>

        <record id="project.project_stage_1" model="project.task.type">
            <field name="project_ids" model="project.project" eval="[(4, obj().search([('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))], limit=1).id)]"/>
        </record>

        <record id="project.project_stage_2" model="project.task.type">
            <field name="project_ids" model="project.project" eval="[(4, obj().search([('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))], limit=1).id)]"/>
        </record>

        <record id="project_task_1" model="project.task">
            <field name="name">Decoration</field>
            <field name="sale_line_id" ref="sale_timesheet.sale_line_13"/>
            <field name="sale_order_id" ref="sale_timesheet.sale_order_1"/>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="stage_id" ref="project.project_stage_2"/>
            <field name="sale_line_id" ref="sale_timesheet.sale_line_13" />
            <field name="sale_order_id" ref="sale_timesheet.sale_order_1" />
            <field name="partner_id" ref="base.res_partner_2" />
        </record>

        <record id="project_task_2" model="project.task">
            <field name="name">Planning</field>
            <field name="sale_line_id" ref="sale_timesheet.sale_line_13"/>
            <field name="sale_order_id" ref="sale_timesheet.sale_order_1"/>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="stage_id" ref="project.project_stage_2"/>
            <field name="sale_line_id" ref="sale_timesheet.sale_line_13" />
            <field name="sale_order_id" ref="sale_timesheet.sale_order_1" />
            <field name="partner_id" ref="base.res_partner_2" />
        </record>

        <record id="project_task_3" model="project.task">
            <field name="name">Furniture</field>
            <field name="sale_line_id" ref="sale_timesheet.sale_line_13"/>
            <field name="sale_order_id" ref="sale_timesheet.sale_order_1"/>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="stage_id" ref="project.project_stage_2"/>
            <field name="sale_line_id" ref="sale_timesheet.sale_line_13" />
            <field name="sale_order_id" ref="sale_timesheet.sale_order_1" />
            <field name="partner_id" ref="base.res_partner_2" />
        </record>

        <record id="project_task_4" model="project.task">
            <field name="name">Furniture Delivery</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="stage_id" ref="project.project_stage_1" />
            <field name="sale_line_id" ref="sale_timesheet.sale_line_13" />
            <field name="sale_order_id" ref="sale_timesheet.sale_order_1" />
            <field name="partner_id" ref="base.res_partner_2" />
            <field name="user_id" ref="base.user_admin"/>
        </record>

        <!-- Timesheet for those tasks -->
        <record id="account_analytic_line_0" model="account.analytic.line">
            <field name="name">On Site Visit</field>
            <field name="employee_id" ref="hr.employee_fme"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_1" model="account.analytic.line">
            <field name="name">Quality analysis</field>
            <field name="employee_id" ref="hr.employee_jgo"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_2" model="account.analytic.line">
            <field name="name">Delivery</field>
            <field name="employee_id" ref="hr.employee_vad"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_3" model="account.analytic.line">
            <field name="name">Requirements analysis</field>
            <field name="employee_id" ref="hr.employee_chs"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_4" model="account.analytic.line">
            <field name="name">On Site Visit</field>
            <field name="employee_id" ref="hr.employee_jve"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_5" model="account.analytic.line">
            <field name="name">Call</field>
            <field name="employee_id" ref="hr.employee_jog"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_6" model="account.analytic.line">
            <field name="name">On Site Visit</field>
            <field name="employee_id" ref="hr.employee_han"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">3</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-90.0</field>
        </record>

        <record id="account_analytic_line_7" model="account.analytic.line">
            <field name="name">Call</field>
            <field name="employee_id" ref="hr.employee_mit"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_8" model="account.analytic.line">
            <field name="name">Sprint</field>
            <field name="employee_id" ref="hr.employee_al"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_9" model="account.analytic.line">
            <field name="name">Design</field>
            <field name="employee_id" ref="hr.employee_hne"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_10" model="account.analytic.line">
            <field name="name">Quality analysis</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_11" model="account.analytic.line">
            <field name="name">On Site Visit</field>
            <field name="employee_id" ref="hr.employee_jth"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-1)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_12" model="account.analytic.line">
            <field name="name">Quality analysis</field>
            <field name="employee_id" ref="hr.employee_ngh"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_13" model="account.analytic.line">
            <field name="name">Training</field>
            <field name="employee_id" ref="hr.employee_jve"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_14" model="account.analytic.line">
            <field name="name">Sprint</field>
            <field name="employee_id" ref="hr.employee_jve"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_15" model="account.analytic.line">
            <field name="name">Requirements analysis</field>
            <field name="employee_id" ref="hr.employee_jth"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_16" model="account.analytic.line">
            <field name="name">Sprint</field>
            <field name="employee_id" ref="hr.employee_han"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">3</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-90.0</field>
        </record>

        <record id="account_analytic_line_17" model="account.analytic.line">
            <field name="name">Delivery</field>
            <field name="employee_id" ref="hr.employee_jod"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">3</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-90.0</field>
        </record>

        <record id="account_analytic_line_18" model="account.analytic.line">
            <field name="name">Quality analysis</field>
            <field name="employee_id" ref="hr.employee_jod"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_19" model="account.analytic.line">
            <field name="name">Requirements analysis</field>
            <field name="employee_id" ref="hr.employee_vad"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_20" model="account.analytic.line">
            <field name="name">Design</field>
            <field name="employee_id" ref="hr.employee_fpi"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">3</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-90.0</field>
        </record>

        <record id="account_analytic_line_21" model="account.analytic.line">
            <field name="name">Presentation</field>
            <field name="employee_id" ref="hr.employee_ngh"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_22" model="account.analytic.line">
            <field name="name">Delivery</field>
            <field name="employee_id" ref="hr.employee_fme"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_23" model="account.analytic.line">
            <field name="name">Delivery</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-2)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_24" model="account.analytic.line">
            <field name="name">Design</field>
            <field name="employee_id" ref="hr.employee_stw"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_25" model="account.analytic.line">
            <field name="name">Training</field>
            <field name="employee_id" ref="hr.employee_han"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_26" model="account.analytic.line">
            <field name="name">Presentation</field>
            <field name="employee_id" ref="hr.employee_hne"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_27" model="account.analytic.line">
            <field name="name">Sprint</field>
            <field name="employee_id" ref="hr.employee_vad"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_28" model="account.analytic.line">
            <field name="name">Call</field>
            <field name="employee_id" ref="hr.employee_stw"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_29" model="account.analytic.line">
            <field name="name">Call</field>
            <field name="employee_id" ref="hr.employee_hne"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_30" model="account.analytic.line">
            <field name="name">Call</field>
            <field name="employee_id" ref="hr.employee_fme"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_31" model="account.analytic.line">
            <field name="name">Presentation</field>
            <field name="employee_id" ref="hr.employee_jod"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_32" model="account.analytic.line">
            <field name="name">On Site Visit</field>
            <field name="employee_id" ref="hr.employee_fme"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_33" model="account.analytic.line">
            <field name="name">Delivery</field>
            <field name="employee_id" ref="hr.employee_stw"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">3</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-90.0</field>
        </record>

        <record id="account_analytic_line_34" model="account.analytic.line">
            <field name="name">Design</field>
            <field name="employee_id" ref="hr.employee_stw"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_35" model="account.analytic.line">
            <field name="name">Delivery</field>
            <field name="employee_id" ref="hr.employee_jth"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-3)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_36" model="account.analytic.line">
            <field name="name">Quality analysis</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_37" model="account.analytic.line">
            <field name="name">Quality analysis</field>
            <field name="employee_id" ref="hr.employee_lur"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_38" model="account.analytic.line">
            <field name="name">Design</field>
            <field name="employee_id" ref="hr.employee_niv"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_39" model="account.analytic.line">
            <field name="name">Requirements analysis</field>
            <field name="employee_id" ref="hr.employee_jve"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_40" model="account.analytic.line">
            <field name="name">Presentation</field>
            <field name="employee_id" ref="hr.employee_mit"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_41" model="account.analytic.line">
            <field name="name">On Site Visit</field>
            <field name="employee_id" ref="hr.employee_vad"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">3</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-90.0</field>
        </record>

        <record id="account_analytic_line_42" model="account.analytic.line">
            <field name="name">Requirements analysis</field>
            <field name="employee_id" ref="hr.employee_jep"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">3</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-90.0</field>
        </record>

        <record id="account_analytic_line_43" model="account.analytic.line">
            <field name="name">Requirements analysis</field>
            <field name="employee_id" ref="hr.employee_jgo"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_44" model="account.analytic.line">
            <field name="name">Design</field>
            <field name="employee_id" ref="hr.employee_mit"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_45" model="account.analytic.line">
            <field name="name">Presentation</field>
            <field name="employee_id" ref="hr.employee_ngh"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_46" model="account.analytic.line">
            <field name="name">Requirements analysis</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_47" model="account.analytic.line">
            <field name="name">Training</field>
            <field name="employee_id" ref="hr.employee_lur"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-4)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_48" model="account.analytic.line">
            <field name="name">Presentation</field>
            <field name="employee_id" ref="hr.employee_chs"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_49" model="account.analytic.line">
            <field name="name">Call</field>
            <field name="employee_id" ref="hr.employee_jth"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_50" model="account.analytic.line">
            <field name="name">Design</field>
            <field name="employee_id" ref="hr.employee_jog"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_51" model="account.analytic.line">
            <field name="name">On Site Visit</field>
            <field name="employee_id" ref="hr.employee_jod"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_52" model="account.analytic.line">
            <field name="name">Design</field>
            <field name="employee_id" ref="hr.employee_jgo"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_53" model="account.analytic.line">
            <field name="name">Delivery</field>
            <field name="employee_id" ref="hr.employee_jog"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_54" model="account.analytic.line">
            <field name="name">Presentation</field>
            <field name="employee_id" ref="hr.employee_mit"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_55" model="account.analytic.line">
            <field name="name">Sprint</field>
            <field name="employee_id" ref="hr.employee_jep"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_56" model="account.analytic.line">
            <field name="name">Training</field>
            <field name="employee_id" ref="hr.employee_stw"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_57" model="account.analytic.line">
            <field name="name">Requirements analysis</field>
            <field name="employee_id" ref="hr.employee_mit"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_58" model="account.analytic.line">
            <field name="name">Delivery</field>
            <field name="employee_id" ref="hr.employee_fpi"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_59" model="account.analytic.line">
            <field name="name">On Site Visit</field>
            <field name="employee_id" ref="hr.employee_jgo"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-5)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_60" model="account.analytic.line">
            <field name="name">Call</field>
            <field name="employee_id" ref="hr.employee_jgo"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_61" model="account.analytic.line">
            <field name="name">Sprint</field>
            <field name="employee_id" ref="hr.employee_vad"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">3</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-90.0</field>
        </record>

        <record id="account_analytic_line_62" model="account.analytic.line">
            <field name="name">Presentation</field>
            <field name="employee_id" ref="hr.employee_niv"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_63" model="account.analytic.line">
            <field name="name">Design</field>
            <field name="employee_id" ref="hr.employee_jog"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">3</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-90.0</field>
        </record>

        <record id="account_analytic_line_64" model="account.analytic.line">
            <field name="name">Quality analysis</field>
            <field name="employee_id" ref="hr.employee_stw"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">3</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-90.0</field>
        </record>

        <record id="account_analytic_line_65" model="account.analytic.line">
            <field name="name">Presentation</field>
            <field name="employee_id" ref="hr.employee_jog"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">3</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-90.0</field>
        </record>

        <record id="account_analytic_line_66" model="account.analytic.line">
            <field name="name">Presentation</field>
            <field name="employee_id" ref="hr.employee_mit"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_67" model="account.analytic.line">
            <field name="name">Requirements analysis</field>
            <field name="employee_id" ref="hr.employee_vad"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_68" model="account.analytic.line">
            <field name="name">Sprint</field>
            <field name="employee_id" ref="hr.employee_fpi"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_69" model="account.analytic.line">
            <field name="name">On Site Visit</field>
            <field name="employee_id" ref="hr.employee_jve"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_70" model="account.analytic.line">
            <field name="name">Presentation</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_71" model="account.analytic.line">
            <field name="name">Presentation</field>
            <field name="employee_id" ref="hr.employee_jth"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-6)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_72" model="account.analytic.line">
            <field name="name">Sprint</field>
            <field name="employee_id" ref="hr.employee_jog"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_73" model="account.analytic.line">
            <field name="name">Call</field>
            <field name="employee_id" ref="hr.employee_mit"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_74" model="account.analytic.line">
            <field name="name">Presentation</field>
            <field name="employee_id" ref="hr.employee_jog"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_75" model="account.analytic.line">
            <field name="name">On Site Visit</field>
            <field name="employee_id" ref="hr.employee_jod"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_76" model="account.analytic.line">
            <field name="name">Training</field>
            <field name="employee_id" ref="hr.employee_al"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_77" model="account.analytic.line">
            <field name="name">Delivery</field>
            <field name="employee_id" ref="hr.employee_jog"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_78" model="account.analytic.line">
            <field name="name">Call</field>
            <field name="employee_id" ref="hr.employee_ngh"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_79" model="account.analytic.line">
            <field name="name">Sprint</field>
            <field name="employee_id" ref="hr.employee_hne"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_80" model="account.analytic.line">
            <field name="name">Requirements analysis</field>
            <field name="employee_id" ref="hr.employee_ngh"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_81" model="account.analytic.line">
            <field name="name">Requirements analysis</field>
            <field name="employee_id" ref="hr.employee_stw"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_82" model="account.analytic.line">
            <field name="name">Call</field>
            <field name="employee_id" ref="hr.employee_al"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_83" model="account.analytic.line">
            <field name="name">Design</field>
            <field name="employee_id" ref="hr.employee_fme"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-7)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_84" model="account.analytic.line">
            <field name="name">Training</field>
            <field name="employee_id" ref="hr.employee_han"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">3</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-90.0</field>
        </record>

        <record id="account_analytic_line_85" model="account.analytic.line">
            <field name="name">Call</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_86" model="account.analytic.line">
            <field name="name">On Site Visit</field>
            <field name="employee_id" ref="hr.employee_jod"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_87" model="account.analytic.line">
            <field name="name">Design</field>
            <field name="employee_id" ref="hr.employee_fme"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_88" model="account.analytic.line">
            <field name="name">Presentation</field>
            <field name="employee_id" ref="hr.employee_niv"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_89" model="account.analytic.line">
            <field name="name">Design</field>
            <field name="employee_id" ref="hr.employee_niv"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_90" model="account.analytic.line">
            <field name="name">Requirements analysis</field>
            <field name="employee_id" ref="hr.employee_jod"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_91" model="account.analytic.line">
            <field name="name">Presentation</field>
            <field name="employee_id" ref="hr.employee_jve"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_92" model="account.analytic.line">
            <field name="name">Call</field>
            <field name="employee_id" ref="hr.employee_chs"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_93" model="account.analytic.line">
            <field name="name">Quality analysis</field>
            <field name="employee_id" ref="hr.employee_jve"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">3</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-90.0</field>
        </record>

        <record id="account_analytic_line_94" model="account.analytic.line">
            <field name="name">Sprint</field>
            <field name="employee_id" ref="hr.employee_jve"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_95" model="account.analytic.line">
            <field name="name">Training</field>
            <field name="employee_id" ref="hr.employee_mit"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-8)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_96" model="account.analytic.line">
            <field name="name">Presentation</field>
            <field name="employee_id" ref="hr.employee_han"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-9)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_97" model="account.analytic.line">
            <field name="name">Training</field>
            <field name="employee_id" ref="hr.employee_niv"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-9)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">3</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-90.0</field>
        </record>

        <record id="account_analytic_line_98" model="account.analytic.line">
            <field name="name">Call</field>
            <field name="employee_id" ref="hr.employee_ngh"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-9)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_99" model="account.analytic.line">
            <field name="name">Call</field>
            <field name="employee_id" ref="hr.employee_fpi"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-9)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_100" model="account.analytic.line">
            <field name="name">Quality analysis</field>
            <field name="employee_id" ref="hr.employee_jgo"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-9)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_101" model="account.analytic.line">
            <field name="name">Sprint</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-9)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_102" model="account.analytic.line">
            <field name="name">On Site Visit</field>
            <field name="employee_id" ref="hr.employee_jep"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-9)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_103" model="account.analytic.line">
            <field name="name">Sprint</field>
            <field name="employee_id" ref="hr.employee_lur"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-9)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_104" model="account.analytic.line">
            <field name="name">Sprint</field>
            <field name="employee_id" ref="hr.employee_ngh"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-9)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_105" model="account.analytic.line">
            <field name="name">Sprint</field>
            <field name="employee_id" ref="hr.employee_jgo"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-9)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_106" model="account.analytic.line">
            <field name="name">Quality analysis</field>
            <field name="employee_id" ref="hr.employee_jgo"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-9)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">3</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-90.0</field>
        </record>

        <record id="account_analytic_line_107" model="account.analytic.line">
            <field name="name">Delivery</field>
            <field name="employee_id" ref="hr.employee_ngh"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-9)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_108" model="account.analytic.line">
            <field name="name">On Site Visit</field>
            <field name="employee_id" ref="hr.employee_niv"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_109" model="account.analytic.line">
            <field name="name">Delivery</field>
            <field name="employee_id" ref="hr.employee_vad"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">3</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-90.0</field>
        </record>

        <record id="account_analytic_line_110" model="account.analytic.line">
            <field name="name">Training</field>
            <field name="employee_id" ref="hr.employee_hne"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">3</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-90.0</field>
        </record>

        <record id="account_analytic_line_111" model="account.analytic.line">
            <field name="name">On Site Visit</field>
            <field name="employee_id" ref="hr.employee_fme"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_112" model="account.analytic.line">
            <field name="name">Presentation</field>
            <field name="employee_id" ref="hr.employee_jod"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_113" model="account.analytic.line">
            <field name="name">Quality analysis</field>
            <field name="employee_id" ref="hr.employee_jog"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_114" model="account.analytic.line">
            <field name="name">On Site Visit</field>
            <field name="employee_id" ref="hr.employee_fme"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">2</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-60.0</field>
        </record>

        <record id="account_analytic_line_115" model="account.analytic.line">
            <field name="name">Quality analysis</field>
            <field name="employee_id" ref="hr.employee_mit"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_1"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_116" model="account.analytic.line">
            <field name="name">Quality analysis</field>
            <field name="employee_id" ref="hr.employee_jod"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_3"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_117" model="account.analytic.line">
            <field name="name">Design</field>
            <field name="employee_id" ref="hr.employee_hne"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_118" model="account.analytic.line">
            <field name="name">Delivery</field>
            <field name="employee_id" ref="hr.employee_fpi"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_4"/>
            <field name="amount">-30.0</field>
        </record>

        <record id="account_analytic_line_119" model="account.analytic.line">
            <field name="name">Training</field>
            <field name="employee_id" ref="hr.employee_stw"/>
            <field name="date" eval="(DateTime.now() + relativedelta(days=-10)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">1</field>
            <field name="project_id" search="[('sale_order_id', '=', ref('sale_timesheet.sale_order_1'))]" />
            <field name="task_id" ref="sale_timesheet.project_task_2"/>
            <field name="amount">-30.0</field>
        </record>

    </data>
</odoo>

```

## File: data\sale_timesheet_filters.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

	<record id="ir_filter_project_profitability_report_costs_and_revenues" model="ir.filters">
        <field name="name">Costs and Revenues</field>
        <field name="model_id">project.profitability.report</field>
        <field name="user_id" eval="False"/>
        <field name="is_default" eval="True"/>
        <field name="context">{
            'group_by': ['line_date'],
            'pivot_measures': ['amount_untaxed_to_invoice', 'amount_untaxed_invoiced', 'timesheet_cost', 'timesheet_unit_amount']
        }</field>
    </record>

</odoo>

```

## File: models\account.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.exceptions import UserError, ValidationError

from odoo import api, fields, models, _
from odoo.osv import expression


class AccountAnalyticLine(models.Model):
    _inherit = 'account.analytic.line'

    def _default_sale_line_domain(self):
        domain = super(AccountAnalyticLine, self)._default_sale_line_domain()
        return expression.OR([domain, [('qty_delivered_method', '=', 'timesheet')]])

    timesheet_invoice_type = fields.Selection([
        ('billable_time', 'Billed on Timesheets'),
        ('billable_fixed', 'Billed at a Fixed price'),
        ('non_billable', 'Non Billable Tasks'),
        ('non_billable_timesheet', 'Non Billable Timesheet'),
        ('non_billable_project', 'No task found')], string="Billable Type", compute='_compute_timesheet_invoice_type', compute_sudo=True, store=True, readonly=True)
    timesheet_invoice_id = fields.Many2one('account.move', string="Invoice", readonly=True, copy=False, help="Invoice created from the timesheet")
    non_allow_billable = fields.Boolean("Non-Billable", help="Your timesheet will not be billed.")
    so_line = fields.Many2one(compute="_compute_so_line", store=True, readonly=False)

    # TODO: [XBO] Since the task_id is not required in this model,  then it should more efficient to depends to bill_type and pricing_type of project (See in master)
    @api.depends('so_line.product_id', 'project_id', 'task_id', 'non_allow_billable', 'task_id.bill_type', 'task_id.pricing_type', 'task_id.non_allow_billable')
    def _compute_timesheet_invoice_type(self):
        non_allowed_billable = self.filtered('non_allow_billable')
        non_allowed_billable.timesheet_invoice_type = 'non_billable_timesheet'
        non_allowed_billable_task = (self - non_allowed_billable).filtered(lambda t: t.task_id.bill_type == 'customer_project' and t.task_id.pricing_type == 'employee_rate' and t.task_id.non_allow_billable)
        non_allowed_billable_task.timesheet_invoice_type = 'non_billable'

        for timesheet in self - non_allowed_billable - non_allowed_billable_task:
            if timesheet.project_id:  # AAL will be set to False
                invoice_type = 'non_billable_project' if not timesheet.task_id else 'non_billable'
                if timesheet.task_id and timesheet.so_line.product_id.type == 'service':
                    if timesheet.so_line.product_id.invoice_policy == 'delivery':
                        if timesheet.so_line.product_id.service_type == 'timesheet':
                            invoice_type = 'billable_time'
                        else:
                            invoice_type = 'billable_fixed'
                    elif timesheet.so_line.product_id.invoice_policy == 'order':
                        invoice_type = 'billable_fixed'
                timesheet.timesheet_invoice_type = invoice_type
            else:
                timesheet.timesheet_invoice_type = False

    @api.onchange('employee_id')
    def _onchange_task_id_employee_id(self):
        if self.project_id and self.task_id.allow_billable:  # timesheet only
            if self.task_id.bill_type == 'customer_task' or self.task_id.pricing_type == 'fixed_rate':
                self.so_line = self.task_id.sale_line_id
            elif self.task_id.pricing_type == 'employee_rate':
                self.so_line = self._timesheet_determine_sale_line(self.task_id, self.employee_id, self.project_id)
            else:
                self.so_line = False

    @api.depends('task_id.sale_line_id', 'project_id.sale_line_id', 'employee_id', 'project_id.allow_billable')
    def _compute_so_line(self):
        for timesheet in self._get_not_billed():  # Get only the timesheets are not yet invoiced
            timesheet.so_line = timesheet.project_id.allow_billable and timesheet._timesheet_determine_sale_line(timesheet.task_id, timesheet.employee_id, timesheet.project_id)
    
    @api.depends('timesheet_invoice_id.state')
    def _compute_partner_id(self):
        super(AccountAnalyticLine, self._get_not_billed())._compute_partner_id()

    def _get_not_billed(self):
        return self.filtered(lambda t: not t.timesheet_invoice_id or t.timesheet_invoice_id.state == 'cancel')

    def _check_timesheet_can_be_billed(self):
        return self.so_line in self.project_id.mapped('sale_line_employee_ids.sale_line_id') | self.task_id.sale_line_id | self.project_id.sale_line_id

    @api.constrains('so_line', 'project_id')
    def _check_sale_line_in_project_map(self):
        if not all(t._check_timesheet_can_be_billed() for t in self._get_not_billed().filtered(lambda t: t.project_id and t.so_line)):
            raise ValidationError(_("This timesheet line cannot be billed: there is no Sale Order Item defined on the task, nor on the project. Please define one to save your timesheet line."))

    def write(self, values):
        # prevent to update invoiced timesheets if one line is of type delivery
        self._check_can_write(values)
        result = super(AccountAnalyticLine, self).write(values)
        return result

    def _check_can_write(self, values):
        if self.sudo().filtered(lambda aal: aal.so_line.product_id.invoice_policy == "delivery") and self.filtered(lambda t: t.timesheet_invoice_id and t.timesheet_invoice_id.state != 'cancel'):
            if any(field_name in values for field_name in ['unit_amount', 'employee_id', 'project_id', 'task_id', 'so_line', 'amount', 'date']):
                raise UserError(_('You can not modify already invoiced timesheets (linked to a Sales order items invoiced on Time and material).'))

    @api.model
    def _timesheet_preprocess(self, values):
        if values.get('task_id') and not values.get('account_id'):
            task = self.env['project.task'].browse(values.get('task_id'))
            if task.analytic_account_id:
                values['account_id'] = task.analytic_account_id.id
                values['company_id'] = task.analytic_account_id.company_id.id or task.company_id.id
        values = super(AccountAnalyticLine, self)._timesheet_preprocess(values)
        return values

    @api.model
    def _timesheet_determine_sale_line(self, task, employee, project):
        """ Deduce the SO line associated to the timesheet line:
            1/ timesheet on task rate: the so line will be the one from the task
            2/ timesheet on employee rate task: find the SO line in the map of the project (even for subtask), or fallback on the SO line of the task, or fallback
                on the one on the project
        """
        if not task:
            if project.bill_type == 'customer_project' and project.pricing_type == 'employee_rate':
                map_entry = self.env['project.sale.line.employee.map'].search([('project_id', '=', project.id), ('employee_id', '=', employee.id)])
                if map_entry:
                    return map_entry.sale_line_id
            if project.sale_line_id:
                return project.sale_line_id
        if task.allow_billable and task.sale_line_id:
            if task.bill_type == 'customer_task':
                return task.sale_line_id
            if task.pricing_type == 'fixed_rate':
                return task.sale_line_id
            elif task.pricing_type == 'employee_rate' and not task.non_allow_billable:
                map_entry = project.sale_line_employee_ids.filtered(lambda map_entry: map_entry.employee_id == employee)
                if map_entry:
                    return map_entry.sale_line_id
                if task.sale_line_id or project.sale_line_id:
                    return task.sale_line_id or project.sale_line_id
        return self.env['sale.order.line']

    def _timesheet_get_portal_domain(self):
        """ Only the timesheets with a product invoiced on delivered quantity are concerned.
            since in ordered quantity, the timesheet quantity is not invoiced,
            thus there is no meaning of showing invoice with ordered quantity.
        """
        domain = super(AccountAnalyticLine, self)._timesheet_get_portal_domain()
        return expression.AND([domain, [('timesheet_invoice_type', 'in', ['billable_time', 'non_billable', 'billable_fixed'])]])

    @api.model
    def _timesheet_get_sale_domain(self, order_lines_ids, invoice_ids):
        if not invoice_ids:
            return [('so_line', 'in', order_lines_ids.ids)]

        return [
            '|',
            '&',
            ('timesheet_invoice_id', 'in', invoice_ids.ids),
            # TODO : Master: Check if non_billable should be removed ?
            ('timesheet_invoice_type', 'in', ['billable_time', 'non_billable']),
            '&',
            ('timesheet_invoice_type', '=', 'billable_fixed'),
            ('so_line', 'in', order_lines_ids.ids)
        ]

    def _get_timesheets_to_merge(self):
        res = super(AccountAnalyticLine, self)._get_timesheets_to_merge()
        return res.filtered(lambda l: not l.timesheet_invoice_id or l.timesheet_invoice_id.state != 'posted')

    def unlink(self):
        if any(line.timesheet_invoice_id and line.timesheet_invoice_id.state == 'posted' for line in self):
            raise UserError(_('You cannot remove a timesheet that has already been invoiced.'))
        return super(AccountAnalyticLine, self).unlink()

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import api, fields, models, _
from odoo.osv import expression


class AccountMove(models.Model):
    _inherit = "account.move"

    timesheet_ids = fields.One2many('account.analytic.line', 'timesheet_invoice_id', string='Timesheets', readonly=True, copy=False)
    timesheet_count = fields.Integer("Number of timesheets", compute='_compute_timesheet_count')

    @api.depends('timesheet_ids')
    def _compute_timesheet_count(self):
        timesheet_data = self.env['account.analytic.line'].read_group([('timesheet_invoice_id', 'in', self.ids)], ['timesheet_invoice_id'], ['timesheet_invoice_id'])
        mapped_data = dict([(t['timesheet_invoice_id'][0], t['timesheet_invoice_id_count']) for t in timesheet_data])
        for invoice in self:
            invoice.timesheet_count = mapped_data.get(invoice.id, 0)

    def action_view_timesheet(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'name': _('Timesheets'),
            'domain': [('project_id', '!=', False)],
            'res_model': 'account.analytic.line',
            'view_id': False,
            'view_mode': 'tree,form',
            'help': _("""
                <p class="o_view_nocontent_smiling_face">
                    Record timesheets
                </p><p>
                    You can register and track your workings hours by project every
                    day. Every time spent on a project will become a cost and can be re-invoiced to
                    customers if required.
                </p>
            """),
            'limit': 80,
            'context': {
                'default_project_id': self.id,
                'search_default_project_id': [self.id]
            }
        }

    def _link_timesheets_to_invoice(self, start_date=None, end_date=None):
        """ Search timesheets from given period and link this timesheets to the invoice

            When we create an invoice from a sale order, we need to
            link the timesheets in this sale order to the invoice.
            Then, we can know which timesheets are invoiced in the sale order.
            :param start_date: the start date of the period
            :param end_date: the end date of the period
        """
        for line in self.filtered(lambda i: i.move_type == 'out_invoice' and i.state == 'draft').invoice_line_ids:
            sale_line_delivery = line.sale_line_ids.filtered(lambda sol: sol.product_id.invoice_policy == 'delivery' and sol.product_id.service_type == 'timesheet')
            if sale_line_delivery:
                domain = line._timesheet_domain_get_invoiced_lines(sale_line_delivery)
                if start_date:
                    domain = expression.AND([domain, [('date', '>=', start_date)]])
                if end_date:
                    domain = expression.AND([domain, [('date', '<=', end_date)]])
                timesheets = self.env['account.analytic.line'].sudo().search(domain)
                timesheets.write({'timesheet_invoice_id': line.move_id.id})


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    @api.model
    def _timesheet_domain_get_invoiced_lines(self, sale_line_delivery):
        """ Get the domain for the timesheet to link to the created invoice
            :param sale_line_delivery: recordset of sale.order.line to invoice
            :return a normalized domain
        """
        return [
            ('so_line', 'in', sale_line_delivery.ids),
            ('project_id', '!=', False),
            '|', '|',
                ('timesheet_invoice_id', '=', False),
                ('timesheet_invoice_id.state', '=', 'cancel'),
                ('timesheet_invoice_id.payment_state', '=', 'reversed')
        ]

    def unlink(self):
        move_line_read_group = self.env['account.move.line'].search_read([
            ('move_id.move_type', '=', 'out_invoice'),
            ('move_id.state', '=', 'draft'),
            ('sale_line_ids.product_id.invoice_policy', '=', 'delivery'),
            ('sale_line_ids.product_id.service_type', '=', 'timesheet'),
            ('id', 'in', self.ids)],
            ['move_id', 'sale_line_ids'])

        sale_line_ids_per_move = defaultdict(lambda: self.env['sale.order.line'])
        for move_line in move_line_read_group:
            sale_line_ids_per_move[move_line['move_id'][0]] += self.env['sale.order.line'].browse(move_line['sale_line_ids'])

        timesheet_read_group = self.sudo().env['account.analytic.line'].read_group([
            ('timesheet_invoice_id.move_type', '=', 'out_invoice'),
            ('timesheet_invoice_id.state', '=', 'draft'),
            ('timesheet_invoice_id', 'in', self.move_id.ids)], 
            ['timesheet_invoice_id', 'so_line', 'ids:array_agg(id)'], 
            ['timesheet_invoice_id', 'so_line'], 
            lazy=False)

        timesheet_ids = []
        for timesheet in timesheet_read_group:
            move_id = timesheet['timesheet_invoice_id'][0]
            if timesheet['so_line'][0] in sale_line_ids_per_move[move_id].ids:
                timesheet_ids += timesheet['ids']

        self.sudo().env['account.analytic.line'].browse(timesheet_ids).write({'timesheet_invoice_id': False})
        return super().unlink()

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import threading

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    service_policy = fields.Selection([
        ('ordered_timesheet', 'Prepaid'),
        ('delivered_timesheet', 'Timesheets on tasks'),
        ('delivered_manual', 'Milestones (manually set quantities on order)')
    ], string="Service Invoicing Policy", compute='_compute_service_policy', inverse='_inverse_service_policy')
    service_type = fields.Selection(selection_add=[
        ('timesheet', 'Timesheets on project (one fare per SO/Project)'),
    ], ondelete={'timesheet': 'set default'})
    # override domain
    project_id = fields.Many2one(domain="[('company_id', '=', current_company_id), ('allow_billable', '=', True), ('bill_type', '=', 'customer_task'), ('allow_timesheets', 'in', [service_policy == 'delivered_timesheet', True])]")
    project_template_id = fields.Many2one(domain="[('company_id', '=', current_company_id), ('allow_billable', '=', True), ('bill_type', '=', 'customer_project'), ('allow_timesheets', 'in', [service_policy == 'delivered_timesheet', True])]")

    def _default_visible_expense_policy(self):
        visibility = self.user_has_groups('project.group_project_user')
        return visibility or super(ProductTemplate, self)._default_visible_expense_policy()

    def _compute_visible_expense_policy(self):
        visibility = self.user_has_groups('project.group_project_user')
        for product_template in self:
            if not product_template.visible_expense_policy:
                product_template.visible_expense_policy = visibility
        return super(ProductTemplate, self)._compute_visible_expense_policy()

    @api.depends('invoice_policy', 'service_type')
    def _compute_service_policy(self):
        for product in self:
            policy = None
            if product.invoice_policy == 'delivery':
                policy = 'delivered_manual' if product.service_type == 'manual' else 'delivered_timesheet'
            elif product.invoice_policy == 'order' and (product.service_type == 'timesheet' or product.type == 'service'):
                policy = 'ordered_timesheet'
            product.service_policy = policy

    def _inverse_service_policy(self):
        for product in self:
            policy = product.service_policy
            if not policy and not product.invoice_policy =='delivery':
                product.invoice_policy = 'order'
                product.service_type = 'manual'
            elif policy == 'ordered_timesheet':
                product.invoice_policy = 'order'
                product.service_type = 'timesheet'
            else:
                product.invoice_policy = 'delivery'
                product.service_type = 'manual' if policy == 'delivered_manual' else 'timesheet'

    @api.onchange('type')
    def _onchange_type(self):
        res = super(ProductTemplate, self)._onchange_type()
        if self.type == 'service' and not self.invoice_policy:
            self.invoice_policy = 'order'
            self.service_type = 'timesheet'
        elif self.type == 'service' and self.invoice_policy == 'order':
            self.service_policy = 'ordered_timesheet'
        elif self.type == 'consu' and not self.invoice_policy and self.service_policy == 'ordered_timesheet':
            self.invoice_policy = 'order'

        if self.type != 'service':
            self.service_tracking = 'no'
        return res

    @api.model
    def _get_onchange_service_policy_updates(self, service_tracking, service_policy, project_id, project_template_id):
        vals = {}
        if service_tracking != 'no' and service_policy == 'delivered_timesheet':
            if project_id and not project_id.allow_timesheets:
                vals['project_id'] = False
            elif project_template_id and not project_template_id.allow_timesheets:
                vals['project_template_id'] = False
        return vals

    @api.onchange('service_policy')
    def _onchange_service_policy(self):
        vals = self._get_onchange_service_policy_updates(self.service_tracking,
                                                        self.service_policy,
                                                        self.project_id,
                                                        self.project_template_id)
        if vals:
            self.update(vals)

    def unlink(self):
        time_product = self.env.ref('sale_timesheet.time_product')
        if time_product.product_tmpl_id in self:
            raise ValidationError(_('The %s product is required by the Timesheet app and cannot be archived/deleted.') % time_product.name)
        return super(ProductTemplate, self).unlink()

    def write(self, vals):
        # timesheet product can't be archived
        test_mode = getattr(threading.currentThread(), 'testing', False) or self.env.registry.in_test_mode()
        if not test_mode and 'active' in vals and not vals['active']:
            time_product = self.env.ref('sale_timesheet.time_product')
            if time_product.product_tmpl_id in self:
                raise ValidationError(_('The %s product is required by the Timesheet app and cannot be archived/deleted.') % time_product.name)
        if 'type' in vals and vals['type'] != 'service':
            vals.update({
                'service_tracking': 'no',
                'project_id': False
            })
        return super(ProductTemplate, self).write(vals)


class ProductProduct(models.Model):
    _inherit = 'product.product'

    def _is_delivered_timesheet(self):
        """ Check if the product is a delivered timesheet """
        self.ensure_one()
        return self.type == 'service' and self.service_policy == 'delivered_timesheet'

    @api.onchange('service_policy')
    def _onchange_service_policy(self):
        vals = self.product_tmpl_id._get_onchange_service_policy_updates(self.service_tracking,
                                                                        self.service_policy,
                                                                        self.project_id,
                                                                        self.project_template_id)
        if vals:
            self.update(vals)

    @api.onchange('type')
    def _onchange_type(self):
        res = super(ProductProduct, self)._onchange_type()
        if self.type != 'service':
            self.service_tracking = 'no'
        return res

    def unlink(self):
        time_product = self.env.ref('sale_timesheet.time_product')
        if time_product in self:
            raise ValidationError(_('The %s product is required by the Timesheet app and cannot be archived/deleted.') % time_product.name)
        return super(ProductProduct, self).unlink()

    def write(self, vals):
        # timesheet product can't be archived
        test_mode = getattr(threading.currentThread(), 'testing', False) or self.env.registry.in_test_mode()
        if not test_mode and 'active' in vals and not vals['active']:
            time_product = self.env.ref('sale_timesheet.time_product')
            if time_product in self:
                raise ValidationError(_('The %s product is required by the Timesheet app and cannot be archived/deleted.') % time_product.name)
        if 'type' in vals and vals['type'] != 'service':
            vals.update({
                'service_tracking': 'no',
                'project_id': False
            })
        return super(ProductProduct, self).write(vals)

```

## File: models\project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


# YTI PLEASE SPLIT ME
class Project(models.Model):
    _inherit = 'project.project'

    @api.model
    def default_get(self, fields):
        """ Pre-fill timesheet product as "Time" data product when creating new project allowing billable tasks by default. """
        result = super(Project, self).default_get(fields)
        if 'timesheet_product_id' in fields and result.get('allow_billable') and result.get('allow_timesheets') and not result.get('timesheet_product_id'):
            default_product = self.env.ref('sale_timesheet.time_product', False)
            if default_product:
                result['timesheet_product_id'] = default_product.id
        return result

    def _default_timesheet_product_id(self):
        return self.env.ref('sale_timesheet.time_product', False)

    bill_type = fields.Selection([
        ('customer_task', 'Different customers'),
        ('customer_project', 'A unique customer')
    ], string="Invoice Tasks to", default="customer_task",
        help='When billing tasks individually, a Sales Order will be created from each task. It is perfect if you would like to bill different services to different customers at different rates. \n When billing the whole project, a Sales Order will be created from the project instead. This option is better if you would like to bill all the tasks of a given project to a specific customer either at a fixed rate, or at an employee rate.')
    pricing_type = fields.Selection([
        ('fixed_rate', 'Project rate'),
        ('employee_rate', 'Employee rate')
    ], string="Pricing", default="fixed_rate",
        help='The fixed rate is perfect if you bill a service at a fixed rate per hour or day worked regardless of the employee who performed it. The employee rate is preferable if your employees deliver the same service at a different rate. For instance, junior and senior consultants would deliver the same service (= consultancy), but at a different rate because of their level of seniority.')
    sale_line_employee_ids = fields.One2many('project.sale.line.employee.map', 'project_id', "Sale line/Employee map", copy=False,
        help="Employee/Sale Order Item Mapping:\n Defines to which sales order item an employee's timesheet entry will be linked."
        "By extension, it defines the rate at which an employee's time on the project is billed.")
    allow_billable = fields.Boolean("Billable", help="Invoice your time and material from tasks.")
    display_create_order = fields.Boolean(compute='_compute_display_create_order')
    timesheet_product_id = fields.Many2one(
        'product.product', string='Timesheet Product',
        domain="""[
            ('type', '=', 'service'),
            ('invoice_policy', '=', 'delivery'),
            ('service_type', '=', 'timesheet'),
            '|', ('company_id', '=', False), ('company_id', '=', company_id)]""",
        help='Select a Service product with which you would like to bill your time spent on tasks.',
        compute="_compute_timesheet_product_id", store=True, readonly=False,
        default=_default_timesheet_product_id)
    warning_employee_rate = fields.Boolean(compute='_compute_warning_employee_rate')

    _sql_constraints = [
        ('timesheet_product_required_if_billable_and_time', """
            CHECK(
                (allow_billable = 't' AND allow_timesheets = 't' AND timesheet_product_id IS NOT NULL)
                OR (allow_billable IS NOT TRUE)
                OR (allow_timesheets IS NOT TRUE)
                OR (allow_billable IS NULL)
                OR (allow_timesheets IS NULL)
            )""", 'The timesheet product is required when the task can be billed and timesheets are allowed.'),

    ]

    @api.depends('allow_billable', 'sale_order_id', 'partner_id', 'bill_type')
    def _compute_display_create_order(self):
        for project in self:
            show = True
            if not project.partner_id or project.bill_type != 'customer_project' or not project.allow_billable or project.sale_order_id:
                show = False
            project.display_create_order = show

    @api.depends('allow_timesheets', 'allow_billable')
    def _compute_timesheet_product_id(self):
        default_product = self.env.ref('sale_timesheet.time_product', False)
        for project in self:
            if not project.allow_timesheets or not project.allow_billable:
                project.timesheet_product_id = False
            elif not project.timesheet_product_id:
                project.timesheet_product_id = default_product

    @api.depends('pricing_type', 'allow_timesheets', 'allow_billable', 'sale_line_employee_ids', 'sale_line_employee_ids.employee_id', 'bill_type')
    def _compute_warning_employee_rate(self):
        projects = self.filtered(lambda p: p.allow_billable and p.allow_timesheets and p.bill_type == 'customer_project' and p.pricing_type == 'employee_rate')
        tasks = projects.task_ids.filtered(lambda t: not t.non_allow_billable)
        employees = self.env['account.analytic.line'].read_group([('task_id', 'in', tasks.ids), ('non_allow_billable', '=', False)], ['employee_id', 'project_id'], ['employee_id', 'project_id'], lazy=False)
        dict_project_employee = defaultdict(list)
        for line in employees:
            dict_project_employee[line['project_id'][0]] += [line['employee_id'][0]] if line['employee_id'] else []
        for project in projects:
            project.warning_employee_rate = any(x not in project.sale_line_employee_ids.employee_id.ids for x in dict_project_employee[project.id])

        (self - projects).warning_employee_rate = False

    @api.constrains('sale_line_id', 'pricing_type')
    def _check_sale_line_type(self):
        for project in self:
            if project.pricing_type == 'fixed_rate':
                if project.sale_line_id and not project.sale_line_id.is_service:
                    raise ValidationError(_("A billable project should be linked to a Sales Order Item having a Service product."))
                if project.sale_line_id and project.sale_line_id.is_expense:
                    raise ValidationError(_("A billable project should be linked to a Sales Order Item that does not come from an expense or a vendor bill."))

    @api.onchange('allow_billable')
    def _onchange_allow_billable(self):
        if self.task_ids._get_timesheet() and self.allow_timesheets and not self.allow_billable:
            message = _("All timesheet hours that are not yet invoiced will be removed from Sales Order on save. Discard to avoid the change.")
            return {'warning': {
                'title': _("Warning"),
                'message': message
            }}

    def write(self, values):
        res = super(Project, self).write(values)
        if 'allow_billable' in values and not values.get('allow_billable'):
            self.task_ids._get_timesheet().write({
                'so_line': False,
            })
        return res

    def _get_not_billed_timesheets(self):
        return self.sudo(False).mapped('timesheet_ids').filtered(
            lambda t: not t.timesheet_invoice_id or t.timesheet_invoice_id.state == 'cancel')

    def _update_timesheets_sale_line_id(self):
        for project in self.filtered(lambda p: p.allow_billable and p.allow_timesheets):
            timesheet_ids = project._get_not_billed_timesheets()
            if not timesheet_ids:
                continue
            for employee_id in project.sale_line_employee_ids.filtered(lambda l: l.project_id == project).employee_id:
                sale_line_id = project.sale_line_employee_ids.filtered(lambda l: l.project_id == project and l.employee_id == employee_id).sale_line_id
                timesheet_ids.filtered(lambda t: t.employee_id == employee_id).sudo().so_line = sale_line_id

    def action_view_timesheet(self):
        self.ensure_one()
        if self.allow_timesheets:
            return self.action_view_timesheet_plan()
        return {
            'type': 'ir.actions.act_window',
            'name': _('Timesheets of %s', self.name),
            'domain': [('project_id', '!=', False)],
            'res_model': 'account.analytic.line',
            'view_id': False,
            'view_mode': 'tree,form',
            'help': _("""
                <p class="o_view_nocontent_smiling_face">
                    Record timesheets
                </p><p>
                    You can register and track your workings hours by project every
                    day. Every time spent on a project will become a cost and can be re-invoiced to
                    customers if required.
                </p>
            """),
            'limit': 80,
            'context': {
                'default_project_id': self.id,
                'search_default_project_id': [self.id]
            }
        }

    def action_view_timesheet_plan(self):
        action = self.env["ir.actions.actions"]._for_xml_id("sale_timesheet.project_timesheet_action_client_timesheet_plan")
        action['params'] = {
            'project_ids': self.ids,
        }
        action['context'] = {
            'active_id': self.id,
            'active_ids': self.ids,
            'search_default_name': self.name,
        }
        return action

    def action_make_billable(self):
        return {
            "name": _("Create Sales Order"),
            "type": 'ir.actions.act_window',
            "res_model": 'project.create.sale.order',
            "views": [[False, "form"]],
            "target": 'new',
            "context": {
                'active_id': self.id,
                'active_model': 'project.project',
                'default_product_id': self.timesheet_product_id.id,
            },
        }


class ProjectTask(models.Model):
    _inherit = "project.task"

    @api.model
    def default_get(self, fields):
        result = super(ProjectTask, self).default_get(fields)

        if not result.get('timesheet_product_id', False) and 'project_id' in result:
            project = self.env['project.project'].browse(result['project_id'])
            if project.bill_type != 'customer_project' or project.pricing_type != 'employee_rate':
                result['timesheet_product_id'] = project.timesheet_product_id.id
        return result

    # override sale_order_id and make it computed stored field instead of regular field.
    sale_order_id = fields.Many2one(compute='_compute_sale_order_id', store=True, readonly=False,
    domain="['|', '|', ('partner_id', '=', partner_id), ('partner_id', 'child_of', commercial_partner_id), ('partner_id', 'parent_of', partner_id)]")
    # content in related parameter of the field definition is removed to manually define the compute and the search method.
    project_sale_order_id = fields.Many2one(compute='_compute_project_sale_order_id', search='_search_project_sale_order_id', related=None)
    analytic_account_id = fields.Many2one('account.analytic.account', related='sale_order_id.analytic_account_id')
    bill_type = fields.Selection(related="project_id.bill_type")
    pricing_type = fields.Selection(related="project_id.pricing_type")
    is_project_map_empty = fields.Boolean("Is Project map empty", compute='_compute_is_project_map_empty')
    has_multi_sol = fields.Boolean(compute='_compute_has_multi_sol', compute_sudo=True)
    allow_billable = fields.Boolean(related="project_id.allow_billable")
    display_create_order = fields.Boolean(compute='_compute_display_create_order')
    timesheet_product_id = fields.Many2one(
        'product.product', string='Service',
        domain="""[
            ('type', '=', 'service'),
            ('invoice_policy', '=', 'delivery'),
            ('service_type', '=', 'timesheet'),
            '|', ('company_id', '=', False), ('company_id', '=', company_id)]""",
        help='Select a Service product with which you would like to bill your time spent on this task.')

    # TODO: [XBO] remove me in master
    non_allow_billable = fields.Boolean("Non-Billable", help="Your timesheets linked to this task will not be billed.")
    remaining_hours_so = fields.Float('Remaining Hours on SO', compute='_compute_remaining_hours_so', compute_sudo=True)
    remaining_hours_available = fields.Boolean(related="sale_line_id.remaining_hours_available")

    @api.depends('sale_line_id', 'timesheet_ids', 'timesheet_ids.unit_amount')
    def _compute_remaining_hours_so(self):
        # TODO This is not yet perfectly working as timesheet.so_line stick to its old value although changed
        #      in the task From View.
        timesheets = self.timesheet_ids.filtered(lambda t: t.task_id.sale_line_id in (t.so_line, t._origin.so_line) and t.so_line.remaining_hours_available)

        mapped_remaining_hours = {task._origin.id: task.sale_line_id and task.sale_line_id.remaining_hours or 0.0 for task in self}
        uom_hour = self.env.ref('uom.product_uom_hour')
        for timesheet in timesheets:
            delta = 0
            if timesheet._origin.so_line == timesheet.task_id.sale_line_id:
                delta += timesheet._origin.unit_amount
            if timesheet.so_line == timesheet.task_id.sale_line_id:
                delta -= timesheet.unit_amount
            if delta:
                mapped_remaining_hours[timesheet.task_id._origin.id] += timesheet.product_uom_id._compute_quantity(delta, uom_hour)

        for task in self:
            task.remaining_hours_so = mapped_remaining_hours[task._origin.id]

    @api.depends(
        'allow_billable', 'allow_timesheets', 'sale_order_id')
    def _compute_display_create_order(self):
        for task in self:
            show = True
            if not task.allow_billable or not task.allow_timesheets or \
                (task.bill_type != 'customer_task' and not task.timesheet_product_id) or (not task.partner_id and task.bill_type != 'customer_task') or \
                task.sale_order_id or (task.bill_type != 'customer_task' and task.pricing_type != 'employee_rate'):
                show = False
            task.display_create_order = show

    @api.onchange('sale_line_id')
    def _onchange_sale_line_id(self):
        # TODO: remove me in master
        return

    @api.onchange('project_id')
    def _onchange_project_id(self):
        # TODO: remove me in master
        return

    @api.depends('analytic_account_id.active')
    def _compute_analytic_account_active(self):
        super()._compute_analytic_account_active()
        for task in self:
            task.analytic_account_active = task.analytic_account_active or task.analytic_account_id.active

    @api.depends('project_id.bill_type', 'project_id.sale_order_id')
    def _compute_project_sale_order_id(self):
        for task in self:
            if task.bill_type != 'customer_task':
                task.project_sale_order_id = task.project_id.sale_order_id
            else:
                task.project_sale_order_id = False

    @api.depends('sale_line_id', 'project_id', 'allow_billable', 'non_allow_billable')
    def _compute_sale_order_id(self):
        for task in self:
            if not task.allow_billable or task.non_allow_billable:
                task.sale_order_id = False
            elif task.allow_billable:
                if task.sale_line_id:
                    task.sale_order_id = task.sale_line_id.sudo().order_id
                elif task.project_id.sale_order_id:
                    task.sale_order_id = task.project_id.sale_order_id
                if task.sale_order_id and not task.partner_id:
                    task.partner_id = task.sale_order_id.partner_id

    @api.depends('commercial_partner_id', 'sale_line_id.order_partner_id.commercial_partner_id', 'parent_id.sale_line_id', 'project_id.sale_line_id', 'allow_billable')
    def _compute_sale_line(self):
        billable_tasks = self.filtered('allow_billable')
        super(ProjectTask, billable_tasks)._compute_sale_line()
        for task in billable_tasks.filtered(lambda t: not t.sale_line_id):
            task.sale_line_id = task._get_last_sol_of_customer()

    @api.depends('project_id.sale_line_employee_ids')
    def _compute_is_project_map_empty(self):
        for task in self:
            task.is_project_map_empty = not bool(task.sudo().project_id.sale_line_employee_ids)

    @api.depends('timesheet_ids')
    def _compute_has_multi_sol(self):
        for task in self:
            task.has_multi_sol = task.timesheet_ids and task.timesheet_ids.so_line != task.sale_line_id

    @api.onchange('project_id')
    def _onchange_project(self):
        if self.project_id and self.project_id.bill_type == 'customer_project':
            if not self.partner_id:
                self.partner_id = self.project_id.partner_id
            if not self.sale_line_id:
                self.sale_line_id = self.project_id.sale_line_id

    def _search_project_sale_order_id(self, operator, value):
        return [('bill_type', '!=', 'customer_task'), ('project_id.sale_order_id', operator, value)]

    def write(self, values):
        res = super(ProjectTask, self).write(values)
        # Done after super to avoid constraints on field recomputation
        if values.get('project_id'):
            project_dest = self.env['project.project'].browse(values['project_id'])
            if project_dest.bill_type == 'customer_project' and project_dest.pricing_type == 'employee_rate':
                self.write({'sale_line_id': False})
        if 'non_allow_billable' in values and self.filtered('allow_timesheets').sudo().timesheet_ids:
            timesheet_ids = self.filtered('allow_timesheets').timesheet_ids.filtered(
                lambda t: (not t.timesheet_invoice_id or t.timesheet_invoice_id.state == 'cancel')
            )
            if values['non_allow_billable']:
                timesheet_ids.write({'so_line': False})
                self.sale_line_id = False
            else:
                # We write project on timesheet lines to call _timesheet_preprocess. This function will set correct the SOL
                for project in timesheet_ids.project_id:
                    current_timesheet_ids = timesheet_ids.filtered(lambda t: t.project_id == project)
                    current_timesheet_ids.task_id.update({'sale_line_id': project.sale_line_id.id})
                    for employee in current_timesheet_ids.employee_id:
                        current_timesheet_ids.filtered(lambda t: t.employee_id == employee).write({'project_id': project.id})

        return res

    def _get_last_sol_of_customer(self):
        # Get the last SOL made for the customer in the current task where we need to compute
        self.ensure_one()
        if not self.commercial_partner_id or not self.allow_billable:
            return False
        domain = [('company_id', '=', self.company_id.id), ('is_service', '=', True), ('order_partner_id', 'child_of', self.commercial_partner_id.id), ('is_expense', '=', False), ('state', 'in', ['sale', 'done'])]
        if self.project_id.bill_type == 'customer_project' and self.project_sale_order_id:
            domain.append(('order_id', '=?', self.project_sale_order_id.id))
        sale_lines = self.env['sale.order.line'].search(domain)
        for line in sale_lines:
            if line.remaining_hours_available and line.remaining_hours > 0:
                return line
        return False

    def action_make_billable(self):
        return {
            "name": _("Create Sales Order"),
            "type": 'ir.actions.act_window',
            "res_model": 'project.task.create.sale.order',
            "views": [[False, "form"]],
            "target": 'new',
            "context": {
                'active_id': self.id,
                'active_model': 'project.task',
                'form_view_initial_mode': 'edit',
                'default_product_id': self.timesheet_product_id.id or self.project_id.timesheet_product_id.id,
            },
        }

    def _get_timesheet(self):
        # return not invoiced timesheet and timesheet without so_line or so_line linked to task
        timesheet_ids = super(ProjectTask, self)._get_timesheet()
        return timesheet_ids.filtered(lambda t: (not t.timesheet_invoice_id or t.timesheet_invoice_id.state == 'cancel') and (not t.so_line or t.so_line == t.task_id._origin.sale_line_id))

    def _get_action_view_so_ids(self):
        return list(set((self.sale_order_id + self.timesheet_ids.so_line.order_id).ids))

class ProjectTaskRecurrence(models.Model):
    _inherit = 'project.task.recurrence'

    @api.model
    def _get_recurring_fields(self):
        return ['analytic_account_id'] + super(ProjectTaskRecurrence, self)._get_recurring_fields()

```

## File: models\project_overview.py

```python
# -*- coding: utf-8 -*-
import babel.dates
from dateutil.relativedelta import relativedelta
import itertools
import json

from odoo import fields, _, models
from odoo.osv import expression
from odoo.tools import float_round
from odoo.tools.misc import get_lang

from odoo.addons.web.controllers.main import clean_action
from datetime import date

DEFAULT_MONTH_RANGE = 3


class Project(models.Model):
    _inherit = 'project.project'


    def _qweb_prepare_qcontext(self, view_id, domain):
        values = super()._qweb_prepare_qcontext(view_id, domain)

        projects = self.search(domain)
        values.update(projects._plan_prepare_values())
        values['actions'] = projects._plan_prepare_actions(values)

        return values

    def _plan_get_employee_ids(self):
        aal_employee_ids = self.env['account.analytic.line'].read_group([('project_id', 'in', self.ids), ('employee_id', '!=', False)], ['employee_id'], ['employee_id'])
        employee_ids = list(map(lambda x: x['employee_id'][0], aal_employee_ids))
        return employee_ids

    def _plan_prepare_values(self):
        currency = self.env.company.currency_id
        uom_hour = self.env.ref('uom.product_uom_hour')
        company_uom = self.env.company.timesheet_encode_uom_id
        is_uom_day = company_uom == self.env.ref('uom.product_uom_day')
        hour_rounding = uom_hour.rounding
        billable_types = ['non_billable', 'non_billable_project', 'billable_time', 'non_billable_timesheet', 'billable_fixed']

        values = {
            'projects': self,
            'currency': currency,
            'timesheet_domain': [('project_id', 'in', self.ids)],
            'profitability_domain': [('project_id', 'in', self.ids)],
            'stat_buttons': self._plan_get_stat_button(),
            'is_uom_day': is_uom_day,
        }

        #
        # Hours, Rates and Profitability
        #
        dashboard_values = {
            'time': dict.fromkeys(billable_types + ['total'], 0.0),
            'rates': dict.fromkeys(billable_types + ['total'], 0.0),
            'profit': {
                'invoiced': 0.0,
                'to_invoice': 0.0,
                'cost': 0.0,
                'total': 0.0,
            }
        }

        # hours from non-invoiced timesheets that are linked to canceled so
        canceled_hours_domain = [('project_id', 'in', self.ids), ('timesheet_invoice_type', '!=', False), ('so_line.state', '=', 'cancel')]
        total_canceled_hours = sum(self.env['account.analytic.line'].search(canceled_hours_domain).mapped('unit_amount'))
        canceled_hours = float_round(total_canceled_hours, precision_rounding=hour_rounding)
        if is_uom_day:
            # convert time from hours to days
            canceled_hours = round(uom_hour._compute_quantity(canceled_hours, company_uom, raise_if_failure=False), 2)
        dashboard_values['time']['canceled'] = canceled_hours
        dashboard_values['time']['total'] += canceled_hours

        # hours (from timesheet) and rates (by billable type)
        dashboard_domain = [('project_id', 'in', self.ids), ('timesheet_invoice_type', '!=', False), '|', ('so_line', '=', False), ('so_line.state', '!=', 'cancel')]  # force billable type
        dashboard_data = self.env['account.analytic.line'].read_group(dashboard_domain, ['unit_amount', 'timesheet_invoice_type'], ['timesheet_invoice_type'])
        dashboard_total_hours = sum([data['unit_amount'] for data in dashboard_data]) + total_canceled_hours
        for data in dashboard_data:
            billable_type = data['timesheet_invoice_type']
            amount = float_round(data.get('unit_amount'), precision_rounding=hour_rounding)
            if is_uom_day:
                # convert time from hours to days
                amount = round(uom_hour._compute_quantity(amount, company_uom, raise_if_failure=False), 2)
            dashboard_values['time'][billable_type] = amount
            dashboard_values['time']['total'] += amount
            # rates
            rate = round(data.get('unit_amount') / dashboard_total_hours * 100, 2) if dashboard_total_hours else 0.0
            dashboard_values['rates'][billable_type] = rate
            dashboard_values['rates']['total'] += rate
        dashboard_values['time']['total'] = round(dashboard_values['time']['total'], 2)

        # rates from non-invoiced timesheets that are linked to canceled so
        dashboard_values['rates']['canceled'] = float_round(100 * total_canceled_hours / (dashboard_total_hours or 1), precision_rounding=hour_rounding)
        
        # profitability, using profitability SQL report
        field_map = {
            'amount_untaxed_invoiced': 'invoiced',
            'amount_untaxed_to_invoice': 'to_invoice',
            'timesheet_cost': 'cost',
            'expense_cost': 'expense_cost',
            'expense_amount_untaxed_invoiced': 'expense_amount_untaxed_invoiced',
            'expense_amount_untaxed_to_invoice': 'expense_amount_untaxed_to_invoice',
            'other_revenues': 'other_revenues'
        }
        profit = dict.fromkeys(list(field_map.values()) + ['other_revenues', 'total'], 0.0)
        profitability_raw_data = self.env['project.profitability.report'].read_group([('project_id', 'in', self.ids)], ['project_id'] + list(field_map), ['project_id'])   
        for data in profitability_raw_data:
            company_id = self.env['project.project'].browse(data.get('project_id')[0]).company_id
            from_currency = company_id.currency_id
            for field in field_map:
                value = data.get(field, 0.0)
                if from_currency != currency:
                    value = from_currency._convert(value, currency, company_id, date.today())
                profit[field_map[field]] += value               
        profit['total'] = sum([profit[item] for item in profit.keys()])
        dashboard_values['profit'] = profit

        values['dashboard'] = dashboard_values

        #
        # Time Repartition (per employee per billable types)
        #
        employee_ids = self._plan_get_employee_ids()
        employee_ids = list(set(employee_ids))
        # Retrieve the employees for which the current user can see theirs timesheets
        employee_domain = expression.AND([[('company_id', 'in', self.env.companies.ids)], self.env['account.analytic.line']._domain_employee_id()])
        employees = self.env['hr.employee'].sudo().browse(employee_ids).filtered_domain(employee_domain)
        repartition_domain = [('project_id', 'in', self.ids), ('employee_id', '!=', False), ('timesheet_invoice_type', '!=', False)]  # force billable type
        # repartition data, without timesheet on cancelled so
        repartition_data = self.env['account.analytic.line'].read_group(repartition_domain + ['|', ('so_line', '=', False), ('so_line.state', '!=', 'cancel')], ['employee_id', 'timesheet_invoice_type', 'unit_amount'], ['employee_id', 'timesheet_invoice_type'], lazy=False)
        # read timesheet on cancelled so
        cancelled_so_timesheet = self.env['account.analytic.line'].read_group(repartition_domain + [('so_line.state', '=', 'cancel')], ['employee_id', 'unit_amount'], ['employee_id'], lazy=False)
        repartition_data += [{**canceled, 'timesheet_invoice_type': 'canceled'} for canceled in cancelled_so_timesheet]

        # set repartition per type per employee
        repartition_employee = {}
        for employee in employees:
            repartition_employee[employee.id] = dict(
                employee_id=employee.id,
                employee_name=employee.name,
                non_billable_project=0.0,
                non_billable=0.0,
                billable_time=0.0,
                non_billable_timesheet=0.0,
                billable_fixed=0.0,
                canceled=0.0,
                total=0.0,
            )
        for data in repartition_data:
            employee_id = data['employee_id'][0]
            repartition_employee.setdefault(employee_id, dict(
                employee_id=data['employee_id'][0],
                employee_name=data['employee_id'][1],
                non_billable_project=0.0,
                non_billable=0.0,
                billable_time=0.0,
                non_billable_timesheet=0.0,
                billable_fixed=0.0,
                canceled=0.0,
                total=0.0,
            ))[data['timesheet_invoice_type']] = float_round(data.get('unit_amount', 0.0), precision_rounding=hour_rounding)
            repartition_employee[employee_id]['__domain_' + data['timesheet_invoice_type']] = data['__domain']
        # compute total
        for employee_id, vals in repartition_employee.items():
            repartition_employee[employee_id]['total'] = sum([vals[inv_type] for inv_type in [*billable_types, 'canceled']])
            if is_uom_day:
                # convert all times from hours to days
                for time_type in ['non_billable_project', 'non_billable', 'billable_time', 'non_billable_timesheet', 'billable_fixed', 'canceled', 'total']:
                    if repartition_employee[employee_id][time_type]:
                        repartition_employee[employee_id][time_type] = round(uom_hour._compute_quantity(repartition_employee[employee_id][time_type], company_uom, raise_if_failure=False), 2)
        hours_per_employee = [repartition_employee[employee_id]['total'] for employee_id in repartition_employee]
        values['repartition_employee_max'] = (max(hours_per_employee) if hours_per_employee else 1) or 1
        values['repartition_employee'] = repartition_employee

        #
        # Table grouped by SO / SOL / Employees
        #
        timesheet_forecast_table_rows = self._table_get_line_values(employees)
        if timesheet_forecast_table_rows:
            values['timesheet_forecast_table'] = timesheet_forecast_table_rows
        return values

    def _table_get_line_values(self, employees=None):
        """ return the header and the rows informations of the table """
        if not self:
            return False

        uom_hour = self.env.ref('uom.product_uom_hour')
        company_uom = self.env.company.timesheet_encode_uom_id
        is_uom_day = company_uom and company_uom == self.env.ref('uom.product_uom_day')

        # build SQL query and fetch raw data
        query, query_params = self._table_rows_sql_query()
        self.env.cr.execute(query, query_params)
        raw_data = self.env.cr.dictfetchall()
        rows_employee = self._table_rows_get_employee_lines(raw_data)
        default_row_vals = self._table_row_default()

        empty_line_ids, empty_order_ids = self._table_get_empty_so_lines()

        # extract row labels
        sale_line_ids = set()
        sale_order_ids = set()
        for key_tuple, row in rows_employee.items():
            if row[0]['sale_line_id']:
                sale_line_ids.add(row[0]['sale_line_id'])
            if row[0]['sale_order_id']:
                sale_order_ids.add(row[0]['sale_order_id'])

        sale_orders = self.env['sale.order'].sudo().browse(sale_order_ids | empty_order_ids)
        sale_order_lines = self.env['sale.order.line'].sudo().browse(sale_line_ids | empty_line_ids)
        map_so_names = {so.id: so.name for so in sale_orders}
        map_so_cancel = {so.id: so.state == 'cancel' for so in sale_orders}
        map_sol = {sol.id: sol for sol in sale_order_lines}
        map_sol_names = {sol.id: sol.name.split('\n')[0] if sol.name else _('No Sales Order Line') for sol in sale_order_lines}
        map_sol_so = {sol.id: sol.order_id.id for sol in sale_order_lines}

        rows_sale_line = {}  # (so, sol) -> [INFO, before, M1, M2, M3, Done, M3, M4, M5, After, Forecasted]
        for sale_line_id in empty_line_ids:  # add service SO line having no timesheet
            sale_line_row_key = (map_sol_so.get(sale_line_id), sale_line_id)
            sale_line = map_sol.get(sale_line_id)
            is_milestone = sale_line.product_id.invoice_policy == 'delivery' and sale_line.product_id.service_type == 'manual' if sale_line else False
            rows_sale_line[sale_line_row_key] = [{'label': map_sol_names.get(sale_line_id, _('No Sales Order Line')), 'res_id': sale_line_id, 'res_model': 'sale.order.line', 'type': 'sale_order_line', 'is_milestone': is_milestone}] + default_row_vals[:]
            if not is_milestone:
                rows_sale_line[sale_line_row_key][-2] = sale_line.product_uom._compute_quantity(sale_line.product_uom_qty, uom_hour, raise_if_failure=False) if sale_line else 0.0

        rows_sale_line_all_data = {}
        if not employees:
            employees = self.env['hr.employee'].sudo().search(self.env['account.analytic.line']._domain_employee_id())
        for row_key, row_employee in rows_employee.items():
            sale_order_id, sale_line_id, employee_id = row_key
            # sale line row
            sale_line_row_key = (sale_order_id, sale_line_id)
            if sale_line_row_key not in rows_sale_line:
                sale_line = map_sol.get(sale_line_id, self.env['sale.order.line'])
                is_milestone = sale_line.product_id.invoice_policy == 'delivery' and sale_line.product_id.service_type == 'manual' if sale_line else False
                rows_sale_line[sale_line_row_key] = [{'label': map_sol_names.get(sale_line.id) if sale_line else _('No Sales Order Line'), 'res_id': sale_line_id, 'res_model': 'sale.order.line', 'type': 'sale_order_line', 'is_milestone': is_milestone}] + default_row_vals[:]  # INFO, before, M1, M2, M3, Done, M3, M4, M5, After, Forecasted
                if not is_milestone:
                    rows_sale_line[sale_line_row_key][-2] = sale_line.product_uom._compute_quantity(sale_line.product_uom_qty, uom_hour, raise_if_failure=False) if sale_line else 0.0

            if sale_line_row_key not in rows_sale_line_all_data:
                rows_sale_line_all_data[sale_line_row_key] = [0] * len(row_employee)
            for index in range(1, len(row_employee)):
                if employee_id in employees.ids:
                    rows_sale_line[sale_line_row_key][index] += row_employee[index]
                rows_sale_line_all_data[sale_line_row_key][index] += row_employee[index]
            if not rows_sale_line[sale_line_row_key][0].get('is_milestone'):
                rows_sale_line[sale_line_row_key][-1] = rows_sale_line[sale_line_row_key][-2] - rows_sale_line_all_data[sale_line_row_key][5]
            else:
                rows_sale_line[sale_line_row_key][-1] = 0

        rows_sale_order = {}  # so -> [INFO, before, M1, M2, M3, Done, M3, M4, M5, After, Forecasted]
        for row_key, row_sale_line in rows_sale_line.items():
            sale_order_id = row_key[0]
            # sale order row
            if sale_order_id not in rows_sale_order:
                rows_sale_order[sale_order_id] = [{'label': map_so_names.get(sale_order_id, _('No Sales Order')), 'canceled': map_so_cancel.get(sale_order_id, False), 'res_id': sale_order_id, 'res_model': 'sale.order', 'type': 'sale_order'}] + default_row_vals[:]  # INFO, before, M1, M2, M3, Done, M3, M4, M5, After, Forecasted

            for index in range(1, len(row_sale_line)):
                rows_sale_order[sale_order_id][index] += row_sale_line[index]

        # group rows SO, SOL and their related employee rows.
        timesheet_forecast_table_rows = []
        for sale_order_id, sale_order_row in rows_sale_order.items():
            timesheet_forecast_table_rows.append(sale_order_row)
            for sale_line_row_key, sale_line_row in rows_sale_line.items():
                if sale_order_id == sale_line_row_key[0]:
                    sale_order_row[0]['has_children'] = True
                    timesheet_forecast_table_rows.append(sale_line_row)
                    for employee_row_key, employee_row in rows_employee.items():
                        if sale_order_id == employee_row_key[0] and sale_line_row_key[1] == employee_row_key[1] and employee_row_key[2] in employees.ids:
                            sale_line_row[0]['has_children'] = True
                            timesheet_forecast_table_rows.append(employee_row)

        if is_uom_day:
            # convert all values from hours to days
            for row in timesheet_forecast_table_rows:
                for index in range(1, len(row)):
                    row[index] = round(uom_hour._compute_quantity(row[index], company_uom, raise_if_failure=False), 2)
        # complete table data
        return {
            'header': self._table_header(),
            'rows': timesheet_forecast_table_rows
        }
    def _table_header(self):
        initial_date = fields.Date.from_string(fields.Date.today())
        ts_months = sorted([fields.Date.to_string(initial_date - relativedelta(months=i, day=1)) for i in range(0, DEFAULT_MONTH_RANGE)])  # M1, M2, M3

        def _to_short_month_name(date):
            month_index = fields.Date.from_string(date).month
            return babel.dates.get_month_names('abbreviated', locale=get_lang(self.env).code)[month_index]

        header_names = [_('Sales Order'), _('Before')] + [_to_short_month_name(date) for date in ts_months] + [_('Total'), _('Sold'), _('Remaining')]

        result = []
        for name in header_names:
            result.append({
                'label': name,
                'tooltip': '',
            })
        # add tooltip for reminaing
        result[-1]['tooltip'] = _('What is still to deliver based on sold hours and hours already done. Equals to sold hours - done hours.')
        return result

    def _table_row_default(self):
        lenght = len(self._table_header())
        return [0.0] * (lenght - 1)  # before, M1, M2, M3, Done, Sold, Remaining

    def _table_rows_sql_query(self):
        initial_date = fields.Date.from_string(fields.Date.today())
        ts_months = sorted([fields.Date.to_string(initial_date - relativedelta(months=i, day=1)) for i in range(0, DEFAULT_MONTH_RANGE)])  # M1, M2, M3
        # build query
        query = """
            SELECT
                'timesheet' AS type,
                date_trunc('month', date)::date AS month_date,
                E.id AS employee_id,
                S.order_id AS sale_order_id,
                A.so_line AS sale_line_id,
                SUM(A.unit_amount) AS number_hours
            FROM account_analytic_line A
                JOIN hr_employee E ON E.id = A.employee_id
                LEFT JOIN sale_order_line S ON S.id = A.so_line
            WHERE A.project_id IS NOT NULL
                AND A.project_id IN %s
                AND A.date < %s
            GROUP BY date_trunc('month', date)::date, S.order_id, A.so_line, E.id
        """

        last_ts_month = fields.Date.to_string(fields.Date.from_string(ts_months[-1]) + relativedelta(months=1))
        query_params = (tuple(self.ids), last_ts_month)
        return query, query_params

    def _table_rows_get_employee_lines(self, data_from_db):
        initial_date = fields.Date.today()
        ts_months = sorted([initial_date - relativedelta(months=i, day=1) for i in range(0, DEFAULT_MONTH_RANGE)])  # M1, M2, M3
        default_row_vals = self._table_row_default()

        # extract employee names
        employee_ids = set()
        for data in data_from_db:
            employee_ids.add(data['employee_id'])
        map_empl_names = {empl.id: empl.name for empl in self.env['hr.employee'].sudo().browse(employee_ids)}

        # extract rows data for employee, sol and so rows
        rows_employee = {}  # (so, sol, employee) -> [INFO, before, M1, M2, M3, Done, M3, M4, M5, After, Forecasted]
        for data in data_from_db:
            sale_line_id = data['sale_line_id']
            sale_order_id = data['sale_order_id']
            # employee row
            row_key = (data['sale_order_id'], sale_line_id, data['employee_id'])
            if row_key not in rows_employee:
                meta_vals = {
                    'label': map_empl_names.get(row_key[2]),
                    'sale_line_id': sale_line_id,
                    'sale_order_id': sale_order_id,
                    'res_id': row_key[2],
                    'res_model': 'hr.employee',
                    'type': 'hr_employee'
                }
                rows_employee[row_key] = [meta_vals] + default_row_vals[:]  # INFO, before, M1, M2, M3, Done, M3, M4, M5, After, Forecasted

            index = False
            if data['type'] == 'timesheet':
                if data['month_date'] in ts_months:
                    index = ts_months.index(data['month_date']) + 2
                elif data['month_date'] < ts_months[0]:
                    index = 1
                rows_employee[row_key][index] += data['number_hours']
                rows_employee[row_key][5] += data['number_hours']
        return rows_employee

    def _table_get_empty_so_lines(self):
        """ get the Sale Order Lines having no timesheet but having generated a task or a project """
        so_lines = self.sudo().mapped('sale_line_id.order_id.order_line').filtered(lambda sol: sol.is_service and not sol.is_expense and not sol.is_downpayment)
        # include the service SO line of SO sharing the same project
        sale_order = self.env['sale.order'].search([('project_id', 'in', self.ids)])
        return set(so_lines.ids) | set(sale_order.mapped('order_line').filtered(lambda sol: sol.is_service and not sol.is_expense).ids), set(so_lines.mapped('order_id').ids) | set(sale_order.ids)

    # --------------------------------------------------
    # Actions: Stat buttons, ...
    # --------------------------------------------------

    def _plan_prepare_actions(self, values):
        actions = []
        if len(self) == 1:
            task_order_line_ids = []
            # retrieve all the sale order line that we will need later below
            if self.env.user.has_group('sales_team.group_sale_salesman') or self.env.user.has_group('sales_team.group_sale_salesman_all_leads'):
                task_order_line_ids = self.env['project.task'].read_group([('project_id', '=', self.id), ('sale_line_id', '!=', False)], ['sale_line_id'], ['sale_line_id'])
                task_order_line_ids = [ol['sale_line_id'][0] for ol in task_order_line_ids]

            if self.env.user.has_group('sales_team.group_sale_salesman'):
                if self.bill_type == 'customer_project' and self.allow_billable and not self.sale_order_id:
                    actions.append({
                        'label': _("Create a Sales Order"),
                        'type': 'action',
                        'action_id': 'sale_timesheet.project_project_action_multi_create_sale_order',
                        'context': json.dumps({'active_id': self.id, 'active_model': 'project.project'}),
                    })
            if self.env.user.has_group('sales_team.group_sale_salesman_all_leads'):
                to_invoice_amount = values['dashboard']['profit'].get('to_invoice', False)  # plan project only takes services SO line with timesheet into account

                sale_order_ids = self.env['sale.order.line'].read_group([('id', 'in', task_order_line_ids)], ['order_id'], ['order_id'])
                sale_order_ids = [s['order_id'][0] for s in sale_order_ids]
                sale_order_ids = self.env['sale.order'].search_read([('id', 'in', sale_order_ids), ('invoice_status', '=', 'to invoice')], ['id'])
                sale_order_ids = list(map(lambda x: x['id'], sale_order_ids))

                if to_invoice_amount and sale_order_ids:
                    if len(sale_order_ids) == 1:
                        actions.append({
                            'label': _("Create Invoice"),
                            'type': 'action',
                            'action_id': 'sale.action_view_sale_advance_payment_inv',
                            'context': json.dumps({'active_ids': sale_order_ids, 'active_model': 'project.project'}),
                        })
                    else:
                        actions.append({
                            'label': _("Create Invoice"),
                            'type': 'action',
                            'action_id': 'sale_timesheet.project_project_action_multi_create_invoice',
                            'context': json.dumps({'active_id': self.id, 'active_model': 'project.project'}),
                        })
        return actions

    def _plan_get_stat_button(self):
        stat_buttons = []
        num_projects = len(self)
        if num_projects == 1:
            action_data = _to_action_data('project.project', res_id=self.id,
                                          views=[[self.env.ref('project.edit_project').id, 'form']])
        else:
            action_data = _to_action_data(action=self.env.ref('project.open_view_project_all_config'),
                                          domain=[('id', 'in', self.ids)])

        stat_buttons.append({
            'name': _('Project') if num_projects == 1 else _('Projects'),
            'count': num_projects,
            'icon': 'fa fa-puzzle-piece',
            'action': action_data
        })

        # if only one project, add it in the context as default value
        tasks_domain = [('project_id', 'in', self.ids)]
        tasks_context = self.env.context.copy()
        tasks_context.pop('search_default_name', False)
        late_tasks_domain = [('project_id', 'in', self.ids), ('date_deadline', '<', fields.Date.to_string(fields.Date.today())), ('date_end', '=', False)]
        overtime_tasks_domain = [('project_id', 'in', self.ids), ('overtime', '>', 0), ('planned_hours', '>', 0)]

        if len(self) == 1:
            tasks_context = {**tasks_context, 'default_project_id': self.id}
        elif len(self):
            task_projects_ids = self.env['project.task'].read_group([('project_id', 'in', self.ids)], ['project_id'], ['project_id'])
            task_projects_ids = [p['project_id'][0] for p in task_projects_ids]
            if len(task_projects_ids) == 1:
                tasks_context = {**tasks_context, 'default_project_id': task_projects_ids[0]}

        stat_buttons.append({
            'name': _('Tasks'),
            'count': sum(self.mapped('task_count')),
            'icon': 'fa fa-tasks',
            'action': _to_action_data(
                action=self.env.ref('project.action_view_task'),
                domain=tasks_domain,
                context=tasks_context
            )
        })
        stat_buttons.append({
            'name': [_("Tasks"), _("Late")],
            'count': self.env['project.task'].search_count(late_tasks_domain),
            'icon': 'fa fa-tasks',
            'action': _to_action_data(
                action=self.env.ref('project.action_view_task'),
                domain=late_tasks_domain,
                context=tasks_context,
            ),
        })
        stat_buttons.append({
            'name': [_("Tasks"), _("in Overtime")],
            'count': self.env['project.task'].search_count(overtime_tasks_domain),
            'icon': 'fa fa-tasks',
            'action': _to_action_data(
                action=self.env.ref('project.action_view_task'),
                domain=overtime_tasks_domain,
                context=tasks_context,
            ),
        })

        if self.env.user.has_group('sales_team.group_sale_salesman_all_leads'):
            # read all the sale orders linked to the projects' tasks
            task_so_ids = self.env['project.task'].search_read([
                ('project_id', 'in', self.ids), ('sale_order_id', '!=', False)
            ], ['sale_order_id'])
            task_so_ids = [o['sale_order_id'][0] for o in task_so_ids]

            sale_orders = self.mapped('sale_line_id.order_id') | self.env['sale.order'].browse(task_so_ids)
            if sale_orders:
                stat_buttons.append({
                    'name': _('Sales Orders'),
                    'count': len(sale_orders),
                    'icon': 'fa fa-dollar',
                    'action': _to_action_data(
                        action=self.env.ref('sale.action_orders'),
                        domain=[('id', 'in', sale_orders.ids)],
                        context={'create': False, 'edit': False, 'delete': False}
                    )
                })

                invoice_ids = self.env['sale.order'].search_read([('id', 'in', sale_orders.ids)], ['invoice_ids'])
                invoice_ids = list(itertools.chain(*[i['invoice_ids'] for i in invoice_ids]))
                invoice_ids = self.env['account.move'].search_read([('id', 'in', invoice_ids), ('move_type', '=', 'out_invoice')], ['id'])
                invoice_ids = list(map(lambda x: x['id'], invoice_ids))

                if invoice_ids:
                    stat_buttons.append({
                        'name': _('Invoices'),
                        'count': len(invoice_ids),
                        'icon': 'fa fa-pencil-square-o',
                        'action': _to_action_data(
                            action=self.env.ref('account.action_move_out_invoice_type'),
                            domain=[('id', 'in', invoice_ids), ('move_type', '=', 'out_invoice')],
                            context={'create': False, 'delete': False}
                        )
                    })

        ts_tree = self.env.ref('hr_timesheet.hr_timesheet_line_tree')
        ts_form = self.env.ref('hr_timesheet.hr_timesheet_line_form')
        if self.env.company.timesheet_encode_uom_id == self.env.ref('uom.product_uom_day'):
            timesheet_label = [_('Days'), _('Recorded')]
        else:
            timesheet_label = [_('Hours'), _('Recorded')]

        stat_buttons.append({
            'name': timesheet_label,
            'count': sum(self.mapped('total_timesheet_time')),
            'icon': 'fa fa-calendar',
            'action': _to_action_data(
                'account.analytic.line',
                domain=[('project_id', 'in', self.ids)],
                views=[(ts_tree.id, 'list'), (ts_form.id, 'form')],
            )
        })

        return stat_buttons


def _to_action_data(model=None, *, action=None, views=None, res_id=None, domain=None, context=None):
    # pass in either action or (model, views)
    if action:
        assert model is None and views is None
        act = {
            field: value
            for field, value in action.sudo().read()[0].items()
            if field in action._get_readable_fields()
        }
        act = clean_action(act, env=action.env)
        model = act['res_model']
        views = act['views']
    # FIXME: search-view-id, possibly help?
    descr = {
        'data-model': model,
        'data-views': json.dumps(views),
    }
    if context is not None: # otherwise copy action's?
        descr['data-context'] = json.dumps(context)
    if res_id:
        descr['data-res-id'] = res_id
    elif domain:
        descr['data-domain'] = json.dumps(domain)
    return descr

```

## File: models\project_sale_line_employee_map.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ProjectProductEmployeeMap(models.Model):
    _name = 'project.sale.line.employee.map'
    _description = 'Project Sales line, employee mapping'

    project_id = fields.Many2one('project.project', "Project", required=True)
    employee_id = fields.Many2one('hr.employee', "Employee", required=True)
    sale_line_id = fields.Many2one('sale.order.line', "Sale Order Item", domain=[('is_service', '=', True)])
    company_id = fields.Many2one('res.company', string='Company', related='project_id.company_id')
    timesheet_product_id = fields.Many2one(
        'product.product', string='Service',
        domain="""[
            ('type', '=', 'service'),
            ('invoice_policy', '=', 'delivery'),
            ('service_type', '=', 'timesheet'),
            '|', ('company_id', '=', False), ('company_id', '=', company_id)]""")
    price_unit = fields.Float("Unit Price", compute='_compute_price_unit', store=True, readonly=True)
    currency_id = fields.Many2one('res.currency', string="Currency", compute='_compute_price_unit', store=True, readonly=False)

    _sql_constraints = [
        ('uniqueness_employee', 'UNIQUE(project_id,employee_id)', 'An employee cannot be selected more than once in the mapping. Please remove duplicate(s) and try again.'),
    ]

    @api.depends('sale_line_id', 'sale_line_id.price_unit', 'timesheet_product_id')
    def _compute_price_unit(self):
        for line in self:
            if line.sale_line_id:
                line.price_unit = line.sale_line_id.price_unit
                line.currency_id = line.sale_line_id.currency_id
            elif line.timesheet_product_id:
                line.price_unit = line.timesheet_product_id.lst_price
                line.currency_id = line.timesheet_product_id.currency_id
            else:
                line.price_unit = 0
                line.currency_id = False

    @api.onchange('timesheet_product_id')
    def _onchange_timesheet_product_id(self):
        if self.timesheet_product_id:
            self.price_unit = self.timesheet_product_id.lst_price
        else:
            self.price_unit = 0.0

    @api.model
    def create(self, values):
        res = super(ProjectProductEmployeeMap, self).create(values)
        res._update_project_timesheet()
        return res

    def write(self, values):
        res = super(ProjectProductEmployeeMap, self).write(values)
        self._update_project_timesheet()
        return res

    def _update_project_timesheet(self):
        self.filtered(lambda l: l.sale_line_id).project_id._update_timesheets_sale_line_id()

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.osv import expression
import math


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    timesheet_ids = fields.Many2many('account.analytic.line', compute='_compute_timesheet_ids', string='Timesheet activities associated to this sale')
    timesheet_count = fields.Float(string='Timesheet activities', compute='_compute_timesheet_ids', groups="hr_timesheet.group_hr_timesheet_user")

    # override domain
    project_id = fields.Many2one(domain="['|', ('bill_type', '=', 'customer_task'), ('pricing_type', '=', 'fixed_rate'), ('analytic_account_id', '!=', False), ('company_id', '=', company_id)]")
    timesheet_encode_uom_id = fields.Many2one('uom.uom', related='company_id.timesheet_encode_uom_id')
    timesheet_total_duration = fields.Integer("Timesheet Total Duration", compute='_compute_timesheet_total_duration', help="Total recorded duration, expressed in the encoding UoM, and rounded to the unit")

    @api.depends('analytic_account_id.line_ids')
    def _compute_timesheet_ids(self):
        for order in self:
            if order.analytic_account_id:
                order.timesheet_ids = self.env['account.analytic.line'].search(
                    [('so_line', 'in', order.order_line.ids),
                        ('amount', '<=', 0.0),
                        ('project_id', '!=', False)])
            else:
                order.timesheet_ids = []
            order.timesheet_count = len(order.timesheet_ids)

    @api.depends('timesheet_ids', 'company_id.timesheet_encode_uom_id')
    def _compute_timesheet_total_duration(self):
        for sale_order in self:
            timesheets = sale_order.timesheet_ids if self.user_has_groups('hr_timesheet.group_hr_timesheet_approver') else sale_order.timesheet_ids.filtered(lambda t: t.user_id.id == self.env.uid)
            total_time = 0.0
            for timesheet in timesheets.filtered(lambda t: not t.non_allow_billable):
                # Timesheets may be stored in a different unit of measure, so first we convert all of them to the reference unit
                total_time += timesheet.unit_amount * timesheet.product_uom_id.factor_inv
            # Now convert to the proper unit of measure
            total_time *= sale_order.timesheet_encode_uom_id.factor
            sale_order.timesheet_total_duration = total_time

    def action_view_project_ids(self):
        self.ensure_one()
        # redirect to form or kanban view
        billable_projects = self.project_ids.filtered(lambda project: project.sale_line_id)
        if len(billable_projects) == 1 and self.env.user.has_group('project.group_project_manager'):
            action = billable_projects[0].action_view_timesheet_plan()
        else:
            action = super().action_view_project_ids()
        return action

    def action_view_timesheet(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("sale_timesheet.timesheet_action_from_sales_order")
        action['context'] = {
            'search_default_billable_timesheet': True
        }  # erase default filters
        if self.timesheet_count > 0:
            action['domain'] = [('so_line', 'in', self.order_line.ids), ('project_id', '!=', False)]
        else:
            action = {'type': 'ir.actions.act_window_close'}
        return action

    def _create_invoices(self, grouped=False, final=False, date=None):
        """Link timesheets to the created invoices. Date interval is injected in the
        context in sale_make_invoice_advance_inv wizard.
        """
        moves = super()._create_invoices(grouped=grouped, final=final, date=date)
        moves._link_timesheets_to_invoice(self.env.context.get("timesheet_start_date"), self.env.context.get("timesheet_end_date"))
        return moves


class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"

    qty_delivered_method = fields.Selection(selection_add=[('timesheet', 'Timesheets')])
    analytic_line_ids = fields.One2many(domain=[('project_id', '=', False)])  # only analytic lines, not timesheets (since this field determine if SO line came from expense)
    remaining_hours_available = fields.Boolean(compute='_compute_remaining_hours_available')
    remaining_hours = fields.Float('Remaining Hours on SO', compute='_compute_remaining_hours')

    def name_get(self):
        res = super(SaleOrderLine, self).name_get()
        if self.env.context.get('with_remaining_hours'):
            names = dict(res)
            result = []
            uom_hour = self.env.ref('uom.product_uom_hour')
            uom_day = self.env.ref('uom.product_uom_day')
            for line in self:
                name = names.get(line.id)
                if line.remaining_hours_available:
                    company = self.env.company
                    encoding_uom = company.timesheet_encode_uom_id
                    remaining_time = ''
                    if encoding_uom == uom_hour:
                        hours, minutes = divmod(abs(line.remaining_hours) * 60, 60)
                        round_minutes = minutes / 30
                        minutes = math.ceil(round_minutes) if line.remaining_hours >= 0 else math.floor(round_minutes)
                        if minutes > 1:
                            minutes = 0
                            hours += 1
                        else:
                            minutes = minutes * 30
                        remaining_time =' ({sign}{hours:02.0f}:{minutes:02.0f})'.format(
                            sign='-' if line.remaining_hours < 0 else '',
                            hours=hours,
                            minutes=minutes)
                    elif encoding_uom == uom_day:
                        remaining_days = company.project_time_mode_id._compute_quantity(line.remaining_hours, encoding_uom, round=False)
                        remaining_time = ' ({qty:.02f} {unit})'.format(
                            qty=remaining_days,
                            unit=_('days') if abs(remaining_days) > 1 else _('day')
                        )
                    name = '{name}{remaining_time}'.format(
                        name=name,
                        remaining_time=remaining_time
                    )
                result.append((line.id, name))
            return result
        return res

    @api.depends('product_id.service_policy')
    def _compute_remaining_hours_available(self):
        uom_hour = self.env.ref('uom.product_uom_hour')
        for line in self:
            is_ordered_timesheet = line.product_id.service_policy == 'ordered_timesheet'
            is_time_product = line.product_uom.category_id == uom_hour.category_id
            line.remaining_hours_available = is_ordered_timesheet and is_time_product

    @api.depends('qty_delivered', 'product_uom_qty', 'analytic_line_ids')
    def _compute_remaining_hours(self):
        uom_hour = self.env.ref('uom.product_uom_hour')
        for line in self:
            remaining_hours = None
            if line.remaining_hours_available:
                qty_left = line.product_uom_qty - line.qty_delivered
                remaining_hours = line.product_uom._compute_quantity(qty_left, uom_hour)
            line.remaining_hours = remaining_hours

    @api.depends('product_id')
    def _compute_qty_delivered_method(self):
        """ Sale Timesheet module compute delivered qty for product [('type', 'in', ['service']), ('service_type', '=', 'timesheet')] """
        super(SaleOrderLine, self)._compute_qty_delivered_method()
        for line in self:
            if not line.is_expense and line.product_id.type == 'service' and line.product_id.service_type == 'timesheet':
                line.qty_delivered_method = 'timesheet'

    @api.depends('analytic_line_ids.project_id', 'analytic_line_ids.non_allow_billable', 'project_id.pricing_type', 'project_id.bill_type')
    def _compute_qty_delivered(self):
        super(SaleOrderLine, self)._compute_qty_delivered()

        lines_by_timesheet = self.filtered(lambda sol: sol.qty_delivered_method == 'timesheet')
        domain = lines_by_timesheet._timesheet_compute_delivered_quantity_domain()
        mapping = lines_by_timesheet.sudo()._get_delivered_quantity_by_analytic(domain)
        for line in lines_by_timesheet:
            line.qty_delivered = mapping.get(line.id or line._origin.id, 0.0)

    def _timesheet_compute_delivered_quantity_domain(self):
        """ Hook for validated timesheet in addionnal module """
        return [('project_id', '!=', False), ('non_allow_billable', '=', False)]

    ###########################################
    # Service : Project and task generation
    ###########################################

    def _convert_qty_company_hours(self, dest_company):
        company_time_uom_id = dest_company.project_time_mode_id
        if self.product_uom.id != company_time_uom_id.id and self.product_uom.category_id.id == company_time_uom_id.category_id.id:
            planned_hours = self.product_uom._compute_quantity(self.product_uom_qty, company_time_uom_id)
        else:
            planned_hours = self.product_uom_qty
        return planned_hours

    def _timesheet_create_project(self):
        project = super()._timesheet_create_project()
        project.write({'allow_timesheets': True})
        return project

    def _timesheet_create_project_prepare_values(self):
        """Generate project values"""
        values = super()._timesheet_create_project_prepare_values()
        values['allow_billable'] = True
        values['bill_type'] = 'customer_project'
        values['pricing_type'] = 'fixed_rate'
        return values

    def _recompute_qty_to_invoice(self, start_date, end_date):
        """ Recompute the qty_to_invoice field for product containing timesheets

            Search the existed timesheets between the given period in parameter.
            Retrieve the unit_amount of this timesheet and then recompute
            the qty_to_invoice for each current product.

            :param start_date: the start date of the period
            :param end_date: the end date of the period
        """
        lines_by_timesheet = self.filtered(lambda sol: sol.product_id and sol.product_id._is_delivered_timesheet())
        domain = lines_by_timesheet._timesheet_compute_delivered_quantity_domain()
        refund_account_moves = self.order_id.invoice_ids.filtered(lambda am: am.state == 'posted' and am.move_type == 'out_refund').reversed_entry_id
        timesheet_domain = [
            '|',
            ('timesheet_invoice_id', '=', False),
            ('timesheet_invoice_id.state', '=', 'cancel')]
        if refund_account_moves:
            credited_timesheet_domain = [('timesheet_invoice_id.state', '=', 'posted'), ('timesheet_invoice_id', 'in', refund_account_moves.ids)]
            timesheet_domain = expression.OR([timesheet_domain, credited_timesheet_domain])
        domain = expression.AND([domain, timesheet_domain])
        if start_date:
            domain = expression.AND([domain, [('date', '>=', start_date)]])
        if end_date:
            domain = expression.AND([domain, [('date', '<=', end_date)]])
        mapping = lines_by_timesheet.sudo()._get_delivered_quantity_by_analytic(domain)

        for line in lines_by_timesheet:
            line.qty_to_invoice = mapping.get(line.id, 0.0)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account
from . import account_move
from . import product
from . import project
from . import project_overview
from . import sale_order
from . import project_sale_line_employee_map

```

## File: report\project_profitability_report_analysis.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, tools


class ProfitabilityAnalysis(models.Model):

    _name = "project.profitability.report"
    _description = "Project Profitability Report"
    _order = 'project_id, sale_line_id'
    _auto = False

    analytic_account_id = fields.Many2one('account.analytic.account', string='Analytic Account', readonly=True)
    project_id = fields.Many2one('project.project', string='Project', readonly=True)
    currency_id = fields.Many2one('res.currency', string='Project Currency', readonly=True)
    company_id = fields.Many2one('res.company', string='Project Company', readonly=True)
    user_id = fields.Many2one('res.users', string='Project Manager', readonly=True)
    partner_id = fields.Many2one('res.partner', string='Customer', readonly=True)
    line_date = fields.Date("Date", readonly=True)
    # cost
    timesheet_unit_amount = fields.Float("Timesheet Duration", digits=(16, 2), readonly=True, group_operator="sum")
    timesheet_cost = fields.Float("Timesheet Cost", digits=(16, 2), readonly=True, group_operator="sum")
    expense_cost = fields.Float("Other Costs", digits=(16, 2), readonly=True, group_operator="sum")
    # sale revenue
    order_confirmation_date = fields.Datetime('Sales Order Confirmation Date', readonly=True)
    sale_line_id = fields.Many2one('sale.order.line', string='Sale Order Line', readonly=True)
    sale_order_id = fields.Many2one('sale.order', string='Sale Order', readonly=True)
    product_id = fields.Many2one('product.product', string='Product', readonly=True)

    amount_untaxed_to_invoice = fields.Float("Untaxed Amount to Invoice", digits=(16, 2), readonly=True, group_operator="sum")
    amount_untaxed_invoiced = fields.Float("Untaxed Amount Invoiced", digits=(16, 2), readonly=True, group_operator="sum")
    expense_amount_untaxed_to_invoice = fields.Float("Untaxed Amount to Re-invoice", digits=(16, 2), readonly=True, group_operator="sum")
    expense_amount_untaxed_invoiced = fields.Float("Untaxed Amount Re-invoiced", digits=(16, 2), readonly=True, group_operator="sum")
    other_revenues = fields.Float("Other Revenues", digits=(16, 2), readonly=True, group_operator="sum",
                                  help="All revenues that are not from timesheets and that are linked to the analytic account of the project.")
    margin = fields.Float("Margin", digits=(16, 2), readonly=True, group_operator="sum")

    _depends = {
        'sale.order.line': [
            'order_id',
            'invoice_status',
            'price_reduce',
            'product_id',
            'qty_invoiced',
            'untaxed_amount_invoiced',
            'untaxed_amount_to_invoice',
            'currency_id',
            'company_id',
            'is_downpayment',
            'project_id',
            'task_id',
            'qty_delivered_method',
        ],
        'sale.order': [
            'date_order',
            'user_id',
            'partner_id',
            'currency_id',
            'analytic_account_id',
            'order_line',
            'invoice_status',
            'amount_untaxed',
            'currency_rate',
            'company_id',
            'project_id',
        ],
    }

    def init(self):
        tools.drop_view_if_exists(self._cr, self._table)
        query = """
            CREATE VIEW %s AS (
                SELECT
                    sub.id as id,
                    sub.project_id as project_id,
                    sub.user_id as user_id,
                    sub.sale_line_id as sale_line_id,
                    sub.analytic_account_id as analytic_account_id,
                    sub.partner_id as partner_id,
                    sub.company_id as company_id,
                    sub.currency_id as currency_id,
                    sub.sale_order_id as sale_order_id,
                    sub.order_confirmation_date as order_confirmation_date,
                    sub.product_id as product_id,
                    sub.sale_qty_delivered_method as sale_qty_delivered_method,
                    sub.expense_amount_untaxed_to_invoice as expense_amount_untaxed_to_invoice,
                    sub.expense_amount_untaxed_invoiced as expense_amount_untaxed_invoiced,
                    sub.amount_untaxed_to_invoice as amount_untaxed_to_invoice,
                    sub.amount_untaxed_invoiced as amount_untaxed_invoiced,
                    sub.timesheet_unit_amount as timesheet_unit_amount,
                    sub.timesheet_cost as timesheet_cost,
                    sub.expense_cost as expense_cost,
                    sub.other_revenues as other_revenues,
                    sub.line_date as line_date,
                    (sub.expense_amount_untaxed_to_invoice + sub.expense_amount_untaxed_invoiced + sub.amount_untaxed_to_invoice +
                        sub.amount_untaxed_invoiced + sub.other_revenues + sub.timesheet_cost + sub.expense_cost)
                        as margin
                FROM (
                    SELECT
                        ROW_NUMBER() OVER (ORDER BY P.id, SOL.id) AS id,
                        P.id AS project_id,
                        P.user_id AS user_id,
                        SOL.id AS sale_line_id,
                        P.analytic_account_id AS analytic_account_id,
                        P.partner_id AS partner_id,
                        C.id AS company_id,
                        C.currency_id AS currency_id,
                        S.id AS sale_order_id,
                        S.date_order AS order_confirmation_date,
                        SOL.product_id AS product_id,
                        SOL.qty_delivered_method AS sale_qty_delivered_method,
                        COST_SUMMARY.expense_amount_untaxed_to_invoice AS expense_amount_untaxed_to_invoice,
                        COST_SUMMARY.expense_amount_untaxed_invoiced AS expense_amount_untaxed_invoiced,
                        COST_SUMMARY.amount_untaxed_to_invoice AS amount_untaxed_to_invoice,
                        COST_SUMMARY.amount_untaxed_invoiced AS amount_untaxed_invoiced,
                        COST_SUMMARY.timesheet_unit_amount AS timesheet_unit_amount,
                        COST_SUMMARY.timesheet_cost AS timesheet_cost,
                        COST_SUMMARY.expense_cost AS expense_cost,
                        COST_SUMMARY.other_revenues AS other_revenues,
                        COST_SUMMARY.line_date::date AS line_date
                    FROM project_project P
                        JOIN res_company C ON C.id = P.company_id
                        LEFT JOIN (
                            -- Each costs and revenues will be retrieved individually by sub-requests
                            -- This is required to able to get the date
                            SELECT
                                project_id,
                                analytic_account_id,
                                sale_line_id,
                                SUM(timesheet_unit_amount) AS timesheet_unit_amount,
                                SUM(timesheet_cost) AS timesheet_cost,
                                SUM(expense_cost) AS expense_cost,
                                SUM(other_revenues) AS other_revenues,
                                SUM(expense_amount_untaxed_to_invoice) AS expense_amount_untaxed_to_invoice,
                                SUM(expense_amount_untaxed_invoiced) AS expense_amount_untaxed_invoiced,
                                SUM(amount_untaxed_to_invoice) AS amount_untaxed_to_invoice,
                                SUM(amount_untaxed_invoiced) AS amount_untaxed_invoiced,
                                line_date AS line_date
                            FROM (
                                -- Get the timesheet costs
                                SELECT
                                    P.id AS project_id,
                                    P.analytic_account_id AS analytic_account_id,
                                    TS.so_line AS sale_line_id,
                                    TS.unit_amount AS timesheet_unit_amount,
                                    TS.amount AS timesheet_cost,
                                    0.0 AS other_revenues,
                                    0.0 AS expense_cost,
                                    0.0 AS expense_amount_untaxed_to_invoice,
                                    0.0 AS expense_amount_untaxed_invoiced,
                                    0.0 AS amount_untaxed_to_invoice,
                                    0.0 AS amount_untaxed_invoiced,
                                    TS.date AS line_date
                                FROM account_analytic_line TS, project_project P
                                WHERE TS.project_id IS NOT NULL AND P.id = TS.project_id AND P.active = 't' AND P.allow_timesheets = 't'

                                UNION ALL

                                -- Get the other revenues (products that are not services)
                                SELECT
                                    P.id AS project_id,
                                    P.analytic_account_id AS analytic_account_id,
                                    AAL.so_line AS sale_line_id,
                                    0.0 AS timesheet_unit_amount,
                                    0.0 AS timesheet_cost,
                                    AAL.amount + COALESCE(AAL_RINV.amount, 0) AS other_revenues,
                                    0.0 AS expense_cost,
                                    0.0 AS expense_amount_untaxed_to_invoice,
                                    0.0 AS expense_amount_untaxed_invoiced,
                                    0.0 AS amount_untaxed_to_invoice,
                                    0.0 AS amount_untaxed_invoiced,
                                    AAL.date AS line_date
                                FROM project_project P
                                    JOIN account_analytic_account AA ON P.analytic_account_id = AA.id
                                    JOIN account_analytic_line AAL ON AAL.account_id = AA.id
                                    LEFT JOIN sale_order_line_invoice_rel SOINV ON SOINV.invoice_line_id = AAL.move_id
                                    LEFT JOIN sale_order_line SOL ON SOINV.order_line_id = SOL.id
                                    LEFT JOIN account_move_line AML ON AAL.move_id = AML.id
                                                                   AND AML.parent_state = 'posted'
                                                                   AND AML.exclude_from_invoice_tab = 'f'
                                    -- Check if it's not a Credit Note for a Vendor Bill
                                    LEFT JOIN account_move RBILL ON RBILL.id = AML.move_id
                                    LEFT JOIN account_move_line BILLL ON BILLL.move_id = RBILL.reversed_entry_id
                                                                  AND BILLL.parent_state = 'posted'
                                                                  AND BILLL.exclude_from_invoice_tab = 'f'
                                                                  AND BILLL.product_id = AML.product_id
                                    -- Check if it's not an Invoice reversed by a Credit Note
                                    LEFT JOIN account_move RINV ON RINV.reversed_entry_id = AML.move_id
                                    LEFT JOIN account_move_line RINVL ON RINVL.move_id = RINV.id
                                                                  AND RINVL.parent_state = 'posted'
                                                                  AND RINVL.exclude_from_invoice_tab = 'f'
                                                                  AND RINVL.product_id = AML.product_id
                                    LEFT JOIN account_analytic_line AAL_RINV ON RINVL.id = AAL_RINV.move_id
                                WHERE AAL.amount > 0.0 AND AAL.project_id IS NULL AND P.active = 't'
                                    AND P.allow_timesheets = 't'
                                    AND BILLL.id IS NULL
                                    AND (SOL.id IS NULL
                                        OR (SOL.is_expense IS NOT TRUE AND SOL.is_downpayment IS NOT TRUE AND SOL.is_service IS NOT TRUE))

                                UNION ALL

                                -- Get the expense costs from account analytic line
                                SELECT
                                    P.id AS project_id,
                                    P.analytic_account_id AS analytic_account_id,
                                    AAL.so_line AS sale_line_id,
                                    0.0 AS timesheet_unit_amount,
                                    0.0 AS timesheet_cost,
                                    0.0 AS other_revenues,
                                    AAL.amount + COALESCE(AML_RBILLL.amount, 0) AS expense_cost,
                                    0.0 AS expense_amount_untaxed_to_invoice,
                                    0.0 AS expense_amount_untaxed_invoiced,
                                    0.0 AS amount_untaxed_to_invoice,
                                    0.0 AS amount_untaxed_invoiced,
                                    AAL.date AS line_date
                                FROM project_project P
                                    JOIN account_analytic_account AA ON P.analytic_account_id = AA.id
                                    JOIN account_analytic_line AAL ON AAL.account_id = AA.id
                                    LEFT JOIN account_move_line AML ON AAL.move_id = AML.id
                                                                   AND AML.parent_state = 'posted'
                                                                   AND AML.exclude_from_invoice_tab = 'f'
                                    -- Check if it's not a Credit Note for an Invoice
                                    LEFT JOIN account_move RINV ON RINV.id = AML.move_id
                                    LEFT JOIN account_move_line INVL ON INVL.move_id = RINV.reversed_entry_id
                                                                    AND INVL.parent_state = 'posted'
                                                                    AND INVL.exclude_from_invoice_tab = 'f'
                                                                    AND INVL.product_id = AML.product_id
                                    -- Check if it's not a Bill reversed by a Credit Note
                                    LEFT JOIN account_move RBILL ON RBILL.reversed_entry_id = AML.move_id
                                    LEFT JOIN account_move_line RBILLL ON RBILLL.move_id = RBILL.id
                                                                      AND RBILLL.parent_state = 'posted'
                                                                      AND RBILLL.exclude_from_invoice_tab = 'f'
                                                                      AND RBILLL.product_id = AML.product_id
                                    LEFT JOIN account_analytic_line AML_RBILLL ON RBILLL.id = AML_RBILLL.move_id
                                    -- Check if the AAL is not related to a consumed downpayment (when the SOL is fully invoiced - with downpayment discounted.)
                                    LEFT JOIN sale_order_line_invoice_rel SOINVDOWN ON SOINVDOWN.invoice_line_id = AML.id
                                    LEFT JOIN sale_order_line SOLDOWN on SOINVDOWN.order_line_id = SOLDOWN.id AND SOLDOWN.is_downpayment = 't'
                                WHERE AAL.amount < 0.0 AND AAL.project_id IS NULL
                                  AND INVL.id IS NULL
                                  AND SOLDOWN.id IS NULL
                                  AND P.active = 't' AND P.allow_timesheets = 't'

                                UNION ALL

                                -- Get the following values: expense amount untaxed to invoice/invoiced, amount untaxed to invoice/invoiced
                                -- These values have to be computed from all the records retrieved just above but grouped by project and sale order line
                                SELECT
                                    AMOUNT_UNTAXED.project_id AS project_id,
                                    AMOUNT_UNTAXED.analytic_account_id AS analytic_account_id,
                                    AMOUNT_UNTAXED.sale_line_id AS sale_line_id,
                                    0.0 AS timesheet_unit_amount,
                                    0.0 AS timesheet_cost,
                                    0.0 AS other_revenues,
                                    0.0 AS expense_cost,
                                    CASE
                                        WHEN SOL.qty_delivered_method = 'analytic' THEN (SOL.untaxed_amount_to_invoice / CASE COALESCE(S.currency_rate, 0) WHEN 0 THEN 1.0 ELSE S.currency_rate END)
                                        ELSE 0.0
                                    END AS expense_amount_untaxed_to_invoice,
                                    CASE
                                        WHEN SOL.qty_delivered_method = 'analytic' AND SOL.invoice_status = 'invoiced'
                                        THEN
                                            CASE
                                                WHEN T.expense_policy = 'sales_price'
                                                THEN (SOL.untaxed_amount_invoiced / CASE COALESCE(S.currency_rate, 0) WHEN 0 THEN 1.0 ELSE S.currency_rate END)
                                                ELSE -AMOUNT_UNTAXED.expense_cost
                                            END
                                        ELSE 0.0
                                    END AS expense_amount_untaxed_invoiced,
                                    CASE
                                        WHEN SOL.qty_delivered_method IN ('timesheet', 'manual') THEN (SOL.untaxed_amount_to_invoice / CASE COALESCE(S.currency_rate, 0) WHEN 0 THEN 1.0 ELSE S.currency_rate END)
                                        ELSE 0.0
                                    END AS amount_untaxed_to_invoice,
                                    CASE
                                        WHEN SOL.qty_delivered_method IN ('timesheet', 'manual') THEN (SOL.untaxed_amount_invoiced / CASE COALESCE(S.currency_rate, 0) WHEN 0 THEN 1.0 ELSE S.currency_rate END)
                                        ELSE 0.0
                                    END AS amount_untaxed_invoiced,
                                    S.date_order AS line_date
                                FROM project_project P
                                    JOIN res_company C ON C.id = P.company_id
                                    LEFT JOIN (
                                        -- Gets SOL linked to timesheets
                                        SELECT
                                            P.id AS project_id,
                                            P.analytic_account_id AS analytic_account_id,
                                            AAL.so_line AS sale_line_id,
                                            0.0 AS expense_cost
                                        FROM account_analytic_line AAL, project_project P
                                        WHERE AAL.project_id IS NOT NULL AND P.id = AAL.project_id AND P.active = 't'
                                        GROUP BY P.id, AAL.so_line
                                        UNION
                                        -- Service SOL linked to a project task AND not yet timesheeted
                                        SELECT
                                            P.id AS project_id,
                                            P.analytic_account_id AS analytic_account_id,
                                            SOL.id AS sale_line_id,
                                            0.0 AS expense_cost
                                        FROM sale_order_line SOL
                                        JOIN project_task T ON T.sale_line_id = SOL.id
                                        JOIN project_project P ON T.project_id = P.id
                                        LEFT JOIN account_analytic_line AAL ON AAL.task_id = T.id
                                        WHERE SOL.is_service = 't'
                                          AND AAL.id IS NULL -- not timesheeted
                                          AND P.active = 't' AND P.allow_timesheets = 't'
                                        GROUP BY P.id, SOL.id
                                        UNION
                                        -- Service SOL linked to project AND not yet timesheeted
                                        SELECT
                                            P.id AS project_id,
                                            P.analytic_account_id AS analytic_account_id,
                                            SOL.id AS sale_line_id,
                                            0.0 AS expense_cost
                                        FROM sale_order_line SOL
                                        JOIN project_project P ON P.sale_line_id = SOL.id
                                        LEFT JOIN account_analytic_line AAL ON AAL.project_id = P.id
                                        LEFT JOIN project_task T ON T.sale_line_id = SOL.id
                                        WHERE SOL.is_service = 't'
                                          AND AAL.id IS NULL -- not timesheeted
                                          AND (T.id IS NULL OR T.project_id != P.id) -- not linked to a task in this project
                                          AND P.active = 't' AND P.allow_timesheets = 't'
                                        GROUP BY P.id, SOL.id
                                        UNION
                                        -- Service SOL linked to analytic account AND not yet timesheeted
                                        SELECT
                                            P.id AS project_id,
                                            P.analytic_account_id AS analytic_account_id,
                                            SOL.id AS sale_line_id,
                                            0.0 AS expense_cost
                                        FROM sale_order_line SOL
                                        JOIN sale_order SO ON SO.id = SOL.order_id
                                        JOIN account_analytic_account AA ON AA.id = SO.analytic_account_id
                                        JOIN project_project P ON P.analytic_account_id = AA.id
                                        LEFT JOIN project_project PSOL ON PSOL.sale_line_id = SOL.id
                                        LEFT JOIN project_task TSOL ON TSOL.sale_line_id = SOL.id
                                        LEFT JOIN account_analytic_line AAL ON AAL.so_line = SOL.id
                                        WHERE SOL.is_service = 't'
                                          AND AAL.id IS NULL -- not timesheeted
                                          AND TSOL.id IS NULL -- not linked to a task
                                          AND PSOL.id IS NULL -- not linked to a project
                                          AND P.active = 't' AND P.allow_timesheets = 't'
                                        GROUP BY P.id, SOL.id
                                        UNION

                                        SELECT
                                            P.id AS project_id,
                                            P.analytic_account_id AS analytic_account_id,
                                            AAL.so_line AS sale_line_id,
                                            0.0 AS expense_cost
                                        FROM project_project P
                                            LEFT JOIN account_analytic_account AA ON P.analytic_account_id = AA.id
                                            LEFT JOIN account_analytic_line AAL ON AAL.account_id = AA.id
                                        WHERE AAL.amount > 0.0 AND AAL.project_id IS NULL AND P.active = 't' AND P.allow_timesheets = 't'
                                        GROUP BY P.id, AA.id, AAL.so_line
                                        UNION
                                        SELECT
                                            P.id AS project_id,
                                            P.analytic_account_id AS analytic_account_id,
                                            AAL.so_line AS sale_line_id,
                                            SUM(AAL.amount) AS expense_cost
                                        FROM project_project P
                                            LEFT JOIN account_analytic_account AA ON P.analytic_account_id = AA.id
                                            LEFT JOIN account_analytic_line AAL ON AAL.account_id = AA.id
                                        WHERE AAL.amount < 0.0 AND AAL.project_id IS NULL AND P.active = 't' AND P.allow_timesheets = 't'
                                        GROUP BY P.id, AA.id, AAL.so_line
                                        UNION
                                        SELECT
                                            P.id AS project_id,
                                            P.analytic_account_id AS analytic_account_id,
                                            SOLDOWN.id AS sale_line_id,
                                            0.0 AS expense_cost
                                        FROM project_project P
                                            LEFT JOIN sale_order_line SOL ON P.sale_line_id = SOL.id
                                            LEFT JOIN sale_order SO ON SO.id = SOL.order_id OR SO.analytic_account_id = P.analytic_account_id
                                            LEFT JOIN sale_order_line SOLDOWN ON SOLDOWN.order_id = SO.id AND SOLDOWN.is_downpayment = 't'
                                            LEFT JOIN sale_order_line_invoice_rel SOINV ON SOINV.order_line_id = SOLDOWN.id
                                            LEFT JOIN account_move_line INVL ON SOINV.invoice_line_id = INVL.id
                                                                            AND INVL.parent_state = 'posted'
                                                                            AND INVL.exclude_from_invoice_tab = 'f'
                                            LEFT JOIN account_move RINV ON INVL.move_id = RINV.reversed_entry_id
                                            LEFT JOIN account_move_line RINVL ON RINV.id = RINVL.move_id
                                                                            AND RINVL.parent_state = 'posted'
                                                                            AND RINVL.exclude_from_invoice_tab = 'f'
                                                                            AND RINVL.product_id = SOLDOWN.product_id
                                            LEFT JOIN account_analytic_line ANLI ON ANLI.move_id = RINVL.id AND ANLI.amount < 0.0
                                        WHERE ANLI.id IS NULL -- there are no credit note for this downpayment
                                          AND P.active = 't' AND P.allow_timesheets = 't'
                                        GROUP BY P.id, SOLDOWN.id
                                        UNION
                                        SELECT
                                            P.id AS project_id,
                                            P.analytic_account_id AS analytic_account_id,
                                            SOL.id AS sale_line_id,
                                            0.0 AS expense_cost
                                        FROM sale_order_line SOL
                                            INNER JOIN project_project P ON SOL.project_id = P.id
                                        WHERE P.active = 't' AND P.allow_timesheets = 't'
                                        UNION
                                        SELECT
                                            P.id AS project_id,
                                            P.analytic_account_id AS analytic_account_id,
                                            SOL.id AS sale_line_id,
                                            0.0 AS expense_cost
                                        FROM sale_order_line SOL
                                            INNER JOIN project_task T ON SOL.task_id = T.id
                                            INNER JOIN project_project P ON P.id = T.project_id
                                        WHERE P.active = 't' AND P.allow_timesheets = 't'
                                    ) AMOUNT_UNTAXED ON AMOUNT_UNTAXED.project_id = P.id
                                    LEFT JOIN sale_order_line SOL ON AMOUNT_UNTAXED.sale_line_id = SOL.id
                                    LEFT JOIN sale_order S ON SOL.order_id = S.id
                                    LEFT JOIN product_product PP on (SOL.product_id = PP.id)
                                    LEFT JOIN product_template T on (PP.product_tmpl_id = T.id)
                                    WHERE P.active = 't' AND P.analytic_account_id IS NOT NULL
                            ) SUB_COST_SUMMARY
                            GROUP BY project_id, analytic_account_id, sale_line_id, line_date
                        ) COST_SUMMARY ON COST_SUMMARY.project_id = P.id
                        LEFT JOIN sale_order_line SOL ON COST_SUMMARY.sale_line_id = SOL.id
                        LEFT JOIN sale_order S ON SOL.order_id = S.id
                        WHERE P.active = 't' AND P.analytic_account_id IS NOT NULL
                    ) AS sub
            )
        """ % self._table
        self._cr.execute(query)

```

## File: report\project_profitability_report_analysis_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="project_profitability_report_view_pivot" model="ir.ui.view">
        <field name="name">project.profitability.report.pivot</field>
        <field name="model">project.profitability.report</field>
        <field name="arch" type="xml">
            <pivot string="Profitability Analysis" display_quantity="true" disable_linking="True" sample="1">
                <field name="project_id" type="row"/>
                <field name="amount_untaxed_to_invoice" type="measure"/>
                <field name="amount_untaxed_invoiced" type="measure"/>
                <field name="timesheet_cost" type="measure"/>
            </pivot>
        </field>
    </record>

    <record id="project_profitability_report_view_graph" model="ir.ui.view">
        <field name="name">project.profitability.report.graph</field>
        <field name="model">project.profitability.report</field>
        <field name="arch" type="xml">
            <graph string="Profitability Analysis" type="bar" sample="1" disable_linking="1">
                <field name="project_id" type="row"/>
                <field name="product_id" type="col"/>
                <field name="amount_untaxed_to_invoice" type="measure"/>
                <field name="amount_untaxed_invoiced" type="measure"/>
                <field name="timesheet_cost" type="measure"/>
                <field name="other_revenues" type="measure"/>
                <field name="margin" type="measure"/>
             </graph>
         </field>
    </record>

    <record id="project_profitability_report_view_search" model="ir.ui.view">
        <field name="name">project.profitability.report.search</field>
        <field name="model">project.profitability.report</field>
        <field name="arch" type="xml">
            <search string="Profitability Analysis">
                <field name="project_id"/>
                <field name="user_id"/>
                <field name="product_id"/>
                <field name="partner_id" filter_domain="[('partner_id', 'child_of', self)]"/>
                <field name="company_id" groups="base.group_multi_company"/>
                <filter string="My Projects" name="my_project" domain="[('user_id','=', uid)]"/>
                <group expand="1" string="Group By">
                    <filter string="Project" name="group_by_project" context="{'group_by':'project_id'}"/>
                    <filter string="Project Manager" name="group_by_user_id" context="{'group_by':'user_id'}"/>
                    <filter string="Customer" name="group_by_partner_id" context="{'group_by':'partner_id'}"/>
                    <filter string="Company" name="group_by_company" context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
                    <filter string="Date" name="group_by_line_date" context="{'group_by':'line_date'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="project_profitability_report_action" model="ir.actions.act_window">
        <field name="name">Project Costs and Revenues</field>
        <field name="res_model">project.profitability.report</field>
        <field name="view_mode">pivot,graph</field>
        <field name="search_view_id" ref="project_profitability_report_view_search"/>
        <field name="context">{
            'group_by_no_leaf':1,
            'group_by':[],
            'sale_show_order_product_name': 1,
        }</field>
        <field name="help">This report allows you to analyse the profitability of your projects: compare the amount to invoice, the ones already invoiced and the project cost (via timesheet cost of your employees).</field>
    </record>

    <menuitem id="menu_project_profitability_analysis"
        parent="project.menu_project_report"
        action="project_profitability_report_action"
        name="Project Costs and Revenues"
        sequence="50"/>

</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import project_profitability_report_analysis

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_project_profitability_report_analysis_manager,project.profitability.report.analysis,model_project_profitability_report,project.group_project_manager,1,1,1,1
access_project_sale_line_employee_map,access_project_sale_line_employee_map,model_project_sale_line_employee_map,base.group_user,1,0,0,0
access_project_sale_line_employee_map_manager,access_project_sale_line_employee_map_project_manager,model_project_sale_line_employee_map,project.group_project_manager,1,1,1,1
access_project_create_sale_order,access.project.create.sale.order,model_project_create_sale_order,sales_team.group_sale_salesman,1,1,1,0
access_project_create_sale_order_line,access.project.create.sale.order.line,model_project_create_sale_order_line,sales_team.group_sale_salesman,1,1,1,1
access_project_create_invoice,access.project.create.invoice,model_project_create_invoice,sales_team.group_sale_salesman_all_leads,1,1,1,0
access_project_task_create_sale_order,access.project.task.create.sale.order,model_project_task_create_sale_order,sales_team.group_sale_salesman,1,1,1,0

```

## File: security\sale_timesheet_security.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo noupdate="1">


        <!-- Override this rule because in hr_timesheet,
            the lowest access right can only see own timesheets (model: account.analytic.line)
            and this ir.rule accept all account.analytic.line in its domain.
            Therefore, we need to override this rule to change the domain, and then the
            rules for the account.analytic.line defined in timesheet will be apply.
         -->
        <record id="account.account_analytic_line_rule_billing_user" model="ir.rule">
            <field name="domain_force">[('project_id', '=', False)]</field>
        </record>

</odoo>

```

## File: static\src\js\project_overview.js

```javascript
odoo.define('sale_timesheet.project_overview', function (require) {
    "use strict";

    var qweb = require('web.qweb');
    var viewRegistry = require('web.view_registry');

    const Controller = qweb.Controller.extend({
        events: _.extend({}, qweb.Controller.prototype.events, {
            'click .project_overview_foldable': '_onFoldToggle',
        }),

        _onFoldToggle(ev) {
            const {model, resId} = ev.target.dataset;
            const shouldOpen = ev.target.classList.contains('fa-caret-right');
            if (model === 'sale.order' && shouldOpen) {
                this._openSaleOrder(resId);
            }
            else if (model === 'sale.order.line' && shouldOpen) {
                this._openSaleOrderLine(resId);
            }
            else {
                const targetClass = `.${model.replace(/\./g, '_')}_${resId || 'None'}`;
                this.$(targetClass).hide();
            }
            $(ev.target).toggleClass('fa-caret-right');
            $(ev.target).toggleClass('fa-caret-down');
        },

        _openSaleOrder(id) {
            id = id || 'None';
            const targetClass = `.sale_order_${id}`;
            this.$(`.o_timesheet_forecast_sale_order_line${targetClass}`).show();
            this.$(`.o_timesheet_forecast_hr_employee${targetClass}`).hide();
            this.$(`.o_timesheet_forecast_sale_order_line${targetClass} .fa`).removeClass('fa-caret-down');
            this.$(`.o_timesheet_forecast_sale_order_line${targetClass} .fa`).addClass('fa-caret-right');

        },

        _openSaleOrderLine(id) {
            id = id || 'None';
            const targetClass = `.sale_order_line_${id}`;
            this.$(`.o_timesheet_forecast_hr_employee${targetClass}`).show();
        },
    });

    var ProjectOverview = qweb.View.extend({
        withSearchBar: true,
        searchMenuTypes: ['filter', 'favorite'],

        config: _.extend({}, qweb.View.prototype.config, {
            Controller: Controller,
        }),
    });

    viewRegistry.add('project_overview', ProjectOverview);
});

```

## File: static\src\js\sale_project_kanban_controller.js

```javascript
odoo.define('sale_timesheet.sale_project_kanban_controller', function (require) {
"use strict";

var core = require('web.core');
var ProjectKanbanController = require('project.project_kanban');
var session = require('web.session');

var QWeb = core.qweb;

// YTI TODO : Master remove file

var SaleProjectKanbanController = ProjectKanbanController.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _showCreateSOButton: async function () {
        var self = this;
        this.activeProjectIds = this.initialState.context.active_ids;

        if (!this.activeProjectIds || this.activeProjectIds.length !== 1) {
            this.showCreateSaleOrder = false;
            return;
        }
        var canCreateSO = await session.user_has_group('sales_team.group_sale_salesman');
        if (canCreateSO) {
            await this._rpc({
                model: 'project.project',
                method: 'search_count',
                args: [[
                    ["id", "in", this.activeProjectIds],
                    ["bill_type", "=", "customer_project"],
                    ["sale_order_id", "=", false],
                    ["allow_billable", "=", true],
                    ["allow_timesheets", "=", true],
                ]],
            }).then(function (projectCount) {
                self.showCreateSaleOrder = projectCount !== 0;
            });
        } else {
            this.showCreateSaleOrder = false;
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onCreateSaleOrder: function (ev) {
        ev.preventDefault();
        this.do_action('sale_timesheet.project_project_action_multi_create_sale_order', {
            additional_context: {
                'active_id': this.activeProjectIds && this.activeProjectIds[0],
                'active_model': "project.project",
            },
            on_close: async () => await this.reload()
        });
    },
});

return SaleProjectKanbanController;

});

```

## File: static\src\xml\sale_project_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <button t-name="SaleProjectKanbanView.buttons" type="button" class="btn btn-secondary o_create_sale_order">
        Create Sales Order
    </button>

</templates>

```

## File: views\account_invoice_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="action_timesheet_from_invoice" model="ir.actions.act_window">
        <field name="name">Timesheet</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">account.analytic.line</field>
        <field name="view_mode">tree,form,graph</field>
        <field name="context">{}</field>
        <field name="domain">[('timesheet_invoice_id', '=', active_id)]</field>
    </record>

    <record id="account_invoice_view_form_inherit_sale_timesheet" model="ir.ui.view">
        <field name="name">account.invoice.form.inherit.timesheet</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <button name="%(sale_timesheet.action_timesheet_from_invoice)d" type="action" class="oe_stat_button" icon="fa-calendar" attrs="{'invisible':[('timesheet_count','=', 0)]}">
                    <field name="timesheet_count" widget="statinfo" string="Timesheets"/>
                </button>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\hr_timesheet_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="assets_backend" name="sale timesheet assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/sale_timesheet/static/src/scss/sale_timesheet.scss"/>
                <script type="text/javascript" src="/sale_timesheet/static/src/js/project_overview.js"></script>
                <script type="text/javascript" src="/sale_timesheet/static/src/js/sale_project_kanban_controller.js"/>
        </xpath>
    </template>

    <record id="timesheet_plan" model="ir.ui.view">
        <field name="name">Timesheet Plan</field>
        <field name="type">qweb</field>
        <field name="model">project.project</field>
        <field name="arch" type="xml">
        <qweb js_class="project_overview">
        <nav class="o_qweb_cp_buttons" t-if="actions">
            <button t-foreach="actions" t-as="action"
                    type="action" class="btn btn-primary"
                    t-att-name="action['action_id']"
                    t-att-data-context="action.get('context')"
            >
                <t t-esc="action['label']"/>
            </button>
        </nav>
        <div class="o_form_view o_form_readonly o_project_plan">
            <div class="o_form_sheet_bg">
                <div class="o_form_sheet o_timesheet_plan_content">
                    <div class="o_timesheet_plan_sale_timesheet">
                        <div class="o_timesheet_plan_sale_timesheet_dashboard">

                            <div class="o_timesheet_plan_stat_buttons oe_button_box o_not_full">
                                <t t-foreach="stat_buttons" t-as="stat_button">
                                    <a class="btn oe_stat_button"
                                       type="action"
                                       t-att="stat_button['action']"
                                    >
                                        <div t-attf-class="fa fa-fw o_button_icon #{stat_button['icon']}" role="img" aria-label="Statistics" title="Statistics"></div>
                                        <div class="o_field_widget o_stat_info o_readonly_modifier" t-att-title="stat_button['name']">
                                            <t t-if="not isinstance(stat_button['name'], list)">
                                                <span class="o_stat_value" t-if="'count' in stat_button">
                                                    <t t-esc="stat_button['count']"/>
                                                </span>
                                                <span class="o_stat_text">
                                                    <t t-esc="stat_button['name']"/>
                                                </span>
                                            </t>
                                            <t t-if="isinstance(stat_button['name'], list)">
                                                <div class="oe_inline">
                                                    <span class="o_stat_value mr-1">
                                                        <t t-esc="stat_button.get('count')"/>
                                                    </span>
                                                    <span class="o_stat_value">
                                                        <t t-esc="stat_button['name'][0]"/>
                                                    </span>
                                                </div>
                                                <span class="o_stat_text">
                                                    <t t-esc="stat_button['name'][1]"/>
                                                </span>
                                            </t>
                                        </div>
                                    </a>
                                </t>
                            </div>

                            <div class="o_title">
                                <h2 t-if="is_uom_day">Recorded Days and Profitability</h2>
                                <h2 t-else="">Recorded Hours and Profitability</h2>
                            </div>

                            <t t-set="display_cost" t-value="dashboard['profit']['expense_cost'] != 0.0"/>
                            <div class="o_profitability_wrapper">
                                <div class="o_profitability_section">
                                    <div>
                                        <table class="table">
                                            <tbody>
                                                <th>
                                                    <a type="action" data-model="account.analytic.line" t-att-data-domain="json.dumps(timesheet_domain)"
                                                       data-context='{"pivot_row_groupby": ["date:month"],"pivot_column_groupby": ["timesheet_invoice_type"], "pivot_measures": ["unit_amount"]}'
                                                       data-views='[[0, "pivot"], [0, "list"]]' tabindex="-1">
                                                        <span t-if="is_uom_day">Days recorded</span>
                                                        <span t-else="">Hours recorded</span>
                                                    </a>
                                                </th>
                                                <tr>
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        <t t-if="is_uom_day" t-esc="dashboard['time']['billable_time']" t-options="{'widget': 'timesheet_uom'}"/>
                                                        <t t-else="" t-esc="dashboard['time']['billable_time']" t-options="{'widget': 'float_time'}"/>
                                                    </td>
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        (<t t-esc="dashboard['rates']['billable_time']"/> %)
                                                    </td>
                                                    <td title="Includes the time logged into tasks for which you invoice based on timesheets on tasks.">
                                                        Billed on Timesheets
                                                    </td>
                                                </tr>
                                                <tr>
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        <t t-if="is_uom_day" t-esc="dashboard['time']['billable_fixed']" t-options="{'widget': 'timesheet_uom'}"/>
                                                        <t t-else="" t-esc="dashboard['time']['billable_fixed']" t-options="{'widget': 'float_time'}"/>
                                                    </td>
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        (<t t-esc="dashboard['rates']['billable_fixed']"/> %)
                                                    </td>
                                                    <td title="Includes the time logged into tasks for which you invoice based on ordered quantities or on milestones.">
                                                        Billed at a Fixed price
                                                    </td>
                                                </tr>
                                                <tr t-if="dashboard['time']['non_billable_project'] != 0">
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        <t t-if="is_uom_day" t-esc="dashboard['time']['non_billable_project']" t-options="{'widget': 'timesheet_uom'}"/>
                                                        <t t-else="" t-esc="dashboard['time']['non_billable_project']" t-options="{'widget': 'float_time'}"/>
                                                    </td>
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        (<t t-esc="dashboard['rates']['non_billable_project']"/> %)
                                                    </td>
                                                    <td title="Includes the time logged from the Timesheet module that is linked to a project, but not to a task.">
                                                        No task found
                                                    </td>
                                                </tr>
                                                <tr>
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        <t t-if="is_uom_day" t-esc="dashboard['time']['non_billable']" t-options="{'widget': 'timesheet_uom'}"/>
                                                        <t t-else="" t-esc="dashboard['time']['non_billable']" t-options="{'widget': 'float_time'}"/>
                                                    </td>
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        (<t t-esc="dashboard['rates']['non_billable']"/> %)
                                                    </td>
                                                    <td>
                                                        <a type="action"
                                                            data-model="project.task"
                                                            data-views='[[false, "list"], [false, "form"]]'
                                                            t-att-data-domain="json.dumps([['project_id', 'in', projects.ids], ['sale_line_id', '=', False]])"
                                                            t-att-data-context="json.dumps({'active_test': False})"
                                                        >
                                                            <span class="btn-link"
                                                                  style="font-weight:normal;"
                                                                  title="Includes the time logged into a task which is not linked to any Sales Order.">
                                                                Non Billable Tasks
                                                            </span>
                                                        </a>
                                                    </td>
                                                </tr>
                                                <tr t-if="dashboard['time']['non_billable_timesheet'] > 0">
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        <t t-if="is_uom_day" t-esc="dashboard['time']['non_billable_timesheet']" t-options="{'widget': 'timesheet_uom'}"/>
                                                        <t t-else="" t-esc="dashboard['time']['non_billable_timesheet']" t-options="{'widget': 'float_time'}"/>
                                                    </td>
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        (<t t-esc="dashboard['rates']['non_billable_timesheet']"/> %)
                                                    </td>
                                                    <td>
                                                        <a type="action"
                                                            data-model="account.analytic.line"
                                                            data-views='[[false, "list"], [false, "form"]]'
                                                            t-att-data-domain="json.dumps(timesheet_domain + [('timesheet_invoice_type','=','non_billable_timesheet')])"
                                                        >
                                                            <span class="btn-link"
                                                                  style="font-weight:normal;">
                                                                Non Billable Timesheets
                                                            </span>
                                                        </a>
                                                    </td>
                                                </tr>
                                                <tr t-if="dashboard['time']['canceled'] > 0">
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        <t t-if="is_uom_day" t-esc="dashboard['time']['canceled']" t-options="{'widget': 'timesheet_uom'}"/>
                                                        <t t-else="" t-esc="dashboard['time']['canceled']" t-options="{'widget': 'float_time'}"/>
                                                    </td>
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        (<t t-esc="dashboard['rates']['canceled']"/> %)
                                                    </td>
                                                    <td title="Includes the time logged into a task which is linked to a cancelled Sales Order.">
                                                        Cancelled
                                                    </td>
                                                </tr>
                                                <tr>
                                                    <td class="o_timesheet_plan_dashboard_total">
                                                        <b>
                                                            <t t-if="is_uom_day" t-esc="dashboard['time']['total']" t-options="{'widget': 'timesheet_uom'}"/>
                                                            <t t-else="" t-esc="dashboard['time']['total']" t-options="{'widget': 'float_time'}"/>
                                                        </b>
                                                    </td>
                                                    <td><b>Total</b></td>
                                                    <td></td>
                                                </tr>
                                            </tbody>
                                        </table>
                                    </div>
                                </div>
                                <div class="o_profitability_section">
                                    <div>
                                        <table class="table">
                                            <tbody>
                                                <th>
                                                    <a type="action" data-model="project.profitability.report" t-att-data-domain="json.dumps(profitability_domain)" data-context="{'group_by_no_leaf':1, 'group_by':[], 'sale_show_order_product_name': 1}" data-views='[[0, "pivot"], [0, "graph"]]' tabindex="-1">Profitability</a>
                                                </th>
                                                <tr>
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        <t t-esc="dashboard['profit']['invoiced']" t-options='{"widget": "monetary", "display_currency": currency}'/>
                                                    </td>
                                                    <td title="Revenues linked to Timesheets already invoiced.">
                                                        Invoiced
                                                    </td>
                                                </tr>
                                                <tr>
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        <t t-esc="dashboard['profit']['to_invoice']" t-options='{"widget": "monetary", "display_currency": currency}'/>
                                                    </td>
                                                    <td title="Revenues linked to Timesheets not yet invoiced.">
                                                        To invoice
                                                    </td>
                                                </tr>
                                                <tr t-if="dashboard['profit']['other_revenues'] > 0">
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        <t t-esc="dashboard['profit']['other_revenues']" t-options='{"widget": "monetary", "display_currency": currency}'/>
                                                    </td>
                                                    <td title="All revenues that are not from timesheets and that are linked to the analytic account of the project.">Other Revenues</td>
                                                </tr>
                                                <tr>
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        <t t-esc="dashboard['profit']['cost']" t-options='{"widget": "monetary", "display_currency": currency}'/>
                                                    </td>
                                                    <td title="This cost is based on the &quot;Timesheet cost&quot; set in the HR Settings of your employees.">
                                                        Timesheet costs
                                                    </td>
                                                </tr>
                                                <tr t-if="display_cost">
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        <t t-esc="dashboard['profit']['expense_cost']" t-options='{"widget": "monetary", "display_currency": currency}'/>
                                                    </td>
                                                    <td title="Any cost linked to the Analytic Account of the Project.">
                                                        Other costs
                                                    </td>
                                                </tr>
                                                <tr t-if="display_cost &amp; (dashboard['profit']['expense_amount_untaxed_invoiced'] != 0)">
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        <t t-esc="dashboard['profit']['expense_amount_untaxed_invoiced']" t-options='{"widget": "monetary", "display_currency": currency}'/>
                                                    </td>
                                                    <td title="Costs from expenses that were reinvoiced to your customer (provided that the Analytic Account of the Project was set on the Expense).">
                                                        Re-invoiced costs
                                                    </td>
                                                </tr>
                                                <tr t-if="display_cost &amp; (dashboard['profit']['expense_amount_untaxed_to_invoice'] != 0)">
                                                    <td class="o_timesheet_plan_dashboard_cell">
                                                        <t t-esc="dashboard['profit']['expense_amount_untaxed_to_invoice']" t-options='{"widget": "monetary", "display_currency": currency}'/>
                                                    </td>
                                                    <td title="Costs from expenses that still need to be reinvoiced to your customer (provided that the Analytic Account of the Project was set on the Expense).">
                                                        To re-invoice costs
                                                    </td>
                                                </tr>
                                                <tr>
                                                    <td class="o_timesheet_plan_dashboard_total">
                                                        <b>
                                                            <t t-esc="dashboard['profit']['total']" t-options='{"widget": "monetary", "display_currency": currency}'/>
                                                        </b>
                                                    </td>
                                                    <td><b>Total</b></td>
                                                </tr>
                                            </tbody>
                                        </table>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div class="o_title">
                            <h2>Time by people</h2>
                        </div>

                        <div class="o_timesheet_plan_sale_timesheet_people_time">
                            <t t-if="not repartition_employee">
                                <p>There are no timesheets for now.</p>
                            </t>
                            <t t-if="repartition_employee">
                                <div class="table-responsive">
                                    <table class="table">
                                        <thead>
                                            <tr>
                                                <th></th>
                                                <th>Employee</th>
                                                <th t-if="is_uom_day" class="text-nowrap text-right pr-5">Days Spent</th>
                                                <th t-else="" class="text-nowrap text-right pr-5">Hours Spent</th>
                                                <td>
                                                    <div class="float-right o_timesheet_plan_badge">
                                                        <span class="badge badge-pill o_progress_billable_time">
                                                            <a type="action" data-model="account.analytic.line" t-att-data-domain="json.dumps(timesheet_domain + [('timesheet_invoice_type','=','billable_time')])" tabindex="-1">Billed on Timesheets</a>
                                                        </span>
                                                        <span class="badge badge-pill o_progress_billable_fixed">
                                                            <a type="action" data-model="account.analytic.line" t-att-data-domain="json.dumps(timesheet_domain + [('timesheet_invoice_type','=','billable_fixed')])" tabindex="-1">Billed at a Fixed price</a>
                                                        </span>
                                                        <span t-if="dashboard['time']['non_billable_project'] != 0" class="badge badge-pill o_progress_non_billable_project">
                                                            <a type="action" data-model="account.analytic.line" t-att-data-domain="json.dumps(timesheet_domain + [('timesheet_invoice_type','=','non_billable_project')])" tabindex="-1">No task found</a>
                                                        </span>
                                                        <span t-if="dashboard['time']['non_billable_timesheet'] != 0" class="badge badge-pill o_progress_non_billable_timesheet">
                                                            <a type="action" data-model="account.analytic.line" t-att-data-domain="json.dumps(timesheet_domain + [('timesheet_invoice_type','=','non_billable_timesheet')])" tabindex="-1">Non billable timesheets</a>
                                                        </span>
                                                        <span class="badge badge-pill o_progress_non_billable">
                                                            <a type="action" data-model="account.analytic.line" t-att-data-domain="json.dumps(timesheet_domain + [('timesheet_invoice_type','=','non_billable')])" tabindex="-1">Non billable tasks</a>
                                                        </span>
                                                        <!-- only show the canceled pill if there were timesheets on canceled so -->
                                                        <t t-if="sum([employee.get('canceled', 0.0) for employee in repartition_employee.values()]) > 0">
                                                            <span class="badge badge-pill o_progress_canceled">
                                                                <a type="action" data-model="account.analytic.line" t-att-data-domain="json.dumps(timesheet_domain + [('so_line.state', '=', 'cancel')])" tabindex="-1">Cancelled</a>
                                                            </span>
                                                        </t>
                                                    </div>
                                                </td>
                                            </tr>
                                        </thead>
                                        <tbody>
                                            <t t-foreach="repartition_employee" t-as="employee_id">
                                                <t t-set="employee" t-value="repartition_employee[employee_id]"/>
                                                <tr>
                                                    <td style="width: 3%; vertical-align: middle;">
                                                        <img class="img rounded-circle mr-2 mb-2" t-attf-src="/web/image?model=hr.employee&amp;field=image_128&amp;id=#{employee['employee_id']}" t-att-title="employee['employee_name']" t-att-alt="employee['employee_name']" width="25" height="25"/>
                                                    </td>
                                                    <td style="width: 15%; vertical-align: middle;" >
                                                        <a type="action" data-model="account.analytic.line" t-att-data-domain="json.dumps(timesheet_domain)" t-att-data-context="json.dumps({'search_default_employee_id': employee_id})" data-views="[[0, &quot;list&quot;]]" tabindex="-1">
                                                            <t t-esc="employee['employee_name']"/>
                                                        </a>
                                                    </td>
                                                    <td class="text-right pr-5" style="width: 10%; vertical-align: middle;">
                                                        <t t-if="is_uom_day" t-esc="employee['total']" t-options="{'widget': 'timesheet_uom'}"/>
                                                        <t t-else="" t-esc="employee['total']" t-options="{'widget': 'float_time'}"/>
                                                    </td>
                                                    <td style="vertical-align:middle">
                                                        <div class="border rounded">
                                                            <div t-if="repartition_employee_max" class="progress" t-attf-style="width: {{max(0, employee['total'] / repartition_employee_max * 100)}}%; margin-bottom: 0em;">

                                                                <t t-set="total" t-value="employee['total'] or 1.0" />
                                                                <t t-call="sale_timesheet.progressbar">
                                                                    <t t-set="label">Billed on Timesheets</t>
                                                                    <t t-set="key" t-translation="off">billable_time</t>
                                                                </t>
                                                                <t t-call="sale_timesheet.progressbar">
                                                                    <t t-set="label">Billed at a Fixed price</t>
                                                                    <t t-set="key" t-translation="off">billable_fixed</t>
                                                                </t>
                                                                <t t-call="sale_timesheet.progressbar">
                                                                    <t t-set="label">No task found</t>
                                                                    <t t-set="key" t-translation="off">non_billable_project</t>
                                                                </t>
                                                                <t t-call="sale_timesheet.progressbar">
                                                                    <t t-set="label">Non billable timesheets</t>
                                                                    <t t-set="key" t-translation="off">non_billable_timesheet</t>
                                                                </t>
                                                                <t t-call="sale_timesheet.progressbar">
                                                                    <t t-set="label">Non billable tasks</t>
                                                                    <t t-set="key" t-translation="off">non_billable</t>
                                                                </t>
                                                                <t t-call="sale_timesheet.progressbar">
                                                                    <t t-set="label">Cancelled</t>
                                                                    <t t-set="key" t-translation="off">canceled</t>
                                                                </t>
                                                            </div>
                                                        </div>
                                                    </td>
                                                </tr>
                                            </t>
                                        </tbody>
                                    </table>
                                </div>
                            </t>
                        </div>

                        <div class="o_title">
                            <h2>Timesheets</h2>
                        </div>

                        <!-- NOTE: this template to display a table works whatever the length of the rows, as project_timesheet_forecast_sale extends the table to add forecasts -->
                        <div class="o_project_plan_project_timesheet_forecast">
                            <t t-if="timesheet_forecast_table and timesheet_forecast_table['rows']">
                                <div class="table-responsive">
                                    <table class="table">
                                        <thead>
                                            <tr>
                                                <th></th>
                                                <th colspan="5" id="table_plan_title" class="o_right_bordered"><h3>Timesheets</h3></th>
                                                <th colspan="2" id="table_plan_total"></th>
                                            </tr>
                                            <tr>
                                                <t t-foreach="timesheet_forecast_table['header']" t-as="header_val">
                                                    <th t-att-class="'o_right_bordered' if header_val_index in [5,10] else ''">
                                                        <span t-att-title="header_val['tooltip']"><t t-esc="header_val['label']"/></span>
                                                    </th>
                                                </t>
                                            </tr>
                                        </thead>
                                        <tbody>
                                            <t t-set="row_is_milestone" t-value="False"/>
                                            <t t-set="current_order" t-value="False"/>
                                            <t t-set="current_order_line" t-value="False"/>
                                            <t t-foreach="timesheet_forecast_table['rows']" t-as="row">
                                                <t t-set="row_type" t-value="row[0].get('type')"/>
                                                <t t-if="row_type == 'sale_order_line'">
                                                    <t t-set="row_is_milestone" t-value="row[0].get('is_milestone')"/>
                                                </t>
                                                <t t-if="row_type == 'sale_order'">
                                                    <t t-set="current_order" t-value="False"/>
                                                    <t t-set="current_order_line" t-value="False"/>
                                                </t>
                                                <t t-if="row_type == 'sale_order_line'">
                                                    <t t-set="current_order_line" t-value="False"/>
                                                </t>
                                                <t t-set="foldable" t-value="row[0].get('has_children')"/>
                                                <tr t-att-class="'o_timesheet_forecast_' + row_type + ' sale_order_' + str(current_order) + ' sale_order_line_' + str(current_order_line)"
                                                    t-att-style="'display: none;' if row_type not in ('sale_order', 'sale_order_line') else ''">
                                                    <t t-foreach="row" t-as="row_value">
                                                        <td t-att-class="'o_right_bordered' if row_value_index in [5,10] else '' + ' text-center' if row_value_index != 0 else ''">
                                                            <t t-if="row_value_index == 0">
                                                                <span t-if="foldable" t-att-class="('fa fa-caret-down' if row_type == 'sale_order' else 'fa fa-caret-right') + (' project_overview_foldable' if foldable else '')"
                                                                      style="cursor: pointer;" t-att-data-model="row[0].get('res_model')" t-att-data-res-id="row[0].get('res_id')"/>
                                                                <t t-if="row_type == 'sale_order'">
                                                                    <t t-if="env.user.has_group('sales_team.group_sale_salesman')">
                                                                        <a type="action" t-att-data-model="row_value['res_model']" t-att-data-res-id="row_value['res_id']" t-att-class="'o_timesheet_plan_redirect' if row_value['res_id'] else ''">
                                                                           <t t-esc="row_value.get('label')"/>
                                                                        </a>
                                                                    </t>
                                                                    <t t-else="">
                                                                        <t t-esc="row_value.get('label')"/>
                                                                    </t>
                                                                    <span t-if="row_value.get('canceled')" class="badge badge-pill o_canceled_tag">
                                                                        Cancelled
                                                                    </span>
                                                                </t>
                                                                <t t-if="row_type != 'sale_order'">
                                                                    <t t-if="not row_is_milestone">
                                                                        <span><t t-esc="row_value.get('label')"/></span>
                                                                    </t>
                                                                     <t t-if="row_is_milestone">
                                                                        <span><i><t t-esc="row_value.get('label')"/></i></span>
                                                                    </t>
                                                                </t>
                                                            </t>
                                                            <t t-if="row_value_index != 0">
                                                                <t t-if="row_value_index &lt; len(row)-2">
                                                                    <t t-if="row_is_milestone">
                                                                        <i t-att-class="'text-muted' if not row_value else ''">
                                                                            <t t-if="is_uom_day" t-esc="row_value" t-options="{'widget': 'timesheet_uom'}"/>
                                                                            <t t-else="" t-esc="row_value" t-options="{'widget': 'float_time'}"/>
                                                                        </i>
                                                                    </t>
                                                                    <t t-else="">
                                                                        <span t-att-class="'text-muted' if not row_value else ''">
                                                                            <t t-if="is_uom_day" t-esc="row_value" t-options="{'widget': 'timesheet_uom'}"/>
                                                                            <t t-else="" t-esc="row_value" t-options="{'widget': 'float_time'}"/>
                                                                        </span>
                                                                    </t>
                                                                </t>
                                                                <t t-else="">
                                                                    <t t-if="not row_is_milestone and not row[0].get('type') == 'hr_employee'">
                                                                        <span t-att-class="'text-muted' if not row_value else ''">
                                                                            <t t-if="is_uom_day" t-esc="row_value" t-options="{'widget': 'timesheet_uom'}"/>
                                                                            <t t-else="" t-esc="row_value" t-options="{'widget': 'float_time'}"/>
                                                                        </span>
                                                                    </t>
                                                                </t>
                                                            </t>
                                                        </td>
                                                    </t>
                                                </tr>
                                                <t t-if="row_type == 'sale_order_line'">
                                                    <t t-set="current_order_line" t-value="row[0].get('res_id')"/>
                                                </t>
                                                <t t-if="row_type == 'sale_order'">
                                                    <t t-set="current_order" t-value="row[0].get('res_id')"/>
                                                </t>
                                            </t>
                                        </tbody>
                                    </table>
                                </div>
                            </t>
                        </div>

                    </div>
                </div>
            </div>
        </div>
        </qweb>
        </field>
    </record>

    <template id="progressbar" name="project overview progressbar segments">
        <t t-set="amount" t-value="employee[key]"/>
        <t t-if="amount &gt; 0">
            <t t-set="title">
                <t t-esc="label"/>: <t t-if="is_uom_day" t-esc="amount" t-options="{'widget': 'timesheet_uom'}"/><t t-else="" t-esc="amount" t-options="{'widget': 'float_time'}"/>
            </t>
            <a t-attf-class="progress-bar o_progress_{{key}}"
               t-attf-style="width: {{amount / total * 100}}%"
               type="action" data-model="account.analytic.line"
               t-att-data-domain="employee['__domain_' + key]"
            >
                <span t-att-title="title" style="font-size: 0px; width: 100%; height: 100%;">
                    <t t-esc="label" />
                </span>
            </a>
        </t>
    </template>

</odoo>

```

## File: views\hr_timesheet_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="timesheet_view_search" model="ir.ui.view">
            <field name="name">account.analytic.line.search</field>
            <field name="model">account.analytic.line</field>
            <field name="inherit_id" ref="hr_timesheet.hr_timesheet_line_search"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='department_id']" position="after">
                    <field name="timesheet_invoice_type"/>
                </xpath>
                <xpath expr="//filter[@name='month']" position="before">
                    <filter name="billable_timesheet" string="Billable" domain="[('so_line', '!=', False), ('non_allow_billable', '=', False)]"/>
                    <filter name="non_billable_timesheet" string="Non Billable" domain="['|', ('so_line', '=', False), ('non_allow_billable', '=', True)]"/>
                    <separator/>
                    <filter name="billable_time" string="Billed on Timesheets" domain="[('timesheet_invoice_type', '=', 'billable_time')]"/>
                    <filter name="billable_fixed" string="Billed at a Fixed Price" domain="[('timesheet_invoice_type', '=', 'billable_fixed')]"/>
                    <filter name="non_billable" string="Non Billable Tasks" domain="[('timesheet_invoice_type', '=', 'non_billable')]"/>
                    <separator/>
                </xpath>
                <xpath expr="//filter[@name='groupby_department']" position="before">
                        <filter string="Billable Type" name="groupby_timesheet_invoice_type" domain="[]" context="{'group_by': 'timesheet_invoice_type'}"/>
                </xpath>
            </field>
    </record>

    <record id="timesheet_view_pivot_revenue" model="ir.ui.view">
        <field name="name">account.analytic.line.pivot.revenue</field>
        <field name="model">account.analytic.line</field>
        <field name="arch" type="xml">
            <pivot string="Timesheet" sample="1">
                <field name="employee_id" type="row"/>
                <field name="date" interval="month" type="col"/>
                <field name="unit_amount" type="measure"/>
            </pivot>
        </field>
    </record>

    <record id="timesheet_view_tree_sale" model="ir.ui.view">
        <field name="name">account.analytic.line.view.tree.with.allow.billable</field>
        <field name="model">account.analytic.line</field>
        <field name="inherit_id" ref="hr_timesheet.timesheet_view_tree_user"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='user_id']" position="after">
                <field name="non_allow_billable" attrs="{'invisible': [('non_allow_billable', '=', False)]}" optional="hide"/>
            </xpath>
        </field>
    </record>

    <!--
        Timesheet from Sales Order
    -->
    <record id="timesheet_action_from_sales_order" model="ir.actions.act_window">
        <field name="name">Timesheets</field>
        <field name="res_model">account.analytic.line</field>
        <field name="search_view_id" ref="hr_timesheet.hr_timesheet_line_search"/>
        <field name="domain">[('project_id', '!=', False)]</field>
    </record>

    <record id="timesheet_action_from_sales_order_tree" model="ir.actions.act_window.view">
        <field name="sequence" eval="4"/>
        <field name="view_mode">tree</field>
        <field name="view_id" ref="hr_timesheet.timesheet_view_tree_user"/>
        <field name="act_window_id" ref="timesheet_action_from_sales_order"/>
    </record>

    <record id="timesheet_action_from_sales_order_form" model="ir.actions.act_window.view">
        <field name="sequence" eval="5"/>
        <field name="view_mode">form</field>
        <field name="view_id" ref="hr_timesheet.timesheet_view_form_user"/>
        <field name="act_window_id" ref="timesheet_action_from_sales_order"/>
    </record>

    <!--
        Reporting
    -->
    <record id="timesheet_action_billing_report" model="ir.actions.act_window">
        <field name="name">Timesheets By Billing Type</field>
        <field name="res_model">account.analytic.line</field>
        <field name="view_mode">pivot,graph</field>
        <field name="domain">[('project_id', '!=', False)]</field>
        <field name="context">{
            'search_default_groupby_date': 1,
        }</field>
        <field name="search_view_id" ref="hr_timesheet.hr_timesheet_line_search"/>
    </record>

    <record id="view_hr_timesheet_line_pivot_billing_rate" model="ir.ui.view">
        <field name="name">account.analytic.line.pivot.billing.rate</field>
        <field name="model">account.analytic.line</field>
        <field name="arch" type="xml">
            <pivot string="Timesheet" sample="1">
                <field name="timesheet_invoice_type" type="col"/>
                <field name="unit_amount" type="measure" widget="timesheet_uom"/>
                <field name="amount" string="Timesheet Costs"/>
            </pivot>
        </field>
    </record>

    <record id="timesheet_action_view_report_by_billing_rate_pivot" model="ir.actions.act_window.view">
        <field name="sequence" eval="5"/>
        <field name="view_mode">pivot</field>
        <field name="view_id" ref="view_hr_timesheet_line_pivot_billing_rate"/>
        <field name="act_window_id" ref="timesheet_action_billing_report"/>
    </record>

    <record id="timesheet_action_view_report_by_billing_rate_graph" model="ir.actions.act_window.view">
        <field name="sequence" eval="6"/>
        <field name="view_mode">graph</field>
        <field name="view_id" ref="hr_timesheet.view_hr_timesheet_line_graph"/>
        <field name="act_window_id" ref="timesheet_action_billing_report"/>
    </record>

    <menuitem id="menu_timesheet_billing_analysis"
            parent="hr_timesheet.menu_timesheets_reports_timesheet"
            action="timesheet_action_billing_report"
            name="By Billing Type"
            sequence="40"/>

    <!--
        Plan
    -->
    <record id="timesheet_action_plan_pivot" model="ir.actions.act_window">
        <field name="name">Timesheet</field>
        <field name="res_model">account.analytic.line</field>
        <field name="view_mode">pivot,tree,form</field>
        <field name="domain">[('project_id', '!=', False)]</field>
        <field name="search_view_id" ref="hr_timesheet.hr_timesheet_line_search"/>
    </record>

    <record id="timesheet_action_from_plan" model="ir.actions.act_window">
        <field name="name">Timesheet</field>
        <field name="res_model">account.analytic.line</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('project_id', '!=', False)]</field>
        <field name="search_view_id" ref="hr_timesheet.hr_timesheet_line_search"/>
    </record>

</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_product_timesheet_form" model="ir.ui.view">
        <field name="name">product.template.timesheet.form</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="sale.product_template_form_view_invoice_policy"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='invoice_policy']" position="attributes">
                <attribute name="invisible">False</attribute>
                <attribute name="attrs">{'invisible': [('type','==','service')]}</attribute>
            </xpath>
            <xpath expr="//field[@name='service_type']" position="after">
                <field name="service_policy" widget="radio" attrs="{'invisible': [('type','!=','service')]}"/>
            </xpath>
        </field>
    </record>

    <record id="product_template_view_search_sale_timesheet" model="ir.ui.view">
        <field name="name">product.template.search.timesheet</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_search_view"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='consumable']" position="after">
                <separator/>
                <filter string="Time-based services" name="product_time_based" domain="[('type', '=', 'service'), ('invoice_policy', '=', 'delivery'), ('service_type', '=', 'timesheet')]"/>
                <filter string="Fixed price services" name="product_service_fixed" domain="[('type', '=', 'service'), ('invoice_policy', '=', 'order'), ('service_type', '=', 'timesheet')]"/>
                <filter string="Milestone services" name="product_service_milestone" domain="[('type', '=', 'service'), ('invoice_policy', '=', 'delivery'), ('service_type', '=', 'manual')]"/>
            </xpath>
        </field>
    </record>

    <record id="product_template_action_default_services" model="ir.actions.act_window">
        <field name="name">Products</field>
        <field name="res_model">product.template</field>
        <field name="view_mode">tree,form</field>
        <field name="search_view_id" ref="sale_timesheet.product_template_view_search_sale_timesheet"/>
        <field name="context">{'search_default_services': 1, 'default_type': 'service'}</field>
    </record>

</odoo>

```

## File: views\project_task_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="project_project_view_form" model="ir.ui.view">
        <field name="name">project.project.form.inherit</field>
        <field name="model">project.project</field>
        <field name="inherit_id" ref="hr_timesheet.project_invoice_form"/>
        <field name="arch" type="xml">
            <div class="oe_button_box" position="inside">
                <button string="Project Overview" class="oe_stat_button" type="object" name="action_view_timesheet" icon="fa-puzzle-piece" attrs="{'invisible': [('allow_billable', '=', False)]}"/>
            </div>
            <xpath expr="//button[@name='action_view_so']" position="attributes">
                <attribute name="attrs">{'invisible': ['|', '|', ('allow_billable', '=', False), ('sale_order_id', '=', False), ('bill_type', '!=', 'customer_project')]}</attribute>
              </xpath>
            <xpath expr="//header" position="inside">
                <button name="action_make_billable" string="Create Sales Order" type="object" attrs="{'invisible': [('display_create_order', '=', False)]}" groups="sales_team.group_sale_salesman"/>
            </xpath>
            <xpath expr="//page[@name='settings']" position="after">
                <page name="billing_employee_rate" string="Invoicing" attrs="{'invisible': [('allow_billable', '=', False)]}">
                    <group>
                        <group>
                            <field name="display_create_order" invisible="1"/>
                            <field name="bill_type" widget="radio"/>
                            <field name="pricing_type" attrs="{'invisible': ['|', ('allow_billable', '=', False), ('bill_type', '!=', 'customer_project')], 'required': ['&amp;', ('allow_billable', '=', True), ('allow_timesheets', '=', True)]}" widget="radio"/>
                            <field name="timesheet_product_id" string="Service" attrs="{'invisible': ['|', '|', ('allow_timesheets', '=', False), ('sale_order_id', '!=', False), ('bill_type', '!=', 'customer_task')], 'required': ['&amp;', ('allow_billable', '=', True), ('allow_timesheets', '=', True)]}" context="{'default_type': 'service', 'default_service_policy': 'delivered_timesheet', 'default_service_type': 'timesheet'}"/>
                            <field name="sale_order_id" attrs="{'invisible': [('bill_type', '!=', 'customer_project')], 'readonly': [('sale_order_id', '!=', False)]}" force_save="1" options="{'no_create': True, 'no_edit': True, 'delete': False, 'no_open': True}"/>
                            <field name="sale_line_id" string="Default Sales Order Item" attrs="{'invisible': [('bill_type', '!=', 'customer_project')]}" options="{'no_create': True, 'no_edit': True, 'delete': False, 'no_open': True}"/>
                        </group>
                    </group>
                    <field name="sale_line_employee_ids" attrs="{'invisible': ['|', ('bill_type', '!=', 'customer_project'), ('pricing_type', '!=', 'employee_rate')]}">
                        <tree editable="top">
                            <field name="company_id" invisible="1"/>
                            <field name="project_id" invisible="1"/>
                            <field name="employee_id" options="{'no_create': True}"/>
                            <field name="timesheet_product_id" attrs="{'column_invisible': [('parent.sale_order_id', '!=', False)]}" invisible="1"/>
                            <field name="sale_line_id" attrs="{'required': True}" options="{'no_create': True}" domain="[('order_id','=',parent.sale_order_id), ('is_service', '=', True)]"/>
                            <field name="price_unit" widget="monetary" force_save="1" options="{'currency_field': 'currency_id'}"/>
                            <field name="currency_id" invisible="1"/>
                        </tree>
                    </field>
                </page>
            </xpath>
            <xpath expr="//div[@id='timesheet_settings']/.." position="after">
                <div class="row mt16 o_settings_container">
                    <div class="col-lg-6 o_setting_box" id="allow_billable_container">
                        <div class="o_setting_left_pane">
                            <field name="allow_billable"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="allow_billable"/>
                            <div class="text-muted" id="allow_billable_setting">
                                Invoice your time and material to customers
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

    <record id="project_project_view_form_salesman" model="ir.ui.view">
        <field name="name">project.project.form.inherit.salesman</field>
        <field name="model">project.project</field>
        <field name="inherit_id" ref="sale_timesheet.project_project_view_form"/>
        <field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
        <field name="arch" type="xml">
            <field name="sale_order_id" position="attributes">
                <attribute name="options">{'no_create': True, 'no_edit': True, 'delete': False}</attribute>
            </field>
            <field name="sale_line_id" position="attributes">
                <attribute name="options">{'no_create': True, 'no_edit': True, 'delete': False}</attribute>
            </field>
        </field>
    </record>

    <record id="project_project_view_form_simplified_inherit" model="ir.ui.view">
        <field name="name">project.project.view.form.simplified.inherit</field>
        <field name="model">project.project</field>
        <field name="inherit_id" ref="hr_timesheet.project_project_view_form_simplified_inherit_timesheet"/>
        <field name="arch" type="xml">
            <field name="allow_timesheets" position="after">
                <field name="company_id" invisible="1"/>
                <field name="allow_billable"/>
            </field>
        </field>
    </record>

    <record id="project_project_view_kanban_inherit_sale_timesheet" model="ir.ui.view">
        <field name="name">project.project.kanban.inherit.sale.timesheet</field>
        <field name="model">project.project</field>
        <field name="inherit_id" ref="hr_timesheet.view_project_kanban_inherited"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='allow_timesheets']" position="after">
                <field name="allow_billable"/>
                <field name="warning_employee_rate" invisible="1"/>
            </xpath>
            <xpath expr="//div[hasclass('o_primary')]//span" position="inside">
                <t t-if="1 == 0">
                   <i class="ml-2 fa fa-exclamation-triangle text-danger small" role="img" title="Some of the employees who are recording time on this project are not linked to any Sales Order Item. This means that their time will be considered as non-billable." aria-label="Some of the employees who are recording time on this project are not linked to any ales Order Item. This means that their time will be considered as non-billable."/>
                </t>
            </xpath>
            <xpath expr="//a[@name='action_view_account_analytic_line']" position="attributes">
                <attribute name="t-if">record.analytic_account_id.raw_value and !record.allow_timesheets.raw_value and record.allow_billable.raw_value</attribute>
            </xpath>
            <xpath expr="//a[@t-if='record.allow_timesheets.raw_value']" position="replace">
                <a t-if="record.allow_timesheets.raw_value and record.allow_billable.raw_value" name="action_view_timesheet" type="object" class="o_project_kanban_box o_project_timesheet_box" groups="project.group_project_manager">
                    <div>
                        <span class="o_label">Overview</span>
                    </div>
                </a>
            </xpath>
        </field>
    </record>

        <record id="view_sale_service_inherit_form2" model="ir.ui.view">
            <field name="name">sale.service.form.view.inherit</field>
            <field name="model">project.task</field>
            <field name="groups_id" eval="[(4, ref('base.group_user'))]"/>
            <field name="inherit_id" ref="project.view_task_form2"/>
            <field name="arch" type="xml">
                <xpath expr="//header" position='inside'>
                    <field name="allow_billable" invisible="1"/>
                    <field name="display_create_order" invisible="1"/>
                    <button name="action_make_billable" string="Create Sales Order" type="object" attrs="{'invisible': [('display_create_order', '=', False)]}" groups="sales_team.group_sale_salesman" invisible="1"/>
                </xpath>
                <xpath expr="//field[@name='email_cc']" position="after">
                    <field name="analytic_account_id" groups="base.group_no_one"/>
                </xpath>
            </field>
        </record>

        <record id="project_task_view_form_inherit_sale_timesheet" model="ir.ui.view">
            <field name="name">project.task.form.inherit.timesheet</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project.view_task_form2"/>
            <field name="groups_id" eval="[(6,0, (ref('hr_timesheet.group_hr_timesheet_user'),))]"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='timesheet_ids']/tree" position="attributes">
                    <attribute name="decoration-muted">timesheet_invoice_id != False</attribute>
                </xpath>
                <xpath expr="//field[@name='user_id']" position="after">
                    <field name="is_project_map_empty" invisible="1"/>
                    <field name="has_multi_sol" invisible="1"/>
                </xpath>
                 <xpath expr="//field[@name='partner_phone']" position="after">
                    <field name="bill_type" invisible="1"/>
                    <field name="pricing_type" invisible="1"/>
                    <field name="timesheet_product_id" invisible="1"/>
                    <field name="non_allow_billable" attrs="{'invisible': ['|', '|', '|', ('allow_billable', '=', False), ('allow_timesheets', '=', False), ('pricing_type', '!=', 'employee_rate'), ('bill_type', '=', 'customer_task')]}" invisible="1"/>
                </xpath>
                <xpath expr="//field[@name='timesheet_ids']/tree/field[@name='unit_amount']" position="before">
                    <field name="timesheet_invoice_id" invisible="1"/>
                    <field name="so_line" readonly="1" attrs="{'column_invisible': [('parent.allow_billable', '=', False)]}" context="{'with_remaining_hours': True}" optional="hide"/>
                </xpath>
                <xpath expr="//field[@name='timesheet_ids']/tree" position="inside">
                    <field name="non_allow_billable" invisible="1"/>
                </xpath>
                <xpath expr="//field[@name='remaining_hours']" position="after">
                    <field name="remaining_hours_available" invisible="1"/>
                    <span id="remaining_hours_so_label" attrs="{'invisible': ['|', '|', '|', '|', ('allow_billable', '=', False), ('sale_order_id', '=', False), ('partner_id', '=', False), ('sale_line_id', '=', False), ('remaining_hours_available', '=', False)]}">
                        <label class="font-weight-bold" for="remaining_hours_so" string="Remaining Hours on SO"
                               attrs="{'invisible': ['|', ('encode_uom_in_days', '=', True), ('remaining_hours_so', '&lt;', 0)]}"/>
                        <label class="font-weight-bold" for="remaining_hours_so" string="Remaining Days on SO"
                               attrs="{'invisible': ['|', ('encode_uom_in_days', '=', False), ('remaining_hours_so', '&lt;', 0)]}"/>
                        <label class="font-weight-bold text-danger" for="remaining_hours_so" string="Remaining Hours on SO"
                               attrs="{'invisible': ['|', ('encode_uom_in_days', '=', True), ('remaining_hours_so', '&gt;=', 0)]}"/>
                        <label class="font-weight-bold text-danger" for="remaining_hours_so" string="Remaining Days on SO"
                               attrs="{'invisible': ['|', ('encode_uom_in_days', '=', False), ('remaining_hours_so', '&gt;=', 0)]}"/>
                    </span>
                    <field name="remaining_hours_so" nolabel="1" widget="timesheet_uom" attrs="{'invisible': ['|', '|', '|', '|', ('allow_billable', '=', False), ('sale_order_id', '=', False), ('partner_id', '=', False), ('sale_line_id', '=', False), ('remaining_hours_available', '=', False)]}"></field>
                </xpath>
            </field>
        </record>

        <record id="view_task_form2_inherit_sale_timesheet" model="ir.ui.view">
            <field name="name">view.task.form2.inherit</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project.view_task_form2"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='sale_line_id']" position="attributes">
                    <attribute name="context">{'with_remaining_hours': True}</attribute>
                    <attribute name="attrs">
                        {'invisible': ['|', ('allow_billable', '=', False), ('partner_id', '=', False)]}
                    </attribute>
                </xpath>
                <xpath expr="//field[@name='sale_order_id']" position="attributes">
                    <attribute name="invisible">1</attribute>
                </xpath>
            </field>
        </record>

        <record id="quick_create_task_form_sale_timesheet" model="ir.ui.view">
            <field name="name">project.task.form.inherit.timesheet</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project.quick_create_task_form"/>
            <field name="arch" type="xml">
                <field name="project_id" position="after">
                    <field name="timesheet_product_id" invisible="1"/>
                </field>
            </field>
        </record>

        <record id="project_timesheet_action_client_timesheet_plan" model="ir.actions.act_window">
            <field name="name">Overview</field>
            <field name="res_model">project.project</field>
            <field name="view_mode">qweb</field>
        </record>

</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="report_invoice_document" inherit_id="account.report_invoice_document">
            <xpath expr="//div[hasclass('page')]/h2" position="after">
                <a t-if="report_type == 'html' and o.move_type == 'out_invoice' and o.state in ('draft', 'posted') and o.timesheet_count > 0" target="_blank" t-att-href="'/my/timesheets?search_in=invoice_id&amp;search=%s' % o.id">View Timesheets</a>
            </xpath>
        </template>
    </data>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.sale.timesheet</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="1"/>
        <field name="inherit_id" ref="hr_timesheet.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='section_leaves']" position="before">
                <h2>Billing</h2>
                <div name="timesheet_billing" class="row mt16 o_settings_container">
                    <div class="col-12 col-lg-6 o_setting_box" id="time_billing_setting">
                        <div class="o_setting_right_pane">
                            <span class="o_form_label">Time Billing</span>
                            <div class="text-muted">
                                Sell services and invoice time spent
                            </div>
                            <div class="content-group" name="msg_module_sale_timesheet">
                                <div class="mt8">
                                    <div>
                                        <button name="%(sale_timesheet.product_template_action_default_services)d" string="Configure your services" type="action" class="btn-link" icon="fa-arrow-right"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="view_order_form_inherit_sale_timesheet" model="ir.ui.view">
            <field name="name">sale.order.form.sale.timesheet</field>
            <field name="model">sale.order</field>
            <field name="inherit_id" ref="sale_project.view_order_form_inherit_sale_project"/>
            <field name="arch" type="xml">
                <data>
                    <xpath expr="//button[@name='action_view_project_ids']" position="attributes">
                        <attribute name="string">Project Overview</attribute>
                    </xpath>
                    <xpath expr="//button[@name='action_view_invoice']" position="before">
                        <field name="timesheet_count" invisible="1" />
                        <button type="object"
                           name="action_view_timesheet"
                           class="oe_stat_button"
                           icon="fa-clock-o"
                           attrs="{'invisible': [('timesheet_count', '=', 0)]}"
                           groups="hr_timesheet.group_hr_timesheet_user">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value">
                                    <field name="timesheet_total_duration" class="mr4" widget="statinfo" nolabel="1"/>
                                    <field name="timesheet_encode_uom_id" options="{'no_open' : True}"/>
                                </span>
                                <span class="o_stat_text">Recorded</span>
                            </div>
                        </button>
                    </xpath>
                </data>
           </field>
        </record>
</odoo>

```

## File: views\sale_timesheet_portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="assets_frontend" name="report timesheet assets" inherit_id="web.assets_frontend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/sale_timesheet/static/src/scss/sale_timesheet_portal.scss"/>
        </xpath>
    </template>

    <!-- TODO: [XBO] Remove me in master -->
    <template id="portal_invoice_page_inherit_timesheet" inherit_id="account.portal_invoice_page">
        <xpath expr="//t[@t-call='portal.portal_record_sidebar']//div[hasclass('o_download_pdf')]" position="after">
            <t t-if="1 == 0">
                <li t-if="timesheets" class="list-group-item flex-grow-1" >
                    <a href="#accordion">Timesheets</a>
                </li>
            </t>
        </xpath>

        <xpath expr="//div[@id='invoice_content']//div[hasclass('o_portal_html_view')]" position="after">
            <t t-if="1 == 0">
                <div t-if="timesheets" class="container">
                    <div id="accordion" class="o_timesheet_accordion mt-4">
                        <div class="card mb-0">
                            <div class="card-header">
                                <h5 class="mb0">
                                    <a class="card-title" data-toggle="collapse" href="#collapseTimesheet">
                                        Timesheets
                                    </a>
                                </h5>
                            </div>
                            <div id="collapseTimesheet" class="card-body show" data-parent="#accordion">
                                <t t-set="nr_tasks" t-value="len(timesheets.mapped('task_id'))"/>
                                <t t-set="nr_projects" t-value="len(timesheets.mapped('project_id'))"/>
                                <table class="table table-sm">
                                    <thead>
                                    <tr>
                                        <th>Date</th>
                                        <th>Employee</th>
                                        <th t-if="nr_projects &gt; 1">Project</th>
                                        <th t-if="nr_tasks &gt; 0">Task</th>
                                        <th>Description</th>
                                        <th t-if="timesheets[0]._is_timesheet_encode_uom_day()" class="text-right">Duration (days)</th>
                                        <th t-else="" class="text-right">Duration (hours)</th>
                                    </tr>
                                    </thead>
                                    <tr t-foreach="timesheets" t-as="timesheet">
                                        <td><t t-esc="timesheet.date" t-options='{"widget": "date"}'/></td>
                                        <td><t t-esc="timesheet.employee_id.name"/></td>
                                        <td t-if="nr_projects &gt; 1"><span t-field="timesheet.project_id"/></td>
                                        <td t-if="nr_tasks &gt; 0"><span t-field="timesheet.task_id"/></td>
                                        <td><t t-esc="timesheet.name"/></td>
                                        <td class="text-right">
                                            <span t-if="timesheet._is_timesheet_encode_uom_day()" t-esc="timesheet._get_timesheet_time_day()" t-options='{"widget": "timesheet_uom"}'/>
                                            <span t-else="" t-field="timesheet.unit_amount" t-options='{"widget": "float_time"}'/>
                                        </td>
                                    </tr>
                                </table>
                            </div>
                        </div>
                    </div>
                </div>
            </t>
        </xpath>
    </template>

    <template id="sale_order_portal_content_inherit" inherit_id="sale.sale_order_portal_content">
        <xpath expr="//td[@id='product_name']" position="inside">
            <a t-if="timesheets and len(timesheets.filtered(lambda t: t.so_line == line)) > 0" t-att-href="'/my/timesheets?search_in=sol_id&amp;search=%s' % line.id">View Timesheets</a>
        </xpath>
    </template>

    <!-- TODO: [XBO] Remove me in master -->
    <template id="sale_order_portal_template_inherit" inherit_id="sale.sale_order_portal_template">
        <xpath expr="//t[@t-call='portal.portal_record_sidebar']//div[hasclass('o_download_pdf')]" position="after">
            <t t-if="1 == 0">
                <li t-if="timesheets" class="list-group-item flex-grow-1" >
                    <a href="#accordion">Timesheets</a>
                </li>
            </t>
        </xpath>

        <xpath expr="//div[@id='sale_order_communication']" position="before">
            <t t-if="1 == 0">
                <div t-if="timesheets" class="container">
                    <div id="accordion" class="o_timesheet_accordion mt-4">
                        <div class="card mb-0">
                            <div class="card-header">
                                <h5 class="mb0">
                                    <a class="card-title" data-toggle="collapse" href="#collapseTimesheet">
                                        Timesheets
                                    </a>
                                </h5>
                            </div>
                            <div id="collapseTimesheet" class="card-body show" data-parent="#accordion">
                                <t t-set="nr_tasks" t-value="len(timesheets.mapped('task_id'))"/>
                                <t t-set="nr_projects" t-value="len(timesheets.mapped('project_id'))"/>
                                <table class="table table-sm">
                                    <thead>
                                        <tr>
                                            <th>Date</th>
                                            <th>Employee</th>
                                            <th t-if="nr_projects &gt; 1">Project</th>
                                            <th t-if="nr_tasks &gt; 0">Task</th>
                                            <th>Description</th>
                                            <th t-if="timesheets[0]._is_timesheet_encode_uom_day()" class="text-right">Duration (days)</th>
                                            <th t-else="" class="text-right">Duration (hours)</th>
                                        </tr>
                                    </thead>
                                    <tr t-foreach="timesheets" t-as="timesheet">
                                        <td><t t-esc="timesheet.date" t-options='{"widget": "date"}'/></td>
                                        <td><t t-esc="timesheet.employee_id.name"/></td>
                                        <td t-if="nr_projects &gt; 1"><span t-field="timesheet.project_id"/></td>
                                        <td t-if="nr_tasks &gt; 0"><span t-field="timesheet.task_id"/></td>
                                        <td><t t-esc="timesheet.name"/></td>
                                        <td class="text-right">
                                            <span t-if="timesheet._is_timesheet_encode_uom_day()" t-esc="timesheet._get_timesheet_time_day()" t-options='{"widget": "timesheet_uom"}'/>
                                            <span t-else="" t-field="timesheet.unit_amount" t-options='{"widget": "float_time"}'/>
                                        </td>
                                    </tr>
                                </table>
                            </div>
                        </div>
                    </div>
                </div>
            </t>
        </xpath>
    </template>

    <template id="portal_my_timesheets_inherit" inherit_id="hr_timesheet.portal_my_timesheets">
        <xpath expr="//thead/tr[contains(@t-attf-class, 'thead-light')]" position="inside">
            <t t-elif="groupby == 'sol'">
                <t t-set="sol" t-value="timesheets[0].so_line"/>
                <th colspan="5">
                    <t t-if="sol">
                        <em class="font-weight-normal text-muted">Timesheets for sales order item:</em>
                        <span t-field="sol.display_name"/>
                        <t t-if="sol.remaining_hours_available">
                            <span class="text-muted font-weight-normal">(<span t-field="sol.product_uom_qty" t-options='{"widget": "float_time"}'></span> <span t-field="sol.product_uom.display_name"></span> Ordered, <span t-field="sol.remaining_hours" t-options='{"widget": "float_time"}'></span> <span t-field="sol.product_uom.display_name"></span> Remaining)</span>
                        </t>
                    </t>
                </th>
                <th colspan="1" class="text-right text-muted font-weight-normal">
                    <t t-if="is_uom_day">
                        Total: <span t-esc="timesheets[0]._convert_hours_to_days(hours_spent)" t-options='{"widget": "timesheet_uom"}'/>
                    </t>
                    <t t-else="">
                        Total: <span t-esc="hours_spent" t-options='{"widget": "float_time"}'/>
                    </t>
                </th>
            </t>
        </xpath>
        <xpath expr="//thead/tr/th[@t-if='is_uom_day']" position="before">
            <th t-if="not groupby == 'sol'">Sale Order Item</th>
        </xpath>
        <xpath expr="//tbody//td[hasclass('text-right')]" position="before">
            <td t-if="not groupby == 'sol'"><span t-field="timesheet.so_line" t-att-title="timesheet.so_line.display_name"></span></td>
        </xpath>
    </template>

</odoo>

```

## File: wizard\project_create_invoice.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class ProjectCreateInvoice(models.TransientModel):
    _name = 'project.create.invoice'
    _description = "Create Invoice from project"

    @api.model
    def default_get(self, fields):
        result = super(ProjectCreateInvoice, self).default_get(fields)

        active_model = self._context.get('active_model')
        if active_model != 'project.project':
            raise UserError(_('You can only apply this action from a project.'))

        active_id = self._context.get('active_id')
        if 'project_id' in fields and active_id:
            result['project_id'] = active_id
        return result

    project_id = fields.Many2one('project.project', "Project", help="Project to make billable", required=True)
    _candidate_orders = fields.Many2many('sale.order', compute='_compute_candidate_orders')
    sale_order_id = fields.Many2one(
        'sale.order', string="Choose the Sales Order to invoice", required=True,
        domain="[('id', 'in', _candidate_orders)]"
    )
    amount_to_invoice = fields.Monetary("Amount to invoice", compute='_compute_amount_to_invoice', currency_field='currency_id', help="Total amount to invoice on the sales order, including all items (services, storables, expenses, ...)")
    currency_id = fields.Many2one(related='sale_order_id.currency_id', readonly=True)

    @api.depends('project_id.tasks.sale_line_id.order_id.invoice_status')
    def _compute_candidate_orders(self):
        for p in self:
            p._candidate_orders = p.project_id\
                .mapped('tasks.sale_line_id.order_id')\
                .filtered(lambda so: so.invoice_status == 'to invoice')

    @api.depends('sale_order_id')
    def _compute_amount_to_invoice(self):
        for wizard in self:
            amount_untaxed = 0.0
            amount_tax = 0.0
            for line in wizard.sale_order_id.order_line.filtered(lambda sol: sol.invoice_status == 'to invoice'):
                amount_untaxed += line.price_reduce * line.qty_to_invoice
                amount_tax += line.price_tax
            wizard.amount_to_invoice = amount_untaxed + amount_tax

    def action_create_invoice(self):
        if not self.sale_order_id and self.sale_order_id.invoice_status != 'to invoice':
            raise UserError(_("The selected Sales Order should contain something to invoice."))
        action = self.env["ir.actions.actions"]._for_xml_id("sale.action_view_sale_advance_payment_inv")
        action['context'] = {
            'active_ids': self.sale_order_id.ids
        }
        return action

```

## File: wizard\project_create_invoice_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="project_create_invoice_view_form" model="ir.ui.view">
        <field name="name">project.create.invoice.view.form</field>
        <field name="model">project.create.invoice</field>
        <field name="arch" type="xml">
            <form string="Create Sales Order from Project">
                <group>
                    <field name="project_id" readonly="1"/>
                    <field name="_candidate_orders" invisible="1"/>
                    <field name="sale_order_id" options="{'no_create_edit': True}" context="{'sale_show_partner_name': True}"/>
                    <field name="amount_to_invoice"/>
                </group>
                <footer>
                    <button string="Create Invoice" type="object" name="action_create_invoice" class="oe_highlight"/>
                    <button string="Cancel" special="cancel" type="object" class="btn btn-secondary oe_inline"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="project_project_action_multi_create_invoice" model="ir.actions.act_window">
        <field name="name">Create Invoice</field>
        <field name="res_model">project.create.invoice</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="project_create_invoice_view_form"/>
        <field name="target">new</field>
    </record>

</odoo>

```

## File: wizard\project_create_sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class ProjectCreateSalesOrder(models.TransientModel):
    _name = 'project.create.sale.order'
    _description = "Create SO from project"

    @api.model
    def default_get(self, fields):
        result = super(ProjectCreateSalesOrder, self).default_get(fields)

        active_model = self._context.get('active_model')
        if active_model != 'project.project':
            raise UserError(_("You can only apply this action from a project."))

        active_id = self._context.get('active_id')
        if 'project_id' in fields and active_id:
            project = self.env['project.project'].browse(active_id)
            if project.sale_order_id:
                raise UserError(_("The project has already a sale order."))
            result['project_id'] = active_id
            if not result.get('partner_id', False):
                result['partner_id'] = project.partner_id.id
            if project.bill_type == 'customer_project' and not result.get('line_ids', False):
                if project.pricing_type == 'employee_rate':
                    default_product = self.env.ref('sale_timesheet.time_product', False)
                    result['line_ids'] = [
                        (0, 0, {
                            'employee_id': e.employee_id.id,
                            'product_id': e.timesheet_product_id.id or default_product.id,
                            'price_unit': e.price_unit if e.timesheet_product_id else default_product.lst_price
                        }) for e in project.sale_line_employee_ids]
                    employee_from_timesheet = project.task_ids.timesheet_ids.employee_id - project.sale_line_employee_ids.employee_id
                    result['line_ids'] += [
                        (0, 0, {
                            'employee_id': e.id,
                            'product_id': default_product.id,
                            'price_unit': default_product.lst_price
                        }) for e in employee_from_timesheet]
                else:
                    result['line_ids'] = [
                        (0, 0, {
                            'product_id': p.id,
                            'price_unit': p.lst_price
                        }) for p in project.task_ids.timesheet_product_id]
        return result

    project_id = fields.Many2one('project.project', "Project", domain=[('sale_line_id', '=', False)], help="Project for which we are creating a sales order", required=True)
    company_id = fields.Many2one(related='project_id.company_id')
    partner_id = fields.Many2one('res.partner', string="Customer", required=True, help="Customer of the sales order")
    commercial_partner_id = fields.Many2one(related='partner_id.commercial_partner_id')

    pricing_type = fields.Selection(related="project_id.pricing_type")
    link_selection = fields.Selection([('create', 'Create a new sales order'), ('link', 'Link to an existing sales order')], required=True, default='create')
    sale_order_id = fields.Many2one(
        'sale.order', string="Sales Order",
        domain="['|', '|', ('partner_id', '=', partner_id), ('partner_id', 'child_of', commercial_partner_id), ('partner_id', 'parent_of', partner_id)]")

    line_ids = fields.One2many('project.create.sale.order.line', 'wizard_id', string='Lines')
    info_invoice = fields.Char(compute='_compute_info_invoice')

    @api.depends('sale_order_id', 'link_selection')
    def _compute_info_invoice(self):
        for line in self:
            tasks = line.project_id.tasks.filtered(lambda t: not t.non_allow_billable)
            domain = self.env['sale.order.line']._timesheet_compute_delivered_quantity_domain()
            timesheet = self.env['account.analytic.line'].read_group(domain + [('task_id', 'in', tasks.ids), ('so_line', '=', False), ('timesheet_invoice_id', '=', False)], ['unit_amount'], ['task_id'])
            unit_amount = round(sum(t.get('unit_amount', 0) for t in timesheet), 2) if timesheet else 0
            if not unit_amount:
                line.info_invoice = False
                continue
            company_uom = self.env.company.timesheet_encode_uom_id
            label = _("hours")
            if company_uom == self.env.ref('uom.product_uom_day'):
                label = _("days")
            if line.link_selection == 'create':
                line.info_invoice = _("%(amount)s %(label)s will be added to the new Sales Order.", amount=unit_amount, label=label)
            else:
                line.info_invoice = _("%(amount)s %(label)s will be added to the selected Sales Order.", amount=unit_amount, label=label)

    @api.onchange('partner_id')
    def _onchange_partner_id(self):
        self.sale_order_id = False

    def action_link_sale_order(self):
        task_no_sale_line = self.project_id.tasks.filtered(lambda task: not task.sale_line_id)
        # link the project to the SO line
        self.project_id.write({
            'sale_line_id': self.sale_order_id.order_line[0].id,
            'sale_order_id': self.sale_order_id.id,
            'partner_id': self.partner_id.id,
        })

        if self.pricing_type == 'employee_rate':
            lines_already_present = dict([(l.employee_id.id, l) for l in self.project_id.sale_line_employee_ids])
            EmployeeMap = self.env['project.sale.line.employee.map'].sudo()

            for wizard_line in self.line_ids:
                if wizard_line.employee_id.id not in lines_already_present:
                    EmployeeMap.create({
                        'project_id': self.project_id.id,
                        'sale_line_id': wizard_line.sale_line_id.id,
                        'employee_id': wizard_line.employee_id.id,
                    })
                else:
                    lines_already_present[wizard_line.employee_id.id].write({
                        'sale_line_id': wizard_line.sale_line_id.id
                    })

            self.project_id.tasks.filtered(lambda task: task.non_allow_billable).sale_line_id = False
            tasks = self.project_id.tasks.filtered(lambda t: not t.non_allow_billable)
            # assign SOL to timesheets
            for map_entry in self.project_id.sale_line_employee_ids:
                self.env['account.analytic.line'].search([('task_id', 'in', tasks.ids), ('employee_id', '=', map_entry.employee_id.id), ('so_line', '=', False)]).write({
                    'so_line': map_entry.sale_line_id.id
                })
        else:
            dict_product_sol = dict([(l.product_id.id, l.id) for l in self.sale_order_id.order_line])
            # remove SOL for task without product
            # and if a task has a product that match a product from a SOL, we put this SOL on task.
            for task in task_no_sale_line:
                if not task.timesheet_product_id:
                    task.sale_line_id = False
                elif task.timesheet_product_id.id in dict_product_sol:
                    task.write({'sale_line_id': dict_product_sol[task.timesheet_product_id.id]})

    def action_create_sale_order(self):
        # if project linked to SO line or at least on tasks with SO line, then we consider project as billable.
        if self.project_id.sale_line_id:
            raise UserError(_("The project is already linked to a sales order item."))
        # at least one line
        if not self.line_ids:
            raise UserError(_("At least one line should be filled."))

        if self.pricing_type == 'employee_rate':
            # all employee having timesheet should be in the wizard map
            timesheet_employees = self.env['account.analytic.line'].search([('task_id', 'in', self.project_id.tasks.ids)]).mapped('employee_id')
            map_employees = self.line_ids.mapped('employee_id')
            missing_meployees = timesheet_employees - map_employees
            if missing_meployees:
                raise UserError(_('The Sales Order cannot be created because you did not enter some employees that entered timesheets on this project. Please list all the relevant employees before creating the Sales Order.\nMissing employee(s): %s') % (', '.join(missing_meployees.mapped('name'))))

        # check here if timesheet already linked to SO line
        timesheet_with_so_line = self.env['account.analytic.line'].search_count([('task_id', 'in', self.project_id.tasks.ids), ('so_line', '!=', False)])
        if timesheet_with_so_line:
            raise UserError(_('The sales order cannot be created because some timesheets of this project are already linked to another sales order.'))

        # create SO according to the chosen billable type
        sale_order = self._create_sale_order()

        view_form_id = self.env.ref('sale.view_order_form').id
        action = self.env["ir.actions.actions"]._for_xml_id("sale.action_orders")
        action.update({
            'views': [(view_form_id, 'form')],
            'view_mode': 'form',
            'name': sale_order.name,
            'res_id': sale_order.id,
        })
        return action

    def _create_sale_order(self):
        """ Private implementation of generating the sales order """
        sale_order = self.env['sale.order'].create({
            'project_id': self.project_id.id,
            'partner_id': self.partner_id.id,
            'analytic_account_id': self.project_id.analytic_account_id.id,
            'client_order_ref': self.project_id.name,
            'company_id': self.project_id.company_id.id,
        })
        sale_order.onchange_partner_id()
        sale_order.onchange_partner_shipping_id()
        # rewrite the user as the onchange_partner_id erases it
        sale_order.write({'user_id': self.project_id.user_id.id})
        sale_order.onchange_user_id()

        # create the sale lines, the map (optional), and assign existing timesheet to sale lines
        self._make_billable(sale_order)

        # confirm SO
        sale_order.action_confirm()
        return sale_order

    def _make_billable(self, sale_order):
        if self.pricing_type == 'fixed_rate':
            self._make_billable_at_project_rate(sale_order)
        else:
            self._make_billable_at_employee_rate(sale_order)

    def _make_billable_at_project_rate(self, sale_order):
        self.ensure_one()
        task_left = self.project_id.tasks.filtered(lambda task: not task.sale_line_id)
        ticket_timesheet_ids = self.env.context.get('ticket_timesheet_ids', [])
        for wizard_line in self.line_ids:
            task_ids = self.project_id.tasks.filtered(lambda task: not task.sale_line_id and task.timesheet_product_id == wizard_line.product_id)
            task_left -= task_ids
            # trying to simulate the SO line created a task, according to the product configuration
            # To avoid, generating a task when confirming the SO
            task_id = False
            if task_ids and wizard_line.product_id.service_tracking in ['task_in_project', 'task_global_project']:
                task_id = task_ids.ids[0]

            # create SO line
            sale_order_line = self.env['sale.order.line'].create({
                'order_id': sale_order.id,
                'product_id': wizard_line.product_id.id,
                'price_unit': wizard_line.price_unit,
                'project_id': self.project_id.id,  # prevent to re-create a project on confirmation
                'task_id': task_id,
                'product_uom_qty': 0.0,
            })

            if ticket_timesheet_ids and not self.project_id.sale_line_id and not task_ids:
                # With pricing = "project rate" in project. When the user wants to create a sale order from a ticket in helpdesk
                # The project cannot contain any tasks. Thus, we need to give the first sale_order_line created to link
                # the timesheet to this first sale order line.
                # link the project to the SO line
                self.project_id.write({
                    'sale_order_id': sale_order.id,
                    'sale_line_id': sale_order_line.id,
                    'partner_id': self.partner_id.id,
                })

            # link the tasks to the SO line
            task_ids.write({
                'sale_line_id': sale_order_line.id,
                'partner_id': sale_order.partner_id.id,
                'email_from': sale_order.partner_id.email,
            })

            # assign SOL to timesheets
            search_domain = [('task_id', 'in', task_ids.ids), ('so_line', '=', False)]
            if ticket_timesheet_ids:
                search_domain = [('id', 'in', ticket_timesheet_ids), ('so_line', '=', False)]

            self.env['account.analytic.line'].search(search_domain).write({
                'so_line': sale_order_line.id
            })
            sale_order_line.with_context({'no_update_planned_hours': True}).write({
                'product_uom_qty': sale_order_line.qty_delivered
            })

        if ticket_timesheet_ids and self.project_id.sale_line_id and not self.project_id.tasks and len(self.line_ids) > 1:
            # Then, we need to give to the project the last sale order line created
            self.project_id.write({
                'sale_line_id': sale_order_line.id
            })
        else:  # Otherwise, we are in the normal behaviour
            # link the project to the SO line
            self.project_id.write({
                'sale_order_id': sale_order.id,
                'sale_line_id': sale_order_line.id,  # we take the last sale_order_line created
                'partner_id': self.partner_id.id,
            })

        if task_left:
            task_left.sale_line_id = False

    def _make_billable_at_employee_rate(self, sale_order):
        # trying to simulate the SO line created a task, according to the product configuration
        # To avoid, generating a task when confirming the SO
        task_id = self.env['project.task'].search([('project_id', '=', self.project_id.id)], order='create_date DESC', limit=1).id
        project_id = self.project_id.id

        lines_already_present = dict([(l.employee_id.id, l) for l in self.project_id.sale_line_employee_ids])

        non_billable_tasks = self.project_id.tasks.filtered(lambda task: not task.sale_line_id)
        non_allow_billable_tasks = self.project_id.tasks.filtered(lambda task: task.non_allow_billable)

        map_entries = self.env['project.sale.line.employee.map']
        EmployeeMap = self.env['project.sale.line.employee.map'].sudo()

        # create SO lines: create on SOL per product/price. So many employee can be linked to the same SOL
        map_product_price_sol = {}  # (product_id, price) --> SOL
        for wizard_line in self.line_ids:
            map_key = (wizard_line.product_id.id, wizard_line.price_unit)
            if map_key not in map_product_price_sol:
                values = {
                    'order_id': sale_order.id,
                    'product_id': wizard_line.product_id.id,
                    'price_unit': wizard_line.price_unit,
                    'product_uom_qty': 0.0,
                }
                if wizard_line.product_id.service_tracking in ['task_in_project', 'task_global_project']:
                    values['task_id'] = task_id
                if wizard_line.product_id.service_tracking in ['task_in_project', 'project_only']:
                    values['project_id'] = project_id

                sale_order_line = self.env['sale.order.line'].create(values)
                map_product_price_sol[map_key] = sale_order_line

            if wizard_line.employee_id.id not in lines_already_present:
                map_entries |= EmployeeMap.create({
                    'project_id': self.project_id.id,
                    'sale_line_id': map_product_price_sol[map_key].id,
                    'employee_id': wizard_line.employee_id.id,
                })
            else:
                map_entries |= lines_already_present[wizard_line.employee_id.id]
                lines_already_present[wizard_line.employee_id.id].write({
                    'sale_line_id': map_product_price_sol[map_key].id
                })

        # link the project to the SO
        self.project_id.write({
            'sale_order_id': sale_order.id,
            'sale_line_id': sale_order.order_line[0].id,
            'partner_id': self.partner_id.id,
        })
        non_billable_tasks.write({
            'partner_id': sale_order.partner_id.id,
            'email_from': sale_order.partner_id.email,
        })
        non_allow_billable_tasks.sale_line_id = False

        tasks = self.project_id.tasks.filtered(lambda t: not t.non_allow_billable)
        # assign SOL to timesheets
        for map_entry in map_entries:
            search_domain = [('employee_id', '=', map_entry.employee_id.id), ('so_line', '=', False)]
            ticket_timesheet_ids = self.env.context.get('ticket_timesheet_ids', [])
            if ticket_timesheet_ids:
                search_domain.append(('id', 'in', ticket_timesheet_ids))
            else:
                search_domain.append(('task_id', 'in', tasks.ids))
            self.env['account.analytic.line'].search(search_domain).write({
                'so_line': map_entry.sale_line_id.id
            })
            map_entry.sale_line_id.with_context({'no_update_planned_hours': True}).write({
                'product_uom_qty': map_entry.sale_line_id.qty_delivered
            })

        return map_entries


class ProjectCreateSalesOrderLine(models.TransientModel):
    _name = 'project.create.sale.order.line'
    _description = 'Create SO Line from project'
    _order = 'id,create_date'

    wizard_id = fields.Many2one('project.create.sale.order', required=True)
    product_id = fields.Many2one('product.product', domain=[('type', '=', 'service'), ('invoice_policy', '=', 'delivery'), ('service_type', '=', 'timesheet')], string="Service",
        help="Product of the sales order item. Must be a service invoiced based on timesheets on tasks.")
    price_unit = fields.Float("Unit Price", help="Unit price of the sales order item.")
    currency_id = fields.Many2one('res.currency', string="Currency")
    employee_id = fields.Many2one('hr.employee', string="Employee", help="Employee that has timesheets on the project.")
    sale_line_id = fields.Many2one('sale.order.line', "Sale Order Item", compute='_compute_sale_line_id', store=True, readonly=False)

    _sql_constraints = [
        ('unique_employee_per_wizard', 'UNIQUE(wizard_id, employee_id)', "An employee cannot be selected more than once in the mapping. Please remove duplicate(s) and try again."),
    ]

    @api.onchange('product_id', 'sale_line_id')
    def _onchange_product_id(self):
        if self.wizard_id.link_selection == 'link':
            self.price_unit = self.sale_line_id.price_unit
            self.currency_id = self.sale_line_id.currency_id
        else:
            self.price_unit = self.product_id.lst_price or 0
            self.currency_id = self.product_id.currency_id

    @api.depends('wizard_id.sale_order_id')
    def _compute_sale_line_id(self):
        for line in self:
            if line.sale_line_id and line.sale_line_id.order_id != line.wizard_id.sale_order_id:
                line.sale_line_id = False

```

## File: wizard\project_create_sale_order_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="project_create_sale_order_view_form" model="ir.ui.view">
        <field name="name">project.create.sale.order.wizard.form</field>
        <field name="model">project.create.sale.order</field>
        <field name="arch" type="xml">
            <form string="Create a Sales Order">
                <field name="link_selection" nolabel="1" widget="radio" options="{'horizontal': true}" invisible="1"/>
                <group>
                    <group>
                        <field name="project_id" readonly="1"/>
                        <field name="company_id" invisible="1"/>
                        <field name="partner_id" domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]"/>
                        <field name="pricing_type" invisible="1"/>
                    </group>
                    <group attrs="{'invisible': [('link_selection', '!=', 'link')]}">
                        <field name="sale_order_id" options="{'no_create': True, 'no_create_edit': True}" attrs="{'required': [('link_selection', '=', 'link')]}"/>
                        <field name="commercial_partner_id" invisible="1"/>
                    </group>
                </group>
                <group attrs="{'invisible': ['&amp;', ('link_selection', '=', 'link'), ('pricing_type', '!=', 'employee_rate')]}">
                    <field name="line_ids" nolabel="1" attrs="{'required': [('pricing_type', '=', 'employee_rate')]}">
                        <tree editable="bottom">
                            <field name="employee_id" options="{'no_create_edit': True, 'no_create': True}" attrs="{'column_invisible': [('parent.pricing_type', '=', 'fixed_rate')], 'required': [('parent.pricing_type', '=', 'employee_rate')]}"/>
                            <field name="sale_line_id" options="{'no_create_edit': True, 'no_create': True, 'no_open': True}" domain="[('is_service', '=', True), ('order_id', '=', parent.sale_order_id)]" attrs="{'column_invisible': ['|', ('parent.link_selection', '!=', 'link'), ('parent.pricing_type', '=', 'fixed_rate')], 'required': ['&amp;', ('parent.link_selection', '=', 'link'), ('parent.pricing_type', '=', 'employee_rate')]}"/>
                            <field name="product_id" options="{'no_create_edit': True, 'no_create': True}" attrs="{'column_invisible': [('parent.link_selection', '=', 'link')], 'required': [('parent.link_selection', '!=', 'link')]}"/>
                            <field name="price_unit" widget='monetary' options="{'currency_field': 'currency_id', 'field_digits': True}" attrs="{'readonly': [('parent.link_selection', '=', 'link')]}"/>
                            <field name="currency_id" invisible="1"/>
                        </tree>
                    </field>
                </group>
                <group attrs="{'invisible': [('info_invoice', '=', False)]}">
                    <field name="info_invoice" nolabel="1"/>
                </group>
                <footer>
                    <button string="Link to Sales Order" type="object" name="action_link_sale_order" class="oe_highlight" attrs="{'invisible': [('link_selection', '!=', 'link')]}"/>
                    <button string="Create Sales Order" type="object" name="action_create_sale_order" class="oe_highlight" attrs="{'invisible': [('link_selection', '=', 'link')]}"/>
                    <button string="Cancel" special="cancel" type="object" class="btn btn-secondary oe_inline"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="project_project_action_multi_create_sale_order" model="ir.actions.act_window">
        <field name="name">Create a Sales Order</field>
        <field name="res_model">project.create.sale.order</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="project_create_sale_order_view_form"/>
        <field name="target">new</field>
    </record>

</odoo>

```

## File: wizard\project_task_create_sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class ProjectTaskCreateSalesOrder(models.TransientModel):
    _name = 'project.task.create.sale.order'
    _description = "Create SO from task"

    @api.model
    def default_get(self, fields):
        result = super(ProjectTaskCreateSalesOrder, self).default_get(fields)

        active_model = self._context.get('active_model')
        if active_model != 'project.task':
            raise UserError(_("You can only apply this action from a task."))

        active_id = self._context.get('active_id')
        if 'task_id' in fields and active_id:
            task = self.env['project.task'].browse(active_id)
            if task.sale_order_id:
                raise UserError(_("The task has already a sale order."))
            result['task_id'] = active_id
            if not result.get('partner_id', False):
                result['partner_id'] = task.partner_id.id
        return result

    link_selection = fields.Selection([('create', 'Create a new sales order'), ('link', 'Link to an existing sales order')], required=True, default='create')

    task_id = fields.Many2one('project.task', "Task", domain=[('sale_line_id', '=', False)], help="Task for which we are creating a sales order", required=True)
    partner_id = fields.Many2one('res.partner', string="Customer", help="Customer of the sales order", required=True)
    product_id = fields.Many2one('product.product', domain=[('type', '=', 'service'), ('invoice_policy', '=', 'delivery'), ('service_type', '=', 'timesheet')], string="Service", help="Product of the sales order item. Must be a service invoiced based on timesheets on tasks. The existing timesheet will be linked to this product.", required=True)
    price_unit = fields.Float("Unit Price", help="Unit price of the sales order item.")
    currency_id = fields.Many2one('res.currency', string="Currency", related='product_id.currency_id', readonly=False)
    commercial_partner_id = fields.Many2one(related='partner_id.commercial_partner_id')
    sale_order_id = fields.Many2one(
        'sale.order', string="Sales Order",
        domain="['|', '|', ('partner_id', '=', partner_id), ('partner_id', 'child_of', commercial_partner_id), ('partner_id', 'parent_of', partner_id)]")
    sale_line_id = fields.Many2one(
        'sale.order.line', 'Sales Order Item',
        domain="[('is_service', '=', True), ('order_partner_id', 'child_of', commercial_partner_id), ('is_expense', '=', False), ('state', 'in', ['sale', 'done']), ('order_id', '=?', sale_order_id)]")

    info_invoice = fields.Char(compute='_compute_info_invoice')

    @api.depends('sale_line_id', 'price_unit', 'link_selection')
    def _compute_info_invoice(self):
        for line in self:
            domain = self.env['sale.order.line']._timesheet_compute_delivered_quantity_domain()
            timesheet = self.env['account.analytic.line'].read_group(domain + [('task_id', '=', self.task_id.id), ('so_line', '=', False), ('timesheet_invoice_id', '=', False)], ['unit_amount'], ['task_id'])
            unit_amount = round(timesheet[0].get('unit_amount', 0), 2) if timesheet else 0
            if not unit_amount:
                line.info_invoice = False
                continue
            company_uom = self.env.company.timesheet_encode_uom_id
            label = _("hours")
            if company_uom == self.env.ref('uom.product_uom_day'):
                label = _("days")
            if line.link_selection == 'create' and line.price_unit:
                line.info_invoice = _("%(amount)s %(label)s will be added to the new Sales Order.", amount=unit_amount, label=label)
            else:
                line.info_invoice = _("%(amount)s %(label)s will be added to the selected Sales Order.", amount=unit_amount, label=label)

    @api.onchange('product_id')
    def _onchange_product_id(self):
        if self.product_id:
            self.price_unit = self.product_id.lst_price
        else:
            self.price_unit = 0.0

    @api.onchange('partner_id')
    def _onchange_partner_id(self):
        self.sale_order_id = False
        self.sale_line_id = False

    @api.onchange('sale_order_id')
    def _onchange_sale_order_id(self):
        self.sale_line_id = False

    def action_link_sale_order(self):
        # link task to SOL
        self.task_id.write({
            'sale_line_id': self.sale_line_id.id,
            'partner_id': self.partner_id.id,
            'email_from': self.partner_id.email,
        })

        # assign SOL to timesheets
        self.env['account.analytic.line'].search([('task_id', '=', self.task_id.id), ('so_line', '=', False), ('timesheet_invoice_id', '=', False)]).write({
            'so_line': self.sale_line_id.id
        })

    def action_create_sale_order(self):
        sale_order = self._prepare_sale_order()
        sale_order.action_confirm()
        view_form_id = self.env.ref('sale.view_order_form').id
        action = self.env["ir.actions.actions"]._for_xml_id("sale.action_orders")
        action.update({
            'views': [(view_form_id, 'form')],
            'view_mode': 'form',
            'name': sale_order.name,
            'res_id': sale_order.id,
        })
        return action

    def _prepare_sale_order(self):
        # if task linked to SO line, then we consider it as billable.
        if self.task_id.sale_line_id:
            raise UserError(_("The task is already linked to a sales order item."))

        # create SO
        sale_order = self.env['sale.order'].create({
            'partner_id': self.partner_id.id,
            'company_id': self.task_id.company_id.id,
            'analytic_account_id': self.task_id.project_id.analytic_account_id.id,
        })
        sale_order.onchange_partner_id()
        sale_order.onchange_partner_shipping_id()
        # rewrite the user as the onchange_partner_id erases it
        sale_order.write({'user_id': self.task_id.user_id.id})
        sale_order.onchange_user_id()

        sale_order_line = self.env['sale.order.line'].create({
            'order_id': sale_order.id,
            'product_id': self.product_id.id,
            'price_unit': self.price_unit,
            'project_id': self.task_id.project_id.id,  # prevent to re-create a project on confirmation
            'task_id': self.task_id.id,
            'product_uom_qty': round(sum(self.task_id.timesheet_ids.filtered(lambda t: not t.non_allow_billable and not t.so_line).mapped('unit_amount')), 2),
        })

        # link task to SOL
        self.task_id.write({
            'sale_line_id': sale_order_line.id,
            'partner_id': sale_order.partner_id.id,
            'email_from': sale_order.partner_id.email,
        })

        # assign SOL to timesheets
        self.env['account.analytic.line'].search([('task_id', '=', self.task_id.id), ('so_line', '=', False), ('timesheet_invoice_id', '=', False)]).write({
            'so_line': sale_order_line.id
        })

        return sale_order

```

## File: wizard\project_task_create_sale_order_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="project_task_create_sale_order_view_form" model="ir.ui.view">
        <field name="name">project.task.create.sale.order.wizard.form</field>
        <field name="model">project.task.create.sale.order</field>
        <field name="arch" type="xml">
            <form string="Create a Sales Order">
                <field name="link_selection" nolabel="1" widget="radio" options="{'horizontal': true}" invisible="1"/>
                <group>
                    <group>
                        <field name="task_id" readonly="1"/>
                        <field name="partner_id"/>
                    </group>
                    <group attrs="{'invisible': [('link_selection', '=', 'link')]}">
                        <field name="product_id" context="{'default_type': 'service', 'default_service_policy': 'delivered_timesheet'}" attrs="{'required': [('link_selection', '!=', 'link')]}"/>
                        <field name="price_unit" widget='monetary' options="{'currency_field': 'currency_id', 'field_digits': True}" attrs="{'required': [('link_selection', '!=', 'link')]}"/>
                        <field name="currency_id" invisible="1"/>
                    </group>
                    <group attrs="{'invisible': [('link_selection', '!=', 'link')]}">
                        <field name="sale_order_id" options="{'no_create': True, 'no_create_edit': True}" attrs="{'required': [('link_selection', '=', 'link')]}"/>
                        <field name="sale_line_id" options="{'no_create': True, 'no_create_edit': True}" attrs="{'required': [('link_selection', '=', 'link')]}"/>
                        <field name="commercial_partner_id" invisible="1"/>
                    </group>
                </group>
                <group attrs="{'invisible': [('info_invoice', '=', False)]}">
                    <field name="info_invoice" nolabel="1"/>
                </group>
                <footer>
                    <button string="Link to Sales Order" type="object" name="action_link_sale_order" class="oe_highlight" attrs="{'invisible': [('link_selection', '!=', 'link')]}"/>
                    <button string="Create Sales Order" type="object" name="action_create_sale_order" class="oe_highlight" attrs="{'invisible': [('link_selection', '=', 'link')]}"/>
                    <button string="Cancel" special="cancel" type="object" class="btn btn-secondary oe_inline"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="project_task_action_multi_create_sale_order" model="ir.actions.act_window">
        <field name="name">Create a Sales Order</field>
        <field name="res_model">project.task.create.sale.order</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="project_task_create_sale_order_view_form"/>
        <field name="target">new</field>
    </record>

</odoo>

```

## File: wizard\sale_make_invoice_advance.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SaleAdvancePaymentInv(models.TransientModel):
    _inherit = "sale.advance.payment.inv"

    @api.model
    def _default_invoicing_timesheet_enabled(self):
        if 'active_id' not in self._context and 'active_ids' not in self._context:
            return False
        sale_orders = self.env['sale.order'].browse(self._context.get('active_id') or self._context.get('active_ids'))
        order_lines = sale_orders.mapped('order_line').filtered(lambda sol: sol.invoice_status == 'to invoice')
        product_ids = order_lines.mapped('product_id').filtered(lambda p: p._is_delivered_timesheet())
        return bool(product_ids)

    date_start_invoice_timesheet = fields.Date(
        string='Start Date',
        help="Only timesheets not yet invoiced (and validated, if applicable) from this period will be invoiced. If the period is not indicated, all timesheets not yet invoiced (and validated, if applicable) will be invoiced without distinction.")
    date_end_invoice_timesheet = fields.Date(
        string='End Date',
        help="Only timesheets not yet invoiced (and validated, if applicable) from this period will be invoiced. If the period is not indicated, all timesheets not yet invoiced (and validated, if applicable) will be invoiced without distinction.")
    invoicing_timesheet_enabled = fields.Boolean(default=_default_invoicing_timesheet_enabled)

    def create_invoices(self):
        """ Override method from sale/wizard/sale_make_invoice_advance.py

            When the user want to invoice the timesheets to the SO
            up to a specific period then we need to recompute the
            qty_to_invoice for each product_id in sale.order.line,
            before creating the invoice.
        """
        sale_orders = self.env['sale.order'].browse(
            self._context.get('active_ids', [])
        )

        if self.advance_payment_method == 'delivered' and self.invoicing_timesheet_enabled:
            if self.date_start_invoice_timesheet or self.date_end_invoice_timesheet:
                sale_orders.mapped('order_line')._recompute_qty_to_invoice(self.date_start_invoice_timesheet, self.date_end_invoice_timesheet)

            sale_orders.with_context(
                timesheet_start_date=self.date_start_invoice_timesheet,
                timesheet_end_date=self.date_end_invoice_timesheet
            )._create_invoices(final=self.deduct_down_payments)

            if self._context.get('open_invoices', False):
                return sale_orders.action_view_invoice()
            return {'type': 'ir.actions.act_window_close'}

        return super(SaleAdvancePaymentInv, self).create_invoices()

```

## File: wizard\sale_make_invoice_advance_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="sale_advance_payment_inv_timesheet_view_form" model="ir.ui.view">
        <field name="name">sale_timesheet.sale.advance.payment.inv.view.form</field>
        <field name="model">sale.advance.payment.inv</field>
        <field name="inherit_id" ref="sale.view_sale_advance_payment_inv"/>
        <field name="arch" type="xml">
            <xpath expr="//form" position="attributes">
                <attribute name="disable_autofocus">true</attribute>
            </xpath>
            <xpath expr="//field[@name='deposit_taxes_id']" position="after">
                <field name="invoicing_timesheet_enabled" invisible="1"/>
                <label for="date_start_invoice_timesheet" string="Timesheets Period" attrs="{'invisible': [ '|', ('invoicing_timesheet_enabled', '=', False), ('advance_payment_method', '!=', 'delivered')]}"/>
                <div class="o_row" attrs="{'invisible': [ '|',('invoicing_timesheet_enabled', '=', False), ('advance_payment_method', '!=', 'delivered')]}">
                    <field name="date_start_invoice_timesheet"
                        class="oe_inline" widget="daterange"
                        options="{'related_end_date': 'date_end_invoice_timesheet'}"
                        title="Only timesheets not yet invoiced (and validated, if applicable) from this period will be invoiced. If the period is not indicated, all timesheets not yet invoiced (and validated, if applicable) will be invoiced without distinction."/>
                    <i class="fa fa-long-arrow-right mx-2" aria-label="Arrow icon" title="Arrow"/>
                    <field name="date_end_invoice_timesheet"
                        class="oe_inline" widget="daterange"
                        options="{'related_start_date': 'date_start_invoice_timesheet'}"
                        title="Only timesheets not yet invoiced (and validated, if applicable) from this period will be invoiced. If the period is not indicated, all timesheets not yet invoiced (and validated, if applicable) will be invoiced without distinction."/>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import project_create_sale_order
from . import project_create_invoice
from . import sale_make_invoice_advance
from . import project_task_create_sale_order

```

