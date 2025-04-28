# Odoo Module: im_support

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': 'Livechat Support',
    'version': '1.0',
    'category': 'Hidden',
    'description': """
        This module allows employee users to communicate with livechat operators
        of another database on which a counterpart addon is installed.
    """,
    'author': 'Odoo SA',
    'depends': [
        'mail',
    ],
    'data': [
        'views/assets.xml',
    ],
    'qweb': [
        "static/src/xml/discuss.xml",
        "static/src/xml/systray.xml",
    ],
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


class ImSupport(http.Controller):

    @http.route('/im_support/tests', type='http', auth="user")
    def test_suite(self, mod=None, **kwargs):
        return request.render('im_support.support_qunit_suite')

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import hmac
from hashlib import sha256

from odoo import models
from odoo.http import request


class Http(models.AbstractModel):
    _inherit = 'ir.http'

    def session_info(self):
        result = super(Http, self).session_info()

        if self.env.user.has_group('base.group_user'):
            icp = request.env['ir.config_parameter'].sudo()
            db_uuid = icp.get_param('database.uuid')
            db_secret = icp.get_param('database.secret')
            message = db_uuid + str(request.uid)
            token = hmac.new(message.encode('utf-8'), db_secret.encode('utf-8'), sha256).hexdigest()

            result['db_uuid'] = db_uuid
            result['support_token'] = token
            result['support_origin'] = False  # must be overridden to specify the correct origin

        return result

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_http

```

## File: static\src\js\discuss.js

```javascript
odoo.define('im_support.Discuss', function (require) {
"use strict";

var Discuss = require('mail.Discuss');

/**
 * This module includes Discuss to handle the case of the Support channel.
 */
Discuss.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Overrides to hide stars in the Support channel, as this feature does not
     * work in that channel.
     *
     * @override
     * @private
     */
    _getThreadRenderingOptions: function () {
        var options = this._super.apply(this, arguments);
        if (this._thread.getType() === 'support_channel') {
            options.displayStars = false;
        }
        return options;
    },
    /**
     * Overrides to hide the attachment button in the composer of the Support
     * channel, as this feature does not work in that channel. Also hides the
     * composer when no operator is available.
     *
     * @override
     * @private
     */
    _setThread: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            var $buttonAddAttachment = self._basicComposer.$('.o_composer_button_add_attachment');
            if (self._thread.getType() === 'support_channel') {
                if (!self._thread.isAvailable()) {
                    self._basicComposer.do_hide();
                }
                $buttonAddAttachment.toggleClass('o_hidden', true);
            } else {
                $buttonAddAttachment.toggleClass('o_hidden', false);
            }
        });
    },
    /**
     * Overrides to handle the Support channel case (hide all buttons).
     *
     * @override
     * @private
     */
    _updateControlPanelButtons: function (thread) {
        this._super.apply(this, arguments);
        if (thread.getType() === 'support_channel') {
            this.$buttons.find('button').hide();
        }
    },
});

});

```

## File: static\src\js\mail_manager.js

```javascript
odoo.define('im_support.MailManager', function (require) {
"use strict";

var MailManager = require('mail.Manager');

var core = require('web.core');
var session = require('web.session');
var WebClient = require('web.WebClient');

var SupportChannel = require('im_support.SupportChannel');
var SupportMessage = require('im_support.SupportMessage');
var supportSession = require('im_support.SupportSession');

var _t = core._t;

var POLL_TIMEOUT_DELAY = 1000 * 60 * 30; // 30 minutes
var POLL_TIMEOUT_KEY = 'im_support.poll_timeout';
var SUPPORT_CHANNEL_STATE_KEY = 'im_support.channel_state';

/**
 * This module includes the MailService to handle the case of the Support
 * channel, allowing the users of the current database to communicate with
 * livechat operators from another database (the Support database).
 */
MailManager.include({
    dependencies: (MailManager.prototype.dependencies || []).concat(['support_bus_service']),
    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Initialises the Support: checks if there is a pending chat session
     * between the user and Support, and if so, re-opens it.
     * Note: we can't directly override init(), because it is already called
     * when the include is applied, so we use this hook instead (called at
     * webclient startup)
     */
    initSupport: function () {
        var self = this;
        this.supportChannelDef = null;
        this.supportChannelUUID = null;
        this.pollTimeout = null;
        this.pollingSupport = false;

        // listen to notifications coming from Support longpolling
        this.call('support_bus_service', 'onNotification', this, this._onSupportNotification);

        // check if there is a pending chat session with the Support
        var timeoutTimestamp = this.call('local_storage', 'getItem', POLL_TIMEOUT_KEY);
        var pollingDelay = timeoutTimestamp && (JSON.parse(timeoutTimestamp) - Date.now());
        if (pollingDelay && pollingDelay > 0) {
            var channelState = this.call('local_storage', 'getItem', SUPPORT_CHANNEL_STATE_KEY);
            this.startSupportLivechat(channelState).then(function () {
                var supportChannel = self.getChannel(self.supportChannelUUID);
                if (supportChannel.isAvailable()) {
                    self.startPollingSupport(pollingDelay);
                }
            });
        }
    },
    /**
     * Overrides to filter out the Support channel from the previews.
     *
     * @override
     * @returns {Promise<Object[]>} list of valid objects for mail.Preview
     *   template
     */
    getChannelPreviews: function () {
        var self = this;
        return this._super.apply(this, arguments)
            .then(function (channelsPreview) {
                return _.reject(channelsPreview, { id: self.supportChannelUUID });
            });
    },
    /**
     * Initiates a longpoll with the server hosting the Support channel.
     *
     * @param {integer} [pollingDelay=POLL_TIMEOUT_DELAY] the longpolling
     *   timeout delay to set
     */
    startPollingSupport: function (pollingDelay) {
        if (!('pollingSupport' in this)) {
            return this.initSupport();
        }
        if (!this.pollingSupport) {
            this.pollingSupport = true;
            this.call('support_bus_service', 'addChannel', this.supportChannelUUID);
            this.call('support_bus_service', 'startPolling');
            this._setPollTimeout(pollingDelay);
        }
    },
    /**
     * Opens the Support channel between a livechat operator from the Support
     * database and the current user (if there is an available operator).
     * Ensures to perform only once the request to create/retrieve the Support
     * channel.
     *
     * @param {string} [channelState='open'] state of the Support
     *   channel (see CHANNEL_STATES for accepted values)
     * @returns {Promise}
     */
    startSupportLivechat: function (channelState) {
        var self = this;
        if (!this.supportChannelDef) {
            // retrieve or create the channel
            this.supportChannelDef = supportSession.rpc('/odoo_im_support/get_support_channel', {
                channel_uuid: session.support_token,
                db_uuid: session.db_uuid,
                user_name: session.name,
            });
        }
        return this.supportChannelDef.then(function (channel) {
            if (!channel) {
                // there is no channel (because there is no online operator, and
                // no support channel has been created yet), so create one to
                // open in a chat window
                channel = {
                    available: false,
                    support_channel: true,
                    type: 'livechat',
                    uuid: "support_unavailable",
                };
            }
            if (!channelState) {
                channelState = self._isDiscussOpen() ? 'closed' : 'open';
                self.call('local_storage', 'setItem', SUPPORT_CHANNEL_STATE_KEY, channelState);
            }
            if (!self.supportChannelUUID) {
                // this part is only executed the first time the RPC is resolved
                self.supportChannelUUID = channel.uuid;

                // add the channel to the MailManager
                return self._addChannel(_.extend(channel, {
                    id: channel.uuid,
                    is_minimized: _.contains(['open', 'folded'], channelState),
                    state: channelState,
                })).then(function () {
                    // display an automatic message in the channel
                    var supportChannel = self.getChannel(channel.uuid);
                    supportChannel.addDefaultMessage();
                });
            } else {
                // the channel has already been added to the MailManager, so
                // simply re-open it
                if (self._isDiscussOpen()) {
                    self._openThreadInDiscuss(self.supportChannelUUID);
                } else {
                    channel = self.getChannel(self.supportChannelUUID);
                    channel.fold(channelState === 'folded');
                }
            }
        }).guardedCatch(function () {
            self.do_warn(_t("The Support server can't be reached."));
        });
    },
    /**
     * Updates the state of the Support channel (stored in the localStorage).
     *
     * @param {string} state ('closed', 'folded' or 'open')
     */
    updateSupportChannelState: function (state) {
        this.call('local_storage', 'setItem', SUPPORT_CHANNEL_STATE_KEY, state);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     * @private
     */
    _makeChannel: function (data, options) {
        if (data.id === this.supportChannelUUID) {
            return new SupportChannel({
                parent: this,
                data: data,
                options: options,
                commands: this._commands
            });
        }
        return this._super.apply(this, arguments);
    },
    /**
     * Overrides to instantiate a SupportMessage when necessary.
     * @override
     */
    _makeMessage: function (data) {
        if (
            this.supportChannelUUID &&
            _.contains(data.channel_ids, this.supportChannelUUID)
        ) {
            return new SupportMessage(this, data, this._emojis);
        }
        return this._super.apply(this, arguments);
    },
    /**
     * Automatically stop polling for Support messages after a given delay of
     * inactivity.
     *
     * @private
     * @param {integer} [pollingDelay=POLL_TIMEOUT_DELAY] the longpolling
     * timeout delay to set
     */
    _setPollTimeout: function (pollingDelay) {
        pollingDelay = pollingDelay || POLL_TIMEOUT_DELAY;
        clearTimeout(this.pollTimeout);
        this.pollTimeout = setTimeout(this._stopPollingSupport.bind(this), pollingDelay);
        // save the timeout expiration datetime into the LocalStorage so that
        // we can re-open the Support channel if necessary on F5
        var timeoutTimestamp = Date.now() + pollingDelay;
        this.call('local_storage', 'setItem', POLL_TIMEOUT_KEY, timeoutTimestamp);
    },
    /**
     * Stops the longpoll with the server hosting the Support channel.
     *
     * @private
     */
    _stopPollingSupport: function () {
        this.pollingSupport = false;
        this.call('support_bus_service', 'stopPolling');
        this.call('local_storage', 'removeItem', POLL_TIMEOUT_KEY);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Handles poll notifications from the Support server.
     *
     * @private
     * @param {Object[]} notifications
     */
    _onSupportNotification: function (notifications) {
        var self = this;
        if (notifications && notifications.length) {
            this._setPollTimeout();
        }
        _.each(notifications, function (notification) {
            if (notification[1]._type === 'history_command') {
                // ignore history requests
                return;
            }
            var messageData = _.extend(notification[1], {
                channel_ids: [self.supportChannelUUID],
            });
            self._handleChannelNotification({ data: messageData });
        });
    },
});


// Unfortunately, we can't override init() of MailService because it is called
// before the include is applied, so we override the WebClient instead to call
// an initialization hook for Livechat Support in the Mail service.
WebClient.include({
    /**
     * Overrides to ask the Mail service to check whether there is a
     * pending chat session with Support, and if so, to re-open it.
     *
     * @override
     */
    show_application: function () {
        this.call('mail_service', 'initSupport');
        return this._super.apply(this, arguments);
    },
});

});


```

## File: static\src\js\support_bus.js

```javascript
odoo.define('im_support.SupportBus', function (require) {
"use strict";

/**
 * This module instantiates and exports the instance of the Bus, parameterized
 * to poll the Support server.
 */

var BusService = require('bus.BusService');
var supportSession = require('im_support.SupportSession');
var core = require('web.core');

var SupportBusService =  BusService.extend({
    LOCAL_STORAGE_PREFIX: 'im_support',
    POLL_ROUTE: '/longpolling/support_poll',

    /**
     * @override _makePoll to force the remote session
     */
    _makePoll: function(data) {
        return supportSession.rpc(this.POLL_ROUTE, data, {shadow : true, timeout: 60000});
    },
});

core.serviceRegistry.add('support_bus_service', SupportBusService);

return SupportBusService;

});


```

## File: static\src\js\support_channel.js

```javascript
odoo.define('im_support.SupportChannel', function (require) {
"use strict";

var supportSession = require('im_support.SupportSession');

var SearchableThread = require('mail.model.SearchableThread');

var core = require('web.core');
var session = require('web.session');

var _t = core._t;

/**
 * This mail model represents support channel, which are communication channels
 * between two different databases for support-related reasons. It is like
 * livechat, but both users are in different databases, and both users
 * communicate from their respective 'backend' access.
 *
 * FIXME: it should inherit from mail.model.Channel, not from
 * mail.model.SearchableThread
 */
var SupportChannel = SearchableThread.extend({

    /**
     * @override
     * @param {Object} params
     * @param {Object} params.data
     * @param {boolean} params.data.available
     * @param {string|integer} params.data.id
     * @param {boolean} params.data.is_minimized
     * @param {Object} params.data.operator
     * @param {string} params.data.state ['open', 'closed', 'folded']
     * @param {string} params.data.uuid
     * @param {string} params.data.welcome_message
     * @param {Object} params.options
     */
    init: function (params) {
        var data = params.data;

        data.type = 'support_channel';
        data.name = _t("Support");

        this._available = data.available;
        this._operator = data.operator;
        this._supportChannelUUID = data.id;
        this._welcomeMessage = data.welcome_message;
        if (!this._available) {
            data.name += _t(" (offline)");
        }

        this._super.apply(this, arguments);

        // force stuff that should probably be in Thread (or at least
        // SearchableThread), but that are currently in Channel
        this._detached = data.is_minimized;
        this._folded = data.state === 'folded';
        this._uuid = data.uuid;
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Adds the default message in the Support channel (depending on its
     * availability).
     */
    addDefaultMessage: function () {
        if (!this._available) {
            this._addSupportNotAvailableMessage();
        } else {
            this._addSupportWelcomeMessage();
        }
    },
    /**
     * Overrides to store the state of the Support channel in the localStorage.
     *
     * @override
     */
    close: function () {
        this._super.apply(this, arguments);
        this.call('mail_service', 'updateSupportChannelState', 'closed');
    },
    /**
     * Overrides to store the state of the Support channel in the localStorage.
     *
     * @override
     */
    detach: function () {
        this._super.apply(this, arguments);
        this.call('mail_service', 'updateSupportChannelState', 'open');
    },
    /**
     * Overrides to store the state of the Support channel in the localStorage.
     *
     * @override
     */
    fold: function (folded) {
        this._super.apply(this, arguments);
        var value = folded ? 'folded' : 'open';
        this.call('mail_service', 'updateSupportChannelState', value);
    },
    /**
     * FIXME: this override is necessary just because the support channel is
     * considered as a channel, even though it does not inherit from
     * mail.model.Channel.
     *
     * @returns {integer}
     */
    getNeedactionCounter: function () {
        return 0;
    },
    /**
     * @return {string} uuid of this channel
     */
    getUUID: function () {
        return this._uuid;
    },
    /**
     * FIXME: this method is necessary just because the support channel is
     * considered as a channel, even though it does not inherit from
     * mail.model.Channel.
     *
     * @returns {boolean}
     */
    hasBeenPreviewed: function () {
        return true;
    },
    /**
     * @returns {boolean} true iff the Support channel is available
     */
    isAvailable: function () {
        return this._available;
    },
    /**
     * FIXME: this override is necessary just because the support channel is
     * considered as a channel, even though it does not inherit from
     * mail.model.Channel.
     *
     * @override
     * @returns {boolean}
     */
    isChannel: function () {
        return true;
    },
    /**
     * Called when fold or detach (or both) status have changed on the support
     * channel.
     *
     * Note: this is partially a hack due to support channel not being a
     * channel. Also, the support channel uses another way to handle its window
     * state, by means of the local storage.
     *
     * @param {Object} params
     * @param {boolean} [params.folded]
     * @param {boolean} [params.detached]
     */
    updateWindowState: function (params) {
        if ('detached' in params) {
            this._detached = params.detached;
        }
        if ('folded' in params) {
            this._folded = params.folded;
        }

        if (!this._detached) {
            this.call('mail_service', 'updateSupportChannelState', 'closed');
        } else {
            var value = this._folded ? 'folded' : 'open';
            this.call('mail_service', 'updateSupportChannelState', value);
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Adds a message in the Support channel indicating that this channel is
     * not available for now.
     *
     * @private
     */
    _addSupportNotAvailableMessage: function () {
        var msg = {
            author_id: this.call('mail_service', 'getOdoobotID'),
            body: _t("None of our operators are available. <a href='https://www.odoo.com/help' " +
                "target='_blank'>Submit a ticket</a> to ask your question now."),
            channel_ids: [this.getID()],
            id: Number.MAX_SAFE_INTEGER, // last message in the channel
        };
        this.call('mail_service', 'addMessage', msg, { silent: true });
    },
    /**
     * Adds the welcome message as first message of the Support channel.
     *
     * @private
     */
    _addSupportWelcomeMessage: function () {
        if (this._welcomeMessage) {
            var msg = {
                author_id: this._operator,
                body: this._welcomeMessage,
                channel_ids: [this.getID()],
                id: -1, // first message of the channel
            };
            this.call('mail_service', 'addMessage', msg, { silent: true });
        }
    },
    /**
     * Fetches the messages from the Support server.
     *
     * @override
     * @private
     */
    _fetchMessages: function (pDomain, loadMore) {
        var self = this;
        var domain = [];
        var cache = this._getCache(pDomain);
        if (pDomain) {
            domain = domain.concat(pDomain || []);
        }
        if (loadMore) {
            // ignore the welcome message (ID==-1)
            var msgs = cache.messages.filter(function(m) {return m.getID() !== -1});
            var minMessageID = msgs[0].getID();
            domain = [['id', '<', minMessageID]].concat(domain);
        }
        return supportSession.rpc('/odoo_im_support/fetch_messages', {
                domain: domain,
                channel_uuid: session.support_token,
                limit: self._FETCH_LIMIT,
        }).then(function (messages) {
            if (!cache.allHistoryLoaded) {
                cache.allHistoryLoaded = messages.length < self._FETCH_LIMIT;
            }
            cache.loaded = true;
            _.each(messages, function (message) {
                message.channel_ids = [self.getID()];
                message.channel_id = self.getID();
                self.call('mail_service', 'addMessage', message, {
                    silent: true,
                    domain: pDomain,
                });
            });
            cache = self._getCache(pDomain || []);
            return cache.messages;
        });
    },

    /**
     * Posts the message on the Support server.
     *
     * @override
     * @private
     * @return {Promise}
     */
    _postMessage: function (data) {
        // ensure that the poll is active before posting the message
        this.call('mail_service', 'startPollingSupport');
        return supportSession.rpc('/odoo_im_support/chat_post', {
            uuid: this._supportChannelUUID,
            message_content: data.content,
        });
    },
});

return SupportChannel;

});

```

## File: static\src\js\support_message.js

```javascript
odoo.define('im_support.SupportMessage', function (require) {
"use strict";

var Message = require('mail.model.Message');

var session = require('web.session');

/**
 * This is a model for messages that are in the the support channel
 */
var SupportMessage = Message.extend({
    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        if (
            !this._serverAuthorID ||
            this._serverAuthorID[0] !== this.call('mail_service', 'getOdoobotID')[0]
        ) {
            if (!this._serverAuthorID[0]) {
                // the author is the client
                this._serverAuthorID = [session.partner_id, session.name];
            } else {
                // the author is the operator
                // prevent from conflicting with partners of this instance
                this._serverAuthorID[0] = -1;
            }
        }
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     * @returns {string}
     */
    getAvatarSource: function () {
        return '/mail/static/src/img/odoo_o.png';
    },
    /**
     * Overrides to prevent clicks on the Support operator from redirecting.
     *
     * @override
     * @returns {boolean}
     */
    shouldRedirectToAuthor: function () {
        return false;
    },
    /**
     * Overrides to prevent from calling the server for messages of the Support
     * channel (which are records on the Support database).
     *
     * @override
     * @returns {Promise}
     */
    toggleStarStatus: function () {
        return Promise.resolve();
    },
});

return SupportMessage;

});

```

## File: static\src\js\support_session.js

```javascript
odoo.define('im_support.SupportSession', function (require) {
"use strict";

var session = require('web.session');
var Session = require('web.Session');

/**
 * This module returns an instance of Session which is linked to the Support
 * server, allowing the current instance to communicate with the Support
 * server (CORS).
 */
return new Session(null, session.support_origin, {
    modules: odoo._modules,
    use_cors: true,
});

});

```

## File: static\src\js\systray_messaging_menu.js

```javascript
odoo.define('im_support.systray.MessagingMenu', function (require) {
"use strict";

var MessagingMenu = require('mail.systray.MessagingMenu');

var config = require('web.config');
var core = require('web.core');
var session = require('web.session');

var _t = core._t;
var SUPPORT_CHANNEL_ID = 'SupportChannel';

// Disable Support in mobile for design purposes (for now at least), and don't
// add it to the messaging dropdown if it isn't available
if (config.device.isMobile || !session.support_token || !session.support_origin) {
    return;
}

/**
 * This module adds a fake Support channel in the messaging menu of the systray.
 */
MessagingMenu.include({
    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this.supportChannel = {
            id: SUPPORT_CHANNEL_ID,
            name: _t("Odoo Live Support"),
            title: _t("Odoo Live Support"),
            imageSRC: '/mail/static/src/img/odoo_o.png',
        };
    },
    /**
     * Overrides to add a className to the bottom part of the dropdown
     * (containing the Support channel), so that the css rules apply. This
     * className can't be added directly in the template, otherwise
     * this.$channels_preview would be a nodeset containing the bottom part as
     * well, and it will cause rendering issues when the dropdown is rerendered.
     *
     * @override
     */
    start: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            self.$('.o_mail_systray_dropdown_bottom').addClass('o_mail_systray_dropdown_items');
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Overrides to handle the click on the fake Support channel.
     *
     * @override
     * @private
     */
    _onClickPreview: function (ev) {
        var id = $(ev.currentTarget).data('preview-id');
        if (id === SUPPORT_CHANNEL_ID) {
            this.call('mail_service', 'startSupportLivechat');
        } else {
            this._super.apply(this, arguments);
        }
    },
});

});

```

## File: static\src\js\thread_window.js

```javascript
odoo.define('im_support.ThreadWindow', function (require) {
"use strict";

/**
 * This module includes ThreadWindow to handle the case of the Support channel.
 */
var ThreadWindow = require('mail.ThreadWindow');

ThreadWindow.include({
    /**
     * Overrides to tweak the options in the case of the Support channel (no
     * star).
     *
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        var channel = this.call('mail_service', 'getChannel', this._getThreadID());
        if (channel && channel.getType() === 'support_channel') {
            this.options.displayStars = false;
        }
    },
    /**
     * Overrides to remove the attachment button in the Support channel as this
     * feature does not work in that channel.
     *
     * @note: this feature could be enabled by storing the attachments on the
     * Support server.
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);
        var channel = this.call('mail_service', 'getChannel', this._getThreadID());
        if (channel && channel.getType() === 'support_channel') {
            this.$('.o_composer .o_composer_button_add_attachment').remove();
            if (!channel.isAvailable()) {
                this.$('.o_thread_composer').remove();
            }
        }
        return def;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Overrides to prevent from displaying a composer if there is no operator
     * available.
     *
     * @override
     * @private
     */
    _needsComposer: function () {
        var channel = this.call('mail_service', 'getChannel', this._getThreadID());
        if (
            channel &&
            channel.getType() === 'support_channel' &&
            !channel.isAvailable()
        ) {
            return false;
        }
        return this._super.apply(this, arguments);
    },
});

});

```

## File: static\src\xml\discuss.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<template>

    <!--
        @param {string|integer} activeThreadID
        @param {mail.model.Channel[]} channels
        @param {boolean} isMyselfModerator
        @param {mail.model.Mailbox} inbox
        @param {mail.model.Mailbox} starred
        @param {mail.model.Mailbox|undefined} moderation set if current user is moderator
        @param {boolean} displayQuickSearch
        @param {Object} options
    -->
    <t t-extend="mail.discuss.Sidebar">
        <t t-jquery="hr" t-operation="before">
            <t t-foreach="channels" t-as="channel">
                <t t-if="channel.getType() == 'support_channel'">
                    <div t-attf-class="o_mail_discuss_title_main o_mail_discuss_item #{(activeThreadID == channel.getID()) ? 'o_active': ''}"
                         t-att-data-thread-id="channel.getID()">
                        <span class="o_thread_name"><i class="fa fa-question-circle mr8"/>Odoo Support</span>
                        <t t-set="counter" t-value="channel.getUnreadCounter()"/>
                        <t t-call="mail.discuss.SidebarCounter"/>
                    </div>
                </t>
            </t>
        </t>
    </t>

</template>

```

## File: static\src\xml\systray.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-extend="mail.systray.MessagingMenu">
        <t t-jquery=".o_mail_systray_dropdown" t-operation="append">
            <div t-if="widget.supportChannel" class="o_mail_systray_dropdown_bottom">
                <t t-set="preview" t-value="widget.supportChannel"/>
                <t t-call="mail.Preview"/>
            </div>
        </t>
    </t>

</templates>

```

## File: views\assets.xml

```xml
<openerp>
  <data>
    <template id="assets_backend" name="im_support_backend assets" inherit_id="web.assets_backend">
      <xpath expr="." position="inside">
        <link rel="stylesheet" href="/im_support/static/src/scss/systray.scss" type="text/scss"/>

        <script type="text/javascript" src="/im_support/static/src/js/mail_manager.js"></script>
        <script type="text/javascript" src="/im_support/static/src/js/discuss.js"></script>
        <script type="text/javascript" src="/im_support/static/src/js/thread_window.js"></script>
        <script type="text/javascript" src="/im_support/static/src/js/support_bus.js"></script>
        <script type="text/javascript" src="/im_support/static/src/js/support_channel.js"></script>
        <script type="text/javascript" src="/im_support/static/src/js/support_message.js"></script>
        <script type="text/javascript" src="/im_support/static/src/js/support_session.js"></script>
        <script type="text/javascript" src="/im_support/static/src/js/systray_messaging_menu.js"></script>
      </xpath>
    </template>

    <template id="qunit_suite" name="im_support_tests" inherit_id="web.qunit_suite">
      <xpath expr="//t[@t-set='head']" position="inside">
        <script type="text/javascript" src="/im_support/static/tests/helpers/test_utils.js"></script>
        <script type="text/javascript" src="/im_support/static/tests/systray_no_support_tests.js"></script>
      </xpath>
    </template>

    <template id="qunit_mobile_suite" name="im_support_mobile_tests" inherit_id="web.qunit_mobile_suite">
      <xpath expr="//t[@t-set='head']" position="inside">
        <script type="text/javascript" src="/im_support/static/tests/helpers/test_utils.js"></script>
      </xpath>
    </template>

    <template id="im_support.support_qunit_suite">
      <t t-call="web.layout">
        <t t-set="html_data" t-value="{'style': 'height: 100%;'}"/>
        <t t-set="title">IM Support Tests</t>
        <t t-set="head">
          <script type="text/javascript">
            odoo.session_info = {
              support_token: 'ABCDEFGHIJ',
              support_origin: 'https://something.com'
            };
          </script>
          <t t-call="web.js_tests_assets"/>

          <script type="text/javascript" src="/im_support/static/tests/helpers/test_utils.js"></script>
          <script type="text/javascript" src="/im_support/static/tests/systray_tests.js"></script>
        </t>

        <div id="qunit"/>
        <div id="qunit-fixture"/>
      </t>
    </template>
  </data>
</openerp>

```

