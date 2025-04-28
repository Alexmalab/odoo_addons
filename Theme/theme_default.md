# Odoo Module: theme_default

Category: Theme

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Default Theme',
    'description': 'Default website theme',
    'category': 'Theme',
    'sequence': 1000,
    'version': '1.0',
    'depends': ['website'],
    'data': [
        'data/generate_primary_template.xml',
    ],
    'images': [
        'static/description/cover.png',
        'static/description/theme_default_screenshot.jpg',
    ],
    'license': 'LGPL-3',
}

```

## File: data\generate_primary_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- Generate primary snippet templates that are not predefined -->
<function model="ir.module.module" name="_generate_primary_snippet_templates">
    <value eval="[ref('base.module_theme_default')]"/>
</function>

</odoo>

```

