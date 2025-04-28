# Odoo Module: website_event_booth_sale_exhibitor

Category: Marketing/Events

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

{
    'name': 'Booths Sale/Exhibitors Bridge',
    'category': 'Marketing/Events',
    'version': '1.0',
    'summary': 'Bridge module between website_event_booth_exhibitor and website_event_booth_sale.',
    'depends': ['website_event_exhibitor', 'website_event_booth_sale'],
    'auto_install': True,
    'assets': {
        'web.assets_tests': [
            'website_event_booth_sale_exhibitor/static/tests/tours/website_event_booth_sale_exhibitor.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\event_booth_registration.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class EventBoothRegistration(models.Model):
    _inherit = 'event.booth.registration'

    sponsor_name = fields.Char(string='Sponsor Name')
    sponsor_email = fields.Char(string='Sponsor Email')
    sponsor_mobile = fields.Char(string='Sponsor Mobile')
    sponsor_phone = fields.Char(string='Sponsor Phone')
    sponsor_subtitle = fields.Char(string='Sponsor Slogan')
    sponsor_website_description = fields.Html(string='Sponsor Description', sanitize_overridable=True,)
    sponsor_image_512 = fields.Image(string='Sponsor Logo')

    def _get_fields_for_booth_confirmation(self):
        return super(EventBoothRegistration, self)._get_fields_for_booth_confirmation() + \
               ['sponsor_name', 'sponsor_email', 'sponsor_mobile', 'sponsor_phone', 'sponsor_subtitle',
                'sponsor_website_description', 'sponsor_image_512']

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_booth_registration

```

