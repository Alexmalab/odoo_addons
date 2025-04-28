# Odoo Module: partner_autocomplete_address_extended

Category: Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': "Partner Autocomplete extends Address autocomplete",
    'summary': "Autocomplete extended addresses",
    'description': """
Correct address formating when both partner_autocomplete and base_address_extended modules are installed""",
    'author': "Odoo SA",
    'category': 'Tools',
    'version': '1.0',
    'depends': ['partner_autocomplete', 'base_address_extended'],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from odoo import api, fields, models


class ResPartner(models.Model):
    _name = 'res.partner'
    _inherit = 'res.partner'

    @api.model
    def _split_street_with_params(self, street_raw, street_format=False):
        regex = '((\d+\w* ?(-|\/) ?\d*\w*)|(\d+\w*))'

        street_name = street_raw
        street_number = ''
        street_number2 = ''

        # Try to find number at beginning
        start_regex = re.compile('^' + regex)
        matches = re.search(start_regex, street_raw)
        if matches and matches.group(0):
            street_number = matches.group(0)
            street_name = re.sub(start_regex, '', street_raw, 1)
        else:
            # Try to find number at end
            end_regex = re.compile(regex + '$')
            matches = re.search(end_regex, street_raw)
            if matches and matches.group(0):
                street_number = matches.group(0)
                street_name = re.sub(end_regex, '', street_raw, 1)

        if street_number:
            street_number_split = street_number.split('/')
            if len(street_number_split) > 1:
                street_number2 = street_number_split.pop(-1)
                street_number = '/'.join(street_number_split)

        return {
            'street_name': street_name.strip(),
            'street_number': street_number.strip(),
            'street_number2': street_number2.strip(),
        }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_partner
```

