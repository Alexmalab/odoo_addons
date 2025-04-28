# Odoo Module: mail_bot

Category: Productivity/Discuss

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
    'version': '1.2',
    'category': 'Productivity/Discuss',
    'summary': 'Add OdooBot in discussions',
    'website': 'https://www.odoo.com/app/discuss',
    'depends': ['mail'],
    'auto_install': True,
    'installable': True,
    'data': [
        'views/res_users_views.xml',
        'data/mailbot_data.xml',
    ],
    'demo': [
        'data/mailbot_demo.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'mail_bot/static/src/scss/odoobot_style.scss',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\mailbot_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="base.user_root" model="res.users">
            <field name="odoobot_state">disabled</field>
        </record>
    </data>
</odoo>

```

## File: data\mailbot_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Disable odoobot on admin so that devs don't hate it -->
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
        if self.env.user._is_internal():
            res['odoobot_initialized'] = self.env.user.odoobot_state not in [False, 'not_initialized']
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
        odoobot_id = self.env['ir.model.data']._xmlid_to_res_id("base.partner_root")
        if len(record) != 1 or values.get("author_id") == odoobot_id or values.get("message_type") != "comment" and not command:
            return
        if self._is_bot_pinged(values) or self._is_bot_in_private_channel(record):
            body = values.get("body", "").replace(u'\xa0', u' ').strip().lower().strip(".!")
            answer = self._get_answer(record, body, values, command)
            if answer:
                message_type = 'comment'
                subtype_id = self.env['ir.model.data']._xmlid_to_res_id('mail.mt_comment')
                record.with_context(mail_create_nosubscribe=True).sudo().message_post(body=answer, author_id=odoobot_id, message_type=message_type, subtype_id=subtype_id)

    def _get_answer(self, record, body, values, command=False):
        # onboarding
        odoobot_state = self.env.user.odoobot_state
        if self._is_bot_in_private_channel(record):
            # main flow
            if odoobot_state == 'onboarding_emoji' and self._body_contains_emoji(body):
                self.env.user.odoobot_state = "onboarding_command"
                self.env.user.odoobot_failed = False
                return _("Great! 👍<br/>To access special commands, <b>start your sentence with</b> <span class=\"o_odoobot_command\">/</span>. Try getting help.")
            elif odoobot_state == 'onboarding_command' and command == 'help':
                self.env.user.odoobot_state = "onboarding_ping"
                self.env.user.odoobot_failed = False
                return _("Wow you are a natural!<br/>Ping someone with @username to grab their attention. <b>Try to ping me using</b> <span class=\"o_odoobot_command\">@OdooBot</span> in a sentence.")
            elif odoobot_state == 'onboarding_ping' and self._is_bot_pinged(values):
                self.env.user.odoobot_state = "onboarding_attachement"
                self.env.user.odoobot_failed = False
                return _("Yep, I am here! 🎉 <br/>Now, try <b>sending an attachment</b>, like a picture of your cute dog...")
            elif odoobot_state == 'onboarding_attachement' and values.get("attachment_ids"):
                self.env.user.odoobot_state = "idle"
                self.env.user.odoobot_failed = False
                return _("I am a simple bot, but if that's a dog, he is the cutest 😊 <br/>Congratulations, you finished this tour. You can now <b>close this conversation</b> or start the tour again with typing <span class=\"o_odoobot_command\">start the tour</span>. Enjoy discovering Odoo!")
            elif odoobot_state in (False, "idle", "not_initialized") and (_('start the tour') in body.lower()):
                self.env.user.odoobot_state = "onboarding_emoji"
                return _("To start, try to send me an emoji :)")
            # easter eggs
            elif odoobot_state == "idle" and body in ['❤️', _('i love you'), _('love')]:
                return _("Aaaaaw that's really cute but, you know, bots don't work that way. You're too human for me! Let's keep it professional ❤️")
            elif _('fuck') in body or "fuck" in body:
                return _("That's not nice! I'm a bot but I have feelings... 💔")
            # help message
            elif self._is_help_requested(body) or odoobot_state == 'idle':
                return _("Unfortunately, I'm just a bot 😞 I don't understand! If you need help discovering our product, please check "
                         "<a href=\"https://www.odoo.com/documentation\" target=\"_blank\">our documentation</a> or "
                         "<a href=\"https://www.odoo.com/slides\" target=\"_blank\">our videos</a>.")
            else:
                # repeat question
                if odoobot_state == 'onboarding_emoji':
                    self.env.user.odoobot_failed = True
                    return _("Not exactly. To continue the tour, send an emoji: <b>type</b> <span class=\"o_odoobot_command\">:)</span> and press enter.")
                elif odoobot_state == 'onboarding_attachement':
                    self.env.user.odoobot_failed = True
                    return _("To <b>send an attachment</b>, click on the <i class=\"fa fa-paperclip\" aria-hidden=\"true\"></i> icon and select a file.")
                elif odoobot_state == 'onboarding_command':
                    self.env.user.odoobot_failed = True
                    return _("Not sure what you are doing. Please, type <span class=\"o_odoobot_command\">/</span> and wait for the propositions. Select <span class=\"o_odoobot_command\">help</span> and press enter")
                elif odoobot_state == 'onboarding_ping':
                    self.env.user.odoobot_failed = True
                    return _("Sorry, I am not listening. To get someone's attention, <b>ping him</b>. Write <span class=\"o_odoobot_command\">@OdooBot</span> and select me.")
                return random.choice([
                    _("I'm not smart enough to answer your question.<br/>To follow my guide, ask: <span class=\"o_odoobot_command\">start the tour</span>."),
                    _("Hmmm..."),
                    _("I'm afraid I don't understand. Sorry!"),
                    _("Sorry I'm sleepy. Or not! Maybe I'm just trying to hide my unawareness of human language...<br/>I can show you features if you write: <span class=\"o_odoobot_command\">start the tour</span>.")
                ])
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
        odoobot_id = self.env['ir.model.data']._xmlid_to_res_id("base.partner_root")
        return odoobot_id in values.get('partner_ids', [])

    def _is_bot_in_private_channel(self, record):
        odoobot_id = self.env['ir.model.data']._xmlid_to_res_id("base.partner_root")
        if record._name == 'mail.channel' and record.channel_type == 'chat':
            return odoobot_id in record.with_context(active_test=False).channel_partner_ids.ids
        return False

    def _is_help_requested(self, body):
        """Returns whether a message linking to the documentation and videos
        should be sent back to the user.
        """
        return any(token in body for token in ['help', _('help'), '?']) or self.env.user.odoobot_failed

```

## File: models\mail_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _


class Channel(models.Model):
    _inherit = 'mail.channel'

    def execute_command_help(self, **kwargs):
        super().execute_command_help(**kwargs)
        self.env['mail.bot']._apply_logic(self, kwargs, command="help")  # kwargs are not usefull but...

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

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, _

class Users(models.Model):
    _inherit = 'res.users'

    odoobot_state = fields.Selection(
        [
            ('not_initialized', 'Not initialized'),
            ('onboarding_emoji', 'Onboarding emoji'),
            ('onboarding_attachement', 'Onboarding attachment'),
            ('onboarding_command', 'Onboarding command'),
            ('onboarding_ping', 'Onboarding ping'),
            ('idle', 'Idle'),
            ('disabled', 'Disabled'),
        ], string="OdooBot Status", readonly=True, required=False)  # keep track of the state: correspond to the code of the last message sent
    odoobot_failed = fields.Boolean(readonly=True)

    @property
    def SELF_READABLE_FIELDS(self):
        return super().SELF_READABLE_FIELDS + ['odoobot_state']

    def _init_messaging(self):
        if self.odoobot_state in [False, 'not_initialized'] and self._is_internal():
            self._init_odoobot()
        return super()._init_messaging()

    def _init_odoobot(self):
        self.ensure_one()
        odoobot_id = self.env['ir.model.data']._xmlid_to_res_id("base.partner_root")
        channel_info = self.env['mail.channel'].channel_get([odoobot_id, self.partner_id.id])
        channel = self.env['mail.channel'].browse(channel_info['id'])
        message = _("Hello,<br/>Odoo's chat helps employees collaborate efficiently. I'm here to help you discover its features.<br/><b>Try to send me an emoji</b> <span class=\"o_odoobot_command\">:)</span>")
        channel.sudo().message_post(body=message, author_id=odoobot_id, message_type="comment", subtype_xmlid="mail.mt_comment")
        self.sudo().odoobot_state = 'onboarding_emoji'
        return channel

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_http
from . import mail_bot
from . import mail_channel
from . import mail_thread
from . import res_users

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
    <linearGradient id="linear-gradient" x1="-909.8" y1="477.94" x2="-910.8" y2="476.94" gradientTransform="matrix(70, 0, 0, -70, 63756, 33455.73)" gradientUnits="userSpaceOnUse">
      <stop offset="0" stop-color="#269396"/>
      <stop offset="1" stop-color="#218689"/>
    </linearGradient>
  </defs>
  <g mask="url(#mask)">
    <g>
      <path d="M0,0H70V70H0Z" fill-rule="evenodd" fill="url(#linear-gradient)"/>
      <path d="M4,1H65c2.67,0,4.33.67,5,2V0H0V3C.67,1.67,2,1,4,1Z" fill="#fff" fill-opacity="0.38" fill-rule="evenodd"/>
      <path d="M21.76,69H4c-2,0-4-.15-4-4.06v-20L21.16,23.82c17.19-5.88,27.55,1.44,28.92,16.84Z" fill="#393939" fill-rule="evenodd" opacity="0.32" style="isolation: isolate"/>
      <path d="M4,69H65c2.67,0,4.33-1,5-3v4H0V66A3.92,3.92,0,0,0,4,69Z" fill-opacity="0.38" fill-rule="evenodd"/>
      <g>
        <path d="M33,21.2c-10.18,0-18.44,6.09-18.44,13.61a11.49,11.49,0,0,0,3.76,8.25A8.54,8.54,0,0,1,15,49.25c-.25.24-.5.44-.42.77a.69.69,0,0,0,.73.51,17.63,17.63,0,0,0,9.12-3.66A23.59,23.59,0,0,0,33,48.43h0c10.18,0,18.44-6.09,18.44-13.62S43.18,21.2,33,21.2ZM24.77,36.64a2.51,2.51,0,0,1-1.77.73,2,2,0,0,1-.49,0,2.43,2.43,0,0,1-.9-.37,2.24,2.24,0,0,1-.38-.31,2.49,2.49,0,0,1-.73-1.77,2.09,2.09,0,0,1,.05-.49,2.44,2.44,0,0,1,.14-.46,2.22,2.22,0,0,1,.23-.43,1.91,1.91,0,0,1,.31-.38,2.24,2.24,0,0,1,.38-.31,3.62,3.62,0,0,1,.43-.24,2.52,2.52,0,0,1,2.73.55,2.33,2.33,0,0,1,.31.38,2.93,2.93,0,0,1,.23.43,2.44,2.44,0,0,1,.14.46,2.81,2.81,0,0,1,.05.49A2.53,2.53,0,0,1,24.77,36.64Zm10,0a2.51,2.51,0,0,1-1.77.73,2,2,0,0,1-.49,0,2.43,2.43,0,0,1-.9-.37,2.24,2.24,0,0,1-.38-.31,2.49,2.49,0,0,1-.73-1.77,2.09,2.09,0,0,1,.05-.49,2.44,2.44,0,0,1,.14-.46,2.22,2.22,0,0,1,.23-.43,1.91,1.91,0,0,1,.31-.38,2.24,2.24,0,0,1,.38-.31,3.62,3.62,0,0,1,.43-.24,2.52,2.52,0,0,1,2.73.55,2.33,2.33,0,0,1,.31.38,2.93,2.93,0,0,1,.23.43,2.44,2.44,0,0,1,.14.46,2.81,2.81,0,0,1,0,.49A2.53,2.53,0,0,1,34.77,36.64Zm10,0a2.51,2.51,0,0,1-1.77.73,2,2,0,0,1-.49,0,2.43,2.43,0,0,1-.9-.37,2.24,2.24,0,0,1-.38-.31,2.49,2.49,0,0,1-.73-1.77,2.09,2.09,0,0,1,0-.49,2.44,2.44,0,0,1,.14-.46,2.22,2.22,0,0,1,.23-.43,1.91,1.91,0,0,1,.31-.38,2.24,2.24,0,0,1,.38-.31,3.62,3.62,0,0,1,.43-.24,2.52,2.52,0,0,1,2.73.55,2.33,2.33,0,0,1,.31.38,2.93,2.93,0,0,1,.23.43,2.44,2.44,0,0,1,.14.46,2.81,2.81,0,0,1,0,.49A2.53,2.53,0,0,1,44.77,36.64Z" opacity="0.4"/>
        <path d="M35,19.2c-10.18,0-18.44,6.09-18.44,13.61a11.49,11.49,0,0,0,3.76,8.25A8.54,8.54,0,0,1,17,47.25c-.25.24-.5.44-.42.77a.69.69,0,0,0,.73.51,17.63,17.63,0,0,0,9.12-3.66A23.59,23.59,0,0,0,35,46.43h0c10.18,0,18.44-6.09,18.44-13.62S45.18,19.2,35,19.2ZM26.77,34.64a2.51,2.51,0,0,1-1.77.73,2,2,0,0,1-.49,0,2.43,2.43,0,0,1-.9-.37,2.24,2.24,0,0,1-.38-.31,2.49,2.49,0,0,1-.73-1.77,2.09,2.09,0,0,1,.05-.49,2.44,2.44,0,0,1,.14-.46,2.22,2.22,0,0,1,.23-.43,1.91,1.91,0,0,1,.31-.38,2.24,2.24,0,0,1,.38-.31,3.62,3.62,0,0,1,.43-.24,2.52,2.52,0,0,1,2.73.55,2.33,2.33,0,0,1,.31.38,2.93,2.93,0,0,1,.23.43,2.44,2.44,0,0,1,.14.46,2.81,2.81,0,0,1,.05.49A2.53,2.53,0,0,1,26.77,34.64Zm10,0a2.51,2.51,0,0,1-1.77.73,2,2,0,0,1-.49,0,2.43,2.43,0,0,1-.9-.37,2.24,2.24,0,0,1-.38-.31,2.49,2.49,0,0,1-.73-1.77,2.09,2.09,0,0,1,0-.49,2.44,2.44,0,0,1,.14-.46,2.22,2.22,0,0,1,.23-.43,1.91,1.91,0,0,1,.31-.38,2.24,2.24,0,0,1,.38-.31,3.62,3.62,0,0,1,.43-.24,2.52,2.52,0,0,1,2.73.55,2.33,2.33,0,0,1,.31.38,2.93,2.93,0,0,1,.23.43,2.44,2.44,0,0,1,.14.46,2.81,2.81,0,0,1,0,.49A2.53,2.53,0,0,1,36.77,34.64Zm10,0a2.51,2.51,0,0,1-1.77.73,2,2,0,0,1-.49,0,2.43,2.43,0,0,1-.9-.37,2.24,2.24,0,0,1-.38-.31,2.49,2.49,0,0,1-.73-1.77,2.09,2.09,0,0,1,0-.49,2.44,2.44,0,0,1,.14-.46,2.22,2.22,0,0,1,.23-.43,1.91,1.91,0,0,1,.31-.38,2.24,2.24,0,0,1,.38-.31,3.62,3.62,0,0,1,.43-.24,2.52,2.52,0,0,1,2.73.55,2.33,2.33,0,0,1,.31.38,2.93,2.93,0,0,1,.23.43,2.44,2.44,0,0,1,.14.46,2.81,2.81,0,0,1,0,.49A2.53,2.53,0,0,1,46.77,34.64Z" fill="#fff"/>
      </g>
    </g>
  </g>
</svg>

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
                    <field name="odoobot_state" readonly="0" groups="base.group_no_one"
                        attrs="{'invisible': [('share', '=', True)]}"/>
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
                <field name="signature" position="before">
                    <field name="odoobot_state" readonly="0" groups="base.group_no_one"
                        attrs="{'invisible': [('share', '=', True)]}"/>
                </field>
            </data>
        </field>
    </record>
</data></odoo>

```

