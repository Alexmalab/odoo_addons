# Odoo Module: l10n_be

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, SUPERUSER_ID

from . import models

def load_translations(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    env.ref('l10n_be.l10nbe_chart_template').process_coa_translations()

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Belgium - Accounting',
    'version': '2.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the base module to manage the accounting chart for Belgium in Odoo.
==============================================================================

After installing this module, the Configuration wizard for accounting is launched.
    * We have the account templates which can be helpful to generate Charts of Accounts.
    * On that particular wizard, you will be asked to pass the name of the company,
      the chart template to follow, the no. of digits to generate, the code for your
      account and bank account, currency to create journals.

Thus, the pure copy of Chart Template is generated.

Wizards provided by this module:
--------------------------------
    * Partner VAT Intra: Enlist the partners with their related VAT and invoiced
      amounts. Prepares an XML file format.

        **Path to access :** Invoicing/Reporting/Legal Reports/Belgium Statements/Partner VAT Intra
    * Periodical VAT Declaration: Prepares an XML file for Vat Declaration of
      the Main company of the User currently Logged in.

        **Path to access :** Invoicing/Reporting/Legal Reports/Belgium Statements/Periodical VAT Declaration
    * Annual Listing Of VAT-Subjected Customers: Prepares an XML file for Vat
      Declaration of the Main company of the User currently Logged in Based on
      Fiscal year.

        **Path to access :** Invoicing/Reporting/Legal Reports/Belgium Statements/Annual Listing Of VAT-Subjected Customers

    """,
    'author': 'Noviat, Odoo SA',
    'depends': [
        'account',
        'base_iban',
        'base_vat',
        'l10n_multilang',
    ],
    'data': [
        'data/account_chart_template_data.xml',
        'data/account.account.template.csv',
        'data/account_pcmn_belgium_data.xml',
        'data/account_data.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_template_data.xml',
        'data/l10n_be_sequence_data.xml',
        'data/fiscal_templates_data.xml',
        'data/account_fiscal_position_tax_template_data.xml',
        'data/account_reconcile_model_template.xml',
        'data/account.group.template.csv',
        'data/account_chart_template_configure_data.xml',
        'data/menuitem_data.xml',
    ],
    'demo': [
        'demo/l10n_be_demo.xml',
        'demo/demo_company.xml',
    ],
    'post_init_hook': 'load_translations',
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","tag_ids/id","reconcile"
"a000","Company creditors, beneficiaries of third party guarantees","000","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a001","Third party guarantees on behalf of the company","001","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a010","Accounts receivable for commitments on bills in circulation","010","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a0110","Creditors of commitments on bills in circulation - Bids ceded by the company under its backing","0110","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a0111","Creditors of commitments on notes in circulation - Other commitments on notes in circulation","0111","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a012","Accounts receivable for other personal guarantees","012","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a013","Creditors of other personal guarantees","013","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a020","Company creditors, beneficiaries of real guarantees","020","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a021","Actual guarantees established for own account","021","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a022","Creditors of third parties, beneficiaries of real guarantees","022","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a023","Real guarantees provided on behalf of third parties","023","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a030","Statutory deposits","030","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a031","Statutory applicants","031","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a032","Guarantees received","032","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a033","Constituents of guarantees","033","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a040","Third parties, holders in their name but at the risks and profits of the business of goods and values","040","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a041","Goods and securities held by third parties on their behalf but at the risk and profit of the company","041","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a050","Acquisition commitments","050","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a051","Creditors of acquisition commitments","051","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a052","Accounts receivable for assignment commitments","052","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a053","Sale commitment","053","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a060","Forward transactions - Goods purchased (to be received)","060","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a061","Creditors for goods purchased at term","061","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a062","Accounts receivable for goods sold forward","062","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a063","Forward transactions - Goods sold (to be delivered)","063","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a064","Forward transactions - Currencies purchased (to be received)","064","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a065","Creditors for forward currency purchases","065","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a066","Accounts receivable for currencies sold forward","066","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a067","Forward transactions - Currencies sold (to be delivered)","067","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a0700","Long-term usage rights - On land and buildings","0700","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a0701","Long-term usage rights - On installations, machines and tools","0701","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a0702","Long-term usage rights - On furniture and rolling stock","0702","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a071","Rent and royalty creditors","071","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a072","Goods and values ​​from third parties received on deposit, consignment or custom","072","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a073","Principals and depositors of goods and securities","073","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a074","Goods and securities held for accounts or at the risk and profit of third parties","074","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a075","Creditors of property and securities held on behalf of third parties or at their risk and profit","075","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a090","Concordat resolution commitments","090","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a091","Concordat resolution claims","091","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a092","Creditors under debt restructuring conditions","092","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a093","Duties on loan conditions","093","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a094","Ongoing litigation","094","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a095","Creditors of pending litigation","095","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a096","Debtors on technical guarantees","096","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a097","Rights on technical guarantees","097","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a098","Holders of options (buying or selling securities)","098","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a099","Options (buy or sell) on securities issued.","099","account.data_account_off_sheet","l10n_be.l10nbe_chart_template","","False"
"a100","Issued capital","100","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a101","Uncalled capital","101","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a11","Share premium account","11","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a120","Revaluation surpluses on intangible fixed assets","120","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a121","Revaluation surpluses on tangible fixed assets","121","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a122","Revaluation surpluses on financial fixed assets","122","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a123","Revaluation surpluses on stocks","123","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a124","Decrease in amounts written down current investments","124","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a130","Legal reserve","130","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a1310","Reserves not available in respect of own shares held","1310","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a1311","Other reserves not available","1311","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a132","Untaxed reserves","132","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a133","Available reserves","133","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a140","Deferred profit","140","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a141","Loss carried forward","141","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a15","Investment grants","15","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a151","Investment grants received in cash","151","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a152","Investment grants received in kind","152","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a160","Provisions for pensions and similar obligations","160","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a161","Provisions for taxation","161","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a162","Provisions for major repairs and maintenance","162","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a163","Provisions for environmental obligations","163","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1680","Deferred taxes on investment grants","1680","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1681","Deferred taxes on gain on disposal of intangible fixed assets","1681","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1682","Deferred taxes on gain on disposal of tangible fixed assets","1682","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1687","Deferred taxes on gain on disposal of securities issued by Belgian public authorities","1687","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1688","Foreign deferred taxes","1688","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1700","Subordinated loans with a remaining term of more than one year - Convertible bonds","1700","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1701","Subordinated loans with a remaining term of more than one year - Non convertible bonds","1701","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1710","Unsubordinated debentures with a remaining term of more than one year - Convertible bonds","1710","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1711","Unsubordinated debentures with a remaining term of more than one year - Non convertible bonds","1711","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1730","Amounts payable to credit institutions with a remaining term of more than one year - Current account payable","1730","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1731","Amounts payable to credit institutions with a remaining term of more than one year - Promissory notes","1731","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1732","Amounts payable to credit institutions with a remaining term of more than one year - Bank acceptances","1732","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a174","Other loans with a remaining term of more than one year","174","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1750","Suppliers (more than one year)","1750","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1751","Bills of exchange payable after more than one year","1751","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a176","Advances received on contracts in progress (more than one year)","176","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a178","Amounts payable with a remaining term of more than one year - Guarantees received in cash","178","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1790","Miscellaneous amounts payable with a remaining term of more than one year - Interest-bearing","1790","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1791","Miscellaneous amounts payable with a remaining term of more than one year - Non interest-bearing or with an abnormally low interest rate","1791","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1792","Miscellaneous amounts payable with a remaining term of more than one year - Cash Deposit","1792","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a19","Advance to associates on the sharing out of the assets","19","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a200","Formation or capital increase expenses","200","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a201","Loan issue expenses","201","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a202","Other formation expenses","202","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a204","Restructuring costs","204","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a210","Research and development costs","210","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a211","Concessions, patents, licences, know-how, brands and similar rights","211","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a212","Goodwill","212","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a213","Intangible fixed assets - Advance payments","213","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a220","Land","220","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2201","Land owned by the association or the foundation in full property","2201","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2202","Other land","2202","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a221","Buildings","221","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2211","Building owned by the association or the foundation in full property","2211","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2212","Other building","2212","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a222","Developed land","222","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2221","Built-up lands owned by the association or the foundation in full property","2221","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2222","Other built-up lands","2222","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a223","Other rights to immovable property","223","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2231","Other rights to immovable property belonging to the association or the foundation in full property","2231","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2232","Other rights to immovable property - Other","2232","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a23","Plant, machinery and equipment","23","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a231","Plant, machinery and equipment owned by the association or the foundation in full property","231","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a232","Other plant, machinery and equipment","232","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a24","Furniture and vehicles","24","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a241","Furniture and vehicles owned by the association or the foundation in full property","241","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a242","Other furniture and vehicles","242","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a250","Leasing and similar rights - Land and buildings","250","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a251","Leasing and similar rights - Plant, machinery and equipment","251","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a252","Leasing and similar rights - Furniture and vehicles","252","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a26","Other tangible fixed assets","26","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a261","Other tangible fixed assets owned by the association or the foundation in full property","261","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a262","Other tangible fixed assets - Other","262","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a27","Tangible fixed assets under construction and advance payments","27","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2800","Participating interests and shares in associated enterprises - Acquisition value","2800","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2801","Participating interests and shares in associated enterprises - Uncalled amounts","2801","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2808","Participating interests and shares in associated enterprises - Revaluation surpluses","2808","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2809","Participating interests and shares in associated enterprises - Amounts written down","2809","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2810","Amounts receivable from affiliated enterprises - Current account","2810","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2811","Amounts receivable from affiliated enterprises - Bills receivable","2811","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2812","Amounts receivable from affiliated enterprises - Fixed income securities","2812","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2817","Other amounts receivable from affiliated enterprises - Doubtful amounts","2817","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2819","Amounts receivable from affiliated enterprises - Amounts written down","2819","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2820","Participating interests and shares in enterprises linked by a participating interest - Acquisition value","2820","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2821","Participating interests and shares in enterprises linked by a participating interest - Uncalled amounts","2821","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2828","Participating interests and shares in enterprises linked by a participating interest - Revaluation surpluses","2828","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2829","Participating interests and shares in enterprises linked by a participating interest - Amounts written down","2829","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2830","Amounts receivable from other enterprises linked by participating interests - Current account","2830","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2831","Amounts receivable from other enterprises linked by participating interests - Bills receivable","2831","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2832","Amounts receivable from other enterprises linked by participating interests - Fixed income securities","2832","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2837","Amounts receivable from other enterprises linked by participating interests - Doubtful amounts","2837","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2839","Amounts receivable from other enterprises linked by participating interests - Amounts written down","2839","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2840","Other participating interests and shares - Acquisition value","2840","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2841","Other participating interests and shares - Uncalled amounts","2841","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2848","Other participating interests and shares - Revaluation surpluses","2848","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2849","Other participating interests and shares - Amounts written down","2849","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2850","Other financial assets - Current account","2850","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2851","Other financial assets - Bills receivable","2851","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2852","Other financial assets - Fixed income securities","2852","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2857","Other financial assets - Doubtful amounts","2857","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2859","Other financial assets - Amounts written down","2859","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a288","Other financial assets - Cash Guarantees","288","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2900","Trade debtors after more than one year - Customer","2900","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2901","Trade debtors after more than one year - Bills receivable","2901","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2906","Trade debtors after more than one year - Advance payments","2906","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2907","Trade debtors after more than one year - Doubtful amounts","2907","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2909","Trade debtors after more than one year - Amounts written down","2909","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2910","Other amounts receivable after more than one year - Current account","2910","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2911","Other amounts receivable after more than one year - Bills receivable","2911","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2915","Non interest-bearing amounts receivable after more than one year or with an abnormally low interest rate","2915","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2917","Other amounts receivable after more than one year - Doubtful amounts","2917","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a2919","Other amounts receivable after more than one year - Amounts written down","2919","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a300","Raw materials - Acquisition value","300","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a309","Raw materials - amounts written down","309","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a310","Consumables - Acquisition value","310","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a319","Consumables - amounts written down","319","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a320","Work in progress - Acquisition value","320","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a329","Work in progress - amounts written down","329","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a330","Finished goods - Acquisition value","330","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a339","Finished goods - amounts written down","339","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a340","Goods purchased for resale - Acquisition value","340","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a349","Goods purchased for resale - amounts written down","349","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a350","Immovable property intended for sale - Acquisition value","350","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a359","Immovable property intended for sale - amounts written down","359","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a360","Advance payments on purchases for stocks - Acquisition value","360","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a369","Advance payments on purchases for stocks - amounts written down","369","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a370","Contracts in progress - Acquisition value","370","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a371","Contracts in progress - Profit recognised","371","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a379","Contracts in progress - amounts written down","379","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a400","Trade debtors within one year - Customer","400","account.data_account_type_receivable","l10n_be.l10nbe_chart_template","","True"
"a4001","Customer (POS)","4001","account.data_account_type_receivable","l10n_be.l10nbe_chart_template","","True"
"a401","Trade debtors within one year - Bills receivable","401","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a404","Trade debtors within one year - Income receivable","404","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","True"
"a406","Trade debtors within one year - Advance payments","406","account.data_account_type_receivable","l10n_be.l10nbe_chart_template","","True"
"a407","Trade debtors within one year - Doubtful amounts","407","account.data_account_type_receivable","l10n_be.l10nbe_chart_template","","True"
"a409","Trade debtors within one year - Amounts written down","409","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a410","Called up capital, unpaid","410","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a411","VAT recoverable","411","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a4112","VAT recoverable - Current Account","4112","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a412","Taxes and withholdings taxes to be recovered","412","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a4128","Taxes and withholdings taxes to be recovered - Foreign taxes","4128","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a413","Grants receivable","413","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a414","Other amounts receivable within one year - Income receivable","414","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a415","Non interest-bearing amounts receivable within one year or with an abnormally low interest rate","415","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a416","Other amounts receivable within one year - Sundry amounts","416","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a417","Other amounts receivable within one year - Doubtful amounts","417","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a418","Other amounts receivable within one year - Guarantees paid in cash","418","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a419","Other amounts receivable within one year - Amounts written down","419","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a4200","Subordinated loans payable after more than one year falling due within one year - Convertible","4200","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4201","Subordinated loans payable after more than one year falling due within one year - Non convertible","4201","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4210","Unsubordinated debentures payable after more than one year falling due within one year - Convertible","4210","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4211","Unsubordinated debentures payable after more than one year falling due within one year - Non convertible","4211","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a422","Leasing and similar obligations payable after more than one year falling due within one year","422","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4230","Amounts payable after more than one year falling due within one year to credit institutions - Current account payable","4230","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4231","Amounts payable after more than one year falling due within one year to credit institutions - Promissory notes","4231","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4232","Amounts payable after more than one year falling due within one year to credit institutions - Bank acceptances","4232","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a424","Other loans payable after more than one year falling due within one year","424","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4250","Amounts payable after more than one year falling due within one year to suppliers","4250","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4251","Bills of exchange payable after more than one year falling due within one year","4251","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a426","Advance payments received on contract in progress payable after more than one year falling due within one year","426","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a428","Amounts payable after more than one year falling due within one year - Guarantees received in cash","428","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a429","Miscellaneous amounts payable after more than one year falling due within one year","429","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a430","Amounts payable within one year to credit institutions - Fixed term loans","430","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a431","Amounts payable within one year to credit institutions - Promissory notes","431","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a432","Amounts payable within one year to credit institutions - Bank acceptances","432","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a433","Amounts payable within one year to credit institutions - Current account payable","433","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a439","Other loans payable within one year","439","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a440","Suppliers payable within one year","440","account.data_account_type_payable","l10n_be.l10nbe_chart_template","","True"
"a441","Bills of exchange payable within one year","441","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a444","Invoices to be received payable within one year","444","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","True"
"a450","Estimated taxes payable","450","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4508","Estimated taxes payable - Foreign taxes","4508","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a451","VAT payable","451","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a451054","VAT payable - compartment 54","451054","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a451055","VAT payable - Intracommunity acquisitions - box 55","451055","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a451056","VAT payable - reverse charge (cocontracting) - compartment 56","451056","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a451057","VAT payable - reverse charge (import) - compartment 57","451057","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a451063","VAT payable - credit notes - compartment 63","451063","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4512","VAT due - Current Account","4512","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a451800","VAT payable - revisions insufficiencies","451800","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a451820","VAT payable - revisions of deductions","451820","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a451830","VAT payable - revisions","451830","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a452","Taxes payable","452","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4528","Taxes payable - Foreign taxes","4528","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a453","Taxes withheld","453","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a454","Remuneration and social security - National Social Security Office","454","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a455","Remuneration and social security - Remuneration","455","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a456","Remuneration and social security - Holiday pay","456","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a459","Remuneration and social security - Other social obligations","459","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a460","Advances to be received within one year","460","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a461","Advances received","461","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a470","Dividends and director's fees relating to prior financial periods","470","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a471","Dividends - Current financial period","471","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a472","Director's fees - Current financial period","472","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a473","Other allocations","473","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a480","Miscellaneous amounts payable within one year - Debentures and matured coupons","480","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a483","Miscellaneous amounts payable within one year - Grants to repay","483","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a487","Lent securities to return","487","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a488","Miscellaneous amounts payable within one year - Guarantees received in cash","488","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4890","Miscellaneous amounts payable within one year - Sundry interest-bearing amounts payable","4890","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4891","Miscellaneous amounts payable within one year - Sundry non interest-bearing amounts payable or with an abnormally low interest rate","4891","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a490","Deferred charges","490","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a491","Accrued income","491","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a492","Accrued charges","492","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a493","Deferred income","493","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a496","Foreign currency translation differences - Assets","496","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a497","Foreign currency translation differences - Liabilities","497","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a499","Suspense account","499","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a500","Current investments other than shares, fixed income securities and term accounts - Cost","500","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a509","Current investments other than shares, fixed income securities and term accounts - Amounts written down","509","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a510","Shares and current investments other than fixed income investments - Acquisition value","510","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a511","Shares and current investments other than fixed income investments - Uncalled amount","511","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a519","Shares and current investments other than fixed income investments - Amounts written down","519","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a520","Fixed income securities - Acquisition value","520","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a529","Fixed income securities - Amounts written down","529","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a530","Fixed term deposit over one year","530","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a531","Fixed term deposit between one month and one year","531","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a532","Fixed term deposit up to one month","532","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a539","Fixed term deposit - Amounts written down","539","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a54","Cash at bank - Amounts overdue and in the process of collection","54","account.data_account_type_liquidity","l10n_be.l10nbe_chart_template","","False"
"a55","Cash at bank - Credit institutions","55","account.data_account_type_liquidity","l10n_be.l10nbe_chart_template","","False"
"a560","Cash at bank - Giro account - Bank account","560","account.data_account_type_liquidity","l10n_be.l10nbe_chart_template","","False"
"a561","Cash at bank - Giro account - Cheques issued","561","account.data_account_type_liquidity","l10n_be.l10nbe_chart_template","","False"
"a57","Cash in hand","57","account.data_account_type_liquidity","l10n_be.l10nbe_chart_template","","False"
"a578","Cash in hand - Stamps","578","account.data_account_type_liquidity","l10n_be.l10nbe_chart_template","","False"
"a58","Cash at bank and in hand - Internal transfers of funds","58","account.data_account_type_liquidity","l10n_be.l10nbe_chart_template","","False"
"a600","Purchases of raw materials","600","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a601","Purchases of consumables","601","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a602","Purchases of services, works and studies","602","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a603","Sub-contracting","603","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a604","Purchases of goods for resale","604","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a605","Purchases of immovable property for resale","605","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a608","Discounts, allowance and rebates received on purchase of raw materials, consumables","608","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6090","Decrease (increase) in stocks of raw materials","6090","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6091","Decrease (increase) in stocks of consumables","6091","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6094","Decrease (increase) in stocks of goods purchased for resale","6094","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6095","Decrease (increase) in immovable property for resale","6095","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a61","Services and other goods","61","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a617","Costs of hired temporary staff and persons placed at the enterprise's disposal","617","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a618","Remuneration, premiums for extra statutory insurance, pensions of the directors, or the management staff which are not allowed following the contract","618","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6200","Remuneration and direct social benefits - Directors and managers","6200","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6201","Remuneration and direct social benefits - Executive","6201","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6202","Remuneration and direct social benefits - Employees","6202","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6203","Remuneration and direct social benefits - Manual workers","6203","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6204","Remuneration and direct social benefits - Other staff members","6204","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a621","Employers' contribution for social security","621","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a622","Employers' premiums for extra statutory insurance","622","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a623","Other personnel costs","623","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6240","Retirement and survivors' pensions - Directors and managers","6240","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6241","Retirement and survivors' pensions - Personnel","6241","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6300","Depreciation of formation expenses","6300","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6301","Depreciation of intangible fixed assets","6301","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6302","Depreciation of tangible fixed assets","6302","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6308","Amounts written off intangible fixed assets","6308","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6309","Amounts written off tangible fixed assets","6309","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6310","Amounts written off stocks - Appropriations","6310","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6311","Amounts written off stocks - Write-backs","6311","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6320","Amounts written off contracts in progress - Appropriations","6320","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6321","Amounts written off contracts in progress - Write-backs","6321","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6330","Amounts written off trade debtors (more than one year) - Appropriations","6330","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6331","Amounts written off trade debtors (more than one year) - Write-backs","6331","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6340","Amounts written off trade debtors (within one year) - Appropriations","6340","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6341","Amounts written off trade debtors (within one year) - Write-backs","6341","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6350","Provisions for pensions and similar obligations - Appropriations","6350","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6351","Provisions for pensions and similar obligations - Uses and write-backs","6351","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6360","Provision for major repairs and maintenance - Appropriations","6360","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6361","Provision for major repairs and maintenance - Uses and write-backs","6361","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6370","Provisions for other risks and charges - Appropriations","6370","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6371","Provisions for other risks and charges - Uses (write-back)","6371","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6380","Provisions for other risks and charges - Provisions for environmental obligations excluded - Appropriations","6380","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6381","Provisions for other risks and charges - Provisions for environmental obligations excluded - Uses (write-back)","6381","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a640","Taxes related to operation","640","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a64012","Non deductible taxes","64012","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a641","Loss on ordinary disposal of tangible fixed assets","641","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a642","Loss on ordinary disposal of trade debtors","642","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a643","Operating charges - Gifts","643","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6431","Operating charges - Gifts with a recovery right","6431","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6432","Operating charges - Gifts without any recovery right","6432","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a649","Operating charges carried to assets as restructuring costs","649","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6500","Interests, commissions and other charges relating to debts","6500","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6501","Depreciation of loan issue expenses","6501","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6502","Other debt charges","6502","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6503","Capitalized Interests","6503","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6510","Amounts written off current assets except stocks, contracts in progress and trade debtors - Appropriations","6510","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6511","Amounts written off current assets except stocks, contracts in progress and trade debtors - Write-backs","6511","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a652","Losses on disposal of current assets","652","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a653","Amount of the discount borne by the enterprise, as a result of negotiating amounts receivable","653","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a654","Financial charges - Exchange differences","654","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a655","Financial charges - Foreign currency translation differences","655","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6560","Provisions of a financial nature - Appropriations","6560","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a6561","Provisions of a financial nature - Uses and write-backs","6561","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a659","Financial charges carried to assets as restructuring costs","659","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6600","Non-recurring depreciation of and amounts written off formation expenses","6600","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_investing","False"
"a6601","Non-recurring depreciation of and amounts written off intangible fixed assets","6601","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6602","Non-recurring depreciation of and amounts written off tangible fixed assets","6602","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a661","Amounts written off financial fixed assets","661","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a66200","Provisions for non-recurring operating liabilities and charges - Appropriations","66200","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a66201","Provisions for non-recurring operating liabilities and charges - Uses","66201","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a66210","Provisions for non-recurring financial liabilities and charges - Appropriations","66210","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a66211","Provisions for non-recurring financial liabilities and charges - Uses","66211","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6630","Capital losses on disposal of intangible and tangible fixed assets","6630","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6631","Capital losses on disposal of financial fixed assets","6631","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a668","Other  non-recurring financial charges","668","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6690","Non-recurring operating charges carried to assets as restructuring costs","6690","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6691","Non-recurring financial charges carried to assets as restructuring costs","6691","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6700","Belgian income taxes on the result of the current period - Income taxes paid and withholding taxes due or paid","6700","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6701","Belgian and foreign income taxes - Income taxes - Withholding taxes on immovables","6701","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6702","Belgian and foreign income taxes - Income taxes - Withholding taxes on investment income","6702","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6703","Belgian and foreign income taxes - Income taxes - Other income taxes","6703","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6710","Belgian income taxes on the result of prior periods - Additional charges for income taxes due or paid","6710","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6711","Belgian income taxes on the result of prior periods - Additional charges for estimated income taxes","6711","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6712","Belgian income taxes on the result of prior periods - Additional charges for income taxes provided for","6712","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a672","Foreign income taxes on the result of the current period","672","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a673","Foreign income taxes on the result of prior periods","673","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a680","Transfer to deferred taxes","680","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a689","Transfer to untaxed reserves","689","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a690","Loss brought forward from previous year","690","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a691","Appropriations to capital and share premium account","691","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6920","Appropriations to legal reserve","6920","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a6921","Appropriations to other reserves","6921","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a693","Profits to be carried forward","693","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a694","Dividends","694","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a695","Directors' or managers' entitlements","695","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a696","Employees' entitlements","696","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a697","Other allocations entitlements","697","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a7000","Sales rendered in Belgium (marchandises)","7000","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7001","Sales rendered in E.E.C. (marchandises)","7001","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7002","Sales rendered for export (marchandises)","7002","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7010","Sales rendered in Belgium (finished goods)","7010","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7011","Sales rendered in E.E.C. (finished goods)","7011","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7012","Sales rendered for export (finished goods)","7012","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7050","Services rendered in Belgium","7050","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7051","Services rendered in E.E.C.","7051","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7052","Services rendered for export","7052","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a708","Discounts, allowances and rebates allowed","708","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a71","Increase (decrease) in stocks of finished goods and work and contracts in progress","71","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a712","Increase (decrease) in work in progress","712","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a713","Increase (decrease) in stocks of finished goods","713","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a715","Increase (decrease) in stocks of immovable property constructed for resale","715","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a7170","Increase (decrease) in contracts in progress - Acquisition value","7170","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a7171","Increase (decrease) in contracts in progress - Profit recognized","7171","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a72","Own work capitalised","72","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a730","Contributions from effective members","730","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a731","Contributions from members","731","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a732","Gifts without any recovery right","732","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a733","Gifts with a recovery right","733","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a734","Legacies without any recovery right","734","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a735","Legacies with a recovery right","735","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a736","Contributions, gifts, legacies and grants - Investment grants and interest subsidies","736","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a737","Operating Subsidies","737","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a738","Compensatory amounts meant to reduce wage costs","738","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a740","Operating subsidies and compensatory amounts","740","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a741","Gain on ordinary disposal of tangible fixed assets","741","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a742","Gain on ordinary disposal of trade debtors","742","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a750","Income from financial fixed assets","750","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a751","Income from current assets","751","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a752","Gain on disposal of current assets","752","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a753","Investment grants and interest subsidies","753","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a754","Financial income - Exchange differences","754","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a755","Financial income - Foreign currency translation differences","755","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a7600","Write-back of depreciation and of amounts written off intangible fixed assets","7600","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_investing","False"
"a7601","Write-back of depreciation and of amounts written off tangible fixed assets","7601","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a761","Write-back of amounts written down financial fixed assets","761","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a7620","Write-back of provisions for non-recurring operating liabilities and charges","7620","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a7621","Write-back of provisions for non-recurring financial liabilities and charges","7621","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a7630","Capital gains on disposal of intangible and tangible fixed asset","7630","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a7631","Capital gains on disposal of financial fixed assets","7631","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a769","Other  non-recurring financial income","769","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a77","Adjustment of income taxes and write-back of tax provisions","77","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a7710","Adjustment of Belgian income taxes - Taxes due or paid","7710","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7711","Adjustment of Belgian income taxes - Estimated taxes","7711","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7712","Adjustment of Belgian income taxes - Tax provisions written back","7712","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a773","Adjustment of foreign income taxes","773","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a780","Transfer from deferred taxes","780","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a789","Transfer from untaxed reserves","789","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a790","Profit brought forward from previous year","790","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a791","Withdrawal from the association or foundation funds","791","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a792","Withdrawal from allocated funds","792","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a793","Losses to be carried forward","793","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a794","Owners' contribution in respect of losses","794","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"

```

## File: data\account.group.template.csv

```csv
id,code_prefix_start,code_prefix_end,name,chart_template_id/id
be_group_1,1,,"Fonds propres, provisions pour risques et charges et dettes à plus d'un an",l10n_be.l10nbe_chart_template
be_group_10,10,,"Capital",l10n_be.l10nbe_chart_template
be_group_100,100,,"Capital souscrit",l10n_be.l10nbe_chart_template
be_group_101,101,,"Capital non appelé (–)",l10n_be.l10nbe_chart_template
be_group_11,11,,"Primes d'émission",l10n_be.l10nbe_chart_template
be_group_12,12,,"Plus-values de réévaluation",l10n_be.l10nbe_chart_template
be_group_120,120,,"Plus-values de réévaluation sur immobilisations incorporelles",l10n_be.l10nbe_chart_template
be_group_121,121,,"Plus-values de réévaluation sur immobilisations corporelles",l10n_be.l10nbe_chart_template
be_group_122,122,,"Plus-values de réévaluation sur immobilisations financières",l10n_be.l10nbe_chart_template
be_group_123,123,,"Plus-values de réévaluation sur stocks",l10n_be.l10nbe_chart_template
be_group_124,124,,"Reprises de réductions de valeur sur placements de trésorerie",l10n_be.l10nbe_chart_template
be_group_13,13,,"Réserves",l10n_be.l10nbe_chart_template
be_group_130,130,,"Réserve légale",l10n_be.l10nbe_chart_template
be_group_131,131,,"Réserves indisponibles",l10n_be.l10nbe_chart_template
be_group_132,132,,"Réserves immunisées",l10n_be.l10nbe_chart_template
be_group_133,133,,"Réserves disponibles",l10n_be.l10nbe_chart_template
be_group_14,14,,"Bénéfice reporté ou Perte reportée (–)",l10n_be.l10nbe_chart_template
be_group_15,15,,"Subsides en capital",l10n_be.l10nbe_chart_template
be_group_16,16,,"Provisions et impôts différés",l10n_be.l10nbe_chart_template
be_group_160,160,,"Provisions pour pensions et obligations similaires",l10n_be.l10nbe_chart_template
be_group_161,161,,"Provisions pour charges fiscales",l10n_be.l10nbe_chart_template
be_group_162,162,,"Provisions pour grosses réparations et gros entretien",l10n_be.l10nbe_chart_template
be_group_163,163,,"Provisions pour obligations environnementales",l10n_be.l10nbe_chart_template
be_group_164,164,165,"Provisions pour autres risques et charges",l10n_be.l10nbe_chart_template
be_group_168,168,,"Impôts différés",l10n_be.l10nbe_chart_template
be_group_17,17,,"Dettes à plus d'un an",l10n_be.l10nbe_chart_template
be_group_170,170,,"Emprunts subordonnés",l10n_be.l10nbe_chart_template
be_group_171,171,,"Emprunts obligataires non subordonnés",l10n_be.l10nbe_chart_template
be_group_172,172,,"Dettes de location-financement et dettes assimilées",l10n_be.l10nbe_chart_template
be_group_173,173,,"Etablissements de crédit",l10n_be.l10nbe_chart_template
be_group_174,174,,"Autres emprunts",l10n_be.l10nbe_chart_template
be_group_175,175,,"Dettes commerciales",l10n_be.l10nbe_chart_template
be_group_176,176,,"Acomptes reçus sur commandes",l10n_be.l10nbe_chart_template
be_group_178,178,,"Cautionnements reçus en numéraire",l10n_be.l10nbe_chart_template
be_group_179,179,,"Dettes diverses",l10n_be.l10nbe_chart_template
be_group_19,19,,"Acompte aux associés sur le partage de l'actif net (-)",l10n_be.l10nbe_chart_template
be_group_2,2,,"Frais d'établissement, actifs immobilisés et créances à plus d'un an",l10n_be.l10nbe_chart_template
be_group_20,20,,"Frais d'établissement",l10n_be.l10nbe_chart_template
be_group_200,200,,"Frais de constitution et d'augmentation de capital",l10n_be.l10nbe_chart_template
be_group_201,201,,"Frais d'émission d'emprunts",l10n_be.l10nbe_chart_template
be_group_202,202,,"Autres frais d'établissement",l10n_be.l10nbe_chart_template
be_group_204,204,,"Frais de restructuration",l10n_be.l10nbe_chart_template
be_group_21,21,,"Immobilisation incorporelles",l10n_be.l10nbe_chart_template
be_group_210,210,,"Frais de recherche et de développement",l10n_be.l10nbe_chart_template
be_group_211,211,,"Concessions, brevets, licences, savoir-faire, marques et droits similaires",l10n_be.l10nbe_chart_template
be_group_212,212,,"Goodwill",l10n_be.l10nbe_chart_template
be_group_213,213,,"Acomptes versés",l10n_be.l10nbe_chart_template
be_group_22,22,,"Terrains et constructions",l10n_be.l10nbe_chart_template
be_group_220,220,,"Terrains",l10n_be.l10nbe_chart_template
be_group_221,221,,"Constructions",l10n_be.l10nbe_chart_template
be_group_222,222,,"Terrains bâtis",l10n_be.l10nbe_chart_template
be_group_223,223,,"Autres droits réels sur des immeubles",l10n_be.l10nbe_chart_template
be_group_23,23,,"Installations, machines et outillage",l10n_be.l10nbe_chart_template
be_group_24,24,,"Mobilier et matériel roulant",l10n_be.l10nbe_chart_template
be_group_25,25,,"Immobilisations détenues en location-financement et droits similaires",l10n_be.l10nbe_chart_template
be_group_250,250,,"Terrains et construction",l10n_be.l10nbe_chart_template
be_group_251,251,,"Installations, machines et outillage",l10n_be.l10nbe_chart_template
be_group_252,252,,"Mobilier et matériel roulant",l10n_be.l10nbe_chart_template
be_group_26,26,,"Autres immobilisations corporelles",l10n_be.l10nbe_chart_template
be_group_27,27,,"Immobilisations corporelles en cours et acomptes versés",l10n_be.l10nbe_chart_template
be_group_28,28,,"Immobilisations financières",l10n_be.l10nbe_chart_template
be_group_280,280,,"Participations dans des entreprises liées",l10n_be.l10nbe_chart_template
be_group_281,281,,"Créances sur des entreprises liées",l10n_be.l10nbe_chart_template
be_group_282,282,,"Participations dans des entreprises avec lesquelles il existe un lien de participation",l10n_be.l10nbe_chart_template
be_group_283,283,,"Créances sur des entreprises avec lesquelles il existe un lien de participation",l10n_be.l10nbe_chart_template
be_group_284,284,,"Autres actions et parts",l10n_be.l10nbe_chart_template
be_group_285,285,,"Autres créances",l10n_be.l10nbe_chart_template
be_group_288,288,,"Cautionnements versés en numéraire",l10n_be.l10nbe_chart_template
be_group_29,29,,"Créances à plus d'un an",l10n_be.l10nbe_chart_template
be_group_290,290,,"Créances commerciales",l10n_be.l10nbe_chart_template
be_group_291,291,,"Autres créances",l10n_be.l10nbe_chart_template
be_group_3,3,,"Stocks et commandes en cours d'exécution",l10n_be.l10nbe_chart_template
be_group_30,30,,"Approvisionnements - Matières premières",l10n_be.l10nbe_chart_template
be_group_300,300,,"Valeur d'acquisition",l10n_be.l10nbe_chart_template
be_group_309,309,,"Réductions de valeur actées (–)",l10n_be.l10nbe_chart_template
be_group_31,31,,"Approvisionnements - Fournitures",l10n_be.l10nbe_chart_template
be_group_310,310,,"Valeur d'acquisition",l10n_be.l10nbe_chart_template
be_group_319,319,,"Réductions de valeur actées (–)",l10n_be.l10nbe_chart_template
be_group_32,32,,"En-cours de fabrication",l10n_be.l10nbe_chart_template
be_group_320,320,,"Valeur d'acquisition",l10n_be.l10nbe_chart_template
be_group_329,329,,"Réductions de valeur actées (–)",l10n_be.l10nbe_chart_template
be_group_33,33,,"Produits finis",l10n_be.l10nbe_chart_template
be_group_330,330,,"Produits finis",l10n_be.l10nbe_chart_template
be_group_339,339,,"Réductions de valeur actées (–)",l10n_be.l10nbe_chart_template
be_group_34,34,,"Marchandises",l10n_be.l10nbe_chart_template
be_group_340,340,,"Valeur d'acquisition",l10n_be.l10nbe_chart_template
be_group_349,349,,"Réductions de valeur actées (–)",l10n_be.l10nbe_chart_template
be_group_35,35,,"Immeubles destinés à la vente",l10n_be.l10nbe_chart_template
be_group_350,350,,"Valeur d'acquisition",l10n_be.l10nbe_chart_template
be_group_359,359,,"Réductions de valeur actées (–)",l10n_be.l10nbe_chart_template
be_group_36,36,,"Acomptes versés sur achats pour stocks",l10n_be.l10nbe_chart_template
be_group_360,360,,"Acomptes versés",l10n_be.l10nbe_chart_template
be_group_369,369,,"Réductions de valeur actées (–)",l10n_be.l10nbe_chart_template
be_group_37,37,,"Commandes en cours d'exécution",l10n_be.l10nbe_chart_template
be_group_370,370,,"Valeur d'acquisition",l10n_be.l10nbe_chart_template
be_group_371,371,,"Bénéfice pris en compte",l10n_be.l10nbe_chart_template
be_group_379,379,,"Réductions de valeur actées (–)",l10n_be.l10nbe_chart_template
be_group_4,4,,"Créances et dettes à un an au plus",l10n_be.l10nbe_chart_template
be_group_40,40,,"Créances commerciales",l10n_be.l10nbe_chart_template
be_group_400,400,,"Clients",l10n_be.l10nbe_chart_template
be_group_401,401,,"Effets à recevoir",l10n_be.l10nbe_chart_template
be_group_404,404,,"Produits à recevoir",l10n_be.l10nbe_chart_template
be_group_406,406,,"Acomptes versés",l10n_be.l10nbe_chart_template
be_group_407,407,,"Créances douteuses",l10n_be.l10nbe_chart_template
be_group_409,409,,"Réductions de valeur actées (–)",l10n_be.l10nbe_chart_template
be_group_41,41,,"Autres créances",l10n_be.l10nbe_chart_template
be_group_410,410,,"Capital appelé, non versé",l10n_be.l10nbe_chart_template
be_group_411,411,,"T.V.A. à récupérer",l10n_be.l10nbe_chart_template
be_group_412,412,,"Impôts et précomptes à récupérer",l10n_be.l10nbe_chart_template
be_group_414,414,,"Produits à recevoir",l10n_be.l10nbe_chart_template
be_group_416,416,,"Créances diverses",l10n_be.l10nbe_chart_template
be_group_417,417,,"Créances douteuses",l10n_be.l10nbe_chart_template
be_group_418,418,,"Cautionnements versés en numéraire",l10n_be.l10nbe_chart_template
be_group_419,419,,"Réductions de valeur actées (–)",l10n_be.l10nbe_chart_template
be_group_42,42,,"Dettes à plus d'un an échéant dans l'année 16 (même subdivision que le 17)",l10n_be.l10nbe_chart_template
be_group_43,43,,"Dettes financières",l10n_be.l10nbe_chart_template
be_group_430,430,,"Etablissements de crédit - Emprunts en compte à terme fixe",l10n_be.l10nbe_chart_template
be_group_431,431,,"Etablissements de crédit - Promesses",l10n_be.l10nbe_chart_template
be_group_432,432,,"Etablissements de crédit - Crédits d'acceptation",l10n_be.l10nbe_chart_template
be_group_433,433,,"Etablissements de crédit - Dettes en compte courant",l10n_be.l10nbe_chart_template
be_group_439,439,,"Autres emprunts",l10n_be.l10nbe_chart_template
be_group_44,44,,"Dettes commerciales",l10n_be.l10nbe_chart_template
be_group_440,440,,"Fournisseurs",l10n_be.l10nbe_chart_template
be_group_441,441,,"Effets à payer",l10n_be.l10nbe_chart_template
be_group_444,444,,"Factures à recevoir",l10n_be.l10nbe_chart_template
be_group_45,45,,"Dettes fiscales, salariales et sociales",l10n_be.l10nbe_chart_template
be_group_450,450,,"Dettes fiscales estimées",l10n_be.l10nbe_chart_template
be_group_451,451,,"T.V.A. à payer",l10n_be.l10nbe_chart_template
be_group_452,452,,"Impôts et taxes à payer",l10n_be.l10nbe_chart_template
be_group_453,453,,"Précomptes retenus",l10n_be.l10nbe_chart_template
be_group_454,454,,"Office national de la sécurité sociale",l10n_be.l10nbe_chart_template
be_group_455,455,,"Rémunérations",l10n_be.l10nbe_chart_template
be_group_456,456,,"Pécules de vacances",l10n_be.l10nbe_chart_template
be_group_459,459,,"Autres dettes sociales",l10n_be.l10nbe_chart_template
be_group_46,46,,"Acomptes reçus sur commandes",l10n_be.l10nbe_chart_template
be_group_47,47,,"Dettes découlant de l'affectation du résultat",l10n_be.l10nbe_chart_template
be_group_470,470,,"Dividendes et tantièmes d'exercices antérieurs",l10n_be.l10nbe_chart_template
be_group_471,471,,"Dividendes de l'exercice",l10n_be.l10nbe_chart_template
be_group_472,472,,"Tantièmes de l'exercice",l10n_be.l10nbe_chart_template
be_group_473,473,,"Autres allocataires",l10n_be.l10nbe_chart_template
be_group_48,48,,"Dettes diverses",l10n_be.l10nbe_chart_template
be_group_480,480,,"Obligations et coupons échus",l10n_be.l10nbe_chart_template
be_group_488,488,,"Cautionnements reçus en numéraire",l10n_be.l10nbe_chart_template
be_group_489,489,,"Autres dettes diverses",l10n_be.l10nbe_chart_template
be_group_49,49,,"Comptes de régularisation et comptes d'attente",l10n_be.l10nbe_chart_template
be_group_490,490,,"Charges à reporter",l10n_be.l10nbe_chart_template
be_group_491,491,,"Produits acquis",l10n_be.l10nbe_chart_template
be_group_492,492,,"Charges à imputer",l10n_be.l10nbe_chart_template
be_group_493,493,,"Produits à reporter",l10n_be.l10nbe_chart_template
be_group_499,499,,"Comptes d'attente",l10n_be.l10nbe_chart_template
be_group_5,5,,"Placements de trésorerie et valeurs disponibles",l10n_be.l10nbe_chart_template
be_group_50,50,,"Actions propres",l10n_be.l10nbe_chart_template
be_group_51,51,,"Actions, parts et placements de trésorerie autres que placements à revenu fixe",l10n_be.l10nbe_chart_template
be_group_510,510,,"Valeur d'acquisition",l10n_be.l10nbe_chart_template
be_group_5100,5100,,"Actions et parts",l10n_be.l10nbe_chart_template
be_group_5101,5101,,"Placements de trésorerie autres que placements à revenu fixe",l10n_be.l10nbe_chart_template
be_group_511,511,,"Montants non appelés (-)",l10n_be.l10nbe_chart_template
be_group_5110,5110,,"Actions et parts",l10n_be.l10nbe_chart_template
be_group_519,519,,"Réductions de valeur actées (-)",l10n_be.l10nbe_chart_template
be_group_5190,5190,,"Actions et parts",l10n_be.l10nbe_chart_template
be_group_5191,5191,,"Placements de trésorerie autres que placements à revenu fixe",l10n_be.l10nbe_chart_template
be_group_52,52,,"Titres à revenu fixe",l10n_be.l10nbe_chart_template
be_group_520,520,,"Valeur d'acquisition",l10n_be.l10nbe_chart_template
be_group_529,529,,"Réductions de valeur actées (–)",l10n_be.l10nbe_chart_template
be_group_53,53,,"Dépôts à terme",l10n_be.l10nbe_chart_template
be_group_530,530,,"De plus d'un an",l10n_be.l10nbe_chart_template
be_group_531,531,,"De plus d'un mois et à un an au plus",l10n_be.l10nbe_chart_template
be_group_532,532,,"D'un mois au plus",l10n_be.l10nbe_chart_template
be_group_539,539,,"Réductions de valeur actées (–)",l10n_be.l10nbe_chart_template
be_group_54,54,,"Valeurs échues à l'encaissement",l10n_be.l10nbe_chart_template
be_group_55,55,,"Etablissements de crédit",l10n_be.l10nbe_chart_template
be_group_550,550,559,"Comptes ouverts auprès des divers établissements, à subdiviser en :",l10n_be.l10nbe_chart_template
be_group_56,56,,"Office des chèques postaux",l10n_be.l10nbe_chart_template
be_group_560,560,,"Compte courant",l10n_be.l10nbe_chart_template
be_group_561,561,,"Chèques émis (–)",l10n_be.l10nbe_chart_template
be_group_57,57,,"Caisses",l10n_be.l10nbe_chart_template
be_group_570,570,577,"Caisses-espèces",l10n_be.l10nbe_chart_template
be_group_578,578,,"Caisses-timbres",l10n_be.l10nbe_chart_template
be_group_58,58,,"Virements internes",l10n_be.l10nbe_chart_template
be_group_6,6,,"Charges",l10n_be.l10nbe_chart_template
be_group_60,60,,"Approvisionnements et marchandises",l10n_be.l10nbe_chart_template
be_group_600,600,,"Achats de matières premières",l10n_be.l10nbe_chart_template
be_group_601,601,,"Achats de fournitures",l10n_be.l10nbe_chart_template
be_group_602,602,,"Achats de services, travaux et études",l10n_be.l10nbe_chart_template
be_group_603,603,,"Sous-traitances générales",l10n_be.l10nbe_chart_template
be_group_604,604,,"Achats de marchandises",l10n_be.l10nbe_chart_template
be_group_605,605,,"Achats d'immeubles destinés à la vente",l10n_be.l10nbe_chart_template
be_group_608,608,,"Remises, ristournes et rabais obtenus (–)",l10n_be.l10nbe_chart_template
be_group_609,609,,"Variations des stocks",l10n_be.l10nbe_chart_template
be_group_61,61,,"Services et biens divers",l10n_be.l10nbe_chart_template
be_group_617,617,,"Personnel intérimaire et personnes mises à la disposition de l'entreprise",l10n_be.l10nbe_chart_template
be_group_618,618,,"Rémunérations, primes pour assurances extralégales, pensions de retraite et de survie des administrateurs, gérants et associés actifs qui ne sont pas attribuées en vertu d'un contrat de travail",l10n_be.l10nbe_chart_template
be_group_62,62,,"Rémunérations, charges sociales et pensions",l10n_be.l10nbe_chart_template
be_group_620,620,,"Rémunérations et avantages sociaux directs",l10n_be.l10nbe_chart_template
be_group_621,621,,"Cotisations patronales d'assurances sociales",l10n_be.l10nbe_chart_template
be_group_622,622,,"Primes patronales pour assurances extra-légales",l10n_be.l10nbe_chart_template
be_group_623,623,,"Autres frais de personnel",l10n_be.l10nbe_chart_template
be_group_624,624,,"Pensions de retraite et de survie",l10n_be.l10nbe_chart_template
be_group_63,63,,"Amortissements, réductions de valeur et provisions pour risques et charges",l10n_be.l10nbe_chart_template
be_group_630,630,,"Dotations aux amortissements et aux réductions de valeur sur immobilisations",l10n_be.l10nbe_chart_template
be_group_631,631,,"Réductions de valeur sur stocks",l10n_be.l10nbe_chart_template
be_group_632,632,,"Réductions de valeur sur commandes en cours",l10n_be.l10nbe_chart_template
be_group_633,633,,"Réductions de valeur sur créances commerciales à plus d'un an",l10n_be.l10nbe_chart_template
be_group_634,634,,"Réductions de valeur sur créances commerciales à un an au plus",l10n_be.l10nbe_chart_template
be_group_635,635,,"Provisions pour pensions et obligations similaires",l10n_be.l10nbe_chart_template
be_group_636,636,,"Provisions pour grosses réparations et gros entretien",l10n_be.l10nbe_chart_template
be_group_637,637,,"Provisions pour obligations environnementales",l10n_be.l10nbe_chart_template
be_group_638,638,,"Provisions pour autres risques et charges",l10n_be.l10nbe_chart_template
be_group_64,64,,"Autres charges d'exploitation",l10n_be.l10nbe_chart_template
be_group_640,640,,"Charges fiscales d'exploitation",l10n_be.l10nbe_chart_template
be_group_641,641,,"Moins-values sur réalisations courantes d'immobilisations corporelles",l10n_be.l10nbe_chart_template
be_group_642,642,,"Moins-values sur réalisations de créances commerciales",l10n_be.l10nbe_chart_template
be_group_643,643,648,"Charges d'exploitation diverses",l10n_be.l10nbe_chart_template
be_group_649,649,,"Charges d'exploitation portées à l'actif au titre de frais de restructuration (–)",l10n_be.l10nbe_chart_template
be_group_65,65,,"Charges financières",l10n_be.l10nbe_chart_template
be_group_650,650,,"Charges des dettes",l10n_be.l10nbe_chart_template
be_group_651,651,,"Réductions de valeur sur actifs circulants",l10n_be.l10nbe_chart_template
be_group_652,652,,"Moins-values sur réalisation d'actifs circulants",l10n_be.l10nbe_chart_template
be_group_653,653,,"Charges d'escompte de créances",l10n_be.l10nbe_chart_template
be_group_654,654,,"Différences de change",l10n_be.l10nbe_chart_template
be_group_655,655,,"Ecarts de conversion des devises",l10n_be.l10nbe_chart_template
be_group_656,656,,"Provisions à caractère financier",l10n_be.l10nbe_chart_template
be_group_657,657,658,"Charges financières diverses",l10n_be.l10nbe_chart_template
be_group_659,659,,"Charges financières portées à l'actif au titre de frais de restructuration",l10n_be.l10nbe_chart_template
be_group_66,66,,"Charges d'exploitation ou financières non récurrentes",l10n_be.l10nbe_chart_template
be_group_660,660,,"Amortissements et réductions de valeur non récurrents (dotations)",l10n_be.l10nbe_chart_template
be_group_661,661,,"Réductions de valeur sur immobilisations financières (dotations)",l10n_be.l10nbe_chart_template
be_group_662,662,,"Provisions pour risques et charges non récurrents",l10n_be.l10nbe_chart_template
be_group_663,663,,"Moins-values sur réalisation d'actifs immobilisés",l10n_be.l10nbe_chart_template
be_group_664,664,667,"Autres charges d'exploitation non récurrentes",l10n_be.l10nbe_chart_template
be_group_668,668,,"Autres charges financières non récurrentes",l10n_be.l10nbe_chart_template
be_group_669,669,,"Charges portées à l'actif au titre de frais de restructuration (-)",l10n_be.l10nbe_chart_template
be_group_67,67,,"Impôts sur le résultat",l10n_be.l10nbe_chart_template
be_group_670,670,,"Impôts belges sur le résultat de l'exercice",l10n_be.l10nbe_chart_template
be_group_671,671,,"Impôts belges sur le résultat d'exercices antérieurs",l10n_be.l10nbe_chart_template
be_group_672,672,,"Impôts étrangers sur le résultat de l'exercice",l10n_be.l10nbe_chart_template
be_group_673,673,,"Impôts étrangers sur le résultat d'exercices antérieurs",l10n_be.l10nbe_chart_template
be_group_68,68,,"Transferts aux impôts différés et aux réserves immunisées",l10n_be.l10nbe_chart_template
be_group_680,680,,"Transferts aux impôts différés",l10n_be.l10nbe_chart_template
be_group_689,689,,"Transferts aux réserves immunisées",l10n_be.l10nbe_chart_template
be_group_69,69,,"Affectations et prélèvements",l10n_be.l10nbe_chart_template
be_group_690,690,,"Perte reportée de l'exercice précédent",l10n_be.l10nbe_chart_template
be_group_691,691,,"Affectations au capital et à la prime d'émission",l10n_be.l10nbe_chart_template
be_group_692,692,,"Dotation aux réserves",l10n_be.l10nbe_chart_template
be_group_693,693,,"Bénéfice à reporter",l10n_be.l10nbe_chart_template
be_group_694,694,,"Rémunération du capital",l10n_be.l10nbe_chart_template
be_group_695,695,,"Administrateurs ou gérants",l10n_be.l10nbe_chart_template
be_group_696,696,,"Employés",l10n_be.l10nbe_chart_template
be_group_697,697,,"Autres applications",l10n_be.l10nbe_chart_template
be_group_7,7,,"Produits",l10n_be.l10nbe_chart_template
be_group_70,70,,"Chiffre d'affaires",l10n_be.l10nbe_chart_template
be_group_700,700,707,"Ventes et prestations de services",l10n_be.l10nbe_chart_template
be_group_708,708,,"Remises, ristournes et rabais accordés (–)",l10n_be.l10nbe_chart_template
be_group_71,71,,"Variation des stocks et des commandes en cours d'exécution",l10n_be.l10nbe_chart_template
be_group_712,712,,"Des en-cours de fabrication",l10n_be.l10nbe_chart_template
be_group_713,713,,"Des produits finis",l10n_be.l10nbe_chart_template
be_group_715,715,,"Des immeubles construits destinés à la vente",l10n_be.l10nbe_chart_template
be_group_717,717,,"Des commandes en cours d'exécution",l10n_be.l10nbe_chart_template
be_group_72,72,,"Production immobilisée",l10n_be.l10nbe_chart_template
be_group_74,74,,"Autres produits d'exploitation",l10n_be.l10nbe_chart_template
be_group_740,740,,"Subsides d'exploitation et montants compensatoires",l10n_be.l10nbe_chart_template
be_group_741,741,,"Plus-values sur réalisations courantes d'immobilisations corporelles",l10n_be.l10nbe_chart_template
be_group_742,742,,"Plus-values sur réalisation de créances commerciales",l10n_be.l10nbe_chart_template
be_group_743,743,749,"Produits d'exploitation divers",l10n_be.l10nbe_chart_template
be_group_75,75,,"Produits financiers",l10n_be.l10nbe_chart_template
be_group_750,750,,"Produits des immobilisations financières",l10n_be.l10nbe_chart_template
be_group_751,751,,"Produits des actifs circulants",l10n_be.l10nbe_chart_template
be_group_752,752,,"Plus-values sur réalisation d'actifs circulants",l10n_be.l10nbe_chart_template
be_group_753,753,,"Subsides en capital et en intérêts",l10n_be.l10nbe_chart_template
be_group_754,754,,"Différences de change",l10n_be.l10nbe_chart_template
be_group_755,755,,"Écart de conversion des devises",l10n_be.l10nbe_chart_template
be_group_756,756,759,"Produits financiers divers",l10n_be.l10nbe_chart_template
be_group_76,76,,"Produits d'exploitation ou financiers non récurrents",l10n_be.l10nbe_chart_template
be_group_760,760,,"Reprises d'amortissements et de réductions de valeur",l10n_be.l10nbe_chart_template
be_group_761,761,,"Reprises de réductions de valeur sur immobilisations financières",l10n_be.l10nbe_chart_template
be_group_762,762,,"Reprises de provisions pour risques et charges non récurrents",l10n_be.l10nbe_chart_template
be_group_763,763,,"Plus-values sur réalisation d'actifs immobilisés",l10n_be.l10nbe_chart_template
be_group_764,764,768,"Autres produits d'exploitation non récurrents",l10n_be.l10nbe_chart_template
be_group_769,769,,"Autres produits financiers non récurrents",l10n_be.l10nbe_chart_template
be_group_77,77,,"Régularisations d'impôts et reprises de provisions fiscales",l10n_be.l10nbe_chart_template
be_group_771,771,,"Impôts belges sur le résultat",l10n_be.l10nbe_chart_template
be_group_773,773,,"Impôts étrangers sur le résultat",l10n_be.l10nbe_chart_template
be_group_78,78,,"Prélèvements sur les réserves immunisées et les impôts différés",l10n_be.l10nbe_chart_template
be_group_780,780,,"Prélèvements sur les impôts différés",l10n_be.l10nbe_chart_template
be_group_789,789,,"Prélèvements sur les réserves immunisées",l10n_be.l10nbe_chart_template
be_group_79,79,,"Affectations et prélèvements",l10n_be.l10nbe_chart_template
be_group_790,790,,"Bénéfice reporté de l'exercice précédent",l10n_be.l10nbe_chart_template
be_group_791,791,,"Prélèvement sur le capital et les primes d'émission",l10n_be.l10nbe_chart_template
be_group_792,792,,"Prélèvement sur les réserves",l10n_be.l10nbe_chart_template
be_group_793,793,,"Perte à reporter",l10n_be.l10nbe_chart_template
be_group_794,794,,"Intervention d'associés (ou du propriétaire) dans la perte",l10n_be.l10nbe_chart_template
be_group_0,0,,"Droits et engagements hors bilan",l10n_be.l10nbe_chart_template
be_group_00,00,,"Garanties constituées par des tiers pour compte de l'entreprise",l10n_be.l10nbe_chart_template
be_group_000,000,,"Créanciers de l'entreprise, bénéficiaires de garanties de tiers",l10n_be.l10nbe_chart_template
be_group_001,001,,"Tiers constituants de garanties pour compte de l'entreprise",l10n_be.l10nbe_chart_template
be_group_01,01,,"Garanties personnelles constituées pour compte de tiers",l10n_be.l10nbe_chart_template
be_group_010,010,,"Débiteurs pour engagements sur effets en circulation",l10n_be.l10nbe_chart_template
be_group_011,011,,"Créanciers d'engagements sur effets en circulation",l10n_be.l10nbe_chart_template
be_group_012,012,,"Débiteurs pour autres garanties personnelles",l10n_be.l10nbe_chart_template
be_group_013,013,,"Créanciers d'autres garanties personnelles",l10n_be.l10nbe_chart_template
be_group_02,02,,"Garanties réelles constituées sur avoirs propres",l10n_be.l10nbe_chart_template
be_group_020,020,,"Créanciers de l'entreprise, bénéficiaires de garanties réelles",l10n_be.l10nbe_chart_template
be_group_021,021,,"Garanties réelles constituées pour compte propre",l10n_be.l10nbe_chart_template
be_group_022,022,,"Créanciers de tiers, bénéficiaires de garanties réelles",l10n_be.l10nbe_chart_template
be_group_023,023,,"Garanties réelles constituées pour compte de tiers",l10n_be.l10nbe_chart_template
be_group_03,03,,"Garanties reçues",l10n_be.l10nbe_chart_template
be_group_030,030,,"Dépôts statutaires",l10n_be.l10nbe_chart_template
be_group_031,031,,"Déposants statutaires",l10n_be.l10nbe_chart_template
be_group_032,032,,"Garanties reçues",l10n_be.l10nbe_chart_template
be_group_033,033,,"Constituants de garanties",l10n_be.l10nbe_chart_template
be_group_04,04,,"Biens et valeurs détenus par des tiers en leur nom mais aux risques et profits de l'entreprise",l10n_be.l10nbe_chart_template
be_group_040,040,,"Tiers, détenteurs en leur nom mais aux risques et profits de l'entreprise de biens et de valeurs",l10n_be.l10nbe_chart_template
be_group_041,041,,"Biens et valeurs détenus par des tiers en leur nom mais aux risques et profits de l'entreprise",l10n_be.l10nbe_chart_template
be_group_05,05,,"Engagements d'acquisition et de cession d'immobilisations",l10n_be.l10nbe_chart_template
be_group_050,050,,"Engagements d'acquisition",l10n_be.l10nbe_chart_template
be_group_051,051,,"Créanciers d'engagements d'acquisition",l10n_be.l10nbe_chart_template
be_group_052,052,,"Débiteurs pour engagements de cession",l10n_be.l10nbe_chart_template
be_group_053,053,,"Engagements de cession",l10n_be.l10nbe_chart_template
be_group_06,06,,"Marchés à terme",l10n_be.l10nbe_chart_template
be_group_060,060,,"Marchandises achetées à terme - à recevoir",l10n_be.l10nbe_chart_template
be_group_061,061,,"Créanciers pour marchandises achetées à terme",l10n_be.l10nbe_chart_template
be_group_062,062,,"Débiteurs pour marchandises vendues à terme",l10n_be.l10nbe_chart_template
be_group_063,063,,"Marchandises vendues à terme - à livrer",l10n_be.l10nbe_chart_template
be_group_064,064,,"Devises achetées à terme - à recevoir",l10n_be.l10nbe_chart_template
be_group_065,065,,"Créanciers pour devises achetées à terme",l10n_be.l10nbe_chart_template
be_group_066,066,,"Débiteurs pour devises vendues à terme",l10n_be.l10nbe_chart_template
be_group_067,067,,"Devises vendues à terme - à livrer",l10n_be.l10nbe_chart_template
be_group_07,07,,"Biens et valeurs de tiers détenus par l'entreprise",l10n_be.l10nbe_chart_template
be_group_070,070,,"Droits d'usage à long terme",l10n_be.l10nbe_chart_template
be_group_071,071,,"Créanciers de loyers et redevances",l10n_be.l10nbe_chart_template
be_group_072,072,,"Biens et valeurs de tiers reçus en dépôt, en consignation ou à façon",l10n_be.l10nbe_chart_template
be_group_073,073,,"Commettants et déposants de biens et de valeurs",l10n_be.l10nbe_chart_template
be_group_074,074,,"Biens et valeurs détenus pour compte ou aux risques et profits de tiers",l10n_be.l10nbe_chart_template
be_group_075,075,,"Créanciers de biens et valeurs détenus pour compte de tiers ou à leurs risques et profits",l10n_be.l10nbe_chart_template
be_group_09,09,,"Droits et engagements divers",l10n_be.l10nbe_chart_template

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_be.l10nbe_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- Chart template -->
        <record id="l10nbe_chart_template" model="account.chart.template">
            <field name="name">Belgian PCMN</field>
            <field name="bank_account_code_prefix">550</field>
            <field name="cash_account_code_prefix">570</field>
            <field name="transfer_account_code_prefix">580</field>
            <field name="currency_id" ref="base.EUR"/>
            <field name="spoken_languages" eval="'nl_BE;nl_NL;fr_FR;fr_BE;de_DE'"/>
        </record>
</odoo>

```

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<odoo>
    <data noupdate="1">

        <!-- Account Tax Group -->
        <record id="tax_group_tva_21" model="account.tax.group">
            <field name="name">TVA 21%</field>
        </record>

        <record id="tax_group_tva_12" model="account.tax.group">
            <field name="name">TVA 12%</field>
        </record>

        <record id="tax_group_tva_6" model="account.tax.group">
            <field name="name">TVA 6%</field>
        </record>

        <record id="tax_group_tva_0" model="account.tax.group">
            <field name="name">TVA 0%</field>
        </record>

    </data>
</odoo>

```

## File: data\account_fiscal_position_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- account.fiscal.position.tax.template -->
        <record id="afpttn_intracom_1" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-00-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-EU-S"/>
        </record>
        <record id="afpttn_intracom_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-00-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-EU-L"/>
        </record>
        <record id="afpttn_intracom_3" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-06-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-EU-S"/>
        </record>
        <record id="afpttn_intracom_4" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-06-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-EU-L"/>
        </record>
        <record id="afpttn_intracom_5" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-12-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-EU-S"/>
        </record>
        <record id="afpttn_intracom_6" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-12-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-EU-L"/>
        </record>
        <record id="afpttn_intracom_7" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-21-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-EU-S"/>
        </record>
        <record id="afpttn_intracom_8" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-21-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-EU-L"/>
        </record>
        <record id="afpttn_intracom_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-00"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-00-EU"/>
        </record>
        <record id="afpttn_intracom_10" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-06"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-06-EU"/>
        </record>
        <record id="afpttn_intracom_11" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-12"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-12-EU"/>
        </record>
        <record id="afpttn_intracom_12" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-21"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-21-EU"/>
        </record>
        <record id="afpttn_intracom_13" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-00-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-00-EU-S"/>
        </record>
        <record id="afpttn_intracom_14" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-00-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-00-EU-G"/>
        </record>
        <record id="afpttn_intracom_15" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-06-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-06-EU-S"/>
        </record>
        <record id="afpttn_intracom_16" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-06-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-06-EU-G"/>
        </record>
        <record id="afpttn_intracom_17" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-12-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-12-EU-S"/>
        </record>
        <record id="afpttn_intracom_18" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-12-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-12-EU-G"/>
        </record>
        <record id="afpttn_intracom_19" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-21-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-21-EU-S"/>
        </record>
        <record id="afpttn_intracom_20" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-21-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-21-EU-G"/>
        </record>
        <record id="afpttn_intracom_21" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-00"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-00-EU"/>
        </record>
        <record id="afpttn_intracom_22" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-06"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-06-EU"/>
        </record>
        <record id="afpttn_intracom_23" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-12"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-12-EU"/>
        </record>
        <record id="afpttn_intracom_24" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-21"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-21-EU"/>
        </record>
        <record id="afpttn_extracom_1" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-00-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-ROW"/>
        </record>
        <record id="afpttn_extracom_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-00-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-ROW"/>
        </record>
        <record id="afpttn_extracom_3" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-06-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-ROW"/>
        </record>
        <record id="afpttn_extracom_4" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-06-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-ROW"/>
        </record>
        <record id="afpttn_extracom_5" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-12-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-ROW"/>
        </record>
        <record id="afpttn_extracom_6" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-12-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-ROW"/>
        </record>
        <record id="afpttn_extracom_7" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-21-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-ROW"/>
        </record>
        <record id="afpttn_extracom_8" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-21-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-ROW"/>
        </record>
        <record id="afpttn_extracom_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-06"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-06-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_10" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-12"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-12-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_11" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-21"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-21-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_12" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-06-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-06-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_13" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-06-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-06-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_14" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-12-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-12-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_15" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-12-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-12-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_16" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-21-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-21-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_17" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-21-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-21-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_18" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-06"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-06-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_19" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-12"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-12-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_20" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-21"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-21-ROW-CC"/>
        </record>
        <record id="afpttn_cocontractant_1" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-00-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-CC"/>
        </record>
        <record id="afpttn_cocontractant_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-00-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-CC"/>
        </record>
        <record id="afpttn_cocontractant_3" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-06-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-CC"/>
        </record>
        <record id="afpttn_cocontractant_4" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-06-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-CC"/>
        </record>
        <record id="afpttn_cocontractant_5" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-12-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-CC"/>
        </record>
        <record id="afpttn_cocontractant_6" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-12-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-CC"/>
        </record>
        <record id="afpttn_cocontractant_7" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-21-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-CC"/>
        </record>
        <record id="afpttn_cocontractant_8" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-21-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-CC"/>
        </record>
        <record id="afpttn_cocontractant_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-06"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-06-CC"/>
        </record>
        <record id="afpttn_cocontractant_10" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-12"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-12-CC"/>
        </record>
        <record id="afpttn_cocontractant_11" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-21"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-21-CC"/>
        </record>
        <record id="afpttn_cocontractant_12" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-06-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-06-CC"/>
        </record>
        <record id="afpttn_cocontractant_13" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-06-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-06-CC"/>
        </record>
        <record id="afpttn_cocontractant_14" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-12-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-12-CC"/>
        </record>
        <record id="afpttn_cocontractant_15" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-12-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-12-CC"/>
        </record>
        <record id="afpttn_cocontractant_16" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-21-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-21-CC"/>
        </record>
        <record id="afpttn_cocontractant_17" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-21-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-21-CC"/>
        </record>
        <record id="afpttn_cocontractant_18" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-06"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-06-CC"/>
        </record>
        <record id="afpttn_cocontractant_19" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-12"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-12-CC"/>
        </record>
        <record id="afpttn_cocontractant_20" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-21"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-21-CC"/>
        </record>
</odoo>

```

## File: data\account_pcmn_belgium_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="l10nbe_chart_template" model="account.chart.template">
            <field name="name">Belgian PCMN</field>
            <field name="code_digits">6</field>
            <field name="property_account_receivable_id" ref="a400"/>
            <field name="property_account_payable_id" ref="a440"/>
            <field name="property_account_expense_categ_id" ref="a600"/>
            <field name="property_account_income_categ_id" ref="a7000"/>
            <field name="expense_currency_exchange_account_id" ref="a654"/>
            <field name="income_currency_exchange_account_id" ref="a754"/>
            <field name="property_tax_payable_account_id" ref="a4512"/>
            <field name="property_tax_receivable_account_id" ref="a4112"/>
            <field name="default_pos_receivable_account_id" ref="a4001" />
            <field name="account_journal_suspense_account_id" ref="a499"/>
        </record>
</odoo>

```

## File: data\account_reconcile_model_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="escompte_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="l10nbe_chart_template"/>
        <field name="name">Escompte</field>
    </record>
    <record id="escompte_line_template" model="account.reconcile.model.line.template">
        <field name="model_id" ref="l10n_be.escompte_template"/>
        <field name="account_id" ref="a653"/>
        <field name="amount_type">percentage</field>
        <field name="amount_string">100</field>
        <field name="label">Escompte accordé</field>
    </record>
    <record id="frais_bancaires_htva_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="l10nbe_chart_template"/>
        <field name="name">Frais bancaires HTVA</field>
    </record>
    <record id="frais_bancaires_htva_line_template" model="account.reconcile.model.line.template">
        <field name="model_id" ref="l10n_be.frais_bancaires_htva_template"/>
        <field name="account_id" ref="a6560" />
        <field name="amount_type">percentage</field>
        <field name="amount_string">100</field>
        <field name="label">Frais bancaires HTVA</field>
    </record>
    <record id="frais_bancaires_tva21_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="l10nbe_chart_template"/>
        <field name="name">Frais bancaires TVA21</field>
    </record>
    <record id="frais_bancaires_tva21_line_template" model="account.reconcile.model.line.template">
        <field name="model_id" ref="l10n_be.frais_bancaires_tva21_template"/>
        <field name="account_id" ref="a6560"/>
        <field name="amount_type">percentage</field>
        <field name="tax_ids" eval="[(6, 0, [ref('l10n_be.attn_TVA-21-inclus-dans-prix')])]"/>
        <field name="amount_string">100</field>
        <field name="label">Frais bancaires TVA21</field>
    </record>
    <record id="virements_internes_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="l10nbe_chart_template"/>
        <field name="name">Virements internes</field>
        <field name="to_check" eval="False"/>
    </record>
    <record id="virements_internes_line_template" model="account.reconcile.model.line.template">
        <field name="model_id" ref="l10n_be.virements_internes_template"/>
        <field name="account_id" search="[('code', '=like', obj().env.ref('l10n_be.l10nbe_chart_template').transfer_account_code_prefix + '%'), ('chart_template_id', '=', obj().env.ref('l10n_be.l10nbe_chart_template').id)]"/>
        <field name="amount_type">percentage</field>
        <field name="amount_string">100</field>
        <field name="label">Virements internes</field>
    </record>
    <record id="compte_attente_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="l10nbe_chart_template"/>
        <field name="name">Compte Attente</field>
        <field name="to_check" eval="True"/>
    </record>
    <record id="compte_attente_line_template" model="account.reconcile.model.line.template">
        <field name="model_id" ref="l10n_be.compte_attente_template"/>
        <field name="account_id" ref="a499"/>
        <field name="amount_type">percentage</field>
        <field name="amount_string">100</field>
        <field name="label"></field>
    </record>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report_vat" model="account.tax.report">
        <field name="name">VAT Report</field>
        <field name="country_id" ref="base.be"/>
    </record>

    <record id="tax_report_title_operations" model="account.tax.report.line">
        <field name="name">Opérations</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_title_operations_sortie" model="account.tax.report.line">
        <field name="name">II A la sortie</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_line_00" model="account.tax.report.line">
        <field name="name">00 - Opérations soumises à un régime particulier</field>
        <field name="code">c00</field>
        <field name="tag_name">00</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_line_01" model="account.tax.report.line">
        <field name="name">01 - Opérations avec TVA à 6%</field>
        <field name="code">c01</field>
        <field name="tag_name">01</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_line_02" model="account.tax.report.line">
        <field name="name">02 - Opérations avec TVA à 12%</field>
        <field name="code">c02</field>
        <field name="tag_name">02</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        <field name="sequence">3</field>
    </record>

    <record id="tax_report_line_03" model="account.tax.report.line">
        <field name="name">03 - Opérations avec TVA à 21%</field>
        <field name="code">c03</field>
        <field name="tag_name">03</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        <field name="sequence">4</field>
    </record>

    <record id="tax_report_line_44" model="account.tax.report.line">
        <field name="name">44 - Services intra-communautaires</field>
        <field name="code">c44</field>
        <field name="tag_name">44</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        <field name="sequence">5</field>
    </record>

    <record id="tax_report_line_45" model="account.tax.report.line">
        <field name="name">45 - Opérations avec TVA due par le cocontractant</field>
        <field name="code">c45</field>
        <field name="tag_name">45</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        <field name="sequence">6</field>
    </record>

    <record id="tax_report_title_operations_sortie_46" model="account.tax.report.line">
        <field name="name">46 - Livraisons intra-communautaires exemptées</field>
        <field name="code">c46</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        <field name="sequence">7</field>
    </record>

    <record id="tax_report_line_46L" model="account.tax.report.line">
        <field name="name">46L - Livraisons biens intra-communautaires exemptées</field>
        <field name="tag_name">46L</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_sortie_46"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_line_46T" model="account.tax.report.line">
        <field name="name">46T - Livraisons biens intra-communautaire exemptées</field>
        <field name="tag_name">46T</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_sortie_46"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_line_47" model="account.tax.report.line">
        <field name="name">47 - Autres opérations exemptées</field>
        <field name="code">c47</field>
        <field name="tag_name">47</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        <field name="sequence">8</field>
    </record>

    <record id="tax_report_title_operations_sortie_48" model="account.tax.report.line">
        <field name="name">48 - Notes de crédit aux opérations grilles [44] et [46]</field>
        <field name="code">c48</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        <field name="sequence">9</field>
    </record>

    <record id="tax_report_line_48s44" model="account.tax.report.line">
        <field name="name">48s44 - Notes de crédit aux opérations grilles [44]</field>
        <field name="tag_name">48s44</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_sortie_48"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_line_48s46L" model="account.tax.report.line">
        <field name="name">48s46L - Notes de crédit aux opérations grilles [46L]</field>
        <field name="tag_name">48s46L</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_sortie_48"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_line_48s46T" model="account.tax.report.line">
        <field name="name">48s46T - Notes de crédit aux opérations grilles [46T]</field>
        <field name="tag_name">48s46T</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_sortie_48"/>
        <field name="sequence">3</field>
    </record>

    <record id="tax_report_line_49" model="account.tax.report.line">
        <field name="name">49 - Notes de crédit aux opérations du point II</field>
        <field name="code">c49</field>
        <field name="tag_name">49</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        <field name="sequence">10</field>
    </record>

    <record id="tax_report_title_operations_entree" model="account.tax.report.line">
        <field name="name">III A l'entrée</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_line_81" model="account.tax.report.line">
        <field name="name">81 - Marchandises, matières premières et auxiliaires</field>
        <field name="code">c81</field>
        <field name="tag_name">81</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_entree"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_line_82" model="account.tax.report.line">
        <field name="name">82 - Services et biens divers</field>
        <field name="code">c82</field>
        <field name="tag_name">82</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_entree"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_line_83" model="account.tax.report.line">
        <field name="name">83 - Biens d'investissement</field>
        <field name="code">c83</field>
        <field name="tag_name">83</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_entree"/>
        <field name="sequence">3</field>
    </record>

    <record id="tax_report_line_84" model="account.tax.report.line">
        <field name="name">84 - Notes de crédits sur opérations case [86] et [88]</field>
        <field name="code">c84</field>
        <field name="tag_name">84</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_entree"/>
        <field name="sequence">4</field>
    </record>

    <record id="tax_report_line_85" model="account.tax.report.line">
        <field name="name">85 - Notes de crédits autres opérations</field>
        <field name="code">c85</field>
        <field name="tag_name">85</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_entree"/>
        <field name="sequence">5</field>
    </record>

    <record id="tax_report_line_86" model="account.tax.report.line">
        <field name="name">86 - Acquisition intra-communautaires et ventes ABC</field>
        <field name="code">c86</field>
        <field name="tag_name">86</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_entree"/>
        <field name="sequence">6</field>
    </record>

    <record id="tax_report_line_87" model="account.tax.report.line">
        <field name="name">87 - Autres opérations</field>
        <field name="code">c87</field>
        <field name="tag_name">87</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_entree"/>
        <field name="sequence">7</field>
    </record>

    <record id="tax_report_line_88" model="account.tax.report.line">
        <field name="name">88 - Acquisition services intra-communautaires</field>
        <field name="code">c88</field>
        <field name="tag_name">88</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_entree"/>
        <field name="sequence">8</field>
    </record>

    <record id="tax_report_title_taxes" model="account.tax.report.line">
        <field name="name">Taxes</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_title_taxes_dues" model="account.tax.report.line">
        <field name="name">IV Dues</field>
        <field name="code">tax_be_iv</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_taxes"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_line_54" model="account.tax.report.line">
        <field name="name">54 - TVA sur opérations des grilles [01], [02], [03]</field>
        <field name="code">c54</field>
        <field name="tag_name">54</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_taxes_dues"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_line_55" model="account.tax.report.line">
        <field name="name">55 - TVA sur opérations des grilles [86] et [88]</field>
        <field name="code">c55</field>
        <field name="tag_name">55</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_taxes_dues"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_line_56" model="account.tax.report.line">
        <field name="name">56 - TVA sur opérations de la grille [87]</field>
        <field name="code">c56</field>
        <field name="tag_name">56</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_taxes_dues"/>
        <field name="sequence">3</field>
    </record>

    <record id="tax_report_line_57" model="account.tax.report.line">
        <field name="name">57 - TVA relatives aux importations</field>
        <field name="code">c57</field>
        <field name="tag_name">57</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_taxes_dues"/>
        <field name="sequence">4</field>
    </record>

    <record id="tax_report_line_61" model="account.tax.report.line">
        <field name="name">61 - Diverses régularisations en faveur de l'Etat</field>
        <field name="code">c61</field>
        <field name="tag_name">61</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_taxes_dues"/>
        <field name="sequence">5</field>
    </record>

    <record id="tax_report_line_63" model="account.tax.report.line">
        <field name="name">63 - TVA à reverser sur notes de crédit recues</field>
        <field name="code">c63</field>
        <field name="tag_name">63</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_taxes_dues"/>
        <field name="sequence">6</field>
    </record>

    <record id="tax_report_title_taxes_deductibles" model="account.tax.report.line">
        <field name="name">V Déductibles</field>
        <field name="code">tax_be_v</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_taxes"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_line_59" model="account.tax.report.line">
        <field name="name">59 - TVA déductible</field>
        <field name="code">c59</field>
        <field name="tag_name">59</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_taxes_deductibles"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_line_62" model="account.tax.report.line">
        <field name="name">62 - Diverses régularisations en faveur du déclarant</field>
        <field name="code">c62</field>
        <field name="tag_name">62</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_taxes_deductibles"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_line_64" model="account.tax.report.line">
        <field name="name">64 - TVA à récupérer sur notes de crédit delivrées</field>
        <field name="code">c64</field>
        <field name="tag_name">64</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_taxes_deductibles"/>
        <field name="sequence">3</field>
    </record>

    <record id="tax_report_title_taxes_soldes" model="account.tax.report.line">
        <field name="name">VI Soldes</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_taxes"/>
        <field name="sequence">3</field>
    </record>

    <record id="tax_report_line_71" model="account.tax.report.line">
        <field name="name">71 - Taxes dues à l'état</field>
        <field name="formula">tax_be_iv&gt;tax_be_v and tax_be_iv-tax_be_v or 0</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_taxes_soldes"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_line_72" model="account.tax.report.line">
        <field name="name">72 - Somme due par l'état</field>
        <field name="formula">tax_be_iv&lt;tax_be_v and tax_be_v-tax_be_iv or 0</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_taxes_soldes"/>
        <field name="sequence">2</field>
    </record>

</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record id="attn_VAT-OUT-21-L" model="account.tax.template">
            <field name="sequence">10</field>
            <field name="description">TVA 21%</field>
            <field name="name">21%</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_03')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451'),
                    'plus_report_line_ids': [ref('tax_report_line_54')],
                }),

            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451'),
                    'plus_report_line_ids': [ref('tax_report_line_64')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-21-S" model="account.tax.template">
            <field name="sequence">11</field>
            <field name="description">TVA 21%</field>
            <field name="name">21% S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_03')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451'),
                    'plus_report_line_ids': [ref('tax_report_line_54')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451'),
                    'plus_report_line_ids': [ref('tax_report_line_64')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-12-S" model="account.tax.template">
            <field name="sequence">20</field>
            <field name="description">TVA 12%</field>
            <field name="name">12% S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_02')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451'),
                    'plus_report_line_ids': [ref('tax_report_line_54')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451'),
                    'plus_report_line_ids': [ref('tax_report_line_64')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-12-L" model="account.tax.template">
            <field name="sequence">21</field>
            <field name="description">TVA 12%</field>
            <field name="name">12%</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_02')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451'),
                    'plus_report_line_ids': [ref('tax_report_line_54')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451'),
                    'plus_report_line_ids': [ref('tax_report_line_64')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-06-S" model="account.tax.template">
            <field name="sequence">30</field>
            <field name="description">TVA 6%</field>
            <field name="name">6% S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_01')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451'),
                    'plus_report_line_ids': [ref('tax_report_line_54')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451'),
                    'plus_report_line_ids': [ref('tax_report_line_64')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-06-L" model="account.tax.template">
            <field name="sequence">31</field>
            <field name="description">TVA 6%</field>
            <field name="name">6%</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_01')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451'),
                    'plus_report_line_ids': [ref('tax_report_line_54')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451'),
                    'plus_report_line_ids': [ref('tax_report_line_64')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-00-S" model="account.tax.template">
            <field name="sequence">40</field>
            <field name="description">TVA 0%</field>
            <field name="name">0% S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_00')],
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
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-00-L" model="account.tax.template">
            <field name="sequence">41</field>
            <field name="description">TVA 0%</field>
            <field name="name">0%</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_00')],
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
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-00-CC" model="account.tax.template">
            <field name="sequence">50</field>
            <field name="description">TVA 0% Cocont.</field>
            <field name="name">0% Cocont.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_45')],
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
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-00-EU-S" model="account.tax.template">
            <field name="sequence">60</field>
            <field name="description">TVA 0% EU</field>
            <field name="name">0% EU S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_44')],
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
                    'plus_report_line_ids': [ref('tax_report_line_48s44')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-00-EU-L" model="account.tax.template">
            <field name="sequence">61</field>
            <field name="description">TVA 0% EU</field>
            <field name="name">0% EU M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_46L')],
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
                    'plus_report_line_ids': [ref('tax_report_line_48s46L')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-00-EU-T" model="account.tax.template">
            <field name="sequence">62</field>
            <field name="description">TVA 0% EU</field>
            <field name="name">0% EU T.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_46T')],
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
                    'plus_report_line_ids': [ref('tax_report_line_48s46T')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-00-ROW" model="account.tax.template">
            <field name="sequence">70</field>
            <field name="description">TVA 0% Non EU</field>
            <field name="name">0% Non EU</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_47')],
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
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-21" model="account.tax.template">
            <field name="sequence">110</field>
            <field name="description">TVA 21%</field>
            <field name="name">21% M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-12" model="account.tax.template">
            <field name="sequence">120</field>
            <field name="description">TVA 12%</field>
            <field name="name">12% M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-06" model="account.tax.template">
            <field name="sequence">130</field>
            <field name="description">TVA 6%</field>
            <field name="name">6% M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-00" model="account.tax.template">
            <field name="sequence">140</field>
            <field name="description">TVA 0%</field>
            <field name="name">0% M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81')],
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
                    'minus_report_line_ids': [ref('tax_report_line_81')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

         <record id="attn_TVA-21-inclus-dans-prix" model="account.tax.template">
            <field name="sequence">150</field>
            <field name="description">TVA 21% TTC</field>
            <field name="name">21% S. TTC</field>
            <field name="price_include" eval="1"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-21-S" model="account.tax.template">
            <field name="sequence">210</field>
            <field name="description">TVA 21%</field>
            <field name="name">21% S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-21-G" model="account.tax.template">
            <field name="sequence">220</field>
            <field name="description">TVA 21%</field>
            <field name="name">21% Biens divers</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-12-S" model="account.tax.template">
            <field name="sequence">230</field>
            <field name="description">TVA 12%</field>
            <field name="name">12% S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-12-G" model="account.tax.template">
            <field name="sequence">240</field>
            <field name="description">TVA 12%</field>
            <field name="name">12% Biens divers</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-06-S" model="account.tax.template">
            <field name="sequence">250</field>
            <field name="description">TVA 6%</field>
            <field name="name">6% S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-06-G" model="account.tax.template">
            <field name="sequence">260</field>
            <field name="description">TVA 6%</field>
            <field name="name">6% Biens divers</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-00-S" model="account.tax.template">
            <field name="sequence">270</field>
            <field name="description">TVA 0%</field>
            <field name="name">0% S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
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
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-00-G" model="account.tax.template">
            <field name="sequence">280</field>
            <field name="description">TVA 0%</field>
            <field name="name">0% Biens divers</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
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
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-21" model="account.tax.template">
            <field name="sequence">310</field>
            <field name="description">TVA 21%</field>
            <field name="name">21% Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-12" model="account.tax.template">
            <field name="sequence">320</field>
            <field name="description">TVA 12%</field>
            <field name="name">12% Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-06" model="account.tax.template">
            <field name="sequence">330</field>
            <field name="description">TVA 6%</field>
            <field name="name">6% Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-00" model="account.tax.template">
            <field name="sequence">340</field>
            <field name="description">TVA 0%</field>
            <field name="name">0% Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83')],
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
                    'minus_report_line_ids': [ref('tax_report_line_83')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-21-CC" model="account.tax.template">
            <field name="sequence">410</field>
            <field name="description">TVA 21% Cocont.</field>
            <field name="name">21% Cocont. M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                    'minus_report_line_ids': [ref('tax_report_line_56')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-12-CC" model="account.tax.template">
            <field name="sequence">420</field>
            <field name="description">TVA 12% Cocont.</field>
            <field name="name">12% Cocont. M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="amount">12</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                    'minus_report_line_ids': [ref('tax_report_line_56')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-06-CC" model="account.tax.template">
            <field name="sequence">430</field>
            <field name="description">TVA 6% Cocont.</field>
            <field name="name">6% Cocont. M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="amount">6</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                    'minus_report_line_ids': [ref('tax_report_line_56')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-00-CC" model="account.tax.template">
            <field name="sequence">440</field>
            <field name="description">TVA 0% Cocont.</field>
            <field name="name">0% Cocont. M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
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
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-21-CC" model="account.tax.template">
            <field name="sequence">510</field>
            <field name="description">TVA 21% Cocont.</field>
            <field name="name">21% Cocont .S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                    'minus_report_line_ids': [ref('tax_report_line_56')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-12-CC" model="account.tax.template">
            <field name="sequence">520</field>
            <field name="description">TVA 12% Cocont.</field>
            <field name="name">12% Cocont. S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                    'minus_report_line_ids': [ref('tax_report_line_56')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-06-CC" model="account.tax.template">
            <field name="sequence">530</field>
            <field name="description">TVA 6% Cocont.</field>
            <field name="name">6% Cocont. S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                    'minus_report_line_ids': [ref('tax_report_line_56')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-00-CC" model="account.tax.template">
            <field name="sequence">540</field>
            <field name="description">TVA 0% Cocont.</field>
            <field name="name">0% Cocont. S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
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
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-21-CC" model="account.tax.template">
            <field name="sequence">610</field>
            <field name="description">TVA 21% Cocont.</field>
            <field name="name">21% Cocont. - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                    'minus_report_line_ids': [ref('tax_report_line_56')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-12-CC" model="account.tax.template">
            <field name="sequence">620</field>
            <field name="description">TVA 12% Cocont.</field>
            <field name="name">12% Cocont. - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                    'minus_report_line_ids': [ref('tax_report_line_56')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-06-CC" model="account.tax.template">
            <field name="sequence">630</field>
            <field name="description">TVA 6% Cocont.</field>
            <field name="name">6% Cocont. - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                    'minus_report_line_ids': [ref('tax_report_line_56')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-00-CC" model="account.tax.template">
            <field name="sequence">640</field>
            <field name="description">TVA 0% Cocont.</field>
            <field name="name">0% Cocont. - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
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
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-CAR-EXC" model="account.tax.template">
            <field name="sequence">720</field>
            <field name="description">TVA 50% Non Déductible - Frais de voiture (Prix Excl.)</field>
            <field name="name">50% Non Déductible - Frais de voiture (Prix Excl.)</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 50,
                    'repartition_type': 'tax',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 50,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 50,
                    'repartition_type': 'tax',
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 50,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-21-EU" model="account.tax.template">
            <field name="sequence">1110</field>
            <field name="description">TVA 21% EU</field>
            <field name="name">21% EU M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_86')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-12-EU" model="account.tax.template">
            <field name="sequence">1120</field>
            <field name="description">TVA 12% EU</field>
            <field name="name">12% EU M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_86')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-06-EU" model="account.tax.template">
            <field name="sequence">1130</field>
            <field name="description">TVA 6% EU</field>
            <field name="name">6% EU M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_86')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-00-EU" model="account.tax.template">
            <field name="sequence">1140</field>
            <field name="description">TVA 0% EU</field>
            <field name="name">0% EU M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0.0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_86')],
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
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-21-EU-S" model="account.tax.template">
            <field name="sequence">1210</field>
            <field name="description">TVA 21% EU</field>
            <field name="name">21% EU S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_88')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_88')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-21-EU-G" model="account.tax.template">
            <field name="sequence">1220</field>
            <field name="description">TVA 21% EU</field>
            <field name="name">21% EU - Biens divers</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_86')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-12-EU-S" model="account.tax.template">
            <field name="sequence">1230</field>
            <field name="description">TVA 12% EU</field>
            <field name="name">12% EU S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_88')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_88')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-12-EU-G" model="account.tax.template">
            <field name="sequence">1240</field>
            <field name="description">TVA 12% EU</field>
            <field name="name">12% EU - Biens divers</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_86')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-06-EU-S" model="account.tax.template">
            <field name="sequence">1250</field>
            <field name="description">TVA 6% EU</field>
            <field name="name">6% EU S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="amount">6</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_88')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_88')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-06-EU-G" model="account.tax.template">
            <field name="sequence">1260</field>
            <field name="description">TVA 6% EU</field>
            <field name="name">6% EU - Biens divers</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_86')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-00-EU-S" model="account.tax.template">
            <field name="sequence">1270</field>
            <field name="description">TVA 0% EU</field>
            <field name="name">0% EU S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0.0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_88')],
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
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_88')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-21-EU" model="account.tax.template">
            <field name="sequence">1310</field>
            <field name="description">TVA 21% EU</field>
            <field name="name">21% EU - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_86')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-00-EU-G" model="account.tax.template">
            <field name="sequence">1280</field>
            <field name="description">TVA 0% EU</field>
            <field name="name">0% EU - Biens divers</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_86')],
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
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-12-EU" model="account.tax.template">
            <field name="sequence">1320</field>
            <field name="description">TVA 12% EU</field>
            <field name="name">12% EU - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="amount">12</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_86')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-06-EU" model="account.tax.template">
            <field name="sequence">1330</field>
            <field name="description">TVA 6% EU</field>
            <field name="name">6% EU - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="amount">6</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_86')],
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-00-EU" model="account.tax.template">
            <field name="sequence">1340</field>
            <field name="description">TVA 0% EU</field>
            <field name="name">0% EU - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0.0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_86')],
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
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-21-ROW-CC" model="account.tax.template">
            <field name="sequence">2110</field>
            <field name="description">TVA 21% Non EU</field>
            <field name="name">21% Non EU M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                    'minus_report_line_ids': [ref('tax_report_line_57')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-12-ROW-CC" model="account.tax.template">
            <field name="sequence">2120</field>
            <field name="description">TVA 12% Non EU</field>
            <field name="name">12% Non EU M.</field>
            <field name="amount">12</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                    'minus_report_line_ids': [ref('tax_report_line_57')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-06-ROW-CC" model="account.tax.template">
            <field name="sequence">2130</field>
            <field name="description">TVA 6% Non EU</field>
            <field name="name">6% Non EU M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                    'minus_report_line_ids': [ref('tax_report_line_57')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-00-ROW-CC" model="account.tax.template">
            <field name="sequence">2140</field>
            <field name="description">TVA 0% Non EU</field>
            <field name="name">0% Non EU M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0.0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
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
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-21-ROW-CC" model="account.tax.template">
            <field name="sequence">2210</field>
            <field name="description">TVA 21% Non EU</field>
            <field name="name">21% Non EU S.</field>
            <field name="amount">21</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                    'minus_report_line_ids': [ref('tax_report_line_57')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-12-ROW-CC" model="account.tax.template">
            <field name="sequence">2220</field>
            <field name="description">TVA 12% Non EU</field>
            <field name="name">12% Non EU S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="amount">12</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                    'minus_report_line_ids': [ref('tax_report_line_57')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-06-ROW-CC" model="account.tax.template">
            <field name="sequence">2230</field>
            <field name="description">TVA 6% Non EU</field>
            <field name="name">6% Non EU S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">6</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                    'minus_report_line_ids': [ref('tax_report_line_57')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-00-ROW-CC" model="account.tax.template">
            <field name="sequence">2240</field>
            <field name="description">TVA 0% Non EU</field>
            <field name="name">0% Non EU S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0.0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
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
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-21-ROW-CC" model="account.tax.template">
            <field name="sequence">2310</field>
            <field name="description">TVA 21% Non EU</field>
            <field name="name">21% Non EU - Biens d'investissement</field>
            <field name="amount">21</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                    'minus_report_line_ids': [ref('tax_report_line_57')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-12-ROW-CC" model="account.tax.template">
            <field name="sequence">2320</field>
            <field name="description">TVA 12% Non EU</field>
            <field name="name">12% Non EU - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="amount">12</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                    'minus_report_line_ids': [ref('tax_report_line_57')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-06-ROW-CC" model="account.tax.template"> <!--merged group-->
            <field name="sequence">2330</field>
            <field name="description">TVA 6% Non EU</field>
            <field name="name">6% Non EU - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">6</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                    'minus_report_line_ids': [ref('tax_report_line_57')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-00-ROW-CC" model="account.tax.template">
            <field name="sequence">2340</field>
            <field name="description">TVA 0% Non EU</field>
            <field name="name">0% Non EU - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0.0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
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
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

</odoo>

```

## File: data\fiscal_templates_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Fiscal Position Templates -->

    <record id="fiscal_position_template_1" model="account.fiscal.position.template">
            <field name="sequence">1</field>
            <field name="name">Régime National</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_id" ref="base.be"/>
    </record>

    <record id="fiscal_position_template_5" model="account.fiscal.position.template">
            <field name="sequence">2</field>
            <field name="name">EU privé</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
    </record>

    <record id="fiscal_position_template_2" model="account.fiscal.position.template">
            <field name="sequence">4</field>
            <field name="name">Régime Extra-Communautaire</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="auto_apply" eval="True"/>
    </record>

    <record id="fiscal_position_template_3" model="account.fiscal.position.template">
            <field name="sequence">3</field>
            <field name="name">Régime Intra-Communautaire</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
    </record>

    <record id="fiscal_position_template_4" model="account.fiscal.position.template">
            <field name="name">Régime Cocontractant</field>
            <field name="sequence">5</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
    </record>

    <!-- Fiscal Position Account Templates -->

    <record id="fiscal_position_account_template_3" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="account_src_id" ref="l10n_be.a7000" />
            <field name="account_dest_id" ref="l10n_be.a7001" />
    </record>

    <record id="fiscal_position_account_template_4" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="account_src_id" ref="l10n_be.a7010" />
            <field name="account_dest_id" ref="l10n_be.a7011" />
    </record>

    <record id="fiscal_position_account_template_6" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="account_src_id" ref="l10n_be.a7050" />
            <field name="account_dest_id" ref="l10n_be.a7051" />
    </record>

    <record id="fiscal_position_account_template_7" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="account_src_id" ref="l10n_be.a7000" />
            <field name="account_dest_id" ref="l10n_be.a7002" />
    </record>
    <record id="fiscal_position_account_template_8" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="account_src_id" ref="l10n_be.a7010" />
            <field name="account_dest_id" ref="l10n_be.a7012" />
    </record>
    <record id="fiscal_position_account_template_10" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="account_src_id" ref="l10n_be.a7050" />
            <field name="account_dest_id" ref="l10n_be.a7052" />
    </record>
</odoo>

```

## File: data\l10n_be_sequence_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!--
    Sequences for declarantnum will be used in wizard for "Listing of VAT Customers"..in creating xml file
        -->
    <record model="ir.sequence" id="seq_declarantnum">
        <field name="name">Declarantnum</field>
        <field name="code">declarantnum</field>
        <field name="padding">5</field>
        <field name="company_id" eval="False"/>
    </record>
</odoo>

```

## File: data\menuitem_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_be_statements_menu" name="Belgium" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>
</odoo>

```

## File: i18n_extra\l10n_be.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
# 	* l10n_be
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 13.0\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2020-01-27 14:01+0000\n"
"PO-Revision-Date: 2020-01-27 14:01+0000\n"
"Last-Translator: \n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-00
msgid "0% Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-00-G
msgid "0% Biens divers"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-00-CC
msgid "0% Cocont."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-00-CC
msgid "0% Cocont. - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-00-CC
msgid "0% Cocont. M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-00-CC
msgid "0% Cocont. S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-00-EU
msgid "0% EU - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-00-EU-G
msgid "0% EU - Biens divers"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-00-EU
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-00-EU-L
msgid "0% EU M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-00-EU-S
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-00-EU-S
msgid "0% EU S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-00-EU-T
msgid "0% EU T."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-00
msgid "0% M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-00-ROW
msgid "0% Non EU"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-00-ROW-CC
msgid "0% Non EU - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-00-ROW-CC
msgid "0% Non EU M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-00-ROW-CC
msgid "0% Non EU S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-00-S
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-00-S
msgid "0% S."
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_00
msgid "00"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_00
msgid "00 - Opérations soumises à un régime particulier"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_01
msgid "01"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_01
msgid "01 - Opérations avec TVA à 6%"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_02
msgid "02"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_02
msgid "02 - Opérations avec TVA à 12%"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_03
msgid "03"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_03
msgid "03 - Opérations avec TVA à 21%"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-12-L
msgid "12%"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-12
msgid "12% Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-12-G
msgid "12% Biens divers"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-12-CC
msgid "12% Cocont. - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-12-CC
msgid "12% Cocont. M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-12-CC
msgid "12% Cocont. S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-12-EU
msgid "12% EU - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-12-EU-G
msgid "12% EU - Biens divers"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-12-EU
msgid "12% EU M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-12-EU-S
msgid "12% EU S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-12
msgid "12% M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-12-ROW-CC
msgid "12% Non EU - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-12-ROW-CC
msgid "12% Non EU M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-12-ROW-CC
msgid "12% Non EU S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-12-S
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-12-S
msgid "12% S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-21-L
msgid "21%"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-21
msgid "21% Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-21-G
msgid "21% Biens divers"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-21-CC
msgid "21% Cocont .S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-21-CC
msgid "21% Cocont. - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-21-CC
msgid "21% Cocont. M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-21-EU
msgid "21% EU - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-21-EU-G
msgid "21% EU - Biens divers"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-21-EU
msgid "21% EU M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-21-EU-S
msgid "21% EU S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-21
msgid "21% M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-21-ROW-CC
msgid "21% Non EU - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-21-ROW-CC
msgid "21% Non EU M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-21-ROW-CC
msgid "21% Non EU S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-21-S
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-21-S
msgid "21% S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_TVA-21-inclus-dans-prix
msgid "21% S. TTC"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_44
msgid "44"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_44
msgid "44 - Services intra-communautaires"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_45
msgid "45"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_45
msgid "45 - Opérations avec TVA due par le cocontractant"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_title_operations_sortie_46
msgid "46 - Livraisons intra-communautaires exemptées"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_46L
msgid "46L"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_46L
msgid "46L - Livraisons biens intra-communautaires exemptées"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_46T
msgid "46T"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_46T
msgid "46T - Livraisons biens intra-communautaire exemptées"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_47
msgid "47"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_47
msgid "47 - Autres opérations exemptées"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_title_operations_sortie_48
msgid "48 - Notes de crédit aux opérations grilles [44] et [46]"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_48s44
msgid "48s44"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_48s44
msgid "48s44 - Notes de crédit aux opérations grilles [44]"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_48s46L
msgid "48s46L"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_48s46L
msgid "48s46L - Notes de crédit aux opérations grilles [46L]"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_48s46T
msgid "48s46T"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_48s46T
msgid "48s46T - Notes de crédit aux opérations grilles [46T]"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_49
msgid "49"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_49
msgid "49 - Notes de crédit aux opérations du point II"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-CAR-EXC
msgid "50% Non Déductible - Frais de voiture (Prix Excl.)"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_54
msgid "54"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_54
msgid "54 - TVA sur opérations des grilles [01], [02], [03]"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_55
msgid "55"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_55
msgid "55 - TVA sur opérations des grilles [86] et [88]"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_56
msgid "56"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_56
msgid "56 - TVA sur opérations de la grille [87]"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_57
msgid "57"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_57
msgid "57 - TVA relatives aux importations"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_59
msgid "59"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_59
msgid "59 - TVA déductible"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-06
msgid "6% Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-06-G
msgid "6% Biens divers"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-06-CC
msgid "6% Cocont. - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-06-CC
msgid "6% Cocont. M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-06-CC
msgid "6% Cocont. S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-06-EU
msgid "6% EU - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-06-EU-G
msgid "6% EU - Biens divers"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-06-EU
msgid "6% EU M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-06-EU-S
msgid "6% EU S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-06
msgid "6% M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-06-ROW-CC
msgid "6% Non EU - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-06-ROW-CC
msgid "6% Non EU M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-06-ROW-CC
msgid "6% Non EU S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-06-S
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-06-S
msgid "6% S."
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_61
msgid "61"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_61
msgid "61 - Diverses régularisations en faveur de l'Etat"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_62
msgid "62"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_62
msgid "62 - Diverses régularisations en faveur du déclarant"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_63
msgid "63"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_63
msgid "63 - TVA à reverser sur notes de crédit recues"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_64
msgid "64"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_64
msgid "64 - TVA à récupérer sur notes de crédit delivrées"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_71
msgid "71 - Taxes dues à l'état"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_72
msgid "72 - Somme due par l'état"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_81
msgid "81"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_81
msgid "81 - Marchandises, matières premières et auxiliaires"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_82
msgid "82"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_82
msgid "82 - Services et biens divers"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_83
msgid "83"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_83
msgid "83 - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_84
msgid "84"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_84
msgid "84 - Notes de crédits sur opérations case [86] et [88]"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_85
msgid "85"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_85
msgid "85 - Notes de crédits autres opérations"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_86
msgid "86"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_86
msgid "86 - Acquisition intra-communautaires et ventes ABC"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_87
msgid "87"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_87
msgid "87 - Autres opérations"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_88
msgid "88"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_88
msgid "88 - Acquisition services intra-communautaires"
msgstr ""

#. module: l10n_be
#: model:ir.model,name:l10n_be.model_account_chart_template
msgid "Account Chart Template"
msgstr ""

#. module: l10n_be
#: model:account.chart.template,name:l10n_be.l10nbe_chart_template
msgid "Belgian PCMN"
msgstr ""

#. module: l10n_be
#: model:ir.ui.menu,name:l10n_be.account_reports_be_statements_menu
msgid "Belgium"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_title_operations_sortie
msgid "II A la sortie"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_title_operations_entree
msgid "III A l'entrée"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_title_taxes_dues
msgid "IV Dues"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_title_operations
msgid "Opérations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4168
msgid "Rabais, ristournes, remises à obtenir et autres avoirs non encore reçus"
msgstr ""

#. module: l10n_be
#: model:account.fiscal.position.template,name:l10n_be.fiscal_position_template_4
msgid "Régime Cocontractant"
msgstr ""

#. module: l10n_be
#: model:account.fiscal.position.template,name:l10n_be.fiscal_position_template_2
msgid "Régime Extra-Communautaire"
msgstr ""

#. module: l10n_be
#: model:account.fiscal.position.template,name:l10n_be.fiscal_position_template_3
msgid "Régime Intra-Communautaire"
msgstr ""

#. module: l10n_be
#: model:account.fiscal.position.template,name:l10n_be.fiscal_position_template_1
msgid "Régime National"
msgstr ""

#. module: l10n_be
#: model:account.tax.group,name:l10n_be.tax_group_tva_0
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-00
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-00-G
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-00-S
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-00
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-00-L
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-00-S
msgid "TVA 0%"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-00-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-00-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-00-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-00-CC
msgid "TVA 0% Cocont."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-00-EU
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-00-EU-G
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-00-EU-S
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-00-EU
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-00-EU-L
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-00-EU-S
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-00-EU-T
msgid "TVA 0% EU"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-00-ROW-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-00-ROW-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-00-ROW-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-00-ROW
msgid "TVA 0% Non EU"
msgstr ""

#. module: l10n_be
#: model:account.tax.group,name:l10n_be.tax_group_tva_12
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-12
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-12-G
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-12-S
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-12
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-12-L
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-12-S
msgid "TVA 12%"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-12-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-12-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-12-CC
msgid "TVA 12% Cocont."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-12-EU
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-12-EU-G
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-12-EU-S
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-12-EU
msgid "TVA 12% EU"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-12-ROW-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-12-ROW-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-12-ROW-CC
msgid "TVA 12% Non EU"
msgstr ""

#. module: l10n_be
#: model:account.tax.group,name:l10n_be.tax_group_tva_21
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-21
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-21-G
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-21-S
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-21
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-21-L
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-21-S
msgid "TVA 21%"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-21-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-21-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-21-CC
msgid "TVA 21% Cocont."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-21-EU
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-21-EU-G
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-21-EU-S
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-21-EU
msgid "TVA 21% EU"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-21-ROW-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-21-ROW-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-21-ROW-CC
msgid "TVA 21% Non EU"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_TVA-21-inclus-dans-prix
msgid "TVA 21% TTC"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-CAR-EXC
msgid "TVA 50% Non Déductible - Frais de voiture (Prix Excl.)"
msgstr ""

#. module: l10n_be
#: model:account.tax.group,name:l10n_be.tax_group_tva_6
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-06
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-06-G
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-06-S
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-06
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-06-L
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-06-S
msgid "TVA 6%"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-06-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-06-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-06-CC
msgid "TVA 6% Cocont."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-06-EU
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-06-EU-G
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-06-EU-S
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-06-EU
msgid "TVA 6% EU"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-06-ROW-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-06-ROW-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-06-ROW-CC
msgid "TVA 6% Non EU"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_title_taxes
msgid "Taxes"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_title_taxes_deductibles
msgid "V Déductibles"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_title_taxes_soldes
msgid "VI Soldes"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a000
msgid "Company creditors, beneficiaries of third party guarantees"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a001
msgid "Third party guarantees on behalf of the company"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a010
msgid "Accounts receivable for commitments on bills in circulation"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a0110
msgid "Creditors of commitments on bills in circulation - Bids ceded by the company under its backing"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a0111
msgid "Creditors of commitments on notes in circulation - Other commitments on notes in circulation"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a012
msgid "Accounts receivable for other personal guarantees"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a013
msgid "Creditors of other personal guarantees"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a020
msgid "Company creditors, beneficiaries of real guarantees"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a021
msgid "Actual guarantees established for own account"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a022
msgid "Creditors of third parties, beneficiaries of real guarantees"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a023
msgid "Real guarantees provided on behalf of third parties"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a030
msgid "Statutory deposits"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a031
msgid "Statutory applicants"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a032
msgid "Guarantees received"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a033
msgid "Constituents of guarantees"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a040
msgid "Third parties, holders in their name but at the risks and profits of the business of goods and values"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a041
msgid "Goods and securities held by third parties on their behalf but at the risk and profit of the company"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a050
msgid "Acquisition commitments"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a051
msgid "Creditors of acquisition commitments"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a052
msgid "Accounts receivable for assignment commitments"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a053
msgid "Sale commitment"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a060
msgid "Forward transactions - Goods purchased (to be received)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a061
msgid "Creditors for goods purchased at term"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a062
msgid "Accounts receivable for goods sold forward"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a063
msgid "Forward transactions - Goods sold (to be delivered)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a064
msgid "Forward transactions - Currencies purchased (to be received)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a065
msgid "Creditors for forward currency purchases"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a066
msgid "Accounts receivable for currencies sold forward"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a067
msgid "Forward transactions - Currencies sold (to be delivered)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a0700
msgid "Long-term usage rights - On land and buildings"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a0701
msgid "Long-term usage rights - On installations, machines and tools"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a0702
msgid "Long-term usage rights - On furniture and rolling stock"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a071
msgid "Rent and royalty creditors"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a072
msgid "Goods and values ​​from third parties received on deposit, consignment or custom"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a073
msgid "Principals and depositors of goods and securities"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a074
msgid "Goods and securities held for accounts or at the risk and profit of third parties"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a075
msgid "Creditors of property and securities held on behalf of third parties or at their risk and profit"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a090
msgid "Concordat resolution commitments"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a091
msgid "Concordat resolution claims"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a092
msgid "Creditors under debt restructuring conditions"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a093
msgid "Duties on loan conditions"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a094
msgid "Ongoing litigation"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a095
msgid "Creditors of pending litigation"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a096
msgid "Debtors on technical guarantees"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a097
msgid "Rights on technical guarantees"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a098
msgid "Holders of options (buying or selling securities)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a099
msgid "Options (buy or sell) on securities issued."
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a100
msgid "Issued capital"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a101
msgid "Uncalled capital"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a11
msgid "Share premium account"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a120
msgid "Revaluation surpluses on intangible fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a121
msgid "Revaluation surpluses on tangible fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a122
msgid "Revaluation surpluses on financial fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a123
msgid "Revaluation surpluses on stocks"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a124
msgid "Decrease in amounts written down current investments"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a130
msgid "Legal reserve"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1310
msgid "Reserves not available in respect of own shares held"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1311
msgid "Other reserves not available"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a132
msgid "Untaxed reserves"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a133
msgid "Available reserves"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a140
msgid "Profit brought forward"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a141
msgid "Loss brought forward"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a15
msgid "Investment grants"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a151
msgid "Investment grants received in cash"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a152
msgid "Investment grants received in kind"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a160
msgid "Provisions for pensions and similar obligations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a161
msgid "Provisions for taxation"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a162
msgid "Provisions for major repairs and maintenance"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a163
msgid "Provisions for environmental obligations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1680
msgid "Deferred taxes on investment grants"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1681
msgid "Deferred taxes on gain on disposal of intangible fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1682
msgid "Deferred taxes on gain on disposal of tangible fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1687
msgid "Deferred taxes on gain on disposal of securities issued by Belgian public authorities"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1688
msgid "Foreign deferred taxes"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1700
msgid "Subordinated loans with a remaining term of more than one year - Convertible bonds"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1701
msgid "Subordinated loans with a remaining term of more than one year - Non convertible bonds"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1710
msgid "Unsubordinated debentures with a remaining term of more than one year - Convertible bonds"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1711
msgid "Unsubordinated debentures with a remaining term of more than one year - Non convertible bonds"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1730
msgid "Amounts payable to credit institutions with a remaining term of more than one year - Current account payable"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1731
msgid "Amounts payable to credit institutions with a remaining term of more than one year - Promissory notes"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1732
msgid "Amounts payable to credit institutions with a remaining term of more than one year - Bank acceptances"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a174
msgid "Other loans with a remaining term of more than one year"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1750
msgid "Suppliers (more than one year)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1751
msgid "Bills of exchange payable after more than one year"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a176
msgid "Advances received on contracts in progress (more than one year)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a178
msgid "Amounts payable with a remaining term of more than one year - Guarantees received in cash"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1790
msgid "Miscellaneous amounts payable with a remaining term of more than one year - Interest-bearing"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1791
msgid "Miscellaneous amounts payable with a remaining term of more than one year - Non interest-bearing or with an abnormally low interest rate"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1792
msgid "Miscellaneous amounts payable with a remaining term of more than one year - Cash Deposit"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a19
msgid "Advance to associates on the sharing out of the assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a200
msgid "Formation or capital increase expenses"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a201
msgid "Loan issue expenses"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a202
msgid "Other formation expenses"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a204
msgid "Restructuring costs"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a210
msgid "Research and development costs"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a211
msgid "Concessions, patents, licences, know-how, brands and similar rights"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a212
#: model:account.group.template,name:l10n_be.be_group_212
msgid "Goodwill"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a213
msgid "Intangible fixed assets - Advance payments"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a220
msgid "Land"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2201
msgid "Land owned by the association or the foundation in full property"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2202
msgid "Other land"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a221
msgid "Buildings"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2211
msgid "Building owned by the association or the foundation in full property"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2212
msgid "Other building"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a222
msgid "Developed land"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2221
msgid "Built-up lands owned by the association or the foundation in full property"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2222
msgid "Other built-up lands"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a223
msgid "Other rights to immovable property"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2231
msgid "Other rights to immovable property belonging to the association or the foundation in full property"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2232
msgid "Other rights to immovable property - Other"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a23
msgid "Plant, machinery and equipment"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a231
msgid "Plant, machinery and equipment owned by the association or the foundation in full property"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a232
msgid "Other plant, machinery and equipment"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a24
msgid "Furniture and vehicles"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a241
msgid "Furniture and vehicles owned by the association or the foundation in full property"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a242
msgid "Other furniture and vehicles"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a250
msgid "Leasing and similar rights - Land and buildings"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a251
msgid "Leasing and similar rights - Plant, machinery and equipment"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a252
msgid "Leasing and similar rights - Furniture and vehicles"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a26
msgid "Other tangible fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a261
msgid "Other tangible fixed assets owned by the association or the foundation in full property"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a262
msgid "Other tangible fixed assets - Other"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a27
msgid "Tangible fixed assets under construction and advance payments"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2800
msgid "Participating interests and shares in associated enterprises - Acquisition value"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2801
msgid "Participating interests and shares in associated enterprises - Uncalled amounts"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2808
msgid "Participating interests and shares in associated enterprises - Revaluation surpluses"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2809
msgid "Participating interests and shares in associated enterprises - Amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2810
msgid "Amounts receivable from affiliated enterprises - Current account"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2811
msgid "Amounts receivable from affiliated enterprises - Bills receivable"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2812
msgid "Amounts receivable from affiliated enterprises - Fixed income securities"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2817
msgid "Other amounts receivable from affiliated enterprises - Doubtful amounts"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2819
msgid "Amounts receivable from affiliated enterprises - Amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2820
msgid "Participating interests and shares in enterprises linked by a participating interest - Acquisition value"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2821
msgid "Participating interests and shares in enterprises linked by a participating interest - Uncalled amounts"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2828
msgid "Participating interests and shares in enterprises linked by a participating interest - Revaluation surpluses"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2829
msgid "Participating interests and shares in enterprises linked by a participating interest - Amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2830
msgid "Amounts receivable from other enterprises linked by participating interests - Current account"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2831
msgid "Amounts receivable from other enterprises linked by participating interests - Bills receivable"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2832
msgid "Amounts receivable from other enterprises linked by participating interests - Fixed income securities"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2837
msgid "Amounts receivable from other enterprises linked by participating interests - Doubtful amounts"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2839
msgid "Amounts receivable from other enterprises linked by participating interests - Amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2840
msgid "Other participating interests and shares - Acquisition value"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2841
msgid "Other participating interests and shares - Uncalled amounts"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2848
msgid "Other participating interests and shares - Revaluation surpluses"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2849
msgid "Other participating interests and shares - Amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2850
msgid "Other financial assets - Current account"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2851
msgid "Other financial assets - Bills receivable"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2852
msgid "Other financial assets - Fixed income securities"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2857
msgid "Other financial assets - Doubtful amounts"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2859
msgid "Other financial assets - Amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a288
msgid "Other financial assets - Cash Guarantees"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2900
msgid "Trade debtors after more than one year - Customer"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2901
msgid "Trade debtors after more than one year - Bills receivable"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2906
msgid "Trade debtors after more than one year - Advance payments"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2907
msgid "Trade debtors after more than one year - Doubtful amounts"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2909
msgid "Trade debtors after more than one year - Amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2910
msgid "Other amounts receivable after more than one year - Current account"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2911
msgid "Other amounts receivable after more than one year - Bills receivable"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2915
msgid "Non interest-bearing amounts receivable after more than one year or with an abnormally low interest rate"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2917
msgid "Other amounts receivable after more than one year - Doubtful amounts"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2919
msgid "Other amounts receivable after more than one year - Amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a300
msgid "Raw materials - Acquisition value"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a309
msgid "Raw materials - amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a310
msgid "Consumables - Acquisition value"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a319
msgid "Consumables - amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a320
msgid "Work in progress - Acquisition value"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a329
msgid "Work in progress - amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a330
msgid "Finished goods - Acquisition value"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a339
msgid "Finished goods - amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a340
msgid "Goods purchased for resale - Acquisition value"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a349
msgid "Goods purchased for resale - amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a350
msgid "Immovable property intended for sale - Acquisition value"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a359
msgid "Immovable property intended for sale - amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a360
msgid "Advance payments on purchases for stocks - Acquisition value"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a369
msgid "Advance payments on purchases for stocks - amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a370
msgid "Contracts in progress - Acquisition value"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a371
msgid "Contracts in progress - Profit recognised"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a379
msgid "Contracts in progress - amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a400
msgid "Trade debtors within one year - Customer"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4001
msgid "Customer (POS)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a401
msgid "Trade debtors within one year - Bills receivable"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a404
msgid "Trade debtors within one year - Income receivable"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a406
msgid "Trade debtors within one year - Advance payments"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a407
msgid "Trade debtors within one year - Doubtful amounts"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a409
msgid "Trade debtors within one year - Amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a410
msgid "Called up capital, unpaid"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a411
msgid "VAT recoverable"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4112
msgid "VAT recoverable - Current Account"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a412
msgid "Taxes and withholdings taxes to be recovered"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4128
msgid "Taxes and withholdings taxes to be recovered - Foreign taxes"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a413
msgid "Grants receivable"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a414
msgid "Other amounts receivable within one year - Income receivable"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a415
msgid "Non interest-bearing amounts receivable within one year or with an abnormally low interest rate"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a416
msgid "Other amounts receivable within one year - Sundry amounts"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a417
msgid "Other amounts receivable within one year - Doubtful amounts"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a418
msgid "Other amounts receivable within one year - Guarantees paid in cash"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a419
msgid "Other amounts receivable within one year - Amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4200
msgid "Subordinated loans payable after more than one year falling due within one year - Convertible"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4201
msgid "Subordinated loans payable after more than one year falling due within one year - Non convertible"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4210
msgid "Unsubordinated debentures payable after more than one year falling due within one year - Convertible"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4211
msgid "Unsubordinated debentures payable after more than one year falling due within one year - Non convertible"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a422
msgid "Leasing and similar obligations payable after more than one year falling due within one year"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4230
msgid "Amounts payable after more than one year falling due within one year to credit institutions - Current account payable"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4231
msgid "Amounts payable after more than one year falling due within one year to credit institutions - Promissory notes"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4232
msgid "Amounts payable after more than one year falling due within one year to credit institutions - Bank acceptances"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a424
msgid "Other loans payable after more than one year falling due within one year"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4250
msgid "Amounts payable after more than one year falling due within one year to suppliers"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4251
msgid "Bills of exchange payable after more than one year falling due within one year"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a426
msgid "Advance payments received on contract in progress payable after more than one year falling due within one year"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a428
msgid "Amounts payable after more than one year falling due within one year - Guarantees received in cash"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a429
msgid "Miscellaneous amounts payable after more than one year falling due within one year"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a430
msgid "Amounts payable within one year to credit institutions - Fixed term loans"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a431
msgid "Amounts payable within one year to credit institutions - Promissory notes"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a432
msgid "Amounts payable within one year to credit institutions - Bank acceptances"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a433
msgid "Amounts payable within one year to credit institutions - Current account payable"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a439
msgid "Other loans payable within one year"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a440
msgid "Suppliers payable within one year"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a441
msgid "Bills of exchange payable within one year"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a444
msgid "Invoices to be received payable within one year"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a450
msgid "Estimated taxes payable"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4508
msgid "Estimated taxes payable - Foreign taxes"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a451
msgid "VAT payable"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a451054
msgid "VAT payable - compartment 54"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a451055
msgid "VAT payable - Intracommunity acquisitions - box 55"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a451056
msgid "VAT payable - reverse charge (cocontracting) - compartment 56"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a451057
msgid "VAT payable - reverse charge (import) - compartment 57"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a451063
msgid "VAT payable - credit notes - compartment 63"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4512
msgid "VAT due - Current Account"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a451800
msgid "VAT payable - revisions insufficiencies"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a451820
msgid "VAT payable - revisions of deductions"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a451830
msgid "VAT payable - revisions"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a452
msgid "Taxes payable"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4528
msgid "Taxes payable - Foreign taxes"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a453
msgid "Taxes withheld"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a454
msgid "Remuneration and social security - National Social Security Office"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a455
msgid "Remuneration and social security - Remuneration"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a456
msgid "Remuneration and social security - Holiday pay"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a459
msgid "Remuneration and social security - Other social obligations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a460
msgid "Advances to be received within one year"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a461
msgid "Advances received"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a470
msgid "Dividends and director's fees relating to prior financial periods"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a471
msgid "Dividends - Current financial period"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a472
msgid "Director's fees - Current financial period"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a473
msgid "Other allocations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a480
msgid "Miscellaneous amounts payable within one year - Debentures and matured coupons"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a483
msgid "Miscellaneous amounts payable within one year - Grants to repay"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a487
msgid "Lent securities to return"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a488
msgid "Miscellaneous amounts payable within one year - Guarantees received in cash"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4890
msgid "Miscellaneous amounts payable within one year - Sundry interest-bearing amounts payable"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4891
msgid "Miscellaneous amounts payable within one year - Sundry non interest-bearing amounts payable or with an abnormally low interest rate"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a490
msgid "Deferred charges"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a491
msgid "Accrued income"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a492
msgid "Accrued charges"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a493
msgid "Deferred income"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a496
msgid "Foreign currency translation differences - Assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a497
msgid "Foreign currency translation differences - Liabilities"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a499
msgid "Suspense account"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a500
msgid "Current investments other than shares, fixed income securities and term accounts - Cost"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a509
msgid "Current investments other than shares, fixed income securities and term accounts - Amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a510
msgid "Shares and current investments other than fixed income investments - Acquisition value"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a511
msgid "Shares and current investments other than fixed income investments - Uncalled amount"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a519
msgid "Shares and current investments other than fixed income investments - Amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a520
msgid "Fixed income securities - Acquisition value"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a529
msgid "Fixed income securities - Amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a530
msgid "Fixed term deposit over one year"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a531
msgid "Fixed term deposit between one month and one year"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a532
msgid "Fixed term deposit up to one month"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a539
msgid "Fixed term deposit - Amounts written down"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a54
msgid "Cash at bank - Amounts overdue and in the process of collection"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a55
msgid "Cash at bank - Credit institutions"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a560
msgid "Cash at bank - Giro account - Bank account"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a561
msgid "Cash at bank - Giro account - Cheques issued"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a57
msgid "Cash in hand"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a578
msgid "Cash in hand - Stamps"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a58
msgid "Cash at bank and in hand - Internal transfers of funds"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a600
msgid "Purchases of raw materials"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a601
msgid "Purchases of consumables"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a602
msgid "Purchases of services, works and studies"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a603
msgid "Sub-contracting"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a604
msgid "Purchases of goods for resale"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a605
msgid "Purchases of immovable property for resale"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a608
msgid "Discounts, allowance and rebates received on purchase of raw materials, consumables"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6090
msgid "Decrease (increase) in stocks of raw materials"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6091
msgid "Decrease (increase) in stocks of consumables"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6094
msgid "Decrease (increase) in stocks of goods purchased for resale"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6095
msgid "Decrease (increase) in immovable property for resale"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a61
msgid "Services and other goods"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a617
msgid "Costs of hired temporary staff and persons placed at the enterprise's disposal"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a618
msgid "Remuneration, premiums for extra statutory insurance, pensions of the directors, or the management staff which are not allowed following the contract"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6200
msgid "Remuneration and direct social benefits - Directors and managers"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6201
msgid "Remuneration and direct social benefits - Executive"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6202
msgid "Remuneration and direct social benefits - Employees"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6203
msgid "Remuneration and direct social benefits - Manual workers"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6204
msgid "Remuneration and direct social benefits - Other staff members"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a621
msgid "Employers' contribution for social security"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a622
msgid "Employers' premiums for extra statutory insurance"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a623
msgid "Other personnel costs"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6240
msgid "Retirement and survivors' pensions - Directors and managers"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6241
msgid "Retirement and survivors' pensions - Personnel"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6300
msgid "Depreciation of formation expenses"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6301
msgid "Depreciation of intangible fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6302
msgid "Depreciation of tangible fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6308
msgid "Amounts written off intangible fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6309
msgid "Amounts written off tangible fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6310
msgid "Amounts written off stocks - Appropriations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6311
msgid "Amounts written off stocks - Write-backs"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6320
msgid "Amounts written off contracts in progress - Appropriations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6321
msgid "Amounts written off contracts in progress - Write-backs"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6330
msgid "Amounts written off trade debtors (more than one year) - Appropriations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6331
msgid "Amounts written off trade debtors (more than one year) - Write-backs"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6340
msgid "Amounts written off trade debtors (within one year) - Appropriations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6341
msgid "Amounts written off trade debtors (within one year) - Write-backs"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6350
msgid "Provisions for pensions and similar obligations - Appropriations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6351
msgid "Provisions for pensions and similar obligations - Uses and write-backs"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6360
msgid "Provision for major repairs and maintenance - Appropriations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6361
msgid "Provision for major repairs and maintenance - Uses and write-backs"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6370
msgid "Provisions for other risks and charges - Appropriations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6371
msgid "Provisions for other risks and charges - Uses (write-back)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6380
msgid "Provisions for other risks and charges - Provisions for environmental obligations excluded - Appropriations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6381
msgid "Provisions for other risks and charges - Provisions for environmental obligations excluded - Uses (write-back)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a640
msgid "Taxes related to operation"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a64012
msgid "Non deductible taxes"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a641
msgid "Loss on ordinary disposal of tangible fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a642
msgid "Loss on ordinary disposal of trade debtors"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a643
msgid "Operating charges - Gifts"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6431
msgid "Operating charges - Gifts with a recovery right"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6432
msgid "Operating charges - Gifts without any recovery right"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a649
msgid "Operating charges carried to assets as restructuring costs"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6500
msgid "Interests, commissions and other charges relating to debts"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6501
msgid "Depreciation of loan issue expenses"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6502
msgid "Other debt charges"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6503
msgid "Capitalized Interests"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6510
msgid "Amounts written off current assets except stocks, contracts in progress and trade debtors - Appropriations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6511
msgid "Amounts written off current assets except stocks, contracts in progress and trade debtors - Write-backs"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a652
msgid "Losses on disposal of current assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a653
msgid "Amount of the discount borne by the enterprise, as a result of negotiating amounts receivable"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a654
msgid "Financial charges - Exchange differences"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a655
msgid "Financial charges - Foreign currency translation differences"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6560
msgid "Provisions of a financial nature - Appropriations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6561
msgid "Provisions of a financial nature - Uses and write-backs"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a659
msgid "Financial charges carried to assets as restructuring costs"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6600
msgid "Non-recurring depreciation of and amounts written off formation expenses"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6601
msgid "Non-recurring depreciation of and amounts written off intangible fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6602
msgid "Non-recurring depreciation of and amounts written off tangible fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a661
msgid "Amounts written off financial fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a66200
msgid "Provisions for non-recurring operating liabilities and charges - Appropriations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a66201
msgid "Provisions for non-recurring operating liabilities and charges - Uses"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a66210
msgid "Provisions for non-recurring financial liabilities and charges - Appropriations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a66211
msgid "Provisions for non-recurring financial liabilities and charges - Uses"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6630
msgid "Capital losses on disposal of intangible and tangible fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6631
msgid "Capital losses on disposal of financial fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a668
msgid "Other  non-recurring financial charges"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6690
msgid "Non-recurring operating charges carried to assets as restructuring costs"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6691
msgid "Non-recurring financial charges carried to assets as restructuring costs"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6700
msgid "Belgian income taxes on the result of the current period - Income taxes paid and withholding taxes due or paid"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6701
msgid "Belgian and foreign income taxes - Income taxes - Withholding taxes on immovables"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6702
msgid "Belgian and foreign income taxes - Income taxes - Withholding taxes on investment income"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6703
msgid "Belgian and foreign income taxes - Income taxes - Other income taxes"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6710
msgid "Belgian income taxes on the result of prior periods - Additional charges for income taxes due or paid"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6711
msgid "Belgian income taxes on the result of prior periods - Additional charges for estimated income taxes"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6712
msgid "Belgian income taxes on the result of prior periods - Additional charges for income taxes provided for"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a672
msgid "Foreign income taxes on the result of the current period"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a673
msgid "Foreign income taxes on the result of prior periods"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a680
msgid "Transfer to deferred taxes"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a689
msgid "Transfer to untaxed reserves"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a690
msgid "Loss brought forward from previous year"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a691
msgid "Appropriations to capital and share premium account"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6920
msgid "Appropriations to legal reserve"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6921
msgid "Appropriations to other reserves"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a693
msgid "Profits to be carried forward"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a694
msgid "Dividends"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a695
msgid "Directors' or managers' entitlements"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a696
msgid "Employees' entitlements"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a697
msgid "Other allocations entitlements"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7000
msgid "Sales rendered in Belgium (marchandises)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7001
msgid "Sales rendered in E.E.C. (marchandises)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7002
msgid "Sales rendered for export (marchandises)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7010
msgid "Sales rendered in Belgium (finished goods)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7011
msgid "Sales rendered in E.E.C. (finished goods)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7012
msgid "Sales rendered for export (finished goods)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7050
msgid "Services rendered in Belgium"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7051
msgid "Services rendered in E.E.C."
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7052
msgid "Services rendered for export"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a708
msgid "Discounts, allowances and rebates allowed"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a71
msgid "Increase (decrease) in stocks of finished goods and work and contracts in progress"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a712
msgid "Increase (decrease) in work in progress"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a713
msgid "Increase (decrease) in stocks of finished goods"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a715
msgid "Increase (decrease) in stocks of immovable property constructed for resale"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7170
msgid "Increase (decrease) in contracts in progress - Acquisition value"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7171
msgid "Increase (decrease) in contracts in progress - Profit recognized"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a72
msgid "Own work capitalised"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a730
msgid "Contributions from effective members"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a731
msgid "Contributions from members"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a732
msgid "Gifts without any recovery right"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a733
msgid "Gifts with a recovery right"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a734
msgid "Legacies without any recovery right"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a735
msgid "Legacies with a recovery right"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a736
msgid "Contributions, gifts, legacies and grants - Investment grants and interest subsidies"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a737
msgid "Operating Subsidies"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a738
msgid "Compensatory amounts meant to reduce wage costs"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a740
msgid "Operating subsidies and compensatory amounts"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a741
msgid "Gain on ordinary disposal of tangible fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a742
msgid "Gain on ordinary disposal of trade debtors"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a750
msgid "Income from financial fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a751
msgid "Income from current assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a752
msgid "Gain on disposal of current assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a753
msgid "Investment grants and interest subsidies"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a754
msgid "Financial income - Exchange differences"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a755
msgid "Financial income - Foreign currency translation differences"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7600
msgid "Write-back of depreciation and of amounts written off intangible fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7601
msgid "Write-back of depreciation and of amounts written off tangible fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a761
msgid "Write-back of amounts written down financial fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7620
msgid "Write-back of provisions for non-recurring operating liabilities and charges"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7621
msgid "Write-back of provisions for non-recurring financial liabilities and charges"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7630
msgid "Capital gains on disposal of intangible and tangible fixed asset"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7631
msgid "Capital gains on disposal of financial fixed assets"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a769
msgid "Other  non-recurring financial income"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a77
msgid "Adjustment of income taxes and write-back of tax provisions"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7710
msgid "Adjustment of Belgian income taxes - Taxes due or paid"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7711
msgid "Adjustment of Belgian income taxes - Estimated taxes"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7712
msgid "Adjustment of Belgian income taxes - Tax provisions written back"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a773
msgid "Adjustment of foreign income taxes"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a780
msgid "Transfer from deferred taxes"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a789
msgid "Transfer from untaxed reserves"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a790
msgid "Profit brought forward from previous year"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a791
msgid "Withdrawal from the association or foundation funds"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a792
msgid "Withdrawal from allocated funds"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a793
msgid "Losses to be carried forward"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a794
msgid "Owners' contribution in respect of losses"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_1
msgid "Fonds propres, provisions pour risques et charges et dettes à plus d'un an"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_10
msgid "Capital"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_100
msgid "Capital souscrit"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_101
msgid "Capital non appelé (–)"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_11
msgid "Primes d'émission"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_12
msgid "Plus-values de réévaluation"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_120
msgid "Plus-values de réévaluation sur immobilisations incorporelles"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_121
msgid "Plus-values de réévaluation sur immobilisations corporelles"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_122
msgid "Plus-values de réévaluation sur immobilisations financières"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_123
msgid "Plus-values de réévaluation sur stocks"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_124
msgid "Reprises de réductions de valeur sur placements de trésorerie"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_13
msgid "Réserves"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_130
msgid "Réserve légale"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_131
msgid "Réserves indisponibles"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_132
msgid "Réserves immunisées"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_133
msgid "Réserves disponibles"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_14
msgid "Bénéfice reporté ou Perte reportée (–)"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_15
msgid "Subsides en capital"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_16
msgid "Provisions et impôts différés"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_160
#: model:account.group.template,name:l10n_be.be_group_635
msgid "Provisions pour pensions et obligations similaires"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_161
msgid "Provisions pour charges fiscales"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_162
#: model:account.group.template,name:l10n_be.be_group_636
msgid "Provisions pour grosses réparations et gros entretien"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_163
#: model:account.group.template,name:l10n_be.be_group_637
msgid "Provisions pour obligations environnementales"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_164
#: model:account.group.template,name:l10n_be.be_group_638
msgid "Provisions pour autres risques et charges"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_168
msgid "Impôts différés"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_17
msgid "Dettes à plus d'un an"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_170
msgid "Emprunts subordonnés"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_171
msgid "Emprunts obligataires non subordonnés"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_172
msgid "Dettes de location-financement et dettes assimilées"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_173
#: model:account.group.template,name:l10n_be.be_group_55
msgid "Etablissements de crédit"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_174
#: model:account.group.template,name:l10n_be.be_group_439
msgid "Autres emprunts"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_175
#: model:account.group.template,name:l10n_be.be_group_44
msgid "Dettes commerciales"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_176
#: model:account.group.template,name:l10n_be.be_group_46
msgid "Acomptes reçus sur commandes"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_178
#: model:account.group.template,name:l10n_be.be_group_488
msgid "Cautionnements reçus en numéraire"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_179
#: model:account.group.template,name:l10n_be.be_group_48
msgid "Dettes diverses"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_19
msgid "Acompte aux associés sur le partage de l'actif net (-)"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_2
msgid "Frais d'établissement, actifs immobilisés et créances à plus d'un an"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_20
msgid "Frais d'établissement"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_200
msgid "Frais de constitution et d'augmentation de capital"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_201
msgid "Frais d'émission d'emprunts"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_202
msgid "Autres frais d'établissement"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_204
msgid "Frais de restructuration"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_21
msgid "Immobilisation incorporelles"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_210
msgid "Frais de recherche et de développement"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_211
msgid "Concessions, brevets, licences, savoir-faire, marques et droits similaires"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_213
#: model:account.group.template,name:l10n_be.be_group_360
#: model:account.group.template,name:l10n_be.be_group_406
msgid "Acomptes versés"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_22
msgid "Terrains et constructions"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_220
msgid "Terrains"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_221
msgid "Constructions"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_222
msgid "Terrains bâtis"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_223
msgid "Autres droits réels sur des immeubles"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_23
#: model:account.group.template,name:l10n_be.be_group_251
msgid "Installations, machines et outillage"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_24
#: model:account.group.template,name:l10n_be.be_group_252
msgid "Mobilier et matériel roulant"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_25
msgid "Immobilisations détenues en location-financement et droits similaires"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_250
msgid "Terrains et construction"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_26
msgid "Autres immobilisations corporelles"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_27
msgid "Immobilisations corporelles en cours et acomptes versés"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_28
msgid "Immobilisations financières"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_280
msgid "Participations dans des entreprises liées"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_281
msgid "Créances sur des entreprises liées"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_282
msgid "Participations dans des entreprises avec lesquelles il existe un lien de participation"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_283
msgid "Créances sur des entreprises avec lesquelles il existe un lien de participation"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_284
msgid "Autres actions et parts"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_285
#: model:account.group.template,name:l10n_be.be_group_291
#: model:account.group.template,name:l10n_be.be_group_41
msgid "Autres créances"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_288
#: model:account.group.template,name:l10n_be.be_group_418
msgid "Cautionnements versés en numéraire"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_29
msgid "Créances à plus d'un an"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_290
#: model:account.group.template,name:l10n_be.be_group_40
msgid "Créances commerciales"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_3
msgid "Stocks et commandes en cours d'exécution"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_30
msgid "Approvisionnements - Matières premières"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_300
#: model:account.group.template,name:l10n_be.be_group_310
#: model:account.group.template,name:l10n_be.be_group_320
#: model:account.group.template,name:l10n_be.be_group_340
#: model:account.group.template,name:l10n_be.be_group_350
#: model:account.group.template,name:l10n_be.be_group_370
#: model:account.group.template,name:l10n_be.be_group_510
#: model:account.group.template,name:l10n_be.be_group_520
msgid "Valeur d'acquisition"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_309
#: model:account.group.template,name:l10n_be.be_group_319
#: model:account.group.template,name:l10n_be.be_group_329
#: model:account.group.template,name:l10n_be.be_group_339
#: model:account.group.template,name:l10n_be.be_group_349
#: model:account.group.template,name:l10n_be.be_group_359
#: model:account.group.template,name:l10n_be.be_group_369
#: model:account.group.template,name:l10n_be.be_group_379
#: model:account.group.template,name:l10n_be.be_group_409
#: model:account.group.template,name:l10n_be.be_group_419
#: model:account.group.template,name:l10n_be.be_group_529
#: model:account.group.template,name:l10n_be.be_group_539
msgid "Réductions de valeur actées (–)"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_31
msgid "Approvisionnements - Fournitures"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_32
msgid "En-cours de fabrication"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_33
#: model:account.group.template,name:l10n_be.be_group_330
msgid "Produits finis"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_34
msgid "Marchandises"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_35
msgid "Immeubles destinés à la vente"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_36
msgid "Acomptes versés sur achats pour stocks"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_37
msgid "Commandes en cours d'exécution"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_371
msgid "Bénéfice pris en compte"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_4
msgid "Créances et dettes à un an au plus"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_400
msgid "Clients"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_401
msgid "Effets à recevoir"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_404
#: model:account.group.template,name:l10n_be.be_group_414
msgid "Produits à recevoir"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_407
#: model:account.group.template,name:l10n_be.be_group_417
msgid "Créances douteuses"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_410
msgid "Capital appelé, non versé"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_411
msgid "T.V.A. à récupérer"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_412
msgid "Impôts et précomptes à récupérer"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_416
msgid "Créances diverses"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_42
msgid "Dettes à plus d'un an échéant dans l'année 16 (même subdivision que le 17)"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_43
msgid "Dettes financières"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_430
msgid "Etablissements de crédit - Emprunts en compte à terme fixe"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_431
msgid "Etablissements de crédit - Promesses"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_432
msgid "Etablissements de crédit - Crédits d'acceptation"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_433
msgid "Etablissements de crédit - Dettes en compte courant"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_440
msgid "Fournisseurs"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_441
msgid "Effets à payer"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_444
msgid "Factures à recevoir"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_45
msgid "Dettes fiscales, salariales et sociales"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_450
msgid "Dettes fiscales estimées"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_451
msgid "T.V.A. à payer"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_452
msgid "Impôts et taxes à payer"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_453
msgid "Précomptes retenus"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_454
msgid "Office national de la sécurité sociale"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_455
msgid "Rémunérations"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_456
msgid "Pécules de vacances"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_459
msgid "Autres dettes sociales"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_47
msgid "Dettes découlant de l'affectation du résultat"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_470
msgid "Dividendes et tantièmes d'exercices antérieurs"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_471
msgid "Dividendes de l'exercice"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_472
msgid "Tantièmes de l'exercice"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_473
msgid "Autres allocataires"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_480
msgid "Obligations et coupons échus"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_489
msgid "Autres dettes diverses"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_49
msgid "Comptes de régularisation et comptes d'attente"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_490
msgid "Charges à reporter"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_491
msgid "Produits acquis"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_492
msgid "Charges à imputer"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_493
msgid "Produits à reporter"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_499
msgid "Comptes d'attente"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_5
msgid "Placements de trésorerie et valeurs disponibles"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_50
msgid "Actions propres"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_51
msgid "Actions, parts et placements de trésorerie autres que placements à revenu fixe"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_5100
#: model:account.group.template,name:l10n_be.be_group_5110
#: model:account.group.template,name:l10n_be.be_group_5190
msgid "Actions et parts"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_5101
#: model:account.group.template,name:l10n_be.be_group_5191
msgid "Placements de trésorerie autres que placements à revenu fixe"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_511
msgid "Montants non appelés (-)"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_519
msgid "Réductions de valeur actées (-)"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_52
msgid "Titres à revenu fixe"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_53
msgid "Dépôts à terme"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_530
msgid "De plus d'un an"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_531
msgid "De plus d'un mois et à un an au plus"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_532
msgid "D'un mois au plus"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_54
msgid "Valeurs échues à l'encaissement"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_550
msgid "Comptes ouverts auprès des divers établissements, à subdiviser en :"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_56
msgid "Office des chèques postaux"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_560
msgid "Compte courant"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_561
msgid "Chèques émis (–)"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_57
msgid "Caisses"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_570
msgid "Caisses-espèces"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_578
msgid "Caisses-timbres"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_58
msgid "Virements internes"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_6
msgid "Charges"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_60
msgid "Approvisionnements et marchandises"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_600
msgid "Achats de matières premières"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_601
msgid "Achats de fournitures"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_602
msgid "Achats de services, travaux et études"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_603
msgid "Sous-traitances générales"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_604
msgid "Achats de marchandises"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_605
msgid "Achats d'immeubles destinés à la vente"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_608
msgid "Remises, ristournes et rabais obtenus (–)"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_609
msgid "Variations des stocks"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_61
msgid "Services et biens divers"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_617
msgid "Personnel intérimaire et personnes mises à la disposition de l'entreprise"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_618
msgid "Rémunérations, primes pour assurances extralégales, pensions de retraite et de survie des administrateurs, gérants et associés actifs qui ne sont pas attribuées en vertu d'un contrat de travail"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_62
msgid "Rémunérations, charges sociales et pensions"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_620
msgid "Rémunérations et avantages sociaux directs"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_621
msgid "Cotisations patronales d'assurances sociales"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_622
msgid "Primes patronales pour assurances extra-légales"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_623
msgid "Autres frais de personnel"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_624
msgid "Pensions de retraite et de survie"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_63
msgid "Amortissements, réductions de valeur et provisions pour risques et charges"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_630
msgid "Dotations aux amortissements et aux réductions de valeur sur immobilisations"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_631
msgid "Réductions de valeur sur stocks"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_632
msgid "Réductions de valeur sur commandes en cours"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_633
msgid "Réductions de valeur sur créances commerciales à plus d'un an"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_634
msgid "Réductions de valeur sur créances commerciales à un an au plus"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_64
msgid "Autres charges d'exploitation"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_640
msgid "Charges fiscales d'exploitation"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_641
msgid "Moins-values sur réalisations courantes d'immobilisations corporelles"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_642
msgid "Moins-values sur réalisations de créances commerciales"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_643
msgid "Charges d'exploitation diverses"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_649
msgid "Charges d'exploitation portées à l'actif au titre de frais de restructuration (–)"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_65
msgid "Charges financières"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_650
msgid "Charges des dettes"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_651
msgid "Réductions de valeur sur actifs circulants"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_652
msgid "Moins-values sur réalisation d'actifs circulants"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_653
msgid "Charges d'escompte de créances"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_654
#: model:account.group.template,name:l10n_be.be_group_754
msgid "Différences de change"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_655
msgid "Ecarts de conversion des devises"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_656
msgid "Provisions à caractère financier"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_657
msgid "Charges financières diverses"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_659
msgid "Charges financières portées à l'actif au titre de frais de restructuration"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_66
msgid "Charges d'exploitation ou financières non récurrentes"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_660
msgid "Amortissements et réductions de valeur non récurrents (dotations)"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_661
msgid "Réductions de valeur sur immobilisations financières (dotations)"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_662
msgid "Provisions pour risques et charges non récurrents"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_663
msgid "Moins-values sur réalisation d'actifs immobilisés"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_664
msgid "Autres charges d'exploitation non récurrentes"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_668
msgid "Autres charges financières non récurrentes"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_669
msgid "Charges portées à l'actif au titre de frais de restructuration (-)"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_67
msgid "Impôts sur le résultat"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_670
msgid "Impôts belges sur le résultat de l'exercice"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_671
msgid "Impôts belges sur le résultat d'exercices antérieurs"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_672
msgid "Impôts étrangers sur le résultat de l'exercice"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_673
msgid "Impôts étrangers sur le résultat d'exercices antérieurs"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_68
msgid "Transferts aux impôts différés et aux réserves immunisées"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_680
msgid "Transferts aux impôts différés"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_689
msgid "Transferts aux réserves immunisées"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_69
#: model:account.group.template,name:l10n_be.be_group_79
msgid "Affectations et prélèvements"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_690
msgid "Perte reportée de l'exercice précédent"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_691
msgid "Affectations au capital et à la prime d'émission"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_692
msgid "Dotation aux réserves"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_693
msgid "Bénéfice à reporter"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_694
msgid "Rémunération du capital"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_695
msgid "Administrateurs ou gérants"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_696
msgid "Employés"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_697
msgid "Autres applications"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_7
msgid "Produits"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_70
msgid "Chiffre d'affaires"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_700
msgid "Ventes et prestations de services"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_708
msgid "Remises, ristournes et rabais accordés (–)"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_71
msgid "Variation des stocks et des commandes en cours d'exécution"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_712
msgid "Des en-cours de fabrication"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_713
msgid "Des produits finis"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_715
msgid "Des immeubles construits destinés à la vente"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_717
msgid "Des commandes en cours d'exécution"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_72
msgid "Production immobilisée"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_74
msgid "Autres produits d'exploitation"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_740
msgid "Subsides d'exploitation et montants compensatoires"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_741
msgid "Plus-values sur réalisations courantes d'immobilisations corporelles"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_742
msgid "Plus-values sur réalisation de créances commerciales"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_743
msgid "Produits d'exploitation divers"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_75
msgid "Produits financiers"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_750
msgid "Produits des immobilisations financières"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_751
msgid "Produits des actifs circulants"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_752
msgid "Plus-values sur réalisation d'actifs circulants"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_753
msgid "Subsides en capital et en intérêts"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_755
msgid "Écart de conversion des devises"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_756
msgid "Produits financiers divers"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_76
msgid "Produits d'exploitation ou financiers non récurrents"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_760
msgid "Reprises d'amortissements et de réductions de valeur"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_761
msgid "Reprises de réductions de valeur sur immobilisations financières"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_762
msgid "Reprises de provisions pour risques et charges non récurrents"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_763
msgid "Plus-values sur réalisation d'actifs immobilisés"
msgstr ""

#. module: l10n_be
#: model:account.fiscal.position.template,name:l10n_be.fiscal_position_template_5
msgid "EU privé"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_764
msgid "Autres produits d'exploitation non récurrents"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_769
msgid "Autres produits financiers non récurrents"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_77
msgid "Régularisations d'impôts et reprises de provisions fiscales"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_771
msgid "Impôts belges sur le résultat"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_773
msgid "Impôts étrangers sur le résultat"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_78
msgid "Prélèvements sur les réserves immunisées et les impôts différés"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_780
msgid "Prélèvements sur les impôts différés"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_789
msgid "Prélèvements sur les réserves immunisées"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_790
msgid "Bénéfice reporté de l'exercice précédent"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_791
msgid "Prélèvement sur le capital et les primes d'émission"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_792
msgid "Prélèvement sur les réserves"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_793
msgid "Perte à reporter"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_794
msgid "Intervention d'associés (ou du propriétaire) dans la perte"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_0
msgid "Droits et engagements hors bilan"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_00
msgid "Garanties constituées par des tiers pour compte de l'entreprise"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_000
msgid "Créanciers de l'entreprise, bénéficiaires de garanties de tiers"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_001
msgid "Tiers constituants de garanties pour compte de l'entreprise"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_01
msgid "Garanties personnelles constituées pour compte de tiers"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_010
msgid "Débiteurs pour engagements sur effets en circulation"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_011
msgid "Créanciers d'engagements sur effets en circulation"
msgstr ""

#. module: l10n_be
msgid "Créanciers d'engagements sur effets en circulation - Effets cédés par l’entreprise sous son endos"
msgstr ""

#. module: l10n_be
msgid "Créanciers d'engagements sur effets en circulation - Autres engagements sur effets en circulation"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_012
msgid "Débiteurs pour autres garanties personnelles"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_013
msgid "Créanciers d'autres garanties personnelles"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_02
msgid "Garanties réelles constituées sur avoirs propres"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_020
msgid "Créanciers de l'entreprise, bénéficiaires de garanties réelles"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_021
msgid "Garanties réelles constituées pour compte propre"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_022
msgid "Créanciers de tiers, bénéficiaires de garanties réelles"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_023
msgid "Garanties réelles constituées pour compte de tiers"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_03
#: model:account.group.template,name:l10n_be.be_group_032
msgid "Garanties reçues"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_030
msgid "Dépôts statutaires"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_031
msgid "Déposants statutaires"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_033
msgid "Constituants de garanties"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_04
#: model:account.group.template,name:l10n_be.be_group_041
msgid "Biens et valeurs détenus par des tiers en leur nom mais aux risques et profits de l'entreprise"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_040
msgid "Tiers, détenteurs en leur nom mais aux risques et profits de l'entreprise de biens et de valeurs"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_05
msgid "Engagements d'acquisition et de cession d'immobilisations"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_050
msgid "Engagements d'acquisition"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_051
msgid "Créanciers d'engagements d'acquisition"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_052
msgid "Débiteurs pour engagements de cession"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_053
msgid "Engagements de cession"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_06
msgid "Marchés à terme"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_060
msgid "Marchandises achetées à terme - à recevoir"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_061
msgid "Créanciers pour marchandises achetées à terme"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_062
msgid "Débiteurs pour marchandises vendues à terme"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_063
msgid "Marchandises vendues à terme - à livrer"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_064
msgid "Devises achetées à terme - à recevoir"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_065
msgid "Créanciers pour devises achetées à terme"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_066
msgid "Débiteurs pour devises vendues à terme"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_067
msgid "Devises vendues à terme - à livrer"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_07
msgid "Biens et valeurs de tiers détenus par l'entreprise"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_070
msgid "Droits d'usage à long terme"
msgstr ""

#. module: l10n_be
msgid "Droits d'usage à long terme - Sur terrains et constructions"
msgstr ""

#. module: l10n_be
msgid "Droits d'usage à long terme - Sur installations, machines et outillage"
msgstr ""

#. module: l10n_be
msgid "Droits d'usage à long terme - Sur mobilier et matériel roulant"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_071
msgid "Créanciers de loyers et redevances"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_072
msgid "Biens et valeurs de tiers reçus en dépôt, en consignation ou à façon"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_073
msgid "Commettants et déposants de biens et de valeurs"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_074
msgid "Biens et valeurs détenus pour compte ou aux risques et profits de tiers"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_075
msgid "Créanciers de biens et valeurs détenus pour compte de tiers ou à leurs risques et profits"
msgstr ""

#. module: l10n_be
#: model:account.group.template,name:l10n_be.be_group_09
msgid "Droits et engagements divers"
msgstr ""

#. module: l10n_be
#: model:ir.model.fields.selection,name:l10n_be.selection__account_journal__invoice_reference_model__be
msgid "Belgium"
msgstr ""

#. module: l10n_be
#: model:ir.model.fields,field_description:l10n_be.field_account_journal__invoice_reference_model
msgid "Communication Standard"
msgstr ""

#. module: l10n_be
#: model:ir.model,name:l10n_be.model_account_journal
msgid "Journal"
msgstr ""

#. module: l10n_be
#: model:ir.model,name:l10n_be.model_account_move
msgid "Journal Entries"
msgstr ""

#. module: l10n_be
#: model:ir.model.fields,help:l10n_be.field_account_journal__invoice_reference_model
msgid ""
"You can choose different models for each type of reference. The default one "
"is the Odoo reference."
msgstr ""

```

## File: models\account_chart_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields


class AccountChartTemplate(models.Model):
    _inherit = 'account.chart.template'

    @api.model
    def _prepare_all_journals(self, acc_template_ref, company, journals_dict=None):
        journal_data = super(AccountChartTemplate, self)._prepare_all_journals(
            acc_template_ref, company, journals_dict)
        for journal in journal_data:
            if journal['type'] in ('sale', 'purchase') and company.country_id.code == "BE":
                journal.update({'refund_sequence': True})
        return journal_data

```

## File: models\account_invoice.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2011 Noviat nv/sa (www.noviat.be). All rights reserved.

import random
import re

from odoo import api, fields, models, _
from odoo.exceptions import UserError

"""
account.move object: add support for Belgian structured communication
"""


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _get_invoice_reference_be_partner(self):
        """ This computes the reference based on the belgian national standard
            “OGM-VCS”.
            For instance, if an invoice is issued for the partner with internal
            reference 'food buyer 654', the digits will be extracted and used as
            the data. This will lead to a check number equal to 72 and the
            reference will be '+++000/0000/65472+++'.
            If no reference is set for the partner, its id in the database will
            be used.
        """
        self.ensure_one()
        bbacomm = (re.sub('\D', '', self.partner_id.ref or '') or str(self.partner_id.id))[-10:].rjust(10, '0')
        base = int(bbacomm)
        mod = base % 97 or 97
        reference = '+++%s/%s/%s%02d+++' % (bbacomm[:3], bbacomm[3:7], bbacomm[7:], mod)
        return reference

    def _get_invoice_reference_be_invoice(self):
        """ This computes the reference based on the belgian national standard
            “OGM-VCS”.
            The data of the reference is the database id number of the invoice.
            For instance, if an invoice is issued with id 654, the check number
            is 72 so the reference will be '+++000/0000/65472+++'.
        """
        self.ensure_one()
        base = self.id
        bbacomm = str(base).rjust(10, '0')
        base = int(bbacomm)
        mod = base % 97 or 97
        reference = '+++%s/%s/%s%02d+++' % (bbacomm[:3], bbacomm[3:7], bbacomm[7:], mod)
        return reference

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    invoice_reference_model = fields.Selection(selection_add=[
        ('be', 'Belgium')
        ], ondelete={'be': lambda recs: recs.write({'invoice_reference_model': 'odoo'})})


```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_chart_template
from . import account_invoice
from . import account_journal

```

