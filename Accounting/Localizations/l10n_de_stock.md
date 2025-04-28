# Odoo Module: l10n_de_stock

Category: Accounting/Localizations

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Germany - Stock',
    'category': 'Accounting/Localizations',
    'depends': [
        'l10n_de',
        'stock',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\stock.py

```python
from odoo import models, fields, api, _
from odoo.tools import format_date


class StockInventory(models.Model):
    _inherit = 'stock.inventory'

    l10n_de_template_data = fields.Binary(compute='_compute_l10n_de_template_data')

    def _compute_l10n_de_template_data(self):
        for record in self:
            record.l10n_de_template_data = data = []
            if record.date:
                data.append((_("Date"), format_date(self.env, record.date)))

```

## File: models\__init__.py

```python
from . import stock

```

