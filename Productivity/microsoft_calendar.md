# Odoo Module: microsoft_calendar

Category: Productivity

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import utils
from . import wizard

from odoo import api, SUPERUSER_ID
import uuid


def init_initiating_microsoft_uuid(cr, registry):
    """ Sets the company name as the default value for the initiating
    party name on all existing companies once the module is installed. """
    env = api.Environment(cr, SUPERUSER_ID, {})
    config_parameter = env['ir.config_parameter'].sudo()
    microsoft_guid = config_parameter.get_param('microsoft_calendar.microsoft_guid', False)
    if not microsoft_guid:
        config_parameter.set_param('microsoft_calendar.microsoft_guid', str(uuid.uuid4()))

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Outlook Calendar',
    'version': '1.0',
    'category': 'Productivity',
    'description': "",
    'depends': ['microsoft_account', 'calendar'],
    'qweb': ['static/src/xml/*.xml'],
    'data': [
        'data/microsoft_calendar_data.xml',
        'security/ir.model.access.csv',
        'wizard/reset_account_views.xml',
        'views/res_config_settings_views.xml',
        'views/res_users_views.xml',
        'views/microsoft_calendar_views.xml',
        'views/microsoft_calendar_templates.xml',
    ],
    'demo': [],
    'installable': True,
    'auto_install': False,
    'post_init_hook': 'init_initiating_microsoft_uuid',
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request


class MicrosoftCalendarController(http.Controller):

    @http.route('/microsoft_calendar/sync_data', type='json', auth='user')
    def sync_data(self, model, **kw):
        """ This route/function is called when we want to synchronize Odoo
            calendar with Microsoft Calendar.
            Function return a dictionary with the status :  need_config_from_admin, need_auth,
            need_refresh, sync_stopped, success if not calendar_event
            The dictionary may contains an url, to allow Odoo Client to redirect user on
            this URL for authorization for example
        """
        if model == 'calendar.event':
            MicrosoftCal = request.env["calendar.event"]._get_microsoft_service()

            # Checking that admin have already configured Microsoft API for microsoft synchronization !
            client_id = request.env['ir.config_parameter'].sudo().get_param('microsoft_calendar_client_id')

            if not client_id or client_id == '':
                action_id = ''
                if MicrosoftCal._can_authorize_microsoft(request.env.user):
                    action_id = request.env.ref('base_setup.action_general_configuration').id
                return {
                    "status": "need_config_from_admin",
                    "url": '',
                    "action": action_id
                }

            # Checking that user have already accepted Odoo to access his calendar !
            if not MicrosoftCal.is_authorized(request.env.user):
                url = MicrosoftCal._microsoft_authentication_url(from_url=kw.get('fromurl'))
                return {
                    "status": "need_auth",
                    "url": url
                }
            # If App authorized, and user access accepted, We launch the synchronization
            need_refresh = request.env.user.sudo()._sync_microsoft_calendar()
            return {
                "status": "need_refresh" if need_refresh else "no_new_event_from_microsoft",
                "url": ''
            }

        return {"status": "success"}

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\microsoft_calendar_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record forcecreate="True" id="ir_cron_sync_all_cals" model="ir.cron">
            <field name="name">Outlook: synchronization</field>
            <field name="model_id" ref="model_res_users"/>
            <field name="state">code</field>
            <field name="code">
model._sync_all_microsoft_calendar()
            </field>
            <field name="user_id" ref="base.user_root" />
            <field name="interval_number">12</field>
            <field name="interval_type">hours</field>
            <field name="numbercall">-1</field>
            <field eval="False" name="doall" />
        </record>
    </data>
</odoo>

```

## File: models\calendar.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pytz
from datetime import datetime
from dateutil.parser import parse
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError
from odoo.tools import html2plaintext, is_html_empty, email_normalize
from odoo.addons.microsoft_calendar.utils.event_id_storage import combine_ids

ATTENDEE_CONVERTER_O2M = {
    'needsAction': 'notresponded',
    'tentative': 'tentativelyaccepted',
    'declined': 'declined',
    'accepted': 'accepted'
}
ATTENDEE_CONVERTER_M2O = {
    'none': 'needsAction',
    'notResponded': 'needsAction',
    'tentativelyAccepted': 'tentative',
    'declined': 'declined',
    'accepted': 'accepted',
    'organizer': 'accepted',
}
MAX_RECURRENT_EVENT = 720

_logger = logging.getLogger(__name__)

class Meeting(models.Model):
    _name = 'calendar.event'
    _inherit = ['calendar.event', 'microsoft.calendar.sync']

    # contains organizer event id and universal event id separated by a ':'
    microsoft_id = fields.Char('Microsoft Calendar Event Id')
    microsoft_recurrence_master_id = fields.Char('Microsoft Recurrence Master Id')

    def _get_organizer(self):
        return self.user_id

    @api.model
    def _get_microsoft_synced_fields(self):
        return {'name', 'description', 'allday', 'start', 'date_end', 'stop',
                'user_id', 'privacy',
                'attendee_ids', 'alarm_ids', 'location', 'show_as', 'active'}

    @api.model_create_multi
    def create(self, vals_list):
        notify_context = self.env.context.get('dont_notify', False)

        # for a recurrent event, we do not create events separately but we directly
        # create the recurrency from the corresponding calendar.recurrence.
        # That's why, events from a recurrency have their `need_sync_m` attribute set to False.
        return super(Meeting, self.with_context(dont_notify=notify_context)).create([
            dict(vals, need_sync_m=False) if vals.get('recurrence_id') or vals.get('recurrency') else vals
            for vals in vals_list
        ])

    def _check_recurrence_overlapping(self, new_start):
        """
        Outlook does not allow to modify time fields of an event if this event crosses
        or overlaps the recurrence. In this case a 400 error with the Outlook code "ErrorOccurrenceCrossingBoundary"
        is returned. That means that the update violates the following Outlook restriction on recurrence exceptions:
        an occurrence cannot be moved to or before the day of the previous occurrence, and cannot be moved to or after
        the day of the following occurrence.
        For example: E1 E2 E3 E4 cannot becomes E1 E3 E2 E4
        """
        before_count = len(self.recurrence_id.calendar_event_ids.filtered(
            lambda e: e.start.date() < self.start.date() and e != self
        ))
        after_count = len(self.recurrence_id.calendar_event_ids.filtered(
            lambda e: e.start.date() < parse(new_start).date() and e != self
        ))
        if before_count != after_count:
            raise UserError(_(
                "Outlook limitation: in a recurrence, an event cannot be moved to or before the day of the "
                "previous event, and cannot be moved to or after the day of the following event."
            ))

    def _is_matching_timeslot(self, start, stop, allday):
        """
        Check if an event matches with the provided timeslot
        """
        self.ensure_one()

        event_start, event_stop = self._range()
        if allday:
            event_start = datetime(event_start.year, event_start.month, event_start.day, 0, 0)
            event_stop = datetime(event_stop.year, event_stop.month, event_stop.day, 0, 0)

        return (event_start, event_stop) == (start, stop)

    def write(self, values):
        recurrence_update_setting = values.get('recurrence_update')

        # check a Outlook limitation in overlapping the actual recurrence
        if recurrence_update_setting == 'self_only' and 'start' in values:
            self._check_recurrence_overlapping(values['start'])

        # if a single event becomes the base event of a recurrency, it should be first
        # removed from the Outlook calendar.
        if 'recurrency' in values and values['recurrency']:
            for e in self.filtered(lambda e: not e.recurrency and not e.recurrence_id):
                e._microsoft_delete(e._get_organizer(), e.ms_organizer_event_id, timeout=3)
                e.microsoft_id = False

        notify_context = self.env.context.get('dont_notify', False)
        res = super(Meeting, self.with_context(dont_notify=notify_context)).write(values)

        if recurrence_update_setting in ('all_events',) and len(self) == 1 \
           and values.keys() & self._get_microsoft_synced_fields():
            self.recurrence_id.need_sync_m = True
        return res

    def _get_microsoft_sync_domain(self):
        # in case of full sync, limit to a range of 1y in past and 1y in the future by default
        ICP = self.env['ir.config_parameter'].sudo()
        day_range = int(ICP.get_param('microsoft_calendar.sync.range_days', default=365))
        lower_bound = fields.Datetime.subtract(fields.Datetime.now(), days=day_range)
        upper_bound = fields.Datetime.add(fields.Datetime.now(), days=day_range)
        return [
            ('partner_ids.user_ids', 'in', self.env.user.id),
            ('stop', '>', lower_bound),
            ('start', '<', upper_bound),
            # Do not sync events that follow the recurrence, they are already synced at recurrence creation
            '!', '&', '&', ('recurrency', '=', True), ('recurrence_id', '!=', False), ('follow_recurrence', '=', True)
        ]


    @api.model
    def _microsoft_to_odoo_values(self, microsoft_event, default_reminders=(), default_values=None, with_ids=False):
        if microsoft_event.is_cancelled():
            return {'active': False}

        sensitivity_o2m = {
            'normal': 'public',
            'private': 'private',
            'confidential': 'confidential',
        }

        commands_attendee, commands_partner = self._odoo_attendee_commands_m(microsoft_event)
        timeZone_start = pytz.timezone(microsoft_event.start.get('timeZone'))
        timeZone_stop = pytz.timezone(microsoft_event.end.get('timeZone'))
        start = parse(microsoft_event.start.get('dateTime')).astimezone(timeZone_start).replace(tzinfo=None)
        if microsoft_event.isAllDay:
            stop = parse(microsoft_event.end.get('dateTime')).astimezone(timeZone_stop).replace(tzinfo=None) - relativedelta(days=1)
        else:
            stop = parse(microsoft_event.end.get('dateTime')).astimezone(timeZone_stop).replace(tzinfo=None)
        values = default_values or {}
        values.update({
            'name': microsoft_event.subject or _("(No title)"),
            'description': microsoft_event.body and microsoft_event.body['content'],
            'location': microsoft_event.location and microsoft_event.location.get('displayName') or False,
            'user_id': microsoft_event.owner_id(self.env),
            'privacy': sensitivity_o2m.get(microsoft_event.sensitivity, self.default_get(['privacy'])['privacy']),
            'attendee_ids': commands_attendee,
            'allday': microsoft_event.isAllDay,
            'start': start,
            'stop': stop,
            'show_as': 'free' if microsoft_event.showAs == 'free' else 'busy',
            'recurrency': microsoft_event.is_recurrent()
        })
        if commands_partner:
            # Add partner_commands only if set from Microsoft. The write method on calendar_events will
            # override attendee commands if the partner_ids command is set but empty.
            values['partner_ids'] = commands_partner

        if microsoft_event.is_recurrent() and not microsoft_event.is_recurrence():
            # Propagate the follow_recurrence according to the Outlook result
            values['follow_recurrence'] = not microsoft_event.is_recurrence_outlier()

        if with_ids:
            values['microsoft_id'] = combine_ids(microsoft_event.id, microsoft_event.iCalUId)

        if microsoft_event.is_recurrent():
            values['microsoft_recurrence_master_id'] = microsoft_event.seriesMasterId

        alarm_commands = self._odoo_reminders_commands_m(microsoft_event)
        if alarm_commands:
            values['alarm_ids'] = alarm_commands

        return values

    @api.model
    def _microsoft_to_odoo_recurrence_values(self, microsoft_event, default_values=None):
        timeZone_start = pytz.timezone(microsoft_event.start.get('timeZone'))
        timeZone_stop = pytz.timezone(microsoft_event.end.get('timeZone'))
        start = parse(microsoft_event.start.get('dateTime')).astimezone(timeZone_start).replace(tzinfo=None)
        if microsoft_event.isAllDay:
            stop = parse(microsoft_event.end.get('dateTime')).astimezone(timeZone_stop).replace(tzinfo=None) - relativedelta(days=1)
        else:
            stop = parse(microsoft_event.end.get('dateTime')).astimezone(timeZone_stop).replace(tzinfo=None)
        values = default_values or {}
        values.update({
            'microsoft_id': combine_ids(microsoft_event.id, microsoft_event.iCalUId),
            'microsoft_recurrence_master_id': microsoft_event.seriesMasterId,
            'start': start,
            'stop': stop,
        })
        return values

    @api.model
    def _odoo_attendee_commands_m(self, microsoft_event):
        commands_attendee = []
        commands_partner = []

        microsoft_attendees = microsoft_event.attendees or []
        emails = [
            a.get('emailAddress').get('address')
            for a in microsoft_attendees
            if email_normalize(a.get('emailAddress').get('address'))
        ]
        existing_attendees = self.env['calendar.attendee']
        if microsoft_event.match_with_odoo_events(self.env):
            existing_attendees = self.env['calendar.attendee'].search([
                ('event_id', '=', microsoft_event.odoo_id(self.env)),
                ('email', 'in', emails)])
        elif self.env.user.partner_id.email not in emails:
            commands_attendee += [(0, 0, {'state': 'accepted', 'partner_id': self.env.user.partner_id.id})]
            commands_partner += [(4, self.env.user.partner_id.id)]
        partners = self.env['mail.thread']._mail_find_partner_from_emails(emails, records=self, force_create=True)
        attendees_by_emails = {a.email: a for a in existing_attendees}
        for email, partner, attendee_info in zip(emails, partners, microsoft_attendees):
            state = ATTENDEE_CONVERTER_M2O.get(attendee_info.get('status').get('response'), 'needsAction')

            if email in attendees_by_emails:
                # Update existing attendees
                commands_attendee += [(1, attendees_by_emails[email].id, {'state': state})]
            else:
                # Create new attendees
                commands_attendee += [(0, 0, {'state': state, 'partner_id': partner.id})]
                commands_partner += [(4, partner.id)]
                if attendee_info.get('emailAddress').get('name') and not partner.name:
                    partner.name = attendee_info.get('emailAddress').get('name')
        for odoo_attendee in attendees_by_emails.values():
            # Remove old attendees
            if odoo_attendee.email not in emails:
                commands_attendee += [(2, odoo_attendee.id)]
                commands_partner += [(3, odoo_attendee.partner_id.id)]
        return commands_attendee, commands_partner

    @api.model
    def _odoo_reminders_commands_m(self, microsoft_event):
        reminders_commands = []
        if microsoft_event.isReminderOn:
            event_id = self.browse(microsoft_event.odoo_id(self.env))
            alarm_type_label = _("Notification")

            minutes = microsoft_event.reminderMinutesBeforeStart or 0
            alarm = self.env['calendar.alarm'].search([
                ('alarm_type', '=', 'notification'),
                ('duration_minutes', '=', minutes)
            ], limit=1)
            if alarm and alarm not in event_id.alarm_ids:
                reminders_commands = [(4, alarm.id)]
            elif not alarm:
                if minutes == 0:
                    interval = 'minutes'
                    duration = minutes
                    name = _("%s - At time of event", alarm_type_label)
                elif minutes % (60*24) == 0:
                    interval = 'days'
                    duration = minutes / 60 / 24
                    name = _(
                        "%(reminder_type)s - %(duration)s Days",
                        reminder_type=alarm_type_label,
                        duration=duration,
                    )
                elif minutes % 60 == 0:
                    interval = 'hours'
                    duration = minutes / 60
                    name = _(
                        "%(reminder_type)s - %(duration)s Hours",
                        reminder_type=alarm_type_label,
                        duration=duration,
                    )
                else:
                    interval = 'minutes'
                    duration = minutes
                    name = _(
                        "%(reminder_type)s - %(duration)s Minutes",
                        reminder_type=alarm_type_label,
                        duration=duration,
                    )
                reminders_commands = [(0, 0, {'duration': duration, 'interval': interval, 'name': name, 'alarm_type': 'notification'})]

            alarm_to_rm = event_id.alarm_ids.filtered(lambda a: a.alarm_type == 'notification' and a.id != alarm.id)
            if alarm_to_rm:
                reminders_commands += [(3, a.id) for a in alarm_to_rm]

        else:
            event_id = self.browse(microsoft_event.odoo_id(self.env))
            alarm_to_rm = event_id.alarm_ids.filtered(lambda a: a.alarm_type == 'notification')
            if alarm_to_rm:
                reminders_commands = [(3, a.id) for a in alarm_to_rm]
        return reminders_commands

    def _get_attendee_status_o2m(self, attendee):
        if self.user_id and self.user_id == attendee.partner_id.user_id:
            return 'organizer'
        return ATTENDEE_CONVERTER_O2M.get(attendee.state, 'None')

    def _microsoft_values(self, fields_to_sync, initial_values={}):
        values = dict(initial_values)
        if not fields_to_sync:
            return values

        microsoft_guid = self.env['ir.config_parameter'].sudo().get_param('microsoft_calendar.microsoft_guid', False)

        if self.microsoft_recurrence_master_id and 'type' not in values:
            values['seriesMasterId'] = self.microsoft_recurrence_master_id
            values['type'] = 'exception'

        if 'name' in fields_to_sync:
            values['subject'] = self.name or ''

        if 'description' in fields_to_sync:
            values['body'] = {
                'content': html2plaintext(self.description) if not is_html_empty(self.description) else '',
                'contentType': "text",
            }

        if any(x in fields_to_sync for x in ['allday', 'start', 'date_end', 'stop']):
            if self.allday:
                start = {'dateTime': self.start_date.isoformat(), 'timeZone': 'Europe/London'}
                end = {'dateTime': (self.stop_date + relativedelta(days=1)).isoformat(), 'timeZone': 'Europe/London'}
            else:
                start = {'dateTime': pytz.utc.localize(self.start).isoformat(), 'timeZone': 'Europe/London'}
                end = {'dateTime': pytz.utc.localize(self.stop).isoformat(), 'timeZone': 'Europe/London'}

            values['start'] = start
            values['end'] = end
            values['isAllDay'] = self.allday

        if 'location' in fields_to_sync:
            values['location'] = {'displayName': self.location or ''}

        if 'alarm_ids' in fields_to_sync:
            alarm_id = self.alarm_ids.filtered(lambda a: a.alarm_type == 'notification')[:1]
            values['isReminderOn'] = bool(alarm_id)
            values['reminderMinutesBeforeStart'] = alarm_id.duration_minutes

        if 'user_id' in fields_to_sync:
            values['organizer'] = {'emailAddress': {'address': self.user_id.email or '', 'name': self.user_id.display_name or ''}}
            values['isOrganizer'] = self.user_id == self.env.user

        if 'attendee_ids' in fields_to_sync:
            attendees = self.attendee_ids.filtered(lambda att: att.partner_id not in self.user_id.partner_id)
            values['attendees'] = [
                {
                    'emailAddress': {'address': attendee.email or '', 'name': attendee.display_name or ''},
                    'status': {'response': self._get_attendee_status_o2m(attendee)}
                } for attendee in attendees]

        if 'privacy' in fields_to_sync or 'show_as' in fields_to_sync:
            values['showAs'] = self.show_as
            sensitivity_o2m = {
                'public': 'normal',
                'private': 'private',
                'confidential': 'confidential',
            }
            values['sensitivity'] = sensitivity_o2m.get(self.privacy)

        if 'active' in fields_to_sync and not self.active:
            values['isCancelled'] = True

        if values.get('type') == 'seriesMaster':
            recurrence = self.recurrence_id
            pattern = {
                'interval': recurrence.interval
            }
            if recurrence.rrule_type in ['daily', 'weekly']:
                pattern['type'] = recurrence.rrule_type
            else:
                prefix = 'absolute' if recurrence.month_by == 'date' else 'relative'
                pattern['type'] = recurrence.rrule_type and prefix + recurrence.rrule_type.capitalize()

            if recurrence.month_by == 'date':
                pattern['dayOfMonth'] = recurrence.day

            if recurrence.month_by == 'day' or recurrence.rrule_type == 'weekly':
                pattern['daysOfWeek'] = [
                    weekday_name for weekday_name, weekday in {
                        'monday': recurrence.mo,
                        'tuesday': recurrence.tu,
                        'wednesday': recurrence.we,
                        'thursday': recurrence.th,
                        'friday': recurrence.fr,
                        'saturday': recurrence.sa,
                        'sunday': recurrence.su,
                    }.items() if weekday]
                pattern['firstDayOfWeek'] = 'sunday'

            if recurrence.rrule_type == 'monthly' and recurrence.month_by == 'day':
                byday_selection = {
                    '1': 'first',
                    '2': 'second',
                    '3': 'third',
                    '4': 'fourth',
                    '-1': 'last',
                }
                pattern['index'] = byday_selection[recurrence.byday]

            dtstart = recurrence.dtstart or fields.Datetime.now()
            rule_range = {
                'startDate': (dtstart.date()).isoformat()
            }

            if recurrence.end_type == 'count':  # e.g. stop after X occurence
                rule_range['numberOfOccurrences'] = min(recurrence.count, MAX_RECURRENT_EVENT)
                rule_range['type'] = 'numbered'
            elif recurrence.end_type == 'forever':
                rule_range['numberOfOccurrences'] = MAX_RECURRENT_EVENT
                rule_range['type'] = 'numbered'
            elif recurrence.end_type == 'end_date':  # e.g. stop after 12/10/2020
                rule_range['endDate'] = recurrence.until.isoformat()
                rule_range['type'] = 'endDate'

            values['recurrence'] = {
                'pattern': pattern,
                'range': rule_range
            }

        return values

    def _ensure_attendees_have_email(self):
        invalid_event_ids = self.env['calendar.event'].search_read(
            domain=[('id', 'in', self.ids), ('attendee_ids.partner_id.email', '=', False)],
            fields=['display_time', 'display_name'],
            order='start',
        )
        if invalid_event_ids:
            list_length_limit = 50
            total_invalid_events = len(invalid_event_ids)
            invalid_event_ids = invalid_event_ids[:list_length_limit]
            invalid_events = ['\t- %s: %s' % (event['display_time'], event['display_name'])
                              for event in invalid_event_ids]
            invalid_events = '\n'.join(invalid_events)
            details = "(%d/%d)" % (list_length_limit, total_invalid_events) if list_length_limit < total_invalid_events else "(%d)" % total_invalid_events
            raise ValidationError(_("For a correct synchronization between Odoo and Outlook Calendar, "
                                    "all attendees must have an email address. However, some events do "
                                    "not respect this condition. As long as the events are incorrect, "
                                    "the calendars will not be synchronized."
                                    "\nEither update the events/attendees or archive these events %s:"
                                    "\n%s", details, invalid_events))

    def _microsoft_values_occurence(self, initial_values={}):
        values = initial_values
        values['type'] = 'occurrence'

        if self.allday:
            start = {'dateTime': self.start_date.isoformat(), 'timeZone': 'Europe/London'}
            end = {'dateTime': (self.stop_date + relativedelta(days=1)).isoformat(), 'timeZone': 'Europe/London'}
        else:
            start = {'dateTime': pytz.utc.localize(self.start).isoformat(), 'timeZone': 'Europe/London'}
            end = {'dateTime': pytz.utc.localize(self.stop).isoformat(), 'timeZone': 'Europe/London'}

        values['start'] = start
        values['end'] = end
        values['isAllDay'] = self.allday

        return values

    def _cancel_microsoft(self):
        """
        Cancel an Microsoft event.
        There are 2 cases:
          1) the organizer is an Odoo user: he's the only one able to delete the Odoo event. Attendees can just decline.
          2) the organizer is NOT an Odoo user: any attendee should remove the Odoo event.
        """
        user = self.env.user
        records = self.filtered(lambda e: not e.user_id or e.user_id == user)
        super(Meeting, records)._cancel_microsoft()

```

## File: models\calendar_attendee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

from odoo.addons.microsoft_calendar.models.microsoft_sync import microsoft_calendar_token
from odoo.addons.microsoft_calendar.utils.microsoft_calendar import MicrosoftCalendarService


class Attendee(models.Model):
    _name = 'calendar.attendee'
    _inherit = 'calendar.attendee'

    def _send_mail_to_attendees(self, template_xmlid, force_send=False, ignore_recurrence=False):
        """ Override the super method
        If not synced with Microsoft Outlook, let Odoo in charge of sending emails
        Otherwise, Microsoft Outlook will send them
        """
        with microsoft_calendar_token(self.env.user.sudo()) as token:
            if not token:
                super()._send_mail_to_attendees(template_xmlid, force_send, ignore_recurrence)

    def do_tentative(self):
        # Synchronize event after state change
        res = super().do_tentative()
        self._microsoft_sync_event('tentativelyAccept')
        return res

    def do_accept(self):
        # Synchronize event after state change
        res = super().do_accept()
        self._microsoft_sync_event('accept')
        return res

    def do_decline(self):
        # Synchronize event after state change
        res = super().do_decline()
        self._microsoft_sync_event('decline')
        return res

    def _microsoft_sync_event(self, answer):
        params = {"comment": "", "sendResponse": True}
        # Microsoft prevent user to answer the meeting when they are the organizer
        linked_events = self.event_id._get_synced_events()
        for event in linked_events.filtered(lambda e: e.user_id != self.env.user):
            event._microsoft_patch(
                event._get_organizer(),
                event.ms_organizer_event_id,
                event._microsoft_values(["attendee_ids"]),
            )

```

## File: models\calendar_recurrence_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class RecurrenceRule(models.Model):
    _name = 'calendar.recurrence'
    _inherit = ['calendar.recurrence', 'microsoft.calendar.sync']


    # Don't sync by default. Sync only when the recurrence is applied
    need_sync_m = fields.Boolean(default=False)

    microsoft_id = fields.Char('Microsoft Calendar Recurrence Id')

    def _compute_rrule(self):
        # Note: 'need_sync_m' is set to False to avoid syncing the updated recurrence with
        # Outlook, as this update may already come from Outlook. If not, this modification will
        # be already synced through the calendar.event.write()
        for recurrence in self:
            if recurrence.rrule != recurrence._rrule_serialize():
                recurrence.write({'rrule': recurrence._rrule_serialize()})

    def _inverse_rrule(self):
        # Note: 'need_sync_m' is set to False to avoid syncing the updated recurrence with
        # Outlook, as this update mainly comes from Outlook (the 'rrule' field is not directly
        # modified in Odoo but computed from other fields).
        for recurrence in self.filtered('rrule'):
            values = self._rrule_parse(recurrence.rrule, recurrence.dtstart)
            recurrence.write(dict(values, need_sync_m=False))

    def _apply_recurrence(self, specific_values_creation=None, no_send_edit=False):
        events = self.filtered('need_sync_m').calendar_event_ids
        detached_events = super()._apply_recurrence(specific_values_creation, no_send_edit)

        # If a synced event becomes a recurrence, the event needs to be deleted from
        # Microsoft since it's now the recurrence which is synced.
        vals = []
        for event in events._get_synced_events():
            if event.active and event.ms_universal_event_id and not event.recurrence_id.ms_universal_event_id:
                vals += [{
                    'name': event.name,
                    'microsoft_id': event.microsoft_id,
                    'start': event.start,
                    'stop': event.stop,
                    'active': False,
                    'need_sync_m': True,
                }]
                event._microsoft_delete(event.user_id, event.ms_organizer_event_id)
                event.ms_universal_event_id = False
        self.env['calendar.event'].create(vals)
        self.calendar_event_ids.need_sync_m = False
        return detached_events

    def _write_events(self, values, dtstart=None):
        # If only some events are updated, sync those events.
        # If all events are updated, sync the recurrence instead.
        values['need_sync_m'] = bool(dtstart) or values.get("need_sync_m", True)
        return super()._write_events(values, dtstart=dtstart)

    def _get_organizer(self):
        return self.base_event_id.user_id

    def _get_rrule(self, dtstart=None):
        if not dtstart and self.dtstart:
            dtstart = self.dtstart
        return super()._get_rrule(dtstart)

    def _get_microsoft_synced_fields(self):
        return {'rrule'} | self.env['calendar.event']._get_microsoft_synced_fields()

    def _has_base_event_time_fields_changed(self, new):
        """
        Indicates if at least one time field of the base event has changed, based
        on provided `new` values.
        Note: for all day event comparison, hours/minutes are ignored.
        """
        def _convert(value, to_convert):
            return value.date() if to_convert else value

        old = self.base_event_id and self.base_event_id.read(['start', 'stop', 'allday'])[0]
        return old and (
            old['allday'] != new['allday']
            or any(
                _convert(new[f], new['allday']) != _convert(old[f], old['allday'])
                for f in ('start', 'stop')
            )
        )

    def _write_from_microsoft(self, microsoft_event, vals):
        current_rrule = self.rrule
        # event_tz is written on event in Microsoft but on recurrence in Odoo
        vals['event_tz'] = microsoft_event.start.get('timeZone')
        super()._write_from_microsoft(microsoft_event, vals)
        new_event_values = self.env["calendar.event"]._microsoft_to_odoo_values(microsoft_event)
        if self._has_base_event_time_fields_changed(new_event_values):
            # we need to recreate the recurrence, time_fields were modified.
            base_event_id = self.base_event_id
            # We archive the old events to recompute the recurrence. These events are already deleted on Microsoft side.
            # We can't call _cancel because events without user_id would not be deleted
            (self.calendar_event_ids - base_event_id).microsoft_id = False
            (self.calendar_event_ids - base_event_id).unlink()
            base_event_id.with_context(dont_notify=True).write(dict(
                new_event_values, microsoft_id=False, need_sync_m=False
            ))
            if self.rrule == current_rrule:
                # if the rrule has changed, it will be recalculated below
                # There is no detached event now
                self.with_context(dont_notify=True)._apply_recurrence()
        else:
            time_fields = (
                    self.env["calendar.event"]._get_time_fields()
                    | self.env["calendar.event"]._get_recurrent_fields()
            )
            # We avoid to write time_fields because they are not shared between events.
            self._write_events(dict({
                field: value
                for field, value in new_event_values.items()
                if field not in time_fields
                }, need_sync_m=False)
            )
        # We apply the rrule check after the time_field check because the microsoft ids are generated according
        # to base_event start datetime.
        if self.rrule != current_rrule:
            detached_events = self._apply_recurrence()
            detached_events.microsoft_id = False
            detached_events.unlink()

    def _get_microsoft_sync_domain(self):
        # Empty rrule may exists in historical data. It is not a desired behavior but it could have been created with
        # older versions of the module. When synced, these recurrence may come back from Microsoft after database cleaning
        # and trigger errors as the records are not properly populated.
        # We also prevent sync of other user recurrent events.
        return [('calendar_event_ids.user_id', '=', self.env.user.id), ('rrule', '!=', False)]

    def _cancel_microsoft(self):
        self.calendar_event_ids._cancel_microsoft()
        super()._cancel_microsoft()

    @api.model
    def _microsoft_to_odoo_values(self, microsoft_recurrence, default_reminders=(), default_values=None, with_ids=False):
        recurrence = microsoft_recurrence.get_recurrence()

        if with_ids:
            recurrence = {
                **recurrence,
                'ms_organizer_event_id': microsoft_recurrence.id,
                'ms_universal_event_id': microsoft_recurrence.iCalUId,
            }

        return recurrence

    def _microsoft_values(self, fields_to_sync):
        """
        Get values to update the whole Outlook event recurrence.
        (done through the first event of the Outlook recurrence).
        """
        return self.base_event_id._microsoft_values(fields_to_sync, initial_values={'type': 'seriesMaster'})

    def _ensure_attendees_have_email(self):
        self.calendar_event_ids.filtered(lambda e: e.active)._ensure_attendees_have_email()

```

## File: models\microsoft_sync.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from contextlib import contextmanager
from functools import wraps
import pytz
from dateutil.parser import parse

from odoo import api, fields, models, registry
from odoo.tools import ormcache_context
from odoo.exceptions import UserError
from odoo.osv import expression

from odoo.addons.microsoft_calendar.utils.microsoft_event import MicrosoftEvent
from odoo.addons.microsoft_calendar.utils.microsoft_calendar import MicrosoftCalendarService
from odoo.addons.microsoft_calendar.utils.event_id_storage import IDS_SEPARATOR, combine_ids, split_ids
from odoo.addons.microsoft_account.models.microsoft_service import TIMEOUT

_logger = logging.getLogger(__name__)

MAX_RECURRENT_EVENT = 720

# API requests are sent to Microsoft Calendar after the current transaction ends.
# This ensures changes are sent to Microsoft only if they really happened in the Odoo database.
# It is particularly important for event creation , otherwise the event might be created
# twice in Microsoft if the first creation crashed in Odoo.
def after_commit(func):
    @wraps(func)
    def wrapped(self, *args, **kwargs):
        dbname = self.env.cr.dbname
        context = self.env.context
        uid = self.env.uid

        if self.env.context.get('no_calendar_sync'):
            return

        @self.env.cr.postcommit.add
        def called_after():
            db_registry = registry(dbname)
            with api.Environment.manage(), db_registry.cursor() as cr:
                env = api.Environment(cr, uid, context)
                try:
                    func(self.with_env(env), *args, **kwargs)
                except Exception as e:
                    _logger.warning("Could not sync record now: %s" % self)
                    _logger.exception(e)

    return wrapped

@contextmanager
def microsoft_calendar_token(user):
    yield user._get_microsoft_calendar_token()

class MicrosoftSync(models.AbstractModel):
    _name = 'microsoft.calendar.sync'
    _description = "Synchronize a record with Microsoft Calendar"

    microsoft_id = fields.Char('Microsoft Calendar Id', copy=False)

    ms_organizer_event_id = fields.Char(
        'Organizer event Id',
        compute='_compute_organizer_event_id',
        inverse='_set_event_id',
        search='_search_organizer_event_id',
    )
    ms_universal_event_id = fields.Char(
        'Universal event Id',
        compute='_compute_universal_event_id',
        inverse='_set_event_id',
        search='_search_universal_event_id',
    )

    # This field helps to know when a microsoft event need to be resynced
    need_sync_m = fields.Boolean(default=True, copy=False)
    active = fields.Boolean(default=True)

    def write(self, vals):
        if 'ms_universal_event_id' in vals:
            self._from_uids.clear_cache(self)

        fields_to_sync = [x for x in vals.keys() if x in self._get_microsoft_synced_fields()]
        if fields_to_sync and 'need_sync_m' not in vals:
            vals['need_sync_m'] = True

        result = super().write(vals)

        for record in self.filtered(lambda e: e.need_sync_m and e.ms_organizer_event_id):
            if not vals.get('active', True):
                # We need to delete the event. Cancel is not sufficant. Errors may occurs
                record._microsoft_delete(record._get_organizer(), record.ms_organizer_event_id, timeout=3)
            elif fields_to_sync:
                values = record._microsoft_values(fields_to_sync)
                if not values:
                    continue
                record._microsoft_patch(record._get_organizer(), record.ms_organizer_event_id, values, timeout=3)

        return result

    @api.model_create_multi
    def create(self, vals_list):
        records = super().create(vals_list)

        records_to_sync = records.filtered(lambda r: r.need_sync_m and r.active)
        for record in records_to_sync:
            record._microsoft_insert(record._microsoft_values(self._get_microsoft_synced_fields()), timeout=3)
        return records

    @api.depends('microsoft_id')
    def _compute_organizer_event_id(self):
        for event in self:
            event.ms_organizer_event_id = split_ids(event.microsoft_id)[0] if event.microsoft_id else False

    @api.depends('microsoft_id')
    def _compute_universal_event_id(self):
        for event in self:
            event.ms_universal_event_id = split_ids(event.microsoft_id)[1] if event.microsoft_id else False

    def _set_event_id(self):
        for event in self:
            event.microsoft_id = combine_ids(event.ms_organizer_event_id, event.ms_universal_event_id)

    def _search_event_id(self, operator, value, with_uid):
        def _domain(v):
            return ('microsoft_id', '=like', f'%{IDS_SEPARATOR}{v}' if with_uid else f'{v}%')

        if operator == '=' and not value:
            return (
                ['|', ('microsoft_id', '=', False), ('microsoft_id', '=ilike', f'%{IDS_SEPARATOR}')]
                if with_uid
                else [('microsoft_id', '=', False)]
            )
        elif operator == '!=' and not value:
            return (
                [('microsoft_id', 'ilike', f'{IDS_SEPARATOR}_')]
                if with_uid
                else [('microsoft_id', '!=', False)]
            )
        return (
            ['|'] * (len(value) - 1) + [_domain(v) for v in value]
            if operator.lower() == 'in'
            else [_domain(value)]
        )

    def _search_organizer_event_id(self, operator, value):
        return self._search_event_id(operator, value, with_uid=False)

    def _search_universal_event_id(self, operator, value):
        return self._search_event_id(operator, value, with_uid=True)

    @api.model
    def _get_microsoft_service(self):
        return MicrosoftCalendarService(self.env['microsoft.service'])

    def _get_synced_events(self):
        """
        Get events already synced with Microsoft Outlook.
        """
        return self.filtered(lambda e: e.ms_universal_event_id)

    def unlink(self):
        synced = self._get_synced_events()
        for ev in synced:
            ev._microsoft_delete(ev._get_organizer(), ev.ms_organizer_event_id)
        return super().unlink()

    def _write_from_microsoft(self, microsoft_event, vals):
        self.write(vals)

    @api.model
    def _create_from_microsoft(self, microsoft_event, vals_list):
        return self.create(vals_list)

    @api.model
    @ormcache_context('uids', keys=('active_test',))
    def _from_uids(self, uids):
        if not uids:
            return self.browse()
        return self.search([('ms_universal_event_id', 'in', uids)])

    def _sync_odoo2microsoft(self):
        if not self:
            return
        if self._active_name:
            records_to_sync = self.filtered(self._active_name)
        else:
            records_to_sync = self
        cancelled_records = self - records_to_sync

        records_to_sync._ensure_attendees_have_email()
        updated_records = records_to_sync._get_synced_events()
        new_records = records_to_sync - updated_records

        for record in cancelled_records._get_synced_events():
            record._microsoft_delete(record._get_organizer(), record.ms_organizer_event_id)
        for record in new_records:
            values = record._microsoft_values(self._get_microsoft_synced_fields())
            if isinstance(values, dict):
                record._microsoft_insert(values)
            else:
                for value in values:
                    record._microsoft_insert(value)
        for record in updated_records.filtered('need_sync_m'):
            values = record._microsoft_values(self._get_microsoft_synced_fields())
            if not values:
                continue
            record._microsoft_patch(record._get_organizer(), record.ms_organizer_event_id, values)

    def _cancel_microsoft(self):
        self.microsoft_id = False
        self.unlink()

    def _sync_recurrence_microsoft2odoo(self, microsoft_events, new_events=None):
        recurrent_masters = new_events.filter(lambda e: e.is_recurrence()) if new_events else []
        recurrents = new_events.filter(lambda e: e.is_recurrent_not_master()) if new_events else []
        default_values = {'need_sync_m': False}

        new_recurrence = self.env['calendar.recurrence']
        updated_events = self.env['calendar.event']

        # --- create new recurrences and associated events ---
        for recurrent_master in recurrent_masters:
            new_calendar_recurrence = dict(
                self.env['calendar.recurrence']._microsoft_to_odoo_values(recurrent_master, default_values, with_ids=True),
                need_sync_m=False
            )
            to_create = recurrents.filter(
                lambda e: e.seriesMasterId == new_calendar_recurrence['ms_organizer_event_id']
            )
            recurrents -= to_create
            base_values = dict(
                self.env['calendar.event']._microsoft_to_odoo_values(recurrent_master, default_values, with_ids=True),
                need_sync_m=False
            )
            to_create_values = []
            if new_calendar_recurrence.get('end_type', False) in ['count', 'forever']:
                to_create = list(to_create)[:MAX_RECURRENT_EVENT]
            for recurrent_event in to_create:
                if recurrent_event.type == 'occurrence':
                    value = self.env['calendar.event']._microsoft_to_odoo_recurrence_values(recurrent_event, base_values)
                else:
                    value = self.env['calendar.event']._microsoft_to_odoo_values(recurrent_event, default_values)

                to_create_values += [dict(value, need_sync_m=False)]

            new_calendar_recurrence['calendar_event_ids'] = [(0, 0, to_create_value) for to_create_value in to_create_values]
            new_recurrence_odoo = self.env['calendar.recurrence'].create(new_calendar_recurrence)
            new_recurrence_odoo.base_event_id = new_recurrence_odoo.calendar_event_ids[0] if new_recurrence_odoo.calendar_event_ids else False
            new_recurrence |= new_recurrence_odoo

        # --- update events in existing recurrences ---
        # Important note:
        # To map existing recurrences with events to update, we must use the universal id
        # (also known as ICalUId in the Microsoft API), as 'seriesMasterId' attribute of events
        # is specific to the Microsoft user calendar.
        ms_recurrence_ids = list({x.seriesMasterId for x in recurrents})
        ms_recurrence_uids = {r.id: r.iCalUId for r in microsoft_events if r.id in ms_recurrence_ids}

        recurrences = self.env['calendar.recurrence'].search([
            ('ms_universal_event_id', 'in', ms_recurrence_uids.values())
        ])
        for recurrent_master_id in ms_recurrence_ids:
            recurrence_id = recurrences.filtered(
                lambda ev: ev.ms_universal_event_id == ms_recurrence_uids[recurrent_master_id]
            )
            to_update = recurrents.filter(lambda e: e.seriesMasterId == recurrent_master_id)
            for recurrent_event in to_update:
                if recurrent_event.type == 'occurrence':
                    value = self.env['calendar.event']._microsoft_to_odoo_recurrence_values(
                        recurrent_event, {'need_sync_m': False}
                    )
                else:
                    value = self.env['calendar.event']._microsoft_to_odoo_values(recurrent_event, default_values)
                existing_event = recurrence_id.calendar_event_ids.filtered(
                    lambda e: e._is_matching_timeslot(value['start'], value['stop'], recurrent_event.isAllDay)
                )
                if not existing_event:
                    continue
                value.pop('start')
                value.pop('stop')
                existing_event._write_from_microsoft(recurrent_event, value)
                updated_events |= existing_event
            new_recurrence |= recurrence_id
        return new_recurrence, updated_events

    def _update_microsoft_recurrence(self, recurrence, events):
        """
        Update Odoo events from Outlook recurrence and events.
        """
        # get the list of events to update ...
        events_to_update = events.filter(lambda e: e.seriesMasterId == self.ms_organizer_event_id)
        if self.end_type in ['count', 'forever']:
            events_to_update = list(events_to_update)[:MAX_RECURRENT_EVENT]

        # ... and update them
        rec_values = {}
        update_events = self.env['calendar.event']
        for e in events_to_update:
            if e.type == "exception":
                event_values = self.env['calendar.event']._microsoft_to_odoo_values(e)
            elif e.type == "occurrence":
                event_values = self.env['calendar.event']._microsoft_to_odoo_recurrence_values(e)
            else:
                event_values = None

            if event_values:
                # keep event values to update the recurrence later
                if any(f for f in ('start', 'stop') if f in event_values):
                    rec_values[(self.id, event_values.get('start'), event_values.get('stop'))] = dict(
                        event_values, need_sync_m=False
                    )

                odoo_event = self.env['calendar.event'].browse(e.odoo_id(self.env)).exists().with_context(
                    no_mail_to_attendees=True, mail_create_nolog=True
                )
                odoo_event.write(dict(event_values, need_sync_m=False))
                update_events |= odoo_event

        # update the recurrence
        detached_events = self._apply_recurrence(rec_values)
        detached_events._cancel_microsoft()

        return update_events

    @api.model
    def _sync_microsoft2odoo(self, microsoft_events: MicrosoftEvent):
        """
        Synchronize Microsoft recurrences in Odoo.
        Creates new recurrences, updates existing ones.
        :return: synchronized odoo
        """
        existing = microsoft_events.match_with_odoo_events(self.env)
        cancelled = microsoft_events.cancelled()
        new = microsoft_events - existing - cancelled
        new_recurrence = new.filter(lambda e: e.is_recurrent())

        # create new events and reccurrences
        odoo_values = [
            dict(self._microsoft_to_odoo_values(e, with_ids=True), need_sync_m=False)
            for e in (new - new_recurrence)
        ]
        synced_events = self.with_context(dont_notify=True)._create_from_microsoft(new, odoo_values)
        synced_recurrences, updated_events = self._sync_recurrence_microsoft2odoo(existing, new_recurrence)
        synced_events |= updated_events

        # remove cancelled events and recurrences
        cancelled_recurrences = self.env['calendar.recurrence'].search([
            '|',
            ('ms_universal_event_id', 'in', cancelled.uids),
            ('ms_organizer_event_id', 'in', cancelled.ids),
        ])
        cancelled_events = self.browse([
            e.odoo_id(self.env)
            for e in cancelled
            if e.id not in [r.ms_organizer_event_id for r in cancelled_recurrences]
        ])
        cancelled_recurrences._cancel_microsoft()
        cancelled_events = cancelled_events.exists()
        cancelled_events._cancel_microsoft()

        synced_recurrences |= cancelled_recurrences
        synced_events |= cancelled_events | cancelled_recurrences.calendar_event_ids

        # update other events
        for mevent in (existing - cancelled).filter(lambda e: e.lastModifiedDateTime):
            # Last updated wins.
            # This could be dangerous if microsoft server time and odoo server time are different
            if mevent.is_recurrence():
                odoo_event = self.env['calendar.recurrence'].browse(mevent.odoo_id(self.env)).exists()
            else:
                odoo_event = self.browse(mevent.odoo_id(self.env)).exists()

            if odoo_event:
                odoo_event_updated_time = pytz.utc.localize(odoo_event.write_date)
                ms_event_updated_time = parse(mevent.lastModifiedDateTime)

                if ms_event_updated_time >= odoo_event_updated_time:
                    vals = dict(odoo_event._microsoft_to_odoo_values(mevent), need_sync_m=False)
                    odoo_event._write_from_microsoft(mevent, vals)

                    if odoo_event._name == 'calendar.recurrence':
                        update_events = odoo_event._update_microsoft_recurrence(mevent, microsoft_events)
                        synced_recurrences |= odoo_event
                        synced_events |= update_events
                    else:
                        synced_events |= odoo_event

        return synced_events, synced_recurrences

    def _impersonate_user(self, user_id):
        """ Impersonate a user (mainly the event organizer) to be able to call the Outlook API with its token """
        return user_id.with_user(user_id)

    @after_commit
    def _microsoft_delete(self, user_id, event_id, timeout=TIMEOUT):
        """
        Once the event has been really removed from the Odoo database, remove it from the Outlook calendar.

        Note that all self attributes to use in this method must be provided as method parameters because
        'self' won't exist when this method will be really called due to @after_commit decorator.
        """
        microsoft_service = self._get_microsoft_service()
        with microsoft_calendar_token(self._impersonate_user(user_id).sudo()) as token:
            if token:
                microsoft_service.delete(event_id, token=token, timeout=timeout)

    @after_commit
    def _microsoft_patch(self, user_id, event_id, values, timeout=TIMEOUT):
        """
        Once the event has been really modified in the Odoo database, modify it in the Outlook calendar.

        Note that all self attributes to use in this method must be provided as method parameters because
        'self' may have been modified between the call of '_microsoft_patch' and its execution,
        due to @after_commit decorator.
        """
        microsoft_service = self._get_microsoft_service()
        with microsoft_calendar_token(self._impersonate_user(user_id).sudo()) as token:
            if token:
                self._ensure_attendees_have_email()
                res = microsoft_service.patch(event_id, values, token=token, timeout=timeout)
                self.write({
                    'need_sync_m': not res,
                })

    @after_commit
    def _microsoft_insert(self, values, timeout=TIMEOUT):
        """
        Once the event has been really added in the Odoo database, add it in the Outlook calendar.

        Note that all self attributes to use in this method must be provided as method parameters because
        'self' may have been modified between the call of '_microsoft_insert' and its execution,
        due to @after_commit decorator.
        """
        if not values:
            return
        microsoft_service = self._get_microsoft_service()
        with microsoft_calendar_token(self.env.user.sudo()) as token:
            if token:
                self._ensure_attendees_have_email()
                event_id, uid = microsoft_service.insert(values, token=token, timeout=timeout)
                self.write({
                    'microsoft_id': combine_ids(event_id, uid),
                    'need_sync_m': False,
                })

    def _microsoft_attendee_answer(self, answer, params, timeout=TIMEOUT):
        if not answer:
            return
        microsoft_service = self._get_microsoft_service()
        with microsoft_calendar_token(self.env.user.sudo()) as token:
            if token:
                self._ensure_attendees_have_email()
                microsoft_service.answer(
                    self.ms_organizer_event_id,
                    answer, params, token=token, timeout=timeout
                )
                self.write({
                    'need_sync_m': False,
                })

    def _get_microsoft_records_to_sync(self, full_sync=False):
        """
        Return records that should be synced from Odoo to Microsoft
        :param full_sync: If True, all events attended by the user are returned
        :return: events
        """
        domain = self._get_microsoft_sync_domain()
        if not full_sync:
            is_active_clause = (self._active_name, '=', True) if self._active_name else expression.TRUE_LEAF
            domain = expression.AND([domain, [
                '|',
                '&', ('ms_universal_event_id', '=', False), is_active_clause,
                ('need_sync_m', '=', True),
            ]])
        return self.with_context(active_test=False).search(domain)

    @api.model
    def _microsoft_to_odoo_values(
        self, microsoft_event: MicrosoftEvent, default_reminders=(), default_values=None, with_ids=False
    ):
        """
        Implements this method to return a dict of Odoo values corresponding
        to the Microsoft event given as parameter
        :return: dict of Odoo formatted values
        """
        raise NotImplementedError()

    def _microsoft_values(self, fields_to_sync):
        """
        Implements this method to return a dict with values formatted
        according to the Microsoft Calendar API
        :return: dict of Microsoft formatted values
        """
        raise NotImplementedError()

    def _ensure_attendees_have_email(self):
        raise NotImplementedError()

    def _get_microsoft_sync_domain(self):
        """
        Return a domain used to search records to synchronize.
        e.g. return a domain to synchronize records owned by the current user.
        """
        raise NotImplementedError()

    def _get_microsoft_synced_fields(self):
        """
        Return a set of field names. Changing one of these fields
        marks the record to be re-synchronized.
        """
        raise NotImplementedError()

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    cal_microsoft_client_id = fields.Char("Microsoft Client_id", config_parameter='microsoft_calendar_client_id', default='')
    cal_microsoft_client_secret = fields.Char("Microsoft Client_key", config_parameter='microsoft_calendar_client_secret', default='')

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import requests
from odoo.addons.microsoft_calendar.models.microsoft_sync import microsoft_calendar_token
from datetime import timedelta

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.loglevels import exception_to_unicode
from odoo.addons.microsoft_account.models.microsoft_service import MICROSOFT_TOKEN_ENDPOINT
from odoo.addons.microsoft_calendar.utils.microsoft_calendar import InvalidSyncToken

_logger = logging.getLogger(__name__)


class User(models.Model):
    _inherit = 'res.users'

    microsoft_calendar_sync_token = fields.Char('Microsoft Next Sync Token', copy=False)

    def _microsoft_calendar_authenticated(self):
        return bool(self.sudo().microsoft_calendar_rtoken)

    def _get_microsoft_calendar_token(self):
        self.ensure_one()
        if self.microsoft_calendar_rtoken and not self._is_microsoft_calendar_valid():
            self._refresh_microsoft_calendar_token()
        return self.microsoft_calendar_token

    def _is_microsoft_calendar_valid(self):
        return self.microsoft_calendar_token_validity and self.microsoft_calendar_token_validity >= (fields.Datetime.now() + timedelta(minutes=1))

    def _refresh_microsoft_calendar_token(self):
        self.ensure_one()
        get_param = self.env['ir.config_parameter'].sudo().get_param
        client_id = get_param('microsoft_calendar_client_id')
        client_secret = get_param('microsoft_calendar_client_secret')

        if not client_id or not client_secret:
            raise UserError(_("The account for the Outlook Calendar service is not configured."))

        headers = {"content-type": "application/x-www-form-urlencoded"}
        data = {
            'refresh_token': self.microsoft_calendar_rtoken,
            'client_id': client_id,
            'client_secret': client_secret,
            'grant_type': 'refresh_token',
        }

        try:
            dummy, response, dummy = self.env['microsoft.service']._do_request(
                MICROSOFT_TOKEN_ENDPOINT, params=data, headers=headers, method='POST', preuri=''
            )
            ttl = response.get('expires_in')
            self.write({
                'microsoft_calendar_token': response.get('access_token'),
                'microsoft_calendar_token_validity': fields.Datetime.now() + timedelta(seconds=ttl),
            })
        except requests.HTTPError as error:
            if error.response.status_code in (400, 401):  # invalid grant or invalid client
                # Delete refresh token and make sure it's commited
                self.env.cr.rollback()
                self.write({
                    'microsoft_calendar_rtoken': False,
                    'microsoft_calendar_token': False,
                    'microsoft_calendar_token_validity': False,
                    'microsoft_calendar_sync_token': False,
                })
                self.env.cr.commit()
            error_key = error.response.json().get("error", "nc")
            error_msg = _(
                "An error occurred while generating the token. Your authorization code may be invalid or has already expired [%s]. "
                "You should check your Client ID and secret on the Microsoft Azure portal or try to stop and restart your calendar synchronisation.",
                error_key)
            raise UserError(error_msg)

    def _sync_microsoft_calendar(self):
        self.ensure_one()
        calendar_service = self.env["calendar.event"]._get_microsoft_service()
        full_sync = not bool(self.microsoft_calendar_sync_token)
        with microsoft_calendar_token(self) as token:
            try:
                events, next_sync_token = calendar_service.get_events(self.microsoft_calendar_sync_token, token=token)
            except InvalidSyncToken:
                events, next_sync_token = calendar_service.get_events(token=token)
                full_sync = True
        self.microsoft_calendar_sync_token = next_sync_token

        # Microsoft -> Odoo
        synced_events, synced_recurrences = self.env['calendar.event']._sync_microsoft2odoo(events) if events else (self.env['calendar.event'], self.env['calendar.recurrence'])

        # Odoo -> Microsoft
        recurrences = self.env['calendar.recurrence']._get_microsoft_records_to_sync(full_sync=full_sync)
        recurrences -= synced_recurrences
        recurrences._sync_odoo2microsoft()
        synced_events |= recurrences.calendar_event_ids

        events = self.env['calendar.event']._get_microsoft_records_to_sync(full_sync=full_sync)
        (events - synced_events)._sync_odoo2microsoft()

        return bool(events | synced_events) or bool(recurrences | synced_recurrences)

    @api.model
    def _sync_all_microsoft_calendar(self):
        """ Cron job """
        users = self.env['res.users'].search([('microsoft_calendar_rtoken', '!=', False)])
        for user in users:
            _logger.info("Calendar Synchro - Starting synchronization for %s", user)
            try:
                user.with_user(user).sudo()._sync_microsoft_calendar()
                self.env.cr.commit()
            except Exception as e:
                _logger.exception("[%s] Calendar Synchro - Exception : %s !", user, exception_to_unicode(e))
                self.env.cr.rollback()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_config_settings
from . import microsoft_sync
from . import calendar
from . import calendar_recurrence_rule
from . import res_users
from . import calendar_attendee

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
microsoft_calendar_account_reset,microsoft_calendar_account_reset_access_right,model_microsoft_calendar_account_reset,base.group_system,1,1,1,0

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70">
    <defs>
        <path id="icon-a" d="M4,5.35309892e-14 C36.4160122,9.87060235e-15 58.0836068,-3.97961823e-14 65,5.07020818e-14 C69,6.733808e-14 70,1 70,5 C70,43.0488877 70,62.4235458 70,65 C70,69 69,70 65,70 C61,70 9,70 4,70 C1,70 7.10542736e-15,69 7.10542736e-15,65 C7.25721566e-15,62.4676575 3.83358709e-14,41.8005206 3.60818146e-14,5 C-1.13686838e-13,1 1,5.75716207e-14 4,5.35309892e-14 Z"/>
        <linearGradient id="icon-c" x1="100%" x2="0%" y1="0%" y2="100%">
            <stop offset="0%" stop-color="#94B6C8"/>
            <stop offset="100%" stop-color="#6A9EBA"/>
        </linearGradient>
    </defs>
    <g fill="none" fill-rule="evenodd">
        <mask id="icon-b" fill="#fff">
            <use xlink:href="#icon-a"/>
        </mask>
        <g mask="url(#icon-b)">
            <rect width="70" height="70" fill="url(#icon-c)"/>
            <path fill="#FFF" fill-opacity=".383" d="M4,1.8 L65,1.8 C67.6666667,1.8 69.3333333,1.13333333 70,-0.2 C70,2.46666667 70,3.46666667 70,2.8 L1.10547097e-14,2.8 C-1.65952376e-14,3.46666667 -2.9161925e-14,2.46666667 -2.66453526e-14,-0.2 C0.666666667,1.13333333 2,1.8 4,1.8 Z" transform="matrix(1 0 0 -1 0 2.8)"/>
            <path fill="#393939" d="M4,53 C2,53 -7.10542736e-15,52.8543956 0,48.9230769 L2.12440903e-16,23.797368 L15,0 L54,0 L54,26.3118387 L57.7908529,26.3118387 L59,37.7115385 L49.1366936,53 L4,53 Z" opacity=".324" transform="translate(0 16)"/>
            <path fill="#000" fill-opacity=".383" d="M4,4 L65,4 C67.6666667,4 69.3333333,3 70,1 C70,3.66666667 70,5 70,5 L1.77635684e-15,5 C1.77635684e-15,5 1.77635684e-15,3.66666667 1.77635684e-15,1 C0.666666667,3 2,4 4,4 Z" transform="translate(0 65)"/>
            <path fill="#000" fill-rule="nonzero" d="M42.0955645,41.4315062 L42.0955645,28.7560864 L40.3833955,28.7560864 C40.2604632,29.4411054 40.0321745,30.0062307 39.6985283,30.4514643 C39.3648683,30.8967319 38.9521929,31.2477943 38.4604987,31.5046545 C37.986355,31.7615447 37.4507537,31.9413579 36.8536946,32.0440934 C36.2566271,32.1297334 35.6420039,32.1725451 35.0098218,32.1725307 L35.0098218,33.9192846 L39.8565751,33.9192846 L39.8565751,46.9685596 L41.0044957,46.9685596 C41.0029977,46.9790279 41.0014988,46.9895081 41,47 C41.6666667,47 42,47.3333333 42,48 C42,48.6666667 42,49 42,49 C40.6666667,49.6666667 40,50.3333333 40,51 C40,51.5412134 40.219684,52.3021107 40.6590519,53.282692 L15,53.282692 L15,24.0484615 L54.763358,24.0484615 L54.763358,37 C51.587786,37 49.6666667,37 49,37 C48.3333333,37 48,37.3333333 48,38 C48,38 48,38.3333333 48,39 C48,39.6666667 47.3333333,40 46,40 C44.6666667,40 43.6666667,40 43,40 C42.6985215,40 42.397043,40.4771687 42.0955645,41.4315062 Z M54.763358,23.0403846 L15,23.0403846 L15,18 L54.763358,18 L54.763358,23.0403846 Z M59.1120093,49.6163985 L60.7668484,50.585058 C60.9558409,50.6956709 61.043561,50.9244177 60.9788291,51.1357594 C60.5489654,52.5392703 59.8150519,53.807242 58.8551784,54.8603674 C58.7072273,55.022782 58.4680825,55.059636 58.279231,54.9491254 L56.6256345,53.9806234 C55.9298948,54.5851374 55.1300224,55.0541223 54.2662772,55.3639727 L54.2662772,57.3006617 C54.2662804,57.5219079 54.1150225,57.7135885 53.9021551,57.7620931 C52.5447646,58.0712105 51.0869171,58.0871979 49.661028,57.7623688 C49.4478436,57.7138157 49.2958963,57.5223205 49.2958963,57.3007404 L49.2958963,55.3639727 C48.4321428,55.0541404 47.6322673,54.5851537 46.936539,53.9806234 L45.2829425,54.9491254 C45.0940909,55.059636 44.8549462,55.022782 44.7069951,54.8603674 C43.7471604,53.807242 43.0132081,52.5392703 42.5833444,51.1357594 C42.5186125,50.9244571 42.6063326,50.6957102 42.795325,50.585058 L44.450203,49.6163985 C44.2839859,48.7026748 44.2839859,47.7658257 44.450203,46.852102 L42.7953639,45.8834424 C42.6063714,45.7728296 42.5186513,45.5440828 42.5833832,45.332741 C43.0132469,43.9292302 43.7471604,42.6612585 44.7070339,41.6081331 C44.8549851,41.4457185 45.0941298,41.4088644 45.2829813,41.5193751 L46.9365778,42.4878771 C47.6323176,41.8833631 48.4321899,41.4143782 49.2959351,41.1045278 L49.2959351,39.1677994 C49.2959319,38.9465532 49.4471898,38.7548726 49.6600572,38.706368 C51.0174478,38.3972506 52.4752952,38.3812631 53.9011843,38.7060923 C54.1143687,38.7546454 54.266316,38.9461406 54.266316,39.1677207 L54.266316,41.1044884 C55.1300695,41.4143207 55.929945,41.8833074 56.6256733,42.4878377 L58.2792698,41.5193357 C58.4681214,41.4088251 58.7072661,41.4456791 58.8552172,41.6080937 C59.8150519,42.6612191 60.5490043,43.9291908 60.9788679,45.3327017 C61.0435999,45.544004 60.9558797,45.7727509 60.7668873,45.8834031 L59.1120093,46.8520626 C59.2782268,47.7657995 59.2782268,48.7026616 59.1120093,49.6163985 Z M54.8876185,48.2342305 C54.8876185,46.497188 53.4940371,45.0839902 51.7811062,45.0839902 C50.0681752,45.0839902 48.6745938,46.497188 48.6745938,48.2342305 C48.6745938,49.9712731 50.0681752,51.3844709 51.7811062,51.3844709 C53.4940371,51.3844709 54.8876185,49.9712731 54.8876185,48.2342305 Z M25.3383523,20.6895722 C25.3383523,19.7564466 24.5910361,19 23.6691756,19 C22.7473152,19 22,19.7564466 22,20.6895722 C22,21.6226969 22.7473152,22.3791434 23.6691756,22.3791434 C24.5910361,22.3791434 25.3383523,21.6226969 25.3383523,20.6895722 Z M48.3383523,20.6895722 C48.3383523,19.7564466 47.5910361,19 46.6691756,19 C45.7473152,19 45,19.7564466 45,20.6895722 C45,21.6226969 45.7473152,22.3791434 46.6691756,22.3791434 C47.5910361,22.3791434 48.3383523,21.6226969 48.3383523,20.6895722 Z M26.4684862,38.4146042 C26.856901,38.3632372 27.2695974,38.3375505 27.7065763,38.3375419 C28.2082784,38.3375505 28.685711,38.4146128 29.1388753,38.56873 C29.5920244,38.705738 29.9804432,38.9283634 30.3041359,39.2366062 C30.6278103,39.5277389 30.8867565,39.8959263 31.0809766,40.3411707 C31.2751764,40.7693028 31.3722814,41.2659282 31.3722915,41.8310479 C31.3722814,42.3790542 31.2670842,42.8756796 31.0567001,43.3209262 C30.86248,43.749055 30.5954416,44.1172436 30.2555839,44.4254907 C29.9157069,44.716619 29.5191949,44.947806 29.0660468,45.1190547 C28.6128825,45.2731815 28.1354499,45.3502438 27.6337468,45.3502428 C26.4522968,45.3502438 25.5540767,44.9820564 24.9390823,44.2456783 C24.3240808,43.4921803 24.0003972,42.5246163 23.9680316,41.3429851 L21.9045489,41.3429851 C21.8883645,42.2848667 22.009746,43.1239918 22.2686932,43.8603645 C22.5438228,44.5967437 22.9322426,45.221807 23.4339538,45.7355554 C23.9356609,46.2321829 24.5425662,46.6174956 25.2546737,46.8914955 C25.966772,47.1483714 26.7597961,47.2768082 27.6337468,47.2768093 C28.4429491,47.2768082 29.2036045,47.156934 29.915716,46.9171833 C30.6439946,46.6774327 31.2751764,46.3263705 31.8092643,45.8639933 C32.343331,45.4016194 32.7641185,44.8279318 33.0716299,44.1429281 C33.3953013,43.4408047 33.5571425,42.635929 33.5571547,41.7282978 C33.5571425,40.6323023 33.2981963,39.6818635 32.780315,38.8769803 C32.2785937,38.0721121 31.501755,37.5497989 30.4497938,37.3100397 L30.4497938,37.2586641 C31.1133351,36.9332983 31.6716884,36.4537981 32.1248557,35.8201615 C32.5941851,35.1694222 32.8288558,34.4416088 32.828867,33.6367202 C32.8288558,32.797607 32.69129,32.0697936 32.4161706,31.4532779 C32.1572132,30.8367933 31.7930699,30.3316054 31.3237385,29.937713 C30.8543879,29.5438538 30.2960346,29.2527287 29.6486766,29.0643354 C29.0174867,28.8588534 28.3296598,28.7561033 27.5851948,28.7560851 C26.7274274,28.7561033 25.966772,28.9016664 25.3032256,29.1927733 C24.6558555,29.4839167 24.1136865,29.8863545 23.6767167,30.4000879 C23.2397419,30.9138557 22.899874,31.5303563 22.6571131,32.2495921 C22.4305335,32.9688579 22.3010609,33.765171 22.2686932,34.6385346 L24.332176,34.6385346 C24.3321719,34.1076707 24.3969093,33.6024827 24.5263859,33.1229686 C24.6558555,32.6434824 24.8500654,32.2239193 25.1090157,31.8642784 C25.3841422,31.5046685 25.7240101,31.222106 26.1286194,31.0165897 C26.5332175,30.7939804 27.0187423,30.6826676 27.5851948,30.6826515 C28.4591324,30.6826676 29.1874202,30.9309809 29.7700581,31.4275901 C30.3526797,31.9242316 30.6439946,32.652045 30.6440038,33.6110324 C30.6439946,34.1076707 30.5549808,34.5443589 30.3769643,34.9210972 C30.1989296,35.2807349 29.9561677,35.5889853 29.6486766,35.8458493 C29.3573536,36.0856106 29.0093946,36.2739857 28.6047975,36.4109754 C28.2163705,36.5308604 27.8036742,36.5907986 27.3667084,36.5907879 C27.2210444,36.5907986 27.0753874,36.5907986 26.9297356,36.5907879 C26.7840726,36.5907986 26.6303224,36.582236 26.4684862,36.5651001 L26.4684862,38.4146042 Z M13.808,57.508 L12.524,57.508 L12.524,56.716 L16.034,56.716 L16.034,57.508 L14.75,57.508 L14.75,61 L13.808,61 L13.808,57.508 Z M16.526,56.716 L17.852,56.716 L18.854,59.662 L18.866,59.662 L19.814,56.716 L21.14,56.716 L21.14,61 L20.258,61 L20.258,57.964 L20.246,57.964 L19.196,61 L18.47,61 L17.42,57.994 L17.408,57.994 L17.408,61 L16.526,61 L16.526,56.716 Z" opacity=".3"/>
            <path fill="#FFF" fill-rule="nonzero" d="M42.0955645,39.4315062 L42.0955645,26.7560864 L40.3833955,26.7560864 C40.2604632,27.4411054 40.0321745,28.0062307 39.6985283,28.4514643 C39.3648683,28.8967319 38.9521929,29.2477943 38.4604987,29.5046545 C37.986355,29.7615447 37.4507537,29.9413579 36.8536946,30.0440934 C36.2566271,30.1297334 35.6420039,30.1725451 35.0098218,30.1725307 L35.0098218,31.9192846 L39.8565751,31.9192846 L39.8565751,44.9685596 L41.0044957,44.9685596 C41.0029977,44.9790279 41.0014988,44.9895081 41,45 C41.6666667,45 42,45.3333333 42,46 C42,46.6666667 42,47 42,47 C40.6666667,47.6666667 40,48.3333333 40,49 C40,49.5412134 40.219684,50.3021107 40.6590519,51.282692 L15,51.282692 L15,22.0484615 L54.763358,22.0484615 L54.763358,35 C51.587786,35 49.6666667,35 49,35 C48.3333333,35 48,35.3333333 48,36 C48,36 48,36.3333333 48,37 C48,37.6666667 47.3333333,38 46,38 C44.6666667,38 43.6666667,38 43,38 C42.6985215,38 42.397043,38.4771687 42.0955645,39.4315062 Z M54.763358,21.0403846 L15,21.0403846 L15,16 L54.763358,16 L54.763358,21.0403846 Z M59.1120093,47.6163985 L60.7668484,48.585058 C60.9558409,48.6956709 61.043561,48.9244177 60.9788291,49.1357594 C60.5489654,50.5392703 59.8150519,51.807242 58.8551784,52.8603674 C58.7072273,53.022782 58.4680825,53.059636 58.279231,52.9491254 L56.6256345,51.9806234 C55.9298948,52.5851374 55.1300224,53.0541223 54.2662772,53.3639727 L54.2662772,55.3006617 C54.2662804,55.5219079 54.1150225,55.7135885 53.9021551,55.7620931 C52.5447646,56.0712105 51.0869171,56.0871979 49.661028,55.7623688 C49.4478436,55.7138157 49.2958963,55.5223205 49.2958963,55.3007404 L49.2958963,53.3639727 C48.4321428,53.0541404 47.6322673,52.5851537 46.936539,51.9806234 L45.2829425,52.9491254 C45.0940909,53.059636 44.8549462,53.022782 44.7069951,52.8603674 C43.7471604,51.807242 43.0132081,50.5392703 42.5833444,49.1357594 C42.5186125,48.9244571 42.6063326,48.6957102 42.795325,48.585058 L44.450203,47.6163985 C44.2839859,46.7026748 44.2839859,45.7658257 44.450203,44.852102 L42.7953639,43.8834424 C42.6063714,43.7728296 42.5186513,43.5440828 42.5833832,43.332741 C43.0132469,41.9292302 43.7471604,40.6612585 44.7070339,39.6081331 C44.8549851,39.4457185 45.0941298,39.4088644 45.2829813,39.5193751 L46.9365778,40.4878771 C47.6323176,39.8833631 48.4321899,39.4143782 49.2959351,39.1045278 L49.2959351,37.1677994 C49.2959319,36.9465532 49.4471898,36.7548726 49.6600572,36.706368 C51.0174478,36.3972506 52.4752952,36.3812631 53.9011843,36.7060923 C54.1143687,36.7546454 54.266316,36.9461406 54.266316,37.1677207 L54.266316,39.1044884 C55.1300695,39.4143207 55.929945,39.8833074 56.6256733,40.4878377 L58.2792698,39.5193357 C58.4681214,39.4088251 58.7072661,39.4456791 58.8552172,39.6080937 C59.8150519,40.6612191 60.5490043,41.9291908 60.9788679,43.3327017 C61.0435999,43.544004 60.9558797,43.7727509 60.7668873,43.8834031 L59.1120093,44.8520626 C59.2782268,45.7657995 59.2782268,46.7026616 59.1120093,47.6163985 Z M54.8876185,46.2342305 C54.8876185,44.497188 53.4940371,43.0839902 51.7811062,43.0839902 C50.0681752,43.0839902 48.6745938,44.497188 48.6745938,46.2342305 C48.6745938,47.9712731 50.0681752,49.3844709 51.7811062,49.3844709 C53.4940371,49.3844709 54.8876185,47.9712731 54.8876185,46.2342305 Z M25.3383523,18.6895722 C25.3383523,17.7564466 24.5910361,17 23.6691756,17 C22.7473152,17 22,17.7564466 22,18.6895722 C22,19.6226969 22.7473152,20.3791434 23.6691756,20.3791434 C24.5910361,20.3791434 25.3383523,19.6226969 25.3383523,18.6895722 Z M48.3383523,18.6895722 C48.3383523,17.7564466 47.5910361,17 46.6691756,17 C45.7473152,17 45,17.7564466 45,18.6895722 C45,19.6226969 45.7473152,20.3791434 46.6691756,20.3791434 C47.5910361,20.3791434 48.3383523,19.6226969 48.3383523,18.6895722 Z M26.4684862,36.4146042 C26.856901,36.3632372 27.2695974,36.3375505 27.7065763,36.3375419 C28.2082784,36.3375505 28.685711,36.4146128 29.1388753,36.56873 C29.5920244,36.705738 29.9804432,36.9283634 30.3041359,37.2366062 C30.6278103,37.5277389 30.8867565,37.8959263 31.0809766,38.3411707 C31.2751764,38.7693028 31.3722814,39.2659282 31.3722915,39.8310479 C31.3722814,40.3790542 31.2670842,40.8756796 31.0567001,41.3209262 C30.86248,41.749055 30.5954416,42.1172436 30.2555839,42.4254907 C29.9157069,42.716619 29.5191949,42.947806 29.0660468,43.1190547 C28.6128825,43.2731815 28.1354499,43.3502438 27.6337468,43.3502428 C26.4522968,43.3502438 25.5540767,42.9820564 24.9390823,42.2456783 C24.3240808,41.4921803 24.0003972,40.5246163 23.9680316,39.3429851 L21.9045489,39.3429851 C21.8883645,40.2848667 22.009746,41.1239918 22.2686932,41.8603645 C22.5438228,42.5967437 22.9322426,43.221807 23.4339538,43.7355554 C23.9356609,44.2321829 24.5425662,44.6174956 25.2546737,44.8914955 C25.966772,45.1483714 26.7597961,45.2768082 27.6337468,45.2768093 C28.4429491,45.2768082 29.2036045,45.156934 29.915716,44.9171833 C30.6439946,44.6774327 31.2751764,44.3263705 31.8092643,43.8639933 C32.343331,43.4016194 32.7641185,42.8279318 33.0716299,42.1429281 C33.3953013,41.4408047 33.5571425,40.635929 33.5571547,39.7282978 C33.5571425,38.6323023 33.2981963,37.6818635 32.780315,36.8769803 C32.2785937,36.0721121 31.501755,35.5497989 30.4497938,35.3100397 L30.4497938,35.2586641 C31.1133351,34.9332983 31.6716884,34.4537981 32.1248557,33.8201615 C32.5941851,33.1694222 32.8288558,32.4416088 32.828867,31.6367202 C32.8288558,30.797607 32.69129,30.0697936 32.4161706,29.4532779 C32.1572132,28.8367933 31.7930699,28.3316054 31.3237385,27.937713 C30.8543879,27.5438538 30.2960346,27.2527287 29.6486766,27.0643354 C29.0174867,26.8588534 28.3296598,26.7561033 27.5851948,26.7560851 C26.7274274,26.7561033 25.966772,26.9016664 25.3032256,27.1927733 C24.6558555,27.4839167 24.1136865,27.8863545 23.6767167,28.4000879 C23.2397419,28.9138557 22.899874,29.5303563 22.6571131,30.2495921 C22.4305335,30.9688579 22.3010609,31.765171 22.2686932,32.6385346 L24.332176,32.6385346 C24.3321719,32.1076707 24.3969093,31.6024827 24.5263859,31.1229686 C24.6558555,30.6434824 24.8500654,30.2239193 25.1090157,29.8642784 C25.3841422,29.5046685 25.7240101,29.222106 26.1286194,29.0165897 C26.5332175,28.7939804 27.0187423,28.6826676 27.5851948,28.6826515 C28.4591324,28.6826676 29.1874202,28.9309809 29.7700581,29.4275901 C30.3526797,29.9242316 30.6439946,30.652045 30.6440038,31.6110324 C30.6439946,32.1076707 30.5549808,32.5443589 30.3769643,32.9210972 C30.1989296,33.2807349 29.9561677,33.5889853 29.6486766,33.8458493 C29.3573536,34.0856106 29.0093946,34.2739857 28.6047975,34.4109754 C28.2163705,34.5308604 27.8036742,34.5907986 27.3667084,34.5907879 C27.2210444,34.5907986 27.0753874,34.5907986 26.9297356,34.5907879 C26.7840726,34.5907986 26.6303224,34.582236 26.4684862,34.5651001 L26.4684862,36.4146042 Z M13.808,55.508 L12.524,55.508 L12.524,54.716 L16.034,54.716 L16.034,55.508 L14.75,55.508 L14.75,59 L13.808,59 L13.808,55.508 Z M16.526,54.716 L17.852,54.716 L18.854,57.662 L18.866,57.662 L19.814,54.716 L21.14,54.716 L21.14,59 L20.258,59 L20.258,55.964 L20.246,55.964 L19.196,59 L18.47,59 L17.42,55.994 L17.408,55.994 L17.408,59 L16.526,59 L16.526,54.716 Z"/>
        </g>
    </g>
</svg>

```

## File: static\src\js\microsoft_calendar.js

```javascript
odoo.define('microsoft_calendar.CalendarView', function (require) {
"use strict";

var core = require('web.core');
var Dialog = require('web.Dialog');
var framework = require('web.framework');
const CalendarView = require('calendar.CalendarView');
const CalendarRenderer = require('calendar.CalendarRenderer');
const CalendarController = require('calendar.CalendarController');
const CalendarModel = require('calendar.CalendarModel');
const viewRegistry = require('web.view_registry');
const session = require('web.session');

var _t = core._t;

const MicrosoftCalendarModel = CalendarModel.include({

    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this.microsoft_is_sync = true;
        this.microsoft_pending_sync = false;
    },

    /**
     * @override
     */
    __get: function () {
        var result = this._super.apply(this, arguments);
        result.microsoft_is_sync = this.microsoft_is_sync;
        return result;
    },

    /**
     * @override
     * @returns {Promise}
     */
    async _loadCalendar() {
        const _super = this._super.bind(this);
        // When the calendar synchronization takes some time, prevents retriggering the sync while navigating the calendar.
        if (this.microsoft_pending_sync) {
            return _super(...arguments);
        }
        try {
            await Promise.race([
                new Promise(resolve => setTimeout(resolve, 1000)),
                this._syncMicrosoftCalendar(true)
            ]);
        } catch (error) {
            if (error.event) {
                error.event.preventDefault();
            }
            console.error("Could not synchronize Outlook events now.", error);
            this.microsoft_pending_sync = false;
        }
        return _super(...arguments);
    },

    _syncMicrosoftCalendar(shadow = false) {
        var self = this;
        this.microsoft_pending_sync = true;
        return this._rpc({
            route: '/microsoft_calendar/sync_data',
            params: {
                model: this.modelName,
                fromurl: window.location.href,
            }
        }, {shadow}).then(function (result) {
            if (result.status === "need_config_from_admin" || result.status === "need_auth") {
                self.microsoft_is_sync = false;
            } else if (result.status === "no_new_event_from_microsoft" || result.status === "need_refresh") {
                self.microsoft_is_sync = true;
            }
            self.microsoft_pending_sync = false;
            return result
        });
    },

    archiveRecords: function (ids, model) {
        return this._rpc({
            model: model,
            method: 'action_archive',
            args: [ids],
            context: session.user_context,
        });
    },
});

const MicrosoftCalendarController = CalendarController.include({
    custom_events: _.extend({}, CalendarController.prototype.custom_events, {
        syncMicrosoftCalendar: '_onSyncMicrosoftCalendar',
        archiveRecord: '_onArchiveRecord',
    }),


    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Try to sync the calendar with Microsoft Calendar. According to the result
     * from Microsoft API, this function may require an action of the user by the
     * mean of a dialog.
     *
     * @private
     * @returns {OdooEvent} event
     */
    _onSyncMicrosoftCalendar: function (event) {
        var self = this;

        return this.model._syncMicrosoftCalendar().then(function (o) {
            if (o.status === "need_auth") {
                Dialog.alert(self, _t("You will be redirected to Outlook to authorize the access to your calendar."), {
                    confirm_callback: function() {
                        framework.redirect(o.url);
                    },
                    title: _t('Redirection'),
                });
            } else if (o.status === "need_config_from_admin") {
                if (!_.isUndefined(o.action) && parseInt(o.action)) {
                    Dialog.confirm(self, _t("The Outlook Synchronization needs to be configured before you can use it, do you want to do it now?"), {
                        confirm_callback: function() {
                            self.do_action(o.action);
                        },
                        title: _t('Configuration'),
                    });
                } else {
                    Dialog.alert(self, _t("An administrator needs to configure Outlook Synchronization before you can use it!"), {
                        title: _t('Configuration'),
                    });
                }
            } else if (o.status === "need_refresh") {
                self.reload();
            }
        }).then(event.data.on_always, event.data.on_always);
    },

    _onArchiveRecord: function (ev) {
        var self = this;
        Dialog.confirm(this, _t("Are you sure you want to archive this record ?"), {
            confirm_callback: function () {
                self.model.archiveRecords([ev.data.id], self.modelName).then(function () {
                    self.reload();
                });
            }
        });
    },
});

const MicrosoftCalendarRenderer = CalendarRenderer.include({
    events: _.extend({}, CalendarRenderer.prototype.events, {
        'click .o_microsoft_sync_button': '_onSyncMicrosoftCalendar',
    }),
    custom_events: _.extend({}, CalendarRenderer.prototype.custom_events, {
        archive_event: '_onArchiveEvent',
    }),

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Adds the Sync with Outlook button in the sidebar
     *
     * @private
     */
    _initSidebar: function () {
        var self = this;
        this._super.apply(this, arguments);
        this.$microsoftButton = $();
        if (this.model === "calendar.event") {
            if (this.state.microsoft_is_sync) {
                this.$microsoftButton = $('<span/>', {html: _t("Synched with Outlook")})
                                .addClass('o_microsoft_sync badge badge-pill badge-success')
                                .prepend($('<i/>', {class: "fa mr-2 fa-check"}))
                                .appendTo(self.$sidebar);
            } else {
                this.$microsoftButton = $('<button/>', {type: 'button', html: _t("Sync with <b>Outlook</b>")})
                                .addClass('o_microsoft_sync_button oe_button btn btn-secondary')
                                .appendTo(self.$sidebar);
            }
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Requests to sync the calendar with Microsoft Calendar
     *
     * @private
     */
    _onSyncMicrosoftCalendar: function () {
        var self = this;
        var context = this.getSession().user_context;
        this.$microsoftButton.prop('disabled', true);
        this.trigger_up('syncMicrosoftCalendar', {
            on_always: function () {
                self.$microsoftButton.prop('disabled', false);
            },
        });
    },

    _onArchiveEvent: function (ev) {
        this._unselectEvent();
        this.trigger_up('archiveRecord', {id: parseInt(ev.data.id, 10)});
    },
});

return {
    MicrosoftCalendarController,
    MicrosoftCalendarModel,
    MicrosoftCalendarRenderer,
};

});

```

## File: static\src\js\microsoft_calendar_popover.js

```javascript
odoo.define('microsoft_calendar.MicrosoftCalendarPopover', function(require) {
    "use strict";

    const CalendarPopover = require('web.CalendarPopover');

    const MicrosoftCalendarPopover = CalendarPopover.include({
        events: _.extend({}, CalendarPopover.prototype.events, {
            'click .o_cw_popover_archive_m': '_onClickPopoverArchive',
        }),

        /**
         * We only want one 'Archive' button in the popover
         * so if Google Sync is also active, it takes precedence
         * over this popvoer.
         */
        isMEventSyncedAndArchivable() {
            if (this.event.extendedProps.record.google_id === undefined) {
                return this.event.extendedProps.record.microsoft_id;
            }
            return !this.event.extendedProps.record.google_id && this.event.extendedProps.record.microsoft_id
        },

        isEventDeletable() {
            return !this.isMEventSyncedAndArchivable() && this._super();
        },

        _onClickPopoverArchive: function (ev) {
            ev.preventDefault();
            this.trigger_up('archive_event', {id: this.event.id});
        },
    });

    return MicrosoftCalendarPopover;
});

```

## File: static\src\xml\microsoft_calendar_popover.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-extend="Calendar.attendee.status.popover">
        <t t-jquery=".o_cw_popover_edit" t-operation="after">
            <a t-if="typeof widget.isMEventSyncedAndArchivable === 'function' and widget.isMEventSyncedAndArchivable() and widget.isEventDetailsVisible()" href="#" class="btn btn-secondary o_cw_popover_archive_m">Archive</a>
        </t>
    </t>
</templates>

```

## File: utils\event_id_storage.py

```python
IDS_SEPARATOR = ':'

def combine_ids(ms_id, ms_uid):
    if not ms_id:
        return False
    return ms_id + IDS_SEPARATOR + (ms_uid if ms_uid else '')

def split_ids(value):
    ids = value.split(IDS_SEPARATOR)
    return tuple(ids) if len(ids) > 1 and ids[1] else (ids[0], False)

```

## File: utils\microsoft_calendar.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import requests
import json
import logging

from werkzeug import urls

from odoo import fields
from odoo.addons.microsoft_calendar.utils.microsoft_event import MicrosoftEvent
from odoo.addons.microsoft_account.models.microsoft_service import TIMEOUT, RESOURCE_NOT_FOUND_STATUSES

_logger = logging.getLogger(__name__)

def requires_auth_token(func):
    def wrapped(self, *args, **kwargs):
        if not kwargs.get('token'):
            raise AttributeError("An authentication token is required")
        return func(self, *args, **kwargs)
    return wrapped

class InvalidSyncToken(Exception):
    pass

# In Outlook, an event can be:
# - a 'singleInstance' event,
# - a 'serie master' which contains all the information about an event reccurrence such as
# - an 'occurrence' which is an event from a reccurrence (serie) that follows this reccurrence
# - an 'exception' which is an event from a reccurrence (serie) but some differences with the reccurrence template (could be
#   the name, the day of occurrence, ...)
#
#  All these kinds of events are identified by:
#  - a event ID (id) which is specific to an Outlook calendar.
#  - a global event ID (iCalUId) which is common to all Outlook calendars containing this event.
#
#  - 'singleInstance' and 'serie master' events are retrieved through the end-point `/v1.0/me/calendarView/delta` which provides
#  the last modified/deleted items since the last sync (or all of these items at the first time).
#  - 'occurrence' and 'exception' events are retrieved through the end-point `/v1.0/me/events/{serieMaster.id}/instances`,
#  using the corresponding serie master ID.

class MicrosoftCalendarService():

    def __init__(self, microsoft_service):
        self.microsoft_service = microsoft_service

    @requires_auth_token
    def _get_events_from_paginated_url(self, url, token=None, params=None, timeout=TIMEOUT):
        """
        Get a list of events from a paginated URL.
        Each page contains a link to the next page, so loop over all the pages to get all the events.
        """
        headers = {
            'Content-type': 'application/json',
            'Authorization': 'Bearer %s' % token,
            'Prefer': 'outlook.body-content-type="text", odata.maxpagesize=50'
        }
        if not params:
            params = {
                'startDateTime': fields.Datetime.subtract(fields.Datetime.now(), years=2).strftime("%Y-%m-%dT00:00:00Z"),
                'endDateTime': fields.Datetime.add(fields.Datetime.now(), years=2).strftime("%Y-%m-%dT00:00:00Z"),
            }

        # get the first page of events
        _, data, _ = self.microsoft_service._do_request(
            url, params, headers, method='GET', timeout=timeout
        )

        # and then, loop on other pages to get all the events
        events = data.get('value', [])
        next_page_token = data.get('@odata.nextLink')
        while next_page_token:
            _, data, _ = self.microsoft_service._do_request(
                next_page_token, {}, headers, preuri='', method='GET', timeout=timeout
            )
            next_page_token = data.get('@odata.nextLink')
            events += data.get('value', [])

        token_url = data.get('@odata.deltaLink')
        next_sync_token = urls.url_parse(token_url).decode_query().get('$deltatoken', False) if token_url else None

        return events, next_sync_token

    @requires_auth_token
    def _get_events_delta(self, sync_token=None, token=None, timeout=TIMEOUT):
        """
        Get a set of events that have been added, deleted or updated in a time range.
        See: https://docs.microsoft.com/en-us/graph/api/event-delta?view=graph-rest-1.0&tabs=http
        """
        url = "/v1.0/me/calendarView/delta"
        params = {'$deltatoken': sync_token} if sync_token else None

        try:
            events, next_sync_token = self._get_events_from_paginated_url(
                url, params=params, token=token, timeout=timeout)
        except requests.HTTPError as e:
            if e.response.status_code == 410 and 'fullSyncRequired' in str(e.response.content) and sync_token:
                # retry with a full sync
                return self._get_events_delta(token=token, timeout=timeout)
            raise e

        # event occurrences (from a recurrence) are retrieved separately to get all their info,
        # # and mainly the iCalUId attribute which is not provided by the 'get_delta' api end point
        events = [e for e in events if e.get('type') != 'occurrence']

        return MicrosoftEvent(events), next_sync_token

    @requires_auth_token
    def _get_occurrence_details(self, serieMasterId, token=None, timeout=TIMEOUT):
        """
        Get all occurrences details from a serie master.
        See: https://docs.microsoft.com/en-us/graph/api/event-list-instances?view=graph-rest-1.0&tabs=http
        """
        url = f"/v1.0/me/events/{serieMasterId}/instances"

        events, _ = self._get_events_from_paginated_url(url, token=token, timeout=timeout)
        return MicrosoftEvent(events)

    @requires_auth_token
    def get_events(self, sync_token=None, token=None, timeout=TIMEOUT):
        """
        Retrieve all the events that have changed (added/updated/removed) from Microsoft Outlook.
        This is done in 2 steps:
        1) get main changed events (so single events and serie masters)
        2) get occurrences linked to a serie masters (to retrieve all needed details such as iCalUId)
        """
        events, next_sync_token = self._get_events_delta(sync_token=sync_token, token=token, timeout=timeout)

        # get occurences details for all serie masters
        for master in filter(lambda e: e.type == 'seriesMaster', events):
            events |= self._get_occurrence_details(master.id, token=token, timeout=timeout)

        return events, next_sync_token

    @requires_auth_token
    def insert(self, values, token=None, timeout=TIMEOUT):
        url = "/v1.0/me/calendar/events"
        headers = {'Content-type': 'application/json', 'Authorization': 'Bearer %s' % token}
        _dummy, data, _dummy = self.microsoft_service._do_request(url, json.dumps(values), headers, method='POST', timeout=timeout)

        return data['id'], data['iCalUId']

    @requires_auth_token
    def patch(self, event_id, values, token=None, timeout=TIMEOUT):
        url = "/v1.0/me/calendar/events/%s" % event_id
        headers = {'Content-type': 'application/json', 'Authorization': 'Bearer %s' % token}
        try:
            status, _dummy, _dummy = self.microsoft_service._do_request(url, json.dumps(values), headers, method='PATCH', timeout=timeout)
        except requests.HTTPError:
            _logger.info("Microsoft event %s has not been updated", event_id)
            return False

        return status not in RESOURCE_NOT_FOUND_STATUSES

    @requires_auth_token
    def delete(self, event_id, token=None, timeout=TIMEOUT):
        url = "/v1.0/me/calendar/events/%s" % event_id
        headers = {'Authorization': 'Bearer %s' % token}
        params = {}
        try:
            status, _dummy, _dummy = self.microsoft_service._do_request(url, params, headers=headers, method='DELETE', timeout=timeout)
        except requests.HTTPError as e:
            # For some unknown reason Microsoft can also return a 403 response when the event is already cancelled.
            status = e.response.status_code
            if status in (410, 403):
                _logger.info("Microsoft event %s was already deleted", event_id)
            else:
                raise e

        return status not in RESOURCE_NOT_FOUND_STATUSES

    @requires_auth_token
    def answer(self, event_id, answer, values, token=None, timeout=TIMEOUT):
        url = "/v1.0/me/calendar/events/%s/%s" % (event_id, answer)
        headers = {'Content-type': 'application/json', 'Authorization': 'Bearer %s' % token}
        status, _dummy, _dummy = self.microsoft_service._do_request(url, json.dumps(values), headers, method='POST', timeout=timeout)

        return status not in RESOURCE_NOT_FOUND_STATUSES


    #####################################
    ##  MANAGE CONNEXION TO MICROSOFT  ##
    #####################################

    def is_authorized(self, user):
        return bool(user.sudo().microsoft_calendar_rtoken)

    def _get_calendar_scope(self):
        return 'offline_access openid Calendars.ReadWrite'

    def _microsoft_authentication_url(self, from_url='http://www.odoo.com'):
        return self.microsoft_service._get_authorize_uri(from_url, service='calendar', scope=self._get_calendar_scope())

    def _can_authorize_microsoft(self, user):
        return user.has_group('base.group_erp_manager')

```

## File: utils\microsoft_event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.api import model
from typing import Iterator, Mapping
from collections import abc
from odoo.tools import ReadonlyDict
from odoo.addons.microsoft_calendar.utils.event_id_storage import combine_ids


class MicrosoftEvent(abc.Set):
    """
    This helper class holds the values of a Microsoft event.
    Inspired by Odoo recordset, one instance can be a single Microsoft event or a
    (immutable) set of Microsoft events.
    All usual set operations are supported (union, intersection, etc).

    :param iterable: iterable of MicrosoftCalendar instances or iterable of dictionnaries
    """

    def __init__(self, iterable=()):
        _events = {}
        for item in iterable:
            if isinstance(item, self.__class__):
                _events[item.id] = item._events[item.id]
            elif isinstance(item, Mapping):
                _events[item.get('id')] = item
            else:
                raise ValueError("Only %s or iterable of dict are supported" % self.__class__.__name__)
        self._events = ReadonlyDict(_events)

    def __iter__(self) -> Iterator['MicrosoftEvent']:
        return iter(MicrosoftEvent([vals]) for vals in self._events.values())

    def __contains__(self, microsoft_event):
        return microsoft_event.id in self._events

    def __len__(self):
        return len(self._events)

    def __bool__(self):
        return bool(self._events)

    def __getattr__(self, name):
        # ensure_one
        try:
            event, = self._events.keys()
        except ValueError:
            raise ValueError("Expected singleton: %s" % self)
        event_id = list(self._events.keys())[0]
        return self._events[event_id].get(name)

    def __repr__(self):
        return '%s%s' % (self.__class__.__name__, self.ids)

    @property
    def ids(self):
        """
        Use 'id' to return an event identifier which is specific to a calendar
        """
        return tuple(e.id for e in self)

    def microsoft_ids(self):
        return tuple(e.id for e in self)

    @property
    def uids(self):
        """
        Use 'iCalUid' to return an identifier which is unique accross all calendars
        """
        return tuple(e.iCalUId for e in self)

    def odoo_id(self, env):
        return self._odoo_id

    def _meta_odoo_id(self, microsoft_guid):
        """Returns the Odoo id stored in the Microsoft Event metadata.
        This id might not actually exists in the database.
        """
        return None

    @property
    def odoo_ids(self):
        """
        Get the list of Odoo event ids already mapped with Outlook events (self)
        """
        return tuple(e._odoo_id for e in self if e._odoo_id)

    def _load_odoo_ids_from_db(self, env, force_model=None):
        """
        Map Microsoft events to existing Odoo events:
        1) extract unmapped events only,
        2) match Odoo events and Outlook events which have both a ICalUId set,
        3) match remaining events,
        Returns the list of mapped events
        """
        mapped_events = [e.id for e in self if e._odoo_id]

        # avoid mapping events if they are already all mapped
        if len(self) == len(mapped_events):
            return self

        unmapped_events = self.filter(lambda e: e.id not in mapped_events)

        model_env = force_model if force_model is not None else self._get_model(env)
        odoo_events = model_env.with_context(active_test=False).search([
            '|',
            ('ms_universal_event_id', "in", unmapped_events.uids),
            ('ms_organizer_event_id', "in", unmapped_events.ids)
        ]).with_env(env)

        # 1. try to match unmapped events with Odoo events using their iCalUId
        unmapped_events_with_uids = unmapped_events.filter(lambda e: e.iCalUId)
        odoo_events_with_uids = odoo_events.filtered(lambda e: e.ms_universal_event_id)
        mapping = {e.ms_universal_event_id: e.id for e in odoo_events_with_uids}

        for ms_event in unmapped_events_with_uids:
            odoo_id = mapping.get(ms_event.iCalUId)
            if odoo_id:
                ms_event._events[ms_event.id]['_odoo_id'] = odoo_id
                mapped_events.append(ms_event.id)

        # 2. try to match unmapped events with Odoo events using their id
        unmapped_events = self.filter(lambda e: e.id not in mapped_events)
        mapping = {e.ms_organizer_event_id: e for e in odoo_events}

        for ms_event in unmapped_events:
            odoo_event = mapping.get(ms_event.id)
            if odoo_event:
                ms_event._events[ms_event.id]['_odoo_id'] = odoo_event.id
                mapped_events.append(ms_event.id)

                # don't forget to also set the global event ID on the Odoo event to ease
                # and improve reliability of future mappings
                odoo_event.write({
                    'microsoft_id': combine_ids(ms_event.id, ms_event.iCalUId),
                    'need_sync_m': False,
                })

        return self.filter(lambda e: e.id in mapped_events)

    def owner_id(self, env):
        """
        Indicates who is the owner of an event (i.e the organizer of the event).

        There are several possible cases:
        1) the current Odoo user is the organizer of the event according to Outlook event, so return his id.
        2) the current Odoo user is NOT the organizer and:
           2.1) we are able to find a Odoo user using the Outlook event organizer email address and we use his id,
           2.2) we are NOT able to find a Odoo user matching the organizer email address and we return False, meaning
                that no Odoo user will be able to modify this event. All modifications will be done from Outlook.
        """
        if self.isOrganizer:
            return env.user.id
        if self.organizer.get('emailAddress') and self.organizer.get('emailAddress').get('address'):
            # Warning: In Microsoft: 1 email = 1 user; but in Odoo several users might have the same email
            user = env['res.users'].search([('email', '=', self.organizer.get('emailAddress').get('address'))], limit=1)
            return user.id if user else False
        return False

    def filter(self, func) -> 'MicrosoftEvent':
        return MicrosoftEvent(e for e in self if func(e))

    def is_recurrence(self):
        return self.type == 'seriesMaster'

    def is_recurrent(self):
        return bool(self.seriesMasterId or self.is_recurrence())

    def is_recurrent_not_master(self):
        return bool(self.seriesMasterId)

    def get_recurrence(self):
        if not self.recurrence:
            return {}
        pattern = self.recurrence['pattern']
        range = self.recurrence['range']
        end_type_dict = {
            'endDate': 'end_date',
            'noEnd': 'forever',
            'numbered': 'count',
        }
        type_dict = {
            'absoluteMonthly': 'monthly',
            'relativeMonthly': 'monthly',
            'absoluteYearly': 'yearly',
            'relativeYearly': 'yearly',
        }
        index_dict = {
            'first': '1',
            'second': '2',
            'third': '3',
            'fourth': '4',
            'last': '-1',
        }
        rrule_type = type_dict.get(pattern['type'], pattern['type'])
        interval = pattern['interval']
        if rrule_type == 'yearly':
            interval *= 12
        result = {
            'rrule_type': rrule_type,
            'end_type': end_type_dict.get(range['type'], False),
            'interval': interval,
            'count': range['numberOfOccurrences'],
            'day': pattern['dayOfMonth'],
            'byday': index_dict.get(pattern['index'], False),
            'until': range['type'] == 'endDate' and range['endDate'],
        }

        month_by_dict = {
            'absoluteMonthly': 'date',
            'relativeMonthly': 'day',
            'absoluteYearly': 'date',
            'relativeYearly': 'day',
        }
        month_by = month_by_dict.get(pattern['type'], False)
        if month_by:
            result['month_by'] = month_by

        # daysOfWeek contains the full name of the day, the fields contain the first 2 letters (mo, tu, etc)
        week_days = [x[:2] for x in pattern.get('daysOfWeek', [])]
        for week_day in ['mo', 'tu', 'we', 'th', 'fr', 'sa', 'su']:
            result[week_day] = week_day in week_days
        if week_days:
            result['weekday'] = week_days[0].upper()
        return result

    def is_cancelled(self):
        return bool(self.isCancelled) or self.is_removed()

    def is_removed(self):
        return self.__getattr__('@removed') and self.__getattr__('@removed').get('reason') == 'deleted'

    def is_recurrence_outlier(self):
        return self.type == "exception"

    def cancelled(self):
        return self.filter(lambda e: e.is_cancelled())

    def match_with_odoo_events(self, env) -> 'MicrosoftEvent':
        """
        Match Outlook events (self) with existing Odoo events, and return the list of matched events
        """
        # first, try to match recurrences
        # Note that when a recurrence is removed, there is no field in Outlook data to identify
        # the item as a recurrence, so select all deleted items by default.
        recurrence_candidates = self.filter(lambda x: x.is_recurrence() or x.is_removed())
        mapped_recurrences = recurrence_candidates._load_odoo_ids_from_db(env, force_model=env["calendar.recurrence"])

        # then, try to match events
        events_candidates = (self - mapped_recurrences).filter(lambda x: not x.is_recurrence())
        mapped_events = events_candidates._load_odoo_ids_from_db(env)

        return mapped_recurrences | mapped_events

    def _get_model(self, env):
        if all(e.is_recurrence() for e in self):
            return env['calendar.recurrence']
        if all(not e.is_recurrence() for e in self):
            return env['calendar.event']
        raise TypeError("Mixing Microsoft events and Microsoft recurrences")

```

## File: utils\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import microsoft_calendar
from . import microsoft_event
from . import event_id_storage

```

## File: views\microsoft_calendar_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="microsoft_calendar assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/microsoft_calendar/static/src/scss/microsoft_calendar.scss"/>
            <script type="text/javascript" src="/microsoft_calendar/static/src/js/microsoft_calendar_popover.js"></script>
            <script type="text/javascript" src="/microsoft_calendar/static/src/js/microsoft_calendar.js"></script>
        </xpath>
    </template>
</odoo>

```

## File: views\microsoft_calendar_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="view_microsoft_calendar_event" model="ir.ui.view">
        <field name="name">microsoft_calendar.event.calendar</field>
        <field name="model">calendar.event</field>
        <field name="inherit_id" ref="calendar.view_calendar_event_calendar"/>
        <field name="arch" type="xml">
            <field name="attendee_status" position="after">
                <field name="microsoft_id" invisible="1"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.microsoft.calendar</field>
            <field name="model">res.config.settings</field>
            <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <div id="msg_module_microsoft_calendar" position="replace">
                    <div class="content-group" attrs="{'invisible': [('module_microsoft_calendar','=',False)]}">
                        <div class="mt16 row">
                            <label for="cal_microsoft_client_id" string="Client ID" class="col-3 col-lg-3 o_light_label"/>
                            <field name="cal_microsoft_client_id" nolabel="1"/>
                            <label for="cal_microsoft_client_secret" string="Client Secret" class="col-3 col-lg-3 o_light_label"/>
                            <field name="cal_microsoft_client_secret" password="True" nolabel="1"/>
                        </div>
                    </div>
                </div>
            </field>
        </record>
</odoo>

```

## File: views\res_users_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record id="view_users_form" model="ir.ui.view">
            <field name="name">res.users.form</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="base.view_users_form"/>
            <field name="arch" type="xml">
                <notebook colspan="4" position="inside">
                    <page string="Outlook Calendar" name="calendar" groups="base.group_system">
                        <group>
                            <field name="microsoft_calendar_rtoken" readonly="1"/>
                            <field name="microsoft_calendar_token" readonly="1"/>
                            <field name="microsoft_calendar_token_validity" readonly="1"/>
                            <field name="microsoft_calendar_sync_token" readonly="1"/>
                        </group>
                        <button
                            string="Reset Account" class="btn btn-secondary"
                            type="action" name="%(microsoft_calendar_reset_account_action)d"
                            context="{'default_user_id': id}"/>
                    </page>
                </notebook>
            </field>
        </record>

</odoo>

```

## File: wizard\reset_account.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

from odoo.addons.microsoft_calendar.models.microsoft_sync import microsoft_calendar_token


class ResetMicrosoftAccount(models.TransientModel):
    _name = 'microsoft.calendar.account.reset'
    _description = 'Microsoft Calendar Account Reset'

    user_id = fields.Many2one('res.users', required=True)
    delete_policy = fields.Selection(
        [('dont_delete', "Leave them untouched"),
         ('delete_microsoft', "Delete from the current Microsoft Calendar account"),
         ('delete_odoo', "Delete from Odoo"),
         ('delete_both', "Delete from both"),
    ], string="User's Existing Events", required=True, default='dont_delete',
    help="This will only affect events for which the user is the owner")
    sync_policy = fields.Selection([
        ('new', "Synchronize only new events"),
        ('all', "Synchronize all existing events"),
    ], string="Next Synchronization", required=True, default='new')

    def reset_account(self):
        microsoft = self.env["calendar.event"]._get_microsoft_service()

        events = self.env['calendar.event'].search([
            ('user_id', '=', self.user_id.id),
            ('ms_universal_event_id', '!=', False)])
        if self.delete_policy in ('delete_microsoft', 'delete_both'):
            with microsoft_calendar_token(self.user_id) as token:
                for event in events:
                    microsoft.delete(event.ms_universal_event_id, token=token)

        if self.delete_policy in ('delete_odoo', 'delete_both'):
            events.microsoft_id = False
            events.unlink()

        if self.sync_policy == 'all':
            events.write({
                'microsoft_id': False,
                'need_sync_m': True,
            })

        self.user_id._set_microsoft_auth_tokens(False, False, 0)
        self.user_id.write({
            'microsoft_calendar_sync_token': False,
        })

```

## File: wizard\reset_account_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="microsoft_calendar_reset_account_view_form" model="ir.ui.view">
        <field name="name">microsoft.calendar.account.reset.form</field>
        <field name="model">microsoft.calendar.account.reset</field>
        <field name="arch" type="xml">
            <form>
                <h1>Reset Outlook Calendar Account</h1>
                <group>
                    <field name="delete_policy" widget="radio"/>
                    <field name="sync_policy" widget="radio"/>
                </group>

                <footer>
                    <button name="reset_account" string="Confirm" type="object" class="btn-primary"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" />
                </footer>
            </form>
        </field>
    </record>

    <record id="microsoft_calendar_reset_account_action" model="ir.actions.act_window">
        <field name="name"></field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">microsoft.calendar.account.reset</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import reset_account

```

