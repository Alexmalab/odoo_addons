# Odoo Module: l10n_din5008_repair

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
    'name': 'DIN 5008 - Repair',
    'category': 'Accounting/Localizations',
    'depends': [
        'l10n_din5008',
        'repair',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\repair.py

```python
from odoo import models, fields, _
from odoo.tools import format_date


class RepairOrder(models.Model):
    _inherit = 'repair.order'

    l10n_din5008_template_data = fields.Binary(compute='_compute_l10n_din5008_template_data')
    l10n_din5008_document_title = fields.Char(compute='_compute_l10n_din5008_document_title')

    def _compute_l10n_din5008_template_data(self):
        for record in self:
            record.l10n_din5008_template_data = data = []
            if record.product_id:
                data.append((_("Product to Repair"), record.product_id.name))
            if record.lot_id:
                data.append((_("Lot/Serial Number"), record.lot_id.name))
            if record.guarantee_limit:
                data.append((_("Warranty"), format_date(self.env, record.guarantee_limit)))
            data.append((_("Printing Date"), format_date(self.env, fields.Date.today())))

    def _compute_l10n_din5008_document_title(self):
        for record in self:
            if record.state == 'draft':
                record.l10n_din5008_document_title = _("Repair Quotation")
            else:
                record.l10n_din5008_document_title = _("Repair Order")

```

## File: models\__init__.py

```python
from . import repair

```

