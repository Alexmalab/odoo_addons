# Odoo Module: mass_mailing_crm_sms

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
    'name': 'Mass mailing sms on lead / opportunities',
    'category': 'Hidden',
    'version': '1.0',
    'summary': 'Add lead / opportunities info on mass mailing sms',
    'description': """Mass mailing sms on lead / opportunities""",
    'depends': ['mass_mailing_crm', 'mass_mailing_sms'],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\utm.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class UtmCampaign(models.Model):
    _inherit = 'utm.campaign'

    ab_testing_sms_winner_selection = fields.Selection(selection_add=[('crm_lead_count', 'Leads')])

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import utm

```

