# Odoo Module: hr_holidays_calendar

Category: Human Resources/Time Off

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import report

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Time Off - Calendar',
    'version': '1.5',
    'category': 'Human Resources/Time Off',
    'summary': '',
    'description': """""",
    'depends': ['hr_holidays', 'calendar'],
    'data': [
        'report/hr_leave_report_calendar_views.xml',
        'security/ir.model.access.csv',
        'security/hr_holidays_calendar_security.xml',
    ],
    'installable': True,
    'application': False,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: report\hr_leave_report_calendar.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools

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
    company_id = fields.Many2one('res.company', readonly=True)

    def init(self):
        tools.drop_view_if_exists(self._cr, 'hr_leave_report_calendar')

        self._cr.execute("""CREATE OR REPLACE VIEW hr_leave_report_calendar AS
        (SELECT 
            row_number() OVER() AS id,
            ce.name AS name,
            ce.start_datetime AS start_datetime,
            ce.stop_datetime AS stop_datetime,
            ce.event_tz AS tz,
            ce.duration AS duration,
            hl.employee_id AS employee_id,
            em.company_id AS company_id
        FROM hr_leave hl
            LEFT JOIN calendar_event ce
                ON ce.id = hl.meeting_id
            LEFT JOIN hr_employee em
                ON em.id = hl.employee_id
        WHERE 
            hl.state = 'validate');
        """)

```

## File: report\hr_leave_report_calendar_views.xml

```xml
<?xml version='1.0' encoding='UTF-8' ?>
<odoo>
    <record id="hr_holidays.action_hr_holidays_dashboard" model="ir.actions.act_window">
        <field name="res_model">hr.leave.report.calendar</field>
        <field name="view_mode">calendar</field>
        <!-- YTI : Ecrabouille explicitely those fields, as this is deployed on 
                   a stable release. -->
        <field name="view_id"/>
        <field name="search_view_id"/>
        <field name="domain">[('employee_id.active','=',True)]</field>
        <field name="context"/>
    </record>

    <record id="hr_leave_report_calendar_view" model="ir.ui.view">
        <field name="name">hr.leave.report.calendar.view</field>
        <field name="model">hr.leave.report.calendar</field>
        <field name="arch" type="xml">
            <calendar string="Time Off" date_start="start_datetime" date_stop="stop_datetime" mode="month" quick_add="False" color="employee_id" event_open_popup="True">
                <field name="name"/>
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

from . import hr_leave_report_calendar

```

## File: security\hr_holidays_calendar_security.xml

```xml
<odoo>
    <record id="hr_leave_report_calendar_rule_multi_company" model="ir.rule">
        <field name="name">Time Off Report Calendar: multi company global rule</field>
        <field name="model_id" ref="model_hr_leave_report_calendar"/>
        <field name="global" eval="True"/>
        <field name="domain_force">['|', ('company_id', '=', False), ('company_id', 'in', company_ids)]</field>
    </record>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_leave_report_calendar,access_hr_leave_report_calendar,model_hr_leave_report_calendar,base.group_user,1,0,0,0

```

