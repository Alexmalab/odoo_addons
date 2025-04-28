# Odoo Module: l10n_tr

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
    'name': 'Türkiye - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['tr'],
    'version': '1.3',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the base module to manage the accounting chart for Türkiye in Odoo
==========================================================================

Türkiye accounting basic charts and localizations
-------------------------------------------------
Activates:

- Chart of Accounts
- Taxes
- Tax Report
    """,
    'author': 'Odoo S.A., Drysharks Consulting and Trading Ltd.',
    'depends': [
        'account',
    ],
    'data': [
        'data/account_tax_report_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="turkey_tax_report" model="account.report">
        <field name="name">Türkiye Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.tr"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tr_base_column" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="tr_tax_column" model="account.report.column">
                <field name="name">Tax</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="purchases_vat" model="account.report.line">
                <field name="name">Purchases VAT</field>
                <field name="sequence">0</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="purchases_vat_line_0" model="account.report.line">
                        <field name="name">Purchases 0% VAT</field>
                        <field name="sequence">1</field>
                        <field name="code">PUR_0</field>
                        <field name="expression_ids">
                            <record id="purchases_vat_expression_0_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_BASE_0</field>
                            </record>
                            <record id="purchases_vat_expression_0_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_TAX_0</field>
                            </record>
                        </field>
                    </record>
                    <record id="purchases_vat_line_1" model="account.report.line">
                        <field name="name">Purchases 1% VAT</field>
                        <field name="sequence">2</field>
                        <field name="code">PUR_1</field>
                        <field name="expression_ids">
                            <record id="purchases_vat_expression_1_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_BASE_1</field>
                            </record>
                            <record id="purchases_vat_expression_1_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_TAX_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="purchases_vat_line_10" model="account.report.line">
                        <field name="name">Purchases 10% VAT</field>
                        <field name="sequence">3</field>
                        <field name="code">PUR_10</field>
                        <field name="expression_ids">
                            <record id="purchases_vat_expression_10_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_BASE_10</field>
                            </record>
                            <record id="purchases_vat_expression_10_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_TAX_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="purchases_vat_line_20" model="account.report.line">
                        <field name="name">Purchases 20% VAT</field>
                        <field name="sequence">4</field>
                        <field name="code">PUR_20</field>
                        <field name="expression_ids">
                            <record id="purchases_vat_expression_20_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_BASE_20</field>
                            </record>
                            <record id="purchases_vat_expression_20_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_TAX_20</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="sales_vat" model="account.report.line">
                <field name="name">Sales VAT</field>
                <field name="sequence">5</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="sales_vat_line_0" model="account.report.line">
                        <field name="name">Export Sales 0%</field>
                        <field name="sequence">6</field>
                        <field name="code">SAL_0</field>
                        <field name="expression_ids">
                            <record id="sales_vat_expression_0_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_BASE_0</field>
                            </record>
                        </field>
                    </record>
                    <record id="sales_vat_line_1" model="account.report.line">
                        <field name="name">Sales 1% VAT</field>
                        <field name="sequence">7</field>
                        <field name="code">SAL_1</field>
                        <field name="expression_ids">
                            <record id="sales_vat_expression_1_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_BASE_1</field>
                            </record>
                            <record id="sales_vat_expression_1_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_TAX_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="sales_vat_line_10" model="account.report.line">
                        <field name="name">Sales 10% VAT</field>
                        <field name="sequence">8</field>
                        <field name="code">SAL_10</field>
                        <field name="expression_ids">
                            <record id="sales_vat_expression_10_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_BASE_10</field>
                            </record>
                            <record id="sales_vat_expression_10_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_TAX_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="sales_vat_line_20" model="account.report.line">
                        <field name="name">Sales 20% VAT</field>
                        <field name="sequence">9</field>
                        <field name="code">SAL_20</field>
                        <field name="expression_ids">
                            <record id="sales_vat_expression_20_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_BASE_20</field>
                            </record>
                            <record id="sales_vat_expression_20_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_TAX_20</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="purchases_wh" model="account.report.line">
                <field name="name">Purchases Withholding</field>
                <field name="sequence">10</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="purchases_wh_line_2" model="account.report.line">
                        <field name="name">Purchases Withholding 20% (2/10)</field>
                        <field name="sequence">11</field>
                        <field name="code">PUR_WH_2</field>
                        <field name="expression_ids">
                            <record id="purchases_wh_expression_2_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_20_BASE_2_10</field>
                            </record>
                            <record id="purchases_wh_expression_2_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_20_TAX_2_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="purchases_wh_line_3" model="account.report.line">
                        <field name="name">Purchases Withholding 20% (3/10)</field>
                        <field name="sequence">12</field>
                        <field name="code">PUR_WH_3</field>
                        <field name="expression_ids">
                            <record id="purchases_wh_expression_3_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_20_BASE_3_10</field>
                            </record>
                            <record id="purchases_wh_expression_3_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_20_TAX_3_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="purchases_wh_line_4" model="account.report.line">
                        <field name="name">Purchases Withholding 20% (4/10)</field>
                        <field name="sequence">13</field>
                        <field name="code">PUR_WH_4</field>
                        <field name="expression_ids">
                            <record id="purchases_wh_expression_4_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_20_BASE_4_10</field>
                            </record>
                            <record id="purchases_wh_expression_4_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_20_TAX_4_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="purchases_wh_line_5" model="account.report.line">
                        <field name="name">Purchases Withholding 20% (5/10)</field>
                        <field name="sequence">14</field>
                        <field name="code">PUR_WH_5</field>
                        <field name="expression_ids">
                            <record id="purchases_wh_expression_5_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_20_BASE_5_10</field>
                            </record>
                            <record id="purchases_wh_expression_5_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_20_TAX_5_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="purchases_wh_line_7" model="account.report.line">
                        <field name="name">Purchases Withholding 20% (7/10)</field>
                        <field name="sequence">15</field>
                        <field name="code">PUR_WH_7</field>
                        <field name="expression_ids">
                            <record id="purchases_wh_expression_7_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_20_BASE_7_10</field>
                            </record>
                            <record id="purchases_wh_expression_7_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_20_TAX_7_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="purchases_wh_line_9" model="account.report.line">
                        <field name="name">Purchases Withholding 20% (9/10)</field>
                        <field name="sequence">16</field>
                        <field name="code">PUR_WH_9</field>
                        <field name="expression_ids">
                            <record id="purchases_wh_expression_9_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_20_BASE_9_10</field>
                            </record>
                            <record id="purchases_wh_expression_9_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_20_TAX_9_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="purchases_wh_line_10" model="account.report.line">
                        <field name="name">Purchases Withholding 20% (10/10)</field>
                        <field name="sequence">17</field>
                        <field name="code">PUR_WH_10</field>
                        <field name="expression_ids">
                            <record id="purchases_wh_expression_10_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_20_BASE_10_10</field>
                            </record>
                            <record id="purchases_wh_expression_10_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">PUR_20_TAX_10_10</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="sales_wh" model="account.report.line">
                <field name="name">Sales Withholding</field>
                <field name="sequence">18</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="sales_wh_line_2" model="account.report.line">
                        <field name="name">Sales Withholding 20% (2/10)</field>
                        <field name="sequence">19</field>
                        <field name="code">SAL_WH_2</field>
                        <field name="expression_ids">
                            <record id="sales_wh_expression_2_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_20_BASE_2_10</field>
                            </record>
                            <record id="sales_wh_expression_2_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_20_TAX_2_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="sales_wh_line_3" model="account.report.line">
                        <field name="name">Sales Withholding 20% (3/10)</field>
                        <field name="sequence">20</field>
                        <field name="code">SAL_WH_3</field>
                        <field name="expression_ids">
                            <record id="sales_wh_expression_3_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_20_BASE_3_10</field>
                            </record>
                            <record id="sales_wh_expression_3_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_20_TAX_3_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="sales_wh_line_4" model="account.report.line">
                        <field name="name">Sales Withholding 20% (4/10)</field>
                        <field name="sequence">21</field>
                        <field name="code">SAL_WH_4</field>
                        <field name="expression_ids">
                            <record id="sales_wh_expression_4_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_20_BASE_4_10</field>
                            </record>
                            <record id="sales_wh_expression_4_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_20_TAX_4_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="sales_wh_line_5" model="account.report.line">
                        <field name="name">Sales Withholding 20% (5/10)</field>
                        <field name="sequence">22</field>
                        <field name="code">SAL_WH_5</field>
                        <field name="expression_ids">
                            <record id="sales_wh_expression_5_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_20_BASE_5_10</field>
                            </record>
                            <record id="sales_wh_expression_5_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_20_TAX_5_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="sales_wh_line_7" model="account.report.line">
                        <field name="name">Sales Withholding 20% (7/10)</field>
                        <field name="sequence">23</field>
                        <field name="code">SAL_WH_7</field>
                        <field name="expression_ids">
                            <record id="sales_wh_expression_7_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_20_BASE_7_10</field>
                            </record>
                            <record id="sales_wh_expression_7_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_20_TAX_7_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="sales_wh_line_9" model="account.report.line">
                        <field name="name">Sales Withholding 20% (9/10)</field>
                        <field name="sequence">24</field>
                        <field name="code">SAL_WH_9</field>
                        <field name="expression_ids">
                            <record id="sales_wh_expression_9_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_20_BASE_9_10</field>
                            </record>
                            <record id="sales_wh_expression_9_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_20_TAX_9_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="sales_wh_line_10" model="account.report.line">
                        <field name="name">Sales Withholding 20% (10/10)</field>
                        <field name="sequence">25</field>
                        <field name="code">SAL_WH_10</field>
                        <field name="expression_ids">
                            <record id="sales_wh_expression_10_balance" model="account.report.expression">
                                <field name="label">base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_20_BASE_10_10</field>
                            </record>
                            <record id="sales_wh_expression_10_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">SAL_20_TAX_10_10</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="net_vat" model="account.report.line">
                <field name="name">Net VAT</field>
                <field name="sequence">26</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="net_vat_line_p" model="account.report.line">
                        <field name="name">Total VAT on Purchases</field>
                        <field name="sequence">27</field>
                        <field name="code">PUR_VAT_TOTAL</field>
                        <field name="expression_ids">
                            <record id="net_vat_expression_p_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">PUR_0.tax + PUR_1.tax + PUR_10.tax + PUR_20.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="net_vat_line_s" model="account.report.line">
                        <field name="name">Total VAT on Sales</field>
                        <field name="sequence">28</field>
                        <field name="code">SAL_VAT_TOTAL</field>
                        <field name="expression_ids">
                            <record id="net_vat_expression_s_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">SAL_1.tax + SAL_10.tax + SAL_20.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="net_vat_line_total" model="account.report.line">
                        <field name="name">Total Net VAT</field>
                        <field name="sequence">29</field>
                        <field name="code">TOTAL_NET_VAT</field>
                        <field name="expression_ids">
                            <record id="net_vat_expression_total" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">(SAL_1.tax + SAL_10.tax + SAL_20.tax) - 
                                                      (PUR_0.tax + PUR_1.tax + PUR_10.tax + PUR_20.tax)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="wh_vat" model="account.report.line">
                <field name="name">Withholding Tax Total</field>
                <field name="sequence">30</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="wh_vat_line_p" model="account.report.line">
                        <field name="name">Total VAT on Purchases Withheld</field>
                        <field name="sequence">31</field>
                        <field name="code">PUR_WH_VAT_TOTAL</field>
                        <field name="expression_ids">
                            <record id="wh_vat_expression_p_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">PUR_WH_2.tax + PUR_WH_3.tax + PUR_WH_4.tax + PUR_WH_5.tax 
                                                        + PUR_WH_7.tax + PUR_WH_9.tax + PUR_WH_10.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="wh_vat_line_s" model="account.report.line">
                        <field name="name">Total VAT on Sales Withheld</field>
                        <field name="sequence">32</field>
                        <field name="code">SAL_WH_VAT_TOTAL</field>
                        <field name="expression_ids">
                            <record id="wh_vat_expression_s_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">SAL_WH_2.tax + SAL_WH_3.tax + SAL_WH_4.tax + SAL_WH_5.tax 
                                                        + SAL_WH_7.tax + SAL_WH_9.tax + SAL_WH_10.tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="vat_due" model="account.report.line">
                <field name="name">VAT Due</field>
                <field name="sequence">33</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="vat_due_line_p" model="account.report.line">
                        <field name="name">Total Purchase Taxes Paid</field>
                        <field name="sequence">34</field>
                        <field name="code">PUR_VAT_PAID_TOTAL</field>
                        <field name="expression_ids">
                            <record id="vat_due_expression_p_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">PUR_VAT_TOTAL.tax - PUR_WH_VAT_TOTAL.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_due_line_s" model="account.report.line">
                        <field name="name">Total Sales Tax Collected</field>
                        <field name="sequence">35</field>
                        <field name="code">SAL_VAT_PAID_TOTAL</field>
                        <field name="expression_ids">
                            <record id="vat_due_expression_s_tax" model="account.report.expression">
                                <field name="label">tax</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">SAL_VAT_TOTAL.tax - SAL_WH_VAT_TOTAL.tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-tr.csv

```csv
"id","name","code","account_type","tag_ids","reconcile","name@tr"
"tr101","Cheques Received","101","asset_current","","False","ALINAN ÇEKLER"
"tr102997","Non-collected Payments","102997","asset_current","","True","TAHSIL EDILMEYEN ÖDEMELER"
"tr102998","Unconfirmed Payments","102998","asset_current","","True","TASDIK EDILMEMIŞ ÖDEMELER"
"tr102999","Bank Temporary Suspended Account","102999","asset_current","","False","BANKA GEÇICI MUALLAK HESABI"
"tr103","Cheques Given And Payment Orders (-)","103","asset_current","","True","VERİLEN ÇEKLER VE ÖDEME EMİRLERİ (-)"
"tr108","Other Liquid Assets","108","asset_current","","False","DİĞER HAZIR DEĞERLER"
"tr110","Common Stocks","110","asset_current","","False","HİSSE SENETLERİ"
"tr111","Private Sector Bonds, Shares And Notes","111","asset_current","","False",""
"tr112","Public Sector Bonds, Shares And Notes","112","asset_current","","False",""
"tr118","Other Marketable Securities","118","asset_current","","False","DİĞER MENKUL KIYMETLER"
"tr119","Provision For Diminution In Value Of Marketable Securities (-)","119","asset_current","","False","MENKUL KIYMETLER DEĞER DÜŞÜKLÜĞÜ KARŞILIĞI (-)"
"tr120","Customers","120","asset_receivable","","True","ALICILAR"
"tr121","Notes Receivable","121","asset_current","","False","ALACAK SENETLERİ"
"tr122","Discount Of Notes Receivable (-)","122","asset_current","","False","ALACAK SENETLERİ REESKONTU (-)"
"tr123","Buyers (POS)","123","asset_receivable","","True","ALICILAR (POS)"
"tr124","Unearned Lease Interest Income (-)","124","asset_current","","False","KAZANILMAMIŞ FİNANSAL KİRALAMA FAİZ GELİRLERİ (-)"
"tr126","Deposits And Guarantees Given","126","asset_current","","False","VERİLEN DEPOZİTO VE TEMİNATLAR"
"tr127","Other Trade Receivables","127","asset_current","","False","DİĞER TİCARİ ALACAKLAR"
"tr128","Doubtful Trade Receivables","128","asset_current","","False","ŞÜPHELİ TİCARİ ALACAKLAR"
"tr129","Provision For Doubtful Trade Receivables (-)","129","asset_current","","False","ŞÜPHELİ TİCARİ ALACAKLAR KARŞILIĞI (-)"
"tr131","Receivables From Shareholders","131","asset_current","","False","ORTAKLARDAN ALACAKLAR"
"tr132","Receivables From Subsidaries","132","asset_current","","False","İŞTİRAKLERDEN ALACAKLAR"
"tr133","Receivables From Affiliated Companies","133","asset_current","","False","BAĞLI ORTAKLIKLARDAN ALACAKLAR"
"tr135","Receivables From Personnel","135","asset_current","","False","PERSONELDEN ALACAKLAR"
"tr136","Other Receivable (-)","136","asset_current","","False","DİĞER ÇEŞİTLİ ALACAKLAR"
"tr137","Discount Of Other Notes Receivables","137","asset_current","","False","DİĞER ALACAK SENETLERİ REESKONTU (-)"
"tr138","Other Doubtful Receivables","138","asset_current","","False","ŞÜPHELİ DİĞER ALACAKLAR"
"tr139","Provision For Other Doubtful Receivables (-)","139","asset_current","","False","ŞÜPHELİ DİĞER ALACAKLAR KARŞILIĞI (-)"
"tr150","Raw Materials And Supplies","150","asset_current","","False","İLK MADDE VE MALZEME"
"tr151","Work-In-Process – Production","151","asset_current","","False","YARI MAMULLER - ÜRETİM"
"tr152","Finished Goods","152","asset_current","","False","MAMÜLLER"
"tr153","Commercial Goods","153","asset_current","","False","TİCARİ MALLAR"
"tr157","Other Stocks","157","asset_current","","False","DİĞER STOKLAR"
"tr158","Provision For Diminution In Value Of Stocks (-)","158","asset_current","","False","STOK DEĞER DÜŞÜKLÜĞÜ KARŞILIĞI (-)"
"tr159","Stock Advances Given","159","asset_current","","False","VERİLEN SİPARİŞ AVANSLARI"
"tr180","Prepaid Expenses For The Following Months","180","expense","","False","GELECEK AYLARA AİT GİDERLER"
"tr181","Income Accruals","181","expense","","False","GELİR TAHAKKUKLARI"
"tr190","Deferred Vat","190","expense","","False","DEVREDEN KATMA DEĞER VERGİSİ (KDV)"
"tr191","Deductible Vat","191","asset_current","","False","İNDİRİLECEK KATMA DEĞER VERGİSİ (KDV)"
"tr192","Other Vat","192","expense","","False","DİĞER KATMA DEĞER VERGİSİ"
"tr193","Prepaid Taxes And Funds","193","expense","","False","PEŞİN ÖDENEN VERGİLER VE FONLAR"
"tr195","Work Advances","195","expense","","False","İŞ AVANSLAR"
"tr196","Advances Given To Personnel","196","expense","","False","PERSONEL AVANSLARI"
"tr197","Stock Count And Delivery Shortages","197","expense","","False","SAYIM VE TESELLÜM NOKSANLARI"
"tr198","Other Current Assets","198","expense","","False","DİĞER DÖNEN VARLIKLAR"
"tr199","Provision For Other Current Assets (-)","199","expense","","False","DİĞER DÖNEN VARLIKLAR KARŞILIĞI (-)"
"tr220","Customers","220","asset_fixed","","False","ALICILAR"
"tr221","Notes Receivables","221","asset_fixed","","False",""
"tr222","Discount Of Notes Receivables (-)","222","asset_fixed","","False","ALACAK SENETLERİ REESKONTU (-)"
"tr224","Interest Income On Non-earned Financial Leases (-)","224","asset_fixed","","False","KAZANILMAMIŞ FINANSAL KIRALAMA FAIZ GELIRLERI(-)"
"tr226","Deposits And Guarantees Given","226","asset_fixed","","False","VERİLEN DEPOZİTO VE TEMİNATLAR"
"tr229","Provision For Doubtful Receivables (-)","229","asset_fixed","","False","ŞÜPHELİ ALACAKLAR KARŞILIĞI (-)"
"tr231","Receivables From Shareholders","231","asset_fixed","","False","ORTAKLARDAN ALACAKLAR"
"tr232","Receivables From Subsidiaries","232","asset_fixed","","False","İŞTİRAKLERDEN ALACAKLAR"
"tr233","Receivables From Affiliated Companies","233","asset_fixed","","False","BAĞLI ORTAKLIKLARDAN ALACAKLAR"
"tr235","Receivables From Personnel","235","asset_fixed","","False","PERSONELDEN ALACAKLAR"
"tr236","Other Miscellaneous Receivables","236","asset_fixed","","False","DİĞER ÇEŞİTLİ ALACAKLAR"
"tr237","Discount Of Other Notes Receivables (-)","237","asset_fixed","","False","DİĞER ALACAK SENETLERİ REESKONTU (-)"
"tr240","Long-Term Marketable Securities","240","asset_fixed","","False","BAĞLI MENKUL KIYMETLER"
"tr241","Provision For Diminution In Value Of Long-Term Securities (-)","241","asset_fixed","","False","BAĞLI MENKUL KIYMETLER DEĞER DÜŞÜKLÜĞÜ KARŞILIĞI (-)"
"tr242","Subsidiaries","242","asset_fixed","","False","İŞTİRAKLER"
"tr243","Capital Commitment For Subsidiaries (-)","243","asset_fixed","","False","İŞTİRAKLERE SERMAYE TAAHHÜTLERİ (-)"
"tr244","Provision For Diminution In Value Of Investments (-)","244","asset_fixed","","False","İŞTİRAKLER SERMAYE PAYLARI DEĞER DÜŞÜKLÜĞÜ KARŞILIĞI (-)"
"tr245","Affiliated Companies","245","asset_fixed","","False","BAĞLI ORTAKLIKLAR"
"tr246","Capital Commitment To Affiliated Companies","246","asset_fixed","","False","BAĞLI ORTAKLIKLARA SERMAYE TAAHHÜTLERİ (-)"
"tr247","Provision For Diminution In Value Of Affiliated Companies (-)","247","asset_fixed","","False","BAĞLI ORTAKLIKLAR SERMAYE PAYLARI DEĞER DÜŞÜKLÜGÜ KARŞILIĞI (-)"
"tr248","Other Non-Current Financial Assets","248","asset_fixed","","False","DİĞER MALİ DURAN VARLIKLAR"
"tr249","Provision For Other Non-Current Financial Assets (-)","249","asset_fixed","","False","DİĞER MALİ DURAN VARLIKLAR KARŞILIĞI (-)"
"tr250","Land","250","asset_fixed","","False","ARAZİ VE ARSALAR"
"tr251","Underground Installations","251","asset_fixed","","False","YER ALTI VE YER ÜSTÜ DÜZENLERİ"
"tr252","Buildings","252","asset_fixed","","False","BİNALAR"
"tr253","Machinery, Equipment And Installations","253","asset_fixed","","False","TESİS, MAKİNE VE CİHAZLAR"
"tr254","Motor Vehicles","254","asset_fixed","","False","TAŞITLAR"
"tr255","Furniture And Fixtures","255","asset_fixed","","False","DEMİRBAŞLAR"
"tr256","Other Tangible Assets","256","asset_fixed","","False","DİĞER MADDİ DURAN VARLIKLAR"
"tr257","Accumulated Depreciation","257","asset_fixed","","False","BİRİKMİŞ AMORTİSMANLAR (-)"
"tr258","Construction-In-Progress","258","asset_fixed","","False","YAPILMAKTA OLAN YATIRIMLAR"
"tr259","Fixed Asset Advances Given","259","asset_fixed","","False","VERİLEN AVANSLAR"
"tr260","Rights","260","asset_fixed","","False","HAKLAR"
"tr261","Goodwill","261","asset_fixed","","False","ŞEREFİYE"
"tr262","Pre-Operating Expenses","262","asset_fixed","","False","KURULUŞ VE ÖRGÜTLENME GİDERLERİ"
"tr263","Research And Development Expenses","263","asset_fixed","","False","ARAŞTIRMA VE GELİŞTİRME GİDERLERİ"
"tr264","Leasehold Improvements","264","asset_fixed","","False","ÖZEL MALİYETLER"
"tr267","Other Intangible Assets","267","asset_fixed","","False","DİĞER MADDİ OLMAYAN DURAN VARLIKLAR"
"tr268","Accumulated Depreciation (-)","268","asset_fixed","","False","BİRİKMİŞ AMORTİSMANLAR (-)"
"tr269","Advances Given","269","asset_fixed","","False","VERİLEN AVANSLAR"
"tr271","Research Expenses","271","expense","","False","ARAMA GİDERLERİ"
"tr272","Preparation And Development Expenses","272","expense","","False","HAZIRLIK VE GELİŞTİRME GİDERLERİ"
"tr277","Other Depletable Assets","277","expense","","False","DİĞER ÖZEL TÜKENMEYE TABİ VARLIKLAR"
"tr278","Accumulated Depletion (-)","278","expense","","False","BİRİKMİŞ TÜKENME PAYLARI (-)"
"tr279","Advances Given","279","expense","","False","VERİLEN AVANSLAR"
"tr280","Prepaid Expenses For Following Years","280","expense","","False","GELECEK YILLARA AİT GİDERLER"
"tr281","Income Accruals","281","expense","","False","GELİR TAHAKKUKLARI"
"tr291","Vat Deductible In The Following Years","291","asset_current","","False","GELECEK YILLARDA İNDİRİLECEK KATMA DEĞER VERGİSİ"
"tr292","Other Vat","292","asset_current","","False","DİĞER KATMA DEĞER VERGİSİ"
"tr293","Long-Term Stocks","293","asset_current","","False","GELECEK YILLAR İHTİYACI STOKLAR"
"tr294","Stocks And Tangible Assets To Be Disposed","294","asset_current","","False","ELDEN ÇIKARILACAK STOKLAR VE MADDİ DURAN VARLIKLAR"
"tr295","Prepaid Tax And Funds","295","asset_current","","False",""
"tr297","Other Non-Current Assets","297","asset_current","","False","DİĞER DURAN VARLIKLAR"
"tr298","Provision For Diminution In Value Of Stocks (-)","298","asset_current","","False","STOK DEĞER DÜŞÜKLÜĞÜ KARŞILIĞI (-)"
"tr299","Accumulated Depreciation (-)","299","asset_current","","False","BİRİKMİŞ AMORTİSMANLAR (-)"
"tr300","Bank Loans","300","liability_current","","False","BANKA KREDİLERİ"
"tr301","Payables from Financial Leasing Transactions","301","liability_current","","False","FINANSAL KIRALAMA İŞLEMLERINDEN BORÇLAR"
"tr302","Deferred Financial Leasing Borrowing Costs (-)","302","liability_current","","False","ERTELENMIŞ FINANSAL KIRALAMA BORÇLANMA MALIYETLERI(-)"
"tr303","Principal And Interest Payments Of Long-Term Loans","303","liability_current","","False","UZUN VADELİ KREDİLERİN ANAPARA TAKSİTLERİ VE FAİZLERİ"
"tr304","Principal, Instalment And Interest Payments Of Bonds","304","liability_current","","False","TAHVİL ANAPARA BORÇ, TAKSİT VE FAİZLERİ"
"tr305","Bonds And Shares Issued","305","liability_current","","False","ÇIKARILMIŞ BONOLAR VE SENETLER"
"tr306","Other Marketable Securities Issued","306","liability_current","","False","ÇIKARILMIŞ DİĞER MENKUL KIYMETLER"
"tr308","Premium Reserves Of Marketable Securities (-)","308","liability_current","","False","MENKUL KIYMETLER İHRAÇ FARKI (-)"
"tr309","Other Financial Liabilities","309","liability_current","","False","DİĞER MALİ BORÇLAR"
"tr320","Suppliers","320","liability_payable","","True","SATICILAR"
"tr321","Notes Payable","321","liability_current","","False","BORÇ SENETLERİ"
"tr322","Discount Of Notes Payable (-)","322","liability_current","","False","BORÇ SENETLERİ REESKONTU (-)"
"tr326","Deposits And Guarantees Given","326","liability_current","","False","VERİLEN DEPOZİTO VE TEMİNATLAR"
"tr329","Other Trade Payables","329","liability_current","","False","DİĞER TİCARİ BORÇLAR"
"tr331","Payables To Shareholders","331","liability_current","","False","ORTAKLARA BORÇLAR"
"tr332","Payables To Subsidiaries","332","liability_current","","False","İŞTİRAKLERE BORÇLAR"
"tr333","Payables To Affiliated Companies","333","liability_current","","False","BAĞLI ORTAKLIKLARA BORÇLAR"
"tr335","Payables To Personnel","335","liability_current","","False","PERSONELE BORÇLAR"
"tr336","Other Miscellaneous Payaples","336","liability_current","","False","DIĞER ÇEŞITLI BORÇLAR"
"tr337","Discount Of Other Notes Payables (-)","337","liability_current","","False","DİĞER BORÇ SENETLERİ REESKONTU (-)"
"tr340","Advances Taken For Orders","340","liability_current","","False","ALINAN SİPARİŞ AVANSLARI"
"tr349","Other Advances Taken","349","liability_current","","False","ALINAN DİĞER AVANSLAR"
"tr360","Taxes And Funds Payable","360","liability_current","","False","ÖDENECEK VERGİ VE FONLAR"
"tr361","Social Security Premiums Payable","361","liability_current","","False","ÖDENECEK SOSYAL GÜVENLİK KESİNTİLERİ"
"tr368","Overdue, Deferred Payables Or Payables On Instalments To The State","368","liability_current","","False","VADESİ GEÇMİŞ ERTELENMİŞ VEYA TAKSİTLENDİRİLMİŞ VERGİ VE DİĞER YÜKÜMLÜLÜKLER"
"tr369","Other Liabilities","369","liability_current","","False","ÖDENECEK DİĞER YÜKÜMLÜLÜKLER"
"tr370","Provisions For Tax And Other Liabilities Relating To The Profit Of The Period","370","liability_current","","False","DÖNEM KÂRI VERGİ VE DİĞER YASAL YÜKÜMLÜLÜK KARŞILIKLARI"
"tr371","Prepaid Tax And Other Liabilities For The Current Year Profit (-)","371","liability_current","","False","DÖNEM KÂRININ PEŞİN ÖDENEN VERGİ VE DİĞER YÜKÜMLÜLÜKLERİ (-)"
"tr372","Provision For Severance Payments","372","liability_current","","False","KIDEM TAZMİNATI KARŞILIĞI"
"tr373","Provision For Expenses Relating To Costing","373","expense","","False","MALİYET GİDERLERİ KARŞILIĞI"
"tr379","Provision For Other Liabilities And Expenses","379","liability_current","","False","DİĞER BORÇ VE GİDER KARŞILIKLARI"
"tr380","Deferred Income For The Following Months","380","liability_current","","False","GELECEK AYLARA AİT GELİRLER"
"tr381","Expense Accruals","381","liability_current","","False","GİDER TAHAKKUKLARI"
"tr391","Vat Calculated","391","liability_current","","False","HESAPLANAN KDV"
"tr392","Other Vat","392","liability_current","","False","DİĞER KATMA DEĞER VERGİSİ"
"tr393","Headquarters and Branches Current Account","393","liability_current","","False","MERKEZ VE ŞUBELER CARİ HESABI"
"tr397","Stock Count And Delivery Surpluses","397","liability_current","","False","SAYIM VE TESELLÜM FAZLALARI(1)"
"tr399","Other Miscellaneous Short-Term Liabilities","399","liability_current","","False","DİĞER ÇEŞİTLİ YABANCI KAYNAKLAR"
"tr400","Bank Loans","400","liability_current","","False","BANKA KREDİLERİ"
"tr401","Payables from Financial Leasing Transactions","401","liability_current","","False","FINANSAL KIRALAMA İŞLEMLERINDEN BORÇLAR"
"tr402","Deferred Leasing Borrowing Costs(-)","402","liability_current","","False","ERTELENMİŞ FİNANSAL KİRALAMA BORÇLANMA MALİYETLERİ (-)"
"tr405","Bonds Issued","405","liability_current","","False","ÇIKARILMIŞ TAHVİLLER"
"tr407","Other Marketable Securities Issued","407","liability_current","","False","ÇIKARILMIŞ DİĞER MENKUL KIYMETLER"
"tr408","Premium Reserves Of Marketable Securities (-)","408","liability_current","","False","MENKUL KIYMETLER İHRAÇ FARKI (-)"
"tr409","Other Financial Liabilities","409","liability_current","","False","DİĞER MALİ BORÇLAR"
"tr420","Suppliers","420","liability_current","","False","SATICILAR"
"tr421","Notes Payable","421","liability_current","","False","BORÇ SENETLERİ"
"tr422","Discount Of Notes Payable (-)","422","liability_current","","False","BORÇ SENETLERİ REESKONTU (-)"
"tr426","Deposits And Guarantees Taken","426","liability_current","","False","ALINAN DEPOZİTO VE TEMİNATLAR"
"tr429","Other Trade Payables","429","liability_current","","False","DİĞER TİCARİ BORÇLAR"
"tr431","Payables To Shareholders","431","liability_current","","False","ORTAKLARA BORÇLAR"
"tr432","Payables To Subsidaries","432","liability_current","","False","İŞTİRAKLERE BORÇLAR"
"tr433","Payables To Affiliated Companies","433","liability_current","","False","BAĞLI ORTAKLIKLARA BORÇLAR"
"tr436","Other Miscellaneous Payables","436","liability_current","","False","DİĞER ÇEŞİTLİ BORÇLAR(1)"
"tr437","Discount Of Other Notes Payable (-)","437","liability_current","","False","KAMUYA OLAN ERTELENMİŞ VEYA TAKSİTLENDİRİLMİŞ BORÇLAR"
"tr438","Liabilities To The State (Deferred Or Payable In Instalments)","438","liability_current","","False","DİĞER ÇEŞİTLİ BORÇLAR"
"tr440","Advances Taken For Orders","440","liability_current","","False","ALINAN SİPARİŞ AVANSLARI"
"tr449","Other Advances Taken","449","liability_current","","False","ALINAN DİĞER AVANSLAR"
"tr472","Provisions For Severance Payments","472","liability_current","","False","KIDEM TAZMİNATI KARŞILIĞI"
"tr479","Provisions For Other Liabilities And Expenses","479","liability_current","","False","DİĞER BORÇ VE GİDER KARŞILIKLARI"
"tr492","Vat Deferred Or Postponed To The Following Years","492","liability_current","","False","GELECEK YILLARA ERTELENEN VEYA TERKİN EDİLEN KATMA DEĞER VERGİSİ(1)"
"tr493","Participation In The Establishment","493","liability_current","","False","TESİSE KATILMA PAYLARI"
"tr499","Other Miscellaneous Long-Term Liabilities","499","liability_current","","False","DİĞER ÇEŞİTLİ UZUN VADELİ YABANCI KAYNAKLAR"
"tr500","Capital","500","equity","","False","SERMAYE"
"tr501","Unpaid Capital (-)","501","equity","","False","ÖDENMEMİŞ SERMAYE (-)"
"tr520","Premium Reserves","520","equity","","False","HİSSE SENETLERİ İHRAÇ PRİMLERİ"
"tr521","Profit From Invalidation Of Shares","521","equity","","False","HİSSE SENEDİ İPTAL KÂRLARI"
"tr522","Fixed Asset Revaluation Fund","522","equity","","False","M.D.V. YENİDEN DEĞERLEME ARTIŞLARI"
"tr524","Cost Increases Fund","524","equity","","False","MALİYET ARTIŞLARI FONU"
"tr529","Other Capital Reserves","529","equity","","False","DİĞER SERMAYE YEDEKLERİ"
"tr540","Legal Reserves","540","equity","","False",""
"tr541","Statuary Reserves","541","equity","","False","STATÜ YEDEKLERİ"
"tr542","General Reserves","542","equity","","False","OLAĞANÜSTÜ YEDEKLER"
"tr548","Other Retained Profits","548","equity","","False","DİĞER KÂR YEDEKLERİ"
"tr549","Special Reserves","549","equity","","False","ÖZEL FONLAR"
"tr570","Previous Years’ Profits","570","equity","","False","GEÇMİŞ YILLAR KÂRLARI"
"tr580","Previous Years’ Losses","580","equity","","False","GEÇMİŞ YILLAR ZARARLARI (-)"
"tr590","Net Profit For The Period","590","equity","","False","DÖNEM NET KÂRI"
"tr591","Net Loss For The Period (-)","591","equity","","False","DÖNEM NET ZARARI (-)"
"tr600","Domestic Sales","600","income","","False","YURTİÇİ SATIŞLAR"
"tr601","Export Sales","601","income","","False","YURTDIŞI SATIŞLAR"
"tr602","Other Sales","602","income_other","","False","DİĞER GELİRLER"
"tr610","Returns From Sales (-)","610","income","","False","SATIŞTAN İADELER (-)"
"tr611","Sales Discounts (-)","611","income","","False","SATIŞ İSKONTOLARI (-)"
"tr612","Other Discounts (-)","612","income","","False",""
"tr620","Cost Of Finished Goods Sold (-)","620","expense_direct_cost","","False","SATILAN MAMÜLLER MALİYETİ (-)"
"tr621","Cost Of Commercial Goods Sold (-)","621","expense_direct_cost","","False","SATILAN TİCARİ MALLAR MALİYETİ (-)"
"tr622","Cost Of Services Sold (-)","622","expense_direct_cost","","False","SATILAN HİZMET MALİYETİ (-)"
"tr623","Cost Of Other Sales (-)","623","expense_direct_cost","","False","DİĞER SATIŞLARIN MALİYETİ (-)"
"tr630","Research And Development Expenses (-)","630","expense","","False","ARAŞTIRMA VE GELİŞTİRME GİDERLERİ (-)"
"tr631","Marketing, Sales And Distributing Expenses (-)","631","expense","","False","PAZARLAMA SATIŞ VE DAĞITIM GİDERLERİ"
"tr632","General And Administrative Expenses (-)","632","expense","","False","GENEL YÖNETİM GİDERLERİ (-)"
"tr640","Dividend Income From Subsidiaries","640","income","","False","İŞTİRAKLERDEN TEMETTÜ GELİRLERİ"
"tr641","Dividend Income From Affiliated Companies","641","income","","False","BAĞLI ORTAKLIKLARDAN TEMETTÜ GELİRLERİ"
"tr642","Interest Income","642","income","","False","FAİZ GELİRLERİ"
"tr643","Commission Income","643","income","","False","KOMİSYON GELİRLERİ"
"tr644","Provision No Longer Required","644","income","","False","KONUSU KALMAYAN KARŞILIKLAR"
"tr645","Marketable Securities Sales Profit","645","income","","False","MENKUL KIYMET SATIŞ KARLARI"
"tr646","Foreign Exchange Gains","646","income","","False","KAMBIYO KARLARI"
"tr647","Discount Interest Income","647","income","","False","REESKONT FAIZ GELIRLERI"
"tr648","Gains From Inflation Adjustments","648","income","","False","ENFLASYON DÜZELTMESI KARLARI"
"tr649","Other Income And Profit From Operations","649","income","","False","FAALİYETLE İLGİLİ DİĞER GELİR VE KÂRLAR"
"tr653","Commission Expenses (-)","653","expense","","False","KOMİSYON GİDERLERİ (-)"
"tr654","Provisions (-)","654","expense","","False","KARŞILIK GİDERLERİ (-)"
"tr655","Marketable Securities Sales Losses (-)","655","expense","","False","MENKUL KIYMET SATIŞ ZARARLARI (-)"
"tr656","Foreign Exchange Losses (-)","656","expense","","False","KAMBIYO ZARARLARI (-)"
"tr657","Interest Expense On Discounted Notes (-)","657","expense","","False","REESKONT FAIZ GIDERLERI (-)"
"tr658","Loss From Inflation Adjustments (-)","658","expense","","False","ENFLASYON DÜZELTMESI ZARARLARI(-)"
"tr659","Other Expenses And Losses (-)","659","expense","","False","DİĞER GİDER VE ZARARLAR (-)"
"tr660","Short-Term Borrowing Expenses (-)","660","expense","","False","KISA VADELİ BORÇLANMA GİDERLERİ (-)"
"tr661","Long-Term Borrowing Expenses (-)","661","expense","","False","UZUN VADELİ BORÇLANMA GİDERLERİ (-)"
"tr671","Income And Profit Relating To Previous Periods","671","income","","False",""
"tr679","Other Extraordinary Income And Profit","679","income","","False","DİĞER OLAĞANDIŞI GELİR VE KÂRLAR"
"tr680","Non-Operating Department Expense And Loss (-)","680","expense","","False","ÇALIŞMAYAN KISIM GİDER VE ZARARLARI (-)"
"tr681","Expense And Loss Relating To Previous Periods (-)","681","expense","","False","ÖNCEKİ DÖNEM GİDER VE ZARARLARI (-)"
"tr689","Other Extraordinary Expense And Loss (-)","689","expense","","False","DİĞER OLAĞANDIŞI GİDER VE ZARARLAR (-)"
"tr690","Net Profit (Loss) For The Period","690","expense","","False","DÖNEM NET KÂRI (ZARARI)"
"tr691","Provisions For Taxation And Other Legal Liabilities (-)","691","expense","","False","DÖNEM KÂRI VERGİ VE DİĞER YASAL YÜKÜMLÜLÜK KARŞILIKLARI (-)"
"tr692","Net Profit (Loss) For The Period","692","expense","","False","DÖNEM NET KÂRI (ZARARI)"
"tr697","Long-term Construction Inflation Adjustment Account","697","expense","","False","YILLARA YAYGIN İNŞAAT VE ENFLASYON DÜZELTME"
"tr698","Inflation Adjustment Account","698","expense","","False","ENFLASYON DÜZELTME HESABI"
"tr700","Transitory Accounts For Accounting","700","expense","","False","MALİYET MUHASEBESİ BAĞLANTI HESABI"
"tr701","Reflection Accounts For Cost Accounting","701","expense","","False","MALİYET MUHASEBESİ YANSITMA HESABI"
"tr710","Direct Raw Materials And Supplies Expenses","710","expense","","False","DİREKT İLKMADDE VE MALZEME GİDERLERİ"
"tr711","Reflection Account For Direct Raw Materials And Supplies","711","expense","","False","DİREKT İLKMADDE VE MALZEME YANSITMA HESABI"
"tr712","Price Differences Of Direct Raw Materials And Supplies","712","expense","","False","DİREKT İLKMADDE VE MALZEME FİYAT FARKI"
"tr713","Quantity Differences Of Direct Raw Materials And Supplies","713","expense","","False","DİREKT İLKMADDE VE MALZEME MİKTAR FARKI"
"tr720","Direct Labour Expenses","720","expense","","False","DİREKT İŞÇİLİK GİDERLERİ"
"tr721","Reflection Account For Direct Labour Expenses","721","expense","","False","DİREKT İŞCİLİK GİDERLERİ YANSITMA HESABI"
"tr722","Direct Labour Wage Differences","722","expense","","False","DİREKT İŞÇİLİK ÜCRET FARKLARI"
"tr723","Direct Labour Time Differences","723","expense","","False","DİREKT İŞÇİLİK SÜRE (ZAMAN) FARKLARI"
"tr730","General Production Expenses","730","expense","","False","GENEL ÜRETİM GİDERLERİ"
"tr731","Reflection Account For General Production Expenses","731","expense","","False","GENEL ÜRETİM GİDERLERİ YANSITMA HESABI"
"tr732","Budget Differences Of General Production Expenses","732","expense","","False","GENEL ÜRETİM GİDERLERİ BÜTÇE FARKLARI"
"tr733","Productivity Differences Of General Production Expenses","733","expense","","False","GENEL ÜRETİM GİDERLERİ VERİMLİLİK FARKLARI"
"tr734","Capacity Differences Of General Production Expenses","734","expense","","False","GENEL ÜRETİM GİDERLERİ KAPASİTE FARKLARI"
"tr740","Cost Of Production Of Services","740","expense","","False","HİZMET ÜRETİM MALİYETİ"
"tr741","Reflection Account For Cost Of Production Of Services","741","expense","","False","HİZMET ÜRETİM MALİYETİ YANSITMA HESABI"
"tr742","Cost Of Production Of Services Difference Account","742","expense","","False","HİZMET ÜRETİM MALİYETİ FARK HESAPLARI"
"tr750","Research And Development Expenses","750","expense","","False","ARAŞTIRMA VE GELİŞTİRME GİDERLERİ"
"tr751","Reflection Account For Research And Development Expenses","751","expense","","False","ARAŞTIRMA VE GELİŞTİRME GİDERLERİ YANSITMA HESABI"
"tr752","Research And Development Expenses Difference Account","752","expense","","False","ARAŞTIRMA VE GELİŞTİRME GİDER FARKLARI"
"tr760","Marketing, Sales And Distribution Expenses","760","expense","","False","PAZARLAMA SATIŞ YE DAĞITIM GİDERLERİ"
"tr761","Reflection Account For Marketing, Sales And Distribution Expenses","761","expense","","False","PAZARLAMA SATIŞ VE DAĞITIM GİDERLERİ YANSITMA HESABI"
"tr762","Marketing, Sales And Distribution Expenses Difference Account","762","expense","","False","PAZARLAMA SATIŞ VE DAĞITIM GİDERLERİ FARK HESABI"
"tr770","General Administrative Expenses","770","expense","","False","GENEL YÖNETİM GİDERLERİ"
"tr771","Reflection Account For General Administrative Expenses","771","expense","","False","GENEL YÖNETİM GİDERLERİ YANSITMA HESABI"
"tr772","General Administrative Expenses Difference Account","772","expense","","False",""
"tr780","Financial Expenses","780","expense","","False","FİNANSMAN GİDERLERİ"
"tr781","Reflection Account For Financial Expenses","781","expense","","False","FİNANSMAN GİDERLERİ YANSITMA HESABI"
"tr782","Financial Expenses Difference Account","782","expense","","False","FİNANSMAN GİDERLERİ FARK HESABI"
"tr790","Direct Raw Materials And Supplies","790","expense","","False","DİREKT İLKMADDE VE MALZEME GİDERLERİ"
"tr791","Wages And Expenses Of Workers","791","expense","","False","İŞÇİ ÜCRET VE GİDERLERİ"
"tr792","Salaries And Expenses Of Personnel","792","expense","","False","MEMUR ÜCRET VE GİDERLERİ"
"tr793","External Utilities And Services Obtained","793","expense","","False","DIŞARIDAN SAĞLANAN FAYDA VE HİZMETLER"
"tr794","Miscellaneous Expenses","794","expense","","False","ÇEŞİTLİ GİDERLER"
"tr795","Taxes, Duties And Fees","795","expense","","False","VERGİ, RESİM VE HARÇLAR"
"tr796","Depreciation And Depletion Expenses","796","expense","","False","AMORTİSMANLAR VE TÜKENME PAYLARI"
"tr797","Financial Expenses","797","expense","","False","FİNANSMAN GİDERLERİ"
"tr798","Reflection Account For Expenses","798","expense","","False","GİDER ÇEŞİTLERİ YANSITMA HESABI"
"tr799","Cost Of Production","799","expense","","False","ÜRETİM MALİYET HESABI"
"tr391001","20% VAT","391001","liability_current","","False","20% KDV"
"tr391002","10% VAT","391002","liability_current","","False","10% KDV"
"tr391003","1% VAT","391003","liability_current","","False","1% KDV"
"tr391004","Sales WH","391004","liability_current","","False","SATIŞ NOKTASI"
"tr391005","Purchases WH","391005","liability_current","","False","SATIN ALIMLAR NE ZAMAN"

```

## File: data\template\account.group-tr.csv

```csv
"id","parent_id","code_prefix_start","code_prefix_end","name","name@tr"
"tr_group_1","","1","1","Current Assets","DÖNEN VARLIKLAR"
"tr_group_10","tr_group_1","10","10","Liquid Assests","HAZIR DEĞERLER"
"tr_group_11","tr_group_1","11","11","Marketable Securities","MENKUL KIYMETLER"
"tr_group_12","tr_group_1","12","12","Trade Receivables","TİCARİ ALACAKLAR"
"tr_group_13","tr_group_1","13","13","Other Receivables","DİĞER ALACAKLAR"
"tr_group_15","tr_group_1","15","15","Stocks","STOKLAR"
"tr_group_18","tr_group_1","18","18","Prepaid Expenses And Income Accruals For The Following Months","GELECEK AYLARA AİT GİDERLER VE GELİR TAHAKKUKLARI"
"tr_group_19","tr_group_1","19","19","Other Current Assets","DİĞER DÖNEN VARLIKLAR"
"tr_group_2","","2","2","Non-Current Assets","DURAN VARLIKLAR"
"tr_group_22","tr_group_2","22","22","Trade Receivables","TİCARİ ALACAKLAR"
"tr_group_23","tr_group_2","23","23","Other Receivables","DİĞER ALACAKLAR"
"tr_group_24","tr_group_2","24","24","Financial Non-Current Assets","MALİ DURAN VARLIKLAR"
"tr_group_25","tr_group_2","25","25","Tangible Non-Current Assets","MADDİ DURAN VARLIKLAR"
"tr_group_26","tr_group_2","26","26","Intangible Non-Current Assets","MADDİ OLMAYAN DURAN VARLIKLAR"
"tr_group_27","tr_group_2","27","27","Assets Subject To Depletion","ÖZEL TÜKENMEYE TABİ VARLIKLAR"
"tr_group_28","tr_group_2","28","28","Prepaid Expenses And Income Accruals For The Following Years","GELECEK YILLARA AİT GİDERLER VE GELİR TAHAKKUKLARI"
"tr_group_29","tr_group_2","29","29","Other Non-Current Assets","DİĞER DURAN VARLIKLAR"
"tr_group_3","","3","3","Short-Term Liabilities","KISA VADELİ YABANCI KAYNAKLAR"
"tr_group_30","tr_group_3","30","30","Financial Liabilities","MALİ BORÇLAR"
"tr_group_32","tr_group_3","32","32","Trade Payables","TİCARİ BORÇLAR"
"tr_group_33","tr_group_3","33","33","Other Payables","DİĞER BORÇLAR"
"tr_group_34","tr_group_3","34","34","Advances Taken","ALINAN AVANSLAR"
"tr_group_36","tr_group_3","36","36","Taxes Payable And Other Liabilities","ÖDENECEK VERGİ VE DİĞER YÜKÜMLÜLÜKLER"
"tr_group_37","tr_group_3","37","37","Provisions For Liabilities And Expenses","BORÇ VE GİDER KARŞILIKLARI"
"tr_group_38","tr_group_3","38","38","Deferred Income And Expenses Accruals For The Following Months","GELECEK AYLARA AİT GELİRLER VE GİDER TAHAKKUKLARI"
"tr_group_39","tr_group_3","39","39","Other Short-Term Liabilities","DİĞER KISA VADELİ YABANCI KAYNAKLAR"
"tr_group_4","","4","4","Long-Term Liabilities","UZUN VADELİ YABANCI KAYNAKLAR"
"tr_group_40","tr_group_4","40","40","Financial Liabilities","MALİ BORÇLAR"
"tr_group_42","tr_group_4","42","42","Trade Payables","TİCARİ BORÇLAR"
"tr_group_43","tr_group_4","43","43","Other Payables","DİĞER BORÇLAR"
"tr_group_44","tr_group_4","44","44","Advances Taken","ALINAN AVANSLAR"
"tr_group_47","tr_group_4","47","47","Provisions For Liabilities And Expenses","BORÇ VE GİDER KARŞILIKLARI"
"tr_group_48","tr_group_4","48","48","Deferred Income And Expenses Accruals For The Following Years","GELECEK YILLARA AİT GELİRLER VE GİDER TAHAKKUKLARI"
"tr_group_49","tr_group_4","49","49","Other Long-Term Liabilities","DİĞER UZUN VADELİ YABANCI KAYNAKLAR"
"tr_group_5","","5","5","Shareholders’ Equity","ÖZ KAYNAKLAR"
"tr_group_50","tr_group_5","50","50","Paid-Up Share Capital","ÖDENMİŞ SERMAYE"
"tr_group_52","tr_group_5","52","52","Capital Reserves","SERMAYE YEDEKLERİ"
"tr_group_54","tr_group_5","54","54","Retained Earnings",""
"tr_group_57","tr_group_5","57","57","Previous Years’ Profits","GEÇMİŞ YILLAR KÂRLARI"
"tr_group_58","tr_group_5","58","58","Previous Years’ Losses","GEÇMİŞ YILLAR ZARARLARI (-)"
"tr_group_59","tr_group_5","59","59","Profit (Loss) For The Period","DÖNEM NET KÂRI (ZARARI)"
"tr_group_6","","6","6","Income Statement","GELİR TABLOSU HESAPLARI"
"tr_group_60","tr_group_6","60","60","Gross Sales","BRÜT SATIŞLAR"
"tr_group_61","tr_group_6","61","61","Sales Discount (-)","SATIŞ İNDİRİMLERİ (-)"
"tr_group_62","tr_group_6","62","62","Cost Of Sales (-)","SATIŞLARIN MALİYETİ (-)"
"tr_group_63","tr_group_6","63","63","Operating Expenses (-)","FAALİYET GİDERLERİ (-)"
"tr_group_64","tr_group_6","64","64","Income And Profit From Other Operations","DİĞER FAALİYETLERDEN OLAĞAN GELİR VE KÂRLAR"
"tr_group_65","tr_group_6","65","65","Expense And Loss From Other Operations (-)","DİĞER FAALİYETLERDEN OLAĞAN GİDER VE ZARARLAR (-)"
"tr_group_66","tr_group_6","66","66","Financial Expenses (-)","FİNANSMAN GİDERLERİ (-)"
"tr_group_67","tr_group_6","67","67","Extraordinary Income And Profit","OLAĞANDIŞI GELİR VE KÂRLAR"
"tr_group_68","tr_group_6","68","68","Extraordinary Expense And Loss (-)","OLAĞANDIŞI GİDER VE ZARARLAR (-)"
"tr_group_69","tr_group_6","69","69","Net Profit (Loss) For The Period","DÖNEM NET KÂRI (ZARARI)"
"tr_group_7","","7","7","Cost Accounting (Alternative 7/A)","MALİYET HESAPLARI (7/A SEÇENEĞİ)"
"tr_group_70","tr_group_7","70","70","Transitory Accounts For Cost Accounting","MALİYET MUHASEBESİ BAĞLANTI HESAPLARI"
"tr_group_71","tr_group_7","71","71","Direct Raw Materials And Supplies","DİREKT İLKMADDE VE MALZEME GİDERLERİ"
"tr_group_72","tr_group_7","72","72","Direct Labour Expenses","DİREKT İŞÇİLİK GİDERLERİ"
"tr_group_73","tr_group_7","73","73","General Production Expenses","GENEL ÜRETİM GİDERLERİ"
"tr_group_74","tr_group_7","74","74","Cost Of Production Of Services","HİZMET ÜRETİM MALİYETİ"
"tr_group_75","tr_group_7","75","75","Research And Development Expenses","ARAŞTIRMA VE GELİŞTİRME GİDERLERİ"
"tr_group_76","tr_group_7","76","76","Marketing, Sales And Distribution Expenses","PAZARLAMA SATIŞ YE DAĞITIM GİDERLERİ"
"tr_group_77","tr_group_7","77","77","General Administrative Expenses","GENEL YÖNETİM GİDERLERİ"
"tr_group_78","tr_group_7","78","78","Financial Expenses","FİNANSMAN GİDERLERİ"
"tr_group_79","tr_group_7","79","79","Expense Types (Alternative 7/B)","GİDER ÇEŞİTLERİ (7/B SEÇENEĞİ)"
"tr_group_9","","9","9","Contingencies And Commitments",""

```

## File: data\template\account.tax-tr.csv

```csv
"id","sequence","description","invoice_label","name","price_include","amount","amount_type","children_tax_ids","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","description@tr"
"tr_s_0_ex","1","Zero-rated exports","0%","0% EX","False","0","percent","","sale","tr_tax_group_kdv_0","base","invoice","","+SAL_BASE_0","0% EX"
"","","","","","","","","","","","tax","invoice","tr391","",""
"","","","","","","","","","","","base","refund","","-SAL_BASE_0",""
"","","","","","","","","","","","tax","refund","tr391","",""
"tr_s_wh_20_2_10","2","Sales VAT with Withholding Tax 20% (2/10)","","WH 20% (2/10)","False","","group","tr_wh_20_2_10,tr_s_20","sale","tr_tax_group_kdv_20","","","","","Stopajlı Satış KDV'si %20 (2/10)"
"tr_wh_20_5_10","4","Withheld VAT 20% (Withholding 5/10)","-10%","WH 20% (5/10)","False","-10","percent","","none","tr_tax_group_wh_10","base","invoice","","+SAL_20_BASE_5_10","Tevkif KDV %20 (Tevkifat 5/10)"
"","","","","","","","","","","","tax","invoice","tr391004","-SAL_20_TAX_5_10",""
"","","","","","","","","","","","base","refund","","-SAL_20_BASE_5_10",""
"","","","","","","","","","","","tax","refund","tr391004","+SAL_20_TAX_5_10",""
"tr_s_wh_20_5_10","5","Sales VAT with Withholding Tax 20% (5/10)","","WH 20% (5/10)","False","","group","tr_wh_20_5_10,tr_s_20","sale","tr_tax_group_kdv_20","","","","","Stopajlı Satış KDV'si %20 (5/10)"
"tr_wh_s_20","6","Withholding Sales Tax 20%","-20%","WH 20%","False","-20","percent","","sale","tr_tax_group_wh_20","base","invoice","","","İhracat Kayıtlı Satış KDV'si (-20%)"
"","","","","","","","","","","","tax","invoice","tr391001","",""
"","","","","","","","","","","","base","refund","","-SAL_BASE_20",""
"","","","","","","","","","","","tax","refund","tr391001","-SAL_TAX_20",""
"tr_p_0","7","Purchase 0%","0%","0%","False","0","percent","","purchase","tr_tax_group_kdv_0","base","invoice","","+PUR_BASE_0","%0 satın al"
"","","","","","","","","","","","tax","invoice","tr191","+PUR_TAX_0",""
"","","","","","","","","","","","base","refund","","-PUR_BASE_0",""
"","","","","","","","","","","","tax","refund","tr191","-PUR_TAX_0",""
"tr_wh_20_7_10","8","Withheld VAT 20% (Withholding 7/10)","-14%","WH 20% (7/10)","False","-14","percent","","none","tr_tax_group_wh_14","base","invoice","","+SAL_20_BASE_7_10","Tevkif KDV %20 (Tevkif 7/10)"
"","","","","","","","","","","","tax","invoice","tr391004","-SAL_20_TAX_7_10",""
"","","","","","","","","","","","base","refund","","-SAL_20_BASE_7_10",""
"","","","","","","","","","","","tax","refund","tr391004","+SAL_20_TAX_7_10",""
"tr_s_wh_20_7_10","9","Sales VAT with Withholding 20% (7/10)","","WH 20% (7/10)","False","","group","tr_wh_20_7_10,tr_s_20","sale","tr_tax_group_kdv_20","","","","","Stopajlı Satış KDV'si %20 (7/10)"
"tr_wh_20_9_10","10","Withheld VAT 20% (Withholding 9/10)","-18%","WH 20% (9/10)","False","-18","percent","","none","tr_tax_group_wh_18","base","invoice","","+SAL_20_BASE_9_10","Tevkif KDV %20 (Tevkif 9/10)"
"","","","","","","","","","","","tax","invoice","tr391004","-SAL_20_TAX_9_10",""
"","","","","","","","","","","","base","refund","","-SAL_20_BASE_9_10",""
"","","","","","","","","","","","tax","refund","tr391004","+SAL_20_TAX_9_10",""
"tr_wh_20_10_10","11","Withheld VAT 20% (Withholding 10/10)","-20%","WH 20% (10/10)","False","-20","percent","","none","tr_tax_group_wh_20","base","invoice","","+SAL_20_BASE_10_10","Tevkif KDV %20 (Tevkif 10/10)"
"","","","","","","","","","","","tax","invoice","tr391004","-SAL_20_TAX_10_10",""
"","","","","","","","","","","","base","refund","","-SAL_20_BASE_10_10",""
"","","","","","","","","","","","tax","refund","tr391004","+SAL_20_TAX_10_10",""
"tr_s_wh_20_10_10","12","Sales VAT with Withholding Tax 20% (10/10)","","WH 20% (10/10)","False","","group","tr_wh_20_10_10,tr_s_20","sale","tr_tax_group_kdv_20","","","","","Stopajlı Satış KDV'si %20 (10/10)"
"tr_s_wh_20_9_10","13","Sales VAT with Withholding Tax 20% (9/10)","","WH 20% (9/10)","False","","group","tr_wh_20_9_10,tr_s_20","sale","tr_tax_group_kdv_20","","","","","Stopajlı Satış KDV'si %20 (9/10)"
"tr_wh_20_3_10","14","Withheld VAT 20% (Withholding 3/10)","-6%","WH 20% (3/10)","False","-6","percent","","none","tr_tax_group_wh_6","base","invoice","","+SAL_20_BASE_3_10","Tevkif KDV %20 (Tevkif 3/10)"
"","","","","","","","","","","","tax","invoice","tr391004","-SAL_20_TAX_3_10",""
"","","","","","","","","","","","base","refund","","-SAL_20_BASE_3_10",""
"","","","","","","","","","","","tax","refund","tr391004","+SAL_20_TAX_3_10",""
"tr_s_wh_20_3_10","15","Sales VAT with Withholding Tax 20% (3/10)","","WH 20% (3/10)","False","","group","tr_wh_20_3_10,tr_s_20","sale","tr_tax_group_kdv_20","","","","","Stopajlı Satış KDV'si %20 (3/10)"
"tr_wh_20_4_10","16","Withheld VAT 20% (Withholding 4/10)","-8%","WH 20% (4/10)","False","-8","percent","","none","tr_tax_group_wh_8","base","invoice","","+SAL_20_BASE_4_10","Stopajlı KDV %20 (Mutlak 4/10)"
"","","","","","","","","","","","tax","invoice","tr391004","-SAL_20_TAX_4_10",""
"","","","","","","","","","","","base","refund","","-SAL_20_BASE_4_10",""
"","","","","","","","","","","","tax","refund","tr391004","+SAL_20_TAX_4_10",""
"tr_s_wh_20_4_10","17","Sales VAT with Withholding Tax 20% (4/10)","","WH 20% (4/10)","False","","group","tr_wh_20_4_10,tr_s_20","sale","tr_tax_group_kdv_20","","","","","Stopajlı Satış KDV'si %20 (4/10)"
"tr_p_wh_20_2_10","18","Purchasing Withheld VAT 20% (2/10)","-4%","WH 20% (2/10)","False","-4","percent","","none","tr_tax_group_wh_4","base","invoice","","+PUR_20_BASE_2_10","Satın Almada Kesilen KDV %20 (2/10)"
"","","","","","","","","","","","tax","invoice","tr391005","-PUR_20_TAX_2_10",""
"","","","","","","","","","","","base","refund","","-PUR_20_BASE_2_10",""
"","","","","","","","","","","","tax","refund","tr391005","+PUR_20_TAX_2_10",""
"tr_p_wh_20_5_10","19","Purchasing Withheld VAT 20% (5/10)","-10%","WH 20% (5/10)","False","-10","percent","","none","tr_tax_group_wh_10","base","invoice","","+PUR_20_BASE_5_10","Satın Almada Kesilen KDV %20 (5/10)"
"","","","","","","","","","","","tax","invoice","tr391005","-PUR_20_TAX_5_10",""
"","","","","","","","","","","","base","refund","","-PUR_20_BASE_5_10",""
"","","","","","","","","","","","tax","refund","tr391005","+PUR_20_TAX_5_10",""
"tr_p_wh_20_7_10","20","Purchasing Withheld VAT 20% (7/10)","-14%","WH 20% (7/10)","False","-14","percent","","none","tr_tax_group_wh_14","base","invoice","","+PUR_20_BASE_7_10","Satın Almada Kesilen KDV %20 (7/10)"
"","","","","","","","","","","","tax","invoice","tr391005","-PUR_20_TAX_7_10",""
"","","","","","","","","","","","base","refund","","-PUR_20_BASE_7_10",""
"","","","","","","","","","","","tax","refund","tr391005","+PUR_20_TAX_7_10",""
"tr_p_wh_20_9_10","21","Purchasing Withheld VAT 20% (9/10)","-18%","WH 20% (9/10)","False","-18","percent","","none","tr_tax_group_wh_18","base","invoice","","+PUR_20_BASE_9_10","Satın Almada Kesilen KDV %20 (9/10)"
"","","","","","","","","","","","tax","invoice","tr391005","-PUR_20_TAX_9_10",""
"","","","","","","","","","","","base","refund","","-PUR_20_BASE_9_10",""
"","","","","","","","","","","","tax","refund","tr391005","+PUR_20_TAX_9_10",""
"tr_p_wh_20_10_10","22","Purchasing Withheld VAT 20% (10/10)","-20%","WH 20% (10/10)","False","-20","percent","","none","tr_tax_group_wh_20","base","invoice","","+PUR_20_BASE_10_10","Satın Almada Kesilen KDV %20 (10/10)"
"","","","","","","","","","","","tax","invoice","tr391005","-PUR_20_TAX_10_10",""
"","","","","","","","","","","","base","refund","","-PUR_20_BASE_10_10",""
"","","","","","","","","","","","tax","refund","tr391005","+PUR_20_TAX_10_10",""
"tr_p_vat_wh_20_2_10","23","Purchase VAT with Withholding Tax 20% (2/10)","","WH 20% (2/10)","False","","group","tr_p_wh_20_2_10,tr_p_20","purchase","tr_tax_group_kdv_20","","","","","Stopajlı Satın Alma KDV'si %20 (2/10)"
"tr_p_vat_wh_20_5_10","24","Purchase VAT with Withholding Tax 20% (5/10)","","WH 20% (5/10)","False","","group","tr_p_wh_20_5_10,tr_p_20","purchase","tr_tax_group_kdv_20","","","","","Stopajlı Satın Alma KDV'si %20 (5/10)"
"tr_p_vat_wh_20_7_10","25","Purchase VAT with Withholding Tax 20% (7/10)","","WH 20% (7/10)","False","","group","tr_p_wh_20_7_10,tr_p_20","purchase","tr_tax_group_kdv_20","","","","","Stopajlı Satın Alma KDV'si %20 (7/10)"
"tr_p_vat_wh_20_9_10","26","Purchase VAT with Withholding Tax 20% (9/10)","","WH 20% (9/10)","False","","group","tr_p_wh_20_9_10,tr_p_20","purchase","tr_tax_group_kdv_20","","","","","Stopajlı Satın Alma KDV'si %20 (9/10)"
"tr_p_vat_wh_20_10_10","27","Purchase VAT with Withholding Tax 20% (10/10)","","WH 20% (10/10)","False","","group","tr_p_wh_20_10_10,tr_p_20","purchase","tr_tax_group_kdv_20","","","","","Stopajlı Satın Alma KDV'si %20 (10/10)"
"tr_p_wh_20_3_10","28","Purchasing Withheld VAT 20% (3/10)","-6%","WH 20% (3/10)","False","-6","percent","","none","tr_tax_group_wh_6","base","invoice","","+PUR_20_BASE_3_10","Satın Almada Kesilen KDV %20 (3/10)"
"","","","","","","","","","","","tax","invoice","tr391005","-PUR_20_TAX_3_10",""
"","","","","","","","","","","","base","refund","","-PUR_20_BASE_3_10",""
"","","","","","","","","","","","tax","refund","tr391005","+PUR_20_TAX_3_10",""
"tr_p_wh_20_4_10","29","Purchasing Withheld VAT 20% (4/10)","-8%","WH 20% (4/10)","False","-8","percent","","none","tr_tax_group_wh_8","base","invoice","","+PUR_20_BASE_4_10","Satın Almada Kesilen KDV %20 (4/10)"
"","","","","","","","","","","","tax","invoice","tr391005","-PUR_20_TAX_4_10",""
"","","","","","","","","","","","base","refund","","-PUR_20_BASE_4_10",""
"","","","","","","","","","","","tax","refund","tr391005","+PUR_20_TAX_4_10",""
"tr_p_vat_wh_20_3_10","30","Purchase VAT with Withholding Tax 20% (3/10)","","WH 20% (3/10)","False","","group","tr_p_wh_20_3_10,tr_p_20","purchase","tr_tax_group_kdv_20","","","","","Stopajlı Satın Alma KDV'si %20 (3/10)"
"tr_p_vat_wh_20_4_10","31","Purchase VAT with Withholding Tax 20% (4/10)","","WH 20% (4/10)","False","","group","tr_p_wh_20_4_10,tr_p_20","purchase","tr_tax_group_kdv_20","","","","","Stopajlı Satın Alma KDV'si %20 (4/10)"
"tr_s_20","32","Sales VAT 20%","20%","20%","False","20","percent","","sale","tr_tax_group_kdv_20","base","invoice","","+SAL_BASE_20","Satış KDV'si %20"
"","","","","","","","","","","","tax","invoice","tr391001","+SAL_TAX_20",""
"","","","","","","","","","","","base","refund","","-SAL_BASE_20",""
"","","","","","","","","","","","tax","refund","tr391001","-SAL_TAX_20",""
"tr_s_10","33","Sales VAT 10%","10%","10%","False","10","percent","","sale","tr_tax_group_kdv_10","base","invoice","","+SAL_BASE_10","Satış KDV'si %10"
"","","","","","","","","","","","tax","invoice","tr391002","+SAL_TAX_10",""
"","","","","","","","","","","","base","refund","","-SAL_BASE_10",""
"","","","","","","","","","","","tax","refund","tr391002","-SAL_TAX_10",""
"tr_s_1","34","Sales VAT 1%","1%","1%","False","1","percent","","sale","tr_tax_group_kdv_1","base","invoice","","+SAL_BASE_1","Satış KDV'si %1"
"","","","","","","","","","","","tax","invoice","tr391003","+SAL_TAX_1",""
"","","","","","","","","","","","base","refund","","-SAL_BASE_1",""
"","","","","","","","","","","","tax","refund","tr391003","-SAL_TAX_1",""
"tr_p_20","35","Purchase VAT 20%","20%","20%","False","20","percent","","purchase","tr_tax_group_kdv_20","base","invoice","","+PUR_BASE_20","Satın alma KDV'si %20"
"","","","","","","","","","","","tax","invoice","tr191","+PUR_TAX_20",""
"","","","","","","","","","","","base","refund","","-PUR_BASE_20",""
"","","","","","","","","","","","tax","refund","tr191","-PUR_TAX_20",""
"tr_p_10","36","Purchase VAT 10%","10%","10%","False","10","percent","","purchase","tr_tax_group_kdv_10","base","invoice","","+PUR_BASE_10","Satın alma KDV'si %10"
"","","","","","","","","","","","tax","invoice","tr191","+PUR_TAX_10",""
"","","","","","","","","","","","base","refund","","-PUR_BASE_10",""
"","","","","","","","","","","","tax","refund","tr191","-PUR_TAX_10",""
"tr_p_1","37","Purchase VAT 1%","1%","1%","False","1","percent","","purchase","tr_tax_group_kdv_1","base","invoice","","+PUR_BASE_1","Satın alma KDV'si %1"
"","","","","","","","","","","","tax","invoice","tr191","+PUR_TAX_1",""
"","","","","","","","","","","","base","refund","","-PUR_BASE_1",""
"","","","","","","","","","","","tax","refund","tr191","-PUR_TAX_1",""
"tr_pr_wh_20","38","Purchase Rent Withholding Tax (20%)","-20%","PR WH 20%","False","-20","percent","","purchase","tr_tax_group_wh_20","base","invoice","","","Satın Alma Kira Stopajı Vergisi (%20)"
"","","","","","","","","","","","tax","invoice","tr191","",""
"","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","tax","refund","tr191","",""
"tr_wh_20_2_10","39","Withheld VAT 20% (Withholding 2/10)","-4%","WH 20% (2/10)","False","-4","percent","","none","tr_tax_group_wh_4","base","invoice","","+SAL_20_BASE_2_10","Tevkif KDV %20 (Tevkif 2/10)"
"","","","","","","","","","","","tax","invoice","tr391004","-SAL_20_TAX_2_10",""
"","","","","","","","","","","","base","refund","","-SAL_20_BASE_2_10",""
"","","","","","","","","","","","tax","refund","tr391004","+SAL_20_TAX_2_10",""

```

## File: data\template\account.tax.group-tr.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id","name@tr"
"tr_tax_group_kdv_18","VAT %18","base.tr","tr360","tr193","KDV %18"
"tr_tax_group_kdv_20","VAT %20","base.tr","tr360","tr193","KDV %20"
"tr_tax_group_kdv_n20","VAT %-20","base.tr","tr360","tr193","KDV %-20"
"tr_tax_group_kdv_0","VAT %0","base.tr","tr360","tr193","KDV %0"
"tr_tax_group_wh_20","WH %-20","base.tr","tr360","tr193","ne zaman %-20"
"tr_tax_group_wh_4","WH %-4","base.tr","tr360","tr193","ne zaman %-4"
"tr_tax_group_wh_10","WH %-10","base.tr","tr360","tr193","ne zaman %-10"
"tr_tax_group_wh_14","WH %-14","base.tr","tr360","tr193","ne zaman %-14"
"tr_tax_group_wh_18","WH %-18","base.tr","tr360","tr193","ne zaman %-18"
"tr_tax_group_wh_6","WH %-6","base.tr","tr360","tr193","ne zaman %-6"
"tr_tax_group_wh_8","WH %-8","base.tr","tr360","tr193","ne zaman %-8"
"tr_tax_group_kdv_10","VAT %10","base.tr","tr360","tr193","KDV %10"
"tr_tax_group_kdv_1","VAT %1","base.tr","tr360","tr193","KDV %1"

```

## File: migrations\1.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'tr')], order="parent_path"):
        env['account.chart.template'].try_loading('tr', company)

```

## File: migrations\1.2\end-migrate_update_package.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'tr')], order="parent_path"):
        env['account.chart.template'].try_loading('tr', company)

```

## File: migrations\1.3\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'tr')], order="parent_path"):
        env['account.chart.template'].try_loading('tr', company)

```

## File: models\template_tr.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('tr')
    def _get_tr_template_data(self):
        return {
            'property_account_receivable_id': 'tr120',
            'property_account_payable_id': 'tr320',
            'property_account_expense_categ_id': 'tr150',
            'property_account_income_categ_id': 'tr600',
            'code_digits': '6',
        }

    @template('tr', 'res.company')
    def _get_tr_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.tr',
                'bank_account_code_prefix': '102',
                'cash_account_code_prefix': '100',
                'transfer_account_code_prefix': '103',
                'account_default_pos_receivable_account_id': 'tr123',
                'income_currency_exchange_account_id': 'tr646',
                'expense_currency_exchange_account_id': 'tr656',
                'account_journal_suspense_account_id': 'tr102999',
                'account_journal_payment_debit_account_id': 'tr102997',
                'account_journal_payment_credit_account_id': 'tr102998',
                'account_sale_tax_id': 'tr_s_20',
                'account_purchase_tax_id': 'tr_p_20',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_tr

```

