# Odoo Module: l10n_cr

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

##############################################################################
#
#    __init__.py
#    l10n_cr_account
#    First author: Carlos Vásquez <carlos.vasquez@clearcorp.co.cr> (ClearCorp S.A.)
#    Copyright (c) 2010-TODAY ClearCorp S.A. (http://clearcorp.co.cr). All rights reserved.
#    
#    Redistribution and use in source and binary forms, with or without modification, are
#    permitted provided that the following conditions are met:
#    
#       1. Redistributions of source code must retain the above copyright notice, this list of
#          conditions and the following disclaimer.
#    
#       2. Redistributions in binary form must reproduce the above copyright notice, this list
#          of conditions and the following disclaimer in the documentation and/or other materials
#          provided with the distribution.
#    
#    THIS SOFTWARE IS PROVIDED BY <COPYRIGHT HOLDER> ``AS IS'' AND ANY EXPRESS OR IMPLIED
#    WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND
#    FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> OR
#    CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
#    CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
#    SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
#    ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
#    NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
#    ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#    
#    The views and conclusions contained in the software and documentation are those of the
#    authors and should not be interpreted as representing official policies, either expressed
#    or implied, of ClearCorp S.A..
#    
##############################################################################

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

##############################################################################
#
#    l10n_cr_account
#    First author: Carlos Vásquez <carlos.vasquez@clearcorp.co.cr> (ClearCorp S.A.)
#    Copyright (c) 2010-TODAY ClearCorp S.A. (http://clearcorp.co.cr). All rights reserved.
#
#    Redistribution and use in source and binary forms, with or without modification, are
#    permitted provided that the following conditions are met:
#
#        1. Redistributions of source code must retain the above copyright notice, this list of
#          conditions and the following disclaimer.
#
#        2. Redistributions in binary form must reproduce the above copyright notice, this list
#          of conditions and the following disclaimer in the documentation and/or other materials
#          provided with the distribution.
#
#    THIS SOFTWARE IS PROVIDED BY <COPYRIGHT HOLDER> ``AS IS'' AND ANY EXPRESS OR IMPLIED
#    WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND
#    FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> OR
#    CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
#    CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
#    SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
#    ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
#    NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
#    ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#
#    The views and conclusions contained in the software and documentation are those of the
#    authors and should not be interpreted as representing official policies, either expressed
#    or implied, of ClearCorp S.A..
#
##############################################################################

{
    'name': 'Costa Rica - Accounting',
    'url': 'https://github.com/CLEARCORP/odoo-costa-rica',
    'author': 'ClearCorp S.A.',
    'website': 'http://clearcorp.co.cr',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Chart of accounts for Costa Rica.
=================================

Includes:
---------
    * account.account.template
    * account.tax.template
    * account.chart.template

Everything is in English with Spanish translation. Further translations are welcome,
please go to http://translations.launchpad.net/openerp-costa-rica.
    """,
    'depends': ['account'],
    'data': [
        'data/l10n_cr_res_partner_title.xml',
        'data/l10n_cr_chart_data.xml',
        'data/account.account.template.csv',
        'data/account_data.xml',
        'data/account_chart_template_data.xml',
        'data/account_tax_template_data.xml',
        'data/account_chart_template_configure_data.xml',
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
"account_account_template_0_111301","0-Cuenta PayPal 1","0-111301","account.data_account_type_liquidity","l10n_cr.account_chart_template_0","False"
"account_account_template_0_111401","0-Fondos en tránsito en tesorería","0-111401","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_111403","0-Fondos en tránsito de PayPal a Bancos","0-111403","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_111501","0-Inversión 1","0-111501","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_112001","0-Cuentas por cobrar comerciales","0-112001","account.data_account_type_receivable","l10n_cr.account_chart_template_0","True"
"account_account_template_0_112011","0-Cuentas por cobrar comerciales (PoS)","0-112011","account.data_account_type_receivable","l10n_cr.account_chart_template_0","True"
"account_account_template_0_112002","0-Cuentas por cobrar a compañías relacionadas","0-112002","account.data_account_type_receivable","l10n_cr.account_chart_template_0","True"
"account_account_template_0_112003","0-Cuentas por cobrar a empleados","0-112003","account.data_account_type_receivable","l10n_cr.account_chart_template_0","True"
"account_account_template_0_112004","0-Otras cuentas por cobrar","0-112004","account.data_account_type_receivable","l10n_cr.account_chart_template_0","True"
"account_account_template_0_112005","0-Inversiones de corto plazo","0-112005","account.data_account_type_receivable","l10n_cr.account_chart_template_0","True"
"account_account_template_0_113101","0-Inventario de producto para la venta","0-113101","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_113102","0-Inventario de consumibles","0-113102","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_121111","0-Terreno 1","0-121111","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_121121","0-Terreno 1","0-121121","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_122111","0-Edificio 1","0-122111","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_122121","0-Edificio 1","0-122121","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_122211","0-Edificio 1","0-122211","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_122221","0-Edificio 1","0-122221","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_122301","0-Maquinaria y equipo de edificios","0-122301","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_122302","0-Herramientas mayores","0-122302","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_122303","0-Moibliario y equipo de oficina","0-122303","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_122304","0-Equipo de cómputo","0-122304","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_122305","0-Vehículos","0-122305","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_131101","0-Edificio 1","0-131101","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_131201","0-Edificio 1","0-131201","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_132101","0-Edificio 1","0-132101","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_132201","0-Edificio 1","0-132201","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_133001","0-Dep. ac. de maquinaria y equipo de edificios","0-133001","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_133002","0-Dep. ac. de herramientas mayores","0-133002","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_133003","0-Dep. ac. de mobiliario y equipo de oficina","0-133003","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_133004","0-Dep. ac. de equipo de cómputo","0-133004","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_133005","0-Dep. ac. de vehículos","0-133005","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","False"
"account_account_template_0_141001","0-Pólizas de seguros prepagadas","0-141001","account.data_account_type_receivable","l10n_cr.account_chart_template_0","True"
"account_account_template_0_142001","0-Depósitos sobre derechos telefónicos","0-142001","account.data_account_type_receivable","l10n_cr.account_chart_template_0","True"
"account_account_template_0_142002","0-Depósitos sobre conexiones de Internet","0-142002","account.data_account_type_receivable","l10n_cr.account_chart_template_0","True"
"account_account_template_0_142003","0-Depósitos sobre locales en alquiler","0-142003","account.data_account_type_receivable","l10n_cr.account_chart_template_0","True"
"account_account_template_0_211001","0-Cuentas por pagar a proveedores","0-211001","account.data_account_type_payable","l10n_cr.account_chart_template_0","True"
"account_account_template_0_211002","0-Cuentas por pagar a empleados","0-211002","account.data_account_type_payable","l10n_cr.account_chart_template_0","True"
"account_account_template_0_211003","0-Cuentas por pagar de provisiones","0-211003","account.data_account_type_payable","l10n_cr.account_chart_template_0","True"
"account_account_template_0_211004","0-Cuentas por pagar a compañías relacionadas","0-211004","account.data_account_type_payable","l10n_cr.account_chart_template_0","True"
"account_account_template_0_212101","0-Impuesto de ventas por pagar","0-212101","account.data_account_type_current_liabilities","l10n_cr.account_chart_template_0","True"
"account_account_template_0_212102","0-Impuesto de ventas pagado","0-212102","account.data_account_type_current_assets","l10n_cr.account_chart_template_0","True"
"account_account_template_0_212201","0-Impuesto de renta por pagar","0-212201","account.data_account_type_current_liabilities","l10n_cr.account_chart_template_0","True"
"account_account_template_0_212202","0-Adelantos de impuesto de renta","0-212202","account.data_account_type_current_liabilities","l10n_cr.account_chart_template_0","True"
"account_account_template_0_212203","0-Retenciones de impuesto de renta","0-212203","account.data_account_type_current_liabilities","l10n_cr.account_chart_template_0","True"
"account_account_template_0_310001","0-Socio 1","0-310001","account.data_account_type_equity","l10n_cr.account_chart_template_0","False"
"account_account_template_0_320001","0-Reserva legal","0-320001","account.data_account_type_equity","l10n_cr.account_chart_template_0","False"
"account_account_template_0_330001","0-Reserva para mejoras","0-330001","account.data_account_type_equity","l10n_cr.account_chart_template_0","False"
"account_account_template_0_330002","0-Reserva para proyectos","0-330002","account.data_account_type_equity","l10n_cr.account_chart_template_0","False"
"account_account_template_0_340001","0-Socio 1","0-340001","account.data_account_type_equity","l10n_cr.account_chart_template_0","False"
"account_account_template_0_350001","0-Superávit de capital","0-350001","account.data_account_type_equity","l10n_cr.account_chart_template_0","False"
"account_account_template_0_350002","0-Superavit por revaluación de activos","0-350002","account.data_account_type_equity","l10n_cr.account_chart_template_0","False"
"account_account_template_0_350003","0-Superavit ganado","0-350003","account.data_account_type_equity","l10n_cr.account_chart_template_0","False"
"account_account_template_0_360001","0-Periodo 1","0-360001","account.data_account_type_equity","l10n_cr.account_chart_template_0","False"
"account_account_template_0_370001","0-Utilidad o pérdida del período actual","0-370001","account.data_account_type_equity","l10n_cr.account_chart_template_0","False"
"account_account_template_0_380001","0-Balance inicial","0-380001","account.data_account_type_equity","l10n_cr.account_chart_template_0","False"
"account_account_template_0_410001","0-Categoría 1","0-410001","account.data_account_type_other_income","l10n_cr.account_chart_template_0","False"
"account_account_template_0_420001","0-Cuota por administración","0-420001","account.data_account_type_revenue","l10n_cr.account_chart_template_0","False"
"account_account_template_0_430001","0-Intereses ganados sobre cuentas corrientes","0-430001","account.data_account_type_revenue","l10n_cr.account_chart_template_0","False"
"account_account_template_0_440001","0-Ajustes","0-440001","account.data_account_type_revenue","l10n_cr.account_chart_template_0","False"
"account_account_template_0_440002","0-Donaciones","0-440002","account.data_account_type_revenue","l10n_cr.account_chart_template_0","False"
"account_account_template_0_450001","0-Diferencial cambiario","0-450001","account.data_account_type_revenue","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511111","0-Salarios","0-511111","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511112","0-Extras","0-511112","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511113","0-Bonificaciones","0-511113","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511114","0-Comisiones","0-511114","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511115","0-Cargas patronales","0-511115","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511116","0-Aguinaldo","0-511116","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511117","0-Preaviso","0-511117","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511118","0-Cesantía","0-511118","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511121","0-Transporte","0-511121","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511122","0-Alimentación","0-511122","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511123","0-Hospedaje","0-511123","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511201","0-Categoría 1","0-511201","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511301","0-Costo de producto","0-511301","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511302","0-Costo de materia prima","0-511302","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511303","0-Costo de producción","0-511303","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511304","0-Costo de almacenamiento","0-511304","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511305","0-Costo de distribución","0-511305","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511401","0-Diseño de imagen","0-511401","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_511402","0-Campañas publicitarias","0-511402","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_512101","0-Compañía administradora 1","0-512101","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_512201","0-Oficina 1","0-512201","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_512311","0-Medidor 1","0-512311","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_512321","0-Medidor 1","0-512321","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_512331","0-Teléfono 1","0-512331","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_512341","0-Contrato 1","0-512341","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_512401","0-Departamento 1","0-512401","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_512501","0-Departamento 1","0-512501","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_520001","0-Ajustes","0-520001","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_520002","0-Gastos Financieros","0-520002","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_520003","0-Depreciación de activo fijo","0-520003","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_520004","0-Perdida por robo","0-520004","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_520005","0-Donaciones deducibles","0-520005","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_530001","0-Donaciones no deducibles","0-530001","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_530002","0-Gastos de presidencia","0-530002","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_530003","0-Multas","0-530003","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
"account_account_template_0_530004","0-Diferencial cambiario","0-530004","account.data_account_type_expenses","l10n_cr.account_chart_template_0","False"
```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_cr.account_chart_template_0')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Account Chart Template -->
    <record id="account_chart_template_0" model="account.chart.template">
        <field name="property_account_receivable_id" ref="account_account_template_0_112001"/>
        <field name="property_account_payable_id" ref="account_account_template_0_211001"/>
        <field name="property_account_income_categ_id" ref="account_account_template_0_410001"/>
        <field name="property_account_expense_categ_id" ref="account_account_template_0_511301"/>
        <field name="income_currency_exchange_account_id" ref="account_account_template_0_450001"/>
        <field name="expense_currency_exchange_account_id" ref="account_account_template_0_530004"/>
        <field name="default_pos_receivable_account_id" ref="account_account_template_0_112011" />
    </record>
</odoo>

```

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Account Tax Group -->
        <record id="tax_group_13" model="account.tax.group">
            <field name="name">Tax 13%</field>
        </record>

    </data>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- Account Tax Template -->
        <record id="account_tax_template_IV_0" model="account.tax.template">
            <field name="name">Sales Tax</field>
            <field name="description">IV</field>
            <field name="sequence">10</field>
            <field name="amount">13</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="account_chart_template_0"/>
            <field name="tax_group_id" ref="tax_group_13"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_template_0_212101'),
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
                    'account_id': ref('account_account_template_0_212101'),
                }),
            ]"/>
        </record>
        <record id="account_tax_template_IV_1" model="account.tax.template">
            <field name="name">Purchase Tax</field>
            <field name="description">IV</field>
            <field name="sequence">10</field>
            <field name="amount">13</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="account_chart_template_0"/>
            <field name="tax_group_id" ref="tax_group_13"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_template_0_212101'),
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
                    'account_id': ref('account_account_template_0_212101'),
                }),
            ]"/>
        </record>

        <!-- Default Tax -->
        <record id="account_account_template_0_410001" model="account.account.template">
            <field name="tax_ids" eval="[(6,0,[ref('account_tax_template_IV_0')])]"/>
        </record>
</odoo>

```

## File: data\l10n_cr_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- Account Chart Template -->

        <record id="account_chart_template_0" model="account.chart.template">
            <field name="name">Costa Rica - Company 0</field>
            <field name="bank_account_code_prefix">0-1112</field>
            <field name="cash_account_code_prefix">0-1111</field>
            <field name="transfer_account_code_prefix">0-1114</field>
            <field name="currency_id" ref="base.CRC"/>
        </record>
</odoo>

```

## File: data\l10n_cr_res_partner_title.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!--
        Resource: res.partner.title
        Update partner titles
        -->
        <record id="res_partner_title_pvt_ltd" model="res.partner.title">
            <field name="name">Corporation</field>
            <field name="shortcut">Corp.</field>
        </record>
        <record id="res_partner_title_ltd" model="res.partner.title">
            <field name="name">Limited Company</field>
            <field name="shortcut">Ltd.</field>
        </record>
        <record id="res_partner_title_sal" model="res.partner.title">
            <field name="name">Sociedad An&#243;nima Laboral</field>
            <field name="shortcut">S.A.L.</field>
        </record>
        <record id="res_partner_title_asoc" model="res.partner.title">
            <field name="name">Asociation</field>
            <field name="shortcut">Asoc.</field>
        </record>
        <record id="res_partner_title_gov" model="res.partner.title">
            <field name="name">Government</field>
            <field name="shortcut">Gov.</field>
        </record>
        <record id="res_partner_title_edu" model="res.partner.title">
            <field name="name">Educational Institution</field>
            <field name="shortcut">Edu.</field>
        </record>
        <record id="res_partner_title_indprof" model="res.partner.title">
            <field name="name">Independant Professional</field>
            <field name="shortcut">Ind. Prof.</field>
        </record>
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
</odoo>

```

