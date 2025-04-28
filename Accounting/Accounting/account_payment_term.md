# Odoo Module: account_payment_term

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Payment Term - Days end of month on the',
    'version': '1.0',
    'category': 'Accounting/Accounting',
    'description': """
Bridge module to add a new payment term - Days end of month on the
""",
    'website': 'https://www.odoo.com/app/invoicing',
    'depends': ['account'],
    'data': [
        'views/account_payment_term_views.xml',
    ],
    'demo': [
        'demo/account_payment_term_demo.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'OEEL-1',
}

```

## File: models\account_payment_term.py

```python
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError
from odoo.tools import date_utils


class AccountPaymentTermLine(models.Model):
    _inherit = "account.payment.term.line"

    delay_type = fields.Selection(
        selection_add=[('days_end_of_month_on_the', 'Days end of month on the')],
        ondelete={'days_end_of_month_on_the': 'set default'}
    )
    display_days_next_month = fields.Boolean(compute='_compute_display_days_next_month')
    days_next_month = fields.Char(
        string='Days on the next month',
        readonly=False,
        store=True,
        default='10',
        size=2,
    )

    @api.constrains('days_next_month')
    def _check_valid_char_value(self):
        for record in self:
            if record.days_next_month and record.days_next_month.isnumeric():
                if not (0 <= int(record.days_next_month) <= 31):
                    raise ValidationError(_('The days added must be between 0 and 31.'))
            else:
                raise ValidationError(_('The days added must be a number and has to be between 0 and 31.'))

    @api.depends('delay_type')
    def _compute_display_days_next_month(self):
        for record in self:
            record.display_days_next_month = record.delay_type == 'days_end_of_month_on_the'

    def _get_due_date(self, date_ref):
        res = super()._get_due_date(date_ref)

        due_date = fields.Date.from_string(date_ref) or fields.Date.today()
        if self.delay_type == 'days_end_of_month_on_the':
            try:
                days_next_month = int(self.days_next_month)
            except ValueError:
                days_next_month = 1

            if not days_next_month:
                return date_utils.end_of(due_date + relativedelta(days=self.nb_days), 'month')

            return due_date + relativedelta(days=self.nb_days) + relativedelta(months=1, day=days_next_month)
        return res

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_payment_term

```

## File: views\account_payment_term_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="account_payment_term_form" model="ir.ui.view">
            <field name="model">account.payment.term</field>
            <field name="name">account.payment.term.form</field>
            <field name="inherit_id" ref="account.view_payment_term_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='line_ids']/tree['delay_type']" position="inside">
                    <field name="display_days_next_month" column_invisible="1"/>
                    <field name="days_next_month" width="30px" nolabel="1" invisible="not display_days_next_month"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

