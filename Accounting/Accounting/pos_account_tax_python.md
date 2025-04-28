# Odoo Module: pos_account_tax_python

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Allow custom taxes in POS",
    'category': 'Accounting/Accounting',
    'version': '1.0',
    'description': """Add code to manage custom taxes to the POS assets bundle""",
    'depends': ['account_tax_python', 'point_of_sale'],
    'assets': {
        'point_of_sale._assets_pos': [
            'account_tax_python/static/src/helpers/*.js',
        ],
    },
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_tax.py

```python
from odoo import api, models


class AccountTax(models.Model):
    _inherit = 'account.tax'

    @api.model
    def _load_pos_data_fields(self, config_id):
        return super()._load_pos_data_fields(config_id) + ['formula_decoded_info']

```

## File: models\__init__.py

```python
from . import account_tax

```

