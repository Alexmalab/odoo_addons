# Odoo Module: account_bank_statement_import

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: account_bank_statement_import.py

```python
# -*- coding: utf-8 -*-

import base64

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.addons.base.models.res_bank import sanitize_account_number

import logging
_logger = logging.getLogger(__name__)


class AccountBankStatementLine(models.Model):
    _inherit = "account.bank.statement.line"

    # Ensure transactions can be imported only once (if the import format provides unique transaction ids)
    unique_import_id = fields.Char(string='Import ID', readonly=True, copy=False)

    _sql_constraints = [
        ('unique_import_id', 'unique (unique_import_id)', 'A bank account transactions can be imported only once !')
    ]


class AccountBankStatementImport(models.TransientModel):
    _name = 'account.bank.statement.import'
    _description = 'Import Bank Statement'

    attachment_ids = fields.Many2many('ir.attachment', string='Files', required=True, help='Get you bank statements in electronic format from your bank and select them here.')

    def import_file(self):
        """ Process the file chosen in the wizard, create bank statement(s) and go to reconciliation. """
        self.ensure_one()
        statement_line_ids_all = []
        notifications_all = []
        # Let the appropriate implementation module parse the file and return the required data
        # The active_id is passed in context in case an implementation module requires information about the wizard state (see QIF)
        for data_file in self.attachment_ids:
            currency_code, account_number, stmts_vals = self.with_context(active_id=self.ids[0])._parse_file(base64.b64decode(data_file.datas))
            # Check raw data
            self._check_parsed_data(stmts_vals, account_number)
            # Try to find the currency and journal in odoo
            currency, journal = self._find_additional_data(currency_code, account_number)
            # If no journal found, ask the user about creating one
            if not journal:
                # The active_id is passed in context so the wizard can call import_file again once the journal is created
                return self.with_context(active_id=self.ids[0])._journal_creation_wizard(currency, account_number)
            if not journal.default_debit_account_id or not journal.default_credit_account_id:
                raise UserError(_('You have to set a Default Debit Account and a Default Credit Account for the journal: %s') % (journal.name,))
            # Prepare statement data to be used for bank statements creation
            stmts_vals = self._complete_stmts_vals(stmts_vals, journal, account_number)
            # Create the bank statements
            statement_line_ids, notifications = self._create_bank_statements(stmts_vals)
            statement_line_ids_all.extend(statement_line_ids)
            notifications_all.extend(notifications)
            # Now that the import worked out, set it as the bank_statements_source of the journal
            if journal.bank_statements_source != 'file_import':
                # Use sudo() because only 'account.group_account_manager'
                # has write access on 'account.journal', but 'account.group_account_user'
                # must be able to import bank statement files
                journal.sudo().bank_statements_source = 'file_import'
        # Finally dispatch to reconciliation interface
        return {
            'type': 'ir.actions.client',
            'tag': 'bank_statement_reconciliation_view',
            'context': {'statement_line_ids': statement_line_ids_all,
                        'company_ids': self.env.user.company_ids.ids,
                        'notifications': notifications_all,
            },
        }

    def _journal_creation_wizard(self, currency, account_number):
        """ Calls a wizard that allows the user to carry on with journal creation """
        return {
            'name': _('Journal Creation'),
            'type': 'ir.actions.act_window',
            'res_model': 'account.bank.statement.import.journal.creation',
            'view_mode': 'form',
            'target': 'new',
            'context': {
                'statement_import_transient_id': self.env.context['active_id'],
                'default_bank_acc_number': account_number,
                'default_name': _('Bank') + ' ' + account_number,
                'default_currency_id': currency and currency.id or False,
                'default_type': 'bank',
            }
        }

    def _parse_file(self, data_file):
        """ Each module adding a file support must extends this method. It processes the file if it can, returns super otherwise, resulting in a chain of responsability.
            This method parses the given file and returns the data required by the bank statement import process, as specified below.
            rtype: triplet (if a value can't be retrieved, use None)
                - currency code: string (e.g: 'EUR')
                    The ISO 4217 currency code, case insensitive
                - account number: string (e.g: 'BE1234567890')
                    The number of the bank account which the statement belongs to
                - bank statements data: list of dict containing (optional items marked by o) :
                    - 'name': string (e.g: '000000123')
                    - 'date': date (e.g: 2013-06-26)
                    -o 'balance_start': float (e.g: 8368.56)
                    -o 'balance_end_real': float (e.g: 8888.88)
                    - 'transactions': list of dict containing :
                        - 'name': string (e.g: 'KBC-INVESTERINGSKREDIET 787-5562831-01')
                        - 'date': date
                        - 'amount': float
                        - 'unique_import_id': string
                        -o 'account_number': string
                            Will be used to find/create the res.partner.bank in odoo
                        -o 'note': string
                        -o 'partner_name': string
                        -o 'ref': string
        """
        raise UserError(_('Could not make sense of the given file.\nDid you install the module to support this type of file ?'))

    def _check_parsed_data(self, stmts_vals, account_number):
        """ Basic and structural verifications """
        extra_msg = _('If it contains transactions for more than one account, it must be imported on each of them.')
        if len(stmts_vals) == 0:
            raise UserError(
                _('This file doesn\'t contain any statement for account %s.') % (account_number,)
                + '\n' + extra_msg
            )

        no_st_line = True
        for vals in stmts_vals:
            if vals['transactions'] and len(vals['transactions']) > 0:
                no_st_line = False
                break
        if no_st_line:
            raise UserError(
                _('This file doesn\'t contain any transaction for account %s.') % (account_number,)
                + '\n' + extra_msg
            )

    def _check_journal_bank_account(self, journal, account_number):
        # Needed for CH to accommodate for non-unique account numbers
        sanitized_acc_number = journal.bank_account_id.sanitized_acc_number
        if " " in sanitized_acc_number:
            sanitized_acc_number = sanitized_acc_number.split(" ")[0]
        # Needed for BNP France
        if len(sanitized_acc_number) == 27 and len(account_number) == 11 and sanitized_acc_number[:2].upper() == "FR":
            return sanitized_acc_number[14:-2] == account_number
        return sanitized_acc_number == account_number

    def _find_additional_data(self, currency_code, account_number):
        """ Look for a res.currency and account.journal using values extracted from the
            statement and make sure it's consistent.
        """
        company_currency = self.env.company.currency_id
        journal_obj = self.env['account.journal']
        currency = None
        sanitized_account_number = sanitize_account_number(account_number)

        if currency_code:
            currency = self.env['res.currency'].search([('name', '=ilike', currency_code)], limit=1)
            if not currency:
                raise UserError(_("No currency found matching '%s'.") % currency_code)
            if currency == company_currency:
                currency = False

        journal = journal_obj.browse(self.env.context.get('journal_id', []))
        if account_number:
            # No bank account on the journal : create one from the account number of the statement
            if journal and not journal.bank_account_id:
                journal.set_bank_account(account_number)
            # No journal passed to the wizard : try to find one using the account number of the statement
            elif not journal:
                journal = journal_obj.search([('bank_account_id.sanitized_acc_number', '=', sanitized_account_number)])
            # Already a bank account on the journal : check it's the same as on the statement
            else:
                if not self._check_journal_bank_account(journal, sanitized_account_number):
                    raise UserError(_('The account of this statement (%s) is not the same as the journal (%s).') % (account_number, journal.bank_account_id.acc_number))

        # If importing into an existing journal, its currency must be the same as the bank statement
        if journal:
            journal_currency = journal.currency_id or journal.company_id.currency_id
            if currency is None:
                currency = journal_currency
            if currency and currency != journal_currency:
                statement_cur_code = not currency and company_currency.name or currency.name
                journal_cur_code = not journal_currency and company_currency.name or journal_currency.name
                raise UserError(_('The currency of the bank statement (%s) is not the same as the currency of the journal (%s).') % (statement_cur_code, journal_cur_code))

        # If we couldn't find / can't create a journal, everything is lost
        if not journal and not account_number:
            raise UserError(_('Cannot find in which journal import this statement. Please manually select a journal.'))

        return currency, journal

    def _complete_stmts_vals(self, stmts_vals, journal, account_number):
        for st_vals in stmts_vals:
            st_vals['journal_id'] = journal.id
            if not st_vals.get('reference'):
                st_vals['reference'] = " ".join(self.attachment_ids.mapped('name'))
            if st_vals.get('number'):
                #build the full name like BNK/2016/00135 by just giving the number '135'
                st_vals['name'] = journal.sequence_id.with_context(ir_sequence_date=st_vals.get('date')).get_next_char(st_vals['number'])
                del(st_vals['number'])
            for line_vals in st_vals['transactions']:
                unique_import_id = line_vals.get('unique_import_id')
                if unique_import_id:
                    sanitized_account_number = sanitize_account_number(account_number)
                    line_vals['unique_import_id'] = (sanitized_account_number and sanitized_account_number + '-' or '') + str(journal.id) + '-' + unique_import_id

                if not line_vals.get('bank_account_id'):
                    # Find the partner and his bank account or create the bank account. The partner selected during the
                    # reconciliation process will be linked to the bank when the statement is closed.
                    identifying_string = line_vals.get('account_number')
                    if identifying_string:
                        partner_bank = self.env['res.partner.bank']\
                            .search([('acc_number', '=', identifying_string), ('company_id', 'in', (False, journal.company_id.id))], limit=1)
                        if partner_bank:
                            line_vals['bank_account_id'] = partner_bank.id
                            line_vals['partner_id'] = partner_bank.partner_id.id
        return stmts_vals

    def _create_bank_statements(self, stmts_vals):
        """ Create new bank statements from imported values, filtering out already imported transactions, and returns data used by the reconciliation widget """
        BankStatement = self.env['account.bank.statement']
        BankStatementLine = self.env['account.bank.statement.line']

        # Filter out already imported transactions and create statements
        statement_line_ids = []
        ignored_statement_lines_import_ids = []
        for st_vals in stmts_vals:
            filtered_st_lines = []
            for line_vals in st_vals['transactions']:
                if 'unique_import_id' not in line_vals \
                   or not line_vals['unique_import_id'] \
                   or not bool(BankStatementLine.sudo().search([('unique_import_id', '=', line_vals['unique_import_id'])], limit=1)):
                    filtered_st_lines.append(line_vals)
                else:
                    ignored_statement_lines_import_ids.append(line_vals['unique_import_id'])
                    if 'balance_start' in st_vals:
                        st_vals['balance_start'] += float(line_vals['amount'])

            if len(filtered_st_lines) > 0:
                # Remove values that won't be used to create records
                st_vals.pop('transactions', None)
                # Create the statement
                st_vals['line_ids'] = [[0, False, line] for line in filtered_st_lines]
                statement_line_ids.extend(BankStatement.create(st_vals).line_ids.ids)
        if len(statement_line_ids) == 0:
            raise UserError(_('You already have imported that file.'))

        # Prepare import feedback
        notifications = []
        num_ignored = len(ignored_statement_lines_import_ids)
        if num_ignored > 0:
            notifications += [{
                'type': 'warning',
                'message': _("%d transactions had already been imported and were ignored.") % num_ignored if num_ignored > 1 else _("1 transaction had already been imported and was ignored."),
                'details': {
                    'name': _('Already imported items'),
                    'model': 'account.bank.statement.line',
                    'ids': BankStatementLine.search([('unique_import_id', 'in', ignored_statement_lines_import_ids)]).ids
                }
            }]
        return statement_line_ids, notifications

```

## File: account_bank_statement_import_view.xml

```xml
<?xml version="1.0" ?>
<odoo>

        <record id="account_bank_statement_import_view" model="ir.ui.view">
            <field name="name">Upload Bank Statements</field>
            <field name="model">account.bank.statement.import</field>
            <field name="priority">1</field>
            <field name="arch" type="xml">
                <form string="Upload Bank Statements">
                    <h2>You can upload your bank statement using:</h2>
                    <ul id="statement_format">

                    </ul>
                    <field name="attachment_ids"  widget="many2many_binary" colspan="2" string="Select Files" nolabel="1"/>
                    <footer>
                        <button name="import_file" string="Upload" type="object" class="btn-primary" />
                        <button string="Cancel" class="btn-secondary" special="cancel"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="install_more_import_formats_action" model="ir.actions.act_window">
            <field name="name">Install Import Format</field>
            <field name="res_model">ir.module.module</field>
            <field name="view_mode">kanban,tree,form</field>
            <field name="context" eval="{'search_default_name': 'account_bank_statement_import'}"/>
            <field name="search_view_id" ref="base.view_module_filter"/>
        </record>

        <record id="action_account_bank_statement_import" model="ir.actions.act_window">
            <field name="name">Upload</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">account.bank.statement.import</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
            <field name="view_id" ref="account_bank_statement_import_view"/>
        </record>

    <record id="journal_dashboard_view_inherit" model="ir.ui.view">
        <field name="name">account.journal.dashboard.kanban.inherit</field>
        <field name="model">account.journal</field>
        <field name="inherit_id" ref="account.account_journal_dashboard_kanban_view"/>
        <field name="arch" type="xml">
            <xpath expr='//span[@name="button_import_placeholder"]' position='inside'>
                <span>or <a type="object" name="import_statement">Import</a></span>
            </xpath>
            <xpath expr='//div[@name="bank_cash_commands"]' position="before">
                <div t-if="journal_type == 'bank'">
                    <a type="object" name="import_statement">Import Statement</a>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: account_import_tip_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>

    </data>
</odoo>

```

## File: account_journal.py

```python
# -*- coding: utf-8 -*-

from odoo import models, api, _


class AccountJournal(models.Model):
    _inherit = "account.journal"

    def _get_bank_statements_available_import_formats(self):
        """ Returns a list of strings representing the supported import formats.
        """
        return []

    def __get_bank_statements_available_sources(self):
        rslt = super(AccountJournal, self).__get_bank_statements_available_sources()
        formats_list = self._get_bank_statements_available_import_formats()
        if formats_list:
            formats_list.sort()
            import_formats_str = ', '.join(formats_list)
            rslt.append(("file_import", _("Import") + "(" + import_formats_str + ")"))
        return rslt

    def import_statement(self):
        """return action to import bank/cash statements. This button should be called only on journals with type =='bank'"""
        action_name = 'action_account_bank_statement_import'
        [action] = self.env.ref('account_bank_statement_import.%s' % action_name).read()
        # Note: this drops action['context'], which is a dict stored as a string, which is not easy to update
        action.update({'context': (u"{'journal_id': " + str(self.id) + u"}")})
        return action

```

## File: __init__.py

```python
# -*- encoding: utf-8 -*-

from . import account_bank_statement_import
from . import account_journal

from . import wizard

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
{
    'name': 'Account Bank Statement Import',
    'category': 'Accounting/Accounting',
    'version': '1.0',
    'depends': ['account'],
    'description': """Generic Wizard to Import Bank Statements.

(This module does not include any type of import format.)

OFX and QIF imports are available in Enterprise version.""",
    'data': [
        'account_bank_statement_import_view.xml',
        'account_import_tip_data.xml',
        'wizard/journal_creation.xml',
        'views/account_bank_statement_import_templates.xml',
    ],
    'demo': [
        'demo/partner_bank.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: static\description\icon_src.svg

```svg
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<svg
   xmlns:dc="http://purl.org/dc/elements/1.1/"
   xmlns:cc="http://creativecommons.org/ns#"
   xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
   xmlns:svg="http://www.w3.org/2000/svg"
   xmlns="http://www.w3.org/2000/svg"
   xmlns:sodipodi="http://sodipodi.sourceforge.net/DTD/sodipodi-0.dtd"
   xmlns:inkscape="http://www.inkscape.org/namespaces/inkscape"
   enable-background="new 0 0 100 100"
   height="100px"
   id="Layer_1"
   version="1.1"
   viewBox="0 0 100 100"
   width="100px"
   xml:space="preserve"
   inkscape:version="0.48.2 r9819"
   sodipodi:docname="1409271720_Noun_Project_100Icon_10px_grid-17.svg"
   inkscape:export-filename="/Users/arthurmaniet/Desktop/icon.png"
   inkscape:export-xdpi="115.2"
   inkscape:export-ydpi="115.2"><metadata
     id="metadata9"><rdf:RDF><cc:Work
         rdf:about=""><dc:format>image/svg+xml</dc:format><dc:type
           rdf:resource="http://purl.org/dc/dcmitype/StillImage" /><dc:title></dc:title></cc:Work></rdf:RDF></metadata><defs
     id="defs7" /><sodipodi:namedview
     pagecolor="#ffffff"
     bordercolor="#666666"
     borderopacity="1"
     objecttolerance="10"
     gridtolerance="10"
     guidetolerance="10"
     inkscape:pageopacity="0"
     inkscape:pageshadow="2"
     inkscape:window-width="1733"
     inkscape:window-height="1001"
     id="namedview5"
     showgrid="false"
     inkscape:zoom="11.62"
     inkscape:cx="21.99675"
     inkscape:cy="56.127828"
     inkscape:window-x="76"
     inkscape:window-y="0"
     inkscape:window-maximized="0"
     inkscape:current-layer="Layer_1" /><path
     d="M79.043,31.615l-5.742,5.742V13h-58v74h58V48.67l11.398-11.399L79.043,31.615z M71.301,39.357L50.758,59.898l-1.414,4.242  l-1.414,4.244l8.486-2.828L71.301,50.67V85h-54V15h54V39.357z M54.564,65.119l-3.182,1.06l-1.248-1.248l1.061-3.182l3.1,3.099  L54.564,65.119z"
     id="path3" /><text
     xml:space="preserve"
     style="font-size:40px;font-style:normal;font-weight:normal;line-height:125%;letter-spacing:0px;word-spacing:0px;fill:#000000;fill-opacity:1;stroke:none;font-family:Sans"
     x="18.006462"
     y="17.887218"
     id="text2986"
     sodipodi:linespacing="125%"><tspan
       sodipodi:role="line"
       x="18.006462"
       y="17.887218"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3520">08/12/13    1000.00    Delta PC</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="21.637218"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3731">08/15/13    75.46        Walts Drugs</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="25.387218"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3733">03/03/13    379.00      Epic Technologies</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="29.137218"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3735">03/04/13    20.28        YOUR LOCAL SU</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="32.887218"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3737">03/03/13    421.35      SPRINGFIELD WA</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="36.637218"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3739">03/03/13    379.00      Epic Technologies</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="40.387218"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3743">03/04/13    20.28        YOUR LOCAL SUP</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="44.137218"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3846">08/15/13    75.46        Walts Drugs</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="47.887218"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3745">08/12/13    1000.00    Delta PC</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="51.637218"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3747">03/03/13    421.35      SPRINGFIELD WA</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="55.387218"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3749">03/04/13    20.28        YOUR LOCAL SU</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="59.137218"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3751">03/03/13    379.00      Epic Technologies</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="62.887218"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3753">08/12/13    1000.00    De  a PC</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="66.637222"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3755">03/03/13    379.00      E       Technologies</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="70.387222"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3757">08/15/13    75.46        Walts Drugs</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="74.137222"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3759">03/04/13    20.28        YOUR LOCAL SU</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="77.887222"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3761">03/03/13    379.00      Epic Technologies</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="81.637222"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3763">08/12/13    1000.00    Delta PC</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="85.387222"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3765">08/15/13    75.46        Walts Drugs</tspan><tspan
       sodipodi:role="line"
       x="18.006462"
       y="89.137222"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3783" /><tspan
       sodipodi:role="line"
       x="18.006462"
       y="92.887222"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3799" /><tspan
       sodipodi:role="line"
       x="18.006462"
       y="96.637222"
       style="font-size:3px;font-style:normal;font-variant:normal;font-weight:bold;font-stretch:normal;font-family:Arial;-inkscape-font-specification:Arial Bold"
       id="tspan3801" /></text>
<text
     xml:space="preserve"
     style="font-size:40px;font-style:normal;font-weight:normal;line-height:125%;letter-spacing:0px;word-spacing:0px;fill:#000000;fill-opacity:1;stroke:none;font-family:Sans"
     x="43.851177"
     y="32.13871"
     id="text3838"
     sodipodi:linespacing="125%"
     inkscape:export-filename="/Users/arthurmaniet/Desktop/icon.png"
     inkscape:export-xdpi="115.2"
     inkscape:export-ydpi="115.2"><tspan
       sodipodi:role="line"
       id="tspan3840"
       x="43.851177"
       y="32.13871"
       style="font-size:16px;font-weight:bold;text-align:center;text-anchor:middle;-inkscape-font-specification:Sans Bold" /></text>
</svg>
```

## File: static\src\js\account_bank_statement_import.js

```javascript
odoo.define('account_bank_statement_import.import', function (require) {
"use strict";

var core = require('web.core');
var BaseImport = require('base_import.import');

var _t = core._t;

BaseImport.DataImport.include({
    renderImportLink: function() {
        this._super();
        if (this.res_model == 'account.bank.statement') {
            this.$(".import-link").prop({"text": _t(" Import Template for Bank Statements"), "href": "/account_bank_statement_import/static/csv/account.bank.statement.csv"});
            this.$(".template-import").removeClass('d-none');
        }
    },   
});

});

```

## File: views\account_bank_statement_import_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="account_bank_statement_import assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/account_bank_statement_import/static/src/js/account_bank_statement_import.js"></script>
        </xpath>
    </template>
</odoo>

```

## File: wizard\journal_creation.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models


class AccountBankStatementImportJounalCreation(models.TransientModel):
    _name = 'account.bank.statement.import.journal.creation'
    _description = 'Journal Creation on Bank Statement Import'

    journal_id = fields.Many2one('account.journal', delegate=True, required=True, ondelete='cascade')

    def create_journal(self):
        """ Create the journal (the record is automatically created in the process of calling this method) and reprocess the statement """
        statement_import_transient = self.env['account.bank.statement.import'].browse(self.env.context['statement_import_transient_id'])
        return statement_import_transient.with_context(journal_id=self.journal_id.id).import_file()

```

## File: wizard\journal_creation.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <data>

        <record id="account_bank_statement_import_journal_creation_view" model="ir.ui.view">
            <field name="name">Journal Creation</field>
            <field name="model">account.bank.statement.import.journal.creation</field>
            <field name="arch" type="xml">
                <form string="Journal Creation">
                    <p>The account of the statement you are uploading is not yet recorded in Odoo. In order to proceed with the upload, you need to create a bank journal for this account.</p>
                    <p>Just click OK to create the account/journal and finish the upload. If this was a mistake, hit cancel to abort the upload.</p>
                    <group>
                        <group>
                            <field name="name" string="Bank Journal Name"/>
                            <field name="bank_acc_number" readonly="1"/>
                            <field name="bank_id"/>
                        </group>
                        <group>
                            <field name="currency_id" readonly="1" groups="base.group_multi_currency"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                    </group>
                    <footer>
                        <button name="create_journal" string="OK" type="object" class="btn-primary"/>
                        <button string="Cancel" class="btn-secondary" special="cancel"/>
                    </footer>
                </form>
            </field>
        </record>

    </data>
</odoo>

```

## File: wizard\setup_wizards.py

```python
# -*- coding: utf-8 -*-

from odoo import models, fields, api


class SetupBarBankConfigWizard(models.TransientModel):
    _inherit = 'account.setup.bank.manual.config'

    def validate(self):
        """ Default the bank statement source of new bank journals as 'file_import'
        """
        super(SetupBarBankConfigWizard, self).validate()
        if (self.num_journals_without_account == 0 or self.linked_journal_id.bank_statements_source == 'undefined') \
          and self.env['account.journal']._get_bank_statements_available_import_formats():
            self.linked_journal_id.bank_statements_source = 'file_import'

```

## File: wizard\__init__.py

```python
from . import journal_creation
from . import setup_wizards
```

