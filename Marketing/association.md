# Odoo Module: association

Category: Marketing

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
    'name': 'Associations Management',
    'version': '0.1',
    'category': 'Marketing',
    'description': """
This module is to configure modules related to an association.
==============================================================

It installs the profile for associations to manage events, registrations, memberships, 
membership products (schemes).
    """,
    'depends': ['base_setup', 'membership', 'event'],
    'data': ['views/association_views.xml'],
    'demo': [],
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: views\association_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- Top menu item -->
        <menuitem name="Members"
            id="membership.menu_association"
            groups="account.group_account_user"
            sequence="245"/>
        <menuitem name="Configuration" id="menu_event_config" parent="membership.menu_association" sequence="100"/>
</odoo>

```

