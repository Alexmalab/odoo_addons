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
    'description': "",
    'depends': ['google_account', 'calendar'],
    'data': [
        'data/google_calendar_data.xml',
        'security/ir.model.access.csv',
        'wizard/reset_account_views.xml',
        'views/res_config_settings_views.xml',
        'views/res_users_views.xml',
        'views/google_calendar_views.xml',
        ],
    'demo': [],
    'installable': True,
    'auto_install': False,
    'assets': {
        'web.assets_backend': [
            'google_calendar/static/src/js/google_calendar_popover.js',
            'google_calendar/static/src/js/google_calendar.js',
            'google_calendar/static/src/scss/google_calendar.scss',
        ],
        'web.qunit_suite_tests': [
            'google_calendar/static/tests/**/*',
        ],
        'web.qunit_mobile_suite_tests': [
            'google_calendar/static/tests/mock_server.js',
        ],
        'web.assets_qweb': [
            'google_calendar/static/src/xml/*.xml',
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


class GoogleCalendarController(http.Controller):

    @http.route('/google_calendar/sync_data', type='json', auth='user')
    def sync_data(self, model, **kw):
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
            client_id = request.env['ir.config_parameter'].sudo().get_param('google_calendar_client_id')

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

            # If synchronization has been stopped
            if not need_refresh and request.env.user.google_synchronization_stopped:
                return {
                    "status": "sync_stopped",
                    "url": ''
                }
            return {
                "status": "need_refresh" if need_refresh else "no_new_event_from_google",
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

## File: models\calendar.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import pytz
from dateutil.parser import parse
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, tools, _


class Meeting(models.Model):
    _name = 'calendar.event'
    _inherit = ['calendar.event', 'google.calendar.sync']

    google_id = fields.Char(
        'Google Calendar Event Id', compute='_compute_google_id', store=True, readonly=False)

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

    @api.model
    def _get_google_synced_fields(self):
        return {'name', 'description', 'allday', 'start', 'date_end', 'stop',
                'attendee_ids', 'alarm_ids', 'location', 'privacy', 'active'}

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

    def write(self, values):
        recurrence_update_setting = values.get('recurrence_update')
        if recurrence_update_setting in ('all_events', 'future_events') and len(self) == 1:
            values = dict(values, need_sync=False)
        notify_context = self.env.context.get('dont_notify', False)
        res = super(Meeting, self.with_context(dont_notify=notify_context)).write(values)
        if recurrence_update_setting in ('all_events',) and len(self) == 1 and values.keys() & self._get_google_synced_fields():
            self.recurrence_id.need_sync = True
        return res

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
            'show_as': 'free' if google_event.is_available() else 'busy'
        }
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
                'responseStatus': 'needsAction',
            }]
        emails = [a.get('email') for a in google_attendees]
        existing_attendees = self.env['calendar.attendee']
        if google_event.exists(self.env):
            event = google_event.get_odoo_event(self.env)
            existing_attendees = event.attendee_ids
        attendees_by_emails = {tools.email_normalize(a.email): a for a in existing_attendees}
        partners = self.env['mail.thread']._mail_find_partner_from_emails(emails, records=self, force_create=True, extra_domain=[('type', '!=', 'private')])
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

    def _google_values(self):
        if self.allday:
            start = {'date': self.start_date.isoformat()}
            end = {'date': (self.stop_date + relativedelta(days=1)).isoformat()}
        else:
            start = {'dateTime': pytz.utc.localize(self.start).isoformat()}
            end = {'dateTime': pytz.utc.localize(self.stop).isoformat()}
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
            'guestsCanModify': True,
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
        if self.privacy:
            values['visibility'] = self.privacy
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
        if self._context.get('all_events'):
            service = GoogleCalendarService(self.env['google.service'].with_user(self.recurrence_id.base_event_id.user_id))
            self.recurrence_id.with_user(self.recurrence_id.base_event_id.user_id)._sync_odoo2google(service)
        else:
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
from odoo import api, models, Command
from odoo.tools import email_normalize

from odoo.addons.google_calendar.utils.google_calendar import GoogleCalendarService


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
        # If only some events are updated, sync those events.
        values['need_sync'] = bool(dtstart)
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
        # event_tz is written on event in Google but on recurrence in Odoo
        vals['event_tz'] = gevent.start.get('timeZone')
        super()._write_from_google(gevent, vals)

        base_event_time_fields = ['start', 'stop', 'allday']
        new_event_values = self.env["calendar.event"]._odoo_values(gevent)
        # We update the attendee status for all events in the recurrence
        google_attendees = gevent.attendees or []
        emails = [a.get('email') for a in google_attendees]
        partners = self.env['mail.thread']._mail_find_partner_from_emails(emails, records=self, force_create=True, extra_domain=[('type', '!=', 'private')])
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

        for odoo_attendee_email in set(existing_attendees.mapped('email')):
            # Remove old attendees. Sometimes, several partners have the same email.
            if email_normalize(odoo_attendee_email) not in emails:
                attendees = existing_attendees.exists().filtered(lambda att: att.email == email_normalize(odoo_attendee_email))
                self.calendar_event_ids.write({'need_sync': False, 'partner_ids': [Command.unlink(att.partner_id.id) for att in attendees]})

        # Update the recurrence values
        old_event_values = self.base_event_id and self.base_event_id.read(base_event_time_fields)[0]
        if old_event_values and any(new_event_values[key] != old_event_values[key] for key in base_event_time_fields):
            # we need to recreate the recurrence, time_fields were modified.
            base_event_id = self.base_event_id
            # We archive the old events to recompute the recurrence. These events are already deleted on Google side.
            # We can't call _cancel because events without user_id would not be deleted
            (self.calendar_event_ids - base_event_id).google_id = False
            (self.calendar_event_ids - base_event_id).unlink()
            base_event_id.with_context(dont_notify=True).write(dict(new_event_values, google_id=False, need_sync=False))
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
                }, need_sync=False)
            )

        # We apply the rrule check after the time_field check because the google_id are generated according
        # to base_event start datetime.
        if self.rrule != current_rrule:
            detached_events = self._apply_recurrence()
            detached_events.google_id = False
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
from odoo.addons.google_calendar.utils.google_calendar import GoogleCalendarService, InvalidSyncToken
from odoo.addons.google_calendar.models.google_sync import google_calendar_token

_logger = logging.getLogger(__name__)

class GoogleCredentials(models.Model):
    """"Google Account of res_users"""

    _name = 'google.calendar.credentials'
    _description = 'Google Calendar Account Data'

    user_ids = fields.One2many('res.users', 'google_cal_account_id', required=True)
    calendar_rtoken = fields.Char('Refresh Token', copy=False, groups="base.group_system")
    calendar_token = fields.Char('User token', copy=False, groups="base.group_system")
    calendar_token_validity = fields.Datetime('Token Validity', copy=False, groups="base.group_system")
    calendar_sync_token = fields.Char('Next Sync Token', copy=False, groups="base.group_system")
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

from odoo import api, fields, models, registry, _
from odoo.tools import ormcache_context
from odoo.exceptions import UserError
from odoo.osv import expression

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
            self._event_ids_from_google_ids.clear_cache(self)
        synced_fields = self._get_google_synced_fields()
        if 'need_sync' not in vals and vals.keys() & synced_fields and not self.env.user.google_synchronization_stopped:
            vals['need_sync'] = True

        result = super().write(vals)
        for record in self.filtered('need_sync'):
            if record.google_id:
                record.with_user(record._get_event_user())._google_patch(google_service, record.google_id, record._google_values(), timeout=3)

        return result

    @api.model_create_multi
    def create(self, vals_list):
        if any(vals.get('google_id') for vals in vals_list):
            self._event_ids_from_google_ids.clear_cache(self)
        if self.env.user.google_synchronization_stopped:
            for vals in vals_list:
                vals.update({'need_sync': False})
        records = super().create(vals_list)

        google_service = GoogleCalendarService(self.env['google.service'])
        records_to_sync = records.filtered(lambda r: r.need_sync and r.active)
        for record in records_to_sync:
            record.with_user(record._get_event_user())._google_insert(google_service, record._google_values(), timeout=3)
        return records

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
        for record in cancelled_records.filtered(lambda e: e.google_id and e.need_sync):
            record.with_user(record._get_event_user())._google_delete(google_service, record.google_id)
        for record in new_records:
            record.with_user(record._get_event_user())._google_insert(google_service, record._google_values())
        for record in updated_records:
            record.with_user(record._get_event_user())._google_patch(google_service, record.google_id, record._google_values())

    def _cancel(self):
        self.google_id = False
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

        odoo_values = [
            dict(self._odoo_values(e, default_reminders), need_sync=False)
            for e in new
        ]
        new_odoo = self.with_context(dont_notify=True)._create_from_google(new, odoo_values)
        cancelled = existing.cancelled()
        cancelled_odoo = self.browse(cancelled.odoo_ids(self.env))

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

        cancelled_odoo._cancel()
        synced_records = new_odoo + cancelled_odoo
        for gevent in existing - cancelled:
            # Last updated wins.
            # This could be dangerous if google server time and odoo server time are different
            updated = parse(gevent.updated)
            odoo_record = self.browse(gevent.odoo_id(self.env))
            # Migration from 13.4 does not fill write_date. Therefore, we force the update from Google.
            if not odoo_record.write_date or updated >= pytz.utc.localize(odoo_record.write_date):
                vals = dict(self._odoo_values(gevent, default_reminders), need_sync=False)
                odoo_record.with_context(dont_notify=True)._write_from_google(gevent, vals)
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

            body = _(
                "The following event could not be synced with Google Calendar. </br>"
                "It will not be synced as long at it is not updated.</br>"
                "%(reason)s", reason=reason)

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
                self.exists().with_context(dont_notify=True).need_sync = False

    @after_commit
    def _google_insert(self, google_service: GoogleCalendarService, values, timeout=TIMEOUT):
        if not values:
            return
        with google_calendar_token(self.env.user.sudo()) as token:
            if token:
                try:
                    send_updates = self._context.get('send_updates', True)
                    google_service.google_service = google_service.google_service.with_context(send_updates=send_updates)
                    google_id = google_service.insert(values, token=token, timeout=timeout)
                    # Everything went smoothly
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

    def _write_from_google(self, gevent, vals):
        self.write(vals)

    @api.model
    def _create_from_google(self, gevents, vals_list):
        return self.create(vals_list)

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

_logger = logging.getLogger(__name__)

class User(models.Model):
    _inherit = 'res.users'

    google_cal_account_id = fields.Many2one('google.calendar.credentials')
    google_calendar_rtoken = fields.Char(related='google_cal_account_id.calendar_rtoken', groups="base.group_system")
    google_calendar_token = fields.Char(related='google_cal_account_id.calendar_token')
    google_calendar_token_validity = fields.Datetime(related='google_cal_account_id.calendar_token_validity')
    google_calendar_sync_token = fields.Char(related='google_cal_account_id.calendar_sync_token')
    google_calendar_cal_id = fields.Char(related='google_cal_account_id.calendar_cal_id')
    google_synchronization_stopped = fields.Boolean(related='google_cal_account_id.synchronization_stopped', readonly=False)

    _sql_constraints = [
        ('google_token_uniq', 'unique (google_cal_account_id)', "The user has already a google account"),
    ]


    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + ['google_synchronization_stopped', 'google_cal_account_id']

    @property
    def SELF_WRITEABLE_FIELDS(self):
        return super().SELF_WRITEABLE_FIELDS + ['google_synchronization_stopped', 'google_cal_account_id']

    def _get_google_calendar_token(self):
        self.ensure_one()
        if self.google_cal_account_id.calendar_rtoken and not self.google_cal_account_id._is_google_calendar_valid():
            self.sudo().google_cal_account_id._refresh_google_calendar_token()
        return self.google_cal_account_id.calendar_token

    def _sync_google_calendar(self, calendar_service: GoogleCalendarService):
        self.ensure_one()
        if self.google_synchronization_stopped:
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
                events, next_sync_token, default_reminders = calendar_service.get_events(self.google_cal_account_id.calendar_sync_token, token=token)
            except InvalidSyncToken:
                events, next_sync_token, default_reminders = calendar_service.get_events(token=token)
                full_sync = True
        self.google_cal_account_id.calendar_sync_token = next_sync_token

        # Google -> Odoo
        send_updates = not full_sync
        events.clear_type_ambiguity(self.env)
        recurrences = events.filter(lambda e: e.is_recurrence())
        synced_recurrences = self.env['calendar.recurrence']._sync_google2odoo(recurrences)
        synced_events = self.env['calendar.event']._sync_google2odoo(events - recurrences, default_reminders=default_reminders)

        # Odoo -> Google
        recurrences = self.env['calendar.recurrence']._get_records_to_sync(full_sync=full_sync)
        recurrences -= synced_recurrences
        recurrences.with_context(send_updates=send_updates)._sync_odoo2google(calendar_service)
        synced_events |= recurrences.calendar_event_ids - recurrences._get_outliers()
        synced_events |= synced_recurrences.calendar_event_ids - synced_recurrences._get_outliers()
        events = self.env['calendar.event']._get_records_to_sync(full_sync=full_sync)
        (events - synced_events).with_context(send_updates=send_updates)._sync_odoo2google(calendar_service)

        return bool(events | synced_events) or bool(recurrences | synced_recurrences)

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
                _logger.exception("[%s] Calendar Synchro - Exception : %s !", user, exception_to_unicode(e))
                self.env.cr.rollback()

    def stop_google_synchronization(self):
        self.ensure_one()
        self.google_synchronization_stopped = True

    def restart_google_synchronization(self):
        self.ensure_one()
        if not self.google_cal_account_id:
            self.google_cal_account_id = self.env['google.calendar.credentials'].sudo().create([{'user_ids': [Command.set(self.ids)]}])
        self.google_synchronization_stopped = False
        self.env['calendar.recurrence']._restart_google_sync()
        self.env['calendar.event']._restart_google_sync()

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

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
google_calendar_account_reset,google_calendar_account_reset_access_right,model_google_calendar_account_reset,base.group_system,1,1,1,0
google_calendar.access_google_calendar_credentials,access_google_calendar_credentials,google_calendar.model_google_calendar_credentials,base.group_user,1,0,0,0
google_calendar_manager,access_google_calendar_credentials_manager,google_calendar.model_google_calendar_credentials,base.group_system,1,1,0,0

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

## File: static\src\js\google_calendar.js

```javascript
odoo.define('google_calendar.CalendarView', function (require) {
"use strict";

var core = require('web.core');
var Dialog = require('web.Dialog');
var framework = require('web.framework');
const CalendarView = require('@calendar/js/calendar_view')[Symbol.for("default")];
const CalendarRenderer = require('@calendar/js/calendar_renderer')[Symbol.for("default")].AttendeeCalendarRenderer;
const CalendarController = require('@calendar/js/calendar_controller')[Symbol.for("default")];
const CalendarModel = require('@calendar/js/calendar_model')[Symbol.for("default")];
const viewRegistry = require('web.view_registry');
const session = require('web.session');

var _t = core._t;

const GoogleCalendarModel = CalendarModel.include({

    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this.google_is_sync = true;
        this.google_pending_sync = false;
    },

    /**
     * @override
     */
    __get: function () {
        var result = this._super.apply(this, arguments);
        result.google_is_sync = this.google_is_sync;
        return result;
    },


    /**
     * @override
     * @returns {Promise}
     */
    async _loadCalendar() {
        const _super = this._super.bind(this);
        // When the calendar synchronization takes some time, prevents retriggering the sync while navigating the calendar.
        if (this.google_pending_sync) {
            return _super(...arguments);
        }
        try {
            await Promise.race([
                new Promise(resolve => setTimeout(resolve, 1000)),
                this._syncGoogleCalendar(true)
            ]);
        } catch (error) {
            if (error.event) {
                error.event.preventDefault();
            }
            console.error("Could not synchronize Google events now.", error);
            this.google_pending_sync = false;
        }
        return _super(...arguments);
    },

    _syncGoogleCalendar(shadow = false) {
        var self = this;
        var context = this.getSession().user_context;
        this.google_pending_sync = true;
        return this._rpc({
            route: '/google_calendar/sync_data',
            params: {
                model: this.modelName,
                fromurl: window.location.href,
            }
        }, {shadow}).then(function (result) {
            if (["need_config_from_admin", "need_auth", "sync_stopped"].includes(result.status)) {
                self.google_is_sync = false;
            } else if (result.status === "no_new_event_from_google" || result.status === "need_refresh") {
                self.google_is_sync = true;
            }
            self.google_pending_sync = false;
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
})

const GoogleCalendarController = CalendarController.include({
    custom_events: _.extend({}, CalendarController.prototype.custom_events, {
        syncGoogleCalendar: '_onGoogleSyncCalendar',
        stopGoogleSynchronization: '_onStopGoogleSynchronization',
        archiveRecord: '_onArchiveRecord',
    }),


    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Try to sync the calendar with Google Calendar. According to the result
     * from Google API, this function may require an action of the user by the
     * mean of a dialog.
     *
     * @private
     * @returns {OdooEvent} event
     */
    _onGoogleSyncCalendar: function (event) {
        var self = this;

        return this._restartGoogleSynchronization().then(() => {return this.model._syncGoogleCalendar();}).then(function (o) {
            if (o.status === "need_auth") {
                Dialog.alert(self, _t("You will be redirected to Google to authorize access to your calendar!"), {
                    confirm_callback: function() {
                        framework.redirect(o.url);
                    },
                    title: _t('Redirection'),
                });
            } else if (o.status === "need_config_from_admin") {
                if (!_.isUndefined(o.action) && parseInt(o.action)) {
                    Dialog.confirm(self, _t("The Google Synchronization needs to be configured before you can use it, do you want to do it now?"), {
                        confirm_callback: function() {
                            self.do_action(o.action);
                        },
                        title: _t('Configuration'),
                    });
                } else {
                    Dialog.alert(self, _t("An administrator needs to configure Google Synchronization before you can use it!"), {
                        title: _t('Configuration'),
                    });
                }
            } else if (o.status === "need_refresh") {
                self.reload();
                return event.data.on_refresh();
            }
        }).then(event.data.on_always, event.data.on_always);
    },

    _onStopGoogleSynchronization: function (event) {
        var self = this;
        Dialog.confirm(this, _t("You are about to stop the synchronization of your calendar with Google. Are you sure you want to continue?"), {
            confirm_callback: function() {
                return self._rpc({
                    model: 'res.users',
                    method: 'stop_google_synchronization',
                    args: [[self.context.uid]],
                }).then(() => {
                    self.displayNotification({
                        title: _t("Success"),
                        message: _t("The synchronization with Google calendar was successfully stopped."),
                        type: 'success',
                    });
                }).then(event.data.on_confirm);
            },
            title: _t('Confirmation'),
        });

        return event.data.on_always();
    },

    _restartGoogleSynchronization: function () {
        return this._rpc({
            model: 'res.users',
            method: 'restart_google_synchronization',
            args: [[this.context.uid]],
        });
    },

    _onArchiveRecord: async function (event) {
        const self = this;
        if (event.data.event.record.recurrency) {
            const recurrenceUpdate = await this._askRecurrenceUpdatePolicy();
            event.data = Object.assign({}, event.data, {
                    'recurrenceUpdate': recurrenceUpdate,
                });
                if (recurrenceUpdate === 'self_only') {
                    self.model.archiveRecords([event.data.id], self.modelName).then(function () {
                    self.reload();
                });
                } else {
                    return this._rpc({
                        model: self.modelName,
                        method: 'action_mass_archive',
                        args: [[event.data.id], recurrenceUpdate],
                    }).then( function () {
                        self.reload();
                    });
                }
        } else {
            Dialog.confirm(this, _t("Are you sure you want to delete this record ?"), {
                confirm_callback: function () {
                    self.model.archiveRecords([event.data.id], self.modelName).then(function () {
                        self.reload();
                    });
                }
            });
        }
    },
});

const GoogleCalendarRenderer = CalendarRenderer.include({
    custom_events: _.extend({}, CalendarRenderer.prototype.custom_events, {
        archive_event: '_onArchiveEvent',
    }),

    events: _.extend({}, CalendarRenderer.prototype.events, {
        'click .o_google_sync_pending': '_onGoogleSyncCalendar',
        'click .o_google_sync_button_configured': '_onStopGoogleSynchronization',
    }),

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _initGooglePillButton: function() {
        // hide the pending button
        this.$calendarSyncContainer.find('#google_sync_pending').hide();
        const switchBadgeClass = elem => elem.toggleClass(['badge-primary', 'badge-danger']);
        this.$('#google_sync_configured').hover(() => {
            switchBadgeClass(this.$calendarSyncContainer.find('#google_sync_configured'));
            this.$calendarSyncContainer.find('#google_check').hide();
            this.$calendarSyncContainer.find('#google_stop').show();
        }, () => {
            switchBadgeClass(this.$calendarSyncContainer.find('#google_sync_configured'));
            this.$calendarSyncContainer.find('#google_stop').hide();
            this.$calendarSyncContainer.find('#google_check').show();
        });
    },

    _getGoogleButton: function () {
        this.$calendarSyncContainer.find('#google_sync_pending').show();
    },

    _getGoogleStopButton: function () {
        this.$calendarSyncContainer.find('#google_sync_configured').show();
    },

    /**
     * Adds the Sync with Google button in the sidebar
     *
     * @private
     */
    _initSidebar: function () {
        var self = this;
        this._super.apply(this, arguments);
        this.$googleButton = this.$('#google_sync_pending');
        this.$googleStopButton = this.$('#google_sync_configured');
        if (this.model === "calendar.event") {
            if (this.state.google_is_sync) {
                this._initGooglePillButton();
            } else {
                // Hide the button needed when the calendar sync is configured
                self.$googleStopButton.hide();
            }
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Requests to sync the calendar with Google Calendar
     *
     * @private
     */
    _onGoogleSyncCalendar: function () {
        var self = this;
        var context = this.getSession().user_context;
        this.$googleButton.prop('disabled', true);
        this.trigger_up('syncGoogleCalendar', {
            on_always: function () {
                self.$googleButton.prop('disabled', false);
            },
            on_refresh: function () {
                self._initGooglePillButton();
            }
        });
    },

    _onStopGoogleSynchronization: function() {
        var self = this;
        this.$googleStopButton.prop('disabled', true);
        this.trigger_up('stopGoogleSynchronization' , {
            on_confirm: function () {
                self.$googleStopButton.hide();
                self.$googleButton.show();
            },
            on_always: function() {
                self.$googleStopButton.prop('disabled', false);
            }
        });
    },

    _onArchiveEvent: function (event) {
        this._unselectEvent();
        this.trigger_up('archiveRecord', {id: parseInt(event.data.id, 10), event: event.target.event.extendedProps});
    },
});

return {
    GoogleCalendarController,
    GoogleCalendarModel,
    GoogleCalendarRenderer,
};

});

```

## File: static\src\js\google_calendar_popover.js

```javascript
odoo.define('google_calendar.GoogleCalendarPopover', function(require) {
    "use strict";

    const CalendarPopover = require('@calendar/js/calendar_renderer')[Symbol.for("default")].AttendeeCalendarPopover;

    const GoogleCalendarPopover = CalendarPopover.include({
        events: _.extend({}, CalendarPopover.prototype.events, {
            'click .o_cw_popover_archive_g': '_onClickPopoverGArchive',
        }),

        isGEventSyncedAndArchivable() {
            return this.isCurrentPartnerOrganizer() && this.event.extendedProps.record.google_id;
        },

        isEventDeletable() {
            return !this.isGEventSyncedAndArchivable() && this._super();
        },

        _onClickPopoverGArchive: function (ev) {
            ev.preventDefault();
            this.trigger_up('archive_event', {id: this.event.id});
        },
    });

    return GoogleCalendarPopover;
});

```

## File: static\src\xml\base_calendar.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-inherit="calendar.CalendarView" t-inherit-mode="extension">
        <xpath expr="//div[@id='calendar_sync']" position="inside">
            <button type="button" id="google_sync_pending" class="o_google_sync_button o_google_sync_pending btn btn-secondary btn">
                <b><i class='fa fa-refresh'/> Google</b>
            </button>
            <button type="button" id="google_sync_configured" class="mr-1 o_google_sync_button o_google_sync_button_configured btn badge-primary">
                <b>
                    <i id="google_check" class='fa fa-check'/>
                    <i id="google_stop" class='fa fa-times' style="display: none"/>&amp;nbsp;Google
                </b>
            </button>
        </xpath>
    </t>
</templates>

```

## File: static\src\xml\google_calendar_popover.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-extend="Calendar.attendee.status.popover">
         <t t-jquery=".o_cw_popover_edit" t-operation="after">
            <a t-if="widget.isGEventSyncedAndArchivable() and widget.isEventDetailsVisible()" href="#" class="btn btn-secondary o_cw_popover_archive_g">Delete</a>
         </t>
    </t>
</templates>

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
from odoo.tools import exception_to_unicode
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
    def get_events(self, sync_token=None, token=None, timeout=TIMEOUT):
        url = "/calendar/v3/calendars/primary/events"
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
    def insert(self, values, token=None, timeout=TIMEOUT):
        send_updates = self.google_service._context.get('send_updates', True)
        url = "/calendar/v3/calendars/primary/events?sendUpdates=%s" % ("all" if send_updates else "none")
        headers = {'Content-type': 'application/json', 'Authorization': 'Bearer %s' % token}
        if not values.get('id'):
            values['id'] = uuid4().hex
        self.google_service._do_request(url, json.dumps(values), headers, method='POST', timeout=timeout)
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
        return self.google_service._get_authorize_uri(from_url, service='calendar', scope=self._get_calendar_scope())

    def _can_authorize_google(self, user):
        return user.has_group('base.group_erp_manager')

```

## File: utils\google_event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

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
        return self._events[event_id].get(name)

    def __repr__(self):
        return '%s%s' % (self.__class__.__name__, self.ids)

    @property
    def ids(self):
        return tuple(e.id for e in self)

    @property
    def rrule(self):
        if self.recurrence and any('RRULE' in item for item in self.recurrence):
            rrule = next(item for item in self.recurrence if 'RRULE' in item)
            return rrule[6:]  # skip "RRULE:" in the rrule string

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
        # UserA creates an event in Odoo (he is the owner) but userB syncs first.
        # There is no way to insert the event into userA's calendar since we don't have
        # any authentication access. The event is therefore inserted into userB's calendar
        # (he is the organizer in Google). The "real" owner (in Odoo) is stored as an
        # extended property. There is currently no support to "transfert" ownership when
        # userA syncs his calendar the first time.
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
        ID_RANGE = re.search(r'\w+_R\d+T\d+', self.recurringEventId).group()
        TIMESTAMP = re.search(r'\d+T\d+Z', self.id).group()
        return f"{ID_RANGE}_{TIMESTAMP}"

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
        video_meeting = list(filter(lambda entryPoints: entryPoints['entryPointType'] == 'video', self.conferenceData['entryPoints']))
        return video_meeting[0]['uri'] if video_meeting else False

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
            <field name="attendee_status" position="after">
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
            <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <div id="msg_module_google_calendar" position="replace">
                    <div class="content-group" attrs="{'invisible': [('module_google_calendar','=',False)]}">
                        <div class="mt16 row">
                            <label for="cal_client_id" string="Client ID" class="col-3 col-lg-3 o_light_label"/>
                            <field name="cal_client_id" nolabel="1"/>
                            <label for="cal_client_secret" string="Client Secret" class="col-3 col-lg-3 o_light_label"/>
                            <field name="cal_client_secret" password="True" nolabel="1"/>
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
                            <field name="google_calendar_rtoken" readonly="1"/>
                            <field name="google_calendar_token" readonly="1"/>
                            <field name="google_calendar_token_validity" readonly="1"/>
                            <field name="google_calendar_sync_token" readonly="1"/>
                            <field name="google_calendar_cal_id" readonly="1"/>
                            <button
                                string="Reset Account" class="btn btn-secondary mt-3"
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
        if self.delete_policy in ('delete_google', 'delete_both'):
            with google_calendar_token(self.user_id) as token:
                for event in events:
                    google.delete(event.google_id, token=token)

        if self.delete_policy in ('delete_odoo', 'delete_both'):
            events.google_id = False
            events.unlink()

        if self.sync_policy == 'all':
            events.write({
                'google_id': False,
                'need_sync': True,
            })

        self.user_id.google_cal_account_id._set_auth_tokens(False, False, 0)
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
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z" />
                </footer>
            </form>
        </field>
    </record>

    <record id="google_calendar_reset_account_action" model="ir.actions.act_window">
        <field name="name"></field>
        <field name="type">ir.actions.act_window</field>
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

