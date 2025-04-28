# Odoo Module: hr_holidays

Category: Human Resources/Time Off

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import populate
from . import report
from . import wizard

from odoo import api, SUPERUSER_ID

def _hr_holiday_post_init(env):
    french_companies = env['res.company'].search_count([('partner_id.country_id.code', '=', 'FR')])
    if french_companies:
        env['ir.module.module'].search([
            ('name', '=', 'l10n_fr_hr_work_entry_holidays'),
            ('state', '=', 'uninstalled')
        ]).sudo().button_install()

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Time Off',
    'version': '1.6',
    'category': 'Human Resources/Time Off',
    'sequence': 85,
    'summary': 'Allocate PTOs and follow leaves requests',
    'website': 'https://www.odoo.com/app/time-off',
    'description': """
Manage time off requests and allocations
=====================================

This application controls the time off schedule of your company. It allows employees to request time off. Then, managers can review requests for time off and approve or reject them. This way you can control the overall time off planning for the company or department.

You can configure several kinds of time off (sickness, paid days, ...) and allocate time off to an employee or department quickly using time off allocation. An employee can also make a request for more days off by making a new time off allocation. It will increase the total of available days for that time off type (if the request is accepted).

You can keep track of time off in different ways by following reports:

* Time Off Summary
* Time Off by Department
* Time Off Analysis

A synchronization with an internal agenda (Meetings of the CRM module) is also possible in order to automatically create a meeting when a time off request is accepted by setting up a type of meeting in time off Type.
""",
    'depends': ['hr', 'calendar', 'resource'],
    'data': [
        'data/report_paperformat.xml',
        'data/mail_activity_type_data.xml',
        'data/mail_message_subtype_data.xml',
        'data/hr_holidays_data.xml',
        'data/ir_cron_data.xml',

        'security/hr_holidays_security.xml',
        'security/ir.model.access.csv',

        'views/resource_views.xml',
        'views/hr_leave_views.xml',
        'views/hr_leave_type_views.xml',
        'views/hr_leave_allocation_views.xml',
        'views/hr_leave_accrual_views.xml',
        'views/hr_leave_mandatory_day_views.xml',
        'views/mail_activity_views.xml',

        'wizard/hr_holidays_cancel_leave_views.xml',
        'wizard/hr_holidays_summary_employees_views.xml',
        'wizard/hr_departure_wizard_views.xml',

        'report/hr_holidays_templates.xml',
        'report/hr_holidays_reports.xml',
        'report/hr_leave_reports.xml',
        'report/hr_leave_report_calendar.xml',
        'report/hr_leave_employee_type_report.xml',

        'views/hr_views.xml',
        'views/hr_holidays_views.xml',
    ],
    'demo': [
        'data/hr_holidays_demo.xml',
    ],
    'installable': True,
    'application': True,
    'assets': {
        'web.assets_backend': [
            'hr_holidays/static/src/**/*',
            # Don't include dark mode files in light mode
            ('remove', 'hr_holidays/static/src/**/*.dark.scss'),
        ],
        "web.assets_web_dark": [
            'hr_holidays/static/src/**/*.dark.scss',
        ],
        'web.tests_assets': [
            'hr_holidays/static/tests/helpers/**/*',
        ],
        'web.qunit_suite_tests': [
            'hr_holidays/static/tests/**/*.js',
            ('remove', 'hr_holidays/static/tests/tours/**/*'),
            ('remove', 'hr_holidays/static/tests/helpers/**/*'),
        ],
        'web.assets_tests': [
            '/hr_holidays/static/tests/tours/**/*'
        ],
    },
    'post_init_hook': '_hr_holiday_post_init',
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.mail.controllers.mail import MailController
from odoo import http


class HrHolidaysController(http.Controller):

    @http.route('/leave/validate', type='http', auth='user', methods=['GET'])
    def hr_holidays_request_validate(self, res_id, token):
        comparison, record, redirect = MailController._check_token_and_record_or_redirect('hr.leave', int(res_id), token)
        if comparison and record:
            try:
                record.action_approve()
            except Exception:
                return MailController._redirect_to_messaging()
        return redirect

    @http.route('/leave/refuse', type='http', auth='user', methods=['GET'])
    def hr_holidays_request_refuse(self, res_id, token):
        comparison, record, redirect = MailController._check_token_and_record_or_redirect('hr.leave', int(res_id), token)
        if comparison and record:
            try:
                record.action_refuse()
            except Exception:
                return MailController._redirect_to_messaging()
        return redirect

    @http.route('/allocation/validate', type='http', auth='user', methods=['GET'])
    def hr_holidays_allocation_validate(self, res_id, token):
        comparison, record, redirect = MailController._check_token_and_record_or_redirect('hr.leave.allocation', int(res_id), token)
        if comparison and record:
            try:
                record.action_approve()
            except Exception:
                return MailController._redirect_to_messaging()
        return redirect

    @http.route('/allocation/refuse', type='http', auth='user', methods=['GET'])
    def hr_holidays_allocation_refuse(self, res_id, token):
        comparison, record, redirect = MailController._check_token_and_record_or_redirect('hr.leave.allocation', int(res_id), token)
        if comparison and record:
            try:
                record.action_refuse()
            except Exception:
                return MailController._redirect_to_messaging()
        return redirect

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*
from . import main

```

## File: data\hr_holidays_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="icon_1" model="ir.attachment">
            <field name="name">Annual_Time_Off.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Annual_Time_Off.svg</field>
        </record>
        <record id="icon_2" model="ir.attachment">
            <field name="name">Annual_Time_Off_2.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Annual_Time_Off_2.svg</field>
        </record>
        <record id="icon_3" model="ir.attachment">
            <field name="name">Annual_Time_Off_3.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Annual_Time_Off_3.svg</field>
        </record>
        <record id="icon_4" model="ir.attachment">
            <field name="name">Compensatory_Time_Off.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Compensatory_Time_Off.svg</field>
        </record>
        <record id="icon_5" model="ir.attachment">
            <field name="name">Compensatory_Time_Off_2.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Compensatory_Time_Off_2.svg</field>
        </record>
        <record id="icon_6" model="ir.attachment">
            <field name="name">Compensatory_Time_Off_3.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Compensatory_Time_Off_3.svg</field>
        </record>
        <record id="icon_7" model="ir.attachment">
            <field name="name">Credit_Time.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Credit_Time.svg</field>
        </record>
        <record id="icon_8" model="ir.attachment">
            <field name="name">Credit_Time_2.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Credit_Time_2.svg</field>
        </record>
        <record id="icon_9" model="ir.attachment">
            <field name="name">Extra_Time_Off.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Extra_Time_Off.svg</field>
        </record>
        <record id="icon_10" model="ir.attachment">
            <field name="name">Extra_Time_Off_2.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Extra_Time_Off_2.svg</field>
        </record>
        <record id="icon_11" model="ir.attachment">
            <field name="name">Maternity_Time_Off.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Maternity_Time_Off.svg</field>
        </record>
        <record id="icon_12" model="ir.attachment">
            <field name="name">Maternity_Time_Off_2.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Maternity_Time_Off_2.svg</field>
        </record>
        <record id="icon_13" model="ir.attachment">
            <field name="name">Maternity_Time_Off_3.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Maternity_Time_Off_3.svg</field>
        </record>
        <record id="icon_14" model="ir.attachment">
            <field name="name">Paid_Time_Off.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Paid_Time_Off.svg</field>
        </record>
        <record id="icon_15" model="ir.attachment">
            <field name="name">Paid_Time_Off_2.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Paid_Time_Off_2.svg</field>
        </record>
        <record id="icon_16" model="ir.attachment">
            <field name="name">Paid_Time_Off_3.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Paid_Time_Off_3.svg</field>
        </record>
        <record id="icon_17" model="ir.attachment">
            <field name="name">Parental_Time_Off.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Parental_Time_Off.svg</field>
        </record>
        <record id="icon_18" model="ir.attachment">
            <field name="name">Parental_Time_Off_2.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Parental_Time_Off_2.svg</field>
        </record>
        <record id="icon_19" model="ir.attachment">
            <field name="name">Recovery_Bank_Holiday.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Recovery_Bank_Holiday.svg</field>
        </record>
        <record id="icon_20" model="ir.attachment">
            <field name="name">Recovery_Bank_Holiday_2.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Recovery_Bank_Holiday_2.svg</field>
        </record>
        <record id="icon_21" model="ir.attachment">
            <field name="name">Sick_Time_Off.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Sick_Time_Off.svg</field>
        </record>
        <record id="icon_22" model="ir.attachment">
            <field name="name">Sick_Time_Off_2.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Sick_Time_Off_2.svg</field>
        </record>
        <record id="icon_23" model="ir.attachment">
            <field name="name">Small_Unemployement.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Small_Unemployement.svg</field>
        </record>
        <record id="icon_24" model="ir.attachment">
            <field name="name">Small_Unemployement_2.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Small_Unemployement_2.svg</field>
        </record>
        <record id="icon_25" model="ir.attachment">
            <field name="name">Small_Unemployement_3.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Small_Unemployement_3.svg</field>
        </record>
        <record id="icon_26" model="ir.attachment">
            <field name="name">Training_Time_Off.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Training_Time_Off.svg</field>
        </record>
        <record id="icon_27" model="ir.attachment">
            <field name="name">Training_Time_Off_2.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Training_Time_Off_2.svg</field>
        </record>
        <record id="icon_28" model="ir.attachment">
            <field name="name">Unpaid_Time_Off.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Unpaid_Time_Off.svg</field>
        </record>
        <record id="icon_29" model="ir.attachment">
            <field name="name">Unpaid_Time_Off_2.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Unpaid_Time_Off_2.svg</field>
        </record>
        <record id="icon_30" model="ir.attachment">
            <field name="name">Work_Accident_Time_Off.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Work_Accident_Time_Off.svg</field>
        </record>
        <record id="icon_31" model="ir.attachment">
            <field name="name">Work_Accident_Time_Off_2.svg</field>
            <field name="res_model">hr.leave.type</field>
            <field name="res_field">icon_id</field>
            <field name="public" eval="True"/>
            <field name="type">url</field>
            <field name="url">/hr_holidays/static/src/img/icons/Work_Accident_Time_Off_2.svg</field>
        </record>
    </data>
    <data noupdate="1">
        <!-- Casual leave -->
        <record id="holiday_status_cl" model="hr.leave.type">
            <field name="name">Paid Time Off</field>
            <field name="requires_allocation">yes</field>
            <field name="employee_requests">no</field>
            <field name="leave_validation_type">both</field>
            <field name="allocation_validation_type">officer</field>
            <field name="leave_notif_subtype_id" ref="mt_leave"/>
            <field name="allocation_notif_subtype_id" ref="mt_leave_allocation"/>
            <field name="responsible_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="icon_id" ref="hr_holidays.icon_14"/>
            <field name="color">2</field>
            <field name="company_id" eval="False"/> <!-- Explicitely set to False for it to be available to all companies -->
            <field name="sequence">1</field>
        </record>

        <!-- Sick leave -->
        <record id="holiday_status_sl" model="hr.leave.type">
            <field name="name">Sick Time Off</field>
            <field name="requires_allocation">no</field>
            <field name="leave_notif_subtype_id" ref="mt_leave_sick"/>
            <field name="responsible_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="support_document">True</field>
            <field name="icon_id" ref="hr_holidays.icon_22"/>
            <field name="color">3</field>
            <field name="company_id" eval="False"/> <!-- Explicitely set to False for it to be available to all companies -->
            <field name="sequence">2</field>
        </record>

        <!-- Compensatory Days -->
        <record id="holiday_status_comp" model="hr.leave.type">
            <field name="name">Compensatory Days</field>
            <field name="requires_allocation">yes</field>
            <field name="employee_requests">yes</field>
            <field name="leave_validation_type">manager</field>
            <field name="allocation_validation_type">officer</field>
            <field name="request_unit">hour</field>
            <field name="leave_notif_subtype_id" ref="mt_leave"/>
            <field name="responsible_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="icon_id" ref="hr_holidays.icon_4"/>
            <field name="color">4</field>
            <field name="company_id" eval="False"/> <!-- Explicitely set to False for it to be available to all companies -->
            <field name="sequence">4</field>
        </record>

        <!--Unpaid Time Off -->
        <record id="holiday_status_unpaid" model="hr.leave.type">
            <field name="name">Unpaid</field>
            <field name="requires_allocation">no</field>
            <field name="leave_validation_type">both</field>
            <field name="allocation_validation_type">officer</field>
            <field name="request_unit">hour</field>
            <field name="unpaid" eval="True"/>
            <field name="leave_notif_subtype_id" ref="mt_leave_unpaid"/>
            <field name="responsible_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="icon_id" ref="hr_holidays.icon_28"/>
            <field name="color">5</field>
            <field name="company_id" eval="False"/> <!-- Explicitely set to False for it to be available to all companies -->
            <field name="sequence">3</field>
        </record>
    </data>
</odoo>

```

## File: data\hr_holidays_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">

    <record id="base.user_demo" model="res.users">
        <field name="groups_id" eval="[(3, ref('hr_holidays.group_hr_holidays_manager'))]"/>
    </record>

    <!--Time Off Type-->
    <record id="hr_holiday_status_dv" model="hr.leave.type">
        <field name="name">Parental Leaves</field>
        <field name="requires_allocation">yes</field>
        <field name="employee_requests">no</field>
        <field name="leave_validation_type">both</field>
        <field name="allocation_validation_type">officer</field>
        <field name="responsible_ids" eval="[(4, ref('base.user_admin'))]"/>
        <field name="icon_id" ref="hr_holidays.icon_11"/>
    </record>

    <record id="holiday_status_training" model="hr.leave.type">
        <field name="name">Training Time Off</field>
        <field name="requires_allocation">yes</field>
        <field name="employee_requests">no</field>
        <field name="leave_validation_type">both</field>
        <field name="allocation_validation_type">officer</field>
        <field name="responsible_ids" eval="[(4, ref('base.user_admin'))]"/>
        <field name="icon_id" ref="hr_holidays.icon_26"/>
        <field name="allows_negative" eval="True"/>
        <field name="max_allowed_negative" eval="20"/>
    </record>

    <!-- Accrual Plan -->
    <record id="hr_accrual_plan_1" model="hr.leave.accrual.plan">
        <field name="name">Seniority Plan</field>
    </record>

    <record id="hr_accrual_level_1" model="hr.leave.accrual.level">
        <field name="accrual_plan_id" ref="hr_accrual_plan_1" />
        <field name="start_count">1</field>
        <field name="start_type">day</field>
        <field name="added_value">1</field>
        <field name="frequency">yearly</field>
    </record>
    <record id="hr_accrual_level_2" model="hr.leave.accrual.level">
        <field name="accrual_plan_id" ref="hr_accrual_plan_1" />
        <field name="start_count">4</field>
        <field name="start_type">year</field>
        <field name="added_value">2</field>
        <field name="frequency">yearly</field>
    </record>
    <record id="hr_accrual_level_3" model="hr.leave.accrual.level">
        <field name="accrual_plan_id" ref="hr_accrual_plan_1" />
        <field name="start_count">8</field>
        <field name="start_type">year</field>
        <field name="added_value">3</field>
        <field name="frequency">yearly</field>
    </record>

    <!-- ++++++++++++++++++++++  Mitchell Admin  ++++++++++++++++++++++ -->

    <record id="hr_holidays_allocation_cl" model="hr.leave.allocation">
        <field name="name">Paid Time Off for Mitchell Admin</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field name="number_of_days">20</field>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_admin'))]"/>
        <field name="state">confirm</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <record id="hr_holidays_int_tour" model="hr.leave.allocation">
        <field name="name">International Tour</field>
        <field name="holiday_status_id" ref="holiday_status_comp"/>
        <field name="number_of_days">7</field>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_admin'))]"/>
        <field name="state">confirm</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <record id="hr_holidays_vc" model="hr.leave.allocation">
        <field name="name">Functional Training</field>
        <field name="holiday_status_id" ref="holiday_status_training"/>
        <field name="number_of_days">7</field>
        <field name="state">confirm</field>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_admin'))]"/>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <record id='hr_holidays_cl_allocation' model="hr.leave.allocation">
        <field name="name">Compensation</field>
        <field name="holiday_status_id" ref="holiday_status_comp"/>
        <field name="number_of_days">12</field>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_admin'))]"/>
        <field name="state">confirm</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>
    <function model="hr.leave.allocation" name="action_validate">
        <value eval="[ref('hr_holidays_allocation_cl'), ref('hr_holidays_int_tour'), ref('hr_holidays_cl_allocation')]"/>
    </function>

    <!-- leave request -->
    <record id="hr_holidays_cl" model="hr.leave">
        <field name="name">Trip with Family</field>
        <field name="holiday_status_id" ref="holiday_status_comp"/>
        <field name="request_date_from" eval="(datetime.today().date() + relativedelta(day=1, weekday=0))"/>
        <field name="request_date_to" eval="(datetime.today().date() + relativedelta(day=1, weekday=0) + relativedelta(weekday=2))"/>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_admin'))]"/>
    </record>

    <record id="hr_holidays_sl" model="hr.leave">
        <field name="name">Doctor Appointment</field>
        <field name="holiday_status_id" ref="holiday_status_sl"/>
        <field name="request_date_from" eval="(datetime.today().date() + relativedelta(day=20, weekday=0))"/>
        <field name="request_date_to" eval="(datetime.today().date() + relativedelta(day=20, weekday=0) + relativedelta(weekday=2))"/>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_admin'))]"/>
        <field name="state">confirm</field>
    </record>
    <function model="hr.leave" name="action_validate">
        <value eval="ref('hr_holidays.hr_holidays_sl')"/>
    </function>

    <record id="hr.employee_al" model="hr.employee">
        <field name="leave_manager_id" ref="base.user_admin"/>
    </record>
    <record id="hr.employee_mit" model="hr.employee">
        <field name="leave_manager_id" ref="base.user_admin"/>
    </record>
    <record id="hr.employee_qdp" model="hr.employee">
        <field name="leave_manager_id" ref="base.user_admin"/>
    </record>
    <record id="hr.employee_niv" model="hr.employee">
        <field name="leave_manager_id" ref="base.user_admin"/>
    </record>
    <record id="hr.employee_jve" model="hr.employee">
        <field name="leave_manager_id" ref="base.user_admin"/>
    </record>

    <!-- ++++++++++++++++++++++  Ronnie Hart  ++++++++++++++++++++++ -->


    <record id="hr_holidays_allocation_cl_al" model="hr.leave.allocation">
        <field name="name">Paid Time Off for Ronnie Hart</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field name="number_of_days">20</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_al'))]"/>
        <field name="state">confirm</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <record id="hr_holidays_allocation_pl_al" model="hr.leave.allocation">
        <field name="name">Parental Leaves</field>
        <field name="holiday_status_id" ref="hr_holiday_status_dv"/>
        <field name="number_of_days">10</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_al'))]"/>
        <field name="state">confirm</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <record id="hr_holidays_vc_al" model="hr.leave.allocation">
        <field name="name">Soft Skills Training</field>
        <field name="holiday_status_id" ref="holiday_status_training"/>
        <field name="number_of_days">12</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_al'))]"/>
        <field name="state">confirm</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>
    <function model="hr.leave.allocation" name="action_validate">
        <value eval="[ref('hr_holidays_allocation_cl_al'), ref('hr_holidays_allocation_pl_al'), ref('hr_holidays_vc_al')]"/>
    </function>

    <!-- leave request -->
    <record id="hr_holidays_cl_al" model="hr.leave">
        <field name="name">Trip with Friends</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field name="request_date_from" eval="time.strftime('%Y-%m-14')"/>
        <field name="request_date_to" eval="time.strftime('%Y-%m-20')"/>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_al'))]"/>
    </record>
    <function model="hr.leave" name="action_validate">
        <value eval="ref('hr_holidays.hr_holidays_cl_al')"/>
    </function>

    <record id="hr_holidays_sl_al" model="hr.leave">
        <field name="name">Dentist appointment</field>
        <field name="holiday_status_id" ref="holiday_status_sl"/>
        <field name="request_date_from" eval="(datetime.today().date() + relativedelta(months=1, day=17, weekday=0))"/>
        <field name="request_date_to" eval="(datetime.today().date() + relativedelta(months=1, day=17, weekday=0) + relativedelta(weekday=2))"/>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_al'))]"/>
        <field name="state">confirm</field>
    </record>
    <function model="hr.leave" name="action_validate">
        <value eval="ref('hr_holidays.hr_holidays_sl_al')"/>
    </function>

    <!-- ++++++++++++++++++++++  Anita Oliver  ++++++++++++++++++++++ -->

    <record id="hr_holidays_allocation_cl_mit" model="hr.leave.allocation">
        <field name="name">Paid Time Off for Anita Oliver</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field name="number_of_days">20</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_mit'))]"/>
        <field name="state">confirm</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>
    <function model="hr.leave.allocation" name="action_validate">
        <value eval="[ref('hr_holidays_allocation_cl_mit')]"/>
    </function>

    <record id="hr_holidays_vc_mit" model="hr.leave.allocation">
        <field name="name">Compliance Training</field>
        <field name="holiday_status_id" ref="holiday_status_training"/>
        <field name="number_of_days">7</field>
        <field name="state">confirm</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_mit'))]"/>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <!-- leave request -->
    <record id="hr_holidays_cl_mit" model="hr.leave">
        <field name="name">Trip to Paris</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field name="request_date_from" eval="time.strftime('%Y-%m-22')"/>
        <field name="request_date_to" eval="time.strftime('%Y-%m-28')"/>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_mit'))]"/>
    </record>
    <function model="hr.leave" name="action_validate">
        <value eval="ref('hr_holidays.hr_holidays_cl_mit')"/>
    </function>

    <record id="hr_holidays_cl_mit_2" model="hr.leave">
        <field name="name">Trip</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field name="request_date_from" eval="(datetime.today().date() + relativedelta(day=5, weekday=0))"/>
        <field name="request_date_to" eval="(datetime.today().date() + relativedelta(day=5, weekday=0) + relativedelta(weekday=2))"/>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_mit'))]"/>
        <field name="state">confirm</field>
    </record>
    <function model="hr.leave" name="action_approve">
        <value eval="ref('hr_holidays.hr_holidays_cl_mit_2')"/>
    </function>

    <!-- ++++++++++++++++++++++  Marc Demo  ++++++++++++++++++++++ -->

    <record id="hr_holidays_allocation_cl_qdp" model="hr.leave.allocation">
        <field name="name">Paid Time Off for Marc Demo</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field name="number_of_days">20</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_qdp'))]"/>
        <field name="state">confirm</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <record id="hr_holidays_vc_qdp" model="hr.leave.allocation">
        <field name="name">Time Management Training</field>
        <field name="holiday_status_id" ref="holiday_status_training"/>
        <field name="number_of_days">7</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_qdp'))]"/>
        <field name="state">confirm</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>
    <function model="hr.leave.allocation" name="action_validate">
        <value eval="[ref('hr_holidays.hr_holidays_allocation_cl_qdp'), ref('hr_holidays.hr_holidays_vc_qdp')]"/>
    </function>

    <!-- leave request -->
    <record id="hr_holidays_cl_qdp" model="hr.leave">
        <field name="name">Sick day</field>
        <field name="holiday_status_id" ref="holiday_status_sl"/>
        <field name="request_date_from" eval="(datetime.today().date()+relativedelta(months=1, day=3, weekday=0))"/>
        <field name="request_date_to" eval="(datetime.today().date()+relativedelta(months=1, day=3, weekday=0) + relativedelta(weekday=2))"/>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_qdp'))]"/>
        <field name="state">confirm</field>
    </record>
    <function model="hr.leave" name="action_validate">
        <value eval="ref('hr_holidays.hr_holidays_cl_qdp')"/>
    </function>

    <record id="hr_holidays_sl_qdp" model="hr.leave">
        <field name="name">Sick day</field>
        <field name="holiday_status_id" ref="holiday_status_sl"/>
        <field name="request_date_from" eval="(datetime.today().date() + relativedelta(day=1, weekday=0))"/>
        <field name="request_date_to" eval="(datetime.today().date() + relativedelta(day=1, weekday=0) + relativedelta(days=2))"/>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_qdp'))]"/>
        <field name="state">confirm</field>
    </record>
    <function model="hr.leave" name="action_validate">
        <value eval="ref('hr_holidays.hr_holidays_sl_qdp')"/>
    </function>

    <!-- ++++++++++++++++++++++  Audrey Peterson  ++++++++++++++++++++++ -->

    <record id="hr_holidays_allocation_cl_fpi" model="hr.leave.allocation">
        <field name="name">Paid Time Off for Audrey Peterson</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field name="number_of_days">20</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_fpi'))]"/>
        <field name="state">confirm</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>
    <function model="hr.leave.allocation" name="action_validate">
        <value eval="[ref('hr_holidays.hr_holidays_allocation_cl_fpi')]"/>
    </function>

    <record id="hr_holidays_vc_fpi" model="hr.leave.allocation">
        <field name="name">Consulting Training</field>
        <field name="holiday_status_id" ref="holiday_status_training"/>
        <field name="number_of_days">7</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_fpi'))]"/>
        <field name="state">confirm</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <!-- ++++++++++++++++++++++   Olivia  ++++++++++++++++++++++ -->

    <record id="hr_holidays_allocation_cl_vad" model="hr.leave.allocation">
        <field name="name">Paid Time Off for Olivia</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field name="number_of_days">20</field>
        <field name="employee_id" ref="hr.employee_niv"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_niv'))]"/>
        <field name="state">confirm</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <record id="hr_holidays_vc_vad" model="hr.leave.allocation">
        <field name="name">Software Development Training</field>
        <field name="holiday_status_id" ref="holiday_status_training"/>
        <field name="number_of_days">5</field>
        <field name="employee_id" ref="hr.employee_niv"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_niv'))]"/>
        <field name="state">confirm</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>
    <function model="hr.leave.allocation" name="action_validate">
        <value eval="[ref('hr_holidays.hr_holidays_allocation_cl_vad'), ref('hr_holidays.hr_holidays_vc_vad')]"/>
    </function>

    <record id="hr_holidays_cl_vad" model="hr.leave">
        <field name="name">Trip to London</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field name="request_date_from" eval="time.strftime('%Y-%m-09')"/>
        <field name="request_date_to" eval="time.strftime('%Y-%m-16')"/>
        <field name="employee_id" ref="hr.employee_niv"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_niv'))]"/>
        <field name="state">confirm</field>
    </record>
    <function model="hr.leave" name="action_validate">
        <value eval="ref('hr_holidays.hr_holidays_cl_vad')"/>
    </function>

    <record id="hr_holidays_sl_vad" model="hr.leave">
        <field name="name">Doctor Appointment</field>
        <field name="holiday_status_id" ref="holiday_status_sl"/>
        <field name="request_date_from" eval="(datetime.today().date() + relativedelta(day=25, weekday=0))"/>
        <field name="request_date_to" eval="(datetime.today().date() + relativedelta(day=25, weekday=0) + relativedelta(weekday=2))"/>
        <field name="employee_id" ref="hr.employee_niv"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_niv'))]"/>
        <field name="state">confirm</field>
    </record>
    <function model="hr.leave" name="action_validate">
        <value eval="ref('hr_holidays.hr_holidays_sl_vad')"/>
    </function>

    <!-- ++++++++++++++++++++++   Kim  ++++++++++++++++++++++ -->

    <record id="hr_holidays_allocation_cl_kim" model="hr.leave.allocation">
        <field name="name">Paid Time Off for Kim</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field name="number_of_days">20</field>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_jve'))]"/>
        <field name="state">confirm</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <record id="hr_holidays_vc_kim" model="hr.leave.allocation">
        <field name="name">Onboarding Training</field>
        <field name="holiday_status_id" ref="holiday_status_training"/>
        <field name="number_of_days">5</field>
        <field name="state">confirm</field>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_jve'))]"/>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <!-- leave request -->
    <record id="hr_holidays_sl_kim" model="hr.leave">
        <field name="name">Dentist appointment</field>
        <field name="holiday_status_id" ref="holiday_status_sl"/>
        <field name="request_date_from" eval="(datetime.today().date() + relativedelta(months=1, day=1, weekday=0))"/>
        <field name="request_date_to" eval="(datetime.today().date() + relativedelta(months=1, day=1, weekday=0))"/>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_jve'))]"/>
        <field name="state">confirm</field>
    </record>
    <function model="hr.leave" name="action_validate">
        <value eval="ref('hr_holidays.hr_holidays_sl_kim')"/>
    </function>

    <record id="hr_holidays_sl_kim_2" model="hr.leave">
        <field name="name">Second dentist appointment</field>
        <field name="holiday_status_id" ref="holiday_status_sl"/>
        <field name="request_date_from" eval="(datetime.today().date()+relativedelta(months=4, day=1, weekday=2))"/>
        <field name="request_date_to" eval="(datetime.today().date()+relativedelta(months=4, day=1, weekday=2))"/>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_jve'))]"/>
        <field name="state">confirm</field>
    </record>
    <function model="hr.leave" name="action_validate">
        <value eval="ref('hr_holidays.hr_holidays_sl_kim_2')"/>
    </function>

    <!-- Public time off -->
    <record id="resource_public_time_off_1" model="resource.calendar.leaves">
        <field name="name">Public Time Off</field>
        <field name="company_id" ref="base.main_company"/>
        <field name="calendar_id" ref="resource.resource_calendar_std"/>
        <field name="date_from" eval="time.strftime('%Y-02-13 05:00:00')"/>
        <field name="date_to" eval="time.strftime('%Y-02-13 17:00:00')"/>
    </record>

    <!-- Mandatory day -->
    <record id="hr_leave_mandatory_day_1" model="hr.leave.mandatory.day">
        <field name="name">Company Celebration</field>
        <field name="company_id" ref="base.main_company"/>
        <field name="start_date" eval="(datetime.today() + relativedelta(days=+7)).strftime('%Y-%m-%d 07:00:00')"></field>
        <field name="end_date" eval="(datetime.today() + relativedelta(days=+7)).strftime('%Y-%m-%d 16:00:00')"></field>
        <field name="color">9</field>
    </record>
</data>
</odoo>

```

## File: data\ir_cron_data.xml

```xml
<?xml version='1.0' encoding='UTF-8' ?>
<odoo>

    <record id="hr_leave_allocation_cron_accrual" model="ir.cron">
        <field name="name">Accrual Time Off: Updates the number of time off</field>
        <field name="model_id" ref="model_hr_leave_allocation"/>
        <field name="state">code</field>
        <field name="code">model._update_accrual()</field>
        <field name="interval_number">1</field>
        <field name="interval_type">days</field>
        <field name="numbercall">-1</field>
        <field name="doall" eval="True"/>
    </record>

    <record id="hr_leave_cron_cancel_invalid" model="ir.cron">
        <field name="name">Time Off: Cancel invalid leaves</field>
        <field name="model_id" ref="model_hr_leave"/>
        <field name="state">code</field>
        <field name="code">model._cancel_invalid_leaves()</field>
        <field name="interval_number">1</field>
        <field name="interval_type">days</field>
        <field name="numbercall">-1</field>
        <field name="doall" eval="True"/>
    </record>
</odoo>

```

## File: data\mail_activity_type_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Leave specific activities -->
        <record id="mail_act_leave_approval" model="mail.activity.type">
            <field name="name">Time Off Approval</field>
            <field name="icon">fa-sun-o</field>
            <field name="res_model">hr.leave</field>
            <field name="delay_count">15</field>
        </record>
        <record id="mail_act_leave_second_approval" model="mail.activity.type">
            <field name="name">Time Off Second Approve</field>
            <field name="icon">fa-sun-o</field>
            <field name="res_model">hr.leave</field>
        </record>

        <!-- Leave specific activities -->
        <record id="mail_act_leave_allocation_approval" model="mail.activity.type">
            <field name="name">Allocation Approval</field>
            <field name="icon">fa-sun-o</field>
            <field name="res_model">hr.leave.allocation</field>
        </record>
    </data>
</odoo>

```

## File: data\mail_message_subtype_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Holidays-related subtypes for messaging / Chatter -->
        <record id="mt_leave" model="mail.message.subtype">
            <field name="name">Time Off</field>
            <field name="res_model">hr.leave</field>
            <field name="description">Time Off Request</field>
        </record>
        <record id="mt_leave_home_working" model="mail.message.subtype">
            <field name="name">Home Working</field>
            <field name="res_model">hr.leave</field>
            <field name="description">Home Working</field>
        </record>
        <record id="mt_leave_sick" model="mail.message.subtype">
            <field name="name">Sick Time Off</field>
            <field name="res_model">hr.leave</field>
            <field name="description">Sick Time Off</field>
        </record>
        <record id="mt_leave_unpaid" model="mail.message.subtype">
            <field name="name">Unpaid Time Off</field>
            <field name="res_model">hr.leave</field>
            <field name="description">Unpaid Time Off</field>
        </record>

        <!-- Allocation-related subtypes for messaging / Chatter -->
        <record id="mt_leave_allocation" model="mail.message.subtype">
            <field name="name">Allocation</field>
            <field name="res_model">hr.leave.allocation</field>
            <field name="description">Allocation Request</field>
        </record>
    </data>
</odoo>

```

## File: data\report_paperformat.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="paperformat_hrsummary" model="report.paperformat">
            <field name="name">Time Off Summary</field>
            <field name="default" eval="True"/>
            <field name="format">custom</field>
            <field name="page_height">297</field>
            <field name="page_width">210</field>
            <field name="orientation">Landscape</field>
            <field name="margin_top">30</field>
            <field name="margin_bottom">23</field>
            <field name="margin_left">5</field>
            <field name="margin_right">5</field>
            <field name="header_line" eval="False"/>
            <field name="header_spacing">20</field>
            <field name="dpi">90</field>
        </record>
</odoo>

```

## File: models\hr_department.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models
from odoo.osv import expression
import ast


class Department(models.Model):

    _inherit = 'hr.department'

    absence_of_today = fields.Integer(
        compute='_compute_leave_count', string='Absence by Today')
    leave_to_approve_count = fields.Integer(
        compute='_compute_leave_count', string='Time Off to Approve')
    allocation_to_approve_count = fields.Integer(
        compute='_compute_leave_count', string='Allocation to Approve')

    def _compute_leave_count(self):
        Requests = self.env['hr.leave']
        Allocations = self.env['hr.leave.allocation']
        today_date = datetime.datetime.utcnow().date()
        today_start = fields.Datetime.to_string(today_date)  # get the midnight of the current utc day
        today_end = fields.Datetime.to_string(today_date + relativedelta(hours=23, minutes=59, seconds=59))

        leave_data = Requests._read_group(
            [('department_id', 'in', self.ids),
             ('state', '=', 'confirm')],
            ['department_id'], ['__count'])
        allocation_data = Allocations._read_group(
            [('department_id', 'in', self.ids),
             ('state', '=', 'confirm')],
            ['department_id'], ['__count'])
        absence_data = Requests._read_group(
            [('department_id', 'in', self.ids), ('state', 'not in', ['cancel', 'refuse']),
             ('date_from', '<=', today_end), ('date_to', '>=', today_start)],
            ['department_id'], ['__count'])

        res_leave = {department.id: count for department, count in leave_data}
        res_allocation = {department.id: count for department, count in allocation_data}
        res_absence = {department.id: count for department, count in absence_data}

        for department in self:
            department.leave_to_approve_count = res_leave.get(department.id, 0)
            department.allocation_to_approve_count = res_allocation.get(department.id, 0)
            department.absence_of_today = res_absence.get(department.id, 0)

    def _get_action_context(self):
        return {
            'search_default_approve': 1,
            'search_default_active_employee': 2,
            'search_default_department_id': self.id,
            'default_department_id': self.id,
            'searchpanel_default_department_id': self.id,
        }

    def action_open_leave_department(self):
        action = self.env["ir.actions.actions"]._for_xml_id("hr_holidays.hr_leave_action_action_approve_department")
        action['context'] = {
            **self._get_action_context(),
            'search_default_active_time_off': 3,
            'hide_employee_name': 1,
            'holiday_status_display_name': False
        }
        return action

    def action_open_allocation_department(self):
        action = self.env["ir.actions.actions"]._for_xml_id("hr_holidays.hr_leave_allocation_action_approve_department")
        action['context'] = self._get_action_context()
        action['context']['search_default_second_approval'] = 3
        action['domain'] = expression.AND([ast.literal_eval(action['domain']), [('state', '=', 'confirm')]])
        return action

```

## File: models\hr_employee.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, time
from collections import defaultdict
from dateutil.relativedelta import relativedelta
import pytz

from odoo import _, api, fields, models


class HrEmployee(models.Model):
    _inherit = 'hr.employee'

    current_leave_id = fields.Many2one('hr.leave.type', compute='_compute_current_leave', string="Current Time Off Type",
                                       groups="hr.group_hr_user")

    def _compute_current_leave(self):
        self.current_leave_id = False

        holidays = self.env['hr.leave'].sudo().search([
            ('employee_id', 'in', self.ids),
            ('date_from', '<=', fields.Datetime.now()),
            ('date_to', '>=', fields.Datetime.now()),
            ('state', '=', 'validate'),
        ])
        for holiday in holidays:
            employee = self.filtered(lambda e: e.id == holiday.employee_id.id)
            employee.current_leave_id = holiday.holiday_status_id.id

    def _get_user_m2o_to_empty_on_archived_employees(self):
        return super()._get_user_m2o_to_empty_on_archived_employees() + ['leave_manager_id']

    def action_time_off_dashboard(self):
        return {
            'name': _('Time Off Dashboard'),
            'type': 'ir.actions.act_window',
            'res_model': 'hr.leave',
            'views': [[self.env.ref('hr_holidays.hr_leave_employee_view_dashboard').id, 'calendar']],
            'domain': [('employee_id', 'in', self.ids)],
            'context': {
                'employee_id': self.ids,
            },
        }

    def _is_leave_user(self):
        return self == self.env.user.employee_id and self.user_has_groups('hr_holidays.group_hr_holidays_user')

    def get_mandatory_days(self, start_date, end_date):
        all_days = {}

        self = self or self.env.user.employee_id

        mandatory_days = self._get_mandatory_days(start_date, end_date)
        for mandatory_day in mandatory_days:
            num_days = (mandatory_day.end_date - mandatory_day.start_date).days
            for d in range(num_days + 1):
                all_days[str(mandatory_day.start_date + relativedelta(days=d))] = mandatory_day.color

        return all_days

    @api.model
    def get_special_days_data(self, date_start, date_end):
        return {
            'mandatoryDays': self.get_mandatory_days_data(date_start, date_end),
            'bankHolidays': self.get_public_holidays_data(date_start, date_end),
        }

    @api.model
    def get_public_holidays_data(self, date_start, date_end):
        self = self._get_contextual_employee()
        employee_tz = pytz.timezone(self._get_tz() if self else self.env.user.tz or 'utc')
        public_holidays = self._get_public_holidays(date_start, date_end).sorted('date_from')
        return list(map(lambda bh: {
            'id': -bh.id,
            'colorIndex': 0,
            'end': datetime.combine(bh.date_to.astimezone(employee_tz), datetime.max.time()).isoformat(),
            'endType': "datetime",
            'isAllDay': True,
            'start': datetime.combine(bh.date_from.astimezone(employee_tz), datetime.min.time()).isoformat(),
            'startType': "datetime",
            'title': bh.name,
        }, public_holidays))

    def _get_public_holidays(self, date_start, date_end):
        domain = [
            ('resource_id', '=', False),
            ('company_id', 'in', self.env.companies.ids),
            ('date_from', '<=', date_end),
            ('date_to', '>=', date_start),
            '|',
            ('calendar_id', '=', False),
            ('calendar_id', '=', self.resource_calendar_id.id),
        ]

        return self.env['resource.calendar.leaves'].search(domain)

    @api.model
    def get_mandatory_days_data(self, date_start, date_end):
        self = self._get_contextual_employee()
        mandatory_days = self._get_mandatory_days(date_start, date_end).sorted('start_date')
        return list(map(lambda sd: {
            'id': -sd.id,
            'colorIndex': sd.color,
            'end': datetime.combine(sd.end_date, datetime.max.time()).isoformat(),
            'endType': "datetime",
            'isAllDay': True,
            'start': datetime.combine(sd.start_date, datetime.min.time()).isoformat(),
            'startType': "datetime",
            'title': sd.name,
        }, mandatory_days))

    def _get_mandatory_days(self, start_date, end_date):
        domain = [
            ('start_date', '<=', end_date),
            ('end_date', '>=', start_date),
            ('company_id', 'in', self.env.companies.ids),
            '|',
            ('resource_calendar_id', '=', False),
            ('resource_calendar_id', '=', self.resource_calendar_id.id),
        ]

        if self.department_id:
            domain += [
                '|',
                ('department_ids', '=', False),
                ('department_ids', 'parent_of', self.department_id.id),
            ]
        else:
            domain += [('department_ids', '=', False)]

        return self.env['hr.leave.mandatory.day'].search(domain)

    @api.model
    def _get_contextual_employee(self):
        ctx = self.env.context
        if self.env.context.get('employee_id') is not None:
            return self.browse(ctx.get('employee_id'))
        if self.env.context.get('default_employee_id') is not None:
            return self.browse(ctx.get('default_employee_id'))
        return self.env.user.employee_id

    def _get_consumed_leaves(self, leave_types, target_date=False, ignore_future=False):
        employees = self or self._get_contextual_employee()
        leaves_domain = [
            ('holiday_status_id', 'in', leave_types.ids),
            ('employee_id', 'in', employees.ids),
            ('state', 'in', ['confirm', 'validate1', 'validate']),
        ]
        if self.env.context.get('ignored_leave_ids'):
            leaves_domain.append(('id', 'not in', self.env.context.get('ignored_leave_ids')))

        if not target_date:
            target_date = fields.Date.today()
        if ignore_future:
            leaves_domain.append(('date_from', '<=', target_date))
        leaves = self.env['hr.leave'].search(leaves_domain)
        leaves_per_employee_type = defaultdict(lambda: defaultdict(lambda: self.env['hr.leave']))
        for leave in leaves:
            leaves_per_employee_type[leave.employee_id][leave.holiday_status_id] |= leave

        allocations = self.env['hr.leave.allocation'].with_context(active_test=False).search([
            ('employee_id', 'in', employees.ids),
            ('holiday_status_id', 'in', leave_types.ids),
            ('state', '=', 'validate'),
        ]).filtered(lambda al: al.active or not al.employee_id.active)
        allocations_per_employee_type = defaultdict(lambda: defaultdict(lambda: self.env['hr.leave.allocation']))
        for allocation in allocations:
            allocations_per_employee_type[allocation.employee_id][allocation.holiday_status_id] |= allocation

        # _get_consumed_leaves returns a tuple of two dictionnaries.
        # 1) The first is a dictionary to map the number of days/hours of leaves taken per allocation
        # The structure is the following:
        # - KEYS:
        # allocation_leaves_consumed
        #  |--employee_id
        #      |--holiday_status_id
        #          |--allocation
        #              |--virtual_leaves_taken
        #              |--leaves_taken
        #              |--virtual_remaining_leaves
        #              |--remaining_leaves
        #              |--max_leaves
        #              |--accrual_bonus
        # - VALUES:
        # Integer representing the number of (virtual) remaining leaves, (virtual) leaves taken or max leaves
        # for each allocation.
        # leaves_taken and remaining_leaves only take into account validated leaves, while the "virtual" equivalent are
        # also based on leaves in "confirm" or "validate1" state.
        # Accrual bonus gives the amount of additional leaves that will have been granted at the given
        # target_date in comparison to today.
        # The unit is in hour or days depending on the leave type request unit
        # 2) The second is a dictionary mapping the remaining days per employee and per leave type that are either
        # not taken into account by the allocations, mainly because accruals don't take future leaves into account.
        # This is used to warn the user if the leaves they takes bring them above their available limit.
        # - KEYS:
        # allocation_leaves_consumed
        #  |--employee_id
        #      |--holiday_status_id
        #          |--to_recheck_leaves
        #          |--excess_days
        #          |--exceeding_duration
        # - VALUES:
        # "to_recheck_leaves" stores every leave that is not yet taken into account by the "allocation_leaves_consumed" dictionary.
        # "excess_days" represents the excess amount that somehow isn't taken into account by the first dictionary.
        # "exceeding_duration" sum up the to_recheck_leaves duration and compares it to the maximum allocated for that time period.
        allocations_leaves_consumed = defaultdict(lambda: defaultdict(lambda: defaultdict(lambda: defaultdict(lambda: 0))))

        to_recheck_leaves_per_leave_type = defaultdict(lambda:
            defaultdict(lambda: {
                'excess_days': defaultdict(lambda: {
                    'amount': 0,
                    'is_virtual': True,
                }),
                'exceeding_duration': 0,
                'to_recheck_leaves': self.env['hr.leave']
            })
        )
        for allocation in allocations:
            allocation_data = allocations_leaves_consumed[allocation.employee_id][allocation.holiday_status_id][allocation]
            future_leaves = 0
            if allocation.allocation_type == 'accrual':
                future_leaves = allocation._get_future_leaves_on(target_date)
            max_leaves = allocation.number_of_hours_display\
                if allocation.holiday_status_id.request_unit in ['hour']\
                else allocation.number_of_days_display
            max_leaves += future_leaves
            allocation_data.update({
                'max_leaves': max_leaves,
                'accrual_bonus': future_leaves,
                'virtual_remaining_leaves': max_leaves,
                'remaining_leaves': max_leaves,
                'leaves_taken': 0,
                'virtual_leaves_taken': 0,
            })

        for employee in employees:
            for leave_type in leave_types:
                allocations_with_date_to = self.env['hr.leave.allocation']
                allocations_without_date_to = self.env['hr.leave.allocation']
                for leave_allocation in allocations_per_employee_type[employee][leave_type]:
                    if leave_allocation.date_to:
                        allocations_with_date_to |= leave_allocation
                    else:
                        allocations_without_date_to |= leave_allocation
                sorted_leave_allocations = allocations_with_date_to.sorted(key='date_to') + allocations_without_date_to

                if leave_type.request_unit in ['day', 'half_day']:
                    leave_duration_field = 'number_of_days'
                    leave_unit = 'days'
                else:
                    leave_duration_field = 'number_of_hours_display'
                    leave_unit = 'hours'

                leave_type_data = allocations_leaves_consumed[employee][leave_type]
                for leave in leaves_per_employee_type[employee][leave_type].sorted('date_from'):
                    leave_duration = leave[leave_duration_field]
                    skip_excess = False

                    if sorted_leave_allocations.filtered(lambda alloc: alloc.allocation_type == 'accrual') and leave.date_from.date() > target_date:
                        to_recheck_leaves_per_leave_type[employee][leave_type]['to_recheck_leaves'] |= leave
                        skip_excess = True
                        continue

                    if leave_type.requires_allocation == 'yes':
                        for allocation in sorted_leave_allocations:
                            # We don't want to include future leaves linked to accruals into the total count of available leaves.
                            # However, we'll need to check if those leaves take more than what will be accrued in total of those days
                            # to give a warning if the total exceeds what will be accrued.
                            if allocation.date_from > leave.date_to.date() or (allocation.date_to and allocation.date_to < leave.date_from.date()):
                                continue
                            interval_start = max(
                                leave.date_from,
                                datetime.combine(allocation.date_from, time.min)
                            )
                            interval_end = min(
                                leave.date_to,
                                datetime.combine(allocation.date_to, time.max)
                                if allocation.date_to else leave.date_to
                            )
                            duration = leave[leave_duration_field]
                            if leave.date_from != interval_start or leave.date_to != interval_end:
                                duration_info = employee._get_calendar_attendances(interval_start.replace(tzinfo=pytz.UTC), interval_end.replace(tzinfo=pytz.UTC))
                                duration = duration_info['hours' if leave_unit == 'hours' else 'days']
                            max_allowed_duration = min(
                                duration,
                                leave_type_data[allocation]['virtual_remaining_leaves']
                            )

                            if not max_allowed_duration:
                                continue

                            allocated_time = min(max_allowed_duration, leave_duration)
                            leave_type_data[allocation]['virtual_leaves_taken'] += allocated_time
                            leave_type_data[allocation]['virtual_remaining_leaves'] -= allocated_time
                            if leave.state == 'validate':
                                leave_type_data[allocation]['leaves_taken'] += allocated_time
                                leave_type_data[allocation]['remaining_leaves'] -= allocated_time

                            leave_duration -= allocated_time
                            if not leave_duration:
                                break
                        if round(leave_duration, 2) > 0 and not skip_excess:
                            to_recheck_leaves_per_leave_type[employee][leave_type]['excess_days'][leave.date_to.date()] = {
                                'amount': leave_duration,
                                'is_virtual': leave.state != 'validate',
                                'leave_id': leave.id,
                            }
                    else:
                        if leave_unit == 'hours':
                            allocated_time = leave.number_of_hours_display
                        else:
                            allocated_time = leave.number_of_days_display
                        leave_type_data[False]['virtual_leaves_taken'] += allocated_time
                        leave_type_data[False]['virtual_remaining_leaves'] = 0
                        leave_type_data[False]['remaining_leaves'] = 0
                        if leave.state == 'validate':
                            leave_type_data[False]['leaves_taken'] += allocated_time

        for employee in to_recheck_leaves_per_leave_type:
            for leave_type in to_recheck_leaves_per_leave_type[employee]:
                content = to_recheck_leaves_per_leave_type[employee][leave_type]
                consumed_content = allocations_leaves_consumed[employee][leave_type]
                if content['to_recheck_leaves']:
                    date_to_simulate = max(content['to_recheck_leaves'].mapped('date_from')).date()
                    latest_accrual_bonus = 0
                    date_accrual_bonus = 0
                    virtual_remaining = 0
                    additional_leaves_duration = 0
                    for allocation in consumed_content:
                        latest_accrual_bonus += allocation and allocation._get_future_leaves_on(date_to_simulate)
                        date_accrual_bonus += consumed_content[allocation]['accrual_bonus']
                        virtual_remaining += consumed_content[allocation]['virtual_remaining_leaves']
                    for leave in content['to_recheck_leaves']:
                        additional_leaves_duration += leave.number_of_hours if leave_type.request_unit == 'hours' else leave.number_of_days
                    latest_remaining = virtual_remaining - date_accrual_bonus + latest_accrual_bonus
                    content['exceeding_duration'] = round(min(0, latest_remaining - additional_leaves_duration), 2)

        return (allocations_leaves_consumed, to_recheck_leaves_per_leave_type)

```

## File: models\hr_employee_base.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, date
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError
from odoo.tools.float_utils import float_round

from odoo.addons.resource.models.utils import HOURS_PER_DAY


class HrEmployeeBase(models.AbstractModel):
    _inherit = "hr.employee.base"

    leave_manager_id = fields.Many2one(
        'res.users', string='Time Off',
        compute='_compute_leave_manager', store=True, readonly=False,
        domain="[('share', '=', False), ('company_ids', 'in', company_id)]",
        help='Select the user responsible for approving "Time Off" of this employee.\n'
             'If empty, the approval is done by an Administrator or Approver (determined in settings/users).')
    remaining_leaves = fields.Float(
        compute='_compute_remaining_leaves', string='Remaining Paid Time Off',
        help='Total number of paid time off allocated to this employee, change this value to create allocation/time off request. '
             'Total based on all the time off types without overriding limit.')
    current_leave_state = fields.Selection(compute='_compute_leave_status', string="Current Time Off Status",
        selection=[
            ('draft', 'New'),
            ('confirm', 'Waiting Approval'),
            ('refuse', 'Refused'),
            ('validate1', 'Waiting Second Approval'),
            ('validate', 'Approved'),
            ('cancel', 'Cancelled')
        ])
    leave_date_from = fields.Date('From Date', compute='_compute_leave_status')
    leave_date_to = fields.Date('To Date', compute='_compute_leave_status')
    leaves_count = fields.Float('Number of Time Off', compute='_compute_remaining_leaves')
    allocation_count = fields.Float('Total number of days allocated.', compute='_compute_allocation_count')
    allocations_count = fields.Integer('Total number of allocations', compute="_compute_allocation_count")
    show_leaves = fields.Boolean('Able to see Remaining Time Off', compute='_compute_show_leaves')
    is_absent = fields.Boolean('Absent Today', compute='_compute_leave_status', search='_search_absent_employee')
    allocation_display = fields.Char(compute='_compute_allocation_remaining_display')
    allocation_remaining_display = fields.Char(compute='_compute_allocation_remaining_display')
    hr_icon_display = fields.Selection(selection_add=[
        ('presence_holiday_absent', 'On leave'),
        ('presence_holiday_present', 'Present but on leave')])

    def _compute_presence_state(self):
        super()._compute_presence_state()
        employees = self.filtered(lambda employee: employee.hr_presence_state != 'present' and employee.is_absent)
        employees.update({'hr_presence_state': 'absent'})

    def _get_remaining_leaves(self):
        """ Helper to compute the remaining leaves for the current employees
            :returns dict where the key is the employee id, and the value is the remain leaves
        """
        self._cr.execute("""
            SELECT
                sum(h.number_of_days) AS days,
                h.employee_id
            FROM
                (
                    SELECT holiday_status_id, number_of_days,
                        state, employee_id
                    FROM hr_leave_allocation
                    UNION ALL
                    SELECT holiday_status_id, (number_of_days * -1) as number_of_days,
                        state, employee_id
                    FROM hr_leave
                ) h
                join hr_leave_type s ON (s.id=h.holiday_status_id)
            WHERE
                s.active = true AND h.state='validate' AND
                s.requires_allocation='yes' AND
                h.employee_id in %s
            GROUP BY h.employee_id""", (tuple(self.ids),))
        return {row['employee_id']: row['days'] for row in self._cr.dictfetchall()}

    def _compute_remaining_leaves(self):
        remaining = {}
        if self.ids:
            remaining = self._get_remaining_leaves()
        for employee in self:
            value = float_round(remaining.get(employee.id, 0.0), precision_digits=2)
            employee.leaves_count = value
            employee.remaining_leaves = value

    def _compute_allocation_count(self):
        # Don't get allocations that are expired
        current_date = date.today()
        data = self.env['hr.leave.allocation']._read_group([
            ('employee_id', 'in', self.ids),
            ('holiday_status_id.active', '=', True),
            ('holiday_status_id.requires_allocation', '=', 'yes'),
            ('state', '=', 'validate'),
            ('date_from', '<=', current_date),
            '|',
            ('date_to', '=', False),
            ('date_to', '>=', current_date),
        ], ['employee_id'], ['__count', 'number_of_days:sum'])
        rg_results = {employee.id: (count, days) for employee, count, days in data}
        for employee in self:
            count, days = rg_results.get(employee.id, (0, 0))
            employee.allocation_count = float_round(days, precision_digits=2)
            employee.allocations_count = count

    def _compute_allocation_remaining_display(self):
        current_date = date.today()
        allocations = self.env['hr.leave.allocation'].search([('employee_id', 'in', self.ids)])
        leaves_taken = self._get_consumed_leaves(allocations.holiday_status_id)[0]
        for employee in self:
            employee_remaining_leaves = 0
            employee_max_leaves = 0
            for leave_type in leaves_taken[employee]:
                if leave_type.requires_allocation == 'no':
                    continue
                for allocation in leaves_taken[employee][leave_type]:
                    if allocation and allocation.date_from <= current_date\
                            and (not allocation.date_to or allocation.date_to >= current_date):
                        virtual_remaining_leaves = leaves_taken[employee][leave_type][allocation]['virtual_remaining_leaves']
                        employee_remaining_leaves += virtual_remaining_leaves\
                            if leave_type.request_unit in ['day', 'half_day']\
                            else virtual_remaining_leaves / (employee.resource_calendar_id.hours_per_day or HOURS_PER_DAY)
                        employee_max_leaves += allocation.number_of_days
            employee.allocation_remaining_display = "%g" % float_round(employee_remaining_leaves, precision_digits=2)
            employee.allocation_display = "%g" % float_round(employee_max_leaves, precision_digits=2)

    def _compute_presence_icon(self):
        super()._compute_presence_icon()
        employees_absent = self.filtered(
            lambda employee: employee.hr_presence_state != 'present' and employee.is_absent)
        employees_absent.update({'hr_icon_display': 'presence_holiday_absent'})
        employees_present = self.filtered(
            lambda employee: employee.hr_presence_state == 'present' and employee.is_absent)
        employees_present.update({'hr_icon_display': 'presence_holiday_present'})

    def _compute_leave_status(self):
        # Used SUPERUSER_ID to forcefully get status of other user's leave, to bypass record rule
        holidays = self.env['hr.leave'].sudo().search([
            ('employee_id', 'in', self.ids),
            ('date_from', '<=', fields.Datetime.now()),
            ('date_to', '>=', fields.Datetime.now()),
            ('state', '=', 'validate'),
        ])
        leave_data = {}
        for holiday in holidays:
            leave_data[holiday.employee_id.id] = {}
            leave_data[holiday.employee_id.id]['leave_date_from'] = holiday.date_from.date()
            leave_data[holiday.employee_id.id]['leave_date_to'] = holiday.date_to.date()
            leave_data[holiday.employee_id.id]['current_leave_state'] = holiday.state

        for employee in self:
            employee.leave_date_from = leave_data.get(employee.id, {}).get('leave_date_from')
            employee.leave_date_to = leave_data.get(employee.id, {}).get('leave_date_to')
            employee.current_leave_state = leave_data.get(employee.id, {}).get('current_leave_state')
            employee.is_absent = leave_data.get(employee.id) and leave_data.get(employee.id).get('current_leave_state') == 'validate'

    @api.depends('parent_id')
    def _compute_leave_manager(self):
        for employee in self:
            previous_manager = employee._origin.parent_id.user_id
            manager = employee.parent_id.user_id
            if manager and employee.leave_manager_id == previous_manager or not employee.leave_manager_id:
                employee.leave_manager_id = manager
            elif not employee.leave_manager_id:
                employee.leave_manager_id = False

    def _compute_show_leaves(self):
        show_leaves = self.env['res.users'].has_group('hr_holidays.group_hr_holidays_user')
        for employee in self:
            if show_leaves or employee.user_id == self.env.user:
                employee.show_leaves = True
            else:
                employee.show_leaves = False

    def _search_absent_employee(self, operator, value):
        if operator not in ('=', '!=') or not isinstance(value, bool):
            raise UserError(_('Operation not supported'))
        # This search is only used for the 'Absent Today' filter however
        # this only returns employees that are absent right now.
        today_date = datetime.utcnow().date()
        today_start = fields.Datetime.to_string(today_date)
        today_end = fields.Datetime.to_string(today_date + relativedelta(hours=23, minutes=59, seconds=59))
        holidays = self.env['hr.leave'].sudo().search([
            ('employee_id', '!=', False),
            ('state', '=', 'validate'),
            ('date_from', '<=', today_end),
            ('date_to', '>=', today_start),
        ])
        operator = ['in', 'not in'][(operator == '=') != value]
        return [('id', operator, holidays.mapped('employee_id').ids)]

    @api.model_create_multi
    def create(self, vals_list):
        if self.env.context.get('salary_simulation'):
            return super().create(vals_list)
        approver_group = self.env.ref('hr_holidays.group_hr_holidays_responsible', raise_if_not_found=False)
        group_updates = []
        for vals in vals_list:
            if 'parent_id' in vals:
                manager = self.env['hr.employee'].browse(vals['parent_id']).user_id
                vals['leave_manager_id'] = vals.get('leave_manager_id', manager.id)
            if approver_group and vals.get('leave_manager_id'):
                group_updates.append((4, vals['leave_manager_id']))
        if group_updates:
            approver_group.sudo().write({'users': group_updates})
        return super().create(vals_list)

    def write(self, values):
        if 'parent_id' in values:
            manager = self.env['hr.employee'].browse(values['parent_id']).user_id
            if manager:
                to_change = self.filtered(lambda e: e.leave_manager_id == e.parent_id.user_id or not e.leave_manager_id)
                to_change.write({'leave_manager_id': values.get('leave_manager_id', manager.id)})

        old_managers = self.env['res.users']
        if 'leave_manager_id' in values:
            old_managers = self.mapped('leave_manager_id')
            if values['leave_manager_id']:
                leave_manager = self.env['res.users'].browse(values['leave_manager_id'])
                old_managers -= leave_manager
                approver_group = self.env.ref('hr_holidays.group_hr_holidays_responsible', raise_if_not_found=False)
                if approver_group and not leave_manager.has_group('hr_holidays.group_hr_holidays_responsible'):
                    leave_manager.sudo().write({'groups_id': [(4, approver_group.id)]})

        res = super().write(values)
        # remove users from the Responsible group if they are no longer leave managers
        old_managers.sudo()._clean_leave_responsible_users()

        # Change the resource calendar of the employee's leaves in the future
        # Other modules can disable this behavior by setting the context key
        # 'no_leave_resource_calendar_update'
        if 'resource_calendar_id' in values and not self.env.context.get('no_leave_resource_calendar_update'):
            try:
                self.env['hr.leave'].search([
                    ('employee_id', 'in', self.ids),
                    ('resource_calendar_id', '!=', int(values['resource_calendar_id'])),
                    ('date_from', '>', fields.Datetime.now())]).write({'resource_calendar_id': values['resource_calendar_id']})
            except ValidationError:
                raise ValidationError(_("Changing this working schedule results in the affected employee(s) not having enough "
                                        "leaves allocated to accomodate for their leaves already taken in the future. Please "
                                        "review this employee's leaves and adjust their allocation accordingly."))

        if 'parent_id' in values or 'department_id' in values:
            today_date = fields.Datetime.now()
            hr_vals = {}
            if values.get('parent_id') is not None:
                hr_vals['manager_id'] = values['parent_id']
            if values.get('department_id') is not None:
                hr_vals['department_id'] = values['department_id']
            holidays = self.env['hr.leave'].sudo().search(['|', ('state', 'in', ['draft', 'confirm']), ('date_from', '>', today_date), ('employee_id', 'in', self.ids)])
            holidays.write(hr_vals)
            allocations = self.env['hr.leave.allocation'].sudo().search([('state', 'in', ['draft', 'confirm']), ('employee_id', 'in', self.ids)])
            allocations.write(hr_vals)
        return res

```

## File: models\hr_leave.py

```python
import logging
import pytz

from collections import namedtuple, defaultdict

from datetime import datetime, timedelta, time
from dateutil.relativedelta import relativedelta
from math import ceil
from pytz import timezone, UTC

from odoo.addons.base.models.ir_model import MODULE_UNINSTALL_FLAG

from odoo import api, Command, fields, models, tools
from odoo.addons.base.models.res_partner import _tz_get
from odoo.addons.resource.models.utils import float_to_time, HOURS_PER_DAY
from odoo.exceptions import AccessError, UserError, ValidationError
from odoo.tools.float_utils import float_round, float_compare
from odoo.tools.misc import format_date
from odoo.tools.translate import _
from odoo.osv import expression

_logger = logging.getLogger(__name__)

# Used to agglomerate the attendances in order to find the hour_from and hour_to
# See _compute_date_from_to
DummyAttendance = namedtuple('DummyAttendance', 'hour_from, hour_to, dayofweek, day_period, week_type')

def get_employee_from_context(values, context, user_employee_id):
    employee_ids_list = [value[2] for value in values.get('employee_ids', []) if len(value) == 3 and value[0] == Command.SET]
    employee_ids = employee_ids_list[-1] if employee_ids_list else []
    employee_id_value = employee_ids[0] if employee_ids else False
    return employee_id_value or context.get('default_employee_id', context.get('employee_id', user_employee_id))

class HolidaysRequest(models.Model):
    """ Time Off Requests Access specifications

     - a regular employee / user
      - can see all leaves;
      - cannot see name field of leaves belonging to other user as it may contain
        private information that we don't want to share to other people than
        HR people;
      - can modify only its own not validated leaves (except writing on state to
        bypass approval);
      - can discuss on its leave requests;
      - can reset only its own leaves;
      - cannot validate any leaves;
     - an Officer
      - can see all leaves;
      - can validate "HR" single validation leaves from people if
       - he is the employee manager;
       - he is the department manager;
       - he is member of the same department;
       - target employee has no manager and no department manager;
      - can validate "Manager" single validation leaves from people if
       - he is the employee manager;
       - he is the department manager;
       - target employee has no manager and no department manager;
      - can first validate "Both" double validation leaves from people like "HR"
        single validation, moving the leaves to validate1 state;
      - cannot validate its own leaves;
      - can reset only its own leaves;
      - can refuse all leaves;
     - a Manager
      - can do everything he wants

    On top of that multicompany rules apply based on company defined on the
    leave request leave type.
    """
    _name = "hr.leave"
    _description = "Time Off"
    _order = "date_from desc"
    _inherit = ['mail.thread.main.attachment', 'mail.activity.mixin']
    _mail_post_access = 'read'

    @api.model
    def default_get(self, fields_list):
        defaults = super(HolidaysRequest, self).default_get(fields_list)
        defaults = self._default_get_request_dates(defaults)

        lt = self.env['hr.leave.type']
        if self.env.context.get('holiday_status_display_name', True) and 'holiday_status_id' in fields_list and not defaults.get('holiday_status_id'):
            lt = self.env['hr.leave.type'].search(['|', ('requires_allocation', '=', 'no'), ('has_valid_allocation', '=', True)], limit=1, order='sequence')
            if lt:
                defaults['holiday_status_id'] = lt.id
                defaults['request_unit_custom'] = False

        if 'state' in fields_list and not defaults.get('state'):
            defaults['state'] = 'confirm' if lt.leave_validation_type != 'no_validation' else 'draft'

        if 'request_date_from' in fields_list and 'request_date_from' not in defaults:
            defaults['request_date_from'] = fields.Date.today()
        if 'request_date_to' in fields_list and 'request_date_to' not in defaults:
            defaults['request_date_to'] = fields.Date.today()

        return defaults

    def _default_get_request_dates(self, values):
        # The UI views initialize date_{from,to} due to how calendar views work.
        # However it is request_date_{from,to} that should be used instead.
        # Instead of overwriting all the javascript methods to use
        # request_date_{from,to} instead of date_{from,to}, we just convert
        # date_{from,to} to request_date_{from,to} here.

        # Request dates are determined during an onchange scenario.
        # To ensure that the values are correct in the client context (UI),
        # the timezone must be applied (because no processing is carried out
        # when these dates are received on the frontend).
        # Note:
        # Without the application of the timezone, days based on UTC datetimes
        # will be returned (and will therefore not be correct for the client).
        client_tz = timezone(self._context.get('tz') or self.env.user.tz or 'UTC')
        if values.get('date_from'):
            if not values.get('request_date_from'):
                values['request_date_from'] = pytz.utc.localize(values['date_from']).astimezone(client_tz)
            del values['date_from']
        if values.get('date_to'):
            if not values.get('request_date_to'):
                values['request_date_to'] = pytz.utc.localize(values['date_to']).astimezone(client_tz)
            del values['date_to']
        return values

    active = fields.Boolean(default=True, readonly=True)
    # description
    name = fields.Char('Description', compute='_compute_description', inverse='_inverse_description', search='_search_description', compute_sudo=False, copy=False)
    private_name = fields.Char('Time Off Description', groups='hr_holidays.group_hr_holidays_user')
    state = fields.Selection(
        [
            ('draft', 'To Submit'),
            ('confirm', 'To Approve'),
            ('refuse', 'Refused'),
            ('validate1', 'Second Approval'),
            ('validate', 'Approved')
        ], string='Status', compute='_compute_state', store=True, tracking=True, copy=False, readonly=False,
        help="The status is set to 'To Submit', when a time off request is created." +
        "\nThe status is 'To Approve', when time off request is confirmed by user." +
        "\nThe status is 'Refused', when time off request is refused by manager." +
        "\nThe status is 'Approved', when time off request is approved by manager.")
    report_note = fields.Text('HR Comments', copy=False, groups="hr_holidays.group_hr_holidays_manager")
    user_id = fields.Many2one('res.users', string='User', related='employee_id.user_id', related_sudo=True, compute_sudo=True, store=True, readonly=True, index=True)
    manager_id = fields.Many2one('hr.employee', compute='_compute_from_employee_id', store=True, readonly=False)
    is_user_only_responsible = fields.Boolean(compute="_compute_is_user_only_responsible")
    # leave type configuration
    holiday_status_id = fields.Many2one(
        "hr.leave.type", compute='_compute_from_employee_id',
        store=True, string="Time Off Type",
        required=True, readonly=False,
        domain="""[
            ('company_id', 'in', [employee_company_id, False]),
            '|',
                ('requires_allocation', '=', 'no'),
                ('has_valid_allocation', '=', True),
        ]""",
        tracking=True)
    color = fields.Integer("Color", related='holiday_status_id.color')
    validation_type = fields.Selection(string='Validation Type', related='holiday_status_id.leave_validation_type', readonly=False)
    # HR data

    employee_id = fields.Many2one(
        'hr.employee', compute='_compute_from_employee_ids', store=True, string='Employee', index=True, readonly=False, ondelete="restrict",
        tracking=True, compute_sudo=False,
        domain=lambda self: self._get_employee_domain())
    employee_company_id = fields.Many2one(related='employee_id.company_id', string="Employee Company", store=True)
    company_id = fields.Many2one('res.company', compute='_compute_company_id', store=True)
    active_employee = fields.Boolean(related='employee_id.active', string='Employee Active')
    tz_mismatch = fields.Boolean(compute='_compute_tz_mismatch')
    tz = fields.Selection(_tz_get, compute='_compute_tz')
    department_id = fields.Many2one(
        'hr.department', compute='_compute_department_id', store=True, string='Department', readonly=False)
    notes = fields.Text('Reasons', readonly=False)
    # duration
    resource_calendar_id = fields.Many2one('resource.calendar', compute='_compute_resource_calendar_id', store=True, readonly=False, copy=False)
    # These dates are computed based on request_date_{to,from} and should
    # therefore never be set directly.
    date_from = fields.Datetime(
        'Start Date', compute='_compute_date_from_to', store=True, index=True, tracking=True)
    date_to = fields.Datetime(
        'End Date', compute='_compute_date_from_to', store=True, tracking=True)
    number_of_days = fields.Float(
        'Duration (Days)', compute='_compute_duration', store=True, tracking=True,
        help='Number of days of the time off request. Used in the calculation.')
    number_of_hours = fields.Float(
        'Duration (Hours)', compute='_compute_duration', store=True, tracking=True,
        help='Number of hours of the time off request. Used in the calculation.')
    last_several_days = fields.Boolean("All day", compute="_compute_last_several_days")
    number_of_days_display = fields.Float(
        'Duration in days', compute='_compute_number_of_days_display',
        help='Number of days of the time off request according to your working schedule. Used for interface.')
    number_of_hours_display = fields.Float(
        'Duration in hours', compute='_compute_number_of_hours_display', readonly=True,
        help='Number of hours of the time off request according to your working schedule. Used for interface.')
    number_of_hours_text = fields.Char(compute='_compute_number_of_hours_text')
    duration_display = fields.Char('Requested (Days/Hours)', compute='_compute_duration_display', store=True,
        help="Field allowing to see the leave request duration in days or hours depending on the leave_type_request_unit")    # details
    # details
    meeting_id = fields.Many2one('calendar.event', string='Meeting', copy=False)
    parent_id = fields.Many2one('hr.leave', string='Parent', copy=False)
    linked_request_ids = fields.One2many('hr.leave', 'parent_id', string='Linked Requests')
    holiday_type = fields.Selection([
        ('employee', 'By Employee'),
        ('company', 'By Company'),
        ('department', 'By Department'),
        ('category', 'By Employee Tag')],
        string='Allocation Mode', readonly=False, required=True, default='employee',
        help='By Employee: Allocation/Request for individual Employee, By Employee Tag: Allocation/Request for group of employees in category')
    employee_ids = fields.Many2many(
        'hr.employee', compute='_compute_from_holiday_type', store=True, string='Employees', readonly=True, groups="hr_holidays.group_hr_holidays_responsible",
        domain=lambda self: self._get_employee_domain())
    multi_employee = fields.Boolean(
        compute='_compute_from_employee_ids', store=True, compute_sudo=False,
        help='Holds whether this allocation concerns more than 1 employee')
    category_id = fields.Many2one(
        'hr.employee.category', compute='_compute_from_holiday_type', store=True, string='Employee Tag',
        readonly=False, help='Category of Employee')
    mode_company_id = fields.Many2one(
        'res.company', compute='_compute_from_holiday_type', store=True, string='Company Mode',
        readonly=False)
    first_approver_id = fields.Many2one(
        'hr.employee', string='First Approval', readonly=True, copy=False,
        help='This area is automatically filled by the user who validate the time off')
    second_approver_id = fields.Many2one(
        'hr.employee', string='Second Approval', readonly=True, copy=False,
        help='This area is automatically filled by the user who validate the time off with second level (If time off type need second validation)')
    can_reset = fields.Boolean('Can reset', compute='_compute_can_reset')
    can_approve = fields.Boolean('Can Approve', compute='_compute_can_approve')
    can_cancel = fields.Boolean('Can Cancel', compute='_compute_can_cancel')

    attachment_ids = fields.One2many('ir.attachment', 'res_id', string="Attachments")
    # To display in form view
    supported_attachment_ids = fields.Many2many(
        'ir.attachment', string="Attach File", compute='_compute_supported_attachment_ids',
        inverse='_inverse_supported_attachment_ids')
    supported_attachment_ids_count = fields.Integer(compute='_compute_supported_attachment_ids')
    # UX fields
    all_employee_ids = fields.Many2many('hr.employee', compute='_compute_all_employees', compute_sudo=True)
    leave_type_request_unit = fields.Selection(related='holiday_status_id.request_unit', readonly=True)
    leave_type_support_document = fields.Boolean(related="holiday_status_id.support_document")
    # Interface fields used when not using hour-based computation
    # These are the fields that should be used to manipulate the start- and
    # end-dates of the leave request. date_from and date_to are computed and
    # should therefore not be set directly.
    request_date_from = fields.Date('Request Start Date')
    request_date_to = fields.Date('Request End Date')
    # Interface fields used when using hour-based computation
    request_hour_from = fields.Selection([
        ('0', '12:00 AM'), ('0.5', '12:30 AM'),
        ('1', '1:00 AM'), ('1.5', '1:30 AM'),
        ('2', '2:00 AM'), ('2.5', '2:30 AM'),
        ('3', '3:00 AM'), ('3.5', '3:30 AM'),
        ('4', '4:00 AM'), ('4.5', '4:30 AM'),
        ('5', '5:00 AM'), ('5.5', '5:30 AM'),
        ('6', '6:00 AM'), ('6.5', '6:30 AM'),
        ('7', '7:00 AM'), ('7.5', '7:30 AM'),
        ('8', '8:00 AM'), ('8.5', '8:30 AM'),
        ('9', '9:00 AM'), ('9.5', '9:30 AM'),
        ('10', '10:00 AM'), ('10.5', '10:30 AM'),
        ('11', '11:00 AM'), ('11.5', '11:30 AM'),
        ('12', '12:00 PM'), ('12.5', '12:30 PM'),
        ('13', '1:00 PM'), ('13.5', '1:30 PM'),
        ('14', '2:00 PM'), ('14.5', '2:30 PM'),
        ('15', '3:00 PM'), ('15.5', '3:30 PM'),
        ('16', '4:00 PM'), ('16.5', '4:30 PM'),
        ('17', '5:00 PM'), ('17.5', '5:30 PM'),
        ('18', '6:00 PM'), ('18.5', '6:30 PM'),
        ('19', '7:00 PM'), ('19.5', '7:30 PM'),
        ('20', '8:00 PM'), ('20.5', '8:30 PM'),
        ('21', '9:00 PM'), ('21.5', '9:30 PM'),
        ('22', '10:00 PM'), ('22.5', '10:30 PM'),
        ('23', '11:00 PM'), ('23.5', '11:30 PM')], string='Hour from')
    request_hour_to = fields.Selection([
        ('0', '12:00 AM'), ('0.5', '12:30 AM'),
        ('1', '1:00 AM'), ('1.5', '1:30 AM'),
        ('2', '2:00 AM'), ('2.5', '2:30 AM'),
        ('3', '3:00 AM'), ('3.5', '3:30 AM'),
        ('4', '4:00 AM'), ('4.5', '4:30 AM'),
        ('5', '5:00 AM'), ('5.5', '5:30 AM'),
        ('6', '6:00 AM'), ('6.5', '6:30 AM'),
        ('7', '7:00 AM'), ('7.5', '7:30 AM'),
        ('8', '8:00 AM'), ('8.5', '8:30 AM'),
        ('9', '9:00 AM'), ('9.5', '9:30 AM'),
        ('10', '10:00 AM'), ('10.5', '10:30 AM'),
        ('11', '11:00 AM'), ('11.5', '11:30 AM'),
        ('12', '12:00 PM'), ('12.5', '12:30 PM'),
        ('13', '1:00 PM'), ('13.5', '1:30 PM'),
        ('14', '2:00 PM'), ('14.5', '2:30 PM'),
        ('15', '3:00 PM'), ('15.5', '3:30 PM'),
        ('16', '4:00 PM'), ('16.5', '4:30 PM'),
        ('17', '5:00 PM'), ('17.5', '5:30 PM'),
        ('18', '6:00 PM'), ('18.5', '6:30 PM'),
        ('19', '7:00 PM'), ('19.5', '7:30 PM'),
        ('20', '8:00 PM'), ('20.5', '8:30 PM'),
        ('21', '9:00 PM'), ('21.5', '9:30 PM'),
        ('22', '10:00 PM'), ('22.5', '10:30 PM'),
        ('23', '11:00 PM'), ('23.5', '11:30 PM')], string='Hour to')
    # used only when the leave is taken in half days
    request_date_from_period = fields.Selection([
        ('am', 'Morning'), ('pm', 'Afternoon')],
        string="Date Period Start", default='am')
    # request type
    request_unit_half = fields.Boolean('Half Day', compute='_compute_request_unit_half', store=True, readonly=False)
    request_unit_hours = fields.Boolean('Custom Hours', compute='_compute_request_unit_hours', store=True, readonly=False)
    # view
    is_hatched = fields.Boolean('Hatched', compute='_compute_is_hatched')
    is_striked = fields.Boolean('Striked', compute='_compute_is_hatched')
    has_mandatory_day = fields.Boolean(compute='_compute_has_mandatory_day')
    leave_type_increases_duration = fields.Boolean(compute='_compute_leave_type_increases_duration')

    _sql_constraints = [
        ('type_value',
         "CHECK((holiday_type='employee' AND (employee_id IS NOT NULL OR multi_employee IS TRUE)) or "
         "(holiday_type='company' AND mode_company_id IS NOT NULL) or "
         "(holiday_type='category' AND category_id IS NOT NULL) or "
         "(holiday_type='department' AND department_id IS NOT NULL) )",
         "The employee, department, company or employee category of this request is missing. Please make sure that your user login is linked to an employee."),
        ('date_check2', "CHECK ((date_from <= date_to))", "The start date must be before or equal to the end date."),
        ('date_check3', "CHECK ((request_date_from <= request_date_to))", "The request start date must be before or equal to the request end date."),
        ('duration_check', "CHECK ( number_of_days >= 0 )", "If you want to change the number of days you should use the 'period' mode"),
    ]

    def _auto_init(self):
        res = super(HolidaysRequest, self)._auto_init()
        tools.create_index(self._cr, 'hr_leave_date_to_date_from_index',
                           self._table, ['date_to', 'date_from'])
        return res

    @api.depends('employee_id', 'employee_ids')
    def _compute_all_employees(self):
        for leave in self:
            leave.all_employee_ids = leave.employee_id | leave.employee_ids

    @api.depends_context('uid')
    def _compute_description(self):
        self.check_access_rights('read')
        self.check_access_rule('read')

        is_officer = self.user_has_groups('hr_holidays.group_hr_holidays_user')

        for leave in self:
            if is_officer or leave.user_id == self.env.user or leave.employee_id.leave_manager_id == self.env.user:
                leave.name = leave.sudo().private_name
            else:
                leave.name = '*****'

    def _inverse_description(self):
        is_officer = self.user_has_groups('hr_holidays.group_hr_holidays_user')

        for leave in self:
            if is_officer or leave.user_id == self.env.user or leave.employee_id.leave_manager_id == self.env.user:
                leave.sudo().private_name = leave.name

    def _search_description(self, operator, value):
        is_officer = self.user_has_groups('hr_holidays.group_hr_holidays_user')
        domain = [('private_name', operator, value)]

        if not is_officer:
            domain = expression.AND([domain, [('user_id', '=', self.env.user.id)]])
        query = self.sudo()._search(domain)
        return [('id', 'inselect', query.select())]

    @api.depends('holiday_status_id')
    def _compute_state(self):
        for leave in self:
            leave.state = 'confirm' if leave.validation_type != 'no_validation' else 'draft'

    @api.depends('holiday_type', 'employee_id', 'department_id', 'mode_company_id')
    def _compute_resource_calendar_id(self):
        for leave in self:
            calendar = False
            if leave.holiday_type == 'employee':
                calendar = leave.employee_id.resource_calendar_id
                # YTI: Crappy hack: Move this to a new dedicated hr_holidays_contract module
                # We use the request dates to find the contracts, because date_from
                # and date_to are not set yet at this point. Since these dates are
                # used to get the contracts for which these leaves apply and
                # contract start- and end-dates are just dates (and not datetimes)
                # these dates are comparable.
                if 'hr.contract' in self.env and leave.employee_id:
                    contracts = self.env['hr.contract'].search([
                        '|', ('state', 'in', ['open', 'close']),
                             '&', ('state', '=', 'draft'),
                                  ('kanban_state', '=', 'done'),
                        ('employee_id', '=', leave.employee_id.id),
                        ('date_start', '<=', leave.request_date_to),
                        '|', ('date_end', '=', False),
                             ('date_end', '>=', leave.request_date_from),
                    ])
                    if contracts:
                        # If there are more than one contract they should all have the
                        # same calendar, otherwise a constraint is violated.
                        calendar = contracts[:1].resource_calendar_id
            elif leave.holiday_type == 'department':
                calendar = leave.department_id.company_id.resource_calendar_id
            elif leave.holiday_type == 'company':
                calendar = leave.mode_company_id.resource_calendar_id
            leave.resource_calendar_id = calendar or self.env.company.resource_calendar_id

    @api.depends('request_date_from_period', 'request_hour_from', 'request_hour_to', 'request_date_from', 'request_date_to',
                 'request_unit_half', 'request_unit_hours', 'employee_id')
    def _compute_date_from_to(self):
        for holiday in self:
            if not holiday.request_date_from:
                holiday.date_from = False
            elif not holiday.request_unit_half and not holiday.request_unit_hours and not holiday.request_date_to:
                holiday.date_to = False
            else:
                if (holiday.request_unit_half or holiday.request_unit_hours) and holiday.request_date_to != holiday.request_date_from:
                    holiday.request_date_to = holiday.request_date_from


                day_period = {
                    'am': 'morning',
                    'pm': 'afternoon'
                }.get(holiday.request_date_from_period, None) if holiday.request_unit_half else None

                attendance_from, attendance_to = holiday._get_attendances(holiday.request_date_from, holiday.request_date_to, day_period=day_period)

                compensated_request_date_from = holiday.request_date_from
                compensated_request_date_to = holiday.request_date_to

                if holiday.request_unit_hours:
                    hour_from = holiday.request_hour_from
                    hour_to = holiday.request_hour_to
                else:
                    hour_from = attendance_from.hour_from
                    hour_to = attendance_to.hour_to

                holiday.date_from = self._to_utc(compensated_request_date_from, hour_from, holiday.employee_id or holiday)
                holiday.date_to = self._to_utc(compensated_request_date_to, hour_to, holiday.employee_id or holiday)

    @api.depends('holiday_status_id', 'request_unit_hours')
    def _compute_request_unit_half(self):
        for holiday in self:
            if holiday.holiday_status_id or holiday.request_unit_hours:
                holiday.request_unit_half = False

    @api.depends('holiday_status_id', 'request_unit_half')
    def _compute_request_unit_hours(self):
        for holiday in self:
            if holiday.holiday_status_id or holiday.request_unit_half:
                holiday.request_unit_hours = False

    @api.depends('employee_ids')
    def _compute_from_employee_ids(self):
        for holiday in self:
            if len(holiday.employee_ids) == 1:
                holiday.employee_id = holiday.employee_ids[0]._origin
            else:
                holiday.employee_id = False
            holiday.multi_employee = (len(holiday.employee_ids) > 1)

    def _get_employee_domain(self):
        domain = [
            ('active', '=', True),
            ('company_id', 'in', self.env.companies.ids),
        ]
        if not self.env.user.has_group('hr_holidays.group_hr_holidays_user'):
            domain += [
                '|',
                ('user_id', '=', self.env.uid),
                ('leave_manager_id', '=', self.env.uid),
            ]
        return domain

    @api.depends('holiday_type')
    def _compute_from_holiday_type(self):
        allocation_from_domain = self.env['hr.leave.allocation']
        if (self._context.get('active_model') == 'hr.leave.allocation' and
           self._context.get('active_id')):
            allocation_from_domain = allocation_from_domain.browse(self._context['active_id'])
        for holiday in self:
            if holiday.holiday_type == 'employee':
                if not holiday.employee_ids:
                    if allocation_from_domain:
                        holiday.employee_ids = allocation_from_domain.employee_id
                        holiday.holiday_status_id = allocation_from_domain.holiday_status_id
                    else:
                        # This handles the case where a request is made with only the employee_id
                        # but does not need to be recomputed on employee_id changes
                        holiday.employee_ids = holiday.employee_id or self.env.user.employee_id
                holiday.mode_company_id = False
                holiday.category_id = False
            elif holiday.holiday_type == 'company':
                holiday.employee_ids = False
                if not holiday.mode_company_id:
                    holiday.mode_company_id = self.env.company.id
                holiday.category_id = False
            elif holiday.holiday_type == 'department':
                holiday.employee_ids = False
                holiday.mode_company_id = False
                holiday.category_id = False
            elif holiday.holiday_type == 'category':
                holiday.employee_ids = False
                holiday.mode_company_id = False
            else:
                holiday.employee_ids = self.env.context.get('default_employee_id') or holiday.employee_id or self.env.user.employee_id

    @api.depends('employee_id', 'employee_ids')
    def _compute_from_employee_id(self):
        for holiday in self:
            holiday.manager_id = holiday.employee_id.parent_id.id
            if holiday.holiday_status_id.requires_allocation == 'no':
                continue
            if not holiday.employee_id or len(holiday.employee_ids) > 1:
                holiday.holiday_status_id = False
            elif holiday.employee_id.user_id != self.env.user and holiday._origin.employee_id != holiday.employee_id:
                if holiday.employee_id and not holiday.holiday_status_id.with_context(employee_id=holiday.employee_id.id).has_valid_allocation:
                    holiday.holiday_status_id = False

    @api.depends_context('uid')
    @api.depends('employee_id')
    def _compute_is_user_only_responsible(self):
        user = self.env.user
        self.is_user_only_responsible = user.has_group('hr_holidays.group_hr_holidays_responsible')\
            and not user.has_group('hr_holidays.group_hr_holidays_user')

    @api.depends('employee_id', 'holiday_type')
    def _compute_department_id(self):
        for holiday in self:
            if holiday.employee_id:
                holiday.department_id = holiday.employee_id.department_id
            elif holiday.holiday_type == 'department':
                if not holiday.department_id:
                    holiday.department_id = self.env.user.employee_id.department_id
            else:
                holiday.department_id = False

    @api.depends('date_from', 'date_to', 'holiday_status_id')
    def _compute_has_mandatory_day(self):
        date_from, date_to = min(self.mapped('date_from')), max(self.mapped('date_to'))
        if date_from and date_to:
            mandatory_days = self.employee_id._get_mandatory_days(
                date_from.date(),
                date_to.date())

            for leave in self:
                domain = [
                    ('start_date', '<=', leave.date_to.date()),
                    ('end_date', '>=', leave.date_from.date()),
                    '|',
                        ('resource_calendar_id', '=', False),
                        ('resource_calendar_id', '=', leave.resource_calendar_id.id),
                ]

                if leave.holiday_status_id.company_id:
                    domain += [('company_id', '=', leave.holiday_status_id.company_id.id)]
                leave.has_mandatory_day = leave.date_from and leave.date_to and mandatory_days.filtered_domain(domain)
        else:
            self.has_mandatory_day = False

    @api.depends('leave_type_request_unit', 'number_of_days')
    def _compute_leave_type_increases_duration(self):
        for holiday in self:
            days = holiday._get_duration(check_leave_type=False)[0]
            holiday.leave_type_increases_duration = holiday.leave_type_request_unit == 'day' and days < holiday.number_of_days

    def _get_duration(self, check_leave_type=True, resource_calendar=None):
        """
        This method is factored out into a separate method from
        _compute_duration so it can be hooked and called without necessarily
        modifying the fields and triggering more computes of fields that
        depend on number_of_hours or number_of_days.
        """
        self.ensure_one()
        resource_calendar = resource_calendar or self.resource_calendar_id

        if not self.date_from or not self.date_to or not resource_calendar:
            return (0, 0)
        hours, days = (0, 0)
        if self.employee_id:
            # We force the company in the domain as we are more than likely in a compute_sudo
            domain = [('time_type', '=', 'leave'),
                      ('company_id', 'in', self.env.companies.ids + self.env.context.get('allowed_company_ids', [])),
                      # When searching for resource leave intervals, we exclude the one that
                      # is related to the leave we're currently trying to compute for.
                      '|', ('holiday_id', '=', False), ('holiday_id', '!=', self.id)]
            if self.leave_type_request_unit == 'day' and check_leave_type:
                # list of tuples (day, hours)
                work_time_per_day_list = self.employee_id.list_work_time_per_day(self.date_from, self.date_to, calendar=resource_calendar, domain=domain)
                days = len(work_time_per_day_list)
                hours = sum(map(lambda t: t[1], work_time_per_day_list))
            else:
                work_days_data = self.employee_id._get_work_days_data_batch(self.date_from, self.date_to, domain=domain, calendar=resource_calendar)[self.employee_id.id]
                hours, days = work_days_data['hours'], work_days_data['days']
        else:
            today_hours = resource_calendar.get_work_hours_count(
                datetime.combine(self.date_from.date(), time.min),
                datetime.combine(self.date_from.date(), time.max),
                False)
            hours = resource_calendar.get_work_hours_count(self.date_from, self.date_to)
            days = hours / (today_hours or HOURS_PER_DAY)
        if self.leave_type_request_unit == 'day' and check_leave_type:
            days = ceil(days)
        return (days, hours)


    @api.depends('date_from', 'date_to', 'resource_calendar_id', 'holiday_status_id.request_unit')
    def _compute_duration(self):
        for holiday in self:
            days, hours = holiday._get_duration()
            holiday.number_of_hours = hours
            holiday.number_of_days = days

    @api.depends('employee_company_id', 'mode_company_id')
    def _compute_company_id(self):
        for holiday in self:
            holiday.company_id = holiday.employee_company_id \
                or holiday.mode_company_id \
                or holiday.department_id.company_id \
                or self.env.company

    @api.depends('number_of_days')
    def _compute_last_several_days(self):
        for holiday in self:
            holiday.last_several_days = holiday.number_of_days > 1

    @api.depends('tz')
    @api.depends_context('uid')
    def _compute_tz_mismatch(self):
        for leave in self:
            leave.tz_mismatch = leave.tz != self.env.user.tz

    @api.depends('resource_calendar_id.tz')
    def _compute_tz(self):
        for leave in self:
            leave.tz = leave.resource_calendar_id.tz or self.env.company.resource_calendar_id.tz or self.env.user.tz or 'UTC'

    @api.depends('number_of_days')
    def _compute_number_of_days_display(self):
        for holiday in self:
            holiday.number_of_days_display = holiday.number_of_days

    @api.depends('number_of_hours')
    def _compute_number_of_hours_display(self):
        for leave in self:
            leave.number_of_hours_display = leave.number_of_hours

    @api.depends('number_of_hours_display', 'number_of_days_display')
    def _compute_duration_display(self):
        for leave in self:
            leave.duration_display = '%g %s' % (
                (float_round(leave.number_of_hours_display, precision_digits=2)
                if leave.leave_type_request_unit == 'hour'
                else float_round(leave.number_of_days_display, precision_digits=2)),
                _('hours') if leave.leave_type_request_unit == 'hour' else _('days'))

    @api.depends('number_of_hours_display')
    def _compute_number_of_hours_text(self):
        # YTI Note: All this because a readonly field takes all the width on edit mode...
        for leave in self:
            leave.number_of_hours_text = '%s%g %s%s' % (
                '' if leave.request_unit_half or leave.request_unit_hours else '(',
                float_round(leave.number_of_hours_display, precision_digits=2),
                _('Hours'),
                '' if leave.request_unit_half or leave.request_unit_hours else ')')

    @api.depends('state', 'employee_id', 'department_id')
    def _compute_can_reset(self):
        for holiday in self:
            try:
                holiday._check_approval_update('draft')
            except (AccessError, UserError):
                holiday.can_reset = False
            else:
                holiday.can_reset = True

    @api.depends('state', 'employee_id', 'department_id')
    def _compute_can_approve(self):
        for holiday in self:
            try:
                if holiday.state == 'confirm' and holiday.validation_type == 'both':
                    holiday._check_approval_update('validate1')
                else:
                    holiday._check_approval_update('validate')
            except (AccessError, UserError):
                holiday.can_approve = False
            else:
                holiday.can_approve = True

    @api.depends_context('uid')
    @api.depends('state', 'employee_id')
    def _compute_can_cancel(self):
        now = fields.Datetime.now().date()
        for leave in self:
            leave.can_cancel = leave.id and leave.employee_id.user_id == self.env.user and leave.state in ['validate', 'validate1'] and leave.date_from and leave.date_from.date() >= now

    @api.depends('state')
    def _compute_is_hatched(self):
        for holiday in self:
            holiday.is_striked = holiday.state == 'refuse'
            holiday.is_hatched = holiday.state not in ['refuse', 'validate']

    @api.depends('leave_type_support_document', 'attachment_ids')
    def _compute_supported_attachment_ids(self):
        for holiday in self:
            holiday.supported_attachment_ids = holiday.attachment_ids
            holiday.supported_attachment_ids_count = len(holiday.attachment_ids.ids)

    def _inverse_supported_attachment_ids(self):
        for holiday in self:
            holiday.attachment_ids = holiday.supported_attachment_ids

    @api.constrains('date_from', 'date_to', 'employee_id')
    def _check_date(self):
        if self.env.context.get('leave_skip_date_check', False):
            return

        all_employees = self.all_employee_ids
        all_leaves = self.search([
            ('date_from', '<', max(self.mapped('date_to'))),
            ('date_to', '>', min(self.mapped('date_from'))),
            ('employee_id', 'in', all_employees.ids),
            ('id', 'not in', self.ids),
            ('state', 'not in', ['cancel', 'refuse']),
        ])
        for holiday in self:
            domain = [
                ('date_from', '<', holiday.date_to),
                ('date_to', '>', holiday.date_from),
                ('id', '!=', holiday.id),
                ('state', 'not in', ['cancel', 'refuse']),
            ]

            employee_ids = (holiday.employee_id | holiday.employee_ids).ids
            search_domain = domain + [('employee_id', 'in', employee_ids)]
            conflicting_holidays = all_leaves.filtered_domain(search_domain)

            if conflicting_holidays:
                conflicting_holidays_list = []
                # Do not display the name of the employee if the conflicting holidays have an employee_id.user_id equivalent to the user id
                holidays_only_have_uid = bool(holiday.employee_id)
                holiday_states = dict(conflicting_holidays.fields_get(allfields=['state'])['state']['selection'])
                for conflicting_holiday in conflicting_holidays:
                    conflicting_holiday_data = {}
                    conflicting_holiday_data['employee_name'] = conflicting_holiday.employee_id.name
                    conflicting_holiday_data['date_from'] = format_date(self.env, min(conflicting_holiday.mapped('date_from')))
                    conflicting_holiday_data['date_to'] = format_date(self.env, min(conflicting_holiday.mapped('date_to')))
                    conflicting_holiday_data['state'] = holiday_states[conflicting_holiday.state]
                    if conflicting_holiday.employee_id.user_id.id != self.env.uid:
                        holidays_only_have_uid = False
                    if conflicting_holiday_data not in conflicting_holidays_list:
                        conflicting_holidays_list.append(conflicting_holiday_data)
                if not conflicting_holidays_list:
                    return
                conflicting_holidays_strings = []
                if holidays_only_have_uid:
                    for conflicting_holiday_data in conflicting_holidays_list:
                        conflicting_holidays_string = _('from %(date_from)s to %(date_to)s - %(state)s',
                                                        date_from=conflicting_holiday_data['date_from'],
                                                        date_to=conflicting_holiday_data['date_to'],
                                                        state=conflicting_holiday_data['state'])
                        conflicting_holidays_strings.append(conflicting_holidays_string)
                    raise ValidationError(_("""\
You've already booked time off which overlaps with this period:
%s
Attempting to double-book your time off won't magically make your vacation 2x better!
""",
                        "\n".join(conflicting_holidays_strings)))
                for conflicting_holiday_data in conflicting_holidays_list:
                    conflicting_holidays_string = "\n" + _('%(employee_name)s - from %(date_from)s to %(date_to)s - %(state)s',
                                                    employee_name=conflicting_holiday_data['employee_name'],
                                                    date_from=conflicting_holiday_data['date_from'],
                                                    date_to=conflicting_holiday_data['date_to'],
                                                    state=conflicting_holiday_data['state'])
                    conflicting_holidays_strings.append(conflicting_holidays_string)
                raise ValidationError(_(
                    "An employee already booked time off which overlaps with this period:%s",
                    "".join(conflicting_holidays_strings)))

    @api.constrains('date_from', 'date_to', 'employee_id')
    def _check_date_state(self):
        if self.env.context.get('leave_skip_state_check'):
            return
        for holiday in self:
            if holiday.state in ['cancel', 'refuse', 'validate1', 'validate']:
                raise ValidationError(_("This modification is not allowed in the current state."))

    def _check_validity(self):
        sorted_leaves = defaultdict(lambda: self.env['hr.leave'])
        for leave in self:
            sorted_leaves[(leave.holiday_status_id, leave.date_from.date())] |= leave
        for (leave_type, date_from), leaves in sorted_leaves.items():
            if leave_type.requires_allocation == 'no':
                continue
            employees = self.env['hr.employee']
            for leave in leaves:
                employees |= leave._get_employees_from_holiday_type()
            leave_data = leave_type.get_allocation_data(employees, date_from)
            if leave_type.allows_negative:
                max_excess = leave_type.max_allowed_negative
                for employee in employees:
                    if leave_data[employee] and leave_data[employee][0][1]['virtual_remaining_leaves'] < -max_excess:
                        raise ValidationError(_("There is no valid allocation to cover that request."))
                continue

            previous_leave_data = leave_type.with_context(
                ignored_leave_ids=leaves.ids
            ).get_allocation_data(employees, date_from)
            for employee in employees:
                previous_emp_data = previous_leave_data[employee] and previous_leave_data[employee][0][1]['virtual_excess_data']
                emp_data = leave_data[employee] and leave_data[employee][0][1]['virtual_excess_data']
                if not previous_emp_data and not emp_data:
                    continue
                if previous_emp_data != emp_data and len(emp_data) >= len(previous_emp_data):
                    raise ValidationError(_("There is no valid allocation to cover that request."))

    ####################################################
    # ORM Overrides methods
    ####################################################

    @api.depends(
        'tz', 'date_from', 'date_to', 'employee_id',
        'holiday_status_id', 'number_of_hours_display',
        'leave_type_request_unit', 'number_of_days', 'mode_company_id',
        'category_id', 'department_id',
    )
    @api.depends_context('short_name', 'hide_employee_name', 'groupby')
    def _compute_display_name(self):
        for leave in self:
            user_tz = timezone(leave.tz)
            date_from_utc = leave.date_from and leave.date_from.astimezone(user_tz).date()
            date_to_utc = leave.date_to and leave.date_to.astimezone(user_tz).date()
            time_off_type_display = leave.holiday_status_id.name
            if self.env.context.get('short_name'):
                short_leave_name = leave.name or time_off_type_display or _('Time Off')
                if leave.leave_type_request_unit == 'hour':
                    leave.display_name = _("%s: %.2f hours", short_leave_name, leave.number_of_hours_display)
                else:
                    leave.display_name = _("%s: %.2f days", short_leave_name, leave.number_of_days)
            else:
                if leave.holiday_type == 'company':
                    target = leave.mode_company_id.name
                elif leave.holiday_type == 'department':
                    target = leave.department_id.name
                elif leave.holiday_type == 'category':
                    target = leave.category_id.name
                elif leave.employee_id:
                    target = leave.employee_id.name
                else:
                    target = ', '.join(leave.employee_ids.mapped('name'))
                display_date = format_date(self.env, date_from_utc) or ""
                if leave.leave_type_request_unit == 'hour':
                    if self.env.context.get('hide_employee_name') and 'employee_id' in self.env.context.get('group_by', []):
                        leave.display_name = _("%(leave_type)s: %(duration).2f hours on %(date)s",
                            leave_type=time_off_type_display,
                            duration=leave.number_of_hours_display,
                            date=display_date,
                        )
                    elif not time_off_type_display:
                        leave.display_name = _("%(person)s: %(duration).2f hours on %(date)s",
                            person=target,
                            duration=leave.number_of_hours_display,
                            date=display_date,
                        )
                    else:
                        leave.display_name = _("%(person)s on %(leave_type)s: %(duration).2f hours on %(date)s",
                            person=target,
                            leave_type=time_off_type_display,
                            duration=leave.number_of_hours_display,
                            date=display_date,
                        )
                else:
                    if leave.number_of_days > 1 and date_from_utc and date_to_utc:
                        display_date += ' - %s' % format_date(self.env, date_to_utc) or ""
                    if not target or self.env.context.get('hide_employee_name') and 'employee_id' in self.env.context.get('group_by', []):
                        leave.display_name = _("%(leave_type)s: %(duration).2f days (%(start)s)",
                            leave_type=time_off_type_display,
                            duration=leave.number_of_days,
                            start=display_date,
                        )
                    elif not time_off_type_display:
                        leave.display_name = _("%(person)s: %(duration).2f days (%(start)s)",
                            person=target,
                            duration=leave.number_of_days,
                            start=display_date,
                        )
                    else:
                        leave.display_name = _("%(person)s on %(leave_type)s: %(duration).2f days (%(start)s)",
                            person=target,
                            leave_type=time_off_type_display,
                            duration=leave.number_of_days,
                            start=display_date,
                        )

    def onchange(self, values, field_names, fields_spec):
        # Try to force the leave_type display_name when creating new records
        # This is called right after pressing create and returns the display_name for
        # most fields in the view.
        if values and 'employee_id' in fields_spec and 'employee_id' not in self._context:
            employee_id = get_employee_from_context(values, self._context, self.env.user.employee_id.id)
            self = self.with_context(employee_id=employee_id)
        return super().onchange(values, field_names, fields_spec)

    def add_follower(self, employee_id):
        employee = self.env['hr.employee'].browse(employee_id)
        if employee.user_id:
            self.message_subscribe(partner_ids=employee.user_id.partner_id.ids)

    @api.constrains('date_from', 'date_to')
    def _check_mandatory_day(self):
        is_leave_user = self.user_has_groups('hr_holidays.group_hr_holidays_user')
        if not is_leave_user and any(leave.has_mandatory_day for leave in self):
            raise ValidationError(_('You are not allowed to request a time off on a Mandatory Day.'))

    def _check_double_validation_rules(self, employees, state):
        if self.user_has_groups('hr_holidays.group_hr_holidays_manager'):
            return

        is_leave_user = self.user_has_groups('hr_holidays.group_hr_holidays_user')
        if state == 'validate1':
            employees = employees.filtered(lambda employee: employee.leave_manager_id != self.env.user)
            if employees and not is_leave_user:
                raise AccessError(_('You cannot first approve a time off for %s, because you are not his time off manager', employees[0].name))
        elif state == 'validate' and not is_leave_user:
            # Is probably handled via ir.rule
            raise AccessError(_('You don\'t have the rights to apply second approval on a time off request'))

    @api.model_create_multi
    def create(self, vals_list):
        employee_ids = []
        for values in vals_list:
            if values.get('employee_id'):
                employee_ids.append(values['employee_id'])
        employees = self.env['hr.employee'].browse(employee_ids)

        """ Override to avoid automatic logging of creation """
        if not self._context.get('leave_fast_create'):
            leave_types = self.env['hr.leave.type'].browse([values.get('holiday_status_id') for values in vals_list if values.get('holiday_status_id')])
            mapped_validation_type = {leave_type.id: leave_type.leave_validation_type for leave_type in leave_types}

            for values in vals_list:
                employee_id = values.get('employee_id', False)
                leave_type_id = values.get('holiday_status_id')
                # Handle automatic department_id
                if not values.get('department_id'):
                    values.update({'department_id': employees.filtered(lambda emp: emp.id == employee_id).department_id.id})

                # Handle no_validation
                if mapped_validation_type[leave_type_id] == 'no_validation':
                    values.update({'state': 'confirm'})

                if 'state' not in values:
                    # To mimic the behavior of compute_state that was always triggered, as the field was readonly
                    values['state'] = 'confirm' if mapped_validation_type[leave_type_id] != 'no_validation' else 'draft'

                # Handle double validation
                if mapped_validation_type[leave_type_id] == 'both':
                    self._check_double_validation_rules(employee_id, values.get('state', False))

        holidays = super(HolidaysRequest, self.with_context(mail_create_nosubscribe=True)).create(vals_list)
        holidays._check_validity()

        for holiday in holidays:
            if not self._context.get('leave_fast_create'):
                # Everything that is done here must be done using sudo because we might
                # have different create and write rights
                # eg : holidays_user can create a leave request with validation_type = 'manager' for someone else
                # but they can only write on it if they are leave_manager_id
                holiday_sudo = holiday.sudo()
                holiday_sudo.add_follower(holiday.employee_id.id)
                if holiday.validation_type == 'manager':
                    holiday_sudo.message_subscribe(partner_ids=holiday.employee_id.leave_manager_id.partner_id.ids)
                if holiday.validation_type == 'no_validation':
                    # Automatic validation should be done in sudo, because user might not have the rights to do it by himself
                    holiday_sudo.action_validate()
                    holiday_sudo.message_subscribe(partner_ids=holiday._get_responsible_for_approval().partner_id.ids)
                    holiday_sudo.message_post(body=_("The time off has been automatically approved"), subtype_xmlid="mail.mt_comment") # Message from OdooBot (sudo)
                elif not self._context.get('import_file'):
                    holiday_sudo.activity_update()
        return holidays

    def write(self, values):
        if 'active' in values and not self.env.context.get('from_cancel_wizard'):
            raise UserError(_("You can't manually archive/unarchive a time off."))

        is_officer = self.env.user.has_group('hr_holidays.group_hr_holidays_user') or self.env.is_superuser()
        if not is_officer and values.keys() - {'attachment_ids', 'supported_attachment_ids', 'message_main_attachment_id'}:
            if any(hol.date_from.date() < fields.Date.today() and hol.employee_id.leave_manager_id != self.env.user
                   and hol.state not in ('confirm', 'draft') for hol in self):
                raise UserError(_('You must have manager rights to modify/validate a time off that already begun'))

        # Unlink existing resource.calendar.leaves for validated time off
        if 'state' in values and values['state'] != 'validate':
            validated_leaves = self.filtered(lambda l: l.state == 'validate')
            validated_leaves._remove_resource_leave()

        employee_id = values.get('employee_id', False)
        if not self.env.context.get('leave_fast_create'):
            if values.get('state'):
                self._check_approval_update(values['state'])
                if any(holiday.validation_type == 'both' for holiday in self):
                    if values.get('employee_id'):
                        employees = self.env['hr.employee'].browse(values.get('employee_id'))
                    else:
                        employees = self.mapped('employee_id')
                    self._check_double_validation_rules(employees, values['state'])
            if 'date_from' in values:
                values['request_date_from'] = values['date_from']
            if 'date_to' in values:
                values['request_date_to'] = values['date_to']
        result = super(HolidaysRequest, self).write(values)
        if any(field in values for field in ['request_date_from', 'date_from', 'request_date_from', 'date_to', 'holiday_status_id', 'employee_id', 'state']):
            self._check_validity()
        if not self.env.context.get('leave_fast_create'):
            for holiday in self:
                if employee_id:
                    holiday.add_follower(employee_id)

        return result

    @api.ondelete(at_uninstall=False)
    def _unlink_if_correct_states(self):
        error_message = _('You cannot delete a time off which is in %s state')
        state_description_values = {elem[0]: elem[1] for elem in self._fields['state']._description_selection(self.env)}
        now = fields.Datetime.now().date()

        if not self.user_has_groups('hr_holidays.group_hr_holidays_user'):
            for hol in self:
                if hol.state not in ['draft', 'confirm', 'validate1']:
                    raise UserError(error_message % state_description_values.get(self[:1].state))
                if hol.date_from.date() < now:
                    raise UserError(_('You cannot delete a time off which is in the past'))
                if hol.sudo().employee_ids and not hol.employee_id:
                    raise UserError(_('You cannot delete a time off assigned to several employees'))
        else:
            for holiday in self.filtered(lambda holiday: holiday.state not in ['draft', 'cancel', 'confirm']):
                raise UserError(error_message % (state_description_values.get(holiday.state),))

    def unlink(self):
        self._force_cancel(_("deleted by %s (uid=%d).",
            self.env.user.display_name, self.env.user.id
        ))
        return super(HolidaysRequest, self.with_context(leave_skip_date_check=True)).unlink()

    def copy_data(self, default=None):
        if default and 'request_date_from' in default and 'request_date_to' in default:
            return super().copy_data(default)
        elif self.state in {"cancel", "refuse"}:  # No overlap constraint in these cases
            return super().copy_data(default)
        raise UserError(_('A time off cannot be duplicated.'))

    def _get_mail_redirect_suggested_company(self):
        return self.holiday_status_id.company_id

    ####################################################
    # Business methods
    ####################################################

    @api.model
    def action_open_records(self, leave_ids):
        if len(leave_ids) == 1:
            return {
                'type': 'ir.actions.act_window',
                'view_mode': 'form',
                'res_id': leave_ids[0],
                'res_model': 'hr.leave',
            }
        return {
            'type': 'ir.actions.act_window',
            'view_mode': [[False, 'tree'], [False, 'form']],
            'domain': [('id', 'in', leave_ids.ids)],
            'res_model': 'hr.leave',
        }

    def _prepare_resource_leave_vals(self):
        """Hook method for others to inject data
        """
        self.ensure_one()
        return {
            'name': _("%s: Time Off", self.employee_id.name),
            'date_from': self.date_from,
            'holiday_id': self.id,
            'date_to': self.date_to,
            'resource_id': self.employee_id.resource_id.id,
            'calendar_id': self.resource_calendar_id.id,
            'time_type': self.holiday_status_id.time_type,
        }

    def _create_resource_leave(self):
        """ This method will create entry in resource calendar time off object at the time of holidays validated
        :returns: created `resource.calendar.leaves`
        """
        vals_list = [leave._prepare_resource_leave_vals() for leave in self]
        return self.env['resource.calendar.leaves'].sudo().create(vals_list)

    def _remove_resource_leave(self):
        """ This method will create entry in resource calendar time off object at the time of holidays cancel/removed """
        return self.env['resource.calendar.leaves'].search([('holiday_id', 'in', self.ids)]).unlink()

    def _validate_leave_request(self):
        """ Validate time off requests (holiday_type='employee')
        by creating a calendar event and a resource time off. """
        holidays = self.filtered(lambda request: request.holiday_type == 'employee' and request.employee_id)
        holidays._create_resource_leave()
        meeting_holidays = holidays.filtered(lambda l: l.holiday_status_id.create_calendar_meeting)
        meetings = self.env['calendar.event']
        if meeting_holidays:
            meeting_values_for_user_id = meeting_holidays._prepare_holidays_meeting_values()
            Meeting = self.env['calendar.event']
            for user_id, meeting_values in meeting_values_for_user_id.items():
                meetings += Meeting.with_user(user_id or self.env.uid).with_context(
                                allowed_company_ids=[],
                                no_mail_to_attendees=True,
                                calendar_no_videocall=True,
                                active_model=self._name
                            ).create(meeting_values)
        Holiday = self.env['hr.leave']
        for meeting in meetings:
            Holiday.browse(meeting.res_id).meeting_id = meeting

        for holiday in holidays:
            user_tz = timezone(holiday.tz)
            utc_tz = pytz.utc.localize(holiday.date_from).astimezone(user_tz)
            notify_partner_ids = holiday.employee_id.user_id.partner_id.ids
            holiday.message_post(
                body=_(
                    'Your %(leave_type)s planned on %(date)s has been accepted',
                    leave_type=holiday.holiday_status_id.display_name,
                    date=utc_tz.replace(tzinfo=None)
                ),
                partner_ids=notify_partner_ids)


    def _prepare_holidays_meeting_values(self):
        result = defaultdict(list)
        for holiday in self:
            user = holiday.user_id
            if holiday.leave_type_request_unit == 'hour':
                meeting_name = _("%s on Time Off : %.2f hour(s)") % (holiday.employee_id.name or holiday.category_id.name, holiday.number_of_hours_display)
                allday_value = float_compare(holiday.number_of_days, 1.0, 1) >= 0
            else:
                meeting_name = _("%s on Time Off : %.2f day(s)") % (holiday.employee_id.name or holiday.category_id.name, holiday.number_of_days)
                allday_value = not holiday.request_unit_half
            meeting_values = {
                'name': meeting_name,
                'duration': holiday.number_of_days * (holiday.resource_calendar_id.hours_per_day or HOURS_PER_DAY),
                'description': holiday.notes,
                'user_id': user.id,
                'start': holiday.date_from,
                'stop': holiday.date_to,
                'allday': allday_value,
                'privacy': 'confidential',
                'event_tz': user.tz,
                'activity_ids': [(5, 0, 0)],
                'res_id': holiday.id,
            }
            # Add the partner_id (if exist) as an attendee
            partner_id = (user and user.partner_id) or (holiday.employee_id and holiday.employee_id.work_contact_id)
            if partner_id:
                meeting_values['partner_ids'] = [(4, partner_id.id)]
            result[user.id].append(meeting_values)
        return result

    def _prepare_employees_holiday_values(self, employees):
        self.ensure_one()
        work_days_data = employees._get_work_days_data_batch(self.date_from, self.date_to)
        return [{
            'name': self.name,
            'holiday_type': 'employee',
            'holiday_status_id': self.holiday_status_id.id,
            'date_from': self.date_from,
            'date_to': self.date_to,
            'request_date_from': self.request_date_from,
            'request_date_to': self.request_date_to,
            'request_hour_from': self.request_hour_from,
            'request_hour_to': self.request_hour_to,
            'request_unit_hours': self.request_unit_hours,
            'request_unit_half': self.request_unit_half,
            'notes': self.notes,
            'number_of_days': work_days_data[employee.id]['days'],
            'number_of_hours': work_days_data[employee.id]['hours'],
            'parent_id': self.id,
            'employee_id': employee.id,
            'employee_ids': employee,
            'state': 'validate',
        } for employee in employees if work_days_data[employee.id]['days']]

    def action_cancel(self):
        self.ensure_one()

        return {
            'name': _('Cancel Time Off'),
            'type': 'ir.actions.act_window',
            'target': 'new',
            'res_model': 'hr.holidays.cancel.leave',
            'view_mode': 'form',
            'views': [[False, 'form']],
            'context': {
                'default_leave_id': self.id,
            }
        }

    def action_draft(self):
        if any(holiday.state not in ['confirm', 'refuse'] for holiday in self):
            raise UserError(_('Time off request state must be "Refused" or "To Approve" in order to be reset to draft.'))
        self.write({
            'state': 'draft',
            'first_approver_id': False,
            'second_approver_id': False,
        })
        linked_requests = self.mapped('linked_request_ids')
        if linked_requests:
            linked_requests.action_draft()
            linked_requests.unlink()
        self.activity_update()
        return True

    def action_confirm(self):
        if self.filtered(lambda holiday: holiday.state != 'draft'):
            raise UserError(_('Time off request must be in Draft state ("To Submit") in order to confirm it.'))
        self.write({'state': 'confirm'})
        holidays = self.filtered(lambda leave: leave.validation_type == 'no_validation')
        if holidays:
            # Automatic validation should be done in sudo, because user might not have the rights to do it by himself
            holidays.sudo().action_validate()
        self.activity_update()
        return True

    def action_approve(self, check_state=True):
        # if validation_type == 'both': this method is the first approval approval
        # if validation_type != 'both': this method calls action_validate() below

        # Do not check the state in case we are redirected from the dashboard
        if check_state and any(holiday.state != 'confirm' for holiday in self):
            raise UserError(_('Time off request must be confirmed ("To Approve") in order to approve it.'))

        current_employee = self.env.user.employee_id
        self.filtered(lambda hol: hol.validation_type == 'both').write({'state': 'validate1', 'first_approver_id': current_employee.id})

        self.filtered(lambda hol: not hol.validation_type == 'both').action_validate()
        if not self.env.context.get('leave_fast_create'):
            self.activity_update()
        return True

    def _get_leaves_on_public_holiday(self):
        return self.filtered(lambda l: l.employee_id and not l.number_of_days)

    def _get_employees_from_holiday_type(self):
        self.ensure_one()
        if self.holiday_type == 'employee':
            employees = self.employee_ids
        elif self.holiday_type == 'category':
            employees = self.category_id.employee_ids
        elif self.holiday_type == 'company':
            employees = self.env['hr.employee'].search([('company_id', '=', self.mode_company_id.id)])
        else:
            employees = self.department_id.member_ids
        return employees

    def _split_leaves(self, split_date_from, split_date_to):
        """
        Split leaves on the given full-day interval. The leaves will be split
        into two new leaves: the period up until (but not including)
        split_date_from and the period starting at (and including)
        split_date_to.

        This means that the period in between split_date_from and split_date_to
        will no longer be covered by the new leaves. In order to split a leave
        without losing any leave coverage, split_date_from and split_date_to
        should therefore be the same.

        Another important note to make is that this method only splits leaves
        on full-day intervals. Logic to split leaves on partial days or hours
        is not straightforward as you have to take into account working hours
        and timezones. It's also not clear that we would want to handle this
        automatically. The method will therefore also only work on leaves that
        are taken in full or half days (Though a half day leave in the interval
        will simply be refused - there are no multi-day spanning half-day
        leaves)

        The method creates one or two new leaves per leave that needs to be
        split and refuses the original leave.
        """
        # Keep track of the original states before refusing the leaves and creating new ones
        original_states = {l.id: l.state for l in self}

        # Refuse all original leaves
        self.action_refuse()
        split_leaves_vals = []

        # Only leaves that span a period outside of the split interval need
        # to be split.
        multi_day_leaves = self.filtered(lambda l: l.request_date_from < split_date_from or l.request_date_to >= split_date_to)

        for leave in multi_day_leaves:
            # Leaves in days
            new_leave_vals = []

            # Get the values to create the leave before the split
            if leave.request_date_from < split_date_from:
                new_leave_vals.append(leave.copy_data({
                    'request_date_from': leave.request_date_from,
                    'request_date_to': split_date_from + timedelta(days=-1),
                    'state': original_states[leave.id],
                })[0])

            # Do the same for the new leave after the split
            if leave.request_date_to >= split_date_to:
                new_leave_vals.append(leave.copy_data({
                    'request_date_from': split_date_to,
                    'request_date_to': leave.request_date_to,
                    'state': original_states[leave.id],
                })[0])

            # For those two new leaves, only create them if they actually
            # have a non-zero duration.
            for leave_vals in new_leave_vals:
                new_leave = self.env['hr.leave'].new(leave_vals)
                new_leave._compute_date_from_to()
                # Could happen for part-time contract, that time off is not necessary
                # anymore.
                # Imagine you work on monday-wednesday-friday only.
                # You take a time off on friday.
                # We create a company time off on friday.
                # By looking at the last attendance before the company time off
                # start date to compute the date_to, you would have a date_from > date_to.
                # Just don't create the leave at that time. That's the reason why we use
                # new instead of create. As the leave is not actually created yet, the sql
                # constraint didn't check date_from < date_to yet.
                if new_leave.date_from < new_leave.date_to:
                    split_leaves_vals.append(new_leave._convert_to_write(new_leave._cache))

        split_leaves = self.env['hr.leave'].with_context(
            tracking_disable=True,
            mail_activity_automation_skip=True,
            leave_fast_create=True,
            leave_skip_state_check=True
        ).create(split_leaves_vals)

        split_leaves.filtered(lambda l: l.state in 'validate')._validate_leave_request()

        return split_leaves

    def action_validate(self):
        current_employee = self.env.user.employee_id
        leaves = self._get_leaves_on_public_holiday()
        if leaves:
            raise ValidationError(_('The following employees are not supposed to work during that period:\n %s') % ','.join(leaves.mapped('employee_id.name')))

        if any(holiday.state not in ['confirm', 'validate1'] and holiday.validation_type != 'no_validation' for holiday in self):
            raise UserError(_('Time off request must be confirmed in order to approve it.'))

        self.write({'state': 'validate'})

        leaves_second_approver = self.env['hr.leave']
        leaves_first_approver = self.env['hr.leave']

        for leave in self:
            if leave.validation_type == 'both':
                leaves_second_approver += leave
            else:
                leaves_first_approver += leave

            if leave.holiday_type != 'employee' or\
                (leave.holiday_type == 'employee' and len(leave.employee_ids) > 1):
                employees = leave._get_employees_from_holiday_type()

                conflicting_leaves = self.env['hr.leave'].with_context(
                    tracking_disable=True,
                    mail_activity_automation_skip=True,
                    leave_fast_create=True
                ).search([
                    ('date_from', '<=', leave.date_to),
                    ('date_to', '>', leave.date_from),
                    ('state', 'not in', ['cancel', 'refuse']),
                    ('holiday_type', '=', 'employee'),
                    ('employee_id', 'in', employees.ids)])

                if conflicting_leaves:
                    # YTI: More complex use cases could be managed in master
                    if leave.leave_type_request_unit != 'day' or any(l.leave_type_request_unit == 'hour' for l in conflicting_leaves):
                        raise ValidationError(_('You can not have 2 time off that overlaps on the same day.'))

                    conflicting_leaves._split_leaves(leave.request_date_from, leave.request_date_to + timedelta(days=1))

                values = leave._prepare_employees_holiday_values(employees)
                leaves = self.env['hr.leave'].with_context(
                    tracking_disable=True,
                    mail_activity_automation_skip=True,
                    leave_fast_create=True,
                    no_calendar_sync=True,
                    leave_skip_state_check=True,
                    # date_from and date_to are computed based on the employee tz
                    # If _compute_date_from_to is used instead, it will trigger _compute_number_of_days
                    # and create a conflict on the number of days calculation between the different leaves
                    leave_compute_date_from_to=True,
                ).create(values)

                leaves._validate_leave_request()

        leaves_second_approver.write({'second_approver_id': current_employee.id})
        leaves_first_approver.write({'first_approver_id': current_employee.id})

        employee_requests = self.filtered(lambda hol: hol.holiday_type == 'employee')
        employee_requests._validate_leave_request()
        if not self.env.context.get('leave_fast_create'):
            employee_requests.filtered(lambda holiday: holiday.validation_type != 'no_validation').activity_update()
        return True

    def action_refuse(self):
        current_employee = self.env.user.employee_id
        if any(holiday.state not in ['draft', 'confirm', 'validate', 'validate1'] for holiday in self):
            raise UserError(_('Time off request must be confirmed or validated in order to refuse it.'))

        self._notify_manager()
        validated_holidays = self.filtered(lambda hol: hol.state == 'validate1')
        validated_holidays.write({'state': 'refuse', 'first_approver_id': current_employee.id})
        (self - validated_holidays).write({'state': 'refuse', 'second_approver_id': current_employee.id})
        # Delete the meeting
        self.mapped('meeting_id').write({'active': False})
        # If a category that created several holidays, cancel all related
        linked_requests = self.mapped('linked_request_ids')
        if linked_requests:
            linked_requests.action_refuse()

        # Post a second message, more verbose than the tracking message
        for holiday in self:
            if holiday.employee_id.user_id:
                holiday.message_post(
                    body=_('Your %(leave_type)s planned on %(date)s has been refused', leave_type=holiday.holiday_status_id.display_name, date=holiday.date_from),
                    partner_ids=holiday.employee_id.user_id.partner_id.ids)

        self.activity_update()
        return True

    def _notify_manager(self):
        leaves = self.filtered(lambda hol: (hol.validation_type == 'both' and hol.state in ['validate1', 'validate']) or (hol.validation_type == 'manager' and hol.state == 'validate'))
        for holiday in leaves:
            responsible = holiday.employee_id.leave_manager_id.partner_id.ids
            if responsible:
                self.env['mail.thread'].sudo().message_notify(
                    partner_ids=responsible,
                    model_description='Time Off',
                    subject=_('Refused Time Off'),
                    body=_(
                        '%(holiday_name)s has been refused.',
                        holiday_name=holiday.display_name,
                    ),
                    email_layout_xmlid='mail.mail_notification_light',
                )

    def _action_user_cancel(self, reason):
        self.ensure_one()
        if not self.can_cancel:
            raise ValidationError(_('This time off cannot be canceled.'))

        self._force_cancel(reason, 'mail.mt_note')

    def _force_cancel(self, reason, msg_subtype='mail.mt_comment'):
        recs = self.browse() if self.env.context.get(MODULE_UNINSTALL_FLAG) else self
        for leave in recs:
            leave.message_post(
                body=_('The time off has been canceled: %s', reason),
                subtype_xmlid=msg_subtype
            )

            responsibles = self.env['res.partner']
            # manager
            if (leave.holiday_status_id.leave_validation_type == 'manager' and leave.state == 'validate') or (leave.holiday_status_id.leave_validation_type == 'both' and leave.state == 'validate1'):
                responsibles = leave.employee_id.leave_manager_id.partner_id
            # officer
            elif leave.holiday_status_id.leave_validation_type == 'hr' and leave.state == 'validate':
                responsibles = leave.holiday_status_id.responsible_ids.partner_id
            # both
            elif leave.holiday_status_id.leave_validation_type == 'both' and leave.state == 'validate':
                responsibles = leave.employee_id.leave_manager_id.partner_id
                responsibles |= leave.holiday_status_id.responsible_ids.partner_id

            if responsibles:
                self.env['mail.thread'].sudo().message_notify(
                    partner_ids=responsibles.ids,
                    model_description='Time Off',
                    subject=_('Canceled Time Off'),
                    body=_(
                        "%(leave_name)s has been cancelled with the justification: <br/> %(reason)s.",
                        leave_name=leave.display_name,
                        reason=reason
                    ),
                    email_layout_xmlid='mail.mail_notification_light',
                )
        leave_sudo = self.sudo()
        leave_sudo.with_context(from_cancel_wizard=True).active = False
        leave_sudo.meeting_id.active = False
        leave_sudo._remove_resource_leave()

    def action_documents(self):
        domain = [('id', 'in', self.attachment_ids.ids)]
        return {
            'name': _("Supporting Documents"),
            'type': 'ir.actions.act_window',
            'res_model': 'ir.attachment',
            'context': {'create': False},
            'view_mode': 'kanban',
            'domain': domain
        }

    def _check_approval_update(self, state):
        """ Check if target state is achievable. """
        if self.env.is_superuser():
            return

        current_employee = self.env.user.employee_id
        is_officer = self.env.user.has_group('hr_holidays.group_hr_holidays_user')
        is_manager = self.env.user.has_group('hr_holidays.group_hr_holidays_manager')

        for holiday in self:
            val_type = holiday.validation_type

            if not is_manager and state != 'confirm':
                if state == 'draft':
                    if holiday.state == 'refuse':
                        raise UserError(_('Only a Time Off Manager can reset a refused leave.'))
                    if holiday.date_from and holiday.date_from.date() <= fields.Date.today():
                        raise UserError(_('Only a Time Off Manager can reset a started leave.'))
                    if holiday.employee_id != current_employee:
                        raise UserError(_('Only a Time Off Manager can reset other people leaves.'))
                else:
                    if val_type == 'no_validation' and current_employee == holiday.employee_id and (is_officer or is_manager):
                        continue
                    # use ir.rule based first access check: department, members, ... (see security.xml)
                    holiday.check_access_rule('write')

                    # This handles states validate1 validate and refuse
                    if holiday.employee_id == current_employee\
                            and self.env.user != holiday.employee_id.leave_manager_id\
                            and not is_officer:
                        raise UserError(_('Only a Time Off Officer or Manager can approve/refuse its own requests.'))

                    if (state == 'validate1' and val_type == 'both') and holiday.holiday_type == 'employee':
                        if not is_officer and self.env.user != holiday.employee_id.leave_manager_id:
                            raise UserError(_('You must be either %s\'s manager or Time off Manager to approve this leave') % (holiday.employee_id.name))

                    if (state == 'validate' and val_type == 'manager')\
                            and self.env.user != (holiday.employee_id | holiday.sudo().employee_ids).leave_manager_id\
                            and not is_officer:
                        if holiday.employee_id:
                            employees = holiday.employee_id
                        else:
                            employees = ', '.join(holiday.employee_ids.filtered(lambda e: e.leave_manager_id != self.env.user).mapped('name'))
                        raise UserError(_('You must be %s\'s Manager to approve this leave', employees))

                    if not is_officer and (state == 'validate' and val_type == 'hr') and holiday.holiday_type == 'employee':
                        raise UserError(_('You must either be a Time off Officer or Time off Manager to approve this leave'))

    # ------------------------------------------------------------
    # Activity methods
    # ------------------------------------------------------------

    def _get_responsible_for_approval(self):
        self.ensure_one()

        responsible = self.env['res.users']

        if self.holiday_type != 'employee':
            return responsible

        if self.validation_type == 'manager' or (self.validation_type == 'both' and self.state == 'confirm'):
            if self.employee_id.leave_manager_id:
                responsible = self.employee_id.leave_manager_id
            elif self.employee_id.parent_id.user_id:
                responsible = self.employee_id.parent_id.user_id
        elif self.validation_type == 'hr' or (self.validation_type == 'both' and self.state == 'validate1'):
            if self.holiday_status_id.responsible_ids:
                responsible = self.holiday_status_id.responsible_ids

        return responsible

    def activity_update(self):
        if self.env.context.get('mail_activity_automation_skip'):
            return False

        to_clean, to_do, to_do_confirm_activity = self.env['hr.leave'], self.env['hr.leave'], self.env['hr.leave']
        activity_vals = []
        today = fields.Date.today()
        model_id = self.env.ref('hr_holidays.model_hr_leave').id
        confirm_activity = self.env.ref('hr_holidays.mail_act_leave_approval')
        approval_activity = self.env.ref('hr_holidays.mail_act_leave_second_approval')
        for holiday in self:
            if holiday.state == 'draft':
                to_clean |= holiday
            elif holiday.state in ['confirm', 'validate1']:
                if holiday.holiday_status_id.leave_validation_type != 'no_validation':
                    if holiday.state == 'confirm':
                        activity_type = confirm_activity
                        note = _(
                            'New %(leave_type)s Request created by %(user)s',
                            leave_type=holiday.holiday_status_id.name,
                            user=holiday.create_uid.name,
                        )
                    else:
                        activity_type = approval_activity
                        note = _(
                            'Second approval request for %(leave_type)s',
                            leave_type=holiday.holiday_status_id.name,
                        )
                        to_do_confirm_activity |= holiday
                    user_ids = holiday.sudo()._get_responsible_for_approval().ids
                    for user_id in user_ids:
                        date_deadline = (
                            (holiday.date_from -
                             relativedelta(**{activity_type.delay_unit: activity_type.delay_count or 0})).date()
                            if holiday.date_from else today)
                        if date_deadline < today:
                            date_deadline = today
                        activity_vals.append({
                            'activity_type_id': activity_type.id,
                            'automated': True,
                            'date_deadline': date_deadline,
                            'note': note,
                            'user_id': user_id,
                            'res_id': holiday.id,
                            'res_model_id': model_id,
                        })
            elif holiday.state == 'validate':
                to_do |= holiday
            elif holiday.state == 'refuse':
                to_clean |= holiday
        if to_clean:
            to_clean.activity_unlink(['hr_holidays.mail_act_leave_approval', 'hr_holidays.mail_act_leave_second_approval'])
        if to_do_confirm_activity:
            to_do_confirm_activity.activity_feedback(['hr_holidays.mail_act_leave_approval'])
        if to_do:
            to_do.activity_feedback(['hr_holidays.mail_act_leave_approval', 'hr_holidays.mail_act_leave_second_approval'])
        self.env['mail.activity'].with_context(short_name=False).create(activity_vals)

    ####################################################
    # Messaging methods
    ####################################################

    def _notify_change(self, message, subtype_xmlid='mail.mt_note'):
        for leave in self:
            leave.message_post(body=message, subtype_xmlid=subtype_xmlid)

            recipient = None
            if leave.user_id:
                recipient = leave.user_id.partner_id.id
            elif leave.employee_id:
                recipient = leave.employee_id.work_contact_id.id

            if recipient:
                self.env['mail.thread'].sudo().message_notify(
                    body=message,
                    partner_ids=[recipient],
                    subject=_('Your Time Off'),
                )

    def _track_subtype(self, init_values):
        if 'state' in init_values and self.state == 'validate':
            leave_notif_subtype = self.holiday_status_id.leave_notif_subtype_id
            return leave_notif_subtype or self.env.ref('hr_holidays.mt_leave')
        return super(HolidaysRequest, self)._track_subtype(init_values)

    def _notify_get_recipients_groups(self, message, model_description, msg_vals=None):
        """ Handle HR users and officers recipients that can validate or refuse holidays
        directly from email. """
        groups = super()._notify_get_recipients_groups(
            message, model_description, msg_vals=msg_vals
        )
        if not self:
            return groups

        local_msg_vals = dict(msg_vals or {})

        self.ensure_one()
        hr_actions = []
        if self.state == 'confirm':
            app_action = self._notify_get_action_link('controller', controller='/leave/validate', **local_msg_vals)
            hr_actions += [{'url': app_action, 'title': _('Approve')}]
        if self.state in ['confirm', 'validate', 'validate1']:
            ref_action = self._notify_get_action_link('controller', controller='/leave/refuse', **local_msg_vals)
            hr_actions += [{'url': ref_action, 'title': _('Refuse')}]

        holiday_user_group_id = self.env.ref('hr_holidays.group_hr_holidays_user').id
        new_group = (
            'group_hr_holidays_user',
            lambda pdata: pdata['type'] == 'user' and holiday_user_group_id in pdata['groups'],
            {
                'actions': hr_actions,
                'active': True,
                'has_button_access': True,
            }
        )

        return [new_group] + groups

    def message_subscribe(self, partner_ids=None, subtype_ids=None):
        # due to record rule can not allow to add follower and mention on validated leave so subscribe through sudo
        if any(holiday.state in ['validate', 'validate1'] for holiday in self):
            self.check_access_rights('read')
            self.check_access_rule('read')
            return super(HolidaysRequest, self.sudo()).message_subscribe(partner_ids=partner_ids, subtype_ids=subtype_ids)
        return super(HolidaysRequest, self).message_subscribe(partner_ids=partner_ids, subtype_ids=subtype_ids)

    @api.model
    def get_unusual_days(self, date_from, date_to=None):
        employee_id = self.env.context.get('employee_id', False)
        employee = self.env['hr.employee'].browse(employee_id) if employee_id else self.env.user.employee_id
        return employee.sudo(False)._get_unusual_days(date_from, date_to)

    def _to_utc(self, date, hour, resource):
        hour = float_to_time(float(hour))
        holiday_tz = timezone(resource.tz or self.env.user.tz or 'UTC')
        return holiday_tz.localize(datetime.combine(date, hour)).astimezone(UTC).replace(tzinfo=None)

    def _get_attendances(self, request_date_from, request_date_to, day_period=None):
        self.ensure_one()
        domain = [
            ('calendar_id', '=', self.resource_calendar_id.id),
            ('display_type', '=', False),
            ('day_period', '!=', 'lunch'),
        ]
        if day_period:
            domain.append(('day_period', '=', day_period))
        attendances = self.env['resource.calendar.attendance']._read_group(domain,
            ['week_type', 'dayofweek', 'day_period'],
            ['hour_from:min', 'hour_to:max'])

        # Must be sorted by dayofweek ASC and day_period DESC
        attendances = sorted([DummyAttendance(hour_from, hour_to, dayofweek, day_period, week_type) for week_type, dayofweek, day_period, hour_from, hour_to in attendances], key=lambda att: (att.dayofweek, att.day_period != 'morning'))

        default_value = DummyAttendance(0, 0, 0, 'morning', False)

        if self.resource_calendar_id.two_weeks_calendar:
            # find week type of start_date
            start_week_type = self.env['resource.calendar.attendance'].get_week_type(request_date_from)
            attendance_actual_week = [att for att in attendances if att.week_type is False or int(att.week_type) == start_week_type]
            attendance_actual_next_week = [att for att in attendances if att.week_type is False or int(att.week_type) != start_week_type]
            # First, add days of actual week coming after date_from
            attendance_filtred = [att for att in attendance_actual_week if int(att.dayofweek) >= request_date_from.weekday()]
            # Second, add days of the other type of week
            attendance_filtred += list(attendance_actual_next_week)
            # Third, add days of actual week (to consider days that we have remove first because they coming before date_from)
            attendance_filtred += list(attendance_actual_week)
            end_week_type = self.env['resource.calendar.attendance'].get_week_type(request_date_to)
            attendance_actual_week = [att for att in attendances if att.week_type is False or int(att.week_type) == end_week_type]
            attendance_actual_next_week = [att for att in attendances if att.week_type is False or int(att.week_type) != end_week_type]
            attendance_filtred_reversed = list(reversed([att for att in attendance_actual_week if int(att.dayofweek) <= request_date_to.weekday()]))
            attendance_filtred_reversed += list(reversed(attendance_actual_next_week))
            attendance_filtred_reversed += list(reversed(attendance_actual_week))

            # find first attendance coming after first_day
            attendance_from = attendance_filtred[0]
            # find last attendance coming before last_day
            attendance_to = attendance_filtred_reversed[0]
        else:
            # find first attendance coming after first_day
            attendance_from = next((att for att in attendances if int(att.dayofweek) >= request_date_from.weekday()), attendances[0] if attendances else default_value)
            # find last attendance coming before last_day
            attendance_to = next((att for att in reversed(attendances) if int(att.dayofweek) <= request_date_to.weekday()), attendances[-1] if attendances else default_value)

        return (attendance_from, attendance_to)

    ####################################################
    # Cron methods
    ####################################################

    @api.model
    def _cancel_invalid_leaves(self):
        inspected_date = fields.Date.today() + timedelta(days=31)
        start_datetime = datetime.combine(fields.Date.today(), datetime.min.time())
        end_datetime = datetime.combine(inspected_date, datetime.max.time())
        concerned_leaves = self.search([
            ('date_from', '>=', start_datetime),
            ('date_from', '<=', end_datetime),
            ('state', 'in', ['confirm', 'validate1', 'validate']),
        ], order='date_from desc')
        accrual_allocations = self.env['hr.leave.allocation'].search([
            ('employee_id', 'in', concerned_leaves.employee_id.ids),
            ('holiday_status_id', 'in', concerned_leaves.holiday_status_id.ids),
            ('allocation_type', '=', 'accrual'),
            ('date_from', '<=', end_datetime),
            '|',
            ('date_to', '>=', start_datetime),
            ('date_to', '=', False),
        ])
        # only take leaves linked to accruals
        concerned_leaves = concerned_leaves\
            .filtered(lambda leave: leave.holiday_status_id in accrual_allocations.holiday_status_id)\
            .sorted('date_from', reverse=True)
        reason = _("the accruated amount is insufficient for that duration.")
        for leave in concerned_leaves:
            leave_type = leave.holiday_status_id
            date = leave.date_from.date()
            leave_type_data = leave_type.get_allocation_data(leave.employee_id, date)
            exceeding_duration = leave_type_data[leave.employee_id][0][1]['total_virtual_excess']
            excess_limit = leave_type.max_allowed_negative if leave_type.allows_negative else 0
            if exceeding_duration <= excess_limit:
                continue
            leave._force_cancel(reason, 'mail.mt_note')

```

## File: models\hr_leave_accrual_plan.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError

from odoo.addons.hr_holidays.models.hr_leave_accrual_plan_level import _get_selection_days

DAY_SELECT_VALUES = [str(i) for i in range(1, 29)] + ['last']
DAY_SELECT_SELECTION_NO_LAST = tuple(zip(DAY_SELECT_VALUES, (str(i) for i in range(1, 29))))


class AccrualPlan(models.Model):
    _name = "hr.leave.accrual.plan"
    _description = "Accrual Plan"

    active = fields.Boolean(default=True)
    name = fields.Char('Name', required=True)
    time_off_type_id = fields.Many2one('hr.leave.type', string="Time Off Type",
        check_company=True,
        help="""Specify if this accrual plan can only be used with this Time Off Type.
                Leave empty if this accrual plan can be used with any Time Off Type.""")
    employees_count = fields.Integer("Employees", compute='_compute_employee_count')
    level_ids = fields.One2many('hr.leave.accrual.level', 'accrual_plan_id', copy=True, string="Milestone")
    allocation_ids = fields.One2many('hr.leave.allocation', 'accrual_plan_id')
    company_id = fields.Many2one('res.company', string='Company',
        compute="_compute_company_id", store="True", readonly=False)
    transition_mode = fields.Selection([
        ('immediately', 'Immediately'),
        ('end_of_accrual', "After this accrual's period")],
        string="Milestone Transition", default="immediately", required=True,
        help="""Specify what occurs if a level transition takes place in the middle of a pay period.\n
                'Immediately' will switch the employee to the new accrual level on the exact date during the ongoing pay period.\n
                'After this accrual's period' will keep the employee on the same accrual level until the ongoing pay period is complete.
                After it is complete, the new level will take effect when the next pay period begins.""")
    show_transition_mode = fields.Boolean(compute='_compute_show_transition_mode')
    is_based_on_worked_time = fields.Boolean("Based on worked time", compute="_compute_is_based_on_worked_time", store=True, readonly=False,
        help="If checked, the accrual period will be calculated according to the work days, not calendar days.")
    accrued_gain_time = fields.Selection([
        ("start", "At the start of the accrual period"),
        ("end", "At the end of the accrual period")],
        default="end", required=True)
    carryover_date = fields.Selection([
        ("year_start", "At the start of the year"),
        ("allocation", "At the allocation date"),
        ("other", "Other")],
        default="year_start", required=True, string="Carry-Over Time")
    carryover_day = fields.Integer(default=1)
    carryover_day_display = fields.Selection(
        _get_selection_days, compute='_compute_carryover_day_display', inverse='_inverse_carryover_day_display')
    carryover_month = fields.Selection([
        ("jan", "January"),
        ("feb", "February"),
        ("mar", "March"),
        ("apr", "April"),
        ("may", "May"),
        ("jun", "June"),
        ("jul", "July"),
        ("aug", "August"),
        ("sep", "September"),
        ("oct", "October"),
        ("nov", "November"),
        ("dec", "December")
    ], default="jan")
    added_value_type = fields.Selection([('day', 'Days'), ('hour', 'Hours')], compute='_compute_added_value_type', store=True)

    @api.depends('level_ids')
    def _compute_show_transition_mode(self):
        for plan in self:
            plan.show_transition_mode = len(plan.level_ids) > 1

    level_count = fields.Integer('Levels', compute='_compute_level_count')

    @api.depends('level_ids')
    def _compute_level_count(self):
        level_read_group = self.env['hr.leave.accrual.level']._read_group(
            [('accrual_plan_id', 'in', self.ids)],
            groupby=['accrual_plan_id'],
            aggregates=['__count'],
        )
        mapped_count = {accrual_plan.id: count for accrual_plan, count in level_read_group}
        for plan in self:
            plan.level_count = mapped_count.get(plan.id, 0)

    @api.depends('allocation_ids')
    def _compute_employee_count(self):
        allocations_read_group = self.env['hr.leave.allocation']._read_group(
            [('accrual_plan_id', 'in', self.ids)],
            ['accrual_plan_id'],
            ['employee_id:count_distinct'],
        )
        allocations_dict = {accrual_plan.id: count for accrual_plan, count in allocations_read_group}
        for plan in self:
            plan.employees_count = allocations_dict.get(plan.id, 0)

    @api.depends('time_off_type_id.company_id')
    def _compute_company_id(self):
        for accrual_plan in self:
            if accrual_plan.time_off_type_id:
                accrual_plan.company_id = accrual_plan.time_off_type_id.company_id
            else:
                accrual_plan.company_id = self.env.company

    @api.depends("accrued_gain_time")
    def _compute_is_based_on_worked_time(self):
        for plan in self:
            if plan.accrued_gain_time == "start":
                plan.is_based_on_worked_time = False

    @api.depends("level_ids")
    def _compute_added_value_type(self):
        for plan in self:
            if plan.level_ids:
                plan.added_value_type = plan.level_ids[0].added_value_type

    @api.depends("carryover_day")
    def _compute_carryover_day_display(self):
        days_select = _get_selection_days(self)
        for plan in self:
            plan.carryover_day_display = days_select[min(plan.carryover_day - 1, 28)][0]

    def _inverse_carryover_day_display(self):
        for plan in self:
            if plan.carryover_day_display == 'last':
                plan.carryover_day = 31
            else:
                plan.carryover_day = DAY_SELECT_VALUES.index(plan.carryover_day_display) + 1

    def action_open_accrual_plan_employees(self):
        self.ensure_one()

        return {
            'name': _("Accrual Plan's Employees"),
            'type': 'ir.actions.act_window',
            'view_mode': 'kanban,tree,form',
            'res_model': 'hr.employee',
            'domain': [('id', 'in', self.allocation_ids.employee_id.ids)],
        }

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        default = dict(default or {},
                       name=_("%s (copy)", self.name))
        return super().copy(default=default)

    @api.ondelete(at_uninstall=False)
    def _prevent_used_plan_unlink(self):
        domain = [
            ('allocation_type', '=', 'accrual'),
            ('accrual_plan_id', 'in', self.ids),
            ('state', 'not in', ('cancel', 'refuse')),
        ]
        if self.env['hr.leave.allocation'].search_count(domain):
            raise ValidationError(_(
                "Some of the accrual plans you're trying to delete are linked to an existing allocation. Delete or cancel them first."
            ))

```

## File: models\hr_leave_accrual_plan_level.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from dateutil.relativedelta import relativedelta

from odoo import _, api, fields, models
from odoo.exceptions import UserError


DAYS = ['sun', 'mon', 'tue', 'wed', 'thu', 'fri', 'sat']
MONTHS = ['jan', 'feb', 'mar', 'apr', 'may', 'jun', 'jul', 'aug', 'sep', 'oct', 'nov', 'dec']
# Used for displaying the days and reversing selection -> integer
DAY_SELECT_VALUES = [str(i) for i in range(1, 29)] + ['last']
DAY_SELECT_SELECTION_NO_LAST = tuple(zip(DAY_SELECT_VALUES, (str(i) for i in range(1, 29))))

def _get_selection_days(self):
    return DAY_SELECT_SELECTION_NO_LAST + (("last", _("last day")),)


class AccrualPlanLevel(models.Model):
    _name = "hr.leave.accrual.level"
    _description = "Accrual Plan Level"
    _order = 'sequence asc'

    sequence = fields.Integer(
        string='sequence', compute='_compute_sequence', store=True,
        help='Sequence is generated automatically by start time delta.')
    accrual_plan_id = fields.Many2one('hr.leave.accrual.plan', "Accrual Plan", required=True, ondelete="cascade")
    accrued_gain_time = fields.Selection(related='accrual_plan_id.accrued_gain_time')
    start_count = fields.Integer(
        "Start after",
        help="The accrual starts after a defined period from the allocation start date. This field defines the number of days, months or years after which accrual is used.", default="1")
    start_type = fields.Selection(
        [('day', 'Days'),
         ('month', 'Months'),
         ('year', 'Years')],
        default='day', string=" ", required=True,
        help="This field defines the unit of time after which the accrual starts.")

    # Accrue of
    added_value = fields.Float(
        "Rate", digits=(16, 5), required=True, default=1,
        help="The number of hours/days that will be incremented in the specified Time Off Type for every period")
    added_value_type = fields.Selection([
        ('day', 'Days'),
        ('hour', 'Hours')
    ], compute="_compute_added_value_type", store=True, required=True, readonly=False, default="day")
    frequency = fields.Selection([
        ('hourly', 'Hourly'),
        ('daily', 'Daily'),
        ('weekly', 'Weekly'),
        ('bimonthly', 'Twice a month'),
        ('monthly', 'Monthly'),
        ('biyearly', 'Twice a year'),
        ('yearly', 'Yearly'),
    ], default='daily', required=True, string="Frequency")
    week_day = fields.Selection([
        ('mon', 'Monday'),
        ('tue', 'Tuesday'),
        ('wed', 'Wednesday'),
        ('thu', 'Thursday'),
        ('fri', 'Friday'),
        ('sat', 'Saturday'),
        ('sun', 'Sunday'),
    ], default='mon', required=True, string="Allocation on")
    first_day = fields.Integer(default=1)
    first_day_display = fields.Selection(
        _get_selection_days, compute='_compute_days_display', inverse='_inverse_first_day_display')
    second_day = fields.Integer(default=15)
    second_day_display = fields.Selection(
        _get_selection_days, compute='_compute_days_display', inverse='_inverse_second_day_display')
    first_month_day = fields.Integer(default=1)
    first_month_day_display = fields.Selection(
        _get_selection_days, compute='_compute_days_display', inverse='_inverse_first_month_day_display')
    first_month = fields.Selection([
        ('jan', 'January'),
        ('feb', 'February'),
        ('mar', 'March'),
        ('apr', 'April'),
        ('may', 'May'),
        ('jun', 'June'),
    ], default="jan")
    second_month_day = fields.Integer(default=1)
    second_month_day_display = fields.Selection(
        _get_selection_days, compute='_compute_days_display', inverse='_inverse_second_month_day_display')
    second_month = fields.Selection([
        ('jul', 'July'),
        ('aug', 'August'),
        ('sep', 'September'),
        ('oct', 'October'),
        ('nov', 'November'),
        ('dec', 'December')
    ], default="jul")
    yearly_month = fields.Selection([
        ('jan', 'January'),
        ('feb', 'February'),
        ('mar', 'March'),
        ('apr', 'April'),
        ('may', 'May'),
        ('jun', 'June'),
        ('jul', 'July'),
        ('aug', 'August'),
        ('sep', 'September'),
        ('oct', 'October'),
        ('nov', 'November'),
        ('dec', 'December')
    ], default="jan")
    yearly_day = fields.Integer(default=1)
    yearly_day_display = fields.Selection(
        _get_selection_days, compute='_compute_days_display', inverse='_inverse_yearly_day_display')
    cap_accrued_time = fields.Boolean("Cap accrued time", default=True)
    maximum_leave = fields.Float(
        'Limit to', digits=(16, 2), compute="_compute_maximum_leave", readonly=False, store=True,
        help="Choose a cap for this accrual.")
    action_with_unused_accruals = fields.Selection(
        [('lost', 'None. Accrued time reset to 0'),
         ('all', 'All accrued time carried over'),
         ('maximum', 'Carry over with a maximum')],
        string="Carry over",
        default='all', required=True)
    postpone_max_days = fields.Integer("Maximum amount of accruals to transfer",
        help="Set a maximum of accruals an allocation keeps at the end of the year.")
    can_modify_value_type = fields.Boolean(compute="_compute_can_modify_value_type")

    _sql_constraints = [
        ('check_dates',
         "CHECK( (frequency IN ('daily', 'hourly')) or"
         "(week_day IS NOT NULL AND frequency = 'weekly') or "
         "(first_day > 0 AND second_day > first_day AND first_day <= 31 AND second_day <= 31 AND frequency = 'bimonthly') or "
         "(first_day > 0 AND first_day <= 31 AND frequency = 'monthly')or "
         "(first_month_day > 0 AND first_month_day <= 31 AND second_month_day > 0 AND second_month_day <= 31 AND frequency = 'biyearly') or "
         "(yearly_day > 0 AND yearly_day <= 31 AND frequency = 'yearly'))",
         "The dates you've set up aren't correct. Please check them."),
        ('start_count_check', "CHECK( start_count >= 0 )", "You can not start an accrual in the past."),
        ('added_value_greater_than_zero', 'CHECK(added_value > 0)', 'You must give a rate greater than 0 in accrual plan levels.')
    ]

    @api.constrains('maximum_leave')
    def _check_maximum_leave(self):
        for level in self:
            if level.cap_accrued_time and level.maximum_leave < 1:
                raise UserError(_("You cannot have a cap on accrued time without setting a maximum amount."))

    @api.depends('start_count', 'start_type')
    def _compute_sequence(self):
        # Not 100% accurate because of odd months/years, but good enough
        start_type_multipliers = {
            'day': 1,
            'month': 30,
            'year': 365,
        }
        for level in self:
            level.sequence = level.start_count * start_type_multipliers[level.start_type]

    @api.depends('accrual_plan_id', 'accrual_plan_id.level_ids', 'accrual_plan_id.time_off_type_id')
    def _compute_can_modify_value_type(self):
        for level in self:
            level.can_modify_value_type = not level.accrual_plan_id.time_off_type_id and level.accrual_plan_id.level_ids and level.accrual_plan_id.level_ids[0] == level

    @api.depends('accrual_plan_id', 'accrual_plan_id.level_ids', 'accrual_plan_id.time_off_type_id')
    def _compute_added_value_type(self):
        for level in self:
            if level.accrual_plan_id.time_off_type_id:
                level.added_value_type = "day" if level.accrual_plan_id.time_off_type_id.request_unit in ["day", "half_day"] else "hour"
            elif level.accrual_plan_id.level_ids and level.accrual_plan_id.level_ids[0] != level:
                level.added_value_type = level.accrual_plan_id.level_ids[0].added_value_type

    @api.depends('first_day', 'second_day', 'first_month_day', 'second_month_day', 'yearly_day')
    def _compute_days_display(self):
        days_select = _get_selection_days(self)
        for level in self:
            level.first_day_display = days_select[min(level.first_day - 1, 28)][0]
            level.second_day_display = days_select[min(level.second_day - 1, 28)][0]
            level.first_month_day_display = days_select[min(level.first_month_day - 1, 28)][0]
            level.second_month_day_display = days_select[min(level.second_month_day - 1, 28)][0]
            level.yearly_day_display = days_select[min(level.yearly_day - 1, 28)][0]

    @api.depends('cap_accrued_time')
    def _compute_maximum_leave(self):
        for level in self:
            level.maximum_leave = 100 if level.cap_accrued_time else 0

    def _inverse_first_day_display(self):
        for level in self:
            if level.first_day_display == 'last':
                level.first_day = 31
            else:
                level.first_day = DAY_SELECT_VALUES.index(level.first_day_display) + 1

    def _inverse_second_day_display(self):
        for level in self:
            if level.second_day_display == 'last':
                level.second_day = 31
            else:
                level.second_day = DAY_SELECT_VALUES.index(level.second_day_display) + 1

    def _inverse_first_month_day_display(self):
        for level in self:
            if level.first_month_day_display == 'last':
                level.first_month_day = 31
            else:
                level.first_month_day = DAY_SELECT_VALUES.index(level.first_month_day_display) + 1

    def _inverse_second_month_day_display(self):
        for level in self:
            if level.second_month_day_display == 'last':
                level.second_month_day = 31
            else:
                level.second_month_day = DAY_SELECT_VALUES.index(level.second_month_day_display) + 1

    def _inverse_yearly_day_display(self):
        for level in self:
            if level.yearly_day_display == 'last':
                level.yearly_day = 31
            else:
                level.yearly_day = DAY_SELECT_VALUES.index(level.yearly_day_display) + 1

    def _get_next_date(self, last_call):
        """
        Returns the next date with the given last call
        """
        self.ensure_one()
        if self.frequency in ['hourly', 'daily']:
            return last_call + relativedelta(days=1)
        elif self.frequency == 'weekly':
            daynames = ['mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun']
            weekday = daynames.index(self.week_day)
            return last_call + relativedelta(days=1, weekday=weekday)
        elif self.frequency == 'bimonthly':
            first_date = last_call + relativedelta(day=self.first_day)
            second_date = last_call + relativedelta(day=self.second_day)
            if last_call < first_date:
                return first_date
            elif last_call < second_date:
                return second_date
            else:
                return last_call + relativedelta(months=1, day=self.first_day)
        elif self.frequency == 'monthly':
            date = last_call + relativedelta(day=self.first_day)
            if last_call < date:
                return date
            else:
                return last_call + relativedelta(months=1, day=self.first_day)
        elif self.frequency == 'biyearly':
            first_month = MONTHS.index(self.first_month) + 1
            second_month = MONTHS.index(self.second_month) + 1
            first_date = last_call + relativedelta(month=first_month, day=self.first_month_day)
            second_date = last_call + relativedelta(month=second_month, day=self.second_month_day)
            if last_call < first_date:
                return first_date
            elif last_call < second_date:
                return second_date
            else:
                return last_call + relativedelta(years=1, month=first_month, day=self.first_month_day)
        elif self.frequency == 'yearly':
            month = MONTHS.index(self.yearly_month) + 1
            date = last_call + relativedelta(month=month, day=self.yearly_day)
            if last_call < date:
                return date
            else:
                return last_call + relativedelta(years=1, month=month, day=self.yearly_day)
        else:
            return False

    def _get_previous_date(self, last_call):
        """
        Returns the date a potential previous call would have been at
        For example if you have a monthly level giving 16/02 would return 01/02
        Contrary to `_get_next_date` this function will return the 01/02 if that date is given
        """
        self.ensure_one()
        if self.frequency in ['hourly', 'daily']:
            return last_call
        elif self.frequency == 'weekly':
            daynames = ['mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun']
            weekday = daynames.index(self.week_day)
            return last_call + relativedelta(days=-6, weekday=weekday)
        elif self.frequency == 'bimonthly':
            second_date = last_call + relativedelta(day=self.second_day)
            first_date = last_call + relativedelta(day=self.first_day)
            if last_call >= second_date:
                return second_date
            elif last_call >= first_date:
                return first_date
            else:
                return last_call + relativedelta(months=-1, day=self.second_day)
        elif self.frequency == 'monthly':
            date = last_call + relativedelta(day=self.first_day)
            if last_call >= date:
                return date
            else:
                return last_call + relativedelta(months=-1, day=self.first_day)
        elif self.frequency == 'biyearly':
            first_month = MONTHS.index(self.first_month) + 1
            second_month = MONTHS.index(self.second_month) + 1
            first_date = last_call + relativedelta(month=first_month, day=self.first_month_day)
            second_date = last_call + relativedelta(month=second_month, day=self.second_month_day)
            if last_call >= second_date:
                return second_date
            elif last_call >= first_date:
                return first_date
            else:
                return last_call + relativedelta(years=-1, month=second_month, day=self.second_month_day)
        elif self.frequency == 'yearly':
            month = MONTHS.index(self.yearly_month) + 1
            year_date = last_call + relativedelta(month=month, day=self.yearly_day)
            if last_call >= year_date:
                return year_date
            else:
                return last_call + relativedelta(years=-1, month=month, day=self.yearly_day)
        else:
            return False

    def _get_level_transition_date(self, allocation_start):
        if self.start_type == 'day':
            return allocation_start + relativedelta(days=self.start_count)
        if self.start_type == 'month':
            return allocation_start + relativedelta(months=self.start_count)
        if self.start_type == 'year':
            return allocation_start + relativedelta(years=self.start_count)

```

## File: models\hr_leave_allocation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2005-2006 Axelor SARL. (http://www.axelor.com)

from datetime import datetime, date, time
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.addons.resource.models.utils import HOURS_PER_DAY
from odoo.addons.hr_holidays.models.hr_leave import get_employee_from_context
from odoo.exceptions import AccessError, UserError, ValidationError
from odoo.tools.float_utils import float_round
from odoo.tools.date_utils import get_timedelta
from odoo.osv import expression


MONTHS_TO_INTEGER = {"jan": 1, "feb": 2, "mar": 3, "apr": 4, "may": 5, "jun": 6, "jul": 7, "aug": 8, "sep": 9, "oct": 10, "nov": 11, "dec": 12}

class HolidaysAllocation(models.Model):
    """ Allocation Requests Access specifications: similar to leave requests """
    _name = "hr.leave.allocation"
    _description = "Time Off Allocation"
    _order = "create_date desc"
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _mail_post_access = 'read'

    def _default_holiday_status_id(self):
        if self.user_has_groups('hr_holidays.group_hr_holidays_user'):
            domain = [('has_valid_allocation', '=', True), ('requires_allocation', '=', 'yes')]
        else:
            domain = [('has_valid_allocation', '=', True), ('requires_allocation', '=', 'yes'), ('employee_requests', '=', 'yes')]
        return self.env['hr.leave.type'].search(domain, limit=1)

    def _domain_holiday_status_id(self):
        if self.user_has_groups('hr_holidays.group_hr_holidays_user'):
            return [('requires_allocation', '=', 'yes')]
        return [('employee_requests', '=', 'yes')]

    name = fields.Char(
        string='Description',
        compute='_compute_description',
        inverse='_inverse_description',
        search='_search_description',
        compute_sudo=False)
    name_validity = fields.Char('Description with validity', compute='_compute_description_validity')
    active = fields.Boolean(default=True)
    private_name = fields.Char('Allocation Description', groups='hr_holidays.group_hr_holidays_user')
    state = fields.Selection([
        ('confirm', 'To Approve'),
        ('refuse', 'Refused'),
        ('validate', 'Approved')],
        string='Status', readonly=True, tracking=True, copy=False, default='confirm',
        help="The status is set to 'To Submit', when an allocation request is created."
        "\nThe status is 'To Approve', when an allocation request is confirmed by user."
        "\nThe status is 'Refused', when an allocation request is refused by manager."
        "\nThe status is 'Approved', when an allocation request is approved by manager.")
    date_from = fields.Date('Start Date', index=True, copy=False, default=fields.Date.context_today,
        tracking=True, required=True)
    date_to = fields.Date('End Date', copy=False, tracking=True)
    holiday_status_id = fields.Many2one(
        "hr.leave.type", compute='_compute_holiday_status_id', store=True, string="Time Off Type", required=True, readonly=False,
        domain=_domain_holiday_status_id,
        default=_default_holiday_status_id)
    employee_id = fields.Many2one(
        'hr.employee', compute='_compute_from_employee_ids', store=True, string='Employee', index=True, readonly=False, ondelete="restrict", tracking=True)
    employee_company_id = fields.Many2one(related='employee_id.company_id', readonly=True, store=True)
    active_employee = fields.Boolean('Active Employee', related='employee_id.active', readonly=True)
    manager_id = fields.Many2one('hr.employee', compute='_compute_manager_id', store=True, string='Manager')
    notes = fields.Text('Reasons', readonly=False)
    # duration
    number_of_days = fields.Float(
        'Number of Days', compute='_compute_from_holiday_status_id', store=True, readonly=False, tracking=True, default=1,
        help='Duration in days. Reference field to use when necessary.')
    number_of_days_display = fields.Float(
        'Duration (days)', compute='_compute_number_of_days_display',
        help="For an Accrual Allocation, this field contains the theorical amount of time given to the employee, due to a previous start date, on the first run of the plan. This can be manually edited.")
    number_of_hours_display = fields.Float(
        'Duration (hours)', compute='_compute_number_of_hours_display',
        help="For an Accrual Allocation, this field contains the theorical amount of time given to the employee, due to a previous start date, on the first run of the plan. This can be manually edited.")
    duration_display = fields.Char('Allocated (Days/Hours)', compute='_compute_duration_display',
        help="Field allowing to see the allocation duration in days or hours depending on the type_request_unit")
    # details
    parent_id = fields.Many2one('hr.leave.allocation', string='Parent')
    linked_request_ids = fields.One2many('hr.leave.allocation', 'parent_id', string='Linked Requests')
    approver_id = fields.Many2one(
        'hr.employee', string='First Approval', readonly=True, copy=False,
        help='This area is automatically filled by the user who validates the allocation')
    validation_type = fields.Selection(string='Validation Type', related='holiday_status_id.allocation_validation_type', readonly=True)
    can_approve = fields.Boolean('Can Approve', compute='_compute_can_approve')
    type_request_unit = fields.Selection([
        ('hour', 'Hours'),
        ('half_day', 'Half Day'),
        ('day', 'Day'),
    ], compute="_compute_type_request_unit")
    # mode
    holiday_type = fields.Selection([
        ('employee', 'By Employee'),
        ('company', 'By Company'),
        ('department', 'By Department'),
        ('category', 'By Employee Tag')],
        string='Allocation Mode', readonly=False, required=True, default='employee',
        help="Allow to create requests in batchs:\n- By Employee: for a specific employee"
             "\n- By Company: all employees of the specified company"
             "\n- By Department: all employees of the specified department"
             "\n- By Employee Tag: all employees of the specific employee group category")
    employee_ids = fields.Many2many(
        'hr.employee', compute='_compute_from_holiday_type', store=True, string='Employees', readonly=False)
    multi_employee = fields.Boolean(
        compute='_compute_from_employee_ids', store=True,
        help='Holds whether this allocation concerns more than 1 employee')
    mode_company_id = fields.Many2one(
        'res.company', compute='_compute_from_holiday_type', store=True, string='Company Mode', readonly=False)
    department_id = fields.Many2one(
        'hr.department', compute='_compute_department_id', store=True, string='Department',
        readonly=False)
    category_id = fields.Many2one(
        'hr.employee.category', compute='_compute_from_holiday_type', store=True, string='Employee Tag', readonly=False)
    # accrual configuration
    lastcall = fields.Date("Date of the last accrual allocation", readonly=True)
    nextcall = fields.Date("Date of the next accrual allocation", readonly=True, default=False)
    already_accrued = fields.Boolean()
    allocation_type = fields.Selection([
        ('regular', 'Regular Allocation'),
        ('accrual', 'Accrual Allocation')
    ], string="Allocation Type", default="regular", required=True, readonly=True)
    is_officer = fields.Boolean(compute='_compute_is_officer')
    accrual_plan_id = fields.Many2one('hr.leave.accrual.plan',
        compute="_compute_from_holiday_status_id", store=True, readonly=False, tracking=True,
        domain="['|', ('time_off_type_id', '=', False), ('time_off_type_id', '=', holiday_status_id)]")
    max_leaves = fields.Float(compute='_compute_leaves')
    leaves_taken = fields.Float(compute='_compute_leaves', string='Time off Taken')
    has_accrual_plan = fields.Boolean(compute='_compute_has_accrual_plan', string='Accrual Plan Available')

    _sql_constraints = [
        ('type_value',
         "CHECK( (holiday_type='employee' AND (employee_id IS NOT NULL OR multi_employee IS TRUE)) or "
         "(holiday_type='category' AND category_id IS NOT NULL) or "
         "(holiday_type='department' AND department_id IS NOT NULL) or "
         "(holiday_type='company' AND mode_company_id IS NOT NULL))",
         "The employee, department, company or employee category of this request is missing. Please make sure that your user login is linked to an employee."),
        ('duration_check', "CHECK( ( number_of_days > 0 AND allocation_type='regular') or (allocation_type != 'regular'))", "The duration must be greater than 0."),
    ]

    @api.constrains('date_from', 'date_to')
    def _check_date_from_date_to(self):
        if any(allocation.date_to and allocation.date_from > allocation.date_to for allocation in self):
            raise UserError(_("The Start Date of the Validity Period must be anterior to the End Date."))

    # The compute does not get triggered without a depends on record creation
    # aka keep the 'useless' depends
    @api.depends_context('uid')
    @api.depends('allocation_type')
    def _compute_is_officer(self):
        self.is_officer = self.env.user.has_group("hr_holidays.group_hr_holidays_user")

    # Useless depends, so that name is computed on new, before saving the record
    @api.depends_context('uid')
    @api.depends('holiday_status_id')
    def _compute_description(self):
        self.check_access_rights('read')
        self.check_access_rule('read')
        for allocation in self:
            if not allocation.env.context.get('is_employee_allocation'):
                allocation.name = allocation.sudo().private_name
            elif not allocation.holiday_status_id:
                allocation.name = _("Allocation Request")
            elif allocation.type_request_unit == 'hour':
                allocation.name = _(
                    '%(name)s (%(duration)s hour(s))',
                    name=allocation.holiday_status_id.name,
                    duration=allocation.number_of_days * (
                        allocation.employee_id.sudo().resource_calendar_id.hours_per_day
                        or allocation.holiday_status_id.company_id.resource_calendar_id.hours_per_day
                        or HOURS_PER_DAY
                    ),
                )
            else:
                allocation.name = _(
                    '%(name)s (%(duration)s day(s))',
                    name=allocation.holiday_status_id.name,
                    duration=allocation.number_of_days,
                )

    def _inverse_description(self):
        is_officer = self.env.user.has_group('hr_holidays.group_hr_holidays_user')
        for allocation in self:
            if is_officer or allocation.employee_id.user_id == self.env.user or allocation.employee_id.leave_manager_id == self.env.user:
                allocation.sudo().private_name = allocation.name

    def _search_description(self, operator, value):
        is_officer = self.env.user.has_group('hr_holidays.group_hr_holidays_user')
        domain = [('private_name', operator, value)]

        if not is_officer:
            domain = expression.AND([domain, [('employee_id.user_id', '=', self.env.user.id)]])

        allocations = self.sudo().search(domain)
        return [('id', 'in', allocations.ids)]

    @api.depends('accrual_plan_id')
    def _compute_has_accrual_plan(self):
        self.has_accrual_plan = bool(self.env['hr.leave.accrual.plan'].sudo().search_count([('active', '=', True)]))

    @api.depends('name', 'date_from', 'date_to')
    def _compute_description_validity(self):
        for allocation in self:
            if allocation.date_to:
                name_validity = _("%s (from %s to %s)", allocation.name, allocation.date_from.strftime("%b %d %Y"), allocation.date_to.strftime("%b %d %Y"))
            else:
                name_validity = _("%s (from %s to No Limit)", allocation.name, allocation.date_from.strftime("%b %d %Y"))
            allocation.name_validity = name_validity

    @api.depends('employee_id', 'holiday_status_id')
    def _compute_leaves(self):
        date_from = fields.Date.from_string(self._context['default_date_from']) if 'default_date_from' in self._context else fields.Date.today()
        employee_days_per_allocation = self.employee_id._get_consumed_leaves(self.holiday_status_id, date_from, ignore_future=True)[0]
        for allocation in self:
            allocation.max_leaves = allocation.number_of_hours_display if allocation.type_request_unit == 'hour' else allocation.number_of_days
            origin = allocation._origin
            allocation.leaves_taken = employee_days_per_allocation[origin.employee_id][origin.holiday_status_id][origin]['leaves_taken']

    @api.depends('number_of_days')
    def _compute_number_of_days_display(self):
        for allocation in self:
            allocation.number_of_days_display = allocation.number_of_days

    @api.depends('number_of_days', 'holiday_status_id', 'employee_id', 'holiday_type')
    def _compute_number_of_hours_display(self):
        for allocation in self:
            allocation_calendar = allocation.holiday_status_id.company_id.resource_calendar_id
            if allocation.holiday_type == 'employee' and allocation.employee_id:
                allocation_calendar = allocation.employee_id.sudo().resource_calendar_id

            allocation.number_of_hours_display = allocation.number_of_days * (allocation_calendar.hours_per_day or HOURS_PER_DAY)

    @api.depends('number_of_hours_display', 'number_of_days_display')
    def _compute_duration_display(self):
        for allocation in self:
            allocation.duration_display = '%g %s' % (
                (float_round(allocation.number_of_hours_display, precision_digits=2)
                if allocation.type_request_unit == 'hour'
                else float_round(allocation.number_of_days_display, precision_digits=2)),
                _('hours') if allocation.type_request_unit == 'hour' else _('days'))

    @api.depends('state', 'employee_id', 'department_id')
    def _compute_can_approve(self):
        for allocation in self:
            try:
                if allocation.state == 'confirm' and allocation.validation_type != 'no':
                    allocation._check_approval_update('validate')
            except (AccessError, UserError):
                allocation.can_approve = False
            else:
                allocation.can_approve = True

    @api.depends('employee_ids', 'allocation_type', 'accrual_plan_id')
    def _compute_from_employee_ids(self):
        for allocation in self:
            accrual_plan = allocation.accrual_plan_id
            if len(allocation.employee_ids) == 1 or (
                allocation.employee_ids and not accrual_plan.is_based_on_worked_time
                and accrual_plan.level_ids and not accrual_plan.level_ids[0].frequency == 'hourly'
            ):
                allocation.employee_id = allocation.employee_ids[0]._origin
            else:
                allocation.employee_id = False
            allocation.multi_employee = len(allocation.employee_ids) > 1

    @api.depends('holiday_type')
    def _compute_from_holiday_type(self):
        default_employee_ids = self.env['hr.employee'].browse(self.env.context.get('default_employee_id')) or self.env.user.employee_id
        for allocation in self:
            if allocation.holiday_type == 'employee':
                if not allocation.employee_ids:
                    allocation.employee_ids = allocation.employee_id or self.env.user.employee_id
                allocation.mode_company_id = False
                allocation.category_id = False
            elif allocation.holiday_type == 'company':
                allocation.employee_ids = False
                if not allocation.mode_company_id:
                    allocation.mode_company_id = self.env.company
                allocation.category_id = False
            elif allocation.holiday_type == 'department':
                allocation.employee_ids = False
                allocation.mode_company_id = False
                allocation.category_id = False
            elif allocation.holiday_type == 'category':
                allocation.employee_ids = False
                allocation.mode_company_id = False
            else:
                allocation.employee_ids = default_employee_ids

    @api.depends('holiday_type', 'employee_id')
    def _compute_department_id(self):
        for allocation in self:
            if allocation.holiday_type == 'employee':
                allocation.department_id = allocation.employee_id.department_id
            elif allocation.holiday_type == 'department':
                if not allocation.department_id:
                    allocation.department_id = self.env.user.employee_id.department_id
            elif allocation.holiday_type == 'category':
                allocation.department_id = False

    @api.depends('employee_id')
    def _compute_manager_id(self):
        for allocation in self:
            allocation.manager_id = allocation.employee_id and allocation.employee_id.parent_id

    @api.depends('accrual_plan_id')
    def _compute_holiday_status_id(self):
        default_holiday_status_id = None
        for allocation in self:
            if not allocation.holiday_status_id:
                if allocation.accrual_plan_id:
                    allocation.holiday_status_id = allocation.accrual_plan_id.time_off_type_id
                else:
                    if not default_holiday_status_id:  # fetch when we need it
                        default_holiday_status_id = self._default_holiday_status_id()
                    allocation.holiday_status_id = default_holiday_status_id

    @api.depends('holiday_status_id', 'allocation_type', 'number_of_hours_display', 'number_of_days_display', 'date_to')
    def _compute_from_holiday_status_id(self):
        accrual_allocations = self.filtered(lambda alloc: alloc.allocation_type == 'accrual' and not alloc.accrual_plan_id and alloc.holiday_status_id)
        accruals_read_group = self.env['hr.leave.accrual.plan']._read_group(
            [('time_off_type_id', 'in', accrual_allocations.holiday_status_id.ids)],
            ['time_off_type_id'],
            ['id:array_agg'],
        )
        accruals_dict = {time_off_type.id: ids for time_off_type, ids in accruals_read_group}
        for allocation in self:
            allocation_unit = allocation._get_request_unit()
            if allocation_unit != 'hour':
                allocation.number_of_days = allocation.number_of_days_display
            else:
                hours_per_day = allocation.employee_id.sudo().resource_calendar_id.hours_per_day\
                    or allocation.holiday_status_id.company_id.resource_calendar_id.hours_per_day\
                    or HOURS_PER_DAY
                allocation.number_of_days = allocation.number_of_hours_display / hours_per_day
            if allocation.accrual_plan_id.time_off_type_id.id not in (False, allocation.holiday_status_id.id):
                allocation.accrual_plan_id = False
            if allocation.allocation_type == 'accrual' and not allocation.accrual_plan_id:
                if allocation.holiday_status_id:
                    allocation.accrual_plan_id = accruals_dict.get(allocation.holiday_status_id.id, [False])[0]

    def _get_request_unit(self):
        self.ensure_one()
        if self.allocation_type == "accrual" and self.accrual_plan_id:
            return self.accrual_plan_id.sudo().added_value_type
        elif self.allocation_type == "regular":
            return self.holiday_status_id.request_unit
        else:
            return "day"

    @api.depends("allocation_type", "holiday_status_id", "accrual_plan_id")
    def _compute_type_request_unit(self):
        for allocation in self:
            allocation.type_request_unit = allocation._get_request_unit()

    def _get_carryover_date(self, date_from):
        self.ensure_one()
        carryover_time = self.accrual_plan_id.carryover_date
        accrual_plan = self.accrual_plan_id
        carryover_date = False
        if carryover_time == 'year_start':
            carryover_date = date(date_from.year, 1, 1)
        elif carryover_time == 'allocation':
            carryover_date = date(date_from.year, self.date_from.month, self.date_from.day)
        else:
            carryover_date = date(date_from.year, MONTHS_TO_INTEGER[accrual_plan.carryover_month], accrual_plan.carryover_day)
        if date_from > carryover_date:
            carryover_date += relativedelta(years=1)
        return carryover_date

    def _add_days_to_allocation(self, current_level, current_level_maximum_leave, leaves_taken, period_start, period_end):
        days_to_add = self._process_accrual_plan_level(
            current_level, period_start, self.lastcall, period_end, self.nextcall)
        self.number_of_days += days_to_add
        if current_level.cap_accrued_time:
            self.number_of_days = min(self.number_of_days, current_level_maximum_leave + leaves_taken)

    def _get_current_accrual_plan_level_id(self, date, level_ids=False):
        """
        Returns a pair (accrual_plan_level, idx) where accrual_plan_level is the level for the given date
        and idx is the index for the plan in the ordered set of levels
        """
        self.ensure_one()
        if not self.accrual_plan_id.level_ids:
            return (False, False)
        # Sort by sequence which should be equivalent to the level
        if not level_ids:
            level_ids = self.accrual_plan_id.level_ids.sorted('sequence')
        current_level = False
        current_level_idx = -1
        for idx, level in enumerate(level_ids):
            if date > self.date_from + get_timedelta(level.start_count, level.start_type):
                current_level = level
                current_level_idx = idx
        # If transition_mode is set to `immediately` or we are currently on the first level
        # the current_level is simply the first level in the list.
        if current_level_idx <= 0 or self.accrual_plan_id.transition_mode == "immediately":
            return (current_level, current_level_idx)
        # In this case we have to verify that the 'previous level' is not the current one due to `end_of_accrual`
        level_start_date = self.date_from + get_timedelta(current_level.start_count, current_level.start_type)
        previous_level = level_ids[current_level_idx - 1]
        # If the next date from the current level's start date is before the last call of the previous level
        # return the previous level
        if current_level._get_next_date(level_start_date) < previous_level._get_next_date(level_start_date):
            return (previous_level, current_level_idx - 1)
        return (current_level, current_level_idx)

    def _get_accrual_plan_level_work_entry_prorata(self, level, start_period, start_date, end_period, end_date):
        self.ensure_one()
        datetime_min_time = datetime.min.time()
        start_dt = datetime.combine(start_date, datetime_min_time)
        end_dt = datetime.combine(end_date, datetime_min_time)
        worked = self.employee_id._get_work_days_data_batch(start_dt, end_dt, calendar=self.employee_id.resource_calendar_id)\
            [self.employee_id.id]['hours']
        if start_period != start_date or end_period != end_date:
            start_dt = datetime.combine(start_period, datetime_min_time)
            end_dt = datetime.combine(end_period, datetime_min_time)
            planned_worked = self.employee_id._get_work_days_data_batch(start_dt, end_dt, calendar=self.employee_id.resource_calendar_id)\
                [self.employee_id.id]['hours']
        else:
            planned_worked = worked
        left = self.employee_id.sudo()._get_leave_days_data_batch(start_dt, end_dt, calendar=self.employee_id._get_calendar(start_dt),
            domain=[('time_type', '=', 'leave')])[self.employee_id.id]['hours']
        if level.frequency == 'hourly':
            if level.accrual_plan_id.is_based_on_worked_time:
                work_entry_prorata = planned_worked
            else:
                work_entry_prorata = planned_worked + left
        else:
            work_entry_prorata = worked / (left + planned_worked) if (left + planned_worked) else 0
        return work_entry_prorata

    def _process_accrual_plan_level(self, level, start_period, start_date, end_period, end_date):
        """
        Returns the added days for that level
        """
        self.ensure_one()
        if level.frequency == 'hourly' or level.accrual_plan_id.is_based_on_worked_time:
            work_entry_prorata = self._get_accrual_plan_level_work_entry_prorata(level, start_period, start_date, end_period, end_date)
            added_value = work_entry_prorata * level.added_value
        else:
            added_value = level.added_value
        # Convert time in hours to time in days in case the level is encoded in hours
        if level.added_value_type == 'hour':
            added_value = added_value / (self.employee_id.sudo().resource_id.calendar_id.hours_per_day or HOURS_PER_DAY)
        period_prorata = 1
        if (start_period != start_date or end_period != end_date) and not level.accrual_plan_id.is_based_on_worked_time:
            period_days = (end_period - start_period)
            call_days = (end_date - start_date)
            period_prorata = min(1, call_days / period_days) if period_days else 1
        return added_value * period_prorata

    def _process_accrual_plans(self, date_to=False, force_period=False, log=True):
        """
        This method is part of the cron's process.
        The goal of this method is to retroactively apply accrual plan levels and progress from nextcall to date_to or today.
        If force_period is set, the accrual will run until date_to in a prorated way (used for end of year accrual actions).
        """
        date_to = date_to or fields.Date.today()
        already_accrued = {allocation.id: allocation.already_accrued or (allocation.number_of_days != 0 and allocation.accrual_plan_id.accrued_gain_time == 'start') for allocation in self}
        first_allocation = _("""This allocation have already ran once, any modification won't be effective to the days allocated to the employee. If you need to change the configuration of the allocation, delete and create a new one.""")
        for allocation in self:
            level_ids = allocation.accrual_plan_id.level_ids.sorted('sequence')
            if not level_ids:
                continue
            # "cache" leaves taken, as it gets recomputed every time allocation.number_of_days is assigned to. Without this,
            # every loop will take 1+ second. It can be removed if computes don't chain in a way to always reassign accrual plan
            # even if the value doesn't change. This is the best performance atm.
            first_level = level_ids[0]
            first_level_start_date = allocation.date_from + get_timedelta(first_level.start_count, first_level.start_type)
            leaves_taken = allocation.leaves_taken if allocation.holiday_status_id.request_unit in ["day", "half_day"] else allocation.leaves_taken / (allocation.employee_id.sudo().resource_id.calendar_id.hours_per_day or HOURS_PER_DAY)
            allocation.already_accrued = already_accrued[allocation.id]
            # first time the plan is run, initialize nextcall and take carryover / level transition into account
            if not allocation.nextcall:
                # Accrual plan is not configured properly or has not started
                if date_to < first_level_start_date:
                    continue
                allocation.lastcall = max(allocation.lastcall, first_level_start_date)
                allocation.nextcall = first_level._get_next_date(allocation.lastcall)
                # adjust nextcall for carryover
                carryover_date = allocation._get_carryover_date(allocation.nextcall)
                allocation.nextcall = min(carryover_date, allocation.nextcall)
                # adjust nextcall for level_transition
                if len(level_ids) > 1:
                    second_level_start_date = allocation.date_from + get_timedelta(level_ids[1].start_count, level_ids[1].start_type)
                    allocation.nextcall = min(second_level_start_date, allocation.nextcall)
                if log:
                    allocation._message_log(body=first_allocation)
            (current_level, current_level_idx) = (False, 0)
            current_level_maximum_leave = 0.0
            # all subsequent runs, at every loop:
            # get current level and normal period boundaries, then set nextcall, adjusted for level transition and carryover
            # add days, trimmed if there is a maximum_leave
            while allocation.nextcall <= date_to:
                (current_level, current_level_idx) = allocation._get_current_accrual_plan_level_id(allocation.nextcall)
                if not current_level:
                    break
                if current_level.cap_accrued_time:
                    current_level_maximum_leave = current_level.maximum_leave if current_level.added_value_type == "day" else current_level.maximum_leave / (allocation.employee_id.sudo().resource_id.calendar_id.hours_per_day or HOURS_PER_DAY)
                nextcall = current_level._get_next_date(allocation.nextcall)
                # Since _get_previous_date returns the given date if it corresponds to a call date
                # this will always return lastcall except possibly on the first call
                # this is used to prorate the first number of days given to the employee
                period_start = current_level._get_previous_date(allocation.lastcall)
                period_end = current_level._get_next_date(allocation.lastcall)
                # There are 2 cases where nextcall could be closer than the normal period:
                # 1. Passing from one level to another, if mode is set to 'immediately'
                if current_level_idx < (len(level_ids) - 1) and allocation.accrual_plan_id.transition_mode == 'immediately':
                    next_level = level_ids[current_level_idx + 1]
                    current_level_last_date = allocation.date_from + get_timedelta(next_level.start_count, next_level.start_type)
                    if allocation.nextcall != current_level_last_date:
                        nextcall = min(nextcall, current_level_last_date)
                # 2. On carry-over date
                carryover_date = allocation._get_carryover_date(allocation.nextcall)
                if allocation.nextcall < carryover_date < nextcall:
                    nextcall = min(nextcall, carryover_date)
                if not allocation.already_accrued:
                    allocation._add_days_to_allocation(current_level, current_level_maximum_leave, leaves_taken, period_start, period_end)
                # if it's the carry-over date, adjust days using current level's carry-over policy, then continue
                if allocation.nextcall == carryover_date:
                    if current_level.action_with_unused_accruals in ['lost', 'maximum']:
                        allocated_days_left = allocation.number_of_days - leaves_taken
                        allocation_max_days = 0 # default if unused_accrual are lost
                        if current_level.action_with_unused_accruals == 'maximum':
                            postpone_max_days = current_level.postpone_max_days if current_level.added_value_type == 'day' else current_level.postpone_max_days / (allocation.employee_id.sudo().resource_id.calendar_id.hours_per_day or HOURS_PER_DAY)
                            allocation_max_days = min(postpone_max_days, allocated_days_left)
                        allocation.number_of_days = min(allocation.number_of_days, allocation_max_days) + leaves_taken

                allocation.lastcall = allocation.nextcall
                allocation.nextcall = nextcall
                allocation.already_accrued = False
                if force_period and allocation.nextcall > date_to:
                    allocation.nextcall = date_to
                    force_period = False

            # if plan.accrued_gain_time == 'start', process next period and set flag 'already_accrued', this will skip adding days
            # once, preventing double allocation.
            if allocation.accrual_plan_id.accrued_gain_time == 'start':
                # check that we are at the start of a period, not on a carry-over or level transition date
                level_start = {level._get_level_transition_date(allocation.date_from): level for level in allocation.accrual_plan_id.level_ids}
                current_level = level_start.get(allocation.lastcall) or current_level or allocation.accrual_plan_id.level_ids[0]
                period_start = current_level._get_previous_date(allocation.lastcall)
                if current_level.cap_accrued_time:
                    current_level_maximum_leave = current_level.maximum_leave if current_level.added_value_type == "day" else current_level.maximum_leave / (allocation.employee_id.sudo().resource_id.calendar_id.hours_per_day or HOURS_PER_DAY)
                allocation._add_days_to_allocation(current_level, current_level_maximum_leave, leaves_taken, period_start, allocation.nextcall)
                allocation.already_accrued = True

    @api.model
    def _update_accrual(self):
        """
        Method called by the cron task in order to increment the number_of_days when
        necessary.
        """
        today = datetime.combine(fields.Date.today(), time(0, 0, 0))
        allocations = self.search([
            ('allocation_type', '=', 'accrual'), ('state', '=', 'validate'),
            ('accrual_plan_id', '!=', False), ('employee_id', '!=', False),
            '|', ('date_to', '=', False), ('date_to', '>', fields.Datetime.now()),
            '|', ('nextcall', '=', False), ('nextcall', '<=', today)])
        allocations._process_accrual_plans()

    def _get_future_leaves_on(self, accrual_date):
        # As computing future accrual allocation days automatically updates the allocation,
        # We need to create a temporary copy of that allocation to return the difference in number of days
        # to see how much more days will be allocated from now until that date.
        self.ensure_one()
        if not accrual_date or accrual_date <= date.today():
            return 0

        if not (self.accrual_plan_id
                and self.state == 'validate'
                and self.allocation_type == 'accrual'
                and (not self.date_to or self.date_to > accrual_date)
                and (not self.nextcall or self.nextcall <= accrual_date)):
            return 0

        fake_allocation = self.env['hr.leave.allocation'].with_context(default_date_from=accrual_date).new(origin=self)
        fake_allocation.sudo().with_context(default_date_from=accrual_date)._process_accrual_plans(accrual_date, log=False)
        if self.holiday_status_id.request_unit in ['hour']:
            res = float_round(fake_allocation.number_of_hours_display - self.number_of_hours_display, precision_digits=2)
        else:
            res = round((fake_allocation.number_of_days - self.number_of_days), 2)
        fake_allocation.invalidate_recordset()
        return res

    ####################################################
    # ORM Overrides methods
    ####################################################

    def onchange(self, values, field_names, fields_spec):
        # Try to force the leave_type display_name when creating new records
        # This is called right after pressing create and returns the display_name for
        # most fields in the view.
        if values and 'employee_id' in fields_spec and 'employee_id' not in self._context:
            employee_id = get_employee_from_context(values, self._context, self.env.user.employee_id.id)
            self = self.with_context(employee_id=employee_id)
        return super().onchange(values, field_names, fields_spec)

    @api.depends(
        'holiday_type', 'mode_company_id', 'department_id',
        'category_id', 'employee_id', 'holiday_status_id',
        'type_request_unit', 'number_of_days',
    )
    def _compute_display_name(self):
        for allocation in self:
            if allocation.holiday_type == 'company':
                target = allocation.mode_company_id.name
            elif allocation.holiday_type == 'department':
                target = allocation.department_id.name
            elif allocation.holiday_type == 'category':
                target = allocation.category_id.name
            elif allocation.employee_id:
                target = allocation.employee_id.name
            elif len(allocation.employee_ids) <= 3:
                target = ', '.join(allocation.employee_ids.sudo().mapped('name'))
            else:
                target = _('%(first)s, %(second)s and %(amount)s others',
                    first=allocation.employee_ids[0].sudo().name,
                    second=allocation.employee_ids[1].sudo().name,
                    amount=len(allocation.employee_ids) - 2)

            allocation.display_name = _("Allocation of %s: %.2f %s to %s",
                allocation.holiday_status_id.sudo().name,
                allocation.number_of_hours_display if allocation.type_request_unit == 'hour' else allocation.number_of_days,
                _('hours') if allocation.type_request_unit == 'hour' else _('days'),
                target,
            )

    def _add_lastcalls(self):
        for allocation in self:
            if allocation.allocation_type != 'accrual':
                continue
            today = fields.Date.today()
            (current_level, current_level_idx) = allocation._get_current_accrual_plan_level_id(today)
            if not allocation.lastcall:
                if not current_level:
                    allocation.lastcall = today
                    continue
                allocation.lastcall = max(
                    current_level._get_previous_date(today),
                    allocation.date_from + get_timedelta(current_level.start_count, current_level.start_type)
                )
            if current_level and not allocation.nextcall:
                accrual_plan = allocation.accrual_plan_id
                allocation.nextcall = current_level._get_next_date(allocation.lastcall)
                if current_level_idx < (len(accrual_plan.level_ids) - 1) and accrual_plan.transition_mode == 'immediately':
                    next_level = accrual_plan.level_ids[current_level_idx + 1]
                    next_level_start = allocation.date_from + get_timedelta(next_level.start_count, next_level.start_type)
                    allocation.nextcall = min(allocation.nextcall, next_level_start)

    def add_follower(self, employee_id):
        employee = self.env['hr.employee'].browse(employee_id)
        if employee.user_id:
            self.message_subscribe(partner_ids=employee.user_id.partner_id.ids)

    @api.model_create_multi
    def create(self, vals_list):
        """ Override to avoid automatic logging of creation """
        for values in vals_list:
            if 'state' in values and values['state'] not in ('draft', 'confirm'):
                raise UserError(_('Incorrect state for new allocation'))
            if values.get('multi_employee', False):
                values['employee_id'] = False
            employee_id = values.get('employee_id', False)
            if not values.get('department_id'):
                values.update({'department_id': self.env['hr.employee'].browse(employee_id).department_id.id})
        allocations = super(HolidaysAllocation, self.with_context(mail_create_nosubscribe=True)).create(vals_list)
        allocations._add_lastcalls()
        for allocation in allocations:
            partners_to_subscribe = set()
            if allocation.employee_id.user_id:
                partners_to_subscribe.add(allocation.employee_id.user_id.partner_id.id)
            if allocation.validation_type == 'officer':
                partners_to_subscribe.add(allocation.employee_id.parent_id.user_id.partner_id.id)
                partners_to_subscribe.add(allocation.employee_id.leave_manager_id.partner_id.id)
            allocation.message_subscribe(partner_ids=tuple(partners_to_subscribe))
            if not self._context.get('import_file'):
                allocation.activity_update()
            if allocation.validation_type == 'no' and allocation.state == 'confirm':
                allocation.action_validate()
        return allocations

    def write(self, values):
        if not self.env.context.get('toggle_active') and not bool(values.get('active', True)):
            if any(allocation.state not in ['refuse'] for allocation in self):
                raise UserError(_('You cannot archive an allocation which is in confirm or validate state.'))
        employee_id = values.get('employee_id', False)
        if values.get('state'):
            self._check_approval_update(values['state'])

        self.add_follower(employee_id)

        if 'number_of_days_display' in values\
                or 'number_of_hours_display' in values\
                or 'number_of_days' in values:
            previous_consumed_leaves = self.employee_id._get_consumed_leaves(leave_types=self.holiday_status_id)
            result = super().write(values)
            if 'allocation_type' in values:
                self._add_lastcalls()
            consumed_leaves = self.employee_id._get_consumed_leaves(leave_types=self.holiday_status_id)
            for allocation in self:
                current_excess = dict(consumed_leaves[1]).get(allocation.employee_id, {}) \
                    .get(allocation.holiday_status_id, {}).get('excess_days', {})
                previous_excess = dict(previous_consumed_leaves[1]).get(allocation.employee_id, {}) \
                    .get(allocation.holiday_status_id, {}).get('excess_days', {})
                total_current_excess = sum(leave_date['amount'] and not leave_date['is_virtual'] for leave_date in current_excess.values())
                total_previous_excess = sum(leave_date['amount'] and not leave_date['is_virtual'] for leave_date in previous_excess.values())

                if total_current_excess <= total_previous_excess:
                    continue
                lt = allocation.holiday_status_id
                if lt.allows_negative and total_current_excess <= lt.max_allowed_negative:
                    continue
                raise ValidationError(
                    _('You cannot reduce the duration below the duration of leaves already taken by the employee.'))
        else:
            result = super().write(values)
            if 'allocation_type' in values:
                self._add_lastcalls()

        if not self.linked_request_ids:
            return result
        write_vals = {
            field: values.get(field)
            for field in [
                'private_name',
                'holiday_type',
                'holiday_status_id',
                'notes',
                'number_of_days',
                'allocation_type',
                'date_from',
                'date_to',
                'accrual_plan_id',
            ] if field in values
        }
        if write_vals:
            self.linked_request_ids.write(write_vals)
        return result

    @api.ondelete(at_uninstall=False)
    def _unlink_if_correct_states(self):
        state_description_values = {elem[0]: elem[1] for elem in self._fields['state']._description_selection(self.env)}
        for allocation in self.filtered(lambda allocation: allocation.state not in ['confirm', 'refuse']):
            raise UserError(_('You cannot delete an allocation request which is in %s state.', state_description_values.get(allocation.state)))

    @api.ondelete(at_uninstall=False)
    def _unlink_if_no_leaves(self):
        if any(allocation.holiday_status_id.requires_allocation == 'yes' and allocation.leaves_taken > 0 for allocation in self):
            raise UserError(_('You cannot delete an allocation request which has some validated leaves.'))

    def _get_mail_redirect_suggested_company(self):
        return self.holiday_status_id.company_id

    ####################################################
    # Business methods
    ####################################################

    def _prepare_holiday_values(self, employees):
        self.ensure_one()
        return [{
            'name': self.name,
            'holiday_type': 'employee',
            'holiday_status_id': self.holiday_status_id.id,
            'notes': self.notes,
            'number_of_days': self.number_of_days,
            'parent_id': self.id,
            'employee_id': employee.id,
            'employee_ids': [(6, 0, [employee.id])],
            'state': 'confirm',
            'allocation_type': self.allocation_type,
            'date_from': self.date_from,
            'date_to': self.date_to,
            'accrual_plan_id': self.accrual_plan_id.id,
        } for employee in employees if (not employee.resource_calendar_id) or employee.resource_calendar_id.hours_per_day]

    def action_validate(self):
        to_validate = self.filtered(lambda alloc: alloc.state != 'validate')
        if to_validate:
            to_validate.write({
                'state': 'validate',
                'approver_id': self.env.user.employee_id.id
            })
            to_validate._action_validate_create_childs()
            to_validate.activity_update()
        return True

    def _action_validate_create_childs(self):
        allocation_vals = []
        for allocation in self:
            # In the case we are in holiday_type `employee` and there is only one employee we can keep the same allocation
            # Otherwise we do need to create an allocation for all employees to have a behaviour that is in line
            # with the other holiday_type
            if allocation.linked_request_ids:
                continue
            if allocation.state == 'validate' and (allocation.holiday_type in ['category', 'department', 'company'] or
                (allocation.holiday_type == 'employee' and len(allocation.employee_ids) > 1)):
                if allocation.holiday_type == 'employee':
                    employees = allocation.employee_ids
                elif allocation.holiday_type == 'category':
                    employees = allocation.category_id.employee_ids
                elif allocation.holiday_type == 'department':
                    employees = allocation.department_id.member_ids
                else:
                    employees = self.env['hr.employee'].search([('company_id', '=', allocation.mode_company_id.id)])

                allocation_vals += allocation._prepare_holiday_values(employees)
        if allocation_vals:
            self.env['hr.leave.allocation'].with_context(
                mail_notify_force_send=False,
                mail_activity_automation_skip=True
            ).create(allocation_vals)
        self.linked_request_ids.filtered(lambda c: c.state != 'validate').action_validate()

    def action_refuse(self):
        current_employee = self.env.user.employee_id
        if any(allocation.state not in ['confirm', 'validate'] for allocation in self):
            raise UserError(_('Allocation request must be confirmed or validated in order to refuse it.'))

        days_per_allocation = self.employee_id._get_consumed_leaves(self.holiday_status_id)[0]

        for allocation in self:
            days_taken = days_per_allocation[allocation.employee_id][allocation.holiday_status_id][allocation]['virtual_leaves_taken']
            if days_taken > 0:
                raise UserError(_('You cannot refuse this allocation request since the employee has already taken leaves for it. Please refuse or delete those leaves first.'))
        self.write({'state': 'refuse', 'approver_id': current_employee.id})
        # If a category that created several holidays, cancel all related
        linked_requests = self.mapped('linked_request_ids')
        if linked_requests:
            linked_requests.action_refuse()
        self.activity_update()
        return True

    def _check_approval_update(self, state):
        """ Check if target state is achievable. """
        if self.env.is_superuser():
            return
        current_employee = self.env.user.employee_id
        if not current_employee:
            return
        is_officer = self.env.user.has_group('hr_holidays.group_hr_holidays_user')
        is_manager = self.env.user.has_group('hr_holidays.group_hr_holidays_manager')
        for allocation in self:
            val_type = allocation.holiday_status_id.sudo().allocation_validation_type
            if state == 'confirm':
                continue

            if not is_officer and self.env.user != allocation.employee_id.leave_manager_id and not val_type == 'no':
                raise UserError(_('Only a time off Officer/Responsible or Manager can approve or refuse time off requests.'))

            if is_officer or self.env.user == allocation.employee_id.leave_manager_id:
                # use ir.rule based first access check: department, members, ... (see security.xml)
                allocation.check_access_rule('write')

            if allocation.employee_id == current_employee and not is_manager and not val_type == 'no':
                raise UserError(_('Only a time off Manager can approve its own requests.'))

    @api.onchange('allocation_type')
    def _onchange_allocation_type(self):
        if self.allocation_type == 'accrual':
            self.number_of_days = 0.0
        elif not self.number_of_days_display:
            self.number_of_days = 1.0

    # Allows user to simulate how many days an accrual plan would give from a certain start date.
    # it uses the actual computation function but resets values of lastcall, nextcall and nbr of days
    # before every run, as if it was run from date_from, after an optional change in the allocation value
    # the user can simply confirm and validate the allocation. The record is in correct state for the next
    # call of the cron job.
    @api.onchange('date_from', 'accrual_plan_id', 'date_to', 'employee_id')
    def _onchange_date_from(self):
        if not self.employee_id and self.allocation_type == 'accrual':
            self.number_of_days = 0
        if not self.date_from or self.allocation_type != 'accrual' or self.state == 'validate' or not self.accrual_plan_id\
            or not self.employee_id:
            return
        self.lastcall = self.date_from
        self.nextcall = False
        self.number_of_days_display = 0.0
        self.number_of_hours_display = 0.0
        self.number_of_days = 0.0
        date_to = min(self.date_to, date.today()) if self.date_to else False
        self._process_accrual_plans(date_to)

    # ------------------------------------------------------------
    # Activity methods
    # ------------------------------------------------------------

    def _get_responsible_for_approval(self):
        self.ensure_one()
        responsible = self.env.user

        if self.validation_type == 'officer' or self.validation_type == 'set':
            if self.holiday_status_id.responsible_ids:
                responsible = self.holiday_status_id.responsible_ids
        return responsible

    def activity_update(self):
        to_clean, to_do = self.env['hr.leave.allocation'], self.env['hr.leave.allocation']
        activity_vals = []
        for allocation in self:
            if allocation.validation_type != 'no':
                note = _(
                    'New Allocation Request created by %(user)s: %(count)s Days of %(allocation_type)s',
                    user=allocation.create_uid.name,
                    count=allocation.number_of_days,
                    allocation_type=allocation.holiday_status_id.name
                )
                if allocation.state == 'confirm':
                    if allocation.holiday_status_id.responsible_ids:
                        user_ids = allocation.sudo()._get_responsible_for_approval().ids
                        for user_id in user_ids:
                            activity_vals.append({
                                'activity_type_id': self.env.ref('hr_holidays.mail_act_leave_allocation_approval').id,
                                'automated': True,
                                'note': note,
                                'user_id': user_id,
                                'res_id': allocation.id,
                                'res_model_id': self.env.ref('hr_holidays.model_hr_leave_allocation').id,
                            })
                elif allocation.state == 'validate':
                    to_do |= allocation
                elif allocation.state == 'refuse':
                    to_clean |= allocation
        if activity_vals:
            self.env['mail.activity'].create(activity_vals)
        if to_clean:
            to_clean.activity_unlink(['hr_holidays.mail_act_leave_allocation_approval'])
        if to_do:
            to_do.activity_feedback(['hr_holidays.mail_act_leave_allocation_approval'])

    ####################################################
    # Messaging methods
    ####################################################

    def _track_subtype(self, init_values):
        if 'state' in init_values and self.state == 'validate':
            allocation_notif_subtype_id = self.holiday_status_id.allocation_notif_subtype_id
            return allocation_notif_subtype_id or self.env.ref('hr_holidays.mt_leave_allocation')
        return super(HolidaysAllocation, self)._track_subtype(init_values)

    def _notify_get_recipients_groups(self, message, model_description, msg_vals=None):
        """ Handle HR users and officers recipients that can validate or refuse holidays
        directly from email. """
        groups = super()._notify_get_recipients_groups(
            message, model_description, msg_vals=msg_vals
        )
        if not self:
            return groups

        local_msg_vals = dict(msg_vals or {})

        self.ensure_one()
        hr_actions = []
        if self.state == 'confirm':
            app_action = self._notify_get_action_link('controller', controller='/allocation/validate', **local_msg_vals)
            hr_actions += [{'url': app_action, 'title': _('Approve')}]
        if self.state in ['confirm', 'validate']:
            ref_action = self._notify_get_action_link('controller', controller='/allocation/refuse', **local_msg_vals)
            hr_actions += [{'url': ref_action, 'title': _('Refuse')}]

        holiday_user_group_id = self.env.ref('hr_holidays.group_hr_holidays_user').id
        new_group = (
            'group_hr_holidays_user',
            lambda pdata: pdata['type'] == 'user' and holiday_user_group_id in pdata['groups'],
            {
                'actions': hr_actions,
                'active': True,
                'has_button_access': True,
            }
        )

        return [new_group] + groups

    def message_subscribe(self, partner_ids=None, subtype_ids=None):
        # due to record rule can not allow to add follower and mention on validated leave so subscribe through sudo
        if any(state in ['validate'] for state in self.mapped('state')):
            self.check_access_rights('read')
            self.check_access_rule('read')
            return super(HolidaysAllocation, self.sudo()).message_subscribe(partner_ids=partner_ids, subtype_ids=subtype_ids)
        return super().message_subscribe(partner_ids=partner_ids, subtype_ids=subtype_ids)

```

## File: models\hr_leave_mandatory_day.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from random import randint

from odoo import fields, models


class MandatoryDay(models.Model):
    _name = 'hr.leave.mandatory.day'
    _description = 'Mandatory Day'
    _order = 'start_date desc, end_date desc'

    name = fields.Char(required=True)
    company_id = fields.Many2one('res.company', default=lambda self: self.env.company, required=True)
    start_date = fields.Date(required=True)
    end_date = fields.Date(required=True)
    color = fields.Integer(default=lambda dummy: randint(1, 11))
    resource_calendar_id = fields.Many2one(
        'resource.calendar', 'Working Hours',
        domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]")
    department_ids = fields.Many2many('hr.department', string="Departments")

    _sql_constraints = [
        ('date_from_after_day_to', 'CHECK(start_date <= end_date)', 'The start date must be anterior than the end date.')
    ]

```

## File: models\hr_leave_type.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2005-2006 Axelor SARL. (http://www.axelor.com)

import logging
import pytz

from collections import defaultdict
from datetime import date, datetime, time

from odoo import api, fields, models
from odoo.exceptions import UserError
from odoo.tools import format_date, frozendict
from odoo.tools.translate import _
from odoo.tools.float_utils import float_round

_logger = logging.getLogger(__name__)


class HolidaysType(models.Model):
    _name = "hr.leave.type"
    _description = "Time Off Type"
    _order = 'sequence'

    @api.model
    def _model_sorting_key(self, leave_type):
        remaining = leave_type.virtual_remaining_leaves > 0
        taken = leave_type.leaves_taken > 0
        return -1 * leave_type.sequence, leave_type.employee_requests == 'no' and remaining, leave_type.employee_requests == 'yes' and remaining, taken

    name = fields.Char('Time Off Type', required=True, translate=True)
    sequence = fields.Integer(default=100,
        help='The type with the smallest sequence is the default value in time off request')
    create_calendar_meeting = fields.Boolean(string="Display Time Off in Calendar", default=True)
    color = fields.Integer(string='Color', help="The color selected here will be used in every screen with the time off type.")
    icon_id = fields.Many2one('ir.attachment', string='Cover Image', domain="[('res_model', '=', 'hr.leave.type'), ('res_field', '=', 'icon_id')]")
    active = fields.Boolean('Active', default=True,
                            help="If the active field is set to false, it will allow you to hide the time off type without removing it.")

    # employee specific computed data
    max_leaves = fields.Float(compute='_compute_leaves', string='Maximum Allowed', search='_search_max_leaves',
        help='This value is given by the sum of all time off requests with a positive value.')
    leaves_taken = fields.Float(
        compute='_compute_leaves', string='Time off Already Taken',
        help='This value is given by the sum of all time off requests with a negative value.')
    virtual_remaining_leaves = fields.Float(
        compute='_compute_leaves', search='_search_virtual_remaining_leaves', string='Virtual Remaining Time Off',
        help='Maximum Time Off Allowed - Time Off Already Taken - Time Off Waiting Approval')

    allocation_count = fields.Integer(
        compute='_compute_allocation_count', string='Allocations')
    group_days_leave = fields.Float(
        compute='_compute_group_days_leave', string='Group Time Off')
    company_id = fields.Many2one('res.company', string='Company', default=lambda self: self.env.company)
    responsible_ids = fields.Many2many(
        'res.users', 'hr_leave_type_res_users_rel', 'hr_leave_type_id', 'res_users_id', string='Notified Time Off Officer',
        domain=lambda self: [('groups_id', 'in', self.env.ref('hr_holidays.group_hr_holidays_user').id),
                             ('share', '=', False),
                             ('company_ids', 'in', self.env.company.id)],
                             auto_join=True,
        help="Choose the Time Off Officers who will be notified to approve allocation or Time Off Request. If empty, nobody will be notified")
    leave_validation_type = fields.Selection([
        ('no_validation', 'No Validation'),
        ('hr', 'By Time Off Officer'),
        ('manager', "By Employee's Approver"),
        ('both', "By Employee's Approver and Time Off Officer")], default='hr', string='Time Off Validation')
    requires_allocation = fields.Selection([
        ('yes', 'Yes'),
        ('no', 'No Limit')], default="yes", required=True, string='Requires allocation',
        help="""Yes: Time off requests need to have a valid allocation.\n
              No Limit: Time Off requests can be taken without any prior allocation.""")
    employee_requests = fields.Selection([
        ('yes', 'Extra Days Requests Allowed'),
        ('no', 'Not Allowed')], default="no", required=True, string="Employee Requests",
        help="""Extra Days Requests Allowed: User can request an allocation for himself.\n
        Not Allowed: User cannot request an allocation.""")
    allocation_validation_type = fields.Selection([
        ('officer', 'Approved by Time Off Officer'),
        ('no', 'No validation needed')], default='no', string='Approval',
        compute='_compute_allocation_validation_type', store=True, readonly=False,
        help="""Select the level of approval needed in case of request by employee
        - No validation needed: The employee's request is automatically approved.
        - Approved by Time Off Officer: The employee's request need to be manually approved by the Time Off Officer.""")
    has_valid_allocation = fields.Boolean(compute='_compute_valid', search='_search_valid', help='This indicates if it is still possible to use this type of leave')
    time_type = fields.Selection([('other', 'Worked Time'), ('leave', 'Absence')], default='leave', string="Kind of Time Off",
                                 help="The distinction between working time (ex. Attendance) and absence (ex. Training) will be used in the computation of Accrual's plan rate.")
    request_unit = fields.Selection([
        ('day', 'Day'),
        ('half_day', 'Half Day'),
        ('hour', 'Hours')], default='day', string='Take Time Off in', required=True)
    unpaid = fields.Boolean('Is Unpaid', default=False)
    leave_notif_subtype_id = fields.Many2one('mail.message.subtype', string='Time Off Notification Subtype', default=lambda self: self.env.ref('hr_holidays.mt_leave', raise_if_not_found=False))
    allocation_notif_subtype_id = fields.Many2one('mail.message.subtype', string='Allocation Notification Subtype', default=lambda self: self.env.ref('hr_holidays.mt_leave_allocation', raise_if_not_found=False))
    support_document = fields.Boolean(string='Supporting Document')
    accruals_ids = fields.One2many('hr.leave.accrual.plan', 'time_off_type_id')
    accrual_count = fields.Float(compute="_compute_accrual_count", string="Accruals count")
    # negative time off
    allows_negative = fields.Boolean(string='Allow Negative Cap',
        help="If checked, users request can exceed the allocated days and balance can go in negative.")
    max_allowed_negative = fields.Integer(string="Amount in Negative",
        help="Define the maximum level of negative days this kind of time off can reach. Value must be at least 1.")

    _sql_constraints = [(
        'check_negative',
        'CHECK(NOT allows_negative OR max_allowed_negative > 0)',
        'The negative amount must be greater than 0. If you want to set 0, disable the negative cap instead.'
    )]

    @api.model
    def _search_valid(self, operator, value):
        """ Returns leave_type ids for which a valid allocation exists
            or that don't need an allocation
            return [('id', domain_operator, [x['id'] for x in res])]
        """

        if {'default_date_from', 'default_date_to', 'tz'} <= set(self._context):
            default_date_from_dt = fields.Datetime.to_datetime(self._context.get('default_date_from'))
            default_date_to_dt = fields.Datetime.to_datetime(self._context.get('default_date_to'))

            # Cast: Datetime -> Date using user's tz
            date_from = fields.Date.context_today(self, default_date_from_dt)
            date_to = fields.Date.context_today(self, default_date_to_dt)

        else:
            date_from = fields.Date.today().strftime('%Y-1-1')
            date_to = fields.Date.today().strftime('%Y-12-31')

        employee_id = self._context.get('default_employee_id', self._context.get('employee_id')) or self.env.user.employee_id.id

        if not isinstance(value, bool):
            raise ValueError('Invalid value: %s' % (value))
        if operator not in ['=', '!=']:
            raise ValueError('Invalid operator: %s' % (operator))
        # '!=' True or '=' False
        if (operator == '=') ^ value:
            new_operator = 'not in'
        # '=' True or '!=' False
        else:
            new_operator = 'in'

        leave_types = self.env['hr.leave.allocation'].search([
            ('employee_id', '=', employee_id),
            ('state', '=', 'validate'),
            ('date_from', '<=', date_to),
            '|',
            ('date_to', '>=', date_from),
            ('date_to', '=', False),
        ]).holiday_status_id

        return [('id', new_operator, leave_types.ids)]

    @api.depends('requires_allocation', 'max_leaves', 'virtual_remaining_leaves')
    def _compute_valid(self):
        date_from = self._context.get('default_date_from', fields.Datetime.today())
        date_to = self._context.get('default_date_to', fields.Datetime.today())
        employee_id = self._context.get('default_employee_id', self._context.get('employee_id', self.env.user.employee_id.id))
        for holiday_type in self:
            if holiday_type.requires_allocation == 'yes':
                allocations = self.env['hr.leave.allocation'].search([
                    ('holiday_status_id', '=', holiday_type.id),
                    ('allocation_type', '=', 'accrual'),
                    ('employee_id', '=', employee_id),
                    ('date_from', '<=', date_from),
                    '|',
                    ('date_to', '>=', date_to),
                    ('date_to', '=', False),
                ])
                allowed_excess = holiday_type.max_allowed_negative if holiday_type.allows_negative else 0
                allocations = allocations.filtered(lambda alloc:
                    alloc.allocation_type == 'accrual'
                    or (alloc.max_leaves > 0 and alloc.virtual_remaining_leaves > -allowed_excess)
                )
                holiday_type.has_valid_allocation = bool(allocations)
            else:
                holiday_type.has_valid_allocation = True

    def _load_records_write(self, values):
        if 'requires_allocation' in values and self.requires_allocation == values['requires_allocation']:
            values.pop('requires_allocation')
        return super()._load_records_write(values)

    @api.constrains('requires_allocation')
    def check_allocation_requirement_edit_validity(self):
        if self.env['hr.leave'].search_count([('holiday_status_id', 'in', self.ids)], limit=1):
            raise UserError(_("The allocation requirement of a time off type cannot be changed once leaves of that type have been taken. You should create a new time off type instead."))

    def _search_max_leaves(self, operator, value):
        value = float(value)
        employee = self.env['hr.employee']._get_contextual_employee()
        leaves = defaultdict(int)

        if employee:
            allocations = self.env['hr.leave.allocation'].search([
                ('employee_id', '=', employee.id),
                ('state', '=', 'validate')
            ])
            for allocation in allocations:
                leaves[allocation.holiday_status_id.id] += allocation.number_of_days
        valid_leave = []
        for leave in leaves:
            if operator == '>':
                if leaves[leave] > value:
                    valid_leave.append(leave)
            elif operator == '<':
                if leaves[leave] < value:
                    valid_leave.append(leave)
            elif operator == '=':
                if leaves[leave] == value:
                    valid_leave.append(leave)
            elif operator == '!=':
                if leaves[leave] != value:
                    valid_leave.append(leave)

        return [('id', 'in', valid_leave)]

    def _search_virtual_remaining_leaves(self, operator, value):
        value = float(value)
        leave_types = self.env['hr.leave.type'].search([])
        valid_leave_types = self.env['hr.leave.type']

        for leave_type in leave_types:
            if leave_type.requires_allocation == "yes":
                if operator == '>' and leave_type.virtual_remaining_leaves > value:
                    valid_leave_types |= leave_type
                elif operator == '<' and leave_type.virtual_remaining_leaves < value:
                    valid_leave_types |= leave_type
                elif operator == '>=' and leave_type.virtual_remaining_leaves >= value:
                    valid_leave_types |= leave_type
                elif operator == '<=' and leave_type.virtual_remaining_leaves <= value:
                    valid_leave_types |= leave_type
                elif operator == '=' and leave_type.virtual_remaining_leaves == value:
                    valid_leave_types |= leave_type
                elif operator == '!=' and leave_type.virtual_remaining_leaves != value:
                    valid_leave_types |= leave_type
            else:
                valid_leave_types |= leave_type

        return [('id', 'in', valid_leave_types.ids)]

    @api.depends_context('employee_id', 'default_employee_id', 'default_date_from')
    def _compute_leaves(self):
        employee = self.env['hr.employee']._get_contextual_employee()
        target_date = self._context['default_date_from'] if 'default_date_from' in self._context else None
        # This is a workaround to save the date value in context for next triggers
        # when context gets cleaned and 'default_' context keys gets removed
        if target_date:
            self.env.context = frozendict(self.env.context, leave_date_from=self._context['default_date_from'])
        else:
            target_date = self._context.get('leave_date_from', None)
        data_days = self.get_allocation_data(employee, target_date)[employee]
        for holiday_status in self:
            result = [item for item in data_days if item[0] == holiday_status.name]
            leave_type_tuple = result[0] if result else ('', {})
            holiday_status.max_leaves = leave_type_tuple[1].get('max_leaves', 0)
            holiday_status.leaves_taken = leave_type_tuple[1].get('leaves_taken', 0)
            holiday_status.virtual_remaining_leaves = leave_type_tuple[1].get('virtual_remaining_leaves', 0)

    def _compute_allocation_count(self):
        min_datetime = fields.Datetime.to_string(datetime.now().replace(month=1, day=1, hour=0, minute=0, second=0, microsecond=0))
        max_datetime = fields.Datetime.to_string(datetime.now().replace(month=12, day=31, hour=23, minute=59, second=59))
        domain = [
            ('holiday_status_id', 'in', self.ids),
            ('date_from', '>=', min_datetime),
            ('date_from', '<=', max_datetime),
            ('state', 'in', ('confirm', 'validate')),
        ]

        grouped_res = self.env['hr.leave.allocation']._read_group(
            domain,
            ['holiday_status_id'],
            ['__count'],
        )
        grouped_dict = {holiday_status.id: count for holiday_status, count in grouped_res}
        for allocation in self:
            allocation.allocation_count = grouped_dict.get(allocation.id, 0)

    def _compute_group_days_leave(self):
        min_datetime = fields.Datetime.to_string(datetime.now().replace(month=1, day=1, hour=0, minute=0, second=0, microsecond=0))
        max_datetime = fields.Datetime.to_string(datetime.now().replace(month=12, day=31, hour=23, minute=59, second=59))
        domain = [
            ('holiday_status_id', 'in', self.ids),
            ('date_from', '>=', min_datetime),
            ('date_from', '<=', max_datetime),
            ('state', 'in', ('validate', 'validate1', 'confirm')),
        ]
        grouped_res = self.env['hr.leave']._read_group(
            domain,
            ['holiday_status_id'],
            ['__count'],
        )
        grouped_dict = {holiday_status.id: count for holiday_status, count in grouped_res}
        for allocation in self:
            allocation.group_days_leave = grouped_dict.get(allocation.id, 0)

    def _compute_accrual_count(self):
        accrual_allocations = self.env['hr.leave.accrual.plan']._read_group([('time_off_type_id', 'in', self.ids)], ['time_off_type_id'], ['__count'])
        mapped_data = {time_off_type.id: count for time_off_type, count in accrual_allocations}
        for leave_type in self:
            leave_type.accrual_count = mapped_data.get(leave_type.id, 0)

    @api.depends('employee_requests')
    def _compute_allocation_validation_type(self):
        for leave_type in self:
            if leave_type.employee_requests == 'no':
                leave_type.allocation_validation_type = 'officer'

    def requested_display_name(self):
        return self._context.get('holiday_status_display_name', True) and self._context.get('employee_id')

    @api.depends('requires_allocation', 'virtual_remaining_leaves', 'max_leaves', 'request_unit')
    @api.depends_context('holiday_status_display_name', 'employee_id', 'from_manager_leave_form')
    def _compute_display_name(self):
        if not self.requested_display_name():
            # leave counts is based on employee_id, would be inaccurate if not based on correct employee
            return super()._compute_display_name()
        for record in self:
            name = record.name
            if record.requires_allocation == "yes" and not self._context.get('from_manager_leave_form'):
                name = "{name} ({count})".format(
                    name=name,
                    count=_('%g remaining out of %g') % (
                        float_round(record.virtual_remaining_leaves, precision_digits=2) or 0.0,
                        float_round(record.max_leaves, precision_digits=2) or 0.0,
                    ) + (_(' hours') if record.request_unit == 'hour' else _(' days')),
            )
            record.display_name = name

    @api.model
    def _search(self, domain, offset=0, limit=None, order=None, access_rights_uid=None):
        """ Override _search to order the results, according to some employee.
        The order is the following

         - allocation fixed first, then allowing allocation, then free allocation
         - virtual remaining leaves (higher the better, so using reverse on sorted)

        This override is necessary because those fields are not stored and depends
        on an employee_id given in context. This sort will be done when there
        is an employee_id in context and that no other order has been given
        to the method.
        """
        employee = self.env['hr.employee']._get_contextual_employee()
        if order == self._order and employee:
            # retrieve all leaves, sort them, then apply offset and limit
            leaves = self.browse(super()._search(domain, access_rights_uid=access_rights_uid))
            leaves = leaves.sorted(key=self._model_sorting_key, reverse=True)
            leaves = leaves[offset:(offset + limit) if limit else None]
            return leaves._as_query()
        return super()._search(domain, offset, limit, order, access_rights_uid)

    def action_see_days_allocated(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("hr_holidays.hr_leave_allocation_action_all")
        action['domain'] = [
            ('holiday_status_id', 'in', self.ids),
        ]
        action['context'] = {
            'default_holiday_type': 'department',
            'default_holiday_status_id': self.ids[0],
            'search_default_approved_state': 1,
            'search_default_year': 1,
        }
        return action

    def action_see_group_leaves(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("hr_holidays.hr_leave_action_action_approve_department")
        action['domain'] = [
            ('holiday_status_id', '=', self.ids[0]),
        ]
        action['context'] = {
            'default_holiday_status_id': self.ids[0],
        }
        return action

    def action_see_accrual_plans(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("hr_holidays.open_view_accrual_plans")
        action['domain'] = [
            ('time_off_type_id', '=', self.id),
        ]
        action['context'] = {
            'default_time_off_type_id': self.id,
        }
        return action

    # ------------------------------------------------------------
    # Leave - Allocation link methods
    # ------------------------------------------------------------

    @api.model
    def has_accrual_allocation(self):
        employee = self.env['hr.employee']._get_contextual_employee()
        if not employee:
            return False
        return bool(self.env['hr.leave.allocation'].search_count([
            ('employee_id', '=', employee.id),
            ('state', '=', 'validate'),
            ('allocation_type', '=', 'accrual'),
            '|',
            ('date_to', '>', date.today()),
            ('date_to', '=', False),
        ]))

    @api.model
    def get_allocation_data_request(self, target_date=None):
        leave_types = self.search([
            '|',
            ('company_id', 'in', self.env.context.get('allowed_company_ids')),
            ('company_id', '=', False),
        ], order='id')
        employee = self.env['hr.employee']._get_contextual_employee()
        if employee:
            return leave_types.get_allocation_data(employee, target_date)[employee]
        return []

    def get_allocation_data(self, employees, target_date=None):
        allocation_data = defaultdict(list)
        if target_date and isinstance(target_date, str):
            target_date = datetime.fromisoformat(target_date).date()
        elif target_date and isinstance(target_date, datetime):
            target_date = target_date.date()
        elif not target_date:
            target_date = fields.Date.today()

        allocations_leaves_consumed, extra_data = employees.with_context(
            ignored_leave_ids=self.env.context.get('ignored_leave_ids')
        )._get_consumed_leaves(self, target_date)
        leave_type_requires_allocation = self.filtered(lambda lt: lt.requires_allocation == 'yes')

        for employee in employees:
            for leave_type in leave_type_requires_allocation:
                if len(allocations_leaves_consumed[employee][leave_type]) == 0:
                    continue
                lt_info = (
                    leave_type.name,
                    {
                        'remaining_leaves': 0,
                        'virtual_remaining_leaves': 0,
                        'max_leaves': 0,
                        'accrual_bonus': 0,
                        'leaves_taken': 0,
                        'virtual_leaves_taken': 0,
                        'leaves_requested': 0,
                        'leaves_approved': 0,
                        'closest_allocation_remaining': 0,
                        'closest_allocation_expire': False,
                        'holds_changes': False,
                        'total_virtual_excess': 0,
                        'virtual_excess_data': {},
                        'exceeding_duration': extra_data[employee][leave_type]['exceeding_duration'],
                        'request_unit': leave_type.request_unit,
                        'icon': leave_type.sudo().icon_id.url,
                        'allows_negative': leave_type.allows_negative,
                        'max_allowed_negative': leave_type.max_allowed_negative,
                    },
                    leave_type.requires_allocation,
                    leave_type.id)
                for excess_date, excess_days in extra_data[employee][leave_type]['excess_days'].items():
                    amount = excess_days['amount']
                    lt_info[1]['virtual_excess_data'].update({
                        excess_date.strftime('%Y-%m-%d'): excess_days
                    }),
                    lt_info[1]['total_virtual_excess'] += amount
                    if not leave_type.allows_negative:
                        continue
                    lt_info[1]['virtual_leaves_taken'] += amount
                    lt_info[1]['virtual_remaining_leaves'] -= amount
                    if excess_days['is_virtual']:
                        lt_info[1]['leaves_requested'] += amount
                    else:
                        lt_info[1]['leaves_approved'] += amount
                        lt_info[1]['leaves_taken'] += amount
                        lt_info[1]['remaining_leaves'] -= amount
                allocations_now = self.env['hr.leave.allocation']
                allocations_date = self.env['hr.leave.allocation']
                allocations_with_remaining_leaves = self.env['hr.leave.allocation']
                for allocation, data in allocations_leaves_consumed[employee][leave_type].items():
                    # We only need the allocation that are valid at the given date
                    if allocation:
                        today = fields.Date.today()
                        if allocation.date_from <= today and (not allocation.date_to or allocation.date_to >= today):
                            # we get each allocation available now to indicate visually if
                            # the future evaluation holds changes compared to now
                            allocations_now |= allocation
                        if allocation.date_from <= target_date and (not allocation.date_to or allocation.date_to >= target_date):
                            # we get each allocation available now to indicate visually if
                            # the future evaluation holds changes compared to now
                            allocations_date |= allocation
                        if allocation.date_from > target_date:
                            continue
                        if allocation.date_to and allocation.date_to < target_date:
                            continue
                    lt_info[1]['remaining_leaves'] += data['remaining_leaves']
                    lt_info[1]['virtual_remaining_leaves'] += data['virtual_remaining_leaves']
                    lt_info[1]['max_leaves'] += data['max_leaves']
                    lt_info[1]['accrual_bonus'] += data['accrual_bonus']
                    lt_info[1]['leaves_taken'] += data['leaves_taken']
                    lt_info[1]['virtual_leaves_taken'] += data['virtual_leaves_taken']
                    lt_info[1]['leaves_requested'] += data['virtual_leaves_taken'] - data['leaves_taken']
                    lt_info[1]['leaves_approved'] += data['leaves_taken']
                    if data['virtual_remaining_leaves'] > 0:
                        allocations_with_remaining_leaves |= allocation
                closest_allocation = allocations_with_remaining_leaves[0] if allocations_with_remaining_leaves else self.env['hr.leave.allocation']
                closest_allocations = allocations_with_remaining_leaves.filtered(lambda a: a.date_to == closest_allocation.date_to)
                closest_allocation_remaining = 0
                for closest_allocation in closest_allocations:
                    closest_allocation_remaining += allocations_leaves_consumed[employee][leave_type][closest_allocation]['virtual_remaining_leaves']
                if closest_allocation.date_to:
                    closest_allocation_expire = format_date(self.env, closest_allocation.date_to)
                    calendar = employee.resource_calendar_id\
                               or employee.company_id.resource_calendar_id
                    # closest_allocation_duration corresponds to the time remaining before the allocation expires
                    closest_allocation_duration =\
                        calendar._attendance_intervals_batch(
                            datetime.combine(closest_allocation.date_to, time.min).replace(tzinfo=pytz.UTC),
                            datetime.combine(target_date, time.max).replace(tzinfo=pytz.UTC))\
                        if leave_type.request_unit in ['hour']\
                        else (closest_allocation.date_to - target_date).days + 1
                else:
                    closest_allocation_expire = False
                    closest_allocation_duration = False
                # the allocations are assumed to be different from today's allocations if there is any
                # accrual days granted or if there is any difference between allocations now and on the selected date
                holds_changes = (lt_info[1]['accrual_bonus'] > 0
                    or bool(allocations_date - allocations_now)
                    or bool(allocations_now - allocations_date))\
                    and target_date != fields.Date.today()
                lt_info[1].update({
                    'closest_allocation_remaining': closest_allocation_remaining,
                    'closest_allocation_expire': closest_allocation_expire,
                    'closest_allocation_duration': closest_allocation_duration,
                    'holds_changes': holds_changes,
                })
                if not self.env.context.get('from_dashboard', False) or lt_info[1]['max_leaves']:
                    allocation_data[employee].append(lt_info)
        for employee in allocation_data:
            for leave_type_data in allocation_data[employee]:
                for key, value in leave_type_data[1].items():
                    if isinstance(value, float):
                        leave_type_data[1][key] = round(value, 2)
        return allocation_data

```

## File: models\mail_message_subtype.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, models

_logger = logging.getLogger(__name__)


class MailMessageSubtype(models.Model):
    _inherit = 'mail.message.subtype'

    def _get_department_subtype(self):
        return self.search([
            ('res_model', '=', 'hr.department'),
            ('parent_id', '=', self.id)])

    def _update_department_subtype(self):
        for subtype in self:
            department_subtype = subtype._get_department_subtype()
            if department_subtype:
                department_subtype.write({
                    'name': subtype.name,
                    'default': subtype.default,
                })
            else:
                department_subtype = self.create({
                    'name': subtype.name,
                    'res_model': 'hr.department',
                    'default': subtype.default or False,
                    'parent_id': subtype.id,
                    'relation_field': 'department_id',
                })
            return department_subtype

    @api.model_create_multi
    def create(self, vals_list):
        result = super(MailMessageSubtype, self).create(vals_list)
        result.filtered(
            lambda st: st.res_model in ['hr.leave', 'hr.leave.allocation']
        )._update_department_subtype()
        return result

    def write(self, vals):
        result = super(MailMessageSubtype, self).write(vals)
        self.filtered(
            lambda subtype: subtype.res_model in ['hr.leave', 'hr.leave.allocation']
        )._update_department_subtype()
        return result

```

## File: models\resource.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _
from odoo.exceptions import ValidationError
from odoo.osv import expression
import pytz
from datetime import datetime

class CalendarLeaves(models.Model):
    _inherit = "resource.calendar.leaves"

    holiday_id = fields.Many2one("hr.leave", string='Time Off Request')

    @api.constrains('date_from', 'date_to', 'calendar_id')
    def _check_compare_dates(self):
        all_existing_leaves = self.env['resource.calendar.leaves'].search([
            ('resource_id', '=', False),
            ('company_id', 'in', self.company_id.ids),
            ('date_from', '<=', max(self.mapped('date_to'))),
            ('date_to', '>=', min(self.mapped('date_from'))),
        ])
        for record in self:
            if not record.resource_id:
                existing_leaves = all_existing_leaves.filtered(lambda leave:
                        record.id != leave.id
                        and record['company_id'] == leave['company_id']
                        and record['date_from'] <= leave['date_to']
                        and record['date_to'] >= leave['date_from'])
                if record.calendar_id:
                    existing_leaves = existing_leaves.filtered(lambda l: not l.calendar_id or l.calendar_id == record.calendar_id)
                if existing_leaves:
                    raise ValidationError(_('Two public holidays cannot overlap each other for the same working hours.'))

    def _get_domain(self, time_domain_dict):
        domain = []
        for date in time_domain_dict:
            domain = expression.OR([domain, [
                    ('employee_company_id', '=', date['company_id']),
                    ('date_to', '>', date['date_from']),
                    ('date_from', '<', date['date_to'])]
            ])
        return expression.AND([domain, [('state', '!=', 'refuse'), ('active', '=', True)]])

    def _get_time_domain_dict(self):
        return [{
            'company_id' : record.company_id.id,
            'date_from' : record.date_from,
            'date_to' : record.date_to
        } for record in self if not record.resource_id]

    def _reevaluate_leaves(self, time_domain_dict):
        if not time_domain_dict:
            return

        domain = self._get_domain(time_domain_dict)
        leaves = self.env['hr.leave'].search(domain)
        if not leaves:
            return

        previous_durations = leaves.mapped('number_of_days')
        previous_states = leaves.mapped('state')
        leaves.sudo().write({
            'state': 'draft',
        })
        self.env.add_to_compute(self.env['hr.leave']._fields['number_of_days'], leaves)
        self.env.add_to_compute(self.env['hr.leave']._fields['duration_display'], leaves)
        sick_time_status = self.env.ref('hr_holidays.holiday_status_sl', raise_if_not_found=False)
        for previous_duration, leave, state in zip(previous_durations, leaves, previous_states):
            duration_difference = previous_duration - leave.number_of_days
            message = False
            if duration_difference > 0 and leave.holiday_status_id.requires_allocation == 'yes':
                message = _("Due to a change in global time offs, you have been granted %s day(s) back.", duration_difference)
            if leave.number_of_days > previous_duration\
                    and (not sick_time_status or leave.holiday_status_id not in sick_time_status):
                message = _("Due to a change in global time offs, %s extra day(s) have been taken from your allocation. Please review this leave if you need it to be changed.", -1 * duration_difference)
            try:
                leave.write({'state': state})
                leave._check_validity()
                if leave.state == 'validate':
                    # recreate the resource leave that were removed by writing state to draft
                    leave.sudo()._create_resource_leave()
            except ValidationError:
                leave.action_refuse()
                message = _("Due to a change in global time offs, this leave no longer has the required amount of available allocation and has been set to refused. Please review this leave.")
            if message:
                leave._notify_change(message)

    def _convert_timezone(self, utc_naive_datetime, tz_from, tz_to):
        """
            Convert a naive date to another timezone that initial timezone
            used to generate the date.
            :param utc_naive_datetime: utc date without tzinfo
            :type utc_naive_datetime: datetime
            :param tz_from: timezone used to obtained `utc_naive_datetime`
            :param tz_to: timezone in which we want the date
            :return: datetime converted into tz_to without tzinfo
            :rtype: datetime
        """
        naive_datetime_from = utc_naive_datetime.astimezone(tz_from).replace(tzinfo=None)
        aware_datetime_to = tz_to.localize(naive_datetime_from)
        utc_naive_datetime_to = aware_datetime_to.astimezone(pytz.utc).replace(tzinfo=None)
        return utc_naive_datetime_to

    def _ensure_datetime(self, datetime_representation, date_format=None):
        """
            Be sure to get a datetime object if we have the necessary information.
            :param datetime_reprentation: object which should represent a datetime
            :rtype: datetime if a correct datetime_represtion, None otherwise
        """
        if isinstance(datetime_representation, datetime):
            return datetime_representation
        elif isinstance(datetime_representation, str) and date_format:
            return datetime.strptime(datetime_representation, date_format)
        else:
            return None

    def _prepare_public_holidays_values(self, vals_list):
        for vals in vals_list:
            # Manage the case of create a Public Time Off in another timezone
            # The datetime created has to be in UTC for the calendar's timezone
            if not vals.get('calendar_id') or vals.get('resource_id') or \
                not isinstance(vals.get('date_from'), (datetime, str)) or \
                not isinstance(vals.get('date_to'), (datetime, str)):
                continue
            user_tz = pytz.timezone(self.env.user.tz) if self.env.user.tz else pytz.utc
            calendar_tz = pytz.timezone(self.env['resource.calendar'].browse(vals['calendar_id']).tz)
            if user_tz != calendar_tz:
                datetime_from = self._ensure_datetime(vals['date_from'], '%Y-%m-%d %H:%M:%S')
                datetime_to = self._ensure_datetime(vals['date_to'], '%Y-%m-%d %H:%M:%S')
                if datetime_from and datetime_to:
                    vals['date_from'] = self._convert_timezone(datetime_from, user_tz, calendar_tz)
                    vals['date_to'] = self._convert_timezone(datetime_to, user_tz, calendar_tz)
        return vals_list

    @api.model_create_multi
    def create(self, vals_list):
        vals_list = self._prepare_public_holidays_values(vals_list)
        res = super().create(vals_list)
        time_domain_dict = res._get_time_domain_dict()
        self._reevaluate_leaves(time_domain_dict)
        return res

    def write(self, vals):
        time_domain_dict = self._get_time_domain_dict()
        res = super().write(vals)
        time_domain_dict.extend(self._get_time_domain_dict())
        self._reevaluate_leaves(time_domain_dict)

        return res

    def unlink(self):
        time_domain_dict = self._get_time_domain_dict()
        res = super().unlink()
        self._reevaluate_leaves(time_domain_dict)

        return res

class ResourceCalendar(models.Model):
    _inherit = "resource.calendar"

    associated_leaves_count = fields.Integer("Time Off Count", compute='_compute_associated_leaves_count')

    def _compute_associated_leaves_count(self):
        leaves_read_group = self.env['resource.calendar.leaves']._read_group(
            [('resource_id', '=', False), ('calendar_id', 'in', [False, *self.ids])],
            ['calendar_id'],
            ['__count'],
        )
        result = {calendar.id if calendar else 'global': count for calendar, count in leaves_read_group}
        global_leave_count = result.get('global', 0)
        for calendar in self:
            calendar.associated_leaves_count = result.get(calendar.id, 0) + global_leave_count

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models
from odoo.tools.misc import DEFAULT_SERVER_DATE_FORMAT


class ResPartner(models.Model):
    _inherit = 'res.partner'

    def _compute_im_status(self):
        super(ResPartner, self)._compute_im_status()
        absent_now = self._get_on_leave_ids()
        for partner in self:
            if partner.id in absent_now:
                if partner.im_status == 'online':
                    partner.im_status = 'leave_online'
                elif partner.im_status == 'away':
                    partner.im_status = 'leave_away'
                else:
                    partner.im_status = 'leave_offline'

    @api.model
    def _get_on_leave_ids(self):
        return self.env['res.users']._get_on_leave_ids(partner=True)

    def mail_partner_format(self, fields=None):
        """Override to add the current leave status."""
        partners_format = super().mail_partner_format(fields=fields)
        if not fields:
            fields = {'out_of_office_date_end': True}
        for partner in self:
            if 'out_of_office_date_end' in fields:
                # in the rare case of multi-user partner, return the earliest possible return date
                dates = partner.mapped('user_ids.leave_date_to')
                states = partner.mapped('user_ids.current_leave_state')
                date = sorted(dates)[0] if dates and all(dates) else False
                state = sorted(states)[0] if states and all(states) else False
                partners_format.get(partner).update({
                    'out_of_office_date_end': date.strftime(DEFAULT_SERVER_DATE_FORMAT) if state == 'validate' and date else False,
                })
        return partners_format

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, Command


class User(models.Model):
    _inherit = "res.users"

    leave_manager_id = fields.Many2one(related='employee_id.leave_manager_id')
    show_leaves = fields.Boolean(related='employee_id.show_leaves')
    allocation_count = fields.Float(related='employee_id.allocation_count')
    leave_date_to = fields.Date(related='employee_id.leave_date_to')
    current_leave_state = fields.Selection(related='employee_id.current_leave_state')
    is_absent = fields.Boolean(related='employee_id.is_absent')
    allocation_remaining_display = fields.Char(related='employee_id.allocation_remaining_display')
    allocation_display = fields.Char(related='employee_id.allocation_display')
    hr_icon_display = fields.Selection(related='employee_id.hr_icon_display')

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + [
            'leave_manager_id',
            'show_leaves',
            'allocation_count',
            'leave_date_to',
            'current_leave_state',
            'is_absent',
            'allocation_remaining_display',
            'allocation_display',
            'hr_icon_display',
        ]

    def _compute_im_status(self):
        super(User, self)._compute_im_status()
        on_leave_user_ids = self._get_on_leave_ids()
        for user in self:
            if user.id in on_leave_user_ids:
                if user.im_status == 'online':
                    user.im_status = 'leave_online'
                elif user.im_status == 'away':
                    user.im_status = 'leave_away'
                else:
                    user.im_status = 'leave_offline'

    @api.model
    def _get_on_leave_ids(self, partner=False):
        now = fields.Datetime.now()
        field = 'partner_id' if partner else 'id'
        self.flush_model(['active'])
        self.env['hr.leave'].flush_model(['user_id', 'state', 'date_from', 'date_to'])
        self.env.cr.execute('''SELECT res_users.%s FROM res_users
                            JOIN hr_leave ON hr_leave.user_id = res_users.id
                            AND hr_leave.state = 'validate'
                            AND hr_leave.active = 't'
                            AND res_users.active = 't'
                            AND hr_leave.date_from <= %%s AND hr_leave.date_to >= %%s''' % field, (now, now))
        return [r[0] for r in self.env.cr.fetchall()]

    def _clean_leave_responsible_users(self):
        # self = old bunch of leave responsibles
        # This method compares the current leave managers
        # and remove the access rights to those who don't
        # need them anymore
        approver_group = 'hr_holidays.group_hr_holidays_responsible'
        if not any(u.has_group(approver_group) for u in self):
            return

        res = self.env['hr.employee']._read_group(
            [('leave_manager_id', 'in', self.ids)],
            ['leave_manager_id'])
        responsibles_to_remove_ids = set(self.ids) - {leave_manager.id for [leave_manager] in res}
        if responsibles_to_remove_ids:
            self.browse(responsibles_to_remove_ids).write({
                'groups_id': [Command.unlink(self.env.ref(approver_group).id)],
            })

    @api.model_create_multi
    def create(self, vals_list):
        users = super().create(vals_list)
        users.sudo()._clean_leave_responsible_users()
        return users

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import resource
from . import hr_employee_base
from . import hr_employee
from . import hr_department
from . import hr_leave
from . import hr_leave_allocation
from . import hr_leave_type
from . import hr_leave_accrual_plan_level
from . import hr_leave_accrual_plan
from . import hr_leave_mandatory_day
from . import mail_message_subtype
from . import res_partner
from . import res_users

```

## File: populate\hr_leave.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime

from odoo import models
from odoo.tools import populate
from dateutil.relativedelta import relativedelta
from itertools import groupby


class HolidaysType(models.Model):
    _inherit = "hr.leave.type"
    _populate_sizes = {"small": 10, "medium": 30, "large": 100}
    _populate_dependencies = ['res.company']

    def _populate_factories(self):

        company_ids = self.env.registry.populated_models['res.company']

        return [
            ('name', populate.constant('leave_type_{counter}')),
            ('company_id', populate.randomize(company_ids)),
            ('requires_allocation', populate.randomize(['yes', 'no'], [0.3, 0.7])),
            ('employee_requests', populate.randomize(['yes', 'no'], [0.2, 0.8])),
            ('request_unit', populate.randomize(['hour', 'day'], [0.2, 0.8])),
        ]


class HolidaysRequest(models.Model):
    _inherit = "hr.leave"
    _populate_sizes = {"small": 100, "medium": 800, "large": 10000}
    _populate_dependencies = ['hr.employee', 'hr.leave.type']

    def _populate_factories(self):

        employee_ids = self.env.registry.populated_models['hr.employee']
        hr_leave_type_ids = self.env.registry.populated_models['hr.leave.type']

        hr_leave_type_records = self.env['hr.leave.type'].browse(hr_leave_type_ids)
        allocationless_leave_type_ids = hr_leave_type_records.filtered(lambda lt: lt.requires_allocation == 'no').ids

        employee_records = self.env['hr.employee'].browse(employee_ids)
        employee_by_company = {k: list(v) for k, v in groupby(employee_records, key=lambda rec: rec['company_id'].id)}
        company_by_type = {rec.id: rec.company_id.id for rec in self.env['hr.leave.type'].browse(hr_leave_type_ids)}

        def compute_employee_id(random=None, values=None, **kwargs):
            company_id = company_by_type[values['holiday_status_id']]
            return random.choice(employee_by_company[company_id]).id

        def compute_request_date_from(counter, **kwargs):
            return datetime.datetime.today() + relativedelta(days=int(3 * int(counter)))

        def compute_request_date_to(counter, random=None, **kwargs):
            return datetime.datetime.today() + relativedelta(days=int(3 * int(counter)) + random.randint(0, 2))

        return [
            ('holiday_status_id', populate.randomize(allocationless_leave_type_ids)),
            ('employee_id', populate.compute(compute_employee_id)),
            ('holiday_type', populate.constant('employee')),
            ('request_date_from', populate.compute(compute_request_date_from)),
            ('request_date_to', populate.compute(compute_request_date_to)),
            ('state', populate.randomize([
                'draft',
                'confirm',
            ])),
        ]

```

## File: populate\hr_leave_allocation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.tools import populate


class HolidaysAllocation(models.Model):
    _inherit = "hr.leave.allocation"
    _populate_sizes = {"small": 100, "medium": 800, "large": 10000}
    _populate_dependencies = ['hr.employee', 'hr.leave.type']

    def _populate_factories(self):

        employee_ids = self.env.registry.populated_models['hr.employee']
        hr_leave_type_ids = self.env['hr.leave.type']\
            .browse(self.env.registry.populated_models['hr.leave.type'])\
            .filtered(lambda lt: lt.requires_allocation == 'yes')\
            .ids

        return [
            ('holiday_status_id', populate.randomize(hr_leave_type_ids)),
            ('employee_id', populate.randomize(employee_ids)),
        ]

```

## File: populate\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_leave, hr_leave_allocation

```

## File: report\holidays_summary_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import babel.dates
import calendar

from datetime import timedelta
from dateutil.relativedelta import relativedelta
from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.tools.misc import format_date, get_lang

COLORS_MAP = {
    0: 'lightgrey',
    1: 'tomato',
    2: 'sandybrown',
    3: 'khaki',
    4: 'skyblue',
    5: 'dimgrey',
    6: 'lightcoral',
    7: 'steelblue',
    8: 'darkslateblue',
    9: 'crimson',
    10: 'mediumseagreen',
    11: 'mediumpurple',
}


class HrHolidaySummaryReport(models.AbstractModel):
    _name = 'report.hr_holidays.report_holidayssummary'
    _description = 'Holidays Summary Report'

    def _get_header_info(self, start_date, holiday_type):
        st_date = fields.Date.from_string(start_date)
        if holiday_type == 'Confirmed':
            holiday_type = _('Confirmed')
        elif holiday_type == 'Approved':
            holiday_type = _('Approved')
        else:
            holiday_type = _('Confirmed and Approved')
        return {
            'start_date': format_date(self.env, st_date),
            'end_date': format_date(self.env, st_date + relativedelta(days=59)),
            'holiday_type': holiday_type
        }

    def _date_is_day_off(self, date):
        return date.weekday() in (calendar.SATURDAY, calendar.SUNDAY,)

    def _get_day(self, start_date):
        res = []
        start_date = fields.Date.from_string(start_date)
        for x in range(0, 60):
            color = '#ababab' if self._date_is_day_off(start_date) else ''
            res.append({'day_str': babel.dates.get_day_names('abbreviated', locale=get_lang(self.env).code)[start_date.weekday()], 'day': start_date.day, 'color': color})
            start_date = start_date + relativedelta(days=1)
        return res

    def _get_months(self, start_date):
        # it works for geting month name between two dates.
        res = []
        start_date = fields.Date.from_string(start_date)
        end_date = start_date + relativedelta(days=59)
        while start_date <= end_date:
            last_date = start_date + relativedelta(day=1, months=+1, days=-1)
            if last_date > end_date:
                last_date = end_date
            month_days = (last_date - start_date).days + 1
            res.append({'month_name': babel.dates.get_month_names(locale=get_lang(self.env).code)[start_date.month], 'days': month_days})
            start_date += relativedelta(day=1, months=+1)
        return res

    def _get_leaves_summary(self, start_date, empid, holiday_type):
        res = []
        count = 0
        start_date = fields.Date.from_string(start_date)
        end_date = start_date + relativedelta(days=59)
        for index in range(0, 60):
            current = start_date + timedelta(index)
            res.append({'day': current.day, 'color': ''})
            if self._date_is_day_off(current) :
                res[index]['color'] = '#ababab'

        holidays = self._get_leaves(start_date, self.env['hr.employee'].browse(empid), holiday_type)

        for holiday in holidays:
            # Convert date to user timezone, otherwise the report will not be consistent with the
            # value displayed in the interface.
            date_from = fields.Datetime.from_string(holiday.date_from)
            date_from = fields.Datetime.context_timestamp(holiday, date_from).date()
            date_to = fields.Datetime.from_string(holiday.date_to)
            date_to = fields.Datetime.context_timestamp(holiday, date_to).date()
            for index in range(0, ((date_to - date_from).days + 1)):
                if date_from >= start_date and date_from <= end_date:
                    res[(date_from-start_date).days]['color'] = COLORS_MAP[holiday.holiday_status_id.color]
                date_from += timedelta(1)
            count += holiday.number_of_days
        employee = self.env['hr.employee'].browse(empid)
        return {'emp': employee.name, 'display': res, 'sum': count}

    def _get_employees(self, data):
        if 'depts' in data:
            return self.env['hr.employee'].search([('department_id', 'in', data['depts'])])
        elif 'emp' in data:
            return self.env['hr.employee'].browse(data['emp'])
        return self.env['hr.employee'].search([])

    def _get_data_from_report(self, data):
        res = []
        if 'depts' in data:
            employees = self._get_employees(data)
            departments = self.env['hr.department'].browse(data['depts'])
            for department in departments:
                res.append({
                    'dept': department.name,
                    'data': [
                        self._get_leaves_summary(data['date_from'], emp.id, data['holiday_type'])
                        for emp in employees.filtered(lambda emp: emp.department_id.id == department.id)
                    ],
                    'color': self._get_day(data['date_from']),
                })
        elif 'emp' in data:
            res.append({'data': [
                self._get_leaves_summary(data['date_from'], emp.id, data['holiday_type'])
                for emp in self._get_employees(data)
            ]})
        return res

    def _get_leaves(self, date_from, employees, holiday_type, date_to=None):
        state = ['confirm', 'validate'] if holiday_type == 'both' else ['confirm'] if holiday_type == 'Confirmed' else ['validate']

        if not date_to:
            date_to = date_from + relativedelta(days=59)

        return self.env['hr.leave'].search([
            ('employee_id', 'in', employees.ids),
            ('state', 'in', state),
            ('date_from', '<=', str(date_to)),
            ('date_to', '>=', str(date_from))
        ])

    def _get_holidays_status(self, data):
        res = []
        employees = self.env['hr.employee']
        if {'depts', 'emp'} & data.keys():
            employees = self._get_employees(data)

        holidays = self._get_leaves(fields.Date.from_string(data['date_from']), employees, data['holiday_type'])

        for leave_type in holidays.holiday_status_id:
            res.append({'color': COLORS_MAP[leave_type.color], 'name': leave_type.name})

        return res

    @api.model
    def _get_report_values(self, docids, data=None):

        holidays_report = self.env['ir.actions.report']._get_report_from_name('hr_holidays.report_holidayssummary')
        holidays = self.env['hr.leave'].browse(self.ids)
        if data and data.get('form'):
            return {
                'doc_ids': self.ids,
                'doc_model': holidays_report.model,
                'docs': holidays,
                'get_header_info': self._get_header_info(data['form']['date_from'], data['form']['holiday_type']),
                'get_day': self._get_day(data['form']['date_from']),
                'get_months': self._get_months(data['form']['date_from']),
                'get_data_from_report': self._get_data_from_report(data['form']),
                'get_holidays_status': self._get_holidays_status(data['form']),
            }

        return {
            'doc_ids': self.ids,
            'doc_model': holidays_report.model,
            'docs': holidays
        }

```

## File: report\hr_holidays_reports.xml

```xml
<?xml version="1.0"?>
<odoo>

    <record id="action_report_holidayssummary" model="ir.actions.report">
        <field name="name">Time Off Summary</field>
        <field name="model">hr.leave.allocation</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">hr_holidays.report_holidayssummary</field>
        <field name="report_file">hr_holidays.report_holidayssummary</field>
        <field name="paperformat_id" ref="hr_holidays.paperformat_hrsummary"/>
    </record>

</odoo>

```

## File: report\hr_holidays_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="report_holidayssummary">
    <t t-call="web.html_container">
        <t t-call="web.internal_layout">
            <div class="page">
                <h3 class="mb32">Time Off Summary</h3>
                <t t-set="info" t-value="get_header_info"/>
                <h3 class="text-center mb32">
                    Analyze from <u><t t-esc="info['start_date']"/></u> to <u><t t-esc="info['end_date']"/></u> of the <u><t t-esc="info['holiday_type']"/></u> Time Off.
                </h3>

                <table class="table table-bordered mb32" style="table-layout:auto">
                    <thead>
                        <tr>
                            <th>Month</th>
                            <t t-foreach="get_months" t-as="month">
                                &lt;th class="text-center" colspan=<t t-esc="month['days']"/>&gt;<t t-esc="month['month_name']"/>&lt;/th&gt;
                            </t>
                            <th/>
                        </tr>
                        <tr>
                            <td rowspan="2">
                                <strong>Departments and Employees</strong>
                            </td>
                            <t t-foreach="get_day" t-as="day">
                                &lt;td class="text-center oe_leftfit oe_rightfit" style="background-color:<t t-esc="day['color']"/>!important; font-size: 8px; min-width: 18px"&gt; <t t-esc="day['day_str']"/>&lt;/td&gt;
                            </t>
                            <td/>
                        </tr>
                        <tr>
                            <t t-foreach="get_day" t-as="day">
                                &lt;td class="text-center oe_leftfit oe_rightfit" style="background-color:<t t-esc="day['color']"/>!important; font-size: 10px" &gt; <t t-esc="day['day']"/>&lt;/td&gt;
                            </t>
                            <td class="text-center">Sum</td>
                        </tr>
                    </thead>
                    <tbody>
                        <t t-foreach="get_data_from_report" t-as="obj">
                            <tr t-if="'dept' in obj">
                                <td style="background-color:#ababab">
                                    <strong><t t-esc="obj['dept']"/></strong>
                                </td>
                                <t t-foreach="obj['color']" t-as="c">
                                    &lt;td style=background-color:<t t-esc="c['color']"/> !important/&gt;
                                </t>
                                <td/>
                            </tr>
                            <tr t-foreach="obj['data']" t-as="emp">
                                <td><t t-esc="emp['emp']"/></td>
                                <t t-foreach="emp['display']" t-as="details">
                                    &lt;td style=background-color:<t t-esc="details['color']"/> !important /&gt;
                                </t>
                                <td class="text-center"><strong><t t-esc="emp['sum']"/></strong></td>
                            </tr>
                        </t>
                    </tbody>
                </table>

                <div class="col-3 offset-5 mt32">
                    <table class="table table-bordered">
                        <thead>
                            <tr>
                                <th class="col-1">Color</th>
                                <th class="text-center">Time Off Type</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr t-foreach="get_holidays_status" t-as="status">
                                <span t-if="status['color']">
                                    &lt;td style=background-color:<t t-esc="status['color']"/>!important &gt;&lt;/td&gt;
                                    <td><t t-esc="status['name']"/></td>
                                </span>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </t>
    </t>
</template>

</odoo>

```

## File: report\hr_leave_employee_type_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools, _


class LeaveReport(models.Model):
    _name = "hr.leave.employee.type.report"
    _description = 'Time Off Summary / Report'
    _auto = False
    _order = "date_from DESC, employee_id"

    employee_id = fields.Many2one('hr.employee', string="Employee", readonly=True)
    active_employee = fields.Boolean(readonly=True)
    number_of_days = fields.Float('Number of Days', readonly=True, group_operator="sum")
    department_id = fields.Many2one('hr.department', string='Department', readonly=True)
    leave_type = fields.Many2one("hr.leave.type", string="Time Off Type", readonly=True)
    holiday_status = fields.Selection([
        ('taken', 'Taken'), #taken = validated
        ('left', 'Left'),
        ('planned', 'Planned')
    ])
    state = fields.Selection([
        ('draft', 'To Submit'),
        ('cancel', 'Cancelled'),
        ('confirm', 'To Approve'),
        ('refuse', 'Refused'),
        ('validate1', 'Second Approval'),
        ('validate', 'Approved')
        ], string='Status', readonly=True)
    date_from = fields.Datetime('Start Date', readonly=True)
    date_to = fields.Datetime('End Date', readonly=True)
    company_id = fields.Many2one('res.company', string="Company", readonly=True)

    def init(self):
        tools.drop_view_if_exists(self._cr, 'hr_leave_employee_type_report')

        self._cr.execute("""
            CREATE or REPLACE view hr_leave_employee_type_report as (
                SELECT row_number() over(ORDER BY leaves.employee_id) as id,
                leaves.employee_id as employee_id,
                leaves.active_employee as active_employee,
                leaves.number_of_days as number_of_days,
                leaves.department_id as department_id,
                leaves.leave_type as leave_type,
                leaves.holiday_status as holiday_status,
                leaves.state as state,
                leaves.date_from as date_from,
                leaves.date_to as date_to,
                leaves.company_id as company_id
                FROM (SELECT
                    allocation.employee_id as employee_id,
                    employee.active as active_employee,
                    CASE
                        WHEN allocation.id = min_allocation_id.min_id
                            THEN aggregate_allocation.number_of_days - COALESCE(aggregate_leave.number_of_days, 0)
                            ELSE 0
                    END as number_of_days,
                    allocation.department_id as department_id,
                    allocation.holiday_status_id as leave_type,
                    allocation.state as state,
                    allocation.date_from as date_from,
                    allocation.date_to as date_to,
                    'left' as holiday_status,
                    allocation.employee_company_id as company_id
                FROM hr_leave_allocation as allocation
                INNER JOIN hr_employee as employee ON (allocation.employee_id = employee.id)

                /* Obtain the minimum id for a given employee and type of leave */
                LEFT JOIN
                    (SELECT employee_id, holiday_status_id, min(id) as min_id
                    FROM hr_leave_allocation GROUP BY employee_id, holiday_status_id) min_allocation_id
                on (allocation.employee_id=min_allocation_id.employee_id and allocation.holiday_status_id=min_allocation_id.holiday_status_id)

                /* Obtain the sum of allocations (validated) */
                LEFT JOIN
                    (SELECT employee_id, holiday_status_id,
                        sum(CASE WHEN state = 'validate' and active = True THEN number_of_days ELSE 0 END) as number_of_days
                    FROM hr_leave_allocation
                    GROUP BY employee_id, holiday_status_id) aggregate_allocation
                on (allocation.employee_id=aggregate_allocation.employee_id and allocation.holiday_status_id=aggregate_allocation.holiday_status_id)

                /* Obtain the sum of requested leaves (validated) */
                LEFT JOIN
                    (SELECT employee_id, holiday_status_id,
                        sum(CASE WHEN state IN ('validate', 'validate1') THEN number_of_days ELSE 0 END) as number_of_days
                    FROM hr_leave

                    GROUP BY employee_id, holiday_status_id) aggregate_leave
                on (allocation.employee_id=aggregate_leave.employee_id and allocation.holiday_status_id = aggregate_leave.holiday_status_id)

                UNION ALL SELECT
                    request.employee_id as employee_id,
                    employee.active as active_employee,
                    request.number_of_days as number_of_days,
                    request.department_id as department_id,
                    request.holiday_status_id as leave_type,
                    request.state as state,
                    request.date_from as date_from,
                    request.date_to as date_to,
                    CASE
                        WHEN request.state IN ('validate1', 'validate') THEN 'taken'
                        WHEN request.state = 'confirm' THEN 'planned'
                    END as holiday_status,
                    request.employee_company_id as company_id
                FROM hr_leave as request
                INNER JOIN hr_employee as employee ON (request.employee_id = employee.id)
                WHERE request.state IN ('confirm', 'validate', 'validate1')) leaves
            );
        """)

    @api.model
    def action_time_off_analysis(self):
        domain = []
        if self.env.context.get('active_ids'):
            domain = [('employee_id', 'in', self.env.context.get('active_ids', []))]

        return {
            'name': _('Time Off Analysis'),
            'type': 'ir.actions.act_window',
            'res_model': 'hr.leave.employee.type.report',
            'view_mode': 'pivot',
            'search_view_id': [self.env.ref('hr_holidays.view_search_hr_holidays_employee_type_report').id],
            'domain': domain,
            'context': {
                'search_default_year': True,
                'search_default_company': True,
                'search_default_employee': True,
                'group_expand': True,
            }
        }

```

## File: report\hr_leave_employee_type_report.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="view_search_hr_holidays_employee_type_report" model="ir.ui.view">
        <field name="name">hr.holidays.filter</field>
        <field name="model">hr.leave.employee.type.report</field>
        <field name="arch" type="xml">
            <search string="Search Time Off">
                <field name="employee_id"/>
                <field name="date_from"/>
                <filter name="year" date="date_from" default_period="this_year" string="Period"/>
                <filter string="Company" name="company" context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
                <filter string="Employee" name="employee" context="{'group_by':'employee_id'}"/>
            </search>
        </field>
    </record>


    <record id="hr_leave_employee_type_report" model="ir.ui.view">
        <field name="name">hr.leave.employee.type.report.view.pivot</field>
        <field name="model">hr.leave.employee.type.report</field>
        <field name="arch" type="xml">
            <pivot sample="1" disable_linking="1">
                <field name="employee_id" type="row"/>
                <field name="number_of_days" type="measure"/>
                <field name="leave_type" type="col"/>
                <field name="holiday_status" type="col"/>
            </pivot>
        </field>
    </record>

    <record id="action_hr_holidays_by_employee_and_type_report" model="ir.actions.server">
        <field name="name">Time off Analysis by Employee and Time Off Type</field>
        <field name="model_id" ref="hr_holidays.model_hr_leave_employee_type_report"/>
        <field name="binding_model_id" ref="hr.model_hr_employee"/>
        <field name="state">code</field>
        <field name="groups_id" eval="[(4, ref('base.group_no_one'))]"/>
        <field name="code">
            action = model.action_time_off_analysis()
        </field>
    </record>

</odoo>

```

## File: report\hr_leave_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools, _
from odoo.osv import expression


class LeaveReport(models.Model):
    _name = "hr.leave.report"
    _description = 'Time Off Summary / Report'
    _auto = False
    _order = "date_from DESC, employee_id"

    employee_id = fields.Many2one('hr.employee', string="Employee", readonly=True)
    leave_id = fields.Many2one('hr.leave', string="Time Off Request", readonly=True)
    name = fields.Char('Description', readonly=True)
    number_of_days = fields.Float('Number of Days', readonly=True)
    leave_type = fields.Selection([
        ('allocation', 'Allocation'),
        ('request', 'Time Off')
        ], string='Request Type', readonly=True)
    department_id = fields.Many2one('hr.department', string='Department', readonly=True)
    category_id = fields.Many2one('hr.employee.category', string='Employee Tag', readonly=True)
    holiday_status_id = fields.Many2one("hr.leave.type", string="Time Off Type", readonly=True)
    state = fields.Selection([
        ('draft', 'To Submit'),
        ('cancel', 'Cancelled'),
        ('confirm', 'To Approve'),
        ('refuse', 'Refused'),
        ('validate1', 'Second Approval'),
        ('validate', 'Approved')
        ], string='Status', readonly=True)
    holiday_type = fields.Selection([
        ('employee', 'By Employee'),
        ('category', 'By Employee Tag')
    ], string='Allocation Mode', readonly=True)
    date_from = fields.Datetime('Start Date', readonly=True)
    date_to = fields.Datetime('End Date', readonly=True)
    company_id = fields.Many2one('res.company', string="Company", readonly=True)

    def init(self):
        tools.drop_view_if_exists(self._cr, 'hr_leave_report')

        self._cr.execute("""
            CREATE or REPLACE view hr_leave_report as (
                SELECT row_number() over(ORDER BY leaves.employee_id) as id,
                leaves.leave_id as leave_id,
                leaves.employee_id as employee_id, leaves.name as name,
                leaves.number_of_days as number_of_days, leaves.leave_type as leave_type,
                leaves.category_id as category_id, leaves.department_id as department_id,
                leaves.holiday_status_id as holiday_status_id, leaves.state as state,
                leaves.holiday_type as holiday_type, leaves.date_from as date_from,
                leaves.date_to as date_to, leaves.company_id
                from (select
                    allocation.id as leave_id,
                    allocation.employee_id as employee_id,
                    allocation.private_name as name,
                    allocation.number_of_days as number_of_days,
                    allocation.category_id as category_id,
                    allocation.department_id as department_id,
                    allocation.holiday_status_id as holiday_status_id,
                    allocation.state as state,
                    allocation.holiday_type,
                    allocation.date_from as date_from,
                    allocation.date_to as date_to,
                    'allocation' as leave_type,
                    allocation.employee_company_id as company_id
                from hr_leave_allocation as allocation
                inner join hr_employee as employee on (allocation.employee_id = employee.id)
                where employee.active IS True AND
                allocation.active IS True
                union all select
                    request.id as leave_id,
                    request.employee_id as employee_id,
                    request.private_name as name,
                    (request.number_of_days * -1) as number_of_days,
                    request.category_id as category_id,
                    request.department_id as department_id,
                    request.holiday_status_id as holiday_status_id,
                    request.state as state,
                    request.holiday_type,
                    request.date_from as date_from,
                    request.date_to as date_to,
                    'request' as leave_type,
                    request.employee_company_id as company_id
                from hr_leave as request
                inner join hr_employee as employee on (request.employee_id = employee.id)
                where employee.active IS True AND
                request.active is True
                ) leaves
            );
        """)

    def action_open_record(self):
        self.ensure_one()

        return {
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_id': self.leave_id.id,
            'res_model': 'hr.leave' if self.leave_type == 'request' else 'hr.leave.allocation',
        }

```

## File: report\hr_leave_reports.xml

```xml
<?xml version="1.0"?>
<odoo>

    <record id="view_hr_holidays_filter_report" model="ir.ui.view">
        <field name="name">hr.holidays.filter</field>
        <field name="model">hr.leave.report</field>
        <field name="arch" type="xml">
            <search string="Search Time Off">
                <field name="employee_id"/>
                <field name="name"/>
                <filter domain="[('state','in',('confirm','validate1'))]" string="To Approve" name="approve"/>
                <filter string="Approved Requests" domain="[('state', '=', 'validate')]" name="validated"/>
                <separator/>
                <filter string="Time off" domain="[('leave_type', '=', 'request')]" name="leave_type_timeoff"/>
                <filter string="Allocations" domain="[('leave_type', '=', 'allocation')]" name="leave_type_allocations"/>
                <separator/>
                <filter string="My Department" name="department" domain="[('department_id.manager_id.user_id', '=', uid)]" help="My Department"/>
                <separator/>
                <filter name="year" date="date_from" default_period="this_year" string="Current Year"/>
                <separator/>
                <filter string="My Requests" name="my_leaves" domain="[('employee_id.user_id', '=', uid)]"/>
                <field name="department_id" operator="child_of"/>
                <field name="holiday_status_id"/>
                <group expand="0" string="Group By">
                    <filter name="group_employee" string="Employee" context="{'group_by':'employee_id'}"/>
                    <filter name="group_type" string="Type" context="{'group_by':'holiday_status_id'}"/>
                    <filter name="group_company" string="Company" context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
                    <separator/>
                    <filter name="group_date_from" string="Start Date" context="{'group_by':'date_from'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="hr_leave_report_tree" model="ir.ui.view">
        <field name="name">report.hr.holidays.report.leave_all.tree</field>
        <field name="model">hr.leave.report</field>
        <field name="arch" type="xml">
            <tree create="0" edit="0" delete="0" action="action_open_record" type="object">
                <field name="employee_id"/>
                <field name="number_of_days" string="Number of Days" sum="Remaining Days"/>
                <field name="leave_type"/>
                <field name="date_from"/>
                <field name="date_to"/>
                <field name="state"/>
                <field name="name"/>
            </tree>
        </field>
    </record>

    <record id="hr_leave_report_graph" model="ir.ui.view">
        <field name="name">report.hr.holidays.report.leave_all.graph</field>
        <field name="model">hr.leave.report</field>
        <field name="arch" type="xml">
            <graph string="Time off Summary" sample="1">
                <field name="employee_id"/>
                <field name="number_of_days" string="Duration (Days)" type="measure"/>
            </graph>
        </field>
    </record>

    <record id="hr_leave_report_pivot" model="ir.ui.view">
        <field name="name">report.hr.holidays.report.leave_all.pivot</field>
        <field name="model">hr.leave.report</field>
        <field name="arch" type="xml">
            <pivot string="Time off Summary" sample="1">
                <field name="employee_id"/>
                <field name="number_of_days" type="measure"/>
            </pivot>
        </field>
    </record>

    <record id="act_hr_employee_holiday_type" model="ir.actions.act_window">
        <field name="name">Time Off Analysis</field>
        <field name="res_model">hr.leave.report</field>
        <field name="view_mode">graph,tree,pivot</field>
        <field name="search_view_id" ref="view_hr_holidays_filter_report"/>
        <field name="domain">[('holiday_type','=','employee')]</field>
        <field name="context">{'search_default_group_type': 1}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No data yet!
            </p>
        </field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No data to display
            </p>
        </field>
    </record>

</odoo>

```

## File: report\hr_leave_report_calendar.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools, SUPERUSER_ID

from odoo.addons.base.models.res_partner import _tz_get


class LeaveReportCalendar(models.Model):
    _name = "hr.leave.report.calendar"
    _description = 'Time Off Calendar'
    _auto = False
    _order = "start_datetime DESC, employee_id"

    name = fields.Char(string='Name', readonly=True)
    start_datetime = fields.Datetime(string='From', readonly=True)
    stop_datetime = fields.Datetime(string='To', readonly=True)
    tz = fields.Selection(_tz_get, string="Timezone", readonly=True)
    duration = fields.Float(string='Duration', readonly=True)
    employee_id = fields.Many2one('hr.employee', readonly=True)
    department_id = fields.Many2one('hr.department', readonly=True)
    job_id = fields.Many2one('hr.job', readonly=True)
    company_id = fields.Many2one('res.company', readonly=True)
    state = fields.Selection([
        ('draft', 'To Submit'),
        ('cancel', 'Cancelled'),  # YTI This state seems to be unused. To remove
        ('confirm', 'To Approve'),
        ('refuse', 'Refused'),
        ('validate1', 'Second Approval'),
        ('validate', 'Approved')
    ], readonly=True)

    is_hatched = fields.Boolean('Hatched', readonly=True)
    is_striked = fields.Boolean('Striked', readonly=True)

    is_absent = fields.Boolean(related='employee_id.is_absent')

    def init(self):
        tools.drop_view_if_exists(self._cr, 'hr_leave_report_calendar')
        self._cr.execute("""CREATE OR REPLACE VIEW hr_leave_report_calendar AS
        (SELECT 
            hl.id AS id,
            CONCAT(em.name, ': ', hl.duration_display) AS name,
            hl.date_from AS start_datetime,
            hl.date_to AS stop_datetime,
            hl.employee_id AS employee_id,
            hl.state AS state,
            hl.department_id AS department_id,
            hl.number_of_days as duration,
            em.company_id AS company_id,
            em.job_id AS job_id,
            COALESCE(
                CASE WHEN hl.holiday_type = 'employee' THEN COALESCE(rr.tz, rc.tz) END,
                cc.tz,
                'UTC'
            ) AS tz,
            hl.state = 'refuse' as is_striked,
            hl.state not in ('validate', 'refuse') as is_hatched
        FROM hr_leave hl
            LEFT JOIN hr_employee em
                ON em.id = hl.employee_id
            LEFT JOIN resource_resource rr
                ON rr.id = em.resource_id
            LEFT JOIN resource_calendar rc
                ON rc.id = em.resource_calendar_id
            LEFT JOIN res_company co
                ON co.id = em.company_id
            LEFT JOIN resource_calendar cc
                ON cc.id = co.resource_calendar_id
        WHERE 
            hl.state IN ('confirm', 'validate', 'validate1')
            AND hl.active IS TRUE
        );
        """)

    def _fetch_query(self, query, fields):
        records = super()._fetch_query(query, fields)
        if self.env.context.get('hide_employee_name') and 'employee_id' in self.env.context.get('group_by', []):
            self.env.cache.update(records, self._fields['name'], [
                record.name.split(':')[-1].strip()
                for record in records.with_user(SUPERUSER_ID)
            ])
        return records

    @api.model
    def get_unusual_days(self, date_from, date_to=None):
        return self.env.user.employee_id._get_unusual_days(date_from, date_to)

```

## File: report\hr_leave_report_calendar.xml

```xml
<?xml version='1.0' encoding='UTF-8' ?>
<odoo>
    <record id="hr_leave_report_calendar_view" model="ir.ui.view">
        <field name="name">hr.leave.report.calendar.view</field>
        <field name="model">hr.leave.report.calendar</field>
        <field name="arch" type="xml">
            <calendar
                string="Time Off"
                date_start="start_datetime"
                date_stop="stop_datetime"
                mode="month"
                quick_create="0"
                color="employee_id"
                event_open_popup="True"
                js_class="time_off_calendar"
                show_unusual_days="True">
                <field name="name"/>
                <field name="employee_id" filters="1" invisible="1"/>
                <field name="is_hatched" invisible="1"/>
            </calendar>
        </field>
    </record>

    <record id="hr_leave_report_calendar_view_form" model="ir.ui.view">
        <field name="name">hr.leave.report.calendar.view.form</field>
        <field name="model">hr.leave.report.calendar</field>
        <field name="arch" type="xml">
            <form string="Time Off">
                <group>
                    <field name="name"/>
                    <field name="start_datetime"/>
                    <field name="stop_datetime"/>
                    <field name="employee_id" />
                </group>
            </form>
        </field>
    </record>

    <record id="hr_leave_report_calendar_view_search" model="ir.ui.view">
        <field name="name">hr.leave.report.calendar.view.search</field>
        <field name="model">hr.leave.report.calendar</field>
        <field name="arch" type="xml">
            <search string="Department search">
                <field name="name"/>
                <field name="employee_id"/>
                <field name="department_id"/>
                <field name="job_id"/>
                <filter name="my_team" string="My Team" domain="['|', ('employee_id.user_id', '=', uid), ('employee_id.parent_id.user_id', '=', uid)]"/>
                <filter string="My Department" name="department"
                        domain="[('employee_id.member_of_department', '=', True)]"
                        help="My Department"/>
                <separator/>
                <filter string="Off Today" name="off_today" domain="[('is_absent', '=', True)]" help="Employees Off Today"/>
                <separator/>
                <filter string="Approved" name="validate" domain="[('state', '=', 'validate')]" help="validate"/>
                <filter string="Waiting for Approval" name="approve" domain="[('state','in',('confirm','validate1'))]"/>
                <filter name="groupby_job_id" string="Job Position" context="{'group_by': 'job_id'}"/>
                <filter name="groupby_company_id" string="Company" context="{'group_by': 'company_id'}" groups="base.group_multi_company"/>
                <filter name="groupby_department_id" context="{'group_by': 'department_id'}"/>
            </search>
        </field>
    </record>

    <record id="action_hr_holidays_dashboard" model="ir.actions.act_window">
        <field name="name">All Time Off</field>
        <field name="res_model">hr.leave.report.calendar</field>
        <field name="view_mode">calendar</field>
        <field name="search_view_id" ref="hr_leave_report_calendar_view_search"/>
        <field name="domain">[('employee_id.active','=',True)]</field>
        <field name="context">{'hide_employee_name': 1, 'search_default_my_team': 1}</field>
    </record>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import holidays_summary_report
from . import hr_leave_report
from . import hr_leave_report_calendar
from . import hr_leave_employee_type_report

```

## File: security\hr_holidays_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.module.category" id="base.module_category_human_resources_time_off">
        <field name="description">A user without any rights on Time Off will be able to see the application, create his own holidays and manage the requests of the users he's manager of.</field>
        <field name="sequence">10</field>
    </record>

    <record id="group_hr_holidays_responsible" model="res.groups">
        <field name="name">Time Off Responsible</field>
        <field name="category_id" ref="base.module_category_hidden"/>
        <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <record id="group_hr_holidays_user" model="res.groups">
        <field name="name">Officer: Manage all requests</field>
        <field name="category_id" ref="base.module_category_human_resources_time_off"/>
        <field name="implied_ids" eval="[(4, ref('hr_holidays.group_hr_holidays_responsible'))]"/>
    </record>

    <record id="group_hr_holidays_manager" model="res.groups">
        <field name="name">Administrator</field>
        <field name="category_id" ref="base.module_category_human_resources_time_off"/>
        <field name="implied_ids" eval="[(4, ref('hr_holidays.group_hr_holidays_user'))]"/>
        <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
    </record>

    <data noupdate="1">

    <record id="base.default_user" model="res.users">
        <field name="groups_id" eval="[(4,ref('hr_holidays.group_hr_holidays_manager'))]"/>
    </record>

    <record id="hr_leave_rule_employee" model="ir.rule">
        <field name="name">Time Off base.group_user read</field>
        <field name="model_id" ref="model_hr_leave"/>
        <field name="domain_force">[('employee_id.user_id', '=', user.id)]</field>
        <field name="perm_create" eval="False"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="groups" eval="[(4,ref('base.group_user'))]"/>
    </record>

    <record id="hr_leave_rule_employee_update" model="ir.rule">
        <field name="name">Time Off base.group_user create/write</field>
        <field name="model_id" ref="model_hr_leave"/>
        <field name="domain_force">[
            ('holiday_type', '=', 'employee'),
            '|',
                '&amp;',
                    ('employee_id.user_id', '=', user.id),
                    ('state', 'not in', ['validate', 'validate1']),
                '&amp;',
                    ('validation_type', 'in', ['manager', 'both', 'no_validation']),
                    ('employee_id.leave_manager_id', '=', user.id),
        ]</field>
        <field name="perm_read" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="groups" eval="[(4,ref('base.group_user'))]"/>
    </record>

    <record id="hr_leave_rule_employee_unlink" model="ir.rule">
        <field name="name">Time Off base.group_user unlink</field>
        <field name="model_id" ref="model_hr_leave"/>
        <field name="domain_force">[('employee_id.user_id', '=', user.id), ('state', 'in', ['draft', 'confirm', 'validate1'])]</field>
        <field name="perm_read" eval="False"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="True"/>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <record id="hr_leave_rule_responsible_read" model="ir.rule">
        <field name="name">Time Off Responsible read</field>
        <field name="model_id" ref="model_hr_leave"/>
        <field name="domain_force">[
                ('employee_id.leave_manager_id', '=', user.id),
        ]</field>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="groups" eval="[(4, ref('hr_holidays.group_hr_holidays_responsible'))]"/>
    </record>

    <record id="hr_leave_rule_responsible_update" model="ir.rule">
        <field name="name">Time Off Responsible create/write</field>
        <field name="model_id" ref="model_hr_leave"/>
        <field name="domain_force">[
            ('holiday_type', '=', 'employee'),
            '|',
                '&amp;',
                    ('employee_id.user_id', '=', user.id),
                    ('state', '!=', 'validate'),
                ('employee_id.leave_manager_id', '=', user.id),
        ]</field>
        <field name="perm_read" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="groups" eval="[(4, ref('hr_holidays.group_hr_holidays_responsible'))]"/>
    </record>

    <record id="hr_leave_rule_user_read" model="ir.rule">
        <field name="name">Time Off All Approver read</field>
        <field name="model_id" ref="model_hr_leave"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="perm_write" eval="True"/>
        <field name="perm_create" eval="True"/>
        <field name="perm_unlink" eval="True"/>
        <field name="groups" eval="[(4, ref('hr_holidays.group_hr_holidays_user'))]"/>
    </record>

    <record id="hr_leave_rule_officer_update" model="ir.rule">
        <field name="name">Time Off All Approver create/write</field>
        <field name="model_id" ref="model_hr_leave"/>
        <field name="domain_force">[
            ('holiday_type', '=', 'employee'),
            '|',
                '&amp;',
                    ('employee_id.user_id', '=', user.id),
                    ('state', '!=', 'validate'),
                '|',
                    ('employee_id.user_id', '!=', user.id),
                    ('employee_id.user_id', '=', False)
        ]</field>
        <field name="perm_read" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="groups" eval="[(4, ref('hr_holidays.group_hr_holidays_user'))]"/>
    </record>

    <record id="hr_leave_rule_manager" model="ir.rule">
        <field name="name">Time Off Administrator</field>
        <field name="model_id" ref="model_hr_leave"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('group_hr_holidays_manager'))]"/>
    </record>

    <record id="hr_leave_rule_multicompany" model="ir.rule">
        <field name="name">Time Off: multi company global rule</field>
        <field name="model_id" ref="model_hr_leave"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>

    <record id="hr_leave_allocation_rule_multicompany" model="ir.rule">
        <field name="name">Time Off: multi company global rule</field>
        <field name="model_id" ref="model_hr_leave_allocation"/>
        <field name="domain_force">[
            '|',
                '&amp;',
                    ('employee_id', '=', False),
                    '|',
                        ('employee_ids', '=', False),
                        ('employee_ids.company_id', 'in', company_ids),
                ('employee_id.company_id', 'in', company_ids),
            ('holiday_status_id.company_id', 'in', company_ids + [False])
        ]</field>
    </record>

    <record id="hr_leave_allocation_rule_employee" model="ir.rule">
        <field name="name">Allocations: employee: read own</field>
        <field name="model_id" ref="model_hr_leave_allocation"/>
        <field name="domain_force">[
            '|',
                ('employee_id.leave_manager_id', '=', user.id),
                ('employee_id.user_id', '=', user.id),
        ]</field>
        <field name="perm_create" eval="False"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="groups" eval="[(4,ref('base.group_user'))]"/>
    </record>

    <record id="hr_leave_allocation_rule_employee_update" model="ir.rule">
        <field name="name">Allocations: base.group_user create/write</field>
        <field name="model_id" ref="model_hr_leave_allocation"/>
        <field name="domain_force">[
            ('holiday_status_id.requires_allocation', '=', 'yes'),
            ('holiday_status_id.employee_requests', '=', 'yes'),
            ('holiday_type', '=', 'employee'),
            '|',
                ('employee_id.user_id', '=', user.id),
                '&amp;',
                    ('validation_type', '=', 'officer'),
                    ('employee_id.leave_manager_id', '=', user.id),
        ]</field>
        <field name="perm_read" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="groups" eval="[(4,ref('base.group_user'))]"/>
    </record>

    <record id="hr_leave_allocation_rule_officer_read" model="ir.rule">
        <field name="name">Allocations: see all time off: read all</field>
        <field name="model_id" ref="model_hr_leave_allocation"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="perm_create" eval="False"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="groups" eval="[(4, ref('hr_holidays.group_hr_holidays_user'))]"/>
    </record>

    <record id="hr_leav_allocation_rule_employee_unlink" model="ir.rule">
        <field name="name">Allocations base.group_user unlink</field>
        <field name="model_id" ref="model_hr_leave_allocation"/>
        <field name="domain_force">[('employee_id.user_id', '=', user.id), ('state', '=', 'draft')]</field>
        <field name="perm_read" eval="False"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <record id="hr_leave_allocation_rule_officer_update" model="ir.rule">
        <field name="name">Allocations: holiday user: create/write</field>
        <field name="model_id" ref="model_hr_leave_allocation"/>
        <field name="domain_force">[
            ('holiday_type', '=', 'employee'),
            '|',
                '&amp;',
                    ('employee_id.user_id', '=', user.id),
                    ('state', '!=', 'validate'),
                '|',
                    ('employee_id.user_id', '!=', user.id),
                    ('employee_id.user_id', '=', False)
        ]</field>
        <field name="groups" eval="[(4,ref('hr_holidays.group_hr_holidays_user'))]"/>
    </record>

    <record id="hr_leave_allocation_rule_manager" model="ir.rule">
        <field name="name">Allocations: administrator: no limit</field>
        <field name="model_id" ref="model_hr_leave_allocation"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('group_hr_holidays_manager'))]"/>
    </record>

    <record id="resource_leaves_base_user" model="ir.rule">
        <field name="name">Time Off Resources: Approver</field>
        <field name="model_id" ref="model_resource_calendar_leaves"/>
        <field name="domain_force">[(1,'=',1)]</field>
        <field name="perm_create" eval="False"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <!-- FIXME RLi Maybe this should be restricted somewhat -->
    <record id="resource_leaves_holidays_user" model="ir.rule">
        <field name="name">Time Off Resources: All Approver</field>
        <field name="model_id" ref="model_resource_calendar_leaves"/>
        <field name="domain_force">[(1,'=',1)]</field>
        <field name="groups" eval="[(4, ref('hr_holidays.group_hr_holidays_user'))]"/>
    </record>

    <record id="hr_holidays_status_rule_multi_company" model="ir.rule">
        <field name="name">Time Off multi company rule</field>
        <field name="model_id" ref="model_hr_leave_type"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="hr_leave_accrual_plan_rule_multi_company" model="ir.rule">
        <field name="name">Accrual plan multi company rule</field>
        <field name="model_id" ref="model_hr_leave_accrual_plan"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="hr_leave_mandatory_day_rule_multi_company" model="ir.rule">
        <field name="name">Mandatory Day: multi company rule</field>
        <field name="model_id" ref="model_hr_leave_mandatory_day"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="hr_leave_report_calendar_rule_multi_company" model="ir.rule">
        <field name="name">Time Off Report Calendar: multi company global rule</field>
        <field name="model_id" ref="model_hr_leave_report_calendar"/>
        <field name="global" eval="True"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="hr_leave_report_rule_multi_company" model="ir.rule">
        <field name="name">Time Off Report: multi company global rule</field>
        <field name="model_id" ref="model_hr_leave_report"/>
        <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="hr_leave_report_rule_group_user" model="ir.rule">
        <field name="name">Time Off Summary / Report: Internal User</field>
        <field name="model_id" ref="model_hr_leave_report"/>
        <field name="domain_force">[('employee_id.user_id', '=', user.id)]</field>
        <field name="perm_create" eval="False"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="groups" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <record id="hr_leave_report_rule_group_holiday_user" model="ir.rule">
        <field name="name">Time Off Summary / Report: All Approver</field>
        <field name="model_id" ref="model_hr_leave_report"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="perm_create" eval="False"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_unlink" eval="False"/>
        <field name="groups" eval="[(4, ref('hr_holidays.group_hr_holidays_user'))]"/>
    </record>

    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_holidays_manager_request,hr.holidays.manager.request,model_hr_leave,hr_holidays.group_hr_holidays_manager,1,1,1,1
access_hr_holidays_user_request,hr.holidays.user.request,model_hr_leave,hr_holidays.group_hr_holidays_user,1,1,1,1
access_hr_holidays_employee_request,hr.holidays.employee.request,model_hr_leave,base.group_user,1,1,1,1
access_hr_holidays_manager_allocation,hr.holidays.manager.allocation,model_hr_leave_allocation,hr_holidays.group_hr_holidays_manager,1,1,1,1
access_hr_holidays_user_allocation,hr.holidays.user.allocation,model_hr_leave_allocation,hr_holidays.group_hr_holidays_user,1,1,1,1
access_hr_holidays_employee_allocation,hr.holidays.employee.allocation,model_hr_leave_allocation,base.group_user,1,1,1,1
access_hr_holidays_status_manager,hr.holidays.status manager,model_hr_leave_type,hr_holidays.group_hr_holidays_manager,1,1,1,1
access_hr_holidays_status_user,hr.holidays.status user,model_hr_leave_type,hr_holidays.group_hr_holidays_user,1,0,0,0
access_hr_holidays_status_employee,hr.holidays.status employee,model_hr_leave_type,base.group_user,1,0,0,0
access_hr_leave_report,access_hr_leave_report,model_hr_leave_report,base.group_user,1,0,0,0
access_resource_calendar_leaves_user,resource_calendar_leaves_user,resource.model_resource_calendar_leaves,hr_holidays.group_hr_holidays_user,1,1,1,1
access_calendar_event_hr_user,calendar.event.hr.user,calendar.model_calendar_event,hr_holidays.group_hr_holidays_user,1,1,1,1
access_calendar_event_type_manager,calendar.event.type.manager,calendar.model_calendar_event_type,hr_holidays.group_hr_holidays_manager,1,1,1,1
access_calendar_attendee_hr_user,calendar.attendee.hr.user,calendar.model_calendar_attendee,hr_holidays.group_hr_holidays_user,1,1,1,1
access_mail_activity_type_holidays_manager,mail.activity.type.holidays.manager,mail.model_mail_activity_type,hr_holidays.group_hr_holidays_manager,1,1,1,1
access_hr_holidays_summary_employee,access.hr.holidays.summary.employee,model_hr_holidays_summary_employee,hr_holidays.group_hr_holidays_user,1,1,1,0
access_hr_leave_report_calendar,access_hr_leave_report_calendar,model_hr_leave_report_calendar,base.group_user,1,0,0,0
access_hr_leave_employee_type_report,access_hr_leave_employee_type_report,model_hr_leave_employee_type_report,base.group_user,1,0,0,0
access_hr_leave_accrual_plan_user,hr.leave.accrual.plan.user,model_hr_leave_accrual_plan,hr_holidays.group_hr_holidays_user,1,0,0,0
access_hr_leave_accrual_level_user,hr.leave.accrual.level.user,model_hr_leave_accrual_level,hr_holidays.group_hr_holidays_user,1,0,0,0
access_hr_leave_accrual_plan_manager,hr.leave.accrual.plan.manager,model_hr_leave_accrual_plan,hr_holidays.group_hr_holidays_manager,1,1,1,1
access_hr_leave_accrual_level_manager,hr.leave.accrual.level.manager,model_hr_leave_accrual_level,hr_holidays.group_hr_holidays_manager,1,1,1,1
access_hr_holidays_cancel_leave,access_hr_holidays_cancel_leave,hr_holidays.model_hr_holidays_cancel_leave,base.group_user,1,1,1,1
access_hr_leave_mandatory_day_user,access_hr_holidays_mandatory_day_user,hr_holidays.model_hr_leave_mandatory_day,base.group_user,1,0,0,0
access_hr_leave_mandatory_day_manager,access_hr_holidays_mandatory_day_manager,hr_holidays.model_hr_leave_mandatory_day,hr_holidays.group_hr_holidays_manager,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg width="51" height="50" viewBox="0 0 51 50" xmlns="http://www.w3.org/2000/svg"><path d="M9.096 43.956a25.151 25.151 0 0 1-1.58-1.483c-.726-.743-.67-1.924.065-2.66l25.95-25.95 1.415 1.415a2 2 0 0 1 0 2.828l-25.85 25.85Z" fill="#985184"/><path d="M15.849 15.808 7.363 7.322A25 25 0 0 1 30.948.708l-15.1 15.1Z" fill="#FBB945"/><path d="m15.848 15.807 15.1-15.1a25 25 0 0 1 11.77 6.614L25.04 25l-9.192-9.192Z" fill="#F78613"/><path d="M33.524 33.485 25.04 25 42.717 7.322a24.997 24.997 0 0 1 6.315 10.655L33.524 33.485Z" fill="#088BF5"/><path d="m33.525 33.485 15.508-15.508a25 25 0 0 1-4.96 23.231c-.715.84-1.988.836-2.77.055l-7.778-7.778Z" fill="#2EBCFA"/></svg>

```

## File: static\src\avatar_card_popover_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.AvatarCardPopover" t-inherit-mode="extension">
        <xpath expr="//span[@name='icon']" position="inside">
            <i t-elif="user.im_status === 'leave_online'" class="fa fa-fw fa-plane text-success" title="Online" role="img" aria-label="User is online"/>
            <i t-elif="user.im_status === 'leave_away'" class="fa fa-fw fa-plane text-warning" title="Idle" role="img" aria-label="User is idle"/>
            <i t-elif="user.im_status === 'leave_offline'" class="fa fa-fw fa-plane text-700" title="Out of office" role="img" aria-label="User is out of office"/>
        </xpath>
    </t>
</templates>

```

## File: static\src\im_status_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.ImStatus" t-inherit-mode="extension">
        <xpath expr="//*[@name='icon']" position="replace">
            <i t-if="props.persona.im_status === 'leave_online'" class="fa fa-plane text-success" title="Online" role="img" aria-label="User is online"/>
            <i t-elif="props.persona.im_status === 'leave_away'" class="fa fa-plane o-away" title="Idle" role="img" aria-label="User is idle"/>
            <i t-elif="props.persona.im_status === 'leave_offline'" class="fa fa-plane text-700" title="Out of office" role="img" aria-label="User is out of office"/>
            <t t-else="">$0</t>
        </xpath>
    </t>
</templates>

```

## File: static\src\persona_service_patch.js

```javascript
/* @odoo-module */

import { personaService, PersonaService } from "@mail/core/common/persona_service";
import { deserializeDateTime } from "@web/core/l10n/dates";
import { _t } from "@web/core/l10n/translation";
import { patch } from "@web/core/utils/patch";

const { DateTime } = luxon;

patch(personaService, {
    dependencies: [...new Set([...personaService.dependencies, "user"])],
});

patch(PersonaService.prototype, {
    setup(env, services) {
        super.setup(env, services);
        this.userService = services.user;
    },

    outOfOfficeText(persona) {
        if (!persona.out_of_office_date_end) {
            return "";
        }
        const currentDate = new Date();
        const date = deserializeDateTime(persona.out_of_office_date_end);
        // const options = { day: "numeric", month: "short" };
        if (currentDate.getFullYear() !== date.year) {
            // options.year = "numeric";
        }
        let localeCode = this.userService.lang.replace(/_/g, "-");
        if (localeCode === "sr@latin") {
            localeCode = "sr-Latn-RS";
        }
        // const fdate = date.toLocaleString(DateTime.TIME_SHORT);
        const fdate = date.toLocaleString(DateTime.DATE_MED);
        // const formattedDate = date.toLocaleDateString(localeCode, options);
        return _t("Out of office until %s", fdate);
    },
});

```

## File: static\src\thread_icon.patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.ThreadIcon" t-inherit-mode="extension">
        <xpath expr="//*[@name='chat_static']" position="replace">
            <div t-if="chatPartner.im_status === 'leave_online'" class="o-mail-ThreadIcon-online fa fa-fw fa-plane text-success" title="Online"/>
            <div t-elif="chatPartner.im_status === 'leave_offline'" class="fa fa-fw fa-plane" title="Out of office"/>
            <div t-elif="chatPartner.im_status === 'leave_away'" class="fa fa-fw fa-plane o-yellow" title="Away"/>
            <t t-else="">$0</t>
        </xpath>
    </t>
</templates>

```

## File: static\src\thread_model_patch.js

```javascript
/* @odoo-module */

import { Thread } from "@mail/core/common/thread_model";
import { patch } from "@web/core/utils/patch";

patch(Thread, {
    get onlineMemberStatuses() {
        return super.onlineMemberStatuses + ["leave_online", "leave_away"];
    },
});

```

## File: static\src\thread_patch.js

```javascript
/* @odoo-module */

import { Thread } from "@mail/core/common/thread";
import { useService } from "@web/core/utils/hooks";
import { patch } from "@web/core/utils/patch";

patch(Thread.prototype, {
    setup() {
        super.setup();
        this.personaService = useService("mail.persona");
    },
});

```

## File: static\src\thread_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.Thread" t-inherit-mode="extension">
        <xpath expr="//*[hasclass('o-mail-Thread')]" position="before">
            <div t-if="props.thread.model === 'discuss.channel' and props.thread.correspondent and personaService.outOfOfficeText(props.thread.correspondent)" class="alert alert-primary rounded-0" t-esc="personaService.outOfOfficeText(props.thread.correspondent)" role="alert"/>
        </xpath>
    </t>
</templates>

```

## File: static\src\components\hr_presence_status\hr_presence_status.js

```javascript
/** @odoo-module **/

import { patch } from "@web/core/utils/patch";

import { HrPresenceStatus } from "@hr/components/hr_presence_status/hr_presence_status";
import { HrPresenceStatusPrivate, hrPresenceStatusPrivate } from "@hr/components/hr_presence_status_private/hr_presence_status_private";

const patchHrPresenceStatus = () => ({
    get icon() {
        if (this.value.startsWith("presence_holiday")) {
            return "fa-plane";
        }
        return super.icon;
    },

    get color() {
        if (this.value.startsWith("presence_holiday")) {
            return `text-${this.value === "presence_holiday_present" ? "success" : "warning"}`;
        }
        return super.color;
    },
});

// Applies common patch on both components
patch(HrPresenceStatus.prototype, patchHrPresenceStatus());
patch(HrPresenceStatusPrivate.prototype, patchHrPresenceStatus());

// Applies patch to hr_presence_status_private to display the time off type instead of default label
patch(HrPresenceStatusPrivate.prototype, {
    get label() {
        return this.props.record.data.current_leave_id
            ? this.props.record.data.current_leave_id[1]
            : super.label;
    }
});

Object.assign(hrPresenceStatusPrivate, {
    fieldDependencies: [
        ...(hrPresenceStatusPrivate.fieldDependencies || []),
        { name: "current_leave_id", type:"many2one"}
    ],
});

```

## File: static\src\dashboard\time_off_card.js

```javascript
/* @odoo-module */

import { usePopover } from "@web/core/popover/popover_hook";
import { formatNumber, useNewAllocationRequest } from "@hr_holidays/views/hooks";
import { useService } from "@web/core/utils/hooks";
import { Component, onWillRender } from "@odoo/owl";

export class TimeOffCardPopover extends Component {
    setup() {
        this.actionService = useService("action");
    }

    async openLeaves() {
        this.actionService.doAction({
            type: "ir.actions.act_window",
            res_model: "hr.leave",
            views: [
                [false, "list"],
                [false, "form"],
            ],
            domain: [["id", "in", this.props.errorLeaves]],
        });
    }
}

TimeOffCardPopover.template = "hr_holidays.TimeOffCardPopover";
TimeOffCardPopover.props = [
    "allocated",
    "accrual_bonus",
    "approved",
    "planned",
    "left",
    "warning",
    "closest",
    "request_unit",
    "exceeding_duration",
    "close?",
    "allows_negative",
    "max_allowed_negative",
    "onClickNewAllocationRequest?",
    "errorLeaves",
    "accrualExcess",
];

export class TimeOffCard extends Component {
    setup() {
        this.popover = usePopover(TimeOffCardPopover, {
            position: "bottom",
            popoverClass: "bg-view",
        });
        this.newAllocationRequest = useNewAllocationRequest();
        this.lang = this.env.services.user.lang;
        this.formatNumber = formatNumber;
        const { data } = this.props;
        this.errorLeaves = Object.values(data.virtual_excess_data).map((data) => data.leave_id);
        this.errorLeavesDuration = Object.values(data.virtual_excess_data).reduce(
            (acc, data) => acc + data.amount,
            0
        );
        this.updateWarning();

        onWillRender(this.updateWarning);
    }

    updateWarning() {
        const { data } = this.props;
        const errorLeavesSignificant = data.allows_negative
            ? this.errorLeavesDuration > data.max_allowed_negative
            : this.errorLeavesDuration > 0;
        const accrualExcess = this.getAccrualExcess(data);
        const closeExpire =
            data.closest_allocation_duration &&
            data.closest_allocation_duration < data.virtual_remaining_leaves;
        this.warning = errorLeavesSignificant || accrualExcess || closeExpire;
    }

    onClickInfo(ev) {
        const { data } = this.props;
        this.popover.open(ev.target, {
            allocated: formatNumber(this.lang, data.max_leaves),
            accrual_bonus: formatNumber(this.lang, data.accrual_bonus),
            approved: formatNumber(this.lang, data.leaves_approved),
            planned: formatNumber(this.lang, data.leaves_requested),
            left: formatNumber(this.lang, data.virtual_remaining_leaves),
            warning: this.warning,
            closest: data.closest_allocation_duration,
            request_unit: data.request_unit,
            exceeding_duration: data.exceeding_duration,
            allows_negative: data.allows_negative,
            max_allowed_negative: data.max_allowed_negative,
            onClickNewAllocationRequest: this.newAllocationRequestFrom.bind(this),
            errorLeaves: this.errorLeaves,
            accrualExcess: this.getAccrualExcess(data),
        });
    }

    getAccrualExcess(data) {
        return data.allows_negative
            ? -data.exceeding_duration > data.max_allowed_negative
            : -data.exceeding_duration > 0;
    }

    async newAllocationRequestFrom() {
        this.popover.close();
        await this.newAllocationRequest(this.props.employeeId, this.props.holidayStatusId);
    }
}

TimeOffCard.template = "hr_holidays.TimeOffCard";
TimeOffCard.props = ["name", "data", "requires_allocation", "employeeId", "holidayStatusId"];

export class TimeOffCardMobile extends TimeOffCard {}

TimeOffCardMobile.template = "hr_holidays.TimeOffCardMobile";

```

## File: static\src\dashboard\time_off_card.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <div t-name="hr_holidays.TimeOffCard" class="o_timeoff_card py-3 text-primary">
        <t t-set="data" t-value="props.data"/>
        <t t-set="duration" t-value="props.requires_allocation ? data.virtual_remaining_leaves : data.virtual_leaves_taken"/>
        <t t-set="parsed_duration" t-value="formatNumber(this.lang, duration)"/>
        <t t-set="show_popover" t-value="true"/>

        <strong class="o_timeoff_name"><t t-esc="props.name"/></strong>
        <span class="o_timeoff_duration">
            <t t-if="data and data.icon">
                <img t-att-src="data.icon" />
            </t>
            <span t-att-style="data.holds_changes ? 'color: #B6D369;' : ''">
                <t t-esc="parsed_duration"/>
            </span>
        </span>
        <span
            t-att-class="'text-uppercase o_timeoff_details p-1' + (warning ? ' alert alert-warning' : '')"
            t-on-click.stop="(ev) => (show_popover &amp;&amp; this.onClickInfo(ev))">
            <t t-if="data.request_unit == 'hour'" name="duration_unit">hours</t>
            <t t-else="">days</t>
            <t t-if="props.requires_allocation" name="duration_type">
                available
            </t>
            <t t-else="">
                taken
            </t>
            <span t-if="show_popover"
                t-att-class="'o_timeoff_info fa' + (warning ? ' fa-exclamation-triangle' : ' fa-question-circle-o')"
            />
        </span>
        <span t-if="props.requires_allocation and data.closest_allocation_expire !== false" class="text-uppercase o_timeoff_validity">
            <t t-if="data.closest_allocation_remaining != data.virtual_remaining_leaves">
                (<t t-esc="data.closest_allocation_remaining"/> <t t-if="data.request_unit == 'hour'">hours</t><t t-else="">days</t>
                valid until <span t-esc="data.closest_allocation_expire"/>)
            </t>
            <t t-else="">
                (valid until <span t-esc="data.closest_allocation_expire"/>)
            </t>
        </span>
    </div>

    <t t-name="hr_holidays.TimeOffCardMobile">
        <t t-set="data" t-value="props.data"/>
        <t t-set="duration" t-value="props.requires_allocation ? data.virtual_remaining_leaves : data.virtual_leaves_taken" />

        <span class="float-end o_timeoff_card_mobile">
            <t t-if="props.requires_allocation" name="duration_type">
                <strong t-esc="duration" class="o_timeoff_green text-success"/> / <span t-esc="data.max_leaves"/> <t t-if="data.request_unit == 'hour'">Hours</t><t t-else="">Days</t> <span class="o_timeoff_green text-success">Available</span>
            </t>
            <t t-else="">
                <strong t-esc="duration"/> <t t-if="data.request_unit == 'hour'">Hours</t><t t-else="">Days</t> <span class="text-primary">Taken</span>
            </t>
        </span>
    </t>

    <t t-name="hr_holidays.TimeOffCardPopover">
        <ul class="list-unstyled p-3 mb-0">
            <li class="d-flex justify-content-between">
                <span>Allocated (<span class="btn-link p-0 cursor-pointer" t-on-click="props.onClickNewAllocationRequest">new request</span>):</span>
                <span class="ps-1" t-esc="props.allocated"/>
            </li>
            <li class="d-flex justify-content-between" t-if="props.accrual_bonus">
                Accrual (Future): <span class="ps-1" t-esc="props.accrual_bonus"/>
            </li>
            <li class="d-flex justify-content-between">Approved: <span class="ps-1" t-esc="props.approved"/></li>
            <li class="d-flex justify-content-between border-bottom">Planned: <span class="ps-1" t-esc="props.planned"/></li>
            <li class="d-flex justify-content-between">Available: <span class="ps-1" t-esc="props.left"/></li>
        </ul>
        <div t-if="props.warning" class="alert alert-warning mb-0 pb-0 o_time_off_card_popover_warning">
            <span class="d-inline-block m-0 mb-3"
                t-if="props.errorLeaves.length">
                Some leaves cannot be linked to any allocation. To see those leaves, 
                <a t-on-click="() => this.openLeaves()" class="cursor-pointer">click here</a>.
            </span>
            <span class="d-inline-block m-0 mb-3"
                t-if="props.closest &amp;&amp; props.closest &lt; props.left">
                <i class="fa fa-warning"/> Only <t t-esc="props.closest"/>
                <t t-if="props.request_unit == 'hour'"> hours</t>
                <t t-else=""> days</t>
                can be used before the allocation expires.
            </span>
            <span class="d-inline-block m-0 mb-3"
                t-if="props.accrualExcess">
                The leaves planned in the future are exceeding the maximum value of the allocation.
                It will not be possible to take all of them.
            </span>
        </div>
    </t>
</templates>

```

## File: static\src\dashboard\time_off_dashboard.js

```javascript
/* @odoo-module */

import { TimeOffCard } from "./time_off_card";
import { useNewAllocationRequest } from "@hr_holidays/views/hooks";
import { useBus, useService } from "@web/core/utils/hooks";
import { DateTimeInput } from "@web/core/datetime/datetime_input";
import { Component, useState, onWillStart } from "@odoo/owl";

export class TimeOffDashboard extends Component {
    setup() {
        this.orm = useService("orm");
        this.newRequest = useNewAllocationRequest();
        this.state = useState({
            date: luxon.DateTime.now(),
            today: luxon.DateTime.now(),
            holidays: [],
        });
        useBus(this.env.timeOffBus, "update_dashboard", async () => {
            await this.loadDashboardData();
        });

        onWillStart(async () => {
            await this.loadDashboardData();
            const context = {};
            if (this.props && this.props.employeeId !== null) {
                context["employee_id"] = this.props.employeeId;
            }
            this.hasAccrualAllocation = await this.orm.call(
                "hr.leave.type",
                "has_accrual_allocation",
                [],
                {
                    context: context,
                }
            );
        });
    }

    async loadDashboardData(date = false) {
        const context = { from_dashboard: true };
        if (this.props && this.props.employeeId !== null) {
            context["employee_id"] = this.props.employeeId;
        }
        if (date) {
            this.state.date = date;
        }
        this.state.holidays = await this.orm.call(
            "hr.leave.type",
            "get_allocation_data_request",
            [this.state.date],
            {
                context: context,
            }
        );
    }

    async newAllocationRequest() {
        await this.newRequest(this.props.employeeId);
    }

    resetDate() {
        this.state.date = luxon.DateTime.now();
        this.loadDashboardData();
    }
}

TimeOffDashboard.components = { TimeOffCard, DateTimeInput };
TimeOffDashboard.template = "hr_holidays.TimeOffDashboard";
TimeOffDashboard.props = ["employeeId"];

```

## File: static\src\dashboard\time_off_dashboard.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <div t-name="hr_holidays.TimeOffDashboard" class="o_timeoff_dashboard">
        <t t-foreach="state.holidays" t-as="holiday" t-key="holiday[3]">
            <TimeOffCard
                name="holiday[0]"
                data="holiday[1]"
                requires_allocation="holiday[2] == 'yes'"
                holidayStatusId="holiday[3]"
                employeeId="props.employeeId"/>
        </t>
        <div class="o_timeoff_card p-0 d-flex justify-content-around">
            <div class="row justify-content-center align-items-center border-bottom h-50 w-100 p-1" t-if="hasAccrualAllocation">
                Balance at the
                <div class="p-1" style="max-width: 100px!important">
                    <DateTimeInput
                        type="'date'"
                        value="state.date"
                        onChange="(date) => this.loadDashboardData(date)"
                        minDate="state.today"
                        placeholder="'Today'"/>
                </div>
                <button class="o_timeoff_today_button btn btn-secondary" t-on-click="resetDate">Today</button>
            </div>
            <div class="row justify-content-center align-items-center h-50 w-100">
                <button class="btn btn-secondary m-auto" t-on-click="newAllocationRequest" type="button" style="width: fit-content;">
                    <t t-if="employeeId">Grant Time</t>
                    <t t-else="">New Allocation Request</t>
                </button>
            </div>
        </div>
    </div>
</templates>

```

## File: static\src\img\icons\Annual_Time_Off.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Annual_Time_Off" data-name="Annual Time Off">
    <g>
      <rect x="13.34" y="24.39" width="73.32" height="59.4" rx="6.37" stroke-width="5" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" fill="none"/>
      <g>
        <path d="M30.5,32.12V16.21a2.5,2.5,0,0,0-5,0V32.12a2.5,2.5,0,0,0,5,0Z" fill="#875b7b"/>
        <path d="M52.5,32.12V16.21a2.5,2.5,0,0,0-5,0V32.12a2.5,2.5,0,1,0,5,0Z" fill="#875b7b"/>
        <path d="M74.5,32.12V16.21a2.5,2.5,0,0,0-5,0V32.12a2.5,2.5,0,1,0,5,0Z" fill="#875b7b"/>
      </g>
      <path d="M13.35,43.32h73.3c3.22,0,3.22-5,0-5H13.35c-3.22,0-3.22,5,0,5Z" fill="#875b7b"/>
      <g>
        <polyline points="28.57 55.08 36.01 55.08 36.01 69.96 28.56 69.96" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="4"/>
        <g>
          <line x1="28.56" y1="62.47" x2="35.54" y2="62.47" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="4"/>
          <polyline points="46.29 62.47 53.73 62.47 53.73 69.96 46.29 69.96 46.29 55.08 53.74 55.08" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="4"/>
          <polyline points="71.44 55.08 64 55.08 64 62.47 71.44 62.47 71.44 69.96 64 69.96" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="4"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Annual_Time_Off_2.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Annual_Time_Off" data-name="Annual Time Off">
    <g>
      <rect x="13.34" y="24.39" width="73.32" height="59.4" rx="6.37" stroke-width="5" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" fill="none"/>
      <g>
        <path d="M30.5,32.12V16.21a2.5,2.5,0,0,0-5,0V32.12a2.5,2.5,0,0,0,5,0Z" fill="#875b7b"/>
        <path d="M52.5,32.12V16.21a2.5,2.5,0,0,0-5,0V32.12a2.5,2.5,0,1,0,5,0Z" fill="#875b7b"/>
        <path d="M74.5,32.12V16.21a2.5,2.5,0,0,0-5,0V32.12a2.5,2.5,0,1,0,5,0Z" fill="#875b7b"/>
      </g>
      <g>
        <path d="M86.65,40.82H13.35v36.6l3.32,6.21,63.63.16,7.62-3.5ZM38,70a2,2,0,0,1-2,2H28.56a2,2,0,0,1,0-4H34V64.47H28.56a2,2,0,0,1,0-4H34V57.08H28.57a2,2,0,0,1,0-4H36a2,2,0,0,1,2,2Zm15.72-9.49a2,2,0,0,1,2,2V70a2,2,0,0,1-2,2H46.29a2,2,0,0,1-2-2V55.08a2,2,0,0,1,2-2h7.45a2,2,0,0,1,0,4H48.29v3.39Zm17.71,0a2,2,0,0,1,2,2V70a2,2,0,0,1-2,2H64a2,2,0,1,1,0-4h5.44V64.47H64a2,2,0,0,1-2-2V55.08a2,2,0,0,1,2-2h7.44a2,2,0,0,1,0,4H66v3.39Z" fill="#875b7b"/>
        <rect x="48.29" y="64.47" width="3.44" height="3.49" fill="#875b7b"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Annual_Time_Off_3.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Annual_Time_Off" data-name="Annual Time Off">
    <g>
      <g>
        <path d="M30.5,32.12V16.21a2.5,2.5,0,0,0-5,0V32.12a2.5,2.5,0,0,0,5,0Z" fill="#875b7b"/>
        <path d="M52.5,32.12V16.21a2.5,2.5,0,0,0-5,0V32.12a2.5,2.5,0,1,0,5,0Z" fill="#875b7b"/>
        <path d="M74.5,32.12V16.21a2.5,2.5,0,0,0-5,0V32.12a2.5,2.5,0,1,0,5,0Z" fill="#875b7b"/>
      </g>
      <g>
        <path d="M36,72H28.56a2,2,0,0,1,0-4H34V57.08H28.57a2,2,0,0,1,0-4H36a2,2,0,0,1,2,2V70A2,2,0,0,1,36,72Z" fill="#875b7b"/>
        <g>
          <path d="M35.54,64.47h-7a2,2,0,0,1,0-4h7a2,2,0,0,1,0,4Z" fill="#875b7b"/>
          <path d="M53.73,72H46.29a2,2,0,0,1-2-2V55.08a2,2,0,0,1,2-2h7.45a2,2,0,0,1,0,4H48.29v3.39h5.44a2,2,0,0,1,2,2V70A2,2,0,0,1,53.73,72Zm-5.44-4h3.44V64.47H48.29Z" fill="#875b7b"/>
          <path d="M71.44,72H64a2,2,0,1,1,0-4h5.44V64.47H64a2,2,0,0,1-2-2V55.08a2,2,0,0,1,2-2h7.44a2,2,0,0,1,0,4H66v3.39h5.44a2,2,0,0,1,2,2V70A2,2,0,0,1,71.44,72Z" fill="#875b7b"/>
        </g>
      </g>
      <path d="M80.29,21.89H79.5V32.12a7.53,7.53,0,0,1-7.27,7.49h-.32a7.56,7.56,0,0,1-7.41-7.5V21.89h-7V32.12a7.53,7.53,0,0,1-7.27,7.49h-.32a7.56,7.56,0,0,1-7.41-7.5V21.89h-7V32.12a7.53,7.53,0,0,1-7.27,7.49h-.32a7.56,7.56,0,0,1-7.41-7.5V21.89h-.79a8.88,8.88,0,0,0-8.87,8.87V77.42a8.88,8.88,0,0,0,8.87,8.87H80.29a8.88,8.88,0,0,0,8.87-8.87V30.76A8.88,8.88,0,0,0,80.29,21.89Zm3.87,55.53a3.87,3.87,0,0,1-3.87,3.87H19.71a3.87,3.87,0,0,1-3.87-3.87V43.32H84.16Z" fill="#875b7b"/>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Compensatory_Time_Off.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Compensatory_Time_Off" data-name="Compensatory Time Off">
    <g>
      <line x1="23.12" y1="26.88" x2="76.88" y2="26.88" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <line x1="23.12" y1="84.74" x2="76.88" y2="84.74" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <line x1="50" y1="15.26" x2="50" y2="84.66" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <line x1="23.45" y1="27.4" x2="11.73" y2="58.24" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <line x1="35.24" y1="57.87" x2="23.52" y2="27.03" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <path d="M23.92,75.75a16.87,16.87,0,0,0,16.55-13.6,2.67,2.67,0,0,0-2.6-3.2H10a2.67,2.67,0,0,0-2.6,3.2A16.87,16.87,0,0,0,23.92,75.75Z" fill="none" stroke="#875b7b" stroke-miterlimit="10" stroke-width="5"/>
      <line x1="75.62" y1="27.4" x2="63.9" y2="58.24" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <line x1="87.41" y1="57.87" x2="75.69" y2="27.03" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <path d="M76.08,75.75a16.87,16.87,0,0,0,16.56-13.6A2.67,2.67,0,0,0,90,59H62.13a2.67,2.67,0,0,0-2.6,3.2A16.87,16.87,0,0,0,76.08,75.75Z" fill="none" stroke="#875b7b" stroke-miterlimit="10" stroke-width="5"/>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Compensatory_Time_Off_2.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Compensatory_Time_Off" data-name="Compensatory Time Off">
    <g>
      <line x1="23.12" y1="26.88" x2="76.88" y2="26.88" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <line x1="23.12" y1="84.74" x2="76.88" y2="84.74" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <line x1="50" y1="15.26" x2="50" y2="84.66" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <line x1="23.45" y1="27.4" x2="11.73" y2="61.24" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <line x1="35.24" y1="60.87" x2="23.52" y2="27.03" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <path d="M23.92,75.75a16.87,16.87,0,0,0,16.55-13.6,2.67,2.67,0,0,0-2.6-3.2H10a2.67,2.67,0,0,0-2.6,3.2A16.87,16.87,0,0,0,23.92,75.75Z" fill="#875b7b"/>
      <line x1="75.62" y1="27.4" x2="63.9" y2="61.24" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <line x1="87.41" y1="60.87" x2="75.69" y2="27.03" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <path d="M76.08,75.75a16.87,16.87,0,0,0,16.56-13.6A2.67,2.67,0,0,0,90,59H62.13a2.67,2.67,0,0,0-2.6,3.2A16.87,16.87,0,0,0,76.08,75.75Z" fill="#875b7b"/>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Compensatory_Time_Off_3.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Compensatory_Time_Off" data-name="Compensatory Time Off">
    <g>
      <polyline points="23.12 66.91 23.12 29.88 76.88 19.88 76.88 59.29" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <line x1="23.12" y1="84.74" x2="76.88" y2="84.74" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <line x1="50" y1="15.26" x2="50" y2="84.66" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <path d="M23.92,75.75a16.87,16.87,0,0,0,16.55-13.6,2.67,2.67,0,0,0-2.6-3.2H10a2.67,2.67,0,0,0-2.6,3.2A16.87,16.87,0,0,0,23.92,75.75Z" fill="#875b7b"/>
      <path d="M76.08,65.75a16.87,16.87,0,0,0,16.56-13.6A2.67,2.67,0,0,0,90,49H62.13a2.67,2.67,0,0,0-2.6,3.2A16.87,16.87,0,0,0,76.08,65.75Z" fill="#875b7b"/>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Credit_Time.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Credit_Time" data-name="Credit Time">
    <g>
      <path d="M50,88.84A36.55,36.55,0,1,1,86.55,52.29,36.59,36.59,0,0,1,50,88.84Zm0-68.09A31.55,31.55,0,1,0,81.55,52.29,31.59,31.59,0,0,0,50,20.75Z" fill="#875b7b"/>
      <path d="M41.35,16.66h17.3a2.5,2.5,0,0,0,0-5H41.35a2.5,2.5,0,0,0,0,5Z" fill="#875b7b"/>
      <path d="M41.35,13.66h17.3a2.5,2.5,0,0,0,0-5H41.35a2.5,2.5,0,0,0,0,5Z" fill="#875b7b"/>
      <path d="M70.66,22.86l2.23,2.23a1.92,1.92,0,0,0,.8.52,2.32,2.32,0,0,0,1.93,0,2,2,0,0,0,.81-.52l.39-.51a2.5,2.5,0,0,0,.34-1.26l-.09-.66a2.5,2.5,0,0,0-.64-1.1l-2.24-2.24a1.83,1.83,0,0,0-.8-.51,2.23,2.23,0,0,0-1.93,0,1.83,1.83,0,0,0-.8.51l-.39.51a2.43,2.43,0,0,0-.34,1.26l.08.67a2.52,2.52,0,0,0,.65,1.1Z" fill="#875b7b"/>
      <path d="M60.78,36.56a13.49,13.49,0,0,0-8.6-3.94V29.87c0-3.21-5-3.22-5,0V33A11.57,11.57,0,0,0,38.43,43.3a11.44,11.44,0,0,0,8.75,11.9V68.42a8.48,8.48,0,0,1-4.42-2.36,2.18,2.18,0,0,1-.64-1.55V61.46A2.5,2.5,0,0,0,39.64,59h0a2.51,2.51,0,0,0-2.5,2.49v3a7.25,7.25,0,0,0,2.1,5.11,13.49,13.49,0,0,0,8,3.87v1.24c0,3.22,5,3.22,5,0V73.34a11.59,11.59,0,0,0,9.39-10.49,11.46,11.46,0,0,0-9.39-12V37.64a8.5,8.5,0,0,1,5.06,2.45,2.15,2.15,0,0,1,.64,1.55v3a2.49,2.49,0,0,0,2.49,2.51h0a2.51,2.51,0,0,0,2.5-2.49V41.66A7.21,7.21,0,0,0,60.78,36.56ZM45.16,48.49a6.4,6.4,0,0,1,2-10.24V50A6.52,6.52,0,0,1,45.16,48.49Zm11.42,14a6.5,6.5,0,0,1-4.4,5.63V56a6.49,6.49,0,0,1,4.4,6.58Z" fill="#875b7b"/>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Credit_Time_2.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Credit_Time" data-name="Credit Time">
    <path d="M50,88.84A36.55,36.55,0,1,1,86.55,52.29,36.59,36.59,0,0,1,50,88.84Zm0-68.09A31.55,31.55,0,1,0,81.55,52.29,31.59,31.59,0,0,0,50,20.75Z" fill="#875b7b"/>
    <path d="M41.35,16.66h17.3a2.5,2.5,0,0,0,0-5H41.35a2.5,2.5,0,0,0,0,5Z" fill="#875b7b"/>
    <path d="M41.35,13.66h17.3a2.5,2.5,0,0,0,0-5H41.35a2.5,2.5,0,0,0,0,5Z" fill="#875b7b"/>
    <path d="M70.66,22.86l2.23,2.23a1.92,1.92,0,0,0,.8.52,2.32,2.32,0,0,0,1.93,0,2,2,0,0,0,.81-.52l.39-.51a2.5,2.5,0,0,0,.34-1.26l-.09-.66a2.5,2.5,0,0,0-.64-1.1l-2.24-2.24a1.83,1.83,0,0,0-.8-.51,2.23,2.23,0,0,0-1.93,0,1.83,1.83,0,0,0-.8.51l-.39.51a2.43,2.43,0,0,0-.34,1.26l.08.67a2.52,2.52,0,0,0,.65,1.1Z" fill="#875b7b"/>
    <g>
      <path d="M43.42,43.62A6.5,6.5,0,0,0,47.18,50V38.25A6.47,6.47,0,0,0,43.42,43.62Z" fill="#875b7b"/>
      <path d="M52.18,68.16a6.44,6.44,0,0,0,0-12.21Z" fill="#875b7b"/>
      <path d="M50,24.51A27.49,27.49,0,1,0,77.49,52,27.49,27.49,0,0,0,50,24.51Zm12.87,20.2a2.51,2.51,0,0,1-2.5,2.49h0a2.49,2.49,0,0,1-2.49-2.51v-3a2.15,2.15,0,0,0-.64-1.55,8.5,8.5,0,0,0-5.06-2.45V50.81a11.45,11.45,0,0,1,0,22.53v1.37c0,3.22-5,3.22-5,0V73.47a13.49,13.49,0,0,1-8-3.87,7.25,7.25,0,0,1-2.1-5.11v-3A2.51,2.51,0,0,1,39.63,59h0a2.5,2.5,0,0,1,2.49,2.51v3.05a2.18,2.18,0,0,0,.64,1.55,8.48,8.48,0,0,0,4.42,2.36V55.2a11.43,11.43,0,0,1,0-22.23v-3.1c0-3.22,5-3.21,5,0v2.75a13.49,13.49,0,0,1,8.6,3.94,7.21,7.21,0,0,1,2.1,5.1Z" fill="#875b7b"/>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Extra_Time_Off.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Extra_Time_Off" data-name="Extra Time Off">
    <g>
      <path d="M56.9,12.07a2.73,2.73,0,0,1,2.79,2.63l.46,9.13A2.77,2.77,0,0,0,64.5,26L72,20.75a2.71,2.71,0,0,1,1.58-.52A2.78,2.78,0,0,1,76,24.28l-4.16,8.13a2.77,2.77,0,0,0,2.45,4h.24l9.11-.76.26,0a2.77,2.77,0,0,1,1.47,5.1l-7.67,5a2.77,2.77,0,0,0,.33,4.83l8.26,3.89a2.76,2.76,0,0,1-1,5.26l-9.13.46A2.77,2.77,0,0,0,74,64.5L79.25,72A2.78,2.78,0,0,1,77,76.36,2.82,2.82,0,0,1,75.72,76l-8.13-4.16a2.82,2.82,0,0,0-1.26-.31,2.77,2.77,0,0,0-2.76,3l.76,9.11a2.74,2.74,0,0,1-2.79,3,2.67,2.67,0,0,1-2.29-1.27l-5-7.67A2.72,2.72,0,0,0,52,76.48a2.75,2.75,0,0,0-2.51,1.59l-3.89,8.26a2.66,2.66,0,0,1-2.47,1.6,2.73,2.73,0,0,1-2.79-2.63l-.46-9.13A2.77,2.77,0,0,0,35.5,74L28,79.25a2.71,2.71,0,0,1-1.58.52,2.78,2.78,0,0,1-2.46-4l4.16-8.13a2.77,2.77,0,0,0-2.45-4h-.24l-9.11.76-.26,0a2.77,2.77,0,0,1-1.47-5.1l7.67-5a2.77,2.77,0,0,0-.33-4.83l-8.26-3.89a2.76,2.76,0,0,1,1-5.26l9.13-.46A2.77,2.77,0,0,0,26,35.5L20.75,28A2.78,2.78,0,0,1,23,23.64a2.82,2.82,0,0,1,1.28.32l8.13,4.16a2.76,2.76,0,0,0,4-2.69l-.76-9.11a2.74,2.74,0,0,1,2.79-3,2.67,2.67,0,0,1,2.29,1.27l5,7.67A2.72,2.72,0,0,0,48,23.52a2.75,2.75,0,0,0,2.51-1.59l3.89-8.26a2.66,2.66,0,0,1,2.47-1.6m0-4h0A6.67,6.67,0,0,0,50.81,12l-3,6.26L44.1,12.41a6.69,6.69,0,0,0-5.64-3.09,6.88,6.88,0,0,0-5.05,2.2,6.67,6.67,0,0,0-1.73,5.14l.58,6.89L26.1,20.4a6.67,6.67,0,0,0-3.1-.76,6.78,6.78,0,0,0-6,3.67,6.65,6.65,0,0,0,.46,7L21.41,36l-6.91.35A6.77,6.77,0,0,0,12,49.19l6.26,3L12.41,55.9a6.77,6.77,0,0,0,3.65,12.45l.6,0,6.89-.58L20.4,73.9a6.68,6.68,0,0,0,.22,6.58,6.85,6.85,0,0,0,5.8,3.29,6.75,6.75,0,0,0,3.86-1.23l5.68-4,.35,6.91A6.76,6.76,0,0,0,49.19,88l3-6.26,3.76,5.81a6.69,6.69,0,0,0,5.64,3.09,6.88,6.88,0,0,0,5.05-2.2,6.68,6.68,0,0,0,1.73-5.14l-.58-6.89L73.9,79.6a6.67,6.67,0,0,0,3.1.76,6.78,6.78,0,0,0,6-3.67,6.65,6.65,0,0,0-.46-7l-4-5.68,6.91-.35A6.77,6.77,0,0,0,88,50.81l-6.26-3,5.81-3.76a6.77,6.77,0,0,0-3.65-12.45l-.6,0-6.89.58L79.6,26.1a6.68,6.68,0,0,0-.22-6.58,6.85,6.85,0,0,0-5.8-3.29,6.75,6.75,0,0,0-3.86,1.23L64,21.41l-.35-6.91A6.71,6.71,0,0,0,56.9,8.07Z" fill="#875b7b"/>
      <g>
        <path d="M34.89,56.11a1.26,1.26,0,0,0,.35-1.73A1.28,1.28,0,0,0,33.5,54l-2.75,1.84a1.23,1.23,0,0,0-.41.49,1.08,1.08,0,0,0-.06,1,1,1,0,0,0,.11.2s0,0,0,0l5.13,7.64a1.17,1.17,0,0,0,1.61.37,1.73,1.73,0,0,0,.24-.1L40,63.75a1.25,1.25,0,0,0-1.39-2.08l-1.72,1.15L35.66,61l1.69-1.13A1.25,1.25,0,0,0,36,57.75l-1.68,1.13-1.09-1.62Z" fill="#875b7b"/>
        <path d="M41.05,49.06a1.51,1.51,0,0,0-1,1.85v0h0a1.5,1.5,0,0,0-1.84,2.36l3.22,2.52,1.1,3.93a1.5,1.5,0,0,0,1.85,1,1.48,1.48,0,0,0,1-1.85h0a1.5,1.5,0,0,0,1.76.06,1.32,1.32,0,0,0,.35-.32,1.51,1.51,0,0,0-.26-2.11L44,54,42.9,50.1A1.51,1.51,0,0,0,41.05,49.06Z" fill="#875b7b"/>
        <path d="M48.13,44.21l-2.79,1.87A1.26,1.26,0,0,0,45,47.81a1.28,1.28,0,0,0,1.74.35l.31-.22,4.46,6.65a1.25,1.25,0,0,0,2.08-1.39l-4.46-6.65.39-.26a1.25,1.25,0,0,0-1.39-2.08Z" fill="#875b7b"/>
        <path d="M58.44,45.63l.82-.55a1.29,1.29,0,0,0,.53-.8,1.25,1.25,0,0,0-.19-.94l-.07-.07-2.36-3.51a1.26,1.26,0,0,0-1.73-.34l-2.58,1.73a1.22,1.22,0,0,0-.63,1.63s0,0,0,0l.08.15,5.11,7.61a1.25,1.25,0,0,0,2.08-1.39l-.42-.63,1.74.65a1.23,1.23,0,0,0,1.6-.74,1.23,1.23,0,0,0-.73-1.6Zm-1.63-1.92-.67.45-1-1.52.66-.45Z" fill="#875b7b"/>
        <path d="M62.5,34.6l-2.58,1.73a1.27,1.27,0,0,0-.48.66,1,1,0,0,0,0,.62.44.44,0,0,0,0,.1.76.76,0,0,0,0,.11.85.85,0,0,0,.13.25l5.1,7.6a1.25,1.25,0,0,0,2.08-1.39L64.83,41.4l.72-.49,2,2.93a1.25,1.25,0,0,0,2.08-1.4l-5.17-7.7A1.21,1.21,0,0,0,62.5,34.6Zm.93,4.72-1.08-1.61.73-.49,1.08,1.62Z" fill="#875b7b"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Extra_Time_Off_2.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Extra_Time_Off" data-name="Extra Time Off">
    <g>
      <path d="M62.35,37.71l1.08,1.61.73-.48-1.08-1.62Z" fill="#875b7b"/>
      <path d="M55.13,42.64c.34.5.68,1,1,1.52l.67-.45-1-1.52Z" fill="#875b7b"/>
      <path d="M86.33,54.43l-8.26-3.89a2.77,2.77,0,0,1-.33-4.83l7.67-5a2.76,2.76,0,0,0-1.73-5.08l-9.11.76a2.77,2.77,0,0,1-2.69-4L76,24.28a2.77,2.77,0,0,0-4-3.53L64.5,26a2.77,2.77,0,0,1-4.35-2.13l-.46-9.13a2.76,2.76,0,0,0-5.26-1l-3.89,8.26a2.77,2.77,0,0,1-4.83.33l-5-7.67a2.76,2.76,0,0,0-5.08,1.73l.76,9.11a2.77,2.77,0,0,1-4,2.69L24.28,24a2.77,2.77,0,0,0-3.53,4L26,35.5a2.77,2.77,0,0,1-2.13,4.35l-9.13.46a2.76,2.76,0,0,0-1,5.26l8.26,3.89a2.77,2.77,0,0,1,.33,4.83l-7.67,5a2.76,2.76,0,0,0,1.73,5.08l9.11-.76a2.77,2.77,0,0,1,2.69,4L24,75.72a2.77,2.77,0,0,0,4,3.53L35.5,74a2.77,2.77,0,0,1,4.35,2.13l.46,9.13a2.76,2.76,0,0,0,5.26,1l3.89-8.26a2.77,2.77,0,0,1,4.83-.33l5,7.67a2.76,2.76,0,0,0,5.08-1.73l-.76-9.11a2.77,2.77,0,0,1,4-2.69L75.72,76a2.77,2.77,0,0,0,3.53-4L74,64.5a2.77,2.77,0,0,1,2.13-4.35l9.13-.46A2.76,2.76,0,0,0,86.33,54.43ZM40,63.75l-2.64,1.77a1.73,1.73,0,0,1-.24.1,1.17,1.17,0,0,1-1.61-.37L30.4,57.61s0,0,0,0a1,1,0,0,1-.11-.2,1.08,1.08,0,0,1,.06-1,1.23,1.23,0,0,1,.41-.49L33.5,54a1.28,1.28,0,0,1,1.74.34,1.26,1.26,0,0,1-.35,1.73l-1.71,1.15,1.09,1.62L36,57.75a1.25,1.25,0,0,1,1.4,2.08L35.66,61l1.25,1.86,1.72-1.15A1.25,1.25,0,0,1,40,63.75Zm7.47-5.09a1.52,1.52,0,0,1-2.11.26h0a1.5,1.5,0,1,1-2.89.81l-1.1-3.93-3.22-2.52A1.5,1.5,0,0,1,40,50.92h0v0a1.5,1.5,0,1,1,2.89-.81L44,54l3.23,2.51A1.51,1.51,0,0,1,47.49,58.66Zm4-4.07c-1.49-2.21-3-4.43-4.46-6.65l-.31.22A1.28,1.28,0,0,1,45,47.81a1.26,1.26,0,0,1,.34-1.73l2.79-1.87a1.25,1.25,0,0,1,1.39,2.08l-.39.26c1.48,2.22,3,4.43,4.46,6.65A1.25,1.25,0,0,1,51.51,54.59Zm10.92-6.14a1.23,1.23,0,0,1-1.6.74l-1.74-.65.42.63a1.25,1.25,0,0,1-2.08,1.39L52.32,43l-.08-.15v0a1.22,1.22,0,0,1,.63-1.63l2.58-1.73a1.26,1.26,0,0,1,1.73.34l2.36,3.51.07.07a1.25,1.25,0,0,1,.19.94,1.29,1.29,0,0,1-.53.8l-.82.55,3.26,1.22A1.23,1.23,0,0,1,62.43,48.45Zm5.08-4.61-2-2.93-.72.49,1.93,2.88a1.25,1.25,0,0,1-2.08,1.39l-5.1-7.6a.85.85,0,0,1-.13-.25.76.76,0,0,1,0-.11.44.44,0,0,1,0-.1,1,1,0,0,1,0-.62,1.27,1.27,0,0,1,.48-.66L62.5,34.6a1.21,1.21,0,0,1,1.92.14l5.17,7.7A1.25,1.25,0,0,1,67.51,43.84Z" fill="#875b7b"/>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Maternity_Time_Off.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Maternity_Time_Off" data-name="Maternity Time Off">
    <g>
      <g>
        <path d="M63,11.81H53.58V38.42H20.75v2.35a24,24,0,0,0,24,24H63a24,24,0,0,0,24-24v-5A24,24,0,0,0,63,11.81Z" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
        <path d="M46.1,40.92H86.75c3.21,0,3.22-5,0-5H46.1c-3.22,0-3.23,5,0,5Z" fill="#875b7b"/>
        <path d="M23.25,38.52V23.4a2.53,2.53,0,0,0-2.5-2.5H13a2.5,2.5,0,0,0,0,5h7.71l-2.5-2.5V38.52a2.5,2.5,0,0,0,5,0Z" fill="#875b7b"/>
        <circle cx="28.65" cy="79.82" r="8.37" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
        <circle cx="78.1" cy="79.82" r="8.37" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
        <path d="M80.64,71.45V59.26a2.5,2.5,0,0,0-5,0V71.45a2.5,2.5,0,0,0,5,0Z" fill="#875b7b"/>
        <path d="M31.25,71.45V59.68a2.5,2.5,0,0,0-5,0V71.45a2.5,2.5,0,0,0,5,0Z" fill="#875b7b"/>
      </g>
      <path d="M28.66,82.27c3.22,0,3.22-5,0-5s-3.22,5,0,5Z" fill="#875b7b"/>
      <path d="M78.08,82.27c3.21,0,3.22-5,0-5s-3.23,5,0,5Z" fill="#875b7b"/>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Maternity_Time_Off_2.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Maternity_Time_Off" data-name="Maternity Time Off">
    <g>
      <path d="M89.46,40.77v-5A26.51,26.51,0,0,0,63,9.31H53.58a2.5,2.5,0,0,0-2.5,2.5V35.92H23.25V23.4a2.53,2.53,0,0,0-2.5-2.5H13a2.5,2.5,0,0,0,0,5h5.21V40.77a26.41,26.41,0,0,0,8,18.94v9.51a10.89,10.89,0,1,0,5,0V63.55a26.38,26.38,0,0,0,13.48,3.7H63A26.25,26.25,0,0,0,75.64,64v5.22a10.87,10.87,0,1,0,5,0V60.48A26.41,26.41,0,0,0,89.46,40.77ZM63,14.31A21.49,21.49,0,0,1,84.46,35.79v.13H56.08V14.31ZM44.73,62.25A21.5,21.5,0,0,1,23.25,40.92h61.2a21.39,21.39,0,0,1-7.64,16.26,2.89,2.89,0,0,0-.47.38A21.33,21.33,0,0,1,63,62.25Z" fill="#875b7b"/>
      <polygon points="21.36 39.59 22.89 51.45 32.66 62.33 55.54 65.12 78.42 60.8 86.94 47.4 86.94 37.63 21.36 39.59" fill="#875b7b"/>
      <line x1="54.7" y1="36.38" x2="69.77" y2="13.5" fill="#875b7b" stroke="#875b7b" stroke-miterlimit="10" stroke-width="5"/>
      <path d="M28.66,82.27c3.22,0,3.22-5,0-5s-3.22,5,0,5Z" fill="#875b7b"/>
      <path d="M78.08,82.27c3.21,0,3.22-5,0-5s-3.23,5,0,5Z" fill="#875b7b"/>
    </g>
    <line x1="85.12" y1="24.8" x2="54.7" y2="36.38" fill="#875b7b" stroke="#875b7b" stroke-miterlimit="10" stroke-width="5"/>
  </g>
</svg>

```

## File: static\src\img\icons\Maternity_Time_Off_3.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Maternity_Time_Off" data-name="Maternity Time Off">
    <g>
      <g>
        <path d="M23.25,38.52V23.4a2.53,2.53,0,0,0-2.5-2.5H13a2.5,2.5,0,0,0,0,5h7.71l-2.5-2.5V38.52a2.5,2.5,0,0,0,5,0Z" fill="#875b7b"/>
        <path d="M28.65,90.69A10.87,10.87,0,1,1,39.53,79.82,10.89,10.89,0,0,1,28.65,90.69Zm0-16.74a5.87,5.87,0,1,0,5.88,5.87A5.87,5.87,0,0,0,28.65,74Z" fill="#875b7b"/>
        <path d="M78.1,90.69A10.87,10.87,0,1,1,89,79.82,10.89,10.89,0,0,1,78.1,90.69ZM78.1,74A5.87,5.87,0,1,0,84,79.82,5.88,5.88,0,0,0,78.1,74Z" fill="#875b7b"/>
        <path d="M80.64,71.45V59.26a2.5,2.5,0,0,0-5,0V71.45a2.5,2.5,0,0,0,5,0Z" fill="#875b7b"/>
        <path d="M31.25,71.45V59.68a2.5,2.5,0,0,0-5,0V71.45a2.5,2.5,0,0,0,5,0Z" fill="#875b7b"/>
        <g>
          <path d="M63,14.31H56.08V35.92H84.46v-.13A21.49,21.49,0,0,0,63,14.31Z" fill="none"/>
          <path d="M89.46,35.79A26.51,26.51,0,0,0,63,9.31H53.58a2.5,2.5,0,0,0-2.5,2.5V35.92H20.75a2.5,2.5,0,0,0-2.5,2.5v2.35a26.36,26.36,0,0,0,4.39,14.59,7.65,7.65,0,0,1,5.88-3.17h.32a7.55,7.55,0,0,1,7.41,7.5v6.17a26.41,26.41,0,0,0,8.48,1.4H63a26.13,26.13,0,0,0,7.65-1.14V59.26a7.54,7.54,0,0,1,7.28-7.49h.32a7.49,7.49,0,0,1,6.53,4,26.31,26.31,0,0,0,4.69-15Zm-5,.13H56.08V14.31H63A21.49,21.49,0,0,1,84.46,35.79Z" fill="#875b7b"/>
        </g>
      </g>
      <path d="M28.66,82.27c3.22,0,3.22-5,0-5s-3.22,5,0,5Z" fill="#875b7b"/>
      <path d="M78.08,82.27c3.21,0,3.22-5,0-5s-3.23,5,0,5Z" fill="#875b7b"/>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Paid_Time_Off.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Paid_Time_Off" data-name="Paid Time Off">
    <g>
      <path d="M21.46,61.83a3,3,0,0,1,4.24-.27L39.5,73.7H78.9a3,3,0,0,1,0,6H74.37v4.81a3,3,0,0,1-6,0V79.7H45.48v4.81a3,3,0,0,1-6,0V79.7H38.37a3,3,0,0,1-2-.75L21.73,66.07A3,3,0,0,1,21.46,61.83Z" fill="#875b7b"/>
      <g>
        <rect x="46.5" y="67.63" width="6" height="0.07" fill="#875b7b"/>
        <path d="M78.87,41.86c0-.18-.09-.36-.09-.54,0-32.94-58-32.94-58,0a5.58,5.58,0,0,1-.1.59c-.2,2.4,2.94,3.29,4.29,1.3,2.45-3.62,10.12-3.71,12.86-.29.57.71,1.84.95,2.26.14,1.88-3.62,17.81-3.54,18.69,0,.21.88,1.66.59,2.23-.11,2.83-3.48,10.92-3.39,13.51.24C76,45.18,79.12,44.25,78.87,41.86Z" fill="none" stroke="#875b7b" stroke-miterlimit="10" stroke-width="5"/>
        <path d="M46.5,40.62v27h6v-27A23.17,23.17,0,0,0,46.5,40.62Z" fill="#875b7b"/>
        <path d="M52.5,11.6a3.08,3.08,0,0,0-2.41-3.06,3,3,0,0,0-3.59,3v3h6Z" fill="#875b7b"/>
      </g>
    </g>
    <path d="M40.13,43.06c-4.26-10.68,0-19.27,9.55-26.59" fill="none" stroke="#875b7b" stroke-miterlimit="10" stroke-width="5"/>
    <path d="M59.57,43.33c4.26-10.67,0-19.26-9.55-26.58" fill="none" stroke="#875b7b" stroke-miterlimit="10" stroke-width="5"/>
  </g>
</svg>

```

## File: static\src\img\icons\Paid_Time_Off_2.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Paid_Time_Off" data-name="Paid Time Off">
    <g>
      <path d="M21.46,61.83a3,3,0,0,1,4.24-.27L39.5,73.7H78.9a3,3,0,0,1,0,6H74.37v4.81a3,3,0,0,1-6,0V79.7H45.48v4.81a3,3,0,0,1-6,0V79.7H38.37a3,3,0,0,1-2-.75L21.73,66.07A3,3,0,0,1,21.46,61.83Z" fill="#875b7b"/>
      <g>
        <rect x="46.5" y="67.63" width="6" height="0.07" fill="#875b7b"/>
        <path d="M78.87,41.86c0-.18-.09-.36-.09-.54,0-32.94-58-32.94-58,0a5.58,5.58,0,0,1-.1.59c-.2,2.4,2.94,3.29,4.29,1.3,2.45-3.62,10.12-3.71,12.86-.29.57.71,1.84.95,2.26.14,1.88-3.62,17.81-3.54,18.69,0,.21.88,1.66.59,2.23-.11,2.83-3.48,10.92-3.39,13.51.24C76,45.18,79.12,44.25,78.87,41.86Z" fill="none" stroke="#875b7b" stroke-miterlimit="10" stroke-width="5"/>
        <path d="M46.5,40.62v27h6v-27A23.17,23.17,0,0,0,46.5,40.62Z" fill="#875b7b"/>
        <path d="M52.5,11.6a3.08,3.08,0,0,0-2.41-3.06,3,3,0,0,0-3.59,3v3h6Z" fill="#875b7b"/>
      </g>
    </g>
    <path d="M21.77,43.82c-3.22-18.89,18.49-25.37,27.4-24.71C43.73,22.74,39,25.49,39,40.56,29.28,37.67,26.2,40.61,21.77,43.82Z" fill="#875b7b"/>
    <path d="M76,43.91c3.32-19.85-17.83-26.36-26.84-24.8,5.44,3.63,9.12,7.5,9.12,22.57C65.64,39.83,69.39,41.65,76,43.91Z" fill="#875b7b"/>
  </g>
</svg>

```

## File: static\src\img\icons\Paid_Time_Off_3.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Paid_Time_Off" data-name="Paid Time Off">
    <g>
      <path d="M21.46,61.83a3,3,0,0,1,4.24-.27L39.5,73.7H78.9a3,3,0,0,1,0,6H74.37v4.81a3,3,0,0,1-6,0V79.7H45.48v4.81a3,3,0,0,1-6,0V79.7H38.37a3,3,0,0,1-2-.75L21.73,66.07A3,3,0,0,1,21.46,61.83Z" fill="#875b7b"/>
      <g>
        <rect x="46.5" y="67.63" width="6" height="0.07" fill="#875b7b"/>
        <path d="M78.78,41.32A29.93,29.93,0,0,0,52.5,19V15.6a3.08,3.08,0,0,0-2.41-3.06,3,3,0,0,0-3.59,3V19A29.92,29.92,0,0,0,20.82,41.32a4.28,4.28,0,0,0-.1.59c-.2,2.4,2.94,3.29,4.29,1.3,2.45-3.62,10.12-3.71,12.86-.29a1.53,1.53,0,0,0,2.26.14c3.79-3.52,14.93-3.52,18.69,0A1.5,1.5,0,0,0,61.05,43c2.83-3.48,10.92-3.39,13.51.24,1.39,2,4.56,1,4.31-1.35A3.38,3.38,0,0,0,78.78,41.32Z" fill="#875b7b"/>
        <rect x="46.5" y="38.79" width="6" height="28.84" fill="#875b7b"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Parental_Time_Off.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Parental_Time_Off" data-name="Parental Time Off">
    <path d="M94.89,52.33,87.17,34.94a3.76,3.76,0,0,0-3.86-2.2,5.87,5.87,0,1,0-7.52.21,3.76,3.76,0,0,0-3.11.75,2,2,0,0,0-1.05,1L58.9,47.42A3.74,3.74,0,0,0,58,49.07l-4,1.81A3.74,3.74,0,0,0,52.19,50a4.28,4.28,0,1,0-5.52,0,3.8,3.8,0,0,0-1.89.72l-2.86-2a3.91,3.91,0,0,0-.82-1.22L27.65,34a3.75,3.75,0,0,0-2.81-1.09l-.19-.05a5.88,5.88,0,1,0-7.64,0,3.77,3.77,0,0,0-4.18,2.16L5.11,52.33A3.77,3.77,0,0,0,12,55.38l.4-.89A5.12,5.12,0,0,0,13,56.62V73.69a3.77,3.77,0,0,0,3.77,3.77h0a3.77,3.77,0,0,0,3.77-3.77V59.29h.15V73.67a3.77,3.77,0,1,0,7.54,0V56.41a5.19,5.19,0,0,0,.54-2.26V45.76l7,7a3.74,3.74,0,0,0,3.74.93l3.75,2.67v9.2a3.69,3.69,0,0,0,.48,1.8v7.44a2.75,2.75,0,1,0,5.49,0v-5.5h.11v5.49a2.75,2.75,0,0,0,5.49,0V67.19a3.64,3.64,0,0,0,.4-1.64V56.34l5.6-2.55a3.75,3.75,0,0,0,3.38-1l8.64-8.64.74,4.59a1.61,1.61,0,0,1-.2,1.07L65.7,63.28a1.86,1.86,0,0,0,1.89,2.65h4v7.76a3.77,3.77,0,0,0,7.54,0V65.93h.15v7.74a3.77,3.77,0,0,0,7.54,0V65.93h3.43a1.85,1.85,0,0,0,1.88-2.65l-5.47-9.66a3.78,3.78,0,0,0-.9-1.57l-1.21-2.14a1.62,1.62,0,0,1-.18-1.14l.22-1.07L88,55.38a3.77,3.77,0,0,0,6.89-3.05Z" fill="none" stroke="#875b7b" stroke-linejoin="round" stroke-width="3"/>
  </g>
</svg>

```

## File: static\src\img\icons\Parental_Time_Off_2.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Parental_Time_Off" data-name="Parental Time Off">
    <path d="M94.89,52.33,87.17,34.94a3.76,3.76,0,0,0-3.86-2.2,5.87,5.87,0,1,0-7.52.21,3.76,3.76,0,0,0-3.11.75,2,2,0,0,0-1.05,1L58.9,47.42A3.74,3.74,0,0,0,58,49.07l-4,1.81A3.74,3.74,0,0,0,52.19,50a4.28,4.28,0,1,0-5.52,0,3.8,3.8,0,0,0-1.89.72l-2.86-2a3.91,3.91,0,0,0-.82-1.22L27.65,34a3.75,3.75,0,0,0-2.81-1.09l-.19-.05a5.88,5.88,0,1,0-7.64,0,3.77,3.77,0,0,0-4.18,2.16L5.11,52.33A3.77,3.77,0,0,0,12,55.38l.4-.89A5.12,5.12,0,0,0,13,56.62V73.69a3.77,3.77,0,0,0,3.77,3.77h0a3.77,3.77,0,0,0,3.77-3.77V59.29h.15V73.67a3.77,3.77,0,1,0,7.54,0V56.41a5.19,5.19,0,0,0,.54-2.26V45.76l7,7a3.74,3.74,0,0,0,3.74.93l3.75,2.67v9.2a3.69,3.69,0,0,0,.48,1.8v7.44a2.75,2.75,0,1,0,5.49,0v-5.5h.11v5.49a2.75,2.75,0,0,0,5.49,0V67.19a3.64,3.64,0,0,0,.4-1.64V56.34l5.6-2.55a3.75,3.75,0,0,0,3.38-1l8.64-8.64.74,4.59a1.61,1.61,0,0,1-.2,1.07L65.7,63.28a1.86,1.86,0,0,0,1.89,2.65h4v7.76a3.77,3.77,0,0,0,7.54,0V65.93h.15v7.74a3.77,3.77,0,0,0,7.54,0V65.93h3.43a1.85,1.85,0,0,0,1.88-2.65l-5.47-9.66a3.78,3.78,0,0,0-.9-1.57l-1.21-2.14a1.62,1.62,0,0,1-.18-1.14l.22-1.07L88,55.38a3.77,3.77,0,0,0,6.89-3.05Z" fill="#875b7b"/>
  </g>
</svg>

```

## File: static\src\img\icons\Recovery_Bank_Holiday.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Recovery_Bank_Holiday" data-name="Recovery Bank Holiday">
    <g>
      <g>
        <path d="M57.46,55.29V43.75a2.5,2.5,0,0,0-5,0V55.29a2.5,2.5,0,0,0,5,0Z" fill="#875b7b"/>
        <path d="M54.78,53.33l-14.85-.08a2.5,2.5,0,0,0,0,5l14.85.08a2.5,2.5,0,0,0,0-5Z" fill="#875b7b"/>
      </g>
      <g>
        <path d="M40.52,24.65A30.41,30.41,0,1,1,19.6,53.55c0-.36,0-.72,0-1.07" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
        <path d="M47.38,14.28l-8.52,8.61a2.53,2.53,0,0,0,.51,3.92l9.37,6.05a2.52,2.52,0,0,0,3.42-.9,2.56,2.56,0,0,0-.9-3.42l-9.37-6,.5,3.92,8.53-8.6a2.5,2.5,0,1,0-3.54-3.54Z" fill="#875b7b"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Recovery_Bank_Holiday_2.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Recovery_Bank_Holiday" data-name="Recovery Bank Holiday">
    <g>
      <path d="M20,45a7.48,7.48,0,0,1,7.23,7.76c0,.27,0,.53,0,.8A22.91,22.91,0,1,0,57.55,31.86a7.74,7.74,0,0,1-.81,2.43,7.52,7.52,0,0,1-6.62,3.9,7.19,7.19,0,0,1-3.93-1.14l-8.87-5.72a7.44,7.44,0,0,1-4.07-7.59A34.22,34.22,0,0,0,16.93,45.55,7.42,7.42,0,0,1,19.79,45ZM40.1,53.24l12.52.07V43.74a2.5,2.5,0,0,1,5,0V55.28a2.34,2.34,0,0,1-.21,1,2.56,2.56,0,0,1-2.46,2L40.1,58.24a2.5,2.5,0,0,1,0-5Z" fill="#875b7b"/>
      <path d="M50.16,20.63c-.65,0-1.3,0-1.95.08,1-1,1.92-1.93,2.87-2.9a2.5,2.5,0,1,0-3.53-3.53l-8.16,8.23a2.48,2.48,0,0,0,.35,4.43l9.16,5.91a2.52,2.52,0,0,0,3.42-.9,2.56,2.56,0,0,0-.89-3.42L47.2,25.81a26.57,26.57,0,0,1,3-.18,27.91,27.91,0,1,1-27.9,27.91c0-.33,0-.65,0-1A2.49,2.49,0,0,0,19.87,50a2.52,2.52,0,0,0-2.59,2.4c0,.39,0,.77,0,1.16a32.91,32.91,0,1,0,32.9-32.91Z" fill="#875b7b"/>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Sick_Time_Off.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Sick_Time_Off" data-name="Sick Time Off">
    <g>
      <path d="M44.38,10.28h6.89a3.1,3.1,0,0,1,3.1,3.1V38.33A21.07,21.07,0,0,1,33.3,59.4h0A21.08,21.08,0,0,1,12.23,38.33V13.39a3.1,3.1,0,0,1,3.11-3.1l6.6,0" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <path d="M32.63,59.93v7.84c0,29.27,44.79,29.27,44.79,0V57.82" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <g>
        <circle cx="77.16" cy="45.77" r="10.6" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
        <path d="M77.19,48.27c3.21,0,3.22-5,0-5s-3.23,5,0,5Z" fill="#875b7b"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Sick_Time_Off_2.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Sick_Time_Off" data-name="Sick Time Off">
    <g>
      <path d="M44.38,10.28h6.89a3.1,3.1,0,0,1,3.1,3.1V38.33A21.07,21.07,0,0,1,33.3,59.4h0A21.08,21.08,0,0,1,12.23,38.33V13.39a3.1,3.1,0,0,1,3.11-3.1l6.6,0" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <path d="M32.63,59.93v7.84c0,29.27,44.79,29.27,44.79,0V57.82" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <g>
        <circle cx="77.16" cy="45.77" r="10.6" fill="#875b7b" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
        <path d="M77.19,48.27c3.21,0,3.22-5,0-5s-3.23,5,0,5Z" fill="#875b7b"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Small_Unemployement.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Small_Unemployement" data-name="Small Unemployement">
    <path d="M50,86.88A36.88,36.88,0,1,1,86.88,50,36.93,36.93,0,0,1,50,86.88Zm0-68.76A31.88,31.88,0,1,0,81.88,50,31.91,31.91,0,0,0,50,18.12Z" fill="#875b7b"/>
    <g>
      <path d="M34.9,64.53a2.14,2.14,0,0,1-2.14-2.14V54a49.9,49.9,0,0,0,13.75,3l-7.53-5a45.14,45.14,0,0,1-6.22-2V48l-4-2.63v17a6.14,6.14,0,0,0,6.14,6.14H64l-6.08-4Z" fill="#875b7b"/>
      <path d="M45.37,35.47h9.26v3.38H46.16L52.49,43H65.1a2.14,2.14,0,0,1,2.14,2.14V50c-.89.33-1.8.63-2.71.91l6.71,4.41V45.15A6.14,6.14,0,0,0,65.1,39H58.63V33.47a2,2,0,0,0-2-2H43.37a2,2,0,0,0-2,2V35.7l4,2.63Z" fill="#875b7b"/>
    </g>
    <path d="M78.42,71.53a2.46,2.46,0,0,1-1.37-.41L20.59,34a2.5,2.5,0,0,1,2.74-4.18L79.8,66.94a2.5,2.5,0,0,1-1.38,4.59Z" fill="#875b7b"/>
  </g>
</svg>

```

## File: static\src\img\icons\Small_Unemployement_2.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Small_Unemployement" data-name="Small Unemployement">
    <path d="M50,86.88A36.88,36.88,0,1,1,86.88,50,36.92,36.92,0,0,1,50,86.88Zm0-68.76A31.88,31.88,0,1,0,81.88,50,31.91,31.91,0,0,0,50,18.12Z" fill="#875b7b"/>
    <g>
      <path d="M71.74,45.15a6.65,6.65,0,0,0-6.64-6.64h-6v-5a2.5,2.5,0,0,0-2.5-2.5H43.37a2.5,2.5,0,0,0-2.5,2.5v1.9L71.74,55.66Zm-17.61-6.8H45.87V36h8.26Z" fill="#875b7b"/>
      <path d="M28.26,45.15V62.39A6.65,6.65,0,0,0,34.9,69H64.77L28.26,45Z" fill="#875b7b"/>
    </g>
    <path d="M78.42,71.53a2.46,2.46,0,0,1-1.37-.41L20.59,34a2.5,2.5,0,0,1,2.74-4.18L79.8,66.94a2.5,2.5,0,0,1-1.38,4.59Z" fill="#875b7b"/>
  </g>
</svg>

```

## File: static\src\img\icons\Small_Unemployement_3.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Small_Unemployement" data-name="Small Unemployement">
    <g>
      <g>
        <path d="M76.31,41.83a8.05,8.05,0,0,0-8-8H61.05v-6.1a3,3,0,0,0-3-3H42a3,3,0,0,0-3,3v6.1h-.74l38.1,25ZM55,33.6H45V30.72H55Z" fill="#875b7b"/>
        <path d="M23.69,62.69a8,8,0,0,0,8,8H65.67l-42-27.59Z" fill="#875b7b"/>
      </g>
      <path d="M82.64,75.33a2.89,2.89,0,0,1-1.59-.47L15.77,31.94A2.89,2.89,0,1,1,19,27.11L84.23,70a2.89,2.89,0,0,1-1.59,5.3Z" fill="#875b7b"/>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Training_Time_Off.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Training_Time_Off" data-name="Training Time Off">
    <g>
      <path d="M47,42.25,8.09,29.61,47.66,17a8.69,8.69,0,0,1,5.22,0l39,12.15L52.44,42.23A8.59,8.59,0,0,1,47,42.25Z" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <path d="M24.23,37.5v7.42c0,15.69,51.54,15.69,51.54,0V37.31" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <line x1="35.89" y1="38.87" x2="35.89" y2="71.8" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <circle cx="35.94" cy="77.78" r="5.68" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Training_Time_Off_2.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Training_Time_Off" data-name="Training Time Off">
    <g>
      <g>
        <path d="M21.73,41.93v3c0,2.84,1.25,6.66,6.66,9.72V44.09Z" fill="#875b7b"/>
        <path d="M49.7,50.18a16.43,16.43,0,0,1-5-.79L43.39,49v9.88a64.34,64.34,0,0,0,6.61.35c14.07,0,28.27-4.41,28.27-14.27V41.53L54.81,49.35A16.23,16.23,0,0,1,49.7,50.18Z" fill="#875b7b"/>
      </g>
      <path d="M92.65,26.69l-39-12.14a11.12,11.12,0,0,0-6.73,0L7.33,27.23a2.5,2.5,0,0,0,0,4.76l26.07,8.47V70a8.16,8.16,0,1,0,5,0V42.08l7.86,2.55a11.06,11.06,0,0,0,7,0L92.7,31.45a2.5,2.5,0,0,0,0-4.76Z" fill="#875b7b"/>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Unpaid_Time_Off.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Unpaid_Time_Off" data-name="Unpaid Time Off">
    <path d="M50,86.55A36.55,36.55,0,1,1,86.55,50,36.6,36.6,0,0,1,50,86.55Zm0-68.1A31.55,31.55,0,1,0,81.55,50,31.59,31.59,0,0,0,50,18.45Z" fill="#875b7b"/>
    <path d="M60.78,34.26a13.53,13.53,0,0,0-8.6-3.93V27.58c0-3.22-5-3.22-5,0v3.09A11.57,11.57,0,0,0,38.43,41a11.4,11.4,0,0,0,3.08,8.61,11.55,11.55,0,0,0,5.67,3.29V66.13a8.48,8.48,0,0,1-4.42-2.36,2.2,2.2,0,0,1-.64-1.55v-3a2.5,2.5,0,0,0-2.49-2.51h0a2.51,2.51,0,0,0-2.5,2.49V62.2a7.25,7.25,0,0,0,2.1,5.11,13.55,13.55,0,0,0,8,3.87v1.24c0,3.22,5,3.22,5,0V71.05a11.6,11.6,0,0,0,9.39-10.49,11.46,11.46,0,0,0-9.39-12V35.35a8.5,8.5,0,0,1,5.06,2.45,2.15,2.15,0,0,1,.64,1.55v3a2.49,2.49,0,0,0,2.49,2.51h0a2.51,2.51,0,0,0,2.5-2.49V39.37A7.25,7.25,0,0,0,60.78,34.26ZM45.16,46.2a6.4,6.4,0,0,1,2-10.24v11.7A6.68,6.68,0,0,1,45.16,46.2Zm11.42,14a6.5,6.5,0,0,1-4.4,5.63V53.66a6.38,6.38,0,0,1,2.66,1.71A6.46,6.46,0,0,1,56.58,60.24Z" fill="#875b7b"/>
    <g>
      <path d="M33.44,40.68a16.67,16.67,0,0,1,.74-3.87L23,29.43a2.49,2.49,0,1,0-2.75,4.15l13.13,8.69C33.4,41.74,33.41,41.21,33.44,40.68Z" fill="#875b7b"/>
      <path d="M78.88,66.42,66.5,58.22a16.83,16.83,0,0,1,.06,2.66,15.9,15.9,0,0,1-.5,3l10.08,6.67a2.48,2.48,0,1,0,2.74-4.14Z" fill="#875b7b"/>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Unpaid_Time_Off_2.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Unpaid_Time_Off" data-name="Unpaid Time Off">
    <g>
      <path d="M61.24,34.26a13.53,13.53,0,0,0-8.6-3.93V27.58c0-3.22-5-3.22-5,0v3.09A11.57,11.57,0,0,0,38.89,41a11.44,11.44,0,0,0,8.75,11.9V66.13a8.5,8.5,0,0,1-4.43-2.36,2.19,2.19,0,0,1-.63-1.55v-3a2.5,2.5,0,0,0-2.49-2.51h0a2.5,2.5,0,0,0-2.5,2.49V62.2a7.21,7.21,0,0,0,2.1,5.11,13.52,13.52,0,0,0,8,3.87v1.24c0,3.22,5,3.22,5,0V71.05A11.58,11.58,0,0,0,62,60.56a11.46,11.46,0,0,0-9.38-12V35.35A8.5,8.5,0,0,1,57.7,37.8a2.2,2.2,0,0,1,.64,1.55v3a2.49,2.49,0,0,0,2.48,2.51h0a2.51,2.51,0,0,0,2.5-2.49V39.37A7.25,7.25,0,0,0,61.24,34.26Zm-13.6,13.4a6.56,6.56,0,0,1-2-1.46,6.4,6.4,0,0,1,2-10.24Zm7.66,7.71a6.42,6.42,0,0,1-2.66,10.5V53.66A6.38,6.38,0,0,1,55.3,55.37Z" fill="#875b7b"/>
      <g>
        <path d="M80.8,64,66.12,54.35A16.47,16.47,0,0,1,67,60.88a15.83,15.83,0,0,1-1.24,5.22l9.53,6.27a4.93,4.93,0,0,0,2.74.82A5,5,0,0,0,80.8,64Z" fill="#875b7b"/>
        <path d="M33.9,40.68a16.07,16.07,0,0,1,1.78-6.37L24.42,26.9a5,5,0,0,0-5.5,8.35L34.25,45.34A16.79,16.79,0,0,1,33.9,40.68Z" fill="#875b7b"/>
      </g>
      <g>
        <line x1="78.33" y1="31.08" x2="21.67" y2="68.47" fill="#875b7b"/>
        <path d="M21.68,73.47a5,5,0,0,1-2.76-9.17L75.57,26.9a5,5,0,0,1,5.51,8.35L24.43,72.64A5,5,0,0,1,21.68,73.47Z" fill="#875b7b"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Unpredictable_Reasons.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Credit_Time" data-name="Credit Time">
    <path d="M90.13,38.78a5.33,5.33,0,0,0-.28-1.44,1,1,0,0,0,0-.16c-.06-.18-.13-.35-.2-.51a6,6,0,0,0-.3-.56L89.21,36a4.82,4.82,0,0,0-4.15-2.26l-.43.06a4.3,4.3,0,0,0-.62,0H71a5,5,0,0,0,0,10H72.3L61.63,54.44,51.26,44.21s-.08-.06-.12-.1l-9-8.88a4.84,4.84,0,0,0-1.78-1.1,4.94,4.94,0,0,0-5.63.93L19.94,49.85v-2.7a5,5,0,0,0-10,0v13.1a3.93,3.93,0,0,0-.07,1,5.33,5.33,0,0,0,.28,1.44,1.09,1.09,0,0,0,0,.17c.06.17.13.34.2.5a6,6,0,0,0,.3.56l.09.14a4.82,4.82,0,0,0,4.15,2.26l.43-.06a4.3,4.3,0,0,0,.62,0H29a5,5,0,0,0,0-10H27.7L38.37,45.56,48.74,55.79s.08.06.12.1l9,8.88a4.84,4.84,0,0,0,1.78,1.1,4.94,4.94,0,0,0,5.63-.93L80.06,50.15v2.7a5,5,0,0,0,10,0V39.75A3.93,3.93,0,0,0,90.13,38.78Z" fill="none" stroke="#875b7b" stroke-miterlimit="10" stroke-width="5"/>
  </g>
</svg>

```

## File: static\src\img\icons\Unpredictable_Reasons_2.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Credit_Time" data-name="Credit Time">
    <path d="M90.13,38.78a5.33,5.33,0,0,0-.28-1.44,1,1,0,0,0,0-.16c-.06-.18-.13-.35-.2-.51a6,6,0,0,0-.3-.56L89.21,36a4.82,4.82,0,0,0-4.15-2.26l-.43.06a4.3,4.3,0,0,0-.62,0H71a5,5,0,0,0,0,10H72.3L61.63,54.44,51.26,44.21s-.08-.06-.12-.1l-9-8.88a4.84,4.84,0,0,0-1.78-1.1,4.94,4.94,0,0,0-5.63.93L19.94,49.85v-2.7a5,5,0,0,0-10,0v13.1a3.93,3.93,0,0,0-.07,1,5.33,5.33,0,0,0,.28,1.44,1.09,1.09,0,0,0,0,.17c.06.17.13.34.2.5a6,6,0,0,0,.3.56l.09.14a4.82,4.82,0,0,0,4.15,2.26l.43-.06a4.3,4.3,0,0,0,.62,0H29a5,5,0,0,0,0-10H27.7L38.37,45.56,48.74,55.79s.08.06.12.1l9,8.88a4.84,4.84,0,0,0,1.78,1.1,4.94,4.94,0,0,0,5.63-.93L80.06,50.15v2.7a5,5,0,0,0,10,0V39.75A3.93,3.93,0,0,0,90.13,38.78Z" fill="#875b7b"/>
  </g>
</svg>

```

## File: static\src\img\icons\Work_Accident_Time_Off.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Work_Accident_Time_Off" data-name="Work Accident Time Off">
    <g>
      <path d="M39,44.17H61A20.59,20.59,0,0,1,81.63,64.75V89.91a0,0,0,0,1,0,0H27.2a8.82,8.82,0,0,1-8.82-8.82V64.75A20.59,20.59,0,0,1,39,44.17Z" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <circle cx="50" cy="27.11" r="17.02" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <path d="M63.63,89.8c11.79,0,11.79-16.12,0-16.12H58.5" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <line x1="26.87" y1="48.15" x2="37.68" y2="88.75" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <line x1="38.96" y1="44.17" x2="67.63" y2="89.68" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
    </g>
  </g>
</svg>

```

## File: static\src\img\icons\Work_Accident_Time_Off_2.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <g id="Work_Accident_Time_Off" data-name="Work Accident Time Off">
    <g>
      <path d="M39,44.17H61A20.59,20.59,0,0,1,81.63,64.75V89.91a0,0,0,0,1,0,0H27.2a8.82,8.82,0,0,1-8.82-8.82V64.75A20.59,20.59,0,0,1,39,44.17Z" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <circle cx="50" cy="27.11" r="17.02" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <path d="M34,73.68H63.63c11.79,0,11.79,16.12,0,16.12" fill="none" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
      <polygon points="37.68 88.75 26.87 48.15 38.96 44.17 67.64 89.68 37.68 88.75" fill="#875b7b" stroke="#875b7b" stroke-linecap="round" stroke-linejoin="round" stroke-width="5"/>
    </g>
  </g>
</svg>

```

## File: static\src\js\accrual_levels_widget.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { X2ManyField, x2ManyField } from "@web/views/fields/x2many/x2many_field";

export class AccrualLevelsX2ManyField extends X2ManyField {};
AccrualLevelsX2ManyField.template = "hr_holidays.AccrualLevelsX2ManyField";

export const accrualLevelsX2ManyField = {
    ...x2ManyField,
    component: AccrualLevelsX2ManyField,
};

registry.category("fields").add("accrual_levels_one2many", accrualLevelsX2ManyField);

```

## File: static\src\leave_stats\leave_stats.js

```javascript
/* @odoo-module */

import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { useRecordObserver } from "@web/model/relational_model/utils";

import { formatDate } from "@web/core/l10n/dates";
import { Component, useState, onWillStart } from "@odoo/owl";

const { DateTime } = luxon;

export class LeaveStatsComponent extends Component {
    setup() {
        this.orm = useService("orm");

        this.state = useState({
            leaves: [],
            departmentLeaves: [],
        });

        this.date = this.props.record.data.date_from || DateTime.now();
        this.department = this.props.record.data.department_id;
        this.employee = this.props.record.data.employee_id;

        onWillStart(async () => {
            await this.loadLeaves(this.date, this.employee);
            await this.loadDepartmentLeaves(this.date, this.department, this.employee);
        });

        useRecordObserver(async (record) => {
            const dateFrom = record.data.date_from || DateTime.now();
            const dateChanged = this.date !== dateFrom;
            const employee = record.data.employee_id;
            const department = record.data.department_id;

            const proms = [];
            if (dateChanged || (employee && (this.employee && this.employee[0]) !== employee[0])) {
                proms.push(this.loadLeaves(dateFrom, employee));
            }

            if (
                dateChanged ||
                (department && (this.department && this.department[0]) !== department[0])
            ) {
                proms.push(this.loadDepartmentLeaves(dateFrom, department, employee));
            }
            await Promise.all(proms);

            this.date = dateFrom;
            this.employee = employee;
            this.department = department;
        });
    }

    get thisYear() {
        return this.date.toFormat("yyyy");
    }

    async loadDepartmentLeaves(date, department, employee) {
        if (!(department && employee && date)) {
            this.state.departmentLeaves = [];
            return;
        }

        const dateFrom = date.startOf("month");
        const dateTo = date.endOf("month");

        const departmentLeaves = await this.orm.searchRead(
            "hr.leave",
            [
                ["department_id", "=", department[0]],
                ["state", "=", "validate"],
                ["holiday_type", "=", "employee"],
                ["date_from", "<=", dateTo],
                ["date_to", ">=", dateFrom],
            ],
            ["employee_id", "date_from", "date_to", "number_of_days"]
        );

        this.state.departmentLeaves = departmentLeaves.map((leave) => {
            return Object.assign({}, leave, {
                dateFrom: formatDate(DateTime.fromSQL(leave.date_from, { zone: "utc" }).toLocal()),
                dateTo: formatDate(DateTime.fromSQL(leave.date_to, { zone: "utc" }).toLocal()),
                sameEmployee: leave.employee_id[0] === employee[0],
            });
        });
    }

    async loadLeaves(date, employee) {
        if (!(employee && date)) {
            this.state.leaves = [];
            return;
        }

        const dateFrom = date.startOf("year");
        const dateTo = date.endOf("year");
        this.state.leaves = await this.orm.readGroup(
            "hr.leave",
            [
                ["employee_id", "=", employee[0]],
                ["state", "=", "validate"],
                ["date_from", "<=", dateTo],
                ["date_to", ">=", dateFrom],
            ],
            ["holiday_status_id", "number_of_days:sum"],
            ["holiday_status_id"]
        );
    }
}

LeaveStatsComponent.template = "hr_holidays.LeaveStatsComponent";

export const leaveStatsComponent = {
    component: LeaveStatsComponent,
};
registry.category("view_widgets").add("hr_leave_stats", leaveStatsComponent);

```

## File: static\src\leave_stats\leave_stats.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates xml:space="preserve">
    <div t-name="hr_holidays.LeaveStatsComponent" class="o_leave_stats">
        <div t-if="employee" id="o_leave_stats_employee">
            <div class="o_hr_leave_subtitle">
                <t t-esc="employee[1]"/> in <t t-esc="thisYear"/>
            </div>
            <div t-if="state.leaves.length === 0">
                None
            </div>
            <div t-foreach="state.leaves" t-as="leave" t-key="leave_index" class="d-flex flex-row justify-content-between">
                <span t-esc="leave.holiday_status_id[1]"/>
                <span><t t-esc="leave.number_of_days"/> day(s)</span>
            </div>
        </div>

        <div t-if="department" id="o_leave_stats_department">
            <div class="o_horizontal_separator o_hr_leave_subtitle">
                <t t-esc="department[1]"/>
            </div>
            <div t-if="state.departmentLeaves.length === 0">
                None
            </div>
            <div t-foreach="state.departmentLeaves" t-as="leave" t-key="leave_index" t-attf-class="d-flex flex-row justify-content-between {{leave.sameEmployee ? 'fw-bold': ''}}">
                <span><t t-esc="leave.employee_id[1]"/>: <t t-esc="leave.number_of_days"/> day(s)</span>
                <span><t t-esc="leave.dateFrom"/> - <t t-esc="leave.dateTo"/></span>
            </div>
        </div>
    </div>
</templates>

```

## File: static\src\radio_image_field\radio_image_field.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { RadioField, radioField } from "@web/views/fields/radio/radio_field";

class RadioImageField extends RadioField {}
RadioImageField.template = "hr_holidays.RadioImageField";

registry.category("fields").add("hr_holidays_radio_image", {
    ...radioField,
    component: RadioImageField,
});

```

## File: static\src\radio_image_field\radio_image_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="hr_holidays.RadioImageField">
        <div role="radiogroup" class="d-flex flex-wrap gap-4" t-att-aria-label="string">
            <t t-if="props.readonly">
                <t t-if="value !== false">
                    <div>
                        <img t-attf-src="/hr_holidays/static/src/img/icons/{{items.find((item) => item[0] === value)[1]}}" width="48" height="48" />
                    </div>
                </t>
            </t>
            <t t-else="">
                <t t-foreach="items" t-as="item" t-key="item[0]">
                    <div class="form-check o_radio_item" aria-atomic="true">
                        <input
                            type="radio"
                            class="form-check-input o_radio_input"
                            t-att-checked="item[0] === value"
                            t-att-disabled="props.readonly"
                            t-att-name="id"
                            t-att-data-value="item[0]"
                            t-att-data-index="item_index"
                            t-att-id="`${id}_${item[0]}`"
                            t-on-change="() => this.onChange(item)"
                        />
                        <label class="form-check-label o_form_label" t-att-for="`${id}_${item[0]}`">
                            <img t-attf-src="/hr_holidays/static/src/img/icons/{{item[1]}}" width="48" height="48" />
                        </label>
                    </div>
                </t>
            </t>
        </div>
    </t>

</templates>

```

## File: static\src\tours\hr_holidays_tour.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { stepUtils } from "@web_tour/tour_service/tour_utils";

const leaveType = "NotLimitedHR";
const leaveDateFrom = "01/17/2022";
const leaveDateTo = "01/17/2022";
const description = 'Days off';

registry.category("web_tour.tours").add('hr_holidays_tour', {
    url: '/web',
    rainbowManMessage: _t("Congrats, we can see that your request has been validated."),
    test: false,
    steps: () => [
    stepUtils.showAppsMenuItem(),
    {
        trigger: '.o_app[data-menu-xmlid="hr_holidays.menu_hr_holidays_root"]',
        content: _t("Let's discover the Time Off application"),
        position: 'bottom',
    },
    {
        trigger: 'button.btn-time-off',
        content: _t("Click on any date or on this button to request a time-off"),
        position: 'bottom',
    },
    {
        trigger: 'div[name="holiday_status_id"] input',
        content: _t("Let's try to create a Sick Time Off, select it in the list"),
        run: `text ${leaveType.slice(0, leaveType.length - 1)}`,
    },
    {
        trigger: `.ui-autocomplete .ui-menu-item a:contains("${leaveType}")`,
        run: "click",
        auto: true,
        in_modal: false,
    },
    {
        trigger: 'input[data-field=request_date_from]',
        extra_trigger: `.o_field_widget[name='holiday_status_id'] input:propValue("${leaveType}")`,
        content: _t("You can select the period you need to take off, from start date to end date"),
        position: 'right',
        run: `text ${leaveDateFrom}`,
    },
    {
        trigger: 'input[data-field=request_date_to]',
        content: _t("You can select the period you need to take off, from start date to end date"),
        position: 'right',
        run: `text ${leaveDateTo}`,
    },
    {
        trigger: 'div[name="name"] textarea',
        content: _t("Add some description for the people that will validate it"),
        run: `text ${description}`,
        position: 'right'
    },
    {
        trigger: `button:contains(${_t('Save')})`,
        content: _t("Submit your request"),
        position: 'bottom',
    },
    {
        trigger: 'button[data-menu-xmlid="hr_holidays.menu_hr_holidays_management"]',
        content: _t("Let's go validate it"),
        position: 'bottom'
    },
    {
        trigger: 'a[data-menu-xmlid="hr_holidays.menu_open_department_leave_approve"]',
        content: _t("Select Time Off"),
        position: 'right'
    },
    {
        trigger: 'table.o_list_table',
        content: _t("Select the request you just created"),
        position: 'bottom',
        run: function(actions) {
            const rows = this.$anchor.find('tr.o_data_row');
            actions.click(rows[0]);
        }
    },
    {
        trigger: 'button[name="action_approve"]',
        content: _t("Let's approve it"),
        position: 'bottom',
    },
    {
        trigger: `tr.o_data_row:first:not(:has(button[name="action_approve"])),table tbody:not(tr.o_data_row)`,
        content: "Verify leave is approved",
        auto: true,
        isCheck: true,
    }
]});

registry.category("web_tour.tours").add('hr_holidays_launch', {
    url: '/web',
    steps: () => [
        stepUtils.showAppsMenuItem(),
        {
            trigger: '.o_app[data-menu-xmlid="hr_holidays.menu_hr_holidays_root"]',
            run: "click",
        },
        {
            trigger: '.o_calendar_container',
            run: ()=>{},
        },
    ]
});

```

## File: static\src\views\hooks.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { useService, useOwnedDialogs } from "@web/core/utils/hooks";
import { FormViewDialog } from "@web/views/view_dialogs/form_view_dialog";
import { useComponent } from "@odoo/owl";
import { pyToJsLocale } from "@web/core/l10n/utils";

export function formatNumber(lang, number, maxDecimals = 2) {
    const userLang = pyToJsLocale(lang);

    const numberFormat = new Intl.NumberFormat(userLang, { maximumFractionDigits: maxDecimals });
    return numberFormat.format(number);
}

export function useMandatoryDays(props) {
    return (info) => {
        const date = luxon.DateTime.fromJSDate(info.date).toISODate();
        const mandatoryDay = props.model.mandatoryDays[date];
        if (mandatoryDay) {
            const dayNumberElTop = info.view.el.querySelector(
                `.fc-day-top[data-date="${info.el.dataset.date}"]`
            );
            const dayNumberEl = info.view.el.querySelector(
                `.fc-day[data-date="${info.el.dataset.date}"]`
            );
            if (dayNumberElTop) {
                dayNumberElTop.classList.add('hr_mandatory_day', `hr_mandatory_day_top_${mandatoryDay}`);
            }
            if (dayNumberEl) {
                dayNumberEl.classList.add('hr_mandatory_day',`hr_mandatory_day_${mandatoryDay}`);
            }
        }
        return props.model.mandatoryDays;
    };
}

export function useLeaveCancelWizard() {
    const action = useService("action");

    return (leaveId, callback) => {
        action.doAction(
            {
                name: _t("Delete Confirmation"),
                type: "ir.actions.act_window",
                res_model: "hr.holidays.cancel.leave",
                target: "new",
                views: [[false, "form"]],
                context: {
                    default_leave_id: leaveId,
                },
            },
            {
                onClose: callback,
            }
        );
    };
}

export function useNewAllocationRequest() {
    const addDialog = useOwnedDialogs();
    const component = useComponent();
    return async (employeeId, holidayStatusId) => {
        const context = {
            form_view_ref: "hr_holidays.hr_leave_allocation_view_form_dashboard",
            is_employee_allocation: true,
        };
        if (employeeId) {
            context["default_employee_id"] = employeeId;
            context["default_employee_ids"] = [employeeId];
            context["form_view_ref"] =
                "hr_holidays.hr_leave_allocation_view_form_manager_dashboard";
        }
        if (holidayStatusId) {
            context["default_holiday_status_id"] = holidayStatusId;
        }
        addDialog(FormViewDialog, {
            resModel: "hr.leave.allocation",
            title: _t("New Allocation"),
            context: context,
            onRecordSaved: () => {
                component.env.timeOffBus.trigger("update_dashboard");
            },
        });
    };
}

```

## File: static\src\views\calendar\calendar_controller.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { CalendarController } from "@web/views/calendar/calendar_controller";
import { FormViewDialog } from "@web/views/view_dialogs/form_view_dialog";
import { Dropdown, DropdownItem } from "@web/core/dropdown/dropdown";

import { serializeDate } from "@web/core/l10n/dates";

import { TimeOffCalendarFilterPanel } from "./filter_panel/calendar_filter_panel";
import { TimeOffFormViewDialog } from "../view_dialog/form_view_dialog";
import { useLeaveCancelWizard } from "../hooks";
import { EventBus, useSubEnv } from "@odoo/owl";

export class TimeOffCalendarController extends CalendarController {
    setup() {
        super.setup();
        useSubEnv({
            timeOffBus: new EventBus(),
        });
        this.leaveCancelWizard = useLeaveCancelWizard();
    }

    get employeeId() {
        return this.model.employeeId;
    }

    get filterPanelProps() {
        return {
            ...super.filterPanelProps,
            employee_id: this.employeeId,
        };
    }

    newTimeOffRequest() {
        const context = {};
        if (this.employeeId) {
            context["default_employee_id"] = this.employeeId;
        }
        if (this.model.meta.scale == "day") {
            context["default_date_from"] = serializeDate(
                this.model.data.range.start.set({ hours: 7 }),
                "datetime"
            );
            context["default_date_to"] = serializeDate(
                this.model.data.range.end.set({ hours: 19 }),
                "datetime"
            );
        }

        this.displayDialog(FormViewDialog, {
            resModel: "hr.leave",
            title: _t("New Time Off"),
            viewId: this.model.formViewId,
            onRecordSaved: () => {
                this.model.load();
                this.env.timeOffBus.trigger("update_dashboard");
            },
            context: context,
        });
    }

    _deleteRecord(resId, canCancel) {
        if (!canCancel) {
            this.displayDialog(ConfirmationDialog, {
                title: _t("Confirmation"),
                body: _t("Are you sure you want to delete this record?"),
                confirm: async () => {
                    await this.model.unlinkRecord(resId);
                    this.env.timeOffBus.trigger("update_dashboard");
                },
                cancel: () => {},
            });
        } else {
            this.leaveCancelWizard(resId, () => {
                this.model.load();
                this.env.timeOffBus.trigger("update_dashboard");
            });
        }
    }

    deleteRecord(record) {
        this._deleteRecord(record.id, record.rawRecord.can_cancel);
    }

    async editRecord(record, context = {}, shouldFetchFormViewId = true) {
        const onDialogClosed = () => {
            this.model.load();
            this.env.timeOffBus.trigger("update_dashboard");
        };

        return new Promise((resolve) => {
            this.displayDialog(
                TimeOffFormViewDialog,
                {
                    resModel: this.model.resModel,
                    resId: record.id || false,
                    context,
                    title: _t("Time Off Request"),
                    viewId: this.model.formViewId,
                    onRecordSaved: onDialogClosed,
                    onRecordDeleted: (record) => this._deleteRecord(record.resId, record.data.can_cancel),
                    onLeaveCancelled: onDialogClosed,
                    size: "md",
                },
                { onClose: () => resolve() }
            );
        });
    }
}
TimeOffCalendarController.components = {
    ...TimeOffCalendarController.components,
    Dropdown,
    DropdownItem,
    FilterPanel: TimeOffCalendarFilterPanel,
};

TimeOffCalendarController.template = "hr_holidays.CalendarController";

```

## File: static\src\views\calendar\calendar_controller.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="hr_holidays.CalendarController" t-inherit="web.CalendarController" t-inherit-mode="primary">
        <xpath expr="//Layout" position="inside">
            <t t-set-slot="control-panel-create-button">
                <span class="o_timeoff_buttons btn-group">
                    <div class="btn-group">
                        <button class="btn btn-primary btn-time-off " t-on-click="newTimeOffRequest" type="button">
                            New
                        </button>
                    </div>
                </span>
            </t>
        </xpath>
        <DatePicker position="replace"/>
    </t>

</templates>
```

## File: static\src\views\calendar\calendar_model.js

```javascript
/** @odoo-module */

import { CalendarModel } from '@web/views/calendar/calendar_model';
import {
    deserializeDate,
    deserializeDateTime,
    serializeDate,
    serializeDateTime,
} from "@web/core/l10n/dates";

export class TimeOffCalendarModel extends CalendarModel {
    setup(params, services) {
        super.setup(params, services);

        this.data.mandatoryDays = {};
        if (this.env.isSmall) {
            this.meta.scale = 'month';
        }
    }

    /**
     * @override
     */
    normalizeRecord(rawRecord) {
        let result = super.normalizeRecord(...arguments);
        if (rawRecord.employee_id) {
            const employee = rawRecord.employee_id[1];
            result.title = [employee, result.title].join(' ');
        }
        return result;
    }

    makeContextDefaults(record) {
        const context = super.makeContextDefaults(record);
        if (this.employeeId) {
            context["default_employee_id"] = this.employeeId;
        }

        function deserialize(str) {
            // "YYYY-MM-DD".length == 10
            return str.length > 10 ? deserializeDateTime(str) : deserializeDate(str);
        }
        if ("default_date_from" in context) {
            context["default_date_from"] = serializeDateTime(
                deserialize(context["default_date_from"]).set({ hours: 7 })
            );
        }
        if ("default_date_to" in context) {
            context["default_date_to"] = serializeDateTime(
                deserialize(context["default_date_to"]).set({ hours: 19 })
            );
        }
        return context;
    }

    async updateData(data) {
        await super.updateData(data);

        data.mandatoryDays = await this.fetchMandatoryDays(data);
    }

    /**
     * @override
     */
    fetchUnusualDays(data) {
        return this.orm.call(this.meta.resModel, "get_unusual_days", [
            serializeDateTime(data.range.start),
            serializeDateTime(data.range.end),
        ],
        {
            context: {
                'employee_id': this.employeeId,
            }
        });
    }

    async fetchMandatoryDays(data) {
        return this.orm.call("hr.employee", "get_mandatory_days", [
            this.employeeId,
            serializeDate(data.range.start, "datetime"),
            serializeDate(data.range.end, "datetime"),
        ]);
    }

    get mandatoryDays() {
        return this.data.mandatoryDays;
    }

    get employeeId() {
        return this.meta.context.employee_id && this.meta.context.employee_id[0] || null;
    }

    fetchRecords(data) {
        const { fieldNames, resModel } = this.meta;
        const context = {};
        if (!this.employeeId) {
            context['short_name'] = 1;
        }
        return this.orm.searchRead(resModel, this.computeDomain(data), fieldNames, { context });
    }
}

```

## File: static\src\views\calendar\calendar_renderer.js

```javascript
/** @odoo-module */

import { CalendarRenderer } from '@web/views/calendar/calendar_renderer';

import { TimeOffCalendarCommonRenderer } from './common/calendar_common_renderer';
import { TimeOffCalendarYearRenderer } from './year/calendar_year_renderer';

import { TimeOffDashboard } from '../../dashboard/time_off_dashboard';

export class TimeOffCalendarRenderer extends CalendarRenderer {
    get employeeId() {
        return this.props.model.employeeId;
    }

    get showDashboard() {
        return false;
    }
}
TimeOffCalendarRenderer.template = 'hr_holidays.CalendarRenderer';
TimeOffCalendarRenderer.components = {
    ...TimeOffCalendarRenderer.components,
    day: TimeOffCalendarCommonRenderer,
    week: TimeOffCalendarCommonRenderer,
    month: TimeOffCalendarCommonRenderer,
    year: TimeOffCalendarYearRenderer,
    TimeOffDashboard,
};

export class TimeOffDashboardCalendarRenderer extends TimeOffCalendarRenderer {
    get showDashboard() {
        return !this.env.isSmall;
    }
}

```

## File: static\src\views\calendar\calendar_renderer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="hr_holidays.CalendarRenderer">
        <div class="o_timeoff_calendar d-flex flex-column">
            <TimeOffDashboard t-if="showDashboard" employeeId="employeeId"/>
            <t t-call="web.CalendarRenderer"/>
        </div>
    </t>

</templates>

```

## File: static\src\views\calendar\calendar_view.js

```javascript
/** @odoo-module */

import { calendarView } from '@web/views/calendar/calendar_view';

import { TimeOffCalendarController } from './calendar_controller';
import { TimeOffCalendarModel } from './calendar_model';
import { TimeOffCalendarRenderer, TimeOffDashboardCalendarRenderer } from './calendar_renderer';

import { registry } from '@web/core/registry';

const TimeOffCalendarView = {
    ...calendarView,

    Controller: TimeOffCalendarController,
    Renderer: TimeOffCalendarRenderer,
    Model: TimeOffCalendarModel,
}

registry.category('views').add('time_off_calendar', TimeOffCalendarView);
registry.category('views').add('time_off_calendar_dashboard', {
    ...TimeOffCalendarView,
    Renderer: TimeOffDashboardCalendarRenderer,
});

```

## File: static\src\views\calendar\common\calendar_common_popover.js

```javascript
/** @odoo-module */

import { CalendarCommonPopover } from "@web/views/calendar/calendar_common/calendar_common_popover";

import { useService } from "@web/core/utils/hooks";

export class TimeOffCalendarCommonPopover extends CalendarCommonPopover {
    setup() {
        super.setup();

        this.dialog = useService('dialog');
        this.action = useService('action');
    }

    get isEventDeletable() {
        const record = this.props.record.rawRecord;
        const state = record.state;
        return record.can_cancel || state && !['validate', 'refuse'].includes(state);
    }

    get isEventEditable() {
        const state = this.props.record.rawRecord.state;
        return state !== undefined;
    }
}

```

## File: static\src\views\calendar\common\calendar_common_renderer.js

```javascript
/** @odoo-module */

import { CalendarCommonRenderer } from '@web/views/calendar/calendar_common/calendar_common_renderer';

import { useMandatoryDays } from '../../hooks';
import { TimeOffCalendarCommonPopover } from './calendar_common_popover';


export class TimeOffCalendarCommonRenderer extends CalendarCommonRenderer {
    setup() {
        super.setup();
        this.mandatoryDays = useMandatoryDays(this.props);
    }

    onDayRender(info) {
        super.onDayRender(info);
        this.mandatoryDays(info);
    }
}
TimeOffCalendarCommonRenderer.components = {
    ...TimeOffCalendarCommonRenderer,
    Popover: TimeOffCalendarCommonPopover,
}

```

## File: static\src\views\calendar\filter_panel\calendar_filter_panel.js

```javascript
/** @odoo-module */

import { CalendarFilterPanel } from "@web/views/calendar/filter_panel/calendar_filter_panel";
import { TimeOffCardMobile } from "../../../dashboard/time_off_card";
import { getFormattedDateSpan } from "@web/views/calendar/utils";

import { useService } from "@web/core/utils/hooks";
import { serializeDate } from "@web/core/l10n/dates";
import { useState, onWillStart, onWillUpdateProps } from "@odoo/owl";

export class TimeOffCalendarFilterPanel extends CalendarFilterPanel {
    setup() {
        super.setup();

        this.orm = useService("orm");
        this.getFormattedDateSpan = getFormattedDateSpan;
        this.leaveState = useState({
            holidays: [],
            mandatoryDays: [],
            bankHolidays: [],
        });

        onWillStart(async () => {
            await this.loadFilterData();
            await this.updateSpecialDays();
        });
        onWillUpdateProps(this.updateSpecialDays);
    }

    async updateSpecialDays() {
        const context = {
            employee_id: this.props.employee_id,
        };
        const specialDays = await this.orm.call(
            "hr.employee",
            "get_special_days_data",
            [
                serializeDate(this.props.model.rangeStart, "datetime"),
                serializeDate(this.props.model.rangeEnd, "datetime"),
            ],
            {
                context: context,
            }
        );
        specialDays["bankHolidays"].forEach((bankHoliday) => {
            bankHoliday.start = luxon.DateTime.fromISO(bankHoliday.start);
            bankHoliday.end = luxon.DateTime.fromISO(bankHoliday.end);
        });
        specialDays["mandatoryDays"].forEach((mandatoryDay) => {
            mandatoryDay.start = luxon.DateTime.fromISO(mandatoryDay.start);
            mandatoryDay.end = luxon.DateTime.fromISO(mandatoryDay.end);
        });
        this.leaveState.bankHolidays = specialDays["bankHolidays"];
        this.leaveState.mandatoryDays = specialDays["mandatoryDays"];
    }

    async loadFilterData() {
        if (!this.env.isSmall) {
            return;
        }

        const filterData = {};
        const data = await this.orm.call("hr.leave.type", "get_allocation_data_request", []);

        data.forEach((leave) => {
            filterData[leave[3]] = leave;
        });
        this.leaveState.holidays = filterData;
    }
}
TimeOffCalendarFilterPanel.template = "hr_holidays.CalendarFilterPanel";
TimeOffCalendarFilterPanel.components = {
    ...TimeOffCalendarFilterPanel.components,
    TimeOffCardMobile,
};
TimeOffCalendarFilterPanel.subTemplates = {
    filter: "hr_holidays.CalendarFilterPanel.filter",
};

```

## File: static\src\views\calendar\filter_panel\calendar_filter_panel.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="hr_holidays.CalendarFilterPanel" t-inherit="web.CalendarFilterPanel" t-inherit-mode="primary">
        <xpath expr="//t[@t-foreach='props.model.filterSections']" position="after">
            <div class="o_calendar_filter mt-4">
                <h5>Legend</h5>
                <div class="d-flex flex-column">
                    <span class="align-middle"><img class="o_calendar_filter_plain" src="/hr/static/src/img/icons/plain.svg"/> Validated</span>
                    <span class="align-middle"><img class="o_calendar_filter_hatched" src="/hr/static/src/img/icons/hatched.svg"/> To Approve</span>
                    <span class="align-middle"><img class="o_calendar_filter_line" src="/hr/static/src/img/icons/line.svg"/> Refused</span>
                </div>

                <div class="d-flex flex-column mt-4" t-if="leaveState.mandatoryDays.length">
                    <h5>Mandatory Days</h5>
                    <ul class="ps-0">
                        <li t-foreach="leaveState.mandatoryDays" t-as="mandatoryDay" t-key="mandatoryDay.id" class="mt-2 list-unstyled">
                            <strong
                                t-esc="getFormattedDateSpan(mandatoryDay.start, mandatoryDay.end)"
                                t-att-class="'hr_mandatory_day_'+mandatoryDay.colorIndex"/>
                            : <t t-esc="mandatoryDay.title"/>
                        </li>
                    </ul>
                </div>

                <div class="d-flex flex-column mt-4" t-if="leaveState.bankHolidays.length">
                    <h5>Public Holidays</h5>
                    <ul class="ps-0">
                        <li t-foreach="leaveState.bankHolidays" t-as="bankHoliday" t-key="bankHoliday.id" class="mt-2 list-unstyled">
                            <strong
                                t-esc="getFormattedDateSpan(bankHoliday.start, bankHoliday.end)"/>
                            : <t t-esc="bankHoliday.title"/>
                        </li>
                    </ul>
                </div>
            </div>
        </xpath>
    </t>


    <t t-name="hr_holidays.CalendarFilterPanel.filter" t-inherit="web.CalendarFilterPanel.filter" t-inherit-mode="primary">
        <xpath expr="//span[@t-esc='filter.label']" position="replace">
            <span class="o_cw_filter_title flex-grow-1 text-truncate lh-base">
                <t t-esc="filter.label"/>

                <t t-if="env.isSmall">
                    <t t-set="holiday" t-value="leaveState.holidays[filter.value]"/>
                    <TimeOffCardMobile t-if="holiday" name="holiday[0]" id="holiday[3]" data="holiday[1]" requires_allocation="holiday[2] === 'yes'"/>
                </t>
            </span>
        </xpath>
    </t>
</templates>

```

## File: static\src\views\calendar\year\calendar_year_popover.js

```javascript
/** @odoo-module **/

import { Dialog } from "@web/core/dialog/dialog";

import { CalendarYearPopover } from "@web/views/calendar/calendar_year/calendar_year_popover";

export class TimeOffCalendarYearPopover extends CalendarYearPopover {}
TimeOffCalendarYearPopover.components = { Dialog };
TimeOffCalendarYearPopover.template = "web.CalendarYearPopover";
TimeOffCalendarYearPopover.subTemplates = {
    ...CalendarYearPopover.subTemplates,
    body: "hr_holidays.MandatoryDayCalendarYearPopover.body",
};

```

## File: static\src\views\calendar\year\calendar_year_popover.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="hr_holidays.MandatoryDayCalendarYearPopover.body">
        <t t-foreach="recordGroups" t-as="recordGroup" t-key="recordGroup.title">
            <div class="fw-bold mt-2" t-esc="recordGroup.title" />
            <t t-foreach="recordGroup.records" t-as="record" t-key="record.id">
                <t t-if="record.id &lt; 0">
                    <p class="">
                        <t t-if="record.startHour"><t t-esc="record.startHour" /> </t>
                        <t t-esc="record.title"/>
                    </p>
                </t>
                <t t-else="">
                    <t t-call="{{ constructor.subTemplates.record }}" />
                </t>
            </t>
        </t>
    </t>
</templates>

```

## File: static\src\views\calendar\year\calendar_year_renderer.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { CalendarYearRenderer } from "@web/views/calendar/calendar_year/calendar_year_renderer";

import { useService } from "@web/core/utils/hooks";
import { useMandatoryDays } from "../../hooks";
import { useCalendarPopover } from "@web/views/calendar/hooks";
import { TimeOffCalendarYearPopover } from "./calendar_year_popover";
import { useEffect } from "@odoo/owl";

export class TimeOffCalendarYearRenderer extends CalendarYearRenderer {
    setup() {
        super.setup();
        this.orm = useService("orm");
        this.mandatoryDays = useMandatoryDays(this.props);
        this.mandatoryDaysList = [];
        this.mandatoryDayPopover = useCalendarPopover(TimeOffCalendarYearPopover);

        useEffect(
            (el) => {
                for (const week of el) {
                    const row = week.parentElement;

                    // Remove the week number if the week is empty.
                    // FullCalendar always displays 6 weeks even when empty.
                    if (
                        !row.children[1].classList.length &&
                        !row.children[row.children.length - 1].classList.length
                    ) {
                        row.remove();
                    }
                }
            },
            () => [
                this.rootRef.el &&
                    this.rootRef.el.querySelectorAll(".fc-content-skeleton td.fc-week-number"),
            ]
        );
    }

    get options() {
        return Object.assign(super.options, {
            weekNumbers: true,
            weekNumbersWithinDays: false,
            weekLabel: _t("Week"),
        });
    }

    /** @override **/
    async onDateClick(info) {
        const is_mandatory_day = [...info.dayEl.classList].some((elClass) =>
            elClass.startsWith("hr_mandatory_day_")
        );
        this.mandatoryDayPopover.close();
        if (is_mandatory_day && !this.env.isSmall) {
            this.popover.close();
            const date = luxon.DateTime.fromISO(info.dateStr);
            const target = info.dayEl;
            const mandatory_days_data = await this.orm.call(
                "hr.employee",
                "get_mandatory_days_data",
                [date, date]
            );
            mandatory_days_data.forEach((mandatory_day_data) => {
                mandatory_day_data["start"] = luxon.DateTime.fromISO(mandatory_day_data["start"]);
                mandatory_day_data["end"] = luxon.DateTime.fromISO(mandatory_day_data["end"]);
            });
            const records = Object.values(this.props.model.records).filter((r) =>
                luxon.Interval.fromDateTimes(r.start.startOf("day"), r.end.endOf("day")).contains(
                    date
                )
            );
            const props = this.getPopoverProps(date, records);
            props["records"] = mandatory_days_data.concat(props["records"]);
            this.mandatoryDayPopover.open(target, props, "o_cw_popover");
        } else {
            super.onDateClick(info);
        }
    }

    onDayRender(info) {
        super.onDayRender(info);
        this.mandatoryDaysList = this.mandatoryDays(info);
    }
}

```

## File: static\src\views\view_dialog\form_view_dialog.js

```javascript
/** @odoo-module */

import { FormViewDialog } from "@web/views/view_dialogs/form_view_dialog";

import { registry } from '@web/core/registry';
import { useService } from "@web/core/utils/hooks";

import { formView } from '@web/views/form/form_view';
import { FormController } from '@web/views/form/form_controller';

import { useLeaveCancelWizard } from '../hooks';

export class TimeOffDialogFormController extends FormController {
    setup() {
        super.setup();
        this.leaveCancelWizard = useLeaveCancelWizard();
        this.orm = useService("orm");
    }

    get record() {
        return this.model.root;
    }

    async onClick(action) {
        const args = (action === 'action_approve') ? [this.record.resId, false] : [this.record.resId];
        await this.orm.call("hr.leave", action, args);
        this.props.onLeaveUpdated();
    }

    get canApprove() {
        return !this.model.root.isNew && (this.record.data.can_approve && ['confirm', 'refuse',].includes(this.record.data.state));
    }

    get canValidate() {
        return !this.model.root.isNew && (this.record.data.can_approve && this.record.data.state === 'validate1');
    }

    get canRefuse() {
        return !this.model.root.isNew && (this.record.data.can_approve && this.record.data.state && ['confirm', 'validate1', 'validate'].includes(this.record.data.state));
    }

    deleteRecord() {
        this.props.onRecordDeleted(this.record);
        this.props.onCancelLeave();
        if (this.record.data.can_cancel) {
            this.leaveCancelWizard(this.record.resId, () => {
                this.props.onLeaveCancelled();
            });
        }
    }

    get canDelete() {
        return !this.model.root.isNew && (this.record.data.can_cancel || this.record.data.state && !['validate', 'refuse'].includes(this.record.data.state));
    }
}

TimeOffDialogFormController.props = {
    ...FormController.props,
    onCancelLeave: Function,
    onRecordDeleted: Function,
    onLeaveCancelled: Function,
    onLeaveUpdated: Function,
}

registry.category('views').add('timeoff_dialog_form', {
    ...formView,
    Controller: TimeOffDialogFormController,
});


export class TimeOffFormViewDialog extends FormViewDialog {
    setup() {
        super.setup();

        this.viewProps = Object.assign(this.viewProps, {
            type: "timeoff_dialog_form",
            buttonTemplate: 'hr_holidays.FormViewDialog.buttons',
            onCancelLeave: () => {
                this.props.close();
            },
            onLeaveUpdated: () => {
                this.props.onRecordSaved();
                this.props.close();
            },
            onRecordDeleted: (record) => {
                this.props.onRecordDeleted(record)
            },
            onLeaveCancelled: this.props.onLeaveCancelled.bind(this),
        })
    }
}
TimeOffFormViewDialog.props = {
    ...TimeOffFormViewDialog.props,
    onRecordDeleted: Function,
    onLeaveCancelled: Function,
}

```

## File: static\src\views\view_dialog\form_view_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="hr_holidays.FormViewDialog.buttons" t-inherit="web.FormViewDialog.ToOne.buttons">
        <xpath expr="//button[contains(@class, 'o_form_button_save')]" position="replace">
            <button class="btn btn-primary o_form_button_save" t-on-click="saveButtonClicked" data-hotkey="q">Save</button>
        </xpath>

        <xpath expr="//button[contains(@class, 'o_form_button_cancel')]" position="after">
            <button class="btn btn-secondary" t-if="canDelete" t-on-click="deleteRecord" data-hotkey="x">Delete</button>
        </xpath>
        <xpath expr="//div[contains(@class, 'o_cp_buttons')]" position="inside">
            <div class="flex-grow-1 d-flex flex-row-reverse gap-1">
                <button class="btn btn-secondary" t-if="canRefuse" t-on-click="() => this.onClick('action_refuse')" data-hotkey="z">Refuse</button>
                <button class="btn btn-secondary" t-if="canApprove" t-on-click="() => this.onClick('action_approve')" data-hotkey="g">Approve</button>
                <button class="btn btn-secondary" t-if="canValidate" t-on-click="() => this.onClick('action_validate')" data-hotkey="v">Validate</button>
            </div>
        </xpath>
    </t>
</templates>

```

## File: static\src\xml\x2many_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="hr_holidays.AccrualLevelsX2ManyField" t-inherit="web.X2ManyField" t-inherit-mode="primary">
        <xpath expr="//div[hasclass('o_x2m_control_panel')]/.." position="before">
            <xpath expr="//ListRenderer" position="move"/>
            <xpath expr="//KanbanRenderer" position="move"/>
        </xpath>
    </t>
</templates>

```

## File: upgrades\1.6\pre-migrate.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

def migrate(cr, version):
    cr.execute("""
      UPDATE ir_rule r
        SET domain_force = '["|", ("employee_id", "=", False), ("employee_id.company_id", "in", company_ids), "|", ("holiday_status_id.company_id", "=", False), ("holiday_status_id.company_id", "in", company_ids)]'
        FROM ir_model_data d
        WHERE d.res_id = r.id
          AND d.model = 'ir.rule'
          AND d.module = 'hr_holidays'
          AND d.name = 'hr_leave_allocation_rule_multicompany'
    """)

```

## File: views\hr_holidays_views.xml

```xml
<?xml version='1.0' encoding='UTF-8' ?>
<odoo>

    <menuitem
        name="Time Off"
        id="menu_hr_holidays_root"
        sequence="225"
        web_icon="hr_holidays,static/description/icon.png"
        groups="base.group_user"/>

    <menuitem
        id="menu_hr_holidays_my_leaves"
        name="My Time"
        parent="menu_hr_holidays_root"
        sequence="1"/>

    <menuitem
        id="hr_leave_menu_new_request"
        parent="menu_hr_holidays_my_leaves"
        action="hr_leave_action_new_request"
        sequence="1"/>

    <menuitem
        id="hr_leave_menu_my"
        parent="menu_hr_holidays_my_leaves"
        action="hr_leave_action_my"
        sequence="2"/>

    <menuitem
        id="menu_open_allocation"
        name="My Allocations"
        parent="menu_hr_holidays_my_leaves"
        action="hr_leave_allocation_action_my"
        sequence="3"/>

    <menuitem
        id="menu_hr_holidays_dashboard"
        name="Overview"
        parent="menu_hr_holidays_root"
        sequence="2"
        action="action_hr_holidays_dashboard"/>

    <menuitem
        id="menu_hr_holidays_management"
        name="Management"
        parent="menu_hr_holidays_root"
        sequence="3"
        groups="hr_holidays.group_hr_holidays_responsible"/>

    <menuitem
        id="menu_open_department_leave_approve"
        name="Time Off"
        parent="menu_hr_holidays_management"
        action="hr_leave_action_action_approve_department"
        sequence="1"/>

    <menuitem
        id="hr_holidays_menu_manager_approve_allocations"
        name="Allocations"
        parent="menu_hr_holidays_management"
        action="hr_leave_allocation_action_approve_department"
        sequence="2"
        groups="hr_holidays.group_hr_holidays_user"/>

    <menuitem
        id="menu_hr_holidays_report"
        name="Reporting"
        parent="menu_hr_holidays_root"
        groups="hr_holidays.group_hr_holidays_user"
        sequence="4"/>

    <menuitem
        id="menu_hr_available_holidays_report_tree"
        name="by Employee"
        parent="menu_hr_holidays_report"
        action="action_hr_available_holidays_report"
        sequence="1"/>

    <menuitem
        id="menu_hr_holidays_summary_all"
        name="by Type"
        parent="menu_hr_holidays_report"
        action="act_hr_employee_holiday_type"
        sequence="3"/>

    <menuitem
        id="menu_hr_holidays_configuration"
        name="Configuration"
        parent="menu_hr_holidays_root"
        groups="hr_holidays.group_hr_holidays_manager"
        sequence="5"/>

    <menuitem
        id="hr_holidays_status_menu_configuration"
        action="open_view_holiday_status"
        name="Time Off Types"
        parent="menu_hr_holidays_configuration"
        groups="hr_holidays.group_hr_holidays_manager"
        sequence="1"/>

    <menuitem
        id="hr_holidays_accrual_menu_configuration"
        action="open_view_accrual_plans"
        name="Accrual Plans"
        parent="menu_hr_holidays_configuration"
        groups="hr_holidays.group_hr_holidays_manager"
        sequence="2"/>

    <menuitem
        id="hr_holidays_public_time_off_menu_configuration"
        action="open_view_public_holiday"
        name="Public Holidays"
        parent="menu_hr_holidays_configuration"
        groups="hr_holidays.group_hr_holidays_manager"
        sequence="3"/>

    <menuitem
        id="hr_holidays_mandatory_day_menu_configuration"
        action="hr_leave_mandatory_day_action"
        name="Mandatory Days"
        parent="menu_hr_holidays_configuration"
        groups="hr_holidays.group_hr_holidays_manager"
        sequence="4"/>

    <menuitem id="hr_holidays_menu_config_activity_type"
        action="mail_activity_type_action_config_hr_holidays"
        parent="menu_hr_holidays_configuration"
        groups="base.group_no_one"/>

</odoo>

```

## File: views\hr_leave_accrual_views.xml

```xml
<?xml version='1.0' encoding='UTF-8' ?>
<odoo>
    <record id="hr_accrual_level_view_form" model="ir.ui.view">
        <field name="name">hr.leave.accrual.level.form</field>
        <field name="model">hr.leave.accrual.level</field>
        <field name="arch" type="xml">
            <form string="Accrual Level">
                <sheet>
                    <group name="accrue" col="1" width="800px">
                        <div class="o_td_label">
                            <label for="added_value" string="Employee accrue"/>
                        </div>
                        <div>
                            <field name="accrued_gain_time" invisible="1"/>
                            <field name="can_modify_value_type" invisible="1"/>
                            <field name="added_value" widget="FloatWithoutTrailingZeros" style="width: 4rem" class="me-1"/>
                            <field name="added_value_type" style="width: 3.4rem" nolabel="1" readonly="not can_modify_value_type"/>
                        </div>
                        <div style="width: 5rem"/>
                        <div name="hourly" invisible="frequency != 'hourly'">
                            <field name="frequency" style="width: 5rem"/>
                        </div>
                        <div name="daily" invisible="frequency != 'daily'">
                            <field name="frequency" style="width: 5rem"/>
                        </div>
                        <div name="weekly" invisible="frequency != 'weekly'">
                            <field name="frequency" style="width: 4.5rem;"/>
                            <label for="week_day" string="on" class="me-1"/><field name="week_day" style="width: 6.6rem"/>
                        </div>
                        <div name="monthly" invisible="frequency != 'monthly'">
                            <field name="frequency" style="width: 4.5rem"/>
                            <label for="first_day_display" string="on the" class="me-1"/>
                            <field name="first_day_display" required="1" style="width: 4rem"/>
                            of the month
                        </div>
                        <div name="bimonthly" invisible="frequency != 'bimonthly'" style="width: 100%">
                            <field name="frequency" style="width: 7rem"/>
                            <label for="first_day_display" string="on the" class="me-1"/><field name="first_day_display" required="1" style="width: 4rem"/>
                            <label for="second_day_display" string="and on the" class="me-1"/><field name="second_day_display" required="1" style="width: 4.1rem"/>
                            of the month
                        </div>
                        <div name="biyearly" invisible="frequency != 'biyearly'">
                            <field name="frequency" style="width: 6rem"/>
                            <label for="first_month_day_display" string="on the" class="me-1"/>
                            <field name="first_month_day_display" required="1" style="width: 4.1rem"/>of <field name="first_month" required="1" style="width: 4.55rem"/>
                            <label for="second_month_day_display" string="and" class="me-1"/>
                            <field name="second_month_day_display" required="1" style="width: 4.1rem"/>of <field name="second_month" required="1" style="width: 5.4rem"/>
                        </div>
                        <div name="yearly" invisible="frequency != 'yearly'">
                            <field name="frequency" style="width: 4rem"/>
                            <label for="yearly_day_display" string="on the" class="me-1"/>
                            <field name="yearly_day_display" required="1" style="width: 4rem"/> of <field name="yearly_month" required="1" style="width: 5.4rem"/>
                        </div>
                    </group>
                    <group name="maximum_leave">
                        <field name="cap_accrued_time" style="width: 9.35rem"/>
                        <div style="width: 9.35rem"/>
                        <div invisible="not cap_accrued_time">
                            <field name="maximum_leave" style="width: 5rem" widget="FloatWithoutTrailingZeros"/>
                            <field name="added_value_type" style="white-space: nowrap;width: 5rem"/>
                        </div>
                    </group>
                    <group name="milestone">
                        <div class="o_td_label">
                            <label for="start_count" string="Milestone reached"/>
                        </div>
                        <div>
                            <field name="start_count" style="width: 2rem"/>
                            <field name="start_type" style="width: 4.75rem"/> after allocation start date
                        </div>
                        <div class="o_td_label">
                            <label for="action_with_unused_accruals" string="Carry over"/>
                        </div>
                        <div>
                            <field name="action_with_unused_accruals" class="o_light_label" style="font-weight: 0" widget="radio"/><br/>
                            <div invisible="action_with_unused_accruals != 'maximum'">
                                <label for="postpone_max_days" string="Up to"/>
                                <field name="postpone_max_days" style="width: 3rem"/>
                                <field name="added_value_type" nolabel="1" readonly="1" force_save="1" style="width: 5rem"/>
                            </div>
                        </div>
                        <field name="postpone_max_days" class="w-25" invisible="action_with_unused_accruals != 'postponed'"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>
    <record id="hr_accrual_plan_view_tree" model="ir.ui.view">
        <field name="name">hr.leave.accrual.plan.tree</field>
        <field name="model">hr.leave.accrual.plan</field>
        <field name="arch" type="xml">
            <tree string="Accrual Plans">
                <field name="name"/>
                <field name="level_count"/>
                <field name="employees_count"/>
            </tree>
        </field>
    </record>
    <record id="hr_accrual_plan_view_form" model="ir.ui.view">
        <field name="name">hr.leave.accrual.plan.form</field>
        <field name="model">hr.leave.accrual.plan</field>
        <field name="arch" type="xml">
            <form string="Accrual Plan">
                <field name="active" invisible="1"/>
                <field name="show_transition_mode" invisible="1" />
                <sheet>
                    <div class="oe_button_box" name="button_box" invisible="not id" >
                        <button name="action_open_accrual_plan_employees" type="object" class="oe_stat_button" icon="fa-users" invisible="employees_count == 0">
                            <field name="employees_count" widget="statinfo"/>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" invisible="active"/>
                    <group>
                        <group>
                            <field name="name"/>
                            <field name="accrued_gain_time" widget="radio"/>
                            <field name="carryover_date" widget="radio"/>
                            <label for="carryover_month" string="Carry-Over Date" invisible="carryover_date != 'other'"/>
                            <div name="carryover" invisible="carryover_date != 'other'">
                                <field name="carryover_day" invisible="1"/>
                                <field name="carryover_day_display"  required="1" style="width: 4.75rem;"/> of
                                <field name="carryover_month"  required="1" style="width: 6rem;"/>
                            </div>
                        </group>
                        <group>
                            <field name="is_based_on_worked_time" readonly="accrued_gain_time == 'start'"/>
                            <field name="transition_mode" widget="radio" invisible="not show_transition_mode"/>
                            <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}"/>
                        </group>
                    </group>
                    <span class="oe_grey" invisible="1">
                    </span>
                    <div class="o_hr_holidays_hierarchy">
                        <div class="o_hr_holidays_title">Rules</div>
                        <div class="o_hr_holidays_hierarchy_readonly" invisible="level_ids">
                            <p>
                                No rule has been set up for this accrual plan.
                            </p>
                        </div>
                        <field name="level_ids" mode="kanban" nolabel="1"
                            class="o_hr_holidays_plan_level_container"
                            widget="accrual_levels_one2many"
                            add-label="New Milestone"
                            >
                            <kanban default_order="sequence">
                                <field name="sequence"/>
                                <field name="start_count"/>
                                <field name="start_type"/>
                                <field name="added_value"/>
                                <field name="added_value_type"/>
                                <field name="frequency"/>
                                <field name="week_day"/>
                                <field name="first_day"/>
                                <field name="second_day"/>
                                <field name="first_month_day"/>
                                <field name="first_month"/>
                                <field name="second_month_day"/>
                                <field name="second_month"/>
                                <field name="yearly_day"/>
                                <field name="yearly_month"/>
                                <field name="maximum_leave"/>
                                <field name="action_with_unused_accruals"/>
                                <field name="cap_accrued_time"/>
                                <templates>
                                    <div t-name="kanban-box" class="border-0 bg-transparent">
                                        <div class="o_hr_holidays_body oe_kanban_global_click">
                                            <div class="o_hr_holidays_timeline text-center">
                                                <t t-if="record.start_count.raw_value > 0">
                                                    after <t t-esc="record.start_count.value"/> <t t-esc="record.start_type.value"/>
                                                </t>
                                                <t t-else="">
                                                    initially
                                                </t>
                                            </div>
                                            <t t-if="!read_only_mode">
                                                <a type="edit" t-attf-class="oe_kanban_action oe_kanban_action_a text-black">
                                                    <t t-call="level_content"/>
                                                </a>
                                            </t>
                                            <t t-else="">
                                                <t t-call="level_content"/>
                                            </t>
                                        </div>
                                    </div>
                                    <t t-name="level_content">
                                        <div class="o_hr_holidays_card">
                                            <div class="content container" style="width: 560px;">
                                                <div class="row w-100">
                                                    <div class="pe-0 me-0" style="width: 6rem;">
                                                        <field name="added_value" widget="FloatWithoutTrailingZeros"/> <field name="added_value_type"/>,
                                                    </div>
                                                    <div class="col-auto m-0 p-0">
                                                        <field name="frequency" class="ms-1"/>
                                                        <t t-if="record.frequency.raw_value === 'weekly'">
                                                            on <field name="week_day"/>
                                                        </t>
                                                        <t t-elif="record.frequency.raw_value === 'monthly'">
                                                            on the <field name="first_day"/> day of the month
                                                        </t>
                                                        <t t-elif="record.frequency.raw_value === 'bimonthly'">
                                                            on the <field name="first_day"/> and on the <field name="second_day"/> days of the months
                                                        </t>
                                                        <t t-elif="record.frequency.raw_value === 'biyearly'">
                                                            on the <field name="first_month_day"/> <field name="first_month"/> and on the <field name="second_month_day"/> <field name="second_month"/>
                                                        </t>
                                                        <t t-elif="record.frequency.raw_value === 'yearly'">
                                                            on <field name="yearly_day"/> <field name="yearly_month"/>
                                                        </t>
                                                    </div>
                                                </div>
                                                <div class="row text-muted">
                                                    <div class="pe-0 me-0" style="width: 6rem;">
                                                        Cap:
                                                    </div>
                                                    <div class="col-3 m-0 ps-1">
                                                        <t t-if="record.cap_accrued_time.raw_value &amp;&amp; record.maximum_leave.raw_value > 0">
                                                            <field name="maximum_leave" widget="FloatWithoutTrailingZeros"/> <field name="added_value_type"/>
                                                        </t>
                                                        <t t-else="">
                                                            Unlimited
                                                        </t>
                                                    </div>
                                                </div>
                                                <div class="row text-muted">
                                                    <div class="pe-0 me-0" style="width: 6rem;">
                                                        Carry over:
                                                    </div>
                                                    <div class="col-3 m-0 ps-1">
                                                        <t t-if="record.action_with_unused_accruals.raw_value === 'all'">all</t>
                                                        <t t-elif="record.action_with_unused_accruals.raw_value == 'maximum'">up to <field name="postpone_max_days" /> <t t-esc="record.added_value_type.raw_value" /></t>
                                                        <t t-else="">no</t>
                                                    </div>
                                                </div>
                                            </div>
                                        </div>
                                    </t>
                                </templates>
                            </kanban>
                        </field>
                    </div>
               </sheet>
            </form>
        </field>
    </record>
    <record id="hr_accrual_plan_view_search" model="ir.ui.view">
        <field name="name">hr.leave.accrual.plan.search</field>
        <field name="model">hr.leave.accrual.plan</field>
        <field name="arch" type="xml">
            <search string="Group By">
                <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                <filter string="Company" name='company_id' context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
            </search>
        </field>
    </record>

    <record id="open_view_accrual_plans" model="ir.actions.act_window">
        <field name="name">Accrual Plans</field>
        <field name="res_model">hr.leave.accrual.plan</field>
        <field name="view_mode">tree,form</field>
    </record>

</odoo>

```

## File: views\hr_leave_allocation_views.xml

```xml
<?xml version='1.0' encoding='UTF-8' ?>
<odoo>

    <record id="view_hr_leave_allocation_filter" model="ir.ui.view">
        <field name="name">hr.holidays.filter_allocations</field>
        <field name="model">hr.leave.allocation</field>
        <field name="arch" type="xml">
            <search string="Search allocations">
                <field name="employee_id"/>
                <field name="name"/>
                <field name="allocation_type"/>
                <filter domain="[('state', '=', 'confirm')]" string="To Approve" name="second_approval"/>
                <filter string="Approved" domain="[('state', '=', 'validate')]" name="validated"/>
                <separator/>
                <filter name="year" string="Currently Valid"
                    domain="[
                        ('date_from', '&lt;=', context_today().strftime('%Y-%m-%d')),
                        '|',
                        ('date_to', '=', False),
                        ('date_to', '&gt;=', context_today().strftime('%Y-%m-%d')),
                    ]"
                    help="Active Allocations"/>
                <separator/>
                <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction','=',True)]" groups="mail.group_mail_notification_type_inbox"/>
                <separator/>
                <filter string="My Team" name="my_team" domain="['|', ('employee_id.leave_manager_id', '=', uid), ('employee_id.user_id', '=', uid)]" help="Time off of people you are manager of"/>
                <filter string="My Department" name="my_team_leaves" domain="[('employee_id.parent_id.user_id', '=', uid)]" groups="hr_holidays.group_hr_holidays_manager" help="Time Off of Your Team Member"/>
                <separator/>
                <filter name="active_types" string="Active Types" domain="[('holiday_status_id.active', '=', True)]" help="Filters only on allocations that belong to a time off type that is 'active' (active field is True)"/>
                <filter string="Active Employee" name="active_employee" domain="[('active_employee','=',True)]"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                <separator/>
                <filter string="My Allocations" name="my_leaves" domain="[('employee_id.user_id', '=', uid)]"/>
                <separator/>
                <field name="holiday_status_id"/>
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))
                        ]"/>
                <separator/>
                <filter name="approved_state" string="To Approve or Approved Allocations" invisible="1"
                    domain="[('state', 'in', ('confirm', 'validate'))]"/>
                <separator/>
                <group expand="0" string="Group By">
                    <filter name="group_employee" string="Employee" context="{'group_by':'employee_id'}"/>
                    <filter name="group_type" string="Time Off Type" context="{'group_by':'holiday_status_id'}"/>
                    <filter name="group_allocation_type" string="Allocation Type" context="{'group_by':'allocation_type'}"/>
                    <filter name="group_state" string="Status" context="{'group_by': 'state'}"/>
                </group>
                <searchpanel>
                    <field name="state" string="Status"/>
                    <field name="department_id" string="Department" icon="fa-users"/>
                </searchpanel>
            </search>
        </field>
    </record>

    <record id="hr_leave_allocation_view_form" model="ir.ui.view">
        <field name="name">hr.leave.allocation.view.form</field>
        <field name="model">hr.leave.allocation</field>
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <form string="Allocation Request">
                <field name="can_approve" invisible="1"/>
                <field name="holiday_type" invisible="1" readonly="state not in ['confirm']"/>
                <header>
                    <field name="state" widget="statusbar" statusbar_visible="confirm,validate"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box"/>
                    <div id="title" class="oe_title">
                        <h2><field name="name" class="w-100"
                            placeholder="e.g. Time Off type (From validity start to validity end / no limit)"
                            invisible="state != 'confirm'" readonly="True" force_save="True"/>
                        <field name="name_validity" invisible="state == 'confirm'"/></h2>
                    </div>
                    <group>
                        <group>
                            <field name="type_request_unit" invisible="1"/>
                            <field name="has_accrual_plan" invisible="1"/>
                            <!-- Save already_accrued when creating record to avoid double allocation when cron runs -->
                            <field name="already_accrued" invisible="1"/>
                            <field name="holiday_status_id"
                                context="{'employee_id':employee_id, 'default_date_from':current_date, 'request_type':'allocation'}"
                                readonly="state == 'validate'"/>
                            <field name="allocation_type"
                                widget="radio"
                                invisible="1"
                                readonly="not is_officer or state == 'validate'"/>
                            <field name="is_officer" invisible="1"/>
                            <field name="accrual_plan_id"
                                invisible="allocation_type == 'regular'"
                                required="allocation_type == 'accrual'"
                                readonly="not is_officer or state == 'validate'"/>
                            <div class="o_td_label" name="validity_label">
                                <label for="date_from" string="Validity Period"
                                    invisible="allocation_type == 'accrual' or state != 'confirm'"/>
                                <label for="date_from" string="Start Date" invisible="allocation_type == 'regular'"/>
                            </div>
                            <div class="o_row" name="validity">
                                <field name="date_from" widget="date" nolabel="1" readonly="1"
                                    invisible="allocation_type == 'regular' and state != 'confirm'"/>
                                <i class="fa fa-long-arrow-right mx-2" aria-label="Arrow icon" title="Arrow"
                                    invisible="allocation_type == 'accrual' or state != 'confirm'"/>
                                <label class="mx-2" for="date_to" string="Run until"
                                    invisible="allocation_type == 'regular'"/>
                                <field name="date_to" widget="date" nolabel="1"
                                    placeholder="No Limit"  readonly="1"
                                    invisible="allocation_type == 'regular' and state != 'confirm'"/>
                                <div id="no_limit_label" class="oe_read_only"
                                    invisible="not id or date_to or state != 'confirm'">No limit</div>
                            </div>
                            <field name="number_of_days" invisible="1"/>
                            <div class="o_td_label">
                                <label for="number_of_days_display" string="Allocation"
                                    readonly="allocation_type == 'accrual' and state == 'validate'"/>
                            </div>
                            <div name="duration_display">
                                <field name="number_of_days_display" nolabel="1" style="width: 5rem;"
                                    invisible="type_request_unit == 'hour'"
                                    readonly="is_officer != True and state not in ('draft','confirm')"/>
                                <field name="number_of_hours_display" nolabel="1" style="width: 5rem;"
                                    invisible="type_request_unit != 'hour'"
                                    readonly="is_officer != True and state not in ('draft','confirm')"/>
                                <span class="ml8" invisible="type_request_unit == 'hour'">Days</span>
                                <span class="ml8" invisible="type_request_unit != 'hour'">Hours</span>
                            </div>
                        </group>
                        <group name="alloc_right_col">
                            <field name="employee_id" invisible="1" readonly="state in ['refuse', 'validate']"/>
                            <field name="department_id" invisible="1" readonly="state not in ['confirm']"/>
                        </group>
                    </group>
                    <field name="notes" nolabel="1" placeholder="Add a reason..." readonly="state not in ['confirm']"/>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="activity_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>

    <record id="hr_leave_allocation_view_form_manager" model="ir.ui.view">
        <field name="name">hr.leave.allocation.view.form.manager</field>
        <field name="model">hr.leave.allocation</field>
        <field name="inherit_id" ref="hr_holidays.hr_leave_allocation_view_form"/>
        <field name="mode">primary</field>
        <field name="priority">16</field>
        <field name="arch" type="xml">
            <field name="state" position="before">
                <button string="Validate" name="action_validate"
                    invisible="state == 'validate'" type="object" class="oe_highlight"/>
                <button string="Refuse" name="action_refuse" type="object"
                    invisible="not can_approve or state not in ('confirm', 'validate')"/>
            </field>
            <div id="title" position="replace">
                <div class="oe_title">
                    <h2><field name="private_name" placeholder="e.g. Time Off type (From validity start to validity end / no limit)" required="1"/></h2>
                </div>
            </div>
            <field name="employee_id" position="before">
                <field name="holiday_type" string="Mode" groups="hr_holidays.group_hr_holidays_user" context="{'employee_id':employee_id}"  readonly="state not in ['confirm']"/>
            </field>
            <field name="employee_id" position="replace">
                <!-- :TestLeaveRequests.test_allocation_request -->
                <field name="employee_id" invisible="1" readonly="state in ['refuse', 'validate']"/>
                <field name="multi_employee" invisible="1" force_save="1"/>
                <!-- Employee id is only visible if the allocation is created specifically for that user in `_action_validate_create_childs` -->
                <field name="employee_id" groups="hr_holidays.group_hr_holidays_user"
                    invisible="holiday_type != 'employee' or not employee_id or state == 'confirm'"
                    readonly="state in ['refuse', 'validate']"
                    widget="many2one_avatar_employee"/>
                <field name="employee_ids" widget="many2many_avatar_employee"
                    groups="hr_holidays.group_hr_holidays_user"
                    invisible="holiday_type != 'employee' or state != 'confirm' and employee_id"
                    readonly="state in ['refuse', 'validate']"
                    required="holiday_type == 'employee' and state == 'confirm'"/>
            </field>
            <field name="employee_id" position="after">
                <field name="category_id"
                    invisible="holiday_type != 'category'"
                    readonly="state in ['refuse', 'validate']"
                    required="holiday_type == 'category'"/>
            </field>
            <field name="department_id" position="replace">
                <field name="department_id" groups="hr_holidays.group_hr_holidays_user"
                    invisible="holiday_type != 'department'"
                    readonly="state not in ['confirm']"
                    required="holiday_type == 'department'"/>
            </field>
            <field name="department_id" position="after">
                <field name="mode_company_id" string="Company" groups="hr_holidays.group_hr_holidays_user"
                    invisible="holiday_type != 'company'"
                    readonly="state in ['refuse', 'validate']"
                    required="holiday_type == 'company'"/>
            </field>
            <field name="allocation_type" position="attributes">
                <attribute name="invisible">0</attribute>
            </field>
            <label for="date_from" position="replace">
                <label for="date_from" string="Validity Period" invisible="allocation_type == 'accrual'"/>
            </label>
            <field name="date_from" position="replace">
                <field name="date_from" widget="date" nolabel="1" readonly="allocation_type == 'accrual' and state != 'confirm'"/>
            </field>
            <xpath expr="//i[hasclass('fa-long-arrow-right')]" position="replace">
                <i class="fa fa-long-arrow-right mx-2" aria-label="Arrow icon" title="Arrow" invisible="allocation_type == 'accrual'"/>
            </xpath>
            <field name="date_to" position="replace">
                <field name="date_to" widget="date" nolabel="1" placeholder="No Limit"  readonly="allocation_type == 'accrual' and state != 'confirm'"/>
            </field>
            <div id="no_limit_label" position="replace">
                <div id="no_limit_label" class="oe_read_only" invisible="not id or date_to">No limit</div>
            </div>
        </field>
    </record>

    <record id="hr_leave_allocation_view_form_dashboard" model="ir.ui.view">
        <field name="name">hr.leave.view.form.dashboard</field>
        <field name="model">hr.leave.allocation</field>
        <field name="inherit_id" ref="hr_holidays.hr_leave_allocation_view_form"/>
        <field name="mode">primary</field>
        <field name="priority">100</field>
        <field name="arch" type="xml">
            <xpath expr="//header" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <div name="button_box" position="attributes">
                <attribute name="invisible">1</attribute>
            </div>
            <div name="validity_label" position="attributes">
                <attribute name="invisible">1</attribute>
            </div>
            <div name="validity" position="attributes">
                <attribute name="invisible">1</attribute>
            </div>
            <label for="date_from" position="attributes">
                <attribute name="invisible">1</attribute>
            </label>
            <div class="oe_chatter" position="replace"/>
        </field>
    </record>

    <record id="hr_leave_allocation_view_form_manager_dashboard" model="ir.ui.view">
        <field name="name">hr.leave.allocation.view.form.manager.dashboard</field>
        <field name="model">hr.leave.allocation</field>
        <field name="inherit_id" ref="hr_holidays.hr_leave_allocation_view_form_manager"/>
        <field name="mode">primary</field>
        <field name="priority">16</field>
        <field name="arch" type="xml">
            <xpath expr="//header" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <div name="button_box" position="attributes">
                <attribute name="invisible">1</attribute>
            </div>
        </field>
    </record>

    <record id="hr_leave_allocation_view_tree" model="ir.ui.view">
        <field name="name">hr.leave.allocation.view.tree</field>
        <field name="model">hr.leave.allocation</field>
        <field name="priority">16</field>
        <field name="arch" type="xml">
            <tree string="Allocation Requests" sample="1">
                <field name="employee_id" decoration-muted="not active_employee" widget="many2one_avatar_user" readonly="state in ['refuse', 'validate']"/>
                <field name="department_id" optional="hide" readonly="state not in ['confirm']"/>
                <field name="holiday_status_id" class="fw-bold" readonly="state in ['refuse', 'validate', 'validate1']"/>
                <field name="name"/>
                <field name="duration_display" string="Amount"/>
                <field name="date_from" string="Validity Start" optional="hide"/>
                <field name="date_to" string="Validity Stop" optional="hide" readonly="state in ['refuse', 'validate', 'validate1']"/>
                <field name="allocation_type" readonly="state not in ['confirm']"/>
                <field name="message_needaction" column_invisible="True"/>
                <field name="active_employee" column_invisible="True"/>
                <field name="state" widget="badge" decoration-warning="state == 'confirm'" decoration-success="state == 'validate'"/>
                <button string="Validate" name="action_validate" type="object"
                    icon="fa-check"
                    invisible="state != 'confirm'"
                    groups="hr_holidays.group_hr_holidays_user"/>
                <button string="Refuse" name="action_refuse" type="object"
                    icon="fa-times"
                    invisible="state != 'confirm'"
                    groups="hr_holidays.group_hr_holidays_user"/>
                <field name="activity_exception_decoration" widget="activity_exception"/>
            </tree>
        </field>
    </record>

    <record id="hr_leave_allocation_view_tree_my" model="ir.ui.view">
        <field name="name">hr.leave.allocation.view.tree.my</field>
        <field name="model">hr.leave.allocation</field>
        <field name="inherit_id" ref="hr_leave_allocation_view_tree"/>
        <field name="mode">primary</field>
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='employee_id']" position="attributes">
                <attribute name="column_invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='department_id']" position="attributes">
                <attribute name="column_invisible">1</attribute>
            </xpath>
            <xpath expr="//button[@name='action_validate']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//button[@name='action_refuse']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
        </field>
    </record>

    <record id="hr_leave_allocation_view_search_my" model="ir.ui.view">
        <field name="name">hr.leave.allocation.view.search.my</field>
        <field name="model">hr.leave.allocation</field>
        <field name="inherit_id" ref="view_hr_leave_allocation_filter"/>
        <field name="mode">primary</field>
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <xpath expr="//searchpanel" position="replace"/>
            <xpath expr="//filter[@name='message_needaction']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//filter[@name='my_team']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//filter[@name='my_team_leaves']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//filter[@name='my_leaves']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//filter[@name='active_employee']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//filter[@name='active_types']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//filter[@name='inactive']" position="attributes">
                <attribute name="string">Archived Time Off Type</attribute>
            </xpath>
            <xpath expr="//filter[@name='group_employee']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
        </field>
    </record>

    <record id="hr_leave_allocation_view_search_manager" model="ir.ui.view">
        <field name="name">hr.leave.allocation.view.search.my</field>
        <field name="model">hr.leave.allocation</field>
        <field name="inherit_id" ref="view_hr_leave_allocation_filter"/>
        <field name="mode">primary</field>
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='message_needaction']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//filter[@name='my_leaves']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
        </field>
    </record>

    <record id="hr_leave_allocation_view_kanban" model="ir.ui.view">
        <field name="name">hr.leave.allocation.view.kanban</field>
        <field name="model">hr.leave.allocation</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile" create="0" sample="1">
                <field name="employee_id"/>
                <field name="date_from"/>
                <field name="date_to"/>
                <field name="name"/>
                <field name="number_of_days"/>
                <field name="can_approve"/>
                <field name="state"/>
                <field name="holiday_status_id"/>
                <templates>
                    <t t-name="kanban-menu" groups="base.group_user">
                        <a t-if="widget.editable" role="menuitem" type="edit" class="dropdown-item">Edit Allocation</a>
                        <a t-if="widget.deletable" role="menuitem" type="delete" class="dropdown-item">Delete</a>
                    </t>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click container">
                            <div class="row g-0">
                                <div class="col-3">
                                    <img t-att-src="kanban_image('hr.employee.public', 'avatar_128', record.employee_id.raw_value)"
                                        t-att-title="record.employee_id.value"
                                        t-att-alt="record.employee_id.value"
                                        class="o_image_64_cover float-start mr4"/>
                                </div>
                                <div class="col-9">
                                    <span class="badge rounded-pill float-end mt4 mr16"><t t-esc="record.number_of_days.value"/> days</span>
                                    <strong class="o_kanban_record_title"><t t-esc="record.employee_id.value"/></strong>
                                    <div class="text-muted o_kanban_record_subtitle">
                                        <t t-esc="record.holiday_status_id.value"/>
                                    </div>
                                    <div t-if="['validate', 'refuse'].includes(record.state.raw_value)">
                                        <span t-if="record.state.raw_value === 'validate'" class="fa fa-check text-muted" aria-label="validated"/>
                                        <span t-else="" class="fa fa-ban text-muted" aria-label="refused"/>
                                        <t t-set="classname" t-value="{'validate': 'text-bg-success', 'refuse': 'text-bg-danger'}[record.state.raw_value] || 'text-bg-light'"/>
                                        <span t-attf-class="badge rounded-pill {{ classname }}"><t t-esc="record.state.value"/></span>
                                    </div>
                                    <div t-if="record.can_approve.raw_value">
                                        <button t-if="record.state.raw_value === 'confirm'" name="action_validate" type="object" class="btn btn-link btn-sm ps-0">
                                            <i class="fa fa-check"/> Validate
                                        </button>
                                        <button t-if="record.state.raw_value === 'confirm'" name="action_refuse" type="object" class="btn btn-link btn-sm ps-0">
                                            <i class="fa fa-times"/> Refuse
                                        </button>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="hr_leave_allocation_view_activity" model="ir.ui.view">
        <field name="name">hr.leave.allocation.view.activity</field>
        <field name="model">hr.leave.allocation</field>
        <field name="arch" type="xml">
            <activity string="Allocation Requests">
                <field name="employee_id"/>
                <templates>
                    <div t-name="activity-box">
                        <img class="rounded" t-att-src="activity_image('hr.employee', 'avatar_128', record.employee_id.raw_value)" t-att-title="record.employee_id.value" t-att-alt="record.employee_id.value"/>
                        <div class="ms-2">
                            <field name="employee_id" class="o_text_block o_text_bold"/> <span class="text-muted">(<field name="number_of_days"/> days)</span>
                            <field name="holiday_status_id" muted="1" display="full"/>
                        </div>
                    </div>
                </templates>
            </activity>
        </field>
    </record>

    <record id="hr_leave_allocation_action_my" model="ir.actions.act_window">
        <field name="name">My Allocations</field>
        <field name="res_model">hr.leave.allocation</field>
        <field name="view_mode">tree,kanban,form,activity</field>
        <field name="search_view_id" ref="hr_holidays.hr_leave_allocation_view_search_my"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new time off allocation request
            </p><p>
                Time Off Officers allocate time off days to employees (e.g. paid time off).<br/>
                Employees request allocations to Time Off Officers (e.g. recuperation days).
            </p>
        </field>
        <field name="context">{'search_default_year': 1 , 'is_employee_allocation': True}</field>
        <field name="domain">[('employee_id.user_id', '=', uid)]</field>
    </record>
    <record id="hr_leave_allocation_action_my_view_tree" model="ir.actions.act_window.view">
        <field name="sequence">1</field>
        <field name="view_mode">tree</field>
        <field name="act_window_id" ref="hr_leave_allocation_action_my"/>
        <field name="view_id" ref="hr_leave_allocation_view_tree_my"/>
    </record>
    <record id="hr_leave_allocation_action_my_view_form" model="ir.actions.act_window.view">
        <field name="sequence">2</field>
        <field name="view_mode">form</field>
        <field name="act_window_id" ref="hr_leave_allocation_action_my"/>
        <field name="view_id" ref="hr_leave_allocation_view_form"/>
    </record>

    <record id="hr_leave_allocation_action_all" model="ir.actions.act_window">
        <field name="name">All Allocations</field>
        <field name="res_model">hr.leave.allocation</field>
        <field name="view_mode">tree,kanban,form,activity</field>
        <field name="context">{}</field>
        <field name="domain">[]</field>
        <field name="search_view_id" ref="hr_holidays.hr_leave_allocation_view_search_manager"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new time off allocation request
            </p><p>
                Time Off Officers allocate time off days to employees (e.g. paid time off).<br/>
                Employees request allocations to Time Off Officers (e.g. recuperation days).
            </p>
        </field>
    </record>

    <record id="hr_leave_allocation_action_approve_department" model="ir.actions.act_window">
        <field name="name">Allocations</field>
        <field name="res_model">hr.leave.allocation</field>
        <field name="view_mode">tree,form,kanban,activity</field>
        <field name="context">{'search_default_my_team': 1,'search_default_approve': 2, 'search_default_active_employee': 3}</field>
        <field name="domain">[]</field>
        <field name="search_view_id" ref="hr_holidays.hr_leave_allocation_view_search_manager"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new time off allocation
            </p><p>
                Time Off Officers allocate time off days to employees (e.g. paid time off).<br/>
                Employees request allocations to Time Off Officers (e.g. recuperation days).
            </p>
        </field>
    </record>

    <record id="ir_actions_server_approve_allocations" model="ir.actions.server">
        <field name="name">Approve Allocations</field>
        <field name="model_id" ref="hr_holidays.model_hr_leave_allocation"/>
        <field name="binding_model_id" ref="hr_holidays.model_hr_leave_allocation"/>
        <field name="binding_view_types">list</field>
        <field name="state">code</field>
        <field name="code">
            if records:
                records.action_validate()
        </field>
    </record>
</odoo>

```

## File: views\hr_leave_mandatory_day_views.xml

```xml
<?xml version='1.0' encoding='UTF-8' ?>
<odoo>
    <record id="hr_leave_mandatory_day_view_form" model="ir.ui.view">
        <field name="model">hr.leave.mandatory.day</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <group>
                            <field name="name"/>
                            <field name="start_date" string="Dates" widget="daterange" options="{'end_date_field': 'end_date'}" />
                            <field name="end_date" invisible="1" />
                        </group>
                        <group>
                            <field name="color" widget="color_picker"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="hr_leave_mandatory_day_view_list" model="ir.ui.view">
        <field name="model">hr.leave.mandatory.day</field>
        <field name="arch" type="xml">
            <tree editable="bottom">
                <field name="company_id" column_invisible="True"/>
                <field name="name"/>
                <field name="company_id" groups="base.group_multi_company"/>
                <field name="department_ids" widget="many2many_tags" optional="hide"/>
                <field name="start_date"/>
                <field name="end_date"/>
                <field name="color" widget="color_picker"/>
            </tree>
        </field>
    </record>

    <record id="hr_leave_mandatory_day_view_search" model="ir.ui.view">
        <field name="model">hr.leave.mandatory.day</field>
        <field name="arch" type="xml">
            <search string="Mandatory Day">
                <field name="name"/>
                <field name="start_date"/>
                <field name="end_date"/>
                <field name="company_id" groups="base.group_multi_company"/>
                <separator />
                <filter name="filter_date" date="start_date" default_period="this_year" string="Period"/>
                <group expand="0" string="Group By">
                    <filter name="department" string="Department" context="{'group_by': 'department_ids'}"/>
                    <filter name="company" string="Company" context="{'group_by':'company_id'}" groups="base.group_multi_company"/>
                </group>
            </search>
        </field>
    </record>

    <record id="hr_leave_mandatory_day_action" model="ir.actions.act_window">
        <field name="name">Mandatory Days</field>
        <field name="res_model">hr.leave.mandatory.day</field>
        <field name="view_mode">tree,form</field>
        <field name="search_view_id" ref="hr_leave_mandatory_day_view_search"/>
        <field name="context">{'search_default_filter_date': True}</field>
    </record>
</odoo>

```

## File: views\hr_leave_type_views.xml

```xml
<?xml version='1.0' encoding='UTF-8' ?>
<odoo>

    <record id="view_holidays_status_filter" model="ir.ui.view">
        <field name="name">hr.leave.type.filter</field>
        <field name="model">hr.leave.type</field>
        <field name="arch" type="xml">
            <search string="Search Time Off Type">
                <field name="name" string="Time Off Types"/>
                <field name="create_calendar_meeting"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
            </search>
        </field>
    </record>

    <record id="edit_holiday_status_form" model="ir.ui.view">
        <field name="name">hr.leave.type.form</field>
        <field name="model">hr.leave.type</field>
        <field name="arch" type="xml">
            <form string="Time Off Type">
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button class="oe_stat_button"
                                type="object"
                                name="action_see_days_allocated"
                                icon="fa-calendar"
                                invisible="requires_allocation == 'no' or not id"
                                help="Count of allocations for this time off type (approved or waiting for approbation) with a validity period starting this year.">
                            <div class="o_stat_info">
                                <field name="allocation_count" class="o_stat_value"/>
                                <span class="o_stat_text">Allocations</span>
                            </div>
                        </button>
                        <button class="oe_stat_button"
                                type="object"
                                name="action_see_group_leaves"
                                icon="fa-calendar"
                                invisible="not id"
                                help="Count of time off requests for this time off type (approved or waiting for approbation) with a start date in the current year.">
                            <div class="o_stat_info">
                                <field name="group_days_leave" class="o_stat_value"/>
                                <span class="o_stat_text">Time Off</span>
                            </div>
                        </button>
                        <button class="oe_stat_button"
                                type="object"
                                name="action_see_accrual_plans"
                                icon="fa-calendar"
                                invisible="not id or accrual_count == 0"
                                help="Count of plans linked to this time off type.">
                            <div class="o_stat_info">
                                <field name="accrual_count" class="o_stat_value"/>
                                <span class="o_stat_text">Accruals</span>
                            </div>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <div class="oe_title">
                        <h1><field name="name"/></h1>
                    </div>
                    <group>
                        <group name="leave_validation" id="time_off_requests" string="Time Off Requests">
                            <field name="active" invisible="1"/>
                            <field name="leave_validation_type" string="Approval" widget="radio"/>
                        </group>
                        <group name="allocation_validation" id="allocation_requests" string="Allocation Requests">
                            <field name="requires_allocation" widget="radio" options="{'horizontal':true}"/>
                            <field name="employee_requests" widget="radio" invisible="requires_allocation == 'no'"/>
                            <field name="allocation_validation_type" string="Approval" widget="radio" invisible="requires_allocation == 'no'"/>
                        </group>
                    </group>
                    <group>
                        <group name="configuration" id="configuration" string="Configuration">
                            <field name="responsible_ids"
                                widget="many2many_tags"
                                placeholder="Nobody will be notified"
                                invisible="leave_validation_type in ['no_validation', 'manager'] and (requires_allocation == 'no' or allocation_validation_type != 'officer')"/>
                            <field name="request_unit"/>
                            <field name="support_document" string="Allow To Attach Supporting Document" />
                            <field name="time_type" required="1"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                        <group name="negative_cap" id="negative_cap"
                            string="Negative Cap"
                            invisible="requires_allocation == 'no'">
                            <field name="allows_negative"/>
                            <field name="max_allowed_negative"
                                class="o_hr_narrow_field"
                                invisible="not allows_negative"
                                required="requires_allocation == 'yes' and allows_negative"/>
                        </group>
                    </group>
                    <group name="visual" id="visual" string="Display Option" class="mw-100 col-lg-12">
                        <field name="color" widget="color_picker" />
                        <field class="o_time_off_icon_types d-flex flex-wrap" name="icon_id" widget="hr_holidays_radio_image" options="{'horizontal': true}"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="hr_holiday_status_view_kanban" model="ir.ui.view">
        <field name="name">hr.leave.type.kanban</field>
        <field name="model">hr.leave.type</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile">
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div>
                                <strong><field name="name"/></strong>
                            </div>
                            <div>
                                <span>Max Time Off: <field name="max_leaves"/></span>
                                <span class="float-end">Time Off Taken: <field name="leaves_taken"/></span>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="view_holiday_status_normal_tree" model="ir.ui.view">
        <field name="name">hr.leave.type.normal.tree</field>
        <field name="model">hr.leave.type</field>
        <field name="arch" type="xml">
            <tree string="Time Off Type" multi_edit="1">
                <field name="sequence" widget="handle"/>
                <field name="display_name"/>
                <field name="allocation_validation_type"/>
                <field name="employee_requests" optional="hide"/>
                <field name="requires_allocation" optional="hide"/>
                <field name="leave_validation_type" optional="hide"/>
                <field name="responsible_ids" widget="many2many_tags" invisible="leave_validation_type in ['no_validation', 'manager'] and (requires_allocation == 'no' or allocation_validation_type != 'officer')" optional="hide"/>
                <field name="company_id" groups="base.group_multi_company" optional="hide"/>
            </tree>
        </field>
    </record>

    <record id="open_view_holiday_status" model="ir.actions.act_window">
        <field name="name">Time Off Types</field>
        <field name="res_model">hr.leave.type</field>
        <field name="view_mode">tree,kanban,form</field>
    </record>

</odoo>

```

## File: views\hr_leave_views.xml

```xml
<?xml version='1.0' encoding='UTF-8' ?>
<odoo>

    <record model="ir.actions.server" id="action_manager_approval">
        <field name="name">Manager Approval</field>
        <field name="model_id" ref="model_hr_leave"/>
        <field name="binding_model_id" ref="model_hr_leave" />
        <field name="state">code</field>
        <field name="code">
            if records:
                records.action_approve()
        </field>
    </record>
    <record model="ir.actions.server" id="action_hr_approval">
        <field name="name">HR Approval</field>
        <field name="model_id" ref="model_hr_leave"/>
        <field name="binding_model_id" ref="model_hr_leave" />
        <field name="state">code</field>
        <field name="code">
            if records:
                records.action_validate()
        </field>
    </record>

    <record id="view_evaluation_report_graph" model="ir.ui.view">
        <field name="name">hr.holidays.graph</field>
        <field name="model">hr.leave</field>
        <field name="arch" type="xml">
            <graph string="Appraisal Analysis" sample="1">
                <field name="employee_id"/>
                <field name="holiday_status_id"/>
                <field name="date_from"/>
                <field name="number_of_days" type="measure"/>
            </graph>
         </field>
    </record>

    <record id="view_hr_holidays_filter" model="ir.ui.view">
        <field name="name">hr.holidays.filter</field>
        <field name="model">hr.leave</field>
        <field name="arch" type="xml">
            <search string="Search Time Off">
                <field name="employee_id"/>
                <field name="holiday_status_id"/>
                <field name="name"/>
                <filter domain="[
                        ('state','in',['confirm']),
                        '|',
                        ('employee_id.user_id', '!=', uid),
                        '&amp;',
                        ('employee_id.user_id', '=', uid),
                        ('employee_id.leave_manager_id', '=', uid)]"
                    string="Waiting For Me"
                    name="waiting_for_me"
                    groups="hr_holidays.group_hr_holidays_responsible,!hr_holidays.group_hr_holidays_user"/>
                <filter domain="[
                        ('state','in',['confirm','validate1']),
                        '|',
                            ('employee_id.user_id', '!=', uid),
                            '|',
                                '&amp;',
                                    ('state','=','confirm'),
                                    ('holiday_status_id.leave_validation_type','=','hr'),
                                ('state','=','validate1')]"
                    string="Waiting For Me"
                    name="waiting_for_me_manager"
                    groups="hr_holidays.group_hr_holidays_user"/>
                <separator/>
                <filter domain="[('state','=','confirm')]" string="First Approval" name="approve"/>
                <filter domain="[('state', '=', 'validate1')]" string="Second Approval" name="second_approval"/>
                <filter string="Approved" domain="[('state', '=', 'validate')]" name="validated"/>
                <separator/>
                <filter string="My Time Off" name="my_leaves" domain="[('employee_id.user_id', '=', uid)]"/>
                <filter string="My Team" name="my_team" domain="['|', ('employee_id.leave_manager_id', '=', uid), ('employee_id.user_id', '=', uid)]" help="Time off of people you are manager of"/>
                <filter string="My Department" name="department"
                        domain="[('employee_id.member_of_department', '=', True)]"
                        help="My Department"/>
                <separator/>
                <filter string="Active Employee" name="active_employee" domain="[('active_employee','=',True)]"/>
                <separator/>
                <filter name="filter_date_from" date="date_from" default_period="this_year" string="Current Year"/>
                <separator/>
                <filter name="active_time_off" string="Active"
                    domain="[('holiday_status_id.active', '=', True)]" help="Active Time Off"/>
                <filter name="archive" string="Archived"
                    domain="[('holiday_status_id.active', '=', False)]" help="Archived Time Off"/>
                <separator/>
                <filter name="archived_leaves" string="Cancelled" domain="[('active', '=', False)]" />
                <filter invisible="1" string="Late Activities" name="activities_overdue"
                    domain="[('my_activity_date_deadline', '&lt;', context_today().strftime('%Y-%m-%d'))]"
                    help="Show all records which has next action date is before today"/>
                <filter invisible="1" string="Today Activities" name="activities_today"
                    domain="[('my_activity_date_deadline', '=', context_today().strftime('%Y-%m-%d'))]"/>
                <filter invisible="1" string="Future Activities" name="activities_upcoming_all"
                        domain="[('my_activity_date_deadline', '&gt;', context_today().strftime('%Y-%m-%d'))
                        ]"/>
                <group expand="0" string="Group By">
                    <filter name="group_employee" string="Employee" context="{'group_by':'employee_id'}"/>
                    <filter name="group_type" string="Type" context="{'group_by':'holiday_status_id'}"/>
                    <filter name="group_state" string="Status" context="{'group_by': 'state'}"/>
                    <filter name="group_company" string="Company" context="{'group_by':'employee_company_id'}" groups="base.group_multi_company"/>
                    <separator/>
                    <filter name="group_date_from" string="Start Date" context="{'group_by':'date_from'}"/>
                </group>
                <searchpanel>
                    <field name="state" string="Status"/>
                    <field name="department_id" string="Department" icon="fa-users"/>
                </searchpanel>
            </search>
        </field>
    </record>

    <record id="hr_leave_view_kanban" model="ir.ui.view">
        <field name="name">hr.leave.view.kanban</field>
        <field name="model">hr.leave</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile" create="0" sample="1">
                <field name="employee_id"/>
                <field name="date_from"/>
                <field name="date_to"/>
                <field name="name"/>
                <field name="number_of_days"/>
                <field name="can_approve"/>
                <field name="holiday_status_id"/>
                <field name="state"/>
                <field name="supported_attachment_ids_count"/>
                <templates>
                    <t t-name="kanban-menu" groups="base.group_user">
                        <a t-if="widget.editable" role="menuitem" type="edit" class="dropdown-item">Edit Time Off</a>
                        <a t-if="widget.deletable" role="menuitem" type="delete" class="dropdown-item">Delete</a>
                    </t>
                    <t t-name="kanban-box">
                        <div class="d-flex flex-column justify-content-between oe_kanban_global_click container">
                            <div class="row g-0">
                                <div class="o_leave_kanban_info col-12">
                                    <span class="badge rounded-pill float-end mt4 mr16"><t t-esc="record.number_of_days.value"/> days</span>
                                    <strong class="o_kanban_record_title"><t t-out="record.employee_id.value"/></strong>
                                    <div class="text-muted o_kanban_record_subtitle">
                                        <t t-esc="record.holiday_status_id.value"/>
                                    </div>
                                    <div>
                                        <span class="text-muted">from </span>
                                        <field name="date_from" widget="date" readonly="state in ['cancel', 'refuse', 'validate', 'validate1']"/>
                                        <span class="text-muted"> to </span>
                                        <field name="date_to" widget="date" readonly="state in ['cancel', 'refuse', 'validate', 'validate1']"/>
                                    </div>
                                    <div class="o_leave_kanban_name p-2">
                                        <field name="name" nolabel="1"/>
                                    </div>
                                </div>
                            </div>
                            <div class="d-flex justify-content-end">
                                <div class="me-2 d-flex align-items-center" t-if="!['draft'].includes(record.state.raw_value)">
                                    <span t-if="record.state.raw_value === 'validate'" class="fa fa-check text-muted me-1" aria-label="validated"/>
                                    <span t-if="record.state.raw_value === 'refuse'" class="fa fa-ban text-muted me-1" aria-label="refused"/>
                                    <span t-if="['confirm', 'validate1'].includes(record.state.raw_value)" class="me-1" aria-label="to refuse"/>
                                    <t t-set="classname"
                                        t-value="{'validate': 'text-bg-success', 'refuse': 'text-bg-danger', 'confirm': 'text-bg-warning', 'validate1': 'text-bg-warning'}[record.state.raw_value] || 'text-bg-light'"/>
                                    <span t-attf-class="badge rounded-pill {{ classname }}">
                                        <t t-out="record.state.value"/>
                                    </span>
                                </div>
                                <div class="me-2 align-items-center" t-if="['confirm', 'validate1'].includes(record.state.raw_value)">
                                    <button t-if="record.state.raw_value === 'confirm'" name="action_approve" type="object" class="btn btn-link btn-sm ps-0"
                                        groups="hr_holidays.group_hr_holidays_user">
                                        <i class="fa fa-thumbs-up"/> Approve
                                    </button>
                                    <button t-if="record.state.raw_value === 'validate1'" name="action_validate" type="object" class="btn btn-link btn-sm ps-0"
                                        groups="hr_holidays.group_hr_holidays_manager">
                                        <i class="fa fa-check"/> Validate
                                    </button>
                                    <button t-if="['confirm', 'validate1'].includes(record.state.raw_value)" name="action_refuse" type="object" class="btn btn-link btn-sm ps-0"
                                        groups="hr_holidays.group_hr_holidays_user">
                                        <i class="fa fa-times"/> Refuse
                                    </button>
                                </div>
                                <div class="text-end">
                                    <button t-if="record.supported_attachment_ids_count.raw_value > 0" name="action_documents" type="object" class="btn btn-link btn-sm ps-0">
                                        <i class="fa fa-paperclip"> <field name="supported_attachment_ids_count" nolabel="1"/></i>
                                    </button>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="hr_leave_view_kanban_approve_department" model="ir.ui.view">
        <field name="name">hr.leave.view.kanban.approve.department</field>
        <field name="inherit_id" ref="hr_leave_view_kanban"/>
        <field name="mode">primary</field>
        <field name="model">hr.leave</field>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('o_leave_kanban_info')]" position='before'>
                <div class="col-3">
                    <img t-att-src="kanban_image('hr.employee.public', 'avatar_128', record.employee_id.raw_value)"
                        t-att-title="record.employee_id.value"
                        t-att-alt="record.employee_id.value"
                        class="o_image_64_cover float-start mr4"/>
                </div>
            </xpath>
            <xpath expr="//div[hasclass('o_leave_kanban_info')]" position="attributes">
                <attribute name="class">o_leave_kanban_info col-9</attribute>
            </xpath>
            <xpath expr="//div[hasclass('o_leave_kanban_name')]" position="replace"/>
        </field>
    </record>

    <record id="hr_leave_view_activity" model="ir.ui.view">
        <field name="name">hr.leave.view.activity</field>
        <field name="model">hr.leave</field>
        <field name="arch" type="xml">
            <activity string="Time Off Request">
                <field name="employee_id"/>
                <templates>
                    <div t-name="activity-box">
                        <img class="rounded" t-att-src="activity_image('hr.employee', 'avatar_128', record.employee_id.raw_value)" t-att-title="record.employee_id.value" t-att-alt="record.employee_id.value"/>
                        <div class="ms-2 flex-grow-1">
                            <div class="d-flex justify-content-between">
                                <field name="name" class="o_text_block o_text_bold"/>
                                <div class="text-muted text-nowrap">(<field name="number_of_days"/> days)</div>
                            </div>
                            <div class="text-muted">
                                <field name="holiday_status_id" display="full"/>
                                <field name="date_from" widget="date"/> <i class="fa fa-long-arrow-right" title="to" /> <field name="date_to" widget="date"/>
                            </div>
                        </div>
                    </div>
                </templates>
            </activity>
        </field>
    </record>

    <record id="hr_leave_view_form" model="ir.ui.view">
        <field name="name">hr.leave.view.form</field>
        <field name="model">hr.leave</field>
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <form string="Time Off Request" class="o_hr_leave_form">
            <field name="active" invisible="1"/>
            <field name="can_reset" invisible="1"/>
            <field name="can_approve" invisible="1"/>
            <field name="can_cancel" invisible="1"/>
            <field name="has_mandatory_day" invisible="1"/>
            <header>
                <button string="Confirm" name="action_confirm" type="object" class="oe_highlight" invisible="id == False or state != 'draft' or not active"/>
                <button string="Approve" name="action_approve" type="object" class="oe_highlight" invisible="not active or not can_approve or state != 'confirm'"/>
                <button string="Validate" name="action_validate" invisible="state != 'validate1'" type="object" groups="hr_holidays.group_hr_holidays_user" class="oe_highlight"/>
                <button string="Refuse" name="action_refuse" type="object" invisible="not active or not can_approve or state not in ('confirm', 'validate1', 'validate')"/>
                <button string="Cancel" name="action_cancel" type="object" invisible="not active or not can_cancel" />
                <button string="Mark as Draft" name="action_draft" type="object"
                        invisible="not can_reset or state not in ['confirm', 'refuse']"/>
                <field name="state" widget="statusbar" statusbar_visible="confirm,validate" invisible="not active"/>
            </header>
            <sheet>
                <widget name="web_ribbon" title="Canceled" bg_color="text-bg-danger" invisible="active"/>
                <div class="alert alert-info" role="alert" invisible="not request_unit_hours or not tz_mismatch or holiday_type == 'category'">
                    <span invisible="holiday_type != 'employee'">
                        The employee has a different timezone than yours! Here dates and times are displayed in the employee's timezone
                    </span>
                    <span invisible="holiday_type != 'department'">
                        The department's company has a different timezone than yours! Here dates and times are displayed in the company's timezone
                    </span>
                    <span invisible="holiday_type != 'company'">
                        The company has a different timezone than yours! Here dates and times are displayed in the company's timezone
                    </span>
                    (<field name="tz"/>).
                </div>
                <field name="tz_mismatch" invisible="1"/>
                <field name="holiday_type" invisible="1" readonly="state not in ['confirm', 'draft']"/>
                <field name="leave_type_request_unit" invisible="1"/>
                <div class="o_hr_leave_content row">
                    <div class="o_hr_leave_column col_left col-md-6 col-12">
                        <div name="title" class="o_hr_leave_title" invisible="1">
                            <field name="employee_id" readonly="1" force_save="1" invisible="1"/>
                            <field name="employee_ids" invisible="1" readonly="state in ['cancel', 'refuse', 'validate', 'validate1']"/>
                            <field name="display_name" invisible="not holiday_status_id"/>
                        </div>
                        <field name="leave_type_increases_duration" invisible="True"/>
                        <group name="col_left">
                            <field name="employee_company_id" invisible="1"/>
                            <field name="holiday_status_id" force_save="1"
                                domain="[
                                    '|',
                                        ('requires_allocation', '=', 'no'),
                                        '&amp;',
                                            ('has_valid_allocation', '=', True),
                                            '|',
                                                ('allows_negative', '=', True),
                                                '&amp;',
                                                    ('virtual_remaining_leaves', '&gt;', 0),
                                                    ('allows_negative', '=', False),
                                ]"
                                context="{'employee_id': employee_id, 'default_date_from': date_from, 'default_date_to': date_to}"
                                options="{'no_create': True, 'no_open': True, 'request_type': 'leave'}"
                                readonly="state in ['cancel', 'refuse', 'validate', 'validate1']"/>
                            <field name="date_from" invisible="1"/>
                            <field name="date_to" invisible="1"/>
                            <field name="request_date_from" invisible="1"/>
                            <field name="request_date_to" invisible="1"/>
                            <!-- half day or custom hours: only show one date -->
                            <label for="request_date_from" invisible="not request_unit_half and not request_unit_hours" string="Date" />
                            <label for="request_date_from" invisible="request_unit_half or request_unit_hours" string="Dates" />
                            <div class="o_row" invisible="not request_unit_half and not request_unit_hours">
                                <field name="request_date_from" class="oe_inline" string="Date" readonly="state not in ('draft', 'confirm')" />
                                <field name="request_date_from_period" invisible="not request_unit_half" required="request_unit_half" readonly="state not in ('draft', 'confirm')"/>
                            </div>
                            <!-- full days: show date start/end with daterange -->
                            <div class="o_row" invisible="request_unit_half or request_unit_hours">
                                <field
                                    name="request_date_from"
                                    widget="daterange"
                                    readonly="state not in ('draft', 'confirm')"
                                    required="not date_from or not date_to"
                                    options="{'end_date_field': 'request_date_to'}"/>
                                <field name="request_date_to" invisible="1" />
                            </div>
                            <label for="request_unit_half" string="" invisible="leave_type_request_unit == 'day'"/>
                            <div class="o_row o_row_readonly oe_edit_only" style="margin-left: -2px;" invisible="leave_type_request_unit == 'day'">
                                <field name="request_unit_half" class="oe_inline" invisible="leave_type_request_unit == 'day'" readonly="state not in ('draft', 'confirm')" />
                                <label for="request_unit_half" invisible="leave_type_request_unit == 'day'" />
                                <field name="request_unit_hours" invisible="leave_type_request_unit != 'hour'" readonly="state not in ('draft', 'confirm')" class="ms-5" />
                                <label for="request_unit_hours" invisible="leave_type_request_unit != 'hour'" />
                            </div>
                            <label for="request_hour_from" string="" invisible="not request_unit_hours"/>
                            <div class="o_row o_row_readonly" invisible="not request_unit_hours">
                                <label for="request_hour_from" string="From" />
                                <field name="request_hour_from" invisible="not request_unit_hours" readonly="state == 'validate'" required="request_unit_hours" />
                                <label for="request_hour_to" string="To" />
                                <field name="request_hour_to" invisible="not request_unit_hours" readonly="state == 'validate'" required="request_unit_hours" />
                            </div>

                            <label for="number_of_days_display" string="Duration" invisible="request_unit_half and leave_type_request_unit != 'hour'" />
                            <div name="duration_display" invisible="request_unit_half and leave_type_request_unit != 'hour'">
                                <div invisible="request_unit_half or request_unit_hours">
                                    <field name="number_of_days_display" class="oe_inline" />
                                    <span class="ms-3">Days</span>
                                </div>
                                <div invisible="leave_type_request_unit != 'hour'" class="o_row">
                                    <field name="number_of_hours_text" />
                                </div>
                            </div>
                            <field name="name" readonly="state not in ('draft', 'confirm')" widget="text" placeholder="Add a description..." />
                            <field name="user_id" invisible="1" />
                            <field name="leave_type_support_document" invisible="1" />
                            <label for="supported_attachment_ids" string="Supporting Document" invisible="not leave_type_support_document or state not in ('draft', 'confirm', 'validate1')" />
                            <field name="supported_attachment_ids" widget="many2many_binary" nolabel="1" invisible="not leave_type_support_document or state not in ('draft', 'confirm', 'validate1')" />
                        </group>
                    </div>
                </div>
                <div name="duration_warning" invisible="not leave_type_increases_duration" class="alert alert-warning mb-0" role="alert">
                    <span >You can only take this time off in whole days, so if your schedule has half days, it won't be used efficiently.</span>
                </div>
            </sheet>
            <div class="o_attachment_preview"/>
            <div class="oe_chatter">
                <field name="message_follower_ids"/>
                <field name="activity_ids"/>
                <field name="message_ids" options="{'post_refresh': 'always'}"/>
            </div>
            </form>
        </field>
    </record>

    <record id="hr_leave_view_form_dashboard" model="ir.ui.view">
        <field name="name">hr.leave.view.form.dashboard</field>
        <field name="model">hr.leave</field>
        <field name="inherit_id" ref="hr_holidays.hr_leave_view_form"/>
        <field name="mode">primary</field>
        <field name="priority">100</field>
        <field name="arch" type="xml">
            <xpath expr="//header" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
        </field>
    </record>

    <record id="hr_leave_view_form_dashboard_new_time_off" model="ir.ui.view">
        <field name="name">hr.leave.view.form.dashboard.new.time.off</field>
        <field name="model">hr.leave</field>
        <field name="inherit_id" ref="hr_holidays.hr_leave_view_form_dashboard"/>
        <field name="mode">primary</field>
        <field name="priority">17</field>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='col_left']" position="attributes">
                <attribute name="colspan">5</attribute>
            </xpath>
            <xpath expr="//div[hasclass('o_hr_leave_content')]" position="attributes">
                <attribute name="class" remove="row my-n4" separator=" "/>
            </xpath>
            <xpath expr="//div[hasclass('o_hr_leave_column')]" position="attributes">
                <attribute name="class" remove="col_left col-md-6" separator=" "/>
            </xpath>
            <xpath expr="//sheet/widget[@name='web_ribbon']" position="after">
                <widget name="web_ribbon" title="Refused" bg_color="bg-danger" invisible="state != 'refuse' or not id"/>
                <widget name="web_ribbon" title="Approved" bg_color="bg-success" invisible="state != 'validate' or not id"/>
            </xpath>
        </field>
    </record>

    <record id="hr_leave_view_dashboard" model="ir.ui.view">
        <field name="name">hr.leave.view.dashboard</field>
        <field name="model">hr.leave</field>
        <field name="arch" type="xml">
            <calendar js_class="time_off_calendar_dashboard"
                    string="Time Off Request"
                    form_view_id="%(hr_holidays.hr_leave_view_form_dashboard_new_time_off)d"
                    event_open_popup="true"
                    date_start="date_from"
                    date_stop="date_to"
                    quick_create="0"
                    show_unusual_days="True"
                    color="color"
                    hide_time="True"
                    mode="year">
                <field name="display_name"/>
                <field name="holiday_status_id" filters="1" invisible="1" color="color"/>
                <field name="state" invisible="1"/>
                <field name="is_hatched" invisible="1" />
                <field name="is_striked" invisible="1"/>
                <field name="can_cancel" invisible="1"/>
            </calendar>
        </field>
    </record>

    <record id="hr_leave_employee_view_dashboard" model="ir.ui.view">
        <field name="name">hr.leave.view.dashboard</field>
        <field name="model">hr.leave</field>
        <field name="arch" type="xml">
            <calendar string="Time Off Request"
                    js_class="time_off_calendar_dashboard"
                    form_view_id="%(hr_holidays.hr_leave_view_form_dashboard_new_time_off)d"
                    event_open_popup="true"
                    date_start="date_from"
                    date_stop="date_to"
                    mode="year"
                    quick_create="0"
                    show_unusual_days="True"
                    color="color"
                    hide_time="True">
                <field name="display_name"/>
                <field name="holiday_status_id" filters="1" invisible="1" color="color"/>
                <field name="state" invisible="1"/>
                <field name="is_hatched" invisible="1" />
                <field name="is_striked" invisible="1"/>
                <field name="can_cancel" invisible="1"/>
            </calendar>
        </field>
    </record>

    <record id="hr_leave_view_form_manager" model="ir.ui.view">
        <field name="name">hr.leave.view.form.manager</field>
        <field name="model">hr.leave</field>
        <field name="inherit_id" ref="hr_leave_view_form"/>
        <field name="mode">primary</field>
        <field name="priority">16</field>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='title']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//field[@name='holiday_status_id']" position="before">
                <field name="is_user_only_responsible" invisible="1"/>
                <field name="holiday_type" string="Mode" groups="hr_holidays.group_hr_holidays_user" readonly="state not in ['confirm', 'draft']"/>
                <field name="mode_company_id" string="Company" groups="hr_holidays.group_hr_holidays_user" invisible="holiday_type != 'company'" readonly="state not in ['confirm', 'draft']" required="holiday_type == 'company'"/>
                <field name="category_id" groups="hr_holidays.group_hr_holidays_user" invisible="holiday_type != 'category'" readonly="state not in ['confirm', 'draft']" required="holiday_type == 'category'"/>
                <field name="department_id" groups="hr_holidays.group_hr_holidays_user" invisible="holiday_type != 'department'" readonly="state in ['cancel', 'refuse', 'validate', 'validate1']" required="holiday_type == 'department'"/>
                <field name="multi_employee" invisible="1" force_save="1"/>
                <field name="employee_id" groups="hr_holidays.group_hr_holidays_user" invisible="holiday_type != 'employee' or state != 'validate' or not employee_id" readonly="state in ['cancel', 'refuse', 'validate', 'validate1']" widget="many2one_avatar_user"/>
                <field name="employee_ids" groups="hr_holidays.group_hr_holidays_user" invisible="holiday_type != 'employee' or (state == 'validate' and employee_id)" readonly="state in ['cancel', 'refuse', 'validate', 'validate1']" required="holiday_type == 'employee' and state in ('draft', 'cancel', 'refuse')" widget="many2many_tags_avatar"/>
            </xpath>
            <field name="name" position="replace"/>
            <field name="user_id" position="before">
                <field name="name" widget="text"/>
            </field>
            <xpath expr="//div[hasclass('col_left')]" position="after">
                <div class="o_hr_leave_column col_right col-md-6 col-12">
                    <widget name="hr_leave_stats"/>
                </div>
            </xpath>
        </field>
    </record>

    <record id="hr_leave_view_calendar" model="ir.ui.view">
        <field name="name">hr.leave.view.calendar</field>
        <field name="model">hr.leave</field>
        <field name="arch" type="xml">
            <calendar js_class="time_off_calendar"
                    string="Time Off Request"
                    form_view_id="%(hr_holidays.hr_leave_view_form_dashboard_new_time_off)d"
                    event_open_popup="true"
                    date_start="date_from"
                    date_stop="date_to"
                    mode="month"
                    show_unusual_days="True"
                    quick_create="0"
                    color="color">
                <field name="display_name"/>
                <field name="holiday_status_id" color="color" filters="1" invisible="1"/>
                <field name="employee_id" filters="1" invisible="1"/>
                <field name="is_hatched" invisible="1" />
                <field name="is_striked" invisible="1"/>
            </calendar>
        </field>
    </record>

    <record id="hr_leave_view_tree" model="ir.ui.view">
        <field name="name">hr.holidays.view.tree</field>
        <field name="model">hr.leave</field>
        <field name="arch" type="xml">
            <tree string="Time Off Requests" sample="1">
                <field name="employee_id" column_invisible="True" readonly="state in ['cancel', 'refuse', 'validate', 'validate1']" />
                <field name="all_employee_ids" widget="many2many_tags_avatar" decoration-muted="not active_employee" string="Employees" />
                <field name="department_id" optional="hidden" readonly="state in ['cancel', 'refuse', 'validate', 'validate1']"/>
                <field name="holiday_type" string="Mode" groups="base.group_no_one" readonly="state not in ['confirm', 'draft']"/>
                <field name="holiday_status_id" class="fw-bold" readonly="state in ['cancel', 'refuse', 'validate', 'validate1']"/>
                <field name="name"/>
                <field name="date_from" readonly="state in ['cancel', 'refuse', 'validate', 'validate1']"/>
                <field name="date_to" readonly="state in ['cancel', 'refuse', 'validate', 'validate1']"/>
                <field name="duration_display" string="Duration"/>
                <field name="state" widget="badge" decoration-info="state == 'draft'" decoration-warning="state in ('confirm','validate1')" decoration-success="state == 'validate'"/>
                <field name="active_employee" column_invisible="True"/>
                <field name="category_id" column_invisible="True" readonly="state not in ['confirm', 'draft']"/>
                <field name="user_id" column_invisible="True"/>
                <field name="message_needaction" column_invisible="True"/>
                <button string="Approve" name="action_approve" type="object"
                    icon="fa-thumbs-up"
                    invisible="state != 'confirm'"
                    groups="hr_holidays.group_hr_holidays_responsible"/>
                <button string="Validate" name="action_validate" type="object"
                    icon="fa-check"
                    invisible="state != 'validate1'"
                    groups="hr_holidays.group_hr_holidays_user"/>
                <button string="Refuse" name="action_refuse" type="object"
                    icon="fa-times"
                    invisible="state not in ('confirm', 'validate1')"
                    groups="hr_holidays.group_hr_holidays_user"/>
                <field name="activity_exception_decoration" widget="activity_exception"/>
            </tree>
        </field>
    </record>

    <record id="hr_leave_view_tree_my" model="ir.ui.view">
        <field name="name">hr.holidays.view.tree</field>
        <field name="model">hr.leave</field>
        <field name="inherit_id" ref="hr_leave_view_tree"/>
        <field name="mode">primary</field>
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='all_employee_ids']" position="attributes">
                <attribute name="column_invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='employee_id']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='department_id']" position="attributes">
                <attribute name="column_invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='holiday_type']" position="attributes">
                <attribute name="column_invisible">1</attribute>
            </xpath>
            <xpath expr="//button[@name='action_approve']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//button[@name='action_validate']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//button[@name='action_refuse']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
        </field>
    </record>

    <record id="hr_leave_view_search_my" model="ir.ui.view">
        <field name="name">hr.holidays.view.search.my</field>
        <field name="model">hr.leave</field>
        <field name="inherit_id" ref="view_hr_holidays_filter"/>
        <field name="mode">primary</field>
        <field name="priority">32</field>
        <field name="arch" type="xml">
            <xpath expr="//searchpanel" position="replace"/>
            <xpath expr="//filter[@name='department']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//filter[@name='my_team']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//filter[@name='active_employee']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//filter[@name='my_leaves']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//filter[@name='group_employee']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
        </field>
    </record>

    <record id="hr_leave_view_search_manager" model="ir.ui.view">
        <field name="name">hr.holidays.view.search.manager</field>
        <field name="model">hr.leave</field>
        <field name="inherit_id" ref="view_hr_holidays_filter"/>
        <field name="mode">primary</field>
        <field name="priority">33</field>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='my_leaves']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//filter[@name='group_employee']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
        </field>
    </record>

    <record id="hr_leave_action_new_request" model="ir.actions.act_window">
        <field name="name">Dashboard</field>
        <field name="res_model">hr.leave</field>
        <field name="view_mode">calendar,tree,form,activity</field>
        <field name="domain">[('user_id', '=', uid), ('employee_id.company_id', 'in', allowed_company_ids)]</field>
        <field name="context">{'short_name': 1, 'search_default_active_time_off': 1}</field>
        <field name="search_view_id" ref="hr_holidays.hr_leave_view_search_my"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Keep track of your PTOs.
            </p><p>
                A great way to keep track on your time off requests, sick days, and approval status.
            </p>
        </field>
    </record>

    <record id="hr_leave_action_new_request_view_calendar" model="ir.actions.act_window.view">
        <field name="sequence">1</field>
        <field name="view_mode">calendar</field>
        <field name="act_window_id" ref="hr_leave_action_new_request"/>
        <field name="view_id" ref="hr_leave_view_dashboard"/>
    </record>

    <record id="hr_leave_action_new_request_view_tree" model="ir.actions.act_window.view">
        <field name="sequence">2</field>
        <field name="view_mode">tree</field>
        <field name="act_window_id" ref="hr_leave_action_new_request"/>
        <field name="view_id" ref="hr_leave_view_tree_my"/>
    </record>

    <record id="hr_leave_action_new_request_view_form" model="ir.actions.act_window.view">
        <field name="sequence">3</field>
        <field name="view_mode">form</field>
        <field name="act_window_id" ref="hr_leave_action_new_request"/>
        <field name="view_id" ref="hr_leave_view_form"/>
    </record>

    <record id="hr_leave_action_my_request" model="ir.actions.act_window">
        <field name="name">Time Off Request</field>
        <field name="res_model">hr.leave</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
    <record id="hr_leave_action_my_request_view_form" model="ir.actions.act_window.view">
        <field name="view_mode">form</field>
        <field name="act_window_id" ref="hr_leave_action_my_request"/>
        <field name="view_id" ref="hr_leave_view_form_dashboard_new_time_off"/>
    </record>

    <record id="hr_leave_view_kanban_my" model="ir.ui.view">
        <field name="name">hr.leave.view.kanban.my</field>
        <field name="inherit_id" ref="hr_leave_view_kanban"/>
        <field name="mode">primary</field>
        <field name="model">hr.leave</field>
        <field name="arch" type="xml">
            <strong class="o_kanban_record_title" position='attributes'>
                <attribute name="invisible">1</attribute>
            </strong>
        </field>
    </record>

    <record id="hr_leave_action_my" model="ir.actions.act_window">
        <field name="name">My Time Off</field>
        <field name="res_model">hr.leave</field>
        <field name="view_mode">tree,form,kanban,activity</field>
        <field name="context">{}</field>
        <field name="search_view_id" ref="hr_leave_view_search_my"/>
        <field name="view_ids" eval="[(5, 0, 0),
                (0, 0, {'view_mode': 'kanban', 'view_id': ref('hr_leave_view_kanban_my')})]"/>
        <field name="domain">[('user_id', '=', uid)]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Keep track of your PTOs.
            </p><p>
                A great way to keep track on your time off requests, sick days, and approval status.
            </p>
        </field>
    </record>

    <record id="hr_leave_action_my_view_tree" model="ir.actions.act_window.view">
        <field name="sequence">1</field>
        <field name="view_mode">tree</field>
        <field name="act_window_id" ref="hr_leave_action_my"/>
        <field name="view_id" ref="hr_leave_view_tree_my"/>
    </record>

    <record id="hr_leave_action_my_view_form" model="ir.actions.act_window.view">
        <field name="sequence">2</field>
        <field name="view_mode">form</field>
        <field name="act_window_id" ref="hr_leave_action_my"/>
        <field name="view_id" ref="hr_leave_view_form"/>
    </record>

    <record id="hr_leave_action_action_approve_department" model="ir.actions.act_window">
        <field name="name">All Time Off</field>
        <field name="res_model">hr.leave</field>
        <field name="view_mode">tree,kanban,form,calendar,activity</field>
        <field name="search_view_id" ref="hr_holidays.hr_leave_view_search_manager"/>
        <field name="context">{
            'search_default_waiting_for_me': 1,
            'search_default_waiting_for_me_manager': 2,
            'hide_employee_name': 1,
            'holiday_status_display_name': False}</field>
        <field name="domain">['|', ('employee_id.company_id', 'in', allowed_company_ids),
                                    '&amp;', ('multi_employee', '=', True),
                                    '&amp;', ('state', 'in', ['draft', 'confirm', 'validate1']),
                                    ('employee_ids.company_id', 'in', allowed_company_ids)]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Meet the time off dashboard.
            </p><p>
                A great way to keep track on employee’s PTOs, sick days, and approval status.
            </p>
        </field>
    </record>

    <record id="hr_leave_action_holiday_allocation_id" model="ir.actions.act_window">
        <field name="name">Time Off</field>
        <field name="res_model">hr.leave</field>
        <field name="view_mode">tree,kanban,form,calendar,activity</field>
        <field name="search_view_id" ref="hr_holidays.hr_leave_view_search_manager"/>
        <field name="context">{
            'hide_employee_name': 1,
            'holiday_status_display_name': False}
        </field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Meet the time off dashboard.
            </p><p>
                A great way to keep track on employee’s PTOs, sick days, and approval status.
            </p>
        </field>
    </record>

    <record id="action_view_tree_manager_approve" model="ir.actions.act_window.view">
        <field name="sequence" eval="1"/>
        <field name="view_mode">tree</field>
        <field name="view_id" ref="hr_leave_view_tree"/>
        <field name="act_window_id" ref="hr_leave_action_action_approve_department"/>
    </record>
    <record id="action_view_kanban_manager_approve" model="ir.actions.act_window.view">
        <field name="sequence" eval="2"/>
        <field name="view_mode">kanban</field>
        <field name="view_id" ref="hr_leave_view_kanban_approve_department"/>
        <field name="act_window_id" ref="hr_leave_action_action_approve_department"/>
    </record>
    <record id="action_view_form_manager_approve" model="ir.actions.act_window.view">
        <field name="sequence" eval="3"/>
        <field name="view_mode">form</field>
        <field name="view_id" ref="hr_leave_view_form_manager"/>
        <field name="act_window_id" ref="hr_leave_action_action_approve_department"/>
    </record>
    <record id="action_view_calendar_manager_approve" model="ir.actions.act_window.view">
        <field name="sequence" eval="4"/>
        <field name="view_mode">calendar</field>
        <field name="view_id" eval="False"/>
        <field name="act_window_id" ref="hr_leave_action_action_approve_department"/>
    </record>
    <record id="action_view_activity_manager_approve" model="ir.actions.act_window.view">
        <field name="sequence" eval="5"/>
        <field name="view_mode">activity</field>
        <field name="view_id" eval="False"/>
        <field name="act_window_id" ref="hr_leave_action_action_approve_department"/>
    </record>

    <record id="hr_leave_action_action_department" model="ir.actions.act_window">
        <field name="name">Time Off Analysis</field>
        <field name="res_model">hr.leave</field>
        <field name="view_mode">graph,pivot</field>
        <field name="context">{
            'search_default_department_id': [active_id],
            'default_department_id': active_id}
        </field>
        <field name="domain">[('holiday_type','=','employee')]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No data yet!
            </p>
        </field>
    </record>

    <record id="view_holiday_pivot" model="ir.ui.view">
        <field name="name">hr.holidays.report_pivot</field>
        <field name="model">hr.leave</field>
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <pivot string="Time Off Summary" sample="1">
                <field name="employee_id" type="row"/>
                <field name="date_from" type="col"/>
                <field name="number_of_days" type="measure"/>
            </pivot>
        </field>
    </record>

    <record id="view_holiday_graph" model="ir.ui.view">
        <field name="name">hr.holidays.report_graph</field>
        <field name="model">hr.leave</field>
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <graph string="Time Off Summary" sample="1">
                <field name="employee_id"/>
                <field name="number_of_hours" widget="float_time"/>
                <field name="number_of_days" type="measure"/>
            </graph>
        </field>
    </record>

    <record id="view_holiday_list" model="ir.ui.view">
        <field name="name">hr.holidays.report_list</field>
        <field name="model">hr.leave</field>
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <tree string="Time Off Summary" sample="1">
                <field name="employee_id"/>
                <field name="number_of_days" string="Number of Days" type="measure"/>
                <field name="date_from"/>
                <field name="date_to"/>
                <field name="state"/>
                <field name="name"/>
            </tree>
        </field>
    </record>

    <record id="hr_leave_report_search_view" model="ir.ui.view">
        <field name="name">he.leave.report.search.view</field>
        <field name="inherit_id" ref="view_hr_holidays_filter"/>
        <field name="mode">primary</field>
        <field name="model">hr.leave</field>
        <field name="arch" type="xml">
            <filter name="active_employee" position="replace"/>
            <filter name="archive" position="replace"/>
            <filter name="active_time_off" position="replace"/>
        </field>
    </record>

    <record id="action_hr_available_holidays_report" model="ir.actions.act_window">
        <field name="name">Time Off Analysis</field>
        <field name="res_model">hr.leave</field>
        <field name="view_mode">tree,graph,pivot,calendar,form</field>
        <field name="search_view_id" ref="hr_leave_report_search_view"/>
        <field name="context">{'search_default_filter_date_from': 1, 'search_default_group_employee': 1, 'search_default_group_type': 1}</field>
        <field name="domain">[('active', '=', True), ('active_employee','=',True)]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No data to display
            </p>
        </field>
    </record>

    <record id="action_window_leave_list" model="ir.actions.act_window.view">
        <field name="sequence" eval="5"/>
        <field name="view_mode">tree</field>
        <field name="view_id" ref="view_holiday_list"/>
        <field name="act_window_id" ref="action_hr_available_holidays_report"/>
    </record>

    <record id="action_window_leave_graph" model="ir.actions.act_window.view">
        <field name="sequence" eval="10"/>
        <field name="view_mode">graph</field>
        <field name="view_id" ref="view_holiday_graph"/>
        <field name="act_window_id" ref="action_hr_available_holidays_report"/>
    </record>

</odoo>

```

## File: views\hr_views.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<odoo>

   <record id="hr_employee_action_from_department" model="ir.actions.act_window">
       <field name="name">Absent Employees</field>
       <field name="res_model">hr.employee</field>
       <field name="view_mode">tree,kanban,form</field>
       <field name="context">{
           'search_default_on_timeoff': 1,
           'searchpanel_default_department_id': active_id,
           'default_department_id': active_id}
       </field>
       <field name="search_view_id" ref="hr.view_employee_filter"/>
   </record>

    <!--Hr Department Inherit Kanban view-->
    <record id="hr_department_view_kanban" model="ir.ui.view">
        <field name="name">hr.department.kanban.inherit</field>
        <field name="model">hr.department</field>
        <field name="inherit_id" ref="hr.hr_department_view_kanban"/>
        <field name="arch" type="xml">
            <data>
                <xpath expr="//templates" position="before">
                    <t groups="hr_holidays.group_hr_holidays_user">
                        <field name="leave_to_approve_count"/>
                        <field name="allocation_to_approve_count"/>
                        <field name="total_employee"/>
                        <field name="absence_of_today"/>
                    </t>
                </xpath>

                <xpath expr="//div[hasclass('o_kanban_primary_right')]" position="inside">
                    <t groups="hr_holidays.group_hr_holidays_user">
                        <div t-if="record.leave_to_approve_count.raw_value > 0" class="row ml16">
                            <div class="col">
                                <a name="action_open_leave_department" type="object">
                                    <field name="leave_to_approve_count"/> Time Off Requests
                                </a>
                            </div>
                        </div>
                        <div t-if="record.allocation_to_approve_count.raw_value > 0" class="row ml16">
                            <div class="col">
                                <a name="action_open_allocation_department" type="object">
                                    <field name="allocation_to_approve_count"/> Allocation Requests
                                </a>
                            </div>
                        </div>
                    </t>
                </xpath>

                <xpath expr="//div[hasclass('o_kanban_card_lower_content')]" position="inside">
                    <t groups="hr_holidays.group_hr_holidays_user">
                        <div class="row o_kanban_primary_bottom bottom_block bg-view"
                             t-if="record.absence_of_today.raw_value > 0">
                            <div class="col-3">
                                <a name="%(hr_employee_action_from_department)d" type="action" title="Absent Employee(s), Whose time off requests are either confirmed or validated on today">Absence</a>
                            </div>
                            <div class="col-9">
                                <field name="absence_of_today" widget="progressbar" options="{'current_value': 'absence_of_today', 'max_value': 'total_employee', 'editable': false}"/>
                            </div>
                        </div>
                    </t>
                </xpath>

                <xpath expr="//div[hasclass('o_kanban_manage_reports')]" position="inside">
                    <a role="menuitem" class="dropdown-item" name="%(hr_leave_action_action_department)d" type="action" groups="hr_holidays.group_hr_holidays_user">
                        Time Off
                    </a>
                </xpath>
            </data>
        </field>
    </record>

    <!--Hr Employee inherit search view-->
    <record id="hr_employee_view_search" model="ir.ui.view">
        <field name="name">hr.employee.search.view.inherit</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='my_team']" position="before">
                <filter name="on_timeoff" string="On Time Off" domain="[('is_absent', '=', True)]"/>
                <filter name="at_work" string="At work" domain="[('is_absent', '=', False)]"/>
                <separator/>
            </xpath>
        </field>
    </record>

    <!-- hr_employee_public_view_kanban -->
    <record id="hr_kanban_view_public_employees_kanban" model="ir.ui.view">
        <field name="name">hr.employee.public.kanban.leaves.status</field>
        <field name="model">hr.employee.public</field>
        <field name="inherit_id" ref="hr.hr_employee_public_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//templates" position="before">
                <field name="is_absent"/>
            </xpath>
        </field>
    </record>

    <record id="hr_kanban_view_employees_kanban" model="ir.ui.view">
        <field name="name">hr.employee.kanban.leaves.status</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.hr_kanban_view_employees"/>
        <field name="arch" type="xml">
            <xpath expr="//templates" position="before">
                <field name="current_leave_id"/>
                <field name="current_leave_state"/>
                <field name="leave_date_from"/>
                <field name="leave_date_to"/>
                <field name="is_absent"/>
            </xpath>
            <xpath expr="//field[@name='hr_icon_display'][hasclass('o_employee_availability')]" position="attributes">
                <attribute name="widget">hr_presence_status_private</attribute>
            </xpath>
        </field>
    </record>

    <!-- Hr employee inherit Legal Leaves -->
    <record id="view_employee_form_leave_inherit" model="ir.ui.view">
        <field name="name">hr.employee.leave.form.inherit</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_form"/>
        <field name = "priority" eval="20" />
        <field name="arch" type="xml">
            <xpath expr="//group[@name='managers']" position="inside">
                <field name="leave_manager_id" widget="many2one_avatar_user"/>
            </xpath>
            <xpath expr="//group[@name='managers']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <field name="show_leaves" invisible="1"/>
                <field name="is_absent" invisible="1"/>
                <field name="hr_icon_display" invisible="1"/>
                <button name="action_time_off_dashboard"
                        type="object"
                        class="oe_stat_button"
                        context="{'search_default_employee_ids': id}"
                        invisible="not is_absent">
                        <div invisible="hr_icon_display != 'presence_holiday_present'"
                              role="img" class="fa fa-fw fa-plane o_button_icon text-success" aria-label="Off Till" title="Off Till"/>
                        <div invisible="hr_icon_display != 'presence_holiday_absent'" role="img"
                             class="fa fa-fw fa-plane o_button_icon text-warning" aria-label="Off Till" title="Off Till"/>
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_text">
                            Off Till
                        </span>
                        <span class="o_stat_value">
                            <field name="leave_date_to"/>
                        </span>
                    </div>
                </button>
                <button name="action_time_off_dashboard"
                        type="object"
                        class="oe_stat_button"
                        icon="fa-calendar"
                        invisible="not show_leaves"
                        context="{'search_default_employee_ids': id}"
                        groups="base.group_user"
                        help="Remaining leaves">
                    <div class="o_field_widget o_stat_info" invisible="allocation_display == '0'">
                        <span class="o_stat_value">
                            <field name="allocation_remaining_display"/>/<field name="allocation_display"/> Days
                        </span>
                        <span class="o_stat_text">
                            Time Off
                        </span>
                    </div>
                    <div class="o_field_widget o_stat_info" invisible="allocation_display != '0'">
                        <span class="o_stat_text">
                           Time Off
                        </span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>

    <record id="view_employee_tree_inherit_leave" model="ir.ui.view">
        <field name="name">hr.employee.tree.leave</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='work_location_id']" position="after">
                <field name="leave_manager_id" optional="hide" string="Time Off Approver" widget="many2one_avatar_user"/>
            </xpath>
        </field>
    </record>

    <record id="hr_employee_public_form_view_inherit" model="ir.ui.view">
        <field name="name">hr.employee.public.leave.form.inherit</field>
        <field name="model">hr.employee.public</field>
        <field name="inherit_id" ref="hr.hr_employee_public_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='coach_id']" position="after">
                <field name="company_id" invisible="1"/>
                <field name="leave_manager_id" widget="many2one_avatar_user"/>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <field name="show_leaves" invisible="1"/>
                <field name="is_absent" invisible="1"/>
                <button disabled="1"
                        class="oe_stat_button"
                        invisible="not is_absent">
                        <div role="img" invisible="hr_icon_display != 'presence_holiday_present'"
                             class="fa fa-fw fa-plane o_button_icon text-success" aria-label="Off Till" title="Off Till"/>
                        <div invisible="hr_icon_display != 'presence_holiday_absent'" role="img"
                             class="fa fa-fw fa-plane o_button_icon text-warning" aria-label="Off Till" title="Off Till"/>
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_text">
                            Off Till
                        </span>
                        <span class="o_stat_value">
                            <field name="leave_date_to"/>
                        </span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>

    <record id="res_users_view_form" model="ir.ui.view">
        <field name="name">hr.user.preferences.view.form.leave.inherit</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="hr.res_users_view_form_profile"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="inside">
                <button name="%(hr_leave_action_new_request)d"
                string="Request Time off"
                type="action"
                class="btn btn-primary"/>
                <button name="%(hr_leave_allocation_action_my)d"
                string="Request Allocation"
                type="action"
                class="btn btn-primary"/>
            </xpath>
            <xpath expr="//group[@name='managers']" position="inside">
                <field name="leave_manager_id" readonly="not can_edit"/>
            </xpath>
            <xpath expr="//group[@name='managers']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <field name="show_leaves" invisible="1"/>
                <field name="employee_ids" invisible="1"/>
                <field name="is_absent" invisible="1"/>
                <field name="hr_icon_display" invisible="1"/>
                <button name="%(hr_leave_action_new_request)d" type="action"
                        class="oe_stat_button"
                        invisible="context.get('from_my_profile', False) or not is_absent">
                        <div invisible="hr_icon_display != 'presence_holiday_present'"
                             role="img" class="fa fa-fw fa-plane o_button_icon text-success" aria-label="Off Till"
                             title="Off Till"/>
                        <div invisible="hr_icon_display != 'presence_holiday_absent'"
                             role="img" class="fa fa-fw fa-plane o_button_icon text-warning" aria-label="Off Till"
                             title="Off Till"/>
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_text">
                            Off Till
                        </span>
                        <span class="o_stat_value">
                            <field name="leave_date_to"/>
                        </span>
                    </div>
                </button>
                <button name="%(hr_leave_action_new_request)d"
                        type="action"
                        class="oe_stat_button"
                        icon="fa-calendar"
                        invisible="not show_leaves"
                        groups="base.group_user"
                        help="Remaining leaves">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="allocation_remaining_display"/>/<field name="allocation_display"/> Days
                        </span>
                        <span class="o_stat_text">
                            Time Off
                        </span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\mail_activity_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <!-- Activity types config -->
    <record id="mail_activity_type_action_config_hr_holidays" model="ir.actions.act_window">
        <field name="name">Activity Types</field>
        <field name="res_model">mail.activity.type</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">['|', ('res_model', '=', False), ('res_model', 'in', ['hr.leave', 'hr.leave.allocation'])]</field>
        <field name="context">{'default_res_model': 'hr.leave'}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No data to display
            </p>
            <p>
                Try to add some records, or make sure that there is no active filter in the search bar.
            </p>
        </field>
    </record>
</odoo>

```

## File: views\resource_views.xml

```xml
<?xml version='1.0' encoding='UTF-8' ?>
<odoo>
    <!-- Holiday on resource leave -->
    <record id="resource_calendar_leaves_view_search_inherit" model="ir.ui.view">
        <field name="name">resource.calendar.leaves.search.inherit</field>
        <field name="model">resource.calendar.leaves</field>
        <field name="inherit_id" ref="resource.view_resource_calendar_leaves_search"/>
        <field name="arch" type="xml">
            <filter name="resource" position="attributes">
                <attribute name="invisible">1</attribute>
            </filter>
        </field>
    </record>
    <record id="resource_calendar_leave_form_inherit" model="ir.ui.view">
        <field name="name">resource.calendar.leaves.form.inherit</field>
        <field name="model">resource.calendar.leaves</field>
        <field name="inherit_id" ref="resource.resource_calendar_leave_form"/>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="holiday_id" options="{'no_quick_create': True}"/>
            </field>
        </field>
    </record>

    <record id="resource_calendar_leaves_tree_inherit" model="ir.ui.view">
        <field name="name">resource.calendar.leaves.tree.inherit</field>
        <field name="model">resource.calendar.leaves</field>
        <field name="inherit_id" ref="resource.resource_calendar_leave_tree"/>
        <field name="mode">primary</field>
        <field name="priority">10</field>
        <field name="arch" type="xml">
            <xpath expr="//tree" position="attributes">
                <attribute name="editable">bottom</attribute>
            </xpath>
            <xpath expr="//field[@name='name']" position="attributes">
                <attribute name="string">Name</attribute>
                <attribute name="required">1</attribute>
            </xpath>
            <xpath expr="//field[@name='resource_id']" position="attributes">
                <attribute name="column_invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='date_to']" position="after">
                <field name="company_id" column_invisible="True"/>
                <xpath expr="//field[@name='calendar_id']" position="move"/>
            </xpath>
        </field>
    </record>

    <record id="resource_calendar_global_leaves_action_from_calendar" model="ir.actions.act_window">
        <field name="name">Resource Time Off</field>
        <field name="res_model">resource.calendar.leaves</field>
        <field name="view_mode">tree</field>
        <field name="view_id" ref="resource_calendar_leaves_tree_inherit"/>
        <field name="domain">['&amp;', ('resource_id', '=', False), ('calendar_id', 'in', (active_id, False))]</field>
        <field name="context">{
            'default_calendar_id': active_id,
            'search_default_filter_date': True}</field>
        <field name="search_view_id" ref="hr_holidays.resource_calendar_leaves_view_search_inherit"/>
    </record>

    <record id="resource_calendar_form_inherit" model="ir.ui.view">
        <field name="name">resource.calendar.form.inherit</field>
        <field name="model">resource.calendar</field>
        <field name="inherit_id" ref="resource.resource_calendar_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='%(resource.resource_calendar_leaves_action_from_calendar)d']" position="before">
                <button name="%(resource_calendar_global_leaves_action_from_calendar)d" type="action"
                                icon="fa-sun-o" class="oe_stat_button">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_text">
                            <field name="associated_leaves_count"/> Public
                        </span>
                        <span class="o_stat_text">
                            Time Off
                        </span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>

    <record id="open_view_public_holiday" model="ir.actions.act_window">
        <field name="name">Public Holidays</field>
        <field name="res_model">resource.calendar.leaves</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('resource_id', '=', False)]</field>
        <field name="view_id" ref="resource_calendar_leaves_tree_inherit"/>
        <field name="context">{
            'search_default_filter_date': True}</field>
    </record>
</odoo>

```

## File: wizard\hr_departure_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, timedelta

from odoo import api, fields, models


class HrDepartureWizard(models.TransientModel):
    _inherit = 'hr.departure.wizard'

    cancel_leaves = fields.Boolean("Cancel Future Time Off", default=True,
        help="Cancel all time off after this date.")
    archive_allocation = fields.Boolean("Archive Employee Allocations", default=True,
        help="Remove employee from existing accrual plans.")

    def action_register_departure(self):
        super(HrDepartureWizard, self).action_register_departure()
        if self.cancel_leaves:
            future_leaves = self.env['hr.leave'].search([('employee_id', '=', self.employee_id.id), 
                                                         ('date_to', '>', self.departure_date),
                                                         ('state', '!=', 'refuse')])
            future_leaves.action_refuse()

        if self.archive_allocation:
            employee_allocations = self.env['hr.leave.allocation'].search([('employee_id', '=', self.employee_id.id)])
            employee_allocations.action_archive()

```

## File: wizard\hr_departure_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_departure_wizard_view_form" model="ir.ui.view">
        <field name="name">hr.departure.wizard.view.form.extend3</field>
        <field name="model">hr.departure.wizard</field>
        <field name="inherit_id" ref="hr.hr_departure_wizard_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//div[@id='activities_label']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//div[@id='activities']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//div[@id='activities']" position="inside">
                <div><field name="cancel_leaves"/><label for="cancel_leaves" string="Time Off"/></div>
                <div><field name="archive_allocation"/><label for="archive_allocation" string="Allocations"/></div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\hr_holidays_cancel_leave.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _


class HrHolidaysCancelLeave(models.TransientModel):
    _name = 'hr.holidays.cancel.leave'
    _description = 'Cancel Time Off Wizard'

    leave_id = fields.Many2one('hr.leave', string="Time Off Request", required=True)
    reason = fields.Text(required=True)

    def action_cancel_leave(self):
        self.ensure_one()

        self.leave_id._action_user_cancel(self.reason)

        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'type': 'success',
                'message': _("Your time off has been canceled."),
                'next': {'type': 'ir.actions.act_window_close'},
            }
        }

```

## File: wizard\hr_holidays_cancel_leave_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_holidays_cancel_leave_form" model="ir.ui.view">
        <field name="model">hr.holidays.cancel.leave</field>
        <field name="arch" type="xml">
            <form string="Cancel Time Off">
                <group>
                    <field name="leave_id" invisible="1" />
                    <field name="reason" placeholder="Provide a reason to delete an approved time off" />
                </group>
                <footer>
                    <button name="action_cancel_leave" type="object" class="btn-primary" string="Delete Time Off" />
                    <button special="cancel" string="Discard" close="1" />
                </footer>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\hr_holidays_summary_employees.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import time

from odoo import api, fields, models


class HolidaysSummaryEmployee(models.TransientModel):

    _name = 'hr.holidays.summary.employee'
    _description = 'HR Time Off Summary Report By Employee'

    date_from = fields.Date(string='From', required=True, default=lambda *a: time.strftime('%Y-%m-01'))
    emp = fields.Many2many('hr.employee', 'summary_emp_rel', 'sum_id', 'emp_id', string='Employee(s)')
    holiday_type = fields.Selection([
        ('Approved', 'Approved'),
        ('Confirmed', 'Confirmed'),
        ('both', 'Both Approved and Confirmed')
    ], string='Select Time Off Type', required=True, default='Approved')

    def print_report(self):
        self.ensure_one()
        [data] = self.read()
        data['emp'] = self.env.context.get('active_ids', [])
        employees = self.env['hr.employee'].browse(data['emp'])
        datas = {
            'ids': [],
            'model': 'hr.employee',
            'form': data
        }
        return self.env.ref('hr_holidays.action_report_holidayssummary').report_action(employees, data=datas)

```

## File: wizard\hr_holidays_summary_employees_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record id="view_hr_holidays_summary_employee" model="ir.ui.view">
            <field name="name">hr.holidays.summary.employee.form</field>
            <field name="model">hr.holidays.summary.employee</field>
            <field name="arch" type="xml">
                <form string="Time Off Summary">
                    <group col="4" colspan="6">
                        <field name="date_from"/>
                        <newline/>
                        <field name="holiday_type"/>
                        <newline/>
                        <field name="emp" invisible="True"/>
                    </group>
                    <footer>
                        <button name="print_report" string="Print" type="object" class="btn-primary" data-hotkey="q"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x" />
                    </footer>
                </form>
            </field>
        </record>

        <record id="action_hr_holidays_summary_employee" model="ir.actions.act_window">
            <field name="name">Time Off Summary</field>
            <field name="res_model">hr.holidays.summary.employee</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
            <field name="binding_model_id" ref="hr.model_hr_employee" />
            <field name="binding_type">report</field>
        </record>

</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_holidays_cancel_leave
from . import hr_holidays_summary_employees
from . import hr_departure_wizard

```

