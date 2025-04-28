# Odoo Module: l10n_nl

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2016 Onestein (<http://www.onestein.eu>).

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2016 Onestein (<http://www.onestein.eu>).

{
    'name': 'Netherlands - Accounting',
    'version': '3.3',
    'category': 'Accounting/Localizations/Account Charts',
    'author': 'Onestein',
    'website': 'http://www.onestein.eu',
    'depends': [
        'account',
        'base_iban',
        'base_vat',
    ],
    'data': [
        'data/account_account_tag.xml',
        'data/account_chart_template.xml',
        'data/account.account.template.csv',
        'data/account_chart_template_post_data.xml',
        'data/account_tax_group_data.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_template.xml',
        'data/account_fiscal_position_template.xml',
        'data/account_fiscal_position_tax_template.xml',
        'data/account_fiscal_position_account_template.xml',
        'data/account_chart_template_data.xml',
        'views/res_partner_views.xml',
        'views/res_company_views.xml',
        'views/res_config_settings_view.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","account_type","chart_template_id/id","tag_ids/id","reconcile"
"0010","Aanschafwaarde Goodwill","0010","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_17","False"
"0019","Afschrijving Goodwill","0019","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_17","False"
"0020","Aanschafwaarde overige immateriele vaste activa","0020","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_17","False"
"0029","Afschrijving overige immateriele vaste activa","0029","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_17","False"
"0101","Aanschafwaarde Grond ","0101","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0110","Aanschafwaarde Gebouwen","0110","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0119","Afschrijving Gebouwen","0119","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0120","Aanschafwaarde Winkels","0120","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0129","Afschrijving Winkels","0129","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0130","Aanschafwaarde Verbouwingen","0130","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0139","Afschrijving Verbouwingen","0139","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0210","Aanschafwaarde Machines 1","0210","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0219","Afschrijving Machines 1","0219","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0220","Aanschafwaarde Machines 2","0220","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0229","Afschrijving Machines 2","0229","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0310","Aanschafwaarde Bedrijfsinventaris","0310","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0319","Afschrijving Bedrijfsinventaris","0319","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0320","Aanschafwaarde Magazijninventaris","0320","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0329","Afschrijving Magazijninventaris","0329","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0330","Aanschafwaarde Fabrieksinventaris","0330","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0339","Afschrijving Fabrieksinventaris","0339","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0340","Aanschafwaarde Gereedschappen","0340","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0349","Afschrijving Gereedschappen","0349","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0350","Aanschafwaarde Kantoorinventaris","0350","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0359","Afschrijving Kantoorinventaris","0359","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0360","Aanschafwaarde Kantoormachines","0360","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0369","Afschrijving Kantoormachines","0369","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0370","Aanschafwaarde Kantine-inventaris","0370","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0379","Afschrijving Kantine-inventaris","0379","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0410","Aanschafwaarde Personenauto's","0410","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0419","Afschrijving Personenauto's","0419","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0420","Aanschafwaarde overige vervoersmiddelen","0420","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0429","Afschrijving overige vervoersmiddelen","0429","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0430","Aanschafwaarde Vrachtauto's","0430","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0439","Afschrijving Vrachtauto's","0439","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_23","False"
"0510","Deelneming 1","0510","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_3","False"
"0520","Deelneming 2","0520","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_3","False"
"0530","Deelneming 3","0530","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_3","False"
"0551","Vorderingen op deelneming 1","0551","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_3","False"
"0552","Vorderingen op deelneming 2","0552","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_3","False"
"0553","Vorderingen op deelneming 3","0553","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_3","False"
"0561","Hypotheken u/g 1","0561","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_3","False"
"0562","Hypotheken u/g 2","0562","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_3","False"
"0563","Hypotheken u/g 3","0563","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_3","False"
"0565","Leningen u/g 1","0565","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_3","False"
"0566","Leningen u/g 2","0566","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_3","False"
"0567","Leningen u/g 3","0567","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_3","False"
"0570","Waarborgsommen","0570","asset_fixed","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_3","False"
"0610","Aandelenkapitaal","0610","equity","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_24","False"
"0620","Wettelijke reserves","0620","equity","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_9","False"
"0630","Agioreserve","0630","equity","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_9","False"
"0640","Algemene reserve","0640","equity","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_9","False"
"0650","Overige reserves","0650","equity","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_9","False"
"0660","Herwaarderingsreserve","0660","equity","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_9","False"
"0710","Voorziening pensioenverplichtingen","0710","liability_non_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_14","False"
"0720","Egalisatiereserve groot onderhoud","0720","liability_non_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_14","False"
"0730","Voorziening deelnemingen","0730","liability_non_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_14","False"
"0740","Latente belastingverplichting","0740","liability_non_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_14","False"
"0750","Overige voorzieningen","0750","liability_non_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_14","False"
"0760","Garantieverplichtingen","0760","liability_non_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_14","False"
"0811","Hypotheken o/g 1","0811","liability_non_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_15","False"
"0812","Hypotheken o/g 2","0812","liability_non_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_15","False"
"0813","Hypotheken o/g 3","0813","liability_non_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_15","False"
"0814","Hypotheken o/g 4","0814","liability_non_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_15","False"
"0815","Hypotheken o/g 5","0815","liability_non_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_15","False"
"0821","Leningen o/g 1","0821","liability_non_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_15","False"
"0822","Leningen o/g 2","0822","liability_non_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_15","False"
"0823","Leningen o/g 3","0823","liability_non_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_15","False"
"0824","Leningen o/g 4","0824","liability_non_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_15","False"
"0825","Leningen o/g 5","0825","liability_non_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_15","False"
"1040","Vraagposten","1040","asset_cash","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_25","False"
"recv","Debiteuren","1100","asset_receivable","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_27","True"
"recv_pos","Debiteuren (PoS)","1101","asset_receivable","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_27","True"
"1150","Dubieuze debiteuren","1150","asset_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_27","False"
"1160","Voorziening dubieuze debiteuren","1160","asset_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_27","False"
"1205","Vooruitbetaalde kosten","1205","asset_prepayments","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_27","False"
"1210","Borg","1210","asset_prepayments","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_27","False"
"1220","Te ontvangen rente","1220","asset_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_27","False"
"1230","Te ontvangen ziekengeld","1230","asset_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_27","False"
"1250","Nog te factureren (uitgaande voorraad)","1250","asset_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_27","False"
"1290","Overige kortlopende vorderingen","1290","asset_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_27","False"
"pay","Crediteuren","1300","liability_payable","l10n_nl.l10nnl_chart_template","","True"
"1350","Betalingen onderweg","1350","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_13","False"
"1405","Vooruit gefactureerde bedragen","1405","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_13","False"
"1410","Te betalen rente","1410","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_13","False"
"1420","Te betalen dividend","1420","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_13","False"
"1430","Te betalen accountantskosten","1430","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_13","False"
"1440","Te betalen telefoonkosten","1440","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_13","False"
"1450","Nog te ontvangen facturen (inkomende voorraad)","1450","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_13","False"
"1460","Te betalen VPB","1460","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"1470","Te betalen VPB voorgaande jaren","1470","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"1480","Te betalen dividendbelasting","1480","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"1490","Overige kortlopende schulden","1490","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_13","False"
"vat_payable_h","Af te dragen BTW hoog tarief","1500","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_l","Af te dragen BTW laag tarief","1501","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_0","Af te dragen BTW overige tarieven","1502","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"1503","Af te dragen BTW privé-gebruik","1503","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_v","Af te dragen BTW heffing verlegd","1504","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"1505","BTW 0% / niet bij mij belast","1505","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"1507","Installatie/televerkoop binnen de EU","1507","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"1508","Aan mij verrichte leveringen buiten EU","1508","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"1509","Aan mij verrichte leveringen binnen EU","1509","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_0","Voorbelasting overige","1510","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_h_non_eu","Af te dragen BTW hoog tarief buiten de EU","1513","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_l_non_eu","Af te dragen BTW laag tarief buiten de EU","1514","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_0_non_eu","Af te dragen BTW overige tarieven buiten de EU","1515","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_h_eu","Af te dragen BTW hoog tarief binnen de EU","1516","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_l_eu","Af te dragen BTW laag tarief binnen de EU","1517","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_0_eu","Af te dragen BTW overige tarieven binnen de EU","1518","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_h","Voorbelasting hoog","1520","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_l","Voorbelasting laag","1521","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_v","Voorbelasting verlegd","1522","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_0_eu","Voorbelasting overige binnen de EU","1523","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_h_eu","Voorbelasting hoog binnen de EU","1524","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_l_eu","Voorbelasting laag binnen de EU","1525","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_v_eu","Voorbelasting verlegd binnen de EU","1526","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_0_non_eu","Voorbelasting overige buiten EU","1527","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_h_non_eu","Voorbelasting hoog buiten EU","1528","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_l_non_eu","Voorbelasting laag buiten EU","1529","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_v_non_eu","Voorbelasting verlegd buiten EU","1530","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"1531","BTW oninbare vorderingen","1531","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"1532","Kleine ondernemersregeling","1532","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_v_d_non_eu","Voorbelasting verlegd buiten EU diensten","1535","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_h_d","Af te dragen BTW hoog tarief diensten","1540","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_l_d","Af te dragen BTW laag tarief diensten","1541","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_0_d","Af te dragen BTW overige tarieven diensten","1542","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_v_d","Af te dragen BTW heffing verlegd diensten","1544","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"1550","Afdrachten/aangifte omzetbelasting","1550","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"1560","Suppletie omzetbelasting","1560","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_0_d","Voorbelasting overige diensten","1570","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_h_d_non_eu","Af te dragen BTW hoog tarief buiten de EU diensten","1573","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_l_d_non_eu","Af te dragen BTW laag tarief buiten de EU diensten","1574","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_0_d_non_eu","Af te dragen BTW overige tarieven buiten de EU diensten","1575","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_h_d_eu","Af te dragen BTW hoog tarief binnen de EU diensten","1576","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_l_d_eu","Af te dragen BTW laag tarief binnen de EU diensten","1577","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_payable_0_d_eu","Af te dragen BTW overige tarieven binnen de EU diensten","1578","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_h_d","Voorbelasting hoog diensten","1580","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_l_d","Voorbelasting laag diensten","1581","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_v_d","Voorbelasting verlegd diensten","1582","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_0_d_eu","Voorbelasting overige binnen de EU diensten","1583","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_h_d_eu","Voorbelasting hoog binnen de EU diensten","1584","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_l_d_eu","Voorbelasting laag binnen de EU diensten","1585","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_v_d_eu","Voorbelasting verlegd binnen de EU diensten","1586","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_0_d_non_eu","Voorbelasting overige buiten EU diensten","1587","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_h_d_non_eu","Voorbelasting hoog buiten EU diensten","1588","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"vat_refund_l_d_non_eu","Voorbelasting laag buiten EU diensten","1589","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"1599","Schuld omzetbelasting (einde boekjaar)","1599","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"1610","Pensioenpremies","1610","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"1615","Afdracht pensioenpremies","1615","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"1620","Netto lonen","1620","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_28","False"
"1621","Voorschot loon","1621","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_28","False"
"1625","Te betalen netto lonen","1625","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_28","False"
"1630","Loonheffing","1630","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_32","False"
"1635","Afdracht loonheffing","1635","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_28","False"
"1640","Te betalen vakantiedagen","1640","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_13","False"
"1650","Te betalen vakantiegeld","1650","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_13","False"
"2010","Tussenrekening beginbalans","2010","liability_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_28","False"
"3001","Grondstoffen 1","3001","asset_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_4","False"
"3002","Grondstoffen 2","3002","asset_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_4","False"
"3100","Hulpstoffen 1","3100","asset_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_4","False"
"3110","Hulpstoffen 2","3110","asset_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_4","False"
"3200","Voorraad 1","3200","asset_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_4","False"
"3210","Voorraad 2","3210","asset_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_4","False"
"3300","Verpakkingsmateriaal","3300","asset_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_4","False"
"3310","Emballage","3310","asset_current","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_4","False"
"4001","Bruto lonen","4001","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_1","False"
"4002","Bonussen en provisies","4002","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_1","False"
"4003","Vakantietoeslag","4003","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_1","False"
"4004","Tantièmes","4004","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_1","False"
"4005","Werknemersbijdrage auto","4005","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_1","False"
"4006","Bijdrage zorgverzekeringswet (SVW)","4006","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_1","False"
"4010","Werkgeversdeel loonheffingen","4010","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_1","False"
"4011","Werkgeversdeel pensioenen","4011","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_20","False"
"4012","Werkgeversdeel sociale lasten","4012","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_18","False"
"4015","Voorziening vakantiedagen","4015","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_1","False"
"4016","Vergoeding woon- werkverkeer","4016","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_1","False"
"4017","Vergoeding studiekosten","4017","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_19","False"
"4018","Vergoeding overige reiskosten","4018","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_19","False"
"4019","Vergoeding overige kosten","4019","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_19","False"
"4020","Managementvergoedingen","4020","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_19","False"
"4021","Ingeleend personeel","4021","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_19","False"
"4022","Werkkostenregeling (WKR max 1.2% brutoloon)","4022","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_19","False"
"4024","Reiskosten ingeleend personeel","4024","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_19","False"
"4029","Doorbelasting directe loonkosten","4029","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_19","False"
"4030","Ziekteverzuimverzekering","4030","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_18","False"
"4040","Kantinekosten","4040","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_5","False"
"4041","Bedrijfskleding","4041","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_5","False"
"4042","Overige reiskosten","4042","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_5","False"
"4043","Congressen, seminars en symposia","4043","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_5","False"
"4044","Wervingskosten personeel","4044","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_19","False"
"4045","Studie- en opleidingskosten","4045","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_19","False"
"4090","Overige personeelskosten","4090","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_19","False"
"4110","Huur onroerend goed","4110","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4111","Groot onderhoud onroerend goed","4111","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4112","Onderhoud onroerend goed","4112","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4113","Toevoeging egalisatiereserve Groot onderhoud","4113","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4120","Belastingen onroerend goed","4120","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4130","Energiekosten","4130","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4140","Schoonmaakkosten","4140","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4150","Assurantie onroerend goed","4150","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4190","Overige huisvestingskosten","4190","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4201","Huur machines","4201","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4202","Lease machines (operational lease)","4202","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4210","Huur inventaris","4210","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4211","Lease inventaris (operational lease)","4211","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4220","Onderhoud machines","4220","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4230","Onderhoud inventaris","4230","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4240","Gereedschappen","4240","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4250","Kleine aanschaffingen","4250","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4260","Energiekosten machines / inventaris","4260","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4270","Assurantie machines / inventaris","4270","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4280","Milieukosten","4280","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4290","Overige bedrijfskosten","4290","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4305","Contributies / abonnementen","4305","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_8","False"
"4310","Kantoorbenodigdheden","4310","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_5","False"
"4320","Onderhoud kantoorinventaris","4320","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_5","False"
"4321","Huur kantoorapparatuur","4321","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_5","False"
"4322","Lease kantoorinventaris (operational lease)","4322","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_5","False"
"4330","Telefoon","4330","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_5","False"
"4340","Internet / e-mail","4340","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_5","False"
"4350","Porti","4350","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_5","False"
"4360","Automatiseringskosten","4360","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_2","False"
"4390","Overige kantoorkosten","4390","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_5","False"
"4401","Brandstof vrachtauto's","4401","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4402","Onderhoud vrachtauto's","4402","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4403","Huur vrachtauto's","4403","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4404","Lease vrachtauto's (operational lease)","4404","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4405","Assurantie vrachtauto's","4405","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4406","Belastingen vrachtauto's","4406","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4408","Overige kosten vrachtauto's","4408","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4411","Brandstof personenauto's","4411","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4412","Onderhoud personenauto's","4412","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4413","Huur personenauto's","4413","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4414","Lease personenauto's (operational lease)","4414","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4415","Assurantie personenauto's","4415","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4416","Belastingen personenauto's","4416","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4418","Privégebruik personenauto's","4418","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4419","Overige kosten personenauto's","4419","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4421","Brandstof overige vervoermiddelen","4421","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4422","Onderhoud overige vervoermiddelen","4422","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4423","Huur overige vervoermiddelen","4423","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4424","Lease overige vervoermiddelen (operational lease)","4424","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4425","Assurantie overige vervoermiddelen","4425","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4426","Belastingen overige vervoermiddelen","4426","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4428","Overige overige vervoermiddelen","4428","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4490","Overige vervoerskosten","4490","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_6","False"
"4505","Relatiegeschenken","4505","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_7","False"
"4510","Reis- en verblijfkosten","4510","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_7","False"
"4515","Representatiekosten","4515","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_7","False"
"4520","Reclamekosten","4520","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_7","False"
"4525","Advertenties","4525","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_7","False"
"4530","Beurskosten","4530","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_7","False"
"4535","Etalagekosten","4535","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_7","False"
"4540","Websitekosten","4540","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_7","False"
"4550","Afschrijving dubieuze debiteuren","4550","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_7","False"
"4560","Kredietbeperking debiteuren","4560","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_7","False"
"4570","Vrachtkosten","4570","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_26","False"
"4590","Overige verkoopkosten","4590","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_7","False"
"4601","Accountantskosten","4601","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_8","False"
"4605","Advieskosten","4605","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_8","False"
"4610","Juridische kosten","4610","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_8","False"
"4620","Assuranties","4620","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_8","False"
"4621","Administratiekosten","4621","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_8","False"
"4630","Bankkosten","4630","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_8","False"
"4690","Overige algemene kosten","4690","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_8","False"
"4705","Rente hypotheek","4705","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_31","False"
"4710","Rente lening","4710","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_31","False"
"4720","Rente rekening-courant bank","4720","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_31","False"
"4790","Overige rentelasten","4790","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_31","False"
"4795","Overige rentebaten","4795","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_30","False"
"4805","Goodwill","4805","expense_depreciation","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_22","False"
"4810","Gebouwen / verbouwingen","4810","expense_depreciation","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_21","False"
"4820","Machines","4820","expense_depreciation","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_21","False"
"4830","Inventaris","4830","expense_depreciation","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_21","False"
"4840","Vervoermiddelen","4840","expense_depreciation","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_21","False"
"4901","Boekwinst vaste activa","4901","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_12","False"
"4902","Boekverlies vaste activa","4902","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_12","False"
"4910","Kleine ondernemingsregeling Omzetbelasting","4910","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_33","False"
"4920","Koersverschillen","4920","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_31","False"
"4930","Betalingsverschillen","4930","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_31","False"
"4940","Kasverschillen","4940","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_31","False"
"4950","Overige baten","4950","income_other","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_31","False"
"4960","Overige lasten","4960","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_31","False"
"4970","Overige baten voorgaande jaren","4970","income_other","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_31","False"
"4980","Overige lasten voorgaande jaren","4980","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_31","False"
"7001","Kostprijs NL handelsgoederen 1","7001","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7002","Kostprijs binnen EU handelsgoederen 1","7002","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7003","Kostprijs buiten EU handelsgoederen 1","7003","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7004","Kostprijs Intercompany handelsgoederen 1","7004","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7005","Kostprijs NL handelsgoederen 2","7005","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7006","Kostprijs binnen EU handelsgoederen 2","7006","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7007","Kostprijs buiten EU handelsgoederen 2","7007","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7008","Kostprijs Intercompany handelsgoederen 2","7008","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7009","Kostprijs NL handelsgoederen 3","7009","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7010","Kostprijs binnen EU handelsgoederen 3","7010","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7011","Kostprijs buiten EU handelsgoederen 3","7011","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7012","Kostprijs Intercompany handelsgoederen 3","7012","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7020","Directe loonkosten","7020","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7030","Productieresultaten","7030","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7040","Kostprijs werk derden","7040","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7050","Voorziening incourante voorraad grondstof","7050","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7051","Voorziening incourante voorraad hulpstof","7051","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7052","Voorziening incourante voorraad handelsgoederen","7052","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7060","Inkoopkosten","7060","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7061","Inkoopprovisie","7061","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7062","Vrachtkosten","7062","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7063","Invoerkosten","7063","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7064","Inkoopbonussen","7064","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7065","Betalingskorting crediteuren","7065","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7070","Garantiekosten","7070","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7071","Mutatie garantieverplichting","7071","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7080","Prijsverschillen kostprijs","7080","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7090","Voorraadaanpassing","7090","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"7091","Afgekeurde voorraad","7091","expense_direct_cost","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_10","False"
"8001","Omzet NL handelsgoederen 1","8001","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8002","Omzet binnen EU handelsgoederen 1","8002","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8003","Omzet buiten EU handelsgoederen 1","8003","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8004","Omzet Intercompany handelsgoederen 1","8004","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8005","Omzet NL diensten 1","8005","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8011","Omzet NL handelsgoederen 2","8011","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8012","Omzet binnen EU handelsgoederen 2","8012","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8013","Omzet buiten EU handelsgoederen 2","8013","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8014","Omzet Intercompany handelsgoederen 2","8014","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8015","Omzet NL diensten 2","8015","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8021","Omzet NL handelsgoederen 3","8021","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8022","Omzet binnen EU handelsgoederen 3","8022","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8023","Omzet buiten EU handelsgoederen 3","8023","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8024","Omzet Intercompany handelsgoederen 3","8024","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8025","Omzet NL diensten 3","8025","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8041","Omzet NL werk derden","8041","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8042","Omzet binnen EU werk derden","8042","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8043","Omzet buiten EU werk derden","8043","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8044","Omzet Intercompany werk derden","8044","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8065","Betalingskorting debiteuren","8065","income_other","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8090","Diverse overige opbrengsten","8090","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8110","Installatie/afstandsverkopen binnen EU handelsgoederen 1","80011","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8120","Installatie/afstandsverkopen binnen EU handelsgoederen 2","80012","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8130","Installatie/afstandsverkopen binnen EU handelsgoederen 3","80013","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"8920","Koersverschillen","8920","income","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_11","False"
"9011","Resultaat deelneming 1","9011","income_other","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_16","False"
"9012","Resultaat deelneming 2","9012","income_other","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_16","False"
"9013","Resultaat deelneming 3","9013","income_other","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_16","False"
"9150","Reorganisatiekosten","9150","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_8","False"
"9200","Foutenrekening","9200","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_29","False"
"9900","Vennootschapsbelasting","9900","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_33","False"
"9950","Vennootschapsbelasting voorgaande jaren","9950","expense","l10n_nl.l10nnl_chart_template","l10n_nl.account_tag_33","False"
```

## File: data\account_account_tag.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">

        <!-- Account Tags -->
        <record id="account_tag_1" model="account.account.tag">
            <field name="name">Lonen en salarissen</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_2" model="account.account.tag">
            <field name="name">Huisvestingskosten</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_3" model="account.account.tag">
            <field name="name">Financiële vaste activa</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_4" model="account.account.tag">
            <field name="name">Voorraden</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_5" model="account.account.tag">
            <field name="name">Kantoorkosten</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_6" model="account.account.tag">
            <field name="name">Vervoerskosten</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_7" model="account.account.tag">
            <field name="name">Verkoopkosten</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_8" model="account.account.tag">
            <field name="name">Algemene Kosten</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_9" model="account.account.tag">
            <field name="name">Eigen vermogen</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_10" model="account.account.tag">
            <field name="name">Kostprijs van de omzet</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_11" model="account.account.tag">
            <field name="name">Netto omzet</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_12" model="account.account.tag">
            <field name="name">Resultaat overige activa</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_13" model="account.account.tag">
            <field name="name">Overige kortlopende schulden</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_14" model="account.account.tag">
            <field name="name">Voorzieningen</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_15" model="account.account.tag">
            <field name="name">Langlopende schulden</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_16" model="account.account.tag">
            <field name="name">Overige bedrijfsopbrengsten</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_17" model="account.account.tag">
            <field name="name">Immateriële vaste activa</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_18" model="account.account.tag">
            <field name="name">Sociale lasten</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_19" model="account.account.tag">
            <field name="name">Overige personeelskosten</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_20" model="account.account.tag">
            <field name="name">Pensioenlasten</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_21" model="account.account.tag">
            <field name="name">Afschrijving materiële vaste activa</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_22" model="account.account.tag">
            <field name="name">Afschrijving immateriële vaste activa</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_23" model="account.account.tag">
            <field name="name">Materiële vaste activa</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_24" model="account.account.tag">
            <field name="name">Effecten</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_25" model="account.account.tag">
            <field name="name">Liquide middelen</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_26" model="account.account.tag">
            <field name="name">Distributiekosten</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_27" model="account.account.tag">
            <field name="name">Vorderingen</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_28" model="account.account.tag">
            <field name="name">Tussenrekeningen</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_29" model="account.account.tag">
            <field name="name">Foutenrekening</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_30" model="account.account.tag">
            <field name="name">Rente baten</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_31" model="account.account.tag">
            <field name="name">Rente- en overige financiële lasten</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_32" model="account.account.tag">
            <field name="name">Belastingen &#38; sociale lasten</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_33" model="account.account.tag">
            <field name="name">Belastingen</field>
            <field name="applicability">accounts</field>
        </record>

    </data>
</odoo>


```

## File: data\account_chart_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<openerp>
    <data>

        <!-- Chart Template -->
        <record id="l10nnl_chart_template" model="account.chart.template">
            <field name="name">Nederlands Grootboekschema</field>
            <field name="cash_account_code_prefix">101</field>
            <field name="bank_account_code_prefix">103</field>
            <field name="transfer_account_code_prefix">1060</field>
            <field name="code_digits">6</field>
            <field name="currency_id" ref="base.EUR"/>
            <field name="country_id" ref="base.nl"/>
            <field name="use_anglo_saxon" eval="True"/>
        </record>

    </data>
</openerp>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_nl.l10nnl_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10nnl_chart_template" model="account.chart.template">
        <field name="code_digits">6</field>
        <field name="property_account_receivable_id" ref="recv"/>
        <field name="property_account_payable_id" ref="pay"/>
        <field name="property_account_expense_categ_id" ref="7001"/>
        <field name="property_account_income_categ_id" ref="8001"/>
        <field name="property_stock_account_input_categ_id" ref="0120"/>
        <field name="property_stock_account_output_categ_id" ref="0129"/>
        <field name="property_stock_valuation_account_id" ref="4830"/>
        <field name="expense_currency_exchange_account_id" ref="4920"/>
        <field name="income_currency_exchange_account_id" ref="8920"/>
        <field name="default_pos_receivable_account_id" ref="recv_pos"/>
        <field name="account_journal_early_pay_discount_loss_account_id" ref="7065"/>
        <field name="account_journal_early_pay_discount_gain_account_id" ref="8065"/>
        <field name="property_tax_payable_account_id" ref="pay"/>
        <field name="property_tax_receivable_account_id" ref="pay"/>
    </record>
</odoo>

```

## File: data\account_fiscal_position_account_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<openerp>
    <data>

        <!-- Fiscal Position Account Templates -->

        <!-- NON-EU Countries -->
        <record
            id="fiscal_position_account_non_eu_1"
            model="account.fiscal.position.account.template">
            <field name="position_id"     ref="fiscal_position_template_non_eu" />
            <field name="account_src_id"  ref="7001" />
            <field name="account_dest_id" ref="7003" />
        </record>
        <record
            id="fiscal_position_account_non_eu_2"
            model="account.fiscal.position.account.template">
            <field name="position_id"     ref="fiscal_position_template_non_eu" />
            <field name="account_src_id"  ref="7005" />
            <field name="account_dest_id" ref="7007" />
        </record>
        <record
            id="fiscal_position_account_non_eu_3"
            model="account.fiscal.position.account.template">
            <field name="position_id"     ref="fiscal_position_template_non_eu" />
            <field name="account_src_id"  ref="7009" />
            <field name="account_dest_id" ref="7011" />
        </record>
        <record
            id="fiscal_position_account_non_eu_4"
            model="account.fiscal.position.account.template">
            <field name="position_id"     ref="fiscal_position_template_non_eu" />
            <field name="account_src_id"  ref="8001" />
            <field name="account_dest_id" ref="8003" />
        </record>
        <record
            id="fiscal_position_account_non_eu_5"
            model="account.fiscal.position.account.template">
            <field name="position_id"     ref="fiscal_position_template_non_eu" />
            <field name="account_src_id"  ref="8011" />
            <field name="account_dest_id" ref="8013" />
        </record>
        <record
            id="fiscal_position_account_non_eu_6"
            model="account.fiscal.position.account.template">
            <field name="position_id"     ref="fiscal_position_template_non_eu" />
            <field name="account_src_id"  ref="8021" />
            <field name="account_dest_id" ref="8023" />
        </record>

        <!-- EU Countries -->
        <record
            id="fiscal_position_account_eu_1"
            model="account.fiscal.position.account.template">
            <field name="position_id"     ref="fiscal_position_template_eu" />
            <field name="account_src_id"  ref="7001" />
            <field name="account_dest_id" ref="7002" />
        </record>
        <record
            id="fiscal_position_account_eu_2"
            model="account.fiscal.position.account.template">
            <field name="position_id"     ref="fiscal_position_template_eu" />
            <field name="account_src_id"  ref="7005" />
            <field name="account_dest_id" ref="7006" />
        </record>
        <record
            id="fiscal_position_account_eu_3"
            model="account.fiscal.position.account.template">
            <field name="position_id"     ref="fiscal_position_template_eu" />
            <field name="account_src_id"  ref="7009" />
            <field name="account_dest_id" ref="7010" />
        </record>
        <record
            id="fiscal_position_account_eu_4"
            model="account.fiscal.position.account.template">
            <field name="position_id"     ref="fiscal_position_template_eu" />
            <field name="account_src_id"  ref="8001" />
            <field name="account_dest_id" ref="8002" />
        </record>
        <record
            id="fiscal_position_account_eu_5"
            model="account.fiscal.position.account.template">
            <field name="position_id"     ref="fiscal_position_template_eu" />
            <field name="account_src_id"  ref="8011" />
            <field name="account_dest_id" ref="8012" />
        </record>
        <record
            id="fiscal_position_account_eu_6"
            model="account.fiscal.position.account.template">
            <field name="position_id"     ref="fiscal_position_template_eu" />
            <field name="account_src_id"  ref="8021" />
            <field name="account_dest_id" ref="8022" />
        </record>

        <!-- Installatie / afstandsverkopen -->
        <record
            id="fiscal_position_account_installatie_afstand_1"
            model="account.fiscal.position.account.template">
            <field name="position_id"     ref="fiscal_position_template_eu_no_taxes_report" />
            <field name="account_src_id"  ref="8001" />
            <field name="account_dest_id" ref="8110" />
        </record>

        <record
            id="fiscal_position_account_installatie_afstand_2"
            model="account.fiscal.position.account.template">
            <field name="position_id"     ref="fiscal_position_template_eu_no_taxes_report" />
            <field name="account_src_id"  ref="8011" />
            <field name="account_dest_id" ref="8120" />
        </record>

        <record
            id="fiscal_position_account_installatie_afstand_3"
            model="account.fiscal.position.account.template">
            <field name="position_id"     ref="fiscal_position_template_eu_no_taxes_report" />
            <field name="account_src_id"  ref="8021" />
            <field name="account_dest_id" ref="8130" />
        </record>

    </data>
</openerp>

```

## File: data\account_fiscal_position_tax_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<openerp>
    <data>

        <!-- account.fiscal.position.tax.template -->

        <!-- NON-EU Countries -->
        <!-- All sales tax will become 0 -->
        <record id="position_tax_extracom_0" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu"/>
            <field name="tax_src_id" ref="btw_0"/>
            <field name="tax_dest_id" ref="btw_X1"/>
        </record>
        <record id="position_tax_extracom_6" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu"/>
            <field name="tax_src_id" ref="btw_6"/>
            <field name="tax_dest_id" ref="btw_X1"/>
        </record>
        <record id="position_tax_extracom_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu"/>
            <field name="tax_src_id" ref="btw_9"/>
            <field name="tax_dest_id" ref="btw_X1"/>
        </record>
        <record id="position_tax_extracom_21" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu"/>
            <field name="tax_src_id" ref="btw_21"/>
            <field name="tax_dest_id" ref="btw_X1"/>
        </record>
        <record id="position_tax_extracom_overig" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu"/>
            <field name="tax_src_id" ref="btw_overig"/>
            <field name="tax_dest_id" ref="btw_X1"/>
        </record>
        <record id="position_tax_extracom_d_0" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu"/>
            <field name="tax_src_id" ref="btw_0_d"/>
            <field name="tax_dest_id" ref="btw_X1"/>
        </record>
        <record id="position_tax_extracom_d_6" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu"/>
            <field name="tax_src_id" ref="btw_6_d"/>
            <field name="tax_dest_id" ref="btw_X3"/>
        </record>
        <record id="position_tax_extracom_d_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu"/>
            <field name="tax_src_id" ref="btw_9_d"/>
            <field name="tax_dest_id" ref="btw_X3"/>
        </record>
        <record id="position_tax_extracom_d_21" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu"/>
            <field name="tax_src_id" ref="btw_21_d"/>
            <field name="tax_dest_id" ref="btw_X1"/>
        </record>
        <record id="position_tax_extracom_d_overig" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu"/>
            <field name="tax_src_id" ref="btw_overig_d"/>
            <field name="tax_dest_id" ref="btw_X1"/>
        </record>
        <!-- VAT on buying from outside the EU -->
        <record id="position_tax_extracom_6" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu"/>
            <field name="tax_src_id" ref="btw_6_buy"/>
            <field name="tax_dest_id" ref="btw_E1"/>
        </record>
        <record id="position_tax_extracom_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu"/>
            <field name="tax_src_id" ref="btw_9_buy"/>
            <field name="tax_dest_id" ref="btw_E1_9"/>
        </record>
        <record id="position_tax_extracom_7" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu"/>
            <field name="tax_src_id" ref="btw_21_buy"/>
            <field name="tax_dest_id" ref="btw_E2"/>
        </record>
        <record id="position_tax_extracom_8" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu"/>
            <field name="tax_src_id" ref="btw_overig_buy"/>
            <field name="tax_dest_id" ref="btw_E_overig"/>
        </record>
        <record id="position_tax_extracom_d_6" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu"/>
            <field name="tax_src_id" ref="btw_6_buy_d"/>
            <field name="tax_dest_id" ref="btw_E1"/>
        </record>
        <record id="position_tax_extracom_d_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu"/>
            <field name="tax_src_id" ref="btw_9_buy_d"/>
            <field name="tax_dest_id" ref="btw_E1_9"/>
        </record>
        <record id="position_tax_extracom_d_7" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu"/>
            <field name="tax_src_id" ref="btw_21_buy_d"/>
            <field name="tax_dest_id" ref="btw_E2"/>
        </record>
        <record id="position_tax_extracom_d_8" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu"/>
            <field name="tax_src_id" ref="btw_overig_buy_d"/>
            <field name="tax_dest_id" ref="btw_E_overig_d"/>
        </record>

        <!-- EU Countries -->
        <record id="position_tax_intracom_1" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu"/>
            <field name="tax_src_id" ref="btw_0"/>
            <field name="tax_dest_id" ref="btw_X0_producten"/>
        </record>
        <record id="position_tax_intracom_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu"/>
            <field name="tax_src_id" ref="btw_6"/>
            <field name="tax_dest_id" ref="btw_X0_producten"/>
        </record>
        <record id="position_tax_intracom_2_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu"/>
            <field name="tax_src_id" ref="btw_9"/>
            <field name="tax_dest_id" ref="btw_X0_producten"/>
        </record>
        <record id="position_tax_intracom_3" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu"/>
            <field name="tax_src_id" ref="btw_21"/>
            <field name="tax_dest_id" ref="btw_X0_producten"/>
        </record>
        <record id="position_tax_intracom_4" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu"/>
            <field name="tax_src_id" ref="btw_overig"/>
            <field name="tax_dest_id" ref="btw_X0_producten"/>
        </record>
        <record id="position_tax_intracom_d_1" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu"/>
            <field name="tax_src_id" ref="btw_0_d"/>
            <field name="tax_dest_id" ref="btw_X0_diensten"/>
        </record>
        <record id="position_tax_intracom_d_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu"/>
            <field name="tax_src_id" ref="btw_6_d"/>
            <field name="tax_dest_id" ref="btw_X0_diensten"/>
        </record>
        <record id="position_tax_intracom_d_2_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu"/>
            <field name="tax_src_id" ref="btw_9_d"/>
            <field name="tax_dest_id" ref="btw_X0_diensten"/>
        </record>
        <record id="position_tax_intracom_d_3" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu"/>
            <field name="tax_src_id" ref="btw_21_d"/>
            <field name="tax_dest_id" ref="btw_X0_diensten"/>
        </record>
        <record id="position_tax_intracom_d_4" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu"/>
            <field name="tax_src_id" ref="btw_overig_d"/>
            <field name="tax_dest_id" ref="btw_X0_diensten"/>
        </record>
        <record id="position_tax_intracom_6" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu"/>
            <field name="tax_src_id" ref="btw_6_buy"/>
            <field name="tax_dest_id" ref="btw_I_6"/>
        </record>
        <record id="position_tax_intracom_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu"/>
            <field name="tax_src_id" ref="btw_9_buy"/>
            <field name="tax_dest_id" ref="btw_I_9"/>
        </record>
        <record id="position_tax_intracom_7" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu"/>
            <field name="tax_src_id" ref="btw_21_buy"/>
            <field name="tax_dest_id" ref="btw_I_21"/>
        </record>
        <record id="position_tax_intracom_8" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu"/>
            <field name="tax_src_id" ref="btw_overig_buy"/>
            <field name="tax_dest_id" ref="btw_I_overig"/>
        </record>
        <record id="position_tax_intracom_d_6" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu"/>
            <field name="tax_src_id" ref="btw_6_buy_d"/>
            <field name="tax_dest_id" ref="btw_I_6_d"/>
        </record>
        <record id="position_tax_intracom_d_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu"/>
            <field name="tax_src_id" ref="btw_9_buy_d"/>
            <field name="tax_dest_id" ref="btw_I_9_d"/>
        </record>
        <record id="position_tax_intracom_d_7" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu"/>
            <field name="tax_src_id" ref="btw_21_buy_d"/>
            <field name="tax_dest_id" ref="btw_I_21_d"/>
        </record>
        <record id="position_tax_intracom_d_8" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu"/>
            <field name="tax_src_id" ref="btw_overig_buy_d"/>
            <field name="tax_dest_id" ref="btw_I_overig_d"/>
        </record>

        <!-- BTW verlegd -->
        <record id="position_tax_transferred" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_transferred"/>
            <field name="tax_src_id" ref="btw_21_buy"/>
            <field name="tax_dest_id" ref="btw_ink_0"/>
        </record>
    </data>
</openerp>

```

## File: data\account_fiscal_position_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<openerp>
    <data>

        <!-- Fiscal Position Templates -->
        <record id="fiscal_position_template_national" model="account.fiscal.position.template">
            <field name="sequence">1</field>
            <field name="name">Binnenland</field>
            <field name="chart_template_id" ref="l10nnl_chart_template" />
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="fiscal_position_template_transferred" model="account.fiscal.position.template">
            <field name="name">BTW verlegd</field>
            <field name="chart_template_id" ref="l10nnl_chart_template" />
        </record>
        <record id="fiscal_position_template_eu_private" model="account.fiscal.position.template">
            <field name="sequence">2</field>
            <field name="name">EU landen privaat</field>
            <field name="chart_template_id" ref="l10nnl_chart_template" />
            <field name="auto_apply" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
        </record>
        <record id="fiscal_position_template_eu" model="account.fiscal.position.template">
            <field name="sequence">3</field>
            <field name="name">EU landen</field>
            <field name="chart_template_id" ref="l10nnl_chart_template" />
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
        </record>
        <record id="fiscal_position_template_non_eu" model="account.fiscal.position.template">
            <field name="sequence">4</field>
            <field name="name">Niet-EU landen</field>
            <field name="chart_template_id" ref="l10nnl_chart_template" />
            <field name="auto_apply" eval="True"/>
        </record>

        <!-- Fiscal Position for customers within the EU who don't have to report taxes -->
        <record id="fiscal_position_template_eu_no_taxes_report" model="account.fiscal.position.template">
            <field name="name">Installatie en Afstandsverkopen</field>
            <field name="chart_template_id" ref="l10nnl_chart_template" />
        </record>
        <record id="position_tax_eu_no_taxes_report_1" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu_no_taxes_report"/>
            <field name="tax_src_id" ref="btw_0"/>
            <field name="tax_dest_id" ref="btw_X2"/>
        </record>
        <record id="position_tax_eu_no_taxes_report_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu_no_taxes_report"/>
            <field name="tax_src_id" ref="btw_verk_0"/>
            <field name="tax_dest_id" ref="btw_X2"/>
        </record>
        <record id="position_tax_eu_no_taxes_report_3" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu_no_taxes_report"/>
            <field name="tax_src_id" ref="btw_6"/>
            <field name="tax_dest_id" ref="btw_X2"/>
        </record>
        <record id="position_tax_eu_no_taxes_report_4" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu_no_taxes_report"/>
            <field name="tax_src_id" ref="btw_9"/>
            <field name="tax_dest_id" ref="btw_X2"/>
        </record>
        <record id="position_tax_eu_no_taxes_report_5" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu_no_taxes_report"/>
            <field name="tax_src_id" ref="btw_21"/>
            <field name="tax_dest_id" ref="btw_X2"/>
        </record>
        <record id="position_tax_eu_no_taxes_report_6" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu_no_taxes_report"/>
            <field name="tax_src_id" ref="btw_overig"/>
            <field name="tax_dest_id" ref="btw_X2"/>
        </record>
        <record id="position_tax_eu_no_taxes_report_7" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu_no_taxes_report"/>
            <field name="tax_src_id" ref="btw_0_d"/>
            <field name="tax_dest_id" ref="btw_X2"/>
        </record>
        <record id="position_tax_eu_no_taxes_report_8" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu_no_taxes_report"/>
            <field name="tax_src_id" ref="btw_6_d"/>
            <field name="tax_dest_id" ref="btw_X2"/>
        </record>
        <record id="position_tax_eu_no_taxes_report_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu_no_taxes_report"/>
            <field name="tax_src_id" ref="btw_9_d"/>
            <field name="tax_dest_id" ref="btw_X2"/>
        </record>
        <record id="position_tax_eu_no_taxes_report_10" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu_no_taxes_report"/>
            <field name="tax_src_id" ref="btw_21_d"/>
            <field name="tax_dest_id" ref="btw_X2"/>
        </record>
        <record id="position_tax_eu_no_taxes_report_11" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu_no_taxes_report"/>
            <field name="tax_src_id" ref="btw_overig_d"/>
            <field name="tax_dest_id" ref="btw_X2"/>
        </record>
    </data>
</openerp>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_0" model="account.tax.group">
            <field name="name">BTW 0%</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="tax_group_6" model="account.tax.group">
            <field name="name">BTW 6%</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="tax_group_9" model="account.tax.group">
            <field name="name">BTW 9%</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="tax_group_21" model="account.tax.group">
            <field name="name">BTW 21%</field>
            <field name="country_id" ref="base.nl"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.nl"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_rub_1" model="account.report.line">
                <field name="name">Rubriek 1: Prestaties binnenland</field>
                <field name="sequence">10</field>
                <field name="aggregation_formula">1A_OMZET.balance + 1B_OMZET.balance + 1C_OMZET.balance + 1D_OMZET.balance + 1E_OMZET.balance</field>
                <field name="children_ids">
                    <record id="tax_report_rub_1a" model="account.report.line">
                        <field name="name">1a. Leveringen/diensten belast met hoog tarief (omzet)</field>
                        <field name="code">1A_OMZET</field>
                        <field name="sequence">20</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_1a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1a (omzet)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_1b" model="account.report.line">
                        <field name="name">1b. Leveringen/diensten belast met laag tarief (omzet)</field>
                        <field name="code">1B_OMZET</field>
                        <field name="sequence">30</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_1b_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1b (omzet)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_1c" model="account.report.line">
                        <field name="name">1c. Leveringen/diensten belast met overige tarieven behalve 0% (omzet)</field>
                        <field name="code">1C_OMZET</field>
                        <field name="sequence">40</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_1c_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1c (omzet)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_1d" model="account.report.line">
                        <field name="name">1d. Privégebruik (omzet)</field>
                        <field name="code">1D_OMZET</field>
                        <field name="sequence">50</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_1d_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1d (omzet)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_1e" model="account.report.line">
                        <field name="name">1e. Leveringen/diensten belast met 0% of niet bij u belast (omzet)</field>
                        <field name="code">1E_OMZET</field>
                        <field name="sequence">60</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_1e_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1e (omzet)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_rub_2" model="account.report.line">
                <field name="name">Rubriek 2: Verleggingsregelingen binnenland (omzet)</field>
                <field name="sequence">100</field>
                <field name="aggregation_formula">2A_OMZET.balance</field>
                <field name="children_ids">
                    <record id="tax_report_rub_2a" model="account.report.line">
                        <field name="name">2a. Leveringen/diensten waarbij de heffing van omzetbelasting naar u is verlegd (omzet)</field>
                        <field name="code">2A_OMZET</field>
                        <field name="sequence">110</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_2a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2a (omzet)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_rub_3" model="account.report.line">
                <field name="name">Rubriek 3: Prestaties naar of in het buitenland (omzet)</field>
                <field name="sequence">200</field>
                <field name="aggregation_formula">3A_OMZET.balance + 3B_OMZET.balance + 3C_OMZET.balance</field>
                <field name="children_ids">
                    <record id="tax_report_rub_3a" model="account.report.line">
                        <field name="name">3a. Leveringen naar landen buiten de EU (uitvoer) (omzet)</field>
                        <field name="code">3A_OMZET</field>
                        <field name="sequence">210</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_3a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3a (omzet)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_3b" model="account.report.line">
                        <field name="name">3b. Leveringen naar/diensten in landen binnen de EU en ABC Sales (omzet)</field>
                        <field name="code">3B_OMZET</field>
                        <field name="sequence">220</field>
                        <field name="aggregation_formula">3BG_OMZET.balance + 3BT_OMZET.balance + 3BS_OMZET.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_rub_3bg" model="account.report.line">
                                <field name="name">3bg. Leveringen producten naar landen binnen de EU (omzet)</field>
                                <field name="code">3BG_OMZET</field>
                                <field name="sequence">223</field>
                                <field name="expression_ids">
                                    <record id="tax_report_rub_3bg_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3bg (omzet)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_rub_3bt" model="account.report.line">
                                <field name="name">3bt. ABC Levering binnen de EU (Omzet)</field>
                                <field name="code">3BT_OMZET</field>
                                <field name="sequence">226</field>
                                <field name="expression_ids">
                                    <record id="tax_report_rub_3bt_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3bt (omzet)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_rub_3bs" model="account.report.line">
                                <field name="name">3bs. Diensten in landen binnen de EU (omzet)</field>
                                <field name="code">3BS_OMZET</field>
                                <field name="sequence">223</field>
                                <field name="expression_ids">
                                    <record id="tax_report_rub_3bs_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3bs (omzet)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_3c" model="account.report.line">
                        <field name="name">3c. Installatie/afstandsverkopen binnen de EU (omzet)</field>
                        <field name="code">3C_OMZET</field>
                        <field name="sequence">230</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_3c_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3c (omzet)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_rub_4" model="account.report.line">
                <field name="name">Rubriek 4: Prestaties vanuit het buitenland aan u verricht (omzet)</field>
                <field name="sequence">300</field>
                <field name="aggregation_formula">4A_OMZET.balance + 4B_OMZET.balance</field>
                <field name="children_ids">
                    <record id="tax_report_rub_4a" model="account.report.line">
                        <field name="name">4a. Leveringen/diensten uit landen buiten de EU (invoer) (omzet)</field>
                        <field name="code">4A_OMZET</field>
                        <field name="sequence">310</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_4a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4a (omzet)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_4b" model="account.report.line">
                        <field name="name">4b. Leveringen/diensten uit landen binnen de EU (omzet)</field>
                        <field name="code">4B_OMZET</field>
                        <field name="sequence">320</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_4b_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4b (omzet)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_rub_btw_1" model="account.report.line">
                <field name="name">Rubriek 1: Prestaties binnenland (BTW)</field>
                <field name="code">NLTAX_B1</field>
                <field name="sequence">400</field>
                <field name="aggregation_formula">1A_BTW.balance + 1B_BTW.balance + 1C_BTW.balance + 1D_BTW.balance + 1E_BTW.balance</field>
                <field name="children_ids">
                    <record id="tax_report_rub_btw_1a" model="account.report.line">
                        <field name="name">1a. Leveringen/diensten belast met 21% (BTW)</field>
                        <field name="code">1A_BTW</field>
                        <field name="sequence">410</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_1a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1a (BTW)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_btw_1b" model="account.report.line">
                        <field name="name">1b. Leveringen/diensten belast met laag tarief (BTW)</field>
                        <field name="code">1B_BTW</field>
                        <field name="sequence">420</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_1b_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1b (BTW)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_btw_1c" model="account.report.line">
                        <field name="name">1c. Leveringen/diensten belast met overige tarieven behalve 0% (BTW)</field>
                        <field name="code">1C_BTW</field>
                        <field name="sequence">430</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_1c_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1c (BTW)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_btw_1d" model="account.report.line">
                        <field name="name">1d. Privégebruik (BTW)</field>
                        <field name="code">1D_BTW</field>
                        <field name="sequence">440</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_1d_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1d (BTW)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_btw_1e" model="account.report.line">
                        <field name="name">1e. Leveringen/diensten belast met 0% of niet bij u belast (BTW)</field>
                        <field name="code">1E_BTW</field>
                        <field name="sequence">450</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_1e_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1e (BTW)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_rub_btw_2" model="account.report.line">
                <field name="name">Rubriek 2: Verleggingsregelingen binnenland (BTW)</field>
                <field name="sequence">500</field>
                <field name="aggregation_formula">NLTAX_B2.balance</field>
                <field name="children_ids">
                    <record id="tax_report_rub_btw_2a" model="account.report.line">
                        <field name="name">2a. Leveringen/diensten waarbij de heffing van Heffing van omzetbelasting naar u is verlegd (BTW)</field>
                        <field name="code">NLTAX_B2</field>
                        <field name="sequence">510</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_2a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2a (BTW)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_rub_btw_4" model="account.report.line">
                <field name="name">Rubriek 4: Prestaties vanuit het buitenland aan u verricht (BTW)</field>
                <field name="sequence">600</field>
                <field name="aggregation_formula">NLTAX_B4a.balance + NLTAX_B4b.balance</field>
                <field name="children_ids">
                    <record id="tax_report_rub_btw_4a" model="account.report.line">
                        <field name="name">4a. Leveringen/diensten uit landen buiten de EU (BTW)</field>
                        <field name="code">NLTAX_B4a</field>
                        <field name="sequence">610</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_4a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4a (BTW)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_btw_4b" model="account.report.line">
                        <field name="name">4b. Leveringen/diensten uit landen binnen de EU (BTW)</field>
                        <field name="code">NLTAX_B4b</field>
                        <field name="sequence">620</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_4b_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4b (BTW)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_rub_btw_5" model="account.report.line">
                <field name="name">Rubriek 5: Voorbelasting, kleineondernemersregeling en totaal (BTW)</field>
                <field name="sequence">700</field>
                <field name="aggregation_formula"></field>
                <field name="children_ids">
                    <record id="tax_report_rub_btw_5a" model="account.report.line">
                        <field name="name">5a. Verschuldigde omzetbelasting (rubrieken 1a t/m 4b) (BTW)</field>
                        <field name="sequence">710</field>
                        <field name="aggregation_formula">NLTAX_B1.balance + NLTAX_B2.balance + NLTAX_B4a.balance + NLTAX_B4b.balance</field>
                    </record>
                    <record id="tax_report_rub_btw_5b" model="account.report.line">
                        <field name="name">5b. Voorbelasting (BTW)</field>
                        <field name="code">NLTAX_B5b</field>
                        <field name="sequence">720</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_5b_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5b (BTW)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_btw_5c" model="account.report.line">
                        <field name="name">5c. Subtotaal (rubriek 5a min 5b) (BTW)</field>
                        <field name="sequence">730</field>
                        <field name="aggregation_formula">NLTAX_B1.balance + NLTAX_B2.balance + NLTAX_B4a.balance + NLTAX_B4b.balance - NLTAX_B5b.balance</field>
                    </record>
                    <record id="tax_report_rub_btw_5d" model="account.report.line">
                        <field name="name">5d. Vermindering volgens de kleineondernemersregeling (BTW)</field>
                        <field name="code">NLTAX_B5d</field>
                        <field name="sequence">740</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_5d_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5d. (BTW)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_btw_5e" model="account.report.line">
                        <field name="name">5e. Schatting vorige aangifte(n) (BTW)</field>
                        <field name="code">NLTAX_B5e</field>
                        <field name="sequence">750</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_5e_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5e. (BTW)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_btw_5f" model="account.report.line">
                        <field name="name">5f. Schatting deze aangifte (BTW)</field>
                        <field name="code">NLTAX_B5f</field>
                        <field name="sequence">760</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_5f_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5f. (BTW)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_btw_5g" model="account.report.line">
                        <field name="name">5g. Totaal te betalen/terug te vragen (BTW)</field>
                        <field name="code">NLTAX_B5g</field>
                        <field name="sequence">770</field>
                        <field name="aggregation_formula">NLTAX_B1.balance + NLTAX_B2.balance + NLTAX_B4a.balance + NLTAX_B4b.balance - NLTAX_B5b.balance - NLTAX_B5d.balance - NLTAX_B5e.balance - NLTAX_B5f.balance</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\account_tax_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Binnen Nederland -->
    <!-- Verkoop BTW -->
    <record id="btw_0" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Verkopen/omzet onbelast (nul-tarief)</field>
        <field name="description">0% BTW</field>
        <field eval="0" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_1e_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_1e_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
    </record>
    <record id="btw_6" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Verkopen/omzet laag 6%</field>
        <field name="description">6% BTW</field>
        <field eval="6" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_6"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_1b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_1b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_1b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_1b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_9" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Verkopen/omzet laag 9%</field>
        <field name="description">9% BTW</field>
        <field eval="9" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_1b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_1b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_1b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_1b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_21" model="account.tax.template">
        <field name="sequence">5</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Verkopen/omzet hoog</field>
        <field name="description">21% BTW</field>
        <field eval="21" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_1a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_1a_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_1a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_1a_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_overig" model="account.tax.template">
        <field name="sequence">15</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Verkopen/omzet overig</field>
        <field name="description">variabel BTW</field>
        <field eval="21" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_1a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_1a_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_1c_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_1c_tag')],
                }),
            ]"/>
    </record>
    <!-- Verkoop BTW Diensten -->
    <record id="btw_0_d" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Verkopen/omzet onbelast (nul-tarief) diensten</field>
        <field name="description">0% BTW diensten</field>
        <field eval="0" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_1e_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_1e_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
    </record>
    <record id="btw_6_d" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Verkopen/omzet laag diensten 6%</field>
        <field name="description">6% BTW diensten</field>
        <field eval="6" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_6"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_1b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_d'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_1b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_1b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_d'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_1b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_9_d" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Verkopen/omzet laag diensten 9%</field>
        <field name="description">9% BTW diensten</field>
        <field eval="9" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_1b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_d'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_1b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_1b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_d'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_1b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_21_d" model="account.tax.template">
        <field name="sequence">6</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Verkopen/omzet hoog diensten</field>
        <field name="description">21% BTW diensten</field>
        <field eval="21" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_1a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h_d'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_1a_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_1a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h_d'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_1a_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_overig_d" model="account.tax.template">
        <field name="sequence">15</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Verkopen/omzet overig diensten</field>
        <field name="description">variabel BTW diensten</field>
        <field eval="21" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_1a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h_d'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_1a_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_1c_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h_d'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_1c_tag')],
                }),
            ]"/>
    </record>
    <!--Inkoop BTW -->
    <record id="btw_6_buy" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">BTW te vorderen laag (inkopen) 6%</field>
        <field name="description">6% BTW</field>
        <field eval="6" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_6"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_9_buy" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">BTW te vorderen laag (inkopen) 9%</field>
        <field name="description">9% BTW</field>
        <field eval="9" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_6_buy_incl" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">BTW te vorderen laag (inkopen incl. BTW) 6%</field>
        <field name="description">6% BTW Incl.</field>
        <field eval="6" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="price_include">True</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_6"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_9_buy_incl" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">BTW te vorderen laag (inkopen incl. BTW) 9%</field>
        <field name="description">9% BTW Incl.</field>
        <field eval="9" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="price_include">True</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_21_buy" model="account.tax.template">
        <field name="sequence">5</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">BTW te vorderen hoog (inkopen)</field>
        <field name="description">21% BTW</field>
        <field eval="21" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_21_buy_incl" model="account.tax.template">
        <field name="sequence">7</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">BTW te vorderen hoog (inkopen incl. BTW)</field>
        <field name="description">21% BTW Incl.</field>
        <field eval="21" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="price_include">True</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_overig_buy" model="account.tax.template">
        <field name="sequence">15</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">BTW te vorderen overig (inkopen)</field>
        <field name="description">variabel BTW</field>
        <field eval="21" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <!--Inkoop BTW diensten -->
    <record id="btw_6_buy_d" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">BTW te vorderen laag (inkopen) diensten 6%</field>
        <field name="description">6% BTW diensten</field>
        <field eval="6" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_6"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_d'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_d'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_9_buy_d" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">BTW te vorderen laag (inkopen) diensten 9%</field>
        <field name="description">9% BTW diensten</field>
        <field eval="9" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_d'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_d'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_21_buy_d" model="account.tax.template">
        <field name="sequence">6</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">BTW te vorderen hoog (inkopen) diensten</field>
        <field name="description">21% BTW diensten</field>
        <field eval="21" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h_d'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h_d'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_overig_buy_d" model="account.tax.template">
        <field name="sequence">15</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">BTW te vorderen overig (inkopen) diensten</field>
        <field name="description">variabel BTW diensten</field>
        <field eval="21" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h_d'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h_d'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <!--BTW verlegd-->
    <record id="btw_verk_0" model="account.tax.template">
        <field name="sequence">15</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">BTW af te dragen verlegd (verkopen)</field>
        <field name="description">0% BTW verlegd</field>
        <field eval="0" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_1e_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_1e_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
    </record>
    <record id="btw_ink_0" model="account.tax.template">
        <field name="sequence">15</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">BTW af te dragen verlegd (inkopen)</field>
        <field name="description">21% BTW verlegd</field>
        <field eval="21" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_2a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_v'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_2a_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_v'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_2a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_v'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_2a_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_v'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <!-- Binnen de EU -->
    <!-- BTW inkoop -->
    <record id="btw_I_6" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Inkopen import binnen EU laag 6%</field>
        <field name="description">6% BTW import binnen EU</field>
        <field eval="6" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_6"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_4b_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_4b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_4b_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_4b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_I_9" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Inkopen import binnen EU laag 9%</field>
        <field name="description">9% BTW import binnen EU</field>
        <field eval="9" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_4b_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_4b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_4b_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_4b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_I_21" model="account.tax.template">
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="sequence">20</field>
        <field name="name">Inkopen import binnen EU hoog</field>
        <field name="description">21% BTW import binnen EU</field>
        <field eval="21" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_4b_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_4b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_4b_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_4b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_I_overig" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Inkopen import binnen EU overig</field>
        <field name="description">0% BTW import binnen EU</field>
        <field eval="0" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_4b_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_4b_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
    </record>
    <!-- BTW verkoop -->
    <record id="btw_X0_producten" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Verkopen export binnen EU (producten)</field>
        <field name="description">BTW export binnen EU (producten)</field>
        <field eval="0" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_3bg_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_3bg_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
    </record>
    <record id="btw_X0_ABC_levering" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">ABC-levering binnen EU</field>
        <field name="description">BTW ABC-levering binnen EU</field>
        <field eval="0" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_3bt_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_3bt_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
    </record>
    <record id="btw_X0_diensten" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Verkopen export binnen EU (diensten)</field>
        <field name="description">BTW export binnen EU (diensten)</field>
        <field eval="0" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_3bs_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_3bs_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
    </record>
    <record id="btw_X2" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Installatie/afstandsverkopen binnen EU</field>
        <field name="description">Inst./afst.verkopen binnen EU</field>
        <field eval="0" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_3c_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_3c_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
    </record>
    <!-- BTW inkoop diensten -->
    <record id="btw_I_6_d" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Inkopen import binnen EU laag diensten 6%</field>
        <field name="description">6% BTW import binnen EU diensten</field>
        <field eval="6" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_6"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_4b_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_d_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_4b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_d_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_4b_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_d_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_4b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_d_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_I_9_d" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Inkopen import binnen EU laag diensten 9%</field>
        <field name="description">9% BTW import binnen EU diensten</field>
        <field eval="9" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_4b_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_d_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_4b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_d_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_4b_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_d_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_4b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_d_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_I_21_d" model="account.tax.template">
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="sequence">20</field>
        <field name="name">Inkopen import binnen EU hoog diensten</field>
        <field name="description">21% BTW import binnen EU diensten</field>
        <field eval="21" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_4b_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h_d_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_4b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h_d_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_4b_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h_d_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_4b_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h_d_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_I_overig_d" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Inkopen import binnen EU overig diensten</field>
        <field name="description">0% BTW import binnen EU diensten</field>
        <field eval="0" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_4b_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_4b_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
    </record>
    <!-- Buiten de EU -->
    <!-- BTW inkoop -->
    <record id="btw_E1" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Inkopen import buiten EU laag 6%</field>
        <field name="description">BTW import buiten EU laag inkopen</field>
        <field eval="6" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_6"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_4a_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_non_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_4a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_non_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_4a_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_non_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_4a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_non_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_E1_9" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Inkopen import buiten EU laag 9%</field>
        <field name="description">BTW import buiten EU laag inkopen</field>
        <field eval="9" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_4a_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_non_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_4a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_non_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_4a_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_non_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_4a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_non_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_E2" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Inkopen import buiten EU hoog</field>
        <field name="description">BTW import buiten EU hoog inkopen</field>
        <field eval="21" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_4a_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h_non_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_4a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h_non_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_4a_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h_non_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_4a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h_non_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_E_overig" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Inkopen import buiten EU overig</field>
        <field name="description">BTW import buiten EU overig inkopen</field>
        <field eval="21" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_4a_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h_non_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_4a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h_non_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_4a_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h_non_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_4a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h_non_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <!-- BTW Verkoop -->
    <record id="btw_X1" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Verkopen export buiten EU</field>
        <field name="description">BTW export buiten EU</field>
        <field eval="0" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_3a_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_3a_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
    </record>
    <record id="btw_X3" model="account.tax.template">
        <field name="sequence">21</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Installatie/afstandsverkopen buiten EU</field>
        <field name="description">Inst./afst.verkopen buiten EU</field>
        <field eval="0" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_3a_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_3a_tag')],
                }),
                (0,0, {'repartition_type': 'tax'}),
            ]"/>
    </record>
    <!-- BTW inkoop diensten -->
    <record id="btw_E1_d" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Inkopen import buiten EU laag diensten 6%</field>
        <field name="description">BTW import buiten EU laag inkopen diensten</field>
        <field eval="6" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_6"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_4a_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_d_non_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_4a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_d_non_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_4a_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_d_non_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_4a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_d_non_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_E1_d_9" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Inkopen import buiten EU laag diensten 9%</field>
        <field name="description">BTW import buiten EU laag inkopen diensten</field>
        <field eval="9" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_4a_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_d_non_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_4a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_d_non_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_4a_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_l_d_non_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_4a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_l_d_non_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_E2_d" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Inkopen import buiten EU hoog diensten</field>
        <field name="description">BTW import buiten EU hoog inkopen diensten</field>
        <field eval="21" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_4a_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h_d_non_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_4a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h_d_non_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_4a_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h_d_non_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_4a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h_d_non_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
    <record id="btw_E_overig_d" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="chart_template_id" ref="l10nnl_chart_template"/>
        <field name="name">Inkopen import buiten EU overig diensten</field>
        <field name="description">BTW import buiten EU overig inkopen diensten</field>
        <field eval="21" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_rub_4a_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h_d_non_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_4a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h_d_non_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_rub_4a_tag')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('vat_payable_h_d_non_eu'),
                    'plus_report_expression_ids': [ref('tax_report_rub_btw_4a_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('vat_refund_h_d_non_eu'),
                    'minus_report_expression_ids': [ref('tax_report_rub_btw_5b_tag')],
                }),
            ]"/>
    </record>
</odoo>

```

## File: migrations\3.1\post-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.addons.account.models.chart_template import update_taxes_from_templates


def migrate(cr, version):
    update_taxes_from_templates(cr, 'l10n_nl.l10nnl_chart_template')

```

## File: migrations\3.3\post-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID
from psycopg2 import sql


def _get_tax_ids_for_xml_id(cr, xml_id):
    cr.execute(sql.SQL(
        """
        SELECT res_id
        FROM ir_model_data
        WHERE model = 'account.tax'
        AND name LIKE '%' || {xml_id}
        """
    ).format(xml_id=sql.Literal(xml_id)))

    return [line['res_id'] for line in cr.dictfetchall()]


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})

    goods_taxes = env['account.tax'].browse(_get_tax_ids_for_xml_id(cr, 'btw_X0_producten'))
    services_taxes = env['account.tax'].browse(_get_tax_ids_for_xml_id(cr, 'btw_X0_diensten'))

    old_3bl_tax_tags = env['account.account.tag']._get_tax_tags('3bl (omzet)', 'nl')
    old_3b_tax_tags = env['account.account.tag']._get_tax_tags('3b (omzet)', 'nl')
    if not old_3bl_tax_tags and not old_3b_tax_tags:
        return

    goods_tax_tags = env['account.account.tag']._get_tax_tags('3bg (omzet)', 'nl')
    services_tax_tags = env['account.account.tag']._get_tax_tags('3bs (omzet)', 'nl')

    old_3bl_tax_tags_plus = old_3bl_tax_tags.filtered(lambda tag: not tag.tax_negate).id
    old_3b_tax_tags_plus = old_3b_tax_tags.filtered(lambda tag: not tag.tax_negate).id
    old_plus_tax_tag_ids = []
    if old_3bl_tax_tags_plus:
        old_plus_tax_tag_ids.append(old_3bl_tax_tags_plus)
    if old_3b_tax_tags_plus:
        old_plus_tax_tag_ids.append(old_3b_tax_tags_plus)

    old_3bl_tax_tags_minus = old_3bl_tax_tags.filtered(lambda tag: tag.tax_negate).id
    old_3b_tax_tags_minus = old_3b_tax_tags.filtered(lambda tag: tag.tax_negate).id
    old_minus_tax_tag_ids = []
    if old_3bl_tax_tags_minus:
        old_minus_tax_tag_ids.append(old_3bl_tax_tags_minus)
    if old_3b_tax_tags_minus:
        old_minus_tax_tag_ids.append(old_3b_tax_tags_minus)

    goods_plus_tax_tag_id = goods_tax_tags.filtered(lambda tag: not tag.tax_negate).id
    goods_minus_tax_tag_id = goods_tax_tags.filtered(lambda tag: tag.tax_negate).id
    services_plus_tax_tag_id = services_tax_tags.filtered(lambda tag: not tag.tax_negate).id
    services_minus_tax_tag_id = services_tax_tags.filtered(lambda tag: tag.tax_negate).id

    insert_query_params = [
        (goods_plus_tax_tag_id, goods_taxes.ids, old_plus_tax_tag_ids, goods_taxes.invoice_repartition_line_ids.ids),
        (services_plus_tax_tag_id, services_taxes.ids, old_plus_tax_tag_ids, services_taxes.invoice_repartition_line_ids.ids),
        (goods_minus_tax_tag_id, goods_taxes.ids, old_minus_tax_tag_ids, goods_taxes.refund_repartition_line_ids.ids),
        (services_minus_tax_tag_id, services_taxes.ids, old_minus_tax_tag_ids, services_taxes.refund_repartition_line_ids.ids),
    ]
    insert_query_parts = []

    for new_tax_tag_id, tax_ids, old_tax_tag_ids, repartition_line_ids in insert_query_params:
        insert_query_parts.append(
            sql.SQL(cr.mogrify("""
                SELECT tag_aml_rel.account_move_line_id, %s
                FROM account_account_tag_account_move_line_rel tag_aml_rel
                JOIN account_move_line_account_tax_rel aml_at_rel ON aml_at_rel.account_move_line_id = tag_aml_rel.account_move_line_id
                WHERE aml_at_rel.account_tax_id = ANY(%s)
                AND tag_aml_rel.account_account_tag_id = ANY(%s)
                """,
                [new_tax_tag_id, tax_ids, old_tax_tag_ids]
            ).decode())
        )

        if len(old_tax_tag_ids) > 1:
            cr.execute(
                """
                    DELETE FROM account_account_tag_account_tax_repartition_line_rel tag_aml_rel
                    WHERE tag_aml_rel.account_account_tag_id = %s
                    AND (
                        SELECT COUNT(*)
                        FROM account_account_tag_account_tax_repartition_line_rel sub_tag_aml_rel
                        WHERE sub_tag_aml_rel.account_tax_repartition_line_id = tag_aml_rel.account_tax_repartition_line_id
                        AND sub_tag_aml_rel.account_account_tag_id = %s
                    ) >= 1
                """,
                [old_tax_tag_ids[0], old_tax_tag_ids[1]],
            )

        cr.execute(
            """
                UPDATE account_account_tag_account_tax_repartition_line_rel
                SET account_account_tag_id = %s
                WHERE account_tax_repartition_line_id = ANY(%s)
                AND account_account_tag_id = ANY(%s)
            """,
            [new_tax_tag_id, repartition_line_ids, old_tax_tag_ids]
        )

    cr.execute(
        sql.SQL("""
            INSERT INTO account_account_tag_account_move_line_rel (account_move_line_id, account_account_tag_id)
                {select_statement}
            ON CONFLICT DO NOTHING
        """).format(
            select_statement=sql.SQL(" UNION ").join(insert_query_parts),
        ).as_string(cr)
    )

```

## File: migrations\9.0.2.0\pre-set_tags_and_taxes_updatable.py

```python
# -*- coding: utf-8 -*-

import odoo

def migrate(cr, version):
    registry = odoo.registry(cr.dbname)
    from odoo.addons.account.models.chart_template import migrate_set_tags_and_taxes_updatable
    migrate_set_tags_and_taxes_updatable(cr, registry, 'l10n_nl')

```

## File: models\account_chart_template.py

```python
# -*- coding: utf-8 -*-

from odoo import api, Command, models


class AccountChartTemplate(models.Model):
    _inherit = 'account.chart.template'

    def _load(self, company):
        # Add tag to 999999 account
        res = super(AccountChartTemplate, self)._load(company)
        if company.account_fiscal_country_id.code == 'NL':
            account = self.env['account.account'].search([('code', '=', '999999'), ('company_id', '=', self.env.company.id)])
            if account:
                account.tag_ids = [(4, self.env.ref('l10n_nl.account_tag_12').id)]
        return res

    @api.model
    def _create_liquidity_journal_suspense_account(self, company, code_digits):
        account = super()._create_liquidity_journal_suspense_account(company, code_digits)
        if company.account_fiscal_country_id.code == 'NL':
            account.tag_ids = [Command.link(self.env.ref('l10n_nl.account_tag_25').id)]
        return account

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models, _


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    @api.model
    def _prepare_liquidity_account_vals(self, company, code, vals):
        # OVERRIDE
        account_vals = super()._prepare_liquidity_account_vals(company, code, vals)

        if company.account_fiscal_country_id.code == 'NL':
            # Ensure the newly liquidity accounts have the right account tag in order to be part
            # of the Dutch financial reports.
            account_vals.setdefault('tag_ids', [])
            account_vals['tag_ids'].append((4, self.env.ref('l10n_nl.account_tag_25').id))

        return account_vals

```

## File: models\res_company.py

```python
# coding: utf-8
from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_nl_kvk = fields.Char(related='partner_id.l10n_nl_kvk', readonly=False)
    l10n_nl_oin = fields.Char(related='partner_id.l10n_nl_oin', readonly=False)

```

## File: models\res_partner.py

```python
# coding: utf-8
from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_nl_kvk = fields.Char(string='KVK-nummer')
    l10n_nl_oin = fields.Char(string='Organisatie Indentificatie Nummer')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_journal
from . import account_chart_template
from . import res_partner
from . import res_company

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.8" y="7.25" width="50.4" height="32.5" maskUnits="userSpaceOnUse">
      <rect x="6.29" y="7.53" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="255" height="170" transform="translate(4.8 7.25) scale(0.2 0.19)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAP8AAAClCAYAAACTHStbAAAACXBIWXMAADf6AAA3+gH8300lAAACIUlEQVR4Xu3WsY0CQQAEwT10SaBPhwzIAIlAcPA/GzxCW0JYf7vKHrelOb7jbw4g57IaAHsSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1Hih6jz+rivNsCGjjnnXI2A/bj9ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUefz/VltgA0d4/Y/VyNgP24/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFD1DnGeK1GwH5++Y8L6avEue0AAAAASUVORK5CYII="/>
    </g>
  </g>
</svg>

```

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_company_form_inherit_nl" model="ir.ui.view">
            <field name="name">res.company.form.inherit.l10n.nl</field>
            <field name="model">res.company</field>
            <field name="inherit_id" ref="base.view_company_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='vat']" position="after">
                    <field name="l10n_nl_kvk" attrs="{'invisible': [('country_code', '!=', 'NL')]}"/>
                    <field name="l10n_nl_oin" attrs="{'invisible': [('country_code', '!=', 'NL')]}"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\res_config_settings_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.ui.view" id="res_config_settings_view_form">
        <field name="name">res.config.settings.view.form</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@data-key='account']/div" position="after">
                <div id="dutch_localization_section" invisible="1">
                    <h2 attrs="{'invisible':[('country_code', '!=', 'NL')]}">Dutch Localization</h2>
                    <div class="row mt16 o_settings_container" id="dutch_localization" attrs="{'invisible':[('country_code', '!=', 'NL')]}">
                    </div>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_partner_form_inherit_l10n_nl" model="ir.ui.view">
        <field name="name">res.partner.form.inherit.l10n_nl</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="after">
                <field name="l10n_nl_kvk" attrs="{'invisible': ['|', ('country_code', '!=', 'NL'), ('is_company', '=', False)]}"/>
                <field name="l10n_nl_oin" attrs="{'invisible': ['|', ('country_code', '!=', 'NL'), ('is_company', '=', False)]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

