# Odoo Module: hr_expense_check

Category: Accounting/Expenses

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': "Check Printing in Expenses",
    'summary': """Print amount in words on checks issued for expenses""",
    'category': 'Accounting/Expenses',
    'description': """
        Print amount in words on checks issued for expenses
    """,
    'version': '1.0',
    'depends': ['account_check_printing', 'hr_expense'],
    'auto_install': True,
    'data': [
        'views/payment.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: models\payment.py

```python
# -*- coding: utf-8 -*-

from odoo import models, fields, api


class HrExpenseRegisterPaymentWizard(models.TransientModel):
    _inherit = "hr.expense.sheet.register.payment.wizard"

    check_amount_in_words = fields.Char(string="Amount in Words")
    check_manual_sequencing = fields.Boolean(related='journal_id.check_manual_sequencing', readonly=False)
    # Note: a check_number == 0 means that it will be attributed when the check is printed
    check_number = fields.Char(string="Check Number", readonly=True, copy=False, default=0,
        help="Number of the check corresponding to this payment. If your pre-printed check are not already numbered, "
             "you can manage the numbering in the journal configuration page.")
    payment_method_code_2 = fields.Char(related='payment_method_id.code',
                                      help="Technical field used to adapt the interface to the payment type selected.",
                                      string="Payment Method Code 2",
                                      readonly=True)

    @api.onchange('journal_id')
    def _onchange_journal_id(self):
        if hasattr(super(HrExpenseRegisterPaymentWizard, self), '_onchange_journal_id'):
            super(HrExpenseRegisterPaymentWizard, self)._onchange_journal_id()
        if self.journal_id.check_manual_sequencing:
            self.check_number = self.journal_id.check_sequence_id.number_next_actual

    @api.onchange('amount')
    def _onchange_amount(self):
        if hasattr(super(HrExpenseRegisterPaymentWizard, self), '_onchange_amount'):
            super(HrExpenseRegisterPaymentWizard, self)._onchange_amount()
        self.check_amount_in_words = self.currency_id.amount_to_text(self.amount)

    def _get_payment_vals(self):
        res = super(HrExpenseRegisterPaymentWizard, self)._get_payment_vals()
        if self.payment_method_id == self.env.ref('account_check_printing.account_payment_method_check'):
            res.update({
                'check_amount_in_words': self.check_amount_in_words,
                'check_manual_sequencing': self.check_manual_sequencing,
            })
        return res

```

## File: models\__init__.py

```python
from . import payment

```

## File: views\payment.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<data>
    <record id="hr_expense_register_payment_view_form_check_inherit" model="ir.ui.view">
        <field name="name">hr.expense.sheet.register.payment.wizard.form.check.inherited</field>
        <field name="model">hr.expense.sheet.register.payment.wizard</field>
        <field name="inherit_id" ref="hr_expense.hr_expense_sheet_register_payment_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//div[@name='amount_div']" position="after">
                <field name="check_amount_in_words" attrs="{'invisible': [('payment_method_code_2', '!=', 'check_printing')]}" groups="base.group_no_one"/>
            </xpath>
            <xpath expr="//field[@name='communication']" position="after">
                <field name="payment_method_code_2" invisible="1"/>
                <field name="check_manual_sequencing" invisible="1"/>
                <field name="check_number" attrs="{'invisible': ['|', ('payment_method_code_2', '!=', 'check_printing'), ('check_manual_sequencing', '=', False)]}"/>
            </xpath>
        </field>
    </record>
</data>

```

