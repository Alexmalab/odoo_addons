# Odoo Module: im_livechat_mail_bot

Category: Productivity/Discuss

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'OdooBot for livechat',
    'version': '1.0',
    'category': 'Productivity/Discuss',
    'summary': 'Add livechat support for OdooBot',
    'website': 'https://www.odoo.com/app/discuss',
    'depends': ['mail_bot', 'im_livechat'],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\discuss.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request, route
from odoo.addons.mail.controllers import discuss


class DiscussController(discuss.DiscussController):
    @route()
    def mail_message_post(self, thread_model, thread_id, post_data, **kwargs):
        if post_data.get("canned_response_ids"):
            request.update_context(canned_response_ids=post_data["canned_response_ids"])
        return super().mail_message_post(thread_model, thread_id, post_data, **kwargs)

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import discuss

```

## File: models\mail_bot.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _


class MailBot(models.AbstractModel):
    _inherit = 'mail.bot'

    def _get_answer(self, record, body, values, command):
        odoobot_state = self.env.user.odoobot_state
        if self._is_bot_in_private_channel(record):
            if odoobot_state == "onboarding_attachement" and values.get("attachment_ids"):
                self.env.user.odoobot_failed = False
                self.env.user.odoobot_state = "onboarding_canned"
                return _("Wonderful! 😇<br/>Try typing <span class=\"o_odoobot_command\">:</span> to use canned responses.")
            elif odoobot_state == "onboarding_canned" and self.env.context.get("canned_response_ids"):
                self.env.user.odoobot_failed = False
                self.env.user.odoobot_state = "idle"
                return _("Good, you can customize canned responses in the live chat application.<br/><br/><b>It's the end of this overview</b>, you can now <b>close this conversation</b> or start the tour again with typing <span class=\"o_odoobot_command\">start the tour</span>. Enjoy discovering Odoo!")
            # repeat question if needed
            elif odoobot_state == 'onboarding_canned' and not self._is_help_requested(body):
                self.env.user.odoobot_failed = True
                return _("Not sure what you are doing. Please, type <span class=\"o_odoobot_command\">:</span> and wait for the propositions. Select one of them and press enter.")
        return super(MailBot, self)._get_answer(record, body, values, command)

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class Users(models.Model):
    _inherit = 'res.users'

    odoobot_state = fields.Selection(selection_add=[
        ('onboarding_canned', 'Onboarding canned'),
    ], ondelete={'onboarding_canned': lambda users: users.write({'odoobot_state': 'disabled'})})

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mail_bot
from . import res_users
```

