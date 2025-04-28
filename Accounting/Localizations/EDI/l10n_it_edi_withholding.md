# Odoo Module: l10n_it_edi_withholding

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging
from . import models

_logger = logging.getLogger(__name__)

def _l10n_it_edi_withholding_post_init(env):
    """ Existing companies that have the Italian Chart of Accounts set """
    for company in env['res.company'].search([('chart_template', '=', 'it') , ('parent_id', '=', False)]):
        _logger.info("Company %s already has the Italian localization installed, updating...", company.name)
        ChartTemplate = env['account.chart.template'].with_company(company)
        ChartTemplate._load_data({
            'account.account': ChartTemplate._get_it_withholding_account_account(),
            'account.tax': ChartTemplate._get_it_withholding_account_tax(),
            'account.tax.group': ChartTemplate._get_it_withholding_account_tax_group(),
        })

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Italy - E-invoicing (Withholding)',
    'countries': ['it'],
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
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/accounting/fiscal_localizations/localizations/italy.html',
    'data': [
        'data/account_withholding_report_data.xml',
        'data/invoice_it_template.xml',
        'views/l10n_it_view.xml'
    ],
    'post_init_hook': '_l10n_it_edi_withholding_post_init',
    'license': 'LGPL-3',
}

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
                    <Natura t-if="pension_fund.vat_tax.l10n_it_exempt_reason" t-esc="format_alphanumeric(pension_fund.vat_tax.l10n_it_exempt_reason)"/>
                    <RiferimentoAmministrazione t-if="pension_fund.vat_tax.description" t-esc="format_alphanumeric(pension_fund.vat_tax.description, 20)"/>
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
            <t t-if="enasarco_values and enasarco_values.get(line.id)">
                <AltriDatiGestionali>
                    <TipoDato>CASSA-PREV</TipoDato>
                    <RiferimentoTesto t-esc="format_alphanumeric('TC07 - ENASARCO (' + format_numbers(abs(enasarco_values[line.id]['amount'])).rstrip('.0') + '%)')"/>
                    <RiferimentoNumero t-esc="format_monetary(enasarco_values[line.id]['tax_amount'], currency)"/>
                </AltriDatiGestionali>
            </t>
            <t t-elif="pension_fund_by_line_id and pension_fund_by_line_id.get(line.id)">
                <t t-set="pension_fund_tax" t-value="pension_fund_by_line_id[line.id]" />
                <AltriDatiGestionali>
                    <TipoDato>AswCassPre</TipoDato>
                    <RiferimentoTesto t-esc="format_alphanumeric(pension_fund_tax.l10n_it_pension_fund_type + ' (' + format_numbers(abs(pension_fund_tax.amount)).rstrip('.0') + '%)')"/>
                </AltriDatiGestionali>
            </t>
        </xpath>
    </template>
</odoo>

```

## File: data\template\account.account-it.csv

```csv
id,code,name,account_type,reconcile,tag_ids
"1611","1611","Crediti per ritenute subite (appoggio)","asset_current","False","l10n_it.account_tag_C_ATT"
"2603","2603","Debiti per ritenute da versare (appoggio)","liability_current","False","l10n_it.account_tag_D_PASS"
"2609","2609","Debiti per ritenute da versare (Fondo pensione)","liability_current","False","l10n_it.account_tag_D_PASS"
"2610","2610","Debiti per ritenute da versare (Enasarco)","liability_current","False","l10n_it.account_tag_D_PASS"
"2611","2611","Debiti per ritenute da versare","liability_current","False","l10n_it.account_tag_D_PASS"

```

## File: data\template\account.tax-it.csv

```csv
"id","description","invoice_label","name","sequence","amount","amount_type","type_tax_use","price_include","tax_group_id","active","tax_scope","l10n_it_withholding_type","l10n_it_withholding_reason","tax_exigibility","cash_basis_transition_account_id","include_base_amount","l10n_it_pension_fund_type","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","name@it"
"20vwi","","20% Ritenuta Persone Fisiche","20% RIT PF","10","-20.0","percent","sale","","tax_group_withholding","","","RT01","A","on_payment","1611","","","base","invoice","","","100",""
"","","","","","","","","","","","","","","","","","","tax","invoice","-RITV","1609","100",""
"","","","","","","","","","","","","","","","","","","base","refund","","","100",""
"","","","","","","","","","","","","","","","","","","tax","refund","+RITV","1609","100",""
"20awi","","20% Ritenuta Persone Fisiche","20% RIT PF","10","-20.0","percent","purchase","","tax_group_withholding","","","RT01","A","on_payment","2603","","","base","invoice","","","100",""
"","","","","","","","","","","","","","","","","","","tax","invoice","+RITA","2602","100",""
"","","","","","","","","","","","","","","","","","","base","refund","","","100",""
"","","","","","","","","","","","","","","","","","","tax","refund","-RITA","2602","100",""
"20vwc","","20% Ritenuta Persone Giuridiche","20% RIT PG","11","-20.0","percent","sale","","tax_group_withholding","","","RT02","A","on_payment","1611","","","base","invoice","","","100",""
"","","","","","","","","","","","","","","","","","","tax","invoice","-RITV","1609","100",""
"","","","","","","","","","","","","","","","","","","base","refund","","","100",""
"","","","","","","","","","","","","","","","","","","tax","refund","+RITV","1609","100",""
"20awc","","20% Ritenuta Persone Giuridiche","20% RIT PG","11","-20.0","percent","purchase","","tax_group_withholding","","","RT02","A","on_payment","2603","","","base","invoice","","","100",""
"","","","","","","","","","","","","","","","","","","tax","invoice","+RITA","2602","100",""
"","","","","","","","","","","","","","","","","","","base","refund","","","100",""
"","","","","","","","","","","","","","","","","","","tax","refund","-RITA","2602","100",""
"23vwo","","23% Ritenuta Agenti e Rappresentanti","23% RIT AG","12","-23.0","percent","sale","","tax_group_withholding","","","RT02","ZO","on_payment","1611","","","base","invoice","","","100",""
"","","","","","","","","","","","","","","","","","","tax","invoice","-RITV","1609","50",""
"","","","","","","","","","","","","","","","","","","base","refund","","","100",""
"","","","","","","","","","","","","","","","","","","tax","refund","+RITV","1609","50",""
"23awo","","23% Ritenuta Agenti e Rappresentanti","23% RIT AG","12","-23.0","percent","purchase","","tax_group_withholding","","","RT02","ZO","on_payment","2603","","","base","invoice","","","100",""
"","","","","","","","","","","","","","","","","","","tax","invoice","+RITA","2602","50",""
"","","","","","","","","","","","","","","","","","","base","refund","","","100",""
"","","","","","","","","","","","","","","","","","","tax","refund","-RITA","2602","50",""
"4vinps","","4% Contributo INPS","4% INPS","5","4.0","percent","sale","","tax_group_pension_fund","","","","","","","True","TC22","base","invoice","","","100",""
"","","","","","","","","","","","","","","","","","","tax","invoice","","2630","100",""
"","","","","","","","","","","","","","","","","","","base","refund","","","100",""
"","","","","","","","","","","","","","","","","","","tax","refund","","2630","100",""
"4ainps","","4% Contributo INPS","4% INPS","5","4.0","percent","purchase","","tax_group_pension_fund","","","","","","","True","TC22","base","invoice","","","100",""
"","","","","","","","","","","","","","","","","","","tax","invoice","","4402","100",""
"","","","","","","","","","","","","","","","","","","base","refund","","","100",""
"","","","","","","","","","","","","","","","","","","tax","refund","","4402","100",""
"4vcp","","4% Contributo Fondo Pensione","4% F.Pens.","20","4.0","percent","sale","","tax_group_pension_fund","","","","","","","True","TC01","base","invoice","","","100",""
"","","","","","","","","","","","","","","","","","","tax","invoice","","2630","100",""
"","","","","","","","","","","","","","","","","","","base","refund","","","100",""
"","","","","","","","","","","","","","","","","","","tax","refund","","2630","100",""
"4acp","","4% Contributo Fondo Pensione","4% F.Pens.","20","4.0","percent","purchase","","tax_group_pension_fund","","","","","","","True","TC01","base","invoice","","","100",""
"","","","","","","","","","","","","","","","","","","tax","invoice","","4402","100",""
"","","","","","","","","","","","","","","","","","","base","refund","","","100",""
"","","","","","","","","","","","","","","","","","","tax","refund","","4402","100",""
"enasarcov","","8.5% Ritenuta ENASARCO","ENASARCO","13","-8.5","percent","sale","","tax_group_enasarco","","","","","on_payment","1611","","TC07","base","invoice","","","100",""
"","","","","","","","","","","","","","","","","","","tax","invoice","-ENASARCOV","2630","100",""
"","","","","","","","","","","","","","","","","","","base","refund","","","100",""
"","","","","","","","","","","","","","","","","","","tax","refund","+ENASARCOV","2630","100",""
"enasarcoa","","8.5% Ritenuta ENASARCO","ENASARCO","13","-8.5","percent","purchase","","tax_group_enasarco","","","","","on_payment","1611","","TC07","base","invoice","","","100",""
"","","","","","","","","","","","","","","","","","","tax","invoice","+ENASARCOA","2630","100",""
"","","","","","","","","","","","","","","","","","","base","refund","","","100",""
"","","","","","","","","","","","","","","","","","","tax","refund","-ENASARCOA","2630","100",""

```

## File: data\template\account.tax.group-it.csv

```csv
"id","name","sequence","country_id","preceding_subtotal","tax_receivable_account_id","tax_payable_account_id"
"tax_group_pension_fund","Fondi Pensione","0","base.it","Fondi Pensione Esclusi","2609","2609"
"tax_group_enasarco","Enasarco","999","base.it","ENASARCO Escluso","2610","2610"
"tax_group_withholding","Ritenute","1000","base.it","Totale Ritenute Escluse","2611","2611"

```

## File: models\account_chart_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('it', 'account.account')
    def _get_it_withholding_account_account(self):
        return self._parse_csv('it', 'account.account', module='l10n_it_edi_withholding')

    @template('it', 'account.tax')
    def _get_it_withholding_account_tax(self):
        additionnal = self._parse_csv('it', 'account.tax', module='l10n_it_edi_withholding')
        self._deref_account_tags('it', additionnal)
        return additionnal

    @template('it', 'account.tax.group')
    def _get_it_withholding_account_tax_group(self):
        return self._parse_csv('it', 'account.tax.group', module='l10n_it_edi_withholding')

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from collections import namedtuple
from markupsafe import Markup
from odoo import _, api, fields, models
from odoo.addons.l10n_it_edi.models.account_move import get_float

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
            totals = {None: 0.0, 'vat':0.0, 'withholding': 0.0, 'pension_fund': 0.0}
            if move.is_invoice(True):
                for line in [line for line in move.line_ids if line.tax_line_id]:
                    kind = line.tax_line_id._l10n_it_get_tax_kind()
                    totals[kind] -= line.balance
            move.l10n_it_amount_vat_signed = totals['vat']
            move.l10n_it_amount_withholding_signed = totals['withholding']
            move.l10n_it_amount_pension_fund_signed = totals['pension_fund']
            move.l10n_it_amount_before_withholding_signed = move.amount_untaxed_signed + totals['vat'] + totals['pension_fund']

    def _l10n_it_edi_filter_tax_details(self, line, tax_values):
        """Filters tax details to only include the positive amounted lines regarding VAT taxes."""
        repartition_line = tax_values['tax_repartition_line']
        repartition_line_vat = repartition_line.tax_id._l10n_it_filter_kind('vat')
        return repartition_line.factor_percent >= 0 and repartition_line_vat and repartition_line_vat.amount >= 0

    def _l10n_it_edi_get_values(self, pdf_values=None):
        """Add withholding and pension_fund features."""
        template_values = super()._l10n_it_edi_get_values(pdf_values)

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

        # Pension fund must be expressed in the AltriDatiGestionali at the line detail level
        pension_fund_by_line_id = {}
        if pension_fund_values:
            base_lines = [
                line._convert_to_tax_base_line_dict()
                for line in self.line_ids.filtered(lambda line: line.display_type == 'product')
            ]
            for base_line in base_lines:
                for pension_fund in pension_fund_values:
                    if pension_fund.tax.id in base_line['taxes'].ids:
                        pension_fund_by_line_id[base_line['record'].id] = pension_fund.tax

        # Enasarco pension fund must be expressed in the AltriDatiGestionali at the line detail level
        enasarco_values = False
        if enasarco_taxes:
            enasarco_values = {}
            enasarco_details = self._prepare_invoice_aggregated_taxes(
                    filter_tax_values_to_apply=lambda line, tax_values: self.env['account.tax'].browse([tax_values['id']]).l10n_it_pension_fund_type == 'TC07')
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
            'pension_fund_by_line_id': pension_fund_by_line_id,
            'enasarco_values': enasarco_values,
            'document_total': document_total,
        })
        return template_values

    def _l10n_it_edi_export_taxes_data_check(self):
        """
            Override to also allow pension_fund, withholding taxes.
            Needs not to call super, because super checks for one tax only per line.
        """
        errors = []
        for invoice_line in self.invoice_line_ids.filtered(lambda x: x.display_type == 'product'):
            all_taxes = invoice_line.tax_ids.flatten_taxes_hierarchy()
            vat_taxes, withholding_taxes, pension_fund_taxes = (all_taxes._l10n_it_filter_kind(kind) for kind in
                                                                ('vat', 'withholding', 'pension_fund'))
            if len(vat_taxes.filtered(lambda x: x.amount >= 0)) != 1:
                errors.append(_("Bad tax configuration for line %s, there must be one and only one VAT tax per line", invoice_line.name))
            if len(pension_fund_taxes) > 1 or len(withholding_taxes) > 1:
                errors.append(_("Bad tax configuration for line %s, there must be one Withholding tax and one Pension Fund tax at max.", invoice_line.name))
        return errors

    # -------------------------------------------------------------------------
    # Import
    # -------------------------------------------------------------------------

    def _l10n_it_edi_search_tax_for_import(self, company, percentage, extra_domain=None, vat_only=True, l10n_it_exempt_reason=False):
        """ In case no withholding_type or pension_fund is specified, exclude taxes that have it.
            It means that we're searching for VAT taxes, especially in the base l10n_it_edi module
        """
        if vat_only:
            extra_domain = (extra_domain or []) + [('l10n_it_withholding_type', '=', False), ('l10n_it_pension_fund_type', '=', False)]
        return super()._l10n_it_edi_search_tax_for_import(company, percentage, extra_domain=extra_domain, l10n_it_exempt_reason=l10n_it_exempt_reason)

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
                message_to_log.append(Markup("%s<br/>%s") % (
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
                message_to_log.append(Markup("%s<br/>%s") % (
                    _("Pension Fund tax not found"),
                    self.env['account.move']._compose_info_message(body_tree, '.'),
                ))
        extra_info["pension_fund_taxes"] = pension_fund_taxes

        return extra_info, message_to_log

    def _l10n_it_edi_import_line(self, element, move_line_form, extra_info=None):
        messages_to_log = super()._l10n_it_edi_import_line(element, move_line_form, extra_info)

        type_tax_use_domain = extra_info['type_tax_use_domain']

        for withholding_tax in extra_info.get('withholding_taxes', []):
            withholding_tags = element.xpath("Ritenuta")
            if withholding_tags and withholding_tags[0].text == 'SI':
                move_line_form.tax_ids |= withholding_tax

        if extra_info['simplified']:
            return messages_to_log

        price_subtotal = move_line_form.price_unit
        company = move_line_form.company_id

        # Pension Funds applied on line level and ENASARCO Pension Fund tax (works as a withholding)
        for other_data_element in element.xpath('.//AltriDatiGestionali'):
            data_kind_element = other_data_element.xpath("./TipoDato")
            text_element = other_data_element.xpath("./RiferimentoTesto")
            if not data_kind_element or not text_element:
                continue
            data_kind, data_text = data_kind_element[0].text.lower(), text_element[0].text.lower()
            if data_kind == 'cassa-prev' and ('enasarco' in data_text or 'tc07' in data_text):
                number_element = other_data_element.xpath("./RiferimentoNumero")
                if not number_element or not price_subtotal:
                    continue
                enasarco_amount = float(number_element[0].text)
                enasarco_percentage = -self.env.company.currency_id.round(enasarco_amount / price_subtotal * 100)
                domain = [('l10n_it_pension_fund_type', '=', 'TC07')] + type_tax_use_domain
                if enasarco_tax := self._l10n_it_edi_search_tax_for_import(company, enasarco_percentage, domain, vat_only=False):
                    move_line_form.tax_ids |= enasarco_tax
                else:
                    messages_to_log.append(Markup("%s<br/>%s") % (
                        _("Enasarco tax not found for line with description '%s'", move_line_form.name),
                        self.env['account.move']._compose_info_message(other_data_element, '.'),
                    ))
            elif data_kind == 'aswcasspre' and 'tc' in data_text:
                for pension_fund_tax in extra_info.get('pension_fund_taxes', []):
                    if pension_fund_tax.l10n_it_pension_fund_type.lower() in data_text:
                        move_line_form.tax_ids |= pension_fund_tax

        return messages_to_log

    def _l10n_it_edi_import_invoice(self, invoice, data, is_new):
        """ Handle the case where ENASARCO pension fund contribution should be applied on the invoice globally.
        In this case, there should only be one element with ENASARCO and these conditions should be fulfilled:
         - AliquotaIVA is defined
         - PrezzoUnitario == 0.0
         - a corresponding DatiRiepilogo with the same AliquotaIVA and a ImponibileImporto
        """
        res = super()._l10n_it_edi_import_invoice(invoice=invoice, data=data, is_new=is_new)
        if not res:
            return
        self = res
        tree = data['xml_tree']
        global_enasarco_lines = []
        for additional_data_element in tree.xpath('//AltriDatiGestionali'):
            data_kind = additional_data_element.xpath('./TipoDato')[0].text.lower()
            if data_kind == 'cassa-prev':
                data_text = additional_data_element.xpath('./RiferimentoTesto')[0].text.lower()
                if 'enasarco' in data_text or 'tc07' in data_text:
                    parent_element = additional_data_element.xpath('..')[0]
                    price_unit = get_float(parent_element, './PrezzoUnitario')
                    if price_unit == 0.0:
                        global_enasarco_lines.append(parent_element)

        if len(global_enasarco_lines) == 1:
            parent_element = global_enasarco_lines[0]
            enasarco_amount = get_float(parent_element, './AltriDatiGestionali/RiferimentoNumero')
            price_unit = get_float(parent_element, './PrezzoUnitario')
            base_amount = self._get_l10_it_edi_get_taxable_amount_from_summary_data(parent_element.xpath('..')[0])
            enasarco_percentage = -self.currency_id.round(enasarco_amount / base_amount * 100) if base_amount else 0.0
            type_tax_use_domain = [('type_tax_use', '=', 'purchase' if self.is_outbound(include_receipts=True) else 'sale')]
            domain = [('l10n_it_pension_fund_type', '=', 'TC07')] + type_tax_use_domain
            if enasarco_tax := self._l10n_it_edi_search_tax_for_import(self.company_id, enasarco_percentage, domain, vat_only=False):
                to_remove_index = int(get_float(parent_element, './NumeroLinea')) - 1
                self.invoice_line_ids[to_remove_index].unlink()
                self.invoice_line_ids.tax_ids |= enasarco_tax

        return self

    def _get_l10_it_edi_get_taxable_amount_from_summary_data(self, element):
        taxable_amount = 0.0
        for summary_data_element in element.xpath('.//DatiRiepilogo'):
            taxable_amount += get_float(summary_data_element, './/ImponibileImporto')
        return taxable_amount

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


class AccountTax(models.Model):
    _inherit = 'account.tax'

    l10n_it_withholding_type = fields.Selection(WITHHOLDING_TYPE_SELECTION, string="Withholding tax type (Italy)", help="Withholding tax type. Only for Italian accounting EDI.")
    l10n_it_withholding_reason = fields.Selection(WITHHOLDING_REASON_SELECTION, string="Withholding tax reason (Italy)", help="Withholding tax reason. Only for Italian accounting EDI.")
    l10n_it_pension_fund_type = fields.Selection(PENSION_FUND_TYPE_SELECTION, string="Pension fund type (Italy)", help="Pension Fund Type. Only for Italian accounting EDI.")

    def _l10n_it_get_tax_kind(self):
        return ((self.l10n_it_withholding_type and 'withholding')
                or (self.l10n_it_pension_fund_type and 'pension_fund')
                or super()._l10n_it_get_tax_kind())

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

```

## File: views\l10n_it_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_view_tax_form_l10n_it_edi_extended" model="ir.ui.view">
        <field name="name">account.tax.form.l10n.it.edi.extended</field>
        <field name="model">account.tax</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="l10n_it.account_tax_form_l10n_it"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='advanced_options']" position="inside">
                <group>
                    <field name="l10n_it_withholding_type"  readonly="amount &gt;= 0.0"/>
                    <field name="l10n_it_withholding_reason" invisible="not l10n_it_withholding_type"/>
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

