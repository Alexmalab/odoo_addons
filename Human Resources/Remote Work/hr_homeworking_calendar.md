# Odoo Module: hr_homeworking_calendar

Category: Human Resources/Remote Work

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models
from . import wizard

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Remote Work with calendar',
    'version': '1.0',
    'category': 'Human Resources/Remote Work',
    'depends': ['hr_homeworking', 'calendar'],
    'data': [
        'security/security.xml',
        'security/ir.model.access.csv',
        'wizard/homework_location_wizard.xml',
    ],
    'installable': True,
    'auto_install': True,
    'assets': {
        'web.assets_backend': [
            'hr_homeworking_calendar/static/src/**/*',
        ],
        'web.qunit_suite_tests': [
            'hr_homeworking_calendar/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\hr_employee.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import models
from odoo.tools import DEFAULT_SERVER_DATE_FORMAT

from odoo.addons.hr_homeworking.models.hr_homeworking import DAYS


class HrEmployeeBase(models.AbstractModel):
    _inherit = "hr.employee.base"

    def _get_worklocation(self, start_date, end_date):
        work_locations_by_employee = defaultdict(dict)
        for employee in self:
            work_locations_by_employee[employee.id].update({
                "user_id": employee.user_id.id,
                "employee_id": employee.id,
                "partner_id": employee.user_partner_id.id or employee.work_contact_id.id,
                "employee_name": employee.name
            })

            for day in DAYS:
                work_locations_by_employee[employee.id][day] = {
                    'location_type': employee[day]["location_type"],
                    'location_name': employee[day]["name"],
                    'work_location_id': employee[day].id,
                }

        exceptions_for_period = self.env['hr.employee.location'].search_read([
            ('employee_id', 'in', self.ids),
            ('date', '>=', start_date),
            ('date', '<=', end_date)
        ], ['employee_id', 'date', 'work_location_name', 'work_location_id', 'work_location_type'])

        for exception in exceptions_for_period:
            date = exception["date"].strftime(DEFAULT_SERVER_DATE_FORMAT)
            exception_value = {
                'hr_employee_location_id': exception["id"],
                'location_type': exception['work_location_type'],
                'location_name': exception['work_location_name'],
                'work_location_id': exception['work_location_id'][0],
            }
            employee_id = exception["employee_id"][0]
            if "exceptions" not in work_locations_by_employee[employee_id]:
                work_locations_by_employee[employee_id]["exceptions"] = {}
            work_locations_by_employee[employee_id]["exceptions"][date] = exception_value

        return work_locations_by_employee

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

class ResPartner(models.Model):
    _inherit = 'res.partner'

    def get_worklocation(self, start_date, end_date):
        employee_id = self.env['hr.employee'].search([
            ('work_contact_id.id', 'in', self.ids),
            ('company_id.id', '=', self.env.company.id)])
        return employee_id._get_worklocation(start_date, end_date)

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import hr_employee
from . import res_partner

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_homework_location_wizard,homework.location.wizard,model_homework_location_wizard,base.group_user,1,1,1,1

```

## File: security\security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="homeworking_location_wizard_own_rule" model="ir.rule">
            <field name="name">homeworking wizard: own</field>
            <field name="model_id" ref="model_homework_location_wizard"/>
            <field name="groups" eval="[(4, ref('base.group_user'))]"/>
            <field name="domain_force">[
                ('employee_id', '=', user.employee_id.id)
            ]</field>
        </record>

        <record id="homeworking_location_wizard_admin_rule" model="ir.rule">
            <field name="name">homeworking wizard: admin</field>
            <field name="model_id" ref="model_homework_location_wizard"/>
            <field name="groups" eval="[(4, ref('hr.group_hr_user'))]"/>
            <field name="domain_force">[(1, '=', 1)]</field>
        </record>
    </data>
</odoo>

```

## File: static\src\calendar\common\calendar_common_renderer.js

```javascript
/** @odoo-module **/

import { AttendeeCalendarCommonRenderer } from "@calendar/views/attendee_calendar/common/attendee_calendar_common_renderer";
import { AttendeeCalendarRenderer } from "@calendar/views/attendee_calendar/attendee_calendar_renderer";
import { user } from "@web/core/user";
import { patch } from "@web/core/utils/patch";
import { renderToString } from "@web/core/utils/render";
import { onPatched } from "@odoo/owl";

const { DateTime } = luxon;

patch(AttendeeCalendarCommonRenderer.prototype, {
    setup() {
        super.setup()

        onPatched(() => {
            // Force to rerender the FC.
            // As it doesn't redraw the header when the event's data changes
            this.fc.api.render();
        });
    },
    get options(){
        return {
            ...super.options,
            eventOrder: function(event1, event2){
                if (event1.extendedProps.worklocation){
                    return -1;
                } else {
                    if(event2.extendedProps.worklocation){
                        return 1;
                    } else {
                        return event1.title.localeCompare(event2.title);
                    }
                }
            },
            dayCellDidMount: this.onDayCellDidMount,
            dayHeaderDidMount: this.onDayHeaderDidMount,
            dayHeaderWillUnmount: this.onDayHeaderWillUnmount,
        };
    },
    handleWorkLocationClick(target, date) {
        let worklocations = this.props.model.worklocations[date.toISODate()];
        const worklocationSet = worklocations && Object.keys(worklocations).length > 0;
        const actionElement = target.closest('.wl_action');
        if (!actionElement) {
            return;
        }
        const { location, id, create } = actionElement.dataset;
        if (worklocationSet && !create) {
            if (!worklocations.id) {
                worklocations = worklocations[location] && worklocations[location].find(wl => wl.id == id);
            }
            if (worklocations) {
                return this.openPopover(target, worklocations);
            }
        }
        return this.props.openWorkLocationWizard(date);
    },
    onDayHeaderDidMount(info) {
        if (this.props.model.scale === 'week' || this.props.model.scale === 'day') {
            const date = DateTime.fromJSDate(info.date);
            const handler = (event) => {
                if (event.target.closest(".o_worklocation_btn")) {
                    this.handleWorkLocationClick(event.target, date);
                }
            };
            this.customListeners = {
                ...this.customListeners,
                [date]: handler,
            };
            info.el.addEventListener("click", handler);
        }
    },
    onDayHeaderWillUnmount(info) {
        if (this.props.model.scale === 'week' || this.props.model.scale === 'day') {
            const date = DateTime.fromJSDate(info.date);
            const customListener = {...this.customListeners}[date];
            if (customListener) {
                info.el.removeEventListener("click", customListener);
                delete this.customListeners[date];
            }
        }
    },
    onDayCellDidMount(info){
        if (this.props.model.scale === 'month'){
            const box = info.el.querySelector(`.fc-daygrid-day-top`);
            if (!box)
                return;
            const content = renderToString(this.constructor.ButtonWorklocationTemplate, this.headerTemplateProps(info.date));
            box.insertAdjacentHTML("beforeend", content);
        }
    },
    onDateClick(info){
        if (info.jsEvent && info.jsEvent.target.closest(".o_worklocation_btn")) {
            const date = DateTime.fromJSDate(info.date);
            this.handleWorkLocationClick(info.jsEvent.target, date);
        } else {
            super.onDateClick(...arguments)
        }
    },
    headerTemplateProps(date) {
        if (this.props.model.scale === "month") {
            return super.headerTemplateProps(date);
        }
        const parsedDate = DateTime.fromJSDate(date).toISODate();
        const multiCalendar = this.props.model.multiCalendar;
        const showLine = ["week", "month"].includes(this.props.model.scale);
        let worklocation = this.props.model.worklocations[parsedDate];
        const workLocationSetForCurrentUser =
            multiCalendar ?
            Object.keys(worklocation).some(key => worklocation[key].some(wlItem => wlItem.userId === user.userId)
            ) : worklocation?.userId === user.userId;

        let displayedWorkLocation = worklocation ? (JSON.parse(JSON.stringify(worklocation))) : {};
        // do not display the work locations of the current user if the user filter is not active
        if (multiCalendar && !this.props.model.data.userFilterActive) {
            for (let wl in worklocation){
                displayedWorkLocation[wl] = worklocation[wl].filter(wlItem => wlItem.userId !== user.userId);
            }
            displayedWorkLocation = Object.fromEntries(Object.entries(displayedWorkLocation).filter(([_, wlItems]) => wlItems.length !== 0));
        }

        return {
            ...super.headerTemplateProps(date),
            worklocation : displayedWorkLocation,
            workLocationSetForCurrentUser,
            multiCalendar,
            showLine,
            iconMap: {
                "office": "fa-building",
                "home": "fa-home",
            },
        }
    }
});


AttendeeCalendarRenderer.props = {
    ...AttendeeCalendarRenderer.props,
    openWorkLocationWizard: { type: Function, optional: true },
}
AttendeeCalendarCommonRenderer.props = {
    ...AttendeeCalendarCommonRenderer.props,
    openWorkLocationWizard: { type: Function, optional: true }
};

AttendeeCalendarCommonRenderer.WorklocationTemplate = "hr_homeworking_calendar.CalendarCommonRenderer.worklocation";
AttendeeCalendarCommonRenderer.ButtonWorklocationTemplate = "hr_homeworking_calendar.CalendarCommonRenderer.buttonWorklocation";
AttendeeCalendarCommonRenderer.headerTemplate = "hr_homeworking_calendar.CalendarCommonRendererHeader";

```

## File: static\src\calendar\common\calendar_common_renderer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>
    <t t-name="hr_homeworking_calendar.CalendarCommonRendererHeader" t-inherit="web.CalendarCommonRendererHeader" t-inherit-mode="primary">
        <xpath expr="//span[hasclass('o_cw_day_number')]" position="after">
            <t t-if="scale != 'month'" t-call="hr_homeworking_calendar.CalendarCommonRenderer.buttonWorklocation"/>
        </xpath>
    </t>

    <t t-name="hr_homeworking_calendar.CalendarCommonRenderer.buttonWorklocation">
        <t t-if="multiCalendar">
            <t t-call="hr_homeworking_calendar.MultiCalendarButtonWorkLocation"/>
        </t>
        <t t-else="">
            <t t-call="hr_homeworking_calendar.SingleCalendarButtonWorkLocation"/>
        </t>
    </t>

    <div t-name="hr_homeworking_calendar.MultiCalendarButtonWorkLocation" class="o_worklocation_btn w-100 d-flex align-items-center">
        <div class="o_homework_multi d-flex gap-1 ms-1">
            <t t-foreach="Object.keys(worklocation)" t-as="location" t-key="location">
                <t t-foreach="worklocation[location]" t-as="wl" t-key="wl.id">
                    <div class="o_homeworking_content wl_action" t-att-data-location="location" t-att-data-id="wl.id" t-att-data-employee="wl.employeeId">
                        <i t-attf-class="o_homework_content d-flex align-items-center justify-content-center flex-wrap border border-1 rounded-circle me-0 fa {{iconMap[wl.icon] || 'fa-map-marker'}} wl_color_{{wl.colorIndex}}"/>
                    </div>
                </t>
                <span class="fw-bolder" t-esc="worklocation[location][0].title"/>
            </t>
        </div>
        <button t-if="!workLocationSetForCurrentUser" class="o_worklocation_text bg-info bg-opacity-25 border-0 rounded-pill px-lg-2 py-0 text-nowrap opacity-0 opacity-100-hover wl_action" data-create="1">
            <i class="add_wl fa fa-map-marker me-lg-1" aria-hidden="true"></i>
        </button>
    </div>

    <div t-name="hr_homeworking_calendar.SingleCalendarButtonWorkLocation" class="o_worklocation_btn w-100 d-flex align-items-center">
        <div t-if="workLocationSetForCurrentUser" class="w-100 d-flex align-items-center wl_action o_homework_single">
            <t t-if="worklocation.display">
                <button t-attf-class='o_worklocation_text border-0 rounded-pill px-lg-2 py-0 text-nowrap wl_color_{{worklocation.colorIndex}}'>
                    <i t-attf-class="fa {{iconMap[worklocation.icon] || 'fa-map-marker'}} me-lg-1" aria-hidden="true"></i>
                    <span class="d-none d-lg-inline fw-bold" t-esc="worklocation.title"/>
                </button>
                <button t-if="showLine" t-attf-class="o_worklocation_line d-lg-block w-100 border-0 p-0 wl_color_{{worklocation.colorIndex}}"/>
            </t>
            <t t-else="">
                <button t-if="showLine" t-attf-class="o_worklocation_line d-lg-block w-100 border-0 p-0 wl_color_{{worklocation.colorIndex}}"/>
            </t>
        </div>
        <div t-else="" class="w-100 d-flex align-items-center opacity-0 opacity-100-hover wl_action" data-create="1">
            <button t-if="!workLocationSetForCurrentUser" class="o_worklocation_text bg-info bg-opacity-25 border-0 rounded-pill px-lg-2 py-0 text-nowrap">
                <i class="fa fa-map-marker me-lg-1" aria-hidden="true"></i>
                <span class="d-lg-inline fw-bold">Set Location</span>
            </button>
            <button t-if="showLine" class="o_worklocation_line d-lg-block w-100 bg-info bg-opacity-25 border-0 p-0"/>
        </div>
    </div>
</templates>

```

## File: static\src\calendar\common\calendar_model.js

```javascript
/** @odoo-module **/

import { AttendeeCalendarModel } from "@calendar/views/attendee_calendar/attendee_calendar_model";
import { serializeDateTime } from "@web/core/l10n/dates";
import { user } from "@web/core/user";
import { getColor } from "@web/views/calendar/colors";
import { patch } from "@web/core/utils/patch";

const { Interval } = luxon;

patch(AttendeeCalendarModel.prototype, {
    fetchEventLocation(data) {
        let attendeeIds;
        const filters = data.filterSections.partner_ids.filters;
        if (filters[filters.length - 1].type === "all" && filters[filters.length - 1].active) {
            attendeeIds = Object.keys(this.partnerColorMap);
        } else {
            attendeeIds = data.filterSections.partner_ids.filters
                .filter(filter => filter.type !== "all" && filter.value && filter.active)
                .map(filter => filter.value)
        }
        if (!attendeeIds.includes(user.partnerId)) {
            attendeeIds.push(user.partnerId);
        }
        return this.orm.call('res.partner', "get_worklocation", [
            attendeeIds,
            serializeDateTime(data.range.start),
            serializeDateTime(data.range.end),
        ]);
    },

    // returns a map of worklocations, display is used to mark the events that are to be shown in the view.
    async loadWorkLocations(data) {
        const res = await this.fetchEventLocation(data)
        this.multiCalendar = Object.values(res).some(location => location.user_id !== user.userId);
        const filters = data.filterSections.partner_ids.filters;
        data.userFilterActive = filters.filter(filter => filter.value === user.partnerId)[0]?.active || filters[filters.length - 1].type === "all" && filters[filters.length - 1].active;
        const events = {};
        let previousDay;
        const rangeInterval = Interval.fromDateTimes(data.range.start.startOf("day"), data.range.end.endOf("day")).splitBy({day: 1});
        for (const day of rangeInterval) {
            const startDay = day.s;
            const dayISO = startDay.toISODate();
            const dayName = startDay.setLocale("en").weekdayLong.toLowerCase();
            for (const employeeId in res) {
                if (this.multiCalendar) {
                    if (!(dayISO in events)) {
                        events[dayISO] = {};
                    }
                    if (res[employeeId].exceptions && dayISO in res[employeeId].exceptions) {
                        // check if exception for that date
                        const { location_type } = res[employeeId].exceptions[dayISO];
                        if (location_type in events[dayISO]) {
                            events[dayISO][location_type].push(this.createHomeworkingRecordAt(res[employeeId], startDay, res[employeeId].exceptions[dayISO]));
                        } else {
                            events[dayISO][location_type] = [this.createHomeworkingRecordAt(res[employeeId], startDay, res[employeeId].exceptions[dayISO])];
                        }
                    }
                    else {
                        const locationKeyName = `${dayName}_location_id`;
                        if (!(locationKeyName in res[employeeId])) {
                            continue;
                        }
                        const {location_type} = res[employeeId][locationKeyName];
                        if (!location_type) {
                            continue;
                        }
                        if (location_type in events[dayISO]) {
                            events[dayISO][location_type].push(this.createHomeworkingRecordAt(res[employeeId], startDay, res[employeeId][locationKeyName]));
                        } else {
                            events[dayISO][location_type] = [this.createHomeworkingRecordAt(res[employeeId], startDay, res[employeeId][locationKeyName])];
                        }
                    }
                } else {
                    const hasException = res[employeeId].exceptions && dayISO in res[employeeId].exceptions;
                    const workLocationData = hasException ? res[employeeId].exceptions[dayISO] : res[employeeId][`${dayName}_location_id`];
                    const currentEvent = this.createHomeworkingRecordAt(res[employeeId], startDay, workLocationData);
                    const previousEvent = events[previousDay];
                    if (previousEvent && previousEvent.icon === currentEvent.icon && previousEvent.title === currentEvent.title) {
                        previousEvent.end = previousEvent.end.plus({days:1});
                        currentEvent.display = false;
                    } else {
                        previousDay = dayISO;
                    }
                    if (currentEvent.title) {
                        events[dayISO] = currentEvent;
                    }
                }
            }
        }
        return events;
    },

    createHomeworkingRecordAt(record, day, workLocationData) {
        const { location_type, location_name, work_location_id, hr_employee_location_id} = workLocationData;
        const ghostRecord = !Boolean(hr_employee_location_id);
        const id = ghostRecord ? `default-location-${record.employee_id}-${day.toMillis()}` : String(hr_employee_location_id);
        return {
            id,
            title: location_name,
            start: day,
            end: day.plus({days:1}),
            display: true,
            multiCalendar: this.multiCalendar,
            homeworking: true,
            employeeId: record.employee_id,
            employeeName: record.employee_name,
            icon: location_type,
            userId: record.user_id,
            partnerId: record.partner_id,
            colorIndex: this.partnerColorMap[record.partner_id],
            resModel: "hr.employee.location",
            work_location_id,
            ghostRecord,
            rawRecord: record,
        };
    },

    get worklocations() {
        return this.data.worklocations;
    },

    mapPartnersToColor(data) {
        return data.filterSections.partner_ids.filters
            .filter(filter => filter.type !== "all" && filter.value)
            .reduce((map, partner) => ({ ...map, [partner.value]: getColor(partner.colorIndex)}), {})
    },

    /**
     * @override
     */
    async updateData(data){
        await super.updateData(...arguments)
        this.partnerColorMap = this.mapPartnersToColor(data);
        data.worklocations = await this.loadWorkLocations(data);
    }
})

```

## File: static\src\calendar\common\hr_homeworking_calendar_controller.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { patch } from "@web/core/utils/patch";
import { useService } from "@web/core/utils/hooks";
import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { AttendeeCalendarController } from "@calendar/views/attendee_calendar/attendee_calendar_controller"
import { serializeDate} from "@web/core/l10n/dates";

patch(AttendeeCalendarController.prototype, {
    setup() {
        super.setup();
        this.action = useService("action");
    },
    async editRecord(record, context = {}, shouldFetchFormViewId = true) {
        if (record.homeworking && 'start' in record) {
            return this.action.doAction('hr_homeworking_calendar.set_location_wizard_action', {
                additionalContext: {
                    'default_date': serializeDate(record.start),
                    'default_work_location_id' : record.work_location_id,
                    'dialog_size': 'medium',
                },
                onClose: async (closeInfo) => {
                    this.model.load()
                },
            });
        }
        return super.editRecord(...arguments)
    },
    deleteRecord(record) {
        if (record.id && record.homeworking) {
            if (record.ghostRecord) {
                this.displayDialog(ConfirmationDialog, {
                    title: _t("Confirmation"),
                    body: _t("Are you sure you want to delete this location?"),
                    confirm: async () => {
                        const dayName = record.start.setLocale("en").weekdayLong.toLowerCase();
                        const locationField = `${dayName}_location_id`;
                        await this.orm.write('res.users', [record.rawRecord.user_id], {[locationField]: false})
                        this.model.load();
                    },
                    cancel: () => {
                    },
                });
            } else {
                this.displayDialog(ConfirmationDialog, {
                    title: _t("Confirmation"),
                    body: _t("Are you sure you want to delete this exception?"),
                    confirm: async () => {
                        await this.orm.unlink("hr.employee.location", [parseInt(record.id)]);
                        this.model.load();
                    },
                    cancel: () => {
                    },
                });
            }
        } else {
            super.deleteRecord(...arguments)
        }
    },
    openWorkLocationWizard(startDate) {
        this.action.doAction('hr_homeworking_calendar.set_location_wizard_action',{
            additionalContext: {
                'default_date': serializeDate(startDate),
                'dialog_size': 'medium',
            },
            onClose: async () => {
                this.model.load()
            },
        })
    },
    get rendererProps() {
        return {
            ...super.rendererProps,
            openWorkLocationWizard: (date) => this.openWorkLocationWizard(date),
        }
    },
})

```

## File: static\src\calendar\common\popover\calendar_common_popover.js

```javascript
import { onWillStart } from "@odoo/owl";
import { user } from "@web/core/user";
import { patch } from "@web/core/utils/patch";
import { AttendeeCalendarCommonPopover } from "@calendar/views/attendee_calendar/common/attendee_calendar_common_popover";
import { Field } from "@web/views/fields/field"

export const patchAttendeeCalendarCommonPopoverClass = {
    template: "homework.AttendeeCalendarCommonPopover",
    subTemplates: {
        ...AttendeeCalendarCommonPopover.subTemplates,
        body: "homework.AttendeeCalendarCommonPopover.body",
        footer: "homework.AttendeeCalendarCommonPopover.footer",
    },
    components: {
        ...AttendeeCalendarCommonPopover.components,
        Field,
    }
}

export const patchAttendeeCalendarCommonPopover = {
    setup() {
        this.fieldNames = ["work_location_id", "work_location_name", "work_location_type", "employee_id", "weekday", "weekly", "start_date", "employee_name"];
        super.setup(...arguments);
        this.values = {
            work_location_id: this.props.record.work_location_id,
            work_location_name: this.props.record.title,
            work_location_type: this.props.record.icon,
            employee_id: this.props.record.employeeId,
            weekday: false,
            weekly: false,
            date: this.props.record.start,
            employee_name: this.props.record.employeeName,
        };
        this.fields = {
            "work_location_id": { name: "Work Location", type: "many2one", relation: "hr.work.location"},
            "work_location_name": { name: "Work Location Name", type: "char"},
            "work_location_type": { name: "work location type", type: "selection"},
            "employee_id": { name: "employee id", type: "many2one", relation: "hr.employee"},
            "weekday": { name: "weekday", type: "integer"},
            "weekly": { name: "weekly", type: "boolean"},
            "date": { name: "date", type: "date"},
            "employee_name": { name: "employee name", type:"char"}
        };
        onWillStart(async () => {
            this.userCanEdit = (await this.orm.read("res.users", [user.userId], ["can_edit"]))[0]['can_edit'];
        });

    },
    isWorkLocationEvent(){
        return this.props.record['resModel'] === 'hr.employee.location';
    },
    get hasFooter() {
        return !this.isWorkLocationEvent() || this.props.record.userId === user.userId
    },
    isCurrentUserIsOwnerWorklocation(){
        return this.isWorkLocationEvent() && this.props.record.userId === user.userId;
    },
    get isEventEditable() {
        return ('resModel' in this.props.record) || super.isEventEditable;
    },
    get isEventViewable() {
        return !('resModel' in this.props.record) || super.isEventViewable;
    },
    get isEventDeletable() {
        if (this.props.record.homeworking) {
            return (this.userCanEdit || !this.props.record.ghostRecord) && super.isEventDeletable
        }
        return super.isEventDeletable;
    },
    get displayAttendeeAnswerChoice() {
        return !('resModel' in this.props.record) && super.displayAttendeeAnswerChoice;
    },
    get isCurrentUserAttendee() {
        return !('resModel' in this.props.record) && super.isCurrentUserAttendee;
    },
}
export const unpatchAttendeeCalendarCommonPopoverClass = patch(AttendeeCalendarCommonPopover, patchAttendeeCalendarCommonPopoverClass);

export const unpatchAttendeeCalendarCommonPopover = patch(AttendeeCalendarCommonPopover.prototype, patchAttendeeCalendarCommonPopover)

```

## File: static\src\calendar\common\popover\calendar_common_popover.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="homework.AttendeeCalendarCommonPopover">
        <t t-if="isWorkLocationEvent()">
        <Record resModel="'hr.employee.location'" fieldNames="fieldNames" fields="fields" activeFields="fields" values="values" mode="'readonly'" t-slot-scope="slot">
            <Dialog t-if="env.isSmall" title="props.record.title" contentClass="'o_calendar_color_'+ props.record.colorIndex">
                <t t-call="{{ constructor.subTemplates.body }}"/>
                <t t-set-slot="footer">
                    <t t-call="calendar.AttendeeCalendarCommonPopover.footer"/>
                </t>
            </Dialog>
            <div t-else="" class="card-header d-flex justify-content-between py-2 pe-2">
                <span class="popover-header border-0">
                    <t t-if="slot.record.data.work_location_type === 'office'">
                        <i class="fa fa-fw fa-building me-2" role="img" t-att-class="`wl_color_${props.record.colorIndex}`"/>
                    </t>
                    <t t-elif="slot.record.data.work_location_type === 'home'">
                        <i class="fa fa-fw fa-home me-2" role="img" />
                    </t>
                    <t t-else="">
                    <i class="fa fa-fw fa-map-marker me-2" t-att-class="`wl_color_${props.record.colorIndex}`" role="img"/>
                    </t>
                    <t t-esc="props.record.title"/>
                </span>
                <span class="o_cw_popover_close ms-4 mt-2 me-2" t-on-click.stop="() => props.close()">
                    <i class="fc-close fc-icon fc-icon-x" />
                </span>
            </div>
            <div class="o_cw_body">
                <t t-call="{{ constructor.subTemplates.body }}" />
                <div class="card-footer border-top d-flex gap-1" t-att-class="{ 'o_footer_shrink': !hasFooter }">
                    <t t-call="{{ constructor.subTemplates.footer }}" />
                </div>
            </div>
        </Record>
        </t>
        <t t-else="">
            <t t-call="web.CalendarCommonPopover" />
        </t>
    </t>
    <t t-name="homework.AttendeeCalendarCommonPopover.body">
        <t t-if="isWorkLocationEvent()">
                <ul class="list-group list-group-flush">
                    <div class="list-group-item">
                        <i class="fa fa-fw fa-calendar me-2" t-att-class="`wl_color_${props.record.colorIndex}`"/>
                        <t t-if="props.record.start.ts != props.record.end.ts">
                            <t t-esc="props.record.start.toFormat('d MMMM')" t-options="{'widget': 'date'}"/>
                        </t>
                        <t t-else="">
                            <t t-esc="props.record.start.toFormat('d MMMM yyyy')" t-options="{'widget': 'date'}"/>
                        </t>
                    </div>
                    <div class="list-group-item">
                        <i class="fa fa-fw fa-user me-2" t-att-class="`wl_color_${props.record.colorIndex}`"/>
                        <Field name="'employee_name'" record="slot.record" readonly="true"/>
                    </div>
                </ul>
        </t>
        <t t-else="">
            <t t-call="calendar.AttendeeCalendarCommonPopover.body" />
        </t>
    </t>

    <t t-name="homework.AttendeeCalendarCommonPopover.footer">
        <t t-if="isWorkLocationEvent()">
            <t t-if="isCurrentUserIsOwnerWorklocation()">
                <t t-call="calendar.AttendeeCalendarCommonPopover.footer" />
            </t>
        </t>
        <t t-else="">
            <t t-call="calendar.AttendeeCalendarCommonPopover.footer" />
        </t>
    </t>
</templates>

```

## File: wizard\homework_location_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api

from odoo.addons.hr_homeworking.models.hr_homeworking import DAYS

class HomeworkLocationWizard(models.TransientModel):
    _name = 'homework.location.wizard'
    _description = 'Set Homework Location Wizard'

    work_location_id = fields.Many2one('hr.work.location', required=True, string="Location")
    work_location_name = fields.Char(related='work_location_id.name', string="Location name")
    work_location_type = fields.Selection(related="work_location_id.location_type")
    employee_id = fields.Many2one('hr.employee', default=lambda self: self.env.user.employee_id, required=True, ondelete="cascade")
    employee_name = fields.Char(related="employee_id.name")
    user_can_edit = fields.Boolean(compute='_compute_user_can_edit')
    weekly = fields.Boolean(default=False)
    date = fields.Date(string="Date")
    day_week_string = fields.Char(compute="_compute_day_week_string")

    @api.depends('date')
    def _compute_day_week_string(self):
        for record in self:
            record.day_week_string = record.date.strftime("%A")

    @api.depends('date')
    def _compute_user_can_edit(self):
        self.user_can_edit = self.env.user.can_edit

    def set_employee_location(self):
        self.ensure_one()
        default_employee_id = self.env.context.get('default_employee_id') or self.env.user.employee_id.id
        employee_id = self.env['hr.employee'].browse(self.employee_id.id or default_employee_id)
        employee_location = self.env['hr.employee.location'].search([
            ('date', '=', self.date),
            ('employee_id', '=', employee_id.id)
        ])
        weekday = self.date.weekday()
        default_location_for_current_date = DAYS[weekday]
        if self.weekly:
            # delete any exceptions on the current date
            if employee_location:
                employee_location.unlink()
            employee_id.sudo().user_id.write({
                default_location_for_current_date: self.work_location_id.id,
            })
        else:
            # check if work_location_id is the same as the default one for that day
            if self.work_location_id.id == employee_id[default_location_for_current_date].id:
                employee_location.unlink()
            # check if worklocation is set for that employee that day
            elif employee_location:
                employee_location.write({
                    'date': self.date,
                    'employee_id': employee_id.id,
                    'work_location_id': self.work_location_id.id
                })
            else:
                self.env['hr.employee.location'].create({
                    'date': self.date,
                    'employee_id': employee_id.id,
                    'work_location_id': self.work_location_id.id
                })

```

## File: wizard\homework_location_wizard.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    <record id="homework_location_wizard_view_form" model="ir.ui.view">
        <field name="name">homework.location.wizard.view.form</field>
        <field name="model">homework.location.wizard</field>
        <field name="arch" type="xml">
            <form string="Location">
                <group>
                    <label for="date"/>
                    <div class="o_row w-100">
                        <field name="date"/>
                    </div>
                    <label for="weekly" invisible="not user_can_edit"/>
                    <div class="o_row w-100" invisible="not user_can_edit">
                        <field name="weekly" class="oe_inline"/>
                        <span class="w-100">
                            Repeat every <field name="day_week_string" class="oe_inline"/>
                        </span>
                    </div>
                    <field name="work_location_id"/>
                </group>
                <footer>
                    <button name="set_employee_location" type="object" class="btn-primary" string="Set Location" />
                </footer>
            </form>
        </field>
    </record>

    <record id="set_location_wizard_action" model="ir.actions.act_window">
        <field name="name">Set Location</field>
        <field name="res_model">homework.location.wizard</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="homework_location_wizard_view_form"/>
        <field name="binding_model_id" ref="model_homework_location_wizard"/>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import homework_location_wizard

```

