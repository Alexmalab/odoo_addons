# Odoo Module: l10n_es

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

# List of contributors:
# Jordi Esteve <jesteve@zikzakmedia.com>
# Ignacio Ibeas <ignacio@acysos.com>
# Dpto. Consultoría Grupo Opentia <consultoria@opentia.es>
# Pedro M. Baeza <pedro.baeza@tecnativa.com>
# Carlos Liébana <carlos.liebana@factorlibre.com>
# Hugo Santos <hugo.santos@factorlibre.com>
# Albert Cabedo <albert@gafic.com>
# Olivier Colson <oco@odoo.com>
# Roberto Lizana <robertolizana@trey.es>

{
    "name" : "Spain - Accounting (PGCE 2008)",
    "version" : "5.0",
    "author" : "Spanish Localization Team",
    'category': 'Localization',
    "description": """
Spanish charts of accounts (PGCE 2008).
========================================

    * Defines the following chart of account templates:
        * Spanish general chart of accounts 2008
        * Spanish general chart of accounts 2008 for small and medium companies
        * Spanish general chart of accounts 2008 for associations
    * Defines templates for sale and purchase VAT
    * Defines tax templates
    * Defines fiscal positions for spanish fiscal legislation
    * Defines tax reports mod 111, 115 and 303
""",
    "depends" : [
        "account",
        "base_iban",
        "base_vat",
    ],
    "data" : [
        'data/account_group.xml',
        'data/account_chart_template_data.xml',
        'data/account.account.template-common.csv',
        'data/account.account.template-pymes.csv',
        'data/account.account.template-assoc.csv',
        'data/account.account.template-full.csv',
        'data/account_chart_template_account_account_link.xml',
        'data/account_data.xml',
        'data/account_tax_data.xml',
        'data/account_fiscal_position_template_data.xml',
        'data/account_chart_template_configure_data.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template-assoc.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","group_id/id","reconcile"
"account_assoc_100","Dotación fundacional","100","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_assoc","l10n_es.account_group_100","False"
"account_assoc_1030","Fundadores, parte no desembolsada en fundaciones","1030","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_assoc","l10n_es.account_group_103","False"
"account_assoc_1034","Asociados, parte no desembolsada en asociaciones","1034","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_assoc","l10n_es.account_group_103","False"
"account_assoc_1040","Fundadores, por aportaciones no dinerarias pendientes, en fundaciones","1040","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_assoc","l10n_es.account_group_104","False"
"account_assoc_1044","Asociados, por aportaciones no dinerarias pendientes, en asociaciones","1044","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_assoc","l10n_es.account_group_104","False"
"account_assoc_120","Remanente","120","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_assoc","l10n_es.account_group_120","False"
"account_assoc_121","Resultados negativos de ejercicios anteriores","121","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_assoc","l10n_es.account_group_121","False"
"account_assoc_129","Resultado del ejercicio","129","account.data_unaffected_earnings","l10n_es.account_chart_template_assoc","l10n_es.account_group_129","False"
"account_assoc_1300","Subvenciones del Estado","1300","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_assoc","l10n_es.account_group_130","False"
"account_assoc_1301","Subvenciones de otras Administraciones Públicas","1301","account.data_account_type_equity","l10n_es.account_chart_template_assoc","l10n_es.account_group_130","False"
"account_assoc_1320","Otras subvenciones","1320","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_assoc","l10n_es.account_group_132","False"
"account_assoc_1321","Otras donaciones y legados","1321","account.data_account_type_equity","l10n_es.account_chart_template_assoc","l10n_es.account_group_132","False"
"account_assoc_207","Derechos sobre activos cedidos en uso","207","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_207","False"
"account_assoc_2400","Monumentos","2400","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_240","False"
"account_assoc_2401","Jardines históricos","2401","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_240","False"
"account_assoc_2402","Conjuntos históricos","2402","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_240","False"
"account_assoc_2403","Sitios históricos","2403","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_240","False"
"account_assoc_2404","Zonas arqueológicas","2404","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_240","False"
"account_assoc_2490","Anticipos sobre bienes inmuebles del Patrimonio Histórico","2490","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_249","False"
"account_assoc_2491","Anticipos sobre archivos del Patrimonio Histórico","2491","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_249","False"
"account_assoc_2492","Anticipos sobre bibliotecas del Patrimonio Histórico","2492","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_249","False"
"account_assoc_2493","Anticipos sobre museos del Patrimonio Histórico","2493","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_249","False"
"account_assoc_2494","Anticipos sobre bienes muebles del Patrimonio Histórico","2494","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_249","False"
"account_assoc_2807","Amortización acumulada de derechos sobre activos cedidos en uso","2807","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_280","False"
"account_assoc_2830","Cesiones de uso del inmovilizado intangible","2830","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_283","False"
"account_assoc_2831","Cesiones de uso del inmovilizado material","2831","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_283","False"
"account_assoc_2907","Deterioro del valor sobre activos cedidos en uso","2907","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_290","False"
"account_assoc_2935","Deterioro de valor de participaciones a largo plazo en otras partes vinculadas","2935","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_293","False"
"account_assoc_296","Deterioro de  valor de participaciones en el patrimonio neto a largo plazo","296","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_296","False"
"account_assoc_2990","Deterioro de valor de bienes del Patrimonio Histórico","2990","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_299","False"
"account_assoc_2991","Deterioro de valor de archivos","2991","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_299","False"
"account_assoc_2992","Deterioro de valor de bibliotecas","2992","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_299","False"
"account_assoc_2993","Deterioro de valor de Museos","2993","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_299","False"
"account_assoc_2994","Deterioro de valor de bienes muebles","2994","account.data_account_type_fixed_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_299","False"
"account_assoc_412","Beneficiarios, acreedores","412","account.data_account_type_payable","l10n_es.account_chart_template_assoc","l10n_es.account_group_412","True"
"account_assoc_447","Usuarios, deudores","447","account.data_account_type_receivable","l10n_es.account_chart_template_assoc","l10n_es.account_group_447","True"
"account_assoc_4480","Patrocinadores","4480","account.data_account_type_receivable","l10n_es.account_chart_template_assoc","l10n_es.account_group_448","True"
"account_assoc_4482","Afiliados","4482","account.data_account_type_receivable","l10n_es.account_chart_template_assoc","l10n_es.account_group_448","True"
"account_assoc_4489","Otros deudores","4489","account.data_account_type_receivable","l10n_es.account_chart_template_assoc","l10n_es.account_group_448","True"
"account_assoc_464","Entregas para gastos a justificar","464","account.data_account_type_receivable","l10n_es.account_chart_template_assoc","l10n_es.account_group_464","True"
"account_assoc_4707","Hacienda Pública, deudora por colaboración en la entrega y distribución de subvenciones (art.12 Ley de Subvenciones)","4707","account.data_account_type_receivable","l10n_es.account_chart_template_assoc","l10n_es.account_group_470","True"
"account_assoc_4757","Hacienda Pública, acreedora por subvenciones recibidas en concepto de entidad colaboradora (art.12 Ley de Subvencioens)","4757","account.data_account_type_receivable","l10n_es.account_chart_template_assoc","l10n_es.account_group_475","True"
"account_assoc_490","Deterioro de valor de créditos por operaciones de la actividad","490","account.data_account_type_current_liabilities","l10n_es.account_chart_template_assoc","l10n_es.account_group_490","False"
"account_assoc_551","Cuenta corriente con patronos y otros","551","account.data_account_type_current_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_551","False"
"account_assoc_5935","Deterioro de valor de participaciones a corto plazo en otras partes vinculadas","5935","account.data_account_type_current_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_593","False"
"account_assoc_596","Deterioro de valor de participaciones a corto plazo","596","account.data_account_type_current_assets","l10n_es.account_chart_template_assoc","l10n_es.account_group_596","False"
"account_assoc_6501","Ayudas monetarias individuales","6501","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_650","False"
"account_assoc_6502","Ayudas monetarias a entidades","6502","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_650","False"
"account_assoc_6503","Ayudas monetarias realizadas a través de otras entidades o centros","6503","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_650","False"
"account_assoc_6504","Ayudas monetarias de cooperación internacional","6504","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_650","False"
"account_assoc_6510","Beneficio transferido (gestor)","6510","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_651","False"
"account_assoc_6511","Ayudas no monetarias individuales","6511","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_651","False"
"account_assoc_6512","Ayudas no monetarias a entidades","6512","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_651","False"
"account_assoc_6513","Ayudas no monetarias realizadas a través de otras entidades o centros","6513","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_651","False"
"account_assoc_6514","Ayudas no monetarias de cooperación internacional","6514","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_651","False"
"account_assoc_653","Compensación de gastos por prestaciones de colaboración","653","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_653","False"
"account_assoc_654","Reembolsos de gastos al órgano de gobierno","654","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_654","False"
"account_assoc_655","Pérdidas de créditos derivados de la actividad incobrables","655","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_655","False"
"account_assoc_6560","Beneficio transferido (gestor)","6560","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_656","False"
"account_assoc_6561","Pérdida soportada(partícipe o asociado no gestor)","6561","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_656","False"
"account_assoc_658","Reintegro de subvenciones, donaciones y legados recibidos, afectos a la actividad propia de la entidad ","658","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_658","False"
"account_assoc_663","Pérdidas por valoración de instrumentos financieros por su valor razonable","663","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_663","False"
"account_assoc_6710","Pérdidas procedentes del inmovilizado material","6710","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_671","False"
"account_assoc_6711","Pérdidas procedentes de bienes del Patrimonio Histórico","6711","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_671","False"
"account_assoc_6910","Pérdidas por deterioro del inmovilizado material","6910","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_691","False"
"account_assoc_6911","Pérdidas por deterioro de bienes del Patrimonio Histórico","6911","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_691","False"
"account_assoc_694","Pérdidas por deterioro de créditos por operaciones de la actividad","694","account.data_account_type_expenses","l10n_es.account_chart_template_assoc","l10n_es.account_group_694","False"
"account_assoc_720","Cuotas de asociados y afiliados","720","account.data_account_type_revenue","l10n_es.account_chart_template_assoc","l10n_es.account_group_720","False"
"account_assoc_721","Cuotas de usuarios","721","account.data_account_type_revenue","l10n_es.account_chart_template_assoc","l10n_es.account_group_721","False"
"account_assoc_722","Promociones de captación de recursos","722","account.data_account_type_revenue","l10n_es.account_chart_template_assoc","l10n_es.account_group_722","False"
"account_assoc_7230","Patrocinio","7230","account.data_account_type_revenue","l10n_es.account_chart_template_assoc","l10n_es.account_group_723","False"
"account_assoc_7231","Patrocinio publicitario","7231","account.data_account_type_revenue","l10n_es.account_chart_template_assoc","l10n_es.account_group_723","False"
"account_assoc_7233","Colaboraciones empresariales","7233","account.data_account_type_revenue","l10n_es.account_chart_template_assoc","l10n_es.account_group_723","False"
"account_assoc_728","Ingresos por reintegro de ayudas y asignaciones","728","account.data_account_type_revenue","l10n_es.account_chart_template_assoc","l10n_es.account_group_728","False"
"account_assoc_763","Beneficios por valoración de instrumentos financieros por su valor razonable","763","account.data_account_type_revenue","l10n_es.account_chart_template_assoc","l10n_es.account_group_763","False"
"account_assoc_791","Reversión del deterioro del inmovilizado material y de bienes del Patrimonio Histórico","791","account.data_account_type_revenue","l10n_es.account_chart_template_assoc","l10n_es.account_group_791","False"
"account_assoc_794","Reversión del deterioro de créditos por operaciones de la actividad","794","account.data_account_type_revenue","l10n_es.account_chart_template_assoc","l10n_es.account_group_794","False"
"account_assoc_7962","Reversión del deterioro de participaciones en instrumentos de patrimonio neto a largo plazo, otras partes vinculadas","7962","account.data_account_type_revenue","l10n_es.account_chart_template_assoc","l10n_es.account_group_796","False"
"account_assoc_7963","Reversión del deterioro de participaciones en instrumentos de patrimonio neto a largo plazo, otras empresas","7963","account.data_account_type_revenue","l10n_es.account_chart_template_assoc","l10n_es.account_group_796","False"

```

## File: data\account.account.template-common.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","group_id/id","reconcile"
"account_common_101","Fondo social","101","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_101","False"
"account_common_102","Capital","102","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_102","False"
"account_common_108","Acciones o participaciones propias en situaciones especiales","108","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_108","False"
"account_common_109","Acciones o participaciones propias para reducción de capital","109","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_109","False"
"account_common_110","Prima de emisión o asunción","110","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_110","False"
"account_common_112","Reserva legal","112","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_112","False"
"account_common_113","Reservas voluntarias","113","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_113","False"
"account_common_1140","Reservas para acciones o participaciones de la sociedad dominante","1140","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1140","False"
"account_common_1141","Reservas estatutarias","1141","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1141","False"
"account_common_1142","Reserva por capital amortizado","1142","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1142","False"
"account_common_1144","Reservas por acciones propias aceptadas en garantía","1144","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1144","False"
"account_common_118","Aportaciones de socios o propietarios","118","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_118","False"
"account_common_119","Diferencias por ajuste del capital a euros","119","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_119","False"
"account_common_131","Donaciones y legados de capital","131","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_131","False"
"account_common_1370","Ingresos fiscales por diferencias permanentes a distribuir en varios ejercicios","1370","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1370","False"
"account_common_1371","Ingresos fiscales por deducciones y bonificaciones a distribuir en varios ejercicios","1371","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1371","False"
"account_common_141","Provisión para impuestos","141","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_141","True"
"account_common_142","Provisión para otras responsabilidades","142","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_142","True"
"account_common_143","Provisión por desmantelamiento, retiro o rehabilitación del inmovilizado","143","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_143","True"
"account_common_145","Provisión para actuaciones medioambientales","145","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_145","True"
"account_common_150","Acciones o participaciones consideradas como pasivos financieros","150","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_150","False"
"account_common_1533","Desembolsos no exigidos, empresas del grupo","1533","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1533","False"
"account_common_1534","Desembolsos no exigidos, empresas asociadas","1534","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1534","False"
"account_common_1535","Desembolsos no exigidos, otras partes vinculadas","1535","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1535","False"
"account_common_1536","Otros desembolsos no exigidos","1536","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1536","False"
"account_common_1543","Aportaciones no dinerarias pendientes, empresas del grupo","1543","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1543","False"
"account_common_1544","Aportaciones no dinerarias pendientes, empresas asociadas","1544","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1544","False"
"account_common_1545","Aportaciones no dinerarias pendientes, otras partes vinculadas","1545","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1545","False"
"account_common_1546","Otras aportaciones no dinerarias pendientes","1546","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1546","False"
"account_common_1603","Deudas con entidades de crédito vinculadas, empresas del grupo","1603","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1603","False"
"account_common_1604","Deudas con entidades de crédito vinculadas, empresas asociadas","1604","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1604","False"
"account_common_1605","Deudas con otras entidades de crédito vinculadas","1605","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1605","False"
"account_common_1613","Proveedores de inmovilizado, empresas del grupo","1613","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1613","False"
"account_common_1614","Proveedores de inmovilizado, empresas asociadas","1614","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1614","False"
"account_common_1615","Proveedores de inmovilizado, otras partes vinculadas","1615","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1615","False"
"account_common_1623","Acreedores por arrendamiento financiero, empresas de grupo","1623","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1623","False"
"account_common_1624","Acreedores por arrendamiento financiero, empresas asociadas","1624","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1624","False"
"account_common_1625","Acreedores por arrendamiento financiero, otras partes vinculadas","1625","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1625","False"
"account_common_1633","Otras deudas, empresas del grupo","1633","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1633","False"
"account_common_1634","Otras deudas, empresas asociadas","1634","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1634","False"
"account_common_1635","Otras deudas, con otras partes vinculadas","1635","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1635","False"
"account_common_170","Deudas con entidades de crédito","170","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_170","False"
"account_common_171","Deudas","171","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_171","False"
"account_common_172","Deudas transformables en subvenciones, donaciones y legados","172","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_172","False"
"account_common_173","Proveedores de inmovilizado","173","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_173","False"
"account_common_174","Acreedores por arrendamiento financiero","174","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_174","False"
"account_common_175","Efectos a pagar","175","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_175","False"
"account_common_176","Pasivos por derivados financieros","176","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_1760","False"
"account_common_177","Obligaciones y bonos","177","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_177","False"
"account_common_179","Deudas representadas en otros valores negociables","179","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_179","False"
"account_common_180","Fianzas recibidas","180","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_180","False"
"account_common_181","Anticipos recibidos por ventas o prestaciones de servicios","181","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_181","False"
"account_common_185","Depósitos recibidos","185","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_185","False"
"account_common_190","Acciones o participaciones emitidas","190","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_190","False"
"account_common_192","Suscriptores de acciones","192","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_192","False"
"account_common_194","Capital emitido pendiente de inscripción","194","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_194","False"
"account_common_195","Acciones o participaciones emitidas consideradas como pasivos financieros","195","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_195","False"
"account_common_197","Suscriptores de acciones consideradas como pasivos financieros","197","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_197","False"
"account_common_199","Acciones o participaciones emitidas consideradas como pasivos financieros pendientes de inscripción","199","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_199","False"
"account_common_200","Investigación","200","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_200","False"
"account_common_201","Desarrollo","201","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_201","False"
"account_common_202","Concesiones administrativas","202","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_202","False"
"account_common_203","Propiedad industrial","203","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_203","False"
"account_common_205","Derechos de traspaso","205","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_205","False"
"account_common_206","Aplicaciones informáticas","206","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_206","False"
"account_common_209","Anticipos para inmovilizaciones intangibles","209","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_209","False"
"account_common_210","Terrenos y bienes naturales","210","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_210","False"
"account_common_211","Construcciones","211","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_211","False"
"account_common_212","Instalaciones técnicas","212","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_212","False"
"account_common_213","Maquinaria","213","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_213","False"
"account_common_214","Utillaje","214","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_214","False"
"account_common_215","Otras instalaciones","215","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_215","False"
"account_common_216","Mobiliario","216","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_216","False"
"account_common_217","Equipos para proceso de información","217","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_217","False"
"account_common_218","Elementos de transporte","218","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_218","False"
"account_common_219","Otro inmovilizado material","219","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_219","False"
"account_common_220","Inversiones en terrenos y bienes naturales","220","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_220","False"
"account_common_221","Inversiones en construcciones","221","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_221","False"
"account_common_230","Adaptación de terrenos y bienes naturales","230","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_230","False"
"account_common_231","Construcciones en curso","231","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_231","False"
"account_common_232","Instalaciones técnicas en montaje","232","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_232","False"
"account_common_233","Maquinaria en montaje","233","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_233","False"
"account_common_237","Equipos para procesos de información en montaje","237","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_237","False"
"account_common_239","Anticipos para inmovilizaciones materiales","239","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_239","False"
"account_common_2413","Valores representativos de deuda de empresas del grupo","2413","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2413","False"
"account_common_2414","Valores representativos de deuda de empresas asociadas","2414","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2414","False"
"account_common_2415","Valores representativos de deuda de otras partes vinculadas","2415","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2415","False"
"account_common_2423","Créditos a empresas del grupo","2423","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2423","False"
"account_common_2424","Créditos a empresas asociadas","2424","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2424","False"
"account_common_2425","Créditos a otras partes vinculadas","2425","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2425","False"
"account_common_250","Inversiones financieras en instrumentos de patrimonio","250","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_250","False"
"account_common_251","Valores representativos de deuda","251","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_251","False"
"account_common_252","Créditos","252","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_252","False"
"account_common_253","Créditos por enajenación de inmovilizado","253","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_253","False"
"account_common_254","Créditos al personal","254","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_254","False"
"account_common_258","Imposiciones","258","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_258","False"
"account_common_259","Desembolsos pendientes sobre participaciones en el patrimonio neto","259","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_259","False"
"account_common_260","Fianzas constituidas","260","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_260","False"
"account_common_265","Depósitos constituidos","265","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_265","False"
"account_common_2800","Amortización acumulada de investigación","2800","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2800","False"
"account_common_2801","Amortización acumulada de desarrollo","2801","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2801","False"
"account_common_2802","Amortización acumulada de concesiones administrativas","2802","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2802","False"
"account_common_2803","Amortización acumulada de propiedad industrial","2803","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2803","False"
"account_common_2805","Amortización acumulada de derechos de traspaso","2805","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2805","False"
"account_common_2806","Amortización acumulada de aplicaciones informáticas","2806","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2806","False"
"account_common_2811","Amortización acumulada de construcciones","2811","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2811","False"
"account_common_2812","Amortización acumulada de instalaciones técnicas","2812","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2812","False"
"account_common_2813","Amortización acumulada de maquinaria","2813","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2813","False"
"account_common_2814","Amortización acumulada de utillaje","2814","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2814","False"
"account_common_2815","Amortización acumulada de otras instalaciones","2815","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2815","False"
"account_common_2816","Amortización acumulada de mobiliario","2816","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2816","False"
"account_common_2817","Amortización acumulada de equipos para proceso de información","2817","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2817","False"
"account_common_2818","Amortización acumulada de elementos de transporte","2818","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2818","False"
"account_common_2819","Amortización acumulada de otro inmovilizado material","2819","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2819","False"
"account_common_282","Amortización acumulada de las inversiones inmobiliarias","282","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_282","False"
"account_common_2900","Deterioro del valor de investigación","2900","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2900","False"
"account_common_2901","Deterioro del valor de desarrollo","2901","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2901","False"
"account_common_2902","Deterioro del valor de concesiones administrativas","2902","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2902","False"
"account_common_2903","Deterioro del valor de propiedad industrial","2903","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2903","False"
"account_common_2905","Deterioro del valor de derechos de traspaso","2905","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2905","False"
"account_common_2906","Deterioro del valor de aplicaciones informáticas","2906","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2906","False"
"account_common_2910","Deterioro de valor de terrenos y bienes naturales","2910","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2910","False"
"account_common_2911","Deterioro de valor de construcciones","2911","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2911","False"
"account_common_2912","Deterioro de valor de instalaciones técnicas","2912","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2912","False"
"account_common_2913","Deterioro de valor de maquinaria","2913","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2913","False"
"account_common_2914","Deterioro de valor de utillaje","2914","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2914","False"
"account_common_2915","Deterioro de valor de otras instalaciones","2915","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2915","False"
"account_common_2916","Deterioro de valor de mobiliario","2916","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2916","False"
"account_common_2917","Deterioro de valor de equipos para proceso de información","2917","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2917","False"
"account_common_2918","Deterioro de valor de elementos de transporte","2918","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2918","False"
"account_common_2919","Deterioro de valor de otro inmovilizado material","2919","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2919","False"
"account_common_2920","Deterioro de valor de los terrenos y bienes naturales","2920","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2920","False"
"account_common_2921","Deterioro de valor de construcciones","2921","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2921","False"
"account_common_2933","Deterioro de valor de participaciones en empresas del grupo","2933","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2933","False"
"account_common_2934","Deterioro de valor de participaciones en empresas asociadas","2934","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2934","False"
"account_common_2943","Deterioro de valor de valores representativos de deuda de empresas del grupo","2943","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2943","False"
"account_common_2944","Deterioro de valor de valores representativos de deuda de empresas asociadas","2944","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2944","False"
"account_common_2945","Deterioro de valor de valores representativos de deuda de otras partes vinculadas","2945","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2945","False"
"account_common_2953","Deterioro de valor de créditos a empresas del grupo","2953","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2953","False"
"account_common_2954","Deterioro de valor de créditos a empresas asociadas","2954","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2954","False"
"account_common_2955","Deterioro de valor de créditos a otras partes vinculadas","2955","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_2955","False"
"account_common_297","Deterioro de valor de valores representativos de deuda","297","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_297","False"
"account_common_298","Deterioro de valor de créditos","298","account.data_account_type_fixed_assets","l10n_es.account_chart_template_common","l10n_es.account_group_298","False"
"account_common_300","Mercaderías A","300","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_300","False"
"account_common_301","Mercaderías B","301","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_301","False"
"account_common_310","Materias primas A","310","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_310","False"
"account_common_311","Materias primas B","311","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_311","False"
"account_common_320","Elementos y conjuntos incorporables","320","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_320","False"
"account_common_321","Combustibles","321","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_321","False"
"account_common_322","Repuestos","322","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_322","False"
"account_common_325","Materiales diversos","325","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_325","False"
"account_common_326","Embalajes","326","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_326","False"
"account_common_327","Envases","327","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_327","False"
"account_common_328","Material de oficina","328","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_328","False"
"account_common_330","Productos en curso A","330","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_330","False"
"account_common_331","Productos en curso B","331","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_331","False"
"account_common_340","Productos semiterminados A","340","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_340","False"
"account_common_341","Productos semiterminados B","341","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_341","False"
"account_common_350","Productos terminados A","350","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_350","False"
"account_common_351","Productos terminados B","351","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_351","False"
"account_common_360","Subproductos A","360","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_360","False"
"account_common_361","Subproductos B","361","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_361","False"
"account_common_365","Residuos A","365","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_365","False"
"account_common_366","Residuos B","366","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_366","False"
"account_common_368","Materiales recuperados A","368","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_368","False"
"account_common_369","Materiales recuperados B","369","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_369","False"
"account_common_390","Deterioro de valor de las mercaderías","390","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_390","False"
"account_common_391","Deterioro de valor de las materias primas","391","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_391","False"
"account_common_392","Deterioro de valor de otros aprovisionamientos","392","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_392","False"
"account_common_393","Deterioro de valor de los productos en curso","393","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_393","False"
"account_common_394","Deterioro de valor de los productos semiterminados","394","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_394","False"
"account_common_395","Deterioro de valor de los productos terminados","395","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_395","False"
"account_common_396","Deterioro de valor de los subproductos, residuos y materiales recuperados","396","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_396","False"
"account_common_4000","Proveedores (euros)","4000","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_4000","True"
"account_common_4004","Proveedores (moneda extranjera)","4004","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_4004","True"
"account_common_4009","Proveedores, facturas pendientes de recibir o de formalizar","4009","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_4009","True"
"account_common_401","Proveedores, efectos comerciales a pagar","401","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_401","True"
"account_common_4030","Proveedores, empresas del grupo (euros)","4030","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_4030","True"
"account_common_4031","Efectos comerciales a pagar, empresas del grupo","4031","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_4031","True"
"account_common_4034","Proveedores, empresas del grupo (moneda extranjera)","4034","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_4034","True"
"account_common_4036","Envases y embalajes a devolver a proveedores, empresas del grupo","4036","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_4036","True"
"account_common_4039","Proveedores, empresas del grupo, facturas pendientes de recibir o de formalizar","4039","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_4039","True"
"account_common_404","Proveedores, empresas asociadas","404","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_404","True"
"account_common_405","Proveedores, otras partes vinculadas","405","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_405","True"
"account_common_406","Envases y embalajes a devolver a proveedores","406","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_406","True"
"account_common_407","Anticipos a proveedores","407","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_407","True"
"account_common_4100","Acreedores por prestaciones de servicios (euros)","4100","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_4100","True"
"account_common_4104","Acreedores por prestaciones de servicios (moneda extranjera)","4104","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_4104","True"
"account_common_4109","Acreedores por prestaciones de servicios, facturas pendientes de recibir o de formalizar","4109","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_4109","True"
"account_common_411","Acreedores, efectos comerciales a pagar","411","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_411","True"
"account_common_419","Acreedores por operaciones en común","419","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_419","True"
"account_common_4300","Clientes (euros)","4300","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4300","True"
"account_common_4301","Clientes (euros) (PoS)","4301","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4300","True"
"account_common_4304","Clientes (moneda extranjera)","4304","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4304","True"
"account_common_4309","Clientes, facturas pendientes de recibir o de formalizar","4309","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4309","True"
"account_common_4310","Efectos comerciales en cartera","4310","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4310","True"
"account_common_4311","Efectos comerciales descontados","4311","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4311","True"
"account_common_4312","Efectos comerciales en gestión de cobro","4312","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4312","True"
"account_common_4315","Efectos comerciales impagados","4315","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4315","True"
"account_common_432","Clientes, operaciones de factoring","432","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_432","True"
"account_common_4330","Clientes empresas del grupo (euros)","4330","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4330","True"
"account_common_4331","Efectos comerciales a cobrar, empresas del grupo","4331","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4331","True"
"account_common_4332","Clientes empresas del grupo, operaciones de factoring","4332","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4332","True"
"account_common_4334","Clientes empresas del grupo (moneda extranjera)","4334","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4334","True"
"account_common_4336","Clientes empresas del grupo de dudoso cobro","4336","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4336","True"
"account_common_4337","Envases y embalajes a devolver a clientes, empresas del grupo","4337","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4337","True"
"account_common_4339","Clientes empresas del grupo, facturas pendientes de formalizar","4339","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4339","True"
"account_common_434","Clientes, empresas asociadas","434","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_434","True"
"account_common_435","Clientes, otras partes vinculadas","435","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_435","True"
"account_common_436","Clientes de dudoso cobro","436","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_436","True"
"account_common_437","Envases y embalajes a devolver por clientes","437","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_437","True"
"account_common_438","Anticipos de clientes","438","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_438","True"
"account_common_4400","Deudores (euros)","4400","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4400","True"
"account_common_4404","Deudores (moneda extranjera)","4404","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4404","True"
"account_common_4409","Deudores, facturas pendientes de formalizar","4409","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4409","True"
"account_common_4410","Deudores, efectos comerciales en cartera","4410","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4410","True"
"account_common_4411","Deudores, efectos comerciales descontados","4411","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4411","True"
"account_common_4412","Deudores, efectos comerciales en gestión de cobro","4412","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4412","True"
"account_common_4415","Deudores, efectos comerciales impagados","4415","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_4415","True"
"account_common_446","Deudores de dudoso cobro","446","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_446","True"
"account_common_449","Deudores por operaciones en común","449","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_449","True"
"account_common_460","Anticipos de remuneraciones","460","account.data_account_type_receivable","l10n_es.account_chart_template_common","l10n_es.account_group_460","True"
"account_common_465","Remuneraciones pendientes de pago","465","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_465","True"
"account_common_4700","Hacienda Pública, deudora por IVA","4700","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_4700","True"
"account_common_4708","Hacienda Pública, deudora por subvenciones concedidas","4708","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_4708","True"
"account_common_4709","Hacienda Pública, deudora por devolución de impuestos","4709","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_4709","True"
"account_common_471","Organismos de la Seguridad Social, deudores","471","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_471","False"
"account_common_472","Hacienda Pública. IVA soportado","472","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_472","False"
"account_common_473","Hacienda Pública, retenciones y pagos a cuenta","473","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_473","True"
"account_common_4740","Activos por diferencias temporarias deducibles","4740","account.data_account_type_non_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_4740","False"
"account_common_4742","Derechos por deducciones y bonificaciones pendientes de aplicar","4742","account.data_account_type_non_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_4742","False"
"account_common_4745","Crédito por pérdidas a compensar del ejercicio","4745","account.data_account_type_non_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_4745","False"
"account_common_4750","Hacienda Pública, acreedora por IVA","4750","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_4750","True"
"account_common_4751","Hacienda Pública, acreedora por retenciones practicadas","4751","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_4751","True"
"account_common_4752","Hacienda Pública, acreedora por impuesto sobre sociedades","4752","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_4752","True"
"account_common_4758","Hacienda Pública, acreedora por subvenciones a reintegrar","4758","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_4758","True"
"account_common_4759","Hacienda Pública, acreedora por otros conceptos","4759","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_4759","False"
"account_common_476","Organismos de la Seguridad Social, acreedores","476","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_476","False"
"account_common_477","Hacienda Pública. IVA repercutido","477","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_477","False"
"account_common_479","Pasivos por diferencias temporarias imponibles","479","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_479","False"
"account_common_480","Gastos anticipados","480","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_480","False"
"account_common_485","Ingresos anticipados","485","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_485","False"
"account_common_4933","Deterioro de valor de créditos por operaciones comerciales con empresas del grupo","4933","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_4933","True"
"account_common_4934","Deterioro de valor de créditos por operaciones comerciales con empresas asociadas","4934","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_4934","True"
"account_common_4935","Deterioro de valor de créditos por operaciones comerciales con otras partes vinculadas","4935","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_4935","True"
"account_common_4994","Provisión por contratos onerosos","4994","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_4994","True"
"account_common_4999","Provisión para otras operaciones comerciales","4999","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_4999","True"
"account_common_500","Obligaciones y bonos a corto plazo","500","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_500","False"
"account_common_502","Acciones o participaciones a corto plazo consideradas como pasivos financieros","502","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_502","False"
"account_common_505","Deudas representadas en otros valores negociables a corto plazo","505","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_505","False"
"account_common_506","Intereses a corto plazo de empréstitos y otras emisiones análogas","506","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_506","False"
"account_common_507","Dividendos de acciones o participaciones consideradas como pasivos financieros","507","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_507","False"
"account_common_5090","Obligaciones y bonos amortizados","5090","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5090","True"
"account_common_5095","Otros valores negociables amortizados","5095","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5095","True"
"account_common_5103","Deudas a corto plazo con entidades de crédito, empresas del grupo","5103","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5103","False"
"account_common_5104","Deudas a corto plazo con entidades de crédito, empresas asociadas","5104","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5104","False"
"account_common_5105","Deudas a corto plazo con otras entidades de crédito vinculadas","5105","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5105","False"
"account_common_5113","Proveedores de inmovilizado a corto plazo, empresas del grupo","5113","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5113","False"
"account_common_5114","Proveedores de inmovilizado a corto plazo, empresas asociadas","5114","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5114","False"
"account_common_5115","Proveedores de inmovilizado a corto plazo, otras partes vinculadas","5115","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5115","False"
"account_common_5123","Acreedores por arrendamiento financiero a corto plazo, empresas del grupo","5123","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5123","False"
"account_common_5124","Acreedores por arrendamiento financiero a corto plazo, empresas asociadas","5124","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5124","False"
"account_common_5125","Acreedores por arrendamiento financiero a corto plazo, otras partes vinculadas","5125","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5125","False"
"account_common_5133","Otras deudas a corto plazo con empresas del grupo","5133","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5133","False"
"account_common_5134","Otras deudas a corto plazo con empresas asociadas","5134","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5134","False"
"account_common_5135","Otras deudas a corto plazo con otras partes vinculadas","5135","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5135","False"
"account_common_5143","Intereses a corto plazo de deudas con empresas del grupo","5143","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5143","False"
"account_common_5144","Intereses a corto plazo de deudas con empresas asociadas","5144","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5144","False"
"account_common_5145","Intereses a corto plazo de deudas con otras partes vinculadas","5145","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5145","False"
"account_common_5200","Préstamos a corto plazo de entidades de crédito","5200","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5200","True"
"account_common_5201","Deudas a corto plazo por crédito dispuesto","5201","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5201","True"
"account_common_5208","Deudas por efectos descontados","5208","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5208","True"
"account_common_5209","Deudas por operaciones de “factoring”","5209","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5209","True"
"account_common_521","Deudas a corto plazo","521","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_521","False"
"account_common_522","Deudas a corto plazo transformables en subvenciones, donaciones y legados","522","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_522","False"
"account_common_523","Proveedores de inmovilizado a corto plazo","523","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_523","True"
"account_common_524","Acreedores por arrendamiento financiero a corto plazo","524","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_524","False"
"account_common_525","Efectos a pagar a corto plazo","525","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_525","False"
"account_common_526","Dividendo activo a pagar","526","account.data_account_type_payable","l10n_es.account_chart_template_common","l10n_es.account_group_526","True"
"account_common_527","Intereses a corto plazo de deudas con entidades de crédito","527","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_527","False"
"account_common_528","Intereses a corto plazo de deudas","528","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_528","False"
"account_common_5290","Provisión a corto plazo por retribuciones al personal","5290","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5290","True"
"account_common_5291","Provisión a corto plazo para impuestos","5291","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5291","True"
"account_common_5292","Provisión a corto plazo para otras responsabilidades","5292","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5292","True"
"account_common_5293","Provisión a corto plazo por desmantelamiento, retiro o rehabilitación del inmovilizado","5293","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5293","True"
"account_common_5295","Provisión a corto plazo para actuaciones medioambientales","5295","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5295","True"
"account_common_5303","Participaciones a corto plazo en empresas del grupo","5303","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5303","False"
"account_common_5304","Participaciones a corto plazo en empresas asociadas","5304","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5304","False"
"account_common_5305","Participaciones a corto plazo en otras partes vinculadas","5305","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5305","False"
"account_common_5313","Valores representativos de deuda a corto plazo de empresas del grupo","5313","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5313","False"
"account_common_5314","Valores representativos de deuda a corto plazo de empresas asociadas","5314","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5314","False"
"account_common_5315","Valores representativos de deuda a corto plazo de otras partes vinculadas","5315","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5315","False"
"account_common_5323","Créditos a corto plazo a empresas del grupo","5323","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5323","False"
"account_common_5324","Créditos a corto plazo a empresas asociadas","5324","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5324","False"
"account_common_5325","Créditos a corto plazo a otras partes vinculadas","5325","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5325","False"
"account_common_5333","Intereses a corto plazo de valores representativos de deuda de empresas del grupo","5333","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5333","False"
"account_common_5334","Intereses a corto plazo de valores representativos de deuda de empresas asociadas","5334","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5334","False"
"account_common_5335","Intereses a corto plazo de valores representativos de deuda de otras partes vinculadas","5335","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5335","False"
"account_common_5343","Intereses a corto plazo de créditos a empresas del grupo","5343","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5343","False"
"account_common_5344","Intereses a corto plazo de créditos a empresas asociadas","5344","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5344","False"
"account_common_5345","Intereses a corto plazo de créditos a otras partes vinculadas","5345","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5345","False"
"account_common_5353","Dividendo a cobrar de inversiones financieras en empresas del grupo","5353","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5353","False"
"account_common_5354","Dividendo a cobrar de inversiones financieras en empresas asociadas","5354","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5354","False"
"account_common_5355","Dividendo a cobrar de inversiones financieras en otras partes vinculadas","5355","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5355","False"
"account_common_5393","Desembolsos pendientes sobre participaciones a corto plazo en empresas del grupo","5393","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5393","False"
"account_common_5394","Desembolsos pendientes sobre participaciones a corto plazo en empresas asociadas","5394","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5394","False"
"account_common_5395","Desembolsos pendientes sobre participaciones a corto plazo en otras partes vinculadas","5395","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5395","False"
"account_common_540","Inversiones financieras a corto plazo en instrumentos de patrimonio","540","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_540","False"
"account_common_541","Valores representativos de deuda a corto plazo","541","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_541","False"
"account_common_542","Créditos a corto plazo","542","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_542","False"
"account_common_543","Créditos a corto plazo por enajenación de inmovilizado","543","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_543","False"
"account_common_544","Créditos a corto plazo al personal","544","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_544","False"
"account_common_545","Dividendo a cobrar","545","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_545","False"
"account_common_546","Intereses a corto plazo de valores representativos de deudas","546","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_546","False"
"account_common_547","Intereses a corto plazo de créditos","547","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_547","False"
"account_common_548","Imposiciones a corto plazo","548","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_548","False"
"account_common_549","Desembolsos pendientes sobre participaciones en el patrimonio neto a corto plazo","549","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_549","False"
"account_common_550","Titular de la explotación","550","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_550","False"
"account_common_5523","Cuenta corriente con empresas del grupo","5523","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5523","False"
"account_common_5524","Cuenta corriente con empresas asociadas","5524","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5524","False"
"account_common_5525","Cuenta corriente con otras partes vinculadas","5525","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5525","False"
"account_common_554","Cuenta corriente con uniones temporales de empresas y comunidades de bienes","554","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_554","False"
"account_common_555","Partidas pendientes de aplicación","555","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_555","False"
"account_common_5563","Desembolsos exigidos sobre participaciones, empresas del grupo","5563","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5563","False"
"account_common_5564","Desembolsos exigidos sobre participaciones, empresas asociadas","5564","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5564","False"
"account_common_5565","Desembolsos exigidos sobre participaciones, otras partes vinculadas","5565","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5565","False"
"account_common_5566","Desembolsos exigidos sobre participaciones de otras empresas","5566","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5566","False"
"account_common_557","Dividendo activo a cuenta","557","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_557","False"
"account_common_5580","Socios por desembolsos exigidos sobre acciones o participaciones ordinarias","5580","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5580","False"
"account_common_5585","Socios por desembolsos exigidos sobre acciones o participaciones consideradas como pasivos financieros","5585","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5585","False"
"account_common_5590","Activos por derivados financieros a corto plazo, cartera de negociación","5590","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5590","False"
"account_common_5595","Pasivos por derivados financieros a corto plazo, cartera de negociación","5595","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_5595","False"
"account_common_560","Fianzas recibidas a corto plazo","560","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_560","False"
"account_common_561","Depósitos recibidos a corto plazo","561","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_561","False"
"account_common_565","Fianzas constituidas a corto plazo","565","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_565","False"
"account_common_566","Depósitos constituidos a corto plazo","566","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_566","False"
"account_common_567","Intereses pagados por anticipado","567","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_567","False"
"account_common_568","Intereses cobrados por anticipado","568","account.data_account_type_current_liabilities","l10n_es.account_chart_template_common","l10n_es.account_group_568","False"
"account_common_570","Caja, euros","570","account.data_account_type_liquidity","l10n_es.account_chart_template_common","l10n_es.account_group_570","False"
"account_common_571","Caja, moneda extranjera","571","account.data_account_type_liquidity","l10n_es.account_chart_template_common","l10n_es.account_group_571","False"
"pgc_572_child","Bancos e instituciones de crédito c/c vista, euros","572","account.data_account_type_liquidity","l10n_es.account_chart_template_common","l10n_es.account_group_572","False"
"account_common_573","Bancos e instituciones de crédito c/c vista, moneda extranjera","573","account.data_account_type_liquidity","l10n_es.account_chart_template_common","l10n_es.account_group_573","False"
"account_common_574","Bancos e instituciones de crédito, cuentas de ahorro, euros","574","account.data_account_type_liquidity","l10n_es.account_chart_template_common","l10n_es.account_group_574","False"
"account_common_575","Bancos e instituciones de crédito, cuentas de ahorro, moneda extranjera","575","account.data_account_type_liquidity","l10n_es.account_chart_template_common","l10n_es.account_group_575","False"
"account_common_576","Inversiones a corto plazo de gran liquidez","576","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_576","False"
"account_common_5933","Deterioro de valor de participaciones a corto plazo en empresas del grupo","5933","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5933","False"
"account_common_5934","Deterioro de valor de participaciones a corto plazo en empresas asociadas","5934","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5934","False"
"account_common_5943","Deterioro de valor de valores representativos de deuda a corto plazo de empresas del grupo","5943","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5943","False"
"account_common_5944","Deterioro de valor de valores representativos de deuda a corto plazo de empresas asociadas","5944","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5944","False"
"account_common_5945","Deterioro de valor de valores representativos de deuda a corto plazo de otras partes vinculadas","5945","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5945","False"
"account_common_5953","Deterioro de valor de créditos a corto plazo a empresas del grupo","5953","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5953","False"
"account_common_5954","Deterioro de valor de créditos a corto plazo a empresas asociadas","5954","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5954","False"
"account_common_5955","Deterioro de valor de créditos a corto plazo a otras partes vinculadas","5955","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_5955","False"
"account_common_597","Deterioro de valor de valores representativos de deuda a corto plazo","597","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_597","False"
"account_common_598","Deterioro de valor de créditos a corto plazo","598","account.data_account_type_current_assets","l10n_es.account_chart_template_common","l10n_es.account_group_598","False"
"account_common_600","Compras de mercaderías","600","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_600","False"
"account_common_601","Compras de materias primas","601","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_601","False"
"account_common_602","Compras de otros aprovisionamientos","602","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_602","False"
"account_common_6060","Descuentos sobre compras por pronto pago de mercaderías","6060","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6060","False"
"account_common_6061","Descuentos sobre compras por pronto pago de materias primas","6061","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6061","False"
"account_common_6062","Descuentos sobre compras por pronto pago de otros aprovisionamientos","6062","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6062","False"
"account_common_607","Trabajos realizados por otras empresas","607","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_607","False"
"account_common_6080","Devoluciones de compras de mercaderías","6080","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6080","False"
"account_common_6081","Devoluciones de compras de materias primas","6081","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6081","False"
"account_common_6082","Devoluciones de compras de otros aprovisionamientos","6082","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6082","False"
"account_common_6090","""Rappels"" por compras de mercaderías","6090","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6090","False"
"account_common_6091","""Rappels"" por compras de materias primas","6091","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6091","False"
"account_common_6092","""Rappels"" por compras de otros aprovisionamientos","6092","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6092","False"
"account_common_610","Variación de existencias de mercaderías","610","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_610","False"
"account_common_611","Variación de existencias de materias primas","611","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_611","False"
"account_common_612","Variación de existencias de otros aprovisionamientos","612","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_612","False"
"account_common_620","Gastos en investigación y desarrollo del ejercicio","620","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_620","False"
"account_common_621","Arrendamientos y cánones","621","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_621","False"
"account_common_622","Reparaciones y conservación","622","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_622","False"
"account_common_623","Servicios de profesionales independientes","623","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_623","False"
"account_common_624","Transportes","624","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_624","False"
"account_common_625","Primas de seguros","625","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_625","False"
"account_common_626","Servicios bancarios y similares","626","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_626","False"
"account_common_627","Publicidad, propaganda y relaciones públicas","627","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_627","False"
"account_common_628","Suministros","628","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_628","False"
"account_common_629","Otros servicios","629","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_629","False"
"account_common_6300","Impuesto corriente","6300","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6300","False"
"account_common_6301","Impuesto diferido","6301","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6301","False"
"account_common_631","Otros tributos","631","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_631","False"
"account_common_633","Ajustes negativos en la imposición sobre beneficios","633","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_633","False"
"account_common_6341","Ajustes negativos en IVA de activo corriente","6341","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6341","False"
"account_common_6342","Ajustes negativos en IVA de inversiones","6342","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6342","False"
"account_common_636","Devolución de impuestos","636","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_636","False"
"account_common_638","Ajustes positivos en la imposición sobre beneficios","638","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_638","False"
"account_common_6391","Ajustes positivos en IVA de activo corriente","6391","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6391","False"
"account_common_6392","Ajustes positivos en IVA de inversiones","6392","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6392","False"
"account_common_640","Sueldos y salarios","640","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_640","False"
"account_common_641","Indemnizaciones","641","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_641","False"
"account_common_642","Seguridad Social a cargo de la empresa","642","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_642","False"
"account_common_649","Otros gastos sociales","649","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_649","False"
"account_common_660","Gastos financieros por actualización de provisiones","660","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_660","False"
"account_common_6610","Intereses de obligaciones y bonos, empresas del grupo","6610","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6610","False"
"account_common_6611","Intereses de obligaciones y bonos, empresas asociadas","6611","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6611","False"
"account_common_6612","Intereses de obligaciones y bonos, otras partes vinculadas","6612","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6612","False"
"account_common_6613","Intereses de obligaciones y bonos, otras empresas","6613","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6613","False"
"account_common_6615","Intereses de obligaciones y bonos a corto plazo, empresas del grupo","6615","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6615","False"
"account_common_6616","Intereses de obligaciones y bonos a corto plazo, empresas asociadas","6616","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6616","False"
"account_common_6617","Intereses de obligaciones y bonos a corto plazo, otras partes vinculadas","6617","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6617","False"
"account_common_6618","Intereses de obligaciones y bonos a corto plazo, otras empresas","6618","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6618","False"
"account_common_6620","Intereses de deudas, empresas del grupo","6620","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6620","False"
"account_common_6621","Intereses de deudas, empresas asociadas","6621","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6621","False"
"account_common_6622","Intereses de deudas, otras partes vinculadas","6622","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6622","False"
"account_common_6623","Intereses de deudas con entidades de crédito","6623","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6623","False"
"account_common_6624","Intereses de deudas, otras empresas","6624","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6624","False"
"account_common_6640","Dividendos de pasivos, empresas del grupo","6640","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6640","False"
"account_common_6641","Dividendos de pasivos, empresas asociadas","6641","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6641","False"
"account_common_6642","Dividendos de pasivos, otras partes vinculadas","6642","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6642","False"
"account_common_6643","Dividendos de pasivos, otras empresas","6643","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6643","False"
"account_common_6650","Intereses por descuento de efectos en entidades de crédito del grupo","6650","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6650","False"
"account_common_6651","Intereses por descuento de efectos en entidades de crédito asociadas","6651","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6651","False"
"account_common_6652","Intereses por descuento de efectos en entidades de crédito vinculadas","6652","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6652","False"
"account_common_6653","Intereses por descuento de efectos en otras entidades de crédito","6653","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6653","False"
"account_common_6654","Intereses por operaciones de ""factoring"" con entidades de crédito del grupo","6654","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6654","False"
"account_common_6655","Intereses por operaciones de ""factoring"" con entidades de crédito asociadas","6655","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6655","False"
"account_common_6656","Intereses por operaciones de ""factoring"" con entidades de crédito vinculadas","6656","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6656","False"
"account_common_6657","Intereses por operaciones de ""factoring"" con otras entidades de crédito","6657","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6657","False"
"account_common_6660","Pérdidas en valores representativos de deuda, empresas del grupo","6660","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6660","False"
"account_common_6661","Pérdidas en valores representativos de deuda, empresas asociadas","6661","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6661","False"
"account_common_6662","Pérdidas en valores representativos de deuda, otras partes vinculadas","6662","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6662","False"
"account_common_6663","Pérdidas en valores representativos de deuda, otras empresas","6663","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6663","False"
"account_common_6665","Pérdidas en valores representativos de deuda a corto plazo, empresas del grupo","6665","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6665","False"
"account_common_6666","Pérdidas en valores representativos de deuda a corto plazo, empresas asociadas","6666","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6666","False"
"account_common_6667","Pérdidas en valores representativos de deuda a corto plazo, otras partes vinculadas","6667","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6667","False"
"account_common_6668","Pérdidas en valores representativos de deuda a corto plazo, otras empresas","6668","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6668","False"
"account_common_6670","Pérdidas de créditos, empresas del grupo","6670","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6670","False"
"account_common_6671","Pérdidas de créditos, empresas asociadas","6671","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6671","False"
"account_common_6672","Pérdidas de créditos, otras partes vinculadas","6672","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6672","False"
"account_common_6673","Pérdidas de créditos, otras empresas","6673","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6673","False"
"account_common_6675","Pérdidas de créditos a corto plazo, empresas del grupo","6675","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6675","False"
"account_common_6676","Pérdidas de créditos a corto plazo, empresas asociadas","6676","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6676","False"
"account_common_6677","Pérdidas de créditos a corto plazo, otras partes vinculadas","6677","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6677","False"
"account_common_6678","Pérdidas de créditos a corto plazo, otras empresas","6678","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6678","False"
"account_common_668","Diferencias negativas de cambio","668","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_668","False"
"account_common_669","Otros gastos financieros","669","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_669","False"
"account_common_670","Pérdidas procedentes del inmovilizado intangible","670","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_670","False"
"account_common_672","Pérdidas procedentes de las inversiones inmobiliarias","672","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_672","False"
"account_common_6733","Pérdidas procedentes de participaciones en, empresas del grupo","6733","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6733","False"
"account_common_6734","Pérdidas procedentes de participaciones, empresas asociadas","6734","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6734","False"
"account_common_6735","Pérdidas procedentes de participaciones, otras partes vinculadas","6735","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6735","False"
"account_common_675","Pérdidas por operaciones con obligaciones propias","675","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_675","False"
"account_common_678","Gastos excepcionales","678","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_678","False"
"account_common_680","Amortización del inmovilizado intangible","680","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_680","False"
"account_common_681","Amortización del inmovilizado material","681","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_681","False"
"account_common_682","Amortización de las inversiones inmobiliarias","682","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_682","False"
"account_common_690","Pérdidas por deterioro del inmovilizado intangible","690","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_690","False"
"account_common_692","Pérdidas por deterioro de las inversiones inmobiliarias","692","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_692","False"
"account_common_6930","Pérdidas por deterioro de productos terminados y en curso de fabricación","6930","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6930","False"
"account_common_6931","Pérdidas por deterioro de mercaderías","6931","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6931","False"
"account_common_6932","Pérdidas por deterioro de materias primas","6932","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6932","False"
"account_common_6933","Pérdidas por deterioro de otros aprovisionamientos","6933","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6933","False"
"account_common_6954","Dotación a la provisión por contratos onerosos","6954","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6954","False"
"account_common_6959","Dotación a la provisión para otras operaciones comerciales","6959","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6959","False"
"account_common_6960","Pérdidas por deterioro de participaciones en instrumentos de patrimonio neto, empresas del grupo","6960","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6960","False"
"account_common_6961","Pérdidas por deterioro de participaciones en instrumentos de patrimonio neto, empresas asociadas","6961","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6961","False"
"account_common_6962","Pérdidas por deterioro de participaciones en instrumentos de patrimonio neto, otras partes vinculadas","6962","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6962","False"
"account_common_6963","Pérdidas por deterioro de participaciones en instrumentos de patrimonio neto, otras empresas","6963","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6963","False"
"account_common_6965","Pérdidas por deterioro en valores representativos de deuda, empresas del grupo","6965","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6965","False"
"account_common_6966","Pérdidas por deterioro en valores representativos de deuda, empresas asociadas","6966","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6966","False"
"account_common_6967","Pérdidas por deterioro en valores representativos de deuda, otras partes vinculadas","6967","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6967","False"
"account_common_6968","Pérdidas por deterioro en valores representativos de deuda, otras empresas","6968","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6968","False"
"account_common_6970","Pérdidas por deterioro de créditos, empresas del grupo","6970","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6970","False"
"account_common_6971","Pérdidas por deterioro de créditos, empresas asociadas","6971","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6971","False"
"account_common_6972","Pérdidas por deterioro de créditos, otras partes vinculadas","6972","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6972","False"
"account_common_6973","Pérdidas por deterioro de créditos, otras empresas","6973","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6973","False"
"account_common_6980","Pérdidas por deterioro de participaciones en instrumentos de patrimonio neto a corto plazo, empresas del grupo","6980","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6980","False"
"account_common_6981","Pérdidas por deterioro de participaciones en instrumentos de patrimonio neto a corto plazo, empresas asociadas","6981","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6981","False"
"account_common_6985","Pérdidas por deterioro en valores representativos de deuda a corto plazo, empresas del grupo","6985","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6985","False"
"account_common_6986","Pérdidas por deterioro en valores representativos de deuda a corto plazo, empresas asociadas","6986","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6986","False"
"account_common_6987","Pérdidas por deterioro en valores representativos de deuda a corto plazo, otras partes vinculadas","6987","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6987","False"
"account_common_6988","Pérdidas por deterioro en valores representativos de deuda a corto plazo, otras empresas","6988","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6988","False"
"account_common_6990","Pérdidas por deterioro de créditos a corto plazo, empresas del grupo","6990","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6990","False"
"account_common_6991","Pérdidas por deterioro de créditos a corto plazo, empresas asociadas","6991","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6991","False"
"account_common_6992","Pérdidas por deterioro de créditos a corto plazo, otras partes vinculadas","6992","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6992","False"
"account_common_6993","Pérdidas por deterioro de créditos a corto plazo, otras empresas","6993","account.data_account_type_expenses","l10n_es.account_chart_template_common","l10n_es.account_group_6993","False"
"account_common_7000","Ventas de mercaderías en España","7000","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_700","False"
"account_common_7001","Ventas de mercaderías Intracomunitarias","7001","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_700","False"
"account_common_7002","Ventas de mercaderías Exportación","7002","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_700","False"
"account_common_7010","Ventas de productos terminados en España","7010","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_701","False"
"account_common_7011","Ventas de productos terminados Intracomunitarias","7011","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_701","False"
"account_common_7012","Ventas de productos terminados Exportación","7012","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_701","False"
"account_common_7020","Ventas de productos semiterminados en España","7020","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_702","False"
"account_common_7021","Ventas de productos semiterminados Intracomunitarias","7021","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_702","False"
"account_common_7022","Ventas de productos semiterminados Exportación","7022","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_702","False"
"account_common_7030","Ventas de subproductos y residuos en España","7030","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_703","False"
"account_common_7031","Ventas de subproductos y residuos Intracomunitarias","7031","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_703","False"
"account_common_7032","Ventas de subproductos y residuos Exportación","7032","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_703","False"
"account_common_7040","Ventas de envases y embalajes en España","7040","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_704","False"
"account_common_7041","Ventas de envases y embalajes Intracomunitarias","7041","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_704","False"
"account_common_7042","Ventas de envases y embalajes Exportación","7042","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_704","False"
"account_common_7050","Prestaciones de servicios en España","7050","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_705","False"
"account_common_7051","Prestaciones de servicios Intracomunitarias","7051","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_705","False"
"account_common_7052","Prestaciones de servicios fuera de la UE","7052","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_705","False"
"account_common_7060","Descuentos sobre ventas por pronto pago de mercaderías","7060","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7060","False"
"account_common_7061","Descuentos sobre ventas por pronto pago de productos terminados","7061","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7061","False"
"account_common_7062","Descuentos sobre ventas por pronto pago de productos semiterminados","7062","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7062","False"
"account_common_7063","Descuentos sobre ventas por pronto pago de subproductos y residuos","7063","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7063","False"
"account_common_7080","Devoluciones de ventas de mercaderías","7080","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7080","False"
"account_common_7081","Devoluciones de ventas de productos terminados","7081","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7081","False"
"account_common_7082","Devoluciones de ventas de productos semiterminados","7082","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7082","False"
"account_common_7083","Devoluciones de ventas de subproductos y residuos","7083","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7083","False"
"account_common_7084","Devoluciones de ventas de envases y embalajes","7084","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7084","False"
"account_common_7090","""Rappels"" sobre ventas de mercaderías","7090","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7090","False"
"account_common_7091","""Rappels"" sobre ventas de productos terminados","7091","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7091","False"
"account_common_7092","""Rappels"" sobre ventas de productos semiterminados","7092","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7092","False"
"account_common_7093","""Rappels"" sobre ventas de subproductos y residuos","7093","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7093","False"
"account_common_7094","""Rappels"" sobre ventas de envases y embalajes","7094","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7094","False"
"account_common_710","Variación de existencias de productos en curso","710","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_710","False"
"account_common_711","Variación de existencias de productos semiterminados","711","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_711","False"
"account_common_712","Variación de existencias de productos terminados","712","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_712","False"
"account_common_713","Variación de existencias de subproductos, residuos y materiales recuperados","713","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_713","False"
"account_common_730","Trabajos realizados para el inmovilizado intangible","730","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_730","False"
"account_common_731","Trabajos realizados para el inmovilizado material","731","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_731","False"
"account_common_732","Trabajos realizados en inversiones inmobiliarias","732","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_732","False"
"account_common_733","Trabajos realizados para el inmovilizado material en curso","733","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_733","False"
"account_common_740","Subvenciones, donaciones y legados a la explotación","740","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_740","False"
"account_common_746","Subvenciones, donaciones y legados de capital transferidos al resultado del ejercicio de carácter no financiero","746","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_746","False"
"account_common_7461","Subvenciones, donaciones y legados de capital transferidos al resultado del ejercicio de carácter financiero","7461","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_746","False"
"account_common_747","Otras subvenciones, donaciones y legados transferidos al resultado del ejercicio","747","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_747","False"
"account_common_7510","Pérdida transferido (gestor)","7510","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7510","False"
"account_common_7511","Beneficio atribuido (partícipe o asociado no gestor)","7511","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7511","False"
"account_common_752","Ingresos por arrendamientos","752","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_752","False"
"account_common_753","Ingresos de propiedad industrial cedida en explotación","753","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_753","False"
"account_common_754","Ingresos por comisiones","754","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_754","False"
"account_common_755","Ingresos por servicios al personal","755","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_755","False"
"account_common_759","Ingresos por servicios diversos","759","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_759","False"
"account_common_7600","Ingresos de participaciones en instrumentos de patrimonio, empresas del grupo","7600","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7600","False"
"account_common_7601","Ingresos de participaciones en instrumentos de patrimonio, empresas asociadas","7601","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7601","False"
"account_common_7602","Ingresos de participaciones en instrumentos de patrimonio, otras partes vinculadas","7602","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7602","False"
"account_common_7603","Ingresos de participaciones en instrumentos de patrimonio, otras empresas","7603","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7603","False"
"account_common_7610","Ingresos de valores representativos de deuda, empresas del grupo","7610","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7610","False"
"account_common_7611","Ingresos de valores representativos de deuda, empresas asociadas","7611","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7611","False"
"account_common_7612","Ingresos de valores representativos de deuda, otras partes vinculadas","7612","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7612","False"
"account_common_7613","Ingresos de valores representativos de deuda, otras empresas","7613","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7613","False"
"account_common_76200","Ingresos de créditos a largo plazo, empresas del grupo","76200","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_76200","False"
"account_common_76201","Ingresos de créditos a largo plazo, empresas asociadas","76201","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_76201","False"
"account_common_76202","Ingresos de créditos a largo plazo, otras partes vinculadas","76202","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_76202","False"
"account_common_76203","Ingresos de créditos a largo plazo, otras empresas","76203","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_76203","False"
"account_common_76210","Ingresos de créditos a corto plazo, empresas del grupo","76210","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_76210","False"
"account_common_76211","Ingresos de créditos a corto plazo, empresas asociadas","76211","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_76211","False"
"account_common_76212","Ingresos de créditos a corto plazo, otras partes vinculadas","76212","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_76212","False"
"account_common_76213","Ingresos de créditos a corto plazo, otras empresas","76213","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_76213","False"
"account_common_7660","Beneficios en valores representativos de deuda a largo plazo, empresas del grupo","7660","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7660","False"
"account_common_7661","Beneficios en valores representativos de deuda a largo plazo, empresas asociadas","7661","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7661","False"
"account_common_7662","Beneficios en valores representativos de deuda a largo plazo, otras partes vinculadas","7662","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7662","False"
"account_common_7663","Beneficios en valores representativos de deuda a largo plazo, otras empresas","7663","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7663","False"
"account_common_7665","Beneficios en valores representativos de deuda a corto plazo, empresas del grupo","7665","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7665","False"
"account_common_7666","Beneficios en valores representativos de deuda a corto plazo, empresas asociadas","7666","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7666","False"
"account_common_7667","Beneficios en valores representativos de deuda a corto plazo, otras partes vinculadas","7667","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7667","False"
"account_common_7668","Beneficios en valores representativos de deuda a corto plazo, otras empresas","7668","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7668","False"
"account_common_768","Diferencias positivas de cambio","768","account.data_account_type_other_income","l10n_es.account_chart_template_common","l10n_es.account_group_768","False"
"account_common_769","Otros ingresos financieros","769","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_769","False"
"account_common_770","Beneficios procedentes del inmovilizado intangible","770","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_770","False"
"account_common_771","Beneficios procedentes del inmovilizado material","771","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_771","False"
"account_common_772","Beneficios procedentes de las inversiones inmobiliarias","772","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_772","False"
"account_common_7733","Beneficios procedentes de participaciones a largo plazo, empresas del grupo","7733","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7733","False"
"account_common_7734","Beneficios procedentes de participaciones a largo plazo, empresas asociadas","7734","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7734","False"
"account_common_7735","Beneficios procedentes de participaciones a largo plazo, otras partes vinculadas","7735","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7735","False"
"account_common_775","Beneficos por operaciones con obligaciones propias","775","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_775","False"
"account_common_778","Ingresos excepcionales","778","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_778","False"
"account_common_790","Reversión del deterioro del inmovilizado intangible","790","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_790","False"
"account_common_792","Reversión del deterioro de las inversiones inmobiliarias","792","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_792","False"
"account_common_7930","Reversión del deterioro de productos terminados y en curso de fabricación","7930","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7930","False"
"account_common_7931","Reversión del deterioro de mercaderías","7931","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7931","False"
"account_common_7932","Reversión del deterioro de materias primas","7932","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7932","False"
"account_common_7933","Reversión del deterioro de otros aprovisionamientos","7933","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7933","False"
"account_common_7951","Exceso de provisión para impuestos","7951","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7951","False"
"account_common_7952","Exceso de provisión para otras responsabilidades","7952","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7952","False"
"account_common_79544","Exceso de provisión por contratos onerosos","79544","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_79544","False"
"account_common_79549","Exceso de provisión para otras operaciones comerciales","79549","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_79549","False"
"account_common_7955","Exceso de provisión para actuaciones medioambientales","7955","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7955","False"
"account_common_7960","Reversión del deterioro de participaciones en instrumentos de patrimonio neto a largo plazo, empresas del grupo","7960","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7960","False"
"account_common_7961","Reversión del deterioro de participaciones en instrumentos de patrimonio neto a largo plazo, empresas asociadas","7961","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7961","False"
"account_common_7965","Reversión del deterioro de valores representativos de deuda a largo plazo, empresas del grupo","7965","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7965","False"
"account_common_7966","Reversión del deterioro de valores representativos de deuda a largo plazo, empresas asociadas","7966","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7966","False"
"account_common_7967","Reversión del deterioro de valores representativos de deuda a largo plazo, otras partes vinculadas","7967","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7967","False"
"account_common_7968","Reversión del deterioro de valores representativos de deuda a largo plazo, otras empresas","7968","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7968","False"
"account_common_7970","Reversión del deterioro de créditos a largo plazo, empresas del grupo","7970","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7970","False"
"account_common_7971","Reversión del deterioro de créditos a largo plazo, empresas asociadas","7971","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7971","False"
"account_common_7972","Reversión del deterioro de créditos a largo plazo, otras partes vinculadas","7972","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7972","False"
"account_common_7973","Reversión del deterioro de créditos a largo plazo, otras empresas","7973","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7973","False"
"account_common_7980","Reversión del deterioro de participaciones en instrumentos de patrimonio neto a corto plazo, empresas del grupo","7980","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7980","False"
"account_common_7981","Reversión del deterioro de participaciones en instrumentos de patrimonio neto a corto plazo, empresas asociadas","7981","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7981","False"
"account_common_7985","Reversión del deterioro de valores representativos de deuda a corto plazo, empresas del grupo","7985","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7985","False"
"account_common_7986","Reversión del deterioro de valores representativos de deuda a corto plazo, empresas asociadas","7986","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7986","False"
"account_common_7987","Reversión del deterioro de valores representativos de deuda a corto plazo, otras partes vinculadas","7987","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7987","False"
"account_common_7988","Reversión del deterioro de valores representativos de deuda a corto plazo, otras empresas","7988","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7988","False"
"account_common_7990","Reversión del deterioro de créditos a corto plazo, empresas del grupo","7990","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7990","False"
"account_common_7991","Reversión del deterioro de créditos a corto plazo, empresas asociadas","7991","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7991","False"
"account_common_7992","Reversión del deterioro de créditos a corto plazo, otras partes vinculadas","7992","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7992","False"
"account_common_7993","Reversión del deterioro de créditos a corto plazo, otras empresas","7993","account.data_account_type_revenue","l10n_es.account_chart_template_common","l10n_es.account_group_7993","False"

```

## File: data\account.account.template-full.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","group_id/id","reconcile"
"account_full_100","Capital social","100","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_100","False"
"account_full_1030","Socios por desembolsos no exigidos, capital social","1030","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_1030","False"
"account_full_1034","Socios por desembolsos no exigidos, capital pendiente de inscripción","1034","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_1034","False"
"account_full_1040","Socios por aportaciones no dinerarias pendientes, capital social","1040","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_1040","False"
"account_full_1044","Socios por aportaciones no dinerarias pendientes, capital pendiente de inscripción","1044","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_1044","False"
"account_full_1110","Patrimonio neto por emision de instrumentos financieros compuestos","1110","account.data_account_type_equity","l10n_es.account_chart_template_full","l10n_es.account_group_1110","False"
"account_full_1111","Resto de instrumentos de patrimonio neto","111100","account.data_account_type_equity","l10n_es.account_chart_template_full","l10n_es.account_group_1111","False"
"account_full_1143","Reserva por fondo de comercio","1143","account.data_account_type_equity","l10n_es.account_chart_template_full","l10n_es.account_group_1143","False"
"account_full_115","Reservas por pérdidas y ganancias actuariales y otros ajustes","115","account.data_account_type_equity","l10n_es.account_chart_template_full","l10n_es.account_group_115","False"
"account_full_120","Remanente","120","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_120","False"
"account_full_121","Resultados negativos de ejercicios anteriores","121","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_121","False"
"account_full_129","Resultado del ejercicio","129","account.data_unaffected_earnings","l10n_es.account_chart_template_full","l10n_es.account_group_129","False"
"account_full_130","Subvenciones oficiales de capital","130","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_130","False"
"account_full_132","Otras subvenciones, donaciones y legados","132","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_132","False"
"account_full_133","Ajustes por valoración en activos financieros disponibles para la venta","133","account.data_account_type_equity","l10n_es.account_chart_template_full","l10n_es.account_group_133","False"
"account_full_1340","Cobertura de flujos de efectivo","1340","account.data_account_type_equity","l10n_es.account_chart_template_full","l10n_es.account_group_1340","False"
"account_full_1341","Cobertura de una inversión neta en un negocio en el extranjero","1341","account.data_account_type_equity","l10n_es.account_chart_template_full","l10n_es.account_group_1341","False"
"account_full_135","Diferencias de conversión","135","account.data_account_type_equity","l10n_es.account_chart_template_full","l10n_es.account_group_135","False"
"account_full_136","Ajustes por valoración de activos no corrientes y grupos enajenables de elementos, mantenidos para la venta","136","account.data_account_type_equity","l10n_es.account_chart_template_full","l10n_es.account_group_136","False"
"account_full_140","Provisión por retribuciones del personal","140","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_140","True"
"account_full_146","Provisión para reestructuraciones","146","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_146","True"
"account_full_147","Provisión por transacciones con pagos basados en instrumentos de patrimonio","147","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_147","True"
"account_full_1765","Pasivos por derivados financieros, carter de negociación","1765","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_1765","True"
"account_full_1768","Pasivos por derivados financieros, instrumentos de cobertura","1768","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_1768","True"
"account_full_178","Obligaciones y bonos convertibles","178","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_178","False"
"account_full_189","Garantías financieras","189","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_189","False"
"account_full_204","Fondo de comercio","204","account.data_account_type_fixed_assets","l10n_es.account_chart_template_full","l10n_es.account_group_204","False"
"account_full_2403","Participaciones en empresas del grupo","2403","account.data_account_type_fixed_assets","l10n_es.account_chart_template_full","l10n_es.account_group_2403","False"
"account_full_2404","Participaciones en empresas asociadas","2404","account.data_account_type_fixed_assets","l10n_es.account_chart_template_full","l10n_es.account_group_2404","False"
"account_full_2405","Participaciones en otras partes vinculadas","2405","account.data_account_type_fixed_assets","l10n_es.account_chart_template_full","l10n_es.account_group_2405","False"
"account_full_2493","Desembolsos pendientes sobre participaciones en empresas del grupo","2493","account.data_account_type_fixed_assets","l10n_es.account_chart_template_full","l10n_es.account_group_2493","False"
"account_full_2494","Desembolsos pendientes sobre participaciones en empresas asociadas","2494","account.data_account_type_fixed_assets","l10n_es.account_chart_template_full","l10n_es.account_group_2494","False"
"account_full_2495","Desembolsos pendientes sobre participaciones en otras partes vinculadas","2495","account.data_account_type_fixed_assets","l10n_es.account_chart_template_full","l10n_es.account_group_2495","False"
"account_full_2550","Activos por derivados financieros, cartera de negociación","2550","account.data_account_type_fixed_assets","l10n_es.account_chart_template_full","l10n_es.account_group_2550","False"
"account_full_2553","Activos por derivados financieros, instrumentos de cobertura","2553","account.data_account_type_fixed_assets","l10n_es.account_chart_template_full","l10n_es.account_group_2553","False"
"account_full_257","Derechos de reembolso derivados de contratos de seguro relativos a retribuciones al personal","257","account.data_account_type_fixed_assets","l10n_es.account_chart_template_full","l10n_es.account_group_257","False"
"account_full_466","Remuneraciones mediante sistemas de aportación definida pendientes de pago","466","account.data_account_type_payable","l10n_es.account_chart_template_full","l10n_es.account_group_466","True"
"account_full_490","Deterioro de valor de créditos por operaciones comerciales","490","account.data_account_type_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_490","False"
"account_full_501","Obligaciones y bonos convertibles a corto plazo","501","account.data_account_type_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_501","False"
"account_full_5091","Obligaciones y bonos convertibles amortizados","5091","account.data_account_type_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_5091","True"
"account_full_5296","Provisión a corto plazo para reestructuraciones de patrimonio","5296","account.data_account_type_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_5296","True"
"account_full_5297","Provisión a corto plazo por transacciones con pagos basados en instrumentos","5297","account.data_account_type_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_5297","True"
"account_full_551","Cuenta corriente con socios y administradores","551","account.data_account_type_current_assets","l10n_es.account_chart_template_full","l10n_es.account_group_551","False"
"account_full_5530","Socios de sociedad disuelta","5530","account.data_account_type_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_5530","False"
"account_full_5531","Socios, cuenta de fusión","5531","account.data_account_type_current_assets","l10n_es.account_chart_template_full","l10n_es.account_group_5531","False"
"account_full_5532","Socios de sociedad escindida","5532","account.data_account_type_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_5532","False"
"account_full_5533","Socios, cuenta de escisión","5533","account.data_account_type_current_assets","l10n_es.account_chart_template_full","l10n_es.account_group_5533","False"
"account_full_5593","Activos por derivados financieros a corto plazo, instrumentos de cobertura","5593","account.data_account_type_current_assets","l10n_es.account_chart_template_full","l10n_es.account_group_5593","False"
"account_full_5598","Pasivos por derivados financieros a corto plazo, instrumentos de cobertura","5598","account.data_account_type_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_5598","False"
"account_full_569","Garantías financieras a corto plazo","569","account.data_account_type_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_569","False"
"account_full_580","Inmovilizado","580","account.data_account_type_current_assets","l10n_es.account_chart_template_full","l10n_es.account_group_580","False"
"account_full_581","Inversiones con personas y entidades vinculadas","581","account.data_account_type_current_assets","l10n_es.account_chart_template_full","l10n_es.account_group_581","False"
"account_full_582","Inversiones financieras","582","account.data_account_type_current_assets","l10n_es.account_chart_template_full","l10n_es.account_group_582","False"
"account_full_583","Existencias, deudores comerciales y otras cuentas a cobrar","583","account.data_account_type_current_assets","l10n_es.account_chart_template_full","l10n_es.account_group_583","False"
"account_full_584","Otros activos","584","account.data_account_type_current_assets","l10n_es.account_chart_template_full","l10n_es.account_group_584","False"
"account_full_585","Provisiones","585","account.data_account_type_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_585","False"
"account_full_586","Deudas con características especiales","586","account.data_account_type_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_586","False"
"account_full_587","Deudas con personas y entidades vinculadas","587","account.data_account_type_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_587","False"
"account_full_588","Acreedores comerciales y otras cuentas a pagar","588","account.data_account_type_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_588","False"
"account_full_589","Otros pasivos","589","account.data_account_type_current_liabilities","l10n_es.account_chart_template_full","l10n_es.account_group_589","False"
"account_full_5990","Deterioro de valor de inmovilizado no corriente mantenido para la venta","5990","account.data_account_type_current_assets","l10n_es.account_chart_template_full","l10n_es.account_group_5990","False"
"account_full_5991","Deterioro de valor de inversiones y entidades vinculadas no corrientes mantenidas para la venta","5991","account.data_account_type_current_assets","l10n_es.account_chart_template_full","l10n_es.account_group_5991","False"
"account_full_5992","Deterioro de valor de inversiones financieras no corrientes mantenidas para la venta","5992","account.data_account_type_current_assets","l10n_es.account_chart_template_full","l10n_es.account_group_5992","False"
"account_full_5993","Deterioro de valor de existencias, deudores comerciales y otras cuentas a cobrar integrados en un grupo enajenable mantenido para la venta","5993","account.data_account_type_current_assets","l10n_es.account_chart_template_full","l10n_es.account_group_5993","False"
"account_full_5994","Deterioro de valor de otros activos mantenidos para la venta","5994","account.data_account_type_current_assets","l10n_es.account_chart_template_full","l10n_es.account_group_5994","False"
"account_full_643","Retribuciones mediante sistemas de aportación definida","643","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_643","False"
"account_full_6440","Contribuciones anuales","6440","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_6440","False"
"account_full_6442","Otros costes","6442","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_6442","False"
"account_full_6450","Retribuciones al personal liquidados con instrumentos de patrimonio","6450","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_6450","False"
"account_full_6457","Retribuciones al personal liquidados en efectivo basado en instrumentos de patrimonio","6457","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_6457","False"
"account_full_650","Pérdidas de créditos comerciales incobrables","650","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_650","False"
"account_full_6510","Beneficio transferido (gestor)","6510","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_6510","False"
"account_full_6511","Pérdida soportada (partícipe o asociado no gestor)","6511","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_6511","False"
"account_full_659","Otras pérdidas en gestión corriente","659","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_659","False"
"account_full_6630","Pérdidas de cartera de negociación","6630","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_6630","False"
"account_full_6631","Pérdidas de designados por la empresa","6631","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_6631","False"
"account_full_6632","Pérdidas de disponibles para la venta","6632","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_6632","False"
"account_full_6633","Pérdidas de instrumentos de cobertura","6633","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_6633","False"
"account_full_671","Pérdidas procedentes del inmovilizado material","671","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_671","False"
"account_full_691","Pérdidas por deterioro del inmovilizado material","691","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_691","False"
"account_full_694","Pérdidas por deterioro de créditos por operaciones comerciales","694","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_694","False"
"account_full_7630","Beneficios de cartera de negociación","7630","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_7630","False"
"account_full_7631","Beneficios de designados por la empresa","7631","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_7631","False"
"account_full_7632","Beneficios de disponibles para la venta","7632","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_7632","False"
"account_full_7633","Beneficios de instrumentos de cobertura","7633","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_7633","False"
"account_full_767","Ingresos de activos afectos y de derechos de reembolso relativos a retribuciones a largo plazo","767","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_767","False"
"account_full_774","Diferencia negativa en combinaciones de negocios","774","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_774","False"
"account_full_791","Reversión del deterioro del inmovilizado material","791","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_791","False"
"account_full_794","Reversión del deterioro de créditos por operaciones comerciales","794","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_794","False"
"account_full_7950","Exceso de provisión por retribuciones al personal","7950","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_7950","False"
"account_full_7956","Exceso de provisión para reestructuraciones","7956","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_7956","False"
"account_full_7957","Exceso de provisión por transacciones con pagos basados en instrumentos de patrimonio","7957","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_7957","False"
"account_full_800","Pérdidas en activos financieros disponibles para la venta","800","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_800","False"
"account_full_802","Transferencia de beneficios en activos financieros disponibles para la venta","802","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_802","False"
"account_full_810","Pérdidas por coberturas de flujos de efectivo","810","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_810","False"
"account_full_811","Pérdidas por coberturas de inversiones netas en un negocio en el extranjero","811","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_811","False"
"account_full_812","Transferencia de beneficios por coberturas de flujos de efectivo","812","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_812","False"
"account_full_813","Transferencia de beneficios por coberturas de inversiones netas en un negocio en el extranjero","813","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_813","False"
"account_full_820","Diferencias de conversión negativas","820","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_820","False"
"account_full_821","Transferencia de diferencias de conversión positivas","821","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_821","False"
"account_full_8300","Impuesto corriente","8300","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_8300","False"
"account_full_8301","Impuesto diferido","8301","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_8301","False"
"account_full_833","Ajustes negativos en la imposición sobre beneficios","833","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_833","False"
"account_full_834","Ingresos fiscales por diferencias permanentes","834","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_834","False"
"account_full_835","Ingresos fiscales por deducciones y bonificaciones","835","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_835","False"
"account_full_836","Transferencia de diferencias permanentes","836","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_836","False"
"account_full_837","Transferencia de deducciones y bonificaciones","837","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_837","False"
"account_full_838","Ajustes positivos en la imposición sobre beneficios","838","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_838","False"
"account_full_840","Transferencia de subvenciones oficiales de capital","840","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_840","False"
"account_full_841","Transferencia de donaciones y legados de capital","841","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_841","False"
"account_full_842","Transferencia de otras subvenciones, donaciones y legados","842","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_842","False"
"account_full_850","Pérdidas actuariales","850","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_850","False"
"account_full_851","Ajustes negativos en activos por retribuciones a largo plazo de prestación definida","851","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_851","False"
"account_full_860","Pérdidas en activos no corrientes y grupos enajenables de elementos mantenidos para la venta","860","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_860","False"
"account_full_862","Transferencia de beneficios en activos no corrientes y grupos enajenables de elementos mantenidos para la venta","862","account.data_account_type_expenses","l10n_es.account_chart_template_full","l10n_es.account_group_862","False"
"account_full_891","Deterioro de participaciones en el patrimonio, empresas del grupo","891","account.data_account_type_depreciation","l10n_es.account_chart_template_full","l10n_es.account_group_891","False"
"account_full_892","Deterioro de participaciones en el patrimonio, empresas asociadas","892","account.data_account_type_depreciation","l10n_es.account_chart_template_full","l10n_es.account_group_892","False"
"account_full_900","Beneficios en activos financieros disponibles para la venta","900","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_900","False"
"account_full_902","Transferencia de pérdidas en activos financieros disponibles para la venta","902","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_902","False"
"account_full_910","Beneficios por coberturas de flujos de efectivo","910","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_910","False"
"account_full_911","Beneficios por coberturas de inversiones netas en un negocio en el extranjero","911","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_911","False"
"account_full_912","Transferencia de pérdidas por coberturas de flujos de efectivo","912","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_912","False"
"account_full_913","Transferencia de pérdidas por coberturas de inversiones netas en un negocio en el extranjero","913","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_913","False"
"account_full_920","Diferencias de conversión positivas","920","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_920","False"
"account_full_921","Transferencia de diferencias de conversión negativas","921","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_921","False"
"account_full_940","Ingresos de subvenciones oficiales de capital","940","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_940","False"
"account_full_941","Ingresos de donaciones y legados de capital","941","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_941","False"
"account_full_942","Ingresos de otras subvenciones, donaciones y legados","942","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_942","False"
"account_full_950","Ganancias actuariales","950","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_950","False"
"account_full_951","Ajustes positivos en activos por retribuciones a largo plazo de prestación definida","951","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_951","False"
"account_full_960","Beneficios en activos no corrientes y grupos enajenables de elementos mantenidos para la venta","960","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_960","False"
"account_full_962","Transferencia de pérdidas en activos no corrientes y grupos enajenables de elementos mantenidos para la venta","962","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_962","False"
"account_full_991","Recuperación de ajustes valorativos negativos previos, empresas del grupo","991","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_991","False"
"account_full_992","Recuperación de ajustes valorativos negativos previos, empresas asociadas","992","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_992","False"
"account_full_993","Transferencia por deterioro de ajustes valorativos negativos previos, empresas del grupo","993","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_993","False"
"account_full_994","Transferencia por deterioro de ajustes valorativos negativos previos, empresas asociadas","994","account.data_account_type_revenue","l10n_es.account_chart_template_full","l10n_es.account_group_994","False"

```

## File: data\account.account.template-pymes.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","group_id/id"
"account_pymes_100","Capital social","100","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_pymes","l10n_es.account_group_100"
"account_pymes_1030","Socios por desembolsos no exigidos, capital social","1030","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_pymes","l10n_es.account_group_1030"
"account_pymes_1034","Socios por desembolsos no exigidos, capital pendiente de inscripción","1034","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_pymes","l10n_es.account_group_1034"
"account_pymes_1040","Socios por aportaciones no dinerarias pendientes, capital social","1040","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_pymes","l10n_es.account_group_1040"
"account_pymes_1044","Socios por aportaciones no dinerarias pendientes, capital pendiente de inscripción","1044","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_pymes","l10n_es.account_group_1044"
"account_pymes_120","Remanente","120","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_pymes","l10n_es.account_group_120"
"account_pymes_121","Resultados negativos de ejercicios anteriores","121","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_pymes","l10n_es.account_group_121"
"account_pymes_129","Resultado del ejercicio","129","account.data_unaffected_earnings","l10n_es.account_chart_template_pymes","l10n_es.account_group_129"
"account_pymes_130","Subvenciones oficiales de capital","130","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_pymes","l10n_es.account_group_130"
"account_pymes_132","Otras subvenciones, donaciones y legados","132","account.data_account_type_non_current_liabilities","l10n_es.account_chart_template_pymes","l10n_es.account_group_132"
"account_pymes_2403","Participaciones en empresas del grupo","2403","account.data_account_type_fixed_assets","l10n_es.account_chart_template_pymes","l10n_es.account_group_2403"
"account_pymes_2404","Participaciones en empresas asociadas","2404","account.data_account_type_fixed_assets","l10n_es.account_chart_template_pymes","l10n_es.account_group_2404"
"account_pymes_2405","Participaciones en otras partes vinculadas","2405","account.data_account_type_fixed_assets","l10n_es.account_chart_template_pymes","l10n_es.account_group_2405"
"account_pymes_2493","Desembolsos pendientes sobre participaciones en empresas del grupo","2493","account.data_account_type_fixed_assets","l10n_es.account_chart_template_pymes","l10n_es.account_group_2493"
"account_pymes_2494","Desembolsos pendientes sobre participaciones en empresas asociadas","2494","account.data_account_type_fixed_assets","l10n_es.account_chart_template_pymes","l10n_es.account_group_2494"
"account_pymes_2495","Desembolsos pendientes sobre participaciones en otras partes vinculadas","2495","account.data_account_type_fixed_assets","l10n_es.account_chart_template_pymes","l10n_es.account_group_2495"
"account_pymes_255","Activos por derivados financieros","255","account.data_account_type_fixed_assets","l10n_es.account_chart_template_pymes","l10n_es.account_group_255"
"account_pymes_2935","Deterioro de valor de participaciones a largo plazo en otras partes vinculadas","2935","account.data_account_type_fixed_assets","l10n_es.account_chart_template_pymes","l10n_es.account_group_293"
"account_pymes_296","Deterioro de  valor de participaciones en el patrimonio neto a largo plazo","296","account.data_account_type_fixed_assets","l10n_es.account_chart_template_pymes","l10n_es.account_group_296"
"pgc_pyme_490","Deterioro de valor de créditos por operaciones comerciales","490","account.data_account_type_current_liabilities","l10n_es.account_chart_template_pymes","l10n_es.account_group_490"
"pgc_pyme_551","Cuenta corriente con socios y administradores","551","account.data_account_type_current_assets","l10n_es.account_chart_template_pymes","l10n_es.account_group_551"
"account_pymes_5935","Deterioro de valor de participaciones a corto plazo en otras partes vinculadas","5935","account.data_account_type_current_assets","l10n_es.account_chart_template_pymes","l10n_es.account_group_593"
"account_pymes_596","Deterioro de valor de participaciones a corto plazo","596","account.data_account_type_current_assets","l10n_es.account_chart_template_pymes","l10n_es.account_group_596"
"account_pymes_650","Pérdidas de créditos comerciales incobrables","650","account.data_account_type_expenses","l10n_es.account_chart_template_pymes","l10n_es.account_group_650"
"account_pymes_6510","Beneficio transferido (gestor)","6510","account.data_account_type_expenses","l10n_es.account_chart_template_pymes","l10n_es.account_group_6510"
"account_pymes_6511","Pérdida soportada (partícipe o asociado no gestor)","6511","account.data_account_type_expenses","l10n_es.account_chart_template_pymes","l10n_es.account_group_6511"
"account_pymes_659","Otras pérdidas en gestión corriente","659","account.data_account_type_expenses","l10n_es.account_chart_template_pymes","l10n_es.account_group_659"
"account_pymes_663","Pérdidas por valoración de instrumentos financieros por su valor razonable","663","account.data_account_type_expenses","l10n_es.account_chart_template_pymes","l10n_es.account_group_663"
"account_pymes_671","Pérdidas procedentes del inmovilizado material","671","account.data_account_type_expenses","l10n_es.account_chart_template_pymes","l10n_es.account_group_671"
"account_pymes_691","Pérdidas por deterioro del inmovilizado material","691","account.data_account_type_expenses","l10n_es.account_chart_template_pymes","l10n_es.account_group_691"
"account_pymes_694","Pérdidas por deterioro de créditos por operaciones comerciales","694","account.data_account_type_expenses","l10n_es.account_chart_template_pymes","l10n_es.account_group_694"
"account_pymes_763","Beneficios por valoración de instrumentos financieros por su valor razonable","763","account.data_account_type_revenue","l10n_es.account_chart_template_pymes","l10n_es.account_group_763"
"account_pymes_791","Reversión del deterioro del inmovilizado material","791","account.data_account_type_revenue","l10n_es.account_chart_template_pymes","l10n_es.account_group_791"
"account_pymes_794","Reversión del deterioro de créditos por operaciones comerciales","794","account.data_account_type_revenue","l10n_es.account_chart_template_pymes","l10n_es.account_group_794"
"account_pymes_7962","Reversión del deterioro de participaciones en instrumentos de patrimonio neto a largo plazo, otras partes vinculadas","7962","account.data_account_type_revenue","l10n_es.account_chart_template_pymes","l10n_es.account_group_796"
"account_pymes_7963","Reversión del deterioro de participaciones en instrumentos de patrimonio neto a largo plazo, otras empresas","7963","account.data_account_type_revenue","l10n_es.account_chart_template_pymes","l10n_es.account_group_796"

```

## File: data\account_chart_template_account_account_link.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <!--We complete the declaration of the chart of account here.
        These data are to be created once the accounts have been created in database,
        they finish "linking" the CoA to them.-->
        <record id="account_chart_template_common" model="account.chart.template">
            <field name="property_account_receivable_id" ref="account_common_4300"/>
            <field name="property_account_payable_id" ref="account_common_4100"/>
            <field name="property_account_expense_categ_id" ref="account_common_600"/>
            <field name="property_account_income_categ_id" ref="account_common_7000"/>
            <field name="income_currency_exchange_account_id" ref="account_common_768"/>
            <field name="expense_currency_exchange_account_id" ref="account_common_668"/>
            <field name="default_pos_receivable_account_id" ref="account_common_4301" />
        </record>

    </data>
</odoo>
```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_es.account_chart_template_pymes')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="account_chart_template_common" model="account.chart.template">
            <field name="name">PGCE común</field>
            <field name="visible" eval="0"/>
            <field name="currency_id" ref="base.EUR"/>
            <field name="cash_account_code_prefix">570</field>
            <field name="bank_account_code_prefix">572</field>
            <field name="transfer_account_code_prefix">572999</field>
        </record>

        <record id="account_chart_template_pymes" model="account.chart.template">
            <field name="name">PGCE PYMEs 2008</field>
            <field name="complete_tax_set" eval="True"/>
            <field name="currency_id" ref="base.EUR"/>
            <field name="cash_account_code_prefix">570</field>
            <field name="bank_account_code_prefix">572</field>
            <field name="transfer_account_code_prefix">572999</field>
            <field name="parent_id" ref="account_chart_template_common"/>
        </record>

        <record id="account_chart_template_assoc" model="account.chart.template">
        <field name="name">PGCE entidades sin ánimo de lucro 2008</field>
        <field name="complete_tax_set" eval="True"/>
        <field name="currency_id" ref="base.EUR"/>
        <field name="cash_account_code_prefix">570</field>
        <field name="bank_account_code_prefix">572</field>
        <field name="transfer_account_code_prefix">572999</field>
        <field name="parent_id" ref="account_chart_template_common"/>
    </record>

    <record id="account_chart_template_full" model="account.chart.template">
        <field name="name">PGCE completo 2008</field>
        <field name="complete_tax_set" eval="True"/>
        <field name="currency_id" ref="base.EUR"/>
        <field name="cash_account_code_prefix">570</field>
        <field name="bank_account_code_prefix">572</field>
        <field name="transfer_account_code_prefix">572999</field>
        <field name="parent_id" ref="account_chart_template_common"/>
    </record>
    </data>
</odoo>

```

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Account Tax Group -->
        <record id="tax_group_iva_0" model="account.tax.group">
            <field name="name">IVA 0%</field>
        </record>
        <record id="tax_group_recargo_0" model="account.tax.group">
            <field name="name">Recargo de Equivalencia 0%</field>
        </record>
        <record id="tax_group_recargo_0-5" model="account.tax.group">
            <field name="name">Recargo de Equivalencia 0.5%</field>
        </record>
        <record id="tax_group_recargo_0-62" model="account.tax.group">
            <field name="name">Recargo de Equivalencia 0.62%</field>
        </record>
        <record id="tax_group_retenciones_1" model="account.tax.group">
            <field name="name">Retenciones 1%</field>
        </record>
        <record id="tax_group_recargo_1-4" model="account.tax.group">
            <field name="name">Recargo de Equivalencia 1.4%</field>
        </record>
        <record id="tax_group_retenciones_2" model="account.tax.group">
            <field name="name">Retenciones 2%</field>
        </record>
        <record id="tax_group_iva_4" model="account.tax.group">
            <field name="name">IVA 4%</field>
        </record>
        <record id="tax_group_recargo_5-2" model="account.tax.group">
            <field name="name">Recargo de Equivalencia 5.2%</field>
        </record>
        <record id="tax_group_retenciones_7" model="account.tax.group">
            <field name="name">Retenciones 7%</field>
        </record>
        <record id="tax_group_retenciones_9" model="account.tax.group">
            <field name="name">Retenciones 9%</field>
        </record>
        <record id="tax_group_iva_10" model="account.tax.group">
            <field name="name">IVA 10%</field>
        </record>
        <record id="tax_group_iva_12" model="account.tax.group">
            <field name="name">IVA 12%</field>
        </record>
        <record id="tax_group_retenciones_15" model="account.tax.group">
            <field name="name">Retenciones 15%</field>
        </record>
        <record id="tax_group_retenciones_18" model="account.tax.group">
            <field name="name">Retenciones 18%</field>
        </record>
        <record id="tax_group_retenciones_19" model="account.tax.group">
            <field name="name">Retenciones 19%</field>
        </record>
        <record id="tax_group_retenciones_19-5" model="account.tax.group">
            <field name="name">Retenciones 19.5%</field>
        </record>
        <record id="tax_group_retenciones_20" model="account.tax.group">
            <field name="name">Retenciones 20%</field>
        </record>
        <record id="tax_group_iva_21" model="account.tax.group">
            <field name="name">IVA 21%</field>
        </record>
        <record id="tax_group_retenciones_21" model="account.tax.group">
            <field name="name">Retenciones 21%</field>
        </record>
        <record id="tax_group_retenciones_24" model="account.tax.group">
            <field name="name">Retenciones 24%</field>
        </record>
        <record id="tax_group_iva_10-5" model="account.tax.group">
            <field name="name">IVA 10,5% REAGYP</field>
        </record>
        <record id="tax_group_iva_nd" model="account.tax.group">
            <field name="name">IVA no deducible</field>
        </record>
        <record id="tax_group_iva_5" model="account.tax.group">
            <field name="name">IVA 5%</field>
        </record>
    </data>
</odoo>

```

## File: data\account_fiscal_position_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- © 2009-2011 Jordi Esteve - Zikzakmedia
     © 2011-2020 Ignacio Ibeas - Acysos
     © 2015 Carlos Liébana - Factor Libre
     © 2015 Albert Cabedo - GAFIC consultores
     © 2015 Vicent Cubells
     © 2013-2016 Pedro M. Baeza
     © 2020 Harald Panten - Sygel Technology
     License AGPL-3.0 or later (http://www.gnu.org/licenses/agpl). -->
<odoo>
    <data>

        <!-- ************************************************************* -->
        <!-- Fiscal Position Templates -->
        <!-- ************************************************************* -->

        <record id="fp_nacional" model="account.fiscal.position.template">
            <field name="sequence">1</field>
            <field name="name">Régimen Nacional</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_id" ref="base.es"/>
        </record>

        <record id="fp_intra_private" model="account.fiscal.position.template">
            <field name="sequence">2</field>
            <field name="name">EU privado</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
            <field name="auto_apply" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
        </record>

        <record id="fp_intra" model="account.fiscal.position.template">
            <field name="sequence">3</field>
            <field name="name">Régimen Intracomunitario</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
        </record>

        <record id="fp_extra" model="account.fiscal.position.template">
            <field name="sequence">4</field>
            <field name="name">Régimen Extracomunitario</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
            <field name="auto_apply" eval="True"/>
        </record>

        <record id="fp_not_subject_tai" model="account.fiscal.position.template">
            <field name="name">Régimen No sujeto por reglas de localización (TAI - Canarias, Ceuta, Melilla...)</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
            <field name="note">
                Está posición fiscal solo aplica en el caso de vender desde territorio
                nacional a una localización no sujeta, por ejemplo, Canarias, Ceuta o Melilla.
                Más información: https://www.agenciatributaria.es/AEAT.internet/en_gb/Inicio/Ayuda/Modelos__Procedimientos_y_Servicios/Ayuda_Modelo_303/Informacion_general/Instrucciones_modelo_303.shtml
            </field>
        </record>


        <record id="fp_recargo" model="account.fiscal.position.template">
            <field name="name">Recargo de Equivalencia</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
        </record>

        <record id="fp_recargo_isp" model="account.fiscal.position.template">
            <field name="name">Recargo de Equivalencia Revendedor con ISP</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
        </record>

        <record id="fp_irpf1" model="account.fiscal.position.template">
            <field name="name">Retención IRPF 1%</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
        </record>

        <record id="fp_irpf2" model="account.fiscal.position.template">
            <field name="name">Retención IRPF 2%</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
        </record>

        <record id="fp_irpf7" model="account.fiscal.position.template">
            <field name="name">Retención IRPF 7%</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
        </record>

        <record id="fp_irpf9" model="account.fiscal.position.template">
            <field name="name">Retención IRPF 9%</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
        </record>

        <record id="fp_irpf15" model="account.fiscal.position.template">
            <field name="name">Retención IRPF 15%</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
        </record>

        <record id="fp_irpf18" model="account.fiscal.position.template">
            <field name="name">Retención IRPF 18%</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
        </record>

        <record id="fp_irpf19" model="account.fiscal.position.template">
            <field name="name">Retención IRPF 19%</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
        </record>

        <record id="fp_irpf19a" model="account.fiscal.position.template">
            <field name="name">Retención 19% arrendamientos</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
        </record>

        <record id="fp_irpf195a" model="account.fiscal.position.template">
            <field name="name">Retención 19,5% arrendamientos</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
        </record>

        <record id="fp_irpf20" model="account.fiscal.position.template">
            <field name="name">Retención IRPF 20%</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
        </record>

        <record id="fp_irpf24" model="account.fiscal.position.template">
            <field name="name">Retención IRPF 24%</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
        </record>

        <record id="fp_irpf20a" model="account.fiscal.position.template">
            <field name="name">Retención 20% arrendamientos</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
        </record>

        <record id="fp_irpf21" model="account.fiscal.position.template">
            <field name="name">Retención IRPF 21%</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
        </record>

        <record id="fp_irpf21a" model="account.fiscal.position.template">
            <field name="name">Retención 21% arrendamientos</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
        </record>

        <record id="fp_ispn" model="account.fiscal.position.template">
            <field name="name">Inversion del Sujeto Pasivo Nacional</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
        </record>

        <record id="fp_rnrisp" model="account.fiscal.position.template">
            <field name="name">Régimen Nacional Revendedor con Inversión del sujeto pasivo</field>
            <field name="chart_template_id" ref="account_chart_template_common"/>
        </record>

        <record id="fp_reagyp_a" model="account.fiscal.position.template">
            <field name="name">REAGYP - Agricultura</field>
            <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        </record>

       <record id="fp_reagyp_gp" model="account.fiscal.position.template">
           <field name="name">REAGYP - Ganadería y pesca</field>
           <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
       </record>

        <!-- ************************************************************* -->
        <!-- Fiscal Position Tax Templates -->
        <!-- ************************************************************* -->

        <!-- Extracomunitarios -->

        <record id="fptt_extra_0b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_s_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ibc"/>
        </record>
        <record id="fptt_extra_0s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_s_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_isc"/>
        </record>
        <record id="fptt_extra_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_ibc"/>
        </record>
        <record id="fptt_extra_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_sp_ex"/>
        </record>
        <record id="fptt_extra_4_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_ibi"/>
        </record>
        <record id="fptt_extra_5b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva5_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva5_ibc"/>
        </record>
        <record id="fptt_extra_5s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva5_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva5_isc"/>
        </record>
        <record id="fptt_extra_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_ibc"/>
        </record>
        <record id="fptt_extra_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_sp_ex"/>
        </record>
        <record id="fptt_extra_10_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_ibi"/>
        </record>
        <record id="fptt_extra_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_ibc"/>
        </record>
        <record id="fptt_extra_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_sp_ex"/>
        </record>
        <record id="fptt_extra_21_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_ibi"/>
        </record>
        <record id="fptt_extra_ventas_0b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_e"/>
        </record>
        <record id="fptt_extra_ventas_0s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva_e"/>
        </record>
        <record id="fptt_extra_ventas_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_e"/>
        </record>
        <record id="fptt_extra_ventas_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva_e"/>
        </record>
        <record id="fptt_extra_ventas_5b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva5b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_e"/>
        </record>
        <record id="fptt_extra_ventas_5s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva5s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva_e"/>
        </record>
        <record id="fptt_extra_ventas_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_e"/>
        </record>
        <record id="fptt_extra_ventas_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva_e"/>
        </record>
        <record id="fptt_extra_ventas_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_e"/>
        </record>
        <record id="fptt_extra_ventas_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva_e"/>
        </record>
        <record id="fptt_extra_ventas_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_e"/>
        </record>

        <!-- Intracomunitarios -->

        <record id="fptt_intra_0b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_s_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ic_bc"/>
        </record>
        <record id="fptt_intra_0s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_s_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ic_sc"/>
        </record>
        <record id="fptt_intra_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_ic_bc"/>
        </record>
        <record id="fptt_intra_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_sp_in"/>
        </record>
        <record id="fptt_intra_4_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_ic_bi"/>
        </record>
        <record id="fptt_intra_5b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva5_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva5_ic_bc"/>
        </record>
        <record id="fptt_intra_5s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva5_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva5_ic_sc"/>
        </record>
        <record id="fptt_intra_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_ic_bc"/>
        </record>
        <record id="fptt_intra_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_sp_in"/>
        </record>
        <record id="fptt_intra_10_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_ic_bi"/>
        </record>
        <record id="fptt_intra_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_ic_bc"/>
        </record>
        <record id="fptt_intra_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_sp_in"/>
        </record>
        <record id="fptt_intra_21b_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_ic_bi"/>
        </record>
        <record id="fptt_intra_ventas_0b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_ic"/>
        </record>
        <record id="fptt_intra_ventas_0s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_sp_i"/>
        </record>
        <record id="fptt_intra_ventas_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_ic"/>
        </record>
        <record id="fptt_intra_ventas_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_sp_i"/>
        </record>
        <record id="fptt_intra_ventas_5b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva5b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_ic"/>
        </record>
        <record id="fptt_intra_ventas_5s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva5s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_sp_i"/>
        </record>
        <record id="fptt_intra_ventas_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_ic"/>
        </record>
        <record id="fptt_intra_ventas_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_sp_i"/>
        </record>
        <record id="fptt_intra_ventas_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_ic"/>
        </record>
        <record id="fptt_intra_ventas_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_sp_i"/>
        </record>
        <record id="fptt_intra_ventas_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_ic"/>
        </record>

        <!-- Not subject by Localization Rules (TAI) -->

        <record id="fptt_not_subject_tai_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_not_subject_tai"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns_b"/>
        </record>
        <record id="fptt_not_subject_tai_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_not_subject_tai"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns"/>
        </record>
        <record id="fptt_not_subject_tai_4_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_not_subject_tai"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns_b"/>
        </record>
        <record id="fptt_not_subject_tai_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_not_subject_tai"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns_b"/>
        </record>
        <record id="fptt_not_subject_tai_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_not_subject_tai"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns"/>
        </record>
        <record id="fptt_not_subject_tai_10_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_not_subject_tai"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns_b"/>
        </record>
        <record id="fptt_not_subject_tai_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_not_subject_tai"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns_b"/>
        </record>
        <record id="fptt_not_subject_tai_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_not_subject_tai"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns"/>
        </record>
        <record id="fptt_not_subject_tai_21_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_not_subject_tai"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns_b"/>
        </record>
        <record id="fptt_not_subject_tai_ventas_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_not_subject_tai"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva_ns_b"/>
        </record>
        <record id="fptt_not_subject_tai_ventas_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_not_subject_tai"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva_ns"/>
        </record>
        <record id="fptt_not_subject_tai_ventas_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_not_subject_tai"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva_ns_b"/>
        </record>
        <record id="fptt_not_subject_tai_ventas_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_not_subject_tai"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva_ns"/>
        </record>
        <record id="fptt_not_subject_tai_ventas_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_not_subject_tai"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva_ns_b"/>
        </record>
        <record id="fptt_not_subject_tai_ventas_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_not_subject_tai"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva_ns"/>
        </record>
        <record id="fptt_not_subject_tai_ventas_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_not_subject_tai"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva_ns_b"/>
        </record>

        <!-- Recargo de equivalencia -->

        <record id="fptt_recargo_0b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0b"/>
        </record>
        <record id="fptt_recargo_0b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_req0"/>
        </record>
        <record id="fptt_recargo_0s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0s"/>
        </record>
        <record id="fptt_recargo_0s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_req0"/>
        </record>
        <record id="fptt_recargo_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4b"/>
        </record>
        <record id="fptt_recargo_4b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_req05"/>
        </record>
        <record id="fptt_recargo_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4s"/>
        </record>
        <record id="fptt_recargo_4s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_req05"/>
        </record>
        <record id="fptt_recargo_5b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva5b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva5b"/>
        </record>
        <record id="fptt_recargo_5b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva5b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_req062"/>
        </record>
        <record id="fptt_recargo_5s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva5s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva5s"/>
        </record>
        <record id="fptt_recargo_5s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva5s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_req062"/>
        </record>
        <record id="fptt_recargo_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10b"/>
        </record>
        <record id="fptt_recargo_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_req014"/>
        </record>
        <record id="fptt_recargo_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10s"/>
        </record>
        <record id="fptt_recargo_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_req014"/>
        </record>
        <record id="fptt_recargo_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21b"/>
        </record>
        <record id="fptt_recargo_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_req52"/>
        </record>
        <record id="fptt_recargo_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21s"/>
        </record>
        <record id="fptt_recargo_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_req52"/>
        </record>
        <record id="fptt_recargo_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21isp"/>
        </record>
        <record id="fptt_recargo_21isp_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_req52"/>
        </record>
        <record id="fptt_recargo_buy_0b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_s_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_s_bc"/>
        </record>
        <record id="fptt_recargo_buy_0b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_s_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req0"/>
        </record>
        <record id="fptt_recargo_buy_0s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_s_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_s_sc"/>
        </record>
        <record id="fptt_recargo_buy_0s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_s_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req0"/>
        </record>
        <record id="fptt_recargo_buy_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_bc"/>
        </record>
        <record id="fptt_recargo_buy_4b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req05"/>
        </record>
        <record id="fptt_recargo_buy_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_sc"/>
        </record>
        <record id="fptt_recargo_buy_4s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req05"/>
        </record>
        <record id="fptt_recargo_buy_4_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_bi"/>
        </record>
        <record id="fptt_recargo_buy_4_2_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req05"/>
        </record>
        <record id="fptt_recargo_buy_5b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva5_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva5_bc"/>
        </record>
        <record id="fptt_recargo_buy_5b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva5_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req062"/>
        </record>
        <record id="fptt_recargo_buy_5s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva5_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva5_sc"/>
        </record>
        <record id="fptt_recargo_buy_5s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva5_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req062"/>
        </record>
        <record id="fptt_recargo_buy_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_bc"/>
        </record>
        <record id="fptt_recargo_buy_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req014"/>
        </record>
        <record id="fptt_recargo_buy_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_sc"/>
        </record>
        <record id="fptt_recargo_buy_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req014"/>
        </record>
        <record id="fptt_recargo_buy_10_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_bi"/>
        </record>
        <record id="fptt_recargo_buy_10_2_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req014"/>
        </record>
        <record id="fptt_recargo_buy_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_bc"/>
        </record>
        <record id="fptt_recargo_buy_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req52"/>
        </record>
        <record id="fptt_recargo_buy_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_sc"/>
        </record>
        <record id="fptt_recargo_buy_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req52"/>
        </record>
        <record id="fptt_recargo_buy_21_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_bi"/>
        </record>
        <record id="fptt_recargo_buy_21_2_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req52"/>
        </record>

        <!-- Recargo de equivalencia ISP -->

        <record id="fptt_recargo_isp_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4b"/>
        </record>
        <record id="fptt_recargo_isp_4b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_req05"/>
        </record>
        <record id="fptt_recargo_isp_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4s"/>
        </record>
        <record id="fptt_recargo_isp_4s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_req05"/>
        </record>
        <record id="fptt_recargo_isp_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10b"/>
        </record>
        <record id="fptt_recargo_isp_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_req014"/>
        </record>
        <record id="fptt_recargo_isp_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10s"/>
        </record>
        <record id="fptt_recargo_isp_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_req014"/>
        </record>
        <record id="fptt_recargo_isp_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21b"/>
        </record>
        <record id="fptt_recargo_isp_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_req52"/>
        </record>
        <record id="fptt_recargo_isp_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21s"/>
        </record>
        <record id="fptt_recargo_isp_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_req52"/>
        </record>
        <record id="fptt_recargo_isp_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_isp"/>
        </record>
        <record id="fptt_recargo_isp_buy_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_bc"/>
        </record>
        <record id="fptt_recargo_isp_buy_4b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req05"/>
        </record>
        <record id="fptt_recargo_isp_buy_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_sc"/>
        </record>
        <record id="fptt_recargo_isp_buy_4s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req05"/>
        </record>
        <record id="fptt_recargo_isp_buy_4_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_bi"/>
        </record>
        <record id="fptt_recargo_isp_buy_4_2_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req05"/>
        </record>
        <record id="fptt_recargo_isp_buy_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_bc"/>
        </record>
        <record id="fptt_recargo_isp_buy_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req014"/>
        </record>
        <record id="fptt_recargo_isp_buy_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_sc"/>
        </record>
        <record id="fptt_recargo_isp_buy_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req014"/>
        </record>
        <record id="fptt_recargo_isp_buy_10_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_bi"/>
        </record>
        <record id="fptt_recargo_isp_buy_10_2_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req014"/>
        </record>
        <record id="fptt_recargo_isp_buy_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_bc"/>
        </record>
        <record id="fptt_recargo_isp_buy_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req52"/>
        </record>
        <record id="fptt_recargo_isp_buy_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_sc"/>
        </record>
        <record id="fptt_recargo_isp_buy_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req52"/>
        </record>
        <record id="fptt_recargo_isp_buy_21_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_bi"/>
        </record>
        <record id="fptt_recargo_isp_buy_21_2_inv"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_recargo_isp"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_req52"/>
        </record>

        <!-- Retención 19% arrendamientos -->

        <record id="fptt_irpf19asale_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21b"/>
        </record>
        <record id="fptt_irpf19asale_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf19a"/>
        </record>
        <record id="fptt_irpf19asale_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21s"/>
        </record>
        <record id="fptt_irpf19asale_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf19a"/>
        </record>
        <record id="fptt_irpf19asale_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21isp"/>
        </record>
        <record id="fptt_irpf19asale_21isp_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf19a"/>
        </record>
        <record id="fptt_irpf19asale_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10b"/>
        </record>
        <record id="fptt_irpf19asale_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf19a"/>
        </record>
        <record id="fptt_irpf19asale_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10s"/>
        </record>
        <record id="fptt_irpf19asale_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf19a"/>
        </record>
        <record id="fptt_irpf19asale_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4b"/>
        </record>
        <record id="fptt_irpf19asale_4b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf19a"/>
        </record>
        <record id="fptt_irpf19asale_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4s"/>
        </record>
        <record id="fptt_irpf19asale_4s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf19a"/>
        </record>
        <record id="fptt_irpf19a_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_bc"/>
        </record>
        <record id="fptt_irpf19a_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf19a"/>
        </record>
        <record id="fptt_irpf19a_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_sc"/>
        </record>
        <record id="fptt_irpf19a_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf19a"/>
        </record>
        <record id="fptt_irpf19a_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_bc"/>
        </record>
        <record id="fptt_irpf19a_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf19a"/>
        </record>
        <record id="fptt_irpf19a_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_sc"/>
        </record>
        <record id="fptt_irpf19a_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf19a"/>
        </record>
        <record id="fptt_irpf19a_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_bc"/>
        </record>
        <record id="fptt_irpf19a_4b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf19a"/>
        </record>
        <record id="fptt_irpf19a_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_sc"/>
        </record>
        <record id="fptt_irpf19a_4s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf19a"/>
        </record>
        <record id="fptt_irpf19asale_ex"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0"/>
        </record>
        <record id="fptt_irpf19asale_ex_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf19a"/>
        </record>
        <record id="fptt_irpf19a_0" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_bc"/>
        </record>
        <record id="fptt_irpf19a_0_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf19a"/>
        </record>

        <!-- Retención 19,5% arrendamientos -->

        <record id="fptt_irpf195asale_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21b"/>
        </record>
        <record id="fptt_irpf195asale_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf195a"/>
        </record>
        <record id="fptt_irpf195asale_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21s"/>
        </record>
        <record id="fptt_irpf195asale_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf195a"/>
        </record>
        <record id="fptt_irpf195asale_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21isp"/>
        </record>
        <record id="fptt_irpf195asale_21isp_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf195a"/>
        </record>
        <record id="fptt_irpf195asale_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10b"/>
        </record>
        <record id="fptt_irpf195asale_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf195a"/>
        </record>
        <record id="fptt_irpf195asale_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10s"/>
        </record>
        <record id="fptt_irpf195asale_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf195a"/>
        </record>
        <record id="fptt_irpf195asale_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4b"/>
        </record>
        <record id="fptt_irpf195asale_4b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf195a"/>
        </record>
        <record id="fptt_irpf195asale_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4s"/>
        </record>
        <record id="fptt_irpf195asale_4s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf195a"/>
        </record>
        <record id="fptt_irpf195a_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_bc"/>
        </record>
        <record id="fptt_irpf195a_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf195a"/>
        </record>
        <record id="fptt_irpf195a_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_sc"/>
        </record>
        <record id="fptt_irpf195a_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf195a"/>
        </record>
        <record id="fptt_irpf195a_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_bc"/>
        </record>
        <record id="fptt_irpf195a_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf195a"/>
        </record>
        <record id="fptt_irpf195a_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_sc"/>
        </record>
        <record id="fptt_irpf195a_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf195a"/>
        </record>
        <record id="fptt_irpf195a_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_bc"/>
        </record>
        <record id="fptt_irpf195a_4b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf195a"/>
        </record>
        <record id="fptt_irpf195a_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_sc"/>
        </record>
        <record id="fptt_irpf195a_4s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf195a"/>
        </record>
        <record id="fptt_irpf195asale_ex"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0"/>
        </record>
        <record id="fptt_irpf195asale_ex_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf195a"/>
        </record>
        <record id="fptt_irpf195a_0" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_bc"/>
        </record>
        <record id="fptt_irpf195a_0_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf195a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf195a"/>
        </record>

        <!-- Retención 20% arrendamientos -->

        <record id="fptt_irpf20asale_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21b"/>
        </record>
        <record id="fptt_irpf20asale_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf20a"/>
        </record>
        <record id="fptt_irpf20asale_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21s"/>
        </record>
        <record id="fptt_irpf20asale_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf20a"/>
        </record>
        <record id="fptt_irpf20asale_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21isp"/>
        </record>
        <record id="fptt_irpf20asale_21isp_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf20a"/>
        </record>
        <record id="fptt_irpf20asale_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10b"/>
        </record>
        <record id="fptt_irpf20asale_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf20a"/>
        </record>
        <record id="fptt_irpf20asale_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10s"/>
        </record>
        <record id="fptt_irpf20asale_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf20a"/>
        </record>
        <record id="fptt_irpf20asale_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4b"/>
        </record>
        <record id="fptt_irpf20asale_4b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf20a"/>
        </record>
        <record id="fptt_irpf20asale_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4s"/>
        </record>
        <record id="fptt_irpf20asale_4s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf20a"/>
        </record>
        <record id="fptt_irpf20a_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_bc"/>
        </record>
        <record id="fptt_irpf20a_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf20a"/>
        </record>
        <record id="fptt_irpf20a_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_sc"/>
        </record>
        <record id="fptt_irpf20a_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf20a"/>
        </record>
        <record id="fptt_irpf20a_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_bc"/>
        </record>
        <record id="fptt_irpf20a_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf20a"/>
        </record>
        <record id="fptt_irpf20a_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_sc"/>
        </record>
        <record id="fptt_irpf20a_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf20a"/>
        </record>
        <record id="fptt_irpf20a_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_bc"/>
        </record>
        <record id="fptt_irpf20a_4b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf20a"/>
        </record>
        <record id="fptt_irpf20a_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_sc"/>
        </record>
        <record id="fptt_irpf20a_4s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf20a"/>
        </record>
        <record id="fptt_irpf20asale_ex"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0"/>
        </record>
        <record id="fptt_irpf20asale_ex_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf20a"/>
        </record>
        <record id="fptt_irpf20a_0" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_bc"/>
        </record>
        <record id="fptt_irpf20a_0_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf20a"/>
        </record>

        <!-- Retención 21% arrendamientos -->

        <record id="fptt_irpf21asale_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21b"/>
        </record>
        <record id="fptt_irpf21asale_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf21a"/>
        </record>
        <record id="fptt_irpf21asale_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21isp"/>
        </record>
        <record id="fptt_irpf21asale_21isp_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf21a"/>
        </record>
        <record id="fptt_irpf21asale_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21s"/>
        </record>
        <record id="fptt_irpf21asale_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf21a"/>
        </record>
        <record id="fptt_irpf21asale_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10b"/>
        </record>
        <record id="fptt_irpf21asale_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf21a"/>
        </record>
        <record id="fptt_irpf21asale_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10s"/>
        </record>
        <record id="fptt_irpf21asale_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf21a"/>
        </record>
        <record id="fptt_irpf21asale_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4b"/>
        </record>
        <record id="fptt_irpf21asale_4b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf21a"/>
        </record>
        <record id="fptt_irpf21asale_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4s"/>
        </record>
        <record id="fptt_irpf21asale_4s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf21a"/>
        </record>
        <record id="fptt_irpf21a_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_bc"/>
        </record>
        <record id="fptt_irpf21a_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf21a"/>
        </record>
        <record id="fptt_irpf21a_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_sc"/>
        </record>
        <record id="fptt_irpf21a_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf21a"/>
        </record>
        <record id="fptt_irpf21a_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_bc"/>
        </record>
        <record id="fptt_irpf21a_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf21a"/>
        </record>
        <record id="fptt_irpf21a_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_sc"/>
        </record>
        <record id="fptt_irpf21a_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf21a"/>
        </record>
        <record id="fptt_irpf21a_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_bc"/>
        </record>
        <record id="fptt_irpf21a_4b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf21a"/>
        </record>
        <record id="fptt_irpf21a_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_sc"/>
        </record>
        <record id="fptt_irpf21a_4s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf21a"/>
        </record>
        <record id="fptt_irpf21asale_ex"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0"/>
        </record>
        <record id="fptt_irpf21asale_ex_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf21a"/>
        </record>
        <record id="fptt_irpf21a_0" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_bc"/>
        </record>
        <record id="fptt_irpf21a_0_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21a"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf21a"/>
        </record>

        <!-- Retenciones IRPF 21% -->

        <record id="fptt_irpf21sale_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21b"/>
        </record>
        <record id="fptt_irpf21sale_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf21"/>
        </record>
        <record id="fptt_irpf21sale_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21isp"/>
        </record>
        <record id="fptt_irpf21sale_21isp_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf21"/>
        </record>
        <record id="fptt_irpf21sale_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21s"/>
        </record>
        <record id="fptt_irpf21sale_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf21"/>
        </record>
        <record id="fptt_irpf21sale_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10b"/>
        </record>
        <record id="fptt_irpf21sale_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf21"/>
        </record>
        <record id="fptt_irpf21sale_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10s"/>
        </record>
        <record id="fptt_irpf21sale_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf21"/>
        </record>
        <record id="fptt_irpf21sale_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4b"/>
        </record>
        <record id="fptt_irpf21sale_4b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf21"/>
        </record>
        <record id="fptt_irpf21sale_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4s"/>
        </record>
        <record id="fptt_irpf21sale_4s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf21"/>
        </record>
        <record id="fptt_irpf21sale_ex"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0"/>
        </record>
        <record id="fptt_irpf21sale_ex_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf21"/>
        </record>
        <record id="fptt_irpf21sale_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_ns"/>
        </record>
        <record id="ptt_irpf21sale_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf21"/>
        </record>
        <record id="fptt_irpf21_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_bc"/>
        </record>
        <record id="fptt_irpf21_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf21p"/>
        </record>
        <record id="fptt_irpf21_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_sc"/>
        </record>
        <record id="fptt_irpf21_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf21p"/>
        </record>
        <record id="fptt_irpf21_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_bc"/>
        </record>
        <record id="fptt_irpf21_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf21p"/>
        </record>
        <record id="fptt_irpf21_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_sc"/>
        </record>
        <record id="fptt_irpf21_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf21p"/>
        </record>
        <record id="fptt_irpf21_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_bc"/>
        </record>
        <record id="fptt_irpf21_4b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf21p"/>
        </record>
        <record id="fptt_irpf21_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_sc"/>
        </record>
        <record id="fptt_irpf21_4s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf21p"/>
        </record>
        <record id="fptt_irpf21_0" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_bc"/>
        </record>
        <record id="fptt_irpf21_0_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf21p"/>
        </record>
        <record id="fptt_irpf21purchase_0_ns_b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns_b"/>
        </record>
        <record id="fptt_irpf21purchase_0_ns_b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf21p"/>
        </record>
        <record id="fptt_irpf21purchase_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns"/>
        </record>
        <record id="fptt_irpf21purchase_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf21"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf21p"/>
        </record>

        <!-- Retenciones IRPF 20% -->

        <record id="fptt_irpf20sale_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21b"/>
        </record>
        <record id="fptt_irpf20sale_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf20"/>
        </record>
        <record id="fptt_irpf20sale_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21isp"/>
        </record>
        <record id="fptt_irpf20sale_21isp_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf20"/>
        </record>
        <record id="fptt_irpf20sale_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21s"/>
        </record>
        <record id="fptt_irpf20sale_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf20"/>
        </record>
        <record id="fptt_irpf20sale_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10b"/>
        </record>
        <record id="fptt_irpf20sale_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf20"/>
        </record>
        <record id="fptt_irpf20sale_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10s"/>
        </record>
        <record id="fptt_irpf20sale_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf20"/>
        </record>
        <record id="fptt_irpf20sale_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4b"/>
        </record>
        <record id="fptt_irpf20sale_4b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf20"/>
        </record>
        <record id="fptt_irpf20sale_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4s"/>
        </record>
        <record id="fptt_irpf20sale_4s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf20"/>
        </record>
        <record id="fptt_irpf20sale_ex"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0"/>
        </record>
        <record id="fptt_irpf20sale_ex_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf20"/>
        </record>
        <record id="fptt_irpf20sale_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_ns"/>
        </record>
        <record id="ptt_irpf20sale_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf20"/>
        </record>
        <record id="fptt_irpf20_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_bc"/>
        </record>
        <record id="fptt_irpf20_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf20"/>
        </record>
        <record id="fptt_irpf20_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_sc"/>
        </record>
        <record id="fptt_irpf20_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf20"/>
        </record>
        <record id="fptt_irpf20_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_bc"/>
        </record>
        <record id="fptt_irpf20_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf20"/>
        </record>
        <record id="fptt_irpf20_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_sc"/>
        </record>
        <record id="fptt_irpf20_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf20"/>
        </record>
        <record id="fptt_irpf20_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_bc"/>
        </record>
        <record id="fptt_irpf20_4b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf20"/>
        </record>
        <record id="fptt_irpf20_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_sc"/>
        </record>
        <record id="fptt_irpf20_4s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf20"/>
        </record>
        <record id="fptt_irpf20_0" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_bc"/>
        </record>
        <record id="fptt_irpf20_0_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf20"/>
        </record>
        <record id="fptt_irpf20purchase_0_ns_b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns_b"/>
        </record>
        <record id="fptt_irpf20purchase_0_ns_b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf20"/>
        </record>
        <record id="fptt_irpf20purchase_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns"/>
        </record>
        <record id="fptt_irpf20purchase_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf20"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf20"/>
        </record>

        <!-- Retenciones IRPF 24% -->

        <record id="fptt_irpf24sale_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21b"/>
        </record>
        <record id="fptt_irpf24sale_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf24"/>
        </record>
        <record id="fptt_irpf24sale_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21isp"/>
        </record>
        <record id="fptt_irpf24sale_21isp_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf24"/>
        </record>
        <record id="fptt_irpf24sale_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21s"/>
        </record>
        <record id="fptt_irpf24sale_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf24"/>
        </record>
        <record id="fptt_irpf24sale_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10b"/>
        </record>
        <record id="fptt_irpf24sale_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf24"/>
        </record>
        <record id="fptt_irpf24sale_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10s"/>
        </record>
        <record id="fptt_irpf24sale_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf24"/>
        </record>
        <record id="fptt_irpf24sale_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4b"/>
        </record>
        <record id="fptt_irpf24sale_4b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf24"/>
        </record>
        <record id="fptt_irpf24sale_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4s"/>
        </record>
        <record id="fptt_irpf24sale_4s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf24"/>
        </record>
        <record id="fptt_irpf24sale_ex"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0"/>
        </record>
        <record id="fptt_irpf24sale_ex_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf24"/>
        </record>
        <record id="fptt_irpf24sale_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_ns"/>
        </record>
        <record id="ptt_irpf24sale_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf24"/>
        </record>
        <record id="fptt_irpf24_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_bc"/>
        </record>
        <record id="fptt_irpf24_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf24"/>
        </record>
        <record id="fptt_irpf24_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_sc"/>
        </record>
        <record id="fptt_irpf24_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf24"/>
        </record>
        <record id="fptt_irpf24_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_bc"/>
        </record>
        <record id="fptt_irpf24_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf24"/>
        </record>
        <record id="fptt_irpf24_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_sc"/>
        </record>
        <record id="fptt_irpf24_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf24"/>
        </record>
        <record id="fptt_irpf24_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_bc"/>
        </record>
        <record id="fptt_irpf24_4b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf24"/>
        </record>
        <record id="fptt_irpf24_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_sc"/>
        </record>
        <record id="fptt_irpf24_4s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf24"/>
        </record>
        <record id="fptt_irpf24_0" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_bc"/>
        </record>
        <record id="fptt_irpf24_0_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf24"/>
        </record>
        <record id="fptt_irpf24purchase_0_ns_b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns_b"/>
        </record>
        <record id="fptt_irpf24purchase_0_ns_b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf24"/>
        </record>
        <record id="fptt_irpf24purchase_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns"/>
        </record>
        <record id="fptt_irpf24purchase_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf24"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf24"/>
        </record>

        <!-- Retenciones IRPF 15% -->

        <record id="fptt_irpf15sale_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21b"/>
        </record>
        <record id="fptt_irpf15sale_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf15"/>
        </record>
        <record id="fptt_irpf15sale_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21isp"/>
        </record>
        <record id="fptt_irpf15sale_21isp_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf15"/>
        </record>
        <record id="fptt_irpf15sale_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21s"/>
        </record>
        <record id="fptt_irpf15sale_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf15"/>
        </record>
        <record id="fptt_irpf15sale_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10b"/>
        </record>
        <record id="fptt_irpf15sale_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf15"/>
        </record>
        <record id="fptt_irpf15sale_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10s"/>
        </record>
        <record id="fptt_irpf15sale_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf15"/>
        </record>
        <record id="fptt_irpf15sale_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4b"/>
        </record>
        <record id="fptt_irpf15sale_4b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf15"/>
        </record>
        <record id="fptt_irpf15sale_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4s"/>
        </record>
        <record id="fptt_irpf15sale_4s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf15"/>
        </record>
        <record id="fptt_irpf15sale_ex"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0"/>
        </record>
        <record id="fptt_irpf15sale_ex_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf15"/>
        </record>
        <record id="fptt_irpf15sale_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_ns"/>
        </record>
        <record id="ptt_irpf15sale_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf15"/>
        </record>
        <record id="fptt_irpf15_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_bc"/>
        </record>
        <record id="fptt_irpf15_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf15"/>
        </record>
        <record id="fptt_irpf15_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_sc"/>
        </record>
        <record id="fptt_irpf15_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf15"/>
        </record>
        <record id="fptt_irpf15_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_bc"/>
        </record>
        <record id="fptt_irpf15_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf15"/>
        </record>
        <record id="fptt_irpf15_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_sc"/>
        </record>
        <record id="fptt_irpf15_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf15"/>
        </record>
        <record id="fptt_irpf15_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_bc"/>
        </record>
        <record id="fptt_irpf15_4b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf15"/>
        </record>
        <record id="fptt_irpf15_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_sc"/>
        </record>
        <record id="fptt_irpf15_4s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf15"/>
        </record>
        <record id="fptt_irpf15_0" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_bc"/>
        </record>
        <record id="fptt_irpf15_0_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf15"/>
        </record>
        <record id="fptt_irpf15purchase_0_ns_b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns_b"/>
        </record>
        <record id="fptt_irpf15purchase_0_ns_b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf15"/>
        </record>
        <record id="fptt_irpf15purchase_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns"/>
        </record>
        <record id="fptt_irpf15purchase_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf15"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf15"/>
        </record>

        <!-- Retenciones IRPF 18% -->

        <record id="fptt_irpf18sale_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21b"/>
        </record>
        <record id="fptt_irpf18sale_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf18"/>
        </record>
        <record id="fptt_irpf18sale_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21isp"/>
        </record>
        <record id="fptt_irpf18sale_21isp_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf18"/>
        </record>
        <record id="fptt_irpf18sale_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21s"/>
        </record>
        <record id="fptt_irpf18sale_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf18"/>
        </record>
        <record id="fptt_irpf18sale_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10b"/>
        </record>
        <record id="fptt_irpf18sale_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf18"/>
        </record>
        <record id="fptt_irpf18sale_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10s"/>
        </record>
        <record id="fptt_irpf18sale_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf18"/>
        </record>
        <record id="fptt_irpf18sale_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4b"/>
        </record>
        <record id="fptt_irpf18sale_4b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf18"/>
        </record>
        <record id="fptt_irpf18sale_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4s"/>
        </record>
        <record id="fptt_irpf18sale_4s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf18"/>
        </record>
        <record id="fptt_irpf18sale_ex"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0"/>
        </record>
        <record id="fptt_irpf18sale_ex_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf18"/>
        </record>
        <record id="fptt_irpf18sale_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_ns"/>
        </record>
        <record id="ptt_irpf18sale_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf18"/>
        </record>
        <record id="fptt_irpf18_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_bc"/>
        </record>
        <record id="fptt_irpf18_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf18"/>
        </record>
        <record id="fptt_irpf18_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_sc"/>
        </record>
        <record id="fptt_irpf18_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf18"/>
        </record>
        <record id="fptt_irpf18_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_bc"/>
        </record>
        <record id="fptt_irpf18_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf18"/>
        </record>
        <record id="fptt_irpf18_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_sc"/>
        </record>
        <record id="fptt_irpf18_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf18"/>
        </record>
        <record id="fptt_irpf18_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_bc"/>
        </record>
        <record id="fptt_irpf18_4b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf18"/>
        </record>
        <record id="fptt_irpf18_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_sc"/>
        </record>
        <record id="fptt_irpf18_4s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf18"/>
        </record>
        <record id="fptt_irpf18_0" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_bc"/>
        </record>
        <record id="fptt_irpf18_0_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf18"/>
        </record>
        <record id="fptt_irpf18purchase_0_ns_b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns_b"/>
        </record>
        <record id="fptt_irpf18purchase_0_ns_b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf18"/>
        </record>
        <record id="fptt_irpf18purchase_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns"/>
        </record>
        <record id="fptt_irpf18purchase_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf18"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf18"/>
        </record>

        <!-- Retenciones IRPF 19% -->

        <record id="fptt_irpf19sale_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21b"/>
        </record>
        <record id="fptt_irpf19sale_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf19"/>
        </record>
        <record id="fptt_irpf19sale_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21isp"/>
        </record>
        <record id="fptt_irpf19sale_21isp_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf19"/>
        </record>
        <record id="fptt_irpf19sale_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21s"/>
        </record>
        <record id="fptt_irpf19sale_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf19"/>
        </record>
        <record id="fptt_irpf19sale_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10b"/>
        </record>
        <record id="fptt_irpf19sale_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf19"/>
        </record>
        <record id="fptt_irpf19sale_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10s"/>
        </record>
        <record id="fptt_irpf19sale_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf19"/>
        </record>
        <record id="fptt_irpf19sale_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4b"/>
        </record>
        <record id="fptt_irpf19sale_4b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf19"/>
        </record>
        <record id="fptt_irpf19sale_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4s"/>
        </record>
        <record id="fptt_irpf19sale_4s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf19"/>
        </record>
        <record id="fptt_irpf19sale_ex"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0"/>
        </record>
        <record id="fptt_irpf19sale_ex_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf19"/>
        </record>
        <record id="fptt_irpf19sale_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_ns"/>
        </record>
        <record id="ptt_irpf19sale_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf19"/>
        </record>
        <record id="fptt_irpf19_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_bc"/>
        </record>
        <record id="fptt_irpf19_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf19"/>
        </record>
        <record id="fptt_irpf19_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_sc"/>
        </record>
        <record id="fptt_irpf19_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf19"/>
        </record>
        <record id="fptt_irpf19_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_bc"/>
        </record>
        <record id="fptt_irpf19_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf19"/>
        </record>
        <record id="fptt_irpf19_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_sc"/>
        </record>
        <record id="fptt_irpf19_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf19"/>
        </record>
        <record id="fptt_irpf19_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_bc"/>
        </record>
        <record id="fptt_irpf19_4b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf19"/>
        </record>
        <record id="fptt_irpf19_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_sc"/>
        </record>
        <record id="fptt_irpf19_4s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf19"/>
        </record>
        <record id="fptt_irpf19_0" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_bc"/>
        </record>
        <record id="fptt_irpf19_0_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf19"/>
        </record>
        <record id="fptt_irpf19purchase_0_ns_b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns_b"/>
        </record>
        <record id="fptt_irpf19purchase_0_ns_b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf19"/>
        </record>
        <record id="fptt_irpf19purchase_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns"/>
        </record>
        <record id="fptt_irpf19purchase_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf19"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf19"/>
        </record>

        <!-- Retenciones IRPF 9% -->

        <record id="fptt_irpf9sale_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21b"/>
        </record>
        <record id="fptt_irpf9sale_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf9"/>
        </record>
        <record id="fptt_irpf9sale_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21isp"/>
        </record>
        <record id="fptt_irpf9sale_21isp_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf9"/>
        </record>
        <record id="fptt_irpf9sale_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21s"/>
        </record>
        <record id="fptt_irpf9sale_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf9"/>
        </record>
        <record id="fptt_irpf9sale_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10b"/>
        </record>
        <record id="fptt_irpf9sale_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf9"/>
        </record>
        <record id="fptt_irpf9sale_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10s"/>
        </record>
        <record id="fptt_irpf9sale_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf9"/>
        </record>
        <record id="fptt_irpf9sale_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4b"/>
        </record>
        <record id="fptt_irpf9sale_4b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf9"/>
        </record>
        <record id="fptt_irpf9sale_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4s"/>
        </record>
        <record id="fptt_irpf9sale_4s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf9"/>
        </record>
        <record id="fptt_irpf9sale_ex"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0"/>
        </record>
        <record id="fptt_irpf9sale_ex_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf9"/>
        </record>
        <record id="fptt_irpf9sale_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_ns"/>
        </record>
        <record id="ptt_irpf9sale_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf9"/>
        </record>
        <record id="fptt_irpf9_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_bc"/>
        </record>
        <record id="fptt_irpf9_21b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf9"/>
        </record>
        <record id="fptt_irpf9_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_sc"/>
        </record>
        <record id="fptt_irpf9_21s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf9"/>
        </record>
        <record id="fptt_irpf9_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_bc"/>
        </record>
        <record id="fptt_irpf9_10b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf9"/>
        </record>
        <record id="fptt_irpf9_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_sc"/>
        </record>
        <record id="fptt_irpf9_10s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf9"/>
        </record>
        <record id="fptt_irpf9_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_bc"/>
        </record>
        <record id="fptt_irpf9_4b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf9"/>
        </record>
        <record id="fptt_irpf9_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_sc"/>
        </record>
        <record id="fptt_irpf9_4s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf9"/>
        </record>
        <record id="fptt_irpf9_0" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_bc"/>
        </record>
        <record id="fptt_irpf9_0_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf9"/>
        </record>
        <record id="fptt_irpf9purchase_0_ns_b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns_b"/>
        </record>
        <record id="fptt_irpf9purchase_0_ns_b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf9"/>
        </record>
        <record id="fptt_irpf9purchase_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns"/>
        </record>
        <record id="fptt_irpf9purchase_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf9"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf9"/>
        </record>

        <!-- Retenciones IRPF 7% -->

        <record id="fptt_irpf7sale_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21b"/>
        </record>
        <record id="fptt_irpf7sale_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf7"/>
        </record>
        <record id="fptt_irpf7sale_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21isp"/>
        </record>
        <record id="fptt_irpf7sale_21isp_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf7"/>
        </record>
        <record id="fptt_irpf7sale_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21s"/>
        </record>
        <record id="fptt_irpf7sale_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf7"/>
        </record>
        <record id="fptt_irpf7sale_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10b"/>
        </record>
        <record id="fptt_irpf7sale_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf7"/>
        </record>
        <record id="fptt_irpf7sale_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10s"/>
        </record>
        <record id="fptt_irpf7sale_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf7"/>
        </record>
        <record id="fptt_irpf7sale_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4b"/>
        </record>
        <record id="fptt_irpf7sale_4b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf7"/>
        </record>
        <record id="fptt_irpf7sale_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4s"/>
        </record>
        <record id="fptt_irpf7sale_4s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf7"/>
        </record>
        <record id="fptt_irpf7sale_ex"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0"/>
        </record>
        <record id="fptt_irpf7sale_ex_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf7"/>
        </record>
        <record id="fptt_irpf7sale_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_ns"/>
        </record>
        <record id="ptt_irpf7sale_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf7"/>
        </record>
        <record id="fptt_irpf7_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_bc"/>
        </record>
        <record id="fptt_irpf7_21b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf7"/>
        </record>
        <record id="fptt_irpf7_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_sc"/>
        </record>
        <record id="fptt_irpf7_21s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf7"/>
        </record>
        <record id="fptt_irpf7_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_bc"/>
        </record>
        <record id="fptt_irpf7_10b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf7"/>
        </record>
        <record id="fptt_irpf7_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_sc"/>
        </record>
        <record id="fptt_irpf7_10s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf7"/>
        </record>
        <record id="fptt_irpf7_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_bc"/>
        </record>
        <record id="fptt_irpf7_4b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf7"/>
        </record>
        <record id="fptt_irpf7_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_sc"/>
        </record>
        <record id="fptt_irpf7_4s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf7"/>
        </record>
        <record id="fptt_irpf7_0" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_bc"/>
        </record>
        <record id="fptt_irpf7_0_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf7"/>
        </record>
        <record id="fptt_irpf7purchase_0_ns_b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns_b"/>
        </record>
        <record id="fptt_irpf7purchase_0_ns_b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf7"/>
        </record>
        <record id="fptt_irpf7purchase_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns"/>
        </record>
        <record id="fptt_irpf7purchase_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf7"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf7"/>
        </record>

        <!-- Retenciones IRPF 2% -->

        <record id="fptt_irpf2sale_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21b"/>
        </record>
        <record id="fptt_irpf2sale_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf2"/>
        </record>
        <record id="fptt_irpf2sale_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21isp"/>
        </record>
        <record id="fptt_irpf2sale_21isp_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf2"/>
        </record>
        <record id="fptt_irpf2sale_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21s"/>
        </record>
        <record id="fptt_irpf2sale_21s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf2"/>
        </record>
        <record id="fptt_irpf2sale_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10b"/>
        </record>
        <record id="fptt_irpf2sale_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf2"/>
        </record>
        <record id="fptt_irpf2sale_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10s"/>
        </record>
        <record id="fptt_irpf2sale_10_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf2"/>
        </record>
        <record id="fptt_irpf2sale_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4b"/>
        </record>
        <record id="fptt_irpf2sale_4b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf2"/>
        </record>
        <record id="fptt_irpf2sale_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4s"/>
        </record>
        <record id="fptt_irpf2sale_4s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf2"/>
        </record>
        <record id="fptt_irpf2sale_ex"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0"/>
        </record>
        <record id="fptt_irpf2sale_ex_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf2"/>
        </record>
        <record id="fptt_irpf2sale_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_ns"/>
        </record>
        <record id="ptt_irpf2sale_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf2"/>
        </record>
        <record id="fptt_irpf2_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_bc"/>
        </record>
        <record id="fptt_irpf2_21b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf2"/>
        </record>
        <record id="fptt_irpf2_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_sc"/>
        </record>
        <record id="fptt_irpf2_21s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf2"/>
        </record>
        <record id="fptt_irpf2_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_bc"/>
        </record>
        <record id="fptt_irpf2_10b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf2"/>
        </record>
        <record id="fptt_irpf2_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_sc"/>
        </record>
        <record id="fptt_irpf2_10s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf2"/>
        </record>
        <record id="fptt_irpf2_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_bc"/>
        </record>
        <record id="fptt_irpf2_4b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf2"/>
        </record>
        <record id="fptt_irpf2_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_sc"/>
        </record>
        <record id="fptt_irpf2_4s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf2"/>
        </record>
        <record id="fptt_irpf2_0" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_bc"/>
        </record>
        <record id="fptt_irpf2_0_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf2"/>
        </record>
        <record id="fptt_irpf2purchase_0_ns_b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns_b"/>
        </record>
        <record id="fptt_irpf2purchase_0_ns_b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf2"/>
        </record>
        <record id="fptt_irpf2purchase_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns"/>
        </record>
        <record id="fptt_irpf2purchase_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf2"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf2"/>
        </record>

        <!-- Retenciones IRPF 1% -->

        <record id="fptt_irpf1sale_21b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21b"/>
        </record>
        <record id="fptt_irpf1sale_21b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf1"/>
        </record>
        <record id="fptt_irpf1sale_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21isp"/>
        </record>
        <record id="fptt_irpf1sale_21isp_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf1"/>
        </record>
        <record id="fptt_irpf1sale_21s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva21s"/>
        </record>
        <record id="fptt_irpf1sale_21_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf1"/>
        </record>
        <record id="fptt_irpf1sale_10b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10b"/>
        </record>
        <record id="fptt_irpf1sale_10b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf1"/>
        </record>
        <record id="fptt_irpf1sale_10s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva10s"/>
        </record>
        <record id="fptt_irpf1sale_10s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf1"/>
        </record>
        <record id="fptt_irpf1sale_4b"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4b"/>
        </record>
        <record id="fptt_irpf1sale_4b_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf1"/>
        </record>
        <record id="fptt_irpf1sale_4s"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva4s"/>
        </record>
        <record id="fptt_irpf1sale_4s_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf1"/>
        </record>
        <record id="fptt_irpf1sale_ex"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0"/>
        </record>
        <record id="fptt_irpf1sale_ex_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf1"/>
        </record>
        <record id="fptt_irpf1sale_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_ns"/>
        </record>
        <record id="ptt_irpf1sale_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_s_irpf1"/>
        </record>
        <record id="fptt_irpf1_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_bc"/>
        </record>
        <record id="fptt_irpf1_21b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf1"/>
        </record>
        <record id="fptt_irpf1_21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_sc"/>
        </record>
        <record id="fptt_irpf1_21s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf1"/>
        </record>
        <record id="fptt_irpf1_10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_bc"/>
        </record>
        <record id="fptt_irpf1_10b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf1"/>
        </record>
        <record id="fptt_irpf1_10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_sc"/>
        </record>
        <record id="fptt_irpf1_10s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf1"/>
        </record>
        <record id="fptt_irpf1_4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_bc"/>
        </record>
        <record id="fptt_irpf1_4b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf1"/>
        </record>
        <record id="fptt_irpf1_4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_sc"/>
        </record>
        <record id="fptt_irpf1_4s_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf1"/>
        </record>
        <record id="fptt_irpf1_0" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_bc"/>
        </record>
        <record id="fptt_irpf1_0_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf1"/>
        </record>
        <record id="fptt_irpf1purchase_0_ns_b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns_b"/>
        </record>
        <record id="fptt_irpf1purchase_0_ns_b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns_b"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf1"/>
        </record>
        <record id="fptt_irpf1purchase_0_ns"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva0_ns"/>
        </record>
        <record id="fptt_irpf1purchase_0_ns_2"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_irpf1"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva0_ns"/>
            <field name="tax_dest_id" ref="account_tax_template_p_irpf1"/>
        </record>

        <!-- Inversion del Sujeto Pasivo Nacional -->

        <record id="fptt_ispn_p_iva4_bc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_ispn"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_isp"/>
        </record>
        <record id="fptt_ispn_p_iva4_sc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_ispn"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_isp"/>
        </record>
        <record id="fptt_ispn_p_iva4_bi" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_ispn"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva4_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva4_isp_bi"/>
        </record>
        <record id="fptt_ispn_p_iva10_bc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_ispn"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_isp"/>
        </record>
        <record id="fptt_ispn_p_iva10_sc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_ispn"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_isp"/>
        </record>
        <record id="fptt_ispn_p_iva10_bi" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_ispn"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva10_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva10_isp_bi"/>
        </record>
        <record id="fptt_ispn_p_iva21_bc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_ispn"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_isp"/>
        </record>
        <record id="fptt_ispn_p_iva21_sc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_ispn"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_sc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_isp"/>
        </record>
        <record id="fptt_ispn_p_iva21_bi" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_ispn"/>
            <field name="tax_src_id" ref="account_tax_template_p_iva21_bi"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva21_isp_bi"/>
        </record>
        <record id="fptt_ispn_s_iva4b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_ispn"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_isp"/>
        </record>
        <record id="fptt_ispn_s_iva4s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_ispn"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva4s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_isp"/>
        </record>
        <record id="fptt_ispn_s_iva10b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_ispn"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_isp"/>
        </record>
        <record id="fptt_ispn_s_iva10s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_ispn"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva10s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_isp"/>
        </record>
        <record id="fptt_ispn_s_iva21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_ispn"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21b"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_isp"/>
        </record>
        <record id="fptt_ispn_s_iva21s" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_ispn"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21s"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_isp"/>
        </record>
        <record id="fptt_ispn_s_21isp"
            model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_ispn"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_isp"/>
        </record>

        <!-- Revendedor inversion sujeto pasivo -->
        <record id="fptt_rnrisp_s_iva21isp" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_rnrisp"/>
            <field name="tax_src_id" ref="account_tax_template_s_iva21isp"/>
            <field name="tax_dest_id" ref="account_tax_template_s_iva0_isp"/>
        </record>

        <!-- Régimen especial de Agricultura, Ganaderia y Pesca -->
        <record id="fptt_reagyp_a_4b_1" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_reagyp_a"/>
            <field name="tax_src_id" ref="l10n_es.account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="l10n_es.account_tax_template_p_iva12_agr"/>
        </record>
        <record id="fptt_reagyp_a_4b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_reagyp_a"/>
            <field name="tax_src_id" ref="l10n_es.account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="l10n_es.account_tax_template_p_irpf2"/>
        </record>
        <record id="fptt_reagyp_gp_4b_1" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_reagyp_gp"/>
            <field name="tax_src_id" ref="l10n_es.account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="account_tax_template_p_iva105_gan"/>
        </record>
        <record id="fptt_reagyp_gp_4b_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_reagyp_gp"/>
            <field name="tax_src_id" ref="l10n_es.account_tax_template_p_iva4_bc"/>
            <field name="tax_dest_id" ref="l10n_es.account_tax_template_p_irpf2"/>
        </record>

        <!-- ************************************************************* -->
        <!-- Fiscal Position Account Templates -->
        <!-- ************************************************************* -->

        <!-- Extracomunitarios -->

        <record id="fpat_extra_acc_1"
            model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="account_src_id" ref="l10n_es.account_common_7000"/>
            <field name="account_dest_id" ref="l10n_es.account_common_7002"/>
        </record>
        <record id="fpat_extra_acc_2"
            model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="account_src_id" ref="l10n_es.account_common_7010"/>
            <field name="account_dest_id" ref="l10n_es.account_common_7012"/>
        </record>
        <record id="fpat_extra_acc_3"
            model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="account_src_id" ref="l10n_es.account_common_7020"/>
            <field name="account_dest_id" ref="l10n_es.account_common_7022"/>
        </record>
        <record id="fpat_extra_acc_4"
            model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="account_src_id" ref="l10n_es.account_common_7030"/>
            <field name="account_dest_id" ref="l10n_es.account_common_7032"/>
        </record>
        <record id="fpat_extra_acc_5"
            model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="account_src_id" ref="l10n_es.account_common_7040"/>
            <field name="account_dest_id" ref="l10n_es.account_common_7042"/>
        </record>
        <record id="fpat_extra_acc_6"
            model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_extra"/>
            <field name="account_src_id" ref="l10n_es.account_common_7050"/>
            <field name="account_dest_id" ref="l10n_es.account_common_7052"/>
        </record>

        <!-- Intracomunitarios -->

        <record id="fpat_intra_acc_1"
            model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="account_src_id" ref="l10n_es.account_common_7000"/>
            <field name="account_dest_id" ref="l10n_es.account_common_7001"/>
        </record>
        <record id="fpat_intra_acc_2"
            model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="account_src_id" ref="l10n_es.account_common_7010"/>
            <field name="account_dest_id" ref="l10n_es.account_common_7011"/>
        </record>
        <record id="fpat_intra_acc_3"
            model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="account_src_id" ref="l10n_es.account_common_7020"/>
            <field name="account_dest_id" ref="l10n_es.account_common_7021"/>
        </record>
        <record id="fpat_intra_acc_4"
            model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="account_src_id" ref="l10n_es.account_common_7030"/>
            <field name="account_dest_id" ref="l10n_es.account_common_7031"/>
        </record>
        <record id="fpat_intra_acc_5"
            model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="account_src_id" ref="l10n_es.account_common_7040"/>
            <field name="account_dest_id" ref="l10n_es.account_common_7041"/>
        </record>
        <record id="fpat_intra_acc_6"
            model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_intra"/>
            <field name="account_src_id" ref="l10n_es.account_common_7050"/>
            <field name="account_dest_id" ref="l10n_es.account_common_7051"/>
        </record>

    </data>
</odoo>

```

## File: data\account_group.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
    <record id="account_group_1" model="account.group">
        <field name="code_prefix">1</field>
        <field name="name">Financiación Básica</field>
    </record>        
    <record id="account_group_10" model="account.group">
        <field name="code_prefix">10</field>
        <field name="name">Capital</field>
        <field name="parent_id" ref="l10n_es.account_group_1"/>
    </record>
    <record id="account_group_100" model="account.group">
        <field name="code_prefix">100</field>
        <field name="name">Capital social</field>
        <field name="parent_id" ref="l10n_es.account_group_10"/>
    </record>
    <record id="account_group_101" model="account.group">
        <field name="code_prefix">101</field>
        <field name="name">Fondo social</field>
        <field name="parent_id" ref="l10n_es.account_group_10"/>
    </record>
    <record id="account_group_102" model="account.group">
        <field name="code_prefix">102</field>
        <field name="name">Capital</field>
        <field name="parent_id" ref="l10n_es.account_group_10"/>
    </record>
    <record id="account_group_103" model="account.group">
        <field name="code_prefix">103</field>
        <field name="name">Socios por desembolsos no exigidos</field>
        <field name="parent_id" ref="l10n_es.account_group_10"/>
    </record>
    <record id="account_group_1030" model="account.group">
        <field name="code_prefix">1030</field>
        <field name="name">Socios por desembolsos no exigidos, capital social</field>
        <field name="parent_id" ref="l10n_es.account_group_103"/>
    </record>
    <record id="account_group_1034" model="account.group">
        <field name="code_prefix">1034</field>
        <field name="name">Socios por desembolsos no exigidos, capital pendiente de inscripción</field>
        <field name="parent_id" ref="l10n_es.account_group_103"/>
    </record>
    <record id="account_group_104" model="account.group">
        <field name="code_prefix">104</field>
        <field name="name">Socios por aportaciones no dinerarias pendientes</field>
        <field name="parent_id" ref="l10n_es.account_group_10"/>
    </record>
    <record id="account_group_1040" model="account.group">
        <field name="code_prefix">1040</field>
        <field name="name">Socios por aportaciones no dinerarias pendientes, capital social</field>
        <field name="parent_id" ref="l10n_es.account_group_104"/>
    </record>
    <record id="account_group_1044" model="account.group">
        <field name="code_prefix">1044</field>
        <field name="name">Socios por aportaciones no dinerarias pendientes, capital pendiente de inscripción</field>
        <field name="parent_id" ref="l10n_es.account_group_104"/>
    </record>
    <record id="account_group_108" model="account.group">
        <field name="code_prefix">108</field>
        <field name="name">Acciones o participaciones propias en situaciones especiales</field>
        <field name="parent_id" ref="l10n_es.account_group_10"/>
    </record>
    <record id="account_group_109" model="account.group">
        <field name="code_prefix">109</field>
        <field name="name">Acciones o participaciones propias para reducción de capital</field>
        <field name="parent_id" ref="l10n_es.account_group_10"/>
    </record>
    <record id="account_group_11" model="account.group">
        <field name="code_prefix">11</field>
        <field name="name">Reservas y otros instrumentos de patrimonio</field>
        <field name="parent_id" ref="l10n_es.account_group_1"/>
    </record>
    <record id="account_group_110" model="account.group">
        <field name="code_prefix">110</field>
        <field name="name">Prima de emisión o asunción</field>
        <field name="parent_id" ref="l10n_es.account_group_11"/>
    </record>
    <record id="account_group_111" model="account.group">
        <field name="code_prefix">111</field>
        <field name="name">Otros instrumentos de patrimonio neto</field>
        <field name="parent_id" ref="l10n_es.account_group_11"/>
    </record>
    <record id="account_group_1110" model="account.group">
        <field name="code_prefix">1110</field>
        <field name="name">Patrimonio neto por emisión de instrumentos financieros compuestos</field>
        <field name="parent_id" ref="l10n_es.account_group_111"/>
    </record>
    <record id="account_group_1111" model="account.group">
        <field name="code_prefix">1111</field>
        <field name="name">Resto de instrumentos de patrimonio neto</field>
        <field name="parent_id" ref="l10n_es.account_group_111"/>
    </record>
    <record id="account_group_112" model="account.group">
        <field name="code_prefix">112</field>
        <field name="name">Reserva legal</field>
        <field name="parent_id" ref="l10n_es.account_group_11"/>
    </record>
    <record id="account_group_113" model="account.group">
        <field name="code_prefix">113</field>
        <field name="name">Reservas voluntarias</field>
        <field name="parent_id" ref="l10n_es.account_group_11"/>
    </record>
    <record id="account_group_114" model="account.group">
        <field name="code_prefix">114</field>
        <field name="name">Reservas especiales</field>
        <field name="parent_id" ref="l10n_es.account_group_11"/>
    </record>
    <record id="account_group_1140" model="account.group">
        <field name="code_prefix">1140</field>
        <field name="name">Reservas para acciones o participaciones de la sociedad dominante</field>
        <field name="parent_id" ref="l10n_es.account_group_114"/>
    </record>
    <record id="account_group_1141" model="account.group">
        <field name="code_prefix">1141</field>
        <field name="name">Reservas estatutarias</field>
        <field name="parent_id" ref="l10n_es.account_group_114"/>
    </record>
    <record id="account_group_1142" model="account.group">
        <field name="code_prefix">1142</field>
        <field name="name">Reserva por capital amortizado</field>
        <field name="parent_id" ref="l10n_es.account_group_114"/>
    </record>
    <record id="account_group_1143" model="account.group">
        <field name="code_prefix">1143</field>
        <field name="name">Reserva por fondo de comercio</field>
        <field name="parent_id" ref="l10n_es.account_group_114"/>
    </record>
    <record id="account_group_1144" model="account.group">
        <field name="code_prefix">1144</field>
        <field name="name">Reservas por acciones propias aceptadas en garantía</field>
        <field name="parent_id" ref="l10n_es.account_group_114"/>
    </record>
    <record id="account_group_115" model="account.group">
        <field name="code_prefix">115</field>
        <field name="name">Reservas por pérdidas y ganancias actuariales y otros ajustes</field>
        <field name="parent_id" ref="l10n_es.account_group_11"/>
    </record>
    <record id="account_group_118" model="account.group">
        <field name="code_prefix">118</field>
        <field name="name">Aportaciones de socios o propietarios</field>
        <field name="parent_id" ref="l10n_es.account_group_11"/>
    </record>
    <record id="account_group_119" model="account.group">
        <field name="code_prefix">119</field>
        <field name="name">Diferencias por ajuste del capital a euros</field>
        <field name="parent_id" ref="l10n_es.account_group_11"/>
    </record>
    <record id="account_group_12" model="account.group">
        <field name="code_prefix">12</field>
        <field name="name">Resultados pendientes de aplicación</field>
        <field name="parent_id" ref="l10n_es.account_group_1"/>
    </record>
    <record id="account_group_120" model="account.group">
        <field name="code_prefix">120</field>
        <field name="name">Remanente</field>
        <field name="parent_id" ref="l10n_es.account_group_12"/>
    </record>
    <record id="account_group_121" model="account.group">
        <field name="code_prefix">121</field>
        <field name="name">Resultados negativos de ejercicios anteriores</field>
        <field name="parent_id" ref="l10n_es.account_group_12"/>
    </record>
    <record id="account_group_129" model="account.group">
        <field name="code_prefix">129</field>
        <field name="name">Resultado del ejercicio</field>
        <field name="parent_id" ref="l10n_es.account_group_12"/>
    </record>
    <record id="account_group_13" model="account.group">
        <field name="code_prefix">13</field>
        <field name="name">Subvenciones, donaciones y ajustes por cambios de valor</field>
        <field name="parent_id" ref="l10n_es.account_group_1"/>
    </record>
    <record id="account_group_130" model="account.group">
        <field name="code_prefix">130</field>
        <field name="name">Subvenciones oficiales de capital</field>
        <field name="parent_id" ref="l10n_es.account_group_13"/>
    </record>
    <record id="account_group_131" model="account.group">
        <field name="code_prefix">131</field>
        <field name="name">Donaciones y legados de capital</field>
        <field name="parent_id" ref="l10n_es.account_group_13"/>
    </record>
    <record id="account_group_132" model="account.group">
        <field name="code_prefix">132</field>
        <field name="name">Otras subvenciones, donaciones y legados</field>
        <field name="parent_id" ref="l10n_es.account_group_13"/>
    </record>
    <record id="account_group_133" model="account.group">
        <field name="code_prefix">133</field>
        <field name="name">Ajustes por valoración en activos financieros disponibles para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_13"/>
    </record>
    <record id="account_group_134" model="account.group">
        <field name="code_prefix">134</field>
        <field name="name">Operaciones de cobertura</field>
        <field name="parent_id" ref="l10n_es.account_group_13"/>
    </record>
    <record id="account_group_1340" model="account.group">
        <field name="code_prefix">1340</field>
        <field name="name">Cobertura de flujos de efectivo</field>
        <field name="parent_id" ref="l10n_es.account_group_134"/>
    </record>
    <record id="account_group_1341" model="account.group">
        <field name="code_prefix">1341</field>
        <field name="name">Cobertura de una inversión neta en un negocio en el extranjero</field>
        <field name="parent_id" ref="l10n_es.account_group_134"/>
    </record>
    <record id="account_group_135" model="account.group">
        <field name="code_prefix">135</field>
        <field name="name">Diferencias de conversión</field>
        <field name="parent_id" ref="l10n_es.account_group_13"/>
    </record>
    <record id="account_group_136" model="account.group">
        <field name="code_prefix">136</field>
        <field name="name">Ajustes por valoración en activos no corrientes y grupos enajenables de elementos, mantenidos para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_13"/>
    </record>
    <record id="account_group_137" model="account.group">
        <field name="code_prefix">137</field>
        <field name="name">Ingresos fiscales a distribuir en varios ejercicios</field>
        <field name="parent_id" ref="l10n_es.account_group_13"/>
    </record>
    <record id="account_group_1370" model="account.group">
        <field name="code_prefix">1370</field>
        <field name="name">Ingresos fiscales por diferencias permanentes a distribuir en varios ejercicios</field>
        <field name="parent_id" ref="l10n_es.account_group_137"/>
    </record>
    <record id="account_group_1371" model="account.group">
        <field name="code_prefix">1371</field>
        <field name="name">Ingresos fiscales por deducciones y bonificaciones a distribuir en varios ejercicios</field>
        <field name="parent_id" ref="l10n_es.account_group_137"/>
    </record>
    <record id="account_group_14" model="account.group">
        <field name="code_prefix">14</field>
        <field name="name">Provisiones</field>
        <field name="parent_id" ref="l10n_es.account_group_1"/>
    </record>
    <record id="account_group_140" model="account.group">
        <field name="code_prefix">140</field>
        <field name="name">Provisión por retribuciones a largo plazo al personal</field>
        <field name="parent_id" ref="l10n_es.account_group_14"/>
    </record>
    <record id="account_group_141" model="account.group">
        <field name="code_prefix">141</field>
        <field name="name">Provisión para impuestos</field>
        <field name="parent_id" ref="l10n_es.account_group_14"/>
    </record>
    <record id="account_group_142" model="account.group">
        <field name="code_prefix">142</field>
        <field name="name">Provisión para otras responsabilidades</field>
        <field name="parent_id" ref="l10n_es.account_group_14"/>
    </record>
    <record id="account_group_143" model="account.group">
        <field name="code_prefix">143</field>
        <field name="name">Provisión por desmantelamiento, retiro o rehabilitación del inmovilizado</field>
        <field name="parent_id" ref="l10n_es.account_group_14"/>
    </record>
    <record id="account_group_145" model="account.group">
        <field name="code_prefix">145</field>
        <field name="name">Provisión para actuaciones medioambientales</field>
        <field name="parent_id" ref="l10n_es.account_group_14"/>
    </record>
    <record id="account_group_146" model="account.group">
        <field name="code_prefix">146</field>
        <field name="name">Provisión para reestructuraciones</field>
        <field name="parent_id" ref="l10n_es.account_group_14"/>
    </record>
    <record id="account_group_147" model="account.group">
        <field name="code_prefix">147</field>
        <field name="name">Provisión por transacciones con pagos basados en instrumentos de patrimonio</field>
        <field name="parent_id" ref="l10n_es.account_group_14"/>
    </record>
    <record id="account_group_15" model="account.group">
        <field name="code_prefix">15</field>
        <field name="name">Deudas a largo plazo con características especiales</field>
        <field name="parent_id" ref="l10n_es.account_group_1"/>
    </record>
    <record id="account_group_150" model="account.group">
        <field name="code_prefix">150</field>
        <field name="name">Acciones o participaciones a largo plazo consideradas como pasivos financieros</field>
        <field name="parent_id" ref="l10n_es.account_group_15"/>
    </record>
    <record id="account_group_153" model="account.group">
        <field name="code_prefix">153</field>
        <field name="name">Desembolsos no exigidos por acciones o participaciones consideradas como pasivos financieros</field>
        <field name="parent_id" ref="l10n_es.account_group_15"/>
    </record>
    <record id="account_group_1533" model="account.group">
        <field name="code_prefix">1533</field>
        <field name="name">Desembolsos no exigidos, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_153"/>
    </record>
    <record id="account_group_1534" model="account.group">
        <field name="code_prefix">1534</field>
        <field name="name">Desembolsos no exigidos, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_153"/>
    </record>
    <record id="account_group_1535" model="account.group">
        <field name="code_prefix">1535</field>
        <field name="name">Desembolsos no exigidos, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_153"/>
    </record>
    <record id="account_group_1536" model="account.group">
        <field name="code_prefix">1536</field>
        <field name="name">Otros desembolsos no exigidos</field>
        <field name="parent_id" ref="l10n_es.account_group_153"/>
    </record>
    <record id="account_group_154" model="account.group">
        <field name="code_prefix">154</field>
        <field name="name">Aportaciones no dinerarias pendientes por acciones o participaciones consideradas como pasivos financieros</field>
        <field name="parent_id" ref="l10n_es.account_group_15"/>
    </record>
    <record id="account_group_1543" model="account.group">
        <field name="code_prefix">1543</field>
        <field name="name">Aportaciones no dinerarias pendientes, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_154"/>
    </record>
    <record id="account_group_1544" model="account.group">
        <field name="code_prefix">1544</field>
        <field name="name">Aportaciones no dinerarias pendientes, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_154"/>
    </record>
    <record id="account_group_1545" model="account.group">
        <field name="code_prefix">1545</field>
        <field name="name">Aportaciones no dinerarias pendientes, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_154"/>
    </record>
    <record id="account_group_1546" model="account.group">
        <field name="code_prefix">1546</field>
        <field name="name">Otras aportaciones no dinerarias pendientes</field>
        <field name="parent_id" ref="l10n_es.account_group_154"/>
    </record>
    <record id="account_group_16" model="account.group">
        <field name="code_prefix">16</field>
        <field name="name">Deudas a largo plazo con partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_1"/>
    </record>
    <record id="account_group_160" model="account.group">
        <field name="code_prefix">160</field>
        <field name="name">Deudas a largo plazo con entidades de crédito vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_16"/>
    </record>
    <record id="account_group_1603" model="account.group">
        <field name="code_prefix">1603</field>
        <field name="name">Deudas a largo plazo con entidades de crédito, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_160"/>
    </record>
    <record id="account_group_1604" model="account.group">
        <field name="code_prefix">1604</field>
        <field name="name">Deudas a largo plazo con entidades de crédito, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_160"/>
    </record>
    <record id="account_group_1605" model="account.group">
        <field name="code_prefix">1605</field>
        <field name="name">Deudas a largo plazo con otras entidades de crédito vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_160"/>
    </record>
    <record id="account_group_161" model="account.group">
        <field name="code_prefix">161</field>
        <field name="name">Proveedores de inmovilizado a largo plazo, partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_16"/>
    </record>
    <record id="account_group_1613" model="account.group">
        <field name="code_prefix">1613</field>
        <field name="name">Proveedores de inmovilizado a largo plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_161"/>
    </record>
    <record id="account_group_1614" model="account.group">
        <field name="code_prefix">1614</field>
        <field name="name">Proveedores de inmovilizado a largo plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_161"/>
    </record>
    <record id="account_group_1615" model="account.group">
        <field name="code_prefix">1615</field>
        <field name="name">Proveedores de inmovilizado a largo plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_161"/>
    </record>
    <record id="account_group_162" model="account.group">
        <field name="code_prefix">162</field>
        <field name="name">Acreedores por arrendamiento financiero a largo plazo, partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_16"/>
    </record>
    <record id="account_group_1623" model="account.group">
        <field name="code_prefix">1623</field>
        <field name="name">Acreedores por arrendamiento financiero a largo plazo, empresas de grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_162"/>
    </record>
    <record id="account_group_1624" model="account.group">
        <field name="code_prefix">1624</field>
        <field name="name">Acreedores por arrendamiento financiero a largo plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_162"/>
    </record>
    <record id="account_group_1625" model="account.group">
        <field name="code_prefix">1625</field>
        <field name="name">Acreedores por arrendamiento financiero a largo plazo, otras partes vinculadas.</field>
        <field name="parent_id" ref="l10n_es.account_group_162"/>
    </record>
    <record id="account_group_163" model="account.group">
        <field name="code_prefix">163</field>
        <field name="name">Otras deudas a largo plazo con partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_16"/>
    </record>
    <record id="account_group_1633" model="account.group">
        <field name="code_prefix">1633</field>
        <field name="name">Otras deudas a largo plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_163"/>
    </record>
    <record id="account_group_1634" model="account.group">
        <field name="code_prefix">1634</field>
        <field name="name">Otras deudas a largo plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_163"/>
    </record>
    <record id="account_group_1635" model="account.group">
        <field name="code_prefix">1635</field>
        <field name="name">Otras deudas a largo plazo, con otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_163"/>
    </record>
    <record id="account_group_17" model="account.group">
        <field name="code_prefix">17</field>
        <field name="name">Deudas a largo plazo por préstamos recibidos, empréstitos y otros conceptos</field>
        <field name="parent_id" ref="l10n_es.account_group_1"/>
    </record>
    <record id="account_group_170" model="account.group">
        <field name="code_prefix">170</field>
        <field name="name">Deudas a largo plazo con entidades de crédito</field>
        <field name="parent_id" ref="l10n_es.account_group_17"/>
    </record>
    <record id="account_group_171" model="account.group">
        <field name="code_prefix">171</field>
        <field name="name">Deudas a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_17"/>
    </record>
    <record id="account_group_172" model="account.group">
        <field name="code_prefix">172</field>
        <field name="name">Deudas a largo plazo transformables en subvenciones, donaciones y legados</field>
        <field name="parent_id" ref="l10n_es.account_group_17"/>
    </record>
    <record id="account_group_173" model="account.group">
        <field name="code_prefix">173</field>
        <field name="name">Proveedores de inmovilizado a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_17"/>
    </record>
    <record id="account_group_174" model="account.group">
        <field name="code_prefix">174</field>
        <field name="name">Acreedores por arrendamiento financiero a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_17"/>
    </record>
    <record id="account_group_175" model="account.group">
        <field name="code_prefix">175</field>
        <field name="name">Efectos a pagar a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_17"/>
    </record>
    <record id="account_group_176" model="account.group">
        <field name="code_prefix">176</field>
        <field name="name">Pasivos por derivados financieros a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_17"/>
    </record>
    <record id="account_group_1760" model="account.group">
        <field name="code_prefix">1760</field>
        <field name="name">Pasivos por derivados financieros</field>
        <field name="parent_id" ref="l10n_es.account_group_176"/>
    </record>
    <record id="account_group_1765" model="account.group">
        <field name="code_prefix">1765</field>
        <field name="name">Pasivos por derivados financieros a largo plazo, cartera de negociación</field>
        <field name="parent_id" ref="l10n_es.account_group_176"/>
    </record>
    <record id="account_group_1768" model="account.group">
        <field name="code_prefix">1768</field>
        <field name="name">Pasivos por derivados financieros a largo plazo, instrumentos de cobertura</field>
        <field name="parent_id" ref="l10n_es.account_group_176"/>
    </record>
    <record id="account_group_177" model="account.group">
        <field name="code_prefix">177</field>
        <field name="name">Obligaciones y bonos</field>
        <field name="parent_id" ref="l10n_es.account_group_17"/>
    </record>
    <record id="account_group_178" model="account.group">
        <field name="code_prefix">178</field>
        <field name="name">Obligaciones y bonos convertibles</field>
        <field name="parent_id" ref="l10n_es.account_group_17"/>
    </record>
    <record id="account_group_179" model="account.group">
        <field name="code_prefix">179</field>
        <field name="name">Deudas representadas en otros valores negociables</field>
        <field name="parent_id" ref="l10n_es.account_group_17"/>
    </record>
    <record id="account_group_18" model="account.group">
        <field name="code_prefix">18</field>
        <field name="name">Pasivos por fianzas, garantías y otros conceptos a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_1"/>
    </record>
    <record id="account_group_180" model="account.group">
        <field name="code_prefix">180</field>
        <field name="name">Fianzas recibidas a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_18"/>
    </record>
    <record id="account_group_181" model="account.group">
        <field name="code_prefix">181</field>
        <field name="name">Anticipos recibidos por ventas o prestaciones de servicios a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_18"/>
    </record>
    <record id="account_group_185" model="account.group">
        <field name="code_prefix">185</field>
        <field name="name">Depósitos recibidos a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_18"/>
    </record>
    <record id="account_group_189" model="account.group">
        <field name="code_prefix">189</field>
        <field name="name">Garantías financieras a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_18"/>
    </record>
    <record id="account_group_19" model="account.group">
        <field name="code_prefix">19</field>
        <field name="name">Situaciones transitorias de financiación</field>
        <field name="parent_id" ref="l10n_es.account_group_1"/>
    </record>
    <record id="account_group_190" model="account.group">
        <field name="code_prefix">190</field>
        <field name="name">Acciones o participaciones emitidas</field>
        <field name="parent_id" ref="l10n_es.account_group_19"/>
    </record>
    <record id="account_group_192" model="account.group">
        <field name="code_prefix">192</field>
        <field name="name">Suscriptores de acciones</field>
        <field name="parent_id" ref="l10n_es.account_group_19"/>
    </record>
    <record id="account_group_194" model="account.group">
        <field name="code_prefix">194</field>
        <field name="name">Capital emitido pendiente de inscripción</field>
        <field name="parent_id" ref="l10n_es.account_group_19"/>
    </record>
    <record id="account_group_195" model="account.group">
        <field name="code_prefix">195</field>
        <field name="name">Acciones o participaciones emitidas consideradas como pasivos financieros</field>
        <field name="parent_id" ref="l10n_es.account_group_19"/>
    </record>
    <record id="account_group_197" model="account.group">
        <field name="code_prefix">197</field>
        <field name="name">Suscriptores de acciones consideradas como pasivos financieros</field>
        <field name="parent_id" ref="l10n_es.account_group_19"/>
    </record>
    <record id="account_group_199" model="account.group">
        <field name="code_prefix">199</field>
        <field name="name">Acciones o participaciones emitidas consideradas como pasivos financieros pendientes de inscripción.</field>
        <field name="parent_id" ref="l10n_es.account_group_19"/>
    </record>
    <record id="account_group_2" model="account.group">
        <field name="code_prefix">2</field>
        <field name="name">Activo no corriente</field>
    </record>   
    <record id="account_group_20" model="account.group">
        <field name="code_prefix">20</field>
        <field name="name">Inmovilizaciones intangibles</field>
        <field name="parent_id" ref="l10n_es.account_group_2"/>
    </record>
    <record id="account_group_200" model="account.group">
        <field name="code_prefix">200</field>
        <field name="name">Investigación</field>
        <field name="parent_id" ref="l10n_es.account_group_20"/>
    </record>
    <record id="account_group_201" model="account.group">
        <field name="code_prefix">201</field>
        <field name="name">Desarrollo</field>
        <field name="parent_id" ref="l10n_es.account_group_20"/>
    </record>
    <record id="account_group_202" model="account.group">
        <field name="code_prefix">202</field>
        <field name="name">Concesiones administrativas</field>
        <field name="parent_id" ref="l10n_es.account_group_20"/>
    </record>
    <record id="account_group_203" model="account.group">
        <field name="code_prefix">203</field>
        <field name="name">Propiedad industrial</field>
        <field name="parent_id" ref="l10n_es.account_group_20"/>
    </record>
    <record id="account_group_204" model="account.group">
        <field name="code_prefix">204</field>
        <field name="name">Fondo de comercio</field>
        <field name="parent_id" ref="l10n_es.account_group_20"/>
    </record>
    <record id="account_group_205" model="account.group">
        <field name="code_prefix">205</field>
        <field name="name">Derechos de traspaso</field>
        <field name="parent_id" ref="l10n_es.account_group_20"/>
    </record>
    <record id="account_group_206" model="account.group">
        <field name="code_prefix">206</field>
        <field name="name">Aplicaciones informáticas</field>
        <field name="parent_id" ref="l10n_es.account_group_20"/>
    </record>
    <record id="account_group_207" model="account.group">
        <field name="code_prefix">207</field>
        <field name="name">Derechos sobre activos cedidos en uso</field>
        <field name="parent_id" ref="l10n_es.account_group_20"/>
    </record>
    <record id="account_group_209" model="account.group">
        <field name="code_prefix">209</field>
        <field name="name">Anticipos para inmovilizaciones intangibles</field>
        <field name="parent_id" ref="l10n_es.account_group_20"/>
    </record>
    <record id="account_group_21" model="account.group">
        <field name="code_prefix">21</field>
        <field name="name">Inmovilizaciones materiales</field>
        <field name="parent_id" ref="l10n_es.account_group_2"/>
    </record>
    <record id="account_group_210" model="account.group">
        <field name="code_prefix">210</field>
        <field name="name">Terrenos y bienes naturales</field>
        <field name="parent_id" ref="l10n_es.account_group_21"/>
    </record>
    <record id="account_group_211" model="account.group">
        <field name="code_prefix">211</field>
        <field name="name">Construcciones</field>
        <field name="parent_id" ref="l10n_es.account_group_21"/>
    </record>
    <record id="account_group_212" model="account.group">
        <field name="code_prefix">212</field>
        <field name="name">Instalaciones técnicas</field>
        <field name="parent_id" ref="l10n_es.account_group_21"/>
    </record>
    <record id="account_group_213" model="account.group">
        <field name="code_prefix">213</field>
        <field name="name">Maquinaria</field>
        <field name="parent_id" ref="l10n_es.account_group_21"/>
    </record>
    <record id="account_group_214" model="account.group">
        <field name="code_prefix">214</field>
        <field name="name">Utillaje</field>
        <field name="parent_id" ref="l10n_es.account_group_21"/>
    </record>
    <record id="account_group_215" model="account.group">
        <field name="code_prefix">215</field>
        <field name="name">Otras instalaciones</field>
        <field name="parent_id" ref="l10n_es.account_group_21"/>
    </record>
    <record id="account_group_216" model="account.group">
        <field name="code_prefix">216</field>
        <field name="name">Mobiliario</field>
        <field name="parent_id" ref="l10n_es.account_group_21"/>
    </record>
    <record id="account_group_217" model="account.group">
        <field name="code_prefix">217</field>
        <field name="name">Equipos para procesos de información</field>
        <field name="parent_id" ref="l10n_es.account_group_21"/>
    </record>
    <record id="account_group_218" model="account.group">
        <field name="code_prefix">218</field>
        <field name="name">Elementos de transporte</field>
        <field name="parent_id" ref="l10n_es.account_group_21"/>
    </record>
    <record id="account_group_219" model="account.group">
        <field name="code_prefix">219</field>
        <field name="name">Otro inmovilizado material</field>
        <field name="parent_id" ref="l10n_es.account_group_21"/>
    </record>
    <record id="account_group_22" model="account.group">
        <field name="code_prefix">22</field>
        <field name="name">Inversiones inmobiliarias</field>
        <field name="parent_id" ref="l10n_es.account_group_2"/>
    </record>
    <record id="account_group_220" model="account.group">
        <field name="code_prefix">220</field>
        <field name="name">Inversiones en terrenos y bienes naturales</field>
        <field name="parent_id" ref="l10n_es.account_group_22"/>
    </record>
    <record id="account_group_221" model="account.group">
        <field name="code_prefix">221</field>
        <field name="name">Inversiones en construcciones</field>
        <field name="parent_id" ref="l10n_es.account_group_22"/>
    </record>
    <record id="account_group_23" model="account.group">
        <field name="code_prefix">23</field>
        <field name="name">Inmovilizaciones materiales en curso</field>
        <field name="parent_id" ref="l10n_es.account_group_2"/>
    </record>
    <record id="account_group_230" model="account.group">
        <field name="code_prefix">230</field>
        <field name="name">Adaptación de terrenos y bienes naturales</field>
        <field name="parent_id" ref="l10n_es.account_group_23"/>
    </record>
    <record id="account_group_231" model="account.group">
        <field name="code_prefix">231</field>
        <field name="name">Construcciones en curso</field>
        <field name="parent_id" ref="l10n_es.account_group_23"/>
    </record>
    <record id="account_group_232" model="account.group">
        <field name="code_prefix">232</field>
        <field name="name">Instalaciones técnicas en montaje</field>
        <field name="parent_id" ref="l10n_es.account_group_23"/>
    </record>
    <record id="account_group_233" model="account.group">
        <field name="code_prefix">233</field>
        <field name="name">Maquinaria en montaje</field>
        <field name="parent_id" ref="l10n_es.account_group_23"/>
    </record>
    <record id="account_group_237" model="account.group">
        <field name="code_prefix">237</field>
        <field name="name">Equipos para procesos de información en montaje</field>
        <field name="parent_id" ref="l10n_es.account_group_23"/>
    </record>
    <record id="account_group_239" model="account.group">
        <field name="code_prefix">239</field>
        <field name="name">Anticipos para inmovilizaciones materiales</field>
        <field name="parent_id" ref="l10n_es.account_group_23"/>
    </record>
    <record id="account_group_24" model="account.group">
        <field name="code_prefix">24</field>
        <field name="name">Inversiones financieras a largo plazo en partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_2"/>
    </record>
    <record id="account_group_240" model="account.group">
        <field name="code_prefix">240</field>
        <field name="name">Participaciones a largo plazo en partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_24"/>
    </record>
    <record id="account_group_2403" model="account.group">
        <field name="code_prefix">2403</field>
        <field name="name">Participaciones a largo plazo en empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_240"/>
    </record>
    <record id="account_group_2404" model="account.group">
        <field name="code_prefix">2404</field>
        <field name="name">Participaciones a largo plazo en empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_240"/>
    </record>
    <record id="account_group_2405" model="account.group">
        <field name="code_prefix">2405</field>
        <field name="name">Participaciones a largo plazo en otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_240"/>
    </record>
    <record id="account_group_241" model="account.group">
        <field name="code_prefix">241</field>
        <field name="name">Valores representativos de deuda a largo plazo de partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_24"/>
    </record>
    <record id="account_group_2413" model="account.group">
        <field name="code_prefix">2413</field>
        <field name="name">Valores representativos de deuda a largo plazo de empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_241"/>
    </record>
    <record id="account_group_2414" model="account.group">
        <field name="code_prefix">2414</field>
        <field name="name">Valores representativos de deuda a largo plazo de empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_241"/>
    </record>
    <record id="account_group_2415" model="account.group">
        <field name="code_prefix">2415</field>
        <field name="name">Valores representativos de deuda a largo plazo de otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_241"/>
    </record>
    <record id="account_group_242" model="account.group">
        <field name="code_prefix">242</field>
        <field name="name">Créditos a largo plazo a partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_24"/>
    </record>
    <record id="account_group_2423" model="account.group">
        <field name="code_prefix">2423</field>
        <field name="name">Créditos a largo plazo a empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_242"/>
    </record>
    <record id="account_group_2424" model="account.group">
        <field name="code_prefix">2424</field>
        <field name="name">Créditos a largo plazo a empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_242"/>
    </record>
    <record id="account_group_2425" model="account.group">
        <field name="code_prefix">2425</field>
        <field name="name">Créditos a largo plazo a otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_242"/>
    </record>
    <record id="account_group_249" model="account.group">
        <field name="code_prefix">249</field>
        <field name="name">Desembolsos pendientes sobre participaciones a largo plazo en partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_24"/>
    </record>
    <record id="account_group_2493" model="account.group">
        <field name="code_prefix">2493</field>
        <field name="name">Desembolsos pendientes sobre participaciones a largo plazo en empresas del grupo.</field>
        <field name="parent_id" ref="l10n_es.account_group_249"/>
    </record>
    <record id="account_group_2494" model="account.group">
        <field name="code_prefix">2494</field>
        <field name="name">Desembolsos pendientes sobre participaciones a largo plazo en empresas asociadas.</field>
        <field name="parent_id" ref="l10n_es.account_group_249"/>
    </record>
    <record id="account_group_2495" model="account.group">
        <field name="code_prefix">2495</field>
        <field name="name">Desembolsos pendientes sobre participaciones a largo plazo en otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_249"/>
    </record>
    <record id="account_group_25" model="account.group">
        <field name="code_prefix">25</field>
        <field name="name">Otras inversiones financieras a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_2"/>
    </record>
    <record id="account_group_250" model="account.group">
        <field name="code_prefix">250</field>
        <field name="name">Inversiones financieras a largo plazo en instrumentos de patrimonio</field>
        <field name="parent_id" ref="l10n_es.account_group_25"/>
    </record>
    <record id="account_group_251" model="account.group">
        <field name="code_prefix">251</field>
        <field name="name">Valores representativos de deuda a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_25"/>
    </record>
    <record id="account_group_252" model="account.group">
        <field name="code_prefix">252</field>
        <field name="name">Créditos a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_25"/>
    </record>
    <record id="account_group_253" model="account.group">
        <field name="code_prefix">253</field>
        <field name="name">Créditos a largo plazo por enajenación de inmovilizado</field>
        <field name="parent_id" ref="l10n_es.account_group_25"/>
    </record>
    <record id="account_group_254" model="account.group">
        <field name="code_prefix">254</field>
        <field name="name">Créditos a largo plazo al personal</field>
        <field name="parent_id" ref="l10n_es.account_group_25"/>
    </record>
    <record id="account_group_255" model="account.group">
        <field name="code_prefix">255</field>
        <field name="name">Activos por derivados financieros a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_25"/>
    </record>
    <record id="account_group_2550" model="account.group">
        <field name="code_prefix">2550</field>
        <field name="name">Activos por derivados financieros a largo plazo, cartera de negociación</field>
        <field name="parent_id" ref="l10n_es.account_group_255"/>
    </record>
    <record id="account_group_2553" model="account.group">
        <field name="code_prefix">2553</field>
        <field name="name">Activos por derivados financieros a largo plazo, instrumentos de cobertura</field>
        <field name="parent_id" ref="l10n_es.account_group_255"/>
    </record>
    <record id="account_group_257" model="account.group">
        <field name="code_prefix">257</field>
        <field name="name">Derechos de reembolso derivados de contratos de seguro relativos a retribuciones a largo plazo al personal</field>
        <field name="parent_id" ref="l10n_es.account_group_25"/>
    </record>
    <record id="account_group_258" model="account.group">
        <field name="code_prefix">258</field>
        <field name="name">Imposiciones a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_25"/>
    </record>
    <record id="account_group_259" model="account.group">
        <field name="code_prefix">259</field>
        <field name="name">Desembolsos pendientes sobre participaciones en el patrimonio neto a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_25"/>
    </record>
    <record id="account_group_26" model="account.group">
        <field name="code_prefix">26</field>
        <field name="name">Fianzas y depósitos constituidos a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_2"/>
    </record>
    <record id="account_group_260" model="account.group">
        <field name="code_prefix">260</field>
        <field name="name">Fianzas constituidas a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_26"/>
    </record>
    <record id="account_group_265" model="account.group">
        <field name="code_prefix">265</field>
        <field name="name">Depósitos constituidos a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_26"/>
    </record>
    <record id="account_group_28" model="account.group">
        <field name="code_prefix">28</field>
        <field name="name">Amortización acumulada del inmovilizado</field>
        <field name="parent_id" ref="l10n_es.account_group_2"/>
    </record>
    <record id="account_group_280" model="account.group">
        <field name="code_prefix">280</field>
        <field name="name">Amortización acumulada del inmovilizado intangible</field>
        <field name="parent_id" ref="l10n_es.account_group_28"/>
    </record>
    <record id="account_group_2800" model="account.group">
        <field name="code_prefix">2800</field>
        <field name="name">Amortización acumulada de investigación</field>
        <field name="parent_id" ref="l10n_es.account_group_280"/>
    </record>
    <record id="account_group_2801" model="account.group">
        <field name="code_prefix">2801</field>
        <field name="name">Amortización acumulada de desarrollo</field>
        <field name="parent_id" ref="l10n_es.account_group_280"/>
    </record>
    <record id="account_group_2802" model="account.group">
        <field name="code_prefix">2802</field>
        <field name="name">Amortización acumulada de concesiones administrativas</field>
        <field name="parent_id" ref="l10n_es.account_group_280"/>
    </record>
    <record id="account_group_2803" model="account.group">
        <field name="code_prefix">2803</field>
        <field name="name">Amortización acumulada de propiedad industrial</field>
        <field name="parent_id" ref="l10n_es.account_group_280"/>
    </record>
    <record id="account_group_2805" model="account.group">
        <field name="code_prefix">2805</field>
        <field name="name">Amortización acumulada de derechos de traspaso</field>
        <field name="parent_id" ref="l10n_es.account_group_280"/>
    </record>
    <record id="account_group_2806" model="account.group">
        <field name="code_prefix">2806</field>
        <field name="name">Amortización acumulada de aplicaciones informáticas</field>
        <field name="parent_id" ref="l10n_es.account_group_280"/>
    </record>
    <record id="account_group_281" model="account.group">
        <field name="code_prefix">281</field>
        <field name="name">Amortización acumulada del inmovilizado material</field>
        <field name="parent_id" ref="l10n_es.account_group_28"/>
    </record>
    <record id="account_group_2811" model="account.group">
        <field name="code_prefix">2811</field>
        <field name="name">Amortización acumulada de construcciones</field>
        <field name="parent_id" ref="l10n_es.account_group_281"/>
    </record>
    <record id="account_group_2812" model="account.group">
        <field name="code_prefix">2812</field>
        <field name="name">Amortización acumulada de instalaciones técnicas</field>
        <field name="parent_id" ref="l10n_es.account_group_281"/>
    </record>
    <record id="account_group_2813" model="account.group">
        <field name="code_prefix">2813</field>
        <field name="name">Amortización acumulada de maquinaria</field>
        <field name="parent_id" ref="l10n_es.account_group_281"/>
    </record>
    <record id="account_group_2814" model="account.group">
        <field name="code_prefix">2814</field>
        <field name="name">Amortización acumulada de utillaje</field>
        <field name="parent_id" ref="l10n_es.account_group_281"/>
    </record>
    <record id="account_group_2815" model="account.group">
        <field name="code_prefix">2815</field>
        <field name="name">Amortización acumulada de otras instalaciones</field>
        <field name="parent_id" ref="l10n_es.account_group_281"/>
    </record>
    <record id="account_group_2816" model="account.group">
        <field name="code_prefix">2816</field>
        <field name="name">Amortización acumulada de mobiliario</field>
        <field name="parent_id" ref="l10n_es.account_group_281"/>
    </record>
    <record id="account_group_2817" model="account.group">
        <field name="code_prefix">2817</field>
        <field name="name">Amortización acumulada de equipos para procesos de información</field>
        <field name="parent_id" ref="l10n_es.account_group_281"/>
    </record>
    <record id="account_group_2818" model="account.group">
        <field name="code_prefix">2818</field>
        <field name="name">Amortización acumulada de elementos de transporte</field>
        <field name="parent_id" ref="l10n_es.account_group_281"/>
    </record>
    <record id="account_group_2819" model="account.group">
        <field name="code_prefix">2819</field>
        <field name="name">Amortización acumulada de otro inmovilizado material</field>
        <field name="parent_id" ref="l10n_es.account_group_281"/>
    </record>
    <record id="account_group_282" model="account.group">
        <field name="code_prefix">282</field>
        <field name="name">Amortización acumulada de las inversiones inmobiliarias</field>
        <field name="parent_id" ref="l10n_es.account_group_28"/>
    </record>
    <record id="account_group_283" model="account.group">
        <field name="code_prefix">283</field>
        <field name="name">Cesiones de uso sin contraprestación</field>
        <field name="parent_id" ref="l10n_es.account_group_28"/>
    </record>
    <record id="account_group_29" model="account.group">
        <field name="code_prefix">29</field>
        <field name="name">Deterioro de valor de activos no corrientes</field>
        <field name="parent_id" ref="l10n_es.account_group_2"/>
    </record>
    <record id="account_group_290" model="account.group">
        <field name="code_prefix">290</field>
        <field name="name">Deterioro de valor del inmovilizado intangible</field>
        <field name="parent_id" ref="l10n_es.account_group_29"/>
    </record>
    <record id="account_group_2900" model="account.group">
        <field name="code_prefix">2900</field>
        <field name="name">Deterioro de valor de investigación</field>
        <field name="parent_id" ref="l10n_es.account_group_290"/>
    </record>
    <record id="account_group_2901" model="account.group">
        <field name="code_prefix">2901</field>
        <field name="name">Deterioro del valor de desarrollo</field>
        <field name="parent_id" ref="l10n_es.account_group_290"/>
    </record>
    <record id="account_group_2902" model="account.group">
        <field name="code_prefix">2902</field>
        <field name="name">Deterioro de valor de concesiones administrativas</field>
        <field name="parent_id" ref="l10n_es.account_group_290"/>
    </record>
    <record id="account_group_2903" model="account.group">
        <field name="code_prefix">2903</field>
        <field name="name">Deterioro de valor de propiedad industrial</field>
        <field name="parent_id" ref="l10n_es.account_group_290"/>
    </record>
    <record id="account_group_2905" model="account.group">
        <field name="code_prefix">2905</field>
        <field name="name">Deterioro de valor de derechos de traspaso</field>
        <field name="parent_id" ref="l10n_es.account_group_290"/>
    </record>
    <record id="account_group_2906" model="account.group">
        <field name="code_prefix">2906</field>
        <field name="name">Deterioro de valor de aplicaciones informáticas</field>
        <field name="parent_id" ref="l10n_es.account_group_290"/>
    </record>
    <record id="account_group_291" model="account.group">
        <field name="code_prefix">291</field>
        <field name="name">Deterioro de valor del inmovilizado material</field>
        <field name="parent_id" ref="l10n_es.account_group_29"/>
    </record>
    <record id="account_group_2910" model="account.group">
        <field name="code_prefix">2910</field>
        <field name="name">Deterioro de valor de terrenos y bienes naturales</field>
        <field name="parent_id" ref="l10n_es.account_group_291"/>
    </record>
    <record id="account_group_2911" model="account.group">
        <field name="code_prefix">2911</field>
        <field name="name">Deterioro de valor de construcciones</field>
        <field name="parent_id" ref="l10n_es.account_group_291"/>
    </record>
    <record id="account_group_2912" model="account.group">
        <field name="code_prefix">2912</field>
        <field name="name">Deterioro de valor de instalaciones técnicas</field>
        <field name="parent_id" ref="l10n_es.account_group_291"/>
    </record>
    <record id="account_group_2913" model="account.group">
        <field name="code_prefix">2913</field>
        <field name="name">Deterioro de valor de maquinaria</field>
        <field name="parent_id" ref="l10n_es.account_group_291"/>
    </record>
    <record id="account_group_2914" model="account.group">
        <field name="code_prefix">2914</field>
        <field name="name">Deterioro de valor de utillaje</field>
        <field name="parent_id" ref="l10n_es.account_group_291"/>
    </record>
    <record id="account_group_2915" model="account.group">
        <field name="code_prefix">2915</field>
        <field name="name">Deterioro de valor de otras instalaciones</field>
        <field name="parent_id" ref="l10n_es.account_group_291"/>
    </record>
    <record id="account_group_2916" model="account.group">
        <field name="code_prefix">2916</field>
        <field name="name">Deterioro de valor de mobiliario</field>
        <field name="parent_id" ref="l10n_es.account_group_291"/>
    </record>
    <record id="account_group_2917" model="account.group">
        <field name="code_prefix">2917</field>
        <field name="name">Deterioro de valor de equipos para procesos de información</field>
        <field name="parent_id" ref="l10n_es.account_group_291"/>
    </record>
    <record id="account_group_2918" model="account.group">
        <field name="code_prefix">2918</field>
        <field name="name">Deterioro de valor de elementos de transporte</field>
        <field name="parent_id" ref="l10n_es.account_group_291"/>
    </record>
    <record id="account_group_2919" model="account.group">
        <field name="code_prefix">2919</field>
        <field name="name">Deterioro de valor de otro inmovilizado material</field>
        <field name="parent_id" ref="l10n_es.account_group_291"/>
    </record>
    <record id="account_group_292" model="account.group">
        <field name="code_prefix">292</field>
        <field name="name">Deterioro de valor de las inversiones inmobiliarias</field>
        <field name="parent_id" ref="l10n_es.account_group_29"/>
    </record>
    <record id="account_group_2920" model="account.group">
        <field name="code_prefix">2920</field>
        <field name="name">Deterioro de valor de los terrenos y bienes naturales</field>
        <field name="parent_id" ref="l10n_es.account_group_292"/>
    </record>
    <record id="account_group_2921" model="account.group">
        <field name="code_prefix">2921</field>
        <field name="name">Deterioro de valor de construcciones</field>
        <field name="parent_id" ref="l10n_es.account_group_292"/>
    </record>
    <record id="account_group_293" model="account.group">
        <field name="code_prefix">293</field>
        <field name="name">Deterioro de valor de participaciones a largo plazo en partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_29"/>
    </record>
    <record id="account_group_2933" model="account.group">
        <field name="code_prefix">2933</field>
        <field name="name">Deterioro de valor de participaciones a largo plazo en empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_293"/>
    </record>
    <record id="account_group_2934" model="account.group">
        <field name="code_prefix">2934</field>
        <field name="name">Deterioro de valor de participaciones a largo plazo en empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_293"/>
    </record>
    <record id="account_group_294" model="account.group">
        <field name="code_prefix">294</field>
        <field name="name">Deterioro de valor de valores representativos de deuda a largo plazo de partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_29"/>
    </record>
    <record id="account_group_2943" model="account.group">
        <field name="code_prefix">2943</field>
        <field name="name">Deterioro de valor de valores representativos de deuda a largo plazo de empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_294"/>
    </record>
    <record id="account_group_2944" model="account.group">
        <field name="code_prefix">2944</field>
        <field name="name">Deterioro de valor de valores representativos de deuda a largo plazo de empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_294"/>
    </record>
    <record id="account_group_2945" model="account.group">
        <field name="code_prefix">2945</field>
        <field name="name">Deterioro de valor de valores representativos de deuda a largo plazo de otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_294"/>
    </record>
    <record id="account_group_295" model="account.group">
        <field name="code_prefix">295</field>
        <field name="name">Deterioro de valor de créditos a largo plazo a partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_29"/>
    </record>
    <record id="account_group_2953" model="account.group">
        <field name="code_prefix">2953</field>
        <field name="name">Deterioro de valor de créditos a largo plazo a empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_295"/>
    </record>
    <record id="account_group_2954" model="account.group">
        <field name="code_prefix">2954</field>
        <field name="name">Deterioro de valor de créditos a largo plazo a empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_295"/>
    </record>
    <record id="account_group_2955" model="account.group">
        <field name="code_prefix">2955</field>
        <field name="name">Deterioro de valor de créditos a largo plazo a otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_295"/>
    </record>
    <record id="account_group_296" model="account.group">
        <field name="code_prefix">296</field>
        <field name="name">Deterioro de valor de participaciones en el patrimonio neto a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_29"/>
    </record>
    <record id="account_group_297" model="account.group">
        <field name="code_prefix">297</field>
        <field name="name">Deterioro de valor de valores representativos de deuda a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_29"/>
    </record>
    <record id="account_group_298" model="account.group">
        <field name="code_prefix">298</field>
        <field name="name">Deterioro de valor de créditos a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_29"/>
    </record>
    <record id="account_group_299" model="account.group">
        <field name="code_prefix">299</field>
        <field name="name">Deterioro de valor de bienes inmuebles</field>
        <field name="parent_id" ref="l10n_es.account_group_29"/>
    </record>
    <record id="account_group_3" model="account.group">
        <field name="code_prefix">3</field>
        <field name="name">Existencias</field>
    </record>
    <record id="account_group_30" model="account.group">
        <field name="code_prefix">30</field>
        <field name="name">Comerciales</field>
        <field name="parent_id" ref="l10n_es.account_group_3"/>
    </record>
    <record id="account_group_300" model="account.group">
        <field name="code_prefix">300</field>
        <field name="name">Mercaderías A</field>
        <field name="parent_id" ref="l10n_es.account_group_30"/>
    </record>
    <record id="account_group_301" model="account.group">
        <field name="code_prefix">301</field>
        <field name="name">Mercaderías B</field>
        <field name="parent_id" ref="l10n_es.account_group_30"/>
    </record>
    <record id="account_group_31" model="account.group">
        <field name="code_prefix">31</field>
        <field name="name">Materias primas</field>
        <field name="parent_id" ref="l10n_es.account_group_3"/>
    </record>
    <record id="account_group_310" model="account.group">
        <field name="code_prefix">310</field>
        <field name="name">Materias primas A</field>
        <field name="parent_id" ref="l10n_es.account_group_31"/>
    </record>
    <record id="account_group_311" model="account.group">
        <field name="code_prefix">311</field>
        <field name="name">Materias primas B</field>
        <field name="parent_id" ref="l10n_es.account_group_31"/>
    </record>
    <record id="account_group_32" model="account.group">
        <field name="code_prefix">32</field>
        <field name="name">Otros aprovisionamientos</field>
        <field name="parent_id" ref="l10n_es.account_group_3"/>
    </record>
    <record id="account_group_320" model="account.group">
        <field name="code_prefix">320</field>
        <field name="name">Elementos y conjuntos incorporables</field>
        <field name="parent_id" ref="l10n_es.account_group_32"/>
    </record>
    <record id="account_group_321" model="account.group">
        <field name="code_prefix">321</field>
        <field name="name">Combustibles</field>
        <field name="parent_id" ref="l10n_es.account_group_32"/>
    </record>
    <record id="account_group_322" model="account.group">
        <field name="code_prefix">322</field>
        <field name="name">Repuestos</field>
        <field name="parent_id" ref="l10n_es.account_group_32"/>
    </record>
    <record id="account_group_325" model="account.group">
        <field name="code_prefix">325</field>
        <field name="name">Materiales diversos</field>
        <field name="parent_id" ref="l10n_es.account_group_32"/>
    </record>
    <record id="account_group_326" model="account.group">
        <field name="code_prefix">326</field>
        <field name="name">Embalajes</field>
        <field name="parent_id" ref="l10n_es.account_group_32"/>
    </record>
    <record id="account_group_327" model="account.group">
        <field name="code_prefix">327</field>
        <field name="name">Envases</field>
        <field name="parent_id" ref="l10n_es.account_group_32"/>
    </record>
    <record id="account_group_328" model="account.group">
        <field name="code_prefix">328</field>
        <field name="name">Material de oficina</field>
        <field name="parent_id" ref="l10n_es.account_group_32"/>
    </record>
    <record id="account_group_33" model="account.group">
        <field name="code_prefix">33</field>
        <field name="name">Productos en curso</field>
        <field name="parent_id" ref="l10n_es.account_group_3"/>
    </record>
    <record id="account_group_330" model="account.group">
        <field name="code_prefix">330</field>
        <field name="name">Productos en curso A</field>
        <field name="parent_id" ref="l10n_es.account_group_33"/>
    </record>
    <record id="account_group_331" model="account.group">
        <field name="code_prefix">331</field>
        <field name="name">Productos en curso B</field>
        <field name="parent_id" ref="l10n_es.account_group_33"/>
    </record>
    <record id="account_group_34" model="account.group">
        <field name="code_prefix">34</field>
        <field name="name">Productos semiterminados</field>
        <field name="parent_id" ref="l10n_es.account_group_3"/>
    </record>
    <record id="account_group_340" model="account.group">
        <field name="code_prefix">340</field>
        <field name="name">Productos semiterminados A</field>
        <field name="parent_id" ref="l10n_es.account_group_34"/>
    </record>
    <record id="account_group_341" model="account.group">
        <field name="code_prefix">341</field>
        <field name="name">Productos semiterminados B</field>
        <field name="parent_id" ref="l10n_es.account_group_34"/>
    </record>
    <record id="account_group_35" model="account.group">
        <field name="code_prefix">35</field>
        <field name="name">Productos terminados</field>
        <field name="parent_id" ref="l10n_es.account_group_3"/>
    </record>
    <record id="account_group_350" model="account.group">
        <field name="code_prefix">350</field>
        <field name="name">Productos terminados A</field>
        <field name="parent_id" ref="l10n_es.account_group_35"/>
    </record>
    <record id="account_group_351" model="account.group">
        <field name="code_prefix">351</field>
        <field name="name">Productos terminados B</field>
        <field name="parent_id" ref="l10n_es.account_group_35"/>
    </record>
    <record id="account_group_36" model="account.group">
        <field name="code_prefix">36</field>
        <field name="name">Subproductos, residuos y materiales recuperados</field>
        <field name="parent_id" ref="l10n_es.account_group_3"/>
    </record>
    <record id="account_group_360" model="account.group">
        <field name="code_prefix">360</field>
        <field name="name">Subproductos A</field>
        <field name="parent_id" ref="l10n_es.account_group_36"/>
    </record>
    <record id="account_group_361" model="account.group">
        <field name="code_prefix">361</field>
        <field name="name">Subproductos B</field>
        <field name="parent_id" ref="l10n_es.account_group_36"/>
    </record>
    <record id="account_group_365" model="account.group">
        <field name="code_prefix">365</field>
        <field name="name">Residuos A</field>
        <field name="parent_id" ref="l10n_es.account_group_36"/>
    </record>
    <record id="account_group_366" model="account.group">
        <field name="code_prefix">366</field>
        <field name="name">Residuos B</field>
        <field name="parent_id" ref="l10n_es.account_group_36"/>
    </record>
    <record id="account_group_368" model="account.group">
        <field name="code_prefix">368</field>
        <field name="name">Materiales recuperados A</field>
        <field name="parent_id" ref="l10n_es.account_group_36"/>
    </record>
    <record id="account_group_369" model="account.group">
        <field name="code_prefix">369</field>
        <field name="name">Materiales recuperados B</field>
        <field name="parent_id" ref="l10n_es.account_group_36"/>
    </record>
    <record id="account_group_39" model="account.group">
        <field name="code_prefix">39</field>
        <field name="name">Deterioro de valor de las existencias</field>
        <field name="parent_id" ref="l10n_es.account_group_3"/>
    </record>
    <record id="account_group_390" model="account.group">
        <field name="code_prefix">390</field>
        <field name="name">Deterioro de valor de las mercaderías</field>
        <field name="parent_id" ref="l10n_es.account_group_39"/>
    </record>
    <record id="account_group_391" model="account.group">
        <field name="code_prefix">391</field>
        <field name="name">Deterioro de valor de las materias primas</field>
        <field name="parent_id" ref="l10n_es.account_group_39"/>
    </record>
    <record id="account_group_392" model="account.group">
        <field name="code_prefix">392</field>
        <field name="name">Deterioro de valor de otros aprovisionamientos</field>
        <field name="parent_id" ref="l10n_es.account_group_39"/>
    </record>
    <record id="account_group_393" model="account.group">
        <field name="code_prefix">393</field>
        <field name="name">Deterioro de valor de los productos en curso</field>
        <field name="parent_id" ref="l10n_es.account_group_39"/>
    </record>
    <record id="account_group_394" model="account.group">
        <field name="code_prefix">394</field>
        <field name="name">Deterioro de valor de los productos semiterminados</field>
        <field name="parent_id" ref="l10n_es.account_group_39"/>
    </record>
    <record id="account_group_395" model="account.group">
        <field name="code_prefix">395</field>
        <field name="name">Deterioro de valor de los productos terminados</field>
        <field name="parent_id" ref="l10n_es.account_group_39"/>
    </record>
    <record id="account_group_396" model="account.group">
        <field name="code_prefix">396</field>
        <field name="name">Deterioro de valor de los subproductos, residuos y materiales recuperados</field>
        <field name="parent_id" ref="l10n_es.account_group_39"/>
    </record>
    <record id="account_group_4" model="account.group">
        <field name="code_prefix">4</field>
        <field name="name">Acreedores y deudores por operaciones comerciales</field>
    </record>
    <record id="account_group_40" model="account.group">
        <field name="code_prefix">40</field>
        <field name="name">Proveedores</field>
        <field name="parent_id" ref="l10n_es.account_group_4"/>
    </record>
    <record id="account_group_400" model="account.group">
        <field name="code_prefix">400</field>
        <field name="name">Proveedores</field>
        <field name="parent_id" ref="l10n_es.account_group_40"/>
    </record>
    <record id="account_group_4000" model="account.group">
        <field name="code_prefix">4000</field>
        <field name="name">Proveedores (euros)</field>
        <field name="parent_id" ref="l10n_es.account_group_400"/>
    </record>
    <record id="account_group_4004" model="account.group">
        <field name="code_prefix">4004</field>
        <field name="name">Proveedores (moneda extranjera)</field>
        <field name="parent_id" ref="l10n_es.account_group_400"/>
    </record>
    <record id="account_group_4009" model="account.group">
        <field name="code_prefix">4009</field>
        <field name="name">Proveedores, facturas pendientes de recibir o de formalizar</field>
        <field name="parent_id" ref="l10n_es.account_group_400"/>
    </record>
    <record id="account_group_401" model="account.group">
        <field name="code_prefix">401</field>
        <field name="name">Proveedores, efectos comerciales a pagar</field>
        <field name="parent_id" ref="l10n_es.account_group_40"/>
    </record>
    <record id="account_group_403" model="account.group">
        <field name="code_prefix">403</field>
        <field name="name">Proveedores, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_40"/>
    </record>
    <record id="account_group_4030" model="account.group">
        <field name="code_prefix">4030</field>
        <field name="name">Proveedores, empresas del grupo (euros)</field>
        <field name="parent_id" ref="l10n_es.account_group_403"/>
    </record>
    <record id="account_group_4031" model="account.group">
        <field name="code_prefix">4031</field>
        <field name="name">Efectos comerciales a pagar, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_403"/>
    </record>
    <record id="account_group_4034" model="account.group">
        <field name="code_prefix">4034</field>
        <field name="name">Proveedores, empresas del grupo (moneda extranjera)</field>
        <field name="parent_id" ref="l10n_es.account_group_403"/>
    </record>
    <record id="account_group_4036" model="account.group">
        <field name="code_prefix">4036</field>
        <field name="name">Envases y embalajes a devolver a proveedores, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_403"/>
    </record>
    <record id="account_group_4039" model="account.group">
        <field name="code_prefix">4039</field>
        <field name="name">Proveedores, empresas del grupo, facturas pendientes de recibir o de formalizar</field>
        <field name="parent_id" ref="l10n_es.account_group_403"/>
    </record>
    <record id="account_group_404" model="account.group">
        <field name="code_prefix">404</field>
        <field name="name">Proveedores, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_40"/>
    </record>
    <record id="account_group_405" model="account.group">
        <field name="code_prefix">405</field>
        <field name="name">Proveedores, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_40"/>
    </record>
    <record id="account_group_406" model="account.group">
        <field name="code_prefix">406</field>
        <field name="name">Envases y embalajes a devolver a proveedores</field>
        <field name="parent_id" ref="l10n_es.account_group_40"/>
    </record>
    <record id="account_group_407" model="account.group">
        <field name="code_prefix">407</field>
        <field name="name">Anticipos a proveedores</field>
        <field name="parent_id" ref="l10n_es.account_group_40"/>
    </record>
    <record id="account_group_41" model="account.group">
        <field name="code_prefix">41</field>
        <field name="name">Acreedores varios</field>
        <field name="parent_id" ref="l10n_es.account_group_4"/>
    </record>
    <record id="account_group_410" model="account.group">
        <field name="code_prefix">410</field>
        <field name="name">Acreedores por prestaciones de servicios</field>
        <field name="parent_id" ref="l10n_es.account_group_41"/>
    </record>
    <record id="account_group_4100" model="account.group">
        <field name="code_prefix">4100</field>
        <field name="name">Acreedores por prestaciones de servicios (euros)</field>
        <field name="parent_id" ref="l10n_es.account_group_410"/>
    </record>
    <record id="account_group_4104" model="account.group">
        <field name="code_prefix">4104</field>
        <field name="name">Acreedores por prestaciones de servicios (moneda extranjera)</field>
        <field name="parent_id" ref="l10n_es.account_group_410"/>
    </record>
    <record id="account_group_4109" model="account.group">
        <field name="code_prefix">4109</field>
        <field name="name">Acreedores por prestaciones de servicios, facturas pendientes de recibir o de formalizar</field>
        <field name="parent_id" ref="l10n_es.account_group_410"/>
    </record>
    <record id="account_group_411" model="account.group">
        <field name="code_prefix">411</field>
        <field name="name">Acreedores, efectos comerciales a pagar</field>
        <field name="parent_id" ref="l10n_es.account_group_41"/>
    </record>
    <record id="account_group_412" model="account.group">
        <field name="code_prefix">412</field>
        <field name="name">Beneficiarios, acreedores</field>
        <field name="parent_id" ref="l10n_es.account_group_41"/>
    </record>
    <record id="account_group_419" model="account.group">
        <field name="code_prefix">419</field>
        <field name="name">Acreedores por operaciones en común</field>
        <field name="parent_id" ref="l10n_es.account_group_41"/>
    </record>
    <record id="account_group_43" model="account.group">
        <field name="code_prefix">43</field>
        <field name="name">Clientes</field>
        <field name="parent_id" ref="l10n_es.account_group_4"/>
    </record>
    <record id="account_group_430" model="account.group">
        <field name="code_prefix">430</field>
        <field name="name">Clientes</field>
        <field name="parent_id" ref="l10n_es.account_group_43"/>
    </record>
    <record id="account_group_4300" model="account.group">
        <field name="code_prefix">4300</field>
        <field name="name">Clientes (euros)</field>
        <field name="parent_id" ref="l10n_es.account_group_430"/>
    </record>
    <record id="account_group_4304" model="account.group">
        <field name="code_prefix">4304</field>
        <field name="name">Clientes (moneda extranjera)</field>
        <field name="parent_id" ref="l10n_es.account_group_430"/>
    </record>
    <record id="account_group_4309" model="account.group">
        <field name="code_prefix">4309</field>
        <field name="name">Clientes, facturas pendientes de formalizar</field>
        <field name="parent_id" ref="l10n_es.account_group_430"/>
    </record>
    <record id="account_group_431" model="account.group">
        <field name="code_prefix">431</field>
        <field name="name">Clientes, efectos comerciales a cobrar</field>
        <field name="parent_id" ref="l10n_es.account_group_43"/>
    </record>
    <record id="account_group_4310" model="account.group">
        <field name="code_prefix">4310</field>
        <field name="name">Efectos comerciales en cartera</field>
        <field name="parent_id" ref="l10n_es.account_group_431"/>
    </record>
    <record id="account_group_4311" model="account.group">
        <field name="code_prefix">4311</field>
        <field name="name">Efectos comerciales descontados</field>
        <field name="parent_id" ref="l10n_es.account_group_431"/>
    </record>
    <record id="account_group_4312" model="account.group">
        <field name="code_prefix">4312</field>
        <field name="name">Efectos comerciales en gestión de cobro</field>
        <field name="parent_id" ref="l10n_es.account_group_431"/>
    </record>
    <record id="account_group_4315" model="account.group">
        <field name="code_prefix">4315</field>
        <field name="name">Efectos comerciales impagados</field>
        <field name="parent_id" ref="l10n_es.account_group_431"/>
    </record>
    <record id="account_group_432" model="account.group">
        <field name="code_prefix">432</field>
        <field name="name">Clientes, operaciones de «factoring»</field>
        <field name="parent_id" ref="l10n_es.account_group_43"/>
    </record>
    <record id="account_group_433" model="account.group">
        <field name="code_prefix">433</field>
        <field name="name">Clientes, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_43"/>
    </record>
    <record id="account_group_4330" model="account.group">
        <field name="code_prefix">4330</field>
        <field name="name">Clientes empresas del grupo (euros)</field>
        <field name="parent_id" ref="l10n_es.account_group_433"/>
    </record>
    <record id="account_group_4331" model="account.group">
        <field name="code_prefix">4331</field>
        <field name="name">Efectos comerciales a cobrar, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_433"/>
    </record>
    <record id="account_group_4332" model="account.group">
        <field name="code_prefix">4332</field>
        <field name="name">Clientes empresas del grupo, operaciones de «factoring»</field>
        <field name="parent_id" ref="l10n_es.account_group_433"/>
    </record>
    <record id="account_group_4334" model="account.group">
        <field name="code_prefix">4334</field>
        <field name="name">Clientes empresas del grupo (moneda extranjera)</field>
        <field name="parent_id" ref="l10n_es.account_group_433"/>
    </record>
    <record id="account_group_4336" model="account.group">
        <field name="code_prefix">4336</field>
        <field name="name">Clientes empresas del grupo de dudoso cobro</field>
        <field name="parent_id" ref="l10n_es.account_group_433"/>
    </record>
    <record id="account_group_4337" model="account.group">
        <field name="code_prefix">4337</field>
        <field name="name">Envases y embalajes a devolver a clientes, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_433"/>
    </record>
    <record id="account_group_4339" model="account.group">
        <field name="code_prefix">4339</field>
        <field name="name">Clientes empresas del grupo, facturas pendientes de formalizar</field>
        <field name="parent_id" ref="l10n_es.account_group_433"/>
    </record>
    <record id="account_group_434" model="account.group">
        <field name="code_prefix">434</field>
        <field name="name">Clientes, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_43"/>
    </record>
    <record id="account_group_435" model="account.group">
        <field name="code_prefix">435</field>
        <field name="name">Clientes, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_43"/>
    </record>
    <record id="account_group_436" model="account.group">
        <field name="code_prefix">436</field>
        <field name="name">Clientes de dudoso cobro</field>
        <field name="parent_id" ref="l10n_es.account_group_43"/>
    </record>
    <record id="account_group_437" model="account.group">
        <field name="code_prefix">437</field>
        <field name="name">Envases y embalajes a devolver por clientes</field>
        <field name="parent_id" ref="l10n_es.account_group_43"/>
    </record>
    <record id="account_group_438" model="account.group">
        <field name="code_prefix">438</field>
        <field name="name">Anticipos de clientes</field>
        <field name="parent_id" ref="l10n_es.account_group_43"/>
    </record>
    <record id="account_group_44" model="account.group">
        <field name="code_prefix">44</field>
        <field name="name">Deudores varios</field>
        <field name="parent_id" ref="l10n_es.account_group_4"/>
    </record>
    <record id="account_group_440" model="account.group">
        <field name="code_prefix">440</field>
        <field name="name">Deudores</field>
        <field name="parent_id" ref="l10n_es.account_group_44"/>
    </record>
    <record id="account_group_4400" model="account.group">
        <field name="code_prefix">4400</field>
        <field name="name">Deudores (euros)</field>
        <field name="parent_id" ref="l10n_es.account_group_440"/>
    </record>
    <record id="account_group_4404" model="account.group">
        <field name="code_prefix">4404</field>
        <field name="name">Deudores (moneda extranjera)</field>
        <field name="parent_id" ref="l10n_es.account_group_440"/>
    </record>
    <record id="account_group_4409" model="account.group">
        <field name="code_prefix">4409</field>
        <field name="name">Deudores, facturas pendientes de formalizar</field>
        <field name="parent_id" ref="l10n_es.account_group_440"/>
    </record>
    <record id="account_group_441" model="account.group">
        <field name="code_prefix">441</field>
        <field name="name">Deudores, efectos comerciales a cobrar</field>
        <field name="parent_id" ref="l10n_es.account_group_44"/>
    </record>
    <record id="account_group_4410" model="account.group">
        <field name="code_prefix">4410</field>
        <field name="name">Deudores, efectos comerciales en cartera</field>
        <field name="parent_id" ref="l10n_es.account_group_441"/>
    </record>
    <record id="account_group_4411" model="account.group">
        <field name="code_prefix">4411</field>
        <field name="name">Deudores, efectos comerciales descontados</field>
        <field name="parent_id" ref="l10n_es.account_group_441"/>
    </record>
    <record id="account_group_4412" model="account.group">
        <field name="code_prefix">4412</field>
        <field name="name">Deudores, efectos comerciales en gestión de cobro</field>
        <field name="parent_id" ref="l10n_es.account_group_441"/>
    </record>
    <record id="account_group_4415" model="account.group">
        <field name="code_prefix">4415</field>
        <field name="name">Deudores, efectos comerciales impagados</field>
        <field name="parent_id" ref="l10n_es.account_group_441"/>
    </record>
    <record id="account_group_446" model="account.group">
        <field name="code_prefix">446</field>
        <field name="name">Deudores de dudoso cobro</field>
        <field name="parent_id" ref="l10n_es.account_group_44"/>
    </record>
    <record id="account_group_447" model="account.group">
        <field name="code_prefix">447</field>
        <field name="name">Usuarios, deudores</field>
        <field name="parent_id" ref="l10n_es.account_group_44"/>
    </record>
    <record id="account_group_448" model="account.group">
        <field name="code_prefix">448</field>
        <field name="name">Patrocinadores, afiliados y otros deudores</field>
        <field name="parent_id" ref="l10n_es.account_group_44"/>
    </record>
    <record id="account_group_449" model="account.group">
        <field name="code_prefix">449</field>
        <field name="name">Deudores por operaciones en común</field>
        <field name="parent_id" ref="l10n_es.account_group_44"/>
    </record>
    <record id="account_group_46" model="account.group">
        <field name="code_prefix">46</field>
        <field name="name">Personal</field>
        <field name="parent_id" ref="l10n_es.account_group_4"/>
    </record>
    <record id="account_group_460" model="account.group">
        <field name="code_prefix">460</field>
        <field name="name">Anticipos de remuneraciones</field>
        <field name="parent_id" ref="l10n_es.account_group_46"/>
    </record>
    <record id="account_group_464" model="account.group">
        <field name="code_prefix">464</field>
        <field name="name">Entregas para gastos a justificar</field>
        <field name="parent_id" ref="l10n_es.account_group_46"/>
    </record>
    <record id="account_group_465" model="account.group">
        <field name="code_prefix">465</field>
        <field name="name">Remuneraciones pendientes de pago</field>
        <field name="parent_id" ref="l10n_es.account_group_46"/>
    </record>
    <record id="account_group_466" model="account.group">
        <field name="code_prefix">466</field>
        <field name="name">Remuneraciones mediante sistemas de aportación definida pendientes de pago</field>
        <field name="parent_id" ref="l10n_es.account_group_46"/>
    </record>
    <record id="account_group_47" model="account.group">
        <field name="code_prefix">47</field>
        <field name="name">Administraciones públicas</field>
        <field name="parent_id" ref="l10n_es.account_group_4"/>
    </record>
    <record id="account_group_470" model="account.group">
        <field name="code_prefix">470</field>
        <field name="name">Hacienda Pública, deudora por diversos conceptos</field>
        <field name="parent_id" ref="l10n_es.account_group_47"/>
    </record>
    <record id="account_group_4700" model="account.group">
        <field name="code_prefix">4700</field>
        <field name="name">Hacienda Pública, deudora por IVA</field>
        <field name="parent_id" ref="l10n_es.account_group_470"/>
    </record>
    <record id="account_group_4708" model="account.group">
        <field name="code_prefix">4708</field>
        <field name="name">Hacienda Pública, deudora por subvenciones concedidas</field>
        <field name="parent_id" ref="l10n_es.account_group_470"/>
    </record>
    <record id="account_group_4709" model="account.group">
        <field name="code_prefix">4709</field>
        <field name="name">Hacienda Pública, deudora por devolución de impuestos</field>
        <field name="parent_id" ref="l10n_es.account_group_470"/>
    </record>
    <record id="account_group_471" model="account.group">
        <field name="code_prefix">471</field>
        <field name="name">Organismos de la Seguridad Social, deudores</field>
        <field name="parent_id" ref="l10n_es.account_group_47"/>
    </record>
    <record id="account_group_472" model="account.group">
        <field name="code_prefix">472</field>
        <field name="name">Hacienda Pública, IVA soportado</field>
        <field name="parent_id" ref="l10n_es.account_group_47"/>
    </record>
    <record id="account_group_473" model="account.group">
        <field name="code_prefix">473</field>
        <field name="name">Hacienda Pública, retenciones y pagos a cuenta</field>
        <field name="parent_id" ref="l10n_es.account_group_47"/>
    </record>
    <record id="account_group_474" model="account.group">
        <field name="code_prefix">474</field>
        <field name="name">Activos por impuesto diferido</field>
        <field name="parent_id" ref="l10n_es.account_group_47"/>
    </record>
    <record id="account_group_4740" model="account.group">
        <field name="code_prefix">4740</field>
        <field name="name">Activos por diferencias temporarias deducibles</field>
        <field name="parent_id" ref="l10n_es.account_group_474"/>
    </record>
    <record id="account_group_4742" model="account.group">
        <field name="code_prefix">4742</field>
        <field name="name">Derechos por deducciones y bonificaciones pendientes de aplicar</field>
        <field name="parent_id" ref="l10n_es.account_group_474"/>
    </record>
    <record id="account_group_4745" model="account.group">
        <field name="code_prefix">4745</field>
        <field name="name">Crédito por pérdidas a compensar del ejercicio</field>
        <field name="parent_id" ref="l10n_es.account_group_474"/>
    </record>
    <record id="account_group_475" model="account.group">
        <field name="code_prefix">475</field>
        <field name="name">Hacienda Pública, acreedora por conceptos fiscales</field>
        <field name="parent_id" ref="l10n_es.account_group_47"/>
    </record>
    <record id="account_group_4750" model="account.group">
        <field name="code_prefix">4750</field>
        <field name="name">Hacienda Pública, acreedora por IVA</field>
        <field name="parent_id" ref="l10n_es.account_group_475"/>
    </record>
    <record id="account_group_4751" model="account.group">
        <field name="code_prefix">4751</field>
        <field name="name">Hacienda Pública, acreedora por retenciones practicadas</field>
        <field name="parent_id" ref="l10n_es.account_group_475"/>
    </record>
    <record id="account_group_4752" model="account.group">
        <field name="code_prefix">4752</field>
        <field name="name">Hacienda Pública, acreedora por impuesto sobre sociedades</field>
        <field name="parent_id" ref="l10n_es.account_group_475"/>
    </record>
    <record id="account_group_4758" model="account.group">
        <field name="code_prefix">4758</field>
        <field name="name">Hacienda Pública, acreedora por subvenciones a reintegrar</field>
        <field name="parent_id" ref="l10n_es.account_group_475"/>
    </record>
    <record id="account_group_4759" model="account.group">
        <field name="code_prefix">4759</field>
        <field name="name">Hacienda Pública, acreedora por otros conceptos</field>
        <field name="parent_id" ref="l10n_es.account_group_475"/>
    </record>
    <record id="account_group_476" model="account.group">
        <field name="code_prefix">476</field>
        <field name="name">Organismos de la Seguridad Social, acreedores</field>
        <field name="parent_id" ref="l10n_es.account_group_47"/>
    </record>
    <record id="account_group_477" model="account.group">
        <field name="code_prefix">477</field>
        <field name="name">Hacienda Pública, IVA repercutido</field>
        <field name="parent_id" ref="l10n_es.account_group_47"/>
    </record>
    <record id="account_group_479" model="account.group">
        <field name="code_prefix">479</field>
        <field name="name">Pasivos por diferencias temporarias imponibles</field>
        <field name="parent_id" ref="l10n_es.account_group_47"/>
    </record>
    <record id="account_group_48" model="account.group">
        <field name="code_prefix">48</field>
        <field name="name">Ajustes por periodificación</field>
        <field name="parent_id" ref="l10n_es.account_group_4"/>
    </record>
    <record id="account_group_480" model="account.group">
        <field name="code_prefix">480</field>
        <field name="name">Gastos anticipados</field>
        <field name="parent_id" ref="l10n_es.account_group_48"/>
    </record>
    <record id="account_group_485" model="account.group">
        <field name="code_prefix">485</field>
        <field name="name">Ingresos anticipados</field>
        <field name="parent_id" ref="l10n_es.account_group_48"/>
    </record>
    <record id="account_group_49" model="account.group">
        <field name="code_prefix">49</field>
        <field name="name">Deterioro de valor de créditos comerciales y provisiones a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_4"/>
    </record>
    <record id="account_group_490" model="account.group">
        <field name="code_prefix">490</field>
        <field name="name">Deterioro de valor de créditos por operaciones comerciales</field>
        <field name="parent_id" ref="l10n_es.account_group_49"/>
    </record>
    <record id="account_group_493" model="account.group">
        <field name="code_prefix">493</field>
        <field name="name">Deterioro de valor de créditos por operaciones comerciales con partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_49"/>
    </record>
    <record id="account_group_4933" model="account.group">
        <field name="code_prefix">4933</field>
        <field name="name">Deterioro de valor de créditos por operaciones comerciales con empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_493"/>
    </record>
    <record id="account_group_4934" model="account.group">
        <field name="code_prefix">4934</field>
        <field name="name">Deterioro de valor de créditos por operaciones comerciales con empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_493"/>
    </record>
    <record id="account_group_4935" model="account.group">
        <field name="code_prefix">4935</field>
        <field name="name">Deterioro de valor de créditos por operaciones comerciales con otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_493"/>
    </record>
    <record id="account_group_499" model="account.group">
        <field name="code_prefix">499</field>
        <field name="name">Provisiones por operaciones comerciales</field>
        <field name="parent_id" ref="l10n_es.account_group_49"/>
    </record>
    <record id="account_group_4994" model="account.group">
        <field name="code_prefix">4994</field>
        <field name="name">Provisión por contratos onerosos</field>
        <field name="parent_id" ref="l10n_es.account_group_499"/>
    </record>
    <record id="account_group_4999" model="account.group">
        <field name="code_prefix">4999</field>
        <field name="name">Provisión para otras operaciones comerciales</field>
        <field name="parent_id" ref="l10n_es.account_group_499"/>
    </record>
    <record id="account_group_5" model="account.group">
        <field name="code_prefix">5</field>
        <field name="name">Cuentas financieras</field>
    </record>
    <record id="account_group_50" model="account.group">
        <field name="code_prefix">50</field>
        <field name="name">Empréstitos, deudas con carácterísticas especiales y otras emisiones análogas a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_5"/>
    </record>
    <record id="account_group_500" model="account.group">
        <field name="code_prefix">500</field>
        <field name="name">Obligaciones y bonos a corto plazo Obligaciones y bonos convertibles a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_50"/>
    </record>
    <record id="account_group_501" model="account.group">
        <field name="code_prefix">501</field>
        <field name="name">Obligaciones y bonos convertibles a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_50"/>
    </record>
    <record id="account_group_502" model="account.group">
        <field name="code_prefix">502</field>
        <field name="name">Acciones o participaciones a corto plazo consideradas como pasivos financieros</field>
        <field name="parent_id" ref="l10n_es.account_group_50"/>
    </record>
    <record id="account_group_505" model="account.group">
        <field name="code_prefix">505</field>
        <field name="name">Deudas representadas en otros valores negociables a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_50"/>
    </record>
    <record id="account_group_506" model="account.group">
        <field name="code_prefix">506</field>
        <field name="name">Intereses a corto plazo de empréstitos y otras emisiones análogas</field>
        <field name="parent_id" ref="l10n_es.account_group_50"/>
    </record>
    <record id="account_group_507" model="account.group">
        <field name="code_prefix">507</field>
        <field name="name">Dividendos de acciones o participaciones consideradas como pasivos financieros</field>
        <field name="parent_id" ref="l10n_es.account_group_50"/>
    </record>
    <record id="account_group_509" model="account.group">
        <field name="code_prefix">509</field>
        <field name="name">Valores negociables amortizados</field>
        <field name="parent_id" ref="l10n_es.account_group_50"/>
    </record>
    <record id="account_group_5090" model="account.group">
        <field name="code_prefix">5090</field>
        <field name="name">Obligaciones y bonos amortizados</field>
        <field name="parent_id" ref="l10n_es.account_group_509"/>
    </record>
    <record id="account_group_5091" model="account.group">
        <field name="code_prefix">5091</field>
        <field name="name">Obligaciones y bonos convertibles amortizados</field>
        <field name="parent_id" ref="l10n_es.account_group_509"/>
    </record>
    <record id="account_group_5095" model="account.group">
        <field name="code_prefix">5095</field>
        <field name="name">Otros valores negociables amortizados</field>
        <field name="parent_id" ref="l10n_es.account_group_509"/>
    </record>
    <record id="account_group_51" model="account.group">
        <field name="code_prefix">51</field>
        <field name="name">Deudas a corto plazo con partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_5"/>
    </record>
    <record id="account_group_510" model="account.group">
        <field name="code_prefix">510</field>
        <field name="name">Deudas a corto plazo con entidades de crédito vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_51"/>
    </record>
    <record id="account_group_5103" model="account.group">
        <field name="code_prefix">5103</field>
        <field name="name">Deudas a corto plazo con entidades de crédito, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_510"/>
    </record>
    <record id="account_group_5104" model="account.group">
        <field name="code_prefix">5104</field>
        <field name="name">Deudas a corto plazo con entidades de crédito, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_510"/>
    </record>
    <record id="account_group_5105" model="account.group">
        <field name="code_prefix">5105</field>
        <field name="name">Deudas a corto plazo con otras entidades de crédito vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_510"/>
    </record>
    <record id="account_group_511" model="account.group">
        <field name="code_prefix">511</field>
        <field name="name">Proveedores de inmovilizado a corto plazo, partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_51"/>
    </record>
    <record id="account_group_5113" model="account.group">
        <field name="code_prefix">5113</field>
        <field name="name">Proveedores de inmovilizado a corto plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_511"/>
    </record>
    <record id="account_group_5114" model="account.group">
        <field name="code_prefix">5114</field>
        <field name="name">Proveedores de inmovilizado a corto plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_511"/>
    </record>
    <record id="account_group_5115" model="account.group">
        <field name="code_prefix">5115</field>
        <field name="name">Proveedores de inmovilizado a corto plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_511"/>
    </record>
    <record id="account_group_512" model="account.group">
        <field name="code_prefix">512</field>
        <field name="name">Acreedores por arrendamiento financiero a corto plazo, partes vinculadas.</field>
        <field name="parent_id" ref="l10n_es.account_group_51"/>
    </record>
    <record id="account_group_5123" model="account.group">
        <field name="code_prefix">5123</field>
        <field name="name">Acreedores por arrendamiento financiero a corto plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_512"/>
    </record>
    <record id="account_group_5124" model="account.group">
        <field name="code_prefix">5124</field>
        <field name="name">Acreedores por arrendamiento financiero a corto plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_512"/>
    </record>
    <record id="account_group_5125" model="account.group">
        <field name="code_prefix">5125</field>
        <field name="name">Acreedores por arrendamiento financiero a corto plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_512"/>
    </record>
    <record id="account_group_513" model="account.group">
        <field name="code_prefix">513</field>
        <field name="name">Otras deudas a corto plazo con partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_51"/>
    </record>
    <record id="account_group_5133" model="account.group">
        <field name="code_prefix">5133</field>
        <field name="name">Otras deudas a corto plazo con empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_513"/>
    </record>
    <record id="account_group_5134" model="account.group">
        <field name="code_prefix">5134</field>
        <field name="name">Otras deudas a corto plazo con empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_513"/>
    </record>
    <record id="account_group_5135" model="account.group">
        <field name="code_prefix">5135</field>
        <field name="name">Otras deudas a corto plazo con otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_513"/>
    </record>
    <record id="account_group_514" model="account.group">
        <field name="code_prefix">514</field>
        <field name="name">Intereses a corto plazo de deudas con partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_51"/>
    </record>
    <record id="account_group_5143" model="account.group">
        <field name="code_prefix">5143</field>
        <field name="name">Intereses a corto plazo de deudas, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_514"/>
    </record>
    <record id="account_group_5144" model="account.group">
        <field name="code_prefix">5144</field>
        <field name="name">Intereses a corto plazo de deudas, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_514"/>
    </record>
    <record id="account_group_5145" model="account.group">
        <field name="code_prefix">5145</field>
        <field name="name">Intereses a corto plazo de deudas, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_514"/>
    </record>
    <record id="account_group_52" model="account.group">
        <field name="code_prefix">52</field>
        <field name="name">Deudas a corto plazo por préstamos recibidos y otros conceptos</field>
        <field name="parent_id" ref="l10n_es.account_group_5"/>
    </record>
    <record id="account_group_520" model="account.group">
        <field name="code_prefix">520</field>
        <field name="name">Deudas a corto plazo con entidades de crédito</field>
        <field name="parent_id" ref="l10n_es.account_group_52"/>
    </record>
    <record id="account_group_5200" model="account.group">
        <field name="code_prefix">5200</field>
        <field name="name">Préstamos a corto plazo de entidades de crédito</field>
        <field name="parent_id" ref="l10n_es.account_group_520"/>
    </record>
    <record id="account_group_5201" model="account.group">
        <field name="code_prefix">5201</field>
        <field name="name">Deudas a corto plazo por crédito dispuesto</field>
        <field name="parent_id" ref="l10n_es.account_group_520"/>
    </record>
    <record id="account_group_5208" model="account.group">
        <field name="code_prefix">5208</field>
        <field name="name">Deudas por efectos descontados</field>
        <field name="parent_id" ref="l10n_es.account_group_520"/>
    </record>
    <record id="account_group_5209" model="account.group">
        <field name="code_prefix">5209</field>
        <field name="name">Deudas por operaciones de «factoring»</field>
        <field name="parent_id" ref="l10n_es.account_group_520"/>
    </record>
    <record id="account_group_521" model="account.group">
        <field name="code_prefix">521</field>
        <field name="name">Deudas a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_52"/>
    </record>
    <record id="account_group_522" model="account.group">
        <field name="code_prefix">522</field>
        <field name="name">Deudas a corto plazo transformables en subvenciones, donaciones y legados</field>
        <field name="parent_id" ref="l10n_es.account_group_52"/>
    </record>
    <record id="account_group_523" model="account.group">
        <field name="code_prefix">523</field>
        <field name="name">Proveedores de inmovilizado a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_52"/>
    </record>
    <record id="account_group_524" model="account.group">
        <field name="code_prefix">524</field>
        <field name="name">Acreedores por arrendamiento financiero a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_52"/>
    </record>
    <record id="account_group_525" model="account.group">
        <field name="code_prefix">525</field>
        <field name="name">Efectos a pagar a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_52"/>
    </record>
    <record id="account_group_526" model="account.group">
        <field name="code_prefix">526</field>
        <field name="name">Dividendo activo a pagar</field>
        <field name="parent_id" ref="l10n_es.account_group_52"/>
    </record>
    <record id="account_group_527" model="account.group">
        <field name="code_prefix">527</field>
        <field name="name">Intereses a corto plazo de deudas con entidades de crédito</field>
        <field name="parent_id" ref="l10n_es.account_group_52"/>
    </record>
    <record id="account_group_528" model="account.group">
        <field name="code_prefix">528</field>
        <field name="name">Intereses a corto plazo de deudas</field>
        <field name="parent_id" ref="l10n_es.account_group_52"/>
    </record>
    <record id="account_group_529" model="account.group">
        <field name="code_prefix">529</field>
        <field name="name">Provisiones a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_52"/>
    </record>
    <record id="account_group_5290" model="account.group">
        <field name="code_prefix">5290</field>
        <field name="name">Provisión a corto plazo por retribuciones al personal</field>
        <field name="parent_id" ref="l10n_es.account_group_529"/>
    </record>
    <record id="account_group_5291" model="account.group">
        <field name="code_prefix">5291</field>
        <field name="name">Provisión a corto plazo para impuestos</field>
        <field name="parent_id" ref="l10n_es.account_group_529"/>
    </record>
    <record id="account_group_5292" model="account.group">
        <field name="code_prefix">5292</field>
        <field name="name">Provisión a corto plazo para otras responsabilidades</field>
        <field name="parent_id" ref="l10n_es.account_group_529"/>
    </record>
    <record id="account_group_5293" model="account.group">
        <field name="code_prefix">5293</field>
        <field name="name">Provisión a corto plazo por desmantelamiento, retiro o rehabilitación del inmovilizado</field>
        <field name="parent_id" ref="l10n_es.account_group_529"/>
    </record>
    <record id="account_group_5295" model="account.group">
        <field name="code_prefix">5295</field>
        <field name="name">Provisión a corto plazo para actuaciones medioambientales</field>
        <field name="parent_id" ref="l10n_es.account_group_529"/>
    </record>
    <record id="account_group_5296" model="account.group">
        <field name="code_prefix">5296</field>
        <field name="name">Provisión a corto plazo para reestructuraciones</field>
        <field name="parent_id" ref="l10n_es.account_group_529"/>
    </record>
    <record id="account_group_5297" model="account.group">
        <field name="code_prefix">5297</field>
        <field name="name">Provisión a corto plazo por transacciones con pagos basados en instrumentos de patrimonio</field>
        <field name="parent_id" ref="l10n_es.account_group_529"/>
    </record>
    <record id="account_group_53" model="account.group">
        <field name="code_prefix">53</field>
        <field name="name">Inversiones financieras a corto plazo en partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_5"/>
    </record>
    <record id="account_group_530" model="account.group">
        <field name="code_prefix">530</field>
        <field name="name">Participaciones a corto plazo en partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_53"/>
    </record>
    <record id="account_group_5303" model="account.group">
        <field name="code_prefix">5303</field>
        <field name="name">Participaciones a corto plazo, en empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_530"/>
    </record>
    <record id="account_group_5304" model="account.group">
        <field name="code_prefix">5304</field>
        <field name="name">Participaciones a corto plazo, en empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_530"/>
    </record>
    <record id="account_group_5305" model="account.group">
        <field name="code_prefix">5305</field>
        <field name="name">Participaciones a corto plazo, en otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_530"/>
    </record>
    <record id="account_group_531" model="account.group">
        <field name="code_prefix">531</field>
        <field name="name">Valores representativos de deuda a corto plazo de partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_53"/>
    </record>
    <record id="account_group_5313" model="account.group">
        <field name="code_prefix">5313</field>
        <field name="name">Valores representativos de deuda a corto plazo de empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_531"/>
    </record>
    <record id="account_group_5314" model="account.group">
        <field name="code_prefix">5314</field>
        <field name="name">Valores representativos de deuda a corto plazo de empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_531"/>
    </record>
    <record id="account_group_5315" model="account.group">
        <field name="code_prefix">5315</field>
        <field name="name">Valores representativos de deuda a corto plazo de otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_531"/>
    </record>
    <record id="account_group_532" model="account.group">
        <field name="code_prefix">532</field>
        <field name="name">Créditos a corto plazo a partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_53"/>
    </record>
    <record id="account_group_5323" model="account.group">
        <field name="code_prefix">5323</field>
        <field name="name">Créditos a corto plazo a empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_532"/>
    </record>
    <record id="account_group_5324" model="account.group">
        <field name="code_prefix">5324</field>
        <field name="name">Créditos a corto plazo a empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_532"/>
    </record>
    <record id="account_group_5325" model="account.group">
        <field name="code_prefix">5325</field>
        <field name="name">Créditos a corto plazo a otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_532"/>
    </record>
    <record id="account_group_533" model="account.group">
        <field name="code_prefix">533</field>
        <field name="name">Intereses a corto plazo de valores representativos de deuda de partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_53"/>
    </record>
    <record id="account_group_5333" model="account.group">
        <field name="code_prefix">5333</field>
        <field name="name">Intereses a corto plazo de valores representativos de deuda de empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_533"/>
    </record>
    <record id="account_group_5334" model="account.group">
        <field name="code_prefix">5334</field>
        <field name="name">Intereses a corto plazo de valores representativos de deuda de empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_533"/>
    </record>
    <record id="account_group_5335" model="account.group">
        <field name="code_prefix">5335</field>
        <field name="name">Intereses a corto plazo de valores representativos de deuda de otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_533"/>
    </record>
    <record id="account_group_534" model="account.group">
        <field name="code_prefix">534</field>
        <field name="name">Intereses a corto plazo de créditos a partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_53"/>
    </record>
    <record id="account_group_5343" model="account.group">
        <field name="code_prefix">5343</field>
        <field name="name">Intereses a corto plazo de créditos a empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_534"/>
    </record>
    <record id="account_group_5344" model="account.group">
        <field name="code_prefix">5344</field>
        <field name="name">Intereses a corto plazo de créditos a empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_534"/>
    </record>
    <record id="account_group_5345" model="account.group">
        <field name="code_prefix">5345</field>
        <field name="name">Intereses a corto plazo de créditos a otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_534"/>
    </record>
    <record id="account_group_535" model="account.group">
        <field name="code_prefix">535</field>
        <field name="name">Dividendo a cobrar de inversiones financieras en partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_53"/>
    </record>
    <record id="account_group_5353" model="account.group">
        <field name="code_prefix">5353</field>
        <field name="name">Dividendo a cobrar de empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_535"/>
    </record>
    <record id="account_group_5354" model="account.group">
        <field name="code_prefix">5354</field>
        <field name="name">Dividendo a cobrar de empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_535"/>
    </record>
    <record id="account_group_5355" model="account.group">
        <field name="code_prefix">5355</field>
        <field name="name">Dividendo a cobrar de otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_535"/>
    </record>
    <record id="account_group_539" model="account.group">
        <field name="code_prefix">539</field>
        <field name="name">Desembolsos pendientes sobre participaciones a corto plazo en partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_53"/>
    </record>
    <record id="account_group_5393" model="account.group">
        <field name="code_prefix">5393</field>
        <field name="name">Desembolsos pendientes sobre participaciones a corto plazo en empresas del grupo.</field>
        <field name="parent_id" ref="l10n_es.account_group_539"/>
    </record>
    <record id="account_group_5394" model="account.group">
        <field name="code_prefix">5394</field>
        <field name="name">Desembolsos pendientes sobre participaciones a corto plazo en empresas asociadas.</field>
        <field name="parent_id" ref="l10n_es.account_group_539"/>
    </record>
    <record id="account_group_5395" model="account.group">
        <field name="code_prefix">5395</field>
        <field name="name">Desembolsos pendientes sobre participaciones a corto plazo en otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_539"/>
    </record>
    <record id="account_group_54" model="account.group">
        <field name="code_prefix">54</field>
        <field name="name">Otras inversiones financieras a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_5"/>
    </record>
    <record id="account_group_540" model="account.group">
        <field name="code_prefix">540</field>
        <field name="name">Inversiones financieras a corto plazo en instrumentos de patrimonio</field>
        <field name="parent_id" ref="l10n_es.account_group_54"/>
    </record>
    <record id="account_group_541" model="account.group">
        <field name="code_prefix">541</field>
        <field name="name">Valores representativos de deuda a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_54"/>
    </record>
    <record id="account_group_542" model="account.group">
        <field name="code_prefix">542</field>
        <field name="name">Créditos a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_54"/>
    </record>
    <record id="account_group_543" model="account.group">
        <field name="code_prefix">543</field>
        <field name="name">Créditos a corto plazo por enajenación de inmovilizado</field>
        <field name="parent_id" ref="l10n_es.account_group_54"/>
    </record>
    <record id="account_group_544" model="account.group">
        <field name="code_prefix">544</field>
        <field name="name">Créditos a corto plazo al personal</field>
        <field name="parent_id" ref="l10n_es.account_group_54"/>
    </record>
    <record id="account_group_545" model="account.group">
        <field name="code_prefix">545</field>
        <field name="name">Dividendo a cobrar</field>
        <field name="parent_id" ref="l10n_es.account_group_54"/>
    </record>
    <record id="account_group_546" model="account.group">
        <field name="code_prefix">546</field>
        <field name="name">Intereses a corto plazo de valores representativos de deudas</field>
        <field name="parent_id" ref="l10n_es.account_group_54"/>
    </record>
    <record id="account_group_547" model="account.group">
        <field name="code_prefix">547</field>
        <field name="name">Intereses a corto plazo de créditos</field>
        <field name="parent_id" ref="l10n_es.account_group_54"/>
    </record>
    <record id="account_group_548" model="account.group">
        <field name="code_prefix">548</field>
        <field name="name">Imposiciones a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_54"/>
    </record>
    <record id="account_group_549" model="account.group">
        <field name="code_prefix">549</field>
        <field name="name">Desembolsos pendientes sobre participaciones en el patrimonio neto a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_54"/>
    </record>
    <record id="account_group_55" model="account.group">
        <field name="code_prefix">55</field>
        <field name="name">Otras cuentas no bancarias</field>
        <field name="parent_id" ref="l10n_es.account_group_5"/>
    </record>
    <record id="account_group_550" model="account.group">
        <field name="code_prefix">550</field>
        <field name="name">Titular de la explotación</field>
        <field name="parent_id" ref="l10n_es.account_group_55"/>
    </record>
    <record id="account_group_551" model="account.group">
        <field name="code_prefix">551</field>
        <field name="name">Cuenta corriente con socios y administradores</field>
        <field name="parent_id" ref="l10n_es.account_group_55"/>
    </record>
    <record id="account_group_552" model="account.group">
        <field name="code_prefix">552</field>
        <field name="name">Cuenta corriente con otras personas y entidades vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_55"/>
    </record>
    <record id="account_group_5523" model="account.group">
        <field name="code_prefix">5523</field>
        <field name="name">Cuenta corriente con empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_552"/>
    </record>
    <record id="account_group_5524" model="account.group">
        <field name="code_prefix">5524</field>
        <field name="name">Cuenta corriente con empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_552"/>
    </record>
    <record id="account_group_5525" model="account.group">
        <field name="code_prefix">5525</field>
        <field name="name">Cuenta corriente con otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_552"/>
    </record>
    <record id="account_group_553" model="account.group">
        <field name="code_prefix">553</field>
        <field name="name">Cuentas corrientes en fusiones y escisiones</field>
        <field name="parent_id" ref="l10n_es.account_group_55"/>
    </record>
    <record id="account_group_5530" model="account.group">
        <field name="code_prefix">5530</field>
        <field name="name">Socios de sociedad disuelta</field>
        <field name="parent_id" ref="l10n_es.account_group_553"/>
    </record>
    <record id="account_group_5531" model="account.group">
        <field name="code_prefix">5531</field>
        <field name="name">Socios, cuenta de fusión</field>
        <field name="parent_id" ref="l10n_es.account_group_553"/>
    </record>
    <record id="account_group_5532" model="account.group">
        <field name="code_prefix">5532</field>
        <field name="name">Socios de sociedad escindida</field>
        <field name="parent_id" ref="l10n_es.account_group_553"/>
    </record>
    <record id="account_group_5533" model="account.group">
        <field name="code_prefix">5533</field>
        <field name="name">Socios, cuenta de escisión</field>
        <field name="parent_id" ref="l10n_es.account_group_553"/>
    </record>
    <record id="account_group_554" model="account.group">
        <field name="code_prefix">554</field>
        <field name="name">Cuenta corriente con uniones temporales de empresas y comunidades de bienes</field>
        <field name="parent_id" ref="l10n_es.account_group_55"/>
    </record>
    <record id="account_group_555" model="account.group">
        <field name="code_prefix">555</field>
        <field name="name">Partidas pendientes de aplicación</field>
        <field name="parent_id" ref="l10n_es.account_group_55"/>
    </record>
    <record id="account_group_556" model="account.group">
        <field name="code_prefix">556</field>
        <field name="name">Desembolsos exigidos sobre participaciones en el patrimonio neto</field>
        <field name="parent_id" ref="l10n_es.account_group_55"/>
    </record>
    <record id="account_group_5563" model="account.group">
        <field name="code_prefix">5563</field>
        <field name="name">Desembolsos exigidos sobre participaciones, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_556"/>
    </record>
    <record id="account_group_5564" model="account.group">
        <field name="code_prefix">5564</field>
        <field name="name">Desembolsos exigidos sobre participaciones, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_556"/>
    </record>
    <record id="account_group_5565" model="account.group">
        <field name="code_prefix">5565</field>
        <field name="name">Desembolsos exigidos sobre participaciones, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_556"/>
    </record>
    <record id="account_group_5566" model="account.group">
        <field name="code_prefix">5566</field>
        <field name="name">Desembolsos exigidos sobre participaciones de otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_556"/>
    </record>
    <record id="account_group_557" model="account.group">
        <field name="code_prefix">557</field>
        <field name="name">Dividendo activo a cuenta</field>
        <field name="parent_id" ref="l10n_es.account_group_55"/>
    </record>
    <record id="account_group_558" model="account.group">
        <field name="code_prefix">558</field>
        <field name="name">Socios por desembolsos exigidos</field>
        <field name="parent_id" ref="l10n_es.account_group_55"/>
    </record>
    <record id="account_group_5580" model="account.group">
        <field name="code_prefix">5580</field>
        <field name="name">Socios por desembolsos exigidos sobre acciones o participaciones ordinarias</field>
        <field name="parent_id" ref="l10n_es.account_group_558"/>
    </record>
    <record id="account_group_5585" model="account.group">
        <field name="code_prefix">5585</field>
        <field name="name">Socios por desembolsos exigidos sobre acciones o participaciones consideradas como pasivos financieros</field>
        <field name="parent_id" ref="l10n_es.account_group_558"/>
    </record>
    <record id="account_group_559" model="account.group">
        <field name="code_prefix">559</field>
        <field name="name">Derivados financieros a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_55"/>
    </record>
    <record id="account_group_5590" model="account.group">
        <field name="code_prefix">5590</field>
        <field name="name">Activos por derivados financieros a corto plazo, cartera de negociación</field>
        <field name="parent_id" ref="l10n_es.account_group_559"/>
    </record>
    <record id="account_group_5593" model="account.group">
        <field name="code_prefix">5593</field>
        <field name="name">Activos por derivados financieros a corto plazo, instrumentos de cobertura</field>
        <field name="parent_id" ref="l10n_es.account_group_559"/>
    </record>
    <record id="account_group_5595" model="account.group">
        <field name="code_prefix">5595</field>
        <field name="name">Pasivos por derivados financieros a corto plazo, cartera de negociación</field>
        <field name="parent_id" ref="l10n_es.account_group_559"/>
    </record>
    <record id="account_group_5598" model="account.group">
        <field name="code_prefix">5598</field>
        <field name="name">Pasivos por derivados financieros a corto plazo, instrumentos de cobertura</field>
        <field name="parent_id" ref="l10n_es.account_group_559"/>
    </record>
    <record id="account_group_56" model="account.group">
        <field name="code_prefix">56</field>
        <field name="name">Fianzas y depósitos recibidos y constituidos a corto plazo y ajustes por periodificación</field>
        <field name="parent_id" ref="l10n_es.account_group_5"/>
    </record>
    <record id="account_group_560" model="account.group">
        <field name="code_prefix">560</field>
        <field name="name">Fianzas recibidas a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_56"/>
    </record>
    <record id="account_group_561" model="account.group">
        <field name="code_prefix">561</field>
        <field name="name">Depósitos recibidos a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_56"/>
    </record>
    <record id="account_group_565" model="account.group">
        <field name="code_prefix">565</field>
        <field name="name">Fianzas constituidas a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_56"/>
    </record>
    <record id="account_group_566" model="account.group">
        <field name="code_prefix">566</field>
        <field name="name">Depósitos constituidos a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_56"/>
    </record>
    <record id="account_group_567" model="account.group">
        <field name="code_prefix">567</field>
        <field name="name">Intereses pagados por anticipado</field>
        <field name="parent_id" ref="l10n_es.account_group_56"/>
    </record>
    <record id="account_group_568" model="account.group">
        <field name="code_prefix">568</field>
        <field name="name">Intereses cobrados por anticipado</field>
        <field name="parent_id" ref="l10n_es.account_group_56"/>
    </record>
    <record id="account_group_569" model="account.group">
        <field name="code_prefix">569</field>
        <field name="name">Garantías financieras a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_56"/>
    </record>
    <record id="account_group_57" model="account.group">
        <field name="code_prefix">57</field>
        <field name="name">Tesorería</field>
        <field name="parent_id" ref="l10n_es.account_group_5"/>
    </record>
    <record id="account_group_570" model="account.group">
        <field name="code_prefix">570</field>
        <field name="name">Caja, euros</field>
        <field name="parent_id" ref="l10n_es.account_group_57"/>
    </record>
    <record id="account_group_571" model="account.group">
        <field name="code_prefix">571</field>
        <field name="name">Caja, moneda extranjera</field>
        <field name="parent_id" ref="l10n_es.account_group_57"/>
    </record>
    <record id="account_group_572" model="account.group">
        <field name="code_prefix">572</field>
        <field name="name">Bancos e instituciones de crédito c/c vista, euros</field>
        <field name="parent_id" ref="l10n_es.account_group_57"/>
    </record>
    <record id="account_group_573" model="account.group">
        <field name="code_prefix">573</field>
        <field name="name">Bancos e instituciones de crédito c/c vista, moneda extranjera</field>
        <field name="parent_id" ref="l10n_es.account_group_57"/>
    </record>
    <record id="account_group_574" model="account.group">
        <field name="code_prefix">574</field>
        <field name="name">Bancos e instituciones de crédito, cuentas de ahorro, euros</field>
        <field name="parent_id" ref="l10n_es.account_group_57"/>
    </record>
    <record id="account_group_575" model="account.group">
        <field name="code_prefix">575</field>
        <field name="name">Bancos e instituciones de crédito, cuentas de ahorro, moneda extranjera</field>
        <field name="parent_id" ref="l10n_es.account_group_57"/>
    </record>
    <record id="account_group_576" model="account.group">
        <field name="code_prefix">576</field>
        <field name="name">Inversiones a corto plazo de gran liquidez</field>
        <field name="parent_id" ref="l10n_es.account_group_57"/>
    </record>
    <record id="account_group_58" model="account.group">
        <field name="code_prefix">58</field>
        <field name="name">Activos no corrientes mantenidos para la venta y activos y pasivos asociados</field>
        <field name="parent_id" ref="l10n_es.account_group_5"/>
    </record>
    <record id="account_group_580" model="account.group">
        <field name="code_prefix">580</field>
        <field name="name">Inmovilizado</field>
        <field name="parent_id" ref="l10n_es.account_group_58"/>
    </record>
    <record id="account_group_581" model="account.group">
        <field name="code_prefix">581</field>
        <field name="name">Inversiones con personas y entidades vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_58"/>
    </record>
    <record id="account_group_582" model="account.group">
        <field name="code_prefix">582</field>
        <field name="name">Inversiones financieras</field>
        <field name="parent_id" ref="l10n_es.account_group_58"/>
    </record>
    <record id="account_group_583" model="account.group">
        <field name="code_prefix">583</field>
        <field name="name">Existencias, deudores comerciales y otras cuentas a cobrar</field>
        <field name="parent_id" ref="l10n_es.account_group_58"/>
    </record>
    <record id="account_group_584" model="account.group">
        <field name="code_prefix">584</field>
        <field name="name">Otros activos</field>
        <field name="parent_id" ref="l10n_es.account_group_58"/>
    </record>
    <record id="account_group_585" model="account.group">
        <field name="code_prefix">585</field>
        <field name="name">Provisiones</field>
        <field name="parent_id" ref="l10n_es.account_group_58"/>
    </record>
    <record id="account_group_586" model="account.group">
        <field name="code_prefix">586</field>
        <field name="name">Deudas con características especiales</field>
        <field name="parent_id" ref="l10n_es.account_group_58"/>
    </record>
    <record id="account_group_587" model="account.group">
        <field name="code_prefix">587</field>
        <field name="name">Deudas con personas y entidades vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_58"/>
    </record>
    <record id="account_group_588" model="account.group">
        <field name="code_prefix">588</field>
        <field name="name">Acreedores comerciales y otras cuentas a pagar</field>
        <field name="parent_id" ref="l10n_es.account_group_58"/>
    </record>
    <record id="account_group_589" model="account.group">
        <field name="code_prefix">589</field>
        <field name="name">Otros pasivos</field>
        <field name="parent_id" ref="l10n_es.account_group_58"/>
    </record>
    <record id="account_group_59" model="account.group">
        <field name="code_prefix">59</field>
        <field name="name">Deterioro del valor de inversiones financieras a corto plazo y de activos no corrientes mantenidos para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_5"/>
    </record>
    <record id="account_group_593" model="account.group">
        <field name="code_prefix">593</field>
        <field name="name">Deterioro de valor de participaciones a corto plazo en partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_59"/>
    </record>
    <record id="account_group_596" model="account.group">
        <field name="code_prefix">596</field>
        <field name="name">Deterioro de valor de participaciones a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_59"/>
    </record>
    <record id="account_group_5933" model="account.group">
        <field name="code_prefix">5933</field>
        <field name="name">Deterioro de valor de participaciones a corto plazo en empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_593"/>
    </record>
    <record id="account_group_5934" model="account.group">
        <field name="code_prefix">5934</field>
        <field name="name">Deterioro de valor de participaciones a corto plazo en empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_593"/>
    </record>
    <record id="account_group_594" model="account.group">
        <field name="code_prefix">594</field>
        <field name="name">Deterioro de valor de valores representativos de deuda a corto plazo de partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_59"/>
    </record>
    <record id="account_group_5943" model="account.group">
        <field name="code_prefix">5943</field>
        <field name="name">Deterioro de valor de valores representativos de deuda a corto plazo de empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_594"/>
    </record>
    <record id="account_group_5944" model="account.group">
        <field name="code_prefix">5944</field>
        <field name="name">Deterioro de valor de valores representativos de deuda a corto plazo de empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_594"/>
    </record>
    <record id="account_group_5945" model="account.group">
        <field name="code_prefix">5945</field>
        <field name="name">Deterioro de valor de valores representativos de deuda a corto plazo de otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_594"/>
    </record>
    <record id="account_group_595" model="account.group">
        <field name="code_prefix">595</field>
        <field name="name">Deterioro de valor de créditos a corto plazo a partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_59"/>
    </record>
    <record id="account_group_5953" model="account.group">
        <field name="code_prefix">5953</field>
        <field name="name">Deterioro de valor de créditos a corto plazo a empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_595"/>
    </record>
    <record id="account_group_5954" model="account.group">
        <field name="code_prefix">5954</field>
        <field name="name">Deterioro de valor de créditos a corto plazo a empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_595"/>
    </record>
    <record id="account_group_5955" model="account.group">
        <field name="code_prefix">5955</field>
        <field name="name">Deterioro de valor de créditos a corto plazo a otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_595"/>
    </record>
    <record id="account_group_597" model="account.group">
        <field name="code_prefix">597</field>
        <field name="name">Deterioro de valor de valores representativos de deuda a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_59"/>
    </record>
    <record id="account_group_598" model="account.group">
        <field name="code_prefix">598</field>
        <field name="name">Deterioro de valor de créditos a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_59"/>
    </record>
    <record id="account_group_599" model="account.group">
        <field name="code_prefix">599</field>
        <field name="name">Deterioro de valor de activos no corrientes mantenidos para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_59"/>
    </record>
    <record id="account_group_5990" model="account.group">
        <field name="code_prefix">5990</field>
        <field name="name">Deterioro de valor de inmovilizado no corriente mantenido para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_599"/>
    </record>
    <record id="account_group_5991" model="account.group">
        <field name="code_prefix">5991</field>
        <field name="name">Deterioro de valor de inversiones con personas y entidades vinculadas no corrientes mantenidas para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_599"/>
    </record>
    <record id="account_group_5992" model="account.group">
        <field name="code_prefix">5992</field>
        <field name="name">Deterioro de valor de inversiones financieras no corrientes mantenidas para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_599"/>
    </record>
    <record id="account_group_5993" model="account.group">
        <field name="code_prefix">5993</field>
        <field name="name">Deterioro de valor de existencias, deudores comerciales y otras cuentas a cobrar integrados en un grupo enajenable mantenido para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_599"/>
    </record>
    <record id="account_group_5994" model="account.group">
        <field name="code_prefix">5994</field>
        <field name="name">Deterioro de valor de otros activos mantenidos para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_599"/>
    </record>
        <record id="account_group_6" model="account.group">
        <field name="code_prefix">6</field>
        <field name="name">Compras y gastos</field>
    </record>
    <record id="account_group_60" model="account.group">
        <field name="code_prefix">60</field>
        <field name="name">Compras</field>
        <field name="parent_id" ref="l10n_es.account_group_6"/>
    </record>
    <record id="account_group_600" model="account.group">
        <field name="code_prefix">600</field>
        <field name="name">Compras de mercaderías</field>
        <field name="parent_id" ref="l10n_es.account_group_60"/>
    </record>
    <record id="account_group_601" model="account.group">
        <field name="code_prefix">601</field>
        <field name="name">Compras de materias primas</field>
        <field name="parent_id" ref="l10n_es.account_group_60"/>
    </record>
    <record id="account_group_602" model="account.group">
        <field name="code_prefix">602</field>
        <field name="name">Compras de otros aprovisionamientos</field>
        <field name="parent_id" ref="l10n_es.account_group_60"/>
    </record>
    <record id="account_group_606" model="account.group">
        <field name="code_prefix">606</field>
        <field name="name">Descuentos sobre compras por pronto pago</field>
        <field name="parent_id" ref="l10n_es.account_group_60"/>
    </record>
    <record id="account_group_6060" model="account.group">
        <field name="code_prefix">6060</field>
        <field name="name">Descuentos sobre compras por pronto pago de mercaderías</field>
        <field name="parent_id" ref="l10n_es.account_group_606"/>
    </record>
    <record id="account_group_6061" model="account.group">
        <field name="code_prefix">6061</field>
        <field name="name">Descuentos sobre compras por pronto pago de materias primas</field>
        <field name="parent_id" ref="l10n_es.account_group_606"/>
    </record>
    <record id="account_group_6062" model="account.group">
        <field name="code_prefix">6062</field>
        <field name="name">Descuentos sobre compras por pronto pago de otros aprovisionamientos</field>
        <field name="parent_id" ref="l10n_es.account_group_606"/>
    </record>
    <record id="account_group_607" model="account.group">
        <field name="code_prefix">607</field>
        <field name="name">Trabajos realizados por otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_60"/>
    </record>
    <record id="account_group_608" model="account.group">
        <field name="code_prefix">608</field>
        <field name="name">Devoluciones de compras y operaciones similares</field>
        <field name="parent_id" ref="l10n_es.account_group_60"/>
    </record>
    <record id="account_group_6080" model="account.group">
        <field name="code_prefix">6080</field>
        <field name="name">Devoluciones de compras de mercaderías</field>
        <field name="parent_id" ref="l10n_es.account_group_608"/>
    </record>
    <record id="account_group_6081" model="account.group">
        <field name="code_prefix">6081</field>
        <field name="name">Devoluciones de compras de materias primas</field>
        <field name="parent_id" ref="l10n_es.account_group_608"/>
    </record>
    <record id="account_group_6082" model="account.group">
        <field name="code_prefix">6082</field>
        <field name="name">Devoluciones de compras de otros aprovisionamientos</field>
        <field name="parent_id" ref="l10n_es.account_group_608"/>
    </record>
    <record id="account_group_609" model="account.group">
        <field name="code_prefix">609</field>
        <field name="name">«Rappels» por compras</field>
        <field name="parent_id" ref="l10n_es.account_group_60"/>
    </record>
    <record id="account_group_6090" model="account.group">
        <field name="code_prefix">6090</field>
        <field name="name">«Rappels» por compras de mercaderías</field>
        <field name="parent_id" ref="l10n_es.account_group_609"/>
    </record>
    <record id="account_group_6091" model="account.group">
        <field name="code_prefix">6091</field>
        <field name="name">«Rappels» por compras de materias primas</field>
        <field name="parent_id" ref="l10n_es.account_group_609"/>
    </record>
    <record id="account_group_6092" model="account.group">
        <field name="code_prefix">6092</field>
        <field name="name">«Rappels» por compras de otros aprovisionamientos</field>
        <field name="parent_id" ref="l10n_es.account_group_609"/>
    </record>
    <record id="account_group_61" model="account.group">
        <field name="code_prefix">61</field>
        <field name="name">Variación de existencias</field>
        <field name="parent_id" ref="l10n_es.account_group_6"/>
    </record>
    <record id="account_group_610" model="account.group">
        <field name="code_prefix">610</field>
        <field name="name">Variación de existencias de mercaderías</field>
        <field name="parent_id" ref="l10n_es.account_group_61"/>
    </record>
    <record id="account_group_611" model="account.group">
        <field name="code_prefix">611</field>
        <field name="name">Variación de existencias de materias primas</field>
        <field name="parent_id" ref="l10n_es.account_group_61"/>
    </record>
    <record id="account_group_612" model="account.group">
        <field name="code_prefix">612</field>
        <field name="name">Variación de existencias de otros aprovisionamientos</field>
        <field name="parent_id" ref="l10n_es.account_group_61"/>
    </record>
    <record id="account_group_62" model="account.group">
        <field name="code_prefix">62</field>
        <field name="name">Servicios exteriores</field>
        <field name="parent_id" ref="l10n_es.account_group_6"/>
    </record>
    <record id="account_group_620" model="account.group">
        <field name="code_prefix">620</field>
        <field name="name">Gastos en investigación y desarrollo del ejercicio</field>
        <field name="parent_id" ref="l10n_es.account_group_62"/>
    </record>
    <record id="account_group_621" model="account.group">
        <field name="code_prefix">621</field>
        <field name="name">Arrendamientos y cánones</field>
        <field name="parent_id" ref="l10n_es.account_group_62"/>
    </record>
    <record id="account_group_622" model="account.group">
        <field name="code_prefix">622</field>
        <field name="name">Reparaciones y conservación</field>
        <field name="parent_id" ref="l10n_es.account_group_62"/>
    </record>
    <record id="account_group_623" model="account.group">
        <field name="code_prefix">623</field>
        <field name="name">Servicios de profesionales independientes</field>
        <field name="parent_id" ref="l10n_es.account_group_62"/>
    </record>
    <record id="account_group_624" model="account.group">
        <field name="code_prefix">624</field>
        <field name="name">Transportes</field>
        <field name="parent_id" ref="l10n_es.account_group_62"/>
    </record>
    <record id="account_group_625" model="account.group">
        <field name="code_prefix">625</field>
        <field name="name">Primas de seguros</field>
        <field name="parent_id" ref="l10n_es.account_group_62"/>
    </record>
    <record id="account_group_626" model="account.group">
        <field name="code_prefix">626</field>
        <field name="name">Servicios bancarios y similares</field>
        <field name="parent_id" ref="l10n_es.account_group_62"/>
    </record>
    <record id="account_group_627" model="account.group">
        <field name="code_prefix">627</field>
        <field name="name">Publicidad, propaganda y relaciones públicas</field>
        <field name="parent_id" ref="l10n_es.account_group_62"/>
    </record>
    <record id="account_group_628" model="account.group">
        <field name="code_prefix">628</field>
        <field name="name">Suministros</field>
        <field name="parent_id" ref="l10n_es.account_group_62"/>
    </record>
    <record id="account_group_629" model="account.group">
        <field name="code_prefix">629</field>
        <field name="name">Otros servicios</field>
        <field name="parent_id" ref="l10n_es.account_group_62"/>
    </record>
    <record id="account_group_63" model="account.group">
        <field name="code_prefix">63</field>
        <field name="name">Tributos</field>
        <field name="parent_id" ref="l10n_es.account_group_6"/>
    </record>
    <record id="account_group_630" model="account.group">
        <field name="code_prefix">630</field>
        <field name="name">Impuesto sobre beneficios</field>
        <field name="parent_id" ref="l10n_es.account_group_63"/>
    </record>
    <record id="account_group_6300" model="account.group">
        <field name="code_prefix">6300</field>
        <field name="name">Impuesto corriente</field>
        <field name="parent_id" ref="l10n_es.account_group_630"/>
    </record>
    <record id="account_group_6301" model="account.group">
        <field name="code_prefix">6301</field>
        <field name="name">Impuesto diferido</field>
        <field name="parent_id" ref="l10n_es.account_group_630"/>
    </record>
    <record id="account_group_631" model="account.group">
        <field name="code_prefix">631</field>
        <field name="name">Otros tributos</field>
        <field name="parent_id" ref="l10n_es.account_group_63"/>
    </record>
    <record id="account_group_633" model="account.group">
        <field name="code_prefix">633</field>
        <field name="name">Ajustes negativos en la imposición sobre beneficios</field>
        <field name="parent_id" ref="l10n_es.account_group_63"/>
    </record>
    <record id="account_group_634" model="account.group">
        <field name="code_prefix">634</field>
        <field name="name">Ajustes negativos en la imposición indirecta</field>
        <field name="parent_id" ref="l10n_es.account_group_63"/>
    </record>
    <record id="account_group_6341" model="account.group">
        <field name="code_prefix">6341</field>
        <field name="name">Ajustes negativos en IVA de activo corriente</field>
        <field name="parent_id" ref="l10n_es.account_group_634"/>
    </record>
    <record id="account_group_6342" model="account.group">
        <field name="code_prefix">6342</field>
        <field name="name">Ajustes negativos en IVA de inversiones</field>
        <field name="parent_id" ref="l10n_es.account_group_634"/>
    </record>
    <record id="account_group_636" model="account.group">
        <field name="code_prefix">636</field>
        <field name="name">Devolución de impuestos</field>
        <field name="parent_id" ref="l10n_es.account_group_63"/>
    </record>
    <record id="account_group_638" model="account.group">
        <field name="code_prefix">638</field>
        <field name="name">Ajustes positivos en la imposición sobre beneficios</field>
        <field name="parent_id" ref="l10n_es.account_group_63"/>
    </record>
    <record id="account_group_639" model="account.group">
        <field name="code_prefix">639</field>
        <field name="name">Ajustes positivos en la imposición indirecta</field>
        <field name="parent_id" ref="l10n_es.account_group_63"/>
    </record>
    <record id="account_group_6391" model="account.group">
        <field name="code_prefix">6391</field>
        <field name="name">Ajustes positivos en IVA de activo corriente</field>
        <field name="parent_id" ref="l10n_es.account_group_639"/>
    </record>
    <record id="account_group_6392" model="account.group">
        <field name="code_prefix">6392</field>
        <field name="name">Ajustes positivos en IVA de inversiones</field>
        <field name="parent_id" ref="l10n_es.account_group_639"/>
    </record>
    <record id="account_group_64" model="account.group">
        <field name="code_prefix">64</field>
        <field name="name">Gastos de personal</field>
        <field name="parent_id" ref="l10n_es.account_group_6"/>
    </record>
    <record id="account_group_640" model="account.group">
        <field name="code_prefix">640</field>
        <field name="name">Sueldos y salarios</field>
        <field name="parent_id" ref="l10n_es.account_group_64"/>
    </record>
    <record id="account_group_641" model="account.group">
        <field name="code_prefix">641</field>
        <field name="name">Indemnizaciones</field>
        <field name="parent_id" ref="l10n_es.account_group_64"/>
    </record>
    <record id="account_group_642" model="account.group">
        <field name="code_prefix">642</field>
        <field name="name">Seguridad Social a cargo de la empresa</field>
        <field name="parent_id" ref="l10n_es.account_group_64"/>
    </record>
    <record id="account_group_643" model="account.group">
        <field name="code_prefix">643</field>
        <field name="name">Retribuciones a largo plazo mediante sistemas de aportación definida</field>
        <field name="parent_id" ref="l10n_es.account_group_64"/>
    </record>
    <record id="account_group_644" model="account.group">
        <field name="code_prefix">644</field>
        <field name="name">Retribuciones a largo plazo mediante sistemas de prestación definida</field>
        <field name="parent_id" ref="l10n_es.account_group_64"/>
    </record>
    <record id="account_group_6440" model="account.group">
        <field name="code_prefix">6440</field>
        <field name="name">Contribuciones anuales</field>
        <field name="parent_id" ref="l10n_es.account_group_644"/>
    </record>
    <record id="account_group_6442" model="account.group">
        <field name="code_prefix">6442</field>
        <field name="name">Otros costes</field>
        <field name="parent_id" ref="l10n_es.account_group_644"/>
    </record>
    <record id="account_group_645" model="account.group">
        <field name="code_prefix">645</field>
        <field name="name">Retribuciones al personal mediante instrumentos de patrimonio</field>
        <field name="parent_id" ref="l10n_es.account_group_64"/>
    </record>
    <record id="account_group_6450" model="account.group">
        <field name="code_prefix">6450</field>
        <field name="name">Retribuciones al personal liquidados con instrumentos de patrimonio</field>
        <field name="parent_id" ref="l10n_es.account_group_645"/>
    </record>
    <record id="account_group_6457" model="account.group">
        <field name="code_prefix">6457</field>
        <field name="name">Retribuciones al personal liquidados en efectivo basado en instrumentos de patrimonio</field>
        <field name="parent_id" ref="l10n_es.account_group_645"/>
    </record>
    <record id="account_group_649" model="account.group">
        <field name="code_prefix">649</field>
        <field name="name">Otros gastos sociales</field>
        <field name="parent_id" ref="l10n_es.account_group_64"/>
    </record>
    <record id="account_group_65" model="account.group">
        <field name="code_prefix">65</field>
        <field name="name">Otros gastos de gestión</field>
        <field name="parent_id" ref="l10n_es.account_group_6"/>
    </record>
    <record id="account_group_650" model="account.group">
        <field name="code_prefix">650</field>
        <field name="name">Pérdidas de créditos comerciales incobrables</field>
        <field name="parent_id" ref="l10n_es.account_group_65"/>
    </record>
    <record id="account_group_651" model="account.group">
        <field name="code_prefix">651</field>
        <field name="name">Resultados de operaciones en común</field>
        <field name="parent_id" ref="l10n_es.account_group_65"/>
    </record>
    <record id="account_group_6510" model="account.group">
        <field name="code_prefix">6510</field>
        <field name="name">Beneficio transferido (gestor)</field>
        <field name="parent_id" ref="l10n_es.account_group_651"/>
    </record>
    <record id="account_group_6511" model="account.group">
        <field name="code_prefix">6511</field>
        <field name="name">Pérdida soportada (partícipe o asociado no gestor)</field>
        <field name="parent_id" ref="l10n_es.account_group_651"/>
    </record>
    <record id="account_group_653" model="account.group">
        <field name="code_prefix">653</field>
        <field name="name">Compensación de gastos por prestaciones de colaboración</field>
        <field name="parent_id" ref="l10n_es.account_group_65"/>
    </record>
    <record id="account_group_654" model="account.group">
        <field name="code_prefix">654</field>
        <field name="name">Reembolsos de gastos al órgano de gobierno</field>
        <field name="parent_id" ref="l10n_es.account_group_65"/>
    </record>
    <record id="account_group_655" model="account.group">
        <field name="code_prefix">655</field>
        <field name="name">Pérdidas de créditos incobrables derivados de la actividad</field>
        <field name="parent_id" ref="l10n_es.account_group_65"/>
    </record>
    <record id="account_group_656" model="account.group">
        <field name="code_prefix">656</field>
        <field name="name">Resultados de operaciones en común</field>
        <field name="parent_id" ref="l10n_es.account_group_65"/>
    </record>
    <record id="account_group_658" model="account.group">
        <field name="code_prefix">658</field>
        <field name="name">Reintegro de subvenciones, donaciones y legados recibidos, afectos a la actividad propia de la entidad</field>
        <field name="parent_id" ref="l10n_es.account_group_65"/>
    </record>
    <record id="account_group_659" model="account.group">
        <field name="code_prefix">659</field>
        <field name="name">Otras pérdidas en gestión corriente</field>
        <field name="parent_id" ref="l10n_es.account_group_65"/>
    </record>
    <record id="account_group_66" model="account.group">
        <field name="code_prefix">66</field>
        <field name="name">Gastos financieros</field>
        <field name="parent_id" ref="l10n_es.account_group_6"/>
    </record>
    <record id="account_group_660" model="account.group">
        <field name="code_prefix">660</field>
        <field name="name">Gastos financieros por actualización de provisiones</field>
        <field name="parent_id" ref="l10n_es.account_group_66"/>
    </record>
    <record id="account_group_661" model="account.group">
        <field name="code_prefix">661</field>
        <field name="name">Intereses de obligaciones y bonos</field>
        <field name="parent_id" ref="l10n_es.account_group_66"/>
    </record>
    <record id="account_group_6610" model="account.group">
        <field name="code_prefix">6610</field>
        <field name="name">Intereses de obligaciones y bonos a largo plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_661"/>
    </record>
    <record id="account_group_6611" model="account.group">
        <field name="code_prefix">6611</field>
        <field name="name">Intereses de obligaciones y bonos a largo plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_661"/>
    </record>
    <record id="account_group_6612" model="account.group">
        <field name="code_prefix">6612</field>
        <field name="name">Intereses de obligaciones y bonos a largo plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_661"/>
    </record>
    <record id="account_group_6613" model="account.group">
        <field name="code_prefix">6613</field>
        <field name="name">Intereses de obligaciones y bonos a largo plazo, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_661"/>
    </record>
    <record id="account_group_6615" model="account.group">
        <field name="code_prefix">6615</field>
        <field name="name">Intereses de obligaciones y bonos a corto plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_661"/>
    </record>
    <record id="account_group_6616" model="account.group">
        <field name="code_prefix">6616</field>
        <field name="name">Intereses de obligaciones y bonos a corto plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_661"/>
    </record>
    <record id="account_group_6617" model="account.group">
        <field name="code_prefix">6617</field>
        <field name="name">Intereses de obligaciones y bonos a corto plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_661"/>
    </record>
    <record id="account_group_6618" model="account.group">
        <field name="code_prefix">6618</field>
        <field name="name">Intereses de obligaciones y bonos a corto plazo, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_661"/>
    </record>
    <record id="account_group_662" model="account.group">
        <field name="code_prefix">662</field>
        <field name="name">Intereses de deudas</field>
        <field name="parent_id" ref="l10n_es.account_group_66"/>
    </record>
    <record id="account_group_6620" model="account.group">
        <field name="code_prefix">6620</field>
        <field name="name">Intereses de deudas, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_662"/>
    </record>
    <record id="account_group_6621" model="account.group">
        <field name="code_prefix">6621</field>
        <field name="name">Intereses de deudas, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_662"/>
    </record>
    <record id="account_group_6622" model="account.group">
        <field name="code_prefix">6622</field>
        <field name="name">Intereses de deudas, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_662"/>
    </record>
    <record id="account_group_6623" model="account.group">
        <field name="code_prefix">6623</field>
        <field name="name">Intereses de deudas con entidades de crédito</field>
        <field name="parent_id" ref="l10n_es.account_group_662"/>
    </record>
    <record id="account_group_6624" model="account.group">
        <field name="code_prefix">6624</field>
        <field name="name">Intereses de deudas, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_662"/>
    </record>
    <record id="account_group_663" model="account.group">
        <field name="code_prefix">663</field>
        <field name="name">Pérdidas por valoración de instrumentos financieros por su valor razonable</field>
        <field name="parent_id" ref="l10n_es.account_group_66"/>
    </record>
    <record id="account_group_6630" model="account.group">
        <field name="code_prefix">6630</field>
        <field name="name">Pérdidas de cartera de negociación</field>
        <field name="parent_id" ref="l10n_es.account_group_663"/>
    </record>
    <record id="account_group_6631" model="account.group">
        <field name="code_prefix">6631</field>
        <field name="name">Pérdidas de designados por la empresa</field>
        <field name="parent_id" ref="l10n_es.account_group_663"/>
    </record>
    <record id="account_group_6632" model="account.group">
        <field name="code_prefix">6632</field>
        <field name="name">Pérdidas de disponibles para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_663"/>
    </record>
    <record id="account_group_6633" model="account.group">
        <field name="code_prefix">6633</field>
        <field name="name">Pérdidas de instrumentos de cobertura</field>
        <field name="parent_id" ref="l10n_es.account_group_663"/>
    </record>
    <record id="account_group_664" model="account.group">
        <field name="code_prefix">664</field>
        <field name="name">Gastos por dividendos de acciones o participaciones consideradas como pasivos financieros</field>
        <field name="parent_id" ref="l10n_es.account_group_66"/>
    </record>
    <record id="account_group_6640" model="account.group">
        <field name="code_prefix">6640</field>
        <field name="name">Dividendos de pasivos, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_664"/>
    </record>
    <record id="account_group_6641" model="account.group">
        <field name="code_prefix">6641</field>
        <field name="name">Dividendos de pasivos, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_664"/>
    </record>
    <record id="account_group_6642" model="account.group">
        <field name="code_prefix">6642</field>
        <field name="name">Dividendos de pasivos, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_664"/>
    </record>
    <record id="account_group_6643" model="account.group">
        <field name="code_prefix">6643</field>
        <field name="name">Dividendos de pasivos, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_664"/>
    </record>
    <record id="account_group_665" model="account.group">
        <field name="code_prefix">665</field>
        <field name="name">Intereses por descuento de efectos y operaciones de «factoring»</field>
        <field name="parent_id" ref="l10n_es.account_group_66"/>
    </record>
    <record id="account_group_6650" model="account.group">
        <field name="code_prefix">6650</field>
        <field name="name">Intereses por descuento de efectos en entidades de crédito del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_665"/>
    </record>
    <record id="account_group_6651" model="account.group">
        <field name="code_prefix">6651</field>
        <field name="name">Intereses por descuento de efectos en entidades de crédito asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_665"/>
    </record>
    <record id="account_group_6652" model="account.group">
        <field name="code_prefix">6652</field>
        <field name="name">Intereses por descuento de efectos en otras entidades de crédito vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_665"/>
    </record>
    <record id="account_group_6653" model="account.group">
        <field name="code_prefix">6653</field>
        <field name="name">Intereses por descuento de efectos en otras entidades de crédito</field>
        <field name="parent_id" ref="l10n_es.account_group_665"/>
    </record>
    <record id="account_group_6654" model="account.group">
        <field name="code_prefix">6654</field>
        <field name="name">Intereses por operaciones de «factoring» con entidades de crédito del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_665"/>
    </record>
    <record id="account_group_6655" model="account.group">
        <field name="code_prefix">6655</field>
        <field name="name">Intereses por operaciones de "factoring" con entidades de crédito asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_665"/>
    </record>
    <record id="account_group_6656" model="account.group">
        <field name="code_prefix">6656</field>
        <field name="name">Intereses por operaciones de "factoring" con entidades de crédito vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_665"/>
    </record>
    <record id="account_group_6657" model="account.group">
        <field name="code_prefix">6657</field>
        <field name="name">Intereses por operaciones de «factoring» con otras entidades de crédito</field>
        <field name="parent_id" ref="l10n_es.account_group_665"/>
    </record>
    <record id="account_group_666" model="account.group">
        <field name="code_prefix">666</field>
        <field name="name">Pérdidas en participaciones y valores representativos de deuda</field>
        <field name="parent_id" ref="l10n_es.account_group_66"/>
    </record>
    <record id="account_group_6660" model="account.group">
        <field name="code_prefix">6660</field>
        <field name="name">Pérdidas en valores representativos de deuda a largo plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_666"/>
    </record>
    <record id="account_group_6661" model="account.group">
        <field name="code_prefix">6661</field>
        <field name="name">Pérdidas en valores representativos de deuda a largo plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_666"/>
    </record>
    <record id="account_group_6662" model="account.group">
        <field name="code_prefix">6662</field>
        <field name="name">Pérdidas en valores representativos de deuda a largo plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_666"/>
    </record>
    <record id="account_group_6663" model="account.group">
        <field name="code_prefix">6663</field>
        <field name="name">Pérdidas en participaciones y valores representativos de deuda a largo plazo, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_666"/>
    </record>
    <record id="account_group_6665" model="account.group">
        <field name="code_prefix">6665</field>
        <field name="name">Pérdidas en participaciones y valores representativos de deuda a corto plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_666"/>
    </record>
    <record id="account_group_6666" model="account.group">
        <field name="code_prefix">6666</field>
        <field name="name">Pérdidas en participaciones y valores representativos de deuda a corto plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_666"/>
    </record>
    <record id="account_group_6667" model="account.group">
        <field name="code_prefix">6667</field>
        <field name="name">Pérdidas en valores representativos de deuda a corto plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_666"/>
    </record>
    <record id="account_group_6668" model="account.group">
        <field name="code_prefix">6668</field>
        <field name="name">Pérdidas en valores representativos de deuda a corto plazo, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_666"/>
    </record>
    <record id="account_group_667" model="account.group">
        <field name="code_prefix">667</field>
        <field name="name">Pérdidas de créditos no comerciales</field>
        <field name="parent_id" ref="l10n_es.account_group_66"/>
    </record>
    <record id="account_group_6670" model="account.group">
        <field name="code_prefix">6670</field>
        <field name="name">Pérdidas de créditos a largo plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_667"/>
    </record>
    <record id="account_group_6671" model="account.group">
        <field name="code_prefix">6671</field>
        <field name="name">Pérdidas de créditos a largo plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_667"/>
    </record>
    <record id="account_group_6672" model="account.group">
        <field name="code_prefix">6672</field>
        <field name="name">Pérdidas de créditos a largo plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_667"/>
    </record>
    <record id="account_group_6673" model="account.group">
        <field name="code_prefix">6673</field>
        <field name="name">Pérdidas de créditos a largo plazo, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_667"/>
    </record>
    <record id="account_group_6675" model="account.group">
        <field name="code_prefix">6675</field>
        <field name="name">Pérdidas de créditos a corto plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_667"/>
    </record>
    <record id="account_group_6676" model="account.group">
        <field name="code_prefix">6676</field>
        <field name="name">Pérdidas de créditos a corto plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_667"/>
    </record>
    <record id="account_group_6677" model="account.group">
        <field name="code_prefix">6677</field>
        <field name="name">Pérdidas de créditos a corto plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_667"/>
    </record>
    <record id="account_group_6678" model="account.group">
        <field name="code_prefix">6678</field>
        <field name="name">Pérdidas de créditos a corto plazo, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_667"/>
    </record>
    <record id="account_group_668" model="account.group">
        <field name="code_prefix">668</field>
        <field name="name">Diferencias negativas de cambio</field>
        <field name="parent_id" ref="l10n_es.account_group_66"/>
    </record>
    <record id="account_group_669" model="account.group">
        <field name="code_prefix">669</field>
        <field name="name">Otros gastos financieros</field>
        <field name="parent_id" ref="l10n_es.account_group_66"/>
    </record>
    <record id="account_group_67" model="account.group">
        <field name="code_prefix">67</field>
        <field name="name">Pérdidas procedentes de activos no corrientes y gastos excepcionales</field>
        <field name="parent_id" ref="l10n_es.account_group_6"/>
    </record>
    <record id="account_group_670" model="account.group">
        <field name="code_prefix">670</field>
        <field name="name">Pérdidas procedentes del inmovilizado intangible</field>
        <field name="parent_id" ref="l10n_es.account_group_67"/>
    </record>
    <record id="account_group_671" model="account.group">
        <field name="code_prefix">671</field>
        <field name="name">Pérdidas procedentes del inmovilizado material</field>
        <field name="parent_id" ref="l10n_es.account_group_67"/>
    </record>
    <record id="account_group_672" model="account.group">
        <field name="code_prefix">672</field>
        <field name="name">Pérdidas procedentes de las inversiones inmobiliarias</field>
        <field name="parent_id" ref="l10n_es.account_group_67"/>
    </record>
    <record id="account_group_673" model="account.group">
        <field name="code_prefix">673</field>
        <field name="name">Pérdidas procedentes de participaciones a largo plazo en partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_67"/>
    </record>
    <record id="account_group_6733" model="account.group">
        <field name="code_prefix">6733</field>
        <field name="name">Pérdidas procedentes de participaciones a largo plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_673"/>
    </record>
    <record id="account_group_6734" model="account.group">
        <field name="code_prefix">6734</field>
        <field name="name">Pérdidas procedentes de participaciones a largo plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_673"/>
    </record>
    <record id="account_group_6735" model="account.group">
        <field name="code_prefix">6735</field>
        <field name="name">Pérdidas procedentes de participaciones a largo plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_673"/>
    </record>
    <record id="account_group_675" model="account.group">
        <field name="code_prefix">675</field>
        <field name="name">Pérdidas por operaciones con obligaciones propias</field>
        <field name="parent_id" ref="l10n_es.account_group_67"/>
    </record>
    <record id="account_group_678" model="account.group">
        <field name="code_prefix">678</field>
        <field name="name">Gastos excepcionales</field>
        <field name="parent_id" ref="l10n_es.account_group_67"/>
    </record>
    <record id="account_group_68" model="account.group">
        <field name="code_prefix">68</field>
        <field name="name">Dotaciones para amortizaciones</field>
        <field name="parent_id" ref="l10n_es.account_group_6"/>
    </record>
    <record id="account_group_680" model="account.group">
        <field name="code_prefix">680</field>
        <field name="name">Amortización del inmovilizado intangible</field>
        <field name="parent_id" ref="l10n_es.account_group_68"/>
    </record>
    <record id="account_group_681" model="account.group">
        <field name="code_prefix">681</field>
        <field name="name">Amortización del inmovilizado material</field>
        <field name="parent_id" ref="l10n_es.account_group_68"/>
    </record>
    <record id="account_group_682" model="account.group">
        <field name="code_prefix">682</field>
        <field name="name">Amortización de las inversiones inmobiliarias</field>
        <field name="parent_id" ref="l10n_es.account_group_68"/>
    </record>
    <record id="account_group_69" model="account.group">
        <field name="code_prefix">69</field>
        <field name="name">Pérdidas por deterioroy otras dotaciones</field>
        <field name="parent_id" ref="l10n_es.account_group_6"/>
    </record>
    <record id="account_group_690" model="account.group">
        <field name="code_prefix">690</field>
        <field name="name">Pérdidas por deterioro del inmovilizado intangible</field>
        <field name="parent_id" ref="l10n_es.account_group_69"/>
    </record>
    <record id="account_group_691" model="account.group">
        <field name="code_prefix">691</field>
        <field name="name">Pérdidas por deterioro del inmovilizado material</field>
        <field name="parent_id" ref="l10n_es.account_group_69"/>
    </record>
    <record id="account_group_692" model="account.group">
        <field name="code_prefix">692</field>
        <field name="name">Pérdidas por deterioro de las inversiones inmobiliarias</field>
        <field name="parent_id" ref="l10n_es.account_group_69"/>
    </record>
    <record id="account_group_693" model="account.group">
        <field name="code_prefix">693</field>
        <field name="name">Pérdidas por deterioro de existencias</field>
        <field name="parent_id" ref="l10n_es.account_group_69"/>
    </record>
    <record id="account_group_6930" model="account.group">
        <field name="code_prefix">6930</field>
        <field name="name">Pérdidas por deterioro de productos terminados y en curso de fabricación</field>
        <field name="parent_id" ref="l10n_es.account_group_693"/>
    </record>
    <record id="account_group_6931" model="account.group">
        <field name="code_prefix">6931</field>
        <field name="name">Pérdidas por deterioro de mercaderías</field>
        <field name="parent_id" ref="l10n_es.account_group_693"/>
    </record>
    <record id="account_group_6932" model="account.group">
        <field name="code_prefix">6932</field>
        <field name="name">Pérdidas por deterioro de materias primas</field>
        <field name="parent_id" ref="l10n_es.account_group_693"/>
    </record>
    <record id="account_group_6933" model="account.group">
        <field name="code_prefix">6933</field>
        <field name="name">Pérdidas por deterioro de otros aprovisionamientos</field>
        <field name="parent_id" ref="l10n_es.account_group_693"/>
    </record>
    <record id="account_group_694" model="account.group">
        <field name="code_prefix">694</field>
        <field name="name">Pérdidas por deterioro de créditos por operaciones comerciales</field>
        <field name="parent_id" ref="l10n_es.account_group_69"/>
    </record>
    <record id="account_group_695" model="account.group">
        <field name="code_prefix">695</field>
        <field name="name">Dotación a la provisión por operaciones comerciales</field>
        <field name="parent_id" ref="l10n_es.account_group_69"/>
    </record>
    <record id="account_group_6954" model="account.group">
        <field name="code_prefix">6954</field>
        <field name="name">Dotación a la provisión por contratos onerosos</field>
        <field name="parent_id" ref="l10n_es.account_group_695"/>
    </record>
    <record id="account_group_6959" model="account.group">
        <field name="code_prefix">6959</field>
        <field name="name">Dotación a la provisión para otras operaciones comerciales</field>
        <field name="parent_id" ref="l10n_es.account_group_695"/>
    </record>
    <record id="account_group_696" model="account.group">
        <field name="code_prefix">696</field>
        <field name="name">Pérdidas por deterioro de participaciones y valores representativos de deuda a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_69"/>
    </record>
    <record id="account_group_6960" model="account.group">
        <field name="code_prefix">6960</field>
        <field name="name">Pérdidas por deterioro de participaciones en instrumentos de patrimonio neto a largo plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_696"/>
    </record>
    <record id="account_group_6961" model="account.group">
        <field name="code_prefix">6961</field>
        <field name="name">Pérdidas por deterioro de participaciones en instrumentos de patrimonio neto a largo plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_696"/>
    </record>
    <record id="account_group_6962" model="account.group">
        <field name="code_prefix">6962</field>
        <field name="name">Pérdidas por deterioro de participaciones en instrumentos de patrimonio neto a largo plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_696"/>
    </record>
    <record id="account_group_6963" model="account.group">
        <field name="code_prefix">6963</field>
        <field name="name">Pérdidas por deterioro de participaciones en instrumentos de patrimonio neto a largo plazo, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_696"/>
    </record>
    <record id="account_group_6965" model="account.group">
        <field name="code_prefix">6965</field>
        <field name="name">Pérdidas por deterioro en valores representativos de deuda a largo plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_696"/>
    </record>
    <record id="account_group_6966" model="account.group">
        <field name="code_prefix">6966</field>
        <field name="name">Pérdidas por deterioro en valores representativos de deuda a largo plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_696"/>
    </record>
    <record id="account_group_6967" model="account.group">
        <field name="code_prefix">6967</field>
        <field name="name">Pérdidas por deterioro en valores representativos de deuda a largo plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_696"/>
    </record>
    <record id="account_group_6968" model="account.group">
        <field name="code_prefix">6968</field>
        <field name="name">Pérdidas por deterioro en valores representativos de deuda a largo plazo, de otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_696"/>
    </record>
    <record id="account_group_697" model="account.group">
        <field name="code_prefix">697</field>
        <field name="name">Pérdidas por deterioro de créditos a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_69"/>
    </record>
    <record id="account_group_6970" model="account.group">
        <field name="code_prefix">6970</field>
        <field name="name">Pérdidas por deterioro de créditos a largo plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_697"/>
    </record>
    <record id="account_group_6971" model="account.group">
        <field name="code_prefix">6971</field>
        <field name="name">Pérdidas por deterioro de créditos a largo plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_697"/>
    </record>
    <record id="account_group_6972" model="account.group">
        <field name="code_prefix">6972</field>
        <field name="name">Pérdidas por deterioro de créditos a largo plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_697"/>
    </record>
    <record id="account_group_6973" model="account.group">
        <field name="code_prefix">6973</field>
        <field name="name">Pérdidas por deterioro de créditos a largo plazo, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_697"/>
    </record>
    <record id="account_group_698" model="account.group">
        <field name="code_prefix">698</field>
        <field name="name">Pérdidas por deterioro de participaciones y valores representativos de deuda a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_69"/>
    </record>
    <record id="account_group_6980" model="account.group">
        <field name="code_prefix">6980</field>
        <field name="name">Pérdidas por deterioro de participaciones en instrumentos de patrimonio neto a corto plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_698"/>
    </record>
    <record id="account_group_6981" model="account.group">
        <field name="code_prefix">6981</field>
        <field name="name">Pérdidas por deterioro de participaciones en instrumentos de patrimonio neto a corto plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_698"/>
    </record>
    <record id="account_group_6985" model="account.group">
        <field name="code_prefix">6985</field>
        <field name="name">Pérdidas por deterioro en valores representativos de deuda a corto plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_698"/>
    </record>
    <record id="account_group_6986" model="account.group">
        <field name="code_prefix">6986</field>
        <field name="name">Pérdidas por deterioro en valores representativos de deuda a corto plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_698"/>
    </record>
    <record id="account_group_6987" model="account.group">
        <field name="code_prefix">6987</field>
        <field name="name">Pérdidas por deterioro en valores representativos de deuda a corto plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_698"/>
    </record>
    <record id="account_group_6988" model="account.group">
        <field name="code_prefix">6988</field>
        <field name="name">Pérdidas por deterioro en valores representativos de deuda a corto plazo, de otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_698"/>
    </record>
    <record id="account_group_699" model="account.group">
        <field name="code_prefix">699</field>
        <field name="name">Pérdidas por deterioro de créditos a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_69"/>
    </record>
    <record id="account_group_6990" model="account.group">
        <field name="code_prefix">6990</field>
        <field name="name">Pérdidas por deterioro de créditos a corto plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_699"/>
    </record>
    <record id="account_group_6991" model="account.group">
        <field name="code_prefix">6991</field>
        <field name="name">Pérdidas por deterioro de créditos a corto plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_699"/>
    </record>
    <record id="account_group_6992" model="account.group">
        <field name="code_prefix">6992</field>
        <field name="name">Pérdidas por deterioro de créditos a corto plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_699"/>
    </record>
    <record id="account_group_6993" model="account.group">
        <field name="code_prefix">6993</field>
        <field name="name">Pérdidas por deterioro de créditos a corto plazo, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_699"/>
    </record>
        <record id="account_group_7" model="account.group">
        <field name="code_prefix">7</field>
        <field name="name">Ventas e ingresos</field>
    </record>
    <record id="account_group_70" model="account.group">
        <field name="code_prefix">70</field>
        <field name="name">Ventas de mercaderías, de producción propia, de servicios, etc</field>
        <field name="parent_id" ref="l10n_es.account_group_7"/>
    </record>
    <record id="account_group_700" model="account.group">
        <field name="code_prefix">700</field>
        <field name="name">Ventas de mercaderías</field>
        <field name="parent_id" ref="l10n_es.account_group_70"/>
    </record>
    <record id="account_group_701" model="account.group">
        <field name="code_prefix">701</field>
        <field name="name">Ventas de productos terminados</field>
        <field name="parent_id" ref="l10n_es.account_group_70"/>
    </record>
    <record id="account_group_702" model="account.group">
        <field name="code_prefix">702</field>
        <field name="name">Ventas de productos semiterminados</field>
        <field name="parent_id" ref="l10n_es.account_group_70"/>
    </record>
    <record id="account_group_703" model="account.group">
        <field name="code_prefix">703</field>
        <field name="name">Ventas de subproductos y residuos</field>
        <field name="parent_id" ref="l10n_es.account_group_70"/>
    </record>
    <record id="account_group_704" model="account.group">
        <field name="code_prefix">704</field>
        <field name="name">Ventas de envases y embalajes</field>
        <field name="parent_id" ref="l10n_es.account_group_70"/>
    </record>
    <record id="account_group_705" model="account.group">
        <field name="code_prefix">705</field>
        <field name="name">Prestaciones de servicios</field>
        <field name="parent_id" ref="l10n_es.account_group_70"/>
    </record>
    <record id="account_group_706" model="account.group">
        <field name="code_prefix">706</field>
        <field name="name">Descuentos sobre ventas por pronto pago</field>
        <field name="parent_id" ref="l10n_es.account_group_70"/>
    </record>
    <record id="account_group_7060" model="account.group">
        <field name="code_prefix">7060</field>
        <field name="name">Descuentos sobre ventas por pronto pago de mercaderías</field>
        <field name="parent_id" ref="l10n_es.account_group_706"/>
    </record>
    <record id="account_group_7061" model="account.group">
        <field name="code_prefix">7061</field>
        <field name="name">Descuentos sobre ventas por pronto pago de productos terminados</field>
        <field name="parent_id" ref="l10n_es.account_group_706"/>
    </record>
    <record id="account_group_7062" model="account.group">
        <field name="code_prefix">7062</field>
        <field name="name">Descuentos sobre ventas por pronto pago de productos semiterminados</field>
        <field name="parent_id" ref="l10n_es.account_group_706"/>
    </record>
    <record id="account_group_7063" model="account.group">
        <field name="code_prefix">7063</field>
        <field name="name">Descuentos sobre ventas por pronto pago de subproductos y residuos</field>
        <field name="parent_id" ref="l10n_es.account_group_706"/>
    </record>
    <record id="account_group_708" model="account.group">
        <field name="code_prefix">708</field>
        <field name="name">Devoluciones de ventas y operaciones similares</field>
        <field name="parent_id" ref="l10n_es.account_group_70"/>
    </record>
    <record id="account_group_7080" model="account.group">
        <field name="code_prefix">7080</field>
        <field name="name">Devoluciones de ventas de mercaderías</field>
        <field name="parent_id" ref="l10n_es.account_group_708"/>
    </record>
    <record id="account_group_7081" model="account.group">
        <field name="code_prefix">7081</field>
        <field name="name">Devoluciones de ventas de productos terminados</field>
        <field name="parent_id" ref="l10n_es.account_group_708"/>
    </record>
    <record id="account_group_7082" model="account.group">
        <field name="code_prefix">7082</field>
        <field name="name">Devoluciones de ventas de productos semiterminados</field>
        <field name="parent_id" ref="l10n_es.account_group_708"/>
    </record>
    <record id="account_group_7083" model="account.group">
        <field name="code_prefix">7083</field>
        <field name="name">Devoluciones de ventas de subproductos y residuos</field>
        <field name="parent_id" ref="l10n_es.account_group_708"/>
    </record>
    <record id="account_group_7084" model="account.group">
        <field name="code_prefix">7084</field>
        <field name="name">Devoluciones de ventas de envases y embalajes</field>
        <field name="parent_id" ref="l10n_es.account_group_708"/>
    </record>
    <record id="account_group_709" model="account.group">
        <field name="code_prefix">709</field>
        <field name="name">«Rappels» sobre ventas</field>
        <field name="parent_id" ref="l10n_es.account_group_70"/>
    </record>
    <record id="account_group_7090" model="account.group">
        <field name="code_prefix">7090</field>
        <field name="name">«Rappels» sobre ventas de mercaderías</field>
        <field name="parent_id" ref="l10n_es.account_group_709"/>
    </record>
    <record id="account_group_7091" model="account.group">
        <field name="code_prefix">7091</field>
        <field name="name">«Rappels» sobre ventas de productos terminados</field>
        <field name="parent_id" ref="l10n_es.account_group_709"/>
    </record>
    <record id="account_group_7092" model="account.group">
        <field name="code_prefix">7092</field>
        <field name="name">«Rappels» sobre ventas de productos semiterminados</field>
        <field name="parent_id" ref="l10n_es.account_group_709"/>
    </record>
    <record id="account_group_7093" model="account.group">
        <field name="code_prefix">7093</field>
        <field name="name">«Rappels» sobre ventas de subproductos y residuos</field>
        <field name="parent_id" ref="l10n_es.account_group_709"/>
    </record>
    <record id="account_group_7094" model="account.group">
        <field name="code_prefix">7094</field>
        <field name="name">«Rappels» sobre ventas de envases y embalajes</field>
        <field name="parent_id" ref="l10n_es.account_group_709"/>
    </record>
    <record id="account_group_71" model="account.group">
        <field name="code_prefix">71</field>
        <field name="name">Variación de existencias</field>
        <field name="parent_id" ref="l10n_es.account_group_7"/>
    </record>
    <record id="account_group_710" model="account.group">
        <field name="code_prefix">710</field>
        <field name="name">Variación de existencias de productos en curso</field>
        <field name="parent_id" ref="l10n_es.account_group_71"/>
    </record>
    <record id="account_group_711" model="account.group">
        <field name="code_prefix">711</field>
        <field name="name">Variación de existencias de productos semiterminados</field>
        <field name="parent_id" ref="l10n_es.account_group_71"/>
    </record>
    <record id="account_group_712" model="account.group">
        <field name="code_prefix">712</field>
        <field name="name">Variación de existencias de productos terminados</field>
        <field name="parent_id" ref="l10n_es.account_group_71"/>
    </record>
    <record id="account_group_713" model="account.group">
        <field name="code_prefix">713</field>
        <field name="name">Variación de existencias de subproductos, residuos y materiales recuperados</field>
        <field name="parent_id" ref="l10n_es.account_group_71"/>
    </record>
    <record id="account_group_72" model="account.group">
        <field name="code_prefix">72</field>
        <field name="name">Ingresos propios de la entidad</field>
        <field name="parent_id" ref="l10n_es.account_group_7"/>
    </record>
    <record id="account_group_720" model="account.group">
        <field name="code_prefix">720</field>
        <field name="name">Cuotas de asociados y afiliados</field>
        <field name="parent_id" ref="l10n_es.account_group_72"/>
    </record>
    <record id="account_group_721" model="account.group">
        <field name="code_prefix">721</field>
        <field name="name">Cuotas de usuarios</field>
        <field name="parent_id" ref="l10n_es.account_group_72"/>
    </record>
    <record id="account_group_722" model="account.group">
        <field name="code_prefix">722</field>
        <field name="name">Promociones para captación de recursos</field>
        <field name="parent_id" ref="l10n_es.account_group_72"/>
    </record>
    <record id="account_group_723" model="account.group">
        <field name="code_prefix">723</field>
        <field name="name">Ingresos de patrocinadores y colaboraciones</field>
        <field name="parent_id" ref="l10n_es.account_group_72"/>
    </record>
    <record id="account_group_728" model="account.group">
        <field name="code_prefix">728</field>
        <field name="name">Ingresos por reintegro de ayudas y asignaciones</field>
        <field name="parent_id" ref="l10n_es.account_group_72"/>
    </record>
    <record id="account_group_73" model="account.group">
        <field name="code_prefix">73</field>
        <field name="name">Trabajos realizados para la empresa</field>
        <field name="parent_id" ref="l10n_es.account_group_7"/>
    </record>
    <record id="account_group_730" model="account.group">
        <field name="code_prefix">730</field>
        <field name="name">Trabajos realizados para el inmovilizado intangible</field>
        <field name="parent_id" ref="l10n_es.account_group_73"/>
    </record>
    <record id="account_group_731" model="account.group">
        <field name="code_prefix">731</field>
        <field name="name">Trabajos realizados para el inmovilizado material</field>
        <field name="parent_id" ref="l10n_es.account_group_73"/>
    </record>
    <record id="account_group_732" model="account.group">
        <field name="code_prefix">732</field>
        <field name="name">Trabajos realizados en inversiones inmobiliarias</field>
        <field name="parent_id" ref="l10n_es.account_group_73"/>
    </record>
    <record id="account_group_733" model="account.group">
        <field name="code_prefix">733</field>
        <field name="name">Trabajos realizados para el inmovilizado material en curso</field>
        <field name="parent_id" ref="l10n_es.account_group_73"/>
    </record>
    <record id="account_group_74" model="account.group">
        <field name="code_prefix">74</field>
        <field name="name">Subvenciones, donaciones y legados</field>
        <field name="parent_id" ref="l10n_es.account_group_7"/>
    </record>
    <record id="account_group_740" model="account.group">
        <field name="code_prefix">740</field>
        <field name="name">Subvenciones, donaciones y legados a la explotación</field>
        <field name="parent_id" ref="l10n_es.account_group_74"/>
    </record>
    <record id="account_group_746" model="account.group">
        <field name="code_prefix">746</field>
        <field name="name">Subvenciones, donaciones y legados de capital transferidos al resultado del ejercicio</field>
        <field name="parent_id" ref="l10n_es.account_group_74"/>
    </record>
    <record id="account_group_747" model="account.group">
        <field name="code_prefix">747</field>
        <field name="name">Otras subvenciones, donaciones y legados transferidos al resultado del ejercicio</field>
        <field name="parent_id" ref="l10n_es.account_group_74"/>
    </record>
    <record id="account_group_75" model="account.group">
        <field name="code_prefix">75</field>
        <field name="name">Otros ingresos de gestión</field>
        <field name="parent_id" ref="l10n_es.account_group_7"/>
    </record>
    <record id="account_group_751" model="account.group">
        <field name="code_prefix">751</field>
        <field name="name">Resultados de operaciones en común</field>
        <field name="parent_id" ref="l10n_es.account_group_75"/>
    </record>
    <record id="account_group_7510" model="account.group">
        <field name="code_prefix">7510</field>
        <field name="name">Pérdida transferida (gestor)</field>
        <field name="parent_id" ref="l10n_es.account_group_751"/>
    </record>
    <record id="account_group_7511" model="account.group">
        <field name="code_prefix">7511</field>
        <field name="name">Beneficio atribuido (partícipe o asociado no gestor)</field>
        <field name="parent_id" ref="l10n_es.account_group_751"/>
    </record>
    <record id="account_group_752" model="account.group">
        <field name="code_prefix">752</field>
        <field name="name">Ingresos por arrendamientos</field>
        <field name="parent_id" ref="l10n_es.account_group_75"/>
    </record>
    <record id="account_group_753" model="account.group">
        <field name="code_prefix">753</field>
        <field name="name">Ingresos de propiedad industrial cedida en explotación</field>
        <field name="parent_id" ref="l10n_es.account_group_75"/>
    </record>
    <record id="account_group_754" model="account.group">
        <field name="code_prefix">754</field>
        <field name="name">Ingresos por comisiones</field>
        <field name="parent_id" ref="l10n_es.account_group_75"/>
    </record>
    <record id="account_group_755" model="account.group">
        <field name="code_prefix">755</field>
        <field name="name">Ingresos por servicios al personal</field>
        <field name="parent_id" ref="l10n_es.account_group_75"/>
    </record>
    <record id="account_group_759" model="account.group">
        <field name="code_prefix">759</field>
        <field name="name">Ingresos por servicios diversos</field>
        <field name="parent_id" ref="l10n_es.account_group_75"/>
    </record>
    <record id="account_group_76" model="account.group">
        <field name="code_prefix">76</field>
        <field name="name">Ingresos financieros</field>
        <field name="parent_id" ref="l10n_es.account_group_7"/>
    </record>
    <record id="account_group_760" model="account.group">
        <field name="code_prefix">760</field>
        <field name="name">Ingresos de participaciones en instrumentos de patrimonio</field>
        <field name="parent_id" ref="l10n_es.account_group_76"/>
    </record>
    <record id="account_group_7600" model="account.group">
        <field name="code_prefix">7600</field>
        <field name="name">Ingresos de participaciones en instrumentos de patrimonio, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_760"/>
    </record>
    <record id="account_group_7601" model="account.group">
        <field name="code_prefix">7601</field>
        <field name="name">Ingresos de participaciones en instrumentos de patrimonio, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_760"/>
    </record>
    <record id="account_group_7602" model="account.group">
        <field name="code_prefix">7602</field>
        <field name="name">Ingresos de participaciones en instrumentos de patrimonio, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_760"/>
    </record>
    <record id="account_group_7603" model="account.group">
        <field name="code_prefix">7603</field>
        <field name="name">Ingresos de participaciones en instrumentos de patrimonio, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_760"/>
    </record>
    <record id="account_group_761" model="account.group">
        <field name="code_prefix">761</field>
        <field name="name">Ingresos de valores representativos de deuda</field>
        <field name="parent_id" ref="l10n_es.account_group_76"/>
    </record>
    <record id="account_group_7610" model="account.group">
        <field name="code_prefix">7610</field>
        <field name="name">Ingresos de valores representativos de deuda, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_761"/>
    </record>
    <record id="account_group_7611" model="account.group">
        <field name="code_prefix">7611</field>
        <field name="name">Ingresos de valores representativos de deuda, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_761"/>
    </record>
    <record id="account_group_7612" model="account.group">
        <field name="code_prefix">7612</field>
        <field name="name">Ingresos de valores representativos de deuda, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_761"/>
    </record>
    <record id="account_group_7613" model="account.group">
        <field name="code_prefix">7613</field>
        <field name="name">Ingresos de valores representativos de deuda, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_761"/>
    </record>
    <record id="account_group_762" model="account.group">
        <field name="code_prefix">762</field>
        <field name="name">Ingresos de créditos</field>
        <field name="parent_id" ref="l10n_es.account_group_76"/>
    </record>
    <record id="account_group_7620" model="account.group">
        <field name="code_prefix">7620</field>
        <field name="name">Ingresos de créditos a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_762"/>
    </record>
    <record id="account_group_76200" model="account.group">
        <field name="code_prefix">76200</field>
        <field name="name">Ingresos de créditos a largo plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_7620"/>
    </record>
    <record id="account_group_76201" model="account.group">
        <field name="code_prefix">76201</field>
        <field name="name">Ingresos de créditos a largo plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_7620"/>
    </record>
    <record id="account_group_76202" model="account.group">
        <field name="code_prefix">76202</field>
        <field name="name">Ingresos de créditos a largo plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_7620"/>
    </record>
    <record id="account_group_76203" model="account.group">
        <field name="code_prefix">76203</field>
        <field name="name">Ingresos de créditos a largo plazo, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_7620"/>
    </record>
    <record id="account_group_7621" model="account.group">
        <field name="code_prefix">7621</field>
        <field name="name">Ingresos de créditos a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_762"/>
    </record>
    <record id="account_group_76210" model="account.group">
        <field name="code_prefix">76210</field>
        <field name="name">Ingresos de créditos a corto plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_7621"/>
    </record>
    <record id="account_group_76211" model="account.group">
        <field name="code_prefix">76211</field>
        <field name="name">Ingresos de créditos a corto plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_7621"/>
    </record>
    <record id="account_group_76212" model="account.group">
        <field name="code_prefix">76212</field>
        <field name="name">Ingresos de créditos a corto plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_7621"/>
    </record>
    <record id="account_group_76213" model="account.group">
        <field name="code_prefix">76213</field>
        <field name="name">Ingresos de créditos a corto plazo, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_7621"/>
    </record>
    <record id="account_group_763" model="account.group">
        <field name="code_prefix">763</field>
        <field name="name">Beneficios por valoración de instrumentos financieros por su valor razonable</field>
        <field name="parent_id" ref="l10n_es.account_group_76"/>
    </record>
    <record id="account_group_7630" model="account.group">
        <field name="code_prefix">7630</field>
        <field name="name">Beneficios de cartera de negociación</field>
        <field name="parent_id" ref="l10n_es.account_group_763"/>
    </record>
    <record id="account_group_7631" model="account.group">
        <field name="code_prefix">7631</field>
        <field name="name">Beneficios de designados por la empresa</field>
        <field name="parent_id" ref="l10n_es.account_group_763"/>
    </record>
    <record id="account_group_7632" model="account.group">
        <field name="code_prefix">7632</field>
        <field name="name">Beneficios de disponibles para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_763"/>
    </record>
    <record id="account_group_7633" model="account.group">
        <field name="code_prefix">7633</field>
        <field name="name">Beneficios de instrumentos de cobertura</field>
        <field name="parent_id" ref="l10n_es.account_group_763"/>
    </record>
    <record id="account_group_766" model="account.group">
        <field name="code_prefix">766</field>
        <field name="name">Beneficios en participaciones y valores representativos de deuda</field>
        <field name="parent_id" ref="l10n_es.account_group_76"/>
    </record>
    <record id="account_group_7660" model="account.group">
        <field name="code_prefix">7660</field>
        <field name="name">Beneficios en valores representativos de deuda a largo plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_766"/>
    </record>
    <record id="account_group_7661" model="account.group">
        <field name="code_prefix">7661</field>
        <field name="name">Beneficios en valores representativos de deuda a largo plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_766"/>
    </record>
    <record id="account_group_7662" model="account.group">
        <field name="code_prefix">7662</field>
        <field name="name">Beneficios en valores representativos de deuda a largo plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_766"/>
    </record>
    <record id="account_group_7663" model="account.group">
        <field name="code_prefix">7663</field>
        <field name="name">Beneficios en participaciones y valores representativos de deuda a largo plazo, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_766"/>
    </record>
    <record id="account_group_7665" model="account.group">
        <field name="code_prefix">7665</field>
        <field name="name">Beneficios en participaciones y valores representativos de deuda a corto plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_766"/>
    </record>
    <record id="account_group_7666" model="account.group">
        <field name="code_prefix">7666</field>
        <field name="name">Beneficios en participaciones y valores representativos de deuda a corto plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_766"/>
    </record>
    <record id="account_group_7667" model="account.group">
        <field name="code_prefix">7667</field>
        <field name="name">Beneficios en valores representativos de deuda a corto plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_766"/>
    </record>
    <record id="account_group_7668" model="account.group">
        <field name="code_prefix">7668</field>
        <field name="name">Beneficios en valores representativos de deuda a corto plazo, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_766"/>
    </record>
    <record id="account_group_767" model="account.group">
        <field name="code_prefix">767</field>
        <field name="name">Ingresos de activos afectos y de derechos de reembolso relativos a retribuciones a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_76"/>
    </record>
    <record id="account_group_768" model="account.group">
        <field name="code_prefix">768</field>
        <field name="name">Diferencias positivas de cambio</field>
        <field name="parent_id" ref="l10n_es.account_group_76"/>
    </record>
    <record id="account_group_769" model="account.group">
        <field name="code_prefix">769</field>
        <field name="name">Otros ingresos financieros</field>
        <field name="parent_id" ref="l10n_es.account_group_76"/>
    </record>
    <record id="account_group_77" model="account.group">
        <field name="code_prefix">77</field>
        <field name="name">Beneficios procedentes de activos no corrientes e ingresos excepcionales</field>
        <field name="parent_id" ref="l10n_es.account_group_7"/>
    </record>
    <record id="account_group_770" model="account.group">
        <field name="code_prefix">770</field>
        <field name="name">Beneficios procedentes del inmovilizado intangible</field>
        <field name="parent_id" ref="l10n_es.account_group_77"/>
    </record>
    <record id="account_group_771" model="account.group">
        <field name="code_prefix">771</field>
        <field name="name">Beneficios procedentes del inmovilizado material</field>
        <field name="parent_id" ref="l10n_es.account_group_77"/>
    </record>
    <record id="account_group_772" model="account.group">
        <field name="code_prefix">772</field>
        <field name="name">Beneficios procedentes de las inversiones inmobiliarias</field>
        <field name="parent_id" ref="l10n_es.account_group_77"/>
    </record>
    <record id="account_group_773" model="account.group">
        <field name="code_prefix">773</field>
        <field name="name">Beneficios procedentes de participaciones a largo plazo en partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_77"/>
    </record>
    <record id="account_group_7733" model="account.group">
        <field name="code_prefix">7733</field>
        <field name="name">Beneficios procedentes de participaciones a largo plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_773"/>
    </record>
    <record id="account_group_7734" model="account.group">
        <field name="code_prefix">7734</field>
        <field name="name">Beneficios procedentes de participaciones a largo plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_773"/>
    </record>
    <record id="account_group_7735" model="account.group">
        <field name="code_prefix">7735</field>
        <field name="name">Beneficios procedentes de participaciones a largo plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_773"/>
    </record>
    <record id="account_group_774" model="account.group">
        <field name="code_prefix">774</field>
        <field name="name">Diferencia negativa en combinaciones de negocios</field>
        <field name="parent_id" ref="l10n_es.account_group_77"/>
    </record>
    <record id="account_group_775" model="account.group">
        <field name="code_prefix">775</field>
        <field name="name">Beneficios por operaciones con obligaciones propias</field>
        <field name="parent_id" ref="l10n_es.account_group_77"/>
    </record>
    <record id="account_group_778" model="account.group">
        <field name="code_prefix">778</field>
        <field name="name">Ingresos excepcionales</field>
        <field name="parent_id" ref="l10n_es.account_group_77"/>
    </record>
    <record id="account_group_79" model="account.group">
        <field name="code_prefix">79</field>
        <field name="name">Excesos y aplicaciones de provisiones y de pérdidas por deterioro</field>
        <field name="parent_id" ref="l10n_es.account_group_7"/>
    </record>
    <record id="account_group_790" model="account.group">
        <field name="code_prefix">790</field>
        <field name="name">Reversión del deterioro del inmovilizado intangible</field>
        <field name="parent_id" ref="l10n_es.account_group_79"/>
    </record>
    <record id="account_group_791" model="account.group">
        <field name="code_prefix">791</field>
        <field name="name">Reversión del deterioro del inmovilizado material</field>
        <field name="parent_id" ref="l10n_es.account_group_79"/>
    </record>
    <record id="account_group_792" model="account.group">
        <field name="code_prefix">792</field>
        <field name="name">Reversión del deterioro de las inversiones inmobiliarias</field>
        <field name="parent_id" ref="l10n_es.account_group_79"/>
    </record>
    <record id="account_group_793" model="account.group">
        <field name="code_prefix">793</field>
        <field name="name">Reversión del deterioro de existencias</field>
        <field name="parent_id" ref="l10n_es.account_group_79"/>
    </record>
    <record id="account_group_7930" model="account.group">
        <field name="code_prefix">7930</field>
        <field name="name">Reversión del deterioro de productos terminados y en curso de fabricación</field>
        <field name="parent_id" ref="l10n_es.account_group_793"/>
    </record>
    <record id="account_group_7931" model="account.group">
        <field name="code_prefix">7931</field>
        <field name="name">Reversión del deterioro de mercaderías</field>
        <field name="parent_id" ref="l10n_es.account_group_793"/>
    </record>
    <record id="account_group_7932" model="account.group">
        <field name="code_prefix">7932</field>
        <field name="name">Reversión del deterioro de materias primas</field>
        <field name="parent_id" ref="l10n_es.account_group_793"/>
    </record>
    <record id="account_group_7933" model="account.group">
        <field name="code_prefix">7933</field>
        <field name="name">Reversión del deterioro de otros aprovisionamientos</field>
        <field name="parent_id" ref="l10n_es.account_group_793"/>
    </record>
    <record id="account_group_794" model="account.group">
        <field name="code_prefix">794</field>
        <field name="name">Reversión del deterioro de créditos por operaciones comerciales</field>
        <field name="parent_id" ref="l10n_es.account_group_79"/>
    </record>
    <record id="account_group_795" model="account.group">
        <field name="code_prefix">795</field>
        <field name="name">Exceso de provisiones</field>
        <field name="parent_id" ref="l10n_es.account_group_79"/>
    </record>
    <record id="account_group_7950" model="account.group">
        <field name="code_prefix">7950</field>
        <field name="name">Exceso de provisión por retribuciones al personal</field>
        <field name="parent_id" ref="l10n_es.account_group_795"/>
    </record>
    <record id="account_group_7951" model="account.group">
        <field name="code_prefix">7951</field>
        <field name="name">Exceso de provisión para impuestos</field>
        <field name="parent_id" ref="l10n_es.account_group_795"/>
    </record>
    <record id="account_group_7952" model="account.group">
        <field name="code_prefix">7952</field>
        <field name="name">Exceso de provisión para otras responsabilidades</field>
        <field name="parent_id" ref="l10n_es.account_group_795"/>
    </record>
    <record id="account_group_7954" model="account.group">
        <field name="code_prefix">7954</field>
        <field name="name">Exceso de provisión por operaciones comerciales</field>
        <field name="parent_id" ref="l10n_es.account_group_795"/>
    </record>
    <record id="account_group_79544" model="account.group">
        <field name="code_prefix">79544</field>
        <field name="name">Exceso de provisión por contratos onerosos</field>
        <field name="parent_id" ref="l10n_es.account_group_7954"/>
    </record>
    <record id="account_group_79549" model="account.group">
        <field name="code_prefix">79549</field>
        <field name="name">Exceso de provisión para otras operaciones comerciales</field>
        <field name="parent_id" ref="l10n_es.account_group_7954"/>
    </record>
    <record id="account_group_7955" model="account.group">
        <field name="code_prefix">7955</field>
        <field name="name">Exceso de provisión para actuaciones medioambientales</field>
        <field name="parent_id" ref="l10n_es.account_group_795"/>
    </record>
    <record id="account_group_7956" model="account.group">
        <field name="code_prefix">7956</field>
        <field name="name">Exceso de provisión para reestructuraciones</field>
        <field name="parent_id" ref="l10n_es.account_group_795"/>
    </record>
    <record id="account_group_7957" model="account.group">
        <field name="code_prefix">7957</field>
        <field name="name">Exceso de provisión por transacciones con pagos basados en instrumentos de patrimonio</field>
        <field name="parent_id" ref="l10n_es.account_group_795"/>
    </record>
    <record id="account_group_796" model="account.group">
        <field name="code_prefix">796</field>
        <field name="name">Reversión del deterioro de participaciones y valores representativos de deuda a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_79"/>
    </record>
    <record id="account_group_7960" model="account.group">
        <field name="code_prefix">7960</field>
        <field name="name">Reversión del deterioro de participaciones en instrumentos de patrimonio neto a largo plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_796"/>
    </record>
    <record id="account_group_7961" model="account.group">
        <field name="code_prefix">7961</field>
        <field name="name">Reversión del deterioro de participaciones en instrumentos de patrimonio neto a largo plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_796"/>
    </record>
    <record id="account_group_7965" model="account.group">
        <field name="code_prefix">7965</field>
        <field name="name">Reversión del deterioro de valores representativos de deuda a largo plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_796"/>
    </record>
    <record id="account_group_7966" model="account.group">
        <field name="code_prefix">7966</field>
        <field name="name">Reversión del deterioro de valores representativos de deuda a largo plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_796"/>
    </record>
    <record id="account_group_7967" model="account.group">
        <field name="code_prefix">7967</field>
        <field name="name">Reversión del deterioro de valores representativos de deuda a largo plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_796"/>
    </record>
    <record id="account_group_7968" model="account.group">
        <field name="code_prefix">7968</field>
        <field name="name">Reversión del deterioro de valores representativos de deuda a largo plazo, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_796"/>
    </record>
    <record id="account_group_797" model="account.group">
        <field name="code_prefix">797</field>
        <field name="name">Reversión del deterioro de créditos a largo plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_79"/>
    </record>
    <record id="account_group_7970" model="account.group">
        <field name="code_prefix">7970</field>
        <field name="name">Reversión del deterioro de créditos a largo plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_797"/>
    </record>
    <record id="account_group_7971" model="account.group">
        <field name="code_prefix">7971</field>
        <field name="name">Reversión del deterioro de créditos a largo plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_797"/>
    </record>
    <record id="account_group_7972" model="account.group">
        <field name="code_prefix">7972</field>
        <field name="name">Reversión del deterioro de créditos a largo plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_797"/>
    </record>
    <record id="account_group_7973" model="account.group">
        <field name="code_prefix">7973</field>
        <field name="name">Reversión del deterioro de créditos a largo plazo, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_797"/>
    </record>
    <record id="account_group_798" model="account.group">
        <field name="code_prefix">798</field>
        <field name="name">Reversión del deterioro de participaciones y valores representativos de deuda a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_79"/>
    </record>
    <record id="account_group_7980" model="account.group">
        <field name="code_prefix">7980</field>
        <field name="name">Reversión del deterioro de participaciones en instrumentos de patrimonio neto a corto plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_798"/>
    </record>
    <record id="account_group_7981" model="account.group">
        <field name="code_prefix">7981</field>
        <field name="name">Reversión del deterioro de participaciones en instrumentos de patrimonio neto a corto plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_798"/>
    </record>
    <record id="account_group_7985" model="account.group">
        <field name="code_prefix">7985</field>
        <field name="name">Reversión del deterioro en valores representativos de deuda a corto plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_798"/>
    </record>
    <record id="account_group_7986" model="account.group">
        <field name="code_prefix">7986</field>
        <field name="name">Reversión del deterioro en valores representativos de deuda a corto plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_798"/>
    </record>
    <record id="account_group_7987" model="account.group">
        <field name="code_prefix">7987</field>
        <field name="name">Reversión del deterioro en valores representativos de deuda a corto plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_798"/>
    </record>
    <record id="account_group_7988" model="account.group">
        <field name="code_prefix">7988</field>
        <field name="name">Reversión del deterioro en valores representativos de deuda a corto plazo, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_798"/>
    </record>
    <record id="account_group_799" model="account.group">
        <field name="code_prefix">799</field>
        <field name="name">Reversión del deterioro de créditos a corto plazo</field>
        <field name="parent_id" ref="l10n_es.account_group_79"/>
    </record>
    <record id="account_group_7990" model="account.group">
        <field name="code_prefix">7990</field>
        <field name="name">Reversión del deterioro de créditos a corto plazo, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_799"/>
    </record>
    <record id="account_group_7991" model="account.group">
        <field name="code_prefix">7991</field>
        <field name="name">Reversión del deterioro de créditos a corto plazo, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_799"/>
    </record>
    <record id="account_group_7992" model="account.group">
        <field name="code_prefix">7992</field>
        <field name="name">Reversión del deterioro de créditos a corto plazo, otras partes vinculadas</field>
        <field name="parent_id" ref="l10n_es.account_group_799"/>
    </record>
    <record id="account_group_7993" model="account.group">
        <field name="code_prefix">7993</field>
        <field name="name">Reversión del deterioro de créditos a corto plazo, otras empresas</field>
        <field name="parent_id" ref="l10n_es.account_group_799"/>
    </record>
    <record id="account_group_8" model="account.group">
        <field name="code_prefix">8</field>
        <field name="name">Gastos imputados al patrimonio neto</field>
    </record>
    <record id="account_group_80" model="account.group">
        <field name="code_prefix">80</field>
        <field name="name">Gastos financieros por valoración de activos y pasivos</field>
        <field name="parent_id" ref="l10n_es.account_group_8"/>
    </record>
    <record id="account_group_800" model="account.group">
        <field name="code_prefix">800</field>
        <field name="name">Pérdidas en activos financieros disponibles para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_80"/>
    </record>
    <record id="account_group_802" model="account.group">
        <field name="code_prefix">802</field>
        <field name="name">Transferencia de beneficios en activos financieros disponibles para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_80"/>
    </record>
    <record id="account_group_81" model="account.group">
        <field name="code_prefix">81</field>
        <field name="name">Gastos en operaciones de cobertura</field>
        <field name="parent_id" ref="l10n_es.account_group_8"/>
    </record>
    <record id="account_group_810" model="account.group">
        <field name="code_prefix">810</field>
        <field name="name">Pérdidas por coberturas de flujos de efectivo</field>
        <field name="parent_id" ref="l10n_es.account_group_81"/>
    </record>
    <record id="account_group_811" model="account.group">
        <field name="code_prefix">811</field>
        <field name="name">Pérdidas por coberturas de inversiones netas en un negocio en el extranjero</field>
        <field name="parent_id" ref="l10n_es.account_group_81"/>
    </record>
    <record id="account_group_812" model="account.group">
        <field name="code_prefix">812</field>
        <field name="name">Transferencia de beneficios por coberturas de flujos de efectivo</field>
        <field name="parent_id" ref="l10n_es.account_group_81"/>
    </record>
    <record id="account_group_813" model="account.group">
        <field name="code_prefix">813</field>
        <field name="name">Transferencia de beneficios por coberturas de inversiones netas en un negocio en el extranjero</field>
        <field name="parent_id" ref="l10n_es.account_group_81"/>
    </record>
    <record id="account_group_82" model="account.group">
        <field name="code_prefix">82</field>
        <field name="name">Gastos por diferencias de conversión</field>
        <field name="parent_id" ref="l10n_es.account_group_8"/>
    </record>
    <record id="account_group_820" model="account.group">
        <field name="code_prefix">820</field>
        <field name="name">Diferencias de conversión negativas</field>
        <field name="parent_id" ref="l10n_es.account_group_82"/>
    </record>
    <record id="account_group_821" model="account.group">
        <field name="code_prefix">821</field>
        <field name="name">Transferencia de diferencias de conversión positivas</field>
        <field name="parent_id" ref="l10n_es.account_group_82"/>
    </record>
    <record id="account_group_83" model="account.group">
        <field name="code_prefix">83</field>
        <field name="name">Impuesto sobre beneficios</field>
        <field name="parent_id" ref="l10n_es.account_group_8"/>
    </record>
    <record id="account_group_830" model="account.group">
        <field name="code_prefix">830</field>
        <field name="name">Impuesto sobre beneficios</field>
        <field name="parent_id" ref="l10n_es.account_group_83"/>
    </record>
    <record id="account_group_8300" model="account.group">
        <field name="code_prefix">8300</field>
        <field name="name">Impuesto corriente</field>
        <field name="parent_id" ref="l10n_es.account_group_830"/>
    </record>
    <record id="account_group_8301" model="account.group">
        <field name="code_prefix">8301</field>
        <field name="name">Impuesto diferido</field>
        <field name="parent_id" ref="l10n_es.account_group_830"/>
    </record>
    <record id="account_group_833" model="account.group">
        <field name="code_prefix">833</field>
        <field name="name">Ajustes negativos en la imposición sobre beneficios</field>
        <field name="parent_id" ref="l10n_es.account_group_83"/>
    </record>
    <record id="account_group_834" model="account.group">
        <field name="code_prefix">834</field>
        <field name="name">Ingresos fiscales por diferencias permanentes</field>
        <field name="parent_id" ref="l10n_es.account_group_83"/>
    </record>
    <record id="account_group_835" model="account.group">
        <field name="code_prefix">835</field>
        <field name="name">Ingresos fiscales por deducciones y bonificaciones</field>
        <field name="parent_id" ref="l10n_es.account_group_83"/>
    </record>
    <record id="account_group_836" model="account.group">
        <field name="code_prefix">836</field>
        <field name="name">Transferencia de diferencias permanentes</field>
        <field name="parent_id" ref="l10n_es.account_group_83"/>
    </record>
    <record id="account_group_837" model="account.group">
        <field name="code_prefix">837</field>
        <field name="name">Transferencia de deducciones y bonificaciones</field>
        <field name="parent_id" ref="l10n_es.account_group_83"/>
    </record>
    <record id="account_group_838" model="account.group">
        <field name="code_prefix">838</field>
        <field name="name">Ajustes positivos en la imposición sobre beneficios</field>
        <field name="parent_id" ref="l10n_es.account_group_83"/>
    </record>
    <record id="account_group_84" model="account.group">
        <field name="code_prefix">84</field>
        <field name="name">Transferencias de subvenciones, donaciones y legados</field>
        <field name="parent_id" ref="l10n_es.account_group_8"/>
    </record>
    <record id="account_group_840" model="account.group">
        <field name="code_prefix">840</field>
        <field name="name">Transferencia de subvenciones oficiales de capital</field>
        <field name="parent_id" ref="l10n_es.account_group_84"/>
    </record>
    <record id="account_group_841" model="account.group">
        <field name="code_prefix">841</field>
        <field name="name">Transferencia de donaciones y legados de capital</field>
        <field name="parent_id" ref="l10n_es.account_group_84"/>
    </record>
    <record id="account_group_842" model="account.group">
        <field name="code_prefix">842</field>
        <field name="name">Transferencia de otras subvenciones, donaciones y legados</field>
        <field name="parent_id" ref="l10n_es.account_group_84"/>
    </record>
    <record id="account_group_85" model="account.group">
        <field name="code_prefix">85</field>
        <field name="name">Gastos por pérdidas actuariales y ajustes en los activos por retribuciones a largo plazo de prestación definida</field>
        <field name="parent_id" ref="l10n_es.account_group_8"/>
    </record>
    <record id="account_group_850" model="account.group">
        <field name="code_prefix">850</field>
        <field name="name">Pérdidas actuariales</field>
        <field name="parent_id" ref="l10n_es.account_group_85"/>
    </record>
    <record id="account_group_851" model="account.group">
        <field name="code_prefix">851</field>
        <field name="name">Ajustes negativos en activos por retribuciones a largo plazo de prestación definida</field>
        <field name="parent_id" ref="l10n_es.account_group_85"/>
    </record>
    <record id="account_group_86" model="account.group">
        <field name="code_prefix">86</field>
        <field name="name">Gastos por activos no corrientes en venta</field>
        <field name="parent_id" ref="l10n_es.account_group_8"/>
    </record>
    <record id="account_group_860" model="account.group">
        <field name="code_prefix">860</field>
        <field name="name">Pérdidas en activos no corrientes y grupos enajenables de elementos mantenidos para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_86"/>
    </record>
    <record id="account_group_862" model="account.group">
        <field name="code_prefix">862</field>
        <field name="name">Transferencia de beneficios en activos no corrientes y grupos enajenables de elementos mantenidos para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_86"/>
    </record>
    <record id="account_group_89" model="account.group">
        <field name="code_prefix">89</field>
        <field name="name">Gastos de participaciones en empresas del grupo o asociadas con ajustes valorativos positivos previos</field>
        <field name="parent_id" ref="l10n_es.account_group_8"/>
    </record>
    <record id="account_group_891" model="account.group">
        <field name="code_prefix">891</field>
        <field name="name">Deterioro de participaciones en el patrimonio, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_89"/>
    </record>
    <record id="account_group_892" model="account.group">
        <field name="code_prefix">892</field>
        <field name="name">Deterioro de participaciones en el patrimonio, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_89"/>
    </record>
    <record id="account_group_9" model="account.group">
        <field name="code_prefix">9</field>
        <field name="name">Ingresos imputados al patrimonio neto</field>
    </record>
    <record id="account_group_90" model="account.group">
        <field name="code_prefix">90</field>
        <field name="name">Ingresos financieros por valoración de activosy pasivos</field>
        <field name="parent_id" ref="l10n_es.account_group_9"/>
    </record>
    <record id="account_group_900" model="account.group">
        <field name="code_prefix">900</field>
        <field name="name">Beneficios en activos financieros disponibles para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_90"/>
    </record>
    <record id="account_group_902" model="account.group">
        <field name="code_prefix">902</field>
        <field name="name">Transferencia de pérdidas de activos financieros disponibles para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_90"/>
    </record>
    <record id="account_group_91" model="account.group">
        <field name="code_prefix">91</field>
        <field name="name">Ingresos en operaciones de cobertura</field>
        <field name="parent_id" ref="l10n_es.account_group_9"/>
    </record>
    <record id="account_group_910" model="account.group">
        <field name="code_prefix">910</field>
        <field name="name">Beneficios por coberturas de flujos de efectivo</field>
        <field name="parent_id" ref="l10n_es.account_group_91"/>
    </record>
    <record id="account_group_911" model="account.group">
        <field name="code_prefix">911</field>
        <field name="name">Beneficios por coberturas de una inversión neta en un negocio en el extranjero</field>
        <field name="parent_id" ref="l10n_es.account_group_91"/>
    </record>
    <record id="account_group_912" model="account.group">
        <field name="code_prefix">912</field>
        <field name="name">Transferencia de pérdidas por coberturas de flujos de efectivo</field>
        <field name="parent_id" ref="l10n_es.account_group_91"/>
    </record>
    <record id="account_group_913" model="account.group">
        <field name="code_prefix">913</field>
        <field name="name">Transferencia de pérdidas por coberturas de una inversión neta en un negocio en el extranjero</field>
        <field name="parent_id" ref="l10n_es.account_group_91"/>
    </record>
    <record id="account_group_92" model="account.group">
        <field name="code_prefix">92</field>
        <field name="name">Ingresos por diferencias de conversión</field>
        <field name="parent_id" ref="l10n_es.account_group_9"/>
    </record>
    <record id="account_group_920" model="account.group">
        <field name="code_prefix">920</field>
        <field name="name">Diferencias de conversión positivas</field>
        <field name="parent_id" ref="l10n_es.account_group_92"/>
    </record>
    <record id="account_group_921" model="account.group">
        <field name="code_prefix">921</field>
        <field name="name">Transferencia de diferencias de conversión negativas</field>
        <field name="parent_id" ref="l10n_es.account_group_92"/>
    </record>
    <record id="account_group_94" model="account.group">
        <field name="code_prefix">94</field>
        <field name="name">Ingresos por subvenciones, donaciones y legados</field>
        <field name="parent_id" ref="l10n_es.account_group_9"/>
    </record>
    <record id="account_group_940" model="account.group">
        <field name="code_prefix">940</field>
        <field name="name">Ingresos de subvenciones oficiales de capital</field>
        <field name="parent_id" ref="l10n_es.account_group_94"/>
    </record>
    <record id="account_group_941" model="account.group">
        <field name="code_prefix">941</field>
        <field name="name">Ingresos de donaciones y legados de capital</field>
        <field name="parent_id" ref="l10n_es.account_group_94"/>
    </record>
    <record id="account_group_942" model="account.group">
        <field name="code_prefix">942</field>
        <field name="name">Ingresos de otras subvenciones, donaciones y legados</field>
        <field name="parent_id" ref="l10n_es.account_group_94"/>
    </record>
    <record id="account_group_95" model="account.group">
        <field name="code_prefix">95</field>
        <field name="name">Ingresos por ganancias actuariales y ajustes en los activos por retribuciones a largo plazo de prestación definida</field>
        <field name="parent_id" ref="l10n_es.account_group_9"/>
    </record>
    <record id="account_group_950" model="account.group">
        <field name="code_prefix">950</field>
        <field name="name">Ganancias actuariales</field>
        <field name="parent_id" ref="l10n_es.account_group_95"/>
    </record>
    <record id="account_group_951" model="account.group">
        <field name="code_prefix">951</field>
        <field name="name">Ajustes positivos en activos por retribuciones a largo plazo de prestación definida</field>
        <field name="parent_id" ref="l10n_es.account_group_95"/>
    </record>
    <record id="account_group_96" model="account.group">
        <field name="code_prefix">96</field>
        <field name="name">Ingresos por activos no corrientes en venta</field>
        <field name="parent_id" ref="l10n_es.account_group_9"/>
    </record>
    <record id="account_group_960" model="account.group">
        <field name="code_prefix">960</field>
        <field name="name">Beneficios en activos no corrientes y grupos enajenables de elementos mantenidos para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_96"/>
    </record>
    <record id="account_group_962" model="account.group">
        <field name="code_prefix">962</field>
        <field name="name">Transferencia de pérdidas en activos no corrientes y grupos enajenables de elementos mantenidos para la venta</field>
        <field name="parent_id" ref="l10n_es.account_group_96"/>
    </record>
    <record id="account_group_99" model="account.group">
        <field name="code_prefix">99</field>
        <field name="name">Ingresos de participaciones en empresas del grupo o asociadas con ajustes valorativos negativos previos</field>
        <field name="parent_id" ref="l10n_es.account_group_9"/>
    </record>
    <record id="account_group_991" model="account.group">
        <field name="code_prefix">991</field>
        <field name="name">Recuperación de ajustes valorativos negativos previos, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_99"/>
    </record>
    <record id="account_group_992" model="account.group">
        <field name="code_prefix">992</field>
        <field name="name">Recuperación de ajustes valorativos negativos previos, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_99"/>
    </record>
    <record id="account_group_993" model="account.group">
        <field name="code_prefix">993</field>
        <field name="name">Transferencia por deterioro de ajustes valorativos negativos previos, empresas del grupo</field>
        <field name="parent_id" ref="l10n_es.account_group_99"/>
    </record>
    <record id="account_group_994" model="account.group">
        <field name="code_prefix">994</field>
        <field name="name">Transferencia por deterioro de ajustes valorativos negativos previos, empresas asociadas</field>
        <field name="parent_id" ref="l10n_es.account_group_99"/>
    </record>    
    </data>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- © 2009-2011 Jordi Esteve - Zikzakmedia
     © 2011 Ignacio Ibeas - Acysos
     © 2014 Pablo Cayuela - Aserti Global Solutions
     © 2014 Ángel Moya - Domatix
     © 2017 Ángel Moya - PESOL
     © 2015 Carlos Liébana - Factor Libre
     © 2015 Albert Cabedo - GAFIC consultores
     © 2015 Vicent Cubells
     © 2013-2020 Pedro M. Baeza
     © 2020 Harald Panten - Sygel Technology
     © 2022 Omar Castiñeira - Comunitea
     License AGPL-3.0 or later (http://www.gnu.org/licenses/agpl). -->
<odoo>

    <record id="mod_111_02" model="account.account.tag">
        <field name="name">mod111[02]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_111_03" model="account.account.tag">
        <field name="name">mod111[03]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_111_05" model="account.account.tag">
        <field name="name">mod111[05]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_111_06" model="account.account.tag">
        <field name="name">mod111[06]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_111_08" model="account.account.tag">
        <field name="name">mod111[08]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_111_09" model="account.account.tag">
        <field name="name">mod111[09]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>

    <record id="mod_115_02" model="account.account.tag">
        <field name="name">mod115[02]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_115_03" model="account.account.tag">
        <field name="name">mod115[03]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>

    <record id="mod_303_150" model="account.account.tag">
        <field name="name">mod303[150]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_152" model="account.account.tag">
        <field name="name">mod303[152]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_01" model="account.account.tag">
        <field name="name">mod303[01]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_03" model="account.account.tag">
        <field name="name">mod303[03]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_153" model="account.account.tag">
        <field name="name">mod303[153]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_155" model="account.account.tag">
        <field name="name">mod303[155]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_04" model="account.account.tag">
        <field name="name">mod303[04]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_06" model="account.account.tag">
        <field name="name">mod303[06]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_07" model="account.account.tag">
        <field name="name">mod303[07]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_09" model="account.account.tag">
        <field name="name">mod303[09]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_10" model="account.account.tag">
        <field name="name">mod303[10]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_11" model="account.account.tag">
        <field name="name">mod303[11]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_12" model="account.account.tag">
        <field name="name">mod303[12]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_13" model="account.account.tag">
        <field name="name">mod303[13]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_14_sale" model="account.account.tag">
        <field name="name">mod303[14]sale</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_14_purchase" model="account.account.tag">
        <field name="name">mod303[14]purchase</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_15" model="account.account.tag">
        <field name="name">mod303[15]purchase</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_156" model="account.account.tag">
        <field name="name">mod303[156]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_158" model="account.account.tag">
        <field name="name">mod303[158]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_16" model="account.account.tag">
        <field name="name">mod303[16]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_18" model="account.account.tag">
        <field name="name">mod303[18]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_19" model="account.account.tag">
        <field name="name">mod303[19]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_21" model="account.account.tag">
        <field name="name">mod303[21]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_22" model="account.account.tag">
        <field name="name">mod303[22]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_24" model="account.account.tag">
        <field name="name">mod303[24]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_25" model="account.account.tag">
        <field name="name">mod303[25]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_26" model="account.account.tag">
        <field name="name">mod303[26]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_28" model="account.account.tag">
        <field name="name">mod303[28]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_29" model="account.account.tag">
        <field name="name">mod303[29]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_30" model="account.account.tag">
        <field name="name">mod303[30]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_31" model="account.account.tag">
        <field name="name">mod303[31]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_32" model="account.account.tag">
        <field name="name">mod303[32]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_33" model="account.account.tag">
        <field name="name">mod303[33]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_34" model="account.account.tag">
        <field name="name">mod303[34]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_35" model="account.account.tag">
        <field name="name">mod303[35]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_36" model="account.account.tag">
        <field name="name">mod303[36]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_37" model="account.account.tag">
        <field name="name">mod303[37]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_38" model="account.account.tag">
        <field name="name">mod303[38]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_39" model="account.account.tag">
        <field name="name">mod303[39]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_40" model="account.account.tag">
        <field name="name">mod303[40]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_41" model="account.account.tag">
        <field name="name">mod303[41]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_42" model="account.account.tag">
        <field name="name">mod303[42]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_59" model="account.account.tag">
        <field name="name">mod303[59]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_60" model="account.account.tag">
        <field name="name">mod303[60]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_61" model="account.account.tag">
        <!--Not used anymore since Q3 2021; replaced by grids 120, 122, 123 and 124-->
        <field name="name">mod303[61]</field>
        <field name="applicability">taxes</field>
        <field name="active" eval="False"/>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_120" model="account.account.tag">
        <field name="name">mod303[120]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_122" model="account.account.tag">
        <field name="name">mod303[122]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_123" model="account.account.tag">
        <field name="name">mod303[123]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>
    <record id="mod_303_124" model="account.account.tag">
        <field name="name">mod303[124]</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.es"/>
    </record>

    <record id="account_tax_template_s_iva21b" model="account.tax.template">
        <field name="sequence" eval="0"/> <!-- Para que sea el impuesto por defecto de ventas -->
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">sale</field>
        <field name="name">IVA 21% (Bienes)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="21"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_07')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_09')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_14_sale')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_iva21s" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">sale</field>
        <field name="name">IVA 21% (Servicios)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="21"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_07')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_09')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_14_sale')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_iva21isp" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">sale</field>
        <field name="name">IVA 21% (ISP)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="21"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_07')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_09')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_14_sale')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva21_bc" model="account.tax.template">
        <field name="sequence" eval="0"/> <!-- Para que sea el impuesto por defecto de compras -->
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">21% IVA soportado (bienes corrientes)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="21"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_29')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_41')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva21_sc" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">21% IVA soportado (servicios corrientes)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="21"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_29')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva21_sp_in" model="account.tax.template">
        <field name="name">IVA 21% Adquisición de servicios intracomunitarios</field>
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount_type">percent</field>
        <field name="amount" eval="21"/>
        <field name="tax_group_id" ref="tax_group_iva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_10'), ref('mod_303_36')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_37')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_11')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_14_purchase'), ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva21_ic_bc" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="amount_type">percent</field>
        <field name="amount" eval="21"/>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 21% Adquisición Intracomunitaria. Bienes corrientes</field>
        <field name="tax_group_id" ref="tax_group_iva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_36'), ref('mod_303_10')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_37')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_11')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva21_ic_bi" model="account.tax.template">
        <field name="name">IVA 21% Adquisición Intracomunitaria. Bienes de inversión</field>
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="21"/>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="tax_group_id" ref="tax_group_iva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_10'), ref('mod_303_38')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_39')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_11')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva21_ibc" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 21% Importaciones bienes corrientes</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="21"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_32')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_33')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva21_ibi" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 21% Importaciones bienes de inversión</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="21"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_34')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_35')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_irpf21td" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">Retenciones IRPF (Trabajadores) dinerarios</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <!-- Obtained from https://www.agenciatributaria.es/AEAT.internet/Inicio/La_Agencia_Tributaria/Campanas/Retenciones/Cuadro_informativo_tipos_de_retencion_aplicables__2020_.shtml -->
        <field name="amount" eval="-15"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_15"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_02')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_03')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_02')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_03')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva4_sp_ex" model="account.tax.template">
        <field name="amount" eval="4"/>
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 4% Adquisición de servicios extracomunitarios</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_12'), ref('mod_303_28')],
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_13')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_29')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_14_purchase'), ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva10_sp_ex" model="account.tax.template">
        <field name="amount" eval="10"/>
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 10% Adquisición de servicios extracomunitarios</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28'), ref('mod_303_12')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_29')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_13')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva21_sp_ex" model="account.tax.template">
        <field name="name">IVA 21% Adquisición de servicios extracomunitarios</field>
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="amount" eval="21"/>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28'), ref('mod_303_12')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_29')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_13')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva4_ic_bc" model="account.tax.template">
        <field name="amount" eval="4"/>
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 4% Adquisición Intracomunitario. Bienes corrientes</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_36'), ref('mod_303_10')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_37')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_11')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva4_ic_bi" model="account.tax.template">
        <field name="amount" eval="4"/>
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 4% Adquisición Intracomunitario. Bienes de inversión</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_38'), ref('mod_303_10')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_39')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_11')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva10_ic_bc" model="account.tax.template">
        <field name="amount" eval="10"/>
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 10% Adquisición Intracomunitario. Bienes corrientes</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_36'), ref('mod_303_10')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_37')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_11')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva10_ic_bi" model="account.tax.template">
        <field name="amount" eval="10"/>
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 10% Adquisición Intracomunitario. Bienes de inversión</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_38'), ref('mod_303_10')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_39')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_11')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva5_ic_bc" model="account.tax.template">
        <field name="amount" eval="5"/>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 5% Adquisición Intracomunitario. Bienes corrientes</field>
        <field name="tax_group_id" ref="tax_group_iva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_36'), ref('mod_303_10')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_37')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_11')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva5_ic_sc" model="account.tax.template">
        <field name="amount" eval="5"/>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 5% Adquisición de servicios intracomunitarios</field>
        <field name="tax_group_id" ref="tax_group_iva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_36'), ref('mod_303_10')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_37')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_11')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva0_ic_bc" model="account.tax.template">
        <field name="amount" eval="0"/>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 0% Adquisición Intracomunitario. Bienes corrientes</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_36'), ref('mod_303_10')],
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
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva0_ic_sc" model="account.tax.template">
        <field name="amount" eval="0"/>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 0% Adquisición de servicios intracomunitarios</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_36'), ref('mod_303_10')],
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
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_iva0_sp_i" model="account.tax.template">
        <field name="description">Intracomunitario exento (Servicios)</field>
        <field name="type_tax_use">sale</field>
        <field name="name">IVA 0% Prestación de servicios intracomunitario</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_59')],
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
                'tag_ids': [ref('mod_303_59')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_iva_ns" model="account.tax.template">
        <field name="description">No sujeto (Servicios)</field>
        <field name="type_tax_use">sale</field>
        <field name="name">No sujeto Repercutido (Servicios)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_120')],
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
                'tag_ids': [ref('mod_303_120')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_iva_ns_b" model="account.tax.template">
        <field name="description">No sujeto (Bienes)</field>
        <field name="type_tax_use">sale</field>
        <field name="name">No sujeto Repercutido (Bienes)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_120')],
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
                'tag_ids': [ref('mod_303_120')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_iva_e" model="account.tax.template">
        <field name="description">Extracomunitario (Servicios)</field>
        <field name="type_tax_use">sale</field>
        <field name="name">IVA 0% Prestación de servicios extracomunitaria</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_122')],
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
                'tag_ids': [ref('mod_303_122')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva4_ibc" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 4% Importaciones bienes corrientes</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="4"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_32')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_33')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva4_ibi" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 4% Importaciones bienes de inversión</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="4"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_34')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_35')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva10_ibc" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 10% Importaciones bienes corrientes</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="10"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_32')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_33')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva10_ibi" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 10% Importaciones bienes de inversión</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="10"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_34')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_35')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva5_ibc" model="account.tax.template">
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 5% Importaciones bienes corrientes</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="5"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_32')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_33')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva5_isc" model="account.tax.template">
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 5% Adquisición de servicios extracomunitarios</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="5"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_12'), ref('mod_303_28')],
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_13')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_29')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_14_purchase'), ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva0_ibc" model="account.tax.template">
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 0% Importaciones bienes corrientes</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_32')],
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
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva0_isc" model="account.tax.template">
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 0% Adquisición de servicios extracomunitarios</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_12'), ref('mod_303_28')],
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
                'tag_ids': [ref('mod_303_14_purchase'), ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva0_s_bc" model="account.tax.template">
        <field name="type_tax_use">purchase</field>
        <field name="name">0% IVA soportado (bienes corrientes)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28')],
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
                'tag_ids': [ref('mod_303_40')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva0_s_sc" model="account.tax.template">
        <field name="type_tax_use">purchase</field>
        <field name="name">0% IVA soportado (servicios corrientes)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28')],
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
                'tag_ids': [ref('mod_303_40')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva4_bi" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">4% IVA Soportado (bienes de inversión)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="4"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_30')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_31')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva4_sc" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">4% IVA soportado (servicios corrientes)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="4"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_29')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva5_bc" model="account.tax.template">
        <field name="type_tax_use">purchase</field>
        <field name="name">5% IVA soportado (bienes corrientes)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="5"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_29')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva5_sc" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">5% IVA soportado (servicios corrientes)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="5"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_29')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva10_bi" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">10% IVA Soportado (bienes de inversión)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="10"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_30')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_31')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva21_bi" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">21% IVA Soportado (bienes de inversión)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="21"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_30')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_31')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva10_bc" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">10% IVA soportado (bienes corrientes)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="10"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_29')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva4_bc" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">4% IVA soportado (bienes corrientes)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="4"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_29')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva10_sc" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">10% IVA soportado (servicios corrientes)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="10"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_29')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_41')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_iva0" model="account.tax.template">
        <field name="description">IVA Exento</field>
        <field name="type_tax_use">sale</field>
        <field name="name">IVA Exento Repercutido Sujeto</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
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
                'account_id': ref('l10n_es.account_common_472'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_iva0_ns" model="account.tax.template">
        <field name="description">IVA Exento No Sujeto</field>
        <field name="type_tax_use">sale</field>
        <field name="name">IVA Exento Repercutido No Sujeto</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
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
    <record id="account_tax_template_s_req0" model="account.tax.template">
        <field name="description">0% Rec. Eq.</field>
        <field name="type_tax_use">sale</field>
        <field name="name">0% Recargo Equivalencia Ventas</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_recargo_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_16')],
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
                'tag_ids': [ref('mod_303_25')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_req05" model="account.tax.template">
        <field name="description">0.50% Rec. Eq.</field>
        <field name="type_tax_use">sale</field>
        <field name="name">0.50% Recargo Equivalencia Ventas</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0.5"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_recargo_0-5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_16')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_18')],
                'account_id': ref('l10n_es.account_common_477'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_25')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_26')],
                'account_id': ref('l10n_es.account_common_477'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_req062" model="account.tax.template">
        <field name="description">0.62% Rec. Eq.</field>
        <field name="type_tax_use">sale</field>
        <field name="name">0.62% Recargo Equivalencia Ventas</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0.62"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_recargo_0-62"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_16')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_18')],
                'account_id': ref('l10n_es.account_common_477'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_25')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_26')],
                'account_id': ref('l10n_es.account_common_477'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_iva4b" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">sale</field>
        <field name="name">IVA 4% (Bienes)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="4"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_01')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_03')],
                'account_id': ref('l10n_es.account_common_477'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_14_sale')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_15')],
                'account_id': ref('l10n_es.account_common_477'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_iva10b" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">sale</field>
        <field name="name">IVA 10% (Bienes)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="10"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_04')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_06')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_14_sale')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva0_nd" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">21% IVA Soportado no deducible</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="21"/>
        <field name="amount_type">percent</field>
        <field name="analytic" eval="True"/>
        <field name="tax_group_id" ref="tax_group_iva_nd"/>
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
    <record id="account_tax_template_p_iva10_nd" model="account.tax.template">
        <field name="type_tax_use">purchase</field>
        <field name="name">10% IVA Soportado no deducible</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="10"/>
        <field name="amount_type">percent</field>
        <field name="analytic" eval="True"/>
        <field name="tax_group_id" ref="tax_group_iva_nd"/>
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
    <record id="account_tax_template_p_iva4_nd" model="account.tax.template">
        <field name="type_tax_use">purchase</field>
        <field name="name">4% IVA Soportado no deducible</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="4"/>
        <field name="amount_type">percent</field>
        <field name="analytic" eval="True"/>
        <field name="tax_group_id" ref="tax_group_iva_nd"/>
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
    <record id="account_tax_template_s_iva0s" model="account.tax.template">
        <field name="type_tax_use">sale</field>
        <field name="name">IVA 0% (Servicios)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_153')],
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
                'tag_ids': [ref('mod_303_14_sale')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_iva0b" model="account.tax.template">
        <field name="type_tax_use">sale</field>
        <field name="name">IVA 0% (Bienes)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_153')],
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
                'tag_ids': [ref('mod_303_14_sale')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_iva4s" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">sale</field>
        <field name="name">IVA 4% (Servicios)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="4"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_01')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_03')],
                'account_id': ref('l10n_es.account_common_477'),
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_14_sale')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_15')],
                'account_id': ref('l10n_es.account_common_477'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_iva5s" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">sale</field>
        <field name="name">IVA 5% (Servicios)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="5"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_153')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_155')],
                'account_id': ref('l10n_es.account_common_477'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_14_sale')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_15')],
                'account_id': ref('l10n_es.account_common_477'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_iva5b" model="account.tax.template">
        <field name="type_tax_use">sale</field>
        <field name="name">IVA 5% (Bienes)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="5"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_153')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_155')],
                'account_id': ref('l10n_es.account_common_477'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_14_sale')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_15')],
                'account_id': ref('l10n_es.account_common_477'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_iva10s" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">sale</field>
        <field name="name">IVA 10% (Servicios)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="10"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_04')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_06')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_14_sale')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_req014" model="account.tax.template">
        <field name="description">1.4% Rec. Eq.</field>
        <field name="type_tax_use">sale</field>
        <field name="name">1.4% Recargo Equivalencia Ventas</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="1.4"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_recargo_1-4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_19')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_21')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_25')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_26')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_req52" model="account.tax.template">
        <field name="description">5.2% Rec. Eq.</field>
        <field name="type_tax_use">sale</field>
        <field name="name">5.2% Recargo Equivalencia Ventas</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="5.2"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_recargo_5-2"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_22')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_24')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_25')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_26')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva0_bc" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA Soportado exento (operaciones corrientes)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
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
    <record id="account_tax_template_p_iva0_ns" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA Soportado no sujeto (Servicios)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
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
    <record id="account_tax_template_p_iva0_ns_b" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA Soportado no sujeto (Bienes)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
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
    <record id="account_tax_template_s_irpf9" model="account.tax.template">
        <field name="description">Retención 9%</field>
        <field name="type_tax_use">sale</field>
        <field name="name">Retenciones a cuenta IRPF 9%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-9"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_473'),
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
                'account_id': ref('l10n_es.account_common_473'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_irpf18" model="account.tax.template">
        <field name="description">Retención 18%</field>
        <field name="type_tax_use">sale</field>
        <field name="name">Retenciones a cuenta IRPF 18%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-18"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_18"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_473'),
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
                'account_id': ref('l10n_es.account_common_473'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_irpf19" model="account.tax.template">
        <field name="description">Retención 19%</field>
        <field name="type_tax_use">sale</field>
        <field name="name">Retenciones a cuenta IRPF 19%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-19"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_473'),
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
                'account_id': ref('l10n_es.account_common_473'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_irpf19a" model="account.tax.template">
        <field name="description">Retención 19% (Arrend.)</field>
        <field name="type_tax_use">sale</field>
        <field name="name">Retenciones a cuenta 19% (Arrendamientos)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-19"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_473'),
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
                'account_id': ref('l10n_es.account_common_473'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_irpf195a" model="account.tax.template">
        <field name="description">Retención 19,5% (Arrend.)</field>
        <field name="type_tax_use">sale</field>
        <field name="name">Retenciones a cuenta 19,5% (Arrendamientos)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-19.5"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_19-5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_473'),
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
                'account_id': ref('l10n_es.account_common_473'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_irpf19" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">Retenciones IRPF 19%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-19"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_irpf20a" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">Retenciones 20% (Arrendamientos)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-20"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_115_02')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_115_03')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_115_02')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_115_03')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_irpf18" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">Retenciones IRPF 18%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-18"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_18"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_irpf19a" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">Retenciones 19% (Arrendamientos)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-19"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_115_02')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_115_03')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_115_02')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_115_03')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_irpf195a" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">Retenciones 19,5% (Arrendamientos)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-19.5"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_19-5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_115_02')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_115_03')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_115_02')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_115_03')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_irpf7" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">Retenciones IRPF 7%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-7"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_7"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_irpf9" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">Retenciones IRPF 9%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-9"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_irpf24" model="account.tax.template">
        <field name="type_tax_use">purchase</field>
        <field name="name">Retenciones IRPF 24%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-24"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_24"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_irpf20" model="account.tax.template">
        <field name="description">Retención 20%</field>
        <field name="type_tax_use">sale</field>
        <field name="name">Retenciones a cuenta IRPF 20%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-20"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_473'),
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
                'account_id': ref('l10n_es.account_common_473'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_irpf20a" model="account.tax.template">
        <field name="description">Retención 20% (Arrend.)</field>
        <field name="type_tax_use">sale</field>
        <field name="name">Retenciones a cuenta 20% (Arrendamientos)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-20"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_473'),
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
                'account_id': ref('l10n_es.account_common_473'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_irpf24" model="account.tax.template">
        <field name="description">Retención 24%</field>
        <field name="type_tax_use">sale</field>
        <field name="name">Retenciones a cuenta IRPF 24%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-24"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_24"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_473'),
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
                'account_id': ref('l10n_es.account_common_473'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva12_agr" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">12% IVA Soportado régimen agricultura</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="12"/>
        <field name="amount_type">percent</field>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_iva_12"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_42')],
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
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_42')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva105_gan" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">10,5% IVA Soportado régimen ganadero o pesca</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="10.5"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_10-5"/>
        <field name="include_base_amount" eval="1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_42')],
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
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_42')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_iva0_e" model="account.tax.template">
        <field name="description">Exportación (Bienes)</field>
        <field name="type_tax_use">sale</field>
        <field name="name">IVA 0% Exportaciones</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_60')],
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
                'tag_ids': [ref('mod_303_60')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_iva0_ic" model="account.tax.template">
        <field name="description">Intracomunitario exento (Bienes)</field>
        <field name="type_tax_use">sale</field>
        <field name="name">IVA 0% Entregas Intracomunitarias exentas</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_59')],
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
                'tag_ids': [ref('mod_303_59')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_req014" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">1.4% Recargo Equivalencia Compras</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="1.4"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_recargo_1-4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_29')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_41')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_req0" model="account.tax.template">
        <field name="type_tax_use">purchase</field>
        <field name="name">0% Recargo Equivalencia Compras</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_recargo_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28')],
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
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_req05" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">0.50% Recargo Equivalencia Compras</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0.5"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_recargo_0-5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_29')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_41')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_req062" model="account.tax.template">
        <field name="type_tax_use">purchase</field>
        <field name="name">0.62% Recargo Equivalencia Compras</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="0.62"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_recargo_0-62"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_29')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_41')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_req52" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">5.2% Recargo Equivalencia Compras</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="5.2"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_recargo_5-2"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_29')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_41')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_irpf1" model="account.tax.template">
        <field name="description">Retención 1%</field>
        <field name="type_tax_use">sale</field>
        <field name="name">Retenciones a cuenta IRPF 1%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-1"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_473'),
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
                'account_id': ref('l10n_es.account_common_473'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_irpf2" model="account.tax.template">
        <field name="description">Retención 2%</field>
        <field name="type_tax_use">sale</field>
        <field name="name">Retenciones a cuenta IRPF 2%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-2"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_2"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_473'),
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
                'account_id': ref('l10n_es.account_common_473'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_irpf21" model="account.tax.template">
        <field name="description">Retención 21%</field>
        <field name="type_tax_use">sale</field>
        <field name="name">Retenciones a cuenta IRPF 21%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-21"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_473'),
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
                'account_id': ref('l10n_es.account_common_473'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_irpf21a" model="account.tax.template">
        <field name="description">Retención 21% (Arrend.)</field>
        <field name="type_tax_use">sale</field>
        <field name="name">Retenciones a cuenta 21% (Arrendamientos)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-21"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_473'),
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
                'account_id': ref('l10n_es.account_common_473'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_irpf7" model="account.tax.template">
        <field name="description">Retención 7%</field>
        <field name="type_tax_use">sale</field>
        <field name="name">Retenciones a cuenta IRPF 7%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-7"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_7"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_473'),
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
                'account_id': ref('l10n_es.account_common_473'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_irpf15" model="account.tax.template">
        <field name="description">Retención 15%</field>
        <field name="type_tax_use">sale</field>
        <field name="name">Retenciones a cuenta IRPF 15%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-15"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_15"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_473'),
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
                'account_id': ref('l10n_es.account_common_473'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_irpf1" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">Retenciones IRPF 1%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-1"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_irpf15" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">Retenciones IRPF 15%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-15"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_15"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_irpf21t" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">Retenciones IRPF (Trabajadores)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <!-- Obtained from https://www.agenciatributaria.es/AEAT.internet/Inicio/La_Agencia_Tributaria/Campanas/Retenciones/Cuadro_informativo_tipos_de_retencion_aplicables__2020_.shtml -->
        <field name="amount" eval="-15"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_15"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_02')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_03')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_02')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_03')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva10_sp_in" model="account.tax.template">
        <field name="amount" eval="10"/>
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 10% Adquisición de servicios intracomunitarios</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_36'), ref('mod_303_10')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_37')],
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_11')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_41')],
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva4_sp_in" model="account.tax.template">
        <field name="amount" eval="4"/>
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="name">IVA 4% Adquisición de servicios intracomunitarios</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_36'), ref('mod_303_10')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_37')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_11')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'tag_ids': [ref('mod_303_41')],
                'account_id': ref('l10n_es.account_common_472'),
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_irpf21te" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">Retenciones IRPF (Trabajadores) en especie</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <!-- Obtained from https://www.agenciatributaria.es/AEAT.internet/Inicio/La_Agencia_Tributaria/Campanas/Retenciones/Cuadro_informativo_tipos_de_retencion_aplicables__2020_.shtml -->
        <field name="amount" eval="-15"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_15"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_05')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_06')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_05')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_06')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_irpf20" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">Retenciones IRPF 20%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-20"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_irpf21a" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">Retenciones 21% (Arrendamientos)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-21"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_115_02')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_115_03')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_115_02')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_115_03')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_irpf21p" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">Retenciones IRPF 21%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-21"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_irpf2" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="type_tax_use">purchase</field>
        <field name="name">Retenciones IRPF 2%</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-2"/>
        <field name="sequence" eval="2"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_2"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_111_08')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
                'tag_ids': [ref('mod_111_09')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_s_iva0_isp" model="account.tax.template">
        <field name="description">IVA 0% ISP</field>
        <field name="name">IVA 0% Venta con Inversión del Sujeto Pasivo</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="0"/>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="tax_group_id" ref="tax_group_iva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_122')],
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
                'tag_ids': [ref('mod_303_122')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva4_isp" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="name">IVA 4% Compra con Inversión del Sujeto Pasivo Nacional</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="4"/>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28'), ref('mod_303_12')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_29')],
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_13')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_41')],
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva10_isp" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="name">IVA 10% Compra con Inversión del Sujeto Pasivo Nacional</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="10"/>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28'), ref('mod_303_12')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_29')],
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_13')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_41')],
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva21_isp" model="account.tax.template">
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="name">IVA 21% Compra con Inversión del Sujeto Pasivo Nacional</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="21"/>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="tax_group_id" ref="tax_group_iva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_28'), ref('mod_303_12')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_29')],
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_13')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_41')],
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva4_isp_bi" model="account.tax.template">
        <field name="name">IVA 4% ISP (bienes de inversión)</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="4"/>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_30'), ref('mod_303_12')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_31')],
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_13')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_41')],
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva10_isp_bi" model="account.tax.template">
        <field name="name">IVA 10% ISP (bienes de inversión)</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="10"/>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_30'), ref('mod_303_12')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_31')],
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_13')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_41')],
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_iva21_isp_bi" model="account.tax.template">
        <field name="name">IVA 21% ISP (bienes de inversión)</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="21"/>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="tax_group_id" ref="tax_group_iva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_30'), ref('mod_303_12')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_31')],
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_13')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [ref('mod_303_40'), ref('mod_303_14_purchase')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_472'),
                'tag_ids': [ref('mod_303_41')],
            }),

            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_477'),
                'tag_ids': [ref('mod_303_15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_rp19" model="account.tax.template">
        <field name="type_tax_use">purchase</field>
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="name">Retenciones 19% (préstamos)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-19"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
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
                'account_id': ref('l10n_es.account_common_4751'),
            }),
        ]"/>
    </record>
    <record id="account_tax_template_p_rrD19" model="account.tax.template">
        <field name="type_tax_use">purchase</field>
        <field name="description"/> <!-- for resetting the value on existing DBs -->
        <field name="name">Retenciones 19% (reparto de dividendos)</field>
        <field name="chart_template_id" ref="l10n_es.account_chart_template_common"/>
        <field name="amount" eval="-19"/>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_retenciones_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_es.account_common_4751'),
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
                'account_id': ref('l10n_es.account_common_4751'),
            }),
        ]"/>
    </record>
</odoo>

```

## File: migrations\13.0.5.0\post-61-tag-split-2021.py

```python
# -*- coding: utf-8 -*-

from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    # For taxes coming from tax templates, replace grid 61 by the right tag.
    # For the other ones, we can't guess what to use, and the user will have to change his
    # config manually, possibly creating a ticket to ask to fix his accounting history.

    def get_taxes_from_templates(templates):
        cr.execute(f"""
            SELECT array_agg(tax.id)
            FROM account_tax tax
            JOIN ir_model_data data
                ON data.model = 'account.tax'
                AND data.res_id = tax.id
                AND data.module = 'l10n_es'
                AND data.name ~ '^[0-9]*_({'|'.join(templates)})\\Z'
        """)

        return cr.fetchone()[0]

    env = api.Environment(cr, SUPERUSER_ID, {})

    templates_mapping = {
        'mod_303_120': ['account_tax_template_s_iva_ns', 'account_tax_template_s_iva_ns_b'],
        'mod_303_122': ['account_tax_template_s_iva_e', 'account_tax_template_s_iva0_isp'],
    }

    # To run in a server action to fix issues on dbs with custom taxes,
    # replace the content of this dict.
    taxes_mapping = {}
    for tag_name, template_names in templates_mapping.items():
        taxes_from_templates = get_taxes_from_templates(template_names)
        if taxes_from_templates:
            taxes_mapping[tag_name] = taxes_from_templates

    old_tag = env.ref('l10n_es.mod_303_61')
    for tag_name, tax_ids in taxes_mapping.items():
        # Grid 61 is only for base repartition.
        # If it was used for taxes repartition, we don't touch it (and it'll require manual check,
        # as the BOE file probably won't pass government checks).

        new_tag = env.ref(f'l10n_es.{tag_name}')

        # Change tax config
        cr.execute("""
            UPDATE account_account_tag_account_tax_repartition_line_rel tax_rep_tag
            SET account_account_tag_id = %s
            FROM account_account_tag new_tag, account_tax_repartition_line repln
            WHERE tax_rep_tag.account_account_tag_id = %s
            AND repln.id = tax_rep_tag.account_tax_repartition_line_id
            AND COALESCE(repln.invoice_tax_id, repln.refund_tax_id) IN %s
        """, [new_tag.id, old_tag.id, tuple(tax_ids)])

        # Change amls in history, starting at Q3 2021 (date of introduction for the new tags)

        # Set tags
        cr.execute("""
            UPDATE account_account_tag_account_move_line_rel aml_tag
            SET account_account_tag_id = %s
            FROM account_move_line aml, account_move_line_account_tax_rel aml_tax
            WHERE aml_tag.account_move_line_id = aml.id
            AND aml_tax.account_move_line_id = aml.id
            AND aml.date >= '2021-07-01'
            AND aml_tax.account_tax_id IN %s
            AND aml_tag.account_account_tag_id = %s
        """, [new_tag.id, tuple(tax_ids), old_tag.id])

        # Fix tax audit string
        cr.execute("""
            UPDATE account_move_line aml
            SET tax_audit = REPLACE(tax_audit, %s, %s)
            FROM account_account_tag_account_move_line_rel aml_tag
            WHERE aml_tag.account_move_line_id = aml.id
            AND aml_tag.account_account_tag_id = %s
        """, [old_tag.name, f"{new_tag.name}:", new_tag.id])

```

