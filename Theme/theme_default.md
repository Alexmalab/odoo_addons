# Odoo Module: theme_default

Category: Theme

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

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
    'depends': ['website', 'website_theme_install'],
    'data': [
    ],
    'images': [
        'static/description/cover.png',
        'static/description/theme_default_screenshot.jpg',
    ],
    'application': False,
    'license': 'LGPL-3',
}

```

## File: models\theme_default.py

```python
from odoo import models


class ThemeDefault(models.AbstractModel):
    _inherit = 'theme.utils'

    def _theme_default_post_copy(self, mod):
        self.disable_view('website_theme_install.customize_modal')

```

## File: models\__init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import theme_default

```

