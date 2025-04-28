# Odoo Module: hr_work_entry

Category: Human Resources/Employees

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
#-*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Work Entries',
    'category': 'Human Resources/Employees',
    'sequence': 39,
    'summary': 'Manage work entries',
    'installable': True,
    'depends': [
        'hr',
    ],
    'data': [
        'security/hr_work_entry_security.xml',
        'security/ir.model.access.csv',
        'data/hr_work_entry_data.xml',
        'views/hr_work_entry_views.xml',
        'views/hr_employee_views.xml',
        'views/resource_calendar_views.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'hr_work_entry/static/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\hr_work_entry_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">

        <!-- Work Entry Type -->
        <record id="work_entry_type_attendance" model="hr.work.entry.type">
            <field name="name">Attendance</field>
            <field name="color">0</field>
            <field name="code">WORK100</field>
        </record>

    </data>
</odoo>

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _

class HrEmployee(models.Model):
    _inherit = 'hr.employee'

    def action_open_work_entries(self, initial_date=False):
        self.ensure_one()
        ctx = {'default_employee_id': self.id}
        if initial_date:
            ctx['initial_date'] = initial_date
        return {
            'type': 'ir.actions.act_window',
            'name': _('%s work entries', self.display_name),
            'view_mode': 'calendar,tree,form',
            'res_model': 'hr.work.entry',
            'context': ctx,
            'domain': [('employee_id', '=', self.id)],
        }

```

## File: models\hr_work_entry.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from contextlib import contextmanager
from dateutil.relativedelta import relativedelta
import itertools
from psycopg2 import OperationalError

from odoo import api, fields, models, tools, _
from odoo.osv import expression


class HrWorkEntry(models.Model):
    _name = 'hr.work.entry'
    _description = 'HR Work Entry'
    _order = 'conflict desc,state,date_start'

    name = fields.Char(required=True, compute='_compute_name', store=True, readonly=False)
    active = fields.Boolean(default=True)
    employee_id = fields.Many2one('hr.employee', required=True, domain="['|', ('company_id', '=', False), ('company_id', '=', company_id)]", index=True)
    date_start = fields.Datetime(required=True, string='From')
    date_stop = fields.Datetime(compute='_compute_date_stop', store=True, readonly=False, string='To')
    duration = fields.Float(compute='_compute_duration', store=True, string="Duration", readonly=False)
    work_entry_type_id = fields.Many2one('hr.work.entry.type', index=True, default=lambda self: self.env['hr.work.entry.type'].search([], limit=1))
    color = fields.Integer(related='work_entry_type_id.color', readonly=True)
    state = fields.Selection([
        ('draft', 'Draft'),
        ('validated', 'Validated'),
        ('conflict', 'Conflict'),
        ('cancelled', 'Cancelled')
    ], default='draft')
    company_id = fields.Many2one('res.company', string='Company', readonly=True, required=True,
        default=lambda self: self.env.company)
    conflict = fields.Boolean('Conflicts', compute='_compute_conflict', store=True)  # Used to show conflicting work entries first
    department_id = fields.Many2one('hr.department', related='employee_id.department_id', store=True)

    # There is no way for _error_checking() to detect conflicts in work
    # entries that have been introduced in concurrent transactions, because of the transaction
    # isolation.
    # So if 2 transactions create work entries in parallel it is possible to create a conflict
    # that will not be visible by either transaction. There is no way to detect conflicts
    # between different records in a safe manner unless a SQL constraint is used, e.g. via
    # an EXCLUSION constraint [1]. This (obscure) type of constraint allows comparing 2 rows
    # using special operator classes and it also supports partial WHERE clauses. Similarly to
    # CHECK constraints, it's backed by an index.
    # 1: https://www.postgresql.org/docs/9.6/sql-createtable.html#SQL-CREATETABLE-EXCLUDE
    _sql_constraints = [
        ('_work_entry_has_end', 'check (date_stop IS NOT NULL)', 'Work entry must end. Please define an end date or a duration.'),
        ('_work_entry_start_before_end', 'check (date_stop > date_start)', 'Starting time should be before end time.'),
        (
            '_work_entries_no_validated_conflict',
            """
                EXCLUDE USING GIST (
                    tsrange(date_start, date_stop, '()') WITH &&,
                    int4range(employee_id, employee_id, '[]') WITH =
                )
                WHERE (state = 'validated' AND active = TRUE)
            """,
            'Validated work entries cannot overlap'
        ),
    ]

    def init(self):
        tools.create_index(self._cr, "hr_work_entry_date_start_date_stop_index", self._table, ["date_start", "date_stop"])

    @api.depends('work_entry_type_id', 'employee_id')
    def _compute_name(self):
        for work_entry in self:
            if not work_entry.employee_id:
                work_entry.name = _('Undefined')
            else:
                work_entry.name = "%s: %s" % (work_entry.work_entry_type_id.name or _('Undefined Type'), work_entry.employee_id.name)

    @api.depends('state')
    def _compute_conflict(self):
        for rec in self:
            rec.conflict = rec.state == 'conflict'

    @api.depends('date_stop', 'date_start')
    def _compute_duration(self):
        durations = self._get_duration_batch()
        for work_entry in self:
            work_entry.duration = durations[work_entry.id]

    @api.depends('date_start', 'duration')
    def _compute_date_stop(self):
        for work_entry in self.filtered(lambda w: w.date_start and w.duration):
            work_entry.date_stop = work_entry.date_start + relativedelta(hours=work_entry.duration)

    def _get_duration_batch(self):
        result = {}
        cached_periods = defaultdict(float)
        for work_entry in self:
            date_start = work_entry.date_start
            date_stop = work_entry.date_stop
            if not date_start or not date_stop:
                result[work_entry.id] = 0.0
                continue
            if (date_start, date_stop) in cached_periods:
                result[work_entry.id] = cached_periods[(date_start, date_stop)]
            else:
                dt = date_stop - date_start
                duration = dt.days * 24 + dt.seconds / 3600  # Number of hours
                cached_periods[(date_start, date_stop)] = duration
                result[work_entry.id] = duration
        return result

    # YTI TODO: Remove me in master: Deprecated, use _get_duration_batch instead
    def _get_duration(self, date_start, date_stop):
        return self._get_duration_batch()[self.id]

    def action_validate(self):
        """
        Try to validate work entries.
        If some errors are found, set `state` to conflict for conflicting work entries
        and validation fails.
        :return: True if validation succeded
        """
        work_entries = self.filtered(lambda work_entry: work_entry.state != 'validated')
        if not work_entries._check_if_error():
            work_entries.write({'state': 'validated'})
            return True
        return False

    def _check_if_error(self):
        if not self:
            return False
        undefined_type = self.filtered(lambda b: not b.work_entry_type_id)
        undefined_type.write({'state': 'conflict'})
        conflict = self._mark_conflicting_work_entries(min(self.mapped('date_start')), max(self.mapped('date_stop')))
        return undefined_type or conflict

    def _mark_conflicting_work_entries(self, start, stop):
        """
        Set `state` to `conflict` for overlapping work entries
        between two dates.
        If `self.ids` is truthy then check conflicts with the corresponding work entries.
        Return True if overlapping work entries were detected.
        """
        # Use the postgresql range type `tsrange` which is a range of timestamp
        # It supports the intersection operator (&&) useful to detect overlap.
        # use '()' to exlude the lower and upper bounds of the range.
        # Filter on date_start and date_stop (both indexed) in the EXISTS clause to
        # limit the resulting set size and fasten the query.
        self.flush_model(['date_start', 'date_stop', 'employee_id', 'active'])
        query = """
            SELECT b1.id,
                   b2.id
              FROM hr_work_entry b1
              JOIN hr_work_entry b2
                ON b1.employee_id = b2.employee_id
               AND b1.id <> b2.id
             WHERE b1.date_start <= %(stop)s
               AND b1.date_stop >= %(start)s
               AND b1.active = TRUE
               AND b2.active = TRUE
               AND tsrange(b1.date_start, b1.date_stop, '()') && tsrange(b2.date_start, b2.date_stop, '()')
               AND {}
        """.format("b2.id IN %(ids)s" if self.ids else "b2.date_start <= %(stop)s AND b2.date_stop >= %(start)s")
        self.env.cr.execute(query, {"stop": stop, "start": start, "ids": tuple(self.ids)})
        conflicts = set(itertools.chain.from_iterable(self.env.cr.fetchall()))
        self.browse(conflicts).write({
            'state': 'conflict',
        })
        return bool(conflicts)

    @api.model_create_multi
    def create(self, vals_list):
        work_entries = super().create(vals_list)
        work_entries._check_if_error()
        return work_entries

    def write(self, vals):
        skip_check = not bool({'date_start', 'date_stop', 'employee_id', 'work_entry_type_id', 'active'} & vals.keys())
        if 'state' in vals:
            if vals['state'] == 'draft':
                vals['active'] = True
            elif vals['state'] == 'cancelled':
                vals['active'] = False
                skip_check &= all(self.mapped(lambda w: w.state != 'conflict'))

        if 'active' in vals:
            vals['state'] = 'draft' if vals['active'] else 'cancelled'

        employee_ids = self.employee_id.ids
        if 'employee_id' in vals and vals['employee_id']:
            employee_ids += [vals['employee_id']]
        with self._error_checking(skip=skip_check, employee_ids=employee_ids):
            return super(HrWorkEntry, self).write(vals)

    def unlink(self):
        employee_ids = self.employee_id.ids
        with self._error_checking(employee_ids=employee_ids):
            return super().unlink()

    def _reset_conflicting_state(self):
        self.filtered(lambda w: w.state == 'conflict').write({'state': 'draft'})

    @contextmanager
    def _error_checking(self, start=None, stop=None, skip=False, employee_ids=False):
        """
        Context manager used for conflicts checking.
        When exiting the context manager, conflicts are checked
        for all work entries within a date range. By default, the start and end dates are
        computed according to `self` (min and max respectively) but it can be overwritten by providing
        other values as parameter.
        :param start: datetime to overwrite the default behaviour
        :param stop: datetime to overwrite the default behaviour
        :param skip: If True, no error checking is done
        """
        try:
            skip = skip or self.env.context.get('hr_work_entry_no_check', False)
            start = start or min(self.mapped('date_start'), default=False)
            stop = stop or max(self.mapped('date_stop'), default=False)
            if not skip and start and stop:
                domain = [
                    ('date_start', '<', stop),
                    ('date_stop', '>', start),
                    ('state', 'not in', ('validated', 'cancelled')),
                ]
                if employee_ids:
                    domain = expression.AND([domain, [('employee_id', 'in', list(employee_ids))]])
                work_entries = self.sudo().with_context(hr_work_entry_no_check=True).search(domain)
                work_entries._reset_conflicting_state()
            yield
        except OperationalError:
            # the cursor is dead, do not attempt to use it or we will shadow the root exception
            # with a "psycopg2.InternalError: current transaction is aborted, ..."
            skip = True
            raise
        finally:
            if not skip and start and stop:
                # New work entries are handled in the create method,
                # no need to reload work entries.
                work_entries.exists()._check_if_error()


class HrWorkEntryType(models.Model):
    _name = 'hr.work.entry.type'
    _description = 'HR Work Entry Type'

    name = fields.Char(required=True, translate=True)
    code = fields.Char(required=True, help="Careful, the Code is used in many references, changing it could lead to unwanted changes.")
    color = fields.Integer(default=0)
    sequence = fields.Integer(default=25)
    active = fields.Boolean(
        'Active', default=True,
        help="If the active field is set to false, it will allow you to hide the work entry type without removing it.")

    _sql_constraints = [
        ('unique_work_entry_code', 'UNIQUE(code)', 'The same code cannot be associated to multiple work entry types.'),
    ]


class Contacts(models.Model):
    """ Personnal calendar filter """

    _name = 'hr.user.work.entry.employee'
    _description = 'Work Entries Employees'

    user_id = fields.Many2one('res.users', 'Me', required=True, default=lambda self: self.env.user)
    employee_id = fields.Many2one('hr.employee', 'Employee', required=True)
    active = fields.Boolean('Active', default=True)

    _sql_constraints = [
        ('user_id_employee_id_unique', 'UNIQUE(user_id,employee_id)', 'You cannot have the same employee twice.')
    ]

```

## File: models\resource.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class ResourceCalendarAttendance(models.Model):
    _inherit = 'resource.calendar.attendance'

    def _default_work_entry_type_id(self):
        return self.env.ref('hr_work_entry.work_entry_type_attendance', raise_if_not_found=False)

    work_entry_type_id = fields.Many2one(
        'hr.work.entry.type', 'Work Entry Type', default=_default_work_entry_type_id,
        groups="hr.group_hr_user")

    def _copy_attendance_vals(self):
        res = super()._copy_attendance_vals()
        res['work_entry_type_id'] = self.work_entry_type_id.id
        return res


class ResourceCalendarLeave(models.Model):
    _inherit = 'resource.calendar.leaves'

    work_entry_type_id = fields.Many2one(
        'hr.work.entry.type', 'Work Entry Type',
        groups="hr.group_hr_user")

    def _copy_leave_vals(self):
        res = super()._copy_leave_vals()
        res['work_entry_type_id'] = self.work_entry_type_id.id
        return res

```

## File: models\__init__.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_work_entry
from . import resource
from . import hr_employee

```

## File: security\hr_work_entry_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

        <record id="hr_user_work_entry_employee" model="ir.rule">
            <field name="name">Work entries/Employee calendar filter: only self</field>
            <field name="model_id" ref="model_hr_user_work_entry_employee"/>
            <field name="domain_force">[('user_id', '=', user.id)]</field>
            <field name="groups" eval="[(4, ref('base.group_user'))]"/>
            <field name="perm_create" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_unlink" eval="1"/>
            <field name="perm_read" eval="0"/>
        </record>

</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_work_entry_officer,access_hr_work_entry_officer,model_hr_work_entry,hr.group_hr_user,1,1,1,0
access_hr_work_entry_system,access_hr_work_entry_system,model_hr_work_entry,base.group_system,1,1,1,1
access_hr_work_entry_type_officer,access_hr_work_entry_type_officer,model_hr_work_entry_type,hr.group_hr_user,1,0,0,0
access_hr_work_entry_type_manager,access_hr_work_entry_type_manager,model_hr_work_entry_type,hr.group_hr_manager,1,1,1,1
access_hr_work_entry_employee,access_hr_work_entry_employee,model_hr_user_work_entry_employee,hr.group_hr_user,1,1,1,1
```

## File: static\src\xml\work_entry_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="hr_work_entry.work_entry_button">
        <button t-if="disabled" disabled="" title="Solve conflicts first" t-attf-class="btn {{primary?'btn-primary':'btn-secondary'}} btn-work-entry {{ event_class }}" type="button" t-esc="button_text"/>
        <button t-else="" t-attf-class="btn {{primary?'btn-primary':'btn-secondary'}} btn-work-entry {{ event_class }}" type="button" t-esc="button_text"/>
    </t>
</templates>

```

## File: views\hr_employee_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_employee_view_form" model="ir.ui.view">
        <field name="name">hr.employee.view.form.inherit.hr.work.entry</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_form"/>
        <field name="priority" eval="90"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button type="object" class="oe_stat_button" id="open_work_entries"
                    icon="fa-calendar" name="action_open_work_entries" string="Work Entries">
                </button>
            </div>
        </field>
    </record>
</odoo>

```

## File: views\hr_work_entry_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- HR WORK ENTRY -->

    <record id="hr_work_entry_action_conflict" model="ir.actions.act_window">
        <field name="name">Work Entry</field>
        <field name="res_model">hr.work.entry</field>
        <field name="context">{'search_default_work_entries_error': 1}</field>
        <field name="view_mode">tree,calendar,form,pivot</field>
    </record>

    <record id="hr_work_entry_action" model="ir.actions.act_window">
        <field name="name">Work Entry</field>
        <field name="res_model">hr.work.entry</field>
        <field name="view_mode">calendar,tree,form,pivot</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_empty_folder">
                No data to display
            </p>
            <p>
                Try to add some records, or make sure that there is no active filter in the search bar.
            </p>
        </field>
    </record>

    <record id="hr_work_entry_view_calendar" model="ir.ui.view">
        <field name="name">hr.work.entry.calendar</field>
        <field name="model">hr.work.entry</field>
        <field name="arch" type="xml">
            <calendar string="Work Entry"
                date_start="date_start"
                date_stop="date_stop"
                mode="month"
                quick_add="False"
                color="color"
                event_limit="5">
                <!-- Sidebar favorites filters -->
                <field name="employee_id" write_model="hr.user.work.entry.employee" write_field="employee_id" avatar_field="avatar_128"/>
                <field name="state"/>
            </calendar>
        </field>
    </record>

    <record id="hr_work_entry_view_form" model="ir.ui.view">
        <field name="name">hr.work.entry.form</field>
        <field name="model">hr.work.entry</field>
        <field name="arch" type="xml">
            <form string="Work Entry" >
                <header>
                    <field name="state" widget="statusbar" readonly="1" statusbar_visible="draft,validated,conflict"/>
                </header>
                <div class="alert alert-warning text-center" role="alert" attrs="{'invisible': [('state', '!=', 'validated')]}">
                    Note: Validated work entries cannot be modified.
                </div>
                <sheet>
                    <group>
                        <group>
                            <field name="name" string="Description" placeholder="Work Entry Name" attrs="{'readonly': [('state', '=', 'validated')]}"/>
                            <field name="employee_id" attrs="{'readonly': [('state', '!=', 'draft')]}" />
                            <field name="work_entry_type_id" attrs="{'readonly': [('state', '=', 'validated')]}" options="{'no_create': True, 'no_open': True}"/>
                        </group>
                        <group>
                            <field name="date_start" attrs="{'readonly': [('state', '!=', 'draft')]}" />
                            <field name="date_stop" attrs="{'readonly': [('state', '!=', 'draft')]}" required="1"/>
                            <label for="duration"/>
                            <div class="o_row mw-50 mw-sm-25">
                                <field name="duration"
                                    nolabel="1"
                                    widget="float_time"
                                    class="o_hr_narrow_field"
                                    attrs="{'readonly': [('state', '!=', 'draft')]}" />
                                <span>Hours</span>
                            </div>
                            <field name="company_id" invisible="1"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="hr_work_entry_view_tree" model="ir.ui.view">
        <field name="name">hr.work.entry.tree</field>
        <field name="model">hr.work.entry</field>
        <field name="arch" type="xml">
            <tree multi_edit="1" sample="1">
                <field name="name"/>
                <field name="work_entry_type_id" options="{'no_create': True, 'no_open': True}"/>
                <field name="duration" widget="float_time" readonly="1"/>
                <field name="state"/>
                <field name="date_start" string="Beginning" readonly="1"/>
                <field name="date_stop" string="End" readonly="1"/>
            </tree>
        </field>
    </record>

    <record id="hr_work_entry_view_pivot" model="ir.ui.view">
        <field name="name">hr.work.entry.pivot</field>
        <field name="model">hr.work.entry</field>
        <field name="arch" type="xml">
            <pivot string="Work Entries" sample="1">
                <field name="duration" widget="float_time" readonly="1"/>
            </pivot>
        </field>
    </record>

    <record id="hr_work_entry_view_search" model="ir.ui.view">
        <field name="name">hr.work.entry.filter</field>
        <field name="model">hr.work.entry</field>
        <field name="arch" type="xml">
            <search string="Search Work Entry">
                <field name="employee_id"/>
                <field name="department_id"/>
                <field name="work_entry_type_id"/>
                <field name="name"/>
                <filter name="work_entries_error" string="Conflicting" domain="[('state', '=', 'conflict')]"/>
                <separator/>
                <filter name="date_filter" string="Date" date="date_start"/>
                <filter name="current_month" string="Current Month" domain="[
                    ('date_stop', '&gt;=', (context_today()).strftime('%Y-%m-01')),
                    ('date_start', '&lt;', (context_today() + relativedelta(months=1)).strftime('%Y-%m-01'))]"/>
                <separator/>
                <filter name="group_employee" string="Employee" context="{'group_by': 'employee_id'}"/>
                <filter name="group_department" string="Department" context="{'group_by': 'department_id'}"/>
                <filter name="group_work_entry_type" string="Type" context="{'group_by': 'work_entry_type_id'}"/>
                <filter name="group_start_date" string="Start Date" context="{'group_by': 'date_start'}"/>
                <separator/>
                <filter name="archived" string="Archived" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <!-- HR WORK ENTRY TYPE -->

    <record id="hr_work_entry_type_view_search" model="ir.ui.view">
        <field name="name">hr.work.entry.type.view.search</field>
        <field name="model">hr.work.entry.type</field>
        <field name="arch" type="xml">
            <search string="Search Work Entry Type">
                <field name="name" filter_domain="['|', ('name', 'ilike', self), ('code', 'ilike', self)]"/>
                <separator/>
                <filter name="archived" string="Archived" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <record id="hr_work_entry_type_action" model="ir.actions.act_window">
        <field name="name">Work Entry Types</field>
        <field name="res_model">hr.work.entry.type</field>
        <field name="view_mode">tree,kanban,form</field>
        <field name="search_view_id" ref="hr_work_entry_type_view_search"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a new work entry type
            </p>
        </field>
    </record>

    <record id="hr_work_entry_type_view_tree" model="ir.ui.view">
        <field name="name">hr.work.entry.type.tree</field>
        <field name="model">hr.work.entry.type</field>
        <field name="arch" type="xml">
            <tree>
                <field name="name"/>
                <field name="code"/>
                <field name="color" widget="color_picker"/>
            </tree>
        </field>
    </record>

    <record id="hr_work_entry_type_view_form" model="ir.ui.view">
        <field name="name">hr.work.entry.type.form</field>
        <field name="model">hr.work.entry.type</field>
        <field name="arch" type="xml">
            <form string="Work Entry Type" >
                <sheet>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title">
                        <h1>
                            <field name="name" placeholder="Work Entry Type Name"/>
                        </h1>
                    </div>
                    <group name="main_group">
                        <group name="identification" class="o_form_fw_labels">
                            <field name="active" invisible="1"/>
                            <field name="code"/>
                            <field name="sequence"/>
                            <field name="color" widget="color_picker"/>
                        </group>
                    </group>
                    <group name="other">
                        <group name="time_off" string="Time Off Options" class="o_form_fw_labels"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="hr_work_entry_type_view_kanban" model="ir.ui.view">
        <field name="name">hr.work.entry.type.kanban.view</field>
        <field name="model">hr.work.entry.type</field>
        <field name="arch" type="xml">
            <kanban>
                <field name="color"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="#{!selection_mode ? kanban_color(record.color.raw_value) : ''} oe_kanban_global_click">
                            <div class="o_dropdown_kanban dropdown" t-if="!selection_mode">
                                <a class="dropdown-toggle o-no-caret btn" role="button" data-bs-toggle="dropdown" href="#" aria-label="Dropdown menu" title="Dropdown menu">
                                    <span class="fa fa-ellipsis-v"/>
                                </a>
                                <div class="dropdown-menu" role="menu">
                                    <ul class="oe_kanban_colorpicker" data-field="color"/>
                                </div>
                            </div>
                            <div class="oe_kanban_content">
                                <div>
                                    <strong class="o_kanban_record_title"><span><field name="name"/></span></strong>
                                </div>
                                <div>
                                    <span class="text-muted o_kanban_record_subtitle"><field name="code"/></span>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

</odoo>

```

## File: views\resource_calendar_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="resource_calendar_leaves_view_search_inherit" model="ir.ui.view">
        <field name="name">resource.calendar.leaves.search.inherit</field>
        <field name="model">resource.calendar.leaves</field>
        <field name="inherit_id" ref="resource.view_resource_calendar_leaves_search"/>
        <field name="arch" type="xml">
            <filter name="resource" position="after">
                <field name="work_entry_type_id" groups="hr.group_hr_user"/>
                <filter name="work_entry" string="Work Entry Type" context="{'group_by':'work_entry_type_id'}" groups="hr.group_hr_user"/>
            </filter>
        </field>
    </record>
    <record id="resource_calendar_attendance_view_tree" model="ir.ui.view">
        <field name="name">resource.calendar.attendance.tree.inherit.hr.work.entry</field>
        <field name="model">resource.calendar.attendance</field>
        <field name="inherit_id" ref="resource.view_resource_calendar_attendance_tree"/>
        <field name="arch" type="xml">
            <field name="week_type" position="after">
                <field name="work_entry_type_id"/>
            </field>
        </field>
    </record>

    <record id="resource_calendar_attendance_view_form" model="ir.ui.view">
        <field name="name">resource.calendar.attendance.form.inherit.hr.work.entry</field>
        <field name="model">resource.calendar.attendance</field>
        <field name="inherit_id" ref="resource.view_resource_calendar_attendance_form"/>
        <field name="arch" type="xml">
            <field name="day_period" position="after">
                <field name="work_entry_type_id"/>
            </field>
        </field>
    </record>

    <record id="resource_calendar_leave_view_form" model="ir.ui.view">
        <field name="name">resource.calendar.leaves.form.inherit.hr.work.entry</field>
        <field name="model">resource.calendar.leaves</field>
        <field name="inherit_id" ref="resource.resource_calendar_leave_form"/>
        <field name="arch" type="xml">
            <field name="resource_id" position="after">
                <field name="work_entry_type_id"/>
            </field>
        </field>
    </record>

    <record id="resource_calendar_leave_view_tree" model="ir.ui.view">
        <field name="name">resource.calendar.leaves.tree.inherit.hr.work.entry</field>
        <field name="model">resource.calendar.leaves</field>
        <field name="inherit_id" ref="resource.resource_calendar_leave_tree"/>
        <field name="arch" type="xml">
            <field name="date_to" position="after">
                <field name="work_entry_type_id"/>
            </field>
        </field>
    </record>
</odoo>

```

