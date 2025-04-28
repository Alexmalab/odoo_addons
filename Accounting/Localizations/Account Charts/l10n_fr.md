# Odoo Module: l10n_fr

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2008 JAILLET Simon - CrysaLEAD - www.crysalead.fr
from . import models


def _l10n_fr_post_init_hook(env):
    _preserve_tag_on_taxes(env)
    _setup_inalterability(env)

def _preserve_tag_on_taxes(env):
    from odoo.addons.account.models.chart_template import preserve_existing_tags_on_taxes
    preserve_existing_tags_on_taxes(env, 'l10n_fr')

def _setup_inalterability(env):
    # enable ping for this module
    env['publisher_warranty.contract'].update_notification(cron_mode=True)

    fr_companies = env['res.company'].search([('partner_id.country_id.code', 'in', env['res.company']._get_unalterable_country())])
    if fr_companies:
        fr_companies._create_secure_sequence(['l10n_fr_closing_sequence_id'])

        for fr_company in fr_companies:
            fr_journals = env['account.journal'].search(env['account.journal']._check_company_domain(fr_company))
            fr_journals.filtered(lambda x: not x.secure_sequence_id)._create_secure_sequence(['secure_sequence_id'])

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'France - Accounting',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations/france.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['fr'],
    'version': '2.1',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the module to manage the accounting chart for France in Odoo.
========================================================================

This module applies to companies based in France mainland. It doesn't apply to
companies based in the DOM-TOMs (Guadeloupe, Martinique, Guyane, Réunion, Mayotte).

This localisation module creates the VAT taxes of type 'tax included' for purchases
(it is notably required when you use the module 'hr_expense'). Beware that these
'tax included' VAT taxes are not managed by the fiscal positions provided by this
module (because it is complex to manage both 'tax excluded' and 'tax included'
scenarios in fiscal positions).

This localisation module doesn't properly handle the scenario when a France-mainland
company sells services to a company based in the DOMs. We could manage it in the
fiscal positions, but it would require to differentiate between 'product' VAT taxes
and 'service' VAT taxes. We consider that it is too 'heavy' to have this by default
in l10n_fr; companies that sell services to DOM-based companies should update the
configuration of their taxes and fiscal positions manually.

**Credits:** Sistheo, Zeekom, CrysaLEAD, Akretion and Camptocamp.
""",
    'depends': [
        'base_iban',
        'base_vat',
        'account',
    ],
    'data': [
        'data/account_chart_template_data.xml',
        'data/account_data.xml',
        'views/l10n_fr_view.xml',
        'data/tax_report_data.xml',
        'data/res_country_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'post_init_hook': '_l10n_fr_post_init_hook',
    'license': 'LGPL-3',
}

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_fr_tag_salaires" model="account.account.tag">
        <field name="name">Salaries</field>
        <field name="applicability">accounts</field>
    </record>

      <record id="account_fr_tag_charges_sociales" model="account.account.tag">
        <field name="name">Social charges</field>
        <field name="applicability">accounts</field>
    </record>

    <menuitem id="account_reports_fr_statements_menu" name="France" parent="account.menu_finance_reports" sequence="5" groups="account.group_account_readonly"/>

    </odoo>

```

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">
        <record forcecreate="True" id="display_name_in_footer_param" model="ir.config_parameter">
            <field name="key">account.display_name_in_footer</field>
            <field name="value">True</field>
        </record>
    </data>
</odoo>

```

## File: data\res_country_data.xml

```xml
<odoo>
    <record id="dom-tom" model="res.country.group">
             <field name="name">DOM-TOM</field>
             <field name="country_ids" eval="[(6,0,[
                                                ref('base.yt'),
                                                ref('base.gp'),
                                                ref('base.mq'),
                                                ref('base.gf'),
                                                ref('base.re'),
                                                ref('base.pf'),
                                                ref('base.pm'),
                                                ref('base.mf'),
                                                ref('base.bl'),
                                                ref('base.nc')])]"/>
       </record>

    <record id="fr_and_mc" model="res.country.group">
        <field name="name">France and Monaco</field>
        <field name="country_ids" eval="[Command.set([ref('base.fr'), ref('base.mc')])]"/>
    </record>
</odoo>

```

## File: data\tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.fr"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_montant_op_realisees" model="account.report.line">
                <field name="name">A. Amount of operations carried out</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_op_imposables_ht" model="account.report.line">
                        <field name="name">Taxable transactions (excl. VAT)</field>
                        <field name="children_ids">
                            <record id="tax_report_A1" model="account.report.line">
                                <field name="name">A1 - Sales, provision of services</field>
                                <field name="code">box_A1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_A1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">A1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_A2" model="account.report.line">
                                <field name="name">A2 - Other taxable transactions</field>
                                <field name="code">box_A2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_A2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">A2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_A3" model="account.report.line">
                                <field name="name">A3 - Intra-Community purchases of services</field>
                                <field name="code">box_A3</field>
                                <field name="expression_ids">
                                    <record id="tax_report_A3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">A3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_A4" model="account.report.line">
                                <field name="name">A4 - Imports (other than petroleum products)</field>
                                <field name="code">box_A4</field>
                                <field name="expression_ids">
                                    <record id="tax_report_A4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">A4</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_A5" model="account.report.line">
                                <field name="name">A5 - Removal from suspensive tax regime (other than petroleum products)</field>
                                <field name="code">box_A5</field>
                                <field name="expression_ids">
                                    <record id="tax_report_A5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">A5</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_B1" model="account.report.line">
                                <field name="name">B1 - Releases for consumption of petroleum products</field>
                                <field name="code">box_B1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_B1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">B1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_B2" model="account.report.line">
                                <field name="name">B2 - Intra-Community acquisitions</field>
                                <field name="code">box_B2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_B2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">B2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_B3" model="account.report.line">
                                <field name="name">B3 - Taxable supplies of electricity, natural gas, heat or cooling in France</field>
                                <field name="code">box_B3</field>
                                <field name="expression_ids">
                                    <record id="tax_report_B3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">B3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_B4" model="account.report.line">
                                <field name="name">B4 - Purchases of goods or services from a taxable person not established in France</field>
                                <field name="code">box_B4</field>
                                <field name="expression_ids">
                                    <record id="tax_report_B4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">B4</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_B5" model="account.report.line">
                                <field name="name">B5 - Regularisations</field>
                                <field name="code">box_B5</field>
                                <field name="expression_ids">
                                    <record id="tax_report_B5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">B5</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_op_non_imposables" model="account.report.line">
                        <field name="name">Untaxed operations</field>
                        <field name="children_ids">
                            <record id="tax_report_E1" model="account.report.line">
                                <field name="name">E1 - Exports outside the EU</field>
                                <field name="code">box_E1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_E1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">E1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_E2" model="account.report.line">
                                <field name="name">E2 - Other non-taxable transactions</field>
                                <field name="code">box_E2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_E2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">E2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_E3" model="account.report.line">
                                <field name="name">E3 - Distance selling taxable in another Member State to non-taxable persons</field>
                                <field name="code">box_E3</field>
                                <field name="expression_ids">
                                    <record id="tax_report_E3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">E3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_E4" model="account.report.line">
                                <field name="name">E4 - Imports (other than petroleum products)</field>
                                <field name="code">box_E4</field>
                                <field name="expression_ids">
                                    <record id="tax_report_E4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">E4</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_E5" model="account.report.line">
                                <field name="name">E5 - Removal from suspensive tax regime (other than petroleum products)</field>
                                <field name="code">box_E5</field>
                                <field name="expression_ids">
                                    <record id="tax_report_E5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">E5</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_E6" model="account.report.line">
                                <field name="name">E6 - Imports under suspensive tax arrangements (other than petroleum products)</field>
                                <field name="code">box_E6</field>
                                <field name="expression_ids">
                                    <record id="tax_report_E6_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">E6</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_F1" model="account.report.line">
                                <field name="name">F1 - Intra-Community acquisitions</field>
                                <field name="code">box_F1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_F1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">F1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_F2" model="account.report.line">
                                <field name="name">F2 - Intra-Community supplies to a taxable person</field>
                                <field name="code">box_F2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_F2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">F2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_F3" model="account.report.line">
                                <field name="name">F3 - Non-taxable supplies of electricity, natural gas, heat or cooling in France</field>
                                <field name="code">box_F3</field>
                                <field name="expression_ids">
                                    <record id="tax_report_F3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">F3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_F4" model="account.report.line">
                                <field name="name">F4 - Releases for consumption of petroleum products</field>
                                <field name="code">box_F4</field>
                                <field name="expression_ids">
                                    <record id="tax_report_F4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">F4</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_F5" model="account.report.line">
                                <field name="name">F5 - Imports of petroleum products under a suspensive tax regime</field>
                                <field name="code">box_F5</field>
                                <field name="expression_ids">
                                    <record id="tax_report_F5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">F5</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_F6" model="account.report.line">
                                <field name="name">F6 - Franchise purchases</field>
                                <field name="code">box_F6</field>
                                <field name="expression_ids">
                                    <record id="tax_report_F6_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">F6</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_F7" model="account.report.line">
                                <field name="name">F7 - Sales of goods or services by a taxable person not established in France</field>
                                <field name="code">box_F7</field>
                                <field name="expression_ids">
                                    <record id="tax_report_F7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">F7</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_F8" model="account.report.line">
                                <field name="name">F8 - Accruals</field>
                                <field name="code">box_7B</field>
                                <field name="expression_ids">
                                    <record id="tax_report_F8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">7B</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_F9" model="account.report.line">
                                <field name="name">F9 - Internal transactions between members of a single taxable person</field>
                                <field name="code">box_F9</field>
                                <field name="expression_ids">
                                    <record id="tax_report_F9_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">F9</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_decompte_tva" model="account.report.line">
                <field name="name">B. Settlement of VAT to be paid</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_tva_brute" model="account.report.line">
                        <field name="name">Gross VAT</field>
                    </record>
                    <record id="tax_report_tva_brute_metropo" model="account.report.line">
                        <field name="name">Operations carried out in mainland France</field>
                        <field name="children_ids">
                            <record id="tax_report_08_base" model="account.report.line">
                                <field name="name">08 - Standard rate 20% (base)</field>
                                <field name="code">box_08_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_08_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">08_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_08_taxe" model="account.report.line">
                                <field name="name">08 - Standard rate 20% (tax)</field>
                                <field name="code">box_08_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_08_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_08_base.balance * 0.2</field>
                                    </record>
                                    <record id="tax_report_08_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">08_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_09_base" model="account.report.line">
                                <field name="name">09 - Reduced rate 5.5% (base)</field>
                                <field name="code">box_09_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_09_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">09_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_09_taxe" model="account.report.line">
                                <field name="name">09 - Reduced rate 5.5% (tax)</field>
                                <field name="code">box_09_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_09_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_09_base.balance * 0.055</field>
                                    </record>
                                    <record id="tax_report_09_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">09_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_9B_base" model="account.report.line">
                                <field name="name">9B - Reduced rate 10% (base)</field>
                                <field name="code">box_9B_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_9B_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">9B_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_9B_taxe" model="account.report.line">
                                <field name="name">9B - Reduced rate 10% (tax)</field>
                                <field name="code">box_9B_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_9B_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_9B_base.balance * 0.1</field>
                                    </record>
                                    <record id="tax_report_9B_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">9B_taxe</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_tva_brute_dom" model="account.report.line">
                        <field name="name">Operations carried out in the DOM</field>
                        <field name="children_ids">
                            <record id="tax_report_10_base" model="account.report.line">
                                <field name="name">10 - Standard rate 8.5% (base)</field>
                                <field name="code">box_10_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_10_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">10_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_10_taxe" model="account.report.line">
                                <field name="name">10 - Standard rate 8.5% (tax)</field>
                                <field name="code">box_10_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_10_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_10_base.balance * 0.085</field>
                                    </record>
                                    <record id="tax_report_10_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">10_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_11_base" model="account.report.line">
                                <field name="name">11 - Reduced rate 2.1% (base)</field>
                                <field name="code">box_11_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_11_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">11_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_11_taxe" model="account.report.line">
                                <field name="name">11 - Reduced rate 2.1% (tax)</field>
                                <field name="code">box_11_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_11_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_11_base.balance * 0.021</field>
                                    </record>
                                    <record id="tax_report_11_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">11_taxe</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_tva_brute_autre" model="account.report.line">
                        <field name="name">Transactions taxable at another rate (metropolitan France or DOM)</field>
                        <field name="children_ids">
                            <record id="tax_report_T1_base" model="account.report.line">
                                <field name="name">T1 - Transactions carried out in the French overseas departments and taxable at 1.75% (base)</field>
                                <field name="code">box_T1_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T1_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T1_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T1_taxe" model="account.report.line">
                                <field name="name">T1 - Transactions carried out in the French overseas departments and taxable at 1.75% (tax)</field>
                                <field name="code">box_T1_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T1_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_T1_base.balance * 0.0175</field>
                                    </record>
                                    <record id="tax_report_T1_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T1_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T2_base" model="account.report.line">
                                <field name="name">T2 - Transactions carried out in the French overseas departments and taxable at 1.05% (base)</field>
                                <field name="code">box_T2_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T2_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T2_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T2_taxe" model="account.report.line">
                                <field name="name">T2 - Transactions carried out in the French overseas departments and taxable at 1,05% (tax)</field>
                                <field name="code">box_T2_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T2_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_T2_base.balance * 0.0105</field>
                                    </record>
                                    <record id="tax_report_T2_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T2_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T3_base" model="account.report.line">
                                <field name="name">T3 - Transactions carried out in Corsica and taxable at the rate of 10% (base)</field>
                                <field name="code">box_T3_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T3_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T3_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T3_taxe" model="account.report.line">
                                <field name="name">T3 - Transactions carried out in Corsica and taxable at the rate of 10% (tax)</field>
                                <field name="code">box_T3_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T3_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_T3_base.balance * 0.1</field>
                                    </record>
                                    <record id="tax_report_T3_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T3_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T4_base" model="account.report.line">
                                <field name="name">T4 - Transactions carried out in Corsica and taxable at the rate of 2,1% (base)</field>
                                <field name="code">box_T4_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T4_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T4_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T4_taxe" model="account.report.line">
                                <field name="name">T4 - Transactions carried out in Corsica and taxable at the rate of 2,1% (tax)</field>
                                <field name="code">box_T4_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T4_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_T4_base.balance * 0.021</field>
                                    </record>
                                    <record id="tax_report_T4_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T4_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T5_base" model="account.report.line">
                                <field name="name">T5 - Transactions carried out in Corsica and taxable at the rate of 0,9% (base)</field>
                                <field name="code">box_T5_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T5_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T5_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T5_taxe" model="account.report.line">
                                <field name="name">T5 - Transactions carried out in Corsica and taxable at the rate of 0,9% (tax)</field>
                                <field name="code">box_T5_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T5_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_T5_base.balance * 0.009</field>
                                    </record>
                                    <record id="tax_report_T5_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T5_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T6_base" model="account.report.line">
                                <field name="name">T6 - Transactions carried out in mainland France at the rate of 2,1% (base)</field>
                                <field name="code">box_T6_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T6_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T6_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T6_taxe" model="account.report.line">
                                <field name="name">T6 - Transactions carried out in mainland France at the rate of 2,1% (tax)</field>
                                <field name="code">box_T6_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T6_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_T6_base.balance * 0.021</field>
                                    </record>
                                    <record id="tax_report_T6_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T6_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T7_base" model="account.report.line">
                                <field name="name">T7 - Withholding of VAT on copyright (base)</field>
                                <field name="code">box_T7_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T7_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T7_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T7_taxe" model="account.report.line">
                                <field name="name">T7 - Withholding of VAT on copyright (tax)</field>
                                <field name="code">box_T7_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T7_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T7_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_13_base" model="account.report.line">
                                <field name="name">13 - Former rates (base)</field>
                                <field name="code">box_13_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_13_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">13_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_13_taxe" model="account.report.line">
                                <field name="name">13 - Former rates (tax)</field>
                                <field name="code">box_13_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_13_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">13_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_14_base" model="account.report.line">
                                <field name="name">14 - Transactions taxable at a particular rate (base)</field>
                                <field name="code">box_14_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_14_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">14_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_14_taxe" model="account.report.line">
                                <field name="name">14 - Transactions taxable at a particular rate (tax)</field>
                                <field name="code">box_14_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_14_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">14_taxe</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_tva_brute_petrolier" model="account.report.line">
                        <field name="name">Oil products</field>
                        <field name="children_ids">
                            <record id="tax_report_P1_base" model="account.report.line">
                                <field name="name">P1 - Standard rate 20% (base)</field>
                                <field name="code">box_P1_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_P1_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">P1_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_P1_taxe" model="account.report.line">
                                <field name="name">P1 - Standard rate 20% (tax)</field>
                                <field name="code">box_P1_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_P1_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_P1_base.balance * 0.2</field>
                                    </record>
                                    <record id="tax_report_P1_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">P1_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_P2_base" model="account.report.line">
                                <field name="name">P2 - Reduced rate 13% (base)</field>
                                <field name="code">box_P2_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_P2_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">P2_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_P2_taxe" model="account.report.line">
                                <field name="name">P2 - Reduced rate 13% (tax)</field>
                                <field name="code">box_P2_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_P2_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_P2_base.balance * 0.13</field>
                                    </record>
                                    <record id="tax_report_P2_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">P2_taxe</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_tva_brute_import" model="account.report.line">
                        <field name="name">Imports</field>
                        <field name="children_ids">
                            <record id="tax_report_I1_base" model="account.report.line">
                                <field name="name">I1 - Standard rate 20% (base)</field>
                                <field name="code">box_I1_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I1_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I1_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I1_taxe" model="account.report.line">
                                <field name="name">I1 - Standard rate 20% (tax)</field>
                                <field name="code">box_I1_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I1_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_I1_base.balance * 0.2</field>
                                    </record>
                                    <record id="tax_report_I1_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I1_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I2_base" model="account.report.line">
                                <field name="name">I2 - Reduced rate 10% (base)</field>
                                <field name="code">box_I2_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I2_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I2_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I2_taxe" model="account.report.line">
                                <field name="name">I2 - Reduced rate 10% (tax)</field>
                                <field name="code">box_I2_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I2_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_I2_base.balance * 0.1</field>
                                    </record>
                                    <record id="tax_report_I2_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I2_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I3_base" model="account.report.line">
                                <field name="name">I3 - Reduced rate 8.5% (base)</field>
                                <field name="code">box_I3_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I3_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I3_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I3_taxe" model="account.report.line">
                                <field name="name">I3 - Reduced rate 8.5% (tax)</field>
                                <field name="code">box_I3_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I3_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_I3_base.balance * 0.085</field>
                                    </record>
                                    <record id="tax_report_I3_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I3_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I4_base" model="account.report.line">
                                <field name="name">I4 - Reduced rate 5.5% (base)</field>
                                <field name="code">box_I4_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I4_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I4_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I4_taxe" model="account.report.line">
                                <field name="name">I4 - Reduced rate 5.5% (tax)</field>
                                <field name="code">box_I4_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I4_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_I4_base.balance * 0.055</field>
                                    </record>
                                    <record id="tax_report_I4_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I4_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I5_base" model="account.report.line">
                                <field name="name">I5 - Reduced rate 2.1% (base)</field>
                                <field name="code">box_I5_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I5_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I5_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I5_taxe" model="account.report.line">
                                <field name="name">I5 - Reduced rate 2.1% (tax)</field>
                                <field name="code">box_I5_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I5_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_I5_base.balance * 0.021</field>
                                    </record>
                                    <record id="tax_report_I5_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I5_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I6_base" model="account.report.line">
                                <field name="name">I6 - Reduced rate 1.05% (base)</field>
                                <field name="code">box_I6_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I6_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I6_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I6_taxe" model="account.report.line">
                                <field name="name">I6 - Reduced rate 1.05% (tax)</field>
                                <field name="code">box_I6_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I6_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_I6_base.balance * 0.0105</field>
                                    </record>
                                    <record id="tax_report_I6_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I6_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_15" model="account.report.line">
                                <field name="name">15 - Previously deducted VAT to be repaid</field>
                                <field name="code">box_15</field>
                                <field name="expression_ids">
                                    <record id="tax_report_15_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">15</field>
                                    </record>
                                </field>
                                <field name="children_ids">
                                    <record id="tax_report_15_1" model="account.report.line">
                                        <field name="name">of which VAT on petroleum products</field>
                                        <field name="code">box_15_1</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_15_1_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">15_1</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_15_2" model="account.report.line">
                                        <field name="name">of which VAT on imported products excluding petroleum products</field>
                                        <field name="code">box_15_2</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_15_2_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">15_2</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_5B" model="account.report.line">
                                <field name="name">5B - Amounts to be added, including advance holiday pay</field>
                                <field name="code">box_5B</field>
                                <field name="expression_ids">
                                    <record id="tax_report_5B_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">5B</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_16" model="account.report.line">
                                <field name="name">16 - Total gross VAT due</field>
                                <field name="code">box_16</field>
                                <field name="aggregation_formula">box_08_taxe.balance_from_tags + box_09_taxe.balance_from_tags + box_9B_taxe.balance_from_tags + box_10_taxe.balance_from_tags + box_11_taxe.balance_from_tags + box_13_taxe.balance + box_14_taxe.balance + box_T1_taxe.balance_from_tags + box_T2_taxe.balance_from_tags + box_T3_taxe.balance_from_tags + box_T4_taxe.balance_from_tags + box_T5_taxe.balance_from_tags + box_T6_taxe.balance_from_tags + box_T7_taxe.balance + box_P1_taxe.balance_from_tags + box_P2_taxe.balance_from_tags + box_I1_taxe.balance_from_tags + box_I2_taxe.balance_from_tags + box_I3_taxe.balance_from_tags + box_I4_taxe.balance_from_tags + box_I5_taxe.balance_from_tags + box_I6_taxe.balance_from_tags + box_15.balance + box_5B.balance</field>
                            </record>
                            <record id="tax_report_17" model="account.report.line">
                                <field name="name">17 - Of which VAT on intra-Community acquisitions</field>
                                <field name="code">box_17</field>
                                <field name="expression_ids">
                                    <record id="tax_report_17_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">17</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_18" model="account.report.line">
                                <field name="name">18 - Of which VAT on transactions to Monaco</field>
                                <field name="code">box_18</field>
                                <field name="expression_ids">
                                    <record id="tax_report_18_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">18</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_tva_deductible" model="account.report.line">
                        <field name="name">Deductible VAT</field>
                        <field name="children_ids">
                            <record id="tax_report_19" model="account.report.line">
                                <field name="name">19 - Assets constituting fixed assets</field>
                                <field name="code">box_19</field>
                                <field name="expression_ids">
                                    <record id="tax_report_19_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">19</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_20" model="account.report.line">
                                <field name="name">20 - Other goods and services</field>
                                <field name="code">box_20</field>
                                <field name="expression_ids">
                                    <record id="tax_report_20_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">20</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_21" model="account.report.line">
                                <field name="name">21 - Other deductible VAT</field>
                                <field name="code">box_21</field>
                                <field name="expression_ids">
                                    <record id="tax_report_21_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">21</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_22" model="account.report.line">
                                <field name="name">22 - Carry-over of credit from line 27 of the previous return</field>
                                <field name="code">box_22</field>
                                <field name="expression_ids">
                                    <record id="tax_report_22_applied_carryover" model="account.report.expression">
                                        <field name="label">_applied_carryover_balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="date_scope">previous_tax_period</field>
                                    </record>
                                    <record id="tax_report_22_tag" model="account.report.expression">
                                        <field name="label">tag</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">22</field>
                                    </record>
                                    <record id="tax_report_22_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_22.tag + box_22._applied_carryover_balance</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_2C" model="account.report.line">
                                <field name="name">2C - Amounts to be charged, including advance holiday pay</field>
                                <field name="code">box_2C</field>
                                <field name="expression_ids">
                                    <record id="tax_report_2C_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">2C</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_22A" model="account.report.line">
                                <field name="name">22A - Enter the single tax rate applicable for the period if different from 100%</field>
                                <field name="code">box_22A</field>
                                <field name="expression_ids">
                                    <record id="tax_report_22A_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">22A</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_23" model="account.report.line">
                                <field name="name">23 - Total deductible VAT</field>
                                <field name="code">box_23</field>
                                <field name="aggregation_formula">box_19.balance + box_20.balance + box_21.balance + box_22.balance + box_2C.balance</field>
                            </record>
                            <record id="tax_report_24" model="account.report.line">
                                <field name="name">24 - Of which deductible VAT on imports</field>
                                <field name="code">box_24</field>
                                <field name="expression_ids">
                                    <record id="tax_report_24_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">24</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_2E" model="account.report.line">
                                <field name="name">2E - Of which deductible VAT on petroleum products</field>
                                <field name="code">box_2E</field>
                                <field name="expression_ids">
                                    <record id="tax_report_2E_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">2E</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_credit" model="account.report.line">
                <field name="name">Credit</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_25" model="account.report.line">
                        <field name="name">25 - VAT credit</field>
                        <field name="code">box_25</field>
                        <field name="expression_ids">
                            <record id="tax_report_25_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">box_23.balance - box_16.balance</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_TD" model="account.report.line">
                        <field name="name">TD - VAT Due</field>
                        <field name="code">box_TD</field>
                        <field name="expression_ids">
                            <record id="tax_report_td_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">box_16.balance - box_23.balance</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_regularisation" model="account.report.line">
                <field name="name">Regularisation of domestic consumption taxes (TIC)</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_credit_constate" model="account.report.line">
                        <field name="name">Recognised credit</field>
                        <field name="children_ids">
                            <record id="tax_report_TICFE" model="account.report.line">
                                <field name="name">TICFE</field>
                                <field name="code">box_TICFE</field>
                                <field name="expression_ids">
                                    <record id="tax_report_TICFE_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TICFE_constate</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_TICGN" model="account.report.line">
                                <field name="name">TICGN</field>
                                <field name="code">box_TICGN</field>
                                <field name="expression_ids">
                                    <record id="tax_report_TICGN_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TICGN_constate</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_TICC" model="account.report.line">
                                <field name="name">TICC</field>
                                <field name="code">box_TICC</field>
                                <field name="expression_ids">
                                    <record id="tax_report_TICC_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TICC_constate</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_TIC_total" model="account.report.line">
                                <field name="name">Total</field>
                                <field name="code">box_TIC_total</field>
                                <field name="aggregation_formula">box_TICFE.balance + box_TICGN.balance + box_TICC.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_credit_impute" model="account.report.line">
                        <field name="name">Credit charged</field>
                        <field name="children_ids">
                            <record id="tax_report_X1" model="account.report.line">
                                <field name="name">X1</field>
                                <field name="code">box_X1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_X1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">X1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_X2" model="account.report.line">
                                <field name="name">X2</field>
                                <field name="code">box_X2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_X2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">X2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_X3" model="account.report.line">
                                <field name="name">X3</field>
                                <field name="code">box_X3</field>
                                <field name="expression_ids">
                                    <record id="tax_report_X3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">X3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_X4" model="account.report.line">
                                <field name="name">X4</field>
                                <field name="code">box_X4</field>
                                <field name="aggregation_formula">box_X1.balance + box_X2.balance + box_X3.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_reliquat" model="account.report.line">
                        <field name="name">Outstanding credit</field>
                        <field name="children_ids">
                            <record id="tax_report_Y1" model="account.report.line">
                                <field name="name">Y1</field>
                                <field name="code">box_Y1</field>
                                <field name="aggregation_formula">box_TICFE.balance - box_X1.balance</field>
                            </record>
                            <record id="tax_report_Y2" model="account.report.line">
                                <field name="name">Y2</field>
                                <field name="code">box_Y2</field>
                                <field name="aggregation_formula">box_TICGN.balance - box_X2.balance</field>
                            </record>
                            <record id="tax_report_Y3" model="account.report.line">
                                <field name="name">Y3</field>
                                <field name="code">box_Y3</field>
                                <field name="aggregation_formula">box_TICC.balance - box_X3.balance</field>
                            </record>
                            <record id="tax_report_Y4" model="account.report.line">
                                <field name="name">Y4</field>
                                <field name="code">box_Y4</field>
                                <field name="aggregation_formula">box_Y1.balance + box_Y2.balance + box_Y3.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_tic_tax" model="account.report.line">
                        <field name="name">Tax due</field>
                        <field name="children_ids">
                            <record id="tax_report_Z1" model="account.report.line">
                                <field name="name">Z1</field>
                                <field name="code">box_Z1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_Z1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Z1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_Z2" model="account.report.line">
                                <field name="name">Z2</field>
                                <field name="code">box_Z2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_Z2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Z2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_Z3" model="account.report.line">
                                <field name="name">Z3</field>
                                <field name="code">box_Z3</field>
                                <field name="expression_ids">
                                    <record id="tax_report_Z3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Z3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_Z4" model="account.report.line">
                                <field name="name">Z4</field>
                                <field name="code">box_Z4</field>
                                <field name="aggregation_formula">box_Z1.balance + box_Z2.balance + box_Z3.balance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_determination" model="account.report.line">
                <field name="name">Determining the amount to be paid and/or VAT and/or TIC credits</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_26" model="account.report.line">
                        <field name="name">26 - Repayment of credit requested on form n°3519 attached</field>
                        <field name="code">box_26</field>
                        <field name="expression_ids">
                            <record id="tax_report_26_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">26</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_AA" model="account.report.line">
                        <field name="name">AA - VAT credit transferred to the head company on the recapitulative return 3310-CA3G</field>
                        <field name="code">box_AA</field>
                        <field name="expression_ids">
                            <record id="tax_report_AA_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">AA</field>
                            </record>
                        </field>
                    </record>

                    <record id="tax_report_27" model="account.report.line">
                        <field name="name">27 - Credit to be carried forward</field>
                        <field name="code">box_27</field>
                        <field name="expression_ids">
                            <record id="tax_report_27_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">box_25.balance - box_26.balance - box_AA.balance</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                            <record id="tax_report_27_carryover" model="account.report.expression">
                                <field name="label">_carryover_balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">box_27.balance</field>
                                <field name="carryover_target">box_22._applied_carryover_balance</field>
                                <field name="subformula" eval="False"/>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_Y5" model="account.report.line">
                        <field name="name">Y5 - Refund of TIC balance requested (carried forward from line Y4)</field>
                        <field name="code">box_Y5</field>
                        <field name="aggregation_formula">box_Y4.balance</field>
                    </record>
                    <record id="tax_report_Y6" model="account.report.line">
                        <field name="name">Y6 - TIC credit transferred to the head company on the 3310-CA3G recapitulative return (carried forward from line Y4)</field>
                        <field name="code">box_Y6</field>
                    </record>
                    <record id="tax_report_X5" model="account.report.line">
                        <field name="name">X5 - TIC credit offset against VAT (carried forward from line X4)</field>
                        <field name="code">box_X5</field>
                        <field name="aggregation_formula">box_X4.balance</field>
                    </record>
                    <record id="tax_report_28" model="account.report.line">
                        <field name="name">28 - Net VAT due</field>
                        <field name="code">box_28</field>
                        <field name="expression_ids">
                            <record id="tax_report_28_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">box_TD.balance - box_X5.balance</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_29" model="account.report.line">
                        <field name="name">29 - Similar taxes calculated on schedule n°3310-A-SD</field>
                        <field name="code">box_29</field>
                        <field name="expression_ids">
                            <record id="tax_report_29_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">29</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_Z5" model="account.report.line">
                        <field name="name">Z5 - Total domestic consumption tax due (carried forward from line Z4)</field>
                        <field name="code">box_Z5</field>
                        <field name="aggregation_formula">box_Z4.balance</field>
                    </record>
                    <record id="tax_report_AB" model="account.report.line">
                        <field name="name">AB - Total to be paid by the head company on the recapitulative declaration 3310-CA3G</field>
                        <field name="code">box_AB</field>
                    </record>
                    <record id="tax_report_32" model="account.report.line">
                        <field name="name">32 - Total payable</field>
                        <field name="code">box_32</field>
                        <field name="aggregation_formula">box_28.balance + box_29.balance + box_Z5.balance</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-fr.csv

```csv
"id","name","code","account_type","tag_ids","reconcile","name@fr"
"pcg_1011","Subscribed capital - uncalled","101100","equity","","False","Capital souscrit - non appelé"
"pcg_1012","Subscribed capital - called up, unpaid","101200","equity","","False","Capital souscrit - appelé non versé"
"pcg_1013","Subscribed capital - called up, paid","101300","equity","","False","Capital souscrit appelé, versé"
"pcg_10131","Capital not written off","101310","equity","","False","Capital non amorti"
"pcg_10132","Capital written off","101320","equity","","False","Capital amorti"
"pcg_1018","Subscribed capital subject to particular regulations","101800","equity","","False","Capital souscrit soumis à des réglementations particulières"
"pcg_102","Trust funds","102000","equity","","False","Fonds fiduciaires"
"pcg_1041","Share premiums","104100","equity","","False","Primes d'émission"
"pcg_1042","Merger premiums","104200","equity","","False","Primes de fusion"
"pcg_1043","Contribution premiums","104300","equity","","False","Primes d'apport"
"pcg_1044","Premiums on conversion of bonds into shares","104400","equity","","False","Primes de conversion d'obligations en actions"
"pcg_1045","Equity warrants","104500","equity","","False","Bons de souscription d'actions"
"pcg_105_account","Revaluation differences","105000","equity","","False","Ecarts de réévaluation"
"pcg_1061","Legal reserve","106100","equity","","False","Réserve légale"
"pcg_1062","Undistributable reserves","106200","equity","","False","Réserves indisponibles"
"pcg_1063","Statutory or contractual reserves","106300","equity","","False","Réserves statutaires ou contractuelles"
"pcg_1064","Tax-regulated reserves","106400","equity","","False","Réserves réglementées"
"pcg_1068","Other reserves","106800","equity","","False","Autres réserves"
"pcg_107","Difference on equity accounted investments","107000","equity","","False","Écarts d'équivalence"
"pcg_108","Drawings account","108000","equity","","False","Compte de l'exploitant"
"pcg_109","Shareholders: Subscribed capital uncalled","109000","equity","","False","Actionnaires : capital souscrit - non appelé"
"pcg_110","Profit carried forward","110000","equity","","False","Report à nouveau (solde créditeur)"
"pcg_119","Loss carried forward","119000","equity","","False","Report à nouveau (solde débiteur)"
"pcg_120","Profit for the financial year","120000","equity","","False","Résultat de l'exercice (bénéfice)"
"pcg_1209","Withholding tax on dividends","120900","equity","","False","Accomptes sur les dividendes"
"pcg_129","Loss for the financial year","129000","equity","","False","Résultat de l'exercice (perte)"
"pcg_131_account","Equipment grants","131000","equity","","False","Subventions d'équipement"
"pcg_139_account","Investment grants recognised in the income statement","139000","equity","","False","Subventions d'investissement inscrites au compte de résultat"
"pcg_14_account","Regulated provisions","140000","liability_non_current","","False","Provisions réglementées"
"pcg_143_account","Regulated provisions for inventories - Price increases","143000","liability_non_current","","False","Provisions réglementées relatives aux stocks - Hausse de prix"
"pcg_145","Depreciation by derogation","145000","liability_non_current","","False","Amortissements dérogatoires"
"pcg_148","Other tax-regulated provisions","148000","liability_non_current","","False","Autres provisions réglementées"
"pcg_1511","Provisions for litigation","151100","liability_non_current","","False","Provisions pour litiges"
"pcg_1512","Provisions for customer warranties","151200","liability_non_current","","False","Provisions pour garanties données aux clients"
"pcg_1514","Provisions for fines and penalties","151400","liability_non_current","","False","Provisions pour amendes et pénalités"
"pcg_1515","Provisions for foreign exchange losses","151500","liability_non_current","","False","Provisions pour pertes de change"
"pcg_1516","Provisions for losses on contracts","151600","liability_non_current","","False","Provisions pour pertes sur contrats"
"pcg_1518","Other provisions for liabilities","151800","liability_non_current","","False","Autres provisions pour risques"
"pcg_1521_account","Provisions for pensions and similar obligations","152100","liability_non_current","","False","Provisions pour pensions et obligations similaires"
"pcg_1522_account","Provisions for restructuring","152200","liability_non_current","","False","Provisions pour restructurations"
"pcg_1523_account","Provisions for taxation","152300","liability_non_current","","False","Provisions pour impôts"
"pcg_1524_account","Provisions for fixed asset renewal - concession entities","152400","liability_non_current","","False","Provisions pour renouvellement des immobilisations - entreprises concessionnaires"
"pcg_1525_account","Provisions for major maintenance and repairs","152500","liability_non_current","","False","Provisions pour gros entretien ou grandes révisions"
"pcg_1526_account","Provisions for repairs","152600","liability_non_current","","False","Provisions pour remises en état"
"pcg_1527_account","Other provisions for charges","152700","liability_non_current","","False","Autres provisions pour charges"
"pcg_161","Convertible debenture loans","161000","liability_non_current","","False","Emprunts obligataires convertibles"
"pcg_1618","Accrued interest on convertible bonds","161800","liability_non_current","","False","Intérêts courus sur emprunts obligataires convertibles"
"pcg_162","Bonds representing net liabilities given in trust","162000","liability_non_current","","False","Obligations représentatives de passifs nets remis en fiducie"
"pcg_163","Other debenture loans","163000","liability_non_current","","False","Autres emprunts obligataires"
"pcg_1638","Accrued interest on other bonds","163800","liability_non_current","","False","Intérêts courus sur autres emprunts obligataires"
"pcg_164","Loans from credit institutions","164000","liability_non_current","","False","Emprunts auprès des établissements de crédit"
"pcg_1648","Accrued interest on bank loans","164800","liability_non_current","","False","Intérêts courus sur emprunts auprès des établissements"
"pcg_1651","Deposits","165100","liability_non_current","","False","Dépôts"
"pcg_1655","Sureties","165500","liability_non_current","","False","Cautionnements"
"pcg_1658","Accrued interest on deposits and guarantees received","165800","liability_non_current","","False","Intérêts courus sur dépôts et cautionnements reçus"
"pcg_1661","Employee profit sharing - Blocked accounts","166100","liability_non_current","","False","Participation des salariés aux résultats - Comptes bloqués"
"pcg_1662","Employee profit sharing - Profit share funds","166200","liability_non_current","","False","Participation des salariés aux résultats - Fonds de participation"
"pcg_1668","Accrued interest on employee profit-sharing","166800","liability_non_current","","False","Intérêts courus sur participation des salariés aux résultats"
"pcg_1671","Loans and debts with special conditions - Issues of non-voting shares","167100","liability_non_current","","False","Emprunts et dettes assortis de conditions particulières - Emissions de titres participatifs"
"pcg_16718","Accrued interest on redeemable shares","167180","liability_non_current","","False","Intérêts courus sur titres participatifs"
"pcg_1674","Loans and debts with special conditions - Advances by the state subject to conditions","167400","liability_non_current","","False","Emprunts et dettes assortis de conditions particulières - Avances conditionnées de l'État"
"pcg_16748","Accrued interest on conditional advances","167480","liability_non_current","","False","Intérêts courus sur avances conditionnées"
"pcg_1675","Loans and debts with special conditions - Participating loans","167500","liability_non_current","","False","Prêts participatifs"
"pcg_16758","Accrued interest on participating loans","167580","liability_non_current","","False","Intérêts courus sur emprunts participatifs"
"pcg_1681","Other loans and similar debts payable - Other loans","168100","liability_non_current","","False","Autres emprunts et dettes assimilées - Autres emprunts"
"pcg_1685","Other loans and similar debts payable - Capitalised life annuities","168500","liability_non_current","","False","Autres emprunts et dettes assimilées - Rentes viagères capitalisées"
"pcg_1687","Other loans and similar debts payable - Other debts payable","168700","liability_non_current","","False","Autres emprunts et dettes assimilées - Autres dettes"
"pcg_1688","Accrued interest","168800","liability_non_current","","False","Intérêts courus"
"pcg_169","Debt redemption premiums","169000","asset_current","","False","Primes de remboursement des obligations"
"pcg_171","Debts payable related to participating interests (group)","171000","liability_non_current","","False","Dettes rattachées à des participations (groupe)"
"pcg_174","Debts payable related to participating interests (apart from group)","174000","liability_non_current","","False","Dettes rattachées à des participations (hors groupe)"
"pcg_178_account","Debts payable related to joint ventures","178000","liability_non_current","","False","Dettes rattachées à des sociétés en participation"
"pcg_181","Reciprocal branch accounts","181000","liability_non_current","","False","Comptes de liaison des établissements"
"pcg_186","Goods and services exchanged between establishments (expenses)","186000","liability_non_current","","False","Biens et prestations de services échangés entre établissements (charges)"
"pcg_187","Goods and services exchanged between establishments (income)","187000","liability_non_current","","False","Biens et prestations de services échangés entre établissements (produits)"
"pcg_188","Reciprocal joint venture accounts","188000","liability_non_current","","False","Comptes de liaison des sociétés en participation"
"pcg_2011","Incorporation costs","201100","asset_fixed","","False","Frais de constitution"
"pcg_2012","Start-up costs","201200","asset_fixed","","False","Frais de premier établissement"
"pcg_20121","Commercial assessment costs","201210","asset_fixed","","False","Frais de prospection"
"pcg_20122","Marketing costs","201220","asset_fixed","","False","Frais de publicité"
"pcg_2013","Capital increase and sundry transaction costs (mergers, demergers, restructurations)","201300","asset_fixed","","False","Frais d'augmentation de capital et d'opérations diverses (fusions, scissions, transformations)"
"pcg_203","Research and development costs","203000","asset_fixed","","False","Frais de recherche et de développement"
"pcg_205","Concessions and similar rights, patents, licences, trade marks, processes, software, rights and similar assets","205000","asset_fixed","","False","Concessions et droits similaires, brevets, licences, marques, procédés, logiciels, droits et valeurs similaires"
"pcg_206","Lease premium","206000","asset_fixed","","False","Droit au bail"
"pcg_207","Goodwill","207000","asset_fixed","","False","Fonds commercial"
"pcg_208","Other intangible fixed assets","208000","asset_fixed","","False","Dépréciations des autres immobilisations incorporelles"
"pcg_2111","Undeveloped land","211100","asset_fixed","","False","Terrains nus"
"pcg_2112","Serviced land","211200","asset_fixed","","False","Terrains aménagés"
"pcg_2113","Basements and sub-basements","211300","asset_fixed","","False","Sous-sols et sur-sols"
"pcg_2114","Mining sites","211400","asset_fixed","","False","Terrains de carrières (Tréfonds)"
"pcg_2115","Developed land","211500","asset_fixed","","False","Terrains bâtis"
"pcg_212","Site development (same allocation as for Account 211)","212000","asset_fixed","","False","Agencements et aménagements de terrains (même ventilation que celle du compte 211)"
"pcg_2131","Buildings","213100","asset_fixed","","False","Bâtiments"
"pcg_2135","Building fixtures and fittings (same allocation as for Account 2131)","213500","asset_fixed","","False","Installations générales, agencements, aménagements des constructions (même ventilation que celle du compte 2131)"
"pcg_2138_account","Infrastructure works","213800","asset_fixed","","False","Ouvrages d'infrastructure"
"pcg_214","Constructions on third-party sites (same allocation as for Account 213)","214000","asset_fixed","","False","Constructions sur sol d'autrui (même ventilation que celle du compte 213)"
"pcg_21511","Specialised complex installations on own site","215110","asset_fixed","","False","Installations complexes spécialisées sur sol propre"
"pcg_21514","Specialised complex installations on third-party site","215140","asset_fixed","","False","Installations complexes spécialisées sur sol d'autrui"
"pcg_21531","Installations of specific nature on own site","215310","asset_fixed","","False","Installations à caractère spécifique sur sol propre"
"pcg_21534","Installations of specific nature on third-party site","215340","asset_fixed","","False","Installations à caractère spécifique sur sol d'autrui"
"pcg_2154","Plant and machinery","215400","asset_fixed","","False","Matériels industriels"
"pcg_2155","Equipment and fixtures","215500","asset_fixed","","False","Outillage industriel"
"pcg_2157","Fixtures and fittings for plant and machinery, equipment and fixtures","215700","asset_fixed","","False","Agencements et aménagements des matériels et outillage industriels"
"pcg_2181","Sundry general fixtures and fittings","218100","asset_fixed","","False","Installations générales agencements aménagements divers"
"pcg_2182","Transport equipment","218200","asset_fixed","","False","Matériel de transport"
"pcg_2183","Office and computing equipment","218300","asset_fixed","","False","Matériel de bureau et matériel informatique"
"pcg_2184","Furnishings","218400","asset_fixed","","False","Mobilier"
"pcg_2185","Livestock","218500","asset_fixed","","False","Cheptel"
"pcg_2186","Recoverable packaging","218600","asset_fixed","","False","Emballages récupérables"
"pcg_2187","Merger malpractice on tangible assets","218700","asset_fixed","","False","Mali de fusions sur actifs corporels"
"pcg_22","Fixed assets in concession","220000","asset_fixed","","False","Immobilisations mises en concession"
"pcg_231_account","Property, plant and equipment in progress","231000","asset_fixed","","False","Immobilisations corporelles en cours"
"pcg_232","Intangible fixed assets in progress","232000","asset_fixed","","False","Dépréciations des immobilisations incorporelles en cours"
"pcg_237","Payments on account on intangible fixed assets","237000","asset_fixed","","False","Avances et acomptes versés sur commandes d'immobilisations incorporelles"
"pcg_238_account","Payments on account on orders for tangible fixed assets","238000","asset_fixed","","False","Avances et acomptes versés sur commandes d'immobilisations corporelles"
"pcg_2611","Long-term equity interests - Shares","261100","asset_fixed","","False","Titres de participation - Actions"
"pcg_2618","Other securities","261800","asset_fixed","","False","Autres valeurs mobilières de placement"
"pcg_262","Equity-valued securities","262000","asset_fixed","","False","Titres évalués par équivalence"
"pcg_266","Other categories of participating interest","266000","asset_fixed","","False","Autres formes de participation"
"pcg_2661","Rights to net assets placed in trust","266100","asset_fixed","","False","Droits représentatifs d’actifs nets remis en fiducie"
"pcg_2671","Debts receivable related to participating interests - group","267100","asset_fixed","","False","Créances rattachées à des participations - groupe"
"pcg_2674","Debts receivable related to participating interests - apart from group","267400","asset_fixed","","False","Créances rattachées à des participations - hors groupe"
"pcg_2675","Payments representing non-capitalised contributions - call for funds","267500","asset_fixed","","False","Versements représentatifs d'apports non capitalisés - appel de fonds"
"pcg_2676","Long-term capital advances","267600","asset_fixed","","False","Avances consolidables"
"pcg_2677","Other debts receivable related to participating interests","267700","asset_fixed","","False","Autres créances rattachées à des participations"
"pcg_2678","Accrued interest","267800","asset_fixed","","False","Intérêts courus"
"pcg_2681","Debts receivable related to joint ventures - Principal","268100","asset_fixed","","False","Créances rattachées à des sociétés en participation - Principal"
"pcg_2688","Debts receivable related to joint ventures - Accrued interest","268800","asset_fixed","","False","Créances rattachées à des sociétés en participation - Intérêts courus"
"pcg_269","Unpaid instalments on unpaid long-term equity interests","269000","asset_fixed","","False","Versements restant à effectuer sur titres de participation non libérés"
"pcg_2711","Shares","271100","asset_fixed","","False","Actions"
"pcg_2718","Other securities","271800","asset_fixed","","False","Autres titres"
"pcg_2721","Bonds","272100","asset_fixed","","False","Obligations"
"pcg_2722","Warrants","272200","asset_fixed","","False","Bons"
"pcg_273","Portfolio long-term investment securities","273000","asset_fixed","","False","Titres immobilisés de l'activité de portefeuille"
"pcg_2741","Participating loans","274100","asset_fixed","","True","Prêts participatifs"
"pcg_2742","Loans to partners/associates","274200","asset_fixed","","True","Prêts aux associés"
"pcg_2743","Loans to personnel","274300","asset_fixed","","True","Prêts au personnel"
"pcg_2748","Other loans","274800","asset_fixed","","True","Autres prêts"
"pcg_2751","Deposits","275100","asset_fixed","","True","Dépôts"
"pcg_2755","Sureties","275500","asset_fixed","","True","Cautionnements"
"pcg_2761","Sundry debts receivable","276100","asset_fixed","","False","Créances diverses"
"pcg_27682","Accrued interest on long-term investment debt securities","276820","asset_fixed","","False","Intérêts courus sur titres immobilisés (droits de créance)"
"pcg_27684","Accrued interest on loans","276840","asset_fixed","","False","Intérêts courus sur prêts"
"pcg_27685","Accrued interest on deposits and sureties","276850","asset_fixed","","False","Intérêts courus sur dépôts et cautionnements"
"pcg_27688","Accrued interest on sundry debts receivable","276880","asset_fixed","","False","Intérêts courus sur créances diverses"
"pcg_2771","Own shares","277100","asset_fixed","","False","Actions propres ou parts propres"
"pcg_2772","Own shares in process of cancellation","277200","asset_fixed","","False","Actions propres ou parts propres en voie d'annulation"
"pcg_278","Merger loss on financial assets","278000","asset_fixed","","False","Mali de fusion sur actifs financiers"
"pcg_279","Unpaid instalments on unpaid long-term investment securities","279000","asset_fixed","","False","Versements restant à effectuer sur titres immobilisés non libérés"
"pcg_2801","Establishment costs (same allocation as for Account 201)","280100","asset_fixed","","False","Frais d'établissement (même ventilation que celle du compte 201)"
"pcg_2803","Research and development costs","280300","asset_fixed","","False","Frais de recherche et de développement"
"pcg_2805","Concessions and similar rights, patents, licences, software, rights and similar assets","280500","asset_fixed","","False","Concessions et droits similaires, brevets, licences, logiciels, droits et valeurs similaires"
"pcg_2806","Tenancy law","280600","asset_fixed","","False","Droit du bail"
"pcg_2807","Goodwill","280700","asset_fixed","","False","Fonds commercial"
"pcg_2808","Other intangible fixed assets","280800","asset_fixed","","False","Dépréciations des autres immobilisations incorporelles"
"pcg_28081","Amortisation of merger loss on intangible assets","280810","asset_fixed","","False","Amortissements du mali de fusion sur actifs incorporels"
"pcg_2812","Site development (same allocation as for Account 212)","281200","asset_fixed","","False","Agencements aménagements de terrains (même ventilation que celle du compte 212)"
"pcg_2813","Constructions (same allocation as for Account 213)","281300","asset_fixed","","False","Constructions (même ventilation que celle du compte 213)"
"pcg_2814","Constructions on third-party site (same allocation as for Account 214)","281400","asset_fixed","","False","Constructions sur sol d'autrui (même ventilation que celle du compte 214)"
"pcg_2815","Installations matériel et outillage industriels (même ventilation que celle du compte 215)","281500","asset_fixed","","False","Installations matériel et outillage industriels (même ventilation que celle du compte 215)"
"pcg_2818","Other tangible fixed assets (same allocation as for Account 218)","281800","asset_fixed","","False","Amortissements des autres immobilisations corporelles (même ventilation que celle du compte 218)"
"pcg_28187","Amortization of merger loss on tangible assets","281870","asset_fixed","","False","Amortissement du mali de fusion sur actifs corporels"
"pcg_282","Depreciation on fixed assets in concession","282000","asset_fixed","","False","Amortissements des immobilisations mises en concession"
"pcg_2901","Establishment costs","290100","asset_fixed","","False","Frais d’établissement"
"pcg_2903","Development costs","290300","asset_fixed","","False","Frais de développement"
"pcg_2905","Trade marks, processes, rights and similar assets","290500","asset_fixed","","False","Marques, procédés, droits et valeurs similaires"
"pcg_2906","Lease premium","290600","asset_fixed","","False","Droit au bail"
"pcg_2907","Goodwill","290700","asset_fixed","","False","Fonds commercial"
"pcg_2908","Other intangible fixed assets","290800","asset_fixed","","False","Dépréciations des autres immobilisations incorporelles"
"pcg_29081","Impairment of merger loss on intangible assets","290810","asset_fixed","","False","Dépréciation du mali de fusion sur actifs incorporels"
"pcg_29187","Impairment of merger loss on tangible assets","291870","asset_fixed","","False","Dépréciation du mali de fusion sur actifs corporels"
"pcg_292","Provisions for diminution in value of fixed assets in concession","292000","asset_fixed","","False","Dépréciations des immobilisations mises en concession"
"pcg_2931","Tangible fixed assets in progress","293100","asset_fixed","","False","Immobilisations corporelles en cours"
"pcg_2932","Intangible fixed assets in progress","293200","asset_fixed","","False","Dépréciations des immobilisations incorporelles en cours"
"pcg_2961","Provisions for depreciation of long-term equity interests","296100","asset_fixed","","False","Provisions pour dépréciation des titres de participation"
"pcg_2962","Equity-valued securities","296200","asset_fixed","","False","Titres évalués par équivalence"
"pcg_2966","Provisions for depreciation of other forms of participation","296600","asset_fixed","","False","Provisions pour dépréciation des autres formes de participation"
"pcg_2967","Provisions for depreciation of debts receivable related to participating interests (same allocation as for Account 267)","296700","asset_fixed","","False","Provisions pour dépréciation des créances rattachées à des participations (même ventilation que celle du compte 267)"
"pcg_2968","Provisions for depreciation of debts receivable related to joint ventures (same allocation as for Account 268)","296800","asset_fixed","","False","Provisions pour dépréciation des créances rattachées à des sociétés en participation (même ventilation que celle du compte 268)"
"pcg_2971","Provisions for depreciation of long-term investment equity securities other than portfolio long-term equity investment securities (same allocation as for Account 271)","297100","asset_fixed","","False","Provisions pour dépréciation des titres immobilisés autres que les titres immobilisés de l'activité de portefeuille - droit de propriété (ventilation : 271)"
"pcg_2972","Provisions for depreciation of long-term investment debt securities (same allocation as for Account 272)","297200","asset_fixed","","False","Provisions pour dépréciation des titres immobilisés - droit de créance (même ventilation que celle du compte 272)"
"pcg_2973","Provisions for depreciation of portfolio long-term investment securities","297300","asset_fixed","","False","Provisions pour dépréciation des titres immobilisés de l'activité de portefeuille"
"pcg_2974","Provisions for depreciation of loans (same allocation as for Account 274)","297400","asset_fixed","","False","Provisions pour dépréciation des prêts (même ventilation que celle du compte 274)"
"pcg_2975","Provisions for depreciation of deposits and sureties advanced (same allocation as for Account 275)","297500","asset_fixed","","False","Provisions pour dépréciation des dépôts et cautionnements versés (même ventilation que celle du compte 275)"
"pcg_2976","Provisions for depreciation of Other debts receivable (same allocation as for Account 276)","297600","asset_fixed","","False","Provisions pour dépréciation des autres créances immobilisées (même ventilation que celle du compte 276)"
"pcg_29787","Impairment of merger loss on financial assets","297870","asset_fixed","","False","Dépréciation du mali de fusion sur actifs financiers"
"pcg_31_account","Raw materials and supplies","310000","asset_current","","False","Matières premières et fournitures"
"pcg_321_account","Consumable materials","321000","asset_current","","False","Matières consommables"
"pcg_3221","Fuels","322100","asset_current","","False","Combustibles"
"pcg_3222","Cleaning products","322200","asset_current","","False","Produits d'entretien"
"pcg_3223","Workshop and factory supplies","322300","asset_current","","False","Fournitures d'atelier et d'usine"
"pcg_3224","Store supplies","322400","asset_current","","False","Fournitures de magasin"
"pcg_3225","Office supplies","322500","asset_current","","False","Fournitures de bureau"
"pcg_3261","Non-returnable packaging","326100","asset_current","","False","Emballages perdus"
"pcg_3265","Unidentifiable recoverable packaging","326500","asset_current","","False","Emballages récupérables non identifiables"
"pcg_3267","Mixed usage packaging","326700","asset_current","","False","Emballages à usage mixte"
"pcg_331_account","Products in progress","331000","asset_current","","False","Produits en cours"
"pcg_335_account","Works in progress","335000","asset_current","","False","Travaux en cours"
"pcg_341_account","Project studies in progress","341000","asset_current","","False","Études en cours"
"pcg_345_account","Supply of services in progress","34000","asset_current","","False","Prestations de services en cours"
"pcg_351_account","Semi-finished products","351000","asset_current","","False","Produits intermédiaires"
"pcg_355_account","Finished products","355000","asset_current","","False","Ventes de produits finis"
"pcg_3581","Waste","358100","asset_current","","False","Déchets"
"pcg_3585","Refuse","358500","asset_current","","False","Rebuts"
"pcg_3586","Recoverable materials","358600","asset_current","","False","Matières de récupération"
"pcg_36","Stocks from fixed assets","360000","asset_current","","False","Stocks provenant d'immobilisations"
"pcg_37_account","Stocks of goods","370000","asset_current","","False","Stocks de marchandises"
"pcg_38","Stocks in transit","380000","asset_current","","False","Stocks en voie d'acheminement"
"pcg_391_account","Provisions for diminution in value of raw materials and supplies","391000","asset_current","","False","Dépréciations des matières premières et fournitures"
"pcg_392_account","Provisions for diminution in value of other consumables","392000","asset_current","","False","Dépréciations des autres approvisionnements"
"pcg_393_account","Provisions for diminution in value of work in progress (goods)","393000","asset_current","","False","Dépréciations des en cours de production de biens"
"pcg_394_account","Provisions for diminution in value of work in progress (services)","394000","asset_current","","False","Dépréciations des en cours de production de services"
"pcg_395_account","Provisions for diminution in value of product stocks","395000","asset_current","","False","Dépréciations des stocks de produits"
"pcg_397_account","Provisions for diminution in value of stocks of goods for resales","397000","asset_current","","False","Dépréciations des stocks de marchandises"
"pcg_400","Suppliers and related accounts","400000","liability_payable","","True","Fournisseurs et comptes rattachés"
"fr_pcg_pay","Suppliers - Purchase of goods and services","401100","liability_payable","","True","Fournisseurs - Achats de biens et prestations de services"
"pcg_4017","Suppliers - Contract performance holdbacks","401700","liability_payable","","True","Fournisseurs - Retenues de garantie"
"pcg_403","Suppliers - Bills payable","403000","liability_payable","","True","Fournisseurs - Effets à payer"
"pcg_4041","Suppliers - Fixed asset purchases","404100","liability_payable","","True","Fournisseurs - Achats d'immobilisations"
"pcg_4047","Fixed asset suppliers - Contract performance holdbacks","404700","liability_payable","","True","Fournisseurs d'immobilisations - Retenues de garantie"
"pcg_405","Fixed asset suppliers - Bills payable","405000","liability_payable","","True","Fournisseurs d'immobilisations - Effets à payer"
"pcg_4081","Suppliers - Invoices outstanding - Suppliers","408100","liability_current","","True","Factures non parvenues - Fournisseurs"
"pcg_4084","Suppliers - Invoices outstanding - Fixed asset suppliers","408400","liability_current","","True","Factures non parvenues - Fournisseurs d'immobilisations"
"pcg_4088","Suppliers - Invoices outstanding - Suppliers - Accrued interest","408800","liability_current","","True","Factures non parvenues - Fournisseurs - Intérêts courus"
"pcg_4091","Suppliers in debit - Payments on account on orders","409100","liability_current","","False","Fournisseurs avance et acomptes versés sur commandes"
"pcg_4096","Suppliers in debit - Debts receivable for returnable packaging andequipment","409600","asset_current","","True","Fournisseurs débiteurs - Créances pour emballages et matériel à rendre"
"pcg_40971","Suppliers in debit - Other debits","409710","asset_current","","True","Fournisseurs débiteurs - Autres avoirs des fournisseurs d'exploitation"
"pcg_40974","Suppliers in debit - Other debits of fixed asset suppliers","409740","asset_current","","True","Fournisseurs débiteurs - Autres avoirs des fournisseurs d'immobilisations"
"pcg_4098","Suppliers in debit - Purchase rebates, discounts, allowances and other outstanding debits","409800","asset_current","","True","Fournisseurs débiteurs - Rabais, remises, ristournes à obtenir et autres avoirs non encore reçus"
"pcg_410","Customers and related accounts","410000","asset_receivable","","True","Clients et comptes rattachés"
"fr_pcg_recv","Customers - Sales of goods or services","411100","asset_receivable","","True","Clients - Ventes de biens ou de prestations de services"
"fr_pcg_recv_pos","Customers - Sales of goods or services (PoS)","411101","asset_receivable","","True","Clients - Ventes de biens ou de prestations de services (PoS)"
"pcg_4117","Customers - Contract performance holdbacks","411700","asset_receivable","","True","Clients - Retenues de garantie"
"pcg_413","Customers - Bills receivable","413000","asset_receivable","","True","Clients - Effets à recevoir"
"pcg_416","Doubtful or contested customer accounts","416000","asset_current","","True","Clients douteux ou litigieux"
"pcg_4181","Customers - Invoices to be made out","418100","asset_current","","True","Clients - Factures à établir"
"pcg_4188","Customers - Accrued interest","418800","asset_current","","True","Clients - Intérêts courus non encore facturés"
"pcg_4191","Customers - Payments on account received on orders","419100","liability_current","","True","Clients créditeurs - Avances et acomptes reçus sur commandes"
"pcg_4196","Customers - Debts payable for returnable packaging and equipment","419600","liability_current","","True","Clients créditeurs - Dettes pour emballages et matériels consignés"
"pcg_4197","Customers - Other credits","419700","liability_current","","True","Clients créditeurs - Autres avoirs"
"pcg_4198","Sales rebates, discounts, allowances and other credits not yet issued","419800","liability_current","","True","Clients créditeurs - Rabais, remises, ristournes à accorder et autres avoirs à établir"
"pcg_421","Personnel - Remuneration payable","421000","liability_current","","True","Personnel - Rémunérations dues"
"pcg_422","Social and Economic Committee","422000","liability_current","","True","Comité social et économique"
"pcg_4246","Employee profit share - Special reserve","424600","liability_current","","True","Participation des salariés aux résultats - Réserve spéciale"
"pcg_4248","Employee profit share - Current accounts","424800","liability_current","","True","Participation des salariés aux résultats - Comptes courants"
"pcg_425","Personnel - Payments on account","425000","asset_current","","True","Personnel - Avances et acomptes"
"pcg_426","Personnel - Deposits","426000","liability_current","","True","Personnel - Dépôts"
"pcg_427","Personnel - Stoppages of payment","427000","liability_current","","True","Personnel - Oppositions"
"pcg_4282","Personnel - Accrued charges payable for holiday pay","428200","liability_current","","True","Personnel - Dettes provisionnées pour congés à payer"
"pcg_4284","Personnel - Accrued charges payable for employee profit share","428400","liability_current","","True","Personnel - Dettes provisionnées pour participation des salariés aux résultats"
"pcg_4286","Personnel - Other accrued charges payable","428600","liability_current","","True","Personnel - Autres charges à payer"
"pcg_431","Social security","431000","liability_current","","True","Sécurité Sociale"
"pcg_437","Other social agencies","437000","liability_current","","True","Autres organismes sociaux"
"pcg_4382","Contributions for holiday pay","438200","liability_current","","True","Charges sociales sur congés à payer"
"pcg_4386","Other accrued charges payable","438600","liability_current","","True","Organismes sociaux - Autres charges à payer"
"pcg_439","Social security - Accrued income","439000","asset_current","","True","Organismes sociaux - Produits à recevoir"
"pcg_441_account","State - Subsidies and grants receivable","441000","asset_current","","True","État - Subventions et aides à recevoir"
"pcg_4421","Withholding tax (Income tax)","442100","liability_current","","True","Prélèvements à la source (Impôt sur le revenu)"
"pcg_4422","Non-liberating flat-rate deductions","442200","liability_current","","True","Prélèvements forfaitaires non libératoires"
"pcg_4423","Deductions and withholdings on distributions","442300","liability_current","","True","Retenues et prélèvements sur les distributions"
"pcg_444","State - Income tax","444000","liability_current","","True","État - Impôts sur les bénéfices"
"pcg_4452","Value added tax due within the European Union","445200","liability_current","","False","TVA due sur acquisitions intracommunautaires"
"pcg_44521","Value added tax to be disbursed","445210","liability_current","","False","TVA due sur prestations intracommunautaires"
"pcg_4453","VAT due on imports (reverse charge)","445300","liability_current","","False","TVA due sur importations (autoliquidation)"
"pcg_44531","VAT due on non-EU supplies","445310","liability_current","","False","TVA due sur prestations hors UE"
"pcg_44551","VAT to be paid","445510","liability_current","","True","TVA à décaisser"
"pcg_44558","Taxes assimilated to VAT","445580","liability_current","","True","Taxes assimilées à la TVA"
"pcg_44562","Deductible VAT on fixed assets","445620","asset_current","","False","TVA déductible sur immobilisations"
"pcg_44563","VAT transferred by other entities","445630","asset_current","","True","TVA transférée par d'autres entités"
"pcg_44564","Deductible VAT on unsettled transactions","445640","asset_current","","True","TVA déductible sur opérations non réglées"
"pcg_44566","Deductible VAT on other goods and services","445660","asset_current","","False","TVA déductible sur autres biens et services"
"pcg_445662","Intra-Community deductible VAT","445662","asset_current","","False","TVA déductible intracommunautaire"
"pcg_445663","Deductible VAT outside the EU (reverse charge)","445663","asset_current","","False","TVA déductible hors UE (autoliquidation)"
"pcg_44567","VAT credit to be carried forward","445670","asset_current","","True","Crédit de TVA à reporter"
"pcg_44568","Deductible taxes assimilated to VAT","445680","asset_current","","True","Taxes déductibles assimilées à la TVA"
"pcg_44571","VAT collected","445710","liability_current","","False","TVA collectée"
"pcg_44574","VAT collected on unsettled transactions","445740","liability_current","","False","TVA collectée sur opérations non réglées"
"pcg_44578","Taxes collected as VAT","445780","liability_current","","True","Taxes collectées assimilées à la TVA"
"pcg_445800","Turnover taxes to be regularised or pending","445800","liability_current","","True","Taxes sur le chiffre d'affaires à régulariser ou en attente"
"pcg_44581","Advance payments - Simplified taxation system","445810","asset_current","","True","Acomptes - Régime simplifié d'imposition"
"pcg_44583","Turnover tax refunds claimed","445830","asset_current","","True","Remboursement de taxes sur le chiffre d'affaires demandé"
"pcg_44584","Prepaid VAT","445840","liability_current","","True","TVA récupérée d'avance"
"pcg_44586","Turnover taxes on non-received invoices","445860","asset_current","","True","Taxes sur le chiffre d'affaires sur factures non parvenues"
"pcg_44587","Turnover taxes on invoices to be issued","445870","liability_current","","True","Taxes sur le chiffre d'affaires sur factures à établir"
"pcg_446","Guaranteed bonds","446000","liability_current","","True","Obligations cautionnées"
"pcg_447","Other taxes, levies and similar payments","447000","liability_current","","True","Autres impôts, taxes et versements assimilés"
"pcg_4481","State - Accrued charges payable","448100","liability_current","","True","État - Charges à payer"
"pcg_44811","Tax on holiday pay","448110","liability_current","","True","Charges fiscales sur congés à payer"
"pcg_4412","Accrued expenses","448120","asset_current","","True","Charges à payer"
"pcg_4482","State - Accrued income receivable","448200","asset_current","","True","État - Produits à recevoir"
"pcg_449","Emission allowances to be surrendered to the State","449000","liability_current","","True","Quotas d'émission à restituer à l'État"
"pcg_451","Group","451000","liability_current","","True","Groupe"
"pcg_4551","Partners/associates - Current accounts - Principal","455100","liability_current","","True","Associés - Comptes courants - Principal"
"pcg_4558","Partners/associates - Current accounts - Accrued interest","455800","liability_current","","True","Associés - Comptes courants - Intérêts courus"
"pcg_45611","Partners/associates - Capital transactions - Contributions in kind","456110","asset_current","","True","Associés - Comptes d'apport en société - Apports en nature"
"pcg_45615","Partners/associates - Capital transactions - Contributions in money","456150","asset_current","","True","Associés - Comptes d'apport en société - Apports en numéraire"
"pcg_45621","Shareholders - Subscribed capital calledup, unpaid","456210","asset_current","","True","Actionnaires - Capital souscrit et appelé, non versé"
"pcg_45625","Partners/associates - Capital called up, unpaid","456250","asset_current","","True","Associés - Capital appelé, non versé"
"pcg_4563","Partners/associates - Payments received for capital increase","456300","asset_current","","True","Associés - Versements reçus sur augmentation de capital"
"pcg_4564","Partners/associates - Advance payments","456400","asset_current","","True","Associés - Versements anticipés"
"pcg_4566","Defaulting shareholders","456600","asset_current","","True","Actionnaires défaillants"
"pcg_4567","Partners/associates - Capital to be reimbursed","456700","asset_current","","True","Associés - Capital à rembourser"
"pcg_457","Partners/associates - Dividends payable","457000","liability_current","","True","Associés - Dividendes à payer"
"pcg_4581","Partners/associates - Joint and Economic Interest Group transaction - Current transactions","458100","asset_current","","True","Associés - Opérations faites en commun et en GIE - Opérations courantes"
"pcg_4588","Partners/associates - Joint and Economic Interest Group transaction - Accrued interest","458800","asset_current","","True","Associés - Opérations faites en commun et en GIE - Intérêts courus"
"pcg_462","Debts receivable on realisation of fixed assets","462000","asset_current","","True","Créances sur cessions d'immobilisations"
"pcg_464","Debts payable on purchases of short-term investment securities","464000","liability_current","","True","Dettes sur acquisitions de valeurs mobilières de placement"
"pcg_465","Debts receivable on realisation of short-term investment securities","465000","asset_current","","True","Créances sur cessions de valeurs mobilières de placement"
"pcg_467","Other accounts receivable and accrued income","467000","liability_current","","True","Divers comptes débiteurs et produits à recevoir"
"pcg_468_account","Other accounts payable and accrued liabilities","468000","liability_current","","False","Divers comptes créditeurs et charges à payer"
"pcg_471","Suspense accounts","471000","asset_current","","True","Compte d'attente"
"pcg_472","Suspense accounts","472000","asset_current","","True","Compte d'attente"
"pcg_473","Suspense accounts","473000","asset_current","","True","Compte d'attente"
"pcg_4741","Valuation differences on forward financial instruments - Assets","474100","asset_current","","True","Différences d'évaluation sur instruments financiers à terme - Actif"
"pcg_4742","Valuation difference on tokens held - Assets","474200","asset_current","","True","Différences d'évaluation sur jetons détenus - Actif"
"pcg_4746","Valuation differences of tokens on liabilities - Assets","474600","asset_current","","True","Différences d’évaluation de jetons sur des passifs - Actif"
"pcg_4751","Valuation differences on forward financial instruments - Liabilities","475100","liability_current","","True","Différences d'évaluation sur instruments financiers à terme - Passif"
"pcg_4752","Valuation differences on tokens held - Liabilities","475200","liability_current","","True","Différences d'évaluation sur jetons détenus - Passif"
"pcg_4756","Valuation differences of tokens on liabilities - Liabilities","475600","liability_current","","True","Différences d’évaluation de jetons sur des passifs - Passif"
"pcg_4761","Decrease in receivables","476100","asset_current","","True","Diminution des créances"
"pcg_4762","Increase in liabilities","476200","asset_current","","True","Augmentation des dettes"
"pcg_4768","Differences offset by currency hedging","476800","asset_current","","True","Différences compensées par couverture de change"
"pcg_4771","Increase in receivables","477100","liability_current","","True","Augmentation des créances"
"pcg_4772","Decrease in liabilities","477200","liability_current","","True","Diminution des dettes"
"pcg_4778","Differences offset by currency hedging","477800","liability_current","","True","Différences compensées par couverture de change"
"pcg_478","Other transitory accounts","478000","liability_current","","False","Autres comptes transitoires"
"pcg_4781","Merger loss on current assets","478100","liability_current","","False","Mali de fusion sur actif circulant"
"pcg_481_account","Debt issuance costs","481000","asset_current","","True","Frais d’émission des emprunts"
"pcg_486","Prepayments","486000","asset_current","","True","Charges constatées d'avance"
"pcg_487","Deferred income","487000","liability_current","","True","Produits constatés d'avance"
"pcg_4886","Periodic load balancing accounts","488600","asset_current","","True","Comptes de répartition périodique des charges"
"pcg_4887","Periodic Revenue Allocation Accounts","488700","liability_current","","True","Comptes de répartition périodique des produits"
"pcg_491","Provisions for impairment of trade receivables","491000","asset_current","","True","Provisions pour dépréciation des comptes de clients"
"pcg_4951","Provisions for impairment of group accounts","495100","asset_current","","True","Provisions pour dépréciation des comptes du groupe"
"pcg_4955","Provisions for depreciation of partners' current accounts","495500","asset_current","","True","Provisions pour dépréciation des comptes courants des associés"
"pcg_4958","Provisions for depreciation of joint and EIG operations","495800","asset_current","","True","Provisions pour dépréciation des opérations faites en commun et en GIE"
"pcg_4962","Provisions for impairment of receivables on disposals of fixed assets","496200","asset_current","","True","Provisions pour dépréciation des créances sur cessions d'immobilisations"
"pcg_4965","Provisions for impairment of receivables on sales of marketable securities","496500","asset_current","","True","Provisions pour dépréciation des créances sur cessions de valeurs mobilières de placement"
"pcg_4967","Provisions for depreciation - Other accounts receivable","496700","asset_current","","True","Provisions pour dépréciation - Autres comptes débiteurs"
"pcg_502","Marketable securities - Own shares","502000","asset_cash","","False","Valeurs mobilières de placement - Actions propres"
"pcg_5021","Shares to be allocated to employees and assigned to specific plans","502100","asset_cash","","False","Actons destinées à être attribuées aux employés et affectées à des plans déterminés"
"pcg_5022","Shares available for allocation to employees or for stock market adjustment","502200","asset_cash","","False","Actons disponibles pour être attribuées aux employés ou pour la régularisation des cours de la bourse"
"pcg_5031","Marketable securities - Listed securities","503100","asset_cash","","False","Valeurs mobilières de placement - Titres cotés"
"pcg_5035","Marketable securities - Unlisted securities","503500","asset_cash","","False","Valeurs mobilières de placement - Titres non cotés"
"pcg_504","Marketable securities - Other securities conferring a right of ownership","504000","asset_cash","","False","Valeurs mobilières de placement - Autres titres conférant un droit de propriété"
"pcg_505","Bonds and notes issued by the company and redeemed by it","505000","asset_cash","","False","Obligations et bons émis par la société et rachetés par elle"
"pcg_5061","Quoted bonds","506100","asset_cash","","False","Obligations cotés"
"pcg_5065","Unquoted bonds","506500","asset_cash","","False","Obligations non cotés"
"pcg_507","Treasury bills and short-term notes","507000","asset_cash","","False","Bons du Trésor et bons de caisse à court terme"
"pcg_5081","Other securities","508100","asset_cash","","False","Autres valeurs mobilières de placement"
"pcg_5082","Equity and bond warrants","508200","asset_cash","","False","Bons de souscription"
"pcg_5088","Accrued interest on bonds, warrants and similar securities","508800","asset_cash","","False","Intérêts courus sur obligations, bons et valeurs assimilées"
"pcg_509","Unpaid instalments on unpaid short-term investment securities","509000","asset_cash","","False","Versements restant à effectuer sur valeurs mobilières de placement non libérées"
"pcg_5111","Outstanding coupons for collection","511100","asset_cash","","False","Coupons échus à l'encaissement"
"pcg_5112","Cheques for collection","511200","asset_current","","True","Chèques à encaisser"
"pcg_5113","Bills for collection","511300","asset_cash","","False","Effets à l'encaissement"
"pcg_5114","Bills for discount","511400","asset_cash","","False","Effets à l'escompte"
"pcg_5121","Banks - Accounts in euros","512100","asset_cash","","False","Comptes en euros"
"pcg_5124","Banks - Accounts in foreign currencies","512400","asset_cash","","False","Banques - Comptes en devises"
"pcg_517","Other financial bodies","517000","asset_cash","","False","Autres organismes financiers"
"pcg_5181","Accrued interest payable","518100","asset_cash","","False","Intérêts courus à payer"
"pcg_5188","Accrued interest receivable","518800","asset_cash","","False","Intérêts courus à recevoir"
"pcg_5191","Current bank advances - Credit for assignment of commercial debts receivable","519100","asset_cash","","False","Concours bancaires courants - Crédit de mobilisation de créances commerciales (CMCC)"
"pcg_5193","Current bank advances - Assignment of debts receivable originating outside France","519300","asset_cash","","False","Concours bancaires courants - Mobilisation de créances nées à l'étranger"
"pcg_5198","Current bank advances - Accrued interest on current bank advances","519800","asset_cash","","False","Concours bancaires courants - Intérêts courus sur concours bancaires courants"
"pcg_52","Financial futures instruments and tokens held","520000","asset_cash","","False","Instruments financiers à terme et jetons détenus"
"pcg_53_account","Cashier","530000","asset_cash","","False","Caisse"
"pcg_58_account","Internal transfers","580000","asset_cash","","False","Virements internes"
"pcg_5903","Provisions for impairment of shares","590300","asset_current","","False","Provisions pour dépréciation des actions"
"pcg_5904","Provisions for depreciation of other securities conferring a right of ownership","590400","asset_current","","False","Provisions pour dépréciation des autres titres conférant un droit de propriété"
"pcg_5906","Provisions for impairment of bonds","590600","asset_current","","False","Provisions pour dépréciation des obligations"
"pcg_5908","Provisions for depreciation of other marketable securities and similar receivables (provisions)","590800","asset_current","","False","Provisions pour dépréciation des autres valeurs mobilières de placement et créances assimilées (provisions)"
"pcg_601_account","Inventory item purchases - Raw materials and supplies","601000","expense","account.account_tag_operating","False","Achats stockés - Matières premières et fournitures"
"pcg_6021_account","Inventory item purchases - Consumable Materials","602100","expense","account.account_tag_operating","False","Achats stockés - Matières consommables"
"pcg_60221","Inventory item purchases - Fuels","602210","expense","account.account_tag_operating","False","Achats stockés - Combustibles"
"pcg_60222","Inventory item purchases - Maintenance products","602220","expense","account.account_tag_operating","False","Achats stockés - Produits d'entretien"
"pcg_60223","Inventory item purchases - Workshop and factory supplies","602230","expense","account.account_tag_operating","False","Achats stockés - Fournitures d'atelier et d'usine"
"pcg_60224","Inventory item purchases - Store supplies","602240","expense","account.account_tag_operating","False","Achats stockés - Fournitures de magasin"
"pcg_60225","Inventory item purchases - Office supplies","602250","expense","account.account_tag_operating","False","Achats stockés - Fournitures de bureau"
"pcg_60261","Inventory item purchases - Non-returnable packaging","602610","expense","account.account_tag_operating","False","Achats stockés - Emballages perdus"
"pcg_60262","Inventory item purchases - Packaging waste","602620","expense","account.account_tag_operating","False","Achats stockés - Malis sur emballages"
"pcg_60265","Inventory item purchases - Unidentifiable recoverable packaging","602650","expense","account.account_tag_operating","False","Achats stockés - Emballages récupérables non identifiables"
"pcg_60267","Inventory item purchases - Mixed usage packaging","602670","expense","account.account_tag_operating","False","Achats stockés - Emballages à usage mixte"
"pcg_6031","Change in stocks of raw materials (and supplies)","603100","expense","account.account_tag_operating","False","Variation des stocks de matières premières (et fournitures)"
"pcg_6032","Change in stocks of other supplies","603200","expense","account.account_tag_operating","False","Variation des stocks des autres approvisionnements"
"pcg_6037","Change in inventories of goods","603700","expense","account.account_tag_operating","False","Variation des stocks de marchandises"
"pcg_604","Purchases of project studies and services","604000","expense","account.account_tag_operating","False","Achats d'études et prestations de services"
"pcg_605","Purchases of equipment, facilities and works","605000","expense","account.account_tag_operating","False","Achats de matériel équipements et travaux"
"pcg_6061","Non-inventoriable supplies (eg. water, energy)","606100","expense","account.account_tag_operating","False","Fournitures non stockables (eau, énergie...)"
"pcg_6063","Maintenance and minor equipment supplies","606300","expense","account.account_tag_operating","False","Fournitures d'entretien et de petit équipement"
"pcg_6064","Administrative supplies","606400","expense","account.account_tag_operating","False","Fournitures administratives"
"pcg_6068","Other materials and supplies","606800","expense","account.account_tag_operating","False","Achats autres matières et fournitures"
"pcg_607_account","Purchase of goods","607000","expense","account.account_tag_operating","False","Achats de marchandises"
"pcg_608","Incidental costs included in purchases","608000","expense","","False","Frais accessoires incorporés aux achats"
"pcg_6091","Discounts, rebates and discounts - Inventory item purchases - Raw materials (and supplies)","609100","expense","account.account_tag_operating","False","Achats stockés matières premières (et fournitures)"
"pcg_6092","Discounts, rebates and discounts - Inventory item purchases - Other inventory item consumables","609200","expense","account.account_tag_operating","False","Rabais, remises et ristournes obtenus sur achats d'autres approvisionnements stockés"
"pcg_6094","Discounts, rebates and discounts - Inventory item purchases - Project studies and services supplied","609400","expense","account.account_tag_operating","False","Rabais, remises et ristournes obtenus sur achats d'études et prestations de services"
"pcg_6095","Discounts, rebates and discounts - Inventory item purchases - Equipment, facilities and works","609500","expense","account.account_tag_operating","False","Rabais, remises et ristournes obtenus sur achats de matériel, équipements et travaux"
"pcg_6096","Discounts, rebates and discounts - Inventory item purchases - Non-inventory consumables","609600","expense","account.account_tag_operating","False","Rabais, remises et ristournes obtenus sur achats d'approvisionnements non stockés"
"pcg_6097","Discounts, rebates and discounts - Inventory item purchases - Goods for resale","609700","expense","account.account_tag_operating","False","Rabais, remises et ristournes obtenus sur achats de marchandises"
"pcg_6098","Unallocated rebates, discounts, allowances","609800","expense","account.account_tag_operating","False","Rabais, remises et ristournes non affectés"
"pcg_611","General subcontracting","611000","expense","account.account_tag_operating","False","Sous-traitance générale"
"pcg_6122","Movable property leases","612200","expense","account.account_tag_operating","False","Redevances de crédit-bail mobilier"
"pcg_6125","Real property leases","612500","expense","account.account_tag_operating","False","Redevances de crédit-bail immobilier"
"pcg_6132","Real property rental","613200","expense","account.account_tag_operating","False","Locations immobilières"
"pcg_6135","Movable property rental","613500","expense","account.account_tag_operating","False","Locations mobilières"
"pcg_614","Rental and joint ownership property costs","614000","expense","account.account_tag_operating","False","Charges locatives et de copropriété"
"pcg_6152","Maintenance and repairs on real property items","615200","expense","account.account_tag_operating","False","Entretien et réparations sur biens immobiliers"
"pcg_6155","Maintenance and repairs on movable property items","615500","expense","account.account_tag_operating","False","Entretien et réparations sur biens mobiliers"
"pcg_6156","Maintenance","615600","expense","account.account_tag_operating","False","Maintenance"
"pcg_6161","Comprehensive risk","616100","expense","account.account_tag_operating","False","Assurance multirisques"
"pcg_6162","Compulsory construction loss insurance","616200","expense","account.account_tag_operating","False","Assurance obligatoire dommage construction"
"pcg_61636","Transport insurance on purchases","616360","expense","account.account_tag_operating","False","Assurance transport sur achats"
"pcg_61637","Transport insurance on sales","616370","expense","account.account_tag_operating","False","Assurance transport sur ventes"
"pcg_61638","Transport insurance on other items","616380","expense","account.account_tag_operating","False","Assurance transport sur autres biens"
"pcg_6164","Operating risks","616400","expense","account.account_tag_operating","False","Assurance risques d'exploitation"
"pcg_6165","Customer insolvency","616500","expense","account.account_tag_operating","False","Assurance insolvabilité clients"
"pcg_617","Project studies, surveys, assessments","617000","expense","account.account_tag_operating","False","Études et recherches"
"pcg_6181","General documentation","618100","expense","account.account_tag_operating","False","Documentation générale"
"pcg_6183","Technical documentation","618300","expense","account.account_tag_operating","False","Documentation technique"
"pcg_6185","Colloquium, seminar, conference costs","618500","expense","account.account_tag_operating","False","Frais de colloques, séminaires, conférences"
"pcg_619","Purchase rebates, discounts, allowances on external services","619000","expense","account.account_tag_operating","False","Rabais, remises et ristournes obtenus sur services extérieurs"
"pcg_6211","Temporary personnel","621100","expense","account.account_tag_operating","False","Personnel intérimaire"
"pcg_6214","Personnel on secondment or loan to the entity","621400","expense","account.account_tag_operating","False","Personnel détaché ou prêté à l'entreprise"
"pcg_6221","Purchase commission and brokerage","622100","expense","account.account_tag_operating","False","Commissions et courtages sur achats"
"pcg_6222","Sales commission and brokerage","622200","expense","account.account_tag_operating","False","Commissions et courtages sur ventes"
"pcg_6224","Payments to forwarding agents","622400","expense","account.account_tag_operating","False","Rémunérations des transitaires"
"pcg_6225","Payments for factoring","622500","expense","account.account_tag_operating","False","Rémunérations d'affacturage"
"pcg_6226","Fees","622600","expense","account.account_tag_operating","False","Honoraires"
"pcg_6227","Legal and litigation fees","622700","expense","account.account_tag_operating","False","Frais d'actes et de contentieux"
"pcg_6228","Remuneration of intermediaries and fees - Sundry","622800","expense","account.account_tag_operating","False","Rémunérations d'intermédiaires et honoraires - Divers"
"pcg_6231","Announcements and advertisements","623100","expense","account.account_tag_operating","False","Annonces et insertions"
"pcg_6232","Samples","623200","expense","account.account_tag_operating","False","Échantillons"
"pcg_6233","Fairs and exhibitions","623300","expense","account.account_tag_operating","False","Foires et expositions"
"pcg_6234","Gifts to customers","623400","expense","account.account_tag_operating","False","Cadeaux à la clientèle"
"pcg_6235","Premiums","623500","expense","account.account_tag_operating","False","Primes"
"pcg_6236","Catalogues and printed material","623600","expense","account.account_tag_operating","False","Catalogues et imprimés"
"pcg_6237","Publications","623700","expense","account.account_tag_operating","False","Publications"
"pcg_6238","Miscellaneous (tips, regular donations)","623800","expense","account.account_tag_operating","False","Divers (pourboires, dons courants)"
"pcg_6241","Transport on purchases","624100","expense","account.account_tag_operating","False","Transports sur achats"
"pcg_6242","Transport on sales","624200","expense","account.account_tag_operating","False","Transports sur ventes"
"pcg_6243","Transport between establishments or work sites","624300","expense","account.account_tag_operating","False","Transports entre établissements ou chantiers"
"pcg_6244","Administrative transport","624400","expense","account.account_tag_operating","False","Transports administratifs"
"pcg_6247","Collective staff transport","624700","expense","account.account_tag_operating","False","Transports collectifs du personnel"
"pcg_6248","Various transport","624800","expense","account.account_tag_operating","False","Transports divers"
"pcg_6251","Travels and journeys","625100","expense","account.account_tag_operating","False","Voyages et déplacements"
"pcg_6255","Relocation costs","625500","expense","account.account_tag_operating","False","Frais de déménagement"
"pcg_6256","Missions","625600","expense","account.account_tag_operating","False","Missions"
"pcg_6257","Receptions","625700","expense","account.account_tag_operating","False","Réceptions"
"pcg_626","Postal and telecommunication costs","626000","expense","account.account_tag_operating","False","Frais postaux et frais de télécommunications"
"pcg_6271","Securities costs (purchase, sale, safe custody)","627100","expense","account.account_tag_operating","False","Frais sur titres (achat, vente, garde)"
"pcg_6272","Commissions and loan issue costs","627200","expense","account.account_tag_operating","False","Commissions et frais sur émission d'emprunts"
"pcg_6275","Charges on bills","627500","expense","account.account_tag_operating","False","Frais sur effets"
"pcg_6276","Rental of safes","627600","expense","account.account_tag_operating","False","Location de coffres"
"pcg_6278","Other expenses and commissions on services supplied","627800","expense","account.account_tag_operating","False","Autres frais et commissions sur prestations de services"
"pcg_6281","Sundry assistance (eg. contributions)","628100","expense","account.account_tag_operating","False","Concours divers (cotisations...)"
"pcg_6284","Personnel recruitment costs","628400","expense","account.account_tag_operating","False","Frais de recrutement de personnel"
"pcg_629","Purchase rebates, discounts, allowances on other external services","629000","expense","account.account_tag_operating","False","Rabais, remises et ristournes obtenus sur autres services extérieurs"
"pcg_6311","Tax on salaries","631100","expense","account.account_tag_operating","False","Taxe sur les salaires"
"pcg_6314","Default contribution for compulsory investment in construction","631400","expense","account.account_tag_operating","False","Cotisation pour défaut d'investissement obligatoire dans la construction"
"pcg_6318","Other taxes and similar payments on remuneration (tax authorities)","631800","expense","account.account_tag_operating","False","Autres impôts, taxes et versements assimilés sur rémunérations (administrations des impôts)"
"pcg_6331","Transport expenditures","633100","expense","account.account_tag_operating","False","Versement de transport"
"pcg_6332","Accommodation allowances","633200","expense","account.account_tag_operating","False","Allocation logement"
"pcg_6333","Employers' contribution to professional training","633300","expense","account.account_tag_operating","False","Contribution unique des employeurs à la formation professionnelle"
"pcg_6334","Employer participation in construction projects","633400","expense","account.account_tag_operating","False","Participation des employeurs à l'effort de construction"
"pcg_6335","Discharge payments entitling exemption from apprenticeship tax","633500","expense","account.account_tag_operating","False","Versements libératoires ouvrant droit à l'exonération de la taxe d'apprentissage"
"pcg_6338","Other taxes and similar payments on salaries (other bodies)","633800","expense","account.account_tag_operating","False","Autres impôts, taxes et versements assimilés sur rémunérations (autres organismes)"
"pcg_63511","Territorial economic contribution","635110","expense","","False","Contribution économique territoriale"
"pcg_63512","Property taxes","635120","expense","account.account_tag_operating","False","Taxes foncières"
"pcg_63513","Other local rates and taxes","635130","expense","account.account_tag_operating","False","Autres impôts locaux"
"pcg_63514","Tax on company vehicles","635140","expense","account.account_tag_operating","False","Taxe sur les véhicules des sociétés"
"pcg_6352","Non-recoverable turnover tax","635200","expense","account.account_tag_operating","False","Taxes sur le chiffre d'affaires non récupérables"
"pcg_6353","Indirect taxes","635300","expense","account.account_tag_operating","False","Impôts indirects"
"pcg_63541","Transfer duty","635410","expense","account.account_tag_operating","False","Droits de mutation"
"pcg_6358","Other duties","635800","expense","account.account_tag_operating","False","Autres droits"
"pcg_6371","Social solidarity contribution chargeable to companies","637100","expense","account.account_tag_operating","False","Contribution sociale de solidarité à la charge des sociétés"
"pcg_6372","Taxes collected by international public bodies","637200","expense","account.account_tag_operating","False","Taxes perçues par les organismes publics internationaux"
"pcg_6374","Taxes and levies due for payment outside France","637400","expense","account.account_tag_operating","False","Impôts et taxes exigibles à l'étranger"
"pcg_6378","Sundry taxes","637800","expense","account.account_tag_operating","False","Taxes diverses (autres organismes)"
"pcg_638","Tax reminder (other than income tax)","638000","expense","account.account_tag_operating","False","Rappel d’impôts (autres qu’impôts sur les bénéfices)"
"pcg_6411","Salaries, emoluments","641100","expense","account.account_tag_operating","False","Salaires et appointements"
"pcg_6412","Holiday pay","641200","expense","account.account_tag_operating","False","Congés payés"
"pcg_6413","Premiums and bonuses","641300","expense","account.account_tag_operating","False","Primes et gratifications"
"pcg_6414","Allowances and sundry benefits","641400","expense","account.account_tag_operating","False","Indemnités et avantages divers"
"pcg_6415","Family income supplement","641500","expense","account.account_tag_operating","False","Supplément familial"
"pcg_644","Owner remuneration","644000","expense","account.account_tag_operating","False","Rémunération du travail de l'exploitant"
"pcg_6451","Social Security Collection Office (URSSAF) contributions","645100","expense","account.account_tag_operating","False","Cotisations à l'URSSAF"
"pcg_6452","Mutual organisation contributions","645200","expense","account.account_tag_operating","False","Cotisations aux mutuelles"
"pcg_6453","Pension fund contributions","645300","expense","account.account_tag_operating","False","Cotisations aux caisses de retraites"
"pcg_6454","Contributions to Pôle emploi","645400","expense","account.account_tag_operating","False","Cotisations à Pôle emploi"
"pcg_6458","Contributions to other social agencies","645800","expense","account.account_tag_operating","False","Cotisations aux autres organismes sociaux"
"pcg_646","Owner social security contributions","646000","expense","account.account_tag_operating","False","Cotisations sociales personnelles de l'exploitant"
"pcg_6471","Direct allowances","647100","expense","account.account_tag_operating","False","Prestations directes"
"pcg_6472","Payments to the social and economic committee","647200","expense","account.account_tag_operating","False","Versements au comité social et économique"
"pcg_6474","Payments to other company benefit schemes","647400","expense","account.account_tag_operating","False","Versements aux autres oeuvres sociales"
"pcg_6475","Occupational medicine, pharmacy","647500","expense","account.account_tag_operating","False","Médecine du travail, pharmacie"
"pcg_648","Other personnel costs","648000","expense","account.account_tag_operating","False","Autres charges de personnel"
"pcg_649","Reimbursement of staff costs (if staff costs are reimbursed)","649000","expense","account.account_tag_operating","False","Remboursements de charges de personnel (si rembourement de personnel)"
"pcg_6511","Concessions, patents, licences, trade marks, processes, software","651100","expense","account.account_tag_operating","False","Redevances pour concessions brevets, licences, marques, procédés, logiciels"
"pcg_6516","Author and reproduction royalties","651600","expense","account.account_tag_operating","False","Droits d'auteur et de reproduction"
"pcg_6518","Other royalties and similar assets","651800","expense","account.account_tag_operating","False","Redevances pour autres droits et valeurs similaires"
"pcg_653","Directors‘ and executive officers’ remuneration","653000","expense","account.account_tag_operating","False","Rémunérations de l’activité des administrateurs et des gérants"
"pcg_6541","Debts receivable for the financial year","654100","expense","account.account_tag_operating","False","Créances de l'exercice"
"pcg_6544","Debts receivable for previous financial years","654400","expense","account.account_tag_operating","False","Créances des exercices antérieurs"
"pcg_6551","Share of profit transferred (accounts of the managing entity)","655100","expense","","False","Quote-part de bénéfice transférée (comptabilité du gérant)"
"pcg_6555","Share of loss (accounts of non-managing partners/associates)","655500","expense","","False","Quote-part de perte supportée (comptabilité des associés non gérants)"
"pcg_657","Book value of intangible assets and property, plant and equipment sold","657000","expense","account.account_tag_investing","False","Valeurs comptables des immobilisations incorporelles et corporelles cédées"
"pcg_6581","Contract penalties (and discounts paid on purchases and sales)","658100","expense","account.account_tag_operating","False","Pénalités sur marchés (et dédits payés sur achats et ventes)"
"pcg_6582","Penalties, tax and criminal fines","658200","expense","account.account_tag_operating","False","Pénalités, amendes fiscales et pénales"
"pcg_6583","Losses resulting from indexation clauses","658300","expense","account.account_tag_operating","False","Malis provenant de clauses d'indexation"
"pcg_6584","Lots","658400","expense","account.account_tag_operating","False","Lots"
"pcg_6588","Setting up or winding up trusts","658800","expense","","False","Opérations de constitution ou liquidation des fiducies"
"pcg_66116","Loans and similar debts payable","661160","expense","account.account_tag_financing","False","Emprunts et dettes assimilées"
"pcg_66117","Debts payable related to participating interests","661170","expense","account.account_tag_financing","False","Dettes rattachées à des participations"
"pcg_6612","Trust expenses, result for the period","661200","expense","","False","Charges de la fiducie, résultat de la période"
"pcg_6615","Current account and credit deposit interest","661500","expense","account.account_tag_financing","False","Intérêts des comptes courants et des dépôts créditeurs"
"pcg_6616","Bank and financing transaction interest (eg. discounting)","661600","expense","account.account_tag_financing","False","Intérêts bancaires et sur opérations de financement (escompte, ...)"
"pcg_6617","Interest on guaranteed bonds","661700","expense","account.account_tag_financing","False","Intérêts des obligations cautionnées"
"pcg_66181","Commercial debts payable","661810","expense","account.account_tag_financing","False","Intérêts des dettes commerciales"
"pcg_66188","Sundry debts payable","661880","expense","account.account_tag_financing","False","Intérêts des dettes diverses"
"pcg_664","Losses on debts receivable related to participating interests","664000","expense","account.account_tag_financing","False","Pertes sur créances liées à des participations"
"pcg_665","Discounts allowed","665000","expense","account.account_tag_financing","False","Escomptes accordés"
"pcg_666","Exchange losses","666000","expense","account.account_tag_financing","False","Pertes de change"
"pcg_6671","Book value of financial assets sold","667100","expense","account.account_tag_financing","False","Valeurs comptables des immobilisations financières cédées"
"pcg_6672","Net expenses on disposals of portfolio securities","667200","expense","account.account_tag_financing","False","Charges nettes sur cessions de titres immobilisés de l’activité de portefeuille"
"pcg_6673","Net expenses on disposals of marketable securities","667300","expense","account.account_tag_financing","False","Charges nettes sur cessions de valeurs mobilières de placement"
"pcg_6674","Net expenses on disposal of tokens","667400","expense","account.account_tag_financing","False","Charges nettes sur cessions de jetons"
"pcg_668","Other financial charges","668000","expense","account.account_tag_financing","False","Autres charges financières"
"pcg_6683","Losses arising from the repurchase by the entity of shares and bonds issued by itself","668300","expense","account.account_tag_investing","False","Mali provenant du rachat par l’entité d’actions et obligations émises par elle-même"
"pcg_669","Transfers of financial expenses","669000","expense","account.account_tag_financing","False","Transferts de charges financières"
"pcg_672","Extraordinary expenses on previous years (during the year only)","672000","expense","","False","Charges exceptionnelles sur exercices antérieurs (en cours d'exercice seulement)"
"pcg_678_account","Other exceptional expenses","678000","expense","account.account_tag_investing","False","Autres charges exceptionnelles"
"pcg_6811","Amortisation of intangible and tangible fixed assets","681100","expense","","False","Dotations aux amortissements sur immobilisations incorporelles et corporelles"
"pcg_68111","Intangible fixed assets and formation expenses","681110","expense","account.account_tag_operating","False","Immobilisations incorporelles et frais d’établissement"
"pcg_68112","Appropriations to depreciation on Tangible fixed assets","681120","expense","account.account_tag_operating","False","Dotations aux amortissements sur immobilisations corporelles"
"pcg_6815","Appropriations to provisions for operating liabilities and charges","681500","expense","account.account_tag_operating","False","Dotations aux provisions pour risques et charges d'exploitation"
"pcg_68161","Impairment of intangible assets","681610","expense","account.account_tag_operating","False","Dotations aux dépréciations des immobilisations incorporelles"
"pcg_68162","Impairment of property, plant and equipment","681620","expense","account.account_tag_operating","False","Dotations aux dépréciations des immobilisations corporelles"
"pcg_68173","Impairment of inventories and work in progress","681730","expense","account.account_tag_operating","False","Dotations aux dépréciations des stocks et en-cours"
"pcg_68174","Impairment of receivables","681740","expense","account.account_tag_operating","False","Dotations aux dépréciations des créances"
"pcg_6861","Appropriations to amortisation of premiums on redemption of debt securities","686100","expense","account.account_tag_financing","False","Dotations aux amortissements des primes de remboursement des obligations"
"pcg_6862","Depreciation of loan issue expenses","686200","expense","account.account_tag_operating","False","Dotations aux amortissements des frais d'émission des emprunts"
"pcg_6865","Appropriations to provisions for financial liabilities and charges","686500","expense","account.account_tag_financing","False","Dotations aux provisions pour risques et charges financiers"
"pcg_68662","Appropriations to provisions for diminution in value of financial components - Financial fixed assets","686620","expense","account.account_tag_financing","False","Dotations aux dépréciations des immobilisations financières"
"pcg_68665","Appropriations to provisions for diminution in value of financial components - Short-term investment securities","686650","expense","account.account_tag_financing","False","Dotations aux dépréciations des valeurs mobilières de placement"
"pcg_6871","Appropriations to extraordinary fixed asset depreciation","687100","expense","account.account_tag_financing","False","Dotations aux amortissements exceptionnels des immobilisations"
"pcg_68725","Appropriations to tax-regulated provisions (fixed assets) - Depreciation by derogation","687250","expense","account.account_tag_financing","False","Dotations aux provisions réglementées exceptionnelles (immobilisations) - Amortissements dérogatoires"
"pcg_6873","Appropriations to tax-regulated provisions (stocks)","687300","expense","account.account_tag_financing","False","Dotations aux provisions réglementées exceptionnelles (stocks)"
"pcg_6874","Appropriations to other tax-regulated provisions","687400","expense","account.account_tag_financing","False","Dotations aux autres provisions réglementées exceptionnelles"
"pcg_6875","Appropriations to provisions for extraordinary liabilities and charges","687500","expense","account.account_tag_financing","False","Dotations aux provisions exceptionnelles"
"pcg_6876","Appropriations to provisions for extraordinary diminutionin value","687600","expense","account.account_tag_financing","False","Dotations aux dépréciations exceptionnelles"
"pcg_691","Employee profit share","691000","expense","","False","Participation des salariés aux résultats"
"pcg_6951","Income tax due in France","695100","expense","","False","Impôts sur les bénéfices dus en France"
"pcg_6952","Additional contribution to income tax","695200","expense","","False","Contribution additionnelle à l'impôt sur les bénéfices"
"pcg_6954","Income tax due outside France","695400","expense","","False","Impôts sur les bénéfices dus à l'étranger"
"pcg_696","Supplementary company tax related to profit distributions","696000","expense","","False","Supplément d'impôt sur les sociétés lié aux distributions"
"pcg_6981","Group tax - Charges","698100","expense","","False","Intégration fiscale - Charges"
"pcg_6989","Group tax - Income","698900","expense","","False","Intégration fiscale - Produits"
"pcg_699","Income - Carry-back of losses","699000","expense","","False","Produits, Reports en arrière des déficits"
"pcg_701_account","Sales of finished products","701000","income","account.account_tag_operating","False","Ventes de produits finis"
"pcg_702","Sales of semi-finished products","702000","income","account.account_tag_operating","False","Rabais, remises et ristournes sur ventes de produits intermédiaires"
"pcg_703","Sales of residual products","703000","income","account.account_tag_operating","False","Ventes de produits résiduels"
"pcg_704_account","Works","704000","income","account.account_tag_operating","False","Travaux"
"pcg_705","Project studies","705000","income","account.account_tag_operating","False","Ventes d'études"
"pcg_706","Services supplied","706000","income","account.account_tag_operating","False","Ventes de prestations de services"
"pcg_707_account","Sales of goods","707000","income","account.account_tag_operating","False","Ventes de marchandises"
"pcg_7081","Income from services operated in the interest of personne","708100","income","account.account_tag_operating","False","Produits des services exploités dans l'intérêt du personnel"
"pcg_7082","Commission and brokerage","708200","income","account.account_tag_operating","False","Commissions et courtages"
"pcg_7083","Sundry rentals","708300","income","account.account_tag_operating","False","Locations diverses"
"pcg_7084","Personnel charged out","708400","income","account.account_tag_operating","False","Mise à disposition de personnel facturée"
"pcg_7085","Carriage and ancillary costs invoiced","708500","income","account.account_tag_operating","False","Ports et frais accessoires facturés"
"pcg_7086","Surplus on recovery of returnable packaging","708600","income","account.account_tag_operating","False","Bonis sur reprises d'emballages consignés"
"pcg_7087","Bonuses obtained from customers and sales premiums","708700","income","account.account_tag_operating","False","Bonifications obtenues des clients et primes sur ventes"
"pcg_7088","Other income from ancillary activities (eg. disposal of consumables)","708800","income","account.account_tag_operating","False","Autres produits d'activités annexes (cessions d'approvisionnements...)"
"pcg_7091","Sales of finished products","709100","income","account.account_tag_operating","False","Ventes de produits finis"
"pcg_7092","Sales of semi-finished products","709200","income","account.account_tag_operating","False","Rabais, remises et ristournes sur ventes de produits intermédiaires"
"pcg_7094","Sales rebates, discounts, allowances granted by the entity on work","709400","income","account.account_tag_operating","False","Rabais, remises et ristournes sur travaux"
"pcg_7095","Sales rebates, discounts, allowances granted by the entity on Project studies","709500","income","account.account_tag_operating","False","Rabais, remises et ristournes sur études"
"pcg_7096","Sales rebates, discounts, allowances granted by the entity on Services supplied","709600","income","account.account_tag_operating","False","Rabais, remises et ristournes sur prestations de services"
"pcg_7097","Sales rebates, discounts, allowances granted by the entity on Sales of goods for resale","709700","income","account.account_tag_operating","False","Rabais, remises et ristournes sur ventes de marchandises"
"pcg_7098","Sales rebates, discounts, allowances granted by the entity on Income from ancillary activities","709800","income","","False","Rabais, remises et ristournes sur produits des activités annexes"
"pcg_71331","Change in work in progress (goods) - Products in progress","713310","income","account.account_tag_operating","False","Variation des en-cours de production de biens - Produits en cours"
"pcg_71335","Change in work in progress (goods) - Works in progress","713350","income","account.account_tag_operating","False","Variation des en-cours de production de biens - Travaux en cours"
"pcg_71341","Change in work in progress (services) - Project studies in progress","713410","income","account.account_tag_operating","False","Variation des en-cours de production de services - Études en cours"
"pcg_71345","Change in work in progress (services) - Supply of services in progress","713450","income","account.account_tag_operating","False","Variation des en-cours de production de services - Prestations de services en cours"
"pcg_71351","Change in product stocks Semi-finished products","713510","income","account.account_tag_operating","False","Variation des stocks de produits intermédiaires"
"pcg_71355","Change in product stocks Finished products","713550","income","account.account_tag_operating","False","Variation des stocks de produits finis"
"pcg_71358","Change in product stocks Residual products","713580","income","account.account_tag_operating","False","Variation des stocks de produits résiduels"
"pcg_721","Own work capitalised - Intangible fixed assets","721000","income","account.account_tag_operating","False","Production immobilisée - Immobilisations incorporelles"
"pcg_722","Own work capitalised - Tangible fixed assets","722000","income","account.account_tag_operating","False","Production immobilisée - Immobilisations corporelles"
"pcg_741","Operating subsidies","741000","income","account.account_tag_operating","False","Subventions d’exploitation"
"pcg_742","Balancing subsidies","742000","income","account.account_tag_investing","False","Subventions d’équilibre"
"pcg_747","Share of investment grants transferred to profit or loss for the year","747000","income","account.account_tag_investing","False","Quote-part des subventions d’investissement virée au résultat de l’exercice"
"pcg_7511","Royalties and licence fees for concessions, patents, licences, trade marks, processes, software","751100","income","account.account_tag_operating","False","Redevances pour concessions, brevets, licences, marques, procédés, logiciels"
"pcg_7516","Author and reproduction royalties","751600","income","account.account_tag_operating","False","Droits d'auteur et de reproduction"
"pcg_7518","Other royalties and similar assets","751800","income","account.account_tag_operating","False","Redevances pour autres droits et valeurs similaires"
"pcg_752","Revenues from buildings not allocated to professional activities","752000","income","account.account_tag_operating","False","Revenus des immeubles non affectés aux activités professionnelles"
"pcg_753","Remuneration of directors and executive managers","753000","income","account.account_tag_operating","False","Rémunérations de l’activité des administrateurs et des gérants"
"pcg_754","Rebates from cooperatives (resulting from surpluses)","754000","income","account.account_tag_operating","False","Ristournes perçues des coopératives (provenant des excédents)"
"pcg_7551","Share of loss transferred (accounts of the managing entity)","755100","income","","False","Quote-part de perte transférée (comptabilité du gérant)"
"pcg_7555","Share of profit (accounts of non-managing partners/associates)","755500","income","","False","Quote-part de bénéfice attribuée (comptabilité des associés non-gérants)"
"pcg_757","Proceeds from disposals of property, plant and equipment and intangible assets","757000","income","account.account_tag_investing","False","Produits des cessions d’immobilisations incorporelles et corporelles"
"pcg_758","Compensation and other income","758000","income","account.account_tag_operating","False","Indemnités et autres produits"
"pcg_7581","Deductions and penalties on purchases and sales","758100","income","account.account_tag_operating","False","Dédits et pénalités perçus sur achats et ventes"
"pcg_7582","Donations received","758200","income","account.account_tag_operating","False","Libéralités reçues"
"pcg_7583","Receipts on amortised receivables","758300","income","account.account_tag_operating","False",""
"pcg_7584","Tax relief other than income tax","758400","income","account.account_tag_operating","False","Dégrèvements d’impôts autres qu’impôts sur les bénéfices"
"pcg_7585","Bonuses from indexation clauses","758500","income","account.account_tag_operating","False","Bonis provenant de clauses d’indexation"
"pcg_7586","Lots","758600","income","account.account_tag_operating","False","Lots"
"pcg_7587","Insurance indemnities (if relating to insurance indemnities)","758700","income","account.account_tag_operating","False","Indemnités d’assurance (si relatif à des indemnités d'assurance)"
"pcg_7588","Setting up or liquidation of trusts","758800","income","","False","Opérations de constitution ou liquidation des fiducies"
"pcg_7611","Income from long-term equity interests","761100","income","account.account_tag_financing","False","Revenus des titres de participation"
"pcg_7612","Trust income, result for the period","761200","income","","False","Produits de la fiducie, résultat de la période"
"pcg_7616","Income from other forms of participating interests","761600","income","account.account_tag_financing","False","Revenus sur autres formes de participation"
"pcg_7617","Income from debts receivable related to participating interests","761700","income","account.account_tag_financing","False","Revenus des créances rattachées à des participations"
"pcg_7621","Income from long-term investment securities","762100","income","account.account_tag_financing","False","Revenus des titres immobilisés"
"pcg_7626","Income from loans","762600","income","account.account_tag_financing","False","Revenus des prêts"
"pcg_7627","Income from capitalised debts receivable","762700","income","account.account_tag_financing","False","Revenus des créances immobilisées"
"pcg_7631","Income from commercial debts receivable","763100","income","account.account_tag_financing","False","Revenus des créances commerciales"
"pcg_7638","Income from sundry debts receivable","763800","income","account.account_tag_financing","False","Revenus des créances diverses"
"pcg_764","Income from short-term investment securities","764000","income","account.account_tag_financing","False","Revenus des valeurs mobilières de placement"
"pcg_765","Discounts obtained","765000","income","account.account_tag_financing","False","Escomptes obtenus"
"pcg_766","Exchange gains","766000","income","account.account_tag_financing","False","Gains de change"
"pcg_7671","Income from disposals of financial assets","767100","income","account.account_tag_investing","False","Produits des cessions d’immobilisations financières"
"pcg_7672","Net income from disposals of portfolio securities","767200","income","account.account_tag_investing","False","Produits nets sur cessions de titres immobilisés de l’activité de portefeuille"
"pcg_7673","Net proceeds from disposals of marketable securities","767300","income","account.account_tag_financing","False","Produits nets sur cessions de valeurs mobilières de placement"
"pcg_7674","Net income on disposal of tokens","767400","income","account.account_tag_financing","False","Produits nets sur cessions de jetons"
"pcg_7683","Bonuses arising from the repurchase by the company of shares and bonds issued by itself","768300","income","account.account_tag_investing","False","Bonis provenant du rachat par l’entreprise d’actions et d’obligations émises par elle-même"
"pcg_772","Extraordinary income on previous years (during the year only)","772000","income","","False","Produits exceptionnels sur exercices antérieurs (en cours d'exercice seulement)"
"pcg_778_account","Other extraordinary income","778000","income","account.account_tag_investing","False","Autres produits exceptionnels"
"pcg_7811","Reversals of amortisation of intangible assets and property, plant and equipment","781100","income","","False","Reprises sur amortissements des immobilisations incorporelles et corporelles"
"pcg_78111","Reversal of amortisation of intangible assets","781110","income","account.account_tag_operating","False","Reprises sur amortissements des immobilisations incorporelles"
"pcg_78112","Reversals of depreciation on tangible fixed assets","781120","income","account.account_tag_operating","False","Reprises sur amortissements des immobilisations corporelles"
"pcg_7815","Provisions for operating liabilities and charges written back","781500","income","account.account_tag_operating","False","Reprises sur provisions d'exploitation"
"pcg_7816","Provisions for diminution in value of intangible and tangible fixed assets written back","781600","income","","False","Reprises sur dépréciations des immobilisations incorporelles et corporelles"
"pcg_78161","Reversals of impairment of intangible assets","781610","income","account.account_tag_operating","False","Reprises sur dépréciations des immobilisations incorporelles"
"pcg_78162","Reversals of impairment of property, plant and equipment","781620","income","account.account_tag_operating","False","Reprises sur dépréciations des immobilisations corporelles"
"pcg_7817","Reversals of impairment of current assets","781700","income","","False","Reprises sur dépréciations des actifs circulants"
"pcg_78173","Reversals of impairment of current assets - Inventories and work in progress","781730","income","account.account_tag_operating","False","Reprises sur dépréciations des actifs circulants - Stocks et en-cours"
"pcg_78174","Reversals of impairment of current assets - Receivables","781740","income","account.account_tag_operating","False","Reprises sur dépréciations des actifs circulants - Créances"
"pcg_7865","Reversals of financial provisions","786500","income","account.account_tag_financing","False","Reprises sur provisions financières"
"pcg_78662","Reversals of impairment of financial assets","786620","income","account.account_tag_financing","False","Reprises sur dépréciations des immobilisations financières"
"pcg_78665","Reversals of impairment of marketable securities","786650","income","account.account_tag_financing","False","Reprises sur dépréciations des valeurs mobilières de placement"
"pcg_78725","Reversals of regulated provisions (fixed assets) - Accelerated depreciation","787250","income","account.account_tag_investing","False","Reprises sur provisions réglementées (immobilisations) - Amortissements dérogatoires"
"pcg_7873","Reversals of regulated provisions (stocks)","787300","income","account.account_tag_investing","False","Reprises sur provisions réglementées (stocks)"
"pcg_7874","Reversals of other regulated provisions","787400","income","account.account_tag_investing","False","Reprises sur autres provisions réglementées"
"pcg_7875","Reversals of exceptional provisions","787500","income","account.account_tag_investing","False","Reprises sur provisions exceptionnelles"
"pcg_7876","Reversals of exceptional depreciation","787600","income","account.account_tag_investing","False","Reprises sur dépréciations exceptionnelles"

```

## File: data\template\account.fiscal.position-fr.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@fr"
"fiscal_position_template_domestic","1","Domestic - France","1","1","","l10n_fr.fr_and_mc","","","Domestique - France"
"fiscal_position_template_intraeub2c","2","EU private","1","","","base.europe","","","EU privé"
"fiscal_position_template_intraeub2b","3","Intra-EU B2B","1","1","","base.europe","tva_normale","tva_sale_good_intra_0","Intra-EU B2B"
"","","","","","","","tva_normale_ttc","tva_sale_good_intra_0",""
"","","","","","","","tva_intermediaire_encaissement","tva_sale_service_intra_0",""
"","","","","","","","tva_normale_encaissement","tva_sale_service_intra_0",""
"","","","","","","","tva_normale_encaissement_ttc","tva_sale_service_intra_0",""
"","","","","","","","tva_intermediaire_encaissement_ttc","tva_sale_service_intra_0",""
"","","","","","","","tva_specifique","tva_sale_good_intra_0",""
"","","","","","","","tva_specifique_ttc","tva_sale_good_intra_0",""
"","","","","","","","tva_intermediaire","tva_sale_good_intra_0",""
"","","","","","","","tva_intermediaire_ttc","tva_sale_good_intra_0",""
"","","","","","","","tva_reduite","tva_sale_good_intra_0",""
"","","","","","","","tva_reduite_ttc","tva_sale_good_intra_0",""
"","","","","","","","tva_reduite_encaissement_ttc","tva_sale_service_intra_0",""
"","","","","","","","tva_reduite_encaissement","tva_sale_service_intra_0",""
"","","","","","","","tva_super_reduite","tva_sale_good_intra_0",""
"","","","","","","","tva_super_reduite_ttc","tva_sale_good_intra_0",""
"","","","","","","","tva_super_reduite_encaissement_ttc","tva_sale_service_intra_0",""
"","","","","","","","tva_super_reduite_encaissement","tva_sale_service_intra_0",""
"","","","","","","","tva_acq_normale","tva_intra_normale_biens",""
"","","","","","","","tva_acq_specifique","tva_intra_specifique_biens",""
"","","","","","","","tva_acq_encaissement","tva_intra_normale_services",""
"","","","","","","","tva_acq_intermediaire_encaissement","tva_intra_intermediaire_services",""
"","","","","","","","tva_acq_intermediaire","tva_intra_intermediaire_biens",""
"","","","","","","","tva_acq_reduite","tva_intra_reduite_biens",""
"","","","","","","","tva_acq_encaissement_reduite","tva_intra_reduite_services",""
"","","","","","","","tva_acq_super_reduite","tva_intra_super_reduite_biens",""
"","","","","","","","tva_acq_encaissement_super_reduite","tva_intra_super_reduite_services",""
"fiscal_position_template_import_export","50","Import/Export Outside Europe + DOM-TOM","1","","","","tva_normale","tva_sale_good_export_0","Import/Export Hors Europe + DOM-TOM"
"","","","","","","","tva_normale_ttc","tva_sale_good_export_0",""
"","","","","","","","tva_normale_encaissement","tva_sale_service_export_0",""
"","","","","","","","tva_intermediaire_encaissement","tva_sale_service_export_0",""
"","","","","","","","tva_normale_encaissement_ttc","tva_sale_service_export_0",""
"","","","","","","","tva_intermediaire_encaissement_ttc","tva_sale_service_export_0",""
"","","","","","","","tva_specifique","tva_sale_good_export_0",""
"","","","","","","","tva_specifique_ttc","tva_sale_good_export_0",""
"","","","","","","","tva_intermediaire","tva_sale_good_export_0",""
"","","","","","","","tva_intermediaire_ttc","tva_sale_good_export_0",""
"","","","","","","","tva_reduite","tva_sale_good_export_0",""
"","","","","","","","tva_reduite_ttc","tva_sale_good_export_0",""
"","","","","","","","tva_reduite_encaissement_ttc","tva_sale_service_export_0",""
"","","","","","","","tva_reduite_encaissement","tva_sale_service_export_0",""
"","","","","","","","tva_super_reduite","tva_sale_good_export_0",""
"","","","","","","","tva_super_reduite_ttc","tva_sale_good_export_0",""
"","","","","","","","tva_super_reduite_encaissement_ttc","tva_sale_service_export_0",""
"","","","","","","","tva_super_reduite_encaissement","tva_sale_service_export_0",""
"","","","","","","","tva_acq_normale","tva_import_outside_eu_20",""
"","","","","","","","tva_acq_specifique","tva_import_outside_eu_8_5",""
"","","","","","","","tva_acq_intermediaire","tva_import_outside_eu_10",""
"","","","","","","","tva_acq_reduite","tva_import_outside_eu_5_5",""
"","","","","","","","tva_acq_super_reduite","tva_import_outside_eu_2_1",""

```

## File: data\template\account.group-fr.csv

```csv
"id","code_prefix_start","name","name@fr"
"pcg_10","10","Capital and reserves","Capital et réserves"
"pcg_101","101","Capital","Capital"
"pcg_104","104","Premiums on share capital","Primes liées au capital social"
"pcg_106","106","Reserves","Réserves"
"pcg_11","11","Retained earnings","Report à nouveau"
"pcg_12","12","Result for the year","Résultat de l'exercice"
"pcg_13","13","Investment grants","Subventions d'investissement"
"pcg_15","15","Provisions","Provisions"
"pcg_151","151","Provisions for risks","Provisions pour risques"
"pcg_158","158","Other provisions for charges","Autres provisions pour charges"
"pcg_16","16","Loans and similar debts payable","Emprunts et dettes assimilées"
"pcg_165","165","Deposits and sureties received","Dépôts et cautionnements reçus"
"pcg_166","166","Employee profit share","Participation des salariés aux résultats"
"pcg_167","167","Loans and debts payable subject to particular conditions","Emprunts et dettes assortis de conditions particulières"
"pcg_168","168","Other loans and similar debts payable","Autres emprunts et dettes assimilées"
"pcg_17","17","Debts payable related to participating interests","Dettes rattachées à des participations"
"pcg_18","18","Reciprocal branch and joint venture accounts","Comptes de liaison des établissements et sociétés en participation"
"pcg_20","20","Intangible fixed assets","Frais d'établissement"
"pcg_201","201","Intangible fixed assets","Frais d'établissement"
"pcg_21","21","Tangible fixed assets","Immobilisations corporelles"
"pcg_211","211","Land","Terrains"
"pcg_213","213","Constructions","Constructions"
"pcg_215","215","Technical installations, plant and machinery, equipment and fixtures","Installations techniques, matériels et outillage industriels"
"pcg_2151","2151","Specialised complex installations","Installations complexes spécialisées"
"pcg_2153","2153","Installations of specific nature","Installations à caractère spécifique"
"pcg_218","218","Other tangible fixed assets","Autres immobilisations corporelles"
"pcg_23","23","Fixed assets in progress","Immobilisations en cours"
"pcg_26","26","Participating interests and related debts receivable","Participations et créances rattachées à des participations"
"pcg_261","261","Long-term equity interests","Titres de participation"
"pcg_267","267","Debts receivable related to participating interests","Créances rattachées à des participations"
"pcg_268","268","Debts receivable related to joint ventures","Créances rattachées à des sociétés en participation"
"pcg_27","27","Other financial fixed asset","Autres immobilisations financières"
"pcg_271","271","Long-term investment equity securities other than portfolio long-term investment equity securities","Titres de participation d'investissement à long terme autres que les titres de participation d'investissement à long terme du portefeuille"
"pcg_272","272","Long-term investment debt securities","Titres immobilisés (droit de créance)"
"pcg_274","274","Loans","Prêts"
"pcg_275","275","Deposits and sureties advanced","Dépôts et cautionnements versés"
"pcg_276","276","Other capitalised debts receivable","Autres créances immobilisées"
"pcg_2768","2768","Accrued interest","Intérêts courus"
"pcg_277","277","(Own shares)","(Actions propres ou parts propres)"
"pcg_28","28","Cumulative depreciation on fixed assets","Amortissements des immobilisations incorporelles"
"pcg_281","281","Depreciation on tangible fixed assets","Amortissements des immobilisations corporelles"
"pcg_29","29","Provisions for diminution in value of fixed assets","Dépréciations des immobilisations incorporelles"
"pcg_291","291","Provisions for diminution in value of tangible fixed assets (same allocation as for Account 21)","Provisions pour dépréciation des immobilisations corporelles (même affectation que pour le compte 21)"
"pcg_293","293","Provisions for diminution in value of fixed assets in progress","Dépréciations des immobilisations en cours"
"pcg_296","296","Impairment of participating interests and receivables from participating interests","Dépréciations des participations et créances rattachées à des participations"
"pcg_297","297","Provisions for diminution in value of other financial fixed assets","Dépréciations des autres immobilisations financières"
"pcg_32","32","Other consumables","Autres approvisionnements"
"pcg_322","322","Consumable supplies","Fournitures consommables"
"pcg_326","326","Packaging","Emballages"
"pcg_33","33","Work in progress (goods)","En cours de production de biens"
"pcg_34","34","Work in progress (services)","En cours de production de services"
"pcg_35","35","Product stocks","Stocks de produits"
"pcg_358","358","Residual products (or recoverable materials)","Produits résiduels (ou matières de récupération)"
"pcg_39","39","Provisions for diminution in value of stocks and work in progress","Dépréciations des stocks et en cours"
"pcg_40","40","Suppliers and related accounts","Fournisseurs et comptes rattachés"
"pcg_401","401","Suppliers","Fournisseurs"
"pcg_404","404","Fixed asset suppliers","Fournisseurs d'immobilisations"
"pcg_408","408","Suppliers - Invoices outstanding","Fournisseurs factures non parvenues"
"pcg_409","409","Suppliers in debit","Fournisseurs débiteurs"
"pcg_4097","4097","Suppliers - Other debits","Fournisseurs autres avoirs"
"pcg_411","411","Customers","Clients"
"pcg_416_group","416","Doubtful customers","Clients douteux"
"pcg_418","418","Customers - Charges not yet invoiced","Clients produits non encore facturés"
"pcg_419","419","Customers in credit","Clients créditeurs"
"pcg_42","42","Personnel and related accounts","Personnel et comptes rattachés"
"pcg_424","424","Employee profit share","Participation des salariés aux résultats"
"pcg_428","428","Personnel - Accrued charges payable and income receivable","Personnel charges à payer et produits à recevoir"
"pcg_43","43","Social security and other social agencies","Sécurité sociale et autres organismes sociaux"
"pcg_438","438","Social agencies - Accrued charges payable and income receivable","Organismes sociaux - charges à payer et produits à recevoir"
"pcg_44","44","State and other public authorities","État et autres collectivités publiques"
"pcg_442","442","State - Taxes and levies recoverable from third parties","Etat impots et taxes recouvrables sur des tiers"
"pcg_445","445","State - Turnover tax","Etat taxes sur le chiffres d'affaires"
"pcg_4455","4455","Turnover tax payable","Taxes sur le chiffre d'affaires à décaisser"
"pcg_4456","4456","Turnover tax deductible","Taxes sur le chiffre d'affaires déductibles"
"pcg_4457","4457","Turnover tax collected by the entity","Taxes sur le chiffre d'affaires collectées par l'entreprise"
"pcg_448","448","State - Accrued charges payable and income receivable","Etat charges à payer et produits à recevoir"
"pcg_45","45","Group and partners/associates","Groupe et associés"
"pcg_455","455","Partners/associates - Current accounts","Associés comptes courants"
"pcg_456","456","Partners/associates - Capital transactions","Associés opérations sur le capital"
"pcg_4561","4561","Partners/associates - Company contribution accounts","Associés comptes d'appport en société"
"pcg_4562","4562","Contributors - Capital called up, unpaid","Apporteurs capital appelé, non versé"
"pcg_458","458","Partners/associates - Joint and Economic Interest Group transactions","Associés opérations faites en commun et en GIE"
"pcg_46","46","Sundry debts receivable and payable","Débiteurs divers et créditeurs divers"
"pcg_47","47","Provisional or suspense accounts","Comptes transitoires ou d'attente"
"pcg_474_group","474","Valuation differences - Assets","Différences d’évaluation – Actif"
"pcg_475_group","475","Valuation differences - Liabilities","Différences d’évaluation – Passif"
"pcg_476","476","Realisable currency exchange losses","Différence de conversion actif"
"pcg_477","477","Realisable currency exchange gains","Différences de conversion passif"
"pcg_48","48","Accrual accounts","Comptes de régularisation"
"pcg_488","488","Periodic allocation of charges and income","Comptes de répartition périodique des charges et des produits"
"pcg_49","49","Provisions for doubtful debts","Dépréciations des comptes de tiers"
"pcg_495","495","Provisions for group and partners/associates doubtful debts","Dépréciations des comptes du groupe et des associés"
"pcg_496","496","Provisions for sundry doubtful debts","Dépréciations des comptes de débiteurs divers"
"pcg_50","50","Short-term investment securities","Valeurs mobilières de placement"
"pcg_503","503","Shares","Actions"
"pcg_506","506","Bonds","Obligations"
"pcg_508","508","Other short-term investment securities and similar debts receivable","Autres valeurs mobilières de placement et autres créances assimilées"
"pcg_51","51","Banks, financial and similar institutions","Banques, établissements financiers et assimilés"
"pcg_511","511","Financial instruments for collection","Valeurs à l'encaissement"
"pcg_512","512","Banks","Banques"
"pcg_518","518","Accrued interest","Intérêts courus"
"pcg_519","519","Current bank advances","Concours bancaires courants"
"pcg_59","59","Provisions for diminution in value of financial assets","Dépréciations des valeurs mobilières de placement"
"pcg_60","60","Purchases (except 603)","Achats (sauf 603)"
"pcg_602","602","Inventory item purchases - Other consumables","Achats stockés autres approvisionnements"
"pcg_6026","6026","Packaging","Emballages"
"pcg_603","603","Changes in inventories (supplies and goods)","Variations des stocks (approvisionnements et marchandises)"
"pcg_606","606","Non-inventory materials and supplies","Achats non stockés de matière et fournitures"
"pcg_609","609","Purchase rebates, discounts, allowances","Rabais, remises et ristournes obtenus sur achats"
"pcg_61","61","External services","Services extérieurs"
"pcg_612","612","Lease instalments","Redevances de crédit bail"
"pcg_615","615","Maintenance and repairs","Entretien et réparations"
"pcg_616","616","Insurance premiums","Primes d'assurances"
"pcg_6163","6163","Transport insurance","Assurance transport"
"pcg_618","618","Sundry","Divers"
"pcg_62","62","Other external services","Autres services extérieurs"
"pcg_621","621","Personnel external to the entity","Personnel extérieur à l'entreprise"
"pcg_622","622","Agents remuneration and fees","Rémunérations d'intermédiaires et honoraires"
"pcg_623","623","Advertising, publications, public relations","Publicité, publications, relations publiques"
"pcg_624","624","Transport of goods and collective personnel transport","Transports de biens et transports collectifs du personnel"
"pcg_625","625","Business travel, missions and receptions","Déplacements, missions et réceptions"
"pcg_627","627","Banking and similar services","Services bancaires et assimilés"
"pcg_628","628","Sundry","Divers"
"pcg_63","63","Taxes, levies and similar payments","Impôts, taxes et versements assimilés"
"pcg_631","631","Taxes, levies and similar payments on wages and salaries (to the tax administration)","Impôts, taxes et versements assimilés sur rémunérations (administrations des impôts)"
"pcg_633","633","Taxes, levies and similar payments on wages and salaries (to otherbodies)","Impôts, taxes et versements assimilés sur rémunérations (autres organismes)"
"pcg_635","635","Other taxes, levies and similar payments (to the tax administration)","Autres impôts, taxes et versements assimilés (administrations des impôts)"
"pcg_6351","6351","Direct taxes (except income tax)","Impôts directs (sauf impôts sur les bénéfices)"
"pcg_6354","6354","Registration and stamp duties","Droits d'enregistrement et de timbre"
"pcg_637","637","Other taxes, levies and similar payments (to other bodies)","Autres impôts, taxes et versements assimilés (autres organismes)"
"pcg_64","64","Personnel costs","Charges de personnel"
"pcg_641","641","Personnel wages and salaries","Rémunérations du personnel"
"pcg_645","645","Social security and provident fund contributions","Charges de sécurité sociale et de prévoyance"
"pcg_647","647","Other welfare costs","Autres charges sociales"
"pcg_65","65","Other current operating charges","Autres charges de gestion courante"
"pcg_651","651","Royalties and licence fees for concessions, patents,licences, trade marks, processes, software, rights and similar assets","Redevances pour concessions, brevets, licences, marques, procédés, logiciels, droits et valeurs simulaires"
"pcg_654","654","Bad debts written off","Pertes sur créances irrécouvrables"
"pcg_655","655","Share of joint venture profit or loss","Quote part de résultat sur opérations faites en commun"
"pcg_658_group","658","Penalties and other expenses","Pénalités et autres charges"
"pcg_66","66","Financial charges","Charges financières"
"pcg_661","661","Interest charges","Charges d'intérêts"
"pcg_6611","6611","Loan and debt interest","Intérêts des emprunts et dettes"
"pcg_6618","6618","Interest on other debts payable","Intérêts des autres dettes"
"pcg_67","67","Extraordinary charges","Charges exceptionnelles"
"pcg_68","68","Appropriations to depreciation and provisions","Dotations aux amortissements, aux dépréciations et aux provisions"
"pcg_681","681","Appropriations to depreciation and provisions - Operating charges","Dotations aux amortissements, aux dépréciations et aux provisions"
"pcg_6816","6816","Impairment of intangible and tangible assets","Dotations pour dépréciations des immobilisations incorporelles et corporelles"
"pcg_6817","6817","Appropriations to provisions for diminution in value of current assets","Dotations pour dépréciations des actifs circulants"
"pcg_686","686","Appropriations to depreciation and provisions - Financial charges","Dotations aux amortissements, aux dépréciations et aux provisions"
"pcg_6866","6866","Appropriations to provisions for diminution in value of financial components","Dotations pour dépréciations des éléments financiers"
"pcg_687","687","Appropriations to depreciation and provisions - Extraordinarcharges","Dotations aux amortissements, aux dépréciations et aux provisions"
"pcg_6872","6872","Appropriations to tax-regulated provisions (fixed assets)","Dotations aux provisions réglementées (immobilisations)"
"pcg_69","69","Employee profit share - Income and similar taxes","Participation des salariés impots sur les bénéfices et assimilés"
"pcg_695","695","Income tax","Impôts sur les bénéfices"
"pcg_698","698","Group tax","Intégration fiscale"
"pcg_70","70","Sales of manufactured products, services, goods for resale","Ventes de produits fabriqués, prestations de services, marchandises"
"pcg_708","708","Income from related activities","Produits des activités annexes"
"pcg_709","709","Sales rebates, discounts, allowances granted by the entity","Rabais, remises et ristournes accordés par l'entreprise"
"pcg_71","71","Change in stocks of finished products and work in progress","Production stockée (ou déstockage)"
"pcg_713","713","Change in stocks (work in progress, products)","Variation des stocks (en cours de production, produits)"
"pcg_7133","7133","Change in work in progress (goods)","Variation des en cours de production de biens"
"pcg_7134","7134","Change in work in progress (services)","Variation des en cours de production de services"
"pcg_7135","7135","Change in product stocks","Variation des stocks de produits"
"pcg_72","72","Own work capitalised","Production immobilisée"
"pcg_74_group","74","Operating grants","Subventions d'exploitation"
"pcg_75","75","Other current operating income","Autres produits de gestion courante"
"pcg_751","751","Royalties and licence fees for concessions, patents, licences, trade marks, processes, software, rights and similar assets","Redevances et droits de licence pour concessions, brevets, licences, marques, procédés, logiciels, droits et actifs similaires"
"pcg_755","755","Share of joint venture profit or loss","Quote part de résultat sur opérations faites en commun"
"pcg_758_group","758","Compensation and other income","Indemnités et autres produits"
"pcg_76","76","Financial income","Produits financiers"
"pcg_761","761","Income from participating interests","Produits de participations"
"pcg_762","762","Income from other financial fixed assets","Produits des autres immobilisations financières"
"pcg_763","763","Income from other debts receivable","Revenus des autres créances"
"pcg_767_group","767","Income from disposal of financial assets","Produits sur cession d’éléments financiers"
"pcg_768_group","768","Other financial income","Autres produits financiers"
"pcg_77","77","Extraordinary income","Produits exceptionnels"
"pcg_78","78","Depreciation and provisions written back","Reprises sur amortissements, dépréciations et provisions"
"pcg_781","781","Depreciation and provisions written back (to be enteredin operating income)","Reprises sur amortissements, dépréciations et provisions (à inscrire dans les produits d'exploitation)"
"pcg_786","786","Provisions for liabilities written back (to be entered in financiaincome)","Reprises sur provisions pour risques et dépréciations (à inscrire dans les produits financiers)"
"pcg_7866","7866","Provisions for diminution in value of financial components written back","Reprises sur dépréciations des éléments financiers"
"pcg_787","787","Provisions written back (to be entered in extraordinary income)","Reprises sur provisions et dépréciations (à inscrire dans les produits exceptionnels)"
"pcg_7872","7872","Reprises sur provisions réglementées (immobilisations)","Reprises sur provisions réglementées (immobilisations)"

```

## File: data\template\account.tax-fr.csv

```csv
"id","name","description","invoice_label","amount","amount_type","sequence","type_tax_use","tax_scope","include_base_amount","tax_group_id","price_include","active","tax_exigibility","cash_basis_transition_account_id","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","description@fr"
"tva_acq_normale","20% G","20% Goods","TVA 20%","20.0","percent","9","purchase","consu","1","tax_group_tva_20","","","","","100","base","invoice","","","20% M"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20",""
"tva_acq_specifique","8.5% G","8.5% Goods","TVA 8.5%","8.5","percent","10","purchase","consu","1","tax_group_tva_85","","","","","100","base","invoice","","","8,5% M"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20",""
"tva_acq_intermediaire","10% G","10% Goods","TVA 10%","10.0","percent","10","purchase","consu","1","tax_group_tva_10","","","","","100","base","invoice","","","10% M"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20",""
"tva_acq_reduite","5.5% G","5.5% Goods","TVA 5.5%","5.5","percent","10","purchase","consu","1","tax_group_tva_55","","","","","100","base","invoice","","","5,5% M"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20",""
"tva_acq_super_reduite","2.1% G","2.1% Goods","TVA 2.1%","2.1","percent","10","purchase","consu","1","tax_group_tva_21","","","","","100","base","invoice","","","2,1% M"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20",""
"tva_purchase_good_fuel","20% F","20% Fuels","TVA 20%","20.0","percent","9","purchase","consu","1","tax_group_tva_20","","","","","100","base","invoice","","","20% Carburant"
"","","","","","","","","","","","","","","","80","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","20","tax","invoice","","",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","80","tax","refund","pcg_44566","-20",""
"","","","","","","","","","","","","","","","20","tax","refund","","",""
"tva_purchase_good_fuel_TTC","20% F INC","20% fuel tax incl.","TVA 20%","20.0","percent","9","purchase","consu","1","tax_group_tva_20","True","","","","100","base","invoice","","","20% Carburant TTC"
"","","","","","","","","","","","","","","","80","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","20","tax","invoice","","",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","80","tax","refund","pcg_44566","-20",""
"","","","","","","","","","","","","","","","20","tax","refund","","",""
"tva_acq_normale_TTC","20% G INC","20% Goods tax incl.","TVA 20%","20.0","percent","10","purchase","consu","","tax_group_tva_20","True","","","","100","base","invoice","","","20% M TTC"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20",""
"tva_acq_specifique_TTC","8.5% G INC","8.5% Goods tax incl.","TVA 8.5%","8.5","percent","10","purchase","consu","","tax_group_tva_85","True","","","","100","base","invoice","","","8,5% M TTC"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20",""
"tva_acq_intermediaire_TTC","10% G INC","10% Goods tax incl.","TVA 10%","10.0","percent","10","purchase","consu","","tax_group_tva_10","True","","","","100","base","invoice","","","10% M TTC"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20",""
"tva_acq_reduite_TTC","5.5% G INC","5.5% Goods tax incl.","TVA 5.5%","5.5","percent","10","purchase","consu","","tax_group_tva_55","True","","","","100","base","invoice","","","5,5% M TTC"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20",""
"tva_acq_super_reduite_TTC","2.1% G INC","2.1% Goods tax incl.","TVA 2.1%","2.1","percent","10","purchase","consu","","tax_group_tva_21","True","","","","100","base","invoice","","","2,1% M TTC"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20",""
"tva_imm_normale","20% R E ","20% real estate","TVA 20%","20.0","percent","10","purchase","consu","1","tax_group_tva_20","","","","","100","base","invoice","","","20% Immo"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44562","+19",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44562","-19",""
"tva_imm_specifique","8.5% R E","8.5% real estate","TVA 8.5%","8.5","percent","10","purchase","consu","1","tax_group_tva_85","","0","","","100","base","invoice","","","8,5% Immo"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44562","+19",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44562","-19",""
"tva_imm_intermediaire","10% R E","10% real estate","TVA 10%","10.0","percent","10","purchase","consu","1","tax_group_tva_10","","0","","","100","base","invoice","","","10% immo"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44562","+19",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44562","-19",""
"tva_imm_reduite","5.5% R E","5.5% real estate","TVA 5.5%","5.5","percent","10","purchase","consu","1","tax_group_tva_55","","0","","","100","base","invoice","","","5,5% Immo"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44562","+19",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44562","-19",""
"tva_imm_super_reduite","2.1% R E","2.1% real estate","TVA 2.1%","2.1","percent","10","purchase","consu","1","tax_group_tva_21","","0","","","100","base","invoice","","","2,1% Immo"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44562","+19",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44562","-19",""
"tva_import_outside_eu_20","20% EX O EU","20% import","TVA 20%","20.0","percent","11","purchase","consu","1","tax_group_tva_20","","","","","100","base","invoice","","+A4||+I1_base","20% Import"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_445663","+20||+24",""
"","","","","","","","","","","","","","","","-100","tax","invoice","pcg_4453","-I1_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A4||-I1_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_445663","-20||-24",""
"","","","","","","","","","","","","","","","-100","tax","refund","pcg_4453","+I1_taxe",""
"tva_import_outside_eu_10","10% EX","10% import","TVA 10%","10.0","percent","11","purchase","consu","1","tax_group_tva_10","","0","","","100","base","invoice","","+A4||+I2_base","10% Import"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_445663","+20||+24",""
"","","","","","","","","","","","","","","","-100","tax","invoice","pcg_4453","-I2_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A4||-I2_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_445663","-20||-24",""
"","","","","","","","","","","","","","","","-100","tax","refund","pcg_4453","+I2_taxe",""
"tva_import_outside_eu_8_5","8.5% EX","8.5% import","TVA 8.5%","8.5","percent","11","purchase","consu","1","tax_group_tva_85","","0","","","100","base","invoice","","+A4||+I3_base","8,5% Import"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_445663","+20||+24",""
"","","","","","","","","","","","","","","","-100","tax","invoice","pcg_4453","-I3_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A4||-I3_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_445663","-20||-24",""
"","","","","","","","","","","","","","","","-100","tax","refund","pcg_4453","+I3_taxe",""
"tva_import_outside_eu_5_5","5.5% EX","5.5% import","TVA 5.5%","5.5","percent","11","purchase","consu","1","tax_group_tva_55","","0","","","100","base","invoice","","+A4||+I4_base","5,5% Import"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_445663","+20||+24",""
"","","","","","","","","","","","","","","","-100","tax","invoice","pcg_4453","-I4_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A4||-I4_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_445663","-20||-24",""
"","","","","","","","","","","","","","","","-100","tax","refund","pcg_4453","+I4_taxe",""
"tva_import_outside_eu_2_1","2.1% EX","2.1% import","TVA 2.1%","2.1","percent","11","purchase","consu","1","tax_group_tva_21","","0","","","100","base","invoice","","+A4||+I5_base","2,1% Import"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_445663","+20||+24",""
"","","","","","","","","","","","","","","","-100","tax","invoice","pcg_4453","-I5_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A4||-I5_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_445663","-20||-24",""
"","","","","","","","","","","","","","","","-100","tax","refund","pcg_4453","+I5_taxe",""
"tva_intra_normale_biens","20% EU G","20% EU G","TVA 20%","20.0","percent","10","purchase","consu","1","tax_group_tva_20","","","","","100","base","invoice","","+B2||+08_base","20% EU M"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+20",""
"","","","","","","","","","","","","","","","-100","tax","invoice","pcg_4452","-08_taxe||-17",""
"","","","","","","","","","","","","","","","100","base","refund","","-B2||-08_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-20",""
"","","","","","","","","","","","","","","","-100","tax","refund","pcg_4452","+08_taxe||+17",""
"tva_intra_normale_services","20% EU S","20% EU S","TVA 20%","20.0","percent","10","purchase","service","1","tax_group_tva_20","","","","","100","base","invoice","","+A3||+08_base","20% EU S"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+20",""
"","","","","","","","","","","","","","","","-100","tax","invoice","pcg_44521","-08_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A3||-08_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-20",""
"","","","","","","","","","","","","","","","-100","tax","refund","pcg_44521","+08_taxe",""
"tva_purchase_service_20_import","20% EX","20% IMPORT","TVA 20%","20.0","percent","10","purchase","service","1","tax_group_tva_20","","","","","100","base","invoice","","+B4||+08_base","20% Import"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_445663","+20",""
"","","","","","","","","","","","","","","","-100","tax","invoice","pcg_44531","-08_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-B4||-08_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_445663","-20",""
"","","","","","","","","","","","","","","","-100","tax","refund","pcg_44531","+08_taxe",""
"tva_purchase_service_0","0% EX","0% EXO","TVA 0%","0.0","percent","10","purchase","service","1","tax_group_tva_0","","","on_invoice","pcg_44574","100","base","invoice","","","0% EXO"
"","","","","","","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","","",""
"tva_acq_encaissement","20% S","20% Service","TVA 20%","20.0","percent","10","purchase","service","1","tax_group_tva_20","","","on_payment","pcg_44564","100","base","invoice","","","20% Service"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20",""
"tva_acq_intermediaire_encaissement","10% S","10% Service","TVA 10%","10.0","percent","10","purchase","service","1","tax_group_tva_10","","0","on_payment","pcg_44564","100","base","invoice","","","10% Service"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20",""
"tva_acq_encaissement_reduite","5.5% S","5.5% Service","TVA 5.5%","5.5","percent","10","purchase","service","1","tax_group_tva_55","","0","on_payment","pcg_44564","100","base","invoice","","","5,5% Service"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20",""
"tva_acq_encaissement_super_reduite","2.1% S","2.1% Service","TVA 2.1%","2.1","percent","10","purchase","service","1","tax_group_tva_21","","0","on_payment","pcg_44564","100","base","invoice","","","2,1% Service"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20",""
"tva_acq_encaissement_TTC","20% S INC","20% Service tax incl.","VAT 20%","20.0","percent","10","purchase","service","","tax_group_tva_20","True","","on_payment","pcg_44564","100","base","invoice","","","20% Service TTC"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20",""
"tva_acq_intermediaire_encaissement_TTC","10% S INC","10% Service tax incl.","TVA 10%","10.0","percent","10","purchase","service","","tax_group_tva_10","True","0","on_payment","pcg_44564","100","base","invoice","","","10% Service TTC"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20",""
"tva_acq_encaissement_reduite_TTC","5.5% S INC","5.5% Service tax incl.","TVA 5.5%","5.5","percent","10","purchase","service","","tax_group_tva_55","True","0","on_payment","pcg_44564","100","base","invoice","","","5,5 Service TTC"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20",""
"tva_acq_encaissement_super_reduite_TTC","2.1% S INC","2.1% Service tax incl.","TVA 2.1%","2.1","percent","10","purchase","service","","tax_group_tva_21","True","0","on_payment","pcg_44564","100","base","invoice","","","2,1% Service TTC"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20",""
"tva_purchase_imm_normale","20% R E","20% real estate","TVA 20%","20.0","percent","10","purchase","service","1","tax_group_tva_20","","","on_payment","pcg_44564","100","base","invoice","","","20% Immo"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44562","+19",""
"","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44562","-19",""
"tva_normale","20% G","20% Goods","TVA 20%","20.0","percent","10","sale","consu","1","tax_group_tva_20","","","","","100","base","invoice","","+A1||+08_base","20% M"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+08_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A1||-08_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-08_taxe",""
"tva_intermediaire","10% G","10% Goods","TVA 10%","10.0","percent","10","sale","consu","1","tax_group_tva_10","","0","","","100","base","invoice","","+A1||+9B_base","10% M"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+9B_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A1||-9B_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-9B_taxe",""
"tva_reduite","5.5% G","5.5% Goods","TVA 5.5%","5.5","percent","10","sale","consu","1","tax_group_tva_55","","0","","","100","base","invoice","","+A1||+09_base","5,5% M"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+09_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A1||-09_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-09_taxe",""
"tva_specifique","8.5% G","8.5% Goods","TVA 8.5%","8.5","percent","10","sale","consu","1","tax_group_tva_85","","0","","","100","base","invoice","","+A1||+10_base","8,5% M"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+10_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A1||-10_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-10_taxe",""
"tva_super_reduite","2.1% G","2.1% Goods","TVA 2.1%","2.1","percent","10","sale","consu","1","tax_group_tva_21","","0","","","100","base","invoice","","+A1||+11_base","2.1% M"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+11_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A1||-11_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-11_taxe",""
"tva_normale_ttc","20% G INC","20% Goods tax incl.","TVA 20%","20.0","percent","10","sale","consu","","tax_group_tva_20","True","0","","","100","base","invoice","","+A1||+08_base","20% M TTC"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+08_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A1||-08_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-08_taxe",""
"tva_intermediaire_ttc","10% G INC","10% Goods tax incl.","TVA 10%","10.0","percent","10","sale","consu","","tax_group_tva_10","True","0","","","100","base","invoice","","+A1||+9B_base","10% M TTC"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+9B_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A1||-9B_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-9B_taxe",""
"tva_specifique_ttc","8.5% G INC","8.5% Goods tax incl.","TVA 8.5%","8.5","percent","10","sale","consu","","tax_group_tva_85","True","0","","","100","base","invoice","","+A1||+10_base","8,5% M TTC"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+10_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A1||-10_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-10_taxe",""
"tva_reduite_ttc","5.5% G INC","5.5% Goods tax incl.","TVA 5.5%","5.5","percent","10","sale","consu","","tax_group_tva_55","True","0","","","100","base","invoice","","+A1||+09_base","5,5% M TTC"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+09_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A1||-09_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-09_taxe",""
"tva_super_reduite_ttc","2.1% G INC","2.1% Goods tax incl.","TVA 2.1%","2.1","percent","10","sale","consu","","tax_group_tva_21","True","0","","","100","base","invoice","","+A1||+11_base","2,1% M TTC"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+11_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A1||-11_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-11_taxe",""
"tva_sale_good_0","0% EXEMPT G","0% EXO","TVA 0%","0.0","percent","10","sale","consu","1","tax_group_tva_0","","","","","100","base","invoice","","+E2","0% EXO"
"","","","","","","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","","","","","","100","base","refund","","-E2",""
"","","","","","","","","","","","","","","","100","tax","refund","","",""
"tva_sale_good_export_0","0% EX G","0% EXPORT","TVA 0%","0.0","percent","10","sale","consu","1","tax_group_tva_0","","","","","100","base","invoice","","+E1","0% EXPORT"
"","","","","","","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","","","","","","100","base","refund","","-E1",""
"","","","","","","","","","","","","","","","100","tax","refund","","",""
"tva_sale_good_intra_0","0% EU G","0% EU G","TVA 0%","0.0","percent","10","sale","consu","1","tax_group_tva_0","","","","","100","base","invoice","","+F2","0% EU M"
"","","","","","","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","","","","","","100","base","refund","","-F2",""
"","","","","","","","","","","","","","","","100","tax","refund","","",""
"tva_normale_encaissement","20% S","20% Service","TVA 20%","20.0","percent","10","sale","service","1","tax_group_tva_20","","","on_payment","pcg_44574","100","base","invoice","","+A1||+08_base","20% Service"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+08_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A1||-08_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-08_taxe",""
"tva_intermediaire_encaissement","10% S","10% Service","TVA 10%","10.0","percent","10","sale","service","1","tax_group_tva_10","","0","on_payment","pcg_44574","100","base","invoice","","+A1||+9B_base","10% Service"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+9B_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A1||-9B_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-9B_taxe",""
"tva_reduite_encaissement","5.5% S","5.5% Service","TVA 5.5%","5.5","percent","10","sale","service","1","tax_group_tva_55","","0","on_payment","pcg_44574","100","base","invoice","","+A1||+09_base","5,5% Service"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+09_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A1||-09_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-09_taxe",""
"tva_super_reduite_encaissement","2.1% S","2.1% Service","TVA 2.1%","2.1","percent","10","sale","service","1","tax_group_tva_21","","0","on_payment","pcg_44574","100","base","invoice","","+A1||+11_base","2,1% Service"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+11_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A1||-11_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-11_taxe",""
"tva_normale_encaissement_ttc","20% S INC","20% Service tax incl.","TVA 20%","20.0","percent","10","sale","service","","tax_group_tva_20","True","0","on_payment","pcg_445800","100","base","invoice","","+A1||+08_base","20% Service TTC"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+08_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A1||-08_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-08_taxe",""
"tva_intermediaire_encaissement_ttc","10% S INC","10% Service tax incl.","VAT 10%","10.0","percent","10","sale","service","","tax_group_tva_10","True","0","on_payment","pcg_445800","100","base","invoice","","+A1||+9B_base","10% Service TTC"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+9B_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A1||-9B_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-9B_taxe",""
"tva_reduite_encaissement_ttc","5.5% S INC","5.5% Service tax incl.","TVA 5.5%","5.5","percent","10","sale","service","","tax_group_tva_55","True","0","on_payment","pcg_445800","100","base","invoice","","+A1||+09_base","5,5% Service TTC"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+09_taxe",""
"","","","","","","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","","","","","","100","base","refund","","-A1||-09_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-09_taxe",""
"","","","","","","","","","","","","","","","100","tax","refund","","",""
"tva_super_reduite_encaissement_ttc","2.1% S INC","2.1% Service tax incl.","TVA 2.1%","2.1","percent","10","sale","service","","tax_group_tva_21","True","0","on_payment","pcg_445800","100","base","invoice","","+A1||+11_base","2,1% Service TTC"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+11_taxe",""
"","","","","","","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","","","","","","100","base","refund","","-A1||-11_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-11_taxe",""
"","","","","","","","","","","","","","","","100","tax","refund","","",""
"tva_sale_service_0","0% EXEMPT S","0% EXEMPTS Service","TVA 0%","0.0","percent","10","sale","service","1","tax_group_tva_0","","","on_payment","pcg_44574","100","base","invoice","","+E2","0% EXO"
"","","","","","","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","","","","","","100","base","refund","","-E2",""
"","","","","","","","","","","","","","","","100","tax","refund","","",""
"tva_sale_service_export_0","0% EX S","0% EXPORT Service","TVA 0%","0.0","percent","10","sale","service","1","tax_group_tva_0","","","","pcg_44574","100","base","invoice","","+E2","0% EXPORT"
"","","","","","","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","","","","","","100","base","refund","","-E2",""
"","","","","","","","","","","","","","","","100","tax","refund","","",""
"tva_sale_service_intra_0","0% EU S","0% EU Service","TVA 0%","0.0","percent","10","sale","service","1","tax_group_tva_0","","","on_payment","pcg_44574","100","base","invoice","","+E2","0% EU Service"
"","","","","","","","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","","","","","","","100","base","refund","","-E2",""
"","","","","","","","","","","","","","","","100","tax","refund","","",""
"tva_intra_specifique_biens","8.5% EU G","8.5% EU Goods","TVA 8.5%","8.5","percent","10","purchase","consu","1","tax_group_tva_85","","0","","","100","base","invoice","","+B2||+10_base","TVA 8,5% EU M"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+20",""
"","","","","","","","","","","","","","","","-100","tax","invoice","pcg_4452","-10_taxe||-17",""
"","","","","","","","","","","","","","","","100","base","refund","","-B2||-10_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-20",""
"","","","","","","","","","","","","","","","-100","tax","refund","pcg_4452","+10_taxe||+17",""
"tva_intra_specifique_services","8.5% EU S","8.5% EU Service","TVA 8.5%","8.5","percent","10","purchase","service","1","tax_group_tva_85","","0","","","100","base","invoice","","+A3||+10_base","TVA 8,5 EU Service"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+17||+10_taxe",""
"","","","","","","","","","","","","","","","-100","tax","invoice","pcg_44521","-20",""
"","","","","","","","","","","","","","","","100","base","refund","","-A3||-10_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-17||-10_taxe",""
"","","","","","","","","","","","","","","","-100","tax","refund","pcg_44521","+20",""
"tva_intra_intermediaire_biens","10% EU G","10% EU Goods","TVA 10%","10.0","percent","10","purchase","consu","1","tax_group_tva_10","","0","","","100","base","invoice","","+B2||+9B_base","10% EU M"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+20",""
"","","","","","","","","","","","","","","","-100","tax","invoice","pcg_4452","-9B_taxe||-17",""
"","","","","","","","","","","","","","","","100","base","refund","","-B2||-9B_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-20",""
"","","","","","","","","","","","","","","","-100","tax","refund","pcg_4452","+9B_taxe||+17",""
"tva_intra_intermediaire_services","10% EU S","10% EU Service","TVA 10%","10.0","percent","10","purchase","service","1","tax_group_tva_10","","0","","","100","base","invoice","","+A3||+9B_base","10% EU M"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+20",""
"","","","","","","","","","","","","","","","-100","tax","invoice","pcg_44521","-9B_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A3||-9B_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-20",""
"","","","","","","","","","","","","","","","-100","tax","refund","pcg_44521","+9B_taxe",""
"tva_intra_reduite_biens","5.5% EU G","5.5% EU Goods","TVA 5.5%","5.5","percent","10","purchase","consu","1","tax_group_tva_55","","0","","","100","base","invoice","","+B2||+09_base","5,5% EU M"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+20",""
"","","","","","","","","","","","","","","","-100","tax","invoice","pcg_4452","-09_taxe||-17",""
"","","","","","","","","","","","","","","","100","base","refund","","-B2||-09_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-20",""
"","","","","","","","","","","","","","","","-100","tax","refund","pcg_4452","+09_taxe||+17",""
"tva_intra_reduite_services","5.5% EU S","5.5% EU Service","TVA 5.5%","5.5","percent","10","purchase","service","1","tax_group_tva_55","","0","","","100","base","invoice","","+A3||+09_base","5,5% EU M"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+20",""
"","","","","","","","","","","","","","","","-100","tax","invoice","pcg_44521","-09_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A3||-09_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-20",""
"","","","","","","","","","","","","","","","-100","tax","refund","pcg_44521","+09_taxe",""
"tva_intra_super_reduite_biens","2.1% EU G","2.1% EU Goods","TVA 2.1%","2.1","percent","10","purchase","consu","1","tax_group_tva_21","","0","","","100","base","invoice","","+B2||+11_base","2,1% EU M"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+20",""
"","","","","","","","","","","","","","","","-100","tax","invoice","pcg_4452","-11_taxe||-17",""
"","","","","","","","","","","","","","","","100","base","refund","","-B2||-11_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-20",""
"","","","","","","","","","","","","","","","-100","tax","refund","pcg_4452","+11_taxe||+17",""
"tva_intra_super_reduite_services","2.1% EU S","2.1% EU Service","TVA 2.1%","2.1","percent","10","purchase","service","1","tax_group_tva_21","","0","","","100","base","invoice","","+A3||+11_base","2,1% EU M"
"","","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+20",""
"","","","","","","","","","","","","","","","-100","tax","invoice","pcg_44521","-11_taxe",""
"","","","","","","","","","","","","","","","100","base","refund","","-A3||-11_base",""
"","","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-20",""
"","","","","","","","","","","","","","","","-100","tax","refund","pcg_44521","+11_taxe",""

```

## File: data\template\account.tax.group-fr.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id","name@fr"
"tax_group_tva_0","VAT 0%","base.fr","pcg_44567","pcg_44551","TVA 0%"
"tax_group_tva_20","VAT 20%","base.fr","pcg_44567","pcg_44551","TVA 20%"
"tax_group_tva_85","VAT 8.5%","base.fr","pcg_44567","pcg_44551","TVA 8,5%"
"tax_group_tva_55","VAT 5.5%","base.fr","pcg_44567","pcg_44551","TVA 5,5%"
"tax_group_tva_10","VAT 10%","base.fr","pcg_44567","pcg_44551","TVA 10%"
"tax_group_tva_21","VAT 2.1%","base.fr","pcg_44567","pcg_44551","TVA 2,1%"

```

## File: migrations\2.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'fr')], order="parent_path"):
        env['account.chart.template'].try_loading('fr', company)

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_fr_closing_sequence_id = fields.Many2one('ir.sequence', 'Sequence to use to build sale closings', readonly=True)
    siret = fields.Char(related='partner_id.siret', string='SIRET', size=14, readonly=False)
    ape = fields.Char(string='APE')
    is_france_country = fields.Boolean(
        compute="_compute_is_france_country",
        string="Is Part of DOM-TOM",
    )

    @api.depends('country_code')
    def _compute_is_france_country(self):
        for company in self:
            company.is_france_country = company.country_code in self._get_france_country_codes()

    l10n_fr_rounding_difference_loss_account_id = fields.Many2one('account.account', check_company=True)
    l10n_fr_rounding_difference_profit_account_id = fields.Many2one('account.account', check_company=True)

    @api.model
    def _get_france_country_codes(self):
        """Returns every country code that can be used to represent France
        """
        return ['FR', 'MF', 'MQ', 'NC', 'PF', 'RE', 'GF', 'GP', 'TF', 'BL', 'PM', 'YT', 'WF']  # These codes correspond to France and DOM-TOM.

    @api.model
    def _get_unalterable_country(self):
        return self._get_france_country_codes()

    def _is_accounting_unalterable(self):
        if not self.vat and not self.country_id:
            return False
        return self.country_id and self.country_id.code in self._get_unalterable_country()

    @api.model_create_multi
    def create(self, vals_list):
        companies = super().create(vals_list)
        for company in companies:
            #when creating a new french company, create the securisation sequence as well
            if company._is_accounting_unalterable():
                sequence_fields = ['l10n_fr_closing_sequence_id']
                company._create_secure_sequence(sequence_fields)
        return companies

    def write(self, vals):
        res = super(ResCompany, self).write(vals)
        #if country changed to fr, create the securisation sequence
        for company in self:
            if company._is_accounting_unalterable():
                sequence_fields = ['l10n_fr_closing_sequence_id']
                company._create_secure_sequence(sequence_fields)
        return res

    def _create_secure_sequence(self, sequence_fields):
        """This function creates a no_gap sequence on each company in self that will ensure
        a unique number is given to all posted account.move in such a way that we can always
        find the previous move of a journal entry on a specific journal.
        """
        for company in self:
            vals_write = {}
            for seq_field in sequence_fields:
                if not company[seq_field]:
                    vals = {
                        'name': _('Securisation of %s - %s', seq_field, company.name),
                        'code': 'FRSECURE%s-%s' % (company.id, seq_field),
                        'implementation': 'no_gap',
                        'prefix': '',
                        'suffix': '',
                        'padding': 0,
                        'company_id': company.id}
                    seq = self.env['ir.sequence'].create(vals)
                    vals_write[seq_field] = seq.id
            if vals_write:
                company.write(vals_write)

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models, api, _


class ResPartner(models.Model):
    _inherit = 'res.partner'

    siret = fields.Char(string='SIRET', size=14)

    def _deduce_country_code(self):
        if self.siret:
            return 'FR'
        return super()._deduce_country_code()

    def _peppol_eas_endpoint_depends(self):
        # extends account_edi_ubl_cii
        return super()._peppol_eas_endpoint_depends() + ['siret']

```

## File: models\template_fr.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, Command
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('fr')
    def _get_fr_template_data(self):
        return {
            'code_digits': 6,
            'property_account_receivable_id': 'fr_pcg_recv',
            'property_account_payable_id': 'fr_pcg_pay',
            'property_account_expense_categ_id': 'pcg_607_account',
            'property_account_income_categ_id': 'pcg_707_account',
        }

    @template('fr', 'res.company')
    def _get_fr_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.fr',
                'bank_account_code_prefix': '512',
                'cash_account_code_prefix': '53',
                'transfer_account_code_prefix': '58',
                'account_default_pos_receivable_account_id': 'fr_pcg_recv_pos',
                'income_currency_exchange_account_id': 'pcg_766',
                'expense_currency_exchange_account_id': 'pcg_666',
                'account_journal_suspense_account_id': 'pcg_471',
                'account_journal_payment_debit_account_id': 'pcg_472',
                'account_journal_payment_credit_account_id': 'pcg_473',
                'account_journal_early_pay_discount_loss_account_id': 'pcg_665',
                'account_journal_early_pay_discount_gain_account_id': 'pcg_765',
                'deferred_expense_account_id': 'pcg_486',
                'deferred_revenue_account_id': 'pcg_487',
                'l10n_fr_rounding_difference_loss_account_id': 'pcg_4768',
                'l10n_fr_rounding_difference_profit_account_id': 'pcg_4778',
                'account_sale_tax_id': 'tva_normale',
                'account_purchase_tax_id': 'tva_acq_normale',
            },
        }

    @template('fr', 'account.journal')
    def _get_fr_account_journal(self):
        return {
            'sale': {'refund_sequence': True},
            'purchase': {'refund_sequence': True},
        }

    @template('fr', 'account.reconcile.model')
    def _get_fr_reconcile_model(self):
        return {
            'bank_charges_reconcile_model': {
                'name': 'Bank fees',
                'line_ids': [
                    Command.create({
                        'account_id': 'pcg_6278',
                        'amount_type': 'percentage',
                        'amount_string': '100',
                    }),
                ],
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_fr
from . import res_partner
from . import res_company

```

## File: views\l10n_fr_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="res_company_form_l10n_fr" model="ir.ui.view">
            <field name="name">res.company.form.l10n.fr</field>
            <field name="model">res.company</field>
            <field name="priority">20</field>
            <field name="inherit_id" ref="account.view_company_form"/>
            <field name="arch" type="xml">
            <data>
                 <xpath expr="//field[@name='company_registry']" position="after">
                     <field name="is_france_country" invisible="1"/>
                     <field name="siret" invisible="not is_france_country"/>
                     <field name="ape" invisible="not is_france_country"/>
                 </xpath>
            </data>
            </field>
        </record>

        <record id="res_partner_form_l10n_fr" model="ir.ui.view">
            <field name="name">res.partner.form.l10n.fr</field>
            <field name="model">res.partner</field>
            <field name="priority">20</field>
            <field name="inherit_id" ref="account.view_partner_property_form"/>
            <field name="arch" type="xml">
            <data>
                 <xpath expr="//field[@name='ref']" position="after">
                    <field name="siret" invisible="'FR' not in fiscal_country_codes or not is_company"/>
                 </xpath>
            </data>
            </field>
        </record>
</odoo>

```

## File: views\report_l10nfrbilan.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
<template id="report_l10nfrbilan">
    <t t-call="web.html_container">
        <t t-call="web.internal_layout">
            <div class="page">
                <h2>Bilan</h2>
                <div class="row mt32 mb32">
                    <div class="col-3">
                        <span t-out="res_company.name"/>
                        <br/>au
                        <span t-out="time.strftime('%d-%m-%Y', time.strptime(date_stop,'%Y-%m-%d'))"/>
                    </div>
                    <div class="col-3">
                        <p>Imprimé le
                            <span t-out="time.strftime('%d-%m-%Y')"/>
                            <br/>Tenue de compte:
                            <span t-out="res_company.currency_id.name"/>
                        </p>
                    </div>
                </div>
                <h3>Actif</h3>
                <table class="table table-sm">
                    <thead>
                        <tr>
                            <th></th>
                            <th>Brut</th>
                            <th>Amortissements et d&#xE9;pr&#xE9;ciations</th>
                            <th>Net</th>
                        </tr>
                    </thead>
                    <tr>
                        <td>Capital souscrit - non appel&#xE9;</td>
                        <td class="text-end">
                            <span t-out="bavar1" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-end">
                            <span t-out="bavar1" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>
                            <strong>ACTIF IMMOBILIS&#xC9;</strong>
                        </td>
                        <td></td>
                        <td></td>
                        <td></td>
                    </tr>
                    <tr>
                        <td>
                            <strong>IMMOBILISATIONS INCORPORELLES</strong>
                        </td>
                        <td></td>
                        <td></td>
                        <td></td>
                    </tr>
                    <tr>
                        <td>Frais d'&#xE9;tablissement</td>
                        <td class="text-end">
                            <span t-out="bavar2" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar2b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar2+bavar2b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Frais de recherche et de d&#xE9;veloppement</td>
                        <td class="text-end">
                            <span t-out="bavar3" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar3b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar3+bavar3b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Concessions, brevets, licences,..., droits et valeurs similaires</td>
                        <td class="text-end">
                            <span t-out="bavar4" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar4b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar4+bavar4b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Fonds commercial</td>
                        <td class="text-end">
                            <span t-out="bavar5" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar5b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar5+bavar5b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Autres</td>
                        <td class="text-end">
                            <span t-out="bavar6" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar6b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar6+bavar6b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Immobilisations incorporelles en cours</td>
                        <td class="text-end">
                            <span t-out="bavar7" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar7b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar7+bavar7b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Avances et acomptes</td>
                        <td class="text-end">
                            <span t-out="bavar8" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-end">
                            <span t-out="bavar8" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>
                            <strong>IMMOBILISATIONS CORPORELLES</strong>
                        </td>
                        <td></td>
                        <td></td>
                        <td></td>
                    </tr>
                    <tr>
                        <td>Terrains</td>
                        <td class="text-end">
                            <span t-out="bavar9" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar9b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar9+bavar9b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Constructions</td>
                        <td class="text-end">
                            <span t-out="bavar10" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar10b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar10+bavar10b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Installations techniques,mat&#xE9;riel et outillage</td>
                        <td class="text-end">
                            <span t-out="bavar11" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar11b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar11+bavar11b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Autres</td>
                        <td class="text-end">
                            <span t-out="bavar12" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar12b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar12+bavar12b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Immobilisations corporelles en cours</td>
                        <td class="text-end">
                            <span t-out="bavar13" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar13b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar13+bavar13b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Avances et acomptes</td>
                        <td class="text-end">
                            <span t-out="bavar14" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-end">
                            <span t-out="bavar14" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>
                            <strong>IMMOBILISATIONS FINANCI&#xC9;RES</strong>
                        </td>
                        <td></td>
                        <td></td>
                        <td></td>
                    </tr>
                    <tr>
                        <td>Participations</td>
                        <td class="text-end">
                            <span t-out="bavar15" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar15b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar15+bavar15b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Cr&#xE9;ances rattach&#xE9;es &#xE0; des participations</td>
                        <td class="text-end">
                            <span t-out="bavar16" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar16b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar16+bavar16b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Titres immobilis&#xE9;s de l'activit&#xE9; de portefeuille</td>
                        <td class="text-end">
                            <span t-out="bavar17" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar17b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar17+bavar17b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Autres titres immobilis&#xE9;s</td>
                        <td class="text-end">
                            <span t-out="bavar18" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar18b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar18+bavar18b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Pr&#xEA;ts</td>
                        <td class="text-end">
                            <span t-out="bavar19" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar19b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar19+bavar19b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Autres</td>
                        <td class="text-end">
                            <span t-out="bavar20" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar20b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar20+bavar20b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td class="text-end">
                            <strong>TOTAL I</strong>
                        </td>
                        <td class="text-end">
                            <span t-out="at1a" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-at1b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="at1" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>ACTIF CIRCULANT</td>
                        <td></td>
                        <td></td>
                        <td></td>
                    </tr>
                    <tr>
                        <td>STOCK EN COURS</td>
                        <td></td>
                        <td></td>
                        <td></td>
                    </tr>
                    <tr>
                        <td>Mati&#xE8;res premi&#xE8;res et autres approvisionnements</td>
                        <td class="text-end">
                            <span t-out="bavar21" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar21b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar21+bavar21b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>En-cours de production [biens et services]</td>
                        <td class="text-end">
                            <span t-out="bavar22" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar22b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar22+bavar22b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Produits interm&#xE9;diaires et finis</td>
                        <td class="text-end">
                            <span t-out="bavar23" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar23b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar23+bavar23b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Marchandises</td>
                        <td class="text-end">
                            <span t-out="bavar24" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar24b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar24+bavar24b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Avances et acomptes vers&#xE9;s sur commandes</td>
                        <td class="text-end">
                            <span t-out="bavar25" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-end">
                            <span t-out="bavar25" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>CR&#xC9;ANCES</td>
                        <td></td>
                        <td></td>
                        <td></td>
                    </tr>
                    <tr>
                        <td>Cr&#xE9;ances clients et comptes rattach&#xE9;s</td>
                        <td class="text-end">
                            <span t-out="bavar26" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar26b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar26+bavar26b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Autres</td>
                        <td class="text-end">
                            <span t-out="bavar27" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar27b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar27+bavar27b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Capital souscrit - appel&#xE9; , non vers&#xE9;</td>
                        <td class="text-end">
                            <span t-out="bavar28" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-end">
                            <span t-out="bavar28" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>VALEURS MOBILI&#xC8;RES DE PLACEMENT</td>
                        <td></td>
                        <td></td>
                        <td></td>
                    </tr>
                    <tr>
                        <td>Actions propres</td>
                        <td class="text-end">
                            <span t-out="bavar29" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar29b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar29+bavar29b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Autres titres</td>
                        <td class="text-end">
                            <span t-out="bavar30" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-bavar30b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="bavar30+bavar30b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Instruments de tr&#xE9;sorerie</td>
                        <td class="text-end">
                            <span t-out="bavar31" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-end">
                            <span t-out="bavar31" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Disponibilit&#xE9;s</td>
                        <td class="text-end">
                            <span t-out="bavar32" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-end">
                            <span t-out="bavar32" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Charges constat&#xE9;s d'avance</td>
                        <td class="text-end">
                            <span t-out="bavar33" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-end">
                            <span t-out="bavar33" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td class="text-end">
                            <strong>TOTAL II</strong>
                        </td>
                        <td class="text-end">
                            <span t-out="at2a" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-at2b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="at2" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Charges &#xE0; r&#xE9;partir sur plusieurs exercices ( III )</td>
                        <td class="text-end">
                            <span t-out="bavar34" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-end">
                            <span t-out="bavar34" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Primes de remboursement des emprunts ( IV )</td>
                        <td class="text-end">
                            <span t-out="bavar35" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-end">
                            <span t-out="bavar35" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>&#xC9;carts de conversion actif ( V )</td>
                        <td class="text-end">
                            <span t-out="bavar36" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-end">
                            <span t-out="bavar36" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td class="text-end">
                            <strong>TOTAL ACTIF ( I + II + III + IV + V )</strong>
                        </td>
                        <td class="text-end">
                            <span t-out="at1a+at2a" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="-at1b-at2b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-end">
                            <span t-out="actif" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                </table>

                <h3>Passif</h3>
                <table class="table table-sm">
                    <tbody>
                        <tr>
                            <td><strong>CAPITAUX PROPRES</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>Capital [dont vers&#xE9;...]</td>
                            <td><span t-out="bpvar1" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Primes d'&#xE9;mission, de fusion, d'apport</td>
                            <td><span t-out="bpvar2" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>&#xC9;carts de r&#xE9;&#xE9;valuation</td>
                            <td><span t-out="bpvar3" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>&#xC9;cart d'&#xE9;quivalence</td>
                            <td><span t-out="bpvar4" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td><strong>R&#xC9;SERVES</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>R&#xE9;serve l&#xE9;gale</td>
                            <td><span t-out="bpvar5" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>R&#xE9;serves statutaires ou contractuelles</td>
                            <td><span t-out="bpvar6" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>R&#xE9;serves r&#xE9;glement&#xE9;es</td><td><span t-out="bpvar7" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td></tr>
                        <tr>
                            <td>Autres r&#xE9;serves</td>
                            <td><span t-out="bpvar8" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Report &#xE0; nouveau</td>
                            <td><span t-out="bpvar9" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td><strong>R&#xC9;SULTAT DE L'EXERCICE [b&#xE9;n&#xE9;fice ou perte]</strong></td>
                            <td><span t-out="bpvar10" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Subventions d'investissement</td>
                            <td><span t-out="bpvar11" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Provisions r&#xE9;glement&#xE9;es</td>
                            <td><span t-out="bpvar12" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td class="text-end"><strong>TOTAL I</strong></td>
                            <td><span t-out="pt1" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td><strong>PROVISIONS</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>Provisions pour risques</td>
                            <td><span t-out="bpvar13" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Provisions pour charges</td>
                            <td><span t-out="bpvar14" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td class="text-end"><strong>TOTAL II</strong></td>
                            <td><span t-out="pt2" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td><strong>DETTES</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>Emprunts obligataires convertibles</td>
                            <td><span t-out="bpvar15" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Autres emprunts obligataires</td>
                            <td><span t-out="bpvar16" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Emprunts et dettes aupr&#xE8;s des &#xE9;tablissements de cr&#xE9;dit</td>
                            <td><span t-out="bpvar17" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Emprunts et dettes financi&#xE8;res diverses</td>
                            <td><span t-out="bpvar18" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Avances et acomptes re&#xE7;us sur commandes en cours</td>
                            <td><span t-out="bpvar19" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Dettes fournisseurs et comptes rattach&#xE9;s </td>
                            <td><span t-out="bpvar20" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Dettes fiscales et sociales</td>
                            <td><span t-out="bpvar21" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Dettes sur immobilisations et comptes rattach&#xE9;s</td>
                            <td><span t-out="bpvar22" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Autres dettes</td>
                            <td><span t-out="bpvar23" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Instruments de tr&#xE9;sorerie</td>
                            <td><span t-out="bpvar24" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Produits constat&#xE9;s d'avance</td>
                            <td><span t-out="bpvar25" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td class="text-end"><strong>TOTAL III</strong></td>
                            <td><span t-out="pt3" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>&#xC9;carts de conversion passif <font face="Times-Roman">( IV )</font></td>
                            <td><span t-out="bpvar26" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td><strong>TOTAL G&#xC9;N&#xC9;RAL (I + II + III + IV)</strong></td>
                            <td><span t-out="passif" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>&amp;nbsp;</td>
                            <td>&amp;nbsp;</td>
                        </tr>
                        <tr>
                            <td><strong>ACTIF - PASSIF</strong></td>
                            <td><span t-out="round(actif-passif,2)" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </t>
    </t>
</template>
</data>
</odoo>

```

## File: views\report_l10nfrresultat.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
<template id="report_l10nfrresultat">
    <t t-call="web.html_container">
        <t t-call="web.internal_layout">
            <div class="page">
                <h2>Compte de résultat</h2>
                <div class="row mt32 mb32">
                    <div class="col-3">
                        <span t-out="res_company.name"/>
                        <br/>au
                        <span t-out="time.strftime('%d-%m-%Y', time.strptime(date_stop,'%Y-%m-%d'))"/>
                    </div>
                    <div class="col-3">
                        <p>Imprimé le
                            <span t-out="time.strftime('%d-%m-%Y')"/>
                            <br/>Tenue de compte:
                            <span t-out="res_company.currency_id.name"/>
                        </p>
                    </div>
                </div>
                <table class="table table-sm">
                    <thead>
                        <tr>
                            <th>Charges (hors taxes)</th>
                            <th></th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td><strong>CHARGES D'EXPLOITATION</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>Achat de marchandises</td>
                            <td>
                                <span class="text-end" t-out="cdrc1" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Variation des stocks</td>
                            <td>
                                <span class="text-end" t-out="cdrc2" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Achats de mati&#xE8;res premi&#xE8;res et autres approvisionnements</td>
                            <td>
                                <span class="text-end" t-out="cdrc3" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Variation des stocks</td>
                            <td>
                                <span class="text-end" t-out="cdrc4" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Autres achats et charges externes</td>
                            <td>
                                <span class="text-end" t-out="cdrc5" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Redevances de cr&#xE9;dit-bail mobilier</td>
                            <td>
                                <span class="text-end" t-out="cdrc6" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Redevances de cr&#xE9;dit-bail immobilier</td>
                            <td>
                                <span class="text-end" t-out="cdrc7" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Imp&#xF4;ts, taxes et versements assimil&#xE9;s</td>
                            <td>
                                <span class="text-end" t-out="cdrc8" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Salaires et traitements</td>
                            <td>
                                <span class="text-end" t-out="cdrc9" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Charges sociales</td>
                            <td>
                                <span class="text-end" t-out="cdrc10" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Dotation aux amortissements et aux d&#xE9;pr&#xE9;ciations</td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>Sur immobilisations : dotations aux amortissements</td>
                            <td>
                                <span class="text-end" t-out="cdrc11" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Sur immobilisations : dotations aux d&#xE9;pr&#xE9;ciations</td>
                            <td>
                                <span class="text-end" t-out="cdrc12" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Sur actif circulant : dotations aux d&#xE9;pr&#xE9;ciations</td>
                            <td>
                                <span class="text-end" t-out="cdrc13" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Dotations aux provisions</td>
                            <td>
                                <span class="text-end" t-out="cdrc14" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Autres charges</td>
                            <td>
                                <span class="text-end" t-out="cdrc15" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-end"><strong>TOTAL I</strong></td>
                            <td>
                                <span class="text-end" t-out="ct1" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td><strong>Quotes-parts de r&#xE9;sultat sur op&#xE9;rations faites en commun ( II )</strong></td>
                            <td>
                                <span class="text-end" t-out="cdrc16" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>                            
                            <td><strong>CHARGES FINANCI&#xC8;RES</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>Dotations aux amortissements, aux d&#xE9;pr&#xE9;ciations et aux provisions</td>
                            <td>
                                <span class="text-end" t-out="cdrc17" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Int&#xE9;r&#xEA;ts et charges assimil&#xE9;es</td>
                            <td>
                                <span class="text-end" t-out="cdrc18" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Diff&#xE9;rences n&#xE9;gatives de change</td>
                            <td>
                                <span class="text-end" t-out="cdrc19" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Charges nettes sur cessions de valeurs mobili&#xE8;res de placement</td>
                            <td>
                                <span class="text-end" t-out="cdrc20" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-end"><strong>TOTAL III</strong></td>
                            <td>
                                <span class="text-end" t-out="ct3" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td><strong>CHARGES EXCEPTIONNELLES</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>Sur op&#xE9;rations de gestion</td>
                            <td>
                                <span class="text-end" t-out="cdrc21" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Sur op&#xE9;rations en capital</td>
                            <td>
                                <span class="text-end" t-out="cdrc22" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Dotations aux amortissements, aux d&#xE9;pr&#xE9;ciations et aux provisions</td>
                            <td>
                                <span class="text-end" t-out="cdrc23" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-end"><strong>TOTAL IV</strong></td>
                            <td>
                                <span class="text-end" t-out="ct4" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td><strong>Participation des salariés aux résultats ( V )</strong></td>
                            <td>
                                <span class="text-end" t-out="cdrc24" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td><strong>Impôts sur les bénéfices ( VI )</strong></td>
                            <td>
                                <span class="text-end" t-out="cdrc25" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-end"><strong>TOTAL CHARGES ( I + II + III + IV+ V+ VI )</strong></td>
                            <td>
                                <span class="text-end" t-out="charges" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>

                    </tbody>
                </table>
                <table class="table table-sm">
                    <thead>
                        <tr>
                            <th>PRODUITS (hors taxes)</th>
                            <th></th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td><strong>PRODUITS D'EXPLOITATION</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>Vente de marchandises</td>
                            <td>
                                <span class="text-end" t-out="cdrp1" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Production vendue [biens et services]</td>
                            <td>
                                <span class="text-end" t-out="cdrp2" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td><strong>Sous-total A - Montant net du chiffre d'affaires</strong></td>
                            <td>
                                <span class="text-end" t-out="pta" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Production stock&#xE9;e</td>
                            <td>
                                <span class="text-end" t-out="cdrp3" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Production immobilis&#xE9;e</td>
                            <td>
                                <span class="text-end" t-out="cdrp4" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Subventions d'exploitation</td>
                            <td>
                                <span class="text-end" t-out="cdrp5" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Reprises sur provisions, d&#xE9;pr&#xE9;ciations (et amortissements) et transferts de charges</td>
                            <td>
                                <span class="text-end" t-out="cdrp6" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Autres produits</td>
                            <td>
                                <span class="text-end" t-out="cdrp7" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td><strong>Sous-total B</strong></td>
                            <td>
                                <span class="text-end" t-out="ptb" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-end"><strong>TOTAL I ( A + B )</strong></td>
                            <td>
                                <span class="text-end" t-out="pt1" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td><strong>Quotes-parts de r&#xE9;sultat sur op&#xE9;rations faites en commun (II)</strong></td>
                            <td>
                                <span class="text-end" t-out="cdrp8" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td><strong>PRODUITS FINANCIERS</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>De participation</td>
                            <td>
                                <span class="text-end" t-out="cdrp9" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>D'autres valeurs mobili&#xE8;res et cr&#xE9;ances de l'actif immobilis&#xE9;</td>
                            <td>
                                <span class="text-end" t-out="cdrp10" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Autres int&#xE9;r&#xEA;ts et produits assimil&#xE9;s</td>
                            <td>
                                <span class="text-end" t-out="cdrp11" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Reprises sur provisions, d&#xE9;pr&#xE9;ciations et transferts de charges</td>
                            <td>
                                <span class="text-end" t-out="cdrp12" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Diff&#xE9;rences positives de change</td>
                            <td>
                                <span class="text-end" t-out="cdrp13" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Produits nets sur cessions de valeurs mobili&#xE8;res de placement</td>
                            <td>
                                <span class="text-end" t-out="cdrp14" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-end"><strong>TOTAL III</strong></td>
                            <td>
                                <span class="text-end" t-out="pt3" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td><strong>PRODUITS EXCEPTIONNELS</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>Sur op&#xE9;rations de gestion</td>
                            <td>
                                <span class="text-end" t-out="cdrp15" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Sur op&#xE9;rations en capital</td>
                            <td>
                                <span class="text-end" t-out="cdrp16" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Reprises sur provisions, d&#xE9;pr&#xE9;ciations et transferts de charges</td>
                            <td>
                                <span class="text-end" t-out="cdrp17" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-end"><strong>TOTAL IV</strong></td>
                            <td>
                                <span class="text-end" t-out="pt4" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-end"><strong>TOTAL DES PRODUITS ( I + II + III + IV )</strong></td>
                            <td>
                                <span class="text-end" t-out="produits" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-end"><strong>PRODUITS - CHARGES</strong></td>
                            <td>
                                <span class="text-end" t-out="produits-charges" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </t>
    </t>
</template>
</data>
</odoo>

```

