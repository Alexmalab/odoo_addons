# Odoo Module: l10n_din5008_stock

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
    'name': 'DIN 5008 - Stock',
    'category': 'Accounting/Localizations',
    'depends': [
        'l10n_din5008',
        'stock',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\stock.py

```python
from odoo import models, fields, _
from odoo.tools import format_date


class StockPicking(models.Model):
    _inherit = 'stock.picking'

    l10n_din5008_addresses = fields.Binary(compute='_compute_l10n_din5008_addresses', exportable=False)

    def _compute_l10n_din5008_addresses(self):
        for record in self:
            record.l10n_din5008_addresses = data = []
            if record.partner_id:
                if record.picking_type_id.code == 'incoming':
                    data.append((_('Vendor Address:'), record.partner_id))
                if record.picking_type_id.code == 'internal':
                    data.append((_('Warehouse Address:'), record.partner_id))
                if record.picking_type_id.code == 'outgoing' and record.move_ids_without_package and record.move_ids_without_package[0].partner_id \
                        and record.move_ids_without_package[0].partner_id.id != record.partner_id.id:
                    data.append((_('Customer Address:'), record.partner_id))

    def check_field_access_rights(self, operation, field_names):
        field_names = super().check_field_access_rights(operation, field_names)
        return [field_name for field_name in field_names if field_name not in {
            'l10n_din5008_addresses',
        }]

```

## File: models\__init__.py

```python
from . import stock

```

