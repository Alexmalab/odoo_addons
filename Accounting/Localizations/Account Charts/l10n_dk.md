# Odoo Module: l10n_dk

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Denmark - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['dk'],
    'version': '1.3',
    'author': 'Odoo House ApS, VK DATA ApS, FlexERP ApS',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations.html',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """

Localization Module for Denmark
===============================

This is the module to manage the **accounting chart for Denmark**. Cover both one-man business as well as I/S, IVS, ApS and A/S

**Modulet opsætter:**

- **Dansk kontoplan**

- Dansk moms
        - 25% moms
        - Resturationsmoms 6,25%
        - Omvendt betalingspligt

- Konteringsgrupper
        - EU (Virksomhed)
        - EU (Privat)
        - 3.lande

- Finans raporter
        - Resulttopgørelse
        - Balance
        - Momsafregning
            - Afregning
            - Rubrik A, B og C

- **Anglo-Saxon regnskabsmetode**

.

Produkt setup:
==============

**Vare**

**Salgsmoms:**      Salgmoms 25%

**Salgskonto:**     1010 Salg af vare, m/moms

**Købsmoms:**       Købsmoms 25%

**Købskonto:**      2010 Direkte omkostninger vare, m/moms

.

**Ydelse**

**Salgsmoms:**      Salgmoms 25%, ydelser

**Salgskonto:**     1011 Salg af ydelser, m/moms

**Købsmoms:**       Købsmoms 25%, ydelser

**Købskonto:**      2011 Direkte omkostninger ydelser, m/moms

.

**Vare med omvendt betalingspligt**

**Salgsmoms:**      Salg omvendt betalingspligt

**Salgskonto:**     1012 Salg af vare, u/moms

**Købsmoms:**       Køb omvendt betalingspligt

**Købskonto:**      2012 Direkte omkostninger vare, u/moms


.

**Restauration**

**Købsmoms:**       Restaurationsmoms 6,25%, købsmoms

**Købskonto:**      4010 Restaurationsbesøg

.

    """,
    'depends': [
        'base_iban',
        'base_vat',
        'account',
    ],
    'auto_install': ['account'],
    'data': [
        'data/account_tax_report_data.xml',
        'data/account.account.tag.csv',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.tag.csv

```csv
"id","name","applicability","country_id/id"
"account_tag_1010","1010","accounts","base.dk"
"account_tag_1050","1050","accounts","base.dk"
"account_tag_1100","1100","accounts","base.dk"
"account_tag_1150","1150","accounts","base.dk"
"account_tag_1200","1200","accounts","base.dk"
"account_tag_1300","1300","accounts","base.dk"
"account_tag_1350","1350","accounts","base.dk"
"account_tag_1410","1410","accounts","base.dk"
"account_tag_1430","1430","accounts","base.dk"
"account_tag_1460","1460","accounts","base.dk"
"account_tag_1510","1510","accounts","base.dk"
"account_tag_1530","1530","accounts","base.dk"
"account_tag_1540","1540","accounts","base.dk"
"account_tag_1550","1550","accounts","base.dk"
"account_tag_1610","1610","accounts","base.dk"
"account_tag_1630","1630","accounts","base.dk"
"account_tag_1650","1650","accounts","base.dk"
"account_tag_1660","1660","accounts","base.dk"
"account_tag_1710","1710","accounts","base.dk"
"account_tag_1740","1740","accounts","base.dk"
"account_tag_1770","1770","accounts","base.dk"
"account_tag_1800","1800","accounts","base.dk"
"account_tag_1830","1830","accounts","base.dk"
"account_tag_1850","1850","accounts","base.dk"
"account_tag_1870","1870","accounts","base.dk"
"account_tag_1890","1890","accounts","base.dk"
"account_tag_1910","1910","accounts","base.dk"
"account_tag_1930","1930","accounts","base.dk"
"account_tag_1950","1950","accounts","base.dk"
"account_tag_1970","1970","accounts","base.dk"
"account_tag_1990","1990","accounts","base.dk"
"account_tag_2030","2030","accounts","base.dk"
"account_tag_2050","2050","accounts","base.dk"
"account_tag_2060","2060","accounts","base.dk"
"account_tag_2070","2070","accounts","base.dk"
"account_tag_2080","2080","accounts","base.dk"
"account_tag_2090","2090","accounts","base.dk"
"account_tag_2100","2100","accounts","base.dk"
"account_tag_2110","2110","accounts","base.dk"
"account_tag_2120","2120","accounts","base.dk"
"account_tag_2130","2130","accounts","base.dk"
"account_tag_2140","2140","accounts","base.dk"
"account_tag_2150","2150","accounts","base.dk"
"account_tag_2160","2160","accounts","base.dk"
"account_tag_2170","2170","accounts","base.dk"
"account_tag_2180","2180","accounts","base.dk"
"account_tag_2190","2190","accounts","base.dk"
"account_tag_2200","2200","accounts","base.dk"
"account_tag_2230","2230","accounts","base.dk"
"account_tag_2240","2240","accounts","base.dk"
"account_tag_2250","2250","accounts","base.dk"
"account_tag_2260","2260","accounts","base.dk"
"account_tag_2270","2270","accounts","base.dk"
"account_tag_2280","2280","accounts","base.dk"
"account_tag_2290","2290","accounts","base.dk"
"account_tag_2300","2300","accounts","base.dk"
"account_tag_2310","2310","accounts","base.dk"
"account_tag_2330","2330","accounts","base.dk"
"account_tag_2350","2350","accounts","base.dk"
"account_tag_2370","2370","accounts","base.dk"
"account_tag_2380","2380","accounts","base.dk"
"account_tag_2390","2390","accounts","base.dk"
"account_tag_2410","2410","accounts","base.dk"
"account_tag_2420","2420","accounts","base.dk"
"account_tag_2450","2450","accounts","base.dk"
"account_tag_2460","2460","accounts","base.dk"
"account_tag_2470","2470","accounts","base.dk"
"account_tag_2480","2480","accounts","base.dk"
"account_tag_2510","2510","accounts","base.dk"
"account_tag_2520","2520","accounts","base.dk"
"account_tag_2530","2530","accounts","base.dk"
"account_tag_2540","2540","accounts","base.dk"
"account_tag_2560","2560","accounts","base.dk"
"account_tag_2620","2620","accounts","base.dk"
"account_tag_2630","2630","accounts","base.dk"
"account_tag_2640","2640","accounts","base.dk"
"account_tag_2650","2650","accounts","base.dk"
"account_tag_2660","2660","accounts","base.dk"
"account_tag_2670","2670","accounts","base.dk"
"account_tag_2680","2680","accounts","base.dk"
"account_tag_2690","2690","accounts","base.dk"
"account_tag_2700","2700","accounts","base.dk"
"account_tag_2710","2710","accounts","base.dk"
"account_tag_2720","2720","accounts","base.dk"
"account_tag_2810","2810","accounts","base.dk"
"account_tag_2850","2850","accounts","base.dk"
"account_tag_2860","2860","accounts","base.dk"
"account_tag_2870","2870","accounts","base.dk"
"account_tag_2880","2880","accounts","base.dk"
"account_tag_2890","2890","accounts","base.dk"
"account_tag_2900","2900","accounts","base.dk"
"account_tag_2910","2910","accounts","base.dk"
"account_tag_2920","2920","accounts","base.dk"
"account_tag_2930","2930","accounts","base.dk"
"account_tag_2940","2940","accounts","base.dk"
"account_tag_2950","2950","accounts","base.dk"
"account_tag_2960","2960","accounts","base.dk"
"account_tag_2965","2965","accounts","base.dk"
"account_tag_2968","2968","accounts","base.dk"
"account_tag_2970","2970","accounts","base.dk"
"account_tag_2980","2980","accounts","base.dk"
"account_tag_3000","3000","accounts","base.dk"
"account_tag_3010","3010","accounts","base.dk"
"account_tag_3020","3020","accounts","base.dk"
"account_tag_3030","3030","accounts","base.dk"
"account_tag_3040","3040","accounts","base.dk"
"account_tag_3050","3050","accounts","base.dk"
"account_tag_3060","3060","accounts","base.dk"
"account_tag_3070","3070","accounts","base.dk"
"account_tag_3080","3080","accounts","base.dk"
"account_tag_3090","3090","accounts","base.dk"
"account_tag_3130","3130","accounts","base.dk"
"account_tag_3160","3160","accounts","base.dk"
"account_tag_3170","3170","accounts","base.dk"
"account_tag_3180","3180","accounts","base.dk"
"account_tag_3200","3200","accounts","base.dk"
"account_tag_3230","3230","accounts","base.dk"
"account_tag_3380","3380","accounts","base.dk"
"account_tag_3400","3400","accounts","base.dk"
"account_tag_3440","3440","accounts","base.dk"
"account_tag_3470","3470","accounts","base.dk"
"account_tag_3490","3490","accounts","base.dk"
"account_tag_3510","3510","accounts","base.dk"
"account_tag_3530","3530","accounts","base.dk"
"account_tag_3560","3560","accounts","base.dk"
"account_tag_3590","3590","accounts","base.dk"
"account_tag_3610","3610","accounts","base.dk"
"account_tag_3620","3620","accounts","base.dk"
"account_tag_3630","3630","accounts","base.dk"
"account_tag_3640","3640","accounts","base.dk"
"account_tag_3650","3650","accounts","base.dk"
"account_tag_3670","3670","accounts","base.dk"
"account_tag_3675","3675","accounts","base.dk"
"account_tag_3680","3680","accounts","base.dk"
"account_tag_3690","3690","accounts","base.dk"
"account_tag_3740","3740","accounts","base.dk"
"account_tag_3760","3760","accounts","base.dk"
"account_tag_3780","3780","accounts","base.dk"
"account_tag_3810","3810","accounts","base.dk"
"account_tag_5010","5010","accounts","base.dk"
"account_tag_5020","5020","accounts","base.dk"
"account_tag_5030","5030","accounts","base.dk"
"account_tag_5040","5040","accounts","base.dk"
"account_tag_5050","5050","accounts","base.dk"
"account_tag_5060","5060","accounts","base.dk"
"account_tag_5080","5080","accounts","base.dk"
"account_tag_5090","5090","accounts","base.dk"
"account_tag_5100","5100","accounts","base.dk"
"account_tag_5110","5110","accounts","base.dk"
"account_tag_5120","5120","accounts","base.dk"
"account_tag_5130","5130","accounts","base.dk"
"account_tag_5160","5160","accounts","base.dk"
"account_tag_5170","5170","accounts","base.dk"
"account_tag_5180","5180","accounts","base.dk"
"account_tag_5190","5190","accounts","base.dk"
"account_tag_5200","5200","accounts","base.dk"
"account_tag_5210","5210","accounts","base.dk"
"account_tag_5220","5220","accounts","base.dk"
"account_tag_5240","5240","accounts","base.dk"
"account_tag_5250","5250","accounts","base.dk"
"account_tag_5260","5260","accounts","base.dk"
"account_tag_5270","5270","accounts","base.dk"
"account_tag_5280","5280","accounts","base.dk"
"account_tag_5290","5290","accounts","base.dk"
"account_tag_5300","5300","accounts","base.dk"
"account_tag_5320","5320","accounts","base.dk"
"account_tag_5330","5330","accounts","base.dk"
"account_tag_5340","5340","accounts","base.dk"
"account_tag_5350","5350","accounts","base.dk"
"account_tag_5370","5370","accounts","base.dk"
"account_tag_5390","5390","accounts","base.dk"
"account_tag_5400","5400","accounts","base.dk"
"account_tag_5420","5420","accounts","base.dk"
"account_tag_5430","5430","accounts","base.dk"
"account_tag_5440","5440","accounts","base.dk"
"account_tag_5450","5450","accounts","base.dk"
"account_tag_5470","5470","accounts","base.dk"
"account_tag_5480","5480","accounts","base.dk"
"account_tag_5500","5500","accounts","base.dk"
"account_tag_5510","5510","accounts","base.dk"
"account_tag_5520","5520","accounts","base.dk"
"account_tag_5530","5530","accounts","base.dk"
"account_tag_5540","5540","accounts","base.dk"
"account_tag_5550","5550","accounts","base.dk"
"account_tag_5570","5570","accounts","base.dk"
"account_tag_5580","5580","accounts","base.dk"
"account_tag_5590","5590","accounts","base.dk"
"account_tag_5600","5600","accounts","base.dk"
"account_tag_5610","5610","accounts","base.dk"
"account_tag_5620","5620","accounts","base.dk"
"account_tag_5640","5640","accounts","base.dk"
"account_tag_5650","5650","accounts","base.dk"
"account_tag_5660","5660","accounts","base.dk"
"account_tag_5670","5670","accounts","base.dk"
"account_tag_5680","5680","accounts","base.dk"
"account_tag_5690","5690","accounts","base.dk"
"account_tag_5710","5710","accounts","base.dk"
"account_tag_5720","5720","accounts","base.dk"
"account_tag_5730","5730","accounts","base.dk"
"account_tag_5740","5740","accounts","base.dk"
"account_tag_5750","5750","accounts","base.dk"
"account_tag_5760","5760","accounts","base.dk"
"account_tag_5800","5800","accounts","base.dk"
"account_tag_5810","5810","accounts","base.dk"
"account_tag_5820","5820","accounts","base.dk"
"account_tag_5830","5830","accounts","base.dk"
"account_tag_5840","5840","accounts","base.dk"
"account_tag_5850","5850","accounts","base.dk"
"account_tag_5870","5870","accounts","base.dk"
"account_tag_5880","5880","accounts","base.dk"
"account_tag_5900","5900","accounts","base.dk"
"account_tag_5910","5910","accounts","base.dk"
"account_tag_5920","5920","accounts","base.dk"
"account_tag_5930","5930","accounts","base.dk"
"account_tag_5940","5940","accounts","base.dk"
"account_tag_5950","5950","accounts","base.dk"
"account_tag_5970","5970","accounts","base.dk"
"account_tag_5980","5980","accounts","base.dk"
"account_tag_6000","6000","accounts","base.dk"
"account_tag_6020","6020","accounts","base.dk"
"account_tag_6030","6030","accounts","base.dk"
"account_tag_6040","6040","accounts","base.dk"
"account_tag_6060","6060","accounts","base.dk"
"account_tag_6080","6080","accounts","base.dk"
"account_tag_6090","6090","accounts","base.dk"
"account_tag_6110","6110","accounts","base.dk"
"account_tag_6120","6120","accounts","base.dk"
"account_tag_6140","6140","accounts","base.dk"
"account_tag_6150","6150","accounts","base.dk"
"account_tag_6170","6170","accounts","base.dk"
"account_tag_6190","6190","accounts","base.dk"
"account_tag_6200","6200","accounts","base.dk"
"account_tag_6220","6220","accounts","base.dk"
"account_tag_6230","6230","accounts","base.dk"
"account_tag_6240","6240","accounts","base.dk"
"account_tag_6250","6250","accounts","base.dk"
"account_tag_6270","6270","accounts","base.dk"
"account_tag_6290","6290","accounts","base.dk"
"account_tag_6300","6300","accounts","base.dk"
"account_tag_6310","6310","accounts","base.dk"
"account_tag_6320","6320","accounts","base.dk"
"account_tag_6330","6330","accounts","base.dk"
"account_tag_6350","6350","accounts","base.dk"
"account_tag_6370","6370","accounts","base.dk"
"account_tag_6390","6390","accounts","base.dk"
"account_tag_6400","6400","accounts","base.dk"
"account_tag_6420","6420","accounts","base.dk"
"account_tag_6450","6450","accounts","base.dk"
"account_tag_6471","6471","accounts","base.dk"
"account_tag_6481","6481","accounts","base.dk"
"account_tag_6482","6482","accounts","base.dk"
"account_tag_6483","6483","accounts","base.dk"
"account_tag_6484","6484","accounts","base.dk"
"account_tag_6510","6510","accounts","base.dk"
"account_tag_6520","6520","accounts","base.dk"
"account_tag_6540","6540","accounts","base.dk"
"account_tag_6560","6560","accounts","base.dk"
"account_tag_6580","6580","accounts","base.dk"
"account_tag_6810","6810","accounts","base.dk"
"account_tag_6830","6830","accounts","base.dk"
"account_tag_6831","6831","accounts","base.dk"
"account_tag_6870","6870","accounts","base.dk"
"account_tag_6890","6890","accounts","base.dk"
"account_tag_6910","6910","accounts","base.dk"
"account_tag_6940","6940","accounts","base.dk"
"account_tag_6960","6960","accounts","base.dk"
"account_tag_7010","7010","accounts","base.dk"
"account_tag_7020","7020","accounts","base.dk"
"account_tag_7040","7040","accounts","base.dk"
"account_tag_7110","7110","accounts","base.dk"
"account_tag_7120","7120","accounts","base.dk"
"account_tag_7160","7160","accounts","base.dk"
"account_tag_7170","7170","accounts","base.dk"
"account_tag_7190","7190","accounts","base.dk"
"account_tag_7210","7210","accounts","base.dk"
"account_tag_7230","7230","accounts","base.dk"
"account_tag_7240","7240","accounts","base.dk"
"account_tag_7250","7250","accounts","base.dk"
"account_tag_7310","7310","accounts","base.dk"
"account_tag_7330","7330","accounts","base.dk"
"account_tag_7350","7350","accounts","base.dk"
"account_tag_7410","7410","accounts","base.dk"
"account_tag_7440","7440","accounts","base.dk"
"account_tag_7510","7510","accounts","base.dk"
"account_tag_7520","7520","accounts","base.dk"
"account_tag_7590","7590","accounts","base.dk"
"account_tag_7610","7610","accounts","base.dk"
"account_tag_7630","7630","accounts","base.dk"
"account_tag_7680","7680","accounts","base.dk"
"account_tag_7700","7700","accounts","base.dk"
"account_tag_7720","7720","accounts","base.dk"
"account_tag_7740","7740","accounts","base.dk"
"account_tag_7760","7760","accounts","base.dk"
"account_tag_7780","7780","accounts","base.dk"
"account_tag_7800","7800","accounts","base.dk"
"account_tag_7810","7810","accounts","base.dk"
"account_tag_7820","7820","accounts","base.dk"
"account_tag_7830","7830","accounts","base.dk"
"account_tag_7840","7840","accounts","base.dk"
"account_tag_7860","7860","accounts","base.dk"
"account_tag_7880","7880","accounts","base.dk"
"account_tag_7900","7900","accounts","base.dk"
"account_tag_7920","7920","accounts","base.dk"
"account_tag_7940","7940","accounts","base.dk"
"account_tag_7960","7960","accounts","base.dk"
"account_tag_7980","7980","accounts","base.dk"
"account_tag_8000","8000","accounts","base.dk"
"account_tag_8040","8040","accounts","base.dk"
"account_tag_8070","8070","accounts","base.dk"

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="account_tax_report_skat_dk" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.dk"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="account_tax_report_skat_dk_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_sales_group" model="account.report.line">
                <field name="name">VAT due</field>
                <field name="code">DUE_VAT</field>
                <field name="aggregation_formula">DK_SALES_VAT.balance + REVERSE_CHARGE_MERCHANDISES_PURCHASED_OUT_DK.balance + REVERSE_CHARGE_SERVICES_PURCHASED_OUT_DK.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_sales_tax" model="account.report.line">
                        <field name="name">Sales VAT (output VAT)</field>
                        <field name="code">DK_SALES_VAT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_sales_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">UM</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_international_purchase_products" model="account.report.line">
                        <field name="name">VAT on purchases of goods abroad (both EU and non-EU)</field>
                        <field name="code">REVERSE_CHARGE_MERCHANDISES_PURCHASED_OUT_DK</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_international_purchase_products_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">MVU</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_international_purchase_services" model="account.report.line">
                        <field name="name">VAT on purchases of services abroad with reverse charge</field>
                        <field name="code">REVERSE_CHARGE_SERVICES_PURCHASED_OUT_DK</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_international_purchase_services_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">MYUOB</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_deduction_group" model="account.report.line">
                <field name="name">Deductions</field>
                <field name="code">VAT_TO_DEDUCED</field>
                <field name="aggregation_formula">PURCHASE_VAT.balance + OIL_AND_BOTTLED_GAS_TAX.balance + ELECTRICITY_TAX.balance + NATURAL_AND_CITY_GAS_TAX.balance + COAL_TAX.balance + CO2_TAX.balance + WATER_TAX.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_deduction_purchase_tax" model="account.report.line">
                        <field name="name">Purchase VAT (input VAT)</field>
                        <field name="code">PURCHASE_VAT</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_deduction_purchase_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KM</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_deduction_oil_bottle_tax" model="account.report.line">
                        <field name="name">Oil and cylinder gas tax</field>
                        <field name="code">OIL_AND_BOTTLED_GAS_TAX</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_deduction_oil_bottle_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">OFA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_deduction_electrical_tax" model="account.report.line">
                        <field name="name">Electricity Tax</field>
                        <field name="code">ELECTRICITY_TAX</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_deduction_electrical_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">EA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_deduction_gas_tax" model="account.report.line">
                        <field name="name">Natural gas and town gas tax</field>
                        <field name="code">NATURAL_AND_CITY_GAS_TAX</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_deduction_gas_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">NOBA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_deduction_coal_tax" model="account.report.line">
                        <field name="name">Coal Tax</field>
                        <field name="code">COAL_TAX</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_deduction_coal_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_deduction_co2_tax" model="account.report.line">
                        <field name="name">CO-2 tax</field>
                        <field name="code">CO2_TAX</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_deduction_co2_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CO2A</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_deduction_water_tax" model="account.report.line">
                        <field name="name">Water tax</field>
                        <field name="code">WATER_TAX</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_deduction_water_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VA</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_vat_statement" model="account.report.line">
                <field name="name">VAT declaration (positive amount = pay, negative amount = money owed)</field>
                <field name="aggregation_formula">DUE_VAT.balance - VAT_TO_DEDUCED.balance</field>
                <field name="hierarchy_level">0</field>
            </record>
            <record id="account_tax_report_line_additional_info" model="account.report.line">
                <field name="name">Supplementary information</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_section_a_products" model="account.report.line">
                        <field name="name">Category A: goods (EU)</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_section_a_products_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">R-A-V</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_section_a_services" model="account.report.line">
                        <field name="name">Category A = benefits (EU)</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_section_a_services_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">R-A-Y</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_section_b_product_eu" model="account.report.line">
                        <field name="name">Category B = Goods (To be declared for EU sales without VAT)</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_section_b_product_eu_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">R-B-MR</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_section_b_products_non_eu" model="account.report.line">
                        <field name="name">Category B = Goods (Not to be reported for EU sales without VAT)</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_section_b_products_non_eu_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">R-B-UR</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_section_b_services" model="account.report.line">
                        <field name="name">Category B = services - (EU sales excluding VAT)</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_section_b_services_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">R-C-MR</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_section_b_triangular" model="account.report.line">
                        <field name="name">Category B = triangular - (EU sales excluding VAT)</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_section_b_triangular_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">R-D-MR</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_section_c" model="account.report.line">
                        <field name="name">Category C - Value of other goods and services (whatever the country)</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_section_c_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">R-C-UR</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-dk.csv

```csv
"id","name","code","account_type","tag_ids","reconcile","name@da_DK"
"dk_coa_1010","Sale of goods and services","1010","income","l10n_dk.account_tag_1010","False","Salg af varer og ydelser"
"dk_coa_1050","Sales of goods abroad, EU","1050","income","l10n_dk.account_tag_1050","False","Salg af varer udland, EU"
"dk_coa_1100","Sales of goods abroad, non-EU","1100","income","l10n_dk.account_tag_1100","False","Salg af varer udland, ikke-EU"
"dk_coa_1150","Sales of services abroad, EU","1150","income","l10n_dk.account_tag_1150","False","Salg af ydelser udland, EU"
"dk_coa_1200","Sales of services abroad, non-EU","1200","income","l10n_dk.account_tag_1200","False","Salg af ydelser udland, ikke-EU"
"dk_coa_1300","Adjustment for work in progress","1300","income","l10n_dk.account_tag_1300","False","Regulering igangværende arbejder"
"dk_coa_1350","Value adjustments on investment property","1350","income","l10n_dk.account_tag_1350","False","Værdireguleringer af investeringsejendomme"
"dk_coa_1410","Inventory adjustment on stocks of finished goods and work in progress","1410","income","l10n_dk.account_tag_1410","False","Varelagerregulering på lagre af færdigvarer og varer under fremstilling"
"dk_coa_1430","Write-down of stocks of finished goods and work in progress","1430","income","l10n_dk.account_tag_1430","False","Nedskrivning på lagre af færdigvarer og varer under fremstilling"
"dk_coa_1460","Other changes in stocks of finished goods and work in progress","1460","income","l10n_dk.account_tag_1460","False","Øvrige ændringer på lagre af færdigvarer og varer under fremstilling"
"dk_coa_1510","Gain on sale of intangible fixed assets","1510","income","l10n_dk.account_tag_1510","False","Gevinst ved salg af immaterielle anlægsaktiver"
"dk_coa_1530","Gain on sale of tangible fixed assets","1530","income","l10n_dk.account_tag_1530","False","Gevinst ved salg af materielle anlægsaktiver"
"dk_coa_1540","Gain on sale of financial fixed assets","1540","income","l10n_dk.account_tag_1540","False","Gevinst ved salg af finansielle anlægsaktiver"
"dk_coa_1550","Other operating income","1550","income","l10n_dk.account_tag_1550","False","Øvrige andre driftsindtægter"
"dk_coa_1610","Purchase of goods","1610","expense_direct_cost","l10n_dk.account_tag_1610","False","Varekøb"
"dk_coa_1630","Purchases of goods abroad, EU","1630","expense_direct_cost","l10n_dk.account_tag_1630","False","Varekøb udland, EU"
"dk_coa_1650","Purchases of goods from abroad, non-EU","1650","expense_direct_cost","l10n_dk.account_tag_1650","False","Varekøb udland, ikke-EU"
"dk_coa_1660","Benefit purchases","1660","expense_direct_cost","l10n_dk.account_tag_1660","False","Ydelseskøb"
"dk_coa_1710","Benefit purchases abroad, EU","1710","expense_direct_cost","l10n_dk.account_tag_1710","False","Ydelseskøb udland, EU"
"dk_coa_1740","Benefit purchases, abroad, non-EU","1740","expense_direct_cost","l10n_dk.account_tag_1740","False","Ydelseskøb, udland, ikke-EU"
"dk_coa_1770","Inventory adjustment on stocks of raw materials and consumables","1770","expense_direct_cost","l10n_dk.account_tag_1770","False","Varelagerregulering på lagre af råvarer og hjælpematerialer"
"dk_coa_1800","Write-down of inventories","1800","expense_direct_cost","l10n_dk.account_tag_1800","False","Nedskrivning på varelager"
"dk_coa_1830","Freight costs","1830","expense","l10n_dk.account_tag_1830","False","Fragtomkostninger"
"dk_coa_1850","Advertising and publicity","1850","expense","l10n_dk.account_tag_1850","False","Annoncering og reklame"
"dk_coa_1870","Exhibitions and decoration","1870","expense","l10n_dk.account_tag_1870","False","Udstillinger og dekoration"
"dk_coa_1890","Catering services","1890","expense","l10n_dk.account_tag_1890","False","Restaurationsbesøg"
"dk_coa_1910","Representation expenses, tax deduction limited","1910","expense","l10n_dk.account_tag_1910","False","Repræsentationsomkostninger, skattemæssigt begrænset fradrag"
"dk_coa_1930","Representation expenses, fully deductible for tax purposes","1930","expense","l10n_dk.account_tag_1930","False","Repræsentationsomkostninger, fuld fradragsret skattemæssigt"
"dk_coa_1950","Other selling expenses","1950","expense","l10n_dk.account_tag_1950","False","Andre salgsomkostninger"
"dk_coa_1970","Newspapers and magazines","1970","expense","l10n_dk.account_tag_1970","False","Aviser og blade"
"dk_coa_1990","Gifts and flowers","1990","expense","l10n_dk.account_tag_1990","False","Gaver og blomster"
"dk_coa_2030","Rent","2030","expense","l10n_dk.account_tag_2030","False","Husleje, ekskl. el, vand og varme"
"dk_coa_2050","Electricity","2050","expense","l10n_dk.account_tag_2050","False","El"
"dk_coa_2060","Electricity tax","2060","expense","l10n_dk.account_tag_2060","False","Elafgift"
"dk_coa_2070","Water","2070","expense","l10n_dk.account_tag_2070","False","Vand"
"dk_coa_2080","Heating","2080","expense","l10n_dk.account_tag_2080","False","Varme"
"dk_coa_2090","Water tax","2090","expense","l10n_dk.account_tag_2090","False","Vandafgift"
"dk_coa_2100","Oil and LPG tax","2100","expense","l10n_dk.account_tag_2100","False","Olie- og flaskegasafgift"
"dk_coa_2110","Coal tax","2110","expense","l10n_dk.account_tag_2110","False","Kulafgift"
"dk_coa_2120","Natural and town gas tax","2120","expense","l10n_dk.account_tag_2120","False","Naturgas- og bygasafgift"
"dk_coa_2130","CO2 tax","2130","expense","l10n_dk.account_tag_2130","False","Co2-afgift"
"dk_coa_2140","Other taxes","2140","expense","l10n_dk.account_tag_2140","False","Øvrige afgifter"
"dk_coa_2150","Cleaning and sanitation (waste management)","2150","expense","l10n_dk.account_tag_2150","False","Rengøring og renovation (affaldshåndtering)"
"dk_coa_2160","Repair and maintenance","2160","expense","l10n_dk.account_tag_2160","False","Reparation og vedligeholdelse"
"dk_coa_2170","Repair and maintenance","2170","expense","l10n_dk.account_tag_2170","False","Reparation og vedligeholdelse, ejendom skattemæssigt afskrivningsberettiget, bygning 1"
"dk_coa_2180","Insurance","2180","expense","l10n_dk.account_tag_2180","False","Forsikringer"
"dk_coa_2190","Property taxes","2190","expense","l10n_dk.account_tag_2190","False","Ejendomsskatter"
"dk_coa_2200","Other premises costs","2200","expense","l10n_dk.account_tag_2200","False","Andre lokaleomkostninger"
"dk_coa_2230","Small acquisitions below the tax threshold for small assets","2230","expense","l10n_dk.account_tag_2230","False","Småanskaffelser under skattemæssig grænse for småaktiver"
"dk_coa_2240","Small acquisitions above the tax threshold for small assets","2240","expense","l10n_dk.account_tag_2240","False","Småanskaffelser over skattemæssig grænse for småaktiver"
"dk_coa_2250","Subcontractors","2250","expense","l10n_dk.account_tag_2250","False","Underleverandører"
"dk_coa_2260","Research and development costs","2260","expense","l10n_dk.account_tag_2260","False","Forsknings- og udviklingsomkostninger"
"dk_coa_2270","Other production costs","2270","expense","l10n_dk.account_tag_2270","False","Øvrige produktionsomkostninger"
"dk_coa_2280","Recognized losses on trade receivables","2280","expense","l10n_dk.account_tag_2280","False","Konstaterede tab på tilgodehavender fra salg og tjenesteydelser"
"dk_coa_2290","Adjustment of impairment loss on trade receivables","2290","expense","l10n_dk.account_tag_2290","False","Regulering af nedskrivning på tilgodehavender fra salg og tjenesteydelser"
"dk_coa_2300","Adjustment of receivables from group enterprises and associates","2300","expense","l10n_dk.account_tag_2300","False","Regulering af tilgodehavender fra tilknyttede virksomheder og associerede virksomheder"
"dk_coa_2310","IT equipment, etc.","2310","expense","l10n_dk.account_tag_2310","False","It-udstyr mv."
"dk_coa_2330","Tax-free travel and transportation allowances","2330","expense","l10n_dk.account_tag_2330","False","Skattefri rejse- og befordringsgodtgørelse"
"dk_coa_2350","Canteen expenses","2350","expense","l10n_dk.account_tag_2350","False","Kantineudgifter"
"dk_coa_2370","Membership fees","2370","expense","l10n_dk.account_tag_2370","False","Kontingenter"
"dk_coa_2380","Professional literature","2380","expense","l10n_dk.account_tag_2380","False","Faglitteratur"
"dk_coa_2390","Postage and charges","2390","expense","l10n_dk.account_tag_2390","False","Porto og gebyrer"
"dk_coa_2410","Telephone and internet etc. (business only)","2410","expense","l10n_dk.account_tag_2410","False","Telefon og internet mv. (kun virksomhed)"
"dk_coa_2420","Telephone and internet etc. (partly private)","2420","expense","l10n_dk.account_tag_2420","False","Telefon og internet mv. (delvist privat)"
"dk_coa_2450","Office supplies","2450","expense","l10n_dk.account_tag_2450","False","Kontorartikler"
"dk_coa_2460","Rent and operating lease payments (excluding rent)","2460","expense","l10n_dk.account_tag_2460","False","Leje og operationelle leasingydelser (ekskl. husleje)"
"dk_coa_2470","Travel expenses","2470","expense","l10n_dk.account_tag_2470","False","Rejseudgifter"
"dk_coa_2480","Temporary employment agency services","2480","expense","l10n_dk.account_tag_2480","False","Vikarassistance"
"dk_coa_2510","Consultancy services","2510","expense","l10n_dk.account_tag_2510","False","Konsulentydelser"
"dk_coa_2520","Training costs","2520","expense","l10n_dk.account_tag_2520","False","Kursusudgifter"
"dk_coa_2530","Leasing costs, passenger cars","2530","expense","l10n_dk.account_tag_2530","False","Leasingomkostninger, personbiler"
"dk_coa_2540","Operating costs, passenger cars","2540","expense","l10n_dk.account_tag_2540","False","Driftsomkostninger, personbiler"
"dk_coa_2560","Operating costs, vans","2560","expense","l10n_dk.account_tag_2560","False","Driftsomkostninger, varebiler"
"dk_coa_2620","Parking costs","2620","expense","l10n_dk.account_tag_2620","False","Parkeringsudgifter"
"dk_coa_2630","Car expenses according to government tariffs","2630","expense","l10n_dk.account_tag_2630","False","Biludgifter efter statens takster"
"dk_coa_2640","Free car","2640","expense","l10n_dk.account_tag_2640","False","Fri bil"
"dk_coa_2650","'Workers' compensation insurance'","2650","expense","l10n_dk.account_tag_2650","False","Arbejdsskadeforsikring"
"dk_coa_2660","Government fees and fines (not tax deductible)","2660","expense","l10n_dk.account_tag_2660","False","Offentlige gebyrer og bøder (ej fradragsberettiget skattemæssigt)"
"dk_coa_2670","Audit and accounting assistance","2670","expense","l10n_dk.account_tag_2670","False","Revision og regnskabsmæssig assistance"
"dk_coa_2680","Legal assistance","2680","expense","l10n_dk.account_tag_2680","False","Advokatmæssig assistance"
"dk_coa_2690","Other consultancy fees","2690","expense","l10n_dk.account_tag_2690","False","Øvrige rådgivningshonorarer"
"dk_coa_2700","Advisory fees not deductible for tax purposes","2700","expense","l10n_dk.account_tag_2700","False","Ej skattemæssigt fradragsberettigede rådgivningshonorarer"
"dk_coa_2710","Administration/management fee","2710","expense","l10n_dk.account_tag_2710","False","Administrationsvederlag/management fee"
"dk_coa_2720","Rounding of pennies/cash differences","2720","expense","l10n_dk.account_tag_2720","False","Øreafrunding/kassedifferencer"
"dk_coa_2810","Other external costs","2810","expense","l10n_dk.account_tag_2810","False","Andre eksterne omkostninger"
"dk_coa_2850","Salaries and wages","2850","expense","l10n_dk.account_tag_2850","False","Lønninger"
"dk_coa_2860","Vacation pay liability","2860","expense","l10n_dk.account_tag_2860","False","Feriepengeforpligtelse"
"dk_coa_2870","Anniversary bonuses and severance pay","2870","expense","l10n_dk.account_tag_2870","False","Jubilæumsgratiale og fratrædelsesgodtgørelse"
"dk_coa_2880","Directors' fees","2880","expense","l10n_dk.account_tag_2880","False","Bestyrelseshonorar"
"dk_coa_2890","AM Contributory A-Income","2890","expense","l10n_dk.account_tag_2890","False","AM Bidragspligtig A-Indkomst"
"dk_coa_2900","AM Contribution-free A-Income","2900","expense","l10n_dk.account_tag_2900","False","AM Bidragsfri A-Indkomst"
"dk_coa_2910","Pensions","2910","expense","l10n_dk.account_tag_2910","False","Pensioner"
"dk_coa_2920","Remuneration in lieu of pension promises","2920","expense","l10n_dk.account_tag_2920","False","Vederlag til afløsning af pensionstilsagn"
"dk_coa_2930","Social security costs","2930","expense","l10n_dk.account_tag_2930","False","Omkostninger til social sikring"
"dk_coa_2940","AER/ AUB","2940","expense","l10n_dk.account_tag_2940","False","AER/ AUB"
"dk_coa_2950","ATP","2950","expense","l10n_dk.account_tag_2950","False","ATP"
"dk_coa_2960","Other personnel costs","2960","expense","l10n_dk.account_tag_2960","False","Andre personaleomkostninger"
"dk_coa_2965","Employee benefits","2965","expense","l10n_dk.account_tag_2965","False","Personalegoder"
"dk_coa_2968","Salary reimbursements","2968","expense","l10n_dk.account_tag_2968","False","Lønrefusioner"
"dk_coa_2970","Tax-free allowances paid in the form of mileage and subsistence allowances","2970","expense","l10n_dk.account_tag_2970","False","Udbetalte skattefrie godtgørelser i form af kørepenge og diæter"
"dk_coa_2980","Payroll tax","2980","expense","l10n_dk.account_tag_2980","False","Lønsumsafgift"
"dk_coa_3000","Amortization and impairment of acquired intangible assets","3000","expense_depreciation","l10n_dk.account_tag_3000","False","Af- og nedskrivninger af erhvervede immaterielle anlægsaktiver"
"dk_coa_3010","Amortization and impairment of goodwill","3010","expense_depreciation","l10n_dk.account_tag_3010","False","Af- og nedskrivninger af goodwill"
"dk_coa_3020","Depreciation and amortization of land and buildings","3020","expense_depreciation","l10n_dk.account_tag_3020","False","Af- og nedskrivninger af grunde og bygninger"
"dk_coa_3030","Depreciation and write-offs of plant and machinery","3030","expense_depreciation","l10n_dk.account_tag_3030","False","Af- og nedskrivninger af produktionsanlæg og maskiner"
"dk_coa_3040","Depreciation and write-offs of leasehold improvements","3040","expense_depreciation","l10n_dk.account_tag_3040","False","Af- og nedskrivninger af indretning af lejede lokaler"
"dk_coa_3050","Depreciation and write-offs of other plant, equipment and furniture","3050","expense_depreciation","l10n_dk.account_tag_3050","False","Af- og nedskrivninger af andre anlæg, driftsmateriel og inventar"
"dk_coa_3060","Depreciation and amortization of software","3060","expense_depreciation","l10n_dk.account_tag_3060","False","Af- og nedskrivninger af software"
"dk_coa_3070","Depreciation and write-downs of land and buildings held under finance leases","3070","expense_depreciation","l10n_dk.account_tag_3070","False","Af- og nedskrivninger af finansielt leasede grunde og bygninger"
"dk_coa_3080","Depreciation and write-downs of plant and machinery held under finance leases","3080","expense_depreciation","l10n_dk.account_tag_3080","False","Af- og nedskrivninger af finansielt leasede produktionsanlæg og maskiner"
"dk_coa_3090","Depreciation, amortization and write-downs of other plant, machinery and equipment held under finance leases","3090","expense_depreciation","l10n_dk.account_tag_3090","False","Af- og nedskrivninger af finansielt leasede andre anlæg, driftsmateriel og inventar"
"dk_coa_3130","Impairment losses on current assets","3130","expense","l10n_dk.account_tag_3130","False","Nedskrivninger af omsætningsaktiver, som overstiger normale nedskrivninger"
"dk_coa_3160","Loss on disposal of intangible fixed assets","3160","expense","l10n_dk.account_tag_3160","False","Tab ved salg af immaterielle anlægsaktiver"
"dk_coa_3170","Loss on disposal of property, plant and equipment","3170","expense","l10n_dk.account_tag_3170","False","Tab ved salg af materielle anlægsaktiver"
"dk_coa_3180","Other operating expenses","3180","expense","l10n_dk.account_tag_3180","False","Øvrige andre driftsomkostninger"
"dk_coa_3200","Income from investments in affiliated enterprises","3200","income_other","l10n_dk.account_tag_3200","False","Indtægter af kapitalandele i tilknyttede virksomheder"
"dk_coa_3230","Income from equity investments in participating interests","3230","income_other","l10n_dk.account_tag_3230","False","Indtægter af kapitalandele i kapitalinteresser"
"dk_coa_3380","Dividends from unquoted portfolio shares (gross dividend)","3380","income_other","l10n_dk.account_tag_3380","False","Udbytte fra unoterede porteføljeaktier (bruttoudbytte)"
"dk_coa_3400","Other income on other equity investments, securities and receivables that are fixed assets","3400","income_other","l10n_dk.account_tag_3400","False","Øvrige indtægter af andre kapitalandele, værdipapirer og tilgodehavender, der er anlægsaktiver"
"dk_coa_3440","Other financial income from affiliated enterprises","3440","income_other","l10n_dk.account_tag_3440","False","Andre finansielle indtægter fra tilknyttede virksomheder"
"dk_coa_3470","Interest from banks","3470","income_other","l10n_dk.account_tag_3470","False","Renter fra banker"
"dk_coa_3490","Interest receivable on trade debtors","3490","income_other","l10n_dk.account_tag_3490","False","Renter vedr. tilgodehavende fra salg af varer og tjenesteydelser"
"dk_coa_3510","Interest subsidies, etc. from general government (non-taxable)","3510","income_other","l10n_dk.account_tag_3510","False","Rentetillæg mv. fra det offentlige (ej skattepligtig)"
"dk_coa_3530","Other financial income","3530","income_other","l10n_dk.account_tag_3530","False","Øvrige finansielle indtægter"
"dk_coa_3560","Write-down of financial assets","3560","expense","l10n_dk.account_tag_3560","False","Nedskrivning af finansielle aktiver"
"dk_coa_3590","Financial expenses from investments in affiliated enterprises","3590","expense","l10n_dk.account_tag_3590","False","Finansielle omkostninger, der hidrører fra tilknyttede virksomheder"
"dk_coa_3610","Exchange rate adjustments","3610","expense","l10n_dk.account_tag_3610","False","Valutakursreguleringer"
"dk_coa_3620","Exchange rate adjustments, foreign subsidiaries","3620","expense","l10n_dk.account_tag_3620","False","Valutakursreguleringer, udenlandske dattervirksomheder"
"dk_coa_3630","Exchange losses on cash and cash equivalents, bank and mortgage debt","3630","expense","l10n_dk.account_tag_3630","False","Kurstab på likvider, bankgæld og prioritetsgæld"
"dk_coa_3640","Interest on financial lease liabilities","3640","expense","l10n_dk.account_tag_3640","False","Renter på finansiel leasinggæld"
"dk_coa_3650","Interest payable to suppliers of goods and services","3650","expense","l10n_dk.account_tag_3650","False","Renter vedr. leverandører af varer og tjenesteydelser"
"dk_coa_3670","Interest payable to banks and mortgage credit institutions","3670","expense","l10n_dk.account_tag_3670","False","Renter til banker og realkreditinstitutter"
"dk_coa_3675","Interest payable to general government (not deductible for tax purposes)","3675","expense","l10n_dk.account_tag_3675","False","Renter til det offentlige (ej fradragsberettiget skattemæssigt)"
"dk_coa_3680","Value adjustments on investment property","3680","expense","l10n_dk.account_tag_3680","False","Værdireguleringer af investeringsejendomme"
"dk_coa_3690","Other financial expenses","3690","expense","l10n_dk.account_tag_3690","False","Andre finansielle omkostninger"
"dk_coa_3740","Current tax","3740","expense","l10n_dk.account_tag_3740","False","Aktuel skat"
"dk_coa_3760","Change in deferred tax","3760","expense","l10n_dk.account_tag_3760","False","Ændring af udskudt skat"
"dk_coa_3780","Adjustments relating to previous years","3780","expense","l10n_dk.account_tag_3780","False","Regulering vedrørende tidligere år"
"dk_coa_3810","Other taxes","3810","expense","l10n_dk.account_tag_3810","False","Andre skatter"
"dk_coa_5010","Goodwill, book value at beginning of period","5010","asset_fixed","l10n_dk.account_tag_5010","False","Goodwill, bogført værdi primo"
"dk_coa_5020","Goodwill, additions during the year","5020","asset_fixed","l10n_dk.account_tag_5020","False","Goodwill, årets tilgange"
"dk_coa_5030","Goodwill, departures during the year","5030","asset_fixed","l10n_dk.account_tag_5030","False","Goodwill, årets afgange"
"dk_coa_5040","Goodwill, other value adjustments","5040","asset_fixed","l10n_dk.account_tag_5040","False","Goodwill, øvrige værdireguleringer"
"dk_coa_5050","Goodwill, amortization and impairment for the year","5050","asset_fixed","l10n_dk.account_tag_5050","False","Goodwill, årets af- og nedskrivninger"
"dk_coa_5060","Goodwill, amortization and impairment reversals","5060","asset_fixed","l10n_dk.account_tag_5060","False","Goodwill, tilbageførte af- og nedskrivninger"
"dk_coa_5080","Acquired intangible fixed assets, book value at beginning of period","5080","asset_fixed","l10n_dk.account_tag_5080","False","Erhvervede immaterielle anlægsaktiver, bogført værdi primo"
"dk_coa_5090","Acquired intangible fixed assets, additions during the year","5090","asset_fixed","l10n_dk.account_tag_5090","False","Erhvervede immaterielle anlægsaktiver, årets tilgange"
"dk_coa_5100","Acquired intangible fixed assets, disposals during the year","5100","asset_fixed","l10n_dk.account_tag_5100","False","Erhvervede immaterielle anlægsaktiver, årets afgange"
"dk_coa_5110","Acquired intangible fixed assets, other value adjustments","5110","asset_fixed","l10n_dk.account_tag_5110","False","Erhvervede immaterielle anlægsaktiver, øvrige værdireguleringer"
"dk_coa_5120","Acquired intangible assets, amortization and impairment losses for the year","5120","asset_fixed","l10n_dk.account_tag_5120","False","Erhvervede immaterielle anlægsaktiver, årets af- og nedskrivninger"
"dk_coa_5130","Acquired intangible assets, amortization and impairment losses reversed","5130","asset_fixed","l10n_dk.account_tag_5130","False","Erhvervede immaterielle anlægsaktiver, tilbageførte af- og nedskrivninger"
"dk_coa_5160","Investment properties, book value at beginning of period","5160","asset_fixed","l10n_dk.account_tag_5160","False","Investeringsejendomme, bogført værdi primo"
"dk_coa_5170","Investment real estate, additions during the year","5170","asset_fixed","l10n_dk.account_tag_5170","False","Investeringsejendomme, årets tilgange"
"dk_coa_5180","Investment real estate, disposals during the year","5180","asset_fixed","l10n_dk.account_tag_5180","False","Investeringsejendomme, årets afgange"
"dk_coa_5190","Investment properties, improvements during the year","5190","asset_fixed","l10n_dk.account_tag_5190","False","Investeringsejendomme, årets forbedringer"
"dk_coa_5200","Investment properties, other value adjustments","5200","asset_fixed","l10n_dk.account_tag_5200","False","Investeringsejendomme, øvrige værdireguleringer"
"dk_coa_5210","Investment property, depreciation, amortization and impairment for the year","5210","asset_fixed","l10n_dk.account_tag_5210","False","Investeringsejendomme, årets af- og nedskrivninger"
"dk_coa_5220","Investment properties, reversal of depreciation and impairment losses","5220","asset_fixed","l10n_dk.account_tag_5220","False","Investeringsejendomme, tilbageførte af- og nedskrivninger"
"dk_coa_5240","Investment properties under construction, book value at beginning of period","5240","asset_fixed","l10n_dk.account_tag_5240","False","Investeringsejendomme under opførelse, bogført værdi primo"
"dk_coa_5250","Investment properties under construction, additions during the year","5250","asset_fixed","l10n_dk.account_tag_5250","False","Investeringsejendomme under opførelse, årets tilgange"
"dk_coa_5260","Investment properties under construction, disposals during the year","5260","asset_fixed","l10n_dk.account_tag_5260","False","Investeringsejendomme under opførelse, årets afgange"
"dk_coa_5270","Investment properties under construction, improvements during the year","5270","asset_fixed","l10n_dk.account_tag_5270","False","Investeringsejendomme under opførelse, årets forbedringer"
"dk_coa_5280","Investment properties under construction, other value adjustments","5280","asset_fixed","l10n_dk.account_tag_5280","False","Investeringsejendomme under opførelse, øvrige værdireguleringer"
"dk_coa_5290","Investment properties under construction, impairment losses for the year","5290","asset_fixed","l10n_dk.account_tag_5290","False","Investeringsejendomme under opførelse, årets nedskrivninger"
"dk_coa_5300","Investment properties under construction, reversal of impairment losses","5300","asset_fixed","l10n_dk.account_tag_5300","False","Investeringsejendomme under opførelse, tilbageførte nedskrivninger"
"dk_coa_5320","Land and buildings, book value at beginning of period","5320","asset_fixed","l10n_dk.account_tag_5320","False","Grunde og bygninger, bogført værdi primo"
"dk_coa_5330","Land and buildings, additions during the year","5330","asset_fixed","l10n_dk.account_tag_5330","False","Grunde og bygninger, årets tilgange"
"dk_coa_5340","Land and buildings, end of year","5340","asset_fixed","l10n_dk.account_tag_5340","False","Grunde og bygninger, årets afgange"
"dk_coa_5350","Land and buildings, improvements during the year","5350","asset_fixed","l10n_dk.account_tag_5350","False","Grunde og bygninger, årets forbedringer"
"dk_coa_5370","Land and buildings, other value adjustments","5370","asset_fixed","l10n_dk.account_tag_5370","False","Grunde og bygninger, øvrige værdireguleringer"
"dk_coa_5390","Land and buildings, depreciation and impairment for the year","5390","asset_fixed","l10n_dk.account_tag_5390","False","Grunde og bygninger, årets af- og nedskrivninger"
"dk_coa_5400","Land and buildings, reversal of depreciation and impairment losses","5400","asset_fixed","l10n_dk.account_tag_5400","False","Grunde og bygninger, tilbageførte af- og nedskrivninger"
"dk_coa_5420","Plant and machinery, book value at beginning of period","5420","asset_fixed","l10n_dk.account_tag_5420","False","Produktionsanlæg og maskiner, bogført værdi primo"
"dk_coa_5430","Production plant and machinery, annual additions","5430","asset_fixed","l10n_dk.account_tag_5430","False","Produktionsanlæg og maskiner, årets tilgange"
"dk_coa_5440","Production plant and machinery, end of year","5440","asset_fixed","l10n_dk.account_tag_5440","False","Produktionsanlæg og maskiner, årets afgange"
"dk_coa_5450","Plant and machinery, other value adjustments","5450","asset_fixed","l10n_dk.account_tag_5450","False","Produktionsanlæg og maskiner, øvrige værdireguleringer"
"dk_coa_5470","Plant and machinery, depreciation and amortization for the year","5470","asset_fixed","l10n_dk.account_tag_5470","False","Produktionsanlæg og maskiner, årets af- og nedskrivninger"
"dk_coa_5480","Plant and machinery, depreciation and write-offs reversed","5480","asset_fixed","l10n_dk.account_tag_5480","False","Produktionsanlæg og maskiner, tilbageførte af- og nedskrivninger"
"dk_coa_5500","Leasehold improvements, book value at beginning of period","5500","asset_fixed","l10n_dk.account_tag_5500","False","Indretning af lejede lokaler, bogført værdi primo"
"dk_coa_5510","Fitting-out of rented premises, annual approaches","5510","asset_fixed","l10n_dk.account_tag_5510","False","Indretning af lejede lokaler, årets tilgange"
"dk_coa_5520","Furnishing of rented premises, yearly departures","5520","asset_fixed","l10n_dk.account_tag_5520","False","Indretning af lejede lokaler, årets afgange"
"dk_coa_5530","Fitting-out of rented premises, other value adjustments","5530","asset_fixed","l10n_dk.account_tag_5530","False","Indretning af lejede lokaler, øvrige værdireguleringer"
"dk_coa_5540","Leasehold improvements, depreciation and amortization for the year","5540","asset_fixed","l10n_dk.account_tag_5540","False","Indretning af lejede lokaler, årets af- og nedskrivninger"
"dk_coa_5550","Fitting-out of rented premises, depreciation and write-offs reversed","5550","asset_fixed","l10n_dk.account_tag_5550","False","Indretning af lejede lokaler, tilbageførte af- og nedskrivninger"
"dk_coa_5570","Other fixtures and fittings, tools and equipment, book value at beginning of period","5570","asset_fixed","l10n_dk.account_tag_5570","False","Andre anlæg, driftsmateriel og inventar, bogført værdi primo"
"dk_coa_5580","Other fixtures and fittings, tools and equipment, additions during the year","5580","asset_fixed","l10n_dk.account_tag_5580","False","Andre anlæg, driftsmateriel og inventar, årets tilgange"
"dk_coa_5590","Other fixtures and fittings, tools and equipment, disposals during the year","5590","asset_fixed","l10n_dk.account_tag_5590","False","Andre anlæg, driftsmateriel og inventar, årets afgange"
"dk_coa_5600","Other fixtures and fittings, tools and equipment, other value adjustments","5600","asset_fixed","l10n_dk.account_tag_5600","False","Andre anlæg, driftsmateriel og inventar, øvrige værdireguleringer"
"dk_coa_5610","Other fixtures and fittings, tools and equipment, depreciation and write-downs for the year","5610","asset_fixed","l10n_dk.account_tag_5610","False","Andre anlæg, driftsmateriel og inventar, årets af- og nedskrivninger"
"dk_coa_5620","Other fixtures and fittings, furniture and equipment, depreciation and write-offs reversed","5620","asset_fixed","l10n_dk.account_tag_5620","False","Andre anlæg, driftsmateriel og inventar, tilbageførte af- og nedskrivninger"
"dk_coa_5640","Tangible fixed assets under construction and prepayments for tangible fixed assets, book value at beginning of period","5640","asset_fixed","l10n_dk.account_tag_5640","False","Materielle anlægsaktiver under udførelse og forudbetalinger for materielle anlægsaktiver, bogført værdi primo"
"dk_coa_5650","Tangible fixed assets under construction and prepayments for tangible fixed assets, additions during the year","5650","asset_fixed","l10n_dk.account_tag_5650","False","Materielle anlægsaktiver under udførelse og forudbetalinger for materielle anlægsaktiver, årets tilgange"
"dk_coa_5660","Tangible fixed assets under construction and prepayments for tangible fixed assets, disposals during the year","5660","asset_fixed","l10n_dk.account_tag_5660","False","Materielle anlægsaktiver under udførelse og forudbetalinger for materielle anlægsaktiver, årets afgange"
"dk_coa_5670","Tangible fixed assets under construction and prepayments for tangible fixed assets, other value adjustments","5670","asset_fixed","l10n_dk.account_tag_5670","False","Materielle anlægsaktiver under udførelse og forudbetalinger for materielle anlægsaktiver, øvrige værdireguleringer"
"dk_coa_5680","Tangible fixed assets under construction and prepayments for tangible fixed assets, impairment losses for the year","5680","asset_fixed","l10n_dk.account_tag_5680","False","Materielle anlægsaktiver under udførelse og forudbetalinger for materielle anlægsaktiver, årets nedskrivninger"
"dk_coa_5690","Tangible fixed assets under construction and prepayments for tangible fixed assets, reversal of impairment losses","5690","asset_fixed","l10n_dk.account_tag_5690","False","Materielle anlægsaktiver under udførelse og forudbetalinger for materielle anlægsaktiver, tilbageførte nedskrivninger"
"dk_coa_5710","Assets held under finance leases, book value at beginning of period","5710","asset_fixed","l10n_dk.account_tag_5710","False","Finansielt leasede aktiver, bogført værdi primo"
"dk_coa_5720","Assets held under finance leases, additions during the year","5720","asset_fixed","l10n_dk.account_tag_5720","False","Finansielt leasede aktiver, årets tilgange"
"dk_coa_5730","Assets held under finance leases, disposals during the year","5730","asset_fixed","l10n_dk.account_tag_5730","False","Finansielt leasede aktiver, årets afgange"
"dk_coa_5740","Finance lease assets, other value adjustments","5740","asset_fixed","l10n_dk.account_tag_5740","False","Finansielt leasede aktiver, øvrige værdireguleringer"
"dk_coa_5750","Assets held under finance leases, depreciation and amortization for the year","5750","asset_fixed","l10n_dk.account_tag_5750","False","Finansielt leasede aktiver, årets af- og nedskrivninger"
"dk_coa_5760","Assets held under finance leases, reversal of depreciation and impairment losses","5760","asset_fixed","l10n_dk.account_tag_5760","False","Finansielt leasede aktiver, tilbageførte af- og nedskrivninger"
"dk_coa_5800","Investments in group enterprises, book value at beginning of period","5800","asset_receivable","l10n_dk.account_tag_5800","True","Kapitalandele i tilknyttede virksomheder, bogført værdi primo"
"dk_coa_5810","Investments in affiliated enterprises, additions during the year","5810","asset_fixed","l10n_dk.account_tag_5810","True","Kapitalandele i tilknyttede virksomheder, årets tilgange"
"dk_coa_5820","Investments in affiliated enterprises, disposals during the year","5820","asset_fixed","l10n_dk.account_tag_5820","True","Kapitalandele i tilknyttede virksomheder, årets afgange"
"dk_coa_5830","Investments in group enterprises, other value adjustments","5830","asset_fixed","l10n_dk.account_tag_5830","True","Kapitalandele i tilknyttede virksomheder, øvrige værdireguleringer"
"dk_coa_5840","Investments in affiliated enterprises, impairment losses for the year","5840","asset_fixed","l10n_dk.account_tag_5840","True","Kapitalandele i tilknyttede virksomheder, årets nedskrivninger"
"dk_coa_5850","Investments in affiliated enterprises, reversal of impairment losses","5850","asset_fixed","l10n_dk.account_tag_5850","True","Kapitalandele i tilknyttede virksomheder, tilbageførte nedskrivninger"
"dk_coa_5870","Long-term receivables from affiliated enterprises","5870","asset_fixed","l10n_dk.account_tag_5870","True","Langfristede tilgodehavender hos tilknyttede virksomheder"
"dk_coa_5880","Impairment losses on long-term receivables from affiliated enterprises","5880","asset_fixed","l10n_dk.account_tag_5880","True","Nedskrivning på langfristede tilgodehavender hos tilknyttede virksomheder"
"dk_coa_5900","Investments in participating interests, book value at beginning of period","5900","asset_fixed","l10n_dk.account_tag_5900","True","Kapitalandele i kapitalinteresser, bogført værdi primo"
"dk_coa_5910","Investments in participating interests, additions during the year","5910","asset_fixed","l10n_dk.account_tag_5910","True","Kapitalandele i kapitalinteresser, årets tilgange"
"dk_coa_5920","Investments in participating interests, disposals during the year","5920","asset_fixed","l10n_dk.account_tag_5920","True","Kapitalandele i kapitalinteresser, årets afgange"
"dk_coa_5930","Investments in participating interests, other value adjustments","5930","asset_fixed","l10n_dk.account_tag_5930","True","Kapitalandele i kapitalinteresser, øvrige værdireguleringer"
"dk_coa_5940","Participating interests, impairment losses for the year","5940","asset_fixed","l10n_dk.account_tag_5940","True","Kapitalandele i kapitalinteresser, årets nedskrivninger"
"dk_coa_5950","Equity investments, opening book value, impairment losses reversed","5950","asset_fixed","l10n_dk.account_tag_5950","True","Kapitalandele i kapitalinteresser, bogført værdi primo, tilbageførte nedskrivninger"
"dk_coa_5970","Long-term receivables from participating interests","5970","asset_fixed","l10n_dk.account_tag_5970","True","Langfristede tilgodehavender hos kapitalinteresser"
"dk_coa_5980","Impairment loss on long-term receivables from participating interests","5980","asset_fixed","l10n_dk.account_tag_5980","True","Nedskrivning på langfristede tilgodehavender hos kapitalinteresser"
"dk_coa_6000","Other securities and equity investments","6000","asset_fixed","l10n_dk.account_tag_6000","True","Andre værdipapirer og kapitalandele"
"dk_coa_6020","Deferred tax assets","6020","asset_fixed","l10n_dk.account_tag_6020","True","Udskudte skatteaktiver"
"dk_coa_6030","Other (non-current) receivables","6030","asset_fixed","l10n_dk.account_tag_6030","True","Øvrige (langfristede) tilgodehavender"
"dk_coa_6040","Deposits","6040","asset_fixed","l10n_dk.account_tag_6040","True","Deposita"
"dk_coa_6060","Amounts owed by enterprise participants and management","6060","asset_fixed","l10n_dk.account_tag_6060","True","Tilgodehavender hos virksomhedsdeltagere og ledelse"
"dk_coa_6080","Raw materials and consumables","6080","asset_fixed","l10n_dk.account_tag_6080","False","Råvarer og hjælpematerialer"
"dk_coa_6090","Write-down of raw materials and consumables","6090","asset_fixed","l10n_dk.account_tag_6090","False","Nedskrivning på råvarer og hjælpematerialer"
"dk_coa_6110","Work in progress","6110","asset_fixed","l10n_dk.account_tag_6110","False","Varer under fremstilling"
"dk_coa_6120","Write-down of work in progress","6120","asset_fixed","l10n_dk.account_tag_6120","False","Nedskrivning på varer under fremstilling"
"dk_coa_6140","Manufactured goods and merchandise","6140","asset_fixed","l10n_dk.account_tag_6140","False","Fremstillede varer og handelsvarer"
"dk_coa_6150","Write-down of manufactured goods and merchandise","6150","asset_fixed","l10n_dk.account_tag_6150","False","Nedskrivning på fremstillede varer og handelsvarer"
"dk_coa_6170","Prepayments for goods","6170","asset_fixed","l10n_dk.account_tag_6170","False","Forudbetalinger for varer"
"dk_coa_6190","Trade and other receivables","6190","asset_receivable","l10n_dk.account_tag_6190","True","Tilgodehavender fra salg og tjenesteydelser"
"dk_coa_6200","Accumulated allowances for losses on trade receivables","6200","asset_current","l10n_dk.account_tag_6200","False","Akkumulerede nedskrivninger til tab på tilgodehavender fra salg og tjenesteydelser"
"dk_coa_6220","Short-term receivables from affiliated enterprises","6220","asset_current","l10n_dk.account_tag_6220","False","Kortfristede tilgodehavender hos tilknyttede virksomheder"
"dk_coa_6230","Accumulated allowances for losses on receivables from related companies","6230","asset_current","l10n_dk.account_tag_6230","False","Akkumulerede nedskrivninger til tab på tilgodehavender fra tilknyttede virksomheder"
"dk_coa_6240","Short-term receivables from participating interests","6240","asset_current","l10n_dk.account_tag_6240","False","Kortfristede tilgodehavender hos kapitalinteresser"
"dk_coa_6250","Accumulated allowances for losses on receivables from participating interests","6250","asset_current","l10n_dk.account_tag_6250","False","Akkumulerede nedskrivninger til tab på tilgodehavender fra kapitalinteresser"
"dk_coa_6270","Contract work in progress","6270","asset_current","l10n_dk.account_tag_6270","False","Igangværende arbejder for fremmed regning"
"dk_coa_6290","Deferred tax assets","6290","asset_current","l10n_dk.account_tag_6290","False","Udskudte skatteaktiver"
"dk_coa_6300","Corporation tax receivable (current)","6300","asset_current","l10n_dk.account_tag_6300","False","Tilgodehavende selskabsskat (kortfristet)"
"dk_coa_6310","Tax receivable at source","6310","asset_current","l10n_dk.account_tag_6310","False","Tilgodehavende kildeskat"
"dk_coa_6320","VAT receivable (current)","6320","asset_current","l10n_dk.account_tag_6320","False","Tilgodehavende moms (kortfristet)"
"dk_coa_6330","Other receivables (current)","6330","asset_current","l10n_dk.account_tag_6330","False","Øvrige tilgodehavender (kortfristede)"
"dk_coa_6350","Claims for payment of working capital and share premium","6350","asset_current","l10n_dk.account_tag_6350","False","Krav på indbetaling af virksomhedskapital og overkurs"
"dk_coa_6370","Short-term receivables from participants and management","6370","asset_current","l10n_dk.account_tag_6370","False","Kortfristede tilgodehavender hos virksomhedsdeltagere og ledelse"
"dk_coa_6390","Prepayments and accrued income","6390","asset_current","l10n_dk.account_tag_6390","False","Periodeafgrænsningsposter, der kan opretholdes skattemæssigt"
"dk_coa_6400","Prepayments and accrued income","6400","asset_current","l10n_dk.account_tag_6400","False","Periodeafgrænsningsposter, der ikke kan opretholdes skattemæssigt"
"dk_coa_6420","Investments in affiliated enterprises","6420","asset_current","l10n_dk.account_tag_6420","False","Kapitalandele i tilknyttede virksomheder"
"dk_coa_6450","Other securities and equity","6450","asset_current","l10n_dk.account_tag_6450","False","Andre værdipapirer og kapitalandele"
"dk_coa_6510","Registered capital, etc.","6510","equity","l10n_dk.account_tag_6510","False","Registreret kapital mv."
"dk_coa_6520","Paid-up registered capital, etc.","6520","equity","l10n_dk.account_tag_6520","False","Indbetalt registreret kapital mv."
"dk_coa_6540","Share premium account","6540","equity","l10n_dk.account_tag_6540","False","Overkurs ved emission"
"dk_coa_6560","Revaluation reserve","6560","equity","l10n_dk.account_tag_6560","False","Reserve for opskrivninger"
"dk_coa_6580","Reserve for net revaluation reserve under the equity method","6580","equity","l10n_dk.account_tag_6580","False","Reserve for nettoopskrivning efter den indre værdis metode"
"dk_coa_6810","Reserve for loans and collateral","6810","equity","l10n_dk.account_tag_6810","False","Reserve for udlån og sikkerhedsstillelse"
"dk_coa_6830","Reserve for unpaid share capital and share premium account","6830","equity","l10n_dk.account_tag_6830","False","Reserve for ikke indbetalt virksomhedskapital og overkurs"
"dk_coa_6870","Other statutory reserves","6870","equity","l10n_dk.account_tag_6870","False","Øvrige lovpligtige reserver"
"dk_coa_6890","Statutory reserves","6890","equity","l10n_dk.account_tag_6890","False","Vedtægtsmæssige reserver"
"dk_coa_6910","Other reserves","6910","equity","l10n_dk.account_tag_6910","False","Øvrige reserver"
"dk_coa_6940","Profit brought forward","6940","equity","l10n_dk.account_tag_6940","False","Overført resultat"
"dk_coa_6960","Proposed dividend recognized in equity","6960","equity","l10n_dk.account_tag_6960","False","Foreslået udbytte indregnet under egenkapitalen"
"dk_coa_7010","Provisions for deferred taxes","7010","equity","l10n_dk.account_tag_7010","False","Hensættelser til udskudt skat"
"dk_coa_7020","Provisions for pensions and similar obligations","7020","equity","l10n_dk.account_tag_7020","False","Hensættelser til pensioner og lignende forpligtelser"
"dk_coa_7040","Other provisions","7040","equity","l10n_dk.account_tag_7040","False","Andre hensatte forpligtelser"
"dk_coa_7110","Amounts owed to credit institutions - long-term debt","7110","liability_non_current","l10n_dk.account_tag_7110","False","Gæld til kreditinstitutter - langfristet gæld"
"dk_coa_7120","Amounts owed to banks - long-term debt","7120","liability_non_current","l10n_dk.account_tag_7120","False","Gæld til banker - langfristet gæld"
"dk_coa_7160","Amounts owed to affiliated undertakings - long-term debt","7160","liability_non_current","l10n_dk.account_tag_7160","False","Gæld til tilknyttede virksomheder - langfristet gæld"
"dk_coa_7170","Amounts owed to participating interests - long-term debt","7170","liability_non_current","l10n_dk.account_tag_7170","False","Gæld til kapitalinteresser - langfristet gæld"
"dk_coa_7190","Other payables - long-term","7190","liability_non_current","l10n_dk.account_tag_7190","True","Anden gæld - langfristet"
"dk_coa_7210","Payables to shareholders and management - long-term debt","7210","liability_non_current","l10n_dk.account_tag_7210","True","Gæld til selskabsdeltagere og ledelse - langfristet gæld"
"dk_coa_7230","Deposits - long-term debt","7230","liability_non_current","l10n_dk.account_tag_7230","False","Deposita - langfristet gæld"
"dk_coa_7240","Lease liability - long-term debt","7240","liability_non_current","l10n_dk.account_tag_7240","False","Leasingforpligtelse - langfristet gæld"
"dk_coa_7250","Corporate income tax - long-term debt","7250","liability_non_current","l10n_dk.account_tag_7250","False","Selskabsskat - langfristet gæld"
"dk_coa_7310","Amounts owed to credit institutions - short-term debt","7310","liability_current","l10n_dk.account_tag_7310","False","Gæld til kreditinstitutter - kortfristet gæld"
"dk_coa_7330","Amounts owed to banks - short-term debt","7330","liability_current","l10n_dk.account_tag_7330","False","Gæld til banker - kortfristet gæld"
"dk_coa_7350","Other credit institutions","7350","liability_current","l10n_dk.account_tag_7350","False","Kreditinstitutter i øvrigt"
"dk_coa_7410","Prepayments received from customers","7410","liability_current","l10n_dk.account_tag_7410","False","Modtagne forudbetalinger fra kunder"
"dk_coa_7440","Suppliers of goods and services","7440","liability_payable","l10n_dk.account_tag_7440","True","Leverandører af varer og tjenesteydelser"
"dk_coa_7510","Amounts owed to affiliated enterprises - current liabilities","7510","liability_current","l10n_dk.account_tag_7510","False","Gæld til tilknyttede virksomheder - kortfristet gæld"
"dk_coa_7520","Amounts owed to participating interests - current liabilities","7520","liability_current","l10n_dk.account_tag_7520","False","Gæld til kapitalinteresser - kortfristet gæld"
"dk_coa_7590","Amounts owed to shareholders and management - current liabilities","7590","liability_current","l10n_dk.account_tag_7590","False","Gæld til selskabsdeltagere og ledelse - kortfristet gæld"
"dk_coa_7610","Deposits - current liabilities","7610","liability_current","l10n_dk.account_tag_7610","False","Deposita - kortfristet gæld"
"dk_coa_7630","Lease liability - current","7630","liability_current","l10n_dk.account_tag_7630","False","Leasingforpligtelse - kortfristet"
"dk_coa_7680","Sales tax","7680","liability_current","l10n_dk.account_tag_7680","False","Salgsmoms"
"dk_coa_7700","VAT on purchases of goods abroad, EU and non-EU","7700","liability_current","l10n_dk.account_tag_7700","False","Moms af varekøb udland, EU og ikke-EU"
"dk_coa_7720","VAT on purchases of services abroad, EU and non-EU","7720","liability_current","l10n_dk.account_tag_7720","False","Moms af ydelseskøb udland, EU og ikke-EU"
"dk_coa_7740","VAT on purchases","7740","liability_current","l10n_dk.account_tag_7740","False","Købsmoms"
"dk_coa_7760","Oil and liquefied petroleum gas tax","7760","liability_current","l10n_dk.account_tag_7760","False","Olie- og flaskegasafgift"
"dk_coa_7780","Electricity tax","7780","liability_current","l10n_dk.account_tag_7780","False","Elafgift"
"dk_coa_7800","Natural and town gas tax","7800","liability_current","l10n_dk.account_tag_7800","False","Naturgas- og bygasafgift"
"dk_coa_7810","Coal tax","7810","liability_current","l10n_dk.account_tag_7810","False","Kulafgift"
"dk_coa_7820","Water tax","7820","liability_current","l10n_dk.account_tag_7820","False","Vandafgift"
"dk_coa_7830","CO2 tax","7830","liability_current","l10n_dk.account_tag_7830","False","Co2-afgift"
"dk_coa_7840","VAT payable","7840","liability_current","l10n_dk.account_tag_7840","True","Skyldig moms"
"dk_coa_7860","Wages and salaries payable","7860","liability_current","l10n_dk.account_tag_7860","True","Skyldig løn og gager"
"dk_coa_7880","Bonus and profit-sharing payable","7880","liability_current","l10n_dk.account_tag_7880","True","Skyldig bonus og tantieme"
"dk_coa_7900","Holiday pay due","7900","liability_current","l10n_dk.account_tag_7900","False","Skyldige feriepenge"
"dk_coa_7920","A tax payable","7920","liability_current","l10n_dk.account_tag_7920","True","Skyldig A-skat"
"dk_coa_7940","AM contribution due","7940","liability_current","l10n_dk.account_tag_7940","False","Skyldigt AM-bidrag"
"dk_coa_7960","ATP contribution due","7960","liability_current","l10n_dk.account_tag_7960","False","Skyldigt ATP-bidrag"
"dk_coa_7980","AMP contribution due","7980","liability_current","l10n_dk.account_tag_7980","False","Skyldigt AMP-bidrag"
"dk_coa_8000","Other pension payable","8000","liability_current","l10n_dk.account_tag_8000","True","Anden skyldig pension"
"dk_coa_8040","Other payables","8040","liability_current","l10n_dk.account_tag_8040","True","Øvrig anden gæld"
"dk_coa_8070","Accrued expenses and deferred income","8070","liability_current","l10n_dk.account_tag_8070","False","Periodeafgrænsningsposter"

```

## File: data\template\account.fiscal.position-dk.csv

```csv
"id","name","auto_apply","vat_required","sequence","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","account_ids/account_src_id","account_ids/account_dest_id","name@da_DK"
"fiscal_position_template_dk_vat","Denmark (Company)","1","1","10","base.dk","","","","","","Danmark (Virksomhed)"
"fiscal_position_template_eu","EU countries (Private)","1","","11","","base.europe","","","","","EU lande (Privat)"
"fiscal_position_template_eu_taxid","EU countries (Company)","1","1","12","","base.europe","tax_s1","tax_s2","","","EU lande (Virksomhed)"
"","","","","","","","tax_s1y","tax_s3","","",""
"","","","","","","","tax_k1","tax_keuv1","","",""
"","","","","","","","tax_k1y","tax_keuy1","","",""
"","","","","","","","","","dk_coa_1010","dk_coa_1050",""
"","","","","","","","","","dk_coa_1610","dk_coa_1630",""
"fiscal_position_template_3lande","3rd countries (Company / Private)","1","","13","","","tax_s1","tax_s6","","","3. lande (Virksomhed / Privat)"
"","","","","","","","tax_s1y","tax_s6","","",""
"","","","","","","","tax_k1","tax_k%euv1","","",""
"","","","","","","","tax_k1y","tax_k%euy1","","",""
"","","","","","","","","","dk_coa_1010","dk_coa_1100",""
"","","","","","","","","","dk_coa_1610","dk_coa_1650",""

```

## File: data\template\account.group-dk.csv

```csv
"id","code_prefix_start","code_prefix_end","name","name@da_DK"
"dk_group_1000","1000","","PROFIT AND LOSS ACCOUNT","RESULTATOPGØRELSE"
"dk_group_1001","1001","","Net turnover","Nettoomsætning"
"dk_group_1400","1400","","Change in stocks of finished goods and work in progress","Ændring i lagre af færdigvarer og varer under fremstilling"
"dk_group_1500","1500","","Other operating income","Andre driftsindtægter"
"dk_group_1600","1600","","Costs of raw materials, consumables and services","Omkostninger til råvarer, hjælpematerialer og ydelser"
"dk_group_1820","1820","","Other external costs","Andre eksterne omkostninger"
"dk_group_1821","1821","","Sales costs","Salgsomkostninger"
"dk_group_2020","2020","","Premises costs","Lokaleomkostninger"
"dk_group_2220","2220","","Administrative costs","Administrationsomkostninger"
"dk_group_2841","2841","","Personnel costs","Personaleomkostninger"
"dk_group_2991","2991","","Depreciation, amortization and impairment of tangible and intangible fixed assets","Af- og nedskrivninger af materielle og immaterielle anlægsaktiver "
"dk_group_3120","3120","","Impairment losses on current assets","Nedskrivninger af omsætningsaktiver"
"dk_group_3150","3150","","Other operating costs","Andre driftsomkostninger"
"dk_group_3191","3191","","Income from investments","Indtægter af kapitalandele
"
"dk_group_3192","3192","","Income from investments in affiliated enterprises","Indtægter af kapitalandele i tilknyttede virksomheder"
"dk_group_3220","3220","","Income from participating interests","Indtægter af kapitalandele i kapitalinteresser"
"dk_group_3370","3370","","Income from other investments, securities and receivables that are fixed assets","Indtægter af andre kapitalandele, værdipapirer og tilgodehavender, der er anlægsaktiver"
"dk_group_3411","3411","","Other financial income from affiliated enterprises ","Andre finansielle indtægter fra tilknyttede virksomheder "
"dk_group_3451","3451","","Other financial income","Andre finansielle indtægter"
"dk_group_3541","3541","","Impairment of financial assets","Nedskrivning af finansielle aktiver"
"dk_group_3571","3571","","Financial costs","Finansielle omkostninger"
"dk_group_3601","3601","","Other financial costs","Øvrige finansielle omkostninger"
"dk_group_3702","3702","","Tax on profit for the year","Skat af årets resultat"
"dk_group_3791","3791","","Other taxes","Andre skatter"
"dk_group_5000","5000","","BALANCE","BALANCE"
"dk_group_5001","5001","","ASSETS","AKTIVER"
"dk_group_5002","5002","","Fixed assets","Anlægsaktiver"
"dk_group_5003","5003","","Intangible fixed assets","Immaterielle anlægsaktiver"
"dk_group_5004","5004","","Goodwill","Goodwill"
"dk_group_5071","5071","","Acquired intangible fixed assets","Erhvervede immaterielle anlægsaktiver"
"dk_group_5142","5142","","Tangible fixed assets","Materielle anlægsaktiver"
"dk_group_5150","5150","","Investment properties","Investeringsejendomme"
"dk_group_5231","5231","","Investment properties under construction","Investeringsejendomme under opførelse"
"dk_group_5311","5311","","Land and buildings","Grunde og bygninger"
"dk_group_5411","5411","","Production plant and machinery","Produktionsanlæg og maskiner"
"dk_group_5491","5491","","Furnishing of rented premises","Indretning af lejede lokaler"
"dk_group_5561","5561","","Other fixtures and fittings, tools and equipment","Andre anlæg, driftsmateriel og inventar"
"dk_group_5631","5631","","Tangible fixed assets under construction and prepayments for tangible fixed assets","Materielle anlægsaktiver under udførelse og forudbetalinger for materielle anlægsaktiver"
"dk_group_5701","5701","","Assets held under finance leases","Finansielt leasede aktiver"
"dk_group_5772","5772","","Financial fixed assets","Finansielle anlægsaktiver"
"dk_group_5773","5773","","Investments in affiliated enterprises","Kapitalandele i tilknyttede virksomheder"
"dk_group_5861","5861","","Long-term receivables from affiliated enterprises","Langfristede tilgodehavender hos tilknyttede virksomheder"
"dk_group_5891","5891","","Investments in participating interests","Kapitalandele i kapitalinteresser"
"dk_group_5961","5961","","Long-term receivables from participating interests","Langfristede tilgodehavender hos kapitalinteresser"
"dk_group_5991","5991","","Other securities and equity","Andre værdipapirer og kapitalandele"
"dk_group_6011","6011","","Other (non-current) receivables","Andre (langfristede) tilgodehavender"
"dk_group_6051","6051","","Receivables from enterprise participants and management","Tilgodehavender hos virksomhedsdeltagere og ledelse"
"dk_group_6073","6073","","Current assets","Omsætningsaktiver"
"dk_group_6074","6074","","Inventories","Varebeholdninger"
"dk_group_6075","6075","","Raw materials and consumables","Råvarer og hjælpematerialer"
"dk_group_6101","6101","","Goods in the course of manufacture","Varer under fremstilling"
"dk_group_6131","6131","","Manufactured and traded goods","Fremstillede varer og handelsvarer"
"dk_group_6161","6161","","Prepayments for goods","Forudbetalinger for varer"
"dk_group_6182","6182","","Receivables","Tilgodehavender"
"dk_group_6183","6183","","Trade and other receivables","Tilgodehavender fra salg og tjenesteydelser"
"dk_group_6211","6211","","Current receivables from affiliated enterprises","Kortfristede tilgodehavender hos tilknyttede virksomheder"
"dk_group_6261","6261","","Work in progress on behalf of third parties","Igangværende arbejder for fremmed regning"
"dk_group_6281","6281","","Other receivables (current)","Andre tilgodehavender (kortfristede)"
"dk_group_6341","6341","","Claims for payment of company capital and share premium","Krav på indbetaling af virksomhedskapital og overkurs"
"dk_group_6361","6361","","Current receivables from enterprise participants and management","Kortfristede tilgodehavender hos virksomhedsdeltagere og ledelse"
"dk_group_6381","6381","","Prepayments and accrued income","Periodeafgrænsningsposter"
"dk_group_6412","6412","","Securities and equity investments","Værdipapirer og kapitalandele"
"dk_group_6413","6413","","Investments in affiliated enterprises","Kapitalandele i tilknyttede virksomheder"
"dk_group_6431","6431","","Other securities and equity","Andre værdipapirer og kapitalandele"
"dk_group_6462","6462","","Cash and cash equivalents","Likvide beholdninger"
"dk_group_6500","6500","","LIABILITIES","PASSIVER"
"dk_group_6501","6501","","Equity capital","Egenkapital"
"dk_group_6502","6502","","Working capital","Virksomhedskapital"
"dk_group_6531","6531","","Share premium on issue","Overkurs ved emission"
"dk_group_6551","6551","","Revaluation reserve","Reserve for opskrivninger"
"dk_group_6571","6571","","Reserve for net revaluation under the equity method","Reserve for nettoopskrivning efter den indre værdis metode"
"dk_group_6591","6591","","Other reserves","Andre reserver"
"dk_group_6951","6951","","Proposed dividend recognized in equity","Foreslået udbytte indregnet under egenkapitalen"
"dk_group_6972","6972","","Provisions for liabilities and charges","Hensatte forpligtelser"
"dk_group_6973","6973","","Provision for deferred taxes","Hensættelse til udskudt skat"
"dk_group_7031","7031","","Other provisions","Andre hensatte forpligtelser"
"dk_group_7052","7052","","Long-term debt","Langfristet gæld"
"dk_group_7053","7053","","Amounts owed to credit institutions - long-term debt","Gæld til kreditinstitutter - langfristet gæld"
"dk_group_7131","7131","","Amounts owed to affiliated enterprises - long-term debt","Gæld til tilknyttede virksomheder - langfristet gæld"
"dk_group_7181","7181","","Other debts, including taxes and social security contributions payable","Anden gæld, herunder skyldige skatter og skyldige bidrag til social sikring"
"dk_group_7262","7262","","Short-term debt","Kortfristet gæld"
"dk_group_7263","7263","","Amounts owed to credit institutions - short-term debt","Gæld til kreditinstitutter - kortfristet gæld"
"dk_group_7361","7361","","Prepayments received from customers","Modtagne forudbetalinger fra kunder"
"dk_group_7421","7421","","Suppliers of goods and services","Leverandører af varer og tjenesteydelser"
"dk_group_7451","7451","","Amounts owed to affiliated enterprises - short-term debt","Gæld til tilknyttede virksomheder - kortfristet gæld"
"dk_group_7531","7531","","Other debts,  including taxes and social security contributions payable","Anden gæld, herunder skyldige skatter og skyldige bidrag til social sikring"
"dk_group_8051","8051","","Prepayments and accrued income","Periodeafgrænsningsposter"

```

## File: data\template\account.tax-dk.csv

```csv
"id","sequence","name","description","invoice_label","amount","amount_type","type_tax_use","active","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","description@da_DK","invoice_label@da_dk"
"tax_s1","5","S-DK-25","Sales subject to VAT (DK), 25% VAT","25 %","25.0","percent","sale","True","base","invoice","","","","Momspligtige salg (DK), 25% moms","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7680","+UM","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7680","-UM","","",""
"tax_s1y","5","S-DK-Y-25","Sales subject to VAT (DK) (Services), 25% VAT","25 %","25.0","percent","sale","True","base","invoice","","","","Momspligtige salg (DK), 25% moms (Ydelser)","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7680","+UM","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7680","-UM","","",""
"tax_s0","6","S-DK-0","Sales subject to VAT (DK) 0% VAT","0 %","0.0","percent","sale","True","base","invoice","","+R-C-UR","","Momspligtige salg (DK) 0% moms","0 %"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","-R-C-UR","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax_s%","7","S-DK-U","Outside the scope of the VAT Act","0 %","0.0","percent","sale","True","base","invoice","","","","Udenfor momslovens anvendelsområde","0 %"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax_smf","8","S-DK-MF","VAT-exempt sale","0 %","0.0","percent","sale","True","base","invoice","","","","Momsfritaget salg ","0 %"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax_sbrugt","9","S-DK-Brugtmoms","Used VAT","25 %","25.0","percent","sale","False","base","invoice","","","","Brugtmoms",""
"","","","","","","","","","tax","invoice","dk_coa_7680","+UM","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7680","-UM","","",""
"tax_smargin","10","S-DK-Marginmoms","Margin VAT","25 %","25.0","percent","sale","False","base","invoice","","","","Marginmoms","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7680","+UM","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7680","-UM","","",""
"tax_smotor","11","S-DK-brugtbil","Used passenger motor vehicles","25 %","25.0","percent","sale","False","base","invoice","","","","Brugte personmotorkøretøjer","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7680","+UM","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7680","-UM","","",""
"tax_sleasing","12","S-DK-leasingbiler mm","Rental companies, driving schools and tow trucks","20 %","20.0","percent","sale","False","base","invoice","","","","Udlejningsvirksomheder, køreskoler og demobiler","20 %"
"","","","","","","","","","tax","invoice","dk_coa_7680","+UM","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7680","-UM","","",""
"tax_skunstnere","13","S-DK-kunstnere","Artists' first sale","5 %","5.0","percent","sale","False","base","invoice","","","","Kunstneres førstegangssalg","5 %"
"","","","","","","","","","tax","invoice","dk_coa_7680","+UM","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7680","-UM","","",""
"tax_slokal","14","S-LokalMoms","Local sales other countries with VAT","0 %","0","percent","sale","False","base","invoice","","","","Lokal salg andre lande med moms","0 %"
"","","","","","","","","","tax","invoice","dk_coa_7680","+UM","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7680","-UM","","",""
"tax_s2","16","S-EU-V-0","Sale of goods B2B (EU), 0% VAT","0 %","0.0","percent","sale","True","base","invoice","","+R-B-MR","","Varesalg B2B (EU), 0% moms   ","0 %"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","-R-B-MR","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax_s7","17","S-EU-V-0%","Sale of goods B2B (EU), 0% VAT","0 %","0.0","percent","sale","True","base","invoice","","+R-B-UR","","Varesalg B2B (EU), 0% moms   ","0 %"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","-R-B-UR","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax_s3","18","S-EU-Y-0","Service sales B2B (EU), 0% VAT","0 %","0.0","percent","sale","True","base","invoice","","+R-C-MR","","Ydelsessalg B2B (EU), 0% moms","0 %"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","-R-C-MR","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax_s8","19","S-EU-T-0","Triangular sales B2B (EU), 0% VAT","0 %","0.0","percent","sale","True","base","invoice","","+R-D-MR","","Trianguleringssalg B2B (EU), 0% moms","0 %"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","-R-D-MR","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax_smfeu","20","S-EU-MF","VAT-exempt sales (EU)","0 %","0.0","percent","sale","True","base","invoice","","","","Momsfritaget salg (EU)","0 %"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax_s4","21","S-%EU-V-0","Sales of goods B2B (% EU), 0% VAT","0 %","0.0","percent","sale","True","base","invoice","","+R-C-UR","","Varesalg B2B (% EU), 0% moms","0 %"
"","","","","","","","","","tax","invoice","dk_coa_7740","","","",""
"","","","","","","","","","tax","invoice","dk_coa_7700","","-100","",""
"","","","","","","","","","base","refund","","-R-C-UR","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","","","",""
"","","","","","","","","","tax","refund","dk_coa_7700","","-100","",""
"tax_s5","22","S-%EU-Y-0","Service sales B2B (% EU), 0% VAT","0 %","0.0","percent","sale","True","base","invoice","","","","Ydelsessalg B2B (% EU), 0% moms","0 %"
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7720","-MYUOB","-100","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"","","","","","","","","","tax","refund","dk_coa_7720","+MYUOB","-100","",""
"tax_s6","23","S-%EU-MF","VAT-exempt sales (outside the EU) § 13","0 %","0.0","percent","sale","True","base","invoice","","","","Momsfritaget salg (udenfor EU) § 13","0 %"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax_k1","29","K-DK-Fradrag","Purchase VAT (DK), full deduction (100%)","25 %","25.0","percent","purchase","True","base","invoice","","","","Indgående moms (DK), fuld fradrag (100%)","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"tax_k1y","29","K-DK-Y-Fradrag","Purchase VAT (DK), full deduction (100%) (Services)","25 %","25.0","percent","purchase","True","base","invoice","","","","Indgående moms (DK), fuld fradrag (100%) (Ydelser)","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"tax_k3","30","K-Dk-SkønsmæssigFradrag","Purchase VAT (DK), discretionary deduction","25 %","25.0","percent","purchase","True","base","invoice","","","","",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"tax_k0","31","K-DK-IngenFradrag","VAT exempt purchase, No deduction, no VAT on purchase","0 %","0.0","percent","purchase","True","base","invoice","","","","Momsfritaget køb, Ingen fradrag, ingen moms på køb ","0 %"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"","","","","","","","","","","","","","","",""
"tax_k7","32","K-DK-VejFradrag","Purchase VAT (DK), statutory partial deduction (2/3 deduction)","25 %","25.0","percent","purchase","False","base","invoice","","","","Indgående moms (DK), lovbestemt delvist fradrag (2/3 fradrag)","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","66.67","",""
"","","","","","","","","","tax","invoice","","","33.33","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","66.67","",""
"","","","","","","","","","tax","refund","","","33.33","",""
"tax_k5a","33","K-DK-FastnettelefonFuld","Purchase VAT (DK), statutory partial deduction (50%)","25 %","25.0","percent","purchase","True","base","invoice","","","","Indgående moms (DK), lovbestemt delvist fradrag (50%)","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","50","",""
"","","","","","","","","","tax","invoice","","","50","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","50","",""
"","","","","","","","","","tax","refund","","","50","",""
"tax_k5b","34","K-DK-FastnettelefonProRata","Purchase VAT (DK), statutory partial deduction (50% x pro rata)","25 %","25.0","percent","purchase","True","base","invoice","","","","Indgående moms (DK), lovbestemt delvist fradrag (50% x pro rata)","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","50","",""
"","","","","","","","","","tax","invoice","","","50","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","50","",""
"","","","","","","","","","tax","refund","","","50","",""
"tax_kl1","35","K-DK-GulpladebilFradragPrivat","Leasing and operation of a yellow plate car, (private and mixed purposes as well as businesses)","25 %","25.0","percent","purchase","True","base","invoice","","","","Leasing og drift af gulpladebil, (private og blandede formål samt erhverv)","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","33.33","",""
"","","","","","","","","","tax","invoice","","","66.67","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","33.33","",""
"","","","","","","","","","tax","refund","","","66.67","",""
"tax_k25a","36","K-DK-RepræsentationFradragFuld","Representation VAT, partly deductible (25%)","25 %","25.0","percent","purchase","True","base","invoice","","","","Repræsentationsmoms, delvist fradragsberettiget (25%)","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","25","",""
"","","","","","","","","","tax","invoice","","","75","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","25","",""
"","","","","","","","","","tax","refund","","","75","",""
"tax_k25b","37","K-DK-RepræsentationFradragProRata","Representation VAT, partly deductible (25% x pro rata)","25 %","25.0","percent","purchase","True","base","invoice","","","","Repræsentationsmoms, delvist fradragsberettiget (25% x pro rata)","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","25","",""
"","","","","","","","","","tax","invoice","","","75","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","25","",""
"","","","","","","","","","tax","refund","","","75","",""
"tax_kl2","38","K-DK-LeasinghvidpladebilFradrag","Leasing of white plate car in min. 6 months (invoice)","25 %","25.0","percent","purchase","True","base","invoice","","","","Leasing af hvidpladebil i min. 6 mdr.  (faktura)","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","25","",""
"","","","","","","","","","tax","invoice","","","75","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","25","",""
"","","","","","","","","","tax","refund","","","75","",""
"tax_kl3","38","K-DK-LeasinghvidpladebilFradragProrata","Leasing of white plate car in min. 6 months (invoice x pro rata)","25 %","25.0","percent","purchase","True","base","invoice","","","","Leasing af hvidpladebil i min. 6 mdr.  (faktura x pro rata)","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","25","",""
"","","","","","","","","","tax","invoice","","","75","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","25","",""
"","","","","","","","","","tax","refund","","","75","",""
"tax_k2","39","K-DK-DelvisFradrag","Purchase VAT (DK), Partial deduction (pro rata)","25 %","25.0","percent","purchase","True","base","invoice","","","","Indgående moms (DK), Delvis fradrag (pro rata)","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"tax_k6","40","K-DK-SektorFradrag","Purchase VAT (DK), sector-based right of deduction","25 %","25.0","percent","purchase","True","base","invoice","","","","Indgående moms (DK), sektorbaseret fradragsret","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"tax_k4","41","K-DK-ArealFradrag","Area-based deductibility real estate","25 %","25.0","percent","purchase","True","base","invoice","","","","Arealbaseret fradragsret fast ejendom","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"tax_kbrugtmoms","42","K-DK-Brugtmoms","Used VAT","0 %","0.0","percent","purchase","False","base","invoice","","","","Brugtmoms","25 %"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax_kmarginmoms","43","K-DK-Marginmoms","Margin VAT","0 %","0.0","percent","purchase","False","base","invoice","","","","Marginmoms","25 %"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax_k_brugtbil","44","K-DK-brugtbil","Used passenger motor vehicles","0 %","0.0","percent","purchase","False","base","invoice","","","","Brugte personmotorkøretøjer","25 %"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax_kleasingbil","45","K-DK-leasingbiler mm","Rental companies, driving schools and tow trucks","0 %","0.0","percent","purchase","False","base","invoice","","","","Udlejningsvirksomheder, køreskoler og demobiler","20 %"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax_kdk01","46","K-DKO-Fradrag","DK reverse obligation to pay, Purchase VAT, fully deductible (100%)","","25.0","percent","purchase","False","base","invoice","","","","DK omvendt betalingspligt, Indgående moms, fuldt fradragsberettiget (100%)",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7680","-UM","-100","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-UM","","",""
"","","","","","","","","","tax","refund","dk_coa_7680","+KM","-100","",""
"tax_kdko2","47","K-DKO-DelvisFradrag","DK reverse obligation to pay, Purchase VAT, Partial deduction (pro rata)","","25.0","percent","purchase","False","base","invoice","","","","DK omvendt betalingspligt, Indgående moms, Delvis fradrag (pro rata)",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7680","-UM","-100","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-UM","","",""
"","","","","","","","","","tax","refund","dk_coa_7680","+KM","-100","",""
"tax_kdko3","48","K-DKO-SkønsmæssigFradrag","DK reverse payment obligation, Purchase VAT, discretionary deduction","","25.0","percent","purchase","False","base","invoice","","","","DK omvendt betalingspligt, Indgående moms, skønsmæssigt fradrag",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7680","-UM","-100","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-UM","","",""
"","","","","","","","","","tax","refund","dk_coa_7680","+KM","-100","",""
"tax_kdko0","49","K-DKO-IngenFradrag","DK reverse payment obligation, No deduction","","25.0","percent","purchase","False","base","invoice","","","","DK omvendt betalingspligt, Ingen fradrag",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7680","-UM","-100","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-UM","","",""
"","","","","","","","","","tax","refund","dk_coa_7680","+KM","-100","",""
"tax_kdko4","50","K-DKO-ArealFradrag","DK reverse payment obligation, Area-based right to deduct real estate","","25.0","percent","purchase","False","base","invoice","","","","DK omvendt betalingspligt, Arealbaseret fradragsret fast ejendom",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7680","-UM","-100","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-UM","","",""
"","","","","","","","","","tax","refund","dk_coa_7680","+KM","-100","",""
"tax_klokalmoms","14","K-LokalMoms","Purchases with local VAT","0 %","0","percent","purchase","True","base","invoice","","","","Køb med lokal moms",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax_kdkoi","52","K-DK-refusion","Purchases covered by the public refund scheme","25 %","25.0","percent","purchase","False","base","invoice","","","","Køb omfattet af det offentliges refusionsordning",""
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax_keuv1","53","K-EU-V-Fradrag","Purchase of goods, acquisition VAT (EU), fully deductible (100%)","","25.0","percent","purchase","True","base","invoice","","+R-A-V","","Varekøb, erhvervelsesmoms (EU), fuldt fradragsberettiget (100%)",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7700","-MVU","-100","",""
"","","","","","","","","","base","refund","","-R-A-V","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"","","","","","","","","","tax","refund","dk_coa_7700","+MVU","-100","",""
"tax_keuv3","54","K-EU-V-SkønsmæssigFradrag","Purchase of goods, acquisition VAT (EU), discretionary deduction","","25.0","percent","purchase","True","base","invoice","","+R-A-V","","Varekøb, erhvervelsesmoms (EU), skønsmæssigt fradrag",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7700","-MVU","-100","",""
"","","","","","","","","","base","refund","","-R-A-V","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"","","","","","","","","","tax","refund","dk_coa_7700","+MVU","-100","",""
"tax_keuv0","55","K-EU-V-IngenFradrag","Purchase of goods, acquisition VAT (EU), discretionary deduction","","25.0","percent","purchase","True","base","invoice","","+R-A-V","","Varekøb, erhvervelsesmoms (EU), skønsmæssigt fradrag",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7700","-MVU","-100","",""
"","","","","","","","","","base","refund","","-R-A-V","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"","","","","","","","","","tax","refund","dk_coa_7700","+MVU","-100","",""
"tax_keuv2","56","K-EU-V-DelvisFradrag","Purchase of goods, acquisition VAT (EU), Partial deduction (pro rata)","","25.0","percent","purchase","True","base","invoice","","+R-A-V","","Varekøb, erhvervelsesmoms (EU), Delvis fradrag (pro rata)",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7700","-MVU","-100","",""
"","","","","","","","","","base","refund","","-R-A-V","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"","","","","","","","","","tax","refund","dk_coa_7700","+MVU","-100","",""
"tax_keuv4","57","K-EU-V-ArealFradrag","Purchase of goods, acquisition VAT (EU), area-based real estate deductibility","","25.0","percent","purchase","True","base","invoice","","+R-A-V","","Varekøb, erhvervelsesmoms (EU), Arealbaseret fradragsret fast ejendom",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7700","-MVU","-100","",""
"","","","","","","","","","base","refund","","-R-A-V","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"","","","","","","","","","tax","refund","dk_coa_7700","+MVU","-100","",""
"tax_keuy1","58","K-EU-Y-Fradrag","Purchase of services, reverse obligation to pay (EU), fully deductible (100%)","","25.0","percent","purchase","True","base","invoice","","+R-A-Y","","Ydelseskøb, omvendt betalingspligt (EU), fuldt fradragsberettiget (100%)",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7720","-MYUOB","-100","",""
"","","","","","","","","","base","refund","","-R-A-Y","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"","","","","","","","","","tax","refund","dk_coa_7720","+MYUOB","-100","",""
"tax_keuy3","59","K-EU-Y-SkønsmæssigFradrag","Purchase of services, reverse obligation to pay (EU), discretionary deduction","","25.0","percent","purchase","True","base","invoice","","+R-A-Y","","Ydelseskøb, omvendt betalingspligt (EU), skønsmæssigt fradrag",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7720","-MYUOB","-100","",""
"","","","","","","","","","base","refund","","-R-A-Y","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"","","","","","","","","","tax","refund","dk_coa_7720","+MYUOB","-100","",""
"tax_keuy0","60","K-EU-Y-IngenFradrag","Purchase of services, reverse obligation to pay (EU), no deduction","","25.0","percent","purchase","True","base","invoice","","+R-A-Y","","Ydelseskøb, omvendt betalingspligt (EU), ingen fradrag",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7720","-MYUOB","-100","",""
"","","","","","","","","","base","refund","","-R-A-Y","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"","","","","","","","","","tax","refund","dk_coa_7720","+MYUOB","-100","",""
"tax_keuy2","61","K-EU-Y-DelvisFradrag","Purchase of services, reverse obligation to pay (EU), Partial deduction","","25.0","percent","purchase","True","base","invoice","","+R-A-Y","","Ydelseskøb, omvendt betalingspligt (EU), Delvis fradrag",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7720","-MYUOB","-100","",""
"","","","","","","","","","base","refund","","-R-A-Y","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"","","","","","","","","","tax","refund","dk_coa_7720","+MYUOB","-100","",""
"tax_keuy4","62","K-EU-Y-ArealFradrag","Purchase of services, reverse payment obligation (EU), Area-based deductibility real estate","","25.0","percent","purchase","True","base","invoice","","+R-A-Y","","Ydelseskøb, omvendt betalingspligt (EU), Arealbaseret fradragsret fast ejendom",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7720","-MYUOB","-100","",""
"","","","","","","","","","base","refund","","-R-A-Y","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"","","","","","","","","","tax","refund","dk_coa_7720","+MYUOB","-100","",""
"tax_kleu2","63","K-EU-LeasinghvidpladebilFradrag","Leasing of white plate car in min. 6 months abroad (invoice)","25 %","25.0","percent","purchase","True","base","invoice","","","","Leasing af hvidpladebil i min. 6 mdr.  Udlandet (faktura)","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","25","",""
"","","","","","","","","","tax","invoice","","","75","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","25","",""
"","","","","","","","","","tax","refund","","","75","",""
"tax_kleu3","64","K-EU-LeasinghvidpladebilDelvisFradrag","Leasing of white plate car in min. 6 months abroad (invoice x pro rata)","25 %","25.0","percent","purchase","True","base","invoice","","","","Leasing af hvidpladebil i min. 6 mdr.  Udlandet (faktura x pro rata)","25 %"
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","25","",""
"","","","","","","","","","tax","invoice","","","75","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","25","",""
"","","","","","","","","","tax","refund","","","75","",""
"tax_keuyl1","65","K-EU-Y-GulpladebilFradragPrivat","Leasing of a yellow plate car from a foreign lessor, (private and mixed purposes as well as businesses)","","25.0","percent","purchase","True","base","invoice","","+R-A-Y","","Leasing af gulpladebil fra udenlandsk leasinggiver, (private og blandede formål samt erhverv)",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7720","-MYUOB","-100","",""
"","","","","","","","","","base","refund","","-R-A-Y","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"","","","","","","","","","tax","refund","dk_coa_7720","+MYUOB","-100","",""
"tax_keumf","66","K-EU-Momsfritaget","VAT-exempt purchases of goods and services (EU)","0 %","0.0","percent","purchase","True","base","invoice","","","","Momsfritagede køb af varer og ydelser (EU) ","0 %"
"","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","","","","",""
"tax_k%euv1","67","K-%EU-V-Fradrag","Purchase of goods, Import VAT (% EU), fully deductible (100%)","","25.0","percent","purchase","True","base","invoice","","","","Varekøb, Importmoms (% EU), fuldt fradragsberettiget (100%)",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+MVU","","",""
"","","","","","","","","","tax","invoice","dk_coa_7700","-KM","-100","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-MVU","","",""
"","","","","","","","","","tax","refund","dk_coa_7700","+KM","-100","",""
"tax_k%euv3","68","K-%EU-V-SkønsmæssigFradrag","Purchase of goods, import VAT (% EU), discretionary deduction","","25.0","percent","purchase","True","base","invoice","","","","Varekøb, importmoms (% EU), skønsmæssigt fradrag",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+MVU","","",""
"","","","","","","","","","tax","invoice","dk_coa_7700","-KM","-100","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-MVU","","",""
"","","","","","","","","","tax","refund","dk_coa_7700","+KM","-100","",""
"tax_k%euv0","69","K-%EU-V-IngenFradrag","Purchase of goods, import VAT (% EU), no deduction","","25.0","percent","purchase","True","base","invoice","","","","Varekøb, importmoms (% EU), ingen fradrag",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+MVU","","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-MVU","","",""
"tax_k%euv2","69","K-%EU-V-DelvisFradrag","Purchase of goods, import VAT (% EU), Partial deduction (pro rata)","","25.0","percent","purchase","True","base","invoice","","","","Varekøb, importmoms (% EU), Delvis fradrag (pro rata)",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+MVU","","",""
"","","","","","","","","","tax","invoice","dk_coa_7700","-KM","-100","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-MVU","","",""
"","","","","","","","","","tax","refund","dk_coa_7700","+KM","-100","",""
"tax_k%euv4","70","K-%EU-V-ArealFradrag","Purchase of goods, import VAT (% EU), Area-based real estate deduction","","25.0","percent","purchase","True","base","invoice","","","","Varekøb, importmoms (% EU), Arealbaseret fradragsret fast ejendom",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+MVU","","",""
"","","","","","","","","","tax","invoice","dk_coa_7700","-KM","-100","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-MVU","","",""
"","","","","","","","","","tax","refund","dk_coa_7700","+KM","-100","",""
"tax_k%euy1","71","K-%EU-Y-Fradrag","Purchase of services, reverse payment obligation (% EU), fully deductible (100%)","","25.0","percent","purchase","True","base","invoice","","","","Ydelseskøb, omvendt betalingspligt (% EU), fuldt fradragsberettiget (100%)",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7720","-MYUOB","-100","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"","","","","","","","","","tax","refund","dk_coa_7720","+MYUOB","-100","",""
"tax_k%euy3","72","K-%EU-DK-Y-SkønsmæssigFradrag","Service purchase, reverse payment obligation (% EU), discretionary deduction","","25.0","percent","purchase","True","base","invoice","","","","Ydelseskøb, omvendt betalingspligt (% EU), skønsmæssigt fradrag",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7720","-MYUOB","-100","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"","","","","","","","","","tax","refund","dk_coa_7720","+MYUOB","-100","",""
"tax_k%euy0","73","K-%EU-DK-Y-IngenFradrag","Purchase of services, reverse payment obligation (% EU), no deductions","","25.0","percent","purchase","True","base","invoice","","","","Ydelseskøb, omvendt betalingspligt (% EU), ingen fradrag",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7720","-MYUOB","-100","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"","","","","","","","","","tax","refund","dk_coa_7720","+MYUOB","-100","",""
"tax_k%euy2","74","K-%EU-Y-DelvisFradrag","Purchase of services, reverse obligation to pay (% EU), Partial deduction","","25.0","percent","purchase","True","base","invoice","","","","Ydelseskøb, omvendt betalingspligt (% EU), Delvis fradrag",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7720","-MYUOB","-100","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"","","","","","","","","","tax","refund","dk_coa_7720","+MYUOB","-100","",""
"tax_k%euy4","75","K-%EU-Y-ArealFradrag","Purchase of services, reverse payment obligation (% EU), Area-based deductibility real estate","","25.0","percent","purchase","True","base","invoice","","","","Ydelseskøb, omvendt betalingspligt (% EU), Arealbaseret fradragsret fast ejendom",""
"","","","","","","","","","tax","invoice","dk_coa_7740","+KM","","",""
"","","","","","","","","","tax","invoice","dk_coa_7720","-MYUOB","-100","",""
"","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","tax","refund","dk_coa_7740","-KM","","",""
"","","","","","","","","","tax","refund","dk_coa_7720","+MYUOB","-100","",""

```

## File: data\template\account.tax.group-dk.csv

```csv
"id","country_id","name","tax_payable_account_id","tax_receivable_account_id"
"default_tax_group","base.dk","Taxes","dk_coa_7840","dk_coa_6320"

```

## File: migrations\1.2\end-migrate.py

```python
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'dk')], order="parent_path"):
        env['account.chart.template'].try_loading('dk', company)

```

## File: migrations\1.3\end-migrate.py

```python
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'dk')], order="parent_path"):
        tax = env.ref(f'account.{company.id}_tax_keumf', raise_if_not_found=False)
        if tax:
            tax.type_tax_use = 'purchase'

```

## File: models\account_account.py

```python
from collections import defaultdict

from odoo import api, models, _
from odoo.exceptions import UserError


class AccountAccount(models.Model):
    _inherit = ['account.account']
    
    @api.ondelete(at_uninstall=False)
    def _unlink_bank_cash_accounts(self):
        nb_account_to_delete_per_company = defaultdict(self.env['account.account'].browse)
        for account in self:
            for company in account.company_ids:
                if company.country_code == 'DK':
                    nb_account_to_delete_per_company[company] |= account

        if not nb_account_to_delete_per_company:
            return

        grouped_counts = self.read_group(
            domain=[('company_ids.account_fiscal_country_id.code', '=', 'DK'), ('account_type', '=', 'asset_cash')],
            fields=['company_ids', 'id:count'],
            groupby=['company_ids'],
        )
        nb_account_per_company = {self.env['res.company'].browse(entry['company_ids'][0]): entry['company_ids_count'] for entry in grouped_counts}

        for company_id, count in nb_account_per_company.items():
            nb_to_delete = sum(1 for account in nb_account_to_delete_per_company.get(company_id) if account.account_type == 'asset_cash')
            if count - nb_to_delete < 1:
                raise UserError(_("You must keep at least one bank and cash account for %(company)s!", company=company_id.name))

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, Command, models


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    @api.model
    def _prepare_liquidity_account_vals(self, company, code, vals):
        # OVERRIDE
        account_vals = super()._prepare_liquidity_account_vals(company, code, vals)

        if company.account_fiscal_country_id.code == 'DK':
            # Ensure the newly liquidity accounts have the right account tag in order to be part
            # of the Danish financial reports.
            account_vals.setdefault('tag_ids', [])
            if vals.get('type') == 'bank':
                account_vals['tag_ids'].append(Command.link(self.env.ref('l10n_dk.account_tag_6481').id))
            elif vals.get('type') == 'cash':
                account_vals['tag_ids'].append(Command.link(self.env.ref('l10n_dk.account_tag_6471').id))

        return account_vals

```

## File: models\res_partner.py

```python
from odoo import api, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    @api.depends('vat', 'country_id')
    def _compute_company_registry(self):
        # OVERRIDE
        # In Denmark, if you have a VAT number, it's also your company registry (CVR) number
        super()._compute_company_registry()
        for partner in self.filtered(lambda p: p.country_id.code == 'DK' and p.vat):
            vat_country, vat_number = self._split_vat(partner.vat)
            if vat_country.isnumeric():
                vat_country = 'dk'
                vat_number = partner.vat
            if vat_country == 'dk' and self.simple_vat_check(vat_country, vat_number):
                partner.company_registry = vat_number

```

## File: models\template_dk.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('dk')
    def _get_dk_template_data(self):
        return {
            'property_account_receivable_id': 'dk_coa_6190',
            'property_account_payable_id': 'dk_coa_7440',
            'property_account_expense_categ_id': 'dk_coa_1610',
            'property_account_income_categ_id': 'dk_coa_1010',
            'code_digits': '4',
        }

    @template('dk', 'res.company')
    def _get_dk_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.dk',
                'bank_account_code_prefix': '648',
                'cash_account_code_prefix': '647',
                'transfer_account_code_prefix': '683',
                'account_default_pos_receivable_account_id': 'dk_coa_6190',
                'income_currency_exchange_account_id': 'dk_coa_3610',
                'expense_currency_exchange_account_id': 'dk_coa_3610',
                'account_journal_early_pay_discount_loss_account_id': 'dk_coa_2720',
                'account_journal_early_pay_discount_gain_account_id': 'dk_coa_2720',
                'account_sale_tax_id': 'tax_s1',
                'account_purchase_tax_id': 'tax_k1',
                'check_account_audit_trail': True,
            },
        }

    def _setup_utility_bank_accounts(self, template_code, company, template_data):
        super()._setup_utility_bank_accounts(template_code, company, template_data)
        if template_code == 'dk':
            company.account_journal_suspense_account_id.tag_ids = self.env.ref('l10n_dk.account_tag_6482')
            company.transfer_account_id.tag_ids = self.env.ref('l10n_dk.account_tag_6831')

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_account
from . import template_dk
from . import account_journal
from . import res_partner

```

