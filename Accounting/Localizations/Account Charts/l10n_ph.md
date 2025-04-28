# Odoo Module: l10n_ph

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    "name": "Philippines - Accounting",
    "summary": """
        This is the module to manage the accounting chart for The Philippines.
    """,
    "category": "Accounting/Localizations/Account Charts",
    "version": "1.0",
    "author": "Odoo PS",
    "website": "https://www.odoo.com",
    "depends": [
        "account",
        "base_vat",
        "l10n_multilang",
    ],
    "data": [
        "data/account_chart_template_data.xml",
        "data/account.account.template.csv",
        "data/account_chart_template_post_data.xml",
        "data/account_tax_group.xml",
        "data/account_tax_template.xml",
        "data/account_fiscal_position_template.xml",
        "data/account_fiscal_position_tax_template.xml",
        "data/account_chart_template_configure_data.xml",
        "wizard/generate_2307_wizard_views.xml",
        "views/account_move_views.xml",
        "views/account_payment_views.xml",
        "views/account_tax_views.xml",
        "views/res_partner_views.xml",
        "security/ir.model.access.csv",
    ],
    "license": "LGPL-3",
    "icon": "/base/static/img/country_flags/ph.png",
}

```

## File: data\account.account.template.csv

```csv
id,name,code,account_type,chart_template_id/id,reconcile
l10n_ph_100000,Bank Suspense Account,100000,asset_current,l10n_ph_chart_template,False
l10n_ph_110000,Accounts Receivable,110000,asset_receivable,l10n_ph_chart_template,True
l10n_ph_110003,Accounts Receivable (POS),110003,asset_receivable,l10n_ph_chart_template,True
l10n_ph_110100,Advances to Suppliers,110100,asset_prepayments,l10n_ph_chart_template,False
l10n_ph_110101,Deposits,110101,asset_prepayments,l10n_ph_chart_template,False
l10n_ph_110102,Employees Receivable-Cash Advance,110102,asset_current,l10n_ph_chart_template,False
l10n_ph_110103,Employees Receivable-Loan,110103,asset_current,l10n_ph_chart_template,False
l10n_ph_110104,ER - Others,110104,asset_current,l10n_ph_chart_template,False
l10n_ph_110105,Prepaid Insurance,110105,asset_prepayments,l10n_ph_chart_template,False
l10n_ph_110106,Prepaid Pension - contribution,110106,asset_prepayments,l10n_ph_chart_template,False
l10n_ph_110200,Prepaid Tax - Tax Credit Certificate,110200,asset_current,l10n_ph_chart_template,False
l10n_ph_110201,Input VAT 12%,110201,asset_current,l10n_ph_chart_template,False
l10n_ph_110202,Deferred Input VAT 12%,110202,asset_current,l10n_ph_chart_template,False
l10n_ph_110203,Deferred Tax Asset,110203,asset_current,l10n_ph_chart_template,False
l10n_ph_110205,Creditable Income Tax,110205,asset_current,l10n_ph_chart_template,False
l10n_ph_110206,Tax Withheld by Customer,110206,asset_current,l10n_ph_chart_template,False
l10n_ph_110207,Deferred Tax Withheld by Customer,110207,asset_current,l10n_ph_chart_template,False
l10n_ph_110300,Inventory,110300,asset_current,l10n_ph_chart_template,False
l10n_ph_110301,Purchases,110301,asset_current,l10n_ph_chart_template,False
l10n_ph_110302,Stock Interim (Received),110302,asset_current,l10n_ph_chart_template,True
l10n_ph_110303,Stock Interim (Delivered),110303,asset_current,l10n_ph_chart_template,True
l10n_ph_110304,Stock Interim (Production),110304,asset_current,l10n_ph_chart_template,True
l10n_ph_110400,Other Non-current Asset,110400,asset_non_current,l10n_ph_chart_template,False
l10n_ph_110500,Building and Leasehold Improvements,110500,asset_fixed,l10n_ph_chart_template,False
l10n_ph_110501,"Furniture, Fixtures and Equipment",110501,asset_fixed,l10n_ph_chart_template,False
l10n_ph_110502,Vehicles,110502,asset_fixed,l10n_ph_chart_template,False
l10n_ph_110503,Computers,110503,asset_fixed,l10n_ph_chart_template,False
l10n_ph_110504,Accum Depreciation-Bldg & Leasehold Imp,110504,asset_fixed,l10n_ph_chart_template,False
l10n_ph_110505,"Accum Depreciation-Furniture, Fixtures and Equipment",110505,asset_fixed,l10n_ph_chart_template,False
l10n_ph_110506,Accum Depreciation-Vehicles,110506,asset_fixed,l10n_ph_chart_template,False
l10n_ph_110507,Accum Depreciation-Computers,110507,asset_fixed,l10n_ph_chart_template,False
l10n_ph_200000,Accounts Payable,200000,liability_payable,l10n_ph_chart_template,True
l10n_ph_200200,Dividends Payable - Common Shares,200200,liability_current,l10n_ph_chart_template,False
l10n_ph_200201,Unearned Revenue,200201,liability_current,l10n_ph_chart_template,False
l10n_ph_200202,Customer Deposits Received,200202,liability_current,l10n_ph_chart_template,False
l10n_ph_200300,Output VAT 12% - Actual,200300,liability_current,l10n_ph_chart_template,False
l10n_ph_200301,Deferred Output VAT 12% - Actual,200301,liability_current,l10n_ph_chart_template,False
l10n_ph_200302,Tax Payable-Withholding Tax Compensation,200302,liability_current,l10n_ph_chart_template,False
l10n_ph_200303,Tax Pay-Withholding Tax Source,200303,liability_current,l10n_ph_chart_template,False
l10n_ph_200304,Deferred Tax Pay-Withholding Tax Source,200304,liability_current,l10n_ph_chart_template,False
l10n_ph_200305,Taxes Payable-Final Withholding Tax,200305,liability_current,l10n_ph_chart_template,False
l10n_ph_200306,Tax Payable-Withholding Tax at Source (Accrual),200306,liability_current,l10n_ph_chart_template,False
l10n_ph_200307,Income Tax Payable,200307,liability_current,l10n_ph_chart_template,False
l10n_ph_200308,Accrued Fringe Benefit Tax,200308,liability_current,l10n_ph_chart_template,False
l10n_ph_200400,Other Non-current Liability,200400,liability_non_current,l10n_ph_chart_template,False
l10n_ph_300000,Capital Stock,300000,equity,l10n_ph_chart_template,False
l10n_ph_300001,Additional Paid in Capital,300001,equity,l10n_ph_chart_template,False
l10n_ph_300002,Retained Earnings,300002,equity,l10n_ph_chart_template,False
l10n_ph_430400,Sales/Revenues,430400,income,l10n_ph_chart_template,False
l10n_ph_510000,Cost of Goods Sold,510000,expense_direct_cost,l10n_ph_chart_template,False
l10n_ph_511100,COGS - Purchases,511100,expense_direct_cost,l10n_ph_chart_template,False
l10n_ph_511600,COGS - Freight Costs,511600,expense_direct_cost,l10n_ph_chart_template,False
l10n_ph_511700,COGS - Direct Labor,511700,expense_direct_cost,l10n_ph_chart_template,False
l10n_ph_511800,COGS - Factory/Processing Overhead,511800,expense_direct_cost,l10n_ph_chart_template,False
l10n_ph_620000,Admin Expense,620000,expense,l10n_ph_chart_template,False
l10n_ph_620001,Marketing Expense,620001,expense,l10n_ph_chart_template,False
l10n_ph_620100,Lease-Corporate Offices,620100,expense,l10n_ph_chart_template,False
l10n_ph_621000,Insurance Expenses,621000,expense,l10n_ph_chart_template,False
l10n_ph_622000,Repairs & Maintenance,622000,expense,l10n_ph_chart_template,False
l10n_ph_623000,Salaries & Wages,623000,expense,l10n_ph_chart_template,False
l10n_ph_623001,Overtime Pay,623001,expense,l10n_ph_chart_template,False
l10n_ph_623002,SSS Contribution,623002,expense,l10n_ph_chart_template,False
l10n_ph_623003,HDMF Contribution,623003,expense,l10n_ph_chart_template,False
l10n_ph_623004,PHIC Contribution,623004,expense,l10n_ph_chart_template,False
l10n_ph_623005,Meal Allowance,623005,expense,l10n_ph_chart_template,False
l10n_ph_623006,Other Allowance,623006,expense,l10n_ph_chart_template,False
l10n_ph_623007,13th Month Pay,623007,expense,l10n_ph_chart_template,False
l10n_ph_623008,Retirement Expenses,623008,expense,l10n_ph_chart_template,False
l10n_ph_623009,Training and Webinar,623009,expense,l10n_ph_chart_template,False
l10n_ph_623014,Sick Leave Expenses,623014,expense,l10n_ph_chart_template,False
l10n_ph_623015,Vacation Leave Expenses,623015,expense,l10n_ph_chart_template,False
l10n_ph_624000,Taxes and Licenses,624000,expense,l10n_ph_chart_template,False
l10n_ph_625000,Utilities Expenses,625000,expense,l10n_ph_chart_template,False
l10n_ph_626000,Security Services,626000,expense,l10n_ph_chart_template,False
l10n_ph_626001,Professional Fees,626001,expense,l10n_ph_chart_template,False
l10n_ph_627000,Travel and Transportation,627000,expense,l10n_ph_chart_template,False
l10n_ph_628000,Advertising and Promotion,628000,expense,l10n_ph_chart_template,False
l10n_ph_629000,Depreciation Expense,629000,expense_depreciation,l10n_ph_chart_template,False
l10n_ph_632000,Commission Expense,632000,expense,l10n_ph_chart_template,False
l10n_ph_633000,Supplies Expenses,633000,expense,l10n_ph_chart_template,False
l10n_ph_634000,Miscellaneous Expenses,634000,expense,l10n_ph_chart_template,False
l10n_ph_634001,Bank Fees,634001,expense,l10n_ph_chart_template,False
l10n_ph_635000,Prov DA-Billed Accts,635000,expense,l10n_ph_chart_template,False
l10n_ph_635001,Prov-Probable Expenses,635001,expense,l10n_ph_chart_template,False
l10n_ph_635002,Prov for Inc Tax - Creditable,635002,expense,l10n_ph_chart_template,False
l10n_ph_635003,Provision for Tax - Final,635003,expense,l10n_ph_chart_template,False
l10n_ph_710100,Forex Gain,710100,income_other,l10n_ph_chart_template,False
l10n_ph_710101,Forex Loss,710101,expense,l10n_ph_chart_template,False
l10n_ph_710102,Cash Difference Gain,710102,income_other,l10n_ph_chart_template,False
l10n_ph_710103,Cash Difference Loss,710103,expense,l10n_ph_chart_template,False
l10n_ph_710200,Gain/Loss- Sale FA,710200,expense,l10n_ph_chart_template,False
l10n_ph_710201,Loss on FA Disposal,710201,expense,l10n_ph_chart_template,False
l10n_ph_710300,Interest Inc-Savings,710300,income_other,l10n_ph_chart_template,False
l10n_ph_710301,Interest Income-Savings Local,710301,income_other,l10n_ph_chart_template,False
l10n_ph_710400,Other Income,710400,income_other,l10n_ph_chart_template,False
l10n_ph_710500,Interest Expense-Suppliers' Credit Local,710500,income_other,l10n_ph_chart_template,False
l10n_ph_999999,Undistributed Profits/Losses,999999,equity_unaffected,l10n_ph_chart_template,False

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo noupdate="1">
    <function model="account.chart.template" name="try_loading">
        <value eval="[ref('l10n_ph.l10n_ph_chart_template')]" />
    </function>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    <record id="l10n_ph_chart_template" model="account.chart.template">
        <field name="name">Philippines Account Template</field>
        <field name="bank_account_code_prefix">1000</field>
        <field name="cash_account_code_prefix">1001</field>
        <field name="transfer_account_code_prefix">1002</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.PHP" />
        <field name="spoken_languages" eval="'tl_PH'"/>
        <field name="use_anglo_saxon" eval="True" />
    </record>
</odoo>

```

## File: data\account_chart_template_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    <record id="l10n_ph_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="l10n_ph_110000" />
        <field name="property_account_payable_id" ref="l10n_ph_200000" />
        <field name="property_account_income_categ_id" ref="l10n_ph_430400" />
        <field name="property_account_expense_categ_id" ref="l10n_ph_620000" />
        <field name="property_stock_valuation_account_id" ref="l10n_ph_110300" />
        <field name="property_stock_account_input_categ_id" ref="l10n_ph_110302" />
        <field name="property_stock_account_output_categ_id" ref="l10n_ph_110303" />
        <field name="income_currency_exchange_account_id" ref="l10n_ph_710100" />
        <field name="expense_currency_exchange_account_id" ref="l10n_ph_710101" />
        <field name="default_pos_receivable_account_id" ref="l10n_ph_110003" />
        <field name="account_journal_suspense_account_id" ref="l10n_ph_100000" />
        <field name="default_cash_difference_income_account_id" ref="l10n_ph_710102" />
        <field name="default_cash_difference_expense_account_id" ref="l10n_ph_710103" />
    </record>
</odoo>

```

## File: data\account_fiscal_position_tax_template.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    <!-- VAT Registered -->
    <record id="l10n_ph_fiscal_position_tax_sale_vat_registered_vat_exempt" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="l10n_ph_tax_sale_vat_exempt" />
        <field name="tax_dest_id" ref="l10n_ph_tax_sale_vat_12" />
        <field name="position_id" ref="l10n_ph_fiscal_position_vat_registered" />
    </record>
    <record id="l10n_ph_fiscal_position_tax_purchase_vat_registered_vat_exempt" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="l10n_ph_tax_purchase_vat_exempt" />
        <field name="tax_dest_id" ref="l10n_ph_tax_purchase_vat_12" />
        <field name="position_id" ref="l10n_ph_fiscal_position_vat_registered" />
    </record>
    <record id="l10n_ph_fiscal_position_tax_sale_vat_exempt_vat_registered" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="l10n_ph_tax_sale_vat_12" />
        <field name="tax_dest_id" ref="l10n_ph_tax_sale_vat_exempt" />
        <field name="position_id" ref="l10n_ph_fiscal_position_vat_exempt" />
    </record>
    <record id="l10n_ph_fiscal_position_tax_purchase_vat_exempt_vat_registered" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="l10n_ph_tax_purchase_vat_12" />
        <field name="tax_dest_id" ref="l10n_ph_tax_purchase_vat_exempt" />
        <field name="position_id" ref="l10n_ph_fiscal_position_vat_exempt" />
    </record>
</odoo>

```

## File: data\account_fiscal_position_template.xml

```xml
<?xml version='1.0' encoding='utf-8' ?>
<odoo>
    <record id="l10n_ph_fiscal_position_vat_registered" model="account.fiscal.position.template">
        <field name="sequence">1</field>
        <field name="name">VAT Registered</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="auto_apply" eval="True" />
        <field name="vat_required" eval="True" />
        <field name="country_id" ref="base.ph" />
    </record>
    <record id="l10n_ph_fiscal_position_vat_exempt" model="account.fiscal.position.template">
        <field name="sequence">2</field>
        <field name="name">VAT Exempt</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="auto_apply" eval="True" />
        <field name="country_id" ref="base.ph" />
    </record>
</odoo>

```

## File: data\account_tax_group.xml

```xml
<?xml version='1.0' encoding='utf-8' ?>
<odoo noupdate="1">
    <record id="l10n_ph_tax_group_vat_12" model="account.tax.group">
        <field name="name">VAT 12%</field>
    </record>
    <record id="l10n_ph_tax_group_vat_exempt" model="account.tax.group">
        <field name="name">VAT Exempt</field>
    </record>
    <record id="l10n_ph_tax_group_wht_p5" model="account.tax.group">
        <field name="name">WHT 0.5%</field>
    </record>
    <record id="l10n_ph_tax_group_wht_1" model="account.tax.group">
        <field name="name">WHT 1%</field>
    </record>
    <record id="l10n_ph_tax_group_wht_2" model="account.tax.group">
        <field name="name">WHT 2%</field>
    </record>
    <record id="l10n_ph_tax_group_wht_5" model="account.tax.group">
        <field name="name">WHT 5%</field>
    </record>
    <record id="l10n_ph_tax_group_wht_10" model="account.tax.group">
        <field name="name">WHT 10%</field>
    </record>
    <record id="l10n_ph_tax_group_wht_15" model="account.tax.group">
        <field name="name">WHT 15%</field>
    </record>
</odoo>

```

## File: data\account_tax_template.xml

```xml
<?xml version='1.0' encoding='utf-8' ?>
<odoo>
    <record id="l10n_ph_tax_sale_vat_12" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">12% VAT</field>
        <field name="description">12% VAT</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">12</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_vat_12" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200300'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200300'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_sale_vat_exempt" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">VAT Exempt</field>
        <field name="description">VAT Exempt</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_vat_exempt" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200300'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200300'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_vat_12" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">12% VAT</field>
        <field name="description">12% VAT</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">12</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_vat_12" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_110201'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_110201'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_vat_exempt" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">VAT Exempt</field>
        <field name="description">VAT Exempt</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_vat_exempt" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_110201'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_110201'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wi010" model="account.tax.template">
        <field name="sequence">30</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">5% WI010 - Prof Fees</field>
        <field name="description">Prof Fees</field>
        <field name="l10n_ph_atc">WI010</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-5</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_5" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wi011" model="account.tax.template">
        <field name="sequence">30</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">10% WI011 - Prof Fees</field>
        <field name="description">Prof Fees</field>
        <field name="l10n_ph_atc">WI011</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-10</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_10" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wi100" model="account.tax.template">
        <field name="sequence">40</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">5% WI100 - Gross rental of property</field>
        <field name="description">Gross rental of property</field>
        <field name="l10n_ph_atc">WI100</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-5</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_5" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wi120" model="account.tax.template">
        <field name="sequence">50</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">2% WI120 - Contractors</field>
        <field name="description">Contractors</field>
        <field name="l10n_ph_atc">WI120</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-2</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_2" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wi139" model="account.tax.template">
        <field name="sequence">60</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">5% WI139 - Commission of service fees</field>
        <field name="description">Commission of service fees</field>
        <field name="l10n_ph_atc">WI139</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-5</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_5" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wi140" model="account.tax.template">
        <field name="sequence">60</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">10% WI140 - Commission of service fees</field>
        <field name="description">Commission of service fees</field>
        <field name="l10n_ph_atc">WI140</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-10</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_10" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wi158_p5" model="account.tax.template">
        <field name="sequence">70</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">0.5% WI158 - Credit card companies</field>
        <field name="description">Credit card companies</field>
        <field name="l10n_ph_atc">WI158</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-0.5</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_p5" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wi640" model="account.tax.template">
        <field name="sequence">80</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">1% WI640 - Supplier of goods</field>
        <field name="description">Supplier of goods</field>
        <field name="l10n_ph_atc">WI640</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-1</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_1" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wi157" model="account.tax.template">
        <field name="sequence">80</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">2% WI157 - Supplier of services</field>
        <field name="description">Supplier of services</field>
        <field name="l10n_ph_atc">WI157</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-2</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_2" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wi158_1" model="account.tax.template">
        <field name="sequence">90</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">1% WI158 - Supplier of goods by top w/holding agents</field>
        <field name="description">Supplier of goods by top w/holding agents</field>
        <field name="l10n_ph_atc">WI158</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-1</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_1" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wi160" model="account.tax.template">
        <field name="sequence">90</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">2% WI160 - Supplier of goods by top w/holding agents</field>
        <field name="description">Supplier of goods by top w/holding agents</field>
        <field name="l10n_ph_atc">WI160</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-2</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_2" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wi515" model="account.tax.template">
        <field name="sequence">100</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">5% WI515 - Commission, rebates, discounts</field>
        <field name="description">Commission, rebates, discounts</field>
        <field name="l10n_ph_atc">WI515</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-5</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_5" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wi516" model="account.tax.template">
        <field name="sequence">100</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">10% WI516 - Commission, rebates, discounts</field>
        <field name="description">Commission, rebates, discounts</field>
        <field name="l10n_ph_atc">WI516</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-10</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_10" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wc010" model="account.tax.template">
        <field name="sequence">110</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">10% WC010 - Prof Fees</field>
        <field name="description">Prof Fees</field>
        <field name="l10n_ph_atc">WC010</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-10</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_10" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wc011" model="account.tax.template">
        <field name="sequence">110</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">15% WC011 - Prof Fees</field>
        <field name="description">Prof Fees</field>
        <field name="l10n_ph_atc">WC011</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-15</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_15" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wc100" model="account.tax.template">
        <field name="sequence">120</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">5% WC100 - Gross rental of property</field>
        <field name="description">Gross rental of property</field>
        <field name="l10n_ph_atc">WC100</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-5</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_5" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wc120" model="account.tax.template">
        <field name="sequence">130</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">2% WC120 - Contractors</field>
        <field name="description">Contractors</field>
        <field name="l10n_ph_atc">WC120</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-2</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_2" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wc139" model="account.tax.template">
        <field name="sequence">140</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">10% WC139 - Commission of service fees</field>
        <field name="description">Commission of service fees</field>
        <field name="l10n_ph_atc">WC139</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-10</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_10" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wc140" model="account.tax.template">
        <field name="sequence">140</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">15% WC140 - Commission of service fees</field>
        <field name="description">Commission of service fees</field>
        <field name="l10n_ph_atc">WC140</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-15</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_15" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wc158_p5" model="account.tax.template">
        <field name="sequence">150</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">0.5% WC158 - Credit card companies</field>
        <field name="description">Credit card companies</field>
        <field name="l10n_ph_atc">WC158</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-0.5</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_p5" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wc640" model="account.tax.template">
        <field name="sequence">160</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">1% WC640 - Supplier of goods</field>
        <field name="description">Supplier of goods</field>
        <field name="l10n_ph_atc">WC640</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-1</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_1" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wc157" model="account.tax.template">
        <field name="sequence">160</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">2% WC157 - Supplier of services</field>
        <field name="description">Supplier of services</field>
        <field name="l10n_ph_atc">WC157</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-2</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_2" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wc158_1" model="account.tax.template">
        <field name="sequence">170</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">1% WC158 - Supplier of goods by top w/holding agents</field>
        <field name="description">Supplier of goods by top w/holding agents</field>
        <field name="l10n_ph_atc">WC158</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-1</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_1" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wc160" model="account.tax.template">
        <field name="sequence">170</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">2% WC160 - Supplier of goods by top w/holding agents</field>
        <field name="description">Supplier of goods by top w/holding agents</field>
        <field name="l10n_ph_atc">WC160</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-2</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_2" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wc515" model="account.tax.template">
        <field name="sequence">180</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">5% WC515 - Commission, rebates, discounts</field>
        <field name="description">Commission, rebates, discounts</field>
        <field name="l10n_ph_atc">WC515</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-5</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_5" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
    <record id="l10n_ph_tax_purchase_wc516" model="account.tax.template">
        <field name="sequence">180</field>
        <field name="chart_template_id" ref="l10n_ph_chart_template" />
        <field name="name">10% WC516 - Commission, rebates, discounts</field>
        <field name="description">Commission, rebates, discounts</field>
        <field name="l10n_ph_atc">WC516</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">-10</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_ph_tax_group_wht_10" />
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ph_200303'),
            }),
        ]"
        />
    </record>
</odoo>

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models
from odoo.exceptions import UserError


class AccountMove(models.Model):
    _inherit = "account.move"

    def action_open_l10n_ph_2307_wizard(self):
        vendor_bills = self.filtered_domain([('move_type', '=', 'in_invoice')])
        if vendor_bills:
            wizard_action = self.env["ir.actions.act_window"]._for_xml_id("l10n_ph.view_l10n_ph_2307_wizard_act_window")
            wizard_action.update({
                'context': {'default_moves_to_export': vendor_bills.ids}
            })
            return wizard_action
        else:
            raise UserError(_('Only Vendor Bills are available.'))

```

## File: models\account_payment.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models
from odoo.exceptions import UserError


class AccountPayment(models.Model):
    _inherit = "account.payment"

    def action_open_l10n_ph_2307_wizard(self):
        self.ensure_one()
        if self.payment_type == 'outbound':
            wizard_action = self.env["ir.actions.act_window"]._for_xml_id("l10n_ph.view_l10n_ph_2307_wizard_act_window")
            wizard_action.update({
                'context': {'default_moves_to_export': self.reconciled_bill_ids.ids}
            })
            return wizard_action
        else:
            raise UserError(_('Only Outbound Payment is available.'))

```

## File: models\account_tax.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class AccountTax(models.Model):
    _inherit = "account.tax"

    l10n_ph_atc = fields.Char("Philippines ATC")

```

## File: models\account_tax_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class AccountTaxTemplate(models.Model):
    _inherit = "account.tax.template"

    l10n_ph_atc = fields.Char("Philippines ATC")

    def _get_tax_vals(self, company, tax_template_to_tax):
        val = super()._get_tax_vals(company, tax_template_to_tax)
        val.update({"l10n_ph_atc": self.l10n_ph_atc})
        return val

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, api, models


class ResPartner(models.Model):
    _inherit = "res.partner"

    branch_code = fields.Char("Branch Code", default='000', compute='_compute_branch_code', store=True)
    first_name = fields.Char("First Name")
    middle_name = fields.Char("Middle Name")
    last_name = fields.Char("Last Name")

    @api.model
    def _commercial_fields(self):
        return super()._commercial_fields() + ['branch_code']

    @api.depends('vat', 'country_id')
    def _compute_branch_code(self):
        for partner in self:
            branch_code = '000'
            if partner.country_id.code == 'PH' and partner.vat:
                match = partner.__check_vat_ph_re.match(partner.vat)
                branch_code = match and match.group(1) and match.group(1)[1:] or branch_code
            partner.branch_code = branch_code

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_partner
from . import account_move
from . import account_payment
from . import account_tax
from . import account_tax_template

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
generate_2307_wizard,access_generate_2307_wizard,model_l10n_ph_2307_wizard,account.group_account_invoice,1,1,1,1

```

## File: views\account_move_views.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <record id="action_account_move_bir_2307" model="ir.actions.server">
        <field name="name">Download BIR 2307 XLS</field>
        <field name="groups_id" eval="[(4, ref('account.group_account_invoice'))]"/>
        <field name="model_id" ref="account.model_account_move"/>
        <field name="binding_model_id" ref="account.model_account_move"/>
        <field name="binding_view_types">list,form</field>
        <field name="state">code</field>
        <field name="code">action = records.action_open_l10n_ph_2307_wizard()</field>
    </record>
</odoo>

```

## File: views\account_payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="action_account_payment_bir_2307" model="ir.actions.server">
        <field name="name">Download BIR 2307 XLS</field>
        <field name="groups_id" eval="[(4, ref('account.group_account_invoice'))]"/>
        <field name="model_id" ref="account.model_account_payment"/>
        <field name="binding_model_id" ref="account.model_account_payment"/>
        <field name="binding_view_types">form</field>
        <field name="state">code</field>
        <field name="code">action = record.action_open_l10n_ph_2307_wizard()</field>
    </record>
</odoo>

```

## File: views\account_tax_views.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    <record id="view_tax_form" model="ir.ui.view">
        <field name="name">account.tax.view.form.inherit.l10n_ph</field>
        <field name="model">account.tax</field>
        <field name="inherit_id" ref="account.view_tax_form" />
        <field name="arch" type="xml">
            <notebook position="inside">
                <page name="l10n_ph" string="Philippines">
                    <group>
                        <group>
                            <field name="l10n_ph_atc" />
                        </group>
                    </group>
                </page>
            </notebook>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <record id="view_partner_form" model="ir.ui.view">
        <field name="name">res.partner.form.inherit.l10n_ph_bir</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="after">
                <field name="branch_code"/>
                <field name="first_name" attrs="{'invisible': [('is_company','=', True)]}"/>
                <field name="middle_name" attrs="{'invisible': [('is_company','=', True)]}"/>
                <field name="last_name" attrs="{'invisible': [('is_company','=', True)]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\generate_2307_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import io
import re
import xlwt

from odoo import fields, models
from odoo.tools.misc import format_date


COLUMN_HEADER_MAP = {
    "Reporting_Month": "invoice_date",
    "Vendor_TIN": "vat",
    "branchCode": "branch_code",
    "companyName": "company_name",
    "surName": "last_name",
    "firstName": "first_name",
    "middleName": "middle_name",
    "address": "address",
    "nature": "product_name",
    "ATC": "atc",
    "income_payment": "price_subtotal",
    "ewt_rate": "amount",
    "tax_amount": "tax_amount",
}

class Generate2307Wizard(models.TransientModel):
    _name = "l10n_ph_2307.wizard"
    _description = "Exports 2307 data to a XLS file."

    moves_to_export = fields.Many2many("account.move", string="Joural To Include")
    generate_xls_file = fields.Binary(
        "Generated file",
        help="Technical field used to temporarily hold the generated XLS file before its downloaded."
    )

    def _write_single_row(self, worksheet, worksheet_row, values):
        for index, field in enumerate(COLUMN_HEADER_MAP.values()):
            worksheet.write(worksheet_row, index, label=values[field])

    def _write_rows(self, worksheet, moves):
        worksheet_row = 0
        for move in moves:
            worksheet_row += 1
            partner = move.partner_id
            partner_address_info = [partner.street, partner.street2, partner.city, partner.state_id.name, partner.country_id.name]
            values = {
                'invoice_date': format_date(self.env, move.invoice_date, date_format="MM/dd/yyyy"),
                'vat': re.sub(r'\-', '', partner.vat)[:9] if partner.vat else '',
                'branch_code': partner.branch_code or '000',
                'company_name': partner.commercial_partner_id.name,
                'first_name': partner.first_name or '',
                'middle_name': partner.middle_name or '',
                'last_name': partner.last_name or '',
                'address': ', '.join([val for val in partner_address_info if val])
            }
            aggregated_taxes = move._prepare_invoice_aggregated_taxes()
            for invoice_line, tax_details_for_line in aggregated_taxes['tax_details_per_record'].items():
                for tax_detail in tax_details_for_line['tax_details'].values():
                    tax = tax_detail['tax']
                    if not tax.l10n_ph_atc:
                        continue

                    product_name = invoice_line.product_id.name or invoice_line.name
                    values['product_name'] = re.sub(r'[\(\)]', '', product_name) if product_name else ""
                    values['atc'] = tax.l10n_ph_atc
                    values['price_subtotal'] = tax_detail['base_amount']
                    values['amount'] = tax.amount
                    values['tax_amount'] = tax_detail['tax_amount']
                    self._write_single_row(worksheet, worksheet_row, values)
                    worksheet_row += 1

    def action_generate(self):
        """ Generate a xls format file for importing to
        https://bir-excel-uploader.com/excel-file-to-bir-dat-format/#bir-form-2307-settings.
        This website will then generate a BIR 2307 format excel file for uploading to the
        PH government.
        """
        self.ensure_one()

        file_data = io.BytesIO()
        workbook = xlwt.Workbook(encoding='utf-8')
        worksheet = workbook.add_sheet('Form2307')

        for index, col_header in enumerate(COLUMN_HEADER_MAP.keys()):
            worksheet.write(0, index, label=col_header)

        self._write_rows(worksheet, self.moves_to_export)

        workbook.save(file_data)
        file_data.seek(0)
        self.generate_xls_file = base64.b64encode(file_data.read())

        return {
            "type": "ir.actions.act_url",
            "target": "self",
            "url": "/web/content?model=l10n_ph_2307.wizard&download=true&field=generate_xls_file&filename=Form_2307.xls&id={}".format(self.id),
        }

```

## File: wizard\generate_2307_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_ph_2307_wizard_view_form" model="ir.ui.view">
        <field name="name">l10n_ph_2307.wizard.form</field>
        <field name="model">l10n_ph_2307.wizard</field>
        <field name="arch" type="xml">
            <form string="Generate BIR 2307 Report">
                This will export a XLS file for BIR 2307.
                <group>
                    <field name="moves_to_export" nolabel="1" readonly="1" colspan="2">
                        <tree>
                            <field name="name" optional="show"/>
                            <field name="invoice_partner_display_name" string="Vendor"/>
                            <field name="invoice_date" string="Bill Date"/>
                            <field name="invoice_date_due"/>
                            <field name="currency_id" invisible="1"/>
                            <field name="amount_tax_signed" string="Tax" sum="Total" optional="hide" modifiers="{'readonly':true}" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                            <field name="amount_total_signed" string="Total" sum="Total" decoration-bf="1" optional="show" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                            <field name="state" widget="badge" decoration-success="state == 'posted'" decoration-info="state == 'draft'" optional="show" on_change="1" modifiers="{'readonly':true, 'required':true}"/>
                        </tree>
                    </field>
                </group>
                <footer>
                    <button string="Generate" type="object" name="action_generate" class="btn btn-primary" data-hotkey="q"/>
                    <button string="Cancel" special="cancel" data-hotkey="z" class="btn btn-secondary"/>
                </footer>
            </form>
        </field>
    </record>
    <record id="view_l10n_ph_2307_wizard_act_window" model="ir.actions.act_window">
        <field name="name">BIR 2307 Report</field>
        <field name="res_model">l10n_ph_2307.wizard</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import generate_2307_wizard

```

