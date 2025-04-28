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
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Denmark - Accounting',
    'version': '1.1',
    'author': 'Odoo House ApS, VK DATA ApS',
    'website': 'http://odoodanmark.dk',
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
    'depends': ['account', 'base_iban', 'base_vat'],
    'data': [
        'data/account_account_tags.xml',
        'data/l10n_dk_chart_template_data.xml',
        'data/account.account.template.csv',
        'data/l10n_dk_chart_template_post_data.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_template_data.xml',
        'data/account_fiscal_position_template.xml',
        'data/account_fiscal_position_tax_template.xml',
        'data/account_fiscal_position_account_template.xml',
        'data/account_chart_template_configuration_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","account_type","tag_ids/id","chart_template_id/id","reconcile"
"a1010","Salg af vare, m/moms","1010","income","account_tag_revenue","dk_chart_template","False"
"a1011","Salg af ydelser, m/moms","1011","income","account_tag_revenue","dk_chart_template","False"
"a1012","Salg af vare, u/moms","1012","income","account_tag_revenue","dk_chart_template","False"
"a1013","Salg af ydelser, u/moms","1013","income","account_tag_revenue","dk_chart_template","False"
"a1020","Salg af vare, EU","1020","income","account_tag_revenue","dk_chart_template","False"
"a1021","Salg af ydelser, EU","1021","income","account_tag_revenue","dk_chart_template","False"
"a1030","Salg af vare, 3. lande","1030","income","account_tag_revenue","dk_chart_template","False"
"a1031","Salg af ydelser, 3. lande","1031","income","account_tag_revenue","dk_chart_template","False"
"a2010","Direkte omkostninger vare, m/moms","2010","expense_direct_cost","account_tag_direct_costs","dk_chart_template","False"
"a2011","Direkte omkostninger ydelser, m/moms","2011","expense_direct_cost","account_tag_direct_costs","dk_chart_template","False"
"a2012","Direkte omkostninger vare, u/moms","2012","expense_direct_cost","account_tag_direct_costs","dk_chart_template","False"
"a2013","Direkte omkostninger ydelser, u/moms","2013","expense_direct_cost","account_tag_direct_costs","dk_chart_template","False"
"a2020","Direkte omkostninger vare, EU","2020","expense_direct_cost","account_tag_direct_costs","dk_chart_template","False"
"a2021","Direkte omkostninger ydelser, EU","2021","expense_direct_cost","account_tag_direct_costs","dk_chart_template","False"
"a2030","Direkte omkostninger vare, 3. lande","2030","expense_direct_cost","account_tag_direct_costs","dk_chart_template","False"
"a2031","Direkte omkostninger ydelser, 3. lande","2031","expense_direct_cost","account_tag_direct_costs","dk_chart_template","False"
"a2100","Vareforbrug, lagermodul","2100","expense_direct_cost","account_tag_direct_costs","dk_chart_template","False"
"a2110","Prisdifferencer","2110","expense_direct_cost","account_tag_direct_costs","dk_chart_template","False"
"a2600","Lager, primo","2600","expense_direct_cost","account_tag_direct_costs","dk_chart_template","False"
"a2605","Lager, ultimo","2605","expense_direct_cost","account_tag_direct_costs","dk_chart_template","False"
"a2610","Lagerændring","2610","expense_direct_cost","account_tag_direct_costs","dk_chart_template","False"
"a3010","Produktionsløn","3010","expense","account_tag_staff_costs","dk_chart_template","False"
"a3020","Gager","3020","expense","account_tag_staff_costs","dk_chart_template","False"
"a3610","Arbejdskadeforsikring","3610","expense","account_tag_staff_costs","dk_chart_template","False"
"a3620","Ansvarsforsikring","3620","expense","account_tag_staff_costs","dk_chart_template","False"
"a3630","Dagpengeforsikring","3630","expense","account_tag_staff_costs","dk_chart_template","False"
"a3640","Gruppelivsforsikring","3640","expense","account_tag_staff_costs","dk_chart_template","False"
"a3655","ATP-bidrag","3655","expense","account_tag_staff_costs","dk_chart_template","False"
"a3680","Kursusudgifter","3680","expense","account_tag_staff_costs","dk_chart_template","False"
"a3690","Personaleudgifter","3690","expense","account_tag_staff_costs","dk_chart_template","False"
"a3695","Fortæring","3695","expense","account_tag_staff_costs","dk_chart_template","False"
"a3700","Befordringsgodtgørelse","3700","expense","account_tag_staff_costs","dk_chart_template","False"
"a3710","Skattefrie godtgørelser","3710","expense","account_tag_staff_costs","dk_chart_template","False"
"a3740","Arbejdstøj","3740","expense","account_tag_staff_costs","dk_chart_template","False"
"a3810","Refunderet sygedagpenge","3810","expense","account_tag_staff_costs","dk_chart_template","False"
"a3820","Løntilskud","3820","expense","account_tag_staff_costs","dk_chart_template","False"
"a3840","Godtgørelse ATP","3840","expense","account_tag_staff_costs","dk_chart_template","False"
"a4010","Restaurationsbesøg","4010","expense","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4020","Anden repræsentation","4020","expense","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4036","Rejseudgifter","4036","expense","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4040","Hotelophold","4040","expense","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4060","Annoncer","4060","expense","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4065","Erhvervsnetværk","4065","expense","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4070","Dekoration/skilte","4070","expense","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4071","Udstillinger","4071","expense","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4073","Tryksager/brochurer","4073","expense","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4074","Reklame-emballage","4074","expense","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4076","Emballage,salgsomkostninger","4076","expense","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4080","Fragt, porto","4080","expense","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4081","Fragt, porto u/moms","4081","expense","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4102","Brændstof vare/lastbil","4102","expense","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4103","Vægtafgift vare/lastbil","4103","expense","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4104","Forsikring vare/lastbil","4104","expense","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4105","Vedligeholdelse vare/lastbil","4105","expense","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4108","Redningskorps vare/lastbil","4108","expense","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4120","Leje vare/lastbil","4120","expense","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4152","Brændstof personbil","4152","expense","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4153","Vægtafgift personbil","4153","expense","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4154","Forsikring personbil","4154","expense","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4155","Vedligeholdelse personbil","4155","expense","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4158","Redningskorps personbil","4158","expense","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4170","Leje af personbil","4170","expense","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4189","Privat andel af bil, drift af biler","4189","expense","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4190","Kørsel i privat bil","4190","expense","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4205","Leje/leasing af driftsmidler","4205","expense","account_tag_rooms_costs","dk_chart_template","False"
"a4235","Vedligeholdelse, driftsmiddelomkostninger","4235","expense","account_tag_rooms_costs","dk_chart_template","False"
"a4240","Småanskaffelser","4240","expense","account_tag_rooms_costs","dk_chart_template","False"
"a4241","Småanskaffelser o/11.000","4241","expense","account_tag_rooms_costs","dk_chart_template","False"
"a4255","Husleje m/moms","4255","expense","account_tag_rooms_costs","dk_chart_template","False"
"a4256","Husleje u/moms","4256","expense","account_tag_rooms_costs","dk_chart_template","False"
"a4260","Lagerleje","4260","expense","account_tag_rooms_costs","dk_chart_template","False"
"a4265","Garageleje","4265","expense","account_tag_rooms_costs","dk_chart_template","False"
"a4271","Varme","4271","expense","account_tag_rooms_costs","dk_chart_template","False"
"a4275","El og vand","4275","expense","account_tag_rooms_costs","dk_chart_template","False"
"a4276","Refunderet energiafgift","4276","expense","account_tag_rooms_costs","dk_chart_template","False"
"a4285","Privat andel el, vand","4285","expense","account_tag_rooms_costs","dk_chart_template","False"
"a4291","Vedligeholdelse, lokaleomkostninger","4291","expense","account_tag_rooms_costs","dk_chart_template","False"
"a4295","Rengøring","4295","expense","account_tag_rooms_costs","dk_chart_template","False"
"a4296","Renovation","4296","expense","account_tag_rooms_costs","dk_chart_template","False"
"a4302","Kontorartikler/tryksager","4302","expense","account_tag_administration_costs","dk_chart_template","False"
"a4307","Inkassoomkostninger","4307","expense","account_tag_administration_costs","dk_chart_template","False"
"a4308","Vedligeholdelse inventar","4308","expense","account_tag_administration_costs","dk_chart_template","False"
"a4310","EDB-udgifter","4310","expense","account_tag_administration_costs","dk_chart_template","False"
"a4311","Mail/Webhotel","4311","expense","account_tag_administration_costs","dk_chart_template","False"
"a4312","Løbende backup","4312","expense","account_tag_administration_costs","dk_chart_template","False"
"a4316","Telefon","4316","expense","account_tag_administration_costs","dk_chart_template","False"
"a4322","Privat andel telefon","4322","expense","account_tag_administration_costs","dk_chart_template","False"
"a4325","Internet","4325","expense","account_tag_administration_costs","dk_chart_template","False"
"a4327","Porto","4327","expense","account_tag_administration_costs","dk_chart_template","False"
"a4335","Regnskabsassistance","4335","expense","account_tag_administration_costs","dk_chart_template","False"
"a4338","Databehandling","4338","expense","account_tag_administration_costs","dk_chart_template","False"
"a4340","Advokathonorar","4340","expense","account_tag_administration_costs","dk_chart_template","False"
"a4343","Konsulentbistand","4343","expense","account_tag_administration_costs","dk_chart_template","False"
"a4350","Forsikringer","4350","expense","account_tag_administration_costs","dk_chart_template","False"
"a4351","Vagtværn/alarmsystem","4351","expense","account_tag_administration_costs","dk_chart_template","False"
"a4360","Faglitteratur/tidsskrifter","4360","expense","account_tag_administration_costs","dk_chart_template","False"
"a4370","Kontingenter","4370","expense","account_tag_administration_costs","dk_chart_template","False"
"a4381","Tab på debitorer","4381","expense","account_tag_administration_costs","dk_chart_template","False"
"a4392","Øvrige kapacitetsudgifter","4392","expense","account_tag_administration_costs","dk_chart_template","False"
"a4451","Driftsafskrivninger, grunde og bygninger","4451","expense_depreciation","account_tag_depreciation","dk_chart_template","False"
"a4454","Driftsafskrivninger, andre anlæg, driftsmateriel og inventar","4454","expense_depreciation","account_tag_depreciation","dk_chart_template","False"
"a4651","Renteindtægter, bank","4651","income_other","account_tag_financial_income","dk_chart_template","False"
"a4664","Renteindtægter, debitorer","4664","income_other","account_tag_financial_income","dk_chart_template","False"
"a4660","Kasserabat","4660","income_other","account_tag_financial_income","dk_chart_template","False"
"a4670","Kursgevinster","4670","income_other","account_tag_financial_income","dk_chart_template","False"
"a4680","Gebyrer, indtægter","4680","income_other","account_tag_financial_income","dk_chart_template","False"
"a4730","Renteudgifter, bank","4730","expense","account_tag_financial_expenses","dk_chart_template","False"
"a4740","Renteudgifter, realkredit","4740","expense","account_tag_financial_expenses","dk_chart_template","False"
"a4743","Renteudgifter, kreditorer","4743","expense","account_tag_financial_expenses","dk_chart_template","False"
"a4744","Renteudgifter, kontrakter","4744","expense","account_tag_financial_expenses","dk_chart_template","False"
"a4745","Renteudgifter, pantebreve","4745","expense","account_tag_financial_expenses","dk_chart_template","False"
"a4746","Renteudgifter, ej fradrag","4746","expense","account_tag_financial_expenses","dk_chart_template","False"
"a4750","Garantiprovision","4750","expense","account_tag_financial_expenses","dk_chart_template","False"
"a4755","Gebyrer, udgifter","4755","expense","account_tag_financial_expenses","dk_chart_template","False"
"a4760","Kasserabat","4760","expense","account_tag_financial_expenses","dk_chart_template","False"
"a4766","Låneomkostninger","4766","expense","account_tag_financial_expenses","dk_chart_template","False"
"a4770","Kurstab","4770","expense","account_tag_financial_expenses","dk_chart_template","False"
"a4805","Fortjeneste, salg anlæg","4805","income_other","account_tag_extraordinary_income","dk_chart_template","False"
"a4819","Indtægter vedr. tidl. år","4819","income_other","account_tag_extraordinary_income","dk_chart_template","False"
"a4847","Andre ekstraordinære indtægter","4847","income_other","account_tag_extraordinary_income","dk_chart_template","False"
"a4854","Tab, salg anlæg","4854","expense","account_tag_extraordinary_expenses","dk_chart_template","False"
"a4890","Udgifter vedr. tidl. år","4890","expense","account_tag_extraordinary_expenses","dk_chart_template","False"
"a4895","Andre ekstraordinære udgifter","4895","expense","account_tag_extraordinary_expenses","dk_chart_template","False"
"a5201","Kostpris, primo,goodwill, licenser mm.","5201","asset_non_current","account_tag_Goodwill_licenses_etc","dk_chart_template","False"
"a5202","Tilgang i årets løb, goodwill, licenser mm.","5202","asset_non_current","account_tag_Goodwill_licenses_etc","dk_chart_template","False"
"a5203","Afgang i årets løb,goodwill, licenser mm.","5203","asset_non_current","account_tag_Goodwill_licenses_etc","dk_chart_template","False"
"a5204","Afskrivninger primo, goodwill, licenser mm.","5204","asset_non_current","account_tag_Goodwill_licenses_etc","dk_chart_template","False"
"a5205","Afskrivninger på afgang, goodwill, licenser mm.","5205","asset_non_current","account_tag_Goodwill_licenses_etc","dk_chart_template","False"
"a5206","Årsafskrivning, goodwill, licenser mm.","5206","asset_non_current","account_tag_Goodwill_licenses_etc","dk_chart_template","False"
"a5511","Kostpris, primo, grunde og bygninger","5511","asset_fixed","account_tag_land_buildings","dk_chart_template","False"
"a5512","Tilgang i årets løb,grunde og bygninger","5512","asset_fixed","account_tag_land_buildings","dk_chart_template","False"
"a5513","Afgang i årets løb,grunde og bygninger","5513","asset_fixed","account_tag_land_buildings","dk_chart_template","False"
"a5514","Afskrivninger primo, grunde og bygninger","5514","asset_fixed","account_tag_land_buildings","dk_chart_template","False"
"a5515","Afskrivninger på afgang, grunde og bygninger","5515","asset_fixed","account_tag_land_buildings","dk_chart_template","False"
"a5516","Årsafskrivning, grunde og bygninger","5516","asset_fixed","account_tag_land_buildings","dk_chart_template","False"
"a5701","Kostpris, primo, andre anlæg, driftsmateriel og inventar","5701","asset_fixed","account_tag_other_equipment_tools_inventory","dk_chart_template","False"
"a5702","Tilgang i årets løb, andre anlæg, driftsmateriel og inventar","5702","asset_fixed","account_tag_other_equipment_tools_inventory","dk_chart_template","False"
"a5703","Afgang i årets løb, andre anlæg, driftsmateriel og inventar","5703","asset_fixed","account_tag_other_equipment_tools_inventory","dk_chart_template","False"
"a5704","Afskrivninger primo, andre anlæg, driftsmateriel og inventar","5704","asset_fixed","account_tag_other_equipment_tools_inventory","dk_chart_template","False"
"a5705","Afskrivninger på afgang, andre anlæg, driftsmateriel og inventar","5705","asset_fixed","account_tag_other_equipment_tools_inventory","dk_chart_template","False"
"a5706","Årsafskrivning, andre anlæg, driftsmateriel og inventar","5706","asset_fixed","account_tag_other_equipment_tools_inventory","dk_chart_template","False"
"a6510","Råvarer","6510","asset_current","account_tag_inventories","dk_chart_template","False"
"a6520","Varer under fremstilling","6520","asset_current","account_tag_inventories","dk_chart_template","False"
"a6525","Fremstillede færdigvarer","6525","asset_current","account_tag_inventories","dk_chart_template","False"
"a6530","Varelager","6530","asset_current","account_tag_inventories","dk_chart_template","False"
"a6540","Igangværende arbejder","6540","asset_current","account_tag_work_in_progress","dk_chart_template","False"
"a6580","Forudbetaling for varer","6580","asset_prepayments","account_tag_prepayments","dk_chart_template","False"
"a6610","Debitorer","6610","asset_receivable","account_tag_receivable","dk_chart_template","True"
"a6611","Debitorer (PoS)","6611","asset_receivable","account_tag_receivable","dk_chart_template","True"
"a6640","Andre tilgodehavender","6640","asset_current","account_tag_other_receivables","dk_chart_template","False"
"a6650","Kundekort/Kreditkort","6650","asset_current","account_tag_other_receivables","dk_chart_template","False"
"a6655","Depositum","6655","asset_non_current","account_tag_other_receivables","dk_chart_template","False"
"a6670","Varer leveret, faktura ikke sendt","6670","asset_current","account_tag_other_receivables","dk_chart_template","True"
"a6680","Periodeafgrænsningsposter, tilgodehavender","6680","asset_current","account_tag_other_receivables","dk_chart_template","True"
"a6860","Bundne likvider","6860","asset_cash","account_tag_liquidity","dk_chart_template","True"
"a7210","Egenkapital, primo","7210","equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7220","Indskud","7220","equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7230","Hævet privat","7230","equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7240","B-skatter og AM-bidrag","7240","equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7245","Restskat og overskydende skat","7245","equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7250","Forbrug af egne varer, egenkapital","7250","equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7260","Privat andel af bil, egenkapital","7260","equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7280","Privat andel, telefon, egenkapital","7280","equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7290","Privat andel i øvrigt, egenkapital","7290","equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7299","Overført resultat","7299","equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7410","Virksomhedskapital","7410","equity","account_tag_equity_corporations","dk_chart_template","False"
"a7420","Overkurs ved emission","7420","equity","account_tag_equity_corporations","dk_chart_template","False"
"a7430","Reserve for opskrivning","7430","equity","account_tag_equity_corporations","dk_chart_template","False"
"a7440","Andre reserver","7440","equity","account_tag_equity_corporations","dk_chart_template","False"
"a7450","Overført resultat","7450","equity","account_tag_equity_corporations","dk_chart_template","False"
"a7460","Forslag til udbytte","7460","equity","account_tag_equity_corporations","dk_chart_template","False"
"a7705","Realkreditinstitutter","7705","liability_non_current","account_tag_credit_institutions","dk_chart_template","False"
"a7710","Private pantebreve","7710","liability_non_current","account_tag_credit_institutions","dk_chart_template","False"
"a7720","Langfristede banklån","7720","liability_non_current","account_tag_credit_institutions","dk_chart_template","False"
"a8400","Kassekredit","8400","liability_non_current","account_tag_credit_institutions","dk_chart_template","False"
"a8410","Anden bankgæld","8410","liability_non_current","account_tag_bank_loans","dk_chart_template","False"
"a8430","Modtagne forudbetalinger","8430","liability_current","account_tag_bank_loans","dk_chart_template","False"
"a8440","Kreditorer","8440","liability_payable","account_tag_payable","dk_chart_template","True"
"a8450","Vare modtaget, faktura ikke modtaget","8450","liability_current","account_tag_payable","dk_chart_template","True"
"a8475","Revisorhonorar","8475","liability_current","account_tag_payable","dk_chart_template","False"
"a8720","Salgsmoms (udgående moms)","8720","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8725","Erhvervelsesmoms med omvendt betlingspligt","8725","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8730","Moms af varekøb i udlandet (Både EU og 3. lande)","8730","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8731","Moms af ydelseskøb i udlandet med omvendt betalingspligt","8731","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8739","Importmoms, modkonto","8739","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8740","Købsmoms (indgående moms)","8740","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8750","EU købsværdi, Rubrik A - vare","8750","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8751","EU købsværdi, Rubrik A - ydelser","8751","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8754","EU salgsværdi, Rubrik B - vare","8754","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8755","EU salgsværdi, Rubrik B - ydelser","8755","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8758","Salgsværdier 3.lande - Rubrik C","8758","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8759","Salgsværdier med omvendt betalingspligt - Rubrik C","8759","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8769","Værdier, modkonto","8769","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8770","Olie- og flaskegasafgift","8770","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8775","Elafgift","8775","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8780","Naturgas- og bygasafgift","8780","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8785","Kulafgift","8785","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8790","CO-2 afgift","8790","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8795","Vandafgift","8795","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8798","Afregning af moms","8798","liability_current","account_tag_vat_taxes","dk_chart_template","False"
"a8801","Skyldig A-skat","8801","liability_current","account_tag_Payroll_related_liabilities","dk_chart_template","False"
"a8805","Skyldig AM-bidrag","8805","liability_current","account_tag_Payroll_related_liabilities","dk_chart_template","False"
"a8810","Skyldig ATP","8810","liability_current","account_tag_Payroll_related_liabilities","dk_chart_template","False"
"a8815","Skyldige feriepenge","8815","liability_current","account_tag_Payroll_related_liabilities","dk_chart_template","False"
"a8820","Afsat skyldige feriepenge","8820","liability_current","account_tag_Payroll_related_liabilities","dk_chart_template","False"
"a8840","Skyldig lån","8840","liability_current","account_tag_Payroll_related_liabilities","dk_chart_template","False"
"a8855","Periodeafgrænsningsposter, gæld","8855","liability_current","account_tag_other_current_liabilities","dk_chart_template","False"
"a8860","Andre skyldige omk.","8860","liability_current","account_tag_other_current_liabilities","dk_chart_template","False"

```

## File: data\account_account_tags.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <!--Resultatopgørelse indtægter-->
    <record id="account_tag_revenue" model="account.account.tag">
        <field name="name">Omsætning</field>
        <field name="applicability">accounts</field>
    </record>

    <!--Resultatopgørelse udgifer-->
    <record id="account_tag_direct_costs" model="account.account.tag">
        <field name="name">Direkte omkostninger</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_staff_costs" model="account.account.tag">
        <field name="name">Personaleomkostninger</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_sales_promotions_costs" model="account.account.tag">
        <field name="name">Salgsfremmende omkostninger</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_cars_equipment_costs" model="account.account.tag">
        <field name="name">Biler og driftmidler</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_rooms_costs" model="account.account.tag">
        <field name="name">Lokaleomkostninger</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_administration_costs" model="account.account.tag">
        <field name="name">Administrationsomkostninger</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_depreciation" model="account.account.tag">
        <field name="name">Af- og nedskrivninger</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_financial_income" model="account.account.tag">
        <field name="name">Finansielle indtægter</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_financial_expenses" model="account.account.tag">
        <field name="name">Finansielle udgifter</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_extraordinary_income" model="account.account.tag">
        <field name="name">Ekstraordinære indtægter</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_extraordinary_expenses" model="account.account.tag">
        <field name="name">Ekstraordinære udgifter</field>
        <field name="applicability">accounts</field>
    </record>

    <!--Assets Aktiver-->
    <record id="account_tag_Goodwill_licenses_etc" model="account.account.tag">
        <field name="name">Goodwill, licenser mm.</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_land_buildings" model="account.account.tag">
        <field name="name">Grunde og bygninger</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_equipment_tools_inventory" model="account.account.tag">
        <field name="name">Andre anlæg, driftsmateriel og inventar</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_inventories" model="account.account.tag">
        <field name="name">Varebeholdninger</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_work_in_progress" model="account.account.tag">
        <field name="name">Igangværende arbejder</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_prepayments" model="account.account.tag">
        <field name="name">Forudbetaling til leverandører</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_receivable" model="account.account.tag">
        <field name="name">Tilgodehavender</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_receivables" model="account.account.tag">
        <field name="name">Andre tilgodehavender</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_liquidity" model="account.account.tag">
        <field name="name">Likvide beholdninger</field>
        <field name="applicability">accounts</field>
    </record>

    <!--Liabilities Passiver-->
     <record id="account_tag_equity_individual_enterprises" model="account.account.tag">
        <field name="name">Egenkapital enkeltmandsvirksomheder</field>
        <field name="applicability">accounts</field>
     </record>
     <record id="account_tag_equity_corporations" model="account.account.tag">
        <field name="name">Egenkapital selskaber</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_credit_institutions" model="account.account.tag">
        <field name="name">Kreditinstitutter</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_bank_loans" model="account.account.tag">
        <field name="name">Bankgæld</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_prepayments_from_customers" model="account.account.tag">
        <field name="name">Forudbetalinger fra kunder</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_payable" model="account.account.tag">
        <field name="name">Leverandørgæld</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_vat_taxes" model="account.account.tag">
        <field name="name">Moms og afgifter</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_Payroll_related_liabilities" model="account.account.tag">
        <field name="name">Løn relateret gæld</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="account_tag_other_current_liabilities" model="account.account.tag">
        <field name="name">Anden kortfristet gæld</field>
        <field name="applicability">accounts</field>
    </record>
</odoo>

```

## File: data\account_chart_template_configuration_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <function model="account.chart.template" name="try_loading">
        <value eval="[ref('l10n_dk.dk_chart_template')]"/>
    </function>
</odoo>

```

## File: data\account_fiscal_position_account_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- 3. lande -->
    <record id="fiscal_position_account_salgvare_3" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_3lande"/>
        <field name="account_src_id" ref="a1010"/>
        <field name="account_dest_id" ref="a1030"/>
    </record>
    <record id="fiscal_position_account_salgydelser_3" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_3lande"/>
        <field name="account_src_id" ref="a1011"/>
        <field name="account_dest_id" ref="a1031"/>
    </record>
    <record id="fiscal_position_account_koebvare_3" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_3lande"/>
        <field name="account_src_id" ref="a2010"/>
        <field name="account_dest_id" ref="a2030"/>
    </record>
    <record id="fiscal_position_account_koebydelser_3" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_3lande"/>
        <field name="account_src_id" ref="a2011"/>
        <field name="account_dest_id" ref="a2031"/>
    </record>

    <!-- EU lande -->
    <record id="fiscal_position_account_salgvare_eu" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_eu_taxid"/>
        <field name="account_src_id" ref="a1010"/>
        <field name="account_dest_id" ref="a1020"/>
    </record>
    <record id="fiscal_position_account_salgydelser_eu" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_eu_taxid"/>
        <field name="account_src_id" ref="a1011"/>
        <field name="account_dest_id" ref="a1021"/>
    </record>
    <record id="fiscal_position_account_koebvare_eu" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_eu_taxid"/>
        <field name="account_src_id" ref="a2010"/>
        <field name="account_dest_id" ref="a2020"/>
    </record>
    <record id="fiscal_position_account_koebydelser_eu_taxid" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_eu_taxid"/>
        <field name="account_src_id" ref="a2011"/>
        <field name="account_dest_id" ref="a2021"/>
    </record>
</odoo>

```

## File: data\account_fiscal_position_tax_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <!-- EU lande Business -->
    <record id="position_tax_salgvare_eu" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_eu_taxid"/>
        <field name="tax_src_id" ref="tax110"/>
        <field name="tax_dest_id" ref="tax210"/>
    </record>
    <record id="position_tax_salgydelser_eu" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_eu_taxid"/>
        <field name="tax_src_id" ref="tax120"/>
        <field name="tax_dest_id" ref="tax220"/>
    </record>
    <record id="position_tax_koebvare_eu" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_eu_taxid"/>
        <field name="tax_src_id" ref="tax400"/>
        <field name="tax_dest_id" ref="tax510"/>
    </record>
    <record id="position_tax_koebvare_indeholt_eu" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_eu_taxid"/>
        <field name="tax_src_id" ref="tax410"/>
        <field name="tax_dest_id" ref="tax510"/>
    </record>
    <record id="position_tax_koebydelser_eu" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_eu_taxid"/>
        <field name="tax_src_id" ref="tax420"/>
        <field name="tax_dest_id" ref="tax520"/>
    </record>
    <record id="position_tax_koebydelser_indeholdt_eu" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_eu_taxid"/>
        <field name="tax_src_id" ref="tax425"/>
        <field name="tax_dest_id" ref="tax520"/>
    </record>

    <!-- 3. lande -->
    <record id="position_tax_salgvare_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3lande"/>
        <field name="tax_src_id" ref="tax110"/>
        <field name="tax_dest_id" ref="tax310"/>
    </record>
    <record id="position_tax_salgydelser_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3lande"/>
        <field name="tax_src_id" ref="tax120"/>
        <field name="tax_dest_id" ref="tax310"/>
    </record>
    <record id="position_tax_koebvare_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3lande"/>
        <field name="tax_src_id" ref="tax400"/>
        <field name="tax_dest_id" ref="tax610"/>
    </record>
    <record id="position_tax_koebvare_indeholdt_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3lande"/>
        <field name="tax_src_id" ref="tax410"/>
        <field name="tax_dest_id" ref="tax610"/>
    </record>
    <record id="position_tax_koebydelser_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3lande"/>
        <field name="tax_src_id" ref="tax420"/>
        <field name="tax_dest_id" ref="tax620"/>
    </record>
    <record id="position_tax_koebydelser_indeholdt_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3lande"/>
        <field name="tax_src_id" ref="tax425"/>
        <field name="tax_dest_id" ref="tax620"/>
    </record>
</odoo>

```

## File: data\account_fiscal_position_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="fiscal_position_template_dk_vat" model="account.fiscal.position.template">
        <field name="name">Danmark (Virksomhed)</field>
        <field name="chart_template_id" ref="dk_chart_template" />
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="True"/>
        <field name="sequence">10</field>
        <field name="country_id" ref="base.dk"/>
    </record>
    <record id="fiscal_position_template_eu" model="account.fiscal.position.template">
        <field name="name">EU lande (Privat)</field>
        <field name="chart_template_id" ref="dk_chart_template" />
        <field name="auto_apply" eval="True"/>
        <field name="sequence">11</field>
        <field name="country_group_id" ref="base.europe"/>
    </record>
    <record id="fiscal_position_template_eu_taxid" model="account.fiscal.position.template">
        <field name="name">EU lande (Virksomhed)</field>
        <field name="chart_template_id" ref="dk_chart_template" />
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="True"/>
        <field name="sequence">12</field>
        <field name="country_group_id" ref="base.europe"/>
    </record>
    <record id="fiscal_position_template_3lande" model="account.fiscal.position.template">
        <field name="name">3. lande (Virksomhed / Privat)</field>
        <field name="chart_template_id" ref="dk_chart_template" />
        <field name="auto_apply" eval="True"/>
        <field name="sequence">13</field>
    </record>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
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
                <field name="name">Skyldig moms</field>
                <field name="code">DUE_VAT</field>
                <field name="aggregation_formula">DK_SALES_VAT.balance + REVERSE_CHARGE_MERCHANDISES_PURCHASED_OUT_DK.balance + REVERSE_CHARGE_SERVICES_PURCHASED_OUT_DK.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_sales_tax" model="account.report.line">
                        <field name="name">Salgsmoms (udgående moms)</field>
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
                        <field name="name">Moms af varekøb i udlandet (både EU og lande uden for EU)</field>
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
                        <field name="name">Moms af ydelseskøb i udlandet med omvendt betalingspligt</field>
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
                <field name="name">Fradrag</field>
                <field name="code">VAT_TO_DEDUCED</field>
                <field name="aggregation_formula">PURCHASE_VAT.balance + OIL_AND_BOTTLED_GAS_TAX.balance + ELECTRICITY_TAX.balance + NATURAL_AND_CITY_GAS_TAX.balance + COAL_TAX.balance + CO2_TAX.balance + WATER_TAX.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_deduction_purchase_tax" model="account.report.line">
                        <field name="name">Købsmoms (indgående moms)</field>
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
                        <field name="name">Olie- og flaskegasafgift</field>
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
                        <field name="name">Elafgift</field>
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
                        <field name="name">Naturgas- og bygasafgift</field>
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
                        <field name="name">Kulafgift</field>
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
                        <field name="name">CO2-afgift</field>
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
                        <field name="name">Vandafgift</field>
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
                <field name="name">Momsangivelse (positivt beløb = betale, negativt beløb = penge tilgode)</field>
                <field name="aggregation_formula">DUE_VAT.balance - VAT_TO_DEDUCED.balance</field>
            </record>
            <record id="account_tax_report_line_additional_info" model="account.report.line">
                <field name="name">Supplerende oplysninger</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_section_a_products" model="account.report.line">
                        <field name="name">Rubrik A: varer (EU)</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_section_a_products_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">R-A-V</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_section_a_services" model="account.report.line">
                        <field name="name">Rubrik A = ydelser (EU)</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_section_a_services_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">R-A-Y</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_section_b_product_eu" model="account.report.line">
                        <field name="name">Rubrik B = varer (Indberettes til EU-salg uden moms)</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_section_b_product_eu_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">R-B-MR</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_section_b_products_non_eu" model="account.report.line">
                        <field name="name">Rubrik B = varer (Indberettes ikke til EU-salg uden moms)</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_section_b_products_non_eu_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">R-B-UR</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_section_b_services" model="account.report.line">
                        <field name="name">Rubrik B = ydelser - (EU-salg uden moms)</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_section_b_services_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">R-C-MR</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_section_c" model="account.report.line">
                        <field name="name">Rubrik C - Værdien af andre varer og ydelser (uanset landet)</field>
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

## File: data\account_tax_template_data.xml

```xml
﻿<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Salgsmoms (VAT) -->
    <!-- DK salgsmoms (taxes to set on Sales in DK) -->
    <record id="tax110" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">110</field>
        <field name="name">Salgsmoms 25%, varer</field>
        <field name="description">25%</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8720'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_sales_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8720'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_sales_tax_tag')],
            }),
        ]"/>
    </record>
    <record id="tax120" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">120</field>
        <field name="name">Salgsmoms 25%, ydelser</field>
        <field name="description">25%</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8720'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_sales_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8720'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_sales_tax_tag')],
            }),
        ]"/>
    </record>
    <!-- EU salgsmoms (taxes to set on Sales in EU when outside of DK) -->
    <record id="tax210" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">210</field>
        <field name="name">EU Varesalg (Virksomheder) - momsfritaget</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_section_b_product_eu_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_section_b_product_eu_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tax220" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">220</field>
        <field name="name">EU Ydelsessalg (Virksomheder) - momsfritaget</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_section_b_services_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_section_b_services_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <!-- 3. Lande salgsmoms (taxes to set on Sales outside of EU) -->
    <record id="tax310" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">310</field>
        <field name="name">3. Land Salg Vare / Ydelser</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_section_c_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_section_c_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <!-- Købsmoms (VAT on purchase) -->
    <!-- DK købsmoms (taxes to set on purchase inside DK) -->
    <!-- 2 different taxes for merchandises and services isn't required for local taxes but is required -->
    <!-- for EU related taxes and with fiscal position and we can't map one tax to 2 other taxes because it's  -->
    <!-- a one on one relation. Thus, we set 2 taxes locally. -->
    <record id="tax400" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">400</field>
        <field name="name">Købsmoms 25%, varer</field>
        <field name="description">25%</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
        ]"/>
    </record>
    <record id="tax410" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">410</field>
        <field name="name">Købsmoms 25% indeholdt, varer</field>
        <field name="description">25% indeholdt</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="price_include" eval="True"/>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
        ]"/>
    </record>
    <record id="tax420" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">420</field>
        <field name="name">Købsmoms 25%, ydelser</field>
        <field name="description">25%</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
        ]"/>
    </record>
    <record id="tax425" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">420</field>
        <field name="name">Købsmoms 25% indeholdt, ydelser</field>
        <field name="description">25%</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="price_include" eval="True"/>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
        ]"/>
    </record>
    <record id="tax430" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">430</field>
        <field name="name">Køb omvendt betalingspligt</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8725'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_sales_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_sales_tax_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8725'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
        ]"/>
    </record>
    <record id="tax450" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">450</field>
        <field name="name">Restaurationsmoms 6,25%, købsmoms</field>
        <field name="description">6,25%</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'factor_percent': 25,
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
            (0,0, {
                'factor_percent': 75,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'factor_percent': 25,
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
            (0,0, {
                'factor_percent': 75,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="tax460" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">450</field>
        <field name="name">Restaurationsmoms indeholdt 6,25%, købsmoms</field>
        <field name="description">6,25%</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'factor_percent': 25,
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
            (0,0, {
                'factor_percent': 75,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'factor_percent': 25,
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
            (0,0, {
                'factor_percent': 75,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <!-- EU købsmoms (taxes to set on purchase in EU outside DK) -->
    <record id="tax510" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">510</field>
        <field name="name">EU Varekøb</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_section_a_products_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8730'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_international_purchase_products_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_section_a_products_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8730'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_international_purchase_products_tag')],
            }),
        ]"/>
    </record>
    <record id="tax520" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">520</field>
        <field name="name">EU Ydelseskøb</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_section_a_services_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8731'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_international_purchase_services_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_section_a_services_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8731'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_international_purchase_services_tag')],
            }),
        ]"/>
    </record>
    <!-- 3. Lande købsmoms (taxes to set on purchase outside of EU) -->
    <record id="tax610" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">610</field>
        <field name="name">3. Land Varekøb</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_international_purchase_products_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8730'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_international_purchase_products_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8730'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
        ]"/>
    </record>
    <record id="tax620" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">620</field>
        <field name="name">3. Land Ydelseskøb</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8731'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_international_purchase_services_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8731'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_international_purchase_services_tag')],
            }),
        ]"/>
    </record>
    <!--  All Energy and water taxes  -->
    <!--  oil and bottled gas tax  -->
    <!--  92.83% of the tax is deductible  -->
    <!--  https://skat.dk/data.aspx?oid=2246453  -->
    <record id="tax710" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">710</field>
        <field name="name">Olie- og flaskegasafgift</field>
        <field name="amount">100</field>
        <field name="amount_type">division</field>
        <field name="price_include" eval="True"/>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'factor_percent': 92.83,
                'repartition_type': 'tax',
                'account_id': ref('a8775'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_oil_bottle_tax_tag')],
            }),
            (0,0, {
                'factor_percent': 7.17,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'factor_percent': 92.83,
                'repartition_type': 'tax',
                'account_id': ref('a8775'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_oil_bottle_tax_tag')],
            }),
            (0,0, {
                'factor_percent': 7.17,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <!-- 0.896 cent per kilowatt (to set as quantity) can be deduced -->
    <!-- https://skat.dk/data.aspx?oid=2246453 -->
    <record id="tax720" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">720</field>
        <field name="name">Elafgift</field>
        <field name="amount">0.896</field>
        <field name="amount_type">fixed</field>
        <field name="price_include" eval="True"/>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8775'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_electrical_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8775'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_electrical_tax_tag')],
            }),
        ]"/>
    </record>
    <!--  city gas and oil tax: same than bottled but appears on a different report line  -->
    <!--  92.83% of the tax is deductible  -->
    <!--  https://skat.dk/data.aspx?oid=2246453  -->
    <record id="tax730" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">730</field>
        <field name="name">Naturgas- og bygasafgift</field>
        <field name="amount">100</field>
        <field name="amount_type">division</field>
        <field name="price_include" eval="True"/>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'factor_percent': 92.83,
                'repartition_type': 'tax',
                'account_id': ref('a8780'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_gas_tax_tag')],
            }),
            (0,0, {
                'factor_percent': 7.17,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'factor_percent': 92.83,
                'repartition_type': 'tax',
                'account_id': ref('a8780'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_gas_tax_tag')],
            }),
            (0,0, {
                'factor_percent': 7.17,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <!--  Coal tax: percentage is the same as oil and bottled gas but with it ends up in its own report line  -->
    <record id="tax740" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">740</field>
        <field name="name">Kulafgift</field>
        <field name="amount">100</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'factor_percent': 92.83,
                'repartition_type': 'tax',
                'account_id': ref('a8785'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_coal_tax_tag')],
            }),
            (0,0, {
                'factor_percent': -92.83,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'factor_percent': 92.83,
                'repartition_type': 'tax',
                'account_id': ref('a8785'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_coal_tax_tag')],
            }),
            (0,0, {
                'factor_percent': -92.83,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <!-- CO2-afgift is rarely used anymore, but we include it as inactive so that it can be customized when the need comes -->
    <record id="tax750" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">750</field>
        <field name="name">CO2-afgift</field>
        <field name="active" eval="False"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8785'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_co2_tax_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a4271'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8785'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_co2_tax_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a4271'),
            }),
        ]"/>
    </record>
    <!-- water tax: 6.18 per cubic meter -->
    <!-- https://skat.dk/data.aspx?oid=2246453 -->
    <record id="tax760" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">760</field>
        <field name="name">Vandafgift</field>
        <field name="amount">6.1800</field>
        <field name="amount_type">fixed</field>
        <field name="price_include" eval="True"/>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8795'),
                'plus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_water_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('a8795'),
                'minus_report_expression_ids': [ref('l10n_dk.account_tax_report_line_deduction_water_tax_tag')],
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\l10n_dk_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="dk_chart_template" model="account.chart.template">
        <field name="name">Denmark - Chart of Accounts</field>
        <field name="code_digits">4</field>
        <field name="currency_id" ref="base.DKK"/>
        <field name="use_anglo_saxon" eval="True"/>
        <field name="cash_account_code_prefix">681</field>
        <field name="bank_account_code_prefix">682</field>
        <field name="transfer_account_code_prefix">683</field>
        <field name="country_id" ref="base.dk"/>
    </record>
</odoo>

```

## File: data\l10n_dk_chart_template_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="dk_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="a6610"/>
        <field name="property_account_payable_id" ref="a8440"/>
        <field name="property_account_expense_categ_id" ref="a2010"/>
        <field name="property_account_income_categ_id" ref="a1010"/>
        <field name="property_account_expense_id" ref="a2010"/>
        <field name="property_account_income_id" ref="a1010"/>
        <field name="property_stock_account_input_categ_id" ref="a8450"/>
        <field name="property_stock_account_output_categ_id" ref="a6670"/>
        <field name="property_stock_valuation_account_id" ref="a6530"/>
        <field name="income_currency_exchange_account_id" ref="a4670"/>
        <field name="expense_currency_exchange_account_id" ref="a4770"/>
        <field name="default_pos_receivable_account_id" ref="a6611"/>
        <field name="account_journal_early_pay_discount_loss_account_id" ref="a4760"/>
        <field name="account_journal_early_pay_discount_gain_account_id" ref="a4660"/>
        <field name="property_tax_payable_account_id" ref="a8798"/>
        <field name="property_tax_receivable_account_id" ref="a8798"/>
    </record>
</odoo>

```

## File: migrations\1.1\pre-migrate.py

```python
def migrate(cr, version):
    # Set noupdate property of "account.tax.template" records to False
    cr.execute(
        """UPDATE ir_model_data
              SET noupdate=false
            WHERE module='l10n_dk'
              AND model='account.tax.template'
        """
    )

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


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
            account_vals['tag_ids'].append((4, self.env.ref('l10n_dk.account_tag_liquidity').id))

        return account_vals

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_journal

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.8" y="3.23" width="50.4" height="41.36" maskUnits="userSpaceOnUse">
      <rect x="6.29" y="7.4" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
    </mask>
    <symbol id="c" data-name="account icon" viewBox="0 0 106 106">
      <g style="mask: url(#a)">
        <g>
          <path d="M0,0H106V106H0Z" style="fill: #5a5a64;fill-rule: evenodd"/>
          <path d="M6.06,1.51H98.43q6.06,0,7.57,3V0H0V4.54Q1.52,1.51,6.06,1.51Z" style="fill: #fff;fill-opacity: 0.382999986410141;fill-rule: evenodd"/>
          <path d="M6.06,104.49H98.43q6.06,0,7.57-4.55V106H0V99.94Q1.52,104.49,6.06,104.49Z" style="fill-opacity: 0.382999986410141;fill-rule: evenodd"/>
          <g>
            <path d="M70.38,104.49H6.06C3,104.49,0,103,0,98.43V61.28L28.77,19.69H59.06a77.33,77.33,0,0,0,21.2,13.87c.07,11.31.07,4.86,0,16.17h3.12l.21,36.82Z" style="fill: #393939;fill-rule: evenodd;opacity: 0.324000000953674;isolation: isolate"/>
            <g style="opacity: 0.30000000000000004">
              <g>
                <path d="M68.77,58.54H76c.76,0,1,.12,1,.46v2.45c0,.31-.24.43-.93.43H61.44c-.66,0-.92-.12-.92-.42,0-.83,0-1.67,0-2.51,0-.29.26-.4.92-.41Z"/>
                <path d="M64.33,77.42c.42.39.76.66,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,4.25,4.25,0,0,1-.48-.47c-.14-.15-.26-.31-.49-.6-.32.37-.54.66-.79.91-.53.53-1.08.58-1.5.15s-.36-.94.15-1.45c.26-.26.54-.5.91-.83-.38-.34-.72-.61-1-.91a.9.9,0,0,1,0-1.36.91.91,0,0,1,1.36,0c.29.28.54.6.93,1A12.1,12.1,0,0,1,64,75.18a.91.91,0,0,1,1.36,0,.87.87,0,0,1,0,1.31C65.07,76.79,64.73,77.06,64.33,77.42Z"/>
                <path d="M62.13,66.9c0-.47,0-.88,0-1.28a.92.92,0,0,1,.92-1,.91.91,0,0,1,1,1c0,.41,0,.81,0,1.3h1.14a1.16,1.16,0,0,1,1.22,1c0,.55-.42.85-1.18.86H64.12c0,.49,0,.91,0,1.34a.94.94,0,1,1-1.88,0c0-.41,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88C61.3,66.89,61.68,66.9,62.13,66.9Z"/>
                <path d="M74.31,76H72.23c-.67,0-1-.34-1-.93a.89.89,0,0,1,1-1q2.18,0,4.35,0a1,1,0,1,1,0,1.91c-.74,0-1.47,0-2.21,0Z"/>
                <path d="M74.28,68.61c-.71,0-1.43,0-2.14,0a.86.86,0,0,1-1-.9.85.85,0,0,1,.92-1c1.5,0,3,0,4.48,0a.93.93,0,0,1,1,1,.91.91,0,0,1-1,.91c-.75,0-1.51,0-2.27,0Z"/>
                <path d="M74.36,78.09c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.57-.38.93-1,.94H72.28c-.75,0-1.09-.32-1.09-.94s.37-1,1.09-1,1.39,0,2.08,0Z"/>
                <path d="M81.29,90.55H56.14a4,4,0,0,1-4-4V53.73a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V86.55A4,4,0,0,1,81.29,90.55ZM56.14,53.73V86.55H81.29V53.73Z"/>
              </g>
              <path d="M43.49,83.26H31.8V25.71H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V34.8c-4.55-3-16.66-12.11-19.69-13.63H30.29a2.68,2.68,0,0,0-3,3V84.77a2.68,2.68,0,0,0,3,3H48.45V83.26ZM60.57,25.71l15.14,10.6H60.57Z"/>
            </g>
            <path d="M60.57,18.68H30.29a2.68,2.68,0,0,0-3,3V82.28a2.68,2.68,0,0,0,3,3H48.45V80.77H31.8V23.22H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V32.31C75.71,29.28,63.6,20.2,60.57,18.68Zm0,15.14V23.22l15.14,10.6Z" style="fill: #a8a9ab"/>
            <g>
              <path d="M68.77,55.78H76c.76,0,1,.13,1,.53v2.85c0,.37-.24.5-.93.5q-7.3,0-14.61,0c-.66,0-.92-.14-.92-.48,0-1,0-2,0-2.93,0-.34.26-.47.92-.47Z" style="fill: #a8a9ab"/>
              <path d="M64.33,76.53c.42.38.76.65,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,5.44,5.44,0,0,1-.48-.48c-.14-.14-.26-.31-.49-.59-.32.36-.54.65-.79.91-.53.53-1.08.57-1.5.14s-.36-.94.15-1.45c.26-.26.54-.49.91-.82-.38-.35-.72-.61-1-.92a.9.9,0,0,1,0-1.36.92.92,0,0,1,1.36,0c.29.28.54.61.93,1A13.78,13.78,0,0,1,64,74.28a.91.91,0,0,1,1.36,0,.88.88,0,0,1,0,1.32C65.07,75.89,64.73,76.16,64.33,76.53Z" style="fill: #a8a9ab"/>
              <path d="M62.13,65.88c0-.48,0-.88,0-1.29a1,1,0,1,1,1.91,0c0,.4,0,.81,0,1.3h1.14a1.15,1.15,0,0,1,1.22,1c0,.54-.42.85-1.18.85H64.12c0,.49,0,.92,0,1.34a.94.94,0,1,1-1.88,0c0-.4,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88Z" style="fill: #a8a9ab"/>
              <path d="M74.31,75.11c-.69,0-1.38,0-2.08,0s-1-.35-1-.94a.89.89,0,0,1,1-1q2.18,0,4.35,0a.91.91,0,0,1,1,1,.93.93,0,0,1-1,1c-.74,0-1.47,0-2.21,0Z" style="fill: #a8a9ab"/>
              <path d="M74.28,67.76H72.14a.87.87,0,0,1-1-.9.84.84,0,0,1,.92-1c1.5,0,3,0,4.48,0a.94.94,0,0,1,1,1,.91.91,0,0,1-1,.91H74.28Z" style="fill: #a8a9ab"/>
              <path d="M74.36,77.2c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.56-.38.93-1,.93q-2.12,0-4.23,0c-.75,0-1.09-.32-1.09-.94s.37-.94,1.09-1,1.39,0,2.08,0Z" style="fill: #a8a9ab"/>
              <path d="M81.29,88.06H56.14a4,4,0,0,1-4-4V51.24a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V84.06A4,4,0,0,1,81.29,88.06ZM56.14,51.24V84.06H81.29V51.24Z" style="fill: #a8a9ab"/>
            </g>
          </g>
        </g>
      </g>
    </symbol>
  </defs>
  <g>
    <use width="106" height="106" transform="translate(-0.07 0)" xlink:href="#c"/>
    <rect x="6.2" y="10.57" width="48.45" height="31.57" rx="1" style="fill: #393939;opacity: 0.44;isolation: isolate"/>
    <g style="mask: url(#b)">
      <image width="635" height="481" transform="translate(4.8 3.23) scale(0.08 0.09)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnsAAAIJCAYAAADd3vtdAAAACXBIWXMAAIt8AACLfAFyfqeNAAAJ/0lEQVR4Xu3ZsU1jQQBF0fk2OQWQIEFpFEEzlEAnNEDAJjgkQAgR7TKEkA2ZtdfnxK+AK73tcVzPAfzafpyPq/mwmp2Uj7v7cbi5Xc0AOILdagAAwP9L7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQNj29+l5rkbAD7tt7C8vVquTMt/ex+fL62oGwBFsc06xBwAQ5cYFAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBA2Nm/P4fVBvhpt4395cVqdVLm2/v4fHldzQA4gu1xXM/VCPi2H+fjaj6sZifl4+5+HG5uVzMAjsCNCwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACBM7AEAhIk9AIAwsQcAECb2AADCxB4AQJjYAwAIE3sAAGFiDwAgTOwBAISJPQCAMLEHABAm9gAAwsQeAECY2AMACBN7AABhYg8AIEzsAQCEiT0AgDCxBwAQJvYAAMLEHgBAmNgDAAgTewAAYWIPACDsC5MlKn9s0cx4AAAAAElFTkSuQmCC"/>
    </g>
  </g>
</svg>

```

