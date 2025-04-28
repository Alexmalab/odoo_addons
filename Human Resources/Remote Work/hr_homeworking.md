# Odoo Module: hr_homeworking

Category: Human Resources/Remote Work

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Remote Work',
    'version': '1.0',
    'category': 'Human Resources/Remote Work',
    'depends': ['hr', 'calendar'],
    'data': [
        'security/security.xml',
        'security/ir.model.access.csv',
        'views/hr_employee_views.xml',
        'views/res_users.xml',
        'wizard/homework_location_wizard.xml',
    ],
    'demo': [
        'data/hr_homeworking_demo.xml',
    ],
    'installable': True,
    'assets': {
        'web.assets_backend': [
            'hr_homeworking/static/src/**/*',
        ],
        'web.qunit_suite_tests': [
            'hr_homeworking/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\hr_homeworking_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="work_location_3" model="hr.work.location">
            <field name="name">GR2</field>
            <field name="location_type">office</field>
            <field name="address_id" ref="base.main_partner"/>
        </record>
    </data>
</odoo>

```

## File: models\hr_employee.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import _, api, fields, models
from odoo.tools import DEFAULT_SERVER_DATE_FORMAT

from .hr_homeworking import DAYS


class HrEmployeeBase(models.AbstractModel):
    _inherit = "hr.employee.base"

    monday_location_id = fields.Many2one('hr.work.location', string='Monday')
    tuesday_location_id = fields.Many2one('hr.work.location', string='Tuesday')
    wednesday_location_id = fields.Many2one('hr.work.location', string='Wednesday')
    thursday_location_id = fields.Many2one('hr.work.location', string='Thursday')
    friday_location_id = fields.Many2one('hr.work.location', string='Friday')
    saturday_location_id = fields.Many2one('hr.work.location', string='Saturday')
    sunday_location_id = fields.Many2one('hr.work.location', string='Sunday')
    exceptional_location_id = fields.Many2one(
        'hr.work.location', string='Current',
        compute='_compute_exceptional_location_id',
        help='This is the exceptional, non-weekly, location set for today.')
    hr_icon_display = fields.Selection(selection_add=[('presence_home', 'At Home'),
                                                      ('presence_office', 'At Office'),
                                                      ('presence_other', 'At Other')])
    name_work_location_display = fields.Char(compute="_compute_name_work_location_display")
    today_location_name = fields.Char()

    @api.model
    def _get_current_day_location_field(self):
        return DAYS[fields.Date.today().weekday()]

    # hack to allow groupby on today's location. Since there are 7 different fields, we have to use a placeholder
    # in the search view and replace it with the correct field every time the views are fetched.
    @api.model
    def get_views(self, views, options=None):
        res = super().get_views(views, options)
        dayfield = self._get_current_day_location_field()
        if 'search' in res['views']:
            res['views']['search']['arch'] = res['views']['search']['arch'].replace('today_location_name', dayfield)
        if 'list' in res['views']:
            res['views']['list']['arch'] = res['views']['list']['arch'].replace('name_work_location_display', dayfield)
        return res

    @api.depends('exceptional_location_id')
    def _compute_name_work_location_display(self):
        dayfield = self._get_current_day_location_field()
        unspecified = _('Unspecified')
        for employee in self:
            current_location_id = employee.exceptional_location_id or employee[dayfield]
            employee.name_work_location_display = current_location_id.name if current_location_id else unspecified

    def _compute_exceptional_location_id(self):
        today = fields.Date.today()
        current_employee_locations = self.env['hr.employee.location'].search([
            ('employee_id', 'in', self.ids),
            ('date', '=', today),
        ])
        employee_work_locations = {l.employee_id.id: l.work_location_id for l in current_employee_locations}

        for employee in self:
            employee.exceptional_location_id = employee_work_locations.get(employee.id, False)

    @api.depends(*DAYS, 'exceptional_location_id')
    def _compute_presence_icon(self):
        super()._compute_presence_icon()
        dayfield = self._get_current_day_location_field()
        for employee in self:
            today_employee_location_id = employee.exceptional_location_id or employee[dayfield]
            if not today_employee_location_id or employee.hr_icon_display.startswith('presence_holiday'):
                continue
            employee.hr_icon_display = f'presence_{today_employee_location_id.location_type}'
            employee.show_hr_icon_display = True

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

## File: models\hr_homeworking.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models

DAYS = ['monday_location_id', 'tuesday_location_id', 'wednesday_location_id', 'thursday_location_id', 'friday_location_id', 'saturday_location_id', 'sunday_location_id']


class HrEmployeeLocation(models.Model):
    _name = "hr.employee.location"
    _description = "Employee Location"

    work_location_id = fields.Many2one('hr.work.location', required=True, string="Location")
    work_location_name = fields.Char(related='work_location_id.name', string="Location name")
    work_location_type = fields.Selection(related="work_location_id.location_type")
    employee_id = fields.Many2one('hr.employee', default=lambda self: self.env.user.employee_id, required=True, ondelete="cascade")
    employee_name = fields.Char(related="employee_id.name")
    date = fields.Date(string="Date")
    day_week_string = fields.Char(compute="_compute_day_week_string")

    _sql_constraints = [
        ('uniq_exceptional_per_day', 'unique(employee_id, date)', _('Only one default work location and one exceptional work location per day per employee.')),
    ]

    @api.depends('date')
    def _compute_day_week_string(self):
        for record in self:
            record.day_week_string = record.date.strftime("%A")

```

## File: models\hr_work_location.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, models
from odoo.exceptions import UserError
from odoo.addons.hr_homeworking.models.hr_homeworking import DAYS

class WorkLocation(models.Model):
    _inherit = "hr.work.location"

    @api.ondelete(at_uninstall=False)
    def _unlink_except_used_by_employee(self):
        domains = [(day, 'in', self.ids) for day in DAYS]
        employee_uses_location = self.env['hr.employee'].search_count(domains, limit=1)
        if employee_uses_location:
            raise UserError(_("You cannot delete locations that are being used by your employees"))
        exceptions_using_location = self.env['hr.employee.location'].search([('work_location_id', 'in', self.ids)])
        exceptions_using_location.unlink()

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

class ResPartner(models.Model):
    _inherit = 'res.partner'

    def _compute_im_status(self):
        super()._compute_im_status()
        for user in self.user_ids:
            dayfield = self.env['hr.employee']._get_current_day_location_field()
            location_type = user[dayfield].location_type
            if not location_type:
                continue
            im_status = user.partner_id.im_status
            if im_status == "online" or im_status == "away" or im_status == "offline":
                user.partner_id.im_status = location_type + "_" + im_status

    def get_worklocation(self, start_date, end_date):
        employee_id = self.env['hr.employee'].search([
            ('work_contact_id', 'in', self.ids),
            ('company_id', '=', self.env.company.id)])
        return employee_id._get_worklocation(start_date, end_date)

```

## File: models\res_users.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields

from .hr_homeworking import DAYS

class User(models.Model):
    _inherit = ['res.users']

    monday_location_id = fields.Many2one("hr.work.location", related="employee_id.monday_location_id", readonly=False, string='Monday')
    tuesday_location_id = fields.Many2one("hr.work.location", related="employee_id.tuesday_location_id", readonly=False, string='Tuesday')
    wednesday_location_id = fields.Many2one("hr.work.location", related="employee_id.wednesday_location_id", readonly=False, string='Wednesday')
    thursday_location_id = fields.Many2one("hr.work.location", related="employee_id.thursday_location_id", readonly=False, string='Thursday')
    friday_location_id = fields.Many2one("hr.work.location", related="employee_id.friday_location_id", readonly=False, string='Friday')
    saturday_location_id = fields.Many2one("hr.work.location", related="employee_id.saturday_location_id", readonly=False, string='Saturday')
    sunday_location_id = fields.Many2one("hr.work.location", related="employee_id.sunday_location_id", readonly=False, string='Sunday')

    def _get_employee_fields_to_sync(self):
        return super()._get_employee_fields_to_sync() + DAYS

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + DAYS

    @property
    def SELF_WRITEABLE_FIELDS(self):
        return super().SELF_WRITEABLE_FIELDS + DAYS

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_employee
from . import res_partner
from . import res_users
from . import hr_homeworking
from . import hr_work_location

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_employee_location,hr.employee.location,model_hr_employee_location,hr.group_hr_user,1,1,1,1
access_user_employee_location,hr.employee.location,model_hr_employee_location,base.group_user,1,1,1,1
access_homework_location_wizard,homework.location.wizard,model_homework_location_wizard,base.group_user,1,1,1,1

```

## File: security\security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="homeworking_own_rule" model="ir.rule">
            <field name="name">homeworking: own</field>
            <field name="model_id" ref="model_hr_employee_location"/>
            <field name="groups" eval="[(4, ref('base.group_user'))]"/>
            <field name="domain_force">[
                ('employee_id', '=', user.employee_id.id)
            ]</field>
        </record>

        <record id="homeworking_admin_rule" model="ir.rule">
            <field name="name">homeworking: admin</field>
            <field name="model_id" ref="model_hr_employee_location"/>
            <field name="groups" eval="[(4, ref('hr.group_hr_user'))]"/>
            <field name="domain_force">[(1, '=', 1)]</field>
        </record>

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

## File: static\src\im_status_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.ImStatus" t-inherit-mode="extension">
        <xpath expr="//*[@name='icon']" position="replace">
            <t t-if="props.persona.im_status">
                <t t-if="props.persona.im_status.split('_').length == 2">
                    <t t-set="location" t-value="props.persona.im_status.split('_')[0]"/>
                    <t t-if="location == 'home' || location == 'office' || location == 'other'">
                        <t t-set="status" t-value="props.persona.im_status.split('_')[1]"/>
                        <t t-set="location_map" t-value="{'home': 'fa-home', 'office': 'fa-building', 'other': 'fa-map-marker'}"/>
                        <t t-set="status_map" t-value="{'online': 'text-success', 'away': 'o-away', 'offline': 'text-7000'}"/>
                        <t t-set="icon" t-value="location_map[location]"/>
                        <t t-set="text" t-value="status_map[status]"/>
                        <i class="fa" t-attf-class="{{icon}} {{text}}" t-attf-title="{{status}}" role="img" t-attf-aria-label=" User work at {{location}} and is {{status}}"/>
                    </t>
                    <t t-else="">$0</t>
                </t>
                <t t-else="">$0</t>
            </t>
            <t t-else="">$0</t>
        </xpath>
    </t>

    <t t-inherit="mail.ThreadIcon" t-inherit-mode="extension">
        <xpath expr="//*[@name='chat_static']" position="replace">
            <t t-if="chatPartner.im_status">
                <t t-if="chatPartner.im_status.split('_').length >= 2">
                    <t t-set="location" t-value="chatPartner.im_status.split('_')[0]"/>
                    <t t-if="location == 'home' || location == 'office' || location == 'other'">
                        <t t-set="status" t-value="chatPartner.im_status.split('_')[1]"/>
                        <t t-set="location_map" t-value="{'home': 'fa-home', 'office': 'fa-building', 'other': 'fa-map-marker'}"/>
                        <t t-set="status_map" t-value="{'online': 'text-success', 'away': 'o-away', 'offline': 'text-7000'}"/>
                        <t t-set="icon" t-value="location_map[location]"/>
                        <t t-set="text" t-value="status_map[status]"/>
                        <i class="fa" t-attf-class="{{icon}} {{text}}" t-attf-title="{{status}}" role="img" t-attf-aria-label=" User work at {{location}} and is {{status}}"/>
                    </t>
                    <t t-else="">$0</t>
                </t>
                <t t-else="">$0</t>
            </t>
            <t t-else="">$0</t>
        </xpath>
    </t>
</templates>
```

## File: static\src\calendar\hr_homeworking_calendar_controller.js

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
        super.setup()
        this.action = useService("action");
    },
    async editRecord(record, context = {}, shouldFetchFormViewId = true) {
        if (record.homeworking && 'start' in record) {
            return this.action.doAction('hr_homeworking.set_location_wizard_action',{
                name: _t("Edit Record"),
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
    createRecord(record) {
        if (record.homework) {
            return this.action.doAction('hr_homeworking.set_location_wizard_action',{
                name: _t("Create Record"),
                additionalContext: {
                    'default_date': serializeDate(record.start),
                    'dialog_size': 'medium',
                },
                onClose: async (closeInfo) => {
                    this.model.load()
                },
            });
        } else {
            return super.createRecord(record);
        }
    },
    deleteRecord(record) {
        if (record.id && record.homeworking) {
            if (record.ghostEvent) {
                this.displayDialog(ConfirmationDialog, {
                    title: _t("Confirmation"),
                    body: _t("Are you sure you want to delete this location?"),
                    confirm: () => {
                        const dayName = record.start.setLocale("en").weekdayLong.toLowerCase();
                        const locationField = `${dayName}_location_id`;
                        this.orm.call('hr.employee', "write", [
                            [record.rawRecord.employee_id],
                            {[locationField]: false}
                        ]);
                        this.model.load();
                    },
                    cancel: () => {
                    },
                });
            } else {
                this.displayDialog(ConfirmationDialog, {
                    title: _t("Confirmation"),
                    body: _t("Are you sure you want to delete this exception?"),
                    confirm: () => {
                        this.orm.call('hr.employee.location', "unlink", [
                            record.id,
                        ]);
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
})

```

## File: static\src\calendar\common\calendar_common_renderer.js

```javascript
/** @odoo-module **/

import { AttendeeCalendarCommonRenderer } from "@calendar/views/attendee_calendar/common/attendee_calendar_common_renderer";
import { patch } from "@web/core/utils/patch";
import { renderToString } from "@web/core/utils/render";

const { DateTime } = luxon;

patch(AttendeeCalendarCommonRenderer.prototype, {
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
        };
    },
    fcEventToRecord(event) {
        const res = super.fcEventToRecord(...arguments);
        res.homework = event.homework;
        return res;
    },
    filterWorkRecord(arr, modelUserId){
        let events = []
        if (arr.length === 1){
            events.push(this.convertWorkRecordToEvent(arr[0]))
            return events;
        }
        // sort ASC by title location. If it's equals put the event of the current user at first
        arr.sort((a,b)=> (a.title > b.title) ? 1 : (a.title < b.title) ? -1 : a.userId === modelUserId ? -1 : b.userId === modelUserId ? 1 : 0)
        let previous_title = "";
        for (const event of arr) {
            if (previous_title !== event.title) {
                events.push(this.convertWorkRecordToEvent(event))
                previous_title = event.title
            }
        }
        return events;
    },
    mapRecordsToEvents() {
        let event = super.mapRecordsToEvents(...arguments);
        let work = [];
        if (this.props.model.multiCalendar) {
            for (const day in this.props.model.worklocations) {
                let events = [];
                const home = this.props.model.worklocations[day].home;
                const office = this.props.model.worklocations[day].office;
                const other = this.props.model.worklocations[day].other;
                const modelUserId = this.env.searchModel.userService.userId;
                if (home && home.length > 0) {
                    events = events.concat(this.filterWorkRecord(home,modelUserId))
                }
                if (office && office.length > 0) {
                    events = events.concat(this.filterWorkRecord(office,modelUserId))
                }
                if (other && other.length > 0) {
                    events = events.concat(this.filterWorkRecord(other,modelUserId))
                }
                work.push(...events);
            }
        } else {
            work = Object.values(this.props.model.worklocations).reduce((wls, wl) => {
                if (wl.display) {
                    wls.push(this.convertWorkRecordToEvent(wl));
                }
                return wls;
            }, []);
        }
        return event.concat(work);
    },
    convertWorkRecordToEvent(record) {
        return {
            id: record.id,
            title: record.title,
            start: record.start.toISO(),
            end: record.end.plus({ days: 1}).toISO(),
            allDay: record.isAllDay,
            icon: record.icon,
            colorIndex: record.colorIndex,
            worklocation: true,
            // to avoid to drag location event
            editable: false,
        };
    },
    onDayRender(info){
        const parsedDate = DateTime.fromJSDate(info.date).toISODate();
        if (this.props.model.scale === 'week' || this.props.model.scale === 'day'){
            const button = info.view.context.calendar.el.querySelector(`.fc-day-header[data-date='${parsedDate}'] .o_worklocation_text`)
            const line = info.view.context.calendar.el.querySelector(`.fc-day-header[data-date='${parsedDate}'] .o_worklocation_line`)
            if (!button || !line)
                return;
            info.homework = true;
            button.onclick = () => this.props.createRecord(this.fcEventToRecord(info));
            line.onclick = () => this.props.createRecord(this.fcEventToRecord(info));
        }
        if (this.props.model.scale === 'month'){
            const box = info.view.el.querySelector(`.fc-day-top[data-date='${parsedDate}']`)
            if (!box)
                return;
            const content = renderToString(this.constructor.ButtonWorklocationTemplate, {})
            const {children } = new DOMParser().parseFromString(content, "text/html").body
            box.appendChild(...children);
            info.homework = true;
        }
        super.onDayRender(...arguments);
    },
    onEventRender(info) {
        const { el, event } = info;
        if (event.extendedProps.worklocation) {
            el.classList.add("o_homework_event");
            const multiCalendar = this.props.model.multiCalendar;
            let injectedContentStr = "";
            const icon = event.extendedProps.icon;
            if (multiCalendar) {
                const parsedDate = DateTime.fromJSDate(info.event.start).toISODate()
                const records = this.props.model.worklocations[parsedDate][icon].filter((rec) => rec.title === event.title);
                let iconStr;
                if (icon === "home") {
                    iconStr = "fa-home";
                }
                else if (icon === "office") {
                    iconStr = "fa-building";
                }
                else {
                    iconStr = "fa-map-marker";
                }
                if (records) {
                    const obj = {records: records, iconStr: iconStr, multiCalendar: multiCalendar};
                    injectedContentStr = renderToString(this.constructor.WorklocationTemplate, obj);
                }
            } else {
                const record = this.props.model.worklocations[DateTime.fromJSDate(info.event.start).toISODate()];
                if (record) {
                    injectedContentStr = renderToString(this.constructor.WorklocationTemplate, record);
                }
            }
            const domParser = new DOMParser();
            const { children } = domParser.parseFromString(injectedContentStr, "text/html").body;
            el.querySelector(".fc-content").replaceWith(...children);
        } else {
            super.onEventRender(...arguments);
        }
    },
    onDblClick(info) {
        if (info.event.extendedProps.worklocation) {
            this.onClick(info);
        } else {
            super.onDblClick(...arguments);
        }
    },
    onClick(info){
        if (info.event.extendedProps.worklocation){
            const elems = document.elementsFromPoint(info.jsEvent.x, info.jsEvent.y)
            const dayElement = elems.find((elem) => elem.classList.contains("fc-day","fc-widget-content"))
            const dayFromElement = dayElement.getAttribute("data-date")
            let workLocation;
            if (this.props.model.multiCalendar) {
                const elem = elems.find((elem) => elem.classList.contains("o_homework_content"))
                if (!elem){
                    return;
                }
                const id = elem.getAttribute("data-id");
                const icon = info.event.extendedProps.icon;
                workLocation = this.props.model.worklocations[dayFromElement][icon].find((wl) => wl.id === id);
            } else {
                workLocation = Object.values(this.props.model.worklocations).find(wl => wl.start.toISODate() === dayFromElement);
            }
            this.openPopover(info.el, workLocation);
            this.highlightEvent(info.event, "o_cw_custom_highlight");
        } else {
            super.onClick(...arguments);
        }
    },
    onDateClick(info){
        if (info.jsEvent.target.closest(".o_worklocation_btn")) {
            info.homework = true
            this.props.createRecord(this.fcEventToRecord(info));
        } else {
            super.onDateClick(...arguments)
        }
    }
});

AttendeeCalendarCommonRenderer.WorklocationTemplate = "hr.homeworking.CalendarCommonRenderer.worklocation";
AttendeeCalendarCommonRenderer.ButtonWorklocationTemplate = "hr.homeworking.CalendarCommonRenderer.buttonWorklocation";
AttendeeCalendarCommonRenderer.headerTemplate = "hr_homeworking.CalendarCommonRendererHeader";

```

## File: static\src\calendar\common\calendar_model.js

```javascript
/** @odoo-module **/

import { AttendeeCalendarModel } from "@calendar/views/attendee_calendar/attendee_calendar_model";
import { serializeDateTime } from "@web/core/l10n/dates";
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
        return this.orm.call('res.partner', "get_worklocation", [
            attendeeIds,
            serializeDateTime(data.range.start),
            serializeDateTime(data.range.end),
        ]);
    },

    // returns a map of worklocations, display is used to mark the events that are to be shown in the view.
    async loadWorkLocations(data) {
        const res = await this.fetchEventLocation(data)
        this.multiCalendar = Object.keys(res).length > 1;
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
                            events[dayISO][location_type].push(this.createHomeworkingEventAt(res[employeeId], startDay, res[employeeId].exceptions[dayISO]));
                        } else {
                            events[dayISO][location_type] = [this.createHomeworkingEventAt(res[employeeId], startDay, res[employeeId].exceptions[dayISO])];
                        }
                    }
                    else {
                        const locationKeyName = `${dayName}_location_id`;
                        if (!(locationKeyName in res[employeeId])) {
                            continue;
                        }
                        const {location_type} = res[employeeId][locationKeyName];
                        if (location_type in events[dayISO]) {
                            events[dayISO][location_type].push(this.createHomeworkingEventAt(res[employeeId], startDay, res[employeeId][locationKeyName]));
                        } else {
                            events[dayISO][location_type] = [this.createHomeworkingEventAt(res[employeeId], startDay, res[employeeId][locationKeyName])];
                        }
                    }
                } else {
                    const hasException = res[employeeId].exceptions && dayISO in res[employeeId].exceptions;
                    const workLocationData = hasException ? res[employeeId].exceptions[dayISO] : res[employeeId][`${dayName}_location_id`];
                    const currentEvent = this.createHomeworkingEventAt(res[employeeId], startDay, workLocationData);
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

    createHomeworkingEventAt(record, day, workLocationData) {
        const { location_type, location_name, work_location_id, hr_employee_location_id} = workLocationData;
        const ghostEvent = !Boolean(hr_employee_location_id);
        const id = ghostEvent ? `default-location-${record.employee_id}-${day.toMillis()}` : String(hr_employee_location_id);
        return {
            id,
            title: location_name,
            isAllDay: true,
            duration : 1,
            start: day,
            end: day.plus({hours: 1}),
            isHatched: false,
            isStriked: false,
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
            ghostEvent,
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

## File: static\src\calendar\common\calender_common_renderer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<templates>
    <t t-name="hr.homeworking.CalendarCommonRenderer.worklocation">
        <t t-if="multiCalendar">
            <div class="o_homework_multi d-flex align-items-center gap-1">
                <t t-foreach="records" t-as="record" t-key="record.id">
                    <div class="o_homeworking_content">
                        <i class="o_homework_content d-flex align-items-center justify-content-center flex-wrap border border-1 rounded-circle me-0" t-att-class="`fa ${iconStr} wl_color_${record.colorIndex}`" t-att-data-id="record.id"/>
                    </div>
                </t>
                <span class="fw-bolder" t-esc="records[0].title"/>
            </div>
        </t>
        <t t-else="">
            <div class="o_homework_single d-flex align-items-center w-100">
                <div class="o_homeworking_content overflow-visible opacity-100 rounded-pill m-0 px-2 text-nowrap" t-att-class="`wl_color_${colorIndex}`">
                    <t t-if="icon === 'office'">
                        <div class="fa fa-building me-lg-1"/>
                    </t>
                    <t t-elif="icon === 'home'">
                        <div class="fa fa-home me-lg-1"/>
                    </t>
                    <t t-else="">
                        <div class="fa fa-map-marker me-lg-1"/>
                    </t>
                    <span class="d-none d-lg-inline fw-bold" t-esc="title"/>
                </div>
                <span class="o_worklocation_line w-100" t-att-class="`wl_color_${colorIndex}`"/>
            </div>
        </t>
    </t>

    <t t-name="hr_homeworking.CalendarCommonRendererHeader" t-inherit="web.CalendarCommonRendererHeader" t-inherit-mode="primary">
        <xpath expr="//span[hasclass('o_cw_day_number')]" position="after">
            <t t-if="scale != 'month'" t-call="hr.homeworking.CalendarCommonRenderer.buttonWorklocation"/>
        </xpath>
    </t>

    <t t-name="hr.homeworking.CalendarCommonRenderer.buttonWorklocation">
        <div class="o_worklocation_btn w-100 d-flex align-items-center opacity-0 opacity-100-hover">
            <button class='o_worklocation_text bg-info bg-opacity-25 border-0 rounded-pill px-lg-2 py-0 text-nowrap ms-1'>
                <i class="fa fa-map-marker me-lg-1" aria-hidden="true"></i>
                <span class="d-lg-inline fw-bold">Set Location</span>
            </button>
            <button class="o_worklocation_line d-lg-block w-100 bg-info bg-opacity-25 border-0 p-0 me-1"/>
        </div>
    </t>
</templates>

```

## File: static\src\calendar\common\popover\calendar_common_popover.js

```javascript
/** @odoo-module **/

import { patch } from "@web/core/utils/patch";
import { AttendeeCalendarCommonPopover } from "@calendar/views/attendee_calendar/common/attendee_calendar_common_popover";
import { Field } from "@web/views/fields/field"

patch(AttendeeCalendarCommonPopover.prototype, {
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
    },
    isWorkLocationEvent(){
        return this.props.record['resModel'] === 'hr.employee.location';
    },
    get hasFooter() {
        return !this.isWorkLocationEvent() || this.props.record.userId === this.user.userId
    },
    isCurrentUserIsOwnerWorklocation(){
        return this.isWorkLocationEvent() && this.props.record.userId === this.slot.record.model.user.userId;
    },
    get isEventEditable() {
        return ('resModel' in this.props.record) || super.isEventEditable;
    },
    get isEventViewable() {
        return !('resModel' in this.props.record) || super.isEventViewable;
    },
    get isEventDeletable() {
        return super.isEventDeletable;
    },
    get displayAttendeeAnswerChoice() {
        return !('resModel' in this.props.record) && super.displayAttendeeAnswerChoice;
    },
    get isCurrentUserAttendee() {
        return !('resModel' in this.props.record) && super.isCurrentUserAttendee;
    },
})

AttendeeCalendarCommonPopover.template = "homework.AttendeeCalendarCommonPopover"


AttendeeCalendarCommonPopover.subTemplates = {
    ...AttendeeCalendarCommonPopover.subTemplates,
    body: "homework.AttendeeCalendarCommonPopover.body",
    footer: "homework.AttendeeCalendarCommonPopover.footer",
}

AttendeeCalendarCommonPopover.components = {
    ...AttendeeCalendarCommonPopover.components,
    Field
}

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
                        <Field name="'employee_name'" record="slot.record"/>
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

## File: static\src\components\hr_presence_status\hr_presence_status.js

```javascript
/** @odoo-module **/

import { patch } from "@web/core/utils/patch";

import { HrPresenceStatus, hrPresenceStatus } from "@hr/components/hr_presence_status/hr_presence_status";
import { HrPresenceStatusPrivate, hrPresenceStatusPrivate } from "@hr/components/hr_presence_status_private/hr_presence_status_private";

const patchHrPresenceStatus = () => ({
    get color() {
        if (this.location) {
            let color = "text-muted";
            if (this.props.record.data.hr_presence_state !== "to_define") {
                color = this.props.record.data.hr_presence_state === "present" ?  "text-success" : "text-warning";
            }
            return color;
        }
        return super.color;
    },

    get icon() {
        if (this.location) {
            switch (this.location) {
                case "home":
                    return "fa-home";
                case "office":
                    return "fa-building";
                case "other":
                    return "fa-map-marker";
            }
        }
        return super.icon;
    },

    get location() {
        let location = this.value?.split("_")[1] || "";
        if (location && !['home', 'office', 'other'].includes(location)) {
            location = "";
        }
        return location;
    },

    get label() {
        if (this.location) {
            return this.props.record.data.name_work_location_display;
        }
        return super.label;
    },
});

// for the both components: first applies the common patch and then applies patch for label
patch(HrPresenceStatus.prototype, patchHrPresenceStatus());
patch(HrPresenceStatusPrivate.prototype, patchHrPresenceStatus());

const additionalFieldDependencies = [
    { name: "hr_presence_state", type: "selection" },
    { name: "name_work_location_display", type: "char" }
];
if (typeof hrPresenceStatus.fieldDependencies === "function") {
    const oldFieldDependencies = hrPresenceStatus.fieldDependencies;
    hrPresenceStatus.fieldDependencies = (widgetInfo) => {
        const fieldDependencies = oldFieldDependencies(widgetInfo);
        fieldDependencies.push(...additionalFieldDependencies);
        return fieldDependencies;
    }
} else {
    hrPresenceStatus.fieldDependencies = [
        ...(hrPresenceStatus.fieldDependencies || []),
        ...additionalFieldDependencies,
    ];
}
hrPresenceStatusPrivate.fieldDependencies = [
    ...(hrPresenceStatusPrivate.fieldDependencies || []),
    ...additionalFieldDependencies,
];

```

## File: views\hr_employee_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_employee_filter" model="ir.ui.view">
        <field name="name">view.employee.form.inherit.hr</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='group_department']" position="after">
                <separator/>
                <filter name="_search_today_location" string="Work location" domain="[]" context="{'group_by':'today_location_name'}"/>
            </xpath>
        </field>
    </record>
    <record id="view_employee_form" model="ir.ui.view">
        <field name="name">view.employee.form.inherit.hr</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='work_location_id']" position="attributes">
                <attribute name="invisible">1</attribute>
            </xpath>
            <xpath expr="//group[@name='departure']" position="after">
                <group string="Remote Work">
                    <span class="text-muted fst-italic oe_inline" colspan="2">Specify your default work location for each day of the week. This schedule will repeat itself each week.</span>
                    <field name="monday_location_id" placeholder="Unspecified" force_save="1"/>
                    <field name="tuesday_location_id" placeholder="Unspecified" force_save="1"/>
                    <field name="wednesday_location_id" placeholder="Unspecified" force_save="1"/>
                    <field name="thursday_location_id" placeholder="Unspecified" force_save="1"/>
                    <field name="friday_location_id" placeholder="Unspecified" force_save="1"/>
                    <field name="saturday_location_id" placeholder="Unspecified" force_save="1"/>
                    <field name="sunday_location_id" placeholder="Unspecified" force_save="1"/>
                </group>
            </xpath>
        </field>
    </record>

    <record id="view_employee_tree" model="ir.ui.view">
        <field name="name">hr.employee.tree.timesheet</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='work_email']" position="after">
            <field name="name_work_location_display" readonly="0" string="Work Location" optional="hide"/>
            </xpath>
            <xpath expr="//field[@name='work_location_id']" position="attributes">
                <attribute name="column_invisible">1</attribute>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_users.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_useurs_view_form_profile" model="ir.ui.view">
            <field name="name">res.users.preferences.form.inherit</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="hr.res_users_view_form_profile"/>
            <field name="arch" type="xml">
                <xpath expr="//group[@name='managers']" position="after">
                    <group name="Homeworking" string="Remote Work">
                        <span class="text-muted fst-italic oe_inline" colspan="2">Specify your default work location for each day of the week. This schedule will repeat itself each week.</span>
                        <field name="monday_location_id"  placeholder="Unspecified"/>
                        <field name="tuesday_location_id"  placeholder="Unspecified"/>
                        <field name="wednesday_location_id" placeholder="Unspecified"/>
                        <field name="thursday_location_id"  placeholder="Unspecified"/>
                        <field name="friday_location_id"  placeholder="Unspecified"/>
                        <field name="saturday_location_id"  placeholder="Unspecified"/>
                        <field name="sunday_location_id"  placeholder="Unspecified"/>
                    </group>
                </xpath>
            </field>
        </record>
    </data>
</odoo>
```

## File: wizard\homework_location_wizard.py

```python
# -*- coding: utf-8 -*-
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

    weekly = fields.Boolean(default=False)
    date = fields.Date(string="Date")
    day_week_string = fields.Char(compute="_compute_day_week_string")

    @api.depends('date')
    def _compute_day_week_string(self):
        for record in self:
            record.day_week_string = record.date.strftime("%A")

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
            employee_id.write({
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
                    <label for="weekly"/>
                    <div class="o_row w-100"> 
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
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import homework_location_wizard

```

