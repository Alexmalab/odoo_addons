# Odoo Module: sms

Category: Hidden/Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

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
    'version': '2.2',
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
        'data/ir_cron_data.xml',
        'wizard/sms_cancel_views.xml',
        'wizard/sms_composer_views.xml',
        'wizard/sms_template_preview_views.xml',
        'wizard/sms_resend_views.xml',
        'views/ir_actions_views.xml',
        'views/mail_notification_views.xml',
        'views/res_config_settings_views.xml',
        'views/res_partner_views.xml',
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
        'mail.assets_discuss_public': [
            'sms/static/src/components/*/*',
            'sms/static/src/models/*/*.js',
        ],
        'web.assets_backend': [
            'sms/static/src/js/fields_phone_widget.js',
            'sms/static/src/js/fields_sms_widget.js',
            'sms/static/src/components/*/*.js',
            'sms/static/src/models/*/*.js',
        ],
        'web.qunit_suite_tests': [
            'sms/static/tests/sms_widget_test.js',
            'sms/static/src/components/*/tests/*.js',
        ],
        'web.assets_qweb': [
            'sms/static/src/components/*/*.xml',
        ],
    },
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
    ], ondelete={'sms': 'cascade'})
    # SMS
    sms_template_id = fields.Many2one(
        'sms.template', 'SMS Template', ondelete='set null',
        domain="[('model_id', '=', model_id)]",
    )
    sms_mass_keep_log = fields.Boolean('Log as Note', default=True)

    @api.constrains('state', 'model_id')
    def _check_sms_capability(self):
        for action in self:
            if action.state == 'sms' and not action.model_id.is_mail_thread:
                raise ValidationError(_("Sending SMS can only be done on a mail.thread model"))

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
            default_composition_mode='mass',
            default_template_id=self.sms_template_id.id,
            default_mass_keep_log=self.sms_mass_keep_log,
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

    def _get_recipient_data(self, records, message_type, subtype_id, pids=None):
        if message_type == 'sms':
            if pids is None:
                sms_pids = records._sms_get_default_partners().ids
            else:
                sms_pids = pids
            res = super(Followers, self)._get_recipient_data(records, message_type, subtype_id, pids=pids)
            new_res = []
            for pid, active, pshare, notif, groups in res:
                if pid and pid in sms_pids:
                    notif = 'sms'
                new_res.append((pid, active, pshare, notif, groups))
            return new_res
        else:
            return super(Followers, self)._get_recipient_data(records, message_type, subtype_id, pids=pids)

```

## File: models\mail_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict
from operator import itemgetter

from odoo import exceptions, fields, models
from odoo.tools import groupby


class MailMessage(models.Model):
    """ Override MailMessage class in order to add a new type: SMS messages.
    Those messages comes with their own notification method, using SMS
    gateway. """
    _inherit = 'mail.message'

    message_type = fields.Selection(selection_add=[
        ('sms', 'SMS')
    ], ondelete={'sms': lambda recs: recs.write({'message_type': 'email'})})
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

    def message_format(self, format_reply=True):
        """ Override in order to retrieves data about SMS (recipient name and
            SMS status)

        TDE FIXME: clean the overall message_format thingy
        """
        message_values = super(MailMessage, self).message_format(format_reply=format_reply)
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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class MailNotification(models.Model):
    _inherit = 'mail.notification'

    notification_type = fields.Selection(selection_add=[
        ('sms', 'SMS')
    ], ondelete={'sms': 'cascade'})
    sms_id = fields.Many2one('sms.sms', string='SMS', index=True, ondelete='set null')
    sms_number = fields.Char('SMS Number')
    failure_type = fields.Selection(selection_add=[
        ('sms_number_missing', 'Missing Number'),
        ('sms_number_format', 'Wrong Number Format'),
        ('sms_credit', 'Insufficient Credit'),
        ('sms_server', 'Server Error'),
        ('sms_acc', 'Unregistered Account')
    ])

```

## File: models\mail_thread.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, models, fields
from odoo.addons.phone_validation.tools import phone_validation
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
            self._cr.execute(""" SELECT msg.res_id, COUNT(msg.res_id) FROM mail_message msg
                                 RIGHT JOIN mail_notification rel
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
            partners = partners.union(*self.mapped(fname))  # ensure ordering
        return partners

    def _sms_get_number_fields(self):
        """ This method returns the fields to use to find the number to use to
        send an SMS on a record. """
        if 'mobile' in self:
            return ['mobile']
        return []

    def _sms_get_recipients_info(self, force_field=False, partner_fallback=True):
        """" Get SMS recipient information on current record set. This method
        checks for numbers and sanitation in order to centralize computation.

        Example of use cases

          * click on a field -> number is actually forced from field, find customer
            linked to record, force its number to field or fallback on customer fields;
          * contact -> find numbers from all possible phone fields on record, find
            customer, force its number to found field number or fallback on customer fields;

        :param force_field: either give a specific field to find phone number, either
            generic heuristic is used to find one based on ``_sms_get_number_fields``;
        :param partner_fallback: if no value found in the record, check its customer
            values based on ``_sms_get_default_partners``;

        :return dict: record.id: {
            'partner': a res.partner recordset that is the customer (void or singleton)
                linked to the recipient. See ``_sms_get_default_partners``;
            'sanitized': sanitized number to use (coming from record's field or partner's
                phone fields). Set to False is number impossible to parse and format;
            'number': original number before sanitation;
            'partner_store': whether the number comes from the customer phone fields. If
                False it means number comes from the record itself, even if linked to a
                customer;
            'field_store': field in which the number has been found (generally mobile or
                phone, see ``_sms_get_number_fields``);
        } for each record in self
        """
        result = dict.fromkeys(self.ids, False)
        tocheck_fields = [force_field] if force_field else self._sms_get_number_fields()
        for record in self:
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
                    'sanitized': valid_number,
                    'number': record[fname],
                    'partner_store': False,
                    'field_store': fname,
                }
            elif all_partners and partner_fallback:
                partner = self.env['res.partner']
                for partner in all_partners:
                    for fname in self.env['res.partner']._sms_get_number_fields():
                        valid_number = phone_validation.phone_sanitize_numbers_w_record([partner[fname]], record)[partner[fname]]['sanitized']
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

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, *args, body='', message_type='notification', **kwargs):
        # When posting an 'SMS' `message_type`, make sure that the body is used as-is in the sms,
        # and reformat the message body for the notification (mainly making URLs clickable).
        if message_type == 'sms':
            kwargs['sms_content'] = body
            body = sms_content_to_rendered_html(body)
        return super().message_post(*args, body=body, message_type=message_type, **kwargs)

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
        }
        if body and not template:
            composer_context['default_body'] = body
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

    def _notify_thread(self, message, msg_vals=False, notify_by_email=True, **kwargs):
        recipients_data = super(MailThread, self)._notify_thread(message, msg_vals=msg_vals, notify_by_email=notify_by_email, **kwargs)
        self._notify_record_by_sms(message, recipients_data, msg_vals=msg_vals, **kwargs)
        return recipients_data

    def _notify_record_by_sms(self, message, recipients_data, msg_vals=False,
                              sms_content=None, sms_numbers=None, sms_pid_to_number=None,
                              check_existing=False, put_in_queue=False, **kwargs):
        """ Notification method: by SMS.

        :param message: mail.message record to notify;
        :param recipients_data: see ``_notify_thread``;
        :param msg_vals: see ``_notify_thread``;

        :param sms_content: plaintext version of body, mainly to avoid
          conversion glitches by splitting html and plain text content formatting
          (e.g.: links, styling.).
          If not given, `msg_vals`'s `body` is used and converted from html to plaintext;
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
        body = sms_content or html2plaintext(msg_vals['body'] if msg_vals and msg_vals.get('body') else message.body)
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
            ]
            sms_create_vals += [dict(
                sms_base_vals,
                partner_id=False,
                number=n,
                state='outgoing' if n else 'error',
                failure_type='' if n else 'sms_number_missing',
            ) for n in tocreate_numbers]

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
                            'sms_id': sms.id,
                            'sms_number': sms.number,
                        })

        if sms_all and not put_in_queue:
            sms_all.filtered(lambda sms: sms.state == 'outgoing').send(auto_commit=False, raise_exception=False)

        return True

```

## File: models\mail_thread_phone.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class PhoneMixin(models.AbstractModel):
    _inherit = 'mail.thread.phone'

    def _sms_get_number_fields(self):
        """ Add fields coming from mail.thread.phone implementation. """
        phone_fields = self._phone_get_number_fields()
        sms_fields = super(PhoneMixin, self)._sms_get_number_fields()
        for fname in (f for f in sms_fields if f not in phone_fields):
            phone_fields.append(fname)
        return phone_fields

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ResPartner(models.Model):
    _name = 'res.partner'
    _inherit = ['mail.thread.phone', 'res.partner']

    def _sms_get_default_partners(self):
        """ Override of mail.thread method.
            SMS recipients on partners are the partners themselves.
        """
        return self

    def _phone_get_number_fields(self):
        """ This method returns the fields to use to find the number to use to
        send an SMS on a record. """
        return ['mobile', 'phone']

```

## File: models\sms_api.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, models
from odoo.addons.iap.tools import iap_tools

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
        return iap_tools.iap_jsonrpc(endpoint + local_endpoint, params=params)

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
        return self._contact_iap('/iap/sms/2/send', params)

    @api.model
    def _get_sms_api_error_messages(self):
        """ Returns a dict containing the error message to display for every known error 'state'
        resulting from the '_send_sms_batch' method.
        We prefer a dict instead of a message-per-error-state based method so we only call
        the 'get_credits_url' once, to avoid extra RPC calls. """

        buy_credits_url = self.sudo().env['iap.account'].get_credits_url(service_name='sms')
        buy_credits = '<a href="%s" target="_blank">%s</a>' % (
            buy_credits_url,
            _('Buy credits.')
        )
        return {
            'unregistered': _("You don't have an eligible IAP account."),
            'insufficient_credit': ' '.join([_('You don\'t have enough credits on your IAP account.'), buy_credits]),
            'wrong_number_format': _("The number you're trying to reach is not correctly formatted."),
        }

```

## File: models\sms_sms.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import threading

from odoo import api, fields, models, tools, _

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
        'server_error': 'sms_server',
        'unregistered': 'sms_acc'
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
    failure_type = fields.Selection([
        ('sms_number_missing', 'Missing Number'),
        ('sms_number_format', 'Wrong Number Format'),
        ('sms_credit', 'Insufficient Credit'),
        ('sms_server', 'Server Error'),
        ('sms_acc', 'Unregistered Account'),
        # mass mode specific codes
        ('sms_blacklist', 'Blacklisted'),
        ('sms_duplicate', 'Duplicate'),
        ('sms_optout', 'Opted Out'),
    ], copy=False)

    def action_set_canceled(self):
        self.state = 'canceled'
        notifications = self.env['mail.notification'].sudo().search([
            ('sms_id', 'in', self.ids),
            # sent is sent -> cannot reset
            ('notification_status', 'not in', ['canceled', 'sent']),
        ])
        if notifications:
            notifications.write({'notification_status': 'canceled'})
            if not self._context.get('sms_skip_msg_notification', False):
                notifications.mail_message_id._notify_message_notification_update()

    def action_set_error(self, failure_type):
        self.state = 'error'
        self.failure_type = failure_type
        notifications = self.env['mail.notification'].sudo().search([
            ('sms_id', 'in', self.ids),
            # sent can be set to error due to IAP feedback
            ('notification_status', '!=', 'exception'),
        ])
        if notifications:
            notifications.write({'notification_status': 'exception', 'failure_type': failure_type})
            if not self._context.get('sms_skip_msg_notification', False):
                notifications.mail_message_id._notify_message_notification_update()

    def action_set_outgoing(self):
        self.state = 'outgoing'
        notifications = self.env['mail.notification'].sudo().search([
            ('sms_id', 'in', self.ids),
            # sent is sent -> cannot reset
            ('notification_status', 'not in', ['ready', 'sent']),
        ])
        if notifications:
            notifications.write({'notification_status': 'ready', 'failure_type': False})
            if not self._context.get('sms_skip_msg_notification', False):
                notifications.mail_message_id._notify_message_notification_update()

    def send(self, unlink_failed=False, unlink_sent=True, auto_commit=False, raise_exception=False):
        """ Main API method to send SMS.

          :param unlink_failed: unlink failed SMS after IAP feedback;
          :param unlink_sent: unlink sent SMS after IAP feedback;
          :param auto_commit: commit after each batch of SMS;
          :param raise_exception: raise if there is an issue contacting IAP;
        """
        self = self.filtered(lambda sms: sms.state == 'outgoing')
        for batch_ids in self._split_batch():
            self.browse(batch_ids)._send(unlink_failed=unlink_failed, unlink_sent=unlink_sent, raise_exception=raise_exception)
            # auto-commit if asked except in testing mode
            if auto_commit is True and not getattr(threading.current_thread(), 'testing', False):
                self._cr.commit()

    def resend_failed(self):
        sms_to_send = self.filtered(lambda sms: sms.state == 'error')
        sms_to_send.state = 'outgoing'
        notification_title = _('Warning')
        notification_type = 'danger'

        if sms_to_send:
            sms_to_send.send()
            success_sms = len(sms_to_send) - len(sms_to_send.exists())
            if success_sms > 0:
                notification_title = _('Success')
                notification_type = 'success'
                notification_message = _('%s out of the %s selected SMS Text Messages have successfully been resent.', success_sms, len(self))
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
            self._postprocess_iap_sent_sms(
                [{'res_id': sms.id, 'state': 'server_error'} for sms in self],
                unlink_failed=unlink_failed, unlink_sent=unlink_sent)
        else:
            _logger.info('Send batch %s SMS: %s: gave %s', len(self.ids), self.ids, iap_results)
            self._postprocess_iap_sent_sms(iap_results, unlink_failed=unlink_failed, unlink_sent=unlink_sent)

    def _postprocess_iap_sent_sms(self, iap_results, failure_reason=None, unlink_failed=False, unlink_sent=True):
        todelete_sms_ids = []
        if unlink_failed:
            todelete_sms_ids += [item['res_id'] for item in iap_results if item['state'] != 'success']
        if unlink_sent:
            todelete_sms_ids += [item['res_id'] for item in iap_results if item['state'] == 'success']

        for state in self.IAP_TO_SMS_STATE.keys():
            sms_ids = [item['res_id'] for item in iap_results if item['state'] == state]
            if sms_ids:
                if state != 'success' and not unlink_failed:
                    self.env['sms.sms'].sudo().browse(sms_ids).write({
                        'state': 'error',
                        'failure_type': self.IAP_TO_SMS_STATE[state],
                    })
                if state == 'success' and not unlink_sent:
                    self.env['sms.sms'].sudo().browse(sms_ids).write({
                        'state': 'sent',
                        'failure_type': False,
                    })
                notifications = self.env['mail.notification'].sudo().search([
                    ('notification_type', '=', 'sms'),
                    ('sms_id', 'in', sms_ids),
                    ('notification_status', 'not in', ('sent', 'canceled')),
                ])
                if notifications:
                    notifications.write({
                        'notification_status': 'sent' if state == 'success' else 'exception',
                        'failure_type': self.IAP_TO_SMS_STATE[state] if state != 'success' else False,
                        'failure_reason': failure_reason if failure_reason else False,
                    })
        self.mail_message_id._notify_message_notification_update()

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
    _inherit = ['mail.render.mixin']
    _description = 'SMS Templates'

    _unrestricted_rendering = True

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

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        default = dict(default or {},
                       name=_("%s (copy)", self.name))
        return super(SMSTemplate, self).copy(default=default)

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
access_sms_cancel,access.sms.cancel,model_sms_cancel,base.group_user,1,1,1,0
access_sms_composer,access.sms.composer,model_sms_composer,base.group_user,1,1,1,0
access_sms_resend_recipient,access.sms.resend.recipient,model_sms_resend_recipient,base.group_user,1,1,1,0
access_sms_resend,access.sms.resend,model_sms_resend,base.group_user,1,1,1,0
access_sms_template_preview,access.sms.template.preview,model_sms_template_preview,base.group_user,1,1,1,0

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
<svg id="Layer_1" data-name="Layer 1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 70 70">
  <defs>
    <mask id="mask" x="0" y="0" width="70" height="70" maskUnits="userSpaceOnUse">
      <g id="b">
        <path id="a" d="M4,0H65c4,0,5,1,5,5V65c0,4-1,5-5,5H4c-3,0-4-1-4-5V5C0,1,1,0,4,0Z" fill="#fff" fill-rule="evenodd"/>
      </g>
    </mask>
    <linearGradient id="linear-gradient" x1="-1172.36" y1="477.94" x2="-1173.36" y2="476.94" gradientTransform="matrix(70, 0, 0, -70, 82134.99, 33455.73)" gradientUnits="userSpaceOnUse">
      <stop offset="0" stop-color="#7cc098"/>
      <stop offset="1" stop-color="#5f8a71"/>
    </linearGradient>
  </defs>
  <g mask="url(#mask)">
    <g>
      <path d="M0,0H70V70H0Z" fill-rule="evenodd" fill="url(#linear-gradient)"/>
      <path d="M4,1H65c2.67,0,4.33.67,5,2V0H0V3C.67,1.67,2,1,4,1Z" fill="#fff" fill-opacity="0.38" fill-rule="evenodd"/>
      <path d="M4,69H65c2.67,0,4.33-1,5-3v4H0V66A3.92,3.92,0,0,0,4,69Z" fill-opacity="0.38" fill-rule="evenodd"/>
      <path d="M4,69a3.66,3.66,0,0,1-4-4V34.17L20.65,13.52l24.78-.66,2,3.78v9.44l-6.11,6.11,2.41,2.11-1.09,1.09,1.31,2L47.32,34l1.52,0,3.8-3.83,1.87,12-7.08,7.1-.56,7.6L34.81,69Z" fill-opacity="0.15" fill-rule="evenodd"/>
      <g>
        <g opacity="0.4" style="isolation: isolate">
          <path d="M21.59,39.06,23,39a2.29,2.29,0,0,0,.49,1,1.5,1.5,0,0,0,1.05.33,1.87,1.87,0,0,0,1-.27.87.87,0,0,0,.33-.66A.64.64,0,0,0,25.7,39a1.88,1.88,0,0,0-.49-.33l-1.15-.33a4.49,4.49,0,0,1-1.59-.71,1.82,1.82,0,0,1-.66-1.42,1.52,1.52,0,0,1,.33-1,1.83,1.83,0,0,1,.88-.71,3.31,3.31,0,0,1,1.37-.22,3.2,3.2,0,0,1,2,.6,2.05,2.05,0,0,1,.72,1.54l-1.43,0a1.3,1.3,0,0,0-.38-.77,1.44,1.44,0,0,0-.88-.22,1.73,1.73,0,0,0-1,.28.51.51,0,0,0-.22.44.52.52,0,0,0,.22.44,3.47,3.47,0,0,0,1.32.49,7.23,7.23,0,0,1,1.53.49,2,2,0,0,1,.77.71,2.34,2.34,0,0,1,.27,1.16A2.3,2.3,0,0,1,27,40.6a2.08,2.08,0,0,1-.94.76,4.1,4.1,0,0,1-1.53.28A3.3,3.3,0,0,1,22.47,41,3.1,3.1,0,0,1,21.59,39.06Z"/>
          <path d="M28.39,41.36V34.29h2.13l1.26,4.83L33,34.29h2.14v7.07H33.87V35.83L32.5,41.36H31.13l-1.37-5.53v5.53Z"/>
          <path d="M36.17,39.06,37.54,39A2.29,2.29,0,0,0,38,40a1.56,1.56,0,0,0,1,.28,1.77,1.77,0,0,0,1-.28.83.83,0,0,0,.33-.65.61.61,0,0,0-.16-.44,2.07,2.07,0,0,0-.49-.33c-.17-.06-.55-.17-1.16-.33A3.37,3.37,0,0,1,37,37.53a1.88,1.88,0,0,1-.65-1.43,1.51,1.51,0,0,1,.33-1,1.93,1.93,0,0,1,.87-.72A3.52,3.52,0,0,1,39,34.18a3.15,3.15,0,0,1,2,.61,2,2,0,0,1,.71,1.53l-1.42.06a1.25,1.25,0,0,0-.39-.77,1.33,1.33,0,0,0-.87-.22,1.81,1.81,0,0,0-1,.27.52.52,0,0,0-.22.44.51.51,0,0,0,.22.44,3.09,3.09,0,0,0,1.32.49,7.13,7.13,0,0,1,1.53.5,2,2,0,0,1,.77.71,2.44,2.44,0,0,1,.27,1.15,2.29,2.29,0,0,1-.33,1.15,1.8,1.8,0,0,1-.93.77,4.36,4.36,0,0,1-1.53.27A3.42,3.42,0,0,1,37,41,3,3,0,0,1,36.17,39.06Z"/>
          <path d="M57.14,37.2l-4.55-4.93a1.37,1.37,0,0,0-2.36,1.06v2.74H45.3v4h4.93v3a1.37,1.37,0,0,0,2.36,1.06l4.55-4.93A1.54,1.54,0,0,0,57.14,37.2Z"/>
          <path d="M18.09,36.07H10a1.78,1.78,0,0,0-1.69,1.34,2,2,0,0,0,1.62,2.66h8.18Z"/>
          <path d="M43.41,14.88H20.11a2,2,0,0,0-2,2V28.09h2.3V21.28a.27.27,0,0,1,.3-.3h22.1a.27.27,0,0,1,.3.3v6.81h2.3V16.88A2,2,0,0,0,43.41,14.88Zm-8.5,4.7h-6.1a.47.47,0,0,1-.5-.5c0-.3.1-.5.4-.5h6.1a.47.47,0,0,1,.5.5A.46.46,0,0,1,34.91,19.58Z"/>
          <path d="M43.11,47.38v6.9a.27.27,0,0,1-.3.3H20.71a.27.27,0,0,1-.3-.3v-6.9h-2.3v10.2a2,2,0,0,0,2,2h23.3a2,2,0,0,0,2-2V47.38Zm-11.3,11.1a1.5,1.5,0,1,1,1.5-1.5A1.47,1.47,0,0,1,31.81,58.48Z"/>
        </g>
        <g style="isolation: isolate">
          <path d="M23.61,37,25,36.93a2.3,2.3,0,0,0,.5,1,1.48,1.48,0,0,0,1,.33,1.85,1.85,0,0,0,1-.27.87.87,0,0,0,.33-.66.62.62,0,0,0-.17-.44,1.88,1.88,0,0,0-.49-.33l-1.15-.33a5,5,0,0,1-1.59-.71,1.84,1.84,0,0,1-.66-1.43,1.51,1.51,0,0,1,.33-1,1.87,1.87,0,0,1,.88-.71,3.31,3.31,0,0,1,1.37-.22,3.23,3.23,0,0,1,2,.6,2.06,2.06,0,0,1,.71,1.53l-1.43.06a1.27,1.27,0,0,0-.38-.77,1.42,1.42,0,0,0-.88-.22,1.7,1.7,0,0,0-1,.28.48.48,0,0,0-.22.43.48.48,0,0,0,.22.44,3.13,3.13,0,0,0,1.31.5,6.81,6.81,0,0,1,1.54.49,1.87,1.87,0,0,1,.76.71,2.33,2.33,0,0,1,.28,1.15A2.27,2.27,0,0,1,29,38.57a2.13,2.13,0,0,1-.93.77,4.37,4.37,0,0,1-1.54.27,3.32,3.32,0,0,1-2.08-.6A3.15,3.15,0,0,1,23.61,37Z" fill="#fff"/>
          <path d="M30.41,39.34V32.27h2.14l1.26,4.82,1.26-4.82H37.2v7.07H35.89V33.81l-1.37,5.53H33.15l-1.37-5.53v5.53Z" fill="#fff"/>
          <path d="M38.19,37l1.37-.11a2.4,2.4,0,0,0,.49,1,1.57,1.57,0,0,0,1.05.27,1.87,1.87,0,0,0,1-.27.85.85,0,0,0,.33-.66.64.64,0,0,0-.17-.44,1.66,1.66,0,0,0-.49-.32c-.17-.06-.55-.17-1.15-.33a3.27,3.27,0,0,1-1.59-.72,1.82,1.82,0,0,1-.66-1.42,1.55,1.55,0,0,1,.33-1,1.83,1.83,0,0,1,.88-.71A3.48,3.48,0,0,1,41,32.16a3.2,3.2,0,0,1,2,.6,2.07,2.07,0,0,1,.72,1.54l-1.43.05a1.24,1.24,0,0,0-.38-.76,1.37,1.37,0,0,0-.88-.22,1.81,1.81,0,0,0-1,.27.5.5,0,0,0-.21.44.51.51,0,0,0,.21.44,3.3,3.3,0,0,0,1.32.49,7.64,7.64,0,0,1,1.53.49,1.92,1.92,0,0,1,.77.72,2.32,2.32,0,0,1-.05,2.3,1.89,1.89,0,0,1-.93.77,4.42,4.42,0,0,1-1.54.27,3.36,3.36,0,0,1-2.08-.6A3,3,0,0,1,38.19,37Z" fill="#fff"/>
          <path d="M59.16,35.18l-4.54-4.93a1.37,1.37,0,0,0-2.36,1.06V34H47.32v4h4.94v3a1.37,1.37,0,0,0,2.36,1.06l4.54-4.93A1.54,1.54,0,0,0,59.16,35.18Z" fill="#fff"/>
          <path d="M20.11,34H12a1.79,1.79,0,0,0-1.7,1.35A2,2,0,0,0,11.94,38h8.17Z" fill="#fff"/>
          <path d="M45.43,12.86H22.13a2,2,0,0,0-2,2V26.07h2.3V19.26a.27.27,0,0,1,.3-.3h22.1a.27.27,0,0,1,.3.3v6.81h2.3V14.86A2,2,0,0,0,45.43,12.86Zm-8.5,4.7h-6.1a.47.47,0,0,1-.5-.5c0-.3.1-.5.4-.5h6.1a.47.47,0,0,1,.5.5A.46.46,0,0,1,36.93,17.56Z" fill="#fff"/>
          <path d="M45.13,45.35v6.91a.27.27,0,0,1-.3.3H22.73a.27.27,0,0,1-.3-.3V45.35h-2.3V55.56a2,2,0,0,0,2,2h23.3a2,2,0,0,0,2-2V45.35ZM33.83,56.46a1.5,1.5,0,1,1,1.5-1.5A1.47,1.47,0,0,1,33.83,56.46Z" fill="#fff"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\img\sms_failure.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="512" height="512"><defs><path id="a" d="M91.974 123.535v-17.07c0-.839-.283-1.542-.851-2.111-.568-.57-1.24-.854-2.017-.854H71.894c-.777 0-1.449.285-2.017.854-.568.569-.851 1.272-.851 2.11v17.071c0 .839.283 1.542.851 2.111.568.57 1.24.854 2.017.854h17.212c.777 0 1.449-.285 2.017-.854.568-.569.851-1.272.851-2.11zm-.179-33.601l1.614-41.239c0-.718-.3-1.287-.897-1.707-.777-.659-1.494-.988-2.151-.988H70.639c-.657 0-1.374.33-2.151.988-.598.42-.897 1.048-.897 1.887l1.524 41.059c0 .599.3 1.093.897 1.482.597.39 1.314.584 2.151.584h16.584c.837 0 1.54-.195 2.107-.584.568-.39.881-.883.941-1.482zM90.54 6.02l68.846 126.5c2.092 3.773 2.032 7.546-.179 11.32a11.276 11.276 0 0 1-4.168 4.133 11.192 11.192 0 0 1-5.693 1.527H11.654c-2.032 0-3.93-.51-5.693-1.527a11.276 11.276 0 0 1-4.168-4.133c-2.211-3.774-2.271-7.547-.18-11.32L70.46 6.02a11.462 11.462 0 0 1 4.213-4.403A11.105 11.105 0 0 1 80.5 0c2.092 0 4.034.54 5.827 1.617A11.462 11.462 0 0 1 90.54 6.02z"/><path id="c" d="M91.974 123.535v-17.07c0-.839-.283-1.542-.851-2.111-.568-.57-1.24-.854-2.017-.854H71.894c-.777 0-1.449.285-2.017.854-.568.569-.851 1.272-.851 2.11v17.071c0 .839.283 1.542.851 2.111.568.57 1.24.854 2.017.854h17.212c.777 0 1.449-.285 2.017-.854.568-.569.851-1.272.851-2.11zm-.179-33.601l1.614-41.239c0-.718-.3-1.287-.897-1.707-.777-.659-1.494-.988-2.151-.988H70.639c-.657 0-1.374.33-2.151.988-.598.42-.897 1.048-.897 1.887l1.524 41.059c0 .599.3 1.093.897 1.482.597.39 1.314.584 2.151.584h16.584c.837 0 1.54-.195 2.107-.584.568-.39.881-.883.941-1.482zM90.54 6.02l68.846 126.5c2.092 3.773 2.032 7.546-.179 11.32a11.276 11.276 0 0 1-4.168 4.133 11.192 11.192 0 0 1-5.693 1.527H11.654c-2.032 0-3.93-.51-5.693-1.527a11.276 11.276 0 0 1-4.168-4.133c-2.211-3.774-2.271-7.547-.18-11.32L70.46 6.02a11.462 11.462 0 0 1 4.213-4.403A11.105 11.105 0 0 1 80.5 0c2.092 0 4.034.54 5.827 1.617A11.462 11.462 0 0 1 90.54 6.02z"/></defs><g fill="none" fill-rule="evenodd"><circle cx="256" cy="253" r="256" fill="#FDA20C"/><path fill="#000" fill-opacity=".3" fill-rule="nonzero" d="M361 98.292C361 90.982 353.978 85 345.396 85H163.604C155.022 85 148 90.981 148 98.292v296.416c0 7.31 7.022 13.292 15.604 13.292h181.792c8.582 0 15.604-5.981 15.604-13.292v-64.636c-5.462 3.323-11.703 6.646-17.945 9.305v33.399c0 1.329-.78 1.994-2.34 1.994h-172.43c-1.56 0-2.34-.665-2.34-1.994V126.87c0-1.329.78-1.993 2.34-1.993h172.43c1.56 0 2.34.664 2.34 1.993v63.113c3.901 0 8.582-.664 12.483-.664H361V98.292zM254.5 380c5.067 0 9.5 4.433 9.5 9.5s-4.433 9.5-9.5 9.5-9.5-4.433-9.5-9.5 4.433-9.5 9.5-9.5zm24.577-266.907h-47.593c-2.341 0-3.902-1.56-3.902-3.9s1.56-3.899 3.902-3.899h47.593c2.34 0 3.901 1.56 3.901 3.9 0 2.339-2.34 3.899-3.901 3.899z"/><path fill="#FFF" fill-rule="nonzero" d="M355.538 321.966c-3.9 0-8.582 0-12.483-.665v45.438c0 1.33-.78 1.996-2.34 1.996h-172.43c-1.56 0-2.34-.665-2.34-1.996V120.58c0-1.33.78-1.996 2.34-1.996h172.43c1.56 0 2.34.665 2.34 1.996v63.81c3.901 0 8.582-.666 12.483-.666H361V91.306C361 83.988 353.978 78 345.396 78H163.604C155.022 78 148 83.988 148 91.306v297.388c0 7.318 7.022 13.306 15.604 13.306h181.792c8.582 0 15.604-5.988 15.604-13.306v-66.728h-5.462zM230.703 98.871h47.594c2.34 0 3.9 1.56 3.9 3.901 0 2.34-1.56 3.902-3.12 3.902h-47.593c-2.341 0-3.902-1.561-3.902-3.902 0-2.34.78-3.901 3.121-3.901zM254.5 394c-5.067 0-9.5-4.433-9.5-9.5s4.433-9.5 9.5-9.5 9.5 4.433 9.5 9.5c0 5.7-4.433 9.5-9.5 9.5z"/><g opacity=".437" transform="translate(217 160)"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g fill="#2F3136" mask="url(#b)"><path d="M0 0H161V161H0z"/></g></g><g transform="translate(217 149)"><mask id="d" fill="#fff"><use xlink:href="#c"/></mask><g fill="#FFF" mask="url(#d)"><path d="M0 0H161V161H0z"/></g></g></g></svg>
```

## File: static\src\components\message\message.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-inherit="mail.Message" t-inherit-mode="extension">
        <xpath expr="//*[@name='failureIcon']" position="replace">
            <t t-if="messageView.message.message_type === 'sms'">
                <i class="o_Message_notificationIcon fa fa-mobile"/> SMS
            </t>
            <t t-else="">$0</t>
        </xpath>

        <xpath expr="//*[@name='notificationIcon']" position="replace">
            <t t-if="messageView.message.message_type === 'sms'">
                <i class="o_Message_notificationIcon fa fa-mobile"/> SMS
            </t>
            <t t-else="">$0</t>
        </xpath>
    </t>

</templates>

```

## File: static\src\components\notification_group\notification_group.js

```javascript
/** @odoo-module **/

import { NotificationGroup } from '@mail/components/notification_group/notification_group';

import { patch } from 'web.utils';

patch(NotificationGroup.prototype, 'sms/static/src/components/notification_group/notification_group.js', {

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    image() {
        if (this.group.notification_type === 'sms') {
            return '/sms/static/img/sms_failure.svg';
        }
        return this._super(...arguments);
    },
});

```

## File: static\src\components\notification_group\notification_group.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-inherit="mail.NotificationGroup" t-inherit-mode="extension">
        <xpath expr="//*[hasclass('o_NotificationGroup_inlineText')]" position="inside">
            <t t-if="group.notification_type === 'sms'">
                An error occurred when sending an SMS.
            </t>
        </xpath>
    </t>

</templates>

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
 * Override of FieldPhone to add a button calling SMS composer if option activated (default)
 */

var Phone = basic_fields.FieldPhone;
Phone.include({
    /**
     * By default, enable_sms is activated
     *
     * @override
     */
    init() {
        this._super.apply(this, arguments);
        this.enableSMS = 'enable_sms' in this.attrs.options ? this.attrs.options.enable_sms : true;
        // reinject in nodeOptions (and thus in this.attrs) to signal the property
        this.attrs.options.enable_sms = this.enableSMS;
    },
    /**
     * When the send SMS button is displayed, $el becomes a div wrapping
     * the original links.
     * This method makes sure we always focus the phone number
     *
     * @override
     */
    getFocusableElement() {
        if (this.enableSMS && this.mode === 'readonly') {
            return this.$el.filter('.' + this.className).find('a');
        }
        return this._super.apply(this, arguments);
    },
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
        ev.stopPropagation();

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
        if (this.enableSMS && this.value) {
            var $composerButton = $('<a>', {
                title: _t('Send SMS Text Message'),
                href: '',
                class: 'ml-3 d-inline-flex align-items-center o_field_phone_sms',
                html: $('<small>', {class: 'font-weight-bold ml-1', html: 'SMS'}),
            });
            $composerButton.prepend($('<i>', {class: 'fa fa-mobile'}));
            $composerButton.on('click', this._onClickSMS.bind(this));
            this.$el = this.$el.add($composerButton);
        }
        return def;
    },
});

return Phone;

});

```

## File: static\src\js\fields_sms_widget.js

```javascript
/** @odoo-module **/

import { _t } from 'web.core';
import fieldRegistry from 'web.field_registry';
import FieldTextEmojis from '@mail/js/field_text_emojis';
/**
 * SmsWidget is a widget to display a textarea (the body) and a text representing
 * the number of SMS and the number of characters. This text is computed every
 * time the user changes the body.
 */
var SmsWidget = FieldTextEmojis.extend({
    className: 'o_field_text',
    enableEmojis: false,
    /**
     * @constructor
     */
    init: function () {
        this._super.apply(this, arguments);
        this.nbrChar = 0;
        this.nbrSMS = 0;
        this.encoding = 'GSM7';
        this.enableEmojis = !!this.nodeOptions.enable_emojis;
    },

    /**
     * @override
     *"This will add the emoji dropdown to a target field (controlled by the "enableEmojis" attribute)
     */
    on_attach_callback: function () {
        if (this.enableEmojis) {
            this._super.apply(this, arguments);
        }
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
        const $sms_container = $('<div class="o_sms_container"/>');
        if (!document.contains($sms_container[0])) {
            this.$el = this.$el.add($sms_container);
        } else {
            $sms_container.empty();
        }
        $sms_container.append(this._renderSMSInfo());
        $sms_container.append(this._renderIAPButton());

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
        const content = this._getValueForSmsCounts();
        this.encoding = this._extractEncoding(content);
        this.nbrChar = content.length;
        this.nbrChar += (content.match(/\n/g) || []).length;
        this.nbrSMS = this._countSMS(this.nbrChar, this.encoding);
    },

    _getValueForSmsCounts: function () {
        return this._getValue();
    },

    _getNbrCharExplanationTemplate: function () {
        return _t("%s characters, fits in %s SMS (%s) ");
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
        const string = _.str.sprintf(this._getNbrCharExplanationTemplate(), this.nbrChar, this.nbrSMS, this.encoding);
        const $span = $('<span>', {
            'class': 'text-muted o_sms_count',
        });
        $span.text(string);
        return $span;
    },

    /**
     * Update widget SMS information with re-computed info about length, ...
     * @private
     */
    _updateSMSInfo: function () {
        this._compute();
        const string = _.str.sprintf(this._getNbrCharExplanationTemplate(), this.nbrChar, this.nbrSMS, this.encoding);
        this.$('.o_sms_count').text(string);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @override
     * @private
     */
    _onBlur: function () {
        var content = this._getValue();
        if( !content.trim().length && content.length > 0) {
            this.displayNotification({ title: _t("Your SMS Text Message must include at least one non-whitespace character"), type: 'danger' });
            this.$input.val(content.trim());
            this._updateSMSInfo();
        }
    },

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

export default SmsWidget;

```

## File: static\src\models\message\message.js

```javascript
/** @odoo-module **/

import {
    registerInstancePatchModel,
} from '@mail/model/model_core';

registerInstancePatchModel('mail.message', 'sms/static/src/models/message/message.js', {

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    openResendAction() {
        if (this.message_type === 'sms') {
            this.env.bus.trigger('do-action', {
                action: 'sms.sms_resend_action',
                options: {
                    additional_context: {
                        default_mail_message_id: this.id,
                    },
                },
            });
        } else {
            this._super(...arguments);
        }
    },
});

```

## File: static\src\models\notification_group\notification_group.js

```javascript
/** @odoo-module **/

import {
    registerInstancePatchModel,
} from '@mail/model/model_core';

registerInstancePatchModel('mail.notification_group', 'sms/static/src/models/notification_group/notification_group.js', {

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    openCancelAction() {
        if (this.notification_type !== 'sms') {
            return this._super(...arguments);
        }
        this.env.bus.trigger('do-action', {
            action: 'sms.sms_cancel_action',
            options: {
                additional_context: {
                    default_model: this.res_model,
                    unread_counter: this.notifications.length,
                },
            },
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _openDocuments() {
        if (this.notification_type !== 'sms') {
            return this._super(...arguments);
        }
        this.env.bus.trigger('do-action', {
            action: {
                name: this.env._t("SMS Failures"),
                type: 'ir.actions.act_window',
                view_mode: 'kanban,list,form',
                views: [[false, 'kanban'], [false, 'list'], [false, 'form']],
                target: 'current',
                res_model: this.res_model,
                domain: [['message_has_sms_error', '=', true]],
            },
        });
        if (this.messaging.device.isMobile) {
            // messaging menu has a higher z-index than views so it must
            // be closed to ensure the visibility of the view
            this.messaging.messagingMenu.close();
        }
    },
});

```

## File: tools\sms_tools.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

from odoo.tools import TEXT_URL_REGEX, create_link, html_escape


def sms_content_to_rendered_html(text):
    """Transforms plaintext into html making urls clickable and preserving newlines"""
    urls = re.findall(TEXT_URL_REGEX, text)
    escaped_text = str(html_escape(text))
    for url in urls:
        escaped_text = escaped_text.replace(url, create_link(url, url))
    return re.sub(r'\r?\n|\r', '<br/>', escaped_text)

```

## File: tools\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sms_tools

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
                    context="{'default_model_id': model_id}"
                    attrs="{'invisible': [('state', '!=', 'sms')],
                            'required': [('state', '=', 'sms')]}"/>
                <field name="sms_mass_keep_log"
                    attrs="{'invisible': [('state', '!=', 'sms')]}"/>
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
        <field name="name">mail.notification.view.tree</field>
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
            <div id="sms_settings" position="inside">
                <widget name="iap_buy_more_credits" service_name="sms" hide_service="1"/>
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
                <field name="phone_blacklisted" invisible="1"/>
                <field name="mobile_blacklisted" invisible="1"/>
                <label for="phone" class="oe_inline"/>
                <div class="o_row o_row_readonly">
                    <button name="phone_action_blacklist_remove" class="fa fa-ban text-danger"
                        title="This phone number is blacklisted for SMS Marketing. Click to unblacklist."
                        type="object" context="{'default_phone': phone}" groups="base.group_user"
                        attrs="{'invisible': [('phone_blacklisted', '=', False)]}"/>
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
                        attrs="{'invisible': [('mobile_blacklisted', '=', False)]}"/>
                    <field name="mobile" widget="phone"/>
                </div>
            </xpath>
        </field>
    </record>

    <!-- Add action entry in the Action Menu for Partners -->
    <record id="res_partner_act_window_sms_composer_multi" model="ir.actions.act_window">
        <field name="name">Send SMS Text Message</field>
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
        <field name="name">Send SMS Text Message</field>
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
                    <button name="send" string="Send Now" type="object" states='outgoing' class="oe_highlight"/>
                    <button name="action_set_outgoing" string="Retry" type="object" states='error,canceled'/>
                    <button name="action_set_canceled" string="Cancel" type="object" states='error,outgoing'/>
                    <field name="state" widget="statusbar"/>
                </header>
                <sheet>
                    <group>
                        <group>
                            <field name="partner_id" string="Contact"/>
                            <field name="mail_message_id" readonly="1" attrs="{'invisible': [('mail_message_id', '=', False)]}"/>
                        </group>
                        <group>
                            <field name="number" required="1"/>
                            <field name="failure_type" readonly="1" attrs="{'invisible': [('failure_type', '=', False)]}"/>
                        </group>
                    </group>
                    <group>
                        <field name="body" widget="sms_widget" string="Message" required="1"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="sms_sms_view_tree" model="ir.ui.view">
        <field name="name">sms.sms.view.tree</field>
        <field name="model">sms.sms</field>
        <field name="arch" type="xml">
            <tree string="SMS Templates" decoration-muted="state == 'canceled'" decoration-info="state == 'outgoing'" decoration-danger="state == 'error'">
                <field name="number"/>
                <field name="partner_id"/>
                <field name="state"/>
                <field name="failure_type"/>
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
                        <page string="Dynamic Placeholder Generator" name="dynamic_placeholder_generator" groups="base.group_no_one">
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
            wizard.help_message = _("Are you sure you want to discard %s SMS delivery failures? You won't be able to re-send these SMS later!") % (wizard._context.get('unread_counter'))

    def action_cancel(self):
        # TDE CHECK: delete pending SMS
        author_id = self.env.user.partner_id.id
        for wizard in self:
            self._cr.execute("""
SELECT notif.id, msg.id
FROM mail_notification notif
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
                self.env['mail.message'].browse(message_ids)._notify_message_notification_update()
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
                    <button string="Discard delivery failures" name="action_cancel" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z" />
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
from odoo.addons.sms.tools.sms_tools import sms_content_to_rendered_html
from odoo.exceptions import UserError


class SendSMS(models.TransientModel):
    _name = 'sms.composer'
    _description = 'Send SMS Wizard'

    @api.model
    def default_get(self, fields):
        result = super(SendSMS, self).default_get(fields)

        result['res_model'] = result.get('res_model') or self.env.context.get('active_model')

        if not result.get('active_domain'):
            result['active_domain'] = repr(self.env.context.get('active_domain', []))
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
        compute='_compute_composition_mode', readonly=False, required=True, store=True)
    res_model = fields.Char('Document Model Name')
    res_id = fields.Integer('Document ID')
    res_ids = fields.Char('Document IDs')
    res_ids_count = fields.Integer(
        'Visible records count', compute='_compute_recipients_count', compute_sudo=False,
        help='Number of recipients that will receive the SMS if sent in mass mode, without applying the Active Domain value')
    use_active_domain = fields.Boolean('Use active domain')
    active_domain = fields.Text('Active domain', readonly=True)
    active_domain_count = fields.Integer(
        'Active records count', compute='_compute_recipients_count', compute_sudo=False,
        help='Number of records found when searching with the value in Active Domain')
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
    recipient_single_description = fields.Text('Recipients (Partners)', compute='_compute_recipient_single', compute_sudo=False)
    recipient_single_number = fields.Char('Stored Recipient Number', compute='_compute_recipient_single', compute_sudo=False)
    recipient_single_number_itf = fields.Char(
        'Recipient Number', compute='_compute_recipient_single',
        readonly=False, compute_sudo=False, store=True,
        help='UX field allowing to edit the recipient number. If changed it will be stored onto the recipient.')
    recipient_single_valid = fields.Boolean("Is valid", compute='_compute_recipient_single_valid', compute_sudo=False)
    number_field_name = fields.Char('Number Field')
    numbers = fields.Char('Recipients (Numbers)')
    sanitized_numbers = fields.Char('Sanitized Number', compute='_compute_sanitized_numbers', compute_sudo=False)
    # content
    template_id = fields.Many2one('sms.template', string='Use Template', domain="[('model', '=', res_model)]")
    body = fields.Text(
        'Message', compute='_compute_body',
        readonly=False, store=True, required=True)

    @api.depends('res_ids_count', 'active_domain_count')
    @api.depends_context('sms_composition_mode')
    def _compute_composition_mode(self):
        for composer in self:
            if self.env.context.get('sms_composition_mode') == 'guess' or not composer.composition_mode:
                if composer.res_ids_count > 1 or (composer.use_active_domain and composer.active_domain_count > 1):
                    composer.composition_mode = 'mass'
                else:
                    composer.composition_mode = 'comment'

    @api.depends('res_model', 'res_id', 'res_ids', 'active_domain')
    def _compute_recipients_count(self):
        for composer in self:
            composer.res_ids_count = len(literal_eval(composer.res_ids)) if composer.res_ids else 0
            if composer.res_model:
                composer.active_domain_count = self.env[composer.res_model].search_count(literal_eval(composer.active_domain or '[]'))
            else:
                composer.active_domain_count = 0

    @api.depends('res_id', 'composition_mode')
    def _compute_comment_single_recipient(self):
        for composer in self:
            composer.comment_single_recipient = bool(composer.res_id and composer.composition_mode == 'comment')

    @api.depends('res_model', 'res_id', 'res_ids', 'use_active_domain', 'composition_mode', 'number_field_name', 'sanitized_numbers')
    def _compute_recipients(self):
        for composer in self:
            composer.recipient_valid_count = 0
            composer.recipient_invalid_count = 0

            if composer.composition_mode not in ('comment', 'mass') or not composer.res_model:
                continue

            records = composer._get_records()
            if records and isinstance(records, self.pool['mail.thread']):
                res = records._sms_get_recipients_info(force_field=composer.number_field_name, partner_fallback=not composer.comment_single_recipient)
                composer.recipient_valid_count = len([rid for rid, rvalues in res.items() if rvalues['sanitized']])
                composer.recipient_invalid_count = len([rid for rid, rvalues in res.items() if not rvalues['sanitized']])
            else:
                composer.recipient_invalid_count = 0 if (
                    composer.sanitized_numbers or (composer.composition_mode == 'mass' and composer.use_active_domain)
                ) else 1

    @api.depends('res_model', 'number_field_name')
    def _compute_recipient_single(self):
        for composer in self:
            records = composer._get_records()
            if not records or not isinstance(records, self.pool['mail.thread']) or not composer.comment_single_recipient:
                composer.recipient_single_description = False
                composer.recipient_single_number = ''
                composer.recipient_single_number_itf = ''
                continue
            records.ensure_one()
            res = records._sms_get_recipients_info(force_field=composer.number_field_name, partner_fallback=True)
            composer.recipient_single_description = res[records.id]['partner'].name or records._sms_get_default_partners().display_name
            composer.recipient_single_number = res[records.id]['number'] or ''
            if not composer.recipient_single_number_itf:
                composer.recipient_single_number_itf = res[records.id]['number'] or ''
            if not composer.number_field_name:
                composer.number_field_name = res[records.id]['field_store']

    @api.depends('recipient_single_number', 'recipient_single_number_itf')
    def _compute_recipient_single_valid(self):
        for composer in self:
            value = composer.recipient_single_number_itf or composer.recipient_single_number
            if value:
                records = composer._get_records()
                sanitized = phone_validation.phone_sanitize_numbers_w_record([value], records)[value]['sanitized']
                composer.recipient_single_valid = bool(sanitized)
            else:
                composer.recipient_single_valid = False

    @api.depends('numbers', 'res_model', 'res_id')
    def _compute_sanitized_numbers(self):
        for composer in self:
            if composer.numbers:
                record = composer._get_records() if composer.res_model and composer.res_id else self.env.user
                numbers = [number.strip() for number in composer.numbers.split(',')]
                sanitize_res = phone_validation.phone_sanitize_numbers_w_record(numbers, record)
                sanitized_numbers = [info['sanitized'] for info in sanitize_res.values() if info['sanitized']]
                invalid_numbers = [number for number, info in sanitize_res.items() if info['code']]
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
    # CRUD
    # ------------------------------------------------------------

    @api.model
    def create(self, values):
        # TDE FIXME: currently have to compute manually to avoid required issue, waiting VFE branch
        if not values.get('body') or not values.get('composition_mode'):
            values_wdef = self._add_missing_default_values(values)
            cache_composer = self.new(values_wdef)
            cache_composer._compute_body()
            cache_composer._compute_composition_mode()
            values['body'] = values.get('body') or cache_composer.body
            values['composition_mode'] = values.get('composition_mode') or cache_composer.composition_mode
        return super(SendSMS, self).create(values)

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
        self.env['sms.api']._send_sms_batch([{
            'res_id': 0,
            'number': number,
            'content': self.body,
        } for number in self.sanitized_numbers.split(',')])
        return True

    def _action_send_sms_comment_single(self, records=None):
        # If we have a recipient_single_original number, it's possible this number has been corrected in the popup
        # if invalid. As a consequence, the test cannot be based on recipient_invalid_count, which count is based
        # on the numbers in the database.
        records = records if records is not None else self._get_records()
        records.ensure_one()
        if not self.number_field_name:
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
                'partner_id': recipients['partner'].id,
                'number': sanitized if sanitized else recipients['number'],
                'state': state,
                'failure_type': failure_type,
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
        if self.use_active_domain:
            active_domain = literal_eval(self.active_domain or '[]')
            records = self.env[self.res_model].search(active_domain)
        elif self.res_ids:
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
                <sheet>
                    <group>
                        <field name="composition_mode" invisible="1"/>
                        <field name="comment_single_recipient" invisible="1"/>
                        <field name="res_id" invisible="1"/>
                        <field name="res_ids" invisible="1"/>
                        <field name="active_domain" invisible="1"/>
                        <field name="res_model" invisible="1"/>
                        <field name="mass_force_send" invisible="1"/>
                        <field name="recipient_single_valid" invisible="1"/>
                        <field name="recipient_single_number" invisible="1"/>
                        <field name="number_field_name" invisible="1"/>
                        <field name="numbers" invisible="1"/>
                        <field name="sanitized_numbers" invisible="1"/>

                        <!-- Single mode information (invalid number) -->
                        <div colspan="2" class="alert alert-danger text-center mb-3" role="alert"
                            attrs="{'invisible': ['|', ('comment_single_recipient', '=', False), ('recipient_single_valid', '=', True)]}">
                            <p class="my-0">Invalid phone number</p>
                        </div>

                        <!-- Mass mode information (res_ids versus active domain) -->
                        <div colspan="2" class="alert alert-info text-center mb-3" role="alert"
                                attrs="{'invisible': [('comment_single_recipient', '=', True)]}">
                            <p class="my-0">
                                <span attrs="{'invisible': [('use_active_domain', '=', 'True')]}">
                                    <field class="oe_inline font-weight-bold" name="res_ids_count"/> records selected.
                                </span>
                                Check <field class="oe_inline ml-2" name="use_active_domain"/> to send to all
                                <field class="oe_inline font-weight-bold ml-2" name="active_domain_count"/> records instead. <br/>
                                <field class="oe_inline font-weight-bold" name="recipient_valid_count"/> recipients are valid
                                and <field class="oe_inline font-weight-bold" name="recipient_invalid_count"/> are invalid.
                            </p>
                        </div>

                        <label for="recipient_single_description" string="Recipient"
                            class="font-weight-bold"
                            attrs="{'invisible': [('comment_single_recipient', '=', False)]}"/>
                        <div attrs="{'invisible': [('comment_single_recipient', '=', False)]}">
                            <field name="recipient_single_description" class="oe_inline" attrs="{'invisible': [('recipient_single_description', '=', False)]}"/>
                            <field name="recipient_single_number_itf" class="oe_inline" nolabel="1" options="{'onchange_on_keydown': True}" placeholder="e.g. +1 415 555 0100"/>
                        </div>
                                                
                        <field name="body" widget="sms_widget" attrs="{'invisible': ['|', ('comment_single_recipient', '=', False), ('recipient_single_valid', '=', True)]}"/>
                        <field name="body" widget="sms_widget" attrs="{'invisible': [('comment_single_recipient', '=', True), ('recipient_single_valid', '=', False)]}" default_focus="1"/>
                        <field name="mass_keep_log" invisible="1"/>
                    </group>
                </sheet>
                <footer>
                    <!-- attrs doesn't work for 'disabled'-->
                    <button string="Send SMS" type="object" class="oe_highlight" name="action_send_sms" data-hotkey="q"
                            attrs="{'invisible': ['|',('composition_mode', 'not in', ('comment', 'numbers')),('recipient_single_valid', '=', False)]}"/>
                    <button string="Send SMS" type="object" class="oe_highlight" name="action_send_sms" data-hotkey="q"
                            attrs="{'invisible': ['|',('composition_mode', 'not in', ('comment', 'numbers')),('recipient_single_valid', '=', True)]}" disabled='1'/>
                    <button string="Put in queue" type="object" class="oe_highlight" name="action_send_sms" data-hotkey="q"
                        attrs="{'invisible': [('composition_mode', '!=', 'mass')]}"/>
                    <button string="Send Now" type="object" name="action_send_sms_mass_now" data-hotkey="w"
                        attrs="{'invisible': [('composition_mode', '!=', 'mass')]}"/>
                    <button string="Close" class="btn btn-secondary" special="cancel" data-hotkey="z"/>
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
    failure_type = fields.Selection(
        related='notification_id.failure_type', related_sudo=True, readonly=True)
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
    has_cancel = fields.Boolean(compute='_compute_has_cancel')
    has_insufficient_credit = fields.Boolean(compute='_compute_has_insufficient_credit')
    has_unregistered_account = fields.Boolean(compute='_compute_has_unregistered_account')

    @api.depends("recipient_ids.failure_type")
    def _compute_has_unregistered_account(self):
        self.has_unregistered_account = self.recipient_ids.filtered(lambda p: p.failure_type == 'sms_acc')

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
            for pid, active, pshare, notif, groups in self.env['mail.followers']._get_recipient_data(record, 'sms', False, pids=pids):
                if pid and notif == 'sms':
                    rdata.append({'id': pid, 'share': pshare, 'active': active, 'notif': notif, 'groups': groups or [], 'type': 'customer' if pshare else 'user'})
            if rdata or numbers:
                record._notify_record_by_sms(
                    self.mail_message_id, rdata, check_existing=True,
                    sms_numbers=numbers, sms_pid_to_number=sms_pid_to_number,
                    put_in_queue=False
                )

        self.mail_message_id._notify_message_notification_update()
        return {'type': 'ir.actions.act_window_close'}

    def action_cancel(self):
        self._check_access()

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
                <field name="has_cancel" invisible="1"/>
                <field name="has_insufficient_credit" invisible="1"/>
                <field name="has_unregistered_account" invisible="1"/>
                <field name="recipient_ids">
                    <tree string="Recipient" editable="top" create="0" delete="0">
                        <field name="partner_name"/>
                        <field name="sms_number"/>
                        <field name="failure_type" string="Reason"/>
                        <field name="resend" widget="boolean_toggle"/>
                        <field name="notification_id" invisible="1"/>
                    </tree>
                </field>
                <div class="alert alert-warning" role="alert" attrs="{'invisible': [('has_cancel', '=', False)]}">
                    <span class="fa fa-info-circle"/> Caution: It won't be possible to send this SMS again to the recipients you did not select.
                </div>
                <footer>
                    <button string="Buy credits" name="action_buy_credits" type="object" class="btn-primary o_mail_send" 
                            attrs="{'invisible': [('has_insufficient_credit', '=', False)]}" data-hotkey="q"/>
                    <button string="Set up an account" name="action_buy_credits" type="object" class="btn-primary o_mail_send" 
                            attrs="{'invisible': [('has_unregistered_account', '=', False)]}" data-hotkey="q"/>
                    <button string="Resend" name="action_resend" type="object" class="btn-primary o_mail_send" data-hotkey="w"/>
                    <button string="Ignore all" name="action_cancel" type="object" class="btn-secondary" data-hotkey="x"/>
                    <button string="Discard" class="btn-secondary" special="cancel" data-hotkey="z"/>
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
                            <field name="resource_ref" class="oe_inline" options="{'hide_model': True, 'no_create': True, 'no_edit': True, 'no_open': True}" attrs="{'invisible': [('no_record', '=', True)]}"/>
                            <span class="text-warning" attrs="{'invisible': [('no_record', '=', False)]}">No records</span>
                        </div>
                    </div>
                    <p>Choose a language: <field name="lang" class="oe_inline ml8"/></p>
                    <label for="body" string="SMS content"/>
                    <hr/>
                        <field name="body" readonly="1" nolabel="1" options='{"safe": True}'/>
                    <hr/>
                    <footer>
                        <button string="Discard" class="btn-secondary" special="cancel" data-hotkey="z"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="sms_template_preview_action" model="ir.actions.act_window">
            <field name="name">Template Preview</field>
            <field name="res_model">sms.template.preview</field>
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

