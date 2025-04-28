# Odoo Module: l10n_mn

Category: Localization

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    "name" : "Mongolia - Accounting",
    "version" : "1.0",
    'category': 'Localization',
    "author" : "BumanIT LLC, Odoo S.A.",
    "description": """
This is the module to manage the accounting chart for Mongolia.
===============================================================

    * the Mongolia Official Chart of Accounts,
    * the Tax Code Chart for Mongolia
    * the main taxes used in Mongolia

Financial requirement contributor: Baskhuu Lodoikhuu. BumanIT LLC
""",
    "depends": ['account'],
    'data': [
        'data/account_chart_template_data.xml',
        'data/account.account.tag.csv',
        'data/account.account.template.csv',
        'data/account.tax.group.csv',
        'data/account_tax_template_data.xml',
        'data/account_fiscal_position_template_data.xml',
        'data/account.chart.template.csv',
        'data/account_chart_template_configuration_data.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.tag.csv

```csv
id,name,applicability,country_id:id
account.account_tag_operating,CF: Үндсэн: Бусад,accounts,
account.account_tag_financing,CF: Хөрөнгө оруулалт: Бусад Урт хугацаат хөрөнгө,accounts,
account.account_tag_investing,CF: Санхүү: Ногдол ашиг,accounts,
account_tag_operating_product,CF: Үндсэн: Бараа үйлчилгээ,accounts,
account_tag_operating_loyalty,CF: Үндсэн: Эрхийн шимтгэл хураамж,accounts,
account_tag_operating_insurance,CF: Үндсэн: Нийгмийн Даатгал,accounts,
account_tag_operating_insurance2,CF: Үндсэн: Бусад Даатгал,accounts,
account_tag_operating_taxrefund,CF: Үндсэн: Татвар,accounts,
account_tag_operating_funding,CF: Үндсэн: Татаас санхүүжилт,accounts,
account_tag_operating_employee,CF: Үндсэн: Ажилчид,accounts,
account_tag_operating_cost,CF: Үндсэн: Ашиглалтын зардал,accounts,
account_tag_operating_transport,CF: Үндсэн: Түлш тээвэр сэлбэг,accounts,
account_tag_operating_interest,CF: Үндсэн: Зээлийн хүү,accounts,
account_tag_investing_fasset,CF: Хөрөнгө оруулалт: Үндсэн хөрөнгө,accounts,
account_tag_investing_iasset,CF: Хөрөнгө оруулалт: Биет бус хөрөнгө,accounts,
account_tag_investing_invest,CF: Хөрөнгө оруулалт: Хөрөнгө оруулалт,accounts,
account_tag_investing_loan,CF: Хөрөнгө оруулалт: Зээл урьдчилгаа,accounts,
account_tag_investing_interest,CF: Хөрөнгө оруулалт: Зээлийн хүү,accounts,
account_tag_investing_dividends,CF: Хөрөнгө оруулалт: Ногдол ашиг,accounts,
account_tag_financing_loan,"CF: Санхүү: Зээл, өрийн үнэт цаас",accounts,
account_tag_financing_rental,CF: Санхүү: Санхүүгийн түрээс,accounts,
account_tag_financing_stock,CF: Санхүү: Хувьцаа өмчийн бусад үнэт цаас,accounts,
account_tag_financing_donation,CF: Санхүү: Хандив,accounts,
account_tag_exchange,CF: Валютын ханшийн зөрүү,accounts,
vat_report_tag2,ТТ3а: НӨАТ-аас чөлөөлөгдөх борлуулалтын орлого (2),taxes,base.mn
vat_report_tag5,ТТ3a: Дотоодын зах зээлд борлуулсан барааны борлуулалтын орлого (5),taxes,base.mn
vat_report_tag6,ТТ3a: Бусад барааны борлуулалтын орлого (6),taxes,base.mn
vat_report_tag7,ТТ3a: Эрх борлуулсны орлого (7),taxes,base.mn
vat_report_tag8,ТТ3a: Татан буугдах үед үлдсэн барааны орлого (8),taxes,base.mn
vat_report_tag9,ТТ3a: Өрийн төлбөрт шилжүүлсэн барааны орлого (9),taxes,base.mn
vat_report_tag11,ТТ3a: Наториатын үйлчилгээний орлого (11),taxes,base.mn
vat_report_tag12,ТТ3a: Өрийн төлбөрт тооцсон ажил үйлчилгээний орлого (12),taxes,base.mn
vat_report_tag13,"ТТ3a: Цахилгаан,  дулаан, ус, шуудан, холбооны үйлчилгээ үзүүлсний орлого (13)",taxes,base.mn
vat_report_tag14,"ТТ3a: Бараа түрээслүүлэх, эзэмшүүлэх, ашиглуулах үйлчилгээний орлого (14)",taxes,base.mn
vat_report_tag15,"ТТ3a: Байр түрээслүүлэх, эзэмшүүлэх, ашиглуулах үйлчилгээний орлого (15)",taxes,base.mn
vat_report_tag16,"ТТ3a: Үл хөдлөх, хөдлөх хөрөнгө түрээслүүлэх, эзэмшүүлэх ашиглуулах үйлчилгээий орлого (16)",taxes,base.mn
vat_report_tag17,"ТТ3a: Шинэ бүтээл, загвар, зохиогчийн эрх, барааны тэмдэг худалдсаны орлого (17)",taxes,base.mn
vat_report_tag18,"ТТ3a: Хонжворт сугалаа, төлбөрт таавар, бооцоот тоглоомын орлого (18)",taxes,base.mn
vat_report_tag19,ТТ3a: Зуучлалын үйлчилгээний орлого (19),taxes,base.mn
vat_report_tag20,"ТТ3a: Бусдаас авсан хүү, торгууль, алдангийн орлого (20)",taxes,base.mn
vat_report_tag21,ТТ3a: Хөрөнгийн үнэлгээний үйлчилгээний орлого (21),taxes,base.mn
vat_report_tag22,"ТТ3a: Төсвийн санхүүжилт, татаас, урамшууллын орлого (22)",taxes,base.mn
vat_report_tag23,"ТТ3a: Өмгөөлөл, хууль зүйн үйлчилгээний орлого (23)",taxes,base.mn
vat_report_tag24,"ТТ3a: Үсчин, гоо сайхан, засвар үйлчилгээ, угаалга, хими цэвэрлэгээний үйлчилгээний орлого (24)",taxes,base.mn
vat_report_tag25,ТТ3a: Хуулийн 13-т зааснаас бусад үйлчилгээний орлого (25),taxes,base.mn
vat_report_tag28,ТТ3a: Экспортонд гаргасан барааны борлуулалтын орлого (28),taxes,base.mn
vat_report_tag29,ТТ3a: Экспортолсон үйлчилгээний борлуулалтын орлого (29),taxes,base.mn
vat_report_tag33,"ТТ3а: Худалдан авсан нийт бараа, ажил үйлчилгээ (33)",taxes,base.mn
vat_report_tag35,"ТТ3a: НӨАТ-тэй авсан Импортын бараа, ажил үйлчилгээний худалдан авалт (35)",taxes,base.mn
vat_report_tag36,"ТТ3a: Дотоодын зах зээлээс НӨАТ-тэй авсан бараа, ажил, үйлчилгээ (36)",taxes,base.mn
vat_report_tag37,"ТТ3a: Суутган төлөгчөөр бүртгүүлэх үед НӨАТ-тэй авсан бараа, үйлчилгээ (37)",taxes,base.mn
vat_report_tag38,"ТТ3a: Мал аж ахуй, тариалан эрхлэгчээс НӨАТ-тэй авсан бараа, үйлчилгээ (38)",taxes,base.mn
vat_report_tag41,"ТТ3а: Суудлын автомашин, түүний эд анги сэлбэгт төлсөн  НӨАТ (41)",taxes,base.mn
vat_report_tag42,"ТТ3а: Ажлын хэрэгцээнд зориулж авсан бараа, үйлчилгээнд төлсөн НӨАТ (42)",taxes,base.mn
vat_report_tag43,"ТТ3а: Үндсэн хөрөнгө бэлтгэхэд зориулж импортоор авсан бараа, үйлчилгээнд төлсөн  НӨАТ (43)",taxes,base.mn
vat_report_tag44,"ТТ3а: Чөлөөлөгдөх үйлдвэрлэл, үйлчилгээнд зориулж авсан бараа, ажил, үйлчилгээнд төлсөн НӨАТ (44)",taxes,base.mn
vat_report_tag45,"ТТ3а: Хайгуулын ажил болон ашиглалтын өмнөх үйл ажиллагаанд зориулж импортоор авсан бараа, ажил, үйлчилгээнд төлсөн НӨАТ (45)",taxes,base.mn
vat_report_tag46,"ТТ3а: Импортоор оруулсан бусад хамааралгүй бараа, ажил, үйлчилгээнд төлсөн НӨАТ (46)",taxes,base.mn
vat_report_tag48,ТТ3а: Cанхүүгийн түрээсээр дотоодын зах зээлд борлуулсны орлого (48),taxes,base.mn
vat_report_tag49,"ТТ3а: Факторинг, форфайтинг хэлцлийн үйлчилгээний орлого (49)",taxes,base.mn
vat_report_tag51,ТТ3а: Санхүүгийн түрээсийн зүйлийг худалдан авахад төлсөн түрээсийн төлбөр (51),taxes,base.mn
vat_report_tag52,"ТТ3а: Факторинг, форфайтинг хэлцлийг авахад төлсөн төлбөр (52)",taxes,base.mn
vat_report_tag58,"ТТ3а: Борлуулалтын орлогын буцаалт, хөнгөлөлт (58)",accounts,
vat_report_tag59,"ТТ3а: Худалдан авалтын буцаалт, хөнгөлөлт (59)",accounts,
tax_report_tag2,ТТ02: Татвараас чөлөөлөгдөх орлого (2),accounts,
tax_report_tag5,ТТ02: Бусад орлогын дүн (5),accounts,
tax_report_tag7,ТТ02: Үндсэн ажил үйлчилгээний орлого (7),accounts,
tax_report_tag8,ТТ02: Туслах ажил үйлчилгээний орлого (8),accounts,
tax_report_tag9,"ТТ02: Хувьцаа, үнэт цаас борлуулсны орлого (9)",accounts,
tax_report_tag10,"ТТ02: Үнэ төлбөргүйгээр бусдаас авсан бараа, үйлчилгээ (10)",accounts,
tax_report_tag11,ТТ02: Биет бус хөрөнгө борлуулсны орлого (11),accounts,
tax_report_tag12,ТТ02: Техникийн зөвлөх үйлчилгээний орлого (12),accounts,
tax_report_tag13,"ТТ02: Гэрээний хүү, торгууль, хохирлын нөхөн төлбөрийн орлого (13)",accounts,
tax_report_tag14,ТТ02: Гадаад валютын ханшийн зөрүүгийн бодит орлого (14),accounts,
tax_report_tag15,ТТ02: Хөдлөх болон үл хөдлөх эд хөрөнгийн түрээсийн орлого (15),accounts,
tax_report_tag16,ТТ02: Хөдлөх эд хөрөнгө борлуулсны орлого (16),accounts,
tax_report_tag17,ТТ02: Албан татвар ногдох бусад орлого (17),accounts,
tax_report_tag22,ТТ02: Татвар ногдох орлогоос хасагдахгүй зардал (22),accounts,
tax_report_tag23,ТТ02: Татвар ногдох орлогыг бууруулах (23),accounts,
tax_report_tag25,ТТ02: Сайн дурын даатгалын хураамжийн хэтрэлт (25),accounts,
tax_report_tag27,ТТ02: Өмнөх жилийн татварын алдагдлаас тайлант хугацаанд шилжүүлсэн дүн (27),accounts,
tax_report_tag30,ТТ02: Татварын хөнгөлөлт (30),accounts,
tax_report_tag32,"ТТ02: Эротик хэвлэл, бичлэг, тоглолт борлуулалтын орлого (32)",accounts,
tax_report_tag34,"ТТ02: Таавар, бооцоот тоглоом, хонжворт сугалааны орлого (34)",accounts,
tax_report_tag35,"ТТ02: Таавар, бооцоот тоглоом, хонжворт сугалааны орлого олоход зарцуулсан зардал (35)",accounts,
tax_report_tag36,ТТ02: Хонжворт олгосон мөнгө болон барааны үнэ (36),accounts,
tax_report_tag39,ТТ02: Хүүгийн орлого (39),accounts,
tax_report_tag41,"ТТ02: Гадаад улсад олсон ногдол ашиг, хүүгийн орлого (41)",accounts,
tax_report_tag43,ТТ02: Гадаад улсад олсон орлого (43),accounts,
tax_report_tag46,ТТ02: Ногдол ашгийн орлого (46),accounts,
tax_report_tag48,ТТ02: Эрхийн шимтгэлийн орлого (48),accounts,
tax_report_tag50,ТТ02: Эрх борлуулсны орлого (50),accounts,
tax_report_tag52,ТТ02: Үл хөдлөх хөрөнгө борлуулсны орлого (52),accounts,

```

## File: data\account.chart.template.csv

```csv
id,property_account_receivable_id/id,property_account_payable_id/id,property_account_expense_categ_id/id,property_account_income_categ_id/id,income_currency_exchange_account_id/id,expense_currency_exchange_account_id/id,property_stock_account_input_categ_id/id,property_stock_account_output_categ_id/id,property_stock_valuation_account_id/id,default_pos_receivable_account_id/id
mn_chart_1,account_template_1201_0201,account_template_3101_0201,account_template_6101_0101,account_template_5101_0101,account_template_5301_0201,account_template_5302_0201,account_template_1407_0101,account_template_1408_0101,account_template_1401_0101,account_template_1201_0202

```

## File: data\account.tax.group.csv

```csv
id,name,sequence
account_tax_group1,НӨАТ ногдох бараа,10
account_tax_group2,НӨАТ ногдох үйлчилгээ,10
account_tax_group3,Экспортын борлуулалт,11
account_tax_group4,"Худалдан авсан бараа, үйлчилгээ",12
account_tax_group5,Санхүүгийн түрээсийн зүйл,13
account_tax_group6,НӨАТ-с чөлөөлөгдөх,14

```

## File: data\account_chart_template_configuration_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <function model="account.chart.template" name="try_loading">
        <value eval="[ref('l10n_mn.mn_chart_1')]"/>
    </function>

</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <menuitem id="account_reports_mn_statements_menu" name="Mongolia" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_user"/>

    <record id="mn_chart_1" model="account.chart.template">
        <field name="name">Mongolia</field>
        <field name="bank_account_code_prefix">11</field>
        <field name="cash_account_code_prefix">10</field>
        <field name="transfer_account_code_prefix">1109</field>
        <field name="code_digits">8</field>
        <field name="currency_id" ref="base.MNT"/>
        <field name="use_anglo_saxon" eval="True"/>
    </record>

</odoo>

```

## File: data\account_fiscal_position_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- = = = = = = = = = = = = = = = -->
    <!-- Fiscal Position Templates     -->
    <!-- = = = = = = = = = = = = = = = -->
    <record id="fiscal_position_vat0" model="account.fiscal.position.template">
        <field name="name">Дотоодын борлуулалт - Худалдан авалт</field>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="auto_apply" eval="True" />
        <field name="country_id" ref="base.mn"/>
    </record>

    <record id="fiscal_position_vat2" model="account.fiscal.position.template">
        <field name="name">НӨАТ 0% (3000000)</field>
        <field name="chart_template_id" ref="mn_chart_1"/>
    </record>

    <record id="fiscal_position_vat3" model="account.fiscal.position.template">
        <field name="name">НӨАТ-гүй (2000000)</field>
        <field name="chart_template_id" ref="mn_chart_1"/>
    </record>

    <record id="fiscal_position_vat4" model="account.fiscal.position.template">
        <field name="name">Гадаад борлуулалт - Худалдан авалт</field>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="auto_apply" eval="True" />
    </record>

    <!-- = = = = = = = = = = = = = = = -->
    <!-- Fiscal Position Tax Templates -->
    <!-- = = = = = = = = = = = = = = = -->

    <record id="fp_tax_template_vat2_sale_vat1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat1" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat4" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat5" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat6" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat7" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat7" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat8" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat8" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat9" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat10" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat11" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat11" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat12" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat12" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

        <record id="fp_tax_template_vat2_sale_vat13" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat13" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat14" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat15" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat15" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat16" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat16" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat17" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat17" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat18" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat18" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat19" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat19" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat20" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat20" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat21" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat21" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat22" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat22" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat1" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat4" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat5" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat6" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat7" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat7" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat8" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat8" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat9" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat10" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat11" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat11" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat12" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat12" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat13" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat13" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat14" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat15" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_dest_id" ref="account_tax_sale_vat15" />
        <field name="tax_src_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat16" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat16" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat17" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat17" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat18" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat18" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat19" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat19" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat20" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat20" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat21" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat21" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat22" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat22" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat1" />
        <field name="tax_dest_id" ref="account_tax_sale_vat23" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat4" />
        <field name="tax_dest_id" ref="account_tax_sale_vat23" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat5" />
        <field name="tax_dest_id" ref="account_tax_sale_vat23" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat6" />
        <field name="tax_dest_id" ref="account_tax_sale_vat23" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat7" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat7" />
        <field name="tax_dest_id" ref="account_tax_sale_vat23" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat8" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat8" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat9" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat10" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat11" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat11" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat12" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat12" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat13" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat13" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat14" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat15" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_dest_id" ref="account_tax_sale_vat15" />
        <field name="tax_src_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat16" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat16" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat17" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat17" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat18" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat18" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat19" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat19" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat20" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat20" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat21" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat21" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat22" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat22" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_purchase_vat1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_purchase_vat1" />
        <field name="tax_dest_id" ref="account_tax_purchase_vat2" />
    </record>

</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="account_tax_sale_vat1" model="account.tax.template">
        <field name="name">НӨАТ бараа</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">1</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group1"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag5')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag5')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag5')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag5')],
            }),
        ]"/>
    </record>

    <record id="account_tax_purchase_vat1" model="account.tax.template">
        <field name="name">НӨАТ-тэй худалдан авалт</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">1</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag36'),ref('vat_report_tag33')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag36')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag36'),ref('vat_report_tag33')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag36')],
            }),
        ]"/>
    </record>

    <record id="account_tax_purchase_vat2" model="account.tax.template">
        <field name="name">НӨАТ-гүй худалдан авалт</field>
        <field name="amount">0.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">0%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33')],
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
                'tag_ids': [ref('vat_report_tag33')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat2" model="account.tax.template">
        <field name="name">НӨАТ 0% борлуулалт</field>
        <field name="amount">0.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">0%</field>
        <field name="tax_group_id" ref="account_tax_group6"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag2')],
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
                'tag_ids': [ref('vat_report_tag2')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat3" model="account.tax.template">
        <field name="name">НӨАТ чөлөөлөх борлуулалт</field>
        <field name="amount">0.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">0%</field>
        <field name="tax_group_id" ref="account_tax_group6"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag2')],
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
                'tag_ids': [ref('vat_report_tag2')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat4" model="account.tax.template">
        <field name="name">НӨАТ бусад барааны борлуулалт</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group1"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag6')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag6')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag6')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag6')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat5" model="account.tax.template">
        <field name="name">НӨАТ эрх борлуулалт</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group1"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag7')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag7')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag7')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag7')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat6" model="account.tax.template">
        <field name="name">НӨАТ татан буугдах үеийн борлуулалт</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group1"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag8')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag8')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag8')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag8')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat7" model="account.tax.template">
        <field name="name">НӨАТ өрийн төлбөрт өгсөн бараа</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group1"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag9')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag9')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag9')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag9')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat8" model="account.tax.template">
        <field name="name">НӨАТ нотариат үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag11')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag11')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag11')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag11')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat9" model="account.tax.template">
        <field name="name">НӨАТ өрийн төлбөрт өгсөн үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag12')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag12')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag12')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag12')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat10" model="account.tax.template">
        <field name="name">НӨАТ цахилгаан, дулаан, ус, шуудан, холбоо</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag13')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag13')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag13')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag13')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat11" model="account.tax.template">
        <field name="name">НӨАТ бараа түрээслүүлэх, эзэмшүүлэх</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag14')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag14')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag14')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag14')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat12" model="account.tax.template">
        <field name="name">НӨАТ байр түрээслүүлэх, эзэмшүүлэх</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag15')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag15')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag15')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag15')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat13" model="account.tax.template">
        <field name="name">НӨАТ хөрөнгө түрээслүүлэх, эзэмшүүлэх</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag16')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag16')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag16')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag16')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat14" model="account.tax.template">
        <field name="name">НӨАТ бүтээл, загвар, зохиогчийн эрх</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag17')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag17')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag17')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag17')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat15" model="account.tax.template">
        <field name="name">НӨАТ хонжворт сугалаа, таавар, бооцоо</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag18')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag18')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag18')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag18')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat16" model="account.tax.template">
        <field name="name">НӨАТ зуучлалын үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag19')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag19')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag19')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag19')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat17" model="account.tax.template">
        <field name="name">НӨАТ авсан хүү торгууль, алданги</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag20')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag20')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag20')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag20')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat18" model="account.tax.template">
        <field name="name">НӨАТ хөрөнгийн үнэлгээний үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag21')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag21')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag21')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag21')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat19" model="account.tax.template">
        <field name="name">НӨАТ төсвийн санхүүжилт, татаас</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag22')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag22')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag22')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag22')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat20" model="account.tax.template">
        <field name="name">НӨАТ хууль зүйн үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag23')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag23')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag23')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag23')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat21" model="account.tax.template">
        <field name="name">НӨАТ үсчин, гоо сайхан, угаалга, цэвэрлэгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag24')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag24')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag24')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag24')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat22" model="account.tax.template">
        <field name="name">НӨАТ бусад үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag25')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag25')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag25')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag25')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat23" model="account.tax.template">
        <field name="name">НӨАТ 0% экспорт бараа</field>
        <field name="amount">0.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">30</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">0%</field>
        <field name="tax_group_id" ref="account_tax_group3"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag28')],
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
                'tag_ids': [ref('vat_report_tag28')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat24" model="account.tax.template">
        <field name="name">НӨАТ 0% экспорт үйлчилгээ</field>
        <field name="amount">0.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">30</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">0%</field>
        <field name="tax_group_id" ref="account_tax_group3"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag29')],
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
                'tag_ids': [ref('vat_report_tag29')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="account_tax_purchase_vat3" model="account.tax.template">
        <field name="name">НӨАТ импортоор авсан бараа, үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'),ref('vat_report_tag35')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag35')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'),ref('vat_report_tag35')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag35')],
            }),
        ]"/>
    </record>

    <record id="account_tax_purchase_vat4" model="account.tax.template">
        <field name="name">НӨАТ төлөгчөөр бүртгүүлэх үед худалдан авсан</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'),ref('vat_report_tag37')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag37')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'),ref('vat_report_tag37')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag37')],
            }),
        ]"/>
    </record>

    <record id="account_tax_purchase_vat5" model="account.tax.template">
        <field name="name">НӨАТ мал аж ахуй эрхлэгчээс худалдан авсан</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'),ref('vat_report_tag38')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag38')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'),ref('vat_report_tag38')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag38')],
            }),
        ]"/>
    </record>

    <record id="account_tax_purchase_vat6" model="account.tax.template">
        <field name="name">НӨАТ автомашин, эд анги худалдан авалт</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag41')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag41')],
            }),
        ]"/>
    </record>

    <record id="account_tax_purchase_vat7" model="account.tax.template">
        <field name="name">НӨАТ ажлын хэрэгцээнд авсан бараа</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag42')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag42')],
            }),
        ]"/>
    </record>

    <record id="account_tax_purchase_vat8" model="account.tax.template">
        <field name="name">НӨАТ импортоор авсан үндсэн хөрөнгө</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag43')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag43')],
            }),
        ]"/>
    </record>

    <record id="account_tax_purchase_vat9" model="account.tax.template">
        <field name="name">НӨАТ чөлөөлөгдөх үйлдвэрлэлд зориулсан бараа, үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag44')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag44')],
            }),
        ]"/>
    </record>

    <record id="account_tax_purchase_vat10" model="account.tax.template">
        <field name="name">НӨАТ хайгуулд зориулсан импортын бараа, үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag45')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag45')],
            }),
        ]"/>
    </record>

    <record id="account_tax_purchase_vat11" model="account.tax.template">
        <field name="name">НӨАТ импортын бусад хамааралгүй бараа, үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag46')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag46')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat25" model="account.tax.template">
        <field name="name">НӨАТ дотоодод зарсан санхүүгийн түрээс</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">50</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group5"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag48')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag48')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag48')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag48')],
            }),
        ]"/>
    </record>

    <record id="account_tax_sale_vat26" model="account.tax.template">
        <field name="name">НӨАТ Факторинг, форфайтинг хэлцлийн үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">50</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group5"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag49')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag49')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag49')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag49')],
            }),
        ]"/>
    </record>

    <record id="account_tax_purchase_vat12" model="account.tax.template">
        <field name="name">НӨАТ Санхүүгийн түрээсийн худалдан авалт</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">50</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group5"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag51')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag51')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag51')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag51')],
            }),
        ]"/>
    </record>

    <record id="account_tax_purchase_vat13" model="account.tax.template">
        <field name="name">НӨАТ Факторинг, форфайтинг худалдан авалт</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">50</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group5"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag52')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag52')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag52')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag52')],
            }),
        ]"/>
    </record>

</odoo>

```

