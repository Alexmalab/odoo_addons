# Odoo Module: google_calendar

Category: Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Google Calendar',
    'version': '1.0',
    'category': 'Tools',
    'description': "",
    'depends': ['google_account', 'calendar'],
    'qweb': ['static/src/xml/*.xml'],
    'data': [
        'data/google_calendar_data.xml',
        'data/google_calendar_data.xml',
        'security/ir.model.access.csv',
        'views/res_config_settings_views.xml',
        'views/res_users_views.xml',
        'views/google_calendar_templates.xml',
    ],
    'demo': [],
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request


class GoogleCalendarController(http.Controller):

    @http.route('/google_calendar/sync_data', type='json', auth='user')
    def sync_data(self, model, **kw):
        """ This route/function is called when we want to synchronize Odoo calendar with Google Calendar
            Function return a dictionary with the status :  need_config_from_admin, need_auth, need_refresh, success if not calendar_event
            The dictionary may contains an url, to allow Odoo Client to redirect user on this URL for authorization for example
        """
        if model == 'calendar.event':
            GoogleService = request.env['google.service']
            GoogleCal = request.env['google.calendar']

            # Checking that admin have already configured Google API for google synchronization !
            context = kw.get('local_context', {})
            client_id = GoogleService.with_context(context).get_client_id('calendar')

            if not client_id or client_id == '':
                action_id = ''
                if GoogleCal.can_authorize_google():
                    action_id = request.env.ref('base_setup.action_general_configuration').id
                return {
                    "status": "need_config_from_admin",
                    "url": '',
                    "action": action_id
                }

            # Checking that user have already accepted Odoo to access his calendar !
            if GoogleCal.need_authorize():
                url = GoogleCal.with_context(context).authorize_google_uri(from_url=kw.get('fromurl'))
                return {
                    "status": "need_auth",
                    "url": url
                }

            # If App authorized, and user access accepted, We launch the synchronization
            return GoogleCal.with_context(context).synchronize_events()

        return {"status": "success"}

    @http.route('/google_calendar/remove_references', type='json', auth='user')
    def remove_references(self, model, **kw):
        """ This route/function is called when we want to remove all the references between one calendar Odoo and one Google Calendar """
        status = "NOP"
        if model == 'calendar.event':
            GoogleCal = request.env['google.calendar']
            # Checking that user have already accepted Odoo to access his calendar !
            context = kw.get('local_context', {})
            if GoogleCal.with_context(context).remove_references():
                status = "OK"
            else:
                status = "KO"
        return {"status": status}

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
            <field name="model_id" ref="model_google_calendar"/>
            <field name="state">code</field>
            <field name="code">
# The context key 'last_sync_hours' allows specifying the minimum delay between consecutive syncs.
# Indeed, in case there are many users / events to sync, the cron might time out. In this case, the
# solution is to force the last_sync_hours to the expected synchronization interval expected. This
# will avoid the synchronization of users that succeeded in the previous failing cron.
model.synchronize_events_cron()
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

from odoo import api, fields, models


class Meeting(models.Model):

    _inherit = "calendar.event"

    oe_update_date = fields.Datetime('Odoo Update Date')

    @api.model
    def get_fields_need_update_google(self):
        recurrent_fields = self._get_recurrent_fields()
        return recurrent_fields + ['name', 'description', 'allday', 'start', 'date_end', 'stop',
                                   'attendee_ids', 'alarm_ids', 'location', 'privacy', 'active',
                                   'start_date', 'start_datetime', 'stop_date', 'stop_datetime']

    def write(self, values):
        sync_fields = set(self.get_fields_need_update_google())
        if (set(values) and sync_fields) and 'oe_update_date' not in values and 'NewMeeting' not in self._context:
            if 'oe_update_date' in self._context:
                values['oe_update_date'] = self._context.get('oe_update_date')
            else:
                values['oe_update_date'] = fields.Datetime.now()
        return super(Meeting, self).write(values)

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        default = default or {}
        if default.get('write_type', False):
            del default['write_type']
        elif default.get('recurrent_id', False):
            default['oe_update_date'] = fields.Datetime.now()
        else:
            default['oe_update_date'] = False
        return super(Meeting, self).copy(default)

    def unlink(self, can_be_deleted=False):
        return super(Meeting, self).unlink(can_be_deleted=can_be_deleted)


class Attendee(models.Model):

    _inherit = 'calendar.attendee'

    google_internal_event_id = fields.Char('Google Calendar Event Id')
    oe_synchro_date = fields.Datetime('Odoo Synchro Date')

    _sql_constraints = [
        ('google_id_uniq', 'unique(google_internal_event_id,partner_id,event_id)', 'Google ID should be unique!')
    ]

    def write(self, values):
        for attendee in self:
            meeting_id_to_update = values.get('event_id', attendee.event_id.id)

            # If attendees are updated, we need to specify that next synchro need an action
            # Except if it come from an update_from_google
            if not self._context.get('curr_attendee', False) and not self._context.get('NewMeeting', False):
                self.env['calendar.event'].browse(meeting_id_to_update).write({'oe_update_date': fields.Datetime.now()})
        return super(Attendee, self).write(values)

```

## File: models\google_calendar.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime, timedelta

import requests
from dateutil import parser
import json
import logging
import operator
import pytz
from werkzeug import urls

from odoo import api, fields, models, tools, _
from odoo.exceptions import UserError
from odoo.osv import expression
from odoo.tools import exception_to_unicode

_logger = logging.getLogger(__name__)


def status_response(status):
    return int(str(status)[0]) == 2


class Meta(type):
    """ This Meta class allow to define class as a structure, and so instancied variable
        in __init__ to avoid to have side effect alike 'static' variable """
    def __new__(typ, name, parents, attrs):
        methods = {k: v for k, v in attrs.items() if callable(v)}
        attrs = {k: v for k, v in attrs.items() if not callable(v)}

        def init(self, **kw):
            for key, val in attrs.items():
                setattr(self, key, val)
            for key, val in kw.items():
                assert key in attrs
                setattr(self, key, val)

        methods['__init__'] = init
        methods['__getitem__'] = getattr
        return type.__new__(typ, name, parents, methods)


Struct = Meta('Struct', (object,), {})

class OdooEvent(Struct):
    event = False
    found = False
    event_id = False
    isRecurrence = False
    isInstance = False
    update = False
    status = False
    attendee_id = False
    synchro = False


class GmailEvent(Struct):
    event = False
    found = False
    isRecurrence = False
    isInstance = False
    update = False
    status = False


class SyncEvent(object):
    def __init__(self):
        self.OE = OdooEvent()
        self.GG = GmailEvent()
        self.OP = None

    def __getitem__(self, key):
        return getattr(self, key)

    def compute_OP(self, modeFull=True):
        #If event are already in Gmail and in Odoo
        if self.OE.found and self.GG.found:
            is_owner = self.OE.event.env.user.id == self.OE.event.user_id.id
            #If the event has been deleted from one side, we delete on other side !
            if self.OE.status != self.GG.status and is_owner:
                self.OP = Delete((self.OE.status and "OE") or (self.GG.status and "GG"),
                                 'The event has been deleted from one side, we delete on other side !')
            #If event is not deleted !
            elif self.OE.status and (self.GG.status or (not is_owner and self.GG.update)):
                if abs(self.OE.update - self.GG.update) > timedelta(seconds=1):
                    if self.OE.update < self.GG.update:
                        tmpSrc = 'GG'
                    elif self.OE.update > self.GG.update:
                        tmpSrc = 'OE'
                    assert tmpSrc in ['GG', 'OE']

                    if self[tmpSrc].isRecurrence:
                        if self[tmpSrc].status:
                            self.OP = Update(tmpSrc, 'Only need to update, because i\'m active')
                        else:
                            self.OP = Exclude(tmpSrc, 'Need to Exclude (Me = First event from recurrence) from recurrence')

                    elif self[tmpSrc].isInstance:
                        self.OP = Update(tmpSrc, 'Only need to update, because already an exclu')
                    else:
                        self.OP = Update(tmpSrc, 'Simply Update... I\'m a single event')
                else:
                    if not self.OE.synchro or self.OE.synchro < (self.OE.update + timedelta(seconds=1)):
                        self.OP = Update('OE', 'Event already updated by another user, but not synchro with my google calendar')
                    else:
                        self.OP = NothingToDo("", 'Not update needed')
            else:
                self.OP = NothingToDo("", "Both are already deleted")

        # New in Odoo...  Create on create_events of synchronize function
        elif self.OE.found and not self.GG.found:
            if self.OE.status:
                self.OP = Delete('OE', 'Update or delete from GOOGLE')
            else:
                if not modeFull:
                    self.OP = Delete('GG', 'Deleted from Odoo, need to delete it from Gmail if already created')
                else:
                    self.OP = NothingToDo("", "Already Deleted in gmail and unlinked in Odoo")
        elif self.GG.found and not self.OE.found:
            tmpSrc = 'GG'
            if not self.GG.status and not self.GG.isInstance:
                # don't need to make something... because event has been created and deleted before the synchronization
                self.OP = NothingToDo("", 'Nothing to do... Create and Delete directly')
            else:
                if self.GG.isInstance:
                    if self[tmpSrc].status:
                        self.OP = Exclude(tmpSrc, 'Need to create the new exclu')
                    else:
                        self.OP = Exclude(tmpSrc, 'Need to copy and Exclude')
                else:
                    self.OP = Create(tmpSrc, 'New EVENT CREATE from GMAIL')

    def __str__(self):
        return self.__repr__()

    def __repr__(self):
        event_str = "\n\n---- A SYNC EVENT ---"
        event_str += "\n    ID          OE: %s " % (self.OE.event and self.OE.event.id)
        event_str += "\n    ID          GG: %s " % (self.GG.event and self.GG.event.get('id', False))
        event_str += "\n    Name        OE: %s " % (self.OE.event and self.OE.event.name.encode('utf8'))
        event_str += "\n    Name        GG: %s " % (self.GG.event and self.GG.event.get('summary', '').encode('utf8'))
        event_str += "\n    Found       OE:%5s vs GG: %5s" % (self.OE.found, self.GG.found)
        event_str += "\n    Recurrence  OE:%5s vs GG: %5s" % (self.OE.isRecurrence, self.GG.isRecurrence)
        event_str += "\n    Instance    OE:%5s vs GG: %5s" % (self.OE.isInstance, self.GG.isInstance)
        event_str += "\n    Synchro     OE: %10s " % (self.OE.synchro)
        event_str += "\n    Update      OE: %10s " % (self.OE.update)
        event_str += "\n    Update      GG: %10s " % (self.GG.update)
        event_str += "\n    Status      OE:%5s vs GG: %5s" % (self.OE.status, self.GG.status)
        if (self.OP is None):
            event_str += "\n    Action      %s" % "---!!!---NONE---!!!---"
        else:
            event_str += "\n    Action      %s" % type(self.OP).__name__
            event_str += "\n    Source      %s" % (self.OP.src)
            event_str += "\n    comment     %s" % (self.OP.info)
        return event_str


class SyncOperation(object):
    def __init__(self, src, info, **kw):
        self.src = src
        self.info = info
        for key, val in kw.items():
            setattr(self, key, val)

    def __str__(self):
        return 'in__STR__'


class Create(SyncOperation):
    pass


class Update(SyncOperation):
    pass


class Delete(SyncOperation):
    pass


class NothingToDo(SyncOperation):
    pass


class Exclude(SyncOperation):
    pass


class GoogleCalendar(models.AbstractModel):
    STR_SERVICE = 'calendar'
    _name = 'google.%s' % STR_SERVICE
    _description = 'Google Calendar'

    def generate_data(self, event, isCreating=False):
        if event.allday:
            start_date = fields.Date.to_string(event.start_date)
            final_date = fields.Date.to_string(event.stop_date + timedelta(days=1))
            type = 'date'
            vstype = 'dateTime'
        else:
            start_date = fields.Datetime.context_timestamp(self, event.start).isoformat('T')
            final_date = fields.Datetime.context_timestamp(self, event.stop).isoformat('T')
            type = 'dateTime'
            vstype = 'date'
        attendee_list = []
        for attendee in event.attendee_ids:
            email = tools.email_split(attendee.email)
            email = email[0] if email else 'NoEmail@mail.com'
            attendee_list.append({
                'email': email,
                'displayName': attendee.partner_id.name,
                'responseStatus': attendee.state or 'needsAction',
            })

        reminders = []
        for alarm in event.alarm_ids:
            reminders.append({
                "method": "email" if alarm.alarm_type == "email" else "popup",
                "minutes": alarm.duration_minutes
            })
        data = {
            "summary": event.name or '',
            "description": event.description or '',
            "start": {
                type: start_date,
                vstype: None,
                'timeZone': self.env.context.get('tz') or self.env.user.tz or 'UTC',
            },
            "end": {
                type: final_date,
                vstype: None,
                'timeZone': self.env.context.get('tz') or self.env.user.tz or 'UTC',
            },
            "attendees": attendee_list,
            "reminders": {
                "overrides": reminders,
                "useDefault": "false"
            },
            "location": event.location or '',
            "visibility": event['privacy'] or 'public',
        }
        if event.recurrency and event.rrule:
            data["recurrence"] = ["RRULE:" + event.rrule]

        if not event.active:
            data["state"] = "cancelled"

        if not self.get_need_synchro_attendee():
            data.pop("attendees")
        if isCreating:
            other_google_ids = [other_att.google_internal_event_id for other_att in event.attendee_ids
                                if other_att.google_internal_event_id and not other_att.google_internal_event_id.startswith('_')]
            if other_google_ids:
                data["id"] = other_google_ids[0]
        return data

    def create_an_event(self, event):
        """ Create a new event in google calendar from the given event in Odoo.
            :param event : record of calendar.event to export to google calendar
        """
        data = self.generate_data(event, isCreating=True)

        url = "/calendar/v3/calendars/%s/events?fields=%s&access_token=%s" % ('primary', urls.url_quote('id,updated'), self.get_token())
        headers = {'Content-type': 'application/json', 'Accept': 'text/plain'}
        data_json = json.dumps(data)
        try:
            return self.env['google.service']._do_request(url, data_json, headers, type='POST')
        except requests.HTTPError as e:
            try:
                response = e.response.json()
                error = response.get('error', {}).get('message')
            except Exception:
                error = None
            if not error:
                raise e
            message = _('The event "%s", %s (ID: %s) cannot be synchronized because of the following error: %s') % (
                event.name, event.start, event.id, error
            )
            raise UserError(message)

    def delete_an_event(self, event_id):
        """ Delete the given event in primary calendar of google cal.
            :param event_id : google cal identifier of the event to delete
        """
        params = {
            'access_token': self.get_token()
        }
        headers = {'Content-type': 'application/json', 'Accept': 'text/plain'}
        url = "/calendar/v3/calendars/%s/events/%s" % ('primary', event_id)

        try:
            response = self.env['google.service']._do_request(url, params, headers, type='DELETE')
        except requests.HTTPError as e:
            # For some unknown reason Google can also return a 403 response when the event is already cancelled.
            if e.response.status_code != 403:
                raise e
            _logger.info("Could not delete Google event %s" % event_id)
            return False
        return response

    def get_calendar_primary_id(self):
        """ In google calendar, you can have multiple calendar. But only one is
            the 'primary' one. This Calendar identifier is 'primary'.
        """
        params = {
            'fields': 'id',
            'access_token': self.get_token()
        }
        headers = {'Content-type': 'application/json', 'Accept': 'text/plain'}

        url = "/calendar/v3/calendars/primary"

        try:
            status, content, ask_time = self.env['google.service']._do_request(url, params, headers, type='GET')
        except requests.HTTPError as e:
            if e.response.status_code == 401:  # Token invalid / Acces unauthorized
                error_msg = _("Your token is invalid or has been revoked !")

                self.env.user.write({'google_calendar_token': False, 'google_calendar_token_validity': False})
                self.env.cr.commit()

                raise self.env['res.config.settings'].get_config_warning(error_msg)
            raise

        return (status_response(status), content['id'] or False, ask_time)

    def get_event_synchro_dict(self, lastSync=False, token=False, nextPageToken=False):
        """ Returns events on the 'primary' calendar from google cal.
            :returns dict where the key is the google_cal event id, and the value the details of the event,
                    defined at https://developers.google.com/google-apps/calendar/v3/reference/events/list
        """
        if not token:
            token = self.get_token()

        params = {
            'fields': 'items,nextPageToken',
            'access_token': token,
            'maxResults': 1000,
        }

        if lastSync:
            params['updatedMin'] = lastSync.strftime("%Y-%m-%dT%H:%M:%S.%fz")
            params['showDeleted'] = True
        else:
            params['timeMin'] = self.get_minTime().strftime("%Y-%m-%dT%H:%M:%S.%fz")

        headers = {'Content-type': 'application/json', 'Accept': 'text/plain'}

        url = "/calendar/v3/calendars/%s/events" % 'primary'
        if nextPageToken:
            params['pageToken'] = nextPageToken

        status, content, ask_time = self.env['google.service']._do_request(url, params, headers, type='GET')

        google_events_dict = {}
        for google_event in content['items']:
            google_events_dict[google_event['id']] = google_event

        if content.get('nextPageToken'):
            google_events_dict.update(
                self.get_event_synchro_dict(lastSync=lastSync, token=token, nextPageToken=content['nextPageToken'])
            )

        return google_events_dict

    def get_one_event_synchro(self, google_id):
        token = self.get_token()

        params = {
            'access_token': token,
            'maxResults': 1000,
            'showDeleted': True,
        }

        headers = {'Content-type': 'application/json', 'Accept': 'text/plain'}

        url = "/calendar/v3/calendars/%s/events/%s" % ('primary', google_id)
        try:
            status, content, ask_time = self.env['google.service']._do_request(url, params, headers, type='GET')
        except Exception as e:
            _logger.info("Calendar Synchro - In except of get_one_event_synchro")
            _logger.info(exception_to_unicode(e))
            return False

        return status_response(status) and content or False

    def update_to_google(self, oe_event, google_event):
        url = "/calendar/v3/calendars/%s/events/%s?fields=%s&access_token=%s" % ('primary', google_event['id'], 'id,updated', self.get_token())
        headers = {'Content-type': 'application/json', 'Accept': 'text/plain'}
        data = self.generate_data(oe_event)
        data['sequence'] = google_event.get('sequence', 0)
        data_json = json.dumps(data)

        try:
            status, content, ask_time = self.env['google.service']._do_request(url, data_json, headers, type='PATCH')
            update_date = datetime.strptime(content['updated'], "%Y-%m-%dT%H:%M:%S.%fz")
            oe_event.write({'oe_update_date': update_date})

            if self.env.context.get('curr_attendee'):
                self.env['calendar.attendee'].browse(self.env.context['curr_attendee']).write(
                    {'oe_synchro_date': update_date})
        except requests.HTTPError as e:
            if e.response.status_code != 403:
                raise e
            _logger.info("Could not update Google event %s" % google_event['id'])

    def update_an_event(self, event):
        data = self.generate_data(event)
        url = "/calendar/v3/calendars/%s/events/%s" % ('primary', event.google_internal_event_id)
        headers = {}
        data['access_token'] = self.get_token()

        status, response, ask_time = self.env['google.service']._do_request(url, data, headers, type='GET')
        #TO_CHECK : , if http fail, no event, do DELETE ?
        return response

    def update_recurrent_event_exclu(self, instance_id, event_ori_google_id, event_new):
        """ Update event on google calendar
            :param instance_id : new google cal identifier
            :param event_ori_google_id : origin google cal identifier
            :param event_new : record of calendar.event to modify
        """
        data = self.generate_data(event_new)
        url = "/calendar/v3/calendars/%s/events/%s?access_token=%s" % ('primary', instance_id, self.get_token())
        headers = {'Content-type': 'application/json'}

        _originalStartTime = dict()
        if event_new.allday:
            _originalStartTime['date'] = event_new.recurrent_id_date.strftime("%Y-%m-%d")
        else:
            _originalStartTime['dateTime'] = event_new.recurrent_id_date.strftime("%Y-%m-%dT%H:%M:%S.%fz")

        data.update(
            recurringEventId=event_ori_google_id,
            originalStartTime=_originalStartTime,
            sequence=self.get_sequence(instance_id)
        )
        data_json = json.dumps(data)
        return self.env['google.service']._do_request(url, data_json, headers, type='PUT')

    def create_from_google(self, event, partner_id):
        context_tmp = dict(self._context, NewMeeting=True)
        res = self.with_context(context_tmp).update_from_google(False, event.GG.event, "create")
        event.OE.event_id = res
        meeting = self.env['calendar.event'].browse(res)
        attendee_record = self.env['calendar.attendee'].search([('partner_id', '=', partner_id), ('event_id', '=', res)])
        # if on GC the user invite n email address which corresponds to contacts merged into a single partner,
        # here the attendee_record will contain n records with same partner_id - event_id
        # We just need to set the google_internal_event_id on the first one
        # As this data come from GC we may not delete attendees
        if not attendee_record.filtered(lambda att: att.google_internal_event_id):
            attendee_record[:1].with_context(context_tmp).write({'google_internal_event_id': event.GG.event['id']})
        attendee_record.with_context(context_tmp).write({'oe_synchro_date': meeting.oe_update_date})
        if meeting.recurrency:
            attendees = self.env['calendar.attendee'].sudo().search([('google_internal_event_id', '=ilike', '%s\_%%' % event.GG.event['id'])])
            excluded_recurrent_event_ids = set(attendee.event_id for attendee in attendees)
            for event in excluded_recurrent_event_ids:
                event.write({'recurrent_id': meeting.id, 'recurrent_id_date': event.start, 'user_id': meeting.user_id.id})
        return event

    def update_from_google(self, event, single_event_dict, type):
        """ Update an event in Odoo with information from google calendar
            :param event : record od calendar.event to update
            :param single_event_dict : dict of google cal event data
        """
        CalendarEvent = self.env['calendar.event'].with_context(no_mail_to_attendees=True)
        ResPartner = self.env['res.partner']
        CalendarAlarm = self.env['calendar.alarm']
        attendee_record = []
        alarm_record = set()
        partner_record = [(4, self.env.user.partner_id.id)]
        result = {}

        if self.get_need_synchro_attendee():
            for google_attendee in single_event_dict.get('attendees', []):
                partner_email = google_attendee.get('email')
                if type == "write":
                    for oe_attendee in event['attendee_ids']:
                        if oe_attendee.email == partner_email or partner_email in oe_attendee.partner_id.user_ids.mapped('google_calendar_cal_id'):
                            oe_attendee.write({'state': google_attendee['responseStatus'], 'google_internal_event_id': single_event_dict.get('id')})
                            google_attendee['found'] = True
                            continue

                if google_attendee.get('found'):
                    continue

                attendee = ResPartner.search([('user_ids.google_calendar_cal_id', '=ilike', partner_email)], limit=1)
                if not attendee:
                    attendee = ResPartner.search([('email', '=ilike', partner_email), ('user_ids', '!=', False)], limit=1)
                if not attendee:
                    attendee = ResPartner.search([('email', '=ilike', partner_email)], limit=1)
                if not attendee:
                    data = {
                        'email': partner_email,
                        'name': google_attendee.get("displayName", False) or partner_email
                    }
                    attendee = ResPartner.create(data)
                attendee = attendee.read(['email'])[0]
                partner_record.append((4, attendee.get('id')))
                attendee['partner_id'] = attendee.pop('id')
                attendee['state'] = google_attendee['responseStatus']
                attendee_record.append((0, 0, attendee))
        for google_alarm in single_event_dict.get('reminders', {}).get('overrides', []):
            alarm = CalendarAlarm.search(
                [
                    ('alarm_type', '=', google_alarm['method'] if google_alarm['method'] == 'email' else 'notification'),
                    ('duration_minutes', '=', google_alarm['minutes'])
                ], limit=1
            )
            if not alarm:
                data = {
                    'alarm_type': google_alarm['method'] if google_alarm['method'] == 'email' else 'notification',
                    'duration': google_alarm['minutes'],
                    'interval': 'minutes',
                    'name': "%s minutes - %s" % (google_alarm['minutes'], google_alarm['method'])
                }
                alarm = CalendarAlarm.create(data)
            alarm_record.add(alarm.id)

        UTC = pytz.timezone('UTC')
        if single_event_dict.get('start') and single_event_dict.get('end'):  # If not cancelled

            if single_event_dict['start'].get('dateTime', False) and single_event_dict['end'].get('dateTime', False):
                date = parser.parse(single_event_dict['start']['dateTime'])
                stop = parser.parse(single_event_dict['end']['dateTime'])
                date = str(date.astimezone(UTC))[:-6]
                stop = str(stop.astimezone(UTC))[:-6]
                allday = False
            else:
                date = single_event_dict['start']['date']
                stop = single_event_dict['end']['date']
                d_end = fields.Date.from_string(stop)
                allday = True
                d_end = d_end + timedelta(days=-1)
                stop = fields.Date.to_string(d_end)

            update_date = datetime.strptime(single_event_dict['updated'], "%Y-%m-%dT%H:%M:%S.%fz")
            result.update({
                'start': date,
                'stop': stop,
                'allday': allday
            })
        result.update({
            'attendee_ids': attendee_record,
            'partner_ids': list(set(partner_record)),
            'alarm_ids': [(6, 0, list(alarm_record))],

            'name': single_event_dict.get('summary', 'Event'),
            'description': single_event_dict.get('description', False),
            'location': single_event_dict.get('location', False),
            'privacy': single_event_dict.get('visibility', 'public'),
            'oe_update_date': update_date,
        })

        if single_event_dict.get("recurrence", False):
            rrule = [rule for rule in single_event_dict["recurrence"] if rule.startswith("RRULE:")][0][6:]
            result['rrule'] = rrule
        if type == "write":
            res = CalendarEvent.browse(event['id']).write(result)
        elif type == "copy":
            result['recurrency'] = True
            res = CalendarEvent.browse([event['id']]).write(result)
        elif type == "create":
            res = CalendarEvent.create(result).id

        if self.env.context.get('curr_attendee'):
            self.env['calendar.attendee'].with_context(no_mail_to_attendees=True).browse([self.env.context['curr_attendee']]).write({'oe_synchro_date': update_date, 'google_internal_event_id': single_event_dict.get('id', False)})
        return res

    def remove_references(self):
        current_user = self.env.user
        reset_data = {
            'google_calendar_rtoken': False,
            'google_calendar_token': False,
            'google_calendar_token_validity': False,
            'google_calendar_last_sync_date': False,
            'google_calendar_cal_id': False,
        }

        all_my_attendees = self.env['calendar.attendee'].search([('partner_id', '=', current_user.partner_id.id)])
        all_my_attendees.write({'oe_synchro_date': False, 'google_internal_event_id': False})
        return current_user.write(reset_data)

    @api.model
    def synchronize_events_cron(self):
        """ Call by the cron. """
        domain = [('google_calendar_last_sync_date', '!=', False)]
        if self.env.context.get('last_sync_hours'):
            last_sync_hours = self.env.context['last_sync_hours']
            last_sync_date = datetime.now() - timedelta(hours=last_sync_hours)
            domain = expression.AND([
                domain,
                [('google_calendar_last_sync_date', '<=', fields.Datetime.to_string(last_sync_date))]
            ])
        users = self.env['res.users'].search(domain, order='google_calendar_last_sync_date')
        _logger.info("Calendar Synchro - Started by cron")

        for user_to_sync in users.ids:
            _logger.info("Calendar Synchro - Starting synchronization for a new user [%s]", user_to_sync)
            try:
                resp = self.with_user(user_to_sync).synchronize_events(lastSync=True)
                if resp.get("status") == "need_reset":
                    _logger.info("[%s] Calendar Synchro - Failed - NEED RESET  !", user_to_sync)
                else:
                    _logger.info("[%s] Calendar Synchro - Done with status : %s  !", user_to_sync, resp.get("status"))
            except Exception as e:
                _logger.info("[%s] Calendar Synchro - Exception : %s !", user_to_sync, exception_to_unicode(e))
            # make commit after processing a user to avoid starting over in case of timeout error
            self.env.cr.commit()
        _logger.info("Calendar Synchro - Ended by cron")

    def synchronize_events(self, lastSync=True):
        """ This method should be called as the user to sync. """
        user_to_sync = self.ids and self.ids[0] or self.env.uid
        current_user = self.env['res.users'].sudo().browse(user_to_sync)

        recs = self.with_user(user_to_sync)
        status, current_google, ask_time = recs.get_calendar_primary_id()
        if current_user.google_calendar_cal_id:
            if current_google != current_user.google_calendar_cal_id:
                return {
                    "status": "need_reset",
                    "info": {
                        "old_name": current_user.google_calendar_cal_id,
                        "new_name": current_google
                    },
                    "url": ''
                }

            if lastSync and recs.get_last_sync_date() and not recs.get_disable_since_synchro():
                lastSync = recs.get_last_sync_date()
                _logger.info("[%s] Calendar Synchro - MODE SINCE_MODIFIED : %s !", user_to_sync, fields.Datetime.to_string(lastSync))
            else:
                lastSync = False
                _logger.info("[%s] Calendar Synchro - MODE FULL SYNCHRO FORCED", user_to_sync)
        else:
            current_user.write({'google_calendar_cal_id': current_google})
            lastSync = False
            _logger.info("[%s] Calendar Synchro - MODE FULL SYNCHRO - NEW CAL ID", user_to_sync)

        new_ids = []
        new_ids += recs.create_new_events()
        new_ids += recs.bind_recurring_events_to_google()

        res = recs.update_events(lastSync)

        current_user.write({'google_calendar_last_sync_date': ask_time})
        return {
            "status": res and "need_refresh" or "no_new_event_from_google",
            "url": ''
        }

    def create_new_events(self):
        """ Create event in google calendar for the event not already
            synchronized, for the current user.
            :returns list of new created event identifier in google calendar
        """
        new_ids = []
        my_partner_id = self.env.user.partner_id.id

        my_attendees = self.env['calendar.attendee'].with_context(virtual_id=False).search([('partner_id', '=', my_partner_id),
            ('google_internal_event_id', '=', False),
            '|',
            ('event_id.stop', '>', fields.Datetime.to_string(self.get_minTime())),
            ('event_id.final_date', '>', fields.Datetime.to_string(self.get_minTime())),
        ])
        for att in my_attendees:
            # Get the attendees of the same event with google id set
            other_attendees = att.event_id.attendee_ids.filtered(lambda other_att: other_att.google_internal_event_id and other_att.id != att.id and not other_att.google_internal_event_id.startswith('_'))
            if att.partner_id in other_attendees.mapped('partner_id'):
                # After a contacts merge there may be events
                # with multiple attendee records of the same partner.
                # Deleting duplicate attendee to avoid raising error on constraint google_id_uniq
                att.unlink()
                continue
            for other_google_id in other_attendees.mapped('google_internal_event_id'):
                # Set google id on this attendee
                if self.get_one_event_synchro(other_google_id):
                    att.write({'google_internal_event_id': other_google_id})
                    break
            else:
                if not att.event_id.recurrent_id or att.event_id.recurrent_id == 0:
                    status, response, ask_time = self.create_an_event(att.event_id)
                    if status_response(status):
                        update_date = datetime.strptime(response['updated'], "%Y-%m-%dT%H:%M:%S.%fz")
                        att.event_id.write({'oe_update_date': update_date})
                        new_ids.append(response['id'])
                        att.write({'google_internal_event_id': response['id'], 'oe_synchro_date': update_date})
                        self.env.cr.commit()
                    else:
                        _logger.warning("Impossible to create event %s. [%s] Enable DEBUG for response detail.", att.event_id.id, status)
                        _logger.debug("Response : %s", response)
        return new_ids

    def get_context_no_virtual(self):
        """ get the current context modified to prevent virtual ids and active test. """
        return dict(self.env.context, virtual_id=False, active_test=False)

    def bind_recurring_events_to_google(self):
        new_ids = []
        CalendarAttendee = self.env['calendar.attendee']
        my_partner_id = self.env.user.partner_id.id
        context_norecurrent = self.get_context_no_virtual()
        my_attendees = CalendarAttendee.with_context(context_norecurrent).search([('partner_id', '=', my_partner_id), ('google_internal_event_id', '=', False)])
        for att in my_attendees:
            new_google_internal_event_id = False
            source_event_record = self.env['calendar.event'].browse(att.event_id.recurrent_id)
            source_attendee_record = CalendarAttendee.search([('partner_id', '=', my_partner_id), ('event_id', '=', source_event_record.id)], limit=1)
            if not source_attendee_record:
                continue

            recurrent_id_date = fields.Datetime.to_string(att.event_id.recurrent_id_date)
            if recurrent_id_date and source_event_record.allday and source_attendee_record.google_internal_event_id:
                new_google_internal_event_id = source_attendee_record.google_internal_event_id + '_' + recurrent_id_date.split(' ')[0].replace('-', '')
            elif recurrent_id_date and source_attendee_record.google_internal_event_id:
                new_google_internal_event_id = source_attendee_record.google_internal_event_id + '_' + recurrent_id_date.replace('-', '').replace(' ', 'T').replace(':', '') + 'Z'

            if new_google_internal_event_id:
                #TODO WARNING, NEED TO CHECK THAT EVENT and ALL instance NOT DELETE IN GMAIL BEFORE !
                try:
                    status, response, ask_time = self.update_recurrent_event_exclu(new_google_internal_event_id, source_attendee_record.google_internal_event_id, att.event_id)
                    if status_response(status):
                        att.write({'google_internal_event_id': new_google_internal_event_id})
                        new_ids.append(new_google_internal_event_id)
                        self.env.cr.commit()
                    else:
                        _logger.warning("Impossible to create event %s. [%s]", att.event_id.id, status)
                        _logger.debug("Response : %s", response)
                except Exception as e:
                    _logger.warning("Exception when updating recurrent event exclusions on google: %s", e)

        return new_ids

    def update_events(self, lastSync=False):
        """ Synchronze events with google calendar : fetching, creating, updating, deleting, ... """
        CalendarEvent = self.env['calendar.event']
        CalendarAttendee = self.env['calendar.attendee']
        my_partner_id = self.env.user.partner_id.id
        context_novirtual = self.get_context_no_virtual()

        if lastSync:
            try:
                all_event_from_google = self.get_event_synchro_dict(lastSync=lastSync)
            except requests.HTTPError as e:
                if e.response.status_code == 410:  # GONE, Google is lost.
                    # we need to force the rollback from this cursor, because it locks my res_users but I need to write in this tuple before to raise.
                    self.env.cr.rollback()
                    self.env.user.write({'google_calendar_last_sync_date': False})
                    self.env.cr.commit()
                error_key = e.response.json()
                error_key = error_key.get('error', {}).get('message', 'nc')
                error_msg = _("Google is lost... the next synchro will be a full synchro. \n\n %s") % error_key
                raise self.env['res.config.settings'].get_config_warning(error_msg)

            my_google_attendees = CalendarAttendee.with_context(context_novirtual).search([
                ('partner_id', '=', my_partner_id),
                ('google_internal_event_id', 'in', list(all_event_from_google))
            ])
            my_google_att_ids = my_google_attendees.ids

            my_odoo_attendees = CalendarAttendee.with_context(context_novirtual).search([
                ('partner_id', '=', my_partner_id),
                ('event_id.oe_update_date', '>', lastSync and fields.Datetime.to_string(lastSync) or self.get_minTime().fields.Datetime.to_string()),
                ('google_internal_event_id', '!=', False),
            ])

            my_odoo_googleinternal_records = my_odoo_attendees.read(['google_internal_event_id', 'event_id'])

            if self.get_print_log():
                _logger.info("Calendar Synchro -  \n\nUPDATE IN GOOGLE\n%s\n\nRETRIEVE FROM OE\n%s\n\nUPDATE IN OE\n%s\n\nRETRIEVE FROM GG\n%s\n\n", all_event_from_google, my_google_att_ids, my_odoo_attendees.ids, my_odoo_googleinternal_records)

            for gi_record in my_odoo_googleinternal_records:
                active = True  # if not sure, we request google
                if gi_record.get('event_id'):
                    active = CalendarEvent.with_context(context_novirtual).browse(int(gi_record.get('event_id')[0])).active

                if gi_record.get('google_internal_event_id') and not all_event_from_google.get(gi_record.get('google_internal_event_id')) and active:
                    one_event = self.get_one_event_synchro(gi_record.get('google_internal_event_id'))
                    if one_event:
                        all_event_from_google[one_event['id']] = one_event

            my_attendees = (my_google_attendees | my_odoo_attendees)

        else:
            domain = [
                ('partner_id', '=', my_partner_id),
                ('google_internal_event_id', '!=', False),
                '|',
                ('event_id.stop', '>', fields.Datetime.to_string(self.get_minTime())),
                ('event_id.final_date', '>', fields.Datetime.to_string(self.get_minTime())),
            ]

            # Select all events from Odoo which have been already synchronized in gmail
            my_attendees = CalendarAttendee.with_context(context_novirtual).search(domain)
            all_event_from_google = self.get_event_synchro_dict(lastSync=False)

        event_to_synchronize = {}
        for att in my_attendees:
            event = att.event_id

            base_event_id = att.google_internal_event_id.rsplit('_', 1)[0]

            if base_event_id not in event_to_synchronize:
                event_to_synchronize[base_event_id] = {}

            if att.google_internal_event_id not in event_to_synchronize[base_event_id]:
                event_to_synchronize[base_event_id][att.google_internal_event_id] = SyncEvent()

            ev_to_sync = event_to_synchronize[base_event_id][att.google_internal_event_id]

            ev_to_sync.OE.attendee_id = att.id
            ev_to_sync.OE.event = event
            ev_to_sync.OE.found = True
            ev_to_sync.OE.event_id = event.id
            ev_to_sync.OE.isRecurrence = event.recurrency
            ev_to_sync.OE.isInstance = bool(event.recurrent_id and event.recurrent_id > 0)
            ev_to_sync.OE.update = event.oe_update_date
            ev_to_sync.OE.status = event.active
            ev_to_sync.OE.synchro = att.oe_synchro_date

        for event in all_event_from_google.values():
            event_id = event.get('id')
            base_event_id = event_id.rsplit('_', 1)[0]

            if base_event_id not in event_to_synchronize:
                event_to_synchronize[base_event_id] = {}

            if event_id not in event_to_synchronize[base_event_id]:
                event_to_synchronize[base_event_id][event_id] = SyncEvent()

            ev_to_sync = event_to_synchronize[base_event_id][event_id]

            ev_to_sync.GG.event = event
            ev_to_sync.GG.found = True
            ev_to_sync.GG.isRecurrence = bool(event.get('recurrence', ''))
            ev_to_sync.GG.isInstance = bool(event.get('recurringEventId', 0))
            ev_to_sync.GG.update = event.get('updated') and parser.parse(event['updated']) or None  # if deleted, no date without browse event
            if ev_to_sync.GG.update:
                ev_to_sync.GG.update = ev_to_sync.GG.update.replace(tzinfo=None)
            ev_to_sync.GG.status = (event.get('status') != 'cancelled')

        ######################
        #   PRE-PROCESSING   #
        ######################
        for base_event in event_to_synchronize:
            for current_event in event_to_synchronize[base_event]:
                event_to_synchronize[base_event][current_event].compute_OP(modeFull=not lastSync)
            if self.get_print_log():
                if not isinstance(event_to_synchronize[base_event][current_event].OP, NothingToDo):
                    _logger.info(event_to_synchronize[base_event])

        ######################
        #      DO ACTION     #
        ######################
        for base_event in event_to_synchronize:
            event_to_synchronize[base_event] = sorted(event_to_synchronize[base_event].items(), key=operator.itemgetter(0))
            for current_event in event_to_synchronize[base_event]:
                self.env.cr.commit()
                event = current_event[1]  # event is an Sync Event !
                actToDo = event.OP
                actSrc = event.OP.src

                # To avoid redefining 'self', all method below should use 'recs' instead of 'self'
                recs = self.with_context(curr_attendee=event.OE.attendee_id)

                if isinstance(actToDo, NothingToDo):
                    continue
                elif isinstance(actToDo, Create):
                    if actSrc == 'GG':
                        self.create_from_google(event, my_partner_id)
                    elif actSrc == 'OE':
                        raise AssertionError("Should be never here, creation for OE is done before update !")
                    #TODO Add to batch
                elif isinstance(actToDo, Update):
                    if actSrc == 'GG':
                        recs.update_from_google(event.OE.event, event.GG.event, 'write')
                    elif actSrc == 'OE':
                        recs.update_to_google(event.OE.event, event.GG.event)
                elif isinstance(actToDo, Exclude):
                    if actSrc == 'OE':
                        recs.delete_an_event(current_event[0])
                    elif actSrc == 'GG':
                        new_google_event_id = event.GG.event['id'].rsplit('_', 1)[1]
                        parent_oe = event_to_synchronize[base_event][0][1].OE.event
                        if 'T' in new_google_event_id:
                            new_google_event_id = new_google_event_id.replace('T', '')[:-1]
                        else:
                            #allday event, need to match the changes that will be applied with _inverse_dates otherwise the exclusion will not occur
                            if parent_oe:
                                new_google_event_id = new_google_event_id + parent_oe.start.strftime("%H%M%S")
                            else:
                                new_google_event_id = new_google_event_id + "000000"

                        if event.GG.status:
                            parent_event = {}
                            if not event_to_synchronize[base_event][0][1].OE.event_id:
                                main_ev = CalendarAttendee.with_context(context_novirtual).search([('google_internal_event_id', '=', event.GG.event['id'].rsplit('_', 1)[0])], limit=1)
                                event_to_synchronize[base_event][0][1].OE.event_id = main_ev.event_id.id

                            if event_to_synchronize[base_event][0][1].OE.event_id:
                                parent_event['id'] = "%s-%s" % (event_to_synchronize[base_event][0][1].OE.event_id, new_google_event_id)
                                res = recs.update_from_google(parent_event, event.GG.event, "copy")
                            else:
                                recs.create_from_google(event, my_partner_id)
                        else:
                            if parent_oe:
                                CalendarEvent.browse("%s-%s" % (parent_oe.id, new_google_event_id)).with_context(curr_attendee=event.OE.attendee_id).unlink(can_be_deleted=True)
                            else:
                                main_att = CalendarAttendee.with_context(context_novirtual).search([('partner_id', '=', my_partner_id), ('google_internal_event_id', '=', event.GG.event['id'].rsplit('_', 1)[0])], limit=1)
                                if main_att:
                                    excluded_event_id = str(main_att.event_id.id) + '-' + new_google_event_id

                                    CalendarEvent.browse(excluded_event_id).with_context(google_internal_event_id=event.GG.event.get('id'), oe_update_date=False).unlink(can_be_deleted=False)
                                else:
                                    _logger.warn("Could not create the correct exclusion for event")

                elif isinstance(actToDo, Delete):
                    if actSrc == 'GG':
                        try:
                            # if already deleted from gmail or never created
                            recs.delete_an_event(current_event[0])
                        except requests.exceptions.HTTPError as e:
                            if e.response.status_code in (401, 410,):
                                _logger.info("Google event %s already deleted or never created" % current_event[0])
                            else:
                                raise e
                    elif actSrc == 'OE':
                        CalendarEvent.browse(event.OE.event_id).unlink(can_be_deleted=False)
        return True

    def check_and_sync(self, oe_event, google_event):
        if oe_event.oe_update_date > datetime.strptime(google_event['updated'], "%Y-%m-%dT%H:%M:%S.%fz"):
            self.update_to_google(oe_event, google_event)
        elif oe_event.oe_update_date < datetime.strptime(google_event['updated'], "%Y-%m-%dT%H:%M:%S.%fz"):
            self.update_from_google(oe_event, google_event, 'write')

    def get_sequence(self, instance_id):
        params = {
            'fields': 'sequence',
            'access_token': self.get_token()
        }
        headers = {'Content-type': 'application/json'}
        url = "/calendar/v3/calendars/%s/events/%s" % ('primary', instance_id)
        status, content, ask_time = self.env['google.service']._do_request(url, params, headers, type='GET')
        return content.get('sequence', 0)

    #################################
    ##  MANAGE CONNEXION TO GMAIL  ##
    #################################

    def get_token(self):
        current_user = self.env.user
        if not current_user.google_calendar_token_validity or \
                current_user.google_calendar_token_validity < (datetime.now() + timedelta(minutes=1)):
            self.do_refresh_token()
            current_user.refresh()
        return current_user.google_calendar_token

    def get_last_sync_date(self):
        current_user = self.env.user
        return current_user.google_calendar_last_sync_date and fields.Datetime.from_string(current_user.google_calendar_last_sync_date) + timedelta(minutes=0) or False

    def do_refresh_token(self):
        current_user = self.env.user
        all_token = self.env['google.service']._refresh_google_token_json(current_user.google_calendar_rtoken, self.STR_SERVICE)

        vals = {}
        vals['google_%s_token_validity' % self.STR_SERVICE] = datetime.now() + timedelta(seconds=all_token.get('expires_in'))
        vals['google_%s_token' % self.STR_SERVICE] = all_token.get('access_token')

        self.env.user.sudo().write(vals)

    def need_authorize(self):
        current_user = self.env.user
        return current_user.google_calendar_rtoken is False

    def get_calendar_scope(self, RO=False):
        readonly = '.readonly' if RO else ''
        return 'https://www.googleapis.com/auth/calendar%s' % (readonly)

    def authorize_google_uri(self, from_url='http://www.odoo.com'):
        url = self.env['google.service']._get_authorize_uri(from_url, self.STR_SERVICE, scope=self.get_calendar_scope())
        return url

    def can_authorize_google(self):
        return self.env['res.users'].has_group('base.group_erp_manager')

    @api.model
    def set_all_tokens(self, authorization_code):
        all_token = self.env['google.service']._get_google_token_json(authorization_code, self.STR_SERVICE)

        vals = {}
        vals['google_%s_rtoken' % self.STR_SERVICE] = all_token.get('refresh_token')
        vals['google_%s_token_validity' % self.STR_SERVICE] = datetime.now() + timedelta(seconds=all_token.get('expires_in'))
        vals['google_%s_token' % self.STR_SERVICE] = all_token.get('access_token')
        self.env.user.sudo().write(vals)

    def get_minTime(self):
        number_of_week = self.env['ir.config_parameter'].sudo().get_param('calendar.week_synchro', default=13)
        return datetime.now() - timedelta(weeks=int(number_of_week))

    def get_need_synchro_attendee(self):
        return not (self.env['ir.config_parameter'].sudo().get_param('calendar.block_synchro_attendee', default=False) == 'True')

    def get_disable_since_synchro(self):
        return self.env['ir.config_parameter'].sudo().get_param('calendar.block_since_synchro', default=False)

    def get_print_log(self):
        return self.env['ir.config_parameter'].sudo().get_param('calendar.debug_print', default=False)

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
    server_uri = fields.Char('URI for tuto')

    @api.model
    def get_values(self):
        res = super(ResConfigSettings, self).get_values()
        get_param = self.env['ir.config_parameter'].sudo().get_param
        res.update(
            server_uri="%s/google_account/authentication" % get_param('web.base.url', default="http://yourcompany.odoo.com"),
        )
        return res

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class User(models.Model):

    _inherit = 'res.users'

    google_calendar_rtoken = fields.Char('Refresh Token', copy=False)
    google_calendar_token = fields.Char('User token', copy=False)
    google_calendar_token_validity = fields.Datetime('Token Validity', copy=False)
    google_calendar_last_sync_date = fields.Datetime('Last synchro date', copy=False)
    google_calendar_cal_id = fields.Char('Calendar ID', copy=False, help='Last Calendar ID who has been synchronized. If it is changed, we remove all links between GoogleID and Odoo Google Internal ID')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_config_settings
from . import calendar
from . import google_calendar
from . import res_users

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_google_calendar_all,access_google_calendar_all,model_google_calendar,,1,0,0,0
access_google_calendar,access_google_calendar,model_google_calendar,base.group_system,1,1,1,1

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
odoo.define('google_calendar.google_calendar', function (require) {
"use strict";

var core = require('web.core');
var Dialog = require('web.Dialog');
var framework = require('web.framework');
var CalendarRenderer = require('web.CalendarRenderer');
var CalendarController = require('web.CalendarController');

var _t = core._t;

CalendarController.include({
    custom_events: _.extend({}, CalendarController.prototype.custom_events, {
        syncCalendar: '_onSyncCalendar',
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
    _onSyncCalendar: function (event) {
        var self = this;
        var context = this.getSession().user_context;

        this._rpc({
            route: '/google_calendar/sync_data',
            params: {
                model: this.modelName,
                fromurl: window.location.href,
                local_context: context,
            }
        }).then(function (o) {
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
            } else if (o.status === "need_reset") {
                var confirmText1 = _t("The account you are trying to synchronize (%s) is not the same as the last one used (%s)!");
                var confirmText2 = _t("In order to do this, you first need to disconnect all existing events from the old account.");
                var confirmText3 = _t("Do you want to do this now?");
                var text = _.str.sprintf(confirmText1 + "\n" + confirmText2 + "\n\n" + confirmText3, o.info.new_name, o.info.old_name);
                Dialog.confirm(self, text, {
                    confirm_callback: function() {
                        self._rpc({
                                route: '/google_calendar/remove_references',
                                params: {
                                    model: self.state.model,
                                    local_context: context,
                                },
                            })
                            .then(function(o) {
                                if (o.status === "OK") {
                                    Dialog.alert(self, _t("All events have been disconnected from your previous account. You can now restart the synchronization"), {
                                        title: _t('Event disconnection success'),
                                    });
                                } else if (o.status === "KO") {
                                    Dialog.alert(self, _t("An error occured while disconnecting events from your previous account. Please retry or contact your administrator."), {
                                        title: _t('Event disconnection error'),
                                    });
                                } // else NOP
                            });
                    },
                    title: _t('Accounts'),
                });
            }
        }).then(event.data.on_always, event.data.on_always);
    }
});

CalendarRenderer.include({
    events: _.extend({}, CalendarRenderer.prototype.events, {
        'click .o_google_sync_button': '_onSyncCalendar',
    }),

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Adds the Sync with Google button in the sidebar
     *
     * @private
     */
    _initSidebar: function () {
        var self = this;
        this._super.apply(this, arguments);
        this.$googleButton = $();
        if (this.model === "calendar.event") {
            this.$googleButton = $('<button/>', {type: 'button', html: _t("Sync with <b>Google</b>")})
                                .addClass('o_google_sync_button oe_button btn btn-secondary')
                                .prepend($('<img/>', {
                                    src: "/google_calendar/static/src/img/calendar_32.png",
                                }))
                                .appendTo(self.$sidebar);
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
    _onSyncCalendar: function () {
        var self = this;
        var context = this.getSession().user_context;
        this.$googleButton.prop('disabled', true);
        this.trigger_up('syncCalendar', {
            on_always: function () {
                self.$googleButton.prop('disabled', false);
            },
        });
    },
});

});

```

## File: views\google_calendar_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="google_calendar assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/google_calendar/static/src/scss/google_calendar.scss"/>
            <script type="text/javascript" src="/google_calendar/static/src/js/google_calendar.js"></script>
        </xpath>
    </template>

    <template id="qunit_suite" name="google_calendar_tests" inherit_id="web.qunit_suite">
        <xpath expr="//t[@t-set='head']" position="inside">
            <script type="text/javascript" src="/google_calendar/static/tests/google_calendar_tests.js"></script>
        </xpath>
    </template>
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
                        <a href="https://www.odoo.com/documentation/13.0/applications/sales/crm/optimize/google_calendar_credentials.html" class="oe-link" target="_blank"><i class="fa fa-fw fa-arrow-right"/>Tutorial</a>
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
                    <page string="Calendar">
                        <group>
                            <field name="google_calendar_rtoken"/>
                            <field name="google_calendar_token"/>
                            <field name="google_calendar_token_validity"/>
                            <field name="google_calendar_last_sync_date"/>
                            <field name="google_calendar_cal_id"/>
                        </group>
                    </page>
                </notebook>
            </field>
        </record>

</odoo>


```

