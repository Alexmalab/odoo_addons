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
    'installable': True,
    'auto_install': True,
    'assets': {
        'web.assets_backend': [
            'event_sms/static/src/template_reference_field/*',
        ],
    },
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
        <field name="body">Ready for "{{ object.event_id.name }}" {{ object.event_date_range }}?
{{ 'It starts at %s' % format_time(time=object.event_begin_date, tz=object.event_id.date_tz, time_format='short', lang_code=object.partner_id.lang) + (', at %s' % object.event_id.address_inline if object.event_id.address_inline else '') + '.\nSee you there!' if object.event_id.address_inline or 'website_published' not in object.event_id._fields else 'Join us on %s/event/%i!' % (object.get_base_url(), object.event_id.id) }}</field>
        <field name="lang">{{ object.partner_id.lang }}</field>
    </record>

</data></odoo>

```

## File: models\event_mail.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class EventTypeMail(models.Model):
    _inherit = 'event.type.mail'

    notification_type = fields.Selection(selection_add=[('sms', 'SMS')])
    template_ref = fields.Reference(ondelete={'sms.template': 'cascade'}, selection_add=[('sms.template', 'SMS')])

    def _compute_notification_type(self):
        super()._compute_notification_type()
        sms_schedulers = self.filtered(lambda scheduler: scheduler.template_ref and scheduler.template_ref._name == 'sms.template')
        sms_schedulers.notification_type = 'sms'


class EventMailScheduler(models.Model):
    _inherit = 'event.mail'

    notification_type = fields.Selection(selection_add=[('sms', 'SMS')])
    template_ref = fields.Reference(ondelete={'sms.template': 'cascade'}, selection_add=[('sms.template', 'SMS')])

    def _compute_notification_type(self):
        super()._compute_notification_type()
        sms_schedulers = self.filtered(lambda scheduler: scheduler.template_ref and scheduler.template_ref._name == 'sms.template')
        sms_schedulers.notification_type = 'sms'

    def _execute_event_based_for_registrations(self, registrations):
        if self.notification_type == "sms":
            self._send_sms(registrations)
        return super()._execute_event_based_for_registrations(registrations)

    def _send_sms(self, registrations):
        """ SMS action: send SMS to attendees """
        registrations._message_sms_schedule_mass(
            template=self.template_ref,
            mass_keep_log=True
        )

    def _template_model_by_notification_type(self):
        info = super()._template_model_by_notification_type()
        info["sms"] = "sms.template"
        return info

```

## File: models\event_mail_registration.py

```python
from odoo import fields, models


class EventMailRegistration(models.Model):
    _inherit = 'event.mail.registration'

    def _execute_on_registrations(self):
        todo = self.filtered(
            lambda r: r.scheduler_id.notification_type == "sms"
        )
        for scheduler, reg_mails in todo.grouped('scheduler_id').items():
            scheduler._send_sms(reg_mails.registration_id)
        todo.mail_sent = True

        return super(EventMailRegistration, self - todo)._execute_on_registrations()

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
    def _search(self, domain, *args, **kwargs):
        """Context-based hack to filter reference field in a m2o search box to emulate a domain the ORM currently does not support.

        As we can not specify a domain on a reference field, we added a context
        key `filter_template_on_event` on the template reference field. If this
        key is set, we add our domain in the `domain` in the `_search`
        method to filtrate the SMS templates.
        """
        if self.env.context.get('filter_template_on_event'):
            domain = expression.AND([[('model', '=', 'event.registration')], domain])
        return super()._search(domain, *args, **kwargs)

    def unlink(self):
        res = super().unlink()
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
from . import event_mail_registration
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

## File: static\src\template_reference_field\field_event_mail_template_reference.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates xml:space="preserve">
    <t t-inherit="event.mailTemplateReferenceField" t-inherit-mode="extension">
        <xpath expr="//span[hasclass('event_template_reference_mail')]" position="after">
            <div t-if="relation === 'sms.template'" class="event_template_reference_sms">
                <span class="fa fa-lg fa-solid fa-mobile" role="icon"/>
                <span role="text">SMS</span>
            </div>
        </xpath>
    </t>
</templates>

```

