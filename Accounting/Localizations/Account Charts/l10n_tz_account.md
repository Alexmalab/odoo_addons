# Odoo Module: l10n_tz_account

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Tanzania - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['tz'],
    'category': 'Accounting/Localizations/Account Charts',
    'version': '1.0',
    'depends': [
        'account',
    ],
    'description': """
    Tanzanian localisation containing:
    - COA
    - Taxes
    - Tax report
    - Fiscal position
    """,
    'data': [
        'data/l10n_tz_chart_data.xml',
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
<odoo auto_sequence="1">
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.tz"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_value" model="account.report.column">
                <field name="name">Value (Excluding VAT)</field>
                <field name="expression_label">value</field>
            </record>
            <record id="tax_report_rate" model="account.report.column">
                <field name="name">VAT Rate</field>
                <field name="expression_label">rate</field>
            </record>
            <record id="tax_report_amount" model="account.report.column">
                <field name="name">VAT Amount</field>
                <field name="expression_label">amount</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_goods_or_services" model="account.report.line">
                <field name="name">Supplies of goods &amp; or Services</field>
                <field name="code">tz_goods_or_services</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_taxable_supplies" model="account.report.line">
                        <field name="name">Taxable supplies</field>
                        <field name="code">taxable_supplies</field>
                        <field name="expression_ids">
                            <record id="tax_report_taxable_supplies_expr_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">02.base</field>
                            </record>
                            <record id="tax_report_taxable_supplies_expr_rate" model="account.report.expression">
                                <field name="label">rate</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                            <record id="tax_report_taxable_supplies_expr_amount" model="account.report.expression">
                                <field name="label">amount</field>
                                <field name="engine">external</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">03.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_capital_good_deferred_vat" model="account.report.line">
                        <field name="name">Value of capital Goods for which VAT is deferred</field>
                        <field name="code">capital_good_deferred_vat</field>
                        <field name="expression_ids">
                            <record id="tax_report_capital_good_deferred_vat_expr_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">05.base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_zero_rated_supplies" model="account.report.line">
                        <field name="name">Zero rated supplies</field>
                        <field name="code">zero_rated_supplies</field>
                        <field name="expression_ids">
                            <record id="tax_report_zero_rated_supplies_expr_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">06.base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_exempt_supplies" model="account.report.line">
                        <field name="name">Exempt supplies</field>
                        <field name="code">exempt_supplies</field>
                        <field name="expression_ids">
                            <record id="tax_report_exempt_supplies_expr_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">07.base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_special_relief" model="account.report.line">
                        <field name="name">Special relief / deferred supplies</field>
                        <field name="code">special_relief</field>
                        <field name="expression_ids">
                            <record id="tax_report_special_relief_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">08.base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_imported_services" model="account.report.line">
                        <field name="name">Value of imported services</field>
                        <field name="code">imported_services</field>
                        <field name="expression_ids">
                            <record id="tax_report_imported_services_expr_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">09.base</field>
                            </record>
                            <record id="tax_report_imported_services_expr_rate" model="account.report.expression">
                                <field name="label">rate</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                            <record id="tax_report_imported_services_expr_amount" model="account.report.expression">
                                <field name="label">amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">10.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_tz_total_output_tax" model="account.report.line">
                        <field name="name">Total output (sales) and Tax</field>
                        <field name="code">tz_total_output_tax</field>
                        <field name="expression_ids">
                            <record id="tax_report_tz_total_output_tax_expr_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">
                                    taxable_supplies.value
                                    + capital_good_deferred_vat.value
                                    + zero_rated_supplies.value
                                    + exempt_supplies.value
                                    + special_relief.value
                                    + imported_services.value
                                </field>
                            </record>
                            <record id="tax_report_tz_total_output_tax_expr_amount" model="account.report.expression">
                                <field name="label">amount</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">
                                    taxable_supplies.amount
                                    + imported_services.amount
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_purchase" model="account.report.line">
                <field name="name">Purchase (Inputs)</field>
                <field name="code">tz_purchase</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_exempt_purchases" model="account.report.line">
                        <field name="name">Exempt (local &amp; imports) purchases</field>
                        <field name="code">exempt_purchases</field>
                        <field name="expression_ids">
                            <record id="tax_report_exempt_purchases_expr_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">14.base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_non_creditable_purchases" model="account.report.line">
                        <field name="name">Non-Creditable purchases</field>
                        <field name="code">non_creditable_purchases</field>
                        <field name="expression_ids">
                            <record id="tax_report_non_creditable_purchases_expr_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">15.base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_capital_goods_deferred_vat_purchases" model="account.report.line">
                        <field name="name">Value of capital Goods for which VAT is deferred</field>
                        <field name="code">capital_goods_deferred_vat_purchases</field>
                        <field name="expression_ids">
                            <record id="tax_report_capital_goods_deferred_vat_purchases_expr_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">16.base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_imported_services_purchases" model="account.report.line">
                        <field name="name">Value of imported services</field>
                        <field name="code">imported_services_purchases</field>
                        <field name="expression_ids">
                            <record id="tax_report_imported_services_purchases_expr_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">17.base</field>
                            </record>
                            <record id="tax_report_imported_services_purchases_expr_rate" model="account.report.expression">
                                <field name="label">rate</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                            <record id="tax_report_imported_services_purchases_expr_amount" model="account.report.expression">
                                <field name="label">amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">18.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_taxable_supplies_purchases" model="account.report.line">
                        <field name="name">Taxable supplies</field>
                        <field name="code">taxable_supplies_purchases</field>
                        <field name="expression_ids">
                            <record id="tax_report_taxable_supplies_purchases_expr_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">20.base</field>
                            </record>
                            <record id="tax_report_taxable_supplies_purchases_expr_rate" model="account.report.expression">
                                <field name="label">rate</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                            <record id="tax_report_taxable_supplies_purchases_expr_amount" model="account.report.expression">
                                <field name="label">amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">21.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_imports_taxable_supplies_purchases" model="account.report.line">
                        <field name="name">Imports of taxable supplies</field>
                        <field name="code">imports_taxable_supplies_purchases</field>
                        <field name="expression_ids">
                            <record id="tax_report_imports_taxable_supplies_purchases_expr_value" model="account.report.expression">
                                <field name="label">value</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">23.base</field>
                            </record>
                            <record id="tax_report_imports_taxable_supplies_purchases_expr_rate" model="account.report.expression">
                                <field name="label">rate</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable</field>
                            </record>
                            <record id="tax_report_imports_taxable_supplies_purchases_expr_amount" model="account.report.expression">
                                <field name="label">amount</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">24.tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_tz_total_input_tax" model="account.report.line">
                        <field name="name">Total input tax</field>
                        <field name="code">tz_total_input_tax</field>
                        <field name="expression_ids">
                            <record id="tax_report_tz_total_input_tax_expr_amount" model="account.report.expression">
                                <field name="label">amount</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">
                                    imported_services_purchases.amount
                                    + taxable_supplies_purchases.amount
                                    + imports_taxable_supplies_purchases.amount
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_total_vat_payable" model="account.report.line">
                        <field name="name">Total VAT Payable/Refundable</field>
                        <field name="code">total_vat_payable</field>
                        <field name="expression_ids">
                            <record id="tax_report_total_vat_payable_expr_amount" model="account.report.expression">
                                <field name="label">amount</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">
                                    tz_total_output_tax.amount -
                                    tz_total_input_tax.amount
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_credit_bought_forward" model="account.report.line">
                <field name="name">VAT credit brought forward</field>
                <field name="code">tz_credit_bought_forward</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_credit_bought_forward_applied_carryover" model="account.report.expression">
                        <field name="label">_applied_carryover_amount</field>
                        <field name="engine">external</field>
                        <field name="formula">most_recent</field>
                        <field name="date_scope">previous_tax_period</field>
                    </record>
                    <record id="tax_report_credit_bought_forward_expr_amount" model="account.report.expression">
                        <field name="label">amount</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">tz_credit_bought_forward._applied_carryover_amount</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_total_vat_due" model="account.report.line">
                <field name="name">Total VAT due or carried forward</field>
                <field name="code">tz_total_vat_due</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="tax_report_total_vat_due_expr_amount" model="account.report.expression">
                        <field name="label">amount</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">total_vat_payable.amount + tz_credit_bought_forward.amount</field>
                    </record>
                    <record id="tz_total_vat_due_carryover" model="account.report.expression">
                        <field name="label">_carryover_amount</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">tz_total_vat_due.amount</field>
                        <field name="carryover_target">tz_credit_bought_forward._applied_carryover_amount</field>
                        <field name="subformula">if_below(TSH(0))</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\l10n_tz_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_tz_statements_menu" name="Tanzania" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>
</odoo>

```

## File: data\template\account.account-tz.csv

```csv
"id","code","name","account_type","reconcile","name@sw"
"tz_102","102","Cash Equivalents","asset_cash","False","Sawa za pesa"
"tz_104","104","Internal Transfers of Funds","asset_current","False","Uhamisho wa Fedha wa Ndani"
"tz_106","106","Other current assets","asset_current","False","Mali zingine za sasa"
"tz_107","107","Tax paid","asset_current","False","Kodi imelipwa"
"tz_108","108","Tax receivable","asset_current","False","Kodi inayopokelewa"
"tz_110","110","Property, Plant And Equipment","asset_fixed","False","Mali, Kiwanda na Vifaa"
"tz_111","111","Land And Land Improvements","asset_fixed","False","Uboreshaji wa Ardhi na Ardhi"
"tz_112","112","Buildings, Structures And Improvements","asset_fixed","False","Majengo, Miundo na Uboreshaji"
"tz_113","113","Machinery And Equipment","asset_fixed","False","Mashine na Vifaa"
"tz_114","114","Furniture And Fixtures","asset_fixed","False","Samani na Fixtures"
"tz_115","115","Right Of Use Assets (Classified As PP&E)","asset_fixed","False","Haki ya Matumizi ya Mali (Imeainishwa kama PP"
"tz_116","116","Additional Property, Plant And Equipment","asset_fixed","False","Mali ya Ziada, Mitambo na Vifaa"
"tz_117","117","Construction In Progress","asset_non_current","False","Ujenzi Unaendelea"
"tz_121","121","Investment Property","asset_non_current","False","Mali ya Uwekezaji"
"tz_122","122","Investment Property Under Construction Or Development","asset_non_current","False","Mali ya Uwekezaji Chini ya Ujenzi au Maendeleo"
"tz_130","130","Goodwill","asset_non_current","False","Nia njema"
"tz_135","135","Advances for Capital Assets","asset_non_current","False","Maendeleo kwa Mali ya Mtaji"
"tz_140","140","Intangible Assets (Excluding Goodwill)","asset_non_current","False","Mali Zisizogusika (Bila Nia Njema)"
"tz_141","141","Intellectual Property","asset_fixed","False","Mali ya kiakili"
"tz_142","142","Computer Software","asset_fixed","False","Programu ya Kompyuta"
"tz_143","143","Trade And Distribution Assets","asset_non_current","False","Biashara na Usambazaji Mali"
"tz_144","144","Contracts And Rights","asset_non_current","False","Mikataba na Haki"
"tz_145","145","Right To Use Assets (Classified By Type)","asset_non_current","False","Haki ya Kutumia Mali (Imeainishwa Kwa Aina)"
"tz_146","146","Other Intangible Assets","asset_non_current","False","Mali Nyingine Zisizogusika"
"tz_147","147","Acquisition In Progress","asset_non_current","False","Upatikanaji Unaendelea"
"tz_148","148","Deferred tax assets","asset_non_current","False","Mali ya ushuru iliyoahirishwa"
"tz_149","149","Available for sale investments","asset_non_current","False","Inapatikana kwa uwekezaji wa mauzo"
"tz_150","150","Financial Assets (Investments)","asset_non_current","False","Mali za Kifedha (Uwekezaji)"
"tz_151","151","Non-Derivative Financial Assets","asset_receivable","True","Mali Zisizotokana na Fedha"
"tz_152","152","Derivative Financial Assets","asset_receivable","True","Derivative Financial Assets"
"tz_153","153","Restricted Cash And Financial Assets","asset_cash","False","Pesa Zilizozuiliwa na Mali za Kifedha"
"tz_154","154","Additional Financial Assets And Investments","asset_non_current","False","Mali ya Ziada ya Fedha na Uwekezaji"
"tz_155","155","Trade receivables (PoS)","asset_receivable","True","Mapokezi ya Biashara (PoS)"
"tz_156","156","Investments in associate","asset_receivable","True","Uwekezaji kwa washirika"
"tz_160","160","Agricultural (Biological) Assets","asset_current","False","Mali za Kilimo (Biolojia)"
"tz_161","161","Bearer Plants","asset_current","False","Mimea ya kubeba"
"tz_162","162","Animals","asset_current","False","Wanyama"
"tz_163","163","Other Agricultural Assets","asset_current","False","Mali Nyingine za Kilimo"
"tz_170","170","Inventory","asset_current","False","Mali"
"tz_171","171","Merchandise","asset_current","False","Bidhaa"
"tz_172","172","Raw Material, Parts And Supplies","asset_current","False","Malighafi, Sehemu na Ugavi"
"tz_173","173","Work In Process","asset_current","False","Fanya Kazi Katika Mchakato"
"tz_174","174","Finished Goods","asset_current","False","Bidhaa za kumaliza"
"tz_175","175","Other Inventory","asset_current","False","Mali nyingine"
"tz_176","176","Income tax assets","asset_current","False","Mali ya kodi ya mapato"
"tz_180","180","Accruals And Additional Assets","asset_receivable","True","Mapato na Mali ya Ziada"
"tz_181","181","Prepaid Expense","asset_current","False","Gharama ya kulipia kabla"
"tz_182","182","Accrued Income","asset_current","False","Mapato ya ziada"
"tz_183","183","Additional Assets","asset_receivable","True","Mali ya Ziada"
"tz_184","184","Investments and financial receivables","asset_receivable","True","Uwekezaji na mapato ya kifedha"
"tz_190","190","Receivables And Contracts","asset_receivable","True","Mapokezi na Mikataba"
"tz_191","191","Accounts, Notes And Loans Receivable","asset_receivable","True","Akaunti, Noti na Mikopo Inayopatikana"
"tz_192","192","Contracts","asset_current","False","Mikataba"
"tz_193","193","Nontrade And Other Receivables","asset_receivable","True","Nontrade na Receivables Nyingine"
"tz_210","210","Owners Equity (Attributable To Owners Of Parent)","equity","False","Sawa ya Wamiliki (Inayohusishwa na Wamiliki wa Mzazi)"
"tz_211","211","Equity At Par (Issued Capital)","equity","False","Equity At Par (Mtaji Uliotolewa)"
"tz_212","212","Retained Earnings","equity","False","Mapato yaliyobaki"
"tz_213","213","Additional Paid-In Capital","equity","False","Mtaji wa Ziada wa Kulipwa"
"tz_220","220","Treasury Stock","equity","False","Hazina ya Hifadhi"
"tz_221","221","Treasury Stock Common","equity","False","Hazina ya kawaida ya hisa"
"tz_222","222","Treasury Stock Preferred","equity","False","Hazina Inapendekezwa"
"tz_230","230","Accumulated OCI","equity","False","OCI iliyokusanywa"
"tz_231","231","Exchange Differences On Translation","equity","False","Badilisha Tofauti kwenye Tafsiri"
"tz_232","232","Remeasurements Cash Flow Hedges","equity","False","Vipimo vya Ua wa mtiririko wa pesa"
"tz_233","233","Remeasurements Available-For-Sale Financial Assets","equity","False","Vipimo Vinavyopatikana-Kwa-Kuuzwa Mali ya Fedha"
"tz_234","234","Remeasurement Of Defined Benefit Plans","equity","False","Upimaji upya wa Mipango ya Faida iliyoainishwa"
"tz_235","235","Revaluation Surplus (IFRS Only)","equity","False","Ziada ya Uhakiki (IFRS Pekee)"
"tz_236","236","Remeasurements Investments In Equity Instruments (IFRS only)","equity","False","Uwekezaji wa Vipimo Katika Hati za Usawa (IFRS pekee)"
"tz_240","240","Other Equity Items","equity","False","Vitu vingine vya Usawa"
"tz_241","241","ESOP Related Items","equity","False","Vipengee vinavyohusiana na ESOP"
"tz_242","242","Subscribed Stock Receivables","equity","False","Mapokezi ya Hisa Unayosajiliwa"
"tz_250","250","Miscellaneous Equity","equity","False","Usawa Mbalimbali"
"tz_260","260","Non-controlling (Minority) Interest","equity","False","Maslahi yasiyo ya kudhibiti (Wachache)"
"tz_270","270","Share capital","equity","False","Shiriki mtaji"
"tz_307","307","Tax received","liability_current","False","Kodi imepokelewa"
"tz_308","308","Tax Payable","liability_current","False","Kodi inayolipwa"
"tz_311","311","Trade Payables","liability_payable","True","Malipo ya Biashara"
"tz_312","312","Dividends Payable","liability_payable","True","Gawio linalolipwa"
"tz_313","313","Interest Payable","liability_payable","True","Riba Inalipwa"
"tz_314","314","Other Payables","liability_payable","True","Malipo mengine"
"tz_320","320","Provisions (Contingencies)","liability_current","False","Masharti (ya Dharura)"
"tz_321","321","Customer Related Provisions","liability_current","False","Masharti Yanayohusiana na Wateja"
"tz_322","322","Ligation And Regulatory Provisions","liability_current","False","Masharti ya Kuunganisha na Udhibiti"
"tz_323","323","Additional Provisions","liability_current","False","Masharti ya Ziada"
"tz_330","330","Financial Liabilities","liability_current","False","Madeni ya Kifedha"
"tz_331","331","Notes Payable","liability_payable","True","Noti Zinazolipwa"
"tz_332","332","Loans Payable","liability_payable","True","Mikopo inayolipwa"
"tz_333","333","Bonds (Debentures)","liability_current","False","Vifungo (Debentures)"
"tz_334","334","Other Debts And Borrowings","liability_current","False","Madeni Mengine na Mikopo"
"tz_335","335","Lease Obligations","liability_current","False","Majukumu ya kukodisha"
"tz_336","336","Derivative Financial Liabilities","liability_current","False","Derivative Financial Liabilities"
"tz_340","340","Accruals And Other Liabilities","liability_current","False","Accruals na Madeni Mengine"
"tz_341","341","Accrued Expenses","liability_current","False","Gharama yatokanayo"
"tz_342","342","Deferred Income (Unearned Revenue)","liability_current","False","Mapato Yaliyoahirishwa (Mapato Yasiyopatikana)"
"tz_343","343","Accrued Taxes (Other Than Payroll)","liability_current","False","Kodi Zilizoongezwa (Zaidi ya Malipo)"
"tz_344","344","Other Liabilities","liability_current","False","Madeni mengine"
"tz_350","350","Banks overdrafts and short-term borrowings","liability_current","False","Overdrafts za benki na mikopo ya muda mfupi"
"tz_360","360","Interest-bearing loans and short term borrowings","liability_current","False","Mikopo yenye riba na mikopo ya muda mfupi"
"tz_370","370","Income tax liabilities","liability_current","False","Madeni ya kodi ya mapato"
"tz_380","380","Interest-bearing loans and short term borrowings","liability_non_current","False","Mikopo yenye riba na mikopo ya muda mfupi"
"tz_390","390","Employee benefits liabilities","liability_non_current","False","Madeni ya faida ya mfanyakazi"
"tz_395","395","Provisions","liability_non_current","False","Masharti"
"tz_396","396","Deferred tax liabilities","liability_non_current","False","Madeni ya ushuru yaliyoahirishwa"
"tz_400","400","Revenue","income","False","Mapato"
"tz_410","410","Recognized Point Of Time","income","False","Wakati Unaotambuliwa"
"tz_411","411","Goods","income","False","Bidhaa"
"tz_412","412","Services","income","False","Huduma"
"tz_420","420","Recognized Over Time","income","False","Inatambuliwa kwa Wakati"
"tz_421","421","Products","income","False","Bidhaa"
"tz_422","422","Services","income","False","Huduma"
"tz_430","430","Adjustments","income","False","Marekebisho"
"tz_431","431","Variable Consideration","income","False","Kuzingatia Tofauti"
"tz_432","432","Consideration Paid (Payable) To Customers","income","False","Kuzingatia Kulipwa (Kulipwa) kwa Wateja"
"tz_433","433","Other Adjustments","income","False","Marekebisho Mengine"
"tz_510","510","Expenses Classified By Nature","expense","False","Gharama Zilizoainishwa Kwa Asili"
"tz_511","511","Material And Merchandise","expense","False","Nyenzo na Bidhaa"
"tz_512","512","Employee Benefits","expense","False","Faida za Wafanyikazi"
"tz_513","513","Services","expense","False","Huduma"
"tz_514","514","Rent, Depreciation, Amortization And Depletion","expense","False","Kukodisha, Kushuka kwa Thamani, Malipo ya Mapato na Kupungua"
"tz_515","515","Increase (Decrease) In Inventories Of Finished Goods And Work In Progress (IFRS only)","expense","False","Ongezeko (Kupungua) Katika Orodha ya Bidhaa Zilizomalizika na Kazi Inayoendelea (IFRS pekee)"
"tz_516","516","Other Work Performed By Entity And Capitalized (IFRS only)","expense","False","Kazi Nyingine Zinazofanywa na Shirika na Mtaji (IFRS pekee)"
"tz_520","520","Expenses Classified By Function","expense","False","Gharama Zimeainishwa Kulingana na Kazi"
"tz_521","521","Cost Of Sales","expense","False","Gharama ya Mauzo"
"tz_522","522","Selling, General And Administrative","expense","False","Kuuza, Jumla na Utawala"
"tz_611","611","Other Revenue","income","False","Mapato mengine"
"tz_612","612","Other Expenses","expense","False","Gharama Nyingine"
"tz_613","613","Change in inventories","income","False","Mabadiliko katika orodha"
"tz_614","614","Change in fair value of investment property","income","False","Mabadiliko ya thamani ya haki ya mali ya uwekezaji"
"tz_615","615","Depreciation, amortisation and impairment of non-financial assets","income","False","Kushuka kwa thamani, punguzo na uharibifu wa mali zisizo za kifedha"
"tz_616","616","Impairment losses of financial assets","expense","False","Hasara za uharibifu wa mali ya kifedha"
"tz_6211","6211","Foreign Currency Transaction Gain","income","False","Faida ya Muamala wa Fedha za Kigeni"
"tz_6212","6212","Foreign Currency Transaction Loss","expense","False","Hasara ya Muamala wa Fedha za Kigeni"
"tz_6221","6221","Gain On Investments","income","False","Faida kwa Uwekezaji"
"tz_6222","6222","Loss On Investments","expense","False","Hasara kwenye Uwekezaji"
"tz_6231","6231","Gain On Derivatives","income","False","Pata kwenye Miche"
"tz_6232","6232","Loss On Derivatives","expense","False","Hasara juu ya derivatives"
"tz_6241","6241","Gain On Disposal Of Assets","income","False","Faida ya Utoaji wa Mali"
"tz_6242","6242","Loss On Disposal Of Assets","expense","False","Hasara ya Uondoaji wa Mali"
"tz_6251","6251","Debt Related Gain","income","False","Faida inayohusiana na deni"
"tz_6252","6252","Debt Related Loss","expense","False","Hasara inayohusiana na deni"
"tz_626","626","Impairment Loss","expense","False","Hasara ya uharibifu"
"tz_627","627","Impairment Loss (Reversal) Financial Assets (IFRS Only)","expense","False","Hasara ya Uharibifu (Kurudishwa) Mali za Kifedha (IFRS Pekee)"
"tz_6281","6281","Other Gains","income","False","Faida Nyingine"
"tz_6282","6282","Other Losses","expense","False","Hasara Nyingine"
"tz_630","630","Taxes (Other Than Income And Payroll) And Fees","expense","False","Kodi (Zaidi ya Mapato na Malipo) na Ada"
"tz_631","631","Real Estate Taxes And Insurance","expense","False","Kodi ya Mali isiyohamishika na Bima"
"tz_632","632","Highway (Road) Taxes And Tolls","expense","False","Ushuru na Ushuru wa Barabara kuu (Barabara)"
"tz_633","633","Direct Tax And License Fees","expense","False","Kodi ya Moja kwa Moja na Ada za Leseni"
"tz_634","634","Excise And Sales Taxes","expense","False","Ushuru na Ushuru wa Mauzo"
"tz_635","635","Customs Fees And Duties (Not Classified As Sales Or Excise)","expense","False","Ada na Ushuru wa Forodha (Hazijaainishwa kama Mauzo au Ushuru)"
"tz_636","636","Non-Deductible VAT (GST)","expense","False","VAT Isiyokatwa (GST)"
"tz_637","637","General Insurance Expense","expense","False","Gharama za Bima ya Jumla"
"tz_638","638","Administrative Fees (Revenue Stamps)","expense","False","Ada za Utawala (Stampu za Mapato)"
"tz_639","639","Fines And Penalties","expense","False","Faini na Adhabu"
"tz_640","640","Income Tax Expense (Benefit)","expense","False","Gharama ya Kodi ya Mapato (Faida)"
"tz_650","650","Miscellaneous Taxes","expense","False","Kodi Mbalimbali"
"tz_660","660","Other Taxes And Fees","expense","False","Kodi Nyingine na Ada"
"tz_671","671","Foreign Exchange Gain","income","False","Faida ya Fedha za Kigeni"
"tz_672","672","Foreign Exchange Loss","expense","False","Hasara ya Fedha za Kigeni"
"tz_680","680","Share of profit from equity accounted investments","income","False","Mgawo wa faida kutoka kwa uwekezaji uliohesabiwa kwa hisa"
"tz_681","681","Finance costs","expense","False","Gharama za fedha"
"tz_682","682","Finance income","income","False","Mapato ya kifedha"
"tz_683","683","Other financial items","expense","False","Vitu vingine vya kifedha"
"tz_690","690","Loss for the year from discontinued operations","expense","False","Hasara kwa mwaka kutokana na shughuli zilizositishwa"

```

## File: data\template\account.fiscal.position-tz.csv

```csv
"id","name","sequence","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@sw"
"fiscal_position_tz_exempt","Exempt taxpayer","1","0","","","","VAT_S_TAXABLE_18","VAT_S_EXEMPT_O","Mlipa kodi"
"","","","","","","","VAT_S_IMPORTED_18","VAT_S_EXEMPT_O",""
"","","","","","","","VAT_P_IMPORTED_18","VAT_P_EXEMPT_O",""
"","","","","","","","VAT_P_TAXABLE_18","VAT_P_EXEMPT_O",""
"","","","","","","","VAT_P_IMPORT_TAXABLE_18","VAT_P_EXEMPT_O",""
"fiscal_position_tz_national","Domestic","2","1","1","base.tz","","","","Ndani"
"fiscal_position_tz_international","International","3","1","","","","VAT_S_TAXABLE_18","VAT_S_IMPORTED_18","Kimataifa"
"","","","","","","","VAT_P_TAXABLE_18","VAT_P_IMPORTED_18",""

```

## File: data\template\account.tax-tz.csv

```csv
"id","sequence","description","invoice_label","name","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","description@sw","invoice_label@sw"
"VAT_S_TAXABLE_18","1","Taxable supply","Taxable supply","18%","18.0","percent","sale","tax_group_vat_18","100","base","invoice","+02.base","","Ugavi wa ushuru","Ugavi wa ushuru"
"","","","","","","","","","100","tax","invoice","+03.tax","tz_307","",""
"","","","","","","","","","100","base","refund","+02.base","","",""
"","","","","","","","","","100","tax","refund","+03.tax","tz_307","",""
"VAT_S_ZERO_O","3","Zero rated supply","Zero rated supply","0%","0.0","percent","sale","tax_group_vat_0","100","base","invoice","+06.base","","Ugavi uliokadiriwa sifuri","Ugavi uliokadiriwa sifuri"
"","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","100","base","refund","+06.base","","",""
"","","","","","","","","","100","tax","refund","","","",""
"VAT_S_EXEMPT_O","4","Exempt supply","Exempt supply","0% Exempt","0.0","percent","sale","tax_group_vat_0","100","base","invoice","+07.base","","Ugavi usioruhusiwa","Ugavi usioruhusiwa"
"","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","100","base","refund","+07.base","","",""
"","","","","","","","","","100","tax","refund","","","",""
"VAT_S_SPRECIAL_RELIEF_O","5","Special relief/deferred supply","Special relief/deferred supply","0% D","0.0","percent","sale","tax_group_vat_0","100","base","invoice","+08.base","","Ugavi maalum wa unafuu/ulioahirishwa","Ugavi maalum wa unafuu/ulioahirishwa"
"","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","100","base","refund","+08.base","","",""
"","","","","","","","","","100","tax","refund","","","",""
"VAT_S_IMPORTED_18","6","Imported service","Imported service","18% EX S","18.0","percent","sale","tax_group_vat_18","100","base","invoice","+09.base","","Huduma iliyoingizwa","Huduma iliyoingizwa"
"","","","","","","","","","100","tax","invoice","+10.tax","tz_307","",""
"","","","","","","","","","100","base","refund","+09.base","","",""
"","","","","","","","","","100","tax","refund","+10.tax","tz_307","",""
"VAT_P_EXEMPT_O","7","Exempt purchase","Exempt purchase","0% Exempt","0.0","percent","purchase","tax_group_vat_0","100","base","invoice","+14.base","","Ununuzi wa msamaha","Ununuzi wa msamaha"
"","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","100","base","refund","+14.base","","",""
"","","","","","","","","","100","tax","refund","","","",""
"VAT_P_NCREDITABLE_O","8","Non-creditable purchase","Non-creditable purchase","0%","0.0","percent","purchase","tax_group_vat_0","100","base","invoice","+15.base","","Ununuzi usio na mkopo","Ununuzi usio na mkopo"
"","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","100","base","refund","+15.base","","",""
"","","","","","","","","","100","tax","refund","","","",""
"VAT_P_IMPORTED_18","10","Imported service","Imported service","18% EX S","18.0","percent","purchase","tax_group_vat_0","100","base","invoice","+17.base","","Huduma iliyoingizwa","Huduma iliyoingizwa"
"","","","","","","","","","100","tax","invoice","+18.tax","tz_107","",""
"","","","","","","","","","100","base","refund","+17.base","","",""
"","","","","","","","","","100","tax","refund","+18.tax","tz_107","",""
"VAT_P_TAXABLE_18","11","Taxable supply","Taxable supply","18%","18.0","percent","purchase","tax_group_vat_0","100","base","invoice","+20.base","","Ugavi wa ushuru","Ugavi wa ushuru"
"","","","","","","","","","100","tax","invoice","+21.tax","tz_107","",""
"","","","","","","","","","100","base","refund","+20.base","","",""
"","","","","","","","","","100","tax","refund","+21.tax","tz_107","",""
"VAT_P_IMPORT_TAXABLE_18","12","Import of taxable supplies","Import of taxable supplies","18% EX G","18.0","percent","purchase","tax_group_vat_0","100","base","invoice","+23.base","","Uagizaji wa vifaa vinavyotozwa ushuru","Uagizaji wa vifaa vinavyotozwa ushuru"
"","","","","","","","","","100","tax","invoice","+24.tax","tz_107","",""
"","","","","","","","","","100","base","refund","+23.base","","",""
"","","","","","","","","","100","tax","refund","+24.tax","tz_107","",""

```

## File: data\template\account.tax.group-tz.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id"
"tax_group_vat_0","VAT 0%","base.tz","tz_108","tz_308"
"tax_group_vat_18","VAT 18%","base.tz","tz_108","tz_308"

```

## File: models\template_tz.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('tz')
    def _get_tz_template_data(self):
        return {
            'code_digits': '4',
            'property_account_receivable_id': 'tz_190',
            'property_account_payable_id': 'tz_311',
            'property_account_expense_categ_id': 'tz_510',
            'property_account_income_categ_id': 'tz_400',
        }

    @template('tz', 'res.company')
    def _get_tz_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.tz',
                'cash_account_code_prefix': '101',
                'bank_account_code_prefix': '103',
                'transfer_account_code_prefix': '105',
                'account_default_pos_receivable_account_id': 'tz_155',
                'income_currency_exchange_account_id': 'tz_671',
                'expense_currency_exchange_account_id': 'tz_672',
                'deferred_revenue_account_id': 'tz_181',
                'deferred_expense_account_id': 'tz_342',
                'account_sale_tax_id': 'VAT_S_TAXABLE_18',
                'account_purchase_tax_id': 'VAT_P_TAXABLE_18',
            },
        }

```

## File: models\__init__.py

```python
from . import template_tz

```

