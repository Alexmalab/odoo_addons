# Odoo Module: hr_calendar

Category: Human Resources/Employees

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Display Working Hours in Calendar",
    'version': '1.0',
    'category': 'Human Resources/Employees',
    'depends': ['hr', 'calendar'],
    'auto_install': True,
    'data': [
        'views/calendar_views_calendarApp.xml',
        'views/res_partner_views.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'hr_calendar/static/src/**/*',
        ],
        'web.qunit_suite_tests': [
            'hr_calendar/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\calendar_event.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from dateutil.relativedelta import relativedelta
from pytz import UTC

from odoo import api, fields, models

from odoo.addons.resource.models.utils import Intervals, sum_intervals, timezone_datetime


class CalendarEvent(models.Model):
    _inherit = "calendar.event"

    unavailable_partner_ids = fields.Many2many('res.partner', compute='_compute_unavailable_partner_ids')

    @api.depends('partner_ids', 'start', 'stop', 'allday')
    def _compute_unavailable_partner_ids(self):
        complete_events = self.filtered(
            lambda event: event.start and event.stop and (event.stop > event.start or (event.stop >= event.start and event.allday)) and event.partner_ids)
        incomplete_event = self - complete_events
        incomplete_event.unavailable_partner_ids = []
        if not complete_events:
            return
        event_intervals = complete_events._get_events_interval()
        for event, event_interval in event_intervals.items():
            # Event_interval is empty when an allday event contains at least one day where the company is closed
            if not event_interval:
                event.unavailable_partner_ids = event.partner_ids
                continue
            start = event_interval._items[0][0]
            stop = event_interval._items[-1][1]
            schedule_by_partner = event.partner_ids._get_schedule(start, stop, merge=False)
            event.unavailable_partner_ids = event._check_employees_availability_for_event(
                schedule_by_partner, event_interval)

    @api.model
    def get_unusual_days(self, date_from, date_to=None):
        return self.env.user.employee_id._get_unusual_days(date_from, date_to)

    def _get_events_interval(self):
        """
        This method will returned an Intervals object that represent the event's interval based of its parameters.

        If an event is scheduled for the entire day, its interval will correspond to the work interval defined by the
        company's calendar.
        If an allday event is scheduled on a day when the company is closed, the interval of this event will be empty.
        """
        start = min(self.mapped('start')).replace(hour=0, minute=0, second=0, tzinfo=UTC)
        stop = max(self.mapped('stop')).replace(hour=23, minute=59, second=59, tzinfo=UTC)
        company_calendar = self.env.company.resource_calendar_id
        global_interval = company_calendar._work_intervals_batch(start, stop)[False]
        interval_by_event = {}
        for event in self:
            if event.allday:
                # Avoid allday event with a duration of 0
                allday_event_interval = Intervals([(
                    event.start.replace(hour=0, minute=0, second=0, tzinfo=UTC),
                    event.stop.replace(hour=23, minute=59, second=59, tzinfo=UTC),
                    self.env['resource.calendar']
                )])

                if any(not (Intervals([(
                    event.start.replace(hour=0, minute=0, second=0, tzinfo=UTC) + relativedelta(days=i),
                    event.start.replace(hour=23, minute=59, second=59, tzinfo=UTC) + relativedelta(days=i),
                    self.env['resource.calendar']
                )]) & global_interval) for i in range(0, (event.stop_date - event.start_date).days + 1)):
                    interval_by_event[event] = Intervals([])
                else:
                    interval_by_event[event] = allday_event_interval & global_interval
            else:
                interval_by_event[event] = Intervals([(
                    timezone_datetime(event.start),
                    timezone_datetime(event.stop),
                    self.env['resource.calendar']
                )])
        return interval_by_event

    def _check_employees_availability_for_event(self, schedule_by_partner, event_interval):
        unavailable_partners = []
        for partner, schedule in schedule_by_partner.items():
            common_interval = schedule & event_interval
            if sum_intervals(common_interval) != sum_intervals(event_interval):
                unavailable_partners.append(partner.id)
        return unavailable_partners

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from pytz import UTC, timezone
from datetime import datetime
from collections import defaultdict
from functools import reduce

from odoo import api, models

from odoo.osv import expression
from odoo.addons.resource.models.utils import Intervals


class Partner(models.Model):
    _inherit = ['res.partner']

    def _get_employees_from_attendees(self, everybody=False):
        domain = [
            ('company_id', 'in', self.env.companies.ids),
            ('work_contact_id', '!=', False),
        ]
        if not everybody:
            domain = expression.AND([
                domain,
                [('work_contact_id', 'in', self.ids)]
            ])
        return dict(self.env['hr.employee'].sudo()._read_group(domain, groupby=['work_contact_id'], aggregates=['id:recordset']))

    def _get_schedule(self, start_period, stop_period, everybody=False, merge=True):
        """
        This method implements the general case where employees might have different resource calendars at different
        times, even though this is not the case with only this module installed.
        This way it will work with these other modules by just overriding
        `_get_calendar_periods`.

        :param datetime start_period: the start of the period
        :param datetime stop_period: the stop of the period
        :param boolean everybody: represents the "everybody" filter on calendar
        :param boolean merge: specifies if calendar's work_intervals needs to be merged
        :return: schedule (merged or not) by partner
        :rtype: defaultdict
        """
        employees_by_partner = self._get_employees_from_attendees(everybody)
        if not employees_by_partner:
            return {}
        interval_by_calendar = defaultdict()
        calendar_periods_by_employee = defaultdict(list)
        resources_by_calendar = defaultdict(lambda: self.env['resource.resource'])

        # Compute employee's calendars's period and order employee by his involved calendars
        employees = sum(employees_by_partner.values(), start=self.env['hr.employee'])
        calendar_periods_by_employee = employees._get_calendar_periods(start_period, stop_period)
        for employee, calendar_periods in calendar_periods_by_employee.items():
            for (start, stop, calendar) in calendar_periods:
                calendar = calendar or self.env.company.resource_calendar_id  # No calendar if fully flexible
                resources_by_calendar[calendar] += employee.resource_id

        # Compute all work intervals per calendar
        for calendar, resources in resources_by_calendar.items():
            work_intervals = calendar._work_intervals_batch(start_period, stop_period, resources=resources, tz=timezone(calendar.tz))
            del work_intervals[False]
            # Merge all employees intervals to avoid to compute it multiples times
            if merge:
                interval_by_calendar[calendar] = reduce(Intervals.__and__, work_intervals.values())
            else:
                interval_by_calendar[calendar] = work_intervals

        # Compute employee's schedule based own his calendar's periods
        schedule_by_employee = defaultdict(list)
        for employee, calendar_periods in calendar_periods_by_employee.items():
            employee_interval = Intervals([])
            for (start, stop, calendar) in calendar_periods:
                calendar = calendar or self.env.company.resource_calendar_id # No calendar if fully flexible
                interval = Intervals([(start, stop, self.env['resource.calendar'])])
                if merge:
                    calendar_interval = interval_by_calendar[calendar]
                else:
                    calendar_interval = interval_by_calendar[calendar][employee.resource_id.id]
                employee_interval = employee_interval | (calendar_interval & interval)
            schedule_by_employee[employee] = employee_interval

        # Compute partner's schedule equals to the union between his employees's schedule
        schedules = defaultdict()
        for partner, employees in employees_by_partner.items():
            partner_schedule = Intervals([])
            for employee in employees:
                if schedule_by_employee[employee]:
                    partner_schedule = partner_schedule | schedule_by_employee[employee]
            schedules[partner] = partner_schedule
        return schedules

    @api.model
    def get_working_hours_for_all_attendees(self, attendee_ids, date_from, date_to, everybody=False):

        start_period = datetime.fromisoformat(date_from).replace(hour=0, minute=0, second=0, tzinfo=UTC)
        stop_period = datetime.fromisoformat(date_to).replace(hour=23, minute=59, second=59, tzinfo=UTC)

        schedule_by_partner = self.env['res.partner'].browse(attendee_ids)._get_schedule(start_period, stop_period, everybody)
        if not schedule_by_partner:
            return []
        return self._interval_to_business_hours(reduce(Intervals.__and__, schedule_by_partner.values()))

    def _interval_to_business_hours(self, working_intervals):
        # This is the format expected by the fullcalendar library to do the overlay
        return [{
            "daysOfWeek": [(interval[0].weekday() + 1) % 7],
            "startTime":  interval[0].astimezone(timezone(self.env.user.tz or 'UTC')).strftime("%H:%M"),
            "endTime": interval[1].astimezone(timezone(self.env.user.tz or 'UTC')).strftime("%H:%M"),
        } for interval in working_intervals] if working_intervals else [{
            # 7 is used a dummy value to gray the full week
            # Returning an empty list would leave the week uncolored
            "daysOfWeek": [7],
            "startTime":  datetime.today().strftime("00:00"),
            "endTime": datetime.today().strftime("00:00"),
        }]

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import calendar_event
from . import res_partner

```

## File: static\src\calendar\calendar_model.js

```javascript
import { AttendeeCalendarModel } from "@calendar/views/attendee_calendar/attendee_calendar_model";
import { serializeDate } from "@web/core/l10n/dates";
import { patch } from "@web/core/utils/patch";

patch(AttendeeCalendarModel.prototype, {
    setup() {
        super.setup(...arguments)
        this.data.workingHours = {};
    },

    get workingHours() {
        return this.data.workingHours;
    },

    async updateData(data) {
        await super.updateData(...arguments)
        data.workingHours = await this.fetchWorkingHours(data);
    },

    async fetchWorkingHours(data){
        if (this.meta.scale !== "day" && this.meta.scale !== "week"){
            return [];
        }
        const attendeeFilters = data.filterSections.partner_ids;
        const activeAttendeeIds = attendeeFilters.filters
            .filter((filter) => filter.type !== "all" && filter.value && filter.active)
            .map((filter) => filter.value);
        const allFilter = attendeeFilters.filters.find((filter) => filter.type === "all");
        return this.orm.call("res.partner", "get_working_hours_for_all_attendees", [
            activeAttendeeIds,
            serializeDate(data.range.start),
            serializeDate(data.range.end),
            allFilter?.active
        ]);
    },
});

```

## File: static\src\calendar\common\calendar_common_renderer.js

```javascript
import { AttendeeCalendarCommonRenderer } from "@calendar/views/attendee_calendar/common/attendee_calendar_common_renderer";
import { patch } from "@web/core/utils/patch";
import { onWillUpdateProps } from "@odoo/owl";

patch(AttendeeCalendarCommonRenderer.prototype, {
	setup() {
		super.setup(...arguments);
		onWillUpdateProps(() => {
			this.fc.api.setOption("businessHours", this.props.model.workingHours)
		});
	},
	get options() {
		return Object.assign(super.options, {
			businessHours: this.props.model.workingHours,
		});
	},
});

```

## File: static\src\calendar\filter_panel\calendar_filter_panel.js

```javascript
/** @odoo-module **/

import { CalendarFilterPanel } from "@web/views/calendar/filter_panel/calendar_filter_panel";
import { patch } from "@web/core/utils/patch";


patch(CalendarFilterPanel.prototype, {
    updateSelectCreateDialogProps(props) {
        const updatedProps = super.updateSelectCreateDialogProps(props);
        updatedProps.context = {
            search_view_ref: 'hr_calendar.view_res_partner_filter_inherit_calendar',
        };
        return updatedProps;
    }
})

```

## File: static\src\views\fields\attendee_tags_list.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="hr_calendar.AttendeeTagsList" t-inherit="calendar.AttendeeTagsList" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('o_tag_badge_text')]" position="after">
            <div t-if="tag.unavailableIcon" title="unavailable" class="ms-1">
                <i class="fa fa-moon-o position-relative" role="img"/>
            </div>
        </xpath>
    </t>
</templates>

```

## File: static\src\views\fields\many2many_attendees.js

```javascript
import { Many2ManyAttendee } from "@calendar/views/fields/many2many_attendee";
import { patch } from "@web/core/utils/patch";
patch(Many2ManyAttendee.prototype, {
    get tags() {
        const tags = super.tags;
        if (this.props.record.data.unavailable_partner_ids) {
            const unavailablePartnerIds = this.props.record.data.unavailable_partner_ids.records;
            for (const tag of tags) {
                if (unavailablePartnerIds.find((partner) => (tag.resId == partner.resId))) {
                    tag.unavailableIcon = true;
                }
            }
        }
        return tags;
    }
})

```

## File: views\calendar_views_calendarApp.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<odoo>
    <record id="view_calendar_event_calendar" model="ir.ui.view">
        <field name="name">view.calendar.event.calendar.inherit.calendar</field>
        <field name="model">calendar.event</field>
        <field name="inherit_id" ref="calendar.view_calendar_event_calendar"/>
        <field name="arch" type="xml">
            <xpath expr="//calendar" position="attributes">
                <attribute name="show_unusual_days">True</attribute>
            </xpath>
        </field>
    </record>

    <record id="view_calendar_event_form_quick_create" model="ir.ui.view">
        <field name="name">view.calendar.event.calendar.quick.create.inherit.calendar</field>
        <field name="model">calendar.event</field>
        <field name="inherit_id" ref="calendar.view_calendar_event_form_quick_create"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='stop']" position="after">
                <field name="unavailable_partner_ids" invisible="1" /> <!-- this field will be used in
                    many2many_attendees widget -->
            </xpath>
        </field>
    </record>

    <record id="view_calendar_event_form" model="ir.ui.view">
        <field name="name">view.calendar.event.calendar.inherit.calendar</field>
        <field name="model">calendar.event</field>
        <field name="inherit_id" ref="calendar.view_calendar_event_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='stop']" position="after">
                <field name="unavailable_partner_ids" invisible="1" /> <!-- this field will be used in
                    many2many_attendees widget -->
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<odoo>
    <record id="view_res_partner_filter_inherit_calendar" model="ir.ui.view">
        <field name="name">res.partner.view.search.inherit.calendar</field>
        <field name="model">res.partner</field>
        <field name="mode">primary</field>
        <field name="inherit_id" ref="base.view_res_partner_filter"/>
        <field name="arch" type="xml">
                <filter name="type_company" position="after">
                    <filter string="My Team" name="type_team" domain="[('employee_ids.department_id.manager_id.user_id', '=', uid)]"/>
                </filter>
        </field>
    </record>
</odoo>

```

