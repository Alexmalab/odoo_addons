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
    'depends': ['microsoft_account', 'calendar'],
    'data': [
        'data/microsoft_calendar_data.xml',
        'security/ir.model.access.csv',
        'wizard/reset_account_views.xml',
        'views/res_config_settings_views.xml',
        'views/res_users_views.xml',
        'views/microsoft_calendar_views.xml',
        ],
    'installable': True,
    'post_init_hook': 'init_initiating_microsoft_uuid',
    'assets': {
        'web.assets_backend': [
            'microsoft_calendar/static/src/scss/microsoft_calendar.scss',
            'microsoft_calendar/static/src/views/**/*',
        ],
        'web.qunit_suite_tests': [
            'microsoft_calendar/static/tests/microsoft_calendar_mock_server.js',
            'microsoft_calendar/static/tests/microsoft_calendar_tests.js',
        ],
        'web.qunit_mobile_suite_tests': [
            'microsoft_calendar/static/tests/microsoft_calendar_mock_server.js',
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
            need_refresh = request.env.user.sudo().with_context(dont_notify=True)._sync_microsoft_calendar()

            # If synchronization has been stopped
            if not need_refresh and request.env.user.microsoft_synchronization_stopped:
                return {
                    "status": "sync_stopped",
                    "url": ''
                }
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

## File: data\neutralize.sql

```sql
-- neutralization of Microsoft calendar
UPDATE res_users
    SET microsoft_calendar_token = NULL,
        microsoft_calendar_rtoken = NULL,
        microsoft_synchronization_stopped = True;

```

## File: models\calendar.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pytz
import re
from datetime import datetime
from dateutil.parser import parse
from dateutil.relativedelta import relativedelta
from collections import defaultdict

from odoo import api, fields, models, _
from odoo.osv import expression
from odoo.exceptions import UserError, ValidationError
from odoo.tools import is_html_empty, email_normalize
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
VIDEOCALL_URL_PATTERNS = (
    r'https://teams.microsoft.com',
)
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

    @api.model
    def _restart_microsoft_sync(self):
        self.env['calendar.event'].with_context(dont_notify=True).search(self._get_microsoft_sync_domain()).write({
            'need_sync_m': True,
        })

    def _check_microsoft_sync_status(self):
        """
        Returns True if synchronization with Outlook Calendar is active and False otherwise.
        The 'microsoft_synchronization_stopped' variable needs to be 'False' and Outlook account must be connected.
        """
        outlook_connected = self.env.user._get_microsoft_calendar_token()
        return outlook_connected and self.env.user.microsoft_synchronization_stopped is False

    @api.model_create_multi
    def create(self, vals_list):
        notify_context = self.env.context.get('dont_notify', False)

        # Forbid recurrence creation in Odoo, suggest its creation in Outlook due to the spam limitation.
        recurrency_in_batch = any(vals.get('recurrency') for vals in vals_list)
        if self._check_microsoft_sync_status() and not notify_context and recurrency_in_batch:
            self._forbid_recurrence_creation()

        for vals in vals_list:
            # If event has a different organizer, check its sync status and verify if the user is listed as attendee.
            sender_user, partner_ids = self._get_organizer_user_change_info(vals)
            partner_included = partner_ids and len(partner_ids) > 0 and sender_user.partner_id.id in partner_ids
            self._check_organizer_validation(sender_user, partner_included)

        # for a recurrent event, we do not create events separately but we directly
        # create the recurrency from the corresponding calendar.recurrence.
        # That's why, events from a recurrency have their `need_sync_m` attribute set to False.
        return super(Meeting, self.with_context(dont_notify=notify_context)).create([
            dict(vals, need_sync_m=False) if vals.get('recurrence_id') or vals.get('recurrency') else vals
            for vals in vals_list
        ])

    def _check_organizer_validation(self, sender_user, partner_included):
        """ Check if the proposed event organizer can be set accordingly. """
        # Edge case: events created or updated from Microsoft should not check organizer validation.
        change_from_microsoft = self.env.context.get('dont_notify', False)
        if sender_user and sender_user != self.env.user and not change_from_microsoft:
            current_sync_status = self._check_microsoft_sync_status()
            sender_sync_status = self.with_user(sender_user)._check_microsoft_sync_status()
            if not sender_sync_status and current_sync_status:
                raise ValidationError(
                    _("For having a different organizer in your event, it is necessary that "
                      "the organizer have its Odoo Calendar synced with Outlook Calendar."))
            elif sender_sync_status and not partner_included:
                raise ValidationError(
                    _("It is necessary adding the proposed organizer as attendee before saving the event."))

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

    def _forbid_recurrence_update(self):
        """
        Suggest user to update recurrences in Outlook due to the Outlook Calendar spam limitation.
        """
        error_msg = _("Due to an Outlook Calendar limitation, recurrence updates must be done directly in Outlook Calendar.")
        if any(not record.microsoft_id for record in self):
            # If any event is not synced, suggest deleting it in Odoo and recreating it in Outlook.
            error_msg = _(
                "Due to an Outlook Calendar limitation, recurrence updates must be done directly in Outlook Calendar.\n"
                "If this recurrence is not shown in Outlook Calendar, you must delete it in Odoo Calendar and recreate it in Outlook Calendar.")

        raise UserError(error_msg)

    def _forbid_recurrence_creation(self):
        """
        Suggest user to update recurrences in Outlook due to the Outlook Calendar spam limitation.
        """
        raise UserError(_("Due to an Outlook Calendar limitation, recurrent events must be created directly in Outlook Calendar."))

    def write(self, values):
        recurrence_update_setting = values.get('recurrence_update')
        notify_context = self.env.context.get('dont_notify', False)

        # Forbid recurrence updates through Odoo and suggest user to update it in Outlook.
        if self._check_microsoft_sync_status():
            recurrency_in_batch = self.filtered(lambda ev: ev.recurrency)
            recurrence_update_attempt = recurrence_update_setting or 'recurrency' in values or recurrency_in_batch and len(recurrency_in_batch) > 0
            if not notify_context and recurrence_update_attempt and not 'active' in values:
                self._forbid_recurrence_update()

        # When changing the organizer, check its sync status and verify if the user is listed as attendee.
        # Updates from Microsoft must skip this check since changing the organizer on their side is not possible.
        change_from_microsoft = self.env.context.get('dont_notify', False)
        deactivated_events_ids = []
        for event in self:
            if values.get('user_id') and event.user_id.id != values['user_id'] and not change_from_microsoft:
                sender_user, partner_ids = event._get_organizer_user_change_info(values)
                partner_included = sender_user.partner_id in event.attendee_ids.partner_id or sender_user.partner_id.id in partner_ids
                event._check_organizer_validation(sender_user, partner_included)
                event._recreate_event_different_organizer(values, sender_user)
                deactivated_events_ids.append(event.id)

        # check a Outlook limitation in overlapping the actual recurrence
        if recurrence_update_setting == 'self_only' and 'start' in values:
            self._check_recurrence_overlapping(values['start'])

        # if a single event becomes the base event of a recurrency, it should be first
        # removed from the Outlook calendar.
        if 'recurrency' in values and values['recurrency']:
            for e in self.filtered(lambda e: not e.recurrency and not e.recurrence_id):
                e._microsoft_delete(e._get_organizer(), e.ms_organizer_event_id, timeout=3)
                e.microsoft_id = False

        deactivated_events = self.browse(deactivated_events_ids)
        # Update attendee status before 'values' variable is overridden in super.
        attendee_ids = values.get('attendee_ids')
        if attendee_ids and values.get('partner_ids'):
            (self - deactivated_events)._update_attendee_status(attendee_ids)

        res = super(Meeting, (self - deactivated_events).with_context(dont_notify=notify_context)).write(values)

        # Deactivate events that were recreated after changing organizer.
        if deactivated_events:
            res |= super(Meeting, deactivated_events.with_context(dont_notify=notify_context)).write({**values, 'active': False})

        if recurrence_update_setting in ('all_events',) and len(self) == 1 \
           and values.keys() & self._get_microsoft_synced_fields():
            self.recurrence_id.need_sync_m = True
        return res

    def unlink(self):
        # Forbid recurrent events unlinking from calendar list view with sync active.
        if self and self._check_microsoft_sync_status():
            synced_events = self._get_synced_events()
            change_from_microsoft = self.env.context.get('dont_notify', False)
            recurrence_deletion = any(ev.recurrency and ev.recurrence_id and ev.follow_recurrence for ev in synced_events)
            if not change_from_microsoft and recurrence_deletion:
                self._forbid_recurrence_update()
        return super().unlink()

    def _recreate_event_different_organizer(self, values, sender_user):
        """ Copy current event values, delete it and recreate it with the new organizer user. """
        self.ensure_one()
        event_copy = {**self.copy_data()[0], 'microsoft_id': False}
        self.env['calendar.event'].with_user(sender_user).create({**event_copy, **values})
        if self.ms_universal_event_id:
            self._microsoft_delete(self._get_organizer(), self.ms_organizer_event_id)

    @api.model
    def _get_organizer_user_change_info(self, values):
        """ Return the sender user of the event and the partner ids listed on the event values. """
        sender_user_id = values.get('user_id', self.env.user.id)
        sender_user = self.env['res.users'].browse(sender_user_id)
        attendee_values = self._attendees_values(values['partner_ids']) if 'partner_ids' in values else []
        partner_ids = []
        if attendee_values:
            for command in attendee_values:
                if len(command) == 3 and isinstance(command[2], dict):
                    partner_ids.append(command[2].get('partner_id'))
        return sender_user, partner_ids

    def _update_attendee_status(self, attendee_ids):
        """ Merge current status from 'attendees_ids' with new attendees values for avoiding their info loss in write().
        Create a dict getting the state of each attendee received from 'attendee_ids' variable and then update their state.
        :param attendee_ids: List of attendee commands carrying a dict with 'partner_id' and 'state' keys in its third position.
        """
        state_by_partner = {}
        for cmd in attendee_ids:
            if len(cmd) == 3 and isinstance(cmd[2], dict) and all(key in cmd[2] for key in ['partner_id', 'state']):
                state_by_partner[cmd[2]['partner_id']] = cmd[2]['state']
        for attendee in self.attendee_ids:
            state_update = state_by_partner.get(attendee.partner_id.id)
            if state_update:
                attendee.state = state_update

    def action_mass_archive(self, recurrence_update_setting):
        # Do not allow archiving if recurrence is synced with Outlook. Suggest updating directly from Outlook.
        self.ensure_one()
        if self._check_microsoft_sync_status() and self.microsoft_id:
            self._forbid_recurrence_update()
        super().action_mass_archive(recurrence_update_setting)

    def _get_microsoft_sync_domain(self):
        # in case of full sync, limit to a range of 1y in past and 1y in the future by default
        ICP = self.env['ir.config_parameter'].sudo()
        day_range = int(ICP.get_param('microsoft_calendar.sync.range_days', default=365))
        lower_bound = fields.Datetime.subtract(fields.Datetime.now(), days=day_range)
        upper_bound = fields.Datetime.add(fields.Datetime.now(), days=day_range)

        # Define 'custom_lower_bound_range' param for limiting old events updates in Odoo and avoid spam on Microsoft.
        custom_lower_bound_range = ICP.get_param('microsoft_calendar.sync.lower_bound_range')
        if custom_lower_bound_range:
            lower_bound = fields.Datetime.subtract(fields.Datetime.now(), days=int(custom_lower_bound_range))
        domain = [
            ('partner_ids.user_ids', 'in', self.env.user.id),
            ('stop', '>', lower_bound),
            ('start', '<', upper_bound),
            '!', '&', '&', ('recurrency', '=', True), ('recurrence_id', '!=', False), ('follow_recurrence', '=', True)
        ]

        # Synchronize events that were created after the first synchronization date, when applicable.
        first_synchronization_date = ICP.get_param('microsoft_calendar.sync.first_synchronization_date')
        if first_synchronization_date:
            domain = expression.AND([domain, [('create_date', '>=', first_synchronization_date)]])

        return self._extend_microsoft_domain(domain)


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

        # if a videocall URL is provided with the Outlook event, use it
        if microsoft_event.isOnlineMeeting and microsoft_event.onlineMeeting.get("joinUrl"):
            values['videocall_location'] = microsoft_event.onlineMeeting["joinUrl"]
        else:
            # if a location is a URL matching a specific pattern (i.e a URL to access to a videocall),
            # copy it in the 'videocall_location' instead
            if values['location'] and any(re.match(p, values['location']) for p in VIDEOCALL_URL_PATTERNS):
                values['videocall_location'] = values['location']
                values['location'] = False

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
            # Responses from external invitations are stored in the 'responseStatus' field.
            # This field only carries the current user's event status because Microsoft hides other user's status.
            if self.env.user.email == email and microsoft_event.responseStatus:
                attendee_microsoft_status = microsoft_event.responseStatus.get('response', 'none')
            else:
                attendee_microsoft_status = attendee_info.get('status').get('response')
            state = ATTENDEE_CONVERTER_M2O.get(attendee_microsoft_status, 'needsAction')

            if email in attendees_by_emails:
                # Update existing attendees
                commands_attendee += [(1, attendees_by_emails[email].id, {'state': state})]
            elif partner:
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
                'content': self.description if not is_html_empty(self.description) else '',
                'contentType': "html",
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
                        'monday': recurrence.mon,
                        'tuesday': recurrence.tue,
                        'wednesday': recurrence.wed,
                        'thursday': recurrence.thu,
                        'friday': recurrence.fri,
                        'saturday': recurrence.sat,
                        'sunday': recurrence.sun,
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
        records = self.filtered(lambda e: not e.user_id or e.user_id == user or user.partner_id in e.partner_ids)
        super(Meeting, records)._cancel_microsoft()
        attendees = (self - records).attendee_ids.filtered(lambda a: a.partner_id == user.partner_id)
        attendees.do_decline()

    def _get_event_user_m(self, user_id=None):
        """ Get the user who will send the request to Microsoft (organizer if synchronized and current user otherwise). """
        self.ensure_one()
        # Current user must have access to token in order to access event properties (non-public user).
        current_user_status = self.env.user._get_microsoft_calendar_token()
        if user_id != self.env.user and current_user_status:
            if user_id is None:
                user_id = self.user_id
            if user_id and self.with_user(user_id).sudo()._check_microsoft_sync_status():
                return user_id
        return self.env.user

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

    def _send_mail_to_attendees(self, mail_template, force_send=False):
        """ Override the super method
        If not synced with Microsoft Outlook, let Odoo in charge of sending emails
        Otherwise, Microsoft Outlook will send them
        """
        with microsoft_calendar_token(self.env.user.sudo()) as token:
            if not token:
                super()._send_mail_to_attendees(mail_template, force_send)

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
        for event in linked_events:
            if event._check_microsoft_sync_status() and self.env.user != event.user_id and self.env.user.partner_id in event.partner_ids:
                if event.recurrency:
                    event._forbid_recurrence_update()
                event._microsoft_attendee_answer(answer, params)

```

## File: models\calendar_recurrence_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.osv import expression


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
            recurrence.with_context(dont_notify=True).write(dict(values, need_sync_m=False))

    def _apply_recurrence(self, specific_values_creation=None, no_send_edit=False, generic_values_creation=None):
        events = self.filtered('need_sync_m').calendar_event_ids
        detached_events = super()._apply_recurrence(specific_values_creation, no_send_edit, generic_values_creation)

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

    @api.model
    def _restart_microsoft_sync(self):
        self.env['calendar.recurrence'].search(self._get_microsoft_sync_domain()).write({
            'need_sync_m': True,
        })

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
        # Edge case:  if the base event was deleted manually in 'self_only' update, skip applying recurrence.
        if self._has_base_event_time_fields_changed(new_event_values) and (new_event_values['start'] >= self.base_event_id.start):
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
            self.with_context(dont_notify=True)._write_events(dict({
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
        # Do not sync Odoo recurrences with Outlook Calendar anymore.
        domain = expression.FALSE_DOMAIN
        return self._extend_microsoft_domain(domain)

    def _cancel_microsoft(self):
        self.calendar_event_ids.with_context(dont_notify=True)._cancel_microsoft()
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

    def _split_from(self, event, recurrence_values=None):
        """
        When a recurrence is splitted, the base event of the new recurrence already
        exist and may be already synced with Outlook.
        In this case, we need to be removed this event on Outlook side to avoid duplicates while posting
        the new recurrence.
        """
        new_recurrence = super()._split_from(event, recurrence_values)
        if new_recurrence and new_recurrence.base_event_id.microsoft_id:
            new_recurrence.base_event_id._microsoft_delete(
                new_recurrence.base_event_id._get_organizer(),
                new_recurrence.base_event_id.ms_organizer_event_id
            )

        return new_recurrence

    def _get_event_user_m(self, user_id=None):
        """ Get the user who will send the request to Microsoft (organizer if synchronized and current user otherwise). """
        self.ensure_one()
        event = self._get_first_event()
        if event:
            return event._get_event_user_m(user_id)
        return self.env.user

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
from datetime import timedelta

from odoo import api, fields, models, registry
from odoo.tools import ormcache_context
from odoo.exceptions import UserError
from odoo.osv import expression
from odoo.sql_db import BaseCursor

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
        if fields_to_sync and 'need_sync_m' not in vals and not self.env.user.microsoft_synchronization_stopped:
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
        if self.env.user.microsoft_synchronization_stopped:
            for vals in vals_list:
                vals.update({'need_sync_m': False})
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
        self.with_context(dont_notify=True).write(vals)

    @api.model
    def _create_from_microsoft(self, microsoft_event, vals_list):
        return self.with_context(dont_notify=True).create(vals_list)

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
            new_recurrence_odoo = self.env['calendar.recurrence'].with_context(dont_notify=True).create(new_calendar_recurrence)
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
                odoo_event.with_context(dont_notify=True).write(dict(event_values, need_sync_m=False))
                update_events |= odoo_event

        # update the recurrence
        detached_events = self.with_context(dont_notify=True)._apply_recurrence(rec_values)
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

        # Get sync lower bound days range for checking if old events must be updated in Odoo.
        ICP = self.env['ir.config_parameter'].sudo()
        lower_bound_day_range = ICP.get_param('microsoft_calendar.sync.lower_bound_range')

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

                # If the update comes from an old event/recurrence, check if time diff between updates is reasonable.
                old_event_update_condition = True
                if lower_bound_day_range:
                    update_time_diff = ms_event_updated_time - odoo_event_updated_time
                    old_event_update_condition = odoo_event._check_old_event_update_required(int(lower_bound_day_range), update_time_diff)

                if ms_event_updated_time >= odoo_event_updated_time and old_event_update_condition:
                    vals = dict(odoo_event._microsoft_to_odoo_values(mevent), need_sync_m=False)
                    odoo_event.with_context(dont_notify=True)._write_from_microsoft(mevent, vals)

                    if odoo_event._name == 'calendar.recurrence':
                        update_events = odoo_event._update_microsoft_recurrence(mevent, microsoft_events)
                        synced_recurrences |= odoo_event
                        synced_events |= update_events
                    else:
                        synced_events |= odoo_event

        return synced_events, synced_recurrences

    def _check_old_event_update_required(self, lower_bound_day_range, update_time_diff):
        """
        Checks if an old event in Odoo should be updated locally. This verification is necessary because
        sometimes events in Odoo have the same state in Microsoft and even so they trigger updates locally
        due to a second or less of update time difference, thus spamming unwanted emails on Microsoft side.
        """
        # Event can be updated locally if its stop date is bigger than lower bound and the update time difference is reasonable (1 hour).
        # For recurrences, if any of the occurrences surpass the lower bound range, we update the recurrence.
        lower_bound = fields.Datetime.subtract(fields.Datetime.now(), days=lower_bound_day_range)
        stop_date_condition = True
        if self._name == 'calendar.event':
            stop_date_condition = self.stop >= lower_bound
        elif self._name == 'calendar.recurrence':
            stop_date_condition = any(event.stop >= lower_bound for event in self.calendar_event_ids)
        return stop_date_condition or update_time_diff >= timedelta(hours=1)

    def _impersonate_user(self, user_id):
        """ Impersonate a user (mainly the event organizer) to be able to call the Outlook API with its token """
        # This method is obsolete, as it has been replaced by the `_get_event_user_m` method, which gets the user who will make the request.
        return user_id.with_user(user_id)

    @after_commit
    def _microsoft_delete(self, user_id, event_id, timeout=TIMEOUT):
        """
        Once the event has been really removed from the Odoo database, remove it from the Outlook calendar.

        Note that all self attributes to use in this method must be provided as method parameters because
        'self' won't exist when this method will be really called due to @after_commit decorator.
        """
        microsoft_service = self._get_microsoft_service()
        sender_user = self._get_event_user_m(user_id)
        with microsoft_calendar_token(sender_user.sudo()) as token:
            if token and not sender_user.microsoft_synchronization_stopped:
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
        sender_user = self._get_event_user_m(user_id)
        with microsoft_calendar_token(sender_user.sudo()) as token:
            if token:
                self._ensure_attendees_have_email()
                res = microsoft_service.patch(event_id, values, token=token, timeout=timeout)
                self.with_context(dont_notify=True).write({
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
        sender_user = self._get_event_user_m()
        with microsoft_calendar_token(sender_user.sudo()) as token:
            if token:
                self._ensure_attendees_have_email()
                event_id, uid = microsoft_service.insert(values, token=token, timeout=timeout)
                self.with_context(dont_notify=True).write({
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
                # Fetch the event's id (ms_organizer_event_id) using its iCalUId (ms_universal_event_id) since the
                # former differs for each attendee. This info is required for sending the event answer and Odoo currently
                # saves the event's id of the last user who synced the event (who might be or not the current user).
                status, event = microsoft_service._get_single_event(self.ms_universal_event_id, token=token)
                if status and event and event.get('value') and len(event.get('value')) == 1:
                    # Send the attendee answer with its own ms_organizer_event_id.
                    res = microsoft_service.answer(
                        event.get('value')[0].get('id'),
                        answer, params, token=token, timeout=timeout
                    )
                    self.need_sync_m = not res

    def _get_microsoft_records_to_sync(self, full_sync=False):
        """
        Return records that should be synced from Odoo to Microsoft
        :param full_sync: If True, all events attended by the user are returned
        :return: events
        """
        domain = self.with_context(full_sync_m=full_sync)._get_microsoft_sync_domain()
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

    @api.model
    def _restart_microsoft_sync(self):
        """ Turns on the microsoft synchronization for all the events of
        a given user.
        """
        raise NotImplementedError()

    def _extend_microsoft_domain(self, domain):
        """ Extends the sync domain based on the full_sync_m context parameter.
        In case of full sync it shouldn't include already synced events.
        """
        if self._context.get('full_sync_m', True):
            domain = expression.AND([domain, [('ms_universal_event_id', '=', False)]])
        else:
            is_active_clause = (self._active_name, '=', True) if self._active_name else expression.TRUE_LEAF
            domain = expression.AND([domain, [
                '|',
                '&', ('ms_universal_event_id', '=', False), is_active_clause,
                ('need_sync_m', '=', True),
            ]])
        return domain

    def _get_event_user_m(self, user_id=None):
        """ Return the correct user to send the request to Microsoft.
        It's possible that a user creates an event and sets another user as the organizer. Using self.env.user will
        cause some issues, and it might not be possible to use this user for sending the request, so this method gets
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
from odoo.addons.microsoft_account.models.microsoft_service import DEFAULT_MICROSOFT_TOKEN_ENDPOINT
from odoo.addons.microsoft_calendar.utils.microsoft_calendar import InvalidSyncToken

_logger = logging.getLogger(__name__)


class User(models.Model):
    _inherit = 'res.users'

    microsoft_calendar_sync_token = fields.Char('Microsoft Next Sync Token', copy=False)
    microsoft_synchronization_stopped = fields.Boolean('Outlook Synchronization stopped', copy=False)

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + ['microsoft_synchronization_stopped']

    @property
    def SELF_WRITEABLE_FIELDS(self):
        return super().SELF_WRITEABLE_FIELDS + ['microsoft_synchronization_stopped']

    def _microsoft_calendar_authenticated(self):
        return bool(self.sudo().microsoft_calendar_rtoken)

    def _get_microsoft_calendar_token(self):
        if not self:
            return None

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
                DEFAULT_MICROSOFT_TOKEN_ENDPOINT, params=data, headers=headers, method='POST', preuri=''
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
        if self.microsoft_synchronization_stopped:
            return False

        # Set the first synchronization date as an ICP parameter before writing the variable
        # 'microsoft_calendar_sync_token' below, so we identify the first synchronization.
        self._set_ICP_first_synchronization_date(fields.Datetime.now())

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
        users = self.env['res.users'].search([('microsoft_calendar_rtoken', '!=', False), ('microsoft_synchronization_stopped', '=', False)])
        for user in users:
            _logger.info("Calendar Synchro - Starting synchronization for %s", user)
            try:
                user.with_user(user).sudo()._sync_microsoft_calendar()
                self.env.cr.commit()
            except Exception as e:
                _logger.exception("[%s] Calendar Synchro - Exception : %s !", user, exception_to_unicode(e))
                self.env.cr.rollback()

    def stop_microsoft_synchronization(self):
        self.ensure_one()
        self.microsoft_synchronization_stopped = True

    def restart_microsoft_synchronization(self):
        self.ensure_one()
        self.microsoft_synchronization_stopped = False
        self.env['calendar.recurrence']._restart_microsoft_sync()
        self.env['calendar.event']._restart_microsoft_sync()

    def _set_ICP_first_synchronization_date(self, now):
        """
        Set the first synchronization date as an ICP parameter when applicable (param not defined yet
        and calendar never synchronized before). This parameter is used for not synchronizing previously
        created Odoo events and thus avoid spamming invitations for those events.
        """
        ICP = self.env['ir.config_parameter'].sudo()
        first_synchronization_date = ICP.get_param('microsoft_calendar.sync.first_synchronization_date')

        if not first_synchronization_date:
            # Check if any calendar has synchronized before by checking the user's tokens.
            any_calendar_synchronized = self.env['res.users'].sudo().search_count(
                domain=[('microsoft_calendar_sync_token', '!=', False)],
                limit=1
            )

            # Check if any user synchronized its calendar before by saving the date token.
            # Add one minute of time diff for avoiding write time delay conflicts with the next sync methods.
            if not any_calendar_synchronized:
                ICP.set_param('microsoft_calendar.sync.first_synchronization_date', now - timedelta(minutes=1))

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
<svg id="Layer_1" data-name="Layer 1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 70 70">
  <defs>
    <mask id="mask" x="0" y="0" width="70" height="70" maskUnits="userSpaceOnUse">
      <g id="mask-2" data-name="mask">
        <g id="b">
          <path id="a" d="M4,0H65c4,0,5,1,5,5V65c0,4-1,5-5,5H4c-3,0-4-1-4-5V5C0,1,1,0,4,0Z" fill="#fff" fill-rule="evenodd"/>
        </g>
      </g>
    </mask>
    <linearGradient id="linear-gradient" x1="-998.51" y1="-216.96" x2="-999.51" y2="-217.96" gradientTransform="matrix(70, 0, 0, -70, 69966, -15187.42)" gradientUnits="userSpaceOnUse">
      <stop offset="0" stop-color="#94b6c8"/>
      <stop offset="1" stop-color="#6a9eba"/>
    </linearGradient>
  </defs>
  <g mask="url(#mask)">
    <g>
      <path d="M0,0H70V70H0Z" fill-rule="evenodd" fill="url(#linear-gradient)"/>
      <path d="M4,1H65c2.67,0,4.33.67,5,2V0H0V3C.67,1.67,2,1,4,1Z" fill="#fff" fill-opacity="0.38" fill-rule="evenodd"/>
      <path d="M45.3,69H4c-2,0-4-.14-4-4V33.43l13-13,21.72.11,2.69-2.68L57.05,23l-.28,29L54.33,54.5l.5,1.33,1.34-1.31.88-.16-.37,3.25Z" fill="#393939" fill-rule="evenodd" opacity="0.32" style="isolation: isolate"/>
      <path d="M4,69H65c2.67,0,4.33-1,5-3v4H0V66A3.92,3.92,0,0,0,4,69Z" fill-opacity="0.38" fill-rule="evenodd"/>
      <g>
        <g opacity="0.4">
          <g>
            <g>
              <ellipse cx="21.42" cy="36.92" rx="2.88" ry="4.39"/>
              <path d="M10.74,22.56V51.79a.57.57,0,0,0,.46.56l21.13,3.9a.56.56,0,0,0,.67-.56V18.3a.57.57,0,0,0-.68-.56L11.19,22A.58.58,0,0,0,10.74,22.56ZM21.42,44.28c-3.23,0-5.85-3.29-5.85-7.36s2.62-7.35,5.85-7.35,5.85,3.29,5.85,7.35S24.65,44.28,21.42,44.28Z"/>
            </g>
            <g>
              <polygon points="36.56 46.29 35.73 46.29 34.89 46.29 34.89 50.77 35.73 50.77 36.56 50.77 37.39 50.77 37.39 46.29 36.56 46.29"/>
              <path d="M54,19.65H34.92V25.7H53.48V52.56H34.92v1.78H54a1.21,1.21,0,0,0,1.26-1.25V21A1.31,1.31,0,0,0,54,19.65Z"/>
              <polygon points="49.91 27.49 48.12 27.49 46.33 27.49 46.33 31.97 48.12 31.97 49.91 31.97 51.7 31.97 51.7 27.49 49.91 27.49"/>
              <polygon points="49.91 33.76 48.12 33.76 46.33 33.76 46.33 38.24 48.12 38.24 49.91 38.24 51.7 38.24 51.7 33.76 49.91 33.76"/>
              <polygon points="49.91 40.02 48.12 40.02 46.33 40.02 46.33 44.5 48.12 44.5 49.91 44.5 51.7 44.5 51.7 40.02 49.91 40.02"/>
              <polygon points="42.76 27.49 40.97 27.49 39.18 27.49 39.18 31.97 40.97 31.97 42.76 31.97 44.54 31.97 44.54 27.49 42.76 27.49"/>
              <polygon points="42.76 33.76 40.97 33.76 39.18 33.76 39.18 38.24 40.97 38.24 42.76 38.24 44.54 38.24 44.54 33.76 42.76 33.76"/>
              <polygon points="42.76 40.02 40.97 40.02 39.18 40.02 39.18 44.5 40.97 44.5 42.76 44.5 44.54 44.5 44.54 40.02 42.76 40.02"/>
              <polygon points="42.76 46.29 40.97 46.29 39.18 46.29 39.18 50.77 40.97 50.77 42.76 50.77 44.54 50.77 44.54 46.29 42.76 46.29"/>
              <polygon points="36.56 27.49 35.73 27.49 34.89 27.49 34.89 31.97 35.73 31.97 36.56 31.97 37.39 31.97 37.39 27.49 36.56 27.49"/>
              <polygon points="36.56 33.76 35.73 33.76 34.89 33.76 34.89 38.24 35.73 38.24 36.56 38.24 37.39 38.24 37.39 33.76 36.56 33.76"/>
              <polygon points="36.56 40.02 35.73 40.02 34.89 40.02 34.89 44.5 35.73 44.5 36.56 44.5 37.39 44.5 37.39 40.02 36.56 40.02"/>
            </g>
            <polygon points="49.37 56.99 48.33 56.99 48.33 56.35 51.17 56.35 51.17 56.99 50.13 56.99 50.13 59.83 49.37 59.83 49.37 56.99 49.37 56.99"/>
            <polygon points="51.33 56.35 52.4 56.35 53.21 58.74 53.22 58.74 53.99 56.35 55.06 56.35 55.06 59.83 54.34 59.83 54.34 57.37 54.34 57.37 53.49 59.83 52.9 59.83 52.05 57.39 52.04 57.39 52.04 59.83 51.33 59.83 51.33 56.35"/>
          </g>
        </g>
        <g>
          <g>
            <ellipse cx="23.41" cy="34.93" rx="2.88" ry="4.39" fill="#fff"/>
            <path d="M12.73,20.57V49.79a.57.57,0,0,0,.47.56l21.13,3.91a.57.57,0,0,0,.67-.57V16.31a.57.57,0,0,0-.68-.57L13.19,20A.57.57,0,0,0,12.73,20.57ZM23.41,42.28c-3.23,0-5.85-3.29-5.85-7.35s2.62-7.36,5.85-7.36,5.86,3.29,5.86,7.36S26.65,42.28,23.41,42.28Z" fill="#fff"/>
          </g>
          <g>
            <polygon points="38.56 44.29 37.72 44.29 36.89 44.29 36.89 48.77 37.72 48.77 38.56 48.77 39.39 48.77 39.39 44.29 38.56 44.29" fill="#fff"/>
            <path d="M56,17.65H36.92V23.7H55.48V50.56H36.92v1.79H56a1.22,1.22,0,0,0,1.26-1.25V19A1.32,1.32,0,0,0,56,17.65Z" fill="#fff"/>
            <polygon points="51.9 25.49 50.12 25.49 48.33 25.49 48.33 29.97 50.12 29.97 51.9 29.97 53.69 29.97 53.69 25.49 51.9 25.49" fill="#fff"/>
            <polygon points="51.9 31.76 50.12 31.76 48.33 31.76 48.33 36.24 50.12 36.24 51.9 36.24 53.69 36.24 53.69 31.76 51.9 31.76" fill="#fff"/>
            <polygon points="51.9 38.02 50.12 38.02 48.33 38.02 48.33 42.5 50.12 42.5 51.9 42.5 53.69 42.5 53.69 38.02 51.9 38.02" fill="#fff"/>
            <polygon points="44.75 25.49 42.97 25.49 41.18 25.49 41.18 29.97 42.97 29.97 44.75 29.97 46.54 29.97 46.54 25.49 44.75 25.49" fill="#fff"/>
            <polygon points="44.75 31.76 42.97 31.76 41.18 31.76 41.18 36.24 42.97 36.24 44.75 36.24 46.54 36.24 46.54 31.76 44.75 31.76" fill="#fff"/>
            <polygon points="44.75 38.02 42.97 38.02 41.18 38.02 41.18 42.5 42.97 42.5 44.75 42.5 46.54 42.5 46.54 38.02 44.75 38.02" fill="#fff"/>
            <polygon points="44.75 44.29 42.97 44.29 41.18 44.29 41.18 48.77 42.97 48.77 44.75 48.77 46.54 48.77 46.54 44.29 44.75 44.29" fill="#fff"/>
            <polygon points="38.56 25.49 37.72 25.49 36.89 25.49 36.89 29.97 37.72 29.97 38.56 29.97 39.39 29.97 39.39 25.49 38.56 25.49" fill="#fff"/>
            <polygon points="38.56 31.76 37.72 31.76 36.89 31.76 36.89 36.24 37.72 36.24 38.56 36.24 39.39 36.24 39.39 31.76 38.56 31.76" fill="#fff"/>
            <polygon points="38.56 38.02 37.72 38.02 36.89 38.02 36.89 42.5 37.72 42.5 38.56 42.5 39.39 42.5 39.39 38.02 38.56 38.02" fill="#fff"/>
          </g>
          <polygon points="51.37 55 50.33 55 50.33 54.36 53.16 54.36 53.16 55 52.13 55 52.13 57.83 51.37 57.83 51.37 55 51.37 55" fill="#fff"/>
          <polygon points="53.33 54.36 54.4 54.36 55.21 56.74 55.22 56.74 55.98 54.36 57.05 54.36 57.05 57.83 56.34 57.83 56.34 55.37 56.33 55.37 55.48 57.83 54.9 57.83 54.05 55.39 54.04 55.39 54.04 57.83 53.33 57.83 53.33 54.36" fill="#fff"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\views\microsoft_calendar\microsoft_calendar_controller.js

```javascript
/** @odoo-module **/

import { AttendeeCalendarController } from "@calendar/views/attendee_calendar/attendee_calendar_controller";
import { patch } from "@web/core/utils/patch";
import { useService } from "@web/core/utils/hooks";
import { ConfirmationDialog, AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";

patch(AttendeeCalendarController.prototype, "microsoft_calendar_microsoft_calendar_controller", {
    setup() {
        this._super(...arguments);
        this.dialog = useService("dialog");
        this.notification = useService("notification");
    },

    async onMicrosoftSyncCalendar() {
        await this.orm.call(
            "res.users",
            "restart_microsoft_synchronization",
            [[this.user.userId]],
        );
        const syncResult = await this.model.syncMicrosoftCalendar();
        if (syncResult.status === "need_auth") {
            window.location.assign(syncResult.url);
        } else if (syncResult.status === "need_config_from_admin") {
            if (this.isSystemUser) {
                this.dialog.add(ConfirmationDialog, {
                    title: this.env._t("Configuration"),
                    body: this.env._t("The Outlook Synchronization needs to be configured before you can use it, do you want to do it now?"),
                    confirm: this.actionService.doAction.bind(this.actionService, syncResult.action),
                });
            } else {
                this.dialog.add(AlertDialog, {
                    title: this.env._t("Configuration"),
                    body: this.env._t("An administrator needs to configure Outlook Synchronization before you can use it!"),
                });
            }
        } else if (syncResult.status === "need_refresh") {
            await this.model.load();
        }
    },

    async onStopMicrosoftSynchronization() {
        this.dialog.add(ConfirmationDialog, {
            body: this.env._t("You are about to stop the synchronization of your calendar with Outlook. Are you sure you want to continue?"),
            confirm: async () => {
                await this.orm.call(
                    "res.users",
                    "stop_microsoft_synchronization",
                    [[this.user.userId]],
                );
                this.notification.add(
                    this.env._t("The synchronization with Outlook calendar was successfully stopped."),
                    {
                        title: this.env._t("Success"),
                        type: "success",
                    },
                );
                await this.model.load();
            },
        });
    }
});

```

## File: static\src\views\microsoft_calendar\microsoft_calendar_controller.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="microsoft_calendar.MicrosoftCalendarController" t-inherit="calendar.AttendeeCalendarController" t-inherit-mode="extension" owl="1">
        <xpath expr="//div[@id='calendar_sync_wrapper']" position="attributes">
            <attribute name="t-if">true</attribute>
        </xpath>
        <xpath expr="//div[@id='microsoft_calendar_sync']" position="replace">
            <div id="microsoft_calendar_sync" class="o_calendar_sync">
                <button t-if="!model.microsoftIsSync" type="button" id="microsoft_sync_pending" class="o_microsoft_sync_button o_microsoft_sync_pending btn btn-secondary btn" t-on-click="onMicrosoftSyncCalendar">
                    <b><i class='fa fa-refresh'/> Outlook</b>
                </button>
                <!-- class change on hover -->
                <button t-else="" type="button" id="microsoft_sync_configured" class="me-1 o_microsoft_sync_button o_microsoft_sync_button_configured btn text-bg-primary" t-on-click="onStopMicrosoftSynchronization"
                    t-on-mouseenter="(ev) => {ev.target.classList.remove('text-bg-primary');ev.target.classList.add('text-bg-danger');}"
                    t-on-mouseleave="(ev) => {ev.target.classList.remove('text-bg-danger');ev.target.classList.add('text-bg-primary');}">
                    <b>
                        <i id="microsoft_check" class='fa fa-check'/>
                        <i id="microsoft_stop" class='fa fa-times'/>
                        <span class="mx-1">Outlook</span>
                    </b>
                </button>
            </div>
        </xpath>
    </t>
</templates>

```

## File: static\src\views\microsoft_calendar\microsoft_calendar_model.js

```javascript
/** @odoo-module **/

import { AttendeeCalendarModel } from "@calendar/views/attendee_calendar/attendee_calendar_model";
import { patch } from "@web/core/utils/patch";

patch(AttendeeCalendarModel, "microsoft_calendar_microsoft_calendar_model", {
    services: [...AttendeeCalendarModel.services, "rpc"],
});

patch(AttendeeCalendarModel.prototype, "microsoft_calendar_microsoft_calendar_model_functions", {
    setup(params, { rpc }) {
        this._super(...arguments);
        this.rpc = rpc;
        this.isAlive = params.isAlive;
        this.microsoftIsSync = true;
        this.microsoftPendingSync = false;
    },

    /**
     * @override
     */
    async updateData() {
        const _super = this._super.bind(this);
        if (this.microsoftPendingSync) {
            return _super(...arguments);
        }
        try {
            await Promise.race([
                new Promise(resolve => setTimeout(resolve, 1000)),
                this.syncMicrosoftCalendar(true)
            ]);
        } catch (error) {
            if (error.event) {
                error.event.preventDefault();
            }
            console.error("Could not synchronize microsoft events now.", error);
            this.microsoftPendingSync = false;
        }
        if (this.isAlive()) {
            return _super(...arguments);
        }
    },

    async syncMicrosoftCalendar(silent = false) {
        this.microsoftPendingSync = true;
        const result = await this.rpc(
            "/microsoft_calendar/sync_data",
            {
                model: this.resModel,
                fromurl: window.location.href
            },
            {
                silent,
            },
        );
        if (["need_config_from_admin", "need_auth", "sync_stopped"].includes(result.status)) {
            this.microsoftIsSync = false;
        } else if (result.status === "no_new_event_from_microsoft" || result.status === "need_refresh") {
            this.microsoftIsSync = true;
        }
        this.microsoftPendingSync = false;
        return result;
    },
});

```

## File: static\src\views\microsoft_calendar\common\microsoft_calendar_common_popover.js

```javascript
/** @odoo-module **/

import { AttendeeCalendarCommonPopover } from "@calendar/views/attendee_calendar/common/attendee_calendar_common_popover";
import { patch } from "@web/core/utils/patch";

patch(AttendeeCalendarCommonPopover.prototype, "microsoft_calendar_microsoft_calendar_common_popover", {
    get isEventArchivable() {
        return this._super() || (this.isCurrentUserOrganizer && this.props.record.rawRecord.microsoft_id);
    },
});

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
    def _get_single_event(self, iCalUId, token, timeout=TIMEOUT):
        """ Fetch a single event from Graph API filtered by its iCalUId. """
        url = "/v1.0/me/events?$filter=iCalUId eq '%s'" % iCalUId
        headers = {'Content-type': 'application/json', 'Authorization': 'Bearer %s' % token}
        status, event, _dummy = self.microsoft_service._do_request(url, {}, headers, method='GET', timeout=timeout)
        return status not in RESOURCE_NOT_FOUND_STATUSES, event

    @requires_auth_token
    def _get_events_from_paginated_url(self, url, token=None, params=None, timeout=TIMEOUT):
        """
        Get a list of events from a paginated URL.
        Each page contains a link to the next page, so loop over all the pages to get all the events.
        """
        headers = {
            'Content-type': 'application/json',
            'Authorization': 'Bearer %s' % token,
            'Prefer': 'outlook.body-content-type="html", odata.maxpagesize=50'
        }
        if not params:
            # By default, fetch events from at most one year in the past and two years in the future.
            # Can be modified by microsoft_calendar.sync.range_days system parameter.
            day_range = int(self.microsoft_service.env['ir.config_parameter'].sudo().get_param('microsoft_calendar.sync.range_days', default=365))
            params = {
                'startDateTime': fields.Datetime.subtract(fields.Datetime.now(), days=day_range).strftime("%Y-%m-%dT00:00:00Z"),
                'endDateTime': fields.Datetime.add(fields.Datetime.now(), days=day_range * 2).strftime("%Y-%m-%dT00:00:00Z"),
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

    def _check_full_sync_required(self, response):
        """ Checks if full sync is required according to the error code received. """
        response_json = response.json()
        response_code = response_json.get('error', {}).get('code', '')
        return any(error_code in response_code for error_code in ['fullSyncRequired', 'SyncStateNotFound'])

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
            full_sync_needed = self._check_full_sync_required(e.response)
            if e.response.status_code == 410 and full_sync_needed and sync_token:
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
import json

from odoo.api import model
from typing import Iterator, Mapping
from collections import abc
from odoo.tools import ReadonlyDict, email_normalize
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
        value = self._events[event_id].get(name)
        json.dumps(value)
        return value

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

        # Query events OR recurrences, get organizer_id and universal_id values by splitting microsoft_id.
        model_env = force_model if force_model is not None else self._get_model(env)
        organiser_ids = tuple(str(v) for v in unmapped_events.ids if v) or ('NULL',)
        universal_ids = tuple(str(v) for v in unmapped_events.uids if v) or ('NULL',)
        model_env.flush_model(['microsoft_id'])
        env.cr.execute(
            """
                SELECT id, organizer_id, universal_id
                FROM (
                        SELECT id,
                                split_part(microsoft_id, ':', 1) AS organizer_id,
                                split_part(microsoft_id, ':', 2) AS universal_id
                            FROM %s
                            WHERE microsoft_id IS NOT NULL) AS splitter
                WHERE organizer_id IN %%s
                OR universal_id IN %%s
            """ % model_env._table, (organiser_ids, universal_ids))

        res = env.cr.fetchall()
        odoo_events_ids = [val[0] for val in res]
        odoo_events = model_env.browse(odoo_events_ids)

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

        if not self.organizer:
            return False

        organizer_email = self.organizer.get('emailAddress') and email_normalize(self.organizer.get('emailAddress').get('address'))
        if organizer_email:
            # Warning: In Microsoft: 1 email = 1 user; but in Odoo several users might have the same email
            user = env['res.users'].search([('email', '=', organizer_email)], limit=1)
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

        # daysOfWeek contains the full name of the day, the fields contain the first 3 letters (mon, tue, etc)
        week_days = [x[:3] for x in pattern.get('daysOfWeek', [])]
        for week_day in ['mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun']:
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
                        </div>
                        <div class="mt16 row">
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
            <field name="inherit_id" ref="calendar.res_users_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//page[@name='calendar']" position='attributes'>
                    <attribute name="invisible">0</attribute>
                </xpath>
                <group name="calendar_accounts" position="inside">
                    <group string="Outlook Calendar">
                        <group>
                            <field name="microsoft_calendar_rtoken" string="Refresh Token" readonly="1"/>
                            <field name="microsoft_calendar_token" string="User Token" readonly="1"/>
                            <field name="microsoft_calendar_token_validity" string="Token Validity" readonly="1"/>
                            <field name="microsoft_calendar_sync_token" string="Next Sync Token" readonly="1"/>
                            <button
                                string="Reset Account" class="btn btn-secondary mt-3"
                                type="action" name="%(microsoft_calendar_reset_account_action)d"
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
        # We don't update recurring events to prevent spam
        events = self.env['calendar.event'].search([
            ('user_id', '=', self.user_id.id),
            ('ms_universal_event_id', '!=', False)])
        non_recurring_events = self.env['calendar.event'].search([
            ('user_id', '=', self.user_id.id),
            ('recurrence_id', '=', False),
            ('ms_universal_event_id', '!=', False)])

        if self.delete_policy in ('delete_microsoft', 'delete_both'):
            for event in non_recurring_events:
                event._microsoft_delete(event._get_organizer(), event.ms_organizer_event_id, timeout=3)

        if self.sync_policy == 'all':
            events.with_context(dont_notify=True).update({
                'microsoft_id': False,
                'need_sync_m': True,
            })

        if self.delete_policy in ('delete_odoo', 'delete_both'):
            events.with_context(dont_notify=True).microsoft_id = False
            events.unlink()

        # We commit to make sure the _microsoft_delete are called when we still have a token on the user.
        self.env.cr.commit()
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
                    <button name="reset_account" string="Confirm" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z" />
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

