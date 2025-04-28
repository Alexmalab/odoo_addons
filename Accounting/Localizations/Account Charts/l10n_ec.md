# Odoo Module: l10n_ec

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models
from . import demo

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    "name": "Ecuadorian Accounting",
    "version": "3.3",
    "description": """
Functional
----------

This module adds accounting features for Ecuadorian localization, which
represent the minimum requirements to operate a business in Ecuador in compliance
with local regulation bodies such as the ecuadorian tax authority -SRI- and the 
Superintendency of Companies -Super Intendencia de Compañías-

Follow the next configuration steps:
1. Go to your company and configure your country as Ecuador
2. Install the invoicing or accounting module, everything will be handled automatically

Highlights:
* Ecuadorian chart of accounts will be automatically installed, based on example provided by Super Intendencia de Compañías
* List of taxes (including withholds) will also be installed, you can switch off the ones your company doesn't use
* Fiscal position, document types, list of local banks, list of local states, etc, will also be installed

Technical
---------
Master Data:
* Chart of Accounts, based on recomendation by Super Cías
* Ecuadorian Taxes, Tax Tags, and Tax Groups
* Ecuadorian Fiscal Positions
* Document types (there are about 41 purchase documents types in Ecuador)
* Identification types
* Ecuador banks
* Partners: Consumidor Final, SRI, IESS, and also basic VAT validation
    """,
    "author": "OPA CONSULTING & TRESCLOUD",
    "category": "Accounting/Localizations/Account Charts",
    "maintainer": "OPA CONSULTING",
    "website": "https://opa-consulting.com",
    "license": "LGPL-3",
    "depends": [
        "base",
        "base_iban",
        "account",
        "account_debit_note",
        "l10n_latam_invoice_document",
        "l10n_latam_base",
    ],
    "data": [
        # Chart of Accounts
        "data/account_chart_template_data.xml",
        "data/account_group_template_data.xml",
        "data/account.account.template.csv",
        "data/account_chart_template_setup_accounts.xml",
        # Taxes
        "data/account_tax_group_data.xml",
        "data/account_tax_report_data.xml",
        "data/account_tax_template_vat_data.xml",
        "data/account_tax_template_withhold_profit_data.xml",
        "data/account_tax_template_withhold_vat_data.xml",
        "data/account_fiscal_position_template.xml",
        # Partners data
        "data/res.bank.csv",
        "data/l10n_latam_identification_type_data.xml",
        "data/res_partner_data.xml",
        # Other data
        "data/l10n_latam.document.type.csv",
        "data/account_chart_template_configure_data.xml",
        "data/l10n_ec.sri.payment.csv",
        "views/account_tax_view.xml",
        "views/l10n_latam_document_type_view.xml",
        "views/l10n_ec_sri_payment.xml",
        "views/account_journal_view.xml",
        # Security
        "security/ir.model.access.csv",
    ],
    "demo": [
        "demo/demo_company.xml",
    ],
    "installable": True,
    "auto_install": False,
    "application": False,
}

```

## File: data\account.account.template.csv

```csv
id,code,name,reconcile,user_type_id/id,chart_template_id/id
l10n_ec_ifrs_liquidity_transfer,111010301,Transferencias interbancarias,TRUE,account.data_account_type_current_assets,l10n_ec_ifrs
ec11010401,11010401,Valores en custodia,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec110201,110201,Activos financieros a valor razonable con cambios en resultados,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec110202,110202,Activos financieros disponibles para la venta,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec110203,110203,Activos financieros mantenidos hasta su vencimiento,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec110204,110204,Provision por deterioro,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec1102050101,1102050101,Cuentas por cobrar clientes no relacionados locales,TRUE,account.data_account_type_receivable,l10n_ec_ifrs
ec1102050102,1102050102,Cuentas por cobrar clientes no relacionados extranjeros,TRUE,account.data_account_type_receivable,l10n_ec_ifrs
ec11020601,11020601,Documentos y cuentas por cobrar cliente relacionados locales,TRUE,account.data_account_type_receivable,l10n_ec_ifrs
ec11020602,11020602,Documentos y cuentas por cobrar cliente relacionados extranjeros,TRUE,account.data_account_type_receivable,l10n_ec_ifrs
ec11020701,11020701,Anticipo sueldos,TRUE,account.data_account_type_receivable,l10n_ec_ifrs
ec11020702,11020702,Anticipo décimo tercer sueldo,TRUE,account.data_account_type_receivable,l10n_ec_ifrs
ec11020703,11020703,Anticipo décimo cuarto sueldo,TRUE,account.data_account_type_receivable,l10n_ec_ifrs
ec11020704,11020704,Anticipo utilidades,TRUE,account.data_account_type_receivable,l10n_ec_ifrs
ec110208,110208,Otras cuentas por cobrar relacionadas,TRUE,account.data_account_type_receivable,l10n_ec_ifrs
ec110209,110209,Otras cuentas por cobrar,TRUE,account.data_account_type_receivable,l10n_ec_ifrs
ec110210,110210,Provision por cuentas incobrables,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec110301,110301,Inventarios de materia prima,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec110302,110302,Inventarios de productos en proceso,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec110303,110303,Inventarios de suministros o materiales a ser consumidos en el proceso de produccion,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec110304,110304,Inventarios de suministros o materiales a ser consumidos en la prestacion de servicios,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec110305,110305,Inventarios de prod. term. y mercad. en almacen - producido por la compañia,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec110306,110306,Inventarios de prod. term. y mercad. en almacen - comprado a terceros,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec110307,110307,Mercaderias en transito,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec110308,110308,Provision de inventarios por valor neto de realizacion,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec110309,110309,Provision de inventarios por deterioro fisico,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec110401,110401,Seguros pagados por anticipado,TRUE,account.data_account_type_prepayments,l10n_ec_ifrs
ec110402,110402,Arriendos pagados por anticipado,TRUE,account.data_account_type_prepayments,l10n_ec_ifrs
ec110403,110403,Anticipos a proveedores,TRUE,account.data_account_type_prepayments,l10n_ec_ifrs
ec_other_downpayments,11040401,Otros anticipos entregados,TRUE,account.data_account_type_prepayments,l10n_ec_ifrs
ec_purchase_vat,11050101,Iva pagado en compras locales,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec_purchase_vat_assets,11050102,Iva pagado en compras locales de activos fijos,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec_purchase_vat_service_imports,11050103,Iva pagado en importacion de servicios,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec_purchase_vat_goods_imports,11050104,Iva pagado en importacion de bienes,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec_purchase_vat_assets_imports,11050105,Iva pagado en importacion de activos fijos,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec_sale_vat_outstanding_withholds,11050106,Iva pagado en retenciones de la fuente,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec_purchase_vat_zero,11050107,Iva en compras 0%,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec110502,110502,Credito tributario a favor de la empresa(i.r.),FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec110503,110503,Anticipo de impuesto a la renta,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec_sale_profit_withhold,110504,Retenciones en la fuente pagadas,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec1106,1106,Activos no corrientes mantenidos para la venta y operaciones discontinuadas,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec1107,1107,Construcciones en proceso (nic 11 y secc.23 pymes),FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec1108,1108,Otros activos corrientes,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec120101,120101,Terrenos,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec120102,120102,Edificios,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec120103,120103,Contrucciones en curso,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec120104,120104,Instalaciones,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec120105,120105,Muebles y enseres,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec120106,120106,Maquinaria y equipo,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec120107,120107,"Naves, aeronaves, barcazas y similares",FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec120108,120108,Equipo de computacion,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec120109,120109,"Vehiculos, equipos de transporte y equipo caminero movil",FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec120110,120110,"Otros propiedades, planta y equipo",FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec120111,120111,Repuestos y herramientas,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011201,12011201,Depreciacion acumulada edificios,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011202,12011202,Depreciacion acumulada contrucciones en curso,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011203,12011203,Depreciacion acumulada instalaciones,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011204,12011204,Depreciacion acumulada muebles y enseres,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011205,12011205,Depreciacion acumulada maquinaria y equipo,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011206,12011206,"Depreciacion acumulada naves, aeronaves, barcazas y similares",FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011207,12011207,Depreciacion acumulada equipo de computacion,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011208,12011208,"Depreciacion acumulada vehiculos, equipos de transporte y equipo caminero movil",FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011209,12011209,"Depreciacion acumulada otros propiedades, planta y equipo",FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011210,12011210,Depreciacion acumulada repuestos y herramientas,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011301,12011301,Deterioro acumulado edificios,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011302,12011302,Deterioro acumulado contrucciones en curso,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011303,12011303,Deterioro acumulado instalaciones,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011304,12011304,Deterioro acumulado muebles y enseres,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011305,12011305,Deterioro acumulado maquinaria y equipo,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011306,12011306,"Deterioro acumulado  naves, aeronaves, barcazas y similares",FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011307,12011307,Deterioro acumulado  equipo de computacion,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011308,12011308,"Deterioro acumulado  vehiculos, equipos de transporte y equipo caminero movil",FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011309,12011309,"Deterioro acumulado otros propiedades, planta y equipo",FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec12011310,12011310,Deterioro acumulado repuestos y herramientas,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec120201,120201,Terrenos,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec120202,120202,Edificios,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec120203,120203,Depreciacion acumulada de propiedades de inversion,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec120204,120204,Deterioro acumulado de propiedades de inversion,FALSE,account.data_account_type_fixed_assets,l10n_ec_ifrs
ec120301,120301,Animales vivos en crecimiento,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec120302,120302,Animales vivos en produccion,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec120303,120303,Plantas en crecimiento,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec120304,120304,Plantas en produccion,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec120305,120305,(-) depreciacion acumulada de activos biológicos,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec120306,120306,(-) deterioro acumulado de activos biologícos,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec120401,120401,Plusvalias,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec120402,120402,"Marcas, patentes, derechos de llave , cuotas patrimoniales y otros similares",FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec120403,120403,Activos de exploracion y explotacion,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec120404,120404,Amortizacion acumulada de activos intangible,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec120405,120405,Deterioro acumulado de activo intangible,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec120406,120406,Otros intangibles,FALSE,account.data_account_type_current_assets,l10n_ec_ifrs
ec120501,120501,Activos por impuestos diferidos,FALSE,account.data_account_type_non_current_assets,l10n_ec_ifrs
ec120601,120601,Activos financieros mantenidos hasta el vencimiento,FALSE,account.data_account_type_non_current_assets,l10n_ec_ifrs
ec120602,120602,Provision por deterioro de activos financieros mantenidos hasta el vencimiento,FALSE,account.data_account_type_non_current_assets,l10n_ec_ifrs
ec120603,120603,Documentos y cuentas por cobrar,TRUE,account.data_account_type_receivable,l10n_ec_ifrs
ec120604,120604,Provision cuentas incobrables de activos financieros no corrientes,FALSE,account.data_account_type_non_current_assets,l10n_ec_ifrs
ec120701,120701,Inversiones subsidiarias,FALSE,account.data_account_type_non_current_assets,l10n_ec_ifrs
ec120702,120702,Inversiones asociadas,FALSE,account.data_account_type_non_current_assets,l10n_ec_ifrs
ec120703,120703,Inversiones negocios conjuntos,FALSE,account.data_account_type_non_current_assets,l10n_ec_ifrs
ec120704,120704,Otras inversiones,FALSE,account.data_account_type_non_current_assets,l10n_ec_ifrs
ec120705,120705,Provision valuacion de inversiones,FALSE,account.data_account_type_non_current_assets,l10n_ec_ifrs
ec120706,120706,Otros activos no corrientes,FALSE,account.data_account_type_non_current_assets,l10n_ec_ifrs
ec2101,2101,Pasivos financieros a valor razonable con cambios en resultados,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec2102,2102,Pasivos por contratos de arrendamiento financieros,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec210301,210301,Cuentas y documentos por pagar locales,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec210302,210302,Cuentas y documentos por pagar del exterior,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec210401,210401,Obligaciones con instituciones financieras locales,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec210402,210402,Obligaciones con instituciones financieras del exterior,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec2105010101,2105010101,Provision vacaciones,TRUE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec2105010102,2105010102,Provision decimo tercero,TRUE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec2105010103,2105010103,Provision decimo cuarto,TRUE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec2105010104,2105010104,Otras provisiones a favor de los empleados,TRUE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec21050102,21050102,Otras provisiones locales,TRUE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec210502,210502,Provisiones del exterior,TRUE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec2106,2106,Porcion corriente de obligaciones emitidas,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec21070101,21070101,Impuesto a la renta retenido a empleados,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec210702,210702,Impuesto a la renta por pagar del ejercicio,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec21070301,21070301,Prestamos quirografarios,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec21070302,21070302,Prestamos hipotecarios,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec21070303,21070303,Aportacion patronal al iess,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec21070304,21070304,Aportacion personal al iess,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec21070305,21070305,Fondos de reserva retenidos a empleados,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec21070306,21070306,Otras oblicaciones corrientes con el iess,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec21070401,21070401,Sueldos y salarios por pagar,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec21070402,21070402,Otras obligaciones corrientes por beneficios de ley a empleados,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec21070403,21070403,Décimos por pagar,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec210705,210705,Participacion trabajadores por pagar del ejercicio,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec210706,210706,Dividendos por pagar,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec2108,2108,Cuentas por pagar diversas/relacionadas,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec2109,2109,Otros pasivos financieros,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec2110,2110,Anticipos de clientes,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec2111,2111,Pasivos directamente asociados con los activos no corrientes y operaciones discontinuadas,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec211201,211201,Otros beneficios a largo plazo para los empleados,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec211301,211301,Retenciones judiciales,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec_sale_vat,2114010101,Iva cobrado en ventas locales,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec_sale_vat_assets,2114010102,Iva cobrado en ventas de activos fijos,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec_sale_vat_goods_exports,2114010103,Iva cobrado en exportacion de bienes,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec_sale_vat_services_exports,2114010104,Iva cobrado en exportacion de servicios,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec_sale_vat_zero,2114010105,Iva en ventas 0%,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ret_ir_1x100,2114010201,Retenciones de la fuente 1%,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ret_ir_1_75x100,2114010202,Retenciones de la fuente 1.75%,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ret_ir_2x100,2114010203,Retenciones de la fuente 2%,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ret_ir_2_75x100,2114010204,Retenciones de la fuente 2.75%,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ret_ir_5x100,2114010205,Retenciones de la fuente 5%,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ret_ir_8x100,2114010206,Retenciones de la fuente 8%,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ret_ir_10x100,2114010207,Retenciones de la fuente 10%,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ret_ir_15x100,2114010208,Retenciones de la fuente 15%,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ret_ir_22x100,2114010209,Retenciones de la fuente 22%,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ret_ir_others,2114010210,Otras retenciones,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec_vat_withhold_10,2114010301,Retenciones de iva 10%,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec_vat_withhold_20,2114010302,Retenciones de iva 20%,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec_vat_withhold_30,2114010303,Retenciones de iva 30%,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec_vat_withhold_50,2114010304,Retenciones de iva 50%,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec_vat_withhold_70,2114010305,Retenciones de iva 70%,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec_vat_withhold_100,2114010306,Retenciones de iva 100%,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec_vat_withhold_others,2114010307,Otras retenciones,FALSE,account.data_account_type_current_liabilities,l10n_ec_ifrs
ec2201,2201,Pasivos por contratos de arrendamiento financiero,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec220201,220201,Cuentas y documentos por pagar locales,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec220202,220202,Cuentas y documentos por pagar del exterior,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec220301,220301,Obligaciones con instituciones financieras locales,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec220302,220302,Obligaciones con instituciones financieras del exterior,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec220401,220401,Cuentas por pagar diversas/relacionadas locales,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec220402,220402,Cuentas por pagar diversas/relacionadas del exterior,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec2205,2205,Obligaciones emitidas,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec2206,2206,Anticipos de clientes,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec220701,220701,Otros beneficios no corrientes para los empleados,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec2208,2208,Otras provisiones,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec220901,220901,Ingresos diferidos,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec220902,220902,Pasivos por impuestos diferidos,TRUE,account.data_account_type_payable,l10n_ec_ifrs
ec2210,2210,Otros pasivos no corrientes,FALSE,account.data_account_type_non_current_liabilities,l10n_ec_ifrs
ec3101,3101,Capital suscrito o asignado,FALSE,account.data_account_type_equity,l10n_ec_ifrs
ec3102,3102,"Capital suscrito no pagado, acciones en tesoreria",FALSE,account.data_account_type_equity,l10n_ec_ifrs
ec32,32,Aportes de socios o accionistas para futura capitalizacion,FALSE,account.data_account_type_equity,l10n_ec_ifrs
ec33,33,Prima por emision primaria de acciones,FALSE,account.data_account_type_equity,l10n_ec_ifrs
ec3401,3401,Reserva legal,FALSE,account.data_account_type_equity,l10n_ec_ifrs
ec3402,3402,Reservas facultativa y estatutaria,FALSE,account.data_account_type_equity,l10n_ec_ifrs
ec3403,3403,Reserva de capital,FALSE,account.data_account_type_equity,l10n_ec_ifrs
ec3404,3404,Otras reservas,FALSE,account.data_account_type_equity,l10n_ec_ifrs
ec3501,3501,Superavit de activos financieros disponibles para la venta,FALSE,account.data_account_type_equity,l10n_ec_ifrs
ec3502,3502,"Superavit por revaluacion de propiedades, planta y equipo",FALSE,account.data_account_type_equity,l10n_ec_ifrs
ec3503,3503,Superavit por revaluacion de activos intangibles,FALSE,account.data_account_type_equity,l10n_ec_ifrs
ec3504,3504,Otros superavit por revaluacion,FALSE,account.data_account_type_equity,l10n_ec_ifrs
ec3601,3601,Ganancias acumuladas,FALSE,account.data_account_type_equity,l10n_ec_ifrs
ec3602,3602,Perdidas acumuladas,FALSE,account.data_account_type_equity,l10n_ec_ifrs
ec3603,3603,Resultados acumulados provenientes de la adopcion por primera vez de las niif,FALSE,account.data_account_type_equity,l10n_ec_ifrs
ec3701,3701,Resultado neto del periodo,FALSE,account.data_unaffected_earnings,l10n_ec_ifrs
ec410101,410101,Venta de bienes,FALSE,account.data_account_type_revenue,l10n_ec_ifrs
ec410201,410201,Prestacion de servicios,FALSE,account.data_account_type_revenue,l10n_ec_ifrs
ec4103,4103,Contratos de construccion,FALSE,account.data_account_type_revenue,l10n_ec_ifrs
ec4104,4104,Subvenciones del gobierno,FALSE,account.data_account_type_revenue,l10n_ec_ifrs
ec4105,4105,Regalias,FALSE,account.data_account_type_revenue,l10n_ec_ifrs
ec4106,4106,Intereses,FALSE,account.data_account_type_revenue,l10n_ec_ifrs
ec4107,4107,Dividendos,FALSE,account.data_account_type_revenue,l10n_ec_ifrs
ec4108,4108,Ganancia por medición a valor razonable de activos biológicos,FALSE,account.data_account_type_revenue,l10n_ec_ifrs
ec4109,4109,Otros ingresos de actividades ordinarias,FALSE,account.data_account_type_revenue,l10n_ec_ifrs
ec4110,4110,(-) descuento en ventas,FALSE,account.data_account_type_revenue,l10n_ec_ifrs
ec4111,4111,(-) devoluciones en ventas,FALSE,account.data_account_type_revenue,l10n_ec_ifrs
ec42,42,Ganancia bruta,FALSE,account.data_account_type_revenue,l10n_ec_ifrs
ec4301,4301,Dividendos,FALSE,account.data_account_type_revenue,l10n_ec_ifrs
ec4302,4302,Intereses financieros,FALSE,account.data_account_type_revenue,l10n_ec_ifrs
ec4303,4303,Ganancia en inversiones en asociadas / subsidiarias y otras,FALSE,account.data_account_type_revenue,l10n_ec_ifrs
ec4304,4304,Valuacion de instrumentos financieros a valor razonable con cambio en resultados,FALSE,account.data_account_type_revenue,l10n_ec_ifrs
ec430501,430501,Ingresos por diferecias de cambio,FALSE,account.data_account_type_revenue,l10n_ec_ifrs
ec430502,430502,Otras rentas,FALSE,account.data_account_type_revenue,l10n_ec_ifrs
ec510101,510101,Inventario inicial de bienes no producidos por la compañia,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec510102,510102,Compras netas locales de bienes no producidos por la compañia,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec510103,510103,Importaciones de bienes no producidos por la compañia,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec510104,510104,Inventario final de bienes no producidos por la compañia,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec510105,510105,Inventario inicial de materia prima,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec510106,510106,Compras netas locales de materia prima,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec510107,510107,Importaciones de materia prima,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec510108,510108,Inventario final de materia prima,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec510109,510109,Inventario inicial de productos en proceso,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec510110,510110,Inventario final de productos en proceso,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec510111,510111,Inventario inicial productos terminados,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec510112,510112,Inventario final de productos terminados,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51020101,51020101,"Sueldos, salarios y demas remuneraciones",FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51020102,51020102,Horas extra,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51020103,51020103,Bonificaciones por desempeño,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51020201,51020201,Seguro medico ,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51020202,51020202,Capacitacion,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51020203,51020203,Uniformes,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51020204,51020204,Alimentacion,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec5102020501,5102020501,Impuesto a la renta empleados asumido por la empresa,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec5102020502,5102020502,Transporte,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51020301,51020301,Aporte patronal,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51020302,51020302,Fondos de reserva,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51020401,51020401,Decimo tercer sueldo,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51020402,51020402,Decimo cuarto sueldo,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51020403,51020403,Vacaciones,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51020404,51020404,Despido intempestivo ,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51020405,51020405,Desahucio,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51020406,51020406,Otros beneficios e indemnizaciones,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51030101,51030101,"Sueldos, salarios y demas remuneraciones",FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51030102,51030102,Horas extra,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51030103,51030103,Bonificaciones por desempeño,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51030201,51030201,Seguro medico ,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51030202,51030202,Capacitacion,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51030203,51030203,Uniformes,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51030204,51030204,Alimentacion,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec5103020501,5103020501,Impuesto a la renta empleados asumido por la empresa,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec5103020502,5103020502,Transporte,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51030301,51030301,Aporte patronal,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51030302,51030302,Fondos de reserva,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51030401,51030401,Decimo tercer sueldo,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51030402,51030402,Decimo cuarto sueldo,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51030403,51030403,Vacaciones,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51030404,51030404,Despido intempestivo ,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51030405,51030405,Desahucio,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51030406,51030406,Otros beneficios e indemnizaciones,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51040101,51040101,Edificios,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec51040102,51040102,Contrucciones en curso,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec51040103,51040103,Instalaciones,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec51040104,51040104,Muebles y enseres,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec51040105,51040105,Maquinaria y equipo,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec51040106,51040106,"Naves, aeronaves, barcazas y similares",FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec51040107,51040107,Equipo de computacion,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec51040108,51040108,"Vehiculos, equipos de transporte y equipo caminero movil",FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec51040109,51040109,"Otros propiedades, planta y equipo",FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec51040110,51040110,Repuestos y herramientas,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec510402,510402,Deterioro o pérdidas de activos biológicos,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51040301,51040301,Edificios,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51040302,51040302,Contrucciones en curso,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51040303,51040303,Instalaciones,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51040304,51040304,Muebles y enseres,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51040305,51040305,Maquinaria y equipo,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51040306,51040306,"Naves, aeronaves, barcazas y similares",FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51040307,51040307,Equipo de computacion,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51040308,51040308,"Vehiculos, equipos de transporte y equipo caminero movil",FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51040309,51040309,"Otros propiedades, planta y equipo",FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51040310,51040310,Repuestos y herramientas,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec510404,510404,Efecto valor neto de realizacion de inventarios,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec510405,510405,Gasto por garantias en venta de productos o servicios,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec510406,510406,Mantenimiento y reparaciones,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec510407,510407,Suministros materiales y repuestos,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec51040801,51040801,Otros costos de produccion,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec52010101,52010101,"Sueldos, salarios y demas remuneraciones",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52010102,52010102,Horas extra,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52010103,52010103,Bonificaciones por desempeño,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52010201,52010201,Aporte patronal,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52010202,52010202,Fondos de reserva,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52010301,52010301,Decimo tercer sueldo,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52010302,52010302,Decimo cuarto sueldo,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52010303,52010303,Vacaciones,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52010304,52010304,Despido intempestivo ,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52010305,52010305,Desahucio,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52010306,52010306,Otros beneficios e indemnizaciones,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52010401,52010401,Seguro medico ,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52010402,52010402,Capacitacion,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52010403,52010403,Uniformes,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52010404,52010404,Alimentacion,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5201040501,5201040501,Impuesto a la renta empleados asumido por la empresa,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec5201040502,5201040502,Transporte,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec520105,520105,"Honorarios, comisiones y dietas a personas naturales",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520106,520106,Remuneraciones a otros trabajadores autonomos,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520107,520107,Honorarios a extranjeros por servicios ocasionales,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520108,520108,Mantenimiento y reparaciones,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520109,520109,Arrendamiento operativo,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520110,520110,Comisiones,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520111,520111,Promocion y publicidad,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520112,520112,Combustibles,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520113,520113,Lubricantes,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520114,520114,Seguros y reaseguros (primas y cesiones),FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520115,520115,Transporte,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520116,520116,"Gastos de gestion (agasajos a accionistas, trabajadores y clientes)",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520117,520117,Gastos de viaje,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52011801,52011801,Agua potable,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52011802,52011802,Energia electrica,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52011803,52011803,Telefonia fija,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52011804,52011804,Telefonia movil,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520119,520119,Notarios y registradores de la propiedad o mercantiles,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520120,520120,"Impuestos, contribuciones y otros",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5201210101,5201210101,Edificios,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec5201210102,5201210102,Contrucciones en curso,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec5201210103,5201210103,Instalaciones,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec5201210104,5201210104,Muebles y enseres,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec5201210105,5201210105,Maquinaria y equipo,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec5201210106,5201210106,"Naves, aeronaves, barcazas y similares",FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec5201210107,5201210107,Equipo de computacion,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec5201210108,5201210108,"Vehiculos, equipos de transporte y equipo caminero movil",FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec5201210109,5201210109,"Otros propiedades, planta y equipo",FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec5201210110,5201210110,Repuestos y herramientas,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec52012102,52012102,Propiedades de inversion,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec52012201,52012201,Intangibles,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012202,52012202,Otros activos,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5201230101,5201230101,Edificios,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5201230102,5201230102,Contrucciones en curso,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5201230103,5201230103,Instalaciones,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5201230104,5201230104,Muebles y enseres,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5201230105,5201230105,Maquinaria y equipo,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5201230106,5201230106,"Naves, aeronaves, barcazas y similares",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5201230107,5201230107,Equipo de computacion,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5201230108,5201230108,"Vehiculos, equipos de transporte y equipo caminero movil",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5201230109,5201230109,"Otros propiedades, planta y equipo",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5201230110,5201230110,Repuestos y herramientas,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012302,52012302,Instrumentos financieros,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012303,52012303,Intangibles,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012304,52012304,Cuentas por cobrar,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012305,52012305,Otros activos,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520124,520124,Gastos por cantidades anormales de utilización en el proceso de producción,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520125,520125,Gasto por reestructuracion,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520126,520126,Valor neto de realizacion de inventarios,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012701,52012701,Servicio de imprenta,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012702,52012702,Suministros de oficina,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012703,52012703,Suministros y articulos de aseo y limpieza,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012704,52012704,Alícuotas y condominio,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012705,52012705,"Guias, couriers, correos",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012706,52012706,"Peajes, parqueaderos y otros",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012707,52012707,"Capacitación, cursos y seminarios",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012708,52012708,Alimentación y refrigerios,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012709,52012709,Seguridad y vigilancia,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012710,52012710,Servicios de limpieza,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012711,52012711,Otros impuestos,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012712,52012712,Mantenimiento equipo computo,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012713,52012713,Gastos legales,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012714,52012714,Servicios contratados a terceros,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52012715,52012715,Otros gastos,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52020101,52020101,"Sueldos, salarios y demas remuneraciones",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52020102,52020102,Horas extra,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52020103,52020103,Bonificaciones por desempeño,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52020201,52020201,Aporte patronal,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52020202,52020202,Fondos de reserva,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52020301,52020301,Decimo tercer sueldo,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52020302,52020302,Decimo cuarto sueldo,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52020303,52020303,Vacaciones,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52020304,52020304,Despido intempestivo ,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52020305,52020305,Desahucio,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52020306,52020306,Otros beneficios e indemnizaciones,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52020401,52020401,Seguro medico ,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52020402,52020402,Capacitacion,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52020403,52020403,Uniformes,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52020404,52020404,Alimentacion,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52020405,52020405,Otros gastos de planes de beneficios a empleados,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202040501,5202040501,Impuesto a la renta empleados asumido por la empresa,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec5202040502,5202040502,Transporte,FALSE,account.data_account_type_direct_costs,l10n_ec_ifrs
ec520205,520205,"Honorarios, comisiones y dietas a personas naturales",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520206,520206,Remuneraciones a otros trabajadores autonomos,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520207,520207,Honorarios a extranjeros por servicios ocasionales,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520208,520208,Mantenimiento y reparaciones,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520209,520209,Arrendamiento operativo,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520210,520210,Comisiones,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520211,520211,Promocion y publicidad,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520212,520212,Combustibles,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520213,520213,Lubricantes,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520214,520214,Seguros y reaseguros (primas y cesiones),FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520215,520215,Transporte,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520216,520216,"Gastos de gestion (agasajos a accionistas, trabajadores y clientes)",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520217,520217,Gastos de viaje,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52021801,52021801,Agua potable,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52021802,52021802,Energia electrica,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52021803,52021803,Telefonia fija,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52021804,52021804,Telefonia movil,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520219,520219,Notarios y registradores de la propiedad o mercantiles,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022001,52022001,Patente municipal,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022002,52022002,1.5x mil activos totales,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022003,52022003,Contribuciones super cias.,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022004,52022004,Camara de comercio,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022005,52022005,Otros impuestos,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202210101,5202210101,Edificios,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec5202210102,5202210102,Contrucciones en curso,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec5202210103,5202210103,Instalaciones,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec5202210104,5202210104,Muebles y enseres,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec5202210105,5202210105,Maquinaria y equipo,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec5202210106,5202210106,"Naves, aeronaves, barcazas y similares",FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec5202210107,5202210107,Equipo de computacion,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec5202210108,5202210108,"Vehiculos, equipos de transporte y equipo caminero movil",FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec5202210109,5202210109,"Otros propiedades, planta y equipo",FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec5202210110,5202210110,Repuestos y herramientas,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec52022102,52022102,Propiedades de inversion,FALSE,account.data_account_type_depreciation,l10n_ec_ifrs
ec52022201,52022201,Intangibles,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022202,52022202,Otros activos,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202230101,5202230101,Edificios,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202230102,5202230102,Contrucciones en curso,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202230103,5202230103,Instalaciones,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202230104,5202230104,Muebles y enseres,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202230105,5202230105,Maquinaria y equipo,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202230106,5202230106,"Naves, aeronaves, barcazas y similares",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202230107,5202230107,Equipo de computacion,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202230108,5202230108,"Vehiculos, equipos de transporte y equipo caminero movil",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202230109,5202230109,"Otros propiedades, planta y equipo",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202230110,5202230110,Repuestos y herramientas,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022302,52022302,Inventarios,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022303,52022303,Instrumentos financieros,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022304,52022304,Intangibles,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022305,52022305,Cuentas por cobrar,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022306,52022306,Otros activos,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520224,520224,Gastos por cantidades anormales de utilización en el proceso de producción,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520225,520225,Gasto por reestructuracion,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520226,520226,Valor neto de realizacion de inventarios,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520227,520227,Gasto impuesto a la renta (activos y pasivos diferidos),FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022801,52022801,Suministros de oficina,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022802,52022802,Suministros y artículos de aseo y limpieza,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022803,52022803,Alícuotas y condominio,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022804,52022804,"Guias, couriers, correos",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022805,52022805,"Peajes, parqueaderos y otros",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022806,52022806,"Capacitación, cursos y seminarios",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022807,52022807,Alimentación y refrigerios,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022808,52022808,Seguridad y vigilancia,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022809,52022809,Servicios de limpieza,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022810,52022810,Otros impuestos,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022811,52022811,Mantenimiento equipo computo,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022812,52022812,Material de osteosintesis,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022813,52022813,Gastos de construcción,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022814,52022814,Gastos legales,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022815,52022815,Servicios contratados a terceros,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022816,52022816,Otros gastos,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52022817,52022817,Recolección residuos,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202281801,5202281801,Gnd suministros,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202281802,5202281802,Gnd gastos de gestion y agasajos a clientes y empleados,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202281803,5202281803,Gnd retenciones asumidas,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202281804,5202281804,Gnd soporte tecnico,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202281805,5202281805,Gnd movilización no deducible,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202281806,5202281806,"Gnd impuestos, intereses y multas",FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202281807,5202281807,Gnd salidas de divisas,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec5202281808,5202281808,Gnd otros gastos no deducibles,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520301,520301,Intereses,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520302,520302,Comisiones,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520303,520303,Gastos de financiamiento de activos,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520304,520304,Diferencia en cambio,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520305,520305,Otros gastos financieros,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec520401,520401,Perdida en inversiones en asociadas / subsidiarias y otras,FALSE,account.data_account_type_expenses,l10n_ec_ifrs
ec52040201,52040201,Otros gastos,FALSE,account.data_account_type_expenses,l10n_ec_ifrs

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <function name="try_loading" model="account.chart.template">
            <value eval="[ref('l10n_ec.l10n_ec_ifrs')]"/>
        </function>

    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="l10n_ec_ifrs" model="account.chart.template">
            <field name="name">Ecuador IFRS (Conforme Superintendencia de Compañías)</field>
            <field name="currency_id" ref="base.USD"/>
            <field name="code_digits">4</field>
            <field name="cash_account_code_prefix">1101010</field>
            <field name="bank_account_code_prefix">1101020</field>
            <field name="transfer_account_code_prefix">1101030</field>
            <field name="country_id" ref="base.ec"></field>
        </record>
    </data>
</odoo>

```

## File: data\account_chart_template_setup_accounts.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="l10n_ec_ifrs" model="account.chart.template">
            <field name="name">Plan Contable NIIF Ecuador conforme Superintendencia de Compañías</field>
            <field name="income_currency_exchange_account_id" ref="ec430501"/>
            <field name="expense_currency_exchange_account_id" ref="ec520304"/>
            <field name="property_account_receivable_id" ref="ec1102050101"/>
            <field name="property_account_payable_id" ref="ec210301"/>
            <field name="property_account_expense_categ_id" ref="ec110307"/>
            <field name="property_account_income_categ_id" ref="ec410201"/>
            <field name="property_account_expense_id" ref="ec52040201"/>
            <field name="property_stock_account_input_categ_id" ref="ec110307"/>
            <field name="property_stock_account_output_categ_id" ref="ec510102"/>
            <field name="property_stock_valuation_account_id" ref="ec110306"/>
            <field name="default_pos_receivable_account_id" ref="ec1102050101"/>
        </record>

    </data>
</odoo>

            
```

## File: data\account_fiscal_position_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="fp_companies" model="account.fiscal.position.template">
        <field name="chart_template_id" ref="l10n_ec_ifrs"/>
        <field name="name">Sociedades - personas juridicas</field>
    </record>
    <record id="fp_special_taxation_companies" model="account.fiscal.position.template">
        <field name="chart_template_id" ref="l10n_ec_ifrs"/>
        <field name="name">Contribuyentes especiales</field>
    </record>
    <record id="fp_public_companies" model="account.fiscal.position.template">
        <field name="chart_template_id" ref="l10n_ec_ifrs"/>
        <field name="name">Sector publico y ep</field>
    </record>
    <record id="fp_person_obligated_accounting" model="account.fiscal.position.template">
        <field name="chart_template_id" ref="l10n_ec_ifrs"/>
        <field name="name">Persona natural obligada a llevar contabilidad</field>
    </record>
    <record id="fp_person_leases" model="account.fiscal.position.template">
        <field name="chart_template_id" ref="l10n_ec_ifrs"/>
        <field name="name">Persona natural no obligada - arriendos</field>
    </record>
    <record id="fp_person_professional" model="account.fiscal.position.template">
        <field name="chart_template_id" ref="l10n_ec_ifrs"/>
        <field name="name">Persona natural no obligada - profesionales</field>
    </record>
    <record id="fp_person_rustic" model="account.fiscal.position.template">
        <field name="chart_template_id" ref="l10n_ec_ifrs"/>
        <field name="name">Persona natural no obligada - liquidaciones de compras</field>
    </record>
    <record id="fp_person_other" model="account.fiscal.position.template">
        <field name="chart_template_id" ref="l10n_ec_ifrs"/>
        <field name="name">Persona natural no obligadas - emite factura o nota de venta</field>
    </record>
    <record id="fp_foreing_company_local" model="account.fiscal.position.template">
        <field name="chart_template_id" ref="l10n_ec_ifrs"/>
        <field name="name">Empresa extranjera - venta local</field>
    </record>
    <record id="fp_foreing_person_local" model="account.fiscal.position.template">
        <field name="chart_template_id" ref="l10n_ec_ifrs"/>
        <field name="name">Persona extranjera - venta local</field>
    </record>
    <record id="fp_foreing_company_exports" model="account.fiscal.position.template">
        <field name="chart_template_id" ref="l10n_ec_ifrs"/>
        <field name="name">Empresa extranjera - exportacion</field>
    </record>
    <record id="fp_foreing_person_exports" model="account.fiscal.position.template">
        <field name="chart_template_id" ref="l10n_ec_ifrs"/>
        <field name="name">Persona extranjera - exportacion</field>
    </record>
    <record id="fp_others" model="account.fiscal.position.template">
        <field name="chart_template_id" ref="l10n_ec_ifrs"/>
        <field name="name">Otras - sin cálculo automático de retención de iva</field>
    </record>
</odoo>

```

## File: data\account_group_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="ec1" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">1</field>
            <field name="name">Activo</field>
        </record>

        <record id="ec11" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">11</field>
            <field name="name">Activo corriente</field>
            <field name="parent_id" ref="ec1"/>
        </record>

        <record id="ec1101" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">1101</field>
            <field name="name">Efectivo y equivalentes al efectivo</field>
            <field name="parent_id" ref="ec11"/>
        </record>

        <record id="ec110101" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">110101</field>
            <field name="name">Efectivo</field>
            <field name="parent_id" ref="ec1101"/>
        </record>

        <record id="ec110102" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">110102</field>
            <field name="name">Bancos</field>
            <field name="parent_id" ref="ec1101"/>
        </record>

        <record id="ec110103" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">110103</field>
            <field name="name">Transferencias</field>
            <field name="parent_id" ref="ec1101"/>
        </record>

        <record id="ec110104" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">110104</field>
            <field name="name">Valores en custodia</field>
            <field name="parent_id" ref="ec1101"/>
        </record>

        <record id="ec1102" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">1102</field>
            <field name="name">Activos financieros</field>
            <field name="parent_id" ref="ec11"/>
        </record>

        <record id="ec110205" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">110205</field>
            <field name="name">Documentos y cuentas por cobrar cliente no relacionados</field>
            <field name="parent_id" ref="ec1102"/>
        </record>

        <record id="ec11020501" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">11020501</field>
            <field name="name">Documentos y cuentas por cobrar cliente no relacionados de actividades ordinarias que no
                generen intereses
            </field>
            <field name="parent_id" ref="ec110205"/>
        </record>

        <record id="ec110206" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">110206</field>
            <field name="name">Documentos y cuentas por cobrar cliente relacionados</field>
            <field name="parent_id" ref="ec1102"/>
        </record>

        <record id="ec110207" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">110207</field>
            <field name="name">Cuentas por cobrar empleados</field>
            <field name="parent_id" ref="ec1102"/>
        </record>

        <record id="ec1103" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">1103</field>
            <field name="name">Inventarios</field>
            <field name="parent_id" ref="ec11"/>
        </record>

        <record id="ec1104" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">1104</field>
            <field name="name">Servicios y otros pagos anticipados</field>
            <field name="parent_id" ref="ec11"/>
        </record>

        <record id="ec110404" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">110404</field>
            <field name="name">Otros anticipos entregados</field>
            <field name="parent_id" ref="ec1104"/>
        </record>

        <record id="ec1105" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">1105</field>
            <field name="name">Activos por impuestos corrientes</field>
            <field name="parent_id" ref="ec11"/>
        </record>

        <record id="ec110501" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">110501</field>
            <field name="name">Credito tributario a favor de la empresa(iva)</field>
            <field name="parent_id" ref="ec1105"/>
        </record>

        <record id="ec12" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">12</field>
            <field name="name">Activo no corriente</field>
            <field name="parent_id" ref="ec1"/>
        </record>

        <record id="ec1201" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">1201</field>
            <field name="name">Propiedades, planta y equipo</field>
            <field name="parent_id" ref="ec12"/>
        </record>

        <record id="ec120112" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">120112</field>
            <field name="name">Depreciacion acumulada propiedades, planta y equipo</field>
            <field name="parent_id" ref="ec1201"/>
        </record>

        <record id="ec120113" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">120113</field>
            <field name="name">Deterioro acumulado de propiedades, planta y equipo</field>
            <field name="parent_id" ref="ec1201"/>
        </record>

        <record id="ec1202" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">1202</field>
            <field name="name">Propiedades de inversion</field>
            <field name="parent_id" ref="ec12"/>
        </record>

        <record id="ec1203" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">1203</field>
            <field name="name">Activos biologicos</field>
            <field name="parent_id" ref="ec12"/>
        </record>

        <record id="ec1204" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">1204</field>
            <field name="name">Activo intangible</field>
            <field name="parent_id" ref="ec12"/>
        </record>

        <record id="ec1205" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">1205</field>
            <field name="name">Activos por impuestos diferidos</field>
            <field name="parent_id" ref="ec12"/>
        </record>

        <record id="ec1206" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">1206</field>
            <field name="name">Activos financieros no corrientes</field>
            <field name="parent_id" ref="ec12"/>
        </record>

        <record id="ec1207" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">1207</field>
            <field name="name">Otros activos no corrientes</field>
            <field name="parent_id" ref="ec12"/>
        </record>

        <record id="ec2" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">2</field>
            <field name="name">Pasivo</field>
        </record>

        <record id="ec21" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">21</field>
            <field name="name">Pasivo corriente</field>
            <field name="parent_id" ref="ec2"/>
        </record>

        <record id="ec2103" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">2103</field>
            <field name="name">Cuentas y documentos por pagar</field>
            <field name="parent_id" ref="ec21"/>
        </record>

        <record id="ec2104" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">2104</field>
            <field name="name">Obligaciones con instituciones financieras</field>
            <field name="parent_id" ref="ec21"/>
        </record>

        <record id="ec2105" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">2105</field>
            <field name="name">Provisiones</field>
            <field name="parent_id" ref="ec21"/>
        </record>

        <record id="ec210501" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">210501</field>
            <field name="name">Provisiones locales</field>
            <field name="parent_id" ref="ec2105"/>
        </record>

        <record id="ec21050101" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">21050101</field>
            <field name="name">Provisiones a favor de los empleados</field>
            <field name="parent_id" ref="ec210501"/>
        </record>

        <record id="ec2107" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">2107</field>
            <field name="name">Otras obligaciones corrientes</field>
            <field name="parent_id" ref="ec21"/>
        </record>

        <record id="ec210701" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">210701</field>
            <field name="name">Otras obligaciones corrientes con la administracion tributaria</field>
            <field name="parent_id" ref="ec2107"/>
        </record>

        <record id="ec210703" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">210703</field>
            <field name="name">Otras obligaciones corrientes con el iess</field>
            <field name="parent_id" ref="ec2107"/>
        </record>

        <record id="ec210704" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">210704</field>
            <field name="name">Otras obligaciones corrientes por beneficios de ley a empleados</field>
            <field name="parent_id" ref="ec2107"/>
        </record>

        <record id="ec2112" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">2112</field>
            <field name="name">Porcion corriente de provisiones por beneficios a empleados</field>
            <field name="parent_id" ref="ec21"/>
        </record>

        <record id="ec2113" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">2113</field>
            <field name="name">Otros pasivos corrientes</field>
            <field name="parent_id" ref="ec21"/>
        </record>

        <record id="ec2114" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">2114</field>
            <field name="name">Obligaciones tributarias</field>
            <field name="parent_id" ref="ec21"/>
        </record>

        <record id="ec211401" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">211401</field>
            <field name="name">Impuestos</field>
            <field name="parent_id" ref="ec2114"/>
        </record>

        <record id="ec21140101" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">21140101</field>
            <field name="name">Iva cobrado</field>
            <field name="parent_id" ref="ec211401"/>
        </record>

        <record id="ec21140102" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">21140102</field>
            <field name="name">Retenciones de la fuente</field>
            <field name="parent_id" ref="ec211401"/>
        </record>

        <record id="ec21140103" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">21140103</field>
            <field name="name">Retenciones de iva</field>
            <field name="parent_id" ref="ec211401"/>
        </record>

        <record id="ec22" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">22</field>
            <field name="name">Pasivo no corriente</field>
            <field name="parent_id" ref="ec2"/>
        </record>

        <record id="ec2202" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">2202</field>
            <field name="name">Cuentas y documentos por pagar</field>
            <field name="parent_id" ref="ec22"/>
        </record>

        <record id="ec2203" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">2203</field>
            <field name="name">Obligaciones con instituciones financieras</field>
            <field name="parent_id" ref="ec22"/>
        </record>

        <record id="ec2204" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">2204</field>
            <field name="name">Cuentas por pagar diversas/relacionadas</field>
            <field name="parent_id" ref="ec22"/>
        </record>

        <record id="ec2207" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">2207</field>
            <field name="name">Provisiones por beneficios a empleados</field>
            <field name="parent_id" ref="ec22"/>
        </record>

        <record id="ec2209" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">2209</field>
            <field name="name">Pasivo diferido</field>
            <field name="parent_id" ref="ec22"/>
        </record>

        <record id="ec3" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">3</field>
            <field name="name">Patrimonio neto</field>
        </record>

        <record id="ec31" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">31</field>
            <field name="name">Capital</field>
            <field name="parent_id" ref="ec3"/>
        </record>

        <record id="ec34" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">34</field>
            <field name="name">Reservas</field>
            <field name="parent_id" ref="ec3"/>
        </record>

        <record id="ec35" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">35</field>
            <field name="name">Otros resultados integrales</field>
            <field name="parent_id" ref="ec3"/>
        </record>

        <record id="ec36" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">36</field>
            <field name="name">Resultados acumulados</field>
            <field name="parent_id" ref="ec3"/>
        </record>

        <record id="ec37" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">37</field>
            <field name="name">Resultados del ejercicio</field>
            <field name="parent_id" ref="ec3"/>
        </record>

        <record id="ec4" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">4</field>
            <field name="name">Ingresos</field>
        </record>

        <record id="ec41" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">41</field>
            <field name="name">Ingresos de actividades ordinarias</field>
            <field name="parent_id" ref="ec4"/>
        </record>

        <record id="ec4101" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">4101</field>
            <field name="name">Venta de bienes</field>
            <field name="parent_id" ref="ec41"/>
        </record>

        <record id="ec4102" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">4102</field>
            <field name="name">Prestacion de servicios</field>
            <field name="parent_id" ref="ec41"/>
        </record>

        <record id="ec43" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">43</field>
            <field name="name">Otros ingresos</field>
            <field name="parent_id" ref="ec4"/>
        </record>

        <record id="ec4305" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">4305</field>
            <field name="name">Otras rentas</field>
            <field name="parent_id" ref="ec43"/>
        </record>

        <record id="ec5" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">5</field>
            <field name="name">Egresos</field>
        </record>

        <record id="ec51" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">51</field>
            <field name="name">Costo de ventas y produccion</field>
            <field name="parent_id" ref="ec5"/>
        </record>

        <record id="ec5101" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">5101</field>
            <field name="name">Materiales utilizados o productos vendidos</field>
            <field name="parent_id" ref="ec51"/>
        </record>

        <record id="ec5102" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">5102</field>
            <field name="name">Mano de obra directa</field>
            <field name="parent_id" ref="ec51"/>
        </record>

        <record id="ec510201" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">510201</field>
            <field name="name">Sueldos, salarios y demas remuneraciones</field>
            <field name="parent_id" ref="ec5102"/>
        </record>

        <record id="ec510202" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">510202</field>
            <field name="name">Gasto planes de beneficios a empleados</field>
            <field name="parent_id" ref="ec5102"/>
        </record>

        <record id="ec51020205" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">51020205</field>
            <field name="name">Otros gastos de planes de beneficios a empleados</field>
            <field name="parent_id" ref="ec510202"/>
        </record>

        <record id="ec510203" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">510203</field>
            <field name="name">Aportes a la seguridad social</field>
            <field name="parent_id" ref="ec5102"/>
        </record>

        <record id="ec510204" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">510204</field>
            <field name="name">Beneficios sociales e indemnizaciones</field>
            <field name="parent_id" ref="ec5102"/>
        </record>

        <record id="ec5103" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">5103</field>
            <field name="name">Mano de obra indirecta</field>
            <field name="parent_id" ref="ec51"/>
        </record>

        <record id="ec510301" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">510301</field>
            <field name="name">Sueldos, salarios y demas remuneraciones</field>
            <field name="parent_id" ref="ec5103"/>
        </record>

        <record id="ec510302" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">510302</field>
            <field name="name">Gasto planes de beneficios a empleados</field>
            <field name="parent_id" ref="ec5103"/>
        </record>

        <record id="ec51030205" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">51030205</field>
            <field name="name">Otros gastos de planes de beneficios a empleados</field>
            <field name="parent_id" ref="ec510302"/>
        </record>

        <record id="ec510303" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">510303</field>
            <field name="name">Aportes a la seguridad social (incluido fondo de reserva)</field>
            <field name="parent_id" ref="ec5103"/>
        </record>

        <record id="ec510304" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">510304</field>
            <field name="name">Beneficios sociales e indemnizaciones</field>
            <field name="parent_id" ref="ec5103"/>
        </record>

        <record id="ec5104" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">5104</field>
            <field name="name">Otros costos indirectos de fabricacion</field>
            <field name="parent_id" ref="ec51"/>
        </record>

        <record id="ec510401" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">510401</field>
            <field name="name">Depreciacion propiedades, planta y equipo</field>
            <field name="parent_id" ref="ec5104"/>
        </record>

        <record id="ec510403" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">510403</field>
            <field name="name">Deterioro de propiedad, planta y equipo</field>
            <field name="parent_id" ref="ec5104"/>
        </record>

        <record id="ec510408" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">510408</field>
            <field name="name">Otros costos de produccion</field>
            <field name="parent_id" ref="ec5104"/>
        </record>

        <record id="ec52" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">52</field>
            <field name="name">Gastos</field>
            <field name="parent_id" ref="ec5"/>
        </record>

        <record id="ec5201" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">5201</field>
            <field name="name">Gastos de ventas</field>
            <field name="parent_id" ref="ec52"/>
        </record>

        <record id="ec520101" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520101</field>
            <field name="name">Sueldos, salarios y demas remuneraciones</field>
            <field name="parent_id" ref="ec5201"/>
        </record>

        <record id="ec520102" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520102</field>
            <field name="name">Aportes a la seguridad social (incluido fondo de reserva)</field>
            <field name="parent_id" ref="ec5201"/>
        </record>

        <record id="ec520103" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520103</field>
            <field name="name">Beneficios sociales e indemnizaciones</field>
            <field name="parent_id" ref="ec5201"/>
        </record>

        <record id="ec520104" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520104</field>
            <field name="name">Gasto planes de beneficios a empleados</field>
            <field name="parent_id" ref="ec5201"/>
        </record>

        <record id="ec52010405" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">52010405</field>
            <field name="name">Otros gastos de planes de beneficios a empleados</field>
            <field name="parent_id" ref="ec520104"/>
        </record>

        <record id="ec520118" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520118</field>
            <field name="name">Agua, energia, luz, y telecomunicaciones</field>
            <field name="parent_id" ref="ec5201"/>
        </record>

        <record id="ec520121" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520121</field>
            <field name="name">Depreciaciones</field>
            <field name="parent_id" ref="ec5201"/>
        </record>

        <record id="ec52012101" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">52012101</field>
            <field name="name">Propiedades, planta y equipo</field>
            <field name="parent_id" ref="ec520121"/>
        </record>

        <record id="ec520122" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520122</field>
            <field name="name">Amortizaciones</field>
            <field name="parent_id" ref="ec5201"/>
        </record>

        <record id="ec520123" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520123</field>
            <field name="name">Gasto deterioro</field>
            <field name="parent_id" ref="ec5201"/>
        </record>

        <record id="ec52012301" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">52012301</field>
            <field name="name">Propiedades, planta y equipo</field>
            <field name="parent_id" ref="ec520123"/>
        </record>

        <record id="ec520127" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520127</field>
            <field name="name">Otros gastos</field>
            <field name="parent_id" ref="ec5201"/>
        </record>

        <record id="ec5202" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">5202</field>
            <field name="name">Gastos administrativos</field>
            <field name="parent_id" ref="ec52"/>
        </record>

        <record id="ec520201" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520201</field>
            <field name="name">Sueldos, salarios y demas remuneraciones</field>
            <field name="parent_id" ref="ec5202"/>
        </record>

        <record id="ec520202" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520202</field>
            <field name="name">Aportes a la seguridad social</field>
            <field name="parent_id" ref="ec5202"/>
        </record>

        <record id="ec520203" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520203</field>
            <field name="name">Beneficios sociales e indemnizaciones</field>
            <field name="parent_id" ref="ec5202"/>
        </record>

        <record id="ec520204" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520204</field>
            <field name="name">Gasto planes de beneficios a empleados</field>
            <field name="parent_id" ref="ec5202"/>
        </record>

        <record id="ec520218" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520218</field>
            <field name="name">Agua, energia, luz, y telecomunicaciones</field>
            <field name="parent_id" ref="ec5202"/>
        </record>

        <record id="ec520220" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520220</field>
            <field name="name">Impuestos, contribuciones y otros</field>
            <field name="parent_id" ref="ec5202"/>
        </record>

        <record id="ec520221" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520221</field>
            <field name="name">Depreciaciones</field>
            <field name="parent_id" ref="ec5202"/>
        </record>

        <record id="ec52022101" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">52022101</field>
            <field name="name">Propiedades, planta y equipo</field>
            <field name="parent_id" ref="ec520221"/>
        </record>

        <record id="ec520222" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520222</field>
            <field name="name">Amortizaciones</field>
            <field name="parent_id" ref="ec5202"/>
        </record>

        <record id="ec520223" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520223</field>
            <field name="name">Gasto deterioro</field>
            <field name="parent_id" ref="ec5202"/>
        </record>

        <record id="ec52022301" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">52022301</field>
            <field name="name">Propiedades, planta y equipo</field>
            <field name="parent_id" ref="ec520223"/>
        </record>

        <record id="ec520228" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520228</field>
            <field name="name">Otros gastos</field>
            <field name="parent_id" ref="ec5202"/>
        </record>

        <record id="ec52022818" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">52022818</field>
            <field name="name">Gnd gastos no deducibles</field>
            <field name="parent_id" ref="ec520228"/>
        </record>

        <record id="ec5203" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">5203</field>
            <field name="name">Gastos financieros</field>
            <field name="parent_id" ref="ec52"/>
        </record>

        <record id="ec5204" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">5204</field>
            <field name="name">Otros gastos</field>
            <field name="parent_id" ref="ec52"/>
        </record>

        <record id="ec520402" model="account.group.template">
            <field name="chart_template_id" eval="ref('l10n_ec.l10n_ec_ifrs')"/>
            <field name="code_prefix_start">520402</field>
            <field name="name">Otros</field>
            <field name="parent_id" ref="ec5204"/>
        </record>

    </data>
</odoo>
```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="tax_group_vat_12" model="account.tax.group">
            <field name="name">VAT 12%</field>
            <field name="l10n_ec_type">vat12</field>
            <field name="sequence">10</field>
            <field name="country_id" ref="base.ec"/>
        </record>

        <record id="tax_group_vat14" model="account.tax.group">
            <field name="name">VAT 14%</field>
            <field name="l10n_ec_type">vat14</field>
            <field name="sequence">20</field>
            <field name="country_id" ref="base.ec"/>
        </record>

        <record id="tax_group_vat0" model="account.tax.group">
            <field name="name">VAT 0%</field>
            <field name="l10n_ec_type">zero_vat</field>
            <field name="sequence">30</field>
            <field name="country_id" ref="base.ec"/>
        </record>

        <record id="tax_group_vat_not_charged" model="account.tax.group">
            <field name="name">VAT Not Charged</field>
            <field name="l10n_ec_type">not_charged_vat</field>
            <field name="sequence">40</field>
            <field name="country_id" ref="base.ec"/>
        </record>

        <record id="tax_group_vat_excempt" model="account.tax.group">
            <field name="name">VAT Excempt</field>
            <field name="l10n_ec_type">exempt_vat</field>
            <field name="sequence">50</field>
            <field name="country_id" ref="base.ec"/>
        </record>

        <record id="tax_group_ice" model="account.tax.group">
            <field name="name">Special Consumptions (ICE)</field>
            <field name="l10n_ec_type">ice</field>
            <field name="sequence">60</field>
            <field name="country_id" ref="base.ec"/>
        </record>

        <record id="tax_group_irbpnr" model="account.tax.group">
            <field name="name">Plastic Bottles (IRBPNR)</field>
            <field name="l10n_ec_type">irbpnr</field>
            <field name="sequence">70</field>
            <field name="country_id" ref="base.ec"/>
        </record>

        <record id="tax_group_withhold_vat" model="account.tax.group">
            <field name="name">VAT Withhold</field>
            <field name="l10n_ec_type">withhold_vat</field>
            <field name="sequence">80</field>
            <field name="country_id" ref="base.ec"/>
        </record>

        <record id="tax_group_withhold_income" model="account.tax.group">
            <field name="name">Profit Withhold</field>
            <field name="l10n_ec_type">withhold_income_tax</field>
            <field name="sequence">90</field>
            <field name="country_id" ref="base.ec"/>
        </record>

        <record id="tax_group_outflows" model="account.tax.group">
            <field name="name">Exchange Outflows</field>
            <field name="l10n_ec_type">outflows_tax</field>
            <field name="sequence">100</field>
            <field name="country_id" ref="base.ec"/>
        </record>

        <record id="tax_group_others" model="account.tax.group">
            <field name="name">Others</field>
            <field name="l10n_ec_type">other</field>
            <field name="sequence">110</field>
            <field name="country_id" ref="base.ec"/>
        </record>

    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id="tax_report_104" model="account.tax.report">
            <field name="name">104</field>
            <field name="country_id" ref="base.ec"/>
        </record>
        <record id="tax_report_line_parent_line_report_1" model="account.tax.report.line">
            <field name="name">RESUMEN DE VENTAS Y OTRAS OPERACIONES DEL PERÍODO QUE DECLARA</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>
        <record id="tax_report_line_parent_line_report_2" model="account.tax.report.line">
            <field name="name">Ventas locales (excluye activos fijos) gravadas tarifa diferente de cero</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_1"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_401" model="account.tax.report.line">
            <field name="name">Valor bruto(401)</field>
            <field name="code">c401</field>
            <field name="tag_name">401 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_2"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_411" model="account.tax.report.line">
            <field name="name">Valor neto(411)</field>
            <field name="code">c411</field>
            <field name="tag_name">411 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_2"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_421" model="account.tax.report.line">
            <field name="name">Impuesto generado(421)</field>
            <field name="code">c421</field>
            <field name="tag_name">421 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_2"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_6" model="account.tax.report.line">
            <field name="name">Ventas de activos fijos gravadas tarifa diferente de cero</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_1"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_402" model="account.tax.report.line">
            <field name="name">Valor bruto(402)</field>
            <field name="code">c402</field>
            <field name="tag_name">402 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_6"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_412" model="account.tax.report.line">
            <field name="name">Valor neto(412)</field>
            <field name="code">c412</field>
            <field name="tag_name">412 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_6"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_422" model="account.tax.report.line">
            <field name="name">Impuesto generado(422)</field>
            <field name="code">c422</field>
            <field name="tag_name">422 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_6"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_10" model="account.tax.report.line">
            <field name="name">IVA generado en la diferencia entre ventas y notas de crédito con distinta tarifa (ajuste
                a pagar)
            </field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_1"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_423" model="account.tax.report.line">
            <field name="name">Impuesto generado(423)</field>
            <field name="code">c423</field>
            <field name="tag_name">423 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_10"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_12" model="account.tax.report.line">
            <field name="name">IVA generado en la diferencia entre ventas y notas de crédito con distinta tarifa (ajuste
                a favor)
            </field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_1"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_424" model="account.tax.report.line">
            <field name="name">Impuesto generado(424)</field>
            <field name="code">c424</field>
            <field name="tag_name">424 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_12"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_14" model="account.tax.report.line">
            <field name="name">Ventas locales (excluye activos fijos) gravadas tarifa 0% que no dan derecho a crédito
                tributario
            </field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_1"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_403" model="account.tax.report.line">
            <field name="name">Valor bruto(403)</field>
            <field name="code">c403</field>
            <field name="tag_name">403 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_14"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_413" model="account.tax.report.line">
            <field name="name">Valor neto(413)</field>
            <field name="code">c413</field>
            <field name="tag_name">413 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_14"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_17" model="account.tax.report.line">
            <field name="name">Ventas de activos fijos gravadas tarifa 0% que no dan derecho a crédito tributario
            </field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_1"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_404" model="account.tax.report.line">
            <field name="name">Valor bruto(404)</field>
            <field name="code">c404</field>
            <field name="tag_name">404 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_17"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_414" model="account.tax.report.line">
            <field name="name">Valor neto(414)</field>
            <field name="code">c414</field>
            <field name="tag_name">414 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_17"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_20" model="account.tax.report.line">
            <field name="name">Ventas locales (excluye activos fijos) gravadas tarifa 0% que dan derecho a crédito
                tributario
            </field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_1"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_405" model="account.tax.report.line">
            <field name="name">Valor bruto(405)</field>
            <field name="code">c405</field>
            <field name="tag_name">405 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_20"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_415" model="account.tax.report.line">
            <field name="name">Valor neto(415)</field>
            <field name="code">c415</field>
            <field name="tag_name">415 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_20"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_23" model="account.tax.report.line">
            <field name="name">Ventas de activos fijos gravadas tarifa 0% que dan derecho a crédito tributario</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_1"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_406" model="account.tax.report.line">
            <field name="name">Valor bruto(406)</field>
            <field name="code">c406</field>
            <field name="tag_name">406 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_23"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_416" model="account.tax.report.line">
            <field name="name">Valor neto(416)</field>
            <field name="code">c416</field>
            <field name="tag_name">416 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_23"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_26" model="account.tax.report.line">
            <field name="name">Exportaciones de bienes</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_1"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_407" model="account.tax.report.line">
            <field name="name">Valor bruto(407)</field>
            <field name="code">c407</field>
            <field name="tag_name">407 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_26"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_417" model="account.tax.report.line">
            <field name="name">Valor neto(417)</field>
            <field name="code">c417</field>
            <field name="tag_name">417 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_26"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_29" model="account.tax.report.line">
            <field name="name">Exportaciones de servicios y/o derechos</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_1"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_408" model="account.tax.report.line">
            <field name="name">Valor bruto(408)</field>
            <field name="code">c408</field>
            <field name="tag_name">408 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_29"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_418" model="account.tax.report.line">
            <field name="name">Valor neto(418)</field>
            <field name="code">c418</field>
            <field name="tag_name">418 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_29"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_32" model="account.tax.report.line">
            <field name="name">TOTAL VENTAS Y OTRAS OPERACIONES</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_1"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_409" model="account.tax.report.line">
            <field name="name">Valor bruto(409)</field>
            <field name="code">c409</field>
            <field name="tag_name">409 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_32"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_419" model="account.tax.report.line">
            <field name="name">Valor neto(419)</field>
            <field name="code">c419</field>
            <field name="tag_name">419 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_32"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_429" model="account.tax.report.line">
            <field name="name">Impuesto generado(429)</field>
            <field name="code">c429</field>
            <field name="tag_name">429 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_32"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_36" model="account.tax.report.line">
            <field name="name">Transferencias no objeto o exentas de IVA</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_1"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_431" model="account.tax.report.line">
            <field name="name">Valor bruto(431)</field>
            <field name="code">c431</field>
            <field name="tag_name">431 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_36"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_441" model="account.tax.report.line">
            <field name="name">Valor neto(441)</field>
            <field name="code">c441</field>
            <field name="tag_name">441 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_36"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_39" model="account.tax.report.line">
            <field name="name">Notas de crédito tarifa 0% por compensar próximo mes</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_1"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_442" model="account.tax.report.line">
            <field name="name">Valor neto(442)</field>
            <field name="code">c442</field>
            <field name="tag_name">442 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_39"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_41" model="account.tax.report.line">
            <field name="name">Notas de crédito tarifa diferente de cero por compensar próximo mes</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_1"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_443" model="account.tax.report.line">
            <field name="name">Valor neto(443)</field>
            <field name="code">c443</field>
            <field name="tag_name">443 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_41"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_453" model="account.tax.report.line">
            <field name="name">Impuesto generado(453)</field>
            <field name="code">c453</field>
            <field name="tag_name">453 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_41"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_44" model="account.tax.report.line">
            <field name="name">Ingresos por reembolso como intermediario / valores facturados por operadoras de
                transporte (informativo)
            </field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_1"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_434" model="account.tax.report.line">
            <field name="name">Valor bruto(434)</field>
            <field name="code">c434</field>
            <field name="tag_name">434 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_44"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_444" model="account.tax.report.line">
            <field name="name">Valor neto(444)</field>
            <field name="code">c444</field>
            <field name="tag_name">444 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_44"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_454" model="account.tax.report.line">
            <field name="name">Impuesto generado(454)</field>
            <field name="code">c454</field>
            <field name="tag_name">454 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_44"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_48" model="account.tax.report.line">
            <field name="name">LIQUIDACIÓN DEL IVA EN EL MES</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_480" model="account.tax.report.line">
            <field name="name">Total transferencias gravadas tarifa diferente de cero a contado este mes(480)</field>
            <field name="code">c480</field>
            <field name="tag_name">480 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_48"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_481" model="account.tax.report.line">
            <field name="name">Total transferencias gravadas tarifa diferente de cero a crédito este mes(481)</field>
            <field name="code">c481</field>
            <field name="tag_name">481 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_48"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_482" model="account.tax.report.line">
            <field name="name">Total impuesto generado(482)</field>
            <field name="code">c482</field>
            <field name="tag_name">482 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_48"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_483" model="account.tax.report.line">
            <field name="name">Impuesto a liquidar del mes anterior(483)</field>
            <field name="code">c483</field>
            <field name="tag_name">483 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_48"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_484" model="account.tax.report.line">
            <field name="name">Impuesto a liquidar en este mes(484)</field>
            <field name="code">c484</field>
            <field name="tag_name">484 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_48"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_495" model="account.tax.report.line">
            <field name="name">Impuesto a liquidar en el próximo mes(495)</field>
            <field name="code">c495</field>
            <field name="tag_name">495 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_48"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_499" model="account.tax.report.line">
            <field name="name">TOTAL IMPUESTO A LIQUIDAR EN ESTE MES(499)</field>
            <field name="code">c499</field>
            <field name="tag_name">499 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_48"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_56" model="account.tax.report.line">
            <field name="name">RESUMEN DE ADQUISICIONES Y PAGOS DEL PERÍODO QUE DECLARA</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>
        <record id="tax_report_line_parent_line_report_57" model="account.tax.report.line">
            <field name="name">Adquisiciones y pagos (excluye activos fijos) gravados tarifa diferente de cero (con
                derecho a crédito tributario)
            </field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_56"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_500" model="account.tax.report.line">
            <field name="name">Valor bruto(500)</field>
            <field name="code">c500</field>
            <field name="tag_name">500 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_57"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_510" model="account.tax.report.line">
            <field name="name">Valor neto(510)</field>
            <field name="code">c510</field>
            <field name="tag_name">510 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_57"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_520" model="account.tax.report.line">
            <field name="name">Impuesto generado(520)</field>
            <field name="code">c520</field>
            <field name="tag_name">520 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_57"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_61" model="account.tax.report.line">
            <field name="name">Adquisiciones locales de activos fijos gravados tarifa diferente de cero (con derecho a
                crédito tributario)
            </field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_56"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_501" model="account.tax.report.line">
            <field name="name">Valor bruto(501)</field>
            <field name="code">c501</field>
            <field name="tag_name">501 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_61"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_511" model="account.tax.report.line">
            <field name="name">Valor neto(511)</field>
            <field name="code">c511</field>
            <field name="tag_name">511 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_61"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_521" model="account.tax.report.line">
            <field name="name">Impuesto generado(521)</field>
            <field name="code">c521</field>
            <field name="tag_name">521 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_61"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_65" model="account.tax.report.line">
            <field name="name">Otras adquisiciones y pagos gravados tarifa diferente de cero (sin derecho a crédito
                tributario)
            </field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_56"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_502" model="account.tax.report.line">
            <field name="name">Valor bruto(502)</field>
            <field name="code">c502</field>
            <field name="tag_name">502 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_65"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_512" model="account.tax.report.line">
            <field name="name">Valor neto(512)</field>
            <field name="code">c512</field>
            <field name="tag_name">512 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_65"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_522" model="account.tax.report.line">
            <field name="name">Impuesto generado(522)</field>
            <field name="code">c522</field>
            <field name="tag_name">522 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_65"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_69" model="account.tax.report.line">
            <field name="name">Importaciones de servicios y/o derechos gravados tarifa diferente de cero</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_56"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_503" model="account.tax.report.line">
            <field name="name">Valor bruto(503)</field>
            <field name="code">c503</field>
            <field name="tag_name">503 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_69"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_513" model="account.tax.report.line">
            <field name="name">Valor neto(513)</field>
            <field name="code">c513</field>
            <field name="tag_name">513 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_69"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_523" model="account.tax.report.line">
            <field name="name">Impuesto generado(523)</field>
            <field name="code">c523</field>
            <field name="tag_name">523 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_69"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_73" model="account.tax.report.line">
            <field name="name">Importaciones de bienes (excluye activos fijos) gravados tarifa diferente de cero</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_56"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_504" model="account.tax.report.line">
            <field name="name">Valor bruto(504)</field>
            <field name="code">c504</field>
            <field name="tag_name">504 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_73"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_514" model="account.tax.report.line">
            <field name="name">Valor neto(514)</field>
            <field name="code">c514</field>
            <field name="tag_name">514 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_73"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_524" model="account.tax.report.line">
            <field name="name">Impuesto generado(524)</field>
            <field name="code">c524</field>
            <field name="tag_name">524 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_73"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_77" model="account.tax.report.line">
            <field name="name">Importaciones de activos fijos gravados tarifa diferente de cero</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_56"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_505" model="account.tax.report.line">
            <field name="name">Valor bruto(505)</field>
            <field name="code">c505</field>
            <field name="tag_name">505 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_77"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_515" model="account.tax.report.line">
            <field name="name">Valor neto(515)</field>
            <field name="code">c515</field>
            <field name="tag_name">515 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_77"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_525" model="account.tax.report.line">
            <field name="name">Impuesto generado(525)</field>
            <field name="code">c525</field>
            <field name="tag_name">525 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_77"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_81" model="account.tax.report.line">
            <field name="name">IVA generado en la diferencia entre adquisiciones y notas de crédito con distinta tarifa
                (ajuste en positivo al crédito tributario)
            </field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_56"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_526" model="account.tax.report.line">
            <field name="name">Impuesto generado(526)</field>
            <field name="code">c526</field>
            <field name="tag_name">526 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_81"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_83" model="account.tax.report.line">
            <field name="name">IVA generado en la diferencia entre adquisiciones y notas de crédito con distinta tarifa
                (ajuste en negativo al crédito tributario)
            </field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_56"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_527" model="account.tax.report.line">
            <field name="name">Impuesto generado(527)</field>
            <field name="code">c527</field>
            <field name="tag_name">527 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_83"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_85" model="account.tax.report.line">
            <field name="name">Importaciones de bienes (incluye activos fijos) gravados tarifa 0%</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_56"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_506" model="account.tax.report.line">
            <field name="name">Valor bruto(506)</field>
            <field name="code">c506</field>
            <field name="tag_name">506 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_85"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_516" model="account.tax.report.line">
            <field name="name">Valor neto(516)</field>
            <field name="code">c516</field>
            <field name="tag_name">516 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_85"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_88" model="account.tax.report.line">
            <field name="name">Adquisiciones y pagos (incluye activos fijos) gravados tarifa 0%</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_56"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_507" model="account.tax.report.line">
            <field name="name">Valor bruto(507)</field>
            <field name="code">c507</field>
            <field name="tag_name">507 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_88"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_517" model="account.tax.report.line">
            <field name="name">Valor neto(517)</field>
            <field name="code">c517</field>
            <field name="tag_name">517 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_88"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_91" model="account.tax.report.line">
            <field name="name">Adquisiciones realizadas a contribuyentes RISE</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_56"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_508" model="account.tax.report.line">
            <field name="name">Valor bruto(508)</field>
            <field name="code">c508</field>
            <field name="tag_name">508 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_91"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_518" model="account.tax.report.line">
            <field name="name">Valor neto(518)</field>
            <field name="code">c518</field>
            <field name="tag_name">518 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_91"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_94" model="account.tax.report.line">
            <field name="name">TOTAL ADQUISICIONES Y PAGOS</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_56"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_509" model="account.tax.report.line">
            <field name="name">Valor bruto(509)</field>
            <field name="code">c509</field>
            <field name="tag_name">509 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_94"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_519" model="account.tax.report.line">
            <field name="name">Valor neto(519)</field>
            <field name="code">c519</field>
            <field name="tag_name">519 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_94"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_529" model="account.tax.report.line">
            <field name="name">Impuesto generado(529)</field>
            <field name="code">c529</field>
            <field name="tag_name">529 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_94"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_98" model="account.tax.report.line">
            <field name="name">Adquisiciones no objeto de IVA</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_56"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_531" model="account.tax.report.line">
            <field name="name">Valor bruto(531)</field>
            <field name="code">c531</field>
            <field name="tag_name">531 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_98"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_541" model="account.tax.report.line">
            <field name="name">Valor neto(541)</field>
            <field name="code">c541</field>
            <field name="tag_name">541 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_98"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_101" model="account.tax.report.line">
            <field name="name">Adquisiciones exentas del pago de IVA</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_56"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_532" model="account.tax.report.line">
            <field name="name">Valor bruto(532)</field>
            <field name="code">c532</field>
            <field name="tag_name">532 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_101"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_542" model="account.tax.report.line">
            <field name="name">Valor neto(542)</field>
            <field name="code">c542</field>
            <field name="tag_name">542 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_101"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_104" model="account.tax.report.line">
            <field name="name">Notas de crédito tarifa 0% por compensar próximo mes</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_56"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_543" model="account.tax.report.line">
            <field name="name">Valor neto(543)</field>
            <field name="code">c543</field>
            <field name="tag_name">543 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_104"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_106" model="account.tax.report.line">
            <field name="name">Notas de crédito tarifa diferente de cero por compensar próximo mes</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_56"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_544" model="account.tax.report.line">
            <field name="name">Valor neto(544)</field>
            <field name="code">c544</field>
            <field name="tag_name">544 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_106"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_554" model="account.tax.report.line">
            <field name="name">Impuesto generado(554)</field>
            <field name="code">c554</field>
            <field name="tag_name">554 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_106"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_109" model="account.tax.report.line">
            <field name="name">Pagos netos por reembolso como intermediario / valores facturados por socios a operadoras
                de transporte (informativo)
            </field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_56"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_535" model="account.tax.report.line">
            <field name="name">Valor bruto(535)</field>
            <field name="code">c535</field>
            <field name="tag_name">535 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_109"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_545" model="account.tax.report.line">
            <field name="name">Valor neto(545)</field>
            <field name="code">c545</field>
            <field name="tag_name">545 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_109"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_555" model="account.tax.report.line">
            <field name="name">Impuesto generado(555)</field>
            <field name="code">c555</field>
            <field name="tag_name">555 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_109"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_113" model="account.tax.report.line">
            <field name="name">RESUMEN IMPOSITIVO: AGENTE DE PERCEPCIÓN DEL IMPUESTO AL VALOR AGREGADO</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_601" model="account.tax.report.line">
            <field name="name">Impuesto causado (si la diferencia de los campos 499-564 es mayor que cero)(601)</field>
            <field name="code">c601</field>
            <field name="tag_name">601 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_113"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_602" model="account.tax.report.line">
            <field name="name">Crédito tributario aplicable en este período (si la diferencia de los campos 499-564 es
                menor que cero)(602)
            </field>
            <field name="code">c602</field>
            <field name="tag_name">602 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_113"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_604" model="account.tax.report.line">
            <field name="name">(-) Compensación de IVA por ventas efectuadas en zonas afectadas - Ley de solidaridad,
                restitución de crédito tributario en resoluciones(604)
            </field>
            <field name="code">c604</field>
            <field name="tag_name">604 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_113"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_117" model="account.tax.report.line">
            <field name="name">(-) Saldo crédito tributario del mes anterior</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_113"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_605" model="account.tax.report.line">
            <field name="name">Por adquisiciones e importaciones (trasládese el campo 615 de la declaración del período
                anterior)(605)
            </field>
            <field name="code">c605</field>
            <field name="tag_name">605 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_117"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_606" model="account.tax.report.line">
            <field name="name">Por retenciones en la fuente de IVA que le han sido efectuadas (trasládese el campo 617
                de la declaración del período anterior)(606)
            </field>
            <field name="code">c606</field>
            <field name="tag_name">606 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_117"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_608" model="account.tax.report.line">
            <field name="name">Por compensación de IVA por ventas efectuadas en zonas afectadas - Ley de solidaridad
                (trasládese el campo 619 de la declaración del período anterior)(608)
            </field>
            <field name="code">c608</field>
            <field name="tag_name">608 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_117"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_609" model="account.tax.report.line">
            <field name="name">(-) Retenciones en la fuente de IVA que le han sido efectuadas en este período(609)
            </field>
            <field name="code">c609</field>
            <field name="tag_name">609 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_113"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_610" model="account.tax.report.line">
            <field name="name">(+) Ajuste por IVA devuelto o descontado por adquisiciones efectuadas con medio
                electrónico(610)
            </field>
            <field name="code">c610</field>
            <field name="tag_name">610 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_113"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_612" model="account.tax.report.line">
            <field name="name">(+) Ajuste por IVA devuelto e IVA rechazado (por concepto de devoluciones de IVA), ajuste
                de IVA por procesos de control y otros (adquisiciones en importaciones), imputables al crédito
                tributario(612)
            </field>
            <field name="code">c612</field>
            <field name="tag_name">612 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_113"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_613" model="account.tax.report.line">
            <field name="name">(+) Ajuste por IVA devuelto e IVA rechazado, ajuste de IVA por procesos de control y
                otros (por concepto retenciones en la fuente de IVA), imputables al crédito tributario(613)
            </field>
            <field name="code">c613</field>
            <field name="tag_name">613 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_113"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_614" model="account.tax.report.line">
            <field name="name">(+) Ajuste por IVA devuelto por otras instituciones del sector público imputable al
                crédito tributario en el mes(614)
            </field>
            <field name="code">c614</field>
            <field name="tag_name">614 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_113"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_126" model="account.tax.report.line">
            <field name="name">Saldo crédito tributario para el próximo mes</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_113"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_615" model="account.tax.report.line">
            <field name="name">Por adquisiciones e Importaciones(615)</field>
            <field name="code">c615</field>
            <field name="tag_name">615 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_126"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_617" model="account.tax.report.line">
            <field name="name">Por retenciones en la fuente de IVA que le han sido efectuadas(617)</field>
            <field name="code">c617</field>
            <field name="tag_name">617 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_126"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_619" model="account.tax.report.line">
            <field name="name">Por compensación de IVA por ventas efectuadas en zonas afectadas - Ley de solidaridad,
                restitución de crédito tributario en resoluciones(619)
            </field>
            <field name="code">c619</field>
            <field name="tag_name">619 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_126"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_620" model="account.tax.report.line">
            <field name="name">SUBTOTAL A PAGAR Si (601-602-603-604-605-606-607-608-609+610+611+612+613+614) > 0(620)
            </field>
            <field name="code">c620</field>
            <field name="tag_name">620 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_113"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_621" model="account.tax.report.line">
            <field name="name">IVA presuntivo de salas de juego (bingo mecánicos) y otros juegos de azar (aplica para
                ejercicios anteriores al 2013)(621)
            </field>
            <field name="code">c621</field>
            <field name="tag_name">621 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_113"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_699" model="account.tax.report.line">
            <field name="name">TOTAL IMPUESTO A PAGAR POR PERCEPCIÓN(699)</field>
            <field name="code">c699</field>
            <field name="tag_name">699 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_113"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_133" model="account.tax.report.line">
            <field name="name">IMPUESTO A LA SALIDA DE DIVISAS A EFECTOS DE DEVOLUCIÓN A EXPORTADORES HABITUALES DE
                BIENES
            </field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>
        <record id="tax_report_line_parent_line_report_134" model="account.tax.report.line">
            <field name="name">Importaciones liquidadas de materias primas, insumos y bienes de capital que sean
                incorporadas en procesos productivos de bienes que se exporten
            </field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_133"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_700" model="account.tax.report.line">
            <field name="name">Valor(700)</field>
            <field name="code">c700</field>
            <field name="tag_name">700 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_134"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_701" model="account.tax.report.line">
            <field name="name">ISD pagado(701)</field>
            <field name="code">c701</field>
            <field name="tag_name">701 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_134"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_137" model="account.tax.report.line">
            <field name="name">AGENTE DE RETENCIÓN DEL IMPUESTO AL VALOR AGREGADO</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_721" model="account.tax.report.line">
            <field name="name">Retención del 10%(721)</field>
            <field name="code">c721</field>
            <field name="tag_name">721 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_137"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_723" model="account.tax.report.line">
            <field name="name">Retención del 20%(723)</field>
            <field name="code">c723</field>
            <field name="tag_name">723 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_137"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_725" model="account.tax.report.line">
            <field name="name">Retención del 30%(725)</field>
            <field name="code">c725</field>
            <field name="tag_name">725 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_137"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_727" model="account.tax.report.line">
            <field name="name">Retención del 50%(727)</field>
            <field name="code">c727</field>
            <field name="tag_name">727 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_137"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_729" model="account.tax.report.line">
            <field name="name">Retención del 70%(729)</field>
            <field name="code">c729</field>
            <field name="tag_name">729 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_137"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_104_731" model="account.tax.report.line">
            <field name="name">Retención del 100%(731)</field>
            <field name="code">c731</field>
            <field name="tag_name">731 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_137"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_144" model="account.tax.report.line">
            <field name="name">TOTAL IMPUESTO RETENIDO</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_137"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_799" model="account.tax.report.line">
            <field name="name">(799)</field>
            <field name="code">c799</field>
            <field name="tag_name">799 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_144"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_146" model="account.tax.report.line">
            <field name="name">Devolución provisional de IVA mediante compensación con retenciones efectuadas</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_137"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_800" model="account.tax.report.line">
            <field name="name">(800)</field>
            <field name="code">c800</field>
            <field name="tag_name">800 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_146"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_148" model="account.tax.report.line">
            <field name="name">TOTAL IMPUESTO A PAGAR POR RETENCIÓN</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_137"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_104_801" model="account.tax.report.line">
            <field name="name">(801)</field>
            <field name="code">c801</field>
            <field name="tag_name">801 (Reporte 104)</field>
            <field name="report_id" ref="tax_report_104"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_148"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_103" model="account.tax.report">
            <field name="name">103</field>
            <field name="country_id" ref="base.ec"/>
        </record>
        <record id="tax_report_line_parent_line_report_150" model="account.tax.report.line">
            <field name="name">POR PAGOS EFECTUADOS A RESIDENTES Y ESTABLECIMIENTOS PERMANENTES</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>
        <record id="tax_report_line_parent_line_report_151" model="account.tax.report.line">
            <field name="name">En relación de dependencia que supera o no la base desgravada</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_150"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_302" model="account.tax.report.line">
            <field name="name">Base imponible(302)</field>
            <field name="code">c302</field>
            <field name="tag_name">302 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_352" model="account.tax.report.line">
            <field name="name">Valor retenido(352)</field>
            <field name="code">c352</field>
            <field name="tag_name">352 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_154" model="account.tax.report.line">
            <field name="name">Servicios</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>
        <record id="tax_report_line_parent_line_report_155" model="account.tax.report.line">
            <field name="name">Honorarios profesionales</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_154"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_303" model="account.tax.report.line">
            <field name="name">Base imponible(303)</field>
            <field name="code">c303</field>
            <field name="tag_name">303 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_155"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_353" model="account.tax.report.line">
            <field name="name">Valor retenido(353)</field>
            <field name="code">c353</field>
            <field name="tag_name">353 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_155"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_158" model="account.tax.report.line">
            <field name="name">Predomina el intelecto</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_154"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_304" model="account.tax.report.line">
            <field name="name">Base imponible(304)</field>
            <field name="code">c304</field>
            <field name="tag_name">304 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_158"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_354" model="account.tax.report.line">
            <field name="name">Valor retenido(354)</field>
            <field name="code">c354</field>
            <field name="tag_name">354 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_158"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_161" model="account.tax.report.line">
            <field name="name">Predomina la mano de obra</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_154"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_307" model="account.tax.report.line">
            <field name="name">Base imponible(307)</field>
            <field name="code">c307</field>
            <field name="tag_name">307 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_161"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_357" model="account.tax.report.line">
            <field name="name">Valor retenido(357)</field>
            <field name="code">c357</field>
            <field name="tag_name">357 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_161"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_164" model="account.tax.report.line">
            <field name="name">Utilización o aprovechamiento de la imagen o renombre</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_154"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_308" model="account.tax.report.line">
            <field name="name">Base imponible(308)</field>
            <field name="code">c308</field>
            <field name="tag_name">308 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_164"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_358" model="account.tax.report.line">
            <field name="name">Valor retenido(358)</field>
            <field name="code">c358</field>
            <field name="tag_name">358 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_164"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_167" model="account.tax.report.line">
            <field name="name">Publicidad y comunicación</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_154"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_309" model="account.tax.report.line">
            <field name="name">Base imponible(309)</field>
            <field name="code">c309</field>
            <field name="tag_name">309 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_167"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_359" model="account.tax.report.line">
            <field name="name">Valor retenido(359)</field>
            <field name="code">c359</field>
            <field name="tag_name">359 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_167"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_170" model="account.tax.report.line">
            <field name="name">Transporte privado de pasajeros o servicio público o privado de carga</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_154"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_310" model="account.tax.report.line">
            <field name="name">Base imponible(310)</field>
            <field name="code">c310</field>
            <field name="tag_name">310 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_170"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_360" model="account.tax.report.line">
            <field name="name">Valor retenido(360)</field>
            <field name="code">c360</field>
            <field name="tag_name">360 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_170"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_173" model="account.tax.report.line">
            <field name="name">A través de liquidaciones de compra (nivel cultural o rusticidad)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_311" model="account.tax.report.line">
            <field name="name">Base imponible(311)</field>
            <field name="code">c311</field>
            <field name="tag_name">311 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_173"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_361" model="account.tax.report.line">
            <field name="name">Valor retenido(361)</field>
            <field name="code">c361</field>
            <field name="tag_name">361 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_173"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_176" model="account.tax.report.line">
            <field name="name">Transferencia de bienes muebles de naturaleza corporal</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_312" model="account.tax.report.line">
            <field name="name">Base imponible(312)</field>
            <field name="code">c312</field>
            <field name="tag_name">312 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_176"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_362" model="account.tax.report.line">
            <field name="name">Valor retenido(362)</field>
            <field name="code">c362</field>
            <field name="tag_name">362 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_176"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_179" model="account.tax.report.line">
            <field name="name">Compra de bienes de origen agrícola, avícola, pecuario, apícola, cunícula, bioacuático,
                forestal y carnes en estado natural
            </field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_3210" model="account.tax.report.line">
            <field name="name">Base imponible(3210)</field>
            <field name="code">c3210</field>
            <field name="tag_name">3210 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_179"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_3620" model="account.tax.report.line">
            <field name="name">Valor retenido(3620)</field>
            <field name="code">c3620</field>
            <field name="tag_name">3620 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_179"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_182" model="account.tax.report.line">
            <field name="name">Por regalías, derechos de autor, marcas, patentes y similares</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_314" model="account.tax.report.line">
            <field name="name">Base imponible(314)</field>
            <field name="code">c314</field>
            <field name="tag_name">314 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_182"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_364" model="account.tax.report.line">
            <field name="name">Valor retenido(364)</field>
            <field name="code">c364</field>
            <field name="tag_name">364 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_182"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_185" model="account.tax.report.line">
            <field name="name">Arrendamiento</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>
        <record id="tax_report_line_parent_line_report_186" model="account.tax.report.line">
            <field name="name">Mercantil</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_185"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_319" model="account.tax.report.line">
            <field name="name">Base imponible(319)</field>
            <field name="code">c319</field>
            <field name="tag_name">319 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_186"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_369" model="account.tax.report.line">
            <field name="name">Valor retenido(369)</field>
            <field name="code">c369</field>
            <field name="tag_name">369 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_186"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_189" model="account.tax.report.line">
            <field name="name">Bienes inmuebles</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_185"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_320" model="account.tax.report.line">
            <field name="name">Base imponible(320)</field>
            <field name="code">c320</field>
            <field name="tag_name">320 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_189"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_370" model="account.tax.report.line">
            <field name="name">Valor retenido(370)</field>
            <field name="code">c370</field>
            <field name="tag_name">370 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_189"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_192" model="account.tax.report.line">
            <field name="name">Seguros y reaseguros (primas y cesiones)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_322" model="account.tax.report.line">
            <field name="name">Base imponible(322)</field>
            <field name="code">c322</field>
            <field name="tag_name">322 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_192"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_372" model="account.tax.report.line">
            <field name="name">Valor retenido(372)</field>
            <field name="code">c372</field>
            <field name="tag_name">372 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_192"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_195" model="account.tax.report.line">
            <field name="name">Rendimientos financieros</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_323" model="account.tax.report.line">
            <field name="name">Base imponible(323)</field>
            <field name="code">c323</field>
            <field name="tag_name">323 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_195"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_373" model="account.tax.report.line">
            <field name="name">Valor retenido(373)</field>
            <field name="code">c373</field>
            <field name="tag_name">373 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_195"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_198" model="account.tax.report.line">
            <field name="name">Rendimientos financieros entre instituciones del sistema financiero y entidades economía
                popular y solidaria
            </field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_324" model="account.tax.report.line">
            <field name="name">Base imponible(324)</field>
            <field name="code">c324</field>
            <field name="tag_name">324 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_198"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_374" model="account.tax.report.line">
            <field name="name">Valor retenido(374)</field>
            <field name="code">c374</field>
            <field name="tag_name">374 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_198"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_201" model="account.tax.report.line">
            <field name="name">Anticipo dividendos</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_325" model="account.tax.report.line">
            <field name="name">Base imponible(325)</field>
            <field name="code">c325</field>
            <field name="tag_name">325 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_201"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_375" model="account.tax.report.line">
            <field name="name">Valor retenido(375)</field>
            <field name="code">c375</field>
            <field name="tag_name">375 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_201"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_204" model="account.tax.report.line">
            <field name="name">Dividendos distribuidos que correspondan al impuesto a la renta único establecido en el
                art. 27 de la LRTI
            </field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_326" model="account.tax.report.line">
            <field name="name">Base imponible(326)</field>
            <field name="code">c326</field>
            <field name="tag_name">326 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_204"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_376" model="account.tax.report.line">
            <field name="name">Valor retenido(376)</field>
            <field name="code">c376</field>
            <field name="tag_name">376 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_204"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_207" model="account.tax.report.line">
            <field name="name">Dividendos distribuidos a personas naturales residentes</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_327" model="account.tax.report.line">
            <field name="name">Base imponible(327)</field>
            <field name="code">c327</field>
            <field name="tag_name">327 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_207"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_377" model="account.tax.report.line">
            <field name="name">Valor retenido(377)</field>
            <field name="code">c377</field>
            <field name="tag_name">377 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_207"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_210" model="account.tax.report.line">
            <field name="name">Dividendos distribuidos a sociedades residentes</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_328" model="account.tax.report.line">
            <field name="name">Base imponible(328)</field>
            <field name="code">c328</field>
            <field name="tag_name">328 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_210"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_378" model="account.tax.report.line">
            <field name="name">Valor retenido(378)</field>
            <field name="code">c378</field>
            <field name="tag_name">378 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_210"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_213" model="account.tax.report.line">
            <field name="name">Dividendos distribuidos a fideicomisos residentes</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_329" model="account.tax.report.line">
            <field name="name">Base imponible(329)</field>
            <field name="code">c329</field>
            <field name="tag_name">329 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_213"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_379" model="account.tax.report.line">
            <field name="name">Valor retenido(379)</field>
            <field name="code">c379</field>
            <field name="tag_name">379 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_213"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_216" model="account.tax.report.line">
            <field name="name">Dividendos en acciones (capitalización de utilidades)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_331" model="account.tax.report.line">
            <field name="name">Base imponible(331)</field>
            <field name="code">c331</field>
            <field name="tag_name">331 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_216"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_218" model="account.tax.report.line">
            <field name="name">Pagos de bienes y servicios no sujetos a retención</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_332" model="account.tax.report.line">
            <field name="name">Base imponible(332)</field>
            <field name="code">c332</field>
            <field name="tag_name">332 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_218"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_220" model="account.tax.report.line">
            <field name="name">Enajenación de derechos representativos de capital y otros derechos cotizados en bolsa
                ecuatoriana
            </field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_333" model="account.tax.report.line">
            <field name="name">Base imponible(333)</field>
            <field name="code">c333</field>
            <field name="tag_name">333 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_220"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_383" model="account.tax.report.line">
            <field name="name">Valor retenido(383)</field>
            <field name="code">c383</field>
            <field name="tag_name">383 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_220"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_223" model="account.tax.report.line">
            <field name="name">Enajenación de derechos representativos de capital y otros derechos no cotizados en bolsa
                ecuatoriana
            </field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_334" model="account.tax.report.line">
            <field name="name">Base imponible(334)</field>
            <field name="code">c334</field>
            <field name="tag_name">334 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_223"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_384" model="account.tax.report.line">
            <field name="name">Valor retenido(384)</field>
            <field name="code">c384</field>
            <field name="tag_name">384 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_223"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_226" model="account.tax.report.line">
            <field name="name">Loterías, rifas, apuestas y similares</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_151"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_335" model="account.tax.report.line">
            <field name="name">Base imponible(335)</field>
            <field name="code">c335</field>
            <field name="tag_name">335 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_226"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_385" model="account.tax.report.line">
            <field name="name">Valor retenido(385)</field>
            <field name="code">c385</field>
            <field name="tag_name">385 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_226"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_229" model="account.tax.report.line">
            <field name="name">Venta de combustibles</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_150"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>
        <record id="tax_report_line_parent_line_report_230" model="account.tax.report.line">
            <field name="name">A comercializadoras</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_229"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_336" model="account.tax.report.line">
            <field name="name">Base imponible(336)</field>
            <field name="code">c336</field>
            <field name="tag_name">336 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_230"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_386" model="account.tax.report.line">
            <field name="name">Valor retenido(386)</field>
            <field name="code">c386</field>
            <field name="tag_name">386 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_230"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_233" model="account.tax.report.line">
            <field name="name">A distribuidores</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_229"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_337" model="account.tax.report.line">
            <field name="name">Base imponible(337)</field>
            <field name="code">c337</field>
            <field name="tag_name">337 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_233"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_387" model="account.tax.report.line">
            <field name="name">Valor retenido(387)</field>
            <field name="code">c387</field>
            <field name="tag_name">387 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_233"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_236" model="account.tax.report.line">
            <field name="name">Producción y venta local de banano producido o no por el mismo sujeto pasivo</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_150"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_3380" model="account.tax.report.line">
            <field name="name">Base imponible(3380)</field>
            <field name="code">c3380</field>
            <field name="tag_name">3380 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_236"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_3880" model="account.tax.report.line">
            <field name="name">Valor retenido(3880)</field>
            <field name="code">c3880</field>
            <field name="tag_name">3880 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_236"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_239" model="account.tax.report.line">
            <field name="name">Impuesto único a la exportación de banano</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_150"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_3400" model="account.tax.report.line">
            <field name="name">Base imponible(3400)</field>
            <field name="code">c3400</field>
            <field name="tag_name">3400 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_239"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_3900" model="account.tax.report.line">
            <field name="name">Valor retenido(3900)</field>
            <field name="code">c3900</field>
            <field name="tag_name">3900 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_239"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_242" model="account.tax.report.line">
            <field name="name">Otras retenciones</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_150"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>
        <record id="tax_report_line_parent_line_report_243" model="account.tax.report.line">
            <field name="name">Aplicables el 1%</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_242"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_343" model="account.tax.report.line">
            <field name="name">Base imponible(343)</field>
            <field name="code">c343</field>
            <field name="tag_name">343 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_243"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_393" model="account.tax.report.line">
            <field name="name">Valor retenido(393)</field>
            <field name="code">c393</field>
            <field name="tag_name">393 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_243"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_246" model="account.tax.report.line">
            <field name="name">Aplicables el 2%</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_242"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_344" model="account.tax.report.line">
            <field name="name">Base imponible(344)</field>
            <field name="code">c344</field>
            <field name="tag_name">344 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_246"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_394" model="account.tax.report.line">
            <field name="name">Valor retenido(394)</field>
            <field name="code">c394</field>
            <field name="tag_name">394 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_246"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_249" model="account.tax.report.line">
            <field name="name">Aplicables el 2,75%</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_242"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_3440" model="account.tax.report.line">
            <field name="name">Base imponible(3440)</field>
            <field name="code">c3440</field>
            <field name="tag_name">3440 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_249"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_3940" model="account.tax.report.line">
            <field name="name">Valor retenido(3940)</field>
            <field name="code">c3940</field>
            <field name="tag_name">3940 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_249"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_252" model="account.tax.report.line">
            <field name="name">Aplicables el 8%</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_242"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_345" model="account.tax.report.line">
            <field name="name">Base imponible(345)</field>
            <field name="code">c345</field>
            <field name="tag_name">345 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_252"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_395" model="account.tax.report.line">
            <field name="name">Valor retenido(395)</field>
            <field name="code">c395</field>
            <field name="tag_name">395 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_252"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_255" model="account.tax.report.line">
            <field name="name">Aplicables a otros porcentajes (incluye régimen Microempresarial)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_242"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_346" model="account.tax.report.line">
            <field name="name">Base imponible(346)</field>
            <field name="code">c346</field>
            <field name="tag_name">346 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_255"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_396" model="account.tax.report.line">
            <field name="name">Valor retenido(396)</field>
            <field name="code">c396</field>
            <field name="tag_name">396 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_255"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_258" model="account.tax.report.line">
            <field name="name">Impuesto único a ingresos provenientes de actividades agropecuarias en etapa de
                producción / comercialización local o exportación
            </field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_150"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_348" model="account.tax.report.line">
            <field name="name">Base imponible(348)</field>
            <field name="code">c348</field>
            <field name="tag_name">348 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_258"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_398" model="account.tax.report.line">
            <field name="name">Valor retenido(398)</field>
            <field name="code">c398</field>
            <field name="tag_name">398 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_258"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_261" model="account.tax.report.line">
            <field name="name">Otras autoretenciones</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_150"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_350" model="account.tax.report.line">
            <field name="name">Base imponible(350)</field>
            <field name="code">c350</field>
            <field name="tag_name">350 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_261"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_400" model="account.tax.report.line">
            <field name="name">Valor retenido(400)</field>
            <field name="code">c400</field>
            <field name="tag_name">400 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_261"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_264" model="account.tax.report.line">
            <field name="name">SUBTOTAL OPERACIONES EFECTUADAS EN EL PAÍS</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_150"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_349" model="account.tax.report.line">
            <field name="name">Base imponible(349)</field>
            <field name="code">c349</field>
            <field name="tag_name">349 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_264"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_399" model="account.tax.report.line">
            <field name="name">Valor retenido(399)</field>
            <field name="code">c399</field>
            <field name="tag_name">399 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_264"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_267" model="account.tax.report.line">
            <field name="name">POR PAGOS A NO RESIDENTES</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>
        <record id="tax_report_line_parent_line_report_268" model="account.tax.report.line">
            <field name="name">Con convenio de doble tributación</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_267"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>
        <record id="tax_report_line_parent_line_report_269" model="account.tax.report.line">
            <field name="name">Intereses por financiamiento de proveedores</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_268"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_402" model="account.tax.report.line">
            <field name="name">Base imponible(402)</field>
            <field name="code">c402</field>
            <field name="tag_name">402 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_269"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_452" model="account.tax.report.line">
            <field name="name">Valor retenido(452)</field>
            <field name="code">c452</field>
            <field name="tag_name">452 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_269"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_272" model="account.tax.report.line">
            <field name="name">Intereses de créditos</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_268"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_403" model="account.tax.report.line">
            <field name="name">Base imponible(403)</field>
            <field name="code">c403</field>
            <field name="tag_name">403 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_272"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_453" model="account.tax.report.line">
            <field name="name">Valor retenido(453)</field>
            <field name="code">c453</field>
            <field name="tag_name">453 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_272"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_275" model="account.tax.report.line">
            <field name="name">Anticipo de dividendos</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_268"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_404" model="account.tax.report.line">
            <field name="name">Base imponible(404)</field>
            <field name="code">c404</field>
            <field name="tag_name">404 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_275"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_454" model="account.tax.report.line">
            <field name="name">Valor retenido(454)</field>
            <field name="code">c454</field>
            <field name="tag_name">454 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_275"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_278" model="account.tax.report.line">
            <field name="name">Dividendos sin beneficiario efectivo persona natural residente en Ecuador</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_268"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_4050" model="account.tax.report.line">
            <field name="name">Base imponible(4050)</field>
            <field name="code">c4050</field>
            <field name="tag_name">4050 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_278"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_4550" model="account.tax.report.line">
            <field name="name">Valor retenido(4550)</field>
            <field name="code">c4550</field>
            <field name="tag_name">4550 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_278"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_281" model="account.tax.report.line">
            <field name="name">Dividendos con beneficiario efectivo persona natural residente en Ecuador</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_268"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_4060" model="account.tax.report.line">
            <field name="name">Base imponible(4060)</field>
            <field name="code">c4060</field>
            <field name="tag_name">4060 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_281"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_4560" model="account.tax.report.line">
            <field name="name">Valor retenido(4560)</field>
            <field name="code">c4560</field>
            <field name="tag_name">4560 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_281"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_284" model="account.tax.report.line">
            <field name="name">Dividendos incumpliendo el deber de informar la composición societaria</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_268"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_4070" model="account.tax.report.line">
            <field name="name">Base imponible(4070)</field>
            <field name="code">c4070</field>
            <field name="tag_name">4070 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_284"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_4570" model="account.tax.report.line">
            <field name="name">Valor retenido(4570)</field>
            <field name="code">c4570</field>
            <field name="tag_name">4570 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_284"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_287" model="account.tax.report.line">
            <field name="name">Enajenación de derechos representativos de capital y otros derechos</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_268"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_408" model="account.tax.report.line">
            <field name="name">Base imponible(408)</field>
            <field name="code">c408</field>
            <field name="tag_name">408 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_287"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_458" model="account.tax.report.line">
            <field name="name">Valor retenido(458)</field>
            <field name="code">c458</field>
            <field name="tag_name">458 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_287"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_290" model="account.tax.report.line">
            <field name="name">Seguros y reaseguros (primas y cesiones)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_268"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_409" model="account.tax.report.line">
            <field name="name">Base imponible(409)</field>
            <field name="code">c409</field>
            <field name="tag_name">409 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_290"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_459" model="account.tax.report.line">
            <field name="name">Valor retenido(459)</field>
            <field name="code">c459</field>
            <field name="tag_name">459 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_290"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_293" model="account.tax.report.line">
            <field name="name">Servicios técnicos, administrativos o de consultoría y regalías</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_268"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_410" model="account.tax.report.line">
            <field name="name">Base imponible(410)</field>
            <field name="code">c410</field>
            <field name="tag_name">410 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_293"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_460" model="account.tax.report.line">
            <field name="name">Valor retenido(460)</field>
            <field name="code">c460</field>
            <field name="tag_name">460 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_293"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_296" model="account.tax.report.line">
            <field name="name">Otros conceptos de ingresos gravados</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_268"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_411" model="account.tax.report.line">
            <field name="name">Base imponible(411)</field>
            <field name="code">c411</field>
            <field name="tag_name">411 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_296"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_461" model="account.tax.report.line">
            <field name="name">Valor retenido(461)</field>
            <field name="code">c461</field>
            <field name="tag_name">461 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_296"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_299" model="account.tax.report.line">
            <field name="name">Otros pagos al exterior no sujetos a retención</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_268"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_412" model="account.tax.report.line">
            <field name="name">Base imponible(412)</field>
            <field name="code">c412</field>
            <field name="tag_name">412 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_299"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_301" model="account.tax.report.line">
            <field name="name">Sin convenio de doble tributación</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_267"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>
        <record id="tax_report_line_parent_line_report_302" model="account.tax.report.line">
            <field name="name">Intereses por financiamiento de proveedores</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_301"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_413" model="account.tax.report.line">
            <field name="name">Base imponible(413)</field>
            <field name="code">c413</field>
            <field name="tag_name">413 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_302"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_463" model="account.tax.report.line">
            <field name="name">Valor retenido(463)</field>
            <field name="code">c463</field>
            <field name="tag_name">463 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_302"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_305" model="account.tax.report.line">
            <field name="name">Intereses de créditos</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_301"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_414" model="account.tax.report.line">
            <field name="name">Base imponible(414)</field>
            <field name="code">c414</field>
            <field name="tag_name">414 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_305"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_464" model="account.tax.report.line">
            <field name="name">Valor retenido(464)</field>
            <field name="code">c464</field>
            <field name="tag_name">464 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_305"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_308" model="account.tax.report.line">
            <field name="name">Anticipo de dividendos</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_301"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_415" model="account.tax.report.line">
            <field name="name">Base imponible(415)</field>
            <field name="code">c415</field>
            <field name="tag_name">415 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_308"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_465" model="account.tax.report.line">
            <field name="name">Valor retenido(465)</field>
            <field name="code">c465</field>
            <field name="tag_name">465 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_308"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_311" model="account.tax.report.line">
            <field name="name">Dividendos sin beneficiario efectivo persona natural residente en Ecuador</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_301"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_4160" model="account.tax.report.line">
            <field name="name">Base imponible(4160)</field>
            <field name="code">c4160</field>
            <field name="tag_name">4160 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_311"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_4660" model="account.tax.report.line">
            <field name="name">Valor retenido(4660)</field>
            <field name="code">c4660</field>
            <field name="tag_name">4660 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_311"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_314" model="account.tax.report.line">
            <field name="name">Dividendos con beneficiario efectivo persona natural residente en Ecuador</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_301"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_4170" model="account.tax.report.line">
            <field name="name">Base imponible(4170)</field>
            <field name="code">c4170</field>
            <field name="tag_name">4170 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_314"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_4670" model="account.tax.report.line">
            <field name="name">Valor retenido(4670)</field>
            <field name="code">c4670</field>
            <field name="tag_name">4670 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_314"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_317" model="account.tax.report.line">
            <field name="name">Dividendos incumpliendo el deber de informar la composición societaria</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_301"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_4180" model="account.tax.report.line">
            <field name="name">Base imponible(4180)</field>
            <field name="code">c4180</field>
            <field name="tag_name">4180 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_317"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_4680" model="account.tax.report.line">
            <field name="name">Valor retenido(4680)</field>
            <field name="code">c4680</field>
            <field name="tag_name">4680 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_317"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_320" model="account.tax.report.line">
            <field name="name">Enajenación de derechos representativos de capital y otros derechos</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_301"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_419" model="account.tax.report.line">
            <field name="name">Base imponible(419)</field>
            <field name="code">c419</field>
            <field name="tag_name">419 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_320"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_469" model="account.tax.report.line">
            <field name="name">Valor retenido(469)</field>
            <field name="code">c469</field>
            <field name="tag_name">469 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_320"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_323" model="account.tax.report.line">
            <field name="name">Seguros y reaseguros (primas y cesiones)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_301"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_420" model="account.tax.report.line">
            <field name="name">Base imponible(420)</field>
            <field name="code">c420</field>
            <field name="tag_name">420 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_323"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_470" model="account.tax.report.line">
            <field name="name">Valor retenido(470)</field>
            <field name="code">c470</field>
            <field name="tag_name">470 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_323"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_326" model="account.tax.report.line">
            <field name="name">Servicios técnicos, administrativos o de consultoría y regalías</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_301"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_421" model="account.tax.report.line">
            <field name="name">Base imponible(421)</field>
            <field name="code">c421</field>
            <field name="tag_name">421 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_326"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_471" model="account.tax.report.line">
            <field name="name">Valor retenido(471)</field>
            <field name="code">c471</field>
            <field name="tag_name">471 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_326"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_329" model="account.tax.report.line">
            <field name="name">Otros conceptos de ingresos gravados</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_301"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_422" model="account.tax.report.line">
            <field name="name">Base imponible(422)</field>
            <field name="code">c422</field>
            <field name="tag_name">422 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_329"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_472" model="account.tax.report.line">
            <field name="name">Valor retenido(472)</field>
            <field name="code">c472</field>
            <field name="tag_name">472 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_329"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_332" model="account.tax.report.line">
            <field name="name">Otros pagos al exterior no sujetos a retención</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_301"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_423" model="account.tax.report.line">
            <field name="name">Base imponible(423)</field>
            <field name="code">c423</field>
            <field name="tag_name">423 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_332"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_334" model="account.tax.report.line">
            <field name="name">En paraísos fiscales o regímenes fiscales preferentes</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_267"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>
        <record id="tax_report_line_parent_line_report_335" model="account.tax.report.line">
            <field name="name">Intereses</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_334"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_424" model="account.tax.report.line">
            <field name="name">Base imponible(424)</field>
            <field name="code">c424</field>
            <field name="tag_name">424 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_335"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_474" model="account.tax.report.line">
            <field name="name">Valor retenido(474)</field>
            <field name="code">c474</field>
            <field name="tag_name">474 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_335"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_338" model="account.tax.report.line">
            <field name="name">Anticipo de dividendos</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_334"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_425" model="account.tax.report.line">
            <field name="name">Base imponible(425)</field>
            <field name="code">c425</field>
            <field name="tag_name">425 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_338"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_475" model="account.tax.report.line">
            <field name="name">Valor retenido(475)</field>
            <field name="code">c475</field>
            <field name="tag_name">475 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_338"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_341" model="account.tax.report.line">
            <field name="name">Dividendos sin beneficiario efectivo persona natural residente en Ecuador</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_334"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_4260" model="account.tax.report.line">
            <field name="name">Base imponible(4260)</field>
            <field name="code">c4260</field>
            <field name="tag_name">4260 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_341"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_4760" model="account.tax.report.line">
            <field name="name">Valor retenido(4760)</field>
            <field name="code">c4760</field>
            <field name="tag_name">4760 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_341"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_344" model="account.tax.report.line">
            <field name="name">Dividendos con beneficiario efectivo persona natural residente en Ecuador</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_334"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_4270" model="account.tax.report.line">
            <field name="name">Base imponible(4270)</field>
            <field name="code">c4270</field>
            <field name="tag_name">4270 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_344"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_4770" model="account.tax.report.line">
            <field name="name">Valor retenido(4770)</field>
            <field name="code">c4770</field>
            <field name="tag_name">4770 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_344"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_347" model="account.tax.report.line">
            <field name="name">Dividendos incumpliendo el deber de informar la composición societaria</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_334"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_4280" model="account.tax.report.line">
            <field name="name">Base imponible(4280)</field>
            <field name="code">c4280</field>
            <field name="tag_name">4280 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_347"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_4780" model="account.tax.report.line">
            <field name="name">Valor retenido(4780)</field>
            <field name="code">c4780</field>
            <field name="tag_name">4780 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_347"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_350" model="account.tax.report.line">
            <field name="name">Enajenación de derechos representativos de capital y otros derechos</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_334"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_429" model="account.tax.report.line">
            <field name="name">Base imponible(429)</field>
            <field name="code">c429</field>
            <field name="tag_name">429 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_350"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_479" model="account.tax.report.line">
            <field name="name">Valor retenido(479)</field>
            <field name="code">c479</field>
            <field name="tag_name">479 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_350"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_353" model="account.tax.report.line">
            <field name="name">Seguros y reaseguros (primas y cesiones)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_334"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_430" model="account.tax.report.line">
            <field name="name">Base imponible(430)</field>
            <field name="code">c430</field>
            <field name="tag_name">430 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_353"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_480" model="account.tax.report.line">
            <field name="name">Valor retenido(480)</field>
            <field name="code">c480</field>
            <field name="tag_name">480 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_353"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_356" model="account.tax.report.line">
            <field name="name">Servicios técnicos, administrativos o de consultoría y regalías</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_334"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_431" model="account.tax.report.line">
            <field name="name">Base imponible(431)</field>
            <field name="code">c431</field>
            <field name="tag_name">431 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_356"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_481" model="account.tax.report.line">
            <field name="name">Valor retenido(481)</field>
            <field name="code">c481</field>
            <field name="tag_name">481 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_356"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_359" model="account.tax.report.line">
            <field name="name">Otros conceptos de ingresos gravados</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_334"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_432" model="account.tax.report.line">
            <field name="name">Base imponible(432)</field>
            <field name="code">c432</field>
            <field name="tag_name">432 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_359"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_482" model="account.tax.report.line">
            <field name="name">Valor retenido(482)</field>
            <field name="code">c482</field>
            <field name="tag_name">482 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_359"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_362" model="account.tax.report.line">
            <field name="name">Otros pagos al exterior no sujetos a retención</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_334"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_433" model="account.tax.report.line">
            <field name="name">Base imponible(433)</field>
            <field name="code">c433</field>
            <field name="tag_name">433 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_362"/>
            <field name="sequence">1</field>
        </record>
        <record id="tax_report_line_parent_line_report_364" model="account.tax.report.line">
            <field name="name">SUBTOTAL OPERACIONES EFECTUADAS CON EL EXTERIOR</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="sequence">1</field>
            <field name="formula">None</field>
        </record>

        <record id="tax_report_line_103_487" model="account.tax.report.line">
            <field name="name">Base imponible(487)</field>
            <field name="code">c487</field>
            <field name="tag_name">487 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_364"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_103_498" model="account.tax.report.line">
            <field name="name">Valor retenido(498)</field>
            <field name="code">c498</field>
            <field name="tag_name">498 (Reporte 103)</field>
            <field name="report_id" ref="tax_report_103"/>
            <field name="parent_id" ref="tax_report_line_parent_line_report_364"/>
            <field name="sequence">1</field>
        </record>

    </data>
</odoo>
```

## File: data\account_tax_template_vat_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data >
        <!-- 
        VAT TAXES *IMPUESTO AL VALOR AGREGADO
        -->
        <record id="tax_vat_411" model="account.tax.template">
            <!-- IVA EN VENTAS LOCALES (EXCLUYE ACTIVOS FIJOS) -->
            <field name="name">Iva 12%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">9</field>
            <field name="amount">12.0</field>
            <field name="description">IVA 12%</field>
            <field name="l10n_ec_code_base">411</field>
            <field name="l10n_ec_code_applied">421</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_411')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat'),
                    'plus_report_line_ids': [ref('tax_report_line_104_421')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_411')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat'),
                    'minus_report_line_ids': [ref('tax_report_line_104_421')],
                })]"/>
        </record>
        <record id="tax_vat_412" model="account.tax.template">
            <!-- IVA EN VENTAS DE ACTIVOS FIJOS -->
            <field name="name">Iva 12% (activos)</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">10</field>
            <field name="amount">12.0</field>
            <field name="description">IVA 12%</field>
            <field name="l10n_ec_code_base">412</field>
            <field name="l10n_ec_code_applied">422</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_412')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat'),
                    'plus_report_line_ids': [ref('tax_report_line_104_422')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_411')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat'),
                    'minus_report_line_ids': [ref('tax_report_line_104_421')],
                })]"/>
        </record>
        <record id="tax_vat_415" model="account.tax.template">
            <!-- IVA EN VENTAS LOCALES 0% (EXCLUYE ACTIVOS FIJOS) CON DERECHO A CREDITO TRIBUTARIO -->
            <field name="name">Iva 0%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">19</field>
            <field name="amount">0.0</field>
            <field name="description">IVA 0%</field>
            <field name="l10n_ec_code_base">415</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_415')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_zero'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_415')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_zero'),
                })]"/>
        </record>
        <record id="tax_vat_416" model="account.tax.template">
            <!-- IVA VENTAS DE ACTIVOS FIJOS GRAVADAS TARIFA 0% CON DERECHO A CREDITO TRIBUTARIO -->
            <field name="name">Iva 0% (activos)</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">20</field>
            <field name="amount">0.0</field>
            <field name="description">IVA 0%</field>
            <field name="l10n_ec_code_base">416</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_414')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_zero'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_414')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_zero'),
                })]"/>
        </record>
        <record id="tax_vat_413" model="account.tax.template">
            <!-- IVA EN VENTAS LOCALES 0% (EXCLUYE ACTIVOS FIJOS) SIN DERECHO A CREDITO TRIBUTARIO -->
            <field name="name">Iva 0% (sin crédito tributario)</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">20</field>
            <field name="amount">0.0</field>
            <field name="description">IVA 0%</field>
            <field name="l10n_ec_code_base">413</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_413')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_zero'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_413')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_zero'),
                })]"/>
        </record>
        <record id="tax_vat_414" model="account.tax.template">
            <!-- IVA EN VENTAS DE ACTIVOS FIJOS GRAVADAS 0% SIN DERECHO A CREDITO TRIBUTARIO -->
            <field name="name">Iva 0% (activos sin crédito tributario)</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">20</field>
            <field name="amount">0.0</field>
            <field name="description">IVA 0%</field>
            <field name="l10n_ec_code_base">414</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_414')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_zero'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_414')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_zero'),
                })]"/>
        </record>
        <record id="tax_vat_417" model="account.tax.template">
            <!-- IVA 0% POR EXPORTACIONES DE BIENES -->
            <field name="name">Iva 0% (exportación bienes)</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">21</field>
            <field name="amount">0.0</field>
            <field name="description">IVA 0%</field>
            <field name="l10n_ec_code_base">417</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_417')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_goods_exports'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_417')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_goods_exports'),
                })]"/>
        </record>
        <record id="tax_vat_418" model="account.tax.template">
            <!-- IVA POR EXPORTACIONES DE SERVICIOS -->
            <field name="name">Iva 0% (exportación servicios)</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">21</field>
            <field name="amount">0.0</field>
            <field name="description">IVA 0%</field>
            <field name="l10n_ec_code_base">418</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_418')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_services_exports'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_418')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_services_exports'),
                })]"/>
        </record>
        <record id="tax_vat_419" model="account.tax.template">
            <!-- TRANSFERENCIAS EN VENTAS NO OBJETO O EXENTAS DE IVA -->
            <field name="name">Iva 0% (no objeto/exentas)</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">40</field>
            <field name="amount">0.0</field>
            <field name="description">IVA EXENTO</field>
            <field name="l10n_ec_code_base">419</field>

            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat_excempt"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_419')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_zero'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_419')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_zero'),
                })]"/>
        </record>
        <record id="tax_vat_444" model="account.tax.template">
            <!-- IVA EN VENTAS POR REEMBOLSO COMO INTERMEDIARIO -->
            <field name="name">Iva 12% (reembolso)</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">10</field>
            <field name="amount">12.0</field>
            <field name="description">IVA 12%</field>
            <field name="l10n_ec_code_base">444</field>
            <field name="l10n_ec_code_applied">454</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_412')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_other_downpayments'),
                    'plus_report_line_ids': [ref('tax_report_line_104_422')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_411')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_other_downpayments'),
                    'minus_report_line_ids': [ref('tax_report_line_104_421')],
                })]"/>
        </record>
        <record id="tax_vat_510" model="account.tax.template">
            <!-- IVA EN COMPRAS LOCALES (EXCLUYE ACTIVOS FIJOS) CON DERECHO A CREDITO TRIBUTARIO -->
            <field name="name">Iva 12%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">9</field>
            <field name="amount">12.0</field>
            <field name="description">IVA 12%</field>
            <field name="l10n_ec_code_base">510</field>
            <field name="l10n_ec_code_applied">520</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_510')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat'),
                    'plus_report_line_ids': [ref('tax_report_line_104_520')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_510')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat'),
                    'minus_report_line_ids': [ref('tax_report_line_104_520')],
                })]"/>
        </record>
        <record id="tax_vat_511" model="account.tax.template">
            <!-- IVA EN COMPRAS LOCALES DE ACTIVOS FIJOS CON DERECHO A CREDITO TRIBUTARIO -->
            <field name="name">Iva 12% (activos)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">10</field>
            <field name="amount">12.0</field>
            <field name="description">IVA 12%</field>
            <field name="l10n_ec_code_base">511</field>
            <field name="l10n_ec_code_applied">521</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_511')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat_assets'),
                    'plus_report_line_ids': [ref('tax_report_line_104_521')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_511')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat_assets'),
                    'minus_report_line_ids': [ref('tax_report_line_104_521')],
                })]"/>
        </record>
        <record id="tax_vat_512" model="account.tax.template">
            <!-- IVA EN OTRAS ADQUISICIONES SIN DERECHO A CREDITO TRIBUTARIO -->
            <field name="name">Iva 12% (sin crédito tributario)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">10</field>
            <field name="amount">12.0</field>
            <field name="description">IVA 12%</field>
            <field name="l10n_ec_code_base">512</field>
            <field name="l10n_ec_code_applied">522</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_512')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'plus_report_line_ids': [ref('tax_report_line_104_522')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_512')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'minus_report_line_ids': [ref('tax_report_line_104_522')],
                })]"/>
        </record>
        <record id="tax_vat_513" model="account.tax.template">
            <!-- IVA EN IMPORTACIONES DE SERVICIOS -->
            <field name="name">Iva 12% (importación servicios)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">10</field>
            <field name="amount">12.0</field>
            <field name="description">IVA 12%</field>
            <field name="l10n_ec_code_base">513</field>
            <field name="l10n_ec_code_applied">523</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_513')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat_service_imports'),
                    'plus_report_line_ids': [ref('tax_report_line_104_523')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_513')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat_service_imports'),
                    'minus_report_line_ids': [ref('tax_report_line_104_523')],
                })]"/>
        </record>
        <record id="tax_vat_514" model="account.tax.template">
            <!-- IVA EN IMPORTACIONES DE BIENES (EXCLUYE ACTIVOS FIJOS) -->
            <field name="name">Iva 12% (importación bienes)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">10</field>
            <field name="amount">12.0</field>
            <field name="description">IVA 12%</field>
            <field name="l10n_ec_code_base">514</field>
            <field name="l10n_ec_code_applied">524</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_514')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat_goods_imports'),
                    'plus_report_line_ids': [ref('tax_report_line_104_524')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_514')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat_goods_imports'),
                    'minus_report_line_ids': [ref('tax_report_line_104_524')],
                })]"/>
        </record>
        <record id="tax_vat_515" model="account.tax.template">
            <!-- IVA EN IMPORTACIONES DE ACTIVOS FIJOS -->
            <field name="name">Iva 12% (importación activos)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">10</field>
            <field name="amount">12.0</field>
            <field name="description">IVA 12%</field>
            <field name="l10n_ec_code_base">515</field>
            <field name="l10n_ec_code_applied">525</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_515')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat_assets_imports'),
                    'plus_report_line_ids': [ref('tax_report_line_104_525')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_515')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat_assets_imports'),
                    'minus_report_line_ids': [ref('tax_report_line_104_525')],
                })]"/>
        </record>
        <record id="tax_vat_517" model="account.tax.template">
            <!-- IVA EN COMPRAS 0% (INCLUYE ACTIVOS FIJOS) -->
            <field name="name">Iva 0%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">20</field>
            <field name="amount">0.0</field>
            <field name="description">IVA 0%</field>
            <field name="l10n_ec_code_base">517</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_517')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat_zero'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_517')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat_zero'),
                })]"/>

        </record>
        <record id="tax_vat_516" model="account.tax.template">
            <!-- IVA 0% EN IMPORTACIONES DE BIENES (INCLUYE ACTIVOS FIJOS) -->
            <field name="name">Iva 0% (importación bienes)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">20</field>
            <field name="amount">0.0</field>
            <field name="description">IVA 0%</field>
            <field name="l10n_ec_code_base">516</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_516')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat_zero'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_516')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat_zero'),
                })]"/>
        </record>
        <record id="tax_vat_518" model="account.tax.template">
            <!-- ADQUISICIONES REALIZADAS A CONTRIBUYENTES RISE -->
            <field name="name">Iva 0% (rise)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">20</field>
            <field name="amount">0.0</field>
            <field name="description">IVA 0%</field>
            <field name="l10n_ec_code_base">518</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_518')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat_zero'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_518')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat_zero'),
                })]"/>
        </record>
        <record id="tax_vat_541" model="account.tax.template">
            <!-- ADQUISICIONES NO OBJETO DE IVA -->
            <field name="name">Iva 0% (no objeto de iva)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">30</field>
            <field name="amount">0.0</field>
            <field name="description">NO OBJETO DE IVA</field>
            <field name="l10n_ec_code_base">541</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat_not_charged"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_541')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat_zero'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_541')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat_zero'),
                })]"/>
        </record>
        <record id="tax_vat_542" model="account.tax.template">
            <!-- ADQUISICIONES EXENTAS DEL PAGO DE IVA -->
            <field name="name">Iva 0% (excento de iva)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">40</field>
            <field name="amount">0.0</field>
            <field name="description">IVA EXENTO</field>
            <field name="l10n_ec_code_base">542</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat_excempt"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_542')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat_zero'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_542')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_purchase_vat_zero'),
                })]"/>
        </record>
        <record id="tax_vat_545" model="account.tax.template">
            <!-- IVA EN COMPRAS POR REEMBOLSO COMO INTERMEDIARIO -->
            <field name="name">Iva 12% (reembolso intermediario)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">10</field>
            <field name="amount">12.0</field>
            <field name="description">IVA 12%</field>
            <field name="l10n_ec_code_base">545</field>
            <field name="l10n_ec_code_applied">555</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_vat_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_104_545')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_other_downpayments'),
                    'plus_report_line_ids': [ref('tax_report_line_104_555')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_104_545')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_other_downpayments'),
                    'minus_report_line_ids': [ref('tax_report_line_104_555')],
                })]"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_template_withhold_profit_data.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<odoo>
    <data>
        <!-- 
		PURCHASE WITHHOLDS OVER ANTICIPADED PROFIT *IMPUESTO A LA RENTA
		-->
        <record id="tax_withhold_profit_303" model="account.tax.template">
            <field name="name">303 10% honorarios profesionales y demas pagos por servicios relacionados con el titulo profesional</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-10.0</field>
            <field name="description">303</field>
            <field name="l10n_ec_code_applied">353</field>
            <field name="l10n_ec_code_base">303</field>
            <field name="l10n_ec_code_ats">303</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_303')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_10x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_353')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_303')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_10x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_353')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_304" model="account.tax.template">
            <field name="name">304 8% servicios predomina el intelecto no relacionados con el titulo profesional</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-8.0</field>
            <field name="description">304</field>
            <field name="l10n_ec_code_applied">354</field>
            <field name="l10n_ec_code_base">304</field>
            <field name="l10n_ec_code_ats">304</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_304')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_354')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_304')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_354')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_304A" model="account.tax.template">
            <field name="name">304a 8% comisiones y demas pagos por servicios predomina intelecto no relacionados con el titulo profesional</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-8.0</field>
            <field name="description">304</field>
            <field name="l10n_ec_code_applied">354</field>
            <field name="l10n_ec_code_base">304</field>
            <field name="l10n_ec_code_ats">304A</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_304')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_354')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_304')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_354')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_304B" model="account.tax.template">
            <field name="name">304b 8% pagos a notarios y registradores de la propiedad y mercantil por sus actividades ejercidas como tales</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-8.0</field>
            <field name="description">304</field>
            <field name="l10n_ec_code_applied">354</field>
            <field name="l10n_ec_code_base">304</field>
            <field name="l10n_ec_code_ats">304B</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_304')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_354')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_304')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_354')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_304C" model="account.tax.template">
            <field name="name">304c 8% pagos a deportistas, entrenadores, arbitros, miembros del cuerpo tecnico por sus actividades ejercidas como tales</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-8.0</field>
            <field name="description">304</field>
            <field name="l10n_ec_code_applied">354</field>
            <field name="l10n_ec_code_base">304</field>
            <field name="l10n_ec_code_ats">304C</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_304')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_354')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_304')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_354')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_304D" model="account.tax.template">
            <field name="name">304d 8% pagos a artistas por sus actividades ejercidas como tales</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-8.0</field>
            <field name="description">304</field>
            <field name="l10n_ec_code_applied">354</field>
            <field name="l10n_ec_code_base">304</field>
            <field name="l10n_ec_code_ats">304D</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_304')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_354')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_304')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_354')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_304E" model="account.tax.template">
            <field name="name">304e 8% honorarios y demas pagos por servicios de docencia</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-8.0</field>
            <field name="description">304</field>
            <field name="l10n_ec_code_applied">354</field>
            <field name="l10n_ec_code_base">304</field>
            <field name="l10n_ec_code_ats">304E</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_304')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_354')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_304')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_354')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_307" model="account.tax.template">
            <field name="name">307 2% servicios predomina mano de obra</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-2.0</field>
            <field name="description">307</field>
            <field name="l10n_ec_code_applied">357</field>
            <field name="l10n_ec_code_base">307</field>
            <field name="l10n_ec_code_ats">307</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_307')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_2x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_357')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_307')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_2x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_357')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_308" model="account.tax.template">
            <field name="name">308 10% utilizacion o aprovechamiento de la imagen o renombre</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-10.0</field>
            <field name="description">308</field>
            <field name="l10n_ec_code_applied">358</field>
            <field name="l10n_ec_code_base">308</field>
            <field name="l10n_ec_code_ats">308</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_308')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_10x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_358')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_308')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_10x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_358')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_309" model="account.tax.template">
            <field name="name">309 1.75% servicios prestados por medios de comunicación y agencias de publicidad</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-1.75</field>
            <field name="description">309</field>
            <field name="l10n_ec_code_applied">359</field>
            <field name="l10n_ec_code_base">309</field>
            <field name="l10n_ec_code_ats">309</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_309')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_1_75x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_359')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_309')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_10x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_359')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_310" model="account.tax.template">
            <field name="name">310 1% servicio de transporte privado de pasajeros o transporte publico o privado de carga</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-1.0</field>
            <field name="description">310</field>
            <field name="l10n_ec_code_applied">360</field>
            <field name="l10n_ec_code_base">310</field>
            <field name="l10n_ec_code_ats">310</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_310')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_1x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_360')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_310')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_1x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_360')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_311" model="account.tax.template">
            <field name="name">311 2% por pagos a traves de liquidacion de compra (nivel cultural o rusticidad)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-2.0</field>
            <field name="description">311</field>
            <field name="l10n_ec_code_applied">361</field>
            <field name="l10n_ec_code_base">311</field>
            <field name="l10n_ec_code_ats">311</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_311')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_2x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_361')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_311')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_2x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_361')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_312" model="account.tax.template">
            <field name="name">312 1.75% transferencia de bienes muebles de naturaleza corporal</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-1.75</field>
            <field name="description">312</field>
            <field name="l10n_ec_code_applied">362</field>
            <field name="l10n_ec_code_base">312</field>
            <field name="l10n_ec_code_ats">312</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_312')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_1_75x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_362')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_312')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_1_75x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_362')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_312A" model="account.tax.template">
            <field name="name">312a 1% compra de bienes de origen agricola, avicola, pecuario, apicola, cunicula, bioacuatico, y forestal</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-1.0</field>
            <field name="description">312</field>
            <field name="l10n_ec_code_applied">362</field>
            <field name="l10n_ec_code_base">312</field>
            <field name="l10n_ec_code_ats">312A</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_312')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_1x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_362')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_312')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_1x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_362')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_314A" model="account.tax.template">
            <field name="name">314a 8% regalias por concepto de franquicias de acuerdo a ley de propiedad intelectual - pago a personas naturales</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-8.0</field>
            <field name="description">314</field>
            <field name="l10n_ec_code_applied">364</field>
            <field name="l10n_ec_code_base">314</field>
            <field name="l10n_ec_code_ats">314A</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_314')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_364')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_314')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_364')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_314B" model="account.tax.template">
            <field name="name">314b 8% casales, derechos de autor, marcas, patentes y similares de acuerdo a ley de propiedad intelectual – pago a personas naturales</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-8.0</field>
            <field name="description">314</field>
            <field name="l10n_ec_code_applied">364</field>
            <field name="l10n_ec_code_base">314</field>
            <field name="l10n_ec_code_ats">314B</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_314')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_364')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_314')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_364')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_314C" model="account.tax.template">
            <field name="name">314c 8% regalias por concepto de franquicias de acuerdo a ley de propiedad intelectual - pago a sociedades</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-8.0</field>
            <field name="description">314</field>
            <field name="l10n_ec_code_applied">364</field>
            <field name="l10n_ec_code_base">314</field>
            <field name="l10n_ec_code_ats">314C</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_314')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_364')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_314')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_364')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_314D" model="account.tax.template">
            <field name="name">314d 8% casales, derechos de autor, marcas, patentes y similares de acuerdo a ley de propiedad intelectual – pago a sociedades</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-8.0</field>
            <field name="description">314</field>
            <field name="l10n_ec_code_applied">364</field>
            <field name="l10n_ec_code_base">314</field>
            <field name="l10n_ec_code_ats">314D</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_314')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_364')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_314')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_364')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_319" model="account.tax.template">
            <field name="name">319 1.75% cuotas de arrendamiento mercantil (prestado por sociedades), inclusive la de opción de compra</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-1.75</field>
            <field name="description">319</field>
            <field name="l10n_ec_code_applied">369</field>
            <field name="l10n_ec_code_base">319</field>
            <field name="l10n_ec_code_ats">319</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_319')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_1_75x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_369')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_319')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_1_75x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_369')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_320" model="account.tax.template">
            <field name="name">320 8% por arrendamiento bienes inmuebles</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-8.0</field>
            <field name="description">320</field>
            <field name="l10n_ec_code_applied">370</field>
            <field name="l10n_ec_code_base">320</field>
            <field name="l10n_ec_code_ats">320</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_320')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_370')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_320')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_8x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_370')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_322" model="account.tax.template">
            <field name="name">322 1.75% seguros y reaseguros (primas y cesiones) 1.75%	</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-1.75</field>
            <field name="description">322</field>
            <field name="l10n_ec_code_applied">372</field>
            <field name="l10n_ec_code_base">322</field>
            <field name="l10n_ec_code_ats">322</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_322')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_1_75x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_372')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_322')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_1_75x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_372')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_332" model="account.tax.template">
            <field name="name">332 0% otras compras de bienes y servicios no sujetas a retencion</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">0.0</field>
            <field name="description">332</field>
            <field name="l10n_ec_code_base">332</field>
            <field name="l10n_ec_code_ats">332</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_332')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_others'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_332')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_others'),
                })]"/>
        </record>
        <record id="tax_withhold_profit_332A" model="account.tax.template">
            <field name="name">332a 0% enajenacion de derechos representativos de capital y otros derechos exentos (mayo 2016)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">0.0</field>
            <field name="description">332</field>
            <field name="l10n_ec_code_base">332</field>
            <field name="l10n_ec_code_ats">332A</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_332')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_others'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_332')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_others'),
                })]"/>
        </record>
        <record id="tax_withhold_profit_332B" model="account.tax.template">
            <field name="name">332b 0% compra de bienes inmuebles</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">0.0</field>
            <field name="description">332</field>
            <field name="l10n_ec_code_base">332</field>
            <field name="l10n_ec_code_ats">332B</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_332')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_others'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_332')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_others'),
                })]"/>
        </record>
        <record id="tax_withhold_profit_332C" model="account.tax.template">
            <field name="name">332c 0% transporte publico de pasajeros</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">0.0</field>
            <field name="description">332</field>
            <field name="l10n_ec_code_base">332</field>
            <field name="l10n_ec_code_ats">332C</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_332')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_others'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_332')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_others'),
                })]"/>
        </record>
        <record id="tax_withhold_profit_332D" model="account.tax.template">
            <field name="name">332d 0% pagos en el pais por transporte de pasajeros o transporte internacional de carga, a compañias nacionales o extranjeras de aviacion o maritimas</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">0.0</field>
            <field name="description">332</field>
            <field name="l10n_ec_code_base">332</field>
            <field name="l10n_ec_code_ats">332D</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_332')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_others'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_332')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_others'),
                })]"/>
        </record>
        <record id="tax_withhold_profit_332G" model="account.tax.template">
            <field name="name">332g 0% pagos con tarjeta de credito</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">0.0</field>
            <field name="description">332</field>
            <field name="l10n_ec_code_base">332</field>
            <field name="l10n_ec_code_ats">332G</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_332')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_others'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_332')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_others'),
                })]"/>
        </record>
        <record id="tax_withhold_profit_332I" model="account.tax.template">
            <field name="name">332i 0% pagos a través de convenios de débito (clientes ifi`s)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">0.0</field>
            <field name="description">332</field>
            <field name="l10n_ec_code_base">332</field>
            <field name="l10n_ec_code_ats">332I</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_332')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_others'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_332')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_others'),
                })]"/>
        </record>
        <record id="tax_withhold_profit_343A" model="account.tax.template">
            <field name="name">343a 1% por energia electrica</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-1.0</field>
            <field name="description">343</field>
            <field name="l10n_ec_code_applied">393</field>
            <field name="l10n_ec_code_base">343</field>
            <field name="l10n_ec_code_ats">343A</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_343')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_1x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_393')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_343')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_1x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_393')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_343B" model="account.tax.template">
            <field name="name">343b 1% por actividades de construccion de obra material inmueble, urbanizacion, lotizacion o actividades similares</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-1.0</field>
            <field name="description">343</field>
            <field name="l10n_ec_code_applied">393</field>
            <field name="l10n_ec_code_base">343</field>
            <field name="l10n_ec_code_ats">343B</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_343')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_1x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_393')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_343')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_1x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_393')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_3440" model="account.tax.template">
            <field name="name">3440 2.75% otras retenciones aplicables el 2,75%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-2.0</field>
            <field name="description">3440</field>
            <field name="l10n_ec_code_applied">394</field>
            <field name="l10n_ec_code_base">3440</field>
            <field name="l10n_ec_code_ats">3440</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_3440')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_2_75x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_394')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_3440')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_2_75x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_394')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_346" model="account.tax.template">
            <field name="name">346 1.75% microempresas (otras retenciones aplicables a otros porcentajes)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-1.75</field>
            <field name="description">346</field>
            <field name="l10n_ec_code_applied">396</field>
            <field name="l10n_ec_code_base">346</field>
            <field name="l10n_ec_code_ats">346</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_396')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_1_75x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_346')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_396')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_1_75x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_346')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_347_346" model="account.tax.template">
            <field name="name">347-346 2% donaciones en dinero -impuesto a la donaciones</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-2.0</field>
            <field name="description">346</field>
            <field name="l10n_ec_code_applied">396</field>
            <field name="l10n_ec_code_base">346</field>
            <field name="l10n_ec_code_ats">347</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_346')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_2x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_396')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_346')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_2x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_396')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_501_411" model="account.tax.template">
            <field name="name">501-411 22% pago al exterior - beneficios empresariales (con convenio de doble tributación)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-22.0</field>
            <field name="description">411</field>
            <field name="l10n_ec_code_applied">461</field>
            <field name="l10n_ec_code_base">411</field>
            <field name="l10n_ec_code_ats">501</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_411')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_461')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_411')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_461')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_501_422" model="account.tax.template">
            <field name="name">501-422 22% pago al exterior - beneficios empresariales (sin convenio de doble tributación)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-22.0</field>
            <field name="description">422</field>
            <field name="l10n_ec_code_applied">472</field>
            <field name="l10n_ec_code_base">422</field>
            <field name="l10n_ec_code_ats">501</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_422')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_472')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_422')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_472')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_502_411" model="account.tax.template">
            <field name="name">502-411 22% pago al exterior - servicios empresariales (con convenio de doble tributación)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-22.0</field>
            <field name="description">411</field>
            <field name="l10n_ec_code_applied">461</field>
            <field name="l10n_ec_code_base">411</field>
            <field name="l10n_ec_code_ats">502</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_411')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_461')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_411')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_461')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_502_422" model="account.tax.template">
            <field name="name">502-422 22% pago al exterior - servicios empresariales (sin convenio de doble tributación)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-22.0</field>
            <field name="description">422</field>
            <field name="l10n_ec_code_applied">472</field>
            <field name="l10n_ec_code_base">422</field>
            <field name="l10n_ec_code_ats">502</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_422')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_472')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_422')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_472')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_509_411" model="account.tax.template">
            <field name="name">509-411 22% pago al exterior - casales, derechos de autor, marcas, patentes y similares (con convenio de doble tributación)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-22.0</field>
            <field name="description">411</field>
            <field name="l10n_ec_code_base">411</field>
            <field name="l10n_ec_code_applied">461</field>
            <field name="l10n_ec_code_ats">509</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_411')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_461')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_411')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_461')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_509_422" model="account.tax.template">
            <field name="name">509-422 pago al exterior - casales, derechos de autor, marcas, patentes y similares (sin convenio de doble tributación)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-22.0</field>
            <field name="description">422</field>
            <field name="l10n_ec_code_base">422</field>
            <field name="l10n_ec_code_applied">472</field>
            <field name="l10n_ec_code_ats">509</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_422')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_472')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_422')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_472')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_511_411" model="account.tax.template">
            <field name="name">511-411 22% pago al exterior - servicios profesionales independientes (con convenio de doble tributación)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-22.0</field>
            <field name="description">411</field>
            <field name="l10n_ec_code_base">411</field>
            <field name="l10n_ec_code_applied">461</field>
            <field name="l10n_ec_code_ats">511</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_411')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_461')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_411')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_461')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_512_411" model="account.tax.template">
            <field name="name">512-411 22% pago al exterior - servicios profesionales dependientes (con convenio de doble tributación)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-22.0</field>
            <field name="description">411</field>
            <field name="l10n_ec_code_base">411</field>
            <field name="l10n_ec_code_applied">461</field>
            <field name="l10n_ec_code_ats">512</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_411')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_461')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_411')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_461')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_517_411" model="account.tax.template">
            <field name="name">517-411 22% pago al exterior - reembolso de gastos (con convenio de doble tributación)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-22.0</field>
            <field name="description">411</field>
            <field name="l10n_ec_code_base">411</field>
            <field name="l10n_ec_code_applied">461</field>
            <field name="l10n_ec_code_ats">517</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_411')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_461')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_411')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_461')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_520D_411" model="account.tax.template">
            <field name="name">520d-411 22% pago al exterior - comisiones por exportaciones y por promocion de turismo receptivo (con convenio de doble tributación)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-22.0</field>
            <field name="description">411</field>
            <field name="l10n_ec_code_base">411</field>
            <field name="l10n_ec_code_applied">461</field>
            <field name="l10n_ec_code_ats">520D</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_411')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_461')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_411')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_461')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_522A_410" model="account.tax.template">
            <field name="name">522a-410 22% pago al exterior - servicios tecnicos, administrativos o de consultoria y regalias con convenio de doble tributacion (con convenio de doble tributación)</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-22.0</field>
            <field name="description">410</field>
            <field name="l10n_ec_code_base">410</field>
            <field name="l10n_ec_code_applied">460</field>
            <field name="l10n_ec_code_ats">522A</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_410')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'plus_report_line_ids': [ref('tax_report_line_103_460')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_410')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ret_ir_22x100'),
                    'minus_report_line_ids': [ref('tax_report_line_103_460')],
                })]"/>
        </record>
        <!-- 
		SALES WITHHOLDS OVER ANTICIPADED PROFIT *IMPUESTO A LA RENTA VENTA
		-->
        <record id="tax_withhold_profit_sale_1x100" model="account.tax.template">
            <field name="name">1% retenciones de la fuente</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-1.0</field>
            <field name="description">1%</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_343')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_profit_withhold'),
                    'plus_report_line_ids': [ref('tax_report_line_103_393')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_343')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_profit_withhold'),
                    'minus_report_line_ids': [ref('tax_report_line_103_393')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_sale_1_75x100" model="account.tax.template">
            <field name="name">1.75% retenciones de la fuente</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-1.75</field>
            <field name="description">1.75%</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_346')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_profit_withhold'),
                    'plus_report_line_ids': [ref('tax_report_line_103_396')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_346')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_profit_withhold'),
                    'minus_report_line_ids': [ref('tax_report_line_103_396')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_sale_2x100" model="account.tax.template">
            <field name="name">2% retenciones de la fuente</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-2.0</field>
            <field name="description">2%</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_344')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_profit_withhold'),
                    'plus_report_line_ids': [ref('tax_report_line_103_394')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_344')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_profit_withhold'),
                    'minus_report_line_ids': [ref('tax_report_line_103_394')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_sale_2_75x100" model="account.tax.template">
            <field name="name">2.75% retenciones de la fuente</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-2.75</field>
            <field name="description">2.75%</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_3440')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_profit_withhold'),
                    'plus_report_line_ids': [ref('tax_report_line_103_3940')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_3440')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_profit_withhold'),
                    'minus_report_line_ids': [ref('tax_report_line_103_3940')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_sale_5x100" model="account.tax.template">
            <field name="name">5% retenciones de la fuente</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-5.0</field>
            <field name="description">5%</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_346')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_profit_withhold'),
                    'plus_report_line_ids': [ref('tax_report_line_103_396')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_346')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_profit_withhold'),
                    'minus_report_line_ids': [ref('tax_report_line_103_396')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_sale_8x100" model="account.tax.template">
            <field name="name">8% retenciones de la fuente</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-8.0</field>
            <field name="description">8%</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_345')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_profit_withhold'),
                    'plus_report_line_ids': [ref('tax_report_line_103_395')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_345')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_profit_withhold'),
                    'minus_report_line_ids': [ref('tax_report_line_103_395')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_sale_10x100" model="account.tax.template">
            <field name="name">10% retenciones de la fuente</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-10.0</field>
            <field name="description">10%</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_346')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_profit_withhold'),
                    'plus_report_line_ids': [ref('tax_report_line_103_396')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_346')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_profit_withhold'),
                    'minus_report_line_ids': [ref('tax_report_line_103_396')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_sale_15x100" model="account.tax.template">
            <field name="name">15% retenciones de la fuente</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-15.0</field>
            <field name="description">15%</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_346')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_profit_withhold'),
                    'plus_report_line_ids': [ref('tax_report_line_103_396')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_103_346')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_profit_withhold'),
                    'minus_report_line_ids': [ref('tax_report_line_103_396')],
                })]"/>
        </record>
        <record id="tax_withhold_profit_sale_22x100" model="account.tax.template">
            <field name="name">22% retenciones de la fuente</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">70</field>
            <field name="amount">-22.0</field>
            <field name="description">22%</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_income"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_346')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_profit_withhold'),
                    'plus_report_line_ids': [ref('tax_report_line_103_396')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_103_346')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_profit_withhold'),
                    'minus_report_line_ids': [ref('tax_report_line_103_396')],
                })]"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_template_withhold_vat_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- 
        PURCHASE WITHHOLDS COMPUTED OVER VAT *RETENCIONES IVA COMPRAS
        -->
        <record id="tax_withhold_vat_10" model="account.tax.template">
            <field name="name">Retencion iva 10%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">50</field>
            <field name="amount">-10.0</field>
            <field name="description">RET IVA 10%</field>
            <field name="l10n_ec_code_applied">721</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_vat"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_vat_withhold_10'),
                    'plus_report_line_ids': [ref('tax_report_line_104_721')],
                }),]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_vat_withhold_10'),
                    'minus_report_line_ids': [ref('tax_report_line_104_721')],
                }),]"/>
        </record>
        <record id="tax_withhold_vat_20" model="account.tax.template">
            <field name="name">Retencion iva 20%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">50</field>
            <field name="amount">-20.0</field>
            <field name="description">RET IVA 20%</field>
            <field name="l10n_ec_code_applied">723</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_vat"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_vat_withhold_20'),
                    'plus_report_line_ids': [ref('tax_report_line_104_723')],
                }),]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_vat_withhold_20'),
                    'minus_report_line_ids': [ref('tax_report_line_104_723')],
                }),]"/>
        </record>
        <record id="tax_withhold_vat_30" model="account.tax.template">
            <field name="name">Retencion iva 30%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">50</field>
            <field name="amount">-30.0</field>
            <field name="description">RET IVA 30%</field>
            <field name="l10n_ec_code_applied">725</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_vat"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_vat_withhold_30'),
                    'plus_report_line_ids': [ref('tax_report_line_104_725')],
                }),]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_vat_withhold_30'),
                    'minus_report_line_ids': [ref('tax_report_line_104_725')],
                }),]"/>
        </record>
        <record id="tax_withhold_vat_50" model="account.tax.template">
            <field name="name">Retencion iva 50%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">50</field>
            <field name="amount">-50.0</field>
            <field name="description">RET IVA 50%</field>
            <field name="l10n_ec_code_applied">727</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_vat"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_vat_withhold_50'),
                    'plus_report_line_ids': [ref('tax_report_line_104_727')],
                }),]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_vat_withhold_50'),
                    'minus_report_line_ids': [ref('tax_report_line_104_727')],
                }),]"/>
        </record>
        <record id="tax_withhold_vat_70" model="account.tax.template">
            <field name="name">Retencion iva 70%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">50</field>
            <field name="amount">-70.0</field>
            <field name="description">RET IVA 70%</field>
            <field name="l10n_ec_code_applied">729</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_vat"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_vat_withhold_70'),
                    'plus_report_line_ids': [ref('tax_report_line_104_729')],
                }),]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_vat_withhold_70'),
                    'minus_report_line_ids': [ref('tax_report_line_104_729')],
                }),]"/>
        </record>
        <record id="tax_withhold_vat_100" model="account.tax.template">
            <field name="name">Retencion iva 100%</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
            <field name="sequence">50</field>
            <field name="amount">-100.0</field>
            <field name="description">RET IVA 100%</field>
            <field name="l10n_ec_code_applied">731</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_vat"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_vat_withhold_100'),
                    'plus_report_line_ids': [ref('tax_report_line_104_731')],
                }),]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_vat_withhold_100'),
                    'minus_report_line_ids': [ref('tax_report_line_104_731')],
                }),]"/>
        </record>
        <!-- 
        SALES WITHHOLDS COMPUTED OVER VAT *RETENCIONES IVA VENTAS
        -->
        <record id="tax_sale_withhold_vat_10" model="account.tax.template">
            <field name="name">Retencion iva 10%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">60</field>
            <field name="amount">-10.0</field>
            <field name="description">RET IVA 10%</field>
            <field name="l10n_ec_code_applied">609</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_vat"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_outstanding_withholds'),
                    'plus_report_line_ids': [ref('tax_report_line_104_609')],
                }),]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_outstanding_withholds'),
                    'minus_report_line_ids': [ref('tax_report_line_104_609')],
                }),]"/>
        </record>
        <record id="tax_sale_withhold_vat_20" model="account.tax.template">
            <field name="name">Retencion iva 20%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">60</field>
            <field name="amount">-20.0</field>
            <field name="description">RET IVA 20%</field>
            <field name="l10n_ec_code_applied">609</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_vat"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_outstanding_withholds'),
                    'plus_report_line_ids': [ref('tax_report_line_104_609')],
                }),]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_outstanding_withholds'),
                    'minus_report_line_ids': [ref('tax_report_line_104_609')],
                }),]"/>
        </record>
        <record id="tax_sale_withhold_vat_30" model="account.tax.template">
            <field name="name">Retencion iva 30%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">60</field>
            <field name="amount">-30.0</field>
            <field name="description">RET IVA 30%</field>
            <field name="l10n_ec_code_applied">609</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_vat"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_outstanding_withholds'),
                    'plus_report_line_ids': [ref('tax_report_line_104_609')],
                }),]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_outstanding_withholds'),
                    'minus_report_line_ids': [ref('tax_report_line_104_609')],
                }),]"/>
        </record>
        <record id="tax_sale_withhold_vat_50" model="account.tax.template">
            <field name="name">Retencion iva 50%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">60</field>
            <field name="amount">-50.0</field>
            <field name="description">RET IVA 50%</field>
            <field name="l10n_ec_code_applied">609</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_vat"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_outstanding_withholds'),
                    'plus_report_line_ids': [ref('tax_report_line_104_609')],
                }),]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_outstanding_withholds'),
                    'minus_report_line_ids': [ref('tax_report_line_104_609')],
                }),]"/>
        </record>
        <record id="tax_sale_withhold_vat_70" model="account.tax.template">
            <field name="name">Retencion iva 70%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">60</field>
            <field name="amount">-70.0</field>
            <field name="description">RET IVA 70%</field>
            <field name="l10n_ec_code_applied">609</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_vat"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_outstanding_withholds'),
                    'plus_report_line_ids': [ref('tax_report_line_104_609')],
                }),]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_outstanding_withholds'),
                    'minus_report_line_ids': [ref('tax_report_line_104_609')],
                }),]"/>
        </record>
        <record id="tax_sale_withhold_vat_100" model="account.tax.template">
            <field name="name">Retencion iva 100%</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="sequence">60</field>
            <field name="amount">-100.0</field>
            <field name="description">RET IVA 100%</field>
            <field name="l10n_ec_code_applied">609</field>
            <field name="chart_template_id" ref="l10n_ec.l10n_ec_ifrs"/>
            <field name="tax_group_id" ref="tax_group_withhold_vat"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_outstanding_withholds'),
                    'plus_report_line_ids': [ref('tax_report_line_104_609')],
                }),]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 12,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_ec.ec_sale_vat_outstanding_withholds'),
                    'minus_report_line_ids': [ref('tax_report_line_104_609')],
                }),]"/>
        </record>
    </data>
</odoo>

```

## File: data\l10n_ec.sri.payment.csv

```csv
id,code,name
P1,01,Sin utilización del sistema financiero
P15,15,Compensación de Deudas
P16,16,Tarjeta de Débito
P17,17,Dinero Electrónico
P18,18,Tarjeta de Prepago
P19,19,Tarjeta de Crédito
P20,20,Otros con utilización del sistema financiero
P21,21,Endoso de Títulos

```

## File: data\l10n_latam.document.type.csv

```csv
"id","sequence","code","name","report_name","internal_type","doc_code_prefix","country_id/id","active","l10n_ec_check_format"
"ec_dt_01",1,"01","Factura","Factura","invoice","FA","base.ec",1,1
"ec_dt_02",2,"02","Nota o boleta de venta","Nota o boleta de venta","invoice","NV","base.ec",1,1
"ec_dt_03",3,"03","Liquidación de compra de Bienes o Prestación de servicios","Liquidación de compra de Bienes o Prestación de servicios","purchase_liquidation","LDC","base.ec",1,1
"ec_dt_04",4,"04","Nota de  Crédito","Nota de  Crédito","credit_note","NC","base.ec",1,1
"ec_dt_05",5,"05","Nota de Débito","Nota de Débito","debit_note","ND","base.ec",1,1
"ec_dt_08",6,"08","Boletos o entradas a espectáculos públicos","Boletos o entradas a espectáculos públicos","invoice","DOC","base.ec",1,0
"ec_dt_09",7,"09","Tiquetes o vales emitidos por máquinas registradoras","Tiquetes o vales emitidos por máquinas registradoras","invoice","DOC","base.ec",1,0
"ec_dt_11",8,"11","Pasajes expedidos por empresas de aviación","Pasajes expedidos por empresas de aviación","invoice","DOC","base.ec",1,0
"ec_dt_12",9,"12","Documentos emitidos por instituciones financieras","Documentos emitidos por instituciones financieras","invoice","DOC","base.ec",1,0
"ec_dt_15",10,"15","Comprobantes  de venta emitidos en exterior","Comprobantes  de venta emitidos en exterior","invoice","DOC","base.ec",1,0
"ec_dt_16",11,"16","Formulario Único de Exportación (FUE) o Declaración Aduanera Única (DAU) o Declaración Andina de Valor (DAV)","Formulario Único de Exportación (FUE) o Declaración Aduanera Única (DAU) o Declaración Andina de Valor (DAV)","invoice","DOC","base.ec",1,0
"ec_dt_18",12,"18","Documentos autorizados utilizados en ventas excepto N/C N/D","Factura","invoice","FA","base.ec",1,1
"ec_dt_19",13,"19","Comprobantes de Pago de Cuotas o Aportes","Comprobantes de Pago de Cuotas o Aportes","invoice","DOC","base.ec",1,0
"ec_dt_20",14,"20","Documentos por Servicios Administrativos emitidos por Inst. del Estado","Documentos por Servicios Administrativos emitidos por Inst. del Estado","invoice","DOC","base.ec",1,0
"ec_dt_21",15,"21","Carta de Porte Aéreo","Carta de Porte Aéreo","invoice","DOC","base.ec",1,0
"ec_dt_22",16,"22","RECAP","RECAP","invoice","RECAP","base.ec",1,0
"ec_dt_23",17,"23","Nota de Crédito TC","Nota de Crédito TC","credit_note","DOC","base.ec",1,1
"ec_dt_24",18,"24","Nota de Débito TC","Nota de Débito TC","debit_note","DOC","base.ec",1,1
"ec_dt_41",19,"41","Comprobante de venta emitido por reembolso","Comprobante de venta emitido por reembolso","invoice","REM","base.ec",1,0
"ec_dt_42",20,"42","Documento retención presuntiva y retención emitida por propio vendedor o por intermediario","Documento retención presuntiva y retención emitida por propio vendedor o por intermediario","invoice","DOC","base.ec",1,0
"ec_dt_43",21,"43","Liquidacion para Explotacion y Exploracion de Hidrocarburos","Liquidacion para Explotacion y Exploracion de Hidrocarburos","invoice","DOC","base.ec",1,0
"ec_dt_44",22,"44","Comprobante de Contribuciones y Aportes","Comprobante de Contribuciones y Aportes","invoice","DOC","base.ec",1,0
"ec_dt_45",23,"45","Liquidación de medicina prepagada","Liquidación de medicina prepagada","purchase_liquidation","DOC","base.ec",1,0
"ec_dt_47",24,"47","N/C por Reembolso Emitida por Intermediario","N/C por Reembolso Emitida por Intermediario","credit_note","NCR","base.ec",1,1
"ec_dt_48",25,"48","N/D por Reembolso Emitida por Intermediario","N/D por Reembolso Emitida por Intermediario","debit_note","NDR","base.ec",1,1
"ec_dt_49",26,"49","Proveedor Directo de Exportador Bajo Régimen Especial","Proveedor Directo de Exportador Bajo Régimen Especial","invoice","DOC","base.ec",1,0
"ec_dt_50",27,"50","A Inst. Estado y Empr. Públicas que percibe ingreso exento de Imp. Renta","A Inst. Estado y Empr. Públicas que percibe ingreso exento de Imp. Renta","invoice","DOC","base.ec",1,0
"ec_dt_51",28,"51","N/C A Inst. Estado y Empr. Públicas que percibe ingreso exento de Imp. Renta","N/C A Inst. Estado y Empr. Públicas que percibe ingreso exento de Imp. Renta","credit_note","DOC","base.ec",1,1
"ec_dt_52",29,"52","N/D A Inst. Estado y Empr. Públicas que percibe ingreso exento de Imp. Renta","N/D A Inst. Estado y Empr. Públicas que percibe ingreso exento de Imp. Renta","debit_note","DOC","base.ec",1,1
"ec_dt_294",30,"294","Liquidación de compra de Bienes Muebles Usados","Liquidación de compra de Bienes Muebles Usados","purchase_liquidation","LDC","base.ec",1,0
"ec_dt_344",31,"344","Liquidación de compra de vehículos usados","Liquidación de compra de vehículos usados","purchase_liquidation","LDC","base.ec",1,0
"ec_dt_364",32,"364","Acta Entrega-Recepción PET","Acta Entrega-Recepción PET","invoice","RPET","base.ec",1,0
"ec_dt_370",33,"370","Factura operadora transporte / socio","Factura operadora transporte / socio","invoice","DOC","base.ec",1,0
"ec_dt_371",34,"371","Comprobante socio a operadora de transporte","Comprobante socio a operadora de transporte","invoice","DOC","base.ec",1,0
"ec_dt_372",35,"372","Nota de  crédito  operadora transporte / socio","Nota de  crédito  operadora transporte / socio","credit_note","DOC","base.ec",1,0
"ec_dt_373",36,"373","Nota de  débito  operadora transporte / socio","Nota de  débito  operadora transporte / socio","debit_note","DOC","base.ec",1,0

```

## File: data\l10n_latam_identification_type_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id='ec_ruc' model='l10n_latam.identification.type'>
            <field name='name'>RUC</field>
            <field name='description'>Registre Unico de Contribuyente</field>
            <field name='country_id' ref='base.ec'/>
            <field name='is_vat' eval='True'/>
            <field name='sequence'>10</field>
        </record>
        <record id='ec_dni' model='l10n_latam.identification.type'>
            <field name='name'>Cédula</field>
            <field name='description'>Cédula de Ciudadanía o Cédula de Identidad</field>
            <field name='country_id' ref='base.ec'/>
            <field name='sequence'>20</field>
        </record>
        <record id='ec_passport' model='l10n_latam.identification.type'>
            <field name='name'>Pasaporte</field>
            <field name='description'>Pasaporte para extranjeros con domicilio en el país</field>
            <field name='country_id' ref='base.ec'/>
            <field name='sequence'>20</field>
        </record>
        <record id='ec_unknown' model='l10n_latam.identification.type'>
            <field name='name'>Unknown</field>
            <field name='description'>Por identificar, util para registro rápido de ventas</field>
            <field name='country_id' ref='base.ec'/>
            <field name='sequence'>110</field>
        </record>
    </data>
</odoo>

```

## File: data\res.bank.csv

```csv
id,bic,name,country
bank_1,1,Banco Central Del Ecuador,Ecuador
bank_2,10,Banco Pichincha C.A.,Ecuador
bank_3,17,Banco De Guayaquil S.A,Ecuador
bank_4,24,Banco City Bank,Ecuador
bank_5,25,Banco Machala,Ecuador
bank_6,29,Banco De Loja,Ecuador
bank_7,30,Banco Del Pacifico,Ecuador
bank_8,32,Banco Internacional,Ecuador
bank_9,34,Banco Amazonas,Ecuador
bank_10,35,Banco Del Austro,Ecuador
bank_11,36,Produbanco / Promerica,Ecuador
bank_12,37,Banco Bolivariano,Ecuador
bank_13,39,Banco Comercial De Manabi,Ecuador
bank_14,42,Banco General Ruminahui S.A.,Ecuador
bank_15,43,Banco Del Litoral S.A.,Ecuador
bank_16,59,Banco Solidario,Ecuador
bank_17,60,Banco Procredit S.A.,Ecuador
bank_18,61,Banco Capital,Ecuador
bank_19,65,Banco Desarrollo De Los Pueblos S.A.,Ecuador
bank_20,66,Banecuador B.P.,Ecuador
bank_21,70,Coop. Ahorro Y Credito 15 De Abril Ltda,Ecuador
bank_22,76,Coop. Jardin Azuayo,Ecuador
bank_23,79,Coop.De Ahorro Y Credito Policia Nacional,Ecuador
bank_24,82,Financoop,Ecuador
bank_25,201,Banco Delbank S.A.,Ecuador
bank_26,202,Banco Ecuatoriano De La Vivienda,Ecuador
bank_27,203,Cacpeco Ltda,Ecuador
bank_28,204,C.Peq.Empresa De Pastaza,Ecuador
bank_29,205,Coop. Ahorro Y Credito 23 De Julio,Ecuador
bank_30,206,Coop. Ahorro Y Credito 29 De Octubre,Ecuador
bank_31,207,Coop. Ahorro Y Credito Andalucia,Ecuador
bank_32,208,Coop. Ahorro Y Credito Cotocollao,Ecuador
bank_33,210,Coop. Ahorro Y Credito El Sagrario,Ecuador
bank_34,211,Coop. Ahorro Y Credito Guaranda Ltda,Ecuador
bank_35,213,Coop. Juventud Ecuatoriana Progresista Ltda.,Ecuador
bank_36,214,Coop. Ahorro Y Credito Manuel,Ecuador
bank_37,216,Coop. Ahorro Y Credito Oscus,Ecuador
bank_38,217,Coop. Ahorro Y Credito Pablo Munoz Vega,Ecuador
bank_39,218,Coop. Ahorro Y Credito Progreso,Ecuador
bank_40,219,Coop. Ahorro Y Credito Riobamba,Ecuador
bank_41,220,Coop. Ahorro Y Credito San Francisco,Ecuador
bank_42,222,Coop. Ahorro Y Credito Tulcan,Ecuador
bank_43,223,Coop. De Ahorro Y Credito Atuntaqui Ltda.,Ecuador
bank_44,224,Coop. De Ahorro Y Credito Comercio Ltda Portoviejo,Ecuador
bank_45,225,Coop. De Ahorro Y Credito La Dolorosa Ltda,Ecuador
bank_46,226,Coop. Prevision Ahorro Y Desarrollo,Ecuador
bank_47,227,Coop.Ahorro Y Credito Alianza Del Valle Ltda,Ecuador
bank_48,228,Coop Ahorro Y Credito Constrc Comercio Y Produc,Ecuador
bank_49,229,Coop.Ahorro Y Credito Chone Ltda,Ecuador
bank_50,231,Cooperativa De Ahorro Y Credito Santa Ana Ltda,Ecuador
bank_51,232,Financiera - Diners Club Del Ecuador,Ecuador
bank_52,233,Mutualista Ambato,Ecuador
bank_53,234,Mutualista Azuay,Ecuador
bank_54,236,Mutualista Imbabura,Ecuador
bank_55,238,Mutualista Pichincha,Ecuador
bank_56,240,Coop De A. Y C. Corporacion Centro Ltda.,Ecuador
bank_57,242,Coop. De A. Y C. Cordillera De Los Andes Ltda.,Ecuador
bank_58,243,Coop. De A. Y C. Puerto Francisco De Orellana,Ecuador
bank_59,244,Coop. De A Y C. Luz Del Valle,Ecuador
bank_60,245,Coop. De A Y C. Esperanza Y Progreso Del Valle,Ecuador
bank_61,246,Coop. De A. Y C. Choco Tungurahua Runa Ltda,Ecuador
bank_62,247,Coop. De A. Y C. Coopartamos Ltda,Ecuador
bank_63,249,Coop. De A. Y C. Emprendedores Coopemprender Ltda.,Ecuador
bank_64,250,Coop. De A. Y C. Nueva Loja Ltda.,Ecuador
bank_65,251,Coop. De A. Y C. Pichncha Ltda.,Ecuador
bank_66,252,Coop. Credito Y Ahorro San Francisco De Asis,Ecuador
bank_67,253,Coop.De Ahorro Y Credito Cacec Ltda. (Cotopaxi),Ecuador
bank_68,254,Coop.De A. Y C. Vencedores De Pichincha Ltda.,Ecuador
bank_69,255,Cooperativa De Ahorro Y Credito San Bartolo Ltda,Ecuador
bank_70,257,Coop. De A. Y C. Del Emigrante Ecuatoriano Y Su Fa,Ecuador
bank_71,258,Coop. Ahorro Y Credito La Libertad Ltda,Ecuador
bank_72,259,Coac Cuna De La Nacionalidad Ltda.,Ecuador
bank_73,260,Cooperativa De Ahorro Y Credito Macodes Ltda,Ecuador
bank_74,262,Cooperativa De Ahorro Y Credito Santa Isabel Ltda.,Ecuador
bank_75,263,Cooperativa De Ahorro Y Credito Ganansol Ltda,Ecuador
bank_76,264,Cooperativa De Ahorro Y Credito Del Azuay,Ecuador
bank_77,265,Cooperativa De Ahorro Y Credito Del Colegio De Arq,Ecuador
bank_78,266,Cooperativa De Ahorro Y Credito Nukanchik,Ecuador
bank_79,267,Coop De Ahorro Y Credito Santa Ana Cuenca,Ecuador
bank_80,268,Cooperativa De Ahorro Y Credito Multiempresarial L,Ecuador
bank_81,269,Coac Del Sindicato De Choferes Profesionales Del A,Ecuador
bank_82,270,Coop. Ahorro Y Credito Juan De Salinas Ltda.,Ecuador
bank_83,271,Coop. Ahorro Y Credito Puellaro Ltda,Ecuador
bank_84,272,Coop. Ahorro Y Credito Nueva Jerusalen,Ecuador
bank_85,273,Coop. Ahorro Y Credito Malchingui Ltda.,Ecuador
bank_86,275,Coop. De Ahorro Y Credito Chola Cuencana Ltda.,Ecuador
bank_87,276,Coop. Ahorro Y Credito Tena Ltda.,Ecuador
bank_88,277,Coop. Ahorro Y Credito De La Pequena Empresa Guala,Ecuador
bank_89,278,Coop. Ahorro Y Credito Alianza Minas Ltda.,Ecuador
bank_90,279,Coop. De Ahorro Y Credito Pedro Moncayo Ltda.,Ecuador
bank_91,281,Cooperativa De Ahorro Y Credito Provida,Ecuador
bank_92,283,Cooperativa De Ahorro Y Credito San Jose S.J.,Ecuador
bank_93,285,Coop. Ahorro Y Credito Senor De Giron,Ecuador
bank_94,287,Coop. De Ahorro Y Credito Educ.Del Tungurahua Ltda,Ecuador
bank_95,290,Cooperativa 15 De Agosto Pilacoto,Ecuador
bank_96,291,Coop. De Ahorro Y Credito Cristo Rey,Ecuador
bank_97,292,Coop. De Ahorro Y Credito Educadores De Chimborazo,Ecuador
bank_98,293,Cooperativa De Ahorro Y Credito Minga Ltda.,Ecuador
bank_99,294,Coop. De Ahorro Y Credito 4 De Octubre Ltda.,Ecuador
bank_100,295,Coop. De Ahorro Y Credito Credi Facil Ltda.,Ecuador
bank_101,296,Coop. De Ahorro Y Credito Alfonso Jaramillo C.C.C.,Ecuador
bank_102,297,Coop. De A. Y C. Indigena Sac Pelileo,Ecuador
bank_103,298,Coop. De A. Y C. Intercultural Tawantinsuyu Ltda.,Ecuador
bank_104,299,Coop. De A. Y C. Ocipsa,Ecuador
bank_105,300,Coop. De A. Y C. 21 De Noviembre Ltda.,Ecuador
bank_106,301,Coop. De A. Y C. La Floresta Ltda.,Ecuador
bank_107,302,Coop. De A. Y C. Corp. Org. Campesinas De Quisapin,Ecuador
bank_108,303,Coop. De A. Y C. Multicultural Banco Indigena Ltda,Ecuador
bank_109,304,Coop De A. Y C. Crecer Winari Ltda.,Ecuador
bank_110,305,Coop. De A. Y C. Banos De Agua Santa Ltda.,Ecuador
bank_111,306,Coop. De Ahorro Y Credito 1 De Julio,Ecuador
bank_112,307,Coop. De A. Y C. Sumak Nan Ltda.,Ecuador
bank_113,308,Cooperativa De Ahorro Y Credito Canar Ltda.,Ecuador
bank_114,309,Cooperativa De Ahorro Y Credito San Antonio Ltda.,Ecuador
bank_115,310,Coop. De Ahorro Y Credito Fundar,Ecuador
bank_116,312,Coop. Ahorro Y Credito Metropolis Ltda.,Ecuador
bank_117,313,Coop. De Ahorro Y Credito El Cafetal,Ecuador
bank_118,314,Coop.De Ahorro Y Credito Microempresarial Sucre,Ecuador
bank_119,315,Coop. De A. Y C. Afroecuatoriana De La Peq. Emp. L,Ecuador
bank_120,316,Cooperativa De Ahorro Y Credito Joyocoto Ltda.,Ecuador
bank_121,317,Coop. Ahorro Y Credito San Antonio Ltda.,Ecuador
bank_122,318,Coop. De A. Y C. Uniotavalo Ltda.,Ecuador
bank_123,319,Coop. De A. Y C. Union El Ejido,Ecuador
bank_124,320,Coop. De A. Y C. Genesis Ltda.,Ecuador
bank_125,321,Coop. De A Y C. Maria Auxiliadora De Quiroga Ltda,Ecuador
bank_126,322,Coop. De A. Y C. Fortaleza,Ecuador
bank_127,323,Banco Para La Asistencia Comunitaria Finca S.A.,Ecuador
bank_128,324,Coop. Calceta Ltda,Ecuador
bank_129,325,Coop. Ahorro Y Credito 9 De Octubre Ltda,Ecuador
bank_130,327,Coop. Ahorro Y Credito D La Peq Empr Cacpe Biblian,Ecuador
bank_131,328,Coop. Ahorro Y Credito San Gabriel Ltda.,Ecuador
bank_132,329,Coop. Ahorro Y Credito San Jose Ltda,Ecuador
bank_133,330,Coop. Ahorro Y Credito San Miguel De Los Bancos,Ecuador
bank_134,332,Coop. Ahorro Y Credito Semilla Del Progreso Ltda,Ecuador
bank_135,333,Coop. Ahorro. Y Credi. Mujeres Unidas Tantanakushk,Ecuador
bank_136,334,Coop. De Ahorro Y Cred. Santa Rosa Ltda,Ecuador
bank_137,335,Coop. De Ahorro Y Credito Once De Junio,Ecuador
bank_138,336,Coop. De A. Y C. Pujili Ltda,Ecuador
bank_139,337,Coop. De A. Y C. Credil Ltda.,Ecuador
bank_140,339,Coop. De A. Y C. Mushuk Pakari Ltda.,Ecuador
bank_141,340,Coop. De A. Y C. Uniblock Y Servicios Ltda.,Ecuador
bank_142,341,Coop. De A. Y C. San Fernando Limitada,Ecuador
bank_143,342,Coop. De A. Y C. Futuro Lamanense,Ecuador
bank_144,343,Coop. De A. Y C. Saquisili Ltda.,Ecuador
bank_145,344,Coop. De A. Y C. Innovacion Andina Ltda.,Ecuador
bank_146,345,Coop.De A.Y C. El Comerciante Ltda (Mies),Ecuador
bank_147,346,Coop. De A. Y C. Mushuk Wasi Ltda ( Mies ),Ecuador
bank_148,347,Cooperativa De Ahorro Y Credito Solidaria Ltda,Ecuador
bank_149,348,Cooperativa De Ahorro Y Credito Llanganates,Ecuador
bank_150,349,Interdin S.A.,Ecuador
bank_151,350,Coop.Aho Y Credito De La Peq. Emp. De Loja Cacpe,Ecuador
bank_152,351,Cooperativa De Ahorro Y Credito Economia Del Sur C,Ecuador
bank_153,352,Cooperativa De Ahorro Y Credito Migrantes Y Empren,Ecuador
bank_154,353,Cooperativa De Ahorro Y Credito San Sebastian,Ecuador
bank_155,354,Coop. De A. Y C. Del Sindicato De Choferes Profesi,Ecuador
bank_156,355,Coop.De Ahorro Y Credito 29 De Enero,Ecuador
bank_157,356,Coop. Ahorro Y Credito Agraria Mushuk Kawsay Ltda.,Ecuador
bank_158,357,Cooperativa De Ahorro Y Credito Runa Shungo Ltda.,Ecuador
bank_159,358,Coop. De A. Y C. San Marcos,Ecuador
bank_160,359,Coop. De A. Y C. Antorcha Ltda.,Ecuador
bank_161,360,Cooperativa De Ahorro Y Credito Socio Amigo,Ecuador
bank_162,361,Coop. De A. Y C. Indigenas Galapagos Ltda.,Ecuador
bank_163,362,Coop. Ahorro Y Credito Camara De Comercio Del Cant,Ecuador
bank_164,363,Cooperativa De Ahorro Y Credito La Benefica Ltda.,Ecuador
bank_165,364,Coop.De Ahorro Y Credito Abdon Calderon Ltda.,Ecuador
bank_166,365,Coop. A Y C Camara De Comercio Canton -El Carmen L,Ecuador
bank_167,366,Coop.De Ahorro Y Credito Agroproductiva Manabi Ltd,Ecuador
bank_168,367,Coop. De Ahorro Y Cred. La Inmaculada De San Placi,Ecuador
bank_169,368,Coop. Ahorro Y Credito Cacpe Manabi,Ecuador
bank_170,369,Coop.Ahorro Y Credito Magisterio Manabita Limitada,Ecuador
bank_171,370,Coac Tienda De Dinero Ltda.,Ecuador
bank_172,371,Coop.A.Y C. Santa Maria De La Manga Del Cura Ltda.,Ecuador
bank_173,375,Coop. Ahorro Y Credito Camara De Comercio Indigena,Ecuador
bank_174,379,Coop. De A. Y C. Guamote Ltda.,Ecuador
bank_175,382,Coop. De A.Y C. Produc. Ahorro Invers. Servicio P.,Ecuador
bank_176,383,Coop. De A. Y C. Frandesc Ltda.,Ecuador
bank_177,384,Coop. De A. Y C. Nuka Llakta Ltda.,Ecuador
bank_178,385,Coop. De A Y C. Camara De Comercio Riobamba,Ecuador
bank_179,386,Coop. De A. Y C. Bashalan Ltda.,Ecuador
bank_180,387,Coop. De A. Y C. Camara De Comercio Santo Domingo,Ecuador
bank_181,388,Coop. De A. Y C. Educadores Tulcan Ltda.,Ecuador
bank_182,400,Coop. Warmikunapak Rikchari,Ecuador
bank_183,403,Coop Ahorro Y Credito Mi Tierra,Ecuador
bank_184,404,Coop Ahorro Y Credito De La Peq Emp Cacpe Yanzatza,Ecuador
bank_185,405,Coop Ahorro Y Credito Fundesarrollo,Ecuador
bank_186,406,Caja Ecuaespana,Ecuador
bank_187,408,Coop De Ahorro Y Credito Nueva Huancavilca,Ecuador
bank_188,410,Coop De Ahorro Y Credito Erco Ltda,Ecuador
bank_189,411,Coop Ahorro Y Credito Camara Comercio De Ambato,Ecuador
bank_190,412,Coop Ahorro Y Credito Mushuc Runa Ltda,Ecuador
bank_191,414,Coop De Ahorro Y Credito Ambato Ltda,Ecuador
bank_192,415,Coop De Ahorro Y Credito Dorado Ltda,Ecuador
bank_193,416,Coop De Ahorro Y Credito Nuestros Abuelos Ltda,Ecuador
bank_194,417,Coop De Ahorro Y Credito Artesanos Ltda,Ecuador
bank_195,418,Coop De Ahorro Y Credito Santa Anita Ltda,Ecuador
bank_196,419,Coop De Ahorro Y Credito Huayco Pungo Ltda,Ecuador
bank_197,420,Coop Manuel Esteban Godoy Ortega Ltda Coopmego,Ecuador
bank_198,421,Coop Ahorro Y Credito Padre Julian Lorente Ltda,Ecuador
bank_199,422,Coop Ahorro Y Credito Cariamanga Ltda,Ecuador
bank_200,423,Coop De Ahorro Y Credito Marcabeli Ltda,Ecuador
bank_201,424,Coop De Ahorro Y Credito Agricola Junin Ltda,Ecuador
bank_202,425,Coop De Ahorro Y Credito Puerto Lopez Ltda,Ecuador
bank_203,427,Coop De Ahorro Y Credito Nueva Esperanza,Ecuador
bank_204,428,Coop De Ahorro Y Credito San Jorge Ltda,Ecuador
bank_205,429,Coop De Ahorro Y Credito Fernando Daquilema,Ecuador
bank_206,435,Coop De A Y C Educadores De Pastaza Ltda,Ecuador
bank_207,450,Coop. A. Y C. De La Peq. Emp. Cacpe Zamora Ltda,Ecuador
bank_208,451,Coop. De A. Y C. Desarrollo Integral Ltda,Ecuador
bank_209,452,Coop. De Ahorro Y Credito El Calvario Ltda,Ecuador
bank_210,453,Coop. De A. Y C. Juan Pio De Mora Ltda,Ecuador
bank_211,454,Coop. De Ahorro Y Credito Pilahuin,Ecuador
bank_212,455,Coop. De Ahorro Y Credito Pucara Ltda,Ecuador
bank_213,456,Coop. A. Y C. Camara De Comercio De Pindal Cadecop,Ecuador
bank_214,461,Banco Del Instituto Ecuatoriano De Seguridad Social,Ecuador
bank_215,462,Coop De A Y C De Serv Publ Del Min Educacion Y Cul,Ecuador
bank_216,463,Coop De A Y C 23 De Mayo Ltda,Ecuador
bank_217,464,Coop De A Y C Coca Ltda,Ecuador
bank_218,465,Coop De Ahorro Y Credito Pilahuin Tio Ltda,Ecuador
bank_219,467,Coop De Ahorro Y Credito Huaicana Ltda,Ecuador
bank_220,468,Coop De A Y C Credisur Ltda,Ecuador
bank_221,469,Fondo De Cesantia Del Magisterio Ecuat Fcme-Fcpc,Ecuador
bank_222,471,Coop De Ahorro Y Credito Huaquillas Ltda,Ecuador
bank_223,472,Coop De A Y C Banos Ltda,Ecuador
bank_224,473,Coop De Ahorro Y Credito Cac-Cica Mies,Ecuador
bank_225,475,Coop De A Y C Vencedores De Tungurahua,Ecuador
bank_226,476,Coop A Y C Ecuafuturo Ltda,Ecuador
bank_227,477,Coop De A Y C Maquita Cushun Ltda,Ecuador
bank_228,478,Coop De A Y C Valles Del Lirio,Ecuador
bank_229,479,Coop Esfuerzo Unido Para El Desarr Del Chilco,Ecuador
bank_230,480,Coop A Y C Carroceros De Tungurahua,Ecuador
bank_231,481,Coop De Ahorro Y Credito Simiatug Ltda,Ecuador
bank_232,482,Coop De A Y C Sinchi Runa Ltda,Ecuador
bank_233,483,Coop De A Y C Santa Rosa De Pautan Ltda,Ecuador
bank_234,484,Coop De Ahorro Y Credito San Miguel De Sigchos,Ecuador
bank_235,485,Coop De A Y C Sierra Centro Ltda,Ecuador
bank_236,486,Coop De A Y C Gonzanama Mies,Ecuador
bank_237,487,Coop De Ahorro Y Credito Quilanga Ltda,Ecuador
bank_238,488,Coop De Ahorro Y Credito 27 De Abril Loja,Ecuador
bank_239,489,Coop De A Y C Crediamigo Ltda Loja Mies,Ecuador
bank_240,490,Coop De Ahorro Y Credito Fortuna Mies,Ecuador
bank_241,491,Coop A Y C Profesionales Del Volante Union Ltda,Ecuador
bank_242,492,Coop De Ahorro Y Credito Cacpe Celica,Ecuador
bank_243,493,Coop De A Y C Futuro Y Progreso De Galapagos Ltda,Ecuador
bank_244,494,Coop De A Y C Sumac Llacta Ltda,Ecuador
bank_245,495,Coop De A Y C Lucha Campesina Ltda,Ecuador
bank_246,497,Coop De A Y C Maquita Cushunchic Ltda,Ecuador
bank_247,498,Coop A Y C Focla,Ecuador
bank_248,499,Coop De A Y C Casag Ltda,Ecuador
bank_249,500,Banco D Miro Sa,Ecuador
bank_250,501,Coop De A Y C General Ruminahui,Ecuador
bank_251,502,Coop De A Y C Financiera Indigena Ltda,Ecuador
bank_252,503,Coop A Y C 20 Febrero Ltda,Ecuador
bank_253,504,Coop De A Y C Del Col. Fisc. Vicente Rocafuerte,Ecuador
bank_254,505,Coop De A Y C Jadan Ltda (Mies),Ecuador
bank_255,509,Coop De A Y C Inka Kipu Ltda,Ecuador
bank_256,510,Coop De A Y C Accion Tungurahua Ltda,Ecuador
bank_257,511,Coop De A Y C Pijal,Ecuador
bank_258,512,Coop De A Y C Escencia Indigena Ltda,Ecuador
bank_259,513,Coop De A Y C Union Mercedaria Ltda,Ecuador
bank_260,514,Coop De A Y C Catamayo Ltda,Ecuador
bank_261,515,Coop De A Y C De La Peq Empresa Cacpe Macara,Ecuador
bank_262,516,Coop A Y C Nuevos Horizontes El Oro Ltda,Ecuador
bank_263,517,Banco Coopnacional Sa,Ecuador
bank_264,518,Coop De A Y C 16 De Junio,Ecuador
bank_265,519,Coop A Y C Esc Sup Politec Agrop De Manabi Manuel,Ecuador
bank_266,526,Coop De Ahorro Y Credito Fasaynan Ltda,Ecuador
bank_267,546,Coop Cacique Guritave,Ecuador
bank_268,547,Coop Solidaridad Y Progreso Oriental,Ecuador
bank_269,567,Coop. De Aho Y Cred Los Andes Latinos Ltda.,Ecuador
bank_270,569,C. De A. Y C Coopac Austro Ltda (Miess),Ecuador
bank_271,570,Coop. De A. Y C. Salasaca,Ecuador
bank_272,571,Coop. De A. Y C. Sumak Samy Ltda.,Ecuador
bank_273,572,C. A. Y C. Intercult. Tarpuk Runa Ltda.,Ecuador
bank_274,573,Coop. De A. Y C. Virgen Del Cisne,Ecuador
bank_275,574,C. De A Y C Educad. De Zamora Chinchipe,Ecuador
bank_276,575,C. De Ah. Y Credito Las Lagunas (Miess),Ecuador
bank_277,577,C. De A Y C. Educadores De El Oro Ltd,Ecuador
bank_278,578,C De A Y C Emplead Bancar Del Oro Ltda,Ecuador
bank_279,579,Cooperativa De Ah. Y Credito Riochico,Ecuador
bank_280,580,C. De A. Y C. San Martin De Tisaleo Ltda.,Ecuador
bank_281,581,Coop. De A. Y C. Alli Tarpuc Ltda.,Ecuador
bank_282,582,C. De A Y C. San Miguel De Pallatanga,Ecuador
bank_283,590,Coop De A Y C Fenix,Ecuador
bank_284,591,Coop.De Ahorro Y Credito Camara De Comercio De Loj,Ecuador
bank_285,592,Coop.De A. Y C. De Crecimiento Economico Rentable,Ecuador
bank_286,593,Cooperativa De Ahorro Y Credito Santiago Ltda,Ecuador
bank_287,594,Coop.De A. Y C. Popular Y Solidaria,Ecuador
bank_288,595,Coop.De Ahorro Y Credito San Placido Ltda.,Ecuador
bank_289,596,Coop.De Ahorro Y Credito Ciudad De Zamora,Ecuador
bank_290,597,Coop. De A Y C De La Pequena Empresa De Palora,Ecuador
bank_291,600,Coop. De A. Y C. Indigena Alfa Y Omega Ltda,Ecuador
bank_292,603,Cooperativa De Ahorro Y Credito Crea Ltda ( Mies),Ecuador
bank_293,604,Coop. De A. Y C. Chibuleo Ltda.,Ecuador
bank_294,605,Coop. De A. Y C. El Tesoro Pillareno,Ecuador
bank_295,606,Coop. De A. Y C. Kisapincha Ltda.,Ecuador
bank_296,607,Coop. De A. Y C. Juventud Unida Ltda.,Ecuador
bank_297,608,Coop. De A. Y C. Union Quisapincha Ltda.,Ecuador
bank_298,609,Coop. De A. Y C. 13 De Abril Ltda,Ecuador
bank_299,610,Coop. De A. Y C. Salinas Ltda.,Ecuador
bank_300,611,Coop. De A. Y C. San Pedro Ltda.,Ecuador
bank_301,612,Coop. De A. Y C. Los Chasquis Pastocalle Ltda.,Ecuador
bank_302,613,Coop. De A. Y C. Coopindigena Ltda.,Ecuador
bank_303,614,Coop. De A. Y C. La Union Ltda.,Ecuador
bank_304,615,Coop. De A. Y C. Padre Vicente Ponce Rubio,Ecuador
bank_305,633,Coop. De A. Y C. Mushug Causay Ltda.,Ecuador
bank_306,634,Coop. De A. Y C. 29 De Agosto,Ecuador
bank_307,641,Coop. De A. Y C. Credisocio,Ecuador
bank_308,673,Coop. De A. Y C. Fondo Para El Desarrollo Y La Vid,Ecuador
bank_309,674,Cooperativa De A Y C Nueva Vision,Ecuador
bank_310,675,Coop. De Ahorro Y Credito Focash Ltda.,Ecuador
bank_311,676,Coop. De A. Y C. San Vicente Del Sur Ltda.,Ecuador
bank_312,677,Coop. De A. Y C. Para La Vivienda Orden Y Segurida,Ecuador
bank_313,678,Coop De A. Y C. Camara De Comercio Joya De Los Sac,Ecuador
bank_314,679,Coop. De A. Y C. Focap,Ecuador
bank_315,680,Coop. De A. Y C. 18 De Noviembre,Ecuador
bank_316,681,Cooperativa De Ahorro Y Credito Dr. Cornelio Saenz,Ecuador
bank_317,682,Coop De A. Y C. Educadores Del Azuay,Ecuador
bank_318,683,Coop. De A. Y C. Indigena Sac Ltda,Ecuador
bank_319,684,Coop. De A. Y C. Credi Ya Ltda,Ecuador
bank_320,685,Coop. De A. Y C. San Bartolome Ltda,Ecuador
bank_321,686,Coop. De Ahorro Y Credito Mushuk Yuyay,Ecuador
bank_322,687,Coop De A. Y C. Sisay Kanari,Ecuador
bank_323,688,Cooperativa De A Y C Accion Imbaburapak Ltda.,Ecuador
bank_324,689,Coop De A. Y C. Hermes Gaibor Verdesoto,Ecuador
bank_325,690,Coop. De A. Y C. Semillas De Pangua,Ecuador
bank_326,691,Cooperativa De A Y C Mushuk Solidaria,Ecuador
bank_327,692,Coop De A. Y C. Panamericana Ltda,Ecuador
bank_328,693,Coop. De A. Y C. Vilcabamba Cacvil,Ecuador
bank_329,694,Coop. De A. Y C. Familia Solidaria,Ecuador
bank_330,695,Cooperativa De Ahorro Y Credito El Paraiso Manga D,Ecuador
bank_331,696,Coac De Los Profesores Empleados Y Trabajadores De,Ecuador
bank_332,697,Coac. Santa Rosa De San Carlos Ltda.,Ecuador
bank_333,698,Coop. De A. Y C. Constructor Del Desarrollo Solida,Ecuador
bank_334,699,Coop De A. Y C. Mushuk Yuyay Ltda,Ecuador
bank_335,700,Corporacion Financiera,Ecuador
bank_336,701,Coop. De A. Y C. Alianza Social Ecu. Alsec,Ecuador
bank_337,702,Coop. De A. Y C. Renovadora Ecu,Ecuador
bank_338,703,Coop. De A. Y C. Mushuk Yuyay - Napo,Ecuador
bank_339,704,Coop. De A. Y C. Santa Ana De Nayon,Ecuador
bank_340,705,Coop. De A. Y C. Educadores Del Napo,Ecuador
bank_341,706,Cooperativa De A. Y C. Textil 14 De Marzo,Ecuador
bank_342,707,Coop. De A Y C San Carlos Ltda.,Ecuador
bank_343,708,Coac A Y C Esperanza De Valle De La Virgen Ltda.,Ecuador
bank_344,709,Coac A Y C Zona De Capital Corcimol,Ecuador
bank_345,710,Coop.De A. Y C. Artesanal Del Azuay,Ecuador
bank_346,711,Cooperativa De Ahorro Y Credito Sumak Sisa,Ecuador
bank_347,712,Coop. De A. Y C. Rey David Ltda,Ecuador
bank_348,713,Coop. De A. Y C. Kullki Wasi Ltda.,Ecuador
bank_349,714,Coop De A. Y C. Sinchi Codefis,Ecuador
bank_350,715,Coop. De A. Y C. Fuerza De Los Andes,Ecuador
bank_351,716,Coop De A. Y C. Camino De Luz Ltda.,Ecuador
bank_352,717,Coac A Y C Educadores De Bolivar,Ecuador
bank_353,718,Coac San Miguel Ltda.,Ecuador
bank_354,719,Coop. De A. Y C. Salinerita,Ecuador
bank_355,720,Coop. De A. Y C. Bola Amarilla,Ecuador
bank_356,721,Coop A Y C Vision De Los Andes Visandes,Ecuador
bank_357,722,Coop. De A. Y C. Senor Del Arbol,Ecuador
bank_358,724,Coop. De A. Y C. Camara De Comercio De La Mana,Ecuador
bank_359,725,Coop De A. Y C. Educadores De Loja,Ecuador
bank_360,726,Coop De A. Y C. Indigena Sac Latacunga Ltda,Ecuador
bank_361,727,Coop De A.Y C. Cadecog Gonzanama,Ecuador
bank_362,728,Coac Coopymec Macara,Ecuador
bank_363,729,Coac Cadecom Macara,Ecuador
bank_364,730,Coop De A. Y C. 22 De Junio-Orianga,Ecuador
bank_365,732,Coop. A.Yc.Sagrada Familia Solidaridad,Ecuador
bank_366,733,Coop. De A. Y C. Naupa Kausay,Ecuador
bank_367,734,Ccop A Y C De Apecap Cac Apecap,Ecuador
bank_368,248,Coop De A. Y C. San Juan De Cotogchoa,Ecuador
bank_369,2557,Coop De Ahorro Y Credito La Merced,Ecuador
bank_370,338,Coop. De A. Y C. Cooptopaxi Ltda.,Ecuador
bank_371,620,Coop. De A. Y C. Grameen Amazonas,Ecuador
bank_372,621,Coop. De A. Y C. El Transportista Cacet,Ecuador
bank_373,626,Coop. De A Y C. Servidores Municipales De Cuenca,Ecuador
bank_374,628,Coop. De Ahorro Y Credito Profuturo Ltda.,Ecuador
bank_375,640,Coop. De A. Y C. Iliniza Ltda.,Ecuador
bank_376,642,Coop. De Ahorro Y Credito Saraguros,Ecuador
bank_377,643,Cooperativa De Ahorro Y Credito Inti Wasi Ltda.,Ecuador
bank_378,647,Cooperativa De Ahorro Y Credito San Isidro Ltda.,Ecuador
bank_379,722,Coop. De A. Y C. Empresas Comunitarias Coocredito,Ecuador
bank_380,723,Silvercross S.A Casa De Valores Sccv,Ecuador
bank_381,724,Real Casa De Valores De Guayaquil S.A.,Ecuador
bank_382,725,Cooperativa De Ahorro Y Credito Etapa,Ecuador
bank_383,726,Coop. Ahorro Y Credito Mascoop,Ecuador
bank_384,727,Coop. De A. Y C. Saint Michel Ltda.,Ecuador
bank_385,8000,Coop. Ahorro Y Credito Manantial De Oro Ltda.,Ecuador
bank_386,8001,Coop. De Ahorro Y Credito Andina Ltda.,Ecuador
bank_387,8002,Coop. De Ahorro Y Credito Accion Y Desarrollo Ltda,Ecuador
bank_388,9901,Coop. De A. Y C. Union Y Desarrollo,Ecuador
bank_389,9902,Coop. De A. Y C. Esperanza Del Futuro Ltda.,Ecuador
bank_390,9903,Coop. De A. Y C. 17 De Marzo Ltda,Ecuador
bank_391,9904,Coop. De A. Y C. Catar Ltda,Ecuador
bank_392,9905,Coop. De A. Y C. De Accion Popular,Ecuador
bank_393,9906,Coop. De A. Y C. Alli Tarpuk Ltda,Ecuador
bank_394,9907,Coop. De A. Y C. 16 De Julio Ltda,Ecuador
bank_395,9908,Coop. De A. Y C. Suboficiales De La Policia Nacion,Ecuador
bank_396,9909,Coop. De A. Y C. Politecnica Ltda.,Ecuador
bank_397,9910,Coop. De A. Y C. Nacional Llano Grande Ltda.,Ecuador
bank_398,9911,Coop. De A. Y C. San Valentin,Ecuador
bank_399,9912,Coop. De A. Y C. Totalife Ltda,Ecuador
bank_400,9913,Acciones Valores Casa De Valores S.A. Accival,Ecuador
bank_401,9914,Santa Fe Casa De Valores S.A. Santafevalores,Ecuador
bank_402,9915,Picaval Casa De Valores S.A,Ecuador
bank_403,9916,Analytica Securties C.A. Casa De Valores,Ecuador
bank_404,9917,Orion Casa De Valores S.A,Ecuador
bank_405,9918,Stratega Casa De Valores S.A.,Ecuador
bank_406,9919,Ecofsa Casa De Valores S.A.,Ecuador
bank_407,9920,Merchantvalores Casa De Valores S.A.,Ecuador
bank_408,9921,Casa De Valores Value S.A.,Ecuador
bank_409,9922,Inmovalor Casa De Valores S.A.,Ecuador
bank_410,9923,Ecuabursatil Casa De Valores S.A.,Ecuador
bank_411,9924,Plus Valores Casa De Valores S.A.,Ecuador
bank_412,9925,Mercapital Casa De Valores S.A.,Ecuador
bank_413,9926,Portafolio Casa De Valores S.A. Portavalor,Ecuador
bank_414,9927,Metrovalores Casa De Valores S.A.,Ecuador
bank_415,9928,Vectorglobal Wmg Casa De Valores S.A.,Ecuador
bank_416,9929,Holdunpartners Casa De Valores S.A.,Ecuador
bank_417,9930,Coop. De Ahorro Y Credito Base De Taura,Ecuador
bank_418,9931,Casa De Valores Banrio S.A.,Ecuador
bank_419,9932,Albion Casa De Valores S.A.,Ecuador
bank_420,9933,Masvalores Casa De Valores S.A Cavamasa,Ecuador
bank_421,9934,Casa De Valores Del Pacifico (Valpacifico) S.A.,Ecuador
bank_422,9935,Casa De Valores Advfin S.A.,Ecuador
bank_423,9936,Kapital One Casa De Valores S.A. Kaovalsa,Ecuador
bank_424,9937,Plusbursatil Casa De Valores S.A,Ecuador
bank_425,9938,Activa Ases.E Intermed.Valores Activalores Casa De,Ecuador
bank_426,9939,Citadel Casa De Valores S.A.,Ecuador
bank_427,9940,Decevale S.A.,Ecuador
bank_428,9941,Coop. De A. Y C. Santa Lucia Ltda,Ecuador
bank_429,9942,Coop. De A. Y C. Produactiva Ltda,Ecuador
bank_430,9943,Coop. De A. Y C. Tamboloma Ltda.,Ecuador
bank_431,9944,Coop De A. Y C. Pushak Runa Hombre Lider,Ecuador
bank_432,9945,Coop. De A. Y C. 15 De Agosto Ltda.,Ecuador
bank_433,9946,Coop. De A. Y C. Migrantes Del Ecuador Ltda,Ecuador
bank_434,9947,Cooperativa De Ahorro Y Credito Wuamanloma Ltda.,Ecuador
bank_435,9948,Coop. De A. Y C. Salate Ltda.,Ecuador
bank_436,9949,Coop. De A. Y C. Union Popular Ltda.,Ecuador
bank_437,9950,Coac Achik Inti Ltda,Ecuador
bank_438,9951,Coac El Migrante Solidario,Ecuador
bank_439,9952,Coop Ac Las Naves,Ecuador
bank_440,9953,Coop. De A. Y C. De Imbabura Amazonas,Ecuador
bank_441,9954,Coop. A. Y C. De Indigenas Chuchuqui Ltda,Ecuador
bank_442,9955,Coop. Ahorro Y Credito Focazsum Ltda.,Ecuador
bank_443,9956,Coop. De A. Y C. Solidaria Ltda.- Cotopaxi,Ecuador
bank_444,9957,Coop. De A. Y C. Unidad Y Progreso,Ecuador
bank_445,9958,Coac Loja Internacional Ltda.,Ecuador
bank_446,9959,Coac 23 De Enero,Ecuador
bank_447,9960,Coac Obras Publicas Fiscales De Loja Y Zamora,Ecuador
bank_448,9961,Coac Del Sind Chof Prof Virgen Del Cisne,Ecuador
bank_449,9962,Coac San Miguel De Chirijos Ltda,Ecuador
bank_450,9963,Coop. Ahorro Y Credito Quevedo Ltda.,Ecuador
bank_451,9964,Coop. De A. Y C. El Altar Ltda.,Ecuador
bank_452,9965,Coop. De A. Y C. Nueva Alianza De Chimborazo Ltda,Ecuador
bank_453,9966,Coop. De A. Y C. Luis Felipe Duchicela Xxvii,Ecuador
bank_454,9967,Coop. De A. Y C. Nizag Ltda.,Ecuador
bank_455,9968,Coop. De A. Y C. Makita Kunchik,Ecuador
bank_456,9969,Coop. De A. Y C. Cerrada Manuela Leon,Ecuador
bank_457,9970,Coop. De A. Y C. El Buen Sembrador Ltda.,Ecuador
bank_458,9971,Coac Sindicato De Choferes Profesionales De Yantza,Ecuador
bank_459,9972,Coop. De A. Y C. 5 De Mayo De Santa Martha De Cuba,Ecuador

```

## File: data\res_partner_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id="ec_final_consumer" model="res.partner">
            <field name="name">Consumidor final</field>
            <field name="country_id" ref="base.ec"/>
            <field name="l10n_latam_identification_type_id" ref="l10n_ec.ec_ruc"/>
            <field name="vat">9999999999999</field>
        </record>
        <record id="ec_social_security" model="res.partner">
            <field name="image_1920" type="base64" file="l10n_ec/static/img/logo_iess.png"/>
            <field name="name">Instituto Ecuatorian de Seguridad Social</field>
            <field name="state_id" ref="base.state_ec_17"/>
            <field name="country_id" ref="base.ec"/>
            <field name="website">https://www.iess.gob.ec</field>
            <field name="is_company">1</field>
            <field name="l10n_latam_identification_type_id" ref="l10n_ec.ec_ruc"/>
            <field name="vat">1760004650001</field>
        </record>
        <record id="ec_tax_authority" model="res.partner">
            <field name="image_1920" type="base64" file="l10n_ec/static/img/logo_sri.png"/>
            <field name="name">Servicio de Rentas Internas</field>
            <field name="state_id" ref="base.state_ec_17"/>
            <field name="country_id" ref="base.ec"/>
            <field name="website">http://www.sri.gob.ec</field>
            <field name="is_company">1</field>
            <field name="l10n_latam_identification_type_id" ref="l10n_ec.ec_ruc"/>
            <field name="vat">1760013210001</field>
        </record>
    </data>
</odoo>

```

## File: models\account_journal.py

```python
from odoo import fields, models


class AccountJournal(models.Model):

    _inherit = "account.journal"

    l10n_ec_entity = fields.Char(string="Emission Entity", size=3, default="001")
    l10n_ec_emission = fields.Char(string="Emission Point", size=3, default="001")
    l10n_ec_emission_address_id = fields.Many2one(
        comodel_name="res.partner",
        string="Emission address",
        domain="['|', ('id', '=', company_partner_id), '&', ('id', 'child_of', company_partner_id), ('type', '!=', 'contact')]",
    )

    l10n_ec_emission_type = fields.Selection(
        string="Emission type",
        selection=[
            ("pre_printed", "Pre Printed"),
            ("auto_printer", "Auto Printer"),
            ("electronic", "Electronic"),
        ],
        default="electronic",
    )

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api
from odoo.addons.l10n_ec.models.res_partner import verify_final_consumer

_DOCUMENTS_MAPPING = {
    "01": [
        'ec_dt_01',
        'ec_dt_02',
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_08',
        'ec_dt_09',
        'ec_dt_11',
        'ec_dt_12',
        'ec_dt_20',
        'ec_dt_21',
        'ec_dt_41',
        'ec_dt_42',
        'ec_dt_43',
        'ec_dt_45',
        'ec_dt_47',
        'ec_dt_48'
    ],
    "02": [
        'ec_dt_03',
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_09',
        'ec_dt_19',
        'ec_dt_41',
        'ec_dt_294',
        'ec_dt_344'
    ],
    "03": [
        'ec_dt_03',
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_09',
        'ec_dt_15',
        'ec_dt_19',
        'ec_dt_41',
        'ec_dt_45',
        'ec_dt_294',
        'ec_dt_344'
    ],
    "04": [
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_18',
        'ec_dt_41',
        'ec_dt_44',
        'ec_dt_47',
        'ec_dt_48',
        'ec_dt_49',
        'ec_dt_50',
        'ec_dt_51',
        'ec_dt_52',
        'ec_dt_370',
        'ec_dt_371',
        'ec_dt_372',
        'ec_dt_373'
    ],
    "05": [
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_18',
        'ec_dt_41',
        'ec_dt_44',
        'ec_dt_47',
        'ec_dt_48',
        'ec_dt_370',
        'ec_dt_371',
        'ec_dt_372',
        'ec_dt_373'
    ],
    "06": [
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_18',
        'ec_dt_41',
        'ec_dt_44',
        'ec_dt_47',
        'ec_dt_48',
        'ec_dt_370',
        'ec_dt_371',
        'ec_dt_372',
        'ec_dt_373'
    ],
    "07": [
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_18'
    ],
    "09": [
        'ec_dt_01',
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_15',
        'ec_dt_16',
        'ec_dt_41',
        'ec_dt_47',
        'ec_dt_48',
    ],
    "20": [
        'ec_dt_01',
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_15',
        'ec_dt_16',
        'ec_dt_41',
        'ec_dt_47',
        'ec_dt_48'
    ],
    "21": [
        'ec_dt_01',
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_15',
        'ec_dt_16',
        'ec_dt_41',
        'ec_dt_47',
        'ec_dt_48'
    ],
}


class AccountMove(models.Model):
    _inherit = "account.move"

    l10n_ec_sri_payment_id = fields.Many2one(
        comodel_name="l10n_ec.sri.payment",
        string="Payment Method (SRI)",
    )

    def _get_l10n_ec_identification_type(self):
        self.ensure_one()
        move = self
        it_ruc = self.env.ref("l10n_ec.ec_ruc", False)
        it_dni = self.env.ref("l10n_ec.ec_dni", False)
        it_passport = self.env.ref("l10n_ec.ec_passport", False)
        is_final_consumer = verify_final_consumer(move.partner_id.commercial_partner_id.vat)
        is_ruc = move.partner_id.commercial_partner_id.l10n_latam_identification_type_id.id == it_ruc.id
        is_dni = move.partner_id.commercial_partner_id.l10n_latam_identification_type_id.id == it_dni.id
        is_passport = move.partner_id.commercial_partner_id.l10n_latam_identification_type_id.id == it_passport.id
        l10n_ec_is_exportation = move.partner_id.commercial_partner_id.country_id.code != 'EC'
        identification_code = False
        if move.move_type in ("in_invoice", "in_refund"):
            if is_ruc:
                identification_code = "01"
            elif is_dni:
                identification_code = "02"
            else:
                identification_code = "03"
        elif move.move_type in ("out_invoice", "out_refund"):
            if not l10n_ec_is_exportation:
                if is_final_consumer:
                    identification_code = "07"
                elif is_ruc:
                    identification_code = "04"
                elif is_dni:
                    identification_code = "05"
                elif is_passport:
                    identification_code = "06"
            else:
                if is_ruc:
                    identification_code = "20"
                elif is_dni:
                    identification_code = "21"
                else:
                    identification_code = "09"
        return identification_code

    @api.model
    def _get_l10n_ec_documents_allowed(self, identification_code):
        documents_allowed = self.env['l10n_latam.document.type']
        for document_ref in _DOCUMENTS_MAPPING.get(identification_code, []):
            document_allowed = self.env.ref('l10n_ec.%s' % document_ref, False)
            if document_allowed:
                documents_allowed |= document_allowed
        return documents_allowed

    def _get_l10n_ec_internal_type(self):
        self.ensure_one()
        internal_type = self.env.context.get("internal_type", "invoice")
        if self.move_type in ("out_refund", "in_refund"):
            internal_type = "credit_note"
        if self.debit_origin_id:
            internal_type = "debit_note"
        return internal_type

    def _get_l10n_latam_documents_domain(self):
        self.ensure_one()
        if self.journal_id.company_id.account_fiscal_country_id != self.env.ref('base.ec') or not \
                self.journal_id.l10n_latam_use_documents:
            return super()._get_l10n_latam_documents_domain()
        domain = [
            ('country_id.code', '=', 'EC'),
            ('internal_type', 'in', ['invoice', 'debit_note', 'credit_note', 'invoice_in'])
        ]
        internal_type = self._get_l10n_ec_internal_type()
        allowed_documents = self._get_l10n_ec_documents_allowed(self._get_l10n_ec_identification_type())
        if internal_type and allowed_documents:
            domain.append(("id", "in", allowed_documents.filtered(lambda x: x.internal_type == internal_type).ids))
        return domain

    def _get_ec_formatted_sequence(self, number=0):
        return "%s %s-%s-%09d" % (
            self.l10n_latam_document_type_id.doc_code_prefix,
            self.journal_id.l10n_ec_entity,
            self.journal_id.l10n_ec_emission,
            number,
        )

    def _get_starting_sequence(self):
        """If use documents then will create a new starting sequence using the document type code prefix and the
        journal document number with a 8 padding number"""
        if (
            self.journal_id.l10n_latam_use_documents
            and self.company_id.country_id.code == "EC"
        ):
            if self.l10n_latam_document_type_id:
                return self._get_ec_formatted_sequence()
        return super()._get_starting_sequence()

    def _get_last_sequence_domain(self, relaxed=False):
        l10n_latam_document_type_model = self.env['l10n_latam.document.type']
        where_string, param = super(AccountMove, self)._get_last_sequence_domain(relaxed)
        if self.country_code == "EC" and self.l10n_latam_use_documents and self.move_type in (
            "out_invoice",
            "out_refund",
            "in_invoice",
            "in_refund",
        ):
            where_string, param = super(AccountMove, self)._get_last_sequence_domain(False)
            internal_type = self._get_l10n_ec_internal_type()
            document_types = l10n_latam_document_type_model.search([
                ('internal_type', '=', internal_type),
                ('country_id.code', '=', 'EC'),
            ])
            if document_types:
                where_string += """
                AND l10n_latam_document_type_id in %(l10n_latam_document_type_id)s
                """
                param["l10n_latam_document_type_id"] = tuple(document_types.ids)
        return where_string, param

```

## File: models\account_tax.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class AccountTax(models.Model):

    _inherit = "account.tax"

    l10n_ec_code_base = fields.Char(
        string="Code base",
        help="Tax declaration code of the base amount prior to the calculation of the tax",
    )
    l10n_ec_code_applied = fields.Char(
        string="Code applied",
        help="Tax declaration code of the resulting amount after the calculation of the tax",
    )
    l10n_ec_code_ats = fields.Char(
        string="Code ATS",
        help="Tax Identification Code for the Simplified Transactional Annex",
    )


class AccountTaxTemplate(models.Model):

    _inherit = "account.tax.template"

    def _get_tax_vals(self, company, tax_template_to_tax):
        vals = super(AccountTaxTemplate, self)._get_tax_vals(
            company, tax_template_to_tax
        )
        vals.update(
            {
                "l10n_ec_code_base": self.l10n_ec_code_base,
                "l10n_ec_code_applied": self.l10n_ec_code_applied,
                "l10n_ec_code_ats": self.l10n_ec_code_ats,
            }
        )
        return vals

    l10n_ec_code_base = fields.Char(
        string="Code base",
        help="Tax declaration code of the base amount prior to the calculation of the tax",
    )
    l10n_ec_code_applied = fields.Char(
        string="Code applied",
        help="Tax declaration code of the resulting amount after the calculation of the tax",
    )
    l10n_ec_code_ats = fields.Char(
        string="Code ATS",
        help="Tax Identification Code for the Simplified Transactional Annex",
    )

```

## File: models\account_tax_group.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

_TYPE_EC = [
    ("vat12", "VAT 12%"),
    ("vat14", "VAT 14%"),
    ("zero_vat", "VAT 0%"),
    ("not_charged_vat", "VAT Not Charged"),
    ("exempt_vat", "VAT Exempt"),
    ("withhold_vat", "VAT Withhold"),
    ("withhold_income_tax", "Profit Withhold"),
    ("ice", "Special Consumptions Tax (ICE)"),
    ("irbpnr", "Plastic Bottles (IRBPNR)"),
    ("outflows_tax", "Exchange Outflows"),
    ("other", "Others"),
]


class AccountTaxGroup(models.Model):
    _inherit = "account.tax.group"

    l10n_ec_type = fields.Selection(
        _TYPE_EC, string="Type Ecuadorian Tax", help="Ecuadorian taxes subtype"
    )

```

## File: models\l10n_ec_sri_payment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class SriPayment(models.Model):

    _name = "l10n_ec.sri.payment"
    _description = "SRI Payment Method"

    name = fields.Char("Name")
    code = fields.Char("Code")

```

## File: models\l10n_latam_document_type.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _
from odoo.exceptions import UserError
import re


class L10nLatamDocumentType(models.Model):
    _inherit = "l10n_latam.document.type"

    internal_type = fields.Selection(
        selection_add=[
            ("purchase_liquidation", "Purchase Liquidation"),
        ]
    )

    l10n_ec_check_format = fields.Boolean(
        string="Check Number Format EC", default=False
    )

    def _format_document_number(self, document_number):
        self.ensure_one()
        if self.country_id != self.env.ref("base.ec"):
            return super()._format_document_number(document_number)
        if not document_number:
            return False
        if self.l10n_ec_check_format:
            document_number = re.sub(r'\s+', "", document_number)  # remove any whitespace
            num_match = re.match(r'(\d{1,3})-(\d{1,3})-(\d{1,9})', document_number)
            if num_match:
                # Fill each number group with zeroes (3, 3 and 9 respectively)
                document_number = "-".join([n.zfill(3 if i < 2 else 9) for i, n in enumerate(num_match.groups())])
            else:
                raise UserError(
                    _(u"Ecuadorian Document %s must be like 001-001-123456789")
                    % (self.display_name)
                )

        return document_number

```

## File: models\res_company.py

```python
from odoo import models


class ResCompany(models.Model):

    _inherit = "res.company"

    def _localization_use_documents(self):
        self.ensure_one()
        return self.account_fiscal_country_id.code == "EC" or super(ResCompany, self)._localization_use_documents()

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api, _
from odoo.exceptions import ValidationError

import logging

_logger = logging.getLogger(__name__)


def verify_final_consumer(vat):
    all_number_9 = False
    try:
        all_number_9 = vat and all(int(number) == 9 for number in vat) or False
    except ValueError as e:
        _logger.debug('Vat is not only numbers %s', e)
    return all_number_9 and len(vat) == 13


class ResPartner(models.Model):

    _inherit = "res.partner"

    @api.constrains("vat", "country_id", "l10n_latam_identification_type_id")
    def check_vat(self):
        it_ruc = self.env.ref("l10n_ec.ec_ruc", False)
        it_dni = self.env.ref("l10n_ec.ec_dni", False)
        ecuadorian_partners = self.filtered(
            lambda x: x.country_id == self.env.ref("base.ec")
        )
        for partner in ecuadorian_partners:
            if partner.vat:
                if partner.l10n_latam_identification_type_id.id in (
                    it_ruc.id,
                    it_dni.id,
                ):
                    if partner.l10n_latam_identification_type_id.id == it_dni.id and len(partner.vat) != 10:
                        raise ValidationError(_('If your identification type is %s, it must be 10 digits')
                                              % it_dni.display_name)
                    if partner.l10n_latam_identification_type_id.id == it_ruc.id and len(partner.vat) != 13:
                        raise ValidationError(_('If your identification type is %s, it must be 13 digits')
                                              % it_ruc.display_name)
                    final_consumer = verify_final_consumer(partner.vat)
                    if final_consumer:
                        valid = True
                    else:
                        valid = self.is_valid_ruc_ec(partner.vat)
                    if not valid:
                        error_message = ""
                        if partner.l10n_latam_identification_type_id.id == it_dni.id:
                            error_message = _("VAT %s is not valid for an Ecuadorian DNI, "
                                              "it must be like this form 0915068258") % partner.vat
                        if partner.l10n_latam_identification_type_id.id == it_ruc.id:
                            error_message = _("VAT %s is not valid for an Ecuadorian company, "
                                              "it must be like this form 0993143790001") % partner.vat
                        raise ValidationError(error_message)
        return super(ResPartner, self - ecuadorian_partners).check_vat()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_partner
from . import l10n_ec_sri_payment
from . import l10n_latam_document_type
from . import account_move
from . import account_tax
from . import account_tax_group
from . import res_company
from . import account_journal

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_ec_sri_payment_public,l10n_ec_sri_payment_public,model_l10n_ec_sri_payment,,1,0,0,0
access_l10n_ec_sri_payment_admin,l10n_ec_sri_payment_public,model_l10n_ec_sri_payment,account.group_account_manager,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.8" y="3.8" width="50.4" height="36.38" maskUnits="userSpaceOnUse">
      <rect x="6.29" y="7.98" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="800" height="533" transform="translate(4.8 3.8) scale(0.06 0.07)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAyAAAAJCCAYAAAA4O6ONAAAACXBIWXMAAK/IAACvyAF3Om7hAAAgAElEQVR4XuzdeZhcdZn3/3dXnTq1V/We7iwdshBCSFiCkCiLCS6gKLghoOI6KM6Moz4zvxln1FHUx1Fn/Dkz6rjrqM+oOOOCCCIMjwFll0VCSICEkKS39Fpde506VfX8cfqcquruADMjZTp8XteVi3RXdXV1VV/k+znf733fbbW91BAREREREWkB39PdQURERERE5PdFAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFpGAURERERERFrGeLo7iIjI78Hc/9vaQKXhY//sfYzZ22xERESOSQogIiLPhoYwUU7D4WkYT0GuDJPlTQDkshu8uxdKuzF4iIEILOmH45dBIFF/DO//1m4wMZgfUho/t9gDzDP912mx/5wiIs9BbbW91J7uTiIi8gy4oaMIY4dh16NwMA/l2PtJmpvpGNjAkvY4kYgJgBkMAWCVigDk8xaHUxmmDz5COfdz2oPX4AvAkgiUipCqQizgfKt0HhKRBZ5CBVYvh95lLM7FuQGUYHC0HthcnVEIBpw/sQiYAQiZ4DN5+sCyGF8LEZFjlAKIiMj/RMNOx9iQEzqeyJ9M1f8Glqzdwqr+ZbR3JIB60HgqbigBSE2nOZzKNN0eDtVvT4QCTbeli2UKxSIHHr+XzZ1XcdopLK6FtwH79sKOPSeT6Pz/vMAG9Z/NqGTIZtMU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w4tc52smG3v/ze7Bzi3nf5K9C68xx4iIiACWzVferQAiIiKc9LZwU1sdl7fsA+Cleat4pXgBoZJl4xxp1DTu5MrOdyj1RTiY7+bhRRdM+FgRETl7WBKgACIiIqQK0MOEAcgmG9MbEhnroKSB4vUw4UkeKyIiZxNLAqcCiIiIiIiIZIRmwRIRERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYxRABERERERkYyxJBKJ8fYRERERERGZFv8fyfEqaFIgRY8AAAAASUVORK5CYII="/>
    </g>
  </g>
</svg>

```

## File: views\account_journal_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_account_journal_form" model="ir.ui.view">
            <field name="model">account.journal</field>
            <field name="name">account.journal.form</field>
            <field name="inherit_id" ref="l10n_latam_invoice_document.view_account_journal_form"/>
            <field name="arch" type="xml">
                <field name="l10n_latam_use_documents" position="after">
                    <field name="l10n_ec_entity"
                           attrs="{'invisible':['|', '|', ('country_code', '!=', 'EC'), ('l10n_latam_use_documents', '=', False), ('type', 'not in', ('sale', 'purchase'))], 'required':[('country_code', '=', 'EC'), ('l10n_latam_use_documents', '=', True), ('type', 'not in', ('sale', 'purchase'))]}"/>
                    <field name="l10n_ec_emission"
                           attrs="{'invisible':['|', '|', ('country_code', '!=', 'EC'), ('l10n_latam_use_documents', '=', False), ('type', 'not in', ('sale', 'purchase'))], 'required':[('country_code', '=', 'EC'), ('l10n_latam_use_documents', '=', True), ('type', 'not in', ('sale', 'purchase'))]}"
                    />
                    <field name="l10n_ec_emission_address_id"
                           attrs="{'invisible':['|', '|', ('country_code', '!=', 'EC'), ('l10n_latam_use_documents', '=', False), ('type', 'not in', ('sale', 'purchase'))], 'required':[('country_code', '=', 'EC'), ('l10n_latam_use_documents', '=', True), ('type', 'not in', ('sale', 'purchase'))]}"
                    />
                    <field name="l10n_ec_emission_type"
                           attrs="{'invisible':['|', '|', ('country_code', '!=', 'EC'), ('l10n_latam_use_documents', '=', False), ('type', 'not in', ('sale', 'purchase'))], 'required':[('country_code', '=', 'EC'), ('l10n_latam_use_documents', '=', True), ('type', 'not in', ('sale', 'purchase'))]}"
                    />
                </field>
            </field>
        </record>
    </data>
</odoo>
```

## File: views\account_tax_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="account_tax_form_view" model="ir.ui.view">
        <field name="name">account.tax.form</field>
        <field name="model">account.tax</field>
        <field name="type">form</field>
        <field name="inherit_id" ref="account.view_tax_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='type_tax_use']" position="after">
                <field name="l10n_ec_code_base" attrs="{'invisible': [('country_code', '!=', 'EC')]}"
                       groups="base.group_no_one"/>
                <field name="l10n_ec_code_applied" attrs="{'invisible': [('country_code', '!=', 'EC')]}"
                       groups="base.group_no_one"/>
                <field name="l10n_ec_code_ats" attrs="{'invisible': [('country_code', '!=', 'EC')]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\l10n_ec_sri_payment.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="account_l10n_ec_sri_payment" model="ir.ui.view">
        <field name="name">l10n.ec.sri.payment.form</field>
        <field name="model">l10n_ec.sri.payment</field>
        <field name="arch" type="xml">
            <tree>
                <field name="name"/>
                <field name="code"/>
            </tree>
        </field>
    </record>

    <record id="action_account_l10n_ec_sri_payment_tree" model="ir.actions.act_window">
        <field name="name">Payment Methods SRI</field>
        <field name="res_model">l10n_ec.sri.payment</field>
        <field name="view_mode">tree,form</field>
    </record>

    <menuitem id="menu_action_account_l10n_ec_sri_payment" action="action_account_l10n_ec_sri_payment_tree"
              groups="account.group_account_manager" parent="account.account_invoicing_menu" sequence="3"/>
</odoo>

```

## File: views\l10n_latam_document_type_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_document_type_conf_form" model="ir.ui.view">
        <field name="name">view.document.type.conf.form</field>
        <field name="model">l10n_latam.document.type</field>
        <field name="inherit_id" ref="l10n_latam_invoice_document.view_document_type_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group" position="after">
                <group string="Ecuadorian Configuration" col="2" colspan="2">
                    <field name="l10n_ec_check_format"/>
                </group>
            </xpath>
        </field>
    </record>

    <record id="view_document_type_conf_tree" model="ir.ui.view">
        <field name="name">view.document.type.conf.tree</field>
        <field name="model">l10n_latam.document.type</field>
        <field name="inherit_id" ref="l10n_latam_invoice_document.view_document_type_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='internal_type']" position="after">
                <field name="l10n_ec_check_format"/>
            </xpath>
        </field>
    </record>

</odoo>

```

