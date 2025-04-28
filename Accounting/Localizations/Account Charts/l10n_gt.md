# Odoo Module: l10n_gt

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
#-*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2009-2010 Soluciones Tecnologócias Prisma S.A. All Rights Reserved.
# José Rodrigo Fernández Menegazzo, Soluciones Tecnologócias Prisma S.A.
# (http://www.solucionesprisma.com)

```

## File: __manifest__.py

```python
#-*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2009-2010 Soluciones Tecnologócias Prisma S.A. All Rights Reserved.
# José Rodrigo Fernández Menegazzo, Soluciones Tecnologócias Prisma S.A.
# (http://www.solucionesprisma.com)

#
# This module provides a minimal Guatemalan chart of accounts that can be use
# to build upon a more complex one.  It also includes a chart of taxes and
# the Quetzal currency.
#
# This module is based on the UK minimal chart of accounts:
# Copyright (c) 2004-2009 Seath Solutions Ltd. All Rights Reserved.
# Geoff Gardiner, Seath Solutions Ltd (http://www.seathsolutions.com/)
#
# This module works with OpenERP 6.0
#

{
    'name': 'Guatemala - Accounting',
    'version': '3.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the base module to manage the accounting chart for Guatemala.
=====================================================================

Agrega una nomenclatura contable para Guatemala. También icluye impuestos y
la moneda del Quetzal. -- Adds accounting chart for Guatemala. It also includes
taxes and the Quetzal currency.""",
    'author': 'José Rodrigo Fernández Menegazzo',
    'website': 'http://solucionesprisma.com/',
    'depends': ['base', 'account'],
    'data': [
        'data/l10n_gt_chart_data.xml',
        'data/account.account.template.csv',
        'data/l10n_gt_chart_post_data.xml',
        'data/account_data.xml',
        'data/account_chart_template_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","reconcile"
"cta110201","Cuentas por Cobrar Generales","1.1.02.01","account.data_account_type_receivable","l10n_gt.cuentas_plantilla","True"
"cta110202","Cuentas por Cobrar Empresas Afilidas","1.1.02.02","account.data_account_type_receivable","l10n_gt.cuentas_plantilla","True"
"cta110203","Prestamos al Personal","1.1.02.03","account.data_account_type_receivable","l10n_gt.cuentas_plantilla","True"
"cta110204","Otras Cuentas por Cobrar","1.1.02.04","account.data_account_type_receivable","l10n_gt.cuentas_plantilla","True"
"cta110205","Cuentas por Cobrar Generales (PoS)","1.1.02.05","account.data_account_type_receivable","l10n_gt.cuentas_plantilla","True"
"cta110301","IVA por Cobrar","1.1.03.01","account.data_account_type_current_assets","l10n_gt.cuentas_plantilla","False"
"cta110302","Retenciones de IVA recibidas","1.1.03.02","account.data_account_type_current_assets","l10n_gt.cuentas_plantilla","False"
"cta120101","Propiedad, Planta y Equipo","1.2.01.01","account.data_account_type_current_assets","l10n_gt.cuentas_plantilla","False"
"cta120201","Depreciaciones Acumuladas","1.2.02.01","account.data_account_type_current_assets","l10n_gt.cuentas_plantilla","False"
"cta130101","Gastos por Amortizar","1.3.01.01","account.data_account_type_current_assets","l10n_gt.cuentas_plantilla","False"
"cta130201","Gastos Anticipados","1.3.02.01","account.data_account_type_current_assets","l10n_gt.cuentas_plantilla","False"
"cta130501","Gastos de Organización","1.3.03.01","account.data_account_type_current_assets","l10n_gt.cuentas_plantilla","False"
"cta130601","Otros Activos","1.3.04.01","account.data_account_type_current_assets","l10n_gt.cuentas_plantilla","False"
"cta210101","Cuentas y Documentos por Pagar","2.1.01.01","account.data_account_type_payable","l10n_gt.cuentas_plantilla","True"
"cta210201","IVA por Pagar","2.1.02.01","account.data_account_type_current_liabilities","l10n_gt.cuentas_plantilla","False"
"cta210301","Impuestos","2.1.03.01","account.data_account_type_current_liabilities","l10n_gt.cuentas_plantilla","False"
"cta220101","Provisión para Indemnizaciones","2.2.01.01","account.data_account_type_current_liabilities","l10n_gt.cuentas_plantilla","False"
"cta230101","Anticipos","2.3.01.01","account.data_account_type_current_liabilities","l10n_gt.cuentas_plantilla","False"
"cta310101","Capital Autorizado, Suscríto y Pagado","3.1.01.01","account.data_account_type_equity","l10n_gt.cuentas_plantilla","False"
"cta310102","Reservas","3.1.01.02","account.data_account_type_equity","l10n_gt.cuentas_plantilla","False"
"cta310103","Perdidas y Ganancias","3.1.01.03","account.data_account_type_equity","l10n_gt.cuentas_plantilla","False"
"cta410101","Ventas","4.1.01.01","account.data_account_type_revenue","l10n_gt.cuentas_plantilla","False"
"cta410102","Descuentos Sobre Ventas","4.1.01.02","account.data_account_type_revenue","l10n_gt.cuentas_plantilla","False"
"cta410103","Ganar cuenta","4.1.01.03","account.data_account_type_other_income","l10n_gt.cuentas_plantilla","False"
"cta420101","Otros Ingresos","4.2.01.01","account.data_account_type_revenue","l10n_gt.cuentas_plantilla","False"
"cta510101","Costos de Ventas","5.1.01.01","account.data_account_type_expenses","l10n_gt.cuentas_plantilla","False"
"cta610101","Gastos de Ventas","6.1.01.01","account.data_account_type_expenses","l10n_gt.cuentas_plantilla","False"
"cta620101","Gastos de Administración","6.2.01.01","account.data_account_type_expenses","l10n_gt.cuentas_plantilla","False"
"cta620201","Otros Gastos de Operación","6.2.02.01","account.data_account_type_expenses","l10n_gt.cuentas_plantilla","False"
"cta630101","Gastos no Deducibles","6.3.01.01","account.data_account_type_expenses","l10n_gt.cuentas_plantilla","False"
"cta710101","Otros Gastos Financieros","7.1.01.01","account.data_account_type_expenses","l10n_gt.cuentas_plantilla","False"
"cta710102","Intereses","7.1.01.02","account.data_account_type_expenses","l10n_gt.cuentas_plantilla","False"

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Compras e IVA por Cobrar -->

        <record id="impuestos_plantilla_iva_por_cobrar" model="account.tax.template">
            <field name="chart_template_id" ref="cuentas_plantilla"/>
            <field name="name">IVA por Cobrar</field>
            <field name="description">IVA por Cobrar</field>
            <field name="amount" eval="12"/>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="True"/>
            <field name="tax_group_id" ref="tax_group_iva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('cta110301'),
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
                    'account_id': ref('cta110301'),
                }),
            ]"/>
        </record>

        <!-- Ventas e IVA por Pagar -->

        <record id="impuestos_plantilla_iva_por_pagar" model="account.tax.template">
            <field name="chart_template_id" ref="cuentas_plantilla"/>
            <field name="name">IVA por Pagar</field>
            <field name="description">IVA por Pagar</field>
            <field name="amount" eval="12"/>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="price_include" eval="True"/>
            <field name="tax_group_id" ref="tax_group_iva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('cta210201'),
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
                    'account_id': ref('cta210201'),
                }),
            ]"/>
        </record>

        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_gt.cuentas_plantilla')]"/>
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
        <record id="tax_group_iva_12" model="account.tax.group">
            <field name="name">IVA 12%</field>
        </record>
    </data>
</odoo>

```

## File: data\l10n_gt_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="cuentas_plantilla" model="account.chart.template">
            <field name="name">Plantilla de cuentas de Guatemala (sencilla)</field>
            <field name="bank_account_code_prefix">1.0.01.0</field>
            <field name="cash_account_code_prefix">1.0.02.0</field>
            <field name="transfer_account_code_prefix">1.0.03.01</field>
            <field name="code_digits">9</field>
            <field name="currency_id" ref="base.GTQ"/>
        </record>
    </data>
</odoo>

```

## File: data\l10n_gt_chart_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="cuentas_plantilla" model="account.chart.template">
        <field name="property_account_receivable_id" ref="cta110201"/>
        <field name="property_account_payable_id" ref="cta210101"/>
        <field name="property_account_income_categ_id" ref="cta410101"/>
        <field name="property_account_expense_categ_id" ref="cta510101"/>
        <field name="income_currency_exchange_account_id" ref="cta410103"/>
        <field name="expense_currency_exchange_account_id" ref="cta710101"/>
        <field name="default_pos_receivable_account_id" ref="cta110205" />
    </record>
</odoo>

```

