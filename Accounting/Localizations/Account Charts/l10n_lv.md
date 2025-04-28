# Odoo Module: l10n_lv

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: Readme.txt

```text
LV:
Pēc šī moduļa uzstādīšanas tiks izveidoti:
* standarta kontu plāns,
* kontu grupu saraksts,
* PVN analītika,
* Latvijas un Lietuvas banku saraksts.

Lai varētu pilnvērtīgi izmantot šī moduļa iespējas, jums būs nepieciešami sekojoši moduļi:
* account,
* base_vat,
* l10n_multilang,
* kā arī account_accountant Enterprise versijai vai analogs modulis Community versijai.

PVN konts ir 5721, bet jūs variet to mainīt sadaļā Grāmatvedība -> Nodokļi.
Konta piesaiste bilances un peļņas vai zaudējumu posteņiem tiek veikta, izmantojot “tag_ids/id” sadaļā Kontu plāns – Birkas.

EN:
by installing these modules, will be created:
* Chart of accounts (standardized),
* the list of account groups,
* VAT analytics,
* The list of banks of Latvia and Lithuania.

To get around and make full use of these features, you will need the following modules:
* account,
* base_vat,
* l10n_multilang,
* as well as account_accountant (for Enterprise version) or similar modules for Community version.

The VAT account is created as 5721, but you can modify and change it via Accounting -> Taxes.
Account linking to Balance Sheet items and Profit/Loss items - via "tag_ids/id" in Chart of Accounts - Tags

```

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
{
    'name': "Latvia - Accounting",
    'version': '1.0.0',
    'description': """
Chart of Accounts (COA) Template for Latvia's Accounting.
This module also includes:
* Tax groups,
* Most common Latvian Taxes,
* Fiscal positions,
* Latvian bank list.

author is Allegro IT (visit for more information https://www.allegro.lv)
co-author is Chick.Farm (visit for more information https://www.myacc.cloud)
    """,
    'license': 'LGPL-3',
    'author': "Allegro IT, Chick.Farm",
    'website': "https://allegro.lv",
    'category': 'Accounting/Localizations/Account Charts',
    'depends': [
        'account',
        'base_vat',
    ],
    'data': [
        'data/vat_tax_report.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
}

```

## File: data\vat_tax_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_lv_vat_main_tax_report" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.lv"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="vat_report_column_code" model="account.report.column">
                <field name="name">Code</field>
                <field name="expression_label">code</field>
                <field name="figure_type">string</field>
            </record>
            <record id="vat_report_column_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="vat_report_line_40" model="account.report.line">
                <field name="name">Value of activities</field>
                <field name="code">LV_40</field>
                <field name="expression_ids">
                    <record id="vat_report_line_40_code" model="account.report.expression">
                        <field name="label">code</field>
                        <field name="auditable" eval="False"/>
                        <field name="engine">aggregation</field>
                        <field name="formula">40</field>
                    </record>
                    <record id="vat_report_line_40_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">LV_41.balance + LV_411.balance + LV_42.balance + LV_421.balance + LV_43.balance + LV_482.balance + LV_49.balance + LV_50.balance + LV_51.balance + LV_511.balance</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="vat_report_line_41" model="account.report.line">
                        <field name="name">Transactions subject to VAT at standard rate</field>
                        <field name="code">LV_41</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_41_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">41</field>
                            </record>
                            <record id="vat_report_line_41_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">41</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_411" model="account.report.line">
                        <field name="name">Domestic transactions for which the recipient of the goods or services is liable for tax</field>
                        <field name="code">LV_411</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_411_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">411</field>
                            </record>
                            <record id="vat_report_line_411_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">411</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_42" model="account.report.line">
                        <field name="name">Transactions subject to VAT at 12%</field>
                        <field name="code">LV_42</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_42_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">42</field>
                            </record>
                            <record id="vat_report_line_42_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">42</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_421" model="account.report.line">
                        <field name="name">Transactions subject to VAT at 5%</field>
                        <field name="code">LV_421</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_421_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">421</field>
                            </record>
                            <record id="vat_report_line_421_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">421</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_43" model="account.report.line">
                        <field name="name">Transactions subject to VAT at 0%</field>
                        <field name="code">LV_43</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_43_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">43</field>
                            </record>
                            <record id="vat_report_line_43_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">LV_44.balance + LV_45.balance + LV_451.balance + LV_46.balance + LV_47.balance + LV_48.balance + LV_481.balance</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="vat_report_line_44" model="account.report.line">
                                <field name="name">Transactions carried out in free ports and SEZ</field>
                                <field name="code">LV_44</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line_44_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">44</field>
                                    </record>
                                    <record id="vat_report_line_44_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">44</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line_45" model="account.report.line">
                                <field name="name">Goods supplied to EU Member States, other than those referred to in Article 42(16) of the Law</field>
                                <field name="code">LV_45</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line_45_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">45</field>
                                    </record>
                                    <record id="vat_report_line_45_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">45</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line_451" model="account.report.line">
                                <field name="name">Goods supplied to EU Member States as referred to in Article 42(16) of the Law</field>
                                <field name="code">LV_451</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line_451_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">451</field>
                                    </record>
                                    <record id="vat_report_line_451_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">451</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line_46" model="account.report.line">
                                <field name="name">Supplies of non-Community goods in customs warehouses and free zones</field>
                                <field name="code">LV_46</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line_46_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">46</field>
                                    </record>
                                    <record id="vat_report_line_46_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">46</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line_47" model="account.report.line">
                                <field name="name">New vehicles delivered to EU Member States</field>
                                <field name="code">LV_47</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line_47_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">47</field>
                                    </record>
                                    <record id="vat_report_line_47_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">47</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line_48" model="account.report.line">
                                <field name="name">For services provided</field>
                                <field name="code">LV_48</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line_48_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">48</field>
                                    </record>
                                    <record id="vat_report_line_48_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">48</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line_481" model="account.report.line">
                                <field name="name">Exported goods</field>
                                <field name="code">LV_481</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line_481_code" model="account.report.expression">
                                        <field name="label">code</field>
                                        <field name="auditable" eval="False"/>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">481</field>
                                    </record>
                                    <record id="vat_report_line_481_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">481</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_482" model="account.report.line">
                        <field name="name">Transactions in other countries</field>
                        <field name="code">LV_482</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_482_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">482</field>
                            </record>
                            <record id="vat_report_line_482_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">482</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_49" model="account.report.line">
                        <field name="name">VAT-exempt transactions</field>
                        <field name="code">LV_49</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_49_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">49</field>
                            </record>
                            <record id="vat_report_line_49_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">49</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_50" model="account.report.line">
                        <field name="name">Goods and services received from EU countries (standard rate)</field>
                        <field name="code">LV_50</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_50_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">50</field>
                            </record>
                            <record id="vat_report_line_50_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">50</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_51" model="account.report.line">
                        <field name="name">Goods received from EU Member States (VAT at 12%)</field>
                        <field name="code">LV_51</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_51_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">51</field>
                            </record>
                            <record id="vat_report_line_51_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">51</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_511" model="account.report.line">
                        <field name="name">Goods received from EU Member States (VAT at 5%)</field>
                        <field name="code">LV_511</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_511_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">511</field>
                            </record>
                            <record id="vat_report_line_511_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">511</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="vat_report_line_5x" model="account.report.line">
                <field name="name">Calculated VAT</field>
                <field name="code">LV_5x</field>
                <field name="expression_ids">
                    <record id="vat_report_line_5x_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">LV_52.balance + LV_53.balance + LV_531.balance + LV_54.balance + LV_55.balance + LV_56.balance + LV_561.balance</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="vat_report_line_52" model="account.report.line">
                        <field name="name">VAT at standard rate received on taxable transactions</field>
                        <field name="code">LV_52</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_52_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">52</field>
                            </record>
                            <record id="vat_report_line_52_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">52</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_53" model="account.report.line">
                        <field name="name">VAT at 12% received on taxable transactions</field>
                        <field name="code">LV_53</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_53_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">53</field>
                            </record>
                            <record id="vat_report_line_53_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">53</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_531" model="account.report.line">
                        <field name="name">VAT at 5% received on taxable transactions</field>
                        <field name="code">LV_531</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_531_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">531</field>
                            </record>
                            <record id="vat_report_line_531_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">531</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_54" model="account.report.line">
                        <field name="name">VAT for services received</field>
                        <field name="code">LV_54</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_54_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">54</field>
                            </record>
                            <record id="vat_report_line_54_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">54</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_55" model="account.report.line">
                        <field name="name">VAT at standard rate on goods and services received from EU countries</field>
                        <field name="code">LV_55</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_55_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">55</field>
                            </record>
                            <record id="vat_report_line_55_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">55</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_56" model="account.report.line">
                        <field name="name">VAT at 12% on goods received from EU countries</field>
                        <field name="code">LV_56</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_56_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">56</field>
                            </record>
                            <record id="vat_report_line_56_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">56</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_561" model="account.report.line">
                        <field name="name">VAT at 5% on goods and services received from EU countries</field>
                        <field name="code">LV_561</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_561_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">561</field>
                            </record>
                            <record id="vat_report_line_561_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">561</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="vat_report_line_60" model="account.report.line">
                <field name="name">VAT paid for goods and services purchased</field>
                <field name="code">LV_60</field>
                <field name="expression_ids">
                    <record id="vat_report_line_60_code" model="account.report.expression">
                        <field name="label">code</field>
                        <field name="auditable" eval="False"/>
                        <field name="engine">aggregation</field>
                        <field name="formula">60</field>
                    </record>
                    <record id="vat_report_line_60_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">LV_61.balance + LV_62.balance + LV_63.balance + LV_64.balance + LV_65.balance</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="vat_report_line_61" model="account.report.line">
                        <field name="name">For imported goods</field>
                        <field name="code">LV_61</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_61_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">61</field>
                            </record>
                            <record id="vat_report_line_61_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">61</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_62" model="account.report.line">
                        <field name="name">For domestic goods and services</field>
                        <field name="code">LV_62</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_62_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">62</field>
                            </record>
                            <record id="vat_report_line_62_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">62</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_63" model="account.report.line">
                        <field name="name">VAT charged in accordance with Article 92(1)(4) of the Law (excluding line 64)</field>
                        <field name="code">LV_63</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_63_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">63</field>
                            </record>
                            <record id="vat_report_line_63_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">63</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_64" model="account.report.line">
                        <field name="name">VAT charged on goods and services received from EU countries</field>
                        <field name="code">LV_64</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_64_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">64</field>
                            </record>
                            <record id="vat_report_line_64_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">64</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line_65" model="account.report.line">
                        <field name="name">Compensation paid to farmers</field>
                        <field name="code">LV_65</field>
                        <field name="expression_ids">
                            <record id="vat_report_line_65_code" model="account.report.expression">
                                <field name="label">code</field>
                                <field name="auditable" eval="False"/>
                                <field name="engine">aggregation</field>
                                <field name="formula">65</field>
                            </record>
                            <record id="vat_report_line_65_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">65</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="vat_report_line_66" model="account.report.line">
                <field name="name">VAT paid not deductible as input tax</field>
                <field name="code">LV_66</field>
                <field name="expression_ids">
                    <record id="vat_report_line_66_code" model="account.report.expression">
                        <field name="label">code</field>
                        <field name="auditable" eval="False"/>
                        <field name="engine">aggregation</field>
                        <field name="formula">66</field>
                    </record>
                    <record id="vat_report_line_66_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">66</field>
                    </record>
                </field>
            </record>
            <record id="vat_report_line_67" model="account.report.line">
                <field name="name">Adjustments - reduction of tax calculated in previous tax periods for payment to the state budget</field>
                <field name="code">LV_67</field>
                <field name="expression_ids">
                    <record id="vat_report_line_67_code" model="account.report.expression">
                        <field name="label">code</field>
                        <field name="auditable" eval="False"/>
                        <field name="engine">aggregation</field>
                        <field name="formula">67</field>
                    </record>
                    <record id="vat_report_line_67_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">external</field>
                        <field name="formula">most_recent</field>
                        <field name="subformula">editable;rounding=2</field>
                    </record>
                </field>
            </record>
            <record id="vat_report_line_57" model="account.report.line">
                <field name="name">Adjustments - reduction of input tax deducted in previous tax periods</field>
                <field name="code">LV_57</field>
                <field name="expression_ids">
                    <record id="vat_report_line_57_code" model="account.report.expression">
                        <field name="label">code</field>
                        <field name="auditable" eval="False"/>
                        <field name="engine">aggregation</field>
                        <field name="formula">57</field>
                    </record>
                    <record id="vat_report_line_57_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">external</field>
                        <field name="formula">most_recent</field>
                        <field name="subformula">editable;rounding=2</field>
                    </record>
                </field>
            </record>
            <record id="vat_report_line_P" model="account.report.line">
                <field name="name">Total - input tax (P)</field>
                <field name="code">LV_P</field>
                <field name="expression_ids">
                    <record id="vat_report_line_P_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">LV_60.balance + LV_67.balance</field>
                    </record>
                </field>
            </record>
            <record id="vat_report_line_S" model="account.report.line">
                <field name="name">Total - tax calculated (S)</field>
                <field name="code">LV_S</field>
                <field name="expression_ids">
                    <record id="vat_report_line_S_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">LV_5x.balance + LV_57.balance</field>
                    </record>
                </field>
            </record>
            <record id="vat_report_line_70" model="account.report.line">
                <field name="name">Amount of tax to be refunded from the state budget or amount of tax attributable to the next tax period, if P is greater than S</field>
                <field name="code">LV_70</field>
                <field name="expression_ids">
                    <record id="vat_report_line_70_code" model="account.report.expression">
                        <field name="label">code</field>
                        <field name="auditable" eval="False"/>
                        <field name="engine">aggregation</field>
                        <field name="formula">70</field>
                    </record>
                    <record id="vat_report_line_70_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">LV_P.balance - LV_S.balance</field>
                        <field name="subformula">if_above(EUR(0))</field>
                    </record>
                </field>
                <field name="aggregation_formula"></field>
            </record>
            <record id="vat_report_line_80" model="account.report.line">
                <field name="name">Amount of tax payable to the national budget if P is less than S</field>
                <field name="code">LV_80</field>
                <field name="expression_ids">
                    <record id="vat_report_line_80_code" model="account.report.expression">
                        <field name="label">code</field>
                        <field name="auditable" eval="False"/>
                        <field name="engine">aggregation</field>
                        <field name="formula">80</field>
                    </record>
                    <record id="vat_report_line_80_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">LV_S.balance - LV_P.balance</field>
                        <field name="subformula">if_above(EUR(0))</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-lv.csv

```csv
"id","name","code","account_type","tag_ids","reconcile","name@lv"
"a1110","Research and company development costs","1110","asset_non_current","","False","Pētniecības un uzņēmuma attīstības izmaksas"
"a1120","Concessions, patents, licenses, trademarks and similar rights","1120","asset_non_current","","False","Koncesijas, patenti, licences, preču zīmes un tamlīdzīgas tiesības"
"a1130","Other intangible assets","1130","asset_non_current","","False","Citi nemateriālie ieguldījumi"
"a1140","Goodwill","1140","asset_non_current","","False","Nemateriālā vērtība"
"a1180","Advance payments for intangible assets","1150","asset_prepayments","","False","Avansa maksājumi par nemateriālajiem ieguldījumiem"
"a1191","Accumulated depreciation on portion of development costs","1191","asset_non_current","","False","Attīstības izmaksu norakstītā daļa (pasīvā)"
"a1192","Write-down portion of concessions, patents, licenses, trademarks and similar rights","1192","asset_non_current","","False","Koncesiju, patentu, licenču, preču zīmju un tamlīdzīgu tiesību norakstītā daļa (pasīvā)"
"a1193","Accumulated depreciation on part of the value of other intangible investments","1193","asset_non_current","","False","Citu nemateriālo ieguldījumu vērtības norakstītā daļa (pasīvā)"
"a1210","Land, buildings, perennial plantings","1210","asset_fixed","","False","Nekustamā īpašuma objekti: Zemes gabali, ēkas un būves"
"a1220","Technological equipment and machinery","1220","asset_fixed","","False","Tehnoloģiskās iekārtas un mašīnas"
"a1230","Other fixed assets and inventory","1230","asset_fixed","","False","Pārējie pamatlīdzekļi un inventārs"
"a1240","Costs of fixed asset creation and unfinished object construction","1240","asset_fixed","","False","Pamatlīdzekļu izveidošanas un nepabeigto celtniecības objektu izmaksas"
"a1250","Long-term investments in leased fixed assets","1250","asset_fixed","","False","Ilgtermiņa ieguldījumi nomātajos pamatlīdzekļos"
"a1260","Long-term investments in fixed assets of the public partner","1260","asset_fixed","","False","Ilgtermiņa ieguldījumi publiskā partnera pamatlīdzekļos"
"a1280","Advance payments for fixed assets","1280","asset_prepayments","","False","Avansa maksājumi par pamatlīdzekļiem"
"a1291","Accumulated depreciation for Land, buildings, perennial plantings","1291","asset_fixed","","False","Zemes gabalu, ēku un būvju un ilggadīgo stādījumu nolietojums (pasīvā)"
"a1292","Accumulated depreciation on plant and machinery","1292","asset_fixed","","False","Iekārtu un mašīnu nolietojums (pasīvā)"
"a1293","Accumulated depreciation for Other fixed assets and inventory","1293","asset_fixed","","False","Pārējo pamatlīdzekļu un inventāra nolietojums (pasīvā)"
"a1294","Accumulated depreciation on long-term investments in fixed assets of the public partner","1294","asset_fixed","","False",""
"a1310","Plots of land","1310","asset_fixed","","False","Zemes gabali"
"a1320","Buildings, structures or parts thereof","1320","asset_fixed","","False","Ēkas, būves vai to daļas"
"a1330","Costs of creating investment properties","1330","asset_fixed","","False","Ieguldījuma īpašumu  izveidošanas izmaksas"
"a1390","Accumulated depreciation on investment properties","1390","asset_fixed","","False","Ieguldījuma īpašumu  uzkrātais nolietojums (pasīva konts)"
"a1410","Animals","1410","asset_fixed","","False","Lopi"
"a1420","Plants","1420","asset_fixed","","False","Augi"
"a1430","Working or production animals","1430","asset_fixed","","False","Darba vai produktīvie dzīvnieki"
"a1440","Perennial plantings","1440","asset_fixed","","False","Ilggadīgie stādījumi"
"a1490","Accumulated depreciation of biologival assets","1490","asset_fixed","","False","Bioloģisko aktīvu  uzkrātais nolietojums (pasīva konts)"
"a1510","Participation in the capital of related companies","1510","asset_current","","False","Līdzdalība radniecīgo sabiedrību kapitālā"
"a1520","Loans to related companies","1520","asset_current","","False","Aizdevumi radniecīgajām sabiedrībām"
"a1530","Participation in the capital of associated companies","1530","asset_current","","False","Līdzdalība asociēto sabiedrību kapitālā"
"a1540","Loans to associated companies","1540","asset_current","","False","Aizdevumi asociētajām sabiedrībām"
"a1550","Other securities and investments","1550","asset_current","","False","Pārējie vērtspapīri un ieguldījumi"
"a1560","Other loans and other long-term debtors","1560","asset_current","","False","Pārējie aizdevumi un citi ilgtermiņa debitori"
"a1570","Own shares","1570","asset_current","","False","Pašu akcijas un daļas"
"a1580","Loans to shareholders or members and management","1580","asset_current","","False","Aizdevumi akcionāriem vai dalībniekiem un vadībai"
"a1590","Deferred tax assets","1590","asset_current","","False","Atliktā nodokļa aktīvi"
"a2110","Raw materials, basic materials and auxiliary materials","2110","asset_current","","False","Izejvielas, pamatmateriāli un palīgmateriāli"
"a2120","Unfinished production","2120","asset_current","","False","Nepabeigtie ražojumi"
"a2130","Finished products and goods for sale","2130","asset_current","","False","Gatavie ražojumi un preces pārdošanai"
"a2140","Uncompleted orders","2140","asset_current","","False","Nepabeigtie pasūtījumi"
"a2150","Low-value inventory","2150","asset_fixed","","False","Mazvērtīgais inventārs"
"a2159","Write-down off low-value inventory","2159","asset_fixed","","False","Mazvērtīgā inventāra norakstītā daļa (pasīva konts)"
"a2190","Advance payments for goods","2190","asset_prepayments","","False","Avansa maksājumi par precēm"
"a2210","Animals","2210","asset_current","","False","Lopi"
"a2220","Young and small animals","2220","asset_current","","False","Jaunlopi un sīkie dzīvnieki"
"a2230","Annual plantings","2230","asset_current","","False","Viengadīgie stādījumi"
"a2310","Accounts receivable – trade","2310","asset_receivable","","True","Norēķini ar pircējiem un pasūtītājiem"
"a2318","Settlement of doubtful debtors","2318","asset_receivable","","True","Norēķini ar šaubīgiem debitoriem"
"a2320","Bills of exchange received for deliveries and customers","2320","asset_receivable","","True","Par piegādēm un pasūtījumiem saņemtie vekseļi"
"a2330","Debts of subsidiaries","2330","asset_receivable","","True","Meitas uzņēmumu parādi"
"a2340","Debts of related companies","2340","asset_receivable","","True","Saistīto uzņēmumu parādissssssssss"
"a2350","Settlements with other debtors","2350","asset_receivable","","True","Norēķini ar citiem debitoriem"
"a2360","Settlement of unpaid amounts of the company's capital","2360","asset_receivable","","True","Norēķini par sabiedrības kapitāla neiemaksātām summām"
"a2370","Short-term loans for company shareholder and management","2370","asset_receivable","","True","Īstermiņa aizdevumi sabiedrības dalībniekiem un valdei"
"a2380","Settlement of claims on staff","2380","asset_receivable","","True","Norēķini par prasībām personālam"
"a2388","Accrued revenue","2388","asset_current","","False","Uzkrātie ieņēmumi"
"a2390","Tax overpayments, taxes paid in advance","2390","asset_current","","False","Pārmaksātie nodokļi, iepriekš samaksātie nodokļi"
"a2391","Tax and fee cancellation (tax account)","2391","asset_current","","False","Nodokļu un nodevu dzēšana (nodokļu konts)"
"a2392","Frozen recoverable VAT","2392","asset_current","","False","Iesaldēts atgūstams PVN"
"a2399","VAT charged but not deducted","2399","asset_current","","False","Aprēķināts, bet neatskaitīts PVN"
"a2410","Deferred expenses","2410","asset_current","","False","Nākamo periodu izmaksas"
"a2420","Share issue valuation","2420","asset_current","","False","Akciju emisijas nocenojums"
"a2510","Shares of the subsidiary company","2510","asset_current","","False","Meitas uzņēmuma akcijas un daļas"
"a2520","Shares of related companies","2520","asset_current","","False","Saistīto uzņēmumu akcijas un daļas"
"a2530","Shares of own company","2530","asset_current","","False","Pašu uzņēmuma akcijas un daļas"
"a2540","Other securities and participations in equity","2540","asset_current","","False","Pārējie vērtspapīri un līdzdalība kapitālos"
"a2550","Derivative financial instruments","2550","asset_current","","False","Atvasinātie finanšu instrumenti"
"a2610","Cash","2610","asset_cash","","False","Skaidras naudas kase"
"a2613","ECR (retail) cash register","2613","asset_cash","","False","EKA (mazumtirdzniecības) kase"
"a2620","Bank account","2620","asset_cash","","False","Bankas konts"
"a26291","Bank Suspense Account","26291","asset_current","","False","Bankas pagaidu konts"
"a26292","Outstanding Receipts","26292","asset_current","","False","Nonoslēgtie ieņēmumi"
"a26293","Outstanding Payments","26293","asset_current","","False","Nonoslēgtie izdevumi"
"a2640","Credit notes, checks and special forms of payment accounts","2640","asset_cash","","False","Akreditīvi, čeki un īpašu formu konti"
"a2650","Other bank accounts","2650","asset_cash","","False","Citi konti bankās"
"a2670","Other funds","2670","asset_cash","","False","Pārējie naudas līdzekļi"
"a2671","POS or payment system settlements","2671","asset_cash","","False","POS vai maksājumu sistēmu norēķini"
"a2696","Clearing of payments","2696","asset_current","","False","Maksājumu ieskaite"
"a2698","Currency exchange account","2698","asset_current","","False","Valūtas konvertācijas konts"
"a2699","Money on the move","2699","asset_current","","True","Naudas līdzekļi ceļā"
"a3110","Capital of own shares","3110","equity","","False","Pamatkapitāls vai līdzdalības kapitāls"
"a3111","Unregistered share capital","3111","equity","","False","Nereģistrēts pamatkapitāls"
"a3120","Share issue premium","3120","equity","","False","Akciju emisijas uzcenojums"
"a3130","Revaluation reserve for long-term investments","3130","equity","","False","Ilgtermiņa ieguldījumu pārvērtēšanas rezerve"
"a3140","Revaluation reserve for financial instruments","3140","equity","","False","Finanšu instrumentu pārvērtēšanas rezerve"
"a3210","Funds withdrawn for private purposes","3210","liability_current","","True","Privātiem nolūkiem izņemtie līdzekļi"
"a3220","Private investment","3220","liability_current","","True","Privātie ieguldījumi"
"a3310","Statutory reserves","3310","equity","","False","Likumā noteiktās rezerves"
"a3320","Reserves for own shares or parts","3320","equity","","False","Rezerve pašu akcijām un daļām"
"a3330","Reserves specified in the company's articles of association","3330","equity","","False","Sabiedrības statūtos noteiktās rezerves"
"a3340","Reserves earmarked for development","3340","equity","","False","Rezerves, kas novirzītas attīstībai"
"a3350","Foreign currency conversion reserve","3350","equity","","False","Ārvalstu valūtu pārrēķināšanas rezerve"
"a3360","Other reserves","3360","equity","","False","Pārējās rezerves"
"a3410","Retained earnings for the current year","3410","equity_unaffected","","False","Pārskata gada nesadalīta peļņa"
"a3420","Retained earnings","3420","equity","","False","Iepriekšējo gadu nesadalītā peļņa"
"a4110","Savings for pensions and similar liabilities","4110","liability_non_current","","False","Uzkrājumi pensijām pielīdzinātām saistībām"
"a4210","Provisions for estimated taxes","4210","liability_non_current","","False","Uzkrājumi paredzamiem nodokļiem"
"a4310","Other savings","4310","liability_non_current","","False","Citi uzkrājumi"
"a4319","Provisions for doubtful debtors","4319","liability_non_current","","False","Uzkrājumi šaubīgiem debitoriem"
"a5110","Loans for bonds","5110","liability_non_current","","False","Aizņēmumi par obligācijām"
"a5120","Loans convertible into shares","5120","liability_non_current","","False","Akcijās pārvēršamie aizņēmumi"
"a5130","Loans with profit participation","5130","liability_non_current","","False","Aizņēmumi ar līdzdalību peļņā"
"a5140","Other loans","5140","liability_non_current","","True","Citi aizņēmumi"
"a5141","Loans from shareholders","5141","liability_non_current","","True","Aizņēmumi no dalībniekiem"
"a5150","Short-term loans from credit institutions","5150","liability_current","","False","Īstermiņa aizņēmumi no kredītiestādēm"
"a5160","Long-term loans from credit institutions","5160","liability_non_current","","False","Ilgtermiņa aizņēmumi no kredītiestādēm"
"a5170","Short-term leasing liabilities","5170","liability_non_current","","False","Īstermiņa līzinga saistības"
"a5180","Long-term leasing liabilities","5180","liability_non_current","","False","Ilgtermiņa līzinga saistības"
"a5190","Loans without term","5190","liability_non_current","","False","Aizņēmumi bez norādītā termiņa"
"a5210","Advances received from customers","5210","liability_current","","True","Norēķini par no pircējiem saņemtiem avansiem"
"a5299","VAT calculated and deducted","5299","liability_current","","False","Aprēķināts un atskaitīts PVN"
"a5310","Accounts payable – trade","5310","liability_payable","","True","Tekošie kreditoru parādi no saimnieciskās darbības (tirdzniecības)"
"a5319","Accrued liabilities to creditors","5319","liability_current","","False","Uzkrātas kreditoru saistības"
"a5410","Settlement of self-issued promissory notes","5410","liability_current","","False","Norēķini par pašu izdotajiem vekseļiem"
"a5510","Settlement of debts owed to subsidiaries","5510","liability_payable","","True","Norēķini par parādiem meitas uzņēmumiem"
"a5520","Settlement of debts owed to related companies","5520","liability_current","","False","Norēķini par parādiem saistītajiem uzņēmumiem"
"a5530","Settlement of debts with companies under participation agreements","5530","liability_current","","False","Norēķini par parādiem ar uzņēmumiem, saskaņā ar līgumiem par līdzdalību"
"a5540","Settlement of debts with other companies","5540","liability_current","","False","Norēķini par parādiem ar citiem uzņēmumiem"
"a5550","Settlement of debts with staff","5550","liability_current","","False","Norēķini par parādiem ar personālu"
"a5610","Wage settlements","5610","liability_current","","True","Norēķini par darba algu"
"a5611","Settlement of wage advances","5611","liability_current","","True","Norēķini par darba algas avansiem"
"a5612","Holiday allowance paid","5612","liability_current","","True","Izmaksāta atvaļinājuma nauda"
"a5613","Expenses and benefits included in salary","5613","liability_current","","True","Pie darbas algas pieskaitāmie izdevumi un gūtie labumi"
"a5619","Allowances payable","5619","liability_current","","True","Izmaksājamie pabalsti"
"a5620","Settlement of deductions from wages","5620","liability_current","","True","Norēķini par ieturējumiem no darba algas"
"a5621","Deductions imposed by bailiffs - first-order deductions","5621","liability_current","","True","Tiesas izpildītāju uzliktie ieturējumi - pirmās kārtās ieturējumi"
"a5630","Other settlements with employees","5630","liability_current","","True","Citi norēķini ar darbiniekiem"
"a5650","Settlement of payables to staff","5650","liability_current","","True","Norēķini par parādiem personālam"
"a5710","Settlement of corporate income taxes","5710","liability_current","","False","Norēķini par uzņēmuma ienākuma nodokļi (UIN)"
"a5711","Settlement of foreign corporate income taxes","5711","liability_current","","False","Norēķini par ārvalstu uzņēmuma ienākuma nodokļi"
"a5721","Settlement of value added taxes (VAT)","5721","liability_current","","False","Norēķini par pievienotās vērtības nodokļi (PVN)"
"a5722","Settlement of personal income tax","5722","liability_current","","False","Norēķini par Iedzīvotāju ienākuma nodokļi (IIN)"
"a5723","Settlement of social security contributions (SSI)","5723","liability_current","","False","Norēķini par sociālās apdrošināšanas maksājumiem (VSAOI)"
"a5724","Settlements for business risk state fees","5724","liability_current","","False","Norēķini par Valsts darba dēvēja riska nodevu (VDRN)"
"a5725","Real estate tax settlement","5725","liability_current","","False","Norēķini par nekustamā īpašuma nodokli (NIN)"
"a5726","VAT settlement for OSS goods and services","5726","liability_current","","False","Norēķini par PVN OSS precēm un pakalpojumiem"
"a5727","Natural resource tax (NRT) settlement","5727","liability_current","","False","Norēķini par dabas resursa nodokli (DRN)"
"a5728","Settlement of company car tax","5728","liability_current","","False","Norēķini par uzņēmumu vieglo transportlīdzekļu nodokli (UVTN)"
"a5729","Settlement of import duties","5729","liability_current","","False","Norēķini par Ievedmuitas nodokļi"
"a5810","Settlement of unpaid dividends","5810","liability_current","","False","Norēķini par neizmaksātajam dividendēm"
"a5910","Deferred income","5910","liability_current","","False","Nākamo periodu ieņēmumi"
"a5920","Accrued liabilities","5920","liability_current","","False","Uzkrātās saistības"
"a5930","Derivative financial instruments","5930","liability_current","","False","Atvasinātie finanšu instrumenti"
"a6110","Sales from operating activities subject, VAT 21%","6110","income","","False","Ar PVN 21% pamatlikmi apliekamie ieņēmumi no pamatdarbības"
"a6120","Sales from operating activities subject, VAT reduced rate 12%","6120","income","","False","Ar PVN 12% samazināto likmi apliekamie pārdošanas ieņēmumi no pamatdarbības"
"a6130","Sales from operating activities subject, VAT reduced rate 5%","6130","income","","False","Ar PVN 5% samazināto likmi apliekamie pārdošanas ieņēmumi no pamatdarbības"
"a6140","Sales from operating activities,VAT 0%","6140","income","","False","Ar PVN 0 % likmi apliekamie pārdošanas ieņēmumi no pamatdarbības"
"a6210","Non-taxable sales revenues","6210","income","","False","Ar nodokļiem neapliekamie pārdošanas ieņēmumi"
"a6220","Sales revenue subject to special taxes","6220","income","","False","Ar īpašiem nodokļiem apliekamie pārdošanas ieņēmumi"
"a6310","Commission and brokerage income","6310","income","","False","Komisijas un starpniecības ieņēmumi"
"a6320","Revenues from waste processing and sale","6320","income","","False","Ieņēmumi no atkritumu pārstrādes un realizācijas"
"a6330","Revenues from the sale of containers","6330","income","","False","Ieņēmumi no taras realizācijas"
"a6340","Non-taxable income","6340","income","","False","Ar nodokļiem neapliekamie ieņēmumi"
"a6410","Sconto discounts granted","6410","income","","False","Piešķirtās skonto atlaides"
"a6420","Bonuses granted","6420","income","","False","Piešķirtie bonusi"
"a6430","Granted rebates and other trade discounts","6430","income","","False","Piešķirtie rabati un citas tirdzniecības atlaides"
"a6510","Income from the increase in the price of securities","6510","income_other","","False","Ieņēmumi no vērtspapīru kursa paaugstināšanās"
"a6520","Income from the sale of fixed assets","6520","income_other","","False","Ieņēmumi no pamatlīdzekļu pārdošanas"
"a6530","Income from the rental of plots of land","6530","income_other","","False","Ieņēmumi no zemes gabalu iznomāšanas"
"a6540","Revenues from the sale of current assets","6540","income_other","","False","Ieņēmumi no apgrozāmo līdzekļu pārdošanas"
"a6550","Income from leasing of fixed assets","6550","income_other","","False","Ieņēmumi no pamatlīdzekļu iznomāšanas"
"a6560","Decrease in reserves and provisions transferred to revenue","6560","income_other","","False","Ieņēmumos pārskaitītais  rezervju un uzkrājumu samazinājums"
"a6570","Tax reduction of previous years","6570","income_other","","False","Iepriekšējo gadu nodokļa samazinājums"
"a6580","Additional investments and other income","6580","income_other","","False","Papildus ieguldījumi un citi ieņēmumi"
"a6590","Works executed for own long-term investments","6590","income_other","","False","Pašu ilgtermiņa ieguldījumiem izpildītie darbi"
"a6610","Changes in stocks and value of finished products","6610","income_other","","False","Gatavas produkcijas krājumu un vērtības izmaiņas"
"a6620","Changes in inventories and value of work-in-progress","6620","income_other","","False","Nepabeigto ražojumu vērtības izmaiņas"
"a6630","Changes in backorder balances and value","6630","income_other","","False","Nepabeigto pasūtījumu atlikumu un vērtības izmaiņas"
"a6640","Changes in the herd value of productive and working animals","6640","income_other","","False","Produktīvo un darba dzīvnieku ganāmpulka vērtības izmaiņas"
"a6710","Revenues of other periods relating to the reporting period","6710","income_other","","False","Citu periodu ieņēmumi, kas attiecas uz pārskata periodu"
"a6910","Income from apartment management","6910","income_other","","False","Dzīvokļu saimniecības ieņēmumi"
"a6920","Utility income","6920","income_other","","False","Komunālas saimniecības ieņēmumi"
"a6930","Revenue from domestic services","6930","income_other","","False","Sadzīves pakalpojumu ieņēmumi"
"a6940","Catering revenue","6940","income_other","","False","Sabiedriskās ēdināšanas ieņēmumi"
"a6950","Income of educational institutions","6950","income_other","","False","Izglītības iestāžu ieņēmumi"
"a6960","Medical service revenue","6960","income_other","","False","Medicīniskās apkalpošanas ieņēmumi"
"a6970","Revenues of sports institutions and from culture events","6970","income_other","","False","Kultūras un sporta iestāžu un pasākumu ieņēmumi"
"a6980","Other revenues of social infrastructure institutions and events","6980","income_other","","False","Pārējie sociālās infrastruktūras iestāžu un pasākumu ieņēmumi"
"a7110","Cost of raw materials and delivery expenses","7110","expense","","False","Izejvielu un materiālu iepirkšanas un piegādes izdevumi"
"a7120","Cost of goods sold and delivery expenses","7120","expense","","False","Preču iepirkšanas un piegādes izdevumi"
"a7130","Discounts received","7130","expense","","False","Saņemtās atlaides"
"a7140","Cost of packaging","7140","expense","","False","Taras izdevumi"
"a7150","Customs and import duties","7150","expense","","False","Muitas un ievednodevas"
"a7160","Other external expenses","7160","expense","","False","Pārējie ārējie izdevumi"
"a7170","Cost of external work and services","7170","expense","","False","Samaksa par darbiem un pakalpojumiem no ārienes"
"a7180","Other material expenses (tax on the use of natural resources, stumpage money, etc.)","7180","expense","","False","Pārējie materiālie izdevumi (nodoklis par dabas resursu izmantošanu, celmu nauda u.c.)"
"a7190","Changes in inventory and value of purchased materials and goods","7190","expense","","False","Pirkto materiālu un preču krājumu un vērtības izmaiņas"
"a7210","Employee salaries","7210","expense","","False","Strādnieku algas"
"a7220","Salaries of management and administrative staff","7220","expense","","False","Pārvaldes personāla un administratīvā personāla algas"
"a7230","Salaries of employees of social infrastructure institutions and events","7230","expense","","False","Sociālās infrastruktūras iestāžu un pasākumu darbinieku algas"
"a7240","Other personnel costs","7240","expense","","False","Pārējās personāla izmaksas"
"a7310","Social tax","7310","expense","","False","Sociālais nodoklis"
"a7320","Other social costs","7320","expense","","False","Pārējās sociālās izmaksas"
"a7330","Social costs of social infrastructure facilities and activities","7330","expense","","False","Sociālās infrastruktūras iestāžu un pasākumu sociālās izmaksas"
"a7340","Business risk state fees","7340","expense","","False","Risku nodeva"
"a7410","Write-down off the value of intangible assets","7410","expense","","False","Nemateriālo ieguldījumu vērtības norakstīšana"
"a7420","Depreciation of fixed assets","7420","expense","","False","Pamatlīdzekļu nolietojums"
"a7430","Regulatory stock losses","7430","expense","","False","Krājumu normatīvie zudumi"
"a7440","Accumulated depreciation on the value of current asset","7440","expense","","False","Apgrozāmo līdzekļu vērtības norakstīšana"
"a7450","Over-normal stock losses","7450","expense","","False","Krājumu virsnormatīvie zudumi"
"a7460","Stock-outs","7460","expense","","False","Krājumu iztrūkumi"
"a7470","Asset impairment charge","7470","expense","","False","Izmaksas no aktīvu vērtības samazinājuma"
"a7510","Nature conservation expenses","7510","expense","","False","Dabas aizsardzības izmaksas"
"a7520","Undepreciated value (residual value) of fixed assets excluded","7520","expense","","False","Izslēgto pamatlīdzekļu nenoamortizētā vērtība (atlikusī vērtība)"
"a7530","Fees for plots of land used in production","7530","expense","","False","Nodevas par ražošanā izmantotiem zemes gabaliem"
"a7540","Insurance payments (except employee insurance)","7540","expense","","False","Apdrošināšanas maksājumi (izņemot darbinieku apdrošināšanu)"
"a7550","Other operating expenses","7550","expense","","False","Pārējās saimnieciskās darbības izmaksas"
"a7560","Recruitment and training expenses","7560","expense","","False","Strādnieku vervēšanas un apmācības izdevumi"
"a7570","Business trip expenses","7570","expense","","False","Komandējumu izdevumi"
"a7580","Non-deductible VAT (to be included in costs)","7580","expense","","False","Neatskaitāmais PVN (iekļaujams izmaksās)"
"a7590","Real estate tax for the year under review","7590","expense","","False","Nekustamā īpašuma nodoklis par pārskata gadu"
"a7610","Costs of packaging material and containers","7610","expense","","False","Iesaiņojamais materiāls, tara"
"a7620","Cost of goods transportation","7620","expense","","False","Preču transporta izmaksas"
"a7630","Insurance of goods transportation","7630","expense","","False","Preču transporta apdrošināšana"
"a7640","Commissions paid","7640","expense","","False","Samaksātās komisijas naudas"
"a7650","Other sales expenses","7650","expense","","False","Citas pārdošanas izmaksas"
"a7710","Communication expenses","7710","expense","","False","Sakaru izdevumi"
"a7720","Office expenses","7720","expense","","False","Kantora (biroja) izdevumi"
"a7730","Cost of legal services","7730","expense","","False","Juristu pakalpojumu izmaksas"
"a7740","Annual report and audit expenses","7740","expense","","False","Gada pārskata un revīzijas izdevumi"
"a7750","Expenses of cash circulation","7750","expense","","False","Naudas apgrozījuma blakus izdevumi"
"a7760","Transport expenses for administration","7760","expense","","False","Transporta izdevumi administrācijas vajadzībām"
"a7770","Other management and administration expenses","7770","expense","","False","Citi vadīšanas un administrācijas izdevumi"
"a7780","Representation expenses","7780","expense","","False","Reprezentācijas izmaksas"
"a7810","Expenses of previous periods to be included in the reporting period","7810","expense","","False","Pārskata periodā iekļaujamie iepriekšējo periodu izdevumi"
"a7910","Apartment management expenses","7910","expense","","False","Dzīvokļu saimniecības izdevumi"
"a7920","Utility expenses","7920","expense","","False","Komunālās saimniecības izdevumi"
"a7930","Household service expenses","7930","expense","","False","Sadzīves pakalpojumu izdevumi"
"a7940","Catering expenses","7940","expense","","False","Sabiedriskās ēdināšanas izdevumi"
"a7950","Expenses of educational institutions","7950","expense","","False","Izglītības iestāžu izdevumi"
"a7960","Medical service expenses","7960","expense","","False","Medicīniskās apkalpošanas izdevumi"
"a7970","Expenses of cultural and sports institutions and events","7970","expense","","False","Kultūras un sporta iestāžu un pasākumu izdevumi"
"a7980","Other social infrastructure maintenance expenses","7980","expense","","False","Pārējie sociālās infrastruktūras uzturēšanas izdevumi"
"a8110","Income from participations, securities and loans","8110","income_other","","False","Ieņēmumi no līdzdalības, vērtspapīriem un aizdevumiem"
"a8120","Other income from interest and similar income","8120","income_other","","False","Pārējie ieņēmumi no procentiem un tiem pielīdzināmi ieņēmumi"
"a8130","Bill discount income","8130","income_other","","False","Vekseļu diskonta ieņēmumi"
"a8140","Donations and other similar income","8140","income_other","","False","Ziedojumi un citi tiem pielīdzināmi ieņēmumi"
"a8150","Income from exchange rate appreciation","8150","income_other","","False","Ienākumi no valūtu kursa paaugstināšanās"
"a8160","Income from fines and contractual penalties received","8160","income_other","","False","Saņemtas soda naudas un līgumsodi"
"a8170","Profit from the sale or purchase of foreign currency","8170","income_other","","False","Peļņa no ārzemju valūtas pārdošanas vai pirkšanas"
"a8190","Other income","8190","income_other","","False","Citi ieņēmumi"
"a8199","Money margin revenue","8199","income_other","","False","Naudas starpības ieņēmumi"
"a8210","Write-down of the value of short-term financial investments","8210","expense","","False","Īstermiņa finanšu ieguldījumu vērtības norakstīšana"
"a8220","Interest paid and related expenses","8220","expense","","False","Samaksātie procenti un tiem pielīdzināmie izdevumi"
"a8230","Costs of promissory notes discounts","8230","expense","","False","Vekseļu diskonta izmaksas"
"a8240","Payment of interest on long-term loans","8240","expense","","False","Ilgtermiņa aizdevumu procentu samaksa"
"a8250","Losses from currency depreciation","8250","expense","","False","Zaudējumi no valūtu kursu pazemināšanas"
"a8260","Paid fines and liquidated damages","8260","expense","","False","Samaksātās soda naudas un līgumsodi"
"a8270","Losses from buying or selling foreign currency","8270","expense","","False","Zaudējumi no ārzemju valūtu pirkšanas vai pārdošanas"
"a8290","Other expenses","8290","expense","","False","Citi izdevumi"
"a8299","Money difference losses","8299","expense","","False","Naudas starpības zaudējumi"
"a8310","Extra income","8310","income_other","","False","Ārkārtas ieņēmumi"
"a8311","Money difference income","8311","income_other","","False","Skaidras naudas līdzekļu starpības ienākumi"
"a8410","Extra expenses","8410","expense","","False","Ārkārtas izdevumi"
"a8411","Money variation losses","8411","expense","","False","Skaidras naudas līdzekļu starpības zaudējumi"
"a8610","Profit or loss","8610","expense","","False","Peļņa vai zaudējumi"
"a8810","Corporate income tax","8810","expense","","False","Nodoklis no peļņas(Uzņēmumu ienākuma nodoklis)"
"a8820","Tax on the exploitation of natural resources","8820","expense","","False","Nodoklis no dabas resursu izmantošanas"
"a8840","Other taxes not included in the operating expenses","8840","expense","","False","Citi saimnieciskās darbības izmaksās neiekļauti nodokļi"
"a9910","Balance entry account","9910","asset_current","","False","Atlikumu ievades konts"

```

## File: data\template\account.fiscal.position-lv.csv

```csv
"id","name","auto_apply","vat_required","country_id","sequence","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_lv","Latvia","1","1","base.lv","10","","",""
"fiscal_position_eu","EU Companies","1","1","","20","base.europe","tax_sale_vat_21","tax_sale_vat_0_eu_g"
"","","","","","","","tax_sale_vat_reverse","tax_sale_vat_0_eu_g"
"","","","","","","","tax_sale_vat_12","tax_sale_vat_0_eu_g"
"","","","","","","","tax_sale_vat_5","tax_sale_vat_0_eu_g"
"","","","","","","","tax_purchase_vat_21","tax_purchase_vat_21_eu"
"","","","","","","","tax_purchase_vat_12","tax_purchase_vat_12_eu"
"","","","","","","","tax_purchase_vat_5","tax_purchase_vat_5_eu"
"fiscal_position_eu_private","EU Consumer","1","","","30","base.europe","",""
"fiscal_position_ex","Outside EU","1","","","40","","tax_sale_vat_5","tax_sale_vat_0_ex_g"
"","","","","","","","tax_sale_vat_12","tax_sale_vat_0_ex_g"
"","","","","","","","tax_sale_vat_reverse","tax_sale_vat_0_ex_g"
"","","","","","","","tax_sale_vat_21","tax_sale_vat_0_ex_g"
"","","","","","","","tax_purchase_vat_21","tax_purchase_vat_21_ex_g"
"","","","","","","","tax_purchase_vat_12","tax_purchase_vat_21_ex_g"
"","","","","","","","tax_purchase_vat_5","tax_purchase_vat_21_ex_g"

```

## File: data\template\account.group-lv.csv

```csv
"id","code_prefix_start","name","name@lv"
"l10n_lv_group_1","1","Long-term investments","Ilgtermiņa ieguldījumi"
"l10n_lv_group_11","11","Intangible assets","Nemateriālie ieguldījumi"
"l10n_lv_group_12","12","Fixed assets (property, plant and equipment, investment property and biological assets)","Pamatlīdzekļi (pamatlīdzekļi, ieguldījuma īpašumi un bioloģiskie aktīvi)"
"l10n_lv_group_13","13","Investment properties","Ieguldījuma īpašumi"
"l10n_lv_group_14","14","Biological assets (Animals and plants)","Bioloģiskie aktīvi (Dzīvnieki un augi)"
"l10n_lv_group_15","15","Long-term financial investments","Ilgtermiņa finanšu ieguldījumi"
"l10n_lv_group_2","2","Current assets","Apgrozāmie līdzekļi"
"l10n_lv_group_21","21","Inventory","Krājumi"
"l10n_lv_group_22","22","Animals and plants","Dzīvnieki un augi"
"l10n_lv_group_23","23","Settlements with debtors","Norēķini par prasībām (ar debitoriem)"
"l10n_lv_group_24","24","Deferred expenses","Nākamo periodu izmaksas"
"l10n_lv_group_25","25","Short-term financial investments","Īstermiņa finanšu ieguldījumi"
"l10n_lv_group_26","26","Money","Naudas līdzekļi"
"l10n_lv_group_28","28","Long-term investments held for sale","Pārdošanai turēti ilgtermiņa ieguldījumi"
"l10n_lv_group_3","3","Equity","Pašu kapitāls"
"l10n_lv_group_31","31","Share capital or participation capital, revaluation reserve for long-term investments","Pamatkapitāls vai līdzdalības kapitāls, ilgtermiņa ieguldījumu pārvērtēšanas rezerve"
"l10n_lv_group_32","32","Private accounts (private investment and consumption of money)","Privātkonti (privātie ieguldījumi un naudas līdzekļu patēriņš)"
"l10n_lv_group_33","33","Reserves","Rezerves"
"l10n_lv_group_34","34","Profit or loss","Peļņa vai zaudējumi"
"l10n_lv_group_4","4","Accruals","Uzkrājumi"
"l10n_lv_group_41","41","Provisions for pensions and similar liabilities","Uzkrājumi pensijām un tamlīdzīgām saistībām"
"l10n_lv_group_42","42","Provisions for expected taxes",""
"l10n_lv_group_43","43","Other provisions",""
"l10n_lv_group_5","5","Creditors","Kreditori"
"l10n_lv_group_51","51","Settlement of loans","Norēķini par aizņēmumiem"
"l10n_lv_group_52","52","Settlement of advances received from customers","Norēķini par saņemtajiem avansiem no pircējiem"
"l10n_lv_group_53","53","Settlements with suppliers and contractors","Norēķini ar piegādātājiem un darbuzņēmējiem"
"l10n_lv_group_54","54","Promissory notes payable","Maksājamie vekseļi"
"l10n_lv_group_55","55","Settlements with companies, members and staff","Norēķini ar uzņēmumiem, dalībniekiem un personālu"
"l10n_lv_group_56","56","Settlement of wages and deductions (excluding taxes)","Norēķini par darba samaksu un ieturējumiem (izņemot nodokļus)"
"l10n_lv_group_57","57","Tax settlement","Norēķini par nodokļiem"
"l10n_lv_group_58","58","Dividends settlement","Norēķini par dividendēm"
"l10n_lv_group_59","59","Deferred income","Nākamo periodu ieņēmumi"
"l10n_lv_group_6","6","Revenue from business activities","Ieņēmumi no uzņēmuma saimnieciskās darbības"
"l10n_lv_group_61","61","Business income subject to general taxation","Saimnieciskās darbības ieņēmumi, kas apliekami ar nodokļiem vispārējā kārtībā"
"l10n_lv_group_62","62","Revenue from sales subject to reduced taxation","Ieņēmumi no pārdošanas, apliekami ar reducētiem (samazinātiem) nodokļiem"
"l10n_lv_group_63","63","Commission, brokerage and other income","Komisijas, starpniecības un citi ieņēmumi"
"l10n_lv_group_64","64","Revenue-reducing discounts","Ieņēmumus samazinošas atlaides"
"l10n_lv_group_65","65","Other operating revenue","Pārējie saimnieciskās darbības ieņēmumi"
"l10n_lv_group_66","66","Changes in stocks and value of finished products and work in progress","Gatavās produkcijas un nepabeigto ražojumu krājumu un vērtības izmaiņas"
"l10n_lv_group_67","67","Revenues of other periods relating to the reporting period","Citu periodu ieņēmumi, kas attiecas uz pārskata periodu"
"l10n_lv_group_69","69","Revenue from social infrastructure","Ieņēmumi no sociālās infrastruktūras objektiem"
"l10n_lv_group_7","7","Cost of economic activity",""
"l10n_lv_group_71","71","Costs of raw materials, supplies and goods","Izejvielu, materiālu un preču iepirkšanai izmaksas"
"l10n_lv_group_72","72","Staff costs","Personāla izmaksas"
"l10n_lv_group_73","73","Social costs and charges","Sociālās izmaksas un nodevas"
"l10n_lv_group_74","74","Depreciation of property, plant and equipment and write-downs of other investments","Pamatlīdzekļu nolietojums un citu ieguldījumu vērtības norakstījumi"
"l10n_lv_group_75","75","Other operating expenses","Pārējās saimnieciskās darbības izmaksas"
"l10n_lv_group_76","76","Cost of goods sold","Preču pārdošanas izmaksas"
"l10n_lv_group_77","77","Administration costs","Administrācijas izmaksas"
"l10n_lv_group_78","78","Prior period costs to be included in the reporting period","Pārskata periodā iekļaujamās iepriekšējo periodu izmaksas"
"l10n_lv_group_79","79","Social infrastructure maintenance costs","Sociālās infrastruktūras uzturēšanas izmaksas"
"l10n_lv_group_8","8","Miscellaneous income and expenses, profit and loss","Dažādi ieņēmumi un izmaksas, peļņa un zaudējumi"
"l10n_lv_group_81","81","Miscellaneous revenues","Dažādi ieņēmumi"
"l10n_lv_group_82","82","Different costs","Dažādas izmaksas"
"l10n_lv_group_83","83","Extra income","Ārkārtas ieņēmumi"
"l10n_lv_group_84","84","Extraordinary costs","Ārkārtas izmaksas"
"l10n_lv_group_86","86","Profit or loss","Peļņa vai zaudējumi"
"l10n_lv_group_88","88","Taxes on profits and other taxes not included in operating expenses","Nodokļi no peļņas un citi saimnieciskās darbības izdevumos neiekļaujamie nodokļi"
"l10n_lv_group_9","9","Additional work accounts","Papildus darba konti"

```

## File: data\template\account.tax-lv.csv

```csv
"id","description","name","type_tax_use","amount","sequence","tax_group_id","amount_type","price_include","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","name@lv"
"tax_sale_vat_21","21%","21%","sale","21.0","100","tax_group_vat_21","percent","","100","base","invoice","+41","","21%"
"","","","","","","","","","100","tax","invoice","+52","a5721",""
"","","","","","","","","","100","base","refund","-41","",""
"","","","","","","","","","100","tax","refund","-52","a5721",""
"tax_sale_vat_reverse","0%","Reverse charge VAT","sale","0.0","110","tax_group_vat_exempt","percent","False","100","base","invoice","+411","","Pārdošana reversa"
"","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","100","base","refund","-411","",""
"","","","","","","","","","100","tax","refund","","",""
"tax_sale_vat_12","12%","12%","sale","12.0","120","tax_group_vat_12","percent","False","100","base","invoice","+42","","12%"
"","","","","","","","","","100","tax","invoice","+53","a5721",""
"","","","","","","","","","100","base","refund","-42","",""
"","","","","","","","","","100","tax","refund","-53","a5721",""
"tax_sale_vat_5","5%","5%","sale","5.0","130","tax_group_vat_5","percent","False","100","base","invoice","+421","","5%"
"","","","","","","","","","100","tax","invoice","+531","a5721",""
"","","","","","","","","","100","base","refund","-421","",""
"","","","","","","","","","100","tax","refund","-531","a5721",""
"tax_sale_vat_0_s","0%","0% S","sale","0.0","140","tax_group_vat_0","percent","False","100","base","invoice","+48","","0% Pakalpojumi"
"","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","100","base","refund","-48","",""
"","","","","","","","","","100","tax","refund","","",""
"tax_sale_vat_0_ex_g","0%","0% EX G","sale","0.0","150","tax_group_vat_0","percent","False","100","base","invoice","+481","","0% EX Preces"
"","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","100","base","refund","-481","",""
"","","","","","","","","","100","tax","refund","","",""
"tax_sale_vat_0_ex_s","0%","0% EX S","sale","0.0","160","tax_group_vat_exempt","percent","False","100","base","invoice","+482","","0% EX Pakalpojumi"
"","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","100","base","refund","-482","",""
"","","","","","","","","","100","tax","refund","","",""
"tax_sale_vat_0_eu_s","0%","0% EU S","sale","0.0","170","tax_group_vat_exempt","percent","False","100","base","invoice","+482","","0% ES Pakalpojumi"
"","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","100","base","refund","-482","",""
"","","","","","","","","","100","tax","refund","","",""
"tax_sale_vat_0_eu_g","0%","0% EU G","sale","0.0","180","tax_group_vat_0","percent","False","100","base","invoice","+45","","0% EU Preces"
"","","","","","","","","","100","tax","invoice","","",""
"","","","","","","","","","100","base","refund","-45","",""
"","","","","","","","","","100","tax","refund","","",""
"tax_purchase_vat_21","21%","21%","purchase","21.0","400","tax_group_vat_21","percent","False","100","base","invoice","","","21%"
"","","","","","","","","","100","tax","invoice","+62","a5721",""
"","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","100","tax","refund","-62","a5721",""
"tax_purchase_vat_21_eu","21%","21% EU","purchase","21.0","410","tax_group_vat_21","percent","False","100","base","invoice","+50","","21% ES"
"","","","","","","","","","100","tax","invoice","+55","a5721",""
"","","","","","","","","","-100","tax","invoice","-64","a5721",""
"","","","","","","","","","100","base","refund","-50","",""
"","","","","","","","","","100","tax","refund","-55","a5721",""
"","","","","","","","","","-100","tax","refund","+64","a5721",""
"tax_purchase_vat_12","12%","12%","purchase","12.0","420","tax_group_vat_12","percent","False","100","base","invoice","","","12%"
"","","","","","","","","","100","tax","invoice","+62","a5721",""
"","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","100","tax","refund","-62","a5721",""
"tax_purchase_vat_12_eu","12%","12% EU","purchase","12.0","430","tax_group_vat_12","percent","False","100","base","invoice","+51","","12% ES"
"","","","","","","","","","100","tax","invoice","+56","a5721",""
"","","","","","","","","","-100","tax","invoice","-64","a5721",""
"","","","","","","","","","100","base","refund","-51","",""
"","","","","","","","","","100","tax","refund","-56","a5721",""
"","","","","","","","","","-100","tax","refund","+64","a5721",""
"tax_purchase_vat_5","5%","5%","purchase","5.0","440","tax_group_vat_5","percent","False","100","base","invoice","","","5%"
"","","","","","","","","","100","tax","invoice","+62","a5721",""
"","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","100","tax","refund","-62","a5721",""
"tax_purchase_vat_5_eu","5%","5% EU","purchase","5.0","450","tax_group_vat_5","percent","False","100","base","invoice","+511","","5% ES"
"","","","","","","","","","100","tax","invoice","+561","a5721",""
"","","","","","","","","","-100","tax","invoice","-64","a5721",""
"","","","","","","","","","100","base","refund","-511","",""
"","","","","","","","","","100","tax","refund","-561","a5721",""
"","","","","","","","","","-100","tax","refund","+64","a5721",""
"tax_purchase_vat_21_reverse","21%","21% reverse charge","purchase","21.0","460","tax_group_vat_21","percent","False","100","base","invoice","","","21% Pārdošana reversa"
"","","","","","","","","","100","tax","invoice","+52","a5721",""
"","","","","","","","","","-100","tax","invoice","-62","a5721",""
"","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","100","tax","refund","-52","a5721",""
"","","","","","","","","","-100","tax","refund","+62","a5721",""
"tax_purchase_vat_21_ex_g","21%","21% EX G","purchase","21.0","470","tax_group_vat_21","percent","False","100","base","invoice","","",""
"","","","","","","","","","100","tax","invoice","+52","a5721",""
"","","","","","","","","","-100","tax","invoice","-63","a5721",""
"","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","100","tax","refund","-52","a5721",""
"","","","","","","","","","-100","tax","refund","+63","a5721",""
"tax_purchase_vat_21_ex_s","21%","21% EX S","purchase","21.0","480","tax_group_vat_21","percent","False","100","base","invoice","","","21% EX Pakalpojumi"
"","","","","","","","","","100","tax","invoice","+54","a5721",""
"","","","","","","","","","-100","tax","invoice","-63","a5721",""
"","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","100","tax","refund","-54","a5721",""
"","","","","","","","","","-100","tax","refund","+63","a5721",""

```

## File: data\template\account.tax.group-lv.csv

```csv
"id","name","country_id","name@lv"
"tax_group_vat_0","VAT 0%","base.lv","PVN 0%"
"tax_group_vat_exempt","W/O VAT","base.lv","BEZ PVN"
"tax_group_vat_5","VAT 5%","base.lv","PVN 5%"
"tax_group_vat_12","VAT 12%","base.lv","PVN 12%"
"tax_group_vat_21","VAT 21%","base.lv",""

```

## File: models\template_lv.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('lv')
    def _get_lv_template_data(self):
        return {
            'property_account_receivable_id': 'a2310',
            'property_account_payable_id': 'a5310',
            'property_account_expense_categ_id': 'a7550',
            'property_account_income_categ_id': 'a6110',
            'code_digits': '4',
        }

    @template('lv', 'res.company')
    def _get_lv_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.lv',
                'bank_account_code_prefix': '2620',
                'cash_account_code_prefix': '2610',
                'transfer_account_code_prefix': '2699',
                'account_default_pos_receivable_account_id': 'a2613',
                'income_currency_exchange_account_id': 'a8150',
                'expense_currency_exchange_account_id': 'a8250',
                'account_journal_suspense_account_id': 'a26291',
                'account_journal_early_pay_discount_loss_account_id': 'a8299',
                'account_journal_early_pay_discount_gain_account_id': 'a8199',
                'account_journal_payment_debit_account_id': 'a26292',
                'account_journal_payment_credit_account_id': 'a26293',
                'default_cash_difference_income_account_id': 'a8199',
                'default_cash_difference_expense_account_id': 'a8299',
                'account_sale_tax_id': 'tax_sale_vat_21',
                'account_purchase_tax_id': 'tax_purchase_vat_21',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_lv

```

