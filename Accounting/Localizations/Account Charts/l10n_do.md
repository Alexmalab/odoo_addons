# Odoo Module: l10n_do

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Dominican Republic - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['do'],
    'version': '2.0',
    'category': 'Accounting/Localizations/Account Charts',
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
    'author': 'Gustavo Valverde - iterativo | Consultores de Odoo (http://iterativo.do)',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations.html',
    'depends': [
        'account',
        'base_iban',
    ],
    'data': [
        'data/account_tax_report_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.do"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_total_operaciones" model="account.report.line">
                <field name="name">II-1 TOTAL OPERATIONS FOR THE PERIOD</field>
                <field name="aggregation_formula">IIA_UNTAXED.balance + IIB_TAXED.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_2A" model="account.report.line">
                        <field name="name">II.A NON-TAXABLE</field>
                        <field name="code">IIA_UNTAXED</field>
                        <field name="aggregation_formula">II_A_2.balance + II_A_3.balance + II_A_4.balance + II_A_5.balance + II_A_6.balance + II_A_7.balance + II_A_8.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_2A2_base" model="account.report.line">
                                <field name="name">II.A.2 INCOME FROM EXPORTS OF GOODS</field>
                                <field name="code">II_A_2</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_2A2_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base.good.export</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_2A3_base" model="account.report.line">
                                <field name="name">II.A.3 INCOME FROM SERVICES EXPORTS</field>
                                <field name="code">II_A_3</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_2A3_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base.service.export</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_2A4_base" model="account.report.line">
                                <field name="name">II.A.4 INCOME FROM LOCAL SALES OF EXEMPT GOODS OR SERVICES</field>
                                <field name="code">II_A_4</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_2A4_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base.untaxed.sales.services</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_2A5_base" model="account.report.line">
                                <field name="name">II.A.5 INCOME FROM SALES OF EXEMPT GOODS OR SERVICES BY DESTINATION</field>
                                <field name="code">II_A_5</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_2A5_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base.untaxed.sales.destination</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_2A6_base" model="account.report.line">
                                <field name="name">II.A.6 NOT SUBJECT TO ITBIS FOR CONSTRUCTION SERVICES</field>
                                <field name="code">II_A_6</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_2A6_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base.untaxed.construction.service</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_2A7_base" model="account.report.line">
                                <field name="name">II.A.7 NOT SUBJECT TO ITBIS FOR COMMISSIONS</field>
                                <field name="code">II_A_7</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_2A7_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base.untaxed.commissions</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_2A8_base" model="account.report.line">
                                <field name="name">II.A.8 INCOME FROM LOCAL SALES OF EXEMPT GOODS</field>
                                <field name="code">II_A_8</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_2A8_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base.untaxed.sales</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_2B" model="account.report.line">
                        <field name="name">II.B. TAXED</field>
                        <field name="code">IIB_TAXED</field>
                        <field name="aggregation_formula">II_B_11.balance + II_B_12.balance + II_B_13.balance + II_B_14.balance + II_B_15.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_2B11_base" model="account.report.line">
                                <field name="name">II.B.11 TRANSACTIONS TAXED AT 18%</field>
                                <field name="code">II_B_11</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_2B11_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base.18%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_2B12_base" model="account.report.line">
                                <field name="name">II.B.12 TRANSACTIONS TAXED AT 16%</field>
                                <field name="code">II_B_12</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_2B12_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base.16%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_2B13_base" model="account.report.line">
                                <field name="name">II.B.13 TRANSACTIONS TAXED AT 9%</field>
                                <field name="code">II_B_13</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_2B13_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base.9%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_2B14_base" model="account.report.line">
                                <field name="name">II.B.13 TRANSACTIONS TAXED AT 8%</field>
                                <field name="code">II_B_14</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_2B14_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base.8%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_2B15_base" model="account.report.line">
                                <field name="name">II.B.15 TRANSACTIONS TAXED ON SALES OF DEPRECIABLE ASSETS</field>
                                <field name="code">II_B_15</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_2B15_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base.depreciable</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_3" model="account.report.line">
                <field name="name">III SETTLEMENT</field>
                <field name="aggregation_formula">III_21.balance + III_25.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_3_21_total" model="account.report.line">
                        <field name="name">III.21 TOTAL ITBIS COLLECTED (Add Boxes 16+17+18+19+20)</field>
                        <field name="code">III_21</field>
                        <field name="aggregation_formula">III_16.balance + III_17.balance + III_18.balance + III_19.balance + III_20.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_3_16_tax" model="account.report.line">
                                <field name="name">III.16 ITBIS COLLECTED (18% of box 11)</field>
                                <field name="code">III_16</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_3_16_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax.18%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_3_17_tax" model="account.report.line">
                                <field name="name">III.17 ITBIS COLLECTED (16% of box 12)</field>
                                <field name="code">III_17</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_3_17_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax.16%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_3_18_tax" model="account.report.line">
                                <field name="name">III.18 ITBIS COLLECTED (9% of box 13)</field>
                                <field name="code">III_18</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_3_18_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax.9%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_3_19_tax" model="account.report.line">
                                <field name="name">III.19 ITBIS COLLECTED (8% of box 14)</field>
                                <field name="code">III_19</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_3_19_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax.8%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_3_20_tax" model="account.report.line">
                                <field name="name">III.20 ITBIS CHARGED ON SALES OF DEPRECIABLE ASSETS</field>
                                <field name="code">III_20</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_3_20_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax.decreciable</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_3_25_total" model="account.report.line">
                        <field name="name">III.25 TOTAL ITBIS DEDUCTIBLE (Add boxes 22+23+24)</field>
                        <field name="code">III_25</field>
                        <field name="aggregation_formula">III_22.balance + III_23.balance + III_24.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_3_22_tax" model="account.report.line">
                                <field name="name">III.22 ITBIS PAID ON LOCAL PURCHASES</field>
                                <field name="code">III_22</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_3_22_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax.paid.purchase</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_3_23_tax" model="account.report.line">
                                <field name="name">III.23 ITBIS PAID FOR DEDUCTIBLE SERVICES</field>
                                <field name="code">III_23</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_3_23_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax.paid.service</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_3_24_tax" model="account.report.line">
                                <field name="name">III.24 ITBIS PAID ON IMPORTS</field>
                                <field name="code">III_24</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_3_24_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax.paid.imports</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_A" model="account.report.line">
                <field name="name">A ITBIS WITHHELD</field>
                <field name="aggregation_formula">A39.balance + A50.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_A39_base" model="account.report.line">
                        <field name="name">A.39 Services Subject to Withholding Individuals = Individuals and Entity</field>
                        <field name="code">A39</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_A39_base_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">A.39</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_A50_tax" model="account.report.line">
                        <field name="name">A.50 ITBIS For Services Subject To Withholding Taxes Individuals</field>
                        <field name="code">A50</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_A50_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">A.50</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-do.csv

```csv
"id","code","name","account_type","reconcile","name@es"
"l10n_do_11010301","11010301","Short-term deposits in RD","asset_cash","False",""
"l10n_do_11010302","11010302","Short-term deposits in USD","asset_cash","False",""
"l10n_do_11020100","11020100","Bonds and Temporary Shares","asset_current","False","Bonos y Acciones Temporales"
"l10n_do_11020200","11020200","Stock market operations","asset_current","False","Operaciones en Bolsa"
"l10n_do_11020300","11020300","Other marketable securities","asset_current","False","Otros Valores Negociables"
"l10n_do_11030101","11030101","Returned Checks Receivable","asset_receivable","True","Cheques Devueltos por Cobrar"
"l10n_do_11030102","11030102","Interest receivable","asset_receivable","True","Intereses por Cobrar"
"l10n_do_11030201","11030201","Accounts Receivable from Customers","asset_receivable","True","Cuentas por Cobrar a Clientes"
"l10n_do_11030210","11030210","Trade Accounts Receivable (PoS)","asset_receivable","True","Cuentas por Cobrar a Clientes (PoS)"
"l10n_do_11030202","11030202","Accounts Receivable from Officers and Employees","asset_receivable","True","Cuentas por Cobrar a Funcionarios y Empleados"
"l10n_do_11030203","11030203","Accounts Receivable from Affiliates","asset_receivable","True","Cuentas por Cobrar a Afiliadas"
"l10n_do_11030204","11030204","Accounts Receivable from Shareholders","asset_receivable","True","Cuentas por Cobrar a Accionistas"
"l10n_do_11030205","11030205","Other Accounts Receivable","asset_receivable","True","Otras Cuentas por Cobrar"
"l10n_do_11040100","11040100","Allowance for doubtful accounts receivable from customers","asset_current","False","Provisiones de Incobrables a Clientes"
"l10n_do_11040200","11040200","Provisions for Uncollectible Provisions to Officers and Employees","asset_current","False","Provisiones de Incobrables a Funcionarios y Empleados"
"l10n_do_11040300","11040300","Allowance for Uncollectible Provisions to Affiliates","asset_current","False","Provisiones de Incobrables a Afiliadas"
"l10n_do_11040400","11040400","Provisions for Uncollectible Provisions to Shareholders","asset_current","False","Provisiones de Incobrables a Accionistas"
"l10n_do_11040500","11040500","Provisions for Other Uncollectible Accounts","asset_current","False","Provisiones de Otras Cuentas Incobrables"
"l10n_do_11050100","11050100","Inventory of merchandise or finished products","asset_current","False","Inventario de Mercancías o Productos Terminados"
"l10n_do_11050200","11050200","Raw Materials Inventory","asset_current","False","Inventario de Materia Prima"
"l10n_do_11050300","11050300","Inventory in Transit","asset_current","False","Inventario en Tránsito"
"l10n_do_11050400","11050400","Materials and Supplies Inventory","asset_current","False","Inventario de Materiales y Suministros"
"l10n_do_11050500","11050500","Fuel Inventory","asset_current","False","Inventario de Combustibles"
"l10n_do_11050600","11050600","Goods Shipped Not Invoiced","asset_current","True","Bienes Enviados No Facturados"
"l10n_do_11060100","11060100","Impairment Accum. of Inventory in Warehouse","asset_current","False","Deterioro Acum. de Inventario en Almacen"
"l10n_do_11060200","11060200","Accumulated Impairment of Materials and Supplies","asset_current","False","Deterioro Acum. de Materiales y Suministros"
"l10n_do_11060300","11060300","Impairment of Fuel Accum.","asset_current","False","Deterioro Acum. de Combustibles"
"l10n_do_11070100","11070100","Warehouse Inventory Obsolescence","asset_current","False","Obsolescencia de Inventario en Almacen"
"l10n_do_11070200","11070200","Obsolescence of Materials and Supplies","asset_current","False","Obsolescencia de Materiales y Suministros"
"l10n_do_11070300","11070300","Fuel Obsolescence","asset_current","False","Obsolescencia de Combustibles"
"l10n_do_11080101","11080101","ITBIS Paid on Purchases of Local Good","asset_current","False","ITBIS Pagado en Compras de Bienes Locales"
"l10n_do_11080102","11080102","ITBIS Paid on Deductible Service Purchases","asset_current","False","ITBIS Pagado en Compras de Servicios Deducibles"
"l10n_do_11080103","11080103","ITBIS Paid on Imports","asset_current","False","ITBIS Pagado en Importaciones"
"l10n_do_11080104","11080104","ITBIS on Purchases of Goods or Services Subject to Proportionality","asset_current","False","ITBIS en Compras de Bienes o Servicios Sujetos a Proporcionalidad"
"l10n_do_11080105","11080105","ITBIS on Imports Subject to Proportionalit","asset_current","False","ITBIS en Importaciones Sujeto a Proporcionalidad"
"l10n_do_11080106","11080106","ITBIS Paid on Imports Subject to Proportionality","asset_current","False","ITBIS Pagado en Importaciones Sujeto a Proporcionalidad"
"l10n_do_11080301","11080301","ITBIS Withheld on Sales (N08-04 and N02-05)","asset_current","False","ITBIS Retenido por Ventas (N08-04 y N02-05)"
"l10n_do_11080302","11080302","ITBIS Retained by State Entities (5%)","asset_current","False","ITBIS Retenido por Entidades del Estado (5%)"
"l10n_do_11080303","11080303","Balance in Favor Previous","asset_current","False","Saldo a Favor Anterior"
"l10n_do_11090100","11090100","Temporary Actions","asset_current","False","Acciones Temporales"
"l10n_do_11090200","11090200","Temporary Time Deposits","asset_current","False","Depósitos a Plazo Temporales"
"l10n_do_11090300","11090300","Temporary Bonds","asset_current","False","Bonos Temporales"
"l10n_do_11090400","11090400","Bonds and Deposits","asset_current","False","Fianzas y Depósitos"
"l10n_do_11100100","11100100","Advance to Suppliers","asset_prepayments","False","Avance a Proveedores"
"l10n_do_11100200","11100200","Prepaid rents","asset_prepayments","False","Alquileres pagados por Anticipado"
"l10n_do_11100300","11100300","Advances for Insurance","asset_prepayments","False","Anticipos para Seguros"
"l10n_do_11100400","11100400","Advances for income tax","asset_prepayments","False","Anticipos para ISR"
"l10n_do_11100500","11100500","Advances for Expenses","asset_prepayments","False","Anticipos para Gastos"
"l10n_do_11100600","11100600","Advances for 1% Assets","asset_prepayments","False","Anticipos para 1% Activos"
"l10n_do_12010101","12010101","Land","asset_fixed","False","Terrenos"
"l10n_do_12010102","12010102","Buildings","asset_fixed","False","Edificaciones"
"l10n_do_12010103","12010103","Building Improvements","asset_fixed","False","Mejoras en Edificaciones"
"l10n_do_12010104","12010104","Improvements to Leased Premises and Buildings","asset_fixed","False","Mejoras en Edificaciones y Locales Arrendados"
"l10n_do_12010201","12010201","Light Transportation Equipment","asset_fixed","False","Equipos de Transporte Liviano"
"l10n_do_12010202","12010202","Office Furniture and Equipment","asset_fixed","False","Mobiliarios y Equipos de Oficina"
"l10n_do_12010301","12010301","Machinery","asset_fixed","False","Maquinarias"
"l10n_do_12010302","12010302","Tools","asset_fixed","False","Herramientas"
"l10n_do_12010303","12010303","Facilities","asset_fixed","False","Instalaciones"
"l10n_do_12010304","12010304","Camera and Security Systems","asset_fixed","False","Sistemas de Cámara y Seguridad"
"l10n_do_12010305","12010305","Miscellaneous or other assets","asset_fixed","False","Misceláneos u otros activos"
"l10n_do_12020100","12020100","Accumulated Depreciation of Buildings","asset_non_current","False","Depreciación Acum. de Edificios"
"l10n_do_12020200","12020200","Accumulated Depreciation of Transportation Equipment","asset_non_current","False","Depreciación Acum. de Equipos de Transporte"
"l10n_do_12020300","12020300","Accumulated Depreciation of Furniture and Equipment","asset_non_current","False","Depreciación Acum. de Mobiliarios y Equipos"
"l10n_do_12020400","12020400","Accumulated Depreciation of Machinery","asset_non_current","False","Depreciación Acum. de Maquinarias"
"l10n_do_12020500","12020500","Accumulated depreciation of tools","asset_non_current","False","Depreciación Acum. de Herramientas"
"l10n_do_12020600","12020600","Accumulated Depreciation of Facilities","asset_non_current","False","Depreciación Acum. de Instalaciones"
"l10n_do_12040100","12040100","Trademarks and Patents","asset_fixed","False","Marcas y Patentes"
"l10n_do_12040200","12040200","Licenses and Software","asset_fixed","False","Licencias y Programas Informáticos"
"l10n_do_12050100","12050100","Leased Buildings and Premises","asset_fixed","False","Edificaciones y Locales Arrendados"
"l10n_do_12050200","12050200","Leased Machinery and Equipment","asset_fixed","False","Maquinarias y Equipos Arrendados"
"l10n_do_12050300","12050300","Leased Facilities","asset_fixed","False","Instalaciones Arrendadas"
"l10n_do_12060100","12060100","Accumulated Depreciation Buildings and Leased Premises","asset_non_current","False","Depreciación Acum. Edificaciones y Locales Arrendados"
"l10n_do_12060200","12060200","Accumulated Depreciation Leased Machinery and Equipment","asset_non_current","False","Depreciación Acum. Maquinarias y Equipos Arrendados"
"l10n_do_12060300","12060300","Accumulated Depreciation Leased Facilities","asset_non_current","False","Depreciación Acum. Instalaciones Arrendadas"
"l10n_do_12070100","12070100","Accumulated Impairment Leased Buildings and Premises","asset_non_current","False","Deterioro Valor Acum. Edificios y Locales Arrendados"
"l10n_do_12070200","12070200","Accumulated Impairment Leased Machinery and Equipment","asset_non_current","False","Deterioro Valor Acum. Maquinaria y Equipo Arrendado"
"l10n_do_12070300","12070300","Accumulated Impairment Leased Facilities","asset_non_current","False","Deterioro Valor Acum. Instalaciones Arrendadas"
"l10n_do_12080100","12080100","Shares in other companies","asset_non_current","False","Acciones en otras sociedades"
"l10n_do_12090100","12090100","Deductible temporary differences","asset_non_current","False","Diferencias temporales deducibles"
"l10n_do_12090200","12090200","Deferred Organizational Expenses","asset_non_current","False","Gastos Organizacionales Diferidos"
"l10n_do_12090300","12090300","Deductible ISR","asset_non_current","False","ISR Deducible"
"l10n_do_13010100","13010100","Construction in Progress Costs","asset_non_current","False","Costos de Construcción en Proceso"
"l10n_do_21010100","21010100","Bank loans payable","liability_payable","True","Préstamos Bancarios por Pagar"
"l10n_do_21010200","21010200","Accounts Payable to Local Suppliers","liability_payable","True",""
"l10n_do_21010300","21010300","Accounts Payable to Foreign Suppliers","liability_payable","True","Cuentas por Pagar a Proveedores del Exterior"
"l10n_do_21010400","21010400","Accounts Payable to Affiliates","liability_payable","True","Cuentas por Pagar a Afiliadas"
"l10n_do_21010500","21010500","Accounts Payable to Shareholders","liability_payable","True","Cuentas por Pagar a Accionistas"
"l10n_do_21010600","21010600","Payrolls Payable","liability_payable","True","Nóminas por Pagar"
"l10n_do_21010700","21010700","Interest payable","liability_payable","True","Intereses por Pagar"
"l10n_do_21010800","21010800","Other accounts payable","liability_payable","True","Otras Cuentas por Pagar"
"l10n_do_21020100","21020100","Dividends Payable","liability_non_current","False","Dividendos por Pagar"
"l10n_do_21020200","21020200","Commissions Payable","liability_non_current","False","Comisiones por Pagar"
"l10n_do_21020300","21020300","Vacation Payable","liability_non_current","False","Vacaciones por Pagar"
"l10n_do_21020400","21020400","Bonuses Payable","liability_non_current","False","Bonificaciones por Pagar"
"l10n_do_21020500","21020500","Easter Royalty Payable","liability_non_current","False","Regalía Pascual por Pagar"
"l10n_do_21020600","21020600","Paid Medical Insurance","liability_non_current","False","Seguro Médico por Pagar"
"l10n_do_21020700","21020700","Family Health Insurance (FHIS) Payable","liability_non_current","False","Seguro Familiar de Salud (SFS) por Pagar"
"l10n_do_21020800","21020800","Pension Fund (AFP) Contribution Payable","liability_non_current","False","Aporte a Fondo de Pensiones (AFP) por Pagar"
"l10n_do_21020900","21020900","Labor Risk Insurance (SRL) Payable","liability_non_current","False","Seguro de Riesgo Laboral (SRL) por Pagar"
"l10n_do_21021000","21021000","INFOTEP Payable","liability_payable","True","INFOTEP por Pagar"
"l10n_do_21021100","21021100","Social Security Treasury (TSS) Payable","liability_payable","True","Tesorería de la Seguridad Social (TSS) por Pagar"
"l10n_do_21021200","21021200","Unbilled Goods Received","liability_non_current","True","Bienes Recibidos no Facturados"
"l10n_do_21030101","21030101","ITBIS on Sale of Services","liability_non_current","False","ITBIS por Venta Servicios"
"l10n_do_21030102","21030102","ITBIS on Sale of Goods","liability_non_current","False","ITBIS por Venta Bienes"
"l10n_do_21030103","21030103","ITBIS for sale of services on behalf of 3th Parties","liability_non_current","False","ITBIS por Venta Servicios en Nombre de Terceros"
"l10n_do_21030201","21030201","ITBIS Withheld from Legal Entity (N02-05)","liability_non_current","False","ITBIS Retenido a Persona Jurídica (N02-05)"
"l10n_do_21030202","21030202","ITBIS Withheld from Individuals (R293-11)","liability_non_current","False","ITBIS Retenido a Persona Física (R293-11)"
"l10n_do_21030203","21030203","ITBIS Withheld from Non-Profit Entities (N01-11)","liability_non_current","False","ITBIS Retenido a Entidades No Lucrativas (N01-11)"
"l10n_do_21030204","21030204","ITBIS Withheld for Professional Services (N02-05)","liability_non_current","False","ITBIS Retenido por Servicios Profesionales (N02-05)"
"l10n_do_21030205","21030205","ITBIS Withheld from Informal Goods (N08-10)","liability_non_current","False","ITBIS Retenido a Informales de Bienes (N08-10)"
"l10n_do_21030301","21030301","Income tax withheld for professional fees of individuals","liability_non_current","False","ISR Retenido por Honorarios Profesionales de Personas Físicas"
"l10n_do_21030302","21030302","ISR withheld on rent paid to individuals","liability_non_current","False","ISR Retenido por Alquileres Pagados a Personas Físicas"
"l10n_do_21030303","21030303","Income tax withheld on dividends paid","liability_non_current","False","ISR Retenido por Dividendos Pagados"
"l10n_do_21030304","21030304","ISR Withheld on Interest Paid Abroad","liability_non_current","False","ISR Retenido por Intereses Pagados al Exterior"
"l10n_do_21030305","21030305","ISR withheld on interest paid","liability_non_current","False","ISR Retenido por Intereses Pagados"
"l10n_do_21030306","21030306","ISR withheld on transfers of securities and properties","liability_non_current","False","ISR Retenido por Transferencias de Títulos y Propiedades"
"l10n_do_21030307","21030307","ISR Withheld on Remittances Abroad (L253-12)","liability_non_current","False","ISR Retenido por Remesas al Exterior (L253-12)"
"l10n_do_21030308","21030308","Other Withholdings (N07-07)","liability_non_current","False","Otras Retenciones (N07-07)"
"l10n_do_21030309","21030309","Other Withholdings","liability_non_current","False","Otras Retenciones"
"l10n_do_21030401","21030401","Income Tax Withholding (IR3 ISR)","liability_non_current","False","Retención de Impuesto al Salario (ISR del IR3)"
"l10n_do_21030402","21030402","Family Health Insurance (FHI) Withholding","liability_non_current","False","Retención de Seguro Familiar de Salud (SFS)"
"l10n_do_21030403","21030403","Pension Fund Retention (AFP)","liability_non_current","False","Retención de Fondo de Pensiones (AFP)"
"l10n_do_21030404","21030404","INFOTEP retention (0.5%)","liability_non_current","False","Retención de INFOTEP (0.5%)"
"l10n_do_21030501","21030501","ISR payable","liability_non_current","False","ISR por Pagar"
"l10n_do_21030502","21030502","Advances ISR payable","liability_non_current","False","Anticipos ISR por Pagar"
"l10n_do_21030503","21030503","Statutory Propina Payable (L16-92)","liability_non_current","False","Propina Legal por Pagar (L16-92)"
"l10n_do_21030504","21030504","Other Taxes Payable","liability_non_current","False","Otros Impuestos por Pagar"
"l10n_do_21040100","21040100","Provisions for Rent Payments","liability_non_current","False","Provisiones Pago Alquileres"
"l10n_do_21040200","21040200","Provisions Finance Lease","liability_non_current","False","Provisiones Arrendamiento Financiero"
"l10n_do_21040300","21040300","Provisions Fixed Expenses","liability_non_current","False","Provisiones Gastos Fijos"
"l10n_do_21050100","21050100","Business Credit Card","liability_non_current","False","Tarjeta de Crédito Empresarial"
"l10n_do_21050200","21050200","Credit Card Premium","liability_non_current","False","Prima de Tarjetas de Crédito"
"l10n_do_22010100","22010100","Long-Term Bank Loans","liability_non_current","False","Préstamos Bancarios Largo Plazo"
"l10n_do_22010200","22010200","LP Mortgage Loans","liability_non_current","False","Préstamos Hipotecarios LP"
"l10n_do_22012300","22012300","Shareholder / Individual Loans","liability_non_current","False","Préstamos Accionistas / Particulares"
"l10n_do_22020100","22020100","Provisions Employee Benefits LP","liability_non_current","False","Provisiones Beneficios de Empleados LP"
"l10n_do_22020200","22020200","Provisions for employee benefits","liability_non_current","False","Provisiones Prestaciones Laborales"
"l10n_do_22020300","22020300","Provisions Indemnifications","liability_non_current","False","Provisiones Indemnizaciones"
"l10n_do_22030100","22030100","Customer Advances or Guarantees","liability_non_current","False","Anticipos o Garantías de Clientes"
"l10n_do_22030200","22030200","Deposits to be identified","liability_non_current","False","Depósitos por Identificar"
"l10n_do_22040100","22040100","Provisions Finance Lease LP","liability_non_current","False","Provisiones Arrendamiento Financiero LP"
"l10n_do_31010100","31010100","Authorized Capital Stock","equity","False","Capital Social Autorizado"
"l10n_do_31010200","31010200","Capital Stock in Unissued Shares","equity","False","Capital Social en Acciones no Emitidas"
"l10n_do_31010300","31010300","Paid-in capital stock","equity","False","Capital Social Pagado"
"l10n_do_31010400","31010400","Unpaid Capital Stock","equity","False","Capital Social No Pagado"
"l10n_do_31010500","31010500","Shares or Treasury Stock","equity","False","Acciones o Participaciones en Tesorería"
"l10n_do_31020100","31020100","Land revaluation surplus","equity","False","Superávit por Revaluación de Terrenos"
"l10n_do_31020200","31020200","Surplus from Revaluation of Buildings","equity","False","Superávit por Revaluación de Edificaciones"
"l10n_do_31020300","31020300","Facilities Revaluation Surplus","equity","False","Superávit por Revaluación de Instalaciones"
"l10n_do_31020400","31020400","Surplus from revaluation of furniture and equipment","equity","False","Superávit por Revaluación de Mobiliario y Equipo"
"l10n_do_32010100","32010100","Legal Reserve","equity","False","Reserva Legal"
"l10n_do_33010100","33010100","Utilidades de Ejercicios Anteriores","equity","False",""
"l10n_do_33010200","33010200","Prior Years' Losses","equity","False","Pérdidas de Ejercicios Anteriores"
"l10n_do_33020100","33020100","Profit for the year","equity","False","Utilidad del Ejercicio"
"l10n_do_33020200","33020200","Loss for the year","equity","False","Pérdidas del Ejercicio"
"l10n_do_33040100","33040100","Surplus in Reserve","equity","False","Superávit en Reserva"
"l10n_do_33040200","33040200","Deficit in Reserve","equity","False","Déficit en Reserva"
"l10n_do_41010100","41010100","Sales of Goods","income","False","Ventas de Bienes"
"l10n_do_41010200","41010200","Sales from Exports of Goods","income","False","Ventas por Exportaciones de Bienes"
"l10n_do_41020100","41020100","Sales of Services","income","False","Ventas de Servicios"
"l10n_do_41020200","41020200","Sales from Services Exports","income","False","Ventas por Exportaciones de Servicios"
"l10n_do_41030100","41030100","Return of Goods","income","False","Devoluciones de Bienes"
"l10n_do_41030200","41030200","Returns for Services","income","False","Devoluciones por Servicios"
"l10n_do_41040100","41040100","Sales discounts (per NC)","income","False","Descuentos por Ventas (por NC)"
"l10n_do_42010100","42010100","Interest on Certificates","income","False","Intereses sobre Certificados"
"l10n_do_42010200","42010200","Interest on Bank Accounts","income","False","Intereses por Cuentas Bancarias"
"l10n_do_42010300","42010300","Interest on financing","income","False","Intereses por Financiamientos"
"l10n_do_42020100","42020100","Revenues from Sales of Depreciable Assets (CAT II and III)","income","False","Ingresos por Ventas de Activos Depreciables (CAT II y III)"
"l10n_do_42030100","42030100","Dividend Income Earned","income","False","Ingresos por Dividendos Ganados"
"l10n_do_42040100","42040100","Income from Exchange Differences","income_other","False","Ingresos por Diferencia Cambiaria"
"l10n_do_42040400","42040400","Cash Discount Gain","income_other","False","Ganancia por descuento"
"l10n_do_42040200","42040200","Collection of uncollectible accounts","income_other","False","Cobro de Cuentas Incobrables"
"l10n_do_42040300","42040300","Surplus Cash Revenues","income_other","False","Ingresos por Sobrante en Caja"
"l10n_do_51010100","51010100","Cost of Goods","expense_direct_cost","False","Costos de Bienes"
"l10n_do_51010200","51010200","Cost of Services","expense_direct_cost","False","Costos de Servicios"
"l10n_do_51010300","51010300","Price Difference","expense_direct_cost","False","Diferencia en Precio"
"l10n_do_51020400","51020400","Import Costs","expense_direct_cost","False","Costos de Importación"
"l10n_do_51010500","51010500","ITBIS carried at Cost","expense_direct_cost","False","ITBIS llevado al Costo"
"l10n_do_51020100","51020100","Raw material costs","expense_direct_cost","False","Costos de Materia Prima"
"l10n_do_51020200","51020200","Labor Costs","expense_direct_cost","False","Costos de Mano de Obra"
"l10n_do_51020300","51020300","Indirect Costs","expense_direct_cost","False","Costos Indirectos"
"l10n_do_61010100","61010100","Wages and Salaries","expense","False","Sueldos y Salarios"
"l10n_do_61010200","61010200","Easter Royalty","expense","False","Regalía Pascual"
"l10n_do_61010300","61010300","Supplementary Remuneration","expense","False","Retribuciones Complementarias"
"l10n_do_61010400","61010400","Vacation Bonus","expense","False","Bono Vacacional"
"l10n_do_61010500","61010500","Performance Bonus","expense","False","Bono por Desempeño"
"l10n_do_61010600","61010600","Employee Commissions","expense","False","Comisiones a Empleados"
"l10n_do_61010700","61010700","Indemnifications","expense","False","Indemnizaciones"
"l10n_do_61010800","61010800","Bonuses and Gratuities","expense","False","Bonificaciones y Gratificaciones"
"l10n_do_61010900","61010900","Meals or Per Diems to Personnel","expense","False","Comidas o Dietas al Personal"
"l10n_do_61011000","61011000","Courses and Training","expense","False","Cursos y Entrenamientos"
"l10n_do_61011100","61011100","Labor Benefits (Notice and Severance Pay)","expense","False","Prestaciones Laborales (Preaviso y Cesantia)"
"l10n_do_61011200","61011200","Incentives","expense","False","Incentivos"
"l10n_do_61011300","61011300","Overtime","expense","False","Horas Extras"
"l10n_do_61011400","61011400","Uniforms","expense","False","Uniformes"
"l10n_do_61011500","61011500","Other Personnel Expenses","expense","False","Otros Gastos de Personal"
"l10n_do_61010102","61010102","Contribution to the Pension Fund Administrators (AFP)","expense","False","Contribución a la Administradora de Fondos de Pensiones (AFP)"
"l10n_do_61010103","61010103","Contribution to Occupational Risk Insurance (SRL)","expense","False","Contribución al Seguro Riesgo Laboral (SRL)"
"l10n_do_61010104","61010104","Contribution to the Family Health Insurance (FHIS)","expense","False","Contribución al Seguro Familiar de Salud (SFS)"
"l10n_do_61010201","61010201","Contribution to the Health Risk Management System (ARS)","expense","False","Contribución a la Administradora de Riesgos de Salud (ARS)"
"l10n_do_61010202","61010202","Contribution to Staff Insurance","expense","False","Contribución a Seguros del Personal"
"l10n_do_61010203","61010203","Contribution to Supplemental Health Plans","expense","False","Contribución a Planes Complementarios de Salud"
"l10n_do_61010204","61010204","Life Insurance Contribution","expense","False","Contribución Seguros de Vida"
"l10n_do_61010205","61010205","Contribution to INFOTEP","expense","False","Contribución al INFOTEP"
"l10n_do_61020100","61020100","Electric Power","expense","False","Energía Eléctrica"
"l10n_do_61020200","61020200","Communications","expense","False","Comunicaciones"
"l10n_do_61020300","61020300","Office Supplies (Stationery and supplies)","expense","False","Suministros de Oficina (Papelería y útiles)"
"l10n_do_61020400","61020400","Cleaning and Cleaning Supplies","expense","False","Útiles de Aseo y Limpieza"
"l10n_do_61020500","61020500","Water and Garbage","expense","False","Agua y Basura"
"l10n_do_61020600","61020600","Building and Premises Insurance","expense","False","Seguro de Edificio y Locales"
"l10n_do_61020700","61020700","Fuels and Lubricants","expense","False","Combustibles y Lubricantes"
"l10n_do_61020800","61020800","Rentals / Leases","expense","False","Alquileres / Arrendamientos"
"l10n_do_61020900","61020900","Franchises","expense","False","Franquicias"
"l10n_do_61021000","61021000","Events","expense","False","Eventos"
"l10n_do_61021100","61021100","Insurance","expense","False","Seguros"
"l10n_do_61021200","61021200","Messaging Services","expense","False","Servicios de Mensajería"
"l10n_do_61021300","61021300","Freight and Cargo","expense","False","Flete y Carga"
"l10n_do_61021400","61021400","Fees and Subscriptions","expense","False","Cuotas y Suscripciones"
"l10n_do_61021500","61021500","Hotel Accommodations","expense","False","Alojamiento en Hoteles"
"l10n_do_61030101","61030101","Legal (Individual)","expense","False","Legales (P. Física)"
"l10n_do_61030102","61030102","Accounting and Auditing (Individual)","expense","False","Contabilidad y Auditoría (P. Física)"
"l10n_do_61030103","61030103","Technology (Individual)","expense","False","Tecnología (P. Física)"
"l10n_do_61030104","61030104","Plant Maintenance (Individual)","expense","False","Mantenimientos de Planta (P. Física)"
"l10n_do_61030105","61030105","Premises Maintenance (Personal)","expense","False","Mantenimientos de Local (P. Física)"
"l10n_do_61030106","61030106","Furniture and Equipment Maintenance (Personal)","expense","False","Mantenimientos Mobiliarios y Equipos (P. Física)"
"l10n_do_61030107","61030107","Counseling (Individual)","expense","False","Asesorías (P. Física)"
"l10n_do_61030108","61030108","Fumigation (Individal)","expense","False","Fumigaciones (P. Física)"
"l10n_do_61030109","61030109","Copies and Scans (Individual)","expense","False","Copias y Escaneos (P. Física)"
"l10n_do_61030110","61030110","Surveillance Services (Individual)","expense","False","Servicios de Vigilancia (P. Física)"
"l10n_do_61030111","61030111","Other Professional Services (Individual)","expense","False","Otros Servicios Profesionales (P. Física)"
"l10n_do_61030201","61030201","Legal (Legal)","expense","False","Legales (P. Jurídica)"
"l10n_do_61030202","61030202","Accounting and Auditing (Legal)","expense","False","Contabilidad y Auditoría (P. Jurídica)"
"l10n_do_61030203","61030203","Technology (Legal)","expense","False","Tecnología (P. Jurídica)"
"l10n_do_61030204","61030204","Plant Maintenance (Legal)","expense","False","Mantenimiento de Planta (P. Jurídica)"
"l10n_do_61030205","61030205","Maintenance of Premises (Legal","expense","False","Mantenimiento del Local (P. Jurídica)"
"l10n_do_61030206","61030206","Furniture and Equipment Maintenance (Legal)","expense","False","Mantenimiento Mobiliario y Equipos (P. Jurídica)"
"l10n_do_61030207","61030207","Counseling (Legal)","expense","False","Asesorías (P. Jurídica)"
"l10n_do_61030208","61030208","Fumigation (Legal)","expense","False","Fumigaciones (P. Jurídica)"
"l10n_do_61030209","61030209","Copies and Scans (Legal)","expense","False","Copias y Escaneos (P. Jurídica)"
"l10n_do_61030210","61030210","Surveillance Services (Legal)","expense","False","Servicios de Vigilancia (P. Jurídica)"
"l10n_do_61030211","61030211","Other Professional Services (Legal)","expense","False","Otros Servicios Profesionales  (P. Jurídica)"
"l10n_do_61030301","61030301","Foreign Service Fees - Related","expense","False","Honorarios por Servicios del Exterior - Relacionadas"
"l10n_do_61030302","61030302","Foreign Services Fees - Third Parties","expense","False","Honorarios por Servicios del Exterior - Terceros"
"l10n_do_61040100","61040100","Building Depreciation Expense","expense_depreciation","False","Gastos por Depreciación de Edificios"
"l10n_do_61040200","61040200","Transportation Equipment Depreciation Expense","expense_depreciation","False","Gastos por Depreciación de Equipo de Transporte"
"l10n_do_61040300","61040300","Depreciation and amortization expense","expense_depreciation","False","Gastos por Depreciación de Mobiliario y Equipos"
"l10n_do_61040400","61040400","Machinery Depreciation Expense","expense_depreciation","False","Gastos por Depreciación de Maquinaria"
"l10n_do_61040500","61040500","Tools Depreciation Expense","expense_depreciation","False","Gastos por Depreciación de Herramientas"
"l10n_do_61040600","61040600","Depreciation and amortization expense","expense_depreciation","False","Gastos por Depreciación de Mobiliario y Equipos"
"l10n_do_61050100","61050100","Building Repair Expenses","expense","False","Gastos por Reparación de Edificios"
"l10n_do_61050200","61050200","Transportation Equipment Repair Expenses","expense","False","Gastos por Reparación de Equipo de Transporte"
"l10n_do_61050300","61050300","Furniture and Equipment Repair Expenses","expense","False","Gastos por Reparación de Mobiliario y Equipos"
"l10n_do_61050400","61050400","Machinery Repair Expenses","expense","False","Gastos por Reparación de Maquinaria"
"l10n_do_61050500","61050500","Tool Repair Expenses","expense","False","Gastos por Reparación de Herramientas"
"l10n_do_61050600","61050600","Expenses for Repair of Facilities","expense","False","Gastos por Reparación de Instalaciones"
"l10n_do_61060100","61060100","Public Relations","expense","False","Relaciones Públicas"
"l10n_do_61060200","61060200","Advertising","expense","False","Publicidad"
"l10n_do_61060300","61060300","Travel and Representation Expenses","expense","False","Gastos de Viajes y Representación"
"l10n_do_61060400","61060400","Donations","expense","False","Donaciones"
"l10n_do_61060500","61060500","Donations to ProIndustria (Law 392-07)","expense","False","Donaciones a ProIndustria (Ley 392-07)"
"l10n_do_61060600","61060600","Promotions","expense","False","Promociones"
"l10n_do_61060700","61060700","Restaurant Expenses","expense","False","Gastos en Restaurantes"
"l10n_do_61070100","61070100","Financial Loan Expenses","expense","False",""
"l10n_do_61070200","61070200","Withholding for Checks or Wire Transfers (0.15%)","expense","False","Retención por Cheques o Transferencias Electrónicas (0.15%)"
"l10n_do_61070300","61070300","Bank Interest","expense","False","Intereses Bancarios"
"l10n_do_61070400","61070400","Banking Commissions","expense","False","Comisiones Bancarias"
"l10n_do_61070500","61070500","Credit Card Fees","expense","False","Comisiones de Tarjeta de Crédito"
"l10n_do_61070600","61070600","Bank Charges","expense","False","Cargos Bancarios"
"l10n_do_61070700","61070700","Bank Loan Insurance","expense","False","Seguro sobre Préstamos Bancarios"
"l10n_do_61070800","61070800","Foreign exchange losses","expense","False","Pérdidas por Diferencia Cambiaria"
"l10n_do_61070900","61070900","Other Financial Expenses","expense","False","Otro Gastos Financieros"
"l10n_do_61080100","61080100","Cash Shortage Losses","expense","False","Pérdidas por Faltante en Caja"
"l10n_do_61080200","61080200","Losses on Sales of Fixed Assets","expense","False","Pérdidas por Ventas de Activos Fijos"
"l10n_do_61080300","61080300","Losses on uncollectible accounts","expense","False","Pérdidas por Cuentas Incobrables"
"l10n_do_61080500","61080500","Asset Taxes","expense","False","Impuestos a los Activos"
"l10n_do_61080600","61080600","Penalty Expenses/DGII Charges","expense","False","Gastos por Penalidades/Recargos de DGII"
"l10n_do_61080700","61080700","TSS Penalty Charges/Penalty Expenses","expense","False","Gastos por Penalidades/Recargos de TSS"
"l10n_do_61080800","61080800","Expenses without Vouchers","expense","False","Gastos sin Comprobantes"
"l10n_do_61080900","61080900","Non-Deductible Expenses","expense","False","Gastos No Deducibles"
"l10n_do_61081000","61081000","Cash Discount Loss","expense","False","Pérdida por descuento"
"l10n_do_71010100","71010100","Profit or Loss","equity_unaffected","False","Ganancias o Pérdidas"
"l10n_do_71020100","71020100","Tax Expenses","expense","False","Gastos por Impuestos"
"l10n_do_71020200","71020200","Income tax expense","expense","False","Gastos por ISR"

```

## File: data\template\account.fiscal.position-do.csv

```csv
"id","name","tax_ids/tax_src_id","tax_ids/tax_dest_id","account_ids/account_dest_id","account_ids/account_src_id","name@es"
"position_person","P. Physical Services","tax_18_purch","tax_group_person_services","","","P. Física de Servicios"
"","","tax_18_purch_serv","tax_group_person_services","","",""
"","","","","l10n_do_61030111","l10n_do_61021500",""
"position_service_moral","P. Legal Services","tax_18_purch","tax_group_moral_services","","","P. Jurídica de Servicios"
"position_security_moral","P. Legal Surveillance","tax_18_purch","ret_100_tax_security","","","P. Jurídica de Vigilancia"
"position_nonformal","Informal Supplier of Goods","tax_18_purch","tax_group_nonformal","","","Proveedor Informal de Bienes"
"","","tax_18_purch_incl","tax_group_nonformal","","",""
"position_exterior","Outside Services","tax_18_purch_serv","ret_27_income_remittance","","","Servicios del Exterior"
"","","tax_18_purch","ret_27_income_remittance","","",""
"","","","","l10n_do_21010300","l10n_do_21010200",""
"position_gov","Governmental","tax_18_sale","ret_5_income_gov","","","Gubernamental"
"","","tax_18_sale_incl","ret_5_income_gov","","",""
"position_nonprofit","Non-Profit Services","tax_18_purch","ret_100_tax_nonprofit","","","No Lucrativa de Servicios"
"","","tax_18_purch_incl","ret_100_tax_nonprofit","","",""
"position_especial","Special Regimes","tax_18_sale","tax_0_sale","","","Regímenes Especiales"
"","","tax_18_sale_incl","tax_0_sale","","",""
"","","tax_group_restaurant_sale","tax_tip_sale","","",""
"position_restaurant","Restaurants","tax_18_purch","tax_group_restaurant_purch","","","Restaurantes"
"","","tax_18_purch_incl","tax_group_restaurant_purch","","",""
"","","tax_18_sale","tax_group_restaurant_sale","","",""
"","","tax_18_sale_incl","tax_group_restaurant_sale","","",""
"position_restaurant_takeout","To Take Away","tax_group_restaurant_sale","tax_18_sale","","","Para Llevar"

```

## File: data\template\account.group-do.csv

```csv
"id","code_prefix_start","code_prefix_end","name","name@es"
"account_group_1","1","","Assets","Activos"
"account_group_11","11","","Current Assets","Activos Corrientes"
"account_group_1101","1101","","Cash and Cash Equivalents","Efectivo y Equivalentes de Efectivo"
"account_group_110101","110101","","Cash","Caja"
"account_group_110102","110102","","Banks","Bancos"
"account_group_110103","110103","","Temporary Investments Less than 90 days","Inversiones Temporales Plazo Menor 90 días"
"account_group_1102","1102","","Short-Term Investments in Securities","Inversiones en Valores a Corto Plazo"
"account_group_1103","1103","","Accounts and notes receivable","Cuentas y Documentos por Cobrar"
"account_group_110301","110301","","Notes Receivable","Documentos por Cobrar"
"account_group_110302","110302","","Accounts Receivable","Cuentas por Cobrar"
"account_group_1104","1104","","Allowance for doubtful accounts","Provisión para Cuentas Incobrables"
"account_group_1105","1105","","Inventories","Inventarios"
"account_group_1106","1106","","Accumulated Impairment of Inventory Value","Deterioro Acumulado de Valor de Inventarios"
"account_group_1107","1107","","Inventory obsolescence estimate","Estimación por Obsolencia de Inventario"
"account_group_1108","1108","","Advanced Taxes","Impuestos Adelantados"
"account_group_110801","110801","","ITBIS Advance on Purchases","ITBIS Adelantado en Compras"
"account_group_110803","110803","","Other Taxes and Balances","Otros Impuestos y Saldos"
"account_group_1109","1109","","Temporary Investments","Inversiones Temporales"
"account_group_1110","1110","","Advance Payments","Pagos Anticipados"
"account_group_12","12","","Fixed Assets","Activo Fijos"
"account_group_1201","1201","","Property, Plant and Equipment","Propiedades, Planta y Equipo"
"account_group_120101","120101","","Real Estate (CAT 1)","Bienes Inmuebles (CAT 1)"
"account_group_120102","120102","","Personal Property (CAT 2)","Bienes Muebles (CAT 2)"
"account_group_120103","120103","","Personal Property (CAT 3)","Bienes Muebles (CAT 3)"
"account_group_1202","1202","","Accumulated Depreciation of Property, Plant and Equipment","Depreciación Acumulada de Propiedades, Planta y Equipo"
"account_group_1203","1203","","Accumulated Impairment of Property, Plant and Equipment","Deterioro de Valor Acumulado de Propiedades, Planta y Equipo"
"account_group_1204","1204","","Intangible Assets","Activos Intangibles"
"account_group_1205","1205","","Leased Assets","Bienes en Arrendamiento"
"account_group_1206","1206","","Accumulated Depreciation of Leased Assets","Depreciación Acumulada de Bienes en Arrendamiento"
"account_group_1207","1207","","Accumulated Impairment Loss on Leased Assets","Deterioro de Valor Acumulado de Bienes en Arrendamiento"
"account_group_1208","1208","","Permanent Investments","Inversiones Permanentes"
"account_group_1209","1209","","Deferred Assets","Activos Diferidos"
"account_group_13","13","","Construction Costs","Costos de Construcción"
"account_group_1301","1301","","Construction in Progress","Construcción en Proceso"
"account_group_130101","130101","","In-Process Construction Costs","Costos de Costrucción en Proceso"
"account_group_2","2","","Liabilities","Pasivos"
"account_group_21","21","","Current Liabilities","Pasivo Corriente"
"account_group_2101","2101","","Short-term accounts payable and notes payable","Cuentas y Documentos por Pagar a Corto Plazo"
"account_group_2102","2102","","Short-Term Benefits Payable","Beneficios por Pagar a Corto Plazo"
"account_group_2103","2103","","Taxes and Withholdings","Impuestos y Retenciones"
"account_group_210301","210301","","ITBIS Payable","ITBIS por Pagar"
"account_group_210302","210302","","ITBIS Withheld","ITBIS Retenido"
"account_group_210303","210303","","ISR withheld","ISR Retenido"
"account_group_210304","210304","","Payroll withholdings","Retenciones en Nómina"
"account_group_210305","210305","","Other Taxes or Withholdings","Otros Impuestos o Retenciones"
"account_group_2104","2104","","Short-Term Provisions","Provisiones a Corto Plazo"
"account_group_2105","2105","","Credit Cards","Tarjetas de Crédito"
"account_group_22","22","","Non-Current Liabilities","Pasivo No Corriente"
"account_group_2201","2201","","Long-term accounts payable and notes payable","Cuentas y Documentos por Pagar a Largo Plazo"
"account_group_2202","2202","","Provision for Labor Obligations","Provisión para Obligaciones Laborales"
"account_group_2203","2203","","Customer Advances and Unidentified Payments","Anticipos de Clientes y Pagos no Identificados"
"account_group_2204","2204","","Long-Term Provisions","Provisiones a Largo Plazo"
"account_group_3","3","","Capital",""
"account_group_31","31","","Stockholders' equity","Capital Contable"
"account_group_3101","3101","","Capital Stock","Capital Social"
"account_group_3102","3102","","Asset revaluation surplus","Superávit por Revaluación de Activos"
"account_group_32","32","","Restricted Utilities","Utilidades Restringidas"
"account_group_3201","3201","","Legal Reserve","Reserva Legal"
"account_group_33","33","","Results","Resultados"
"account_group_3301","3301","","Accumulated Results","Resultados Acumulados"
"account_group_3302","3302","","Results for the year","Resultados del Ejercicio"
"account_group_3304","3304","","Other Equity Reserves","Otras Reservas de Patrimonio"
"account_group_4","4","","Revenues and Earnings","Ingresos y Ganancias"
"account_group_41","41","","Operating Revenues","Ingresos por Operaciones"
"account_group_4101","4101","","Sales of Goods","Ventas de Bienes"
"account_group_4102","4102","","Sales of Services","Ventas de Servicios"
"account_group_4103","4103","","Returns","Devoluciones"
"account_group_4104","4104","","Discounts","Descuentos"
"account_group_42","42","","Non-Operating Income","Ingresos No Operacionales"
"account_group_4201","4201","","Interest Earned","Intereses Ganados"
"account_group_4202","4202","","Sales of Assets","Ventas de Activos"
"account_group_4203","4203","","Dividends Earned","Dividendos Ganados"
"account_group_4204","4204","","Extraordinary Income","Ingresos Extraordinarios"
"account_group_5","5","","Costs, Expenses and Losses","Costos, Gastos y Pérdidas"
"account_group_51","51","","Operating Costs","Costos de Operación"
"account_group_5101","5101","","Cost of sales","Costos de Ventas"
"account_group_5102","5102","","Production Costs","Costos de Producción"
"account_group_6","6","","Expenses and Losses","Gastos y Pérdidas"
"account_group_61","61","","Operating Expenses","Gastos de Operación"
"account_group_6101","6101","","Personnel Expenses","Gastos de Personal"
"account_group_610101","610101","","Social Security Contributions","Aportes a la Seguridad Social"
"account_group_610102","610102","","Other Employer's Liabilities","Otras Cargas Patronales"
"account_group_6102","6102","","Administrative Expenses","Gastos de Administración"
"account_group_6103","6103","","Expenses for Labor, Supplies and Services","Gastos por Trabajo, Suministros y Servicios"
"account_group_610301","610301","","Expenses Fees for Professional Services (Individual)","Gastos Honorarios por Servicios Profesionales (P. Física)"
"account_group_610302","610302","","Fees for professional services (Legal)","Gastos Honorarios por Servicios Profesionales (P. Jurídica)"
"account_group_610303","610303","","Fees for professional services (Legal)","Gastos Honorarios por Servicios Profesionales (P. Jurídica)"
"account_group_6104","6104","","Depreciation expense","Gastos por Depreciación"
"account_group_6105","6105","","Expenses for repairs","Gastos por Reparaciones"
"account_group_6106","6106","","Representation Expenses","Gastos de Representación"
"account_group_6107","6107","","Financial Expenses","Gastos Financieros"
"account_group_6108","6108","","Extraordinary Expenses","Gastos Extraordinarios"
"account_group_7","7","","Income Statement Clearing Accounts","Cuentas Liquidadoras de Resultados"
"account_group_71","71","","Settlement Account","Cuenta Liquidadora"
"account_group_7101","7101","","Profit and loss","Pérdidas y Ganancias"

```

## File: data\template\account.tax-do.csv

```csv
"id","sequence","name","description","invoice_label","amount","amount_type","type_tax_use","price_include","tax_group_id","children_tax_ids","active","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","description@es"
"tax_0_sale","4","ITBIS Exempt","ITBIS Exempt Sales","ITBIS Exempt Sales","0.0","percent","sale","False","tax_group_itbis","","","base","invoice","+base.untaxed.sales.services","","Exento ITBIS Ventas"
"","","","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","","","base","refund","-base.untaxed.sales.services","",""
"","","","","","","","","","","","","tax","refund","","",""
"tax_0_purch","10","ITBIS Exempt","ITBIS Exempt Purchases","ITBIS Exempt Purchases","0.0","percent","purchase","False","tax_group_itbis","","","base","invoice","","","Exento ITBIS Compras"
"","","","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","",""
"tax_18_sale","1","18% ITBIS","18% ITBIS Sales","18% ITBIS Sales","18.0","percent","sale","False","tax_group_itbis","","","base","invoice","+base.18%","","18% ITBIS Ventas"
"","","","","","","","","","","","","tax","invoice","+tax.18%","l10n_do_21030102",""
"","","","","","","","","","","","","base","refund","-base.18%","",""
"","","","","","","","","","","","","tax","refund","-tax.18%","l10n_do_21030102",""
"tax_18_sale_incl","2","18% ITBIS Incl.","18% ITBIS Incl. Sales","18% ITBIS Incl. Sales","18.0","percent","sale","True","tax_group_itbis","","","base","invoice","+base.18%","","18% ITBIS Incl. Ventas"
"","","","","","","","","","","","","tax","invoice","+tax.18%","l10n_do_21030102",""
"","","","","","","","","","","","","base","refund","-base.18%","",""
"","","","","","","","","","","","","tax","refund","-tax.18%","l10n_do_21030102",""
"tax_tip_sale","3","10% Propina","10% Propina Legal","10% Propina Legal","10.0","percent","sale","False","tax_group_tip","","","base","invoice","","",""
"","","","","","","","","","","","","tax","invoice","","l10n_do_21030503",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_21030503",""
"tax_18_purch","11","18% ITBIS","18% ITBIS Purchases","18% ITBIS Purchases","18.0","percent","purchase","False","tax_group_itbis","","","base","invoice","","","18% ITBIS Compras"
"","","","","","","","","","","","","tax","invoice","+tax.16%","l10n_do_11080101",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","-tax.16%","l10n_do_11080101",""
"tax_18_purch_incl","12","18% ITBIS Incl.","18% ITBIS Incl. Purchases","18% ITBIS Incl. Purchases","18.0","percent","purchase","True","tax_group_itbis","","","base","invoice","","","18% ITBIS Incl. Compras"
"","","","","","","","","","","","","tax","invoice","+tax.16%","l10n_do_11080101",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","-tax.16%","l10n_do_11080101",""
"tax_16_purch","13","16% ITBIS","16% ITBIS Purchases","16% ITBIS Purchases","16.0","percent","purchase","False","tax_group_itbis","","","base","invoice","","","16% ITBIS Compras"
"","","","","","","","","","","","","tax","invoice","","l10n_do_11080101",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_11080101",""
"tax_16_purch_incl","14","16% ITBIS Incl.","16% ITBIS Incl. Purchases","16% ITBIS Incl. Purchases","16.0","percent","purchase","True","tax_group_itbis","","","base","invoice","","","16% ITBIS Incl. Compras"
"","","","","","","","","","","","","tax","invoice","","l10n_do_11080101",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_11080101",""
"tax_9_purch","15","9% ITBIS","9% ITBIS Purchase (L690-16)","9% ITBIS Purchase (L690-16)","9.0","percent","purchase","False","tax_group_itbis","","","base","invoice","","","9% ITBIS Compras"
"","","","","","","","","","","","","tax","invoice","","l10n_do_11080101",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_11080101",""
"tax_9_purch_incl","16","9% ITBIS Incl.","9% ITBIS Purchase Incl. (L690-16)","9% ITBIS Purchase Incl. (L690-16)","9.0","percent","purchase","True","tax_group_itbis","","","base","invoice","","","9% ITBIS Incl. Compras"
"","","","","","","","","","","","","tax","invoice","","l10n_do_11080101",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_11080101",""
"tax_8_purch","17","8% ITBIS","8% ITBIS Purchase (L690-16)","8% ITBIS Purchase (L690-16)","8.0","percent","purchase","False","tax_group_itbis","","","base","invoice","","","8% ITBIS Compras"
"","","","","","","","","","","","","tax","invoice","","l10n_do_11080101",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_11080101",""
"tax_8_purch_incl","18","8% ITBIS Incl.","8% ITBIS Purchase Incl. (L690-16)","8% ITBIS Purchase Incl. (L690-16)","8.0","percent","purchase","True","tax_group_itbis","","","base","invoice","","","8% ITBIS Incl. Compras"
"","","","","","","","","","","","","tax","invoice","","l10n_do_11080101",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_11080101",""
"tax_tip_purch","19","10% Propina","10% Propina","10% Propina","10.0","percent","purchase","False","tax_group_tip","","","base","invoice","","",""
"","","","","","","","","","","","","tax","invoice","","l10n_do_61080900",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_61080900",""
"tax_18_purch_serv","20","18% ITBIS Serv.","18% ITBIS Purchases - Services","18% ITBIS Purchases - Services","18.0","percent","purchase","False","tax_group_itbis","","","base","invoice","","","18% ITBIS Compras - Servicios"
"","","","","","","","","","","","","tax","invoice","+tax.16%","l10n_do_11080102",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","-tax.16%","l10n_do_11080102",""
"tax_18_purch_serv_incl","20","18% ITBIS Incl. Serv.","18% ITBIS Incl. Purchases - Services","18% ITBIS Incl. Purchases - Services","18.0","percent","purchase","True","tax_group_itbis","","","base","invoice","","","18% ITBIS Incl. Compras - Servicios"
"","","","","","","","","","","","","tax","invoice","+tax.16%","l10n_do_11080102",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","-tax.16%","l10n_do_11080102",""
"tax_18_importation","20","18% ITBIS Imp.","18% ITBIS - Imports","18% ITBIS - Imports","18.0","percent","purchase","False","tax_group_itbis","","","base","invoice","","","18% ITBIS - Importaciones"
"","","","","","","","","","","","","tax","invoice","+tax.paid.imports","l10n_do_11080103",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","-tax.paid.imports","l10n_do_11080103",""
"tax_18_of_10","20","18% of 10%","18% ITBIS on 10% of the Total Amount","18% ITBIS on 10% of the Total Amount","1.8","percent","sale","False","tax_group_itbis","","","base","invoice","","","18% ITBIS sobre el 10% del Monto Total"
"","","","","","","","","","","","","tax","invoice","","l10n_do_21030102",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_21030102",""
"tax_0015_bank","30","0.15% Trans.","0.15% Bank Transfer","0.15% Bank Transfer","0.0015","percent","none","True","tax_group_other_tax","","","base","invoice","","","0.15% Transferencia Bancaria"
"","","","","","","","","","","","","tax","invoice","","l10n_do_61070200",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_61070200",""
"tax_10_telco","30","10% ISC","10% ISC Telecommunications","10% ISC Telecommunications","10.0","percent","purchase","False","tax_group_isc","","","base","invoice","","","10% ISC Telecomunicaciones"
"","","","","","","","","","","","","tax","invoice","","l10n_do_61020200",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_61020200",""
"tax_2_telco","30","2% CDT","2% CDT Telecommunications","2% CDT Telecommunications","2.0","percent","purchase","False","tax_group_other_tax","","","base","invoice","","","2% CDT Telecomunicaciones"
"","","","","","","","","","","","","tax","invoice","","l10n_do_61020200",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_61020200",""
"tax_group_telco","30","Telecom","Telecommunications Taxes","Telecommunications Taxes","18.0","group","purchase","","tax_group_other_tax","tax_18_purch,tax_10_telco,tax_2_telco","","","","","","Impuestos a las Telecomunicaciones"
"ret_100_tax_security","40","-100% ITBIS (N07-09)","Withholding 100% ITBIS Security Services (N07-09)","Withholding 100% ITBIS Security Services (N07-09)","-18.0","percent","purchase","False","tax_group_itbis","","","base","invoice","","","Retención 100% ITBIS Servicios de Seguridad (N07-09)"
"","","","","","","","","","","","","tax","invoice","","l10n_do_21030201",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_21030201",""
"ret_100_tax_nonprofit","41","-100% ITBIS (N01-11)","Withholding 100% ITBIS Non-profit Services (N01-11)","Withholding 100% ITBIS Non-profit Services (N01-11)","-18.0","percent","purchase","False","tax_group_itbis","","","base","invoice","+A.39","","Retención 100% ITBIS Servicios No Lucrativas (N01-11)"
"","","","","","","","","","","","","tax","invoice","-A.50","l10n_do_21030203",""
"","","","","","","","","","","","","base","refund","-A.39","",""
"","","","","","","","","","","","","tax","refund","+A.50","l10n_do_21030203",""
"ret_100_tax_person","42","-100% ITBIS (R293-11)","Withholding 100% ITBIS Services to Individuals (R293-11)","Withholding 100% ITBIS Services to Individuals (R293-11)","-18.0","percent","purchase","False","tax_group_itbis","","","base","invoice","+A.39","","Retención 100% ITBIS Servicios a Físicas (R293-11)"
"","","","","","","","","","","","","tax","invoice","-A.50","l10n_do_21030202",""
"","","","","","","","","","","","","base","refund","-A.39","",""
"","","","","","","","","","","","","tax","refund","+A.50","l10n_do_21030202",""
"ret_30_tax_moral","43","-30% ITBIS Leg. (N02-05)","Withholding 30% ITBIS Services to Legal Entities (N02-05)","Withholding 30% ITBIS Services to Legal Entities (N02-05)","-5.4","percent","purchase","False","tax_group_itbis","","","base","invoice","","","Retención 30% ITBIS Servicios a Jurídicas (N02-05)"
"","","","","","","","","","","","","tax","invoice","","l10n_do_21030201",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_21030201",""
"ret_30_tax_freelance","43","-30% ITBIS Prof. (N02-05)","Withholding 30% ITBIS Professional Services (N02-05)","Withholding 30% ITBIS Professional Services (N02-05)","-5.4","percent","none","False","tax_group_itbis","","False","base","invoice","","","Retención 30% ITBIS Servicios Profesionales (N02-05)"
"","","","","","","","","","","","","tax","invoice","","l10n_do_21030201",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_21030201",""
"ret_75_tax_nonformal","44","-75% ITBIS (N08-10)","Withholding 75% ITBIS Goods to Informal Workers (N08-10)","Withholding 75% ITBIS Goods to Informal Workers (N08-10)","-13.5","percent","purchase","False","tax_group_itbis","","","base","invoice","","","Retención 75% ITBIS Bienes a Informales (N08-10)"
"","","","","","","","","","","","","tax","invoice","","l10n_do_21030205",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_21030205",""
"ret_10_income_person","40","-10% ISR Fee","Withholding 10% Income Tax Fees to Individuals","Withholding 10% Income Tax Fees to Individuals","-10.0","percent","purchase","False","tax_group_isr","","","base","invoice","","","Retención 10% ISR Honorarios a Físicas"
"","","","","","","","","","","","","tax","invoice","","l10n_do_21030301",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_21030301",""
"ret_10_income_rent","50","-10% ISR Rent.","Withholding 10% Income Tax Rentals to Individuals","Withholding 10% Income Tax Rentals to Individuals","-10.0","percent","purchase","False","tax_group_isr","","","base","invoice","","","Retención 10% ISR Alquileres a Físicas"
"","","","","","","","","","","","","tax","invoice","","l10n_do_21030302",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_21030302",""
"ret_10_income_dividend","51","-10% ISR (L253-12)","Withholding 10% Income Tax on Dividends (L253-12)","Withholding 10% Income Tax on Dividends (L253-12)","-10.0","percent","purchase","False","tax_group_isr","","","base","invoice","","","Retención 10% ISR por Dividendos (L253-12)"
"","","","","","","","","","","","","tax","invoice","","l10n_do_21030303",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_21030303",""
"ret_2_income_person","52","-2% ISR (N07-07)","Withholding 2% ISR to Individuals (N07-07)","Withholding 2% ISR to Individuals (N07-07)","-2.0","percent","purchase","False","tax_group_isr","","","base","invoice","","","Retención 2% ISR a Física (con Materiales)"
"","","","","","","","","","","","","tax","invoice","","l10n_do_21030308",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_21030308",""
"ret_2_income_transfer","53","-2% ISR Mat.","Withholding 2% ISR to Individuals (with Materials)","Withholding 2% ISR to Individuals (with Materials)","-2.0","percent","purchase","False","tax_group_isr","","","base","invoice","","","Retención 2% ISR a Física (con Materiales)"
"","","","","","","","","","","","","tax","invoice","","l10n_do_21030306",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_21030306",""
"ret_27_income_remittance","49","-27% ISR (L253-12)","Withholding 27% Income Tax on Remittances Abroad (L253-12)","Withholding 27% Income Tax on Remittances Abroad (L253-12)","-27.0","percent","purchase","False","tax_group_isr","","","base","invoice","","","Retención 27% ISR por Remesas al Exterior (L253-12)"
"","","","","","","","","","","","","tax","invoice","","l10n_do_21030307",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_21030307",""
"ret_5_income_gov","50","-5% ISR Gov.","Withholding 5% Governmental Income Tax","Withholding 5% Governmental Income Tax","-5.0","percent","sale","False","tax_group_isr","","","base","invoice","","","Retención 5% ISR Gubernamentales"
"","","","","","","","","","","","","tax","invoice","","l10n_do_11080302",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_11080302",""
"tax_group_nonformal","60","75%","Withholding from Informal Suppliers of Goods (75%)","Withholding from Informal Suppliers of Goods (75%)","18.0","group","purchase","","tax_group_ret","tax_18_purch,ret_75_tax_nonformal","","","","","","Retención a Proveedores Informales de Bienes (75%)"
"tax_group_person_construction","61","2% Mat.","Withholding to Individuals for Services with Materials (2%)","Withholding to Individuals for Services with Materials (2%)","18.0","group","purchase","","tax_group_ret","tax_18_purch,ret_100_tax_person,ret_2_income_person","","","","","","Retención a Jurídicas por Servicios Profesionales (30%)"
"tax_group_person_services","58","10% Serv.","Withholding to Individuals for Person Services","Withholding to Individuals for Person Services","18.0","group","purchase","","tax_group_ret","tax_18_purch,ret_100_tax_person,ret_10_income_person","","","","","","Retención a Físicas por Honorarios por Servicios (10%)"
"tax_group_moral_services","58","2% Serv P.","Withholding to Individuals for Moral Services","Withholding to Individuals for Moral Services","18.0","group","purchase","","tax_group_ret","tax_18_purch,ret_30_tax_moral","","","","","","Retención a Jurídicas por Servicios Profesionales (30%)"
"tax_group_restaurant_sale","64","Restaurant","Restaurant Sales","Restaurant Sales","18.0","group","sale","","tax_group_ret","tax_18_sale,tax_tip_sale","","","","","","Ventas del Restaurante"
"tax_group_restaurant_purch","65","Restaurant","Restaurant Purchases","Restaurant Purchases","18.0","group","purchase","","tax_group_ret","tax_18_purch,tax_tip_purch","","","","","","Compras a Restaurantes"
"tax_18_10_total_mount","","18% of 10%","18% ITBIS on 10% of the Total Amount","18% ITBIS on 10% of the Total Amount","1.8","percent","purchase","False","tax_group_itbis","","","base","invoice","","","18% ITBIS sobre el 10% del Monto Total"
"","","","","","","","","","","","","tax","invoice","","l10n_do_11080102",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","","l10n_do_11080102",""
"tax_18_property_cost","","18% Cost Good","18% ITBIS carried at Cost Goods","18% ITBIS carried at Cost Goods","18.0","percent","purchase","False","tax_group_itbis","","","base","invoice","","","18% ITBIS llevado al Costo Bienes"
"","","","","","","","","","","","","tax","invoice","+tax.16%","l10n_do_51010500",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","+tax.16%","l10n_do_51010500",""
"tax_18_serv_cost","","18% Cost Serv.","18% ITBIS carried at Cost Services","18% ITBIS carried at Cost Services","18.0","percent","purchase","False","tax_group_itbis","","","base","invoice","","","18% ITBIS llevado al Costo Servicios"
"","","","","","","","","","","","","tax","invoice","+tax.16%","l10n_do_51010500",""
"","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","tax","refund","+tax.16%","l10n_do_51010500",""

```

## File: data\template\account.tax.group-do.csv

```csv
"id","name","country_id","name@es"
"tax_group_itbis","ITBIS","base.do","ITBIS"
"tax_group_isr","ISR","base.do",""
"tax_group_ret","Withholdings","base.do","Retenciones"
"tax_group_other_tax","Other Taxes","base.do","Otros Impuestos"
"tax_group_isc","ISC","base.do",""
"tax_group_itbis_0","Exempt","base.do","Exento"
"tax_group_tip","Propina","base.do",""
"tax_group_itbis_18","ITBIS 18%","base.do","ITBIS 18%"
"tax_group_itbis_00015","ITBIS 0.0015%","base.do","ITBIS 0.0015%"
"tax_group_isr_retencion_2","ISR -2%","base.do",""
"tax_group_isr_retencion_10","ISR -10%","base.do",""
"tax_group_isr_retencion_27","ISR -27%","base.do",""
"tax_group_itbis_retencion_30","ITBIS -30%","base.do","ITBIS -30%"
"tax_group_itbis_retencion_100","ITBIS -100%","base.do","ITBIS -100%"

```

## File: models\template_do.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('do')
    def _get_do_template_data(self):
        return {
            'code_digits': '8',
            'property_account_receivable_id': 'l10n_do_11030201',
            'property_account_payable_id': 'l10n_do_21010200',
            'property_account_income_categ_id': 'l10n_do_41010100',
            'property_account_expense_categ_id': 'l10n_do_51010100',
            'property_stock_account_input_categ_id': 'l10n_do_21021200',
            'property_stock_account_output_categ_id': 'l10n_do_11050600',
            'property_stock_valuation_account_id': 'l10n_do_11050100',
        }

    @template('do', 'res.company')
    def _get_do_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.do',
                'bank_account_code_prefix': '110102',
                'cash_account_code_prefix': '110101',
                'transfer_account_code_prefix': '11010100',
                'account_default_pos_receivable_account_id': 'l10n_do_11030210',
                'income_currency_exchange_account_id': 'l10n_do_42040100',
                'expense_currency_exchange_account_id': 'l10n_do_61070800',
                'account_journal_early_pay_discount_loss_account_id': 'l10n_do_61081000',
                'account_journal_early_pay_discount_gain_account_id': 'l10n_do_42040400',
                'default_cash_difference_income_account_id': 'l10n_do_42040400',
                'default_cash_difference_expense_account_id': 'l10n_do_61081000',
                'account_sale_tax_id': 'tax_18_sale',
                'account_purchase_tax_id': 'tax_18_purch',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_do

```

