# Odoo Module: hr_presence

Category: Human Resources

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

from odoo import api, SUPERUSER_ID


def post_init_hook(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})

    email = env['ir.config_parameter'].get_param('hr_presence.hr_presence_control_email')
    ip = env['ir.config_parameter'].get_param('hr_presence.hr_presence_control_ip')

    if not email and not ip:
        env['ir.config_parameter'].sudo().set_param('hr_presence.hr_presence_control_email', True)

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
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
        'data/ir_actions_server.xml',
        'views/hr_employee_views.xml',
        'data/mail_template_data.xml',
        'data/sms_data.xml',
        'data/ir_cron.xml',
    ],
    'installable': True,
    'post_init_hook': 'post_init_hook',
    'license': 'LGPL-3',
}

```

## File: data\ir_actions_server.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_actions_server_action_open_presence_view" model="ir.actions.server">
        <field name="name">Compute presence and open presence view</field>
        <field name="model_id" ref="hr.model_hr_employee" />
        <field name="state">code</field>
        <field name="code">
action = env['hr.employee']._action_open_presence_view()
        </field>
    </record>

    <menuitem id="menu_hr_presence_view" name="Presence" action="ir_actions_server_action_open_presence_view" parent="hr.hr_menu_hr_reports" groups="hr.group_hr_manager"/>

</odoo>

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
            <field name="numbercall">-1</field>
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
                    Exception made if there was a mistake of ours, it seems that you are not at your office and there is not request of time off from you.<br/>
                    Please, take appropriate measures in order to carry out this work absence.<br/>
                    Do not hesitate to contact your manager or the human resource department.
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
            <field name="body">Exception made if there was a mistake of ours, it seems that you are not at your office and there is not request of time off from you.
Please, take appropriate measures in order to carry out this work absence.
Do not hesitate to contact your manager or the human resource department.</field>
        </record>
    </data>
</odoo>

```

## File: models\hr_employee.py

```python
# -*- coding: utf-8 -*-
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

    # Stored field used in the presence kanban reporting view
    # to allow group by state.
    hr_presence_state_display = fields.Selection([
        ('to_define', 'To Define'),
        ('present', 'Present'),
        ('absent', 'Absent'),
        ])

    def _compute_presence_state(self):
        super()._compute_presence_state()
        employees = self.filtered(lambda e: e.hr_presence_state != 'present' and not e.is_absent)
        company = self.env.company
        employee_to_check_working = employees.filtered(lambda e:
                                                       not e.is_absent and
                                                       (e.email_sent or e.ip_connected or e.manually_set_present))
        working_now_list = employee_to_check_working._get_employee_working_now()
        for employee in employees:
            if not employee.is_absent and company.hr_presence_last_compute_date and employee.id in working_now_list and \
                    company.hr_presence_last_compute_date.day == Datetime.now().day and \
                    (employee.email_sent or employee.ip_connected or employee.manually_set_present):
                employee.hr_presence_state = 'present'

    @api.model
    def _check_presence(self):
        company = self.env.company
        if not company.hr_presence_last_compute_date or \
                company.hr_presence_last_compute_date.day != Datetime.now().day:
            self.env['hr.employee'].search([
                ('company_id', '=', company.id)
            ]).write({
                'email_sent': False,
                'ip_connected': False,
                'manually_set_present': False
            })

        employees = self.env['hr.employee'].search([('company_id', '=', company.id)])
        all_employees = employees


        # Check on IP
        if literal_eval(self.env['ir.config_parameter'].sudo().get_param('hr_presence.hr_presence_control_ip', 'False')):
            ip_list = company.hr_presence_control_ip_list
            ip_list = ip_list.split(',') if ip_list else []
            ip_employees = self.env['hr.employee']
            for employee in employees:
                employee_ips = self.env['res.users.log'].search([
                    ('create_uid', '=', employee.user_id.id),
                    ('ip', '!=', False),
                    ('create_date', '>=', Datetime.to_string(Datetime.now().replace(hour=0, minute=0, second=0, microsecond=0)))]
                ).mapped('ip')
                if any(ip in ip_list for ip in employee_ips):
                    ip_employees |= employee
            ip_employees.write({'ip_connected': True})
            employees = employees - ip_employees

        # Check on sent emails
        if literal_eval(self.env['ir.config_parameter'].sudo().get_param('hr_presence.hr_presence_control_email', 'False')):
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

    @api.model
    def _action_open_presence_view(self):
        # Compute the presence/absence for the employees on the same
        # company than the HR/manager. Then opens the kanban view
        # of the employees with an undefined presence/absence

        _logger.info("Employees presence checked by: %s" % self.env.user.name)

        self._check_presence()

        return {
            "type": "ir.actions.act_window",
            "res_model": "hr.employee",
            "views": [[self.env.ref('hr_presence.hr_employee_view_kanban').id, "kanban"], [False, "tree"], [False, "form"]],
            'view_mode': 'kanban,tree,form',
            "domain": [],
            "name": _("Employee's Presence to Define"),
            "search_view_id": [self.env.ref('hr_presence.hr_employee_view_presence_search').id, 'search'],
            "context": {'search_default_group_hr_presence_state': 1,
                        'searchpanel_default_hr_presence_state_display': 'to_define'},
        }

    def _action_set_manual_presence(self, state):
        if not self.env.user.has_group('hr.group_hr_manager'):
            raise UserError(_("You don't have the right to do this. Please contact an Administrator."))
        self.write({'manually_set_present': state})

    def action_set_present(self):
        self._action_set_manual_presence(True)

    def action_set_absent(self):
        self._action_set_manual_presence(False)

    def write(self, vals):
        if vals.get('hr_presence_state_display') == 'present':
            vals['manually_set_present'] = True
        return super().write(vals)

    def action_open_leave_request(self):
        self.ensure_one()
        return {
            "type": "ir.actions.act_window",
            "res_model": "hr.leave",
            "views": [[False, "form"]],
            "view_mode": 'form',
            "context": {'default_employee_id': self.id},
        }

    # --------------------------------------------------
    # Messaging
    # --------------------------------------------------

    def action_send_sms(self):
        self.ensure_one()
        if not self.env.user.has_group('hr.group_hr_manager'):
            raise UserError(_("You don't have the right to do this. Please contact an Administrator."))
        if not self.mobile_phone:
            raise UserError(_("There is no professional mobile for this employee."))

        context = dict(self.env.context)
        context.update(default_res_model='hr.employee', default_res_id=self.id, default_composition_mode='comment', default_number_field_name='mobile_phone')

        template = self.env.ref('hr_presence.sms_template_presence', False)
        if not template:
            context['default_body'] = _("""Exception made if there was a mistake of ours, it seems that you are not at your office and there is not request of time off from you.
Please, take appropriate measures in order to carry out this work absence.
Do not hesitate to contact your manager or the human resource department.""")
        else:
            context['default_template_id'] = template.id

        return {
            "type": "ir.actions.act_window",
            "res_model": "sms.composer",
            "view_mode": 'form',
            "context": context,
            "name": "Send SMS Text Message",
            "target": "new",
        }

    def action_send_mail(self):
        self.ensure_one()
        if not self.env.user.has_group('hr.group_hr_manager'):
            raise UserError(_("You don't have the right to do this. Please contact an Administrator."))
        if not self.work_email:
            raise UserError(_("There is no professional email address for this employee."))
        template = self.env.ref('hr_presence.mail_template_presence', False)
        compose_form = self.env.ref('mail.email_compose_message_wizard_form', False)
        ctx = dict(
            default_model="hr.employee",
            default_res_id=self.id,
            default_use_template=bool(template),
            default_template_id=template.id,
            default_composition_mode='comment',
            default_is_log=True,
            default_email_layout_xmlid='mail.mail_notification_light',
        )
        return {
            'name': _('Compose Email'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'mail.compose.message',
            'views': [(compose_form.id, 'form')],
            'view_id': compose_form.id,
            'target': 'new',
            'context': ctx,
        }

```

## File: models\ir_websocket.py

```python
# -*- coding: utf-8 -*-

from odoo import models, registry
from odoo.api import Environment
from odoo.fields import Datetime
from odoo.http import request
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
            users_log = req.env['res.users.log'].search_count([
                ('create_uid', '=', req.env.user.id),
                ('ip', '=', ip_address),
                ('create_date', '>=', Datetime.to_string(Datetime.now().replace(hour=0, minute=0, second=0, microsecond=0)))])
            if not users_log:
                with registry(req.env.cr.dbname).cursor() as cr:
                    env = Environment(cr, req.env.user.id, {})
                    env['res.users.log'].create({'ip': ip_address})

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

from . import hr_employee
from . import ir_websocket
from . import res_company
from . import res_users_log

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

## File: views\hr_employee_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hr_employee_view_search" model="ir.ui.view">
        <field name="name">hr.employee.view.search</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.view_employee_filter"/>
        <field name="arch" type="xml">
            <filter name="group_manager" position="after">
                <filter name="group_hr_presence_state" string="Presence" domain="[]" context="{'group_by':'hr_presence_state_display'}" groups="hr.group_hr_manager"/>
            </filter>
        </field>
    </record>

    <record id="hr_employee_view_presence_search" model="ir.ui.view">
        <field name="name">hr.employee.view.search.presence</field>
        <field name="model">hr.employee</field>
        <field name="arch" type="xml">
            <search string="Employees">
                <filter name="group_hr_presence_state" string="Presence" domain="[]" context="{'group_by':'hr_presence_state_display'}"/>
                <searchpanel>
                    <field name="company_id" groups="base.group_multi_company" icon="fa-building"/>
                    <field name="hr_presence_state_display" string="Absence/Presence" />
                    <field name="department_id" icon="fa-users"/>
                </searchpanel>
            </search>
        </field>
    </record>

    <record id="hr_employee_view_kanban" model="ir.ui.view">
        <field name="name">hr.employee.view.kanban</field>
        <field name="model">hr.employee</field>
        <field name="inherit_id" ref="hr.hr_kanban_view_employees"/>
        <field name="groups_id" eval="[(4, ref('hr.group_hr_manager'))]"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//kanban" position="attributes">
                <attribute name="create">0</attribute>
            </xpath>
            <xpath expr="//div[hasclass('oe_kanban_details')]" position="inside">
                <div class="oe_kanban_content">
                    <div class="oe_kanban_row">
                        <field name="hr_presence_state" invisible="1"/>
                        <button name="action_set_present" type="object" class="btn btn-primary btn-sm mt8 fa fa-check" title="Set as present" attrs="{'invisible': [('hr_presence_state', '=', 'present')]}"> </button>
                        <button name="action_set_absent" type="object" class="btn btn-primary btn-sm mt8 fa fa-times" title="Set as absent" attrs="{'invisible': [('hr_presence_state', '!=', 'present')]}"> </button>
                        <button name="action_open_leave_request" type="object" class="btn btn-primary btn-sm mt8">Time Off</button>
                    </div>
                    <div class="oe_kanban_row">
                        <button name="action_send_sms" type="object" class="btn btn-primary btn-sm mt8">SMS</button>
                        <button name="action_send_mail" type="object" class="btn btn-primary btn-sm mt8">Log</button>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

