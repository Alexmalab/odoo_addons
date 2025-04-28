# Odoo Module: mail_bot

Category: Discuss

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'OdooBot',
    'version': '1.0',
    'category': 'Discuss',
    'summary': 'Add OdooBot in discussions',
    'description': "",
    'website': 'https://www.odoo.com/page/discuss',
    'depends': ['mail'],
    'installable': True,
    'application': False,
    'data': [
        'views/assets.xml',
        'views/res_users_views.xml',
        'data/mailbot_data.xml',
    ],
    'demo': [
        'data/mailbot_demo.xml',
    ],
    'qweb': [
        'views/discuss.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\mailbot_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="base.partner_root" model="res.partner">
            <field name="name">OdooBot</field>
            <field name="email">odoobot@example.com</field>
            <field name="image_1920" type="base64" file="mail/static/src/img/odoobot.png"/>
        </record>
    </data>
</odoo>

```

## File: data\mailbot_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Disable odoobot on admin so that devs don't hate him -->
        <record id="base.user_admin" model="res.users">
            <field name="odoobot_state">disabled</field>
        </record>
    </data>
</odoo>

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Http(models.AbstractModel):
    _inherit = 'ir.http'

    def session_info(self):
        res = super(Http, self).session_info()
        if self.env.user.has_group('base.group_user'):
            res['odoobot_initialized'] = self.env.user.odoobot_state != 'not_initialized'
        return res

```

## File: models\mail_bot.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import itertools
import random

from odoo import models, _


class MailBot(models.AbstractModel):
    _name = 'mail.bot'
    _description = 'Mail Bot'

    def _apply_logic(self, record, values, command=None):
        """ Apply bot logic to generate an answer (or not) for the user
        The logic will only be applied if odoobot is in a chat with a user or
        if someone pinged odoobot.

         :param record: the mail_thread (or mail_channel) where the user
            message was posted/odoobot will answer.
         :param values: msg_values of the message_post or other values needed by logic
         :param command: the name of the called command if the logic is not triggered by a message_post
        """
        odoobot_id = self.env['ir.model.data'].xmlid_to_res_id("base.partner_root")
        if len(record) != 1 or values.get("author_id") == odoobot_id:
            return
        if self._is_bot_pinged(values) or self._is_bot_in_private_channel(record):
            body = values.get("body", "").replace(u'\xa0', u' ').strip().lower().strip(".?!")
            answer = self._get_answer(record, body, values, command)
            if answer:
                message_type = values.get('message_type', 'comment')
                subtype_id = values.get('subtype_id', self.env['ir.model.data'].xmlid_to_res_id('mail.mt_comment'))
                record.with_context(mail_create_nosubscribe=True).sudo().message_post(body=answer, author_id=odoobot_id, message_type=message_type, subtype_id=subtype_id)

    def _get_answer(self, record, body, values, command=False):
        # onboarding
        odoobot_state = self.env.user.odoobot_state
        if self._is_bot_in_private_channel(record):
            # main flow
            if odoobot_state == 'onboarding_emoji' and self._body_contains_emoji(body):
                self.env.user.odoobot_state = "onboarding_attachement"
                return _("Great! 👍<br/>Now, try to <b>send an attachment</b>, like a picture of your cute dog...")
            elif odoobot_state == 'onboarding_attachement' and values.get("attachment_ids"):
                self.env.user.odoobot_state = "onboarding_command"
                return _("Not a cute dog, but you get it 😊<br/>To access special features, <b>start your sentence with '/'</b>. Try to get help.")
            elif odoobot_state == 'onboarding_command' and command == 'help':
                self.env.user.odoobot_state = "onboarding_ping"
                return _("Wow you are a natural!<br/>Ping someone to grab its attention with @nameoftheuser. <b>Try to ping me using @OdooBot</b> in a sentence.")
            elif odoobot_state == 'onboarding_ping' and self._is_bot_pinged(values):
                self.env.user.odoobot_state = "idle"
                return _("Yep, I am here! 🎉 <br/>You finished the tour, you can <b>close this chat window</b>. Enjoy discovering Odoo.")
            elif odoobot_state == "idle" and (_('start the tour') in body.lower()):
                self.env.user.odoobot_state = "onboarding_emoji"
                return _("To start, try to send me an emoji :)")
            # easter eggs
            elif odoobot_state == "idle" and body in ['❤️', _('i love you'), _('love')]:
                return _("Aaaaaw that's really cute but, you know, bots don't work that way. You're too human for me! Let's keep it professional ❤️")
            elif odoobot_state == "idle" and (('help' in body) or _('help') in body):
                return _("I'm just a bot... :( You can check <a href=\"https://www.odoo.com/page/docs\">our documentation</a>) for more information!")
            elif _('fuck') in body or "fuck" in body:
                return _("That's not nice! I'm a bot but I have feelings... 💔")
            else:
                #repeat question
                if odoobot_state == 'onboarding_emoji':
                    return _("Not exactly. To continue the tour, send an emoji, <b>type \":)\"</b> and press enter.")
                elif odoobot_state == 'onboarding_attachement':
                    return _("To <b>send an attachment</b>, click the 📎 icon on the right, and select a file.")
                elif odoobot_state == 'onboarding_command':
                    return _("Not sure wat you are doing. Please press / and wait for the propositions. Select \"help\" and press enter")
                elif odoobot_state == 'onboarding_ping':
                    return _("Sorry, I am not listening. To get someone's attention, <b>ping him</b>. Write \"@odoobot\" and select me.")
                return random.choice([
                    _("I'm not smart enough to answer your question.<br/>To follow my guide, ask") + ": <b>"+_('start the tour') + "</b>",
                    _("Hmmm..."),
                    _("I'm afraid I don't understand. Sorry!"),
                    _("Sorry I'm sleepy. Or not! Maybe I'm just trying to hide my unawareness of human language...<br/>I can show you features if you write")+ ": '<b>"+_('start the tour')+"</b>'.",
                ])
        elif self._is_bot_pinged(values):
            return random.choice([_("Yep, OdooBot is in the place!"), _("Pong.")])
        return False

    def _body_contains_emoji(self, body):
        # coming from https://unicode.org/emoji/charts/full-emoji-list.html
        emoji_list = itertools.chain(
            range(0x231A, 0x231c),
            range(0x23E9, 0x23f4),
            range(0x23F8, 0x23fb),
            range(0x25AA, 0x25ac),
            range(0x25FB, 0x25ff),
            range(0x2600, 0x2605),
            range(0x2614, 0x2616),
            range(0x2622, 0x2624),
            range(0x262E, 0x2630),
            range(0x2638, 0x263b),
            range(0x2648, 0x2654),
            range(0x265F, 0x2661),
            range(0x2665, 0x2667),
            range(0x267E, 0x2680),
            range(0x2692, 0x2698),
            range(0x269B, 0x269d),
            range(0x26A0, 0x26a2),
            range(0x26AA, 0x26ac),
            range(0x26B0, 0x26b2),
            range(0x26BD, 0x26bf),
            range(0x26C4, 0x26c6),
            range(0x26D3, 0x26d5),
            range(0x26E9, 0x26eb),
            range(0x26F0, 0x26f6),
            range(0x26F7, 0x26fb),
            range(0x2708, 0x270a),
            range(0x270A, 0x270c),
            range(0x270C, 0x270e),
            range(0x2733, 0x2735),
            range(0x2753, 0x2756),
            range(0x2763, 0x2765),
            range(0x2795, 0x2798),
            range(0x2934, 0x2936),
            range(0x2B05, 0x2b08),
            range(0x2B1B, 0x2b1d),
            range(0x1F170, 0x1f172),
            range(0x1F191, 0x1f19b),
            range(0x1F1E6, 0x1f200),
            range(0x1F201, 0x1f203),
            range(0x1F232, 0x1f23b),
            range(0x1F250, 0x1f252),
            range(0x1F300, 0x1f321),
            range(0x1F324, 0x1f32d),
            range(0x1F32D, 0x1f330),
            range(0x1F330, 0x1f336),
            range(0x1F337, 0x1f37d),
            range(0x1F37E, 0x1f380),
            range(0x1F380, 0x1f394),
            range(0x1F396, 0x1f398),
            range(0x1F399, 0x1f39c),
            range(0x1F39E, 0x1f3a0),
            range(0x1F3A0, 0x1f3c5),
            range(0x1F3C6, 0x1f3cb),
            range(0x1F3CB, 0x1f3cf),
            range(0x1F3CF, 0x1f3d4),
            range(0x1F3D4, 0x1f3e0),
            range(0x1F3E0, 0x1f3f1),
            range(0x1F3F3, 0x1f3f6),
            range(0x1F3F8, 0x1f400),
            range(0x1F400, 0x1f43f),
            range(0x1F442, 0x1f4f8),
            range(0x1F4F9, 0x1f4fd),
            range(0x1F500, 0x1f53e),
            range(0x1F549, 0x1f54b),
            range(0x1F54B, 0x1f54f),
            range(0x1F550, 0x1f568),
            range(0x1F56F, 0x1f571),
            range(0x1F573, 0x1f57a),
            range(0x1F58A, 0x1f58e),
            range(0x1F595, 0x1f597),
            range(0x1F5B1, 0x1f5b3),
            range(0x1F5C2, 0x1f5c5),
            range(0x1F5D1, 0x1f5d4),
            range(0x1F5DC, 0x1f5df),
            range(0x1F5FB, 0x1f600),
            range(0x1F601, 0x1f611),
            range(0x1F612, 0x1f615),
            range(0x1F61C, 0x1f61f),
            range(0x1F620, 0x1f626),
            range(0x1F626, 0x1f628),
            range(0x1F628, 0x1f62c),
            range(0x1F62E, 0x1f630),
            range(0x1F630, 0x1f634),
            range(0x1F635, 0x1f641),
            range(0x1F641, 0x1f643),
            range(0x1F643, 0x1f645),
            range(0x1F645, 0x1f650),
            range(0x1F680, 0x1f6c6),
            range(0x1F6CB, 0x1f6d0),
            range(0x1F6D1, 0x1f6d3),
            range(0x1F6E0, 0x1f6e6),
            range(0x1F6EB, 0x1f6ed),
            range(0x1F6F4, 0x1f6f7),
            range(0x1F6F7, 0x1f6f9),
            range(0x1F910, 0x1f919),
            range(0x1F919, 0x1f91f),
            range(0x1F920, 0x1f928),
            range(0x1F928, 0x1f930),
            range(0x1F931, 0x1f933),
            range(0x1F933, 0x1f93b),
            range(0x1F93C, 0x1f93f),
            range(0x1F940, 0x1f946),
            range(0x1F947, 0x1f94c),
            range(0x1F94D, 0x1f950),
            range(0x1F950, 0x1f95f),
            range(0x1F95F, 0x1f96c),
            range(0x1F96C, 0x1f971),
            range(0x1F973, 0x1f977),
            range(0x1F97C, 0x1f980),
            range(0x1F980, 0x1f985),
            range(0x1F985, 0x1f992),
            range(0x1F992, 0x1f998),
            range(0x1F998, 0x1f9a3),
            range(0x1F9B0, 0x1f9ba),
            range(0x1F9C1, 0x1f9c3),
            range(0x1F9D0, 0x1f9e7),
            range(0x1F9E7, 0x1fa00),
            [0x2328, 0x23cf, 0x24c2, 0x25b6, 0x25c0, 0x260e, 0x2611, 0x2618, 0x261d, 0x2620, 0x2626,
             0x262a, 0x2640, 0x2642, 0x2663, 0x2668, 0x267b, 0x2699, 0x26c8, 0x26ce, 0x26cf,
             0x26d1, 0x26fd, 0x2702, 0x2705, 0x270f, 0x2712, 0x2714, 0x2716, 0x271d, 0x2721, 0x2728, 0x2744, 0x2747, 0x274c,
             0x274e, 0x2757, 0x27a1, 0x27b0, 0x27bf, 0x2b50, 0x2b55, 0x3030, 0x303d, 0x3297, 0x3299, 0x1f004, 0x1f0cf, 0x1f17e,
             0x1f17f, 0x1f18e, 0x1f21a, 0x1f22f, 0x1f321, 0x1f336, 0x1f37d, 0x1f3c5, 0x1f3f7, 0x1f43f, 0x1f440, 0x1f441, 0x1f4f8,
             0x1f4fd, 0x1f4ff, 0x1f57a, 0x1f587, 0x1f590, 0x1f5a4, 0x1f5a5, 0x1f5a8, 0x1f5bc, 0x1f5e1, 0x1f5e3, 0x1f5e8, 0x1f5ef,
             0x1f5f3, 0x1f5fa, 0x1f600, 0x1f611, 0x1f615, 0x1f616, 0x1f617, 0x1f618, 0x1f619, 0x1f61a, 0x1f61b, 0x1f61f, 0x1f62c,
             0x1f62d, 0x1f634, 0x1f6d0, 0x1f6e9, 0x1f6f0, 0x1f6f3, 0x1f6f9, 0x1f91f, 0x1f930, 0x1f94c, 0x1f97a, 0x1f9c0]
        )
        if any(chr(emoji) in body for emoji in emoji_list):
            return True
        return False

    def _is_bot_pinged(self, values):
        odoobot_id = self.env['ir.model.data'].xmlid_to_res_id("base.partner_root")
        return odoobot_id in values.get('partner_ids', [])

    def _is_bot_in_private_channel(self, record):
        odoobot_id = self.env['ir.model.data'].xmlid_to_res_id("base.partner_root")
        if record._name == 'mail.channel' and record.channel_type == 'chat':
            return odoobot_id in record.with_context(active_test=False).channel_partner_ids.ids
        return False

```

## File: models\mail_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _


class Channel(models.Model):
    _inherit = 'mail.channel'

    def _execute_command_help(self, **kwargs):
        super(Channel, self)._execute_command_help(**kwargs)
        self.env['mail.bot']._apply_logic(self, kwargs, command="help")  # kwargs are not usefull but...

    @api.model
    def init_odoobot(self):
        if self.env.user.odoobot_state == 'not_initialized':
            partner = self.env.user.partner_id
            odoobot_id = self.env['ir.model.data'].xmlid_to_res_id("base.partner_root")
            channel = self.with_context(mail_create_nosubscribe=True).create({
                'channel_partner_ids': [(4, partner.id), (4, odoobot_id)],
                'public': 'private',
                'channel_type': 'chat',
                'email_send': False,
                'name': 'OdooBot'
            })
            message = _("Hello,<br/>Odoo's chat helps employees collaborate efficiently. I'm here to help you discover its features.<br/><b>Try to send me an emoji :)</b>")
            channel.sudo().message_post(body=message, author_id=odoobot_id, message_type="comment", subtype="mail.mt_comment")
            self.env.user.odoobot_state = 'onboarding_emoji'
            return channel

```

## File: models\mail_thread.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class MailThread(models.AbstractModel):
    _inherit = 'mail.thread'

    def _message_post_after_hook(self, message, msg_vals):
        self.env['mail.bot']._apply_logic(self, msg_vals)
        return super(MailThread, self)._message_post_after_hook(message, msg_vals)

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, models


class Partner(models.Model):
    _inherit = 'res.partner'

    def _compute_im_status(self):
        #we asume that mail_bot _compute_im_status will be executed after bus _compute_im_status
        super(Partner, self)._compute_im_status()
        odoobot_id = self.env['ir.model.data'].xmlid_to_res_id("base.partner_root")
        for partner in self:
            if partner.id == odoobot_id:
                partner.im_status = 'bot'

    @api.model
    def get_mention_suggestions(self, search, limit=8):
        #add odoobot in mention suggestion when pinging in mail_thread
        [users, partners] = super(Partner, self).get_mention_suggestions(search, limit=limit)
        if len(partners) + len(users) < limit and "odoobot".startswith(search.lower()):
            odoobot = self.env.ref("base.partner_root")
            if not any([elem['id'] == odoobot.id for elem in partners]):
                if odoobot:
                    partners.append({'id': odoobot.id, 'name': odoobot.name, 'email': odoobot.email})
        return [users, partners]


```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields

class Users(models.Model):
    _inherit = 'res.users'
    odoobot_state = fields.Selection(
        [
            ('not_initialized', 'Not initialized'),
            ('onboarding_emoji', 'Onboarding emoji'),
            ('onboarding_attachement', 'Onboarding attachement'),
            ('onboarding_command', 'Onboarding command'),
            ('onboarding_ping', 'Onboarding ping'),
            ('idle', 'Idle'),
            ('disabled', 'Disabled'),
        ], string="OdooBot Status", readonly=True, required=True, default="not_initialized")  # keep track of the state: correspond to the code of the last message sent

    def __init__(self, pool, cr):
        """ Override of __init__ to add access rights.
            Access rights are disabled by default, but allowed
            on some specific fields defined in self.SELF_{READ/WRITE}ABLE_FIELDS.
        """
        init_res = super(Users, self).__init__(pool, cr)
        # duplicate list to avoid modifying the original reference
        type(self).SELF_READABLE_FIELDS = type(self).SELF_READABLE_FIELDS + ['odoobot_state']
        return init_res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_http
from . import mail_bot
from . import mail_channel
from . import mail_thread
from . import res_partner
from . import res_users

```

## File: static\src\js\mailbot_service.js

```javascript
odoo.define('mail_bot.MailBotService', function (require) {
"use strict";

var AbstractService = require('web.AbstractService');
var core = require('web.core');
var session = require('web.session');

var _t = core._t;

var MailBotService =  AbstractService.extend({
    /**
     * @override
     */
    start: function () {
        this._hasRequest = (window.Notification && window.Notification.permission === "default") || false;
        if ('odoobot_initialized' in session && ! session.odoobot_initialized) {
            this._showOdoobotTimeout();
        }
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Get the previews related to the OdooBot (conversation not included).
     * For instance, when there is no conversation with OdooBot and OdooBot has
     * a request, it should display a preview in the systray messaging menu.
     *
     * @param {string|undefined} [filter]
     * @returns {Object[]} list of objects that are compatible with the
     *   'mail.Preview' template.
     */
    getPreviews: function (filter) {
        if (!this.isRequestingForNativeNotifications()) {
            return [];
        }
        if (filter && filter !== 'mailbox_inbox') {
            return [];
        }
        var previews = [{
            title: _t("OdooBot has a request"),
            imageSRC: "/mail/static/src/img/odoobot.png",
            status: 'bot',
            body:  _t("Enable desktop notifications to chat"),
            id: 'request_notification',
            unreadCounter: 1,
        }];
        return previews;
    },
    /**
     * Tell whether OdooBot is requesting to enable push notifications.
     *
     * @returns {boolean}
     */
    isRequestingForNativeNotifications: function () {
        return this._hasRequest;
    },
    /**
     * Called when user either accepts or refuses push notifications.
     */
    removeRequest: function () {
        this._hasRequest = false;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _showOdoobotTimeout: function () {
        var self = this;
        setTimeout(function () {
            session.odoobot_initialized = true;
            self._rpc({
                model: 'mail.channel',
                method: 'init_odoobot',
            });
        }, 2*60*1000);
    },
});

core.serviceRegistry.add('mailbot_service', MailBotService);
return MailBotService;

});

```

## File: static\src\js\notification_alert.js

```javascript
odoo.define('mail.NotificationAlert', function (require) {
"use strict";

var Widget = require('web.Widget');
var widgetRegistry = require('web.widget_registry');

// -----------------------------------------------------------------------------
// Display Notification alert on user preferences form view
// -----------------------------------------------------------------------------
var NotificationAlert = Widget.extend({
   template: 'mail.NotificationAlert',
   /**
    * @override
    */
   init: function () {
      this._super.apply(this, arguments);
      var hasRequest = this.call('mailbot_service', 'isRequestingForNativeNotifications');
      this.isNotificationBlocked = window.Notification && window.Notification.permission !== "granted" && !hasRequest;
   },
});

widgetRegistry.add('notification_alert', NotificationAlert);

return NotificationAlert;

});

```

## File: static\src\js\systray_messaging_menu.js

```javascript
odoo.define('mail_bot.systray.MessagingMenu', function (require) {
"use strict";

var MessagingMenu = require('mail.systray.MessagingMenu');
var core = require('web.core');

var _t = core._t;

return MessagingMenu.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Override so that 'mailbot has a request' is included in the computation
     * of of the counter.
     *
     * @override
     * @private
     * @returns {integer}
     */
    _computeCounter: function () {
        var counter = this._super.apply(this, arguments);
        if (this.call('mailbot_service', 'isRequestingForNativeNotifications')) {
            counter++;
        }
        return counter;
    },
    /**
     * Override so that the mailbot previews are included in the systray
     * messaging menu (e.g. 'OdooBot has a request')
     *
     * @override
     * @private
     * @returns {Promise<Object[]>} resolved with list of previews that are
     *   compatible with the 'mail.Preview' template.
     */
    _getPreviews: function () {
        var mailbotPreviews = this.call('mailbot_service', 'getPreviews', this._filter);
        return this._super.apply(this, arguments).then(function (previews) {
            return _.union(mailbotPreviews, previews);
        });
    },
    /**
     * Handle the response of the user when prompted whether push notifications
     * are granted or denied.
     *
     * Also refreshes the counter after a response from a push notification
     * request. This is useful because the counter contains a part for the
     * OdooBot, and the OdooBot influences the counter by 1 when it requests
     * for notifications. This should no longer be the case when push
     * notifications are either granted or denied.
     *
     * @private
     * @param {string} value
     */
    _handleResponseNotificationPermission: function (value) {
        this.call('mailbot_service', 'removeRequest');
        if (value !== 'granted') {
            this.call('bus_service', 'sendNotification', _t('Permission denied'),
                _t('Odoo will not have the permission to send native notifications on this device.'));
        }
        this._updateCounter();
    },
    /**
     * Display the browser notification request dialog when the user clicks on
     * systray's corresponding notification
     *
     * @private
     */
    _requestNotificationPermission: function () {
        var def = window.Notification && window.Notification.requestPermission();
        if (def) {
            def.then(this._handleResponseNotificationPermission.bind(this));
        }
        this.$('.o_mail_navbar_request_permission').slideUp();
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Override so that it handles preview related to OdooBot
     *
     * @override
     * @private
     * @param {MouseEvent} ev
     */
    _onClickPreview: function (ev) {
        var previewID = $(ev.currentTarget).data('preview-id');
        if (previewID === 'request_notification') {
            this._requestNotificationPermission();
        } else {
            this._super.apply(this, arguments);
        }
    },
    /**
     * Override so that it handles clicking on 'mark as read' similarly to
     * requesting push notification permission.
     *
     * @override
     * @private
     * @param {MouseEvent} ev
     */
    _onClickPreviewMarkAsRead: function (ev) {
        ev.stopPropagation();
        var $preview = $(ev.currentTarget).closest('.o_mail_preview');
        var previewID = $preview.data('preview-id');
        if (previewID === 'request_notification') {
            this._requestNotificationPermission();
        } else {
            this._super.apply(this, arguments);
        }
    }
});

});

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="assets_backend" name="mailbot assets" inherit_id="web.assets_backend">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/mail_bot/static/src/js/mailbot_service.js"></script>
                <script type="text/javascript" src="/mail_bot/static/src/js/notification_alert.js"></script>
                <script type="text/javascript" src="/mail_bot/static/src/js/systray_messaging_menu.js"></script>
            </xpath>
        </template>
        <template id="qunit_suite" name="mail_tests" inherit_id="web.qunit_suite">
            <xpath expr="//t[@t-set='head']" position="inside">
                <script type="text/javascript" src="/mail_bot/static/tests/systray_messaging_menu_tests.js"></script>
            </xpath>
        </template>
    </data>
</odoo>

```

## File: views\discuss.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-extend="mail.UserStatus">
        <t t-jquery="i:last" t-operation="after">
            <i t-if="status == 'bot'" class="o_mail_user_status o_user_online fa fa-heart" title="Bot" role="img" aria-label="User is a bot"/>
        </t>
    </t>

    <!--
        @param {mail.NotificationAlert} widget
    -->
    <t t-name="mail.NotificationAlert">
       <center t-if="widget.isNotificationBlocked" class="o_notification_alert alert alert-primary">
           Odoo Push notifications have been blocked. Go to your browser settings to allow them.
       </center>
    </t>
</templates>

```

## File: views\res_users_views.xml

```xml
<?xml version="1.0"?>
<odoo><data>
    <!-- Update Preferences form !-->
    <record id="res_users_view_form_preferences" model="ir.ui.view">
        <field name="name">res.users.view.form.preferences.mail_bot</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="mail.view_users_form_simple_modif_mail"/>
        <field name="arch" type="xml">
            <data>
                <field name="notification_type" position="after">
                    <field name="odoobot_state" readonly="0" groups="base.group_no_one"/>
                </field>
            </data>
        </field>
    </record>

    <!-- Update user form !-->
    <record id="res_users_view_form" model="ir.ui.view">
        <field name="name">res.users.view.form.mail_bot</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="mail.view_users_form_mail"/>
        <field name="arch" type="xml">
            <data>
                <field name="alias_contact" position="after">
                    <field name="odoobot_state" readonly="0" groups="base.group_no_one"/>
                </field>
            </data>
        </field>
    </record>

    <!-- Blocked notification alert -->
    <record id="view_users_form_inherit_mail_notification_alert" model="ir.ui.view">
        <field name="name">res.users.preferences.form.notification.alert</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="base.view_users_form_simple_modif"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='image_1920']" position="before">
                <widget name="notification_alert"/>
            </xpath>
        </field>
    </record>

</data></odoo>

```

