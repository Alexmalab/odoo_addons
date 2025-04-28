# Odoo Module: hr_holidays

Category: Human Resources/Time Off

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import report
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Time Off',
    'version': '1.5',
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
        'data/mail_data.xml',
        'data/hr_holidays_data.xml',
        'data/ir_cron_data.xml',

        'security/hr_holidays_security.xml',
        'security/ir.model.access.csv',

        'views/resource_views.xml',
        'views/hr_leave_views.xml',
        'views/hr_leave_type_views.xml',
        'views/hr_leave_allocation_views.xml',
        'views/hr_leave_accrual_views.xml',
        'views/mail_activity_views.xml',

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
    'auto_install': False,
    'assets': {
        'mail.assets_discuss_public': [
            'hr_holidays/static/src/components/*/*',
            'hr_holidays/static/src/models/*/*.js',
        ],
        'web.assets_backend': [
            'hr_holidays/static/src/js/time_off_calendar.js',
            'hr_holidays/static/src/js/float_without_trailing_zeros.js',
            'hr_holidays/static/src/js/time_off_calendar_employee.js',
            'hr_holidays/static/src/js/radio_image.js',
            'hr_holidays/static/src/js/leave_stats_widget.js',
            'hr_holidays/static/src/models/*/*.js',
            'hr_holidays/static/src/scss/time_off.scss',
            'hr_holidays/static/src/components/*/*.scss',
            'hr_holidays/static/src/scss/accrual_plan_level.scss',
        ],
        'web.tests_assets': [
            'hr_holidays/static/tests/helpers/**/*',
        ],
        'web.qunit_suite_tests': [
            'hr_holidays/static/src/components/*/tests/*.js',
            'hr_holidays/static/tests/test_leave_stats_widget.js',
        ],
        'web.assets_qweb': [
            'hr_holidays/static/src/components/*/*.xml',
            'hr_holidays/static/src/xml/*.xml',
        ],
        'web.assets_tests': [
            '/hr_holidays/static/tests/tours/**/**.js'
        ],
    },
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
            <field name="responsible_id" ref="base.user_admin"/>
            <field name="icon_id" ref="hr_holidays.icon_14"/>
            <field name="color">2</field>
        </record>

        <!-- Sick leave -->
        <record id="holiday_status_sl" model="hr.leave.type">
            <field name="name">Sick Time Off</field>
            <field name="requires_allocation">no</field>
            <field name="color_name">red</field>
            <field name="leave_notif_subtype_id" ref="mt_leave_sick"/>
            <field name="responsible_id" ref="base.user_admin"/>
            <field name="support_document">True</field>
            <field name="icon_id" ref="hr_holidays.icon_21"/>
            <field name="color">3</field>
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
            <field name="responsible_id" ref="base.user_admin"/>
            <field name="icon_id" ref="hr_holidays.icon_4"/>
            <field name="color">4</field>
        </record>

        <!--Unpaid Leave -->
        <record id="holiday_status_unpaid" model="hr.leave.type">
            <field name="name">Unpaid</field>
            <field name="requires_allocation">no</field>
            <field name="leave_validation_type">both</field>
            <field name="allocation_validation_type">officer</field>
            <field name="request_unit">hour</field>
            <field name="unpaid" eval="True"/>
            <field name="leave_notif_subtype_id" ref="mt_leave_unpaid"/>
            <field name="responsible_id" ref="base.user_admin"/>
            <field name="icon_id" ref="hr_holidays.icon_28"/>
            <field name="color">5</field>
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
        <field name="groups_id" eval="[(4, ref('hr_holidays.group_hr_holidays_user'))]"/>
    </record>

    <!--Leave Type-->
    <record id="hr_holiday_status_dv" model="hr.leave.type">
        <field name="name">Parental Leaves</field>
        <field name="requires_allocation">yes</field>
        <field name="employee_requests">no</field>
        <field name="color_name">brown</field>
        <field name="leave_validation_type">both</field>
        <field name="allocation_validation_type">officer</field>
        <field name="responsible_id" ref="base.user_admin"/>
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
        <field name="state">validate</field>
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
    <function model="hr.leave.allocation" name="action_validate">
        <value eval="ref('hr_holidays.hr_holidays_int_tour')"/>
    </function>

    <record id="hr_holidays_vc" model="hr.leave.allocation">
        <field name="name">Summer Vacation</field>
        <field name="holiday_status_id" ref="holiday_status_unpaid"/>
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
        <field name="state">validate</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <!-- leave request -->
    <record id="hr_holidays_cl" model="hr.leave">
        <field name="name">Trip with Family</field>
        <field name="holiday_status_id" ref="holiday_status_comp"/>
        <field eval="(datetime.now() + relativedelta(day=1, weekday=0)).strftime('%Y-%m-%d 04:00:00')" name="date_from"/>
        <field eval="(datetime.now() + relativedelta(day=1, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 20:00:00')" name="date_to"/>
        <field eval="(datetime.now() + relativedelta(day=1, weekday=0)).strftime('%Y-%m-%d 04:00:00')" name="request_date_from"/>
        <field eval="(datetime.now() + relativedelta(day=1, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 20:00:00')" name="request_date_to"/>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_admin'))]"/>
    </record>

    <record id="hr_holidays_sl" model="hr.leave">
        <field name="name">Doctor Appointment</field>
        <field name="holiday_status_id" ref="holiday_status_sl"/>
        <field eval="(datetime.now() + relativedelta(day=20, weekday=0)).strftime('%Y-%m-%d 04:00:00')" name="date_from"/>
        <field eval="(datetime.now() + relativedelta(day=20, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 20:00:00')" name="date_to"/>
        <field eval="(datetime.now() + relativedelta(day=20, weekday=0)).strftime('%Y-%m-%d 04:00:00')" name="request_date_from"/>
        <field eval="(datetime.now() + relativedelta(day=20, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 20:00:00')" name="request_date_to"/>
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
        <field name="state">validate</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <record id="hr_holidays_allocation_pl_al" model="hr.leave.allocation">
        <field name="name">Parental Leaves</field>
        <field name="holiday_status_id" ref="hr_holiday_status_dv"/>
        <field name="number_of_days">10</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_al'))]"/>
        <field name="state">validate</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <record id="hr_holidays_vc_al" model="hr.leave.allocation">
        <field name="name">Summer Vacation</field>
        <field name="holiday_status_id" ref="holiday_status_unpaid"/>
        <field name="number_of_days">12</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_al'))]"/>
        <field name="state">validate</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <!-- leave request -->
    <record id="hr_holidays_cl_al" model="hr.leave">
        <field name="name">Trip with Friends</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field eval="time.strftime('%Y-%m-04')" name="date_from"/>
        <field eval="time.strftime('%Y-%m-10')" name="date_to"/>
        <field eval="time.strftime('%Y-%m-04')" name="request_date_from"/>
        <field eval="time.strftime('%Y-%m-10')" name="request_date_to"/>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_al'))]"/>
    </record>
    <function model="hr.leave" name="action_validate">
        <value eval="ref('hr_holidays.hr_holidays_cl_al')"/>
    </function>

    <record id="hr_holidays_sl_al" model="hr.leave">
        <field name="name">Dentist appointment</field>
        <field name="holiday_status_id" ref="holiday_status_sl"/>
        <field eval="(datetime.now()+relativedelta(months=1, day=17, weekday=0)).strftime('%Y-%m-%d 04:00:00')" name="date_from"/>
        <field eval="(datetime.now()+relativedelta(months=1, day=17, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 20:00:00')" name="date_to"/>
        <field eval="(datetime.now()+relativedelta(months=1, day=17, weekday=0)).strftime('%Y-%m-%d 04:00:00')" name="request_date_from"/>
        <field eval="(datetime.now()+relativedelta(months=1, day=17, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 20:00:00')" name="request_date_to"/>
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
        <field name="state">validate</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <record id="hr_holidays_vc_mit" model="hr.leave.allocation">
        <field name="name">Summer Vacation</field>
        <field name="holiday_status_id" ref="holiday_status_unpaid"/>
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
        <field eval="time.strftime('%Y-%m-18')" name="date_from"/>
        <field eval="time.strftime('%Y-%m-24')" name="date_to"/>
        <field eval="time.strftime('%Y-%m-18')" name="request_date_from"/>
        <field eval="time.strftime('%Y-%m-24')" name="request_date_to"/>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_mit'))]"/>
    </record>
    <function model="hr.leave" name="action_validate">
        <value eval="ref('hr_holidays.hr_holidays_cl_mit')"/>
    </function>

    <record id="hr_holidays_cl_mit_2" model="hr.leave">
        <field name="name">Trip</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field eval="(datetime.now() + relativedelta(day=5, weekday=0)).strftime('%Y-%m-%d 04:00:00')" name="date_from"/>
        <field eval="(datetime.now() + relativedelta(day=5, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 20:00:00')" name="date_to"/>
        <field eval="(datetime.now() + relativedelta(day=5, weekday=0)).strftime('%Y-%m-%d 04:00:00')" name="request_date_from"/>
        <field eval="(datetime.now() + relativedelta(day=5, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 20:00:00')" name="request_date_to"/>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_mit'))]"/>
    </record>

    <!-- ++++++++++++++++++++++  Marc Demo  ++++++++++++++++++++++ -->

    <record id="hr_holidays_allocation_cl_qdp" model="hr.leave.allocation">
        <field name="name">Paid Time Off for Marc Demo</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field name="number_of_days">20</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_qdp'))]"/>
        <field name="state">validate</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <record id="hr_holidays_vc_qdp" model="hr.leave.allocation">
        <field name="name">Summer Vacation</field>
        <field name="holiday_status_id" ref="holiday_status_unpaid"/>
        <field name="number_of_days">7</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_qdp'))]"/>
        <field name="state">confirm</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>
    <function model="hr.leave.allocation" name="action_validate">
        <value eval="ref('hr_holidays.hr_holidays_vc_qdp')"/>
    </function>

    <!-- leave request -->
    <record id="hr_holidays_cl_qdp" model="hr.leave">
        <field name="name">Sick day</field>
        <field name="holiday_status_id" ref="holiday_status_sl"/>
        <field eval="(datetime.now()+relativedelta(months=1, day=3, weekday=0)).strftime('%Y-%m-%d 01:00:00')" name="date_from"/>
        <field eval="(datetime.now()+relativedelta(months=1, day=3, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 23:00:00')" name="date_to"/>
        <field eval="(datetime.now()+relativedelta(months=1, day=3, weekday=0)).strftime('%Y-%m-%d 01:00:00')" name="request_date_from"/>
        <field eval="(datetime.now()+relativedelta(months=1, day=3, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 23:00:00')" name="request_date_to"/>
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
        <field eval="(datetime.now() + relativedelta(day=1, weekday=0)).strftime('%Y-%m-%d 01:00:00')" name="date_from"/>
        <field eval="(datetime.now() + relativedelta(day=1, weekday=0) + relativedelta(days=2)).strftime('%Y-%m-%d 23:00:00')" name="date_to"/>
        <field eval="(datetime.now() + relativedelta(day=1, weekday=0)).strftime('%Y-%m-%d 01:00:00')" name="request_date_from"/>
        <field eval="(datetime.now() + relativedelta(day=1, weekday=0) + relativedelta(days=2)).strftime('%Y-%m-%d 23:00:00')" name="request_date_to"/>
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
        <field name="state">validate</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <record id="hr_holidays_vc_fpi" model="hr.leave.allocation">
        <field name="name">Summer Vacation</field>
        <field name="holiday_status_id" ref="holiday_status_unpaid"/>
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
        <field name="state">validate</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>

    <record id="hr_holidays_vc_vad" model="hr.leave.allocation">
        <field name="name">Summer Vacation</field>
        <field name="holiday_status_id" ref="holiday_status_unpaid"/>
        <field name="number_of_days">5</field>
        <field name="employee_id" ref="hr.employee_niv"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_niv'))]"/>
        <field name="state">confirm</field>
        <field name="date_from" eval="time.strftime('%Y-1-1')"/>
        <field name="date_to" eval="time.strftime('%Y-12-31')"/>
    </record>
    <function model="hr.leave.allocation" name="action_validate">
        <value eval="ref('hr_holidays.hr_holidays_vc_vad')"/>
    </function>


    <record id="hr_holidays_cl_vad" model="hr.leave">
        <field name="name">Trip to London</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field eval="time.strftime('%Y-%m-09')" name="date_from"/>
        <field eval="time.strftime('%Y-%m-16')" name="date_to"/>
        <field eval="time.strftime('%Y-%m-09')" name="request_date_from"/>
        <field eval="time.strftime('%Y-%m-16')" name="request_date_to"/>
        <field name="employee_id" ref="hr.employee_niv"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_niv'))]"/>
        <field name="state">confirm</field>
    </record>

    <record id="hr_holidays_sl_vad" model="hr.leave">
        <field name="name">Doctor Appointment</field>
        <field name="holiday_status_id" ref="holiday_status_sl"/>
        <field eval="(datetime.now() + relativedelta(day=25, weekday=0)).strftime('%Y-%m-%d 01:00:00')" name="date_from"/>
        <field eval="(datetime.now() + relativedelta(day=25, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 23:00:00')" name="date_to"/>
        <field eval="(datetime.now() + relativedelta(day=25, weekday=0)).strftime('%Y-%m-%d 01:00:00')" name="request_date_from"/>
        <field eval="(datetime.now() + relativedelta(day=25, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 23:00:00')" name="request_date_to"/>
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
        <field name="name">Summer Vacation</field>
        <field name="holiday_status_id" ref="holiday_status_unpaid"/>
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
        <field eval="(datetime.now()+relativedelta(months=1, day=1, weekday=0)).strftime('%Y-%m-%d 01:00:00')" name="date_from"/>
        <field eval="(datetime.now()+relativedelta(months=1, day=1, weekday=0)).strftime('%Y-%m-%d 23:00:00')" name="date_to"/>
        <field eval="(datetime.now()+relativedelta(months=1, day=1, weekday=0)).strftime('%Y-%m-%d 01:00:00')" name="request_date_from"/>
        <field eval="(datetime.now()+relativedelta(months=1, day=1, weekday=0)).strftime('%Y-%m-%d 23:00:00')" name="request_date_to"/>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_jve'))]"/>
        <field name="state">confirm</field>
    </record>

    <record id="hr_holidays_sl_kim_2" model="hr.leave">
        <field name="name">Second dentist appointment</field>
        <field name="holiday_status_id" ref="holiday_status_sl"/>
        <field eval="(datetime.now()+relativedelta(months=4, day=1, weekday=2)).strftime('%Y-%m-%d 01:00:00')" name="date_from"/>
        <field eval="(datetime.now()+relativedelta(months=4, day=1, weekday=2)).strftime('%Y-%m-%d 23:00:00')" name="date_to"/>
        <field eval="(datetime.now()+relativedelta(months=4, day=1, weekday=2)).strftime('%Y-%m-%d 01:00:00')" name="request_date_from"/>
        <field eval="(datetime.now()+relativedelta(months=4, day=1, weekday=2)).strftime('%Y-%m-%d 23:00:00')" name="request_date_to"/>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="employee_ids" eval="[(4, ref('hr.employee_jve'))]"/>
        <field name="state">confirm</field>
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
</odoo>

```

## File: data\mail_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Leave specific activities -->
        <record id="mail_act_leave_approval" model="mail.activity.type">
            <field name="name">Time Off Approval</field>
            <field name="icon">fa-sun-o</field>
            <field name="res_model">hr.leave</field>
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
        <record id="mail_act_leave_allocation_second_approval" model="mail.activity.type">
            <field name="name">Allocation Second Approve</field>
            <field name="icon">fa-sun-o</field>
            <field name="res_model">hr.leave.allocation</field>
        </record>

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

        leave_data = Requests.read_group(
            [('department_id', 'in', self.ids),
             ('state', '=', 'confirm')],
            ['department_id'], ['department_id'])
        allocation_data = Allocations.read_group(
            [('department_id', 'in', self.ids),
             ('state', '=', 'confirm')],
            ['department_id'], ['department_id'])
        absence_data = Requests.read_group(
            [('department_id', 'in', self.ids), ('state', 'not in', ['cancel', 'refuse']),
             ('date_from', '<=', today_end), ('date_to', '>=', today_start)],
            ['department_id'], ['department_id'])

        res_leave = dict((data['department_id'][0], data['department_id_count']) for data in leave_data)
        res_allocation = dict((data['department_id'][0], data['department_id_count']) for data in allocation_data)
        res_absence = dict((data['department_id'][0], data['department_id_count']) for data in absence_data)

        for department in self:
            department.leave_to_approve_count = res_leave.get(department.id, 0)
            department.allocation_to_approve_count = res_allocation.get(department.id, 0)
            department.absence_of_today = res_absence.get(department.id, 0)

    def action_open_leave_department(self):
        action = self.env["ir.actions.actions"]._for_xml_id("hr_holidays.hr_leave_action_action_approve_department")
        action['context'] = {
            'searchpanel_default_department_id': self.id,
            'default_department_id': self.id,
            **ast.literal_eval(action['context'])
        }
        return action

    def action_open_allocation_department(self):
        action = self.env["ir.actions.actions"]._for_xml_id("hr_holidays.hr_leave_allocation_action_approve_department")
        action['context'] = {
            'searchpanel_default_department_id': self.id,
            'default_department_id': self.id,
            **ast.literal_eval(action['context'])
        }
        action['domain'] = expression.AND([ast.literal_eval(action['domain']), [('state', '=', 'confirm')]])
        return action

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime
from dateutil.relativedelta import relativedelta

from odoo import _, api, fields, models
from odoo.exceptions import UserError
from odoo.tools.float_utils import float_round


class HrEmployeeBase(models.AbstractModel):
    _inherit = "hr.employee.base"

    leave_manager_id = fields.Many2one(
        'res.users', string='Time Off',
        compute='_compute_leave_manager', store=True, readonly=False,
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
    allocation_used_count = fields.Float('Total number of days off used', compute='_compute_total_allocation_used')
    show_leaves = fields.Boolean('Able to see Remaining Time Off', compute='_compute_show_leaves')
    is_absent = fields.Boolean('Absent Today', compute='_compute_leave_status', search='_search_absent_employee')
    allocation_display = fields.Char(compute='_compute_allocation_count')
    allocation_used_display = fields.Char(compute='_compute_total_allocation_used')
    hr_icon_display = fields.Selection(selection_add=[('presence_holiday_absent', 'On leave'),
                                                      ('presence_holiday_present', 'Present but on leave')])

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
        return dict((row['employee_id'], row['days']) for row in self._cr.dictfetchall())

    def _compute_remaining_leaves(self):
        remaining = {}
        if self.ids:
            remaining = self._get_remaining_leaves()
        for employee in self:
            value = float_round(remaining.get(employee.id, 0.0), precision_digits=2)
            employee.leaves_count = value
            employee.remaining_leaves = value

    def _compute_allocation_count(self):
        data = self.env['hr.leave.allocation'].read_group([
            ('employee_id', 'in', self.ids),
            ('holiday_status_id.active', '=', True),
            ('state', '=', 'validate'),
        ], ['number_of_days:sum', 'employee_id'], ['employee_id'])
        rg_results = dict((d['employee_id'][0], {"employee_id_count": d['employee_id_count'], "number_of_days": d['number_of_days']}) for d in data)
        for employee in self:
            result = rg_results.get(employee.id)
            employee.allocation_count = float_round(result['number_of_days'], precision_digits=2) if result else 0.0
            employee.allocation_display = "%g" % employee.allocation_count
            employee.allocations_count = result['employee_id_count'] if result else 0.0

    def _compute_total_allocation_used(self):
        for employee in self:
            employee.allocation_used_count = float_round(employee.allocation_count - employee.remaining_leaves, precision_digits=2)
            employee.allocation_used_display = "%g" % employee.allocation_used_count

    def _compute_presence_state(self):
        super()._compute_presence_state()
        employees = self.filtered(lambda employee: employee.hr_presence_state != 'present' and employee.is_absent)
        employees.update({'hr_presence_state': 'absent'})

    def _compute_presence_icon(self):
        super()._compute_presence_icon()
        employees_absent = self.filtered(lambda employee:
                                         employee.hr_icon_display not in ['presence_present', 'presence_absent_active']
                                         and employee.is_absent)
        employees_absent.update({'hr_icon_display': 'presence_holiday_absent'})
        employees_present = self.filtered(lambda employee:
                                          employee.hr_icon_display in ['presence_present', 'presence_absent_active']
                                          and employee.is_absent)
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
            employee.is_absent = leave_data.get(employee.id) and leave_data.get(employee.id, {}).get('current_leave_state') in ['validate']

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
        today_date = datetime.datetime.utcnow().date()
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

    @api.model
    def create(self, values):
        if 'parent_id' in values:
            manager = self.env['hr.employee'].browse(values['parent_id']).user_id
            values['leave_manager_id'] = values.get('leave_manager_id', manager.id)
        if values.get('leave_manager_id', False):
            approver_group = self.env.ref('hr_holidays.group_hr_holidays_responsible', raise_if_not_found=False)
            if approver_group:
                approver_group.sudo().write({'users': [(4, values['leave_manager_id'])]})
        return super(HrEmployeeBase, self).create(values)

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
                old_managers -= self.env['res.users'].browse(values['leave_manager_id'])
                approver_group = self.env.ref('hr_holidays.group_hr_holidays_responsible', raise_if_not_found=False)
                if approver_group:
                    approver_group.sudo().write({'users': [(4, values['leave_manager_id'])]})

        res = super(HrEmployeeBase, self).write(values)
        # remove users from the Responsible group if they are no longer leave managers
        old_managers._clean_leave_responsible_users()

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
            ('state', '=', 'validate')
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

```

## File: models\hr_leave.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2005-2006 Axelor SARL. (http://www.axelor.com)

import logging
import pytz

from collections import namedtuple, defaultdict

from datetime import datetime, timedelta, time
from pytz import timezone, UTC

from odoo import api, fields, models, tools
from odoo.addons.base.models.res_partner import _tz_get
from odoo.addons.resource.models.resource import float_to_time, HOURS_PER_DAY
from odoo.exceptions import AccessError, UserError, ValidationError
from odoo.tools import float_compare, format_date
from odoo.tools.float_utils import float_round
from odoo.tools.translate import _
from odoo.osv import expression

_logger = logging.getLogger(__name__)

# Used to agglomerate the attendances in order to find the hour_from and hour_to
# See _compute_date_from_to
DummyAttendance = namedtuple('DummyAttendance', 'hour_from, hour_to, dayofweek, day_period, week_type')

class HolidaysRequest(models.Model):
    """ Leave Requests Access specifications

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
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _mail_post_access = 'read'

    @api.model
    def default_get(self, fields_list):
        defaults = super(HolidaysRequest, self).default_get(fields_list)
        defaults = self._default_get_request_parameters(defaults)

        employee = self.env['hr.employee'].browse(defaults['employee_id']) if defaults.get('employee_id') else self.env.user.employee_id
        # In some cases, the context does not contain the employee's working time for the day requested
        if defaults.get('request_date_from') and defaults.get('request_date_to'):
            default_attendance_from, default_attendance_to = self._get_attendances(employee, defaults['request_date_from'], defaults['request_date_to'])
            defaults['date_from'] = self._get_start_or_end_from_attendance(default_attendance_from.hour_from, defaults['request_date_from'], employee)
            defaults['date_to'] = self._get_start_or_end_from_attendance(default_attendance_to.hour_to, defaults['request_date_to'], employee)

        if 'holiday_status_id' in fields_list and not defaults.get('holiday_status_id'):
            lt = self.env['hr.leave.type'].search(['|', ('requires_allocation', '=', 'no'), ('has_valid_allocation', '=', True)], limit=1)

            if lt:
                defaults['holiday_status_id'] = lt.id
                defaults['request_unit_custom'] = False

        if 'state' in fields_list and not defaults.get('state'):
            lt = self.env['hr.leave.type'].browse(defaults.get('holiday_status_id'))
            defaults['state'] = 'confirm' if lt and lt.leave_validation_type != 'no_validation' else 'draft'

        now = fields.Datetime.now()
        if 'date_from' not in defaults:
            defaults.update({'date_from': now})
        if 'date_to' not in defaults:
            defaults.update({'date_to': now})
        default_start_time = self._get_start_or_end_from_attendance(7, datetime.now().date(), employee).time()
        default_end_time = self._get_start_or_end_from_attendance(19, datetime.now().date(), employee).time()
        if defaults['date_from'].time() == default_start_time and defaults['date_to'].time() == default_end_time:
            date_from = defaults['date_from'].date()
            date_to = defaults['date_to'].date()
            attendance_from, attendance_to = self._get_attendances(employee, date_from, date_to)
            defaults['date_from'] = self._get_start_or_end_from_attendance(attendance_from.hour_from, date_from, employee)
            defaults['date_to'] = self._get_start_or_end_from_attendance(attendance_to.hour_to, date_to, employee)

        return defaults

    def _get_start_or_end_from_attendance(self, hour, date, employee):
        hour = float_to_time(float(hour))
        holiday_tz = timezone(employee.tz or self.env.user.tz or 'utc')
        return holiday_tz.localize(datetime.combine(date, hour)).astimezone(UTC).replace(tzinfo=None)

    def _get_attendances(self, employee, request_date_from, request_date_to):
        resource_calendar_id = employee.resource_calendar_id or self.env.company.resource_calendar_id
        domain = [('calendar_id', '=', resource_calendar_id.id), ('display_type', '=', False)]
        attendances = self.env['resource.calendar.attendance'].read_group(domain,
            ['ids:array_agg(id)', 'hour_from:min(hour_from)', 'hour_to:max(hour_to)',
             'week_type', 'dayofweek', 'day_period'],
            ['week_type', 'dayofweek', 'day_period'], lazy=False)

        # Must be sorted by dayofweek ASC and day_period DESC
        attendances = sorted([DummyAttendance(group['hour_from'], group['hour_to'], group['dayofweek'], group['day_period'], group['week_type']) for group in attendances], key=lambda att: (att.dayofweek, att.day_period != 'morning'))

        default_value = DummyAttendance(0, 0, 0, 'morning', False)

        if resource_calendar_id.two_weeks_calendar:
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

        return attendance_from, attendance_to

    def _default_get_request_parameters(self, values):
        new_values = dict(values)
        global_from, global_to = False, False
        # TDE FIXME: consider a mapping on several days that is not the standard
        # calendar widget 7-19 in user's TZ is some custom input
        if values.get('date_from'):
            user_tz = self.env.user.tz or 'UTC'
            localized_dt = timezone('UTC').localize(values['date_from']).astimezone(timezone(user_tz))
            global_from = localized_dt.time().hour == 7 and localized_dt.time().minute == 0
            new_values['request_date_from'] = localized_dt.date()
        if values.get('date_to'):
            user_tz = self.env.user.tz or 'UTC'
            localized_dt = timezone('UTC').localize(values['date_to']).astimezone(timezone(user_tz))
            global_to = localized_dt.time().hour == 19 and localized_dt.time().minute == 0
            new_values['request_date_to'] = localized_dt.date()
        if global_from and global_to:
            new_values['request_unit_custom'] = True
        return new_values

    # description
    name = fields.Char('Description', compute='_compute_description', inverse='_inverse_description', search='_search_description', compute_sudo=False)
    private_name = fields.Char('Time Off Description', groups='hr_holidays.group_hr_holidays_user')
    state = fields.Selection([
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
    user_id = fields.Many2one('res.users', string='User', related='employee_id.user_id', related_sudo=True, compute_sudo=True, store=True, readonly=True)
    manager_id = fields.Many2one('hr.employee', compute='_compute_from_employee_id', store=True, readonly=False)
    # leave type configuration
    holiday_status_id = fields.Many2one(
        "hr.leave.type", compute='_compute_from_employee_id', store=True, string="Time Off Type", required=True, readonly=False,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]},
        domain=['|', ('requires_allocation', '=', 'no'), ('has_valid_allocation', '=', True)])
    holiday_allocation_id = fields.Many2one(
        'hr.leave.allocation', compute='_compute_from_holiday_status_id', string="Allocation", store=True, readonly=False)
    color = fields.Integer("Color", related='holiday_status_id.color')
    validation_type = fields.Selection(string='Validation Type', related='holiday_status_id.leave_validation_type', readonly=False)
    # HR data

    employee_id = fields.Many2one(
        'hr.employee', compute='_compute_from_employee_ids', store=True, string='Employee', index=True, readonly=False, ondelete="restrict",
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]},
        tracking=True, compute_sudo=False)
    employee_company_id = fields.Many2one(related='employee_id.company_id', readonly=True, store=True)
    active_employee = fields.Boolean(related='employee_id.active', readonly=True)
    tz_mismatch = fields.Boolean(compute='_compute_tz_mismatch')
    tz = fields.Selection(_tz_get, compute='_compute_tz')
    department_id = fields.Many2one(
        'hr.department', compute='_compute_department_id', store=True, string='Department', readonly=False,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]})
    notes = fields.Text('Reasons', readonly=True, states={'draft': [('readonly', False)], 'confirm': [('readonly', False)]})
    # duration
    date_from = fields.Datetime(
        'Start Date', compute='_compute_date_from_to', store=True, readonly=False, index=True, copy=False, required=True, tracking=True,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]})
    date_to = fields.Datetime(
        'End Date', compute='_compute_date_from_to', store=True, readonly=False, copy=False, required=True, tracking=True,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]})
    number_of_days = fields.Float(
        'Duration (Days)', compute='_compute_number_of_days', store=True, readonly=False, copy=False, tracking=True,
        help='Number of days of the time off request. Used in the calculation. To manually correct the duration, use this field.')
    number_of_days_display = fields.Float(
        'Duration in days', compute='_compute_number_of_days_display', readonly=True,
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
        string='Allocation Mode', readonly=True, required=True, default='employee',
        states={'draft': [('readonly', False)], 'confirm': [('readonly', False)]},
        help='By Employee: Allocation/Request for individual Employee, By Employee Tag: Allocation/Request for group of employees in category')
    employee_ids = fields.Many2many(
        'hr.employee', compute='_compute_from_holiday_type', store=True, string='Employees', readonly=False, groups="hr_holidays.group_hr_holidays_user",
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]})
    multi_employee = fields.Boolean(
        compute='_compute_from_employee_ids', store=True, compute_sudo=False,
        help='Holds whether this allocation concerns more than 1 employee')
    category_id = fields.Many2one(
        'hr.employee.category', compute='_compute_from_holiday_type', store=True, string='Employee Tag',
        states={'draft': [('readonly', False)], 'confirm': [('readonly', False)]}, help='Category of Employee')
    mode_company_id = fields.Many2one(
        'res.company', compute='_compute_from_holiday_type', store=True, string='Company Mode',
        states={'draft': [('readonly', False)], 'confirm': [('readonly', False)]})
    first_approver_id = fields.Many2one(
        'hr.employee', string='First Approval', readonly=True, copy=False,
        help='This area is automatically filled by the user who validate the time off')
    second_approver_id = fields.Many2one(
        'hr.employee', string='Second Approval', readonly=True, copy=False,
        help='This area is automatically filled by the user who validate the time off with second level (If time off type need second validation)')
    can_reset = fields.Boolean('Can reset', compute='_compute_can_reset')
    can_approve = fields.Boolean('Can Approve', compute='_compute_can_approve')

    attachment_ids = fields.One2many('ir.attachment', 'res_id', string="Attachments")
    # To display in form view
    supported_attachment_ids = fields.Many2many(
        'ir.attachment', string="Attach File", compute='_compute_supported_attachment_ids',
        inverse='_inverse_supported_attachment_ids')
    supported_attachment_ids_count = fields.Integer(compute='_compute_supported_attachment_ids')
    # UX fields
    leave_type_request_unit = fields.Selection(related='holiday_status_id.request_unit', readonly=True)
    leave_type_support_document = fields.Boolean(related="holiday_status_id.support_document")
    # Interface fields used when not using hour-based computation
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
    request_unit_custom = fields.Boolean('Days-long custom hours', compute='_compute_request_unit_custom', store=True, readonly=False)
    # view
    is_hatched = fields.Boolean('Hatched', compute='_compute_is_hatched')
    is_striked = fields.Boolean('Striked', compute='_compute_is_hatched')

    _sql_constraints = [
        ('type_value',
         "CHECK((holiday_type='employee' AND (employee_id IS NOT NULL OR multi_employee IS TRUE)) or "
         "(holiday_type='company' AND mode_company_id IS NOT NULL) or "
         "(holiday_type='category' AND category_id IS NOT NULL) or "
         "(holiday_type='department' AND department_id IS NOT NULL) )",
         "The employee, department, company or employee category of this request is missing. Please make sure that your user login is linked to an employee."),
        ('date_check2', "CHECK ((date_from <= date_to))", "The start date must be anterior to the end date."),
        ('duration_check', "CHECK ( number_of_days >= 0 )", "If you want to change the number of days you should use the 'period' mode"),
    ]

    def _auto_init(self):
        res = super(HolidaysRequest, self)._auto_init()
        tools.create_index(self._cr, 'hr_leave_date_to_date_from_index',
                           self._table, ['date_to', 'date_from'])
        return res

    @api.constrains('holiday_status_id', 'number_of_days')
    def _check_allocation_duration(self):
        # Deprecated as part of https://github.com/odoo/odoo/pull/96545
        # TODO: remove in master
        return

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

        leaves = self.search(domain)
        return [('id', 'in', leaves.ids)]

    @api.depends('holiday_status_id')
    def _compute_state(self):
        for leave in self:
            leave.state = 'confirm' if leave.validation_type != 'no_validation' else 'draft'

    @api.depends('holiday_status_id.requires_allocation', 'validation_type', 'employee_id', 'date_from', 'date_to')
    def _compute_from_holiday_status_id(self):
        # Deprecated as part of https://github.com/odoo/odoo/pull/96545
        # TODO: remove in master
        self.holiday_allocation_id = False

    @api.depends('request_date_from_period', 'request_hour_from', 'request_hour_to', 'request_date_from', 'request_date_to',
                'request_unit_half', 'request_unit_hours', 'request_unit_custom', 'employee_id')
    def _compute_date_from_to(self):
        for holiday in self:
            if holiday.request_date_from and holiday.request_date_to and holiday.request_date_from > holiday.request_date_to:
                holiday.request_date_to = holiday.request_date_from
            if not holiday.request_date_from:
                holiday.date_from = False
            elif not holiday.request_unit_half and not holiday.request_unit_hours and not holiday.request_date_to:
                holiday.date_to = False
            else:
                if holiday.request_unit_half or holiday.request_unit_hours:
                    holiday.request_date_to = holiday.request_date_from
                resource_calendar_id = holiday.employee_id.resource_calendar_id or self.env.company.resource_calendar_id
                domain = [('calendar_id', '=', resource_calendar_id.id), ('display_type', '=', False)]
                attendances = self.env['resource.calendar.attendance'].read_group(domain, ['ids:array_agg(id)', 'hour_from:min(hour_from)', 'hour_to:max(hour_to)', 'week_type', 'dayofweek', 'day_period'], ['week_type', 'dayofweek', 'day_period'], lazy=False)

                # Must be sorted by dayofweek ASC and day_period DESC
                attendances = sorted([DummyAttendance(group['hour_from'], group['hour_to'], group['dayofweek'], group['day_period'], group['week_type']) for group in attendances], key=lambda att: (att.dayofweek, att.day_period != 'morning'))

                default_value = DummyAttendance(0, 0, 0, 'morning', False)

                if resource_calendar_id.two_weeks_calendar:
                    # find week type of start_date
                    start_week_type = self.env['resource.calendar.attendance'].get_week_type(holiday.request_date_from)
                    attendance_actual_week = [att for att in attendances if att.week_type is False or int(att.week_type) == start_week_type]
                    attendance_actual_next_week = [att for att in attendances if att.week_type is False or int(att.week_type) != start_week_type]
                    # First, add days of actual week coming after date_from
                    attendance_filtred = [att for att in attendance_actual_week if int(att.dayofweek) >= holiday.request_date_from.weekday()]
                    # Second, add days of the other type of week
                    attendance_filtred += list(attendance_actual_next_week)
                    # Third, add days of actual week (to consider days that we have remove first because they coming before date_from)
                    attendance_filtred += list(attendance_actual_week)
                    end_week_type = self.env['resource.calendar.attendance'].get_week_type(holiday.request_date_to)
                    attendance_actual_week = [att for att in attendances if att.week_type is False or int(att.week_type) == end_week_type]
                    attendance_actual_next_week = [att for att in attendances if att.week_type is False or int(att.week_type) != end_week_type]
                    attendance_filtred_reversed = list(reversed([att for att in attendance_actual_week if int(att.dayofweek) <= holiday.request_date_to.weekday()]))
                    attendance_filtred_reversed += list(reversed(attendance_actual_next_week))
                    attendance_filtred_reversed += list(reversed(attendance_actual_week))

                    # find first attendance coming after first_day
                    attendance_from = attendance_filtred[0]
                    # find last attendance coming before last_day
                    attendance_to = attendance_filtred_reversed[0]
                else:
                    # find first attendance coming after first_day
                    attendance_from = next((att for att in attendances if int(att.dayofweek) >= holiday.request_date_from.weekday()), attendances[0] if attendances else default_value)
                    # find last attendance coming before last_day
                    attendance_to = next((att for att in reversed(attendances) if int(att.dayofweek) <= holiday.request_date_to.weekday()), attendances[-1] if attendances else default_value)

                compensated_request_date_from = holiday.request_date_from
                compensated_request_date_to = holiday.request_date_to

                if holiday.request_unit_half:
                    if holiday.request_date_from_period == 'am':
                        hour_from = float_to_time(attendance_from.hour_from)
                        hour_to = float_to_time(attendance_from.hour_to)
                    else:
                        hour_from = float_to_time(attendance_to.hour_from)
                        hour_to = float_to_time(attendance_to.hour_to)
                elif holiday.request_unit_hours:
                    hour_from = float_to_time(float(holiday.request_hour_from))
                    hour_to = float_to_time(float(holiday.request_hour_to))
                elif holiday.request_unit_custom:
                    hour_from = holiday.date_from.time()
                    hour_to = holiday.date_to.time()
                    compensated_request_date_from = holiday._adjust_date_based_on_tz(holiday.request_date_from, hour_from)
                    compensated_request_date_to = holiday._adjust_date_based_on_tz(holiday.request_date_to, hour_to)
                else:
                    hour_from = float_to_time(attendance_from.hour_from)
                    hour_to = float_to_time(attendance_to.hour_to)

                holiday.date_from = timezone(holiday.tz).localize(datetime.combine(compensated_request_date_from, hour_from)).astimezone(UTC).replace(tzinfo=None)
                holiday.date_to = timezone(holiday.tz).localize(datetime.combine(compensated_request_date_to, hour_to)).astimezone(UTC).replace(tzinfo=None)

    @api.depends('holiday_status_id', 'request_unit_hours', 'request_unit_custom')
    def _compute_request_unit_half(self):
        for holiday in self:
            if holiday.holiday_status_id or holiday.request_unit_hours or holiday.request_unit_custom:
                holiday.request_unit_half = False

    @api.depends('holiday_status_id', 'request_unit_half', 'request_unit_custom')
    def _compute_request_unit_hours(self):
        for holiday in self:
            if holiday.holiday_status_id or holiday.request_unit_half or holiday.request_unit_custom:
                holiday.request_unit_hours = False

    @api.depends('holiday_status_id', 'request_unit_half', 'request_unit_hours')
    def _compute_request_unit_custom(self):
        for holiday in self:
            if holiday.holiday_status_id or holiday.request_unit_half or holiday.request_unit_hours:
                holiday.request_unit_custom = False

    @api.depends('employee_ids')
    def _compute_from_employee_ids(self):
        for holiday in self:
            if len(holiday.employee_ids) == 1:
                holiday.employee_id = holiday.employee_ids[0]._origin
            else:
                holiday.employee_id = False
            holiday.multi_employee = (len(holiday.employee_ids) > 1)

    @api.depends('holiday_type')
    def _compute_from_holiday_type(self):
        for holiday in self:
            if holiday.holiday_type == 'employee':
                if not holiday.employee_ids:
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

    @api.depends('employee_id')
    def _compute_from_employee_id(self):
        for holiday in self:
            holiday.manager_id = holiday.employee_id.parent_id.id
            if holiday.employee_id.user_id != self.env.user and holiday._origin.employee_id != holiday.employee_id:
                holiday.holiday_status_id = False

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

    @api.depends('date_from', 'date_to', 'employee_id')
    def _compute_number_of_days(self):
        for holiday in self:
            if holiday.date_from and holiday.date_to:
                holiday.number_of_days = holiday._get_number_of_days(holiday.date_from, holiday.date_to, holiday.employee_id.id)['days']
            else:
                holiday.number_of_days = 0

    @api.depends('tz')
    @api.depends_context('uid')
    def _compute_tz_mismatch(self):
        for leave in self:
            leave.tz_mismatch = leave.tz != self.env.user.tz

    @api.depends('request_unit_custom', 'employee_id', 'holiday_type', 'department_id.company_id.resource_calendar_id.tz', 'mode_company_id.resource_calendar_id.tz')
    def _compute_tz(self):
        for leave in self:
            tz = False
            if leave.request_unit_custom:
                tz = 'UTC' # custom -> already in UTC
            elif leave.holiday_type == 'employee':
                tz = leave.employee_id.tz
            elif leave.holiday_type == 'department':
                tz = leave.department_id.company_id.resource_calendar_id.tz
            elif leave.holiday_type == 'company':
                tz = leave.mode_company_id.resource_calendar_id.tz
            leave.tz = tz or self.env.company.resource_calendar_id.tz or self.env.user.tz or 'UTC'

    @api.depends('number_of_days')
    def _compute_number_of_days_display(self):
        for holiday in self:
            holiday.number_of_days_display = holiday.number_of_days

    def _get_calendar(self):
        self.ensure_one()
        return self.employee_id.resource_calendar_id or self.env.company.resource_calendar_id

    @api.depends('number_of_days')
    def _compute_number_of_hours_display(self):
        for holiday in self:
            calendar = holiday._get_calendar()
            if holiday.date_from and holiday.date_to:
                # Take attendances into account, in case the leave validated
                # Otherwise, this will result into number_of_hours = 0
                # and number_of_hours_display = 0 or (#day * calendar.hours_per_day),
                # which could be wrong if the employee doesn't work the same number
                # hours each day
                if holiday.state == 'validate':
                    start_dt = holiday.date_from
                    end_dt = holiday.date_to
                    if not start_dt.tzinfo:
                        start_dt = start_dt.replace(tzinfo=UTC)
                    if not end_dt.tzinfo:
                        end_dt = end_dt.replace(tzinfo=UTC)
                    resource = holiday.employee_id.resource_id
                    intervals = calendar._attendance_intervals_batch(start_dt, end_dt, resource)[resource.id] \
                                - calendar._leave_intervals_batch(start_dt, end_dt, None)[False]  # Substract Global Leaves
                    number_of_hours = sum((stop - start).total_seconds() / 3600 for start, stop, dummy in intervals)
                else:
                    number_of_hours = holiday._get_number_of_days(holiday.date_from, holiday.date_to, holiday.employee_id.id)['hours']
                holiday.number_of_hours_display = number_of_hours or (holiday.number_of_days * (calendar.hours_per_day or HOURS_PER_DAY))
            else:
                holiday.number_of_hours_display = 0

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
        for holiday in self.filtered('employee_id'):
            domain = [
                ('date_from', '<', holiday.date_to),
                ('date_to', '>', holiday.date_from),
                ('employee_id', '=', holiday.employee_id.id),
                ('id', '!=', holiday.id),
                ('state', 'not in', ['cancel', 'refuse']),
            ]
            nholidays = self.search_count(domain)
            if nholidays:
                raise ValidationError(
                    _('You can not set 2 time off that overlaps on the same day for the same employee.') + '\n- %s' % (holiday.display_name))

    @api.constrains('state', 'number_of_days', 'holiday_status_id')
    def _check_holidays(self):
        mapped_days = self.holiday_status_id.get_employees_days((self.employee_id | self.sudo().employee_ids).ids)
        for holiday in self:
            if holiday.holiday_type != 'employee'\
                    or not holiday.employee_id and not holiday.employee_ids\
                    or holiday.holiday_status_id.requires_allocation == 'no':
                continue
            if holiday.employee_id:
                leave_days = mapped_days[holiday.employee_id.id][holiday.holiday_status_id.id]
                if float_compare(leave_days['remaining_leaves'], 0, precision_digits=2) == -1\
                        or float_compare(leave_days['virtual_remaining_leaves'], 0, precision_digits=2) == -1:
                    raise ValidationError(_('The number of remaining time off is not sufficient for this time off type.\n'
                                            'Please also check the time off waiting for validation.'))
            else:
                unallocated_employees = []
                for employee in holiday.employee_ids:
                    leave_days = mapped_days[employee.id][holiday.holiday_status_id.id]
                    if float_compare(leave_days['remaining_leaves'], holiday.number_of_days, precision_digits=2) == -1\
                            or float_compare(leave_days['virtual_remaining_leaves'], holiday.number_of_days, precision_digits=2) == -1:
                        unallocated_employees.append(employee.name)
                if unallocated_employees:
                    raise ValidationError(_('The number of remaining time off is not sufficient for this time off type.\n'
                                            'Please also check the time off waiting for validation.')
                                        + _('\nThe employees that lack allocation days are:\n%s',
                                            (', '.join(unallocated_employees))))

    @api.constrains('date_from', 'date_to', 'employee_id')
    def _check_date_state(self):
        if self.env.context.get('leave_skip_state_check'):
            return
        for holiday in self:
            if holiday.state in ['cancel', 'refuse', 'validate1', 'validate']:
                raise ValidationError(_("This modification is not allowed in the current state."))

    def _get_number_of_days(self, date_from, date_to, employee_id):
        """ Returns a float equals to the timedelta between two dates given as string."""
        if employee_id:
            employee = self.env['hr.employee'].browse(employee_id)
            # We force the company in the domain as we are more than likely in a compute_sudo
            domain = [('time_type', '=', 'leave'),
                      ('company_id', '=', employee.company_id.id)]
            result = employee._get_work_days_data_batch(date_from, date_to, domain=domain)[employee.id]
            if self.request_unit_half and result['hours'] > 0:
                result['days'] = 0.5
            return result

        today_hours = self.env.company.resource_calendar_id.get_work_hours_count(
            datetime.combine(date_from.date(), time.min),
            datetime.combine(date_from.date(), time.max),
            False)

        hours = self.env.company.resource_calendar_id.get_work_hours_count(date_from, date_to)
        days = hours / (today_hours or HOURS_PER_DAY) if not self.request_unit_half else 0.5
        return {'days': days, 'hours': hours}

    def _adjust_date_based_on_tz(self, leave_date, hour):
        """ request_date_{from,to} are local to the user's tz but hour_{from,to} are in UTC.

        In some cases they are combined (assuming they are in the same tz) as a datetime. When
        that happens it's possible we need to adjust one of the dates. This function adjust the
        date, so that it can be passed to datetime().

        E.g. a leave in America/Los_Angeles for one day:
        - request_date_from: 1st of Jan
        - request_date_to:   1st of Jan
        - hour_from:         15:00 (7:00 local)
        - hour_to:           03:00 (19:00 local) <-- this happens on the 2nd of Jan in UTC
        """
        user_tz = timezone(self.env.user.tz if self.env.user.tz else 'UTC')
        request_date_to_utc = UTC.localize(datetime.combine(leave_date, hour)).astimezone(user_tz).replace(tzinfo=None)
        if request_date_to_utc.date() < leave_date:
            return leave_date + timedelta(days=1)
        elif request_date_to_utc.date() > leave_date:
            return leave_date - timedelta(days=1)
        else:
            return leave_date

    ####################################################
    # ORM Overrides methods
    ####################################################

    def name_get(self):
        res = []
        for leave in self:
            user_tz = timezone(leave.tz)
            date_from_utc = leave.date_from and leave.date_from.astimezone(user_tz).date()
            date_to_utc = leave.date_to and leave.date_to.astimezone(user_tz).date()
            if self.env.context.get('short_name'):
                if leave.leave_type_request_unit == 'hour':
                    res.append((leave.id, _("%s : %.2f hours") % (leave.name or leave.holiday_status_id.name, leave.number_of_hours_display)))
                else:
                    res.append((leave.id, _("%s : %.2f days") % (leave.name or leave.holiday_status_id.name, leave.number_of_days)))
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
                        res.append((
                            leave.id,
                            _("%(person)s on %(leave_type)s: %(duration).2f hours on %(date)s",
                                person=target,
                                leave_type=leave.holiday_status_id.name,
                                duration=leave.number_of_hours_display,
                                date=display_date,
                            )
                        ))
                    else:
                        res.append((
                            leave.id,
                            _("%(person)s on %(leave_type)s: %(duration).2f hours on %(date)s",
                                person=target,
                                leave_type=leave.holiday_status_id.name,
                                duration=leave.number_of_hours_display,
                                date=display_date,
                            )
                        ))
                else:
                    if leave.number_of_days > 1 and date_from_utc and date_to_utc:
                        display_date += ' / %s' % format_date(self.env, date_to_utc) or ""
                    if self.env.context.get('hide_employee_name') and 'employee_id' in self.env.context.get('group_by', []):
                        res.append((
                            leave.id,
                            _("%(leave_type)s: %(duration).2f days (%(start)s)",
                                leave_type=leave.holiday_status_id.name,
                                duration=leave.number_of_days,
                                start=display_date,
                            )
                        ))
                    else:
                        res.append((
                            leave.id,
                            _("%(person)s on %(leave_type)s: %(duration).2f days (%(start)s)",
                                person=target,
                                leave_type=leave.holiday_status_id.name,
                                duration=leave.number_of_days,
                                start=display_date,
                            )
                        ))
        return res

    def add_follower(self, employee_id):
        employee = self.env['hr.employee'].browse(employee_id)
        if employee.user_id:
            self.message_subscribe(partner_ids=employee.user_id.partner_id.ids)

    @api.constrains('holiday_allocation_id')
    def _check_allocation_id(self):
        # Deprecated as part of https://github.com/odoo/odoo/pull/96545
        # TODO: remove in master
        return

    @api.constrains('holiday_allocation_id', 'date_to', 'date_from')
    def _check_leave_type_validity(self):
        # Deprecated as part of https://github.com/odoo/odoo/pull/96545
        # TODO: remove in master
        return

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
        """ Override to avoid automatic logging of creation """
        if not self._context.get('leave_fast_create'):
            leave_types = self.env['hr.leave.type'].browse([values.get('holiday_status_id') for values in vals_list if values.get('holiday_status_id')])
            mapped_validation_type = {leave_type.id: leave_type.leave_validation_type for leave_type in leave_types}

            for values in vals_list:
                employee_id = values.get('employee_id', False)
                leave_type_id = values.get('holiday_status_id')
                # Handle automatic department_id
                if not values.get('department_id'):
                    values.update({'department_id': self.env['hr.employee'].browse(employee_id).department_id.id})

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
                    holiday_sudo.message_subscribe(partner_ids=[holiday._get_responsible_for_approval().partner_id.id])
                    holiday_sudo.message_post(body=_("The time off has been automatically approved"), subtype_xmlid="mail.mt_comment") # Message from OdooBot (sudo)
                elif not self._context.get('import_file'):
                    holiday_sudo.activity_update()
        return holidays

    def write(self, values):
        is_officer = self.env.user.has_group('hr_holidays.group_hr_holidays_user') or self.env.is_superuser()
        if not is_officer and values.keys() - {'attachment_ids', 'supported_attachment_ids', 'message_main_attachment_id'}:
            if any(hol.date_from.date() < fields.Date.today() and hol.employee_id.leave_manager_id != self.env.user for hol in self):
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
        if not self.env.context.get('leave_fast_create'):
            for holiday in self:
                if employee_id:
                    holiday.add_follower(employee_id)

        return result

    @api.ondelete(at_uninstall=False)
    def _unlink_if_correct_states(self):
        error_message = _('You cannot delete a time off which is in %s state')
        state_description_values = {elem[0]: elem[1] for elem in self._fields['state']._description_selection(self.env)}
        now = fields.Datetime.now()

        if not self.user_has_groups('hr_holidays.group_hr_holidays_user'):
            if any(hol.state not in ['draft', 'confirm'] for hol in self):
                raise UserError(error_message % state_description_values.get(self[:1].state))
            if any(hol.date_from < now for hol in self):
                raise UserError(_('You cannot delete a time off which is in the past'))
        else:
            for holiday in self.filtered(lambda holiday: holiday.state not in ['draft', 'cancel', 'confirm']):
                raise UserError(error_message % (state_description_values.get(holiday.state),))

    def unlink(self):
        return super(HolidaysRequest, self.with_context(leave_skip_date_check=True)).unlink()

    def copy_data(self, default=None):
        if default and 'date_from' in default and 'date_to' in default:
            default['request_date_from'] = default.get('date_from')
            default['request_date_to'] = default.get('date_to')
            return super().copy_data(default)
        elif self.state in {"cancel", "refuse"}:  # No overlap constraint in these cases
            return super().copy_data(default)
        raise UserError(_('A time off cannot be duplicated.'))

    def _get_mail_redirect_suggested_company(self):
        return self.holiday_status_id.company_id

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        if not self.user_has_groups('hr_holidays.group_hr_holidays_user') and 'private_name' in groupby:
            raise UserError(_('Such grouping is not allowed.'))
        return super(HolidaysRequest, self).read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)

    ####################################################
    # Business methods
    ####################################################

    def _prepare_resource_leave_vals_list(self):
        """Hook method for others to inject data
        """
        return [{
            'name': _("%s: Time Off", leave.employee_id.name),
            'date_from': leave.date_from,
            'holiday_id': leave.id,
            'date_to': leave.date_to,
            'resource_id': leave.employee_id.resource_id.id,
            'calendar_id': leave.employee_id.resource_calendar_id.id,
            'time_type': leave.holiday_status_id.time_type,
        } for leave in self]

    def _create_resource_leave(self):
        """ This method will create entry in resource calendar time off object at the time of holidays validated
        :returns: created `resource.calendar.leaves`
        """
        vals_list = self._prepare_resource_leave_vals_list()
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

    def _prepare_holidays_meeting_values(self):
        result = defaultdict(list)
        company_calendar = self.env.company.resource_calendar_id
        for holiday in self:
            calendar = holiday.employee_id.resource_calendar_id or company_calendar
            user = holiday.user_id
            if holiday.leave_type_request_unit == 'hour':
                meeting_name = _("%s on Time Off : %.2f hour(s)") % (holiday.employee_id.name or holiday.category_id.name, holiday.number_of_hours_display)
            else:
                meeting_name = _("%s on Time Off : %.2f day(s)") % (holiday.employee_id.name or holiday.category_id.name, holiday.number_of_days)
            meeting_values = {
                'name': meeting_name,
                'duration': holiday.number_of_days * (calendar.hours_per_day or HOURS_PER_DAY),
                'description': holiday.notes,
                'user_id': user.id,
                'start': holiday.date_from,
                'stop': holiday.date_to,
                'allday': False,
                'privacy': 'confidential',
                'event_tz': user.tz,
                'activity_ids': [(5, 0, 0)],
                'res_id': holiday.id,
            }
            # Add the partner_id (if exist) as an attendee
            if user and user.partner_id:
                meeting_values['partner_ids'] = [
                    (4, user.partner_id.id)]
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
            'notes': self.notes,
            'number_of_days': work_days_data[employee.id]['days'],
            'parent_id': self.id,
            'employee_id': employee.id,
            'employee_ids': employee,
            'state': 'validate',
        } for employee in employees if work_days_data[employee.id]['days']]

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

    def action_approve(self):
        # if validation_type == 'both': this method is the first approval approval
        # if validation_type != 'both': this method calls action_validate() below
        if any(holiday.state != 'confirm' for holiday in self):
            raise UserError(_('Time off request must be confirmed ("To Approve") in order to approve it.'))

        current_employee = self.env.user.employee_id
        self.filtered(lambda hol: hol.validation_type == 'both').write({'state': 'validate1', 'first_approver_id': current_employee.id})


        # Post a second message, more verbose than the tracking message
        for holiday in self.filtered(lambda holiday: holiday.employee_id.user_id):
            user_tz = timezone(holiday.tz)
            utc_tz = pytz.utc.localize(holiday.date_from).astimezone(user_tz)
            holiday.message_post(
                body=_(
                    'Your %(leave_type)s planned on %(date)s has been accepted',
                    leave_type=holiday.holiday_status_id.display_name,
                    date=utc_tz.replace(tzinfo=None)
                ),
                partner_ids=holiday.employee_id.user_id.partner_id.ids)

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

                    # keep track of conflicting leaves states before refusal
                    target_states = {l.id: l.state for l in conflicting_leaves}
                    conflicting_leaves.action_refuse()
                    split_leaves_vals = []
                    for conflicting_leave in conflicting_leaves:
                        if conflicting_leave.leave_type_request_unit == 'half_day' and conflicting_leave.request_unit_half:
                            continue

                        # Leaves in days
                        if conflicting_leave.date_from < leave.date_from:
                            before_leave_vals = conflicting_leave.copy_data({
                                'date_from': conflicting_leave.date_from.date(),
                                'date_to': leave.date_from.date() + timedelta(days=-1),
                                'state': target_states[conflicting_leave.id],
                            })[0]
                            before_leave = self.env['hr.leave'].new(before_leave_vals)
                            before_leave._compute_date_from_to()

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
                            if before_leave.date_from < before_leave.date_to:
                                split_leaves_vals.append(before_leave._convert_to_write(before_leave._cache))
                        if conflicting_leave.date_to > leave.date_to:
                            after_leave_vals = conflicting_leave.copy_data({
                                'date_from': leave.date_to.date() + timedelta(days=1),
                                'date_to': conflicting_leave.date_to.date(),
                                'state': target_states[conflicting_leave.id],
                            })[0]
                            after_leave = self.env['hr.leave'].new(after_leave_vals)
                            after_leave._compute_date_from_to()
                            # Could happen for part-time contract, that time off is not necessary
                            # anymore.
                            if after_leave.date_from < after_leave.date_to:
                                split_leaves_vals.append(after_leave._convert_to_write(after_leave._cache))

                    split_leaves = self.env['hr.leave'].with_context(
                        tracking_disable=True,
                        mail_activity_automation_skip=True,
                        leave_fast_create=True,
                        leave_skip_state_check=True
                    ).create(split_leaves_vals)

                    split_leaves.filtered(lambda l: l.state in 'validate')._validate_leave_request()

                values = leave._prepare_employees_holiday_values(employees)
                leaves = self.env['hr.leave'].with_context(
                    tracking_disable=True,
                    mail_activity_automation_skip=True,
                    leave_fast_create=True,
                    no_calendar_sync=True,
                    leave_skip_state_check=True,
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

    def action_documents(self):
        domain = [('id', 'in', self.attachment_ids.ids)]
        return {
            'name': _("Supporting Documents"),
            'type': 'ir.actions.act_window',
            'res_model': 'ir.attachment',
            'context': {'create': False},
            'view_mode': 'list',
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
                    if val_type == 'no_validation' and current_employee == holiday.employee_id:
                        continue
                    # use ir.rule based first access check: department, members, ... (see security.xml)
                    holiday.check_access_rule('write')

                    # This handles states validate1 validate and refuse
                    if holiday.employee_id == current_employee:
                        raise UserError(_('Only a Time Off Manager can approve/refuse its own requests.'))

                    if (state == 'validate1' and val_type == 'both') and holiday.holiday_type == 'employee':
                        if not is_officer and self.env.user != holiday.employee_id.leave_manager_id:
                            raise UserError(_('You must be either %s\'s manager or Time off Manager to approve this leave') % (holiday.employee_id.name))

                    if (state == 'validate' and val_type == 'manager') and self.env.user != (holiday.employee_id | holiday.sudo().employee_ids).leave_manager_id:
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

        responsible = self.env.user

        if self.holiday_type != 'employee':
            return responsible

        if self.validation_type == 'manager' or (self.validation_type == 'both' and self.state == 'confirm'):
            if self.employee_id.leave_manager_id:
                responsible = self.employee_id.leave_manager_id
            elif self.employee_id.parent_id.user_id:
                responsible = self.employee_id.parent_id.user_id
        elif self.validation_type == 'hr' or (self.validation_type == 'both' and self.state == 'validate1'):
            if self.holiday_status_id.responsible_id:
                responsible = self.holiday_status_id.responsible_id

        return responsible

    def activity_update(self):
        to_clean, to_do = self.env['hr.leave'], self.env['hr.leave']
        for holiday in self:
            note = _(
                'New %(leave_type)s Request created by %(user)s',
                leave_type=holiday.holiday_status_id.name,
                user=holiday.create_uid.name,
            )
            if holiday.state == 'draft':
                to_clean |= holiday
            elif holiday.state == 'confirm':
                holiday.activity_schedule(
                    'hr_holidays.mail_act_leave_approval',
                    note=note,
                    user_id=holiday.sudo()._get_responsible_for_approval().id or self.env.user.id)
            elif holiday.state == 'validate1':
                holiday.activity_feedback(['hr_holidays.mail_act_leave_approval'])
                holiday.activity_schedule(
                    'hr_holidays.mail_act_leave_second_approval',
                    note=note,
                    user_id=holiday.sudo()._get_responsible_for_approval().id or self.env.user.id)
            elif holiday.state == 'validate':
                to_do |= holiday
            elif holiday.state == 'refuse':
                to_clean |= holiday
        if to_clean:
            to_clean.activity_unlink(['hr_holidays.mail_act_leave_approval', 'hr_holidays.mail_act_leave_second_approval'])
        if to_do:
            to_do.activity_feedback(['hr_holidays.mail_act_leave_approval', 'hr_holidays.mail_act_leave_second_approval'])

    ####################################################
    # Messaging methods
    ####################################################

    def _track_subtype(self, init_values):
        if 'state' in init_values and self.state == 'validate':
            leave_notif_subtype = self.holiday_status_id.leave_notif_subtype_id
            return leave_notif_subtype or self.env.ref('hr_holidays.mt_leave')
        return super(HolidaysRequest, self)._track_subtype(init_values)

    def _notify_get_groups(self, msg_vals=None):
        """ Handle HR users and officers recipients that can validate or refuse holidays
        directly from email. """
        groups = super(HolidaysRequest, self)._notify_get_groups(msg_vals=msg_vals)
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
            'group_hr_holidays_user', lambda pdata: pdata['type'] == 'user' and holiday_user_group_id in pdata['groups'], {
                'actions': hr_actions,
            })

        return [new_group] + groups

    def message_subscribe(self, partner_ids=None, subtype_ids=None):
        # due to record rule can not allow to add follower and mention on validated leave so subscribe through sudo
        if self.state in ['validate', 'validate1']:
            self.check_access_rights('read')
            self.check_access_rule('read')
            return super(HolidaysRequest, self.sudo()).message_subscribe(partner_ids=partner_ids, subtype_ids=subtype_ids)
        return super(HolidaysRequest, self).message_subscribe(partner_ids=partner_ids, subtype_ids=subtype_ids)

    @api.model
    def get_unusual_days(self, date_from, date_to=None):
        return self.env.user.employee_id.sudo(False)._get_unusual_days(date_from, date_to)

```

## File: models\hr_leave_accrual_plan.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class AccrualPlan(models.Model):
    _name = "hr.leave.accrual.plan"
    _description = "Accrual Plan"

    name = fields.Char('Name', required=True)
    time_off_type_id = fields.Many2one('hr.leave.type', string="Time Off Type",
        help="""Specify if this accrual plan can only be used with this Time Off Type.
                Leave empty if this accrual plan can be used with any Time Off Type.""")
    employees_count = fields.Integer("Employees", compute='_compute_employee_count')
    level_ids = fields.One2many('hr.leave.accrual.level', 'accrual_plan_id', copy=True)
    allocation_ids = fields.One2many('hr.leave.allocation', 'accrual_plan_id')
    transition_mode = fields.Selection([
        ('immediately', 'Immediately'),
        ('end_of_accrual', "After this accrual's period")],
        string="Level Transition", default="immediately", required=True,
        help="""Immediately: When the date corresponds to the new level, your accrual is automatically computed, granted and you switch to new level
                After this accrual's period: When the accrual is complete (a week, a month), and granted, you switch to next level if allocation date corresponds""")
    level_count = fields.Integer('Levels', compute='_compute_level_count')

    @api.depends('level_ids')
    def _compute_level_count(self):
        level_read_group = self.env['hr.leave.accrual.level'].read_group(
            [('accrual_plan_id', 'in', self.ids)],
            fields=['accrual_plan_id'],
            groupby=['accrual_plan_id'],
        )
        mapped_count = {group['accrual_plan_id'][0]: group['accrual_plan_id_count'] for group in level_read_group}
        for plan in self:
            plan.level_count = mapped_count.get(plan.id, 0)

    @api.depends('allocation_ids')
    def _compute_employee_count(self):
        allocations_read_group = self.env['hr.leave.allocation'].read_group(
            [('accrual_plan_id', 'in', self.ids)],
            ['accrual_plan_id', 'employee_count:count_distinct(employee_id)'],
            ['accrual_plan_id'],
        )
        allocations_dict = {res['accrual_plan_id'][0]: res['employee_count'] for res in allocations_read_group}
        for plan in self:
            plan.employees_count = allocations_dict.get(plan.id, 0)

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

import datetime
import calendar

from dateutil.relativedelta import relativedelta

from odoo import _, api, fields, models
from odoo.tools.date_utils import get_timedelta


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
    level = fields.Integer(compute='_compute_level', help='Level computed through the sequence.')
    accrual_plan_id = fields.Many2one('hr.leave.accrual.plan', "Accrual Plan", required=True)
    start_count = fields.Integer(
        "Start after",
        help="The accrual starts after a defined period from the allocation start date. This field defines the number of days, months or years after which accrual is used.", default="1")
    start_type = fields.Selection(
        [('day', 'day(s)'),
         ('month', 'month(s)'),
         ('year', 'year(s)')],
        default='day', string=" ", required=True,
        help="This field defines the unit of time after which the accrual starts.")
    is_based_on_worked_time = fields.Boolean("Based on worked time",
        help="Only accrue for the time worked by the employee. This is the time when the employee did not take time off.")

    # Accrue of
    added_value = fields.Float(
        "Rate", digits=(16, 5), required=True,
        help="The number of hours/days that will be incremented in the specified Time Off Type for every period")
    added_value_type = fields.Selection(
        [('days', 'Days'),
         ('hours', 'Hours')],
        default='days', required=True)
    frequency = fields.Selection([
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
    maximum_leave = fields.Float(
        'Limit to', required=False, default=100,
        help="Choose a cap for this accrual. 0 means no cap.")
    parent_id = fields.Many2one(
        'hr.leave.accrual.level', string="Previous Level",
        help="If this field is empty, this level is the first one.")
    action_with_unused_accruals = fields.Selection(
        [('postponed', 'Transferred to the next year'),
         ('lost', 'Lost')],
        string="At the end of the calendar year, unused accruals will be",
        default='postponed', required='True')

    _sql_constraints = [
        ('check_dates',
         "CHECK( (frequency = 'daily') or"
         "(week_day IS NOT NULL AND frequency = 'weekly') or "
         "(first_day > 0 AND second_day > first_day AND first_day <= 31 AND second_day <= 31 AND frequency = 'bimonthly') or "
         "(first_day > 0 AND first_day <= 31 AND frequency = 'monthly')or "
         "(first_month_day > 0 AND first_month_day <= 31 AND second_month_day > 0 AND second_month_day <= 31 AND frequency = 'biyearly') or "
         "(yearly_day > 0 AND yearly_day <= 31 AND frequency = 'yearly'))",
         "The dates you've set up aren't correct. Please check them."),
        ('start_count_check', "CHECK( start_count >= 0 )", "You can not start an accrual in the past."),
        ('added_value_greater_than_zero', 'CHECK(added_value > 0)', 'You must give a rate greater than 0 in accrual plan levels.')
    ]

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

    @api.depends('sequence', 'accrual_plan_id')
    def _compute_level(self):
        #Mapped level_ids.ids ordered by sequence per plan
        mapped_level_ids = {}
        for plan in self.accrual_plan_id:
            # We can not use .ids here because we also deal with NewIds
            mapped_level_ids[plan] = [level.id for level in plan.level_ids.sorted('sequence')]
        for level in self:
            if level.accrual_plan_id:
                level.level = mapped_level_ids[level.accrual_plan_id].index(level.id) + 1
            else:
                level.level = 1

    @api.depends('first_day', 'second_day', 'first_month_day', 'second_month_day', 'yearly_day')
    def _compute_days_display(self):
        days_select = _get_selection_days(self)
        for level in self:
            level.first_day_display = days_select[min(level.first_day - 1, 28)][0]
            level.second_day_display = days_select[min(level.second_day - 1, 28)][0]
            level.first_month_day_display = days_select[min(level.first_month_day - 1, 28)][0]
            level.second_month_day_display = days_select[min(level.second_month_day - 1, 28)][0]
            level.yearly_day_display = days_select[min(level.yearly_day - 1, 28)][0]

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
        if self.frequency == 'daily':
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
        if self.frequency == 'daily':
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

```

## File: models\hr_leave_allocation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2005-2006 Axelor SARL. (http://www.axelor.com)

from collections import defaultdict
import logging

from datetime import datetime, time, timedelta
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models
from odoo.addons.resource.models.resource import HOURS_PER_DAY
from odoo.exceptions import AccessError, UserError, ValidationError
from odoo.tools.translate import _
from odoo.tools.float_utils import float_round
from odoo.tools.date_utils import get_timedelta
from odoo.osv import expression

_logger = logging.getLogger(__name__)


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

    name = fields.Char('Description', compute='_compute_description', inverse='_inverse_description', search='_search_description', compute_sudo=False)
    name_validity = fields.Char('Description with validity', compute='_compute_description_validity')
    active = fields.Boolean(default=True)
    private_name = fields.Char('Allocation Description', groups='hr_holidays.group_hr_holidays_user')
    state = fields.Selection([
        ('draft', 'To Submit'),
        ('cancel', 'Cancelled'),
        ('confirm', 'To Approve'),
        ('refuse', 'Refused'),
        ('validate', 'Approved')
        ], string='Status', readonly=True, tracking=True, copy=False, default='draft',
        help="The status is set to 'To Submit', when an allocation request is created." +
        "\nThe status is 'To Approve', when an allocation request is confirmed by user." +
        "\nThe status is 'Refused', when an allocation request is refused by manager." +
        "\nThe status is 'Approved', when an allocation request is approved by manager.")
    date_from = fields.Date('Start Date', index=True, copy=False, default=fields.Date.context_today,
        states={'draft': [('readonly', False)], 'confirm': [('readonly', False)]}, tracking=True, required=True)
    date_to = fields.Date('End Date', copy=False, tracking=True,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]})
    holiday_status_id = fields.Many2one(
        "hr.leave.type", compute='_compute_holiday_status_id', store=True, string="Time Off Type", required=True, readonly=False,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]},
        domain=_domain_holiday_status_id,
        default=_default_holiday_status_id)
    employee_id = fields.Many2one(
        'hr.employee', compute='_compute_from_employee_ids', store=True, string='Employee', index=True, readonly=False, ondelete="restrict", tracking=True,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate': [('readonly', True)]})
    employee_company_id = fields.Many2one(related='employee_id.company_id', readonly=True, store=True)
    active_employee = fields.Boolean('Active Employee', related='employee_id.active', readonly=True)
    manager_id = fields.Many2one('hr.employee', compute='_compute_manager_id', store=True, string='Manager')
    notes = fields.Text('Reasons', readonly=True, states={'draft': [('readonly', False)], 'confirm': [('readonly', False)]})
    # duration
    number_of_days = fields.Float(
        'Number of Days', compute='_compute_from_holiday_status_id', store=True, readonly=False, tracking=True, default=1,
        help='Duration in days. Reference field to use when necessary.')
    number_of_days_display = fields.Float(
        'Duration (days)', compute='_compute_number_of_days_display',
        states={'draft': [('readonly', False)], 'confirm': [('readonly', False)]},
        help="If Accrual Allocation: Number of days allocated in addition to the ones you will get via the accrual' system.")
    number_of_hours_display = fields.Float(
        'Duration (hours)', compute='_compute_number_of_hours_display',
        help="If Accrual Allocation: Number of hours allocated in addition to the ones you will get via the accrual' system.")
    duration_display = fields.Char('Allocated (Days/Hours)', compute='_compute_duration_display',
        help="Field allowing to see the allocation duration in days or hours depending on the type_request_unit")
    # details
    parent_id = fields.Many2one('hr.leave.allocation', string='Parent')
    linked_request_ids = fields.One2many('hr.leave.allocation', 'parent_id', string='Linked Requests')
    approver_id = fields.Many2one(
        'hr.employee', string='First Approval', readonly=True, copy=False,
        help='This area is automatically filled by the user who validates the allocation')
    validation_type = fields.Selection(string='Validation Type', related='holiday_status_id.allocation_validation_type', readonly=True)
    can_reset = fields.Boolean('Can reset', compute='_compute_can_reset')
    can_approve = fields.Boolean('Can Approve', compute='_compute_can_approve')
    type_request_unit = fields.Selection(related='holiday_status_id.request_unit', readonly=True)
    # mode
    holiday_type = fields.Selection([
        ('employee', 'By Employee'),
        ('company', 'By Company'),
        ('department', 'By Department'),
        ('category', 'By Employee Tag')],
        string='Allocation Mode', readonly=True, required=True, default='employee',
        states={'draft': [('readonly', False)], 'confirm': [('readonly', False)]},
        help="Allow to create requests in batchs:\n- By Employee: for a specific employee"
             "\n- By Company: all employees of the specified company"
             "\n- By Department: all employees of the specified department"
             "\n- By Employee Tag: all employees of the specific employee group category")
    employee_ids = fields.Many2many(
        'hr.employee', compute='_compute_from_holiday_type', store=True, string='Employees', readonly=False,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate': [('readonly', True)]})
    multi_employee = fields.Boolean(
        compute='_compute_from_employee_ids', store=True,
        help='Holds whether this allocation concerns more than 1 employee')
    mode_company_id = fields.Many2one(
        'res.company', compute='_compute_from_holiday_type', store=True, string='Company Mode', readonly=False,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate': [('readonly', True)]})
    department_id = fields.Many2one(
        'hr.department', compute='_compute_department_id', store=True, string='Department',
        states={'draft': [('readonly', False)], 'confirm': [('readonly', False)]})
    category_id = fields.Many2one(
        'hr.employee.category', compute='_compute_from_holiday_type', store=True, string='Employee Tag', readonly=False,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate': [('readonly', True)]})
    # accrual configuration
    lastcall = fields.Date("Date of the last accrual allocation", readonly=True, default=fields.Date.context_today)
    nextcall = fields.Date("Date of the next accrual allocation", default=False, readonly=True)
    allocation_type = fields.Selection(
        [
            ('regular', 'Regular Allocation'),
            ('accrual', 'Accrual Allocation')
        ], string="Allocation Type", default="regular", required=True, readonly=True,
        states={'draft': [('readonly', False)], 'confirm': [('readonly', False)]})
    is_officer = fields.Boolean(compute='_compute_is_officer')
    accrual_plan_id = fields.Many2one('hr.leave.accrual.plan', compute="_compute_from_holiday_status_id", store=True, readonly=False, domain="['|', ('time_off_type_id', '=', False), ('time_off_type_id', '=', holiday_status_id)]", tracking=True)
    max_leaves = fields.Float(compute='_compute_leaves')
    leaves_taken = fields.Float(compute='_compute_leaves')
    taken_leave_ids = fields.One2many('hr.leave', 'holiday_allocation_id', domain="[('state', 'in', ['confirm', 'validate1', 'validate'])]")

    _sql_constraints = [
        ('type_value',
         "CHECK( (holiday_type='employee' AND (employee_id IS NOT NULL OR multi_employee IS TRUE)) or "
         "(holiday_type='category' AND category_id IS NOT NULL) or "
         "(holiday_type='department' AND department_id IS NOT NULL) or "
         "(holiday_type='company' AND mode_company_id IS NOT NULL))",
         "The employee, department, company or employee category of this request is missing. Please make sure that your user login is linked to an employee."),
        ('duration_check', "CHECK( ( number_of_days > 0 AND allocation_type='regular') or (allocation_type != 'regular'))", "The duration must be greater than 0."),
    ]

    # The compute does not get triggered without a depends on record creation
    # aka keep the 'useless' depends
    @api.depends_context('uid')
    @api.depends('allocation_type')
    def _compute_is_officer(self):
        self.is_officer = self.env.user.has_group("hr_holidays.group_hr_holidays_user")

    @api.depends_context('uid')
    def _compute_description(self):
        self.check_access_rights('read')
        self.check_access_rule('read')

        is_officer = self.env.user.has_group('hr_holidays.group_hr_holidays_user')

        for allocation in self:
            if is_officer or allocation.employee_id.user_id == self.env.user or allocation.employee_id.leave_manager_id == self.env.user:
                allocation.name = allocation.sudo().private_name
            else:
                allocation.name = '*****'

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

    @api.depends('name', 'date_from', 'date_to')
    def _compute_description_validity(self):
        for allocation in self:
            if allocation.date_to:
                name_validity = _("%s (from %s to %s)", allocation.name, allocation.date_from.strftime("%b %d %Y"), allocation.date_to.strftime("%b %d %Y"))
            else:
                name_validity = _("%s (from %s to No Limit)", allocation.name, allocation.date_from.strftime("%b %d %Y"))
            allocation.name_validity = name_validity

    @api.depends('employee_id', 'holiday_status_id', 'taken_leave_ids.number_of_days', 'taken_leave_ids.state')
    def _compute_leaves(self):
        employee_days_per_allocation = self.holiday_status_id._get_employees_days_per_allocation(self.employee_id.ids)
        for allocation in self:
            allocation.max_leaves = allocation.number_of_hours_display if allocation.type_request_unit == 'hour' else allocation.number_of_days
            allocation.leaves_taken = employee_days_per_allocation[allocation.employee_id.id][allocation.holiday_status_id][allocation]['leaves_taken']

    @api.depends('number_of_days')
    def _compute_number_of_days_display(self):
        for allocation in self:
            allocation.number_of_days_display = allocation.number_of_days

    @api.depends('number_of_days', 'employee_id')
    def _compute_number_of_hours_display(self):
        for allocation in self:
            if allocation.parent_id and allocation.parent_id.type_request_unit == "hour":
                allocation.number_of_hours_display = allocation.number_of_days * HOURS_PER_DAY
            elif allocation.number_of_days:
                allocation.number_of_hours_display = allocation.number_of_days * (allocation.employee_id.sudo().resource_id.calendar_id.hours_per_day or HOURS_PER_DAY)
            else:
                allocation.number_of_hours_display = 0.0

    @api.depends('number_of_hours_display', 'number_of_days_display')
    def _compute_duration_display(self):
        for allocation in self:
            allocation.duration_display = '%g %s' % (
                (float_round(allocation.number_of_hours_display, precision_digits=2)
                if allocation.type_request_unit == 'hour'
                else float_round(allocation.number_of_days_display, precision_digits=2)),
                _('hours') if allocation.type_request_unit == 'hour' else _('days'))

    @api.depends('state', 'employee_id', 'department_id')
    def _compute_can_reset(self):
        for allocation in self:
            try:
                allocation._check_approval_update('draft')
            except (AccessError, UserError):
                allocation.can_reset = False
            else:
                allocation.can_reset = True

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

    @api.depends('employee_ids')
    def _compute_from_employee_ids(self):
        for allocation in self:
            if len(allocation.employee_ids) == 1:
                allocation.employee_id = allocation.employee_ids[0]._origin
            else:
                allocation.employee_id = False
            allocation.multi_employee = (len(allocation.employee_ids) > 1)

    @api.depends('holiday_type')
    def _compute_from_holiday_type(self):
        default_employee_ids = self.env['hr.employee'].browse(self.env.context.get('default_employee_id')) or self.env.user.employee_id
        for allocation in self:
            if allocation.holiday_type == 'employee':
                if not allocation.employee_ids:
                    allocation.employee_ids = self.env.user.employee_id
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
        for holiday in self:
            if not holiday.holiday_status_id:
                if holiday.accrual_plan_id:
                    holiday.holiday_status_id = holiday.accrual_plan_id.time_off_type_id
                else:
                    if not default_holiday_status_id:  # fetch when we need it
                        default_holiday_status_id = self._default_holiday_status_id()
                    holiday.holiday_status_id = default_holiday_status_id

    @api.depends('holiday_status_id', 'allocation_type', 'number_of_hours_display', 'number_of_days_display', 'date_to')
    def _compute_from_holiday_status_id(self):
        accrual_allocations = self.filtered(lambda alloc: alloc.allocation_type == 'accrual' and not alloc.accrual_plan_id and alloc.holiday_status_id)
        accruals_dict = {}
        if accrual_allocations:
            accruals_read_group = self.env['hr.leave.accrual.plan'].read_group(
                [('time_off_type_id', 'in', accrual_allocations.holiday_status_id.ids)],
                ['time_off_type_id', 'ids:array_agg(id)'],
                ['time_off_type_id'],
            )
            accruals_dict = {res['time_off_type_id'][0]: res['ids'] for res in accruals_read_group}
        for allocation in self:
            allocation.number_of_days = allocation.number_of_days_display
            if allocation.type_request_unit == 'hour':
                allocation.number_of_days = allocation.number_of_hours_display / (allocation.employee_id.sudo().resource_calendar_id.hours_per_day or HOURS_PER_DAY)
            if allocation.accrual_plan_id.time_off_type_id.id not in (False, allocation.holiday_status_id.id):
                allocation.accrual_plan_id = False
            if allocation.allocation_type == 'accrual' and not allocation.accrual_plan_id:
                if allocation.holiday_status_id:
                    allocation.accrual_plan_id = accruals_dict.get(allocation.holiday_status_id.id, [False])[0]

    def _end_of_year_accrual(self):
        # to override in payroll
        first_day_this_year = fields.Date.today() + relativedelta(month=1, day=1)
        for allocation in self:
            current_level = allocation._get_current_accrual_plan_level_id(first_day_this_year)[0]
            if current_level and current_level.action_with_unused_accruals == 'lost':
                lastcall = current_level._get_previous_date(first_day_this_year)
                nextcall = current_level._get_next_date(first_day_this_year)
                if lastcall == first_day_this_year:
                    lastcall = current_level._get_previous_date(first_day_this_year - relativedelta(days=1))
                    nextcall = first_day_this_year
                # Allocations are lost but number_of_days should not be lower than leaves_taken
                allocation.write({'number_of_days': allocation.leaves_taken, 'lastcall': lastcall, 'nextcall': nextcall})

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

    def _process_accrual_plan_level(self, level, start_period, start_date, end_period, end_date):
        """
        Returns the added days for that level
        """
        self.ensure_one()
        if level.is_based_on_worked_time:
            start_dt = datetime.combine(start_date, datetime.min.time())
            end_dt = datetime.combine(end_date, datetime.min.time())
            worked = self.employee_id._get_work_days_data_batch(start_dt, end_dt, calendar=self.employee_id.resource_calendar_id)\
                [self.employee_id.id]['hours']
            if start_period != start_date or end_period != end_date:
                start_dt = datetime.combine(start_period, datetime.min.time())
                end_dt = datetime.combine(end_period, datetime.min.time())
                planned_worked = self.employee_id._get_work_days_data_batch(start_dt, end_dt, calendar=self.employee_id.resource_calendar_id)\
                    [self.employee_id.id]['hours']
            else:
                planned_worked = worked
            left = self.employee_id.sudo()._get_leave_days_data_batch(start_dt, end_dt,
                domain=[('time_type', '=', 'leave')])[self.employee_id.id]['hours']
            work_entry_prorata = worked / (left + planned_worked) if (left + planned_worked) else 0
            added_value = work_entry_prorata * level.added_value
        else:
            added_value = level.added_value
        # Convert time in hours to time in days in case the level is encoded in hours
        if level.added_value_type == 'hours':
            added_value = added_value / (self.employee_id.sudo().resource_id.calendar_id.hours_per_day or HOURS_PER_DAY)
        period_prorata = 1
        if (start_period != start_date or end_period != end_date) and not level.is_based_on_worked_time:
            period_days = (end_period - start_period)
            call_days = (end_date - start_date)
            period_prorata = min(1, call_days / period_days) if period_days else 1
        return added_value * period_prorata

    def _process_accrual_plans(self):
        """
        This method is part of the cron's process.
        The goal of this method is to retroactively apply accrual plan levels and progress from nextcall to today
        """
        today = fields.Date.today()
        first_allocation = _("""This allocation have already ran once, any modification won't be effective to the days allocated to the employee. If you need to change the configuration of the allocation, cancel and create a new one.""")
        for allocation in self:
            level_ids = allocation.accrual_plan_id.level_ids.sorted('sequence')
            if not level_ids:
                continue
            if not allocation.nextcall:
                first_level = level_ids[0]
                first_level_start_date = allocation.date_from + get_timedelta(first_level.start_count, first_level.start_type)
                if today < first_level_start_date:
                    # Accrual plan is not configured properly or has not started
                    continue
                allocation.lastcall = max(allocation.lastcall, first_level_start_date)
                allocation.nextcall = first_level._get_next_date(allocation.lastcall)
                if len(level_ids) > 1:
                    second_level_start_date = allocation.date_from + get_timedelta(level_ids[1].start_count, level_ids[1].start_type)
                    allocation.nextcall = min(second_level_start_date, allocation.nextcall)
                allocation._message_log(body=first_allocation)
            days_added_per_level = defaultdict(lambda: 0)
            while allocation.nextcall <= today:
                (current_level, current_level_idx) = allocation._get_current_accrual_plan_level_id(allocation.nextcall)
                current_level_maximum_leave = current_level.maximum_leave if current_level.added_value_type == "days" else current_level.maximum_leave / (allocation.employee_id.sudo().resource_id.calendar_id.hours_per_day or HOURS_PER_DAY)
                nextcall = current_level._get_next_date(allocation.nextcall)
                # Since _get_previous_date returns the given date if it corresponds to a call date
                # this will always return lastcall except possibly on the first call
                # this is used to prorate the first number of days given to the employee
                period_start = current_level._get_previous_date(allocation.lastcall)
                period_end = current_level._get_next_date(allocation.lastcall)
                # Also prorate this accrual in the event that we are passing from one level to another
                if current_level_idx < (len(level_ids) - 1) and allocation.accrual_plan_id.transition_mode == 'immediately':
                    next_level = level_ids[current_level_idx + 1]
                    current_level_last_date = allocation.date_from + get_timedelta(next_level.start_count, next_level.start_type)
                    if allocation.nextcall != current_level_last_date:
                        nextcall = min(nextcall, current_level_last_date)
                days_added_per_level[current_level] += allocation._process_accrual_plan_level(
                    current_level, period_start, allocation.lastcall, period_end, allocation.nextcall)
                if current_level_maximum_leave > 0 and sum(days_added_per_level.values()) > current_level_maximum_leave:
                    days_added_per_level[current_level] -= sum(days_added_per_level.values()) - current_level_maximum_leave
                allocation.lastcall = allocation.nextcall
                allocation.nextcall = nextcall
            if days_added_per_level:
                number_of_days_to_add = allocation.number_of_days + sum(days_added_per_level.values())
                max_allocation_days = current_level_maximum_leave + (allocation.leaves_taken if allocation.type_request_unit != "hour" else allocation.leaves_taken / (allocation.employee_id.sudo().resource_id.calendar_id.hours_per_day or HOURS_PER_DAY))
                # Let's assume the limit of the last level is the correct one
                allocation.number_of_days = min(number_of_days_to_add, max_allocation_days) if current_level_maximum_leave > 0 else number_of_days_to_add

    @api.model
    def _update_accrual(self):
        """
            Method called by the cron task in order to increment the number_of_days when
            necessary.
        """
        # Get the current date to determine the start and end of the accrual period
        today = datetime.combine(fields.Date.today(), time(0, 0, 0))
        this_year_first_day = today + relativedelta(day=1, month=1)
        end_of_year_allocations = self.search(
        [('allocation_type', '=', 'accrual'), ('state', '=', 'validate'), ('accrual_plan_id', '!=', False), ('employee_id', '!=', False),
            '|', ('date_to', '=', False), ('date_to', '>', fields.Datetime.now()), ('lastcall', '<', this_year_first_day)])
        end_of_year_allocations._end_of_year_accrual()
        end_of_year_allocations.flush()
        allocations = self.search(
        [('allocation_type', '=', 'accrual'), ('state', '=', 'validate'), ('accrual_plan_id', '!=', False), ('employee_id', '!=', False),
            '|', ('date_to', '=', False), ('date_to', '>', fields.Datetime.now()),
            '|', ('nextcall', '=', False), ('nextcall', '<=', today)])
        allocations._process_accrual_plans()

    ####################################################
    # ORM Overrides methods
    ####################################################

    def name_get(self):
        res = []
        for allocation in self:
            if allocation.holiday_type == 'company':
                target = allocation.mode_company_id.name
            elif allocation.holiday_type == 'department':
                target = allocation.department_id.name
            elif allocation.holiday_type == 'category':
                target = allocation.category_id.name
            elif allocation.employee_id:
                target = allocation.employee_id.name
            else:
                target = ', '.join(allocation.employee_ids.sudo().mapped('name'))

            res.append(
                (allocation.id,
                 _("Allocation of %(allocation_name)s : %(duration).2f %(duration_type)s to %(person)s",
                   allocation_name=allocation.holiday_status_id.sudo().name,
                   duration=allocation.number_of_hours_display if allocation.type_request_unit == 'hour' else allocation.number_of_days,
                   duration_type=_('hours') if allocation.type_request_unit == 'hour' else _('days'),
                   person=target
                ))
            )
        return res

    def add_follower(self, employee_id):
        employee = self.env['hr.employee'].browse(employee_id)
        if employee.user_id:
            self.message_subscribe(partner_ids=employee.user_id.partner_id.ids)

    @api.model_create_multi
    def create(self, vals_list):
        """ Override to avoid automatic logging of creation """
        for values in vals_list:
            employee_id = values.get('employee_id', False)
            if not values.get('department_id'):
                values.update({'department_id': self.env['hr.employee'].browse(employee_id).department_id.id})
            # default `lastcall` to `nextcall`
            if 'date_from' in values and 'lastcall' not in values:
                values['lastcall'] = values['date_from']
        holidays = super(HolidaysAllocation, self.with_context(mail_create_nosubscribe=True)).create(vals_list)
        for holiday in holidays:
            partners_to_subscribe = set()
            if holiday.employee_id.user_id:
                partners_to_subscribe.add(holiday.employee_id.user_id.partner_id.id)
            if holiday.validation_type == 'officer':
                partners_to_subscribe.add(holiday.employee_id.parent_id.user_id.partner_id.id)
                partners_to_subscribe.add(holiday.employee_id.leave_manager_id.partner_id.id)
            holiday.message_subscribe(partner_ids=tuple(partners_to_subscribe))
            if not self._context.get('import_file'):
                holiday.activity_update()
            if holiday.validation_type == 'no':
                if holiday.state == 'draft':
                    holiday.action_confirm()
                    holiday.action_validate()
        return holidays

    def write(self, values):
        if not self.env.context.get('toggle_active') and not bool(values.get('active', True)):
            if any(allocation.state not in ['draft', 'cancel', 'refuse'] for allocation in self):
                raise UserError(_('You cannot archive an allocation which is in confirm or validate state.'))
        employee_id = values.get('employee_id', False)
        if values.get('state'):
            self._check_approval_update(values['state'])
        result = super(HolidaysAllocation, self).write(values)
        self.add_follower(employee_id)
        return result

    @api.ondelete(at_uninstall=False)
    def _unlink_if_correct_states(self):
        state_description_values = {elem[0]: elem[1] for elem in self._fields['state']._description_selection(self.env)}
        for holiday in self.filtered(lambda holiday: holiday.state not in ['draft', 'cancel', 'confirm']):
            raise UserError(_('You cannot delete an allocation request which is in %s state.') % (state_description_values.get(holiday.state),))

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
        } for employee in employees]

    def action_draft(self):
        if any(holiday.state not in ['confirm', 'refuse'] for holiday in self):
            raise UserError(_('Allocation request state must be "Refused" or "To Approve" in order to be reset to Draft.'))
        self.write({
            'state': 'draft',
            'approver_id': False,
        })
        linked_requests = self.mapped('linked_request_ids')
        if linked_requests:
            linked_requests.action_draft()
            linked_requests.unlink()
        self.activity_update()
        return True

    def action_confirm(self):
        if self.filtered(lambda holiday: holiday.state != 'draft'):
            raise UserError(_('Allocation request must be in Draft state ("To Submit") in order to confirm it.'))
        res = self.write({'state': 'confirm'})
        self.activity_update()
        return res

    def action_validate(self):
        current_employee = self.env.user.employee_id
        if any(holiday.state != 'confirm' for holiday in self):
            raise UserError(_('Allocation request must be confirmed in order to approve it.'))

        self.write({
            'state': 'validate',
            'approver_id': current_employee.id
        })

        for holiday in self:
            holiday._action_validate_create_childs()
        self.activity_update()
        return True

    def _action_validate_create_childs(self):
        childs = self.env['hr.leave.allocation']
        # In the case we are in holiday_type `employee` and there is only one employee we can keep the same allocation
        # Otherwise we do need to create an allocation for all employees to have a behaviour that is in line
        # with the other holiday_type
        if self.state == 'validate' and (self.holiday_type in ['category', 'department', 'company'] or
            (self.holiday_type == 'employee' and len(self.employee_ids) > 1)):
            if self.holiday_type == 'employee':
                employees = self.employee_ids
            elif self.holiday_type == 'category':
                employees = self.category_id.employee_ids
            elif self.holiday_type == 'department':
                employees = self.department_id.member_ids
            else:
                employees = self.env['hr.employee'].search([('company_id', '=', self.mode_company_id.id)])

            allocation_create_vals = self._prepare_holiday_values(employees)
            childs += self.with_context(
                mail_notify_force_send=False,
                mail_activity_automation_skip=True
            ).create(allocation_create_vals)
            if childs:
                childs.action_validate()
        return childs

    def action_refuse(self):
        current_employee = self.env.user.employee_id
        if any(holiday.state not in ['confirm', 'validate', 'validate1'] for holiday in self):
            raise UserError(_('Allocation request must be confirmed or validated in order to refuse it.'))

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
        for holiday in self:
            val_type = holiday.holiday_status_id.sudo().allocation_validation_type
            if state == 'confirm':
                continue

            if state == 'draft':
                if holiday.employee_id != current_employee and not is_manager:
                    raise UserError(_('Only a time off Manager can reset other people allocation.'))
                continue

            if not is_officer and self.env.user != holiday.employee_id.leave_manager_id and not val_type == 'no':
                raise UserError(_('Only a time off Officer/Responsible or Manager can approve or refuse time off requests.'))

            if is_officer or self.env.user == holiday.employee_id.leave_manager_id:
                # use ir.rule based first access check: department, members, ... (see security.xml)
                holiday.check_access_rule('write')

            if holiday.employee_id == current_employee and not is_manager and not val_type == 'no':
                raise UserError(_('Only a time off Manager can approve its own requests.'))

    @api.onchange('allocation_type')
    def _onchange_allocation_type(self):
        if self.allocation_type == 'accrual':
            self.number_of_days = 0.0
        elif not self.number_of_days_display:
            self.number_of_days = 1.0

    # ------------------------------------------------------------
    # Activity methods
    # ------------------------------------------------------------

    def _get_responsible_for_approval(self):
        self.ensure_one()
        responsible = self.env.user

        if self.validation_type == 'officer' or self.validation_type == 'set':
            if self.holiday_status_id.responsible_id:
                responsible = self.holiday_status_id.responsible_id

        return responsible

    def activity_update(self):
        to_clean, to_do = self.env['hr.leave.allocation'], self.env['hr.leave.allocation']
        for allocation in self:
            note = _(
                'New Allocation Request created by %(user)s: %(count)s Days of %(allocation_type)s',
                user=allocation.create_uid.name,
                count=allocation.number_of_days,
                allocation_type=allocation.holiday_status_id.name
            )
            if allocation.state == 'draft':
                to_clean |= allocation
            elif allocation.state == 'confirm':
                allocation.activity_schedule(
                    'hr_holidays.mail_act_leave_allocation_approval',
                    note=note,
                    user_id=allocation.sudo()._get_responsible_for_approval().id or self.env.user.id)
            elif allocation.state == 'validate1':
                allocation.activity_feedback(['hr_holidays.mail_act_leave_allocation_approval'])
                allocation.activity_schedule(
                    'hr_holidays.mail_act_leave_allocation_second_approval',
                    note=note,
                    user_id=allocation.sudo()._get_responsible_for_approval().id or self.env.user.id)
            elif allocation.state == 'validate':
                to_do |= allocation
            elif allocation.state == 'refuse':
                to_clean |= allocation
        if to_clean:
            to_clean.activity_unlink(['hr_holidays.mail_act_leave_allocation_approval', 'hr_holidays.mail_act_leave_allocation_second_approval'])
        if to_do:
            to_do.activity_feedback(['hr_holidays.mail_act_leave_allocation_approval', 'hr_holidays.mail_act_leave_allocation_second_approval'])

    ####################################################
    # Messaging methods
    ####################################################

    def _track_subtype(self, init_values):
        if 'state' in init_values and self.state == 'validate':
            allocation_notif_subtype_id = self.holiday_status_id.allocation_notif_subtype_id
            return allocation_notif_subtype_id or self.env.ref('hr_holidays.mt_leave_allocation')
        return super(HolidaysAllocation, self)._track_subtype(init_values)

    def _notify_get_groups(self, msg_vals=None):
        """ Handle HR users and officers recipients that can validate or refuse holidays
        directly from email. """
        groups = super(HolidaysAllocation, self)._notify_get_groups(msg_vals=msg_vals)
        local_msg_vals = dict(msg_vals or {})

        self.ensure_one()
        hr_actions = []
        if self.state == 'confirm':
            app_action = self._notify_get_action_link('controller', controller='/allocation/validate', **local_msg_vals)
            hr_actions += [{'url': app_action, 'title': _('Approve')}]
        if self.state in ['confirm', 'validate', 'validate1']:
            ref_action = self._notify_get_action_link('controller', controller='/allocation/refuse', **local_msg_vals)
            hr_actions += [{'url': ref_action, 'title': _('Refuse')}]

        holiday_user_group_id = self.env.ref('hr_holidays.group_hr_holidays_user').id
        new_group = (
            'group_hr_holidays_user', lambda pdata: pdata['type'] == 'user' and holiday_user_group_id in pdata['groups'], {
                'actions': hr_actions,
            })

        return [new_group] + groups

    def message_subscribe(self, partner_ids=None, subtype_ids=None):
        # due to record rule can not allow to add follower and mention on validated leave so subscribe through sudo
        if self.state in ['validate', 'validate1']:
            self.check_access_rights('read')
            self.check_access_rule('read')
            return super(HolidaysAllocation, self.sudo()).message_subscribe(partner_ids=partner_ids, subtype_ids=subtype_ids)
        return super(HolidaysAllocation, self).message_subscribe(partner_ids=partner_ids, subtype_ids=subtype_ids)

```

## File: models\hr_leave_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2005-2006 Axelor SARL. (http://www.axelor.com)

import datetime
import logging

from collections import defaultdict
from datetime import time, timedelta

from odoo import api, fields, models
from odoo.osv import expression
from odoo.tools.translate import _
from odoo.tools.float_utils import float_round
from odoo.addons.resource.models.resource import Intervals

_logger = logging.getLogger(__name__)


class HolidaysType(models.Model):
    _name = "hr.leave.type"
    _description = "Time Off Type"
    _order = 'sequence'

    @api.model
    def _model_sorting_key(self, leave_type):
        remaining = leave_type.virtual_remaining_leaves > 0
        taken = leave_type.leaves_taken > 0
        return -1*leave_type.sequence, leave_type.employee_requests == 'no' and remaining, leave_type.employee_requests == 'yes' and remaining, taken

    name = fields.Char('Time Off Type', required=True, translate=True)
    sequence = fields.Integer(default=100,
                              help='The type with the smallest sequence is the default value in time off request')
    create_calendar_meeting = fields.Boolean(string="Display Time Off in Calendar", default=True)
    color_name = fields.Selection([
        ('red', 'Red'),
        ('blue', 'Blue'),
        ('lightgreen', 'Light Green'),
        ('lightblue', 'Light Blue'),
        ('lightyellow', 'Light Yellow'),
        ('magenta', 'Magenta'),
        ('lightcyan', 'Light Cyan'),
        ('black', 'Black'),
        ('lightpink', 'Light Pink'),
        ('brown', 'Brown'),
        ('violet', 'Violet'),
        ('lightcoral', 'Light Coral'),
        ('lightsalmon', 'Light Salmon'),
        ('lavender', 'Lavender'),
        ('wheat', 'Wheat'),
        ('ivory', 'Ivory')], string='Color in Report', required=True, default='red',
         help='This color will be used in the time off summary located in Reporting > Time off by Department.')
    color = fields.Integer(string='Color', help="The color selected here will be used in every screen with the time off type.")
    icon_id = fields.Many2one('ir.attachment', string='Cover Image', domain="[('res_model', '=', 'hr.leave.type'), ('res_field', '=', 'icon_id')]")
    active = fields.Boolean('Active', default=True,
                            help="If the active field is set to false, it will allow you to hide the time off type without removing it.")
    max_leaves = fields.Float(compute='_compute_leaves', string='Maximum Allowed', search='_search_max_leaves',
                              help='This value is given by the sum of all time off requests with a positive value.')
    leaves_taken = fields.Float(
        compute='_compute_leaves', string='Time off Already Taken',
        help='This value is given by the sum of all time off requests with a negative value.')
    remaining_leaves = fields.Float(
        compute='_compute_leaves', string='Remaining Time Off',
        help='Maximum Time Off Allowed - Time Off Already Taken')
    virtual_remaining_leaves = fields.Float(
        compute='_compute_leaves', search='_search_virtual_remaining_leaves', string='Virtual Remaining Time Off',
        help='Maximum Time Off Allowed - Time Off Already Taken - Time Off Waiting Approval')
    virtual_leaves_taken = fields.Float(
        compute='_compute_leaves', string='Virtual Time Off Already Taken',
        help='Sum of validated and non validated time off requests.')
    # KBA TODO in master: rename, change to int
    group_days_allocation = fields.Float(
        compute='_compute_group_days_allocation', string='Days Allocated')
    group_days_leave = fields.Float(
        compute='_compute_group_days_leave', string='Group Time Off')
    company_id = fields.Many2one('res.company', string='Company', default=lambda self: self.env.company)
    responsible_id = fields.Many2one(
        'res.users', 'Responsible Time Off Officer',
        domain=lambda self: [('groups_id', 'in', self.env.ref('hr_holidays.group_hr_holidays_user').id),
                             ('share', '=', False)],
        help="Choose the Time Off Officer who will be notified to approve allocation or Time Off request")
    leave_validation_type = fields.Selection([
        ('no_validation', 'No Validation'),
        ('hr', 'By Time Off Officer'),
        ('manager', "By Employee's Approver"),
        ('both', "By Employee's Approver and Time Off Officer")], default='hr', string='Leave Validation')
    requires_allocation = fields.Selection([
        ('yes', 'Yes'),
        ('no', 'No Limit')], default="yes", required=True, string='Requires allocation')
    employee_requests = fields.Selection([
        ('yes', 'Extra Days Requests Allowed'),
        ('no', 'Not Allowed')], default="no", required=True, string="Employee Requests")
    allocation_validation_type = fields.Selection([
        ('no', 'No validation needed'),
        ('officer', 'Approved by Time Off Officer'),
        ('set', "Set by Time Off Officer")], default='officer', string='Approval')
    has_valid_allocation = fields.Boolean(compute='_compute_valid', search='_search_valid', help='This indicates if it is still possible to use this type of leave')
    time_type = fields.Selection([('leave', 'Time Off'), ('other', 'Other')], default='leave', string="Kind of Leave",
                                 help="Whether this should be computed as a holiday or as work time (eg: formation)")
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
            date_to = fields.Date.context_today(self, default_date_from_dt)
            date_from = fields.Date.context_today(self, default_date_to_dt)

        else:
            date_to = fields.Date.today().strftime('%Y-1-1')
            date_from = fields.Date.today().strftime('%Y-12-31')

        employee_id = self._context.get('default_employee_id', self._context.get('employee_id')) or self.env.user.employee_id.id

        if not isinstance(value, bool):
            raise ValueError('Invalid value: %s' % (value))
        if operator not in ['=', '!=']:
            raise ValueError('Invalid operator: %s' % (operator))
        new_operator = 'in' if operator == '=' else 'not in'

        query = '''
        SELECT
            holiday_status_id
        FROM
            hr_leave_allocation alloc
        WHERE
            alloc.employee_id = %s AND
            alloc.active = True AND alloc.state = 'validate' AND
            (alloc.date_to >= %s OR alloc.date_to IS NULL) AND
            alloc.date_from <= %s 
        '''

        self._cr.execute(query, (employee_id or None, date_to, date_from))

        return [('id', new_operator, [x['holiday_status_id'] for x in self._cr.dictfetchall()])]


    @api.depends('requires_allocation')
    def _compute_valid(self):
        date_to = self._context.get('default_date_to', fields.Datetime.today())
        date_from = self._context.get('default_date_from', fields.Datetime.today())
        employee_id = self._context.get('default_employee_id', self._context.get('employee_id', self.env.user.employee_id.id))
        for holiday_type in self:
            if holiday_type.requires_allocation == 'yes':
                allocation = self.env['hr.leave.allocation'].search([
                    ('holiday_status_id', '=', holiday_type.id),
                    ('employee_id', '=', employee_id),
                    '|',
                    ('date_to', '>=', date_to),
                    '&',
                    ('date_to', '=', False),
                    ('date_from', '<=', date_from)])
                holiday_type.has_valid_allocation = bool(allocation)
            else:
                holiday_type.has_valid_allocation = True

    def _search_max_leaves(self, operator, value):
        value = float(value)
        employee_id = self._get_contextual_employee_id()
        leaves = defaultdict(int)

        if employee_id:
            allocations = self.env['hr.leave.allocation'].search([
                ('employee_id', '=', employee_id),
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

    def _get_employees_days_per_allocation(self, employee_ids, date=None):
        leaves = self.env['hr.leave'].search([
            ('employee_id', 'in', employee_ids),
            ('state', 'in', ['confirm', 'validate1', 'validate']),
            ('holiday_status_id', 'in', self.ids)
        ])

        allocations = self.env['hr.leave.allocation'].with_context(active_test=False).search([
            ('employee_id', 'in', employee_ids),
            ('state', 'in', ['validate']),
            ('holiday_status_id', 'in', self.ids),
        ])

        if not date:
            date = fields.Date.to_date(self.env.context.get('default_date_from')) or fields.Date.context_today(self)

        # The allocation_employees dictionary groups the allocations based on the employee and the holiday type
        # The structure is the following:
        # - KEYS:
        # allocation_employees
        #   |--employee_id
        #      |--holiday_status_id
        # - VALUES:
        # Intervals with the start and end date of each allocation and associated allocations within this interval
        allocation_employees = defaultdict(lambda: defaultdict(list))

        ### Creation of the allocation intervals ###
        for holiday_status_id in allocations.holiday_status_id:
            for employee_id in employee_ids:
                allocation_intervals = Intervals([(
                    fields.datetime.combine(allocation.date_from, time.min),
                    fields.datetime.combine(allocation.date_to or datetime.date.max, time.max),
                    allocation)
                    for allocation in allocations.filtered(lambda allocation: allocation.employee_id.id == employee_id and allocation.holiday_status_id == holiday_status_id)])

                allocation_employees[employee_id][holiday_status_id] = allocation_intervals

        # The leave_employees dictionary groups the leavess based on the employee and the holiday type
        # The structure is the following:
        # - KEYS:
        # leave_employees
        #   |--employee_id
        #      |--holiday_status_id
        # - VALUES:
        # Intervals with the start and end date of each leave and associated leave within this interval
        leaves_employees = defaultdict(lambda: defaultdict(list))
        leave_intervals = []

        ### Creation of the leave intervals ###
        if leaves:
            for holiday_status_id in leaves.holiday_status_id:
                for employee_id in employee_ids:
                    leave_intervals = Intervals([(
                        fields.datetime.combine(leave.date_from, time.min),
                        fields.datetime.combine(leave.date_to, time.max),
                        leave)
                        for leave in leaves.filtered(lambda leave: leave.employee_id.id == employee_id and leave.holiday_status_id == holiday_status_id)])

                    leaves_employees[employee_id][holiday_status_id] = leave_intervals

        # allocation_days_consumed is a dictionary to map the number of days/hours of leaves taken per allocation
        # The structure is the following:
        # - KEYS:
        # allocation_days_consumed
        #  |--employee_id
        #      |--holiday_status_id
        #          |--allocation
        #              |--virtual_leaves_taken
        #              |--leaves_taken
        #              |--virtual_remaining_leaves
        #              |--remaining_leaves
        #              |--max_leaves
        # - VALUES:
        # Integer representing the number of (virtual) remaining leaves, (virtual) leaves taken or max leaves for each allocation.
        # The unit is in hour or days depending on the leave type request unit
        allocations_days_consumed = defaultdict(lambda: defaultdict(lambda: defaultdict(lambda: defaultdict(lambda: 0))))

        company_domain = [('company_id', 'in', list(set(self.env.company.ids + self.env.context.get('allowed_company_ids', []))))]

        ### Existing leaves assigned to allocations ###
        if leaves_employees:
            for employee_id, leaves_interval_by_status in leaves_employees.items():
                for holiday_status_id in leaves_interval_by_status:
                    days_consumed = allocations_days_consumed[employee_id][holiday_status_id]
                    if allocation_employees[employee_id][holiday_status_id]:
                        allocations = allocation_employees[employee_id][holiday_status_id] & leaves_interval_by_status[holiday_status_id]
                        available_allocations = self.env['hr.leave.allocation']
                        for allocation_interval in allocations._items:
                            available_allocations |= allocation_interval[2]
                        # Consume the allocations that are close to expiration first
                        sorted_available_allocations = available_allocations.filtered('date_to').sorted(key='date_to')
                        sorted_available_allocations += available_allocations.filtered(lambda allocation: not allocation.date_to)
                        leave_intervals = leaves_interval_by_status[holiday_status_id]._items
                        for leave_interval in leave_intervals:
                            leaves = leave_interval[2]
                            for leave in leaves:
                                if leave.leave_type_request_unit in ['day', 'half_day']:
                                    leave_duration = leave.number_of_days
                                    leave_unit = 'days'
                                else:
                                    leave_duration = leave.number_of_hours_display
                                    leave_unit = 'hours'
                                if holiday_status_id.requires_allocation != 'no':
                                    for available_allocation in sorted_available_allocations:
                                        if (available_allocation.date_to and available_allocation.date_to < leave.date_from.date()) \
                                            or (available_allocation.date_from > leave.date_to.date()):
                                            continue
                                        virtual_remaining_leaves = (available_allocation.number_of_days if leave_unit == 'days' else available_allocation.number_of_hours_display) - allocations_days_consumed[employee_id][holiday_status_id][available_allocation]['virtual_leaves_taken']
                                        max_leaves = min(virtual_remaining_leaves, leave_duration)
                                        days_consumed[available_allocation]['virtual_leaves_taken'] += max_leaves
                                        if leave.state == 'validate':
                                            days_consumed[available_allocation]['leaves_taken'] += max_leaves
                                        leave_duration -= max_leaves
                                    if leave_duration > 0:
                                        # There are not enough allocation for the number of leaves
                                        days_consumed[False]['virtual_remaining_leaves'] -= leave_duration
                                        # Hack to make sure that a sum of several allocations does not hide an error
                                        days_consumed['error']['virtual_remaining_leaves'] -= leave_duration
                                else:
                                    days_consumed[False]['virtual_leaves_taken'] += leave_duration
                                    if leave.state == 'validate':
                                        days_consumed[False]['leaves_taken'] += leave_duration

        # Future available leaves
        future_allocations_date_from = fields.datetime.combine(date, time.min)
        future_allocations_date_to = fields.datetime.combine(date, time.max) + timedelta(days=5*365)
        for employee_id, allocation_intervals_by_status in allocation_employees.items():
            employee = self.env['hr.employee'].browse(employee_id)
            for holiday_status_id, intervals in allocation_intervals_by_status.items():
                if not intervals:
                    continue
                future_allocation_intervals = intervals & Intervals([(
                    future_allocations_date_from,
                    future_allocations_date_to,
                    self.env['hr.leave'])])
                search_date = date
                for interval_from, interval_to, interval_allocations in future_allocation_intervals._items:
                    if interval_from.date() > search_date:
                        continue
                    interval_allocations = interval_allocations.filtered('active')
                    if not interval_allocations:
                        continue
                    # If no end date to the allocation, consider the number of days remaining as infinite
                    employee_quantity_available = (
                        employee._get_work_days_data_batch(interval_from, interval_to, compute_leaves=False, domain=company_domain)[employee_id]
                        if interval_to != future_allocations_date_to
                        else {'days': float('inf'), 'hours': float('inf')}
                    )
                    for allocation in interval_allocations:
                        if allocation.date_from > search_date:
                            continue
                        days_consumed = allocations_days_consumed[employee_id][holiday_status_id][allocation]
                        if allocation.type_request_unit in ['day', 'half_day']:
                            quantity_available = employee_quantity_available['days']
                            remaining_days_allocation = (allocation.number_of_days - days_consumed['virtual_leaves_taken'])
                        else:
                            quantity_available = employee_quantity_available['hours']
                            remaining_days_allocation = (allocation.number_of_hours_display - days_consumed['virtual_leaves_taken'])
                        if quantity_available <= remaining_days_allocation:
                            search_date = interval_to.date() + timedelta(days=1)
                        days_consumed['virtual_remaining_leaves'] += min(quantity_available, remaining_days_allocation)
                        days_consumed['max_leaves'] = allocation.number_of_days if allocation.type_request_unit in ['day', 'half_day'] else allocation.number_of_hours_display
                        days_consumed['remaining_leaves'] = days_consumed['max_leaves'] - days_consumed['leaves_taken']
                        if remaining_days_allocation >= quantity_available:
                            break

        return allocations_days_consumed


    def get_employees_days(self, employee_ids, date=None):

        result = {
            employee_id: {
                leave_type.id: {
                    'max_leaves': 0,
                    'leaves_taken': 0,
                    'remaining_leaves': 0,
                    'virtual_remaining_leaves': 0,
                    'virtual_leaves_taken': 0,
                } for leave_type in self
            } for employee_id in employee_ids
        }

        if not date:
            date = fields.Date.to_date(self.env.context.get('default_date_from')) or fields.Date.context_today(self)

        allocations_days_consumed = self._get_employees_days_per_allocation(employee_ids, date)

        leave_keys = ['max_leaves', 'leaves_taken', 'remaining_leaves', 'virtual_remaining_leaves', 'virtual_leaves_taken']

        for employee_id in allocations_days_consumed:
            for holiday_status_id in allocations_days_consumed[employee_id]:
                if allocations_days_consumed[employee_id][holiday_status_id].get('error'):
                    for leave_key in leave_keys:
                        result[employee_id][holiday_status_id if isinstance(holiday_status_id, int) else holiday_status_id.id][leave_key] = allocations_days_consumed[employee_id][holiday_status_id]['error'][leave_key]
                    continue
                for allocation in allocations_days_consumed[employee_id][holiday_status_id]:
                    if allocation and allocation.date_to and (allocation.date_to < date or allocation.date_from > date):
                        continue
                    for leave_key in leave_keys:
                        result[employee_id][holiday_status_id if isinstance(holiday_status_id, int) else holiday_status_id.id][leave_key] += allocations_days_consumed[employee_id][holiday_status_id][allocation][leave_key]

        return result

    @api.model
    def get_days_all_request(self):
        leave_types = sorted(self.search([]).filtered(lambda x: ((x.virtual_remaining_leaves > 0 or x.max_leaves))), key=self._model_sorting_key, reverse=True)
        return [lt._get_days_request() for lt in leave_types]

    def _get_days_request(self):
        self.ensure_one()
        return (self.name, {
                'remaining_leaves': ('%.2f' % self.remaining_leaves).rstrip('0').rstrip('.'),
                'usable_remaining_leaves': ('%.2f' % self.virtual_remaining_leaves).rstrip('0').rstrip('.'),
                'virtual_remaining_leaves': ('%.2f' % (self.max_leaves - self.virtual_leaves_taken)).rstrip('0').rstrip('.'),
                'max_leaves': ('%.2f' % self.max_leaves).rstrip('0').rstrip('.'),
                'leaves_taken': ('%.2f' % self.leaves_taken).rstrip('0').rstrip('.'),
                'virtual_leaves_taken': ('%.2f' % self.virtual_leaves_taken).rstrip('0').rstrip('.'),
                'request_unit': self.request_unit,
                'icon': self.sudo().icon_id.url,
                }, self.requires_allocation, self.id)

    def _get_contextual_employee_id(self):
        if 'employee_id' in self._context:
            employee_id = self._context['employee_id']
        elif 'default_employee_id' in self._context:
            employee_id = self._context['default_employee_id']
        else:
            employee_id = self.env.user.employee_id.id
        return employee_id

    @api.depends_context('employee_id', 'default_employee_id')
    def _compute_leaves(self):
        data_days = {}
        employee_id = self._get_contextual_employee_id()

        if employee_id:
            data_days = (self.get_employees_days(employee_id)[employee_id[0]] if isinstance(employee_id, list) else
                         self.get_employees_days([employee_id])[employee_id])

        for holiday_status in self:
            result = data_days.get(holiday_status.id, {})
            holiday_status.max_leaves = result.get('max_leaves', 0)
            holiday_status.leaves_taken = result.get('leaves_taken', 0)
            holiday_status.remaining_leaves = result.get('remaining_leaves', 0)
            holiday_status.virtual_remaining_leaves = result.get('virtual_remaining_leaves', 0)
            holiday_status.virtual_leaves_taken = result.get('virtual_leaves_taken', 0)

    def _compute_group_days_allocation(self):
        grouped_res = self.env['hr.leave.allocation'].read_group(
            [('holiday_status_id', 'in', self.ids), ],
            ['holiday_status_id'],
            ['holiday_status_id'],
        )
        grouped_dict = dict((data['holiday_status_id'][0], data['holiday_status_id_count']) for data in grouped_res)
        for allocation in self:
            allocation.group_days_allocation = grouped_dict.get(allocation.id, 0)

    def _compute_group_days_leave(self):
        grouped_res = self.env['hr.leave'].read_group(
            [('holiday_status_id', 'in', self.ids),
             ('date_from', '>=', fields.Datetime.to_string(datetime.datetime.now().replace(month=1, day=1, hour=0, minute=0, second=0, microsecond=0)))],
            ['holiday_status_id'],
            ['holiday_status_id'],
        )
        grouped_dict = dict((data['holiday_status_id'][0], data['holiday_status_id_count']) for data in grouped_res)
        for allocation in self:
            allocation.group_days_leave = grouped_dict.get(allocation.id, 0)

    def _compute_accrual_count(self):
        accrual_allocations = self.env['hr.leave.accrual.plan'].read_group([('time_off_type_id', 'in', self.ids)], ['time_off_type_id'], ['time_off_type_id'])
        mapped_data = dict((data['time_off_type_id'][0], data['time_off_type_id_count']) for data in accrual_allocations)
        for leave_type in self:
            leave_type.accrual_count = mapped_data.get(leave_type.id, 0)

    def name_get(self):
        if not self._context.get('employee_id'):
            # leave counts is based on employee_id, would be inaccurate if not based on correct employee
            return super(HolidaysType, self).name_get()
        res = []
        for record in self:
            name = record.name
            if record.requires_allocation == "yes" and not self._context.get('from_manager_leave_form'):
                name = "%(name)s (%(count)s)" % {
                    'name': name,
                    'count': _('%g remaining out of %g') % (
                        float_round(record.virtual_remaining_leaves, precision_digits=2) or 0.0,
                        float_round(record.max_leaves, precision_digits=2) or 0.0,
                    ) + (_(' hours') if record.request_unit == 'hour' else _(' days'))
                }
            res.append((record.id, name))
        return res

    @api.model
    def _search(self, args, offset=0, limit=None, order=None, count=False, access_rights_uid=None):
        """ Override _search to order the results, according to some employee.
        The order is the following

         - allocation fixed first, then allowing allocation, then free allocation
         - virtual remaining leaves (higher the better, so using reverse on sorted)

        This override is necessary because those fields are not stored and depends
        on an employee_id given in context. This sort will be done when there
        is an employee_id in context and that no other order has been given
        to the method.
        """
        employee_id = self._get_contextual_employee_id()
        post_sort = (not count and not order and employee_id)
        leave_ids = super(HolidaysType, self)._search(args, offset=offset, limit=(None if post_sort else limit), order=order, count=count, access_rights_uid=access_rights_uid)
        leaves = self.browse(leave_ids)
        if post_sort:
            return leaves.sorted(key=self._model_sorting_key, reverse=True).ids[:limit or None]
        return leave_ids

    def action_see_days_allocated(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("hr_holidays.hr_leave_allocation_action_all")
        action['domain'] = [
            ('holiday_status_id', 'in', self.ids),
        ]
        action['context'] = {
            'default_holiday_type': 'department',
            'default_holiday_status_id': self.ids[0],
        }
        return action

    def action_see_group_leaves(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("hr_holidays.hr_leave_action_action_approve_department")
        action['domain'] = [
            ('holiday_status_id', '=', self.ids[0]),
            ('date_from', '>=', fields.Datetime.to_string(datetime.datetime.now().replace(month=1, day=1, hour=0, minute=0, second=0, microsecond=0)))
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
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class CalendarLeaves(models.Model):
    _inherit = "resource.calendar.leaves"

    holiday_id = fields.Many2one("hr.leave", string='Leave Request')

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

    def mail_partner_format(self):
        """Override to add the current leave status."""
        partners_format = super().mail_partner_format()
        for partner in self:
            # in the rare case of multi-user partner, return the earliest possible return date
            dates = partner.mapped('user_ids.leave_date_to')
            date = sorted(dates)[0] if dates and all(dates) else False
            partners_format.get(partner).update({
                'out_of_office_date_end': date.strftime(DEFAULT_SERVER_DATE_FORMAT) if date else False,
            })
        return partners_format

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class User(models.Model):
    _inherit = "res.users"

    leave_manager_id = fields.Many2one(related='employee_id.leave_manager_id')
    show_leaves = fields.Boolean(related='employee_id.show_leaves')
    allocation_used_count = fields.Float(related='employee_id.allocation_used_count')
    allocation_count = fields.Float(related='employee_id.allocation_count')
    leave_date_to = fields.Date(related='employee_id.leave_date_to')
    current_leave_state = fields.Selection(related='employee_id.current_leave_state')
    is_absent = fields.Boolean(related='employee_id.is_absent')
    allocation_used_display = fields.Char(related='employee_id.allocation_used_display')
    allocation_display = fields.Char(related='employee_id.allocation_display')
    hr_icon_display = fields.Selection(related='employee_id.hr_icon_display')

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + [
            'leave_manager_id',
            'show_leaves',
            'allocation_used_count',
            'allocation_count',
            'leave_date_to',
            'current_leave_state',
            'is_absent',
            'allocation_used_display',
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
        self.env['res.users'].flush(fnames=['active'])
        self.env['hr.leave'].flush(fnames=['user_id', 'state', 'date_from', 'date_to'])
        self.env.cr.execute('''SELECT res_users.%s FROM res_users
                            JOIN hr_leave ON hr_leave.user_id = res_users.id
                            AND state not in ('cancel', 'refuse')
                            AND res_users.active = 't'
                            AND date_from <= %%s AND date_to >= %%s''' % field, (now, now))
        return [r[0] for r in self.env.cr.fetchall()]

    def _clean_leave_responsible_users(self):
        # self = old bunch of leave responsibles
        # This method compares the current leave managers
        # and remove the access rights to those who don't
        # need them anymore
        approver_group = self.env.ref('hr_holidays.group_hr_holidays_responsible', raise_if_not_found=False)
        if not self or not approver_group:
            return
        res = self.env['hr.employee'].read_group(
            [('leave_manager_id', 'in', self.ids)],
            ['leave_manager_id'],
            ['leave_manager_id'])
        responsibles_to_remove_ids = set(self.ids) - {x['leave_manager_id'][0] for x in res}
        approver_group.sudo().write({
            'users': [(3, manager_id) for manager_id in responsibles_to_remove_ids]})

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import resource
from . import hr_employee
from . import hr_department
from . import hr_leave
from . import hr_leave_allocation
from . import hr_leave_type
from . import hr_leave_accrual_plan_level
from . import hr_leave_accrual_plan
from . import mail_message_subtype
from . import res_partner
from . import res_users

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
            res.append({'day_str': babel.dates.get_day_names('abbreviated', locale=get_lang(self.env).code)[start_date.weekday()], 'day': start_date.day , 'color': color})
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
        # count and get leave summary details.
        holiday_type = ['confirm','validate'] if holiday_type == 'both' else ['confirm'] if holiday_type == 'Confirmed' else ['validate']
        holidays = self.env['hr.leave'].search([
            ('employee_id', '=', empid), ('state', 'in', holiday_type),
            ('date_from', '<=', str(end_date)),
            ('date_to', '>=', str(start_date))
        ])
        for holiday in holidays:
            # Convert date to user timezone, otherwise the report will not be consistent with the
            # value displayed in the interface.
            date_from = fields.Datetime.from_string(holiday.date_from)
            date_from = fields.Datetime.context_timestamp(holiday, date_from).date()
            date_to = fields.Datetime.from_string(holiday.date_to)
            date_to = fields.Datetime.context_timestamp(holiday, date_to).date()
            for index in range(0, ((date_to - date_from).days + 1)):
                if date_from >= start_date and date_from <= end_date:
                    res[(date_from-start_date).days]['color'] = holiday.holiday_status_id.color_name
                date_from += timedelta(1)
            count += holiday.number_of_days
        employee = self.env['hr.employee'].browse(empid)
        return {'emp': employee.name, 'display': res, 'sum': count}

    def _get_data_from_report(self, data):
        res = []
        Employee = self.env['hr.employee']
        if 'depts' in data:
            for department in self.env['hr.department'].browse(data['depts']):
                res.append({
                    'dept': department.name,
                    'data': [
                        self._get_leaves_summary(data['date_from'], emp.id, data['holiday_type'])
                        for emp in Employee.search([('department_id', '=', department.id)])
                    ],
                    'color': self._get_day(data['date_from']),
                })
        elif 'emp' in data:
            res.append({'data': [
                self._get_leaves_summary(data['date_from'], emp.id, data['holiday_type'])
                for emp in Employee.browse(data['emp'])
            ]})
        return res

    def _get_holidays_status(self):
        res = []
        for holiday in self.env['hr.leave.type'].search([]):
            res.append({'color': holiday.color_name, 'name': holiday.name})
        return res

    @api.model
    def _get_report_values(self, docids, data=None):
        if not data.get('form'):
            raise UserError(_("Form content is missing, this report cannot be printed."))

        holidays_report = self.env['ir.actions.report']._get_report_from_name('hr_holidays.report_holidayssummary')
        holidays = self.env['hr.leave'].browse(self.ids)
        return {
            'doc_ids': self.ids,
            'doc_model': holidays_report.model,
            'docs': holidays,
            'get_header_info': self._get_header_info(data['form']['date_from'], data['form']['holiday_type']),
            'get_day': self._get_day(data['form']['date_from']),
            'get_months': self._get_months(data['form']['date_from']),
            'get_data_from_report': self._get_data_from_report(data['form']),
            'get_holidays_status': self._get_holidays_status(),
        }

```

## File: report\hr_holidays_reports.xml

```xml
<?xml version="1.0"?>
<odoo>

        <record id="action_report_holidayssummary" model="ir.actions.report">
            <field name="name">Time Off Summary</field>
            <field name="model">hr.holidays.summary.dept</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">hr_holidays.report_holidayssummary</field>
            <field name="report_file">hr_holidays.report_holidayssummary</field>
        </record>

        <record id="action_report_holidayssummary" model="ir.actions.report">
            <field name="paperformat_id" ref="hr_holidays.paperformat_hrsummary"/>
        </record>

        <record id="action_report_holidayssummary2" model="ir.actions.report">
            <field name="name">Time Off Summary</field>
            <field name="model">hr.leave.allocation</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">hr_holidays.report_holidayssummary</field>
            <field name="report_file">hr_holidays.report_holidayssummary</field>
        </record>

        <record id="action_report_holidayssummary" model="ir.actions.report">
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
                                &lt;td style=background-color:<t t-esc="status['color']"/>!important &gt;&lt;/td&gt;
                                <td><t t-esc="status['name']"/></td>
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
    active_employee = fields.Boolean(related='employee_id.active', readonly=True)
    number_of_days = fields.Float('Number of Days', readonly=True, group_operator="sum")
    department_id = fields.Many2one('hr.department', string='Department', readonly=True)
    leave_type = fields.Many2one("hr.leave.type", string="Leave Type", readonly=True)
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
                    WHERE request.state IN ('confirm', 'validate', 'validate1')
                ) leaves
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
    active_employee = fields.Boolean(related='employee_id.active', readonly=True)
    name = fields.Char('Description', readonly=True)
    number_of_days = fields.Float('Number of Days', readonly=True)
    leave_type = fields.Selection([
        ('allocation', 'Allocation'),
        ('request', 'Time Off')
        ], string='Request Type', readonly=True)
    department_id = fields.Many2one('hr.department', string='Department', readonly=True)
    category_id = fields.Many2one('hr.employee.category', string='Employee Tag', readonly=True)
    holiday_status_id = fields.Many2one("hr.leave.type", string="Leave Type", readonly=True)
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
                leaves.employee_id as employee_id, leaves.name as name,
                leaves.number_of_days as number_of_days, leaves.leave_type as leave_type,
                leaves.category_id as category_id, leaves.department_id as department_id,
                leaves.holiday_status_id as holiday_status_id, leaves.state as state,
                leaves.holiday_type as holiday_type, leaves.date_from as date_from,
                leaves.date_to as date_to, leaves.company_id
                from (select
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
                union all select
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
                from hr_leave as request) leaves
            );
        """)

    @api.model
    def action_time_off_analysis(self):
        domain = [('holiday_type', '=', 'employee')]

        if self.env.context.get('active_ids'):
            domain = expression.AND([
                domain,
                [('employee_id', 'in', self.env.context.get('active_ids', []))]
            ])

        return {
            'name': _('Time Off Analysis'),
            'type': 'ir.actions.act_window',
            'res_model': 'hr.leave.report',
            'view_mode': 'tree,pivot,form',
            'search_view_id': [self.env.ref('hr_holidays.view_hr_holidays_filter_report').id],
            'domain': domain,
            'context': {
                'search_default_group_type': True,
                'search_default_year': True,
                'search_default_validated': True,
                'search_default_active_employee': True,
            }
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
                <filter name="active_types" string="Active Types" domain="[('holiday_status_id.active', '=', True)]" help="Filters only on requests that belong to an time off type that is 'active' (active field is True)"/>
                <separator/>
                <filter string="My Department" name="department" domain="[('department_id.manager_id.user_id', '=', uid)]" help="My Department"/>
                <separator/>
                <filter string="Active Employee" name="active_employee" domain="[('active_employee','=',True)]"/>
                <separator/>
                <filter name="year" string="Current Year"
                    domain="[('holiday_status_id.active', '=', True)]" help="Active Time Off"/>
                <separator/>
                <filter string="My Requests" name="my_leaves" domain="[('employee_id.user_id', '=', uid)]"/>
                <separator/>
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
            <tree create="0" edit="0" delete="0">
                <field name="employee_id" decoration-muted="not active_employee"/>
                <field name="number_of_days" string="Number of Days" sum="Remaining Days"/>
                <field name="leave_type"/>
                <field name="date_from"/>
                <field name="date_to"/>
                <field name="state"/>
                <field name="name"/>
                <field name="active_employee" invisible="1"/>
            </tree>
        </field>
    </record>

    <record id="hr_leave_report_pivot" model="ir.ui.view">
        <field name="name">report.hr.holidays.report.leave_all.pivot</field>
        <field name="model">hr.leave.report</field>
        <field name="arch" type="xml">
            <pivot>
                <field name="employee_id" decoration-muted="not active_employee"/>
                <field name="number_of_days" type="measure"/>
                <field name="leave_type"/>
                <field name="date_from"/>
                <field name="date_to"/>
                <field name="state"/>
                <field name="name"/>
                <field name="active_employee" invisible="1"/>
            </pivot>
        </field>
    </record>

    <record id="act_hr_employee_holiday_request" model="ir.actions.server">
        <field name="name">Time off Analysis</field>
        <field name="model_id" ref="hr_holidays.model_hr_leave_report"/>
        <field name="binding_model_id" ref="hr.model_hr_employee"/>
        <field name="state">code</field>
        <field name="groups_id" eval="[(4, ref('base.group_user'))]"/>
        <field name="code">
        action = model.action_time_off_analysis()
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
    duration = fields.Float(string='Duration', readonly=True, store=False)
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
        );
        """)

    def _read(self, fields):
        res = super()._read(fields)
        if self.env.context.get('hide_employee_name') and 'employee_id' in self.env.context.get('group_by', []):
            name_field = self._fields['name']
            for record in self.with_user(SUPERUSER_ID):
                self.env.cache.set(record, name_field, record.name.split(':')[-1].strip())
        return res

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
            <calendar string="Time Off" date_start="start_datetime" date_stop="stop_datetime" mode="month" quick_add="False" color="employee_id" event_open_popup="True" js_class="time_off_calendar_all" show_unusual_days="True">
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
                <filter string="My Department" name="department" domain="['|', ('department_id.member_ids.user_id', '=', uid), ('employee_id.user_id', '=', uid)]" help="My Department"/>
                <separator/>
                <filter string="Off Today" name="off_today" domain="[('start_datetime', '&lt;=', context_today().strftime('%Y-%m-%d')), ('stop_datetime', '&gt;=', context_today().strftime('%Y-%m-%d'))]" help="My Department"/>
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
        <field name="description">Helps you manage your time off.</field>
        <field name="sequence">10</field>
    </record>

    <record id="group_hr_holidays_responsible" model="res.groups">
        <field name="name">Time Off Responsible</field>
        <field name="category_id" ref="base.module_category_hidden"/>
        <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <record id="group_hr_holidays_user" model="res.groups">
        <field name="name">Time Off Officer</field>
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
        <field name="domain_force">[('employee_id.user_id', '=', user.id), ('state', 'in', ['draft', 'confirm'])]</field>
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
        <field name="domain_force">['|', ('holiday_status_id', '=', False), ('holiday_status_id.company_id', 'in', company_ids + [False])]</field>
    </record>

    <record id="hr_leave_allocation_rule_multicompany" model="ir.rule">
        <field name="name">Leave Allocations: multi company global rule</field>
        <field name="model_id" ref="model_hr_leave_allocation"/>
        <field name="domain_force">[('holiday_status_id.company_id', 'in', company_ids + [False])]</field>
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
                '&amp;',
                    ('employee_id.user_id', '=', user.id),
                    ('state', '!=', 'validate'),
                '&amp;',
                    ('validation_type', 'in', ['officer', 'set']),
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
        <field name="name">Allocations: holiday user : create/write</field>
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

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70">
    <defs>
        <path id="icon-a" d="M4,5.35309892e-14 C36.4160122,9.87060235e-15 58.0836068,-3.97961823e-14 65,5.07020818e-14 C69,6.733808e-14 70,1 70,5 C70,43.0488877 70,62.4235458 70,65 C70,69 69,70 65,70 C61,70 9,70 4,70 C1,70 7.10542736e-15,69 7.10542736e-15,65 C7.25721566e-15,62.4676575 3.83358709e-14,41.8005206 3.60818146e-14,5 C-1.13686838e-13,1 1,5.75716207e-14 4,5.35309892e-14 Z"/>
        <linearGradient id="icon-c" x1="100%" x2="0%" y1="0%" y2="100%">
            <stop offset="0%" stop-color="#CDC484"/>
            <stop offset="100%" stop-color="#B5AA59"/>
        </linearGradient>
        <path id="icon-d" d="M27.2777778,16.2348485 C32.4714536,16.2348485 36.681713,20.506348 36.681713,25.7755682 C36.681713,31.0447884 32.4714536,35.3162879 27.2777778,35.3162879 C22.0841019,35.3162879 17.8738426,31.0447884 17.8738426,25.7755682 C17.8738426,20.506348 22.0841019,16.2348485 27.2777778,16.2348485 Z M32,36 C32,35.5868456 24.1175854,37.5736818 20.6967864,35.0773525 L16.5053937,36.1404272 C13.9936026,36.7775087 12.2314815,39.0671622 12.2314815,41.6939608 L12.2314815,43.9029356 C12.2314815,45.4837136 13.4945475,46.7651515 15.052662,46.7651515 C25.363233,46.7651515 30.5185185,46.7651515 30.5185185,46.7651515 C28.6666667,40.4242424 32,36.4131544 32,36 Z M48.5116455,29.4292211 L49.8821338,32.7804044 C50.108418,33.3336802 50.7775488,33.5511019 51.2858154,33.2365172 L54.3643115,31.3309087 C55.1962939,30.8159288 56.2338818,31.569761 56.0011523,32.5201752 L55.140166,36.0368953 C54.9979932,36.61751 55.4115674,37.186738 56.0077051,37.230942 L59.6183691,37.4987982 C60.5941357,37.5712005 60.9904688,38.790921 60.2436182,39.4230444 L57.4800293,41.7619923 C57.0237549,42.1481736 57.0237549,42.8517324 57.4800293,43.2379137 L60.2436719,45.5769153 C60.9905762,46.2090387 60.5942432,47.4287592 59.6184229,47.5011615 L56.0077588,47.7690177 C55.4116211,47.8132217 54.9980469,48.3824497 55.1402197,48.9630644 L56.0012061,52.4797846 C56.2338818,53.4301987 55.1962939,54.1840309 54.3643652,53.669051 L51.2858691,51.7634425 C50.7776025,51.4488041 50.108418,51.6662258 49.8821875,52.2195553 L48.5116992,55.5707386 C48.1413086,56.4764115 46.8587988,56.4764115 46.4884082,55.5707386 L45.1179199,52.2195553 C44.8916357,51.6662795 44.2225049,51.4488578 43.7142383,51.7634425 L40.6356885,53.669051 C39.8037061,54.1840309 38.7661182,53.4301987 38.9988477,52.4797846 L39.859834,48.9630644 C40.0020068,48.3824497 39.5884326,47.8132217 38.9922949,47.7690177 L35.3816309,47.5011615 C34.4058643,47.4287592 34.0095313,46.2090387 34.7563818,45.5769153 L37.5199707,43.2379674 C37.9762451,42.8517862 37.9762451,42.1482273 37.5199707,41.762046 L34.7563281,39.4230444 C34.0094238,38.790921 34.4057568,37.5712005 35.3815771,37.4987982 L38.9922412,37.230942 C39.5883789,37.186738 40.0019531,36.61751 39.8597803,36.0368953 L38.9987939,32.5201752 C38.7661182,31.569761 39.8037061,30.8159288 40.6356348,31.3309087 L43.7141846,33.2365172 C44.2224512,33.5511556 44.891582,33.3337339 45.1178662,32.7804044 L46.4883545,29.4292211 C46.8587451,28.5236019 48.1412549,28.5236019 48.5116455,29.4292211 Z M54.8046875,42.4999799 C54.8046875,38.4721469 51.5277832,35.1952995 47.5,35.1952995 C43.4721631,35.1952995 40.1953125,38.4721469 40.1953125,42.4999799 C40.1953125,46.5278128 43.4721631,49.8046602 47.5,49.8046602 C51.5277832,49.8046602 54.8046875,46.5278128 54.8046875,42.4999799 Z M53.0859375,42.4999799 C53.0859375,45.5800843 50.5801074,48.0859119 47.5,48.0859119 C44.4198926,48.0859119 41.9140625,45.5800843 41.9140625,42.4999799 C41.9140625,39.4198754 44.4198926,36.9140478 47.5,36.9140478 C50.5801074,36.9140478 53.0859375,39.4198754 53.0859375,42.4999799 Z"/>
        <path id="icon-e" d="M27.2777778,14.2348485 C32.4714536,14.2348485 36.681713,18.506348 36.681713,23.7755682 C36.681713,29.0447884 32.4714536,33.3162879 27.2777778,33.3162879 C22.0841019,33.3162879 17.8738426,29.0447884 17.8738426,23.7755682 C17.8738426,18.506348 22.0841019,14.2348485 27.2777778,14.2348485 Z M32,34 C32,33.5868456 24.1175854,35.5736818 20.6967864,33.0773525 L16.5053937,34.1404272 C13.9936026,34.7775087 12.2314815,37.0671622 12.2314815,39.6939608 L12.2314815,41.9029356 C12.2314815,43.4837136 13.4945475,44.7651515 15.052662,44.7651515 C25.363233,44.7651515 30.5185185,44.7651515 30.5185185,44.7651515 C28.6666667,38.4242424 32,34.4131544 32,34 Z M48.5116455,27.4292211 L49.8821338,30.7804044 C50.108418,31.3336802 50.7775488,31.5511019 51.2858154,31.2365172 L54.3643115,29.3309087 C55.1962939,28.8159288 56.2338818,29.569761 56.0011523,30.5201752 L55.140166,34.0368953 C54.9979932,34.61751 55.4115674,35.186738 56.0077051,35.230942 L59.6183691,35.4987982 C60.5941357,35.5712005 60.9904688,36.790921 60.2436182,37.4230444 L57.4800293,39.7619923 C57.0237549,40.1481736 57.0237549,40.8517324 57.4800293,41.2379137 L60.2436719,43.5769153 C60.9905762,44.2090387 60.5942432,45.4287592 59.6184229,45.5011615 L56.0077588,45.7690177 C55.4116211,45.8132217 54.9980469,46.3824497 55.1402197,46.9630644 L56.0012061,50.4797846 C56.2338818,51.4301987 55.1962939,52.1840309 54.3643652,51.669051 L51.2858691,49.7634425 C50.7776025,49.4488041 50.108418,49.6662258 49.8821875,50.2195553 L48.5116992,53.5707386 C48.1413086,54.4764115 46.8587988,54.4764115 46.4884082,53.5707386 L45.1179199,50.2195553 C44.8916357,49.6662795 44.2225049,49.4488578 43.7142383,49.7634425 L40.6356885,51.669051 C39.8037061,52.1840309 38.7661182,51.4301987 38.9988477,50.4797846 L39.859834,46.9630644 C40.0020068,46.3824497 39.5884326,45.8132217 38.9922949,45.7690177 L35.3816309,45.5011615 C34.4058643,45.4287592 34.0095313,44.2090387 34.7563818,43.5769153 L37.5199707,41.2379674 C37.9762451,40.8517862 37.9762451,40.1482273 37.5199707,39.762046 L34.7563281,37.4230444 C34.0094238,36.790921 34.4057568,35.5712005 35.3815771,35.4987982 L38.9922412,35.230942 C39.5883789,35.186738 40.0019531,34.61751 39.8597803,34.0368953 L38.9987939,30.5201752 C38.7661182,29.569761 39.8037061,28.8159288 40.6356348,29.3309087 L43.7141846,31.2365172 C44.2224512,31.5511556 44.891582,31.3337339 45.1178662,30.7804044 L46.4883545,27.4292211 C46.8587451,26.5236019 48.1412549,26.5236019 48.5116455,27.4292211 Z M54.8046875,40.4999799 C54.8046875,36.4721469 51.5277832,33.1952995 47.5,33.1952995 C43.4721631,33.1952995 40.1953125,36.4721469 40.1953125,40.4999799 C40.1953125,44.5278128 43.4721631,47.8046602 47.5,47.8046602 C51.5277832,47.8046602 54.8046875,44.5278128 54.8046875,40.4999799 Z M53.0859375,40.4999799 C53.0859375,43.5800843 50.5801074,46.0859119 47.5,46.0859119 C44.4198926,46.0859119 41.9140625,43.5800843 41.9140625,40.4999799 C41.9140625,37.4198754 44.4198926,34.9140478 47.5,34.9140478 C50.5801074,34.9140478 53.0859375,37.4198754 53.0859375,40.4999799 Z"/>
    </defs>
    <g fill="none" fill-rule="evenodd">
        <mask id="icon-b" fill="#fff">
            <use xlink:href="#icon-a"/>
        </mask>
        <g mask="url(#icon-b)">
            <rect width="70" height="70" fill="url(#icon-c)"/>
            <path fill="#FFF" fill-opacity=".383" d="M4,1.8 L65,1.8 C67.6666667,1.8 69.3333333,1.13333333 70,-0.2 C70,2.46666667 70,3.46666667 70,2.8 L1.10547097e-14,2.8 C-1.65952376e-14,3.46666667 -2.9161925e-14,2.46666667 -2.66453526e-14,-0.2 C0.666666667,1.13333333 2,1.8 4,1.8 Z" transform="matrix(1 0 0 -1 0 2.8)"/>
            <path fill="#393939" d="M43.6444444,52 L4,52 C2,52 -7.10542736e-15,51.8514286 0,47.84 L2.21121142e-16,23.3197384 L20,0 L36,9.36 L30.6694253,16.3853185 L31.9690606,16.0436983 L28.4526012,19.3069352 L28.4526012,24.5134324 L39.2011791,11.8378465 L42.1902641,14.2434656 L46.8367307,9.36 L51,14.56 L60.3723121,27.0292074 L43.6444444,52 Z" opacity=".324" transform="translate(0 18)"/>
            <path fill="#000" fill-opacity=".383" d="M4,4 L65,4 C67.6666667,4 69.3333333,3 70,1 C70,3.66666667 70,5 70,5 L1.77635684e-15,5 C1.77635684e-15,5 1.77635684e-15,3.66666667 1.77635684e-15,1 C0.666666667,3 2,4 4,4 Z" transform="translate(0 65)"/>
            <use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#icon-d"/>
            <use fill="#FFF" fill-rule="nonzero" xlink:href="#icon-e"/>
        </g>
    </g>
</svg>

```

## File: static\src\components\partner_im_status_icon\partner_im_status_icon.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.PartnerImStatusIcon" t-inherit-mode="extension">
        <xpath expr="//*[@name='rootCondition']" position="inside">
            <t t-if="partner.im_status === 'leave_online'">
                <i class="o_PartnerImStatusIcon_icon o-online fa fa-plane fa-stack-1x" title="Online" role="img" aria-label="User is online"/>
            </t>
            <t t-if="partner.im_status === 'leave_away'">
                <i class="o_PartnerImStatusIcon_icon o-away fa fa-plane fa-stack-1x" title="Away" role="img" aria-label="User is away"/>
            </t>
            <t t-if="partner.im_status === 'leave_offline'">
                <i class="o_PartnerImStatusIcon_icon o-offline fa fa-plane fa-stack-1x" title="Out of office" role="img" aria-label="User is out of office"/>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\components\thread_icon\thread_icon.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.ThreadIcon" t-inherit-mode="extension">
        <xpath expr="//*[@name='noImStatusCondition']" position="before">
            <t t-elif="thread.correspondent.im_status === 'leave_online'">
                <div class="o_ThreadIcon_online fa fa-plane" title="Online"/>
            </t>
            <t t-elif="thread.correspondent.im_status === 'leave_away'">
                <div class="o_ThreadIcon_away fa fa-plane" title="Away"/>
            </t>
            <t t-elif="thread.correspondent.im_status === 'leave_offline'">
                <div class="o_ThreadIcon_offline fa fa-plane" title="Out of office"/>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\components\thread_view\thread_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.ThreadView" t-inherit-mode="extension">
        <xpath expr="//*[@name='loadingCondition']" position="before">
            <t t-if="threadView.thread.correspondent and threadView.thread.correspondent.outOfOfficeText">
                <div class="o_ThreadView_outOfOffice alert alert-primary" t-esc="threadView.thread.correspondent.outOfOfficeText" role="alert"/>
            </t>
        </xpath>
    </t>
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

## File: static\src\img\icons\Refused.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><path d="M21.93,55H78.07a5,5,0,0,0,0-10H21.93a5,5,0,0,0,0,10Z" style="fill:#875b7b"/></svg>
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

## File: static\src\img\icons\To_Approve.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><path d="M83.13,35.69a7.9,7.9,0,0,0-4.94-7.33L34.33,72.22H47.06L83.13,36.15Z" style="fill:#875b7b"/><path d="M53.39,27.78,16.87,64.3h0a7.9,7.9,0,0,0,5.21,7.43l44-44Z" style="fill:#875b7b"/><path d="M24.78,27.78a7.91,7.91,0,0,0-7.91,7.91V51.57L40.66,27.78Z" style="fill:#875b7b"/><path d="M59.78,72.22H75.22a7.91,7.91,0,0,0,7.91-7.91V48.87Z" style="fill:#875b7b"/></svg>
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

## File: static\src\img\icons\Validated.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><rect x="16.87" y="27.78" width="66.26" height="44.44" rx="7.91" style="fill:#875b7b"/></svg>
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

## File: static\src\js\float_without_trailing_zeros.js

```javascript
/** @odoo-module **/
import FieldRegistry from 'web.field_registry';
import basic_fields from 'web.basic_fields';

var FieldFloat = basic_fields.FieldFloat;

var FloatWithoutTrailingZeros = FieldFloat.extend({
    _renderReadonly: function () {
        var value = this._formatValue(this.value);
        var parsed_value = parseFloat(value);
        value = parsed_value.toString().replace(/\.0+$/, '');
        this.$el.text(value);
    }
});

FieldRegistry.add('float_without_trailing_zeros', FloatWithoutTrailingZeros);


```

## File: static\src\js\leave_stats_widget.js

```javascript
odoo.define('hr_holidays.LeaveStatsWidget', function (require) {
    "use strict";

    var time = require('web.time');
    var Widget = require('web.Widget');
    var widget_registry = require('web.widget_registry');
    var fieldUtils = require('web.field_utils');

    var LeaveStatsWidget = Widget.extend({
        template: 'hr_holidays.leave_stats',

        /**
         * @override
         * @param {Widget|null} parent
         * @param {Object} params
         */
        init: function (parent, params) {
            this._setState(params);
            this._super(parent);
        },

        //--------------------------------------------------------------------------
        // Public
        //--------------------------------------------------------------------------

        /**
         * @override to fetch data before rendering.
         */
        willStart: function () {
            return Promise.all([this._super(), this._fetchLeaveTypesData(), this._fetchDepartmentLeaves()]);
        },

        /**
         * Fetch new data if needed (according to updated fields) and re-render the widget.
         * Called by the basic renderer when the view changes.
         * @param {Object} state
         * @returns {Promise}
         */
        updateState: function (state) {
            var self = this;
            var to_await = [];
            var updatedFields = this._setState(state);

            if (_.intersection(updatedFields, ['employee', 'date']).length) {
                to_await.push(this._fetchLeaveTypesData());
            }
            if (_.intersection(updatedFields, ['department', 'date']).length) {
                to_await.push(this._fetchDepartmentLeaves());
            }
            return Promise.all(to_await).then(function () {
                self.renderElement();
            });
        },

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------

        /**
         * Update the state
         * @param {Object} state
         * @returns {String[]} list of updated fields
         */
        _setState: function (state) {
            var updatedFields = [];
            if (state.data.employee_id.res_id !== (this.employee && this.employee.res_id)) {
                updatedFields.push('employee');
                this.employee = state.data.employee_id;
            }
            if (state.data.department_id.res_id !== (this.department && this.department.res_id)) {
                updatedFields.push('department');
                this.department = state.data.department_id;
            }
            if (state.data.date_from !== this.date) {
                updatedFields.push('date');
                this.date = state.data.date_from;
            }
            return updatedFields;
        },

        /**
         * Fetch leaves taken by members of ``this.department`` in the
         * month of ``this.date``.
         * Three fields are fetched for each leave, namely: employee_id, date_from
         * and date_to.
         * The resulting data is assigned to ``this.departmentLeaves``
         * @private
         * @returns {Promise}
         */
        _fetchDepartmentLeaves: function () {
            if (!this.date || !this.department) {
                this.departmentLeaves = null;
                return Promise.resolve();
            }
            var self = this;
            var month_date_from = this.date.clone().startOf('month');
            var month_date_to = this.date.clone().endOf('month');
            return this._rpc({
                model: 'hr.leave',
                method: 'search_read',
                args: [
                    [['department_id', '=', this.department.res_id],
                    ['state', '=', 'validate'],
                    ['holiday_type', '=', 'employee'],
                    ['date_from', '<=', month_date_to],
                    ['date_to', '>=', month_date_from]],
                    ['employee_id', 'date_from', 'date_to', 'number_of_days'],
                ],
            }).then(function (data) {
                var dateFormat = time.getLangDateFormat();
                self.departmentLeaves = data.map(function (leave) {
                    // Format datetimes to date (in the user's format)
                    return _.extend(leave, {
                        date_from: fieldUtils.parse.datetime(
                            leave.date_from,
                            false,
                            { isUTC: true }).local().format(dateFormat),
                        date_to: fieldUtils.parse.datetime(
                            leave.date_to,
                            false,
                            { isUTC: true }).local().format(dateFormat),
                        number_of_days: leave.number_of_days,
                    });
                });
            });
        },

        /**
         * Fetch the number of leaves, grouped by leave type, taken by ``this.employee``
         * in the year of ``this.date``.
         * The resulting data is assigned to ``this.leavesPerType``
         * @private
         * @returns {Promise}
         */
        _fetchLeaveTypesData: function () {
            if (!this.date || !this.employee) {
                this.leavesPerType = null;
                return Promise.resolve();
            }
            var self = this;
            var year_date_from = this.date.clone().startOf('year');
            var year_date_to = this.date.clone().endOf('year');
            return this._rpc({
                model: 'hr.leave',
                method: 'read_group',
                kwargs: {
                    domain: [['employee_id', '=', this.employee.res_id], ['state', '=', 'validate'], ['date_from', '<=', year_date_to], ['date_to', '>=', year_date_from]],
                    fields: ['holiday_status_id', 'number_of_days:sum'],
                    groupby: ['holiday_status_id'],
                },
            }).then(function (data) {
                self.leavesPerType = data;
            });
        }
    });

    widget_registry.add('hr_leave_stats', LeaveStatsWidget);

    return LeaveStatsWidget;
});

```

## File: static\src\js\radio_image.js

```javascript
/** @odoo-module **/

import { FieldRadio } from 'web.relational_fields';
import FieldRegistry from 'web.field_registry';

import Core from 'web.core';
const QWeb = Core.qweb;

const FieldRadioImage = FieldRadio.extend({
    _render() {
        const self = this;
        let currentValue;
        if (this.field.type === 'many2one') {
            currentValue = this.value && this.value.data.id;
        } else {
            currentValue = this.value;
        }
        this.$el.empty();
        this.$el.attr('role', 'radiogroup')
            .attr('aria-label', this.string);
        _.each(this.values, function (value, index) {
            if (self.mode === 'edit' || value[0] === currentValue) {
                self.$el.append(QWeb.render('FieldRadioIcon.button', {
                    checked: value[0] === currentValue,
                    id: self.unique_id + '_' + value[0],
                    index: index,
                    name: self.unique_id,
                    value: value,
                    disabled: self.hasReadonlyModifier,
                    mode: self.mode,
                }));
            }
        });
    }
});

FieldRegistry.add('radio_image', FieldRadioImage);

```

## File: static\src\js\time_off_calendar.js

```javascript
odoo.define('hr_holidays.dashboard.view_custo', function(require) {
    'use strict';

    var core = require('web.core');
    const config = require('web.config');
    var CalendarPopover = require('web.CalendarPopover');
    var CalendarController = require("web.CalendarController");
    var CalendarRenderer = require("web.CalendarRenderer");
    var CalendarModel = require("web.CalendarModel");
    var CalendarView = require("web.CalendarView");
    var dialogs = require('web.view_dialogs');
    var viewRegistry = require('web.view_registry');

    var _t = core._t;
    var QWeb = core.qweb;

    var TimeOffCalendarPopover = CalendarPopover.extend({
        template: 'hr_holidays.calendar.popover',

        init: function (parent, eventInfo) {
            this._super.apply(this, arguments);
            const state = this.event.extendedProps.record.state;
            this.canDelete = state && ['validate', 'refuse'].indexOf(state) === -1;
            this.canEdit = state !== undefined;
            this.displayFields = [];

            if (this.modelName === "hr.leave.report.calendar") {
                const duration = this.event.extendedProps.record.display_name.split(':').slice(-1);
                this.display_name = _.str.sprintf(_t("Time Off : %s"), duration);
            } else {
                this.display_name = this.event.extendedProps.record.display_name;
            }
        },
    });

    var TimeOffCalendarController = CalendarController.extend({
        events: _.extend({}, CalendarController.prototype.events, {
            'click .btn-time-off': '_onNewTimeOff',
            'click .btn-allocation': '_onNewAllocation',
        }),

        /**
         * @override
         */
        start: function () {
            this.$el.addClass('o_timeoff_calendar');
            return this._super(...arguments);
        },

        //--------------------------------------------------------------------------
        // Public
        //--------------------------------------------------------------------------

         /**
         * Render the buttons and add new button about
         * time off and allocations request
         *
         * @override
         */

        renderButtons: function ($node) {
            this._super.apply(this, arguments);

            $(QWeb.render('hr_holidays.dashboard.calendar.button', {
                time_off: _t('New Time Off'),
                request: _t('Allocation Request'),
            })).appendTo(this.$buttons);

            if ($node) {
                this.$buttons.appendTo($node);
            } else {
                this.$('.o_calendar_buttons').replaceWith(this.$buttons);
            }
        },

        //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------

        _getNewTimeOffContext: function() {
            const { date_from, date_to } = this.model._getTimeOffDates(moment());
            return {
                'default_date_from': date_from,
                'default_date_to': date_to,
                'lang': this.context.lang
            }
        },

        /**
         * Action: create a new time off request
         *
         * @private
         */
        _onNewTimeOff: function () {
            this._rpc({
                model: 'ir.ui.view',
                method: 'get_view_id',
                args: ['hr_holidays.hr_leave_view_form_dashboard_new_time_off'],
            }).then((ids) => {
                this.timeOffDialog = new dialogs.FormViewDialog(this, {
                    res_model: "hr.leave",
                    view_id: ids,
                    context: this._getNewTimeOffContext(),
                    title: _t("New time off"),
                    disable_multiple_selection: true,
                    on_saved: () => {
                        this.reload();
                    },
                });
                this.timeOffDialog.open();
            });
        },

        /**
         * Action: create a new allocation request
         *
         * @private
         */
        _onNewAllocation: function () {
            let self = this;

            self._rpc({
                model: 'ir.ui.view',
                method: 'get_view_id',
                args: ['hr_holidays.hr_leave_allocation_view_form_dashboard'],
            }).then(function(ids) {
                self.allocationDialog = new dialogs.FormViewDialog(self, {
                    res_model: "hr.leave.allocation",
                    view_id: ids,
                    context: {
                        'default_state': 'confirm',
                        'lang': self.context.lang,
                    },
                    title: _t("New Allocation"),
                    disable_multiple_selection: true,
                    on_saved: function() {
                        self.reload();
                    },
                });
                self.allocationDialog.open();
            });
        },

        /**
         * @override
         */
        _setEventTitle: function () {
            return _t('Time Off Request');
        },
    });

    var TimeOffPopoverRenderer = CalendarRenderer.extend({
        template: "TimeOff.CalendarView.extend",
        /**
         * We're overriding this to display the weeknumbers on the year view
         *
         * @override
         * @private
         */
        _getFullCalendarOptions: function () {
            const oldOptions = this._super(...arguments);
            // Parameters
            oldOptions.views.dayGridYear.weekNumbers = true;
            oldOptions.views.dayGridYear.weekNumbersWithinDays = false;
            return oldOptions;
        },

        config: _.extend({}, CalendarRenderer.prototype.config, {
            CalendarPopover: TimeOffCalendarPopover,
        }),

        _getPopoverParams: function (eventData) {
            let params = this._super.apply(this, arguments);
            let calendarIcon;
            let state = eventData.extendedProps.record.state;

            if (state === 'validate') {
                calendarIcon = 'fa-calendar-check-o';
            } else if (state === 'refuse') {
                calendarIcon = 'fa-calendar-times-o';
            } else if(state) {
                calendarIcon = 'fa-calendar-o';
            }

            params['title'] = eventData.extendedProps.record.display_name.split(':').slice(0, -1).join(':');
            params['template'] = QWeb.render('hr_holidays.calendar.popover.placeholder', {color: this.getColor(eventData.color_index), calendarIcon: calendarIcon});
            return params;
        },

        _render: function () {
            var self = this;
            return this._super.apply(this, arguments).then(function () {
                self.$el.parent().find('.o_calendar_mini').hide();
            });
        },
        /**
         * @override
         * @private
         */
        _renderCalendar: function() {
            this._super.apply(this, arguments);
            let weekNumbers = this.$el.find('.fc-week-number');
            weekNumbers.each( function() {
                let weekRow = this.parentNode;
                // By default, each month has 6 weeks displayed, hide the week number if there is no days for the week
                if(!weekRow.children[1].classList.length && !weekRow.children[weekRow.children.length-1].classList.length) {
                    this.innerHTML = '';
                }
            });
        },
    });

    var TimeOffCalendarRenderer = TimeOffPopoverRenderer.extend({
        _render: function () {
            var self = this;
            return this._super.apply(this, arguments).then(function () {
                return self._rpc({
                    model: 'hr.leave.type',
                    method: 'get_days_all_request',
                    context: self.state.context,
                });
            }).then(function (result) {
                self.$el.parent().find('.o_calendar_mini').hide();
                self.$el.parent().find('.o_timeoff_container').remove();

                // Do not display header if there is no element to display
                if (result.length > 0) {
                    if (config.device.isMobile) {
                        result.forEach((data) => {
                            const elem = QWeb.render('hr_holidays.dashboard_calendar_header_mobile', {
                                timeoff: data,
                            });
                            self.$el.find('.o_calendar_filter_item[data-value=' + data[3] + '] .o_cw_filter_title').append(elem);
                        });
                    } else {
                        const elem = QWeb.render('hr_holidays.dashboard_calendar_header', {
                            timeoffs: result,
                        });
                        self.$el.before(elem);
                    }
                }
            });
        },
    });

    const TimeOffCalendarModel = CalendarModel.extend({
        calendarEventToRecord(event) {
            const res = this._super(...arguments);
            if (['day', 'week'].includes(this.data.scale)) {
                const { date_from, date_to } = this._getTimeOffDates(event.start.clone());

                res['date_from'] = date_from;
                res['date_to'] = date_to;
            }

            return res;
        },

        _getTimeOffDates(date_from) {
            date_from.set({
                'hour': 0,
                'minute': 0,
                'second': 0
            });
            let date_to = date_from.clone().set({
                'hour': 23,
                'minute': 59,
                'second': 59
            });

            date_from.subtract(this.getSession().getTZOffset(date_from), 'minutes');
            date_from = date_from.locale('en').format('YYYY-MM-DD HH:mm:ss');
            date_to.subtract(this.getSession().getTZOffset(date_to), 'minutes');
            date_to = date_to.locale('en').format('YYYY-MM-DD HH:mm:ss');

            return {
                date_from,
                date_to,
            }
        },
    });

    var TimeOffCalendarView = CalendarView.extend({
        config: _.extend({}, CalendarView.prototype.config, {
            Controller: TimeOffCalendarController,
            Renderer: TimeOffCalendarRenderer,
            Model: TimeOffCalendarModel,
        }),
    });

    /**
     * Calendar shown in the "Everyone" menu
     */
    var TimeOffCalendarAllView = CalendarView.extend({
        config: _.extend({}, CalendarView.prototype.config, {
            Controller: TimeOffCalendarController,
            Renderer: TimeOffPopoverRenderer,
            Model: TimeOffCalendarModel,
        }),
    });

    viewRegistry.add('time_off_calendar', TimeOffCalendarView);
    viewRegistry.add('time_off_calendar_all', TimeOffCalendarAllView);
});

```

## File: static\src\js\time_off_calendar_employee.js

```javascript
odoo.define('hr_holidays.employee.dashboard.views', function(require) {
    'use strict';

    var core = require('web.core');
    const config = require('web.config');
    var CalendarController = require("web.CalendarController");
    var CalendarRenderer = require("web.CalendarRenderer");
    var CalendarPopover = require('web.CalendarPopover');
    var CalendarView = require("web.CalendarView");
    var dialogs = require('web.view_dialogs');
    var viewRegistry = require('web.view_registry');

    var _t = core._t;
    var QWeb = core.qweb;

    // TODO in master: Split the timeoff calendar components into several files to reuse or extend them here

    var TimeOffCalendarPopover = CalendarPopover.extend({
        template: 'hr_holidays.calendar.popover',

        init: function (parent, eventInfo) {
            this._super.apply(this, arguments);
            const state = this.event.extendedProps.record.state;
            this.canDelete = state && ['validate', 'refuse'].indexOf(state) === -1;
            this.canEdit = state !== undefined;
            this.displayFields = [];

            if (this.modelName === "hr.leave.report.calendar") {
                const duration = this.event.extendedProps.record.display_name.split(':').slice(-1);
                this.display_name = _.str.sprintf(_t("Time Off : %s"), duration);
            } else {
                this.display_name = this.event.extendedProps.record.display_name;
            }
        },
    });

    var TimeOffPopoverRenderer = CalendarRenderer.extend({
        template: "TimeOff.CalendarView.extend",
        /**
         * We're overriding this to display the weeknumbers on the year view
         *
         * @override
         * @private
         */
        _getFullCalendarOptions: function () {
            const oldOptions = this._super(...arguments);
            // Parameters
            oldOptions.views.dayGridYear.weekNumbers = true;
            oldOptions.views.dayGridYear.weekNumbersWithinDays = false;
            return oldOptions;
        },

        config: _.extend({}, CalendarRenderer.prototype.config, {
            CalendarPopover: TimeOffCalendarPopover,
        }),

        _getPopoverParams: function (eventData) {
            let params = this._super.apply(this, arguments);
            let calendarIcon;
            let state = eventData.extendedProps.record.state;

            if (state === 'validate') {
                calendarIcon = 'fa-calendar-check-o';
            } else if (state === 'refuse') {
                calendarIcon = 'fa-calendar-times-o';
            } else if(state) {
                calendarIcon = 'fa-calendar-o';
            }

            params['title'] = eventData.extendedProps.record.display_name.split(':').slice(0, -1).join(':');
            params['template'] = QWeb.render('hr_holidays.calendar.popover.placeholder', {color: this.getColor(eventData.color_index), calendarIcon: calendarIcon});
            return params;
        },

        _render: function () {
            var self = this;
            return this._super.apply(this, arguments).then(function () {
                self.$el.parent().find('.o_calendar_mini').hide();
            });
        },
        /**
         * @override
         * @private
         */
        _renderCalendar: function() {
            this._super.apply(this, arguments);
            let weekNumbers = this.$el.find('.fc-week-number');
            weekNumbers.each( function() {
                let weekRow = this.parentNode;
                // By default, each month has 6 weeks displayed, hide the week number if there is no days for the week
                if(!weekRow.children[1].classList.length && !weekRow.children[weekRow.children.length-1].classList.length) {
                    this.innerHTML = '';
                }
            });
        },
    });

    var TimeOffCalendarRenderer = TimeOffPopoverRenderer.extend({
        _render: function () {
            var self = this;
            return this._super.apply(this, arguments).then(function () {
                return self._rpc({
                    model: 'hr.leave.type',
                    method: 'get_days_all_request',
                    context: self.state.context,
                });
            }).then(function (result) {
                self.$el.parent().find('.o_calendar_mini').hide();
                self.$el.parent().find('.o_timeoff_container').remove();

                // Do not display header if there is no element to display
                if (result.length > 0) {
                    if (config.device.isMobile) {
                        result.forEach((data) => {
                            const elem = QWeb.render('hr_holidays.dashboard_calendar_header_mobile', {
                                timeoff: data,
                            });
                            self.$el.find('.o_calendar_filter_item[data-value=' + data[4] + '] .o_cw_filter_title').append(elem);
                        });
                    } else {
                        const elem = QWeb.render('hr_holidays.dashboard_calendar_header', {
                            timeoffs: result,
                        });
                        self.$el.before(elem);
                    }
                }
            });
        },
    });

    var TimeOffCalendarEmployeeController = CalendarController.extend({

        events: _.extend({}, CalendarController.prototype.events, {
            'click .btn-time-off': '_onNewTimeOff',
            'click .btn-allocation': '_onNewAllocation',
        }),

        /**
         * @override
         */
         start: function () {
            this.$el.addClass('o_timeoff_calendar');
            return this._super(...arguments);
        },

        //--------------------------------------------------------------------------
        // Public
        //--------------------------------------------------------------------------

         /**
         * Render the buttons and add new button about
         * time off and allocations request
         *
         * @override
         */

        renderButtons: function ($node) {
            this._super.apply(this, arguments);

            $(QWeb.render('hr_holidays.dashboard.calendar.button', {
                time_off: _t('New Time Off'),
                request: _t('New Allocation Request'),
            })).appendTo(this.$buttons);

            if ($node) {
                this.$buttons.appendTo($node);
            } else {
                this.$('.o_calendar_buttons').replaceWith(this.$buttons);
            }
        },

        //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------

        _getNewTimeOffContext: function() {
            let date_from = moment().set({
                'hour': 0,
                'minute': 0,
                'second': 0
            });
            date_from.subtract(this.getSession().getTZOffset(date_from), 'minutes');
            date_from = date_from.locale('en').format('YYYY-MM-DD HH:mm:ss');
            let date_to = moment().set({
                'hour': 23,
                'minute': 59,
                'second': 59
            });
            date_to.subtract(this.getSession().getTZOffset(date_to), 'minutes');
            date_to = date_to.locale('en').format('YYYY-MM-DD HH:mm:ss');
            return {
                'default_date_from': date_from,
                'default_date_to': date_to,
                'lang': this.context.lang,
                'default_employee_id': this.context.employee_id[0],
            }
        },

        /**
         * Action: create a new time off request
         *
         * @private
         */
        _onNewTimeOff: function () {
            this._rpc({
                model: 'ir.ui.view',
                method: 'get_view_id',
                args: ['hr_holidays.hr_leave_view_form_dashboard_new_time_off'],
            }).then((ids) => {
                this.timeOffDialog = new dialogs.FormViewDialog(this, {
                    res_model: "hr.leave",
                    view_id: ids,
                    context: this._getNewTimeOffContext(this),
                    title: _t("New time off"),
                    disable_multiple_selection: true,
                    on_saved: () => {
                        this.reload();
                    },
                });
                this.timeOffDialog.open();
            });
        },

        /**
         * Action: create a new allocation request
         *
         * @private
         */
        _onNewAllocation: function () {
            let self = this;

            self._rpc({
                model: 'ir.ui.view',
                method: 'get_view_id',
                args: ['hr_holidays.hr_leave_allocation_view_form_manager_dashboard'],
            }).then(function(ids) {
                self.allocationDialog = new dialogs.FormViewDialog(self, {
                    res_model: "hr.leave.allocation",
                    view_id: ids,
                    context: {
                        'default_employee_ids': self.context.employee_id,
                        'default_state': 'confirm',
                        'lang': self.context.lang,
                    },
                    title: _t("New Allocation"),
                    disable_multiple_selection: true,
                    on_saved: function() {
                        self.reload();
                    },
                });
                self.allocationDialog.open();
            });
        },

        _onOpenCreate: function () {
            this.context['default_employee_id'] = this.context.employee_id[0];
            this._super(...arguments);
        },

        /**
         * @override
         */
         _setEventTitle: function () {
            return _t('Time Off Request');
        },
    });

    var TimeOffCalendarEmployeeView = CalendarView.extend({
        config: _.extend({}, CalendarView.prototype.config, {
            Controller: TimeOffCalendarEmployeeController,
            Renderer: TimeOffCalendarRenderer,
        }),
    });

    viewRegistry.add('time_off_employee_calendar', TimeOffCalendarEmployeeView);
});

```

## File: static\src\models\partner\partner.js

```javascript
odoo.define('hr_holidays/static/src/models/partner/partner.js', function (require) {
'use strict';

const {
    registerClassPatchModel,
    registerFieldPatchModel,
    registerInstancePatchModel,
} = require('@mail/model/model_core');
const { attr } = require('@mail/model/model_field');
const { clear } = require('@mail/model/model_field_command');

const { str_to_date } = require('web.time');

registerClassPatchModel('mail.partner', 'hr_holidays/static/src/models/partner/partner.js', {
    /**
     * @override
     */
    convertData(data) {
        const data2 = this._super(data);
        if ('out_of_office_date_end' in data) {
            data2.outOfOfficeDateEnd = data.out_of_office_date_end ? data.out_of_office_date_end : clear();
        }
        return data2;
    },
});

registerInstancePatchModel('mail.partner', 'hr_holidays/static/src/models/partner/partner.js', {
    /**
     * @private
     */
    _computeOutOfOfficeText() {
        if (!this.outOfOfficeDateEnd) {
            return clear();
        }
        if (!this.messaging.locale || !this.messaging.locale.language) {
            return clear();
        }
        const currentDate = new Date();
        const date = str_to_date(this.outOfOfficeDateEnd);
        const options = { day: 'numeric', month: 'short' };
        if (currentDate.getFullYear() !== date.getFullYear()) {
            options.year = 'numeric';
        }
        let localeCode = this.messaging.locale.language.replace(/_/g, '-');
        if (localeCode == "sr@latin") {
            localeCode = "sr-Latn-RS";
        }
        const formattedDate = date.toLocaleDateString(localeCode, options);
        return _.str.sprintf(this.env._t("Out of office until %s"), formattedDate);
    },
    /**
     * @override
     */
    _computeIsOnline() {
        if (['leave_online', 'leave_away'].includes(this.im_status)) {
            return true;
        }
        return this._super();
    },
});

registerFieldPatchModel('mail.partner', 'hr/static/src/models/partner/partner.js', {
    /**
     * Date of end of the out of office period of the partner as string.
     * String is expected to use Odoo's date string format
     * (examples: '2011-12-01' or '2011-12-01').
     */
    outOfOfficeDateEnd: attr(),
    /**
     * Text shown when partner is out of office.
     */
    outOfOfficeText: attr({
        compute: '_computeOutOfOfficeText',
    }),
});

});

```

## File: static\src\xml\leave_stats_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates id="template" xml:space="preserve">
    <t t-name="hr_holidays.leave_per_type">
        <table class="o_group o_inner_group table-striped">
        <thead>
            <tr>
                <td colspan="2">
                    <div class="o_horizontal_separator"><t t-esc="widget.employee.data.display_name"/> in <t t-esc="widget.date.format('YYYY')"/></div>
                </td>
            </tr>
        </thead>
        <tbody>
            <t t-if="widget.leavesPerType.length === 0">
                <tr>
                    <td>None</td>
                </tr>
            </t>
            <t t-foreach="widget.leavesPerType" t-as="leave_type">
                <tr>
                    <td><t t-esc="leave_type.holiday_status_id[1]"/></td>
                    <td  class="w-50"><t t-esc="leave_type.number_of_days"/> day(s)</td>
                </tr>
            </t>
        </tbody>
        </table>
    </t>

    <t t-name="hr_holidays.department_leave">
        <table class="o_group o_inner_group table-striped">
        <thead>
            <tr>
                <td colspan="2">
                    <div class="o_horizontal_separator"><t t-esc="widget.department.data.display_name"/> in <t t-esc="widget.date.format('MMMM')"/></div>
                </td>
            </tr>
        </thead>
        <tbody>
            <t t-if="widget.departmentLeaves.length === 0">
                <tr>
                    <td>None</td>
                </tr>
            </t>
            <t t-foreach="widget.departmentLeaves" t-as="leave">
                <tr t-attf-class="{{leave.employee_id[0] === widget.employee.res_id ? 'font-weight-bold' : ''}}">
                    <td><t t-esc="leave.employee_id[1]"/>: <t t-esc="leave.number_of_days"/> day(s) </td>
                    <td class="w-50"><t t-esc="leave.date_from"/> - <t t-esc="leave.date_to"/></td>
                </tr>
            </t>
        </tbody>
        </table>
    </t>

    <div t-name="hr_holidays.leave_stats" class="o_leave_stats">
        <t t-if="widget.employee">
            <t t-if="widget.leavesPerType">
                <t t-call="hr_holidays.leave_per_type"/>
            </t>
            <t t-if="widget.departmentLeaves">
                <t t-call="hr_holidays.department_leave"/>
            </t>
        </t>
    </div>

</templates>

```

## File: static\src\xml\time_off_calendar.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates id="template" xml:space="preserve">
    <t t-name="hr_holidays.dashboard_calendar_header">
        <div class="o_timeoff_container d-flex">
            <div t-foreach="timeoffs" t-as="timeoff" t-attf-class="o_timeoff_card flex-grow-1 d-flex flex-column py-3 {{ timeoff_last ? 'o_timeoff_card_last' : '' }}">
                <t t-set="requires_allocation" t-value="timeoff[2] === 'yes'"/>
                <t t-set="has_icon" t-value="timeoff[1]['icon'] !== false"/>
                <t t-set="cl" t-value="'text-muted'"/>

                <b><span t-esc="timeoff[0]" class="o_timeoff_name o_timeoff_big"/></b>

                <div class="mt-1">
                    <t t-if="requires_allocation">
                        <t t-if="has_icon"><img height="30px" t-attf-src="{{timeoff[1]['icon']}}"/></t><span t-esc="timeoff[1]['virtual_remaining_leaves']" class="o_timeoff_huge o_timeoff_purple font-weight-bold align-middle"/><br/>
                        <t class="o_timeoff_big o_timeoff_purple" t-if="timeoff[1]['request_unit'] == 'hour'"><span class="o_timeoff_purple">HOURS</span></t><t class="o_timeoff_big o_timeoff_purple" t-else=""><span class="o_timeoff_purple">DAYS</span></t> <span class="o_timeoff_purple">AVAILABLE </span>
                    </t>
                    <t t-else="">
                        <t t-if="has_icon"><img height="30px" t-attf-src="{{timeoff[1]['icon']}}"/></t><span t-esc="timeoff[1]['virtual_leaves_taken']" class="o_timeoff_huge o_timeoff_purple font-weight-bold align-middle"/><br/>
                        <t t-if="timeoff[1]['request_unit'] == 'hour'"><span class="o_timeoff_purple">HOURS</span></t><t t-else=""><span class="o_timeoff_purple">DAYS</span></t> <span class="o_timeoff_purple">TAKEN</span>
                    </t>
                </div>
            </div>
        </div>
    </t>

    <t t-name="hr_holidays.dashboard_calendar_header_mobile">
        <t t-set="requires_allocation" t-value="timeoff[2] === 'yes'"/>
        <span class="pull-right">
            <span>
                <t t-if="requires_allocation">
                    <strong t-esc="timeoff[1]['virtual_remaining_leaves']" class="o_timeoff_green"/> / <span t-esc="timeoff[1]['max_leaves']"/>
                    <t t-if="timeoff[1]['request_unit'] == 'hour'">Hours</t><t t-else="">Days</t><span class="o_timeoff_green ml-2">Available </span>
                </t>
                <t t-else="">
                    <strong t-esc="timeoff[1]['virtual_leaves_taken']"/> <t t-if="timeoff[1]['request_unit'] == 'hour'">Hours</t><t t-else="">Days</t><span class="o_timeoff_purple ml-2">Taken</span>
                </t>
            </span>
        </span>
    </t>

    <t t-name="hr_holidays.dashboard.calendar.button">
        <button class="btn btn-primary btn-time-off" type="button">
            <t t-esc="time_off"/>
        </button>
        <button class="btn btn-secondary btn-allocation" type="button">
            <t t-esc="request"/>
        </button>
    </t>

    <t t-name="hr_holidays.calendar.popover.placeholder">
        <div t-attf-class="o_cw_popover popover card shadow #{typeof color === 'number' ? _.str.sprintf('o_calendar_color_%s', color) : ''}" role="tooltip">
            <div class="arrow"/>
            <div class="card-header d-flex justify-content-between py-2 pr-2">
                <h4 class="popover-header border-0 p-0 pt-1"/>
                <div class="ml-4">
                    <i t-if="calendarIcon" t-attf-class="fa {{calendarIcon}}"></i>
                    <span class="o_cw_popover_close ml-1"><i class="fa fa-close small"/></span>
                </div>
            </div>
            <div class="o_cw_body">
            </div>
        </div>
    </t>

    <t t-name="hr_holidays.calendar.popover">
        <div class="o_cw_body">
            <ul class="list-group list-group-flush">
                <li t-if="!widget.hideDate and widget.eventDate.date" class="list-group-item">
                    <b class="text-capitalize" t-esc="widget.eventDate.date"/> <small t-if="widget.eventDate.duration"><b t-esc="_.str.sprintf('(%s)', widget.eventDate.duration)"/></small>
                </li>
                <li t-if="!widget.hideTime and widget.eventTime.time" class="list-group-item">
                    <b t-esc="widget.eventTime.time"/> <small t-if="widget.eventTime.duration"><b t-esc="_.str.sprintf('(%s)', widget.eventTime.duration)"/></small>
                </li>
            </ul>
            <ul class="list-group list-group-flush o_cw_popover_fields_secondary" t-if="widget.display_name">
                <li class="list-group-item">
                    <span class="o_field_char o_field_widget" t-esc="widget.display_name" />
                </li>
            </ul>
            <div class="card-footer border-top" t-if="widget.canEdit or widget.canDelete">
                <a t-if="widget.canEdit" href="#" class="btn btn-primary o_cw_popover_edit">Edit</a>
                <a t-if="widget.canDelete" href="#" class="btn btn-secondary o_cw_popover_delete ml-2">Delete</a>
            </div>
        </div>
    </t>

    <t t-name="FieldRadioIcon.button">
        <div t-if="mode === 'edit'" class="custom-control custom-radio o_radio_item" aria-atomic="true">
            <input type="radio" class="custom-control-input o_radio_input" t-att-checked="checked ? true : undefined" t-att-disabled="disabled ? true : undefined"
                t-att-name="name" t-att-data-value="value[0]" t-att-data-index="index" t-att-id="id"/>
            <label class="custom-control-label o_form_label" t-att-for="id"><img t-attf-src="/hr_holidays/static/src/img/icons/{{value[1]}}" width="48" height="48" /></label>
        </div>
        <div t-else="">
            <img t-attf-src="/hr_holidays/static/src/img/icons/{{value[1]}}" width="48" height="48"/>
        </div>
    </t>

    <t t-name="TimeOff.CalendarView.extend" t-extend="CalendarView">
        <t t-jquery='.o_calendar_sidebar_container' t-operation="append">
            <div class="o_timeoff_legend">
                <h5>Legend</h5>
                <img src="/hr_holidays/static/src/img/icons/Validated.svg" width="30px"/><span>Validated</span><br/>
                <img src="/hr_holidays/static/src/img/icons/To_Approve.svg" width="30px"/><span>To Approve</span><br/>
                <img src="/hr_holidays/static/src/img/icons/Refused.svg" width="30px"/><span>Refused</span><br/>
            </div>
        </t>
    </t>

</templates>

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
        name="My Time Off"
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
        id="menu_hr_holidays_approvals"
        name="Approvals"
        parent="menu_hr_holidays_root"
        sequence="3"
        groups="hr_holidays.group_hr_holidays_responsible"/>

    <menuitem
        id="menu_open_department_leave_approve"
        name="Time Off"
        parent="menu_hr_holidays_approvals"
        action="hr_leave_action_action_approve_department"
        sequence="1"/>

    <menuitem
        id="hr_holidays_menu_manager_approve_allocations"
        name="Allocations"
        parent="menu_hr_holidays_approvals"
        action="hr_leave_allocation_action_approve_department"
        sequence="2"/>

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
        action="act_hr_employee_holiday_request"
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
                    <group>
                        <field name="sequence" invisible="1" force_save="1"/>
                        <label for="start_count"/>
                        <div class="o_col">
                            <div class="o_row">
                                <field name="start_count"/>
                                <field name="start_type" nolabel="1"/><label for="start_type" string="after allocation date" class="o_form_label"/>
                            </div>
                        </div>
                        <field name="is_based_on_worked_time"/>
                        <label for="added_value"/>
                        <div class="o_col">
                            <div class="o_row">
                                <field name="added_value"/>
                                <field name="added_value_type" nolabel="1"/>
                            </div>
                        </div>
                        <label for="frequency"/>
                        <div class="o_col">
                            <field name="frequency"/>
                            <div class="o_row" attrs="{'invisible': [('frequency', '!=', 'weekly')]}">
                                <label for="week_day" string="on"/><field name="week_day"/>
                            </div>
                            <div class="o_row" attrs="{'invisible': [('frequency', '!=', 'monthly')]}">
                                <label for="first_day_display" string="on the"/><field name="first_day_display" required="1"/> of the month
                            </div>
                            <div class="o_row" attrs="{'invisible': [('frequency', '!=', 'bimonthly')]}">
                                <label for="first_day_display" string="on the"/><field name="first_day_display" required="1"/> and on the <field name="second_day_display" required="1"/> of the month
                            </div>
                            <div class="o_row" attrs="{'invisible': [('frequency', '!=', 'biyearly')]}">
                                <label for="first_month_day_display" string="on the"/><field name="first_month_day_display" required="1"/> of <field name="first_month"/> and on the <field name="second_month_day_display" required="1"/> of <field name="second_month"/>
                            </div>
                            <div class="o_row" attrs="{'invisible': [('frequency', '!=', 'yearly')]}">
                                <label for="yearly_day_display" string="on the"/><field name="yearly_day_display" required="1"/> of <field name="yearly_month"/>
                            </div>
                        </div>
                        <label for="maximum_leave"/>
                        <div class="o_col">
                            <div class="o_row">
                                <field name="maximum_leave"/>
                                <!-- force_save is required here since it would otherwise not be saved due to the readonly
                                     there is an issue when the field is present twice in the view. -->
                                <field name="added_value_type" nolabel="1" readonly="1" force_save="1"/>
                            </div>
                        </div>
                        <field name="action_with_unused_accruals"/>
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
                <field name="time_off_type_id"/>
                <field name="employees_count"/>
            </tree>
        </field>
    </record>
    <record id="hr_accrual_plan_view_form" model="ir.ui.view">
        <field name="name">hr.leave.accrual.plan.form</field>
        <field name="model">hr.leave.accrual.plan</field>
        <field name="arch" type="xml">
            <form string="Accrual Plan">
                <sheet>
                    <div class="oe_button_box" name="button_box" attrs="{'invisible': [('id', '=', False)]}">
                        <button name="action_open_accrual_plan_employees" type="object" class="oe_stat_button" icon="fa-users">
                            <field name="employees_count" widget="statinfo"/>
                        </button>
                    </div>
                    <group>
                        <group>
                            <field name="name"/>
                        </group>
                        <group>
                            <field name="time_off_type_id"/>
                        </group>
                        <group>
                            <field name="transition_mode" widget="radio"/>
                        </group>
                    </group>
                    <span class="oe_grey" invisible="1">
                        
                    </span>
                    <div class="o_hr_holidays_hierarchy">
                        <div class="o_hr_holidays_title">Rules</div>
                        <div class="o_hr_holidays_hierarchy_readonly oe_read_only" attrs="{'invisible': [('level_ids', '!=', [])]}">
                            <h3>No rules added</h3>
                            <p>
                                Click on the 'Edit' button to add new rules.
                            </p>
                        </div>
                        <field name="level_ids" mode="kanban" nolabel="1"
                            
                            class="o_hr_holidays_plan_level_container o_hr_holidays_plan_level_hierarchy"
                            add-label="Add a new level"
                            >
                            <kanban default_order="level">
                                <field name="start_count"/>
                                <field name="start_type"/>
                                <field name="level"/>
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
                                <field name="is_based_on_worked_time"/>
                                <templates>
                                    <div t-name="kanban-box">
                                        <div class="o_hr_holidays_body oe_kanban_global_click">
                                            <div class="o_hr_holidays_timeline text-center">
                                                Level <field name="level"/>
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
                                            <div class="content">
                                                <div>
                                                    <t t-if="record.start_count.value > 0">
                                                        Starts <field name="start_count" widget="integer"/> <field name="start_type"/> after allocation start date
                                                    </t>
                                                    <t t-else="">
                                                        Starts immediately after allocation start date
                                                    </t>
                                                </div>
                                                <div>
                                                    Adds <field name="added_value" widget="float_without_trailing_zeros"/> <field name="added_value_type"/>
                                                    <t t-if="record.is_based_on_worked_time.raw_value">(based on worked time)</t>
                                                </div>
                                                <div>
                                                    <field name="frequency"/>
                                                    <t t-if="record.frequency.raw_value == 'weekly'">
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
                                                <div t-if="record.maximum_leave.value">
                                                    Limit of <field name="maximum_leave" widget="float_without_trailing_zeros"/> <field name="added_value_type"/>
                                                </div>
                                                <div t-if="record.action_with_unused_accruals.raw_value">
                                                    At the end of the year, unused accruals will be <t t-if="record.action_with_unused_accruals.raw_value == 'postponed'">postponed</t><t t-else="">lost</t>
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

    <record id="open_view_accrual_plans" model="ir.actions.act_window">
        <field name="name">Accrual Plans</field>
        <field name="type">ir.actions.act_window</field>
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
                <filter string="Approved Allocations" domain="[('state', '=', 'validate')]" name="validated"/>
                <filter string="Accrual Allocations" domain="[('allocation_type', '=', 'accrual')]" name="accrual_allocation"/>
                <separator/>
                <filter name="active_types" string="Active Types" domain="[('holiday_status_id.active', '=', True)]" help="Filters only on allocations that belong to an time off type that is 'active' (active field is True)"/>
                <separator/>
                <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction','=',True)]"/>
                <separator/>
                <filter string="My Team" name="my_team" domain="['|', ('employee_id.leave_manager_id', '=', uid), ('employee_id.user_id', '=', uid)]" help="Time off of people you are manager of"/>
                <filter string="My Department" name="my_team_leaves" domain="[('employee_id.parent_id.user_id', '=', uid)]" groups="hr_holidays.group_hr_holidays_manager" help="Time Off of Your Team Member"/>
                <separator/>
                <filter string="Active Employee" name="active_employee" domain="[('active_employee','=',True)]"/>
                <filter string="Archived" name="inactive" domain="[('active','=',False)]"/>
                <separator/>
                <filter name="year" string="Current Year"
                    domain="[('holiday_status_id.active', '=', True)]" help="Active Allocations"/>
                <separator/>
                <filter string="My Allocations" name="my_leaves" domain="[('employee_id.user_id', '=', uid)]"/>
                <separator/>
                <field name="department_id" operator="child_of"/>
                <field name="holiday_status_id"/>
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
                <field name="can_reset" invisible="1"/>
                <field name="can_approve" invisible="1"/>
                <field name="holiday_type" invisible="1"/>
                <header>
                    <button string="Confirm" name="action_confirm" states="draft" type="object" class="oe_highlight"/>
                    <field name="state" widget="statusbar" statusbar_visible="draft,confirm,validate"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button
                            class="oe_stat_button"
                            icon="fa-calendar"
                            type="action"
                            invisible="1"
                            name="%(hr_leave_action_holiday_allocation_id)d"
                            help="Time off Taken/Total Allocated">
                            <div class="o_stat_info">
                                <span class="o_stat_value">
                                    <field name="leaves_taken" digits="[42,1]"/>/<field name="max_leaves" digits="[42,1]"/>
                                </span>
                                <span class="o_stat_text">
                                    Time Off
                                </span>
                            </div>
                        </button>
                    </div>
                    <div id="title" class="oe_title">
                        <h2><field name="name" placeholder="e.g. Time Off type (From validity start to validity end / no limit)" attrs="{'invisible': [('state', 'not in', ('draft', 'confirm'))]}" required="1"/>
                        <field name="name_validity" attrs="{'invisible': [('state', 'in', ('draft', 'confirm'))]}"/></h2>
                    </div>
                    <group>
                        <group>
                            <field name="type_request_unit" invisible="1"/>
                            <field name="holiday_status_id" context="{'employee_id':employee_id, 'default_date_from':current_date}"/>

                            <field name="allocation_type" invisible="1" widget="radio"
                                attrs="{'readonly': ['|', ('is_officer', '=', False), ('state', 'not in', ('draft', 'confirm'))]}"/>
                            <field name="is_officer" invisible="1"/>
                            <field name="accrual_plan_id" attrs="{'invisible': [('allocation_type', '=', 'regular')], 'readonly': ['|', ('is_officer', '=', False), ('state', 'not in', ('draft', 'confirm'))]}"/>
                            <div class="o_td_label">
                                <label for="date_from" string="Validity Period" attrs="{'invisible': ['|', ('allocation_type', '=', 'accrual'), ('state', 'not in', ('draft', 'confirm'))]}"/>
                                <label for="date_from" string="Start Date" attrs="{'invisible': [('allocation_type', '=', 'regular')]}"/>
                            </div>
                            <div class="o_row" name="validity">
                                <field name="date_from" widget="date" nolabel="1" readonly="1" attrs="{'invisible': ['&amp;', ('allocation_type', '=', 'regular'), ('state', 'not in', ('draft', 'confirm'))]}"/>
                                <i class="fa fa-long-arrow-right mx-2" aria-label="Arrow icon" title="Arrow" attrs="{'invisible': ['|', ('allocation_type', '=', 'accrual'), ('state', 'not in', ('draft', 'confirm'))]}"/>
                                <label class="mx-2" for="date_to" string="Run until" attrs="{'invisible': [('allocation_type', '=', 'regular')]}"/>
                                <field name="date_to" widget="date" nolabel="1" readonly="1" placeholder="No Limit"  attrs="{'invisible': ['&amp;', ('allocation_type', '=', 'regular'), ('state', 'not in', ('draft', 'confirm'))]}"/>
                                <div id="no_limit_label" class="oe_read_only" attrs="{'invisible': ['|', '|', ('id', '=', False), ('date_to', '!=', False), ('state', 'not in', ('draft', 'confirm'))]}">No limit</div>
                            </div>

                            <field name="number_of_days" invisible="1"/>
                            <div class="o_td_label">
                                <label for="number_of_days" string="Duration"
                                    attrs="{'readonly': [('allocation_type', '=', 'accrual'), ('state', 'not in', ('draft', 'confirm'))]}"/>
                            </div>
                            <div name="duration_display">
                                <field name="number_of_days_display" class="oe_inline" nolabel="1"
                                    attrs="{'readonly': ['|', '|', ('type_request_unit', '=', 'hour'), ('state', 'not in', ('draft', 'confirm')), ('allocation_type', '=', 'accrual')], 'invisible': [('type_request_unit', '=', 'hour')]}"/>
                                <field name="number_of_hours_display" class="oe_inline" nolabel="1"
                                    attrs="{'readonly': ['|', '|', ('type_request_unit', '!=', 'hour'), ('state', 'not in', ('draft', 'confirm')), ('allocation_type', '=', 'accrual')], 'invisible': [('type_request_unit', '!=', 'hour')]}"/>
                                <span class="ml8" attrs="{'invisible': [('type_request_unit', '=', 'hour')]}">Days</span>
                                <span class="ml8" attrs="{'invisible': [('type_request_unit', '!=', 'hour')]}">Hours</span>
                            </div>
                        </group>
                        <group name="alloc_right_col">
                            <field name="employee_id" invisible="1" groups="hr_holidays.group_hr_holidays_user"/>
                            <field name="department_id" invisible="1"/>
                        </group>
                    </group>
                    <field name="notes" nolabel="1" placeholder="Add a reason..."/>
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
            <xpath expr="//button[@name='action_confirm']" position="after">
                <button string="Validate" name="action_validate" states="confirm" type="object" class="oe_highlight"/>
                <button string="Refuse" name="action_refuse" type="object"
                    attrs="{'invisible': ['|', ('can_approve', '=', False), ('state', 'not in', ('confirm','validate'))]}"/>
                <button string="Mark as Draft" name="action_draft" type="object"
                        attrs="{'invisible': ['|', ('can_reset', '=', False), ('state', 'not in', ['confirm', 'refuse'])]}"/>
            </xpath>
            <xpath expr="//div[@id='title']" position="replace">
                <div class="oe_title">
                    <h2><field name="name" placeholder="e.g. Time Off type (From validity start to validity end / no limit)" required="1"/></h2>
                </div>
            </xpath>
            <xpath expr="//field[@name='employee_id']" position="before">
                <field name="holiday_type" string="Mode" groups="hr_holidays.group_hr_holidays_user" context="{'employee_id':employee_id}" />
            </xpath>
            <xpath expr="//field[@name='employee_id']" position="replace">
                <field name="multi_employee" invisible="1" force_save="1"/>
                <!-- Employee id is only visible if the allocation is created specifically for that user in `_action_validate_create_childs` -->
                <field name="employee_id" groups="hr_holidays.group_hr_holidays_user"
                    attrs="{'invisible': ['|', '|', ('holiday_type', '!=', 'employee'), ('employee_id', '=', False), ('state', 'in', ('draft', 'cancel', 'confirm'))]}"/>
                <field name="employee_ids" widget="many2many_tags"
                    groups="hr_holidays.group_hr_holidays_user"
                    attrs="{'required': [('holiday_type', '=', 'employee'), ('state', 'in', ('draft', 'cancel', 'confirm'))],
                    'invisible': ['|', ('holiday_type', '!=', 'employee'), '&amp;', ('state', 'not in', ('draft', 'cancel', 'confirm')), ('employee_id', '!=', False)]}"/>
            </xpath>
            <xpath expr="//field[@name='employee_id']" position="after">
                <field name="category_id"
                    attrs="{'required': [('holiday_type', '=', 'category')], 'invisible': [('holiday_type', '!=', 'category')]}"/>
            </xpath>
            <xpath expr="//field[@name='department_id']" position="replace">
                <field name="department_id" groups="hr_holidays.group_hr_holidays_user"
                    attrs="{'required': [('holiday_type', '=', 'department')], 'invisible': [('holiday_type', '!=', 'department')]}"/>
            </xpath>
            <xpath expr="//field[@name='department_id']" position="after">
                <field name="mode_company_id" string="Company" groups="hr_holidays.group_hr_holidays_user"
                    attrs="{'required': [('holiday_type', '=', 'company')], 'invisible': [('holiday_type', '!=', 'company')]}"/>
            </xpath>
            <xpath expr="//field[@name='allocation_type']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <xpath expr="//label[@for='date_from']" position="replace">
                <label for="date_from" string="Validity Period" attrs="{'invisible': [('allocation_type', '=', 'accrual')]}"/>
            </xpath>
            <xpath expr="//field[@name='date_from']" position="replace">
                <field name="date_from" widget="date" nolabel="1" readonly="0" attrs="{'readonly': [('allocation_type', '=', 'accrual'), ('state', 'not in', ('draft', 'confirm'))]}"/>
            </xpath>
            <xpath expr="//i[hasclass('fa-long-arrow-right')]" position="replace">
                <i class="fa fa-long-arrow-right mx-2" aria-label="Arrow icon" title="Arrow" attrs="{'invisible': [('allocation_type', '=', 'accrual')]}"/>
            </xpath>
            <xpath expr="//field[@name='date_to']" position="replace">
                <field name="date_to" widget="date" nolabel="1" readonly="0" placeholder="No Limit"  attrs="{'readonly': [('allocation_type', '=', 'accrual'), ('state', 'not in', ('draft', 'confirm'))]}"/>
            </xpath>
            <xpath expr="//div[@id='no_limit_label']" position="replace">
                <div id="no_limit_label" class="oe_read_only" attrs="{'invisible': ['|', ('id', '=', False), ('date_to', '!=', False)]}">No limit</div>
            </xpath>
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
            <div name="validity" position="attributes">
                <attribute name="invisible">1</attribute>
            </div>
            <label for="date_from" position="attributes">
                <attribute name="invisible">1</attribute>
            </label>
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
            <tree string="Allocation Requests" sample="1" decoration-info="state == 'draft'">
                <field name="employee_id" decoration-muted="not active_employee"/>
                <field name="department_id" optional="hide"/>
                <field name="holiday_status_id" class="font-weight-bold"/>
                <field name="name"/>
                <field name="duration_display" string="Duration"/>
                <field name="date_from" string="Validity Start" optional="hide"/>
                <field name="date_to" string="Validity Stop" optional="hide"/>
                <field name="allocation_type"/>
                <field name="message_needaction" invisible="1"/>
                <field name="active_employee" invisible="1"/>
                <field name="state" widget="badge" decoration-info="state == 'draft'" decoration-warning="state == 'confirm'" decoration-success="state == 'validate'"/>
                <button string="Validate" name="action_validate" type="object"
                    icon="fa-check"
                    states="confirm"
                    groups="hr_holidays.group_hr_holidays_manager"/>
                <button string="Refuse" name="action_refuse" type="object"
                    icon="fa-times"
                    states="confirm"
                    groups="hr_holidays.group_hr_holidays_manager"/>
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
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='department_id']" position="attributes">
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
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click container">
                            <div class="row no-gutters">
                                <div class="col-3">
                                    <img t-att-src="kanban_image('hr.employee', 'avatar_128', record.employee_id.raw_value)"
                                        t-att-title="record.employee_id.value"
                                        t-att-alt="record.employee_id.value"
                                        class="o_image_64_cover float-left mr4"/>
                                </div>
                                <div class="col-9">
                                    <span class="badge badge-pill float-right mt4 mr16"><t t-esc="record.number_of_days.value"/> days</span>
                                    <strong class="o_kanban_record_title"><t t-esc="record.employee_id.value"/></strong>
                                    <div class="text-muted o_kanban_record_subtitle">
                                        <t t-esc="record.holiday_status_id.value"/>
                                    </div>
                                    <div class="o_dropdown_kanban dropdown" groups="base.group_user">
                                        <a role="button" class="dropdown-toggle o-no-caret btn" data-toggle="dropdown" href="#" aria-label="Dropdown menu" title="Dropdown menu">
                                            <span class="fa fa-ellipsis-v"/>
                                        </a>
                                        <div class="dropdown-menu" role="menu">
                                            <a t-if="widget.editable" role="menuitem" type="edit" class="dropdown-item">Edit Allocation</a>
                                            <a t-if="widget.deletable" role="menuitem" type="delete" class="dropdown-item">Delete</a>
                                        </div>
                                    </div>
                                    <div t-if="['validate', 'refuse'].includes(record.state.raw_value)">
                                        <span t-if="record.state.raw_value === 'validate'" class="fa fa-check text-muted" aria-label="validated"/>
                                        <span t-else="" class="fa fa-ban text-muted" aria-label="refused"/>
                                        <t t-set="classname" t-value="{'validate': 'badge-success', 'refuse': 'badge-danger'}[record.state.raw_value] || 'badge-light'"/>
                                        <span t-attf-class="badge badge-pill {{ classname }}"><t t-esc="record.state.value"/></span>
                                    </div>
                                    <div t-if="record.can_approve.raw_value">
                                        <button t-if="record.state.raw_value === 'confirm'" name="action_validate" type="object" class="btn btn-link btn-sm pl-0">
                                            <i class="fa fa-check"/> Validate
                                        </button>
                                        <button t-if="record.state.raw_value === 'confirm'" name="action_refuse" type="object" class="btn btn-link btn-sm pl-0">
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
                        <img t-att-src="activity_image('hr.employee', 'avatar_128', record.employee_id.raw_value)" t-att-title="record.employee_id.value" t-att-alt="record.employee_id.value"/>
                        <div>
                            <field name="employee_id"/>
                            <span class="ml-3 text-muted">
                                <field name="number_of_days"/> days
                            </span>
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
        <field name="context">{}</field>
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
                        <button class="oe_stat_button" type="object" name="action_see_days_allocated" icon="fa-calendar" attrs="{'invisible': ['|', ('requires_allocation', '=', 'no'), ('id', '=', False)]}">
                            <div class="o_stat_info">
                                <field name="group_days_allocation"/>
                                <span class="o_stat_text">Allocations</span>
                            </div>
                        </button>
                        <button class="oe_stat_button" type="object" name="action_see_group_leaves" icon="fa-calendar" attrs="{'invisible': [('id', '=', False)]}">
                            <div class="o_stat_info">
                                <field name="group_days_leave"/>
                                <span class="o_stat_text">Time Off</span>
                            </div>
                        </button>
                        <button class="oe_stat_button" type="object" name="action_see_accrual_plans" icon="fa-calendar" attrs="{'invisible': ['|', ('id', '=', False), ('accrual_count', '=', 0)]}">
                            <div class="o_stat_info">
                                <field name="accrual_count"/>
                                <span class="o_stat_text">Accruals</span>
                            </div>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title">
                        <h1><field name="name"/></h1>
                    </div>
                    <group>
                        <group name="leave_validation" id="time_off_requests">
                            <h2>Time Off Requests</h2>
                            <field name="active" invisible="1"/>
                            <field name="leave_validation_type" string="Approval" widget="radio"/>
                            <field name="responsible_id"
                                attrs="{
                                'invisible': [('leave_validation_type', 'in', ['no_validation', 'manager']), '|', ('requires_allocation', '=', 'no'), ('allocation_validation_type', '!=', 'officer')],
                                'required': ['|',('leave_validation_type', 'in', ['both', 'hr']), ('requires_allocation', '=', 'yes'), ('allocation_validation_type', '=', 'officer')]}"/>
                            <field name="request_unit" widget="radio-inline"/>
                            <field name="support_document" string="Allow To Join Supporting Document" />
                            <field name="time_type" required="1"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                        <group name="allocation_validation" id="allocation_requests">
                        <h2>Allocation Requests</h2>
                            <field name="requires_allocation" widget="radio" options="{'horizontal':true}"/>
                            <field name="employee_requests" widget="radio" attrs="{'invisible': [('requires_allocation', '=', 'no')]}"/>
                            <field name="allocation_validation_type" string="Approval" widget="radio" attrs="{'invisible': [('requires_allocation', '=', 'no')]}"/>
                        </group>
                    </group>
                    <group>
                        <group id="payroll">
                        </group>
                    </group>
                    <group name="visual" id="visual" >
                        <group colspan="4">
                            <h2>Display Option</h2>
                        </group>
                        <group colspan="4">
                            <field name="color" widget="color_picker" />
                            <field class="d-flex flex-wrap" name="icon_id" widget="radio_image" options="{'horizontal': true}"/>
                        </group>
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
                                <span class="float-right">Time Off Taken: <field name="leaves_taken"/></span>
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
            <tree string="Time Off Type">
                <field name="sequence" widget="handle"/>
                <field name="display_name"/>
                <field name="allocation_validation_type"/>
                <field name="employee_requests" optional="hide"/>
                <field name="requires_allocation" optional="hide"/>
                <field name="leave_validation_type" optional="hide"/>
                <field name="company_id" groups="base.group_multi_company" optional="hide"/>
            </tree>
        </field>
    </record>

    <record id="open_view_holiday_status" model="ir.actions.act_window">
        <field name="name">Time Off Types</field>
        <field name="type">ir.actions.act_window</field>
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
                <field name="department_id" operator="child_of"/>
                <field name="holiday_status_id"/>
                <field name="name"/>
                <filter domain="[('state','in',('confirm','validate1'))]" string="To Approve" name="approve"/>
                <filter domain="[('state', '=', 'validate1')]" string="Need Second Approval" name="second_approval"/>
                <filter string="Approved Time Off" domain="[('state', '=', 'validate')]" name="validated"/>
                <separator/>
                <filter string="My Time Off" name="my_leaves" domain="[('employee_id.user_id', '=', uid)]"/>
                <filter string="My Team" name="my_team" domain="['|', ('employee_id.leave_manager_id', '=', uid), ('employee_id.user_id', '=', uid)]" help="Time off of people you are manager of"/>
                <filter string="My Department" name="department" domain="['|', ('department_id.member_ids.user_id', '=', uid), ('employee_id.user_id', '=', uid)]" help="My Department"/>
                <separator/>
                <filter string="Active Employee" name="active_employee" domain="[('active_employee','=',True)]"/>
                <filter name="filter_date_from" date="date_from"/>
                <separator/>
                <filter name="active_time_off" string="Active Time Off"
                    domain="[('holiday_status_id.active', '=', True), '|', ('employee_id', '!=', False), '&amp;', ('employee_id', '=', False), ('state', '!=', 'validate')]" help="Active Time Off"/>
                <filter name="archive" string="Archived Time Off"
                    domain="[('holiday_status_id.active', '=', False)]" help="Archived Time Off"/>
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
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click container">
                            <div class="row no-gutters">
                                <div class="col-3">
                                    <img t-att-src="kanban_image('hr.employee', 'avatar_128', record.employee_id.raw_value)"
                                        t-att-title="record.employee_id.value"
                                        t-att-alt="record.employee_id.value"
                                        class="o_image_64_cover float-left mr4"/>
                                </div>
                                <div class="col-9">
                                    <span class="badge badge-pill float-right mt4 mr16"><t t-esc="record.number_of_days.value"/> days</span>
                                    <strong class="o_kanban_record_title"><t t-esc="record.employee_id.value"/></strong>
                                    <div class="text-muted o_kanban_record_subtitle">
                                        <t t-esc="record.holiday_status_id.value"/>
                                    </div>
                                    <div class="o_dropdown_kanban dropdown" groups="base.group_user">
                                        <a role="button" class="dropdown-toggle o-no-caret btn" data-toggle="dropdown" href="#" aria-label="Dropdown menu" title="Dropdown menu">
                                            <span class="fa fa-ellipsis-v"/>
                                        </a>
                                        <div class="dropdown-menu" role="menu">
                                            <a t-if="widget.editable" role="menuitem" type="edit" class="dropdown-item">Edit Time Off</a>
                                            <a t-if="widget.deletable" role="menuitem" type="delete" class="dropdown-item">Delete</a>
                                        </div>
                                    </div>
                                    <div>
                                        <span class="text-muted">from </span>
                                        <field name="date_from" widget="date"/>
                                        <span class="text-muted">to </span>
                                        <field name="date_to" widget="date"/>
                                    </div>
                                </div>
                            </div>
                            <div class="row no-gutters">
                                <div class="col-3"/>
                                <div class="col-6" t-if="['validate', 'refuse'].includes(record.state.raw_value)">
                                    <span t-if="record.state.raw_value === 'validate'" class="fa fa-check text-muted" aria-label="validated"/>
                                    <span t-else="" class="fa fa-ban text-muted" aria-label="refused"/>
                                    <t t-set="classname" t-value="{'validate': 'badge-success', 'refuse': 'badge-danger'}[record.state.raw_value] || 'badge-light'"/>
                                    <span t-attf-class="badge badge-pill {{ classname }}"><t t-esc="record.state.value"/></span>
                                </div>
                                <div class="col-6" t-if="['confirm', 'validate1'].includes(record.state.raw_value)">
                                    <button t-if="record.state.raw_value === 'confirm'" name="action_approve" type="object" class="btn btn-link btn-sm pl-0">
                                        <i class="fa fa-thumbs-up"/> Approve
                                    </button>
                                    <button t-if="record.state.raw_value === 'validate1'" name="action_validate" type="object" class="btn btn-link btn-sm pl-0" groups="hr_holidays.group_hr_holidays_manager">
                                        <i class="fa fa-check"/> Validate
                                    </button>
                                    <button t-if="['confirm', 'validate1'].includes(record.state.raw_value)" name="action_refuse" type="object" class="btn btn-link btn-sm pl-0">
                                        <i class="fa fa-times"/> Refuse
                                    </button>
                                </div>
                                <div class="col-3 text-right">
                                    <button t-if="record.supported_attachment_ids_count.raw_value > 0" name="action_documents" type="object" class="btn btn-link btn-sm pl-0">
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

    <record id="hr_leave_view_activity" model="ir.ui.view">
        <field name="name">hr.leave.view.activity</field>
        <field name="model">hr.leave</field>
        <field name="arch" type="xml">
            <activity string="Time Off Request">
                <field name="employee_id"/>
                <field name="date_from"/>
                <field name="date_to"/>
                <field name="number_of_days"/>
                <templates>
                    <div t-name="activity-box">
                        <img t-att-src="activity_image('hr.employee', 'avatar_128', record.employee_id.raw_value)" t-att-title="record.employee_id.value" t-att-alt="record.employee_id.value" width="50" height="50"/>
                        <div>
                            <field name="name"/>
                            <span class="ml-3 text-muted">
                                <field name="number_of_days"/> days
                            </span>
                            <div class="text-muted">
                                <div>From: <field name="date_from" widget="date"/></div>
                                <div>To: <field name="date_to" widget="date"/></div>
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
            <form string="Time Off Request">
            <field name="can_reset" invisible="1"/>
            <field name="can_approve" invisible="1"/>
            <field name="holiday_allocation_id" invisible="1" force_save="1"/>
            <header>
                <button string="Confirm" name="action_confirm" type="object" class="oe_highlight" attrs="{'invisible': ['|', ('id', '=', False), ('state', '!=', 'draft')]}"/>
                <button string="Approve" name="action_approve" type="object" class="oe_highlight" attrs="{'invisible': ['|', '|', ('id', '=', False), ('can_approve', '=', False), ('state', '!=', 'confirm')]}"/>
                <button string="Validate" name="action_validate" states="validate1" type="object" groups="hr_holidays.group_hr_holidays_manager" class="oe_highlight"/>
                <button string="Refuse" name="action_refuse" type="object" attrs="{'invisible': ['|', '|', ('id', '=', False), ('can_approve', '=', False), ('state', 'not in', ('confirm','validate1','validate'))]}"/>
                <button string="Mark as Draft" name="action_draft" type="object"
                        attrs="{'invisible': ['|', '|', ('id', '=', False), ('can_reset', '=', False), ('state', 'not in', ['confirm', 'refuse'])]}"/>
                <field name="state" widget="statusbar" statusbar_visible="confirm,validate"/>
            </header>
            <sheet>
                <div class="alert alert-info" role="alert" attrs="{'invisible': ['|', ('request_unit_hours', '=', False), '|', ('tz_mismatch', '=', False), ('holiday_type', '=', 'category')]}">
                    <span attrs="{'invisible': [('holiday_type', '!=', 'employee')]}">
                        The employee has a different timezone than yours! Here dates and times are displayed in the employee's timezone
                    </span>
                    <span attrs="{'invisible': [('holiday_type', '!=', 'department')]}">
                        The department's company has a different timezone than yours! Here dates and times are displayed in the company's timezone
                    </span>
                    <span attrs="{'invisible': [('holiday_type', '!=', 'company')]}">
                        The company has a different timezone than yours! Here dates and times are displayed in the company's timezone
                    </span>
                    (<field name="tz"/>).
                </div>
                <field name="tz_mismatch" invisible="1"/>
                <field name="holiday_type" invisible="1"/>
                <field name="leave_type_request_unit" invisible="1"/>
                <div name="title">
                    <field name="display_name" invisible="1"/>
                    <field name="employee_id" nolabel="1" readonly="1" force_save="1" invisible="1"/>
                    <field name="employee_ids" invisible="1"/>
                </div>
                <group>
                    <group name="col_left">
                        <field name="holiday_status_id" force_save="1" domain="['|', ('requires_allocation', '=', 'no'), '&amp;', ('has_valid_allocation', '=', True), '&amp;', ('virtual_remaining_leaves', '&gt;', 0), ('max_leaves', '>', '0')]" context="{'employee_id':employee_id, 'default_date_from':date_from, 'default_date_to':date_to}" options="{'no_create': True, 'no_open': True}" class="w-100"/>
                        <label for="request_date_from" string="Dates" id="label_dates"/>
                        <div>
                            <field name="date_from" invisible="1" widget="daterange"/>
                            <field name="date_to" invisible="1"/>
                            <div class="o_row o_row_readonly">
                                <span class="oe_inline"
                                    attrs="{'invisible': ['|', ('request_unit_half', '=', True), ('request_unit_hours', '=', True)]}">
                                    From
                                </span>
                                <field name="request_date_from" class="oe_inline" nolabel="1"
                                    attrs="{'readonly': [('state', 'not in', ('draft', 'confirm'))],
                                            'required': ['|', ('date_from', '=', False), ('date_to', '=', False)]
                                            }"
                                    widget="daterange" options="{'related_end_date': 'request_date_to'}"/>
                                <span class="oe_inline"
                                    attrs="{'invisible': ['|', ('request_unit_half', '=', True), ('request_unit_hours', '=', True)]}">
                                    To
                                </span>
                                <field name="request_date_to" class="oe_inline"
                                    attrs="{
                                        'readonly': [('state', 'not in', ('draft', 'confirm'))],
                                        'invisible': ['|', ('request_unit_half', '=', True), ('request_unit_hours', '=', True)],
                                        'required': ['|', ('date_from', '=', False), ('date_to', '=', False)]
                                            }"
                                    widget="daterange" options="{'related_start_date': 'request_date_from'}"/>
                                <field name="request_date_from_period" class="oe_inline"
                                    string="In"
                                    options="{'horizontal': True}"
                                    attrs="{
                                        'readonly': [('state', 'not in', ('draft', 'confirm'))],
                                        'required': [('request_unit_half', '=', True)],
                                        'invisible': [('request_unit_half', '=', False)]}"/>
                            </div>
                            <div class="o_row o_row_readonly oe_edit_only" style="margin-left: -2px;">
                                <field name="request_unit_half" attrs="{
                                    'readonly': [('state', 'not in', ('draft', 'confirm'))],
                                    'invisible': [('leave_type_request_unit', '=', 'day')]
                                }"/>
                                <label for="request_unit_half" attrs="{
                                    'invisible': [('leave_type_request_unit', '=', 'day')]
                                 }"/>
                                <field name="request_unit_hours" attrs="{
                                    'readonly': [('state', 'not in', ('draft', 'confirm'))],
                                    'invisible': [('leave_type_request_unit', '!=', 'hour')]
                                 }" class="ml-5"/>
                                <label for="request_unit_hours" attrs="{
                                    'invisible': [('leave_type_request_unit', '!=', 'hour')]
                                }"/>

                                <field name="request_unit_custom" invisible="1" attrs="{
                                    'readonly': [('state', 'not in', ('draft', 'confirm'))],
                                 }" class="ml-5"/>
                                <label for="request_unit_custom" invisible="1"/>
                            </div>
                            <div class="o_row o_row_readonly">
                                <label for="request_hour_from" string="From"
                                    attrs="{'invisible': [('request_unit_hours', '=', False)]}"/>
                                <field name="request_hour_from"
                                    attrs="{
                                        'readonly': [('state', '=', 'validate')],
                                        'required': [('request_unit_hours', '=', True)],
                                        'invisible': [('request_unit_hours', '=', False)]}"/>
                                <label for="request_hour_to" string="To"
                                    attrs="{'invisible': [('request_unit_hours', '=', False)]}"/>
                                <field name="request_hour_to"
                                    attrs="{
                                        'readonly': [('state', '=', 'validate')],
                                        'required': [('request_unit_hours', '=', True)],
                                        'invisible': [('request_unit_hours', '=', False)]}"/>
                            </div>
                        </div>

                        <!-- When the user is leave manager, he should always see `number_of_days` to allow
                        him to edit the value. `number_of_hours_display` is only an informative field -->
                        <label for="number_of_days" string="Duration" attrs="{'invisible': [('request_unit_half', '=', True), ('leave_type_request_unit', '!=', 'hour')]}"/>
                        <div name="duration_display">
                            <div class="o_row">
                                <div groups="!hr_holidays.group_hr_holidays_manager" attrs="{'invisible': ['|', ('request_unit_half', '=', True), ('request_unit_hours', '=', True)]}" class="o_row">
                                    <field name="number_of_days_display" nolabel="1" readonly="1" class="oe_inline"/>
                                    <span>Days</span>
                                </div>
                                <div groups="hr_holidays.group_hr_holidays_manager" class="o_row" attrs="{'invisible': ['|', ('request_unit_half', '=', True), ('request_unit_hours', '=', True)]}">
                                    <field name="number_of_days" nolabel="1" class="oe_inline"/>
                                    <span>Days</span>
                                </div>
                                <div attrs="{'invisible': [('leave_type_request_unit', '!=', 'hour')]}" class="o_row">
                                    <field name="number_of_hours_text" nolabel="1" class="oe_inline"/>
                                </div>
                            </div>
                        </div>
                        <field name="name" attrs="{'readonly': [('state', 'not in', ('draft', 'confirm'))]}" widget="text"/>
                        <field name="user_id" invisible="1"/>
                        <field name="leave_type_support_document" invisible="1"/>
                        <label for="supported_attachment_ids" string="Supporting Document"
                            attrs="{'invisible': ['|', ('leave_type_support_document', '=', False), ('state', 'not in', ('draft', 'confirm', 'validate1'))]}"/>
                        <field name="supported_attachment_ids" widget="many2many_binary" nolabel="1"
                            attrs="{'invisible': ['|', ('leave_type_support_document', '=', False), ('state', 'not in', ('draft', 'confirm', 'validate1'))]}"/>
                    </group>
                    <group name="col_right">
                        <field name="department_id" groups="hr_holidays.group_hr_holidays_user" invisible="1"/>
                    </group>
                </group>
            </sheet>
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
        </field>
    </record>

    <record id="hr_leave_view_dashboard" model="ir.ui.view">
        <field name="name">hr.leave.view.dashboard</field>
        <field name="model">hr.leave</field>
        <field name="arch" type="xml">
            <calendar js_class="time_off_calendar" string="Time Off Request" form_view_id="%(hr_holidays.hr_leave_view_form_dashboard_new_time_off)d" event_open_popup="true" date_start="date_from" date_stop="date_to" mode="year" quick_add="False" show_unusual_days="True" color="color" hide_time="True">
                <field name="display_name"/>
                <field name="holiday_status_id" filters="1" invisible="1" color="color"/>
                <field name="state" invisible="1"/>
                <field name="is_hatched" invisible="1" />
                <field name="is_striked" invisible="1"/>
            </calendar>
        </field>
    </record>

    <record id="hr_leave_employee_view_dashboard" model="ir.ui.view">
        <field name="name">hr.leave.view.dashboard</field>
        <field name="model">hr.leave</field>
        <field name="arch" type="xml">
            <calendar string="Time Off Request" js_class="time_off_employee_calendar" form_view_id="%(hr_holidays.hr_leave_view_form_dashboard_new_time_off)d" event_open_popup="true" date_start="date_from" date_stop="date_to" mode="year" quick_add="False" show_unusual_days="True" color="color" hide_time="True">
                <field name="display_name"/>
                <field name="holiday_status_id" filters="1" invisible="1" color="color"/>
                <field name="state" invisible="1"/>
                <field name="is_hatched" invisible="1" />
                <field name="is_striked" invisible="1"/>
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
            <field name="holiday_status_id" position="replace"/>
            <div name="title" position="inside">
                <h1 class="d-flex flex-row justify-content-between">
                    <field name="holiday_status_id" options="{'no_open': True}" context="{'from_manager_leave_form': True ,'employee_id': employee_id, 'default_date_from':date_from, 'default_date_to':date_to}"/>
                </h1>
            </div>
            <field name="employee_id" position="replace"/>
            <label id="label_dates" position="before">
                    <field name="multi_employee" invisible="1" force_save="1"/>
                    <field name="employee_id" groups="hr_holidays.group_hr_holidays_user" attrs="{
                        'invisible': ['|', '|', ('holiday_type', '!=', 'employee'), ('state', '!=', 'validate'), ('employee_id', '=', False)]
                        }" widget="many2one_avatar_employee"/>
                    <field name="employee_ids" groups="hr_holidays.group_hr_holidays_user" attrs="{
                        'required': [('holiday_type', '=', 'employee'), ('state', 'in', ('draft', 'cancel', 'refuse'))],
                        'invisible': ['|', ('holiday_type', '!=', 'employee'), '&amp;', ('state', '=', 'validate'), ('employee_id', '!=', False)],
                        }" widget="many2many_tags"/>
            </label>
            <field name="name" position="replace"/>
            <field name="user_id" position="before">
                <field name="name"/>
            </field>
            <xpath expr="//group[@name='col_right']" position="replace">
                <group>
                    <widget name="hr_leave_stats"/>
                </group>
                <group>
                    <field name="holiday_type" string="Mode"
                        groups="hr_holidays.group_hr_holidays_user"/>
                    <field name="mode_company_id" string="Company" groups="hr_holidays.group_hr_holidays_user" attrs="{
                        'required': [('holiday_type', '=', 'company')],
                        'invisible': [('holiday_type', '!=', 'company')]
                        }"/>
                    <field name="category_id" groups="hr_holidays.group_hr_holidays_user" attrs="{
                        'required': [('holiday_type', '=', 'category')],
                        'invisible': [('holiday_type', '!=','category')]
                        }"/>
                    <field name="department_id" groups="hr_holidays.group_hr_holidays_user" attrs="{
                        'required': [('holiday_type', '=', 'department')],
                        'invisible': [('holiday_type', '!=', 'department')]
                        }"/>
                </group>
            </xpath>
        </field>
    </record>

    <record id="hr_leave_view_calendar" model="ir.ui.view">
        <field name="name">hr.leave.view.calendar</field>
        <field name="model">hr.leave</field>
        <field name="arch" type="xml">
            <calendar js_class="time_off_calendar_all" string="Time Off Request" form_view_id="%(hr_holidays.hr_leave_view_form_dashboard)d" event_open_popup="true" date_start="date_from" date_stop="date_to" mode="month" show_unusual_days="True" quick_add="False" color="color">
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
                <field name="employee_id" widget="many2one_avatar_employee" decoration-muted="not active_employee"/>
                <field name="department_id" optional="hidden"/>
                <field name="holiday_type" string="Mode" groups="base.group_no_one"/>
                <field name="holiday_status_id" class="font-weight-bold"/>
                <field name="name"/>
                <field name="date_from"/>
                <field name="date_to"/>
                <field name="duration_display" string="Duration"/>
                <field name="state" widget="badge" decoration-info="state == 'draft'" decoration-warning="state in ('confirm','validate1')" decoration-success="state == 'validate'"/>
                <field name="active_employee" invisible="1"/>
                <field name="category_id" invisible="1"/>
                <field name="user_id" invisible="1"/>
                <field name="message_needaction" invisible="1"/>
                <button string="Approve" name="action_approve" type="object"
                    icon="fa-thumbs-up"
                    states="confirm"
                    groups="hr_holidays.group_hr_holidays_responsible"/>
                <button string="Validate" name="action_validate" type="object"
                    icon="fa-check"
                    states="validate1"
                    groups="hr_holidays.group_hr_holidays_manager"/>
                <button string="Refuse" name="action_refuse" type="object"
                    icon="fa-times"
                    states="confirm,validate1"
                    groups="hr_holidays.group_hr_holidays_manager"/>
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
            <xpath expr="//field[@name='employee_id']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='department_id']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='holiday_type']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//button[@name='action_approve']" position="attributes">
                <attribute name="states"/>
                <attribute name="attrs">{'invisible': 1}</attribute>
            </xpath>
            <xpath expr="//button[@name='action_validate']" position="attributes">
                <attribute name="states"/>
                <attribute name="attrs">{'invisible': 1}</attribute>
            </xpath>
            <xpath expr="//button[@name='action_refuse']" position="attributes">
                <attribute name="states"/>
                <attribute name="attrs">{'invisible': 1}</attribute>
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
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">hr.leave</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
    <record id="hr_leave_action_my_request_view_form" model="ir.actions.act_window.view">
        <field name="view_mode">form</field>
        <field name="act_window_id" ref="hr_leave_action_my_request"/>
        <field name="view_id" ref="hr_leave_view_form_dashboard_new_time_off"/>
    </record>

    <record id="hr_leave_action_my" model="ir.actions.act_window">
        <field name="name">My Time Off</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">hr.leave</field>
        <field name="view_mode">tree,form,kanban,activity</field>
        <field name="context">{}</field>
        <field name="search_view_id" ref="hr_leave_view_search_my"/>
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
            'search_default_approve': 1,
            'search_default_my_team': 2,
            'search_default_active_employee': 3,
            'search_default_active_time_off': 4,
            'hide_employee_name': 1}</field>
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
            'hide_employee_name': 1}
        </field>
        <field name="domain">[('holiday_allocation_id', '=', active_id)]</field>
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
        <field name="view_id" ref="hr_leave_view_kanban"/>
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
                <field name="number_of_days" type="measure"/>
            </graph>
        </field>
    </record>

    <record id="action_hr_available_holidays_report" model="ir.actions.act_window">
        <field name="name">Time Off Analysis</field>
        <field name="res_model">hr.leave</field>
        <field name="view_mode">graph,pivot,calendar,form</field>
        <field name="context">{'search_default_year': 1, 'search_default_active_employee': 2, 'search_default_group_employee': 1, 'search_default_group_type': 1}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No data yet!
            </p>
        </field>
    </record>

    <record id="action_window_leave_graph" model="ir.actions.act_window.view">
        <field name="sequence" eval="1"/>
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
       <field name="view_mode">kanban,tree,form</field>
       <field name="context">{
           'search_default_is_absent': 1,
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
        <field name="groups_id" eval="[(4,ref('hr_holidays.group_hr_holidays_user'))]"/>
        <field name="arch" type="xml">
            <data>
                <xpath expr="//templates" position="before">
                    <field name="leave_to_approve_count"/>
                    <field name="allocation_to_approve_count"/>
                    <field name="total_employee"/>
                    <field name="absence_of_today"/>
                </xpath>

                <xpath expr="//div[hasclass('o_kanban_primary_right')]" position="inside">
                    <div t-if="record.leave_to_approve_count.raw_value > 0" class="row ml16">
                        <div class="col-9">
                            <a name="action_open_leave_department" type="object">
                                Time Off Requests
                            </a>
                        </div>
                        <div class="col-3 text-right">
                            <field name="leave_to_approve_count"/>
                        </div>
                    </div>
                    <div t-if="record.allocation_to_approve_count.raw_value > 0" class="row ml16">
                        <div class="col-9">
                            <a name="action_open_allocation_department" type="object">
                                Allocation Requests
                            </a>
                        </div>
                        <div class="col-3 text-right">
                            <field name="allocation_to_approve_count"/>
                        </div>
                    </div>
                </xpath>

                <xpath expr="//div[hasclass('o_kanban_card_upper_content')]" position="after">
                    <div class="row o_kanban_primary_bottom bottom_block">
                        <div class="col-3">
                            <a name="%(hr_employee_action_from_department)d" type="action" title="Absent Employee(s), Whose time off requests are either confirmed or validated on today">Absence</a>
                        </div>
                        <div class="col-9">
                            <field name="absence_of_today" widget="progressbar" options="{'current_value': 'absence_of_today', 'max_value': 'total_employee', 'editable': false}"/>
                        </div>
                    </div>
                </xpath>

                <xpath expr="//div[hasclass('o_kanban_manage_reports')]" position="inside">
                    <a role="menuitem" class="dropdown-item" name="%(hr_leave_action_action_department)d" type="action">
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
            <xpath expr="//field[@name='job_id']" position="after">
                <filter name="is_absent" string="Absent Today" domain="[('is_absent', '=', True)]"/>
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
            <xpath expr="//strong[hasclass('o_kanban_record_title')]" position="inside">
                 <!-- Employee is absent, in holiday but he is connected -->
                <div class="float-right"
                     t-if="record.hr_icon_display.raw_value == 'presence_holiday_present'">
                    <span class="fa fa-plane text-success" role="img" aria-label="Present but on leave"
                          title="Present but on leave" name="presence_absent_active">
                    </span>
                </div>
                <!-- Employee is on holiday, not present and not connected -->
                <div class="float-right"
                     t-if="record.hr_icon_display.raw_value == 'presence_holiday_absent'"
                     name="presence_absent">
                    <span class="fa fa-plane text-warning" role="img" aria-label="To define" title="On Leave"/>
                </div>
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
            <xpath expr="//div[@name='presence_absent_active']" position="after">
                <!-- Employee is absent, in holiday but he is connected -->
                <!-- green plane -->
                <div class="float-right"
                     t-if="record.hr_icon_display.raw_value == 'presence_holiday_present'">
                    <span class="fa fa-plane text-success" role="img" aria-label="Present but on leave"
                          title="Present but on leave" name="presence_absent_active">
                    </span>
                </div>
                <!-- Employee is on holiday, not present and not connected -->
                <!-- orange plane -->
                <div class="float-right"
                     t-if="record.hr_icon_display.raw_value == 'presence_holiday_absent'"
                     name="presence_absent">
                    <span class="fa fa-plane text-warning" role="img" aria-label="To define" title="On Leave"/>
                </div>
           </xpath>
            <xpath expr="//li[@id='last_login']" position="inside">
                <span t-if="record.current_leave_id.raw_value" style="font-size: 100%%"
                        t-att-class="record.current_leave_state.raw_value=='validate'?'oe_kanban_button oe_kanban_color_3':'oe_kanban_button oe_kanban_color_2'"
                        t-att-title="moment(record.leave_date_from.raw_value).format('ddd Do MMM') + ' - ' + moment(record.leave_date_to.raw_value).format('ddd Do MMM')">
                    <field name="current_leave_id"/>
                </span>
            </xpath>
        </field>
    </record>

    <!-- Hr employee inherit Legal Leaves -->
    <record id="view_employee_form_leave_inherit" model="ir.ui.view">
        <field name="name">hr.employee.leave.form.inherit</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@id='hr_presence_button']" position="attributes">
                <attribute name="attrs">
                    {'invisible': ['|', '|', '|', ('last_activity', '=', False), ('is_absent', '=', True), ('user_id', '=', False), ('id', '=', False)]}
                </attribute>
            </xpath>
            <xpath expr="//group[@name='managers']" position="inside">
                <field name="leave_manager_id"/>
            </xpath>
            <xpath expr="//group[@name='managers']" position="attributes">
                <attribute name="invisible">0</attribute>
            </xpath>
            <div name="button_box" position="inside">
                <field name="show_leaves" invisible="1"/>
                <field name="is_absent" invisible="1"/>
                <field name="hr_icon_display" invisible="1"/>
                <button name="action_time_off_dashboard"
                        type="object"
                        class="oe_stat_button"
                        context="{'search_default_employee_id': id}"
                        attrs="{'invisible': [('is_absent', '=', False)]}">
                        <div attrs="{'invisible': [('hr_icon_display', '!=', 'presence_holiday_present')]}"
                              role="img" class="fa fa-fw fa-plane o_button_icon text-success" aria-label="Off Till" title="Off Till"/>
                        <div attrs="{'invisible': [('hr_icon_display', '!=', 'presence_holiday_absent')]}" role="img"
                             class="fa fa-fw fa-plane o_button_icon text-warning" aria-label="Off Till" title="Off Till"/>
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="leave_date_to"/>
                        </span>
                        <span class="o_stat_text">
                            Off Till
                        </span>
                    </div>
                </button>
                <button name="action_time_off_dashboard"
                        type="object"
                        class="oe_stat_button"
                        icon="fa-calendar"
                        attrs="{'invisible': [('show_leaves','=', False)]}"
                        context="{'search_default_employee_id': id}"
                        groups="base.group_user"
                        help="Remaining leaves">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="allocation_used_display"/>/<field name="allocation_display"/> Days
                        </span>
                        <span class="o_stat_text">
                            Time Off
                        </span>
                    </div>
                </button>
            </div>
        </field>
    </record>

    <record id="view_employee_tree_inherit_leave" model="ir.ui.view">
        <field name="name">hr.employee.tree.leave</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='work_location_id']" position="after">
                <field name="leave_manager_id" optional="hide" string="Time Off Approver"/>
            </xpath>
        </field>
    </record>

    <record id="hr_employee_public_form_view_inherit" model="ir.ui.view">
        <field name="name">hr.employee.public.leave.form.inherit</field>
        <field name="model">hr.employee.public</field>
        <field name="inherit_id" ref="hr.hr_employee_public_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='coach_id']" position="after">
                <field name="leave_manager_id"/>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <field name="show_leaves" invisible="1"/>
                <field name="is_absent" invisible="1"/>
                <field name="hr_icon_display" invisible="1"/>
                <button disabled="1"
                        class="oe_stat_button"
                        attrs="{'invisible': [('is_absent', '=', False)]}">
                        <div role="img" attrs="{'invisible': [('hr_icon_display', '!=', 'presence_holiday_present')]}"
                             class="fa fa-fw fa-plane o_button_icon text-success" aria-label="Off Till" title="Off Till"/>
                        <div attrs="{'invisible': [('hr_icon_display', '!=', 'presence_holiday_absent')]}" role="img"
                             class="fa fa-fw fa-plane o_button_icon text-warning" aria-label="Off Till" title="Off Till"/>
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="leave_date_to"/>
                        </span>
                        <span class="o_stat_text">
                            Off Till
                        </span>
                        <t t-esc="hr_icon_display == 'presence_holiday_present'"/>
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
                <field name="leave_manager_id" attrs="{'readonly': [('can_edit', '=', False)]}"/>
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
                        invisible="context.get('from_my_profile', False)"
                        attrs="{'invisible': [('is_absent', '=', False)]}">
                        <div attrs="{'invisible': [('hr_icon_display', '!=', 'presence_holiday_present')]}"
                             role="img" class="fa fa-fw fa-plane o_button_icon text-success" aria-label="Off Till"
                             title="Off Till"/>
                        <div attrs="{'invisible': [('hr_icon_display', '!=', 'presence_holiday_absent')]}"
                             role="img" class="fa fa-fw fa-plane o_button_icon text-warning" aria-label="Off Till"
                             title="Off Till"/>
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="leave_date_to"/>
                        </span>
                        <span class="o_stat_text">
                            Off Till
                        </span>
                    </div>
                </button>
                <button name="%(hr_leave_action_new_request)d"
                        type="action"
                        class="oe_stat_button"
                        icon="fa-calendar"
                        attrs="{'invisible': [('show_leaves','=', False)]}"
                        groups="base.group_user"
                        help="Remaining leaves">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value">
                            <field name="allocation_used_display"/>/<field name="allocation_display"/> Days
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
                <field name="holiday_id"/>
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
            </xpath>
            <xpath expr="//field[@name='resource_id']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//field[@name='date_to']" position="after">
                <xpath expr="//field[@name='calendar_id']" position="move"/>
            </xpath>
        </field>
    </record>

    <record id="resource_calendar_global_leaves_action_from_calendar" model="ir.actions.act_window">
        <field name="name">Resource Time Off</field>
        <field name="res_model">resource.calendar.leaves</field>
        <field name="view_mode">tree</field>
        <field name="view_id" ref="resource_calendar_leaves_tree_inherit"/>
        <field name="domain">[('resource_id', '=', False)]</field>
        <field name="context">{
            'default_calendar_id': active_id,
            'search_default_calendar_id': active_id,
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
                                string="Public Time Off" icon="fa-sun-o"
                                class="oe_stat_button"/>
            </xpath>
        </field>
    </record>

    <record id="open_view_public_holiday" model="ir.actions.act_window">
        <field name="name">Public Holidays</field>
        <field name="type">ir.actions.act_window</field>
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

    cancel_leaves = fields.Boolean("Cancel Future Leaves", default=True,
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
                        <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z" />
                    </footer>
                </form>
            </field>
        </record>

        <record id="action_hr_holidays_summary_employee" model="ir.actions.act_window">
            <field name="name">Time Off Summary</field>
            <field name="type">ir.actions.act_window</field>
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

from . import hr_holidays_summary_employees
from . import hr_departure_wizard

```

