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
        'security/hr_timesheet_attendance_report_security.xml',
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
    company_id = fields.Many2one('res.company', string='Company', readonly=True)

    def init(self):
        tools.drop_view_if_exists(self.env.cr, self._table)
        self._cr.execute("""CREATE OR REPLACE VIEW %s AS (
            SELECT
                max(id) AS id,
                t.user_id,
                t.date,
                t.company_id,
                coalesce(sum(t.attendance), 0) AS total_attendance,
                coalesce(sum(t.timesheet), 0) AS total_timesheet,
                coalesce(sum(t.attendance), 0) - coalesce(sum(t.timesheet), 0) as total_difference
            FROM (
                SELECT
                    -hr_attendance.id AS id,
                    resource_resource.user_id AS user_id,
                    hr_attendance.worked_hours AS attendance,
                    NULL AS timesheet,
                    hr_attendance.check_in::date AS date,
                    resource_resource.company_id as company_id
                FROM hr_attendance
                LEFT JOIN hr_employee ON hr_employee.id = hr_attendance.employee_id
                LEFT JOIN resource_resource on resource_resource.id = hr_employee.resource_id
            UNION ALL
                SELECT
                    ts.id AS id,
                    ts.user_id AS user_id,
                    NULL AS attendance,
                    ts.unit_amount AS timesheet,
                    ts.date AS date,
                    ts.company_id AS company_id
                FROM account_analytic_line AS ts
                WHERE ts.project_id IS NOT NULL
            ) AS t
            GROUP BY t.user_id, t.date, t.company_id
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
                <search string="Timesheet Attendance">
                    <field name="user_id" string="Employee"/>
                    <filter name="month" string="Date" date="date"/>
                    <filter name="group_by_user" string="Employee" context="{'group_by': 'user_id'}"/>
                    <filter name="group_by_month" string="Date" date="date" context="{'group_by': 'date'}"/>
                </search>
            </field>
        </record>
        <record id="view_hr_timesheet_attendance_report_pivot" model="ir.ui.view">
            <field name="name">HR timesheet attendance report: Pivot</field>
            <field name="model">hr.timesheet.attendance.report</field>
            <field name="arch" type="xml">
                <pivot string="Timesheet Attendance" disable_linking="1" sample="1">
                    <field name="date" interval="month" type="row"/>
                    <field name="total_attendance" type="measure" widget="timesheet_uom"/>
                    <field name="total_timesheet" type="measure" widget="timesheet_uom"/>
                    <field name="total_difference" type="measure" widget="timesheet_uom"/>
                </pivot>
            </field>
        </record>

        <record id="hr_timesheet_attendance_report_view_tree" model="ir.ui.view">
            <field name="name">hr.timesheet.attendance.report.view.tree</field>
            <field name="model">hr.timesheet.attendance.report</field>
            <field name="arch" type="xml">
                <tree string="Timesheet Attendance">
                    <field name="date"/>
                    <field name="user_id" optional="show" widget="many2one_avatar_user"/>
                    <field name="total_timesheet" optional="show" sum="Sum of Total Timesheet"/>
                    <field name="total_attendance" optional="show" sum="Sum of Total Attendance"/>
                    <field name="total_difference" optional="show" sum="Sum of Total Difference"/>
                </tree>
            </field>
        </record>

        <record id="hr_timesheet_attendance_report_view_graph" model="ir.ui.view">
            <field name="name">hr.timesheet.attendance.report.view.graph</field>
            <field name="model">hr.timesheet.attendance.report</field>
            <field name="arch" type="xml">
                <graph string="Timesheet Attendance" sample="1" disable_linking="1">
                    <field name="total_difference" type="measure" widget="timesheet_uom"/>
                </graph>
            </field>
        </record>

        <record id="action_hr_timesheet_attendance_report" model="ir.actions.act_window">
            <field name="name">Timesheet / Attendance</field>
            <field name="res_model">hr.timesheet.attendance.report</field>
            <field name="view_mode">graph,pivot</field>
            <field name="view_id" ref="view_hr_timesheet_attendance_report_pivot"/>
            <field name="context">{'search_default_group_by_month': True}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_smiling_face">
                    No data yet!
                </p>
            </field>
        </record>

        <record id="action_hr_timesheet_attendance_report_graph" model="ir.actions.act_window.view">
            <field name="view_mode">graph</field>
            <field name="view_id" ref="hr_timesheet_attendance_report_view_graph"/>
            <field name="act_window_id" ref="action_hr_timesheet_attendance_report"/>
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

## File: security\hr_timesheet_attendance_report_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="hr_timesheet_attendance_report_restricted_company_rule" model="ir.rule">
        <field name="name">Restricted Timesheet attendance Record: multi-company</field>
        <field name="model_id" ref="model_hr_timesheet_attendance_report"/>
        <field name="domain_force"> [('company_id', 'in', company_ids + [False])]</field>
    </record>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_timesheet_attendance_report,access_hr_timesheet_attendance_report,model_hr_timesheet_attendance_report,hr_timesheet.group_hr_timesheet_user,1,0,0,0

```

