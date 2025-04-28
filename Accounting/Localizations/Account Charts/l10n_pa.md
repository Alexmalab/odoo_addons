# Odoo Module: l10n_pa

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2011 Cubic ERP - Teradata SAC. (http://cubicerp.com).

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2011 Cubic ERP - Teradata SAC. (http://cubicerp.com).

{
    "name": "Panama - Accounting",
    "description": """
Panamenian accounting chart and tax localization.

Plan contable panameño e impuestos de acuerdo a disposiciones vigentes

Con la Colaboración de
- AHMNET CORP http://www.ahmnet.com

    """,
    "author": "Cubic ERP",
    'category': 'Accounting/Localizations/Account Charts',
    "depends": ["account"],
    "data": [
        "data/l10n_pa_chart_data.xml",
        "data/account_tax_group_data.xml",
        "data/account_tax_data.xml",
        "data/account_chart_template_data.xml",
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_pa.l10npa_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ITAX_19" model="account.tax.template">
        <field name="chart_template_id" ref="l10npa_chart_template"/>
        <field name="name">ITBMS 7% Venta</field>
        <field name="description">ITBMS 7% Venta</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_7"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('231'),
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
                'account_id': ref('231'),
            }),
        ]"/>
    </record>

    <record id="OTAX_19" model="account.tax.template">
        <field name="chart_template_id" ref="l10npa_chart_template"/>
        <field name="name">ITBMS 7% Compra</field>
        <field name="description">ITBMS 7% Compra</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_7"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('231'),
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
                'account_id': ref('231'),
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_7" model="account.tax.group">
            <field name="name">ITBMS 7%</field>
            <field name="country_id" ref="base.pa"/>
        </record>
    </data>
</odoo>

```

## File: data\l10n_pa_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Account Chart Templates -->
    <record id="l10npa_chart_template" model="account.chart.template">
        <field name="name">Panamá - Plan de Cuentas</field>
        <field name="bank_account_code_prefix">111.</field>
        <field name="cash_account_code_prefix">113.</field>
        <field name="transfer_account_code_prefix">112.</field>
        <field name="code_digits">7</field>
        <field name="currency_id" ref="base.PAB"/>
        <field name="country_id" ref="base.pa"/>
    </record>
    <!-- Account Templates -->

            <record id="114_001" model="account.account.template"><field name="name">Caja y Bancos.../ BCO. CTA CTE PAB</field><field name="code">114.001</field><field name="reconcile" eval="True"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="115" model="account.account.template"><field name="name">Caja y Bancos - Valores a Depositar </field><field name="code">115</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="116" model="account.account.template"><field name="name">Caja y Bancos - Recaudaciones a Depositar </field><field name="code">116</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="121" model="account.account.template"><field name="name">Cuentas por Cobrar / Deudores por Ventas</field><field name="code">121</field><field name="reconcile" eval="True"/><field ref="account.data_account_type_receivable" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/><field name="reconcile">1</field></record>
            <record id="121_01" model="account.account.template"><field name="name">Cuentas por Cobrar / Deudores por Ventas (PoS)</field><field name="code">121.01</field><field name="reconcile" eval="True"/><field ref="account.data_account_type_receivable" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/><field name="reconcile">1</field></record>
            <record id="122" model="account.account.template"><field name="name">Cuentas por Cobrar / Deudores Morosos</field><field name="code">122</field><field name="reconcile" eval="True"/><field ref="account.data_account_type_receivable" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/><field name="reconcile">1</field></record>
            <record id="123" model="account.account.template"><field name="name">Cuentas por Cobrar / Deudores en Gestión Judicial</field><field name="code">123</field><field name="reconcile" eval="True"/><field ref="account.data_account_type_receivable" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="124" model="account.account.template"><field name="name">Cuentas por Cobrar / Deudores Varios</field><field name="code">124</field><field name="reconcile" eval="True"/><field ref="account.data_account_type_receivable" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="125" model="account.account.template"><field name="name">Cuentas por Cobrar / (-) Previsión para Incobrables</field><field name="code">125</field><field name="reconcile" eval="True"/><field ref="account.data_account_type_receivable" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/><field name="reconcile">1</field></record>
            <record id="131" model="account.account.template"><field name="name">Otras Cuentas por Cobrar / Préstamos otorgados</field><field name="code">131</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="132" model="account.account.template"><field name="name">Otras Cuentas por Cobrar / Anticipos a Proveedores</field><field name="code">132</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="133" model="account.account.template"><field name="name">Otras Cuentas por Cobrar / Anticipo de Impuestos</field><field name="code">133</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="134" model="account.account.template"><field name="name">Otras Cuentas por Cobrar / Anticipo al Personal</field><field name="code">134</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="135" model="account.account.template"><field name="name">Otras Cuentas por Cobrar / Alquileres Pagados por Adelantado</field><field name="code">135</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="136" model="account.account.template"><field name="name">Otras Cuentas por Cobrar / Intereses Pagados por Adelantado</field><field name="code">136</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="137" model="account.account.template"><field name="name">Otras Cuentas por Cobrar / Accionistas</field><field name="code">137</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="138" model="account.account.template"><field name="name">Otras Cuentas por Cobrar / (-) Previsión para Descuentos</field><field name="code">138</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="139" model="account.account.template"><field name="name">Otras Cuentas por Cobrar / (-) Intereses (+) a Devengar</field><field name="code">139</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="141" model="account.account.template"><field name="name">Inversiones / Acciones Transitorias</field><field name="code">141</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="142" model="account.account.template"><field name="name">Inversiones / Acciones Permanentes</field><field name="code">142</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="143" model="account.account.template"><field name="name">Inversiones / Títulos Públicos</field><field name="code">143</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="144" model="account.account.template"><field name="name">Inversiones / (-) Previsión para Devalorización de Acciones</field><field name="code">144</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="151_01" model="account.account.template"><field name="name">Inventarios - Mercancias / Categoria de productos 01</field><field name="code">151.01</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="152" model="account.account.template"><field name="name">Inventarios - Mercancias en Tránsito</field><field name="code">152</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="153" model="account.account.template"><field name="name">Materias primas</field><field name="code">153</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="154" model="account.account.template"><field name="name">Productos en Curso de Elaboración</field><field name="code">154</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="155" model="account.account.template"><field name="name">Productos Elaborados</field><field name="code">155</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="156" model="account.account.template"><field name="name">Materiales Varios </field><field name="code">156</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="157" model="account.account.template"><field name="name">(-) Previsión para Desvalorización de Inventarios</field><field name="code">157</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="161" model="account.account.template"><field name="name">Activo Fijo / Inmuebles</field><field name="code">161</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="162" model="account.account.template"><field name="name">Activo Fijo / Maquinaria</field><field name="code">162</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="163" model="account.account.template"><field name="name">Activo Fijo / Equipos</field><field name="code">163</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="164" model="account.account.template"><field name="name">Activo Fijo / Material Rodante Motorizado</field><field name="code">164</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="165" model="account.account.template"><field name="name">Activo Fijo / (-) Depreciación Acumulada</field><field name="code">165</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="171" model="account.account.template"><field name="name">Activo Intangible / Derecho de Llaves</field><field name="code">171</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="172" model="account.account.template"><field name="name">Activo Intangible / Concesiones y Franquicias</field><field name="code">172</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="173" model="account.account.template"><field name="name">Activo Intangible / Marcas y Patentes de Invención</field><field name="code">173</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="174" model="account.account.template"><field name="name">Activo Intangible / (-) Amortización Acumulada</field><field name="code">174</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_assets" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="211" model="account.account.template"><field name="name">Cuentas por Pagar / Proveedores</field><field name="code">211</field><field name="reconcile" eval="True"/><field ref="account.data_account_type_payable" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/><field name="reconcile">1</field></record>
            <record id="212" model="account.account.template"><field name="name">Cuentas por Pagar / Anticipos de Clientes</field><field name="code">212</field><field name="reconcile" eval="True"/><field ref="account.data_account_type_payable" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/><field name="reconcile">1</field></record>
            <record id="213" model="account.account.template"><field name="name">Cuentas por Pagar / (-) Intereses a Devengar por Compras al Crédito</field><field name="code">213</field><field name="reconcile" eval="True"/><field ref="account.data_account_type_payable" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/><field name="reconcile">1</field></record>
            <record id="221" model="account.account.template"><field name="name">Pasivo Circulante / Adelantos en Cuenta Corriente</field><field name="code">221</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_liabilities" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="222" model="account.account.template"><field name="name">Pasivo Circulante / Prestamos</field><field name="code">222</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_liabilities" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="223" model="account.account.template"><field name="name">Pasivo Circulante / Obligaciones a Pagar</field><field name="code">223</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_liabilities" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="224" model="account.account.template"><field name="name">Pasivo Circulante / Intereses a Pagar</field><field name="code">224</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_liabilities" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="225" model="account.account.template"><field name="name">Pasivo Circulante / Debentures Emitidos</field><field name="code">225</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_liabilities" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="231" model="account.account.template"><field name="name">Impuestos por Pagar / ITBMS a Pagar</field><field name="code">231</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_liabilities" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="232" model="account.account.template"><field name="name">Impuestos por Pagar / Impuesto sobre la Renta a Pagar</field><field name="code">232</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_liabilities" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="241" model="account.account.template"><field name="name">Salarios por Pagar / Sueldos a Pagar</field><field name="code">241</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_liabilities" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="242" model="account.account.template"><field name="name">Salarios por Pagar / Cargas Sociales a Pagar</field><field name="code">242</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_liabilities" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="243" model="account.account.template"><field name="name">Salarios por Pagar / Provisión para Sueldo Anual Complementario</field><field name="code">243</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_liabilities" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="244" model="account.account.template"><field name="name">Salarios por Pagar / Retenciones a Depositar</field><field name="code">244</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_liabilities" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="251" model="account.account.template"><field name="name">Otras Cuentas por Pagar / Acreedores Varios</field><field name="code">251</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_liabilities" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="252" model="account.account.template"><field name="name">Otras Cuentas por Pagar / Dividendos a Pagar</field><field name="code">252</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_liabilities" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="253" model="account.account.template"><field name="name">Otras Cuentas por Pagar / Cobros por Adelantado</field><field name="code">253</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_liabilities" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="254" model="account.account.template"><field name="name">Otras Cuentas por Pagar / Honorarios Directores y Síndicos a Pagar</field><field name="code">254</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_liabilities" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="261" model="account.account.template"><field name="name">Provisiones / Previsión Indemnización por Despidos</field><field name="code">261</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_liabilities" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="262" model="account.account.template"><field name="name">Provisiones / Previsión para juicios Pendientes</field><field name="code">262</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_liabilities" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="263" model="account.account.template"><field name="name">Provisiones / Previsión para Garantías por Service</field><field name="code">263</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_current_liabilities" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="311" model="account.account.template"><field name="name">Capital / Capital Propio</field><field name="code">311</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_equity" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="312" model="account.account.template"><field name="name">Capital / Acciones en Circulación</field><field name="code">312</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_equity" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="313" model="account.account.template"><field name="name">Capital / Dividendos a Distribuir en Acciones</field><field name="code">313</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_equity" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="314" model="account.account.template"><field name="name">Capital / (-) Descuento de Emisión de Acciones</field><field name="code">314</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_equity" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="321" model="account.account.template"><field name="name">Aportes No Capitalizados / Primas de Emsión</field><field name="code">321</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_equity" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="322" model="account.account.template"><field name="name">Aportes No Capitalizados / Aportes Irrevocables Futura Suscripción de Acciones</field><field name="code">322</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_equity" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="331" model="account.account.template"><field name="name">Ajustes al Patrimonio / Revaluo Técnico de Activo Fijo</field><field name="code">331</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_equity" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="341" model="account.account.template"><field name="name">Reserva Legal</field><field name="code">341</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_equity" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="342" model="account.account.template"><field name="name">Reserva Estatutaria</field><field name="code">342</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_equity" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="343" model="account.account.template"><field name="name">Reserva Facultativa</field><field name="code">343</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_equity" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="344" model="account.account.template"><field name="name">Reserva para Renovación de Activo Fijo</field><field name="code">344</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_equity" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="351" model="account.account.template"><field name="name">Resultados Acumulados</field><field name="code">351</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_equity" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="352" model="account.account.template"><field name="name">Resultados Acumulados del Ejercicio Anterior</field><field name="code">352</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_equity" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="353" model="account.account.template"><field name="name">Utilidades y Pérdidas del Ejercicio</field><field name="code">353</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_equity" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="354" model="account.account.template"><field name="name">Resultado del Ejercicio</field><field name="code">354</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_equity" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>

            <record id="411_01" model="account.account.template"><field name="name">Ventas - Categoria de productos 01</field><field name="code">411.01</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_revenue" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="412" model="account.account.template"><field name="name">Intereses gananados, obtenidos, percibidos</field><field name="code">412</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_other_income" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="413" model="account.account.template"><field name="name">Alquileres gananados, obtenidos, percibidos</field><field name="code">413</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_other_income" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="414" model="account.account.template"><field name="name">Comisiones gananados, obtenidos, percibidos</field><field name="code">414</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_other_income" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="415" model="account.account.template"><field name="name">Descuentos gananados, obtenidos, percibidos</field><field name="code">415</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_other_income" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="416" model="account.account.template"><field name="name">Interese sobre Inversiones</field><field name="code">416</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_other_income" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="417" model="account.account.template"><field name="name">Honorarios gananados, obtenidos, percibidos</field><field name="code">417</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_other_income" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="418" model="account.account.template"><field name="name">Ganancia Venta de Acciones</field><field name="code">418</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_other_income" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="421" model="account.account.template"><field name="name">Recupero de Rezagos</field><field name="code">421</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_other_income" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="422" model="account.account.template"><field name="name">Recupero de Deudores Incobrables</field><field name="code">422</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_other_income" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="423" model="account.account.template"><field name="name">Ganancia Venta de Activo Fijo</field><field name="code">423</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_other_income" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="424" model="account.account.template"><field name="name">Donaciones obtenidas, ganandas, percibidas</field><field name="code">424</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_other_income" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="425" model="account.account.template"><field name="name">Ganancia Venta Inversiones Permanentes</field><field name="code">425</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_other_income" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="512" model="account.account.template"><field name="name">Gastos en Depreciación de Activo Fijo</field><field name="code">512</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_expenses" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="513" model="account.account.template"><field name="name">Gastos en Amortización</field><field name="code">513</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_expenses" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="514" model="account.account.template"><field name="name">Gastos en Salarios</field><field name="code">514</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_expenses" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="515" model="account.account.template"><field name="name">Gastos en Cargas Sociales</field><field name="code">515</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_expenses" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="516" model="account.account.template"><field name="name">Gastos en Impuestos</field><field name="code">516</field><field name="reconcile" eval="False"/><field ref="account.data_account_off_sheet" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="517" model="account.account.template"><field name="name">Gastos Bancarios</field><field name="code">517</field><field name="reconcile" eval="False"/><field ref="account.data_account_off_sheet" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="518" model="account.account.template"><field name="name">Gastos en Servicios Públicos</field><field name="code">518</field><field name="reconcile" eval="False"/><field ref="account.data_account_off_sheet" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="519" model="account.account.template"><field name="name">Gastos de Publicidad y Propaganda</field><field name="code">519</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_expenses" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="521" model="account.account.template"><field name="name">Gastos en Siniestros</field><field name="code">521</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_expenses" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="522" model="account.account.template"><field name="name">Donaciones Cedidas, Otorgadas</field><field name="code">522</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_expenses" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="523" model="account.account.template"><field name="name">Pérdida Venta Activo Fijo</field><field name="code">523</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_expenses" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>


            <record id="61_01" model="account.account.template"><field name="name">Costo de Venta - Categoria de productos 01</field><field name="code">61.01</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_expenses" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>

            <record id="62_01" model="account.account.template"><field name="name">Compras - Categoria de productos 01</field><field name="code">62.01</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_expenses" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="63" model="account.account.template"><field name="name">Costos de Producción</field><field name="code">63</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_expenses" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="64" model="account.account.template"><field name="name">Gastos de Administración</field><field name="code">64</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_expenses" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>
            <record id="65" model="account.account.template"><field name="name">Gastos de Comercialización</field><field name="code">65</field><field name="reconcile" eval="False"/><field ref="account.data_account_type_expenses" name="user_type_id"/><field name="chart_template_id" ref="l10npa_chart_template"/></record>

            <record id="711" model="account.account.template"><field name="name">Mercaderias Recibidas en Consignación</field><field name="code">711</field><field name="reconcile" eval="False"/><field ref="account.data_account_off_sheet" name="user_type_id"/></record>
            <record id="712" model="account.account.template"><field name="name">Depósito de Valores Recibos en Garantía</field><field name="code">712</field><field name="reconcile" eval="False"/><field ref="account.data_account_off_sheet" name="user_type_id"/></record>
            <record id="713" model="account.account.template"><field name="name">Garantias Otorgadas</field><field name="code">713</field><field name="reconcile" eval="False"/><field ref="account.data_account_off_sheet" name="user_type_id"/></record>
            <record id="714" model="account.account.template"><field name="name">Documentos Descontados</field><field name="code">714</field><field name="reconcile" eval="False"/><field ref="account.data_account_off_sheet" name="user_type_id"/></record>
            <record id="715" model="account.account.template"><field name="name">Documentos Endosados</field><field name="code">715</field><field name="reconcile" eval="False"/><field ref="account.data_account_off_sheet" name="user_type_id"/></record>

            <record id="721" model="account.account.template"><field name="name">Comitente por Mercaderias Recibidas en Consignación</field><field name="code">721</field><field name="reconcile" eval="False"/><field ref="account.data_account_off_sheet" name="user_type_id"/></record>
            <record id="722" model="account.account.template"><field name="name">Acreedor por Garantías Otorgadas</field><field name="code">722</field><field name="reconcile" eval="False"/><field ref="account.data_account_off_sheet" name="user_type_id"/></record>
            <record id="723" model="account.account.template"><field name="name">Acreedor por Documentos Descontados</field><field name="code">723</field><field name="reconcile" eval="False"/><field ref="account.data_account_off_sheet" name="user_type_id"/></record>

        <record id="gain81_01" model="account.account.template">
            <field name="code">81</field>
            <field name="name">Cuenta de cambio (Ganancia)</field>
            <field name="user_type_id" ref="account.data_account_type_other_income"/>
            <field name="chart_template_id" ref="l10npa_chart_template"/>
        </record>

        <record id="loss81_01" model="account.account.template">
            <field name="code">82</field>
            <field name="name">Cuenta de cambio (Pérdida)</field>
            <field name="user_type_id" ref="account.data_account_type_expenses"/>
            <field name="chart_template_id" ref="l10npa_chart_template"/>
        </record>


    <record id="l10npa_chart_template" model="account.chart.template">
      <field name="property_account_receivable_id" ref="121"/>
      <field name="property_account_payable_id" ref="211"/>
      <field name="property_account_expense_categ_id" ref="62_01"/>
      <field name="property_account_income_categ_id" ref="411_01"/>
      <field name="income_currency_exchange_account_id" ref="gain81_01"/>
      <field name="expense_currency_exchange_account_id" ref="loss81_01"/>
      <field name="default_pos_receivable_account_id" ref="121_01" />
    </record>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.9" y="6.07" width="49.2" height="34.06" maskUnits="userSpaceOnUse">
      <rect x="6.35" y="7.65" width="47.3" height="31.57" rx="1" style="fill: #fff"/>
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
      <g>
        <image width="1920" height="1152" transform="translate(4.9 7.37) scale(0.03 0.03)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAB4EAAAT/CAYAAAAL/IRxAAAACXBIWXMAAbAlAAGwJQHEyoSNAAAgAElEQVR4XuzbMQ3AMBAEQTsKf7Ah8IHg2quZ+hisbq+1ZgEAAAAAAABwvZnZz2kEAAAAAAAAwD1EYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAEJEYAAAAAAAAIAQERgAAAAAAAAgRAQGAAAAAAAACBGBAQAAAAAAAELe77QAAAAAAAAA4BqewAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAwM++HRMBAMMwEFu6hj+sUiqEzvmTZjP4MwAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAyMWfXoAABERSURBVAAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhIjAAAAAAAABAiAgMAAAAAAAAECICAwAAAAAAAISIwAAAAAAAAAAhZ+5vAgAAAAAAAMAWnsAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAAAAABAiAgMAAAAAAACEiMAAAAAAAAAAISIwAAAAAAAAQIgIDAAAAPDas2MiAEAYgIEd8O8JZ5XATO5/joMAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQIgJDAAAAAAAABBiAgMAAAAAAACEmMAAAAAAAAAAISYwAAAAAAAAQMiZmfuKAAAAAAAAAPjDAkxSEJDzDMi+AAAAAElFTkSuQmCC"/>
        <image width="200" height="133" transform="translate(4.9 6.07) scale(0.25 0.25)" xlink:href="data:image/png;base64,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"/>
      </g>
    </g>
  </g>
</svg>

```

