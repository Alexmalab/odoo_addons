# Odoo Module: sms

Category: Hidden/Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import tools
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'SMS gateway',
    'version': '3.0',
    'category': 'Hidden/Tools',
    'summary': 'SMS Text Messaging',
    'description': """
This module gives a framework for SMS text messaging
----------------------------------------------------

The service is provided by the In App Purchase Odoo platform.
""",
    'depends': [
        'base',
        'iap_mail',
        'mail',
        'phone_validation'
    ],
    'data': [
        'data/iap_service_data.xml',
        'data/ir_cron_data.xml',
        'wizard/sms_account_code_views.xml',
        'wizard/sms_account_phone_views.xml',
        'wizard/sms_account_sender_views.xml',
        'wizard/sms_composer_views.xml',
        'wizard/sms_template_preview_views.xml',
        'wizard/sms_resend_views.xml',
        'wizard/sms_template_reset_views.xml',
        'views/ir_actions_server_views.xml',
        'views/mail_notification_views.xml',
        'views/res_config_settings_views.xml',
        'views/res_partner_views.xml',
        'views/iap_account_views.xml',
        'views/sms_sms_views.xml',
        'views/sms_template_views.xml',
        'security/ir.model.access.csv',
        'security/sms_security.xml',
    ],
    'demo': [
        'data/sms_demo.xml',
        'data/mail_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
    'assets': {
        'web.assets_backend': [
            'sms/static/src/**/*',
        ],
        'web.assets_unit_tests': [
            'sms/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import re

from odoo.exceptions import UserError
from odoo.http import Controller, request, route

_logger = logging.getLogger(__name__)


class SmsController(Controller):

    @route('/sms/status', type='json', auth='public')
    def update_sms_status(self, message_statuses):
        """Receive a batch of delivery reports from IAP

        :param message_statuses:
            [
                {
                    'sms_status': status0,
                    'uuids': [uuid00, uuid01, ...],
                }, {
                    'sms_status': status1,
                    'uuids': [uuid10, uuid11, ...],
                },
                ...
            ]
        """
        all_uuids = []
        for uuids, iap_status in ((status['uuids'], status['sms_status']) for status in message_statuses):
            self._check_status_values(uuids, iap_status, message_statuses)
            if sms_trackers_sudo := request.env['sms.tracker'].sudo().search([('sms_uuid', 'in', uuids)]):
                if state := request.env['sms.sms'].IAP_TO_SMS_STATE_SUCCESS.get(iap_status):
                    sms_trackers_sudo._action_update_from_sms_state(state)
                else:
                    sms_trackers_sudo._action_update_from_provider_error(iap_status)
            all_uuids += uuids
        request.env['sms.sms'].sudo().search([('uuid', 'in', all_uuids), ('to_delete', '=', False)]).to_delete = True
        return 'OK'

    @staticmethod
    def _check_status_values(uuids, iap_status, message_statuses):
        """Basic checks to avoid unnecessary queries and allow debugging."""
        if (not uuids or not iap_status or not re.match(r'^\w+$', iap_status)
                or any(not re.match(r'^[0-9a-f]{32}$', uuid) for uuid in uuids)):
            _logger.warning('Received ill-formatted SMS delivery report event: \n%s', message_statuses)
            raise UserError("Bad parameters")

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\iap_service_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="iap_service_sms" model="iap.service">
            <field name="name">SMS</field>
            <field name="technical_name">sms</field>
            <field name="description">Send SMS to your contacts directly from your database.</field>
            <field name="unit_name">Credits</field>
            <field name="integer_balance">False</field>
        </record>
    </data>
</odoo>

```

## File: data\ir_cron_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <record forcecreate="True" id="ir_cron_sms_scheduler_action" model="ir.cron">
        <field name="name">SMS: SMS Queue Manager</field>
        <field name="model_id" ref="model_sms_sms"/>
        <field name="state">code</field>
        <field name="code">model._process_queue()</field>
        <field name="user_id" ref="base.user_root"/>
        <field name="interval_number">1</field>
        <field name="interval_type">hours</field>
    </record>
</data></odoo>
```

## File: data\mail_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <record id="message_demo_partner_1_0" model="mail.message">
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_28"/>
        <field name="body" type="html"><p>Hello! This is an example of incoming email.</p></field>
        <field name="message_type">email</field>
        <field name="subtype_id" ref="mail.mt_comment"/>
        <field name="author_id" ref="base.partner_demo"/>
        <field name="date" eval="(DateTime.today() - timedelta(days=5)).strftime('%Y-%m-%d %H:%M:00')"/>
    </record>
    <record id="message_demo_partner_1_1" model="mail.message">
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_28"/>
        <field name="body" type="html"><p>Hello! This is an example of user comment.</p></field>
        <field name="message_type">comment</field>
        <field name="subtype_id" ref="mail.mt_comment"/>
        <field name="author_id" ref="base.partner_admin"/>
        <field name="date" eval="(DateTime.today() - timedelta(days=4)).strftime('%Y-%m-%d %H:%M:00')"/>
    </record>
    <record id="message_demo_partner_1_2_notif_0" model="mail.notification">
        <field name="author_id" ref="base.partner_admin"/>
        <field name="mail_message_id" ref="message_demo_partner_1_1"/>
        <field name="res_partner_id" ref="base.res_partner_address_28"/>
        <field name="notification_type">email</field>
        <field name="notification_status">exception</field>
        <field name="failure_type">mail_smtp</field>
    </record>

    <record id="message_demo_partner_1_2" model="mail.message">
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_28"/>
        <field name="body" type="html"><p>Hello! This is an example of SMS.</p></field>
        <field name="message_type">sms</field>
        <field name="subtype_id" ref="mail.mt_note"/>
        <field name="author_id" ref="base.partner_demo"/>
        <field name="date" eval="(DateTime.today() - timedelta(days=3)).strftime('%Y-%m-%d %H:%M:00')"/>
    </record>
    <record id="message_demo_partner_1_3" model="mail.message">
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_28"/>
        <field name="body" type="html"><p>Hello! This is an example of another SMS with notifications and an unregistered account.</p></field>
        <field name="message_type">sms</field>
        <field name="subtype_id" ref="mail.mt_note"/>
        <field name="author_id" ref="base.partner_admin"/>
        <field name="date" eval="(DateTime.today() - timedelta(days=2)).strftime('%Y-%m-%d %H:%M:00')"/>
    </record>
    <record id="message_demo_partner_1_3_notif_0" model="mail.notification">
        <field name="author_id" ref="base.partner_admin"/>
        <field name="mail_message_id" ref="message_demo_partner_1_3"/>
        <field name="res_partner_id" ref="base.res_partner_address_28"/>
        <field name="notification_type">sms</field>
        <field name="notification_status">exception</field>
        <field name="failure_type">sms_acc</field>
    </record>
    <record id="message_demo_partner_1_4" model="mail.message">
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_28"/>
        <field name="body" type="html"><p>Hello! This is an example of a sent SMS with notifications.</p></field>
        <field name="message_type">sms</field>
        <field name="subtype_id" ref="mail.mt_note"/>
        <field name="author_id" ref="base.partner_admin"/>
        <field name="date" eval="(DateTime.today() - timedelta(days=1,hours=22)).strftime('%Y-%m-%d %H:%M:00')"/>
    </record>
    <record id="message_demo_partner_1_4_notif_0" model="mail.notification">
        <field name="author_id" ref="base.partner_admin"/>
        <field name="mail_message_id" ref="message_demo_partner_1_4"/>
        <field name="res_partner_id" ref="base.res_partner_address_28"/>
        <field name="notification_type">sms</field>
        <field name="notification_status">sent</field>
    </record>
    <record id="message_demo_partner_1_5" model="mail.message">
        <field name="model">res.partner</field>
        <field name="res_id" ref="base.res_partner_address_16"/>
        <field name="body" type="html"><p>Hello! This is an example of another SMS with notifications without credits.</p></field>
        <field name="message_type">sms</field>
        <field name="subtype_id" ref="mail.mt_note"/>
        <field name="author_id" ref="base.partner_admin"/>
        <field name="date" eval="(DateTime.today() - timedelta(days=1)).strftime('%Y-%m-%d %H:%M:00')"/>
    </record>
    <record id="message_demo_partner_1_5_notif_0" model="mail.notification">
        <field name="author_id" ref="base.partner_admin"/>
        <field name="mail_message_id" ref="message_demo_partner_1_5"/>
        <field name="res_partner_id" ref="base.res_partner_address_16"/>
        <field name="notification_type">sms</field>
        <field name="notification_status">exception</field>
        <field name="failure_type">sms_credit</field>
    </record>

</data></odoo>

```

## File: data\sms_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <record id="sms_template_demo_0" model="sms.template">
        <field name="name">Customer: automated SMS</field>
        <field name="model_id" ref="base.model_res_partner"/>
        <field name="body">Dear {{ object.display_name }} this is an automated SMS.</field>
    </record>
</data></odoo>

```

## File: models\iap_account.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _


class IapAccount(models.Model):
    _inherit = 'iap.account'

    sender_name = fields.Char(help="This is the name that will be displayed as the sender of the SMS.", readonly=True)

    def action_open_registration_wizard(self):
        return {
            'type': 'ir.actions.act_window',
            'target': 'new',
            'name': _('Register Account'),
            'view_mode': 'form',
            'res_model': 'sms.account.phone',
            'context': {'default_account_id': self.id},
        }

    def action_open_sender_name_wizard(self):
        return {
            'type': 'ir.actions.act_window',
            'target': 'new',
            'name': _('Choose your sender name'),
            'view_mode': 'form',
            'res_model': 'sms.account.sender',
            'context': {'default_account_id': self.id},
        }

    def _get_account_info(self, account_id, balance, information):
        res = super()._get_account_info(account_id, balance, information)
        if account_id.service_name == 'sms':
            res['sender_name'] = information.get('sender_name')
        return res

```

## File: models\ir_actions_server.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError


class ServerActions(models.Model):
    """ Add SMS option in server actions. """
    _name = 'ir.actions.server'
    _inherit = ['ir.actions.server']

    state = fields.Selection(selection_add=[
        ('sms', 'Send SMS'), ('followers',),
    ], ondelete={'sms': 'cascade'})
    # SMS
    sms_template_id = fields.Many2one(
        'sms.template', 'SMS Template',
        compute='_compute_sms_template_id',
        ondelete='set null', readonly=False, store=True,
        domain="[('model_id', '=', model_id)]",
    )
    sms_method = fields.Selection(
        selection=[('sms', 'SMS (without note)'), ('comment', 'SMS (with note)'), ('note', 'Note only')],
        string='Send SMS As',
        compute='_compute_sms_method',
        readonly=False, store=True)

    @api.depends('state')
    def _compute_available_model_ids(self):
        mail_thread_based = self.filtered(lambda action: action.state == 'sms')
        if mail_thread_based:
            mail_models = self.env['ir.model'].search([('is_mail_thread', '=', True), ('transient', '=', False)])
            for action in mail_thread_based:
                action.available_model_ids = mail_models.ids
        super(ServerActions, self - mail_thread_based)._compute_available_model_ids()

    @api.depends('model_id', 'state')
    def _compute_sms_template_id(self):
        to_reset = self.filtered(
            lambda act: act.state != 'sms' or \
                        (act.model_id != act.sms_template_id.model_id)
        )
        if to_reset:
            to_reset.sms_template_id = False

    @api.depends('state')
    def _compute_sms_method(self):
        to_reset = self.filtered(lambda act: act.state != 'sms')
        if to_reset:
            to_reset.sms_method = False
        other = self - to_reset
        if other:
            other.sms_method = 'sms'

    @api.constrains('state', 'model_id')
    def _check_sms_model_coherency(self):
        for action in self:
            if action.state == 'sms' and (action.model_id.transient or not action.model_id.is_mail_thread):
                raise ValidationError(_("Sending SMS can only be done on a not transient mail.thread model"))

    @api.constrains('model_id', 'template_id')
    def _check_sms_template_model(self):
        for action in self.filtered(lambda action: action.state == 'sms'):
            if action.sms_template_id and action.sms_template_id.model_id != action.model_id:
                raise ValidationError(
                    _('SMS template model of %(action_name)s does not match action model.',
                      action_name=action.name
                     )
                )

    def _run_action_sms_multi(self, eval_context=None):
        # TDE CLEANME: when going to new api with server action, remove action
        if not self.sms_template_id or self._is_recompute():
            return False

        records = eval_context.get('records') or eval_context.get('record')
        if not records:
            return False

        composer = self.env['sms.composer'].with_context(
            default_res_model=records._name,
            default_res_ids=records.ids,
            default_composition_mode='comment' if self.sms_method == 'comment' else 'mass',
            default_template_id=self.sms_template_id.id,
            default_mass_keep_log=self.sms_method == 'note',
        ).create({})
        composer.action_send_sms()
        return False

```

## File: models\ir_model.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class IrModel(models.Model):
    _inherit = 'ir.model'

    is_mail_thread_sms = fields.Boolean(
        string="Mail Thread SMS", default=False,
        store=False, compute='_compute_is_mail_thread_sms', search='_search_is_mail_thread_sms',
        help="Whether this model supports messages and notifications through SMS",
    )

    @api.depends('is_mail_thread')
    def _compute_is_mail_thread_sms(self):
        for model in self:
            if model.is_mail_thread:
                ModelObject = self.env[model.model]
                potential_fields = ModelObject._phone_get_number_fields() + ModelObject._mail_get_partner_fields()
                if any(fname in ModelObject._fields for fname in potential_fields):
                    model.is_mail_thread_sms = True
                    continue
            model.is_mail_thread_sms = False

    def _search_is_mail_thread_sms(self, operator, value):
        thread_models = self.search([('is_mail_thread', '=', True)])
        valid_models = self.env['ir.model']
        for model in thread_models:
            if model.model not in self.env:
                continue
            ModelObject = self.env[model.model]
            potential_fields = ModelObject._phone_get_number_fields() + ModelObject._mail_get_partner_fields()
            if any(fname in ModelObject._fields for fname in potential_fields):
                valid_models |= model

        search_sms = (operator == '=' and value) or (operator == '!=' and not value)
        if search_sms:
            return [('id', 'in', valid_models.ids)]
        return [('id', 'not in', valid_models.ids)]

```

## File: models\mail_followers.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Followers(models.Model):
    _inherit = ['mail.followers']

    def _get_recipient_data(self, records, message_type, subtype_id, pids=None):
        recipients_data = super()._get_recipient_data(records, message_type, subtype_id, pids=pids)
        if message_type != 'sms' or not (pids or records):
            return recipients_data

        if pids is None and records:
            records_pids = dict(
                (rec_id, partners.ids)
                for rec_id, partners in records._mail_get_partners().items()
            )
        elif pids and records:
            records_pids = dict((record.id, pids) for record in records)
        else:
            records_pids = {0: pids if pids else []}
        for rid, rdata in recipients_data.items():
            sms_pids = records_pids.get(rid) or []
            for pid, pdata in rdata.items():
                if pid in sms_pids:
                    pdata['notif'] = 'sms'
        return recipients_data

```

## File: models\mail_message.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class MailMessage(models.Model):
    """ Override MailMessage class in order to add a new type: SMS messages.
    Those messages comes with their own notification method, using SMS
    gateway. """
    _inherit = 'mail.message'

    message_type = fields.Selection(
        selection_add=[('sms', 'SMS')],
        ondelete={'sms': lambda recs: recs.write({'message_type': 'comment'})})
    has_sms_error = fields.Boolean(
        'Has SMS error', compute='_compute_has_sms_error', search='_search_has_sms_error')

    def _compute_has_sms_error(self):
        sms_error_from_notification = self.env['mail.notification'].sudo().search([
            ('notification_type', '=', 'sms'),
            ('mail_message_id', 'in', self.ids),
            ('notification_status', '=', 'exception')]).mapped('mail_message_id')
        for message in self:
            message.has_sms_error = message in sms_error_from_notification

    def _search_has_sms_error(self, operator, operand):
        if operator == '=' and operand:
            return ['&', ('notification_ids.notification_status', '=', 'exception'), ('notification_ids.notification_type', '=', 'sms')]
        raise NotImplementedError()

```

## File: models\mail_notification.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class MailNotification(models.Model):
    _inherit = 'mail.notification'

    notification_type = fields.Selection(selection_add=[
        ('sms', 'SMS')
    ], ondelete={'sms': 'cascade'})
    sms_id_int = fields.Integer('SMS ID', index='btree_not_null')
    # Used to give links on form view without foreign key. In most cases, you'd want to use sms_id_int or sms_tracker_ids.sms_uuid.
    sms_id = fields.Many2one('sms.sms', string='SMS', store=False, compute='_compute_sms_id')
    sms_tracker_ids = fields.One2many('sms.tracker', 'mail_notification_id', string="SMS Trackers")
    sms_number = fields.Char('SMS Number')
    failure_type = fields.Selection(selection_add=[
        ('sms_number_missing', 'Missing Number'),
        ('sms_number_format', 'Wrong Number Format'),
        ('sms_credit', 'Insufficient Credit'),
        ('sms_country_not_supported', 'Country Not Supported'),
        ('sms_registration_needed', 'Country-specific Registration Required'),
        ('sms_server', 'Server Error'),
        ('sms_acc', 'Unregistered Account'),
        # delivery report errors
        ('sms_expired', 'Expired'),
        ('sms_invalid_destination', 'Invalid Destination'),
        ('sms_not_allowed', 'Not Allowed'),
        ('sms_not_delivered', 'Not Delivered'),
        ('sms_rejected', 'Rejected'),
    ])

    @api.depends('sms_id_int', 'notification_type')
    def _compute_sms_id(self):
        self.sms_id = False
        sms_notifications = self.filtered(lambda n: n.notification_type == 'sms' and bool(n.sms_id_int))
        if not sms_notifications:
            return
        existing_sms_ids = self.env['sms.sms'].sudo().search([
            ('id', 'in', sms_notifications.mapped('sms_id_int')), ('to_delete', '!=', True)
        ]).ids
        for sms_notification in sms_notifications.filtered(lambda n: n.sms_id_int in set(existing_sms_ids)):
            sms_notification.sms_id = sms_notification.sms_id_int

```

## File: models\mail_thread.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, Command, models, fields
from odoo.addons.sms.tools.sms_tools import sms_content_to_rendered_html
from odoo.tools import html2plaintext

_logger = logging.getLogger(__name__)


class MailThread(models.AbstractModel):
    _inherit = 'mail.thread'

    message_has_sms_error = fields.Boolean(
        'SMS Delivery error', compute='_compute_message_has_sms_error', search='_search_message_has_sms_error',
        help="If checked, some messages have a delivery error.")

    def _compute_message_has_sms_error(self):
        res = {}
        if self.ids:
            self.env.cr.execute("""
                    SELECT msg.res_id, COUNT(msg.res_id)
                      FROM mail_message msg
                INNER JOIN mail_notification notif
                        ON notif.mail_message_id = msg.id
                     WHERE notif.notification_type = 'sms'
                       AND notif.notification_status = 'exception'
                       AND notif.author_id = %(author_id)s
                       AND msg.model = %(model_name)s
                       AND msg.res_id in %(res_ids)s
                       AND msg.message_type != 'user_notification'
                  GROUP BY msg.res_id
            """, {'author_id': self.env.user.partner_id.id, 'model_name': self._name, 'res_ids': tuple(self.ids)})
            res.update(self._cr.fetchall())

        for record in self:
            record.message_has_sms_error = bool(res.get(record._origin.id, 0))

    @api.model
    def _search_message_has_sms_error(self, operator, operand):
        return ['&', ('message_ids.has_sms_error', operator, operand), ('message_ids.author_id', '=', self.env.user.partner_id.id)]

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, *args, body='', message_type='notification', **kwargs):
        # When posting an 'SMS' `message_type`, make sure that the body is used as-is in the sms,
        # and reformat the message body for the notification (mainly making URLs clickable).
        if message_type == 'sms':
            kwargs['sms_content'] = body
            body = sms_content_to_rendered_html(body)
        return super().message_post(*args, body=body, message_type=message_type, **kwargs)

    def _message_sms_schedule_mass(self, body='', template=False, **composer_values):
        """ Shortcut method to schedule a mass sms sending on a recordset.

        :param template: an optional sms.template record;
        """
        composer_context = {
            'default_res_model': self._name,
            'default_composition_mode': 'mass',
            'default_template_id': template.id if template else False,
            'default_res_ids': self.ids,
        }
        if body and not template:
            composer_context['default_body'] = body

        create_vals = {
            'mass_force_send': False,
            'mass_keep_log': True,
        }
        if composer_values:
            create_vals.update(composer_values)

        composer = self.env['sms.composer'].with_context(**composer_context).create(create_vals)
        return composer._action_send_sms()

    def _message_sms_with_template(self, template=False, template_xmlid=False, template_fallback='', partner_ids=False, **kwargs):
        """ Shortcut method to perform a _message_sms with an sms.template.

        :param template: a valid sms.template record;
        :param template_xmlid: XML ID of an sms.template (if no template given);
        :param template_fallback: plaintext (inline_template-enabled) in case template
          and template xml id are falsy (for example due to deleted data);
        """
        self.ensure_one()
        if not template and template_xmlid:
            template = self.env.ref(template_xmlid, raise_if_not_found=False)
        if template:
            body = template._render_field('body', self.ids, compute_lang=True)[self.id]
        else:
            body = self.env['sms.template']._render_template(template_fallback, self._name, self.ids)[self.id]
        return self._message_sms(body, partner_ids=partner_ids, **kwargs)

    def _message_sms(self, body, subtype_id=False, partner_ids=False, number_field=False,
                     sms_numbers=None, sms_pid_to_number=None, **kwargs):
        """ Main method to post a message on a record using SMS-based notification
        method.

        :param body: content of SMS;
        :param subtype_id: mail.message.subtype used in mail.message associated
          to the sms notification process;
        :param partner_ids: if set is a record set of partners to notify;
        :param number_field: if set is a name of field to use on current record
          to compute a number to notify;
        :param sms_numbers: see ``_notify_thread_by_sms``;
        :param sms_pid_to_number: see ``_notify_thread_by_sms``;
        """
        self.ensure_one()
        sms_pid_to_number = sms_pid_to_number if sms_pid_to_number is not None else {}

        if number_field or (partner_ids is False and sms_numbers is None):
            info = self._sms_get_recipients_info(force_field=number_field)[self.id]
            info_partner_ids = info['partner'].ids if info['partner'] else False
            info_number = info['sanitized'] if info['sanitized'] else info['number']
            if info_partner_ids and info_number:
                sms_pid_to_number[info_partner_ids[0]] = info_number
            if info_partner_ids:
                partner_ids = info_partner_ids + (partner_ids or [])
            if not info_partner_ids:
                if info_number:
                    sms_numbers = [info_number] + (sms_numbers or [])
                    # will send a falsy notification allowing to fix it through SMS wizards
                elif not sms_numbers:
                    sms_numbers = [False]

        if subtype_id is False:
            subtype_id = self.env['ir.model.data']._xmlid_to_res_id('mail.mt_note')

        return self.message_post(
            body=body, partner_ids=partner_ids or [],  # TDE FIXME: temp fix otherwise crash mail_thread.py
            message_type='sms', subtype_id=subtype_id,
            sms_numbers=sms_numbers, sms_pid_to_number=sms_pid_to_number,
            **kwargs
        )

    def _notify_thread(self, message, msg_vals=False, **kwargs):
        scheduled_date = self._is_notification_scheduled(kwargs.get('scheduled_date'))
        recipients_data = super(MailThread, self)._notify_thread(message, msg_vals=msg_vals, **kwargs)
        if not scheduled_date:
            self._notify_thread_by_sms(message, recipients_data, msg_vals=msg_vals, **kwargs)
        return recipients_data

    def _notify_thread_by_sms(self, message, recipients_data, msg_vals=False,
                              sms_content=None, sms_numbers=None, sms_pid_to_number=None,
                              resend_existing=False, put_in_queue=False, **kwargs):
        """ Notification method: by SMS.

        :param message: ``mail.message`` record to notify;
        :param recipients_data: list of recipients information (based on res.partner
          records), formatted like
            [{'active': partner.active;
              'id': id of the res.partner being recipient to notify;
              'groups': res.group IDs if linked to a user;
              'notif': 'inbox', 'email', 'sms' (SMS App);
              'share': partner.partner_share;
              'type': 'customer', 'portal', 'user;'
             }, {...}].
          See ``MailThread._notify_get_recipients``;
        :param msg_vals: dictionary of values used to create the message. If given it
          may be used to access values related to ``message`` without accessing it
          directly. It lessens query count in some optimized use cases by avoiding
          access message content in db;

        :param sms_content: plaintext version of body, mainly to avoid
          conversion glitches by splitting html and plain text content formatting
          (e.g.: links, styling.).
          If not given, `msg_vals`'s `body` is used and converted from html to plaintext;
        :param sms_numbers: additional numbers to notify in addition to partners
          and classic recipients;
        :param pid_to_number: force a number to notify for a given partner ID
          instead of taking its mobile / phone number;
        :param resend_existing: check for existing notifications to update based on
          mailed recipient, otherwise create new notifications;
        :param put_in_queue: use cron to send queued SMS instead of sending them
          directly;
        """
        sms_pid_to_number = sms_pid_to_number if sms_pid_to_number is not None else {}
        sms_numbers = sms_numbers if sms_numbers is not None else []
        sms_create_vals = []
        sms_all = self.env['sms.sms'].sudo()

        # pre-compute SMS data
        body = sms_content or html2plaintext(msg_vals['body'] if msg_vals and 'body' in msg_vals else message.body)
        sms_base_vals = {
            'body': body,
            'mail_message_id': message.id,
            'state': 'outgoing',
        }

        # notify from computed recipients_data (followers, specific recipients)
        partners_data = [r for r in recipients_data if r['notif'] == 'sms']
        partner_ids = [r['id'] for r in partners_data]
        if partner_ids:
            for partner in self.env['res.partner'].sudo().browse(partner_ids):
                number = sms_pid_to_number.get(partner.id) or partner.mobile or partner.phone
                sms_create_vals.append(dict(
                    sms_base_vals,
                    partner_id=partner.id,
                    number=partner._phone_format(number=number) or number,
                ))

        # notify from additional numbers
        if sms_numbers:
            tocreate_numbers = [
                self._phone_format(number=sms_number) or sms_number
                for sms_number in sms_numbers
            ]
            existing_partners_numbers = {vals_dict['number'] for vals_dict in sms_create_vals}
            sms_create_vals += [dict(
                sms_base_vals,
                partner_id=False,
                number=n,
                state='outgoing' if n else 'error',
                failure_type='' if n else 'sms_number_missing',
            ) for n in tocreate_numbers if n not in existing_partners_numbers]

        # create sms and notification
        existing_pids, existing_numbers = [], []
        if sms_create_vals:
            sms_all |= self.env['sms.sms'].sudo().create(sms_create_vals)

            if resend_existing:
                existing = self.env['mail.notification'].sudo().search([
                    '|', ('res_partner_id', 'in', partner_ids),
                    '&', ('res_partner_id', '=', False), ('sms_number', 'in', sms_numbers),
                    ('notification_type', '=', 'sms'),
                    ('mail_message_id', 'in', message.ids),
                ])
                for n in existing:
                    if n.res_partner_id.id in partner_ids and n.mail_message_id == message:
                        existing_pids.append(n.res_partner_id.id)
                    if not n.res_partner_id and n.sms_number in sms_numbers and n.mail_message_id == message:
                        existing_numbers.append(n.sms_number)

            notif_create_values = [{
                'author_id': message.author_id.id,
                'mail_message_id': message.id,
                'res_partner_id': sms.partner_id.id,
                'sms_number': sms.number,
                'notification_type': 'sms',
                'sms_id_int': sms.id,
                'sms_tracker_ids': [Command.create({'sms_uuid': sms.uuid})] if sms.state == 'outgoing' else False,
                'is_read': True,  # discard Inbox notification
                'notification_status': 'ready' if sms.state == 'outgoing' else 'exception',
                'failure_type': '' if sms.state == 'outgoing' else sms.failure_type,
            } for sms in sms_all if (sms.partner_id and sms.partner_id.id not in existing_pids) or (not sms.partner_id and sms.number not in existing_numbers)]
            if notif_create_values:
                self.env['mail.notification'].sudo().create(notif_create_values)

            if existing_pids or existing_numbers:
                for sms in sms_all:
                    notif = next((n for n in existing if
                                 (n.res_partner_id.id in existing_pids and n.res_partner_id.id == sms.partner_id.id) or
                                 (not n.res_partner_id and n.sms_number in existing_numbers and n.sms_number == sms.number)), False)
                    if notif:
                        notif.write({
                            'notification_type': 'sms',
                            'notification_status': 'ready',
                            'sms_id_int': sms.id,
                            'sms_tracker_ids': [Command.create({'sms_uuid': sms.uuid})],
                            'sms_number': sms.number,
                        })

        if sms_all and not put_in_queue:
            sms_all.filtered(lambda sms: sms.state == 'outgoing').send(auto_commit=False, raise_exception=False)

        return True

    def _get_notify_valid_parameters(self):
        return super()._get_notify_valid_parameters() | {
            'put_in_queue', 'sms_numbers', 'sms_pid_to_number', 'sms_content',
        }

    @api.model
    def notify_cancel_by_type(self, notification_type):
        super().notify_cancel_by_type(notification_type)
        if notification_type == 'sms':
            # TDE CHECK: delete pending SMS
            self._notify_cancel_by_type_generic('sms')
        return True

```

## File: models\models.py

```python
from odoo import models
from odoo.addons.phone_validation.tools import phone_validation


class BaseModel(models.AbstractModel):
    _inherit = 'base'

    def _sms_get_recipients_info(self, force_field=False, partner_fallback=True):
        """" Get SMS recipient information on current record set. This method
        checks for numbers and sanitation in order to centralize computation.

        Example of use cases

          * click on a field -> number is actually forced from field, find customer
            linked to record, force its number to field or fallback on customer fields;
          * contact -> find numbers from all possible phone fields on record, find
            customer, force its number to found field number or fallback on customer fields;

        :param force_field: either give a specific field to find phone number, either
            generic heuristic is used to find one based on ``_phone_get_number_fields``;
        :param partner_fallback: if no value found in the record, check its customer
            values based on ``_mail_get_partners``;

        :return dict: record.id: {
            'partner': a res.partner recordset that is the customer (void or singleton)
                linked to the recipient. See ``_mail_get_partners``;
            'sanitized': sanitized number to use (coming from record's field or partner's
                phone fields). Set to False is number impossible to parse and format;
            'number': original number before sanitation;
            'partner_store': whether the number comes from the customer phone fields. If
                False it means number comes from the record itself, even if linked to a
                customer;
            'field_store': field in which the number has been found (generally mobile or
                phone, see ``_phone_get_number_fields``);
        } for each record in self
        """
        result = dict.fromkeys(self.ids, False)
        tocheck_fields = [force_field] if force_field else self._phone_get_number_fields()
        for record in self:
            all_numbers = [record[fname] for fname in tocheck_fields if fname in record]
            all_partners = record._mail_get_partners()[record.id]

            valid_number, fname = False, False
            for fname in [f for f in tocheck_fields if f in record]:
                valid_number = record._phone_format(fname=fname)
                if valid_number:
                    break

            if valid_number:
                result[record.id] = {
                    'partner': all_partners[0] if all_partners else self.env['res.partner'],
                    'sanitized': valid_number,
                    'number': record[fname],
                    'partner_store': False,
                    'field_store': fname,
                }
            elif all_partners and partner_fallback:
                partner = self.env['res.partner']
                for partner in all_partners:
                    for fname in self.env['res.partner']._phone_get_number_fields():
                        valid_number = partner._phone_format(fname=fname)
                        if valid_number:
                            break

                if not valid_number:
                    fname = 'mobile' if partner.mobile else ('phone' if partner.phone else 'mobile')

                result[record.id] = {
                    'partner': partner,
                    'sanitized': valid_number if valid_number else False,
                    'number': partner[fname],
                    'partner_store': True,
                    'field_store': fname,
                }
            else:
                # did not find any sanitized number -> take first set value as fallback;
                # if none, just assign False to the first available number field
                value, fname = next(
                    ((value, fname) for value, fname in zip(all_numbers, tocheck_fields) if value),
                    (False, tocheck_fields[0] if tocheck_fields else False)
                )
                result[record.id] = {
                    'partner': self.env['res.partner'],
                    'sanitized': False,
                    'number': value,
                    'partner_store': False,
                    'field_store': fname
                }
        return result

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ResPartner(models.Model):
    _name = 'res.partner'
    _inherit = ['mail.thread.phone', 'res.partner']

```

## File: models\sms_sms.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import threading
from uuid import uuid4

from werkzeug.urls import url_join

from odoo import api, fields, models, tools, _
from odoo.addons.sms.tools.sms_api import SmsApi

_logger = logging.getLogger(__name__)


class SmsSms(models.Model):
    _name = 'sms.sms'
    _description = 'Outgoing SMS'
    _rec_name = 'number'
    _order = 'id DESC'

    IAP_TO_SMS_STATE_SUCCESS = {
        'processing': 'process',
        'success': 'pending',
        # These below are not returned in responses from IAP API in _send but are received via webhook events.
        'sent': 'pending',
        'delivered': 'sent',
    }
    IAP_TO_SMS_FAILURE_TYPE = {
        'insufficient_credit': 'sms_credit',
        'wrong_number_format': 'sms_number_format',
        'country_not_supported': 'sms_country_not_supported',
        'server_error': 'sms_server',
        'unregistered': 'sms_acc'
    }

    BOUNCE_DELIVERY_ERRORS = {'sms_invalid_destination', 'sms_not_allowed', 'sms_rejected'}
    DELIVERY_ERRORS = {'sms_expired', 'sms_not_delivered', *BOUNCE_DELIVERY_ERRORS}

    uuid = fields.Char('UUID', copy=False, readonly=True, default=lambda self: uuid4().hex,
                       help='Alternate way to identify a SMS record, used for delivery reports')
    number = fields.Char('Number')
    body = fields.Text()
    partner_id = fields.Many2one('res.partner', 'Customer')
    mail_message_id = fields.Many2one('mail.message', index=True)
    state = fields.Selection([
        ('outgoing', 'In Queue'),
        ('process', 'Processing'),
        ('pending', 'Sent'),
        ('sent', 'Delivered'),  # As for notifications and traces
        ('error', 'Error'),
        ('canceled', 'Cancelled')
    ], 'SMS Status', readonly=True, copy=False, default='outgoing', required=True)
    failure_type = fields.Selection([
        ("unknown", "Unknown error"),
        ('sms_number_missing', 'Missing Number'),
        ('sms_number_format', 'Wrong Number Format'),
        ('sms_country_not_supported', 'Country Not Supported'),
        ('sms_registration_needed', 'Country-specific Registration Required'),
        ('sms_credit', 'Insufficient Credit'),
        ('sms_server', 'Server Error'),
        ('sms_acc', 'Unregistered Account'),
        # mass mode specific codes, generated internally, not returned by IAP.
        ('sms_blacklist', 'Blacklisted'),
        ('sms_duplicate', 'Duplicate'),
        ('sms_optout', 'Opted Out'),
    ], copy=False)
    sms_tracker_id = fields.Many2one('sms.tracker', string='SMS trackers', compute='_compute_sms_tracker_id')
    to_delete = fields.Boolean(
        'Marked for deletion', default=False,
        help='Will automatically be deleted, while notifications will not be deleted in any case.'
    )

    _sql_constraints = [
        ('uuid_unique', 'unique(uuid)', 'UUID must be unique'),
    ]

    @api.depends('uuid')
    def _compute_sms_tracker_id(self):
        self.sms_tracker_id = False
        existing_trackers = self.env['sms.tracker'].search([('sms_uuid', 'in', self.filtered('uuid').mapped('uuid'))])
        tracker_ids_by_sms_uuid = {tracker.sms_uuid: tracker.id for tracker in existing_trackers}
        for sms in self.filtered(lambda s: s.uuid in tracker_ids_by_sms_uuid):
            sms.sms_tracker_id = tracker_ids_by_sms_uuid[sms.uuid]

    def action_set_canceled(self):
        self._update_sms_state_and_trackers('canceled')

    def action_set_error(self, failure_type):
        self._update_sms_state_and_trackers('error', failure_type=failure_type)

    def action_set_outgoing(self):
        self._update_sms_state_and_trackers('outgoing', failure_type=False)

    def send(self, unlink_failed=False, unlink_sent=True, auto_commit=False, raise_exception=False):
        """ Main API method to send SMS.

          :param unlink_failed: unlink failed SMS after IAP feedback;
          :param unlink_sent: unlink sent SMS after IAP feedback;
          :param auto_commit: commit after each batch of SMS;
          :param raise_exception: raise if there is an issue contacting IAP;
        """
        self = self.filtered(lambda sms: sms.state == 'outgoing' and not sms.to_delete)
        for batch_ids in self._split_batch():
            self.browse(batch_ids)._send(unlink_failed=unlink_failed, unlink_sent=unlink_sent, raise_exception=raise_exception)
            # auto-commit if asked except in testing mode
            if auto_commit is True and not getattr(threading.current_thread(), 'testing', False):
                self._cr.commit()

    def resend_failed(self):
        sms_to_send = self.filtered(lambda sms: sms.state == 'error' and not sms.to_delete)
        sms_to_send.state = 'outgoing'
        notification_title = _('Warning')
        notification_type = 'danger'

        if sms_to_send:
            sms_to_send.send()
            success_sms = len(sms_to_send) - len(sms_to_send.exists())
            if success_sms > 0:
                notification_title = _('Success')
                notification_type = 'success'
                notification_message = _('%(count)s out of the %(total)s selected SMS Text Messages have successfully been resent.', count=success_sms, total=len(self))
            else:
                notification_message = _('The SMS Text Messages could not be resent.')
        else:
            notification_message = _('There are no SMS Text Messages to resend.')
        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'title': notification_title,
                'message': notification_message,
                'type': notification_type,
            }
        }

    @api.model
    def _process_queue(self, ids=None):
        """ Send immediately queued messages, committing after each message is sent.
        This is not transactional and should not be called during another transaction!

       :param list ids: optional list of emails ids to send. If passed no search
         is performed, and these ids are used instead.
        """
        domain = [('state', '=', 'outgoing'), ('to_delete', '!=', True)]

        filtered_ids = self.search(domain, limit=10000).ids  # TDE note: arbitrary limit we might have to update
        if ids:
            ids = list(set(filtered_ids) & set(ids))
        else:
            ids = filtered_ids
        ids.sort()

        res = None
        try:
            # auto-commit except in testing mode
            auto_commit = not getattr(threading.current_thread(), 'testing', False)
            res = self.browse(ids).send(unlink_failed=False, unlink_sent=True, auto_commit=auto_commit, raise_exception=False)
        except Exception:
            _logger.exception("Failed processing SMS queue")
        return res

    def _split_batch(self):
        batch_size = int(self.env['ir.config_parameter'].sudo().get_param('sms.session.batch.size', 500))
        for sms_batch in tools.split_every(batch_size, self.ids):
            yield sms_batch

    def _send(self, unlink_failed=False, unlink_sent=True, raise_exception=False):
        """Send SMS after checking the number (presence and formatting)."""
        messages = [{
            'content': body,
            'numbers': [{'number': sms.number, 'uuid': sms.uuid} for sms in body_sms_records],
        } for body, body_sms_records in self.grouped('body').items()]

        delivery_reports_url = url_join(self[0].get_base_url(), '/sms/status')
        try:
            results = SmsApi(self.env)._send_sms_batch(messages, delivery_reports_url=delivery_reports_url)
        except Exception as e:
            _logger.info('Sent batch %s SMS: %s: failed with exception %s', len(self.ids), self.ids, e)
            if raise_exception:
                raise
            results = [{'uuid': sms.uuid, 'state': 'server_error'} for sms in self]
        else:
            _logger.info('Send batch %s SMS: %s: gave %s', len(self.ids), self.ids, results)

        results_uuids = [result['uuid'] for result in results]
        all_sms_sudo = self.env['sms.sms'].sudo().search([('uuid', 'in', results_uuids)]).with_context(sms_skip_msg_notification=True)

        for iap_state, results_group in tools.groupby(results, key=lambda result: result['state']):
            sms_sudo = all_sms_sudo.filtered(lambda s: s.uuid in {result['uuid'] for result in results_group})
            if success_state := self.IAP_TO_SMS_STATE_SUCCESS.get(iap_state):
                sms_sudo.sms_tracker_id._action_update_from_sms_state(success_state)
                to_delete = {'to_delete': True} if unlink_sent else {}
                sms_sudo.write({'state': success_state, 'failure_type': False, **to_delete})
            else:
                failure_type = self.IAP_TO_SMS_FAILURE_TYPE.get(iap_state, 'unknown')
                if failure_type != 'unknown':
                    sms_sudo.sms_tracker_id._action_update_from_sms_state('error', failure_type=failure_type)
                else:
                    sms_sudo.sms_tracker_id._action_update_from_provider_error(iap_state)
                to_delete = {'to_delete': True} if unlink_failed else {}
                sms_sudo.write({'state': 'error', 'failure_type': failure_type, **to_delete})

        all_sms_sudo.mail_message_id._notify_message_notification_update()

    def _update_sms_state_and_trackers(self, new_state, failure_type=None):
        """Update sms state update and related tracking records (notifications, traces)."""
        self.write({'state': new_state, 'failure_type': failure_type})
        self.sms_tracker_id._action_update_from_sms_state(new_state, failure_type=failure_type)

    @api.autovacuum
    def _gc_device(self):
        self._cr.execute("DELETE FROM sms_sms WHERE to_delete = TRUE")
        _logger.info("GC'd %d sms marked for deletion", self._cr.rowcount)

```

## File: models\sms_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class SMSTemplate(models.Model):
    "Templates for sending SMS"
    _name = "sms.template"
    _inherit = ['mail.render.mixin', 'template.reset.mixin']
    _description = 'SMS Templates'

    _unrestricted_rendering = True

    @api.model
    def default_get(self, fields):
        res = super().default_get(fields)
        if 'model_id' in fields and not res.get('model_id') and res.get('model'):
            res['model_id'] = self.env['ir.model']._get(res['model']).id
        return res

    name = fields.Char('Name', translate=True)
    model_id = fields.Many2one(
        'ir.model', string='Applies to', required=True,
        domain=['&', ('is_mail_thread_sms', '=', True), ('transient', '=', False)],
        help="The type of document this template can be used with", ondelete='cascade')
    model = fields.Char('Related Document Model', related='model_id.model', index=True, store=True, readonly=True)
    body = fields.Char('Body', translate=True, required=True)
    # Use to create contextual action (same as for email template)
    sidebar_action_id = fields.Many2one('ir.actions.act_window', 'Sidebar action', readonly=True, copy=False,
                                        help="Sidebar action to make this template available on records "
                                        "of the related document model")

    # Overrides of mail.render.mixin
    @api.depends('model')
    def _compute_render_model(self):
        for template in self:
            template.render_model = template.model

    # ------------------------------------------------------------
    # CRUD
    # ------------------------------------------------------------

    def copy_data(self, default=None):
        vals_list = super().copy_data(default=default)
        return [dict(vals, name=self.env._("%s (copy)", template.name)) for template, vals in zip(self, vals_list)]

    def unlink(self):
        self.sudo().mapped('sidebar_action_id').unlink()
        return super(SMSTemplate, self).unlink()

    def action_create_sidebar_action(self):
        ActWindow = self.env['ir.actions.act_window']
        view = self.env.ref('sms.sms_composer_view_form')

        for template in self:
            button_name = _('Send SMS (%s)', template.name)
            action = ActWindow.create({
                'name': button_name,
                'type': 'ir.actions.act_window',
                'res_model': 'sms.composer',
                # Add default_composition_mode to guess to determine if need to use mass or comment composer
                'context': "{'default_template_id' : %d, 'sms_composition_mode': 'guess', 'default_res_ids': active_ids, 'default_res_id': active_id}" % (template.id),
                'view_mode': 'form',
                'view_id': view.id,
                'target': 'new',
                'binding_model_id': template.model_id.id,
            })
            template.write({'sidebar_action_id': action.id})
        return True

    def action_unlink_sidebar_action(self):
        for template in self:
            if template.sidebar_action_id:
                template.sidebar_action_id.unlink()
        return True

```

## File: models\sms_tracker.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class SmsTracker(models.Model):
    """Relationship between a sent SMS and tracking records such as notifications and traces.

    This model acts as an extension of a `mail.notification` or a `mailing.trace` and allows to
    update those based on the SMS provider responses both at sending and when later receiving
    sent/delivery reports (see `SmsController`).
    SMS trackers are supposed to be created manually when necessary, and tied to their related
    SMS through the SMS UUID field. (They are not tied to the SMS records directly as those can
    be deleted when sent).

    Note: Only admins/system user should need to access (a fortiori modify) these technical
      records so no "sudo" is used nor should be required here.
    """
    _name = 'sms.tracker'
    _description = "Link SMS to mailing/sms tracking models"

    SMS_STATE_TO_NOTIFICATION_STATUS = {
        'canceled': 'canceled',
        'process': 'process',
        'error': 'exception',
        'outgoing': 'ready',
        'sent': 'sent',
        'pending': 'pending',
    }

    sms_uuid = fields.Char('SMS uuid', required=True)
    mail_notification_id = fields.Many2one('mail.notification', ondelete='cascade')

    _sql_constraints = [
        ('sms_uuid_unique', 'unique(sms_uuid)', 'A record for this UUID already exists'),
    ]

    def _action_update_from_provider_error(self, provider_error):
        """
        :param str provider_error: value returned by SMS service provider (IAP) or any string.
            If provided, notification values will be derived from it.
            (see ``_get_tracker_values_from_provider_error``)
        """
        failure_reason = False
        failure_type = f'sms_{provider_error}'
        error_status = None
        if failure_type not in self.env['sms.sms'].DELIVERY_ERRORS:
            failure_type = 'unknown'
            failure_reason = provider_error
        elif failure_type in self.env['sms.sms'].BOUNCE_DELIVERY_ERRORS:
            error_status = "bounce"

        self._update_sms_notifications(error_status or 'exception', failure_type=failure_type, failure_reason=failure_reason)
        return error_status, failure_type, failure_reason

    def _action_update_from_sms_state(self, sms_state, failure_type=False, failure_reason=False):
        notification_status = self.SMS_STATE_TO_NOTIFICATION_STATUS[sms_state]
        self._update_sms_notifications(notification_status, failure_type=failure_type, failure_reason=failure_reason)

    def _update_sms_notifications(self, notification_status, failure_type=False, failure_reason=False):
        # canceled is a state which means that the SMS sending order should not be sent to the SMS service.
        # `process`, `pending` are sent to IAP which is not revertible (as `sent` which means "delivered").
        notifications_statuses_to_ignore = {
            'canceled': ['canceled', 'process', 'pending', 'sent'],
            'ready': ['ready', 'process', 'pending', 'sent'],
            'process': ['process', 'pending', 'sent'],
            'pending': ['pending', 'sent'],
            'bounce': ['bounce', 'sent'],
            'sent': ['sent'],
            'exception': ['exception'],
        }[notification_status]
        notifications = self.mail_notification_id.filtered(
            lambda n: n.notification_status not in notifications_statuses_to_ignore
        )
        if notifications:
            notifications.write({
                'notification_status': notification_status,
                'failure_type': failure_type,
                'failure_reason': failure_reason,
            })
            if not self.env.context.get('sms_skip_msg_notification'):
                notifications.mail_message_id._notify_message_notification_update()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import iap_account
from . import ir_actions_server
from . import ir_model
from . import mail_followers
from . import mail_message
from . import mail_notification
from . import mail_thread
from . import models
from . import res_partner
from . import sms_sms
from . import sms_template
from . import sms_tracker

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_sms_sms_all,access.sms.sms.all,model_sms_sms,,0,0,0,0
access_sms_sms_system,access.sms.sms.system,model_sms_sms,base.group_system,1,1,1,1
access_sms_template_all,access.sms.template.all,model_sms_template,,0,0,0,0
access_sms_template_user,access.sms.template.user,model_sms_template,base.group_user,1,0,0,0
access_sms_template_system,access.sms.template.system,model_sms_template,base.group_system,1,1,1,1
access_sms_tracker_all,access.sms.tracker.all,model_sms_tracker,,0,0,0,0
access_sms_tracker_system,access.sms.tracker.system,model_sms_tracker,base.group_system,1,1,1,1
access_sms_composer,access.sms.composer,model_sms_composer,base.group_user,1,1,1,0
access_sms_resend_recipient,access.sms.resend.recipient,model_sms_resend_recipient,base.group_user,1,1,1,0
access_sms_resend,access.sms.resend,model_sms_resend,base.group_user,1,1,1,0
access_sms_template_preview,access.sms.template.preview,model_sms_template_preview,base.group_user,1,1,1,0
access_sms_template_reset,access.sms.template.reset,model_sms_template_reset,mail.group_mail_template_editor,1,1,1,1
access_sms_account_registration_phone_number_wizard_system,access.sms.account.phone.system,model_sms_account_phone,base.group_system,1,1,1,1
access_sms_account_verification_code_wizard_system,access.sms.account.code.system,model_sms_account_code,base.group_system,1,1,1,1
access_sms_account_sender_name_wizard_system,access.sms.account.sender.system,model_sms_account_sender,base.group_system,1,1,1,1

```

## File: security\sms_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_rule_sms_template_system" model="ir.rule">
        <field name="name">SMS Template: system group granted all</field>
        <field name="model_id" ref="sms.model_sms_template"/>
        <field name="groups" eval="[(4, ref('base.group_system'))]"/>
        <field name="domain_force">[(1, '=', 1)]</field>
    </record>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M13 8a4 4 0 0 1 4-4h16a4 4 0 0 1 4 4v34a4 4 0 0 1-4 4H17a4 4 0 0 1-4-4V8Z" fill="#1AD3BB"/><path d="M37 19v18a9 9 0 1 1 0-18Z" fill="#1A6F66"/><path d="M13 31V13a9 9 0 1 1 0 18Z" fill="#005E7A"/><path d="M13 13H4v9a9 9 0 0 0 9 9V13Z" fill="#985184"/><path d="M37 37h9v-9a9 9 0 0 0-9-9v18Z" fill="#FC868B"/></svg>

```

## File: static\img\sms_failure.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="512" height="512"><defs><path id="a" d="M91.974 123.535v-17.07c0-.839-.283-1.542-.851-2.111-.568-.57-1.24-.854-2.017-.854H71.894c-.777 0-1.449.285-2.017.854-.568.569-.851 1.272-.851 2.11v17.071c0 .839.283 1.542.851 2.111.568.57 1.24.854 2.017.854h17.212c.777 0 1.449-.285 2.017-.854.568-.569.851-1.272.851-2.11zm-.179-33.601l1.614-41.239c0-.718-.3-1.287-.897-1.707-.777-.659-1.494-.988-2.151-.988H70.639c-.657 0-1.374.33-2.151.988-.598.42-.897 1.048-.897 1.887l1.524 41.059c0 .599.3 1.093.897 1.482.597.39 1.314.584 2.151.584h16.584c.837 0 1.54-.195 2.107-.584.568-.39.881-.883.941-1.482zM90.54 6.02l68.846 126.5c2.092 3.773 2.032 7.546-.179 11.32a11.276 11.276 0 0 1-4.168 4.133 11.192 11.192 0 0 1-5.693 1.527H11.654c-2.032 0-3.93-.51-5.693-1.527a11.276 11.276 0 0 1-4.168-4.133c-2.211-3.774-2.271-7.547-.18-11.32L70.46 6.02a11.462 11.462 0 0 1 4.213-4.403A11.105 11.105 0 0 1 80.5 0c2.092 0 4.034.54 5.827 1.617A11.462 11.462 0 0 1 90.54 6.02z"/><path id="c" d="M91.974 123.535v-17.07c0-.839-.283-1.542-.851-2.111-.568-.57-1.24-.854-2.017-.854H71.894c-.777 0-1.449.285-2.017.854-.568.569-.851 1.272-.851 2.11v17.071c0 .839.283 1.542.851 2.111.568.57 1.24.854 2.017.854h17.212c.777 0 1.449-.285 2.017-.854.568-.569.851-1.272.851-2.11zm-.179-33.601l1.614-41.239c0-.718-.3-1.287-.897-1.707-.777-.659-1.494-.988-2.151-.988H70.639c-.657 0-1.374.33-2.151.988-.598.42-.897 1.048-.897 1.887l1.524 41.059c0 .599.3 1.093.897 1.482.597.39 1.314.584 2.151.584h16.584c.837 0 1.54-.195 2.107-.584.568-.39.881-.883.941-1.482zM90.54 6.02l68.846 126.5c2.092 3.773 2.032 7.546-.179 11.32a11.276 11.276 0 0 1-4.168 4.133 11.192 11.192 0 0 1-5.693 1.527H11.654c-2.032 0-3.93-.51-5.693-1.527a11.276 11.276 0 0 1-4.168-4.133c-2.211-3.774-2.271-7.547-.18-11.32L70.46 6.02a11.462 11.462 0 0 1 4.213-4.403A11.105 11.105 0 0 1 80.5 0c2.092 0 4.034.54 5.827 1.617A11.462 11.462 0 0 1 90.54 6.02z"/></defs><g fill="none" fill-rule="evenodd"><circle cx="256" cy="253" r="256" fill="#FDA20C"/><path fill="#000" fill-opacity=".3" fill-rule="nonzero" d="M361 98.292C361 90.982 353.978 85 345.396 85H163.604C155.022 85 148 90.981 148 98.292v296.416c0 7.31 7.022 13.292 15.604 13.292h181.792c8.582 0 15.604-5.981 15.604-13.292v-64.636c-5.462 3.323-11.703 6.646-17.945 9.305v33.399c0 1.329-.78 1.994-2.34 1.994h-172.43c-1.56 0-2.34-.665-2.34-1.994V126.87c0-1.329.78-1.993 2.34-1.993h172.43c1.56 0 2.34.664 2.34 1.993v63.113c3.901 0 8.582-.664 12.483-.664H361V98.292zM254.5 380c5.067 0 9.5 4.433 9.5 9.5s-4.433 9.5-9.5 9.5-9.5-4.433-9.5-9.5 4.433-9.5 9.5-9.5zm24.577-266.907h-47.593c-2.341 0-3.902-1.56-3.902-3.9s1.56-3.899 3.902-3.899h47.593c2.34 0 3.901 1.56 3.901 3.9 0 2.339-2.34 3.899-3.901 3.899z"/><path fill="#FFF" fill-rule="nonzero" d="M355.538 321.966c-3.9 0-8.582 0-12.483-.665v45.438c0 1.33-.78 1.996-2.34 1.996h-172.43c-1.56 0-2.34-.665-2.34-1.996V120.58c0-1.33.78-1.996 2.34-1.996h172.43c1.56 0 2.34.665 2.34 1.996v63.81c3.901 0 8.582-.666 12.483-.666H361V91.306C361 83.988 353.978 78 345.396 78H163.604C155.022 78 148 83.988 148 91.306v297.388c0 7.318 7.022 13.306 15.604 13.306h181.792c8.582 0 15.604-5.988 15.604-13.306v-66.728h-5.462zM230.703 98.871h47.594c2.34 0 3.9 1.56 3.9 3.901 0 2.34-1.56 3.902-3.12 3.902h-47.593c-2.341 0-3.902-1.561-3.902-3.902 0-2.34.78-3.901 3.121-3.901zM254.5 394c-5.067 0-9.5-4.433-9.5-9.5s4.433-9.5 9.5-9.5 9.5 4.433 9.5 9.5c0 5.7-4.433 9.5-9.5 9.5z"/><g opacity=".437" transform="translate(217 160)"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g fill="#2F3136" mask="url(#b)"><path d="M0 0H161V161H0z"/></g></g><g transform="translate(217 149)"><mask id="d" fill="#fff"><use xlink:href="#c"/></mask><g fill="#FFF" mask="url(#d)"><path d="M0 0H161V161H0z"/></g></g></g></svg>
```

## File: static\src\components\phone_field\phone_field.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { patch } from "@web/core/utils/patch";
import { PhoneField, phoneField, formPhoneField } from "@web/views/fields/phone/phone_field";
import { SendSMSButton } from '@sms/components/sms_button/sms_button';

patch(PhoneField, {
    components: {
        ...PhoneField.components,
        SendSMSButton
    },
    defaultProps: {
        ...PhoneField.defaultProps,
        enableButton: true,
    },
    props: {
        ...PhoneField.props,
        enableButton: { type: Boolean, optional: true },
    },
});

const patchDescr = () => ({
    extractProps({ options }) {
        const props = super.extractProps(...arguments);
        props.enableButton = options.enable_sms;
        return props;
    },
    supportedOptions: [{
        label: _t("Enable SMS"),
        name: "enable_sms",
        type: "boolean",
        default: true,
    }],
});

patch(phoneField, patchDescr());
patch(formPhoneField, patchDescr());

```

## File: static\src\components\phone_field\phone_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-inherit="web.PhoneField" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('o_phone_content')]//a" position="after">
            <t t-if="props.enableButton and props.record.data[props.name].length > 0">
                <SendSMSButton t-props="props" />
            </t>
        </xpath>
    </t>

    <t t-inherit="web.FormPhoneField" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('o_phone_content')]" position="inside">
            <t t-if="props.enableButton and props.record.data[props.name].length > 0">
                <SendSMSButton t-props="props" />
            </t>
        </xpath>
    </t>

</templates>

```

## File: static\src\components\sms_button\sms_button.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { user } from "@web/core/user";
import { useService } from "@web/core/utils/hooks";
import { Component, status } from "@odoo/owl";

export class SendSMSButton extends Component {
    static template = "sms.SendSMSButton";
    static props = ["*"];
    setup() {
        this.action = useService("action");
        this.title = _t("Send SMS");
    }
    get phoneHref() {
        return "sms:" + this.props.record.data[this.props.name].replace(/\s+/g, "");
    }
    async onClick() {
        await this.props.record.save();
        this.action.doAction(
            {
                type: "ir.actions.act_window",
                target: "new",
                name: this.title,
                res_model: "sms.composer",
                views: [[false, "form"]],
                context: {
                    ...user.context,
                    default_res_model: this.props.record.resModel,
                    default_res_id: this.props.record.resId,
                    default_number_field_name: this.props.name,
                    default_composition_mode: "comment",
                },
            },
            {
                onClose: () => {
                    if (status(this) === "destroyed") {
                        return;
                    }
                    this.props.record.load();
                },
            }
        );
    }
}

```

## File: static\src\components\sms_button\sms_button.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="sms.SendSMSButton">
        <a
            t-att-title="title"
            t-att-href="phoneHref"
            t-on-click.prevent.stop="onClick"
            class="ms-3 d-inline-flex align-items-center o_field_phone_sms"
        ><i class="fa fa-mobile"></i><small class="fw-bold ms-1">SMS</small></a>
    </t>

</templates>

```

## File: static\src\components\sms_widget\fields_sms_widget.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import {
    EmojisTextField,
    emojisTextField,
} from "@mail/views/web/fields/emojis_text_field/emojis_text_field";
import { useService } from "@web/core/utils/hooks";
import { registry } from "@web/core/registry";

/**
 * SmsWidget is a widget to display a textarea (the body) and a text representing
 * the number of SMS and the number of characters. This text is computed every
 * time the user changes the body.
 */
export class SmsWidget extends EmojisTextField {
    static template = "sms.SmsWidget";
    setup() {
        super.setup();
        this._emojiAdded = () => this.props.record.update({ [this.props.name]: this.targetEditElement.el.value });
        this.notification = useService('notification');
    }

    get encoding() {
        return this._extractEncoding(this.props.record.data[this.props.name] || '');
    }
    get nbrChar() {
        const content = this._getValueForSmsCounts(this.props.record.data[this.props.name] || "");
        return content.length + (content.match(/\n/g) || []).length;
    }
    get nbrCharExplanation() {
        return "";
    }
    get nbrSMS() {
        return this._countSMS(this.nbrChar, this.encoding);
    }

    //--------------------------------------------------------------------------
    // Private: SMS
    //--------------------------------------------------------------------------

    /**
     * Count the number of SMS of the content
     * @private
     * @returns {integer} Number of SMS
     */
    _countSMS(nbrChar, encoding) {
        if (nbrChar === 0) {
            return 0;
        }
        if (encoding === 'UNICODE') {
            if (nbrChar <= 70) {
                return 1;
            }
            return Math.ceil(nbrChar / 67);
        }
        if (nbrChar <= 160) {
            return 1;
        }
        return Math.ceil(nbrChar / 153);
    }

    /**
     * Extract the encoding depending on the characters in the content
     * @private
     * @param {String} content Content of the SMS
     * @returns {String} Encoding of the content (GSM7 or UNICODE)
     */
    _extractEncoding(content) {
        if (String(content).match(RegExp("^[@£$¥èéùìòÇ\\nØø\\rÅåΔ_ΦΓΛΩΠΨΣΘΞÆæßÉ !\\\"#¤%&'()*+,-./0123456789:;<=>?¡ABCDEFGHIJKLMNOPQRSTUVWXYZÄÖÑÜ§¿abcdefghijklmnopqrstuvwxyzäöñüà]*$"))) {
            return 'GSM7';
        }
        return 'UNICODE';
    }

    /**
     * Implement if more characters are going to be sent then those appearing in
     * value, if that value is processed before being sent.
     * E.g., links are converted to trackers in mass_mailing_sms.
     *
     * Note: goes with an explanation in nbrCharExplanation
     *
     * @param {String} value content to be parsed for counting extra characters
     * @return string length-corrected value placeholder for the post-processed
     * state
     */
    _getValueForSmsCounts(value) {
        return value;
    }

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @override
     * @private
     */
    async onBlur() {
        await super.onBlur();
        var content = this.props.record.data[this.props.name] || '';
        if( !content.trim().length && content.length > 0) {
            this.notification.add(
                _t("Your SMS Text Message must include at least one non-whitespace character"),
                { type: 'danger' },
            )
            await this.props.record.update({ [this.props.name]: content.trim() });
        }
    }

    /**
     * @override
     * @private
     */
    async onInput(ev) {
        super.onInput(...arguments);
        await this.props.record.update({ [this.props.name]: this.targetEditElement.el.value });
    }
}

export const smsWidget = {
    ...emojisTextField,
    component: SmsWidget,
    additionalClasses: [
        ...(emojisTextField.additionalClasses || []),
        "o_field_text",
        "o_field_text_emojis",
    ],
};

registry.category("fields").add("sms_widget", smsWidget);

```

## File: static\src\components\sms_widget\fields_sms_widget.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<templates xml:space="preserve">
    <t t-name="sms.SmsWidget" t-inherit="mail.EmojisTextField" t-inherit-mode="primary">
        <xpath expr="/*[last()]/*[last()]" position="after">
            <div class="o_sms_container">
                <span class="text-muted o_sms_count">
                    <t t-out="nbrChar"/> characters<t t-out="nbrCharExplanation"/>, fits in <t t-out="nbrSMS"/> SMS (<t t-out="encoding"/>)
                    <a href="https://iap-services.odoo.com/iap/sms/pricing" target="_blank"
                        title="SMS Pricing" aria-label="SMS Pricing" class="fa fa-lg fa-info-circle"/>
                </span>
            </div>
        </xpath>
    </t>
</templates>

```

## File: static\src\core\failure_model_patch.js

```javascript
/** @odoo-module */

import { Failure } from "@mail/core/common/failure_model";
import { _t } from "@web/core/l10n/translation";
import { patch } from "@web/core/utils/patch";

patch(Failure.prototype, {
    get iconSrc() {
        if (this.type === "sms") {
            return "/sms/static/img/sms_failure.svg";
        }
        return super.iconSrc;
    },
    get body() {
        if (this.type === "sms") {
            if (this.notifications.length === 1 && this.lastMessage?.thread) {
                return _t("An error occurred when sending an SMS on “%(record_name)s”", {
                    record_name: this.lastMessage.thread.name,
                });
            }
            return _t("An error occurred when sending an SMS");
        }
        return super.body;
    },
});

```

## File: static\src\core\notification_model_patch.js

```javascript
/** @odoo-module */

import { Notification } from "@mail/core/common/notification_model";
import { _t } from "@web/core/l10n/translation";
import { patch } from "@web/core/utils/patch";

patch(Notification.prototype, {
    get icon() {
        if (this.notification_type === "sms") {
            return "fa fa-mobile";
        }
        return super.icon;
    },
    get label() {
        if (this.notification_type === "sms") {
            return _t("SMS");
        }
        return super.label;
    },
});

```

## File: static\src\messaging_menu\messaging_menu_patch.js

```javascript
/** @odoo-module */

import { MessagingMenu } from "@mail/core/public_web/messaging_menu";
import { _t } from "@web/core/l10n/translation";
import { patch } from "@web/core/utils/patch";

patch(MessagingMenu.prototype, {
    openFailureView(failure) {
        if (failure.type === "email") {
            return super.openFailureView(failure);
        }
        this.env.services.action.doAction({
            name: _t("SMS Failures"),
            type: "ir.actions.act_window",
            view_mode: "kanban,list,form",
            views: [
                [false, "kanban"],
                [false, "list"],
                [false, "form"],
            ],
            target: "current",
            res_model: failure.resModel,
            domain: [["message_has_sms_error", "=", true]],
            context: { create: false },
        });
        this.dropdown.close();
    },
    getFailureNotificationName(failure) {
        if (failure.type === "sms") {
            return _t("SMS Failure: %(modelName)s", { modelName: failure.modelName });
        }
        return super.getFailureNotificationName(...arguments);
    },
});

```

## File: static\src\thread\message_patch.js

```javascript
/** @odoo-module */

import { Message } from "@mail/core/common/message";

import { patch } from "@web/core/utils/patch";

patch(Message.prototype, {
    onClickFailure() {
        if (this.message.message_type === "sms") {
            this.env.services.action.doAction("sms.sms_resend_action", {
                additionalContext: {
                    default_mail_message_id: this.message.id,
                },
            });
        } else {
            super.onClickFailure(...arguments);
        }
    },
});

```

## File: tools\sms_api.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import exceptions
from odoo.addons.iap.tools import iap_tools
from odoo.tools.translate import _, LazyTranslate

_lt = LazyTranslate(__name__)

ERROR_MESSAGES = {
    # Errors that could occur while updating sender name
    'format_error': _lt("Your sender name must be between 3 and 11 characters long and only contain alphanumeric characters."),
    'unregistered_account': _lt("Your sms account has not been activated yet."),
    'existing_sender': _lt("This account already has an existing sender name and it cannot be changed."),

    # Errors that could occur while sending the verification code
    'invalid_phone_number': _lt("Invalid phone number. Please make sure to follow the international format, i.e. a plus sign (+), then country code, city code, and local phone number. For example: +1 555-555-555"),
    'verification_sms_delivery': _lt(
        "We were not able to reach you via your phone number. "
        "If you have requested multiple codes recently, please retry later."
    ),
    'closed_feature': _lt("The SMS Service is currently unavailable for new users and new accounts registrations are suspended."),
    'banned_account': _lt("This phone number/account has been banned from our service."),

    # Errors that could occur while verifying the code
    'invalid_code': _lt("The verification code is incorrect."),
    'no_sms_account': _lt("We were not able to find your account in our database."),
    'too_many_attempts': _lt("You tried too many times. Please retry later."),

    # Default error
    'unknown_error': _lt("An unknown error occurred. Please contact Odoo support if this error persists."),
}


class SmsApi:
    DEFAULT_ENDPOINT = 'https://sms.api.odoo.com'

    def __init__(self, env, account=None):
        self.env = env
        self.account = account or self.env['iap.account'].get('sms')

    def _contact_iap(self, local_endpoint, params, timeout=15):
        if not self.env.registry.ready:  # Don't reach IAP servers during module installation
            raise exceptions.AccessError("Unavailable during module installation.")

        params['account_token'] = self.account.account_token
        endpoint = self.env['ir.config_parameter'].sudo().get_param('sms.endpoint', self.DEFAULT_ENDPOINT)
        return iap_tools.iap_jsonrpc(endpoint + local_endpoint, params=params, timeout=timeout)

    def _send_sms_batch(self, messages, delivery_reports_url=False):
        """ Send SMS using IAP in batch mode

        :param list messages: list of SMS (grouped by content) to send:
          formatted as ```[
              {
                  'content' : str,
                  'numbers' : [
                      { 'uuid' : str, 'number' : str },
                      { 'uuid' : str, 'number' : str },
                      ...
                  ]
              }, ...
          ]```
        :param str delivery_reports_url: url to route receiving delivery reports
        :return: response from the endpoint called, which is a list of results
          formatted as ```[
              {
                  uuid: UUID of the request,
                  state: ONE of: {
                      'success', 'processing', 'server_error', 'unregistered', 'insufficient_credit',
                      'wrong_number_format', 'duplicate_message', 'country_not_supported', 'registration_needed',
                  },
                  credit: Optional: Credits spent to send SMS (provided if the actual price is known)
              }, ...
          ]```
        """
        return self._contact_iap('/api/sms/3/send', {
            'messages': messages,
            'webhook_url': delivery_reports_url,
            'dbuuid': self.env['ir.config_parameter'].sudo().get_param('database.uuid')
        })

    def _get_sms_api_error_messages(self):
        """Return a mapping of `_send_sms_batch` errors to an error message.

        We prefer a dict instead of a message-per-error-state based method so that we only call
        config parameters getters once and avoid extra RPC calls."""
        buy_credits_url = self.env['iap.account'].sudo().get_credits_url(service_name='sms')
        buy_credits = '<a href="{}" target="_blank">{}</a>'.format(buy_credits_url, _("Buy credits."))

        sms_endpoint = self.env['ir.config_parameter'].sudo().get_param('sms.endpoint', self.DEFAULT_ENDPOINT)
        sms_account_token = self.env['iap.account'].sudo().get('sms').account_token
        register_now = f'<a href="{sms_endpoint}/1/account?account_token={sms_account_token}" target="_blank">%s</a>' % (
            _('Register now.')
        )

        return {
            'unregistered': _("You don't have an eligible IAP account."),
            'insufficient_credit': ' '.join([_("You don't have enough credits on your IAP account."), buy_credits]),
            'wrong_number_format': _("The number you're trying to reach is not correctly formatted."),
            'duplicate_message': _("This SMS has been removed as the number was already used."),
            'country_not_supported': _("The destination country is not supported."),
            'incompatible_content': _("The content of the message violates rules applied by our providers."),
            'registration_needed': ' '.join([_("Country-specific registration required."), register_now]),
        }

    def _send_verification_sms(self, phone_number):
        return self._contact_iap('/api/sms/1/account/create', {
            'phone_number': phone_number,
        })

    def _verify_account(self, verification_code):
        return self._contact_iap('/api/sms/2/account/verify', {
            'code': verification_code,
        })

    def _set_sender_name(self, sender_name):
        return self._contact_iap('/api/sms/1/account/update_sender', {
            'sender_name': sender_name,
        })

```

## File: tools\sms_tools.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

import markupsafe

from odoo.tools import html_escape
from odoo.tools.mail import create_link, TEXT_URL_REGEX


def sms_content_to_rendered_html(text):
    """Transforms plaintext into html making urls clickable and preserving newlines"""
    urls = re.findall(TEXT_URL_REGEX, text)
    escaped_text = html_escape(text)
    for url in urls:
        escaped_text = escaped_text.replace(url, markupsafe.Markup(create_link(url, url)))
    return markupsafe.Markup(re.sub(r'\r?\n|\r', '<br/>', escaped_text))

```

## File: tools\__init__.py

```python
from . import sms_api
from . import sms_tools

```

## File: views\iap_account_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id="iap_account_view_form" model="ir.ui.view">
            <field name="name">iap.account.view.form</field>
            <field name="model">iap.account</field>
            <field name="inherit_id" ref="iap.iap_account_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//group[@name='account']" position="before">
                    <div role="status" class="alert alert-danger mb-0" invisible="service_name != 'sms' or (state != 'unregistered' and state)">
                        <i class="fa fa-warning"/> You cannot send SMS while your account is not registered.
                        <button type="object"
                            name="action_open_registration_wizard" class="btn-link p-0">
                            <i class="oi oi-arrow-right"/>
                            Register
                        </button>
                    </div>
                    <div role="status" class="alert alert-danger mb-0" invisible="service_name != 'sms' or (state != 'banned')">
                        <i class="fa fa-warning"/> An error occurred with your account. Please contact the support for more information...
                    </div>
                </xpath>
                <xpath expr="//button[@name='action_buy_credits']" position="attributes">
                    <attribute name="invisible">service_name == 'sms' and not state</attribute>
                </xpath>
                <xpath expr="//group[@name='external']/*[1]" position="before">
                    <label for="sender_name" class="oe_inline" invisible="service_name != 'sms' or not (state == 'registered' and state)"/>
                    <div invisible="service_name != 'sms'  or not (state == 'registered' and state)">
                        <field name="sender_name" class="oe_inline" invisible="not sender_name"/>
                        <button type="object" invisible="sender_name"
                            name="action_open_sender_name_wizard" class="btn-link p-0">
                            <i class="oi oi-arrow-right"/>
                            Set Sender Name
                        </button>
                    </div>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\ir_actions_server_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>

    <record id="ir_actions_server_view_form"  model="ir.ui.view">
        <field name="name">ir.actions.server.view.form.inherit.sms</field>
        <field name="model">ir.actions.server</field>
        <field name="inherit_id" ref="base.view_server_action_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='link_field_id']" position="after">
                <field name="sms_template_id"
                    placeholder="Choose a template..."
                    context="{'default_model': model_name}"
                    invisible="state != 'sms'"
                    required="state == 'sms'"/>
                <label for="sms_method" invisible="state != 'sms'"/>
                    <div class="d-flex flex-column" invisible="state != 'sms'">
                        <field name="sms_method" required="state == 'sms'"/>
                        <div class="text-muted">
                            <span invisible="sms_method != 'sms'">
                                The message will be sent as an SMS to the recipients of the template
                                and will not appear in the messaging history.
                            </span>
                            <span invisible="sms_method != 'note'">
                                The SMS will not be sent, it will only be posted as an
                                internal note in the messaging history.
                            </span>
                            <span invisible="sms_method != 'comment'">
                                The SMS will be sent as an SMS to the recipients of the
                                template and it will also be posted as an internal note
                                in the messaging history.
                            </span>
                        </div>
                    </div>
            </xpath>
        </field>
    </record>

</data></odoo>

```

## File: views\mail_notification_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>
    <record id="mail_notification_view_tree" model="ir.ui.view">
        <field name="name">mail.notification.view.list</field>
        <field name="model">mail.notification</field>
        <field name="inherit_id" ref="mail.mail_notification_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='res_partner_id']" position="after">
                <field name="sms_number"/>
            </xpath>
        </field>
    </record>

    <record id="mail_notification_view_form" model="ir.ui.view">
        <field name="name">mail.notification.view.form</field>
        <field name="model">mail.notification</field>
        <field name="inherit_id" ref="mail.mail_notification_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='res_partner_id']" position="after">
                <field name="sms_number"/>
            </xpath>
            <xpath expr="//field[@name='mail_mail_id']" position="after">
                <field name="sms_id"/>
            </xpath>
        </field>
    </record>
</data></odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.sms</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <setting id="sms" position="inside">
                <widget name="iap_buy_more_credits" service_name="sms" hide_service="1"/>
            </setting>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Add action entry in the Action Menu for Partners -->
    <record id="res_partner_view_form" model="ir.ui.view">
        <field name="name">res.partner.view.form.inherit.sms</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="priority">10</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='mobile']" position="after">
                <field name="phone_sanitized" groups="base.group_no_one" invisible="1"/>
            </xpath>
            <xpath expr="//field[@name='phone']" position="replace">
                <field name="phone_blacklisted" invisible="1"/>
                <field name="mobile_blacklisted" invisible="1"/>
                <label for="phone" class="oe_inline"/>
                <div class="o_row o_row_readonly">
                    <button name="phone_action_blacklist_remove" class="fa fa-ban text-danger"
                        title="This phone number is blacklisted for SMS Marketing. Click to unblacklist."
                        type="object" context="{'default_phone': phone}" groups="base.group_user"
                        invisible="not phone_blacklisted"/>
                    <field name="phone" widget="phone"/>
                </div>
            </xpath>
            <xpath expr="//field[@name='mobile']" position="replace">
                <field name="phone_blacklisted" invisible="1"/>
                <field name="mobile_blacklisted" invisible="1"/>
                <label for="mobile" class="oe_inline"/>
                <div class="o_row o_row_readonly">
                    <button name="phone_action_blacklist_remove" class="fa fa-ban text-danger"
                        title="This phone number is blacklisted for SMS Marketing. Click to unblacklist."
                        type="object" context="{'default_phone': mobile}" groups="base.group_user"
                        invisible="not mobile_blacklisted"/>
                    <field name="mobile" widget="phone"/>
                </div>
            </xpath>
        </field>
    </record>

    <!-- Add action entry in the Action Menu for Partners -->
    <record id="res_partner_act_window_sms_composer_multi" model="ir.actions.act_window">
        <field name="name">Send SMS</field>
        <field name="res_model">sms.composer</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="context">{
            'default_composition_mode': 'mass',
            'default_mass_keep_log': True,
            'default_res_ids': active_ids
        }</field>
        <field name="binding_model_id" ref="base.model_res_partner"/>
        <field name="binding_view_types">list</field>
    </record>
    <record id="res_partner_act_window_sms_composer_single" model="ir.actions.act_window">
        <field name="name">Send SMS</field>
        <field name="res_model">sms.composer</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="context">{
            'default_composition_mode': 'comment',
            'default_res_id': active_id,
        }</field>
        <field name="binding_model_id" ref="base.model_res_partner"/>
        <field name="binding_view_types">form</field>
    </record>

    <record id="res_partner_view_search" model="ir.ui.view">
        <field name="name">res.partner.view.search.inherit.sms</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_res_partner_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='phone']" position="replace">
                <field name="phone_mobile_search"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\sms_sms_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo><data>
    <record id="sms_tsms_view_form" model="ir.ui.view">
        <field name="name">sms.sms.view.form</field>
        <field name="model">sms.sms</field>
        <field name="arch" type="xml">
            <form string="SMS">
                <header>
                    <field name="to_delete" invisible="1"/>
                    <button name="send" string="Send Now" type="object" invisible="state != 'outgoing' or to_delete" class="oe_highlight"/>
                    <button name="action_set_outgoing" string="Retry" type="object" invisible="state not in ('error', 'canceled')"/>
                    <button name="action_set_canceled" string="Cancel" type="object" invisible="state not in ('error', 'outgoing')"/>
                    <field name="state" widget="statusbar" statusbar_visible="outgoing,sent,error,canceled"/>
                </header>
                <sheet>
                    <group>
                        <group>
                            <field name="partner_id" string="Contact"/>
                            <field name="mail_message_id" readonly="1" invisible="not mail_message_id"/>
                        </group>
                        <group>
                            <field name="number" required="1"/>
                            <field name="failure_type" readonly="1" invisible="not failure_type"/>
                        </group>
                    </group>
                    <group>
                        <field name="body" widget="sms_widget" string="Message" required="1"
                            readonly="state in ('process', 'sent')"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="sms_sms_view_tree" model="ir.ui.view">
        <field name="name">sms.sms.view.list</field>
        <field name="model">sms.sms</field>
        <field name="arch" type="xml">
            <list string="SMS Templates">
                <field name="number"/>
                <field name="partner_id"/>
                <field name="failure_type"/>
                <field name="state" widget="badge" decoration-info="state == 'outgoing'" decoration-muted="state == 'canceled'" decoration-success="state == 'sent'" decoration-danger="state == 'error'"/>
                <button name="send" string="Send Now" type="object" icon="fa-paper-plane" invisible="state != 'outgoing'"/>
                <button name="action_set_outgoing" string="Retry" type="object" icon="fa-repeat" invisible="state not in ('error', 'canceled')"/>
                <button name="action_set_canceled" string="Cancel" type="object" icon="fa-times-circle" invisible="state not in ('error', 'outgoing')"/>
            </list>
        </field>
    </record>

    <record id="sms_sms_view_search" model="ir.ui.view">
        <field name="name">sms.sms.view.search</field>
        <field name="model">sms.sms</field>
        <field name="arch" type="xml">
            <search string="Search SMS Templates">
                <field name="number"/>
                <field name="partner_id"/>
            </search>
        </field>
    </record>

    <record id="sms_sms_action" model="ir.actions.act_window">
        <field name="name">SMS</field>
        <field name="res_model">sms.sms</field>
        <field name="view_mode">list,form</field>
        <field name="domain">[('to_delete', '!=', True)]</field>
    </record>

    <menuitem id="sms_sms_menu"
        parent="phone_validation.phone_menu_main"
        action="sms_sms_action"
        sequence="1"/>

    <record id="ir_actions_server_sms_sms_resend" model="ir.actions.server">
        <field name="name">Resend</field>
        <field name="model_id" ref="sms.model_sms_sms"/>
        <field name="binding_model_id" ref="sms.model_sms_sms"/>
        <field name="binding_view_types">list</field>
        <field name="state">code</field>
        <field name="code">action = records.resend_failed()</field>
    </record>
</data>
</odoo>

```

## File: views\sms_template_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo><data>
    <record id="sms_template_view_form" model="ir.ui.view">
        <field name="name">sms.template.view.form</field>
        <field name="model">sms.template</field>
        <field name="arch" type="xml">
            <form string="SMS Templates">
                <header>
                    <field name="template_fs" invisible="1"/>
                    <button string="Reset Template"
                            name="%(sms_template_reset_action)d" type="action"
                            groups="base.group_system"
                            invisible="not template_fs"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <field name="sidebar_action_id" invisible="1"/>
                        <button name="action_create_sidebar_action" type="object"
                                groups="base.group_no_one"
                                class="oe_stat_button"
                                invisible="sidebar_action_id" icon="fa-plus"
                                help="Add a contextual action on the related model to open a sms composer with this template">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_text">Add</span>
                                <span class="o_stat_text">Context Action</span>
                            </div>
                        </button>
                        <button name="action_unlink_sidebar_action" type="object"
                                groups="base.group_no_one"
                                class="oe_stat_button" icon="fa-minus"
                                invisible="not sidebar_action_id"
                                help="Remove the contextual action of the related model" widget="statinfo">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_text">Remove</span>
                                <span class="o_stat_text">Context Action</span>
                            </div>
                        </button>
                        <button class="oe_stat_button" name="%(sms_template_preview_action)d" icon="fa-search-plus" type="action" target="new">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_text">Preview</span>
                            </div>
                        </button>
                    </div>
                    <div class="oe_title">
                        <label for="name" string="SMS Template"/>
                        <h1><field name="name" placeholder="e.g. Calendar Reminder" required="1"/></h1>
                        <group>
                            <field name="model_id" placeholder="e.g. Contact" required="1" options="{'no_create': True}"/>
                            <field name="model" invisible="1"/>
                            <field name="lang" groups="base.group_no_one" placeholder="e.g. en_US or {{ object.partner_id.lang }}"/>
                        </group>
                    </div>
                    <notebook>
                        <page string="Content" name="content">
                            <group>
                                <field name="body" widget="sms_widget" nolabel="1"/>
                            </group>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="sms_template_view_tree" model="ir.ui.view">
        <field name="name">sms.template.view.list</field>
        <field name="model">sms.template</field>
        <field name="arch" type="xml">
            <list string="SMS Templates">
                <field name="name"/>
                <field name="model_id"/>
            </list>
        </field>
    </record>

    <record id="sms_template_view_search" model="ir.ui.view">
        <field name="name">sms.template.view.search</field>
        <field name="model">sms.template</field>
        <field name="arch" type="xml">
            <search string="Search SMS Templates">
                <field name="name"/>
                <field name="model_id"/>
            </search>
        </field>
    </record>

    <record id="sms_template_action" model="ir.actions.act_window">
        <field name="name">Templates</field>
        <field name="res_model">sms.template</field>
        <field name="view_mode">list,form</field>
    </record>

    <menuitem id="sms_template_menu"
        name="SMS Templates"
        parent="phone_validation.phone_menu_main"
        sequence="2"
        action="sms_template_action"/>

</data></odoo>

```

## File: wizard\sms_account_code.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _
from odoo.addons.sms.tools.sms_api import ERROR_MESSAGES, SmsApi
from odoo.exceptions import ValidationError


class SMSAccountCode(models.TransientModel):
    _name = 'sms.account.code'
    _description = 'SMS Account Verification Code Wizard'

    account_id = fields.Many2one('iap.account', required=True)
    verification_code = fields.Char(required=True)

    def action_register(self):
        status = SmsApi(self.env, self.account_id)._verify_account(self.verification_code)['state']
        if status != 'success':
            raise ValidationError(ERROR_MESSAGES.get(status, ERROR_MESSAGES['unknown_error']))

        self.account_id.state = "registered"
        self.env['iap.account']._send_success_notification(
            message=_("Your SMS account has been successfully registered."),
        )

        sender_name_wizard = self.env['sms.account.sender'].create({
            'account_id': self.account_id.id,
        })

        return {
            'type': 'ir.actions.act_window',
            'target': 'new',
            'name': _('Choose your sender name'),
            'view_mode': 'form',
            'res_model': 'sms.account.sender',
            'res_id': sender_name_wizard.id,
        }

```

## File: wizard\sms_account_code_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sms_account_code_view_form" model="ir.ui.view">
        <field name="name">sms.account.code.view.form</field>
        <field name="model">sms.account.code</field>
        <field name="arch" type="xml">
            <form string="Register your SMS account">
                <sheet>
                    <group>
                        <field name="verification_code" required="1"/>
                    </group>
                    <footer>
                        <button string="Register" name="action_register" type="object" class="btn btn-primary"/>
                        <button string="Cancel" class="btn-secondary" special="cancel"/>
                    </footer>
                </sheet>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\sms_account_phone.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _
from odoo.addons.sms.tools.sms_api import ERROR_MESSAGES, SmsApi
from odoo.exceptions import ValidationError


class SMSAccountPhone(models.TransientModel):
    _name = 'sms.account.phone'
    _description = 'SMS Account Registration Phone Number Wizard'

    account_id = fields.Many2one('iap.account', required=True)
    phone_number = fields.Char(required=True)

    def action_send_verification_code(self):
        status = SmsApi(self.env, self.account_id)._send_verification_sms(self.phone_number)['state']
        if status != 'success':
            raise ValidationError(ERROR_MESSAGES.get(status, ERROR_MESSAGES['unknown_error']))

        return {
            'type': 'ir.actions.act_window',
            'target': 'new',
            'name': _('Register Account'),
            'view_mode': 'form',
            'res_model': 'sms.account.code',
            'context': {'default_account_id': self.account_id.id},
        }

```

## File: wizard\sms_account_phone_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sms_account_phone_view_form" model="ir.ui.view">
        <field name="name">sms.account.phone.view.form</field>
        <field name="model">sms.account.phone</field>
        <field name="arch" type="xml">
            <form string="Register your SMS account">
                <sheet>
                    <h5>Enter a phone number to get an SMS with a verification code.</h5>
                    <group>
                        <field name="phone_number" placeholder="+1 555-555-555"/>
                    </group>
                    <footer>
                        <button string="Send verification code" name="action_send_verification_code" type="object" class="btn btn-primary"/>
                        <button string="Cancel" class="btn-secondary" special="cancel"/>
                    </footer>
                </sheet>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\sms_account_sender.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from odoo import api, fields, models
from odoo.addons.sms.tools.sms_api import ERROR_MESSAGES, SmsApi
from odoo.exceptions import ValidationError


class SMSAccountSender(models.TransientModel):
    _name = 'sms.account.sender'
    _description = 'SMS Account Sender Name Wizard'

    account_id = fields.Many2one('iap.account', required=True)
    sender_name = fields.Char()

    @api.constrains("sender_name")
    def _check_sender_name(self):
        for record in self:
            if not re.match(r"[a-zA-Z0-9\- ]{3,11}", record.sender_name):
                raise ValidationError("Your sender name must be between 3 and 11 characters long and only contain alphanumeric characters.")

    def action_set_sender_name(self):
        status = SmsApi(self.env, self.account_id)._set_sender_name(self.sender_name)['state']
        if status != 'success':
            raise ValidationError(ERROR_MESSAGES.get(status, ERROR_MESSAGES['unknown_error']))

```

## File: wizard\sms_account_sender_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sms_account_sender_view_form" model="ir.ui.view">
        <field name="name">sms.account.sender.view.form</field>
        <field name="model">sms.account.sender</field>
        <field name="arch" type="xml">
            <form string="Choose your sender name">
                <sheet>
                    <p>
                        Your sender name must be between 3 and 11 characters long and only contain alphanumeric characters.
                        It must fit your company name, and you aren't allowed to modify it once you registered one, choose it carefully.
                    </p>
                    <p>
                        Note that this is not required, if you don't set a sender name, your SMS will be sent from a short code.
                    </p>
                    <group>
                        <field name="sender_name" required="1"/>
                    </group>
                    <footer>
                        <button string="Set sender name" name="action_set_sender_name" type="object" class="btn btn-primary"/>
                        <button string="Skip for now" class="btn-secondary" special="cancel"/>
                    </footer>
                </sheet>
            </form>
        </field>
    </record>
</odoo>

```

## File: wizard\sms_composer.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval
from uuid import uuid4

from odoo import api, fields, models, _
from odoo.addons.sms.tools.sms_tools import sms_content_to_rendered_html
from odoo.exceptions import UserError


class SendSMS(models.TransientModel):
    _name = 'sms.composer'
    _description = 'Send SMS Wizard'

    @api.model
    def default_get(self, fields):
        result = super(SendSMS, self).default_get(fields)

        result['res_model'] = result.get('res_model') or self.env.context.get('active_model')

        if not result.get('res_ids'):
            if not result.get('res_id') and self.env.context.get('active_ids') and len(self.env.context.get('active_ids')) > 1:
                result['res_ids'] = repr(self.env.context.get('active_ids'))
        if not result.get('res_id'):
            if not result.get('res_ids') and self.env.context.get('active_id'):
                result['res_id'] = self.env.context.get('active_id')

        return result

    # documents
    composition_mode = fields.Selection([
        ('numbers', 'Send to numbers'),
        ('comment', 'Post on a document'),
        ('mass', 'Send SMS in batch')], string='Composition Mode',
        compute='_compute_composition_mode', precompute=True, readonly=False, required=True, store=True)
    res_model = fields.Char('Document Model Name')
    res_model_description = fields.Char('Document Model Description', compute='_compute_res_model_description')
    res_id = fields.Integer('Document ID')
    res_ids = fields.Char('Document IDs')
    res_ids_count = fields.Integer(
        'Visible records count', compute='_compute_res_ids_count', compute_sudo=False,
        help='Number of recipients that will receive the SMS if sent in mass mode, without applying the Active Domain value')
    comment_single_recipient = fields.Boolean(
        'Single Mode', compute='_compute_comment_single_recipient', compute_sudo=False,
        help='Indicates if the SMS composer targets a single specific recipient')
    # options for comment and mass mode
    mass_keep_log = fields.Boolean('Keep a note on document', default=True)
    mass_force_send = fields.Boolean('Send directly', default=False)
    mass_use_blacklist = fields.Boolean('Use blacklist', default=True)
    # recipients
    recipient_valid_count = fields.Integer('# Valid recipients', compute='_compute_recipients', compute_sudo=False)
    recipient_invalid_count = fields.Integer('# Invalid recipients', compute='_compute_recipients', compute_sudo=False)
    recipient_single_description = fields.Text('Recipients (Partners)', compute='_compute_recipient_single_non_stored', compute_sudo=False)
    recipient_single_number = fields.Char('Stored Recipient Number', compute='_compute_recipient_single_non_stored', compute_sudo=False)
    recipient_single_number_itf = fields.Char(
        'Recipient Number', compute='_compute_recipient_single_stored',
        readonly=False, compute_sudo=False, store=True,
        help='Phone number of the recipient. If changed, it will be recorded on recipient\'s profile.')
    recipient_single_valid = fields.Boolean("Is valid", compute='_compute_recipient_single_valid', compute_sudo=False)
    number_field_name = fields.Char('Number Field')
    numbers = fields.Char('Recipients (Numbers)')
    sanitized_numbers = fields.Char('Sanitized Number', compute='_compute_sanitized_numbers', compute_sudo=False)
    # content
    template_id = fields.Many2one('sms.template', string='Use Template', domain="[('model', '=', res_model)]")
    body = fields.Text(
        'Message', compute='_compute_body',
        precompute=True, readonly=False, store=True, required=True)

    @api.depends('res_ids_count')
    @api.depends_context('sms_composition_mode')
    def _compute_composition_mode(self):
        for composer in self:
            if self.env.context.get('sms_composition_mode') == 'guess' or not composer.composition_mode:
                if composer.res_ids_count > 1:
                    composer.composition_mode = 'mass'
                else:
                    composer.composition_mode = 'comment'

    @api.depends('res_model')
    def _compute_res_model_description(self):
        self.res_model_description = False
        for composer in self.filtered('res_model'):
            composer.res_model_description = self.env['ir.model']._get(composer.res_model).display_name

    @api.depends('res_model', 'res_id', 'res_ids')
    def _compute_res_ids_count(self):
        for composer in self:
            composer.res_ids_count = len(literal_eval(composer.res_ids)) if composer.res_ids else 0

    @api.depends('res_id', 'composition_mode')
    def _compute_comment_single_recipient(self):
        for composer in self:
            composer.comment_single_recipient = bool(composer.res_id and composer.composition_mode == 'comment')

    @api.depends('res_model', 'res_id', 'res_ids', 'composition_mode', 'number_field_name', 'sanitized_numbers')
    def _compute_recipients(self):
        for composer in self:
            composer.recipient_valid_count = 0
            composer.recipient_invalid_count = 0

            if composer.composition_mode not in ('comment', 'mass') or not composer.res_model:
                continue

            records = composer._get_records()
            if records:
                res = records._sms_get_recipients_info(force_field=composer.number_field_name, partner_fallback=not composer.comment_single_recipient)
                composer.recipient_valid_count = len([rid for rid, rvalues in res.items() if rvalues['sanitized']])
                composer.recipient_invalid_count = len([rid for rid, rvalues in res.items() if not rvalues['sanitized']])
            else:
                composer.recipient_invalid_count = 0 if (
                    composer.sanitized_numbers or composer.composition_mode == 'mass'
                ) else 1

    @api.depends('res_model', 'number_field_name')
    def _compute_recipient_single_stored(self):
        for composer in self:
            records = composer._get_records()
            if not records or not composer.comment_single_recipient:
                composer.recipient_single_number_itf = ''
                continue
            records.ensure_one()
            res = records._sms_get_recipients_info(force_field=composer.number_field_name, partner_fallback=True)
            if not composer.recipient_single_number_itf:
                composer.recipient_single_number_itf = res[records.id]['sanitized'] or res[records.id]['number'] or ''
            if not composer.number_field_name:
                composer.number_field_name = res[records.id]['field_store']

    @api.depends('res_model', 'number_field_name')
    def _compute_recipient_single_non_stored(self):
        for composer in self:
            records = composer._get_records()
            if not records or not composer.comment_single_recipient:
                composer.recipient_single_description = False
                composer.recipient_single_number = ''
                continue
            records.ensure_one()
            res = records._sms_get_recipients_info(force_field=composer.number_field_name, partner_fallback=True)
            composer.recipient_single_description = res[records.id]['partner'].name or records._mail_get_partners()[records[0].id].display_name
            composer.recipient_single_number = res[records.id]['sanitized'] or res[records.id]['number'] or ''

    @api.depends('recipient_single_number', 'recipient_single_number_itf')
    def _compute_recipient_single_valid(self):
        for composer in self:
            value = composer.recipient_single_number_itf or composer.recipient_single_number
            if value:
                records = composer._get_records()
                composer.recipient_single_valid = bool(records._phone_format(number=value)) if len(records) == 1 else False
            else:
                composer.recipient_single_valid = False

    @api.depends('numbers', 'res_model', 'res_id')
    def _compute_sanitized_numbers(self):
        for composer in self:
            if composer.numbers:
                record = composer._get_records() if composer.res_model and composer.res_id else self.env.user
                numbers = [number.strip() for number in composer.numbers.split(',')]
                sanitized_numbers = [record._phone_format(number=number) for number in numbers]
                invalid_numbers = [number for sanitized, number in zip(sanitized_numbers, numbers) if not sanitized]
                if invalid_numbers:
                    raise UserError(_('Following numbers are not correctly encoded: %s', repr(invalid_numbers)))
                composer.sanitized_numbers = ','.join(sanitized_numbers)
            else:
                composer.sanitized_numbers = False

    @api.depends('composition_mode', 'res_model', 'res_id', 'template_id')
    def _compute_body(self):
        for record in self:
            if record.template_id and record.composition_mode == 'comment' and record.res_id:
                record.body = record.template_id._render_field('body', [record.res_id], compute_lang=True)[record.res_id]
            elif record.template_id:
                record.body = record.template_id.body

    # ------------------------------------------------------------
    # Actions
    # ------------------------------------------------------------

    def action_send_sms(self):
        if self.composition_mode in ('numbers', 'comment'):
            if self.comment_single_recipient and not self.recipient_single_valid:
                raise UserError(_('Invalid recipient number. Please update it.'))
            elif not self.comment_single_recipient and self.recipient_invalid_count:
                raise UserError(_('%s invalid recipients', self.recipient_invalid_count))
        self._action_send_sms()
        return False

    def action_send_sms_mass_now(self):
        if not self.mass_force_send:
            self.write({'mass_force_send': True})
        return self.action_send_sms()

    def _action_send_sms(self):
        records = self._get_records()
        if self.composition_mode == 'numbers':
            return self._action_send_sms_numbers()
        elif self.composition_mode == 'comment':
            if records is None or not isinstance(records, self.pool['mail.thread']):
                return self._action_send_sms_numbers()
            if self.comment_single_recipient:
                return self._action_send_sms_comment_single(records)
            else:
                return self._action_send_sms_comment(records)
        else:
            return self._action_send_sms_mass(records)

    def _action_send_sms_numbers(self):
        sms_values = [
            {
                'body': self.body,
                'number': number
            } for number in (
                self.sanitized_numbers.split(',') if self.sanitized_numbers else [self.recipient_single_number_itf or self.recipient_single_number or '']
            )
        ]
        self.env['sms.sms'].sudo().create(sms_values).send()
        return True

    def _action_send_sms_comment_single(self, records=None):
        # If we have a recipient_single_original number, it's possible this number has been corrected in the popup
        # if invalid. As a consequence, the test cannot be based on recipient_invalid_count, which count is based
        # on the numbers in the database.
        records = records if records is not None else self._get_records()
        records.ensure_one()
        if not self.number_field_name or self.number_field_name not in records:
            self.numbers = self.recipient_single_number_itf or self.recipient_single_number
        elif self.recipient_single_number_itf and self.recipient_single_number_itf != self.recipient_single_number:
            records.write({self.number_field_name: self.recipient_single_number_itf})
        return self._action_send_sms_comment(records=records)

    def _action_send_sms_comment(self, records=None):
        records = records if records is not None else self._get_records()
        subtype_id = self.env['ir.model.data']._xmlid_to_res_id('mail.mt_note')

        messages = self.env['mail.message']
        all_bodies = self._prepare_body_values(records)

        for record in records:
            messages += record._message_sms(
                all_bodies[record.id],
                subtype_id=subtype_id,
                number_field=self.number_field_name,
                sms_numbers=self.sanitized_numbers.split(',') if self.sanitized_numbers else None)
        return messages

    def _action_send_sms_mass(self, records=None):
        records = records if records is not None else self._get_records()

        sms_record_values = self._prepare_mass_sms_values(records)
        sms_all = self._prepare_mass_sms(records, sms_record_values)
        if sms_all and self.mass_keep_log and records and isinstance(records, self.pool['mail.thread']):
            log_values = self._prepare_mass_log_values(records, sms_record_values)
            records._message_log_batch(**log_values)

        if sms_all and self.mass_force_send:
            sms_all.filtered(lambda sms: sms.state == 'outgoing').send(auto_commit=False, raise_exception=False)
            return self.env['sms.sms'].sudo().search([('id', 'in', sms_all.ids)])
        return sms_all

    # ------------------------------------------------------------
    # Mass mode specific
    # ------------------------------------------------------------

    def _get_blacklist_record_ids(self, records, recipients_info):
        """ Get a list of blacklisted records. Those will be directly canceled
        with the right error code. """
        if self.mass_use_blacklist:
            bl_numbers = self.env['phone.blacklist'].sudo().search([]).mapped('number')
            return [r.id for r in records if recipients_info[r.id]['sanitized'] in bl_numbers]
        return []

    def _get_optout_record_ids(self, records, recipients_info):
        """ Compute opt-outed contacts, not necessarily blacklisted. Void by default
        as no opt-out mechanism exist in SMS, see SMS Marketing. """
        return []

    def _get_done_record_ids(self, records, recipients_info):
        """ Get a list of already-done records. Order of record set is used to
        spot duplicates so pay attention to it if necessary. """
        done_ids, done = [], []
        for record in records:
            sanitized = recipients_info[record.id]['sanitized']
            if sanitized in done:
                done_ids.append(record.id)
            else:
                done.append(sanitized)
        return done_ids

    def _prepare_recipient_values(self, records):
        recipients_info = records._sms_get_recipients_info(force_field=self.number_field_name)
        return recipients_info

    def _prepare_body_values(self, records):
        if self.template_id and self.body == self.template_id.body:
            all_bodies = self.template_id._render_field('body', records.ids, compute_lang=True)
        else:
            all_bodies = self.env['mail.render.mixin']._render_template(self.body, records._name, records.ids)
        return all_bodies

    def _prepare_mass_sms_values(self, records):
        all_bodies = self._prepare_body_values(records)
        all_recipients = self._prepare_recipient_values(records)
        blacklist_ids = self._get_blacklist_record_ids(records, all_recipients)
        optout_ids = self._get_optout_record_ids(records, all_recipients)
        done_ids = self._get_done_record_ids(records, all_recipients)

        result = {}
        for record in records:
            recipients = all_recipients[record.id]
            sanitized = recipients['sanitized']
            if sanitized and record.id in blacklist_ids:
                state = 'canceled'
                failure_type = 'sms_blacklist'
            elif sanitized and record.id in optout_ids:
                state = 'canceled'
                failure_type = 'sms_optout'
            elif sanitized and record.id in done_ids:
                state = 'canceled'
                failure_type = 'sms_duplicate'
            elif not sanitized:
                state = 'canceled'
                failure_type = 'sms_number_format' if recipients['number'] else 'sms_number_missing'
            else:
                state = 'outgoing'
                failure_type = ''

            result[record.id] = {
                'body': all_bodies[record.id],
                'failure_type': failure_type,
                'number': sanitized if sanitized else recipients['number'],
                'partner_id': recipients['partner'].id,
                'state': state,
                'uuid': uuid4().hex,
            }
        return result

    def _prepare_mass_sms(self, records, sms_record_values):
        sms_create_vals = [sms_record_values[record.id] for record in records]
        return self.env['sms.sms'].sudo().create(sms_create_vals)

    def _prepare_log_body_values(self, sms_records_values):
        result = {}
        for record_id, sms_values in sms_records_values.items():
            result[record_id] = sms_content_to_rendered_html(sms_values['body'])
        return result

    def _prepare_mass_log_values(self, records, sms_records_values):
        return {
            'bodies': self._prepare_log_body_values(sms_records_values),
            'message_type': 'sms',
        }

    # ------------------------------------------------------------
    # Tools
    # ------------------------------------------------------------

    def _get_composer_values(self, composition_mode, res_model, res_id, body, template_id):
        result = {}
        if composition_mode == 'comment':
            if not body and template_id and res_id:
                template = self.env['sms.template'].browse(template_id)
                result['body'] = template._render_template(template.body, res_model, [res_id])[res_id]
            elif template_id:
                template = self.env['sms.template'].browse(template_id)
                result['body'] = template.body
        else:
            if not body and template_id:
                template = self.env['sms.template'].browse(template_id)
                result['body'] = template.body
        return result

    def _get_records(self):
        if not self.res_model:
            return None
        if self.res_ids:
            records = self.env[self.res_model].browse(literal_eval(self.res_ids))
        elif self.res_id:
            records = self.env[self.res_model].browse(self.res_id)
        else:
            records = self.env[self.res_model]

        records = records.with_context(mail_notify_author=True)
        return records

```

## File: wizard\sms_composer_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sms_composer_view_form" model="ir.ui.view">
        <field name="name">sms.composer.view.form</field>
        <field name="model">sms.composer</field>
        <field name="arch" type="xml">
            <form string="Send an SMS">
                <!-- Single mode information (invalid number) -->
                <div class="alert alert-danger text-center" role="alert"
                    invisible="not res_model_description or not comment_single_recipient or recipient_single_valid">
                    <p class="my-0">
                        <strong>Invalid number:</strong>
                        <span> make sure to set a country on the </span>
                        <span><field name="res_model_description"/></span>
                        <span> or to specify the country code.</span>
                    </p>
                </div>

                <!-- Mass mode information (res_ids versus active domain) -->
                <div class="alert alert-info text-center" role="alert"
                        invisible="comment_single_recipient or recipient_invalid_count == 0">
                    <p class="my-0">
                        <field class="oe_inline fw-bold" name="recipient_invalid_count"/> out of
                        <field class="oe_inline fw-bold" name="res_ids_count"/> recipients have an invalid phone number and will not receive this text message.
                    </p>
                </div>
                <sheet>
                    <group>
                        <field name="composition_mode" invisible="1"/>
                        <field name="comment_single_recipient" invisible="1"/>
                        <field name="res_id" invisible="1"/>
                        <field name="res_ids" invisible="1"/>
                        <field name="res_model" invisible="1"/>
                        <field name="mass_force_send" invisible="1"/>
                        <field name="recipient_single_valid" invisible="1"/>
                        <field name="recipient_single_number" invisible="1"/>
                        <field name="number_field_name" invisible="1"/>
                        <field name="numbers" invisible="1"/>
                        <field name="sanitized_numbers" invisible="1"/>
                        <field name="template_id" invisible="1"/>

                        <label for="recipient_single_description" string="Recipient"
                            class="fw-bold"
                            invisible="not comment_single_recipient"/>
                        <div invisible="not comment_single_recipient">
                            <field name="recipient_single_description" class="oe_inline" invisible="not recipient_single_description"/>
                            <field name="recipient_single_number_itf" class="oe_inline" nolabel="1" onchange_on_keydown="True" placeholder="e.g. +1 415 555 0100"/>
                        </div>
                        <field name="body" widget="sms_widget" invisible="comment_single_recipient or recipient_single_valid"
                            options="{'dynamic_placeholder': true, 'dynamic_placeholder_model_reference_field': 'res_model'}"/>
                        <field name="body" widget="sms_widget" invisible="comment_single_recipient or not recipient_single_valid" default_focus="1"
                            options="{'dynamic_placeholder': true, 'dynamic_placeholder_model_reference_field': 'res_model'}"/>
                        <field name="body" widget="sms_widget" invisible="not comment_single_recipient"/>
                        <field name="mass_keep_log" invisible="1"/>
                    </group>
                </sheet>
                <footer>
                    <!-- attrs doesn't work for 'disabled'-->
                    <button string="Send SMS" type="object" class="oe_highlight" name="action_send_sms" data-hotkey="q"
                            invisible="composition_mode not in ('comment', 'numbers') or not recipient_single_valid"/>
                    <button string="Send SMS" type="object" class="oe_highlight" name="action_send_sms" data-hotkey="q"
                            invisible="composition_mode not in ('comment', 'numbers') or recipient_single_valid" disabled='1'/>
                    <button string="Put in queue" type="object" class="oe_highlight" name="action_send_sms" data-hotkey="q"
                        invisible="composition_mode != 'mass'"/>
                    <button string="Send Now" type="object" name="action_send_sms_mass_now" data-hotkey="w"
                        invisible="composition_mode != 'mass'"/>
                    <button string="Close" class="btn btn-secondary" special="cancel" data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="sms_composer_action_form" model="ir.actions.act_window">
        <field name="name">Send SMS</field>
        <field name="res_model">sms.composer</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>

</odoo>

```

## File: wizard\sms_resend.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, exceptions, fields, models


class SMSRecipient(models.TransientModel):
    _name = 'sms.resend.recipient'
    _description = 'Resend Notification'
    _rec_name = 'sms_resend_id'

    sms_resend_id = fields.Many2one('sms.resend', required=True)
    notification_id = fields.Many2one('mail.notification', required=True, ondelete='cascade')
    resend = fields.Boolean(string='Try Again', default=True)
    failure_type = fields.Selection(
        related='notification_id.failure_type', string='Error Message', related_sudo=True, readonly=True)
    partner_id = fields.Many2one('res.partner', 'Partner', related='notification_id.res_partner_id', readonly=True)
    partner_name = fields.Char(string='Recipient Name', readonly=True)
    sms_number = fields.Char(string='Phone Number')


class SMSResend(models.TransientModel):
    _name = 'sms.resend'
    _description = 'SMS Resend'
    _rec_name = 'mail_message_id'

    @api.model
    def default_get(self, fields):
        result = super(SMSResend, self).default_get(fields)
        if 'recipient_ids' in fields and result.get('mail_message_id'):
            mail_message_id = self.env['mail.message'].browse(result['mail_message_id'])
            result['recipient_ids'] = [(0, 0, {
                'notification_id': notif.id,
                'resend': True,
                'failure_type': notif.failure_type,
                'partner_name': notif.res_partner_id.display_name or mail_message_id.record_name,
                'sms_number': notif.sms_number,
            }) for notif in mail_message_id.notification_ids if notif.notification_type == 'sms' and notif.notification_status in ('exception', 'bounce')]
        return result

    mail_message_id = fields.Many2one('mail.message', 'Message', readonly=True, required=True)
    recipient_ids = fields.One2many('sms.resend.recipient', 'sms_resend_id', string='Recipients')
    can_cancel = fields.Boolean(compute='_compute_can_cancel')
    can_resend = fields.Boolean(compute='_compute_can_resend')
    has_insufficient_credit = fields.Boolean(compute='_compute_has_insufficient_credit')
    has_unregistered_account = fields.Boolean(compute='_compute_has_unregistered_account')

    @api.depends("recipient_ids.failure_type")
    def _compute_has_unregistered_account(self):
        self.has_unregistered_account = self.recipient_ids.filtered(lambda p: p.failure_type == 'sms_acc')

    @api.depends("recipient_ids.failure_type")
    def _compute_has_insufficient_credit(self):
        self.has_insufficient_credit = self.recipient_ids.filtered(lambda p: p.failure_type == 'sms_credit')

    @api.depends("recipient_ids.resend")
    def _compute_can_cancel(self):
        self.can_cancel = self.recipient_ids.filtered(lambda p: not p.resend)

    @api.depends('recipient_ids.resend')
    def _compute_can_resend(self):
        self.can_resend = any([recipient.resend for recipient in self.recipient_ids])

    def _check_special_access(self):
        if not self.mail_message_id or not self.mail_message_id.model or not self.mail_message_id.res_id:
            raise exceptions.UserError(_('You do not have access to the message and/or related document.'))
        record = self.env[self.mail_message_id.model].browse(self.mail_message_id.res_id)
        record.check_access('read')

    def action_resend(self):
        self._check_special_access()

        all_notifications = self.env['mail.notification'].sudo().search([
            ('mail_message_id', '=', self.mail_message_id.id),
            ('notification_type', '=', 'sms'),
            ('notification_status', 'in', ('exception', 'bounce'))
        ])
        sudo_self = self.sudo()
        to_cancel_ids = [r.notification_id.id for r in sudo_self.recipient_ids if not r.resend]
        to_resend_ids = [r.notification_id.id for r in sudo_self.recipient_ids if r.resend]

        if to_cancel_ids:
            all_notifications.filtered(lambda n: n.id in to_cancel_ids).write({'notification_status': 'canceled'})

        if to_resend_ids:
            record = self.env[self.mail_message_id.model].browse(self.mail_message_id.res_id)

            sms_pid_to_number = dict((r.partner_id.id, r.sms_number) for  r in self.recipient_ids if r.resend and r.partner_id)
            pids = list(sms_pid_to_number.keys())
            numbers = [r.sms_number for r in self.recipient_ids if r.resend and not r.partner_id]

            recipients_data = []
            all_recipients_data = self.env['mail.followers']._get_recipient_data(record, 'sms', False, pids=pids)[record.id]
            for pid, pdata in all_recipients_data.items():
                if pid and pdata['notif'] == 'sms':
                    recipients_data.append(pdata)
            if recipients_data or numbers:
                record._notify_thread_by_sms(
                    self.mail_message_id, recipients_data,
                    sms_numbers=numbers, sms_pid_to_number=sms_pid_to_number,
                    resend_existing=True, put_in_queue=False
                )

        self.mail_message_id._notify_message_notification_update()
        return {'type': 'ir.actions.act_window_close'}

    def action_cancel(self):
        self._check_special_access()

        sudo_self = self.sudo()
        sudo_self.mapped('recipient_ids.notification_id').write({'notification_status': 'canceled'})
        self.mail_message_id._notify_message_notification_update()
        return {'type': 'ir.actions.act_window_close'}

    def action_buy_credits(self):
        url = self.env['iap.account'].get_credits_url(service_name='sms')
        return {
            'type': 'ir.actions.act_url',
            'url': url,
        }

```

## File: wizard\sms_resend_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="mail_resend_message_view_form" model="ir.ui.view">
        <field name="name">sms.resend.form</field>
        <field name="model">sms.resend</field>
        <field name="groups_id" eval="[(4,ref('base.group_user'))]"/>
        <field name="arch" type="xml">
            <form string="Edit Partners">
                <field name="mail_message_id" invisible="1"/>
                <field name="can_resend" invisible="1"/>
                <field name="has_insufficient_credit" invisible="1"/>
                <field name="has_unregistered_account" invisible="1"/>
                <field name="recipient_ids">
                    <list string="Recipient" editable="top" create="0" delete="0">
                        <field name="partner_name"/>
                        <field name="sms_number"/>
                        <field name="failure_type" string="Reason" class="text-wrap"/>
                        <field name="resend" widget="boolean_toggle"/>
                        <field name="notification_id" column_invisible="True"/>
                    </list>
                </field>
                <footer>
                    <button string="Buy credits" name="action_buy_credits" type="object" class="btn-primary o_mail_send"
                            invisible="not has_insufficient_credit" data-hotkey="q"/>
                    <button string="Set up an account" name="action_buy_credits" type="object" class="btn-primary o_mail_send"
                            invisible="not has_unregistered_account" data-hotkey="q"/>
                    <button string="Send &amp; Close" name="action_resend" type="object" class="btn-primary o_mail_send"
                            invisible="not has_unregistered_account or not can_resend" data-hotkey="w"/>
                    <button string="Ignore all" name="action_cancel" type="object" class="btn-primary"
                            invisible="has_insufficient_credit or has_unregistered_account or has_unregistered_account and can_resend" data-hotkey="z"/>
                    <button string="Ignore all" name="action_cancel" type="object" class="btn-secondary"
                            invisible="not has_insufficient_credit and not has_unregistered_account and (not has_unregistered_account or not can_resend)" data-hotkey="z"/>
                    <button string="Close" class="btn-secondary" special="cancel" data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="sms_resend_action" model="ir.actions.act_window">
        <field name="name">Sending Failures</field>
        <field name="res_model">sms.resend</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
</data></odoo>

```

## File: wizard\sms_template_preview.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SMSTemplatePreview(models.TransientModel):
    _name = "sms.template.preview"
    _description = "SMS Template Preview"

    @api.model
    def _selection_target_model(self):
        return [(model.model, model.name) for model in self.env['ir.model'].sudo().search([])]

    @api.model
    def _selection_languages(self):
        return self.env['res.lang'].get_installed()

    @api.model
    def default_get(self, fields):
        result = super(SMSTemplatePreview, self).default_get(fields)
        sms_template_id = self.env.context.get('default_sms_template_id')
        if not sms_template_id or 'resource_ref' not in fields:
            return result
        sms_template = self.env['sms.template'].browse(sms_template_id)
        res = self.env[sms_template.model_id.model].search([], limit=1)
        if res:
            result['resource_ref'] = '%s,%s' % (sms_template.model_id.model, res.id)
        return result

    sms_template_id = fields.Many2one('sms.template', required=True, ondelete='cascade')
    lang = fields.Selection(_selection_languages, string='Template Preview Language')
    model_id = fields.Many2one('ir.model', related="sms_template_id.model_id")
    body = fields.Char('Body', compute='_compute_sms_template_fields')
    resource_ref = fields.Reference(string='Record reference', selection='_selection_target_model')
    no_record = fields.Boolean('No Record', compute='_compute_no_record')

    @api.depends('model_id')
    def _compute_no_record(self):
        for preview in self:
            preview.no_record = (self.env[preview.model_id.model].search_count([]) == 0) if preview.model_id else True

    @api.depends('lang', 'resource_ref')
    def _compute_sms_template_fields(self):
        for wizard in self:
            if wizard.sms_template_id and wizard.resource_ref:
                wizard.body = wizard.sms_template_id._render_field('body', [wizard.resource_ref.id], set_lang=wizard.lang)[wizard.resource_ref.id]
            else:
                wizard.body = wizard.sms_template_id.body

```

## File: wizard\sms_template_preview_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- SMS Template Preview -->
        <record model="ir.ui.view" id="sms_template_preview_form">
            <field name="name">sms.template.preview.form</field>
            <field name="model">sms.template.preview</field>
            <field name="arch" type="xml">
                <form string="SMS Preview">
                    <h3>Preview of <field name="sms_template_id" readonly="1" nolabel="1" class="oe_inline"/></h3>
                    <field name="no_record" invisible="1"/>
                    <div class="o_row">
                        <span>Choose an example <field name="model_id" readonly="1"/> record:</span>
                        <div>
                            <field name="resource_ref" class="oe_inline" options="{'hide_model': True, 'no_create': True, 'no_open': True}" invisible="no_record"/>
                            <span class="text-warning" invisible="not no_record">No records</span>
                        </div>
                    </div>
                    <p>Choose a language: <field name="lang" class="oe_inline ml8"/></p>
                    <label for="body" string="SMS content"/>
                    <hr/>
                        <field name="body" readonly="1" nolabel="1" options='{"safe": True}'/>
                    <hr/>
                    <footer>
                        <button string="Discard" class="btn-secondary" special="cancel" data-hotkey="x"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="sms_template_preview_action" model="ir.actions.act_window">
            <field name="name">Template Preview</field>
            <field name="res_model">sms.template.preview</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="sms_template_preview_form"/>
            <field name="target">new</field>
            <field name="context">{'default_sms_template_id':active_id}</field>
        </record>

    </data>
</odoo>

```

## File: wizard\sms_template_reset.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _


class SMSTemplateReset(models.TransientModel):
    _name = 'sms.template.reset'
    _description = 'SMS Template Reset'

    template_ids = fields.Many2many('sms.template')

    def reset_template(self):
        if not self.template_ids:
            return False
        self.template_ids.reset_template()
        if self.env.context.get('params', {}).get('view_type') == 'list':
            next_action = {'type': 'ir.actions.client', 'tag': 'reload'}
        else:
            next_action = {'type': 'ir.actions.act_window_close'}
        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'type': 'success',
                'message': _('SMS Templates have been reset'),
                'next': next_action,
            }
        }

```

## File: wizard\sms_template_reset_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="sms_template_reset_view_form" model="ir.ui.view">
        <field name="name">sms.template.reset.view.form</field>
        <field name="model">sms.template.reset</field>
        <field name="priority">1000</field>
        <field name="arch" type="xml">
            <form>
                <div>
                    Are you sure you want to reset these sms templates to their original configuration? Changes and translations will be lost.
                </div>
                <footer>
                    <button string="Proceed" class="btn btn-primary" type="object" name="reset_template" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>

	<record id="sms_template_reset_action" model="ir.actions.act_window">
        <field name="name">Reset SMS Template</field>
        <field name="res_model">sms.template.reset</field>
        <field name="binding_model_id" ref="sms.model_sms_template"/>
        <field name="binding_view_types">list</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="context">{
            'default_template_ids': active_ids
        }</field>
        <field name="view_id" ref="sms_template_reset_view_form"/>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-

from . import sms_account_code
from . import sms_account_phone
from . import sms_account_sender
from . import sms_composer
from . import sms_resend
from . import sms_template_preview
from . import sms_template_reset

```

