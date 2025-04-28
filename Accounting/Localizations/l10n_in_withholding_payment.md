# Odoo Module: l10n_in_withholding_payment

Category: Accounting/Localizations

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models
from . import wizard

```

## File: __manifest__.py

```python
{
    'name': 'Indian - TDS For Payment',
    'version': '1.0',
    'description': """
        Support for Indian TDS (Tax Deducted at Source) for Payment.
    """,
    'category': 'Accounting/Localizations',
    'depends': ['l10n_in_withholding'],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_move.py

```python
from odoo import models, fields


class AccountMove(models.Model):
    _inherit = "account.move"

    l10n_in_withholding_ref_payment_id = fields.Many2one(
        comodel_name='account.payment',
        string="Indian TDS Ref Payment",
        readonly=True,
        copy=False,
        help="Reference Payment for withholding entry",
    )

```

## File: models\account_payment.py

```python
from odoo import models, fields


class AccountPayment(models.Model):
    _inherit = "account.payment"

    l10n_in_withhold_move_ids = fields.One2many(
        'account.move', 'l10n_in_withholding_ref_payment_id',
        string="Indian Payment TDS Entries",
        related=False
    )
    l10n_in_total_withholding_amount = fields.Monetary(compute='_compute_l10n_in_total_withholding_amount', related=False)

    def _compute_l10n_in_total_withholding_amount(self):
        for payment in self:
            payment.l10n_in_total_withholding_amount = sum(payment.l10n_in_withhold_move_ids.filtered(
                lambda m: m.state == 'posted').l10n_in_withholding_line_ids.mapped('l10n_in_withhold_tax_amount'))

```

## File: models\__init__.py

```python
from . import account_move
from . import account_payment

```

## File: wizard\l10n_in_withhold_wizard.py

```python
from odoo import models


class L10nInWithholdWizard(models.TransientModel):
    _inherit = 'l10n_in.withhold.wizard'

    def _prepare_withhold_header(self):
        res = super()._prepare_withhold_header()
        if self.related_payment_id:
            res['l10n_in_withholding_ref_payment_id'] = self.related_payment_id.id
        return res

```

## File: wizard\__init__.py

```python
from . import l10n_in_withhold_wizard

```

