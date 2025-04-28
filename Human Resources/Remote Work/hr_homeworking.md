# Odoo Module: hr_homeworking

Category: Human Resources/Remote Work

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
    'name': 'Remote Work',
    'version': '2.0',
    'category': 'Human Resources/Remote Work',
    'depends': ['hr'],
    'data': [
        'security/security.xml',
        'security/ir.model.access.csv',
        'views/hr_employee_views.xml',
        'views/res_users.xml',
    ],
    'installable': True,
    'assets': {
        'web.assets_backend': [
            'hr_homeworking/static/src/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\hr_employee.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models

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
            res['views']['list']['arch'] = res['views']['list']['arch'].replace('work_location_name', dayfield)
        return res

    @api.depends("work_location_id.name", "work_location_id.location_type", "exceptional_location_id")
    def _compute_work_location_name_type(self):
        super()._compute_work_location_name_type()
        dayfield = self._get_current_day_location_field()
        for employee in self:
            current_location_id = employee.exceptional_location_id or employee[dayfield]
            employee.work_location_name = current_location_id.name or employee.work_location_name
            employee.work_location_type = current_location_id.location_type or employee.work_location_type

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

```

## File: models\hr_homeworking.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models

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
        ('uniq_exceptional_per_day', 'unique(employee_id, date)', 'Only one default work location and one exceptional work location per day per employee.'),
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

    def _compute_im_status(self):
        super()._compute_im_status()
        dayfield = self.env['hr.employee']._get_current_day_location_field()
        for user in self:
            location_type = user[dayfield].location_type
            if not location_type:
                continue
            im_status = user.im_status
            if im_status == "online" or im_status == "away" or im_status == "offline":
                user.im_status = "presence_" + location_type + "_" + im_status

    def _is_user_available(self):
        location_types = self.env['hr.work.location']._fields['location_type'].get_values(self.env)
        return self.im_status in ['online'] + [f'presence_{location_type}_online' for location_type in location_types]

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
    </data>
</odoo>

```

## File: static\src\avatar_card_popover_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.AvatarCardPopover" t-inherit-mode="extension">
        <xpath expr="//span[@name='icon']" position="inside">
            <i t-elif="user.im_status === 'presence_home_online'" class="fa fa-fw fa-home text-success" title="At Home - Online" role="img" aria-label="At Home - Online"/>
            <i t-elif="user.im_status === 'presence_home_away'" class="fa fa-fw fa-home text-warning" title="At Home - Idle" role="img" aria-label="At Home - Idle"/>
            <i t-elif="user.im_status === 'presence_home_offline'" class="fa fa-fw fa-home text-700" title="At Home - Offline" role="img" aria-label="At Home - Offline"/>
            <i t-elif="user.im_status === 'presence_office_online'" class="fa fa-fw fa-building text-success" title="At Office - Online" role="img" aria-label="At Office - Online"/>
            <i t-elif="user.im_status === 'presence_office_away'" class="fa fa-fw fa-building text-warning" title="At Office - Idle" role="img" aria-label="At Office - Idle"/>
            <i t-elif="user.im_status === 'presence_office_offline'" class="fa fa-fw fa-building text-700" title="At Office - Offline" role="img" aria-label="At Office - Offline"/>
            <i t-elif="user.im_status === 'presence_other_online'" class="fa fa-fw fa-map-marker text-success" title="At Other - Online" role="img" aria-label="At Other - Online"/>
            <i t-elif="user.im_status === 'presence_other_away'" class="fa fa-fw fa-map-marker text-warning" title="At Other - Idle" role="img" aria-label="At Other - Idle"/>
            <i t-elif="user.im_status === 'presence_other_offline'" class="fa fa-fw fa-map-marker text-700" title="At Other - Offline" role="img" aria-label="At Other - Offline"/>
        </xpath>
    </t>
</templates>

```

## File: static\src\im_status_patch.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="mail.ImStatus" t-inherit-mode="extension">
        <xpath expr="//*[@name='icon']" position="replace">
            <t t-if="persona.im_status">
                <t t-if="persona.im_status.split('_').length == 2">
                    <t t-set="location" t-value="persona.im_status.split('_')[0]"/>
                    <t t-if="location == 'home' || location == 'office' || location == 'other'">
                        <t t-set="status" t-value="persona.im_status.split('_')[1]"/>
                        <t t-set="location_map" t-value="{'home': 'fa-home', 'office': 'fa-building', 'other': 'fa-map-marker'}"/>
                        <t t-set="status_map" t-value="{'online': 'text-success', 'away': 'o-away', 'offline': 'text-7000'}"/>
                        <t t-set="icon" t-value="location_map[location]"/>
                        <t t-set="text" t-value="status_map[status]"/>
                        <i class="fa" t-attf-class="{{icon}} {{text}}" t-attf-title="At {{location}} - {{status}}" role="img" t-attf-aria-label=" User work at {{location}} and is {{status}}"/>
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
            <t t-if="correspondent.persona.im_status">
                <t t-if="correspondent.persona.im_status.split('_').length >= 2">
                    <t t-set="location" t-value="correspondent.persona.im_status.split('_')[0]"/>
                    <t t-if="location == 'home' || location == 'office' || location == 'other'">
                        <t t-set="status" t-value="correspondent.persona.im_status.split('_')[1]"/>
                        <t t-set="location_map" t-value="{'home': 'fa-home', 'office': 'fa-building', 'other': 'fa-map-marker'}"/>
                        <t t-set="status_map" t-value="{'online': 'text-success', 'away': 'o-away', 'offline': 'text-7000'}"/>
                        <t t-set="icon" t-value="location_map[location]"/>
                        <t t-set="text" t-value="status_map[status]"/>
                        <i class="fa" t-attf-class="{{icon}} {{text}}" t-attf-title="At {{location}} - {{status}}" role="img" t-attf-aria-label=" User work at {{location}} and is {{status}}"/>
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

## File: static\src\components\avatar_card_resource\avatar_card_resource_popover.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="resource_mail.AvatarCardResourcePopover" t-inherit-mode="extension">
        <xpath expr="//span[@name='icon'][hasclass('o_user_im_status')]/i[hasclass('fa-question-circle')]" position="before">
            <i t-elif="record.im_status === 'presence_home_online'" class="fa fa-fw fa-home text-success" title="At Home - Online" role="img" aria-label="At Home - Online"/>
            <i t-elif="record.im_status === 'presence_home_away'" class="fa fa-fw fa-home text-warning" title="At Home - Idle" role="img" aria-label="At Home - Idle"/>
            <i t-elif="record.im_status === 'presence_home_offline'" class="fa fa-fw fa-home text-700" title="At Home - Offline" role="img" aria-label="At Home - Offline"/>
            <i t-elif="record.im_status === 'presence_office_online'" class="fa fa-fw fa-building text-success" title="At Office - Online" role="img" aria-label="At Office - Online"/>
            <i t-elif="record.im_status === 'presence_office_away'" class="fa fa-fw fa-building text-warning" title="At Office - Idle" role="img" aria-label="At Office - Idle"/>
            <i t-elif="record.im_status === 'presence_office_offline'" class="fa fa-fw fa-building text-700" title="At Office - Offline" role="img" aria-label="At Office - Offline"/>
            <i t-elif="record.im_status === 'presence_other_online'" class="fa fa-fw fa-map-marker text-success" title="At Other - Online" role="img" aria-label="At Other - Online"/>
            <i t-elif="record.im_status === 'presence_other_away'" class="fa fa-fw fa-map-marker text-warning" title="At Other - Idle" role="img" aria-label="At Other - Idle"/>
            <i t-elif="record.im_status === 'presence_other_offline'" class="fa fa-fw fa-map-marker text-700" title="At Other - Offline" role="img" aria-label="At Other - Offline"/>
        </xpath>
        <xpath expr="//span[@name='icon'][hasclass('o_employee_presence_status')]" position="inside">
            <i t-if="record.hr_icon_display === 'presence_home'" class="fa fa-fw fa-home text-success me-1" title="At Home - Present" role="img" aria-label="At Home"/>
            <i t-if="record.hr_icon_display === 'presence_office'" class="fa fa-fw fa-building text-success me-1" title="At Office - Present" role="img" aria-label="At Office"/>
            <i t-if="record.hr_icon_display === 'presence_other'" class="fa fa-fw fa-map-marker text-success me-1" title="Present" role="img" aria-label="At Other"/>
        </xpath>
    </t>
</templates>

```

## File: static\src\components\hr_presence_status\hr_presence_status.js

```javascript
/** @odoo-module **/

import { patch } from "@web/core/utils/patch";

import { HrPresenceStatus, hrPresenceStatus } from "@hr/components/hr_presence_status/hr_presence_status";
import { HrPresenceStatusPrivate, hrPresenceStatusPrivate } from "@hr/components/hr_presence_status_private/hr_presence_status_private";
import { _t } from "@web/core/l10n/translation";

const patchHrPresenceStatus = () => ({
    get color() {
        if (this.location) {
            let color = "text-muted";
            if (this.props.record.data.hr_presence_state !== "out_of_working_hour") {
                color = this.props.record.data.hr_presence_state === "present" ?  "text-success" : "o_icon_employee_absent";
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
            return this.props.record.data.work_location_name || _t("Unspecified");
        }
        return super.label;
    },
});

// for the both components: first applies the common patch and then applies patch for label
patch(HrPresenceStatus.prototype, patchHrPresenceStatus());
patch(HrPresenceStatusPrivate.prototype, patchHrPresenceStatus());

const additionalFieldDependencies = [
    { name: "hr_presence_state", type: "selection" },
    { name: "work_location_name", type: "char" },
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
        <field name="name">hr.employee.list.timesheet</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='work_email']" position="after">
            <field name="work_location_name" readonly="0" string="Work Location" optional="hide"/>
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

