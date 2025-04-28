# Odoo Module: mass_mailing_event_sms

Category: Hidden

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
    'name': 'Event Attendees SMS Marketing',
    'category': 'Hidden',
    'version': '1.0',
    'description':
        """
SMS Marketing on event attendees
================================

Bridge module adding UX requirements to ease SMS marketing o, event attendees.
        """,
    'depends': [
        'event',
        'mass_mailing',
        'mass_mailing_event',
        'mass_mailing_sms',
        'sms',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Event(models.Model):
    _inherit = "event.event"

    def action_mass_mailing_attendees(self):
        # Minimal override: set form view being the one mixing sms and mail (not prioritized one)
        action = super(Event, self).action_mass_mailing_attendees()
        action['view_id'] = self.env.ref('mass_mailing_sms.mailing_mailing_view_form_mixed').id
        return action

    def action_invite_contacts(self):
        # Minimal override: set form view being the one mixing sms and mail (not prioritized one)
        action = super(Event, self).action_invite_contacts()
        action['view_id'] = self.env.ref('mass_mailing_sms.mailing_mailing_view_form_mixed').id
        return action

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event

```

