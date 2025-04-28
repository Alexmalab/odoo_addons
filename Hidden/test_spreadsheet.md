# Odoo Module: test_spreadsheet

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Spreadsheet Test',
    'version': '1.0',
    'category': 'Hidden',
    'summary': 'Spreadsheet Test, mainly to test the mixin behavior',
    'description': """This module contains tests related to spreadsheet.
    The modules exposes some mixin that are only implemented in other functional modules.
    When trying to test a global behavior of the mixin, it makes no sense to test it in
    each module implementing the mixin but rather test a dummy implementation of the later,
    hence the need for this test module.
    """,
    'depends': ['spreadsheet'],
    'license': 'LGPL-3',
    'data': ['security/ir.model.access.csv'],
}

```

## File: models\spreadsheet_mixin_test.py

```python
from odoo import models


class SpreadsheetDummy(models.Model):
    """ A very simple model only inheriting from spreadsheet.mixin to test
    its model functioning."""
    _description = 'Dummy Spreadsheet'
    _name = 'spreadsheet.test'
    _inherit = ['spreadsheet.mixin']

```

## File: models\__init__.py

```python
from . import spreadsheet_mixin_test

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_spreadsheet_test_all,access.spreadsheet.test.all,model_spreadsheet_test,base.group_user,1,1,1,1

```

