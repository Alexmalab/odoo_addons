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
from odoo import api, SUPERUSER_ID

def load_translations(cr, registry):
    """Load template translations."""
    env = api.Environment(cr, SUPERUSER_ID, {})
    env.ref('l10n_lv.chart_template_latvia').process_coa_translations()

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
        'l10n_multilang',
        'account',
        'base_vat',
    ],
    'data': [
        'data/account_chart_template_data.xml',
        'data/account.account.template.csv',
        'data/account.group.template.csv',
        'data/account_chart_template_setup_data.xml',
        'data/vat_tax_report.xml',
        'data/account_tax_group_data.xml',
        'data/account_tax_template_data.xml',
        'data/account_fiscal_position_template_data.xml',
        'data/account_chart_template_load.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'post_init_hook': 'load_translations',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","account_type","chart_template_id/id","tag_ids/id","reconcile"
"a1110","Research and company development costs","1110","asset_non_current","l10n_lv.chart_template_latvia",,"False"
"a1120","Concessions, patents, licenses, trademarks and similar rights","1120","asset_non_current","l10n_lv.chart_template_latvia",,"False"
"a1130","Other intangible assets","1130","asset_non_current","l10n_lv.chart_template_latvia",,"False"
"a1140","Goodwill","1140","asset_non_current","l10n_lv.chart_template_latvia",,"False"
"a1180","Advance payments for intangible assets","1150","asset_prepayments","l10n_lv.chart_template_latvia",,"False"
"a1191","Accumulated depreciation on portion of development costs","1191","asset_non_current","l10n_lv.chart_template_latvia",,"False"
"a1192","Write-down portion of concessions, patents, licenses, trademarks and similar rights","1192","asset_non_current","l10n_lv.chart_template_latvia",,"False"
"a1193","Accumulated depreciation on part of the value of other intangible investments","1193","asset_non_current","l10n_lv.chart_template_latvia",,"False"
"a1210","Land, buildings, perennial plantings","1210","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1220","Technological equipment and machinery","1220","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1230","Other fixed assets and inventory","1230","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1240","Costs of fixed asset creation and unfinished object construction","1240","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1250","Long-term investments in leased fixed assets","1250","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1260","Long-term investments in fixed assets of the public partner","1260","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1280","Advance payments for fixed assets","1280","asset_prepayments","l10n_lv.chart_template_latvia",,"False"
"a1291","Accumulated depreciation for Land, buildings, perennial plantings","1291","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1292","Accumulated depreciation on plant and machinery","1292","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1293","Accumulated depreciation for Other fixed assets and inventory","1293","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1294","Accumulated depreciation on long-term investments in fixed assets of the public partner","1294","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1310","Plots of land","1310","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1320","Buildings, structures or parts thereof","1320","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1330","Costs of creating investment properties","1330","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1390","Accumulated depreciation on investment properties","1390","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1410","Animals","1410","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1420","Plants","1420","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1430","Working or production animals","1430","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1440","Perennial plantings","1440","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1490","Accumulated depreciation of biologival assets","1490","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a1510","Participation in the capital of related companies","1510","asset_current","l10n_lv.chart_template_latvia",,"False"
"a1520","Loans to related companies","1520","asset_current","l10n_lv.chart_template_latvia",,"False"
"a1530","Participation in the capital of associated companies","1530","asset_current","l10n_lv.chart_template_latvia",,"False"
"a1540","Loans to associated companies","1540","asset_current","l10n_lv.chart_template_latvia",,"False"
"a1550","Other securities and investments","1550","asset_current","l10n_lv.chart_template_latvia",,"False"
"a1560","Other loans and other long-term debtors","1560","asset_current","l10n_lv.chart_template_latvia",,"False"
"a1570","Own shares","1570","asset_current","l10n_lv.chart_template_latvia",,"False"
"a1580","Loans to shareholders or members and management","1580","asset_current","l10n_lv.chart_template_latvia",,"False"
"a1590","Deferred tax assets","1590","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2110","Raw materials, basic materials and auxiliary materials","2110","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2120","Unfinished production","2120","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2130","Finished products and goods for sale","2130","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2140","Uncompleted orders","2140","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2150","Low-value inventory","2150","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a2159","Write-down off low-value inventory","2159","asset_fixed","l10n_lv.chart_template_latvia",,"False"
"a2190","Advance payments for goods","2190","Prepayments","l10n_lv.chart_template_latvia",,"False"
"a2210","Animals","2210","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2220","Young and small animals","2220","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2230","Annual plantings","2230","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2310","Accounts receivable – trade","2310","asset_receivable","l10n_lv.chart_template_latvia",,"True"
"a2318","Settlement of doubtful debtors","2318","asset_receivable","l10n_lv.chart_template_latvia",,"True"
"a2320","Bills of exchange received for deliveries and customers","2320","asset_receivable","l10n_lv.chart_template_latvia",,"True"
"a2330","Debts of subsidiaries","2330","asset_receivable","l10n_lv.chart_template_latvia",,"True"
"a2340","Debts of related companies","2340","asset_receivable","l10n_lv.chart_template_latvia",,"True"
"a2350","Settlements with other debtors","2350","asset_receivable","l10n_lv.chart_template_latvia",,"True"
"a2360","Settlement of unpaid amounts of the company's capital","2360","asset_receivable","l10n_lv.chart_template_latvia",,"True"
"a2370","Short-term loans for company shareholder and management","2370","asset_receivable","l10n_lv.chart_template_latvia",,"True"
"a2380","Settlement of claims on staff","2380","asset_receivable","l10n_lv.chart_template_latvia",,"True"
"a2388","Accrued revenue","2388","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2390","Tax overpayments, taxes paid in advance","2390","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2391","Tax and fee cancellation (tax account)","2391","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2392","Frozen recoverable VAT","2392","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2399","VAT charged but not deducted","2399","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2410","Deferred expenses","2410","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2420","Share issue valuation","2420","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2510","Shares of the subsidiary company","2510","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2520","Shares of related companies","2520","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2530","Shares of own company","2530","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2540","Other securities and participations in equity","2540","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2550","Derivative financial instruments","2550","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2610","Cash","2610","asset_cash","l10n_lv.chart_template_latvia",,"False"
"a2613","ECR (retail) cash register","2613","asset_cash","l10n_lv.chart_template_latvia",,"False"
"a2620","Bank account","2620","asset_cash","l10n_lv.chart_template_latvia",,"False"
"a26291","Bank Suspense Account","26291","asset_current","l10n_lv.chart_template_latvia",,"False"
"a26292","Outstanding Receipts","26292","asset_current","l10n_lv.chart_template_latvia",,"False"
"a26293","Outstanding Payments","26293","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2640","Credit notes, checks and special forms of payment accounts","2640","asset_cash","l10n_lv.chart_template_latvia",,"False"
"a2650","Other bank accounts","2650","asset_cash","l10n_lv.chart_template_latvia",,"False"
"a2670","Other funds","2670","asset_cash","l10n_lv.chart_template_latvia",,"False"
"a2671","POS or payment system settlements","2671","asset_cash","l10n_lv.chart_template_latvia",,"False"
"a2696","Clearing of payments","2696","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2698","Currency exchange account","2698","asset_current","l10n_lv.chart_template_latvia",,"False"
"a2699","Money on the move","2699","asset_current","l10n_lv.chart_template_latvia",,"True"
"a3110","Capital of own shares","3110","equity","l10n_lv.chart_template_latvia",,"False"
"a3111","Unregistered share capital","3111","equity","l10n_lv.chart_template_latvia",,"False"
"a3120","Share issue premium","3120","equity","l10n_lv.chart_template_latvia",,"False"
"a3130","Revaluation reserve for long-term investments","3130","equity","l10n_lv.chart_template_latvia",,"False"
"a3140","Revaluation reserve for financial instruments","3140","equity","l10n_lv.chart_template_latvia",,"False"
"a3210","Funds withdrawn for private purposes","3210","liability_current","l10n_lv.chart_template_latvia",,"True"
"a3220","Private investment","3220","liability_current","l10n_lv.chart_template_latvia",,"True"
"a3310","Statutory reserves","3310","equity","l10n_lv.chart_template_latvia",,"False"
"a3320","Reserves for own shares or parts","3320","equity","l10n_lv.chart_template_latvia",,"False"
"a3330","Reserves specified in the company's articles of association","3330","equity","l10n_lv.chart_template_latvia",,"False"
"a3340","Reserves earmarked for development","3340","equity","l10n_lv.chart_template_latvia",,"False"
"a3350","Foreign currency conversion reserve","3350","equity","l10n_lv.chart_template_latvia",,"False"
"a3360","Other reserves","3360","equity","l10n_lv.chart_template_latvia",,"False"
"a3410","Retained earnings for the current year","3410","equity_unaffected","l10n_lv.chart_template_latvia",,"False"
"a3420","Retained earnings","3420","equity","l10n_lv.chart_template_latvia",,"False"
"a4110","Savings for pensions and similar liabilities","4110","liability_non_current","l10n_lv.chart_template_latvia",,"False"
"a4210","Provisions for estimated taxes","4210","liability_non_current","l10n_lv.chart_template_latvia",,"False"
"a4310","Other savings","4310","liability_non_current","l10n_lv.chart_template_latvia",,"False"
"a4319","Provisions for doubtful debtors","4319","liability_non_current","l10n_lv.chart_template_latvia",,"False"
"a5110","Loans for bonds","5110","liability_non_current","l10n_lv.chart_template_latvia",,"False"
"a5120","Loans convertible into shares","5120","liability_non_current","l10n_lv.chart_template_latvia",,"False"
"a5130","Loans with profit participation","5130","liability_non_current","l10n_lv.chart_template_latvia",,"False"
"a5140","Other loans","5140","liability_non_current","l10n_lv.chart_template_latvia",,"True"
"a5141","Loans from shareholders","5141","liability_non_current","l10n_lv.chart_template_latvia",,"True"
"a5150","Short-term loans from credit institutions","5150","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5160","Long-term loans from credit institutions","5160","liability_non_current","l10n_lv.chart_template_latvia",,"False"
"a5170","Short-term leasing liabilities","5170","liability_non_current","l10n_lv.chart_template_latvia",,"False"
"a5180","Long-term leasing liabilities","5180","liability_non_current","l10n_lv.chart_template_latvia",,"False"
"a5190","Loans without term","5190","liability_non_current","l10n_lv.chart_template_latvia",,"False"
"a5210","Advances received from customers","5210","liability_current","l10n_lv.chart_template_latvia",,"True"
"a5299","VAT calculated and deducted","5299","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5310","Accounts payable – trade","5310","liability_payable","l10n_lv.chart_template_latvia",,"True"
"a5319","Accrued liabilities to creditors","5319","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5410","Settlement of self-issued promissory notes","5410","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5510","Settlement of debts owed to subsidiaries","5510","liability_payable","l10n_lv.chart_template_latvia",,"True"
"a5520","Settlement of debts owed to related companies","5520","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5530","Settlement of debts with companies under participation agreements","5530","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5540","Settlement of debts with other companies","5540","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5550","Settlement of debts with staff","5550","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5610","Wage settlements","5610","liability_current","l10n_lv.chart_template_latvia",,"True"
"a5611","Settlement of wage advances","5611","liability_current","l10n_lv.chart_template_latvia",,"True"
"a5612","Holiday allowance paid","5612","liability_current","l10n_lv.chart_template_latvia",,"True"
"a5613","Expenses and benefits included in salary","5613","liability_current","l10n_lv.chart_template_latvia",,"True"
"a5619","Allowances payable","5619","liability_current","l10n_lv.chart_template_latvia",,"True"
"a5620","Settlement of deductions from wages","5620","liability_current","l10n_lv.chart_template_latvia",,"True"
"a5621","Deductions imposed by bailiffs - first-order deductions","5621","liability_current","l10n_lv.chart_template_latvia",,"True"
"a5630","Other settlements with employees","5630","liability_current","l10n_lv.chart_template_latvia",,"True"
"a5650","Settlement of payables to staff","5650","liability_current","l10n_lv.chart_template_latvia",,"True"
"a5710","Settlement of corporate income taxes","5710","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5711","Settlement of foreign corporate income taxes","5711","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5721","Settlement of value added taxes (VAT)","5721","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5722","Settlement of personal income tax","5722","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5723","Settlement of social security contributions (SSI)","5723","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5724","Settlements for business risk state fees","5724","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5725","Real estate tax settlement","5725","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5726","VAT settlement for OSS goods and services","5726","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5727","Natural resource tax (NRT) settlement","5727","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5728","Settlement of company car tax","5728","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5729","Settlement of import duties","5729","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5810","Settlement of unpaid dividends","5810","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5910","Deferred income","5910","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5920","Accrued liabilities","5920","liability_current","l10n_lv.chart_template_latvia",,"False"
"a5930","Derivative financial instruments","5930","liability_current","l10n_lv.chart_template_latvia",,"False"
"a6110","Sales from operating activities subject, VAT 21%","6110","income","l10n_lv.chart_template_latvia",,"False"
"a6120","Sales from operating activities subject, VAT reduced rate 12%","6120","income","l10n_lv.chart_template_latvia",,"False"
"a6130","Sales from operating activities subject, VAT reduced rate 5%","6130","income","l10n_lv.chart_template_latvia",,"False"
"a6140","Sales from operating activities,VAT 0%","6140","income","l10n_lv.chart_template_latvia",,"False"
"a6210","Non-taxable sales revenues","6210","income","l10n_lv.chart_template_latvia",,"False"
"a6220","Sales revenue subject to special taxes","6220","income","l10n_lv.chart_template_latvia",,"False"
"a6310","Commission and brokerage income","6310","income","l10n_lv.chart_template_latvia",,"False"
"a6320","Revenues from waste processing and sale","6320","income","l10n_lv.chart_template_latvia",,"False"
"a6330","Revenues from the sale of containers","6330","income","l10n_lv.chart_template_latvia",,"False"
"a6340","Non-taxable income","6340","income","l10n_lv.chart_template_latvia",,"False"
"a6410","Sconto discounts granted","6410","income","l10n_lv.chart_template_latvia",,"False"
"a6420","Bonuses granted","6420","income","l10n_lv.chart_template_latvia",,"False"
"a6430","Granted rebates and other trade discounts","6430","income","l10n_lv.chart_template_latvia",,"False"
"a6510","Income from the increase in the price of securities","6510","income_other","l10n_lv.chart_template_latvia",,"False"
"a6520","Income from the sale of fixed assets","6520","income_other","l10n_lv.chart_template_latvia",,"False"
"a6530","Income from the rental of plots of land","6530","income_other","l10n_lv.chart_template_latvia",,"False"
"a6540","Revenues from the sale of current assets","6540","income_other","l10n_lv.chart_template_latvia",,"False"
"a6550","Income from leasing of fixed assets","6550","income_other","l10n_lv.chart_template_latvia",,"False"
"a6560","Decrease in reserves and provisions transferred to revenue","6560","income_other","l10n_lv.chart_template_latvia",,"False"
"a6570","Tax reduction of previous years","6570","income_other","l10n_lv.chart_template_latvia",,"False"
"a6580","Additional investments and other income","6580","income_other","l10n_lv.chart_template_latvia",,"False"
"a6590","Works executed for own long-term investments","6590","income_other","l10n_lv.chart_template_latvia",,"False"
"a6610","Changes in stocks and value of finished products","6610","income_other","l10n_lv.chart_template_latvia",,"False"
"a6620","Changes in inventories and value of work-in-progress","6620","income_other","l10n_lv.chart_template_latvia",,"False"
"a6630","Changes in backorder balances and value","6630","income_other","l10n_lv.chart_template_latvia",,"False"
"a6640","Changes in the herd value of productive and working animals","6640","income_other","l10n_lv.chart_template_latvia",,"False"
"a6710","Revenues of other periods relating to the reporting period","6710","income_other","l10n_lv.chart_template_latvia",,"False"
"a6910","Income from apartment management","6910","income_other","l10n_lv.chart_template_latvia",,"False"
"a6920","Utility income","6920","income_other","l10n_lv.chart_template_latvia",,"False"
"a6930","Revenue from domestic services","6930","income_other","l10n_lv.chart_template_latvia",,"False"
"a6940","Catering revenue","6940","income_other","l10n_lv.chart_template_latvia",,"False"
"a6950","Income of educational institutions","6950","income_other","l10n_lv.chart_template_latvia",,"False"
"a6960","Medical service revenue","6960","income_other","l10n_lv.chart_template_latvia",,"False"
"a6970","Revenues of sports institutions and from culture events","6970","income_other","l10n_lv.chart_template_latvia",,"False"
"a6980","Other revenues of social infrastructure institutions and events","6980","income_other","l10n_lv.chart_template_latvia",,"False"
"a7110","Cost of raw materials and delivery expenses","7110","expense","l10n_lv.chart_template_latvia",,"False"
"a7120","Cost of goods sold and delivery expenses","7120","expense","l10n_lv.chart_template_latvia",,"False"
"a7130","Discounts received","7130","expense","l10n_lv.chart_template_latvia",,"False"
"a7140","Cost of packaging","7140","expense","l10n_lv.chart_template_latvia",,"False"
"a7150","Customs and import duties","7150","expense","l10n_lv.chart_template_latvia",,"False"
"a7160","Other external expenses","7160","expense","l10n_lv.chart_template_latvia",,"False"
"a7170","Cost of external work and services","7170","expense","l10n_lv.chart_template_latvia",,"False"
"a7180","Other material expenses (tax on the use of natural resources, stumpage money, etc.)","7180","expense","l10n_lv.chart_template_latvia",,"False"
"a7190","Changes in inventory and value of purchased materials and goods","7190","expense","l10n_lv.chart_template_latvia",,"False"
"a7210","Employee salaries","7210","expense","l10n_lv.chart_template_latvia",,"False"
"a7220","Salaries of management and administrative staff","7220","expense","l10n_lv.chart_template_latvia",,"False"
"a7230","Salaries of employees of social infrastructure institutions and events","7230","expense","l10n_lv.chart_template_latvia",,"False"
"a7240","Other personnel costs","7240","expense","l10n_lv.chart_template_latvia",,"False"
"a7310","Social tax","7310","expense","l10n_lv.chart_template_latvia",,"False"
"a7320","Other social costs","7320","expense","l10n_lv.chart_template_latvia",,"False"
"a7330","Social costs of social infrastructure facilities and activities","7330","expense","l10n_lv.chart_template_latvia",,"False"
"a7340","Business risk state fees","7340","expense","l10n_lv.chart_template_latvia",,"False"
"a7410","Write-down off the value of intangible assets","7410","expense","l10n_lv.chart_template_latvia",,"False"
"a7420","Depreciation of fixed assets","7420","expense","l10n_lv.chart_template_latvia",,"False"
"a7430","Regulatory stock losses","7430","expense","l10n_lv.chart_template_latvia",,"False"
"a7440","Accumulated depreciation on the value of current asset","7440","expense","l10n_lv.chart_template_latvia",,"False"
"a7450","Over-normal stock losses","7450","expense","l10n_lv.chart_template_latvia",,"False"
"a7460","Stock-outs","7460","expense","l10n_lv.chart_template_latvia",,"False"
"a7470","Asset impairment charge","7470","expense","l10n_lv.chart_template_latvia",,"False"
"a7510","Nature conservation expenses","7510","expense","l10n_lv.chart_template_latvia",,"False"
"a7520","Undepreciated value (residual value) of fixed assets excluded","7520","expense","l10n_lv.chart_template_latvia",,"False"
"a7530","Fees for plots of land used in production","7530","expense","l10n_lv.chart_template_latvia",,"False"
"a7540","Insurance payments (except employee insurance)","7540","expense","l10n_lv.chart_template_latvia",,"False"
"a7550","Other operating expenses","7550","expense","l10n_lv.chart_template_latvia",,"False"
"a7560","Recruitment and training expenses","7560","expense","l10n_lv.chart_template_latvia",,"False"
"a7570","Business trip expenses","7570","expense","l10n_lv.chart_template_latvia",,"False"
"a7580","Non-deductible VAT (to be included in costs)","7580","expense","l10n_lv.chart_template_latvia",,"False"
"a7590","Real estate tax for the year under review","7590","expense","l10n_lv.chart_template_latvia",,"False"
"a7610","Costs of packaging material and containers","7610","expense","l10n_lv.chart_template_latvia",,"False"
"a7620","Cost of goods transportation","7620","expense","l10n_lv.chart_template_latvia",,"False"
"a7630","Insurance of goods transportation","7630","expense","l10n_lv.chart_template_latvia",,"False"
"a7640","Commissions paid","7640","expense","l10n_lv.chart_template_latvia",,"False"
"a7650","Other sales expenses","7650","expense","l10n_lv.chart_template_latvia",,"False"
"a7710","Communication expenses","7710","expense","l10n_lv.chart_template_latvia",,"False"
"a7720","Office expenses","7720","expense","l10n_lv.chart_template_latvia",,"False"
"a7730","Cost of legal services","7730","expense","l10n_lv.chart_template_latvia",,"False"
"a7740","Annual report and audit expenses","7740","expense","l10n_lv.chart_template_latvia",,"False"
"a7750","Expenses of cash circulation","7750","expense","l10n_lv.chart_template_latvia",,"False"
"a7760","Transport expenses for administration","7760","expense","l10n_lv.chart_template_latvia",,"False"
"a7770","Other management and administration expenses","7770","expense","l10n_lv.chart_template_latvia",,"False"
"a7780","Representation expenses","7780","expense","l10n_lv.chart_template_latvia",,"False"
"a7810","Expenses of previous periods to be included in the reporting period","7810","expense","l10n_lv.chart_template_latvia",,"False"
"a7910","Apartment management expenses","7910","expense","l10n_lv.chart_template_latvia",,"False"
"a7920","Utility expenses","7920","expense","l10n_lv.chart_template_latvia",,"False"
"a7930","Household service expenses","7930","expense","l10n_lv.chart_template_latvia",,"False"
"a7940","Catering expenses","7940","expense","l10n_lv.chart_template_latvia",,"False"
"a7950","Expenses of educational institutions","7950","expense","l10n_lv.chart_template_latvia",,"False"
"a7960","Medical service expenses","7960","expense","l10n_lv.chart_template_latvia",,"False"
"a7970","Expenses of cultural and sports institutions and events","7970","expense","l10n_lv.chart_template_latvia",,"False"
"a7980","Other social infrastructure maintenance expenses","7980","expense","l10n_lv.chart_template_latvia",,"False"
"a8110","Income from participations, securities and loans","8110","income_other","l10n_lv.chart_template_latvia",,"False"
"a8120","Other income from interest and similar income","8120","income_other","l10n_lv.chart_template_latvia",,"False"
"a8130","Bill discount income","8130","income_other","l10n_lv.chart_template_latvia",,"False"
"a8140","Donations and other similar income","8140","income_other","l10n_lv.chart_template_latvia",,"False"
"a8150","Income from exchange rate appreciation","8150","income_other","l10n_lv.chart_template_latvia",,"False"
"a8160","Income from fines and contractual penalties received","8160","income_other","l10n_lv.chart_template_latvia",,"False"
"a8170","Profit from the sale or purchase of foreign currency","8170","income_other","l10n_lv.chart_template_latvia",,"False"
"a8190","Other income","8190","income_other","l10n_lv.chart_template_latvia",,"False"
"a8199","Money margin revenue","8199","income_other","l10n_lv.chart_template_latvia",,"False"
"a8210","Write-down of the value of short-term financial investments","8210","expense","l10n_lv.chart_template_latvia",,"False"
"a8220","Interest paid and related expenses","8220","expense","l10n_lv.chart_template_latvia",,"False"
"a8230","Costs of promissory notes discounts","8230","expense","l10n_lv.chart_template_latvia",,"False"
"a8240","Payment of interest on long-term loans","8240","expense","l10n_lv.chart_template_latvia",,"False"
"a8250","Losses from currency depreciation","8250","expense","l10n_lv.chart_template_latvia",,"False"
"a8260","Paid fines and liquidated damages","8260","expense","l10n_lv.chart_template_latvia",,"False"
"a8270","Losses from buying or selling foreign currency","8270","expense","l10n_lv.chart_template_latvia",,"False"
"a8290","Other expenses","8290","expense","l10n_lv.chart_template_latvia",,"False"
"a8299","Money difference losses","8299","expense","l10n_lv.chart_template_latvia",,"False"
"a8310","Extra income","8310","income_other","l10n_lv.chart_template_latvia",,"False"
"a8311","Money difference income","8311","income_other","l10n_lv.chart_template_latvia",,"False"
"a8410","Extra expenses","8410","expense","l10n_lv.chart_template_latvia",,"False"
"a8411","Money variation losses","8411","expense","l10n_lv.chart_template_latvia",,"False"
"a8610","Profit or loss","8610","expense","l10n_lv.chart_template_latvia",,"False"
"a8810","Corporate income tax","8810","expense","l10n_lv.chart_template_latvia",,"False"
"a8820","Tax on the exploitation of natural resources","8820","expense","l10n_lv.chart_template_latvia",,"False"
"a8840","Other taxes not included in the operating expenses","8840","expense","l10n_lv.chart_template_latvia",,"False"
"a9910","Balance entry account","9910","asset_current","l10n_lv.chart_template_latvia",,"False"

```

## File: data\account.group.template.csv

```csv
"id","code_prefix_start","name","chart_template_id/id"
"l10n_lv_group_1","1","Long-term investments","l10n_lv.chart_template_latvia"
"l10n_lv_group_11","11","Intangible assets","l10n_lv.chart_template_latvia"
"l10n_lv_group_12","12","Fixed assets (property, plant and equipment, investment property and biological assets)","l10n_lv.chart_template_latvia"
"l10n_lv_group_13","13","Investment properties","l10n_lv.chart_template_latvia"
"l10n_lv_group_14","14","Biological assets (Animals and plants)","l10n_lv.chart_template_latvia"
"l10n_lv_group_15","15","Long-term financial investments","l10n_lv.chart_template_latvia"
"l10n_lv_group_2","2","Current assets","l10n_lv.chart_template_latvia"
"l10n_lv_group_21","21","Inventory","l10n_lv.chart_template_latvia"
"l10n_lv_group_22","22","Animals and plants","l10n_lv.chart_template_latvia"
"l10n_lv_group_23","23","Settlements with debtors","l10n_lv.chart_template_latvia"
"l10n_lv_group_24","24","Deferred expenses","l10n_lv.chart_template_latvia"
"l10n_lv_group_25","25","Short-term financial investments","l10n_lv.chart_template_latvia"
"l10n_lv_group_26","26","Money","l10n_lv.chart_template_latvia"
"l10n_lv_group_28","28","Long-term investments held for sale","l10n_lv.chart_template_latvia"
"l10n_lv_group_3","3","Equity","l10n_lv.chart_template_latvia"
"l10n_lv_group_31","31","Share capital or participation capital, revaluation reserve for long-term investments","l10n_lv.chart_template_latvia"
"l10n_lv_group_32","32","Private accounts (private investment and consumption of money)","l10n_lv.chart_template_latvia"
"l10n_lv_group_33","33","Reserves","l10n_lv.chart_template_latvia"
"l10n_lv_group_34","34","Profit or loss","l10n_lv.chart_template_latvia"
"l10n_lv_group_4","4","Accruals","l10n_lv.chart_template_latvia"
"l10n_lv_group_41","41","Provisions for pensions and similar liabilities","l10n_lv.chart_template_latvia"
"l10n_lv_group_42","42","Provisions for expected taxes","l10n_lv.chart_template_latvia"
"l10n_lv_group_43","43","Other provisions","l10n_lv.chart_template_latvia"
"l10n_lv_group_5","5","Creditors","l10n_lv.chart_template_latvia"
"l10n_lv_group_51","51","Settlement of loans","l10n_lv.chart_template_latvia"
"l10n_lv_group_52","52","Settlement of advances received from customers","l10n_lv.chart_template_latvia"
"l10n_lv_group_53","53","Settlements with suppliers and contractors","l10n_lv.chart_template_latvia"
"l10n_lv_group_54","54","Promissory notes payable","l10n_lv.chart_template_latvia"
"l10n_lv_group_55","55","Settlements with companies, members and staff","l10n_lv.chart_template_latvia"
"l10n_lv_group_56","56","Settlement of wages and deductions (excluding taxes)","l10n_lv.chart_template_latvia"
"l10n_lv_group_57","57","Tax settlement","l10n_lv.chart_template_latvia"
"l10n_lv_group_58","58","Dividends settlement","l10n_lv.chart_template_latvia"
"l10n_lv_group_59","59","Deferred income","l10n_lv.chart_template_latvia"
"l10n_lv_group_6","6","Revenue from business activities","l10n_lv.chart_template_latvia"
"l10n_lv_group_61","61","Business income subject to general taxation","l10n_lv.chart_template_latvia"
"l10n_lv_group_62","62","Revenue from sales subject to reduced taxation","l10n_lv.chart_template_latvia"
"l10n_lv_group_63","63","Commission, brokerage and other income","l10n_lv.chart_template_latvia"
"l10n_lv_group_64","64","Revenue-reducing discounts","l10n_lv.chart_template_latvia"
"l10n_lv_group_65","65","Other operating revenue","l10n_lv.chart_template_latvia"
"l10n_lv_group_66","66","Changes in stocks and value of finished products and work in progress","l10n_lv.chart_template_latvia"
"l10n_lv_group_67","67","Revenues of other periods relating to the reporting period","l10n_lv.chart_template_latvia"
"l10n_lv_group_69","69","Revenue from social infrastructure","l10n_lv.chart_template_latvia"
"l10n_lv_group_7","7","Cost of economic activity","l10n_lv.chart_template_latvia"
"l10n_lv_group_71","71","Costs of raw materials, supplies and goods","l10n_lv.chart_template_latvia"
"l10n_lv_group_72","72","Staff costs","l10n_lv.chart_template_latvia"
"l10n_lv_group_73","73","Social costs and charges","l10n_lv.chart_template_latvia"
"l10n_lv_group_74","74","Depreciation of property, plant and equipment and write-downs of other investments","l10n_lv.chart_template_latvia"
"l10n_lv_group_75","75","Other operating expenses","l10n_lv.chart_template_latvia"
"l10n_lv_group_76","76","Cost of goods sold","l10n_lv.chart_template_latvia"
"l10n_lv_group_77","77","Administration costs","l10n_lv.chart_template_latvia"
"l10n_lv_group_78","78","Prior period costs to be included in the reporting period","l10n_lv.chart_template_latvia"
"l10n_lv_group_79","79","Social infrastructure maintenance costs","l10n_lv.chart_template_latvia"
"l10n_lv_group_8","8","Miscellaneous income and expenses, profit and loss","l10n_lv.chart_template_latvia"
"l10n_lv_group_81","81","Miscellaneous revenues","l10n_lv.chart_template_latvia"
"l10n_lv_group_82","82","Different costs","l10n_lv.chart_template_latvia"
"l10n_lv_group_83","83","Extra income","l10n_lv.chart_template_latvia"
"l10n_lv_group_84","84","Extraordinary costs","l10n_lv.chart_template_latvia"
"l10n_lv_group_86","86","Profit or loss","l10n_lv.chart_template_latvia"
"l10n_lv_group_88","88","Taxes on profits and other taxes not included in operating expenses","l10n_lv.chart_template_latvia"
"l10n_lv_group_9","9","Additional work accounts","l10n_lv.chart_template_latvia"

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <!-- Chart of Accounts Template -->
    <record id="chart_template_latvia" model="account.chart.template">
        <field name="name">Latvia - Accounting</field>
        <field name="code_digits">4</field>
        <field name="currency_id" ref="base.EUR" />
        <!-- Anglo-Saxon Accounting is used for reporting cost of goods sold
            when products are sold/delivered.-->
        <field name="use_anglo_saxon" eval="True" />
        <field name="spoken_languages" eval="'lv_LV'" />
        <field name="bank_account_code_prefix">2620</field>
        <field name="cash_account_code_prefix">2610</field>
        <field name="transfer_account_code_prefix">2699</field>
        <field name="country_id" ref="base.lv" />
    </record>
</odoo>

```

## File: data\account_chart_template_load.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <function model="account.chart.template" name="try_loading">
        <value eval="[ref('l10n_lv.chart_template_latvia')]"/>
    </function>
</odoo>

```

## File: data\account_chart_template_setup_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <!-- Setup Chart of Accounts Template -->
    <record id="chart_template_latvia" model="account.chart.template">
        <field name="property_account_receivable_id" ref="a2310"/>
        <field name="property_account_payable_id" ref="a5310"/>
        <field name="property_account_expense_categ_id" ref="a7550"/>
        <field name="property_account_income_categ_id" ref="a6110"/>
        <field name="income_currency_exchange_account_id" ref="a8150"/>
        <field name="expense_currency_exchange_account_id" ref="a8250"/>
        <field name="default_pos_receivable_account_id" ref="a2613"/>
        <field name="default_cash_difference_expense_account_id" ref="a8299"/>
        <field name="default_cash_difference_income_account_id" ref="a8199"/>
        <field name="account_journal_early_pay_discount_loss_account_id" ref="a8299"/>
        <field name="account_journal_early_pay_discount_gain_account_id" ref="a8199"/>
        <field name="account_journal_suspense_account_id" ref="a26291"/>
        <field name="account_journal_payment_credit_account_id" ref="a26293"/>
        <field name="account_journal_payment_debit_account_id" ref="a26292"/>
        <field name="account_journal_suspense_account_id" ref="a26291"/>
    </record>
</odoo>

```

## File: data\account_fiscal_position_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <!-- Account Fiscal Position Template -->
    <!--For Latvian VAT/non VAT tax subjects-->
    <record id="fiscal_position_lv" model="account.fiscal.position.template">
        <field name="name">Latvia</field>
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="True"/>
        <field name="country_id" ref="base.lv"/>
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence" eval="10"/>
    </record>

    <!--For all EU companies -->
    <record id="fiscal_position_eu" model="account.fiscal.position.template">
        <field name="name">EU Companies</field>
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="True"/>
        <field name="country_group_id" ref="base.europe"/>
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence" eval="20"/>
    </record>

    <!--For all EU consumers -->
    <record id="fiscal_position_eu_private" model="account.fiscal.position.template">
        <field name="name">EU Consumer</field>
        <field name="auto_apply" eval="True"/>
        <field name="country_group_id" ref="base.europe"/>
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence" eval="30"/>
    </record>

    <!--For all 3rd country members -->
    <record id="fiscal_position_ex" model="account.fiscal.position.template">
        <field name="name">Outside EU</field>
        <field name="auto_apply" eval="True"/>
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence" eval="40"/>
    </record>

    <!--EU members-->
    <record id="account_fiscal_position_tax_template_eu_sale_21" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_eu"/>
        <field name="tax_src_id" ref="tax_sale_vat_21"/>
        <field name="tax_dest_id" ref="tax_sale_vat_0_eu_g"/>
    </record>

    <record id="account_fiscal_position_tax_template_eu_sale_reverse" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_eu"/>
        <field name="tax_src_id" ref="tax_sale_vat_reverse"/>
        <field name="tax_dest_id" ref="tax_sale_vat_0_eu_g"/>
    </record>

    <record id="account_fiscal_position_tax_template_eu_sale_12" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_eu"/>
        <field name="tax_src_id" ref="tax_sale_vat_12"/>
        <field name="tax_dest_id" ref="tax_sale_vat_0_eu_g"/>
    </record>

    <record id="account_fiscal_position_tax_template_eu_sale_5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_eu"/>
        <field name="tax_src_id" ref="tax_sale_vat_5"/>
        <field name="tax_dest_id" ref="tax_sale_vat_0_eu_g"/>
    </record>

    <record id="account_fiscal_position_tax_template_eu_purchase_21" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_eu"/>
        <field name="tax_src_id" ref="tax_purchase_vat_21"/>
        <field name="tax_dest_id" ref="tax_purchase_vat_21_eu"/>
    </record>

    <record id="account_fiscal_position_tax_template_eu_purchase_12" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_eu"/>
        <field name="tax_src_id" ref="tax_purchase_vat_12"/>
        <field name="tax_dest_id" ref="tax_purchase_vat_12_eu"/>
    </record>

    <record id="account_fiscal_position_tax_template_eu_purchase_5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_eu"/>
        <field name="tax_src_id" ref="tax_purchase_vat_5"/>
        <field name="tax_dest_id" ref="tax_purchase_vat_5_eu"/>
    </record>

    <!-- Extracom members-->
    <record id="account_fiscal_position_tax_template_ex_sale_5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_ex"/>
        <field name="tax_src_id" ref="tax_sale_vat_5"/>
        <field name="tax_dest_id" ref="tax_sale_vat_0_ex_g"/>
    </record>

    <record id="account_fiscal_position_tax_template_ex_sale_12" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_ex"/>
        <field name="tax_src_id" ref="tax_sale_vat_12"/>
        <field name="tax_dest_id" ref="tax_sale_vat_0_ex_g"/>
    </record>

    <record id="account_fiscal_position_tax_template_ex_sale_reverse" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_ex"/>
        <field name="tax_src_id" ref="tax_sale_vat_reverse"/>
        <field name="tax_dest_id" ref="tax_sale_vat_0_ex_g"/>
    </record>

    <record id="account_fiscal_position_tax_template_ex_sale_21" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_ex"/>
        <field name="tax_src_id" ref="tax_sale_vat_21"/>
        <field name="tax_dest_id" ref="tax_sale_vat_0_ex_g"/>
    </record>

    <record id="account_fiscal_position_tax_template_ex_purchase_21" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_ex"/>
        <field name="tax_src_id" ref="tax_purchase_vat_21"/>
        <field name="tax_dest_id" ref="tax_purchase_vat_21_ex_g"/>
    </record>

    <record id="account_fiscal_position_tax_template_ex_purchase_12" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_ex"/>
        <field name="tax_src_id" ref="tax_purchase_vat_12"/>
        <field name="tax_dest_id" ref="tax_purchase_vat_21_ex_g"/>
    </record>

    <record id="account_fiscal_position_tax_template_ex_purchase_5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_ex"/>
        <field name="tax_src_id" ref="tax_purchase_vat_5"/>
        <field name="tax_dest_id" ref="tax_purchase_vat_21_ex_g"/>
    </record>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo noupdate="1">
    <record id="tax_group_vat_0" model="account.tax.group">
        <field name="name">VAT 0%</field>
        <field name="country_id" ref="base.lv"/>
    </record>
    <record id="tax_group_vat_exempt" model="account.tax.group">
        <field name="name">W/O VAT</field>
        <field name="country_id" ref="base.lv"/>
    </record>
    <record id="tax_group_vat_5" model="account.tax.group">
        <field name="name">VAT 5%</field>
        <field name="country_id" ref="base.lv"/>
    </record>
    <record id="tax_group_vat_12" model="account.tax.group">
        <field name="name">VAT 12%</field>
        <field name="country_id" ref="base.lv"/>
    </record>
    <record id="tax_group_vat_21" model="account.tax.group">
        <field name="name">VAT 21%</field>
        <field name="country_id" ref="base.lv"/>
    </record>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <!-- Sales -->
    <!-- 21% VAT in Latvia -->
    <record id="tax_sale_vat_21" model="account.tax.template">
        <field name="chart_template_id" ref="chart_template_latvia" />
        <field name="description">21%</field>
        <field name="name">21%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">21.0</field>
        <field name="sequence">100</field>
        <field name="tax_group_id" ref="tax_group_vat_21" />
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('vat_report_line_41_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'plus_report_expression_ids': [ref('vat_report_line_52_balance')],
                }),
            ]"/>
    <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('vat_report_line_41_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'minus_report_expression_ids': [ref('vat_report_line_52_balance')],
                }),
            ]"/>
    </record>

    <!--  Reverse-charge VAT in Latvia -->
    <record id="tax_sale_vat_reverse" model="account.tax.template">
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence">110</field>
        <field name="description">0%</field>
        <field name="name">Reverse charge VAT</field>
        <field name="price_include" eval="0"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_exempt"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('vat_report_line_411_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('vat_report_line_411_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
    </record>

    <!-- 12% VAT in Latvia -->
    <record id="tax_sale_vat_12" model="account.tax.template">
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence">120</field>
        <field name="description">12%</field>
        <field name="name">12%</field>
        <field name="price_include" eval="0"/>
        <field name="amount">12</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_12"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('vat_report_line_42_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'plus_report_expression_ids': [ref('vat_report_line_53_balance')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('vat_report_line_42_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'minus_report_expression_ids': [ref('vat_report_line_53_balance')],
                }),
            ]"/>
    </record>

    <!-- 5% VAT in Latvia -->
    <record id="tax_sale_vat_5" model="account.tax.template">
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence">130</field>
        <field name="description">5%</field>
        <field name="name">5%</field>
        <field name="price_include" eval="0"/>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('vat_report_line_421_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'plus_report_expression_ids': [ref('vat_report_line_531_balance')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('vat_report_line_421_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'minus_report_expression_ids': [ref('vat_report_line_531_balance')],
                }),
            ]"/>
    </record>

    <!--  0% VAT services in Latvia -->
    <record id="tax_sale_vat_0_s" model="account.tax.template">
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence">140</field>
        <field name="description">0%</field>
        <field name="name">0% S</field>
        <field name="price_include" eval="0"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('vat_report_line_48_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('vat_report_line_48_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
    </record>

    <!--  0% goods export -->
    <record id="tax_sale_vat_0_ex_g" model="account.tax.template">
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence">150</field>
        <field name="description">0%</field>
        <field name="name">0% EX G</field>
        <field name="price_include" eval="0"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('vat_report_line_481_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('vat_report_line_481_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
    </record>

    <!-- 0% Exempt services export -->
    <record id="tax_sale_vat_0_ex_s" model="account.tax.template">
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence">160</field>
        <field name="description">0%</field>
        <field name="name">0% EX S</field>
        <field name="price_include" eval="0"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_exempt"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('vat_report_line_482_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('vat_report_line_482_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
    </record>

    <!-- 0% EU services-->
    <record id="tax_sale_vat_0_eu_s" model="account.tax.template">
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence">170</field>
        <field name="description">0%</field>
        <field name="name">0% EU S</field>
        <field name="price_include" eval="0"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_exempt"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('vat_report_line_482_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('vat_report_line_482_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
    </record>

    <!--  0% EU goods -->
    <record id="tax_sale_vat_0_eu_g" model="account.tax.template">
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence">180</field>
        <field name="description">0%</field>
        <field name="name">0% EU G</field>
        <field name="price_include" eval="0"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('vat_report_line_45_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
               (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('vat_report_line_45_balance')],
               }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
    </record>

    <!-- Purchases -->
    <!-- 21% VAT in Latvia -->
    <record id="tax_purchase_vat_21" model="account.tax.template">
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence">400</field>
        <field name="description">21%</field>
        <field name="name">21%</field>
        <field name="price_include" eval="0"/>
        <field name="amount">21.0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'plus_report_expression_ids': [ref('vat_report_line_62_balance')],
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
                    'account_id': ref('a5721'),
                    'minus_report_expression_ids': [ref('vat_report_line_62_balance')],
                }),
            ]"/>
    </record>

    <!-- 21% VAT in EU -->
    <record id="tax_purchase_vat_21_eu" model="account.tax.template">
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence">410</field>
        <field name="description">21%</field>
        <field name="name">21% EU</field>
        <field name="price_include" eval="0"/>
        <field name="amount">21</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('vat_report_line_50_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'plus_report_expression_ids': [ref('vat_report_line_55_balance')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'minus_report_expression_ids': [ref('vat_report_line_64_balance')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('vat_report_line_50_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'minus_report_expression_ids': [ref('vat_report_line_55_balance')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'plus_report_expression_ids': [ref('vat_report_line_64_balance')],
                }),
            ]"/>
    </record>

    <!-- 12% VAT in Latvia -->
    <record id="tax_purchase_vat_12" model="account.tax.template">
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence">420</field>
        <field name="description">12%</field>
        <field name="name">12%</field>
        <field name="price_include" eval="0"/>
        <field name="amount">12.0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_12"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'plus_report_expression_ids': [ref('vat_report_line_62_balance')],
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
                    'account_id': ref('a5721'),
                    'minus_report_expression_ids': [ref('vat_report_line_62_balance')],
                }),
            ]"/>
    </record>

    <!-- 12% VAT in EU -->
    <record id="tax_purchase_vat_12_eu" model="account.tax.template">
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence">430</field>
        <field name="description">12%</field>
        <field name="name">12% EU</field>
        <field name="price_include" eval="0"/>
        <field name="amount">12</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_12"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('vat_report_line_51_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'plus_report_expression_ids': [ref('vat_report_line_56_balance')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'minus_report_expression_ids': [ref('vat_report_line_64_balance')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('vat_report_line_51_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'minus_report_expression_ids': [ref('vat_report_line_56_balance')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'plus_report_expression_ids': [ref('vat_report_line_64_balance')],
                }),
            ]"/>
    </record>

    <!-- 5% VAT in Latvia -->
    <record id="tax_purchase_vat_5" model="account.tax.template">
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence">440</field>
        <field name="description">5%</field>
        <field name="name">5%</field>
        <field name="price_include" eval="0"/>
        <field name="amount">5.0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'plus_report_expression_ids': [ref('vat_report_line_62_balance')],
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
                    'account_id': ref('a5721'),
                    'minus_report_expression_ids': [ref('vat_report_line_62_balance')],
                }),
            ]"/>
    </record>

    <!-- 5% VAT in EU -->
    <record id="tax_purchase_vat_5_eu" model="account.tax.template">
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence">450</field>
        <field name="description">5%</field>
        <field name="name">5% EU</field>
        <field name="price_include" eval="0"/>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('vat_report_line_511_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'plus_report_expression_ids': [ref('vat_report_line_561_balance')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'minus_report_expression_ids': [ref('vat_report_line_64_balance')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('vat_report_line_511_balance')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'minus_report_expression_ids': [ref('vat_report_line_561_balance')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'plus_report_expression_ids': [ref('vat_report_line_64_balance')],
                }),
            ]"/>
    </record>

    <!-- 21% reverse-charge VAT -->
    <record id="tax_purchase_vat_21_reverse" model="account.tax.template">
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence">460</field>
        <field name="description">21%</field>
        <field name="name">21% reverse charge</field>
        <field name="price_include" eval="0"/>
        <field name="amount">21</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
               (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
               }),
               (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'plus_report_expression_ids': [ref('vat_report_line_52_balance')],
               }),
               (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'minus_report_expression_ids': [ref('vat_report_line_62_balance')],
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
                    'account_id': ref('a5721'),
                    'minus_report_expression_ids': [ref('vat_report_line_52_balance')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'plus_report_expression_ids': [ref('vat_report_line_62_balance')],
                }),
            ]"/>
    </record>

    <!-- 21% reverse-charge for Extracom goods -->
    <record id="tax_purchase_vat_21_ex_g" model="account.tax.template">
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence">470</field>
        <field name="description">21%</field>
        <field name="name">21% EX G</field>
        <field name="price_include" eval="0"/>
        <field name="amount">21</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
               (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
               }),
               (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'plus_report_expression_ids': [ref('vat_report_line_52_balance')],
               }),
               (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'minus_report_expression_ids': [ref('vat_report_line_63_balance')],
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
                    'account_id': ref('a5721'),
                    'minus_report_expression_ids': [ref('vat_report_line_52_balance')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'plus_report_expression_ids': [ref('vat_report_line_63_balance')],
                }),
            ]"/>
    </record>

    <!-- 21% reverse-charge for Extracom services -->
    <record id="tax_purchase_vat_21_ex_s" model="account.tax.template">
        <field name="chart_template_id" ref="chart_template_latvia"/>
        <field name="sequence">480</field>
        <field name="description">21%</field>
        <field name="name">21% EX S</field>
        <field name="price_include" eval="0"/>
        <field name="amount">21</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
               (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
               }),
               (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'plus_report_expression_ids': [ref('vat_report_line_54_balance')],
               }),
               (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'minus_report_expression_ids': [ref('vat_report_line_63_balance')],
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
                    'account_id': ref('a5721'),
                    'minus_report_expression_ids': [ref('vat_report_line_54_balance')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a5721'),
                    'plus_report_expression_ids': [ref('vat_report_line_63_balance')],
                }),
            ]"/>
    </record>
</odoo>

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
                <field name="figure_type">none</field>
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

