# Odoo Module: google_calendar

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

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Google Calendar',
    'version': '1.0',
    'category': 'Productivity',
    'depends': ['google_account', 'calendar'],
    'data': [
        'data/google_calendar_data.xml',
        'security/google_calendar_security.xml',
        'security/ir.model.access.csv',
        'wizard/reset_account_views.xml',
        'views/res_config_settings_views.xml',
        'views/res_users_views.xml',
        'views/google_calendar_views.xml',
        ],
    'installable': True,
    'assets': {
        'web.assets_backend': [
            'google_calendar/static/src/scss/google_calendar.scss',
            'google_calendar/static/src/views/**/*',
        ],
        'web.qunit_suite_tests': [
            'google_calendar/static/tests/**/*',
        ],
        'web.qunit_mobile_suite_tests': [
            'google_calendar/static/tests/google_calendar_mock_server.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request
from odoo.addons.google_calendar.utils.google_calendar import GoogleCalendarService
from odoo.addons.calendar.controllers.main import CalendarController


class GoogleCalendarController(CalendarController):

    @http.route('/google_calendar/sync_data', type='json', auth='user')
    def google_calendar_sync_data(self, model, **kw):
        """ This route/function is called when we want to synchronize Odoo
            calendar with Google Calendar.
            Function return a dictionary with the status :  need_config_from_admin, need_auth,
            need_refresh, sync_stopped, success if not calendar_event
            The dictionary may contains an url, to allow Odoo Client to redirect user on
            this URL for authorization for example
        """
        if model == 'calendar.event':
            base_url = request.httprequest.url_root.strip('/')
            GoogleCal = GoogleCalendarService(request.env['google.service'].with_context(base_url=base_url))

            # Checking that admin have already configured Google API for google synchronization !
            client_id = request.env['google.service']._get_client_id('calendar')

            if not client_id or client_id == '':
                action_id = ''
                if GoogleCal._can_authorize_google(request.env.user):
                    action_id = request.env.ref('base_setup.action_general_configuration').id
                return {
                    "status": "need_config_from_admin",
                    "url": '',
                    "action": action_id
                }

            # Checking that user have already accepted Odoo to access his calendar !
            if not GoogleCal.is_authorized(request.env.user):
                url = GoogleCal._google_authentication_url(from_url=kw.get('fromurl'))
                return {
                    "status": "need_auth",
                    "url": url
                }
            # If App authorized, and user access accepted, We launch the synchronization
            need_refresh = request.env.user.sudo()._sync_google_calendar(GoogleCal)

            # If synchronization has been stopped or paused
            sync_status = request.env.user._get_google_sync_status()
            if not need_refresh and sync_status != "sync_active":
                return {
                    "status": sync_status,
                    "url": ''
                }
            return {
                "status": "need_refresh" if need_refresh else "no_new_event_from_google",
                "url": ''
            }

        return {"status": "success"}

    @http.route()
    def check_calendar_credentials(self):
        res = super().check_calendar_credentials()
        get_param = request.env['ir.config_parameter'].sudo().get_param
        client_id = get_param('google_calendar_client_id')
        client_secret = get_param('google_calendar_client_secret')
        res['google_calendar'] = bool(client_id and client_secret)
        return res

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\google_calendar_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record forcecreate="True" id="ir_cron_sync_all_cals" model="ir.cron">
            <field name="name">Google Calendar: synchronization</field>
            <field name="model_id" ref="model_res_users"/>
            <field name="state">code</field>
            <field name="code">
# LUL TODO check this comment
# The context key 'last_sync_hours' allows specifying the minimum delay between consecutive syncs.
# Indeed, in case there are many users / events to sync, the cron might time out. In this case, the
# solution is to force the last_sync_hours to the expected synchronization interval expected. This
# will avoid the synchronization of users that succeeded in the previous failing cron.
model._sync_all_google_calendar()
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

## File: data\neutralize.sql

```sql
-- neutralization of Google calendar
UPDATE google_calendar_credentials
    SET calendar_rtoken = NULL,
        calendar_token = NULL,
        synchronization_stopped = True;

```

## File: models\calendar.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import pytz
from dateutil.parser import parse
from dateutil.relativedelta import relativedelta
from uuid import uuid4

from odoo import api, fields, models, tools, _
from odoo.exceptions import ValidationError

from odoo.addons.google_calendar.utils.google_calendar import GoogleCalendarService

class Meeting(models.Model):
    _name = 'calendar.event'
    _inherit = ['calendar.event', 'google.calendar.sync']

    MEET_ROUTE = 'meet.google.com'

    google_id = fields.Char(
        'Google Calendar Event Id', compute='_compute_google_id', store=True, readonly=False)
    guests_readonly = fields.Boolean(
        'Guests Event Modification Permission', default=False)
    videocall_source = fields.Selection(selection_add=[('google_meet', 'Google Meet')], ondelete={'google_meet': 'set discuss'})

    @api.depends('recurrence_id.google_id')
    def _compute_google_id(self):
        # google ids of recurring events are built from the recurrence id and the
        # original starting time in the recurrence.
        # The `start` field does not appear in the dependencies on purpose!
        # Event if the event is moved, the google_id remains the same.
        for event in self:
            google_recurrence_id = event.recurrence_id._get_event_google_id(event)
            if not event.google_id and google_recurrence_id:
                event.google_id = google_recurrence_id
            elif not event.google_id:
                event.google_id = False

    @api.depends('videocall_location')
    def _compute_videocall_source(self):
        events_with_google_url = self.filtered(lambda event: self.MEET_ROUTE in (event.videocall_location or ''))
        events_with_google_url.videocall_source = 'google_meet'
        super(Meeting, self - events_with_google_url)._compute_videocall_source()

    @api.model
    def _get_google_synced_fields(self):
        return {'name', 'description', 'allday', 'start', 'date_end', 'stop',
                'attendee_ids', 'alarm_ids', 'location', 'privacy', 'active', 'show_as'}

    @api.model
    def _restart_google_sync(self):
        self.env['calendar.event'].search(self._get_sync_domain()).write({
            'need_sync': True,
        })

    @api.model_create_multi
    def create(self, vals_list):
        notify_context = self.env.context.get('dont_notify', False)
        return super(Meeting, self.with_context(dont_notify=notify_context)).create([
            dict(vals, need_sync=False) if vals.get('recurrence_id') or vals.get('recurrency') else vals
            for vals in vals_list
        ])

    @api.model
    def _check_values_to_sync(self, values):
        """ Return True if values being updated intersects with Google synced values and False otherwise. """
        synced_fields = self._get_google_synced_fields()
        values_to_sync = any(key in synced_fields for key in values)
        return values_to_sync

    @api.model
    def _get_update_future_events_values(self):
        """ Add parameters for updating events within the _update_future_events function scope. """
        update_future_events_values = super()._get_update_future_events_values()
        return {**update_future_events_values, 'need_sync': False}

    @api.model
    def _get_remove_sync_id_values(self):
        """ Add parameters for removing event synchronization while updating the events in super class. """
        remove_sync_id_values = super()._get_remove_sync_id_values()
        return {**remove_sync_id_values, 'google_id': False}

    @api.model
    def _get_archive_values(self):
        """ Return the parameters for archiving events. Do not synchronize events after archiving. """
        archive_values = super()._get_archive_values()
        return {**archive_values, 'need_sync': False}

    def write(self, values):
        recurrence_update_setting = values.get('recurrence_update')
        if recurrence_update_setting in ('all_events', 'future_events') and len(self) == 1:
            values = dict(values, need_sync=False)
        notify_context = self.env.context.get('dont_notify', False)
        if not notify_context and ([self.env.user.id != record.user_id.id for record in self]):
            self._check_modify_event_permission(values)
        res = super(Meeting, self.with_context(dont_notify=notify_context)).write(values)
        if recurrence_update_setting in ('all_events',) and len(self) == 1 and values.keys() & self._get_google_synced_fields():
            self.recurrence_id.need_sync = True
        return res

    def _check_modify_event_permission(self, values):
        # Check if event modification attempt by attendee is valid to avoid duplicate events creation.
        for event in self:
            # Edge case: when restarting the synchronization, guests can write 'need_sync=True' on events.
            google_sync_restart = values.get('need_sync') and len(values)
            if not google_sync_restart and (event.guests_readonly and self.env.user.id != event.user_id.id):
                raise ValidationError(_("The following event can only be updated by the organizer "
                                        "according to the event permissions set on Google Calendar."))

    def _get_sync_domain(self):
        # in case of full sync, limit to a range of 1y in past and 1y in the future by default
        ICP = self.env['ir.config_parameter'].sudo()
        day_range = int(ICP.get_param('google_calendar.sync.range_days', default=365))
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
    def _odoo_values(self, google_event, default_reminders=()):
        if google_event.is_cancelled():
            return {'active': False}

        # default_reminders is never () it is set to google's default reminder (30 min before)
        # we need to check 'useDefault' for the event to determine if we have to use google's
        # default reminder or not
        reminder_command = google_event.reminders.get('overrides')
        if not reminder_command:
            reminder_command = google_event.reminders.get('useDefault') and default_reminders or ()
        alarm_commands = self._odoo_reminders_commands(reminder_command)
        attendee_commands, partner_commands = self._odoo_attendee_commands(google_event)
        related_event = self.search([('google_id', '=', google_event.id)], limit=1)
        name = google_event.summary or related_event and related_event.name or _("(No title)")
        values = {
            'name': name,
            'description': google_event.description and tools.html_sanitize(google_event.description),
            'location': google_event.location,
            'user_id': google_event.owner(self.env).id,
            'privacy': google_event.visibility or self.default_get(['privacy'])['privacy'],
            'attendee_ids': attendee_commands,
            'alarm_ids': alarm_commands,
            'recurrency': google_event.is_recurrent(),
            'videocall_location': google_event.get_meeting_url(),
            'show_as': 'free' if google_event.is_available() else 'busy',
            'guests_readonly': not bool(google_event.guestsCanModify)
        }
        # Remove 'videocall_location' when not sent by Google, otherwise the local videocall will be discarded.
        if not values.get('videocall_location'):
            values.pop('videocall_location', False)
        if partner_commands:
            # Add partner_commands only if set from Google. The write method on calendar_events will
            # override attendee commands if the partner_ids command is set but empty.
            values['partner_ids'] = partner_commands
        if not google_event.is_recurrence():
            values['google_id'] = google_event.id
        if google_event.is_recurrent() and not google_event.is_recurrence():
            # Propagate the follow_recurrence according to the google result
            values['follow_recurrence'] = google_event.is_recurrence_follower()
        if google_event.start.get('dateTime'):
            # starting from python3.7, use the new [datetime, date].fromisoformat method
            start = parse(google_event.start.get('dateTime')).astimezone(pytz.utc).replace(tzinfo=None)
            stop = parse(google_event.end.get('dateTime')).astimezone(pytz.utc).replace(tzinfo=None)
            values['allday'] = False
        else:
            start = parse(google_event.start.get('date'))
            stop = parse(google_event.end.get('date')) - relativedelta(days=1)
            # Stop date should be exclusive as defined here https://developers.google.com/calendar/v3/reference/events#resource
            # but it seems that's not always the case for old event
            if stop < start:
                stop = parse(google_event.end.get('date'))
            values['allday'] = True
        if related_event['start'] != start:
            values['start'] = start
        if related_event['stop'] != stop:
            values['stop'] = stop
        return values

    @api.model
    def _odoo_attendee_commands(self, google_event):
        attendee_commands = []
        partner_commands = []
        google_attendees = google_event.attendees or []
        if len(google_attendees) == 0 and google_event.organizer and google_event.organizer.get('self', False):
            user = google_event.owner(self.env)
            google_attendees += [{
                'email': user.partner_id.email,
                'responseStatus': 'accepted',
            }]
        emails = [a.get('email') for a in google_attendees]
        existing_attendees = self.env['calendar.attendee']
        if google_event.exists(self.env):
            event = google_event.get_odoo_event(self.env)
            existing_attendees = event.attendee_ids
        attendees_by_emails = {tools.email_normalize(a.email): a for a in existing_attendees}
        partners = self._get_sync_partner(emails)
        for attendee in zip(emails, partners, google_attendees):
            email = attendee[0]
            if email in attendees_by_emails:
                # Update existing attendees
                attendee_commands += [(1, attendees_by_emails[email].id, {'state': attendee[2].get('responseStatus')})]
            else:
                # Create new attendees
                if attendee[2].get('self'):
                    partner = self.env.user.partner_id
                elif attendee[1]:
                    partner = attendee[1]
                else:
                    continue
                attendee_commands += [(0, 0, {'state': attendee[2].get('responseStatus'), 'partner_id': partner.id})]
                partner_commands += [(4, partner.id)]
                if attendee[2].get('displayName') and not partner.name:
                    partner.name = attendee[2].get('displayName')
        for odoo_attendee in attendees_by_emails.values():
            # Remove old attendees but only if it does not correspond to the current user.
            email = tools.email_normalize(odoo_attendee.email)
            if email not in emails and email != self.env.user.email:
                attendee_commands += [(2, odoo_attendee.id)]
                partner_commands += [(3, odoo_attendee.partner_id.id)]
        return attendee_commands, partner_commands

    @api.model
    def _odoo_reminders_commands(self, reminders=()):
        commands = []
        for reminder in reminders:
            alarm_type = 'email' if reminder.get('method') == 'email' else 'notification'
            alarm_type_label = _("Email") if alarm_type == 'email' else _("Notification")

            minutes = reminder.get('minutes', 0)
            alarm = self.env['calendar.alarm'].search([
                ('alarm_type', '=', alarm_type),
                ('duration_minutes', '=', minutes)
            ], limit=1)
            if alarm:
                commands += [(4, alarm.id)]
            else:
                if minutes % (60*24) == 0:
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
                commands += [(0, 0, {'duration': duration, 'interval': interval, 'name': name, 'alarm_type': alarm_type})]
        return commands

    def action_mass_archive(self, recurrence_update_setting):
        """ Delete recurrence in Odoo if in 'all_events' or in 'future_events' edge case, triggering one mail. """
        self.ensure_one()
        google_service = GoogleCalendarService(self.env['google.service'])
        archive_future_events = recurrence_update_setting == 'future_events' and self == self.recurrence_id.base_event_id
        if recurrence_update_setting == 'all_events' or archive_future_events:
            self.recurrence_id.with_context(is_recurrence=True)._google_delete(google_service, self.recurrence_id.google_id)
            # Increase performance handling 'future_events' edge case as it was an 'all_events' update.
            if archive_future_events:
                recurrence_update_setting = 'all_events'
        super(Meeting, self).action_mass_archive(recurrence_update_setting)

    def _google_values(self):
        # In Google API, all-day events must have their 'dateTime' information set
        # as null and timed events must have their 'date' information set as null.
        # This is mandatory for allowing changing timed events to all-day and vice versa.
        start = {'date': None, 'dateTime': None}
        end = {'date': None, 'dateTime': None}
        if self.allday:
            # For all-day events, 'dateTime' must be set to None to indicate that it's an all-day event.
            # Otherwise, if both 'date' and 'dateTime' are set, Google may not recognize it as an all-day event.
            start['date'] = self.start_date.isoformat()
            end['date'] = (self.stop_date + relativedelta(days=1)).isoformat()
        else:
            # For timed events, 'date' must be set to None to indicate that it's not an all-day event.
            # Otherwise, if both 'date' and 'dateTime' are set, Google may not recognize it as a timed event
            start['dateTime'] = pytz.utc.localize(self.start).isoformat()
            end['dateTime'] = pytz.utc.localize(self.stop).isoformat()
        reminders = [{
            'method': "email" if alarm.alarm_type == "email" else "popup",
            'minutes': alarm.duration_minutes
        } for alarm in self.alarm_ids]

        attendees = self.attendee_ids
        attendee_values = [{
            'email': attendee.partner_id.email_normalized,
            'responseStatus': attendee.state or 'needsAction',
        } for attendee in attendees if attendee.partner_id.email_normalized]
        # We sort the attendees to avoid undeterministic test fails. It's not mandatory for Google.
        attendee_values.sort(key=lambda k: k['email'])
        values = {
            'id': self.google_id,
            'start': start,
            'end': end,
            'summary': self.name,
            'description': tools.html_sanitize(self.description) if not tools.is_html_empty(self.description) else '',
            'location': self.location or '',
            'guestsCanModify': not self.guests_readonly,
            'organizer': {'email': self.user_id.email, 'self': self.user_id == self.env.user},
            'attendees': attendee_values,
            'extendedProperties': {
                'shared': {
                    '%s_odoo_id' % self.env.cr.dbname: self.id,
                },
            },
            'reminders': {
                'overrides': reminders,
                'useDefault': False,
            }
        }
        if not self.google_id and not self.videocall_location and not self.location:
            values['conferenceData'] = {'createRequest': {'requestId': uuid4().hex}}
        if self.privacy:
            values['visibility'] = self.privacy
        if self.show_as:
            values['transparency'] = 'opaque' if self.show_as == 'busy' else 'transparent'
        if not self.active:
            values['status'] = 'cancelled'
        if self.user_id and self.user_id != self.env.user and not bool(self.user_id.sudo().google_calendar_token):
            # The organizer is an Odoo user that do not sync his calendar
            values['extendedProperties']['shared']['%s_owner_id' % self.env.cr.dbname] = self.user_id.id
        elif not self.user_id:
            # We can't store on the shared properties in that case without getting a 403. It can happen when
            # the owner is not an Odoo user: We don't store the real owner identity (mail)
            # If we are not the owner, we should change the post values to avoid errors because we don't have
            # write permissions
            # See https://developers.google.com/calendar/concepts/sharing
            keep_keys = ['id', 'summary', 'attendees', 'start', 'end', 'reminders']
            values = {key: val for key, val in values.items() if key in keep_keys}
            # values['extendedProperties']['private] should be used if the owner is not an odoo user
            values['extendedProperties'] = {
                'private': {
                    '%s_odoo_id' % self.env.cr.dbname: self.id,
                },
            }
        return values

    def _cancel(self):
        # only owner can delete => others refuse the event
        user = self.env.user
        my_cancelled_records = self.filtered(lambda e: e.user_id == user)
        for event in self:
            # remove the tracking data to avoid calling _track_template in the pre-commit phase
            self.env.cr.precommit.data.pop(f'mail.tracking.create.{event._name}.{event.id}', None)
        super(Meeting, my_cancelled_records)._cancel()
        attendees = (self - my_cancelled_records).attendee_ids.filtered(lambda a: a.partner_id == user.partner_id)
        attendees.state = 'declined'

    def _get_event_user(self):
        self.ensure_one()
        if self.user_id and self.user_id.sudo().google_calendar_token:
            return self.user_id
        return self.env.user

```

## File: models\calendar_attendee.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

from odoo.addons.google_calendar.models.google_sync import google_calendar_token
from odoo.addons.google_calendar.utils.google_calendar import GoogleCalendarService

class Attendee(models.Model):
    _name = 'calendar.attendee'
    _inherit = 'calendar.attendee'

    def _send_mail_to_attendees(self, mail_template, force_send=False):
        """ Override
        If not synced with Google, let Odoo in charge of sending emails
        Otherwise, nothing to do: Google will send them
        """
        with google_calendar_token(self.env.user.sudo()) as token:
            if not token:
                super()._send_mail_to_attendees(mail_template, force_send)

    def do_tentative(self):
        # Synchronize event after state change
        res = super().do_tentative()
        self._sync_event()
        return res

    def do_accept(self):
        # Synchronize event after state change
        res = super().do_accept()
        self._sync_event()
        return res


    def do_decline(self):
        # Synchronize event after state change
        res = super().do_decline()
        self._sync_event()
        return res

    def _sync_event(self):
        # For weird reasons, we can't sync status when we are not the responsible
        # We can't adapt google_value to only keep ['id', 'summary', 'attendees', 'start', 'end', 'reminders']
        # and send that. We get a Forbidden for non-organizer error even if we only send start, end that are mandatory !
        all_events = self.mapped('event_id').filtered(lambda e: e.google_id)
        other_events = all_events.filtered(lambda e: e.user_id and e.user_id.id != self.env.user.id)
        for user in other_events.mapped('user_id'):
            service = GoogleCalendarService(self.env['google.service'].with_user(user))
            other_events.filtered(lambda ev: ev.user_id.id == user.id).with_user(user)._sync_odoo2google(service)
        google_service = GoogleCalendarService(self.env['google.service'])
        (all_events - other_events)._sync_odoo2google(google_service)

```

## File: models\calendar_recurrence_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
import logging

from odoo import api, models, Command
from odoo.tools import email_normalize

from odoo.addons.google_calendar.utils.google_calendar import GoogleCalendarService

_logger = logging.getLogger(__name__)

class RecurrenceRule(models.Model):
    _name = 'calendar.recurrence'
    _inherit = ['calendar.recurrence', 'google.calendar.sync']


    def _apply_recurrence(self, specific_values_creation=None, no_send_edit=False, generic_values_creation=None):
        events = self.filtered('need_sync').calendar_event_ids
        detached_events = super()._apply_recurrence(specific_values_creation, no_send_edit,
                                                    generic_values_creation)

        google_service = GoogleCalendarService(self.env['google.service'])

        # If a synced event becomes a recurrence, the event needs to be deleted from
        # Google since it's now the recurrence which is synced.
        # Those events are kept in the database and their google_id is updated
        # according to the recurrence google_id, therefore we need to keep an inactive copy
        # of those events with the original google id. The next sync will then correctly
        # delete those events from Google.
        vals = []
        for event in events.filtered('google_id'):
            if event.active and event.google_id != event.recurrence_id._get_event_google_id(event):
                vals += [{
                    'name': event.name,
                    'google_id': event.google_id,
                    'start': event.start,
                    'stop': event.stop,
                    'active': False,
                    'need_sync': True,
                }]
                event.with_user(event._get_event_user())._google_delete(google_service, event.google_id)
                event.google_id = False
        self.env['calendar.event'].create(vals)

        self.calendar_event_ids.need_sync = False
        return detached_events

    def _get_event_google_id(self, event):
        """Return the Google id of recurring event.
        Google ids of recurrence instances are formatted as: {recurrence google_id}_{UTC starting time in compacted ISO8601}
        """
        if self.google_id:
            if event.allday:
                time_id = event.start_date.isoformat().replace('-', '')
            else:
                # '-' and ':' are optional in ISO8601
                start_compacted_iso8601 = event.start.isoformat().replace('-', '').replace(':', '')
                # Z at the end for UTC
                time_id = '%sZ' % start_compacted_iso8601
            return '%s_%s' % (self.google_id, time_id)
        return False

    def _write_events(self, values, dtstart=None):
        values.pop('google_id', False)
        # Events will be updated by patch requests, do not sync events for avoiding spam.
        values['need_sync'] = False
        return super()._write_events(values, dtstart=dtstart)

    def _cancel(self):
        self.calendar_event_ids._cancel()
        super()._cancel()

    def _get_google_synced_fields(self):
        return {'rrule'}

    @api.model
    def _restart_google_sync(self):
        self.env['calendar.recurrence'].search(self._get_sync_domain()).write({
            'need_sync': True,
        })

    def _write_from_google(self, gevent, vals):
        current_rrule = self.rrule
        current_parsed_rrule = self._rrule_parse(current_rrule, self.dtstart)
        # event_tz is written on event in Google but on recurrence in Odoo
        vals['event_tz'] = gevent.start.get('timeZone')
        super()._write_from_google(gevent, vals)

        base_event_time_fields = ['start', 'stop', 'allday']
        new_event_values = self.env["calendar.event"]._odoo_values(gevent)
        new_parsed_rrule = self._rrule_parse(self.rrule, self.dtstart)
        # We update the attendee status for all events in the recurrence
        google_attendees = gevent.attendees or []
        emails = [a.get('email') for a in google_attendees]
        partners = self._get_sync_partner(emails)
        existing_attendees = self.calendar_event_ids.attendee_ids
        for attendee in zip(emails, partners, google_attendees):
            email = attendee[0]
            if email in existing_attendees.mapped('email'):
                # Update existing attendees
                existing_attendees.filtered(lambda att: att.email == email).write({'state': attendee[2].get('responseStatus')})
            else:
                # Create new attendees
                if attendee[2].get('self'):
                    partner = self.env.user.partner_id
                elif attendee[1]:
                    partner = attendee[1]
                else:
                    continue
                self.calendar_event_ids.write({'attendee_ids': [(0, 0, {'state': attendee[2].get('responseStatus'), 'partner_id': partner.id})]})
                if attendee[2].get('displayName') and not partner.name:
                    partner.name = attendee[2].get('displayName')

        organizers_partner_ids = [event.user_id.partner_id for event in self.calendar_event_ids if event.user_id]
        for odoo_attendee_email in set(existing_attendees.mapped('email')):
            # Sometimes, several partners have the same email. Remove old attendees except organizer, otherwise the events will disappear.
            if email_normalize(odoo_attendee_email) not in emails:
                attendees = existing_attendees.exists().filtered(lambda att: att.email == email_normalize(odoo_attendee_email) and att.partner_id not in organizers_partner_ids)
                self.calendar_event_ids.write({'need_sync': False, 'partner_ids': [Command.unlink(att.partner_id.id) for att in attendees]})

        old_event_values = self.base_event_id and self.base_event_id.read(base_event_time_fields)[0]
        if old_event_values and any(new_event_values.get(key) and new_event_values[key] != old_event_values[key] for key in base_event_time_fields):
            # we need to recreate the recurrence, time_fields were modified.
            base_event_id = self.base_event_id
            non_equal_values = [
                (key, old_event_values[key] and old_event_values[key].strftime('%m/%d/%Y, %H:%M:%S'), '-->',
                      new_event_values[key] and new_event_values[key].strftime('%m/%d/%Y, %H:%M:%S')
                 ) for key in ['start', 'stop'] if new_event_values[key] != old_event_values[key]
            ]
            log_msg = f"Recurrence {self.id} {self.rrule} has all events ({len(self.calendar_event_ids.ids)})  deleted because of base event value change: {non_equal_values}"
            _logger.info(log_msg)
            # We archive the old events to recompute the recurrence. These events are already deleted on Google side.
            # We can't call _cancel because events without user_id would not be deleted
            (self.calendar_event_ids - base_event_id).google_id = False
            (self.calendar_event_ids - base_event_id).unlink()
            base_event_id.with_context(dont_notify=True).write(dict(new_event_values, google_id=False, need_sync=False))
            if new_parsed_rrule == current_parsed_rrule:
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
                }, need_sync=False)
            )

        # We apply the rrule check after the time_field check because the google_id are generated according
        # to base_event start datetime.
        if new_parsed_rrule != current_parsed_rrule:
            detached_events = self._apply_recurrence()
            detached_events.google_id = False
            log_msg = f"Recurrence #{self.id} | current rule: {current_rrule} | new rule: {self.rrule} | remaining: {len(self.calendar_event_ids)} | removed: {len(detached_events)}"
            _logger.info(log_msg)
            detached_events.unlink()

    def _create_from_google(self, gevents, vals_list):
        attendee_values = {}
        for gevent, vals in zip(gevents, vals_list):
            base_values = dict(
                self.env['calendar.event']._odoo_values(gevent),  # FIXME default reminders
                need_sync=False,
            )
            # If we convert a single event into a recurrency on Google, we should reuse this event on Odoo
            # Google reuse the event google_id to identify the recurrence in that case
            base_event = self.env['calendar.event'].search([('google_id', '=', vals['google_id'])])
            if not base_event:
                base_event = self.env['calendar.event'].create(base_values)
            else:
                # We override the base_event values because they could have been changed in Google interface
                # The event google_id will be recalculated once the recurrence is created
                base_event.write(dict(base_values, google_id=False))
            vals['base_event_id'] = base_event.id
            vals['calendar_event_ids'] = [(4, base_event.id)]
            # event_tz is written on event in Google but on recurrence in Odoo
            vals['event_tz'] = gevent.start.get('timeZone')
            attendee_values[base_event.id] = {'attendee_ids': base_values.get('attendee_ids')}

        recurrence = super(RecurrenceRule, self.with_context(dont_notify=True))._create_from_google(gevents, vals_list)
        generic_values_creation = {
            rec.id: attendee_values[rec.base_event_id.id]
            for rec in recurrence if attendee_values.get(rec.base_event_id.id)
        }
        recurrence.with_context(dont_notify=True)._apply_recurrence(generic_values_creation=generic_values_creation)
        return recurrence

    def _get_sync_domain(self):
        # Empty rrule may exists in historical data. It is not a desired behavior but it could have been created with
        # older versions of the module. When synced, these recurrency may come back from Google after database cleaning
        # and trigger errors as the records are not properly populated.
        # We also prevent sync of other user recurrent events.
        return [('calendar_event_ids.user_id', '=', self.env.user.id), ('rrule', '!=', False)]

    @api.model
    def _odoo_values(self, google_recurrence, default_reminders=()):
        return {
            'rrule': google_recurrence.rrule,
            'google_id': google_recurrence.id,
        }

    def _google_values(self):
        event = self._get_first_event()
        if not event:
            return {}
        values = event._google_values()
        values['id'] = self.google_id
        if not self._is_allday():
            values['start']['timeZone'] = self.event_tz or 'Etc/UTC'
            values['end']['timeZone'] = self.event_tz or 'Etc/UTC'

        # DTSTART is not allowed by Google Calendar API.
        # Event start and end times are specified in the start and end fields.
        rrule = re.sub('DTSTART:[0-9]{8}T[0-9]{1,8}\\n', '', self.rrule)
        # UNTIL must be in UTC (appending Z)
        # We want to only add a 'Z' to non UTC UNTIL values and avoid adding a second.
        # 'RRULE:FREQ=DAILY;UNTIL=20210224T235959;INTERVAL=3 --> match UNTIL=20210224T235959
        # 'RRULE:FREQ=DAILY;UNTIL=20210224T235959 --> match
        rrule = re.sub(r"(UNTIL=\d{8}T\d{6})($|;)", r"\1Z\2", rrule)
        values['recurrence'] = ['RRULE:%s' % rrule] if 'RRULE:' not in rrule else [rrule]
        property_location = 'shared' if event.user_id else 'private'
        values['extendedProperties'] = {
            property_location: {
                '%s_odoo_id' % self.env.cr.dbname: self.id,
            },
        }
        return values

    def _get_event_user(self):
        self.ensure_one()
        event = self._get_first_event()
        if event:
            return event._get_event_user()
        return self.env.user

```

## File: models\google_credentials.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import requests
from datetime import timedelta

from odoo import fields, models, _
from odoo.exceptions import UserError
from odoo.addons.google_account.models.google_service import GOOGLE_TOKEN_ENDPOINT

_logger = logging.getLogger(__name__)

class GoogleCredentials(models.Model):
    """"Google Account of res_users"""

    _name = 'google.calendar.credentials'
    _description = 'Google Calendar Account Data'

    user_ids = fields.One2many('res.users', 'google_calendar_account_id', required=True)
    calendar_rtoken = fields.Char('Refresh Token', copy=False)
    calendar_token = fields.Char('User token', copy=False)
    calendar_token_validity = fields.Datetime('Token Validity', copy=False)
    calendar_sync_token = fields.Char('Next Sync Token', copy=False)

    calendar_cal_id = fields.Char('Calendar ID', copy=False, help='Last Calendar ID who has been synchronized. If it is changed, we remove all links between GoogleID and Odoo Google Internal ID')
    synchronization_stopped = fields.Boolean('Google Synchronization stopped', copy=False)

    def _set_auth_tokens(self, access_token, refresh_token, ttl):
        self.write({
            'calendar_rtoken': refresh_token,
            'calendar_token': access_token,
            'calendar_token_validity': fields.Datetime.now() + timedelta(seconds=ttl) if ttl else False,
        })

    def _google_calendar_authenticated(self):
        self.ensure_one()
        return bool(self.sudo().calendar_rtoken)

    def _is_google_calendar_valid(self):
        self.ensure_one()
        return self.calendar_token_validity and self.calendar_token_validity >= (fields.Datetime.now() + timedelta(minutes=1))

    def _refresh_google_calendar_token(self):
        # LUL TODO similar code exists in google_drive. Should be factorized in google_account
        self.ensure_one()
        get_param = self.env['ir.config_parameter'].sudo().get_param
        client_id = get_param('google_calendar_client_id')
        client_secret = get_param('google_calendar_client_secret')

        if not client_id or not client_secret:
            raise UserError(_("The account for the Google Calendar service is not configured."))

        headers = {"content-type": "application/x-www-form-urlencoded"}
        data = {
            'refresh_token': self.calendar_rtoken,
            'client_id': client_id,
            'client_secret': client_secret,
            'grant_type': 'refresh_token',
        }

        try:
            _dummy, response, _dummy = self.env['google.service']._do_request(GOOGLE_TOKEN_ENDPOINT, params=data, headers=headers, method='POST', preuri='')
            ttl = response.get('expires_in')
            self.write({
                'calendar_token': response.get('access_token'),
                'calendar_token_validity': fields.Datetime.now() + timedelta(seconds=ttl),
            })
        except requests.HTTPError as error:
            if error.response.status_code in (400, 401):  # invalid grant or invalid client
                # Delete refresh token and make sure it's commited
                self.env.cr.rollback()
                self._set_auth_tokens(False, False, 0)
                self.env.cr.commit()
            error_key = error.response.json().get("error", "nc")
            error_msg = _("An error occurred while generating the token. Your authorization code may be invalid or has already expired [%s]. "
                          "You should check your Client ID and secret on the Google APIs plateform or try to stop and restart your calendar synchronisation.",
                          error_key)
            raise UserError(error_msg)

```

## File: models\google_sync.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from contextlib import contextmanager
from functools import wraps
from requests import HTTPError
import pytz
from dateutil.parser import parse
from markupsafe import Markup

from odoo import api, fields, models, registry, _
from odoo.tools import ormcache_context, email_normalize
from odoo.osv import expression
from odoo.sql_db import BaseCursor

from odoo.addons.google_calendar.utils.google_event import GoogleEvent
from odoo.addons.google_calendar.utils.google_calendar import GoogleCalendarService
from odoo.addons.google_account.models.google_service import TIMEOUT

_logger = logging.getLogger(__name__)


# API requests are sent to Google Calendar after the current transaction ends.
# This ensures changes are sent to Google only if they really happened in the Odoo database.
# It is particularly important for event creation , otherwise the event might be created
# twice in Google if the first creation crashed in Odoo.
def after_commit(func):
    @wraps(func)
    def wrapped(self, *args, **kwargs):
        assert isinstance(self.env.cr, BaseCursor)
        dbname = self.env.cr.dbname
        context = self.env.context
        uid = self.env.uid

        if self.env.context.get('no_calendar_sync'):
            return

        @self.env.cr.postcommit.add
        def called_after():
            db_registry = registry(dbname)
            with db_registry.cursor() as cr:
                env = api.Environment(cr, uid, context)
                try:
                    func(self.with_env(env), *args, **kwargs)
                except Exception as e:
                    _logger.warning("Could not sync record now: %s" % self)
                    _logger.exception(e)

    return wrapped

@contextmanager
def google_calendar_token(user):
    yield user._get_google_calendar_token()


class GoogleSync(models.AbstractModel):
    _name = 'google.calendar.sync'
    _description = "Synchronize a record with Google Calendar"

    google_id = fields.Char('Google Calendar Id', copy=False)
    need_sync = fields.Boolean(default=True, copy=False)
    active = fields.Boolean(default=True)

    def write(self, vals):
        google_service = GoogleCalendarService(self.env['google.service'])
        if 'google_id' in vals:
            self.env.registry.clear_cache()  # _event_ids_from_google_ids
        synced_fields = self._get_google_synced_fields()
        if 'need_sync' not in vals and vals.keys() & synced_fields and not self.env.user.google_synchronization_stopped:
            vals['need_sync'] = True

        result = super().write(vals)
        if self.env.user._get_google_sync_status() != "sync_paused":
            for record in self:
                if record.need_sync and record.google_id:
                    record.with_user(record._get_event_user())._google_patch(google_service, record.google_id, record._google_values(), timeout=3)

        return result

    @api.model_create_multi
    def create(self, vals_list):
        if any(vals.get('google_id') for vals in vals_list):
            self.env.registry.clear_cache()  # _event_ids_from_google_ids
        if self.env.user.google_synchronization_stopped:
            for vals in vals_list:
                vals.update({'need_sync': False})
        records = super().create(vals_list)
        self._handle_allday_recurrences_edge_case(records, vals_list)

        google_service = GoogleCalendarService(self.env['google.service'])
        if self.env.user._get_google_sync_status() != "sync_paused":
            for record in records:
                if record.need_sync and record.active:
                    record.with_user(record._get_event_user())._google_insert(google_service, record._google_values(), timeout=3)
        return records

    def _handle_allday_recurrences_edge_case(self, records, vals_list):
        """
        When creating 'All Day' recurrent event, the first event is wrongly synchronized as
        a single event and then its recurrence creates a duplicated event. We must manually
        set the 'need_sync' attribute as False in order to avoid this unwanted behavior.
        """
        if vals_list and self._name == 'calendar.event':
            forbid_sync = all(not vals.get('need_sync', True) for vals in vals_list)
            records_to_skip = records.filtered(lambda r: r.need_sync and r.allday and r.recurrency and not r.recurrence_id)
            if forbid_sync and records_to_skip:
                records_to_skip.with_context(send_updates=False).need_sync = False

    def unlink(self):
        """We can't delete an event that is also in Google Calendar. Otherwise we would
        have no clue that the event must must deleted from Google Calendar at the next sync.
        """
        synced = self.filtered('google_id')
        # LUL TODO find a way to get rid of this context key
        if self.env.context.get('archive_on_error') and self._active_name:
            synced.write({self._active_name: False})
            self = self - synced
        elif synced:
            # Since we can not delete such an event (see method comment), we archive it.
            # Notice that archiving an event will delete the associated event on Google.
            # Then, since it has been deleted on Google, the event is also deleted on Odoo DB (_sync_google2odoo).
            self.action_archive()
            return True
        return super().unlink()

    def _from_google_ids(self, google_ids):
        if not google_ids:
            return self.browse()
        return self.browse(self._event_ids_from_google_ids(google_ids))

    @api.model
    @ormcache_context('google_ids', keys=('active_test',))
    def _event_ids_from_google_ids(self, google_ids):
        return self.search([('google_id', 'in', google_ids)]).ids

    def _sync_odoo2google(self, google_service: GoogleCalendarService):
        if not self:
            return
        if self._active_name:
            records_to_sync = self.filtered(self._active_name)
        else:
            records_to_sync = self
        cancelled_records = self - records_to_sync

        updated_records = records_to_sync.filtered('google_id')
        new_records = records_to_sync - updated_records
        if self.env.user._get_google_sync_status() != "sync_paused":
            for record in cancelled_records:
                if record.google_id and record.need_sync:
                    record.with_user(record._get_event_user())._google_delete(google_service, record.google_id)
            for record in new_records:
                record.with_user(record._get_event_user())._google_insert(google_service, record._google_values())
            for record in updated_records:
                record.with_user(record._get_event_user())._google_patch(google_service, record.google_id, record._google_values())

    def _cancel(self):
        self.with_context(dont_notify=True).write({'google_id': False})
        self.unlink()

    @api.model
    def _sync_google2odoo(self, google_events: GoogleEvent, default_reminders=()):
        """Synchronize Google recurrences in Odoo. Creates new recurrences, updates
        existing ones.

        :param google_recurrences: Google recurrences to synchronize in Odoo
        :return: synchronized odoo recurrences
        """
        existing = google_events.exists(self.env)
        new = google_events - existing - google_events.cancelled()
        write_dates = self._context.get('write_dates', {})
        # Pop 'write_dates' from the context (frozendict) after using it for resetting the old value in the call stack.
        context = dict(self.env.context, dont_notify=True)
        context.pop('write_dates', None)

        odoo_values = [
            dict(self._odoo_values(e, default_reminders), need_sync=False)
            for e in new
        ]
        new_odoo = self.with_context(dont_notify=True)._create_from_google(new, odoo_values)
        cancelled = existing.cancelled()
        cancelled_odoo = self.browse(cancelled.odoo_ids(self.env)).exists()

        # Check if it is a recurring event that has been rescheduled.
        # We have to check if an event already exists in Odoo.
        # Explanation:
        # A recurrent event with `google_id` is equal to ID_RANGE_TIMESTAMP can be rescheduled.
        # The new `google_id` will be equal to ID_TIMESTAMP.
        # We have to delete the event created under the old `google_id`.
        rescheduled_events = new.filter(lambda gevent: not gevent.is_recurrence_follower())
        if rescheduled_events:
            google_ids_to_remove = [event.full_recurring_event_id() for event in rescheduled_events]
            cancelled_odoo += self.env['calendar.event'].search([('google_id', 'in', google_ids_to_remove)])

        cancelled_odoo.exists()._cancel()
        synced_records = new_odoo + cancelled_odoo
        pending = existing - cancelled
        pending_odoo = self.browse(pending.odoo_ids(self.env)).exists()
        for gevent in pending:
            odoo_record = self.browse(gevent.odoo_id(self.env))
            if odoo_record not in pending_odoo:
                # The record must have been deleted in the mean time; nothing left to sync
                continue
            # Last updated wins.
            # This could be dangerous if google server time and odoo server time are different
            updated = parse(gevent.updated)
            # Use the record's write_date to apply Google updates only if they are newer than Odoo's write_date.
            odoo_record_write_date = write_dates.get(odoo_record.id, odoo_record.write_date)
            # Migration from 13.4 does not fill write_date. Therefore, we force the update from Google.
            if not odoo_record_write_date or updated >= pytz.utc.localize(odoo_record_write_date):
                vals = dict(self._odoo_values(gevent, default_reminders), need_sync=False)
                odoo_record.with_context(context)._write_from_google(gevent, vals)
                synced_records |= odoo_record

        return synced_records

    def _google_error_handling(self, http_error):
        # We only handle the most problematic errors of sync events.
        if http_error.response.status_code in (403, 400):
            response = http_error.response.json()
            if not self.exists():
                reason = "Google gave the following explanation: %s" % response['error'].get('message')
                error_log = "Error while syncing record. It does not exists anymore in the database. %s" % reason
                _logger.error(error_log)
                return

            if self._name == 'calendar.event':
                start = self.start and self.start.strftime('%Y-%m-%d at %H:%M') or _("undefined time")
                event_ids = self.id
                name = self.name
                error_log = "Error while syncing event: "
                event = self
            else:
                # calendar recurrence is triggering the error
                event = self.base_event_id or self._get_first_event(include_outliers=True)
                start = event.start and event.start.strftime('%Y-%m-%d at %H:%M') or _("undefined time")
                event_ids = _("%(id)s and %(length)s following", id=event.id, length=len(self.calendar_event_ids.ids))
                name = event.name
                # prevent to sync other events
                self.calendar_event_ids.need_sync = False
                error_log = "Error while syncing recurrence [{id} - {name} - {rrule}]: ".format(id=self.id, name=self.name, rrule=self.rrule)

            # We don't have right access on the event or the request paramaters were bad.
            # https://developers.google.com/calendar/v3/errors#403_forbidden_for_non-organizer
            if http_error.response.status_code == 403 and "forbiddenForNonOrganizer" in http_error.response.text:
                reason = _("you don't seem to have permission to modify this event on Google Calendar")
            else:
                reason = _("Google gave the following explanation: %s", response['error'].get('message'))

            error_log += "The event (%(id)s - %(name)s at %(start)s) could not be synced. It will not be synced while " \
                         "it is not updated. Reason: %(reason)s" % {'id': event_ids, 'start': start, 'name': name,
                                                                    'reason': reason}
            _logger.warning(error_log)

            body = _("The following event could not be synced with Google Calendar.") + Markup("<br/>") + \
                   _("It will not be synced as long at it is not updated.") + Markup("<br/>") + \
                   reason

            if event:
                event.message_post(
                    body=body,
                    message_type='comment',
                    subtype_xmlid='mail.mt_note',
                )

    @after_commit
    def _google_delete(self, google_service: GoogleCalendarService, google_id, timeout=TIMEOUT):
        with google_calendar_token(self.env.user.sudo()) as token:
            if token:
                is_recurrence = self._context.get('is_recurrence', False)
                google_service.google_service = google_service.google_service.with_context(is_recurrence=is_recurrence)
                google_service.delete(google_id, token=token, timeout=timeout)
                # When the record has been deleted on our side, we need to delete it on google but we don't want
                # to raise an error because the record don't exists anymore.
                self.exists().with_context(dont_notify=True).need_sync = False

    @after_commit
    def _google_patch(self, google_service: GoogleCalendarService, google_id, values, timeout=TIMEOUT):
        with google_calendar_token(self.env.user.sudo()) as token:
            if token:
                try:
                    google_service.patch(google_id, values, token=token, timeout=timeout)
                except HTTPError as e:
                    if e.response.status_code in (400, 403):
                        self._google_error_handling(e)
                if values:
                    self.exists().with_context(dont_notify=True).need_sync = False

    def _get_post_sync_values(self, request_values, google_values):
        """ Method to be override: select the information that will be written in the event after insertion. """
        self.ensure_one()
        return {'google_id': google_values['id'], 'need_sync': False}

    def write_insertion_values(self, request_values, google_values):
        """ Callback method: get post synchronization values and write them in the event. """
        writeable_values = self._get_post_sync_values(request_values, google_values)
        self.with_context(dont_notify=True).write(writeable_values)

    @after_commit
    def _google_insert(self, google_service: GoogleCalendarService, values, timeout=TIMEOUT):
        if not values:
            return
        with google_calendar_token(self.env.user.sudo()) as token:
            if token:
                try:
                    send_updates = self._context.get('send_updates', True)
                    google_service.google_service = google_service.google_service.with_context(send_updates=send_updates)
                    # Use callback method in stable branches to be called after insertion, updating the event right after insertion.
                    # This approach was needed because the insert function returns only the id information and discards event info from Google.
                    google_id = google_service.insert(values, token=token, timeout=timeout, callback_method=self.write_insertion_values)
                    if not self.google_id:
                        self.with_context(dont_notify=True).write({
                            'google_id': google_id,
                            'need_sync': False,
                        })
                except HTTPError as e:
                    if e.response.status_code in (400, 403):
                        self._google_error_handling(e)
                        self.with_context(dont_notify=True).need_sync = False

    def _get_records_to_sync(self, full_sync=False):
        """Return records that should be synced from Odoo to Google

        :param full_sync: If True, all events attended by the user are returned
        :return: events
        """
        domain = self._get_sync_domain()
        if not full_sync:
            is_active_clause = (self._active_name, '=', True) if self._active_name else expression.TRUE_LEAF
            domain = expression.AND([domain, [
                '|',
                    '&', ('google_id', '=', False), is_active_clause,
                    ('need_sync', '=', True),
            ]])
        # We want to limit to 200 event sync per transaction, it shouldn't be a problem for the day to day
        # but it allows to run the first synchro within an acceptable time without timeout.
        # If there is a lot of event to synchronize to google the first time,
        # they will be synchronized eventually with the cron running few times a day
        return self.with_context(active_test=False).search(domain, limit=200)

    def _check_any_records_to_sync(self):
        """ Returns True if there are pending records to be synchronized from Odoo to Google, False otherwise. """
        is_active_clause = (self._active_name, '=', True) if self._active_name else expression.TRUE_LEAF
        domain = expression.AND([self._get_sync_domain(), [
            '|',
                '&', ('google_id', '=', False), is_active_clause,
                ('need_sync', '=', True),
        ]])
        return self.search_count(domain, limit=1) > 0

    def _write_from_google(self, gevent, vals):
        self.write(vals)

    @api.model
    def _create_from_google(self, gevents, vals_list):
        return self.create(vals_list)

    @api.model
    def _get_sync_partner(self, emails):
        normalized_emails = [email_normalize(contact) for contact in emails if email_normalize(contact)]
        user_partners = self.env['mail.thread']._mail_search_on_user(normalized_emails, extra_domain=[('share', '=', False)])
        partners = list(user_partners)
        remaining = [email for email in normalized_emails if
                     email not in [partner.email_normalized for partner in partners]]
        if remaining:
            partners += self.env['mail.thread']._mail_find_partner_from_emails(remaining, records=self, force_create=True)
        unsorted_partners = self.env['res.partner'].browse([p.id for p in partners if p.id])
        # partners needs to be sorted according to the emails order provided by google
        k = {value: idx for idx, value in enumerate(emails)}
        result = unsorted_partners.sorted(key=lambda p: k.get(p.email_normalized, -1))
        return result

    @api.model
    def _odoo_values(self, google_event: GoogleEvent, default_reminders=()):
        """Implements this method to return a dict of Odoo values corresponding
        to the Google event given as parameter
        :return: dict of Odoo formatted values
        """
        raise NotImplementedError()

    def _google_values(self):
        """Implements this method to return a dict with values formatted
        according to the Google Calendar API
        :return: dict of Google formatted values
        """
        raise NotImplementedError()

    def _get_sync_domain(self):
        """Return a domain used to search records to synchronize.
        e.g. return a domain to synchronize records owned by the current user.
        """
        raise NotImplementedError()

    def _get_google_synced_fields(self):
        """Return a set of field names. Changing one of these fields
        marks the record to be re-synchronized.
        """
        raise NotImplementedError()

    @api.model
    def _restart_google_sync(self):
        """ Turns on the google synchronization for all the events of
        a given user.
        """
        raise NotImplementedError()

    def _get_event_user(self):
        """ Return the correct user to send the request to Google.
        It's possible that a user creates an event and sets another user as the organizer. Using self.env.user will
        cause some issues, and It might not be possible to use this user for sending the request, so this method gets
        the appropriate user accordingly.
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

    cal_client_id = fields.Char("Client_id", config_parameter='google_calendar_client_id', default='')
    cal_client_secret = fields.Char("Client_key", config_parameter='google_calendar_client_secret', default='')
    cal_sync_paused = fields.Boolean("Google Synchronization Paused", config_parameter='google_calendar_sync_paused',
        help="Indicates if synchronization with Google Calendar is paused or not.")

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging


from odoo import api, fields, models, Command
from odoo.addons.google_calendar.utils.google_calendar import GoogleCalendarService, InvalidSyncToken
from odoo.addons.google_calendar.models.google_sync import google_calendar_token
from odoo.loglevels import exception_to_unicode
from odoo.tools import str2bool

_logger = logging.getLogger(__name__)

class User(models.Model):
    _inherit = 'res.users'

    google_calendar_account_id = fields.Many2one('google.calendar.credentials')
    google_calendar_rtoken = fields.Char(related='google_calendar_account_id.calendar_rtoken', groups="base.group_system")
    google_calendar_token = fields.Char(related='google_calendar_account_id.calendar_token')
    google_calendar_token_validity = fields.Datetime(related='google_calendar_account_id.calendar_token_validity')
    google_calendar_sync_token = fields.Char(related='google_calendar_account_id.calendar_sync_token')
    google_calendar_cal_id = fields.Char(related='google_calendar_account_id.calendar_cal_id')
    google_synchronization_stopped = fields.Boolean(related='google_calendar_account_id.synchronization_stopped', readonly=False)

    _sql_constraints = [
        ('google_token_uniq', 'unique (google_calendar_account_id)', "The user has already a google account"),
    ]


    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + ['google_synchronization_stopped', 'google_calendar_account_id']

    @property
    def SELF_WRITEABLE_FIELDS(self):
        return super().SELF_WRITEABLE_FIELDS + ['google_synchronization_stopped', 'google_calendar_account_id']

    def _get_google_calendar_token(self):
        self.ensure_one()
        if self.google_calendar_account_id.calendar_rtoken and not self.google_calendar_account_id._is_google_calendar_valid():
            self.sudo().google_calendar_account_id._refresh_google_calendar_token()
        return self.google_calendar_account_id.calendar_token

    def _get_google_sync_status(self):
        """ Returns the calendar synchronization status (active, paused or stopped). """
        status = "sync_active"
        if str2bool(self.env['ir.config_parameter'].sudo().get_param("google_calendar_sync_paused"), default=False):
            status = "sync_paused"
        elif self.google_synchronization_stopped:
            status = "sync_stopped"
        return status

    def _check_pending_odoo_records(self):
        """ Returns True if sync is active and there are records to be synchronized to Google. """
        if self._get_google_sync_status() != "sync_active":
            return False
        pending_events = self.env['calendar.event']._check_any_records_to_sync()
        pending_recurrences = self.env['calendar.recurrence']._check_any_records_to_sync()
        return pending_events or pending_recurrences

    def _sync_google_calendar(self, calendar_service: GoogleCalendarService):
        self.ensure_one()
        results = self._sync_request(calendar_service)
        if not results or (not results.get('events') and not self._check_pending_odoo_records()):
            return False
        events, default_reminders, full_sync = results.values()
        # Google -> Odoo
        send_updates = not full_sync
        events.clear_type_ambiguity(self.env)
        recurrences = events.filter(lambda e: e.is_recurrence())

        # We apply Google updates only if their write date is later than the write date in Odoo.
        # It's possible that multiple updates affect the same record, maybe not directly.
        # To handle this, we preserve the write dates in Odoo before applying any updates,
        # and use these dates instead of the current live dates.
        odoo_events = self.env['calendar.event'].browse((events - recurrences).odoo_ids(self.env))
        odoo_recurrences = self.env['calendar.recurrence'].browse(recurrences.odoo_ids(self.env))
        recurrences_write_dates = {r.id: r.write_date for r in odoo_recurrences}
        events_write_dates = {e.id: e.write_date for e in odoo_events}
        synced_recurrences = self.env['calendar.recurrence'].with_context(write_dates=recurrences_write_dates)._sync_google2odoo(recurrences)
        synced_events = self.env['calendar.event'].with_context(write_dates=events_write_dates)._sync_google2odoo(events - recurrences, default_reminders=default_reminders)

        # Odoo -> Google
        recurrences = self.env['calendar.recurrence']._get_records_to_sync(full_sync=full_sync)
        recurrences -= synced_recurrences
        recurrences.with_context(send_updates=send_updates)._sync_odoo2google(calendar_service)
        synced_events |= recurrences.calendar_event_ids - recurrences._get_outliers()
        synced_events |= synced_recurrences.calendar_event_ids - synced_recurrences._get_outliers()
        events = self.env['calendar.event']._get_records_to_sync(full_sync=full_sync)
        (events - synced_events).with_context(send_updates=send_updates)._sync_odoo2google(calendar_service)

        return bool(results) and (bool(events | synced_events) or bool(recurrences | synced_recurrences))

    def _sync_single_event(self, calendar_service: GoogleCalendarService, odoo_event, event_id):
        self.ensure_one()
        results = self._sync_request(calendar_service, event_id)
        if not results or not results.get('events'):
            return False
        event, default_reminders, full_sync = results.values()
        # Google -> Odoo
        send_updates = not full_sync
        event.clear_type_ambiguity(self.env)
        synced_events = self.env['calendar.event']._sync_google2odoo(event, default_reminders=default_reminders)
        # Odoo -> Google
        odoo_event.with_context(send_updates=send_updates)._sync_odoo2google(calendar_service)
        return bool(odoo_event | synced_events)

    def _sync_request(self, calendar_service, event_id=None):
        if self._get_google_sync_status() != "sync_active":
            return False
        # don't attempt to sync when another sync is already in progress, as we wouldn't be
        # able to commit the transaction anyway (row is locked)
        self.env.cr.execute("""SELECT id FROM res_users WHERE id = %s FOR NO KEY UPDATE SKIP LOCKED""", [self.id])
        if not self.env.cr.rowcount:
            _logger.info("skipping calendar sync, locked user %s", self.login)
            return False

        full_sync = not bool(self.google_calendar_sync_token)
        with google_calendar_token(self) as token:
            try:
                if not event_id:
                    events, next_sync_token, default_reminders = calendar_service.get_events(self.google_calendar_account_id.calendar_sync_token, token=token)
                else:
                    # We force the sync_token parameter to avoid doing a full sync.
                    # Other events are fetched when the calendar view is displayed.
                    events, next_sync_token, default_reminders = calendar_service.get_events(sync_token=token, token=token, event_id=event_id)
            except InvalidSyncToken:
                events, next_sync_token, default_reminders = calendar_service.get_events(token=token)
                full_sync = True
        if next_sync_token:
            self.google_calendar_account_id.calendar_sync_token = next_sync_token
        return {
            'events': events,
            'default_reminders': default_reminders,
            'full_sync': full_sync,
        }

    @api.model
    def _sync_all_google_calendar(self):
        """ Cron job """
        users = self.env['res.users'].search([('google_calendar_rtoken', '!=', False), ('google_synchronization_stopped', '=', False)])
        google = GoogleCalendarService(self.env['google.service'])
        for user in users:
            _logger.info("Calendar Synchro - Starting synchronization for %s", user)
            try:
                user.with_user(user).sudo()._sync_google_calendar(google)
                self.env.cr.commit()
            except Exception as e:
                _logger.exception("[%s] Calendar Synchro - Exception : %s!", user, exception_to_unicode(e))
                self.env.cr.rollback()

    def is_google_calendar_synced(self):
        """ True if Google Calendar settings are filled (Client ID / Secret) and user calendar is synced
        meaning we can make API calls, false otherwise."""
        self.ensure_one()
        return self.google_calendar_token and self._get_google_sync_status() == 'sync_active'

    def stop_google_synchronization(self):
        self.ensure_one()
        self.google_synchronization_stopped = True

    def restart_google_synchronization(self):
        self.ensure_one()
        if not self.google_calendar_account_id:
            self.google_calendar_account_id = self.env['google.calendar.credentials'].sudo().create([{'user_ids': [Command.set(self.ids)]}])
        self.google_synchronization_stopped = False
        self.env['calendar.recurrence']._restart_google_sync()
        self.env['calendar.event']._restart_google_sync()

    def unpause_google_synchronization(self):
        self.env['ir.config_parameter'].sudo().set_param("google_calendar_sync_paused", False)

    def pause_google_synchronization(self):
        self.env['ir.config_parameter'].sudo().set_param("google_calendar_sync_paused", True)

    @api.model
    def check_calendar_credentials(self):
        res = super().check_calendar_credentials()
        get_param = self.env['ir.config_parameter'].sudo().get_param
        client_id = get_param('google_calendar_client_id')
        client_secret = get_param('google_calendar_client_secret')
        res['google_calendar'] = bool(client_id and client_secret)
        return res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_config_settings
from . import google_sync
from . import calendar
from . import calendar_recurrence_rule
from . import res_users
from . import calendar_attendee
from . import google_credentials

```

## File: security\google_calendar_security.xml

```xml
<odoo>
    <data noupdate="1">
        <record id="google_calendar_not_own_token_rule" model="ir.rule">
            <field name="name">Google Calendar NOT own token access rule</field>
            <field name="model_id" ref="model_google_calendar_credentials"/>
            <field name="domain_force">[('user_ids', 'not in', user.id)]</field>
            <field name="groups" eval="[(4, ref('base.group_user'))]"/>
            <field name="perm_create" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_read" eval="1"/>
        </record>
        <record id="google_calendar_own_token_rule" model="ir.rule">
            <field name="name">Google Calendar own token access rule</field>
            <field name="model_id" ref="model_google_calendar_credentials"/>
            <field name="domain_force">[('user_ids', 'in', user.id)]</field>
            <field name="groups" eval="[(4, ref('base.group_user'))]"/>
            <field name="perm_create" eval="0"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_unlink" eval="0"/>
            <field name="perm_read" eval="1"/>
        </record>
        <record id="google_calendar_token_system_access" model="ir.rule">
            <field name="name">Google Calendar manage all tokens</field>
            <field name="model_id" ref="model_google_calendar_credentials"/>
            <field name="domain_force">[(1,'=',1)]</field>
            <field name="groups" eval="[(4, ref('base.group_system'))]"/>
        </record>
    </data>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
google_calendar_account_reset,google_calendar_account_reset_access_right,model_google_calendar_account_reset,base.group_system,1,1,1,0
google_calendar.access_google_calendar_credentials,access_google_calendar_credentials,google_calendar.model_google_calendar_credentials,base.group_user,1,1,0,0
google_calendar_manager,access_google_calendar_credentials_manager,google_calendar.model_google_calendar_credentials,base.group_system,1,1,0,0

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="m36.053 13.947-9.948-1.105-12.158 1.105L12.842 25l1.105 11.053L25 37.435l11.053-1.382 1.105-11.329-1.105-10.777Z" fill="#fff"/><path d="M18.482 31.096c-.826-.558-1.398-1.373-1.71-2.451l1.917-.79c.174.663.478 1.177.912 1.542.431.364.956.544 1.57.544.627 0 1.166-.19 1.616-.572.45-.381.677-.868.677-1.456 0-.603-.238-1.095-.713-1.476-.475-.38-1.072-.572-1.785-.572h-1.108v-1.898h.995c.613 0 1.13-.166 1.55-.497.42-.332.63-.785.63-1.362 0-.514-.188-.924-.564-1.23-.376-.307-.85-.462-1.428-.462-.564 0-1.012.15-1.343.451a2.641 2.641 0 0 0-.724 1.108l-1.898-.79c.25-.713.712-1.343 1.39-1.888.676-.544 1.541-.817 2.591-.817.777 0 1.476.149 2.095.45a3.584 3.584 0 0 1 1.456 1.249c.35.533.525 1.13.525 1.793 0 .677-.163 1.249-.49 1.719-.325.47-.726.829-1.201 1.08v.113a3.65 3.65 0 0 1 1.541 1.202c.401.54.603 1.183.603 1.935 0 .751-.19 1.423-.572 2.011a3.96 3.96 0 0 1-1.578 1.39c-.671.337-1.426.508-2.263.508a4.684 4.684 0 0 1-2.691-.834ZM30.25 21.58l-2.095 1.522-1.053-1.597 3.778-2.724h1.448v12.851H30.25V21.58Z" fill="#1A73E8"/><path d="M36.052 46 46 36.053l-4.973-2.21-4.974 2.21-2.21 4.973L36.052 46Z" fill="#EA4335"/><path d="M11.737 41.026 13.947 46h22.106v-9.947H13.948l-2.21 4.973Z" fill="#34A853"/><path d="M7.316 4A3.315 3.315 0 0 0 4 7.316v28.737l4.974 2.21 4.973-2.21V13.947h22.106l2.21-4.973L36.053 4H7.316Z" fill="#4285F4"/><path d="M4 36.053v6.631A3.315 3.315 0 0 0 7.316 46h6.631v-9.947H4Z" fill="#188038"/><path d="M36.053 13.947v22.106H46V13.947l-4.974-2.21-4.973 2.21Z" fill="#FBBC04"/><path d="M46 13.947V7.316A3.315 3.315 0 0 0 42.684 4h-6.631v9.947H46Z" fill="#1967D2"/></svg>

```

## File: static\description\index.html

```html
<section class="oe_container">
    <div class="oe_row oe_spaced">
        <h2 class="oe_slogan">Google Calendar</h2>
        <h3 class="oe_slogan">Get your meetings, your leaves...  Get your calendar anywhere and never forget an event.</h3>
        <div class="oe_span12">
            <img src="the_calendar.png" class="oe_picture oe_screenshot">                            
        </div>        
    </div>
</section>

<section class="oe_container oe_dark">
    <div class="oe_row">
         <h2 class="oe_slogan">Keep an eye on your events</h2>
         <div class="oe_span6">
            <p class='oe_mt32'>
				See easily the purpose of the meeting, the start time and also the attendee(s)... All that without click on anything...
			</p>			
        </div>
        <div class="oe_span6">
            <img class="oe_picture oe_screenshot" src="an_event.png">
        </div>
    </div>
</section>

<section class="oe_container">
    <div class="oe_row">
         <h2 class="oe_slogan">Create so easily an event</h2>
         <div class="oe_span6">
            <img class="oe_picture oe_screenshot" src="create_quick.png">
         </div>
         <div class="oe_span6">
            <p class='oe_mt32'>
				In just one click you can create an event...<br/>
				You can drag and drop your event if you want moved it to another timing.<br/>
				You can shrink or extend the event if you need to change the start's hours or the duration of your meeting.
			</p>			
         </div>        
    </div>
</section>

<section class="oe_container oe_dark">
    <div class="oe_row">
         <h2 class="oe_slogan">Create recurrent event</h2>
         <div class="oe_span6">
            <p class='oe_mt32'>
				You can also create recurrent events with only one event.<br/>
				You need to create an event each monday of the week ? With only one it's possible, you could specify the recurrence and if one of this event is moved, or deleted, it's not a problem, you can untie your event from the others recurrences.
			</p>			
         </div>
         <div class="oe_span6">
            <img class="oe_picture oe_screenshot" src="recurrent.png">
         </div>
        
    </div>
</section>

<section class="oe_container ">
    <div class="oe_row">
         <h2 class="oe_slogan">See all events you wants </h2>
         <div class="oe_span6">
            <img class="oe_picture oe_screenshot" src="coworker.png">
        </div>
        <div class="oe_span6">
            <p class='oe_mt32'>
				See in your calendar, the event from others peoples where your are attendee, but also their events by simply adding your favorites coworkers.<br/>
				Every coworker will have their own color in your calendar, and every attendee will have their avatar in the event...<br/>
			</p>			
        </div>
        
    </div>
</section>

<section class="oe_container oe_dark">
    <div class="oe_row">
         <h2 class="oe_slogan">Get an email</h2>
         <div class="oe_span6">
            <p class='oe_mt32'>
				You will receive an email at creation of an event where you are attendee, but also when this event is updated for some fields as date start, ...
			</p>			
         </div>
         <div class="oe_span6">
            <img class="oe_picture oe_screenshot" src="email.png">
         </div>
        
    </div>
</section>

<section class="oe_container ">
    <div class="oe_row">
         <h2 class="oe_slogan">Be notified </h2>
         <div class="oe_span6">
            <img class="oe_picture oe_screenshot" src="notification.png">
        </div>
        <div class="oe_span6">
            <p class='oe_mt32'>
				You can ask to have a alarm of type 'notification' in your Odoo.<br/>
				You will have a notification in you Odoo which ever the page you are.
			</p>			
        </div>
        
    </div>
</section>
<section class="oe_container oe_dark">
    <div class="oe_row">
        <h2 class="oe_slogan">Google Calendar</h2>
        <div class="oe_span6">
            <p class='oe_mt32'>
				With the plugin Google_calendar, you can synchronize your Odoo calendar with Google Calendar.
            </p>
        </div>
        <div class="oe_span6">
            <img class="oe_picture oe_screenshot" src="calendar_in_action.png">
        </div>
        
    </div>
</section>

```

## File: static\src\views\google_calendar\google_calendar_controller.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { AttendeeCalendarController } from "@calendar/views/attendee_calendar/attendee_calendar_controller";
import { patch } from "@web/core/utils/patch";
import { useService } from "@web/core/utils/hooks";
import { ConfirmationDialog, AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";

patch(AttendeeCalendarController.prototype, {
    setup() {
        super.setup(...arguments);
        this.dialog = useService("dialog");
        this.notification = useService("notification");
        this.rpc = useService("rpc");
    },

    async onGoogleSyncCalendar() {
        await this.orm.call(
            "res.users",
            "restart_google_synchronization",
            [[this.user.userId]],
        );
        const syncResult = await this.model.syncGoogleCalendar();
        if (syncResult.status === "need_auth") {
            window.location.assign(syncResult.url);
        } else if (syncResult.status === "need_config_from_admin") {
            if (this.isSystemUser) {
                this.dialog.add(ConfirmationDialog, {
                    title: _t("Configuration"),
                    body: _t("The Google Synchronization needs to be configured before you can use it, do you want to do it now?"),
                    confirm: this.actionService.doAction.bind(this.actionService, syncResult.action),
                    confirmLabel: _t("Configure"),
                    cancel: () => {},
                    cancelLabel: _t("Discard"),
                });
            } else {
                this.dialog.add(AlertDialog, {
                    title: _t("Configuration"),
                    body: _t("An administrator needs to configure Google Synchronization before you can use it!"),
                });
            }
        } else if (syncResult.status === "need_refresh") {
            await this.model.load();
        }
    },

    async onStopGoogleSynchronization() {
        this.dialog.add(ConfirmationDialog, {
            body: _t("You are about to stop the synchronization of your calendar with Google. Are you sure you want to continue?"),
            confirmLabel: _t("Stop Synchronization"),
            confirm: async () => {
                await this.orm.call(
                    "res.users",
                    "stop_google_synchronization",
                    [[this.user.userId]],
                );
                this.notification.add(
                    _t("The synchronization with Google calendar was successfully stopped."),
                    {
                        title: _t("Success"),
                        type: "success",
                    },
                );
                await this.model.load();
            },
        });
    },

    onGoogleSyncUnpause() {
        if (this.isSystemUser) {
            this.env.services.action.doAction("calendar.calendar_settings_action");
        } else {
            this.dialog.add(AlertDialog, {
                title: _t("Configuration"),
                body: _t("Your administrator paused the synchronization with Google Calendar."),
            });
        }
    }
});

```

## File: static\src\views\google_calendar\google_calendar_controller.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="google_calendar.GoogleCalendarController" t-inherit="calendar.AttendeeCalendarController" t-inherit-mode="extension">
        <FilterPanel position="after">
            <div id="google_calendar_sync" class="o_calendar_sync" t-if="model.googleCredentialsSet">
                <button t-if="!model.state.googleIsSync and model.state.googleIsPaused" type="button" id="google_sync_paused" class="o_google_sync_button o_google_sync_paused btn btn-warning" t-on-click="onGoogleSyncUnpause">
                    <b>
                        <i id="google_paused" class='fa fa-pause'/>
                        <span class="mx-1">Google</span>
                    </b>
                </button>
                <button t-elif="!model.state.googleIsSync" type="button" id="google_sync_pending" class="o_google_sync_button o_google_sync_pending btn btn-secondary text-nowrap" t-on-click="onGoogleSyncCalendar">
                    <b><i class='fa fa-refresh'/> Google</b>
                </button>
                <!-- class change on hover -->
                <button t-else="" type="button" id="google_sync_configured" class="me-1 o_google_sync_button o_google_sync_button_configured btn btn-outline-primary" t-on-click="onStopGoogleSynchronization"
                    t-on-mouseenter="(ev) => {ev.target.classList.remove('btn-outline-primary');ev.target.classList.add('btn-danger');}"
                    t-on-mouseleave="(ev) => {ev.target.classList.remove('btn-danger');ev.target.classList.add('btn-outline-primary');}">
                    <b>
                        <i id="google_check" class='fa fa-check fa-fw'/>
                        <i id="google_stop" class='fa fa-times fa-fw'/>
                        <span class="mx-1">Google</span>
                    </b>
                </button>
            </div>
        </FilterPanel>
    </t>
</templates>

```

## File: static\src\views\google_calendar\google_calendar_model.js

```javascript
/** @odoo-module **/

import { AttendeeCalendarModel } from "@calendar/views/attendee_calendar/attendee_calendar_model";
import { patch } from "@web/core/utils/patch";
import { useState } from "@odoo/owl";

patch(AttendeeCalendarModel, {
    services: [...AttendeeCalendarModel.services, "rpc"],
});

patch(AttendeeCalendarModel.prototype, {
    setup(params, { rpc }) {
        super.setup(...arguments);
        this.rpc = rpc;
        this.isAlive = params.isAlive;
        this.googlePendingSync = false;
        this.state = useState({
            googleIsSync: true,
            googleIsPaused: false,
        });
    },

    /**
     * @override
     */
    async updateData() {
        if (this.googlePendingSync) {
            return super.updateData(...arguments);
        }
        try {
            await Promise.race([
                new Promise(resolve => setTimeout(resolve, 1000)),
                this.syncGoogleCalendar(true)
            ]);
        } catch (error) {
            if (error.event) {
                error.event.preventDefault();
            }
            console.error("Could not synchronize Google events now.", error);
            this.googlePendingSync = false;
        }
        if (this.isAlive()) {
            return super.updateData(...arguments);
        }
        return new Promise(() => {});
    },

    async syncGoogleCalendar(silent = false) {
        this.googlePendingSync = true;
        const result = await this.rpc(
            "/google_calendar/sync_data",
            {
                model: this.resModel,
                fromurl: window.location.href
            },
            {
                silent,
            },
        );
        if (["need_config_from_admin", "need_auth", "sync_stopped", "sync_paused"].includes(result.status)) {
            this.state.googleIsSync = false;
        } else if (result.status === "no_new_event_from_google" || result.status === "need_refresh") {
            this.state.googleIsSync = true;
        }
        this.state.googleIsPaused = result.status == "sync_paused";
        this.googlePendingSync = false;
        return result;
    },

    get googleCredentialsSet() {
        return this.credentialStatus['google_calendar'] ?? false;
    }
});

```

## File: static\src\views\google_calendar\common\google_calendar_common_popover.js

```javascript
/** @odoo-module **/

import { AttendeeCalendarCommonPopover } from "@calendar/views/attendee_calendar/common/attendee_calendar_common_popover";
import { patch } from "@web/core/utils/patch";

patch(AttendeeCalendarCommonPopover.prototype, {
    get isEventArchivable() {
        return super.isEventArchivable || (this.isCurrentUserOrganizer && this.props.record.rawRecord.google_id);
    },
});

```

## File: utils\google_calendar.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from uuid import uuid4
import requests
import json
import logging

from odoo import fields
from odoo.addons.google_calendar.utils.google_event import GoogleEvent
from odoo.addons.google_account.models.google_service import TIMEOUT


_logger = logging.getLogger(__name__)

def requires_auth_token(func):
    def wrapped(self, *args, **kwargs):
        if not kwargs.get('token'):
            raise AttributeError("An authentication token is required")
        return func(self, *args, **kwargs)
    return wrapped

class InvalidSyncToken(Exception):
    pass

class GoogleCalendarService():

    def __init__(self, google_service):
        self.google_service = google_service

    @requires_auth_token
    def get_events(self, sync_token=None, token=None, event_id=None, timeout=TIMEOUT):
        url = "/calendar/v3/calendars/primary/events"
        if event_id:
            url += f"/{event_id}"
        headers = {'Content-type': 'application/json'}
        params = {'access_token': token}
        if sync_token:
            params['syncToken'] = sync_token
        else:
            # full sync, limit to a range of 1y in past to 1y in the futur by default
            ICP = self.google_service.env['ir.config_parameter'].sudo()
            day_range = int(ICP.get_param('google_calendar.sync.range_days', default=365))
            _logger.info("Full cal sync, restricting to %s days range", day_range)
            lower_bound = fields.Datetime.subtract(fields.Datetime.now(), days=day_range)
            upper_bound = fields.Datetime.add(fields.Datetime.now(), days=day_range)
            params['timeMin'] = lower_bound.isoformat() + 'Z'  # Z = UTC (RFC3339)
            params['timeMax'] = upper_bound.isoformat() + 'Z'  # Z = UTC (RFC3339)
        try:
            status, data, time = self.google_service._do_request(url, params, headers, method='GET', timeout=timeout)
        except requests.HTTPError as e:
            if e.response.status_code == 410 and 'fullSyncRequired' in str(e.response.content):
                raise InvalidSyncToken("Invalid sync token. Full sync required")
            raise e

        if event_id:
            next_sync_token = None
            default_reminders = ()
            return GoogleEvent([data]), next_sync_token, default_reminders

        events = data.get('items', [])
        next_page_token = data.get('nextPageToken')
        while next_page_token:
            params = {'access_token': token, 'pageToken': next_page_token}
            status, data, time = self.google_service._do_request(url, params, headers, method='GET', timeout=timeout)
            next_page_token = data.get('nextPageToken')
            events += data.get('items', [])

        next_sync_token = data.get('nextSyncToken')
        default_reminders = data.get('defaultReminders')

        return GoogleEvent(events), next_sync_token, default_reminders

    @requires_auth_token
    def insert(self, values, token=None, timeout=TIMEOUT, callback_method=None):
        send_updates = self.google_service._context.get('send_updates', True)
        url = "/calendar/v3/calendars/primary/events?conferenceDataVersion=1&sendUpdates=%s" % ("all" if send_updates else "none")
        headers = {'Content-type': 'application/json', 'Authorization': 'Bearer %s' % token}
        if not values.get('id'):
            values['id'] = uuid4().hex
        request_values = self.google_service._do_request(url, json.dumps(values), headers, method='POST', timeout=timeout)
        # Execute optional callback function using the returned values from insertion. TODO (gdpf) refactor in master.
        if callable(callback_method):
            callback_method(request_values, values)
        return values['id']

    @requires_auth_token
    def patch(self, event_id, values, token=None, timeout=TIMEOUT):
        url = "/calendar/v3/calendars/primary/events/%s?sendUpdates=all" % event_id
        headers = {'Content-type': 'application/json', 'Authorization': 'Bearer %s' % token}
        self.google_service._do_request(url, json.dumps(values), headers, method='PATCH', timeout=timeout)

    @requires_auth_token
    def delete(self, event_id, token=None, timeout=TIMEOUT):
        url = "/calendar/v3/calendars/primary/events/%s?sendUpdates=all" % event_id
        headers = {'Content-type': 'application/json'}
        params = {'access_token': token}
        # Delete all events from recurrence in a single request to Google and triggering a single mail.
        # The 'singleEvents' parameter is a trick that tells Google API to delete all recurrent events individually,
        # making the deletion be handled entirely on their side, and then we archive the events in Odoo.
        is_recurrence = self.google_service._context.get('is_recurrence', True)
        if is_recurrence:
            params['singleEvents'] = 'true'
        try:
            self.google_service._do_request(url, params, headers=headers, method='DELETE', timeout=timeout)
        except requests.HTTPError as e:
            # For some unknown reason Google can also return a 403 response when the event is already cancelled.
            if e.response.status_code not in (410, 403):
                raise e
            _logger.info("Google event %s was already deleted" % event_id)


    #################################
    ##  MANAGE CONNEXION TO GMAIL  ##
    #################################


    def is_authorized(self, user):
        return bool(user.sudo().google_calendar_rtoken)

    def _get_calendar_scope(self, RO=False):
        readonly = '.readonly' if RO else ''
        return 'https://www.googleapis.com/auth/calendar%s' % (readonly)

    def _google_authentication_url(self, from_url='http://www.odoo.com'):
        state = {
            'd': self.google_service.env.cr.dbname,
            's': 'calendar',
            'f': from_url
        }
        base_url = self.google_service._context.get('base_url') or self.google_service.get_base_url()
        return self.google_service._get_authorize_uri(
            'calendar',
            self._get_calendar_scope(),
            base_url + '/google_account/authentication',
            state=json.dumps(state),
            approval_prompt='force',
            access_type='offline'
        )

    def _can_authorize_google(self, user):
        return user.has_group('base.group_erp_manager')

```

## File: utils\google_event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import json

from odoo.tools import email_normalize, ReadonlyDict
import logging
from typing import Iterator, Mapping
from collections import abc
import re

_logger = logging.getLogger(__name__)


class GoogleEvent(abc.Set):
    """This helper class holds the values of a Google event.
    Inspired by Odoo recordset, one instance can be a single Google event or a
    (immutable) set of Google events.
    All usual set operations are supported (union, intersection, etc).

    A list of all attributes can be found in the API documentation.
    https://developers.google.com/calendar/v3/reference/events#resource

    :param iterable: iterable of GoogleCalendar instances or iterable of dictionnaries

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

    def __iter__(self) ->  Iterator['GoogleEvent']:
        return iter(GoogleEvent([vals]) for vals in self._events.values())

    def __contains__(self, google_event):
        return google_event.id in self._events

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
        value = self._events[event_id].get(name)
        json.dumps(value)
        return value

    def __repr__(self):
        return '%s%s' % (self.__class__.__name__, self.ids)

    @property
    def ids(self):
        return tuple(e.id for e in self)

    @property
    def rrule(self):
        if self.recurrence and any('RRULE' in item for item in self.recurrence):
            return next(item for item in self.recurrence if 'RRULE' in item)

    def odoo_id(self, env):
        self.odoo_ids(env)  # load ids
        return self._odoo_id

    def _meta_odoo_id(self, dbname):
        """Returns the Odoo id stored in the Google Event metadata.
        This id might not actually exists in the database.
        """
        properties = self.extendedProperties and (self.extendedProperties.get('shared', {}) or self.extendedProperties.get('private', {})) or {}
        o_id = properties.get('%s_odoo_id' % dbname)
        if o_id:
            return int(o_id)

    def odoo_ids(self, env):
        ids = tuple(e._odoo_id for e in self if e._odoo_id)
        if len(ids) == len(self):
            return ids
        model = self._get_model(env)
        found = self._load_odoo_ids_from_db(env, model)
        unsure = self - found
        if unsure:
            unsure._load_odoo_ids_from_metadata(env, model)

        return tuple(e._odoo_id for e in self)

    def _load_odoo_ids_from_metadata(self, env, model):
        unsure_odoo_ids = tuple(e._meta_odoo_id(env.cr.dbname) for e in self)
        odoo_events = model.browse(_id for _id in unsure_odoo_ids if _id)

        # Extended properties are copied when splitting a recurrence Google side.
        # Hence, we may have two Google recurrences linked to the same Odoo id.
        # Therefore, we only consider Odoo records without google id when trying
        # to match events.
        o_ids = odoo_events.exists().filtered(lambda e: not e.google_id).ids
        for e in self:
            odoo_id = e._meta_odoo_id(env.cr.dbname)
            if odoo_id in o_ids:
                e._events[e.id]['_odoo_id'] = odoo_id

    def _load_odoo_ids_from_db(self, env, model):
        odoo_events = model.with_context(active_test=False)._from_google_ids(self.ids)
        mapping = {e.google_id: e.id for e in odoo_events}  # {google_id: odoo_id}
        existing_google_ids = odoo_events.mapped('google_id')
        for e in self:
            odoo_id = mapping.get(e.id)
            if odoo_id:
                e._events[e.id]['_odoo_id'] = odoo_id
        return self.filter(lambda e: e.id in existing_google_ids)


    def owner(self, env):
        # Owner/organizer could be desynchronised between Google and Odoo.
        # Let userA, userB be two new users (never synced to Google before).
        # UserA creates an event in Odoo (they are the owner) but userB syncs first.
        # There is no way to insert the event into userA's calendar since we don't have
        # any authentication access. The event is therefore inserted into userB's calendar
        # (they are the organizer in Google). The "real" owner (in Odoo) is stored as an
        # extended property. There is currently no support to "transfert" ownership when
        # userA syncs their calendar the first time.
        real_owner_id = self.extendedProperties and self.extendedProperties.get('shared', {}).get('%s_owner_id' % env.cr.dbname)
        try:
            # If we create an event without user_id, the event properties will be 'false'
            # and python will interpret this a a NoneType, that's why we have the 'except TypeError'
            real_owner_id = int(real_owner_id)
        except (ValueError, TypeError):
            real_owner_id = False
        real_owner = real_owner_id and env['res.users'].browse(real_owner_id) or env['res.users']
        if real_owner_id and real_owner.exists():
            return real_owner
        elif self.organizer and self.organizer.get('self'):
            return env.user
        elif self.organizer and self.organizer.get('email'):
            # In Google: 1 email = 1 user; but in Odoo several users might have the same email :/
            org_email = email_normalize(self.organizer.get('email'))
            return env['res.users'].search([('email_normalized', '=', org_email)], limit=1)
        else:
            return env['res.users']

    def filter(self, func) -> 'GoogleEvent':
        return GoogleEvent(e for e in self if func(e))

    def clear_type_ambiguity(self, env):
        ambiguous_events = self.filter(GoogleEvent._is_type_ambiguous)
        recurrences = ambiguous_events._load_odoo_ids_from_db(env, env['calendar.recurrence'])
        for recurrence in recurrences:
            self._events[recurrence.id]['recurrence'] = True
        for event in ambiguous_events - recurrences:
            self._events[event.id]['recurrence'] = False

    def is_recurrence(self):
        if self._is_type_ambiguous():
            _logger.warning("Ambiguous event type: cannot accurately tell whether a cancelled event is a recurrence or not")
        return bool(self.recurrence)

    def is_recurrent(self):
        return bool(self.recurringEventId or self.is_recurrence())

    def is_cancelled(self):
        return self.status == 'cancelled'

    def is_recurrence_follower(self):
        return bool(not self.originalStartTime or self.originalStartTime == self.start)

    def full_recurring_event_id(self):
        """
            Give the complete identifier with elements
            in `id` and `recurringEventId`.
            :return: concatenation of the id created by the recurrence
                    and the id created by the modification of a specific event
            :rtype: string if recurrent event and correct ids, `None` otherwise
        """
        # Regex expressions to match elements (according to the google support [not documented]):
        # - ID: [a-zA-Z0-9]+
        # - RANGE: R[0-9]+T[0-9]+
        # - TIMESTAMP: [0-9]+T[0-9]+Z
        # With:
        # - id: 'ID_TIMESTAMP'
        # - recurringEventID: 'ID_RANGE'
        # Find: 'ID_RANGE_TIMESTAMP'
        if not self.is_recurrent():
            return None
        # Check if ids are the same
        id_value = re.match(r'(\w+_)', self.id)
        recurringEventId_value = re.match(r'(\w+_)', self.recurringEventId)
        if not id_value or not recurringEventId_value or id_value.group(1) != recurringEventId_value.group(1):
            return None
        rec_pattern = re.search(r'(\w+_R\d+(?:T\d+)?(?:Z)?)', self.recurringEventId)
        if not rec_pattern:
            return None
        id_range = rec_pattern.group()

        ts_pattern = re.search(r'(\d{8}(?:T\d+Z?)?)$', self.id)
        if not ts_pattern:
            return None
        timestamp = ts_pattern.group()

        return f"{id_range}_{timestamp}"

    def cancelled(self):
        return self.filter(lambda e: e.status == 'cancelled')

    def exists(self, env) -> 'GoogleEvent':
        recurrences = self.filter(GoogleEvent.is_recurrence)
        events = self - recurrences
        recurrences.odoo_ids(env)
        events.odoo_ids(env)

        return self.filter(lambda e: e._odoo_id)

    def _is_type_ambiguous(self):
        """For cancelled events/recurrences, Google only send the id and
        the cancelled status. There is no way to know if it was a recurrence
        or simple event."""
        return self.is_cancelled() and 'recurrence' not in self._events[self.id]

    def _get_model(self, env):
        if all(e.is_recurrence() for e in self):
            return env['calendar.recurrence']
        if all(not e.is_recurrence() for e in self):
            return env['calendar.event']
        raise TypeError("Mixing Google events and Google recurrences")

    def get_meeting_url(self):
        if not self.conferenceData:
            return False
        video_meeting = list(filter(lambda entryPoints: entryPoints.get('entryPointType') == 'video', self.conferenceData.get('entryPoints', [])))
        return video_meeting[0].get('uri') if video_meeting else False

    def is_available(self):
        return self.transparency == 'transparent'

    def get_odoo_event(self, env):
        if self._get_model(env)._name == 'calendar.event':
            return env['calendar.event'].browse(self.odoo_id(self.env))
        else:
            return env['calendar.recurrence'].browse(self.odoo_id(self.env)).base_event_id

```

## File: utils\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import google_calendar
from . import google_event

```

## File: views\google_calendar_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="view_google_calendar_event" model="ir.ui.view">
        <field name="name">google_calendar.event.calendar</field>
        <field name="model">calendar.event</field>
        <field name="inherit_id" ref="calendar.view_calendar_event_calendar"/>
        <field name="arch" type="xml">
            <field name="location" position="after">
                <field name="google_id" invisible="1"/>
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
            <field name="name">res.config.settings.view.form.inherit.google.calendar</field>
            <field name="model">res.config.settings</field>
            <field name="inherit_id" ref="calendar.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <div id="msg_module_google_calendar" position="replace">
                    <div class="content-group" invisible="not module_google_calendar">
                        <div class="mt16 row">
                            <label for="cal_client_id" string="Client ID" class="col-3 col-lg-3 o_light_label"/>
                            <field name="cal_client_id" nolabel="1"/>
                        </div>
                        <div class="mt16 row">
                            <label for="cal_client_secret" string="Client Secret" class="col-3 col-lg-3 o_light_label"/>
                            <field name="cal_client_secret" password="True" nolabel="1"/>
                        </div>
                        <div class="mt16 d-flex">
                            <label for="cal_sync_paused" string="Pause Synchronization" class="o_light_label"/>
                            <field name="cal_sync_paused" class="ml16"/>
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
            <field name="inherit_id" ref="calendar.res_users_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//page[@name='calendar']" position='attributes'>
                    <attribute name="invisible">0</attribute>
                </xpath>
                <group name="calendar_accounts" position="inside">
                    <group string="Google Calendar">
                        <group class="text-break">
                            <field name="google_calendar_rtoken" readonly="1" groups="base.group_system"/>
                            <field name="google_calendar_token" readonly="1"/>
                            <field name="google_calendar_token_validity" readonly="1"/>
                            <field name="google_calendar_sync_token" readonly="1"/>
                            <field name="google_calendar_cal_id" readonly="1"/>
                            <button
                                string="Reset Account" colspan="2" class="btn btn-secondary mt-3"
                                type="action" name="%(google_calendar_reset_account_action)d"
                                context="{'default_user_id': id}"/>
                        </group>
                    </group>
                </group>
            </field>
        </record>

</odoo>

```

## File: wizard\reset_account.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

from odoo.addons.google_calendar.models.google_sync import google_calendar_token
from odoo.addons.google_calendar.utils.google_calendar import GoogleCalendarService


class ResetGoogleAccount(models.TransientModel):
    _name = 'google.calendar.account.reset'
    _description = 'Google Calendar Account Reset'

    user_id = fields.Many2one('res.users', required=True)
    delete_policy = fields.Selection(
        [('dont_delete', "Leave them untouched"),
         ('delete_google', "Delete from the current Google Calendar account"),
         ('delete_odoo', "Delete from Odoo"),
         ('delete_both', "Delete from both"),
        ], string="User's Existing Events", required=True, default='dont_delete',
        help="This will only affect events for which the user is the owner")
    sync_policy = fields.Selection([
        ('new', "Synchronize only new events"),
        ('all', "Synchronize all existing events"),
    ], string="Next Synchronization", required=True, default='new')

    def reset_account(self):
        google = GoogleCalendarService(self.env['google.service'])

        events = self.env['calendar.event'].search([
            ('user_id', '=', self.user_id.id),
            ('google_id', '!=', False)])
        recurrences = self.env['calendar.recurrence'].search([
            ('base_event_id', 'in', events.ids),
            ('google_id', '!=', False)])

        if self.delete_policy in ('delete_google', 'delete_both'):
            with google_calendar_token(self.user_id) as token:
                for event in events:
                    google.delete(event.google_id, token=token)

        # Delete events according to the selected policy. If the deletion is only in
        # Google, we won't keep track of the 'google_id' field for events and recurrences.
        if self.delete_policy in ('delete_odoo', 'delete_both', 'delete_google'):
            events.google_id = False
            recurrences.google_id = False
            if self.delete_policy != 'delete_google':
                events.unlink()

        # Define which events must be synchronized in the next synchronization:
        # in 'all' sync policy we activate the sync for all events and in the 'new'
        # sync policy we set it as False for skipping existing events synchronization.
        next_sync_update = {}
        if self.sync_policy == 'all':
            next_sync_update['need_sync'] = True
        elif self.sync_policy == 'new':
            next_sync_update['need_sync'] = False

        # Write the next sync update attribute on the existing events.
        if self.delete_policy not in ('delete_odoo', 'delete_both'):
            events.write(next_sync_update)

        self.user_id.google_calendar_account_id._set_auth_tokens(False, False, 0)
        self.user_id.write({
            'google_calendar_sync_token': False,
            'google_calendar_cal_id': False,
        })

```

## File: wizard\reset_account_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="google_calendar_reset_account_view_form" model="ir.ui.view">
        <field name="name">google.calendar.account.reset.form</field>
        <field name="model">google.calendar.account.reset</field>
        <field name="arch" type="xml">
            <form>
                <h1>Reset Google Calendar Account</h1>
                <group>
                    <field name="delete_policy" widget="radio"/>
                    <field name="sync_policy" widget="radio"/>
                </group>

                <footer>
                    <button name="reset_account" string="Confirm" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="x" />
                </footer>
            </form>
        </field>
    </record>

    <record id="google_calendar_reset_account_action" model="ir.actions.act_window">
        <field name="name"></field>
        <field name="res_model">google.calendar.account.reset</field>
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

