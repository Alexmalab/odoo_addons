# Odoo Module: account_edi_extended

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
from . import models


def account_edi_block_level(cr, registery):
    ''' The default value for blocking_level is 'error', but without this module,
    the behavior is the same as a blocking_level of 'warning' so we need to set
    all documents in error.
    '''
    from odoo import api, SUPERUSER_ID

    env = api.Environment(cr, SUPERUSER_ID, {})
    env['account.edi.document'].search([('error', '!=', False)]).write({'blocking_level': 'warning'})

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name' : 'Additionnal features for account_edi',
    'description':"""
        This module add features to account_edi to support new Edi formats.
    """,
    'version' : '1.0',
    'category': 'Accounting/Accounting',
    'depends' : ['account_edi'],
    'data': [
        'views/account_edi_document_views.xml',
        'views/account_move_views.xml',
        'views/account_payment_views.xml',
    ],
    'installable': True,
    'application': False,
    'auto_install': False,
    'post_init_hook': 'account_edi_block_level',
    'license': 'LGPL-3',
}

```

## File: models\account_edi_document.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields, _
import logging

_logger = logging.getLogger(__name__)
DEFAULT_BLOCKING_LEVEL = 'warning'  # Keep previous behavior. TODO : when account_edi_extended is merged with account_edi, should be 'error' (document will not be processed again until forced retry or reset to draft)


class AccountEdiDocument(models.Model):
    _inherit = 'account.edi.document'

    blocking_level = fields.Selection(selection=[('info', 'Info'), ('warning', 'Warning'), ('error', 'Error')],
                                     help="Blocks the document current operation depending on the error severity :\n"
                                          "  * Info: the document is not blocked and everything is working as it should.\n"
                                          "  * Warning : there is an error that doesn't prevent the current Electronic Invoicing operation to succeed.\n"
                                          "  * Error : there is an error that blocks the current Electronic Invoicing operation.")


```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields, _


class AccountMove(models.Model):
    _inherit = 'account.move'

    edi_show_abandon_cancel_button = fields.Boolean(
        compute='_compute_edi_show_abandon_cancel_button')
    edi_error_message = fields.Html(compute='_compute_edi_error_message')
    edi_blocking_level = fields.Selection(selection=[('info', 'Info'), ('warning', 'Warning'), ('error', 'Error')], compute='_compute_edi_error_message')

    @api.depends(
        'edi_document_ids',
        'edi_document_ids.state',
        'edi_document_ids.blocking_level',
        'edi_document_ids.edi_format_id',
        'edi_document_ids.edi_format_id.name')
    def _compute_edi_web_services_to_process(self):
        # OVERRIDE to take blocking_level into account
        for move in self:
            to_process = move.edi_document_ids.filtered(lambda d: d.state in ['to_send', 'to_cancel'] and d.blocking_level != 'error')
            format_web_services = to_process.edi_format_id.filtered(lambda f: f._needs_web_services())
            move.edi_web_services_to_process = ', '.join(f.name for f in format_web_services)

    @api.depends(
        'state',
        'edi_document_ids.state',
        'edi_document_ids.attachment_id')
    def _compute_edi_show_abandon_cancel_button(self):
        for move in self:
            move.edi_show_abandon_cancel_button = any(doc.edi_format_id._needs_web_services()
                                                      and doc.state == 'to_cancel'
                                                      and move.is_invoice(include_receipts=True)
                                                      and doc.edi_format_id._is_required_for_invoice(move)
                                                      for doc in move.edi_document_ids)

    @api.depends('edi_error_count', 'edi_document_ids.error', 'edi_document_ids.blocking_level')
    def _compute_edi_error_message(self):
        for move in self:
            if move.edi_error_count == 0:
                move.edi_error_message = None
                move.edi_blocking_level = None
            elif move.edi_error_count == 1:
                error_doc = move.edi_document_ids.filtered(lambda d: d.error)
                move.edi_error_message = error_doc.error
                move.edi_blocking_level = error_doc.blocking_level
            else:
                error_levels = set([doc.blocking_level for doc in move.edi_document_ids])
                if 'error' in error_levels:
                    move.edi_error_message = str(move.edi_error_count) + _(" Electronic invoicing error(s)")
                    move.edi_blocking_level = 'error'
                elif 'warning' in error_levels:
                    move.edi_error_message = str(move.edi_error_count) + _(" Electronic invoicing warning(s)")
                    move.edi_blocking_level = 'warning'
                else:
                    move.edi_error_message = str(move.edi_error_count) + _(" Electronic invoicing info(s)")
                    move.edi_blocking_level = 'info'

    def action_retry_edi_documents_error(self):
        self.edi_document_ids.write({'error': False, 'blocking_level': False})
        self.action_process_edi_web_services()

    def button_abandon_cancel_posted_posted_moves(self):
        '''Cancel the request for cancellation of the EDI.
        '''
        documents = self.env['account.edi.document']
        for move in self:
            is_move_marked = False
            for doc in move.edi_document_ids:
                if doc.state == 'to_cancel' \
                        and move.is_invoice(include_receipts=True) \
                        and doc.edi_format_id._is_required_for_invoice(move):
                    documents |= doc
                    is_move_marked = True
            if is_move_marked:
                move.message_post(body=_("A request for cancellation of the EDI has been called off."))

        documents.write({'state': 'sent'})

```

## File: models\account_payment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class AccountPayment(models.Model):
    _inherit = 'account.payment'

    def action_retry_edi_documents_error(self):
        self.ensure_one()
        return self.move_id.action_retry_edi_documents_error()

```

## File: models\__init__.py

```python
from . import account_edi_document
from . import account_move
from . import account_payment

```

## File: views\account_edi_document_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_tree_account_edi_document_inherit" model="ir.ui.view">
            <field name="name">Account.edi.document.tree.inherit</field>
            <field name="model">account.edi.document</field>
            <field name="inherit_id" ref="account_edi.view_tree_account_edi_document"/>
            <field name="arch" type="xml">
                <xpath expr="//tree" position="inside">
                    <field name="blocking_level" invisible="1" />
                </xpath>
                <xpath expr="//tree" position="attributes">
                    <attribute name="decoration-info">blocking_level == 'info'</attribute>
                    <attribute name="decoration-warning">blocking_level == 'warning'</attribute>
                    <attribute name="decoration-danger">blocking_level == 'error'</attribute>
                </xpath>
                
            </field>
        </record>
    </data>
</odoo>

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="view_move_form_inherit" model="ir.ui.view">
            <field name="name">account.move.form.inherit</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account_edi.view_move_form_inherit" />
            <field name="arch" type="xml">
                <xpath expr="//button[@name='%(account_edi.action_open_edi_documents)d']" position="after">
                    <button name="action_retry_edi_documents_error" type="object" class="oe_link oe_inline" string="Retry" />
                </xpath>
                <xpath expr="//button[@name='button_cancel_posted_moves']" position="after">
                    <field name="edi_show_abandon_cancel_button" invisible="1"/>
                    <button name="button_abandon_cancel_posted_posted_moves"
                            string="Call off EDI Cancellation"
                            type="object"
                            groups="account.group_account_invoice"
                            attrs="{'invisible' : [('edi_show_abandon_cancel_button', '=', False)]}"/>
                </xpath>
                <!-- Nasty xpath to replace the error count warning banner. In master, it will be merged. -->
                <xpath expr="//div[hasclass('alert-warning')]" position="replace">
                    <field name="edi_blocking_level" invisible="1" />
                    <field name="edi_error_count" invisible="1" />
                    <div class="alert alert-danger" role="alert" style="margin-bottom:0px;"
                        attrs="{'invisible': ['|', ('edi_error_count', '=', 0), ('edi_blocking_level', '!=', 'error')]}">
                        <div class="o_row">
                            <field name="edi_error_message" />
                            <button name="%(account_edi.action_open_edi_documents)d" string="⇒ See errors" type="action" class="oe_link" attrs="{'invisible': [('edi_error_count', '=', 1)]}" /> 
                            <button name="action_retry_edi_documents_error" type="object" class="oe_link oe_inline" string="Retry" />
                        </div>
                    </div>
                    <div class="alert alert-warning" role="alert" style="margin-bottom:0px;"
                        attrs="{'invisible': ['|', ('edi_error_count', '=', 0), ('edi_blocking_level', '!=', 'warning')]}">
                        <div class="o_row">
                            <field name="edi_error_message" />
                            <button name="%(account_edi.action_open_edi_documents)d" string="⇒ See errors" type="action" class="oe_link" attrs="{'invisible': [('edi_error_count', '=', 1)]}" /> 
                        </div>
                    </div>
                    <div class="alert alert-info" role="alert" style="margin-bottom:0px;"
                        attrs="{'invisible': ['|', ('edi_error_count', '=', 0), ('edi_blocking_level', '!=', 'info')]}">
                        <div class="o_row">
                            <field name="edi_error_message" />
                            <button name="%(account_edi.action_open_edi_documents)d" string="⇒ See errors" type="action" class="oe_link" attrs="{'invisible': [('edi_error_count', '=', 1)]}" /> 
                        </div>
                    </div>
                </xpath>
                <xpath expr="//field[@name='edi_document_ids']/tree" position="inside">
                    <field name="blocking_level" invisible="1"/>
                </xpath>
                <xpath expr="//button[@name='action_export_xml']" position="attributes">
                    <attribute name="attrs">{'invisible': ['|', ('error', '=', False), ('blocking_level', '=', 'info')]}</attribute>
                </xpath>

            </field>
        </record>
    </data>
</odoo>

```

## File: views\account_payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_payment_form_inherit" model="ir.ui.view">
            <field name="name">account.payment.form.inherit</field>
            <field name="model">account.payment</field>
            <field name="inherit_id" ref="account_edi.view_payment_form_inherit" />
            <field name="arch" type="xml">
                <xpath expr="//button[@name='%(account_edi.action_open_payment_edi_documents)d']" position="after">
                    <button name="action_retry_edi_documents_error" type="object" class="oe_link oe_inline" string="Retry" />
                </xpath>
                <xpath expr="//field[@name='edi_document_ids']/tree" position="inside">
                    <field name="blocking_level" invisible="1"/>
                </xpath>
                <xpath expr="//button[@name='action_export_xml']" position="attributes">
                    <attribute name="attrs">{'invisible': ['|', ('error', '=', False), ('blocking_level', '=', 'info')]}</attribute>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

