# Odoo Module: l10n_ua

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
# Copyright (C) 2019 Bohdan Lisnenko <bohdan.lisnenko@erp.co.ua>, ERP Ukraine
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Ukraine - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['ua'],
    'author': 'ERP Ukraine (https://erp.co.ua)',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations.html',
    'version': '1.4',
    'description': """
Ukraine - Chart of accounts.
============================
    """,
    'category': 'Accounting/Localizations/Account Charts',
    'depends': [
        'account',
    ],
    'data': [
        'data/account_account_tag_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_account_tag_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="acc_tag_vat" model="account.account.tag">
            <field name="name">ПДВ</field>
            <field name="applicability">accounts</field>
        </record>
    </data>
</odoo>

```

## File: data\template\account.account-ua_ias.csv

```csv
"id","code","name","account_type","reconcile","tag_ids"
"ua_ias_1000","1000","Назви брендів","asset_fixed","False",""
"ua_ias_1001","1001","Заголовки та назви видань","asset_fixed","False",""
"ua_ias_1002","1002","Комп’ютерне програмне забезпечення","asset_fixed","False",""
"ua_ias_1003","1003","Ліцензії та привілеї","asset_fixed","False",""
"ua_ias_1004","1004","Авторські права, патенти, та інші права","asset_fixed","False",""
"ua_ias_1005","1005","Нематеріальні активи на етапі розробки","asset_fixed","False",""
"ua_ias_1009","1009","Амортизація нематеріальних активів","expense_depreciation","False",""
"ua_ias_1010","1010","Земля","asset_fixed","False",""
"ua_ias_1011","1011","Будівлі","asset_fixed","False",""
"ua_ias_1012","1012","Машини та обладнання","asset_fixed","False",""
"ua_ias_1013","1013","Автомобілі","asset_fixed","False",""
"ua_ias_1014","1014","Меблі та приладдя","asset_fixed","False",""
"ua_ias_1015","1015","Офісне обладнання","asset_fixed","False",""
"ua_ias_1019","1019","Амортизація основних засобів","expense_depreciation","False",""
"ua_ias_1020","1020","Гудвіл","asset_fixed","False",""
"ua_ias_1030","1030","Біологічні активи","asset_fixed","False",""
"ua_ias_1040","1040","Інвестиції","asset_fixed","False",""
"ua_ias_1100","1100","Запаси (Склад)","asset_current","False",""
"ua_ias_1101","1101","Незавершене виробництво","asset_current","False",""
"ua_ias_1102","1102","Готова продукція","asset_current","False",""
"ua_ias_1120","1120","Дебіторська заборгованість за товари та послуги","asset_receivable","True",""
"ua_ias_1121","1121","Відвантажено зі складу без рахунку-фактури","asset_current","True",""
"ua_ias_1122","1122","Дебіторська заборгованість за товари та послуги (PoS)","asset_receivable","True",""
"ua_ias_1130","1130","Аванси партнерам","liability_payable","True",""
"ua_ias_1131","1131","Інші оборотні активи","liability_payable","True",""
"ua_ias_1140","1140","Податковий кредит по ПДВ","asset_current","False",""
"ua_ias_1141","1141","Аванс по податку на прибуток","liability_payable","True",""
"ua_ias_1142","1142","Кошти на СЕА ПДВ","liability_payable","True",""
"ua_ias_1143","1143","Переплата по лицьовому рахунку з ПДВ","asset_current","False",""
"ua_ias_1144","1144","Аванс по ЄСВ","liability_payable","True",""
"ua_ias_1150","1150","Короткострокові фінансові інвестиції","asset_current","False",""
"ua_ias_1160","1160","Довгострокові інвестиції","asset_non_current","False",""
"ua_ias_1170","1170","Довгострокова дебіторська заборгованість","asset_non_current","False",""
"ua_ias_1180","1180","Відстрочені податкові активи","asset_non_current","False",""
"ua_ias_1200","1200","Кредиторська заборгованість за товари та послуги","liability_payable","True",""
"ua_ias_1201","1201","Отримано на склад без рахунку-фактури","liability_current","True",""
"ua_ias_1202","1202","Податок на прибуток","liability_payable","True",""
"ua_ias_1203","1203","ПДВ","liability_payable","True","l10n_ua.acc_tag_vat"
"ua_ias_1204","1204","Податкові зобов’язання по ПДВ","liability_current","False",""
"ua_ias_1205","1205","ПДФО","liability_payable","True",""
"ua_ias_12051","12051","Військовий збір","liability_payable","True",""
"ua_ias_1206","1206","ЄСВ","liability_payable","True",""
"ua_ias_1207","1207","Інші податки","liability_payable","True",""
"ua_ias_1208","1208","Заробітна плата та винагороди","liability_payable","True",""
"ua_ias_1210","1210","Аванси одержані від покупців та замовників","asset_receivable","True",""
"ua_ias_1220","1220","Інші короткострокові зобов’язання","liability_current","False",""
"ua_ias_1230","1230","Короткострокові позики","liability_current","False",""
"ua_ias_1240","1240","Короткострокова частка за довгострокові позики","liability_current","False",""
"ua_ias_1300","1300","Довгострокові позики","liability_non_current","False",""
"ua_ias_1301","1301","Довгострокові кредити","liability_non_current","False",""
"ua_ias_1302","1302","Іпотека","liability_non_current","False",""
"ua_ias_1310","1310","Відстрочені податкові зобов’язання","liability_non_current","False",""
"ua_ias_1320","1320","Витрати майбутніх періодів","liability_non_current","False",""
"ua_ias_1330","1330","Інші довгострокові зобов’язання","liability_non_current","False",""
"ua_ias_1400","1400","Статутний капітал","equity","False",""
"ua_ias_1401","1401","Емісійний фонд","equity","False",""
"ua_ias_1402","1402","Вилучений капітал","equity","False",""
"ua_ias_1403","1403","Додатковий оплачений капітал","equity","False",""
"ua_ias_1404","1404","Резерв переоцінки","equity","False",""
"ua_ias_1405","1405","Неоплачений капітал","equity","False",""
"ua_ias_1406","1406","Нерозподілений прибуток (непокритий збиток)","equity","False",""
"ua_ias_1407","1407","Частка меншості","equity","False",""
"ua_ias_1408","1408","Ефект курсових різниць","equity","False",""
"ua_ias_2000","2000","Дохід від продажу (категорія № 1)","income","False",""
"ua_ias_2001","2001","Дохід від продажу (категорія № 2)","income","False",""
"ua_ias_2002","2002","Дохід від продажу (категорія № 3)","income","False",""
"ua_ias_2003","2003","Дохід від продажу (категорія № 4)","income","False",""
"ua_ias_2100","2100","Фінансовий результат","equity_unaffected","False",""
"ua_ias_2200","2200","Собівартість продажу (категорія № 1)","expense_direct_cost","False",""
"ua_ias_2201","2201","Собівартість продажу (категорія № 2)","expense_direct_cost","False",""
"ua_ias_2202","2202","Собівартість продажу (категорія № 3)","expense_direct_cost","False",""
"ua_ias_2203","2203","Собівартість продажу (категорія № 4)","expense_direct_cost","False",""
"ua_ias_2300","2300","Адміністративні витрати","expense","False",""
"ua_ias_2400","2400","Витрати на збут","expense","False",""
"ua_ias_2500","2500","Інші витрати","expense","False",""
"ua_ias_2600","2600","Податок на прибуток","expense","False",""

```

## File: data\template\account.tax-ua_ias.csv

```csv
"id","sequence","name","description","invoice_label","amount","type_tax_use","tax_group_id","price_include","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id"
"sale_tax_template_vat20","9","Реалізація з ПДВ 20%","","+ ПДВ 20%","20.0","sale","tax_group_vat20","","base","invoice",""
"","","","","","","","","","tax","invoice","ua_ias_1204"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_ias_1204"
"sale_tax_template_vat20incl","9","Реалізація в т. ч. ПДВ 20%","","в т. ч. ПДВ 20%","20.0","sale","tax_group_vat20","True","base","invoice",""
"","","","","","","","","","tax","invoice","ua_ias_1204"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_ias_1204"
"sale_tax_template_vat14","10","Реалізація з ПДВ 14%","","+ ПДВ 14%","14.0","sale","tax_group_vat14","","base","invoice",""
"","","","","","","","","","tax","invoice","ua_ias_1204"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_ias_1204"
"sale_tax_template_vat14incl","10","Реалізація в т. ч. ПДВ 14%","","в т. ч. ПДВ 14%","14.0","sale","tax_group_vat14","True","base","invoice",""
"","","","","","","","","","tax","invoice","ua_ias_1204"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_ias_1204"
"sale_tax_template_vat7","11","Реалізація з ПДВ 7%","","+ ПДВ 7%","7.0","sale","tax_group_vat7","","base","invoice",""
"","","","","","","","","","tax","invoice","ua_ias_1204"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_ias_1204"
"sale_tax_template_vat7incl","11","Реалізація в т. ч. ПДВ 7%","","в т .ч. ПДВ 7%","7.0","sale","tax_group_vat7","True","base","invoice",""
"","","","","","","","","","tax","invoice","ua_ias_1204"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_ias_1204"
"sale_tax_template_vat0","12","Реалізація з ПДВ 0%","","ПДВ 0%","0.0","sale","","","base","invoice",""
"","","","","","","","","","tax","invoice",""
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund",""
"sale_tax_template_vat_free","13","Реалізація звільнена від ПДВ","","Звільнено від ПДВ","0.0","sale","","","base","invoice",""
"","","","","","","","","","tax","invoice",""
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund",""
"sale_tax_template_vat_not","14","Реалізація не є об'єктом ПДВ","","Не є об'єктом ПДВ","0.0","sale","","","base","invoice",""
"","","","","","","","","","tax","invoice",""
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund",""
"purchase_tax_template_vat20","19","Придбання з ПДВ 20%","","+ ПДВ 20%","20.0","purchase","tax_group_vat20","","base","invoice",""
"","","","","","","","","","tax","invoice","ua_ias_1140"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_ias_1140"
"purchase_tax_template_vat20incl","19","Придбання в т. ч. ПДВ 20%","","в т. ч. ПДВ 20%","20.0","purchase","tax_group_vat20","True","base","invoice",""
"","","","","","","","","","tax","invoice","ua_ias_1140"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_ias_1140"
"purchase_tax_template_vat14","20","Придбання з ПДВ 14%","","+ ПДВ 14%","14.0","purchase","tax_group_vat14","","base","invoice",""
"","","","","","","","","","tax","invoice","ua_ias_1140"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_ias_1140"
"purchase_tax_template_vat14incl","20","Придбання в т. ч. ПДВ 14%","","в т. ч. ПДВ 14%","14.0","purchase","tax_group_vat14","True","base","invoice",""
"","","","","","","","","","tax","invoice","ua_ias_1140"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_ias_1140"
"purchase_tax_template_vat7","21","Придбання з ПДВ 7%","","+ ПДВ 7%","7.0","purchase","tax_group_vat7","","base","invoice",""
"","","","","","","","","","tax","invoice","ua_ias_1140"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_ias_1140"
"purchase_tax_template_vat7incl","21","Придбання в т. ч. ПДВ 7%","","в т. ч. ПДВ 7%","7.0","purchase","tax_group_vat7","True","base","invoice",""
"","","","","","","","","","tax","invoice","ua_ias_1140"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_ias_1140"
"purchase_tax_template_vat0","22","Придбання з ПДВ 0%","","ПДВ 0%","0.0","purchase","","","base","invoice",""
"","","","","","","","","","tax","invoice",""
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund",""
"purchase_tax_template_vat_free","23","Придбання звільнене від ПДВ","","Звільнено від ПДВ","0.0","purchase","","","base","invoice",""
"","","","","","","","","","tax","invoice",""
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund",""
"purchase_tax_template_vat_not","24","Придбання не є об'єктом ПДВ","","Не є об'єктом ПДВ","0.0","purchase","","","base","invoice",""
"","","","","","","","","","tax","invoice",""
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund",""
"simple_tax_sale_product","30","Дохід від продажу товарів","","Дохід від продажу товарів","0.0","sale","","","","",""
"simple_tax_sale_gift","31","Дохід від безоплатно отриманих товарів","","Дохід від безоплатно отриманих товарів","0.0","sale","","","","",""
"simple_tax_sale_old","32","Дохід від заборгованності, за якою минув строк позивної давності","","Дохід від заборгованності, за якою минув строк позивної давності","0.0","sale","","","","",""
"simple_tax_sale_15","33","Дохід, що оподатковується за ставкою 15%","","Дохід за ставкою 15%","0.0","sale","","","","",""
"simple_tax_purchase_product","40","Витрати від продажу товарів","","Витрати від продажу товарів","0.0","purchase","","","","",""
"simple_tax_purchase_salary","41","Витрати на оплату праці найманих працівників","","Витрати на оплату праці найманих працівників","0.0","purchase","","","","",""
"simple_tax_purchase_esv","42","Витрати на ЄСВ","","Витрати на ЄСВ","0.0","purchase","","","","",""
"simple_tax_purchase_other","43","Витрати інші","","Витрати інші","0.0","purchase","","","","",""

```

## File: data\template\account.tax-ua_psbo.csv

```csv
"id","sequence","name","description","invoice_label","amount","type_tax_use","tax_group_id","price_include","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id"
"sale_tax_template_vat20_psbo","9","Реалізація ПДВ 20%","","+ ПДВ 20%","20.0","sale","tax_group_vat20","","base","invoice",""
"","","","","","","","","","tax","invoice","ua_psbp_6431"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_psbp_6431"
"sale_tax_template_vat20incl_psbo","9","Реалізація в т. ч. ПДВ 20%","","в т. ч. ПДВ 20%","20.0","sale","tax_group_vat20","True","base","invoice",""
"","","","","","","","","","tax","invoice","ua_psbp_6431"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_psbp_6431"
"sale_tax_template_vat14_psbo","10","Реалізація ПДВ 14%","","+ ПДВ 14%","14.0","sale","tax_group_vat14","","base","invoice",""
"","","","","","","","","","tax","invoice","ua_psbp_6431"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_psbp_6431"
"sale_tax_template_vat14incl_psbo","10","Реалізація в т. ч. ПДВ 14%","","в т. ч. ПДВ 14%","14.0","sale","tax_group_vat14","True","base","invoice",""
"","","","","","","","","","tax","invoice","ua_psbp_6431"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_psbp_6431"
"sale_tax_template_vat7_psbo","11","Реалізація ПДВ 7%","","+ ПДВ 7%","7.0","sale","tax_group_vat7","","base","invoice",""
"","","","","","","","","","tax","invoice","ua_psbp_6431"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_psbp_6431"
"sale_tax_template_vat7incl_psbo","11","Реалізація в т. ч. ПДВ 7%","","в т .ч. ПДВ 7%","7.0","sale","tax_group_vat7","True","base","invoice",""
"","","","","","","","","","tax","invoice","ua_psbp_6431"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_psbp_6431"
"sale_tax_template_vat0_psbo","12","Реалізація ПДВ 0%","","ПДВ 0%","0.0","sale","","","base","invoice",""
"","","","","","","","","","tax","invoice",""
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund",""
"sale_tax_template_vat_free_psbo","13","Реалізація звільнена від  ПДВ","","Звільнено від ПДВ","0.0","sale","","","base","invoice",""
"","","","","","","","","","tax","invoice",""
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund",""
"sale_tax_template_vat_not_psbo","14","Реалізація Не є об'єктом ПДВ","","Не є об'єктом ПДВ","0.0","sale","","","base","invoice",""
"","","","","","","","","","tax","invoice",""
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund",""
"purchase_tax_template_vat20_psbo","19","Придбання ПДВ 20%","","+ ПДВ 20%","20.0","purchase","tax_group_vat20","","base","invoice",""
"","","","","","","","","","tax","invoice","ua_psbp_6441"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_psbp_6441"
"purchase_tax_template_vat20incl_psbo","19","Придбання в т. ч. ПДВ 20%","","в т. ч. ПДВ 20%","20.0","purchase","tax_group_vat20","True","base","invoice",""
"","","","","","","","","","tax","invoice","ua_psbp_6441"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_psbp_6441"
"purchase_tax_template_vat14_psbo","20","Придбання ПДВ 14%","","+ ПДВ 14%","14.0","purchase","tax_group_vat14","","base","invoice",""
"","","","","","","","","","tax","invoice","ua_psbp_6441"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_psbp_6441"
"purchase_tax_template_vat14incl_psbo","20","Придбання в т. ч. ПДВ 14%","","в т. ч. ПДВ 14%","14.0","purchase","tax_group_vat14","True","base","invoice",""
"","","","","","","","","","tax","invoice","ua_psbp_6441"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_psbp_6441"
"purchase_tax_template_vat7_psbo","21","Придбання ПДВ 7%","","+ ПДВ 7%","7.0","purchase","tax_group_vat7","","base","invoice",""
"","","","","","","","","","tax","invoice","ua_psbp_6441"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_psbp_6441"
"purchase_tax_template_vat7incl_psbo","21","Придбання в т. ч. ПДВ 7%","","в т. ч. ПДВ 7%","7.0","purchase","tax_group_vat7","True","base","invoice",""
"","","","","","","","","","tax","invoice","ua_psbp_6441"
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund","ua_psbp_6441"
"purchase_tax_template_vat0_psbo","22","Придбання ПДВ 0%","","ПДВ 0%","0.0","purchase","","","base","invoice",""
"","","","","","","","","","tax","invoice",""
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund",""
"purchase_tax_template_vat_free_psbo","23","Придбання звільнене від  ПДВ","","Звільнено від ПДВ","0.0","purchase","","","base","invoice",""
"","","","","","","","","","tax","invoice",""
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund",""
"purchase_tax_template_vat_not_psbo","24","Придбання Не є об'єктом ПДВ","","Не є об'єктом ПДВ","0.0","purchase","","","base","invoice",""
"","","","","","","","","","tax","invoice",""
"","","","","","","","","","base","refund",""
"","","","","","","","","","tax","refund",""
"simple_tax_sale_product_psbo","30","Дохід від продажу  товарів","","Дохід від продажу товарів","0.0","sale","","","","",""
"simple_tax_sale_gift_psbo","31","Дохід від безоплатно отриманих  товарів","","Дохід від безоплатно отриманих товарів","0.0","sale","","","","",""
"simple_tax_sale_old_psbo","32","Дохід від заборгованності за якою минув строк позивної давності","","Дохід від заборгованності, за якою минув строк позивної давності","0.0","sale","","","","",""
"simple_tax_sale_15_psbo","33","Дохід, за ставкою 15%","","Дохід за ставкою 15%","0.0","sale","","","","",""
"simple_tax_purchase_product_psbo","40","Витрати від продажу  товарів","","Витрати від продажу товарів","0.0","purchase","","","","",""
"simple_tax_purchase_salary_psbo","41","Витрати на оплату праці","","Витрати на оплату праці найманих працівників","0.0","purchase","","","","",""
"simple_tax_purchase_esv_psbo","42","Витрати ЄСВ","","Витрати на ЄСВ","0.0","purchase","","","","",""
"simple_tax_purchase_other_psbo","43","Витрати  інші","","Витрати інші","0.0","purchase","","","","",""

```

## File: data\template\account.tax.group-ua_ias.csv

```csv
"id","name","country_id"
"tax_group_vat20","ПДВ 20%","base.ua"
"tax_group_vat14","ПДВ 14%","base.ua"
"tax_group_vat7","ПДВ 7%","base.ua"
"tax_group_vat0","ПДВ 0%","base.ua"
"tax_group_vat_free","Звільнено від ПДВ","base.ua"
"tax_group_not_vat","Не є ПДВ","base.ua"

```

## File: data\template\account.tax.group-ua_psbo.csv

```csv
"id","name","country_id"
"tax_group_vat20","ПДВ 20%","base.ua"
"tax_group_vat14","ПДВ 14%","base.ua"
"tax_group_vat7","ПДВ 7%","base.ua"
"tax_group_vat0","ПДВ 0%","base.ua"
"tax_group_vat_free","Звільнено від ПДВ","base.ua"
"tax_group_not_vat","Не є ПДВ","base.ua"

```

## File: models\template_ua_ias.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ua_ias')
    def _get_ua_ias_template_data(self):
        return {
            'property_account_receivable_id': 'ua_ias_1120',
            'property_account_payable_id': 'ua_ias_1200',
            'property_account_expense_categ_id': 'ua_ias_2200',
            'property_account_income_categ_id': 'ua_ias_2000',
            'property_stock_account_input_categ_id': 'ua_ias_1201',
            'property_stock_account_output_categ_id': 'ua_ias_1121',
            'property_stock_valuation_account_id': 'ua_ias_1100',
            'name': 'План рахунків МСФЗ',
            'code_digits': '6',
            'use_storno_accounting': True,
            'display_invoice_amount_total_words': True,
        }

    @template('ua_ias', 'res.company')
    def _get_ua_ias_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.ua',
                'bank_account_code_prefix': '1112',
                'cash_account_code_prefix': '1111',
                'transfer_account_code_prefix': '1119',
                'account_default_pos_receivable_account_id': 'ua_ias_1122',
                'income_currency_exchange_account_id': 'ua_ias_2100',
                'expense_currency_exchange_account_id': 'ua_ias_2500',
                'account_sale_tax_id': 'sale_tax_template_vat20',
                'account_purchase_tax_id': 'purchase_tax_template_vat20',
            },
        }

```

## File: models\template_ua_psbo.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ua_psbo')
    def _get_ua_psbo_template_data(self):
        return {
            'property_account_receivable_id': 'ua_psbp_361',
            'property_account_payable_id': 'ua_psbp_631',
            'property_account_expense_categ_id': 'ua_psbp_901',
            'property_account_income_categ_id': 'ua_psbp_701',
            'property_stock_account_input_categ_id': 'ua_psbp_2812',
            'property_stock_account_output_categ_id': 'ua_psbp_2811',
            'property_stock_valuation_account_id': 'ua_psbp_281',
            'name': 'План рахунків ПСБО',
            'code_digits': '6',
            'use_storno_accounting': True,
            'display_invoice_amount_total_words': True,
        }

    @template('ua_psbo', 'res.company')
    def _get_ua_psbo_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.ua',
                'bank_account_code_prefix': '311',
                'cash_account_code_prefix': '301',
                'transfer_account_code_prefix': '333',
                'account_default_pos_receivable_account_id': 'ua_psbp_366',
                'income_currency_exchange_account_id': 'ua_psbp_711',
                'expense_currency_exchange_account_id': 'ua_psbp_942',
                'account_sale_tax_id': 'sale_tax_template_vat20_psbo',
                'account_purchase_tax_id': 'purchase_tax_template_vat20_psbo',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_ua_ias
from . import template_ua_psbo

```

