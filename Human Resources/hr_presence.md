# Odoo Module: hr_presence

Category: Human Resources

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
    'name': 'Employee Presence Control',
    'version': '1.0',
    'category': 'Human Resources',
    'description': """
Control Employees Presence
==========================

Based on:
    * The IP Address
    * The User's Session
    * The Sent Emails

Allows to contact directly the employee in case of unjustified absence.
    """,
    'depends': ['hr', 'hr_holidays', 'sms'],
    'data': [
        'security/sms_security.xml',
        'security/ir.model.access.csv',
        'views/hr_employee_views.xml',
        'data/mail_template_data.xml',
        'data/sms_data.xml',
        'data/ir_cron.xml',
    ],
    'installable': True,
    'license': 'LGPL-3',
     'assets': {
        'web.assets_backend': [
            'hr_presence/static/src/**/*',
        ],
    }
}

```

## File: data\ir_cron.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record forcecreate="True" id="ir_cron_presence_control" model="ir.cron">
            <field name="name">HR Presence: cron</field>
            <field name="model_id" ref="hr.model_hr_employee"/>
            <field name="state">code</field>
            <field name="code">model._check_presence()</field>
            <field eval="True" name="active" />
            <field name="user_id" ref="base.user_root" />
            <field name="interval_number">1</field>
            <field name="interval_type">hours</field>
        </record>
    </data>
</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mail_template_presence" model="mail.template">
            <field name="name">HR: Employee Absence email</field>
            <field name="model_id" ref="hr.model_hr_employee"/>
            <field name="subject">Unexpected Absence</field>
            <field name="email_from">{{ user.email_formatted }}</field>
            <field name="email_to">{{ (object.user_id.email_formatted or object.work_email) }}</field>
            <field name="auto_delete" eval="False"/>
            <field name="description">Sent manually in presence module when an employee wasn't working despite not being off</field>
            <field name="body_html" type="html">
                <div>
                    Dear <t t-out="object.name or ''">Abigail Peterson</t>,<br/><br/>
We hope this message finds you well. It has come to our attention that you are currently not present at work, and there is no record of a time off request from you. If this absence is due to an oversight on our part, we sincerely apologize for any confusion.
Please take the necessary steps to address this unplanned absence. Should you have any questions or need assistance, do not hesitate to reach out to your manager or the HR department at your earliest convenience.
Thank you for your prompt attention to this matter.
                    <br/>Best Regards,<br/><br/>
                </div>
            </field>
        </record>
    </data>
</odoo>

```

## File: data\sms_data.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <data noupdate="1">
        <record id="sms_template_data_hr_presence" model="sms.template">
            <field name="name">Employee: Presence Reminder</field>
            <field name="model_id" ref="hr.model_hr_employee"/>
            <field name="body">Hi, we noticed you're not at work and no time-off was submitted. If this is an oversight from us, we apologize. Please contact your manager or HR ASAP. Thanks</field>
        </record>
    </data>
</odoo>

```

## File: models\hr_employee.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from ast import literal_eval
from odoo import fields, models, _, api
from odoo.exceptions import UserError
from odoo.fields import Datetime

_logger = logging.getLogger(__name__)


class Employee(models.AbstractModel):
    _inherit = 'hr.employee.base'

    email_sent = fields.Boolean(default=False)
    ip_connected = fields.Boolean(default=False)
    manually_set_present = fields.Boolean(default=False)
    manually_set_presence = fields.Boolean(default=False)

    # Stored field used in the presence kanban reporting view
    # to allow group by state.
    hr_presence_state_display = fields.Selection([
        ('out_of_working_hour', 'Out of Working Hours'),
        ('present', 'Present'),
        ('absent', 'Absent'),
        ], default='out_of_working_hour')

    @api.model
    def _check_presence(self):
        company = self.env.company
        employees = self.env['hr.employee'].search([('company_id', '=', company.id)])

        employees.write({
            'email_sent': False,
            'ip_connected': False,
            'manually_set_present': False,
            'manually_set_presence': False,
        })

        all_employees = employees


        # Check on IP
        if company.hr_presence_control_ip:
            ip_list = company.hr_presence_control_ip_list
            ip_list = ip_list.split(',') if ip_list else []
            ip_employees = self.env['hr.employee']
            for employee in employees:
                employee_ips = self.env['res.users.log'].sudo().search([
                    ('create_uid', '=', employee.user_id.id),
                    ('ip', '!=', False),
                    ('create_date', '>=', Datetime.to_string(Datetime.now().replace(hour=0, minute=0, second=0, microsecond=0)))]
                ).mapped('ip')
                if any(ip in ip_list for ip in employee_ips):
                    ip_employees |= employee
            ip_employees.write({'ip_connected': True})
            employees = employees - ip_employees

        # Check on sent emails
        if company.hr_presence_control_email:
            email_employees = self.env['hr.employee']
            threshold = company.hr_presence_control_email_amount
            for employee in employees:
                sent_emails = self.env['mail.message'].search_count([
                    ('author_id', '=', employee.user_id.partner_id.id),
                    ('date', '>=', Datetime.to_string(Datetime.now().replace(hour=0, minute=0, second=0, microsecond=0))),
                    ('date', '<=', Datetime.to_string(Datetime.now()))])
                if sent_emails >= threshold:
                    email_employees |= employee
            email_employees.write({'email_sent': True})
            employees = employees - email_employees

        company.sudo().hr_presence_last_compute_date = Datetime.now()

        for employee in all_employees:
            employee.hr_presence_state_display = employee.hr_presence_state

    def get_presence_server_action_data(self):
        server_action_xmlids = [
            'action_hr_employee_presence_present',
            'action_hr_employee_presence_absent',
            'action_hr_employee_presence_log',
            'action_hr_employee_presence_sms',
            'action_hr_employee_presence_time_off',
        ]
        actions = self.env['ir.actions.server'].sudo()
        for xmlid in server_action_xmlids:
            actions += actions.env.ref(f"hr_presence.{xmlid}")
        return actions.read(['id', 'value'])

    def _action_set_manual_presence(self, state):
        if not self.env.user.has_group('hr.group_hr_manager'):
            raise UserError(_("You don't have the right to do this. Please contact an Administrator."))
        self.write({
            'manually_set_present': state,
            'manually_set_presence': True,
            "hr_presence_state_display": 'present' if state else 'absent',
        })

    def action_set_present(self):
        self._action_set_manual_presence(True)

    def action_set_absent(self):
        self._action_set_manual_presence(False)

    def write(self, vals):
        if vals.get('hr_presence_state_display') == 'present':
            vals['manually_set_present'] = True
        return super().write(vals)

    def action_open_leave_request(self):
        if len(self) == 1:
            model = 'hr.leave'
            context = {'default_employee_id': self.id}
        else:
            model = 'hr.leave.generate.multi.wizard'
            context = {
                'default_employee_ids': self.ids,
                'default_date_from': fields.Date.today(),
                'default_date_to': fields.Date.today(),
                'default_name': _('Unplanned Absence'),
            }

        return {
            'type': 'ir.actions.act_window',
            'res_model': model,
            'views': [[False, 'form']],
            'view_mode': 'form',
            'context': context,
            'target': 'new',
        }

    # --------------------------------------------------
    # Messaging
    # --------------------------------------------------

    def action_send_sms(self):
        if not self.env.user.has_group('hr.group_hr_manager'):
            raise UserError(_("You don't have the right to do this. Please contact an Administrator."))

        context = dict(self.env.context)
        context.update(default_res_model='hr.employee', default_res_ids=self.ids, default_composition_mode='mass', default_number_field_name='mobile_phone', default_mass_keep_log=True)

        template = self.env.ref('hr_presence.sms_template_presence', False)
        if not template:
            context['default_body'] = _("""We hope this message finds you well. It has come to our attention that you are currently not present at work, and there is no record of a time off request from you. If this absence is due to an oversight on our part, we sincerely apologize for any confusion.
Please take the necessary steps to address this unplanned absence. Should you have any questions or need assistance, do not hesitate to reach out to your manager or the HR department at your earliest convenience.
Thank you for your prompt attention to this matter.""")
        else:
            context['default_template_id'] = template.id

        return {
            "type": "ir.actions.act_window",
            "res_model": "sms.composer",
            "view_mode": 'form',
            "context": context,
            "name": _("Send SMS"),
            "target": "new",
        }

    def action_send_log(self):
        if not self.env.user.has_group('hr.group_hr_manager'):
            raise UserError(_("You don't have the right to do this. Please contact an Administrator."))

        for employee in self:
            employee.message_post(body=_(
                "%(name)s has been noted as %(state)s today",
                name=employee.name,
                state=employee.hr_presence_state_display))

```

## File: models\hr_employee_base.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, fields, models


class HrEmployeeBase(models.AbstractModel):
    _inherit = "hr.employee.base"

    @api.depends("user_id.im_status", "hr_presence_state_display")
    def _compute_presence_state(self):
        super()._compute_presence_state()
        company = self.env.company
        working_now_list = self._get_employee_working_now()
        for employee in self:
            if employee.manually_set_presence:
                employee.hr_presence_state = employee.hr_presence_state_display
                continue

            if not employee.company_id.hr_presence_control_email and not employee.company_id.hr_presence_control_ip:
                continue
            if company.hr_presence_last_compute_date and employee.id in working_now_list and \
                    company.hr_presence_last_compute_date.day == fields.Datetime.now().day and \
                    (employee.email_sent or employee.ip_connected or employee.manually_set_present):
                employee.hr_presence_state = 'present'
            elif employee.id in working_now_list and employee.is_absent and \
                not (employee.email_sent or employee.ip_connected or employee.manually_set_present):
                employee.hr_presence_state = 'absent'
            else:
                employee.hr_presence_state = 'out_of_working_hour'

```

## File: models\ir_websocket.py

```python
# -*- coding: utf-8 -*-

from odoo import models
from odoo.api import Environment
from odoo.fields import Datetime
from odoo.http import request
from odoo.modules.registry import Registry
from odoo.addons.bus.websocket import wsrequest

class IrWebsocket(models.AbstractModel):
    _inherit = 'ir.websocket'

    def _update_bus_presence(self, inactivity_period, im_status_ids_by_model):
        super()._update_bus_presence(inactivity_period, im_status_ids_by_model)
        #  This method can either be called due to an http or a
        #  websocket request. The request itself is necessary to
        #  retrieve the current guest. Let's retrieve the proper
        #  request.
        req = request or wsrequest
        if req.env.user._is_internal():
            ip_address = req.httprequest.remote_addr
            users_log = req.env['res.users.log'].sudo().search_count([
                ('create_uid', '=', req.env.user.id),
                ('ip', '=', ip_address),
                ('create_date', '>=', Datetime.to_string(Datetime.now().replace(hour=0, minute=0, second=0, microsecond=0)))])
            if not users_log:
                with Registry(req.env.cr.dbname).cursor() as cr:
                    env = Environment(cr, req.env.user.id, {})
                    env['res.users.log'].sudo().create({'ip': ip_address})

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    hr_presence_last_compute_date = fields.Datetime()

```

## File: models\res_config_settings.py

```python

from odoo import models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    def create(self, vals):
        configs = super().create(vals)
        if any(config.hr_presence_control_ip or config.hr_presence_control_email for config in configs):
            self.env['hr.employee.base']._check_presence()
        return configs

```

## File: models\res_users_log.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models


class ResUsersLog(models.Model):
    _inherit = 'res.users.log'

    create_uid = fields.Integer(index=True)
    ip = fields.Char(string="IP Address")

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import hr_employee_base
from . import hr_employee
from . import ir_websocket
from . import res_company
from . import res_users_log
from . import res_config_settings

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_sms_template_hr_manager,access.sms.template.hr.manager,sms.model_sms_template,hr.group_hr_manager,1,1,1,1

```

## File: security\sms_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_rule_sms_template_hr_manager" model="ir.rule">
        <field name="name">SMS Template: hr manager CUD on employee templates</field>
        <field name="model_id" ref="sms.model_sms_template"/>
        <field name="groups" eval="[(4, ref('hr.group_hr_manager'))]"/>
        <field name="domain_force">[('model_id.model', '=', 'hr.employee')]</field>
        <field name="perm_read" eval="False"/>
    </record>
</odoo>

```

## File: static\src\search\hr_presence_action_menus\hr_presence_action_menus.js

```javascript
import { ActionMenus } from "@web/search/action_menus/action_menus";
import { getActionRecords, getPresenceActionItems } from "../../views/hooks";

/**
 * @extends ActionMenus
 */
export class HrPresenceActionMenus extends ActionMenus {
    static template = "hr_presence.actionmenu";

    get PresenceActionItems() {
        return (this.presenceActionItems || []).map((action) => {
            return {
                action,
                description: action.name,
                key: action.id,
                groupNumber: action.groupNumber,
            };
        });
    }

    /**
     * @override
     */
    async getActionItems(props) {
        const records = await getActionRecords(this.orm);
        const result = getPresenceActionItems(props.items.action, records);

        props.items.action = result[0];
        this.presenceActionItems = result[1];

        return await super.getActionItems(props);
    }
}

```

## File: static\src\search\hr_presence_action_menus\hr_presence_action_menus.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="hr_presence.actionmenu" t-inherit="web.ActionMenus" t-inherit-mode="primary">
        <xpath expr="//div[@t-if='props.items.print?.length']" position="before">
            <t t-if="PresenceActionItems.length" class="d-inline-block">
                <t t-call="hr_presence.menu">
                    <t t-set="is_actionmenu" t-value="'True'"/>
                </t>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\search\hr_presence_cog_menu\hr_presence_cog_menu.js

```javascript
import { FormCogMenu } from "@web/views/form/form_cog_menu/form_cog_menu";
import { onWillStart } from "@odoo/owl";
import { getActionRecords, getPresenceActionItems } from "../../views/hooks";

/**
 * @extends CogMenu
 */
export class HrPresenceCogMenu extends FormCogMenu {
    static template = "hr_presence.cogmenu";

    setup() {
        super.setup();

        onWillStart(async () => {
            await super.onWillStart;
            this.records = await getActionRecords(this.orm);
        });
    }

    /**
     * @override
     */
    get cogItems() {
        var result = super.cogItems;
        result = getPresenceActionItems(result, this.records);
        this.presenceActionItems = result[1];
        return result[0];
    }

    get PresenceActionItems() {
        return this.presenceActionItems;
    }
}

```

## File: static\src\search\hr_presence_cog_menu\hr_presence_cog_menu.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="hr_presence.cogmenu" t-inherit="web.FormCogMenu" t-inherit-mode="primary">
        <xpath expr="//t[@t-if='state.printItems.length']" position="after">
            <t t-if="PresenceActionItems.length">
                <t t-call="hr_presence.menu">
                    <t t-set="is_actionmenu" t-value="''"/>
                </t>
            </t>
        </xpath>
    </t>

    <t t-name="hr_presence.menu">
        <Dropdown>
            <button t-attf-class="{{ is_actionmenu ? 'btn btn-secondary' : '' }}">
                <i class="fa fa-clock-o me-1"/>
                <span class="o_dropdown_title">Presence Control</span>
            </button>
            <t t-set-slot="content">
                <t t-foreach="PresenceActionItems" t-as="item" t-key="item.key">
                    <t t-if="currentGroup !== null and currentGroup !== item.groupNumber">
                        <div role="separator" class="dropdown-divider"/>
                    </t>
                    <DropdownItem class="'o_menu_item'" onSelected="() => this.onItemSelected(item)">
                        <t t-esc="item.description"/>
                    </DropdownItem>
                    <t t-set="currentGroup" t-value="item.groupNumber"/>
                </t>
            </t>
        </Dropdown>
    </t>

</templates>

```

## File: static\src\views\hooks.js

```javascript
export async function getActionRecords(orm) {
    return await orm.call("hr.employee.base", "get_presence_server_action_data", [[]]);
}
export function getPresenceActionItems(actions, records) {
    const presenceActionItems = [];
    const presenceRecords = {
        group_1: records
            .filter((record) => record.value === "presence_group_1")
            .map((record) => record.id),
        group_2: records
            .filter((record) => record.value === "presence_group_2")
            .map((record) => record.id),
    };
    actions = (actions || []).filter((action) => {
        const isInGroup1 = presenceRecords["group_1"].includes(action.key || action.id);
        const isInGroup2 = presenceRecords["group_2"].includes(action.key || action.id);
        if (isInGroup1 || isInGroup2) {
            action.groupNumber = isInGroup1 ? 50 : 60;
            presenceActionItems.push(action);
            return false; // Exclude this object from the resulting array
        }
        return true; // Include this object in the resulting array
    });
    return [actions, presenceActionItems];
}

```

## File: static\src\views\hr_presence_form_view.js

```javascript
import { patch } from "@web/core/utils/patch";
import { EmployeeFormController } from "@hr/views/form_view";
import { HrPresenceCogMenu } from "../search/hr_presence_cog_menu/hr_presence_cog_menu";

patch(EmployeeFormController, {
    components: {
        ...EmployeeFormController.components,
        CogMenu: HrPresenceCogMenu,
    },
});

```

## File: static\src\views\hr_presence_list_view.js

```javascript
import { patch } from "@web/core/utils/patch";
import { EmployeeListController } from '@hr/views/list_view';
import { HrPresenceActionMenus } from "../search/hr_presence_action_menus/hr_presence_action_menus";

patch(EmployeeListController, {
    components: {
        ...EmployeeListController.components,
        ActionMenus: HrPresenceActionMenus,
    },
});

```

## File: views\hr_employee_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_employee_view_search" model="ir.ui.view">
        <field name="name">hr.employee.view.search</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_filter"/>
        <field name="arch" type="xml">
            <filter name="at_work" position="after">
                <filter name="absent" string="Absent" domain="[('hr_presence_state_display', '=', 'absent')]"/>
                <filter name="out_of_working_hours" string="Out of Working Hours" domain="[('hr_presence_state_display', '=', 'out_of_working_hour')]"/>
            </filter>
            <filter name="group_start" position="before">
                <filter name="group_hr_presence_state" string="Presence/Absence" domain="[]" context="{'group_by':'hr_presence_state_display'}" groups="hr.group_hr_manager"/>
            </filter>
        </field>
    </record>

    <record id="hr_employee_view_tree" model="ir.ui.view">
        <field name="name">hr.employee.view.tree</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_tree"/>
        <field name="arch" type="xml">
            <field name="work_email" position="after">
                <field name="hr_presence_state_display" string="Presence" optional="show"/>
            </field>
        </field>
    </record>

    <record id="action_hr_employee_presence_present" model="ir.actions.server">
        <field name="name">Set Present</field>
        <field name="value">presence_group_1</field>
        <field name="model_id" ref="hr.model_hr_employee"/>
        <field name="binding_model_id" ref="hr.model_hr_employee"/>
        <field name="binding_view_types">form,list</field>
        <field name="state">code</field>
        <field name="code">
            action = records.action_set_present()
        </field>
    </record>

    <record id="action_hr_employee_presence_absent" model="ir.actions.server">
        <field name="name">Set Absent</field>
        <field name="value">presence_group_1</field>
        <field name="model_id" ref="hr.model_hr_employee"/>
        <field name="binding_model_id" ref="hr.model_hr_employee"/>
        <field name="binding_view_types">form,list</field>
        <field name="state">code</field>
        <field name="code">
            action = records.action_set_absent()
        </field>
    </record>

    <record id="action_hr_employee_presence_log" model="ir.actions.server">
        <field name="name">Add a log note</field>
        <field name="model_id" ref="hr.model_hr_employee"/>
        <field name="value">presence_group_2</field>
        <field name="binding_model_id" ref="hr.model_hr_employee"/>
        <field name="binding_view_types">form,list</field>
        <field name="state">code</field>
        <field name="code">
            action = records.action_send_log()
        </field>
    </record>

    <record id="action_hr_employee_presence_sms" model="ir.actions.server">
        <field name="name">Send a SMS</field>
        <field name="model_id" ref="hr.model_hr_employee"/>
        <field name="value">presence_group_2</field>
        <field name="binding_model_id" ref="hr.model_hr_employee"/>
        <field name="binding_view_types">form,list</field>
        <field name="state">code</field>
        <field name="code">
            action = records.action_send_sms()
        </field>
    </record>

    <record id="action_hr_employee_presence_time_off" model="ir.actions.server">
        <field name="name">Create a Time Off</field>
        <field name="model_id" ref="hr.model_hr_employee"/>
        <field name="value">presence_group_2</field>
        <field name="binding_model_id" ref="hr.model_hr_employee"/>
        <field name="binding_view_types">form,list</field>
        <field name="state">code</field>
        <field name="code">
            action = records.action_open_leave_request()
        </field>
    </record>

</odoo>

```

