# Odoo Module: account_debit_note

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Debit Notes',
    'version': '1.0',
    'category': 'Accounting/Accounting',
    'summary': 'Debit Notes',
    'description': """
In a lot of countries, a debit note is used as an increase of the amounts of an existing invoice 
or in some specific cases to cancel a credit note. 
It is like a regular invoice, but we need to keep track of the link with the original invoice.  
The wizard used is similar as the one for the credit note.
    """,
    'depends': ['account'],
    'data': [
        'wizard/account_debit_note_view.xml',
        'views/account_move_view.xml',
        'security/ir.model.access.csv',
    ],
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, _

class AccountMove(models.Model):
    _inherit = "account.move"

    debit_origin_id = fields.Many2one('account.move', 'Original Invoice Debited', readonly=True, copy=False)
    debit_note_ids = fields.One2many('account.move', 'debit_origin_id', 'Debit Notes',
                                     help="The debit notes created for this invoice")
    debit_note_count = fields.Integer('Number of Debit Notes', compute='_compute_debit_count')

    @api.depends('debit_note_ids')
    def _compute_debit_count(self):
        debit_data = self.env['account.move'].read_group([('debit_origin_id', 'in', self.ids)],
                                                        ['debit_origin_id'], ['debit_origin_id'])
        data_map = {datum['debit_origin_id'][0]: datum['debit_origin_id_count'] for datum in debit_data}
        for inv in self:
            inv.debit_note_count = data_map.get(inv.id, 0.0)

    def action_view_debit_notes(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'name': _('Debit Notes'),
            'res_model': 'account.move',
            'view_mode': 'tree,form',
            'domain': [('debit_origin_id', '=', self.id)],
        }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
"access_account_debit_note_user","account_debit_note_group_invoice","model_account_debit_note","account.group_account_invoice",1,1,1,0

```

## File: views\account_move_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_move_form_debit" model="ir.ui.view">
        <field name="name">account.move.form.debit</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <button name="action_reverse" position="after">
                <field name="debit_origin_id" invisible="1"/>
                <button name="%(action_view_account_move_debit)d" string='Add Debit Note'
                                type='action' groups="account.group_account_invoice"
                                attrs="{'invisible': ['|', '|', ('debit_origin_id', '!=', False),
                                        ('move_type', 'not in', ('out_invoice', 'in_invoice', 'out_refund', 'in_refund')), ('state', '!=', 'posted')]}"/>
            </button>
            <div class="oe_button_box" position="inside">
                <button type="object" class="oe_stat_button" name="action_view_debit_notes" icon="fa-plus" attrs="{'invisible': [('debit_note_count', '=', 0)]}">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_value"><field name="debit_note_count"/></span>
                        <span class="o_stat_text">Debit Notes</span>
                    </div>
                </button>
            </div>
            <field name="invoice_origin" position="after">
                <field name="debit_origin_id" attrs="{'invisible': [('debit_origin_id', '=', False)]}"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: wizard\account_debit_note.py

```python
# -*- coding: utf-8 -*-
from odoo import models, fields, api
from odoo.tools.translate import _
from odoo.exceptions import UserError


class AccountDebitNote(models.TransientModel):
    """
    Add Debit Note wizard: when you want to correct an invoice with a positive amount.
    Opposite of a Credit Note, but different from a regular invoice as you need the link to the original invoice.
    In some cases, also used to cancel Credit Notes
    """
    _name = 'account.debit.note'
    _description = 'Add Debit Note wizard'

    move_ids = fields.Many2many('account.move', 'account_move_debit_move', 'debit_id', 'move_id',
                                domain=[('state', '=', 'posted')])
    date = fields.Date(string='Debit Note Date', default=fields.Date.context_today, required=True)
    reason = fields.Char(string='Reason')
    journal_id = fields.Many2one('account.journal', string='Use Specific Journal',
                                 help='If empty, uses the journal of the journal entry to be debited.')
    copy_lines = fields.Boolean("Copy Lines",
                                help="In case you need to do corrections for every line, it can be in handy to copy them.  "
                                     "We won't copy them for debit notes from credit notes. ")
    # computed fields
    move_type = fields.Char(compute="_compute_from_moves")
    journal_type = fields.Char(compute="_compute_from_moves")
    country_code = fields.Char(related='move_ids.company_id.country_id.code')

    @api.model
    def default_get(self, fields):
        res = super(AccountDebitNote, self).default_get(fields)
        move_ids = self.env['account.move'].browse(self.env.context['active_ids']) if self.env.context.get('active_model') == 'account.move' else self.env['account.move']
        if any(move.state != "posted" for move in move_ids):
            raise UserError(_('You can only debit posted moves.'))
        res['move_ids'] = [(6, 0, move_ids.ids)]
        return res

    @api.depends('move_ids')
    def _compute_from_moves(self):
        for record in self:
            move_ids = record.move_ids
            record.move_type = move_ids[0].move_type if len(move_ids) == 1 or not any(m.move_type != move_ids[0].move_type for m in move_ids) else False
            record.journal_type = record.move_type in ['in_refund', 'in_invoice'] and 'purchase' or 'sale'

    def _prepare_default_values(self, move):
        if move.move_type in ('in_refund', 'out_refund'):
            type = 'in_invoice' if move.move_type == 'in_refund' else 'out_invoice'
        else:
            type = move.move_type
        default_values = {
                'ref': '%s, %s' % (move.name, self.reason) if self.reason else move.name,
                'date': self.date or move.date,
                'invoice_date': move.is_invoice(include_receipts=True) and (self.date or move.date) or False,
                'journal_id': self.journal_id and self.journal_id.id or move.journal_id.id,
                'invoice_payment_term_id': None,
                'debit_origin_id': move.id,
                'move_type': type,
            }
        if not self.copy_lines or move.move_type in [('in_refund', 'out_refund')]:
            default_values['line_ids'] = [(5, 0, 0)]
        return default_values

    def create_debit(self):
        self.ensure_one()
        new_moves = self.env['account.move']
        for move in self.move_ids.with_context(include_business_fields=True): #copy sale/purchase links
            default_values = self._prepare_default_values(move)
            new_move = move.copy(default=default_values)
            move_msg = _(
                "This debit note was created from:") + " <a href=# data-oe-model=account.move data-oe-id=%d>%s</a>" % (
                       move.id, move.name)
            new_move.message_post(body=move_msg)
            new_moves |= new_move

        action = {
            'name': _('Debit Notes'),
            'type': 'ir.actions.act_window',
            'res_model': 'account.move',
            'context': {'default_move_type': default_values['move_type']},
        }
        if len(new_moves) == 1:
            action.update({
                'view_mode': 'form',
                'res_id': new_moves.id,
            })
        else:
            action.update({
                'view_mode': 'tree,form',
                'domain': [('id', 'in', new_moves.ids)],
            })
        return action

```

## File: wizard\account_debit_note_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_account_debit_note" model="ir.ui.view">
            <field name="name">account.debit.note.form</field>
            <field name="model">account.debit.note</field>
            <field name="arch" type="xml">
                <form string="Create Debit Note">
                    <field name="move_type" invisible="1"/>
                    <field name="journal_type" invisible="1"/>
                    <field name="move_ids" invisible="1"/>
                    <group>
                         <group>
                             <field name="reason"/>
                             <field name="date" string="Debit Note Date"/>
                             <field name="copy_lines" attrs="{'invisible': [('move_type', 'in', ['in_refund', 'out_refund'])]}"/>
                         </group>
                         <group>
                             <field name="journal_id" domain="[('type', '=', journal_type)]"/>
                         </group>
                    </group>
                    <footer>
                        <button string='Create Debit Note' name="create_debit" type="object" class="btn-primary" data-hotkey="q"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z"/>
                    </footer>
               </form>
            </field>
        </record>

        <record id="action_view_account_move_debit" model="ir.actions.act_window">
            <field name="name">Create Debit Note</field>
            <field name="res_model">account.debit.note</field>
            <field name="view_mode">tree,form</field>
            <field name="view_id" ref="view_account_debit_note"/>
            <field name="target">new</field>
            <field name="binding_model_id" ref="account.model_account_move" />
            <field name="binding_view_types">list</field>
        </record>
    </data>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_debit_note
```

