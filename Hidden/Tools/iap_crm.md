# Odoo Module: iap_crm

Category: Hidden/Tools

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
    'name': "IAP / CRM",
    'summary': """Bridge between IAP and CRM""",
    'description': """Bridge between IAP and CRM""",
    'category': 'Hidden/Tools',
    'version': '1.0',
    'depends': [
        'crm',
        'iap_mail',
    ],
    'application': False,
    'installable': True,
    'auto_install': True,
    'data': [
    ],
    'license': 'LGPL-3',
}

```

## File: models\crm_lead.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Lead(models.Model):
    _inherit = 'crm.lead'

    reveal_id = fields.Char(string='Reveal ID', help="Technical ID of reveal request done by IAP.")

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_lead

```

