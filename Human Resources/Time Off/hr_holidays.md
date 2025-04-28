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
    'summary': 'Allocate time off and follow time off requests',
    'website': 'https://www.odoo.com/page/leaves',
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
        'views/mail_activity_views.xml',

        'wizard/hr_holidays_summary_employees_views.xml',
        'wizard/hr_departure_wizard_views.xml',

        'report/hr_holidays_templates.xml',
        'report/hr_holidays_reports.xml',
        'report/hr_leave_reports.xml',
        'report/hr_leave_report_calendar.xml',

        'views/hr_views.xml',
        'views/hr_leave_template.xml',
        'views/hr_holidays_views.xml',
    ],
    'demo': [
        'data/hr_holidays_demo.xml',
    ],
    'qweb': [
        'static/src/bugfix/bugfix.xml',
        'static/src/components/partner_im_status_icon/partner_im_status_icon.xml',
        'static/src/components/thread_icon/thread_icon.xml',
        'static/src/components/thread_view/thread_view.xml',
        'static/src/xml/*.xml',
    ],
    'installable': True,
    'application': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.mail.controllers.main import MailController
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
    <data noupdate="1">
        <!-- Casual leave -->
        <record id="holiday_status_cl" model="hr.leave.type">
            <field name="name">Paid Time Off</field>
            <field name="allocation_type">fixed</field>
            <field name="leave_validation_type">both</field>
            <field name="allocation_validation_type">both</field>
            <field name="color_name">black</field>
            <field name="leave_notif_subtype_id" ref="mt_leave"/>
            <field name="allocation_notif_subtype_id" ref="mt_leave_allocation"/>
            <field name="validity_start" eval="time.strftime('%Y-%m-01')"/>
            <field name="responsible_id" ref="base.user_admin"/>
        </record>

        <!-- Sick leave -->
        <record id="holiday_status_sl" model="hr.leave.type">
            <field name="name">Sick Time Off</field>
            <field name="allocation_type">no</field>
            <field name="color_name">red</field>
            <field name="validity_start" eval="time.strftime('%Y-01-01')"/>
            <field name="leave_notif_subtype_id" ref="mt_leave_sick"/>
            <field name="responsible_id" ref="base.user_admin"/>
        </record>

        <!-- Compensatory Days -->
        <record id="holiday_status_comp" model="hr.leave.type">
            <field name="name">Compensatory Days</field>
            <field name="allocation_type">fixed_allocation</field>
            <field name="leave_validation_type">manager</field>
            <field name="allocation_validation_type">manager</field>
            <field name="request_unit">hour</field>
            <field name="color_name">lavender</field>
            <field name="validity_start" eval="False"/>
            <field name="leave_notif_subtype_id" ref="mt_leave"/>
            <field name="responsible_id" ref="base.user_admin"/>
        </record>

        <!--Unpaid Leave -->
        <record id="holiday_status_unpaid" model="hr.leave.type">
            <field name="name">Unpaid</field>
            <field name="allocation_type">no</field>
            <field name="leave_validation_type">both</field>
            <field name="allocation_validation_type">both</field>
            <field name="color_name">brown</field>
            <field name="request_unit">hour</field>
            <field name="unpaid" eval="True"/>
            <field name="validity_start" eval="time.strftime('%Y-01-01')"/>
            <field name="leave_notif_subtype_id" ref="mt_leave_unpaid"/>
            <field name="responsible_id" ref="base.user_admin"/>
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
        <field name="allocation_type">fixed</field>
        <field name="color_name">brown</field>
        <field name="leave_validation_type">both</field>
        <field name="allocation_validation_type">both</field>
        <field name="validity_start" eval="False"/>
        <field name="responsible_id" ref="base.user_admin"/>
    </record>

    <!-- ++++++++++++++++++++++  Mitchell Admin  ++++++++++++++++++++++ -->

    <record id="hr_holidays_allocation_cl" model="hr.leave.allocation">
        <field name="name">Paid Time Off for Mitchell Admin</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field name="number_of_days">20</field>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="state">validate</field>
    </record>

    <record id="hr_holidays_int_tour" model="hr.leave.allocation">
        <field name="name">International Tour</field>
        <field name="holiday_status_id" ref="holiday_status_comp"/>
        <field name="number_of_days">7</field>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="state">confirm</field>
    </record>
    <function model="hr.leave.allocation" name="action_approve">
        <value eval="ref('hr_holidays.hr_holidays_int_tour')"/>
    </function>

    <record id="hr_holidays_vc" model="hr.leave.allocation">
        <field name="name">Summer Vacation</field>
        <field name="holiday_status_id" ref="holiday_status_unpaid"/>
        <field name="number_of_days">7</field>
        <field name="employee_id" ref="hr.employee_admin"/>
    </record>

    <record id='hr_holidays_cl_allocation' model="hr.leave.allocation">
        <field name="name">Compensation</field>
        <field name="holiday_status_id" ref="holiday_status_comp"/>
        <field name="number_of_days">12</field>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="state">validate</field>
    </record>

    <!-- leave request -->
    <record id="hr_holidays_cl" model="hr.leave">
        <field name="name">Trip with Family</field>
        <field name="holiday_status_id" ref="holiday_status_comp"/>
        <field eval="(datetime.now() + relativedelta(day=1, weekday=0)).strftime('%Y-%m-%d 08:00:00')" name="date_from"/>
        <field eval="(datetime.now() + relativedelta(day=1, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 17:00:00')" name="date_to"/>
        <field eval="(datetime.now() + relativedelta(day=1, weekday=0)).strftime('%Y-%m-%d 08:00:00')" name="request_date_from"/>
        <field eval="(datetime.now() + relativedelta(day=1, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 17:00:00')" name="request_date_to"/>
        <field name="employee_id" ref="hr.employee_admin"/>
    </record>

    <record id="hr_holidays_sl" model="hr.leave">
        <field name="name">Doctor Appointment</field>
        <field name="holiday_status_id" ref="holiday_status_sl"/>
        <field eval="(datetime.now() + relativedelta(day=20, weekday=0)).strftime('%Y-%m-%d 08:00:00')" name="date_from"/>
        <field eval="(datetime.now() + relativedelta(day=20, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 17:00:00')" name="date_to"/>
        <field eval="(datetime.now() + relativedelta(day=20, weekday=0)).strftime('%Y-%m-%d 08:00:00')" name="request_date_from"/>
        <field eval="(datetime.now() + relativedelta(day=20, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 17:00:00')" name="request_date_to"/>
        <field name="employee_id" ref="hr.employee_admin"/>
        <field name="state">confirm</field>
    </record>
    <function model="hr.leave" name="action_approve">
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
        <field name="state">validate</field>
    </record>

    <record id="hr_holidays_allocation_pl_al" model="hr.leave.allocation">
        <field name="name">Parental Leaves</field>
        <field name="holiday_status_id" ref="hr_holiday_status_dv"/>
        <field name="number_of_days">10</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="state">validate</field>
    </record>

    <record id="hr_holidays_vc_al" model="hr.leave.allocation">
        <field name="name">Summer Vacation</field>
        <field name="holiday_status_id" ref="holiday_status_unpaid"/>
        <field name="number_of_days">12</field>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="state">validate</field>
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
    </record>
    <function model="hr.leave" name="action_approve">
        <value eval="ref('hr_holidays.hr_holidays_cl_al')"/>
    </function>

    <record id="hr_holidays_sl_al" model="hr.leave">
        <field name="name">Dentist appointment</field>
        <field name="holiday_status_id" ref="holiday_status_sl"/>
        <field eval="(datetime.now()+relativedelta(months=1, day=17, weekday=0)).strftime('%Y-%m-%d 08:00:00')" name="date_from"/>
        <field eval="(datetime.now()+relativedelta(months=1, day=17, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 17:00:00')" name="date_to"/>
        <field eval="(datetime.now()+relativedelta(months=1, day=17, weekday=0)).strftime('%Y-%m-%d 08:00:00')" name="request_date_from"/>
        <field eval="(datetime.now()+relativedelta(months=1, day=17, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 17:00:00')" name="request_date_to"/>
        <field name="employee_id" ref="hr.employee_al"/>
        <field name="state">confirm</field>
    </record>
    <function model="hr.leave" name="action_approve">
        <value eval="ref('hr_holidays.hr_holidays_sl_al')"/>
    </function>

    <!-- ++++++++++++++++++++++  Anita Oliver  ++++++++++++++++++++++ -->

    <record id="hr_holidays_allocation_cl_mit" model="hr.leave.allocation">
        <field name="name">Paid Time Off for Anita Oliver</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field name="number_of_days">20</field>
        <field name="employee_id" ref="hr.employee_mit"/>
        <field name="state">validate</field>
    </record>

    <record id="hr_holidays_vc_mit" model="hr.leave.allocation">
        <field name="name">Summer Vacation</field>
        <field name="holiday_status_id" ref="holiday_status_unpaid"/>
        <field name="number_of_days">7</field>
        <field name="employee_id" ref="hr.employee_mit"/>
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
    </record>
    <function model="hr.leave" name="action_approve">
        <value eval="ref('hr_holidays.hr_holidays_cl_mit')"/>
    </function>

    <record id="hr_holidays_cl_mit_2" model="hr.leave">
        <field name="name">Trip</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field eval="(datetime.now() + relativedelta(day=5, weekday=0)).strftime('%Y-%m-%d 08:00:00')" name="date_from"/>
        <field eval="(datetime.now() + relativedelta(day=5, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 17:00:00')" name="date_to"/>
        <field eval="(datetime.now() + relativedelta(day=5, weekday=0)).strftime('%Y-%m-%d 08:00:00')" name="request_date_from"/>
        <field eval="(datetime.now() + relativedelta(day=5, weekday=0) + relativedelta(weekday=2)).strftime('%Y-%m-%d 17:00:00')" name="request_date_to"/>
        <field name="employee_id" ref="hr.employee_mit"/>
    </record>

    <!-- ++++++++++++++++++++++  Marc Demo  ++++++++++++++++++++++ -->

    <record id="hr_holidays_allocation_cl_qdp" model="hr.leave.allocation">
        <field name="name">Paid Time Off for Marc Demo</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field name="number_of_days">20</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
        <field name="state">validate</field>
    </record>

    <record id="hr_holidays_vc_qdp" model="hr.leave.allocation">
        <field name="name">Summer Vacation</field>
        <field name="holiday_status_id" ref="holiday_status_unpaid"/>
        <field name="number_of_days">7</field>
        <field name="employee_id" ref="hr.employee_qdp"/>
    </record>
    <function model="hr.leave.allocation" name="action_approve">
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
        <field name="state">confirm</field>
    </record>
    <function model="hr.leave" name="action_approve">
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
        <field name="state">confirm</field>
    </record>
    <function model="hr.leave" name="action_approve">
        <value eval="ref('hr_holidays.hr_holidays_sl_qdp')"/>
    </function>

    <!-- ++++++++++++++++++++++  Audrey Peterson  ++++++++++++++++++++++ -->

    <record id="hr_holidays_allocation_cl_fpi" model="hr.leave.allocation">
        <field name="name">Paid Time Off for Audrey Peterson</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field name="number_of_days">20</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
        <field name="state">validate</field>
    </record>

    <record id="hr_holidays_vc_fpi" model="hr.leave.allocation">
        <field name="name">Summer Vacation</field>
        <field name="holiday_status_id" ref="holiday_status_unpaid"/>
        <field name="number_of_days">7</field>
        <field name="employee_id" ref="hr.employee_fpi"/>
    </record>

    <!-- ++++++++++++++++++++++   Olivia  ++++++++++++++++++++++ -->

    <record id="hr_holidays_allocation_cl_vad" model="hr.leave.allocation">
        <field name="name">Paid Time Off for Olivia</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field name="number_of_days">20</field>
        <field name="employee_id" ref="hr.employee_niv"/>
        <field name="state">validate</field>
    </record>

    <record id="hr_holidays_vc_vad" model="hr.leave.allocation">
        <field name="name">Summer Vacation</field>
        <field name="holiday_status_id" ref="holiday_status_unpaid"/>
        <field name="number_of_days">5</field>
        <field name="employee_id" ref="hr.employee_niv"/>
    </record>
    <function model="hr.leave.allocation" name="action_approve">
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
        <field name="state">confirm</field>
    </record>
    <function model="hr.leave" name="action_approve">
        <value eval="ref('hr_holidays.hr_holidays_sl_vad')"/>
    </function>

    <!-- ++++++++++++++++++++++   Kim  ++++++++++++++++++++++ -->

    <record id="hr_holidays_allocation_cl_kim" model="hr.leave.allocation">
        <field name="name">Paid Time Off for Kim</field>
        <field name="holiday_status_id" ref="holiday_status_cl"/>
        <field name="number_of_days">20</field>
        <field name="employee_id" ref="hr.employee_jve"/>
        <field name="state">confirm</field>
    </record>

    <record id="hr_holidays_vc_kim" model="hr.leave.allocation">
        <field name="name">Summer Vacation</field>
        <field name="holiday_status_id" ref="holiday_status_unpaid"/>
        <field name="number_of_days">5</field>
        <field name="employee_id" ref="hr.employee_jve"/>
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
            <field name="res_model_id" ref="hr_holidays.model_hr_leave"/>
        </record>
        <record id="mail_act_leave_second_approval" model="mail.activity.type">
            <field name="name">Time Off Second Approve</field>
            <field name="icon">fa-sun-o</field>
            <field name="res_model_id" ref="hr_holidays.model_hr_leave"/>
        </record>

        <!-- Leave specific activities -->
        <record id="mail_act_leave_allocation_approval" model="mail.activity.type">
            <field name="name">Allocation Approval</field>
            <field name="icon">fa-sun-o</field>
            <field name="res_model_id" ref="hr_holidays.model_hr_leave_allocation"/>
        </record>
        <record id="mail_act_leave_allocation_second_approval" model="mail.activity.type">
            <field name="name">Allocation Second Approve</field>
            <field name="icon">fa-sun-o</field>
            <field name="res_model_id" ref="hr_holidays.model_hr_leave_allocation"/>
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


class Department(models.Model):

    _inherit = 'hr.department'

    absence_of_today = fields.Integer(
        compute='_compute_leave_count', string='Absence by Today')
    leave_to_approve_count = fields.Integer(
        compute='_compute_leave_count', string='Time Off to Approve')
    allocation_to_approve_count = fields.Integer(
        compute='_compute_leave_count', string='Allocation to Approve')
    total_employee = fields.Integer(
        compute='_compute_total_employee', string='Total Employee')

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

    def _compute_total_employee(self):
        emp_data = self.env['hr.employee'].read_group([('department_id', 'in', self.ids)], ['department_id'], ['department_id'])
        result = dict((data['department_id'][0], data['department_id_count']) for data in emp_data)
        for department in self:
            department.total_employee = result.get(department.id, 0)

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
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
    current_leave_id = fields.Many2one('hr.leave.type', compute='_compute_leave_status', string="Current Time Off Type")
    leave_date_from = fields.Date('From Date', compute='_compute_leave_status')
    leave_date_to = fields.Date('To Date', compute='_compute_leave_status')
    leaves_count = fields.Float('Number of Time Off', compute='_compute_remaining_leaves')
    allocation_count = fields.Float('Total number of days allocated.', compute='_compute_allocation_count')
    allocation_used_count = fields.Float('Total number of days off used', compute='_compute_total_allocation_used')
    show_leaves = fields.Boolean('Able to see Remaining Time Off', compute='_compute_show_leaves')
    is_absent = fields.Boolean('Absent Today', compute='_compute_leave_status', search='_search_absent_employee')
    allocation_display = fields.Char(compute='_compute_allocation_count')
    allocation_used_display = fields.Char(compute='_compute_total_allocation_used')
    hr_icon_display = fields.Selection(selection_add=[('presence_holiday_absent', 'On leave'),
                                                      ('presence_holiday_present', 'Present but on leave')])

    def _get_date_start_work(self):
        return self.create_date

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
                (s.allocation_type='fixed' OR s.allocation_type='fixed_allocation') AND
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
        rg_results = dict((d['employee_id'][0], d['number_of_days']) for d in data)
        for employee in self:
            employee.allocation_count = float_round(rg_results.get(employee.id, 0.0), precision_digits=2)
            employee.allocation_display = "%g" % employee.allocation_count

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
            leave_data[holiday.employee_id.id]['current_leave_id'] = holiday.holiday_status_id.id

        for employee in self:
            employee.leave_date_from = leave_data.get(employee.id, {}).get('leave_date_from')
            employee.leave_date_to = leave_data.get(employee.id, {}).get('leave_date_to')
            employee.current_leave_state = leave_data.get(employee.id, {}).get('current_leave_state')
            employee.current_leave_id = leave_data.get(employee.id, {}).get('current_leave_id')
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
            ('state', '=', 'validate1'),
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

class HrEmployeePrivate(models.Model):
    _inherit = 'hr.employee'

class HrEmployeePublic(models.Model):
    _inherit = 'hr.employee.public'

    def _compute_leave_status(self):
        super()._compute_leave_status()
        self.current_leave_id = False

```

## File: models\hr_leave.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2005-2006 Axelor SARL. (http://www.axelor.com)

import logging
import math

from collections import defaultdict, namedtuple

from datetime import datetime, date, timedelta, time
from dateutil.rrule import rrule, DAILY
from pytz import timezone, UTC

from odoo import api, fields, models, SUPERUSER_ID, tools
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

        if 'holiday_status_id' in fields_list and not defaults.get('holiday_status_id'):
            lt = self.env['hr.leave.type'].search([('valid', '=', True)], limit=1)

            if lt:
                defaults['holiday_status_id'] = lt.id
                defaults['request_unit_custom'] = False

        if 'state' in fields_list and not defaults.get('state'):
            lt = self.env['hr.leave.type'].browse(defaults.get('holiday_status_id'))
            defaults['state'] = 'confirm'

        now = fields.Datetime.now()
        if 'date_from' not in defaults:
            defaults.update({'date_from': now})
        if 'date_to' not in defaults:
            defaults.update({'date_to': now})
        return defaults

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
        ('cancel', 'Cancelled'),  # YTI This state seems to be unused. To remove
        ('confirm', 'To Approve'),
        ('refuse', 'Refused'),
        ('validate1', 'Second Approval'),
        ('validate', 'Approved')
        ], string='Status', compute='_compute_state', store=True, tracking=True, copy=False, readonly=False,
        help="The status is set to 'To Submit', when a time off request is created." +
        "\nThe status is 'To Approve', when time off request is confirmed by user." +
        "\nThe status is 'Refused', when time off request is refused by manager." +
        "\nThe status is 'Approved', when time off request is approved by manager.")
    payslip_status = fields.Boolean('Reported in last payslips', help='Green this button when the time off has been taken into account in the payslip.', copy=False)
    report_note = fields.Text('HR Comments', copy=False, groups="hr_holidays.group_hr_holidays_manager")
    user_id = fields.Many2one('res.users', string='User', related='employee_id.user_id', related_sudo=True, compute_sudo=True, store=True, default=lambda self: self.env.uid, readonly=True)
    manager_id = fields.Many2one('hr.employee', compute='_compute_from_employee_id', store=True, readonly=False)
    # leave type configuration
    holiday_status_id = fields.Many2one(
        "hr.leave.type", compute='_compute_from_employee_id', store=True, string="Time Off Type", required=True, readonly=False,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]},
        domain=[('valid', '=', True)])
    validation_type = fields.Selection(string='Validation Type', related='holiday_status_id.leave_validation_type', readonly=False)
    # HR data

    employee_id = fields.Many2one(
        'hr.employee', compute='_compute_from_holiday_type', store=True, string='Employee', index=True, readonly=False, ondelete="restrict",
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]},
        tracking=True)
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

    # UX fields
    leave_type_request_unit = fields.Selection(related='holiday_status_id.request_unit', readonly=True)
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

    _sql_constraints = [
        ('type_value',
         "CHECK((holiday_type='employee' AND employee_id IS NOT NULL) or "
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
        for holiday in self:
            if self.env.context.get('unlink') and holiday.state == 'draft':
                # Otherwise the record in draft with validation_type in (hr, manager, both) will be set to confirm
                # and a simple internal user will not be able to delete his own draft record
                holiday.state = 'draft'
            else:
                holiday.state = 'confirm' if holiday.validation_type != 'no_validation' else 'draft'

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
                    start_week_type = int(math.floor((holiday.request_date_from.toordinal() - 1) / 7) % 2)
                    attendance_actual_week = [att for att in attendances if att.week_type is False or int(att.week_type) == start_week_type]
                    attendance_actual_next_week = [att for att in attendances if att.week_type is False or int(att.week_type) != start_week_type]
                    # First, add days of actual week coming after date_from
                    attendance_filtred = [att for att in attendance_actual_week if int(att.dayofweek) >= holiday.request_date_from.weekday()]
                    # Second, add days of the other type of week
                    attendance_filtred += list(attendance_actual_next_week)
                    # Third, add days of actual week (to consider days that we have remove first because they coming before date_from)
                    attendance_filtred += list(attendance_actual_week)

                    end_week_type = int(math.floor((holiday.request_date_to.toordinal() - 1) / 7) % 2)
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

    @api.depends('holiday_type')
    def _compute_from_holiday_type(self):
        for holiday in self:
            if holiday.holiday_type == 'employee':
                if not holiday.employee_id:
                    holiday.employee_id = self.env.user.employee_id
                holiday.mode_company_id = False
                holiday.category_id = False
            elif holiday.holiday_type == 'company':
                holiday.employee_id = False
                if not holiday.mode_company_id:
                    holiday.mode_company_id = self.env.company.id
                holiday.category_id = False
            elif holiday.holiday_type == 'department':
                holiday.employee_id = False
                holiday.mode_company_id = False
                holiday.category_id = False
            elif holiday.holiday_type == 'category':
                holiday.employee_id = False
                holiday.mode_company_id = False
            else:
                holiday.employee_id = self.env.context.get('default_employee_id') or self.env.user.employee_id

    @api.depends('employee_id')
    def _compute_from_employee_id(self):
        for holiday in self:
            holiday.manager_id = holiday.employee_id.parent_id.id
            if holiday.employee_id.user_id != self.env.user and self._origin.employee_id != holiday.employee_id:
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
                raise ValidationError(_('You can not set 2 time off that overlaps on the same day for the same employee.'))

    @api.constrains('state', 'number_of_days', 'holiday_status_id')
    def _check_holidays(self):
        mapped_days = self.mapped('holiday_status_id').get_employees_days(self.mapped('employee_id').ids)
        for holiday in self:
            if holiday.holiday_type != 'employee' or not holiday.employee_id or holiday.holiday_status_id.allocation_type == 'no':
                continue
            leave_days = mapped_days[holiday.employee_id.id][holiday.holiday_status_id.id]
            if float_compare(leave_days['remaining_leaves'], 0, precision_digits=2) == -1 or float_compare(leave_days['virtual_remaining_leaves'], 0, precision_digits=2) == -1:
                raise ValidationError(_('The number of remaining time off is not sufficient for this time off type.\n'
                                        'Please also check the time off waiting for validation.'))

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
            domain = [('company_id', 'in', self.env.company.ids + self.env.context.get('allowed_company_ids', []))]
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

        E.g. a leave in US/Pacific for one day:
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
                else:
                    target = leave.employee_id.name
                display_date = format_date(self.env, leave.date_from)
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
                    if leave.number_of_days > 1:
                        display_date += ' ⇨ %s' % format_date(self.env, leave.date_to)
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

    @api.constrains('holiday_status_id', 'date_to', 'date_from')
    def _check_leave_type_validity(self):
        for leave in self:
            vstart = leave.holiday_status_id.validity_start
            vstop = leave.holiday_status_id.validity_stop
            dfrom = leave.date_from
            dto = leave.date_to
            if leave.holiday_status_id.validity_start and leave.holiday_status_id.validity_stop:
                if dfrom and dto and (dfrom.date() < vstart or dto.date() > vstop):
                    raise ValidationError(_(
                        '%(leave_type)s are only valid between %(start)s and %(end)s',
                        leave_type=leave.holiday_status_id.display_name,
                        start=leave.holiday_status_id.validity_start,
                        end=leave.holiday_status_id.validity_stop
                    ))
            elif leave.holiday_status_id.validity_start:
                if dfrom and (dfrom.date() < vstart):
                    raise ValidationError(_(
                        '%(leave_type)s are only valid starting from %(date)s',
                        leave_type=leave.holiday_status_id.display_name,
                        date=leave.holiday_status_id.validity_start
                    ))
            elif leave.holiday_status_id.validity_stop:
                if dto and (dto.date() > vstop):
                    raise ValidationError(_(
                        '%(leave_type)s are only valid until %(date)s',
                        leave_type=leave.holiday_status_id.display_name,
                        date=leave.holiday_status_id.validity_stop
                    ))

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
                holiday_sudo.add_follower(employee_id)
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

        if not is_officer and values.keys() - {'message_main_attachment_id'}:
            if any(hol.date_from.date() < fields.Date.today() and hol.employee_id.leave_manager_id != self.env.user for hol in self):
                raise UserError(_('You must have manager rights to modify/validate a time off that already begun'))

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

    def unlink(self):
        error_message = _('You cannot delete a time off which is in %s state')
        state_description_values = {elem[0]: elem[1] for elem in self._fields['state']._description_selection(self.env)}

        if not self.user_has_groups('hr_holidays.group_hr_holidays_user'):
            if any(hol.state != 'draft' for hol in self):
                raise UserError(error_message % state_description_values.get(self[:1].state))
        else:
            for holiday in self.filtered(lambda holiday: holiday.state not in ['draft', 'cancel', 'confirm']):
                raise UserError(error_message % (state_description_values.get(holiday.state),))
        return super(HolidaysRequest, self.with_context(leave_skip_date_check=True, unlink=True)).unlink()

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
            'name': leave.name,
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
        holidays = self.filtered(lambda request: request.holiday_type == 'employee')
        holidays._create_resource_leave()
        meeting_holidays = holidays.filtered(lambda l: l.holiday_status_id.create_calendar_meeting)
        meetings = self.env['calendar.event']
        if meeting_holidays:
            meeting_values_for_user_id = meeting_holidays._prepare_holidays_meeting_values()
            for user_id, meeting_values in meeting_values_for_user_id.items():
                meetings += self.env['calendar.event'].with_user(user_id or self.env.uid).with_context(
                                allowed_company_ids=[],
                                no_mail_to_attendees=True,
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

    # YTI TODO: Remove me in master
    def _prepare_holiday_values(self, employee):
        return self._prepare_employees_holiday_values(employee)[0]

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
            holiday.message_post(
                body=_(
                    'Your %(leave_type)s planned on %(date)s has been accepted',
                    leave_type=holiday.holiday_status_id.display_name,
                    date=holiday.date_from
                ),
                partner_ids=holiday.employee_id.user_id.partner_id.ids)

        self.filtered(lambda hol: not hol.validation_type == 'both').action_validate()
        if not self.env.context.get('leave_fast_create'):
            self.activity_update()
        return True

    def action_validate(self):
        current_employee = self.env.user.employee_id
        leaves = self.filtered(lambda l: l.employee_id and not l.number_of_days)
        if leaves:
            raise ValidationError(_('The following employees are not supposed to work during that period:\n %s') % ','.join(leaves.mapped('employee_id.name')))

        if any(holiday.state not in ['confirm', 'validate1'] and holiday.validation_type != 'no_validation' for holiday in self):
            raise UserError(_('Time off request must be confirmed in order to approve it.'))

        self.write({'state': 'validate'})
        self.filtered(lambda holiday: holiday.validation_type == 'both').write({'second_approver_id': current_employee.id})
        self.filtered(lambda holiday: holiday.validation_type != 'both').write({'first_approver_id': current_employee.id})

        for holiday in self.filtered(lambda holiday: holiday.holiday_type != 'employee'):
            if holiday.holiday_type == 'category':
                employees = holiday.category_id.employee_ids
            elif holiday.holiday_type == 'company':
                employees = self.env['hr.employee'].search([('company_id', '=', holiday.mode_company_id.id)])
            else:
                employees = holiday.department_id.member_ids

            conflicting_leaves = self.env['hr.leave'].with_context(
                tracking_disable=True,
                mail_activity_automation_skip=True,
                leave_fast_create=True
            ).search([
                ('date_from', '<=', holiday.date_to),
                ('date_to', '>', holiday.date_from),
                ('state', 'not in', ['cancel', 'refuse']),
                ('holiday_type', '=', 'employee'),
                ('employee_id', 'in', employees.ids)])

            if conflicting_leaves:
                # YTI: More complex use cases could be managed in master
                if holiday.leave_type_request_unit != 'day' or any(l.leave_type_request_unit == 'hour' for l in conflicting_leaves):
                    raise ValidationError(_('You can not have 2 time off that overlaps on the same day.'))

                # keep track of conflicting leaves states before refusal
                target_states = {l.id: l.state for l in conflicting_leaves}
                conflicting_leaves.action_refuse()
                split_leaves_vals = []
                for conflicting_leave in conflicting_leaves:
                    if conflicting_leave.leave_type_request_unit == 'half_day' and conflicting_leave.request_unit_half:
                        continue

                    # Leaves in days
                    if conflicting_leave.date_from < holiday.date_from:
                        before_leave_vals = conflicting_leave.copy_data({
                            'date_from': conflicting_leave.date_from.date(),
                            'date_to': holiday.date_from.date() + timedelta(days=-1),
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
                    if conflicting_leave.date_to > holiday.date_to:
                        after_leave_vals = conflicting_leave.copy_data({
                            'date_from': holiday.date_to.date() + timedelta(days=1),
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

            values = holiday._prepare_employees_holiday_values(employees)
            leaves = self.env['hr.leave'].with_context(
                tracking_disable=True,
                mail_activity_automation_skip=True,
                leave_fast_create=True,
                leave_skip_state_check=True,
            ).create(values)

            leaves._validate_leave_request()

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

        self._remove_resource_leave()
        self.activity_update()
        return True

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

                    if (state == 'validate1' and val_type == 'both') or (state == 'validate' and val_type == 'manager') and holiday.holiday_type == 'employee':
                        if not is_officer and self.env.user != holiday.employee_id.leave_manager_id:
                            raise UserError(_('You must be either %s\'s manager or Time off Manager to approve this leave') % (holiday.employee_id.name))

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
            start = UTC.localize(holiday.date_from).astimezone(timezone(holiday.employee_id.tz or 'UTC'))
            end = UTC.localize(holiday.date_to).astimezone(timezone(holiday.employee_id.tz or 'UTC'))
            note = _(
                'New %(leave_type)s Request created by %(user)s from %(start)s to %(end)s',
                leave_type=holiday.holiday_status_id.name,
                user=holiday.create_uid.name,
                start=start,
                end=end
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

    def message_subscribe(self, partner_ids=None, channel_ids=None, subtype_ids=None):
        # due to record rule can not allow to add follower and mention on validated leave so subscribe through sudo
        if self.state in ['validate', 'validate1']:
            self.check_access_rights('read')
            self.check_access_rule('read')
            return super(HolidaysRequest, self.sudo()).message_subscribe(partner_ids=partner_ids, channel_ids=channel_ids, subtype_ids=subtype_ids)
        return super(HolidaysRequest, self).message_subscribe(partner_ids=partner_ids, channel_ids=channel_ids, subtype_ids=subtype_ids)

    @api.model
    def get_unusual_days(self, date_from, date_to=None):
        # Checking the calendar directly allows to not grey out the leaves taken
        # by the employee
        calendar = self.env.user.employee_id.resource_calendar_id
        if not calendar:
            return {}
        dfrom = datetime.combine(fields.Date.from_string(date_from), time.min).replace(tzinfo=UTC)
        dto = datetime.combine(fields.Date.from_string(date_to), time.max).replace(tzinfo=UTC)

        works = {d[0].date() for d in calendar._work_intervals_batch(dfrom, dto)[False]}
        return {fields.Date.to_string(day.date()): (day.date() not in works) for day in rrule(DAILY, dfrom, until=dto)}

```

## File: models\hr_leave_allocation.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2005-2006 Axelor SARL. (http://www.axelor.com)

import logging

from datetime import datetime, time
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models
from odoo.addons.resource.models.resource import HOURS_PER_DAY
from odoo.exceptions import AccessError, UserError, ValidationError
from odoo.tools.translate import _
from odoo.tools.float_utils import float_round
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
            domain = [('valid', '=', True)]
        else:
            domain = [('valid', '=', True), ('allocation_type', '=', 'fixed_allocation')]
        return self.env['hr.leave.type'].search(domain, limit=1)

    def _holiday_status_id_domain(self):
        if self.user_has_groups('hr_holidays.group_hr_holidays_manager'):
            return [('valid', '=', True), ('allocation_type', '!=', 'no')]
        return [('valid', '=', True), ('allocation_type', '=', 'fixed_allocation')]

    name = fields.Char('Description', compute='_compute_description', inverse='_inverse_description', search='_search_description', compute_sudo=False)
    private_name = fields.Char('Allocation Description', groups='hr_holidays.group_hr_holidays_user')
    state = fields.Selection([
        ('draft', 'To Submit'),
        ('cancel', 'Cancelled'),
        ('confirm', 'To Approve'),
        ('refuse', 'Refused'),
        ('validate1', 'Second Approval'),
        ('validate', 'Approved')
        ], string='Status', readonly=True, tracking=True, copy=False, default='confirm',
        help="The status is set to 'To Submit', when an allocation request is created." +
        "\nThe status is 'To Approve', when an allocation request is confirmed by user." +
        "\nThe status is 'Refused', when an allocation request is refused by manager." +
        "\nThe status is 'Approved', when an allocation request is approved by manager.")
    date_from = fields.Datetime(
        'Start Date', readonly=True, index=True, copy=False, default=fields.Date.context_today,
        states={'draft': [('readonly', False)], 'confirm': [('readonly', False)]}, tracking=True)
    date_to = fields.Datetime(
        'End Date', compute='_compute_from_holiday_status_id', store=True, readonly=False, copy=False, tracking=True,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]})
    holiday_status_id = fields.Many2one(
        "hr.leave.type", compute='_compute_from_employee_id', store=True, string="Time Off Type", required=True, readonly=False,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]},
        domain=_holiday_status_id_domain)
    employee_id = fields.Many2one(
        'hr.employee', compute='_compute_from_holiday_type', store=True, string='Employee', index=True, readonly=False, ondelete="restrict", tracking=True,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]})
    manager_id = fields.Many2one('hr.employee', compute='_compute_from_employee_id', store=True, string='Manager')
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
    first_approver_id = fields.Many2one(
        'hr.employee', string='First Approval', readonly=True, copy=False,
        help='This area is automatically filled by the user who validates the allocation')
    second_approver_id = fields.Many2one(
        'hr.employee', string='Second Approval', readonly=True, copy=False,
        help='This area is automatically filled by the user who validates the allocation with second level (If allocation type need second validation)')
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
    mode_company_id = fields.Many2one(
        'res.company', compute='_compute_from_holiday_type', store=True, string='Company Mode', readonly=False,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]})
    department_id = fields.Many2one(
        'hr.department', compute='_compute_department_id', store=True, string='Department',
        states={'draft': [('readonly', False)], 'confirm': [('readonly', False)]})
    category_id = fields.Many2one(
        'hr.employee.category', compute='_compute_from_holiday_type', store=True, string='Employee Tag', readonly=False,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]})
    # accrual configuration
    allocation_type = fields.Selection(
        [
            ('regular', 'Regular Allocation'),
            ('accrual', 'Accrual Allocation')
        ], string="Allocation Type", default="regular", required=True, readonly=True,
        states={'draft': [('readonly', False)], 'confirm': [('readonly', False)]})
    accrual_limit = fields.Integer('Balance limit', default=0, help="Maximum of allocation for accrual; 0 means no maximum.")
    number_per_interval = fields.Float("Number of unit per interval", compute='_compute_from_holiday_status_id', store=True, readonly=False,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]})
    interval_number = fields.Integer("Number of unit between two intervals", compute='_compute_from_holiday_status_id', store=True, readonly=False,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]})
    unit_per_interval = fields.Selection([
        ('hours', 'Hours'),
        ('days', 'Days')
        ], compute='_compute_from_holiday_status_id', store=True, string="Unit of time added at each interval", readonly=False,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]})
    interval_unit = fields.Selection([
        ('days', 'Days'),
        ('weeks', 'Weeks'),
        ('months', 'Months'),
        ('years', 'Years')
        ], compute='_compute_from_holiday_status_id', store=True, string="Unit of time between two intervals", readonly=False,
        states={'cancel': [('readonly', True)], 'refuse': [('readonly', True)], 'validate1': [('readonly', True)], 'validate': [('readonly', True)]})
    nextcall = fields.Date("Date of the next accrual allocation", default=False, readonly=True)
    max_leaves = fields.Float(compute='_compute_leaves')
    leaves_taken = fields.Float(compute='_compute_leaves')

    _sql_constraints = [
        ('type_value',
         "CHECK( (holiday_type='employee' AND employee_id IS NOT NULL) or "
         "(holiday_type='category' AND category_id IS NOT NULL) or "
         "(holiday_type='department' AND department_id IS NOT NULL) or "
         "(holiday_type='company' AND mode_company_id IS NOT NULL))",
         "The employee, department, company or employee category of this request is missing. Please make sure that your user login is linked to an employee."),
        ('duration_check', "CHECK ( number_of_days >= 0 )", "The number of days must be greater than 0."),
        ('number_per_interval_check', "CHECK(number_per_interval > 0)", "The number per interval should be greater than 0"),
        ('interval_number_check', "CHECK(interval_number > 0)", "The interval number should be greater than 0"),
    ]

    @api.model
    def _update_accrual(self):
        """
            Method called by the cron task in order to increment the number_of_days when
            necessary.
        """
        today = fields.Date.from_string(fields.Date.today())

        holidays = self.search([('allocation_type', '=', 'accrual'), ('employee_id.active', '=', True), ('state', '=', 'validate'), ('holiday_type', '=', 'employee'),
                                '|', ('date_to', '=', False), ('date_to', '>', fields.Datetime.now()),
                                '|', ('nextcall', '=', False), ('nextcall', '<=', today)])

        for holiday in holidays:
            values = {}

            delta = relativedelta(days=0)

            if holiday.interval_unit == 'days':
                delta = relativedelta(days=holiday.interval_number)
            if holiday.interval_unit == 'weeks':
                delta = relativedelta(weeks=holiday.interval_number)
            if holiday.interval_unit == 'months':
                delta = relativedelta(months=holiday.interval_number)
            if holiday.interval_unit == 'years':
                delta = relativedelta(years=holiday.interval_number)

            if holiday.nextcall:
                values['nextcall'] = holiday.nextcall + delta
            else:
                values['nextcall'] = holiday.date_from
                while values['nextcall'] <= datetime.combine(today, time(0, 0, 0)):
                    values['nextcall'] += delta

            period_start = datetime.combine(today, time(0, 0, 0)) - delta
            period_end = datetime.combine(today, time(0, 0, 0))

            # We have to check when the employee has been created
            # in order to not allocate him/her too much leaves
            start_date = holiday.employee_id._get_date_start_work()
            # If employee is created after the period, we cancel the computation
            if period_end <= start_date or period_end < holiday.date_from:
                holiday.write(values)
                continue

            # If employee created during the period, taking the date at which he has been created
            if period_start <= start_date:
                period_start = start_date

            employee = holiday.employee_id
            worked = employee._get_work_days_data_batch(
                period_start, period_end,
                domain=[('holiday_id.holiday_status_id.unpaid', '=', True), ('time_type', '=', 'leave')]
            )[employee.id]['days']
            left = employee._get_leave_days_data_batch(
                period_start, period_end,
                domain=[('holiday_id.holiday_status_id.unpaid', '=', True), ('time_type', '=', 'leave')]
            )[employee.id]['days']
            prorata = worked / (left + worked) if worked else 0

            days_to_give = holiday.number_per_interval
            if holiday.unit_per_interval == 'hours':
                # As we encode everything in days in the database we need to convert
                # the number of hours into days for this we use the
                # mean number of hours set on the employee's calendar
                days_to_give = days_to_give / (employee.resource_calendar_id.hours_per_day or HOURS_PER_DAY)

            values['number_of_days'] = holiday.number_of_days + days_to_give * prorata
            if holiday.accrual_limit > 0:
                values['number_of_days'] = min(values['number_of_days'], holiday.accrual_limit)

            holiday.write(values)

    @api.depends_context('uid')
    def _compute_description(self):
        self.check_access_rights('read')
        self.check_access_rule('read')

        is_officer = self.env.user.has_group('hr_holidays.group_hr_holidays_user')

        for allocation in self:
            if is_officer or allocation.employee_id.user_id == self.env.user or allocation.manager_id == self.env.user:
                allocation.name = allocation.sudo().private_name
            else:
                allocation.name = '*****'

    def _inverse_description(self):
        is_officer = self.env.user.has_group('hr_holidays.group_hr_holidays_user')
        for allocation in self:
            if is_officer or allocation.employee_id.user_id == self.env.user or allocation.manager_id == self.env.user:
                allocation.sudo().private_name = allocation.name

    def _search_description(self, operator, value):
        is_officer = self.env.user.has_group('hr_holidays.group_hr_holidays_user')
        domain = [('private_name', operator, value)]

        if not is_officer:
            domain = expression.AND([domain, [('employee_id.user_id', '=', self.env.user.id)]])

        allocations = self.sudo().search(domain)
        return [('id', 'in', allocations.ids)]

    @api.depends('employee_id', 'holiday_status_id')
    def _compute_leaves(self):
        for allocation in self:
            leave_type = allocation.holiday_status_id.with_context(employee_id=allocation.employee_id.id)
            allocation.max_leaves = leave_type.max_leaves
            allocation.leaves_taken = leave_type.leaves_taken

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
                if allocation.state == 'confirm' and allocation.holiday_status_id.allocation_type == "fixed_allocation" and allocation.validation_type == 'both':
                    allocation._check_approval_update('validate1')
                else:
                    allocation._check_approval_update('validate')
            except (AccessError, UserError):
                allocation.can_approve = False
            else:
                allocation.can_approve = True

    @api.depends('holiday_type')
    def _compute_from_holiday_type(self):
        for allocation in self:
            if allocation.holiday_type == 'employee':
                if not allocation.employee_id:
                    allocation.employee_id = self.env.user.employee_id
                allocation.mode_company_id = False
                allocation.category_id = False
            if allocation.holiday_type == 'company':
                allocation.employee_id = False
                if not allocation.mode_company_id:
                    allocation.mode_company_id = self.env.company
                allocation.category_id = False
            elif allocation.holiday_type == 'department':
                allocation.employee_id = False
                allocation.mode_company_id = False
                allocation.category_id = False
            elif allocation.holiday_type == 'category':
                allocation.employee_id = False
                allocation.mode_company_id = False
            elif not allocation.employee_id and not allocation._origin.employee_id:
                allocation.employee_id = self.env.context.get('default_employee_id') or self.env.user.employee_id

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
    def _compute_from_employee_id(self):
        default_holiday_status_id = self._default_holiday_status_id()
        for holiday in self:
            holiday.manager_id = holiday.employee_id and holiday.employee_id.parent_id
            if holiday.employee_id.user_id != self.env.user and holiday._origin.employee_id != holiday.employee_id:
                holiday.holiday_status_id = False
            elif not holiday.holiday_status_id and not holiday._origin.holiday_status_id:
                holiday.holiday_status_id = default_holiday_status_id

    @api.depends('holiday_status_id', 'allocation_type', 'number_of_hours_display', 'number_of_days_display')
    def _compute_from_holiday_status_id(self):
        for allocation in self:
            allocation.number_of_days = allocation.number_of_days_display
            if allocation.type_request_unit == 'hour':
                allocation.number_of_days = allocation.number_of_hours_display / (allocation.employee_id.sudo().resource_calendar_id.hours_per_day or HOURS_PER_DAY)

            # set default values
            if not allocation.interval_number and not allocation._origin.interval_number:
                allocation.interval_number = 1
            if not allocation.number_per_interval and not allocation._origin.number_per_interval:
                allocation.number_per_interval = 1
            if not allocation.unit_per_interval and not allocation._origin.unit_per_interval:
                allocation.unit_per_interval = 'hours'
            if not allocation.interval_unit and not allocation._origin.interval_unit:
                allocation.interval_unit = 'weeks'

            if allocation.holiday_status_id.validity_stop and allocation.date_to:
                new_date_to = datetime.combine(allocation.holiday_status_id.validity_stop, time.max)
                if new_date_to < allocation.date_to:
                    allocation.date_to = new_date_to

            if allocation.allocation_type == 'accrual':
                if allocation.holiday_status_id.request_unit == 'hour':
                    allocation.unit_per_interval = 'hours'
                else:
                    allocation.unit_per_interval = 'days'
            else:
                allocation.interval_number = 1
                allocation.interval_unit = 'weeks'
                allocation.number_per_interval = 1
                allocation.unit_per_interval = 'hours'

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
            else:
                target = allocation.employee_id.sudo().name

            res.append(
                (allocation.id,
                 _("Allocation of %(allocation_name)s : %(duration).2f %(duration_type)s to %(person)s",
                   allocation_name=allocation.holiday_status_id.sudo().name,
                   duration=allocation.number_of_hours_display if allocation.type_request_unit == 'hour' else allocation.number_of_days,
                   duration_type='hours' if allocation.type_request_unit == 'hour' else 'days',
                   person=target
                ))
            )
        return res

    def add_follower(self, employee_id):
        employee = self.env['hr.employee'].browse(employee_id)
        if employee.user_id:
            self.message_subscribe(partner_ids=employee.user_id.partner_id.ids)

    @api.constrains('holiday_status_id')
    def _check_leave_type_validity(self):
        for allocation in self:
            if allocation.holiday_status_id.validity_stop:
                vstop = allocation.holiday_status_id.validity_stop
                today = fields.Date.today()

                if vstop < today:
                    raise ValidationError(_(
                        'You can allocate %(allocation_type)s only before %(date)s.',
                        allocation_type=allocation.holiday_status_id.display_name,
                        date=allocation.holiday_status_id.validity_stop
                    ))

    @api.model
    def create(self, values):
        """ Override to avoid automatic logging of creation """
        employee_id = values.get('employee_id', False)
        if not values.get('department_id'):
            values.update({'department_id': self.env['hr.employee'].browse(employee_id).department_id.id})
        holiday = super(HolidaysAllocation, self.with_context(mail_create_nosubscribe=True)).create(values)
        holiday.add_follower(employee_id)
        if holiday.validation_type == 'hr':
            holiday.message_subscribe(partner_ids=(holiday.employee_id.parent_id.user_id.partner_id | holiday.employee_id.leave_manager_id.partner_id).ids)
        if not self._context.get('import_file'):
            holiday.activity_update()
        return holiday

    def write(self, values):
        employee_id = values.get('employee_id', False)
        if values.get('state'):
            self._check_approval_update(values['state'])
        result = super(HolidaysAllocation, self).write(values)
        self.add_follower(employee_id)
        return result

    def unlink(self):
        state_description_values = {elem[0]: elem[1] for elem in self._fields['state']._description_selection(self.env)}
        for holiday in self.filtered(lambda holiday: holiday.state not in ['draft', 'cancel', 'confirm']):
            raise UserError(_('You cannot delete an allocation request which is in %s state.') % (state_description_values.get(holiday.state),))
        return super(HolidaysAllocation, self).unlink()

    def _get_mail_redirect_suggested_company(self):
        return self.holiday_status_id.company_id

    ####################################################
    # Business methods
    ####################################################

    def _prepare_holiday_values(self, employee):
        self.ensure_one()
        values = {
            'name': self.name,
            'holiday_type': 'employee',
            'holiday_status_id': self.holiday_status_id.id,
            'notes': self.notes,
            'number_of_days': self.number_of_days,
            'parent_id': self.id,
            'employee_id': employee.id,
            'allocation_type': self.allocation_type,
            'date_from': self.date_from,
            'date_to': self.date_to,
            'interval_unit': self.interval_unit,
            'interval_number': self.interval_number,
            'number_per_interval': self.number_per_interval,
            'unit_per_interval': self.unit_per_interval,
        }
        return values

    def action_draft(self):
        if any(holiday.state not in ['confirm', 'refuse'] for holiday in self):
            raise UserError(_('Allocation request state must be "Refused" or "To Approve" in order to be reset to Draft.'))
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
            raise UserError(_('Allocation request must be in Draft state ("To Submit") in order to confirm it.'))
        res = self.write({'state': 'confirm'})
        self.activity_update()
        return res

    def action_approve(self):
        # if validation_type == 'both': this method is the first approval approval
        # if validation_type != 'both': this method calls action_validate() below
        if any(holiday.state != 'confirm' for holiday in self):
            raise UserError(_('Allocation request must be confirmed ("To Approve") in order to approve it.'))

        current_employee = self.env.user.employee_id

        self.filtered(lambda hol: hol.validation_type == 'both').write({'state': 'validate1', 'first_approver_id': current_employee.id})
        self.filtered(lambda hol: not hol.validation_type == 'both').action_validate()
        self.activity_update()

    def action_validate(self):
        current_employee = self.env.user.employee_id
        for holiday in self:
            if holiday.state not in ['confirm', 'validate1']:
                raise UserError(_('Allocation request must be confirmed in order to approve it.'))

            holiday.write({'state': 'validate'})
            if holiday.validation_type == 'both':
                holiday.write({'second_approver_id': current_employee.id})
            else:
                holiday.write({'first_approver_id': current_employee.id})

            holiday._action_validate_create_childs()
        self.activity_update()
        return True

    def _action_validate_create_childs(self):
        childs = self.env['hr.leave.allocation']
        if self.state == 'validate' and self.holiday_type in ['category', 'department', 'company']:
            if self.holiday_type == 'category':
                employees = self.category_id.employee_ids
            elif self.holiday_type == 'department':
                employees = self.department_id.member_ids
            else:
                employees = self.env['hr.employee'].search([('company_id', '=', self.mode_company_id.id)])

            for employee in employees:
                childs += self.with_context(
                    mail_notify_force_send=False,
                    mail_activity_automation_skip=True
                ).create(self._prepare_holiday_values(employee))
            # TODO is it necessary to interleave the calls?
            childs.action_approve()
            if childs and self.validation_type == 'both':
                childs.action_validate()
        return childs

    def action_refuse(self):
        current_employee = self.env.user.employee_id
        if any(holiday.state not in ['confirm', 'validate', 'validate1'] for holiday in self):
            raise UserError(_('Allocation request must be confirmed or validated in order to refuse it.'))

        validated_holidays = self.filtered(lambda hol: hol.state == 'validate1')
        validated_holidays.write({'state': 'refuse', 'first_approver_id': current_employee.id})
        (self - validated_holidays).write({'state': 'refuse', 'second_approver_id': current_employee.id})
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

            if not is_officer and self.env.user != holiday.employee_id.leave_manager_id:
                raise UserError(_('Only a time off Officer/Responsible or Manager can approve or refuse time off requests.'))

            if is_officer or self.env.user == holiday.employee_id.leave_manager_id:
                # use ir.rule based first access check: department, members, ... (see security.xml)
                holiday.check_access_rule('write')

            if holiday.employee_id == current_employee and not is_manager:
                raise UserError(_('Only a time off Manager can approve its own requests.'))

            if (state == 'validate1' and val_type == 'both') or (state == 'validate' and val_type == 'manager'):
                if self.env.user == holiday.employee_id.leave_manager_id and self.env.user != holiday.employee_id.user_id:
                    continue
                manager = holiday.employee_id.parent_id or holiday.employee_id.department_id.manager_id
                if (manager != current_employee) and not is_manager:
                    raise UserError(_('You must be either %s\'s manager or time off manager to approve this time off') % (holiday.employee_id.name))

            if state == 'validate' and val_type == 'both':
                if not is_officer:
                    raise UserError(_('Only a Time off Approver can apply the second approval on allocation requests.'))

    # ------------------------------------------------------------
    # Activity methods
    # ------------------------------------------------------------

    def _get_responsible_for_approval(self):
        self.ensure_one()
        responsible = self.env.user

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

    def message_subscribe(self, partner_ids=None, channel_ids=None, subtype_ids=None):
        # due to record rule can not allow to add follower and mention on validated leave so subscribe through sudo
        if self.state in ['validate', 'validate1']:
            self.check_access_rights('read')
            self.check_access_rule('read')
            return super(HolidaysAllocation, self.sudo()).message_subscribe(partner_ids=partner_ids, channel_ids=channel_ids, subtype_ids=subtype_ids)
        return super(HolidaysAllocation, self).message_subscribe(partner_ids=partner_ids, channel_ids=channel_ids, subtype_ids=subtype_ids)

```

## File: models\hr_leave_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2005-2006 Axelor SARL. (http://www.axelor.com)

import datetime
import logging

from collections import defaultdict

from odoo import api, fields, models
from odoo.exceptions import ValidationError
from odoo.osv import expression
from odoo.tools.translate import _
from odoo.tools.float_utils import float_round

_logger = logging.getLogger(__name__)


class HolidaysType(models.Model):
    _name = "hr.leave.type"
    _description = "Time Off Type"

    @api.model
    def _model_sorting_key(self, leave_type):
        remaining = leave_type.virtual_remaining_leaves > 0
        taken = leave_type.leaves_taken > 0
        return leave_type.allocation_type == 'fixed' and remaining, leave_type.allocation_type == 'fixed_allocation' and remaining, taken

    name = fields.Char('Time Off Type', required=True, translate=True)
    code = fields.Char('Code')
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
    group_days_allocation = fields.Float(
        compute='_compute_group_days_allocation', string='Days Allocated')
    group_days_leave = fields.Float(
        compute='_compute_group_days_leave', string='Group Time Off')
    company_id = fields.Many2one('res.company', string='Company', default=lambda self: self.env.company)
    responsible_id = fields.Many2one('res.users', 'Responsible',
        domain=lambda self: [('groups_id', 'in', self.env.ref('hr_holidays.group_hr_holidays_user').id)],
        help="This user will be responsible for approving this type of time off. "
        "This is only used when validation is 'By Time Off Officer' or 'By Employee's Manager and Time Off Officer'",)
    leave_validation_type = fields.Selection([
        ('no_validation', 'No Validation'),
        ('hr', 'By Time Off Officer'),
        ('manager', "By Employee's Manager"),
        ('both', "By Employee's Manager and Time Off Officer")], default='hr', string='Leave Validation')
    allocation_validation_type = fields.Selection([
        ('hr', 'By Time Off Officer'),
        ('manager', "By Employee's Manager"),
        ('both', "By Employee's Manager and Time Off Officer")], default='manager', string='Allocation Validation')
    allocation_type = fields.Selection([
        ('no', 'No Limit'),
        ('fixed_allocation', 'Allow Employees Requests'),
        ('fixed', 'Set by Time Off Officer')],
        default='no', string='Mode',
        help='\tNo Limit: no allocation by default, users can freely request time off; '
             '\tAllow Employees Requests: allocated by HR and users can request time off and allocations; '
             '\tSet by Time Off Officer: allocated by HR and cannot be bypassed; users can request time off;')
    validity_start = fields.Date("From",
                                 help='Adding validity to types of time off so that it cannot be selected outside this time period')
    validity_stop = fields.Date("To")
    valid = fields.Boolean(compute='_compute_valid', search='_search_valid', help='This indicates if it is still possible to use this type of leave')
    time_type = fields.Selection([('leave', 'Time Off'), ('other', 'Other')], default='leave', string="Kind of Leave",
                                 help="Whether this should be computed as a holiday or as work time (eg: formation)")
    request_unit = fields.Selection([
        ('day', 'Day'), ('half_day', 'Half Day'), ('hour', 'Hours')],
        default='day', string='Take Time Off in', required=True)
    unpaid = fields.Boolean('Is Unpaid', default=False)
    leave_notif_subtype_id = fields.Many2one('mail.message.subtype', string='Time Off Notification Subtype', default=lambda self: self.env.ref('hr_holidays.mt_leave', raise_if_not_found=False))
    allocation_notif_subtype_id = fields.Many2one('mail.message.subtype', string='Allocation Notification Subtype', default=lambda self: self.env.ref('hr_holidays.mt_leave_allocation', raise_if_not_found=False))

    @api.constrains('validity_start', 'validity_stop')
    def _check_validity_dates(self):
        for leave_type in self:
            if leave_type.validity_start and leave_type.validity_stop and \
               leave_type.validity_start > leave_type.validity_stop:
                raise ValidationError(_("End of validity period should be greater than start of validity period"))

    @api.depends('validity_start', 'validity_stop')
    def _compute_valid(self):
        dt = self._context.get('default_date_from') or fields.Date.context_today(self)

        for holiday_type in self:
            if holiday_type.validity_start and holiday_type.validity_stop:
                holiday_type.valid = ((dt < holiday_type.validity_stop) and (dt > holiday_type.validity_start))
            elif holiday_type.validity_start and (dt > holiday_type.validity_start):
                holiday_type.valid = False
            else:
                holiday_type.valid = True

    def _search_valid(self, operator, value):
        dt = self._context.get('default_date_from', False)

        if not dt:
            return []
        signs = ['>=', '<='] if operator == '=' else ['<=', '>=']

        return ['|', ('validity_stop', operator, False), '&',
                ('validity_stop', signs[0] if value else signs[1], dt),
                ('validity_start', signs[1] if value else signs[0], dt)]

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
            if leave_type.allocation_type != 'no':
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

    # YTI TODO: Remove me in master
    def get_days(self, employee_id):
        return self.get_employees_days([employee_id])[employee_id]

    def get_employees_days(self, employee_ids):
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

        requests = self.env['hr.leave'].search([
            ('employee_id', 'in', employee_ids),
            ('state', 'in', ['confirm', 'validate1', 'validate']),
            ('holiday_status_id', 'in', self.ids)
        ])

        allocations = self.env['hr.leave.allocation'].search([
            ('employee_id', 'in', employee_ids),
            ('state', 'in', ['confirm', 'validate1', 'validate']),
            ('holiday_status_id', 'in', self.ids)
        ])

        for request in requests:
            status_dict = result[request.employee_id.id][request.holiday_status_id.id]
            status_dict['virtual_remaining_leaves'] -= (request.number_of_hours_display
                                                    if request.leave_type_request_unit == 'hour'
                                                    else request.number_of_days)
            status_dict['virtual_leaves_taken'] += (request.number_of_hours_display
                                                if request.leave_type_request_unit == 'hour'
                                                else request.number_of_days)
            if request.state == 'validate':
                status_dict['leaves_taken'] += (request.number_of_hours_display
                                            if request.leave_type_request_unit == 'hour'
                                            else request.number_of_days)
                status_dict['remaining_leaves'] -= (request.number_of_hours_display
                                                if request.leave_type_request_unit == 'hour'
                                                else request.number_of_days)

        for allocation in allocations.sudo():
            status_dict = result[allocation.employee_id.id][allocation.holiday_status_id.id]
            if allocation.state == 'validate':
                # note: add only validated allocation even for the virtual
                # count; otherwise pending then refused allocation allow
                # the employee to create more leaves than possible
                status_dict['virtual_remaining_leaves'] += (allocation.number_of_hours_display
                                                          if allocation.type_request_unit == 'hour'
                                                          else allocation.number_of_days)
                status_dict['max_leaves'] += (allocation.number_of_hours_display
                                            if allocation.type_request_unit == 'hour'
                                            else allocation.number_of_days)
                status_dict['remaining_leaves'] += (allocation.number_of_hours_display
                                                  if allocation.type_request_unit == 'hour'
                                                  else allocation.number_of_days)
        return result

    @api.model
    def get_days_all_request(self):
        leave_types = sorted(self.search([]).filtered(lambda x: x.virtual_remaining_leaves or x.max_leaves), key=self._model_sorting_key, reverse=True)
        return [(lt.name, {
                    'remaining_leaves': ('%.2f' % lt.remaining_leaves).rstrip('0').rstrip('.'),
                    'virtual_remaining_leaves': ('%.2f' % lt.virtual_remaining_leaves).rstrip('0').rstrip('.'),
                    'max_leaves': ('%.2f' % lt.max_leaves).rstrip('0').rstrip('.'),
                    'leaves_taken': ('%.2f' % lt.leaves_taken).rstrip('0').rstrip('.'),
                    'virtual_leaves_taken': ('%.2f' % lt.virtual_leaves_taken).rstrip('0').rstrip('.'),
                    'request_unit': lt.request_unit,
                }, lt.allocation_type, lt.validity_stop)
            for lt in leave_types]

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
            data_days = self.get_employees_days([employee_id])[employee_id]

        for holiday_status in self:
            result = data_days.get(holiday_status.id, {})
            holiday_status.max_leaves = result.get('max_leaves', 0)
            holiday_status.leaves_taken = result.get('leaves_taken', 0)
            holiday_status.remaining_leaves = result.get('remaining_leaves', 0)
            holiday_status.virtual_remaining_leaves = result.get('virtual_remaining_leaves', 0)
            holiday_status.virtual_leaves_taken = result.get('virtual_leaves_taken', 0)

    def _compute_group_days_allocation(self):
        domain = [
            ('holiday_status_id', 'in', self.ids),
            ('holiday_type', '!=', 'employee'),
            ('state', '=', 'validate'),
        ]
        domain2 = [
            '|',
            ('date_from', '>=', fields.Datetime.to_string(datetime.datetime.now().replace(month=1, day=1, hour=0, minute=0, second=0, microsecond=0))),
            ('date_from', '=', False),
        ]
        grouped_res = self.env['hr.leave.allocation'].read_group(
            expression.AND([domain, domain2]),
            ['holiday_status_id', 'number_of_days'],
            ['holiday_status_id'],
        )
        grouped_dict = dict((data['holiday_status_id'][0], data['number_of_days']) for data in grouped_res)
        for allocation in self:
            allocation.group_days_allocation = grouped_dict.get(allocation.id, 0)

    def _compute_group_days_leave(self):
        grouped_res = self.env['hr.leave'].read_group(
            [('holiday_status_id', 'in', self.ids), ('holiday_type', '=', 'employee'), ('state', '=', 'validate'),
             ('date_from', '>=', fields.Datetime.to_string(datetime.datetime.now().replace(month=1, day=1, hour=0, minute=0, second=0, microsecond=0)))],
            ['holiday_status_id'],
            ['holiday_status_id'],
        )
        grouped_dict = dict((data['holiday_status_id'][0], data['holiday_status_id_count']) for data in grouped_res)
        for allocation in self:
            allocation.group_days_leave = grouped_dict.get(allocation.id, 0)

    def name_get(self):
        if not self._context.get('employee_id'):
            # leave counts is based on employee_id, would be inaccurate if not based on correct employee
            return super(HolidaysType, self).name_get()
        res = []
        for record in self:
            name = record.name
            if record.allocation_type != 'no':
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
        domain = [
            ('holiday_status_id', 'in', self.ids),
            ('holiday_type', '!=', 'employee'),
        ]
        domain2 = [
            '|',
            ('date_from', '>=', fields.Datetime.to_string(datetime.datetime.now().replace(month=1, day=1, hour=0, minute=0, second=0, microsecond=0))),
            ('date_from', '=', False),
        ]
        action['domain'] = expression.AND([domain, domain2])
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

```

## File: models\mail_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Channel(models.Model):
    _inherit = 'mail.channel'

    def partner_info(self, all_partners, direct_partners):
        partner_infos = super(Channel, self).partner_info(all_partners, direct_partners)
        # only search for leave out_of_office_date_end if im_status is on leave
        partners_on_leave = [partner_id for partner_id in direct_partners.ids if 'leave' in partner_infos[partner_id]['im_status']]
        if partners_on_leave:
            now = fields.Datetime.now()
            self.env.cr.execute('''SELECT res_users.partner_id as partner_id, hr_leave.date_to as date_to
                                FROM res_users
                                JOIN hr_leave ON hr_leave.user_id = res_users.id
                                AND hr_leave.state not in ('cancel', 'refuse')
                                AND res_users.active = 't'
                                AND hr_leave.date_from <= %s
                                AND hr_leave.date_to >= %s
                                AND res_users.partner_id in %s''', (now, now, tuple(partners_on_leave)))
            out_of_office_infos = dict(((res['partner_id'], res) for res in self.env.cr.dictfetchall()))
            for partner_id, out_of_office_info in out_of_office_infos.items():
                partner_infos[partner_id]['out_of_office_date_end'] = out_of_office_info['date_to']

        # fill empty values for the consistency of the result
        for partner_info in partner_infos.values():
            partner_info.setdefault('out_of_office_date_end', False)

        return partner_infos

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

    @api.model
    def create(self, vals):
        result = super(MailMessageSubtype, self).create(vals)
        if result.res_model in ['hr.leave', 'hr.leave.allocation']:
            result._update_department_subtype()
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


class ResPartner(models.Model):
    _inherit = 'res.partner'

    def _compute_im_status(self):
        super(ResPartner, self)._compute_im_status()
        absent_now = self._get_on_leave_ids()
        for partner in self:
            if partner.id in absent_now:
                if partner.im_status == 'online':
                    partner.im_status = 'leave_online'
                else:
                    partner.im_status = 'leave_offline'

    @api.model
    def _get_on_leave_ids(self):
        return self.env['res.users']._get_on_leave_ids(partner=True)

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

    def __init__(self, pool, cr):
        """ Override of __init__ to add access rights.
            Access rights are disabled by default, but allowed
            on some specific fields defined in self.SELF_{READ/WRITE}ABLE_FIELDS.
        """

        readable_fields = [
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
        init_res = super(User, self).__init__(pool, cr)
        # duplicate list to avoid modifying the original reference
        pool[self._name].SELF_READABLE_FIELDS = pool[self._name].SELF_READABLE_FIELDS + readable_fields
        return init_res

    def _compute_im_status(self):
        super(User, self)._compute_im_status()
        on_leave_user_ids = self._get_on_leave_ids()
        for user in self:
            if user.id in on_leave_user_ids:
                if user.im_status == 'online':
                    user.im_status = 'leave_online'
                else:
                    user.im_status = 'leave_offline'

    @api.model
    def _get_on_leave_ids(self, partner=False):
        now = fields.Datetime.now()
        field = 'partner_id' if partner else 'id'
        self.env.cr.execute('''SELECT res_users.%s FROM res_users
                            JOIN hr_leave ON hr_leave.user_id = res_users.id
                            AND state in ('validate')
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
from . import mail_channel
from . import mail_message_subtype
from . import res_partner
from . import res_users

```

## File: report\holidays_summary_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import calendar

from datetime import timedelta
from dateutil.relativedelta import relativedelta
from odoo import api, fields, models, _
from odoo.exceptions import UserError


class HrHolidaySummaryReport(models.AbstractModel):
    _name = 'report.hr_holidays.report_holidayssummary'
    _description = 'Holidays Summary Report'

    def _get_header_info(self, start_date, holiday_type):
        st_date = fields.Date.from_string(start_date)
        return {
            'start_date': fields.Date.to_string(st_date),
            'end_date': fields.Date.to_string(st_date + relativedelta(days=59)),
            'holiday_type': 'Confirmed and Approved' if holiday_type == 'both' else holiday_type
        }

    def _date_is_day_off(self, date):
        return date.weekday() in (calendar.SATURDAY, calendar.SUNDAY,)

    def _get_day(self, start_date):
        res = []
        start_date = fields.Date.from_string(start_date)
        for x in range(0, 60):
            color = '#ababab' if self._date_is_day_off(start_date) else ''
            res.append({'day_str': start_date.strftime('%a'), 'day': start_date.day , 'color': color})
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
            res.append({'month_name': start_date.strftime('%B'), 'days': month_days})
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

## File: report\hr_leave_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools, exceptions, _
from odoo.osv import expression


class LeaveReport(models.Model):
    _name = "hr.leave.report"
    _description = 'Time Off Summary / Report'
    _auto = False
    _order = "date_from DESC, employee_id"

    employee_id = fields.Many2one('hr.employee', string="Employee", readonly=True)
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
    payslip_status = fields.Boolean('Reported in last payslips', readonly=True)

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
                leaves.date_to as date_to, leaves.payslip_status as payslip_status
                from (select
                    allocation.employee_id as employee_id,
                    allocation.private_name as name,
                    allocation.number_of_days as number_of_days,
                    allocation.category_id as category_id,
                    allocation.department_id as department_id,
                    allocation.holiday_status_id as holiday_status_id,
                    allocation.state as state,
                    allocation.holiday_type,
                    null as date_from,
                    null as date_to,
                    FALSE as payslip_status,
                    'allocation' as leave_type
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
                    request.payslip_status as payslip_status,
                    'request' as leave_type
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
            'search_view_id': self.env.ref('hr_holidays.view_hr_holidays_filter_report').id,
            'domain': domain,
            'context': {
                'search_default_group_type': True,
                'search_default_year': True,
                'search_default_validated': True,
            }
        }

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        if not self.user_has_groups('hr_holidays.group_hr_holidays_user') and 'name' in groupby:
            raise exceptions.UserError(_('Such grouping is not allowed.'))
        return super(LeaveReport, self).read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)

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
                <field name="name"/>
                <filter domain="[('state','in',('confirm','validate1'))]" string="To Approve" name="approve"/>
                <filter string="Approved Requests" domain="[('state', '=', 'validate')]" name="validated"/>
                <separator/>
                <filter name="active_types" string="Active Types" domain="[('holiday_status_id.active', '=', True)]" help="Filters only on requests that belong to an time off type that is 'active' (active field is True)"/>
                <separator/>
                <filter string="My Department Time Off" name="department" domain="[('department_id.manager_id.user_id', '=', uid)]" help="My Department Time Off"/>
                <filter name="my_team_leaves" string="My Team Time Off" domain="[('employee_id.parent_id.user_id', '=', uid)]" groups="hr_holidays.group_hr_holidays_manager" help="Time Off of Your Team Member"/>
                <separator/>
                <filter name="year" string="Current Year"
                    domain="[('holiday_status_id.active', '=', True)]" help="Active Time Off"/>
                <separator/>
                <filter string="My Requests" name="my_leaves" domain="[('employee_id.user_id', '=', uid)]"/>
                <separator/>
                <field name="employee_id"/>
                <field name="department_id" operator="child_of"/>
                <field name="holiday_status_id"/>
                <group expand="0" string="Group By">
                    <filter name="group_employee" string="Employee" context="{'group_by':'employee_id'}"/>
                    <filter name="group_type" string="Type" context="{'group_by':'holiday_status_id'}"/>
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

    <!-- TODO: See if we need to keep this -->
    <record id="hr_leave_report_kanban" model="ir.ui.view">
        <field name="name">report.hr.holidays.report.leave_all.kanban</field>
        <field name="model">hr.leave.report</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile" create="0">
                <field name="employee_id"/>
                <field name="date_from"/>
                <field name="date_to"/>
                <field name="name"/>
                <field name="number_of_days"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div>
                                <span>
                                    <img t-att-src="kanban_image('hr.employee', 'image_128', record.employee_id.raw_value)" t-att-title="record.employee_id.value" t-att-alt="record.employee_id.value" class="oe_kanban_avatar o_image_40_cover float-left mr4"/>
                                </span>
                                <span>
                                    <div>
                                        <strong class="o_kanban_record_title"><t t-esc="record.employee_id.value"/></strong>
                                        <span class="float-right">
                                            <field name="state" widget="label_selection" options="{'classes': {'draft': 'default', 'validate': 'success','confirm': 'default', 'cancel': 'danger'}}"/>
                                        </span>
                                    </div>
                                    <div class="text-muted o_kanban_record_subtitle">
                                        <t t-esc="record.name.value"/>
                                    </div>
                                </span>
                            </div>
                            <hr class="mt4 mb8"/>
                            <div class="o_kanban_record_bottom mt8 mb4">
                                <div t-attf-class="oe_kanban_bottom_left #{record.date_from.value ? 'mt8 mb4': ''}">
                                    <table class="text-right" t-if="record.date_from.value">
                                        <tr>
                                            <td style="padding-bottom:4px"><small class="text-muted">from</small></td>
                                            <td style="padding:0 0 4px 4px"><t t-esc="record.date_from.value"/></td>
                                        </tr>
                                        <tr>
                                            <td><small class="text-muted">to</small></td>
                                            <td style="padding-left:4px"><t t-esc="record.date_to.value"/></td>
                                        </tr>
                                    </table>
                                </div>
                                <div t-attf-class="oe_kanban_bottom_right #{record.date_from.value ? 'mt8': ''}">
                                    <span class="badge badge-pill"><t t-esc="record.number_of_days.value"/> days</span>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
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
    company_id = fields.Many2one('res.company', readonly=True)
    state = fields.Selection([
        ('draft', 'To Submit'),
        ('cancel', 'Cancelled'),  # YTI This state seems to be unused. To remove
        ('confirm', 'To Approve'),
        ('refuse', 'Refused'),
        ('validate1', 'Second Approval'),
        ('validate', 'Approved')
    ], readonly=True)

    def init(self):
        tools.drop_view_if_exists(self._cr, 'hr_leave_report_calendar')
        self._cr.execute("""CREATE OR REPLACE VIEW hr_leave_report_calendar AS
        (SELECT 
            row_number() OVER() AS id,
            CONCAT(em.name, ': ', hl.duration_display) AS name,
            hl.date_from AS start_datetime,
            hl.date_to AS stop_datetime,
            hl.employee_id AS employee_id,
            hl.state AS state,
            em.company_id AS company_id,
            CASE
                WHEN hl.holiday_type = 'employee' THEN rr.tz
                ELSE %s
            END AS tz
        FROM hr_leave hl
            LEFT JOIN hr_employee em
                ON em.id = hl.employee_id
            LEFT JOIN resource_resource rr
                ON rr.id = em.resource_id
        WHERE 
            hl.state IN ('confirm', 'validate', 'validate1')
        ORDER BY id);
        """, [self.env.company.resource_calendar_id.tz or self.env.user.tz or 'UTC'])

    def _read(self, fields):
        res = super()._read(fields)
        if self.env.context.get('hide_employee_name') and 'employee_id' in self.env.context.get('group_by', []):
            name_field = self._fields['name']
            for record in self.with_user(SUPERUSER_ID):
                self.env.cache.set(record, name_field, record.name.split(':')[-1].strip())
        return res

    @api.model
    def get_unusual_days(self, date_from, date_to=None):
        # Checking the calendar directly allows to not grey out the leaves taken
        # by the employee
        return self.env['hr.leave'].get_unusual_days(date_from, date_to=date_to)

```

## File: report\hr_leave_report_calendar.xml

```xml
<?xml version='1.0' encoding='UTF-8' ?>
<odoo>
    <record id="action_hr_holidays_dashboard" model="ir.actions.act_window">
        <field name="name">All Time Off</field>
        <field name="res_model">hr.leave.report.calendar</field>
        <field name="view_mode">calendar</field>
        <!-- YTI : Ecrabouille explicitely those fields, as this is deployed on 
                   a stable release. -->
        <field name="view_id"/>
        <field name="search_view_id"/>
        <field name="domain">[('employee_id.active','=',True)]</field>
        <field name="context">{'hide_employee_name': 1}</field>
    </record>

    <record id="hr_leave_report_calendar_view" model="ir.ui.view">
        <field name="name">hr.leave.report.calendar.view</field>
        <field name="model">hr.leave.report.calendar</field>
        <field name="arch" type="xml">
            <calendar string="Time Off" date_start="start_datetime" date_stop="stop_datetime" mode="month" quick_add="False" color="employee_id" event_open_popup="True" js_class="time_off_calendar_all" show_unusual_days="True">
                <field name="name"/>
                <field name="employee_id" filters="1" invisible="1"/>
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
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import holidays_summary_report
from . import hr_leave_report
from . import hr_leave_report_calendar

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
        <field name="name">Responsible</field>
        <field name="category_id" ref="base.module_category_human_resources_time_off"/>
        <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <record id="group_hr_holidays_user" model="res.groups">
        <field name="name">All Approver</field>
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
        <field name="domain_force">[('employee_id.user_id', '=', user.id), ('state', '=', 'draft')]</field>
        <field name="perm_read" eval="False"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
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
        <field name="domain_force">[('holiday_status_id.company_id', 'in', company_ids + [False])]</field>
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
            ('holiday_status_id.allocation_type', '=', 'fixed_allocation'),
            ('holiday_type', '=', 'employee'),
            '|',
                '&amp;',
                    ('employee_id.user_id', '=', user.id),
                    ('state', 'not in', ['validate', 'validate1']),
                '&amp;',
                    ('validation_type', 'in', ['manager', 'both']),
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

## File: static\src\bugfix\bugfix.js

```javascript
/**
 * This file allows introducing new JS modules without contaminating other files.
 * This is useful when bug fixing requires adding such JS modules in stable
 * versions of Odoo. Any module that is defined in this file should be isolated
 * in its own file in master.
 */
odoo.define('hr_holidays/static/src/bugfix/bugfix.js', function (require) {
'use strict';

});

// FIXME move me in hr_holidays/static/src/models/partner/partner.js
odoo.define('hr_holidays/static/src/models/partner/partner.js', function (require) {
'use strict';

const {
    registerClassPatchModel,
    registerFieldPatchModel,
    registerInstancePatchModel,
} = require('mail/static/src/model/model_core.js');
const { attr, one2one } = require('mail/static/src/model/model_field.js');
const { clear } = require('mail/static/src/model/model_field_command.js');

const { str_to_datetime } = require('web.time');

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
        if (!this.env.messaging.locale.language) {
            return clear();
        }
        const currentDate = new Date();
        const date = str_to_datetime(this.outOfOfficeDateEnd);
        const options = { day: 'numeric', month: 'short' };
        if (currentDate.getFullYear() !== date.getFullYear()) {
            options.year = 'numeric';
        }
        let localeCode = this.env.messaging.locale.language.replace(/_/g,'-');
        if (localeCode == "sr@latin") {
            localeCode = "sr-Latn-RS";
        }
        const formattedDate = date.toLocaleDateString(localeCode, options);
        return _.str.sprintf(this.env._t("Out of office until %s"), formattedDate);
    },

});

registerFieldPatchModel('mail.partner', 'hr/static/src/models/partner/partner.js', {
    /**
     * Serves as compute dependency.
     */
    locale: one2one('mail.locale', {
        related: 'messaging.locale',
    }),
    /**
     * Date of end of the out of office period of the partner as string.
     * String is expected to use Odoo's datetime string format
     * (examples: '2011-12-01 15:12:35.832' or '2011-12-01 15:12:35').
     */
    outOfOfficeDateEnd: attr(),
    /**
     * Text shown when partner is out of office.
     */
    outOfOfficeText: attr({
        compute: '_computeOutOfOfficeText',
        dependencies: [
            'locale',
            'outOfOfficeDateEnd',
        ],
    }),
});

});

```

## File: static\src\bugfix\bugfix.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

<!--
    This file allows introducing new static templates without contaminating other files.
    This is useful when bug fixing requires adding new components for instance in stable
    versions of Odoo. Any template that is defined in this file should be isolated
    in its own file in master.
-->

</templates>

```

## File: static\src\bugfix\bugfix_tests.js

```javascript
odoo.define('hr_holidays/static/src/bugfix/bugfix_tests.js', function (require) {
'use strict';

/**
 * This file allows introducing new QUnit test modules without contaminating
 * other test files. This is useful when bug fixing requires adding new
 * components for instance in stable versions of Odoo. Any test that is defined
 * in this file should be isolated in its own file in master.
 */
QUnit.module('hr_holidays', {}, function () {
QUnit.module('bugfix', {}, function () {
QUnit.module('bugfix_tests.js', {

});
});
});

});

// FIXME move me in hr_holidays/static/src/components/thread_view/thread_view_tests.js
odoo.define('hr_holidays/static/src/components/thread_view/thread_view_tests.js', function (require) {
'use strict';

const components = {
    ThreadView: require('mail/static/src/components/thread_view/thread_view.js'),
};
const {
    afterEach,
    beforeEach,
    createRootComponent,
    start,
} = require('mail/static/src/utils/test_utils.js');

QUnit.module('hr_holidays', {}, function () {
QUnit.module('components', {}, function () {
QUnit.module('thread_view', {}, function () {
QUnit.module('thread_view_tests.js', {
    beforeEach() {
        beforeEach(this);

        /**
         * @param {mail.thread_view} threadView
         * @param {Object} [otherProps={}]
         */
        this.createThreadViewComponent = async (threadView, otherProps = {}) => {
            const target = this.widget.el;
            const props = Object.assign({ threadViewLocalId: threadView.localId }, otherProps);
            await createRootComponent(this, components.ThreadView, { props, target });
        };

        this.start = async params => {
            const { afterEvent, env, widget } = await start(Object.assign({}, params, {
                data: this.data,
            }));
            this.afterEvent = afterEvent;
            this.env = env;
            this.widget = widget;
        };
    },
    afterEach() {
        afterEach(this);
    },
});

QUnit.test('out of office message on direct chat with out of office partner', async function (assert) {
    assert.expect(2);

    // Returning date of the out of office partner, simulates he'll be back in a month
    const returningDate = moment.utc().add(1, 'month');
    // Needed partner & user to allow simulation of message reception
    this.data['res.partner'].records.push({
        id: 11,
        name: "Foreigner partner",
        out_of_office_date_end: returningDate.format("YYYY-MM-DD HH:mm:ss"),
    });
    this.data['mail.channel'].records = [{
        channel_type: 'chat',
        id: 20,
        members: [this.data.currentPartnerId, 11],
    }];
    await this.start();
    const thread = this.env.models['mail.thread'].findFromIdentifyingData({
        id: 20,
        model: 'mail.channel'
    });
    const threadViewer = this.env.models['mail.thread_viewer'].create({
        hasThreadView: true,
        thread: [['link', thread]],
    });
    await this.createThreadViewComponent(threadViewer.threadView, { hasComposer: true });
    assert.containsOnce(
        document.body,
        '.o_ThreadView_outOfOffice',
        "should have an out of office alert on thread view"
    );
    const formattedDate = returningDate.toDate().toLocaleDateString(
        this.env.messaging.locale.language.replace(/_/g,'-'),
        { day: 'numeric', month: 'short' }
    );
    assert.ok(
        document.querySelector('.o_ThreadView_outOfOffice').textContent.includes(formattedDate),
        "out of office message should mention the returning date"
    );
});

});
});
});

});

```

## File: static\src\components\partner_im_status_icon\partner_im_status_icon.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.PartnerImStatusIcon" t-inherit-mode="extension">
        <xpath expr="//*[@name='rootCondition']" position="inside">
            <t t-if="partner.im_status === 'leave_online'">
                <i class="o_PartnerImStatusIcon_icon o-leave-online fa fa-plane fa-stack-1x" title="Online" role="img" aria-label="User is online"/>
            </t>
            <t t-if="partner.im_status === 'leave_offline'">
                <i class="o_PartnerImStatusIcon_icon o-leave-offline fa fa-plane fa-stack-1x" title="Out of office" role="img" aria-label="User is out of office"/>
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
                <div class="o_ThreadIcon_leaveOnline fa fa-plane" title="Online"/>
            </t>
            <t t-elif="thread.correspondent.im_status === 'leave_offline'">
                <div class="o_ThreadIcon_leaveOffline fa fa-plane" title="Out of office"/>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\components\thread_view\thread_view.js

```javascript
odoo.define('hr_holidays/static/src/components/thread_view/thread_view.js', function (require) {
'use strict';

const components = {
    ThreadView: require('mail/static/src/components/thread_view/thread_view.js'),
};

const { patch } = require('web.utils');

patch(components.ThreadView, 'hr_holidays/static/src/components/thread_view/thread_view.js', {
    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _useStoreSelector(props) {
        const res = this._super(...arguments);
        const thread = res.thread;
        const correspondent = thread && thread.correspondent;
        return Object.assign({}, res, {
            correspondentOutOfOfficeText: correspondent && correspondent.outOfOfficeText,
        });
    },
});

});

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

## File: static\src\js\time_off_calendar.js

```javascript
odoo.define('hr_holidays.dashboard.view_custo', function(require) {
    'use strict';

    var core = require('web.core');
    var CalendarModel = require('web.CalendarModel');
    var CalendarPopover = require('web.CalendarPopover');
    var CalendarController = require("web.CalendarController");
    var CalendarRenderer = require("web.CalendarRenderer");
    var CalendarView = require("web.CalendarView");
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
                request: _t('New Allocation'),
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

        /**
         * Action: create a new time off request
         *
         * @private
         */
        _onNewTimeOff: function () {
            var self = this;

            this.do_action('hr_holidays.hr_leave_action_my_request', {
                on_close: function () {
                    self.reload();
                }
            });
        },

        /**
         * Action: create a new allocation request
         *
         * @private
         */
        _onNewAllocation: function () {
            var self = this;
            this.do_action({
                type: 'ir.actions.act_window',
                res_model: 'hr.leave.allocation',
                name: 'New Allocation Request',
                views: [[false,'form']],
                context: {'form_view_ref': 'hr_holidays.hr_leave_allocation_view_form_dashboard'},
                target: 'new',
            }, {
                on_close: function () {
                    self.reload();
                }
            });
        },
    });

    var TimeOffPopoverRenderer = CalendarRenderer.extend({
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
    });

    var TimeOffCalendarRenderer = TimeOffPopoverRenderer.extend({
        _render: function () {
            var self = this;
            return this._super.apply(this, arguments).then(function () {
                return self._rpc({
                    model: 'hr.leave.type',
                    method: 'get_days_all_request',
                    context: self.context,
                });
            }).then(function (result) {
                self.$el.parent().find('.o_calendar_mini').hide();
                self.$el.parent().find('.o_timeoff_container').remove();
                var elem = QWeb.render('hr_holidays.dashboard_calendar_header', {
                    timeoffs: result,
                });
                self.$el.before(elem);
            });
        },
    });
    var TimeOffCalendarView = CalendarView.extend({
        config: _.extend({}, CalendarView.prototype.config, {
            Controller: TimeOffCalendarController,
            Renderer: TimeOffCalendarRenderer,
        }),
    });

    /**
     * Calendar shown in the "Everyone" menu
     */
    var TimeOffCalendarAllView = CalendarView.extend({
        config: _.extend({}, CalendarView.prototype.config, {
            Renderer: TimeOffPopoverRenderer,
        }),
    });

    viewRegistry.add('time_off_calendar', TimeOffCalendarView);
    viewRegistry.add('time_off_calendar_all', TimeOffCalendarAllView);
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
            <div t-foreach="timeoffs" t-as="timeoff" t-attf-class="o_timeoff_card flex-grow-1 d-flex flex-column {{ timeoff_last ? 'o_timeoff_card_last' : '' }}">
                <t t-set="need_allocation" t-value="timeoff[2] !== 'no'"/>
                <t t-set="cl" t-value="'text-muted'"/>

                <t t-if="need_allocation &amp;&amp; timeoff[1]['virtual_remaining_leaves'] &gt; 0">
                    <t t-set="cl" t-value="'o_timeoff_green'"/>
                </t>

                <div class="mt-2">
                    <t t-if="need_allocation">
                        <span t-esc="timeoff[1]['virtual_remaining_leaves']" class="o_timeoff_big o_timeoff_purple"/> / <span t-esc="timeoff[1]['max_leaves']"/> <t t-if="timeoff[1]['request_unit'] == 'hour'">Hours</t><t t-else="">Days</t>
                    </t>
                    <t t-else="">
                        <span t-esc="timeoff[1]['virtual_leaves_taken']" class="o_timeoff_big o_timeoff_purple"/> <t t-if="timeoff[1]['request_unit'] == 'hour'">Hours</t><t t-else="">Days</t>
                    </t>
                </div>

                <b><span t-esc="timeoff[0]" class="o_timeoff_name"/></b>

                <span class="mb-4" t-if="need_allocation">
                    <span t-attf-class="mr-1 font-weight-bold {{ cl }}" t-esc="timeoff[1]['virtual_leaves_taken']"/><span>taken</span>
                    <t t-if="timeoff[3]"> (Expire on <span t-esc="moment(timeoff[3]).format('L')"/>)</t>
                </span>
            </div>
        </div>
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
</templates>

```

## File: views\hr_holidays_views.xml

```xml
<?xml version='1.0' encoding='UTF-8' ?>
<odoo>

    <menuitem
        name="Time Off"
        id="menu_hr_holidays_root"
        sequence="95"
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
        name="My Allocation Requests"
        parent="menu_hr_holidays_my_leaves"
        action="hr_leave_allocation_action_my"
        sequence="3"/>

    <menuitem
        id="menu_hr_holidays_dashboard"
        name="Everyone"
        parent="menu_hr_holidays_root"
        sequence="2"
        action="action_hr_holidays_dashboard"/>

    <menuitem
        id="menu_hr_holidays_approvals"
        name="Managers"
        parent="menu_hr_holidays_root"
        sequence="3"/>

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
        groups="hr_holidays.group_hr_holidays_manager"
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
        sequence="5"/>

    <menuitem
        id="hr_holidays_status_menu_configuration"
        action="open_view_holiday_status"
        name="Time Off Types"
        parent="menu_hr_holidays_configuration"
        groups="hr_holidays.group_hr_holidays_user"
        sequence="1"/>

    <menuitem id="hr_holidays_menu_config_activity_type"
        action="mail_activity_type_action_config_hr_holidays"
        parent="menu_hr_holidays_configuration"
        groups="base.group_no_one"/>

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
                <filter domain="[('state','in',('confirm', 'validate1'))]" string="To Approve" name="approve"/>
                <filter domain="[('state', '=', 'validate1')]" string="Need Second Approval" name="second_approval"/>
                <filter string="Approved Allocations" domain="[('state', '=', 'validate')]" name="validated"/>
                <filter string="Accrual Allocations" domain="[('allocation_type', '=', 'accrual')]" name="accrual_allocation"/>
                <separator/>
                <filter name="active_types" string="Active Types" domain="[('holiday_status_id.active', '=', True)]" help="Filters only on allocations that belong to an time off type that is 'active' (active field is True)"/>
                <separator/>
                <filter string="Unread Messages" name="message_needaction" domain="[('message_needaction','=',True)]"/>
                <separator/>
                <filter string="People I Manage" name="managed_people" domain="[('employee_id.leave_manager_id', '=', uid)]" help="Time off of people you are manager of"/>
                <filter string="My Team Time Off" name="my_team_leaves" domain="[('employee_id.parent_id.user_id', '=', uid)]" groups="hr_holidays.group_hr_holidays_manager" help="Time Off of Your Team Member"/>
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
                    <filter name="group_type" string="Type" context="{'group_by':'holiday_status_id'}"/>
                    <filter name="group_state" string="Status" context="{'group_by': 'state'}"/>
                </group>
                <searchpanel>
                    <field name="state" string="Status"/>
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
                            attrs="{'invisible': [('holiday_type', '!=', 'employee')]}"
                            name="%(hr_leave_action_action_approve_department)d"
                            context="{'search_default_employee_id': [employee_id], 'search_default_holiday_status_id': [holiday_status_id], 'default_holiday_status_id': holiday_status_id}"
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
                    <div class="oe_title">
                        <h2><field name="name" placeholder="e.g. Public Holiday Allocation" attrs="{'readonly': [('state', 'not in', ('draft', 'confirm'))]}" required="1"/></h2>
                    </div>
                    <group>
                        <group>
                            <field name="type_request_unit" invisible="1"/>
                            <field name="holiday_status_id" context="{'employee_id':employee_id}"/>

                            <field name="allocation_type" invisible="1" widget="radio"/>

                            <field name="number_of_days_display" invisible="1"/>
                            <field name="number_of_days" invisible="1"/>
                            <div class="o_td_label">
                                <label for="number_of_days" string="Duration" attrs="{'invisible': [('allocation_type', '=', 'accrual')]}"/>
                                <label for="number_of_days" string="Extra days" attrs="{'invisible': [('allocation_type', '!=', 'accrual')]}"/>
                            </div>
                            <div>
                                <field name="number_of_days_display" class="oe_inline" nolabel="1"
                                    attrs="{'readonly': ['|', ('type_request_unit', '=', 'hour'), ('state', 'not in', ('draft', 'confirm'))], 'invisible': [('type_request_unit', '=', 'hour')]}"/>
                                <field name="number_of_hours_display" class="oe_inline" nolabel="1"
                                    attrs="{'readonly': ['|', ('type_request_unit', '!=', 'hour'), ('state', 'not in', ('draft', 'confirm'))], 'invisible': [('type_request_unit', '!=', 'hour')]}"/>
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
                <button string="Approve" name="action_approve" states="confirm" type="object" class="oe_highlight"
                    attrs="{'invisible': ['|', ('can_approve', '=', False), ('state', '!=', 'confirm')]}"/>
                <button string="Validate" name="action_validate" states="validate1" type="object" class="oe_highlight"/>
                <button string="Refuse" name="action_refuse" type="object"
                    attrs="{'invisible': ['|', ('can_approve', '=', False), ('state', 'not in', ('confirm','validate1','validate'))]}"/>
                <button string="Mark as Draft" name="action_draft" type="object"
                        attrs="{'invisible': ['|', ('can_reset', '=', False), ('state', 'not in', ['confirm', 'refuse'])]}"/>
            </xpath>
            <xpath expr="//field[@name='employee_id']" position="before">
                <field name="holiday_type" string="Mode" groups="hr_holidays.group_hr_holidays_user" context="{'employee_id':employee_id}" />
            </xpath>
            <xpath expr="//field[@name='employee_id']" position="replace">
                <field name="employee_id" groups="hr_holidays.group_hr_holidays_user"
                    attrs="{'required': [('holiday_type', '=', 'employee')], 'invisible': [('holiday_type', '!=', 'employee')]}"/>
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
            <xpath expr="//field[@name='allocation_type']" position="after">
                <label for="date_from" attrs="{'invisible': [('allocation_type', '=', 'regular')]}"/>
                <div class="o_row" attrs="{'invisible': [('allocation_type', '=', 'regular')]}">
                    <field name="date_from" class="mr-2" attrs="{'required': [('allocation_type', '=', 'accrual')]}" />
                    Run until <field name="date_to" help="If no value set, runs indefinitely" class="ml-2"/>
                </div>
                <label for="date_from" invisible="1"/>
                <div attrs="{'invisible': [('allocation_type', '=', 'regular')]}">
                    <div class="o_row">
                        <span>Add</span>
                        <field name="number_per_interval" class="ml8"
                            attrs="{'required': [('allocation_type', '=', 'accrual')]}"/>
                        <field name="unit_per_interval"
                            attrs="{'required': [('allocation_type', '=', 'accrual')]}"/>
                        <span class="ml8">of time off every</span>
                        <field name="interval_number" class="ml8"
                            attrs="{'required': [('allocation_type', '=', 'accrual')]}"/>
                        <field name="interval_unit"
                            attrs="{'required': [('allocation_type', '=', 'accrual')]}"/>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

    <record id="hr_leave_allocation_view_tree" model="ir.ui.view">
        <field name="name">hr.leave.allocation.view.tree</field>
        <field name="model">hr.leave.allocation</field>
        <field name="priority">16</field>
        <field name="arch" type="xml">
            <tree string="Allocation Requests" sample="1">
                <field name="employee_id"/>
                <field name="department_id" optional="hide"/>
                <field name="holiday_status_id" class="font-weight-bold"/>
                <field name="name"/>
                <field name="duration_display" string="Duration"/>
                <field name="message_needaction" invisible="1"/>
                <field name="state" widget="badge" decoration-info="state == 'draft'" decoration-warning="state in ('confirm','validate1')" decoration-success="state == 'validate'"/>
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
            <xpath expr="//filter[@name='managed_people']" position="attributes">
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
                                <div class="col-2">
                                    <img t-att-src="kanban_image('hr.employee', 'image_128', record.employee_id.raw_value)"
                                        t-att-title="record.employee_id.value"
                                        t-att-alt="record.employee_id.value"
                                        class="oe_kanban_avatar o_image_40_cover float-left mr4"/>
                                </div>
                                <div class="col-10">
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
                                </div>
                            </div>
                            <div class="row no-gutters" t-if="['validate', 'refuse'].includes(record.state.raw_value)">
                                <div class="col-2"/>
                                <div class="col-10">
                                    <span t-if="record.state.raw_value === 'validate'" class="fa fa-check text-muted" aria-label="validated"/>
                                    <span t-else="" class="fa fa-ban text-muted" aria-label="refused"/>
                                    <span class="text-muted"><t t-esc="record.state.value"/></span>
                                </div>
                            </div>
                            <div class="o_kanban_record_bottom" t-if="record.can_approve.raw_value" >
                                <div class="oe_kanban_bottom_left" t-if="record.can_approve.raw_value">
                                    <button t-if="record.state.raw_value === 'confirm'" name="action_approve" type="object" class="btn btn-primary btn-sm mt8">Approve</button>
                                    <button t-if="record.state.raw_value === 'validate1'" name="action_validate" type="object" class="btn btn-primary btn-sm mt8">Validate</button>
                                    <button t-if="['confirm', 'validate1'].includes(record.state.raw_value)" name="action_refuse" type="object" class="btn btn-secondary btn-sm mt8">Refuse</button>
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
                        <img t-att-src="activity_image('hr.employee', 'image_128', record.employee_id.raw_value)" t-att-title="record.employee_id.value" t-att-alt="record.employee_id.value"/>
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
        <field name="context">{'search_default_managed_people': 1}</field>
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
        <field name="code">action = records.action_approve()</field>
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
        </field>
    </record>

</odoo>

```

## File: views\hr_leave_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
  <template id="assets_backend" inherit_id="web.assets_backend">
    <xpath expr="script[last()]" position="after">
      <script type="text/javascript" src="/hr_holidays/static/src/js/time_off_calendar.js"></script>
      <script type="text/javascript" src="/hr_holidays/static/src/js/leave_stats_widget.js"></script>
      <script type="text/javascript" src="/hr_holidays/static/src/bugfix/bugfix.js"/>
      <script type="text/javascript" src="/hr_holidays/static/src/components/thread_view/thread_view.js"/>
    </xpath>
    <xpath expr="link[last()]" position="after">
      <link rel="stylesheet" type="text/scss" href="/hr_holidays/static/scss/hr_leave_mobile.scss"/>
      <link rel="stylesheet" type="text/scss" href="/hr_holidays/static/src/scss/time_off.scss"/>
      <link rel="stylesheet" type="text/scss" href="/hr_holidays/static/src/bugfix/bugfix.scss"/>
      <link rel="stylesheet" type="text/scss" href="/hr_holidays/static/src/components/partner_im_status_icon/partner_im_status_icon.scss"/>
      <link rel="stylesheet" type="text/scss" href="/hr_holidays/static/src/components/thread_icon/thread_icon.scss"/>
      <link rel="stylesheet" type="text/scss" href="/hr_holidays/static/src/components/thread_view/thread_view.scss"/>
    </xpath>
  </template>
  <template id="qunit_suite" name="hr_holidays_qunit_suite" inherit_id="web.qunit_suite_tests">
      <xpath expr="." position="inside">
          <script type="text/javascript" src="/hr_holidays/static/src/bugfix/bugfix_tests.js"></script>
          <script type="text/javascript" src="/hr_holidays/static/tests/helpers/mock_models.js"/>
          <script type="text/javascript" src="/hr_holidays/static/tests/helpers/mock_server.js"/>
          <script type="text/javascript" src="/hr_holidays/static/tests/test_leave_stats_widget.js"></script>
      </xpath>
  </template>
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
                <field name="code"/>
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
                        <button class="oe_stat_button" type="object" name="action_see_days_allocated" icon="fa-archive" attrs="{'invisible': [('allocation_type', '=', 'no')]}">
                            <div class="o_stat_info">
                                <field name="group_days_allocation"/>
                                <span class="o_stat_text">Group Days Allocated</span>
                            </div>
                        </button>
                        <button class="oe_stat_button" type="object" name="action_see_group_leaves" icon="fa-archive">
                            <div class="o_stat_info">
                                <field name="group_days_leave"/>
                                <span class="o_stat_text">Group Time Off</span>
                            </div>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title">
                        <h1><field name="name"/></h1>
                    </div>
                    <group>
                        <group name="description" string="Options">
                            <field name="active" invisible="1"/>
                            <field name="code" string="Payroll Code"/>
                            <field name="request_unit" widget="radio"/>
                        </group>
                        <group name="validity" string="Validity">
                            <field name="validity_start"/>
                            <field name="validity_stop"/>
                        </group>
                        <group name="allocation_validation" string="Allocation Requests">
                            <field name="allocation_type" widget="radio" force_save="1"/>
                            <field name="allocation_validation_type" string="Approval" widget="radio" attrs="{'invisible': [('allocation_type', '!=', 'fixed_allocation')]}"/>
                            <field name="responsible_id" domain="[('share', '=', False)]"
                                attrs="{
                                'invisible': [('leave_validation_type', 'in', ['no_validation', 'manager']), '|', ('allocation_type', '=', 'no'), ('allocation_validation_type', '=', 'manager')],
                                'required': ['|', ('leave_validation_type', 'in', ['hr', 'both']), '&amp;', ('allocation_type', 'in', ['fixed_allocation', 'fixed']), ('allocation_validation_type', 'in', ['hr', 'both'])]}"/>
                        </group>
                        <group name="leave_validation" string="Time Off Requests">
                            <field name="leave_validation_type" string="Approval" widget="radio"/>
                        </group>
                        <group name="notification" string="Notification" colspan="2" groups="base.group_no_one">
                            <group>
                                <field name="leave_notif_subtype_id"
                                       domain="[('res_model','=','hr.leave')]"
                                       context="{'default_name': name, 'default_res_model': 'hr.leave'}"/>
                                <field name="allocation_notif_subtype_id"
                                       domain="[('res_model','=','hr.leave.allocation')]"
                                       context="{'default_name': name, 'default_res_model': 'hr.leave.allocation'}"/>
                            </group>
                         </group>
                        <group name="calendar" string="Calendar" groups="base.group_no_one">
                            <field name="create_calendar_meeting"/>
                            <field name="color_name"/>
                            <field name="company_id" groups="base.group_multi_company"/>
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
                <field name="display_name"/>
                <field name="code"/>
                <field name="allocation_type"/>
                <field name="leave_validation_type"/>
                <field name="allocation_validation_type"/>
                <field name="validity_start"/>
                <field name="validity_stop"/>
                <field name="company_id" groups="base.group_multi_company"/>
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

    <record model="ir.actions.server" id="action_report_to_payslip">
        <field name="name">Report to Payslip</field>
        <field name="model_id" ref="model_hr_leave"/>
        <field name="binding_model_id" ref="model_hr_leave" />
        <field name="state">code</field>
        <field name="code">
            if records:
                records.write({'payslip_status': True})
        </field>
    </record>

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
            <graph string="Appraisal Analysis" stacked="True" sample="1">
                <field name="employee_id" type="row"/>
                <field name="holiday_status_id" type="row"/>
                <field name="date_from" type="col"/>
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
                <filter string="My Department Time Off" name="department" domain="['|', ('department_id.member_ids.user_id', '=', uid), ('employee_id.user_id', '=', uid)]" help="My Department Time Off"/>
                <filter string="People I Manage" name="managed_people" domain="[('employee_id.leave_manager_id', '=', uid)]" help="Time off of people you are manager of"/>
                <filter string="My Time Off" name="my_leaves" domain="[('employee_id.user_id', '=', uid)]"/>
                <separator/>
                <filter string="To Report in Payslip" name="gray" domain="[('payslip_status', '=', False)]" groups="hr_holidays.group_hr_holidays_user"/>
                <separator/>
                <filter name="filter_date_from" date="date_from"/>
                <separator/>
                <filter name="year" string="Active Time Off"
                    domain="[('holiday_status_id.active', '=', True)]" help="Active Time Off"/>
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
                    <separator/>
                    <filter name="group_date_from" string="Start Date" context="{'group_by':'date_from'}"/>
                </group>
                <searchpanel>
                    <field name="state" string="Status"/>
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
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click container">
                            <div class="row no-gutters">
                                <div class="col-2">
                                    <img t-att-src="kanban_image('hr.employee', 'image_128', record.employee_id.raw_value)"
                                        t-att-title="record.employee_id.value"
                                        t-att-alt="record.employee_id.value"
                                        class="oe_kanban_avatar o_image_40_cover float-left mr4"/>
                                </div>
                                <div class="col-10">
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
                                </div>
                            </div>
                            <div class="row no-gutters justify-content-end">
                                <div class="col-2"/>
                                <div class="col-10">
                                    <span class="text-muted">from </span>
                                    <field name="date_from" widget="date"/>
                                    <span class="text-muted">to </span>
                                    <field name="date_to" widget="date"/>
                                </div>
                            </div>
                            <div class="row no-gutters" t-if="['validate', 'refuse'].includes(record.state.raw_value)">
                                <div class="col-2"/>
                                <div class="col-10">
                                    <span t-if="record.state.raw_value === 'validate'" class="fa fa-check text-muted" aria-label="validated"/>
                                    <span t-else="" class="fa fa-ban text-muted" aria-label="refused"/>
                                    <span class="text-muted"><t t-esc="record.state.value"/></span>
                                </div>
                            </div>
                            <div class="o_kanban_record_bottom" t-if="record.can_approve.raw_value">
                                <div class="oe_kanban_bottom_left">
                                    <button t-if="record.state.raw_value === 'confirm'" name="action_approve" type="object" class="btn btn-primary btn-sm mt8">Approve</button>
                                    <button t-if="record.state.raw_value === 'validate1'" name="action_validate" type="object" class="btn btn-primary btn-sm mt8" groups="hr_holidays.group_hr_holidays_manager">Validate</button>
                                    <button t-if="['confirm', 'validate1'].includes(record.state.raw_value)" name="action_refuse" type="object" class="btn btn-secondary btn-sm mt8">Refuse</button>
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
                        <img t-att-src="activity_image('hr.employee', 'image_128', record.employee_id.raw_value)" t-att-title="record.employee_id.value" t-att-alt="record.employee_id.value" width="50" height="50"/>
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
            <header>
                <button string="Confirm" name="action_confirm" states="draft" type="object" class="oe_highlight"/>
                <button string="Approve" name="action_approve" type="object" class="oe_highlight" attrs="{'invisible': ['|', ('can_approve', '=', False), ('state', '!=', 'confirm')]}"/>
                <button string="Validate" name="action_validate" states="validate1" type="object" groups="hr_holidays.group_hr_holidays_manager" class="oe_highlight"/>
                <button string="Refuse" name="action_refuse" type="object" attrs="{'invisible': ['|', ('can_approve', '=', False), ('state', 'not in', ('confirm','validate1','validate'))]}"/>
                <button string="Mark as Draft" name="action_draft" type="object"
                        attrs="{'invisible': ['|', ('can_reset', '=', False), ('state', 'not in', ['confirm', 'refuse'])]}"/>
                <field name="state" widget="statusbar" statusbar_visible="confirm,validate"/>
            </header>
            <sheet>
                <div class="alert alert-info" role="alert" attrs="{'invisible': ['|', ('tz_mismatch', '=', False), ('holiday_type', '=', 'category')]}">
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
                <div class="oe_title" name="title">
                    <field name="display_name" invisible="1"/>
                    <h1>
                        <field name="employee_id" nolabel="1" readonly="1" force_save="1" invisible="1"/>
                    </h1>
                    <h2>
                        <field name="holiday_status_id" nolabel="1" domain="['&amp;', ('virtual_remaining_leaves', '&gt;', 0), '|', ('allocation_type', 'in', ['fixed_allocation', 'no']),'&amp;',('allocation_type', '=', 'fixed'), ('max_leaves', '>', '0')]" context="{'employee_id':employee_id, 'default_date_from':date_from}" options="{'no_create': True, 'no_open': True}" class="w-100"/>
                    </h2>
                </div>
                <group>
                    <group>

                        <label for="request_date_from" string="Dates"/>
                        <div>
                            <field name="date_from" invisible="1"/>
                            <field name="date_to" invisible="1"/>
                            <div class="o_row o_row_readonly o_hr_holidays_dates">
                                <span class="oe_inline"
                                    attrs="{'invisible': ['|', ('request_unit_half', '=', True), ('request_unit_hours', '=', True)]}">
                                    From
                                </span>
                                <field name="request_date_from" class="oe_inline" nolabel="1"
                                    attrs="{'readonly': [('state', 'not in', ('draft', 'confirm'))],
                                            'required': ['|', ('date_from', '=', False), ('date_to', '=', False)]
                                            }"/>
                                <span class="oe_inline"
                                    attrs="{'invisible': ['|', ('request_unit_half', '=', True), ('request_unit_hours', '=', True)]}">
                                    To
                                </span>
                                <field name="request_date_to" class="oe_inline"
                                    attrs="{
                                        'readonly': [('state', 'not in', ('draft', 'confirm'))],
                                        'invisible': ['|', ('request_unit_half', '=', True), ('request_unit_hours', '=', True)],
                                        'required': ['|', ('date_from', '=', False), ('date_to', '=', False)]
                                    }"/>
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
                                    'readonly': [('state', 'not in', ('draft', 'confirm'))],
                                    'invisible': [('leave_type_request_unit', '=', 'day')]
                                }"/>
                                <field name="request_unit_hours" attrs="{
                                    'readonly': [('state', 'not in', ('draft', 'confirm'))],
                                    'invisible': [('leave_type_request_unit', '!=', 'hour')]
                                }" class="ml-5"/>
                                <label for="request_unit_hours" attrs="{
                                    'readonly': [('state', 'not in', ('draft', 'confirm'))],
                                    'invisible': [('leave_type_request_unit', '!=', 'hour')]
                                }"/>
                                <field name="request_unit_custom" invisible="1" attrs="{
                                    'readonly': [('state', 'not in', ('draft', 'confirm'))],
                                }"/>
                                <label for="request_unit_custom" invisible="1" attrs="{
                                    'readonly': [('state', 'not in', ('draft', 'confirm'))],
                                }"/>
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
                        <div>
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
                    </group>
                    <group name="col_right">
                        <field name="department_id" groups="hr_holidays.group_hr_holidays_user" invisible="1"/>
                    </group>
                </group>
            </sheet>
            <div class="oe_chatter">
                <field name="message_follower_ids"/>
                <field name="activity_ids"/>
                <field name="message_ids"/>
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

    <record id="hr_leave_view_dashboard" model="ir.ui.view">
        <field name="name">hr.leave.view.dashboard</field>
        <field name="model">hr.leave</field>
        <field name="arch" type="xml">
            <calendar js_class="time_off_calendar" string="Time Off Request" form_view_id="%(hr_holidays.hr_leave_view_form_dashboard)d" event_open_popup="true" date_start="date_from" date_stop="date_to" mode="month" quick_add="False" show_unusual_days="True" color="holiday_status_id" hide_time="True">
                <field name="display_name"/>
                <field name="holiday_status_id" filters="1" invisible="1"/>
                <field name="state" invisible="1"/>
            </calendar>
        </field>
    </record>

    <record id="hr_leave_view_form_dashboard_new_time_off" model="ir.ui.view">
        <field name="name">hr.leave.view.form.dashboard.new.time.off</field>
        <field name="model">hr.leave</field>
        <field name="inherit_id" ref="hr_holidays.hr_leave_view_form_dashboard"/>
        <field name="mode">primary</field>
        <field name="priority">17</field>
        <field name="arch" type="xml">
            <xpath expr="//sheet" position="after">
                <footer>
                    <button string="Save" special="save" class="oe_highlight" />
                    <button string="Discard" special="cancel" />
                </footer>
            </xpath>
        </field>
    </record>

    <record id="hr_leave_view_form_manager" model="ir.ui.view">
        <field name="name">hr.leave.view.form.manager</field>
        <field name="model">hr.leave</field>
        <field name="inherit_id" ref="hr_leave_view_form"/>
        <field name="mode">primary</field>
        <field name="priority">16</field>
        <field name="arch" type="xml">
            <div name="title" position="replace">
                <field name="display_name" invisible="1"/>
                <div class="oe_title" name="title">
                    <h2>
                        <field name="holiday_status_id" nolabel="1"/>
                    </h2>
                </div>
            </div>
            <field name="name" position="replace"/>
            <field name="user_id" position="before">
                <field name="name"/>
            </field>
            <xpath expr="//group[@name='col_right']" position="replace">
                <group>
                    <field name="holiday_type" string="Mode"
                        groups="hr_holidays.group_hr_holidays_user"/>
                    <field name="employee_id" groups="hr_holidays.group_hr_holidays_user" attrs="{
                        'required': [('holiday_type', '=', 'employee')],
                        'invisible': [('holiday_type', '!=', 'employee')]
                        }"/>
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
                        'invisible': [('holiday_type', 'not in', ('employee', 'department'))]
                        }"/>
                    <field name="payslip_status" groups="hr_holidays.group_hr_holidays_user" widget="toggle_button"/>
                </group>
                <group groups="hr_holidays.group_hr_holidays_manager" string="Manager's Comment">
                    <field name="report_note" placeholder="e.g. Report to the next month..." nolabel="1"/>
                </group>
                <group>
                    <widget name="hr_leave_stats"/>
                </group>
            </xpath>
        </field>
    </record>

    <record id="hr_leave_view_calendar" model="ir.ui.view">
        <field name="name">hr.leave.view.calendar</field>
        <field name="model">hr.leave</field>
        <field name="arch" type="xml">
            <calendar js_class="time_off_calendar_all" string="Time Off Request" event_open_popup="true" date_start="date_from" date_stop="date_to" mode="month" quick_add="False" color="employee_id">
                <field name="display_name"/>
                <field name="holiday_status_id" filters="1" invisible="1"/>
                <field name="employee_id" filters="1" invisible="1"/>
            </calendar>
        </field>
    </record>

    <record id="hr_leave_view_tree" model="ir.ui.view">
        <field name="name">hr.holidays.view.tree</field>
        <field name="model">hr.leave</field>
        <field name="arch" type="xml">
            <tree string="Time Off Requests" sample="1">
                <field name="employee_id" widget="many2one_avatar_employee"/>
                <field name="department_id" optional="hidden"/>
                <field name="holiday_type" string="Mode" groups="base.group_no_one"/>
                <field name="holiday_status_id" class="font-weight-bold"/>
                <field name="name"/>
                <field name="date_from"/>
                <field name="date_to"/>
                <field name="duration_display" string="Duration"/>
                <field name="payslip_status" widget="toggle_button" options='{"active": "Reported in last payslips", "inactive": "To Report in Payslip"}' groups="hr_holidays.group_hr_holidays_user" nolabel="1"/>
                <field name="state" widget="badge" decoration-info="state == 'draft'" decoration-warning="state in ('confirm','validate1')" decoration-success="state == 'validate'"/>
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
            <xpath expr="//field[@name='payslip_status']" position="attributes">
                <attribute name="invisible">1</attribute>
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
            <xpath expr="//filter[@name='managed_people']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//filter[@name='my_leaves']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//filter[@name='gray']" position="attributes">
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
            <xpath expr="//filter[@name='gray']" position="attributes">
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
        <field name="context">{'short_name': 1}</field>
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
        <field name="name">Time Off Requests</field>
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
        <field name="name">My Time Off Requests</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">hr.leave</field>
        <field name="view_mode">tree,form,calendar,kanban,activity</field>
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
            'search_default_managed_people': 1,
            'hide_employee_name': 1}
        </field>
        <field name="domain">[('employee_id.company_id', 'in', allowed_company_ids)]</field>
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
        <field name="context">{'search_default_year': 1, 'search_default_group_employee': 1, 'search_default_group_type': 1}</field>
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
                            <a name="%(hr_leave_action_action_approve_department)d" type="action">
                                Time Off Requests
                            </a>
                        </div>
                        <div class="col-3 text-right">
                            <field name="leave_to_approve_count"/>
                        </div>
                    </div>
                    <div t-if="record.allocation_to_approve_count.raw_value > 0" class="row ml16">
                        <div class="col-9">
                            <a name="%(hr_leave_allocation_action_approve_department)d" type="action">
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
                <field name="current_leave_id" invisible="1"/>
                <field name="show_leaves" invisible="1"/>
                <field name="is_absent" invisible="1"/>
                <field name="hr_icon_display" invisible="1"/>
                <button disabled="1"
                        class="oe_stat_button"
                        context="{'search_default_employee_id': active_id}"
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
                <button name="%(act_hr_employee_holiday_request)d"
                        type="action"
                        class="oe_stat_button"
                        icon="fa-calendar"
                        attrs="{'invisible': [('show_leaves','=', False)]}"
                        context="{'search_default_employee_id': active_id}"
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
            <xpath expr="//field[@name='work_location']" position="after">
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
                <field name="current_leave_id" invisible="1"/>
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
        <field name="domain">['|', ('res_model_id', '=', False), ('res_model_id.model', 'in', ['hr.leave', 'hr.leave.allocation'])]</field>
        <field name="context">{'default_res_model': 'hr.leave'}</field>
    </record>
</odoo>

```

## File: views\resource_views.xml

```xml
<?xml version='1.0' encoding='UTF-8' ?>
<odoo>
    <!-- Holiday on resource leave -->
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

    cancel_leaves = fields.Boolean("Cancel Future Leaves", default=True)

    def action_register_departure(self):
        super(HrDepartureWizard, self).action_register_departure()
        if self.cancel_leaves:
            future_leaves = self.env['hr.leave'].search([('employee_id', '=', self.employee_id.id), 
                                                         ('date_to', '>', self.departure_date),
                                                         ('state', 'not in', ['cancel', 'refuse'])])
            future_leaves.write({'state': 'cancel'})

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
            <xpath expr="//group[@id='date']" position="inside">
                <field name="cancel_leaves" string="Cancel all time off after this date"/>
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
                        <button name="print_report" string="Print" type="object" class="btn-primary"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" />
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

