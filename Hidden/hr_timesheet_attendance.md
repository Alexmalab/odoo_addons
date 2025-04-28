# Odoo Module: hr_timesheet_attendance

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
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
    'version': '1.1',

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

## File: models\ir_ui_menu.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class IrUiMenu(models.Model):
    _inherit = 'ir.ui.menu'

    def _load_menus_blacklist(self):
        res = super()._load_menus_blacklist()
        if not (self.env.user.has_group('hr_attendance.group_hr_attendance_user') and self.env.user.has_group('hr_timesheet.group_hr_timesheet_user')):
            res.append(self.env.ref('hr_timesheet_attendance.menu_hr_timesheet_attendance_report').id)
        return res

```

## File: models\__init__.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_ui_menu

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

    user_id = fields.Many2one('res.users', readonly=True)
    date = fields.Date(readonly=True)
    total_timesheet = fields.Float("Timesheets Hours", readonly=True)
    total_attendance = fields.Float("Attendance Hours", readonly=True)
    total_difference = fields.Float("Hours Difference", readonly=True)
    timesheets_cost = fields.Float("Timesheet Cost", readonly=True)
    attendance_cost = fields.Float("Attendance Cost", readonly=True)
    cost_difference = fields.Float("Cost Difference", readonly=True)
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
                coalesce(sum(t.attendance), 0) - coalesce(sum(t.timesheet), 0) as total_difference,
                NULLIF(sum(t.timesheet) * t.emp_cost, 0) as timesheets_cost,
                NULLIF(sum(t.attendance) * t.emp_cost, 0) as attendance_cost,
                NULLIF((coalesce(sum(t.attendance), 0) -  coalesce(sum(t.timesheet), 0)) * t.emp_cost, 0)  as cost_difference
            FROM (
                SELECT
                    -hr_attendance.id AS id,
                    hr_employee.hourly_cost AS emp_cost,
                    resource_resource.user_id AS user_id,
                    hr_attendance.worked_hours AS attendance,
                    NULL AS timesheet,
                    CAST(hr_attendance.check_in
                            at time zone 'utc'
                            at time zone
                                (SELECT calendar.tz FROM resource_calendar as calendar
                                INNER JOIN hr_employee as employee ON employee.id = employee_id
                                WHERE calendar.id = employee.resource_calendar_id)
                    as DATE) as date,
                    resource_resource.company_id as company_id
                FROM hr_attendance
                LEFT JOIN hr_employee ON hr_employee.id = hr_attendance.employee_id
                LEFT JOIN resource_resource on resource_resource.id = hr_employee.resource_id
            UNION ALL
                SELECT
                    ts.id AS id,
                    hr_employee.hourly_cost AS emp_cost,
                    ts.user_id AS user_id,
                    NULL AS attendance,
                    ts.unit_amount AS timesheet,
                    ts.date AS date,
                    ts.company_id AS company_id
                FROM account_analytic_line AS ts
                LEFT JOIN hr_employee ON hr_employee.id = ts.employee_id
                WHERE ts.project_id IS NOT NULL
            ) AS t
            GROUP BY t.user_id, t.date, t.company_id, t.emp_cost
            ORDER BY t.date
        )
        """ % self._table)

    @api.model
    def read_group(self, domain, fields, groupby, offset=0, limit=None, orderby=False, lazy=True):
        if not orderby and groupby:
            orderby_list = [groupby] if isinstance(groupby, str) else groupby
            orderby_list = [field.split(':')[0] for field in orderby_list]
            orderby = ','.join([f"{field} desc" if field == 'date' else field for field in orderby_list])
        return super().read_group(domain, fields, groupby, offset=offset, limit=limit, orderby=orderby, lazy=lazy)

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
                    <field name="timesheets_cost"/>
                    <field name="attendance_cost"/>
                    <field name="cost_difference"/>
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
                    <field name="date" interval="month"/>
                    <field name="total_difference" type="measure" widget="timesheet_uom"/>
                </graph>
            </field>
        </record>

        <record id="action_hr_timesheet_attendance_report" model="ir.actions.act_window">
            <field name="name">Timesheet / Attendance</field>
            <field name="res_model">hr.timesheet.attendance.report</field>
            <field name="view_mode">graph,pivot</field>
            <field name="view_id" eval="False"/>
            <field name="context">{}</field>
            <field name="help" type="html">
                <p class="o_view_nocontent_empty_folder">
                    No data yet!
                </p><p>
                    Compare the time recorded by your employees with their attendance.
                </p>
            </field>
        </record>

        <record id="action_hr_timesheet_attendance_report_pivot" model="ir.actions.act_window.view">
            <field name="sequence" eval="1"/>
            <field name="view_mode">pivot</field>
            <field name="view_id" ref="view_hr_timesheet_attendance_report_pivot"/>
            <field name="act_window_id" ref="action_hr_timesheet_attendance_report"/>
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

    <record id="hr_timesheet_attendance_report_rule_user" model="ir.rule">
        <field name="name">Timesheet attendance Report: User</field>
        <field name="model_id" ref="model_hr_timesheet_attendance_report"/>
        <field name="domain_force">[('user_id', '=', user.id)]</field>
        <field name="groups" eval="[(4, ref('hr_timesheet.group_hr_timesheet_user'))]"/>
    </record>

    <record id="hr_timesheet_attendance_report_rule_approver" model="ir.rule">
        <field name="name">Timesheet attendance Report: Approver</field>
        <field name="model_id" ref="model_hr_timesheet_attendance_report"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('hr_timesheet.group_hr_timesheet_approver'))]"/>
    </record>

    <record id="hr_timesheet_attendance_report_rule_manager" model="ir.rule">
        <field name="name">Timesheet attendance Report: Administrator</field>
        <field name="model_id" ref="model_hr_timesheet_attendance_report"/>
        <field name="domain_force">[(1, '=', 1)]</field>
        <field name="groups" eval="[(4, ref('hr_timesheet.group_timesheet_manager'))]"/>
    </record>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_timesheet_attendance_report,access_hr_timesheet_attendance_report,model_hr_timesheet_attendance_report,hr_timesheet.group_hr_timesheet_user,1,0,0,0

```

## File: upgrades\1.1\pre-migrate.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


def migrate(cr, version):
    cr.execute("""
        UPDATE ir_rule r
           SET domain_force = '[(1, "=", 1)]'
          FROM ir_model_data d
         WHERE d.res_id = r.id
           AND d.model = 'ir.rule'
           AND d.module = 'hr_timesheet_attendance'
           AND d.name = 'hr_timesheet_attendance_report_rule_approver'
    """)

```

