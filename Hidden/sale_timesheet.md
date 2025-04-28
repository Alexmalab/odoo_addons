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
        'views/project_update_templates.xml',
        'views/hr_timesheet_views.xml',
        'views/res_config_settings_views.xml',
        'views/sale_timesheet_portal_templates.xml',
        'views/project_sharing_views.xml',
        'report/project_profitability_report_analysis_views.xml',
        'data/sale_timesheet_filters.xml',
        'wizard/project_create_sale_order_views.xml',
        'wizard/project_create_invoice_views.xml',
        'wizard/sale_make_invoice_advance_views.xml',
    ],
    'demo': [
        'data/sale_service_demo.xml',
    ],
    'auto_install': True,
    'uninstall_hook': 'uninstall_hook',
    'assets': {
        'web.assets_frontend': [
            'sale_timesheet/static/src/scss/sale_timesheet_portal.scss',
        ],
        'web.assets_backend': [
            'sale_timesheet/static/src/js/so_line_one2many.js',
        ],
        'web.assets_tests': [
            'sale_timesheet/static/tests/**/*',
        ],
        'web.assets_qweb': [
            'sale_timesheet/static/src/xml/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http, _
from odoo.exceptions import AccessError, MissingError
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

class SaleTimesheetCustomerPortal(TimesheetCustomerPortal):

    def _get_searchbar_inputs(self):
        searchbar_inputs = super()._get_searchbar_inputs()
        searchbar_inputs.update(
            so={'input': 'so', 'label': _('Search in Sales Order')},
            sol={'input': 'sol', 'label': _('Search in Sales Order Item')},
            invoice={'input': 'invoice', 'label': _('Search in Invoice')})
        return searchbar_inputs

    def _get_searchbar_groupby(self):
        searchbar_groupby = super()._get_searchbar_groupby()
        searchbar_groupby.update(
            so={'input': 'so', 'label': _('Sales Order')},
            sol={'input': 'sol', 'label': _('Sales Order Item')},
            invoice={'input': 'invoice', 'label': _('Invoice')})
        return searchbar_groupby

    def _get_search_domain(self, search_in, search):
        search_domain = super()._get_search_domain(search_in, search)
        if search_in in ('sol', 'all'):
            search_domain = expression.OR([search_domain, [('so_line', 'ilike', search)]])
        if search_in in ('so', 'all'):
            search_domain = expression.OR([search_domain, [('so_line.order_id.name', 'ilike', search)]])
        if search_in in ('invoice', 'all'):
            invoices = request.env['account.move'].sudo().search([('name', 'ilike', search)])
            domain = request.env['account.analytic.line']._timesheet_get_sale_domain(invoices.mapped('invoice_line_ids.sale_line_ids'), invoices)
            search_domain = expression.OR([search_domain, domain])
        return search_domain

    def _get_groupby_mapping(self):
        groupby_mapping = super()._get_groupby_mapping()
        groupby_mapping.update(
            sol='so_line',
            so='order_id',
            invoice='timesheet_invoice_id')
        return groupby_mapping

    def _get_searchbar_sortings(self):
        searchbar_sortings = super()._get_searchbar_sortings()
        searchbar_sortings.update(
            sol={'label': _('Sales Order Item'), 'order': 'so_line'})
        return searchbar_sortings

    def _task_get_page_view_values(self, task, access_token, **kwargs):
        values = super()._task_get_page_view_values(task, access_token, **kwargs)
        values['so_accessible'] = False
        try:
            if task.sale_order_id and self._document_check_access('sale.order', task.sale_order_id.id):
                values['so_accessible'] = True
        except (AccessError, MissingError):
            pass

        values['invoices_accessible'] = []
        for invoice in task.sale_order_id.invoice_ids:
            try:
                if self._document_check_access('account.move', invoice.id):
                    values['invoices_accessible'].append(invoice.id)
            except (AccessError, MissingError):
                pass
        return values

    @http.route(['/my/timesheets', '/my/timesheets/page/<int:page>'], type='http', auth="user", website=True)
    def portal_my_timesheets(self, page=1, sortby=None, filterby=None, search=None, search_in='all', groupby='sol', **kw):
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
            <field name="user_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="create_date" eval="DateTime.now() - relativedelta(months=5)"/>
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
            <value eval="{'user_ids': [(4, ref('base.user_admin'))]}"/>
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
            <field name="date" eval="(DateTime.now() + relativedelta(months=-2, days=-10)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">8.00</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="task_id" ref="project_task_internal"/>
        </record>
        <record id="timesheet_24" model="account.analytic.line">
            <field name="name">Internal training</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(months=-2, days=-12)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">8.00</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="task_id" ref="project_task_internal"/>
        </record>
        <record id="timesheet_25" model="account.analytic.line">
            <field name="name">Internal discussion</field>
            <field name="employee_id" ref="hr.employee_admin"/>
            <field name="date" eval="(DateTime.now() + relativedelta(months=-2, days=-13)).strftime('%Y-%m-%d')"/>
            <field name="unit_amount">8.00</field>
            <field name="project_id" ref="project.project_project_1"/>
            <field name="task_id" ref="project_task_internal"/>
        </record>
        <record id="timesheet_26" model="account.analytic.line">
            <field name="name">Details improvement</field>
            <field name="employee_id" ref="hr.employee_qdp"/>
            <field name="date" eval="(DateTime.now() + relativedelta(months=-2, days=-11)).strftime('%Y-%m-%d')"/>
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
            <field name="user_ids" eval="[(4, ref('base.user_admin'))]"/>
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

        <record id="project_update_1" model="project.update" context="{'default_project_id': ref('project.project_project_1')}">
            <field name="name">Weekly review</field>
            <field name="user_id" eval="ref('base.user_demo')"/>
            <field name="progress" eval="50"/>
            <field name="status">on_track</field>
        </record>
        <record id="project_update_2" model="project.update" context="{'default_project_id': ref('project.project_project_2')}">
            <field name="name">Weekly review</field>
            <field name="user_id" eval="ref('base.user_admin')"/>
            <field name="progress" eval="35"/>
            <field name="status">on_hold</field>
        </record>
        <record id="project_update_3" model="project.update" context="{'default_project_id': ref('sale_timesheet.project_support')}">
            <field name="name">Review of the situation</field>
            <field name="user_id" eval="ref('base.user_admin')"/>
            <field name="progress" eval="30"/>
            <field name="status">at_risk</field>
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
            'pivot_measures': ['amount_untaxed_to_invoice', 'amount_untaxed_invoiced', 'timesheet_cost', 'margin']
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
        ('timesheet_revenues', 'Timesheet Revenues'),
        ('service_revenues', 'Service Revenues'),
        ('other_revenues', 'Other Revenues'),
        ('other_costs', 'Other Costs')], string="Billable Type",
            compute='_compute_timesheet_invoice_type', compute_sudo=True, store=True, readonly=True)
    commercial_partner_id = fields.Many2one('res.partner', compute="_compute_commercial_partner")
    timesheet_invoice_id = fields.Many2one('account.move', string="Invoice", readonly=True, copy=False, help="Invoice created from the timesheet")
    so_line = fields.Many2one(compute="_compute_so_line", store=True, readonly=False, domain="[('is_service', '=', True), ('is_expense', '=', False), ('state', 'in', ['sale', 'done']), ('order_partner_id', 'child_of', commercial_partner_id)]")
    # we needed to store it only in order to be able to groupby in the portal
    order_id = fields.Many2one(related='so_line.order_id', store=True, readonly=True)
    is_so_line_edited = fields.Boolean("Is Sales Order Item Manually Edited")

    @api.depends('project_id.commercial_partner_id', 'task_id.commercial_partner_id')
    def _compute_commercial_partner(self):
        for timesheet in self:
            timesheet.commercial_partner_id = timesheet.task_id.commercial_partner_id or timesheet.project_id.commercial_partner_id

    # When user edit Sale Order Item(so_line) for timesheet make is_so_line_edited field true
    @api.onchange('so_line')
    def _onchange_so_line(self):
        # TODO: [XBO] remove me in master
        return

    @api.depends('so_line.product_id', 'project_id', 'amount')
    def _compute_timesheet_invoice_type(self):
        for timesheet in self:
            if timesheet.project_id:  # AAL will be set to False
                invoice_type = 'non_billable' if not timesheet.so_line else False
                if timesheet.so_line and timesheet.so_line.product_id.type == 'service':
                    if timesheet.so_line.product_id.invoice_policy == 'delivery':
                        if timesheet.so_line.product_id.service_type == 'timesheet':
                            invoice_type = 'timesheet_revenues' if timesheet.amount > 0 else 'billable_time'
                        else:
                            invoice_type = 'billable_fixed'
                    elif timesheet.so_line.product_id.invoice_policy == 'order':
                        invoice_type = 'billable_fixed'
                timesheet.timesheet_invoice_type = invoice_type
            else:
                if timesheet.so_line and timesheet.so_line.product_id.type == 'service':
                    timesheet.timesheet_invoice_type = 'service_revenues'
                else:
                    timesheet.timesheet_invoice_type = 'other_revenues' if timesheet.amount >= 0 else 'other_costs'

    @api.depends('task_id.sale_line_id', 'project_id.sale_line_id', 'employee_id', 'project_id.allow_billable')
    def _compute_so_line(self):
        for timesheet in self.filtered(lambda t: not t.is_so_line_edited and t._is_not_billed()):  # Get only the timesheets are not yet invoiced
            timesheet.so_line = timesheet.project_id.allow_billable and timesheet._timesheet_determine_sale_line()
    
    @api.depends('timesheet_invoice_id.state')
    def _compute_partner_id(self):
        super(AccountAnalyticLine, self.filtered(lambda t: t._is_not_billed()))._compute_partner_id()

    @api.depends('timesheet_invoice_id.state')
    def _compute_project_id(self):
        super(AccountAnalyticLine, self.filtered(lambda t: t._is_not_billed()))._compute_project_id()

    def _is_not_billed(self):
        self.ensure_one()
        return not self.timesheet_invoice_id or self.timesheet_invoice_id.state == 'cancel'

    def _check_timesheet_can_be_billed(self):
        return self.so_line in self.project_id.mapped('sale_line_employee_ids.sale_line_id') | self.task_id.sale_line_id | self.project_id.sale_line_id

    def write(self, values):
        # prevent to update invoiced timesheets if one line is of type delivery
        self._check_can_write(values)
        result = super(AccountAnalyticLine, self).write(values)
        return result

    def _check_can_write(self, values):
        if self.sudo().filtered(lambda aal: aal.so_line.product_id.invoice_policy == "delivery") and self.filtered(lambda t: t.timesheet_invoice_id and t.timesheet_invoice_id.state != 'cancel'):
            if any(field_name in values for field_name in ['unit_amount', 'employee_id', 'project_id', 'task_id', 'so_line', 'amount', 'date']):
                raise UserError(_('You cannot modify timesheets that are already invoiced.'))

    @api.model
    def _timesheet_preprocess(self, values):
        # TODO: remove me in master
        return super()._timesheet_preprocess(values)

    def _timesheet_determine_sale_line(self):
        """ Deduce the SO line associated to the timesheet line:
            1/ timesheet on task rate: the so line will be the one from the task
            2/ timesheet on employee rate task: find the SO line in the map of the project (even for subtask), or fallback on the SO line of the task, or fallback
                on the one on the project
        """
        self.ensure_one()

        if not self.task_id:
            if self.project_id.pricing_type == 'employee_rate':
                map_entry = self._get_employee_mapping_entry()
                if map_entry:
                    return map_entry.sale_line_id
            if self.project_id.sale_line_id:
                return self.project_id.sale_line_id
        if self.task_id.allow_billable and self.task_id.sale_line_id:
            if self.task_id.pricing_type in ('task_rate', 'fixed_rate'):
                return self.task_id.sale_line_id
            else:  # then pricing_type = 'employee_rate'
                map_entry = self.project_id.sale_line_employee_ids.filtered(
                    lambda map_entry:
                        map_entry.employee_id == self.employee_id
                        and map_entry.sale_line_id.order_partner_id.commercial_partner_id == self.task_id.commercial_partner_id
                )
                if map_entry:
                    return map_entry.sale_line_id
                return self.task_id.sale_line_id
        return False

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
                '&',
                ('so_line', 'in', order_lines_ids.ids),
                ('timesheet_invoice_id', '=', False),
        ]

    def _get_timesheets_to_merge(self):
        res = super(AccountAnalyticLine, self)._get_timesheets_to_merge()
        return res.filtered(lambda l: not l.timesheet_invoice_id or l.timesheet_invoice_id.state != 'posted')

    @api.ondelete(at_uninstall=False)
    def _unlink_except_invoiced(self):
        if any(line.timesheet_invoice_id and line.timesheet_invoice_id.state == 'posted' for line in self):
            raise UserError(_('You cannot remove a timesheet that has already been invoiced.'))

    def _get_employee_mapping_entry(self):
        self.ensure_one()
        return self.env['project.sale.line.employee.map'].search([('project_id', '=', self.project_id.id), ('employee_id', '=', self.employee_id.id or self.env.user.employee_id.id)])

    def _employee_timesheet_cost(self):
        if self.project_id.pricing_type == 'employee_rate':
            mapping_entry = self._get_employee_mapping_entry()
            if mapping_entry:
                return mapping_entry.cost
        return super()._employee_timesheet_cost()

    def _timesheet_convert_sol_uom(self, sol, to_unit):
        to_uom = self.env.ref(to_unit)
        return round(sol.product_uom._compute_quantity(sol.product_uom_qty, to_uom, raise_if_failure=False), 2)

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
    timesheet_encode_uom_id = fields.Many2one('uom.uom', related='company_id.timesheet_encode_uom_id')
    timesheet_total_duration = fields.Integer("Timesheet Total Duration", compute='_compute_timesheet_total_duration', help="Total recorded duration, expressed in the encoding UoM, and rounded to the unit")

    @api.depends('timesheet_ids', 'company_id.timesheet_encode_uom_id')
    def _compute_timesheet_total_duration(self):
        if not self.user_has_groups('hr_timesheet.group_hr_timesheet_user'):
            self.timesheet_total_duration = 0
            return
        group_data = self.env['account.analytic.line'].read_group([
            ('timesheet_invoice_id', 'in', self.ids)
        ], ['timesheet_invoice_id', 'unit_amount'], ['timesheet_invoice_id'])
        timesheet_unit_amount_dict = defaultdict(float)
        timesheet_unit_amount_dict.update({data['timesheet_invoice_id'][0]: data['unit_amount'] for data in group_data})
        for invoice in self:
            total_time = invoice.company_id.project_time_mode_id._compute_quantity(
                timesheet_unit_amount_dict[invoice.id],
                invoice.timesheet_encode_uom_id,
                rounding_method='HALF-UP',
            )
            invoice.timesheet_total_duration = round(total_time)

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
            if timesheet['so_line'] and timesheet['so_line'][0] in sale_line_ids_per_move[move_id].ids:
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

SERVICE_POLICY = [
    # (service_policy, (invoice_policy, service_type), string)
    ('ordered_timesheet', ('order', 'timesheet'), 'Prepaid/Fixed Price'),
    ('delivered_timesheet', ('delivery', 'timesheet'), 'Based on Timesheets'),
    ('delivered_manual', ('delivery', 'manual'), 'Based on Milestones'),
]
SERVICE_TO_GENERAL = {policy[0]: policy[1] for policy in SERVICE_POLICY}
GENERAL_TO_SERVICE = {policy[1]: policy[0] for policy in SERVICE_POLICY}


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    service_policy = fields.Selection([
        (policy[0], policy[2]) for policy in SERVICE_POLICY
    ], string="Service Invoicing Policy", compute='_compute_service_policy', inverse='_inverse_service_policy')
    service_type = fields.Selection(selection_add=[
        ('timesheet', 'Timesheets on project (one fare per SO/Project)'),
    ], ondelete={'timesheet': 'set default'})
    # override domain
    project_id = fields.Many2one(domain="[('company_id', '=', current_company_id), ('allow_billable', '=', True), ('pricing_type', '=', 'task_rate'), ('allow_timesheets', 'in', [service_policy == 'delivered_timesheet', True])]")
    project_template_id = fields.Many2one(domain="[('company_id', '=', current_company_id), ('allow_billable', '=', True), ('allow_timesheets', 'in', [service_policy == 'delivered_timesheet', True])]")
    service_upsell_threshold = fields.Float('Threshold', default=1, help="Percentage of time delivered compared to the prepaid amount that must be reached for the upselling opportunity activity to be triggered.")
    service_upsell_threshold_ratio = fields.Char(compute='_compute_service_upsell_threshold_ratio')

    @api.depends('uom_id')
    def _compute_service_upsell_threshold_ratio(self):
        product_uom_hour = self.env.ref('uom.product_uom_hour')
        for record in self:
            if not record.uom_id:
                record.service_upsell_threshold_ratio = False
                continue
            record.service_upsell_threshold_ratio = f"1 {record.uom_id.name} = {product_uom_hour.factor / record.uom_id.factor:.2f} Hours"

    def _compute_visible_expense_policy(self):
        visibility = self.user_has_groups('project.group_project_user')
        for product_template in self:
            if not product_template.visible_expense_policy:
                product_template.visible_expense_policy = visibility
        return super(ProductTemplate, self)._compute_visible_expense_policy()

    @api.depends('invoice_policy', 'service_type', 'type')
    def _compute_service_policy(self):
        for product in self:
            product.service_policy = GENERAL_TO_SERVICE.get((product.invoice_policy, product.service_type), False)
            if not product.service_policy and product.type == 'service':
                product.service_policy = 'ordered_timesheet'

    @api.onchange('service_policy')
    def _inverse_service_policy(self):
        for product in self:
            if product.service_policy:
                product.invoice_policy, product.service_type = SERVICE_TO_GENERAL.get(product.service_policy, (False, False))

    @api.depends('service_tracking', 'service_policy', 'type')
    def _compute_product_tooltip(self):
        super()._compute_product_tooltip()
        for record in self.filtered(lambda record: record.type == 'service'):
            if record.service_policy == 'ordered_timesheet':
                pass
            elif record.service_policy == 'delivered_timesheet':
                if record.service_tracking == 'no':
                    record.product_tooltip = _(
                        "Invoice based on timesheets (delivered quantity) on projects or tasks "
                        "you'll create later on."
                    )
                elif record.service_tracking == 'task_global_project':
                    record.product_tooltip = _(
                        "Invoice based on timesheets (delivered quantity), and create a task in "
                        "an existing project to track the time spent."
                    )
                elif record.service_tracking == 'task_in_project':
                    record.product_tooltip = _(
                        "Invoice based on timesheets (delivered quantity), and create a project "
                        "for the order with a task for each sales order line to track the time "
                        "spent."
                    )
                elif record.service_tracking == 'project_only':
                    record.product_tooltip = _(
                        "Invoice based on timesheets (delivered quantity), and create an empty "
                        "project for the order to track the time spent."
                    )
            elif record.service_policy == 'delivered_manual':
                if record.service_tracking == 'no':
                    record.product_tooltip = _(
                        "Sales order lines define milestones of the project to invoice by setting "
                        "the delivered quantity."
                    )
                elif record.service_tracking == 'task_global_project':
                    record.product_tooltip = _(
                        "Sales order lines define milestones of the project to invoice by setting "
                        "the delivered quantity. Create a task in an existing project to track the"
                        " time spent."
                    )
                elif record.service_tracking == 'task_in_project':
                    record.product_tooltip = _(
                        "Sales order lines define milestones of the project to invoice by setting "
                        "the delivered quantity. Create an empty project for the order to track "
                        "the time spent."
                    )
                elif record.service_tracking == 'project_only':
                    record.product_tooltip = _(
                        "Sales order lines define milestones of the project to invoice by setting "
                        "the delivered quantity. Create a project for the order with a task for "
                        "each sales order line to track the time spent."
                    )

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
        self._inverse_service_policy()
        vals = self._get_onchange_service_policy_updates(self.service_tracking,
                                                        self.service_policy,
                                                        self.project_id,
                                                        self.project_template_id)
        if vals:
            self.update(vals)

    @api.ondelete(at_uninstall=False)
    def _unlink_except_master_data(self):
        time_product = self.env.ref('sale_timesheet.time_product')
        if time_product.product_tmpl_id in self:
            raise ValidationError(_('The %s product is required by the Timesheets app and cannot be archived nor deleted.') % time_product.name)

    def write(self, vals):
        # timesheet product can't be archived
        test_mode = getattr(threading.current_thread(), 'testing', False) or self.env.registry.in_test_mode()
        if not test_mode and 'active' in vals and not vals['active']:
            time_product = self.env.ref('sale_timesheet.time_product')
            if time_product.product_tmpl_id in self:
                raise ValidationError(_('The %s product is required by the Timesheets app and cannot be archived nor deleted.') % time_product.name)
        return super(ProductTemplate, self).write(vals)


class ProductProduct(models.Model):
    _inherit = 'product.product'

    def _is_delivered_timesheet(self):
        """ Check if the product is a delivered timesheet """
        self.ensure_one()
        return self.type == 'service' and self.service_policy == 'delivered_timesheet'

    def _inverse_service_policy(self):
        for product in self:
            if product.service_policy:
                product.invoice_policy, product.service_type = SERVICE_TO_GENERAL.get(product.service_policy, (False, False))

    @api.onchange('service_policy')
    def _onchange_service_policy(self):
        self._inverse_service_policy()
        vals = self.product_tmpl_id._get_onchange_service_policy_updates(self.service_tracking,
                                                                        self.service_policy,
                                                                        self.project_id,
                                                                        self.project_template_id)
        if vals:
            self.update(vals)

    @api.ondelete(at_uninstall=False)
    def _unlink_except_master_data(self):
        time_product = self.env.ref('sale_timesheet.time_product')
        if time_product in self:
            raise ValidationError(_('The %s product is required by the Timesheets app and cannot be archived nor deleted.') % time_product.name)

    def write(self, vals):
        # timesheet product can't be archived
        test_mode = getattr(threading.current_thread(), 'testing', False) or self.env.registry.in_test_mode()
        if not test_mode and 'active' in vals and not vals['active']:
            time_product = self.env.ref('sale_timesheet.time_product')
            if time_product in self:
                raise ValidationError(_('The %s product is required by the Timesheets app and cannot be archived nor deleted.') % time_product.name)
        return super(ProductProduct, self).write(vals)

```

## File: models\project.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
from collections import defaultdict

from odoo import api, fields, models, _, _lt
from odoo.osv import expression
from odoo.exceptions import ValidationError, UserError
from odoo.tools import format_amount, float_is_zero, formatLang

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

    pricing_type = fields.Selection([
        ('task_rate', 'Task rate'),
        ('fixed_rate', 'Project rate'),
        ('employee_rate', 'Employee rate')
    ], string="Pricing", default="task_rate",
        compute='_compute_pricing_type',
        search='_search_pricing_type',
        help='The task rate is perfect if you would like to bill different services to different customers at different rates. The fixed rate is perfect if you bill a service at a fixed rate per hour or day worked regardless of the employee who performed it. The employee rate is preferable if your employees deliver the same service at a different rate. For instance, junior and senior consultants would deliver the same service (= consultancy), but at a different rate because of their level of seniority.')
    sale_line_employee_ids = fields.One2many('project.sale.line.employee.map', 'project_id', "Sale line/Employee map", copy=False,
        help="Employee/Sale Order Item Mapping:\n Defines to which sales order item an employee's timesheet entry will be linked."
        "By extension, it defines the rate at which an employee's time on the project is billed.")
    allow_billable = fields.Boolean("Billable", help="Invoice your time and material from tasks.")
    billable_percentage = fields.Integer(
        compute='_compute_billable_percentage', groups='hr_timesheet.group_hr_timesheet_approver',
        help="% of timesheets that are billable compared to the total number of timesheets linked to the AA of the project, rounded to the unit.")
    display_create_order = fields.Boolean(compute='_compute_display_create_order')
    timesheet_product_id = fields.Many2one(
        'product.product', string='Timesheet Product',
        domain="""[
            ('detailed_type', '=', 'service'),
            ('invoice_policy', '=', 'delivery'),
            ('service_type', '=', 'timesheet'),
            '|', ('company_id', '=', False), ('company_id', '=', company_id)]""",
        help='Select a Service product with which you would like to bill your time spent on tasks.',
        compute="_compute_timesheet_product_id", store=True, readonly=False,
        default=_default_timesheet_product_id)
    warning_employee_rate = fields.Boolean(compute='_compute_warning_employee_rate', compute_sudo=True)
    partner_id = fields.Many2one(compute='_compute_partner_id', store=True, readonly=False)

    @api.depends('sale_line_id', 'sale_line_employee_ids', 'allow_billable')
    def _compute_pricing_type(self):
        billable_projects = self.filtered('allow_billable')
        for project in billable_projects:
            if project.sale_line_employee_ids:
                project.pricing_type = 'employee_rate'
            elif project.sale_line_id:
                project.pricing_type = 'fixed_rate'
            else:
                project.pricing_type = 'task_rate'
        (self - billable_projects).update({'pricing_type': False})

    def _search_pricing_type(self, operator, value):
        """ Search method for pricing_type field.

            This method returns a domain based on the operator and the value given in parameter:
            - operator = '=':
                - value = 'task_rate': [('sale_line_employee_ids', '=', False), ('sale_line_id', '=', False), ('allow_billable', '=', True)]
                - value = 'fixed_rate': [('sale_line_employee_ids', '=', False), ('sale_line_id', '!=', False), ('allow_billable', '=', True)]
                - value = 'employee_rate': [('sale_line_employee_ids', '!=', False), ('allow_billable', '=', True)]
                - value is False: [('allow_billable', '=', False)]
            - operator = '!=':
                - value = 'task_rate': ['|', '|', ('sale_line_employee_ids', '!=', False), ('sale_line_id', '!=', False), ('allow_billable', '=', False)]
                - value = 'fixed_rate': ['|', '|', ('sale_line_employee_ids', '!=', False), ('sale_line_id', '=', False), ('allow_billable', '=', False)]
                - value = 'employee_rate': ['|', ('sale_line_employee_ids', '=', False), ('allow_billable', '=', False)]
                - value is False: [('allow_billable', '!=', False)]

            :param operator: the supported operator is either '=' or '!='.
            :param value: the value than the field should be is among these values into the following tuple: (False, 'task_rate', 'fixed_rate', 'employee_rate').

            :returns: the domain to find the expected projects.
        """
        if operator not in ('=', '!='):
            raise UserError(_('Operation not supported'))
        if not ((isinstance(value, bool) and value is False) or (isinstance(value, str) and value in ('task_rate', 'fixed_rate', 'employee_rate'))):
            raise UserError(_('Value does not exist in the pricing type'))
        if value is False:
            return [('allow_billable', operator, value)]

        sol_cond = ('sale_line_id', '!=', False)
        mapping_cond = ('sale_line_employee_ids', '!=', False)
        if value == 'task_rate':
            domain = [expression.NOT_OPERATOR, sol_cond, expression.NOT_OPERATOR, mapping_cond]
        elif value == 'fixed_rate':
            domain = [sol_cond, expression.NOT_OPERATOR, mapping_cond]
        else:  # value == 'employee_rate'
            domain = [mapping_cond]

        domain = expression.AND([domain, [('allow_billable', '=', True)]])
        domain = expression.normalize_domain(domain)
        if operator != '=':
            domain.insert(0, expression.NOT_OPERATOR)
        domain = expression.distribute_not(domain)
        return domain

    @api.depends('analytic_account_id', 'timesheet_ids')
    def _compute_billable_percentage(self):
        timesheets_read_group = self.env['account.analytic.line'].read_group([('project_id', 'in', self.ids)], ['project_id', 'so_line', 'unit_amount'], ['project_id', 'so_line'], lazy=False)
        timesheets_by_project = defaultdict(list)
        for res in timesheets_read_group:
            timesheets_by_project[res['project_id'][0]].append((res['unit_amount'], bool(res['so_line'])))
        for project in self:
            timesheet_total = timesheet_billable = 0.0
            for unit_amount, is_billable_timesheet in timesheets_by_project[project.id]:
                timesheet_total += unit_amount
                if is_billable_timesheet:
                    timesheet_billable += unit_amount
            billable_percentage = timesheet_billable / timesheet_total * 100 if timesheet_total > 0 else 0
            project.billable_percentage = round(billable_percentage)

    @api.depends('partner_id', 'pricing_type')
    def _compute_display_create_order(self):
        for project in self:
            project.display_create_order = project.partner_id and project.pricing_type == 'task_rate'

    @api.depends('allow_timesheets', 'allow_billable')
    def _compute_timesheet_product_id(self):
        default_product = self.env.ref('sale_timesheet.time_product', False)
        for project in self:
            if not project.allow_timesheets or not project.allow_billable:
                project.timesheet_product_id = False
            elif not project.timesheet_product_id:
                project.timesheet_product_id = default_product

    @api.depends('pricing_type', 'allow_timesheets', 'allow_billable', 'sale_line_employee_ids', 'sale_line_employee_ids.employee_id')
    def _compute_warning_employee_rate(self):
        projects = self.filtered(lambda p: p.allow_billable and p.allow_timesheets and p.pricing_type == 'employee_rate')
        employees = self.env['account.analytic.line'].read_group([('task_id', 'in', projects.task_ids.ids)], ['employee_id', 'project_id'], ['employee_id', 'project_id'], lazy=False)
        dict_project_employee = defaultdict(list)
        for line in employees:
            dict_project_employee[line['project_id'][0]] += [line['employee_id'][0]] if line['employee_id'] else []
        for project in projects:
            project.warning_employee_rate = any(x not in project.sale_line_employee_ids.employee_id.ids for x in dict_project_employee[project.id])

        (self - projects).warning_employee_rate = False

    @api.depends('sale_line_employee_ids.sale_line_id', 'sale_line_id')
    def _compute_partner_id(self):
        for project in self:
            if project.partner_id:
                continue
            if project.allow_billable and project.allow_timesheets and project.pricing_type != 'task_rate':
                sol = project.sale_line_id or project.sale_line_employee_ids.sale_line_id[:1]
                project.partner_id = sol.order_partner_id

    @api.depends('partner_id')
    def _compute_sale_line_id(self):
        super()._compute_sale_line_id()
        for project in self.filtered(lambda p: not p.sale_line_id and p.partner_id and p.pricing_type == 'employee_rate'):
            # Give a SOL by default either the last SOL with service product and remaining_hours > 0
            sol = self.env['sale.order.line'].search([
                ('is_service', '=', True),
                ('order_partner_id', 'child_of', project.partner_id.commercial_partner_id.id),
                ('is_expense', '=', False),
                ('state', 'in', ['sale', 'done']),
                ('remaining_hours', '>', 0)
            ], limit=1)
            project.sale_line_id = sol or project.sale_line_employee_ids.sale_line_id[:1]  # get the first SOL containing in the employee mappings if no sol found in the search

    @api.constrains('sale_line_id')
    def _check_sale_line_type(self):
        for project in self.filtered(lambda project: project.sale_line_id):
            if not project.sale_line_id.is_service:
                raise ValidationError(_("You cannot link a billable project to a sales order item that is not a service."))
            if project.sale_line_id.is_expense:
                raise ValidationError(_("You cannot link a billable project to a sales order item that comes from an expense or a vendor bill."))

    def write(self, values):
        res = super(Project, self).write(values)
        if 'allow_billable' in values and not values.get('allow_billable'):
            self.task_ids._get_timesheet().write({
                'so_line': False,
            })
        return res

    def _update_timesheets_sale_line_id(self):
        for project in self.filtered(lambda p: p.allow_billable and p.allow_timesheets):
            timesheet_ids = project.sudo(False).mapped('timesheet_ids').filtered(lambda t: not t.is_so_line_edited and t._is_not_billed())
            if not timesheet_ids:
                continue
            for employee_id in project.sale_line_employee_ids.filtered(lambda l: l.project_id == project).employee_id:
                sale_line_id = project.sale_line_employee_ids.filtered(lambda l: l.project_id == project and l.employee_id == employee_id).sale_line_id
                timesheet_ids.filtered(lambda t: t.employee_id == employee_id).sudo().so_line = sale_line_id

    def action_open_project_invoices(self):
        invoices = self.env['account.move'].search([
            ('line_ids.analytic_account_id', '!=', False),
            ('line_ids.analytic_account_id', 'in', self.analytic_account_id.ids),
            ('move_type', '=', 'out_invoice')
        ])
        action = {
            'name': _('Invoices'),
            'type': 'ir.actions.act_window',
            'res_model': 'account.move',
            'views': [[False, 'tree'], [False, 'form'], [False, 'kanban']],
            'domain': [('id', 'in', invoices.ids)],
            'context': {
                'create': False,
            }
        }
        if len(invoices) == 1:
            action['views'] = [[False, 'form']]
            action['res_id'] = invoices.id
        return action

    def action_view_timesheet(self):
        self.ensure_one()
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

    def action_billable_time_button(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("hr_timesheet.timesheet_action_all")
        action.update({
            'context': {
                'search_default_groupby_task': True,
                'default_project_id': self.id,
            },
            'domain': [('project_id', '=', self.id)],
            'view_mode': 'tree,kanban,pivot,graph,form',
            'views': [
                [self.env.ref('hr_timesheet.timesheet_view_tree_user').id, 'tree'],
                [self.env.ref('hr_timesheet.view_kanban_account_analytic_line').id, 'kanban'],
                [self.env.ref('hr_timesheet.view_hr_timesheet_line_pivot').id, 'pivot'],
                [self.env.ref('hr_timesheet.view_hr_timesheet_line_graph_all').id, 'graph'],
                [self.env.ref('hr_timesheet.timesheet_view_form_user').id, 'form'],
            ],
        })
        return action

    def action_view_all_rating(self):
        return {
            'name': _('Rating'),
            'type': 'ir.actions.act_window',
            'res_model': 'rating.rating',
            'view_mode': 'kanban,list,graph,pivot,form',
            'view_type': 'ir.actions.act_window',
            'context': {
                'search_default_rating_last_30_days': True,
            },
            'domain': [('consumed','=',True), ('parent_res_model','=','project.project'), ('parent_res_id', '=', self.id)],
        }

    # ----------------------------
    #  Project Updates
    # ----------------------------

    def get_panel_data(self):
        panel_data = super(Project, self).get_panel_data()
        return {
            **panel_data,
            'analytic_account_id': self.analytic_account_id.id,
            'sold_items': self._get_sold_items(),
            'profitability_items': self._get_profitability_items(),
        }

    def _get_sale_order_lines(self):
        sale_orders = self._get_sale_orders()
        return self.env['sale.order.line'].search([('order_id', 'in', sale_orders.ids), ('is_service', '=', True), ('is_downpayment', '=', False)], order='id asc')

    def _get_sold_items(self):
        timesheet_encode_uom = self.env.company.timesheet_encode_uom_id
        product_uom_unit = self.env.ref('uom.product_uom_unit')
        product_uom_hour = self.env.ref('uom.product_uom_hour')

        sols = self._get_sale_order_lines()
        number_sale_orders = len(sols.order_id)
        sold_items = {
            'allow_billable': self.allow_billable,
            'data': [],
            'number_sols': len(sols),
            'total_sold': 0,
            'effective_sold': 0,
            'company_unit_name': timesheet_encode_uom.name
        }

        for sol in sols:
            name = [x[1] for x in sol.name_get()] if number_sale_orders > 1 else sol.name

            product_uom_convert = sol.product_uom
            if product_uom_convert == product_uom_unit:
                product_uom_convert = product_uom_hour
            qty_delivered = product_uom_convert._compute_quantity(sol.qty_delivered, timesheet_encode_uom, raise_if_failure=False)
            product_uom_qty = product_uom_convert._compute_quantity(sol.product_uom_qty, timesheet_encode_uom, raise_if_failure=False)
            if product_uom_convert.category_id == timesheet_encode_uom.category_id:
                product_uom_convert = timesheet_encode_uom

            if qty_delivered > 0 or product_uom_qty > 0:
                sold_items['data'].append({
                    'name': name,
                    'value': '%s / %s %s' % (formatLang(self.env, qty_delivered, 1), formatLang(self.env, product_uom_qty, 1), product_uom_convert.name),
                    'color': 'red' if qty_delivered > product_uom_qty else 'black'
                })
                #We only want to consider hours and days for this calculation, and eventually units if the service policy is not based on milestones
                if sol.product_uom.category_id == timesheet_encode_uom.category_id or (sol.product_uom == product_uom_unit and sol.product_id.service_policy != 'delivered_manual'):
                    sold_items['total_sold'] += product_uom_qty
                    sold_items['effective_sold'] += qty_delivered
        remaining = sold_items['total_sold'] - sold_items['effective_sold']
        sold_items['remaining'] = {
            'value': remaining,
            'color': 'red' if remaining < 0 else 'black',
        }
        return sold_items

    def _fetch_sale_order_item_ids(self, domain_per_model=None, limit=None, offset=None):
        if not self or not self.filtered('allow_billable'):
            return []
        return super()._fetch_sale_order_item_ids(domain_per_model, limit, offset)

    def _get_sale_order_items_query(self, domain_per_model=None):
        billable_project_domain = [('allow_billable', '=', True)]
        if domain_per_model is None:
            domain_per_model = {
                'project.project': billable_project_domain,
                'project.task': billable_project_domain,
            }
        else:
            domain_per_model['project.project'] = expression.AND([
                domain_per_model.get('project.project', []),
                billable_project_domain,
            ])
            domain_per_model['project.task'] = expression.AND([
                domain_per_model.get('project.task', []),
                billable_project_domain,
            ])
        return super()._get_sale_order_items_query(domain_per_model)

    def _get_profitability_items(self):
        if not self.user_has_groups('project.group_project_manager'):
            return {'data': []}
        data = []
        if self.allow_billable:
            profitability = self._get_profitability_common()
            margin_color = False
            if not float_is_zero(profitability['margin'], precision_digits=0):
                margin_color = profitability['margin'] > 0 and 'green' or 'red'
            data += [{
                'name': _("Revenues"),
                'value': format_amount(self.env, profitability['revenues'], self.company_id.currency_id)
            }, {
                'name': _("Costs"),
                'value': format_amount(self.env, profitability['costs'], self.company_id.currency_id)
            }, {
                'name': _("Margin"),
                'color': margin_color,
                'value': format_amount(self.env, profitability['margin'], self.company_id.currency_id)
            }]
        return {
            'action': self.allow_billable and self.allow_timesheets and "action_view_timesheet",
            'allow_billable': self.allow_billable,
            'data': data,
        }

    def _get_profitability_common(self):
        self.ensure_one()
        result = {
            'costs': 0.0,
            'margin': 0.0,
            'revenues': 0.0
        }

        profitability = self.env['project.profitability.report'].read_group(
            [('project_id', '=', self.id)],
            ['project_id',
                'amount_untaxed_to_invoice',
                'amount_untaxed_invoiced',
                'expense_amount_untaxed_to_invoice',
                'expense_amount_untaxed_invoiced',
                'other_revenues',
                'expense_cost',
                'timesheet_cost',
                'margin'],
            ['project_id'], limit=1)
        if profitability:
            profitability = profitability[0]
            result.update({
                'costs': profitability['timesheet_cost'] + profitability['expense_cost'],
                'margin': profitability['margin'],
                'revenues': (profitability['amount_untaxed_invoiced'] + profitability['amount_untaxed_to_invoice'] +
                                profitability['expense_amount_untaxed_invoiced'] + profitability['expense_amount_untaxed_to_invoice'] +
                                profitability['other_revenues']),
            })
        return result

    def _get_sale_order_stat_button(self):
        so_button = super()._get_sale_order_stat_button()
        so_button['show'] &= self.allow_billable
        return so_button

    def _get_stat_buttons(self):
        buttons = super(Project, self)._get_stat_buttons()
        if self.user_has_groups('hr_timesheet.group_hr_timesheet_approver'):
            buttons.append({
                'icon': 'clock-o',
                'text': _lt('Billable Time'),
                'number': '%s %%' % (self.billable_percentage),
                'action_type': 'object',
                'action': 'action_billable_time_button',
                'additional_context': json.dumps({
                    'active_id': self.id,
                    'default_project_id': self.id
                }),
                'show': self.allow_timesheets and bool(self.analytic_account_id),
                'sequence': 9,
            })
        return buttons

class ProjectTask(models.Model):
    _inherit = "project.task"

    def _get_default_partner_id(self, project, parent):
        res = super()._get_default_partner_id(project, parent)
        if not res and project:
            # project in sudo if the current user is a portal user.
            related_project = project if not self.user_has_groups('!base.group_user,base.group_portal') else project.sudo()
            if related_project.pricing_type == 'employee_rate':
                return related_project.sale_line_employee_ids.sale_line_id.order_partner_id[:1]
        return res

    sale_order_id = fields.Many2one(domain="['|', '|', ('partner_id', '=', partner_id), ('partner_id', 'child_of', commercial_partner_id), ('partner_id', 'parent_of', partner_id)]")
    so_analytic_account_id = fields.Many2one(related='sale_order_id.analytic_account_id', string='Sale Order Analytic Account')
    pricing_type = fields.Selection(related="project_id.pricing_type")
    is_project_map_empty = fields.Boolean("Is Project map empty", compute='_compute_is_project_map_empty')
    has_multi_sol = fields.Boolean(compute='_compute_has_multi_sol', compute_sudo=True)
    allow_billable = fields.Boolean(related="project_id.allow_billable")
    timesheet_product_id = fields.Many2one(related="project_id.timesheet_product_id")
    remaining_hours_so = fields.Float('Remaining Hours on SO', compute='_compute_remaining_hours_so', compute_sudo=True)
    remaining_hours_available = fields.Boolean(related="sale_line_id.remaining_hours_available")

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS | {
            'allow_billable',
            'remaining_hours_available',
            'remaining_hours_so',
        }

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

    @api.depends('so_analytic_account_id.active')
    def _compute_analytic_account_active(self):
        super()._compute_analytic_account_active()
        for task in self:
            task.analytic_account_active = task.analytic_account_active or task.so_analytic_account_id.active

    @api.depends('allow_billable')
    def _compute_sale_order_id(self):
        billable_tasks = self.filtered('allow_billable')
        super(ProjectTask, billable_tasks)._compute_sale_order_id()
        (self - billable_tasks).sale_order_id = False

    @api.depends('commercial_partner_id', 'sale_line_id.order_partner_id.commercial_partner_id', 'parent_id.sale_line_id', 'project_id.sale_line_id', 'allow_billable')
    def _compute_sale_line(self):
        billable_tasks = self.filtered('allow_billable')
        (self - billable_tasks).update({'sale_line_id': False})
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

    def _get_last_sol_of_customer(self):
        # Get the last SOL made for the customer in the current task where we need to compute
        self.ensure_one()
        if not self.commercial_partner_id or not self.allow_billable:
            return False
        domain = [('company_id', '=', self.company_id.id), ('is_service', '=', True), ('order_partner_id', 'child_of', self.commercial_partner_id.id), ('is_expense', '=', False), ('state', 'in', ['sale', 'done']), ('remaining_hours', '>', 0)]
        if self.project_id.pricing_type != 'task_rate' and self.project_sale_order_id and self.commercial_partner_id == self.project_id.partner_id.commercial_partner_id:
            domain.append(('order_id', '=?', self.project_sale_order_id.id))
        return self.env['sale.order.line'].search(domain, limit=1)

    def _get_timesheet(self):
        # return not invoiced timesheet and timesheet without so_line or so_line linked to task
        timesheet_ids = super(ProjectTask, self)._get_timesheet()
        return timesheet_ids.filtered(lambda t: t._is_not_billed())

    def _get_action_view_so_ids(self):
        return list(set((self.sale_order_id + self.timesheet_ids.so_line.order_id).ids))

class ProjectTaskRecurrence(models.Model):
    _inherit = 'project.task.recurrence'

    @api.model
    def _get_recurring_fields(self):
        return ['so_analytic_account_id'] + super(ProjectTaskRecurrence, self)._get_recurring_fields()

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
    sale_line_id = fields.Many2one('sale.order.line', "Sale Order Item", compute="_compute_sale_line_id", store=True, readonly=False,
        domain="""[
            ('is_service', '=', True),
            ('is_expense', '=', False),
            ('state', 'in', ['sale', 'done']),
            ('order_partner_id', '=?', partner_id),
            '|', ('company_id', '=', False), ('company_id', '=', company_id)]""")
    company_id = fields.Many2one('res.company', string='Company', related='project_id.company_id')
    partner_id = fields.Many2one(related='project_id.partner_id')
    price_unit = fields.Float("Unit Price", compute='_compute_price_unit', store=True, readonly=True)
    currency_id = fields.Many2one('res.currency', string="Currency", compute='_compute_currency_id', store=True, readonly=False)
    cost = fields.Monetary(currency_field='cost_currency_id', compute='_compute_cost', store=True, readonly=False,
                           help="This cost overrides the employee's default timesheet cost in employee's HR Settings")
    cost_currency_id = fields.Many2one('res.currency', string="Cost Currency", related='employee_id.currency_id', readonly=True)
    is_cost_changed = fields.Boolean('Is Cost Manually Changed', compute='_compute_is_cost_changed', store=True)

    _sql_constraints = [
        ('uniqueness_employee', 'UNIQUE(project_id,employee_id)', 'An employee cannot be selected more than once in the mapping. Please remove duplicate(s) and try again.'),
    ]

    @api.depends('partner_id')
    def _compute_sale_line_id(self):
        self.filtered(
            lambda map_entry:
                map_entry.sale_line_id
                and map_entry.partner_id
                and map_entry.sale_line_id.order_partner_id.commercial_partner_id != map_entry.partner_id.commercial_partner_id
        ).update({'sale_line_id': False})

    @api.depends('sale_line_id.price_unit')
    def _compute_price_unit(self):
        for line in self:
            if line.sale_line_id:
                line.price_unit = line.sale_line_id.price_unit
            else:
                line.price_unit = 0

    @api.depends('sale_line_id.price_unit')
    def _compute_currency_id(self):
        for line in self:
            line.currency_id = line.sale_line_id.currency_id if line.sale_line_id else False

    @api.depends('employee_id.timesheet_cost')
    def _compute_cost(self):
        self.env.remove_to_compute(self._fields['is_cost_changed'], self)
        for map_entry in self:
            if not map_entry.is_cost_changed:
                map_entry.cost = map_entry.employee_id.timesheet_cost or 0.0

    @api.depends('cost')
    def _compute_is_cost_changed(self):
        for map_entry in self:
            map_entry.is_cost_changed = map_entry.employee_id and map_entry.cost != map_entry.employee_id.timesheet_cost

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

## File: models\project_update.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models
from odoo.tools import float_utils, format_amount, formatLang


class ProjectUpdate(models.Model):
    _inherit = 'project.update'

    @api.model
    def _get_template_values(self, project):
        template_values = super(ProjectUpdate, self)._get_template_values(project)
        services = self._get_services_values(project)
        profitability = self._get_profitability_values(project)
        show_sold = template_values['project'].allow_billable and len(services.get('data', [])) > 0
        return {
            **template_values,
            'show_sold': show_sold,
            'show_profitability': bool(profitability),
            'show_activities': template_values['show_activities'] or show_sold or bool(profitability),
            'services': services,
            'profitability': profitability,
        }

    @api.model
    def _get_project_sols(self, project):
        # TODO: remove me in master
        return

    @api.model
    def _get_services_values(self, project):
        if not project.allow_billable:
            return {}
        services = []
        total_sold, total_effective, total_remaining = 0, 0, 0
        sols = project._get_sale_order_lines()
        name_by_sol = dict(sols.name_get())
        product_uom_unit = self.env.ref('uom.product_uom_unit')
        for sol in sols:
            #We only want to consider hours and days for this calculation
            is_unit = sol.product_uom == product_uom_unit
            if sol.product_uom.category_id == self.env.company.timesheet_encode_uom_id.category_id or is_unit:
                product_uom_qty = sol.product_uom._compute_quantity(sol.product_uom_qty, self.env.company.timesheet_encode_uom_id, raise_if_failure=False)
                qty_delivered = sol.product_uom._compute_quantity(sol.qty_delivered, self.env.company.timesheet_encode_uom_id, raise_if_failure=False)
                services.append({
                    'name': name_by_sol[sol.id] if len(sols.order_id) > 1 else sol.name,
                    'sold_value': product_uom_qty,
                    'effective_value': qty_delivered,
                    'remaining_value': product_uom_qty - qty_delivered,
                    'unit': sol.product_uom.name if is_unit else self.env.company.timesheet_encode_uom_id.name,
                    'is_unit': is_unit,
                    'sol': sol,
                })
                if sol.product_uom.category_id == self.env.company.timesheet_encode_uom_id.category_id:
                    total_sold += product_uom_qty
                    total_effective += qty_delivered
        total_remaining = total_sold - total_effective
        return {
            'data': services,
            'total_sold': total_sold,
            'total_effective': total_effective,
            'total_remaining': total_remaining,
            'company_unit_name': self.env.company.timesheet_encode_uom_id.name,
        }

    @api.model
    def _get_profitability_values(self, project):
        costs_revenues = project.analytic_account_id and project.allow_billable
        if not (self.user_has_groups('project.group_project_manager') and costs_revenues):
            return {}
        profitability = project._get_profitability_common()
        return {
            'analytic_account_id': project.analytic_account_id,
            'costs': format_amount(self.env, -profitability['costs'], self.env.company.currency_id),
            'revenues': format_amount(self.env, profitability['revenues'], self.env.company.currency_id),
            'margin': profitability['margin'],
            'margin_formatted': format_amount(self.env, profitability['margin'], self.env.company.currency_id),
            'margin_percentage': formatLang(self.env,
                                            not float_utils.float_is_zero(profitability['costs'], precision_digits=2) and -(profitability['margin'] / profitability['costs']) * 100 or 0.0,
                                            digits=0),
        }

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    invoice_policy = fields.Boolean(string="Invoice Policy", help="Timesheets taken when invoicing time spent")

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import math
from collections import defaultdict

from odoo import api, fields, models, _
from odoo.osv import expression
from odoo.tools import float_compare, format_amount


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    timesheet_ids = fields.Many2many('account.analytic.line', compute='_compute_timesheet_ids', string='Timesheet activities associated to this sale')
    timesheet_count = fields.Float(string='Timesheet activities', compute='_compute_timesheet_ids', groups="hr_timesheet.group_hr_timesheet_user")

    # override domain
    project_id = fields.Many2one(domain="[('pricing_type', '!=', 'employee_rate'), ('analytic_account_id', '!=', False), ('company_id', '=', company_id)]")
    timesheet_encode_uom_id = fields.Many2one('uom.uom', related='company_id.timesheet_encode_uom_id')
    timesheet_total_duration = fields.Integer("Timesheet Total Duration", compute='_compute_timesheet_total_duration', help="Total recorded duration, expressed in the encoding UoM, and rounded to the unit")

    def _compute_timesheet_ids(self):
        timesheet_groups = self.env['account.analytic.line'].sudo().read_group(
            [('so_line', 'in', self.mapped('order_line').ids), ('project_id', '!=', False)],
            ['so_line', 'ids:array_agg(id)'],
            ['so_line'])
        timesheets_per_sol = {group['so_line'][0]: (group['ids'], group['so_line_count']) for group in timesheet_groups}

        for order in self:
            timesheet_ids = []
            timesheet_count = 0
            for sale_line_id in order.order_line.filtered('is_service').ids:
                list_timesheet_ids, count = timesheets_per_sol.get(sale_line_id, ([], 0))
                timesheet_ids.extend(list_timesheet_ids)
                timesheet_count += count

            order.update({
                'timesheet_ids': self.env['account.analytic.line'].browse(timesheet_ids),
                'timesheet_count': timesheet_count,
            })

    @api.depends('company_id.project_time_mode_id', 'timesheet_ids', 'company_id.timesheet_encode_uom_id')
    def _compute_timesheet_total_duration(self):
        if not self.user_has_groups('hr_timesheet.group_hr_timesheet_user'):
            self.update({'timesheet_total_duration': 0})
            return
        group_data = self.env['account.analytic.line'].sudo().read_group([
            ('order_id', 'in', self.ids), ('project_id', '!=', False)
        ], ['order_id', 'unit_amount'], ['order_id'])
        timesheet_unit_amount_dict = defaultdict(float)
        timesheet_unit_amount_dict.update({data['order_id'][0]: data['unit_amount'] for data in group_data})
        for sale_order in self:
            total_time = sale_order.company_id.project_time_mode_id._compute_quantity(
                timesheet_unit_amount_dict[sale_order.id],
                sale_order.timesheet_encode_uom_id,
                rounding_method='HALF-UP',
            )
            sale_order.timesheet_total_duration = round(total_time)

    def _compute_field_value(self, field):
        if field.name != 'invoice_status' or self.env.context.get('mail_activity_automation_skip'):
            return super()._compute_field_value(field)

        # Get SOs which their state is not equal to upselling or invoied and if at least a SOL has warning prepaid service upsell set to True and the warning has not already been displayed
        upsellable_orders = self.filtered(lambda so:
            so.state == 'sale'
            and so.invoice_status not in ('upselling', 'invoiced')
            and (so.user_id or so.partner_id.user_id)  # salesperson needed to assign upsell activity
        )
        super(SaleOrder, upsellable_orders.with_context(mail_activity_automation_skip=True))._compute_field_value(field)
        for order in upsellable_orders:
            upsellable_lines = order._get_prepaid_service_lines_to_upsell()
            if upsellable_lines:
                order._create_upsell_activity()
                # We want to display only one time the warning for each SOL
                upsellable_lines.write({'has_displayed_warning_upsell': True})
        super(SaleOrder, self - upsellable_orders)._compute_field_value(field)

    def _get_prepaid_service_lines_to_upsell(self):
        """ Retrieve all sols which need to display an upsell activity warning in the SO

            These SOLs should contain a product which has:
                - type="service",
                - service_policy="ordered_timesheet",
        """
        self.ensure_one()
        precision = self.env['decimal.precision'].precision_get('Product Unit of Measure')
        return self.order_line.filtered(lambda sol:
            sol.is_service
            and not sol.has_displayed_warning_upsell  # we don't want to display many times the warning each time we timesheet on the SOL
            and sol.product_id.service_policy == 'ordered_timesheet'
            and float_compare(
                sol.qty_delivered,
                sol.product_uom_qty * (sol.product_id.service_upsell_threshold or 1.0),
                precision_digits=precision
            ) > 0
        )

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
    remaining_hours_available = fields.Boolean(compute='_compute_remaining_hours_available', compute_sudo=True)
    remaining_hours = fields.Float('Remaining Hours on SO', compute='_compute_remaining_hours', compute_sudo=True, store=True)
    has_displayed_warning_upsell = fields.Boolean('Has Displayed Warning Upsell', copy=False)

    def name_get(self):
        res = super(SaleOrderLine, self).name_get()
        with_remaining_hours = self.env.context.get('with_remaining_hours')
        with_price_unit = self.env.context.get('with_price_unit')
        if with_remaining_hours or with_price_unit:
            names = dict(res)
            result = []
            uom_hour = with_remaining_hours and self.env.ref('uom.product_uom_hour')
            uom_day = with_remaining_hours and self.env.ref('uom.product_uom_day')
            sols_by_so_dict = with_price_unit and defaultdict(lambda: self.env[self._name])  # key: (sale_order_id, product_id), value: sale order line
            for line in self:
                if with_remaining_hours:
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
                            remaining_time = ' ({sign}{hours:02.0f}:{minutes:02.0f})'.format(
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
                        if with_price_unit:
                            names[line.id] = name
                    if not with_price_unit:
                        result.append((line.id, name))
                if with_price_unit:
                    sols_by_so_dict[line.order_id.id, line.product_id.id] += line

            if with_price_unit:
                for sols in sols_by_so_dict.values():
                    if len(sols) > 1:
                        result += [(
                            line.id,
                            '%s - %s' % (
                                names.get(line.id), format_amount(self.env, line.price_unit, line.currency_id))
                        ) for line in sols]
                    else:
                        result.append((sols.id, names.get(sols.id)))
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

    @api.depends('analytic_line_ids.project_id', 'project_id.pricing_type')
    def _compute_qty_delivered(self):
        super(SaleOrderLine, self)._compute_qty_delivered()

        lines_by_timesheet = self.filtered(lambda sol: sol.qty_delivered_method == 'timesheet')
        domain = lines_by_timesheet._timesheet_compute_delivered_quantity_domain()
        mapping = lines_by_timesheet.sudo()._get_delivered_quantity_by_analytic(domain)
        for line in lines_by_timesheet:
            line.qty_delivered = mapping.get(line.id or line._origin.id, 0.0)

    def _timesheet_compute_delivered_quantity_domain(self):
        """ Hook for validated timesheet in addionnal module """
        domain = [('project_id', '!=', False)]
        if self._context.get('accrual_entry_date'):
            domain += [('date', '<=', self._context['accrual_entry_date'])]
        return domain

    ###########################################
    # Service : Project and task generation
    ###########################################

    def _convert_qty_company_hours(self, dest_company):
        company_time_uom_id = dest_company.project_time_mode_id
        planned_hours = 0.0
        product_uom = self.product_uom
        if product_uom == self.env.ref('uom.product_uom_unit'):
            product_uom = self.env.ref('uom.product_uom_hour')
        if product_uom.category_id == company_time_uom_id.category_id:
            if product_uom != company_time_uom_id:
                planned_hours = product_uom._compute_quantity(self.product_uom_qty, company_time_uom_id)
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
            qty_to_invoice = mapping.get(line.id, 0.0)
            if qty_to_invoice:
                line.qty_to_invoice = qty_to_invoice
            else:
                prev_inv_status = line.invoice_status
                line.qty_to_invoice = qty_to_invoice
                line.invoice_status = prev_inv_status

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account
from . import account_move
from . import product
from . import project
from . import project_update
from . import sale_order
from . import res_config_settings
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
    task_id = fields.Many2one('project.task', string='Task', readonly=True)
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

    amount_untaxed_to_invoice = fields.Float("Amount to Invoice", digits=(16, 2), readonly=True, group_operator="sum")
    amount_untaxed_invoiced = fields.Float("Amount Invoiced", digits=(16, 2), readonly=True, group_operator="sum")
    expense_amount_untaxed_to_invoice = fields.Float("Amount to Re-invoice", digits=(16, 2), readonly=True, group_operator="sum")
    expense_amount_untaxed_invoiced = fields.Float("Amount Re-invoiced", digits=(16, 2), readonly=True, group_operator="sum")
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
                    sub.task_id as task_id,
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
                        SOL.task_id AS task_id,
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
                                        WHEN SOL.qty_delivered_method IN ('timesheet', 'manual', 'stock_move') AND SOL.is_service IS TRUE THEN (SOL.untaxed_amount_to_invoice / CASE COALESCE(S.currency_rate, 0) WHEN 0 THEN 1.0 ELSE S.currency_rate END)
                                        ELSE 0.0
                                    END AS amount_untaxed_to_invoice,
                                    CASE
                                        WHEN SOL.qty_delivered_method IN ('timesheet', 'manual', 'stock_move') AND SOL.is_service IS TRUE THEN (SOL.untaxed_amount_invoiced / CASE COALESCE(S.currency_rate, 0) WHEN 0 THEN 1.0 ELSE S.currency_rate END)
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
            <pivot string="Profitability Analysis" display_quantity="1" sample="1">
                <field name="project_id" type="row"/>
                <field name="amount_untaxed_to_invoice" type="measure"/>
                <field name="amount_untaxed_invoiced" type="measure"/>
                <field name="timesheet_cost" type="measure"/>
                <field name="margin" type="measure"/>
                <field name="timesheet_unit_amount" widget="timesheet_uom" type="measure"/>
            </pivot>
        </field>
    </record>

    <record id="project_profitability_report_view_graph" model="ir.ui.view">
        <field name="name">project.profitability.report.graph</field>
        <field name="model">project.profitability.report</field>
        <field name="arch" type="xml">
            <graph string="Profitability Analysis" sample="1" js_class="hr_timesheet_graphview">
                <field name="project_id"/>
                <field name="product_id"/>
                <field name="amount_untaxed_to_invoice" type="measure"/>
                <field name="amount_untaxed_invoiced" type="measure"/>
                <field name="timesheet_cost" type="measure"/>
                <field name="other_revenues" type="measure"/>
                <field name="margin" type="measure"/>
                <field name="timesheet_unit_amount" widget="timesheet_uom" type="measure"/>
             </graph>
         </field>
    </record>

    <record id="project_profitability_report_view_tree" model="ir.ui.view">
        <field name="name">project.profitability.report.view.tree</field>
        <field name="model">project.profitability.report</field>
        <field name="arch" type="xml">
            <tree string="Profitability Analysis">
                <field name="project_id"/>
                <field name="user_id" optional="hide"/>
                <field name="task_id" optional="hide"/>
                <field name="partner_id" optional="hide"/>
                <field name="analytic_account_id" optional="hide"/>
                <field name="line_date" optional="show"/>
                <field name="currency_id" optional="show" invisible="1"/>
                <field name="amount_untaxed_to_invoice" optional="show" sum="Sum of Amount to Invoice"
                       widget="monetary"/>
                <field name="amount_untaxed_invoiced" optional="hide" sum="Sum of Amount Invoiced" widget="monetary"/>
                <field name="other_revenues" optional="hide" widget="monetary"/>
                <field name="timesheet_cost" optional="show" sum="Sum of Timesheet Cost" widget="monetary"/>
                <field name="expense_cost" optional="hide" widget="monetary"/>
                <field name="margin" optional="show" sum="Sum of Margin" widget="monetary"/>
            </tree>
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
                    <filter string="Task" name="group_by_task_id" context="{'group_by':'task_id'}"/>
                    <filter string="Customer" name="group_by_partner_id" context="{'group_by':'partner_id'}"/>
                    <filter string="Company" name="group_by_company" context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
                    <filter string="Date" name="group_by_line_date" context="{'group_by':'line_date'}"/>
                </group>
            </search>
        </field>
    </record>

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

## File: static\description\icon.svg

```svg
<svg id="Layer_1" data-name="Layer 1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 70 70">
  <defs>
    <mask id="mask" x="0" y="0" width="70" height="70" maskUnits="userSpaceOnUse">
      <g id="b">
        <path id="a" d="M4,0H65c4,0,5,1,5,5V65c0,4-1,5-5,5H4c-3,0-4-1-4-5V5C0,1,1,0,4,0Z" fill="#fff" fill-rule="evenodd"/>
      </g>
    </mask>
    <linearGradient id="linear-gradient" x1="-1349.79" y1="477.94" x2="-1350.79" y2="476.94" gradientTransform="matrix(70, 0, 0, -70, 94554.99, 33455.73)" gradientUnits="userSpaceOnUse">
      <stop offset="0" stop-color="#da956b"/>
      <stop offset="1" stop-color="#cc7039"/>
    </linearGradient>
  </defs>
  <g mask="url(#mask)">
    <g>
      <path d="M0,0H70V70H0Z" fill-rule="evenodd" fill="url(#linear-gradient)"/>
      <path d="M4,1H65c2.67,0,4.33.67,5,2V0H0V3C.67,1.67,2,1,4,1Z" fill="#fff" fill-opacity="0.38" fill-rule="evenodd"/>
      <path d="M44.25,69H4c-2,0-4-.15-4-4.08V29L13,16h3V43L31.33,27.71,32,28l7-7,10.86,1.83L53,35.66,36.2,52.47l-.1.7L47.93,41.34l4.19,1.57L51,44c2.55.46,2.8,1.59,4.33,3.57a14.92,14.92,0,0,0,3-.14l-1.22,1.21L53.94,59.32Z" fill="#393939" fill-rule="evenodd" opacity="0.32" style="isolation: isolate"/>
      <path d="M4,69H65c2.67,0,4.33-1,5-3v4H0V66A3.92,3.92,0,0,0,4,69Z" fill-opacity="0.38" fill-rule="evenodd"/>
      <g>
        <g opacity="0.4">
          <path d="M21.4,46a1,1,0,0,0,1.49,0l7.19-8.17,4.86,5.52a1,1,0,0,0,1.35.13.53.53,0,0,0,.13-.13L46,32.53l5,5.13L53,20,37,23l5,5-6.3,7.25-4.87-5.53a1,1,0,0,0-1.48,0L18.92,41.42a1.31,1.31,0,0,0,0,1.68Z"/>
          <path d="M34.24,54.91a12.79,12.79,0,0,1,.17-2.63H14.08V18.65A1.12,1.12,0,0,0,13,17.46h-1.5a1.12,1.12,0,0,0-1,1.19v35.8a1.12,1.12,0,0,0,1,1.19H34.31C34.28,55.4,34.25,55.16,34.24,54.91Z"/>
          <path d="M49.15,50.29v4.85a.34.34,0,0,1-.1.25.33.33,0,0,1-.25.1H45.33a.32.32,0,0,1-.25-.1.3.3,0,0,1-.1-.25v-.69a.33.33,0,0,1,.1-.25.36.36,0,0,1,.25-.1h2.43V50.29a.34.34,0,0,1,.35-.35h.69a.33.33,0,0,1,.25.1A.34.34,0,0,1,49.15,50.29Zm4.51,3.81a5.86,5.86,0,0,0-.79-3A6,6,0,0,0,50.72,49a5.94,5.94,0,0,0-5.92,0,5.91,5.91,0,0,0-2.15,2.15,5.94,5.94,0,0,0,0,5.92,5.84,5.84,0,0,0,2.15,2.15,5.94,5.94,0,0,0,5.92,0,5.91,5.91,0,0,0,2.15-2.15A5.83,5.83,0,0,0,53.66,54.1Zm.41-5.44,1.08-.82a.42.42,0,0,1,.57.08l.73,1a.4.4,0,0,1-.08.57l-1.17.88A8.35,8.35,0,0,1,55,58.28a8.26,8.26,0,0,1-7.21,4.15,8.17,8.17,0,0,1-4.18-1.11,8.26,8.26,0,0,1-3-3,8.36,8.36,0,0,1,0-8.36,8.31,8.31,0,0,1,3-3,8.11,8.11,0,0,1,3.16-1.06V45h-.81a.41.41,0,0,1-.41-.41v-.81a.41.41,0,0,1,.41-.4H50a.4.4,0,0,1,.41.4v.81A.41.41,0,0,1,50,45h-.81v.93a8.19,8.19,0,0,1,2.76,1A8.31,8.31,0,0,1,54.07,48.66Z" fill-rule="evenodd"/>
        </g>
        <g>
          <path d="M23.4,44a1,1,0,0,0,1.49,0l7.19-8.17,4.86,5.52a1,1,0,0,0,1.35.13.53.53,0,0,0,.13-.13L48,30.53l5,5.13L55,18,39,21l5,5-6.3,7.25-4.87-5.53a1,1,0,0,0-1.48,0L20.92,39.42a1.31,1.31,0,0,0,0,1.68Z" fill="#fff"/>
          <path d="M36.24,52.91a12.79,12.79,0,0,1,.17-2.63H16.08V16.65a1.12,1.12,0,0,0-1-1.19h-1.5a1.12,1.12,0,0,0-1,1.19v35.8a1.12,1.12,0,0,0,1,1.19H36.31C36.28,53.4,36.25,53.16,36.24,52.91Z" fill="#fff"/>
          <path d="M51.15,48.29v4.85a.34.34,0,0,1-.1.25.33.33,0,0,1-.25.1H47.33a.32.32,0,0,1-.25-.1.3.3,0,0,1-.1-.25v-.69a.33.33,0,0,1,.1-.25.36.36,0,0,1,.25-.1h2.43V48.29a.34.34,0,0,1,.35-.35h.69a.33.33,0,0,1,.25.1A.34.34,0,0,1,51.15,48.29Zm4.51,3.81a5.86,5.86,0,0,0-.79-3A6,6,0,0,0,52.72,47a5.94,5.94,0,0,0-5.92,0,5.91,5.91,0,0,0-2.15,2.15,5.94,5.94,0,0,0,0,5.92,5.84,5.84,0,0,0,2.15,2.15,5.94,5.94,0,0,0,5.92,0,5.91,5.91,0,0,0,2.15-2.15A5.83,5.83,0,0,0,55.66,52.1Zm.41-5.44,1.08-.82a.42.42,0,0,1,.57.08l.73,1a.4.4,0,0,1-.08.57l-1.17.88A8.35,8.35,0,0,1,57,56.28a8.26,8.26,0,0,1-7.21,4.15,8.17,8.17,0,0,1-4.18-1.11,8.26,8.26,0,0,1-3-3,8.36,8.36,0,0,1,0-8.36,8.31,8.31,0,0,1,3-3,8.11,8.11,0,0,1,3.16-1.06V43h-.81a.41.41,0,0,1-.41-.41v-.81a.41.41,0,0,1,.41-.4H52a.4.4,0,0,1,.41.4v.81A.41.41,0,0,1,52,43h-.81v.93a8.19,8.19,0,0,1,2.76,1A8.31,8.31,0,0,1,56.07,46.66Z" fill="#fff" fill-rule="evenodd"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\js\so_line_one2many.js

```javascript
odoo.define('sale_timesheet.so_line_many2one', function (require) {
"use strict";

const fieldRegistry = require('web.field_registry');
const { FieldOne2Many, FieldMany2One } = require('web.relational_fields');

const SoLineOne2Many = FieldOne2Many.extend({
    _onFieldChanged: function (ev) {
        if (
            ev.data.changes &&
            ev.data.changes.hasOwnProperty('timesheet_ids') &&
            ev.data.changes.timesheet_ids.operation === 'UPDATE' &&
            ev.data.changes.timesheet_ids.data &&
            ev.data.changes.timesheet_ids.data.hasOwnProperty('so_line')) {
            const line = this.value.data.find(line => {
                return line.id === ev.data.changes.timesheet_ids.id;
            });
            if (!line.is_so_line_edited) {
                ev.data.changes.timesheet_ids.data.is_so_line_edited = true;
            }
        }
        this._super.apply(this, arguments);
    }
});

const SoLineMany2one = FieldMany2One.extend({
    /**
     * @override
     *
     * When the user manually changes the field, we need to change the is_so_line_edited field in this model
     * to know the changes is manual and not via a compute method.
     */
    _onFieldChanged(ev) {
        if (ev.data.changes && ev.data.changes.hasOwnProperty('so_line') && !ev.data.changes.so_line.is_so_line_edited) {
            ev.data.changes.is_so_line_edited = true;
        }
        this._super.apply(this, arguments);
    },
});


fieldRegistry.add('so_line_one2many', SoLineOne2Many);
fieldRegistry.add('so_line_many2one', SoLineMany2one);

return SoLineOne2Many;

});

```

## File: static\src\xml\sale_project_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <button t-name="SaleProjectKanbanView.buttons" type="button" class="btn btn-secondary o_create_sale_order">
        Create Sales Order
    </button>

    <t t-inherit="project.ProjectRightPanel" t-inherit-mode="extension" owl="1">
        <xpath expr="//t[@t-call='project.StatButtonsSection']" position="after">
            <t t-call="sale_timesheet.SoldSection"/>
            <t t-call="sale_timesheet.TotalSoldSection"/>
            <t t-call="sale_timesheet.ProfitabilitySection"/>
        </xpath>
    </t>

    <t t-name="sale_timesheet.SoldSection" owl="1">
        <t t-set="sold_items" t-value="state.data.sold_items"/>
        <t t-if="sold_items.number_sols and sold_items.allow_billable"> 
            <div name="sold" class="o_rightpanel_section">
                <div class="o_rightpanel_header">
                    <div class="o_rightpanel_title">
                        Sold
                    </div>
                </div>

                <div class="o_rightpanel_data">
                    <div t-foreach="sold_items.data" t-as="sold_item" t-key="sold_item.name" class="o_rightpanel_data_row d-flex justify-content-between">
                        <span class="o_rightpanel_left_col o_rightpanel_left_text" t-attf-class="o_color_{{sold_item.color}}"><t t-esc="sold_item.name"/></span>
                        <span t-attf-class="o_rightpanel_right_col o_color_{{sold_item.color}}"><b><t t-esc="sold_item.value"/></b></span>
                    </div>
                </div>
            </div>
        </t>
    </t>

    <t t-name="sale_timesheet.TotalSoldSection" owl="1">
        <t t-set="sold_items" t-value="state.data.sold_items"/>
        <t t-if="sold_items.allow_billable"> 
            <div name="total_sold" class="o_rightpanel_section">
                <div class="o_rightpanel_header">
                    <div class="o_rightpanel_title">
                        <span class="o_rightpanel_left_col o_rightpanel_left_text"> Total Sold </span>
                    </div>
                    <span class="o_rightpanel_right_col font-weight-bold float-right"><t t-esc="formatFloat(sold_items.total_sold)"/> <t t-esc="sold_items.company_unit_name"/></span>
                </div>
                <div class="o_rightpanel_data">
                    <div class="o_rightpanel_data_row">
                        <span class="o_rightpanel_left_col o_rightpanel_left_text">Effective</span>
                        <span class="o_rightpanel_right_col"><t t-esc="formatFloat(sold_items.effective_sold)"/> <t t-esc="sold_items.company_unit_name"/></span>
                    </div>
                    <div class="o_rightpanel_data_row" t-if="sold_items.planned_sold > 0 and sold_items.allow_forecast">
                        <span class="o_rightpanel_left_col o_rightpanel_left_text">Planned</span>
                        <span class="o_rightpanel_right_col"><t t-esc="formatFloat(sold_items.planned_sold)"/> <t t-esc="sold_items.company_unit_name"/></span>
                    </div>
                    <div class="o_rightpanel_data_row">
                        <span class="o_rightpanel_left_col o_rightpanel_left_text">Remaining</span>
                        <span class="o_rightpanel_right_col" t-attf-class="o_color_{{sold_items.remaining.color}}"><t t-esc="formatFloat(sold_items.remaining['value'])"/> <t t-esc="sold_items.company_unit_name"/></span>
                    </div>
                </div>
            </div>
        </t>
    </t>
    
    <t t-name="sale_timesheet.ProfitabilitySection" owl="1">
        <t t-if="state.data.user.is_project_user &amp;&amp; state.data.profitability_items.data.length > 0 &amp;&amp; state.data.analytic_account_id">
            <t t-set="profitability_items" t-value="state.data.profitability_items"/>
            <div name="profitability" class="o_rightpanel_section">
                <div class="o_rightpanel_header">
                    <div class="o_rightpanel_title">
                        Profitability
                    </div>
                </div>

                <div class="o_rightpanel_data">
                    <div t-foreach="profitability_items.data" t-as="profitability" t-key="profitability.name" class="o_rightpanel_data_row">
                        <span class="o_rightpanel_left_col o_rightpanel_left_text"><t t-esc="profitability.name"/></span>
                        <t t-if="profitability.color">
                            <span t-attf-class="o_rightpanel_right_col o_color_{{profitability.color}}"><b><t t-esc="profitability.value"/></b></span>
                        </t>
                        <t t-else="">
                            <span class="o_rightpanel_right_col"><t t-esc="profitability.value"/></span>
                        </t>
                    </div>
                </div>
            </div>
        </t>
    </t>

</templates>

```

## File: views\account_invoice_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="action_timesheet_from_invoice" model="ir.actions.act_window">
        <field name="name">Timesheets</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">account.analytic.line</field>
        <field name="view_mode">tree,form,graph,pivot,kanban</field>
        <field name="context">{'create': False, 'edit': False, 'delete': False}</field>
        <field name="domain">[('timesheet_invoice_id', '=', active_id)]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No activities found
            </p>
            <p>
                Track your working hours by projects every day and invoice this time to your customers.
            </p>
        </field>
    </record>

    <record id="action_timesheet_from_invoice_view_tree" model="ir.actions.act_window.view">
        <field name="sequence" eval="1"/>
        <field name="view_mode">tree</field>
        <field name="view_id" ref="hr_timesheet.hr_timesheet_line_tree"/>
        <field name="act_window_id" ref="action_timesheet_from_invoice"/>
    </record>

    <record id="action_timesheet_from_invoice_view_form" model="ir.actions.act_window.view">
        <field name="sequence" eval="4"/>
        <field name="view_mode">form</field>
        <field name="view_id" ref="hr_timesheet.hr_timesheet_line_form"/>
        <field name="act_window_id" ref="action_timesheet_from_invoice"/>
    </record>

    <record id="action_timesheet_from_invoice_view_kanban" model="ir.actions.act_window.view">
        <field name="sequence" eval="5"/>
        <field name="view_mode">kanban</field>
        <field name="view_id" ref="hr_timesheet.view_kanban_account_analytic_line"/>
        <field name="act_window_id" ref="action_timesheet_from_invoice"/>
    </record>

    <record id="action_timesheet_from_invoice_view_pivot" model="ir.actions.act_window.view">
        <field name="sequence" eval="6"/>
        <field name="view_mode">pivot</field>
        <field name="view_id" ref="hr_timesheet.view_hr_timesheet_line_pivot"/>
        <field name="act_window_id" ref="action_timesheet_from_invoice"/>
    </record>

    <record id="action_timesheet_from_invoice_view_graph" model="ir.actions.act_window.view">
        <field name="sequence" eval="7"/>
        <field name="view_mode">graph</field>
        <field name="view_id" ref="hr_timesheet.view_hr_timesheet_line_graph"/>
        <field name="act_window_id" ref="action_timesheet_from_invoice"/>
    </record>

    <record id="account_invoice_view_form_inherit_sale_timesheet" model="ir.ui.view">
        <field name="name">account.invoice.form.inherit.timesheet</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <field name="timesheet_count" invisible="1"/>
                <button name="%(sale_timesheet.action_timesheet_from_invoice)d" type="action" class="oe_stat_button" icon="fa-clock-o" attrs="{'invisible': [('timesheet_count', '=', 0)]}" groups="hr_timesheet.group_hr_timesheet_user">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="timesheet_total_duration" class="mr4" widget="statinfo" nolabel="1"/>
                            <field name="timesheet_encode_uom_id" options="{'no_open': True}"/>
                        </span>
                        <span class="o_stat_text">Recorded</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>

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
                <xpath expr="//field[@name='task_id']" position="after">
                    <field name="so_line" groups="sales_team.group_sale_salesman"/>
                </xpath>
                <xpath expr="//filter[@name='month']" position="before">
                    <filter name="billable_time" string="Billed on Timesheets" domain="[('timesheet_invoice_type', '=', 'billable_time')]"/>
                    <filter name="billable_fixed" string="Billed at a Fixed Price" domain="[('timesheet_invoice_type', '=', 'billable_fixed')]"/>
                    <filter name="non_billable" string="Non Billable" domain="[('timesheet_invoice_type', '=', 'non_billable')]"/>
                    <separator/>
                </xpath>
                <xpath expr="//filter[@name='groupby_employee']" position="after">
                    <filter string="Sales Order Item" name="groupby_sale_order_item" domain="[]" context="{'group_by': 'so_line'}" groups="sales_team.group_sale_salesman"/>
                    <filter string="Invoice" name="groupby_invoice" domain="[]" context="{'group_by': 'timesheet_invoice_id'}"/>
                    <filter string="Billable Type" name="groupby_timesheet_invoice_type" domain="[]" context="{'group_by': 'timesheet_invoice_type'}"/>
                </xpath>
            </field>
    </record>

    <record id="hr_timesheet_line_tree_inherit" model="ir.ui.view">
        <field name="name">account.analytic.line.tree.inherit</field>
        <field name="model">account.analytic.line</field>
        <field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
        <field name="inherit_id" ref="hr_timesheet.hr_timesheet_line_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//tree/field[@name='unit_amount']" position="before">
                <field name="commercial_partner_id" invisible="1"/>
                <field name="is_so_line_edited" invisible="1"/>
                <field name="so_line" widget="so_line_many2one" optional="hide" options="{'no_create': True}" context="{'create': False, 'edit': False, 'delete': False}"/>
            </xpath>
        </field>
    </record>

    <record id="hr_timesheet_line_form_inherit" model="ir.ui.view">
        <field name="name">account.analytic.line.form.inherit</field>
        <field name="model">account.analytic.line</field>
        <field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
        <field name="inherit_id" ref="hr_timesheet.hr_timesheet_line_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='task_id']" position="after">
                <field name="commercial_partner_id" invisible="1"/>
                <field name="is_so_line_edited" invisible="1" />
                <field name="so_line" widget="so_line_many2one" options='{"no_create": True}' context="{'create': False, 'edit': False, 'delete': False}"/>
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
        <field name="name">Timesheets by Billing Type</field>
        <field name="res_model">account.analytic.line</field>
        <field name="view_mode">tree,form,pivot,graph,kanban</field>
        <field name="domain">[('project_id', '!=', False)]</field>
        <field name="context">{
            'search_default_groupby_timesheet_invoice_type': 1,
            'pivot_row_groupby': ['date:month'],
        }</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No data yet!
            </p>
            <p>Review your timesheets by billing type and make sure your time is billable.</p>
        </field>
        <field name="search_view_id" ref="hr_timesheet.hr_timesheet_line_search"/>
    </record>

    <record id="view_hr_timesheet_line_pivot_billing_rate" model="ir.ui.view">
        <field name="name">account.analytic.line.pivot.billing.rate</field>
        <field name="model">account.analytic.line</field>
        <field name="arch" type="xml">
            <pivot string="Timesheets" sample="1">
                <field name="timesheet_invoice_type" type="col"/>
                <field name="unit_amount" type="measure" widget="timesheet_uom"/>
                <field name="amount" string="Timesheet Costs"/>
            </pivot>
        </field>
    </record>

    <record id="timesheet_action_view_report_by_billing_rate_form" model="ir.actions.act_window.view">
        <field name="sequence" eval="10"/>
        <field name="view_mode">form</field>
        <field name="view_id" ref="hr_timesheet.hr_timesheet_line_form"/>
        <field name="act_window_id" ref="timesheet_action_billing_report"/>
    </record>

    <record id="timesheet_action_view_report_by_billing_rate_kanban" model="ir.actions.act_window.view">
        <field name="sequence" eval="10"/>
        <field name="view_mode">kanban</field>
        <field name="view_id" ref="hr_timesheet.view_kanban_account_analytic_line"/>
        <field name="act_window_id" ref="timesheet_action_billing_report"/>
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

    <record id="timesheet_action_view_report_by_billing_rate_tree" model="ir.actions.act_window.view">
        <field name="sequence" eval="8"/>
        <field name="view_mode">tree</field>
        <field name="view_id" ref="hr_timesheet.hr_timesheet_line_tree"/>
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
                <attribute name="attrs">{'invisible': [('type', '=', 'service')]}</attribute>
            </xpath>
            <xpath expr="//field[@name='invoice_policy']" position="after">
                <field name="service_policy" string="Invoicing Policy" attrs="{'invisible': [('type', '!=', 'service')], 'required': [('type', '=', 'service')]}"/>
            </xpath>
            <field name="product_tooltip" position="after">
                <label for="product_tooltip" string="" attrs="{'invisible': ['|', ('type', '!=', 'service'), ('service_policy', '!=', 'ordered_timesheet')]}"/>
                <div attrs="{'invisible': ['|', ('type', '!=', 'service'), ('service_policy', '!=', 'ordered_timesheet')]}" class="font-italic text-muted">
                    Warn the salesperson for an upsell when work done exceeds
                    <field name="service_upsell_threshold" widget="percentage" class="oe_inline"/>
                    of hours sold. (<field name="service_upsell_threshold_ratio" class="oe_inline"/>)
                </div>
            </field>
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
        <field name="name">Services</field>
        <field name="res_model">product.template</field>
        <field name="view_mode">tree,form</field>
        <field name="search_view_id" ref="sale_timesheet.product_template_view_search_sale_timesheet"/>
        <field name="context">{'search_default_services': 1, 'default_detailed_type': 'service'}</field>
    </record>

</odoo>

```

## File: views\project_sharing_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="project_sharing_inherit_project_task_view_form" model="ir.ui.view">
        <field name="name">project.task.form.inherit.timesheet</field>
        <field name="model">project.task</field>
        <field name="inherit_id" ref="project.project_sharing_project_task_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='timesheet_ids']/tree" position="attributes">
                <attribute name="decoration-muted">timesheet_invoice_id != False</attribute>
            </xpath>
            <xpath expr="//field[@name='timesheet_ids']/tree/field[@name='unit_amount']" position="before">
                <field name="timesheet_invoice_id" invisible="1"/>
                <field name="so_line"
                    attrs="{'column_invisible': [('parent.allow_billable', '=', False)]}"
                    context="{'with_remaining_hours': True, 'with_price_unit': True}" options="{'no_create': True, 'no_open': True}"
                    optional="hide"/>
            </xpath>
            <xpath expr="//field[@name='remaining_hours']" position="after">
                <field name="allow_billable" invisible="1" />
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
            <xpath expr="//button[@name='action_view_so']" position="attributes">
                <attribute name="attrs">{'invisible': ['|', ('allow_billable', '=', False), ('sale_order_id', '=', False)]}</attribute>
            </xpath>
            <xpath expr="//button[@name='%(project.action_project_task_burndown_chart_report)d']" position="after">
                <button name="action_billable_time_button" type="object" class="oe_stat_button" icon="fa-clock-o" attrs="{'invisible': ['|', ('allow_timesheets', '=', False), ('analytic_account_id', '=', False)]}" groups="hr_timesheet.group_hr_timesheet_approver">
                    <div class="o_field_widget o_stat_info">
                        <div class="oe_inline">
                            <span class="o_stat_value mr-1">
                                <field name="billable_percentage" widget="statinfo" nolabel="1"/>%
                            </span>
                        </div>
                        <span class="o_stat_text">
                        Billable Time
                        </span>
                    </div>
                </button>
            </xpath>
            <xpath expr="//header" position="inside">
                <!-- To be removed in master -->
                <button name="action_make_billable" string="Create Sales Order" type="object" groups="sales_team.group_sale_salesman" invisible="1"/>
            </xpath>
            <xpath expr="//page[@name='settings']" position="after">
                <page name="billing_employee_rate" string="Invoicing" attrs="{'invisible': ['|', ('allow_billable', '=', False), ('partner_id', '=', False)]}">
                    <group>
                        <group>
                            <field name="display_create_order" invisible="1"/>
                            <field name="pricing_type" invisible="1" widget="radio"/>
                            <field name="timesheet_product_id" string="Default Service" invisible="1" context="{'default_detailed_type': 'service', 'default_service_policy': 'delivered_timesheet', 'default_service_type': 'timesheet'}"/>
                            <field name="sale_order_id" invisible="1" options="{'no_create': True, 'no_edit': True, 'delete': False, 'no_open': True}"/>
                            <field name="sale_line_id" string="Default Sales Order Item" options="{'no_create': True, 'no_edit': True, 'delete': False, 'no_open': True}"/>
                        </group>
                    </group>
                    <field name="sale_line_employee_ids">
                        <tree editable="bottom">
                            <field name="company_id" invisible="1"/>
                            <field name="partner_id" invisible="1"/>
                            <field name="employee_id" options="{'no_create': True}"/>
                            <field name="sale_line_id" attrs="{'required': True}" options="{'no_create': True}"/>
                            <field name="price_unit" widget="monetary" force_save="1" options="{'currency_field': 'currency_id'}"/>
                            <field name="cost"/>
                            <field name="is_cost_changed" invisible="1"/>
                            <field name="currency_id" invisible="1"/>
                            <field name="cost_currency_id" invisible="1"/>
                        </tree>
                    </field>
                </page>
            </xpath>
            <xpath expr="//div[@id='timesheet_settings']" position="after">
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
            </xpath>
        </field>
    </record>

    <record id="project_project_view_form_salesman" model="ir.ui.view">
        <field name="name">project.project.form.inherit.salesman</field>
        <field name="model">project.project</field>
        <field name="inherit_id" ref="sale_timesheet.project_project_view_form"/>
        <field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
        <field name="arch" type="xml">
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
            <field name="allow_timesheets" position="before">
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
                <field name="sale_order_id" invisible="1"/>
                <field name="pricing_type" invisible="1"/>
            </xpath>
            <xpath expr="//div[hasclass('o_kanban_manage_reporting')]" position="inside">
                <div role="menuitem" t-if="record.rating_active.raw_value" groups="project.group_project_manager">
                   <a name="action_view_all_rating" type="object">
                    Customer Ratings
                    </a>
                </div> 
            </xpath>
            <xpath expr="//div[hasclass('o_kanban_manage_view')]" position="inside">
                <div t-if="record.allow_billable.raw_value and record.sale_order_id.raw_value and record.pricing_type.raw_value != 'task_rate'"
                    role="menuitem"
                    groups="sales_team.group_sale_salesman_all_leads">
                    <a name="action_view_so" type="object">Sales Order</a>
                </div>
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
                <xpath expr="//field[@name='user_ids']" position="after">
                    <field name="is_project_map_empty" invisible="1"/>
                    <field name="has_multi_sol" invisible="1"/>
                </xpath>
                 <xpath expr="//field[@name='partner_phone']" position="after">
                    <field name="pricing_type" invisible="1"/>
                </xpath>
                <xpath expr="//field[@name='timesheet_ids']" position="attributes">
                    <attribute name="widget">so_line_one2many</attribute>
                </xpath>
                <xpath expr="//field[@name='timesheet_ids']/tree" position="inside">
                    <field name="is_so_line_edited" invisible="1" />
                </xpath>
                <xpath expr="//field[@name='timesheet_ids']/tree/field[@name='unit_amount']" position="before">
                    <field name="timesheet_invoice_id" invisible="1"/>
                    <field name="so_line"
                        attrs="{'column_invisible': [('parent.allow_billable', '=', False)]}"
                        context="{'with_remaining_hours': True, 'with_price_unit': True}" options="{'no_create': True, 'no_open': True}"
                        domain="[('is_service', '=', True), ('order_partner_id', 'child_of', parent.commercial_partner_id), ('is_expense', '=', False), ('state', 'in', ['sale', 'done'])]"
                        optional="hide"/>
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

        <record id="project_task_view_form_inherit_sale_timesheet_editable" model="ir.ui.view">
            <field name="name">project.task.form.view.form.inherit.sale.timesheet.editable</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project_task_view_form_inherit_sale_timesheet"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='timesheet_ids']/tree/field[@name='so_line']" position="attributes">
                    <attribute name="options">{'no_create': True}</attribute>
                </xpath>
            </field>
            <field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman'))]"/>
        </record>

        <record id="view_task_form2_inherit_sale_timesheet" model="ir.ui.view">
            <field name="name">view.task.form2.inherit</field>
            <field name="model">project.task</field>
            <field name="inherit_id" ref="project.view_task_form2"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='sale_line_id']" position="attributes">
                    <attribute name="context">{'with_remaining_hours': True, 'with_price_unit': True}</attribute>
                    <attribute name="attrs">
                        {'invisible': ['|', ('allow_billable', '=', False), ('partner_id', '=', False)]}
                    </attribute>
                </xpath>
                <xpath expr="//field[@name='sale_order_id']" position="attributes">
                    <attribute name="invisible">1</attribute>
                </xpath>
            </field>
        </record>
</odoo>

```

## File: views\project_update_templates.xml

```xml
<?xml version="1.0"?>
<odoo>
    <template id="project_update_default_description" inherit_id="project.project_update_default_description">
        <!--As this template is rendered in an html field, the spaces may be interpreted as nbsp while editing. -->
        <xpath expr="//div[@name='milestone']" position="before">
<br/>
<div t-if="show_sold">
<h3 style="font-weight: bolder"><u>Sold</u></h3>
<table class="table table-bordered">
<tbody>
<thead>
<td style="font-weight: bolder">Service</td>
<td style="font-weight: bolder">Sold</td>
<td style="font-weight: bolder">Effective</td>
<td style="font-weight: bolder">Remaining</td>
</thead>
<tr t-foreach="services['data']" t-as="service">
<t t-set="is_unit" t-value="service['is_unit']"/>
<td t-attf-class="#{ 'font-italic' if is_unit else ''}"><t t-esc="service['name']"/></td>
<td t-attf-class="#{ 'font-italic' if is_unit else ''}" style="text-align: right; vertical-align: middle;"><t t-esc="format_lang(service['sold_value'], 1)"/> <t t-esc="service['unit']"/></td>
<td t-attf-class="#{ 'font-italic' if is_unit else ''}" style="text-align: right; vertical-align: middle;"><t t-esc="format_lang(service['effective_value'], 1)"/> <t t-esc="service['unit']"/></td>
<td t-attf-class="#{ 'font-italic' if is_unit else ''}" style="text-align: right; vertical-align: middle;"><t t-esc="format_lang(service['remaining_value'], 1)"/> <t t-esc="service['unit']"/></td>
</tr>
<tfoot>
<td style="font-weight: bolder; text-align: right">Total</td>
<td style="font-weight: bolder; text-align: right; vertical-align: middle;"><t t-esc="format_lang(services['total_sold'], 1)"/> <t t-esc="services['company_unit_name']"/></td>
<td style="font-weight: bolder; text-align: right; vertical-align: middle;"><t t-esc="format_lang(services['total_effective'], 1)"/> <t t-esc="services['company_unit_name']"/></td>
<td style="font-weight: bolder; text-align: right; vertical-align: middle;"><t t-esc="format_lang(services['total_remaining'], 1)"/> <t t-esc="services['company_unit_name']"/></td>
</tfoot>
</tbody>
</table>
<br/>
</div>        
        
<div name="profitability" t-if="show_profitability">
<t t-if="project.analytic_account_id and project.allow_billable and user.has_group('project.group_project_manager')" name="costs">
<h3 style="font-weight: bolder"><u>Profitability</u></h3>
The cost of the project is now at <t t-esc="profitability['costs']"/>, for a revenue of <t t-esc="profitability['revenues']"/>, leading to a
<span>
<font t-if="profitability['margin'] &gt; 0"  style="color: rgb(0, 128, 0)">
<b><t t-esc="profitability['margin_formatted']"/></b>
</font>
<font t-elif="profitability['margin'] &lt; 0" style="color: rgb(128, 0, 0)">
<b><t t-esc="profitability['margin_formatted']"/></b>
</font>
<t t-else="" t-esc="profitability['margin_formatted']"/>
</span> margin (<t t-esc="profitability['margin_percentage']"/>%).
</t>
</div>
        </xpath>
    </template>

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
                    <div class="col-12 col-lg-6 o_setting_box" name="invoice_policy">
                        <div class="o_setting_left_pane">
                            <field name="invoice_policy" widget="upgrade_boolean"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="invoice_policy"/>
                            <div class="text-muted">
                                Timesheets taken into account when invoicing your time
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
                    <xpath expr="//button[@name='action_view_invoice']" position="before">
                        <field name="timesheet_count" invisible="1"/>
                        <button type="object"
                           name="action_view_timesheet"
                           class="oe_stat_button"
                           icon="fa-clock-o"
                           attrs="{'invisible': [('timesheet_count', '=', 0)]}"
                           groups="hr_timesheet.group_hr_timesheet_user">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_value">
                                    <field name="timesheet_total_duration" class="mr4" widget="statinfo" nolabel="1"/>
                                    <field name="timesheet_encode_uom_id" options="{'no_open': True}"/>
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

    <template id="sale_order_portal_content_inherit" inherit_id="sale.sale_order_portal_template">
        <xpath expr="//li[.//a[@id='print_invoice_report']]" position="after">
            <li t-if="sale_order.timesheet_count > 0 and sale_order.state in ('sale', 'done')" class="list-group-item flex-grow-1">
                <div class="btn-toolbar flex-sm-nowrap justify-content-center">
                    <div class="btn-group">
                        <a t-att-href="'/my/timesheets?search_in=so&amp;search=%s' % sale_order.name">View Timesheets</a>
                    </div>
                </div>
            </li>
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
                            <span class="text-muted font-weight-normal">
                                <t t-if="is_uom_day">
                                    (<span t-esc="timesheets._timesheet_convert_sol_uom(sol, 'uom.product_uom_day')" t-options='{"widget": "timesheet_uom"}'></span> Days Ordered, <span t-esc="timesheets._convert_hours_to_days(sol.remaining_hours)" t-options='{"widget": "timesheet_uom"}'></span> Days Remaining)
                                </t>
                                <t t-else="">
                                    (<span t-esc="timesheets._timesheet_convert_sol_uom(sol, 'uom.product_uom_hour')" t-options='{"widget": "float_time"}'></span> Hours Ordered, <span t-esc="sol.remaining_hours" t-options='{"widget": "float_time"}'></span> Hours Remaining)
                                </t>
                            </span>
                        </t>
                    </t>
                </th>
                <th colspan="1" class="text-right text-muted font-weight-normal">
                    <t t-if="is_uom_day">
                        Total: <span t-esc="timesheets._convert_hours_to_days(hours_spent)" t-options='{"widget": "timesheet_uom"}'/>
                    </t>
                    <t t-else="">
                        Total: <span t-esc="hours_spent" t-options='{"widget": "float_time"}'/>
                    </t>
                </th>
            </t>
            <t t-elif="groupby == 'so'">
                <t t-set="so" t-value="timesheets[0].order_id"/>
                <th colspan="6">
                    <t t-if="so">
                        <em class="font-weight-normal text-muted">Timesheets for sales order:</em>
                        <span t-field="so.display_name"/>
                    </t>
                </th>
                <th colspan="1" class="text-right text-muted">
                    <t t-if="is_uom_day">
                        Total: <span t-esc="timesheets._convert_hours_to_days(hours_spent)" t-options='{"widget": "timesheet_uom"}'/>
                    </t>
                    <t t-else="">
                        Total: <span t-esc="hours_spent" t-options='{"widget": "float_time"}'/>
                    </t>
                </th>
            </t>
            <t t-elif="groupby == 'invoice'">
                <t t-set="invoice" t-value="timesheets.timesheet_invoice_id"/>
                <th colspan="6">
                    <t t-if="invoice">
                        <em class="font-weight-normal text-muted">Timesheets for Invoice:</em>
                        <span t-field="invoice.display_name"/>
                    </t>
                </th>
                <th colspan="1" class="text-right text-muted">
                    <t t-if="is_uom_day">
                        Total: <span t-esc="timesheets._convert_hours_to_days(hours_spent)" t-options='{"widget": "timesheet_uom"}'/>
                    </t>
                    <t t-else="">
                        Total: <span t-esc="hours_spent" t-options='{"widget": "float_time"}'/>
                    </t>
                </th>
            </t>
        </xpath>
        <xpath expr="//thead/tr/th[@t-if='is_uom_day']" position="before">
            <th t-if="not groupby == 'sol'">Sales Order Item</th>
        </xpath>
        <xpath expr="//tbody//td[hasclass('text-right')]" position="before">
            <td t-if="not groupby == 'sol'"><span t-field="timesheet.so_line" t-att-title="timesheet.so_line.display_name"></span></td>
        </xpath>
    </template>

    <template id="portal_invoice_page_inherit" inherit_id="account.portal_invoice_page">
        <xpath expr="//t[@t-set='entries']/ul/li[.//a[@id='print_invoice_report']]" position="after">
            <li class="list-group-item flex-grow-1">
                <div class="btn-toolbar flex-sm-nowrap justify-content-center">
                    <div class="btn-group mb-1">
                        <a t-if="invoice.move_type == 'out_invoice' and invoice.state in ('draft', 'posted') and invoice.timesheet_count > 0"
                        target="_blank" t-att-href="'/my/timesheets?search_in=invoice&amp;search=%s' % invoice.name">View Timesheets</a>
                    </div>
                </div>
            </li>
        </xpath>
    </template>

    <template id="portal_my_task_inherit" inherit_id="project.portal_my_task">
        <xpath expr="//div[@name='portal_my_task_second_column']" position="inside">
            <t t-if="task.project_id.allow_billable">
                <div t-if="task.sale_order_id"><strong>Sales Order:</strong>
                    <span t-if="so_accessible"><a t-attf-href="/my/orders/{{ task.sale_order_id.id }}" t-field="task.sale_order_id"></a></span>
                    <span t-else="" t-field="task.sale_order_id"></span>
                </div>
                <div t-if="task.sale_order_id.invoice_ids"><strong>Invoices:</strong>
                    <span t-foreach="task.sale_order_id.invoice_ids" t-as="invoice_line">
                        <t t-if="invoice_line.id in invoices_accessible">
                            <a t-attf-href="/my/invoices/{{ invoice_line.id }}" t-esc="invoice_line.name"></a><span t-if="not invoice_line_last">,</span>
                        </t>
                        <t t-else=""><span t-esc="invoice_line.name"></span><span t-if="not invoice_line_last">,</span></t>
                    </span>
                </div>
                <div t-if="task.sale_line_id.untaxed_amount_invoiced > 0"><strong>Invoiced:</strong>
                    <span t-field="task.sale_line_id.untaxed_amount_invoiced"/>
                </div>
                <div t-if="task.sale_line_id.untaxed_amount_to_invoice > 0"><strong>To invoice:</strong>
                    <span t-field="task.sale_line_id.untaxed_amount_to_invoice"/>
                </div>
            </t>
        </xpath>
    </template>

    <template id="portal_timesheet_table_inherit" inherit_id="hr_timesheet.portal_timesheet_table">
        <xpath expr="//div[@name='planned_time']" position="after">
            <span t-if="task.allow_billable and task.sale_line_id">
                <div t-if="is_uom_day">Remaining Days on SO: <span t-esc="timesheets._convert_hours_to_days(task.remaining_hours_so)" t-options='{"widget": "timesheet_uom"}'/></div>
                <div t-else="">Remaining Hours on SO: <span t-esc="task.remaining_hours_so" t-options='{"widget": "float_time"}'/></div>
            </span>
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
                    <button string="Create Invoice" type="object" name="action_create_invoice" class="oe_highlight" data-hotkey="q"/>
                    <button string="Cancel" special="cancel" data-hotkey="z" type="object" class="btn btn-secondary oe_inline"/>
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
            if project.pricing_type != 'task_rate' and not result.get('line_ids', False):
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
        return result

    project_id = fields.Many2one('project.project', "Project", domain=[('sale_line_id', '=', False)], help="Project for which we are creating a sales order", required=True)
    company_id = fields.Many2one(related='project_id.company_id')
    partner_id = fields.Many2one('res.partner', string="Customer", required=True, help="Customer of the sales order")
    commercial_partner_id = fields.Many2one(related='partner_id.commercial_partner_id')

    sale_order_id = fields.Many2one(
        'sale.order', string="Sales Order",
        domain="['|', '|', ('partner_id', '=', partner_id), ('partner_id', 'child_of', commercial_partner_id), ('partner_id', 'parent_of', partner_id)]")

    line_ids = fields.One2many('project.create.sale.order.line', 'wizard_id', string='Lines')
    info_invoice = fields.Char(compute='_compute_info_invoice')

    @api.depends('sale_order_id')
    def _compute_info_invoice(self):
        for line in self:
            domain = self.env['sale.order.line']._timesheet_compute_delivered_quantity_domain()
            timesheet = self.env['account.analytic.line'].read_group(domain + [('task_id', 'in', line.project_id.tasks.ids), ('so_line', '=', False), ('timesheet_invoice_id', '=', False)], ['unit_amount'], ['task_id'])
            unit_amount = round(sum(t.get('unit_amount', 0) for t in timesheet), 2) if timesheet else 0
            if not unit_amount:
                line.info_invoice = False
                continue
            company_uom = self.env.company.timesheet_encode_uom_id
            label = _("hours")
            if company_uom == self.env.ref('uom.product_uom_day'):
                label = _("days")
            line.info_invoice = _("%(amount)s %(label)s will be added to the new Sales Order.", amount=unit_amount, label=label)

    @api.onchange('partner_id')
    def _onchange_partner_id(self):
        self.sale_order_id = False

    def action_create_sale_order(self):
        # if project linked to SO line or at least on tasks with SO line, then we consider project as billable.
        if self.project_id.sale_line_id:
            raise UserError(_("The project is already linked to a sales order item."))
        # at least one line
        if not self.line_ids:
            raise UserError(_("At least one line should be filled."))

        if self.line_ids.employee_id:
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
        if not self.line_ids.employee_id:  # Then we configure the project with pricing type is equal to project rate
            self._make_billable_at_project_rate(sale_order)
        else:  # Then we configure the project with pricing type is equal to employee rate
            self._make_billable_at_employee_rate(sale_order)

    def _make_billable_at_project_rate(self, sale_order):
        self.ensure_one()
        task_left = self.project_id.tasks.filtered(lambda task: not task.sale_line_id)
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

            # link the tasks to the SO line
            task_ids.write({
                'sale_line_id': sale_order_line.id,
                'partner_id': sale_order.partner_id.id,
                'email_from': sale_order.partner_id.email,
            })

            # assign SOL to timesheets
            search_domain = [('task_id', 'in', task_ids.ids), ('so_line', '=', False)]
            self.env['account.analytic.line'].search(search_domain).write({
                'so_line': sale_order_line.id
            })
            sale_order_line.with_context({'no_update_planned_hours': True}).write({
                'product_uom_qty': sale_order_line.qty_delivered
            })

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

        # assign SOL to timesheets
        for map_entry in map_entries:
            search_domain = [('employee_id', '=', map_entry.employee_id.id), ('so_line', '=', False), ('task_id', 'in', self.project_id.tasks.ids)]
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
    product_id = fields.Many2one('product.product', domain=[('detailed_type', '=', 'service'), ('invoice_policy', '=', 'delivery'), ('service_type', '=', 'timesheet')], string="Service",
        help="Product of the sales order item. Must be a service invoiced based on timesheets on tasks.")
    price_unit = fields.Float("Unit Price", help="Unit price of the sales order item.")
    currency_id = fields.Many2one('res.currency', string="Currency")
    employee_id = fields.Many2one('hr.employee', string="Employee", help="Employee that has timesheets on the project.")

    _sql_constraints = [
        ('unique_employee_per_wizard', 'UNIQUE(wizard_id, employee_id)', "An employee cannot be selected more than once in the mapping. Please remove duplicate(s) and try again."),
    ]

    @api.onchange('product_id')
    def _onchange_product_id(self):
        self.price_unit = self.product_id.lst_price or 0
        self.currency_id = self.product_id.currency_id

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
                <group>
                    <group>
                        <field name="project_id" readonly="1"/>
                        <field name="company_id" invisible="1"/>
                        <field name="partner_id" domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]"/>
                    </group>
                </group>
                <group>
                    <field name="line_ids" nolabel="1">
                        <tree editable="bottom">
                            <field name="employee_id" options="{'no_create_edit': True, 'no_create': True}"/>
                            <field name="product_id" options="{'no_create_edit': True, 'no_create': True}"/>
                            <field name="price_unit" widget='monetary' options="{'currency_field': 'currency_id', 'field_digits': True}"/>
                            <field name="currency_id" invisible="1"/>
                        </tree>
                    </field>
                </group>
                <group attrs="{'invisible': [('info_invoice', '=', False)]}">
                    <field name="info_invoice" nolabel="1"/>
                </group>
                <footer>
                    <button string="Create Sales Order" type="object" name="action_create_sale_order" class="oe_highlight" data-hotkey="q"/>
                    <button string="Cancel" special="cancel" data-hotkey="z" type="object" class="btn btn-secondary oe_inline"/>
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
        sale_orders = self.env['sale.order'].browse(self._context.get('active_ids') or self._context.get('active_id'))
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

```

