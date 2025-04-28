# Odoo Module: l10n_mx

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

#    Coded by: Alejandro Negrin anegrin@vauxoo.com,
#    Planified by: Alejandro Negrin, Humberto Arocha, Moises Lopez
#    Finance by: Vauxoo.
#    Audited by: Humberto Arocha (hbto@vauxoo.com) y Moises Lopez (moylop260@vauxoo.com)

{
    "name": "Mexico - Accounting",
    "version": "2.1",
    "author": "Vauxoo",
    'category': 'Accounting/Localizations/Account Charts',
    "description": """
Minimal accounting configuration for Mexico.
============================================

This Chart of account is a minimal proposal to be able to use OoB the
accounting feature of Odoo.

This doesn't pretend be all the localization for MX it is just the minimal
data required to start from 0 in mexican localization.

This modules and its content is updated frequently by openerp-mexico team.

With this module you will have:

 - Minimal chart of account tested in production environments.
 - Minimal chart of taxes, to comply with SAT_ requirements.

.. _SAT: http://www.sat.gob.mx/
    """,
    "depends": [
        "account",
    ],
    "data": [
        "data/account.account.tag.csv",
        "data/l10n_mx_chart_data.xml",
        "data/account.account.template.csv",
        "data/l10n_mx_chart_post_data.xml",
        "data/account_tax_group_data.xml",
        "data/account.group.template.csv",
        "data/account_tax_data.xml",
        "data/fiscal_position_data.xml",
        "data/account_chart_template_data.xml",
        "data/res_bank_data.xml",
        "views/partner_view.xml",
        "views/res_bank_view.xml",
        "views/res_config_settings_views.xml",
        "views/account_views.xml",
        "data/l10n_mx_uom.xml",
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.tag.csv

```csv
id,name,applicability,country_id/id,color
tag_iva,IVA,taxes,base.mx,0
tag_isr,ISR,taxes,base.mx,0
tag_ieps,IEPS,taxes,base.mx,0
tag_diot_16,DIOT: 16%,taxes,base.mx,0
tag_diot_16_non_cre,DIOT: 16% NO ACREDITABLE,taxes,base.mx,0
tag_diot_16_imp,DIOT: 16% IMP,taxes,base.mx,0
tag_diot_0,DIOT: 0%,taxes,base.mx,0
tag_diot_8,DIOT: 8%,taxes,base.mx,0
tag_diot_8_non_cre,DIOT: 8% NO ACREDITABLE,taxes,base.mx,0
tag_diot_ret,DIOT: Retención,taxes,base.mx,0
tag_diot_exento,DIOT: Exento,taxes,base.mx,0
tag_debit_balance_account,Debit Balance Account,accounts,base.mx,0
tag_credit_balance_account,Credit Balance Account,accounts,base.mx,0
```

## File: data\account.account.template.csv

```csv
id,name,code,account_type,chart_template_id/id,reconcile,tag_ids/id
cuenta102_02,Transferencias bancarias moneda extranjera,102.02.01,asset_current,l10n_mx.mx_coa,True,l10n_mx.tag_debit_balance_account
cuenta105_01,Clientes nacionales,105.01.01,asset_receivable,l10n_mx.mx_coa,True,l10n_mx.tag_debit_balance_account
cuenta105_02,Clientes nacionales (PoS),105.01.02,asset_receivable,l10n_mx.mx_coa,True,l10n_mx.tag_debit_balance_account
cuenta107_05_01,Mercancías Enviadas - No Facturas,107.05.01,asset_current,l10n_mx.mx_coa,True,l10n_mx.tag_debit_balance_account
cuenta108_01,Estimación de cuentas incobrables nacional,108.01.01,asset_current,l10n_mx.mx_coa,True,l10n_mx.tag_debit_balance_account
cuenta108_02,Estimación de cuentas incobrables extranjero,108.02.01,asset_current,l10n_mx.mx_coa,True,l10n_mx.tag_debit_balance_account
cuenta115_01,Inventario,115.01.01,asset_current,l10n_mx.mx_coa,False,l10n_mx.tag_debit_balance_account
cuenta115_02,Materia prima y materiales,115.02.01,asset_current,l10n_mx.mx_coa,False,l10n_mx.tag_debit_balance_account
cuenta115_03,Producción en proceso,115.03.01,asset_current,l10n_mx.mx_coa,False,l10n_mx.tag_debit_balance_account
cuenta115_04,Productos terminados,115.04.01,asset_current,l10n_mx.mx_coa,False,l10n_mx.tag_debit_balance_account
cuenta115_05,Mercancías en tránsito,115.05.01,asset_current,l10n_mx.mx_coa,True,l10n_mx.tag_debit_balance_account
cuenta115_06,Mercancías en poder de terceros,115.06.01,asset_current,l10n_mx.mx_coa,False,l10n_mx.tag_debit_balance_account
cuenta118_01,IVA acreditable pagado,118.01.01,asset_current,l10n_mx.mx_coa,False,l10n_mx.tag_debit_balance_account
cuenta118_03,IEPS acreditable pagado,118.03.01,asset_current,l10n_mx.mx_coa,False,l10n_mx.tag_debit_balance_account
cuenta119_01,IVA pendiente de pago,119.01.01,asset_current,l10n_mx.mx_coa,True,l10n_mx.tag_debit_balance_account
cuenta119_03,IEPS pendiente de pago,119.03.01,asset_current,l10n_mx.mx_coa,True,l10n_mx.tag_debit_balance_account
cuenta120_01,Anticipo a proveedores nacional,120.01.01,asset_current,l10n_mx.mx_coa,True,l10n_mx.tag_debit_balance_account
cuenta120_02,Anticipo a proveedores extranjero,120.02.01,asset_current,l10n_mx.mx_coa,True,l10n_mx.tag_debit_balance_account
cuenta201_01,Proveedores nacionales,201.01.01,liability_payable,l10n_mx.mx_coa,True,l10n_mx.tag_credit_balance_account
cuenta205_06_01,Mercancías Recibidas - No Facturas,205.06.01,liability_current,l10n_mx.mx_coa,True,l10n_mx.tag_credit_balance_account
cuenta206_01,Anticipo de cliente nacional,206.01.01,liability_current,l10n_mx.mx_coa,True,l10n_mx.tag_credit_balance_account
cuenta206_02,Anticipo de cliente extranjero,206.02.01,liability_current,l10n_mx.mx_coa,True,l10n_mx.tag_credit_balance_account
cuenta206_05_01,Otros anticipos de clientes,206.05.01,liability_current,l10n_mx.mx_coa,True,l10n_mx.tag_credit_balance_account
cuenta208_01,IVA trasladado cobrado,208.01.01,liability_current,l10n_mx.mx_coa,False,l10n_mx.tag_credit_balance_account
cuenta208_02,IEPS trasladado cobrado,208.02.01,liability_current,l10n_mx.mx_coa,False,l10n_mx.tag_credit_balance_account
cuenta209_01,IVA trasladado no cobrado,209.01.01,liability_current,l10n_mx.mx_coa,True,l10n_mx.tag_credit_balance_account
cuenta209_02,IEPS trasladado no cobrado,209.02.01,liability_current,l10n_mx.mx_coa,True,l10n_mx.tag_credit_balance_account
cuenta216_03,Impuestos retenidos de ISR por arrendamiento,216.03.01,liability_current,l10n_mx.mx_coa,False,l10n_mx.tag_credit_balance_account
cuenta216_04,Impuestos retenidos de ISR por servicios profesionales,216.04.01,liability_current,l10n_mx.mx_coa,False,l10n_mx.tag_credit_balance_account
cuenta216_10,Impuestos retenidos de IVA,216.10.10,liability_current,l10n_mx.mx_coa,False,l10n_mx.tag_credit_balance_account
cuenta216_10_20,Impuestos retenidos de iva efectivamente pagados,216.10.20,liability_current,l10n_mx.mx_coa,False,l10n_mx.tag_credit_balance_account
cuenta302_01,Patrimonio,302.01.01,equity,l10n_mx.mx_coa,False,l10n_mx.tag_credit_balance_account
cuenta304_01,Utilidad de ejercicios anteriores,304.01.01,equity,l10n_mx.mx_coa,False,l10n_mx.tag_credit_balance_account
cuenta305_01,Utilidad del ejercicio,305.01.01,equity,l10n_mx.mx_coa,False,l10n_mx.tag_credit_balance_account
cuenta401_01,Ventas y/o servicios gravados a la tasa general,401.01.01,income,l10n_mx.mx_coa,False,l10n_mx.tag_credit_balance_account
cuenta501_01,Costo de venta,501.01.01,expense_direct_cost,l10n_mx.mx_coa,False,l10n_mx.tag_debit_balance_account
cuenta601_84,Otros gastos generales,601.84.01,expense,l10n_mx.mx_coa,False,l10n_mx.tag_debit_balance_account
cuenta701_01,Pérdida cambiaria,701.01.01,expense,l10n_mx.mx_coa,False,l10n_mx.tag_debit_balance_account
cuenta702_01,Utilidad cambiaria,702.01.01,income,l10n_mx.mx_coa,False,l10n_mx.tag_credit_balance_account
cuenta801_01,Utilidad o pérdida fiscal en venta y/o baja de activo fijo,811.01.01,expense,l10n_mx.mx_coa,False,l10n_mx.tag_debit_balance_account
cuenta801_01_99,Base Imponible de Impuestos en Base a Flujo de Efectivo,899.01.99,expense,l10n_mx.mx_coa,False,l10n_mx.tag_debit_balance_account
cuenta9993,Cash Discount Loss,9993,expense,l10n_mx.mx_coa,False,l10n_mx.tag_debit_balance_account
cuenta9994,Cash Discount Gain,9994,income_other,l10n_mx.mx_coa,False,l10n_mx.tag_credit_balance_account

```

## File: data\account.group.template.csv

```csv
"id","chart_template_id/id","name","code_prefix_start"
"account_group_activos","l10n_mx.mx_coa","Activos","1"
"account_subgroup_activo_a_corto_plazo","l10n_mx.mx_coa","Activo a corto plazo","100.01"
"account_subgroup_caja","l10n_mx.mx_coa","Caja","101"
"account_subgroup_caja_y_efectivo","l10n_mx.mx_coa","Caja y efectivo","101.01"
"account_subgroup_bancos","l10n_mx.mx_coa","Bancos","102"
"account_subgroup_bancos_nacionales","l10n_mx.mx_coa","Bancos nacionales","102.01"
"account_subgroup_bancos_extranjeros","l10n_mx.mx_coa","Bancos extranjeros","102.02"
"account_subgroup_inversiones","l10n_mx.mx_coa","Inversiones","103"
"account_subgroup_inversiones_temporales","l10n_mx.mx_coa","Inversiones temporales","103.01"
"account_subgroup_inversiones_en_fideicomisos","l10n_mx.mx_coa","Inversiones en fideicomisos","103.02"
"account_subgroup_otras_inversiones","l10n_mx.mx_coa","Otras inversiones","103.03"
"account_subgroup_otros_instrumentos_financieros","l10n_mx.mx_coa","Otros instrumentos financieros","104"
"account_subgroup_otros_instrumentos_financieros_1","l10n_mx.mx_coa","Otros instrumentos financieros","104.01"
"account_subgroup_clientes","l10n_mx.mx_coa","Clientes","105"
"account_subgroup_clientes_nacionales","l10n_mx.mx_coa","Clientes nacionales","105.01"
"account_subgroup_clientes_extranjeros","l10n_mx.mx_coa","Clientes extranjeros","105.02"
"account_subgroup_clientes_nacionales_parte_relacionada","l10n_mx.mx_coa","Clientes nacionales parte relacionada","105.03"
"account_subgroup_clientes_extranjeros_parte_relacionada","l10n_mx.mx_coa","Clientes extranjeros parte relacionada","105.04"
"account_subgroup_cuentas_y_documentos_por_cobrar_a_corto_plazo","l10n_mx.mx_coa","Cuentas y documentos por cobrar a corto plazo","106"
"account_subgroup_cuentas_y_documentos_por_cobrar_a_corto_plazo_nacional","l10n_mx.mx_coa","Cuentas y documentos por cobrar a corto plazo nacional","106.01"
"account_subgroup_cuentas_y_documentos_por_cobrar_a_corto_plazo_extranjero","l10n_mx.mx_coa","Cuentas y documentos por cobrar a corto plazo extranjero","106.02"
"account_subgroup_cuentas_y_documentos_por_cobrar_a_corto_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Cuentas y documentos por cobrar a corto plazo nacional parte relacionada","106.03"
"account_subgroup_cuentas_y_documentos_por_cobrar_a_corto_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Cuentas y documentos por cobrar a corto plazo extranjero parte relacionada","106.04"
"account_subgroup_intereses_por_cobrar_a_corto_plazo_nacional","l10n_mx.mx_coa","Intereses por cobrar a corto plazo nacional","106.05"
"account_subgroup_intereses_por_cobrar_a_corto_plazo_extranjero","l10n_mx.mx_coa","Intereses por cobrar a corto plazo extranjero","106.06"
"account_subgroup_intereses_por_cobrar_a_corto_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Intereses por cobrar a corto plazo nacional parte relacionada","106.07"
"account_subgroup_intereses_por_cobrar_a_corto_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Intereses por cobrar a corto plazo extranjero parte relacionada","106.08"
"account_subgroup_otras_cuentas_y_documentos_por_cobrar_a_corto_plazo","l10n_mx.mx_coa","Otras cuentas y documentos por cobrar a corto plazo","106.09"
"account_subgroup_otras_cuentas_y_documentos_por_cobrar_a_corto_plazo_parte_relacionada","l10n_mx.mx_coa","Otras cuentas y documentos por cobrar a corto plazo parte relacionada","106.10"
"account_subgroup_deudores_diversos","l10n_mx.mx_coa","Deudores diversos","107"
"account_subgroup_funcionarios_y_empleados","l10n_mx.mx_coa","Funcionarios y empleados","107.01"
"account_subgroup_socios_y_accionistas","l10n_mx.mx_coa","Socios y accionistas","107.02"
"account_subgroup_partes_relacionadas_nacionales","l10n_mx.mx_coa","Partes relacionadas nacionales","107.03"
"account_subgroup_partes_relacionadas_extranjeros","l10n_mx.mx_coa","Partes relacionadas extranjeros","107.04"
"account_subgroup_otros_deudores_diversos","l10n_mx.mx_coa","Otros deudores diversos","107.05"
"account_subgroup_estimación_de_cuentas_incobrables","l10n_mx.mx_coa","Estimación de cuentas incobrables","108"
"account_subgroup_estimación_de_cuentas_incobrables_nacional","l10n_mx.mx_coa","Estimación de cuentas incobrables nacional","108.01"
"account_subgroup_estimación_de_cuentas_incobrables_extranjero","l10n_mx.mx_coa","Estimación de cuentas incobrables extranjero","108.02"
"account_subgroup_estimación_de_cuentas_incobrables_nacional_parte_relacionada","l10n_mx.mx_coa","Estimación de cuentas incobrables nacional parte relacionada","108.03"
"account_subgroup_estimación_de_cuentas_incobrables_extranjero_parte_relacionada","l10n_mx.mx_coa","Estimación de cuentas incobrables extranjero parte relacionada","108.04"
"account_subgroup_pagos_anticipados","l10n_mx.mx_coa","Pagos anticipados","109"
"account_subgroup_seguros_y_fianzas_pagados_por_anticipado_nacional","l10n_mx.mx_coa","Seguros y fianzas pagados por anticipado nacional","109.01"
"account_subgroup_seguros_y_fianzas_pagados_por_anticipado_extranjero","l10n_mx.mx_coa","Seguros y fianzas pagados por anticipado extranjero","109.02"
"account_subgroup_seguros_y_fianzas_pagados_por_anticipado_nacional_parte_relacionada","l10n_mx.mx_coa","Seguros y fianzas pagados por anticipado nacional parte relacionada","109.03"
"account_subgroup_seguros_y_fianzas_pagados_por_anticipado_extranjero_parte_relacionada","l10n_mx.mx_coa","Seguros y fianzas pagados por anticipado extranjero parte relacionada","109.04"
"account_subgroup_rentas_pagados_por_anticipado_nacional","l10n_mx.mx_coa","Rentas pagados por anticipado nacional","109.05"
"account_subgroup_rentas_pagados_por_anticipado_extranjero","l10n_mx.mx_coa","Rentas pagados por anticipado extranjero","109.06"
"account_subgroup_rentas_pagados_por_anticipado_nacional_parte_relacionada","l10n_mx.mx_coa","Rentas pagados por anticipado nacional parte relacionada","109.07"
"account_subgroup_rentas_pagados_por_anticipado_extranjero_parte_relacionada","l10n_mx.mx_coa","Rentas pagados por anticipado extranjero parte relacionada","109.08"
"account_subgroup_intereses_pagados_por_anticipado_nacional","l10n_mx.mx_coa","Intereses pagados por anticipado nacional","109.09"
"account_subgroup_intereses_pagados_por_anticipado_extranjero","l10n_mx.mx_coa","Intereses pagados por anticipado extranjero","109.10"
"account_subgroup_intereses_pagados_por_anticipado_nacional_parte_relacionada","l10n_mx.mx_coa","Intereses pagados por anticipado nacional parte relacionada","109.11"
"account_subgroup_intereses_pagados_por_anticipado_extranjero_parte_relacionada","l10n_mx.mx_coa","Intereses pagados por anticipado extranjero parte relacionada","109.12"
"account_subgroup_factoraje_financiero_pagados_por_anticipado_nacional","l10n_mx.mx_coa","Factoraje financiero pagados por anticipado nacional","109.13"
"account_subgroup_factoraje_financiero_pagados_por_anticipado_extranjero","l10n_mx.mx_coa","Factoraje financiero pagados por anticipado extranjero","109.14"
"account_subgroup_factoraje_financiero_pagados_por_anticipado_nacional_parte_relacionada","l10n_mx.mx_coa","Factoraje financiero pagados por anticipado nacional parte relacionada","109.15"
"account_subgroup_factoraje_financiero_pagados_por_anticipado_extranjero_parte_relacionada","l10n_mx.mx_coa","Factoraje financiero pagados por anticipado extranjero parte relacionada","109.16"
"account_subgroup_arrendamiento_financiero_pagados_por_anticipado_nacional","l10n_mx.mx_coa","Arrendamiento financiero pagados por anticipado nacional","109.17"
"account_subgroup_arrendamiento_financiero_pagados_por_anticipado_extranjero","l10n_mx.mx_coa","Arrendamiento financiero pagados por anticipado extranjero","109.18"
"account_subgroup_arrendamiento_financiero_pagados_por_anticipado_nacional_parte_relacionada","l10n_mx.mx_coa","Arrendamiento financiero pagados por anticipado nacional parte relacionada","109.19"
"account_subgroup_arrendamiento_financiero_pagados_por_anticipado_extranjero_parte_relacionada","l10n_mx.mx_coa","Arrendamiento financiero pagados por anticipado extranjero parte relacionada","109.20"
"account_subgroup_pérdida_por_deterioro_de_pagos_anticipados","l10n_mx.mx_coa","Pérdida por deterioro de pagos anticipados","109.21"
"account_subgroup_derechos_fiduciarios","l10n_mx.mx_coa","Derechos fiduciarios","109.22"
"account_subgroup_otros_pagos_anticipados","l10n_mx.mx_coa","Otros pagos anticipados","109.23"
"account_subgroup_subsidio_al_empleo_por_aplicar","l10n_mx.mx_coa","Subsidio al empleo por aplicar","110"
"account_subgroup_subsidio_al_empleo_por_aplicar_1","l10n_mx.mx_coa","Subsidio al empleo por aplicar","110.01"
"account_subgroup_crédito_al_diesel_por_acreditar","l10n_mx.mx_coa","Crédito al diesel por acreditar","111"
"account_subgroup_crédito_al_diesel_por_acreditar_1","l10n_mx.mx_coa","Crédito al diesel por acreditar","111.01"
"account_subgroup_otros_estímulos","l10n_mx.mx_coa","Otros estímulos","112"
"account_subgroup_otros_estímulos_1","l10n_mx.mx_coa","Otros estímulos","112.01"
"account_subgroup_impuestos_a_favor","l10n_mx.mx_coa","Impuestos a favor","113"
"account_subgroup_iva_a_favor","l10n_mx.mx_coa","IVA a favor","113.01"
"account_subgroup_isr_a_favor","l10n_mx.mx_coa","ISR a favor","113.02"
"account_subgroup_ietu_a_favor","l10n_mx.mx_coa","IETU a favor","113.03"
"account_subgroup_ide_a_favor","l10n_mx.mx_coa","IDE a favor","113.04"
"account_subgroup_ia_a_favor","l10n_mx.mx_coa","IA a favor","113.05"
"account_subgroup_subsidio_al_empleo","l10n_mx.mx_coa","Subsidio al empleo","113.06"
"account_subgroup_pago_de_lo_indebido","l10n_mx.mx_coa","Pago de lo indebido","113.07"
"account_subgroup_otros_impuestos_a_favor","l10n_mx.mx_coa","Otros impuestos a favor","113.08"
"account_subgroup_pagos_provisionales","l10n_mx.mx_coa","Pagos provisionales","114"
"account_subgroup_pagos_provisionales_de_isr","l10n_mx.mx_coa","Pagos provisionales de ISR","114.01"
"account_subgroup_inventario","l10n_mx.mx_coa","Inventario","115"
"account_subgroup_inventario_1","l10n_mx.mx_coa","Inventario","115.01"
"account_subgroup_materia_prima_y_materiales","l10n_mx.mx_coa","Materia prima y materiales","115.02"
"account_subgroup_producción_en_proceso","l10n_mx.mx_coa","Producción en proceso","115.03"
"account_subgroup_productos_terminados","l10n_mx.mx_coa","Productos terminados","115.04"
"account_subgroup_mercancías_en_tránsito","l10n_mx.mx_coa","Mercancías en tránsito","115.05"
"account_subgroup_mercancías_en_poder_de_terceros","l10n_mx.mx_coa","Mercancías en poder de terceros","115.06"
"account_subgroup_otros","l10n_mx.mx_coa","Otros","115.07"
"account_subgroup_estimación_de_inventarios_obsoletos_y_de_lento_movimiento","l10n_mx.mx_coa","Estimación de inventarios obsoletos y de lento movimiento","116"
"account_subgroup_estimación_de_inventarios_obsoletos_y_de_lento_movimiento_1","l10n_mx.mx_coa","Estimación de inventarios obsoletos y de lento movimiento","116.01"
"account_subgroup_obras_en_proceso_de_inmuebles","l10n_mx.mx_coa","Obras en proceso de inmuebles","117"
"account_subgroup_obras_en_proceso_de_inmuebles_1","l10n_mx.mx_coa","Obras en proceso de inmuebles","117.01"
"account_subgroup_impuestos_acreditables_pagados","l10n_mx.mx_coa","Impuestos acreditables pagados","118"
"account_subgroup_iva_acreditable_pagado","l10n_mx.mx_coa","IVA acreditable pagado","118.01"
"account_subgroup_iva_acreditable_de_importación_pagado","l10n_mx.mx_coa","IVA acreditable de importación pagado","118.02"
"account_subgroup_ieps_acreditable_pagado","l10n_mx.mx_coa","IEPS acreditable pagado","118.03"
"account_subgroup_ieps_pagado_en_importación","l10n_mx.mx_coa","IEPS pagado en importación","118.04"
"account_subgroup_impuestos_acreditables_por_pagar","l10n_mx.mx_coa","Impuestos acreditables por pagar","119"
"account_subgroup_iva_pendiente_de_pago","l10n_mx.mx_coa","IVA pendiente de pago","119.01"
"account_subgroup_iva_de_importación_pendiente_de_pago","l10n_mx.mx_coa","IVA de importación pendiente de pago","119.02"
"account_subgroup_ieps_pendiente_de_pago","l10n_mx.mx_coa","IEPS pendiente de pago","119.03"
"account_subgroup_ieps_pendiente_de_pago_en_importación","l10n_mx.mx_coa","IEPS pendiente de pago en importación","119.04"
"account_subgroup_anticipo_a_proveedores","l10n_mx.mx_coa","Anticipo a proveedores","120"
"account_subgroup_anticipo_a_proveedores_nacional","l10n_mx.mx_coa","Anticipo a proveedores nacional","120.01"
"account_subgroup_anticipo_a_proveedores_extranjero","l10n_mx.mx_coa","Anticipo a proveedores extranjero","120.02"
"account_subgroup_anticipo_a_proveedores_nacional_parte_relacionada","l10n_mx.mx_coa","Anticipo a proveedores nacional parte relacionada","120.03"
"account_subgroup_anticipo_a_proveedores_extranjero_parte_relacionada","l10n_mx.mx_coa","Anticipo a proveedores extranjero parte relacionada","120.04"
"account_subgroup_otros_activos_a_corto_plazo","l10n_mx.mx_coa","Otros activos a corto plazo","121"
"account_subgroup_otros_activos_a_corto_plazo_1","l10n_mx.mx_coa","Otros activos a corto plazo","121.01"
"account_subgroup_activo_a_largo_plazo","l10n_mx.mx_coa","Activo a largo plazo","100.02"
"account_subgroup_terrenos","l10n_mx.mx_coa","Terrenos","151"
"account_subgroup_terrenos_1","l10n_mx.mx_coa","Terrenos","151.01"
"account_subgroup_edificios","l10n_mx.mx_coa","Edificios","152"
"account_subgroup_edificios_1","l10n_mx.mx_coa","Edificios","152.01"
"account_subgroup_maquinaria_y_equipo","l10n_mx.mx_coa","Maquinaria y equipo","153"
"account_subgroup_maquinaria_y_equipo_1","l10n_mx.mx_coa","Maquinaria y equipo","153.01"
"account_subgroup_automóviles,_autobuses,_camiones_de_carga,_tractocamiones,_montacargas_y_remolques","l10n_mx.mx_coa","Automóviles, autobuses, camiones de carga, tractocamiones, montacargas y remolques","154"
"account_subgroup_automóviles,_autobuses,_camiones_de_carga,_tractocamiones,_montacargas_y_remolques_1","l10n_mx.mx_coa","Automóviles, autobuses, camiones de carga, tractocamiones, montacargas y remolques","154.01"
"account_subgroup_mobiliario_y_equipo_de_oficina","l10n_mx.mx_coa","Mobiliario y equipo de oficina","155"
"account_subgroup_mobiliario_y_equipo_de_oficina_1","l10n_mx.mx_coa","Mobiliario y equipo de oficina","155.01"
"account_subgroup_equipo_de_cómputo","l10n_mx.mx_coa","Equipo de cómputo","156"
"account_subgroup_equipo_de_cómputo_1","l10n_mx.mx_coa","Equipo de cómputo","156.01"
"account_subgroup_equipo_de_comunicación","l10n_mx.mx_coa","Equipo de comunicación","157"
"account_subgroup_equipo_de_comunicación_1","l10n_mx.mx_coa","Equipo de comunicación","157.01"
"account_subgroup_activos_biológicos,_vegetales_y_semovientes","l10n_mx.mx_coa","Activos biológicos, vegetales y semovientes","158"
"account_subgroup_activos_biológicos,_vegetales_y_semovientes_1","l10n_mx.mx_coa","Activos biológicos, vegetales y semovientes","158.01"
"account_subgroup_obras_en_proceso_de_activos_fijos","l10n_mx.mx_coa","Obras en proceso de activos fijos","159"
"account_subgroup_obras_en_proceso_de_activos_fijos_1","l10n_mx.mx_coa","Obras en proceso de activos fijos","159.01"
"account_subgroup_otros_activos_fijos","l10n_mx.mx_coa","Otros activos fijos","160"
"account_subgroup_otros_activos_fijos_1","l10n_mx.mx_coa","Otros activos fijos","160.01"
"account_subgroup_ferrocarriles","l10n_mx.mx_coa","Ferrocarriles","161"
"account_subgroup_ferrocarriles_1","l10n_mx.mx_coa","Ferrocarriles","161.01"
"account_subgroup_embarcaciones","l10n_mx.mx_coa","Embarcaciones","162"
"account_subgroup_embarcaciones_1","l10n_mx.mx_coa","Embarcaciones","162.01"
"account_subgroup_aviones","l10n_mx.mx_coa","Aviones","163"
"account_subgroup_aviones_1","l10n_mx.mx_coa","Aviones","163.01"
"account_subgroup_troqueles,_moldes,_matrices_y_herramental","l10n_mx.mx_coa","Troqueles, moldes, matrices y herramental","164"
"account_subgroup_troqueles,_moldes,_matrices_y_herramental_1","l10n_mx.mx_coa","Troqueles, moldes, matrices y herramental","164.01"
"account_subgroup_equipo_de_comunicaciones_telefónicas","l10n_mx.mx_coa","Equipo de comunicaciones telefónicas","165"
"account_subgroup_equipo_de_comunicaciones_telefónicas_1","l10n_mx.mx_coa","Equipo de comunicaciones telefónicas","165.01"
"account_subgroup_equipo_de_comunicación_satelital","l10n_mx.mx_coa","Equipo de comunicación satelital","166"
"account_subgroup_equipo_de_comunicación_satelital_1","l10n_mx.mx_coa","Equipo de comunicación satelital","166.01"
"account_subgroup_equipo_de_adaptaciones_para_personas_con_capacidades_diferentes","l10n_mx.mx_coa","Equipo de adaptaciones para personas con capacidades diferentes","167"
"account_subgroup_equipo_de_adaptaciones_para_personas_con_capacidades_diferentes_1","l10n_mx.mx_coa","Equipo de adaptaciones para personas con capacidades diferentes","167.01"
"account_subgroup_maquinaria_y_equipo_de_generación_de_energía_de_fuentes_renovables_o_de_sistemas_de_cogeneración_de_electricidad_eficiente","l10n_mx.mx_coa","Maquinaria y equipo de generación de energía de fuentes renovables o de sistemas de cogeneración de electricidad eficiente","168"
"account_subgroup_maquinaria_y_equipo_de_generación_de_energía_de_fuentes_renovables_o_de_sistemas_de_cogeneración_de_electricidad_eficiente_1","l10n_mx.mx_coa","Maquinaria y equipo de generación de energía de fuentes renovables o de sistemas de cogeneración de electricidad eficiente","168.01"
"account_subgroup_otra_maquinaria_y_equipo","l10n_mx.mx_coa","Otra maquinaria y equipo","169"
"account_subgroup_otra_maquinaria_y_equipo_1","l10n_mx.mx_coa","Otra maquinaria y equipo","169.01"
"account_subgroup_adaptaciones_y_mejoras","l10n_mx.mx_coa","Adaptaciones y mejoras","170"
"account_subgroup_adaptaciones_y_mejoras_1","l10n_mx.mx_coa","Adaptaciones y mejoras","170.01"
"account_subgroup_depreciación_acumulada_de_activos_fijos","l10n_mx.mx_coa","Depreciación acumulada de activos fijos","171"
"account_subgroup_depreciación_acumulada_de_edificios","l10n_mx.mx_coa","Depreciación acumulada de edificios","171.01"
"account_subgroup_depreciación_acumulada_de_maquinaria_y_equipo","l10n_mx.mx_coa","Depreciación acumulada de maquinaria y equipo","171.02"
"account_subgroup_depreciación_acumulada_de_automóviles,_autobuses,_camiones_de_carga,_tractocamiones,_montacargas_y_remolques","l10n_mx.mx_coa","Depreciación acumulada de automóviles, autobuses, camiones de carga, tractocamiones, montacargas y remolques","171.03"
"account_subgroup_depreciación_acumulada_de_mobiliario_y_equipo_de_oficina","l10n_mx.mx_coa","Depreciación acumulada de mobiliario y equipo de oficina","171.04"
"account_subgroup_depreciación_acumulada_de_equipo_de_cómputo","l10n_mx.mx_coa","Depreciación acumulada de equipo de cómputo","171.05"
"account_subgroup_depreciación_acumulada_de_equipo_de_comunicación","l10n_mx.mx_coa","Depreciación acumulada de equipo de comunicación","171.06"
"account_subgroup_depreciación_acumulada_de_activos_biológicos,_vegetales_y_semovientes","l10n_mx.mx_coa","Depreciación acumulada de activos biológicos, vegetales y semovientes","171.07"
"account_subgroup_depreciación_acumulada_de_otros_activos_fijos","l10n_mx.mx_coa","Depreciación acumulada de otros activos fijos","171.08"
"account_subgroup_depreciación_acumulada_de_ferrocarriles","l10n_mx.mx_coa","Depreciación acumulada de ferrocarriles","171.09"
"account_subgroup_depreciación_acumulada_de_embarcaciones","l10n_mx.mx_coa","Depreciación acumulada de embarcaciones","171.10"
"account_subgroup_depreciación_acumulada_de_aviones","l10n_mx.mx_coa","Depreciación acumulada de aviones","171.11"
"account_subgroup_depreciación_acumulada_de_troqueles,_moldes,_matrices_y_herramental","l10n_mx.mx_coa","Depreciación acumulada de troqueles, moldes, matrices y herramental","171.12"
"account_subgroup_depreciación_acumulada_de_equipo_de_comunicaciones_telefónicas","l10n_mx.mx_coa","Depreciación acumulada de equipo de comunicaciones telefónicas","171.13"
"account_subgroup_depreciación_acumulada_de_equipo_de_comunicación_satelital","l10n_mx.mx_coa","Depreciación acumulada de equipo de comunicación satelital","171.14"
"account_subgroup_depreciación_acumulada_de_equipo_de_adaptaciones_para_personas_con_capacidades_diferentes","l10n_mx.mx_coa","Depreciación acumulada de equipo de adaptaciones para personas con capacidades diferentes","171.15"
"account_subgroup_depreciación_acumulada_de_maquinaria_y_equipo_de_generación_de_energía_de_fuentes_renovables_o_de_sistemas_de_cogeneración_de_electricidad_eficiente","l10n_mx.mx_coa","Depreciación acumulada de maquinaria y equipo de generación de energía de fuentes renovables o de sistemas de cogeneración de electricidad eficiente","171.16"
"account_subgroup_depreciación_acumulada_de_adaptaciones_y_mejoras","l10n_mx.mx_coa","Depreciación acumulada de adaptaciones y mejoras","171.17"
"account_subgroup_depreciación_acumulada_de_otra_maquinaria_y_equipo","l10n_mx.mx_coa","Depreciación acumulada de otra maquinaria y equipo","171.18"
"account_subgroup_pérdida_por_deterioro_acumulado_de_activos_fijos","l10n_mx.mx_coa","Pérdida por deterioro acumulado de activos fijos","172"
"account_subgroup_pérdida_por_deterioro_acumulado_de_edificios","l10n_mx.mx_coa","Pérdida por deterioro acumulado de edificios","172.01"
"account_subgroup_pérdida_por_deterioro_acumulado_de_maquinaria_y_equipo","l10n_mx.mx_coa","Pérdida por deterioro acumulado de maquinaria y equipo","172.02"
"account_subgroup_pérdida_por_deterioro_acumulado_de_automóviles,_autobuses,_camiones_de_carga,_tractocamiones,_montacargas_y_remolques","l10n_mx.mx_coa","Pérdida por deterioro acumulado de automóviles, autobuses, camiones de carga, tractocamiones, montacargas y remolques","172.03"
"account_subgroup_pérdida_por_deterioro_acumulado_de_mobiliario_y_equipo_de_oficina","l10n_mx.mx_coa","Pérdida por deterioro acumulado de mobiliario y equipo de oficina","172.04"
"account_subgroup_pérdida_por_deterioro_acumulado_de_equipo_de_cómputo","l10n_mx.mx_coa","Pérdida por deterioro acumulado de equipo de cómputo","172.05"
"account_subgroup_pérdida_por_deterioro_acumulado_de_equipo_de_comunicación","l10n_mx.mx_coa","Pérdida por deterioro acumulado de equipo de comunicación","172.06"
"account_subgroup_pérdida_por_deterioro_acumulado_de_activos_biológicos,_vegetales_y_semovientes","l10n_mx.mx_coa","Pérdida por deterioro acumulado de activos biológicos, vegetales y semovientes","172.07"
"account_subgroup_pérdida_por_deterioro_acumulado_de_otros_activos_fijos","l10n_mx.mx_coa","Pérdida por deterioro acumulado de otros activos fijos","172.08"
"account_subgroup_pérdida_por_deterioro_acumulado_de_ferrocarriles","l10n_mx.mx_coa","Pérdida por deterioro acumulado de ferrocarriles","172.09"
"account_subgroup_pérdida_por_deterioro_acumulado_de_embarcaciones","l10n_mx.mx_coa","Pérdida por deterioro acumulado de embarcaciones","172.10"
"account_subgroup_pérdida_por_deterioro_acumulado_de_aviones","l10n_mx.mx_coa","Pérdida por deterioro acumulado de aviones","172.11"
"account_subgroup_pérdida_por_deterioro_acumulado_de_troqueles,_moldes,_matrices_y_herramental","l10n_mx.mx_coa","Pérdida por deterioro acumulado de troqueles, moldes, matrices y herramental","172.12"
"account_subgroup_pérdida_por_deterioro_acumulado_de_equipo_de_comunicaciones_telefónicas","l10n_mx.mx_coa","Pérdida por deterioro acumulado de equipo de comunicaciones telefónicas","172.13"
"account_subgroup_pérdida_por_deterioro_acumulado_de_equipo_de_comunicación_satelital","l10n_mx.mx_coa","Pérdida por deterioro acumulado de equipo de comunicación satelital","172.14"
"account_subgroup_pérdida_por_deterioro_acumulado_de_equipo_de_adaptaciones_para_personas_con_capacidades_diferentes","l10n_mx.mx_coa","Pérdida por deterioro acumulado de equipo de adaptaciones para personas con capacidades diferentes","172.15"
"account_subgroup_pérdida_por_deterioro_acumulado_de_maquinaria_y_equipo_de_generación_de_energía_de_fuentes_renovables_o_de_sistemas_de_cogeneración_de_electricidad_eficiente","l10n_mx.mx_coa","Pérdida por deterioro acumulado de maquinaria y equipo de generación de energía de fuentes renovables o de sistemas de cogeneración de electricidad eficiente","172.16"
"account_subgroup_pérdida_por_deterioro_acumulado_de_adaptaciones_y_mejoras","l10n_mx.mx_coa","Pérdida por deterioro acumulado de adaptaciones y mejoras","172.17"
"account_subgroup_pérdida_por_deterioro_acumulado_de_otra_maquinaria_y_equipo","l10n_mx.mx_coa","Pérdida por deterioro acumulado de otra maquinaria y equipo","172.18"
"account_subgroup_gastos_diferidos","l10n_mx.mx_coa","Gastos diferidos","173"
"account_subgroup_gastos_diferidos_1","l10n_mx.mx_coa","Gastos diferidos","173.01"
"account_subgroup_gastos_pre_operativos","l10n_mx.mx_coa","Gastos pre operativos","174"
"account_subgroup_gastos_pre_operativos_1","l10n_mx.mx_coa","Gastos pre operativos","174.01"
"account_subgroup_regalías,_asistencia_técnica_y_otros_gastos_diferidos","l10n_mx.mx_coa","Regalías, asistencia técnica y otros gastos diferidos","175"
"account_subgroup_regalías,_asistencia_técnica_y_otros_gastos_diferidos_1","l10n_mx.mx_coa","Regalías, asistencia técnica y otros gastos diferidos","175.01"
"account_subgroup_activos_intangibles","l10n_mx.mx_coa","Activos intangibles","176"
"account_subgroup_activos_intangibles_1","l10n_mx.mx_coa","Activos intangibles","176.01"
"account_subgroup_gastos_de_organización","l10n_mx.mx_coa","Gastos de organización","177"
"account_subgroup_gastos_de_organización_1","l10n_mx.mx_coa","Gastos de organización","177.01"
"account_subgroup_investigación_y_desarrollo_de_mercado","l10n_mx.mx_coa","Investigación y desarrollo de mercado","178"
"account_subgroup_investigación_y_desarrollo_de_mercado_1","l10n_mx.mx_coa","Investigación y desarrollo de mercado","178.01"
"account_subgroup_marcas_y_patentes","l10n_mx.mx_coa","Marcas y patentes","179"
"account_subgroup_marcas_y_patentes_1","l10n_mx.mx_coa","Marcas y patentes","179.01"
"account_subgroup_crédito_mercantil","l10n_mx.mx_coa","Crédito mercantil","180"
"account_subgroup_crédito_mercantil_1","l10n_mx.mx_coa","Crédito mercantil","180.01"
"account_subgroup_gastos_de_instalación","l10n_mx.mx_coa","Gastos de instalación","181"
"account_subgroup_gastos_de_instalación_1","l10n_mx.mx_coa","Gastos de instalación","181.01"
"account_subgroup_otros_activos_diferidos","l10n_mx.mx_coa","Otros activos diferidos","182"
"account_subgroup_otros_activos_diferidos_1","l10n_mx.mx_coa","Otros activos diferidos","182.01"
"account_subgroup_amortización_acumulada_de_activos_diferidos","l10n_mx.mx_coa","Amortización acumulada de activos diferidos","183"
"account_subgroup_amortización_acumulada_de_gastos_diferidos","l10n_mx.mx_coa","Amortización acumulada de gastos diferidos","183.01"
"account_subgroup_amortización_acumulada_de_gastos_pre_operativos","l10n_mx.mx_coa","Amortización acumulada de gastos pre operativos","183.02"
"account_subgroup_amortización_acumulada_de_regalías,_asistencia_técnica_y_otros_gastos_diferidos","l10n_mx.mx_coa","Amortización acumulada de regalías, asistencia técnica y otros gastos diferidos","183.03"
"account_subgroup_amortización_acumulada_de_activos_intangibles","l10n_mx.mx_coa","Amortización acumulada de activos intangibles","183.04"
"account_subgroup_amortización_acumulada_de_gastos_de_organización","l10n_mx.mx_coa","Amortización acumulada de gastos de organización","183.05"
"account_subgroup_amortización_acumulada_de_investigación_y_desarrollo_de_mercado","l10n_mx.mx_coa","Amortización acumulada de investigación y desarrollo de mercado","183.06"
"account_subgroup_amortización_acumulada_de_marcas_y_patentes","l10n_mx.mx_coa","Amortización acumulada de marcas y patentes","183.07"
"account_subgroup_amortización_acumulada_de_crédito_mercantil","l10n_mx.mx_coa","Amortización acumulada de crédito mercantil","183.08"
"account_subgroup_amortización_acumulada_de_gastos_de_instalación","l10n_mx.mx_coa","Amortización acumulada de gastos de instalación","183.09"
"account_subgroup_amortización_acumulada_de_otros_activos_diferidos","l10n_mx.mx_coa","Amortización acumulada de otros activos diferidos","183.10"
"account_subgroup_depósitos_en_garantía","l10n_mx.mx_coa","Depósitos en garantía","184"
"account_subgroup_depósitos_de_fianzas","l10n_mx.mx_coa","Depósitos de fianzas","184.01"
"account_subgroup_depósitos_de_arrendamiento_de_bienes_inmuebles","l10n_mx.mx_coa","Depósitos de arrendamiento de bienes inmuebles","184.02"
"account_subgroup_otros_depósitos_en_garantía","l10n_mx.mx_coa","Otros depósitos en garantía","184.03"
"account_subgroup_impuestos_diferidos","l10n_mx.mx_coa","Impuestos diferidos","185"
"account_subgroup_impuestos_diferidos_isr","l10n_mx.mx_coa","Impuestos diferidos ISR","185.01"
"account_subgroup_cuentas_y_documentos_por_cobrar_a_largo_plazo","l10n_mx.mx_coa","Cuentas y documentos por cobrar a largo plazo","186"
"account_subgroup_cuentas_y_documentos_por_cobrar_a_largo_plazo_nacional","l10n_mx.mx_coa","Cuentas y documentos por cobrar a largo plazo nacional","186.01"
"account_subgroup_cuentas_y_documentos_por_cobrar_a_largo_plazo_extranjero","l10n_mx.mx_coa","Cuentas y documentos por cobrar a largo plazo extranjero","186.02"
"account_subgroup_cuentas_y_documentos_por_cobrar_a_largo_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Cuentas y documentos por cobrar a largo plazo nacional parte relacionada","186.03"
"account_subgroup_cuentas_y_documentos_por_cobrar_a_largo_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Cuentas y documentos por cobrar a largo plazo extranjero parte relacionada","186.04"
"account_subgroup_intereses_por_cobrar_a_largo_plazo_nacional","l10n_mx.mx_coa","Intereses por cobrar a largo plazo nacional","186.05"
"account_subgroup_intereses_por_cobrar_a_largo_plazo_extranjero","l10n_mx.mx_coa","Intereses por cobrar a largo plazo extranjero","186.06"
"account_subgroup_intereses_por_cobrar_a_largo_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Intereses por cobrar a largo plazo nacional parte relacionada","186.07"
"account_subgroup_intereses_por_cobrar_a_largo_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Intereses por cobrar a largo plazo extranjero parte relacionada","186.08"
"account_subgroup_otras_cuentas_y_documentos_por_cobrar_a_largo_plazo","l10n_mx.mx_coa","Otras cuentas y documentos por cobrar a largo plazo","186.09"
"account_subgroup_otras_cuentas_y_documentos_por_cobrar_a_largo_plazo_parte_relacionada","l10n_mx.mx_coa","Otras cuentas y documentos por cobrar a largo plazo parte relacionada","186.10"
"account_subgroup_participación_de_los_trabajadores_en_las_utilidades_diferidas","l10n_mx.mx_coa","Participación de los trabajadores en las utilidades diferidas","187"
"account_subgroup_participación_de_los_trabajadores_en_las_utilidades_diferidas_1","l10n_mx.mx_coa","Participación de los trabajadores en las utilidades diferidas","187.01"
"account_subgroup_inversiones_permanentes_en_acciones","l10n_mx.mx_coa","Inversiones permanentes en acciones","188"
"account_subgroup_inversiones_a_largo_plazo_en_subsidiarias","l10n_mx.mx_coa","Inversiones a largo plazo en subsidiarias","188.01"
"account_subgroup_inversiones_a_largo_plazo_en_asociadas","l10n_mx.mx_coa","Inversiones a largo plazo en asociadas","188.02"
"account_subgroup_otras_inversiones_permanentes_en_acciones","l10n_mx.mx_coa","Otras inversiones permanentes en acciones","188.03"
"account_subgroup_estimación_por_deterioro_de_inversiones_permanentes_en_acciones","l10n_mx.mx_coa","Estimación por deterioro de inversiones permanentes en acciones","189"
"account_subgroup_estimación_por_deterioro_de_inversiones_permanentes_en_acciones_1","l10n_mx.mx_coa","Estimación por deterioro de inversiones permanentes en acciones","189.01"
"account_subgroup_otros_instrumentos_financieros_2","l10n_mx.mx_coa","Otros instrumentos financieros","190"
"account_subgroup_otros_instrumentos_financieros_3","l10n_mx.mx_coa","Otros instrumentos financieros","190.01"
"account_subgroup_otros_activos_a_largo_plazo","l10n_mx.mx_coa","Otros activos a largo plazo","191"
"account_subgroup_otros_activos_a_largo_plazo_1","l10n_mx.mx_coa","Otros activos a largo plazo","191.01"
"account_group_pasivos","l10n_mx.mx_coa","Pasivos","2"
"account_subgroup_pasivo_a_corto_plazo","l10n_mx.mx_coa","Pasivo a corto plazo","200.01"
"account_subgroup_proveedores","l10n_mx.mx_coa","Proveedores","201"
"account_subgroup_proveedores_nacionales","l10n_mx.mx_coa","Proveedores nacionales","201.01"
"account_subgroup_proveedores_extranjeros","l10n_mx.mx_coa","Proveedores extranjeros","201.02"
"account_subgroup_proveedores_nacionales_parte_relacionada","l10n_mx.mx_coa","Proveedores nacionales parte relacionada","201.03"
"account_subgroup_proveedores_extranjeros_parte_relacionada","l10n_mx.mx_coa","Proveedores extranjeros parte relacionada","201.04"
"account_subgroup_cuentas_por_pagar_a_corto_plazo","l10n_mx.mx_coa","Cuentas por pagar a corto plazo","202"
"account_subgroup_documentos_por_pagar_bancario_y_financiero_nacional","l10n_mx.mx_coa","Documentos por pagar bancario y financiero nacional","202.01"
"account_subgroup_documentos_por_pagar_bancario_y_financiero_extranjero","l10n_mx.mx_coa","Documentos por pagar bancario y financiero extranjero","202.02"
"account_subgroup_documentos_y_cuentas_por_pagar_a_corto_plazo_nacional","l10n_mx.mx_coa","Documentos y cuentas por pagar a corto plazo nacional","202.03"
"account_subgroup_documentos_y_cuentas_por_pagar_a_corto_plazo_extranjero","l10n_mx.mx_coa","Documentos y cuentas por pagar a corto plazo extranjero","202.04"
"account_subgroup_documentos_y_cuentas_por_pagar_a_corto_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Documentos y cuentas por pagar a corto plazo nacional parte relacionada","202.05"
"account_subgroup_documentos_y_cuentas_por_pagar_a_corto_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Documentos y cuentas por pagar a corto plazo extranjero parte relacionada","202.06"
"account_subgroup_intereses_por_pagar_a_corto_plazo_nacional","l10n_mx.mx_coa","Intereses por pagar a corto plazo nacional","202.07"
"account_subgroup_intereses_por_pagar_a_corto_plazo_extranjero","l10n_mx.mx_coa","Intereses por pagar a corto plazo extranjero","202.08"
"account_subgroup_intereses_por_pagar_a_corto_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Intereses por pagar a corto plazo nacional parte relacionada","202.09"
"account_subgroup_intereses_por_pagar_a_corto_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Intereses por pagar a corto plazo extranjero parte relacionada","202.10"
"account_subgroup_dividendo_por_pagar_nacional","l10n_mx.mx_coa","Dividendo por pagar nacional","202.11"
"account_subgroup_dividendo_por_pagar_extranjero","l10n_mx.mx_coa","Dividendo por pagar extranjero","202.12"
"account_subgroup_cobros_anticipados_a_corto_plazo","l10n_mx.mx_coa","Cobros anticipados a corto plazo","203"
"account_subgroup_rentas_cobradas_por_anticipado_a_corto_plazo_nacional","l10n_mx.mx_coa","Rentas cobradas por anticipado a corto plazo nacional","203.01"
"account_subgroup_rentas_cobradas_por_anticipado_a_corto_plazo_extranjero","l10n_mx.mx_coa","Rentas cobradas por anticipado a corto plazo extranjero","203.02"
"account_subgroup_rentas_cobradas_por_anticipado_a_corto_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Rentas cobradas por anticipado a corto plazo nacional parte relacionada","203.03"
"account_subgroup_rentas_cobradas_por_anticipado_a_corto_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Rentas cobradas por anticipado a corto plazo extranjero parte relacionada","203.04"
"account_subgroup_intereses_cobrados_por_anticipado_a_corto_plazo_nacional","l10n_mx.mx_coa","Intereses cobrados por anticipado a corto plazo nacional","203.05"
"account_subgroup_intereses_cobrados_por_anticipado_a_corto_plazo_extranjero","l10n_mx.mx_coa","Intereses cobrados por anticipado a corto plazo extranjero","203.06"
"account_subgroup_intereses_cobrados_por_anticipado_a_corto_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Intereses cobrados por anticipado a corto plazo nacional parte relacionada","203.07"
"account_subgroup_intereses_cobrados_por_anticipado_a_corto_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Intereses cobrados por anticipado a corto plazo extranjero parte relacionada","203.08"
"account_subgroup_factoraje_financiero_cobrados_por_anticipado_a_corto_plazo_nacional","l10n_mx.mx_coa","Factoraje financiero cobrados por anticipado a corto plazo nacional","203.09"
"account_subgroup_factoraje_financiero_cobrados_por_anticipado_a_corto_plazo_extranjero","l10n_mx.mx_coa","Factoraje financiero cobrados por anticipado a corto plazo extranjero","203.10"
"account_subgroup_factoraje_financiero_cobrados_por_anticipado_a_corto_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Factoraje financiero cobrados por anticipado a corto plazo nacional parte relacionada","203.11"
"account_subgroup_factoraje_financiero_cobrados_por_anticipado_a_corto_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Factoraje financiero cobrados por anticipado a corto plazo extranjero parte relacionada","203.12"
"account_subgroup_arrendamiento_financiero_cobrados_por_anticipado_a_corto_plazo_nacional","l10n_mx.mx_coa","Arrendamiento financiero cobrados por anticipado a corto plazo nacional","203.13"
"account_subgroup_arrendamiento_financiero_cobrados_por_anticipado_a_corto_plazo_extranjero","l10n_mx.mx_coa","Arrendamiento financiero cobrados por anticipado a corto plazo extranjero","203.14"
"account_subgroup_arrendamiento_financiero_cobrados_por_anticipado_a_corto_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Arrendamiento financiero cobrados por anticipado a corto plazo nacional parte relacionada","203.15"
"account_subgroup_arrendamiento_financiero_cobrados_por_anticipado_a_corto_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Arrendamiento financiero cobrados por anticipado a corto plazo extranjero parte relacionada","203.16"
"account_subgroup_derechos_fiduciarios_1","l10n_mx.mx_coa","Derechos fiduciarios","203.17"
"account_subgroup_otros_cobros_anticipados","l10n_mx.mx_coa","Otros cobros anticipados","203.18"
"account_subgroup_instrumentos_financieros_a_corto_plazo","l10n_mx.mx_coa","Instrumentos financieros a corto plazo","204"
"account_subgroup_instrumentos_financieros_a_corto_plazo_1","l10n_mx.mx_coa","Instrumentos financieros a corto plazo","204.01"
"account_subgroup_acreedores_diversos_a_corto_plazo","l10n_mx.mx_coa","Acreedores diversos a corto plazo","205"
"account_subgroup_socios,_accionistas_o_representante_legal","l10n_mx.mx_coa","Socios, accionistas o representante legal","205.01"
"account_subgroup_acreedores_diversos_a_corto_plazo_nacional","l10n_mx.mx_coa","Acreedores diversos a corto plazo nacional","205.02"
"account_subgroup_acreedores_diversos_a_corto_plazo_extranjero","l10n_mx.mx_coa","Acreedores diversos a corto plazo extranjero","205.03"
"account_subgroup_acreedores_diversos_a_corto_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Acreedores diversos a corto plazo nacional parte relacionada","205.04"
"account_subgroup_acreedores_diversos_a_corto_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Acreedores diversos a corto plazo extranjero parte relacionada","205.05"
"account_subgroup_otros_acreedores_diversos_a_corto_plazo","l10n_mx.mx_coa","Otros acreedores diversos a corto plazo","205.06"
"account_subgroup_anticipo_de_cliente","l10n_mx.mx_coa","Anticipo de cliente","206"
"account_subgroup_anticipo_de_cliente_nacional","l10n_mx.mx_coa","Anticipo de cliente nacional","206.01"
"account_subgroup_anticipo_de_cliente_extranjero","l10n_mx.mx_coa","Anticipo de cliente extranjero","206.02"
"account_subgroup_anticipo_de_cliente_nacional_parte_relacionada","l10n_mx.mx_coa","Anticipo de cliente nacional parte relacionada","206.03"
"account_subgroup_anticipo_de_cliente_extranjero_parte_relacionada","l10n_mx.mx_coa","Anticipo de cliente extranjero parte relacionada","206.04"
"account_subgroup_otros_anticipos_de_clientes","l10n_mx.mx_coa","Otros anticipos de clientes","206.05"
"account_subgroup_impuestos_trasladados","l10n_mx.mx_coa","Impuestos trasladados","207"
"account_subgroup_iva_trasladado","l10n_mx.mx_coa","IVA trasladado","207.01"
"account_subgroup_ieps_trasladado","l10n_mx.mx_coa","IEPS trasladado","207.02"
"account_subgroup_impuestos_trasladados_cobrados","l10n_mx.mx_coa","Impuestos trasladados cobrados","208"
"account_subgroup_iva_trasladado_cobrado","l10n_mx.mx_coa","IVA trasladado cobrado","208.01"
"account_subgroup_ieps_trasladado_cobrado","l10n_mx.mx_coa","IEPS trasladado cobrado","208.02"
"account_subgroup_impuestos_trasladados_no_cobrados","l10n_mx.mx_coa","Impuestos trasladados no cobrados","209"
"account_subgroup_iva_trasladado_no_cobrado","l10n_mx.mx_coa","IVA trasladado no cobrado","209.01"
"account_subgroup_ieps_trasladado_no_cobrado","l10n_mx.mx_coa","IEPS trasladado no cobrado","209.02"
"account_subgroup_provisión_de_sueldos_y_salarios_por_pagar","l10n_mx.mx_coa","Provisión de sueldos y salarios por pagar","210"
"account_subgroup_provisión_de_sueldos_y_salarios_por_pagar_1","l10n_mx.mx_coa","Provisión de sueldos y salarios por pagar","210.01"
"account_subgroup_provisión_de_vacaciones_por_pagar","l10n_mx.mx_coa","Provisión de vacaciones por pagar","210.02"
"account_subgroup_provisión_de_aguinaldo_por_pagar","l10n_mx.mx_coa","Provisión de aguinaldo por pagar","210.03"
"account_subgroup_provisión_de_fondo_de_ahorro_por_pagar","l10n_mx.mx_coa","Provisión de fondo de ahorro por pagar","210.04"
"account_subgroup_provisión_de_asimilados_a_salarios_por_pagar","l10n_mx.mx_coa","Provisión de asimilados a salarios por pagar","210.05"
"account_subgroup_provisión_de_anticipos_o_remanentes_por_distribuir","l10n_mx.mx_coa","Provisión de anticipos o remanentes por distribuir","210.06"
"account_subgroup_provisión_de_otros_sueldos_y_salarios_por_pagar","l10n_mx.mx_coa","Provisión de otros sueldos y salarios por pagar","210.07"
"account_subgroup_provisión_de_contribuciones_de_seguridad_social_por_pagar","l10n_mx.mx_coa","Provisión de contribuciones de seguridad social por pagar","211"
"account_subgroup_provisión_de_imss_patronal_por_pagar","l10n_mx.mx_coa","Provisión de IMSS patronal por pagar","211.01"
"account_subgroup_provisión_de_sar_por_pagar","l10n_mx.mx_coa","Provisión de SAR por pagar","211.02"
"account_subgroup_provisión_de_infonavit_por_pagar","l10n_mx.mx_coa","Provisión de infonavit por pagar","211.03"
"account_subgroup_provisión_de_impuesto_estatal_sobre_nómina_por_pagar","l10n_mx.mx_coa","Provisión de impuesto estatal sobre nómina por pagar","212"
"account_subgroup_provisión_de_impuesto_estatal_sobre_nómina_por_pagar_1","l10n_mx.mx_coa","Provisión de impuesto estatal sobre nómina por pagar","212.01"
"account_subgroup_impuestos_y_derechos_por_pagar","l10n_mx.mx_coa","Impuestos y derechos por pagar","213"
"account_subgroup_iva_por_pagar","l10n_mx.mx_coa","IVA por pagar","213.01"
"account_subgroup_ieps_por_pagar","l10n_mx.mx_coa","IEPS por pagar","213.02"
"account_subgroup_isr_por_pagar","l10n_mx.mx_coa","ISR por pagar","213.03"
"account_subgroup_impuesto_estatal_sobre_nómina_por_pagar","l10n_mx.mx_coa","Impuesto estatal sobre nómina por pagar","213.04"
"account_subgroup_impuesto_estatal_y_municipal_por_pagar","l10n_mx.mx_coa","Impuesto estatal y municipal por pagar","213.05"
"account_subgroup_derechos_por_pagar","l10n_mx.mx_coa","Derechos por pagar","213.06"
"account_subgroup_otros_impuestos_por_pagar","l10n_mx.mx_coa","Otros impuestos por pagar","213.07"
"account_subgroup_dividendos_por_pagar","l10n_mx.mx_coa","Dividendos por pagar","214"
"account_subgroup_dividendos_por_pagar_1","l10n_mx.mx_coa","Dividendos por pagar","214.01"
"account_subgroup_ptu_por_pagar","l10n_mx.mx_coa","PTU por pagar","215"
"account_subgroup_ptu_por_pagar_1","l10n_mx.mx_coa","PTU por pagar","215.01"
"account_subgroup_ptu_por_pagar_de_ejercicios_anteriores","l10n_mx.mx_coa","PTU por pagar de ejercicios anteriores","215.02"
"account_subgroup_provisión_de_ptu_por_pagar","l10n_mx.mx_coa","Provisión de PTU por pagar","215.03"
"account_subgroup_impuestos_retenidos","l10n_mx.mx_coa","Impuestos retenidos","216"
"account_subgroup_impuestos_retenidos_de_isr_por_sueldos_y_salarios","l10n_mx.mx_coa","Impuestos retenidos de ISR por sueldos y salarios","216.01"
"account_subgroup_impuestos_retenidos_de_isr_por_asimilados_a_salarios","l10n_mx.mx_coa","Impuestos retenidos de ISR por asimilados a salarios","216.02"
"account_subgroup_impuestos_retenidos_de_isr_por_arrendamiento","l10n_mx.mx_coa","Impuestos retenidos de ISR por arrendamiento","216.03"
"account_subgroup_impuestos_retenidos_de_isr_por_servicios_profesionales","l10n_mx.mx_coa","Impuestos retenidos de ISR por servicios profesionales","216.04"
"account_subgroup_impuestos_retenidos_de_isr_por_dividendos","l10n_mx.mx_coa","Impuestos retenidos de ISR por dividendos","216.05"
"account_subgroup_impuestos_retenidos_de_isr_por_intereses","l10n_mx.mx_coa","Impuestos retenidos de ISR por intereses","216.06"
"account_subgroup_impuestos_retenidos_de_isr_por_pagos_al_extranjero","l10n_mx.mx_coa","Impuestos retenidos de ISR por pagos al extranjero","216.07"
"account_subgroup_impuestos_retenidos_de_isr_por_venta_de_acciones","l10n_mx.mx_coa","Impuestos retenidos de ISR por venta de acciones","216.08"
"account_subgroup_impuestos_retenidos_de_isr_por_venta_de_partes_sociales","l10n_mx.mx_coa","Impuestos retenidos de ISR por venta de partes sociales","216.09"
"account_subgroup_impuestos_retenidos_de_iva","l10n_mx.mx_coa","Impuestos retenidos de IVA","216.10"
"account_subgroup_retenciones_de_imss_a_los_trabajadores","l10n_mx.mx_coa","Retenciones de IMSS a los trabajadores","216.11"
"account_subgroup_otras_impuestos_retenidos","l10n_mx.mx_coa","Otras impuestos retenidos","216.12"
"account_subgroup_pagos_realizados_por_cuenta_de_terceros","l10n_mx.mx_coa","Pagos realizados por cuenta de terceros","217"
"account_subgroup_pagos_realizados_por_cuenta_de_terceros_1","l10n_mx.mx_coa","Pagos realizados por cuenta de terceros","217.01"
"account_subgroup_otros_pasivos_a_corto_plazo","l10n_mx.mx_coa","Otros pasivos a corto plazo","218"
"account_subgroup_otros_pasivos_a_corto_plazo_1","l10n_mx.mx_coa","Otros pasivos a corto plazo","218.01"
"account_subgroup_pasivo_a_largo_plazo","l10n_mx.mx_coa","Pasivo a largo plazo","200.02"
"account_subgroup_acreedores_diversos_a_largo_plazo","l10n_mx.mx_coa","Acreedores diversos a largo plazo","251"
"account_subgroup_socios,_accionistas_o_representante_legal_1","l10n_mx.mx_coa","Socios, accionistas o representante legal","251.01"
"account_subgroup_acreedores_diversos_a_largo_plazo_nacional","l10n_mx.mx_coa","Acreedores diversos a largo plazo nacional","251.02"
"account_subgroup_acreedores_diversos_a_largo_plazo_extranjero","l10n_mx.mx_coa","Acreedores diversos a largo plazo extranjero","251.03"
"account_subgroup_acreedores_diversos_a_largo_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Acreedores diversos a largo plazo nacional parte relacionada","251.04"
"account_subgroup_acreedores_diversos_a_largo_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Acreedores diversos a largo plazo extranjero parte relacionada","251.05"
"account_subgroup_otros_acreedores_diversos_a_largo_plazo","l10n_mx.mx_coa","Otros acreedores diversos a largo plazo","251.06"
"account_subgroup_cuentas_por_pagar_a_largo_plazo","l10n_mx.mx_coa","Cuentas por pagar a largo plazo","252"
"account_subgroup_documentos_bancarios_y_financieros_por_pagar_a_largo_plazo_nacional","l10n_mx.mx_coa","Documentos bancarios y financieros por pagar a largo plazo nacional","252.01"
"account_subgroup_documentos_bancarios_y_financieros_por_pagar_a_largo_plazo_extranjero","l10n_mx.mx_coa","Documentos bancarios y financieros por pagar a largo plazo extranjero","252.02"
"account_subgroup_documentos_y_cuentas_por_pagar_a_largo_plazo_nacional","l10n_mx.mx_coa","Documentos y cuentas por pagar a largo plazo nacional","252.03"
"account_subgroup_documentos_y_cuentas_por_pagar_a_largo_plazo_extranjero","l10n_mx.mx_coa","Documentos y cuentas por pagar a largo plazo extranjero","252.04"
"account_subgroup_documentos_y_cuentas_por_pagar_a_largo_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Documentos y cuentas por pagar a largo plazo nacional parte relacionada","252.05"
"account_subgroup_documentos_y_cuentas_por_pagar_a_largo_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Documentos y cuentas por pagar a largo plazo extranjero parte relacionada","252.06"
"account_subgroup_hipotecas_por_pagar_a_largo_plazo_nacional","l10n_mx.mx_coa","Hipotecas por pagar a largo plazo nacional","252.07"
"account_subgroup_hipotecas_por_pagar_a_largo_plazo_extranjero","l10n_mx.mx_coa","Hipotecas por pagar a largo plazo extranjero","252.08"
"account_subgroup_hipotecas_por_pagar_a_largo_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Hipotecas por pagar a largo plazo nacional parte relacionada","252.09"
"account_subgroup_hipotecas_por_pagar_a_largo_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Hipotecas por pagar a largo plazo extranjero parte relacionada","252.10"
"account_subgroup_intereses_por_pagar_a_largo_plazo_nacional","l10n_mx.mx_coa","Intereses por pagar a largo plazo nacional","252.11"
"account_subgroup_intereses_por_pagar_a_largo_plazo_extranjero","l10n_mx.mx_coa","Intereses por pagar a largo plazo extranjero","252.12"
"account_subgroup_intereses_por_pagar_a_largo_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Intereses por pagar a largo plazo nacional parte relacionada","252.13"
"account_subgroup_intereses_por_pagar_a_largo_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Intereses por pagar a largo plazo extranjero parte relacionada","252.14"
"account_subgroup_dividendos_por_pagar_nacionales","l10n_mx.mx_coa","Dividendos por pagar nacionales","252.15"
"account_subgroup_dividendos_por_pagar_extranjeros","l10n_mx.mx_coa","Dividendos por pagar extranjeros","252.16"
"account_subgroup_otras_cuentas_y_documentos_por_pagar_a_largo_plazo","l10n_mx.mx_coa","Otras cuentas y documentos por pagar a largo plazo","252.17"
"account_subgroup_cobros_anticipados_a_largo_plazo","l10n_mx.mx_coa","Cobros anticipados a largo plazo","253"
"account_subgroup_rentas_cobradas_por_anticipado_a_largo_plazo_nacional","l10n_mx.mx_coa","Rentas cobradas por anticipado a largo plazo nacional","253.01"
"account_subgroup_rentas_cobradas_por_anticipado_a_largo_plazo_extranjero","l10n_mx.mx_coa","Rentas cobradas por anticipado a largo plazo extranjero","253.02"
"account_subgroup_rentas_cobradas_por_anticipado_a_largo_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Rentas cobradas por anticipado a largo plazo nacional parte relacionada","253.03"
"account_subgroup_rentas_cobradas_por_anticipado_a_largo_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Rentas cobradas por anticipado a largo plazo extranjero parte relacionada","253.04"
"account_subgroup_intereses_cobrados_por_anticipado_a_largo_plazo_nacional","l10n_mx.mx_coa","Intereses cobrados por anticipado a largo plazo nacional","253.05"
"account_subgroup_intereses_cobrados_por_anticipado_a_largo_plazo_extranjero","l10n_mx.mx_coa","Intereses cobrados por anticipado a largo plazo extranjero","253.06"
"account_subgroup_intereses_cobrados_por_anticipado_a_largo_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Intereses cobrados por anticipado a largo plazo nacional parte relacionada","253.07"
"account_subgroup_intereses_cobrados_por_anticipado_a_largo_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Intereses cobrados por anticipado a largo plazo extranjero parte relacionada","253.08"
"account_subgroup_factoraje_financiero_cobrados_por_anticipado_a_largo_plazo_nacional","l10n_mx.mx_coa","Factoraje financiero cobrados por anticipado a largo plazo nacional","253.09"
"account_subgroup_factoraje_financiero_cobrados_por_anticipado_a_largo_plazo_extranjero","l10n_mx.mx_coa","Factoraje financiero cobrados por anticipado a largo plazo extranjero","253.10"
"account_subgroup_factoraje_financiero_cobrados_por_anticipado_a_largo_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Factoraje financiero cobrados por anticipado a largo plazo nacional parte relacionada","253.11"
"account_subgroup_factoraje_financiero_cobrados_por_anticipado_a_largo_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Factoraje financiero cobrados por anticipado a largo plazo extranjero parte relacionada","253.12"
"account_subgroup_arrendamiento_financiero_cobrados_por_anticipado_a_largo_plazo_nacional","l10n_mx.mx_coa","Arrendamiento financiero cobrados por anticipado a largo plazo nacional","253.13"
"account_subgroup_arrendamiento_financiero_cobrados_por_anticipado_a_largo_plazo_extranjero","l10n_mx.mx_coa","Arrendamiento financiero cobrados por anticipado a largo plazo extranjero","253.14"
"account_subgroup_arrendamiento_financiero_cobrados_por_anticipado_a_largo_plazo_nacional_parte_relacionada","l10n_mx.mx_coa","Arrendamiento financiero cobrados por anticipado a largo plazo nacional parte relacionada","253.15"
"account_subgroup_arrendamiento_financiero_cobrados_por_anticipado_a_largo_plazo_extranjero_parte_relacionada","l10n_mx.mx_coa","Arrendamiento financiero cobrados por anticipado a largo plazo extranjero parte relacionada","253.16"
"account_subgroup_derechos_fiduciarios_2","l10n_mx.mx_coa","Derechos fiduciarios","253.17"
"account_subgroup_otros_cobros_anticipados_1","l10n_mx.mx_coa","Otros cobros anticipados","253.18"
"account_subgroup_instrumentos_financieros_a_largo_plazo","l10n_mx.mx_coa","Instrumentos financieros a largo plazo","254"
"account_subgroup_instrumentos_financieros_a_largo_plazo_1","l10n_mx.mx_coa","Instrumentos financieros a largo plazo","254.01"
"account_subgroup_pasivos_por_beneficios_a_los_empleados_a_largo_plazo","l10n_mx.mx_coa","Pasivos por beneficios a los empleados a largo plazo","255"
"account_subgroup_pasivos_por_beneficios_a_los_empleados_a_largo_plazo_1","l10n_mx.mx_coa","Pasivos por beneficios a los empleados a largo plazo","255.01"
"account_subgroup_otros_pasivos_a_largo_plazo","l10n_mx.mx_coa","Otros pasivos a largo plazo","256"
"account_subgroup_otros_pasivos_a_largo_plazo_1","l10n_mx.mx_coa","Otros pasivos a largo plazo","256.01"
"account_subgroup_participación_de_los_trabajadores_en_las_utilidades_diferida","l10n_mx.mx_coa","Participación de los trabajadores en las utilidades diferida","257"
"account_subgroup_participación_de_los_trabajadores_en_las_utilidades_diferida_1","l10n_mx.mx_coa","Participación de los trabajadores en las utilidades diferida","257.01"
"account_subgroup_obligaciones_contraídas_de_fideicomisos","l10n_mx.mx_coa","Obligaciones contraídas de fideicomisos","258"
"account_subgroup_obligaciones_contraídas_de_fideicomisos_1","l10n_mx.mx_coa","Obligaciones contraídas de fideicomisos","258.01"
"account_subgroup_impuestos_diferidos_1","l10n_mx.mx_coa","Impuestos diferidos","259"
"account_subgroup_isr_diferido","l10n_mx.mx_coa","ISR diferido","259.01"
"account_subgroup_isr_por_dividendo_diferido","l10n_mx.mx_coa","ISR por dividendo diferido","259.02"
"account_subgroup_otros_impuestos_diferidos","l10n_mx.mx_coa","Otros impuestos diferidos","259.03"
"account_subgroup_pasivos_diferidos","l10n_mx.mx_coa","Pasivos diferidos","260"
"account_subgroup_pasivos_diferidos_1","l10n_mx.mx_coa","Pasivos diferidos","260.01"
"account_group_capital_contable","l10n_mx.mx_coa","Capital Contable","3"
"account_subgroup_capital_social","l10n_mx.mx_coa","Capital social","301"
"account_subgroup_capital_fijo","l10n_mx.mx_coa","Capital fijo","301.01"
"account_subgroup_capital_variable","l10n_mx.mx_coa","Capital variable","301.02"
"account_subgroup_aportaciones_para_futuros_aumentos_de_capital","l10n_mx.mx_coa","Aportaciones para futuros aumentos de capital","301.03"
"account_subgroup_prima_en_suscripción_de_acciones","l10n_mx.mx_coa","Prima en suscripción de acciones","301.04"
"account_subgroup_prima_en_suscripción_de_partes_sociales","l10n_mx.mx_coa","Prima en suscripción de partes sociales","301.05"
"account_subgroup_patrimonio","l10n_mx.mx_coa","Patrimonio","302"
"account_subgroup_patrimonio_1","l10n_mx.mx_coa","Patrimonio","302.01"
"account_subgroup_aportación_patrimonial","l10n_mx.mx_coa","Aportación patrimonial","302.02"
"account_subgroup_déficit_o_remanente_del_ejercicio","l10n_mx.mx_coa","Déficit o remanente del ejercicio","302.03"
"account_subgroup_reserva_legal","l10n_mx.mx_coa","Reserva legal","303"
"account_subgroup_reserva_legal_1","l10n_mx.mx_coa","Reserva legal","303.01"
"account_subgroup_resultado_de_ejercicios_anteriores","l10n_mx.mx_coa","Resultado de ejercicios anteriores","304"
"account_subgroup_utilidad_de_ejercicios_anteriores","l10n_mx.mx_coa","Utilidad de ejercicios anteriores","304.01"
"account_subgroup_pérdida_de_ejercicios_anteriores","l10n_mx.mx_coa","Pérdida de ejercicios anteriores","304.02"
"account_subgroup_resultado_integral_de_ejercicios_anteriores","l10n_mx.mx_coa","Resultado integral de ejercicios anteriores","304.03"
"account_subgroup_déficit_o_remanente_de_ejercicio_anteriores","l10n_mx.mx_coa","Déficit o remanente de ejercicio anteriores","304.04"
"account_subgroup_resultado_del_ejercicio","l10n_mx.mx_coa","Resultado del ejercicio","305"
"account_subgroup_utilidad_del_ejercicio","l10n_mx.mx_coa","Utilidad del ejercicio","305.01"
"account_subgroup_pérdida_del_ejercicio","l10n_mx.mx_coa","Pérdida del ejercicio","305.02"
"account_subgroup_resultado_integral","l10n_mx.mx_coa","Resultado integral","305.03"
"account_subgroup_otras_cuentas_de_capital","l10n_mx.mx_coa","Otras cuentas de capital","306"
"account_subgroup_otras_cuentas_de_capital_1","l10n_mx.mx_coa","Otras cuentas de capital","306.01"
"account_group_ingresos","l10n_mx.mx_coa","Ingresos","4"
"account_subgroup_ingresos","l10n_mx.mx_coa","Ingresos","401"
"account_subgroup_ventas_y/o_servicios_gravados_a_la_tasa_general","l10n_mx.mx_coa","Ventas y/o servicios gravados a la tasa general","401.01"
"account_subgroup_ventas_y/o_servicios_gravados_a_la_tasa_general_de_contado","l10n_mx.mx_coa","Ventas y/o servicios gravados a la tasa general de contado","401.02"
"account_subgroup_ventas_y/o_servicios_gravados_a_la_tasa_general_a_crédito","l10n_mx.mx_coa","Ventas y/o servicios gravados a la tasa general a crédito","401.03"
"account_subgroup_ventas_y/o_servicios_gravados_al_0%","l10n_mx.mx_coa","Ventas y/o servicios gravados al 0%","401.04"
"account_subgroup_ventas_y/o_servicios_gravados_al_0%_de_contado","l10n_mx.mx_coa","Ventas y/o servicios gravados al 0% de contado","401.05"
"account_subgroup_ventas_y/o_servicios_gravados_al_0%_a_crédito","l10n_mx.mx_coa","Ventas y/o servicios gravados al 0% a crédito","401.06"
"account_subgroup_ventas_y/o_servicios_exentos","l10n_mx.mx_coa","Ventas y/o servicios exentos","401.07"
"account_subgroup_ventas_y/o_servicios_exentos_de_contado","l10n_mx.mx_coa","Ventas y/o servicios exentos de contado","401.08"
"account_subgroup_ventas_y/o_servicios_exentos_a_crédito","l10n_mx.mx_coa","Ventas y/o servicios exentos a crédito","401.09"
"account_subgroup_ventas_y/o_servicios_gravados_a_la_tasa_general_nacionales_partes_relacionadas","l10n_mx.mx_coa","Ventas y/o servicios gravados a la tasa general nacionales partes relacionadas","401.10"
"account_subgroup_ventas_y/o_servicios_gravados_a_la_tasa_general_extranjeros_partes_relacionadas","l10n_mx.mx_coa","Ventas y/o servicios gravados a la tasa general extranjeros partes relacionadas","401.11"
"account_subgroup_ventas_y/o_servicios_gravados_al_0%_nacionales_partes_relacionadas","l10n_mx.mx_coa","Ventas y/o servicios gravados al 0% nacionales partes relacionadas","401.12"
"account_subgroup_ventas_y/o_servicios_gravados_al_0%_extranjeros_partes_relacionadas","l10n_mx.mx_coa","Ventas y/o servicios gravados al 0% extranjeros partes relacionadas","401.13"
"account_subgroup_ventas_y/o_servicios_exentos_nacionales_partes_relacionadas","l10n_mx.mx_coa","Ventas y/o servicios exentos nacionales partes relacionadas","401.14"
"account_subgroup_ventas_y/o_servicios_exentos_extranjeros_partes_relacionadas","l10n_mx.mx_coa","Ventas y/o servicios exentos extranjeros partes relacionadas","401.15"
"account_subgroup_ingresos_por_servicios_administrativos","l10n_mx.mx_coa","Ingresos por servicios administrativos","401.16"
"account_subgroup_ingresos_por_servicios_administrativos_nacionales_partes_relacionadas","l10n_mx.mx_coa","Ingresos por servicios administrativos nacionales partes relacionadas","401.17"
"account_subgroup_ingresos_por_servicios_administrativos_extranjeros_partes_relacionadas","l10n_mx.mx_coa","Ingresos por servicios administrativos extranjeros partes relacionadas","401.18"
"account_subgroup_ingresos_por_servicios_profesionales","l10n_mx.mx_coa","Ingresos por servicios profesionales","401.19"
"account_subgroup_ingresos_por_servicios_profesionales_nacionales_partes_relacionadas","l10n_mx.mx_coa","Ingresos por servicios profesionales nacionales partes relacionadas","401.20"
"account_subgroup_ingresos_por_servicios_profesionales_extranjeros_partes_relacionadas","l10n_mx.mx_coa","Ingresos por servicios profesionales extranjeros partes relacionadas","401.21"
"account_subgroup_ingresos_por_arrendamiento","l10n_mx.mx_coa","Ingresos por arrendamiento","401.22"
"account_subgroup_ingresos_por_arrendamiento_nacionales_partes_relacionadas","l10n_mx.mx_coa","Ingresos por arrendamiento nacionales partes relacionadas","401.23"
"account_subgroup_ingresos_por_arrendamiento_extranjeros_partes_relacionadas","l10n_mx.mx_coa","Ingresos por arrendamiento extranjeros partes relacionadas","401.24"
"account_subgroup_ingresos_por_exportación","l10n_mx.mx_coa","Ingresos por exportación","401.25"
"account_subgroup_ingresos_por_comisiones","l10n_mx.mx_coa","Ingresos por comisiones","401.26"
"account_subgroup_ingresos_por_maquila","l10n_mx.mx_coa","Ingresos por maquila","401.27"
"account_subgroup_ingresos_por_coordinados","l10n_mx.mx_coa","Ingresos por coordinados","401.28"
"account_subgroup_ingresos_por_regalías","l10n_mx.mx_coa","Ingresos por regalías","401.29"
"account_subgroup_ingresos_por_asistencia_técnica","l10n_mx.mx_coa","Ingresos por asistencia técnica","401.30"
"account_subgroup_ingresos_por_donativos","l10n_mx.mx_coa","Ingresos por donativos","401.31"
"account_subgroup_ingresos_por_intereses_(actividad_propia)","l10n_mx.mx_coa","Ingresos por intereses (actividad propia)","401.32"
"account_subgroup_ingresos_de_copropiedad","l10n_mx.mx_coa","Ingresos de copropiedad","401.33"
"account_subgroup_ingresos_por_fideicomisos","l10n_mx.mx_coa","Ingresos por fideicomisos","401.34"
"account_subgroup_ingresos_por_factoraje_financiero","l10n_mx.mx_coa","Ingresos por factoraje financiero","401.35"
"account_subgroup_ingresos_por_arrendamiento_financiero","l10n_mx.mx_coa","Ingresos por arrendamiento financiero","401.36"
"account_subgroup_ingresos_de_extranjeros_con_establecimiento_en_el_país","l10n_mx.mx_coa","Ingresos de extranjeros con establecimiento en el país","401.37"
"account_subgroup_otros_ingresos_propios","l10n_mx.mx_coa","Otros ingresos propios","401.38"
"account_subgroup_ventas_y/o_servicios_gravados_realizados_en_zona_fronteriza_norte","l10n_mx.mx_coa","Ventas y/o servicios gravados realizados en zona fronteriza norte","401.39"
"account_subgroup_ventas_y/o_servicios_gravados_realizados_en_zona_fronteriza_norte_de_contado","l10n_mx.mx_coa","Ventas y/o servicios gravados realizados en zona fronteriza norte de contado","401.40"
"account_subgroup_ventas_y/o_servicios_gravados_realizados_en_zona_fronteriza_norte_a_crédito","l10n_mx.mx_coa","Ventas y/o servicios gravados realizados en zona fronteriza norte a crédito","401.41"
"account_subgroup_devoluciones,_descuentos_o_bonificaciones_sobre_ingresos","l10n_mx.mx_coa","Devoluciones, descuentos o bonificaciones sobre ingresos","402"
"account_subgroup_devoluciones,_descuentos_o_bonificaciones_sobre_ventas_y/o_servicios_a_la_tasa_general","l10n_mx.mx_coa","Devoluciones, descuentos o bonificaciones sobre ventas y/o servicios a la tasa general","402.01"
"account_subgroup_devoluciones,_descuentos_o_bonificaciones_sobre_ventas_y/o_servicios_al_0%","l10n_mx.mx_coa","Devoluciones, descuentos o bonificaciones sobre ventas y/o servicios al 0%","402.02"
"account_subgroup_devoluciones,_descuentos_o_bonificaciones_sobre_ventas_y/o_servicios_exentos","l10n_mx.mx_coa","Devoluciones, descuentos o bonificaciones sobre ventas y/o servicios exentos","402.03"
"account_subgroup_devoluciones,_descuentos_o_bonificaciones_de_otros_ingresos","l10n_mx.mx_coa","Devoluciones, descuentos o bonificaciones de otros ingresos","402.04"
"account_subgroup_devoluciones,_descuentos_o_bonificaciones_sobre_ventas_y/o_servicios_en_zona_fronteriza_norte","l10n_mx.mx_coa","Devoluciones, descuentos o bonificaciones sobre ventas y/o servicios en zona fronteriza norte","402.05"
"account_subgroup_otros_ingresos","l10n_mx.mx_coa","Otros ingresos","403"
"account_subgroup_otros_ingresos_1","l10n_mx.mx_coa","Otros Ingresos","403.01"
"account_subgroup_otros_ingresos_nacionales_parte_relacionada","l10n_mx.mx_coa","Otros ingresos nacionales parte relacionada","403.02"
"account_subgroup_otros_ingresos_extranjeros_parte_relacionada","l10n_mx.mx_coa","Otros ingresos extranjeros parte relacionada","403.03"
"account_subgroup_ingresos_por_operaciones_discontinuas","l10n_mx.mx_coa","Ingresos por operaciones discontinuas","403.04"
"account_subgroup_ingresos_por_condonación_de_adeudo","l10n_mx.mx_coa","Ingresos por condonación de adeudo","403.05"
"account_group_costos","l10n_mx.mx_coa","Costos","5"
"account_subgroup_costo_de_venta_y/o_servicio","l10n_mx.mx_coa","Costo de venta y/o servicio","501"
"account_subgroup_costo_de_venta","l10n_mx.mx_coa","Costo de venta","501.01"
"account_subgroup_costo_de_servicios_(mano_de_obra)","l10n_mx.mx_coa","Costo de servicios (Mano de obra)","501.02"
"account_subgroup_materia_prima_directa_utilizada_para_la_producción","l10n_mx.mx_coa","Materia prima directa utilizada para la producción","501.03"
"account_subgroup_materia_prima_consumida_en_el_proceso_productivo","l10n_mx.mx_coa","Materia prima consumida en el proceso productivo","501.04"
"account_subgroup_mano_de_obra_directa_consumida","l10n_mx.mx_coa","Mano de obra directa consumida","501.05"
"account_subgroup_mano_de_obra_directa","l10n_mx.mx_coa","Mano de obra directa","501.06"
"account_subgroup_cargos_indirectos_de_producción","l10n_mx.mx_coa","Cargos indirectos de producción","501.07"
"account_subgroup_otros_conceptos_de_costo","l10n_mx.mx_coa","Otros conceptos de costo","501.08"
"account_subgroup_compras","l10n_mx.mx_coa","Compras","502"
"account_subgroup_compras_nacionales","l10n_mx.mx_coa","Compras nacionales","502.01"
"account_subgroup_compras_nacionales_parte_relacionada","l10n_mx.mx_coa","Compras nacionales parte relacionada","502.02"
"account_subgroup_compras_de_importación","l10n_mx.mx_coa","Compras de Importación","502.03"
"account_subgroup_compras_de_importación_partes_relacionadas","l10n_mx.mx_coa","Compras de Importación partes relacionadas","502.04"
"account_subgroup_devoluciones,_descuentos_o_bonificaciones_sobre_compras","l10n_mx.mx_coa","Devoluciones, descuentos o bonificaciones sobre compras","503"
"account_subgroup_devoluciones,_descuentos_o_bonificaciones_sobre_compras_1","l10n_mx.mx_coa","Devoluciones, descuentos o bonificaciones sobre compras","503.01"
"account_subgroup_otras_cuentas_de_costos","l10n_mx.mx_coa","Otras cuentas de costos","504"
"account_subgroup_gastos_indirectos_de_fabricación","l10n_mx.mx_coa","Gastos indirectos de fabricación","504.01"
"account_subgroup_gastos_indirectos_de_fabricación_de_partes_relacionadas_nacionales","l10n_mx.mx_coa","Gastos indirectos de fabricación de partes relacionadas nacionales","504.02"
"account_subgroup_gastos_indirectos_de_fabricación_de_partes_relacionadas_extranjeras","l10n_mx.mx_coa","Gastos indirectos de fabricación de partes relacionadas extranjeras","504.03"
"account_subgroup_otras_cuentas_de_costos_incurridos","l10n_mx.mx_coa","Otras cuentas de costos incurridos","504.04"
"account_subgroup_otras_cuentas_de_costos_incurridos_con_partes_relacionadas_nacionales","l10n_mx.mx_coa","Otras cuentas de costos incurridos con partes relacionadas nacionales","504.05"
"account_subgroup_otras_cuentas_de_costos_incurridos_con_partes_relacionadas_extranjeras","l10n_mx.mx_coa","Otras cuentas de costos incurridos con partes relacionadas extranjeras","504.06"
"account_subgroup_depreciación_de_edificios","l10n_mx.mx_coa","Depreciación de edificios","504.07"
"account_subgroup_depreciación_de_maquinaria_y_equipo","l10n_mx.mx_coa","Depreciación de maquinaria y equipo","504.08"
"account_subgroup_depreciación_de_automóviles,_autobuses,_camiones_de_carga,_tractocamiones,_montacargas_y_remolques","l10n_mx.mx_coa","Depreciación de automóviles, autobuses, camiones de carga, tractocamiones, montacargas y remolques","504.09"
"account_subgroup_depreciación_de_mobiliario_y_equipo_de_oficina","l10n_mx.mx_coa","Depreciación de mobiliario y equipo de oficina","504.10"
"account_subgroup_depreciación_de_equipo_de_cómputo","l10n_mx.mx_coa","Depreciación de equipo de cómputo","504.11"
"account_subgroup_depreciación_de_equipo_de_comunicación","l10n_mx.mx_coa","Depreciación de equipo de comunicación","504.12"
"account_subgroup_depreciación_de_activos_biológicos,_vegetales_y_semovientes","l10n_mx.mx_coa","Depreciación de activos biológicos, vegetales y semovientes","504.13"
"account_subgroup_depreciación_de_otros_activos_fijos","l10n_mx.mx_coa","Depreciación de otros activos fijos","504.14"
"account_subgroup_depreciación_de_ferrocarriles","l10n_mx.mx_coa","Depreciación de ferrocarriles","504.15"
"account_subgroup_depreciación_de_embarcaciones","l10n_mx.mx_coa","Depreciación de embarcaciones","504.16"
"account_subgroup_depreciación_de_aviones","l10n_mx.mx_coa","Depreciación de aviones","504.17"
"account_subgroup_depreciación_de_troqueles,_moldes,_matrices_y_herramental","l10n_mx.mx_coa","Depreciación de troqueles, moldes, matrices y herramental","504.18"
"account_subgroup_depreciación_de_equipo_de_comunicaciones_telefónicas","l10n_mx.mx_coa","Depreciación de equipo de comunicaciones telefónicas","504.19"
"account_subgroup_depreciación_de_equipo_de_comunicación_satelital","l10n_mx.mx_coa","Depreciación de equipo de comunicación satelital","504.20"
"account_subgroup_depreciación_de_equipo_de_adaptaciones_para_personas_con_capacidades_diferentes","l10n_mx.mx_coa","Depreciación de equipo de adaptaciones para personas con capacidades diferentes","504.21"
"account_subgroup_depreciación_de_maquinaria_y_equipo_de_generación_de_energía_de_fuentes_renovables_o_de_sistemas_de_cogeneración_de_electricidad_eficiente","l10n_mx.mx_coa","Depreciación de maquinaria y equipo de generación de energía de fuentes renovables o de sistemas de cogeneración de electricidad eficiente","504.22"
"account_subgroup_depreciación_de_adaptaciones_y_mejoras","l10n_mx.mx_coa","Depreciación de adaptaciones y mejoras","504.23"
"account_subgroup_depreciación_de_otra_maquinaria_y_equipo","l10n_mx.mx_coa","Depreciación de otra maquinaria y equipo","504.24"
"account_subgroup_otras_cuentas_de_costos_1","l10n_mx.mx_coa","Otras cuentas de costos","504.25"
"account_subgroup_costo_de_activo_fijo","l10n_mx.mx_coa","Costo de activo fijo","505"
"account_subgroup_costo_por_venta_de_activo_fijo","l10n_mx.mx_coa","Costo por venta de activo fijo","505.01"
"account_subgroup_costo_por_baja_de_activo_fijo","l10n_mx.mx_coa","Costo por baja de activo fijo","505.02"
"account_group_gastos","l10n_mx.mx_coa","Gastos","6"
"account_subgroup_gastos_generales","l10n_mx.mx_coa","Gastos generales","601"
"account_subgroup_sueldos_y_salarios","l10n_mx.mx_coa","Sueldos y salarios","601.01"
"account_subgroup_compensaciones","l10n_mx.mx_coa","Compensaciones","601.02"
"account_subgroup_tiempos_extras","l10n_mx.mx_coa","Tiempos extras","601.03"
"account_subgroup_premios_de_asistencia","l10n_mx.mx_coa","Premios de asistencia","601.04"
"account_subgroup_premios_de_puntualidad","l10n_mx.mx_coa","Premios de puntualidad","601.05"
"account_subgroup_vacaciones","l10n_mx.mx_coa","Vacaciones","601.06"
"account_subgroup_prima_vacacional","l10n_mx.mx_coa","Prima vacacional","601.07"
"account_subgroup_prima_dominical","l10n_mx.mx_coa","Prima dominical","601.08"
"account_subgroup_días_festivos","l10n_mx.mx_coa","Días festivos","601.09"
"account_subgroup_gratificaciones","l10n_mx.mx_coa","Gratificaciones","601.10"
"account_subgroup_primas_de_antigüedad","l10n_mx.mx_coa","Primas de antigüedad","601.11"
"account_subgroup_aguinaldo","l10n_mx.mx_coa","Aguinaldo","601.12"
"account_subgroup_indemnizaciones","l10n_mx.mx_coa","Indemnizaciones","601.13"
"account_subgroup_destajo","l10n_mx.mx_coa","Destajo","601.14"
"account_subgroup_despensa","l10n_mx.mx_coa","Despensa","601.15"
"account_subgroup_transporte","l10n_mx.mx_coa","Transporte","601.16"
"account_subgroup_servicio_médico","l10n_mx.mx_coa","Servicio médico","601.17"
"account_subgroup_ayuda_en_gastos_funerarios","l10n_mx.mx_coa","Ayuda en gastos funerarios","601.18"
"account_subgroup_fondo_de_ahorro","l10n_mx.mx_coa","Fondo de ahorro","601.19"
"account_subgroup_cuotas_sindicales","l10n_mx.mx_coa","Cuotas sindicales","601.20"
"account_subgroup_ptu","l10n_mx.mx_coa","PTU","601.21"
"account_subgroup_estímulo_al_personal","l10n_mx.mx_coa","Estímulo al personal","601.22"
"account_subgroup_previsión_social","l10n_mx.mx_coa","Previsión social","601.23"
"account_subgroup_aportaciones_para_el_plan_de_jubilación","l10n_mx.mx_coa","Aportaciones para el plan de jubilación","601.24"
"account_subgroup_otras_prestaciones_al_personal","l10n_mx.mx_coa","Otras prestaciones al personal","601.25"
"account_subgroup_cuotas_al_imss","l10n_mx.mx_coa","Cuotas al IMSS","601.26"
"account_subgroup_aportaciones_al_infonavit","l10n_mx.mx_coa","Aportaciones al infonavit","601.27"
"account_subgroup_aportaciones_al_sar","l10n_mx.mx_coa","Aportaciones al SAR","601.28"
"account_subgroup_impuesto_estatal_sobre_nóminas","l10n_mx.mx_coa","Impuesto estatal sobre nóminas","601.29"
"account_subgroup_otras_aportaciones","l10n_mx.mx_coa","Otras aportaciones","601.30"
"account_subgroup_asimilados_a_salarios","l10n_mx.mx_coa","Asimilados a salarios","601.31"
"account_subgroup_servicios_administrativos","l10n_mx.mx_coa","Servicios administrativos","601.32"
"account_subgroup_servicios_administrativos_partes_relacionadas","l10n_mx.mx_coa","Servicios administrativos partes relacionadas","601.33"
"account_subgroup_honorarios_a_personas_físicas_residentes_nacionales","l10n_mx.mx_coa","Honorarios a personas físicas residentes nacionales","601.34"
"account_subgroup_honorarios_a_personas_físicas_residentes_nacionales_partes_relacionadas","l10n_mx.mx_coa","Honorarios a personas físicas residentes nacionales partes relacionadas","601.35"
"account_subgroup_honorarios_a_personas_físicas_residentes_del_extranjero","l10n_mx.mx_coa","Honorarios a personas físicas residentes del extranjero","601.36"
"account_subgroup_honorarios_a_personas_físicas_residentes_del_extranjero_partes_relacionadas","l10n_mx.mx_coa","Honorarios a personas físicas residentes del extranjero partes relacionadas","601.37"
"account_subgroup_honorarios_a_personas_morales_residentes_nacionales","l10n_mx.mx_coa","Honorarios a personas morales residentes nacionales","601.38"
"account_subgroup_honorarios_a_personas_morales_residentes_nacionales_partes_relacionadas","l10n_mx.mx_coa","Honorarios a personas morales residentes nacionales partes relacionadas","601.39"
"account_subgroup_honorarios_a_personas_morales_residentes_del_extranjero","l10n_mx.mx_coa","Honorarios a personas morales residentes del extranjero","601.40"
"account_subgroup_honorarios_a_personas_morales_residentes_del_extranjero_partes_relacionadas","l10n_mx.mx_coa","Honorarios a personas morales residentes del extranjero partes relacionadas","601.41"
"account_subgroup_honorarios_aduanales_personas_físicas","l10n_mx.mx_coa","Honorarios aduanales personas físicas","601.42"
"account_subgroup_honorarios_aduanales_personas_morales","l10n_mx.mx_coa","Honorarios aduanales personas morales","601.43"
"account_subgroup_honorarios_al_consejo_de_administración","l10n_mx.mx_coa","Honorarios al consejo de administración","601.44"
"account_subgroup_arrendamiento_a_personas_físicas_residentes_nacionales","l10n_mx.mx_coa","Arrendamiento a personas físicas residentes nacionales","601.45"
"account_subgroup_arrendamiento_a_personas_morales_residentes_nacionales","l10n_mx.mx_coa","Arrendamiento a personas morales residentes nacionales","601.46"
"account_subgroup_arrendamiento_a_residentes_del_extranjero","l10n_mx.mx_coa","Arrendamiento a residentes del extranjero","601.47"
"account_subgroup_combustibles_y_lubricantes","l10n_mx.mx_coa","Combustibles y lubricantes","601.48"
"account_subgroup_viáticos_y_gastos_de_viaje","l10n_mx.mx_coa","Viáticos y gastos de viaje","601.49"
"account_subgroup_teléfono,_internet","l10n_mx.mx_coa","Teléfono, internet","601.50"
"account_subgroup_agua","l10n_mx.mx_coa","Agua","601.51"
"account_subgroup_energía_eléctrica","l10n_mx.mx_coa","Energía eléctrica","601.52"
"account_subgroup_vigilancia_y_seguridad","l10n_mx.mx_coa","Vigilancia y seguridad","601.53"
"account_subgroup_limpieza","l10n_mx.mx_coa","Limpieza","601.54"
"account_subgroup_papelería_y_artículos_de_oficina","l10n_mx.mx_coa","Papelería y artículos de oficina","601.55"
"account_subgroup_mantenimiento_y_conservación","l10n_mx.mx_coa","Mantenimiento y conservación","601.56"
"account_subgroup_seguros_y_fianzas","l10n_mx.mx_coa","Seguros y fianzas","601.57"
"account_subgroup_otros_impuestos_y_derechos","l10n_mx.mx_coa","Otros impuestos y derechos","601.58"
"account_subgroup_recargos_fiscales","l10n_mx.mx_coa","Recargos fiscales","601.59"
"account_subgroup_cuotas_y_suscripciones","l10n_mx.mx_coa","Cuotas y suscripciones","601.60"
"account_subgroup_propaganda_y_publicidad","l10n_mx.mx_coa","Propaganda y publicidad","601.61"
"account_subgroup_capacitación_al_personal","l10n_mx.mx_coa","Capacitación al personal","601.62"
"account_subgroup_donativos_y_ayudas","l10n_mx.mx_coa","Donativos y ayudas","601.63"
"account_subgroup_asistencia_técnica","l10n_mx.mx_coa","Asistencia técnica","601.64"
"account_subgroup_regalías_sujetas_a_otros_porcentajes","l10n_mx.mx_coa","Regalías sujetas a otros porcentajes","601.65"
"account_subgroup_regalías_sujetas_al_5%","l10n_mx.mx_coa","Regalías sujetas al 5%","601.66"
"account_subgroup_regalías_sujetas_al_10%","l10n_mx.mx_coa","Regalías sujetas al 10%","601.67"
"account_subgroup_regalías_sujetas_al_15%","l10n_mx.mx_coa","Regalías sujetas al 15%","601.68"
"account_subgroup_regalías_sujetas_al_25%","l10n_mx.mx_coa","Regalías sujetas al 25%","601.69"
"account_subgroup_regalías_sujetas_al_30%","l10n_mx.mx_coa","Regalías sujetas al 30%","601.70"
"account_subgroup_regalías_sin_retención","l10n_mx.mx_coa","Regalías sin retención","601.71"
"account_subgroup_fletes_y_acarreos","l10n_mx.mx_coa","Fletes y acarreos","601.72"
"account_subgroup_gastos_de_importación","l10n_mx.mx_coa","Gastos de importación","601.73"
"account_subgroup_comisiones_sobre_ventas","l10n_mx.mx_coa","Comisiones sobre ventas","601.74"
"account_subgroup_comisiones_por_tarjetas_de_crédito","l10n_mx.mx_coa","Comisiones por tarjetas de crédito","601.75"
"account_subgroup_patentes_y_marcas","l10n_mx.mx_coa","Patentes y marcas","601.76"
"account_subgroup_uniformes","l10n_mx.mx_coa","Uniformes","601.77"
"account_subgroup_prediales","l10n_mx.mx_coa","Prediales","601.78"
"account_subgroup_gastos_generales_de_urbanización","l10n_mx.mx_coa","Gastos generales de urbanización","601.79"
"account_subgroup_gastos_generales_de_construcción","l10n_mx.mx_coa","Gastos generales de construcción","601.80"
"account_subgroup_fletes_del_extranjero","l10n_mx.mx_coa","Fletes del extranjero","601.81"
"account_subgroup_recolección_de_bienes_del_sector_agropecuario_y/o_ganadero","l10n_mx.mx_coa","Recolección de bienes del sector agropecuario y/o ganadero","601.82"
"account_subgroup_gastos_no_deducibles_(sin_requisitos_fiscales)","l10n_mx.mx_coa","Gastos no deducibles (sin requisitos fiscales)","601.83"
"account_subgroup_otros_gastos_generales","l10n_mx.mx_coa","Otros gastos generales","601.84"
"account_subgroup_gastos_de_venta","l10n_mx.mx_coa","Gastos de venta","602"
"account_subgroup_sueldos_y_salarios_1","l10n_mx.mx_coa","Sueldos y salarios","602.01"
"account_subgroup_compensaciones_1","l10n_mx.mx_coa","Compensaciones","602.02"
"account_subgroup_tiempos_extras_1","l10n_mx.mx_coa","Tiempos extras","602.03"
"account_subgroup_premios_de_asistencia_1","l10n_mx.mx_coa","Premios de asistencia","602.04"
"account_subgroup_premios_de_puntualidad_1","l10n_mx.mx_coa","Premios de puntualidad","602.05"
"account_subgroup_vacaciones_1","l10n_mx.mx_coa","Vacaciones","602.06"
"account_subgroup_prima_vacacional_1","l10n_mx.mx_coa","Prima vacacional","602.07"
"account_subgroup_prima_dominical_1","l10n_mx.mx_coa","Prima dominical","602.08"
"account_subgroup_días_festivos_1","l10n_mx.mx_coa","Días festivos","602.09"
"account_subgroup_gratificaciones_1","l10n_mx.mx_coa","Gratificaciones","602.10"
"account_subgroup_primas_de_antigüedad_1","l10n_mx.mx_coa","Primas de antigüedad","602.11"
"account_subgroup_aguinaldo_1","l10n_mx.mx_coa","Aguinaldo","602.12"
"account_subgroup_indemnizaciones_1","l10n_mx.mx_coa","Indemnizaciones","602.13"
"account_subgroup_destajo_1","l10n_mx.mx_coa","Destajo","602.14"
"account_subgroup_despensa_1","l10n_mx.mx_coa","Despensa","602.15"
"account_subgroup_transporte_1","l10n_mx.mx_coa","Transporte","602.16"
"account_subgroup_servicio_médico_1","l10n_mx.mx_coa","Servicio médico","602.17"
"account_subgroup_ayuda_en_gastos_funerarios_1","l10n_mx.mx_coa","Ayuda en gastos funerarios","602.18"
"account_subgroup_fondo_de_ahorro_1","l10n_mx.mx_coa","Fondo de ahorro","602.19"
"account_subgroup_cuotas_sindicales_1","l10n_mx.mx_coa","Cuotas sindicales","602.20"
"account_subgroup_ptu_1","l10n_mx.mx_coa","PTU","602.21"
"account_subgroup_estímulo_al_personal_1","l10n_mx.mx_coa","Estímulo al personal","602.22"
"account_subgroup_previsión_social_1","l10n_mx.mx_coa","Previsión social","602.23"
"account_subgroup_aportaciones_para_el_plan_de_jubilación_1","l10n_mx.mx_coa","Aportaciones para el plan de jubilación","602.24"
"account_subgroup_otras_prestaciones_al_personal_1","l10n_mx.mx_coa","Otras prestaciones al personal","602.25"
"account_subgroup_cuotas_al_imss_1","l10n_mx.mx_coa","Cuotas al IMSS","602.26"
"account_subgroup_aportaciones_al_infonavit_1","l10n_mx.mx_coa","Aportaciones al infonavit","602.27"
"account_subgroup_aportaciones_al_sar_1","l10n_mx.mx_coa","Aportaciones al SAR","602.28"
"account_subgroup_impuesto_estatal_sobre_nóminas_1","l10n_mx.mx_coa","Impuesto estatal sobre nóminas","602.29"
"account_subgroup_otras_aportaciones_1","l10n_mx.mx_coa","Otras aportaciones","602.30"
"account_subgroup_asimilados_a_salarios_1","l10n_mx.mx_coa","Asimilados a salarios","602.31"
"account_subgroup_servicios_administrativos_1","l10n_mx.mx_coa","Servicios administrativos","602.32"
"account_subgroup_servicios_administrativos_partes_relacionadas_1","l10n_mx.mx_coa","Servicios administrativos partes relacionadas","602.33"
"account_subgroup_honorarios_a_personas_físicas_residentes_nacionales_1","l10n_mx.mx_coa","Honorarios a personas físicas residentes nacionales","602.34"
"account_subgroup_honorarios_a_personas_físicas_residentes_nacionales_partes_relacionadas_1","l10n_mx.mx_coa","Honorarios a personas físicas residentes nacionales partes relacionadas","602.35"
"account_subgroup_honorarios_a_personas_físicas_residentes_del_extranjero_1","l10n_mx.mx_coa","Honorarios a personas físicas residentes del extranjero","602.36"
"account_subgroup_honorarios_a_personas_físicas_residentes_del_extranjero_partes_relacionadas_1","l10n_mx.mx_coa","Honorarios a personas físicas residentes del extranjero partes relacionadas","602.37"
"account_subgroup_honorarios_a_personas_morales_residentes_nacionales_1","l10n_mx.mx_coa","Honorarios a personas morales residentes nacionales","602.38"
"account_subgroup_honorarios_a_personas_morales_residentes_nacionales_partes_relacionadas_1","l10n_mx.mx_coa","Honorarios a personas morales residentes nacionales partes relacionadas","602.39"
"account_subgroup_honorarios_a_personas_morales_residentes_del_extranjero_1","l10n_mx.mx_coa","Honorarios a personas morales residentes del extranjero","602.40"
"account_subgroup_honorarios_a_personas_morales_residentes_del_extranjero_partes_relacionadas_1","l10n_mx.mx_coa","Honorarios a personas morales residentes del extranjero partes relacionadas","602.41"
"account_subgroup_honorarios_aduanales_personas_físicas_1","l10n_mx.mx_coa","Honorarios aduanales personas físicas","602.42"
"account_subgroup_honorarios_aduanales_personas_morales_1","l10n_mx.mx_coa","Honorarios aduanales personas morales","602.43"
"account_subgroup_honorarios_al_consejo_de_administración_1","l10n_mx.mx_coa","Honorarios al consejo de administración","602.44"
"account_subgroup_arrendamiento_a_personas_físicas_residentes_nacionales_1","l10n_mx.mx_coa","Arrendamiento a personas físicas residentes nacionales","602.45"
"account_subgroup_arrendamiento_a_personas_morales_residentes_nacionales_1","l10n_mx.mx_coa","Arrendamiento a personas morales residentes nacionales","602.46"
"account_subgroup_arrendamiento_a_residentes_del_extranjero_1","l10n_mx.mx_coa","Arrendamiento a residentes del extranjero","602.47"
"account_subgroup_combustibles_y_lubricantes_1","l10n_mx.mx_coa","Combustibles y lubricantes","602.48"
"account_subgroup_viáticos_y_gastos_de_viaje_1","l10n_mx.mx_coa","Viáticos y gastos de viaje","602.49"
"account_subgroup_teléfono,_internet_1","l10n_mx.mx_coa","Teléfono, internet","602.50"
"account_subgroup_agua_1","l10n_mx.mx_coa","Agua","602.51"
"account_subgroup_energía_eléctrica_1","l10n_mx.mx_coa","Energía eléctrica","602.52"
"account_subgroup_vigilancia_y_seguridad_1","l10n_mx.mx_coa","Vigilancia y seguridad","602.53"
"account_subgroup_limpieza_1","l10n_mx.mx_coa","Limpieza","602.54"
"account_subgroup_papelería_y_artículos_de_oficina_1","l10n_mx.mx_coa","Papelería y artículos de oficina","602.55"
"account_subgroup_mantenimiento_y_conservación_1","l10n_mx.mx_coa","Mantenimiento y conservación","602.56"
"account_subgroup_seguros_y_fianzas_1","l10n_mx.mx_coa","Seguros y fianzas","602.57"
"account_subgroup_otros_impuestos_y_derechos_1","l10n_mx.mx_coa","Otros impuestos y derechos","602.58"
"account_subgroup_recargos_fiscales_1","l10n_mx.mx_coa","Recargos fiscales","602.59"
"account_subgroup_cuotas_y_suscripciones_1","l10n_mx.mx_coa","Cuotas y suscripciones","602.60"
"account_subgroup_propaganda_y_publicidad_1","l10n_mx.mx_coa","Propaganda y publicidad","602.61"
"account_subgroup_capacitación_al_personal_1","l10n_mx.mx_coa","Capacitación al personal","602.62"
"account_subgroup_donativos_y_ayudas_1","l10n_mx.mx_coa","Donativos y ayudas","602.63"
"account_subgroup_asistencia_técnica_1","l10n_mx.mx_coa","Asistencia técnica","602.64"
"account_subgroup_regalías_sujetas_a_otros_porcentajes_1","l10n_mx.mx_coa","Regalías sujetas a otros porcentajes","602.65"
"account_subgroup_regalías_sujetas_al_5%_1","l10n_mx.mx_coa","Regalías sujetas al 5%","602.66"
"account_subgroup_regalías_sujetas_al_10%_1","l10n_mx.mx_coa","Regalías sujetas al 10%","602.67"
"account_subgroup_regalías_sujetas_al_15%_1","l10n_mx.mx_coa","Regalías sujetas al 15%","602.68"
"account_subgroup_regalías_sujetas_al_25%_1","l10n_mx.mx_coa","Regalías sujetas al 25%","602.69"
"account_subgroup_regalías_sujetas_al_30%_1","l10n_mx.mx_coa","Regalías sujetas al 30%","602.70"
"account_subgroup_regalías_sin_retención_1","l10n_mx.mx_coa","Regalías sin retención","602.71"
"account_subgroup_fletes_y_acarreos_1","l10n_mx.mx_coa","Fletes y acarreos","602.72"
"account_subgroup_gastos_de_importación_1","l10n_mx.mx_coa","Gastos de importación","602.73"
"account_subgroup_comisiones_sobre_ventas_1","l10n_mx.mx_coa","Comisiones sobre ventas","602.74"
"account_subgroup_comisiones_por_tarjetas_de_crédito_1","l10n_mx.mx_coa","Comisiones por tarjetas de crédito","602.75"
"account_subgroup_patentes_y_marcas_1","l10n_mx.mx_coa","Patentes y marcas","602.76"
"account_subgroup_uniformes_1","l10n_mx.mx_coa","Uniformes","602.77"
"account_subgroup_prediales_1","l10n_mx.mx_coa","Prediales","602.78"
"account_subgroup_gastos_de_venta_de_urbanización","l10n_mx.mx_coa","Gastos de venta de urbanización","602.79"
"account_subgroup_gastos_de_venta_de_construcción","l10n_mx.mx_coa","Gastos de venta de construcción","602.80"
"account_subgroup_fletes_del_extranjero_1","l10n_mx.mx_coa","Fletes del extranjero","602.81"
"account_subgroup_recolección_de_bienes_del_sector_agropecuario_y/o_ganadero_1","l10n_mx.mx_coa","Recolección de bienes del sector agropecuario y/o ganadero","602.82"
"account_subgroup_gastos_no_deducibles_(sin_requisitos_fiscales)_1","l10n_mx.mx_coa","Gastos no deducibles (sin requisitos fiscales)","602.83"
"account_subgroup_otros_gastos_de_venta","l10n_mx.mx_coa","Otros gastos de venta","602.84"
"account_subgroup_gastos_de_administración","l10n_mx.mx_coa","Gastos de administración","603"
"account_subgroup_sueldos_y_salarios_2","l10n_mx.mx_coa","Sueldos y salarios","603.01"
"account_subgroup_compensaciones_2","l10n_mx.mx_coa","Compensaciones","603.02"
"account_subgroup_tiempos_extras_2","l10n_mx.mx_coa","Tiempos extras","603.03"
"account_subgroup_premios_de_asistencia_2","l10n_mx.mx_coa","Premios de asistencia","603.04"
"account_subgroup_premios_de_puntualidad_2","l10n_mx.mx_coa","Premios de puntualidad","603.05"
"account_subgroup_vacaciones_2","l10n_mx.mx_coa","Vacaciones","603.06"
"account_subgroup_prima_vacacional_2","l10n_mx.mx_coa","Prima vacacional","603.07"
"account_subgroup_prima_dominical_2","l10n_mx.mx_coa","Prima dominical","603.08"
"account_subgroup_días_festivos_2","l10n_mx.mx_coa","Días festivos","603.09"
"account_subgroup_gratificaciones_2","l10n_mx.mx_coa","Gratificaciones","603.10"
"account_subgroup_primas_de_antigüedad_2","l10n_mx.mx_coa","Primas de antigüedad","603.11"
"account_subgroup_aguinaldo_2","l10n_mx.mx_coa","Aguinaldo","603.12"
"account_subgroup_indemnizaciones_2","l10n_mx.mx_coa","Indemnizaciones","603.13"
"account_subgroup_destajo_2","l10n_mx.mx_coa","Destajo","603.14"
"account_subgroup_despensa_2","l10n_mx.mx_coa","Despensa","603.15"
"account_subgroup_transporte_2","l10n_mx.mx_coa","Transporte","603.16"
"account_subgroup_servicio_médico_2","l10n_mx.mx_coa","Servicio médico","603.17"
"account_subgroup_ayuda_en_gastos_funerarios_2","l10n_mx.mx_coa","Ayuda en gastos funerarios","603.18"
"account_subgroup_fondo_de_ahorro_2","l10n_mx.mx_coa","Fondo de ahorro","603.19"
"account_subgroup_cuotas_sindicales_2","l10n_mx.mx_coa","Cuotas sindicales","603.20"
"account_subgroup_ptu_2","l10n_mx.mx_coa","PTU","603.21"
"account_subgroup_estímulo_al_personal_2","l10n_mx.mx_coa","Estímulo al personal","603.22"
"account_subgroup_previsión_social_2","l10n_mx.mx_coa","Previsión social","603.23"
"account_subgroup_aportaciones_para_el_plan_de_jubilación_2","l10n_mx.mx_coa","Aportaciones para el plan de jubilación","603.24"
"account_subgroup_otras_prestaciones_al_personal_2","l10n_mx.mx_coa","Otras prestaciones al personal","603.25"
"account_subgroup_cuotas_al_imss_2","l10n_mx.mx_coa","Cuotas al IMSS","603.26"
"account_subgroup_aportaciones_al_infonavit_2","l10n_mx.mx_coa","Aportaciones al infonavit","603.27"
"account_subgroup_aportaciones_al_sar_2","l10n_mx.mx_coa","Aportaciones al SAR","603.28"
"account_subgroup_impuesto_estatal_sobre_nóminas_2","l10n_mx.mx_coa","Impuesto estatal sobre nóminas","603.29"
"account_subgroup_otras_aportaciones_2","l10n_mx.mx_coa","Otras aportaciones","603.30"
"account_subgroup_asimilados_a_salarios_2","l10n_mx.mx_coa","Asimilados a salarios","603.31"
"account_subgroup_servicios_administrativos_2","l10n_mx.mx_coa","Servicios administrativos","603.32"
"account_subgroup_servicios_administrativos_partes_relacionadas_2","l10n_mx.mx_coa","Servicios administrativos partes relacionadas","603.33"
"account_subgroup_honorarios_a_personas_físicas_residentes_nacionales_2","l10n_mx.mx_coa","Honorarios a personas físicas residentes nacionales","603.34"
"account_subgroup_honorarios_a_personas_físicas_residentes_nacionales_partes_relacionadas_2","l10n_mx.mx_coa","Honorarios a personas físicas residentes nacionales partes relacionadas","603.35"
"account_subgroup_honorarios_a_personas_físicas_residentes_del_extranjero_2","l10n_mx.mx_coa","Honorarios a personas físicas residentes del extranjero","603.36"
"account_subgroup_honorarios_a_personas_físicas_residentes_del_extranjero_partes_relacionadas_2","l10n_mx.mx_coa","Honorarios a personas físicas residentes del extranjero partes relacionadas","603.37"
"account_subgroup_honorarios_a_personas_morales_residentes_nacionales_2","l10n_mx.mx_coa","Honorarios a personas morales residentes nacionales","603.38"
"account_subgroup_honorarios_a_personas_morales_residentes_nacionales_partes_relacionadas_2","l10n_mx.mx_coa","Honorarios a personas morales residentes nacionales partes relacionadas","603.39"
"account_subgroup_honorarios_a_personas_morales_residentes_del_extranjero_2","l10n_mx.mx_coa","Honorarios a personas morales residentes del extranjero","603.40"
"account_subgroup_honorarios_a_personas_morales_residentes_del_extranjero_partes_relacionadas_2","l10n_mx.mx_coa","Honorarios a personas morales residentes del extranjero partes relacionadas","603.41"
"account_subgroup_honorarios_aduanales_personas_físicas_2","l10n_mx.mx_coa","Honorarios aduanales personas físicas","603.42"
"account_subgroup_honorarios_aduanales_personas_morales_2","l10n_mx.mx_coa","Honorarios aduanales personas morales","603.43"
"account_subgroup_honorarios_al_consejo_de_administración_2","l10n_mx.mx_coa","Honorarios al consejo de administración","603.44"
"account_subgroup_arrendamiento_a_personas_físicas_residentes_nacionales_2","l10n_mx.mx_coa","Arrendamiento a personas físicas residentes nacionales","603.45"
"account_subgroup_arrendamiento_a_personas_morales_residentes_nacionales_2","l10n_mx.mx_coa","Arrendamiento a personas morales residentes nacionales","603.46"
"account_subgroup_arrendamiento_a_residentes_del_extranjero_2","l10n_mx.mx_coa","Arrendamiento a residentes del extranjero","603.47"
"account_subgroup_combustibles_y_lubricantes_2","l10n_mx.mx_coa","Combustibles y lubricantes","603.48"
"account_subgroup_viáticos_y_gastos_de_viaje_2","l10n_mx.mx_coa","Viáticos y gastos de viaje","603.49"
"account_subgroup_teléfono,_internet_2","l10n_mx.mx_coa","Teléfono, internet","603.50"
"account_subgroup_agua_2","l10n_mx.mx_coa","Agua","603.51"
"account_subgroup_energía_eléctrica_2","l10n_mx.mx_coa","Energía eléctrica","603.52"
"account_subgroup_vigilancia_y_seguridad_2","l10n_mx.mx_coa","Vigilancia y seguridad","603.53"
"account_subgroup_limpieza_2","l10n_mx.mx_coa","Limpieza","603.54"
"account_subgroup_papelería_y_artículos_de_oficina_2","l10n_mx.mx_coa","Papelería y artículos de oficina","603.55"
"account_subgroup_mantenimiento_y_conservación_2","l10n_mx.mx_coa","Mantenimiento y conservación","603.56"
"account_subgroup_seguros_y_fianzas_2","l10n_mx.mx_coa","Seguros y fianzas","603.57"
"account_subgroup_otros_impuestos_y_derechos_2","l10n_mx.mx_coa","Otros impuestos y derechos","603.58"
"account_subgroup_recargos_fiscales_2","l10n_mx.mx_coa","Recargos fiscales","603.59"
"account_subgroup_cuotas_y_suscripciones_2","l10n_mx.mx_coa","Cuotas y suscripciones","603.60"
"account_subgroup_propaganda_y_publicidad_2","l10n_mx.mx_coa","Propaganda y publicidad","603.61"
"account_subgroup_capacitación_al_personal_2","l10n_mx.mx_coa","Capacitación al personal","603.62"
"account_subgroup_donativos_y_ayudas_2","l10n_mx.mx_coa","Donativos y ayudas","603.63"
"account_subgroup_asistencia_técnica_2","l10n_mx.mx_coa","Asistencia técnica","603.64"
"account_subgroup_regalías_sujetas_a_otros_porcentajes_2","l10n_mx.mx_coa","Regalías sujetas a otros porcentajes","603.65"
"account_subgroup_regalías_sujetas_al_5%_2","l10n_mx.mx_coa","Regalías sujetas al 5%","603.66"
"account_subgroup_regalías_sujetas_al_10%_2","l10n_mx.mx_coa","Regalías sujetas al 10%","603.67"
"account_subgroup_regalías_sujetas_al_15%_2","l10n_mx.mx_coa","Regalías sujetas al 15%","603.68"
"account_subgroup_regalías_sujetas_al_25%_2","l10n_mx.mx_coa","Regalías sujetas al 25%","603.69"
"account_subgroup_regalías_sujetas_al_30%_2","l10n_mx.mx_coa","Regalías sujetas al 30%","603.70"
"account_subgroup_regalías_sin_retención_2","l10n_mx.mx_coa","Regalías sin retención","603.71"
"account_subgroup_fletes_y_acarreos_2","l10n_mx.mx_coa","Fletes y acarreos","603.72"
"account_subgroup_gastos_de_importación_2","l10n_mx.mx_coa","Gastos de importación","603.73"
"account_subgroup_patentes_y_marcas_2","l10n_mx.mx_coa","Patentes y marcas","603.74"
"account_subgroup_uniformes_2","l10n_mx.mx_coa","Uniformes","603.75"
"account_subgroup_prediales_2","l10n_mx.mx_coa","Prediales","603.76"
"account_subgroup_gastos_de_administración_de_urbanización","l10n_mx.mx_coa","Gastos de administración de urbanización","603.77"
"account_subgroup_gastos_de_administración_de_construcción","l10n_mx.mx_coa","Gastos de administración de construcción","603.78"
"account_subgroup_fletes_del_extranjero_2","l10n_mx.mx_coa","Fletes del extranjero","603.79"
"account_subgroup_recolección_de_bienes_del_sector_agropecuario_y/o_ganadero_2","l10n_mx.mx_coa","Recolección de bienes del sector agropecuario y/o ganadero","603.80"
"account_subgroup_gastos_no_deducibles_(sin_requisitos_fiscales)_2","l10n_mx.mx_coa","Gastos no deducibles (sin requisitos fiscales)","603.81"
"account_subgroup_otros_gastos_de_administración","l10n_mx.mx_coa","Otros gastos de administración","603.82"
"account_subgroup_gastos_de_fabricación","l10n_mx.mx_coa","Gastos de fabricación","604"
"account_subgroup_sueldos_y_salarios_3","l10n_mx.mx_coa","Sueldos y salarios","604.01"
"account_subgroup_compensaciones_3","l10n_mx.mx_coa","Compensaciones","604.02"
"account_subgroup_tiempos_extras_3","l10n_mx.mx_coa","Tiempos extras","604.03"
"account_subgroup_premios_de_asistencia_3","l10n_mx.mx_coa","Premios de asistencia","604.04"
"account_subgroup_premios_de_puntualidad_3","l10n_mx.mx_coa","Premios de puntualidad","604.05"
"account_subgroup_vacaciones_3","l10n_mx.mx_coa","Vacaciones","604.06"
"account_subgroup_prima_vacacional_3","l10n_mx.mx_coa","Prima vacacional","604.07"
"account_subgroup_prima_dominical_3","l10n_mx.mx_coa","Prima dominical","604.08"
"account_subgroup_días_festivos_3","l10n_mx.mx_coa","Días festivos","604.09"
"account_subgroup_gratificaciones_3","l10n_mx.mx_coa","Gratificaciones","604.10"
"account_subgroup_primas_de_antigüedad_3","l10n_mx.mx_coa","Primas de antigüedad","604.11"
"account_subgroup_aguinaldo_3","l10n_mx.mx_coa","Aguinaldo","604.12"
"account_subgroup_indemnizaciones_3","l10n_mx.mx_coa","Indemnizaciones","604.13"
"account_subgroup_destajo_3","l10n_mx.mx_coa","Destajo","604.14"
"account_subgroup_despensa_3","l10n_mx.mx_coa","Despensa","604.15"
"account_subgroup_transporte_3","l10n_mx.mx_coa","Transporte","604.16"
"account_subgroup_servicio_médico_3","l10n_mx.mx_coa","Servicio médico","604.17"
"account_subgroup_ayuda_en_gastos_funerarios_3","l10n_mx.mx_coa","Ayuda en gastos funerarios","604.18"
"account_subgroup_fondo_de_ahorro_3","l10n_mx.mx_coa","Fondo de ahorro","604.19"
"account_subgroup_cuotas_sindicales_3","l10n_mx.mx_coa","Cuotas sindicales","604.20"
"account_subgroup_ptu_3","l10n_mx.mx_coa","PTU","604.21"
"account_subgroup_estímulo_al_personal_3","l10n_mx.mx_coa","Estímulo al personal","604.22"
"account_subgroup_previsión_social_3","l10n_mx.mx_coa","Previsión social","604.23"
"account_subgroup_aportaciones_para_el_plan_de_jubilación_3","l10n_mx.mx_coa","Aportaciones para el plan de jubilación","604.24"
"account_subgroup_otras_prestaciones_al_personal_3","l10n_mx.mx_coa","Otras prestaciones al personal","604.25"
"account_subgroup_cuotas_al_imss_3","l10n_mx.mx_coa","Cuotas al IMSS","604.26"
"account_subgroup_aportaciones_al_infonavit_3","l10n_mx.mx_coa","Aportaciones al infonavit","604.27"
"account_subgroup_aportaciones_al_sar_3","l10n_mx.mx_coa","Aportaciones al SAR","604.28"
"account_subgroup_impuesto_estatal_sobre_nóminas_3","l10n_mx.mx_coa","Impuesto estatal sobre nóminas","604.29"
"account_subgroup_otras_aportaciones_3","l10n_mx.mx_coa","Otras aportaciones","604.30"
"account_subgroup_asimilados_a_salarios_3","l10n_mx.mx_coa","Asimilados a salarios","604.31"
"account_subgroup_servicios_administrativos_3","l10n_mx.mx_coa","Servicios administrativos","604.32"
"account_subgroup_servicios_administrativos_partes_relacionadas_3","l10n_mx.mx_coa","Servicios administrativos partes relacionadas","604.33"
"account_subgroup_honorarios_a_personas_físicas_residentes_nacionales_3","l10n_mx.mx_coa","Honorarios a personas físicas residentes nacionales","604.34"
"account_subgroup_honorarios_a_personas_físicas_residentes_nacionales_partes_relacionadas_3","l10n_mx.mx_coa","Honorarios a personas físicas residentes nacionales partes relacionadas","604.35"
"account_subgroup_honorarios_a_personas_físicas_residentes_del_extranjero_3","l10n_mx.mx_coa","Honorarios a personas físicas residentes del extranjero","604.36"
"account_subgroup_honorarios_a_personas_físicas_residentes_del_extranjero_partes_relacionadas_3","l10n_mx.mx_coa","Honorarios a personas físicas residentes del extranjero partes relacionadas","604.37"
"account_subgroup_honorarios_a_personas_morales_residentes_nacionales_3","l10n_mx.mx_coa","Honorarios a personas morales residentes nacionales","604.38"
"account_subgroup_honorarios_a_personas_morales_residentes_nacionales_partes_relacionadas_3","l10n_mx.mx_coa","Honorarios a personas morales residentes nacionales partes relacionadas","604.39"
"account_subgroup_honorarios_a_personas_morales_residentes_del_extranjero_3","l10n_mx.mx_coa","Honorarios a personas morales residentes del extranjero","604.40"
"account_subgroup_honorarios_a_personas_morales_residentes_del_extranjero_partes_relacionadas_3","l10n_mx.mx_coa","Honorarios a personas morales residentes del extranjero partes relacionadas","604.41"
"account_subgroup_honorarios_aduanales_personas_físicas_3","l10n_mx.mx_coa","Honorarios aduanales personas físicas","604.42"
"account_subgroup_honorarios_aduanales_personas_morales_3","l10n_mx.mx_coa","Honorarios aduanales personas morales","604.43"
"account_subgroup_honorarios_al_consejo_de_administración_3","l10n_mx.mx_coa","Honorarios al consejo de administración","604.44"
"account_subgroup_arrendamiento_a_personas_físicas_residentes_nacionales_3","l10n_mx.mx_coa","Arrendamiento a personas físicas residentes nacionales","604.45"
"account_subgroup_arrendamiento_a_personas_morales_residentes_nacionales_3","l10n_mx.mx_coa","Arrendamiento a personas morales residentes nacionales","604.46"
"account_subgroup_arrendamiento_a_residentes_del_extranjero_3","l10n_mx.mx_coa","Arrendamiento a residentes del extranjero","604.47"
"account_subgroup_combustibles_y_lubricantes_3","l10n_mx.mx_coa","Combustibles y lubricantes","604.48"
"account_subgroup_viáticos_y_gastos_de_viaje_3","l10n_mx.mx_coa","Viáticos y gastos de viaje","604.49"
"account_subgroup_teléfono,_internet_3","l10n_mx.mx_coa","Teléfono, internet","604.50"
"account_subgroup_agua_3","l10n_mx.mx_coa","Agua","604.51"
"account_subgroup_energía_eléctrica_3","l10n_mx.mx_coa","Energía eléctrica","604.52"
"account_subgroup_vigilancia_y_seguridad_3","l10n_mx.mx_coa","Vigilancia y seguridad","604.53"
"account_subgroup_limpieza_3","l10n_mx.mx_coa","Limpieza","604.54"
"account_subgroup_papelería_y_artículos_de_oficina_3","l10n_mx.mx_coa","Papelería y artículos de oficina","604.55"
"account_subgroup_mantenimiento_y_conservación_3","l10n_mx.mx_coa","Mantenimiento y conservación","604.56"
"account_subgroup_seguros_y_fianzas_3","l10n_mx.mx_coa","Seguros y fianzas","604.57"
"account_subgroup_otros_impuestos_y_derechos_3","l10n_mx.mx_coa","Otros impuestos y derechos","604.58"
"account_subgroup_recargos_fiscales_3","l10n_mx.mx_coa","Recargos fiscales","604.59"
"account_subgroup_cuotas_y_suscripciones_3","l10n_mx.mx_coa","Cuotas y suscripciones","604.60"
"account_subgroup_propaganda_y_publicidad_3","l10n_mx.mx_coa","Propaganda y publicidad","604.61"
"account_subgroup_capacitación_al_personal_3","l10n_mx.mx_coa","Capacitación al personal","604.62"
"account_subgroup_donativos_y_ayudas_3","l10n_mx.mx_coa","Donativos y ayudas","604.63"
"account_subgroup_asistencia_técnica_3","l10n_mx.mx_coa","Asistencia técnica","604.64"
"account_subgroup_regalías_sujetas_a_otros_porcentajes_3","l10n_mx.mx_coa","Regalías sujetas a otros porcentajes","604.65"
"account_subgroup_regalías_sujetas_al_5%_3","l10n_mx.mx_coa","Regalías sujetas al 5%","604.66"
"account_subgroup_regalías_sujetas_al_10%_3","l10n_mx.mx_coa","Regalías sujetas al 10%","604.67"
"account_subgroup_regalías_sujetas_al_15%_3","l10n_mx.mx_coa","Regalías sujetas al 15%","604.68"
"account_subgroup_regalías_sujetas_al_25%_3","l10n_mx.mx_coa","Regalías sujetas al 25%","604.69"
"account_subgroup_regalías_sujetas_al_30%_3","l10n_mx.mx_coa","Regalías sujetas al 30%","604.70"
"account_subgroup_regalías_sin_retención_3","l10n_mx.mx_coa","Regalías sin retención","604.71"
"account_subgroup_fletes_y_acarreos_3","l10n_mx.mx_coa","Fletes y acarreos","604.72"
"account_subgroup_gastos_de_importación_3","l10n_mx.mx_coa","Gastos de importación","604.73"
"account_subgroup_patentes_y_marcas_3","l10n_mx.mx_coa","Patentes y marcas","604.74"
"account_subgroup_uniformes_3","l10n_mx.mx_coa","Uniformes","604.75"
"account_subgroup_prediales_3","l10n_mx.mx_coa","Prediales","604.76"
"account_subgroup_gastos_de_fabricación_de_urbanización","l10n_mx.mx_coa","Gastos de fabricación de urbanización","604.77"
"account_subgroup_gastos_de_fabricación_de_construcción","l10n_mx.mx_coa","Gastos de fabricación de construcción","604.78"
"account_subgroup_fletes_del_extranjero_3","l10n_mx.mx_coa","Fletes del extranjero","604.79"
"account_subgroup_recolección_de_bienes_del_sector_agropecuario_y/o_ganadero_3","l10n_mx.mx_coa","Recolección de bienes del sector agropecuario y/o ganadero","604.80"
"account_subgroup_gastos_no_deducibles_(sin_requisitos_fiscales)_3","l10n_mx.mx_coa","Gastos no deducibles (sin requisitos fiscales)","604.81"
"account_subgroup_otros_gastos_de_fabricación","l10n_mx.mx_coa","Otros gastos de fabricación","604.82"
"account_subgroup_mano_de_obra_directa_1","l10n_mx.mx_coa","Mano de obra directa","605"
"account_subgroup_mano_de_obra","l10n_mx.mx_coa","Mano de obra","605.01"
"account_subgroup_sueldos_y_salarios_4","l10n_mx.mx_coa","Sueldos y Salarios","605.02"
"account_subgroup_compensaciones_4","l10n_mx.mx_coa","Compensaciones","605.03"
"account_subgroup_tiempos_extras_4","l10n_mx.mx_coa","Tiempos extras","605.04"
"account_subgroup_premios_de_asistencia_4","l10n_mx.mx_coa","Premios de asistencia","605.05"
"account_subgroup_premios_de_puntualidad_4","l10n_mx.mx_coa","Premios de puntualidad","605.06"
"account_subgroup_vacaciones_4","l10n_mx.mx_coa","Vacaciones","605.07"
"account_subgroup_prima_vacacional_4","l10n_mx.mx_coa","Prima vacacional","605.08"
"account_subgroup_prima_dominical_4","l10n_mx.mx_coa","Prima dominical","605.09"
"account_subgroup_días_festivos_4","l10n_mx.mx_coa","Días festivos","605.10"
"account_subgroup_gratificaciones_4","l10n_mx.mx_coa","Gratificaciones","605.11"
"account_subgroup_primas_de_antigüedad_4","l10n_mx.mx_coa","Primas de antigüedad","605.12"
"account_subgroup_aguinaldo_4","l10n_mx.mx_coa","Aguinaldo","605.13"
"account_subgroup_indemnizaciones_4","l10n_mx.mx_coa","Indemnizaciones","605.14"
"account_subgroup_destajo_4","l10n_mx.mx_coa","Destajo","605.15"
"account_subgroup_despensa_4","l10n_mx.mx_coa","Despensa","605.16"
"account_subgroup_transporte_4","l10n_mx.mx_coa","Transporte","605.17"
"account_subgroup_servicio_médico_4","l10n_mx.mx_coa","Servicio médico","605.18"
"account_subgroup_ayuda_en_gastos_funerarios_4","l10n_mx.mx_coa","Ayuda en gastos funerarios","605.19"
"account_subgroup_fondo_de_ahorro_4","l10n_mx.mx_coa","Fondo de ahorro","605.20"
"account_subgroup_cuotas_sindicales_4","l10n_mx.mx_coa","Cuotas sindicales","605.21"
"account_subgroup_ptu_4","l10n_mx.mx_coa","PTU","605.22"
"account_subgroup_estímulo_al_personal_4","l10n_mx.mx_coa","Estímulo al personal","605.23"
"account_subgroup_previsión_social_4","l10n_mx.mx_coa","Previsión social","605.24"
"account_subgroup_aportaciones_para_el_plan_de_jubilación_4","l10n_mx.mx_coa","Aportaciones para el plan de jubilación","605.25"
"account_subgroup_otras_prestaciones_al_personal_4","l10n_mx.mx_coa","Otras prestaciones al personal","605.26"
"account_subgroup_asimilados_a_salarios_4","l10n_mx.mx_coa","Asimilados a salarios","605.27"
"account_subgroup_cuotas_al_imss_4","l10n_mx.mx_coa","Cuotas al IMSS","605.28"
"account_subgroup_aportaciones_al_infonavit_4","l10n_mx.mx_coa","Aportaciones al infonavit","605.29"
"account_subgroup_aportaciones_al_sar_4","l10n_mx.mx_coa","Aportaciones al SAR","605.30"
"account_subgroup_otros_costos_de_mano_de_obra_directa","l10n_mx.mx_coa","Otros costos de mano de obra directa","605.31"
"account_subgroup_facilidades_administrativas_fiscales","l10n_mx.mx_coa","Facilidades administrativas fiscales","606"
"account_subgroup_facilidades_administrativas_fiscales_1","l10n_mx.mx_coa","Facilidades administrativas fiscales","606.01"
"account_subgroup_participación_de_los_trabajadores_en_las_utilidades","l10n_mx.mx_coa","Participación de los trabajadores en las utilidades","607"
"account_subgroup_participación_de_los_trabajadores_en_las_utilidades_1","l10n_mx.mx_coa","Participación de los trabajadores en las utilidades","607.01"
"account_subgroup_participación_en_resultados_de_subsidiarias","l10n_mx.mx_coa","Participación en resultados de subsidiarias","608"
"account_subgroup_participación_en_resultados_de_subsidiarias_1","l10n_mx.mx_coa","Participación en resultados de subsidiarias","608.01"
"account_subgroup_participación_en_resultados_de_asociadas","l10n_mx.mx_coa","Participación en resultados de asociadas","609"
"account_subgroup_participación_en_resultados_de_asociadas_1","l10n_mx.mx_coa","Participación en resultados de asociadas","609.01"
"account_subgroup_participación_de_los_trabajadores_en_las_utilidades_diferida_2","l10n_mx.mx_coa","Participación de los trabajadores en las utilidades diferida","610"
"account_subgroup_participación_de_los_trabajadores_en_las_utilidades_diferida_3","l10n_mx.mx_coa","Participación de los trabajadores en las utilidades diferida","610.01"
"account_subgroup_impuesto_sobre_la_renta","l10n_mx.mx_coa","Impuesto Sobre la renta","611"
"account_subgroup_impuesto_sobre_la_renta_1","l10n_mx.mx_coa","Impuesto Sobre la renta","611.01"
"account_subgroup_impuesto_sobre_la_renta_por_remanente_distribuible","l10n_mx.mx_coa","Impuesto Sobre la renta por remanente distribuible","611.02"
"account_subgroup_gastos_no_deducibles_para_cufin","l10n_mx.mx_coa","Gastos no deducibles para CUFIN","612"
"account_subgroup_gastos_no_deducibles_para_cufin_1","l10n_mx.mx_coa","Gastos no deducibles para CUFIN","612.01"
"account_subgroup_depreciación_contable","l10n_mx.mx_coa","Depreciación contable","613"
"account_subgroup_depreciación_de_edificios_1","l10n_mx.mx_coa","Depreciación de edificios","613.01"
"account_subgroup_depreciación_de_maquinaria_y_equipo_1","l10n_mx.mx_coa","Depreciación de maquinaria y equipo","613.02"
"account_subgroup_depreciación_de_automóviles,_autobuses,_camiones_de_carga,_tractocamiones,_montacargas_y_remolques_1","l10n_mx.mx_coa","Depreciación de automóviles, autobuses, camiones de carga, tractocamiones, montacargas y remolques","613.03"
"account_subgroup_depreciación_de_mobiliario_y_equipo_de_oficina_1","l10n_mx.mx_coa","Depreciación de mobiliario y equipo de oficina","613.04"
"account_subgroup_depreciación_de_equipo_de_cómputo_1","l10n_mx.mx_coa","Depreciación de equipo de cómputo","613.05"
"account_subgroup_depreciación_de_equipo_de_comunicación_1","l10n_mx.mx_coa","Depreciación de equipo de comunicación","613.06"
"account_subgroup_depreciación_de_activos_biológicos,_vegetales_y_semovientes_1","l10n_mx.mx_coa","Depreciación de activos biológicos, vegetales y semovientes","613.07"
"account_subgroup_depreciación_de_otros_activos_fijos_1","l10n_mx.mx_coa","Depreciación de otros activos fijos","613.08"
"account_subgroup_depreciación_de_ferrocarriles_1","l10n_mx.mx_coa","Depreciación de ferrocarriles","613.09"
"account_subgroup_depreciación_de_embarcaciones_1","l10n_mx.mx_coa","Depreciación de embarcaciones","613.10"
"account_subgroup_depreciación_de_aviones_1","l10n_mx.mx_coa","Depreciación de aviones","613.11"
"account_subgroup_depreciación_de_troqueles,_moldes,_matrices_y_herramental_1","l10n_mx.mx_coa","Depreciación de troqueles, moldes, matrices y herramental","613.12"
"account_subgroup_depreciación_de_equipo_de_comunicaciones_telefónicas_1","l10n_mx.mx_coa","Depreciación de equipo de comunicaciones telefónicas","613.13"
"account_subgroup_depreciación_de_equipo_de_comunicación_satelital_1","l10n_mx.mx_coa","Depreciación de equipo de comunicación satelital","613.14"
"account_subgroup_depreciación_de_equipo_de_adaptaciones_para_personas_con_capacidades_diferentes_1","l10n_mx.mx_coa","Depreciación de equipo de adaptaciones para personas con capacidades diferentes","613.15"
"account_subgroup_depreciación_de_maquinaria_y_equipo_de_generación_de_energía_de_fuentes_renovables_o_de_sistemas_de_cogeneración_de_electricidad_eficiente_1","l10n_mx.mx_coa","Depreciación de maquinaria y equipo de generación de energía de fuentes renovables o de sistemas de cogeneración de electricidad eficiente","613.16"
"account_subgroup_depreciación_de_adaptaciones_y_mejoras_1","l10n_mx.mx_coa","Depreciación de adaptaciones y mejoras","613.17"
"account_subgroup_depreciación_de_otra_maquinaria_y_equipo_1","l10n_mx.mx_coa","Depreciación de otra maquinaria y equipo","613.18"
"account_subgroup_amortización_contable","l10n_mx.mx_coa","Amortización contable","614"
"account_subgroup_amortización_de_gastos_diferidos","l10n_mx.mx_coa","Amortización de gastos diferidos","614.01"
"account_subgroup_amortización_de_gastos_pre_operativos","l10n_mx.mx_coa","Amortización de gastos pre operativos","614.02"
"account_subgroup_amortización_de_regalías,_asistencia_técnica_y_otros_gastos_diferidos","l10n_mx.mx_coa","Amortización de regalías, asistencia técnica y otros gastos diferidos","614.03"
"account_subgroup_amortización_de_activos_intangibles","l10n_mx.mx_coa","Amortización de activos intangibles","614.04"
"account_subgroup_amortización_de_gastos_de_organización","l10n_mx.mx_coa","Amortización de gastos de organización","614.05"
"account_subgroup_amortización_de_investigación_y_desarrollo_de_mercado","l10n_mx.mx_coa","Amortización de investigación y desarrollo de mercado","614.06"
"account_subgroup_amortización_de_marcas_y_patentes","l10n_mx.mx_coa","Amortización de marcas y patentes","614.07"
"account_subgroup_amortización_de_crédito_mercantil","l10n_mx.mx_coa","Amortización de crédito mercantil","614.08"
"account_subgroup_amortización_de_gastos_de_instalación","l10n_mx.mx_coa","Amortización de gastos de instalación","614.09"
"account_subgroup_amortización_de_otros_activos_diferidos","l10n_mx.mx_coa","Amortización de otros activos diferidos","614.10"
"account_group_resultado","l10n_mx.mx_coa","Resultado","7"
"account_subgroup_gastos_financieros","l10n_mx.mx_coa","Gastos financieros","701"
"account_subgroup_pérdida_cambiaria","l10n_mx.mx_coa","Pérdida cambiaria","701.01"
"account_subgroup_pérdida_cambiaria_nacional_parte_relacionada","l10n_mx.mx_coa","Pérdida cambiaria nacional parte relacionada","701.02"
"account_subgroup_pérdida_cambiaria_extranjero_parte_relacionada","l10n_mx.mx_coa","Pérdida cambiaria extranjero parte relacionada","701.03"
"account_subgroup_intereses_a_cargo_bancario_nacional","l10n_mx.mx_coa","Intereses a cargo bancario nacional","701.04"
"account_subgroup_intereses_a_cargo_bancario_extranjero","l10n_mx.mx_coa","Intereses a cargo bancario extranjero","701.05"
"account_subgroup_intereses_a_cargo_de_personas_físicas_nacional","l10n_mx.mx_coa","Intereses a cargo de personas físicas nacional","701.06"
"account_subgroup_intereses_a_cargo_de_personas_físicas_extranjero","l10n_mx.mx_coa","Intereses a cargo de personas físicas extranjero","701.07"
"account_subgroup_intereses_a_cargo_de_personas_morales_nacional","l10n_mx.mx_coa","Intereses a cargo de personas morales nacional","701.08"
"account_subgroup_intereses_a_cargo_de_personas_morales_extranjero","l10n_mx.mx_coa","Intereses a cargo de personas morales extranjero","701.09"
"account_subgroup_comisiones_bancarias","l10n_mx.mx_coa","Comisiones bancarias","701.10"
"account_subgroup_otros_gastos_financieros","l10n_mx.mx_coa","Otros gastos financieros","701.11"
"account_subgroup_productos_financieros","l10n_mx.mx_coa","Productos financieros","702"
"account_subgroup_utilidad_cambiaria","l10n_mx.mx_coa","Utilidad cambiaria","702.01"
"account_subgroup_utilidad_cambiaria_nacional_parte_relacionada","l10n_mx.mx_coa","Utilidad cambiaria nacional parte relacionada","702.02"
"account_subgroup_utilidad_cambiaria_extranjero_parte_relacionada","l10n_mx.mx_coa","Utilidad cambiaria extranjero parte relacionada","702.03"
"account_subgroup_intereses_a_favor_bancarios_nacional","l10n_mx.mx_coa","Intereses a favor bancarios nacional","702.04"
"account_subgroup_intereses_a_favor_bancarios_extranjero","l10n_mx.mx_coa","Intereses a favor bancarios extranjero","702.05"
"account_subgroup_intereses_a_favor_de_personas_físicas_nacional","l10n_mx.mx_coa","Intereses a favor de personas físicas nacional","702.06"
"account_subgroup_intereses_a_favor_de_personas_físicas_extranjero","l10n_mx.mx_coa","Intereses a favor de personas físicas extranjero","702.07"
"account_subgroup_intereses_a_favor_de_personas_morales_nacional","l10n_mx.mx_coa","Intereses a favor de personas morales nacional","702.08"
"account_subgroup_intereses_a_favor_de_personas_morales_extranjero","l10n_mx.mx_coa","Intereses a favor de personas morales extranjero","702.09"
"account_subgroup_otros_productos_financieros","l10n_mx.mx_coa","Otros productos financieros","702.10"
"account_subgroup_otros_gastos","l10n_mx.mx_coa","Otros gastos","703"
"account_subgroup_pérdida_en_venta_y/o_baja_de_terrenos","l10n_mx.mx_coa","Pérdida en venta y/o baja de terrenos","703.01"
"account_subgroup_pérdida_en_venta_y/o_baja_de_edificios","l10n_mx.mx_coa","Pérdida en venta y/o baja de edificios","703.02"
"account_subgroup_pérdida_en_venta_y/o_baja_de_maquinaria_y_equipo","l10n_mx.mx_coa","Pérdida en venta y/o baja de maquinaria y equipo","703.03"
"account_subgroup_pérdida_en_venta_y/o_baja_de_automóviles,_autobuses,_camiones_de_carga,_tractocamiones,_montacargas_y_remolques","l10n_mx.mx_coa","Pérdida en venta y/o baja de automóviles, autobuses, camiones de carga, tractocamiones, montacargas y remolques","703.04"
"account_subgroup_pérdida_en_venta_y/o_baja_de_mobiliario_y_equipo_de_oficina","l10n_mx.mx_coa","Pérdida en venta y/o baja de mobiliario y equipo de oficina","703.05"
"account_subgroup_pérdida_en_venta_y/o_baja_de_equipo_de_cómputo","l10n_mx.mx_coa","Pérdida en venta y/o baja de equipo de cómputo","703.06"
"account_subgroup_pérdida_en_venta_y/o_baja_de_equipo_de_comunicación","l10n_mx.mx_coa","Pérdida en venta y/o baja de equipo de comunicación","703.07"
"account_subgroup_pérdida_en_venta_y/o_baja_de_activos_biológicos,_vegetales_y_semovientes","l10n_mx.mx_coa","Pérdida en venta y/o baja de activos biológicos, vegetales y semovientes","703.08"
"account_subgroup_pérdida_en_venta_y/o_baja_de_otros_activos_fijos","l10n_mx.mx_coa","Pérdida en venta y/o baja de otros activos fijos","703.09"
"account_subgroup_pérdida_en_venta_y/o_baja_de_ferrocarriles","l10n_mx.mx_coa","Pérdida en venta y/o baja de ferrocarriles","703.10"
"account_subgroup_pérdida_en_venta_y/o_baja_de_embarcaciones","l10n_mx.mx_coa","Pérdida en venta y/o baja de embarcaciones","703.11"
"account_subgroup_pérdida_en_venta_y/o_baja_de_aviones","l10n_mx.mx_coa","Pérdida en venta y/o baja de aviones","703.12"
"account_subgroup_pérdida_en_venta_y/o_baja_de_troqueles,_moldes,_matrices_y_herramental","l10n_mx.mx_coa","Pérdida en venta y/o baja de troqueles, moldes, matrices y herramental","703.13"
"account_subgroup_pérdida_en_venta_y/o_baja_de_equipo_de_comunicaciones_telefónicas","l10n_mx.mx_coa","Pérdida en venta y/o baja de equipo de comunicaciones telefónicas","703.14"
"account_subgroup__pérdida_en_venta_y/o_baja_de_equipo_de_comunicación_satelital","l10n_mx.mx_coa"," Pérdida en venta y/o baja de equipo de comunicación satelital","703.15"
"account_subgroup_pérdida_en_venta_y/o_baja_de_equipo_de_adaptaciones_para_personas_con_capacidades_diferentes","l10n_mx.mx_coa","Pérdida en venta y/o baja de equipo de adaptaciones para personas con capacidades diferentes","703.16"
"account_subgroup_pérdida_en_venta_y/o_baja_de_maquinaria_y_equipo_de_generación_de_energía_de_fuentes_renovables_o_de_sistemas_de_cogeneración_de_electricidad_eficiente","l10n_mx.mx_coa","Pérdida en venta y/o baja de maquinaria y equipo de generación de energía de fuentes renovables o de sistemas de cogeneración de electricidad eficiente","703.17"
"account_subgroup_pérdida_en_venta_y/o_baja_de_otra_maquinaria_y_equipo","l10n_mx.mx_coa","Pérdida en venta y/o baja de otra maquinaria y equipo","703.18"
"account_subgroup_pérdida_por_enajenación_de_acciones","l10n_mx.mx_coa","Pérdida por enajenación de acciones","703.19"
"account_subgroup_pérdida_por_enajenación_de_partes_sociales","l10n_mx.mx_coa","Pérdida por enajenación de partes sociales","703.20"
"account_subgroup_otros_gastos_1","l10n_mx.mx_coa","Otros gastos","703.21"
"account_subgroup_otros_productos","l10n_mx.mx_coa","Otros productos","704"
"account_subgroup_ganancia_en_venta_y/o_baja_de_terrenos","l10n_mx.mx_coa","Ganancia en venta y/o baja de terrenos","704.01"
"account_subgroup_ganancia_en_venta_y/o_baja_de_edificios","l10n_mx.mx_coa","Ganancia en venta y/o baja de edificios","704.02"
"account_subgroup_ganancia_en_venta_y/o_baja_de_maquinaria_y_equipo","l10n_mx.mx_coa","Ganancia en venta y/o baja de maquinaria y equipo","704.03"
"account_subgroup_ganancia_en_venta_y/o_baja_de_automóviles,_autobuses,_camiones_de_carga,_tractocamiones,_montacargas_y_remolques","l10n_mx.mx_coa","Ganancia en venta y/o baja de automóviles, autobuses, camiones de carga, tractocamiones, montacargas y remolques","704.04"
"account_subgroup_ganancia_en_venta_y/o_baja_de_mobiliario_y_equipo_de_oficina","l10n_mx.mx_coa","Ganancia en venta y/o baja de mobiliario y equipo de oficina","704.05"
"account_subgroup_ganancia_en_venta_y/o_baja_de_equipo_de_cómputo","l10n_mx.mx_coa","Ganancia en venta y/o baja de equipo de cómputo","704.06"
"account_subgroup_ganancia_en_venta_y/o_baja_de_equipo_de_comunicación","l10n_mx.mx_coa","Ganancia en venta y/o baja de equipo de comunicación","704.07"
"account_subgroup_ganancia_en_venta_y/o_baja_de_activos_biológicos,_vegetales_y_semovientes","l10n_mx.mx_coa","Ganancia en venta y/o baja de activos biológicos, vegetales y semovientes","704.08"
"account_subgroup_ganancia_en_venta_y/o_baja_de_otros_activos_fijos","l10n_mx.mx_coa","Ganancia en venta y/o baja de otros activos fijos","704.09"
"account_subgroup_ganancia_en_venta_y/o_baja_de_ferrocarriles","l10n_mx.mx_coa","Ganancia en venta y/o baja de ferrocarriles","704.10"
"account_subgroup_ganancia_en_venta_y/o_baja_de_embarcaciones","l10n_mx.mx_coa","Ganancia en venta y/o baja de embarcaciones","704.11"
"account_subgroup_ganancia_en_venta_y/o_baja_de_aviones","l10n_mx.mx_coa","Ganancia en venta y/o baja de aviones","704.12"
"account_subgroup_ganancia_en_venta_y/o_baja_de_troqueles,_moldes,_matrices_y_herramental","l10n_mx.mx_coa","Ganancia en venta y/o baja de troqueles, moldes, matrices y herramental","704.13"
"account_subgroup_ganancia_en_venta_y/o_baja_de_equipo_de_comunicaciones_telefónicas","l10n_mx.mx_coa","Ganancia en venta y/o baja de equipo de comunicaciones telefónicas","704.14"
"account_subgroup_ganancia_en_venta_y/o_baja_de_equipo_de_comunicación_satelital","l10n_mx.mx_coa","Ganancia en venta y/o baja de equipo de comunicación satelital","704.15"
"account_subgroup_ganancia_en_venta_y/o_baja_de_equipo_de_adaptaciones_para_personas_con_capacidades_diferentes","l10n_mx.mx_coa","Ganancia en venta y/o baja de equipo de adaptaciones para personas con capacidades diferentes","704.16"
"account_subgroup_ganancia_en_venta_de_maquinaria_y_equipo_de_generación_de_energía_de_fuentes_renovables_o_de_sistemas_de_cogeneración_de_electricidad_eficiente","l10n_mx.mx_coa","Ganancia en venta de maquinaria y equipo de generación de energía de fuentes renovables o de sistemas de cogeneración de electricidad eficiente","704.17"
"account_subgroup_ganancia_en_venta_y/o_baja_de_otra_maquinaria_y_equipo","l10n_mx.mx_coa","Ganancia en venta y/o baja de otra maquinaria y equipo","704.18"
"account_subgroup_ganancia_por_enajenación_de_acciones","l10n_mx.mx_coa","Ganancia por enajenación de acciones","704.19"
"account_subgroup_ganancia_por_enajenación_de_partes_sociales","l10n_mx.mx_coa","Ganancia por enajenación de partes sociales","704.20"
"account_subgroup_ingresos_por_estímulos_fiscales","l10n_mx.mx_coa","Ingresos por estímulos fiscales","704.21"
"account_subgroup_ingresos_por_condonación_de_adeudo_1","l10n_mx.mx_coa","Ingresos por condonación de adeudo","704.22"
"account_subgroup_otros_productos_1","l10n_mx.mx_coa","Otros productos","704.23"
"account_group_cuentas_de_orden","l10n_mx.mx_coa","Cuentas de Orden","8"
"account_subgroup_ufin_del_ejercicio","l10n_mx.mx_coa","UFIN del ejercicio","801"
"account_subgroup_ufin","l10n_mx.mx_coa","UFIN","801.01"
"account_subgroup_contra_cuenta_ufin","l10n_mx.mx_coa","Contra cuenta UFIN","801.02"
"account_subgroup_cufin_del_ejercicio","l10n_mx.mx_coa","CUFIN del ejercicio","802"
"account_subgroup_cufin","l10n_mx.mx_coa","CUFIN","802.01"
"account_subgroup_contra_cuenta_cufin","l10n_mx.mx_coa","Contra cuenta CUFIN","802.02"
"account_subgroup_cufin_de_ejercicios_anteriores","l10n_mx.mx_coa","CUFIN de ejercicios anteriores","803"
"account_subgroup_cufin_de_ejercicios_anteriores_1","l10n_mx.mx_coa","CUFIN de ejercicios anteriores","803.01"
"account_subgroup_contra_cuenta_cufin_de_ejercicios_anteriores","l10n_mx.mx_coa","Contra cuenta CUFIN de ejercicios anteriores","803.02"
"account_subgroup_cufinre_del_ejercicio","l10n_mx.mx_coa","CUFINRE del ejercicio","804"
"account_subgroup_cufinre","l10n_mx.mx_coa","CUFINRE","804.01"
"account_subgroup_contra_cuenta_cufinre","l10n_mx.mx_coa","Contra cuenta CUFINRE","804.02"
"account_subgroup_cufinre_de_ejercicios_anteriores","l10n_mx.mx_coa","CUFINRE de ejercicios anteriores","805"
"account_subgroup_cufinre_de_ejercicios_anteriores_1","l10n_mx.mx_coa","CUFINRE de ejercicios anteriores","805.01"
"account_subgroup_contra_cuenta_cufinre_de_ejercicios_anteriores","l10n_mx.mx_coa","Contra cuenta CUFINRE de ejercicios anteriores","805.02"
"account_subgroup_cuca_del_ejercicio","l10n_mx.mx_coa","CUCA del ejercicio","806"
"account_subgroup_cuca","l10n_mx.mx_coa","CUCA","806.01"
"account_subgroup_contra_cuenta_cuca","l10n_mx.mx_coa","Contra cuenta CUCA","806.02"
"account_subgroup_cuca_de_ejercicios_anteriores","l10n_mx.mx_coa","CUCA de ejercicios anteriores","807"
"account_subgroup_cuca_de_ejercicios_anteriores_1","l10n_mx.mx_coa","CUCA de ejercicios anteriores","807.01"
"account_subgroup_contra_cuenta_cuca_de_ejercicios_anteriores","l10n_mx.mx_coa","Contra cuenta CUCA de ejercicios anteriores","807.02"
"account_subgroup_ajuste_anual_por_inflación_acumulable","l10n_mx.mx_coa","Ajuste anual por inflación acumulable","808"
"account_subgroup_ajuste_anual_por_inflación_acumulable_1","l10n_mx.mx_coa","Ajuste anual por inflación acumulable","808.01"
"account_subgroup_acumulación_del_ajuste_anual_inflacionario","l10n_mx.mx_coa","Acumulación del ajuste anual inflacionario","808.02"
"account_subgroup_ajuste_anual_por_inflación_deducible","l10n_mx.mx_coa","Ajuste anual por inflación deducible","809"
"account_subgroup_ajuste_anual_por_inflación_deducible_1","l10n_mx.mx_coa","Ajuste anual por inflación deducible","809.01"
"account_subgroup_deducción_del_ajuste_anual_inflacionario","l10n_mx.mx_coa","Deducción del ajuste anual inflacionario","809.02"
"account_subgroup_deducción_de_inversión","l10n_mx.mx_coa","Deducción de inversión","810"
"account_subgroup_deducción_de_inversión_1","l10n_mx.mx_coa","Deducción de inversión","810.01"
"account_subgroup_contra_cuenta_deducción_de_inversiones","l10n_mx.mx_coa","Contra cuenta deducción de inversiones","810.02"
"account_subgroup_utilidad_o_pérdida_fiscal_en_venta_y/o_baja_de_activo_fijo","l10n_mx.mx_coa","Utilidad o pérdida fiscal en venta y/o baja de activo fijo","811"
"account_subgroup_utilidad_o_pérdida_fiscal_en_venta_y/o_baja_de_activo_fijo_1","l10n_mx.mx_coa","Utilidad o pérdida fiscal en venta y/o baja de activo fijo","811.01"
"account_subgroup_contra_cuenta_utilidad_o_pérdida_fiscal_en_venta_y/o_baja_de_activo_fijo","l10n_mx.mx_coa","Contra cuenta utilidad o pérdida fiscal en venta y/o baja de activo fijo","811.02"
"account_subgroup_utilidad_o_pérdida_fiscal_en_venta_acciones_o_partes_sociales","l10n_mx.mx_coa","Utilidad o pérdida fiscal en venta acciones o partes sociales","812"
"account_subgroup_utilidad_o_pérdida_fiscal_en_venta_acciones_o_partes_sociales_1","l10n_mx.mx_coa","Utilidad o pérdida fiscal en venta acciones o partes sociales","812.01"
"account_subgroup_contra_cuenta_utilidad_o_pérdida_fiscal_en_venta_acciones_o_partes_sociales","l10n_mx.mx_coa","Contra cuenta utilidad o pérdida fiscal en venta acciones o partes sociales","812.02"
"account_subgroup_pérdidas_fiscales_pendientes_de_amortizar_actualizadas_de_ejercicios_anteriores","l10n_mx.mx_coa","Pérdidas fiscales pendientes de amortizar actualizadas de ejercicios anteriores","813"
"account_subgroup_pérdidas_fiscales_pendientes_de_amortizar_actualizadas_de_ejercicios_anteriores_1","l10n_mx.mx_coa","Pérdidas fiscales pendientes de amortizar actualizadas de ejercicios anteriores","813.01"
"account_subgroup_actualización_de_pérdidas_fiscales_pendientes_de_amortizar_de_ejercicios_anteriores","l10n_mx.mx_coa","Actualización de pérdidas fiscales pendientes de amortizar de ejercicios anteriores","813.02"
"account_subgroup_mercancías_recibidas_en_consignación","l10n_mx.mx_coa","Mercancías recibidas en consignación","814"
"account_subgroup_mercancías_recibidas_en_consignación_1","l10n_mx.mx_coa","Mercancías recibidas en consignación","814.01"
"account_subgroup_consignación_de_mercancías_recibidas","l10n_mx.mx_coa","Consignación de mercancías recibidas","814.02"
"account_subgroup_crédito_fiscal_de_iva_e_ieps_por_la_importación_de_mercancías_para_empresas_certificadas","l10n_mx.mx_coa","Crédito fiscal de IVA e IEPS por la importación de mercancías para empresas certificadas","815"
"account_subgroup_crédito_fiscal_de_iva_e_ieps_por_la_importación_de_mercancías","l10n_mx.mx_coa","Crédito fiscal de IVA e IEPS por la importación de mercancías","815.01"
"account_subgroup_importación_de_mercancías_con_aplicación_de_crédito_fiscal_de_iva_e_ieps","l10n_mx.mx_coa","Importación de mercancías con aplicación de crédito fiscal de IVA e IEPS","815.02"
"account_subgroup_crédito_fiscal_de_iva_e_ieps_por_la_importación_de_activos_fijos_para_empresas_certificadas","l10n_mx.mx_coa","Crédito fiscal de IVA e IEPS por la importación de activos fijos para empresas certificadas","816"
"account_subgroup_crédito_fiscal_de_iva_e_ieps_por_la_importación_de_activo_fijo","l10n_mx.mx_coa","Crédito fiscal de IVA e IEPS por la importación de activo fijo","816.01"
"account_subgroup_importación_de_activo_fijo_con_aplicación_de_crédito_fiscal_de_iva_e_ieps","l10n_mx.mx_coa","Importación de activo fijo con aplicación de crédito fiscal de IVA e IEPS","816.02"
"account_subgroup_otras_cuentas_de_orden","l10n_mx.mx_coa","Otras cuentas de orden","899"
"account_subgroup_otras_cuentas_de_orden_1","l10n_mx.mx_coa","Otras cuentas de orden","899.01"
"account_subgroup_contra_cuenta_otras_cuentas_de_orden","l10n_mx.mx_coa","Contra cuenta otras cuentas de orden","899.02"

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_mx.mx_coa')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="ieps_8_sale" model="account.tax.template">
            <field name="sequence" eval="0"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="active" eval="False"/>
            <field name="name">IEPS 8% VENTAS</field>
            <field name="description">IEPS 8%</field>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_ieps_8"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="include_base_amount">1</field>
            <field name="cash_basis_transition_account_id" ref="cuenta209_02"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'tag_ids': [ref('l10n_mx.tag_ieps')],
                'account_id': ref('l10n_mx.cuenta208_02'),
            }),
        ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'tag_ids': [ref('l10n_mx.tag_ieps')],
                'account_id': ref('l10n_mx.cuenta208_02'),
            }),
        ]"/>
        </record>
        <record id="ieps_8_purchase" model="account.tax.template">
            <field name="sequence" eval="1"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="name">IEPS 8% COMPRAS</field>
            <field name="description">IEPS 8%</field>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_ieps_8"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="include_base_amount">1</field>
            <field name="cash_basis_transition_account_id" ref="cuenta119_03"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'tag_ids': [ref('l10n_mx.tag_ieps')],
                    'account_id': ref('l10n_mx.cuenta118_03'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'tag_ids': [ref('l10n_mx.tag_ieps')],
                    'account_id': ref('l10n_mx.cuenta118_03'),
                }),
            ]"/>
        </record>
        <record id="ieps_25_sale" model="account.tax.template">
            <field name="sequence" eval="2"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="active" eval="False"/>
            <field name="name">IEPS 25% VENTAS</field>
            <field name="description">IEPS 25%</field>
            <field name="amount">25</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_ieps_25"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="include_base_amount">1</field>
            <field name="cash_basis_transition_account_id" ref="cuenta209_02"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'tag_ids': [ref('l10n_mx.tag_ieps')],
                'account_id': ref('l10n_mx.cuenta208_02'),
            }),
        ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'tag_ids': [ref('l10n_mx.tag_ieps')],
                'account_id': ref('l10n_mx.cuenta208_02'),
            }),
        ]"/>
        </record>
        <record id="ieps_25_purchase" model="account.tax.template">
            <field name="sequence" eval="3"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="name">IEPS 25% COMPRAS</field>
            <field name="description">IEPS 25%</field>
            <field name="amount">25</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_ieps_25"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="include_base_amount">1</field>
            <field name="cash_basis_transition_account_id" ref="cuenta119_03"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'tag_ids': [ref('l10n_mx.tag_ieps')],
                    'account_id': ref('l10n_mx.cuenta118_03'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'tag_ids': [ref('l10n_mx.tag_ieps')],
                    'account_id': ref('l10n_mx.cuenta118_03'),
                }),
            ]"/>
        </record>
        <record id="ieps_26_5_sale" model="account.tax.template">
            <field name="sequence" eval="4"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="active" eval="False"/>
            <field name="name">IEPS 26.5% VENTAS</field>
            <field name="description">IEPS 26.5%</field>
            <field name="amount">26.5</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_ieps_26_5"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="include_base_amount">1</field>
            <field name="cash_basis_transition_account_id" ref="cuenta209_02"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'tag_ids': [ref('l10n_mx.tag_ieps')],
                    'account_id': ref('l10n_mx.cuenta208_02'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'tag_ids': [ref('l10n_mx.tag_ieps')],
                    'account_id': ref('l10n_mx.cuenta208_02'),
                }),
            ]"/>
        </record>
        <record id="ieps_26_5_purchase" model="account.tax.template">
            <field name="sequence" eval="5"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="name">IEPS 26.5% COMPRAS</field>
            <field name="description">IEPS 26.5%</field>
            <field name="amount">26.5</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="include_base_amount">1</field>
            <field name="tax_group_id" ref="tax_group_ieps_26_5"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="cash_basis_transition_account_id" ref="cuenta119_03"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'tag_ids': [ref('l10n_mx.tag_ieps')],
                    'account_id': ref('l10n_mx.cuenta118_03'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'tag_ids': [ref('l10n_mx.tag_ieps')],
                    'account_id': ref('l10n_mx.cuenta118_03'),
                }),
            ]"/>
        </record>
        <record id="ieps_30_sale" model="account.tax.template">
            <field name="sequence" eval="6"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="active" eval="False"/>
            <field name="name">IEPS 30% VENTAS</field>
            <field name="description">IEPS 30%</field>
            <field name="amount">30</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_ieps_30"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="include_base_amount">1</field>
            <field name="cash_basis_transition_account_id" ref="cuenta209_02"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'tag_ids': [ref('l10n_mx.tag_ieps')],
                    'account_id': ref('l10n_mx.cuenta208_02'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'tag_ids': [ref('l10n_mx.tag_ieps')],
                    'account_id': ref('l10n_mx.cuenta208_02'),
                }),
            ]"/>
        </record>
        <record id="ieps_30_purchase" model="account.tax.template">
            <field name="sequence" eval="7"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="name">IEPS 30% COMPRAS</field>
            <field name="description">IEPS 30%</field>
            <field name="amount">30</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_ieps_30"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="include_base_amount">1</field>
            <field name="cash_basis_transition_account_id" ref="cuenta119_03"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'tag_ids': [ref('l10n_mx.tag_ieps')],
                    'account_id': ref('l10n_mx.cuenta118_03'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'tag_ids': [ref('l10n_mx.tag_ieps')],
                    'account_id': ref('l10n_mx.cuenta118_03'),
                }),
            ]"/>
        </record>
        <record id="ieps_53_sale" model="account.tax.template">
            <field name="sequence" eval="8"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="active" eval="False"/>
            <field name="name">IEPS 53% VENTAS</field>
            <field name="description">IEPS 53%</field>
            <field name="amount">53</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_ieps_53"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="include_base_amount">1</field>
            <field name="cash_basis_transition_account_id" ref="cuenta209_02"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'tag_ids': [ref('l10n_mx.tag_ieps')],
                    'account_id': ref('l10n_mx.cuenta208_02'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'tag_ids': [ref('l10n_mx.tag_ieps')],
                    'account_id': ref('l10n_mx.cuenta208_02'),
                }),
            ]"/>
        </record>
        <record id="ieps_53_purchase" model="account.tax.template">
            <field name="sequence" eval="9"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="name">IEPS 53% COMPRAS</field>
            <field name="description">IEPS 53%</field>
            <field name="amount">53</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_ieps_53"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="include_base_amount">1</field>
            <field name="cash_basis_transition_account_id" ref="cuenta119_03"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'tag_ids': [ref('l10n_mx.tag_ieps')],
                    'account_id': ref('l10n_mx.cuenta118_03'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'tag_ids': [ref('l10n_mx.tag_ieps')],
                    'account_id': ref('l10n_mx.cuenta118_03'),
                }),
            ]"/>
        </record>
        <record id="tax1" model="account.tax.template">
            <field name="sequence" eval="10"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="name">RET IVA FLETES 4%</field>
            <field name="description">RET IVA -4%</field>
            <field name="amount">-4</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_iva_ret_4"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="cash_basis_transition_account_id" ref="cuenta216_10"/>
            <field name="l10n_mx_tax_type">Tasa</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tag_diot_ret')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('cuenta216_10_20'),
            }),
        ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tag_diot_ret')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('cuenta216_10_20'),
            }),
        ]"/>
        </record>
        <record id="tax2" model="account.tax.template">
            <field name="sequence" eval="11"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="name">RET IVA ARRENDAMIENTO 10%</field>
            <field name="description">RET IVA -10%</field>
            <field name="amount">-10</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_iva_ret_10"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="cash_basis_transition_account_id" ref="cuenta216_10"/>
            <field name="l10n_mx_tax_type">Tasa</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tag_diot_ret')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('cuenta216_10_20'),
            }),
        ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tag_diot_ret')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('cuenta216_10_20'),
            }),
        ]"/>
        </record>
        <record id="tax3" model="account.tax.template">
            <field name="sequence" eval="12"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="name">RET ISR ARRENDAMIENTO 10%</field>
            <field name="description">RET ISR -10%</field>
            <field name="amount">-10</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_isr_ret_10"/>
            <field name="l10n_mx_tax_type">Tasa</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('cuenta216_03'),
            }),
        ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('cuenta216_03'),
            }),
        ]"/>
        </record>
        <record id="tax5" model="account.tax.template">
            <field name="sequence" eval="13"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="name">RET ISR HONORARIOS 10%</field>
            <field name="description">RET ISR -10%</field>
            <field name="amount">-10</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_isr_ret_10"/>
            <field name="l10n_mx_tax_type">Tasa</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('cuenta216_04'),
            }),
        ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('cuenta216_04'),
            }),
        ]"/>
        </record>
        <record id="tax7" model="account.tax.template">
            <field name="sequence" eval="14"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="name">RET IVA ARRENDAMIENTO 10.67%</field>
            <field name="description">RET IVA -10.67%</field>
            <field name="amount">-10.67</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_iva_ret_1067"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="cash_basis_transition_account_id" ref="cuenta216_10"/>
            <field name="l10n_mx_tax_type">Tasa</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tag_diot_ret')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('cuenta216_10_20'),
            }),
        ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tag_diot_ret')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('cuenta216_10_20'),
            }),
        ]"/>
        </record>
        <record id="tax8" model="account.tax.template">
            <field name="sequence" eval="15"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="name">RET IVA HONORARIOS 10.67%</field>
            <field name="description">RET IVA -10.67%</field>
            <field name="amount">-10.67</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_iva_ret_1067"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="cash_basis_transition_account_id" ref="cuenta216_10"/>
            <field name="l10n_mx_tax_type">Tasa</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tag_diot_ret')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('cuenta216_10_20'),
            }),
        ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tag_diot_ret')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('cuenta216_10_20'),
            }),
        ]"/>
        </record>
        <record id="tax9" model="account.tax.template">
            <field name="sequence" eval="16"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="name">IVA 0% VENTAS</field>
            <field name="description">IVA 0%</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_iva_0"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="cash_basis_transition_account_id" ref="cuenta209_01"/>
            <field name="l10n_mx_tax_type">Tasa</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('cuenta208_01'),
                    'tag_ids': [ref('tag_iva')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('cuenta208_01'),
                    'tag_ids': [ref('tag_iva')],
                }),
            ]"/>
        </record>
        <record id="tax12" model="account.tax.template">
            <field name="sequence" eval="17"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="name">IVA 16% VENTAS</field>
            <field name="description">IVA 16%</field>
            <field name="amount">16</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_iva_16"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="cash_basis_transition_account_id" ref="cuenta209_01"/>
            <field name="l10n_mx_tax_type">Tasa</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('cuenta208_01'),
                    'tag_ids': [ref('tag_iva')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('cuenta208_01'),
                    'tag_ids': [ref('tag_iva')],
                }),
            ]"/>
        </record>
        <record id="tax13" model="account.tax.template">
            <field name="sequence" eval="18"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="name">IVA 0% COMPRAS</field>
            <field name="description">IVA 0%</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_iva_0"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="cash_basis_transition_account_id" ref="cuenta119_01"/>
            <field name="l10n_mx_tax_type">Tasa</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tag_diot_0')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('cuenta118_01'),
            }),
        ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tag_diot_0')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('cuenta118_01'),
            }),
        ]"/>
        </record>
        <record id="tax14" model="account.tax.template">
            <field name="sequence" eval="19"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="name">IVA 16% COMPRAS</field>
            <field name="description">IVA 16%</field>
            <field name="amount">16</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_iva_16"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="cash_basis_transition_account_id" ref="cuenta119_01"/>
            <field name="l10n_mx_tax_type">Tasa</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tag_diot_16')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('cuenta118_01'),
            }),
        ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tag_diot_16')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('cuenta118_01'),
            }),
        ]"/>
        </record>
        <record id="tax16" model="account.tax.template">
            <field name="sequence" eval="20"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="name">IVA 8% COMPRAS</field>
            <field name="description">IVA 8%</field>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_iva_8"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="cash_basis_transition_account_id" ref="cuenta119_01"/>
            <field name="l10n_mx_tax_type">Tasa</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tag_diot_8')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('cuenta118_01'),
            }),
        ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tag_diot_8')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('cuenta118_01'),
            }),
        ]"/>
        </record>
        <record id="tax17" model="account.tax.template">
            <field name="sequence" eval="21"/>
            <field name="chart_template_id" ref="mx_coa"/>
            <field name="name">IVA 8% VENTAS</field>
            <field name="description">IVA 8%</field>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_iva_8"/>
            <field name="tax_exigibility">on_payment</field>
            <field name="cash_basis_transition_account_id" ref="cuenta209_01"/>
            <field name="l10n_mx_tax_type">Tasa</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'tag_ids': [ref('l10n_mx.tag_iva')],
                'account_id': ref('cuenta208_01'),
                'tag_ids': [ref('tag_iva')],
            }),
        ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'tag_ids': [ref('l10n_mx.tag_iva')],
                'account_id': ref('cuenta208_01'),
                'tag_ids': [ref('tag_iva')],
            }),
        ]"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_iva_0" model="account.tax.group">
            <field name="name">IVA 0%</field>
            <field name="country_id" ref="base.mx"/>
        </record>
        <record id="tax_group_iva_16" model="account.tax.group">
            <field name="name">IVA 16% </field>
            <field name="country_id" ref="base.mx"/>
        </record>
        <record id="tax_group_iva_8" model="account.tax.group">
            <field name="name">IVA 8%</field>
            <field name="country_id" ref="base.mx"/>
        </record>
        <record id="tax_group_iva_ret_4" model="account.tax.group">
            <field name="name">IVA Retencion 4%</field>
            <field name="country_id" ref="base.mx"/>
        </record>
        <record id="tax_group_iva_ret_10" model="account.tax.group">
            <field name="name">IVA Retencion 10%</field>
            <field name="country_id" ref="base.mx"/>
        </record>
        <record id="tax_group_iva_ret_1067" model="account.tax.group">
            <field name="name">IVA Retencion 10.67%</field>
            <field name="country_id" ref="base.mx"/>
        </record>
        <record id="tax_group_isr_ret_10" model="account.tax.group">
            <field name="name">ISR Retencion 10%</field>
            <field name="country_id" ref="base.mx"/>
        </record>
        <record id="tax_group_ieps_8" model="account.tax.group">
            <field name="name">IEPS 8%</field>
        </record>
        <record id="tax_group_ieps_25" model="account.tax.group">
            <field name="name">IEPS 25%</field>
        </record>
        <record id="tax_group_ieps_26_5" model="account.tax.group">
            <field name="name">IEPS 26.5%</field>
        </record>
        <record id="tax_group_ieps_30" model="account.tax.group">
            <field name="name">IEPS 30%</field>
        </record>
        <record id="tax_group_ieps_53" model="account.tax.group">
            <field name="name">IEPS 53%</field>
        </record>
    </data>
</odoo>

```

## File: data\fiscal_position_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="account_fiscal_position_foreign" model="account.fiscal.position.template">
            <field name="name">Foreign Customer</field>
            <field name="chart_template_id" ref="mx_coa"/>
        </record>

        <record id="account_fiscal_position0_purchase" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="l10n_mx.tax14"/>
            <field name="tax_dest_id" ref="l10n_mx.tax13"/>
            <field name="position_id" ref="account_fiscal_position_foreign"/>
        </record>

        <record id="account_fiscal_position0_sale" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="l10n_mx.tax12"/>
            <field name="tax_dest_id" ref="l10n_mx.tax9"/>
            <field name="position_id" ref="account_fiscal_position_foreign"/>
        </record>

    </data>
</odoo>

```

## File: data\l10n_mx_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
    <!--
        CoA Base
    -->
    <record id="mx_coa" model="account.chart.template">
        <field name="name">Plan de Cuentas para Mexico</field>
        <field name="bank_account_code_prefix">102.01.0</field>
        <field name="cash_account_code_prefix">101.01.0</field>
        <field name="transfer_account_code_prefix">102.01.01</field>
        <field name="code_digits">3</field>
        <field name="currency_id" ref="base.MXN"/>
        <field name="use_anglo_saxon" eval="True"/>
        <field name="country_id" ref="base.mx"/>
    </record>
    </data>
</odoo>

```

## File: data\l10n_mx_chart_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!--
        CoA setting accounts
    -->
    <record id="mx_coa" model="account.chart.template">
        <field name="property_account_receivable_id" ref="cuenta105_01"/>
        <field name="property_account_payable_id" ref="cuenta201_01"/>
        <field name="property_account_expense_categ_id" ref="cuenta601_84"/>
        <field name="property_account_income_categ_id" ref="cuenta401_01"/>
        <field name="property_stock_account_input_categ_id" ref="cuenta205_06_01"/>
        <field name="property_stock_account_output_categ_id" ref="cuenta107_05_01"/>
        <field name="property_stock_valuation_account_id" ref="cuenta115_01"/>
        <field name="property_cash_basis_base_account_id" ref="cuenta801_01_99"/>
        <field name="income_currency_exchange_account_id" ref="cuenta702_01"/>
        <field name="expense_currency_exchange_account_id" ref="cuenta701_01"/>
        <field name="default_pos_receivable_account_id" ref="cuenta105_02"/>
        <field name="account_journal_early_pay_discount_loss_account_id" ref="cuenta9993"/>
        <field name="account_journal_early_pay_discount_gain_account_id" ref="cuenta9994"/>
    </record>
</odoo>

```

## File: data\l10n_mx_uom.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <!-- UOM Categories -->
    <record id="product_uom_categ_service" model="uom.category">
        <field name="name">Service</field>
    </record>

    <!-- UOM.UOM -->
    <!-- SERVICE -->
    <record id="product_uom_service_unit" model="uom.uom">
        <field name="category_id" ref="product_uom_categ_service"/>
        <field name="name">Service Unit</field>
        <field name="factor" eval="1"/>
        <field name="uom_type">reference</field>
    </record>
    <record id="product_uom_activity" model="uom.uom">
        <field name="category_id" ref="product_uom_categ_service"/>
        <field name="name">Activity</field>
        <field name="factor" eval="1"/>
        <field name="uom_type">smaller</field>
    </record>
    <record id="product_uom_job" model="uom.uom">
        <field name="category_id" ref="product_uom_categ_service"/>
        <field name="name">Job</field>
        <field name="factor" eval="1"/>
        <field name="uom_type">smaller</field>
    </record>
</odoo>

```

## File: data\res_bank_data.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <record id='acc_bank_002_BANAMEX' model='res.bank'>
        <field name='name'>BANAMEX</field>
        <field name='l10n_mx_edi_code'>002</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_006_BANCOMEXT' model='res.bank'>
        <field name='name'>BANCOMEXT</field>
        <field name='l10n_mx_edi_code'>006</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_009_BANOBRAS' model='res.bank'>
        <field name='name'>BANOBRAS</field>
        <field name='l10n_mx_edi_code'>009</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_012_BBVA_BANCOMER' model='res.bank'>
        <field name='name'>BBVA BANCOMER</field>
        <field name='l10n_mx_edi_code'>012</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_014_SANTANDER' model='res.bank'>
        <field name='name'>SANTANDER</field>
        <field name='l10n_mx_edi_code'>014</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_019_BANJERCITO' model='res.bank'>
        <field name='name'>BANJERCITO</field>
        <field name='l10n_mx_edi_code'>019</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_021_HSBC' model='res.bank'>
        <field name='name'>HSBC</field>
        <field name='l10n_mx_edi_code'>021</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_030_BAJIO' model='res.bank'>
        <field name='name'>BAJIO</field>
        <field name='l10n_mx_edi_code'>030</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_036_INBURSA' model='res.bank'>
        <field name='name'>INBURSA</field>
        <field name='l10n_mx_edi_code'>036</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_037_INTERACCIONES' model='res.bank'>
        <field name='name'>INTERACCIONES</field>
        <field name='l10n_mx_edi_code'>037</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_042_MIFEL' model='res.bank'>
        <field name='name'>MIFEL</field>
        <field name='l10n_mx_edi_code'>042</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_044_SCOTIABANK' model='res.bank'>
        <field name='name'>SCOTIABANK</field>
        <field name='l10n_mx_edi_code'>044</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_058_BANREGIO' model='res.bank'>
        <field name='name'>BANREGIO</field>
        <field name='l10n_mx_edi_code'>058</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_059_INVEX' model='res.bank'>
        <field name='name'>INVEX</field>
        <field name='l10n_mx_edi_code'>059</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_060_BANSI' model='res.bank'>
        <field name='name'>BANSI</field>
        <field name='l10n_mx_edi_code'>060</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_062_AFIRME' model='res.bank'>
        <field name='name'>AFIRME</field>
        <field name='l10n_mx_edi_code'>062</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_072_BANORTE' model='res.bank'>
        <field name='name'>BANORTE/IXE</field>
        <field name='l10n_mx_edi_code'>072</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_102_THE_ROYAL_BANK' model='res.bank'>
        <field name='name'>THE ROYAL BANK</field>
        <field name='l10n_mx_edi_code'>102</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_103_AMERICAN_EXPRESS' model='res.bank'>
        <field name='name'>AMERICAN EXPRESS</field>
        <field name='l10n_mx_edi_code'>103</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_106_BAMSA' model='res.bank'>
        <field name='name'>BAMSA</field>
        <field name='l10n_mx_edi_code'>106</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_108_TOKYO' model='res.bank'>
        <field name='name'>TOKYO</field>
        <field name='l10n_mx_edi_code'>108</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_110_JP_MORGAN' model='res.bank'>
        <field name='name'>JP MORGAN</field>
        <field name='l10n_mx_edi_code'>110</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_112_BMONEX' model='res.bank'>
        <field name='name'>BMONEX</field>
        <field name='l10n_mx_edi_code'>112</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_113_VE_POR_MAS' model='res.bank'>
        <field name='name'>VE POR MAS</field>
        <field name='l10n_mx_edi_code'>113</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_124_DEUTSCHE' model='res.bank'>
        <field name='name'>DEUTSCHE</field>
        <field name='l10n_mx_edi_code'>124</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_126_CREDIT_SUISSE' model='res.bank'>
        <field name='name'>CREDIT SUISSE</field>
        <field name='l10n_mx_edi_code'>126</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_127_AZTECA' model='res.bank'>
        <field name='name'>AZTECA</field>
        <field name='l10n_mx_edi_code'>127</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_128_AUTOFIN' model='res.bank'>
        <field name='name'>AUTOFIN</field>
        <field name='l10n_mx_edi_code'>128</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_129_BARCLAYS' model='res.bank'>
        <field name='name'>BARCLAYS</field>
        <field name='l10n_mx_edi_code'>129</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_130_COMPARTAMOS' model='res.bank'>
        <field name='name'>COMPARTAMOS</field>
        <field name='l10n_mx_edi_code'>130</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_131_BANCO_FAMSA' model='res.bank'>
        <field name='name'>BANCO FAMSA</field>
        <field name='l10n_mx_edi_code'>131</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_132_BMULTIVA' model='res.bank'>
        <field name='name'>BMULTIVA</field>
        <field name='l10n_mx_edi_code'>132</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_133_ACTINVER' model='res.bank'>
        <field name='name'>ACTINVER</field>
        <field name='l10n_mx_edi_code'>133</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_135_NAFIN' model='res.bank'>
        <field name='name'>NAFIN</field>
        <field name='l10n_mx_edi_code'>135</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_136_INTERBANCO' model='res.bank'>
        <field name='name'>INTERCAM BANCO</field>
        <field name='l10n_mx_edi_code'>136</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_137_BANCOPPEL' model='res.bank'>
        <field name='name'>BANCOPPEL</field>
        <field name='l10n_mx_edi_code'>137</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_138_ABC_CAPITAL' model='res.bank'>
        <field name='name'>ABC CAPITAL</field>
        <field name='l10n_mx_edi_code'>138</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_139_UBS_BANK' model='res.bank'>
        <field name='name'>UBS BANK</field>
        <field name='l10n_mx_edi_code'>139</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_140_CONSUBANCO' model='res.bank'>
        <field name='name'>CONSUBANCO</field>
        <field name='l10n_mx_edi_code'>140</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_141_VOLKSWAGEN' model='res.bank'>
        <field name='name'>VOLKSWAGEN</field>
        <field name='l10n_mx_edi_code'>141</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_143_CIBANCO' model='res.bank'>
        <field name='name'>CIBANCO</field>
        <field name='l10n_mx_edi_code'>143</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_145_BBASE' model='res.bank'>
        <field name='name'>BBASE</field>
        <field name='l10n_mx_edi_code'>145</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_166_BANSEFI' model='res.bank'>
        <field name='name'>BANSEFI</field>
        <field name='l10n_mx_edi_code'>166</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_168_HIPOTECARIA_FEDERAL' model='res.bank'>
        <field name='name'>HIPOTECARIA FEDERAL</field>
        <field name='l10n_mx_edi_code'>168</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_600_MONEXCB' model='res.bank'>
        <field name='name'>MONEXCB</field>
        <field name='l10n_mx_edi_code'>600</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_601_GBM' model='res.bank'>
        <field name='name'>GBM</field>
        <field name='l10n_mx_edi_code'>601</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_602_MASARI' model='res.bank'>
        <field name='name'>MASARI</field>
        <field name='l10n_mx_edi_code'>602</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_605_VALUE' model='res.bank'>
        <field name='name'>VALUE</field>
        <field name='l10n_mx_edi_code'>605</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_606_ESTRUCTURADORES' model='res.bank'>
        <field name='name'>ESTRUCTURADORES</field>
        <field name='l10n_mx_edi_code'>606</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_607_TIBER' model='res.bank'>
        <field name='name'>TIBER</field>
        <field name='l10n_mx_edi_code'>607</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_608_VECTOR' model='res.bank'>
        <field name='name'>VECTOR</field>
        <field name='l10n_mx_edi_code'>608</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_610_BB' model='res.bank'>
        <field name='name'>B&amp;B</field>
        <field name='l10n_mx_edi_code'>610</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_614_ACCIVAL' model='res.bank'>
        <field name='name'>ACCIVAL</field>
        <field name='l10n_mx_edi_code'>614</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_615_MERRILL_LYNCH' model='res.bank'>
        <field name='name'>MERRILL LYNCH</field>
        <field name='l10n_mx_edi_code'>615</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_616_FINAMEX' model='res.bank'>
        <field name='name'>FINAMEX</field>
        <field name='l10n_mx_edi_code'>616</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_617_VALMEX' model='res.bank'>
        <field name='name'>VALMEX</field>
        <field name='l10n_mx_edi_code'>617</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_618_UNICA' model='res.bank'>
        <field name='name'>UNICA</field>
        <field name='l10n_mx_edi_code'>618</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_619_MAPFRE' model='res.bank'>
        <field name='name'>MAPFRE</field>
        <field name='l10n_mx_edi_code'>619</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_620_PROFUTURO' model='res.bank'>
        <field name='name'>PROFUTURO</field>
        <field name='l10n_mx_edi_code'>620</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_621_CB_ACTINVER' model='res.bank'>
        <field name='name'>CB ACTINVER</field>
        <field name='l10n_mx_edi_code'>621</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_622_OACTIN' model='res.bank'>
        <field name='name'>OACTIN</field>
        <field name='l10n_mx_edi_code'>622</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_623_SKANDIA' model='res.bank'>
        <field name='name'>SKANDIA</field>
        <field name='l10n_mx_edi_code'>623</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_626_CBDEUTSCHE' model='res.bank'>
        <field name='name'>CBDEUTSCHE</field>
        <field name='l10n_mx_edi_code'>626</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_627_ZURICH' model='res.bank'>
        <field name='name'>ZURICH</field>
        <field name='l10n_mx_edi_code'>627</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_628_ZURICHVI' model='res.bank'>
        <field name='name'>ZURICHVI</field>
        <field name='l10n_mx_edi_code'>628</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_629_SU_CASITA' model='res.bank'>
        <field name='name'>SU CASITA</field>
        <field name='l10n_mx_edi_code'>629</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_630_CB_INTERCAM' model='res.bank'>
        <field name='name'>CB INTERCAM</field>
        <field name='l10n_mx_edi_code'>630</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_631_CI_BOLSA_CI' model='res.bank'>
        <field name='name'>CI BOLSA</field>
        <field name='l10n_mx_edi_code'>631</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_632_BULLTICK_CB' model='res.bank'>
        <field name='name'>BULLTICK CB</field>
        <field name='l10n_mx_edi_code'>632</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_633_STERLING' model='res.bank'>
        <field name='name'>STERLING</field>
        <field name='l10n_mx_edi_code'>633</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_634_FINCOMUN' model='res.bank'>
        <field name='name'>FINCOMUN</field>
        <field name='l10n_mx_edi_code'>634</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_636_HDI_SEGUROS' model='res.bank'>
        <field name='name'>HDI SEGUROS</field>
        <field name='l10n_mx_edi_code'>636</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_637_ORDER' model='res.bank'>
        <field name='name'>ORDER</field>
        <field name='l10n_mx_edi_code'>637</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_638_AKALA' model='res.bank'>
        <field name='name'>NU</field>
        <field name='l10n_mx_edi_code'>638</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_640_CB_JPMORGAN' model='res.bank'>
        <field name='name'>CB JPMORGAN</field>
        <field name='l10n_mx_edi_code'>640</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_642_REFORMA' model='res.bank'>
        <field name='name'>REFORMA</field>
        <field name='l10n_mx_edi_code'>642</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_646_STP' model='res.bank'>
        <field name='name'>STP</field>
        <field name='l10n_mx_edi_code'>646</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_647_TELECOMM' model='res.bank'>
        <field name='name'>TELECOMM</field>
        <field name='l10n_mx_edi_code'>647</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_648_EVERCORE' model='res.bank'>
        <field name='name'>EVERCORE</field>
        <field name='l10n_mx_edi_code'>648</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_649_SKANDIA' model='res.bank'>
        <field name='name'>SKANDIA</field>
        <field name='l10n_mx_edi_code'>649</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_651_SEGMTY' model='res.bank'>
        <field name='name'>SEGMTY</field>
        <field name='l10n_mx_edi_code'>651</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_652_ASEA' model='res.bank'>
        <field name='name'>ASEA</field>
        <field name='l10n_mx_edi_code'>652</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_653_KUSPIT' model='res.bank'>
        <field name='name'>KUSPIT</field>
        <field name='l10n_mx_edi_code'>653</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_655_SOFIEXPRESS' model='res.bank'>
        <field name='name'>SOFIEXPRESS</field>
        <field name='l10n_mx_edi_code'>655</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_656_UNAGRA' model='res.bank'>
        <field name='name'>UNAGRA</field>
        <field name='l10n_mx_edi_code'>656</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_659_OPCIONES_EMPRESARIALES_DEL_NOROESTE' model='res.bank'>
        <field name='name'>OPCIONES EMPRESARIALES DEL NOROESTE</field>
        <field name='l10n_mx_edi_code'>659</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_901_CLS' model='res.bank'>
        <field name='name'>CLS</field>
        <field name='l10n_mx_edi_code'>901</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_902_INDEVAL' model='res.bank'>
        <field name='name'>INDEVAL</field>
        <field name='l10n_mx_edi_code'>902</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_670_LIBERTAD' model='res.bank'>
        <field name='name'>LIBERTAD</field>
        <field name='l10n_mx_edi_code'>670</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_999_NA' model='res.bank'>
        <field name='name'>N/A</field>
        <field name='l10n_mx_edi_code'>999</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_154_BANCO_FINTERRA' model='res.bank'>
        <field name='name'>BANCO FINTERRA</field>
        <field name='l10n_mx_edi_code'>154</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_160_BANCO_S3' model='res.bank'>
        <field name='name'>BANCO S3</field>
        <field name='l10n_mx_edi_code'>160</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_152_BANCREA' model='res.bank'>
        <field name='name'>BANCREA</field>
        <field name='l10n_mx_edi_code'>152</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_159_BANK_OF_CHINA' model='res.bank'>
        <field name='name'>BANK OF CHINA</field>
        <field name='l10n_mx_edi_code'>159</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_147_BANKAOOL' model='res.bank'>
        <field name='name'>BANKAOOL</field>
        <field name='l10n_mx_edi_code'>147</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_151_DONDÉ' model='res.bank'>
        <field name='name'>DONDÉ</field>
        <field name='l10n_mx_edi_code'>151</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_149_FORJADORES' model='res.bank'>
        <field name='name'>FORJADORES</field>
        <field name='l10n_mx_edi_code'>149</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_155_ICBC' model='res.bank'>
        <field name='name'>ICBC</field>
        <field name='l10n_mx_edi_code'>155</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_150_INMOBILIARIO' model='res.bank'>
        <field name='name'>INMOBILIARIO</field>
        <field name='l10n_mx_edi_code'>150</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_136_INTERCAM_BANCO' model='res.bank'>
        <field name='name'>INTERCAM BANCO</field>
        <field name='l10n_mx_edi_code'>136</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_158_MIZUHO_BANK' model='res.bank'>
        <field name='name'>MIZUHO BANK</field>
        <field name='l10n_mx_edi_code'>158</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_148_PAGATODO' model='res.bank'>
        <field name='name'>PAGATODO</field>
        <field name='l10n_mx_edi_code'>148</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_153_PROGRESO' model='res.bank'>
        <field name='name'>PROGRESO</field>
        <field name='l10n_mx_edi_code'>153</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_156_SABADELL' model='res.bank'>
        <field name='name'>SABADELL</field>
        <field name='l10n_mx_edi_code'>156</field>
        <field name='country' ref="base.mx"/>
    </record>
    <record id='acc_bank_157_SHINHAN' model='res.bank'>
        <field name='name'>SHINHAN</field>
        <field name='l10n_mx_edi_code'>157</field>
        <field name='country' ref="base.mx"/>
    </record>
</odoo>

```

## File: migrations\2.1\post-migrate.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, Command, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    # Migrate wrong data tag on account cuenta102_02
    debit_tag = env.ref('l10n_mx.tag_debit_balance_account')
    credit_tag = env.ref('l10n_mx.tag_credit_balance_account')
    account_102_ids = env['ir.model.data'].search([
        ('name', 'ilike', '%_cuenta102_02'),
        ('model', '=', 'account.account'),
    ]).mapped('res_id')
    accounts_102 = env['account.account'].search([('id', 'in', account_102_ids)])
    accounts_102.tag_ids = [Command.unlink(credit_tag.id), Command.link(debit_tag.id)]

```

## File: models\account_account.py

```python
from odoo import api, Command, models


class AccountAccount(models.Model):
    _inherit = 'account.account'

    @api.model_create_multi
    def create(self, vals_list):
        # EXTENDS account - ensure there is a tag on created MX accounts
        # The computation is a bit naive and might not be correct in all cases.
        accounts = super().create(vals_list)
        debit_tag = self.env.ref('l10n_mx.tag_debit_balance_account')
        credit_tag = self.env.ref('l10n_mx.tag_credit_balance_account')
        mx_account_no_tags = accounts.filtered(lambda a: a.company_id.country_code == 'MX' and not a.tag_ids & (credit_tag + debit_tag))
        DEBIT_CODES = ['1', '5', '6', '7']  # all other codes are considered "credit"
        for account in mx_account_no_tags:
            tag_id = debit_tag.id if account.code[0] in DEBIT_CODES else credit_tag.id
            account.tag_ids = [Command.link(tag_id)]
        return accounts

```

## File: models\account_tax.py

```python
# coding: utf-8
from odoo import models, fields


class AccountTaxTemplate(models.Model):
    _inherit = 'account.tax.template'

    l10n_mx_tax_type = fields.Selection(
        selection=[
            ('Tasa', "Tasa"),
            ('Cuota', "Cuota"),
            ('Exento', "Exento"),
        ],
        string="Factor Type",
        default='Tasa',
        help="The CFDI version 3.3 have the attribute 'TipoFactor' in the tax lines. In it is indicated the factor "
             "type that is applied to the base of the tax.")

    def _get_tax_vals(self, company, tax_template_to_tax):
        # OVERRIDE
        res = super()._get_tax_vals(company, tax_template_to_tax)
        res['l10n_mx_tax_type'] = self.l10n_mx_tax_type
        return res


class AccountTax(models.Model):
    _inherit = 'account.tax'

    l10n_mx_tax_type = fields.Selection(
        selection=[
            ('Tasa', "Tasa"),
            ('Cuota', "Cuota"),
            ('Exento', "Exento"),
        ],
        string="Factor Type",
        default='Tasa',
        help="The CFDI version 3.3 have the attribute 'TipoFactor' in the tax lines. In it is indicated the factor "
             "type that is applied to the base of the tax.")

```

## File: models\chart_template.py

```python
# coding: utf-8
# Copyright 2016 Vauxoo (https://www.vauxoo.com) <info@vauxoo.com>
# License LGPL-3.0 or later (http://www.gnu.org/licenses/lgpl).

from odoo import models, api, _


class AccountChartTemplate(models.Model):
    _inherit = "account.chart.template"

    def _load(self, company):
        res = super()._load(company)
        if company.chart_template_id == self.env.ref('l10n_mx.mx_coa'):
            company.write({
                'account_sale_tax_id': self.env.ref(f'l10n_mx.{company.id}_tax12'),
                'account_purchase_tax_id': self.env.ref(f'l10n_mx.{company.id}_tax14'),
            })
        return res

    @api.model
    def generate_journals(self, acc_template_ref, company, journals_dict=None):
        """Set the tax_cash_basis_journal_id on the company"""
        res = super(AccountChartTemplate, self).generate_journals(
            acc_template_ref, company, journals_dict=journals_dict)
        if not self == self.env.ref('l10n_mx.mx_coa'):
            return res
        journal_basis = self.env['account.journal'].search([
            ('company_id', '=', company.id),
            ('type', '=', 'general'),
            ('code', '=', 'CBMX')], limit=1)
        company.write({'tax_cash_basis_journal_id': journal_basis.id})
        return res

    def _prepare_all_journals(self, acc_template_ref, company, journals_dict=None):
        """Create the tax_cash_basis_journal_id"""
        res = super(AccountChartTemplate, self)._prepare_all_journals(
            acc_template_ref, company, journals_dict=journals_dict)
        if not self == self.env.ref('l10n_mx.mx_coa'):
            return res
        account = acc_template_ref.get(self.env.ref('l10n_mx.cuenta118_01').id)
        res.append({
            'type': 'general',
            'name': _('Effectively Paid'),
            'code': 'CBMX',
            'company_id': company.id,
            'default_account_id': account,
            'show_on_dashboard': True,
        })
        return res

```

## File: models\res_bank.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class Bank(models.Model):
    _inherit = "res.bank"

    l10n_mx_edi_code = fields.Char(
        "ABM Code",
        help="Three-digit number assigned by the ABM to identify banking "
        "institutions (ABM is an acronym for Asociación de Bancos de México)")


class ResPartnerBank(models.Model):
    _inherit = "res.partner.bank"

    l10n_mx_edi_clabe = fields.Char(
        "CLABE", help="Standardized banking cipher for Mexico. More info "
        "wikipedia.org/wiki/CLABE")

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    module_l10n_mx_edi = fields.Boolean('Mexican Electronic Invoicing')

```

## File: models\__init__.py

```python
# coding: utf-8
# Copyright 2016 Vauxoo (https://www.vauxoo.com) <info@vauxoo.com>
# License LGPL-3.0 or later (http://www.gnu.org/licenses/lgpl).

from . import account_account
from . import account_tax
from . import res_bank
from . import res_config_settings
from . import chart_template

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="3.61" y="7.25" width="52.63" height="32.5" maskUnits="userSpaceOnUse">
      <rect x="6.3" y="7.53" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="800" height="456" transform="translate(3.61 7.25) scale(0.07 0.07)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAyAAAAHuCAYAAABnDWiFAAAACXBIWXMAAKg+AACoPgE2//rVAAAgAElEQVR4XuzdeXhcd2Hv//eZOWd2aTQa7Yu1WJblfYu3xFmcBEJCCmUrECht723pcht+t7R0fwp0+91e2l5Kf5dCgbAnKTQJCSEhkD1OvO+2bEu2tVi7NBppNPuZ5ffHOHJMQhwoDNvn9Tx64khnzszoH8/b383g924sIiLy/WSSNG/cycjv/f2VrhSRX3Dz7/kA0bv/FSehK10qIr+4nnFc6QoREREREZEfFQWIiIiIiIiUjQJERERERETKRgEiIiIiIiJlowAREREREZGyUYCIiIiIiEjZKEBERERERKRsFCAiIiIiIlI2ChARERERESkbBYiIiIiIiJSNAkRERERERMpGASIiIiIiImWjABERERERkbJRgIiIiIiISNkoQEREREREpGwUICIiIiIiUjYKEBERERERKRsFiIiIiIiIlI0CREREREREykYBIiIiIiIiZaMAERERERGRslGAiIiIiIhI2ShARERERESkbBQgIiIiIiJSNgoQEREREREpGwWIiIiIiIiUjQJERERERETKRgEiIiIiIiJlowAREREREZGyUYCIiIiIiEjZKEBERERERKRsFCAiIiIiIlI2ChARERERESkbBYiIiIiIiJSNAkRERERERMpGASIiIiIiImWjABERERERkbJRgIiIiIiISNkoQEREREREpGwUICIiIiIiUjYKEBERERERKRsFiIiIiIiIlI0CREREREREykYBIiIiIiIiZaMAERERERGRslGAiIiIiIhI2ShARERERESkbBQgIiIiIiJSNgoQEREREREpGwWIiIiIiIiUjQJERERERETKRgEiIiIiIiJlowAREREREZGyUYCIiIiIiEjZKEBERERERKRsFCAiIiIiIlI2ChARERERESkbBYiIiIiIiJSNAkRERERERMpGASIiIiIiImWjABERERERkbJRgIiIiIiISNkoQEREREREpGwUICIiIiIiUjYKEBERERERKRsFiIiIiIiIlI0CREREREREykYBIiIiIiIiZaMAERERERGRslGAiIiIiIhI2ShARERERESkbBQgIiIiIiJSNgoQEREREREpGwWIiIiIiIiUjQJERERERETKRgEiIiIiIiJlowAREREREZGyUYCIiIiIiEjZKEBERERERKRsFCAiIiIiIlI2ChARERERESkbBYiIiIiIiJSNAkRERERERMpGASIiIiIiImWjABERERERkbJRgIiIiIiISNkoQEREREREpGwUICIiIiIiUjYKEBERERERKRsFiIiIiIiIlI0CREREREREykYBIiIiIiIiZaMAERERERGRslGAiIiIiIhI2ShARERERESkbBQgIiIiIiJSNgoQEREREREpGwWIiIiIiIiUjQJERERERETKRgEiIiIiIiJlowAREREREZGyUYCIiIiIiEjZKEBERERERKRsFCAiIiIiIlI2ChARERERESkbBYiIiIiIiJSNAkRERERERMpGASIiIiIiImWjABERERERkbJRgIiIiIiISNkoQEREREREpGwUICIiIiIiUjYKEBERERERKRsFiIiIiIiIlI0CREREREREykYBIiIiIiIiZaMAERERERGRslGAiIiIiIhI2ShARERERESkbBQgIiIiIiJSNgoQEREREREpGwWIiIiIiIiUjQJERERERETKRgEiIiIiIiJlowAREREREZGyUYCIiIiIiEjZKEBERERERKRsFCAiIiIiIlI2ChARERERESkbBYiIiIiIiJSNAkRERERERMpGASIiIiIiImWjABERERERkbJRgIiIiIiISNkoQEREREREpGwUICIiIiIiUjYKEBERERERKRsFiIiIiIiIlI0CREREREREykYBIiIiIiIiZaMAERERERGRslGAiIiIiIhI2ShARERERESkbBQgIiIiIiJSNgoQEREREREpGwWIiIiIiIiUjXmlC0RERF7N1OwsszMzRCbGAfBUBgEI19bi83qprqzANK1Xu4WIiPwCUYCIiMgPbGp2lm8/9BDPPv0E9sQxGDq1+LN0HgxPCIBEZQ3VAROrYS2N9c3UNzexbPlylq9eTWM4hNvj+35PISIiP6cUICIi8gM5eOQIn/+738XVf4B1Zo5g0MLsdmO9JCbsdBLbLpBfmCIxA7Njx5h3mJzMO3guU8TwhHA3NeDvupZtV13FqvXrWdq9giqf+1WeWUREfh5oDYiIiIiIiJSNRkBEROSKMukkqYITVyHFE//wHpYP9BJuC9C942oqQgHyLg9u6/J/08rYBXKxBbKpDMlojNmxCMnoOOnZODBFdmGKyWePsfsZF48YVbibGliy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XMBph+pmTNVNMK4AERE5+yiAiIicxTq8Pfz43Eu5fta845880D6yh+ONl2EwSACZLhc937yXZ7ZfzN3vrqRhz0ZjaGHQR8PWdXxyz0Zeveu3xuDCyz46dN0LEBGRs5iWYImInMUyXa7RwwdAeRn5f/gdETrJ/8PvonM9hjOZzFw/ax71tz2I54Gnufe6b4HV2KHqic9+d2hquoiIyABVQERE5MQ+83HGjn1n1PBxrEyXi+8tvoQLi0pZV1Vx4mAjIiJnNQUQERE5udMIH8MtKilU5UNERE5IS7BERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERERCRmFEBERERE5P9vv44FAAAAAAb5W89iV1kEGwEBAAA2AgIAAGwEBAAA2AgIAACwERAAAGAjIAAAwEZAAACAjYAAAAAbAQEAADYCAgAAbAQEAADYCAgAALAREAAAYBNVYX2VR39epgAAAABJRU5ErkJggg=="/>
    </g>
  </g>
</svg>

```

## File: views\account_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_account_journal_form_inherit" model="ir.ui.view">
        <field name="name">account.journal.form</field>
        <field name="model">account.journal</field>
        <field name="inherit_id" ref="account.view_account_journal_form"/>
        <field name="arch" type="xml">
            <field name="restrict_mode_hash_table" position="attributes">
                <attribute name="groups" eval=""/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\partner_view.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<odoo>
    <record model="ir.ui.view" id="view_res_partner_inherit_l10n_mx_edi_bank">
        <field name="name">view.res.partner.inherit.l10n_mx_edi_bank</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='bank_ids']//field[@name='acc_number']" position="after">
                <field name="l10n_mx_edi_clabe"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_bank_view.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<odoo>
    <record model="ir.ui.view" id="view_res_bank_inherit_l10n_mx_edi_bank">
        <field name="name">view.res.bank.inherit.l10n_mx_edi_bank</field>
        <field name="model">res.bank</field>
        <field name="inherit_id" ref="base.view_res_bank_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='bic']" position="after">
                <field name="l10n_mx_edi_code"/>
            </xpath>
        </field>
    </record>

    <record model="ir.ui.view" id="view_partner_bank_form_l10n_mx_edi_bank">
        <field name="name">view.partner.bank.form.mx.inherit</field>
        <field name="model">res.partner.bank</field>
        <field name="inherit_id" ref="base.view_partner_bank_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='bank_id']" position="after">
                <field name="l10n_mx_edi_clabe" colspan="2"/>
            </xpath>
        </field>
    </record>

    <record model="ir.ui.view" id="view_partner_bank_tree_l10n_mx_edi_bank">
        <field name="name">view.partner.bank.tree.mx.inherit</field>
        <field name="model">res.partner.bank</field>
        <field name="inherit_id" ref="base.view_partner_bank_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='bank_name']" position="after">
                <field name="l10n_mx_edi_clabe"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.l10n.mx</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr=".//div[@id='invoicing_settings']" position="inside">
                <div class="col-12 col-lg-6 o_setting_box" id="electronic_invoices_mx" attrs="{'invisible': [('country_code', '!=', 'MX')]}">
                    <div class="o_setting_left_pane">
                        <field name="module_l10n_mx_edi" class="oe_inline" widget="upgrade_boolean"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="module_l10n_mx_edi"/>
                        <div class="text-muted">
                            Create your electronic invoices automatically (CFDI format)
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

