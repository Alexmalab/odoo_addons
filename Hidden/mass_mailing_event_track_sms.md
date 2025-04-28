# Odoo Module: mass_mailing_event_track_sms

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
    'name': 'Track Speakers SMS Marketing',
    'category': 'Hidden',
    'version': '1.0',
    'description':
        """
SMS Marketing on event track speakers
=====================================

Bridge module adding UX requirements to ease SMS marketing on event track
speakers..
        """,
    'depends': [
        'mass_mailing',
        'mass_mailing_sms',
        'sms',
        'website_event_track'
    ],
    'data': [
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

    def action_mass_mailing_track_speakers(self):
        # Minimal override: set form view being the one mixing sms and mail (not prioritized one)
        action = super(Event, self).action_mass_mailing_track_speakers()
        action['view_id'] = self.env.ref('mass_mailing_sms.mailing_mailing_view_form_mixed').id
        return action

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event

```

