# Odoo Module: l10n_fi

Category: Localization

This file contains the source code of the Odoo module.

## File: __init__.py

```python
#-*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from. import models
from. import tests

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    "name": "Finnish Localization",
    "version": "13.0.1",
    "author": "Avoin.Systems, "
              "Tawasta, "
              "Vizucom, "
              "Sprintit",
    "category": "Localization",
    "description": """
This is the Odoo module to manage the accounting in Finland.
============================================================

After installing this module, you'll have access to :
    * Finnish chart of account
    * Fiscal positions
    * Invoice Payment Reference Types (Finnish Standard Reference & Finnish Creditor Reference (RF))
    * Finnish Reference format for Sale Orders

Set the payment reference type from the Sales Journal.
    """,
    "depends": [
        'account',
        'base_iban',
        'base_vat',
    ],
    "data": [
        'data/account_account_tag_data.xml',
        'data/account_chart_template_data.xml',
        'data/account.account.template.csv',
        'data/account_tax_report_line.xml',
        'data/account_data.xml',
        'data/account_tax_template_data.xml',
        'data/l10n_fi_chart_post_data.xml',
        'data/account_fiscal_position_template_data.xml',
        'data/account_chart_template_configuration_data.xml'
    ],
    "active": True,
    "installable": True,
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","code","name","user_type_id/id","chart_template_id/id","reconcile","tag_ids/id"
"account_1020","1020","Kehittämismenot","account.data_account_type_non_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_development_costs"
"account_1030","1030","Aineettomat oikeudet","account.data_account_type_non_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_immaterial_rights"
"account_1050","1050","Liikearvo","account.data_account_type_non_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_goodwill"
"account_1060","1060","Konserniliikearvo","account.data_account_type_non_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_goodwill"
"account_1070","1070","Muut pitkävaikutteiset menot","account.data_account_type_non_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_long_term_expenses"
"account_1090","1090","Ennakkomaksut aineettomista hyödykkeistä","account.data_account_type_non_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_advance_payments_intangible"
"account_1100","1100","Maa- ja vesialueet","account.data_account_type_non_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_land_and_water_areas_owned"
"account_1120","1120","Rakennukset ja rakennelmat","account.data_account_type_non_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_buildings_owned"
"account_1160","1160","Koneet ja kalusto","account.data_account_type_non_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_machines_and_hardware"
"account_1300","1300","Muut aineelliset hyödykkeet","account.data_account_type_non_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_tangible_assets"
"account_1380","1380","Ennakkomaksut ja keskeneräiset hankinnat","account.data_account_type_non_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_payments_on_account_tangible"
"account_1400","1400","Osuudet saman konsernin yrityksissä","account.data_account_type_non_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_shares_in_group_companies"
"account_1410","1410","Saamiset saman konsernin yrityksiltä","account.data_account_type_non_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_receivables_from_group_long-term"
"account_1420","1420","Osuudet omistusyhteysyrityksissä","account.data_account_type_non_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_shares_in_associated_companies"
"account_1430","1430","Pitkäaikaiset saamiset omistusyhteysyrityksiltä","account.data_account_type_non_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_receivables_from_associated_companies_non-current"
"account_1440","1440","Muut osakkeet ja osuudet","account.data_account_type_non_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_shares_and_participations"
"account_1470","1470","Muut pitkäaikaiset saamiset","account.data_account_type_non_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_long-term_receivables"
"account_1500","1500","Aineet ja tarvikkeet","account.data_account_type_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_materials_and_supplies_inventories"
"account_1510","1510","Keskeneräiset tuotteet","account.data_account_type_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_work_in_progress"
"account_1520","1520","Valmiit tuotteet","account.data_account_type_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_finished_goods_inventories"
"account_1530","1530","Tavarat","account.data_account_type_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_goods_in_transit"
"account_1540","1540","Muu vaihto-omaisuus","account.data_account_type_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_current_assets"
"account_1550","1550","Ennakkomaksut vaihto-omaisuudesta","account.data_account_type_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_advance_payments_for_property"
"account_1600","1600","Pitkäaikaiset myyntisaamiset","account.data_account_type_receivable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_trade_receivables_long-term"
"account_1630","1630","Saamiset saman konsernin yrityksiltä, pitkäaikaiset","account.data_account_type_receivable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_receivables_from_group_long-term"
"account_1640","1640","Saamiset omistusyhteysyrityksiltä, pitkäaikaiset","account.data_account_type_receivable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_receivables_from_associated_companies_non-current"
"account_1650","1650","Lainasaamiset, pitkäaikaiset","account.data_account_type_receivable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_loan_receivables_long-term"
"account_1660","1660","Muut saamiset, pitkäaikaiset","account.data_account_type_receivable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_other_long-term_receivables"
"account_1670","1670","Pitkäaikaiset maksamattomat osakkeet / osuudet","account.data_account_type_receivable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_unpaid_shares_of_long-term"
"account_1680","1680","Pitkäaikaiset siirtosaamiset","account.data_account_type_receivable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_prepaid_expenses_and_long-term"
"account_1690","1690","Pitkäaikaiset laskennalliset verosaamiset","account.data_account_type_receivable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_deferred_tax_assets_are_long-term"
"account_1700","1700","Myyntisaamiset","account.data_account_type_receivable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_accounts_receivable"
"account_1701","1701","Myyntisaamiset (myyntipiste)","account.data_account_type_receivable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_accounts_receivable"
"account_1710","1710","Myyntisaamiset vanha järjestelmä","account.data_account_type_receivable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_accounts_receivable"
"account_1720","1720","Korttisaamiset","account.data_account_type_receivable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_accounts_receivable"
"account_1730","1730","Saamiset saman konsernin yrityksiltä, lyhytaikaiset","account.data_account_type_receivable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_receivables_from_group"
"account_1740","1740","Saamiset omistusyhteysyrityksiltä, lyhytaikaiset","account.data_account_type_receivable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_receivables_from_associated_companies"
"account_1750","1750","Lainasaamiset, lyhytaikaiset","account.data_account_type_receivable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_loans"
"account_1760","1760","Muut saamiset, lyhytaikaiset","account.data_account_type_receivable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_other_receivables"
"account_1765","1765","ALV-saamiset","account.data_account_type_current_assets","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_other_receivables"
"account_1777","1777","Selvittelytili","account.data_account_type_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_receivables"
"account_1780","1780","Maksamattomat osakkeet / osuudet, lyhytaikaiset","account.data_account_type_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_unpaid_contributions"
"account_1800","1800","Siirtosaamiset, lyhytaikaiset","account.data_account_type_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_prepayments_and_accrued_income"
"account_1850","1850","Laskennalliset verosaamiset","account.data_account_type_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_deferred_tax_assets"
"account_1860","1860","Osuudet saman konsernin yrityksissä","account.data_account_type_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_shares_in_group_companies"
"account_1880","1880","Muut osakkeet ja osuudet (rahoitusomaisuus)","account.data_account_type_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_shares_and_marketable_securities"
"account_1890","1890","Muut rahoitusarvopaperit","account.data_account_type_current_assets","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_financial_assets"
"account_1900","1900","Rahat / käteisvarat","account.data_account_type_liquidity","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_liquidity"
"account_1910","1910","Pankkisaamiset","account.data_account_type_liquidity","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_liquidity"
"account_1920","1920","Pankkisaamiset, pankki2","account.data_account_type_liquidity","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_liquidity"
"account_1930","1930","Pankkisaamiset, pankki3","account.data_account_type_liquidity","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_liquidity"
"account_1940","1940","Pankkisaamiset, pankki4","account.data_account_type_liquidity","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_liquidity"
"account_1990","1990","Rahansiirrot ja täsmäytykset","account.data_account_type_liquidity","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_liquidity"
"account_2000","2000","Osakepääoma","account.data_account_type_equity","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_stock_capital"
"account_2010","2010","Osakeanti","account.data_account_type_equity","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_stock_capital"
"account_2020","2020","Ylikurssirahasto","account.data_account_type_equity","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_share_premium_account"
"account_2030","2030","Arvonkorotusrahasto (oy)","account.data_account_type_equity","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_revaluation_reserve"
"account_2050","2050","Vararahasto (oy)","account.data_account_type_equity","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_reserve"
"account_2060","2060","Sijoitetun vapaan oman pääoman rahasto","account.data_account_type_equity","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_the_invested_unrestricted_equity_fund"
"account_2070","2070","Yhtiöjärjestyksen mukainen rahasto","account.data_account_type_equity","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_the_articles_of_association_or_fund_under_the_rules"
"account_2080","2080","Muut rahastot (oy)","account.data_account_type_equity","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_reserves"
"account_2100","2100","Osuuspääoma","account.data_account_type_equity","l10n_fi.fi_chart_template","False",
"account_2110","2110","Arvonkorotusrahasto (osk)","account.data_account_type_equity","l10n_fi.fi_chart_template","False",
"account_2120","2120","Vararahasto (osk)","account.data_account_type_equity","l10n_fi.fi_chart_template","False",
"account_2130","2130","Sääntöjen mukainen rahasto","account.data_account_type_equity","l10n_fi.fi_chart_template","False",
"account_2140","2140","Muut rahastot (osk)","account.data_account_type_equity","l10n_fi.fi_chart_template","False",
"account_2150","2150","Pääomapanokset (ay)","account.data_account_type_equity","l10n_fi.fi_chart_template","False",
"account_2160","2160","Arvonkorotusrahasto (ay)","account.data_account_type_equity","l10n_fi.fi_chart_template","False",
"account_2170","2170","Pääomapanokset (ky)","account.data_account_type_equity","l10n_fi.fi_chart_template","False",
"account_2190","2190","Arvonkorotusrahasto (ky)","account.data_account_type_equity","l10n_fi.fi_chart_template","False",
"account_2200","2200","Peruspääoma","account.data_account_type_equity","l10n_fi.fi_chart_template","False",
"account_2210","2210","Arvonkorotusrahasto (tmi)","account.data_account_type_equity","l10n_fi.fi_chart_template","False",
"account_2250","2250","Edellisten tilikausien voitto / tappio","account.data_account_type_equity","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_retained_earnings"
"account_2330","2330","Pääomavajaus edellisiltä tilikausilta","account.data_account_type_equity","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_retained_earnings"
"account_2340","2340","Yksityistilit tilikaudella","account.data_account_type_equity","l10n_fi.fi_chart_template","False",
"account_2370","2370","Tilikauden tulos","account.data_unaffected_earnings","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_profit_for_the_period"
"account_2390","2390","Vähemmistöosuus","account.data_account_type_non_current_liabilities","l10n_fi.fi_chart_template","False",
"account_2400","2400","Poistoero","account.data_account_type_non_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_depreciation_difference"
"account_2450","2450","Verotusperusteiset varaukset","account.data_account_type_non_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_tax_based_reservations"
"account_2500","2500","Eläkevaraukset","account.data_account_type_non_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_provisions_for_pensions"
"account_2530","2530","Verovaraukset","account.data_account_type_non_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_tax_provisions"
"account_2550","2550","Muut pakolliset varaukset","account.data_account_type_non_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_provisions"
"account_2590","2590","Konsernireservi","account.data_account_type_non_current_liabilities","l10n_fi.fi_chart_template","False",
"account_2600","2600","Pääomalainat, pitkäaikaiset","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_subordinated_loans_long-term"
"account_2610","2610","Joukkovelkakirjalainat, pitkäaikaiset","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_bonds_are_long-term"
"account_2620","2620","Pitkäaikaiset vaihtovelkakirjalainat","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_convertible_bonds_long-term"
"account_2630","2630","Lainat rahoituslaitoksilta","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_loans_from_financial_institutions_long-term"
"account_2650","2650","Pitkäaikaiset takaisinlainat työeläkelaitoksilta","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_pension_loans_long-term"
"account_2660","2660","Pitkäaikaiset saadut ennakot","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_advances_received_long-term"
"account_2670","2670","Ostovelat, pitkäaikaiset","account.data_account_type_payable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_accounts_payable-current"
"account_2690","2690","Pitkäaikaiset rahoitusvekselit","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_bills_of_long-term"
"account_2700","2700","Velat saman konsernin yrityksille, pitkäaikaiset","account.data_account_type_payable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_consolidated_long-term_liabilities"
"account_2710","2710","Velat omistusyhteysyrityksille, pitkäaikaiset","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_payables_to_associated_companies_non-current"
"account_2720","2720","Muut velat, pitkäaikaiset","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_long-term_liabilities"
"account_2750","2750","Pitkäaikaiset siirtovelat","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_accrued_long-term"
"account_2770","2770","Pitkäaikaiset laskennalliset verovelat","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_deferred_tax_liabilities_are_long-term"
"account_2800","2800","Pääomalainat, lyhytaikaiset","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_capital_loans_are_short-term"
"account_2810","2810","Joukkovelkakirjalainat","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_bonds_short-term"
"account_2820","2820","Vaihtovelkakirjalainat","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_convertible_bonds_short-term"
"account_2830","2830","Lainat rahoituslaitoksilta","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_loans_from_financial_institutions_short-term"
"account_2850","2850","Työeläkelaitoksien takaisinlainojen lyhennyserät","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_pension_loans_short-term"
"account_2860","2860","Saadut ennakot","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_advances_received_short-term"
"account_2870","2870","Ostovelat","account.data_account_type_payable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_accounts_payable"
"account_2880","2880","Maksuliikennetili","account.data_account_type_payable","l10n_fi.fi_chart_template","True","l10n_fi.account_tag_accounts_payable"
"account_2890","2890","Rahoitusvekselit","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_bills_of_short-term"
"account_2900","2900","Velat saman konsernin yrityksille","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_group_short-term_liabilities"
"account_2910","2910","Velat omistusyhteysyrityksille","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_payables_to_associated_companies"
"account_2920","2920","Ennakonpidätys- ja stm-velka","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_short-term_liabilities"
"account_2930","2930","Suoritettava arvonlisävero","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_short-term_liabilities"
"account_2935","2935","OmaVero-tapahtumat","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_short-term_liabilities"
"account_2940","2940","Muut velat","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_short-term_liabilities"
"account_2950","2950","Palkkojen siirtovelat","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_accrued_liabilities_short-term"
"account_2951","2951","Muut siirtovelat","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_accrued_liabilities_short-term"
"account_2952","2952","Palkkojen jaksotus","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_accrued_liabilities_short-term"
"account_2980","2980","Laskennalliset verovelat","account.data_account_type_current_liabilities","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_deferred_tax_liabilities_are_short-term"
"account_3000","3000","Myynti","account.data_account_type_revenue","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_sales"
"account_3200","3200","Oheispalvelut","account.data_account_type_revenue","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_sales"
"account_3250","3250","Toimitusveloitukset ja osamaksulisät","account.data_account_type_revenue","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_sales"
"account_3300","3300","Komissiokauppa ja agentuuri","account.data_account_type_revenue","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_sales"
"account_3330","3330","Tavaramyynti Ahvenanmaalle","account.data_account_type_revenue","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_sales"
"account_3350","3350","Yhteisömyynti","account.data_account_type_revenue","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_sales"
"account_3380","3380","Myynti yhteisön ulkopuolelle","account.data_account_type_revenue","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_sales"
"account_3400","3400","Myynti, käytetyt tavarat ja taide-, keräily- ja antiikkiesineet","account.data_account_type_revenue","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_sales"
"account_3440","3440","Myynti, arvopaperit ja kiinteistöt","account.data_account_type_revenue","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_sales"
"account_3500","3500","Myynnin alennukset","account.data_account_type_revenue","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_sales"
"account_3550","3550","Välilliset verot","account.data_account_type_revenue","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_sales_adjustment_items"
"account_3570","3570","Tulonsiirto- ja läpikulkuerät","account.data_account_type_revenue","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_sales_adjustment_items"
"account_3580","3580","Myynnin valuuttakurssierot","account.data_account_type_revenue","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_sales_adjustment_items"
"account_3590","3590","Muut myynnin oikaisuerät","account.data_account_type_revenue","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_sales_adjustment_items"
"account_3600","3600","Valmiiden ja keskeneräisten tuotteiden varastojen lisäys (+) tai vähennys (-)","account.data_account_type_direct_costs","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_manufacturing_warehouse_change"
"account_3630","3630","Valmistus omaan käyttöön","account.data_account_type_direct_costs","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_manufacturing_own_use"
"account_3650","3650","Myyntivoitot pysyvistä vastaavista","account.data_account_type_other_income","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_income_other"
"account_3700","3700","Leasinghyvitykset","account.data_account_type_other_income","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_income_other"
"account_3710","3710","Keskeytys- ym. vakuuskorvaukset","account.data_account_type_other_income","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_income_other"
"account_3750","3750","Vuokratuotot","account.data_account_type_other_income","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_income_other"
"account_3800","3800","Saadut avustukset ja tuet","account.data_account_type_other_income","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_income_other"
"account_3850","3850","Palvelutuotot","account.data_account_type_other_income","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_income_other"
"account_3900","3900","Palkkiot ja korvaukset","account.data_account_type_other_income","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_income_other"
"account_3980","3980","Muut tuotot","account.data_account_type_other_income","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_income_other"
"account_4000","4000","Ostot","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_purchases"
"account_4090","4090","Tavaraostot Ahvenanmaalta","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_purchases"
"account_4110","4110","Yhteisöhankinnat","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_purchases"
"account_4130","4130","Tuontiostot yhteisön ulkopuolelta","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_purchases"
"account_4150","4150","Ostot, käytetyt tavarat ja taide-, keräily- ja antiikkiesineet","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_purchases"
"account_4200","4200","Ostot, arvopaperit ja kiinteistöt","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_purchases"
"account_4230","4230","Ostojen alennukset","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_purchases_adjustment_items"
"account_4260","4260","Palautetut tavarat","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_purchases_adjustment_items"
"account_4270","4270","Saadut vahingonkorvaukset ja avustukset","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_purchases_adjustment_items"
"account_4290","4290","Rahdit, huolinta ja muut hankintakulut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_purchases_adjustment_items"
"account_4340","4340","Siirrot muuhun kuin myyntitarkoitukseen","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_purchases_adjustment_items"
"account_4370","4370","Ostojen valuuttakurssierot","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_purchases_adjustment_items"
"account_4380","4380","Muut ostojen oikaisuerät","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_purchases_adjustment_items"
"account_4400","4400","Varastojen lisäys (+) tai vähennys (-)","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_change_in_inventories"
"account_4450","4450","Alihankinta","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_external_services"
"account_4480","4480","Vuokrattu työvoima","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_external_services"
"account_4490","4490","Muut ulkopuoliset palvelut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_external_services"
"account_5000","5000","Työssäoloajan normaalipalkat","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_wages_and_salaries_in_production"
"account_5100","5100","Lisät ja korvaukset","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_wages_and_salaries_other"
"account_5200","5200","Palkkiot","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_wages_and_salaries_other"
"account_5300","5300","Loma-ajan ja sosiaalipalkat","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_wages_and_salaries_other"
"account_5330","5330","Palkkojen jaksotus","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_wages_and_salaries_other"
"account_5400","5400","Luontoisedut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_wages_and_salaries_other"
"account_5470","5470","Saadut korvaukset palkoista","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_wages_and_salaries_other"
"account_5600","5600","Johdon palkat ja palkkiot","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_wages_and_salaries_other"
"account_5700","5700","Johdon luontoisedut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_wages_and_salaries_other"
"account_5770","5770","Saadut korvaukset johdon palkoista","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_wages_and_salaries_other"
"account_5800","5800","Osakkaiden ja omaisten palkat ja palkkiot","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_wages_and_salaries_other"
"account_5900","5900","Osakkaiden ja omaisten luontoisedut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_wages_and_salaries_other"
"account_5960","5960","Saadut korvaukset osakkaiden ja omaisten palkoista","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_wages_and_salaries_other"
"account_5990","5990","Luontoisetujen vastatili","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_wages_and_salaries_other"
"account_6000","6000","Maksetut eläkkeet","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_pension_costs_of_production"
"account_6100","6100","Eläkevakuutusmaksut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_pension_costs_other"
"account_6290","6290","Tilikauden aikainen jaksotus","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_additional_expenses_other"
"account_6300","6300","Sosiaaliturvamaksut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_additional_expenses_other"
"account_6400","6400","Pakolliset vakuutusmaksut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_additional_expenses_other"
"account_6500","6500","Muut henkilöstön vakuutusmaksut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_additional_expenses_other"
"account_6690","6690","Tilikauden aikainen jaksotus","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_additional_expenses_other"
"account_6800","6800","Suunnitelman mukaiset poistot","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_depreciation_according_to_plan"
"account_6930","6930","Konserniliikearvon poisto ja konsenireservin vähennys","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_change_in_depreciation_difference"
"account_6950","6950","Arvonalentumiset pysyvien vastaavien hyödykkeistä","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_impairment_of_fixed_assets"
"account_6990","6990","Vaihtuvien vastaavien poikkeukselliset arvonalentumiset","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_impairment_losses_financial_assets_current_assets"
"account_7000","7000","Vapaaehtoiset henkilösivukulut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_personnel_other"
"account_7200","7200","Toimitilakulut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_premises_costs"
"account_7230","7230","Toimitilavuokrat","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_premises_costs"
"account_7250","7250","Varastovuokrat","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_premises_costs"
"account_7270","7270","Autotalli- ja autopaikkavuokrat","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_premises_costs"
"account_7500","7500","Ajoneuvokulut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_vehicle_expenses"
"account_7520","7520","Ajoneuvovuokrat","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_vehicle_expenses"
"account_7640","7640","Atk-laite ja -ohjelmakulut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_it_expenses"
"account_7650","7650","Atk-laite ja ohjelm. vuokrat","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_it_expenses"
"account_7710","7710","Kone- ja kalustokulut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_in_machinery_and_equipment_expenses"
"account_7720","7720","Kone- ja kalustovuokrat","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_in_machinery_and_equipment_expenses"
"account_7800","7800","Matkakulut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_travelling"
"account_7950","7950","Edustuskulut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_representation"
"account_8000","8000","Myyntikulut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_selling_expenses"
"account_8050","8050","Markkinointikulut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_marketing_expenses"
"account_8300","8300","Tutkimus- ja kehityskulut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_research_and_development"
"account_8370","8370","Ostetut hallintopalvelut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_administrative_services"
"account_8450","8450","Muut hallintokulut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_administrative_expenses"
"account_8451","8451","Puhelin- ja tietoliikennekulut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_it_expenses"
"account_8452","8452","Vakuutukset ja vahingonkorvaukset","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_fixed_costs"
"account_8453","8453","Toimisto- ja hallintokulut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_administrative_expenses"
"account_8455","8455","Muut hallintokulut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_administrative_expenses"
"account_8700","8700","Muut liikekulut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_fixed_costs"
"account_8730","8730","Myynnin luottotappiot","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_fixed_costs"
"account_8790","8790","Fuusiotappio","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_fixed_costs"
"account_8800","8800","Vähennyskelvottomat liikekulut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_fixed_costs"
"account_8850","8850","Käyttöomaisuuden luovutustappiot","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_fixed_costs"
"account_8890","8890","Täsmäytyserot","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_fixed_costs"
"account_8990","8990","Osuus osakkuusyritysten tuloksesta","account.data_account_type_expenses","l10n_fi.fi_chart_template","False",
"account_9000","9000","Tuotot osuuksista saman konsernin yrityksissä","account.data_account_type_other_income","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_income_from_shares_in_group"
"account_9030","9030","Osuus osakkuusyritysten voitosta (tappiosta)","account.data_account_type_other_income","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_income_from_associated_company"
"account_9040","9040","Tuotot osuuksista omistusyhteysyrityksissä","account.data_account_type_other_income","l10n_fi.fi_chart_template","False",
"account_9070","9070","Tuotot osuuksista muissa omistusyhteysyrityksissä","account.data_account_type_other_income","l10n_fi.fi_chart_template","False",
"account_9080","9080","Sijoitustuotot pysyvien vastaavien sijoituksista, konserni","account.data_account_type_other_income","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_investment_income_other_current_assets_group"
"account_9090","9090","Sijoitustuotot pysyvien vastaavien sijoituksista, muut","account.data_account_type_other_income","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_investment_income_from_fixed_assets"
"account_9150","9150","Muut korko- ja rahoitustuotot, konserni","account.data_account_type_other_income","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_interest_and_financial_income_from_group"
"account_9160","9160","Muut korko- ja rahoitustuotot, muut","account.data_account_type_other_income","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_interest_and_financial_income"
"account_9300","9300","Arvonalentumiset pysyvien vastaavien sijoituksista","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_impairment_losses_on_investments_in_fixed_assets"
"account_9370","9370","Arvonalentumiset vaihtuvien vastaavien rahoitusarvopapereista","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_impairment_losses_financial_assets_current_assets"
"account_9420","9420","Korkokulut ja muut rahoituskulut, konserni","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_financial_expenses_group"
"account_9440","9440","Korkokulut ja muut rahoituskulut, muut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_other_financial_expenses"
"account_9800","9800","Poistoeron lisäys (-) tai vähennys (+)","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_change_in_depreciation_difference"
"account_9840","9840","Verotusperusteisten varausten lisäys (-) tai vähennys (+)","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_change_in_provisions"
"account_9850","9850","Konserniavustukset, saadut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_group_contribution"
"account_9860","9860","Konserniavustukset, maksetut","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_group_contribution"
"account_9900","9900","Tilikauden ja aikaisempien tilikausien verot","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_taxes_for_the_period"
"account_9970","9970","Laskennalliset verot","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_taxes_for_the_period"
"account_9980","9980","Muut välittömät verot","account.data_account_type_expenses","l10n_fi.fi_chart_template","False","l10n_fi.account_tag_taxes_for_the_period"
"account_9990","9990","Vähemmistöosuudet","account.data_account_type_expenses","l10n_fi.fi_chart_template","False",

```

## File: data\account_account_tag_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <!-- Tags for accounts -->
    <record id="account_tag_development_costs" model="account.account.tag">
        <field name="name">Tase: Kehittämismenot</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_immaterial_rights" model="account.account.tag">
        <field name="name">Tase: Aineettomat oikeudet</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_goodwill" model="account.account.tag">
        <field name="name">Tase: Liikearvo</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_long_term_expenses" model="account.account.tag">
        <field name="name">Tase: Muut pitkävaikutteiset menot</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_advance_payments_intangible" model="account.account.tag">
        <field name="name">Tase: Ennakkomaksut aineettomat</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_land_and_water_areas_owned" model="account.account.tag">
        <field name="name">Tase: Maa- ja vesialueet omistetut</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_land_and_water_rights_lease" model="account.account.tag">
        <field name="name">Tase: Maa- ja vesialueet vuokraoikeudet</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_buildings_owned" model="account.account.tag">
        <field name="name">Tase: Rakennukset omistetut</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_building_leases" model="account.account.tag">
        <field name="name">Tase: Rakennukset vuokraoikeudet</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_machines_and_hardware" model="account.account.tag">
        <field name="name">Tase: Koneet ja kalusto</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_tangible_assets" model="account.account.tag">
        <field name="name">Tase: Muut aineelliset hyödykkeet</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_payments_on_account_tangible" model="account.account.tag">
        <field name="name">Tase: Ennakkomaksut aineelliset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_shares_in_group_companies" model="account.account.tag">
        <field name="name">Tase: Osuudet saman konsernin yrityksissä</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_receivables_from_associated_companies_investments" model="account.account.tag">
        <field name="name">Tase: Saamiset saman konsernin yrityksiltä sijoitukset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_shares_in_associated_companies" model="account.account.tag">
        <field name="name">Tase: Osuudet omistusyhteysyrityksissä</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_receivables_from_associated_companies_investments" model="account.account.tag">
        <field name="name">Tase: Saamiset omistusyhteysyrityksiltä sijoitukset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_shares_and_participations" model="account.account.tag">
        <field name="name">Tase: Muut osakkeet ja osuudet</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_receivables_investments" model="account.account.tag">
        <field name="name">Tase: Muut saamiset sijoitukset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_materials_and_supplies_inventories" model="account.account.tag">
        <field name="name">Tase: Aineet ja tarvikkeet vaihto-omaisuus</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_work_in_progress" model="account.account.tag">
        <field name="name">Tase: Keskeneräiset tuotteet</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_finished_goods_inventories" model="account.account.tag">
        <field name="name">Tase: Valmiit tuotteet ja tavarat vaihto-omaisuus</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_goods_in_transit" model="account.account.tag">
        <field name="name">Tase: Matkalla olevat tavarat</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_current_assets" model="account.account.tag">
        <field name="name">Tase: Muu vaihto-omaisuus</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_advance_payments_for_property" model="account.account.tag">
        <field name="name">Tase: Ennakkomaksut vaihto-omaisuus</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_trade_receivables_long-term" model="account.account.tag">
        <field name="name">Tase: Myyntisaamiset pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_receivables_from_group_long-term" model="account.account.tag">
        <field name="name">Tase: Saamiset konserni pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_receivables_from_associated_companies_non-current" model="account.account.tag">
        <field name="name">Tase: Saamiset omistusyhteysyritykset pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_loan_receivables_long-term" model="account.account.tag">
        <field name="name">Tase: Lainasaamiset pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_deferred_tax_assets_are_long-term" model="account.account.tag">
        <field name="name">Tase: Laskennalliset verosaamiset pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_long-term_receivables" model="account.account.tag">
        <field name="name">Tase: Muut saamiset pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_unpaid_shares_of_long-term" model="account.account.tag">
        <field name="name">Tase: Maksamattomat osuudet pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_prepaid_expenses_and_long-term" model="account.account.tag">
        <field name="name">Tase: Siirtosaamiset pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_accounts_receivable" model="account.account.tag">
        <field name="name">Tase: Myyntisaamiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_trade_group" model="account.account.tag">
        <field name="name">Tase: Myyntisaamiset konserni</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_receivables_from_group" model="account.account.tag">
        <field name="name">Tase: Saamiset konserni</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_accounts_receivable_from_associated_companies" model="account.account.tag">
        <field name="name">Tase: Myyntisaamiset omistusyhteysyritykset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_receivables_from_associated_companies" model="account.account.tag">
        <field name="name">Tase: Saamiset omistusyhteysyritykset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_loans" model="account.account.tag">
        <field name="name">Tase: Lainasaamiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_deferred_tax_assets" model="account.account.tag">
        <field name="name">Tase: Laskennalliset verosaamiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_receivables" model="account.account.tag">
        <field name="name">Tase: Muut saamiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_unpaid_contributions" model="account.account.tag">
        <field name="name">Tase: Maksamattomat osuudet</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_prepayments_and_accrued_income" model="account.account.tag">
        <field name="name">Tase: Siirtosaamiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_shares_in_the_consolidated_financial_securities" model="account.account.tag">
        <field name="name">Tase: Osuudet konserni rahoitusarvopaperit</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_shares_and_marketable_securities" model="account.account.tag">
        <field name="name">Tase: Muut osakkeet ja osuudet rahoitusarvopaperit</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_financial_assets" model="account.account.tag">
        <field name="name">Tase: Muut rahoitusarvopaperit</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_liquidity" model="account.account.tag">
        <field name="name">Tase: Rahat ja pankkisaamiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_stock_capital" model="account.account.tag">
        <field name="name">Tase: Osake-, osuus- tai muu vastaava pääoma</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_share_premium_account" model="account.account.tag">
        <field name="name">Tase: Ylikurssirahasto</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_revaluation_reserve" model="account.account.tag">
        <field name="name">Tase: Arvonkorotusrahasto</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_the_invested_unrestricted_equity_fund" model="account.account.tag">
        <field name="name">Tase: Sijoitetun vapaan oman pääoman rahasto</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_reserve" model="account.account.tag">
        <field name="name">Tase: Vararahasto</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_the_articles_of_association_or_fund_under_the_rules" model="account.account.tag">
        <field name="name">Tase: Yhtiöjärjestyksen tai sääntöjen mukainen rahasto</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_the_fair_value_reserve" model="account.account.tag">
        <field name="name">Tase: Käyvän arvon rahasto</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_reserves" model="account.account.tag">
        <field name="name">Tase: Muut rahastot</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_retained_earnings" model="account.account.tag">
        <field name="name">Tase: Edellisten tilikausien tulos</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_profit_for_the_period" model="account.account.tag">
        <field name="name">Tase: Tilikauden tulos</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_depreciation_difference" model="account.account.tag">
        <field name="name">Tase: Poistoero</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_tax_based_reservations" model="account.account.tag">
        <field name="name">Tase: Verotusperäiset varaukset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_provisions_for_pensions" model="account.account.tag">
        <field name="name">Tase: Eläkevaraukset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_tax_provisions" model="account.account.tag">
        <field name="name">Tase: Verovaraukset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_provisions" model="account.account.tag">
        <field name="name">Tase: Muut pakolliset varaukset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_subordinated_loans_long-term" model="account.account.tag">
        <field name="name">Tase: Pääomalainat pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_bonds_are_long-term" model="account.account.tag">
        <field name="name">Tase: Joukkovelkakirjalainat pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_convertible_bonds_long-term" model="account.account.tag">
        <field name="name">Tase: Vaihtovelkakirjalainat pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_loans_from_financial_institutions_long-term" model="account.account.tag">
        <field name="name">Tase: Lainat rahoituslaitoksilta pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_pension_loans_long-term" model="account.account.tag">
        <field name="name">Tase: Eläkelainat pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_advances_received_long-term" model="account.account.tag">
        <field name="name">Tase: Saadut ennakot pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_accounts_payable-current" model="account.account.tag">
        <field name="name">Tase: Ostovelat pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_bills_of_long-term" model="account.account.tag">
        <field name="name">Tase: Rahoitusvekselit pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_consolidated_long-term_liabilities" model="account.account.tag">
        <field name="name">Tase: Velat konserni pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_payables_to_associated_companies_non-current" model="account.account.tag">
        <field name="name">Tase: Velat omistusyhteysyritykset pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_deferred_tax_liabilities_are_long-term" model="account.account.tag">
        <field name="name">Tase: Laskennalliset verovelat pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_long-term_liabilities" model="account.account.tag">
        <field name="name">Tase: Muut velat pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_accrued_long-term" model="account.account.tag">
        <field name="name">Tase: Siirtovelat pitkäaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_capital_loans_are_short-term" model="account.account.tag">
        <field name="name">Tase: Pääomalainat lyhytaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_bonds_short-term" model="account.account.tag">
        <field name="name">Tase: Joukkovelkakirjalainat lyhytaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_convertible_bonds_short-term" model="account.account.tag">
        <field name="name">Tase: Vaihtovelkakirjalainat lyhytaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_loans_from_financial_institutions_short-term" model="account.account.tag">
        <field name="name">Tase: Lainat rahoituslaitoksilta lyhytaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_pension_loans_short-term" model="account.account.tag">
        <field name="name">Tase: Eläkelainat lyhytaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_advances_received_short-term" model="account.account.tag">
        <field name="name">Tase: Saadut ennakot lyhytaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_accounts_payable" model="account.account.tag">
        <field name="name">Tase: Ostovelat</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_bills_of_short-term" model="account.account.tag">
        <field name="name">Tase: Rahoitusvekselit lyhytaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_trade_payables_to_group" model="account.account.tag">
        <field name="name">Tase: Ostovelat konserni</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_group_short-term_liabilities" model="account.account.tag">
        <field name="name">Tase: Velat konserni lyhytaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_trade_payables_to_associated_companies" model="account.account.tag">
        <field name="name">Tase: Ostovelat omistusyhteysyritykset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_payables_to_associated_companies" model="account.account.tag">
        <field name="name">Tase: Velat omistusyhteysyritykset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_deferred_tax_liabilities_are_short-term" model="account.account.tag">
        <field name="name">Tase: Laskennalliset verovelat lyhytaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_short-term_liabilities" model="account.account.tag">
        <field name="name">Tase: Muut velat lyhytaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_accrued_liabilities_short-term" model="account.account.tag">
        <field name="name">Tase: Siirtovelat lyhytaikaiset</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_sales" model="account.account.tag">
        <field name="name">Tulos: Myynti</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_sales_adjustment_items" model="account.account.tag">
        <field name="name">Tulos: Myynnin oikaisuerät</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_manufacturing_warehouse_change" model="account.account.tag">
        <field name="name">Tulos: Valmiiden ja keskeneräisten tuotteiden varastojen muutos</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_manufacturing_own_use" model="account.account.tag">
        <field name="name">Tulos: Valmistus omaan käyttöön</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_income_other" model="account.account.tag">
        <field name="name">Tulos: Liiketoiminnan muut tuotot</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_purchases" model="account.account.tag">
        <field name="name">Tulos: Ostot</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_purchases_adjustment_items" model="account.account.tag">
        <field name="name">Tulos: Ostojen oikaisuerät</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_change_in_inventories" model="account.account.tag">
        <field name="name">Tulos: Varastojen muutos</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_external_services" model="account.account.tag">
        <field name="name">Tulos: Ulkopuoliset palvelut</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_wages_and_salaries_in_production" model="account.account.tag">
        <field name="name">Tulos: Palkat ja palkkiot tuotanto</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_wages_and_salaries_other" model="account.account.tag">
        <field name="name">Tulos: Palkat ja palkkiot muut</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_pension_costs_of_production" model="account.account.tag">
        <field name="name">Tulos: Eläkekulut tuotanto</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_pension_costs_other" model="account.account.tag">
        <field name="name">Tulos: Eläkekulut muut</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_additional_expenses_production" model="account.account.tag">
        <field name="name">Tulos: Muut sivukulut tuotanto</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_additional_expenses_other" model="account.account.tag">
        <field name="name">Tulos: Muut sivukulut muut</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_depreciation_according_to_plan" model="account.account.tag">
        <field name="name">Tulos: Suunnitelman mukaiset poistot</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_impairment_of_fixed_assets" model="account.account.tag">
        <field name="name">Tulos: Arvonalentumiset pysyvät vastaavat</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_impairment_losses_financial_assets_current_assets" model="account.account.tag">
        <field name="name">Tulos: Arvonalentumiset vaihtuvat vastaavat</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_personnel_other" model="account.account.tag">
        <field name="name">Tulos: Henkilöstö muut</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_premises_costs" model="account.account.tag">
        <field name="name">Tulos: Toimitilakulut</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_vehicle_expenses" model="account.account.tag">
        <field name="name">Tulos: Ajoneuvokulut</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_it_expenses" model="account.account.tag">
        <field name="name">Tulos: IT-kulut</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_in_machinery_and_equipment_expenses" model="account.account.tag">
        <field name="name">Tulos: Kone- ja kalustokulut</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_travelling" model="account.account.tag">
        <field name="name">Tulos: Matkat</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_representation" model="account.account.tag">
        <field name="name">Tulos: Edustus</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_selling_expenses" model="account.account.tag">
        <field name="name">Tulos: Myyntikulut</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_marketing_expenses" model="account.account.tag">
        <field name="name">Tulos: Markkinointikulut</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_research_and_development" model="account.account.tag">
        <field name="name">Tulos: Tutkimus ja kehitys</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_administrative_services" model="account.account.tag">
        <field name="name">Tulos: Hallintopalvelut</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_administrative_expenses" model="account.account.tag">
        <field name="name">Tulos: Muut hallintokulut</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_fixed_costs" model="account.account.tag">
        <field name="name">Tulos: Muut kiinteät kulut</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_income_from_shares_in_group" model="account.account.tag">
        <field name="name">Tulos: Tuotot osuuksista konserni</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_income_from_associated_company" model="account.account.tag">
        <field name="name">Tulos: Tuotot osuuksista omistusyhteysyritys</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_investment_income_other_current_assets_group" model="account.account.tag">
        <field name="name">Tulos: Sijoitustuotot muut pysyvät vastaavat konserni</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_investment_income_from_fixed_assets" model="account.account.tag">
        <field name="name">Tulos: Muut sijoitustuotot pysyvät vastaavat</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_interest_and_financial_income_from_group" model="account.account.tag">
        <field name="name">Tulos: Korko ja rahoitustuotot konserni</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_interest_and_financial_income" model="account.account.tag">
        <field name="name">Tulos: Muut korko ja rahoitustuotot</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_impairment_losses_on_investments_in_fixed_assets" model="account.account.tag">
        <field name="name">Tulos: Arvonalentumiset sijoitukset pysyvät vastaavat</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_impairment_losses_financial_assets_current_assets" model="account.account.tag">
        <field name="name">Tulos: Arvonalentumiset rahoitusarvopaperit vaihtuvat vastaavat</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_financial_expenses_group" model="account.account.tag">
        <field name="name">Tulos: Rahoituskulut konserni</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_financial_expenses" model="account.account.tag">
        <field name="name">Tulos: Rahoituskulut muut</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_change_in_depreciation_difference" model="account.account.tag">
        <field name="name">Tulos: Poistoeron muutos</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_change_in_provisions" model="account.account.tag">
        <field name="name">Tulos: Varausten muutos</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_group_contribution" model="account.account.tag">
        <field name="name">Tulos: Konserniavustus</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_taxes_for_the_period" model="account.account.tag">
        <field name="name">Tulos: Tilikauden verot</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_deferred_taxes" model="account.account.tag">
        <field name="name">Tulos: Laskennalliset verot</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_tax" model="account.account.tag">
        <field name="name">Tulos: Muut välittömät verot</field>
        <field name="applicability">accounts</field>
    </record>
</odoo>

```

## File: data\account_chart_template_configuration_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <function model="account.chart.template" name="try_loading">
        <value eval="[ref('l10n_fi.fi_chart_template')]"/>
    </function>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_fi_statements_menu" name="Finland" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_user"/>
    <!-- Account Chart Template -->
    <record id="fi_chart_template" model="account.chart.template">
        <field name="name">Finnish Chart of Accounts</field>
        <field name="cash_account_code_prefix">1910</field>
        <field name="bank_account_code_prefix">1921</field>
        <field name="transfer_account_code_prefix">1950</field>
        <field name="code_digits">4</field>
        <field name="currency_id" ref="base.EUR"/>
    </record>
</odoo>

```

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Account Tax Group -->
        <record id="tax_group_24" model="account.tax.group">
            <field name="name">VAT 24%</field>
        </record>
        <record id="tax_group_14" model="account.tax.group">
            <field name="name">VAT 14%</field>
        </record>
        <record id="tax_group_10" model="account.tax.group">
            <field name="name">VAT 10%</field>
        </record>
        <record id="tax_group_0" model="account.tax.group">
            <field name="name">VAT 0%</field>
        </record>

    </data>
</odoo>

```

## File: data\account_fiscal_position_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- # Fiscal positions -->
    <record id="aland" model="account.fiscal.position.template">
        <field name="name">Aland</field>
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="country_id" ref="base.ax"/>
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="True"/>
        <field name="sequence">1</field>
    </record>

    <record id="finland" model="account.fiscal.position.template">
        <field name="name">Finland</field>
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="country_id" ref="base.fi"/>
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="True"/>
        <field name="sequence">3</field>
    </record>

    <record id="eu" model="account.fiscal.position.template">
        <field name="name">EU</field>
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="country_group_id" ref="base.europe"/>
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="True"/>
        <field name="sequence">4</field>
    </record>

    <record id="eu_no_vat" model="account.fiscal.position.template">
        <field name="name">EU no VAT</field>
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="country_group_id" ref="base.europe"/>
        <field name="auto_apply" eval="True"/>
        <field name="sequence">5</field>
    </record>

    <record id="non_eu" model="account.fiscal.position.template">
        <field name="name">Non EU</field>
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="auto_apply" eval="True"/>
        <field name="sequence">6</field>
    </record>

    <record id="construction" model="account.fiscal.position.template">
        <field name="name">Construction services + Scrap metal</field>
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">7</field>
    </record>

    <!-- # Fiscal Position Rules -->
    
    <!-- ## Aland -->
    <!-- ### Aland:sales -->
    <!-- #### Aland:sales:goods -->
    <record id="aland_sales_goods_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="aland"/>
        <field name="tax_src_id" ref="tax_dom_sales_goods_24"/>
        <field name="tax_dest_id" ref="aland_sales_0"/>
    </record>
    <record id="aland_sales_goods_14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="aland"/>
        <field name="tax_src_id" ref="tax_dom_sales_goods_14"/>
        <field name="tax_dest_id" ref="aland_sales_0"/>
    </record>
    <record id="aland_sales_goods_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="aland"/>
        <field name="tax_src_id" ref="tax_dom_sales_goods_10"/>
        <field name="tax_dest_id" ref="aland_sales_0"/>
    </record>
    <!-- #### Aland:sales:service -->
    <record id="aland_sales_service_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="aland"/>
        <field name="tax_src_id" ref="tax_dom_sales_service_24"/>
        <field name="tax_dest_id" ref="aland_sales_0"/>
    </record>
    <record id="aland_sales_service_14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="aland"/>
        <field name="tax_src_id" ref="tax_dom_sales_service_14"/>
        <field name="tax_dest_id" ref="aland_sales_0"/>
    </record>
    <record id="aland_sales_service_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="aland"/>
        <field name="tax_src_id" ref="tax_dom_sales_service_10"/>
        <field name="tax_dest_id" ref="aland_sales_0"/>
    </record>
    <!-- ### Aland:purchase -->
    <!-- #### Aland:purchase:goods -->
    <record id="aland_purchase_goods_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="aland"/>
        <field name="tax_src_id" ref="tax_dom_purchase_goods_24"/>
        <field name="tax_dest_id" ref="tax_non_eu_purchase_goods_24"/>
    </record>
    <record id="aland_purchase_goods_14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="aland"/>
        <field name="tax_src_id" ref="tax_dom_purchase_goods_14"/>
        <field name="tax_dest_id" ref="tax_non_eu_purchase_goods_14"/>
    </record>
    <record id="aland_purchase_goods_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="aland"/>
        <field name="tax_src_id" ref="tax_dom_purchase_goods_10"/>
        <field name="tax_dest_id" ref="tax_non_eu_purchase_goods_10"/>
    </record>
    <!-- #### Aland:purchase:services -->
    <record id="aland_purchase_service_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="aland"/>
        <field name="tax_src_id" ref="tax_dom_purchase_service_24"/>
        <field name="tax_dest_id" ref="tax_dom_purchase_0"/>
    </record>
    <record id="aland_purchase_service_14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="aland"/>
        <field name="tax_src_id" ref="tax_dom_purchase_service_14"/>
        <field name="tax_dest_id" ref="tax_dom_purchase_0"/>
    </record>
    <record id="aland_purchase_service_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="aland"/>
        <field name="tax_src_id" ref="tax_dom_purchase_service_10"/>
        <field name="tax_dest_id" ref="tax_dom_purchase_0"/>
    </record>

    <!-- ## EU -->
    <!-- ### EU:sales -->
    <!-- #### EU:sales:goods -->
    <record id="eu_sales_goods_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="eu"/>
        <field name="tax_src_id" ref="tax_dom_sales_goods_24"/>
        <field name="tax_dest_id" ref="tax_eu_sales_goods_0"/>
    </record>
    <record id="eu_sales_goods_14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="eu"/>
        <field name="tax_src_id" ref="tax_dom_sales_goods_14"/>
        <field name="tax_dest_id" ref="tax_eu_sales_goods_0"/>
    </record>
    <record id="eu_sales_goods_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="eu"/>
        <field name="tax_src_id" ref="tax_dom_sales_goods_10"/>
        <field name="tax_dest_id" ref="tax_eu_sales_goods_0"/>
    </record>
    <!-- #### EU:sales:service -->
    <record id="eu_sales_service_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="eu"/>
        <field name="tax_src_id" ref="tax_dom_sales_service_24"/>
        <field name="tax_dest_id" ref="tax_eu_sales_service_0"/>
    </record>
    <record id="eu_sales_service_14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="eu"/>
        <field name="tax_src_id" ref="tax_dom_sales_service_14"/>
        <field name="tax_dest_id" ref="tax_eu_sales_service_0"/>
    </record>
    <record id="eu_sales_service_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="eu"/>
        <field name="tax_src_id" ref="tax_dom_sales_service_10"/>
        <field name="tax_dest_id" ref="tax_eu_sales_service_0"/>
    </record>
    <!-- ### EU:purchase -->
    <!-- #### EU:purchase:goods -->
    <record id="eu_purchase_goods_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="eu"/>
        <field name="tax_src_id" ref="tax_dom_purchase_goods_24"/>
        <field name="tax_dest_id" ref="tax_eu_purchase_goods_24"/>
    </record>
    <record id="eu_purchase_goods_14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="eu"/>
        <field name="tax_src_id" ref="tax_dom_purchase_goods_14"/>
        <field name="tax_dest_id" ref="tax_eu_purchase_goods_14"/>
    </record>
    <record id="eu_purchase_goods_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="eu"/>
        <field name="tax_src_id" ref="tax_dom_purchase_goods_10"/>
        <field name="tax_dest_id" ref="tax_eu_purchase_goods_10"/>
    </record>
    <!-- #### EU:purchase:service -->
    <record id="eu_purchase_service_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="eu"/>
        <field name="tax_src_id" ref="tax_dom_purchase_service_24"/>
        <field name="tax_dest_id" ref="tax_eu_purchase_service_24"/>
    </record>
    <record id="eu_purchase_service_14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="eu"/>
        <field name="tax_src_id" ref="tax_dom_purchase_service_14"/>
        <field name="tax_dest_id" ref="tax_eu_purchase_service_14"/>
    </record>
    <record id="eu_purchase_service_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="eu"/>
        <field name="tax_src_id" ref="tax_dom_purchase_service_10"/>
        <field name="tax_dest_id" ref="tax_eu_purchase_service_10"/>
    </record>

    <!-- ## Non EU -->
    <!-- ### Non EU:sales -->
    <!-- #### Non EU:sales:goods -->
    <record id="non_eu_sale_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="non_eu"/>
        <field name="tax_src_id" ref="tax_dom_sales_goods_24"/>
        <field name="tax_dest_id" ref="vat0export"/>
    </record>
    <record id="non_eu_sale_14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="non_eu"/>
        <field name="tax_src_id" ref="tax_dom_sales_goods_14"/>
        <field name="tax_dest_id" ref="vat0export"/>
    </record>
    <record id="non_eu_sale_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="non_eu"/>
        <field name="tax_src_id" ref="tax_dom_sales_goods_10"/>
        <field name="tax_dest_id" ref="vat0export"/>
    </record>
    <!-- #### Non EU:sales:service -->
    <record id="non_eu_sale_service_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="non_eu"/>
        <field name="tax_src_id" ref="tax_dom_sales_service_24"/>
        <field name="tax_dest_id" ref="vat0export"/>
    </record>
    <record id="non_eu_sale_service_14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="non_eu"/>
        <field name="tax_src_id" ref="tax_dom_sales_service_14"/>
        <field name="tax_dest_id" ref="vat0export"/>
    </record>
    <record id="non_eu_sale_service_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="non_eu"/>
        <field name="tax_src_id" ref="tax_dom_sales_service_10"/>
        <field name="tax_dest_id" ref="vat0export"/>
    </record>
    <!-- ### Non EU:purchase -->
    <!-- #### Non EU:purchase:goods -->
    <record id="non_eu_purchase_goods_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="non_eu"/>
        <field name="tax_src_id" ref="tax_dom_purchase_goods_24"/>
        <field name="tax_dest_id" ref="tax_non_eu_purchase_goods_24"/>
    </record>
    <record id="non_eu_purchase_goods_14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="non_eu"/>
        <field name="tax_src_id" ref="tax_dom_purchase_goods_14"/>
        <field name="tax_dest_id" ref="tax_non_eu_purchase_goods_24"/>
    </record>
    <record id="non_eu_purchase_goods_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="non_eu"/>
        <field name="tax_src_id" ref="tax_dom_purchase_goods_10"/>
        <field name="tax_dest_id" ref="tax_non_eu_purchase_goods_24"/>
    </record>
    <!-- #### Non EU:purchase:services -->
    <record id="non_eu_purchase_service_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="non_eu"/>
        <field name="tax_src_id" ref="tax_dom_purchase_service_24"/>
        <field name="tax_dest_id" ref="tax_dom_purchase_0"/>
    </record>
    <record id="non_eu_purchase_service_14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="non_eu"/>
        <field name="tax_src_id" ref="tax_dom_purchase_service_14"/>
        <field name="tax_dest_id" ref="tax_dom_purchase_0"/>
    </record>
    <record id="non_eu_purchase_service_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="non_eu"/>
        <field name="tax_src_id" ref="tax_dom_purchase_service_10"/>
        <field name="tax_dest_id" ref="tax_dom_purchase_0"/>
    </record>
    <!-- #### Non EU:purchase:brutto -->
    <record id="non_eu_purchase_brutto_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="non_eu"/>
        <field name="tax_src_id" ref="tax_dom_purchase_brutto_24"/>
        <field name="tax_dest_id" ref="tax_dom_purchase_0"/>
    </record>
    <record id="non_eu_purchase_brutto_14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="non_eu"/>
        <field name="tax_src_id" ref="tax_dom_purchase_brutto_14"/>
        <field name="tax_dest_id" ref="tax_dom_purchase_0"/>
    </record>
    <record id="non_eu_purchase_brutto_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="non_eu"/>
        <field name="tax_src_id" ref="tax_dom_purchase_brutto_10"/>
        <field name="tax_dest_id" ref="tax_dom_purchase_0"/>
    </record>

    <!-- ## Construction -->
    <record id="construction_fiscpos_tax_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="construction"/>
        <field name="tax_src_id" ref="tax_construct_purchase_24"/>
        <field name="tax_dest_id" ref="tax_construct_purchase_24_finland"/>
    </record>
    <record id="construction_fiscpos_tax_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="construction"/>
        <field name="tax_src_id" ref="tax_dom_sales_goods_24"/>
        <field name="tax_dest_id" ref="tax_construct_sales_0"/>
    </record>

</odoo>

```

## File: data\account_tax_report_line.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report_sales_title" model="account.tax.report.line">
        <field name="name">Vero kotimaan myynneistä verokannoittain</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">0</field>
    </record>
    <record id="tax_report_sales_24" model="account.tax.report.line">
        <field name="name">24 %:n vero</field>
        <field name="code">sale_24</field>
        <field name="tag_name">fi_301</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">1</field>
        <field name="parent_id" ref="tax_report_sales_title"/>
    </record>
    <record id="tax_report_sales_14" model="account.tax.report.line">
        <field name="name">14 %:n vero</field>
        <field name="code">sale_14</field>
        <field name="tag_name">fi_302</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">2</field>
        <field name="parent_id" ref="tax_report_sales_title"/>
    </record>
    <record id="tax_report_sales_10" model="account.tax.report.line">
        <field name="name">10 %:n vero</field>
        <field name="code">sale_10</field>
        <field name="tag_name">fi_303</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">3</field>
        <field name="parent_id" ref="tax_report_sales_title"/>
    </record>
    <record id="tax_report_tax_purchase_goods_eu" model="account.tax.report.line">
        <field name="name">Vero tavaraostoista muista EU-maista</field>
        <field name="code">goods_eu</field>
        <field name="tag_name">fi_305</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">4</field>
    </record>
    <record id="tax_report_tax_purchase_service_eu" model="account.tax.report.line">
        <field name="name">Vero palveluostoista muista EU-maista</field>
        <field name="code">service_eu</field>
        <field name="tag_name">fi_tax_306</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">5</field>
    </record>
    <record id="tax_report_tax_import_goods_no_eu" model="account.tax.report.line">
        <field name="name">Vero tavaroiden maahantuonneista EU:n ulkopuolelta</field>
        <field name="code">goods_no_eu</field>
        <field name="tag_name">fi_340</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">6</field>
    </record>
    <record id="tax_report_tax_purchase_construct_service" model="account.tax.report.line">
        <field name="name">Vero rakentamispalvelun ja metalliromun ostoista (käännetty verovelvollisuus)</field>
        <field name="code">construct</field>
        <field name="tag_name">fi_318</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">7</field>
    </record>
    <record id="tax_report_deductible" model="account.tax.report.line">
        <field name="name">Verokauden vähennettävä vero</field>
        <field name="code">deductible</field>
        <field name="tag_name">fi_307</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">8</field>
    </record>
    <record id="tax_report_vat_relief" model="account.tax.report.line">
        <field name="name">Alarajahuojennuksen määrä</field>
        <field name="tag_name">fi_vat_relief</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">9</field>
    </record>
    <record id="tax_report_tax_payable" model="account.tax.report.line">
        <field name="name">Maksettava vero / Palautukseen oikeuttava vero (-)</field>
        <field name="formula">sale_24 + sale_14 + sale_10 + goods_eu + service_eu + goods_no_eu + construct - deductible</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">10</field>
    </record>

    <record id="tax_report_base_turnover_0_vat" model="account.tax.report.line">
        <field name="name">0-verokannan alainen liikevaihto</field>
        <field name="tag_name">fi_304</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">11</field>
    </record>
    <record id="tax_report_base_sales_goods_eu" model="account.tax.report.line">
        <field name="name">Tavaroiden myynnit muihin EU-maihin</field>
        <field name="tag_name">fi_311</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">12</field>
    </record>
    <record id="tax_report_base_sales_service_eu" model="account.tax.report.line">
        <field name="name">Palvelujen myynnit muihin EU-maihin</field>
        <field name="tag_name">fi_312</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">13</field>
    </record>
    <record id="tax_report_base_purchase_goods_eu" model="account.tax.report.line">
        <field name="name">Tavaraostot muista EU-maista</field>
        <field name="tag_name">fi_base_purchase_goods_eu</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">14</field>
    </record>
    <record id="tax_report_base_purchase_service_eu" model="account.tax.report.line">
        <field name="name">Palveluostot muista EU-maista</field>
        <field name="tag_name">fi_base_306</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">15</field>
    </record>
    <record id="tax_report_base_import_goods_no_eu" model="account.tax.report.line">
        <field name="name">Tavaroiden maahantuonnit EU:n ulkopuolelta</field>
        <field name="tag_name">fi_base_340</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">16</field>
    </record>
    <record id="tax_report_base_sales_construct_service" model="account.tax.report.line">
        <field name="name">Rakentamispalvelun ja metalliromun myynnit (käännetty verovelvollisuus)</field>
        <field name="tag_name">fi_319</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">17</field>
    </record>
    <record id="tax_report_base_purchase_construct_service" model="account.tax.report.line">
        <field name="name">Rakentamispalvelun ja metalliromun ostot (käännetty verovelvollisuus)</field>
        <field name="tag_name">fi_base_318</field>
        <field name="country_id" ref="base.fi"/>
        <field name="sequence">18</field>
    </record>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- # Domestic -->
    <!-- ## Domestic:sales -->
    <!-- ### Domestic:sales:goods -->
    <record id="tax_dom_sales_goods_24" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">VAT 24%</field>
        <field name="description">VAT 24%</field>
        <field name="amount">24.0</field>
        <field name="tax_group_id" ref="tax_group_24"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
              'plus_report_line_ids':  [ref('tax_report_sales_24')],
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
              'account_id': ref('account_2930'),
              'minus_report_line_ids':  [ref('tax_report_sales_24')],
            }),
        ]"/>
    </record>
    <record id="tax_dom_sales_goods_14" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">VAT 14%</field>
        <field name="description">VAT 14%</field>
        <field name="amount">14.0</field>
        <field name="tax_group_id" ref="tax_group_14"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
              'plus_report_line_ids':  [ref('tax_report_sales_14')],
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
              'account_id': ref('account_2930'),
              'minus_report_line_ids':  [ref('tax_report_sales_14')],
            }),
        ]"/>
    </record>
    <record id="tax_dom_sales_goods_10" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">VAT 10%</field>
        <field name="description">VAT 10%</field>
        <field name="amount">10.0</field>
        <field name="tax_group_id" ref="tax_group_10"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>

        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
              'plus_report_line_ids':  [ref('tax_report_sales_10')],
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
              'account_id': ref('account_2930'),
              'minus_report_line_ids':  [ref('tax_report_sales_10')],
            }),
        ]"/>
    </record>
    <record id="tax_dom_sales_goods_0" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">VAT 0%</field>
        <field name="description">VAT 0%</field>
        <field name="amount">0.0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>

        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids':  [ref('tax_report_base_turnover_0_vat')],
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
              'minus_report_line_ids':  [ref('tax_report_base_turnover_0_vat')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <!-- ### Domestic:sales:service -->
    <record id="tax_dom_sales_service_24" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">VAT 24% Service</field>
        <field name="description">VAT 24% Service</field>
        <field name="amount">24.0</field>
        <field name="tax_group_id" ref="tax_group_24"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>

        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
              'plus_report_line_ids':  [ref('tax_report_sales_24')],
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
              'account_id': ref('account_2930'),
              'minus_report_line_ids':  [ref('tax_report_sales_24')],
            }),
        ]"/>
    </record>
    <record id="tax_dom_sales_service_14" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">VAT 14% Service</field>
        <field name="description">VAT 14% Service</field>
        <field name="amount">14.0</field>
        <field name="tax_group_id" ref="tax_group_14"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>

        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
              'plus_report_line_ids':  [ref('tax_report_sales_14')],
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
              'account_id': ref('account_2930'),
              'minus_report_line_ids':  [ref('tax_report_sales_14')],
            }),
        ]"/>
    </record>
    <record id="tax_dom_sales_service_10" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">VAT 10% Service</field>
        <field name="description">VAT 10% Service</field>
        <field name="amount">10.0</field>
        <field name="tax_group_id" ref="tax_group_10"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
              'plus_report_line_ids':  [ref('tax_report_sales_10')],
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
              'account_id': ref('account_2930'),
              'minus_report_line_ids':  [ref('tax_report_sales_10')],
            }),
        ]"/>
    </record>

    <!-- ## Domestic:purchase -->

    <!-- ### Domestic:purchase:goods -->
    <record id="tax_dom_purchase_goods_24" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 24%</field>
        <field name="description">Purchase 24%</field>
        <field name="amount">24.0</field>
        <field name="tax_group_id" ref="tax_group_24"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids':  [ref('tax_report_deductible')],
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
              'account_id': ref('account_1765'),
              'minus_report_line_ids':  [ref('tax_report_deductible')],
            }),
        ]"/>
    </record>
    <record id="tax_dom_purchase_goods_14" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 14%</field>
        <field name="description">Purchase 14%</field>
        <field name="amount">14.0</field>
        <field name="tax_group_id" ref="tax_group_14"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible')],
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
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible')],
            }),
        ]"/>
    </record>
    <record id="tax_dom_purchase_goods_10" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 10%</field>
        <field name="description">Purchase 10%</field>
        <field name="amount">10.0</field>
        <field name="tax_group_id" ref="tax_group_10"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible')],
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
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible')],
            }),
        ]"/>
    </record>

    <!-- ### Domestic:purchase:service -->
    <record id="tax_dom_purchase_service_24" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 24% Service</field>
        <field name="description">Purchase 24% Service</field>
        <field name="amount">24.0</field>
        <field name="tax_group_id" ref="tax_group_24"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible')],
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
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible')],
            }),
        ]"/>
    </record>
    <record id="tax_dom_purchase_service_14" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 14% Service</field>
        <field name="description">Purchase 14% Service</field>
        <field name="amount">14.0</field>
        <field name="tax_group_id" ref="tax_group_14"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible')],
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
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible')],
            }),
        ]"/>
    </record>
    <record id="tax_dom_purchase_service_10" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 10% Service</field>
        <field name="description">Purchase 10% Service</field>
        <field name="amount">10.0</field>
        <field name="tax_group_id" ref="tax_group_10"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible')],
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
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible')],
            }),
        ]"/>
    </record>

    <!-- ### Domestic:purchase:brutto -->
    <record id="tax_dom_purchase_brutto_24" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 24% (brutto)</field>
        <field name="description">Purchase 24% (brutto)</field>
        <field name="amount">24.0</field>
        <field name="tax_group_id" ref="tax_group_24"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible')],
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
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible')],
            }),
        ]"/>
    </record>
    <record id="tax_dom_purchase_brutto_14" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 14% (brutto)</field>
        <field name="description">Purchase 14% (brutto)</field>
        <field name="amount">14.0</field>
        <field name="tax_group_id" ref="tax_group_14"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible')],
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
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible')],
            }),
        ]"/>
    </record>
    <record id="tax_dom_purchase_brutto_10" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 10% (brutto)</field>
        <field name="description">Purchase 10% (brutto)</field>
        <field name="amount">10.0</field>
        <field name="tax_group_id" ref="tax_group_10"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible')],
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
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible')],
            }),
        ]"/>
    </record>

    <!-- ### Domestic:purchase:no_category -->
    <record id="tax_dom_purchase_0" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 0%</field>
        <field name="description">Purchase 0%</field>
        <field name="amount">0.0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
    </record>

    <!-- # Europe -->
    <!-- ## Europe:sales -->
    <!-- ### Europe:sales:goods -->
    <record id="tax_eu_sales_goods_0" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">VAT 0% EU Goods</field>
        <field name="description">VAT 0% EU Goods</field>
        <field name="amount">0.0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids':  [ref('tax_report_base_sales_goods_eu')],
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
              'minus_report_line_ids':  [ref('tax_report_base_sales_goods_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <!-- ### Europe:sales:service -->
    <record id="tax_eu_sales_service_0" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">VAT 0% EU Service</field>
        <field name="description">VAT 0% EU Service</field>
        <field name="amount">0.0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids':  [ref('tax_report_base_sales_service_eu')],
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
              'minus_report_line_ids':  [ref('tax_report_base_sales_service_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <!-- ## Europe:purchase -->
    <!-- ### Europe:purchase:goods -->
    <record id="tax_eu_purchase_goods_24" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 24% EU Goods</field>
        <field name="description">Purchase 24% EU Goods</field>
        <field name="amount">24.0</field>
        <field name="tax_group_id" ref="tax_group_24"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('tax_report_base_purchase_goods_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible'), ref('tax_report_tax_purchase_goods_eu')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('tax_report_base_purchase_goods_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible'), ref('tax_report_tax_purchase_goods_eu')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
            }),
        ]"/>
    </record>
    <record id="tax_eu_purchase_goods_14" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 14% EU Goods</field>
        <field name="description">Purchase 14% EU Goods</field>
        <field name="amount">14.0</field>
        <field name="tax_group_id" ref="tax_group_14"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('tax_report_base_purchase_goods_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible'), ref('tax_report_tax_purchase_goods_eu')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('tax_report_base_purchase_goods_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible'), ref('tax_report_tax_purchase_goods_eu')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
            }),
        ]"/>
    </record>
    <record id="tax_eu_purchase_goods_10" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 10% EU Goods</field>
        <field name="description">Purchase 10% EU Goods</field>
        <field name="amount">10.0</field>
        <field name="tax_group_id" ref="tax_group_10"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('tax_report_base_purchase_goods_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible'), ref('tax_report_tax_purchase_goods_eu')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('tax_report_base_purchase_goods_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible'), ref('tax_report_tax_purchase_goods_eu')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
            }),
        ]"/>
    </record>

    <!-- ### Europe:purchase:service -->
    <record id="tax_eu_purchase_service_24" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 24% EU Service</field>
        <field name="description">Purchase 24% EU Service</field>
        <field name="amount">24.0</field>
        <field name="tax_group_id" ref="tax_group_24"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('tax_report_base_purchase_service_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible'), ref('tax_report_tax_purchase_service_eu')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('tax_report_base_purchase_service_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible'), ref('tax_report_tax_purchase_service_eu')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
            }),
        ]"/>
    </record>
    <record id="tax_eu_purchase_service_14" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 14% EU Service</field>
        <field name="description">Purchase 14% EU Service</field>
        <field name="amount">14.0</field>
        <field name="tax_group_id" ref="tax_group_14"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('tax_report_base_purchase_service_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible'), ref('tax_report_tax_purchase_service_eu')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('tax_report_base_purchase_service_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible'), ref('tax_report_tax_purchase_service_eu')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
            }),
        ]"/>
    </record>
    <record id="tax_eu_purchase_service_10" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 10% EU Service</field>
        <field name="description">Purchase 10% EU Service</field>
        <field name="amount">10.0</field>
        <field name="tax_group_id" ref="tax_group_10"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('tax_report_base_purchase_service_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible'), ref('tax_report_tax_purchase_service_eu')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('tax_report_base_purchase_service_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible'), ref('tax_report_tax_purchase_service_eu')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
            }),
        ]"/>
    </record>

    <!-- # TRIANGULATION -->
    <record id="vat0triangulation" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">VAT 0% Triangulation</field>
        <field name="description">VAT 0% Triangulation</field>
        <field name="amount">0.0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
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
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="triangulation_purchase" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Triangulation Purchase</field>
        <field name="description">Triangulation Purchase</field>
        <field name="amount">0.0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
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
              'account_id': ref('account_1765'),
            }),
        ]"/>
    </record>

    <!-- # Construct -->
    <record id="tax_construct_sales_0" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Construct 0%</field>
        <field name="description">Construct 0%</field>
        <field name="amount">0.0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids':  [ref('tax_report_base_sales_construct_service')],
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
              'minus_report_line_ids':  [ref('tax_report_base_sales_construct_service')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="tax_construct_purchase_24" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 24% Construct</field>
        <field name="description">Purchase 24% Construct</field>
        <field name="amount">24.0</field>
        <field name="tax_group_id" ref="tax_group_24"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible')],
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
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible')],
            }),
        ]"/>
    </record>
    <record id="tax_construct_purchase_24_finland" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 24% FI Construct</field>
        <field name="description">Purchase 24% FI Construct</field>
        <field name="tax_group_id" ref="tax_group_24"/>
        <field name="amount">24.0</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('tax_report_base_purchase_construct_service')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
              'minus_report_line_ids': [ref('tax_report_tax_purchase_construct_service')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('tax_report_base_purchase_construct_service')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
              'plus_report_line_ids': [ref('tax_report_tax_purchase_construct_service')],
            }),
        ]"/>
    </record>

    <!-- # Aland -->
    <!-- ## Aland:sales -->
    <record id="aland_sales_0" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Aland 0%</field>
        <field name="description">Aland 0%</field>
        <field name="amount">0.0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids':  [ref('tax_report_base_turnover_0_vat')],
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
              'minus_report_line_ids':  [ref('tax_report_base_turnover_0_vat')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <!-- # Non EU -->
    <!-- ## Non EU:purchase -->
    <!-- ## Non EU:purchase:goods -->
    <record id="tax_non_eu_purchase_goods_24" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 24% Non EU Goods</field>
        <field name="description">Purchase 24% Non EU Goods</field>
        <field name="amount">24.0</field>
        <field name="tax_group_id" ref="tax_group_24"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('tax_report_base_import_goods_no_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible'), ref('tax_report_tax_import_goods_no_eu')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('tax_report_base_import_goods_no_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible'), ref('tax_report_tax_import_goods_no_eu')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
            }),
        ]"/>
    </record>
    <record id="tax_non_eu_purchase_goods_14" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 14% Non EU Goods</field>
        <field name="description">Purchase 14% Non EU Goods</field>
        <field name="amount">14.0</field>
        <field name="tax_group_id" ref="tax_group_14"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('tax_report_base_import_goods_no_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible'), ref('tax_report_tax_import_goods_no_eu')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('tax_report_base_import_goods_no_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible'), ref('tax_report_tax_import_goods_no_eu')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
            }),
        ]"/>
    </record>
    <record id="tax_non_eu_purchase_goods_10" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Purchase 10% Non EU Goods</field>
        <field name="description">Purchase 10% Non EU Goods</field>
        <field name="amount">10.0</field>
        <field name="tax_group_id" ref="tax_group_10"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('tax_report_base_import_goods_no_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible'), ref('tax_report_tax_import_goods_no_eu')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('tax_report_base_import_goods_no_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible'), ref('tax_report_tax_import_goods_no_eu')],
            }),
            (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('account_2930'),
            }),
        ]"/>
    </record>

    <!-- ## Non EU:others -->
    <record id="vat0export" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">VAT 0% Export</field>
        <field name="description">VAT 0% Export</field>
        <field name="amount">0.0</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>

        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids':  [ref('tax_report_base_turnover_0_vat')],
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
              'minus_report_line_ids':  [ref('tax_report_base_turnover_0_vat')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="import_pay24" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Import Pay24</field>
        <field name="description">24%</field>
        <field name="amount">24.0</field>
        <field name="tax_group_id" ref="tax_group_24"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('tax_report_base_import_goods_no_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_tax_import_goods_no_eu')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('tax_report_base_import_goods_no_eu')],
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_tax_import_goods_no_eu')],
            }),
        ]"/>
    </record>
    <record id="import_deduct24" model="account.tax.template">
        <field name="chart_template_id" ref="fi_chart_template"/>
        <field name="name">Import Deduct24</field>
        <field name="description">24%</field>
        <field name="amount">24.0</field>
        <field name="tax_group_id" ref="tax_group_24"/>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('account_1765'),
              'plus_report_line_ids': [ref('tax_report_deductible')],
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
              'account_id': ref('account_1765'),
              'minus_report_line_ids': [ref('tax_report_deductible')],
            }),
        ]"/>
    </record>

</odoo>

```

## File: data\l10n_fi_chart_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="fi_chart_template" model="account.chart.template">
        <field name="code_digits">4</field>
        <field name="property_account_receivable_id" ref="account_1700"/>
        <field name="property_account_payable_id" ref="account_2700"/>
        <field name="property_account_expense_categ_id" ref="account_4000"/>
        <field name="property_account_income_categ_id" ref="account_3000"/>
        <field name="property_account_expense_id" ref="account_4000"/>
        <field name="property_account_income_id" ref="account_3000"/>
        <field name="expense_currency_exchange_account_id" ref="account_4380"/>
        <field name="income_currency_exchange_account_id" ref="account_3500"/>
        <field name="property_tax_payable_account_id" ref="account_2930"/>
        <field name="property_tax_receivable_account_id" ref="account_1765"/>
        <field name="default_pos_receivable_account_id" ref="account_1701"/>
    </record>
</odoo>

```

## File: models\account_journal.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields


class AccountJournal(models.Model):

    _inherit = 'account.journal'

    invoice_reference_model = fields.Selection(selection_add=[
        ('fi', 'Finnish Standard Reference'),
        ('fi_rf', 'Finnish Creditor Reference (RF)'),
    ])

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import re
from odoo import api, models, _
from odoo.exceptions import UserError
import logging

log = logging.getLogger(__name__)


class AccountInvoiceFinnish(models.Model):
    _inherit = 'account.move'

    @api.model
    def number2numeric(self, number):

        invoice_number = re.sub(r'\D', '', number)

        if invoice_number == '' or invoice_number is False:
            raise UserError(_('Invoice number must contain numeric characters'))

        # Make sure the base number is 3...19 characters long
        if len(invoice_number) < 3:
            invoice_number = ('11' + invoice_number)[-3:]
        elif len(invoice_number) > 19:
            invoice_number = invoice_number[:19]

        return invoice_number

    @api.model
    def get_finnish_check_digit(self, base_number):
        # Multiply digits from end to beginning with 7, 3 and 1 and
        # calculate the sum of the products
        total = sum((7, 3, 1)[idx % 3] * int(val) for idx, val in
                    enumerate(base_number[::-1]))

        # Subtract the sum from the next decade. 10 = 0
        return str((10 - (total % 10)) % 10)

    @api.model
    def get_rf_check_digits(self, base_number):
        check_base = base_number + 'RF00'
        # 1. Convert all non-digits to digits
        # 2. Calculate the modulo 97
        # 3. Subtract the remainder from 98
        # 4. Add leading zeros if necessary
        return ''.join(
            ['00', str(98 - (int(''.join(
                [x if x.isdigit() else str(ord(x) - 55) for x in
                 check_base])) % 97))])[-2:]

    @api.model
    def compute_payment_reference_finnish(self, number):
        # Drop all non-numeric characters
        invoice_number = self.number2numeric(number)

        # Calculate the Finnish check digit
        check_digit = self.get_finnish_check_digit(invoice_number)

        return invoice_number + check_digit

    @api.model
    def compute_payment_reference_finnish_rf(self, number):
        # Drop all non-numeric characters
        invoice_number = self.number2numeric(number)

        # Calculate the Finnish check digit
        invoice_number += self.get_finnish_check_digit(invoice_number)

        # Calculate the RF check digits
        rf_check_digits = self.get_rf_check_digits(invoice_number)

        return 'RF' + rf_check_digits + invoice_number

    def _get_invoice_reference_fi_rf_invoice(self):
        self.ensure_one()
        return self.compute_payment_reference_finnish_rf(self.name)

    def _get_invoice_reference_fi_rf_partner(self):
        self.ensure_one()
        return self.compute_payment_reference_finnish_rf(str(self.partner_id.id))

    def _get_invoice_reference_fi_invoice(self):
        self.ensure_one()
        return self.compute_payment_reference_finnish(self.name)

    def _get_invoice_reference_fi_partner(self):
        self.ensure_one()
        return self.compute_payment_reference_finnish(str(self.partner_id.id))

```

## File: models\__init__.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_journal
from . import account_move

```

