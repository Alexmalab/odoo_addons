# Odoo Module: website_event_jitsi

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Event / Jitsi',
    'category': 'Marketing/Events',
    'sequence': 1002,
    'version': '1.0',
    'summary': 'Event / Jitsi',
    'website': 'https://www.odoo.com/app/events',
    'depends': [
        'website_event',
        'website_jitsi',
    ],
    'data': [
        'views/res_config_settings_views.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\neutralize.sql

```sql
-- disable website_event_jitsi
DELETE FROM ir_config_parameter
WHERE key = 'website_jitsi.jitsi_server_domain';
```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    jitsi_server_domain = fields.Char(
        'Jitsi Server Domain',
        default='meet.jit.si',
        config_parameter='website_jitsi.jitsi_server_domain',
        help='The Jitsi server domain can be customized through the settings to use a different server than the default "meet.jit.si"')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_config_settings

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.website.event.jitsi</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="event.res_config_settings_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//block[@name='events_setting_container']" position="inside">
                <setting>
                    <field name="jitsi_server_domain" placeholder="meet.jit.si"/>
                </setting>
            </xpath>
        </field>
    </record>
</odoo>

```

