# Odoo Module: test_html_field_history

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
    'name': 'Test - html_field_history',
    'version': '1.0',
    'category': 'Hidden',
    'depends': ['web_editor'],
    'data': [
        'security/ir.model.access.csv',
    ],
    'license': 'LGPL-3',
}

```

## File: models\model_html_field_history_test.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ModelHtmlFieldHistoryTest(models.Model):
    _description = "Test html_field_history Model"
    _name = "html.field.history.test"
    _inherit = ["html.field.history.mixin"]

    def _get_versioned_fields(self):
        return [
            ModelHtmlFieldHistoryTest.versioned_field_1.name,
            ModelHtmlFieldHistoryTest.versioned_field_2.name,
        ]

    versioned_field_1 = fields.Html(string="vf1")
    versioned_field_2 = fields.Html(string="vf2", sanitize=False)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import model_html_field_history_test

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_resource_test_all,resource.test.all,model_html_field_history_test,base.group_user,1,1,1,1

```

