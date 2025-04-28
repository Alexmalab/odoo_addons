# Odoo Module: odoo_referral

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
    'name': "Odoo referral program",
    'summary': """Allow you to refer your friends to Odoo and get rewards""",
    'category': 'Hidden',
    'version': '1.0',
    'depends': ['base', 'web'],
    'data': [
        'views/templates.xml',
    ],
    'qweb': [
        "static/src/xml/systray.xml",
    ],
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: static\src\js\systray.js

```javascript


```

## File: static\src\xml\systray.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates>
   <t t-name="systray_odoo_referral.gift_icon">
   </t>
</templates>

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
   <data>
       <template id="assets_backend" name="systray_odoo_referral_icon" inherit_id="web.assets_backend">
           <xpath expr=".">
               <script type="text/javascript" src="/odoo_referral/static/src/js/systray.js"/>
           </xpath>
       </template>
   </data>
</odoo>

```

