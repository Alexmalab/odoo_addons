# Odoo Module: l10n_it_edi_withholding

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import psycopg2
from . import models
from odoo import api, SUPERUSER_ID
import logging

_logger = logging.getLogger(__name__)

def _l10n_it_edi_add_accounts(env, company):
    """ Create the transition accounts """
    generated_accounts_ref = {}
    account_templates = {account_code: env.ref(f'l10n_it_edi_withholding.{account_code}') for account_code in ['1611', '2603']}
    try:
        with env.cr.savepoint():
            for account_code, account_template in account_templates.items():
                template_vals = [(account_template, company.chart_template_id._get_account_vals(company, account_template, account_code, {}))]
                generated_accounts_ref[account_template] = company.chart_template_id._create_records_with_xmlid('account.account', template_vals, company)
            _logger.info("Created withholding accounts for company: %s(%s).", company.name, company.id)
    except psycopg2.errors.UniqueViolation:
        generated_accounts_ref = {}
        _logger.error("Cash basis transition accounts already exist for company: %s(%s).", company.name, company.id)
    return generated_accounts_ref

def _l10n_it_edi_withholding_add_taxes(env, company):
    """ Create the new taxes on existing company """
    templates = env['account.tax.template']
    generated_taxes_ref = {}
    try:
        with env.cr.savepoint():
            for xml_id in (
                '20awi', '20vwi',
                '20awc', '20vwc',
                '23awo', '23vwo',
                '4vcp', '4acp',
                '4vinps', '4ainps'
            ):
                templates |= env.ref(f"l10n_it_edi_withholding.{xml_id}")
            generated_taxes_ref = templates._generate_tax(company)
            _logger.info("Created withholding taxes for company: %s(%s).", company.name, company.id)

            # Increase the sequence number of the old taxes multiplying it by 10 and adding 21
            # so that the withholding can have sequence=10 and the pension fund sequence=20
            offset = 21
            all_taxes = env['account.tax'].with_context(active_test=False).search([('company_id', '=', company.id)])
            for tax in all_taxes.filtered(lambda x: (x.sequence <= 20 and not x.l10n_it_pension_fund_type and not x.l10n_it_withholding_type)):
                if not tax.l10n_it_withholding_type and not tax.l10n_it_pension_fund_type:
                    tax.sequence += offset
            _logger.info("Increased sequence number of old taxes by %s for company: %s(%s).", offset, company.name, company.id)
    except psycopg2.errors.UniqueViolation:
        generated_taxes_ref = {}
        _logger.error("Withholding and Pension Fund taxes already exist for company: %s(%s).", company.name, company.id)

    return generated_taxes_ref

def _l10n_it_edi_setup_accounts_on_taxes(env, company, generated_taxes_ref, generated_accounts_ref):
    """ Setup the accounts on the taxes, after the accounts have been created """
    # Set the transition account
    for tax, value in generated_taxes_ref['account_dict']['account.tax'].items():
        transition_account = value['cash_basis_transition_account_id']
        if transition_account:
            tax.cash_basis_transition_account_id = generated_accounts_ref.get(transition_account)

    # The tax repartition lines accounts has already been generated from the template by l10n_it
    referenced_accounts = {}
    repartitions_dict = generated_taxes_ref['account_dict']['account.tax.repartition.line']
    for dummy, value_dict in repartitions_dict.items():
        account_template = value_dict['account_id']
        template_xml_id = account_template.get_external_id()[account_template.id]
        xml_id = template_xml_id.split('.')
        referenced_accounts[template_xml_id] = env.ref(f'{xml_id[0]}.{company.id}_{xml_id[1]}')

    # Set the tax repartition lines accounts on the generated taxes
    for repartition_line, value in repartitions_dict.items():
        template = value['account_id']
        if template:
            repartition_line.account_id = referenced_accounts[template.get_external_id()[template.id]]
    _logger.info("Cash Basis Transition Accounts and Repartition accounts on taxes for company: %s(%s).", company.name, company.id)

def _l10n_it_edi_withholding_post_init(cr, registry):
    """ Existing companies that have the Italian Chart of Accounts set """
    env = api.Environment(cr, SUPERUSER_ID, {})
    chart_template = env.ref('l10n_it.l10n_it_chart_template_generic')
    if chart_template:
        for company in env['res.company'].search([('chart_template_id', '=', chart_template.id)]):
            _logger.info("Company %s already has the Italian localization installed, updating...", company.name)
            generated_taxes_ref = _l10n_it_edi_withholding_add_taxes(env, company)
            generated_accounts_ref = {} if not generated_taxes_ref else _l10n_it_edi_add_accounts(env, company)
            if generated_accounts_ref:
                _l10n_it_edi_setup_accounts_on_taxes(env, company, generated_taxes_ref, generated_accounts_ref)

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Italy - E-invoicing (Withholding)',
    'version': '0.1',
    'depends': [
        'l10n_it_edi'
    ],
    'author': 'Odoo',
    'description': """
Withholding and Pension Fund handling for the E-invoice implementation for Italy.

    The Withholding tax and the Pension Fund tax are computed like every other tax
    with the ordering by sequence, so please be careful with the order of the taxes
    in your tax configuration.

    Please also update the Italian Accounting module (l10n_it) when you install this module.
    """,
    'category': 'Accounting/Localizations/EDI',
    'website': 'https://www.odoo.com/documentation/16.0/applications/finance/accounting/fiscal_localizations/localizations/italy.html',
    'data': [
        'data/account_tax_group_data.xml',
        'data/account_withholding_report_data.xml',
        'data/account.account.template.csv',
        'data/account_tax_template.xml',
        'data/invoice_it_template.xml',
        'views/l10n_it_view.xml'
    ],
    'post_init_hook': '_l10n_it_edi_withholding_post_init',
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,code,name,account_type,reconcile,chart_template_id:id,tag_ids:id
1611,1611,Crediti per ritenute subite (appoggio),asset_current,FALSE,l10n_it.l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
2603,2603,Debiti per ritenute da versare (appoggio),liability_current,FALSE,l10n_it.l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_pension_fund" model="account.tax.group">
            <field name="name">Fondi Pensione</field>
            <field name="sequence">0</field>
            <field name="country_id" ref="base.it"/>
            <field name="preceding_subtotal">Fondi Pensione Esclusi</field>
        </record>
        <record id="tax_group_enasarco" model="account.tax.group">
            <field name="name">Enasarco</field>
            <field name="sequence">999</field>
            <field name="country_id" ref="base.it"/>
            <field name="preceding_subtotal">ENASARCO Escluso</field>
        </record>
        <record id="tax_group_withholding" model="account.tax.group">
            <field name="name">Ritenute</field>
            <field name="sequence">1000</field>
            <field name="country_id" ref="base.it"/>
            <field name="preceding_subtotal">Totale Ritenute Escluse</field>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_template.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<odoo>

    <!-- Withholding Taxes (Ritenute) ......................................................... -->
    <!-- Individuals -->
    <record id="20vwi" model="account.tax.template">
        <field name="description">20% Ritenuta Persone Fisiche</field>
        <field name="chart_template_id" ref="l10n_it.l10n_it_chart_template_generic"/>
        <field name="name">20% RIT PF</field>
        <field name="sequence">10</field>
        <field name="amount">-20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="l10n_it_withholding_type">RT01</field>
        <field name="l10n_it_withholding_reason">A</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="l10n_it_edi_withholding.1611"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.1609'),
                'minus_report_expression_ids': [ref('withh_sale_tax_report_it_line_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.1609'),
                'plus_report_expression_ids': [ref('withh_sale_tax_report_it_line_tag')],
            }),
        ]"/>
    </record>

    <record id="20awi" model="account.tax.template">
        <field name="description">20% Ritenuta Persone Fisiche</field>
        <field name="chart_template_id" ref="l10n_it.l10n_it_chart_template_generic"/>
        <field name="name">20% RIT PF</field>
        <field name="sequence">10</field>
        <field name="amount">-20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="l10n_it_withholding_type">RT01</field>
        <field name="l10n_it_withholding_reason">A</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="l10n_it_edi_withholding.2603"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.2602'),
                'plus_report_expression_ids': [ref('withh_purchase_tax_report_it_line_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.2602'),
                'minus_report_expression_ids': [ref('withh_purchase_tax_report_it_line_tag')],
            }),
        ]"/>
    </record>

    <!-- Individual Companies -->
    <record id="20vwc" model="account.tax.template">
        <field name="description">20% Ritenuta Persone Giuridiche</field>
        <field name="chart_template_id" ref="l10n_it.l10n_it_chart_template_generic"/>
        <field name="name">20% RIT PG</field>
        <field name="sequence">11</field>
        <field name="amount">-20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="l10n_it_withholding_type">RT02</field>
        <field name="l10n_it_withholding_reason">A</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="l10n_it_edi_withholding.1611"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.1609'),
                'minus_report_expression_ids': [ref('withh_sale_tax_report_it_line_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.1609'),
                'plus_report_expression_ids': [ref('withh_sale_tax_report_it_line_tag')],
            }),
        ]"/>
    </record>

    <record id="20awc" model="account.tax.template">
        <field name="description">20% Ritenuta Persone Giuridiche</field>
        <field name="chart_template_id" ref="l10n_it.l10n_it_chart_template_generic"/>
        <field name="name">20% RIT PG</field>
        <field name="sequence">11</field>
        <field name="amount">-20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="l10n_it_withholding_type">RT02</field>
        <field name="l10n_it_withholding_reason">A</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="l10n_it_edi_withholding.2603"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.2602'),
                'plus_report_expression_ids': [ref('withh_purchase_tax_report_it_line_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.2602'),
                'minus_report_expression_ids': [ref('withh_purchase_tax_report_it_line_tag')],
            }),
        ]"/>
    </record>

    <!-- Agents and Representatives -->
    <record id="23vwo" model="account.tax.template">
        <field name="description">23% Ritenuta Agenti e Rappresentanti</field>
        <field name="chart_template_id" ref="l10n_it.l10n_it_chart_template_generic"/>
        <field name="name">23% RIT AG</field>
        <field name="sequence">12</field>
        <field name="amount">-23</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="l10n_it_withholding_type">RT02</field>
        <field name="l10n_it_withholding_reason">ZO</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="l10n_it_edi_withholding.1611"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.1609'),
                'minus_report_expression_ids': [ref('withh_sale_tax_report_it_line_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.1609'),
                'plus_report_expression_ids': [ref('withh_sale_tax_report_it_line_tag')],
            }),
        ]"/>
    </record>

    <record id="23awo" model="account.tax.template">
        <field name="description">23% Ritenuta Agenti e Rappresentanti</field>
        <field name="chart_template_id" ref="l10n_it.l10n_it_chart_template_generic"/>
        <field name="name">23% RIT AG</field>
        <field name="sequence">12</field>
        <field name="amount">-23</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_withholding"/>
        <field name="l10n_it_withholding_type">RT02</field>
        <field name="l10n_it_withholding_reason">ZO</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="l10n_it_edi_withholding.2603"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.2602'),
                'plus_report_expression_ids': [ref('withh_purchase_tax_report_it_line_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.2602'),
                'minus_report_expression_ids': [ref('withh_purchase_tax_report_it_line_tag')],
            }),
        ]"/>
    </record>

    <!-- INPS ................................................................................. -->
    <record id="4vinps" model="account.tax.template">
        <field name="description">4% Contributo INPS</field>
        <field name="chart_template_id" ref="l10n_it.l10n_it_chart_template_generic"/>
        <field name="name">4% INPS</field>
        <field name="sequence">5</field>
        <field name="amount">4</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="include_base_amount">True</field>
        <field name="tax_group_id" ref="tax_group_pension_fund"/>
        <field name="l10n_it_pension_fund_type">TC22</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.2630'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.2630'),
            }),
        ]"/>
    </record>

    <record id="4ainps" model="account.tax.template">
        <field name="description">4% Contributo INPS</field>
        <field name="chart_template_id" ref="l10n_it.l10n_it_chart_template_generic"/>
        <field name="name">4% INPS</field>
        <field name="sequence">5</field>
        <field name="amount">4</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="include_base_amount">True</field>
        <field name="l10n_it_pension_fund_type">TC22</field>
        <field name="tax_group_id" ref="tax_group_pension_fund"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.4402'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.4402'),
            }),
        ]"/>
   </record>

    <!-- Pension Fund (Cassa Previdenziale) ................................................... -->
    <record id="4vcp" model="account.tax.template">
        <field name="description">4% Contributo Fondo Pensione</field>
        <field name="chart_template_id" ref="l10n_it.l10n_it_chart_template_generic"/>
        <field name="name">4% F.Pens.</field>
        <field name="sequence">20</field>
        <field name="amount">4</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="include_base_amount">True</field>
        <field name="tax_group_id" ref="tax_group_pension_fund"/>
        <field name="l10n_it_pension_fund_type">TC01</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.2630'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.2630'),
            }),
        ]"/>
    </record>

    <record id="4acp" model="account.tax.template">
        <field name="description">4% Contributo Fondo Pensione</field>
        <field name="chart_template_id" ref="l10n_it.l10n_it_chart_template_generic"/>
        <field name="name">4% F.Pens.</field>
        <field name="sequence">20</field>
        <field name="amount">4</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="include_base_amount">True</field>
        <field name="l10n_it_pension_fund_type">TC01</field>
        <field name="tax_group_id" ref="tax_group_pension_fund"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.4402'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.4402'),
            }),
        ]"/>
   </record>

    <!-- ENASARCO ................................................................ -->
    <record id="enasarcov" model="account.tax.template">
        <field name="description">8.5% Ritenuta ENASARCO</field>
        <field name="chart_template_id" ref="l10n_it.l10n_it_chart_template_generic"/>
        <field name="name">ENASARCO</field>
        <field name="sequence">13</field>
        <field name="amount">-8.5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_enasarco"/>
        <field name="l10n_it_pension_fund_type">TC07</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="l10n_it_edi_withholding.1611"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.2630'),
                'minus_report_expression_ids': [ref('enasarco_sale_tax_report_it_line_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.2630'),
                'plus_report_expression_ids': [ref('enasarco_sale_tax_report_it_line_tag')],
            }),
        ]"/>
    </record>

    <record id="enasarcoa" model="account.tax.template">
        <field name="description">8.5% Ritenuta ENASARCO</field>
        <field name="chart_template_id" ref="l10n_it.l10n_it_chart_template_generic"/>
        <field name="name">ENASARCO</field>
        <field name="sequence">13</field>
        <field name="amount">-8.5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_enasarco"/>
        <field name="l10n_it_pension_fund_type">TC07</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="l10n_it_edi_withholding.1611"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.2630'),
                'plus_report_expression_ids': [ref('enasarco_purchase_tax_report_it_line_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_it.2630'),
                'minus_report_expression_ids': [ref('enasarco_purchase_tax_report_it_line_tag')],
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\account_withholding_report_data.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<odoo>
    <record id="withh_tax_report_it" model="account.report">
        <field name="name">Withholding Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.it"/>
        <field name="column_ids">
            <record id="withh_tax_report_balance" model="account.report.column">
                <field name="name">Total</field>
                <field name="expression_label">total</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="withh_sale_tax_report_it_line" model="account.report.line">
                <field name="name">Withholding Amount (Sales)</field>
                <field name="code">ritv</field>
                <field name="expression_ids">
                    <record id="withh_sale_tax_report_it_line_tag" model="account.report.expression">
                        <field name="label">total</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">RITV</field>
                    </record>
                </field>
            </record>
            <record id="enasarco_sale_tax_report_it_line" model="account.report.line">
                <field name="name">ENASARCO Amount (Sales)</field>
                <field name="code">enasarcov</field>
                <field name="expression_ids">
                    <record id="enasarco_sale_tax_report_it_line_tag" model="account.report.expression">
                        <field name="label">total</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">ENASARCOV</field>
                    </record>
                </field>
            </record>
            <record id="withh_purchase_tax_report_it_line" model="account.report.line">
                <field name="name">Withholding Amount (Purchase)</field>
                <field name="code">rita</field>
                <field name="expression_ids">
                    <record id="withh_purchase_tax_report_it_line_tag" model="account.report.expression">
                        <field name="label">total</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">RITA</field>
                    </record>
                </field>
            </record>
            <record id="enasarco_purchase_tax_report_it_line" model="account.report.line">
                <field name="name">ENASARCO Amount (Purchase)</field>
                <field name="code">enasarcoa</field>
                <field name="expression_ids">
                    <record id="enasarco_purchase_tax_report_it_line_tag" model="account.report.expression">
                        <field name="label">total</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">ENASARCOA</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\invoice_it_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="account_invoice_it_FatturaPA_export_withholding" inherit_id="l10n_it_edi.account_invoice_it_FatturaPA_export">
        <xpath expr="//DatiGeneraliDocumento/Numero" position="after">
            <t t-if="withholding_values" t-foreach="withholding_values" t-as="withholding">
                <DatiRitenuta>
                    <TipoRitenuta t-esc="format_alphanumeric(withholding.tax.l10n_it_withholding_type)"/>
                    <ImportoRitenuta t-esc="format_monetary(withholding.tax_amount, currency)"/>
                    <AliquotaRitenuta t-esc="format_numbers(abs(withholding.tax.amount))"/>
                    <CausalePagamento t-esc="format_alphanumeric(withholding.tax.l10n_it_withholding_reason)"/>
                </DatiRitenuta>
            </t>
        </xpath>
        <xpath expr="//DatiGeneraliDocumento/ImportoTotaleDocumento" position="before">
            <t t-if="pension_fund_values" t-foreach="pension_fund_values" t-as="pension_fund">
                <DatiCassaPrevidenziale>
                    <TipoCassa t-esc="format_alphanumeric(pension_fund.tax.l10n_it_pension_fund_type)"/>
                    <AlCassa t-esc="format_numbers(pension_fund.tax.amount)"/>
                    <ImportoContributoCassa t-esc="format_monetary(pension_fund.tax_amount, currency)"/>
                    <ImponibileCassa t-esc="format_monetary(pension_fund.base_amount, currency)"/>
                    <AliquotaIVA t-esc="format_numbers(pension_fund.vat_tax.amount or 0.0)"/>
                    <Ritenuta t-if="pension_fund.withholding_tax and pension_fund.withholding_tax.sequence > pension_fund.tax.sequence">SI</Ritenuta>
                    <Natura t-if="pension_fund.vat_tax.l10n_it_has_exoneration" t-esc="format_alphanumeric(pension_fund.vat_tax.l10n_it_kind_exoneration)"/>
                    <RiferimentoAmministrazione t-if="pension_fund.tax.description" t-esc="format_alphanumeric(pension_fund.tax.description)[:20]"/>
                </DatiCassaPrevidenziale>
            </t>
        </xpath>
    </template>

    <template id="account_invoice_line_it_FatturaPA_withholding" inherit_id="l10n_it_edi.account_invoice_line_it_FatturaPA">
        <xpath expr="//AliquotaIVA" position="after">
            <t t-if="line.tax_ids._l10n_it_filter_kind('withholding')">
                <Ritenuta>SI</Ritenuta>
            </t>
        </xpath>
        <xpath expr="//DettaglioLinee" position="inside">
            <t t-if="enasarco_values and enasarco_values[line.id]">
                <AltriDatiGestionali>
                    <TipoDato>CASSA-PREV</TipoDato>
                    <RiferimentoTesto t-esc="format_alphanumeric('TC07 - ENASARCO (' + format_numbers(abs(enasarco_values[line.id]['amount'])).rstrip('.0') + '%)')"/>
                    <RiferimentoNumero t-esc="format_monetary(enasarco_values[line.id]['tax_amount'], currency)"/>
                </AltriDatiGestionali>
            </t>
        </xpath>
    </template>
</odoo>

```

## File: models\account_chart_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from odoo import models

_logger = logging.getLogger(__name__)


class AccountChartTemplate(models.Model):
    _inherit = 'account.chart.template'

    def _load(self, company):
        """
            Override normal default taxes, which are the ones with lowest sequence.
        """
        result = super()._load(company)
        if company.chart_template_id == self.env.ref('l10n_it.l10n_it_chart_template_generic'):
            company.account_sale_tax_id = self.env.ref(f'l10n_it.{company.id}_22v')
            company.account_purchase_tax_id = self.env.ref(f'l10n_it.{company.id}_22am')
        return result

```

## File: models\account_edi_format.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _
import logging


_logger = logging.getLogger(__name__)


class AccountEdiFormat(models.Model):
    _inherit = 'account.edi.format'

    def _l10n_it_edi_search_tax_for_import(self, company, percentage, extra_domain=None, vat_only=True):
        """ In case no withholding_type or pension_fund is specified, exclude taxes that have it.
            It means that we're searching for VAT taxes, especially in the base l10n_it_edi module
        """
        if vat_only:
            extra_domain += [('l10n_it_withholding_type', '=', False), ('l10n_it_pension_fund_type', '=', False)]
        return super()._l10n_it_edi_search_tax_for_import(company, percentage, extra_domain)

    def _l10n_it_edi_check_taxes_configuration(self, invoice):
        """
            Override to also allow pension_fund, withholding taxes.
            Needs not to call super, because super checks for one tax only per line.
        """
        errors = []
        for invoice_line in invoice.invoice_line_ids.filtered(lambda x: x.display_type == 'product'):
            all_taxes = invoice_line.tax_ids.flatten_taxes_hierarchy()
            vat_taxes, withholding_taxes, pension_fund_taxes = (all_taxes._l10n_it_filter_kind(kind) for kind in ('vat', 'withholding', 'pension_fund'))
            if len(vat_taxes.filtered(lambda x: x.amount >= 0)) != 1:
                errors.append(_("Bad tax configuration for line %s, there must be one and only one VAT tax per line", invoice_line.name))
            if len(pension_fund_taxes) > 1 or len(withholding_taxes) > 1:
                errors.append(_("Bad tax configuration for line %s, there must be one Withholding tax and one Pension Fund tax at max.", invoice_line.name))
        return errors

    def _l10n_it_edi_get_extra_info(self, company, document_type, body_tree, incoming=True):
        extra_info, message_to_log = super()._l10n_it_edi_get_extra_info(company, document_type, body_tree, incoming=incoming)

        type_tax_use_domain = extra_info['type_tax_use_domain']

        withholding_elements = body_tree.xpath('.//DatiGeneraliDocumento/DatiRitenuta')
        withholding_taxes = []
        for withholding in (withholding_elements or []):
            tipo_ritenuta = withholding.find("TipoRitenuta")
            reason = withholding.find("CausalePagamento")
            percentage = withholding.find('AliquotaRitenuta')
            withholding_type = tipo_ritenuta.text if tipo_ritenuta is not None else "RT02"
            withholding_reason = reason.text if reason is not None else "A"
            withholding_percentage = -float(percentage.text if percentage is not None else "0.0")
            withholding_tax = self._l10n_it_edi_search_tax_for_import(
                company,
                withholding_percentage,
                ([('l10n_it_withholding_type', '=', withholding_type),
                  ('l10n_it_withholding_reason', '=', withholding_reason)]
                 + type_tax_use_domain),
                vat_only=False)
            if withholding_tax:
                withholding_taxes.append(withholding_tax)
            else:
                message_to_log.append("%s<br/>%s" % (
                    _("Withholding tax not found"),
                    self.env['account.move']._compose_info_message(body_tree, '.'),
                ))
        extra_info["withholding_taxes"] = withholding_taxes

        pension_fund_elements = body_tree.xpath('.//DatiGeneraliDocumento/DatiCassaPrevidenziale')
        pension_fund_taxes = []
        for pension_fund in (pension_fund_elements or []):
            pension_fund_type = pension_fund.find("TipoCassa")
            tax_factor_percent = pension_fund.find("AlCassa")
            vat_tax_factor_percent = pension_fund.find("AliquotaIVA")
            pension_fund_type = pension_fund_type.text if pension_fund_type is not None else ""
            tax_factor_percent = float(tax_factor_percent.text or "0.0")
            vat_tax_factor_percent = float(vat_tax_factor_percent.text or "0.0")
            pension_fund_tax = self._l10n_it_edi_search_tax_for_import(
                company,
                tax_factor_percent,
                ([('l10n_it_pension_fund_type', '=', pension_fund_type)]
                 + type_tax_use_domain),
                vat_only=False)
            if pension_fund_tax:
                pension_fund_taxes.append(pension_fund_tax)
            else:
                message_to_log.append("%s<br/>%s" % (
                    _("Pension Fund tax not found"),
                    self.env['account.move']._compose_info_message(body_tree, '.'),
                ))
        extra_info["pension_fund_taxes"] = pension_fund_taxes

        return extra_info, message_to_log

    def _import_fattura_pa_line(self, element, invoice_line_form, extra_info):
        messages_to_log = super()._import_fattura_pa_line(element, invoice_line_form, extra_info)

        type_tax_use_domain = extra_info['type_tax_use_domain']

        for withholding_tax in extra_info.get('withholding_taxes', []):
            withholding_tags = element.xpath("Ritenuta")
            if withholding_tags and withholding_tags[0].text == 'SI':
                invoice_line_form.tax_ids |= withholding_tax
        for pension_fund_tax in extra_info.get('pension_fund_taxes', []):
            invoice_line_form.tax_ids |= pension_fund_tax

        if extra_info['simplified']:
            return messages_to_log

        price_subtotal = invoice_line_form.price_unit
        company = invoice_line_form.company_id

        # ENASARCO Pension Fund tax (works as a withholding)
        for other_data_element in element.xpath('.//AltriDatiGestionali'):
            data_kind_element = other_data_element.xpath("./TipoDato")
            text_element = other_data_element.xpath("./RiferimentoTesto")
            number_element = other_data_element.xpath("./RiferimentoNumero")
            if not data_kind_element or not text_element or not number_element:
                continue
            data_kind, data_text, number_text = data_kind_element[0].text.lower(), text_element[0].text.lower(), number_element[0].text
            if data_kind != 'cassa-prev' or ('enasarco' not in data_text and not 'tc07' in data_text):
                continue
            enasarco_amount = float(number_text)
            enasarco_percentage = -self.env.company.currency_id.round(enasarco_amount / price_subtotal * 100)
            enasarco_tax = self._l10n_it_edi_search_tax_for_import(
                company,
                enasarco_percentage,
                [('l10n_it_pension_fund_type', '=', 'TC07')] + type_tax_use_domain,
                vat_only=False)
            if enasarco_tax:
                invoice_line_form.tax_ids |= enasarco_tax
            else:
                messages_to_log.append("%s<br/>%s" % (
                    _("Enasarco tax not found for line with description '%s'", invoice_line_form.name),
                    self.env['account.move']._compose_info_message(other_data_element, '.'),
                ))

        return messages_to_log

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from collections import namedtuple
from odoo import api, fields, models

_logger = logging.getLogger(__name__)


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_it_amount_vat_signed = fields.Monetary(string='VAT', compute='_compute_amount_extended', currency_field='company_currency_id')
    l10n_it_amount_pension_fund_signed = fields.Monetary(string='Pension Fund', compute='_compute_amount_extended', currency_field='company_currency_id')
    l10n_it_amount_withholding_signed = fields.Monetary(string='Withholding', compute='_compute_amount_extended', currency_field='company_currency_id')
    l10n_it_amount_before_withholding_signed = fields.Monetary(string='Total Before Withholding', compute='_compute_amount_extended', currency_field='company_currency_id')

    @api.depends('amount_total_signed')
    def _compute_amount_extended(self):
        for move in self:
            totals = dict(vat=0.0, withholding=0.0, pension_fund=0.0)
            if move.is_invoice(True):
                for line in [line for line in move.line_ids if line.tax_line_id]:
                    totals[line.tax_line_id._l10n_it_get_tax_kind()] -= line.balance
            move.l10n_it_amount_vat_signed = totals['vat']
            move.l10n_it_amount_withholding_signed = totals['withholding']
            move.l10n_it_amount_pension_fund_signed = totals['pension_fund']
            move.l10n_it_amount_before_withholding_signed = move.amount_untaxed_signed + totals['vat'] + totals['pension_fund']

    def _l10n_it_edi_filter_fatturapa_tax_details(self, line, tax_values):
        """Filters tax details to only include the positive amounted lines regarding VAT taxes."""
        repartition_line = tax_values['tax_repartition_line']
        repartition_line_vat = repartition_line.tax_id._l10n_it_filter_kind('vat')
        return repartition_line.factor_percent >= 0 and repartition_line_vat and repartition_line_vat.amount >= 0

    def _prepare_fatturapa_export_values(self):
        """Add withholding and pension_fund features."""
        template_values = super()._prepare_fatturapa_export_values()

        # Withholding tax data
        WithholdingTaxData = namedtuple('TaxData', ['tax', 'tax_amount'])
        withholding_lines = self.line_ids.filtered(lambda x: x.tax_line_id._l10n_it_filter_kind('withholding'))
        withholding_values = [WithholdingTaxData(x.tax_line_id, abs(x.balance)) for x in withholding_lines]

        # Eventually fix the total as it must be computed before applying the Withholding.
        # Withholding amount is negatively signed, so we need to subtract it
        document_total = template_values['document_total']
        document_total -= self.l10n_it_amount_withholding_signed

        # Pension fund tax data, I need the base amount so I have to sum the amounts of the lines with the tax
        PensionFundTaxData = namedtuple('TaxData', ['tax', 'base_amount', 'tax_amount', 'vat_tax', 'withholding_tax'])
        pension_fund_lines = self.line_ids.filtered(lambda line: line.tax_line_id._l10n_it_filter_kind('pension_fund'))
        pension_fund_mapping = {}
        for line in self.line_ids:
            pension_fund_tax = line.tax_ids._l10n_it_filter_kind('pension_fund')
            if pension_fund_tax:
                pension_fund_mapping[pension_fund_tax.id] = (line.tax_ids._l10n_it_filter_kind('vat'), line.tax_ids._l10n_it_filter_kind('withholding'))

        # Pension fund taxes in the XML must have a reference to their VAT tax (Aliquota tag)
        pension_fund_values = []
        enasarco_taxes = []
        for line in pension_fund_lines:
            # Enasarco must be treated separately
            if line.tax_line_id.l10n_it_pension_fund_type == 'TC07':
                enasarco_taxes.append(line.tax_line_id)
                continue
            pension_fund_tax = line.tax_line_id
            # Here we are supposing that the same pension_fund is always associated to the same VAT and Withholding taxes
            # That's also what the "Aliquota" tag seems to imply in the XML.
            vat_tax, withholding_tax = pension_fund_mapping[pension_fund_tax.id]
            pension_fund_values.append(PensionFundTaxData(pension_fund_tax, line.tax_base_amount, abs(line.balance), vat_tax, withholding_tax))

        # Enasarco pension fund must be expressed in the AltriDatiGestionali at the line detail level
        enasarco_values = False
        if enasarco_taxes:
            enasarco_values = {}
            enasarco_details = self._prepare_edi_tax_details(filter_to_apply=lambda line, tax_values: self.env['account.tax'].browse([tax_values['id']]).l10n_it_pension_fund_type == 'TC07')
            for detail in enasarco_details['tax_details_per_record'].values():
                for subdetail in detail['tax_details'].values():
                    # Withholdings are removed from the total, we have to re-add them
                    document_total += abs(subdetail['tax_amount'])
                    line = subdetail['records'].pop()
                    enasarco_values[line.id] = {
                        'amount': subdetail['tax'].amount,
                        'tax_amount': abs(subdetail['tax_amount']),
                    }

        # Update the template_values that will be read while rendering
        template_values.update({
            'withholding_values': withholding_values,
            'pension_fund_values': pension_fund_values,
            'enasarco_values': enasarco_values,
            'document_total': document_total,
        })
        return template_values

```

## File: models\account_tax.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError

TAX_KIND_SELECTION = [
    ('vat', 'VAT tax'),
    ('withholding', 'Withholding tax'),
    ('pension_fund', 'Pension Fund tax'),
]

WITHHOLDING_TYPE_SELECTION = [
    ('RT01', '[RT01] Withholding for persons'),
    ('RT02', '[RT02] Withholding for personal businesses'),
    ('RT03', '[RT03] INPS Pension fund contribution'),
    ('RT04', '[RT04] ENASARCO pension fund contribution'),
    ('RT05', '[RT05] ENPAM pension fund contribution'),
    ('RT06', '[RT06] Other pension fund contribution'),
]

WITHHOLDING_REASON_SELECTION = [
    ('A', '[A] Autonomous work in the fields of art or profession'),
    ('B', '[B] Income from the use of intellectual properties or patents or processes, formulas and informations in the fields of science, commerce or science'),
    ('C', '[C] Income from work as part of association groups or other cooperation determined by contracts'),
    ('D', '[D] Income as partner or founder of a corporation'),
    ('E', '[E] Income from client-related bill protests made by town secretaries'),
    ('G', '[G] Compensation for the end of a professional sport career'),
    ('H', '[H] Compensation for the end of a societary career (excluded those earned before 31.12.2003) and already taxed'),
    ('I', '[I] Compensation for the end of a notary career'),
    ('K', '[K] Civil service checks, ref art. 16 D.lgs. n.40 6/03/2017'),
    ('L', '[L] Income from the use of intellectual properties or patents or processes, formulas and informations in the fields of science, commerce or science, but not made by the author/inventor'),
    ('L1', '[L1] Income from the use of intellectual properties or patents or processes, formulas and informations in the fields of science, commerce or science, from someone who actively bought the use rights'),
    ('M', '[M] Autonomous work which isn\'t part of usual professional/artistic duties, or incomes due for an obligation to act, not to act, or to allow'),
    ('M1', '[M1] Incomes due for an obligation to act, not to act, or to allow'),
    ('M2', '[M2] Autonomous work which isn\'t part of usual professional/artistic duties, or incomes due for an obligation to act, not to act, or to allow - that require being registered to the "Gestione separata"'),
    ('N', '[N] Compensation for travel, expenses, prizes, or other compensations for amateur sport activities'),
    ('O', '[O] Autonomous work which isn\'t part of usual professional/artistic duties, or incomes due for an obligation to act, not to act, or to allow - that do not require being registered to the "Gestione separata"'),
    ('O1', '[O1] Incomes due for an obligation to act, not to act, or to allow - that do not require being registered to the "Gestione Separata"'),
    ('P', '[P] Compensation for people residing abroad for continuous use or concession of industrial machinery, commercial or scientific tools that are on the Italian soil'),
    ('Q', '[Q] Provisions for exclusive agents or sales representatives\' work'),
    ('R', '[R] Provisions for non-exclusive agents or sales representatives\' work'),
    ('S', '[S] Provisions for commissioner work'),
    ('T', '[T] Provisions for mediator work'),
    ('U', '[U] Provisions for procurer work'),
    ('V', '[V] Provisions for door-to-door sales persons and newspaper selling in kiosks'),
    ('V1', '[V1] Income from unusual commercial activities (such as provisions for occasional work or sales representative, mediator, procurer)'),
    ('V2', '[V2] Income from unusual work activities from door-to-door sales representatives'),
    ('W', '[W] Income from 2015 tinders subject to law art. 25-ter D.P.R. 600/1973'),
    ('X', '[X] Income from 2014 for foreign companies or institutions subject to law art. 26-quater, c. 1, lett. a) and b) D.P.R. 600/1973'),
    ('Y', '[Y] Income from 1.01.2005 to 26.07.2005 from companies or institutions not included in the description above'),
    ('Z', '[Z] Deprecated'),
    ('ZO', '[ZO] Other reason'),
]

PENSION_FUND_TYPE_SELECTION = [
    ('TC01', 'National pension fund for lawyers and solicitors'),
    ('TC02', 'Pension fund for accountants with a degree'),
    ('TC03', 'Pension fund for surveyors'),
    ('TC04', 'National pension fund for associated engineers and architects'),
    ('TC05', 'National pension fund for notaries'),
    ('TC06', 'Pension fund for accountants without a degree and commercial experts'),
    ('TC07', 'ENASARCO pension fund for sales agents'),
    ('TC08', 'ENPACL pension fund for labor consultants'),
    ('TC09', 'ENPAM pension fund for doctors'),
    ('TC10', 'ENPAF pension fund for chemists'),
    ('TC11', 'ENPAV pension fund for veterinaries'),
    ('TC12', 'ENPAIA pension fund for people working in agriculture'),
    ('TC13', 'Pension fund for employees in delivery and marine agencies'),
    ('TC14', 'INPGI pension fund for journalists'),
    ('TC15', 'ONAOSI fund for sanitary orphans'),
    ('TC16', 'CASAGIT Additional pension fund for journalists'),
    ('TC17', 'EPPI pension fund for industrial experts'),
    ('TC18', 'EPAP pension fund'),
    ('TC19', 'ENPAB national pension fund for biologists'),
    ('TC20', 'ENPAPI national pension fund for nurses'),
    ('TC21', 'ENPAP national pension fund for psychologists'),
    ('TC22', 'INPS national pension fund'),
]


class AccountTaxTemplate(models.Model):
    _inherit = 'account.tax.template'

    l10n_it_withholding_type = fields.Selection(WITHHOLDING_TYPE_SELECTION, string="Withholding tax type (Italy)", help="Withholding tax type. Only for Italian accounting EDI.")
    l10n_it_withholding_reason = fields.Selection(WITHHOLDING_REASON_SELECTION, string="Withholding tax reason (Italy)", help="Withholding tax reason. Only for Italian accounting EDI.")
    l10n_it_pension_fund_type = fields.Selection(PENSION_FUND_TYPE_SELECTION, string="Pension fund type (Italy)", help="Pension Fund Type. Only for Italian accounting EDI.")

    def _get_tax_vals(self, company, tax_template_to_tax):
        values = super()._get_tax_vals(company, tax_template_to_tax)
        values.update({
            "l10n_it_withholding_type": self.l10n_it_withholding_type,
            "l10n_it_withholding_reason": self.l10n_it_withholding_reason,
            "l10n_it_pension_fund_type": self.l10n_it_pension_fund_type,
        })
        return values

class AccountTax(models.Model):
    _inherit = 'account.tax'

    l10n_it_withholding_type = fields.Selection(WITHHOLDING_TYPE_SELECTION, string="Withholding tax type (Italy)", help="Withholding tax type. Only for Italian accounting EDI.")
    l10n_it_withholding_reason = fields.Selection(WITHHOLDING_REASON_SELECTION, string="Withholding tax reason (Italy)", help="Withholding tax reason. Only for Italian accounting EDI.")
    l10n_it_pension_fund_type = fields.Selection(PENSION_FUND_TYPE_SELECTION, string="Pension fund type (Italy)", help="Pension Fund Type. Only for Italian accounting EDI.")

    def _l10n_it_get_tax_kind(self):
        return ((self.l10n_it_withholding_type and 'withholding')
                or (self.l10n_it_pension_fund_type and 'pension_fund')
                or 'vat')

    def _l10n_it_filter_kind(self, kind):
        """ Filters taxes depending on _l10n_it_get_tax_kind. """
        return self.filtered(lambda tax: tax._l10n_it_get_tax_kind() == kind)

    @api.constrains('amount', 'l10n_it_withholding_type', 'l10n_it_withholding_reason', 'l10n_it_pension_fund_type')
    def _validate_withholding(self):
        for tax in self:
            if tax.l10n_it_withholding_type and tax.l10n_it_withholding_type != 'RT04' and tax.amount >= 0:
                raise ValidationError(_("Tax '%s' has a withholding type so the amount must be negative.", tax.name))
            if tax.l10n_it_withholding_type and not tax.l10n_it_withholding_reason:
                raise ValidationError(_("Tax '%s' has a withholding type, so the withholding reason must also be specified", tax.name))
            if tax.l10n_it_withholding_reason and not tax.l10n_it_withholding_type:
                raise ValidationError(_("Tax '%s' has a withholding reason, so the withholding type must also be specified", tax.name))
            if (tax.l10n_it_withholding_type or tax.l10n_it_withholding_reason) and tax.l10n_it_pension_fund_type:
                raise ValidationError(_("Tax '%s' cannot be both a Withholding tax and a Pension fund tax. Please create two separate ones.", tax.name))

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_tax
from . import account_chart_template
from . import account_move
from . import account_edi_format

```

## File: views\l10n_it_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_view_tax_form_l10n_it_edi_extended" model="ir.ui.view">
        <field name="name">account.tax.form.l10n.it.edi.extended</field>
        <field name="model">account.tax</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="l10n_it_edi.account_tax_form_l10n_it"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='advanced_options']" position="inside">
                <group>
                    <field name="l10n_it_withholding_type"  attrs="{'readonly': [('amount', '&gt;=', 0.0)]}"/>
                    <field name="l10n_it_withholding_reason" attrs="{'invisible': [('l10n_it_withholding_type', '=', False)]}"/>
                    <field name="l10n_it_pension_fund_type"/>
                </group>
            </xpath>
        </field>
    </record>

    <record id="view_invoice_tree_l10n_it_edi_extended" model="ir.ui.view">
        <field name="name">account.invoice.tree.l10n.it.edi.extended</field>
        <field name="model">account.move</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="account.view_invoice_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='amount_untaxed_signed']" position="after">
                <field name="l10n_it_amount_vat_signed" string="VAT" sum="Total" optional="hide"/>
                <field name="l10n_it_amount_pension_fund_signed" string="Pension Fund" sum="Total" optional="hide"/>
                <field name="l10n_it_amount_withholding_signed" string="Withholding" sum="Total" optional="hide"/>
                <field name="l10n_it_amount_before_withholding_signed" string="All Taxes Included" sum="Total" optional="hide"/>
            </xpath>
        </field>
    </record>

</odoo>

```

