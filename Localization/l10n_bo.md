# Odoo Module: l10n_bo

Category: Localization

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2011 Cubic ERP - Teradata SAC. (https://cubicerp.com).

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2011 Cubic ERP - Teradata SAC. (https://cubicerp.com).

{
    "name": "Bolivia - Accounting",
    "version": "2.0",
    "description": """
Bolivian accounting chart and tax localization.

Plan contable boliviano e impuestos de acuerdo a disposiciones vigentes

    """,
    "author": "Cubic ERP",
    'category': 'Localization',
    "depends": ["account"],
    "data": [
        "data/l10n_bo_chart_data.xml",
        "data/account.account.template.csv",
        "data/l10n_bo_chart_post_data.xml",
        'data/account_data.xml',
        'data/account_tax_report_data.xml',
        "data/account_tax_data.xml",
        "data/account_chart_template_data.xml",
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","currency_id/id","reconcile"
"112_001","Caja y bancos - Caja / efectivo USD","112.001","account.data_account_type_current_assets","l10n_bo.bo_chart_template","base.USD","False"
"113_001","Caja y ...- Fondos fijos / caja chica 01 BOB","113.001","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"115","Caja y bancos - Valores a Depositar ","115","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"116","Caja y bancos - Recaudaciones a Depositar ","116","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"121","Créditos fiscal IVA / Deudores por Ventas","121","account.data_account_type_receivable","l10n_bo.bo_chart_template","","True"
"121_001","Créditos fiscal IVA / Deudores por Ventas (PoS)","121.001","account.data_account_type_receivable","l10n_bo.bo_chart_template","","True"
"122","Créditos fiscal IVA / Deudores Morosos","122","account.data_account_type_receivable","","","True"
"123","Créditos fiscal IVA / Deudores en Gestión Judicial","123","account.data_account_type_receivable","l10n_bo.bo_chart_template","","True"
"124","Créditos fiscal IVA / Deudores Varios","124","account.data_account_type_receivable","l10n_bo.bo_chart_template","","True"
"125","Créditos fiscal IVA / (-) Previsión para Ds. Incobrables","125","account.data_account_type_receivable","l10n_bo.bo_chart_template","","True"
"131","Otros Créditos / Préstamos otorgados","131","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"132","Otros Créditos / Anticipos a Proveedores","132","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"133","Otros Créditos / Anticipo de Impuestos","133","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"134","Otros Créditos / Anticipo al Personal","134","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"135","Otros Créditos / Alquileres Pagados por Adelantado","135","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"136","Otros Créditos / Intereses Pagados por Adelantado","136","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"137","Otros Créditos / Accionistas","137","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"138","Otros Créditos / (-) Previsión para Descuentos","138","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"139","Otros Créditos / (-) Intereses (+) a Devengar","139","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"141","Inversiones / Acciones Transitorias","141","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"142","Inversiones / Acciones Permanentes","142","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"143","Inversiones / Títulos Públicos","143","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"144","Inversiones / (-) Previsión para Devalorización de Acciones","144","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"151_01","Bienes de Cambio - Existencia de Mercaderías / Categoria de productos 01","151.01","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"152","Bienes de Cambio - Mercaderías en Tránsito","152","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"153","Materias primas","153","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"154","Productos en Curso de Elaboración","154","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"155","Productos Elaborados","155","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"156","Materiales Varios ","156","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"157","(-) Desvalorización de Existencias","157","account.data_account_type_current_assets","l10n_bo.bo_chart_template","","False"
"161","Bienes de Uso / Inmuebles","161","account.data_account_type_fixed_assets","l10n_bo.bo_chart_template","","False"
"162","Bienes de Uso / Maquinaria","162","account.data_account_type_fixed_assets","l10n_bo.bo_chart_template","","False"
"163","Bienes de Uso / Equipos","163","account.data_account_type_fixed_assets","l10n_bo.bo_chart_template","","False"
"164","Bienes de Uso / Vehículos","164","account.data_account_type_fixed_assets","l10n_bo.bo_chart_template","","False"
"165","Bienes de Uso / (-) Depreciación Acumulada Bienes de Uso","165","account.data_account_type_fixed_assets","l10n_bo.bo_chart_template","","False"
"171","Bienes Intangibles / Marcas de Fábrica","171","account.data_account_type_fixed_assets","l10n_bo.bo_chart_template","","False"
"172","Bienes Intangibles / Concesiones y Franquicias","172","account.data_account_type_fixed_assets","l10n_bo.bo_chart_template","","False"
"173","Bienes Intangibles / Patentes de Invención","173","account.data_account_type_fixed_assets","l10n_bo.bo_chart_template","","False"
"174","Bienes Intangibles / (-) Amortización Acumulada","174","account.data_account_type_fixed_assets","l10n_bo.bo_chart_template","","False"
"211","Deudas Comerciales / Proveedores","211","account.data_account_type_payable","l10n_bo.bo_chart_template","","True"
"212","Deudas Comerciales / Anticipos de Clientes","212","account.data_account_type_payable","l10n_bo.bo_chart_template","","True"
"213","Deudas Comerciales / (-) Intereses a Devengar por Compras al Crédito","213","account.data_account_type_payable","l10n_bo.bo_chart_template","","True"
"221","Deudas Bancarias y Financieras / Adelantos en Cuenta Corriente","221","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"222","Deudas Bancarias y Financieras / Prestamos","222","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"223","Deudas Bancarias y Financieras / Obligaciones a Pagar","223","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"224","Deudas Bancarias y Financieras / Intereses a Pagar","224","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"225","Deudas Bancarias y Financieras / Letras de Cambio Emitidos","225","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"231","Deudas Fiscales / IVA a Pagar","231","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"232","Deudas Fiscales / Impuesto a las Transacciones IT a Pagar","232","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"233","Deudas Fiscales / Impuesto a las Utilidades de Empresas IUE a Pagar","233","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"241","Deudas Sociales / Sueldos a Pagar","241","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"242","Deudas Sociales / Cargas Sociales a Pagar","242","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"243","Deudas Sociales / Provisión para Sueldo Anual Complementario","243","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"244","Deudas Sociales / Retenciones a Depositar","244","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"251","Otras Deudas / Acreedores Varios","251","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"252","Otras Deudas / Dividendos a Pagar","252","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"253","Otras Deudas / Cobros por Adelantado","253","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"254","Otras Deudas / Honorarios Directores y Síndicos a Pagar","254","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"261","Previsiones / Previsión Indemnización por Despidos","261","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"262","Previsiones / Previsión para juicios Pendientes","262","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"263","Previsiones / Previsión para Garantías por Service","263","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"311","Capital social / Capital Suscripto","311","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"312","Capital social / Acciones en Circulación","312","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"313","Capital social / Dividendos a Distribuir en Acciones","313","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"314","Capital social / (-) Descuento de Emisión de Acciones","314","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"321","Aportes No Capitalizados / Primas de Emsión","321","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"322","Aportes No Capitalizados / Aportes Irrevocables Futura Suscripción de Acciones","322","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"331","Ajustes al Patrimonio / Revaluo Técnico de Bienes de Uso","331","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"341","Reserva Legal","341","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"342","Reserva Estatutaria","342","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"343","Reserva Facultativa","343","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"344","Reserva para Renovación de Bienes de Uso","344","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"351","Resultados Acumulados","351","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"352","Resultados Acumulados del Ejercicio Anterior","352","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"353","Ganancias y Pérdidas del Ejercicio","353","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"354","Resultado del Ejercicio","354","account.data_account_type_current_liabilities","l10n_bo.bo_chart_template","","False"
"411_01","Ventas - Categoria de productos 01","411.01","account.data_account_type_revenue","l10n_bo.bo_chart_template","","False"
"412","Intereses gananados, obtenidos, percibidos","412","account.data_account_type_revenue","l10n_bo.bo_chart_template","","False"
"413","Alquileres gananados, obtenidos, percibidos","413","account.data_account_type_other_income","l10n_bo.bo_chart_template","","False"
"414","Comisiones gananados, obtenidos, percibidos","414","account.data_account_type_other_income","l10n_bo.bo_chart_template","","False"
"415","Descuentos gananados, obtenidos, percibidos","415","account.data_account_type_other_income","l10n_bo.bo_chart_template","","False"
"416","Renta de Títulos Públicos","416","account.data_account_type_other_income","l10n_bo.bo_chart_template","","False"
"417","Honorarios gananados, obtenidos, percibidos","417","account.data_account_type_other_income","l10n_bo.bo_chart_template","","False"
"418","Ganancia Venta de Acciones","418","account.data_account_type_other_income","l10n_bo.bo_chart_template","","False"
"421","Recupero de Rezagos","421","account.data_account_type_other_income","l10n_bo.bo_chart_template","","False"
"422","Recupero de Deudores Incobrables","422","account.data_account_type_other_income","l10n_bo.bo_chart_template","","False"
"423","Ganancia Venta de Bienes de Uso","423","account.data_account_type_other_income","l10n_bo.bo_chart_template","","False"
"424","Donaciones obtenidas, ganandas, percibidas","424","account.data_account_type_other_income","l10n_bo.bo_chart_template","","False"
"425","Ganancia Venta Inversiones Permanentes","425","account.data_account_type_other_income","l10n_bo.bo_chart_template","","False"
"ch_bo_426","La ganancia cambiaria","426","account.data_account_type_other_income","l10n_bo.bo_chart_template","","False"
"511_01","Costo de Mercaderías Vendidas - Categoria de productos 01","511.01","account.data_account_type_expenses","l10n_bo.bo_chart_template","","False"
"512","Gastos en Depreciación de Bienes de Uso","512","account.data_account_type_expenses","l10n_bo.bo_chart_template","","False"
"513","Gastos en Amortización","513","account.data_account_type_expenses","l10n_bo.bo_chart_template","","False"
"514","Gastos en Sueldos y Jormales","514","account.data_account_type_expenses","l10n_bo.bo_chart_template","","False"
"515","Gastos en Cargas Sociales","515","account.data_account_type_expenses","l10n_bo.bo_chart_template","","False"
"516","Gastos en Impuestos","516","account.data_account_off_sheet","l10n_bo.bo_chart_template","","False"
"517","Gastos Bancarios","517","account.data_account_off_sheet","l10n_bo.bo_chart_template","","False"
"518","Gastos en Servicios Públicos","518","account.data_account_off_sheet","l10n_bo.bo_chart_template","","False"
"519","Gastos de Publicidad y Propaganda","519","account.data_account_type_expenses","l10n_bo.bo_chart_template","","False"
"521","Gastos en Siniestros","521","account.data_account_type_expenses","l10n_bo.bo_chart_template","","False"
"522","Donaciones Cedidas, Otorgadas","522","account.data_account_type_expenses","l10n_bo.bo_chart_template","","False"
"523","Pérdida Venta Bienes de Uso","523","account.data_account_type_expenses","l10n_bo.bo_chart_template","","False"
"ch_bo_524","La pérdida cambiaria","524","account.data_account_type_expenses","l10n_bo.bo_chart_template","","False"
"61_01","Compras - Categoria de productos 01","61.01","account.data_account_type_expenses","l10n_bo.bo_chart_template","","False"
"62","Costos de Producción","62","account.data_account_type_expenses","l10n_bo.bo_chart_template","","False"
"63","Gastos de Administración","63","account.data_account_type_expenses","l10n_bo.bo_chart_template","","False"
"64","Gastos de Comercialización","64","account.data_account_type_expenses","l10n_bo.bo_chart_template","","False"
"711","Mercaderias Recibidas en Consignación","711","account.data_account_off_sheet","l10n_bo.bo_chart_template","","False"
"712","Depósito de Valores Recibos en Garantía","712","account.data_account_off_sheet","l10n_bo.bo_chart_template","","False"
"713","Garantias Otorgadas","713","account.data_account_off_sheet","l10n_bo.bo_chart_template","","False"
"714","Documentos Descontados","714","account.data_account_off_sheet","l10n_bo.bo_chart_template","","False"
"715","Documentos Endosados","715","account.data_account_off_sheet","l10n_bo.bo_chart_template","","False"
"721","Comitente por Mercaderias Recibidas en Consignación","721","account.data_account_off_sheet","l10n_bo.bo_chart_template","","False"
"722","Acreedor por Garantías Otorgadas","722","account.data_account_off_sheet","l10n_bo.bo_chart_template","","False"
"723","Acreedor por Documentos Descontados","723","account.data_account_off_sheet","l10n_bo.bo_chart_template","","False"

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_bo.bo_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Account Tax Group -->
        <record id="tax_group_iva_13" model="account.tax.group">
            <field name="name">IVA 13%</field>
        </record>

        <record id="tax_group_it_3" model="account.tax.group">
            <field name="name">IT 3%</field>
        </record>

    </data>
</odoo>
```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ITAX_21" model="account.tax.template">
        <field name="chart_template_id" ref="bo_chart_template"/>
        <field name="name">IVA 13% Venta</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="include_base_amount">1</field>
        <field name="tax_group_id" ref="tax_group_iva_13"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_base_imponible_venras_iva')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('231'),
                'plus_report_line_ids': [ref('tax_report_impuesto_ttl_cob_iva')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_base_imponible_venras_iva')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('231'),
                'minus_report_line_ids': [ref('tax_report_impuesto_ttl_cob_iva')],
            }),
        ]"/>
    </record>

    <record id="OTAX_21" model="account.tax.template">
        <field name="chart_template_id" ref="bo_chart_template"/>
        <field name="name">IVA 13% Compra</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="include_base_amount">1</field>
        <field name="tax_group_id" ref="tax_group_iva_13"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_base_imponible_compras_iva')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('231'),
                'plus_report_line_ids': [ref('tax_report_impuesto_ttl_pag_iva')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_base_imponible_compras_iva')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('231'),
                'minus_report_line_ids': [ref('tax_report_impuesto_ttl_pag_iva')],
            }),
        ]"/>
    </record>

    <record id="ITAX_03" model="account.tax.template">
        <field name="chart_template_id" ref="bo_chart_template"/>
        <field name="name">IT 3%</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="price_include" eval="True"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_it_3"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_base_imponible_venras_iva')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('232'),
                'plus_report_line_ids': [ref('tax_report_impuesto_trans_3')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_base_imponible_venras_iva')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('232'),
                'minus_report_line_ids': [ref('tax_report_impuesto_trans_3')],
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

	<record id="tax_report_impuesto_25" model="account.tax.report.line">
        <field name="name">Impuesto a las Utilidades de la Empresa IUE (25%)</field>
        <field name="sequence" eval="1"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_impuesto_trans" model="account.tax.report.line">
        <field name="name">Impuesto a las Transacciones IT (3%)</field>
        <field name="sequence" eval="2"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_impuesto_trans_3" model="account.tax.report.line">
        <field name="name">Impuesto a las Transacciones IT (3%)</field>
        <field name="tag_name">Impuesto a las Transacciones IT (3%)</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_impuesto_trans"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_impuesto_ttl" model="account.tax.report.line">
        <field name="name">Impuesto al Valor Agregado (IVA) Total a Pagar</field>
        <field name="sequence" eval="3"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_impuesto_ttl_cob" model="account.tax.report.line">
        <field name="name">Impuesto Cobrado</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_impuesto_ttl"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_impuesto_ttl_cob_fuera" model="account.tax.report.line">
        <field name="name">Impuesto Cobrado Fuera de Ámbito</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_impuesto_ttl_cob"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_impuesto_ttl_cob_exon" model="account.tax.report.line">
        <field name="name">Impuesto Cobrado de Exonerados al IVA</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_impuesto_ttl_cob"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_impuesto_ttl_cob_iva" model="account.tax.report.line">
        <field name="name">Impuesto Cobrado IVA</field>
        <field name="tag_name">Impuesto Cobrado IVA</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_impuesto_ttl_cob"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_impuesto_ttl_pag" model="account.tax.report.line">
        <field name="name">Impuesto Pagado</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_impuesto_ttl"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_impuesto_ttl_pag_fuera" model="account.tax.report.line">
        <field name="name">Impuesto Pagado Fuera de Ámbito</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_impuesto_ttl_pag"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_impuesto_ttl_pag_exon" model="account.tax.report.line">
        <field name="name">Impuesto Pagado de Exonerados al IVA</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_impuesto_ttl_pag"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_impuesto_ttl_pag_iva" model="account.tax.report.line">
        <field name="name">Impuesto Pagado IVA</field>
        <field name="tag_name">Impuesto Pagado IVA</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_impuesto_ttl_pag"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_base_imponible" model="account.tax.report.line">
        <field name="name">Base Imponible</field>
        <field name="sequence" eval="4"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_base_imponible_compras" model="account.tax.report.line">
        <field name="name">Base Imponible - Compras</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_base_imponible"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_base_imponible_compras_fuera" model="account.tax.report.line">
        <field name="name">Compras Gravadas Fuera de Ámbito</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_base_imponible_compras"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_base_imponible_compras_gravadas" model="account.tax.report.line">
        <field name="name">Compras NO Gravadas (Exoneradas)</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_base_imponible_compras"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_base_imponible_compras_iva" model="account.tax.report.line">
        <field name="name">Compras Gravadas con IVA</field>
        <field name="tag_name">Compras Gravadas con IVA</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_base_imponible_compras"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_base_imponible_venras" model="account.tax.report.line">
        <field name="name">Base Imponible - Ventas</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_base_imponible"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_base_imponible_venras_fuera" model="account.tax.report.line">
        <field name="name">Ventas Gravadas Fuera de Ámbito</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_base_imponible_venras"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_base_imponible_venras_exon" model="account.tax.report.line">
        <field name="name">Ventas NO Gravadas (Exoneradas)</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_base_imponible_venras"/>
        <field name="country_id" ref="base.bo"/>
    </record>

    <record id="tax_report_base_imponible_venras_iva" model="account.tax.report.line">
        <field name="name">Impuesto al Valor Agregado con IVA</field>
        <field name="tag_name">Impuesto al Valor Agregado con IVA</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_base_imponible_venras"/>
        <field name="country_id" ref="base.bo"/>
    </record>

</odoo>

```

## File: data\l10n_bo_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_bo_statements_menu" name="Bolivia" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_user"/>

    <record id="bo_chart_template" model="account.chart.template">
        <field name="name">Bolivia - Plan de Cuentas</field>
        <field name="code_digits">6</field>
        <field name="bank_account_code_prefix">114</field>
        <field name="cash_account_code_prefix">111</field>
        <field name="transfer_account_code_prefix">117</field>
        <field name="currency_id" ref="base.BOB"/>
    </record>
</odoo>

```

## File: data\l10n_bo_chart_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="bo_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="121"/>
        <field name="property_account_payable_id" ref="211"/>
        <field name="property_account_expense_categ_id" ref="61_01"/>
        <field name="property_account_income_categ_id" ref="411_01"/>
        <field name="income_currency_exchange_account_id" ref="ch_bo_426"/>
        <field name="expense_currency_exchange_account_id" ref="ch_bo_524"/>
        <field name="default_pos_receivable_account_id" ref="121_001" />
    </record>
</odoo>

```

