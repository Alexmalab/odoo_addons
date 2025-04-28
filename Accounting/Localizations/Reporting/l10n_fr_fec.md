# Odoo Module: l10n_fr_fec

Category: Accounting/Localizations/Reporting

This file contains the source code of the Odoo module.

## File: README.rst

```rst
Fichier d'Échange Informatisé (FEC) pour la France
==================================================

Ce module permet de générer le fichier FEC tel que définit par `l'arrêté du 29
Juillet 2013 <http://legifrance.gouv.fr/eli/arrete/2013/7/29/BUDE1315492A/jo/texte>`
portant modification des dispositions de l'article A. 47 A-1 du
livre des procédures fiscales.

Cet arrêté prévoit l'obligation pour les sociétés ayant une comptabilité
informatisée de pouvoir fournir à l'administration fiscale un fichier
regroupant l'ensemble des écritures comptables de l'exercice. Le format de ce
fichier, appelé *FEC*, est définit dans l'arrêté.

Le détail du format du FEC est spécifié dans le bulletin officiel des finances publiques `BOI-CF-IOR-60-40-20-20131213 <http://bofip.impots.gouv.fr/bofip/ext/pdf/createPdfWithAnnexePermalien/BOI-CF-IOR-60-40-20-20131213.pdf?doc=9028-PGP&identifiant=BOI-CF-IOR-60-40-20-20131213>` du 13 Décembre 2013. Ce module implémente le fichier
FEC au format texte et non au format XML, car le format texte sera facilement
lisible et vérifiable par le comptable en utilisant un tableur.

La structure du fichier FEC généré par ce module a été vérifiée avec le logiciel
*Test Compta Demat* version 1_00_05 disponible sur
`le site de la direction générale des finances publiques <http://www.economie.gouv.fr/dgfip/outil-test-des-fichiers-des-ecritures-comptables-fec>`
en utilisant une base de donnée Odoo réelle.

Configuration
=============

Aucune configuration n'est nécessaire.

Utilisation
===========

Pour générer le *FEC*, allez dans le menu *Accounting > Reporting > French Statements > FEC* qui va démarrer l'assistant de génération du FEC.

Credits
=======

Contributors
------------

* Alexis de Lattre <alexis.delattre@akretion.com>


```

## File: __init__.py

```python
#-*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2013-2015 Akretion (http://www.akretion.com)

from . import wizard

```

## File: __manifest__.py

```python
#-*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2013-2015 Akretion (http://www.akretion.com)

{
    'name': 'France - FEC',
    'category': 'Accounting/Localizations/Reporting',
    'summary': "Fichier d'Échange Informatisé (FEC) for France",
    'author': "Akretion,Odoo Community Association (OCA)",
    'depends': ['l10n_fr', 'account'],
    'data': [
        'security/ir.model.access.csv',
        'security/security.xml',
        'wizard/account_fr_fec_view.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_account_fr_fec,account.fr.fec,model_account_fr_fec,account.group_account_user,1,1,1,0

```

## File: security\security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="account_fr_fec_rule" model="ir.rule">
        <field name="name">Account Fr Fec Rule</field>
        <field name="model_id" ref="model_account_fr_fec"/>
        <field name="domain_force">[('create_uid', '=', user.id)]</field>
    </record>
</odoo>

```

## File: wizard\account_fr_fec.py

```python
#-*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2013-2015 Akretion (http://www.akretion.com)

import base64
import io

from odoo import api, fields, models, _
from odoo.exceptions import UserError, AccessDenied
from odoo.tools import float_is_zero, pycompat
from odoo.tools.misc import get_lang
from stdnum.fr import siren


class AccountFrFec(models.TransientModel):
    _name = 'account.fr.fec'
    _description = 'Ficher Echange Informatise'

    date_from = fields.Date(string='Start Date', required=True)
    date_to = fields.Date(string='End Date', required=True)
    fec_data = fields.Binary('FEC File', readonly=True)
    filename = fields.Char(string='Filename', size=256, readonly=True)
    test_file = fields.Boolean()
    export_type = fields.Selection([
        ('official', 'Official FEC report (posted entries only)'),
        ('nonofficial', 'Non-official FEC report (posted and unposted entries)'),
        ], string='Export Type', required=True, default='official')

    @api.onchange('test_file')
    def _onchange_export_file(self):
        if not self.test_file:
            self.export_type = 'official'

    def _do_query_unaffected_earnings(self):
        ''' Compute the sum of ending balances for all accounts that are of a type that does not bring forward the balance in new fiscal years.
            This is needed because we have to display only one line for the initial balance of all expense/revenue accounts in the FEC.
        '''

        sql_query = '''
        SELECT
            'OUV' AS JournalCode,
            'Balance initiale' AS JournalLib,
            'OUVERTURE/' || %s AS EcritureNum,
            %s AS EcritureDate,
            '120/129' AS CompteNum,
            'Benefice (perte) reporte(e)' AS CompteLib,
            '' AS CompAuxNum,
            '' AS CompAuxLib,
            '-' AS PieceRef,
            %s AS PieceDate,
            '/' AS EcritureLib,
            replace(CASE WHEN COALESCE(sum(aml.balance), 0) <= 0 THEN '0,00' ELSE to_char(SUM(aml.balance), '000000000000000D99') END, '.', ',') AS Debit,
            replace(CASE WHEN COALESCE(sum(aml.balance), 0) >= 0 THEN '0,00' ELSE to_char(-SUM(aml.balance), '000000000000000D99') END, '.', ',') AS Credit,
            '' AS EcritureLet,
            '' AS DateLet,
            %s AS ValidDate,
            '' AS Montantdevise,
            '' AS Idevise
        FROM
            account_move_line aml
            LEFT JOIN account_move am ON am.id=aml.move_id
            JOIN account_account aa ON aa.id = aml.account_id
            LEFT JOIN account_account_type aat ON aa.user_type_id = aat.id
        WHERE
            am.date < %s
            AND am.company_id = %s
            AND aat.include_initial_balance IS NOT TRUE
        '''
        # For official report: only use posted entries
        if self.export_type == "official":
            sql_query += '''
            AND am.state = 'posted'
            '''
        company = self.env.company
        formatted_date_from = fields.Date.to_string(self.date_from).replace('-', '')
        date_from = self.date_from
        formatted_date_year = date_from.year
        self._cr.execute(
            sql_query, (formatted_date_year, formatted_date_from, formatted_date_from, formatted_date_from, self.date_from, company.id))
        listrow = []
        row = self._cr.fetchone()
        listrow = list(row)
        return listrow

    def _get_company_legal_data(self, company):
        """
        Dom-Tom are excluded from the EU's fiscal territory
        Those regions do not have SIREN
        sources:
            https://www.service-public.fr/professionnels-entreprises/vosdroits/F23570
            http://www.douane.gouv.fr/articles/a11024-tva-dans-les-dom

        * Returns the siren if the company is french or an empty siren for dom-tom
        * For non-french companies -> returns the complete vat number
        """
        dom_tom_group = self.env.ref('l10n_fr.dom-tom')
        is_dom_tom = company.country_id.code in dom_tom_group.country_ids.mapped('code')
        if not company.vat or is_dom_tom:
            return ''
        elif company.country_id.code == 'FR' and len(company.vat) >= 13 and siren.is_valid(company.vat[4:13]):
            return company.vat[4:13]
        else:
            return company.vat

    def generate_fec(self):
        self.ensure_one()
        if not (self.env.is_admin() or self.env.user.has_group('account.group_account_user')):
            raise AccessDenied()
        # We choose to implement the flat file instead of the XML
        # file for 2 reasons :
        # 1) the XSD file impose to have the label on the account.move
        # but Odoo has the label on the account.move.line, so that's a
        # problem !
        # 2) CSV files are easier to read/use for a regular accountant.
        # So it will be easier for the accountant to check the file before
        # sending it to the fiscal administration
        today = fields.Date.today()
        if self.date_from > today or self.date_to > today:
            raise UserError(_('You could not set the start date or the end date in the future.'))
        if self.date_from >= self.date_to:
            raise UserError(_('The start date must be inferior to the end date.'))

        company = self.env.company
        company_legal_data = self._get_company_legal_data(company)

        header = [
            u'JournalCode',    # 0
            u'JournalLib',     # 1
            u'EcritureNum',    # 2
            u'EcritureDate',   # 3
            u'CompteNum',      # 4
            u'CompteLib',      # 5
            u'CompAuxNum',     # 6  We use partner.id
            u'CompAuxLib',     # 7
            u'PieceRef',       # 8
            u'PieceDate',      # 9
            u'EcritureLib',    # 10
            u'Debit',          # 11
            u'Credit',         # 12
            u'EcritureLet',    # 13
            u'DateLet',        # 14
            u'ValidDate',      # 15
            u'Montantdevise',  # 16
            u'Idevise',        # 17
            ]

        rows_to_write = [header]
        # INITIAL BALANCE
        unaffected_earnings_xml_ref = self.env.ref('account.data_unaffected_earnings')
        unaffected_earnings_line = True  # used to make sure that we add the unaffected earning initial balance only once
        if unaffected_earnings_xml_ref:
            #compute the benefit/loss of last year to add in the initial balance of the current year earnings account
            unaffected_earnings_results = self._do_query_unaffected_earnings()
            unaffected_earnings_line = False

        sql_query = '''
        SELECT
            'OUV' AS JournalCode,
            'Balance initiale' AS JournalLib,
            'OUVERTURE/' || %s AS EcritureNum,
            %s AS EcritureDate,
            MIN(aa.code) AS CompteNum,
            replace(replace(MIN(aa.name), '|', '/'), '\t', '') AS CompteLib,
            '' AS CompAuxNum,
            '' AS CompAuxLib,
            '-' AS PieceRef,
            %s AS PieceDate,
            '/' AS EcritureLib,
            replace(CASE WHEN sum(aml.balance) <= 0 THEN '0,00' ELSE to_char(SUM(aml.balance), '000000000000000D99') END, '.', ',') AS Debit,
            replace(CASE WHEN sum(aml.balance) >= 0 THEN '0,00' ELSE to_char(-SUM(aml.balance), '000000000000000D99') END, '.', ',') AS Credit,
            '' AS EcritureLet,
            '' AS DateLet,
            %s AS ValidDate,
            '' AS Montantdevise,
            '' AS Idevise,
            MIN(aa.id) AS CompteID
        FROM
            account_move_line aml
            LEFT JOIN account_move am ON am.id=aml.move_id
            JOIN account_account aa ON aa.id = aml.account_id
            LEFT JOIN account_account_type aat ON aa.user_type_id = aat.id
        WHERE
            am.date < %s
            AND am.company_id = %s
            AND aat.include_initial_balance = 't'
        '''

        # For official report: only use posted entries
        if self.export_type == "official":
            sql_query += '''
            AND am.state = 'posted'
            '''

        sql_query += '''
        GROUP BY aml.account_id, aat.type
        HAVING aat.type not in ('receivable', 'payable')
        '''
        formatted_date_from = fields.Date.to_string(self.date_from).replace('-', '')
        date_from = self.date_from
        formatted_date_year = date_from.year
        currency_digits = 2

        self._cr.execute(
            sql_query, (formatted_date_year, formatted_date_from, formatted_date_from, formatted_date_from, self.date_from, company.id))

        for row in self._cr.fetchall():
            listrow = list(row)
            account_id = listrow.pop()
            if not unaffected_earnings_line:
                account = self.env['account.account'].browse(account_id)
                if account.user_type_id.id == self.env.ref('account.data_unaffected_earnings').id:
                    #add the benefit/loss of previous fiscal year to the first unaffected earnings account found.
                    unaffected_earnings_line = True
                    current_amount = float(listrow[11].replace(',', '.')) - float(listrow[12].replace(',', '.'))
                    unaffected_earnings_amount = float(unaffected_earnings_results[11].replace(',', '.')) - float(unaffected_earnings_results[12].replace(',', '.'))
                    listrow_amount = current_amount + unaffected_earnings_amount
                    if float_is_zero(listrow_amount, precision_digits=currency_digits):
                        continue
                    if listrow_amount > 0:
                        listrow[11] = str(listrow_amount).replace('.', ',')
                        listrow[12] = '0,00'
                    else:
                        listrow[11] = '0,00'
                        listrow[12] = str(-listrow_amount).replace('.', ',')
            rows_to_write.append(listrow)

        #if the unaffected earnings account wasn't in the selection yet: add it manually
        if (not unaffected_earnings_line
            and unaffected_earnings_results
            and (unaffected_earnings_results[11] != '0,00'
                 or unaffected_earnings_results[12] != '0,00')):
            #search an unaffected earnings account
            unaffected_earnings_account = self.env['account.account'].search([('user_type_id', '=', self.env.ref('account.data_unaffected_earnings').id),
                                                                              ('company_id', '=', company.id)], limit=1)
            if unaffected_earnings_account:
                unaffected_earnings_results[4] = unaffected_earnings_account.code
                unaffected_earnings_results[5] = unaffected_earnings_account.name
            rows_to_write.append(unaffected_earnings_results)

        # INITIAL BALANCE - receivable/payable
        sql_query = '''
        SELECT
            'OUV' AS JournalCode,
            'Balance initiale' AS JournalLib,
            'OUVERTURE/' || %s AS EcritureNum,
            %s AS EcritureDate,
            MIN(aa.code) AS CompteNum,
            replace(MIN(aa.name), '|', '/') AS CompteLib,
            CASE WHEN MIN(aat.type) IN ('receivable', 'payable')
            THEN
                CASE WHEN rp.ref IS null OR rp.ref = ''
                THEN rp.id::text
                ELSE replace(rp.ref, '|', '/')
                END
            ELSE ''
            END
            AS CompAuxNum,
            CASE WHEN aat.type IN ('receivable', 'payable')
            THEN COALESCE(replace(rp.name, '|', '/'), '')
            ELSE ''
            END AS CompAuxLib,
            '-' AS PieceRef,
            %s AS PieceDate,
            '/' AS EcritureLib,
            replace(CASE WHEN sum(aml.balance) <= 0 THEN '0,00' ELSE to_char(SUM(aml.balance), '000000000000000D99') END, '.', ',') AS Debit,
            replace(CASE WHEN sum(aml.balance) >= 0 THEN '0,00' ELSE to_char(-SUM(aml.balance), '000000000000000D99') END, '.', ',') AS Credit,
            '' AS EcritureLet,
            '' AS DateLet,
            %s AS ValidDate,
            '' AS Montantdevise,
            '' AS Idevise,
            MIN(aa.id) AS CompteID
        FROM
            account_move_line aml
            LEFT JOIN account_move am ON am.id=aml.move_id
            LEFT JOIN res_partner rp ON rp.id=aml.partner_id
            JOIN account_account aa ON aa.id = aml.account_id
            LEFT JOIN account_account_type aat ON aa.user_type_id = aat.id
        WHERE
            am.date < %s
            AND am.company_id = %s
            AND aat.include_initial_balance = 't'
        '''

        # For official report: only use posted entries
        if self.export_type == "official":
            sql_query += '''
            AND am.state = 'posted'
            '''

        sql_query += '''
        GROUP BY aml.account_id, aat.type, rp.ref, rp.id
        HAVING aat.type in ('receivable', 'payable')
        '''
        self._cr.execute(
            sql_query, (formatted_date_year, formatted_date_from, formatted_date_from, formatted_date_from, self.date_from, company.id))

        for row in self._cr.fetchall():
            listrow = list(row)
            account_id = listrow.pop()
            rows_to_write.append(listrow)

        # LINES
        query_limit = int(self.env['ir.config_parameter'].sudo().get_param('l10n_fr_fec.batch_size', 500000)) # To prevent memory errors when fetching the results
        sql_query = f'''
        SELECT
            REGEXP_REPLACE(replace(aj.code, '|', '/'), '[\\t\\r\\n]', ' ', 'g') AS JournalCode,
            REGEXP_REPLACE(replace(COALESCE(aj__name.value, aj.name), '|', '/'), '[\\t\\r\\n]', ' ', 'g') AS JournalLib,
            REGEXP_REPLACE(replace(am.name, '|', '/'), '[\\t\\r\\n]', ' ', 'g') AS EcritureNum,
            TO_CHAR(am.date, 'YYYYMMDD') AS EcritureDate,
            aa.code AS CompteNum,
            REGEXP_REPLACE(replace(aa.name, '|', '/'), '[\\t\\r\\n]', ' ', 'g') AS CompteLib,
            CASE WHEN aat.type IN ('receivable', 'payable')
            THEN
                CASE WHEN rp.ref IS null OR rp.ref = ''
                THEN rp.id::text
                ELSE replace(rp.ref, '|', '/')
                END
            ELSE ''
            END
            AS CompAuxNum,
            CASE WHEN aat.type IN ('receivable', 'payable')
            THEN COALESCE(REGEXP_REPLACE(replace(rp.name, '|', '/'), '[\\t\\r\\n]', ' ', 'g'), '')
            ELSE ''
            END AS CompAuxLib,
            CASE WHEN am.ref IS null OR am.ref = ''
            THEN '-'
            ELSE REGEXP_REPLACE(replace(am.ref, '|', '/'), '[\\t\\r\\n]', ' ', 'g')
            END
            AS PieceRef,
            TO_CHAR(COALESCE(am.invoice_date, am.date), 'YYYYMMDD') AS PieceDate,
            CASE WHEN aml.name IS NULL OR aml.name = '' THEN '/'
                WHEN aml.name SIMILAR TO '[\\t|\\s|\\n]*' THEN '/'
                ELSE REGEXP_REPLACE(replace(aml.name, '|', '/'), '[\\t\\n\\r]', ' ', 'g') END AS EcritureLib,
            replace(CASE WHEN aml.debit = 0 THEN '0,00' ELSE to_char(aml.debit, '000000000000000D99') END, '.', ',') AS Debit,
            replace(CASE WHEN aml.credit = 0 THEN '0,00' ELSE to_char(aml.credit, '000000000000000D99') END, '.', ',') AS Credit,
            CASE WHEN rec.name IS NULL THEN '' ELSE rec.name END AS EcritureLet,
            CASE WHEN aml.full_reconcile_id IS NULL THEN '' ELSE TO_CHAR(rec.create_date, 'YYYYMMDD') END AS DateLet,
            TO_CHAR(am.date, 'YYYYMMDD') AS ValidDate,
            CASE
                WHEN aml.amount_currency IS NULL OR aml.amount_currency = 0 THEN ''
                ELSE replace(to_char(aml.amount_currency, '000000000000000D99'), '.', ',')
            END AS Montantdevise,
            CASE WHEN aml.currency_id IS NULL THEN '' ELSE rc.name END AS Idevise
        FROM
            account_move_line aml
            LEFT JOIN account_move am ON am.id=aml.move_id
            LEFT JOIN res_partner rp ON rp.id=aml.partner_id
            JOIN account_journal aj ON aj.id = am.journal_id
            LEFT JOIN ir_translation aj__name ON aj__name.res_id = aj.id
                                             AND aj__name.type = 'model'
                                             AND aj__name.name = 'account.journal,name'
                                             AND aj__name.lang = %s
                                             AND aj__name.value != ''
            JOIN account_account aa ON aa.id = aml.account_id
            LEFT JOIN account_account_type aat ON aa.user_type_id = aat.id
            LEFT JOIN res_currency rc ON rc.id = aml.currency_id
            LEFT JOIN account_full_reconcile rec ON rec.id = aml.full_reconcile_id
        WHERE
            am.date >= %s
            AND am.date <= %s
            AND am.company_id = %s
            {"AND am.state = 'posted'" if self.export_type == 'official' else ""}
        ORDER BY
            am.date,
            am.name,
            aml.id
        LIMIT %s
        OFFSET %s
        '''

        lang = self.env.user.lang or get_lang(self.env).code

        with io.BytesIO() as fecfile:
            csv_writer = pycompat.csv_writer(fecfile, delimiter='|', lineterminator='')

            # Write header and initial balances
            for initial_row in rows_to_write:
                initial_row = list(initial_row)
                # We don't skip \n at then end of the file if there are only initial balances, for simplicity. An empty period export shouldn't happen IRL.
                initial_row[-1] += u'\r\n'
                csv_writer.writerow(initial_row)

            # Write current period's data
            query_offset = 0
            has_more_results = True
            while has_more_results:
                self._cr.execute(
                    sql_query,
                    (lang, self.date_from, self.date_to, company.id, query_limit + 1, query_offset)
                )
                query_offset += query_limit
                has_more_results = self._cr.rowcount > query_limit # we load one more result than the limit to check if there is more
                query_results = self._cr.fetchall()
                for i, row in enumerate(query_results[:query_limit]):
                    if i < len(query_results) - 1:
                        # The file is not allowed to end with an empty line, so we can't use lineterminator on the writer
                        row = list(row)
                        row[-1] += u'\r\n'
                    csv_writer.writerow(row)

            base64_result = base64.encodebytes(fecfile.getvalue())

        end_date = fields.Date.to_string(self.date_to).replace('-', '')
        suffix = ''
        if self.export_type == "nonofficial":
            suffix = '-NONOFFICIAL'

        self.write({
            'fec_data': base64_result,
            # Filename = <siren>FECYYYYMMDD where YYYMMDD is the closing date
            'filename': '%sFEC%s%s.csv' % (company_legal_data, end_date, suffix),
            })

        # Set fiscal year lock date to the end date (not in test)
        fiscalyear_lock_date = self.env.company.fiscalyear_lock_date
        if not self.test_file and (not fiscalyear_lock_date or fiscalyear_lock_date < self.date_to):
            self.env.company.write({'fiscalyear_lock_date': self.date_to})
        return {
            'name': 'FEC',
            'type': 'ir.actions.act_url',
            'url': "web/content/?model=account.fr.fec&id=" + str(self.id) + "&filename_field=filename&field=fec_data&download=true&filename=" + self.filename,
            'target': 'self',
        }

    def _csv_write_rows(self, rows, lineterminator=u'\r\n'): #DEPRECATED; will disappear in master
        """
        Write FEC rows into a file
        It seems that Bercy's bureaucracy is not too happy about the
        empty new line at the End Of File.

        @param {list(list)} rows: the list of rows. Each row is a list of strings
        @param {unicode string} [optional] lineterminator: effective line terminator
            Has nothing to do with the csv writer parameter
            The last line written won't be terminated with it

        @return the value of the file
        """
        fecfile = io.BytesIO()
        writer = pycompat.csv_writer(fecfile, delimiter='|', lineterminator='')

        rows_length = len(rows)
        for i, row in enumerate(rows):
            if not i == rows_length - 1:
                row[-1] += lineterminator
            writer.writerow(row)

        fecvalue = fecfile.getvalue()
        fecfile.close()
        return fecvalue

```

## File: wizard\account_fr_fec_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<record id="account_fr_fec_view" model="ir.ui.view">
    <field name="name">account.fr.fec.form.view</field>
    <field name="model">account.fr.fec</field>
    <field name="arch" type="xml">
        <form string="FEC File Generation">
            <div class="alert alert-info" role="alert" attrs="{'invisible': [('test_file', '=', True)]}">
                When you download a FEC file, the lock date is set to the end date.
                If you want to test the FEC file generation, please tick the test file checkbox.
            </div>
            <div class="alert alert-info" role="alert" attrs="{'invisible': [('test_file', '=', False)]}">
                You are in test mode. The FEC file generation will not set the lock date.
            </div>
            <notebook>
                <page string="Options" name="options">
                    <group>
                        <field name="date_from"/>
                        <field name="date_to"/>
                        <field name="test_file"/>
                        <field name="export_type" attrs="{'invisible': [('test_file', '=', False)]}"/>
                    </group>
                </page>
                <page string="Technical Info" name="technical_info">
                    <group>
                        <div>
                        The encoding of this text file is UTF-8. The structure of file is CSV separated by pipe '|'.
                        </div>
                    </group>
                    <group>
                        <table style="width:80%">
                            <tr>
                                <th>Technical Name</th>
                                <th>Column</th>
                                <th>Comment</th>
                            </tr>
                            <tr>
                                <td>JournalCode</td>
                                <td># 0</td>
                            </tr>
                            <tr>
                                <td>JournalLib</td>
                                <td>
                                    # 1</td>
                            </tr>
                            <tr>
                                <td>EcritureNum</td>
                                <td># 2</td>
                            </tr>
                            <tr>
                                <td>EcritureDate</td>
                                <td>
                                    # 3</td>
                            </tr>
                            <tr>
                                <td>CompteNum</td>
                                <td># 4</td>
                            </tr>
                            <tr>
                                <td>CompteLib</td>
                                <td># 5</td>
                            </tr>
                            <tr>
                                <td>CompAuxNum</td>
                                <td># 6</td>
                                <td>We use partner.id</td>
                            </tr>
                            <tr>
                                <td>CompAuxLib</td>
                                <td># 7</td>
                            </tr>
                            <tr>
                                <td>PieceRef</td>
                                <td># 8</td>
                            </tr>
                            <tr>
                                <td>PieceDate</td>
                                <td># 9</td>
                            </tr>
                            <tr>
                                <td>EcritureLib</td>
                                <td># 10</td>
                            </tr>
                            <tr>
                                <td>Debit</td>
                                <td># 11</td>
                            </tr>
                            <tr>
                                <td>Credit</td>
                                <td># 12</td>
                            </tr>
                            <tr>
                                <td>EcritureLet</td>
                                <td># 13</td>
                            </tr>
                            <tr>
                                <td>DateLet</td>
                                <td># 14</td>
                            </tr>
                            <tr>
                                <td>ValidDate</td>
                                <td># 15</td>
                            </tr>
                            <tr>
                                <td>Montantdevise</td>
                                <td># 16</td>
                            </tr>
                            <tr>
                                <td>Idevise</td>
                                <td># 17</td>
                            </tr>
                        </table>
                    </group>
                </page>
            </notebook>
            <footer>
                <button string="Generate" name="generate_fec" type="object"
                    class="oe_highlight"/>
                <button string="Cancel" class="btn btn-secondary" special="cancel"/>
            </footer>
        </form>
    </field>
</record>

<record id="account_fr_fec_action" model="ir.actions.act_window">
    <field name="name">FEC</field>
    <field name="res_model">account.fr.fec</field>
    <field name="view_mode">form</field>
    <field name="target">new</field>
</record>

<menuitem id="account_fr_fec_menu"
        parent="l10n_fr.account_reports_fr_statements_menu"
        action="account_fr_fec_action"
        sequence="100" />
</odoo>

```

## File: wizard\__init__.py

```python
#-*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2013-2015 Akretion (http://www.akretion.com)

from . import account_fr_fec

```

