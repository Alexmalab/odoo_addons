# Odoo Module: l10n_dk

Category: Localization

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
    'version': '1.0',
    'author': 'Odoo House ApS, VK DATA ApS',
    'website': 'http://odoodanmark.dk',
    'category': 'Localization',
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
        'data/menuitem_data.xml'
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","user_type_id/id","tag_ids/id","chart_template_id/id","reconcile"
"a1010","Salg af vare, m/moms","1010","account.data_account_type_revenue","account_tag_revenue","dk_chart_template","False"
"a1011","Salg af ydelser, m/moms","1011","account.data_account_type_revenue","account_tag_revenue","dk_chart_template","False"
"a1012","Salg af vare, u/moms","1012","account.data_account_type_revenue","account_tag_revenue","dk_chart_template","False"
"a1013","Salg af ydelser, u/moms","1013","account.data_account_type_revenue","account_tag_revenue","dk_chart_template","False"
"a1020","Salg af vare, EU","1020","account.data_account_type_revenue","account_tag_revenue","dk_chart_template","False"
"a1021","Salg af ydelser, EU","1021","account.data_account_type_revenue","account_tag_revenue","dk_chart_template","False"
"a1030","Salg af vare, 3. lande","1030","account.data_account_type_revenue","account_tag_revenue","dk_chart_template","False"
"a1031","Salg af ydelser, 3. lande","1031","account.data_account_type_revenue","account_tag_revenue","dk_chart_template","False"
"a2010","Direkte omkostninger vare, m/moms","2010","account.data_account_type_direct_costs","account_tag_direct_costs","dk_chart_template","False"
"a2011","Direkte omkostninger ydelser, m/moms","2011","account.data_account_type_direct_costs","account_tag_direct_costs","dk_chart_template","False"
"a2012","Direkte omkostninger vare, u/moms","2012","account.data_account_type_direct_costs","account_tag_direct_costs","dk_chart_template","False"
"a2013","Direkte omkostninger ydelser, u/moms","2013","account.data_account_type_direct_costs","account_tag_direct_costs","dk_chart_template","False"
"a2020","Direkte omkostninger vare, EU","2020","account.data_account_type_direct_costs","account_tag_direct_costs","dk_chart_template","False"
"a2021","Direkte omkostninger ydelser, EU","2021","account.data_account_type_direct_costs","account_tag_direct_costs","dk_chart_template","False"
"a2030","Direkte omkostninger vare, 3. lande","2030","account.data_account_type_direct_costs","account_tag_direct_costs","dk_chart_template","False"
"a2031","Direkte omkostninger ydelser, 3. lande","2031","account.data_account_type_direct_costs","account_tag_direct_costs","dk_chart_template","False"
"a2100","Vareforbrug, lagermodul","2100","account.data_account_type_direct_costs","account_tag_direct_costs","dk_chart_template","False"
"a2110","Prisdifferencer","2110","account.data_account_type_direct_costs","account_tag_direct_costs","dk_chart_template","False"
"a2600","Lager, primo","2600","account.data_account_type_direct_costs","account_tag_direct_costs","dk_chart_template","False"
"a2605","Lager, ultimo","2605","account.data_account_type_direct_costs","account_tag_direct_costs","dk_chart_template","False"
"a2610","Lagerændring","2610","account.data_account_type_direct_costs","account_tag_direct_costs","dk_chart_template","False"
"a3010","Produktionsløn","3010","account.data_account_type_expenses","account_tag_staff_costs","dk_chart_template","False"
"a3020","Gager","3020","account.data_account_type_expenses","account_tag_staff_costs","dk_chart_template","False"
"a3610","Arbejdskadeforsikring","3610","account.data_account_type_expenses","account_tag_staff_costs","dk_chart_template","False"
"a3620","Ansvarsforsikring","3620","account.data_account_type_expenses","account_tag_staff_costs","dk_chart_template","False"
"a3630","Dagpengeforsikring","3630","account.data_account_type_expenses","account_tag_staff_costs","dk_chart_template","False"
"a3640","Gruppelivsforsikring","3640","account.data_account_type_expenses","account_tag_staff_costs","dk_chart_template","False"
"a3655","ATP-bidrag","3655","account.data_account_type_expenses","account_tag_staff_costs","dk_chart_template","False"
"a3680","Kursusudgifter","3680","account.data_account_type_expenses","account_tag_staff_costs","dk_chart_template","False"
"a3690","Personaleudgifter","3690","account.data_account_type_expenses","account_tag_staff_costs","dk_chart_template","False"
"a3695","Fortæring","3695","account.data_account_type_expenses","account_tag_staff_costs","dk_chart_template","False"
"a3700","Befordringsgodtgørelse","3700","account.data_account_type_expenses","account_tag_staff_costs","dk_chart_template","False"
"a3710","Skattefrie godtgørelser","3710","account.data_account_type_expenses","account_tag_staff_costs","dk_chart_template","False"
"a3740","Arbejdstøj","3740","account.data_account_type_expenses","account_tag_staff_costs","dk_chart_template","False"
"a3810","Refunderet sygedagpenge","3810","account.data_account_type_expenses","account_tag_staff_costs","dk_chart_template","False"
"a3820","Løntilskud","3820","account.data_account_type_expenses","account_tag_staff_costs","dk_chart_template","False"
"a3840","Godtgørelse ATP","3840","account.data_account_type_expenses","account_tag_staff_costs","dk_chart_template","False"
"a4010","Restaurationsbesøg","4010","account.data_account_type_expenses","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4020","Anden repræsentation","4020","account.data_account_type_expenses","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4036","Rejseudgifter","4036","account.data_account_type_expenses","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4040","Hotelophold","4040","account.data_account_type_expenses","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4060","Annoncer","4060","account.data_account_type_expenses","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4065","Erhvervsnetværk","4065","account.data_account_type_expenses","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4070","Dekoration/skilte","4070","account.data_account_type_expenses","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4071","Udstillinger","4071","account.data_account_type_expenses","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4073","Tryksager/brochurer","4073","account.data_account_type_expenses","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4074","Reklame-emballage","4074","account.data_account_type_expenses","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4076","Emballage,salgsomkostninger","4076","account.data_account_type_expenses","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4080","Fragt, porto","4080","account.data_account_type_expenses","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4081","Fragt, porto u/moms","4081","account.data_account_type_expenses","account_tag_sales_promotions_costs","dk_chart_template","False"
"a4102","Brændstof vare/lastbil","4102","account.data_account_type_expenses","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4103","Vægtafgift vare/lastbil","4103","account.data_account_type_expenses","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4104","Forsikring vare/lastbil","4104","account.data_account_type_expenses","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4105","Vedligeholdelse vare/lastbil","4105","account.data_account_type_expenses","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4108","Redningskorps vare/lastbil","4108","account.data_account_type_expenses","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4120","Leje vare/lastbil","4120","account.data_account_type_expenses","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4152","Brændstof personbil","4152","account.data_account_type_expenses","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4153","Vægtafgift personbil","4153","account.data_account_type_expenses","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4154","Forsikring personbil","4154","account.data_account_type_expenses","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4155","Vedligeholdelse personbil","4155","account.data_account_type_expenses","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4158","Redningskorps personbil","4158","account.data_account_type_expenses","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4170","Leje af personbil","4170","account.data_account_type_expenses","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4189","Privat andel af bil, drift af biler","4189","account.data_account_type_expenses","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4190","Kørsel i privat bil","4190","account.data_account_type_expenses","account_tag_cars_equipment_costs","dk_chart_template","False"
"a4205","Leje/leasing af driftsmidler","4205","account.data_account_type_expenses","account_tag_rooms_costs","dk_chart_template","False"
"a4235","Vedligeholdelse, driftsmiddelomkostninger","4235","account.data_account_type_expenses","account_tag_rooms_costs","dk_chart_template","False"
"a4240","Småanskaffelser","4240","account.data_account_type_expenses","account_tag_rooms_costs","dk_chart_template","False"
"a4241","Småanskaffelser o/11.000","4241","account.data_account_type_expenses","account_tag_rooms_costs","dk_chart_template","False"
"a4255","Husleje m/moms","4255","account.data_account_type_expenses","account_tag_rooms_costs","dk_chart_template","False"
"a4256","Husleje u/moms","4256","account.data_account_type_expenses","account_tag_rooms_costs","dk_chart_template","False"
"a4260","Lagerleje","4260","account.data_account_type_expenses","account_tag_rooms_costs","dk_chart_template","False"
"a4265","Garageleje","4265","account.data_account_type_expenses","account_tag_rooms_costs","dk_chart_template","False"
"a4271","Varme","4271","account.data_account_type_expenses","account_tag_rooms_costs","dk_chart_template","False"
"a4275","El og vand","4275","account.data_account_type_expenses","account_tag_rooms_costs","dk_chart_template","False"
"a4276","Refunderet energiafgift","4276","account.data_account_type_expenses","account_tag_rooms_costs","dk_chart_template","False"
"a4285","Privat andel el, vand","4285","account.data_account_type_expenses","account_tag_rooms_costs","dk_chart_template","False"
"a4291","Vedligeholdelse, lokaleomkostninger","4291","account.data_account_type_expenses","account_tag_rooms_costs","dk_chart_template","False"
"a4295","Rengøring","4295","account.data_account_type_expenses","account_tag_rooms_costs","dk_chart_template","False"
"a4296","Renovation","4296","account.data_account_type_expenses","account_tag_rooms_costs","dk_chart_template","False"
"a4302","Kontorartikler/tryksager","4302","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4307","Inkassoomkostninger","4307","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4308","Vedligeholdelse inventar","4308","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4310","EDB-udgifter","4310","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4311","Mail/Webhotel","4311","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4312","Løbende backup","4312","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4316","Telefon","4316","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4322","Privat andel telefon","4322","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4325","Internet","4325","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4327","Porto","4327","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4335","Regnskabsassistance","4335","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4338","Databehandling","4338","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4340","Advokathonorar","4340","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4343","Konsulentbistand","4343","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4350","Forsikringer","4350","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4351","Vagtværn/alarmsystem","4351","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4360","Faglitteratur/tidsskrifter","4360","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4370","Kontingenter","4370","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4381","Tab på debitorer","4381","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4392","Øvrige kapacitetsudgifter","4392","account.data_account_type_expenses","account_tag_administration_costs","dk_chart_template","False"
"a4451","Driftsafskrivninger, grunde og bygninger","4451","account.data_account_type_depreciation","account_tag_depreciation","dk_chart_template","False"
"a4454","Driftsafskrivninger, andre anlæg, driftsmateriel og inventar","4454","account.data_account_type_depreciation","account_tag_depreciation","dk_chart_template","False"
"a4651","Renteindtægter, bank","4651","account.data_account_type_other_income","account_tag_financial_income","dk_chart_template","False"
"a4664","Renteindtægter, debitorer","4664","account.data_account_type_other_income","account_tag_financial_income","dk_chart_template","False"
"a4670","Kursgevinster","4670","account.data_account_type_other_income","account_tag_financial_income","dk_chart_template","False"
"a4680","Gebyrer, indtægter","4680","account.data_account_type_other_income","account_tag_financial_income","dk_chart_template","False"
"a4730","Renteudgifter, bank","4730","account.data_account_type_expenses","account_tag_financial_expenses","dk_chart_template","False"
"a4740","Renteudgifter, realkredit","4740","account.data_account_type_expenses","account_tag_financial_expenses","dk_chart_template","False"
"a4743","Renteudgifter, kreditorer","4743","account.data_account_type_expenses","account_tag_financial_expenses","dk_chart_template","False"
"a4744","Renteudgifter, kontrakter","4744","account.data_account_type_expenses","account_tag_financial_expenses","dk_chart_template","False"
"a4745","Renteudgifter, pantebreve","4745","account.data_account_type_expenses","account_tag_financial_expenses","dk_chart_template","False"
"a4746","Renteudgifter, ej fradrag","4746","account.data_account_type_expenses","account_tag_financial_expenses","dk_chart_template","False"
"a4750","Garantiprovision","4750","account.data_account_type_expenses","account_tag_financial_expenses","dk_chart_template","False"
"a4755","Gebyrer, udgifter","4755","account.data_account_type_expenses","account_tag_financial_expenses","dk_chart_template","False"
"a4760","Kasserabat","4760","account.data_account_type_expenses","account_tag_financial_expenses","dk_chart_template","False"
"a4766","Låneomkostninger","4766","account.data_account_type_expenses","account_tag_financial_expenses","dk_chart_template","False"
"a4770","Kurstab","4770","account.data_account_type_expenses","account_tag_financial_expenses","dk_chart_template","False"
"a4805","Fortjeneste, salg anlæg","4805","account.data_account_type_other_income","account_tag_extraordinary_income","dk_chart_template","False"
"a4819","Indtægter vedr. tidl. år","4819","account.data_account_type_other_income","account_tag_extraordinary_income","dk_chart_template","False"
"a4847","Andre ekstraordinære indtægter","4847","account.data_account_type_other_income","account_tag_extraordinary_income","dk_chart_template","False"
"a4854","Tab, salg anlæg","4854","account.data_account_type_expenses","account_tag_extraordinary_expenses","dk_chart_template","False"
"a4890","Udgifter vedr. tidl. år","4890","account.data_account_type_expenses","account_tag_extraordinary_expenses","dk_chart_template","False"
"a4895","Andre ekstraordinære udgifter","4895","account.data_account_type_expenses","account_tag_extraordinary_expenses","dk_chart_template","False"
"a5201","Kostpris, primo,goodwill, licenser mm.","5201","account.data_account_type_non_current_assets","account_tag_Goodwill_licenses_etc","dk_chart_template","False"
"a5202","Tilgang i årets løb, goodwill, licenser mm.","5202","account.data_account_type_non_current_assets","account_tag_Goodwill_licenses_etc","dk_chart_template","False"
"a5203","Afgang i årets løb,goodwill, licenser mm.","5203","account.data_account_type_non_current_assets","account_tag_Goodwill_licenses_etc","dk_chart_template","False"
"a5204","Afskrivninger primo, goodwill, licenser mm.","5204","account.data_account_type_non_current_assets","account_tag_Goodwill_licenses_etc","dk_chart_template","False"
"a5205","Afskrivninger på afgang, goodwill, licenser mm.","5205","account.data_account_type_non_current_assets","account_tag_Goodwill_licenses_etc","dk_chart_template","False"
"a5206","Årsafskrivning, goodwill, licenser mm.","5206","account.data_account_type_non_current_assets","account_tag_Goodwill_licenses_etc","dk_chart_template","False"
"a5511","Kostpris, primo, grunde og bygninger","5511","account.data_account_type_fixed_assets","account_tag_land_buildings","dk_chart_template","False"
"a5512","Tilgang i årets løb,grunde og bygninger","5512","account.data_account_type_fixed_assets","account_tag_land_buildings","dk_chart_template","False"
"a5513","Afgang i årets løb,grunde og bygninger","5513","account.data_account_type_fixed_assets","account_tag_land_buildings","dk_chart_template","False"
"a5514","Afskrivninger primo, grunde og bygninger","5514","account.data_account_type_fixed_assets","account_tag_land_buildings","dk_chart_template","False"
"a5515","Afskrivninger på afgang, grunde og bygninger","5515","account.data_account_type_fixed_assets","account_tag_land_buildings","dk_chart_template","False"
"a5516","Årsafskrivning, grunde og bygninger","5516","account.data_account_type_fixed_assets","account_tag_land_buildings","dk_chart_template","False"
"a5701","Kostpris, primo, andre anlæg, driftsmateriel og inventar","5701","account.data_account_type_fixed_assets","account_tag_other_equipment_tools_inventory","dk_chart_template","False"
"a5702","Tilgang i årets løb, andre anlæg, driftsmateriel og inventar","5702","account.data_account_type_fixed_assets","account_tag_other_equipment_tools_inventory","dk_chart_template","False"
"a5703","Afgang i årets løb, andre anlæg, driftsmateriel og inventar","5703","account.data_account_type_fixed_assets","account_tag_other_equipment_tools_inventory","dk_chart_template","False"
"a5704","Afskrivninger primo, andre anlæg, driftsmateriel og inventar","5704","account.data_account_type_fixed_assets","account_tag_other_equipment_tools_inventory","dk_chart_template","False"
"a5705","Afskrivninger på afgang, andre anlæg, driftsmateriel og inventar","5705","account.data_account_type_fixed_assets","account_tag_other_equipment_tools_inventory","dk_chart_template","False"
"a5706","Årsafskrivning, andre anlæg, driftsmateriel og inventar","5706","account.data_account_type_fixed_assets","account_tag_other_equipment_tools_inventory","dk_chart_template","False"
"a6510","Råvarer","6510","account.data_account_type_current_assets","account_tag_inventories","dk_chart_template","False"
"a6520","Varer under fremstilling","6520","account.data_account_type_current_assets","account_tag_inventories","dk_chart_template","False"
"a6525","Fremstillede færdigvarer","6525","account.data_account_type_current_assets","account_tag_inventories","dk_chart_template","False"
"a6530","Varelager","6530","account.data_account_type_current_assets","account_tag_inventories","dk_chart_template","False"
"a6540","Igangværende arbejder","6540","account.data_account_type_current_assets","account_tag_work_in_progress","dk_chart_template","False"
"a6580","Forudbetaling for varer","6580","account.data_account_type_prepayments","account_tag_prepayments","dk_chart_template","False"
"a6610","Debitorer","6610","account.data_account_type_receivable","account_tag_receivable","dk_chart_template","True"
"a6611","Debitorer (PoS)","6611","account.data_account_type_receivable","account_tag_receivable","dk_chart_template","True"
"a6640","Andre tilgodehavender","6640","account.data_account_type_current_assets","account_tag_other_receivables","dk_chart_template","False"
"a6650","Kundekort/Kreditkort","6650","account.data_account_type_current_assets","account_tag_other_receivables","dk_chart_template","False"
"a6655","Depositum","6655","account.data_account_type_non_current_assets","account_tag_other_receivables","dk_chart_template","False"
"a6670","Varer leveret, faktura ikke sendt","6670","account.data_account_type_current_assets","account_tag_other_receivables","dk_chart_template","True"
"a6680","Periodeafgrænsningsposter, tilgodehavender","6680","account.data_account_type_current_assets","account_tag_other_receivables","dk_chart_template","True"
"a6860","Bundne likvider","6860","account.data_account_type_liquidity","account_tag_liquidity","dk_chart_template","True"
"a7210","Egenkapital, primo","7210","account.data_account_type_equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7220","Indskud","7220","account.data_account_type_equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7230","Hævet privat","7230","account.data_account_type_equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7240","B-skatter og AM-bidrag","7240","account.data_account_type_equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7245","Restskat og overskydende skat","7245","account.data_account_type_equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7250","Forbrug af egne varer, egenkapital","7250","account.data_account_type_equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7260","Privat andel af bil, egenkapital","7260","account.data_account_type_equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7280","Privat andel, telefon, egenkapital","7280","account.data_account_type_equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7290","Privat andel i øvrigt, egenkapital","7290","account.data_account_type_equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7299","Overført resultat","7299","account.data_account_type_equity","account_tag_equity_individual_enterprises","dk_chart_template","False"
"a7410","Virksomhedskapital","7410","account.data_account_type_equity","account_tag_equity_corporations","dk_chart_template","False"
"a7420","Overkurs ved emission","7420","account.data_account_type_equity","account_tag_equity_corporations","dk_chart_template","False"
"a7430","Reserve for opskrivning","7430","account.data_account_type_equity","account_tag_equity_corporations","dk_chart_template","False"
"a7440","Andre reserver","7440","account.data_account_type_equity","account_tag_equity_corporations","dk_chart_template","False"
"a7450","Overført resultat","7450","account.data_account_type_equity","account_tag_equity_corporations","dk_chart_template","False"
"a7460","Forslag til udbytte","7460","account.data_account_type_equity","account_tag_equity_corporations","dk_chart_template","False"
"a7705","Realkreditinstitutter","7705","account.data_account_type_non_current_liabilities","account_tag_credit_institutions","dk_chart_template","False"
"a7710","Private pantebreve","7710","account.data_account_type_non_current_liabilities","account_tag_credit_institutions","dk_chart_template","False"
"a7720","Langfristede banklån","7720","account.data_account_type_non_current_liabilities","account_tag_credit_institutions","dk_chart_template","False"
"a8400","Kassekredit","8400","account.data_account_type_non_current_liabilities","account_tag_credit_institutions","dk_chart_template","False"
"a8410","Anden bankgæld","8410","account.data_account_type_non_current_liabilities","account_tag_bank_loans","dk_chart_template","False"
"a8430","Modtagne forudbetalinger","8430","account.data_account_type_current_liabilities","account_tag_bank_loans","dk_chart_template","False"
"a8440","Kreditorer","8440","account.data_account_type_payable","account_tag_payable","dk_chart_template","True"
"a8450","Vare modtaget, faktura ikke modtaget","8450","account.data_account_type_current_liabilities","account_tag_payable","dk_chart_template","True"
"a8475","Revisorhonorar","8475","account.data_account_type_current_liabilities","account_tag_payable","dk_chart_template","False"
"a8720","Salgsmoms (udgående moms)","8720","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8725","Erhvervelsesmoms med omvendt betlingspligt","8725","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8730","Moms af varekøb i udlandet (Både EU og 3. lande)","8730","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8731","Moms af ydelseskøb i udlandet med omvendt betalingspligt","8731","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8739","Importmoms, modkonto","8739","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8740","Købsmoms (indgående moms)","8740","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8750","EU købsværdi, Rubrik A - vare","8750","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8751","EU købsværdi, Rubrik A - ydelser","8751","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8754","EU salgsværdi, Rubrik B - vare","8754","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8755","EU salgsværdi, Rubrik B - ydelser","8755","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8758","Salgsværdier 3.lande - Rubrik C","8758","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8759","Salgsværdier med omvendt betalingspligt - Rubrik C","8759","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8769","Værdier, modkonto","8769","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8770","Olie- og flaskegasafgift","8770","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8775","Elafgift","8775","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8780","Naturgas- og bygasafgift","8780","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8785","Kulafgift","8785","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8790","CO-2 afgift","8790","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8795","Vandafgift","8795","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8798","Afregning af moms","8798","account.data_account_type_current_liabilities","account_tag_vat_taxes","dk_chart_template","False"
"a8801","Skyldig A-skat","8801","account.data_account_type_current_liabilities","account_tag_Payroll_related_liabilities","dk_chart_template","False"
"a8805","Skyldig AM-bidrag","8805","account.data_account_type_current_liabilities","account_tag_Payroll_related_liabilities","dk_chart_template","False"
"a8810","Skyldig ATP","8810","account.data_account_type_current_liabilities","account_tag_Payroll_related_liabilities","dk_chart_template","False"
"a8815","Skyldige feriepenge","8815","account.data_account_type_current_liabilities","account_tag_Payroll_related_liabilities","dk_chart_template","False"
"a8820","Afsat skyldige feriepenge","8820","account.data_account_type_current_liabilities","account_tag_Payroll_related_liabilities","dk_chart_template","False"
"a8840","Skyldig lån","8840","account.data_account_type_current_liabilities","account_tag_Payroll_related_liabilities","dk_chart_template","False"
"a8855","Periodeafgrænsningsposter, gæld","8855","account.data_account_type_current_liabilities","account_tag_other_current_liabilities","dk_chart_template","False"
"a8860","Andre skyldige omk.","8860","account.data_account_type_current_liabilities","account_tag_other_current_liabilities","dk_chart_template","False"

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
    <function model="account.chart.template" name="try_loading_for_current_company">
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
        <field name="name">Danmark (Virkomshed)</field>
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
    <!--  Due VAT  -->
    <record id="account_tax_report_line_sales_group" model="account.tax.report.line">
        <field name="name">Skyldig moms</field>
        <field name="code">DUE_VAT</field>
        <field name="sequence" eval="1"/>
        <field name="country_id" ref="base.dk"/>
    </record>
    <!--  classic VAT on sales  -->
    <record id="account_tax_report_line_sales_tax" model="account.tax.report.line">
        <field name="name">Salgsmoms (udgående moms)</field>
        <field name="code">SALES_VAT</field>
        <field name="tag_name">UM</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="l10n_dk.account_tax_report_line_sales_group"/>
        <field name="country_id" ref="base.dk"/>
    </record>
    <!--  VAT on goods purchases abroad with reverse charge  -->
    <record id="account_tax_report_line_international_purchase_products" model="account.tax.report.line">
        <field name="name">Moms af varekøb i udlandet (både EU og lande uden for EU)</field>
        <field name="code">REVERSE_CHARGE_MERCHANDISES_PURCHASED_OUT_DK</field>
        <field name="tag_name">MVU</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="l10n_dk.account_tax_report_line_sales_group"/>
        <field name="country_id" ref="base.dk"/>
    </record>
    <!--  VAT on service purchases abroad with reverse charge  -->
    <record id="account_tax_report_line_international_purchase_services" model="account.tax.report.line">
        <field name="name">Moms af ydelseskøb i udlandet med omvendt betalingspligt</field>
        <field name="code">REVERSE_CHARGE_SERVICES_PURCHASED_OUT_DK</field>
        <field name="tag_name">MYUOB</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="l10n_dk.account_tax_report_line_sales_group"/>
        <field name="country_id" ref="base.dk"/>
    </record>
    
    <!--  Deduction VAT  -->
    <record id="account_tax_report_line_deduction_group" model="account.tax.report.line">
        <field name="name">Fradrag</field>
        <field name="code">VAT_TO_DEDUCED</field>
        <field name="sequence" eval="2"/>
        <field name="country_id" ref="base.dk"/>
    </record>
    <record id="account_tax_report_line_deduction_purchase_tax" model="account.tax.report.line">
        <field name="name">Købsmoms (indgående moms)</field>
        <field name="code">PURCHASE_VAT</field>
        <field name="tag_name">KM</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="l10n_dk.account_tax_report_line_deduction_group"/>
        <field name="country_id" ref="base.dk"/>
    </record>
    <record id="account_tax_report_line_deduction_oil_bottle_tax" model="account.tax.report.line">
        <field name="name">Olie- og flaskegasafgift</field>
        <field name="code">OIL_AND_BOTTLED_GAS_TAX</field>
        <field name="tag_name">OFA</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="l10n_dk.account_tax_report_line_deduction_group"/>
        <field name="country_id" ref="base.dk"/>
    </record>
    <record id="account_tax_report_line_deduction_electrical_tax" model="account.tax.report.line">
        <field name="name">Elafgift</field>
        <field name="code">ELECTRICITY_TAX</field>
        <field name="tag_name">EA</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="l10n_dk.account_tax_report_line_deduction_group"/>
        <field name="country_id" ref="base.dk"/>
    </record>
    <record id="account_tax_report_line_deduction_gas_tax" model="account.tax.report.line">
        <field name="name">Naturgas- og bygasafgift</field>
        <field name="code">NATURAL_AND_CITY_GAS_TAX</field>
        <field name="tag_name">NOBA</field>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="l10n_dk.account_tax_report_line_deduction_group"/>
        <field name="country_id" ref="base.dk"/>
    </record>
    <record id="account_tax_report_line_deduction_coal_tax" model="account.tax.report.line">
        <field name="name">Kulafgift</field>
        <field name="code">COAL_TAX</field>
        <field name="tag_name">KA</field>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="l10n_dk.account_tax_report_line_deduction_group"/>
        <field name="country_id" ref="base.dk"/>
    </record>
    <record id="account_tax_report_line_deduction_co2_tax" model="account.tax.report.line">
        <field name="name">CO2-afgift</field>
        <field name="code">CO2_TAX</field>
        <field name="tag_name">CO2A</field>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="l10n_dk.account_tax_report_line_deduction_group"/>
        <field name="country_id" ref="base.dk"/>
    </record>
    <record id="account_tax_report_line_deduction_water_tax" model="account.tax.report.line">
        <field name="name">Vandafgift</field>
        <field name="code">WATER_TAX</field>
        <field name="tag_name">VA</field>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="l10n_dk.account_tax_report_line_deduction_group"/>
        <field name="country_id" ref="base.dk"/>
    </record>

    <!--  Report Results  -->
    <record id="account_tax_report_line_vat_statement" model="account.tax.report.line">
        <field name="name">Momsangivelse (positivt beløb = betale, negativt beløb = penge tilgode)</field>
        <field name="sequence" eval="3"/>
        <field name="country_id" ref="base.dk"/>
        <field name="formula">DUE_VAT - VAT_TO_DEDUCED</field>
    </record>

    <!--  additional required information  -->
    <record id="account_tax_report_line_additional_info" model="account.tax.report.line">
        <field name="name">Supplerende oplysninger</field>
        <field name="sequence" eval="4"/>
        <field name="country_id" ref="base.dk"/>
        <field name="formula">None</field>
    </record>
    <!--  Notes: "in EU" implicitly also mean "out of DK"  -->
    <!--  Bought merchandises base value in EU-->
    <record id="account_tax_report_line_section_a_products" model="account.tax.report.line">
        <!--  Rubrik A = varer - Værdien uden moms af varekøb i andre EU-lande (EU-erhvervelser)-->
        <field name="name">Rubrik A: varer (EU)</field>
        <field name="tag_name">R-A-V</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="l10n_dk.account_tax_report_line_additional_info"/>
        <field name="country_id" ref="base.dk"/>
    </record>
    <!--  Bought services base value in EU -->
    <record id="account_tax_report_line_section_a_services" model="account.tax.report.line">
        <field name="name">Rubrik A = ydelser (EU)</field>
        <field name="tag_name">R-A-Y</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="l10n_dk.account_tax_report_line_additional_info"/>
        <field name="country_id" ref="base.dk"/>
    </record>
    <!--  Merchandises base value sold in EU and reported under the no-VAT system (~reverse charge system)-->
    <record id="account_tax_report_line_section_b_product_eu" model="account.tax.report.line">
        <field name="name">Rubrik B = varer (Indberettes til EU-salg uden moms)</field>
        <field name="tag_name">R-B-MR</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="l10n_dk.account_tax_report_line_additional_info"/>
        <field name="country_id" ref="base.dk"/>
    </record>
    <!-- Merchandises base value sold in EU without VAT (and thus not reported under the VAT system) to entity/people not subject to VAT -->
    <record id="account_tax_report_line_section_b_products_non_eu" model="account.tax.report.line">
        <field name="name">Rubrik B = varer (Indberettes ikke til EU-salg uden moms)</field>
        <field name="tag_name">R-B-UR</field>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="l10n_dk.account_tax_report_line_additional_info"/>
        <field name="country_id" ref="base.dk"/>
    </record>
    <!--  Reported services base value sold in EU without VAT -->
    <record id="account_tax_report_line_section_b_services" model="account.tax.report.line">
        <field name="name">Rubrik B = ydelser - (EU-salg uden moms)</field>
        <field name="tag_name">R-C-MR</field>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="l10n_dk.account_tax_report_line_additional_info"/>
        <field name="country_id" ref="base.dk"/>
    </record>
    <!--  Any other base value sold without tax (e.g. exportation out of EU) (amount that would go in any of the previous Rubrik) -->
    <record id="account_tax_report_line_section_c" model="account.tax.report.line">
        <field name="name">Rubrik C - Værdien af andre varer og ydelser (uanset landet)</field>
        <field name="tag_name">R-C-UR</field>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="l10n_dk.account_tax_report_line_additional_info"/>
        <field name="country_id" ref="base.dk"/>
    </record>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
﻿<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a8720'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_sales_tax')],
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
                'account_id': ref('a8720'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_sales_tax')],
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a8720'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_sales_tax')],
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
                'account_id': ref('a8720'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_sales_tax')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_section_b_product_eu')],
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
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_section_b_product_eu')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_section_b_services')],
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
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_section_b_services')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_section_c')],
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
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_section_c')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
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
                'account_id': ref('a8740'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
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
                'account_id': ref('a8740'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
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
                'account_id': ref('a8740'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
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
                'account_id': ref('a8740'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8725'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_sales_tax')],
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
                'account_id': ref('a8740'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_sales_tax')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8725'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
            }),
        ]"/>
    </record>
    <record id="tax450" model="account.tax.template">
        <field name="chart_template_id" ref="dk_chart_template"/>
        <field name="sequence">450</field>
        <field name="name">Restaurationsmoms 6,25%, købsmoms</field>
        <field name="description">6,25%</field>
        <field name="amount">6.25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
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
                'account_id': ref('a8740'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_section_a_products')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8730'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_international_purchase_products')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_section_a_products')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8730'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_international_purchase_products')],
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
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_section_a_services')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8731'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_international_purchase_services')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_section_a_services')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8731'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_international_purchase_services')],
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_international_purchase_products')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8730'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
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
                'account_id': ref('a8740'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_international_purchase_products')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8730'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a8740'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8731'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_international_purchase_services')],
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
                'account_id': ref('a8740'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_purchase_tax')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a8731'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_international_purchase_services')],
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
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 92.83,
                'repartition_type': 'tax',
                'account_id': ref('a8775'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_oil_bottle_tax')],
            }),
            (0,0, {
                'factor_percent': -92.83,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 92.83,
                'repartition_type': 'tax',
                'account_id': ref('a8775'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_oil_bottle_tax')],
            }),
            (0,0, {
                'factor_percent': -92.83,
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a8775'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_electrical_tax')],
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
                'account_id': ref('a8775'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_electrical_tax')],
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
        <field name="active" eval="False"/>
        <field name="amount">100</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 92.83,
                'repartition_type': 'tax',
                'account_id': ref('a8780'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_gas_tax')],
            }),
            (0,0, {
                'factor_percent': -92.83,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 92.83,
                'repartition_type': 'tax',
                'account_id': ref('a8780'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_gas_tax')],
            }),
            (0,0, {
                'factor_percent': -92.83,
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 92.83,
                'repartition_type': 'tax',
                'account_id': ref('a8785'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_coal_tax')],
            }),
            (0,0, {
                'factor_percent': -92.83,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 92.83,
                'repartition_type': 'tax',
                'account_id': ref('a8785'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_coal_tax')],
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a8785'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_co2_tax')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('a4271'),
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
                'account_id': ref('a8785'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_co2_tax')],
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a8795'),
                'plus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_water_tax')],
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
                'account_id': ref('a8795'),
                'minus_report_line_ids': [ref('l10n_dk.account_tax_report_line_deduction_water_tax')],
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
    </record>
</odoo>

```

## File: data\menuitem_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_dk_statements_menu" name="Denmark" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_user"/>
</odoo>

```

## File: models\account_chart_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class AccountChartTemplate(models.Model):
    _inherit = 'account.chart.template'

    @api.model
    def _prepare_transfer_account_for_direct_creation(self, name, company):
        res = super(AccountChartTemplate, self)._prepare_transfer_account_for_direct_creation(name, company)
        if company.country_id.code == 'DK':
            account_tag_liquidity = self.env.ref('l10n_dk.account_tag_liquidity')
            res['tag_ids'] = [(6, 0, account_tag_liquidity.ids)]
            res['name'] = 'Bank i transfer'
        return res

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    @api.model
    def _prepare_liquidity_account(self, name, company, currency_id, type):
        account_vals = super(AccountJournal, self)._prepare_liquidity_account(name, company, currency_id, type)

        if company.country_id.code == 'DK':
            # Ensure the newly liquidity accounts have the right account tag in order to be part
            # of the Danish financial reports.
            xml_id = self.env.ref('l10n_dk.account_tag_liquidity').id
            account_vals['tag_ids'] = [(6, 0, [xml_id])]

        return account_vals

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_journal
from . import account_chart_template

```

