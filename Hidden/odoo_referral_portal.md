# Odoo Module: odoo_referral_portal

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': "Odoo referral program bridge with portal (Deprecated)",
    'summary': """Allow you to refer your friends to Odoo and get rewards - Deprecated""",
    'category': 'Hidden',
    'version': '0.1',
    'depends': ['website', 'odoo_referral'],
    'data': [
        'views/referral_template.xml',
    ],
    'auto_install': False, # As deprecated,
    'license': 'LGPL-3',
}

```

## File: views\referral_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <!--
        Yep, this file is empty on purpose, no need to open a PR about it.
        The file (with the entire module) should be dropped on 13.4.
     -->
</odoo>

```

