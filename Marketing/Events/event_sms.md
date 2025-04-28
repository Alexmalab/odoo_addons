# Odoo Module: event_sms

Category: Marketing/Events

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'SMS on Events',
    'version': '1.0',
    'category': 'Marketing/Events',
    'description': """Schedule SMS in event management""",
    'depends': ['event', 'sms'],
    'data': [
        'data/sms_data.xml',
        'security/ir.model.access.csv',
        'security/sms_security.xml',
    ],
    'demo': [
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\sms_data.xml

```xml
<?xml version="1.0"?>
<odoo><data noupdate="1">

    <record id="sms_template_data_event_registration" model="sms.template">
        <field name="name">Event: Registration</field>
        <field name="model_id" ref="event.model_event_registration"/>
        <field name="body">{{ object.event_id.organizer_id.name or object.event_id.company_id.name or user.env.company.name }}: We are happy to confirm your registration for the {{ object.event_id.name }} event.</field>
        <field name="lang">{{ object.partner_id.lang }}</field>
    </record>

    <record id="sms_template_data_event_reminder" model="sms.template">
        <field name="name">Event: Reminder</field>
        <field name="model_id" ref="event.model_event_registration"/>
        <field name="body">{{ object.event_id.organizer_id.name or object.event_id.company_id.name or user.env.company.name }}: We are excited to remind you that the {{ object.event_id.name }} event is starting {{ object.get_date_range_str() }}. We confirm your registration and hope to meet you there.</field>
        <field name="lang">{{ object.partner_id.lang }}</field>
    </record>

</data></odoo>

```

## File: models\event_mail.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class EventTypeMail(models.Model):
    _inherit = 'event.type.mail'

    @api.model
    def _selection_template_model(self):
        return super(EventTypeMail, self)._selection_template_model() + [('sms.template', 'SMS')]

    notification_type = fields.Selection(selection_add=[('sms', 'SMS')], ondelete={'sms': 'set default'})

    @api.depends('notification_type')
    def _compute_template_model_id(self):
        sms_model = self.env['ir.model']._get('sms.template')
        sms_mails = self.filtered(lambda mail: mail.notification_type == 'sms')
        sms_mails.template_model_id = sms_model
        super(EventTypeMail, self - sms_mails)._compute_template_model_id()


class EventMailScheduler(models.Model):
    _inherit = 'event.mail'

    @api.model
    def _selection_template_model(self):
        return super(EventMailScheduler, self)._selection_template_model() + [('sms.template', 'SMS')]

    def _selection_template_model_get_mapping(self):
        return {**super(EventMailScheduler, self)._selection_template_model_get_mapping(), 'sms': 'sms.template'}

    notification_type = fields.Selection(selection_add=[('sms', 'SMS')], ondelete={'sms': 'set default'})

    @api.depends('notification_type')
    def _compute_template_model_id(self):
        sms_model = self.env['ir.model']._get('sms.template')
        sms_mails = self.filtered(lambda mail: mail.notification_type == 'sms')
        sms_mails.template_model_id = sms_model
        super(EventMailScheduler, self - sms_mails)._compute_template_model_id()

    def execute(self):
        for scheduler in self:
            now = fields.Datetime.now()
            if scheduler.interval_type != 'after_sub' and scheduler.notification_type == 'sms':
                # before or after event -> one shot email
                if scheduler.mail_done:
                    continue
                # no template -> ill configured, skip and avoid crash
                if not scheduler.template_ref:
                    continue
                # Do not send SMS if the communication was scheduled before the event but the event is over
                if scheduler.scheduled_date <= now and (scheduler.interval_type != 'before_event' or scheduler.event_id.date_end > now):
                    self.env['event.registration']._message_sms_schedule_mass(
                        template=scheduler.template_ref,
                        active_domain=[('event_id', '=', scheduler.event_id.id), ('state', '!=', 'cancel')],
                        mass_keep_log=True
                    )
                    scheduler.update({
                        'mail_done': True,
                        'mail_count_done': scheduler.event_id.seats_expected,
                    })

        return super(EventMailScheduler, self).execute()

    @api.onchange('notification_type')
    def set_template_ref_model(self):
        super().set_template_ref_model()
        mail_model = self.env['sms.template']
        if self.notification_type == 'sms':
            record = mail_model.search([('model', '=', 'event.registration')], limit=1)
            self.template_ref = "{},{}".format('sms.template', record.id) if record else False


class EventMailRegistration(models.Model):
    _inherit = 'event.mail.registration'

    def execute(self):
        now = fields.Datetime.now()
        todo = self.filtered(lambda reg_mail:
            not reg_mail.mail_sent and \
            reg_mail.registration_id.state in ['open', 'done'] and \
            (reg_mail.scheduled_date and reg_mail.scheduled_date <= now) and \
            reg_mail.scheduler_id.notification_type == 'sms'
        )
        for reg_mail in todo:
            reg_mail.registration_id._message_sms_schedule_mass(
                template=reg_mail.scheduler_id.template_ref,
                mass_keep_log=True
            )
        todo.write({'mail_sent': True})

        return super(EventMailRegistration, self).execute()

```

## File: models\event_registration.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Registration(models.Model):
    _inherit = 'event.registration'

    def _sms_get_number_fields(self):
        """ This method returns the fields to use to find the number to use to
        send an SMS on a record. """
        return ['mobile', 'phone']

```

## File: models\sms_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models
from odoo.osv import expression


class SmsTemplate(models.Model):
    _inherit = 'sms.template'

    @api.model
    def _name_search(self, name, args=None, operator='ilike', limit=100, name_get_uid=None):
        """Context-based hack to filter reference field in a m2o search box to emulate a domain the ORM currently does not support.

        As we can not specify a domain on a reference field, we added a context
        key `filter_template_on_event` on the template reference field. If this
        key is set, we add our domain in the `args` in the `_name_search`
        method to filtrate the SMS templates.
        """
        if self.env.context.get('filter_template_on_event'):
            args = expression.AND([[('model', '=', 'event.registration')], args])
        return super(SmsTemplate, self)._name_search(name, args, operator, limit, name_get_uid)

    def unlink(self):
        res = super(SmsTemplate, self).unlink()
        domain = ('template_ref', 'in', [f"{template._name},{template.id}" for template in self])
        self.env['event.mail'].sudo().search([domain]).unlink()
        self.env['event.type.mail'].sudo().search([domain]).unlink()
        return res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_mail
from . import event_registration
from . import sms_template

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_sms_template_event_manager,access.sms.template.event.manager,sms.model_sms_template,event.group_event_manager,1,1,1,1

```

## File: security\sms_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="ir_rule_sms_template_event_manager" model="ir.rule">
        <field name="name">SMS Template: event manager CUD on event / registrations templates</field>
        <field name="model_id" ref="sms.model_sms_template"/>
        <field name="groups" eval="[(4, ref('event.group_event_manager'))]"/>
        <field name="domain_force">[('model_id.model', 'in', ('event.event', 'event.registration'))]</field>
        <field name="perm_read" eval="False"/>
    </record>
</odoo>

```

