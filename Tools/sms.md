# Odoo Module: sms

Category: Tools

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
    'name': 'SMS gateway',
    'version': '2.0',
    'category': 'Tools',
    'summary': 'SMS Text Messaging',
    'description': """
This module gives a framework for SMS text messaging
----------------------------------------------------

The service is provided by the In App Purchase Odoo platform.
""",
    'depends': ['base', 'iap', 'mail', 'phone_validation'],
    'data': [
        'data/ir_cron_data.xml',
        'wizard/sms_cancel_views.xml',
        'wizard/sms_composer_views.xml',
        'wizard/sms_template_preview_views.xml',
        'wizard/sms_resend_views.xml',
        'views/ir_actions_views.xml',
        'views/res_config_settings_views.xml',
        'views/res_partner_views.xml',
        'views/assets.xml',
        'views/sms_sms_views.xml',
        'views/sms_template_views.xml',
        'security/ir.model.access.csv',
        'security/sms_security.xml',
    ],
    'demo': [
        'data/sms_demo.xml',
        'data/mail_demo.xml',
    ],
    'qweb': [
        'static/src/xml/thread.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

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
        <field name="numbercall">-1</field>
        <field eval="False" name="doall"/>
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
        <field name="mail_message_id" ref="message_demo_partner_1_1"/>
        <field name="res_partner_id" ref="base.res_partner_address_28"/>
        <field name="notification_type">email</field>
        <field name="notification_status">exception</field>
        <field name="failure_type">SMTP</field>
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
        <field name="body" type="html"><p>Hello! This is an example of another SMS with notifications.</p></field>
        <field name="message_type">sms</field>
        <field name="subtype_id" ref="mail.mt_note"/>
        <field name="author_id" ref="base.partner_admin"/>
        <field name="date" eval="(DateTime.today() - timedelta(days=2)).strftime('%Y-%m-%d %H:%M:00')"/>
    </record>
    <record id="message_demo_partner_1_3_notif_0" model="mail.notification">
        <field name="mail_message_id" ref="message_demo_partner_1_3"/>
        <field name="res_partner_id" ref="base.res_partner_address_28"/>
        <field name="notification_type">sms</field>
        <field name="notification_status">exception</field>
        <field name="failure_type">sms_credit</field>
    </record>
    <record id="message_demo_partner_1_3_notif_1" model="mail.notification">
        <field name="mail_message_id" ref="message_demo_partner_1_3"/>
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
        <field name="body">Dear ${object.display_name} this is an automated SMS.</field>
    </record>
</data></odoo>

```

## File: models\ir_actions.py

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
        ('sms', 'Send SMS Text Message'),
    ])
    # SMS
    sms_template_id = fields.Many2one(
        'sms.template', 'SMS Template', ondelete='set null',
        domain="[('model_id', '=', model_id)]",
    )
    sms_mass_keep_log = fields.Boolean('Log a note', default=True)

    @api.constrains('state', 'model_id')
    def _check_sms_capability(self):
        for action in self:
            if action.state == 'sms' and not action.model_id.is_mail_thread:
                raise ValidationError(_("Sending SMS can only be done on a mail.thread model"))

    @api.model
    def run_action_sms_multi(self, action, eval_context=None):
        # TDE CLEANME: when going to new api with server action, remove action
        if not action.sms_template_id or self._is_recompute(action):
            return False

        records = eval_context.get('records') or eval_context.get('record')
        if not records:
            return False

        composer = self.env['sms.composer'].with_context(
            default_res_model=records._name,
            default_res_ids=records.ids,
            default_composition_mode='mass',
            default_template_id=action.sms_template_id.id,
            default_mass_keep_log=action.sms_mass_keep_log,
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
                potential_fields = ModelObject._sms_get_number_fields() + ModelObject._sms_get_partner_fields()
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
            potential_fields = ModelObject._sms_get_number_fields() + ModelObject._sms_get_partner_fields()
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

from odoo import api, fields, models


class Followers(models.Model):
    _inherit = ['mail.followers']

    def _get_recipient_data(self, records, message_type, subtype_id, pids=None, cids=None):
        if message_type == 'sms':
            if pids is None:
                sms_pids = records._sms_get_default_partners().ids
            else:
                sms_pids = pids
            res = super(Followers, self)._get_recipient_data(records, message_type, subtype_id, pids=pids, cids=cids)
            new_res = []
            for pid, cid, pactive, pshare, ctype, notif, groups in res:
                if pid and pid in sms_pids:
                    notif = 'sms'
                new_res.append((pid, cid, pactive, pshare, ctype, notif, groups))
            return new_res
        else:
            return super(Followers, self)._get_recipient_data(records, message_type, subtype_id, pids=pids, cids=cids)

```

## File: models\mail_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from operator import itemgetter

from odoo import api, exceptions, fields, models
from odoo.tools import groupby


class MailMessage(models.Model):
    """ Override MailMessage class in order to add a new type: SMS messages.
    Those messages comes with their own notification method, using SMS
    gateway. """
    _inherit = 'mail.message'

    message_type = fields.Selection(selection_add=[('sms', 'SMS')])
    has_sms_error = fields.Boolean(
        'Has SMS error', compute='_compute_has_sms_error', search='_search_has_sms_error',
        help='Has error')

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

    def _format_mail_failures(self):
        """ A shorter message to notify a SMS delivery failure update

        TDE FIXME: should be cleaned
        """
        res = super(MailMessage, self)._format_mail_failures()

        # prepare notifications computation in batch
        all_notifications = self.env['mail.notification'].sudo().search([
            ('mail_message_id', 'in', self.ids)
        ])
        msgid_to_notif = defaultdict(lambda: self.env['mail.notification'].sudo())
        for notif in all_notifications:
            msgid_to_notif[notif.mail_message_id.id] += notif

        for message in self:
            notifications = msgid_to_notif[message.id]
            if not any(notification.notification_type == 'sms' for notification in notifications):
                continue
            info = dict(message._get_mail_failure_dict(),
                        failure_type='sms',
                        notifications=dict((notif.res_partner_id.id, (notif.notification_status, notif.res_partner_id.name)) for notif in notifications if notif.notification_type == 'sms'),
                        module_icon='/sms/static/img/sms_failure.svg'
                        )
            res.append(info)
        return res

    def _notify_sms_update(self):
        """ Send bus notifications to update status of notifications in chatter.
        Purpose is to send the updated status per author.

        TDE FIXME: author_id strategy seems curious, check with JS """
        messages = self.env['mail.message']
        for message in self:
            # YTI FIXME: check allowed_company_ids if necessary
            if message.model and message.res_id:
                record = self.env[message.model].browse(message.res_id)
                try:
                    record.check_access_rights('read')
                    record.check_access_rule('read')
                except exceptions.AccessError:
                    continue
                else:
                    messages |= message

        """ Notify channels after update of SMS status """
        updates = [[
            (self._cr.dbname, 'res.partner', author.id),
            {'type': 'sms_update', 'elements': self.env['mail.message'].concat(*author_messages)._format_mail_failures()}
        ] for author, author_messages in groupby(messages, itemgetter('author_id'))]
        self.env['bus.bus'].sendmany(updates)

    def message_format(self):
        """ Override in order to retrieves data about SMS (recipient name and
            SMS status)

        TDE FIXME: clean the overall message_format thingy
        """
        message_values = super(MailMessage, self).message_format()
        all_sms_notifications = self.env['mail.notification'].sudo().search([
            ('mail_message_id', 'in', [r['id'] for r in message_values]),
            ('notification_type', '=', 'sms')
        ])
        msgid_to_notif = defaultdict(lambda: self.env['mail.notification'].sudo())
        for notif in all_sms_notifications:
            msgid_to_notif[notif.mail_message_id.id] += notif

        for message in message_values:
            customer_sms_data = [(notif.id, notif.res_partner_id.display_name or notif.sms_number, notif.notification_status) for notif in msgid_to_notif.get(message['id'], [])]
            message['sms_ids'] = customer_sms_data
        return message_values

```

## File: models\mail_notification.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models
from odoo.tools.translate import _


class Notification(models.Model):
    _inherit = 'mail.notification'

    notification_type = fields.Selection(selection_add=[('sms', 'SMS')])
    sms_id = fields.Many2one('sms.sms', string='SMS', index=True, ondelete='set null')
    sms_number = fields.Char('SMS Number')
    failure_type = fields.Selection(selection_add=[
        ('sms_number_missing', 'Missing Number'),
        ('sms_number_format', 'Wrong Number Format'),
        ('sms_credit', 'Insufficient Credit'),
        ('sms_server', 'Server Error')]
    )

```

## File: models\mail_thread.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, models, fields
from odoo.addons.phone_validation.tools import phone_validation
from odoo.tools import html2plaintext, plaintext2html

_logger = logging.getLogger(__name__)


class MailThread(models.AbstractModel):
    _inherit = 'mail.thread'

    message_has_sms_error = fields.Boolean(
        'SMS Delivery error', compute='_compute_message_has_sms_error', search='_search_message_has_sms_error',
        help="If checked, some messages have a delivery error.")

    def _compute_message_has_sms_error(self):
        res = {}
        if self.ids:
            self._cr.execute(""" SELECT msg.res_id, COUNT(msg.res_id) FROM mail_message msg
                                 RIGHT JOIN mail_message_res_partner_needaction_rel rel
                                 ON rel.mail_message_id = msg.id AND rel.notification_type = 'sms' AND rel.notification_status in ('exception')
                                 WHERE msg.author_id = %s AND msg.model = %s AND msg.res_id in %s AND msg.message_type != 'user_notification'
                                 GROUP BY msg.res_id""",
                             (self.env.user.partner_id.id, self._name, tuple(self.ids),))
            res.update(self._cr.fetchall())

        for record in self:
            record.message_has_sms_error = bool(res.get(record._origin.id, 0))

    @api.model
    def _search_message_has_sms_error(self, operator, operand):
        return ['&', ('message_ids.has_sms_error', operator, operand), ('message_ids.author_id', '=', self.env.user.partner_id.id)]

    def _sms_get_partner_fields(self):
        """ This method returns the fields to use to find the contact to link
        whensending an SMS. Having partner is not necessary, having only phone
        number fields is possible. However it gives more flexibility to
        notifications management when having partners. """
        fields = []
        if hasattr(self, 'partner_id'):
            fields.append('partner_id')
        if hasattr(self, 'partner_ids'):
            fields.append('partner_ids')
        return fields

    def _sms_get_default_partners(self):
        """ This method will likely need to be overridden by inherited models.
               :returns partners: recordset of res.partner
        """
        partners = self.env['res.partner']
        for fname in self._sms_get_partner_fields():
            partners |= self.mapped(fname)
        return partners

    def _sms_get_number_fields(self):
        """ This method returns the fields to use to find the number to use to
        send an SMS on a record. """
        return ['mobile']

    def _sms_get_recipients_info(self, force_field=False):
        """" Get SMS recipient information on current record set. This method
        checks for numbers and sanitation in order to centralize computation.

        Example of use cases

          * click on a field -> number is actually forced from field, find customer
            linked to record, force its number to field or fallback on customer fields;
          * contact -> find numbers from all possible phone fields on record, find
            customer, force its number to found field number or fallback on customer fields;

        :return dict: record.id: {
            'partner': a res.partner recordset that is the customer (void or singleton);
            'sanitized': sanitized number to use (coming from record's field or partner's mobile
              or phone). Set to False is number impossible to parse and format;
            'number': original number before sanitation;
        } for each record in self
        """
        result = dict.fromkeys(self.ids, False)
        number_fields = self._sms_get_number_fields()
        for record in self:
            tocheck_fields = [force_field] if force_field else number_fields
            all_numbers = [record[fname] for fname in tocheck_fields if fname in record]
            all_partners = record._sms_get_default_partners()

            valid_number = False
            for fname in [f for f in tocheck_fields if f in record]:
                valid_number = phone_validation.phone_sanitize_numbers_w_record([record[fname]], record)[record[fname]]['sanitized']
                if valid_number:
                    break

            if valid_number:
                result[record.id] = {
                    'partner': all_partners[0] if all_partners else self.env['res.partner'],
                    'sanitized': valid_number, 'number': valid_number,
                }
            elif all_partners:
                partner_number, partner = False, self.env['res.partner']
                for partner in all_partners:
                    partner_number = partner.mobile or partner.phone
                    if partner_number:
                        partner_number = phone_validation.phone_sanitize_numbers_w_record([partner_number], record)[partner_number]['sanitized']
                    if partner_number:
                        break

                if partner_number:
                    result[record.id] = {'partner': partner, 'sanitized': partner_number, 'number': partner_number}
                else:
                    result[record.id] = {'partner': partner, 'sanitized': False, 'number': partner.mobile or partner.phone}
            elif all_numbers:
                result[record.id] = {'partner': self.env['res.partner'], 'sanitized': False, 'number': all_numbers[0]}
            else:
                result[record.id] = {'partner': self.env['res.partner'], 'sanitized': False, 'number': False}
        return result

    def _message_sms_schedule_mass(self, body='', template=False, active_domain=None, **composer_values):
        """ Shortcut method to schedule a mass sms sending on a recordset.

        :param template: an optional sms.template record;
        :param active_domain: bypass self.ids and apply composer on active_domain
          instead;
        """
        composer_context = {
            'default_res_model': self._name,
            'default_composition_mode': 'mass',
            'default_template_id': template.id if template else False,
            'default_body': body if body and not template else False,
        }
        if active_domain is not None:
            composer_context['default_use_active_domain'] = True
            composer_context['default_active_domain'] = repr(active_domain)
        else:
            composer_context['default_res_ids'] = self.ids

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
        :param template_fallback: plaintext (jinja-enabled) in case template
          and template xml id are falsy (for example due to deleted data);
        """
        self.ensure_one()
        if not template and template_xmlid:
            template = self.env.ref(template_xmlid, raise_if_not_found=False)
        if template:
            template_w_lang = template._get_context_lang_per_id(self.ids)[self.id]
            body = template._render_template(template_w_lang.body, self._name, self.ids)[self.id]
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
        :param sms_numbers: see ``_notify_record_by_sms``;
        :param sms_pid_to_number: see ``_notify_record_by_sms``;
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
            if info_number and not info_partner_ids:
                sms_numbers = [info_number] + (sms_numbers or [])

        if subtype_id is False:
            subtype_id = self.env['ir.model.data'].xmlid_to_res_id('mail.mt_note')

        return self.message_post(
            body=plaintext2html(html2plaintext(body)), partner_ids=partner_ids or [],  # TDE FIXME: temp fix otherwise crash mail_thread.py
            message_type='sms', subtype_id=subtype_id,
            sms_numbers=sms_numbers, sms_pid_to_number=sms_pid_to_number,
            **kwargs
        )

    def _notify_thread(self, message, msg_vals=False, **kwargs):
        recipients_data = super(MailThread, self)._notify_thread(message, msg_vals=msg_vals, **kwargs)
        self._notify_record_by_sms(message, recipients_data, msg_vals=msg_vals, **kwargs)
        return recipients_data

    def _notify_record_by_sms(self, message, recipients_data, msg_vals=False,
                              sms_numbers=None, sms_pid_to_number=None,
                              check_existing=False, put_in_queue=False, **kwargs):
        """ Notification method: by SMS.

        :param message: mail.message record to notify;
        :param recipients_data: see ``_notify_thread``;
        :param msg_vals: see ``_notify_thread``;

        :param sms_numbers: additional numbers to notify in addition to partners
          and classic recipients;
        :param pid_to_number: force a number to notify for a given partner ID
              instead of taking its mobile / phone number;
        :param check_existing: check for existing notifications to update based on
          mailed recipient, otherwise create new notifications;
        :param put_in_queue: use cron to send queued SMS instead of sending them
          directly;
        """
        sms_pid_to_number = sms_pid_to_number if sms_pid_to_number is not None else {}
        sms_numbers = sms_numbers if sms_numbers is not None else []
        sms_create_vals = []
        sms_all = self.env['sms.sms'].sudo()

        # pre-compute SMS data
        body = msg_vals['body'] if msg_vals and msg_vals.get('body') else message.body
        sms_base_vals = {
            'body': html2plaintext(body),
            'mail_message_id': message.id,
            'state': 'outgoing',
        }

        # notify from computed recipients_data (followers, specific recipients)
        partners_data = [r for r in recipients_data['partners'] if r['notif'] == 'sms']
        partner_ids = [r['id'] for r in partners_data]
        if partner_ids:
            for partner in self.env['res.partner'].sudo().browse(partner_ids):
                number = sms_pid_to_number.get(partner.id) or partner.mobile or partner.phone
                sanitize_res = phone_validation.phone_sanitize_numbers_w_record([number], partner)[number]
                number = sanitize_res['sanitized'] or number
                sms_create_vals.append(dict(
                    sms_base_vals,
                    partner_id=partner.id,
                    number=number
                ))

        # notify from additional numbers
        if sms_numbers:
            sanitized = phone_validation.phone_sanitize_numbers_w_record(sms_numbers, self)
            tocreate_numbers = [
                value['sanitized'] or original
                for original, value in sanitized.items()
                if value['code'] != 'empty'
            ]
            sms_create_vals += [dict(sms_base_vals, partner_id=False, number=n) for n in tocreate_numbers]

        # create sms and notification
        existing_pids, existing_numbers = [], []
        if sms_create_vals:
            sms_all |= self.env['sms.sms'].sudo().create(sms_create_vals)

            if check_existing:
                existing = self.env['mail.notification'].sudo().search([
                    '|', ('res_partner_id', 'in', partner_ids),
                    '&', ('res_partner_id', '=', False), ('sms_number', 'in', sms_numbers),
                    ('notification_type', '=', 'sms'),
                    ('mail_message_id', '=', message.id)
                ])
                for n in existing:
                    if n.res_partner_id.id in partner_ids and n.mail_message_id == message:
                        existing_pids.append(n.res_partner_id.id)
                    if not n.res_partner_id and n.sms_number in sms_numbers and n.mail_message_id == message:
                        existing_numbers.append(n.sms_number)

            notif_create_values = [{
                'mail_message_id': message.id,
                'res_partner_id': sms.partner_id.id,
                'sms_number': sms.number,
                'notification_type': 'sms',
                'sms_id': sms.id,
                'is_read': True,  # discard Inbox notification
                'notification_status': 'ready',
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
                            'sms_id': sms.id,
                            'sms_number': sms.number,
                        })

        if sms_all and not put_in_queue:
            sms_all.send(auto_commit=False, raise_exception=False)

        return True

```

## File: models\mail_thread_phone.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class PhoneMixin(models.AbstractModel):
    _inherit = 'mail.thread.phone'

    def _phone_get_number_fields(self):
        """ Add fields coming from sms implementation. """
        sms_fields = self._sms_get_number_fields()
        res = super(PhoneMixin, self)._phone_get_number_fields()
        for fname in (f for f in res if f not in sms_fields):
            sms_fields.append(fname)
        return sms_fields

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ResPartner(models.Model):
    _name = 'res.partner'
    _inherit = ['res.partner', 'mail.thread.phone']

    def _sms_get_default_partners(self):
        """ Override of mail.thread method.
            SMS recipients on partners are the partners themselves.
        """
        return self

    def _sms_get_number_fields(self):
        """ This method returns the fields to use to find the number to use to
        send an SMS on a record. """
        return ['mobile', 'phone']

```

## File: models\sms_api.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models
from odoo.addons.iap.models import iap

DEFAULT_ENDPOINT = 'https://iap-sms.odoo.com'


class SmsApi(models.AbstractModel):
    _name = 'sms.api'
    _description = 'SMS API'

    @api.model
    def _contact_iap(self, local_endpoint, params):
        account = self.env['iap.account'].get('sms')
        params['account_token'] = account.account_token
        endpoint = self.env['ir.config_parameter'].sudo().get_param('sms.endpoint', DEFAULT_ENDPOINT)
        # TODO PRO, the default timeout is 15, do we have to increase it ?
        return iap.jsonrpc(endpoint + local_endpoint, params=params)

    @api.model
    def _send_sms(self, numbers, message):
        """ Send a single message to several numbers

        :param numbers: list of E164 formatted phone numbers
        :param message: content to send

        :raises ? TDE FIXME
        """
        params = {
            'numbers': numbers,
            'message': message,
        }
        return self._contact_iap('/iap/message_send', params)

    @api.model
    def _send_sms_batch(self, messages):
        """ Send SMS using IAP in batch mode

        :param messages: list of SMS to send, structured as dict [{
            'res_id':  integer: ID of sms.sms,
            'number':  string: E164 formatted phone number,
            'content': string: content to send
        }]

        :return: return of /iap/sms/1/send controller which is a list of dict [{
            'res_id': integer: ID of sms.sms,
            'state':  string: 'insufficient_credit' or 'wrong_number_format' or 'success',
            'credit': integer: number of credits spent to send this SMS,
        }]

        :raises: normally none
        """
        params = {
            'messages': messages
        }
        return self._contact_iap('/iap/sms/1/send', params)

```

## File: models\sms_sms.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import threading

from odoo import api, fields, models, tools

_logger = logging.getLogger(__name__)


class SmsSms(models.Model):
    _name = 'sms.sms'
    _description = 'Outgoing SMS'
    _rec_name = 'number'
    _order = 'id DESC'

    IAP_TO_SMS_STATE = {
        'success': 'sent',
        'insufficient_credit': 'sms_credit',
        'wrong_number_format': 'sms_number_format',
        'server_error': 'sms_server'
    }

    number = fields.Char('Number')
    body = fields.Text()
    partner_id = fields.Many2one('res.partner', 'Customer')
    mail_message_id = fields.Many2one('mail.message', index=True)
    state = fields.Selection([
        ('outgoing', 'In Queue'),
        ('sent', 'Sent'),
        ('error', 'Error'),
        ('canceled', 'Canceled')
    ], 'SMS Status', readonly=True, copy=False, default='outgoing', required=True)
    error_code = fields.Selection([
        ('sms_number_missing', 'Missing Number'),
        ('sms_number_format', 'Wrong Number Format'),
        ('sms_credit', 'Insufficient Credit'),
        ('sms_server', 'Server Error'),
        # mass mode specific codes
        ('sms_blacklist', 'Blacklisted'),
        ('sms_duplicate', 'Duplicate'),
    ], copy=False)

    def send(self, delete_all=False, auto_commit=False, raise_exception=False):
        """ Main API method to send SMS.

          :param delete_all: delete all SMS (sent or not); otherwise delete only
            sent SMS;
          :param auto_commit: commit after each batch of SMS;
          :param raise_exception: raise if there is an issue contacting IAP;
        """
        for batch_ids in self._split_batch():
            self.browse(batch_ids)._send(delete_all=delete_all, raise_exception=raise_exception)
            # auto-commit if asked except in testing mode
            if auto_commit is True and not getattr(threading.currentThread(), 'testing', False):
                self._cr.commit()

    def cancel(self):
        self.state = 'canceled'

    @api.model
    def _process_queue(self, ids=None):
        """ Send immediately queued messages, committing after each message is sent.
        This is not transactional and should not be called during another transaction!

       :param list ids: optional list of emails ids to send. If passed no search
         is performed, and these ids are used instead.
        """
        domain = [('state', '=', 'outgoing')]

        filtered_ids = self.search(domain, limit=10000).ids  # TDE note: arbitrary limit we might have to update
        if ids:
            ids = list(set(filtered_ids) & set(ids))
        else:
            ids = filtered_ids
        ids.sort()

        res = None
        try:
            # auto-commit except in testing mode
            auto_commit = not getattr(threading.currentThread(), 'testing', False)
            res = self.browse(ids).send(delete_all=False, auto_commit=auto_commit, raise_exception=False)
        except Exception:
            _logger.exception("Failed processing SMS queue")
        return res

    def _split_batch(self):
        batch_size = int(self.env['ir.config_parameter'].sudo().get_param('sms.session.batch.size', 500))
        for sms_batch in tools.split_every(batch_size, self.ids):
            yield sms_batch

    def _send(self, delete_all=False, raise_exception=False):
        """ This method tries to send SMS after checking the number (presence and
        formatting). """
        iap_data = [{
            'res_id': record.id,
            'number': record.number,
            'content': record.body,
        } for record in self]

        try:
            iap_results = self.env['sms.api']._send_sms_batch(iap_data)
        except Exception as e:
            _logger.info('Sent batch %s SMS: %s: failed with exception %s', len(self.ids), self.ids, e)
            if raise_exception:
                raise
            self._postprocess_iap_sent_sms([{'res_id': sms.id, 'state': 'server_error'} for sms in self], delete_all=delete_all)
        else:
            _logger.info('Send batch %s SMS: %s: gave %s', len(self.ids), self.ids, iap_results)
            self._postprocess_iap_sent_sms(iap_results, delete_all=delete_all)

    def _postprocess_iap_sent_sms(self, iap_results, failure_reason=None, delete_all=False):
        if delete_all:
            todelete_sms_ids = [item['res_id'] for item in iap_results]
        else:
            todelete_sms_ids = [item['res_id'] for item in iap_results if item['state'] == 'success']

        for state in self.IAP_TO_SMS_STATE.keys():
            sms_ids = [item['res_id'] for item in iap_results if item['state'] == state]
            if sms_ids:
                if state != 'success' and not delete_all:
                    self.env['sms.sms'].sudo().browse(sms_ids).write({
                        'state': 'error',
                        'error_code': self.IAP_TO_SMS_STATE[state],
                    })
                notifications = self.env['mail.notification'].sudo().search([
                    ('notification_type', '=', 'sms'),
                    ('sms_id', 'in', sms_ids),
                    ('notification_status', 'not in', ('sent', 'canceled'))]
                )
                if notifications:
                    notifications.write({
                        'notification_status': 'sent' if state == 'success' else 'exception',
                        'failure_type': self.IAP_TO_SMS_STATE[state] if state != 'success' else False,
                        'failure_reason': failure_reason if failure_reason else False,
                    })

        if todelete_sms_ids:
            self.browse(todelete_sms_ids).sudo().unlink()

```

## File: models\sms_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class SMSTemplate(models.Model):
    "Templates for sending SMS"
    _name = "sms.template"
    _description = 'SMS Templates'

    @api.model
    def default_get(self, fields):
        res = super(SMSTemplate, self).default_get(fields)
        if not fields or 'model_id' in fields and not res.get('model_id') and res.get('model'):
            res['model_id'] = self.env['ir.model']._get(res['model']).id
        return res

    name = fields.Char('Name', translate=True)
    model_id = fields.Many2one(
        'ir.model', string='Applies to', required=True,
        domain=['&', ('is_mail_thread_sms', '=', True), ('transient', '=', False)],
        help="The type of document this template can be used with")
    model = fields.Char('Related Document Model', related='model_id.model', index=True, store=True, readonly=True)
    body = fields.Char('Body', translate=True, required=True)
    lang = fields.Char('Language', help="Use this field to either force a specific language (ISO code) or dynamically "
                                        "detect the language of your recipient by a placeholder expression "
                                        "(e.g. ${object.partner_id.lang})")
    # Use to create contextual action (same as for email template)
    sidebar_action_id = fields.Many2one('ir.actions.act_window', 'Sidebar action', readonly=True, copy=False,
                                        help="Sidebar action to make this template available on records "
                                        "of the related document model")
    # Fake fields used to implement the placeholder assistant
    model_object_field = fields.Many2one('ir.model.fields', string="Field", store=False,
                                         help="Select target field from the related document model.\n"
                                              "If it is a relationship field you will be able to select "
                                              "a target field at the destination of the relationship.")
    sub_object = fields.Many2one('ir.model', 'Sub-model', readonly=True, store=False,
                                 help="When a relationship field is selected as first field, "
                                      "this field shows the document model the relationship goes to.")
    sub_model_object_field = fields.Many2one('ir.model.fields', 'Sub-field', store=False,
                                             help="When a relationship field is selected as first field, "
                                                  "this field lets you select the target field within the "
                                                  "destination document model (sub-model).")
    null_value = fields.Char('Default Value',  store=False, help="Optional value to use if the target field is empty")
    copyvalue = fields.Char('Placeholder Expression', store=False,
                            help="Final placeholder expression, to be copy-pasted in the desired template field.")

    @api.onchange('model_object_field', 'sub_model_object_field', 'null_value')
    def _onchange_dynamic_placeholder(self):
        """ Generate the dynamic placeholder """
        if self.model_object_field:
            if self.model_object_field.ttype in ['many2one', 'one2many', 'many2many']:
                model = self.env['ir.model']._get(self.model_object_field.relation)
                if model:
                    self.sub_object = model.id
                    sub_field_name = self.sub_model_object_field.name
                    self.copyvalue = self._build_expression(self.model_object_field.name,
                                                            sub_field_name, self.null_value or False)
            else:
                self.sub_object = False
                self.sub_model_object_field = False
                self.copyvalue = self._build_expression(self.model_object_field.name, False, self.null_value or False)
        else:
            self.sub_object = False
            self.copyvalue = False
            self.sub_model_object_field = False
            self.null_value = False

    @api.model
    def _build_expression(self, field_name, sub_field_name, null_value):
        """Returns a placeholder expression for use in a template field,
        based on the values provided in the placeholder assistant.

        :param field_name: main field name
        :param sub_field_name: sub field name (M2O)
        :param null_value: default value if the target value is empty
        :return: final placeholder expression """
        expression = ''
        if field_name:
            expression = "${object." + field_name
            if sub_field_name:
                expression += "." + sub_field_name
            if null_value:
                expression += " or '''%s'''" % null_value
            expression += "}"
        return expression

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        default = dict(default or {},
                       name=_("%s (copy)") % self.name)
        return super(SMSTemplate, self).copy(default=default)

    def action_create_sidebar_action(self):
        ActWindow = self.env['ir.actions.act_window']
        view = self.env.ref('sms.sms_composer_view_form')

        for template in self:
            button_name = _('Send SMS (%s)') % template.name
            action = ActWindow.create({
                'name': button_name,
                'type': 'ir.actions.act_window',
                'res_model': 'sms.composer',
                # Add default_composition_mode to guess to determine if need to use mass or comment composer
                'context': "{'default_template_id' : %d, 'default_composition_mode': 'guess', 'default_res_ids': active_ids, 'default_res_id': active_id}" % (template.id),
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

    def _get_context_lang_per_id(self, res_ids):
        self.ensure_one()
        if res_ids is None:
            return {None: self}

        if self.env.context.get('template_preview_lang'):
            lang = self.env.context.get('template_preview_lang')
            results = dict((res_id, self.with_context(lang=lang)) for res_id in res_ids)
        else:
            rendered_langs = self._render_template(self.lang, self.model, res_ids)
            results = dict((res_id, self.with_context(lang=lang) if lang else self)
                for res_id, lang in rendered_langs.items())

        return results

    def _get_ids_per_lang(self, res_ids):
        self.ensure_one()

        rids_to_tpl = self._get_context_lang_per_id(res_ids)
        tpl_to_rids = {}
        for res_id, template in rids_to_tpl.items():
            tpl_to_rids.setdefault(template._context.get('lang', self.env.user.lang), []).append(res_id)

        return tpl_to_rids

    def _get_translated_bodies(self, res_ids):
        """ return sms translated bodies into a dict {'res_id':'body'} """
        self.ensure_one()
        lang_to_rids = self._get_ids_per_lang(res_ids)
        all_bodies = {}
        for lang, rids in lang_to_rids.items():
            template = self.with_context(lang=lang)
            all_bodies.update(template._render_template(template.body, self.model, rids))
        return all_bodies

    @api.model
    def _render_template(self, template_txt, model, res_ids):
        """ Render the jinja template """
        return self.env['mail.template']._render_template(template_txt, model, res_ids)

    def unlink(self):
        self.sudo().mapped('sidebar_action_id').unlink()
        return super(SMSTemplate, self).unlink()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_actions
from . import ir_model
from . import mail_followers
from . import mail_message
from . import mail_notification
from . import mail_thread
from . import mail_thread_phone
from . import res_partner
from . import sms_api
from . import sms_sms
from . import sms_template

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_sms_sms_all,access.sms.sms.all,model_sms_sms,,0,0,0,0
access_sms_sms_system,access.sms.sms.system,model_sms_sms,base.group_system,1,1,1,1
access_sms_template_all,access.sms.template.all,model_sms_template,,0,0,0,0
access_sms_template_user,access.sms.template.user,model_sms_template,base.group_user,1,0,0,0
access_sms_template_system,access.sms.template.system,model_sms_template,base.group_system,1,1,1,1

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

## File: static\img\sms_failure.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="512" height="512"><defs><path id="a" d="M91.974 123.535v-17.07c0-.839-.283-1.542-.851-2.111-.568-.57-1.24-.854-2.017-.854H71.894c-.777 0-1.449.285-2.017.854-.568.569-.851 1.272-.851 2.11v17.071c0 .839.283 1.542.851 2.111.568.57 1.24.854 2.017.854h17.212c.777 0 1.449-.285 2.017-.854.568-.569.851-1.272.851-2.11zm-.179-33.601l1.614-41.239c0-.718-.3-1.287-.897-1.707-.777-.659-1.494-.988-2.151-.988H70.639c-.657 0-1.374.33-2.151.988-.598.42-.897 1.048-.897 1.887l1.524 41.059c0 .599.3 1.093.897 1.482.597.39 1.314.584 2.151.584h16.584c.837 0 1.54-.195 2.107-.584.568-.39.881-.883.941-1.482zM90.54 6.02l68.846 126.5c2.092 3.773 2.032 7.546-.179 11.32a11.276 11.276 0 0 1-4.168 4.133 11.192 11.192 0 0 1-5.693 1.527H11.654c-2.032 0-3.93-.51-5.693-1.527a11.276 11.276 0 0 1-4.168-4.133c-2.211-3.774-2.271-7.547-.18-11.32L70.46 6.02a11.462 11.462 0 0 1 4.213-4.403A11.105 11.105 0 0 1 80.5 0c2.092 0 4.034.54 5.827 1.617A11.462 11.462 0 0 1 90.54 6.02z"/><path id="c" d="M91.974 123.535v-17.07c0-.839-.283-1.542-.851-2.111-.568-.57-1.24-.854-2.017-.854H71.894c-.777 0-1.449.285-2.017.854-.568.569-.851 1.272-.851 2.11v17.071c0 .839.283 1.542.851 2.111.568.57 1.24.854 2.017.854h17.212c.777 0 1.449-.285 2.017-.854.568-.569.851-1.272.851-2.11zm-.179-33.601l1.614-41.239c0-.718-.3-1.287-.897-1.707-.777-.659-1.494-.988-2.151-.988H70.639c-.657 0-1.374.33-2.151.988-.598.42-.897 1.048-.897 1.887l1.524 41.059c0 .599.3 1.093.897 1.482.597.39 1.314.584 2.151.584h16.584c.837 0 1.54-.195 2.107-.584.568-.39.881-.883.941-1.482zM90.54 6.02l68.846 126.5c2.092 3.773 2.032 7.546-.179 11.32a11.276 11.276 0 0 1-4.168 4.133 11.192 11.192 0 0 1-5.693 1.527H11.654c-2.032 0-3.93-.51-5.693-1.527a11.276 11.276 0 0 1-4.168-4.133c-2.211-3.774-2.271-7.547-.18-11.32L70.46 6.02a11.462 11.462 0 0 1 4.213-4.403A11.105 11.105 0 0 1 80.5 0c2.092 0 4.034.54 5.827 1.617A11.462 11.462 0 0 1 90.54 6.02z"/></defs><g fill="none" fill-rule="evenodd"><circle cx="256" cy="253" r="256" fill="#FDA20C"/><path fill="#000" fill-opacity=".3" fill-rule="nonzero" d="M361 98.292C361 90.982 353.978 85 345.396 85H163.604C155.022 85 148 90.981 148 98.292v296.416c0 7.31 7.022 13.292 15.604 13.292h181.792c8.582 0 15.604-5.981 15.604-13.292v-64.636c-5.462 3.323-11.703 6.646-17.945 9.305v33.399c0 1.329-.78 1.994-2.34 1.994h-172.43c-1.56 0-2.34-.665-2.34-1.994V126.87c0-1.329.78-1.993 2.34-1.993h172.43c1.56 0 2.34.664 2.34 1.993v63.113c3.901 0 8.582-.664 12.483-.664H361V98.292zM254.5 380c5.067 0 9.5 4.433 9.5 9.5s-4.433 9.5-9.5 9.5-9.5-4.433-9.5-9.5 4.433-9.5 9.5-9.5zm24.577-266.907h-47.593c-2.341 0-3.902-1.56-3.902-3.9s1.56-3.899 3.902-3.899h47.593c2.34 0 3.901 1.56 3.901 3.9 0 2.339-2.34 3.899-3.901 3.899z"/><path fill="#FFF" fill-rule="nonzero" d="M355.538 321.966c-3.9 0-8.582 0-12.483-.665v45.438c0 1.33-.78 1.996-2.34 1.996h-172.43c-1.56 0-2.34-.665-2.34-1.996V120.58c0-1.33.78-1.996 2.34-1.996h172.43c1.56 0 2.34.665 2.34 1.996v63.81c3.901 0 8.582-.666 12.483-.666H361V91.306C361 83.988 353.978 78 345.396 78H163.604C155.022 78 148 83.988 148 91.306v297.388c0 7.318 7.022 13.306 15.604 13.306h181.792c8.582 0 15.604-5.988 15.604-13.306v-66.728h-5.462zM230.703 98.871h47.594c2.34 0 3.9 1.56 3.9 3.901 0 2.34-1.56 3.902-3.12 3.902h-47.593c-2.341 0-3.902-1.561-3.902-3.902 0-2.34.78-3.901 3.121-3.901zM254.5 394c-5.067 0-9.5-4.433-9.5-9.5s4.433-9.5 9.5-9.5 9.5 4.433 9.5 9.5c0 5.7-4.433 9.5-9.5 9.5z"/><g opacity=".437" transform="translate(217 160)"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g fill="#2F3136" mask="url(#b)"><path d="M0 0H161V161H0z"/></g></g><g transform="translate(217 149)"><mask id="d" fill="#fff"><use xlink:href="#c"/></mask><g fill="#FFF" mask="url(#d)"><path d="M0 0H161V161H0z"/></g></g></g></svg>
```

## File: static\src\js\fields_phone_widget.js

```javascript
odoo.define('sms.fields', function (require) {
"use strict";

var basic_fields = require('web.basic_fields');
var core = require('web.core');
var session = require('web.session');

var _t = core._t;

/**
 * Override of FieldPhone to use add a button calling SMS composer if option activated
 */

var Phone = basic_fields.FieldPhone;
Phone.include({
    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Open SMS composer wizard
     *
     * @private
     */
    _onClickSMS: function (ev) {
        ev.preventDefault();

        var context = session.user_context;
        context = _.extend({}, context, {
            default_res_model: this.model,
            default_res_id: parseInt(this.res_id),
            default_number_field_name: this.name,
            default_composition_mode: 'comment',
        });
        var self = this;
        return this.do_action({
            title: _t('Send SMS Text Message'),
            type: 'ir.actions.act_window',
            res_model: 'sms.composer',
            target: 'new',
            views: [[false, 'form']],
            context: context,
        }, {
        on_close: function () {
            self.trigger_up('reload');
        }});
    },

    /**
     * Add a button to call the composer wizard
     *
     * @override
     * @private
     */
    _renderReadonly: function () {
        var def = this._super.apply(this, arguments);
        if (this.nodeOptions.enable_sms) {
            var $composerButton = $('<a>', {
                title: _t('Send SMS Text Message'),
                href: '',
                class: 'ml-3 d-inline-flex align-items-center o_field_phone_sms',
                html: $('<small>', {class: 'font-weight-bold ml-1', html: 'SMS'}),
            });
            $composerButton.prepend($('<i>', {class: 'fa fa-mobile'}));
            $composerButton.on('click', this._onClickSMS.bind(this));
            this.$el = $('<div/>').append(this.$el).append($composerButton);
        }

        return def;
    },
});

return Phone;

});

```

## File: static\src\js\fields_sms_widget.js

```javascript
odoo.define('sms.sms_widget', function (require) {
"use strict";

var basicFields = require('web.basic_fields');
var core = require('web.core');
var fieldRegistry = require('web.field_registry');

var FieldText = basicFields.FieldText;

var _t = core._t;
/**
 * SmsWidget is a widget to display a textarea (the body) and a text representing
 * the number of SMS and the number of characters. This text is computed every
 * time the user changes the body.
 */
var SmsWidget = FieldText.extend({
    className: 'o_field_text',
    /**
     * @constructor
     */
    init: function () {
        this._super.apply(this, arguments);
        this.nbrChar = 0;
        this.nbrSMS = 0;
        this.encoding = 'GSM7';
    },

    //--------------------------------------------------------------------------
    // Private: override widget
    //--------------------------------------------------------------------------

    /**
     * @private
     * @override
     */
    _renderEdit: function () {
        var def = this._super.apply(this, arguments);

        this._compute();
        $('.o_sms_container').remove();
        var $sms_container = $('<div class="o_sms_container"/>');
        $sms_container.append(this._renderSMSInfo());
        $sms_container.append(this._renderIAPButton());
        this.$el = this.$el.add($sms_container);

        return def;
    },

    //--------------------------------------------------------------------------
    // Private: SMS
    //--------------------------------------------------------------------------

    /**
     * Compute the number of characters and sms
     * @private
     */
    _compute: function () {
        var content = this._getValue();
        this.encoding = this._extractEncoding(content);
        this.nbrChar = content.length;
        this.nbrChar += (content.match(/\n/g) || []).length;
        this.nbrSMS = this._countSMS(this.nbrChar, this.encoding);
    },

    /**
     * Count the number of SMS of the content
     * @private
     * @returns {integer} Number of SMS
     */
    _countSMS: function () {
        if (this.nbrChar === 0) {
            return 0;
        }
        if (this.encoding === 'UNICODE') {
            if (this.nbrChar <= 70) {
                return 1;
            }
            return Math.ceil(this.nbrChar / 67);
        }
        if (this.nbrChar <= 160) {
            return 1;
        }
        return Math.ceil(this.nbrChar / 153);
    },

    /**
     * Extract the encoding depending on the characters in the content
     * @private
     * @param {String} content Content of the SMS
     * @returns {String} Encoding of the content (GSM7 or UNICODE)
     */
    _extractEncoding: function (content) {
        if (String(content).match(RegExp("^[@£$¥èéùìòÇ\\nØø\\rÅåΔ_ΦΓΛΩΠΨΣΘΞÆæßÉ !\\\"#¤%&'()*+,-./0123456789:;<=>?¡ABCDEFGHIJKLMNOPQRSTUVWXYZÄÖÑÜ§¿abcdefghijklmnopqrstuvwxyzäöñüà]*$"))) {
            return 'GSM7';
        }
        return 'UNICODE';
    },

    /**
     * Render the IAP button to redirect to IAP pricing
     * @private
     */
    _renderIAPButton: function () {
        return $('<a>', {
            'href': 'https://iap-services.odoo.com/iap/sms/pricing',
            'target': '_blank',
            'title': _t('SMS Pricing'),
            'aria-label': _t('SMS Pricing'),
            'class': 'fa fa-lg fa-info-circle',
        });
    },

    /**
     * Render the number of characters, sms and the encoding.
     * @private
     */
    _renderSMSInfo: function () {
        var string = _.str.sprintf(_t('%s characters, fits in %s SMS (%s) '), this.nbrChar, this.nbrSMS, this.encoding);
        var $span = $('<span>', {
            'class': 'text-muted o_sms_count',
        });
        $span.text(string);
        return $span;
    },

    /**
     * Update widget SMS information with re-computed info about length, ...
     * @private
     */
    _updateSMSInfo: function ()  {
        this._compute();
        var string = _.str.sprintf(_t('%s characters, fits in %s SMS (%s) '), this.nbrChar, this.nbrSMS, this.encoding);
        this.$('.o_sms_count').text(string);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @override
     * @private
     */
    _onChange: function () {
        this._super.apply(this, arguments);
        this._updateSMSInfo();
    },

    /**
     * @override
     * @private
     */
    _onInput: function () {
        this._super.apply(this, arguments);
        this._updateSMSInfo();
    },
});

fieldRegistry.add('sms_widget', SmsWidget);

return SmsWidget;
});

```

## File: static\src\js\mail_failure.js

```javascript
odoo.define('sms.model.MailFailure', function (require) {
'use strict';

var MailFailure = require('mail.model.MailFailure');
var core = require('web.core');
var _t = core._t;

MailFailure.include({

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    getPreview: function () {
        var preview = this._super.apply(this, arguments);
        if (this._failureType === 'sms') {
            _.extend(preview, {
                body: _t('An error occurred when sending an SMS'),
                id: 'sms_failure',
            });
        }
        return preview;
    },
});

});

```

## File: static\src\js\message.js

```javascript
odoo.define('sms.model.Message', function (require) {
"use strict";

var Message = require('mail.model.Message');

Message.include({

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Retrieves the list of SMS
     *
     * @returns {Array} Array of Array(smsID, recipientName, smsStatus)
     */
    getSmsIds: function () {
        return this._smsIds;
    },
    /**
     * Retrieves the SMS status
     *
     * @return {string}
     */
    getSmsStatus: function () {
        var self = this;
        this._smsStatus = 'sent';
        _.each(this._smsIds, function (sms) {
            if (sms[2] === 'bounce' || sms[2] === 'exception') {
                self._smsStatus = 'error';
            }
        });
        return this._smsStatus;
    },
    /**
     * Whether message has nay SMS-related notification
     *
     * @returns {boolean}
     */
    hasSmsData: function () {
        return !!(this._smsIds && (this._smsIds.length > 0));
    },
    /**
     * Does the message contains at least one SMS failure
     *
     * @returns {boolean}
     */
    isError: function () {
        return this.getSmsStatus() === 'error';
    },
    /**
     * Update the status of a SMS
     *
     * @param {integer} smsID ID of the SMS to update
     * @param {string} smsStatus New status of the SMS
     */
    setSmsStatus: function (smsID, smsStatus) {
        var self = this;
        var sms = _.find(self._smsIds, function (sms) {
            return sms[0] === smsID;
        });
        if (sms !== undefined && smsStatus !== undefined) {
            sms[2] = smsStatus;
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     * @private
     */
    _setInitialData: function (data) {
        this._super.apply(this, arguments);
        this._smsIds = data.sms_ids;
        if (this._smsStatus === false) {
            this._smsStatus = 'sent';
        }
    },
});
});

```

## File: static\src\js\sms_notification_manager.js

```javascript
odoo.define('sms.NotificationManager', function (require) {
"use strict";

var MailManager = require('mail.Manager');
var MailFailure = require('mail.model.MailFailure');

MailManager.include({

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @override
     * @param {Object} data structure depending on the type
     * @param {integer} data.id
     */
    _handlePartnerNotification: function (data) {
        if (data.type === 'sms_update') {
            this._handleSMSUpdateNotification(data);
        } else {
            this._super.apply(this, arguments);
        }
    },
    /**
     * Updates message in thread when there's an update in a SMS letter
     *
     * @private
     * @param {Object} datas
     * @param {Object[]} datas.elements list of SMS failure data
     * @param {string} datas.elements[].message_id ID of related message that
     *   has a sms failure.
     */
    _handleSMSUpdateNotification: function (datas) {
        var self = this;
        _.each(datas.elements, function (data) {
            var isNewFailure = data.sms_status === 'error';
            var matchedFailure = _.find(self._mailFailures, function (failure) {
                return failure.getMessageID() === data.message_id;
            });

            if (matchedFailure) {
                var index = _.findIndex(self._mailFailures, matchedFailure);
                if (isNewFailure) {
                    self._mailFailures[index] = new MailFailure(self, data);
                } else {
                    self._mailFailures.splice(index, 1);
                }
            } else if (isNewFailure) {
                self._mailFailures.push(new MailFailure(self, data));
            }
            var message = _.find(self._messages, function (msg) {
                return msg.getID() === data.message_id;
            });
            if (message) {
                message.setSmsStatus(data.sms_id, data.sms_status);
                self._mailBus.trigger('update_message', message);
            }
        });
        this._mailBus.trigger('update_needaction', this.needactionCounter);
    },
});

});

```

## File: static\src\js\systray_messaging_menu.js

```javascript
odoo.define('sms.systray.MessagingMenu', function (require) {
"use strict";

var core = require('web.core');
var MessagingMenu = require('mail.systray.MessagingMenu');

var _t = core._t;

MessagingMenu.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Called when clicking on a preview related to a snailmail failure
     *
     * @private
     * @param {$.Element} $target DOM of preview element clicked
     */
    _clickSMSFailurePreview: function ($target) {
        var documentID = $target.data('document-id');
        var documentModel = $target.data('document-model');
        if (documentModel && documentID) {
            this._openDocument(documentModel, documentID);
        } else if (documentModel !== 'mail.channel') {
            // preview of SMS failures grouped to different document of same model
            this.do_action({
                name: _t('SMS Failures'),
                type: 'ir.actions.act_window',
                view_mode: 'kanban,list,form',
                views: [[false, 'kanban'], [false, 'list'], [false, 'form']],
                target: 'current',
                res_model: documentModel,
                domain: [['message_has_sms_error', '=', true]],
            });
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @override
     */
   _onClickPreview: function (ev) {
       var $target = $(ev.currentTarget);
       var previewID = $target.data('preview-id');
        if (previewID === 'sms_failure') {
            this._clickSMSFailurePreview($target);
        } else {
            this._super.apply(this, arguments);
        }
   },
    /**
     * @private
     * @override
     */
    _onClickPreviewMarkAsRead: function (ev) {
        ev.stopPropagation();
        var $preview = $(ev.currentTarget).closest('.o_mail_preview');
        var previewID = $preview.data('preview-id');
        if (previewID === 'sms_failure') {
            var documentModel = $preview.data('document-model');
            var unreadCounter = $preview.data('unread-counter');
            this.do_action('sms.sms_cancel_action', {
                additional_context: {
                    default_model: documentModel,
                    unread_counter: unreadCounter
                }
            });
        } else {
            this._super.apply(this, arguments);
        }
    },
});

});

```

## File: static\src\js\thread_widget.js

```javascript
odoo.define('sms.widget.Thread', function (require) {
"use strict";

var ThreadWidget = require('mail.widget.Thread');
var core = require('web.core');

var QWeb = core.qweb;

ThreadWidget.include({
    events: _.extend({}, ThreadWidget.prototype.events, {
        'click .o_thread_message_sms_error': '_onClickSMSError'
    }),
    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this._enabledOptions = _.defaults(this._enabledOptions, {
            displaySmsIcons: true,
        });
        this._disabledOptions = _.defaults(this._disabledOptions, {
            displaySmsIcons: false,
        });
    },
    /**
     * @override
     */
    render: function (thread, options) {
        this._super.apply(this, arguments);
        var messages = _.clone(thread.getMessages({domain: options.domain || []}));
        this._renderMessageSmsPopover(messages);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Render the popover when mouse-hovering on the mail icon of a message
     * in the thread. There is at most one such popover at any given time.
     *
     * @private
     * @param {mail.model.AbstractMessage[]} messages list of messages in the
     *   rendered thread, for which popover on mouseover interaction is
     *   permitted.
     */
    _renderMessageSmsPopover: function (messages) {
        if (this._messageSmsPopover) {
            this._messageSmsPopover.popover('hide');
        }
        if (!this.$('.o_thread_sms_tooltip').length) {
            return;
        }
        this._messageSmsPopover = this.$('.o_thread_sms_tooltip').popover({
            html: true,
            boundary: 'viewport',
            placement: 'auto',
            trigger: 'hover',
            offset: '0, 1',
            content: function () {
                var messageID = $(this).data('message-id');
                var message = _.find(messages, function (message) {
                    return message.getID() === messageID;
                });
                return QWeb.render('sms.widget.Thread.Message.SmsTooltip', {
                    data: message.getSmsIds()
                });
            },
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {MouseEvent} ev
     */
    _onClickSMSError: function (ev) {
        var messageID = $(ev.currentTarget).data('message-id');
        this.do_action('sms.sms_resend_action', {
            additional_context: {
                default_mail_message_id: messageID
            }
        });
    },
});
});

```

## File: static\src\xml\thread.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<template id="template" xml:space="preserve">

     <t t-extend="mail.widget.Thread.Message">
        <t t-jquery=".o_thread_tooltip_container" t-operation="after">
            <span t-if="message.getType() === 'sms' and message.hasSmsData() and options.displaySmsIcons" class="o_thread_sms_tooltip_container ml-1">
                <t t-set="thread_icon_class" t-value="'o_thread_sms_tooltip o_thread_message_sms o_thread_message_sms_' + message.getSmsStatus()" />
                <span t-attf-class="d-inline-flex align-items-center o_thread_sms_tooltip o_thread_message_sms o_thread_message_sms_#{message.getSmsStatus()} #{message.isError() ? 'o_thread_message_sms_error' : ''}"
                       t-att-data-message-id="message.getID()">
                    <i class="fa fa-mobile"/>
                    <small class="font-weight-bold ml-1">SMS</small>
                </span>
            </span>
        </t>
    </t>

     <t t-name="sms.widget.Thread.Message.SmsTooltip">
        <div t-foreach="data" t-as="customerSMSData">
            <span class="d-inline-block text-center o_thread_tooltip_icon">
                <i t-if="customerSMSData[2] === 'sent'" class='fa fa-check' title="Sent" role="img" aria-label="Sent"/>
                <i t-if="customerSMSData[2] === 'canceled'" class='fa fa-trash-o' title="Canceled" role="img" aria-label="Canceled"/>
                <i t-if="customerSMSData[2] === 'outgoing'" class='fa fa-clock-o' title="Awaiting Dispatch" role="img" aria-label="Awaiting Dispatch"/>
                <i t-if="customerSMSData[2] === 'exception'" class='fa fa-exclamation text-danger' title="Error" role="img" aria-label="Error"/>
            </span>
            <span t-esc="customerSMSData[1]"/>
        </div>
    </t>

 </template>

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="assets_backend" name="sms_assets" inherit_id="web.assets_backend">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/sms/static/src/js/fields_phone_widget.js"></script>
                <script type="text/javascript" src="/sms/static/src/js/fields_sms_widget.js"></script>
                <script type="text/javascript" src="/sms/static/src/js/mail_failure.js"></script>
                <script type="text/javascript" src="/sms/static/src/js/message.js"></script>
                <script type="text/javascript" src="/sms/static/src/js/sms_notification_manager.js"></script>
                <script type="text/javascript" src="/sms/static/src/js/systray_messaging_menu.js"></script>
                <script type="text/javascript" src="/sms/static/src/js/thread_widget.js"></script>
                <link rel="stylesheet" type="text/scss" href="/sms/static/src/scss/thread.scss"/>
            </xpath>
        </template>

        <template id="qunit_suite" name="sms_widget_tests" inherit_id="web.qunit_suite">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/sms/static/tests/sms_widget_test.js"></script>
            </xpath>
        </template>
    </data>
</odoo>

```

## File: views\ir_actions_views.xml

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
                    attrs="{'invisible': [('state', '!=', 'sms')],
                            'required': [('state', '=', 'sms')]}"/>
                <field name="sms_mass_keep_log"
                    attrs="{'invisible': [('state', '!=', 'sms')]}"/>
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
            <div id="sms_settings" position="inside">
                <widget name="iap_buy_more_credits" service_name="sms"/>
            </div>
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
                <label for="phone"/>
                <div class="o_row">
                    <field name="phone" widget="phone" options="{'enable_sms': True}"/>
                </div>
            </xpath>
            <xpath expr="//field[@name='mobile']" position="replace">
                <label for="mobile"/>
                <div class="o_row">
                    <field name="mobile" widget="phone" options="{'enable_sms': True}"/>
                </div>
            </xpath>
        </field>
    </record>

    <!-- Add action entry in the Action Menu for Partners -->
    <act_window id="res_partner_act_window_sms_composer_multi"
        name="Send SMS Text Message"
        binding_model="res.partner"
        res_model="sms.composer"
        binding_views="list"
        view_mode="form"
        target="new"
        context="{
            'default_composition_mode': 'mass',
            'default_mass_keep_log': True,
            'default_res_ids': active_ids
        }"
    />
    <act_window id="res_partner_act_window_sms_composer_single"
        name="Send SMS Text Message"
        binding_model="res.partner"
        res_model="sms.composer"
        binding_views="form"
        view_mode="form"
        target="new"
        context="{
            'default_composition_mode': 'comment',
            'default_res_id': active_id,
        }"
    />

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
                    <button name="send" string="Send Now" type="object" states='outgoing' class="oe_highlight"/>
                    <button name="cancel" string="Cancel" type="object" states='outgoing'/>
                    <field name="state" widget="statusbar"/>
                </header>
                <sheet>
                    <group>
                        <field name="body"/>
                    </group>
                    <group>
                        <group>
                            <field name="partner_id"/>
                            <field name="number"/>
                        </group>
                        <group>
                            <field name="error_code"/>
                            <field name="mail_message_id"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="sms_sms_view_tree" model="ir.ui.view">
        <field name="name">sms.sms.view.tree</field>
        <field name="model">sms.sms</field>
        <field name="arch" type="xml">
            <tree string="SMS Templates">
                <field name="number"/>
                <field name="partner_id"/>
                <field name="state"/>
                <field name="error_code"/>
            </tree>
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
        <field name="view_mode">tree,form</field>
    </record>

    <menuitem id="sms_sms_menu"
        parent="phone_validation.phone_menu_main"
        action="sms_sms_action"
        sequence="1"/>

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
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <field name="sidebar_action_id" invisible="1"/>
                        <button name="action_create_sidebar_action" type="object"
                                groups="base.group_system"
                                class="oe_stat_button"  
                                attrs="{'invisible':[('sidebar_action_id','!=',False)]}" icon="fa-plus"
                                help="Add a contextual action on the related model to open a sms composer with this template">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_text">Add</span>
                                <span class="o_stat_text">Context Action</span>
                            </div>
                        </button>
                        <button name="action_unlink_sidebar_action" type="object"
                                groups="base.group_system"
                                class="oe_stat_button" icon="fa-minus"
                                attrs="{'invisible':[('sidebar_action_id','=',False)]}"
                                help="Remove the contextual action of the related model" widget="statinfo">
                            <div class="o_field_widget o_stat_info">
                                <span class="o_stat_text">Remove</span>
                                <span class="o_stat_text">Context Action</span>
                            </div>
                        </button>
                        <button class="oe_stat_button" name="%(sms_template_preview_action)d" icon="fa-search-plus" string="Preview" type="action" target="new"/>
                    </div>
                    <div class="oe_title">
                        <label for="name" class="oe_edit_only" string="SMS Template"/>
                        <h1><field name="name" required="1"/></h1>
                        <group>
                            <field name="model_id" required="1" options="{'no_create': True}"/>
                            <field name="model" invisible="1"/>
                            <field name="lang" groups="base.group_no_one" placeholder="e.g. en_US or ${object.partner_id.lang}"/>
                        </group>
                    </div>
                    <notebook>
                        <page string="Content">
                            <group>
                                <field name="body" widget="sms_widget" nolabel="1"/>
                            </group>
                        </page>
                        <page string="Dynamic Placeholder Generator" groups="base.group_no_one">
                            <group>
                                <field name="model_object_field"
                                        domain="[('model_id','=',model_id),('ttype','!=','one2many'),('ttype','!=','many2many')]"/>
                                <field name="sub_object" readonly="1"/>
                                <field name="sub_model_object_field"
                                        domain="[('model_id','=',sub_object),('ttype','!=','one2many'),('ttype','!=','many2many')]"
                                        attrs="{'readonly':[('sub_object','=',False)],'required':[('sub_object','!=',False)]}"/>
                                <field name="null_value"/>
                                <field name="copyvalue"/>
                            </group>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="sms_template_view_tree" model="ir.ui.view">
        <field name="name">sms.template.view.tree</field>
        <field name="model">sms.template</field>
        <field name="arch" type="xml">
            <tree string="SMS Templates">
                <field name="name"/>
                <field name="model_id"/>
            </tree>
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
        <field name="view_mode">tree,form</field>
    </record>

    <menuitem id="sms_template_menu"
        name="SMS Templates"
        parent="phone_validation.phone_menu_main"
        sequence="2"
        action="sms_template_action"/>

</data></odoo>

```

## File: wizard\sms_cancel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models


class SMSCancel(models.TransientModel):
    _name = 'sms.cancel'
    _description = 'Dismiss notification for resend by model'

    model = fields.Char(string='Model', required=True)
    help_message = fields.Char(string='Help message', compute='_compute_help_message')

    @api.depends('model')
    def _compute_help_message(self):
        for wizard in self:
            wizard.help_message = _("Are you sure you want to discard %s SMS delivery failures. You won't be able to re-send these SMS later!") % (wizard._context.get('unread_counter'))

    def action_cancel(self):
        # TDE CHECK: delete pending SMS
        author_id = self.env.user.partner_id.id
        for wizard in self:
            self._cr.execute("""
SELECT notif.id, msg.id
FROM mail_message_res_partner_needaction_rel notif
JOIN mail_message msg
    ON notif.mail_message_id = msg.id
WHERE notif.notification_type = 'sms' IS TRUE AND notif.notification_status IN ('bounce', 'exception')
    AND msg.model = %s
    AND msg.author_id = %s """, (wizard.model, author_id))
            res = self._cr.fetchall()
            notif_ids = [row[0] for row in res]
            message_ids = list(set([row[1] for row in res]))
            if notif_ids:
                self.env['mail.notification'].browse(notif_ids).sudo().write({'notification_status': 'canceled'})
            if message_ids:
                self.env['mail.message'].browse(message_ids)._notify_sms_update()
        return {'type': 'ir.actions.act_window_close'}

```

## File: wizard\sms_cancel_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="sms_cancel" model="ir.ui.view">
        <field name="name">sms.cancel.form</field>
        <field name="model">sms.cancel</field>
        <field name="groups_id" eval="[(4,ref('base.group_user'))]"/>
        <field name="arch" type="xml">
            <form string="Cancel notification in failure">
                <field name="model" invisible='1'/>
                <field name="help_message"/>
                <p>If you want to re-send them, click Cancel now, then click on the notification and review them one by one by clicking on the red icon next to each message.</p>
                <footer>  
                    <button string="Discard delivery failures" name="action_cancel" type="object" class="btn-primary" />
                    <button string="Cancel" class="btn-secondary" special="cancel" />
                </footer>
            </form>
        </field>
    </record>

    <record id="sms_cancel_action" model="ir.actions.act_window">
        <field name="name">Discard SMS delivery failures</field>
        <field name="res_model">sms.cancel</field>
        <field name="type">ir.actions.act_window</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
</data></odoo>

```

## File: wizard\sms_composer.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval

from odoo import api, fields, models, _
from odoo.addons.phone_validation.tools import phone_validation
from odoo.exceptions import UserError
from odoo.tools.safe_eval import safe_eval
from odoo.tools import html2plaintext


class SendSMS(models.TransientModel):
    _name = 'sms.composer'
    _description = 'Send SMS Wizard'

    @api.model
    def default_get(self, fields):
        result = super(SendSMS, self).default_get(fields)
        if fields == 'partner_ids':
            # shortcut because default_get in cache, avoid issues
            return result

        result['res_model'] = result.get('res_model') or self.env.context.get('active_model')
        result['composition_mode'] = result.get('composition_mode')

        # guess the composition mode, if multiples ids => mass composition else comment
        if self.env.context.get('default_composition_mode') and self.env.context.get('default_composition_mode') == "guess":
            if self.env.context.get('active_ids') and len(self.env.context.get('active_ids')) > 1:
                result['composition_mode'] = 'mass'
                result['res_id'] = False
            else:
                result['composition_mode'] = 'comment'
                result['res_ids'] = False

        if not result.get('active_domain'):
            result['active_domain'] = repr(self.env.context.get('active_domain', []))
        if not result.get('res_id'):
            if not result.get('res_ids') and self.env.context.get('active_id'):
                result['res_id'] = self.env.context.get('active_id')
        if not result.get('res_ids'):
            if not result.get('res_id') and self.env.context.get('active_ids'):
                result['res_ids'] = repr(self.env.context.get('active_ids'))

        if result['res_model']:
            result.update(
                self._get_composer_values(
                    result['composition_mode'], result['res_model'], result.get('res_id'),
                    result.get('body'), result.get('template_id')
                )
            )
        return result

    # documents
    composition_mode = fields.Selection([
        ('numbers', 'Send to numbers'),
        ('comment', 'Post on a document'),
        ('mass', 'Send SMS in batch')],
        string='Composition Mode', default='comment', required=True)
    res_model = fields.Char('Document Model Name')
    res_id = fields.Integer('Document ID')
    res_ids = fields.Char('Document IDs')
    res_ids_count = fields.Integer(
        'Visible records count', compute='_compute_recipients_count', compute_sudo=False,
        help='UX field computing the number of recipients in mass mode without active domain')
    use_active_domain = fields.Boolean('Use active domain')
    active_domain = fields.Text('Active domain', readonly=True)
    active_domain_count = fields.Integer(
        'Active records count', compute='_compute_recipients_count', compute_sudo=False,
        help='UX field computing the number of recipients in mass mode based on given active domain')
    # options for comment and mass mode
    mass_keep_log = fields.Boolean('Keep a note on document', default=True)
    mass_force_send = fields.Boolean('Send directly', default=False)
    mass_use_blacklist = fields.Boolean('Use blacklist', default=True)
    # recipients
    recipient_description = fields.Text('Recipients (Partners)', compute='_compute_recipients', compute_sudo=False)
    recipient_count = fields.Integer('# Valid recipients', compute='_compute_recipients', compute_sudo=False)
    recipient_invalid_count = fields.Integer('# Invalid recipients', compute='_compute_recipients', compute_sudo=False)
    number_field_name = fields.Char(string='Field holding number')
    partner_ids = fields.Many2many('res.partner')
    numbers = fields.Char('Recipients (Numbers)')
    sanitized_numbers = fields.Char('Sanitized Number', compute='_compute_sanitized_numbers', compute_sudo=False)
    # content
    template_id = fields.Many2one('sms.template', string='Use Template', domain="[('model', '=', res_model)]")
    body = fields.Text('Message', required=True)

    @api.depends('res_model', 'res_ids', 'active_domain')
    def _compute_recipients_count(self):
        self.res_ids_count = len(literal_eval(self.res_ids)) if self.res_ids else 0
        if self.res_model:
            self.active_domain_count = self.env[self.res_model].search_count(safe_eval(self.active_domain or '[]'))
        else:
            self.active_domain_count = 0

    @api.depends('partner_ids', 'res_model', 'res_id', 'res_ids', 'use_active_domain', 'composition_mode', 'number_field_name', 'sanitized_numbers')
    def _compute_recipients(self):
        self.recipient_description = False
        self.recipient_count = 0
        self.recipient_invalid_count = 0

        if self.partner_ids:
            if len(self.partner_ids) == 1:
                self.recipient_description = '%s (%s)' % (self.partner_ids[0].display_name, self.partner_ids[0].mobile or self.partner_ids[0].phone or _('Missing number'))
            self.recipient_count = len(self.partner_ids)

        elif self.composition_mode in ('comment', 'mass') and self.res_model:
            records = self._get_records()

            if records and issubclass(type(records), self.pool['mail.thread']):
                res = records._sms_get_recipients_info(force_field=self.number_field_name)
                valid_ids = [rid for rid, rvalues in res.items() if rvalues['sanitized']]
                invalid_ids = [rid for rid, rvalues in res.items() if not rvalues['sanitized']]
                self.recipient_count = len(valid_ids)
                self.recipient_invalid_count = len(invalid_ids)
                if len(records) == 1:
                    self.recipient_description = '%s (%s)' % (
                        res[records.id]['partner'].name or records.display_name,
                        res[records.id]['sanitized'] or _("Invalid number")
                    )
            else:
                self.recipient_invalid_count = 0 if (self.sanitized_numbers or (self.composition_mode == 'mass' and self.use_active_domain)) else 1

    @api.depends('numbers', 'res_model', 'res_id')
    def _compute_sanitized_numbers(self):
        if self.numbers:
            record = self._get_records() if self.res_model and self.res_id else self.env.user
            numbers = [number.strip() for number in self.numbers.split(',')]
            sanitize_res = phone_validation.phone_sanitize_numbers_w_record(numbers, record)
            sanitized_numbers = [info['sanitized'] for info in sanitize_res.values() if info['sanitized']]
            invalid_numbers = [number for number, info in sanitize_res.items() if info['code']]
            if invalid_numbers:
                raise UserError(_('Following numbers are not correctly encoded: %s') % repr(invalid_numbers))
            self.sanitized_numbers = ','.join(sanitized_numbers)
        else:
            self.sanitized_numbers = False

    @api.onchange('composition_mode', 'res_model', 'res_id', 'template_id')
    def _onchange_template_id(self):
        if self.template_id and self.composition_mode == 'comment' and self.res_id:
            self.body = self.template_id._get_translated_bodies([self.res_id])[self.res_id]
        elif self.template_id:
            self.body = self.template_id.body

    # ------------------------------------------------------------
    # Actions
    # ------------------------------------------------------------

    def action_send_sms(self):
        if self.composition_mode in ('numbers', 'comment') and self.recipient_invalid_count:
            raise UserError(_('%s invalid recipients') % self.recipient_invalid_count)
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
            if records is not None and issubclass(type(records), self.pool['mail.thread']):
                return self._action_send_sms_comment(records)
            return self._action_send_sms_numbers()
        else:
            return self._action_send_sms_mass(records)

    def _action_send_sms_numbers(self):
        self.env['sms.api']._send_sms_batch([{
            'res_id': 0,
            'number': number,
            'content': self.body,
        } for number in self.sanitized_numbers.split(',')])
        return True

    def _action_send_sms_comment(self, records=None):
        records = records if records is not None else self._get_records()
        subtype_id = self.env['ir.model.data'].xmlid_to_res_id('mail.mt_note')

        messages = self.env['mail.message']
        all_bodies = self._prepare_body_values(records)

        for record in records:
            messages += record._message_sms(
                all_bodies[record.id],
                subtype_id=subtype_id,
                partner_ids=self.partner_ids.ids or False,
                number_field=self.number_field_name,
                sms_numbers=self.sanitized_numbers.split(',') if self.sanitized_numbers else None)
        return messages

    def _action_send_sms_mass(self, records=None):
        records = records if records is not None else self._get_records()

        sms_record_values = self._prepare_mass_sms_values(records)
        sms_all = self._prepare_mass_sms(records, sms_record_values)

        if sms_all and self.mass_keep_log and records and issubclass(type(records), self.pool['mail.thread']):
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
            all_bodies = self.template_id._get_translated_bodies(records.ids)
        else:
            all_bodies = self.env['mail.template']._render_template(self.body, records._name, records.ids)
        return all_bodies

    def _prepare_mass_sms_values(self, records):
        all_bodies = self._prepare_body_values(records)
        all_recipients = self._prepare_recipient_values(records)
        blacklist_ids = self._get_blacklist_record_ids(records, all_recipients)
        done_ids = self._get_done_record_ids(records, all_recipients)

        result = {}
        for record in records:
            recipients = all_recipients[record.id]
            sanitized = recipients['sanitized']
            if sanitized and record.id in blacklist_ids:
                state = 'canceled'
                error_code = 'sms_blacklist'
            elif sanitized and record.id in done_ids:
                state = 'canceled'
                error_code = 'sms_duplicate'
            elif not sanitized:
                state = 'error'
                error_code = 'sms_number_format' if recipients['number'] else 'sms_number_missing'
            else:
                state = 'outgoing'
                error_code = ''

            result[record.id] = {
                'body': all_bodies[record.id],
                'partner_id': recipients['partner'].id,
                'number': sanitized if sanitized else recipients['number'],
                'state': state,
                'error_code': error_code,
            }
        return result

    def _prepare_mass_sms(self, records, sms_record_values):
        sms_create_vals = [sms_record_values[record.id] for record in records]
        return self.env['sms.sms'].sudo().create(sms_create_vals)

    def _prepare_log_body_values(self, sms_records_values):
        result = {}
        for record_id, sms_values in sms_records_values.items():
            result[record_id] = html2plaintext(sms_values['body'])
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
        if self.use_active_domain:
            active_domain = safe_eval(self.active_domain or '[]')
            records = self.env[self.res_model].search(active_domain)
        elif self.res_id:
            records = self.env[self.res_model].browse(self.res_id)
        else:
            records = self.env[self.res_model].browse(literal_eval(self.res_ids or '[]'))
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
                <sheet>
                    <group>
                        <field name="composition_mode" invisible="1"/>
                        <field name="res_id" invisible="1"/>
                        <field name="res_ids" invisible="1"/>
                        <field name="active_domain" invisible="1"/>
                        <field name="res_model" invisible="1"/>
                        <field name="mass_force_send" invisible="1"/>
                        <field name="number_field_name" invisible="1"/>
                        <field name="partner_ids" invisible="1"/>
                        <field name="numbers" invisible="1"/>
                        <field name="sanitized_numbers" invisible="1"/>

                        <!-- Mass mode information (res_ids versus active domain) -->
                        <div colspan="2" class="alert alert-info text-center mb-3" role="alert"
                                attrs="{'invisible': [('composition_mode', '=', 'comment')]}">
                            <p class="my-0">
                                <span attrs="{'invisible': [('use_active_domain', '=', 'True')]}">
                                    <field class="oe_inline font-weight-bold" name="res_ids_count"/> records selected.
                                </span>
                                Check <field class="oe_inline ml-2" name="use_active_domain"/> to send to all
                                <field class="oe_inline font-weight-bold ml-2" name="active_domain_count"/> records instead. <br/>
                                <field class="oe_inline font-weight-bold" name="recipient_count"/> recipients are valid
                                and <field class="oe_inline font-weight-bold" name="recipient_invalid_count"/> are invalid.
                            </p>
                        </div>
                        <field name="recipient_description" string="Recipients"
                            readonly="1"
                            attrs="{'invisible': [('composition_mode', '!=', 'comment')]}"/>
                        <field name="body" widget="sms_widget"/>
                        <field name="mass_keep_log" invisible="1"/>
                    </group>
                </sheet>
                <footer>
                    <button string="Send SMS" type="object" class="oe_highlight" name="action_send_sms"
                        attrs="{'invisible': [('composition_mode', 'not in', ('comment', 'numbers'))]}"/>
                    <button string="Put in queue" type="object" class="oe_highlight" name="action_send_sms"
                        attrs="{'invisible': [('composition_mode', '!=', 'mass')]}"/>
                    <button string="Send Now" type="object" name="action_send_sms_mass_now"
                        attrs="{'invisible': [('composition_mode', '!=', 'mass')]}"/>
                    <button string="Close" class="btn btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="sms_composer_action_form" model="ir.actions.act_window">
        <field name="name">Send SMS Text Message</field>
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
    resend = fields.Boolean(string="Resend", default=True)
    failure_type = fields.Selection([
        ('sms_number_missing', 'Missing Number'),
        ('sms_number_format', 'Wrong Number Format'),
        ('sms_credit', 'Insufficient Credit'),
        ('sms_server', 'Server Error')], related='notification_id.failure_type', related_sudo=True, readonly=True)
    partner_id = fields.Many2one('res.partner', 'Partner', related='notification_id.res_partner_id', readonly=True)
    partner_name = fields.Char('Recipient', readonly='True')
    sms_number = fields.Char('Number')


class SMSResend(models.TransientModel):
    _name = 'sms.resend'
    _description = 'SMS Resend'
    _rec_name = 'mail_message_id'

    @api.model
    def default_get(self, fields):
        result = super(SMSResend, self).default_get(fields)
        if result.get('mail_message_id'):
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
    has_cancel = fields.Boolean(compute='_compute_has_cancel')
    has_insufficient_credit = fields.Boolean(compute='_compute_has_insufficient_credit') 

    @api.depends("recipient_ids.failure_type")
    def _compute_has_insufficient_credit(self):
        self.has_insufficient_credit = self.recipient_ids.filtered(lambda p: p.failure_type == 'sms_credit')

    @api.depends("recipient_ids.resend")
    def _compute_has_cancel(self):
        self.has_cancel = self.recipient_ids.filtered(lambda p: not p.resend)

    def _check_access(self):
        if not self.mail_message_id or not self.mail_message_id.model or not self.mail_message_id.res_id:
            raise exceptions.UserError(_('You do not have access to the message and/or related document.'))
        record = self.env[self.mail_message_id.model].browse(self.mail_message_id.res_id)
        record.check_access_rights('read')
        record.check_access_rule('read')

    def action_resend(self):
        self._check_access()

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

            rdata = []
            for pid, cid, active, pshare, ctype, notif, groups in self.env['mail.followers']._get_recipient_data(record, 'sms', False, pids=pids):
                if pid and notif == 'sms':
                    rdata.append({'id': pid, 'share': pshare, 'active': active, 'notif': notif, 'groups': groups or [], 'type': 'customer' if pshare else 'user'})
            if rdata or numbers:
                record._notify_record_by_sms(
                    self.mail_message_id, {'partners': rdata}, check_existing=True,
                    sms_numbers=numbers, sms_pid_to_number=sms_pid_to_number,
                    put_in_queue=False
                )

        self.mail_message_id._notify_sms_update()
        return {'type': 'ir.actions.act_window_close'}

    def action_cancel(self):
        self._check_access()

        sudo_self = self.sudo()
        sudo_self.mapped('recipient_ids.notification_id').write({'notification_status': 'canceled'})
        self.mail_message_id._notify_sms_update()
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
                <field name="has_cancel" invisible="1"/>
                <field name="has_insufficient_credit" invisible="1"/>
                <field name="recipient_ids">
                    <tree string="Recipient" editable="top" create="0" delete="0">
                        <field name="partner_name"/>
                        <field name="sms_number"/>
                        <field name="failure_type" string="Reason"/>
                        <field name="resend" widget="boolean_toggle"/>
                    </tree>
                </field>
                <div class="alert alert-warning" role="alert" attrs="{'invisible': [('has_cancel', '=', False)]}">
                    <span class="fa fa-info-circle"/> Caution: It won't be possible to send this SMS again to the recipients you did not select.
                </div>
                <footer>
                    <button string="Buy credits" name="action_buy_credits" type="object" class="btn-primary o_mail_send" 
                            attrs="{'invisible': [('has_insufficient_credit', '=', False)]}"/>
                    <button string="Resend" name="action_resend" type="object" class="btn-primary o_mail_send"/>
                    <button string="Ignore all" name="action_cancel" type="object" class="btn-secondary" />
                    <button string="Cancel" class="btn-secondary" special="cancel" />
                </footer>
            </form>
        </field>
    </record>
    
    <record id="sms_resend_action" model="ir.actions.act_window">
        <field name="name">Sending Failures</field>
        <field name="res_model">sms.resend</field>
        <field name="type">ir.actions.act_window</field>
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
    _inherit = "sms.template"
    _name = "sms.template.preview"
    _description = "SMS Template Preview"

    @api.model
    def _selection_target_model(self):
        models = self.env['ir.model'].search([])
        return [(model.model, model.name) for model in models]

    @api.model
    def _selection_languages(self):
        return self.env['res.lang'].get_installed()

    @api.model
    def default_get(self, fields):
        result = super(SMSTemplatePreview, self).default_get(fields)
        sms_template = self._context.get('default_sms_template_id') and self.env['sms.template'].browse(self._context['default_sms_template_id']) or False
        if sms_template and not result.get('res_id'):
            result['res_id'] = self.env[sms_template.model].search([], limit=1)
        return result

    sms_template_id = fields.Many2one('sms.template') # NOTE This should probably be required

    lang = fields.Selection(_selection_languages, string='Template Preview Language')
    model_id = fields.Many2one('ir.model', related="sms_template_id.model_id")
    res_id = fields.Integer(string='Record ID')
    resource_ref = fields.Reference(string='Record reference', selection='_selection_target_model', compute='_compute_resource_ref', inverse='_inverse_resource_ref')

    @api.depends('model_id', 'res_id')
    def _compute_resource_ref(self):
        for preview in self:
            if preview.model_id:
                preview.resource_ref = '%s,%s' % (preview.model_id.model, preview.res_id or 0)
            else:
                preview.resource_ref = False

    def _inverse_resource_ref(self):
        for preview in self:
            if preview.resource_ref:
                preview.res_id = preview.resource_ref.id

    @api.onchange('lang', 'resource_ref')
    def on_change_resource_ref(self):
        # Update res_id and body depending of the resource_ref
        if self.resource_ref:
            self.res_id = self.resource_ref.id
        if self.sms_template_id:
            template = self.sms_template_id.with_context(lang=self.lang)
            self.body = template._render_template(template.body, template.model, self.res_id or 0) if self.resource_ref else template.body

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
                    <div class="o_row">
                        <span>Choose an example <field name="model_id" readonly="1"/> record:</span>
                        <div>
                            <field name="resource_ref" class="oe_inline" options="{'hide_model': True, 'no_create': True, 'no_edit': True, 'no_open': True}"/>
                            <span class="text-warning" attrs="{'invisible':[('resource_ref','!=', False)]}">No records</span>
                        </div>
                    </div>
                    <p>Choose a language: <field name="lang" class="oe_inline ml8"/></p>
                    <label for="body" string="SMS content"/>
                    <hr/>
                        <field name="body" readonly="1" nolabel="1" options='{"safe": True}'/>
                    <hr/>
                    <footer>
                        <button string="Discard" class="btn-secondary" special="cancel"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="sms_template_preview_action" model="ir.actions.act_window">
            <field name="name">Template Preview</field>
            <field name="res_model">sms.template.preview</field>
            <field name="binding_model_id" ref="sms.model_sms_template"/>
            <field name="type">ir.actions.act_window</field>
            <field name="view_mode">form</field>
            <field name="view_id" ref="sms_template_preview_form"/>
            <field name="target">new</field>
            <field name="context">{'default_sms_template_id':active_id}</field>
        </record>

    </data>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-

from . import sms_cancel
from . import sms_composer
from . import sms_resend
from . import sms_template_preview

```

