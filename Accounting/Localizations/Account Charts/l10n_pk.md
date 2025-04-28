# Odoo Module: l10n_pk

Category: Accounting/Localizations/Account Charts

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
    'name': 'Pakistan - Accounting',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['pk'],
    'version': '1.1',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Pakistan Accounting Module
=======================================================
Pakistan accounting basic charts and localization.

Activates:

- Chart of Accounts
- Taxes
- Tax Report
- Withholding Tax Report
    """,
    'depends': ['account'],
    'data': [
        'data/account_tax_vat_report.xml',
        'data/account_tax_wh_report.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_tax_vat_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_pk_vat_form" model="account.report">
        <field name="name">Tax Report Pakistan (PK)</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.pk"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="l10n_pk_vat_column_net" model="account.report.column">
                <field name="name">base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="l10n_pk_vat_column_tax" model="account.report.column">
                <field name="name">Tax Amount</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_pk_vat_title_purchases_vat" model="account.report.line">
                <field name="name">Purchases Taxes</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_pk_vat_line_SP_17" model="account.report.line">
                        <field name="name">Standard Purchases Tax 17%</field>
                        <field name="code">PUR_ST_17</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_SP_0_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Purchases Tax 17% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_SP_0_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Purchases Tax 17% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_P_195" model="account.report.line">
                        <field name="name">Purchases Telecommunication Services Tax 19.5%</field>
                        <field name="code">PUR_TL</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_P_195_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Telecommunication Services Tax 19.5% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_P_195_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Telecommunication Services Tax 19.5% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_P_17" model="account.report.line">
                        <field name="name">Purchases Services Tax 17%</field>
                        <field name="code">PUR_SR_17</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_P_17_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 17% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_P_17_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 17% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_P_16" model="account.report.line">
                        <field name="name">Purchases Services Tax 16%</field>
                        <field name="code">PUR_SR_16</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_P_16_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 16% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_P_16_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 16% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_P_15" model="account.report.line">
                        <field name="name">Purchases Services Tax 15%</field>
                        <field name="code">PUR_SR_15</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_P_15_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 15% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_P_15_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 15% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_P_13" model="account.report.line">
                        <field name="name">Purchases Services Tax 13%</field>
                        <field name="code">PUR_SR_13</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_P_13_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 13% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_P_13_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 13% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_P_10" model="account.report.line">
                        <field name="name">Purchases Services Tax 10%</field>
                        <field name="code">PUR_SR_10</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_P_10_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 10% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_P_10_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 10% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_P_8" model="account.report.line">
                        <field name="name">Purchases Services Tax 8%</field>
                        <field name="code">PUR_SR_8</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_P_8_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 8% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_P_8_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 8% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_P_5" model="account.report.line">
                        <field name="name">Purchases Services Tax 5%</field>
                        <field name="code">PUR_SR_5</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_P_5_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 5% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_P_5_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 5% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_P_3" model="account.report.line">
                        <field name="name">Purchases Services Tax 3%</field>
                        <field name="code">PUR_SR_3</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_P_3_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 3% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_P_3_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 3% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_P_2" model="account.report.line">
                        <field name="name">Purchases Services Tax 2%</field>
                        <field name="code">PUR_SR_2</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_P_2_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 2% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_P_2_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 2% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_P_1" model="account.report.line">
                        <field name="name">Purchases Services Tax 1%</field>
                        <field name="code">PUR_SR_1</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_P_1_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 1% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_P_1_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 1% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_P_0" model="account.report.line">
                        <field name="name">Purchases Services Tax 0%</field>
                        <field name="code">PUR_SR_0</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_P_0_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 0% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_P_0_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchases Services Tax 0% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_PP_16" model="account.report.line">
                        <field name="name">Standard Purchases Service Tax Punjab 16%</field>
                        <field name="code">PUR_PUN</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_PP_16_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Purchases Service Tax Punjab 16% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_PP_16_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Purchases Service Tax Punjab 16% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_ICP_16" model="account.report.line">
                        <field name="name">Standard Purchases Service Tax ICT 16%</field>
                        <field name="code">PUR_ICT</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_ICP_16_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Purchases Service Tax ICT 16% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_ICP_16_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Purchases Service Tax ICT 16% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_AJP_16" model="account.report.line">
                        <field name="name">Standard Purchases Service Tax AJK 16%</field>
                        <field name="code">PUR_AJK</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_AJP_16_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Purchases Service Tax AJK 16% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_AJP_16_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Purchases Service Tax AJK 16% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_KP_16" model="account.report.line">
                        <field name="name">Standard Purchases Service Tax KP 15%</field>
                        <field name="code">PUR_KP</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_KP_16_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Purchases Service Tax KP 15% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_KP_16_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Purchases Service Tax KP 15% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_STP_16" model="account.report.line">
                        <field name="name">Standard Purchases Service Tax Balochistan 15%</field>
                        <field name="code">PUR_BAL</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_STP_16_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Purchases Service Tax Balochistan 15% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_STP_16_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Purchases Service Tax Balochistan 15% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_SP_13" model="account.report.line">
                        <field name="name">Standard Purchases Service Tax Sindh 13%</field>
                        <field name="code">PUR_SIN</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_SP_13_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Purchases Service Tax Sindh 13% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_SP_13_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Purchases Service Tax Sindh 13% (Tax)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_pk_vat_title_sales_vat" model="account.report.line">
                <field name="name">Sales Taxes</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_pk_vat_line_SS_17" model="account.report.line">
                        <field name="name">Standard Sales Tax 17%</field>
                        <field name="code">SAL_ST_17</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_SS_17_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Sales Tax 17% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_SS_17_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Sales Tax 17% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_S_195" model="account.report.line">
                        <field name="name">Sales Tax Telecommunication Services 19.5%</field>
                        <field name="code">SAL_TL</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_S_195_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Tax Telecommunication Services 19.5% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_S_195_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Tax Telecommunication Services 19.5% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_S_17" model="account.report.line">
                        <field name="name">Sales Services Tax 17%</field>
                        <field name="code">SAL_SR_17</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_S_17_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 17% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_S_17_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 17% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_S_16" model="account.report.line">
                        <field name="name">Sales Services Tax 16%</field>
                        <field name="code">SAL_SR_16</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_S_16_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 16% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_S_16_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 16% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_S_15" model="account.report.line">
                        <field name="name">Sales Services Tax 15%</field>
                        <field name="code">SAL_SR_15</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_S_15_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 15% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_S_15_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 15% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_S_13" model="account.report.line">
                        <field name="name">Sales Services Tax 13%</field>
                        <field name="code">SAL_SR_13</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_S_13_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 13% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_S_13_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 13% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_S_10" model="account.report.line">
                        <field name="name">Sales Services Tax 10%</field>
                        <field name="code">SAL_SR_10</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_S_10_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 10% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_S_10_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 10% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_S_8" model="account.report.line">
                        <field name="name">Sales Services Tax 8%</field>
                        <field name="code">SAL_SR_8</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_S_8_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 8% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_S_8_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 8% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_S_5" model="account.report.line">
                        <field name="name">Sales Services Tax 5%</field>
                        <field name="code">SAL_SR_5</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_S_5_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 5% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_S_5_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 5% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_S_3" model="account.report.line">
                        <field name="name">Sales Services Tax 3%</field>
                        <field name="code">SAL_SR_3</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_S_3_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 3% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_S_3_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 3% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_S_2" model="account.report.line">
                        <field name="name">Sales Services Tax 2%</field>
                        <field name="code">SAL_SR_2</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_S_2_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 2% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_S_2_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 2% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_S_1" model="account.report.line">
                        <field name="name">Sales Services Tax 1%</field>
                        <field name="code">SAL_SR_1</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_S_1_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 1% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_S_1_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 1% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_S_0" model="account.report.line">
                        <field name="name">Sales Services Tax 0%</field>
                        <field name="code">SAL_SR_0</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_S_0_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 0% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_S_0_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales Services Tax 0% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_SP_16" model="account.report.line">
                        <field name="name">Standard Sales Service Tax Punjab 16%</field>
                        <field name="code">SAL_PUN</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_SP_16_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Sales Service Tax Punjab 16% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_SP_16_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Sales Service Tax Punjab 16% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_ICS_16" model="account.report.line">
                        <field name="name">Standard Sales Service Tax ICT 16%</field>
                        <field name="code">SAL_ICT</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_ICS_16_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Sales Service Tax ICT 16% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_ICS_16_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Sales Service Tax ICT 16% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_AJS_16" model="account.report.line">
                        <field name="name">Standard Sales Service Tax AJK 16%</field>
                        <field name="code">SAL_AJK</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_AJS_16_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Sales Service Tax AJK 16% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_AJS_16_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Sales Service Tax AJK 16% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_KPS_15" model="account.report.line">
                        <field name="name">Standard Sales Service Tax KP 15%</field>
                        <field name="code">SAL_KP</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_KPS_15_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Sales Service Tax KP 15% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_KPS_15_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Sales Service Tax KP 15% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_BAS_15" model="account.report.line">
                        <field name="name">Standard Sales Service Tax Balochistan 15%</field>
                        <field name="code">SAL_BAL</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_BAS_15_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Sales Service Tax Balochistan 15% (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_BAS_15_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Sales Service Tax Balochistan 15% (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_SIS_13" model="account.report.line">
                        <field name="name">Standard Sales Tax Services 13% Sindh</field>
                        <field name="code">SAL_SIN</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_SIS_13_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Sales Tax Services 13% Sindh (Base)</field>
                            </record>
                            <record id="l10n_pk_vat_expression_SIS_13_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Standard Sales Tax Services 13% Sindh (Tax)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_pk_vat_title_NS_vat" model="account.report.line">
                <field name="name">Net Standard Tax</field>
                <field name="hierarchy_level">0</field>
                <field name="code">NET_STANDARD_TAX</field>
                <field name="expression_ids">
                    <record id="l10n_pk_vat_expression_NS_1" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">(SAL_NS_TAX.tax) - (PUR_NS_TAX.tax)</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="l10n_pk_vat_line_NS_S" model="account.report.line">
                        <field name="name">Net Standard Tax Sales (Payable)</field>
                        <field name="code">SAL_NS_TAX</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_NS_S1" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">SAL_ST_17.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_NS_P" model="account.report.line">
                        <field name="name">Net Standard Tax Purchases (Recoverable)</field>
                        <field name="code">PUR_NS_TAX</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_NS_P1" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">PUR_ST_17.tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_pk_vat_title_NST_vat" model="account.report.line">
                <field name="name">Net Service Tax</field>
                <field name="hierarchy_level">0</field>
                <field name="code">NET_SERVICE_TAX</field>
                <field name="expression_ids">
                    <record id="l10n_pk_vat_expression_NST_1" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">(SAL_NST_TAX.tax) - (PUR_NST_TAX.tax)</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="l10n_pk_vat_line_NST_S" model="account.report.line">
                        <field name="name">Net Service Tax Sales (Payable)</field>
                        <field name="code">SAL_NST_TAX</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_NST_S1" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">SAL_TL.tax + SAL_SR_17.tax + SAL_SR_16.tax + SAL_SR_15.tax + SAL_SR_13.tax 
                                                    + SAL_SR_10.tax + SAL_SR_8.tax + SAL_SR_5.tax + SAL_SR_3.tax + SAL_SR_2.tax 
                                                    + SAL_SR_1.tax + SAL_SR_0.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_NST_P" model="account.report.line">
                        <field name="name">Net Service Tax Purchases (Recoverable)</field>
                        <field name="code">PUR_NST_TAX</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_NST_P1" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">PUR_TL.tax + PUR_SR_17.tax + PUR_SR_16.tax + PUR_SR_15.tax + PUR_SR_13.tax 
                                                    + PUR_SR_10.tax + PUR_SR_8.tax + PUR_SR_5.tax + PUR_SR_3.tax + PUR_SR_2.tax 
                                                    + PUR_SR_1.tax + PUR_SR_0.tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_pk_vat_title_NSTP_vat" model="account.report.line">
                <field name="name">Net Standard Service Tax Per Province</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_pk_vat_line_NSTP_PUN" model="account.report.line">
                        <field name="name">Net Due Punjab Tax</field>
                        <field name="code">SAL_NSTP_PUN_TAX</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_NSTP_PUN" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">(SAL_PUN.tax) - (PUR_PUN.tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_NSTP_ICT" model="account.report.line">
                        <field name="name">Net Due ICT</field>
                        <field name="code">SAL_NSTP_ICT_TAX</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_NSTP_ICT" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">SAL_ICT.tax - PUR_ICT.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_NSTP_AJK" model="account.report.line">
                        <field name="name">Net Due AJK</field>
                        <field name="code">SAL_NSTP_AJK_TAX</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_NSTP_AJK" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">SAL_AJK.tax - PUR_AJK.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_NSTP_KP" model="account.report.line">
                        <field name="name">Net Due KP</field>
                        <field name="code">SAL_NSTP_KP_TAX</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_NSTP_KP" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">SAL_KP.tax - PUR_KP.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_NSTP_BAL" model="account.report.line">
                        <field name="name">Net Due Balochistan</field>
                        <field name="code">SAL_NSTP_BAL_TAX</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_NSTP_BAL" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">SAL_BAL.tax - PUR_BAL.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_vat_line_NSTP_SIN" model="account.report.line">
                        <field name="name">Net Due Sindh</field>
                        <field name="code">SAL_NSTP_SIN_TAX</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_vat_expression_NSTP_SIN" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">SAL_SIN.tax - PUR_SIN.tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\account_tax_wh_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_pk_wh_vat_form" model="account.report">
        <field name="name">Withholding Tax Report - Pakistan (PK)</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.pk"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="l10n_pk_wh_vat_column_net" model="account.report.column">
                <field name="name">base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="l10n_pk_wh_vat_column_tax" model="account.report.column">
                <field name="name">Tax Amount</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_pk_wh_vat_title_purchases_vat" model="account.report.line">
                <field name="name">Purchase Withholding Tax</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_pk_wh_vat_line_P1" model="account.report.line">
                        <field name="name">Purchase -1% WH</field>
                        <field name="code">PURWH1</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_P1_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -1% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_P1_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -1% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_P2" model="account.report.line">
                        <field name="name">Purchase -1.5% WH</field>
                        <field name="code">PURWH1.5</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_P2_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -1.5% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_P2_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -1.5% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_P3" model="account.report.line">
                        <field name="name">Purchase -2% WH</field>
                        <field name="code">PURWH2</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_P3_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -2% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_P3_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -2% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_P4" model="account.report.line">
                        <field name="name">Purchase -3.5% WH</field>
                        <field name="code">PURWH3.5</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_P4_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -3.5% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_P4_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -3.5% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_P5" model="account.report.line">
                        <field name="name">Purchase -4% WH</field>
                        <field name="code">PURWH4</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_P5_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -4% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_P5_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -4% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_P6" model="account.report.line">
                        <field name="name">Purchase -5% WH</field>
                        <field name="code">PURWH5K</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_P6_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -5% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_P6_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -5% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_P7" model="account.report.line">
                        <field name="name">Purchase -5.5% WH</field>
                        <field name="code">PURWH5.5</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_P7_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -5.5% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_P7_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -5.5% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_P8" model="account.report.line">
                        <field name="name">Purchase -6% WH</field>
                        <field name="code">PURWH6</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_P8_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -6% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_P8_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -6% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_P9" model="account.report.line">
                        <field name="name">Purchase -7.5% WH</field>
                        <field name="code">PURWH7.5</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_P9_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -7.5% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_P9_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -7.5% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_P10" model="account.report.line">
                        <field name="name">Purchase -8% WH</field>
                        <field name="code">PURWH8</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_P10_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -8% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_P10_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -8% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_P11" model="account.report.line">
                        <field name="name">Purchase -9% WH</field>
                        <field name="code">PURWH9</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_P11_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -9% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_P11_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -9% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_P12" model="account.report.line">
                        <field name="name">Purchase -10% WH</field>
                        <field name="code">PURWH10</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_P12_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -10% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_P12_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -10% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_P13" model="account.report.line">
                        <field name="name">Purchase -11% WH</field>
                        <field name="code">PURWH11</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_P13_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -11% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_P13_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Purchase -11% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_pk_wh_vat_title_sales_vat" model="account.report.line">
                <field name="name">Sales Withholding Tax</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_pk_wh_vat_line_S1" model="account.report.line">
                        <field name="name">Sales -1% WH</field>
                        <field name="code">SALWH1</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_S1_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -1% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_S1_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -1% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_S2" model="account.report.line">
                        <field name="name">Sales -1.5% WH</field>
                        <field name="code">SALWH1.5</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_S2_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -1.5% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_S2_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -1.5% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_S3" model="account.report.line">
                        <field name="name">Sales -4% WH</field>
                        <field name="code">SALWH4</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_S3_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -4% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_S3_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -4% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_S4" model="account.report.line">
                        <field name="name">Sales -5% WH</field>
                        <field name="code">SALWH5</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_S4_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -5% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_S4_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -5% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_S5" model="account.report.line">
                        <field name="name">Sales -5.5% WH</field>
                        <field name="code">SALWH5.5</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_S5_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -5.5% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_S5_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -5.5% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_S6" model="account.report.line">
                        <field name="name">Sales -7.5% WH</field>
                        <field name="code">SALWH7.5</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_S6_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -7.5% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_S6_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -7.5% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_S7" model="account.report.line">
                        <field name="name">Sales -8% WH</field>
                        <field name="code">SALWH8</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_S7_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -8% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_S7_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -8% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_S8" model="account.report.line">
                        <field name="name">Sales -9% WH</field>
                        <field name="code">SALWH9</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_S8_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -9% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_S8_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -9% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_S9" model="account.report.line">
                        <field name="name">Sales -10% WH</field>
                        <field name="code">SALWH10</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_S9_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -10% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_S9_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -10% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_pk_wh_vat_line_S10" model="account.report.line">
                        <field name="name">Sales -11% WH</field>
                        <field name="code">SALWH11</field>
                        <field name="expression_ids">
                            <record id="l10n_pk_wh_vat_expression_S10_base" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -11% WH (Base)</field>
                            </record>
                            <record id="l10n_pk_wh_vat_expression_S10_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Sales -11% WH (Tax)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-pk.csv

```csv
"id","name","code","account_type","tag_ids","reconcile"
"l10n_pk_1111001","Building","1111001","asset_fixed","","False"
"l10n_pk_1111002","Furniture and Fixture","1111002","asset_fixed","","False"
"l10n_pk_1111003","Tools and Equipment","1111003","asset_fixed","","False"
"l10n_pk_1111004","Plant and Machinery","1111004","asset_fixed","","False"
"l10n_pk_1112001","ERP System","1112001","asset_non_current","","False"
"l10n_pk_1113000","Investment Property","1113000","asset_non_current","","False"
"l10n_pk_1114000","Long Term Investments","1114000","asset_non_current","","False"
"l10n_pk_1115000","Long Term Deposits","1115000","asset_non_current","","False"
"l10n_pk_1116000","Biological Assets","1116000","asset_non_current","","False"
"l10n_pk_1117000","Investments in Associates","1117000","asset_non_current","","False"
"l10n_pk_1118000","Investments in Jointly Controlled Entities","1118000","asset_non_current","","False"
"l10n_pk_1119000","Other Financial Assets","1119000","asset_non_current","","False"
"l10n_pk_1121001","Receivable from Customers","1121001","asset_receivable","","True"
"l10n_pk_1122001","Advances to Suppliers","1122001","asset_current","","False"
"l10n_pk_1122002","Withholding Tax- Advance","1122002","asset_current","","False"
"l10n_pk_1122003","Loan to Employees","1122003","asset_current","","False"
"l10n_pk_1123001","Security Deposits","1123001","asset_current","","False"
"l10n_pk_1123002","Prepaid Rent","1123002","asset_current","","False"
"l10n_pk_1123003","Income Tax Refundable","1123003","asset_current","","False"
"l10n_pk_1123004","Sales Tax Refundable","1123004","asset_current","","False"
"l10n_pk_1124000","Other Current Assets","1124000","asset_current","","False"
"l10n_pk_1125001","Stock in Hand","1125001","asset_current","","False"
"l10n_pk_1125002","Work in Process","1125002","asset_current","","False"
"l10n_pk_1125003","Finished Goods","1125003","asset_current","","False"
"l10n_pk_2111000","Share Capital","2111000","equity","","False"
"l10n_pk_2111002","Reserves","2112000","equity","","False"
"l10n_pk_2113000","Revaluation Surplus on Property and Equipment","2113000","equity","","False"
"l10n_pk_2211001","Long term Loan","2211001","liability_non_current","","False"
"l10n_pk_2211002","Liabilities Against Assets Subject to Finance Lease","2211002","liability_non_current","","False"
"l10n_pk_2212000","Deferred Liabilities","2212000","liability_non_current","","False"
"l10n_pk_2221001","Payable to Suppliers","2221001","liability_payable","","True"
"l10n_pk_2221002","Accrued Expenses","2221002","liability_current","","True"
"l10n_pk_2221003","Salaries Payable","2221003","liability_payable","","True"
"l10n_pk_2221004","Mark-up accrued","2221004","liability_payable","","True"
"l10n_pk_2221005","Sales Tax Payable","2221005","liability_current","","True"
"l10n_pk_2221006","Withholding Tax Payable","2221006","liability_current","","True"
"l10n_pk_2221007","Payable to Canteen","2221007","liability_payable","","True"
"l10n_pk_2222001","Current Portion of Long Term Liabilities","2222001","liability_current","","False"
"l10n_pk_2223001","Provision for Employee Benefit","2223001","liability_current","","False"
"l10n_pk_2223002","Other Provision","2223002","liability_current","","False"
"l10n_pk_2224000","Unpaid Dividend","2224000","liability_current","","False"
"l10n_pk_2225000","Unclaimed Dividend","2225000","liability_current","","False"
"l10n_pk_2226000","Suspense Account","2226000","liability_current","","False"
"l10n_pk_3111001","Sales Income","3111001","income","","False"
"l10n_pk_3112001","Bank Profit","3112001","income","","False"
"l10n_pk_3112002","Employee Fine","3112002","income","","False"
"l10n_pk_3112003","Misc Income","3112003","income_other","","False"
"l10n_pk_3112004","Cash Discount Gain","3112004","income_other","","False"
"l10n_pk_4111001","Purchases","4111001","expense","","False"
"l10n_pk_4111002","Carriage Inwards","4111002","expense","","False"
"l10n_pk_4111003","Cost of Goods Sold","4111003","expense","","False"
"l10n_pk_4211001","Salary and Wages","4211001","expense","","False"
"l10n_pk_4211002","Other Staff Benefits","4211002","expense","","False"
"l10n_pk_4221001","Rent Rates and Taxes","4221001","expense","","False"
"l10n_pk_4221002","Traveling and Conveyance","4221002","expense","","False"
"l10n_pk_4221003","Vehicle Running Expenses","4221003","expense","","False"
"l10n_pk_4221004","Telephone and Internet","4221004","expense","","False"
"l10n_pk_4221005","Electricity Fitting","4221005","expense","","False"
"l10n_pk_4221006","Printing and Stationery","4221006","expense","","False"
"l10n_pk_4221007","Fee and Subscription","4221007","expense","","False"
"l10n_pk_4221008","Legal and Professional Charges","4221008","expense","","False"
"l10n_pk_4221009","Office Consumables and Supplies","4221009","expense","","False"
"l10n_pk_4221010","Entertainment Expenses","4221010","expense","","False"
"l10n_pk_4221011","Repair and maintenance","4221011","expense","","False"
"l10n_pk_4221012","Postage Expenses","4221012","expense","","False"
"l10n_pk_4221013","Canteen Expenses","4221013","expense","","False"
"l10n_pk_4221014","Bad Debts Written Off","4221014","expense","","False"
"l10n_pk_4221015","Charity and Donations","4221015","expense","","False"
"l10n_pk_4221016","Insurance","4221016","expense","","False"
"l10n_pk_4221017","Misc Expenses","4221017","expense","","False"
"l10n_pk_4221018","Auditor's Remuneration","4221018","expense","","False"
"l10n_pk_4221019","Depreciation Expenses","4221019","expense","","False"
"l10n_pk_4221020","Electricity Expenses","4221020","expense","","False"
"l10n_pk_4221021","Suspense Account","4221021","expense","","False"
"l10n_pk_4311001","Advertisement and Marketing","4311001","expense","","False"
"l10n_pk_4311002","Commission on Sales","4311002","expense","","False"
"l10n_pk_4311003","Freight Outward","4311003","expense","","False"
"l10n_pk_4311004","Sample Expense","4311004","expense","","False"
"l10n_pk_4411001","Bank Charges and Commission","4411001","expense","","False"
"l10n_pk_4411002","Foreign Exchange Difference","4411002","expense","","False"
"l10n_pk_4411003","Cash Discount Loss","4411003","expense","","False"
"l10n_pk_4511000","Income Tax Expense","4511000","expense","","False"

```

## File: data\template\account.group-pk.csv

```csv
"id","code_prefix_start","code_prefix_end","name"
"l10n_pk_group_1","1","","Assets"
"l10n_pk_group_111","111","","Non Current Assets"
"l10n_pk_group_1111","1111","","Property, Plant and Equipment"
"l10n_pk_group_1112","1112","","Intangible Assets"
"l10n_pk_group_1113","1113","","Investment Property"
"l10n_pk_group_1114","1114","","Long Term Investments"
"l10n_pk_group_1115","1115","","Long Term Deposits"
"l10n_pk_group_1116","1116","","Biological Assets"
"l10n_pk_group_1117","1117","","Investments in Associates"
"l10n_pk_group_1118","1118","","Investments in Jointly Controlled Entities"
"l10n_pk_group_1119","1119","","Other Financial Assets"
"l10n_pk_group_112","112","","Current Assets"
"l10n_pk_group_1121","1121","","Trade Receivable"
"l10n_pk_group_1122","1122","","Loans and Advances"
"l10n_pk_group_1123","1123","","Trade Deposits and Short Term Prepayments"
"l10n_pk_group_1124","1124","","Other Current Assets"
"l10n_pk_group_1125","1125","","Stock in Trade"
"l10n_pk_group_1126","1126","","Cash and Bank"
"l10n_pk_group_2","2","","Equity and Liabilities"
"l10n_pk_group_21","21","","Equity"
"l10n_pk_group_2111","2111","","Share Capital"
"l10n_pk_group_2112","2112","","Reserves"
"l10n_pk_group_2113","2113","","Revaluation Surplus on Property and Equipment"
"l10n_pk_group_22","22","","Liabilities"
"l10n_pk_group_221","221","","Non Current Liabilities"
"l10n_pk_group_2211","2211","","Long Term Financing"
"l10n_pk_group_2212","2212","","Deferred Liabilities"
"l10n_pk_group_222","222","","Current Liabilities"
"l10n_pk_group_2221","2221","","Trade and Payables"
"l10n_pk_group_2222","2222","","Short Term Borrowings"
"l10n_pk_group_2223","2223","","Provisions"
"l10n_pk_group_2224","2224","","Unpaid Dividend"
"l10n_pk_group_2225","2225","","Unclaimed Dividend"
"l10n_pk_group_3","3","","Revenue"
"l10n_pk_group_3111","3111","","Sales"
"l10n_pk_group_3112","3112","","Other Income"
"l10n_pk_group_4","4","","Expenses"
"l10n_pk_group_41","41","","Cost of Sales"
"l10n_pk_group_42","42","","General and Administrative Expenses"
"l10n_pk_group_421","421","","Staff Cost"
"l10n_pk_group_422","422","","Other Operating and General Expenses"
"l10n_pk_group_43","43","","Selling and Distribution Expenses"
"l10n_pk_group_44","44","","Banking and Finance Costs"
"l10n_pk_group_45","45","","Income Tax Expense"

```

## File: data\template\account.tax-pk.csv

```csv
"id","name","type_tax_use","amount","amount_type","description","invoice_label","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids"
"purchases_tax_17","17%","purchase","17.0","percent","Standard Purchases Tax 17%","Standard Purchases Tax 17%","tax_group_pk_17","base","invoice","","+Standard Purchases Tax 17% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Standard Purchases Tax 17% (Tax)"
"","","","","","","","","base","refund","","-Standard Purchases Tax 17% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Standard Purchases Tax 17% (Tax)"
"purchases_tax_services_19_5","19.5% T","purchase","19.5","percent","Purchases Tax Telecommunication Services 19.5%","Purchases Tax Telecommunication Services 19.5%","tax_group_pk_195","base","invoice","","+Purchases Telecommunication Services Tax 19.5% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Purchases Telecommunication Services Tax 19.5% (Tax)"
"","","","","","","","","base","refund","","-Purchases Telecommunication Services Tax 19.5% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Purchases Telecommunication Services Tax 19.5% (Tax)"
"purchases_tax_services_17","17% S","purchase","17.0","percent","Purchases Tax Services 17%","Purchases Tax Services 17%","tax_group_pk_17","base","invoice","","+Purchases Services Tax 17% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Purchases Services Tax 17% (Tax)"
"","","","","","","","","base","refund","","-Purchases Services Tax 17% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Purchases Services Tax 17% (Tax)"
"purchases_tax_services_16","16% S","purchase","16.0","percent","Purchases Tax Services 16%","Purchases Tax Services 16%","tax_group_pk_16","base","invoice","","+Purchases Services Tax 16% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Purchases Services Tax 16% (Tax)"
"","","","","","","","","base","refund","","-Purchases Services Tax 16% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Purchases Services Tax 16% (Tax)"
"purchases_tax_services_16_punjab","16% S Pun","purchase","16.0","percent","Standard Purchases Service Tax Punjab 16%","Standard Purchases Service Tax Punjab 16%","tax_group_pk_16","base","invoice","","+Standard Purchases Service Tax Punjab 16% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Standard Purchases Service Tax Punjab 16% (Tax)"
"","","","","","","","","base","refund","","-Standard Purchases Service Tax Punjab 16% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Standard Purchases Service Tax Punjab 16% (Tax)"
"purchases_tax_services_16_ict","16% S ICT","purchase","16.0","percent","Standard Purchases Service Tax Islamabad Capital Territory 16%","Standard Purchases Service Tax Islamabad Capital Territory 16%","tax_group_pk_16","base","invoice","","+Standard Purchases Service Tax ICT 16% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Standard Purchases Service Tax ICT 16% (Tax)"
"","","","","","","","","base","refund","","-Standard Purchases Service Tax ICT 16% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Standard Purchases Service Tax ICT 16% (Tax)"
"purchases_tax_services_16_ajk","16% S AJK","purchase","16.0","percent","Standard Purchases Service Tax Azad Jammu and Kashmir 16%","Standard Purchases Service Tax Azad Jammu and Kashmir 16%","tax_group_pk_16","base","invoice","","+Standard Purchases Service Tax AJK 16% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Standard Purchases Service Tax AJK 16% (Tax)"
"","","","","","","","","base","refund","","-Standard Purchases Service Tax AJK 16% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Standard Purchases Service Tax AJK 16% (Tax)"
"purchases_tax_services_15","15% S","purchase","15.0","percent","Purchases Tax Services 15%","Purchases Tax Services 15%","tax_group_pk_15","base","invoice","","+Purchases Services Tax 15% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Purchases Services Tax 15% (Tax)"
"","","","","","","","","base","refund","","-Purchases Services Tax 15% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Purchases Services Tax 15% (Tax)"
"purchases_tax_services_15_kp","15% S KP","purchase","15.0","percent","Standard Purchases Service Tax Khyber Pakhtunkhwa 15%","Standard Purchases Service Tax Khyber Pakhtunkhwa 15%","tax_group_pk_15","base","invoice","","+Standard Purchases Service Tax KP 15% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Standard Purchases Service Tax KP 15% (Tax)"
"","","","","","","","","base","refund","","-Standard Purchases Service Tax KP 15% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Standard Purchases Service Tax KP 15% (Tax)"
"purchases_tax_services_15_balochistan","15% S Balo","purchase","15.0","percent","Standard Purchases Service Tax Balochistan 15%","Standard Purchases Service Tax Balochistan 15%","tax_group_pk_15","base","invoice","","+Standard Purchases Service Tax Balochistan 15% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Standard Purchases Service Tax Balochistan 15% (Tax)"
"","","","","","","","","base","refund","","-Standard Purchases Service Tax Balochistan 15% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Standard Purchases Service Tax Balochistan 15% (Tax)"
"purchases_tax_services_13","13% S","purchase","13.0","percent","Purchases Tax Services 13%","Purchases Tax Services 13%","tax_group_pk_13","base","invoice","","+Purchases Services Tax 13% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Purchases Services Tax 13% (Tax)"
"","","","","","","","","base","refund","","-Purchases Services Tax 13% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Purchases Services Tax 13% (Tax)"
"purchases_tax_services_13_sindh","13% S Sindh","purchase","13.0","percent","Standard Purchases Tax Services 13% Sindh","Standard Purchases Tax Services 13% Sindh","tax_group_pk_13","base","invoice","","+Standard Purchases Service Tax Sindh 13% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Standard Purchases Service Tax Sindh 13% (Tax)"
"","","","","","","","","base","refund","","-Standard Purchases Service Tax Sindh 13% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Standard Purchases Service Tax Sindh 13% (Tax)"
"purchases_tax_services_10","10% S","purchase","10.0","percent","Purchases Tax Services 10%","Purchases Tax Services 10%","tax_group_pk_10","base","invoice","","+Purchases Services Tax 10% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Purchases Services Tax 10% (Tax)"
"","","","","","","","","base","refund","","-Purchases Services Tax 10% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Purchases Services Tax 10% (Tax)"
"purchases_tax_services_8","8% S","purchase","8.0","percent","Purchases Tax Services 8%","Purchases Tax Services 8%","tax_group_pk_8","base","invoice","","+Purchases Services Tax 8% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Purchases Services Tax 8% (Tax)"
"","","","","","","","","base","refund","","-Purchases Services Tax 8% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Purchases Services Tax 8% (Tax)"
"purchases_tax_services_5","5% S","purchase","5.0","percent","Purchases Tax Services 5%","Purchases Tax Services 5%","tax_group_pk_5","base","invoice","","+Purchases Services Tax 5% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Purchases Services Tax 5% (Tax)"
"","","","","","","","","base","refund","","-Purchases Services Tax 5% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Purchases Services Tax 5% (Tax)"
"purchases_tax_services_3","3% S","purchase","3.0","percent","Purchases Tax Services 3%","Purchases Tax Services 3%","tax_group_pk_3","base","invoice","","+Purchases Services Tax 3% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Purchases Services Tax 3% (Tax)"
"","","","","","","","","base","refund","","-Purchases Services Tax 3% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Purchases Services Tax 3% (Tax)"
"purchases_tax_services_2","2% S","purchase","2.0","percent","Purchases Tax Services 2%","Purchases Tax Services 2%","tax_group_pk_2","base","invoice","","+Purchases Services Tax 2% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Purchases Services Tax 2% (Tax)"
"","","","","","","","","base","refund","","-Purchases Services Tax 2% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Purchases Services Tax 2% (Tax)"
"purchases_tax_services_1","1% S","purchase","1.0","percent","Purchases Tax Services 1%","Purchases Tax Services 1%","tax_group_pk_1","base","invoice","","+Purchases Services Tax 1% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Purchases Services Tax 1% (Tax)"
"","","","","","","","","base","refund","","-Purchases Services Tax 1% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Purchases Services Tax 1% (Tax)"
"purchases_tax_services_0","0% S","purchase","0.0","percent","Purchases Tax Services 0%","Purchases Tax Services 0%","tax_group_pk_0","base","invoice","","+Purchases Services Tax 0% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Purchases Services Tax 0% (Tax)"
"","","","","","","","","base","refund","","-Purchases Services Tax 0% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Purchases Services Tax 0% (Tax)"
"pk_sales_tax_17","17%","sale","17.0","percent","Standard Sales Tax 17%","Standard Sales Tax 17%","tax_group_pk_17","base","invoice","","+Standard Sales Tax 17% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Standard Sales Tax 17% (Tax)"
"","","","","","","","","base","refund","","-Standard Sales Tax 17% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Standard Sales Tax 17% (Tax)"
"pk_sales_tax_services_19_5","19.5% T","sale","19.5","percent","Sales Tax Telecommunication Services 19.5%","Sales Tax Telecommunication Services 19.5%","tax_group_pk_195","base","invoice","","+Sales Tax Telecommunication Services 19.5% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Sales Tax Telecommunication Services 19.5% (Tax)"
"","","","","","","","","base","refund","","+Sales Tax Telecommunication Services 19.5% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","+Sales Tax Telecommunication Services 19.5% (Tax)"
"pk_sales_tax_services_17","17% S","sale","17.0","percent","Sales Tax Services 17%","Sales Tax Services 17%","tax_group_pk_17","base","invoice","","+Sales Services Tax 17% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Sales Services Tax 17% (Tax)"
"","","","","","","","","base","refund","","-Sales Services Tax 17% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Sales Services Tax 17% (Tax)"
"pk_sales_tax_services_16","16% S","sale","16.0","percent","Sales Tax Services 16%","Sales Tax Services 16%","tax_group_pk_16","base","invoice","","+Sales Services Tax 16% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Sales Services Tax 16% (Tax)"
"","","","","","","","","base","refund","","-Sales Services Tax 16% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Sales Services Tax 16% (Tax)"
"pk_sales_tax_services_16_punjab","16% S Pun","sale","16.0","percent","Standard Sales Service Tax Punjab 16%","Standard Sales Service Tax Punjab 16%","tax_group_pk_16","base","invoice","","+Standard Sales Service Tax Punjab 16% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Standard Sales Service Tax Punjab 16% (Tax)"
"","","","","","","","","base","refund","","-Standard Sales Service Tax Punjab 16% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Standard Sales Service Tax Punjab 16% (Tax)"
"pk_sales_tax_services_16_ict","16% S ICT","sale","16.0","percent","Standard Sales Service Tax Islamabad Capital Territory 16%","Standard Sales Service Tax Islamabad Capital Territory 16%","tax_group_pk_16","base","invoice","","+Standard Sales Service Tax ICT 16% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Standard Sales Service Tax ICT 16% (Tax)"
"","","","","","","","","base","refund","","-Standard Sales Service Tax ICT 16% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Standard Sales Service Tax ICT 16% (Tax)"
"pk_sales_tax_services_16_ajk","16% S AJK","sale","16.0","percent","Standard Sales Service Tax Azad Jammu and Kashmir 16%","Standard Sales Service Tax Azad Jammu and Kashmir 16%","tax_group_pk_16","base","invoice","","+Standard Sales Service Tax AJK 16% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Standard Sales Service Tax AJK 16% (Tax)"
"","","","","","","","","base","refund","","-Standard Sales Service Tax AJK 16% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Standard Sales Service Tax AJK 16% (Tax)"
"pk_sales_tax_services_15","15% S","sale","15.0","percent","Sales Tax Services 15%","Sales Tax Services 15%","tax_group_pk_15","base","invoice","","+Sales Services Tax 15% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Sales Services Tax 15% (Tax)"
"","","","","","","","","base","refund","","-Sales Services Tax 15% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Sales Services Tax 15% (Tax)"
"pk_sales_tax_services_15_kp","15% S KP","sale","15.0","percent","Standard Sales Service Tax Khyber Pakhtunkhwa 15%","Standard Sales Service Tax Khyber Pakhtunkhwa 15%","tax_group_pk_15","base","invoice","","+Standard Sales Service Tax KP 15% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Standard Sales Service Tax KP 15% (Tax)"
"","","","","","","","","base","refund","","-Standard Sales Service Tax KP 15% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Standard Sales Service Tax KP 15% (Tax)"
"pk_sales_tax_services_15_balochistan","15% S Balo","sale","15.0","percent","Standard Sales Service Tax Balochistan 15%","Standard Sales Service Tax Balochistan 15%","tax_group_pk_15","base","invoice","","+Standard Sales Service Tax Balochistan 15% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Standard Sales Service Tax Balochistan 15% (Tax)"
"","","","","","","","","base","refund","","-Standard Sales Service Tax Balochistan 15% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Standard Sales Service Tax Balochistan 15% (Tax)"
"pk_sales_tax_services_13","13% S","sale","13.0","percent","Sales Tax Services 13%","Sales Tax Services 13%","tax_group_pk_13","base","invoice","","+Sales Services Tax 13% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Sales Services Tax 13% (Tax)"
"","","","","","","","","base","refund","","-Sales Services Tax 13% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Sales Services Tax 13% (Tax)"
"pk_sales_tax_services_13_sindh","13% S Sindh","sale","13.0","percent","Standard Sales Tax Services 13% Sindh","Standard Sales Tax Services 13% Sindh","tax_group_pk_13","base","invoice","","+Standard Sales Tax Services 13% Sindh (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Standard Sales Tax Services 13% Sindh (Tax)"
"","","","","","","","","base","refund","","-Standard Sales Tax Services 13% Sindh (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Standard Sales Tax Services 13% Sindh (Tax)"
"pk_sales_tax_services_10","10% S","sale","10.0","percent","Sales Tax Services 10%","Sales Tax Services 10%","tax_group_pk_10","base","invoice","","+Sales Services Tax 10% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Sales Services Tax 10% (Tax)"
"","","","","","","","","base","refund","","-Sales Services Tax 10% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Sales Services Tax 10% (Tax)"
"pk_sales_tax_services_8","8% S","sale","8.0","percent","Sales Tax Services 8%","Sales Tax Services 8%","tax_group_pk_8","base","invoice","","+Sales Services Tax 8% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Sales Services Tax 8% (Tax)"
"","","","","","","","","base","refund","","-Sales Services Tax 8% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Sales Services Tax 8% (Tax)"
"pk_sales_tax_services_5","5% S","sale","5.0","percent","Sales Tax Services 5%","Sales Tax Services 5%","tax_group_pk_5","base","invoice","","+Sales Services Tax 5% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Sales Services Tax 5% (Tax)"
"","","","","","","","","base","refund","","-Sales Services Tax 5% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Sales Services Tax 5% (Tax)"
"pk_sales_tax_services_3","3% S","sale","3.0","percent","Sales Tax Services 3%","Sales Tax Services 3%","tax_group_pk_3","base","invoice","","+Sales Services Tax 3% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Sales Services Tax 3% (Tax)"
"","","","","","","","","base","refund","","-Sales Services Tax 3% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Sales Services Tax 3% (Tax)"
"pk_sales_tax_services_2","2% S","sale","2.0","percent","Sales Tax Services 2%","Sales Tax Services 2%","tax_group_pk_2","base","invoice","","+Sales Services Tax 2% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Sales Services Tax 2% (Tax)"
"","","","","","","","","base","refund","","-Sales Services Tax 2% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Sales Services Tax 2% (Tax)"
"pk_sales_tax_services_1","1% S","sale","1.0","percent","Sales Tax Services 1%","Sales Tax Services 1%","tax_group_pk_1","base","invoice","","+Sales Services Tax 1% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Sales Services Tax 1% (Tax)"
"","","","","","","","","base","refund","","-Sales Services Tax 1% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Sales Services Tax 1% (Tax)"
"pk_sales_tax_services_0","0% S","sale","0.0","percent","Sales Tax Services 0%","Sales Tax Services 0%","tax_group_pk_0","base","invoice","","+Sales Services Tax 0% (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221005","+Sales Services Tax 0% (Tax)"
"","","","","","","","","base","refund","","-Sales Services Tax 0% (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221005","-Sales Services Tax 0% (Tax)"
"pk_tax_wh_P_6","WH TAX -6%","purchase","-6","percent","WH TAX -6%","-6%","tax_group_pk_wt","base","invoice","","+Purchase -6% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221006","+Purchase -6% WH (Tax)"
"","","","","","","","","base","refund","","-Purchase -6% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221006","-Purchase -6% WH (Tax)"
"pk_tax_wh_P_75","WH TAX -7.5%","purchase","-7.5","percent","WH TAX -7.5%","-7.50%","tax_group_pk_wt","base","invoice","","+Purchase -7.5% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221006","+Purchase -7.5% WH (Tax)"
"","","","","","","","","base","refund","","-Purchase -7.5% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221006","-Purchase -7.5% WH (Tax)"
"pk_tax_wh_P_8","WH TAX -8%","purchase","-8","percent","WH TAX -8%","-8%","tax_group_pk_wt","base","invoice","","+Purchase -8% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221006","+Purchase -8% WH (Tax)"
"","","","","","","","","base","refund","","-Purchase -8% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221006","-Purchase -8% WH (Tax)"
"pk_tax_wh_P_9","WH TAX -9%","purchase","-9","percent","WH TAX -9%","-9%","tax_group_pk_wt","base","invoice","","+Purchase -9% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221006","+Purchase -9% WH (Tax)"
"","","","","","","","","base","refund","","-Purchase -9% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221006","-Purchase -9% WH (Tax)"
"pk_tax_wh_S_15","WH TAX -1.5%","sale","-1.5","percent","WH TAX -1.5%","-1.50%","tax_group_pk_wt","base","invoice","","+Sales -1.5% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_1122002","+Sales -1.5% WH (Tax)"
"","","","","","","","","base","refund","","-Sales -1.5% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_1122002","-Sales -1.5% WH (Tax)"
"pk_tax_wh_S_10","WH TAX -10%","sale","-10","percent","WH TAX -10%","-10%","tax_group_pk_wt","base","invoice","","+Sales -10% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_1122002","+Sales -10% WH (Tax)"
"","","","","","","","","base","refund","","-Sales -10% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_1122002","-Sales -10% WH (Tax)"
"pk_tax_wh_S_11","WH TAX -11%","sale","-11","percent","WH TAX -11%","-11%","tax_group_pk_wt","base","invoice","","+Sales -11% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_1122002","+Sales -11% WH (Tax)"
"","","","","","","","","base","refund","","-Sales -11% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_1122002","-Sales -11% WH (Tax)"
"pk_tax_wh_S_4","WH TAX -4%","sale","-4","percent","WH TAX -4%","-4%","tax_group_pk_wt","base","invoice","","+Sales -4% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_1122002","+Sales -4% WH (Tax)"
"","","","","","","","","base","refund","","-Sales -4% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_1122002","-Sales -4% WH (Tax)"
"pk_tax_wh_S_5","WH TAX -5%","sale","-5","percent","WH TAX -5%","-5%","tax_group_pk_wt","base","invoice","","+Sales -5% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_1122002","+Sales -5% WH (Tax)"
"","","","","","","","","base","refund","","-Sales -5% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_1122002","-Sales -5% WH (Tax)"
"pk_tax_wh_S_55","WH TAX -5.5%","sale","-5.5","percent","WH TAX -5.5%","-5.50%","tax_group_pk_wt","base","invoice","","+Sales -5.5% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_1122002","+Sales -5.5% WH (Tax)"
"","","","","","","","","base","refund","","-Sales -5.5% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_1122002","-Sales -5.5% WH (Tax)"
"pk_tax_wh_S_75","WH TAX -7.5%","sale","-7.5","percent","WH TAX -7.5%","-7.50%","tax_group_pk_wt","base","invoice","","+Sales -7.5% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_1122002","+Sales -7.5% WH (Tax)"
"","","","","","","","","base","refund","","-Sales -7.5% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_1122002","-Sales -7.5% WH (Tax)"
"pk_tax_wh_S_8","WH TAX -8%","sale","-8","percent","WH TAX -8%","-8%","tax_group_pk_wt","base","invoice","","+Sales -8% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_1122002","+Sales -8% WH (Tax)"
"","","","","","","","","base","refund","","-Sales -8% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_1122002","-Sales -8% WH (Tax)"
"pk_tax_wh_S_9","WH TAX -9%","sale","-9","percent","WH TAX -9%","-9%","tax_group_pk_wt","base","invoice","","+Sales -9% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_1122002","+Sales -9% WH (Tax)"
"","","","","","","","","base","refund","","-Sales -9% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_1122002","-Sales -9% WH (Tax)"
"pk_tax_wh_P_1","WH TAX -1%","purchase","-1","percent","WH TAX -1%","-1%","tax_group_pk_wt","base","invoice","","+Purchase -1% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221006","+Purchase -1% WH (Tax)"
"","","","","","","","","base","refund","","-Purchase -1% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221006","-Purchase -1% WH (Tax)"
"pk_tax_wh_P_15","WH TAX -1.5%","purchase","-1.5","percent","WH TAX -1.5%","-1.50%","tax_group_pk_wt","base","invoice","","+Purchase -1.5% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221006","+Purchase -1.5% WH (Tax)"
"","","","","","","","","base","refund","","-Purchase -1.5% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221006","-Purchase -1.5% WH (Tax)"
"pk_tax_wh_P_10","WH TAX -10%","purchase","-10","percent","WH TAX -10%","-10%","tax_group_pk_wt","base","invoice","","+Purchase -10% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221006","+Purchase -10% WH (Tax)"
"","","","","","","","","base","refund","","-Purchase -10% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221006","-Purchase -10% WH (Tax)"
"pk_tax_wh_P_11","WH TAX -11%","purchase","-11","percent","WH TAX -11%","-11%","tax_group_pk_wt","base","invoice","","+Purchase -11% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221006","+Purchase -11% WH (Tax)"
"","","","","","","","","base","refund","","-Purchase -11% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221006","-Purchase -11% WH (Tax)"
"pk_tax_wh_P_2","WH TAX -2%","purchase","-2","percent","WH TAX -2%","-2%","tax_group_pk_wt","base","invoice","","+Purchase -2% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221006","+Purchase -2% WH (Tax)"
"","","","","","","","","base","refund","","-Purchase -2% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221006","-Purchase -2% WH (Tax)"
"pk_tax_wh_P_35","WH TAX -3.5%","purchase","-3.5","percent","WH TAX -3.5%","-3.50%","tax_group_pk_wt","base","invoice","","+Purchase -3.5% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221006","+Purchase -3.5% WH (Tax)"
"","","","","","","","","base","refund","","-Purchase -3.5% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221006","-Purchase -3.5% WH (Tax)"
"pk_tax_wh_P_4","WH TAX -4%","purchase","-4","percent","WH TAX -4%","-4%","tax_group_pk_wt","base","invoice","","+Purchase -4% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221006","+Purchase -4% WH (Tax)"
"","","","","","","","","base","refund","","-Purchase -4% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221006","-Purchase -4% WH (Tax)"
"pk_tax_wh_P_5","WH TAX -5%","purchase","-5","percent","WH TAX -5%","-5%","tax_group_pk_wt","base","invoice","","+Purchase -5% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221006","+Purchase -5% WH (Tax)"
"","","","","","","","","base","refund","","-Purchase -5% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221006","-Purchase -5% WH (Tax)"
"pk_tax_wh_P_55","WH TAX -5.5%","purchase","-5.5","percent","WH TAX -5.5%","-5.50%","tax_group_pk_wt","base","invoice","","+Purchase -5.5% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_2221006","+Purchase -5.5% WH (Tax)"
"","","","","","","","","base","refund","","-Purchase -5.5% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_2221006","-Purchase -5.5% WH (Tax)"
"pk_tax_wh_S_1","WH TAX -1%","sale","-1","percent","WH TAX -1%","-1%","tax_group_pk_wt","base","invoice","","+Sales -1% WH (Base)"
"","","","","","","","","tax","invoice","l10n_pk_1122002","+Sales -1% WH (Tax)"
"","","","","","","","","base","refund","","-Sales -1% WH (Base)"
"","","","","","","","","tax","refund","l10n_pk_1122002","-Sales -1% WH (Tax)"

```

## File: data\template\account.tax.group-pk.csv

```csv
"id","country_id","name","tax_payable_account_id","tax_receivable_account_id"
"tax_group_pk_wt","base.pk","Withholding Tax","l10n_pk_2221005","l10n_pk_1123004"
"tax_group_pk_0","base.pk","Tax 0%","l10n_pk_2221005","l10n_pk_1123004"
"tax_group_pk_1","base.pk","Tax 1%","l10n_pk_2221005","l10n_pk_1123004"
"tax_group_pk_2","base.pk","Tax 2%","l10n_pk_2221005","l10n_pk_1123004"
"tax_group_pk_3","base.pk","Tax 3%","l10n_pk_2221005","l10n_pk_1123004"
"tax_group_pk_5","base.pk","Tax 5%","l10n_pk_2221005","l10n_pk_1123004"
"tax_group_pk_8","base.pk","Tax 8%","l10n_pk_2221005","l10n_pk_1123004"
"tax_group_pk_10","base.pk","Tax 10%","l10n_pk_2221005","l10n_pk_1123004"
"tax_group_pk_13","base.pk","Tax 13%","l10n_pk_2221005","l10n_pk_1123004"
"tax_group_pk_15","base.pk","Tax 15%","l10n_pk_2221005","l10n_pk_1123004"
"tax_group_pk_16","base.pk","Tax 16%","l10n_pk_2221005","l10n_pk_1123004"
"tax_group_pk_17","base.pk","Tax 17%","l10n_pk_2221005","l10n_pk_1123004"
"tax_group_pk_195","base.pk","Tax 19.5%","l10n_pk_2221005","l10n_pk_1123004"

```

## File: migrations\1.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'pk')], order="parent_path"):
        env['account.chart.template'].try_loading('pk', company)

```

## File: models\template_pk.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('pk')
    def _get_pk_template_data(self):
        return {
            'property_account_receivable_id': 'l10n_pk_1121001',
            'property_account_payable_id': 'l10n_pk_2221001',
            'property_account_income_categ_id': 'l10n_pk_3111001',
            'property_account_expense_categ_id': 'l10n_pk_4111001',
            'code_digits': '7',
        }

    @template('pk', 'res.company')
    def _get_pk_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.pk',
                'bank_account_code_prefix': '112600',
                'cash_account_code_prefix': '112600',
                'transfer_account_code_prefix': '112600',
                'account_default_pos_receivable_account_id': 'l10n_pk_1121001',
                'account_journal_suspense_account_id': 'l10n_pk_2226000',
                'account_journal_early_pay_discount_loss_account_id': 'l10n_pk_4411003',
                'account_journal_early_pay_discount_gain_account_id': 'l10n_pk_3112004',
                'account_sale_tax_id': 'pk_sales_tax_17',
                'account_purchase_tax_id': 'purchases_tax_17',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_pk

```

