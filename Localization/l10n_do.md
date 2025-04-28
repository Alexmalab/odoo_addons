# Odoo Module: l10n_do

Category: Localization

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

# Author: Gustavo Valverde <gvalverde@iterativo.do> iterativo | Consultores
# Contributors: Edser Solis - iterativo

# Odoo 8.0 author: Eneldo Serrata <eneldo@marcos.do>
# (Marcos Organizador de Negocios SRL..)
# Odoo 7.0 author: Jose Ernesto Mendez <tecnologia@obsdr.com>
# (Open Business Solutions SRL.)

# Copyright (c) 2016 - Present | iterativo, SRL. - http://iterativo.do
# All rights reserved.

{
    'name': 'Dominican Republic - Accounting',
    'version': '2.0',
    'category': 'Localization',
    'description': """

Localization Module for Dominican Republic
===========================================

Catálogo de Cuentas e Impuestos para República Dominicana, Compatible para
**Internacionalización** con **NIIF** y alineado a las normas y regulaciones
de la Dirección General de Impuestos Internos (**DGII**).

**Este módulo consiste de:**

- Catálogo de Cuentas Estándar (alineado a DGII y NIIF)
- Catálogo de Impuestos con la mayoría de Impuestos Preconfigurados
        - ITBIS para compras y ventas
        - Retenciones de ITBIS
        - Retenciones de ISR
        - Grupos de Impuestos y Retenciones:
                - Telecomunicaiones
                - Proveedores de Materiales de Construcción
                - Personas Físicas Proveedoras de Servicios
        - Otros impuestos
- Secuencias Preconfiguradas para manejo de todos los NCF
        - Facturas con Valor Fiscal (para Ventas)
        - Facturas para Consumidores Finales
        - Notas de Débito y Crédito
        - Registro de Proveedores Informales
        - Registro de Ingreso Único
        - Registro de Gastos Menores
        - Gubernamentales
- Posiciones Fiscales para automatización de impuestos y retenciones
        - Cambios de Impuestos a Exenciones (Ej. Ventas al Estado)
        - Cambios de Impuestos a Retenciones (Ej. Compra Servicios al Exterior)
        - Entre otros

**Nota:**
Esta localización, aunque posee las secuencias para NCF, las mismas no pueden
ser utilizadas sin la instalación de módulos de terceros o desarrollo
adicional.

Estructura de Codificación del Catálogo de Cuentas:
===================================================

**Un dígito** representa la categoría/tipo de cuenta del del estado financiero.
**1** - Activo        **4** - Cuentas de Ingresos y Ganancias
**2** - Pasivo        **5** - Costos, Gastos y Pérdidas
**3** - Capital       **6** - Cuentas Liquidadoras de Resultados

**Dos dígitos** representan los rubros de agrupación:
11- Activo Corriente
21- Pasivo Corriente
31- Capital Contable

**Cuatro dígitos** se asignan a las cuentas de mayor: cuentas de primer orden
1101- Efectivo y Equivalentes de Efectivo
2101- Cuentas y Documentos por pagar
3101- Capital Social

**Seis dígitos** se asignan a las sub-cuentas: cuentas de segundo orden
110101 - Caja
210101 - Proveedores locales

**Ocho dígitos** son para las cuentas de tercer orden (las visualizadas
en Odoo):
1101- Efectivo y Equivalentes
110101- Caja
11010101 Caja General
    """,
    'author': 'Gustavo Valverde - iterativo | Consultores de Odoo',
    'website': 'http://iterativo.do',
    'depends': ['account',
                'base_iban'
                ],
    'data': [
        # Basic accounting data
        'data/account_group.xml',
        'data/l10n_do_chart_data.xml',
        'data/account_account_tag_data.xml',
        'data/account.account.template.csv',
        'data/account_chart_template_data.xml',
        'data/account_data.xml',
        'data/account_tax_report_data.xml',
        'data/account.tax.template.xml',
        'data/l10n_do_res_partner_title.xml',
        # Adds fiscal position
        'data/fiscal_position_template.xml',
        # configuration wizard, views, reports...
        'data/account_chart_template_configure_data.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,code,name,user_type_id/id,reconcile,chart_template_id/id,tag_ids/id,group_id/id
do_niif_11010301,11010301,Depósitos a corto plazo en RD$,account.data_account_type_liquidity,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1101,account_tag_110103",l10n_do.account_group_110103
do_niif_11010302,11010302,Depósitos a corto plazo en USD$,account.data_account_type_liquidity,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1101,account_tag_110103",l10n_do.account_group_110103
do_niif_11020100,11020100,Bonos y Acciones Temporales,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1102",l10n_do.account_group_1102
do_niif_11020200,11020200,Operaciones en Bolsa,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1102",l10n_do.account_group_1102
do_niif_11020300,11020300,Otros Valores Negociables,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1102",l10n_do.account_group_1102
do_niif_11030101,11030101,Cheques Devueltos por Cobrar,account.data_account_type_receivable,TRUE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1103,account_tag_110301",l10n_do.account_group_110301
do_niif_11030102,11030102,Intereses por Cobrar,account.data_account_type_receivable,TRUE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1103,account_tag_110301",l10n_do.account_group_110301
do_niif_11030201,11030201,Cuentas por Cobrar a Clientes,account.data_account_type_receivable,TRUE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1103,account_tag_110302",l10n_do.account_group_110302
do_niif_11030210,11030210,Cuentas por Cobrar a Clientes (PoS),account.data_account_type_receivable,TRUE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1103,account_tag_110302",l10n_do.account_group_110302
do_niif_11030202,11030202,Cuentas por Cobrar a Funcionarios y Empleados,account.data_account_type_receivable,TRUE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1103,account_tag_110302",l10n_do.account_group_110302
do_niif_11030203,11030203,Cuentas por Cobrar a Afiliadas,account.data_account_type_receivable,TRUE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1103,account_tag_110302",l10n_do.account_group_110302
do_niif_11030204,11030204,Cuentas por Cobrar a Accionistas,account.data_account_type_receivable,TRUE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1103,account_tag_110302",l10n_do.account_group_110302
do_niif_11030205,11030205,Otras Cuentas por Cobrar,account.data_account_type_receivable,TRUE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1103,account_tag_110302",l10n_do.account_group_110302
do_niif_11040100,11040100,Provisiones de Incobrables a Clientes,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1104",l10n_do.account_group_1104
do_niif_11040200,11040200,Provisiones de Incobrables a Funcionarios y Empleados,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1104",l10n_do.account_group_1104
do_niif_11040300,11040300,Provisiones de Incobrables a Afiliadas,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1104",l10n_do.account_group_1104
do_niif_11040400,11040400,Provisiones de Incobrables a Accionistas,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1104",l10n_do.account_group_1104
do_niif_11040500,11040500,Provisiones de Otras Cuentas Incobrables,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1104",l10n_do.account_group_1104
do_niif_11050100,11050100,Inventario de Mercancías o Productos Terminados,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1105",l10n_do.account_group_1105
do_niif_11050200,11050200,Inventario de Materia Prima,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1105",l10n_do.account_group_1105
do_niif_11050300,11050300,Inventario en Tránsito,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1105",l10n_do.account_group_1105
do_niif_11050400,11050400,Inventario de Materiales y Suministros,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1105",l10n_do.account_group_1105
do_niif_11050500,11050500,Inventario de Combustibles,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1105",l10n_do.account_group_1105
do_niif_11050600,11050600,Bienes Enviados No Facturados,account.data_account_type_current_assets,TRUE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1105",l10n_do.account_group_1105
do_niif_11060100,11060100,Deterioro Acum. de Inventario en Almacen,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1106",l10n_do.account_group_1106
do_niif_11060200,11060200,Deterioro Acum. de Materiales y Suministros,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1106",l10n_do.account_group_1106
do_niif_11060300,11060300,Deterioro Acum. de Combustibles,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1106",l10n_do.account_group_1106
do_niif_11070100,11070100,Obsolescencia de Inventario en Almacen,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1107",l10n_do.account_group_1107
do_niif_11070200,11070200,Obsolescencia de Materiales y Suministros,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1107",l10n_do.account_group_1107
do_niif_11070300,11070300,Obsolescencia de Combustibles,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1107",l10n_do.account_group_1107
do_niif_11080101,11080101,ITBIS Pagado en Compras de Bienes Locales,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1108,account_tag_110801",l10n_do.account_group_110801
do_niif_11080102,11080102,ITBIS Pagado en Compras de Servicios Deducibles,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1108,account_tag_110801",l10n_do.account_group_110801
do_niif_11080103,11080103,ITBIS Pagado en Importaciones,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1108,account_tag_110801",l10n_do.account_group_110801
do_niif_11080104,11080104,ITBIS en Compras de Bienes o Servicios Sujetos a Proporcionalidad,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1108,account_tag_110801",l10n_do.account_group_110801
do_niif_11080105,11080105,ITBIS en Importaciones Sujeto a Proporcionalidad,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1108,account_tag_110801",l10n_do.account_group_110801
do_niif_11080106,11080106,ITBIS Pagado en Importaciones Sujeto a Proporcionalidad,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1108,account_tag_110801",l10n_do.account_group_110801
do_niif_11080301,11080301,ITBIS Retenido por Ventas (N08-04 y N02-05),account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1108,account_tag_110803",l10n_do.account_group_110803
do_niif_11080302,11080302,ITBIS Retenido por Entidades del Estado (5%),account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1108,account_tag_110803",l10n_do.account_group_110803
do_niif_11080303,11080303,Saldo a Favor Anterior,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1108,account_tag_110803",l10n_do.account_group_110803
do_niif_11090100,11090100,Acciones Temporales,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1109",l10n_do.account_group_1109
do_niif_11090200,11090200,Depósitos a Plazo Temporales,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1109",l10n_do.account_group_1109
do_niif_11090300,11090300,Bonos Temporales,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1109",l10n_do.account_group_1109
do_niif_11090400,11090400,Fianzas y Depósitos,account.data_account_type_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1109",l10n_do.account_group_1109
do_niif_11100100,11100100,Avance a Proveedores,account.data_account_type_prepayments,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1110",l10n_do.account_group_1110
do_niif_11100200,11100200,Alquileres pagados por Anticipado,account.data_account_type_prepayments,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1110",l10n_do.account_group_1110
do_niif_11100300,11100300,Anticipos para Seguros,account.data_account_type_prepayments,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1110",l10n_do.account_group_1110
do_niif_11100400,11100400,Anticipos para ISR,account.data_account_type_prepayments,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1110",l10n_do.account_group_1110
do_niif_11100500,11100500,Anticipos para Gastos,account.data_account_type_prepayments,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1110",l10n_do.account_group_1110
do_niif_11100600,11100600,Anticipos para 1% Activos,account.data_account_type_prepayments,FALSE,do_chart_template,"account_tag_1,account_tag_11,account_tag_1110",l10n_do.account_group_1110
do_niif_12010101,12010101,Terrenos,account.data_account_type_fixed_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1201,account_tag_120101",l10n_do.account_group_120101
do_niif_12010102,12010102,Edificaciones,account.data_account_type_fixed_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1201,account_tag_120101",l10n_do.account_group_120101
do_niif_12010103,12010103,Mejoras en Edificaciones,account.data_account_type_fixed_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1201,account_tag_120101",l10n_do.account_group_120101
do_niif_12010104,12010104,Mejoras en Edificaciones y Locales Arrendados,account.data_account_type_fixed_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1201,account_tag_120101",l10n_do.account_group_120101
do_niif_12010201,12010201,Equipos de Transporte Liviano,account.data_account_type_fixed_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1201,account_tag_120102",l10n_do.account_group_120102
do_niif_12010202,12010202,Mobiliarios y Equipos de Oficina,account.data_account_type_fixed_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1201,account_tag_120102",l10n_do.account_group_120102
do_niif_12010301,12010301,Maquinarias,account.data_account_type_fixed_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1201,account_tag_120103",l10n_do.account_group_120103
do_niif_12010302,12010302,Herramientas,account.data_account_type_fixed_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1201,account_tag_120103",l10n_do.account_group_120103
do_niif_12010303,12010303,Instalaciones,account.data_account_type_fixed_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1201,account_tag_120103",l10n_do.account_group_120103
do_niif_12010304,12010304,Sistemas de Cámara y Seguridad,account.data_account_type_fixed_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1201,account_tag_120103",l10n_do.account_group_120103
do_niif_12010305,12010305,Misceláneos u otros activos,account.data_account_type_fixed_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1201,account_tag_120103",l10n_do.account_group_120103
do_niif_12020100,12020100,Depreciación Acum. de Edificios,account.data_account_type_non_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1202",l10n_do.account_group_1202
do_niif_12020200,12020200,Depreciación Acum. de Equipos de Transporte,account.data_account_type_non_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1202",l10n_do.account_group_1202
do_niif_12020300,12020300,Depreciación Acum. de Mobiliarios y Equipos,account.data_account_type_non_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1202",l10n_do.account_group_1202
do_niif_12020400,12020400,Depreciación Acum. de Maquinarias,account.data_account_type_non_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1202",l10n_do.account_group_1202
do_niif_12020500,12020500,Depreciación Acum. de Herramientas,account.data_account_type_non_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1202",l10n_do.account_group_1202
do_niif_12020600,12020600,Depreciación Acum. de Instalaciones,account.data_account_type_non_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1202",l10n_do.account_group_1202
do_niif_12040100,12040100,Marcas y Patentes,account.data_account_type_fixed_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1204",l10n_do.account_group_1204
do_niif_12040200,12040200,Licencias y Programas Informáticos,account.data_account_type_fixed_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1204",l10n_do.account_group_1204
do_niif_12050100,12050100,Edificaciones y Locales Arrendados,account.data_account_type_fixed_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1205",l10n_do.account_group_1205
do_niif_12050200,12050200,Maquinarias y Equipos Arrendados,account.data_account_type_fixed_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1205",l10n_do.account_group_1205
do_niif_12050300,12050300,Instalaciones Arrendadas,account.data_account_type_fixed_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1205",l10n_do.account_group_1205
do_niif_12060100,12060100,Depreciación Acum. Edificaciones y Locales Arrendados,account.data_account_type_non_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1206",l10n_do.account_group_1206
do_niif_12060200,12060200,Depreciación Acum. Maquinarias y Equipos Arrendados,account.data_account_type_non_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1206",l10n_do.account_group_1206
do_niif_12060300,12060300,Depreciación Acum. Instalaciones Arrendadas,account.data_account_type_non_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1206",l10n_do.account_group_1206
do_niif_12070100,12070100,Deterioro Valor Acum. Edificios y Locales Arrendados,account.data_account_type_non_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1207",l10n_do.account_group_1207
do_niif_12070200,12070200,Deterioro Valor Acum. Maquinaria y Equipo Arrendado,account.data_account_type_non_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1207",l10n_do.account_group_1207
do_niif_12070300,12070300,Deterioro Valor Acum. Instalaciones Arrendadas,account.data_account_type_non_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1207",l10n_do.account_group_1207
do_niif_12080100,12080100,Acciones en otras sociedades,account.data_account_type_non_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1208",l10n_do.account_group_1208
do_niif_12090100,12090100,Diferencias temporales deducibles,account.data_account_type_non_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1209",l10n_do.account_group_1209
do_niif_12090200,12090200,Gastos Organizacionales Diferidos,account.data_account_type_non_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1209",l10n_do.account_group_1209
do_niif_12090300,12090300,ISR Deducible,account.data_account_type_non_current_assets,FALSE,do_chart_template,"account_tag_1,account_tag_12,account_tag_1209",l10n_do.account_group_1209
do_niif_13010100,13010101,Costos de Construcción en Proceso,account.data_account_type_non_current_assets,FALSE,do_chart_template,,
do_niif_21010100,21010100,Préstamos Bancarios por Pagar,account.data_account_type_payable,TRUE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2101",l10n_do.account_group_2101
do_niif_21010200,21010200,Cuentas por Pagar a Proveedores Locales,account.data_account_type_payable,TRUE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2101",l10n_do.account_group_2101
do_niif_21010300,21010300,Cuentas por Pagar a Proveedores del Exterior,account.data_account_type_payable,TRUE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2101",l10n_do.account_group_2101
do_niif_21010400,21010400,Cuentas por Pagar a Afiliadas,account.data_account_type_payable,TRUE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2101",l10n_do.account_group_2101
do_niif_21010500,21010500,Cuentas por Pagar a Accionistas,account.data_account_type_payable,TRUE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2101",l10n_do.account_group_2101
do_niif_21010600,21010600,Nóminas por Pagar,account.data_account_type_payable,TRUE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2101",l10n_do.account_group_2101
do_niif_21010700,21010700,Intereses por Pagar,account.data_account_type_payable,TRUE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2101",l10n_do.account_group_2101
do_niif_21010800,21010800,Otras Cuentas por Pagar,account.data_account_type_payable,TRUE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2101",l10n_do.account_group_2101
do_niif_21020100,21020100,Dividendos por Pagar,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2102",l10n_do.account_group_2102
do_niif_21020200,21020200,Comisiones por Pagar,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2102",l10n_do.account_group_2102
do_niif_21020300,21020300,Vacaciones por Pagar,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2102",l10n_do.account_group_2102
do_niif_21020400,21020400,Bonificaciones por Pagar,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2102",l10n_do.account_group_2102
do_niif_21020500,21020500,Regalía Pascual por Pagar,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2102",l10n_do.account_group_2102
do_niif_21020600,21020600,Seguro Médico por Pagar,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2102",l10n_do.account_group_2102
do_niif_21020700,21020700,Seguro Familiar de Salud (SFS) por Pagar,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2102",l10n_do.account_group_2102
do_niif_21020800,21020800,Aporte a Fondo de Pensiones (AFP) por Pagar,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2102",l10n_do.account_group_2102
do_niif_21020900,21020900,Seguro de Riesgo Laboral (SRL) por Pagar,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2102",l10n_do.account_group_2102
do_niif_21021000,21021000,INFOTEP por Pagar,account.data_account_type_payable,TRUE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2102",l10n_do.account_group_2102
do_niif_21021100,21021100,Tesorería de la Seguridad Social (TSS) por Pagar,account.data_account_type_payable,TRUE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2102",l10n_do.account_group_2102
do_niif_21021200,21021200,Bienes Recibidos no Facturados,account.data_account_type_non_current_liabilities,TRUE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2102",l10n_do.account_group_2102
do_niif_21030101,21030101,ITBIS por Venta Servicios,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210301",l10n_do.account_group_210301
do_niif_21030102,21030102,ITBIS por Venta Bienes,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210301",l10n_do.account_group_210301
do_niif_21030103,21030103,ITBIS por Venta Servicios en Nombre de Terceros,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210301",l10n_do.account_group_210301
do_niif_21030201,21030201,ITBIS Retenido a Persona Jurídica (N02-05),account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210302",l10n_do.account_group_210302
do_niif_21030202,21030202,ITBIS Retenido a Persona Física (R293-11),account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210302",l10n_do.account_group_210302
do_niif_21030203,21030203,ITBIS Retenido a Entidades No Lucrativas (N01-11),account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210302",l10n_do.account_group_210302
do_niif_21030204,21030204,ITBIS Retenido por Servicios Profesionales (N02-05),account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210302",l10n_do.account_group_210302
do_niif_21030205,21030205,ITBIS Retenido a Informales de Bienes (N08-10),account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210302",l10n_do.account_group_210302
do_niif_21030301,21030301,ISR Retenido por Honorarios Profesionales de Personas Físicas,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210303",l10n_do.account_group_210303
do_niif_21030302,21030302,ISR Retenido por Alquileres Pagados a Personas Físicas,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210303",l10n_do.account_group_210303
do_niif_21030303,21030303,ISR Retenido por Dividendos Pagados,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210303",l10n_do.account_group_210303
do_niif_21030304,21030304,ISR Retenido por Intereses Pagados al Exterior,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210303",l10n_do.account_group_210303
do_niif_21030305,21030305,ISR Retenido por Intereses Pagados,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210303",l10n_do.account_group_210303
do_niif_21030306,21030306,ISR Retenido por Transferencias de Títulos y Propiedades,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210303",l10n_do.account_group_210303
do_niif_21030307,21030307,ISR Retenido por Remesas al Exterior (L253-12),account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210303",l10n_do.account_group_210303
do_niif_21030308,21030308,Otras Retenciones (N07-07),account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210303",l10n_do.account_group_210303
do_niif_21030309,21030309,Otras Retenciones,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210303",l10n_do.account_group_210303
do_niif_21030401,21030401,Retención de Impuesto al Salario (ISR del IR3),account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210304",l10n_do.account_group_210304
do_niif_21030402,21030402,Retención de Seguro Familiar de Salud (SFS),account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210304",l10n_do.account_group_210304
do_niif_21030403,21030403,Retención de Fondo de Pensiones (AFP),account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210304",l10n_do.account_group_210304
do_niif_21030404,21030404,Retención de INFOTEP (0.5%),account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210304",l10n_do.account_group_210304
do_niif_21030501,21030501,ISR por Pagar,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210305",l10n_do.account_group_210305
do_niif_21030502,21030502,Anticipos ISR por Pagar,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210305",l10n_do.account_group_210305
do_niif_21030503,21030503,Propina Legal por Pagar (L16-92),account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210305",l10n_do.account_group_210305
do_niif_21030504,21030504,Otros Impuestos por Pagar,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2103,account_tag_210305",l10n_do.account_group_210305
do_niif_21040100,21040100,Provisiones Pago Alquileres,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2104",l10n_do.account_group_2104
do_niif_21040200,21040200,Provisiones Arrendamiento Financiero,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2104",l10n_do.account_group_2104
do_niif_21040300,21040300,Provisiones Gastos Fijos,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2104",l10n_do.account_group_2104
do_niif_21050100,21050100,Tarjeta de Crédito Empresarial,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2105",l10n_do.account_group_2105
do_niif_21050200,21050200,Prima de Tarjetas de Crédito,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_21,account_tag_2105",l10n_do.account_group_2105
do_niif_22010100,22010100,Préstamos Bancarios Largo Plazo,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_22,account_tag_2201",l10n_do.account_group_2201
do_niif_22010200,22010200,Préstamos Hipotecarios LP,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_22,account_tag_2201",l10n_do.account_group_2201
do_niif_22012300,22012300,Préstamos Accionistas / Particulares,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_22,account_tag_2201",l10n_do.account_group_2201
do_niif_22020100,22020100,Provisiones Beneficios de Empleados LP,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_22,account_tag_2202",l10n_do.account_group_2202
do_niif_22020200,22020200,Provisiones Prestaciones Laborales,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_22,account_tag_2202",l10n_do.account_group_2202
do_niif_22020300,22020300,Provisiones Indemnizaciones,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_22,account_tag_2202",l10n_do.account_group_2202
do_niif_22030100,22030100,Anticipos o Garantías de Clientes,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_22,account_tag_2203",l10n_do.account_group_2203
do_niif_22030200,22030200,Depósitos por Identificar,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_22,account_tag_2203",l10n_do.account_group_2203
do_niif_22040100,22040100,Provisiones Arrendamiento Financiero LP,account.data_account_type_non_current_liabilities,FALSE,do_chart_template,"account_tag_2,account_tag_22,account_tag_2204",l10n_do.account_group_2204
do_niif_31010100,31010100,Capital Social Autorizado,account.data_account_type_equity,FALSE,do_chart_template,"account_tag_3,account_tag_31,account_tag_3101",l10n_do.account_group_3101
do_niif_31010200,31010200,Capital Social en Acciones no Emitidas,account.data_account_type_equity,FALSE,do_chart_template,"account_tag_3,account_tag_31,account_tag_3101",l10n_do.account_group_3101
do_niif_31010300,31010300,Capital Social Pagado,account.data_account_type_equity,FALSE,do_chart_template,"account_tag_3,account_tag_31,account_tag_3101",l10n_do.account_group_3101
do_niif_31010400,31010400,Capital Social No Pagado,account.data_account_type_equity,FALSE,do_chart_template,"account_tag_3,account_tag_31,account_tag_3101",l10n_do.account_group_3101
do_niif_31010500,31010500,Acciones o Participaciones en Tesorería,account.data_account_type_equity,FALSE,do_chart_template,"account_tag_3,account_tag_31,account_tag_3101",l10n_do.account_group_3101
do_niif_31020100,31020100,Superávit por Revaluación de Terrenos,account.data_account_type_equity,FALSE,do_chart_template,"account_tag_3,account_tag_31,account_tag_3102",l10n_do.account_group_3102
do_niif_31020200,31020200,Superávit por Revaluación de Edificaciones,account.data_account_type_equity,FALSE,do_chart_template,"account_tag_3,account_tag_31,account_tag_3102",l10n_do.account_group_3102
do_niif_31020300,31020300,Superávit por Revaluación de Instalaciones,account.data_account_type_equity,FALSE,do_chart_template,"account_tag_3,account_tag_31,account_tag_3102",l10n_do.account_group_3102
do_niif_31020400,31020400,Superávit por Revaluación de Mobiliario y Equipo,account.data_account_type_equity,FALSE,do_chart_template,"account_tag_3,account_tag_31,account_tag_3102",l10n_do.account_group_3102
do_niif_32010100,32010100,Reserva Legal,account.data_account_type_equity,FALSE,do_chart_template,"account_tag_3,account_tag_32,account_tag_3201",l10n_do.account_group_3201
do_niif_33010100,33010100,Utilidades de Ejercicios Anteriores,account.data_account_type_equity,FALSE,do_chart_template,"account_tag_3,account_tag_33,account_tag_3301",l10n_do.account_group_3301
do_niif_33010200,33010200,Pérdidas de Ejercicios Anteriores,account.data_account_type_equity,FALSE,do_chart_template,"account_tag_3,account_tag_33,account_tag_3301",l10n_do.account_group_3301
do_niif_33020100,33020100,Utilidad del Ejercicio,account.data_account_type_equity,FALSE,do_chart_template,"account_tag_3,account_tag_33,account_tag_3302",l10n_do.account_group_3302
do_niif_33020200,33020200,Pérdidas del Ejercicio,account.data_account_type_equity,FALSE,do_chart_template,"account_tag_3,account_tag_33,account_tag_3302",l10n_do.account_group_3302
do_niif_33040100,33040100,Superávit en Reserva,account.data_account_type_equity,FALSE,do_chart_template,"account_tag_3,account_tag_33,account_tag_3304",l10n_do.account_group_3304
do_niif_33040200,33040200,Déficit en Reserva,account.data_account_type_equity,FALSE,do_chart_template,"account_tag_3,account_tag_33,account_tag_3304",l10n_do.account_group_3304
do_niif_41010100,41010100,Ventas de Bienes,account.data_account_type_revenue,FALSE,do_chart_template,"account_tag_4,account_tag_41,account_tag_4101",l10n_do.account_group_4101
do_niif_41010200,41010200,Ventas por Exportaciones de Bienes,account.data_account_type_revenue,FALSE,do_chart_template,"account_tag_4,account_tag_41,account_tag_4101",l10n_do.account_group_4101
do_niif_41020100,41020100,Ventas de Servicios,account.data_account_type_revenue,FALSE,do_chart_template,"account_tag_4,account_tag_41,account_tag_4102",l10n_do.account_group_4102
do_niif_41020200,41020200,Ventas por Exportaciones de Servicios,account.data_account_type_revenue,FALSE,do_chart_template,"account_tag_4,account_tag_41,account_tag_4102",l10n_do.account_group_4102
do_niif_41030100,41030100,Devoluciones de Bienes,account.data_account_type_revenue,FALSE,do_chart_template,"account_tag_4,account_tag_41,account_tag_4103",l10n_do.account_group_4103
do_niif_41030200,41030200,Devoluciones por Servicios,account.data_account_type_revenue,FALSE,do_chart_template,"account_tag_4,account_tag_41,account_tag_4103",l10n_do.account_group_4103
do_niif_41040100,41040100,Descuentos por Ventas (por NC),account.data_account_type_revenue,FALSE,do_chart_template,"account_tag_4,account_tag_41,account_tag_4104",l10n_do.account_group_4104
do_niif_42010100,42010100,Intereses sobre Certificados,account.data_account_type_revenue,FALSE,do_chart_template,"account_tag_4,account_tag_42,account_tag_4201",l10n_do.account_group_4201
do_niif_42010200,42010200,Intereses por Cuentas Bancarias,account.data_account_type_revenue,FALSE,do_chart_template,"account_tag_4,account_tag_42,account_tag_4201",l10n_do.account_group_4201
do_niif_42010300,42010300,Intereses por Financiamientos,account.data_account_type_revenue,FALSE,do_chart_template,"account_tag_4,account_tag_42,account_tag_4201",l10n_do.account_group_4201
do_niif_42020100,42020100,Ingresos por Ventas de Activos Depreciables (CAT II y III) ,account.data_account_type_revenue,FALSE,do_chart_template,"account_tag_4,account_tag_42,account_tag_4202",l10n_do.account_group_4202
do_niif_42030100,42030100,Ingresos por Dividendos Ganados,account.data_account_type_revenue,FALSE,do_chart_template,"account_tag_4,account_tag_42,account_tag_4203",l10n_do.account_group_4203
do_niif_42040100,42040100,Ingresos por Diferencia Cambiaria,account.data_account_type_other_income,FALSE,do_chart_template,"account_tag_4,account_tag_42,account_tag_4204",l10n_do.account_group_4204
do_niif_42040200,42040200,Cobro de Cuentas Incobrables,account.data_account_type_other_income,FALSE,do_chart_template,"account_tag_4,account_tag_42,account_tag_4204",l10n_do.account_group_4204
do_niif_42040300,42040300,Ingresos por Sobrante en Caja,account.data_account_type_other_income,FALSE,do_chart_template,"account_tag_4,account_tag_42,account_tag_4204",l10n_do.account_group_4204
do_niif_51010100,51010100,Costos de Bienes,account.data_account_type_direct_costs,FALSE,do_chart_template,"account_tag_5,account_tag_51,account_tag_5101",l10n_do.account_group_5101
do_niif_51010200,51010200,Costos de Servicios,account.data_account_type_direct_costs,FALSE,do_chart_template,"account_tag_5,account_tag_51,account_tag_5101",l10n_do.account_group_5101
do_niif_51010300,51010300,Diferencia en Precio,account.data_account_type_direct_costs,FALSE,do_chart_template,"account_tag_5,account_tag_51,account_tag_5101",l10n_do.account_group_5101
do_niif_51020400,51020400,Costos de Importación,account.data_account_type_direct_costs,FALSE,do_chart_template,"account_tag_5,account_tag_51,account_tag_5102",l10n_do.account_group_5102
do_niif_51010500,51010500,ITBIS llevado al Costo,account.data_account_type_direct_costs,FALSE,do_chart_template,"account_tag_5,account_tag_51,account_tag_5101",l10n_do.account_group_5101
do_niif_51020100,51020100,Costos de Materia Prima,account.data_account_type_direct_costs,FALSE,do_chart_template,"account_tag_5,account_tag_51,account_tag_5102",l10n_do.account_group_5102
do_niif_51020200,51020200,Costos de Mano de Obra,account.data_account_type_direct_costs,FALSE,do_chart_template,"account_tag_5,account_tag_51,account_tag_5102",l10n_do.account_group_5102
do_niif_51020300,51020300,Costos Indirectos,account.data_account_type_direct_costs,FALSE,do_chart_template,"account_tag_5,account_tag_51,account_tag_5102",l10n_do.account_group_5102
do_niif_52010100,61010100,Sueldos y Salarios,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201",l10n_do.account_group_6101
do_niif_52010200,61010200,Regalía Pascual,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201",l10n_do.account_group_6101
do_niif_52010300,61010300,Retribuciones Complementarias,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201",l10n_do.account_group_6101
do_niif_52010400,61010400,Bono Vacacional,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201",l10n_do.account_group_6101
do_niif_52010500,61010500,Bono por Desempeño,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201",l10n_do.account_group_6101
do_niif_52010600,61010600,Comisiones a Empleados,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201",l10n_do.account_group_6101
do_niif_52010700,61010700,Indemnizaciones,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201",l10n_do.account_group_6101
do_niif_52010800,61010800,Bonificaciones y Gratificaciones,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201",l10n_do.account_group_6101
do_niif_52010900,61010900,Comidas o Dietas al Personal,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201",l10n_do.account_group_6101
do_niif_52011000,61011000,Cursos y Entrenamientos,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201",l10n_do.account_group_6101
do_niif_52011100,61011100,Prestaciones Laborales (Preaviso y Cesantia),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201",l10n_do.account_group_6101
do_niif_52011200,61011200,Incentivos,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201",l10n_do.account_group_6101
do_niif_52011300,61011300,Horas Extras,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201",l10n_do.account_group_6101
do_niif_52011400,61011400,Uniformes,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201",l10n_do.account_group_6101
do_niif_52011500,61011500,Otros Gastos de Personal,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201",l10n_do.account_group_6101
do_niif_52010102,61010102,Contribución a la Administradora de Fondos de Pensiones (AFP),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201,account_tag_520101",l10n_do.account_group_610101
do_niif_52010103,61010103,Contribución al Seguro Riesgo Laboral (SRL),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201,account_tag_520101",l10n_do.account_group_610101
do_niif_52010104,61010104,Contribución al Seguro Familiar de Salud (SFS),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201,account_tag_520101",l10n_do.account_group_610101
do_niif_52010201,61010201,Contribución a la Administradora de Riesgos de Salud (ARS),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201,account_tag_520102",l10n_do.account_group_610102
do_niif_52010202,61010202,Contribución a Seguros del Personal,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201,account_tag_520102",l10n_do.account_group_610102
do_niif_52010203,61010203,Contribución a Planes Complementarios de Salud,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201,account_tag_520102",l10n_do.account_group_610102
do_niif_52010204,61010204,Contribución Seguros de Vida,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201,account_tag_520102",l10n_do.account_group_610102
do_niif_52010205,61010205,Contribución al INFOTEP,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5201,account_tag_520102",l10n_do.account_group_610102
do_niif_52020100,61020100,Energía Eléctrica,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5202",l10n_do.account_group_6102
do_niif_52020200,61020200,Comunicaciones,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5202",l10n_do.account_group_6102
do_niif_52020300,61020300,Suministros de Oficina (Papelería y útiles),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5202",l10n_do.account_group_6102
do_niif_52020400,61020400,Útiles de Aseo y Limpieza,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5202",l10n_do.account_group_6102
do_niif_52020500,61020500,Agua y Basura,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5202",l10n_do.account_group_6102
do_niif_52020600,61020600,Seguro de Edificio y Locales,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5202",l10n_do.account_group_6102
do_niif_52020700,61020700,Combustibles y Lubricantes,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5202",l10n_do.account_group_6102
do_niif_52020800,61020800,Alquileres / Arrendamientos,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5202",l10n_do.account_group_6102
do_niif_52020900,61020900,Franquicias,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5202",l10n_do.account_group_6102
do_niif_52021000,61021000,Eventos,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5202",l10n_do.account_group_6102
do_niif_52021100,61021100,Seguros,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5202",l10n_do.account_group_6102
do_niif_52021200,61021200,Servicios de Mensajería,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5202",l10n_do.account_group_6102
do_niif_52021300,61021300,Flete y Carga,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5202",l10n_do.account_group_6102
do_niif_52021400,61021400,Cuotas y Suscripciones,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5202",l10n_do.account_group_6102
do_niif_52021500,61021500,Alojamiento en Hoteles,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5202",l10n_do.account_group_6102
do_niif_52030101,61030101,Legales (P. Física),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520301",l10n_do.account_group_610301
do_niif_52030102,61030102,Contabilidad y Auditoría (P. Física),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520301",l10n_do.account_group_610301
do_niif_52030103,61030103,Tecnología (P. Física),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520301",l10n_do.account_group_610301
do_niif_52030104,61030104,Mantenimientos de Planta (P. Física),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520301",l10n_do.account_group_610301
do_niif_52030105,61030105,Mantenimientos de Local (P. Física),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520301",l10n_do.account_group_610301
do_niif_52030106,61030106,Mantenimientos Mobiliarios y Equipos (P. Física),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520301",l10n_do.account_group_610301
do_niif_52030107,61030107,Asesorías (P. Física),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520301",l10n_do.account_group_610301
do_niif_52030108,61030108,Fumigaciones (P. Física),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520301",l10n_do.account_group_610301
do_niif_52030109,61030109,Copias y Escaneos (P. Física),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520301",l10n_do.account_group_610301
do_niif_52030110,61030110,Servicios de Vigilancia (P. Física),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520301",l10n_do.account_group_610301
do_niif_52030111,61030111,Otros Servicios Profesionales (P. Física),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520301",l10n_do.account_group_610301
do_niif_52030201,61030201,Legales (P. Jurídica),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520302",l10n_do.account_group_610302
do_niif_52030202,61030202,Contabilidad y Auditoría (P. Jurídica),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520302",l10n_do.account_group_610302
do_niif_52030203,61030203,Tecnología (P. Jurídica),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520302",l10n_do.account_group_610302
do_niif_52030204,61030204,Mantenimiento de Planta (P. Jurídica),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520302",l10n_do.account_group_610302
do_niif_52030205,61030205,Mantenimiento del Local (P. Jurídica),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520302",l10n_do.account_group_610302
do_niif_52030206,61030206,Mantenimiento Mobiliario y Equipos (P. Jurídica),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520302",l10n_do.account_group_610302
do_niif_52030207,61030207,Asesorías (P. Jurídica),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520302",l10n_do.account_group_610302
do_niif_52030208,61030208,Fumigaciones (P. Jurídica),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520302",l10n_do.account_group_610302
do_niif_52030209,61030209,Copias y Escaneos (P. Jurídica),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520302",l10n_do.account_group_610302
do_niif_52030210,61030210,Servicios de Vigilancia (P. Jurídica),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520302",l10n_do.account_group_610302
do_niif_52030211,61030211,Otros Servicios Profesionales  (P. Jurídica),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520302",l10n_do.account_group_610302
do_niif_52030301,61030301,Honorarios por Servicios del Exterior - Relacionadas,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520303",l10n_do.account_group_610303
do_niif_52030302,61030302,Honorarios por Servicios del Exterior - Terceros,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5203,account_tag_520303",l10n_do.account_group_610303
do_niif_52040100,61040100,Gastos por Depreciación de Edificios,account.data_account_type_depreciation,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5204",l10n_do.account_group_6104
do_niif_52040200,61040200,Gastos por Depreciación de Equipo de Transporte,account.data_account_type_depreciation,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5204",l10n_do.account_group_6104
do_niif_52040300,61040300,Gastos por Depreciación de Mobiliario y Equipos,account.data_account_type_depreciation,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5204",l10n_do.account_group_6104
do_niif_52040400,61040400,Gastos por Depreciación de Maquinaria,account.data_account_type_depreciation,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5204",l10n_do.account_group_6104
do_niif_52040500,61040500,Gastos por Depreciación de Herramientas,account.data_account_type_depreciation,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5204",l10n_do.account_group_6104
do_niif_52040600,61040600,Gastos por Depreciación de Instalaciones,account.data_account_type_depreciation,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5204",l10n_do.account_group_6104
do_niif_52050100,61050100,Gastos por Reparación de Edificios,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5205",l10n_do.account_group_6105
do_niif_52050200,61050200,Gastos por Reparación de Equipo de Transporte,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5205",l10n_do.account_group_6105
do_niif_52050300,61050300,Gastos por Reparación de Mobiliario y Equipos,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5205",l10n_do.account_group_6105
do_niif_52050400,61050400,Gastos por Reparación de Maquinaria,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5205",l10n_do.account_group_6105
do_niif_52050500,61050500,Gastos por Reparación de Herramientas,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5205",l10n_do.account_group_6105
do_niif_52050600,61050600,Gastos por Reparación de Instalaciones,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5205",l10n_do.account_group_6105
do_niif_52060100,61060100,Relaciones Públicas,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5206",l10n_do.account_group_6106
do_niif_52060200,61060200,Publicidad,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5206",l10n_do.account_group_6106
do_niif_52060300,61060300,Gastos de Viajes y Representación,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5206",l10n_do.account_group_6106
do_niif_52060400,61060400,Donaciones,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5206",l10n_do.account_group_6106
do_niif_52060500,61060500,Donaciones a ProIndustria (Ley 392-07),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5206",l10n_do.account_group_6106
do_niif_52060600,61060600,Promociones,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5206",l10n_do.account_group_6106
do_niif_52060700,61060700,Gastos en Restaurantes,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5206",l10n_do.account_group_6106
do_niif_52070100,61070100,Gastos por Préstamos Financieros,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5207",l10n_do.account_group_6107
do_niif_52070200,61070200,Retención por Cheques o Transferencias Electrónicas (0.15%),account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5207",l10n_do.account_group_6107
do_niif_52070300,61070300,Intereses Bancarios,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5207",l10n_do.account_group_6107
do_niif_52070400,61070400,Comisiones Bancarias,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5207",l10n_do.account_group_6107
do_niif_52070500,61070500,Comisiones de Tarjeta de Crédito,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5207",l10n_do.account_group_6107
do_niif_52070600,61070600,Cargos Bancarios,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5207",l10n_do.account_group_6107
do_niif_52070700,61070700,Seguro sobre Préstamos Bancarios,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5207",l10n_do.account_group_6107
do_niif_52070800,61070800,Pérdidas por Diferencia Cambiaria,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5207",l10n_do.account_group_6107
do_niif_52070900,61070900,Otro Gastos Financieros,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5207",l10n_do.account_group_6107
do_niif_52080100,61080100,Pérdidas por Faltante en Caja,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5208",l10n_do.account_group_6108
do_niif_52080200,61080200,Pérdidas por Ventas de Activos Fijos,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5208",l10n_do.account_group_6108
do_niif_52080300,61080300,Pérdidas por Cuentas Incobrables,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5208",l10n_do.account_group_6108
do_niif_52080400,61080400,Gastos por ISR,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5208",l10n_do.account_group_6108
do_niif_52080500,61080500,Impuestos a los Activos,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5208",l10n_do.account_group_6108
do_niif_52080600,61080600,Gastos por Penalidades/Recargos de DGII,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5208",l10n_do.account_group_6108
do_niif_52080700,61080700,Gastos por Penalidades/Recargos de TSS,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5208",l10n_do.account_group_6108
do_niif_52080800,61080800,Gastos sin Comprobantes,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5208",l10n_do.account_group_6108
do_niif_52080900,61080900,Gastos No Deducibles,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5208",l10n_do.account_group_6108
do_niif_52081000,61081000,Gastos por Impuestos,account.data_account_type_expenses,FALSE,do_chart_template,"account_tag_6,account_tag_52,account_tag_5208",l10n_do.account_group_6108
do_niif_61010100,71010100,Ganancias o Pérdidas,account.data_unaffected_earnings,FALSE,do_chart_template,"account_tag_7,account_tag_61,account_tag_6101",l10n_do.account_group_7101

```

## File: data\account.tax.template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <!-- Chart template for Taxes -->
        <record id="tax_0_sale" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">4</field>
            <field name="name">Exento ITBIS Ventas</field>
            <field name="description">Exento</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_2A3_ingrs_local_vnts')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_2A3_ingrs_local_vnts')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="tax_0_purch" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">10</field>
            <field name="name">Exento ITBIS Compras</field>
            <field name="description">Exento</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
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

        <record id="tax_18_sale" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">1</field>
            <field name="name">18% ITBIS Ventas</field>
            <field name="description">18% ITBIS</field>
            <field name="amount">18</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_2b6_grvd')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030102'),
                    'plus_report_line_ids': [ref('account_tax_report_itbs_18_casilla')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_2b6_grvd')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030102'),
                    'minus_report_line_ids': [ref('account_tax_report_itbs_18_casilla')],
                }),
            ]"/>
        </record>

        <record id="tax_18_sale_incl" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">2</field>
            <field name="name">18% ITBIS Incl. Ventas</field>
            <field name="description">18% ITBIS</field>
            <field name="amount">18</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field eval="1" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_2b6_grvd')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030102'),
                    'plus_report_line_ids': [ref('account_tax_report_itbs_18_casilla')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_2b6_grvd')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030102'),
                    'minus_report_line_ids': [ref('account_tax_report_itbs_18_casilla')],
                }),
            ]"/>
        </record>

        <record id="tax_tip_sale" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">3</field>
            <field name="name">10% Propina Legal</field>
            <field name="description">10% Legal</field>
            <field name="amount">10</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="tax_group_tip"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030503'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030503'),
                }),
            ]"/>
        </record>

        <record id="tax_18_purch" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">18% ITBIS Compras</field>
            <field name="sequence">11</field>
            <field name="description">18% ITBIS (B)</field>
            <field name="amount">18</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080101'),
                    'plus_report_line_ids': [ref('account_tax_report_itbs_pgdo_locales')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080101'),
                    'minus_report_line_ids': [ref('account_tax_report_itbs_pgdo_locales')],
                }),
            ]"/>
        </record>

        <record id="tax_18_purch_incl" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">12</field>
            <field name="name">18% ITBIS Incl. Compras</field>
            <field name="description">18% ITBIS (Incl B)</field>
            <field name="amount">18</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="1" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080101'),
                    'plus_report_line_ids': [ref('account_tax_report_itbs_pgdo_locales')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080101'),
                    'minus_report_line_ids': [ref('account_tax_report_itbs_pgdo_locales')],
                }),
            ]"/>
        </record>

        <record id="tax_16_purch" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">16% ITBIS Compras</field>
            <field name="sequence">13</field>
            <field name="description">16% ITBIS (B)</field>
            <field name="amount">16</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080101'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080101'),
                }),
            ]"/>
        </record>

        <record id="tax_16_purch_incl" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">16% ITBIS Incl. Compras</field>
            <field name="sequence">14</field>
            <field name="description">16% ITBIS (Incl B)</field>
            <field name="amount">16</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="1" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080101'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080101'),
                }),
            ]"/>
        </record>

        <record id="tax_9_purch" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">9% ITBIS Compras</field>
            <field name="sequence">15</field>
            <field name="description">9% ITBIS (L690-16)</field>
            <field name="amount">9</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080101'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080101'),
                }),
            ]"/>
        </record>

        <record id="tax_9_purch_incl" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">9% ITBIS Incl. Compras</field>
            <field name="sequence">16</field>
            <field name="description">9% ITBIS (L690-16)</field>
            <field name="amount">9</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="1" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080101'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080101'),
                }),
            ]"/>
        </record>

        <record id="tax_8_purch" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">8% ITBIS Compras</field>
            <field name="sequence">17</field>
            <field name="description">8% ITBIS (L690-16)</field>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080101'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080101'),
                }),
            ]"/>
        </record>

        <record id="tax_8_purch_incl" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">8% ITBIS Incl. Compras</field>
            <field name="sequence">18</field>
            <field name="description">8% ITBIS (L690-16)</field>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="1" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080101'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080101'),
                }),
            ]"/>
        </record>

        <record id="tax_tip_purch" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">19</field>
            <field name="name">10% Propina Legal</field>
            <field name="description">10% Legal</field>
            <field name="amount">10</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="tax_group_tip"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_52080900'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_52080900'),
                }),
            ]"/>
        </record>

        <record id="tax_18_purch_serv" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">20</field>
            <field name="name">18% ITBIS Compras - Servicios</field>
            <field name="description">18% ITBIS (S)</field>
            <field name="amount">18</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080102'),
                    'plus_report_line_ids': [ref('account_tax_report_itbs_pgdo_locales')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080102'),
                    'minus_report_line_ids': [ref('account_tax_report_itbs_pgdo_locales')],
                }),
            ]"/>
        </record>

        <record id="tax_18_purch_serv_incl" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">20</field>
            <field name="name">18% ITBIS Incl. Compras - Servicios</field>
            <field name="description">18% ITBIS (Incl S)</field>
            <field name="amount">18</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="1" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080102'),
                    'plus_report_line_ids': [ref('account_tax_report_itbs_pgdo_locales')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080102'),
                    'minus_report_line_ids': [ref('account_tax_report_itbs_pgdo_locales')],
                }),
            ]"/>
        </record>

        <record id="tax_18_importation" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">20</field>
            <field name="name">18% ITBIS - Importaciones</field>
            <field name="description">18% ITBIS (IMP)</field>
            <field name="amount">18</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080103'),
                    'plus_report_line_ids': [ref('account_tax_report_itbs_pgdo_imptcn')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080103'),
                    'minus_report_line_ids': [ref('account_tax_report_itbs_pgdo_imptcn')],
                }),
            ]"/>
        </record>

        <record id="tax_18_of_10" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">20</field>
            <field name="name">18% ITBIS sobre el 10% del Monto Total</field>
            <field name="description">18% del 10%</field>
            <field name="amount">1.8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030102'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030102'),
                }),
            ]"/>
        </record>

        <record id="tax_0015_bank" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">30</field>
            <field name="name">0.15% Transferencia Bancaria</field>
            <field name="description">0.15% Trans</field>
            <field name="amount">0.0015</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">none</field>
            <field eval="1" name="price_include"/>
            <field name="tax_group_id" ref="group_tax"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_52070200'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_52070200'),
                }),
            ]"/>
        </record>

        <record id="tax_10_telco" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">30</field>
            <field name="name">10% ISC Telecomunicaciones</field>
            <field name="description">10% ISC</field>
            <field name="amount">10</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="tax_group_isc"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_52020200'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_52020200'),
                }),
            ]"/>
        </record>

        <record id="tax_2_telco" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">30</field>
            <field name="name">2% CDT Telecomunicaciones</field>
            <field name="description">2% CDT</field>
            <field name="amount">2</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_tax"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_52020200'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_52020200'),
                }),
            ]"/>
        </record>

        <record id="tax_group_telco" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">30</field>
            <field name="name">Impuestos a las Telecomunicaciones</field>
            <field eval="18" name="amount"/>
            <field name="amount_type">group</field>
            <field name="type_tax_use">purchase</field>
            <field eval="[(6, 0, [ref('tax_18_purch'), ref('tax_10_telco'), ref('tax_2_telco')])]" name="children_tax_ids"/>
            <field name="tax_group_id" ref="group_tax"/>
        </record>

        <record id="ret_100_tax_security" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">40</field>
            <field name="name">Retención 100% ITBIS Servicios de Seguridad (N07-09)</field>
            <field name="description">-100% ITBIS (N07-09)</field>
            <field name="amount">-18</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030201'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030201'),
                }),
            ]"/>
        </record>

        <record id="ret_100_tax_nonprofit" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">41</field>
            <field name="name">Retención 100% ITBIS Servicios No Lucrativas (N01-11)</field>
            <field name="description">-100% ITBIS (N01-11)</field>
            <field name="amount">-18</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_retcn_prsn')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030203'),
                    'minus_report_line_ids': [ref('account_tax_report_itbs_retcn_prsn')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_retcn_prsn')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030203'),
                    'plus_report_line_ids': [ref('account_tax_report_itbs_retcn_prsn')],
                }),
            ]"/>
        </record>

        <record id="ret_100_tax_person" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">42</field>
            <field name="name">Retención 100% ITBIS Servicios a Físicas (R293-11)</field>
            <field name="description">-100% ITBIS (R293-11)</field>
            <field name="amount">-18</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_retcn_prsn')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030202'),
                    'minus_report_line_ids': [ref('account_tax_report_itbs_retcn_prsn')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_retcn_prsn')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030202'),
                    'plus_report_line_ids': [ref('account_tax_report_itbs_retcn_prsn')],
                }),
            ]"/>
        </record>

        <record id="ret_30_tax_moral" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">43</field>
            <field name="name">Retención 30% ITBIS Servicios a Jurídicas (N02-05)</field>
            <field name="description">-30% ITBIS (N02-05)</field>
            <field name="amount">-5.4</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030201'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030201'),
                }),
            ]"/>
        </record>

        <record id="ret_30_tax_freelance" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="active">False</field>
            <field name="sequence">43</field>
            <field name="name">Retención 30% ITBIS Servicios Profesionales (N02-05)</field>
            <field name="description">-30% ITBIS (N02-05)</field>
            <field name="amount">-5.4</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">none</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030201'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030201'),
                }),
            ]"/>
        </record>

        <record id="ret_75_tax_nonformal" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">44</field>
            <field name="name">Retención 75% ITBIS Bienes a Informales (N08-10)</field>
            <field name="description">-75% ITBIS (N08-10)</field>
            <field name="amount">-13.5</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030205'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030205'),
                }),
            ]"/>
        </record>

        <record id="ret_10_income_person" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">40</field>
            <field name="name">Retención 10% ISR Honorarios a Físicas</field>
            <field name="description">-10% ISR</field>
            <field name="amount">-10</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_isr"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030301'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030301'),
                }),
            ]"/>
        </record>

        <record id="ret_10_income_rent" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">50</field>
            <field name="name">Retención 10% ISR Alquileres a Físicas</field>
            <field name="description">-10% ISR</field>
            <field name="amount">-10</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_isr"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030302'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030302'),
                }),
            ]"/>
        </record>

        <record id="ret_10_income_dividend" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">51</field>
            <field name="name">Retención 10% ISR por Dividendos (L253-12)</field>
            <field name="description">-10% ISR</field>
            <field name="amount">-10</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_isr"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030303'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030303'),
                }),
            ]"/>
        </record>

        <record id="ret_2_income_person" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">52</field>
            <field name="name">Retención 2% ISR a Física (con Materiales)</field>
            <field name="description">-2% ISR (N07-07)</field>
            <field name="amount">-2</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_isr"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030308'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030308'),
                }),
            ]"/>
        </record>

        <record id="ret_2_income_transfer" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">53</field>
            <field name="name">Retención 2% ISR por Transferencia de Títulos</field>
            <field name="description">-2% ISR</field>
            <field name="amount">-2</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_isr"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030306'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030306'),
                }),
            ]"/>
        </record>

        <record id="ret_27_income_remittance" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">49</field>
            <field name="name">Retención 27% ISR por Remesas al Exterior (L253-12)</field>
            <field name="description">-27% ISR</field>
            <field name="amount">-27</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_isr"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030307'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_21030307'),
                }),
            ]"/>
        </record>

        <record id="ret_5_income_gov" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">50</field>
            <field name="name">Retención 5% ISR Gubernamentales</field>
            <field name="description">-5% ISR</field>
            <field name="amount">-5</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_isr"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080302'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080302'),
                }),
            ]"/>
        </record>

        <record id="tax_group_nonformal" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">60</field>
            <field name="name">Retención a Proveedores Informales de Bienes (75%)</field>
            <field name="amount_type">group</field>
            <field eval="18" name="amount"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="group_ret"/>
            <field eval="[(6, 0, [ref('tax_18_purch'), ref('ret_75_tax_nonformal')])]" name="children_tax_ids"/>
        </record>

        <record id="tax_group_person_construction" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">61</field>
            <field name="name">Retención a Físicas por Servicios con Materiales (2%)</field>
            <field name="amount_type">group</field>
            <field eval="18" name="amount"/>
            <field name="type_tax_use">purchase</field>
            <field eval="[(6, 0, [ref('tax_18_purch'), ref('ret_100_tax_person'), ref('ret_2_income_person')])]" name="children_tax_ids"/>
            <field name="tax_group_id" ref="group_ret"/>
        </record>

        <record id="tax_group_person_services" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">58</field>
            <field name="name">Retención a Físicas por Honorarios por Servicios (10%)</field>
            <field name="amount_type">group</field>
            <field eval="18" name="amount"/>
            <field name="type_tax_use">purchase</field>
            <field eval="[(6, 0, [ref('tax_18_purch'), ref('ret_100_tax_person'), ref('ret_10_income_person')])]" name="children_tax_ids"/>
            <field name="tax_group_id" ref="group_ret"/>
        </record>

        <record id="tax_group_moral_services" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">58</field>
            <field name="name">Retención a Jurídicas por Servicios Profesionales (30%)</field>
            <field name="amount_type">group</field>
            <field eval="18" name="amount"/>
            <field name="type_tax_use">purchase</field>
            <field eval="[(6, 0, [ref('tax_18_purch'), ref('ret_30_tax_moral')])]" name="children_tax_ids"/>
            <field name="tax_group_id" ref="group_ret"/>
        </record>

        <record id="tax_group_restaurant_sale" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">64</field>
            <field name="name">Ventas del Restaurante</field>
            <field name="amount_type">group</field>
            <field eval="18" name="amount"/>
            <field name="type_tax_use">sale</field>
            <field eval="[(6, 0, [ref('tax_18_sale'), ref('tax_tip_sale')])]" name="children_tax_ids"/>
            <field name="tax_group_id" ref="group_ret"/>
        </record>

        <record id="tax_group_restaurant_purch" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="sequence">65</field>
            <field name="name">Compras a Restaurantes</field>
            <field name="amount_type">group</field>
            <field eval="18" name="amount"/>
            <field name="type_tax_use">purchase</field>
            <field eval="[(6, 0, [ref('tax_18_purch'), ref('tax_tip_purch')])]" name="children_tax_ids"/>
            <field name="tax_group_id" ref="group_ret"/>
        </record>

        <record id="tax_18_10_total_mount" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">18% ITBIS sobre el 10% del Monto Total</field>
            <field name="description">18% del 10%</field>
            <field name="amount">1.8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080102'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_11080102'),
                }),
            ]"/>
        </record>

        <record id="tax_18_property_cost" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">18% ITBIS llevado al Costo Bienes</field>
            <field name="description">18%</field>
            <field name="amount">18</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_51010500'),
                    'plus_report_line_ids': [ref('account_tax_report_itbs_pgdo_locales')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_51010500'),
                    'plus_report_line_ids': [ref('account_tax_report_itbs_pgdo_locales')],
                }),
            ]"/>
        </record>

        <record id="tax_18_serv_cost" model="account.tax.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">18% ITBIS llevado al Costo Servicios</field>
            <field name="description">18%</field>
            <field name="amount">18</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="price_include"/>
            <field name="tax_group_id" ref="group_itbis"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_51010500'),
                    'plus_report_line_ids': [ref('account_tax_report_itbs_pgdo_locales')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('do_niif_51010500'),
                    'plus_report_line_ids': [ref('account_tax_report_itbs_pgdo_locales')],
                }),
            ]"/>
        </record>

    </data>
</odoo>

```

## File: data\account_account_tag_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

    <!-- Account Tags -->

        <record id='account_tag_1' model='account.account.tag'>
                <field name='name'>1 Activos</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_11' model='account.account.tag'>
                <field name='name'>11 Activos Corrientes</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1101' model='account.account.tag'>
                <field name='name'>1101 Efectivo y Equivalentes de Efectivo</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_110101' model='account.account.tag'>
                <field name='name'>110101 Caja</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_110102' model='account.account.tag'>
                <field name='name'>110102 Bancos</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_110103' model='account.account.tag'>
                <field name='name'>110103 Inversiones Temporales Plazo Menor 90 días</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1102' model='account.account.tag'>
                <field name='name'>1102 Inversiones en Valores a Corto Plazo</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1103' model='account.account.tag'>
                <field name='name'>1103 Cuentas y Documentos por Cobrar</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_110301' model='account.account.tag'>
                <field name='name'>110301 Documentos por Cobrar</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_110302' model='account.account.tag'>
                <field name='name'>110302 Cuentas por Cobrar</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1104' model='account.account.tag'>
                <field name='name'>1104 Provisión para Cuentas Incobrables</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1105' model='account.account.tag'>
                <field name='name'>1105 Inventarios</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1106' model='account.account.tag'>
                <field name='name'>1106 Deterioro Acumulado de Valor de Inventarios</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1107' model='account.account.tag'>
                <field name='name'>1107 Estimación por Obsolecencia de Inventario</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1108' model='account.account.tag'>
                <field name='name'>1108 Impuestos Adelantados</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_110801' model='account.account.tag'>
                <field name='name'>110801 ITBIS Pagado en Compras</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_110803' model='account.account.tag'>
                <field name='name'>110803 Otros Impuestos y Saldos</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1109' model='account.account.tag'>
                <field name='name'>1109 Inversiones Temporales</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1110' model='account.account.tag'>
                <field name='name'>1110 Pagos Anticipados</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_12' model='account.account.tag'>
                <field name='name'>12 Activos Fijos</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1201' model='account.account.tag'>
                <field name='name'>1201 Propiedades, Planta y Equipo</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_120101' model='account.account.tag'>
                <field name='name'>120101 Bienes Inmuebles (CAT 1)</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_120102' model='account.account.tag'>
                <field name='name'>120102 Bienes Muebles (CAT 2)</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_120103' model='account.account.tag'>
                <field name='name'>120103 Bienes Muebles (CAT 3)</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1202' model='account.account.tag'>
                <field name='name'>1202 Depreciación Acumulada de Propiedades, Planta y Equipo</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1203' model='account.account.tag'>
                <field name='name'> 1203 Deterioro de Valor Acumulado de Propiedades, Planta y Equipo</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1204' model='account.account.tag'>
                <field name='name'>1204 Activos Intangibles</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1205' model='account.account.tag'>
                <field name='name'>1205 Bienes en Arrendamiento</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1206' model='account.account.tag'>
                <field name='name'>1206 Depreciación Acumulada de Bienes en Arrendamiento</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1207' model='account.account.tag'>
                <field name='name'>1207 Deterioro de Valor Acumulado de Bienes en Arrendamiento</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1208' model='account.account.tag'>
                <field name='name'>1208 Inversiones Permanentes</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_1209' model='account.account.tag'>
                <field name='name'>1209 Activos Diferidos</field>
                <field name='color'>1</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_2' model='account.account.tag'>
                <field name='name'>2 Pasivos</field>
                <field name='color'>8</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_21' model='account.account.tag'>
                <field name='name'>21 Pasivo Corriente</field>
                <field name='color'>8</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_2101' model='account.account.tag'>
                <field name='name'>2101 Cuentas y Documentos por Pagar a Corto Plazo</field>
                <field name='color'>8</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_2102' model='account.account.tag'>
                <field name='name'>2102 Beneficios por Pagar a Corto Plazo</field>
                <field name='color'>8</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_2103' model='account.account.tag'>
                <field name='name'>2103 Impuestos y Retenciones</field>
                <field name='color'>8</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_210301' model='account.account.tag'>
                <field name='name'>210301 ITBIS por Pagar</field>
                <field name='color'>8</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_210302' model='account.account.tag'>
                <field name='name'>210302 ITBIS Retenido</field>
                <field name='color'>8</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_210303' model='account.account.tag'>
                <field name='name'>210303 ISR Retenido</field>
                <field name='color'>8</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_210304' model='account.account.tag'>
                <field name='name'>210304 Retenciones en Nómina</field>
                <field name='color'>8</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_210305' model='account.account.tag'>
                <field name='name'>210305 Otros Impuestos o Retenciones</field>
                <field name='color'>8</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_2104' model='account.account.tag'>
                <field name='name'>2104 Provisiones a Corto Plazo</field>
                <field name='color'>8</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_2105' model='account.account.tag'>
                <field name='name'>2105 Tarjetas de Crédito</field>
                <field name='color'>8</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_22' model='account.account.tag'>
                <field name='name'>22 Pasivo No Corriente</field>
                <field name='color'>8</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_2201' model='account.account.tag'>
                <field name='name'>2201 Cuentas y Documentos por Pagar a Largo Plazo</field>
                <field name='color'>8</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_2202' model='account.account.tag'>
                <field name='name'>2202 Provisión para Obligaciones Laborales</field>
                <field name='color'>8</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_2203' model='account.account.tag'>
                <field name='name'>2203 Anticipos y Garantías de Clientes</field>
                <field name='color'>8</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_2204' model='account.account.tag'>
                <field name='name'>2204 Provisiones a Largo Plazo</field>
                <field name='color'>8</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_3' model='account.account.tag'>
                <field name='name'>3 Capital</field>
                <field name='color'>6</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_31' model='account.account.tag'>
                <field name='name'>31 Capital Contable</field>
                <field name='color'>6</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_3101' model='account.account.tag'>
                <field name='name'>3101 Capital Social</field>
                <field name='color'>6</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_3102' model='account.account.tag'>
                <field name='name'>3102 Superávit por Revaluación de Activos</field>
                <field name='color'>6</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_32' model='account.account.tag'>
                <field name='name'>32 Utilidades Restringidas</field>
                <field name='color'>6</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_3201' model='account.account.tag'>
                <field name='name'>3201 Reserva Legal</field>
                <field name='color'>6</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_33' model='account.account.tag'>
                <field name='name'>33 Resultados</field>
                <field name='color'>6</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_3301' model='account.account.tag'>
                <field name='name'>3301 Resultados Acumulados</field>
                <field name='color'>6</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_3302' model='account.account.tag'>
                <field name='name'>3302 Resultados del Ejercicio</field>
                <field name='color'>6</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_3304' model='account.account.tag'>
                <field name='name'>3304 Otras Reservas de Patrimonio</field>
                <field name='color'>6</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_4' model='account.account.tag'>
                <field name='name'>4 Ingresos y Ganancias</field>
                <field name='color'>7</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_41' model='account.account.tag'>
                <field name='name'>41 Ingresos por Operaciones</field>
                <field name='color'>7</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_4101' model='account.account.tag'>
                <field name='name'>4101 Ventas de Bienes</field>
                <field name='color'>7</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_4102' model='account.account.tag'>
                <field name='name'>4102 Ventas de Servicios</field>
                <field name='color'>7</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_4103' model='account.account.tag'>
                <field name='name'>4103 Devoluciones</field>
                <field name='color'>7</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_4104' model='account.account.tag'>
                <field name='name'>4104 Descuentos</field>
                <field name='color'>7</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_42' model='account.account.tag'>
                <field name='name'>42 Ingresos No Operacionales</field>
                <field name='color'>7</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_4201' model='account.account.tag'>
                <field name='name'>4201 Intereses Ganados</field>
                <field name='color'>7</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_4202' model='account.account.tag'>
                <field name='name'>4202 Ventas de Activos</field>
                <field name='color'>7</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_4203' model='account.account.tag'>
                <field name='name'>4203 Dividendos Ganados</field>
                <field name='color'>7</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_4204' model='account.account.tag'>
                <field name='name'>4204 Ingresos Extraordinarios</field>
                <field name='color'>7</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_5' model='account.account.tag'>
                <field name='name'>5 Costos Directos e Indirectos</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_51' model='account.account.tag'>
                <field name='name'>51 Costos y Gastos de Operación</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_5101' model='account.account.tag'>
                <field name='name'>5101 Costos de Ventas</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_5102' model='account.account.tag'>
                <field name='name'>5102 Costos de Producción</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_6' model='account.account.tag'>
                <field name='name'>6 Gastos y Pérdidas</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_52' model='account.account.tag'>
                <field name='name'>61 Gastos de Operación</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_5201' model='account.account.tag'>
                <field name='name'>6101 Gastos de Personal</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_520101' model='account.account.tag'>
                <field name='name'>610101 Aportes a la Seguridad Social</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_520102' model='account.account.tag'>
                <field name='name'>610102 Otras Cargas Patronales</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_5202' model='account.account.tag'>
                <field name='name'>6102 Gastos de Administración</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_5203' model='account.account.tag'>
                <field name='name'>6103 Gastos por Trabajo, Suministros y Servicios</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_520301' model='account.account.tag'>
                <field name='name'>610301 Gastos Honorarios por Servicios Profesionales (P. Física)</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_520302' model='account.account.tag'>
                <field name='name'>610302 Gastos Honorarios por Servicios Profesionales (P. Jurídica)</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_520303' model='account.account.tag'>
                <field name='name'>610303 Gastos Honorarios por Servicios Profesionales (P. Jurídica)</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_5204' model='account.account.tag'>
                <field name='name'>6104 Gastos por Depreciación</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_5205' model='account.account.tag'>
                <field name='name'>6105 Gastos por Reparaciones</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_5206' model='account.account.tag'>
                <field name='name'>6106 Gastos de Representación</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_5207' model='account.account.tag'>
                <field name='name'>6107 Gastos Financieros</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_5208' model='account.account.tag'>
                <field name='name'>6108 Gastos Extraordinarios</field>
                <field name='color'>4</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_7' model='account.account.tag'>
                <field name='name'>7 Cuentas Liquidadoras de Resultados</field>
                <field name='color'>5</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_61' model='account.account.tag'>
                <field name='name'>71 Cuenta Liquidadora</field>
                <field name='color'>5</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_6101' model='account.account.tag'>
                <field name='name'>7101 Pérdidas y Ganancias</field>
                <field name='color'>5</field>
                <field name='applicability'>accounts</field>
        </record>

        <record id='account_tag_6102' model='account.account.tag'>
                <field name='name'>7102 Gastos por Impuestos</field>
                <field name='color'>5</field>
                <field name='applicability'>accounts</field>
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
            <value eval="[ref('l10n_do.do_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Chart Template for Defaults -->
    <record id="do_chart_template" model="account.chart.template">
        <field eval="True" name="use_anglo_saxon"/>
        <field name="property_account_receivable_id" ref="do_niif_11030201"/>
        <field name="property_account_payable_id" ref="do_niif_21010200"/>
        <field name="property_account_income_categ_id" ref="do_niif_41010100"/>
        <field name="property_account_expense_categ_id" ref="do_niif_51010100"/>
        <field name="property_stock_account_input_categ_id" ref="do_niif_21021200"/>
        <field name="property_stock_account_output_categ_id" ref="do_niif_11050600"/>
        <field name="property_stock_valuation_account_id" ref="do_niif_11050100"/>
        <field name="expense_currency_exchange_account_id" ref="do_niif_52070800"/>
        <field name="income_currency_exchange_account_id" ref="do_niif_42040100"/>
        <field name="default_pos_receivable_account_id" ref="do_niif_11030210" />
    </record>
</odoo>

```

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <!-- Account Tax Group -->
        <record id="group_itbis" model="account.tax.group">
            <field name="name">ITBIS</field>
        </record>
        <record id="group_isr" model="account.tax.group">
            <field name="name">ISR</field>
        </record>
        <record id="group_ret" model="account.tax.group">
            <field name="name">Retenciones</field>
        </record>
        <record id="group_tax" model="account.tax.group">
            <field name="name">Otros Impuestos</field>
        </record>
        <record id="tax_group_isc" model="account.tax.group">
            <field name="name">ISC</field>
        </record>
        <record id="tax_group_itbis_0" model="account.tax.group">
            <field name="name">Exento</field>
        </record>
        <record id="tax_group_tip" model="account.tax.group">
            <field name="name">Propina</field>
        </record>
        <record id="tax_group_itbis_18" model="account.tax.group">
            <field name="name">ITBIS 18%</field>
        </record>
        <record id="tax_group_itbis_00015" model="account.tax.group">
            <field name="name">ITBIS 0.0015%</field>
        </record>
        <record id="tax_group_retencion_2" model="account.tax.group">
            <field name="name">ISR -2%</field>
        </record>
        <record id="tax_group_retencion_10" model="account.tax.group">
            <field name="name">ISR -10%</field>
        </record>
        <record id="tax_group_retencion_27" model="account.tax.group">
            <field name="name">ISR -27%</field>
        </record>
        <record id="tax_group_retencion_54" model="account.tax.group">
            <field name="name">ITBIS -30%</field>
        </record>
        <record id="tax_group_retencion_18" model="account.tax.group">
            <field name="name">ITBIS -100%</field>
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
            <field name="name">Activos</field>
            <field name="code_prefix">1</field>
        </record>
        <record id="account_group_11" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_1"/>
            <field name="name">Activos Corrientes</field>
            <field name="code_prefix">11</field>
        </record>
        <record id="account_group_1101" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_11"/>
            <field name="name">Efectivo y Equivalentes de Efectivo</field>
            <field name="code_prefix">1101</field>
        </record>
        <record id="account_group_110101" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_1101"/>
            <field name="name">Caja</field>
            <field name="code_prefix">110101</field>
        </record>
        <record id="account_group_110102" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_1101"/>
            <field name="name">Bancos</field>
            <field name="code_prefix">110102</field>
        </record>
        <record id="account_group_110103" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_1101"/>
            <field name="name">Inversiones Temporales Plazo Menor 90 días</field>
            <field name="code_prefix">110103</field>
        </record>
        <record id="account_group_1102" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_11"/>
            <field name="name">Inversiones en Valores a Corto Plazo</field>
            <field name="code_prefix">1102</field>
        </record>
        <record id="account_group_110301" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_11"/>
            <field name="name">Cuentas y Documentos por Cobrar</field>
            <field name="code_prefix">1103</field>
        </record>
        <record id="account_group_documentosporcobrar" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_110301"/>
            <field name="name">Documentos por Cobrar</field>
            <field name="code_prefix">110301</field>
        </record>
        <record id="account_group_110302" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_110301"/>
            <field name="name">Cuentas por Cobrar</field>
            <field name="code_prefix">110302</field>
        </record>
        <record id="account_group_1104" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_11"/>
            <field name="name">Provisión para Cuentas Incobrables</field>
            <field name="code_prefix">1104</field>
        </record>
        <record id="account_group_1105" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_11"/>
            <field name="name">Inventarios</field>
            <field name="code_prefix">1105</field>
        </record>
        <record id="account_group_1106" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_11"/>
            <field name="name">Deterioro Acumulado de Valor de Inventarios</field>
            <field name="code_prefix">1106</field>
        </record>
        <record id="account_group_1107" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_11"/>
            <field name="name">Estimación por Obsolencia de Inventario</field>
            <field name="code_prefix">1107</field>
        </record>
        <record id="account_group_1108" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_11"/>
            <field name="name">Impuestos Adelantados</field>
            <field name="code_prefix">1108</field>
        </record>
        <record id="account_group_110801" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_1108"/>
            <field name="name">ITBIS Adelantado en Compras</field>
            <field name="code_prefix">110801</field>
        </record>
        <record id="account_group_110803" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_1108"/>
            <field name="name">Otros Impuestos y Saldos</field>
            <field name="code_prefix">110803</field>
        </record>
        <record id="account_group_1109" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_11"/>
            <field name="name">Inversiones Temporales</field>
            <field name="code_prefix">1109</field>
        </record>
        <record id="account_group_1110" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_11"/>
            <field name="name">Pagos Anticipados</field>
            <field name="code_prefix">1110</field>
        </record>
        <record id="account_group_12" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_1"/>
            <field name="name">Activo Fijos</field>
            <field name="code_prefix">12</field>
        </record>
        <record id="account_group_1201" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_12"/>
            <field name="name">Propiedades, Planta y Equipo</field>
            <field name="code_prefix">1201</field>
        </record>
        <record id="account_group_120101" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_1201"/>
            <field name="name">Bienes Inmuebles (CAT 1)</field>
            <field name="code_prefix">120101</field>
        </record>
        <record id="account_group_120102" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_1201"/>
            <field name="name">Bienes Muebles (CAT 2)</field>
            <field name="code_prefix">120102</field>
        </record>
        <record id="account_group_120103" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_1201"/>
            <field name="name">Bienes Muebles (CAT 3)</field>
            <field name="code_prefix">120103</field>
        </record>
        <record id="account_group_1202" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_12"/>
            <field name="name">Depreciación Acumulada de Propiedades, Planta y Equipo</field>
            <field name="code_prefix">1202</field>
        </record>
        <record id="account_group_1203" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_12"/>
            <field name="name">Deterioro de Valor Acumulado de Propiedades, Planta y Equipo</field>
            <field name="code_prefix">1203</field>
        </record>
        <record id="account_group_1204" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_12"/>
            <field name="name">Activos Intangibles</field>
            <field name="code_prefix">1204</field>
        </record>
        <record id="account_group_1205" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_12"/>
            <field name="name">Bienes en Arrendamiento</field>
            <field name="code_prefix">1205</field>
        </record>
        <record id="account_group_1206" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_12"/>
            <field name="name">Depreciación Acumulada de Bienes en Arrendamiento</field>
            <field name="code_prefix">1206</field>
        </record>
        <record id="account_group_1207" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_12"/>
            <field name="name">Deterioro de Valor Acumulado de Bienes en Arrendamiento</field>
            <field name="code_prefix">1207</field>
        </record>
        <record id="account_group_1208" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_12"/>
            <field name="name">Inversiones Permanentes</field>
            <field name="code_prefix">1208</field>
        </record>
        <record id="account_group_1209" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_12"/>
            <field name="name">Activos Diferidos</field>
            <field name="code_prefix">1209</field>
        </record>
        <record id="account_group_13" model="account.group">
            <field name="name">Costos de Construcción</field>
            <field name="code_prefix">13</field>
        </record>
        <record id="account_group_1301" model="account.group">
            <field name="name">Construcción en Proceso</field>
            <field name="code_prefix">1301</field>
        </record>
        <record id="account_group_130101" model="account.group">
            <field name="name">Costos de Costrucción en Proceso</field>
            <field name="code_prefix">130101</field>
        </record>
        <record id="account_group_2" model="account.group">
            <field name="name">Pasivos</field>
            <field name="code_prefix">2</field>
        </record>
        <record id="account_group_21" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_2"/>
            <field name="name">Pasivo Corriente</field>
            <field name="code_prefix">21</field>
        </record>
        <record id="account_group_2101" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_21"/>
            <field name="name">Cuentas y Documentos por Pagar a Corto Plazo</field>
            <field name="code_prefix">2101</field>
        </record>
        <record id="account_group_2102" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_21"/>
            <field name="name">Beneficios por Pagar a Corto Plazo</field>
            <field name="code_prefix">2102</field>
        </record>
        <record id="account_group_2103" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_21"/>
            <field name="name">Impuestos y Retenciones</field>
            <field name="code_prefix">2103</field>
        </record>
        <record id="account_group_210301" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_2103"/>
            <field name="name">ITBIS por Pagar</field>
            <field name="code_prefix">210301</field>
        </record>
        <record id="account_group_210302" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_2103"/>
            <field name="name">ITBIS Retenido</field>
            <field name="code_prefix">210302</field>
        </record>
        <record id="account_group_210303" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_2103"/>
            <field name="name">ISR Retenido</field>
            <field name="code_prefix">210303</field>
        </record>
        <record id="account_group_210304" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_2103"/>
            <field name="name">Retenciones en Nómina</field>
            <field name="code_prefix">210304</field>
        </record>
        <record id="account_group_210305" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_2103"/>
            <field name="name">Otros Impuestos o Retenciones</field>
            <field name="code_prefix">210305</field>
        </record>
        <record id="account_group_2104" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_21"/>
            <field name="name">Provisiones a Corto Plazo</field>
            <field name="code_prefix">2104</field>
        </record>
        <record id="account_group_2105" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_21"/>
            <field name="name">Tarjetas de Crédito</field>
            <field name="code_prefix">2105</field>
        </record>
        <record id="account_group_22" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_2"/>
            <field name="name">Pasivo No Corriente</field>
            <field name="code_prefix">22</field>
        </record>
        <record id="account_group_2201" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_22"/>
            <field name="name">Cuentas y Documentos por Pagar a Largo Plazo</field>
            <field name="code_prefix">2201</field>
        </record>
        <record id="account_group_2202" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_22"/>
            <field name="name">Provisión para Obligaciones Laborales</field>
            <field name="code_prefix">2202</field>
        </record>
        <record id="account_group_2203" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_22"/>
            <field name="name">Anticipos de Clientes y Pagos no Identificados</field>
            <field name="code_prefix">2203</field>
        </record>
        <record id="account_group_2204" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_22"/>
            <field name="name">Provisiones a Largo Plazo</field>
            <field name="code_prefix">2204</field>
        </record>
        <record id="account_group_3" model="account.group">
            <field name="name">Capital</field>
            <field name="code_prefix">3</field>
        </record>
        <record id="account_group_31" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_3"/>
            <field name="name">Capital Contable</field>
            <field name="code_prefix">31</field>
        </record>
        <record id="account_group_3101" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_31"/>
            <field name="name">Capital Social</field>
            <field name="code_prefix">3101</field>
        </record>
        <record id="account_group_3102" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_31"/>
            <field name="name">Superávit por Revaluación de Activos</field>
            <field name="code_prefix">3102</field>
        </record>
        <record id="account_group_32" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_3"/>
            <field name="name">Utilidades Restringidas</field>
            <field name="code_prefix">32</field>
        </record>
        <record id="account_group_3201" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_32"/>
            <field name="name">Reserva Legal</field>
            <field name="code_prefix">3201</field>
        </record>
        <record id="account_group_33" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_3"/>
            <field name="name">Resultados</field>
            <field name="code_prefix">33</field>
        </record>
        <record id="account_group_3301" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_33"/>
            <field name="name">Resultados Acumulados</field>
            <field name="code_prefix">3301</field>
        </record>
        <record id="account_group_3302" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_33"/>
            <field name="name">Resultados del Ejercicio</field>
            <field name="code_prefix">3302</field>
        </record>
        <record id="account_group_3304" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_33"/>
            <field name="name">Otras Reservas de Patrimonio</field>
            <field name="code_prefix">3304</field>
        </record>
        <record id="account_group_4" model="account.group">
            <field name="name">Ingresos y Ganancias</field>
            <field name="code_prefix">4</field>
        </record>
        <record id="account_group_41" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_4"/>
            <field name="name">Ingresos por Operaciones</field>
            <field name="code_prefix">41</field>
        </record>
        <record id="account_group_4101" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_41"/>
            <field name="name">Ventas de Bienes</field>
            <field name="code_prefix">4101</field>
        </record>
        <record id="account_group_4102" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_41"/>
            <field name="name">Ventas de Servicios</field>
            <field name="code_prefix">4102</field>
        </record>
        <record id="account_group_4103" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_41"/>
            <field name="name">Devoluciones</field>
            <field name="code_prefix">4103</field>
        </record>
        <record id="account_group_4104" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_41"/>
            <field name="name">Descuentos</field>
            <field name="code_prefix">4104</field>
        </record>
        <record id="account_group_42" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_41"/>
            <field name="name">Ingresos No Operacionales</field>
            <field name="code_prefix">42</field>
        </record>
        <record id="account_group_4201" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_4"/>
            <field name="name">Intereses Ganados</field>
            <field name="code_prefix">4201</field>
        </record>
        <record id="account_group_4202" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_42"/>
            <field name="name">Ventas de Activos</field>
            <field name="code_prefix">4202</field>
        </record>
        <record id="account_group_4203" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_42"/>
            <field name="name">Dividendos Ganados</field>
            <field name="code_prefix">4203</field>
        </record>
        <record id="account_group_4204" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_42"/>
            <field name="name">Ingresos Extraordinarios</field>
            <field name="code_prefix">4204</field>
        </record>
        <record id="account_group_5" model="account.group">
            <field name="name">Costos, Gastos y Pérdidas</field>
            <field name="code_prefix">5</field>
        </record>
        <record id="account_group_51" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_5"/>
            <field name="name">Costos de Operación</field>
            <field name="code_prefix">51</field>
        </record>
        <record id="account_group_5101" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_51"/>
            <field name="name">Costos de Ventas</field>
            <field name="code_prefix">5101</field>
        </record>
        <record id="account_group_5102" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_51"/>
            <field name="name">Costos de Producción</field>
            <field name="code_prefix">5102</field>
        </record>
        <record id="account_group_6" model="account.group">
            <field name="name">Gastos y Pérdidas</field>
            <field name="code_prefix">6</field>
        </record>
        <record id="account_group_61" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_6"/>
            <field name="name">Gastos de Operación</field>
            <field name="code_prefix">61</field>
        </record>
        <record id="account_group_6101" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_61"/>
            <field name="name">Gastos de Personal</field>
            <field name="code_prefix">6101</field>
        </record>
        <record id="account_group_610101" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_6101"/>
            <field name="name">Aportes a la Seguridad Social</field>
            <field name="code_prefix">610101</field>
        </record>
        <record id="account_group_610102" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_6101"/>
            <field name="name">Otras Cargas Patronales</field>
            <field name="code_prefix">610102</field>
        </record>
        <record id="account_group_6102" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_61"/>
            <field name="name">Gastos de Administración</field>
            <field name="code_prefix">6102</field>
        </record>
        <record id="account_group_6103" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_61"/>
            <field name="name">Gastos por Trabajo, Suministros y Servicios</field>
            <field name="code_prefix">6103</field>
        </record>
        <record id="account_group_610301" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_6103"/>
            <field name="name">Gastos Honorarios por Servicios Profesionales (P. Física)</field>
            <field name="code_prefix">610301</field>
        </record>
        <record id="account_group_610302" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_6103"/>
            <field name="name">Gastos Honorarios por Servicios Profesionales (P. Jurídica)</field>
            <field name="code_prefix">610302</field>
        </record>
        <record id="account_group_610303" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_6103"/>
            <field name="name">Gastos Honorarios por Servicios Profesionales (P. Jurídica)</field>
            <field name="code_prefix">610303</field>
        </record>
        <record id="account_group_6104" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_61"/>
            <field name="name">Gastos por Depreciación</field>
            <field name="code_prefix">6104</field>
        </record>
        <record id="account_group_6105" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_61"/>
            <field name="name">Gastos por Reparaciones</field>
            <field name="code_prefix">6105</field>
        </record>
        <record id="account_group_6106" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_61"/>
            <field name="name">Gastos de Representación</field>
            <field name="code_prefix">6106</field>
        </record>
        <record id="account_group_6107" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_61"/>
            <field name="name">Gastos Financieros</field>
            <field name="code_prefix">6107</field>
        </record>
        <record id="account_group_6108" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_61"/>
            <field name="name">Gastos Extraordinarios</field>
            <field name="code_prefix">6108</field>
        </record>
        <record id="account_group_7" model="account.group">
            <field name="name">Cuentas Liquidadoras de Resultados</field>
            <field name="code_prefix">7</field>
        </record>
        <record id="account_group_71" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_7"/>
            <field name="name">Cuenta Liquidadora</field>
            <field name="code_prefix">71</field>
        </record>
        <record id="account_group_7101" model="account.group">
            <field name="parent_id" ref="l10n_do.account_group_71"/>
            <field name="name">Pérdidas y Ganancias</field>
            <field name="code_prefix">7101</field>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="account_tax_report_total_operaciones" model="account.tax.report.line">
        <field name="name">II-1 Total de Operaciones</field>
        <field name="sequence" eval="1"/>
        <field name="country_id" ref="base.do"/>
    </record>

    <record id="account_tax_report_2A_ingrs_exprt" model="account.tax.report.line">
        <field name="name">II.A INGRESOS POR EXPORTACIONES DE BIENES O SERVICIOS EXENTOS</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_total_operaciones"/>
        <field name="country_id" ref="base.do"/>
    </record>

    <record id="account_tax_report_2A2_ingrs_exprt" model="account.tax.report.line">
        <field name="name">II.A.2 INGRESOS POR EXPORTACIONES DE BIENES O SERVICIOS EXENTOS</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_2A_ingrs_exprt"/>
        <field name="country_id" ref="base.do"/>
    </record>

    <record id="account_tax_report_2A3_ingrs_local_vnts" model="account.tax.report.line">
        <field name="name">II.A.3 INGRESOS POR VENTAS LOCALES DE BIENES O SERVICIOS EXENTOS</field>
        <field name="tag_name">II.A.3</field>
        <field name="code">DOTAX010102</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_2A_ingrs_exprt"/>
        <field name="country_id" ref="base.do"/>
    </record>

    <record id="account_tax_report_2b_grvd" model="account.tax.report.line">
        <field name="name">II.B GRAVADAS</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_total_operaciones"/>
        <field name="country_id" ref="base.do"/>
    </record>

    <record id="account_tax_report_2b6_grvd" model="account.tax.report.line">
        <field name="name">II.B.6 OPERACIONES GRAVADAS AL 18%</field>
        <field name="tag_name">II.B.6</field>
        <field name="code">DOTAX010201</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_2b_grvd"/>
        <field name="country_id" ref="base.do"/>
    </record>

    <record id="account_tax_report_2b7_grvd" model="account.tax.report.line">
        <field name="name">II.B.7 OPERACIONES GRAVADAS AL 11%</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_2b_grvd"/>
        <field name="country_id" ref="base.do"/>
    </record>

    <record id="account_tax_report_3_liquidacion" model="account.tax.report.line">
        <field name="name">III LIQUIDACION</field>
        <field name="sequence" eval="2"/>
        <field name="country_id" ref="base.do"/>
    </record>

    <record id="account_financial_report_line_02_01_do account_tax_report_total_itbs" model="account.tax.report.line">
        <field name="name">III.10 TOTAL ITBIS COBRADO (Sumar casillas 8+9)</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_3_liquidacion"/>
        <field name="country_id" ref="base.do"/>
    </record>

    <record id="account_tax_report_itbs_18_casilla" model="account.tax.report.line">
        <field name="name">III.8 ITBIS COBRADO (18% de la casilla 6)</field>
        <field name="tag_name">III.8</field>
        <field name="code">DOTAX020101</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_financial_report_line_02_01_do account_tax_report_total_itbs"/>
        <field name="country_id" ref="base.do"/>
    </record>

    <record id="account_tax_report_tbs_11_casilla" model="account.tax.report.line">
        <field name="name">III.9 TBIS COBRADO (11% de la casilla 7)</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_financial_report_line_02_01_do account_tax_report_total_itbs"/>
        <field name="country_id" ref="base.do"/>
    </record>

    <record id="account_tax_report_total_itbs_pgdo" model="account.tax.report.line">
        <field name="name">III.14 TOTAL ITBIS PAGADO (Sumar casillas 11+12+13)</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_3_liquidacion"/>
        <field name="country_id" ref="base.do"/>
    </record>

    <record id="account_tax_report_itbs_pgdo_locales" model="account.tax.report.line">
        <field name="name">III.11 ITBIS PAGADO EN COMPRAS LOCALES</field>
        <field name="tag_name">III.11</field>
        <field name="code">DOTAX020201</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_total_itbs_pgdo"/>
        <field name="country_id" ref="base.do"/>
    </record>

    <record id="account_tax_report_itbs_pgdo_dedubl" model="account.tax.report.line">
        <field name="name">III.12 ITBIS PAGADO POR SERVICIOS DEDUCIBLES</field>
        <field name="tag_name">III.12</field>
        <field name="code">DOTAX020202</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_total_itbs_pgdo"/>
        <field name="country_id" ref="base.do"/>
    </record>

    <record id="account_tax_report_itbs_pgdo_imptcn" model="account.tax.report.line">
        <field name="name">III.13 ITBIS PAGADO EN IMPORTACIONES</field>
        <field name="tag_name">III.13</field>
        <field name="code">DOTAX020203</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_total_itbs_pgdo"/>
        <field name="country_id" ref="base.do"/>
    </record>

    <record id="account_tax_report_itbs_rntnd" model="account.tax.report.line">
        <field name="name">A,1 ITBIS RETENIDO</field>
        <field name="sequence" eval="3"/>
        <field name="country_id" ref="base.do"/>
    </record>

    <record id="account_tax_report_retcn_prsn" model="account.tax.report.line">
        <field name="name">A.25 Servicios Sujetos a Retención Personas = Físicas y Entidad</field>
        <field name="tag_name">A.25</field>
        <field name="code">DOTAX0301</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_itbs_rntnd"/>
        <field name="country_id" ref="base.do"/>
    </record>

    <record id="account_tax_report_itbs_retcn_prsn" model="account.tax.report.line">
        <field name="name">A.30 Itbis Por Servicios Sujetos A Retencion Personas Físicas</field>
        <field name="tag_name">A.30</field>
        <field name="code">DOTAX0302</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_itbs_rntnd"/>
        <field name="country_id" ref="base.do"/>
    </record>

</odoo>
```

## File: data\fiscal_position_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- = = = = = = = = = = = = = = = -->
        <!-- Fiscal Position Templates     -->
        <!-- = = = = = = = = = = = = = = = -->
        <!-- Principal Fiscal Position for Dominican Republic internally -->
        <record id="position_person" model="account.fiscal.position.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">P. Física de Servicios</field>
        </record>
        <record id="position_service_moral" model="account.fiscal.position.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">P. Jurídica de Servicios</field>
        </record>
        <record id="position_security_moral" model="account.fiscal.position.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">P. Jurídica de Vigilancia</field>
        </record>
        <record id="position_nonformal" model="account.fiscal.position.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">Proveedor Informal de Bienes</field>
        </record>
        <record id="position_exterior" model="account.fiscal.position.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">Servicios del Exterior</field>
        </record>
        <record id="position_gov" model="account.fiscal.position.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">Gubernamental</field>
        </record>
        <record id="position_nonprofit" model="account.fiscal.position.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">No Lucrativa de Servicios</field>
        </record>
        <record id="position_especial" model="account.fiscal.position.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">Regímenes Especiales</field>
        </record>
        <record id="position_restaurant" model="account.fiscal.position.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">Restaurantes</field>
        </record>
        <record id="position_restaurant_takeout" model="account.fiscal.position.template">
            <field name="chart_template_id" ref="do_chart_template"/>
            <field name="name">Para Llevar</field>
        </record>
        <!-- = = = = = = = = = = = = = = = -->
        <!-- Fiscal Position Tax Templates -->
        <!-- = = = = = = = = = = = = = = = -->
        <!-- Locales -->
        <!-- Proveedor Informal Bienes -->
        <record id="fiscal_position_tax_3" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_18_purch"/>
            <field name="position_id" ref="position_nonformal"/>
            <field name="tax_dest_id" ref="tax_group_nonformal"/>
        </record>
        <record id="fiscal_position_tax_4" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_18_purch_incl"/>
            <field name="position_id" ref="position_nonformal"/>
            <field name="tax_dest_id" ref="tax_group_nonformal"/>
        </record>
        <!-- Persona Física de Servicios>-->
        <record id="fiscal_position_tax_9" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_18_purch"/>
            <field name="position_id" ref="position_person"/>
            <field name="tax_dest_id" ref="tax_group_person_services"/>
        </record>
        <record id="fiscal_position_tax_10" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_18_purch_serv"/>
            <field name="position_id" ref="position_person"/>
            <field name="tax_dest_id" ref="tax_group_person_services"/>
        </record>
        <!-- Gubernamental -->
        <record id="fiscal_position_tax_1" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_18_sale"/>
            <field name="position_id" ref="position_gov"/>
            <field name="tax_dest_id" ref="ret_5_income_gov"/>
        </record>
        <record id="fiscal_position_tax_2" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_18_sale_incl"/>
            <field name="position_id" ref="position_gov"/>
            <field name="tax_dest_id" ref="ret_5_income_gov"/>
        </record>
        <!-- No Lucrativas -->
        <record id="fiscal_position_tax_12" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_18_purch"/>
            <field name="position_id" ref="position_nonprofit"/>
            <field name="tax_dest_id" ref="ret_100_tax_nonprofit"/>
        </record>
        <record id="fiscal_position_tax_13" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_18_purch_incl"/>
            <field name="position_id" ref="position_nonprofit"/>
            <field name="tax_dest_id" ref="ret_100_tax_nonprofit"/>
        </record>
        <!-- Proveedor Moral de Seguridad -->
        <record id="fiscal_position_tax_8" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_18_purch"/>
            <field name="position_id" ref="position_security_moral"/>
            <field name="tax_dest_id" ref="ret_100_tax_security"/>
        </record>
        <!-- Proveedor Moral de Servicio -->
        <record id="fiscal_position_tax_7" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_18_purch"/>
            <field name="position_id" ref="position_service_moral"/>
            <field name="tax_dest_id" ref="tax_group_moral_services"/>
        </record>
        <!-- Importación / Exportación -->
        <!-- Proveedor del Exterior -->
        <record id="fiscal_position_tax_5" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_18_purch_serv"/>
            <field name="position_id" ref="position_exterior"/>
            <field name="tax_dest_id" ref="ret_27_income_remittance"/>
        </record>
        <record id="fiscal_position_tax_6" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_18_purch"/>
            <field name="position_id" ref="position_exterior"/>
            <field name="tax_dest_id" ref="ret_27_income_remittance"/>
        </record>
        <!-- Restaurantes -->
        <record id="fiscal_position_tax_14" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_18_purch"/>
            <field name="position_id" ref="position_restaurant"/>
            <field name="tax_dest_id" ref="tax_group_restaurant_purch"/>
        </record>
        <record id="fiscal_position_tax_15" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_18_purch_incl"/>
            <field name="position_id" ref="position_restaurant"/>
            <field name="tax_dest_id" ref="tax_group_restaurant_purch"/>
        </record>
        <record id="fiscal_position_tax_16" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_18_sale"/>
            <field name="position_id" ref="position_restaurant"/>
            <field name="tax_dest_id" ref="tax_group_restaurant_sale"/>
        </record>
        <record id="fiscal_position_tax_17" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_18_sale_incl"/>
            <field name="position_id" ref="position_restaurant"/>
            <field name="tax_dest_id" ref="tax_group_restaurant_sale"/>
        </record>

        <!-- Restaurantes Take-Out -->
        <record id="fiscal_position_tax_18" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_group_restaurant_sale"/>
            <field name="position_id" ref="position_restaurant_takeout"/>
            <field name="tax_dest_id" ref="tax_18_sale"/>
        </record>

        <!-- Regímenes Especiales -->
        <record id="fiscal_position_tax_19" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_18_sale"/>
            <field name="position_id" ref="position_especial"/>
            <field name="tax_dest_id" ref="tax_0_sale"/>
        </record>
        <record id="fiscal_position_tax_20" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_18_sale_incl"/>
            <field name="position_id" ref="position_especial"/>
            <field name="tax_dest_id" ref="tax_0_sale"/>
        </record>
        <record id="fiscal_position_tax_21" model="account.fiscal.position.tax.template">
            <field name="tax_src_id" ref="tax_group_restaurant_sale"/>
            <field name="position_id" ref="position_especial"/>
            <field name="tax_dest_id" ref="tax_tip_sale"/>
        </record>

        <!-- = = = = = = = = = = = = = = = = = -->
        <!-- Fiscal Position Accounts Template -->
        <!-- = = = = = = = = = = = = = = = = = -->
        <!-- Locales -->
        <!-- Persona Física de Servicios -->
        <record id="fiscal_position_account_2" model="account.fiscal.position.account.template">
            <field name="position_id" ref="position_person"/>
            <field name="account_dest_id" ref="do_niif_52030111"/>
            <field name="account_src_id" ref="do_niif_52021500"/>
        </record>
        <!-- Importación / Exportación -->
        <!-- Proveedor del Exterior -->
        <record id="fiscal_position_account_1" model="account.fiscal.position.account.template">
            <field name="position_id" ref="position_exterior"/>
            <field name="account_dest_id" ref="do_niif_21010300"/>
            <field name="account_src_id" ref="do_niif_21010200"/>
        </record>
    </data>
</odoo>

```

## File: data\l10n_do_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_do_statements_menu" name="Dominican Republic" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_user"/>

    <!-- Chart of Accounts Template -->
    <record id="do_chart_template" model="account.chart.template">
        <field name="name">Catálogo de Cuentas Dominicano (NIIF)</field>
        <field name="code_digits">8</field>
        <field name="cash_account_code_prefix">110101</field>
        <field name="bank_account_code_prefix">110102</field>
        <field name="transfer_account_code_prefix">11010100</field>
        <field name="currency_id" ref="base.DOP"/>
    </record>
</odoo>

```

## File: data\l10n_do_res_partner_title.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!--
    Resource: res.partner.title
    Update partner titles
    -->
        <record id="res_partner_title_dra" model="res.partner.title">
            <field name="name">Doctora</field>
            <field name="shortcut">Dra.</field>
        </record>
        <record id="res_partner_title_msc" model="res.partner.title">
            <field name="name">Msc.</field>
            <field name="shortcut">Msc.</field>
        </record>
        <record id="res_partner_title_mba" model="res.partner.title">
            <field name="name">MBA</field>
            <field name="shortcut">MBA</field>
        </record>
        <record id="res_partner_title_lic" model="res.partner.title">
            <field name="name">Licenciado</field>
            <field name="shortcut">Lic.</field>
        </record>
        <record id="res_partner_title_licda" model="res.partner.title">
            <field name="name">Licenciada</field>
            <field name="shortcut">Licda.</field>
        </record>
        <record id="res_partner_title_ing" model="res.partner.title">
            <field name="name">Ingeniero/a</field>
            <field name="shortcut">Ing.</field>
        </record>
    </data>
</odoo>

```

## File: models\chart_template.py

```python
# coding: utf-8
# Copyright 2016 iterativo (https://www.iterativo.do) <info@iterativo.do>

from odoo import models, api, _


class AccountChartTemplate(models.Model):
    _inherit = "account.chart.template"

    @api.model
    def _get_default_bank_journals_data(self):
        if self.env.company.country_id and self.env.company.country_id.code.upper() == 'DO':
            return [
                {'acc_name': _('Cash'), 'account_type': 'cash'},
                {'acc_name': _('Caja Chica'), 'account_type': 'cash'},
                {'acc_name': _('Cheques Clientes'), 'account_type': 'cash'},
                {'acc_name': _('Bank'), 'account_type': 'bank'}
            ]
        return super(AccountChartTemplate, self)._get_default_bank_journals_data()

    def _prepare_all_journals(self, acc_template_ref, company, journals_dict=None):
        """Create fiscal journals for buys"""
        res = super(AccountChartTemplate, self)._prepare_all_journals(
            acc_template_ref, company, journals_dict=journals_dict)
        if not self == self.env.ref('l10n_do.do_chart_template'):
            return res
        for journal in res:
            if journal['code'] == 'FACT':
                journal['name'] = _('Compras Fiscales')
        res += [{
            'type': 'purchase',
            'name': _('Gastos No Deducibles'),
            'code': 'GASTO',
            'company_id': company.id,
            'show_on_dashboard': True
        }, {
            'type': 'purchase',
            'name': _('Migración CxP'),
            'code': 'CXP',
            'company_id': company.id,
            'show_on_dashboard': True
        }, {
            'type': 'sale',
            'name': _('Migración CxC'),
            'code': 'CXC',
            'company_id': company.id,
            'show_on_dashboard': True
        }]
        return res


```

## File: models\__init__.py

```python
# coding: utf-8
# Copyright 2016 iterativo (https://www.iterativo.do) <info@iterativo.do>

from . import chart_template

```

