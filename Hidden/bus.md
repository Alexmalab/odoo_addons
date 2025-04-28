# Odoo Module: bus

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
from . import models
from . import controllers

```

## File: __manifest__.py

```python
{
    'name' : 'IM Bus',
    'version': '1.0',
    'category': 'Hidden',
    'complexity': 'easy',
    'description': "Instant Messaging Bus allow you to send messages to users, in live.",
    'depends': ['base', 'web'],
    'data': [
        'views/bus.xml',
        'security/ir.model.access.csv',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-

from odoo import exceptions, _
from odoo.http import Controller, request, route
from odoo.addons.bus.models.bus import dispatch


class BusController(Controller):
    """ Examples:
    openerp.jsonRpc('/longpolling/poll','call',{"channels":["c1"],last:0}).then(function(r){console.log(r)});
    openerp.jsonRpc('/longpolling/send','call',{"channel":"c1","message":"m1"});
    openerp.jsonRpc('/longpolling/send','call',{"channel":"c2","message":"m2"});
    """

    @route('/longpolling/send', type="json", auth="public")
    def send(self, channel, message):
        if not isinstance(channel, str):
            raise Exception("bus.Bus only string channels are allowed.")
        return request.env['bus.bus'].sendone(channel, message)

    # override to add channels
    def _poll(self, dbname, channels, last, options):
        # update the user presence
        if request.session.uid and 'bus_inactivity' in options:
            request.env['bus.presence'].update(options.get('bus_inactivity'))
        request.cr.close()
        request._cr = None
        return dispatch.poll(dbname, channels, last, options)

    @route('/longpolling/poll', type="json", auth="public", cors="*")
    def poll(self, channels, last, options=None):
        if options is None:
            options = {}
        if not dispatch:
            raise Exception("bus.Bus unavailable")
        if [c for c in channels if not isinstance(c, str)]:
            raise Exception("bus.Bus only string channels are allowed.")
        if request.registry.in_test_mode():
            raise exceptions.UserError(_("bus.Bus not available in test mode"))
        return self._poll(request.db, channels, last, options)

    @route('/longpolling/im_status', type="json", auth="user")
    def im_status(self, partner_ids):
        return request.env['res.partner'].with_context(active_test=False).search([('id', 'in', partner_ids)]).read(['im_status'])

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
from . import main

```

## File: models\bus.py

```python
# -*- coding: utf-8 -*-
import datetime
import json
import logging
import os
import random
import select
import threading
import time

import odoo
from odoo import api, fields, models, SUPERUSER_ID
from odoo.tools.misc import DEFAULT_SERVER_DATETIME_FORMAT
from odoo.tools import date_utils

from psycopg2 import sql

_logger = logging.getLogger(__name__)

# longpolling timeout connection
TIMEOUT = 50

# custom function to call instead of NOTIFY postgresql command (opt-in)
ODOO_NOTIFY_FUNCTION = os.environ.get('ODOO_NOTIFY_FUNCTION')

#----------------------------------------------------------
# Bus
#----------------------------------------------------------
def json_dump(v):
    return json.dumps(v, separators=(',', ':'), default=date_utils.json_default)

def hashable(key):
    if isinstance(key, list):
        key = tuple(key)
    return key


class ImBus(models.Model):

    _name = 'bus.bus'
    _description = 'Communication Bus'

    channel = fields.Char('Channel')
    message = fields.Char('Message')

    @api.autovacuum
    def _gc_messages(self):
        timeout_ago = datetime.datetime.utcnow()-datetime.timedelta(seconds=TIMEOUT*2)
        domain = [('create_date', '<', timeout_ago.strftime(DEFAULT_SERVER_DATETIME_FORMAT))]
        return self.sudo().search(domain).unlink()

    @api.model
    def sendmany(self, notifications):
        channels = set()
        for channel, message in notifications:
            channels.add(channel)
            values = {
                "channel": json_dump(channel),
                "message": json_dump(message)
            }
            self.sudo().create(values)
        if channels:
            # We have to wait until the notifications are commited in database.
            # When calling `NOTIFY imbus`, some concurrent threads will be
            # awakened and will fetch the notification in the bus table. If the
            # transaction is not commited yet, there will be nothing to fetch,
            # and the longpolling will return no notification.
            @self.env.cr.postcommit.add
            def notify():
                with odoo.sql_db.db_connect('postgres').cursor() as cr:
                    if ODOO_NOTIFY_FUNCTION:
                        query = sql.SQL("SELECT {}('imbus', %s)").format(sql.Identifier(ODOO_NOTIFY_FUNCTION))
                    else:
                        query = "NOTIFY imbus, %s"
                    cr.execute(query, (json_dump(list(channels)), ))

    @api.model
    def sendone(self, channel, message):
        self.sendmany([[channel, message]])

    @api.model
    def poll(self, channels, last=0, options=None):
        if options is None:
            options = {}
        # first poll return the notification in the 'buffer'
        if last == 0:
            timeout_ago = datetime.datetime.utcnow()-datetime.timedelta(seconds=TIMEOUT)
            domain = [('create_date', '>', timeout_ago.strftime(DEFAULT_SERVER_DATETIME_FORMAT))]
        else:  # else returns the unread notifications
            domain = [('id', '>', last)]
        channels = [json_dump(c) for c in channels]
        domain.append(('channel', 'in', channels))
        notifications = self.sudo().search_read(domain)
        # list of notification to return
        result = []
        for notif in notifications:
            result.append({
                'id': notif['id'],
                'channel': json.loads(notif['channel']),
                'message': json.loads(notif['message']),
            })
        return result


#----------------------------------------------------------
# Dispatcher
#----------------------------------------------------------
class ImDispatch(object):
    def __init__(self):
        self.channels = {}
        self.started = False
        self.Event = None

    def poll(self, dbname, channels, last, options=None, timeout=TIMEOUT):
        if options is None:
            options = {}
        # Dont hang ctrl-c for a poll request, we need to bypass private
        # attribute access because we dont know before starting the thread that
        # it will handle a longpolling request
        if not odoo.evented:
            current = threading.current_thread()
            current._daemonic = True
            # rename the thread to avoid tests waiting for a longpolling
            current.setName("openerp.longpolling.request.%s" % current.ident)

        registry = odoo.registry(dbname)

        # immediatly returns if past notifications exist
        with registry.cursor() as cr:
            env = api.Environment(cr, SUPERUSER_ID, {})
            notifications = env['bus.bus'].poll(channels, last, options)

        # immediatly returns in peek mode
        if options.get('peek'):
            return dict(notifications=notifications, channels=channels)

        # or wait for future ones
        if not notifications:
            if not self.started:
                # Lazy start of events listener
                self.start()

            event = self.Event()
            for channel in channels:
                self.channels.setdefault(hashable(channel), set()).add(event)
            try:
                event.wait(timeout=timeout)
                with registry.cursor() as cr:
                    env = api.Environment(cr, SUPERUSER_ID, {})
                    notifications = env['bus.bus'].poll(channels, last, options)
            except Exception:
                # timeout
                pass
            finally:
                # gc pointers to event
                for channel in channels:
                    channel_events = self.channels.get(hashable(channel))
                    if channel_events and event in channel_events:
                        channel_events.remove(event)
        return notifications

    def loop(self):
        """ Dispatch postgres notifications to the relevant polling threads/greenlets """
        _logger.info("Bus.loop listen imbus on db postgres")
        with odoo.sql_db.db_connect('postgres').cursor() as cr:
            conn = cr._cnx
            cr.execute("listen imbus")
            cr.commit();
            while True:
                if select.select([conn], [], [], TIMEOUT) == ([], [], []):
                    pass
                else:
                    conn.poll()
                    channels = []
                    while conn.notifies:
                        channels.extend(json.loads(conn.notifies.pop().payload))
                    # dispatch to local threads/greenlets
                    events = set()
                    for channel in channels:
                        events.update(self.channels.pop(hashable(channel), set()))
                    for event in events:
                        event.set()

    def run(self):
        while True:
            try:
                self.loop()
            except Exception as e:
                _logger.exception("Bus.loop error, sleep and retry")
                time.sleep(TIMEOUT)

    def start(self):
        if odoo.evented:
            # gevent mode
            import gevent.event  # pylint: disable=import-outside-toplevel
            self.Event = gevent.event.Event
            gevent.spawn(self.run)
        else:
            # threaded mode
            self.Event = threading.Event
            t = threading.Thread(name="%s.Bus" % __name__, target=self.run)
            t.daemon = True
            t.start()
        self.started = True
        return self

# Partially undo a2ed3d3d5bdb6025a1ba14ad557a115a86413e65
# IMDispatch has a lazy start, so we could initialize it anyway
# And this avoids the Bus unavailable error messages
dispatch = ImDispatch()

```

## File: models\bus_presence.py

```python
# -*- coding: utf-8 -*-
import datetime
import time

from psycopg2 import OperationalError

from odoo import api, fields, models
from odoo import tools
from odoo.addons.bus.models.bus import TIMEOUT
from odoo.service.model import PG_CONCURRENCY_ERRORS_TO_RETRY
from odoo.tools.misc import DEFAULT_SERVER_DATETIME_FORMAT

DISCONNECTION_TIMER = TIMEOUT + 5
AWAY_TIMER = 1800  # 30 minutes


class BusPresence(models.Model):
    """ User Presence
        Its status is 'online', 'away' or 'offline'. This model should be a one2one, but is not
        attached to res_users to avoid database concurrence errors. Since the 'update' method is executed
        at each poll, if the user have multiple opened tabs, concurrence errors can happend, but are 'muted-logged'.
    """

    _name = 'bus.presence'
    _description = 'User Presence'
    _log_access = False

    _sql_constraints = [('bus_user_presence_unique', 'unique(user_id)', 'A user can only have one IM status.')]

    user_id = fields.Many2one('res.users', 'Users', required=True, index=True, ondelete='cascade')
    last_poll = fields.Datetime('Last Poll', default=lambda self: fields.Datetime.now())
    last_presence = fields.Datetime('Last Presence', default=lambda self: fields.Datetime.now())
    status = fields.Selection([('online', 'Online'), ('away', 'Away'), ('offline', 'Offline')], 'IM Status', default='offline')

    @api.model
    def update(self, inactivity_period):
        """ Updates the last_poll and last_presence of the current user
            :param inactivity_period: duration in milliseconds
        """
        # This method is called in method _poll() and cursor is closed right
        # after; see bus/controllers/main.py.
        try:
            self._update(inactivity_period)
            # commit on success
            self.env.cr.commit()
        except OperationalError as e:
            if e.pgcode in PG_CONCURRENCY_ERRORS_TO_RETRY:
                # ignore concurrency error
                return self.env.cr.rollback()
            raise

    @api.model
    def _update(self, inactivity_period):
        presence = self.search([('user_id', '=', self._uid)], limit=1)
        # compute last_presence timestamp
        last_presence = datetime.datetime.now() - datetime.timedelta(milliseconds=inactivity_period)
        values = {
            'last_poll': time.strftime(DEFAULT_SERVER_DATETIME_FORMAT),
        }
        # update the presence or a create a new one
        if not presence:  # create a new presence for the user
            values['user_id'] = self._uid
            values['last_presence'] = last_presence
            self.create(values)
        else:  # update the last_presence if necessary, and write values
            if presence.last_presence < last_presence:
                values['last_presence'] = last_presence
            # Hide transaction serialization errors, which can be ignored, the presence update is not essential
            with tools.mute_logger('odoo.sql_db'):
                presence.write(values)
                presence.flush()

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models
from odoo.addons.bus.models.bus_presence import AWAY_TIMER
from odoo.addons.bus.models.bus_presence import DISCONNECTION_TIMER


class ResPartner(models.Model):
    _inherit = 'res.partner'

    im_status = fields.Char('IM Status', compute='_compute_im_status')

    def _compute_im_status(self):
        self.env.cr.execute("""
            SELECT
                U.partner_id as id,
                CASE WHEN max(B.last_poll) IS NULL THEN 'offline'
                    WHEN age(now() AT TIME ZONE 'UTC', max(B.last_poll)) > interval %s THEN 'offline'
                    WHEN age(now() AT TIME ZONE 'UTC', max(B.last_presence)) > interval %s THEN 'away'
                    ELSE 'online'
                END as status
            FROM bus_presence B
            RIGHT JOIN res_users U ON B.user_id = U.id
            WHERE U.partner_id IN %s AND U.active = 't'
         GROUP BY U.partner_id
        """, ("%s seconds" % DISCONNECTION_TIMER, "%s seconds" % AWAY_TIMER, tuple(self.ids)))
        res = dict(((status['id'], status['status']) for status in self.env.cr.dictfetchall()))
        for partner in self:
            partner.im_status = res.get(partner.id, 'im_partner')  # if not found, it is a partner, useful to avoid to refresh status in js

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models
from odoo.addons.bus.models.bus_presence import AWAY_TIMER
from odoo.addons.bus.models.bus_presence import DISCONNECTION_TIMER


class ResUsers(models.Model):

    _inherit = "res.users"

    im_status = fields.Char('IM Status', compute='_compute_im_status')

    def _compute_im_status(self):
        """ Compute the im_status of the users """
        self.env.cr.execute("""
            SELECT
                user_id as id,
                CASE WHEN age(now() AT TIME ZONE 'UTC', last_poll) > interval %s THEN 'offline'
                     WHEN age(now() AT TIME ZONE 'UTC', last_presence) > interval %s THEN 'away'
                     ELSE 'online'
                END as status
            FROM bus_presence
            WHERE user_id IN %s
        """, ("%s seconds" % DISCONNECTION_TIMER, "%s seconds" % AWAY_TIMER, tuple(self.ids)))
        res = dict(((status['id'], status['status']) for status in self.env.cr.dictfetchall()))
        for user in self:
            user.im_status = res.get(user.id, 'offline')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
from . import bus
from . import bus_presence
from . import res_users
from . import res_partner

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_bus_bus,bus.bus public,model_bus_bus,,0,0,0,0
access_bus_presence,bus.presence,model_bus_presence,base.group_user,1,1,1,1
access_bus_presence_portal,bus.presence,model_bus_presence,base.group_portal,1,1,1,1

```

## File: static\src\js\crosstab_bus.js

```javascript
odoo.define('bus.CrossTab', function (require) {
"use strict";

var Longpolling = require('bus.Longpolling');

var session = require('web.session');

/**
 * CrossTab
 *
 * This is an extension of the longpolling bus with browser cross-tab synchronization.
 * It uses a Master/Slaves with Leader Election architecture:
 * - a single tab handles longpolling.
 * - tabs are synchronized by means of the local storage.
 *
 * localStorage used keys are:
 * - {LOCAL_STORAGE_PREFIX}.{sanitizedOrigin}.channels : shared public channel list to listen during the poll
 * - {LOCAL_STORAGE_PREFIX}.{sanitizedOrigin}.options : shared options
 * - {LOCAL_STORAGE_PREFIX}.{sanitizedOrigin}.notification : the received notifications from the last poll
 * - {LOCAL_STORAGE_PREFIX}.{sanitizedOrigin}.tab_list : list of opened tab ids
 * - {LOCAL_STORAGE_PREFIX}.{sanitizedOrigin}.tab_master : generated id of the master tab
 *
 * trigger:
 * - window_focus : when the window is focused
 * - notification : when a notification is receive from the long polling
 * - become_master : when this tab became the master
 * - no_longer_master : when this tab is not longer the master (the user swith tab)
 */
var CrossTabBus = Longpolling.extend({
    // constants
    TAB_HEARTBEAT_PERIOD: 10000, // 10 seconds
    MASTER_TAB_HEARTBEAT_PERIOD: 1500, // 1.5 seconds
    HEARTBEAT_OUT_OF_DATE_PERIOD: 5000, // 5 seconds
    HEARTBEAT_KILL_OLD_PERIOD: 15000, // 15 seconds
    LOCAL_STORAGE_PREFIX: 'bus',

    // properties
    _isMasterTab: false,
    _isRegistered: false,

    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        var now = new Date().getTime();
        // used to prefix localStorage keys
        this._sanitizedOrigin = session.origin.replace(/:\/{0,2}/g, '_');
        // prevents collisions between different tabs and in tests
        this._id = _.uniqueId(this.LOCAL_STORAGE_PREFIX) + ':' + now;
        if (this._callLocalStorage('getItem', 'last_ts', 0) + 50000 < now) {
            this._callLocalStorage('removeItem', 'last');
        }
        this._lastNotificationID = this._callLocalStorage('getItem', 'last', 0);
        this.call('local_storage', 'onStorage', this, this._onStorage);
    },
    destroy: function () {
        this._super();
        clearTimeout(this._heartbeatTimeout);
    },
    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------
    /**
     * Share the bus channels with the others tab by the local storage
     *
     * @override
     */
    addChannel: function () {
        this._super.apply(this, arguments);
        this._callLocalStorage('setItem', 'channels', this._channels);
    },
    /**
     * Share the bus channels with the others tab by the local storage
     *
     * @override
     */
    deleteChannel: function () {
        this._super.apply(this, arguments);
        this._callLocalStorage('setItem', 'channels', this._channels);
    },
    /**
     * @return {string}
     */
    getTabId: function () {
        return this._id;
    },
    /**
     * Tells whether this bus is related to the master tab.
     *
     * @returns {boolean}
     */
    isMasterTab: function () {
        return this._isMasterTab;
    },
    /**
     * Use the local storage to share the long polling from the master tab.
     *
     * @override
     */
    startPolling: function () {
        if (this._isActive === null) {
            this._heartbeat = this._heartbeat.bind(this);
        }
        if (!this._isRegistered) {
            this._isRegistered = true;

            var peers = this._callLocalStorage('getItem', 'peers', {});
            peers[this._id] = new Date().getTime();
            this._callLocalStorage('setItem', 'peers', peers);

            this._registerWindowUnload();

            if (!this._callLocalStorage('getItem', 'master')) {
                this._startElection();
            }

            this._heartbeat();

            if (this._isMasterTab) {
                this._callLocalStorage('setItem', 'channels', this._channels);
                this._callLocalStorage('setItem', 'options', this._options);
            } else {
                this._channels = this._callLocalStorage('getItem', 'channels', this._channels);
                this._options = this._callLocalStorage('getItem', 'options', this._options);
            }
            return;  // startPolling will be called again on tab registration
        }

        if (this._isMasterTab) {
            this._super.apply(this, arguments);
        }
    },
    /**
     * Share the option with the local storage
     *
     * @override
     */
    updateOption: function () {
        this._super.apply(this, arguments);
        this._callLocalStorage('setItem', 'options', this._options);
    },
    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------
    /**
     * Call local_storage service
     *
     * @private
     * @param {string} method (getItem, setItem, removeItem, on)
     * @param {string} key
     * @param {any} param
     * @returns service information
     */
    _callLocalStorage: function (method, key, param) {
        return this.call('local_storage', method, this._generateKey(key), param);
    },
    /**
     * Generates localStorage keys prefixed by bus. (LOCAL_STORAGE_PREFIX = the name
     * of this addon), and the sanitized origin, to prevent keys from
     * conflicting when several bus instances (polling different origins)
     * co-exist.
     *
     * @private
     * @param {string} key
     * @returns key prefixed with the origin
     */
    _generateKey: function (key) {
        return this.LOCAL_STORAGE_PREFIX + '.' + this._sanitizedOrigin + '.' + key;
    },
    /**
     * @override
     * @returns {integer} number of milliseconds since 1 January 1970 00:00:00
     */
    _getLastPresence: function () {
        return this._callLocalStorage('getItem', 'lastPresence') || this._super();
    },
    /**
     * Check all the time (according to the constants) if the tab is the master tab and
     * check if it is active. Use the local storage for this checks.
     *
     * @private
     * @see _startElection method
     */
    _heartbeat: function () {
        var now = new Date().getTime();
        var heartbeatValue = parseInt(this._callLocalStorage('getItem', 'heartbeat', 0));
        var peers = this._callLocalStorage('getItem', 'peers', {});

        if ((heartbeatValue + this.HEARTBEAT_OUT_OF_DATE_PERIOD) < now) {
            // Heartbeat is out of date. Electing new master
            this._startElection();
            heartbeatValue = parseInt(this._callLocalStorage('getItem', 'heartbeat', 0));
        }

        if (this._isMasterTab) {
            //walk through all peers and kill old
            var cleanedPeers = {};
            for (var peerName in peers) {
                if (peers[peerName] + this.HEARTBEAT_KILL_OLD_PERIOD > now) {
                    cleanedPeers[peerName] = peers[peerName];
                }
            }

            if (heartbeatValue !== this.lastHeartbeat) {
                // someone else is also master...
                // it should not happen, except in some race condition situation.
                this._isMasterTab = false;
                this.lastHeartbeat = 0;
                peers[this._id] = now;
                this._callLocalStorage('setItem', 'peers', peers);
                this.stopPolling();
                this.trigger('no_longer_master');
            } else {
                this.lastHeartbeat = now;
                this._callLocalStorage('setItem', 'heartbeat', now);
                this._callLocalStorage('setItem', 'peers', cleanedPeers);
            }
        } else {
            //update own heartbeat
            peers[this._id] = now;
            this._callLocalStorage('setItem', 'peers', peers);
        }

        // Write lastPresence in local storage if it has been updated since last heartbeat
        var hbPeriod = this._isMasterTab ? this.MASTER_TAB_HEARTBEAT_PERIOD : this.TAB_HEARTBEAT_PERIOD;
        if (this._lastPresenceTime + hbPeriod > now) {
            this._callLocalStorage('setItem', 'lastPresence', this._lastPresenceTime);
        }

        this._heartbeatTimeout = setTimeout(this._heartbeat.bind(this), hbPeriod);
    },
    /**
     * @private
     */
    _registerWindowUnload: function () {
        $(window).on('unload.' + this._id, this._onUnload.bind(this));
    },
    /**
     * Check with the local storage if the current tab is the master tab.
     * If this tab became the master, trigger 'become_master' event
     *
     * @private
     */
    _startElection: function () {
        if (this._isMasterTab) {
            return;
        }
        //check who's next
        var now = new Date().getTime();
        var peers = this._callLocalStorage('getItem', 'peers', {});
        var heartbeatKillOld = now - this.HEARTBEAT_KILL_OLD_PERIOD;
        var newMaster;
        for (var peerName in peers) {
            //check for dead peers
            if (peers[peerName] < heartbeatKillOld) {
                continue;
            }
            newMaster = peerName;
            break;
        }

        if (newMaster === this._id) {
            //we're next in queue. Electing as master
            this.lastHeartbeat = now;
            this._callLocalStorage('setItem', 'heartbeat', this.lastHeartbeat);
            this._callLocalStorage('setItem', 'master', true);
            this._isMasterTab = true;
            this.startPolling();
            this.trigger('become_master');

            //removing master peer from queue
            delete peers[newMaster];
            this._callLocalStorage('setItem', 'peers', peers);
        }
    },
    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------
    /**
     * @override
     */
    _onFocusChange: function (params) {
        this._super.apply(this, arguments);
        this._callLocalStorage('setItem', 'focus', params.focus);
    },
    /**
     * If it's the master tab, the notifications ares broadcasted to other tabs by the
     * local storage.
     *
     * @override
     */
    _onPoll: function (notifications) {
        var notifs = this._super(notifications);
        if (this._isMasterTab && notifs.length) {
            this._callLocalStorage('setItem', 'last', this._lastNotificationID);
            this._callLocalStorage('setItem', 'last_ts', new Date().getTime());
            this._callLocalStorage('setItem', 'notification', notifs);
        }
    },
    /**
     * Handler when the local storage is updated
     *
     * @private
     * @param {OdooEvent} event
     * @param {string} event.key
     * @param {string} event.newValue
     */
    _onStorage: function (e) {
        var value = JSON.parse(e.newValue);
        var key = e.key;

        if (this._isRegistered && key === this._generateKey('master') && !value) {
            //master was unloaded
            this._startElection();
        }

        // last notification id changed
        if (key === this._generateKey('last')) {
            this._lastNotificationID = value || 0;
        }
        // notifications changed
        else if (key === this._generateKey('notification')) {
            if (!this._isMasterTab) {
                this.trigger("notification", value);
            }
        }
        // update channels
        else if (key === this._generateKey('channels')) {
            this._channels = value;
        }
        // update options
        else if (key === this._generateKey('options')) {
            this._options = value;
        }
        // update focus
        else if (key === this._generateKey('focus')) {
            this._isOdooFocused = value;
            this.trigger('window_focus', this._isOdooFocused);
        }
    },
    /**
     * Handler when unload the window
     *
     * @private
     */
    _onUnload: function () {
        // unload peer
        var peers = this._callLocalStorage('getItem', 'peers') || {};
        delete peers[this._id];
        this._callLocalStorage('setItem', 'peers', peers);

        // unload master
        if (this._isMasterTab) {
            this._callLocalStorage('removeItem', 'master');
        }
    },
});

return CrossTabBus;

});


```

## File: static\src\js\longpolling_bus.js

```javascript
odoo.define('bus.Longpolling', function (require) {
"use strict";

var Bus = require('web.Bus');
var ServicesMixin = require('web.ServicesMixin');


/**
 * Event Longpolling bus used to bind events on the server long polling return
 *
 * trigger:
 * - window_focus : when the window focus change (true for focused, false for blur)
 * - notification : when a notification is receive from the long polling
 *
 * @class Longpolling
 */
var LongpollingBus = Bus.extend(ServicesMixin, {
    // constants
    PARTNERS_PRESENCE_CHECK_PERIOD: 30000,  // don't check presence more than once every 30s
    ERROR_RETRY_DELAY: 10000, // 10 seconds
    POLL_ROUTE: '/longpolling/poll',

    // properties
    _isActive: null,
    _lastNotificationID: 0,
    _isOdooFocused: true,
    _pollRetryTimeout: null,

    /**
     * @override
     */
    init: function (parent, params) {
        this._super.apply(this, arguments);
        this._id = _.uniqueId('bus');

        // the _id is modified by crosstab_bus, so we can't use it to unbind the events in the destroy.
        this._longPollingBusId = this._id;
        this._options = {};
        this._channels = [];

        // bus presence
        this._lastPresenceTime = new Date().getTime();
        $(window).on("focus." + this._longPollingBusId, this._onFocusChange.bind(this, {focus: true}));
        $(window).on("blur." + this._longPollingBusId, this._onFocusChange.bind(this, {focus: false}));
        $(window).on("unload." + this._longPollingBusId, this._onFocusChange.bind(this, {focus: false}));

        $(window).on("click." + this._longPollingBusId, this._onPresence.bind(this));
        $(window).on("keydown." + this._longPollingBusId, this._onPresence.bind(this));
        $(window).on("keyup." + this._longPollingBusId, this._onPresence.bind(this));
    },
    /**
     * @override
     */
    destroy: function () {
        this.stopPolling();
        $(window).off("focus." + this._longPollingBusId);
        $(window).off("blur." + this._longPollingBusId);
        $(window).off("unload." + this._longPollingBusId);
        $(window).off("click." + this._longPollingBusId);
        $(window).off("keydown." + this._longPollingBusId);
        $(window).off("keyup." + this._longPollingBusId);
        this._super();
    },
    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------
    /**
     * Register a new channel to listen on the longpoll (ignore if already
     * listening on this channel).
     * Aborts a pending longpoll, in order to re-start another longpoll, so
     * that we can immediately get notifications on newly registered channel.
     *
     * @param {string} channel
     */
    addChannel: function (channel) {
        if (this._channels.indexOf(channel) === -1) {
            this._channels.push(channel);
            if (this._pollRpc) {
                this._pollRpc.abort();
            } else {
                this.startPolling();
            }
        }
    },
    /**
     * Unregister a channel from listening on the longpoll.
     *
     * Aborts a pending longpoll, in order to re-start another longpoll, so
     * that we immediately remove ourselves from listening on notifications
     * on this channel.
     *
     * @param {string} channel
     */
    deleteChannel: function (channel) {
        var index = this._channels.indexOf(channel);
        if (index !== -1) {
            this._channels.splice(index, 1);
            if (this._pollRpc) {
                this._pollRpc.abort();
            }
        }
    },
    /**
     * Tell whether odoo is focused or not
     *
     * @returns {boolean}
     */
    isOdooFocused: function () {
        return this._isOdooFocused;
    },
    /**
     * Start a long polling, i.e. it continually opens a long poll
     * connection as long as it is not stopped (@see `stopPolling`)
     */
    startPolling: function () {
        if (this._isActive === null) {
            this._poll = this._poll.bind(this);
        }
        if (!this._isActive) {
            this._isActive = true;
            this._poll();
        }
    },
    /**
     * Stops any started long polling
     *
     * Aborts a pending longpoll so that we immediately remove ourselves
     * from listening on notifications on this channel.
     */
    stopPolling: function () {
        this._isActive = false;
        this._channels = [];
        clearTimeout(this._pollRetryTimeout);
        if (this._pollRpc) {
            this._pollRpc.abort();
        }
    },
    /**
     * Add or update an option on the longpoll bus.
     * Stored options are sent to the server whenever a poll is started.
     *
     * @param {string} key
     * @param {any} value
     */
    updateOption: function (key, value) {
        this._options[key] = value;
    },
    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------
    /**
     * returns the last recorded presence
     *
     * @private
     * @returns {integer} number of milliseconds since 1 January 1970 00:00:00
     */
    _getLastPresence: function () {
        return this._lastPresenceTime;
    },
    /**
     * Continually start a poll:
     *
     * A poll is a connection that is kept open for a relatively long period
     * (up to 1 minute). Local bus data are sent to the server each time a poll
     * is initiated, and the server may return some "real-time" notifications
     * about registered channels.
     *
     * A poll ends on timeout, on abort, on receiving some notifications, or on
     * receiving an error. Another poll usually starts afterward, except if the
     * poll is aborted or stopped (@see stopPolling).
     *
     * @private
     */
    _poll: function () {
        var self = this;
        if (!this._isActive) {
            return;
        }
        var now = new Date().getTime();
        var options = _.extend({}, this._options, {
            bus_inactivity: now - this._getLastPresence(),
        });
        var data = {channels: this._channels, last: this._lastNotificationID, options: options};
        // The backend has a maximum cycle time of 50 seconds so give +10 seconds
        this._pollRpc = this._makePoll(data);
        this._pollRpc.then(function (result) {
            self._pollRpc = false;
            self._onPoll(result);
            self._poll();
        }).guardedCatch(function (result) {
            self._pollRpc = false;
            // no error popup if request is interrupted or fails for any reason
            result.event.preventDefault();
            if (result.message === "XmlHttpRequestError abort") {
                self._poll();
            } else {
                // random delay to avoid massive longpolling
                self._pollRetryTimeout = setTimeout(self._poll, self.ERROR_RETRY_DELAY + (Math.floor((Math.random()*20)+1)*1000));
            }
        });
    },

    /**
     * @private
     * @param data: object with poll parameters
     */
    _makePoll: function(data) {
        return this._rpc({route: this.POLL_ROUTE, params: data}, {shadow : true, timeout: 60000});
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------
    /**
     * Handler when the focus of the window change.
     * Trigger the 'window_focus' event.
     *
     * @private
     * @param {Object} params
     * @param {Boolean} params.focus
     */
    _onFocusChange: function (params) {
        this._isOdooFocused = params.focus;
        if (params.focus) {
            this._lastPresenceTime = new Date().getTime();
            this.trigger('window_focus', this._isOdooFocused);
        }
    },
    /**
     * Handler when the long polling receive the new notifications
     * Update the last notification id received.
     * Triggered the 'notification' event with a list [channel, message] from notifications.
     *
     * @private
     * @param {Object[]} notifications, Input notifications have an id, channel, message
     * @returns {Array[]} Output arrays have notification's channel and message
     */
    _onPoll: function (notifications) {
        var self = this;
        var notifs = _.map(notifications, function (notif) {
            if (notif.id > self._lastNotificationID) {
                self._lastNotificationID = notif.id;
            }
            return [notif.channel, notif.message];
        });
        this.trigger("notification", notifs);
        return notifs;
    },
    /**
     * Handler when they are an activity on the window (click, keydown, keyup)
     * Update the last presence date.
     *
     * @private
     */
    _onPresence: function () {
        this._lastPresenceTime = new Date().getTime();
    },
});

return LongpollingBus;

});

```

## File: static\src\js\web_client_bus.js

```javascript
odoo.define('bus.WebClient', function (require) {
    "use strict";

    const core = require('web.core');
    const WebClient = require('web.WebClient');

    const _t = core._t;

    WebClient.include({

        //----------------------------------------------------------------------
        // Public
        //----------------------------------------------------------------------

        /**
         * Detects the presence of assets in DOM's HEAD
         *
         * @override
         */
        async start() {
            this._assetsChangedNotificationId = null;
            this._assets = {};
            await this._super(...arguments);
        },
        /**
         * Assigns handler to bus notification
         *
         * @override
         */
        show_application() {
            const shown = this._super(...arguments);
            document.querySelectorAll('*[data-asset-xmlid]').forEach(el => {
                this._assets[el.getAttribute('data-asset-xmlid')] = el.getAttribute('data-asset-version');
            });
            this.call('bus_service', 'onNotification', this, this._onNotification);
            this.call('bus_service', 'addChannel', 'bundle_changed');
            return shown;
        },

        //----------------------------------------------------------------------
        // Private
        //----------------------------------------------------------------------

        /**
         * Displays one notification on user's screen when assets have changed
         *
         * @private
         */
        _displayBundleChangedNotification() {
            if (!this._assetsChangedNotificationId) {
                // Wrap the notification inside a delay.
                // The server may be overwhelmed with recomputing assets
                // We wait until things settle down
                clearTimeout(this._bundleNotifTimerID);
                this._bundleNotifTimerID = setTimeout(() => {
                    this._assetsChangedNotificationId = this.call('notification', 'notify', {
                        title: _t('Refresh'),
                        message: _t('The page appears to be out of date.'),
                        sticky: true,
                        onClose: () => {
                            this._assetsChangedNotificationId = null;
                        },
                        buttons: [{
                            text: _t('Refresh'),
                            primary: true,
                            click: () => {
                                window.location.reload(true);
                            }
                        }],
                    });
                }, this._getBundleNotificationDelay());
            }
        },
        /**
         * Computes a random delay to avoid hammering the server
         * when bundles change with all the users reloading
         * at the same time
         *
         * @private
         * @return {number} delay in milliseconds
         */
        _getBundleNotificationDelay() {
            return 10000 + Math.floor(Math.random()*50) * 1000;
        },

        //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------

        /**
         * Reacts to bus's notification
         *
         * @private
         * @param {Array} notifications: list of received notifications
         */
        _onNotification(notifications) {
            for (const notif of notifications) {
                if (notif[0][1] === 'bundle_changed') {
                    const bundleXmlId = notif[1][0];
                    const bundleVersion = notif[1][1];
                    if (bundleXmlId in this._assets && bundleVersion !== this._assets[bundleXmlId]) {
                        this._displayBundleChangedNotification();
                        break;
                    }
                }
            }
        }
    });
});

```

## File: static\src\js\services\bus_service.js

```javascript
odoo.define('bus.BusService', function (require) {
"use strict";

var CrossTab = require('bus.CrossTab');
var core = require('web.core');
var ServicesMixin = require('web.ServicesMixin');
const session = require('web.session');

var BusService =  CrossTab.extend(ServicesMixin, {
    dependencies : ['local_storage'],

    // properties
    _audio: null,

    /**
     * As the BusService doesn't extend AbstractService, we have to replicate
     * here what is done in AbstractService
     *
     * @param {Object} env
     */
    init: function (env) {
        this.env = env;
        this._super();
    },

    /**
     * Replicate the behavior of AbstractService:
     *
     * Directly calls the requested service, instead of triggering a
     * 'call_service' event up, which wouldn't work as services have no parent.
     *
     * @param {OdooEvent} ev
     */
    _trigger_up: function (ev) {
        if (ev.name === 'call_service') {
            const payload = ev.data;
            let args = payload.args || [];
            if (payload.service === 'ajax' && payload.method === 'rpc') {
                // ajax service uses an extra 'target' argument for rpc
                args = args.concat(ev.target);
            }
            const service = this.env.services[payload.service];
            const result = service[payload.method].apply(service, args);
            payload.callback(result);
        }
    },
    /**
     * This method is necessary in order for this Class to be used to instantiate services
     *
     * @abstract
     */
    start: function () {},

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Send a notification, and notify once per browser's tab
     *
     * @param {string} title
     * @param {string} content
     * @param {function} [callback] if given callback will be called when user clicks on notification
     */
    sendNotification: function (title, content, callback) {
        if (window.Notification && Notification.permission === "granted") {
            if (this.isMasterTab()) {
                try {
                    this._sendNativeNotification(title, content, callback);
                } catch (error) {
                    // Notification without Serviceworker in Chrome Android doesn't works anymore
                    // So we fallback to do_notify() in this case
                    // https://bugs.chromium.org/p/chromium/issues/detail?id=481856
                    if (error.message.indexOf('ServiceWorkerRegistration') > -1) {
                        this.do_notify(title, content);
                        this._beep();
                    } else {
                        throw error;
                    }
                }
            }
        } else {
            this.do_notify(title, content);
            if (this.isMasterTab()) {
                this._beep();
            }
        }
    },
    /**
     * Register listeners on notifications received on this bus service
     *
     * @param {Object} receiver
     * @param {function} func
     */
    onNotification: function () {
        this.on.apply(this, ["notification"].concat(Array.prototype.slice.call(arguments)));
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Lazily play the 'beep' audio on sent notification
     *
     * @private
     */
    _beep: function () {
        if (typeof(Audio) !== "undefined") {
            if (!this._audio) {
                this._audio = new Audio();
                var ext = this._audio.canPlayType("audio/ogg; codecs=vorbis") ? ".ogg" : ".mp3";
                this._audio.src = session.url("/mail/static/src/audio/ting" + ext);
            }
            Promise.resolve(this._audio.play()).catch(_.noop);
        }
    },
    /**
     * Show a browser notification
     *
     * @private
     * @param {string} title
     * @param {string} content
     * @param {function} [callback] if given callback will be called when user clicks on notification
     */
    _sendNativeNotification: function (title, content, callback) {
        var notification = new Notification(
            // The native Notification API works with plain text and not HTML
            // unescaping is safe because done only at the **last** step
            _.unescape(title),
            {
                body: _.unescape(content),
                icon: "/mail/static/src/img/odoobot_transparent.png"
            });
        notification.onclick = function () {
            window.focus();
            if (this.cancel) {
                this.cancel();
            } else if (this.close) {
                this.close();
            }
            if (callback) {
                callback();
            }
        };
    },

});

core.serviceRegistry.add('bus_service', BusService);

return BusService;

});

```

## File: views\bus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="im assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/bus/static/src/js/longpolling_bus.js"></script>
            <script type="text/javascript" src="/bus/static/src/js/crosstab_bus.js"></script>
            <script type="text/javascript" src="/bus/static/src/js/services/bus_service.js"></script>
            <script type="text/javascript" src="/bus/static/src/js/web_client_bus.js"></script>
        </xpath>
    </template>

    <template id="assets_frontend" name="im assets" inherit_id="web.assets_frontend">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/bus/static/src/js/longpolling_bus.js"></script>
            <script type="text/javascript" src="/bus/static/src/js/crosstab_bus.js"></script>
            <script type="text/javascript" src="/bus/static/src/js/services/bus_service.js"></script>
        </xpath>
    </template>

    <template id="qunit_suite" name="bus tests" inherit_id="web.qunit_suite_tests">
        <xpath expr="//script[last()]" position="after">
            <script type="text/javascript" src="/bus/static/tests/bus_tests.js"/>
        </xpath>
    </template>
    <template id="assets_tests" name="Bus Assets Tests" inherit_id="web.assets_tests">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/bus/static/tests/bus_tests_tour.js"></script>
        </xpath>
    </template>
</odoo>

```

