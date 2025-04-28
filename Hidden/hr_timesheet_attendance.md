# Odoo Module: hr_timesheet_attendance

Category: Hidden

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
    'name': "Timesheets/attendances reporting",
    'description': """
    Module linking the attendance module to the timesheet app.
    """,
    'category': 'Hidden',
    'version': '1.0',

    'depends': ['hr_timesheet', 'hr_attendance'],
    'data': [
        'security/ir.model.access.csv',
        'report/hr_timesheet_attendance_report_view.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: report\hr_timesheet_attendance_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools


class TimesheetAttendance(models.Model):
    _name = 'hr.timesheet.attendance.report'
    _auto = False
    _description = 'Timesheet Attendance Report'

    user_id = fields.Many2one('res.users')
    date = fields.Date()
    total_timesheet = fields.Float()
    total_attendance = fields.Float()
    total_difference = fields.Float()

    def init(self):
        tools.drop_view_if_exists(self.env.cr, self._table)
        self._cr.execute("""CREATE OR REPLACE VIEW %s AS (
            SELECT
                max(id) AS id,
                t.user_id,
                t.date,
                coalesce(sum(t.attendance), 0) AS total_attendance,
                coalesce(sum(t.timesheet), 0) AS total_timesheet,
                coalesce(sum(t.attendance), 0) - coalesce(sum(t.timesheet), 0) as total_difference
            FROM (
                SELECT
                    -hr_attendance.id AS id,
                    resource_resource.user_id AS user_id,
                    hr_attendance.worked_hours AS attendance,
                    NULL AS timesheet,
                    hr_attendance.check_in::date AS date
                FROM hr_attendance
                LEFT JOIN hr_employee ON hr_employee.id = hr_attendance.employee_id
                LEFT JOIN resource_resource on resource_resource.id = hr_employee.resource_id
            UNION ALL
                SELECT
                    ts.id AS id,
                    ts.user_id AS user_id,
                    NULL AS attendance,
                    ts.unit_amount AS timesheet,
                    ts.date AS date
                FROM account_analytic_line AS ts
                WHERE ts.project_id IS NOT NULL
            ) AS t
            GROUP BY t.user_id, t.date
            ORDER BY t.date
        )
        """ % self._table)

```

## File: report\hr_timesheet_attendance_report_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_hr_timesheet_attendance_report_search" model="ir.ui.view">
            <field name="name">Search for HR timesheet attendance report</field>
            <field name="model">hr.timesheet.attendance.report</field>
            <field name="arch" type="xml">
                <search string="timesheet attendance">
                    <field name="user_id"/>
                    <filter name="month" string="Date" date="date"/>
                </search>
            </field>
        </record>
        <record id="view_hr_timesheet_attendance_report_pivot" model="ir.ui.view">
            <field name="name">HR timesheet attendance report: Pivot</field>
            <field name="model">hr.timesheet.attendance.report</field>
            <field name="arch" type="xml">
                <pivot string="timesheet attendance" disable_linking="True">
                    <field name="user_id" type="row"/>
                    <field name="date" interval="day" type="col"/>
                    <field name="total_difference" type="measure" widget="float_time"/>
                    <field name="total_timesheet" type="measure" widget="float_time"/>
                    <field name="total_attendance" type="measure" widget="float_time"/>
                </pivot>
            </field>
        </record>

        <record id="action_hr_timesheet_attendance_report" model="ir.actions.act_window">
            <field name="name">HR Timesheet/Attendance Report</field>
            <field name="res_model">hr.timesheet.attendance.report</field>
            <field name="view_mode">pivot</field>
            <field name="context">{'search_default_week': True}</field>
        </record>

        <menuitem id="menu_hr_timesheet_attendance_report"
                  parent="hr_timesheet.menu_timesheets_reports"
                  action="action_hr_timesheet_attendance_report"
                  name="Timesheet / Attendance"/>
    </data>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_timesheet_attendance_report

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_timesheet_attendance_report,access_hr_timesheet_attendance_report,model_hr_timesheet_attendance_report,hr_timesheet.group_hr_timesheet_user,1,0,0,0

```

