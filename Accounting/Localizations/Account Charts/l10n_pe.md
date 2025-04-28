# Odoo Module: l10n_pe

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: README.rst

```rst
Minimal set of accounts to start to work in Perú.
=================================================

The usage of this CoA must refer to the official documentation on MEF.

https://www.mef.gob.pe/contenidos/conta_publ/documentac/VERSION_MODIFICADA_PCG_EMPRESARIAL.pdf
https://www.mef.gob.pe/contenidos/conta_publ/documentac/PCGE_2019.pdf

All the legal references can be found here.

http://www.sunat.gob.pe/legislacion/general/index.html

Considerations.
===============

Chart of account:
-----------------

The tree of the CoA is done using account groups, the most common accounts 
are available within their group, if you want to create a new account use 
the groups as reference. 

Taxes:
------

'IGV': {'name': 'VAT', 'code': 'S'},
'IVAP': {'name': 'VAT', 'code': ''},
'ISC': {'name': 'EXC', 'code': 'S'},
'ICBPER': {'name': 'OTH', 'code': ''},
'EXP': {'name': 'FRE', 'code': 'G'},
'GRA': {'name': 'FRE', 'code': 'Z'},
'EXO': {'name': 'VAT', 'code': 'E'},
'INA': {'name': 'FRE', 'code': 'O'},
'OTROS': {'name': 'OTH', 'code': 'S'},

We added on this module the 3 concepts in taxes (necessary for the EDI
signature)

EDI Peruvian Code: used to select the type of tax from the SUNAT
EDI UNECE code: used to select the type of tax based on the United Nations
Economic Commission
EDI Affect. Reason: type of affectation to the IGV based on the Catalog 07

Products:
---------

Code for products to be used in the EDI are availables here, in order to decide
which tax use due to which code following this reference and python code:

https://docs.google.com/spreadsheets/d/1f1fxV8uGhA-Qz9-R1L1-dJirZ8xi3Wfg/edit#gid=662652969

**Nota:**
---------

**RELACIÓN ENTRE EL PCGE Y LA LEGISLACIÓN TRIBUTARIA:**

Este PCGE ha sido preparado como una herramienta de carácter contable, para acumular información que
requiere ser expuesta en el cuerpo de los estados financieros o en las notas a dichos estados. Esa acumulación se
efectúa en los libros o registros contables, cuya denominación y naturaleza depende de las actividades que se
efectúen, y que permiten acciones de verificación, control y seguimiento. Las NIIF completas y la NIIF PYMES no
contienen prescripciones sobre teneduría de libros, y consecuentemente, sobre los libros y otros registros
de naturaleza contable. Por otro lado, si bien es cierto la contabilidad es también un insumo, dentro de otros, para
labores de cumplimiento tributario, este PCGE no ha sido elaborado para satisfacer prescripciones tributarias ni su
verificación. No obstante ello, donde no hubo oposición entre la contabilidad financiera prescrita por las NIIF y
la legislación tributaria, este PCGE ha incluido subcuentas, divisionarias y sub divisionarias, para
distinguir componentes con validez tributaria, dentro del conjunto de componentes que corresponden a una
perspectiva contable íntegramente. Por lo tanto, este PCGE no debe ser considerado en ningún aspecto
como una guía con propósitos distintos del contable.

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import demo

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Peru - Accounting',
    "version": "3.0",
    'summary': "PCGE Simplified",
    'category': 'Accounting/Localizations/Account Charts',
    'author': 'Vauxoo, Odoo',
    'website': 'https://www.odoo.com/documentation/16.0/applications/finance/accounting/fiscal_localizations/localizations/peru.html',
    'license': 'LGPL-3',
    'depends': [
        'base_vat',
        'base_address_extended',
        'l10n_latam_base',
        'l10n_latam_invoice_document',
        'account_debit_note',
    ],
    'data': [
        'security/ir.model.access.csv',
        'views/account_tax_view.xml',
        'data/l10n_pe_chart_data.xml',
        'data/account.group.template.csv',
        'data/account.account.template.csv',
        'data/l10n_pe_chart_post_data.xml',
        'data/account_tax_group_data.xml',
        'data/account_tax_data.xml',
        'data/fiscal_position_data.xml',
        'data/l10n_latam_document_type_data.xml',
        'data/account_chart_template_data.xml',
        'data/res.city.csv',
        'data/l10n_pe.res.city.district.csv',
        'data/res_country_data.xml',
        'data/l10n_latam_identification_type_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
        'demo/demo_partner.xml',
    ],
}

```

## File: data\account.account.template.csv

```csv
id,code,name,account_type,chart_template_id:id,reconcile
chart0111,0111,Bienes y valores entregados,off_balance,pe_chart_template,False
chart0112,0112,Derechos sobre instrumentos financiero,off_balance,pe_chart_template,False
chart0113,0113,Otras cuentas de orden deudora,off_balance,pe_chart_template,False
chart0114,0114,Deudoras por contra,off_balance,pe_chart_template,False
chart0126,0126,Bienes y valores recibido,off_balance,pe_chart_template,False
chart0127,0127,Compromisos sobre instrumentos financieros,off_balance,pe_chart_template,False
chart0128,0128,Otras cuentas de orden acreedoras,off_balance,pe_chart_template,False
chart0129,0129,Acreedoras por contra,off_balance,pe_chart_template,False
chart0211,0211,Bienes y valores entregados,off_balance,pe_chart_template,False
chart0213,0213,Derechos sobre instrumentos financieros,off_balance,pe_chart_template,False
chart0214,0214,Otras cuentas de orden deudoras,off_balance,pe_chart_template,False
chart0215,0215,Contrapartida cuentas de orden deudora,off_balance,pe_chart_template,False
chart0226,0226,Bienes y valores recibidos,off_balance,pe_chart_template,False
chart0227,0227,Compromisos sobre instrumentos financieros,off_balance,pe_chart_template,False
chart0228,0228,Otras cuentas de orden acreedoras,off_balance,pe_chart_template,False
chart0229,0229,Contrapartidacuentas de orden acreedoras,off_balance,pe_chart_template,False
chart101,101,Caja,asset_cash,pe_chart_template,False
chart102,102,Fondos fijos,asset_cash,pe_chart_template,False
chart1031,1031,Fondos fijos - Efectivo en tránsito,asset_cash,pe_chart_template,False
chart1032,1032,Fondos fijos - Cheques en tránsito,asset_cash,pe_chart_template,False
chart1041,1041,Cuentas corrientes en instituciones financieras - Cuentas corrientes operativas,asset_cash,pe_chart_template,False
chart1042,1042,Cuentas corrientes en instituciones financieras - Cuentas corrientes para fines específicos,asset_cash,pe_chart_template,False
chart1051,1051,Otros equivalentes de efectivo - Otro equivalentes de efectivo,asset_cash,pe_chart_template,False
chart1061,1061,Depósitos en instituciones financieras - Depósitos de ahorro,asset_cash,pe_chart_template,False
chart1062,1062,Depósitos en instituciones financieras - Depósitos a plazo,asset_cash,pe_chart_template,False
chart1071,1071,Fondos sujetos a restricción - Fondos en garantía,asset_cash,pe_chart_template,False
chart1072,1072,Fondos sujetos a restricción - Fondos retenidos por mandato de la autoridad,asset_cash,pe_chart_template,False
chart1073,1073,Fondos sujetos a restricción - Otros fondos sujetos a restricción,asset_cash,pe_chart_template,False
chart11111,11111,Inversiones mantenidas para negociación - Valores emitidos o garantizados por el Estado - Costo,asset_receivable,pe_chart_template,True
chart11112,11112,Inversiones mantenidas para negociación - Valores emitidos o garantizados por el Estado - Valor Razonable,asset_receivable,pe_chart_template,True
chart11121,11121,Inversiones mantenidas para negociación - Valores emitidos por el sistema financiero - Costo,asset_receivable,pe_chart_template,True
chart11122,11122,Inversiones mantenidas para negociación - Valores emitidos por el sistema financiero - Valor Razonable,asset_receivable,pe_chart_template,True
chart11131,11131,Inversiones mantenidas para negociación - Valores emitidos por entidades - Costo,asset_receivable,pe_chart_template,True
chart11132,11132,Inversiones mantenidas para negociación - Valores emitidos por entidades - Valor Razonable,asset_receivable,pe_chart_template,True
chart11141,11141,Inversiones mantenidas para negociación - Otros títulos representativos de deuda - Costo,asset_receivable,pe_chart_template,True
chart11142,11142,Inversiones mantenidas para negociación - Otros títulos representativos de deuda - Valor Razonable,asset_receivable,pe_chart_template,True
chart11151,11151,Inversiones mantenidas para negociación - Participaciones en entidades - Costo,asset_receivable,pe_chart_template,True
chart11152,11152,Inversiones mantenidas para negociación - Participaciones en entidades - Valor Razonable,asset_receivable,pe_chart_template,True
chart11211,11211,Otras inversiones financieras - Otras inversiones financieras - Costo,asset_receivable,pe_chart_template,True
chart11212,11212,Otras inversiones financieras - Otras inversiones financieras - Valor Razonable,asset_receivable,pe_chart_template,True
chart11311,11311,Activos financieros – Acuerdo de compra - Inversiones mantenidas para negociación – Acuerdo de compra - Costo,asset_receivable,pe_chart_template,True
chart11312,11312,Activos financieros – Acuerdo de compra - Inversiones mantenidas para negociación – Acuerdo de compra - Valor Razonable,asset_receivable,pe_chart_template,True
chart11321,11321,Activos financieros – Acuerdo de compra - Otras inversiones financieras - Costo,asset_receivable,pe_chart_template,True
chart11322,11322,Activos financieros – Acuerdo de compra - Otras inversiones financieras - Valor Razonable,asset_receivable,pe_chart_template,True
chart1211,1211,"Facturas, boletas y otros comprobantes por cobrar - No emitidas",asset_receivable,pe_chart_template,True
chart1212,1212,"Facturas, boletas y otros comprobantes por cobrar - Emitidas en cartera",asset_receivable,pe_chart_template,True
chart1213,1213,"Facturas, boletas y otros comprobantes por cobrar - En cobranza",asset_receivable,pe_chart_template,True
chart1214,1214,"Facturas, boletas y otros comprobantes por cobrar - En descuento",asset_receivable,pe_chart_template,True
chart1215,1215,"Facturas, boletas y otros comprobantes por cobrar - En cobranza PoS",asset_receivable,pe_chart_template,True
chart122,122,Anticipos de clientes,asset_receivable,pe_chart_template,True
chart1232,1232,Letras por cobrar - En cartera,asset_receivable,pe_chart_template,True
chart1233,1233,Letras por cobrar - En cobranza,asset_receivable,pe_chart_template,True
chart1234,1234,Letras por cobrar - En descuento,asset_receivable,pe_chart_template,True
chart1311,1311,"Facturas, boletas y otros comprobantes por cobrar - No emitidas",asset_receivable,pe_chart_template,True
chart1312,1312,"Facturas, boletas y otros comprobantes por cobrar - En cartera",asset_receivable,pe_chart_template,True
chart1313,1313,"Facturas, boletas y otros comprobantes por cobrar - En cobranza",asset_receivable,pe_chart_template,True
chart1314,1314,"Facturas, boletas y otros comprobantes por cobrar - En descuento",asset_receivable,pe_chart_template,True
chart1321,1321,Anticipos recibidos - Anticipos recibidos,asset_receivable,pe_chart_template,True
chart1331,1331,Letras por cobrar - En cartera,asset_receivable,pe_chart_template,True
chart1332,1332,Letras por cobrar - En cobranza,asset_receivable,pe_chart_template,True
chart1333,1333,Letras por cobrar - En descuento,asset_receivable,pe_chart_template,True
chart1411,1411,Personal - Préstamos,asset_receivable,pe_chart_template,True
chart1412,1412,Personal - Adelanto de remuneraciones,asset_receivable,pe_chart_template,True
chart1413,1413,Personal - Entregas a rendir cuenta,asset_receivable,pe_chart_template,True
chart1419,1419,Personal - Otras cuentas por cobrar al personal,asset_receivable,pe_chart_template,True
chart1421,1421,Accionistas (o socios) - Suscripciones por cobrar a socios o accionistas,asset_receivable,pe_chart_template,True
chart1422,1422,Accionistas (o socios) - Préstamos,asset_receivable,pe_chart_template,True
chart1431,1431,Directores - Préstamos,asset_receivable,pe_chart_template,True
chart1432,1432,Directores - Adelanto de dietas,asset_receivable,pe_chart_template,True
chart1433,1433,Directores - Entregas a rendir cuenta,asset_receivable,pe_chart_template,True
chart149,149,Diversas,asset_receivable,pe_chart_template,True
chart1611,1611,Préstamos - Con garantía,asset_receivable,pe_chart_template,True
chart1612,1612,Préstamos - Sin garantía,asset_receivable,pe_chart_template,True
chart1621,1621,Reclamaciones a terceros - Compañías aseguradoras,asset_receivable,pe_chart_template,True
chart1622,1622,Reclamaciones a terceros - Transportadoras,asset_receivable,pe_chart_template,True
chart1623,1623,Reclamaciones a terceros - Servicios públicos,asset_receivable,pe_chart_template,True
chart1624,1624,Reclamaciones a terceros - Tributos,asset_receivable,pe_chart_template,True
chart1629,1629,Reclamaciones a terceros - Otras,asset_receivable,pe_chart_template,True
chart1631,1631,"Intereses, regalías y dividendos - Intereses",asset_receivable,pe_chart_template,True
chart1632,1632,"Intereses, regalías y dividendos - Regalías",asset_receivable,pe_chart_template,True
chart1633,1633,"Intereses, regalías y dividendos - Dividendos",asset_receivable,pe_chart_template,True
chart1641,1641,Depósitos otorgados en garantía - Préstamos de instituciones financieras,asset_receivable,pe_chart_template,True
chart1642,1642,Depósitos otorgados en garantía - Préstamos de instituciones no financieras,asset_receivable,pe_chart_template,True
chart1643,1643,Depósitos otorgados en garantía - Depósitos en garantía por alquileres,asset_receivable,pe_chart_template,True
chart1649,1649,Depósitos otorgados en garantía - Otros depósitos en garantía,asset_receivable,pe_chart_template,True
chart1651,1651,Venta de activo inmovilizado - Inversión mobiliaria,asset_receivable,pe_chart_template,True
chart1652,1652,Venta de activo inmovilizado - Propiedades de inversión,asset_receivable,pe_chart_template,True
chart1653,1653,"Venta de activo inmovilizado - Propiedad, planta y equipo",asset_receivable,pe_chart_template,True
chart1654,1654,Venta de activo inmovilizado - Intangibles,asset_receivable,pe_chart_template,True
chart1655,1655,Venta de activo inmovilizado - Activos biológicos,asset_receivable,pe_chart_template,True
chart1659,1659,Venta de activo inmovilizado - Otros activos inmovilizados,asset_receivable,pe_chart_template,True
chart16611,16611,Activos por instrumentos financieros - Instrumentos financieros primarios - Costo,asset_receivable,pe_chart_template,True
chart16612,16612,Activos por instrumentos financieros - Instrumentos financieros primarios - Valor razonable,asset_receivable,pe_chart_template,True
chart16621,16621,Activos por instrumentos financieros - Instrumentos financieros derivados - Costo,asset_receivable,pe_chart_template,True
chart16622,16622,Activos por instrumentos financieros - Instrumentos financieros derivados - Valor razonable,asset_receivable,pe_chart_template,True
chart1671,1671,Tributos por acreditar - Pagos a cuenta del impuesto a la renta,asset_receivable,pe_chart_template,True
chart1672,1672,Tributos por acreditar - Pagos a cuenta de ITAN,asset_receivable,pe_chart_template,True
chart1673,1673,Tributos por acreditar - IGV por acreditar en compras,asset_receivable,pe_chart_template,True
chart1674,1674,Tributos por acreditar - IGV por acreditar no domiciliados,asset_receivable,pe_chart_template,True
chart1675,1675,Tributos por acreditar - Obras por impuestos,asset_receivable,pe_chart_template,True
chart1691,1691,Otras cuentas por cobrar diversas - Entregas a rendir cuenta a terceros,asset_receivable,pe_chart_template,True
chart1699,1699,Otras cuentas por cobrar diversas - Otras cuentas por cobrar diversas,asset_receivable,pe_chart_template,True
chart1711,1711,Préstamos - Con garantía,asset_receivable,pe_chart_template,True
chart1712,1712,Préstamos - Sin garantía,asset_receivable,pe_chart_template,True
chart1731,1731,"Intereses, regalías y dividendos - Intereses",asset_receivable,pe_chart_template,True
chart1732,1732,"Intereses, regalías y dividendos - Regalías",asset_receivable,pe_chart_template,True
chart1733,1733,"Intereses, regalías y dividendos - Dividendos",asset_receivable,pe_chart_template,True
chart1741,1741,Depósitos otorgados en garantía - Préstamos de instituciones financieras,asset_receivable,pe_chart_template,True
chart1742,1742,Depósitos otorgados en garantía - Préstamos de instituciones no financieras,asset_receivable,pe_chart_template,True
chart1743,1743,Depósitos otorgados en garantía - Depósitos en garantía por alquileres,asset_receivable,pe_chart_template,True
chart1749,1749,Depósitos otorgados en garantía - Otros depósitos en garantía,asset_receivable,pe_chart_template,True
chart1751,1751,Venta de activo inmovilizado - Inversión mobiliaria,asset_receivable,pe_chart_template,True
chart1752,1752,Venta de activo inmovilizado - Propiedades de inversión,asset_receivable,pe_chart_template,True
chart1753,1753,"Venta de activo inmovilizado - Propiedad, planta y equipo",asset_receivable,pe_chart_template,True
chart1754,1754,Venta de activo inmovilizado - Intangibles,asset_receivable,pe_chart_template,True
chart1755,1755,Venta de activo inmovilizado - Activos biológicos,asset_receivable,pe_chart_template,True
chart1759,1759,Venta de activo inmovilizado - Otros activos inmovilizados,asset_receivable,pe_chart_template,True
chart17611,17611,Activos por instrumentos financieros - Instrumentos financieros primarios - Costo,asset_receivable,pe_chart_template,True
chart17612,17612,Activos por instrumentos financieros - Instrumentos financieros primarios - Valor razonable,asset_receivable,pe_chart_template,True
chart17621,17621,Activos por instrumentos financieros - Instrumentos financieros derivados - Costo,asset_receivable,pe_chart_template,True
chart17622,17622,Activos por instrumentos financieros - Instrumentos financieros derivados - Valor razonable,asset_receivable,pe_chart_template,True
chart179,179,Otras cuentas por cobrar diversas,asset_receivable,pe_chart_template,True
chart181,181,Costos financieros,asset_prepayments,pe_chart_template,False
chart182,182,Seguros,asset_prepayments,pe_chart_template,False
chart183,183,Alquileres,asset_prepayments,pe_chart_template,False
chart184,184,Primas pagadas por opciones,asset_prepayments,pe_chart_template,False
chart185,185,Mantenimiento de activos inmovilizados,asset_prepayments,pe_chart_template,False
chart189,189,Otros gastos contratados por anticipado,asset_prepayments,pe_chart_template,False
chart1911,1911,"Cuentas por cobrar comerciales – Terceros - Facturas, boletas y otros comprobantes por cobrar",asset_receivable,pe_chart_template,True
chart1913,1913,Cuentas por cobrar comerciales – Terceros - Letras por cobrar,asset_receivable,pe_chart_template,True
chart1921,1921,"Cuentas por cobrar comerciales – Relacionadas - Facturas, boletas y otros comprobantes por cobrar",asset_receivable,pe_chart_template,True
chart1923,1923,Cuentas por cobrar comerciales – Relacionadas - Letras por cobrar,asset_receivable,pe_chart_template,True
chart1931,1931,"Cuentas por cobrar al personal, a los accionistas (socios) y directores - Personal",asset_receivable,pe_chart_template,True
chart1932,1932,"Cuentas por cobrar al personal, a los accionistas (socios) y directores - Accionistas (o socios)",asset_receivable,pe_chart_template,True
chart1933,1933,"Cuentas por cobrar al personal, a los accionistas (socios) y directores - Directores",asset_receivable,pe_chart_template,True
chart1939,1939,"Cuentas por cobrar al personal, a los accionistas (socios) y directores - Diversas",asset_receivable,pe_chart_template,True
chart1941,1941,Cuentas por cobrar diversas – Terceros - Préstamos,asset_receivable,pe_chart_template,True
chart1942,1942,Cuentas por cobrar diversas – Terceros - Reclamaciones a terceros,asset_receivable,pe_chart_template,True
chart1943,1943,"Cuentas por cobrar diversas – Terceros - Intereses, regalías y dividendos",asset_receivable,pe_chart_template,True
chart1944,1944,Cuentas por cobrar diversas – Terceros - Depósitos otorgados en garantía,asset_receivable,pe_chart_template,True
chart1945,1945,Cuentas por cobrar diversas – Terceros - Venta de activo inmovilizado,asset_receivable,pe_chart_template,True
chart1946,1946,Cuentas por cobrar diversas – Terceros - Activos por instrumentos financieros,asset_receivable,pe_chart_template,True
chart1949,1949,Cuentas por cobrar diversas – Terceros - Otras cuentas por cobrar diversas,asset_receivable,pe_chart_template,True
chart1951,1951,Cuentas por cobrar diversas – Relacionadas - Préstamos,asset_receivable,pe_chart_template,True
chart1953,1953,"Cuentas por cobrar diversas – Relacionadas - Intereses, regalías y dividendos",asset_receivable,pe_chart_template,True
chart1954,1954,Cuentas por cobrar diversas – Relacionadas - Depósitos otorgados en garantía,asset_receivable,pe_chart_template,True
chart1955,1955,Cuentas por cobrar diversas – Relacionadas - Venta de activo inmovilizado,asset_receivable,pe_chart_template,True
chart1956,1956,Cuentas por cobrar diversas – Relacionadas - Activos por instrumentos financieros,asset_receivable,pe_chart_template,True
chart1959,1959,Cuentas por cobrar diversas – Relacionadas - Otras cuentas por cobrar diversas,asset_receivable,pe_chart_template,True
chart20111,20111,Mercaderías - Mercaderías - Costo,asset_current,pe_chart_template,False
chart20114,20114,Mercaderías - Mercaderías - Valor razonable,asset_current,pe_chart_template,False
chart21111,21111,Productos terminados - Productos terminados - Costo,asset_current,pe_chart_template,False
chart21113,21113,Productos terminados - Productos terminados - Costo de financiación,asset_current,pe_chart_template,False
chart21114,21114,Productos terminados - Productos terminados - Valor razonable,asset_current,pe_chart_template,False
chart21511,21511,Inventario de servicios terminados - Servicios terminados - Costo,asset_current,pe_chart_template,False
chart221,221,Subproductos,asset_current,pe_chart_template,False
chart222,222,Desechos y desperdicios,asset_current,pe_chart_template,False
chart23111,23111,Productos en proceso - Productos en proceso - Costo,asset_current,pe_chart_template,False
chart23113,23113,Productos en proceso - Productos en proceso - Costo de financiación,asset_current,pe_chart_template,False
chart23511,23511,Inventario de servicios en proceso - Servicios en proceso - Costo,asset_current,pe_chart_template,False
chart24111,24111,Materias primas - Materias primas - Costo,asset_current,pe_chart_template,False
chart24114,24114,Materias primas - Materias primas - Valor razonable,asset_current,pe_chart_template,False
chart251,251,Materiales auxiliares,asset_current,pe_chart_template,False
chart2521,2521,Suministros - Combustibles,asset_current,pe_chart_template,False
chart2522,2522,Suministros - Lubricantes,asset_current,pe_chart_template,False
chart2523,2523,Suministros - Energía,asset_current,pe_chart_template,False
chart2524,2524,Suministros - Otros suministros,asset_current,pe_chart_template,False
chart253,253,Repuestos,asset_current,pe_chart_template,False
chart261,261,Envases,asset_current,pe_chart_template,False
chart262,262,Embalajes,asset_current,pe_chart_template,False
chart27111,27111,Propiedades de inversión - Terrenos - Costo,asset_current,pe_chart_template,False
chart27112,27112,Propiedades de inversión - Terrenos - Revaluación,asset_current,pe_chart_template,False
chart27114,27114,Propiedades de inversión - Terrenos - Valor razonable,asset_current,pe_chart_template,False
chart27121,27121,Propiedades de inversión - Edificaciones - Costo,asset_current,pe_chart_template,False
chart27122,27122,Propiedades de inversión - Edificaciones - Revaluación,asset_current,pe_chart_template,False
chart27123,27123,Propiedades de inversión - Edificaciones - Costos de financiación,asset_current,pe_chart_template,False
chart27124,27124,Propiedades de inversión - Edificaciones - Valor razonable,asset_current,pe_chart_template,False
chart27201,27201,"Propiedad, planta y equipo - Planta productora en producción - Costo",asset_current,pe_chart_template,False
chart27202,27202,"Propiedad, planta y equipo - Planta productora en producción - Revaluación",asset_current,pe_chart_template,False
chart27203,27203,"Propiedad, planta y equipo - Planta productora en producción - Costo de financiación",asset_current,pe_chart_template,False
chart27204,27204,"Propiedad, planta y equipo - Planta productora en producción - Valor razonable",asset_current,pe_chart_template,False
chart27211,27211,"Propiedad, planta y equipo - Planta productora en desarrollo - Costo",asset_current,pe_chart_template,False
chart27212,27212,"Propiedad, planta y equipo - Planta productora en desarrollo - Revaluación",asset_current,pe_chart_template,False
chart27213,27213,"Propiedad, planta y equipo - Planta productora en desarrollo - Costo de financiación",asset_current,pe_chart_template,False
chart27214,27214,"Propiedad, planta y equipo - Planta productora en desarrollo - Valor razonable",asset_current,pe_chart_template,False
chart27221,27221,"Propiedad, planta y equipo - Terrenos - Costo",asset_current,pe_chart_template,False
chart27222,27222,"Propiedad, planta y equipo - Terrenos - Revaluación",asset_current,pe_chart_template,False
chart27231,27231,"Propiedad, planta y equipo - Edificaciones - Costo",asset_current,pe_chart_template,False
chart27232,27232,"Propiedad, planta y equipo - Edificaciones - Revaluación",asset_current,pe_chart_template,False
chart27233,27233,"Propiedad, planta y equipo - Edificaciones - Costo de financiación",asset_current,pe_chart_template,False
chart27241,27241,"Propiedad, planta y equipo - Maquinarias y equipos de explotación - Costo",asset_current,pe_chart_template,False
chart27242,27242,"Propiedad, planta y equipo - Maquinarias y equipos de explotación - Revaluación",asset_current,pe_chart_template,False
chart27243,27243,"Propiedad, planta y equipo - Maquinarias y equipos de explotación - Costo de financiación",asset_current,pe_chart_template,False
chart27251,27251,"Propiedad, planta y equipo - Unidades de transporte - Costo",asset_current,pe_chart_template,False
chart27252,27252,"Propiedad, planta y equipo - Unidades de transporte - Revaluación",asset_current,pe_chart_template,False
chart27261,27261,"Propiedad, planta y equipo - Muebles y enseres - Costo",asset_current,pe_chart_template,False
chart27262,27262,"Propiedad, planta y equipo - Muebles y enseres - Revaluación",asset_current,pe_chart_template,False
chart27271,27271,"Propiedad, planta y equipo - Equipos diversos - Costo",asset_current,pe_chart_template,False
chart27272,27272,"Propiedad, planta y equipo - Equipos diversos - Revaluación",asset_current,pe_chart_template,False
chart27281,27281,"Propiedad, planta y equipo - Herramientas y unidades de reemplazo - Costo",asset_current,pe_chart_template,False
chart27282,27282,"Propiedad, planta y equipo - Herramientas y unidades de reemplazo - Revaluación",asset_current,pe_chart_template,False
chart27291,27291,"Propiedad, planta y equipo - Obras en curso - Costo",asset_current,pe_chart_template,False
chart27292,27292,"Propiedad, planta y equipo - Obras en curso - Revaluación",asset_current,pe_chart_template,False
chart27311,27311,"Intangibles - Concesiones, licencias y derechos - Costo",asset_current,pe_chart_template,False
chart27312,27312,"Intangibles - Concesiones, licencias y derechos - Revaluación",asset_current,pe_chart_template,False
chart27321,27321,Intangibles - Patentes y propiedad industrial - Costo,asset_current,pe_chart_template,False
chart27322,27322,Intangibles - Patentes y propiedad industrial - Revaluación,asset_current,pe_chart_template,False
chart27331,27331,Intangibles - Programas de computadora (software) - Costo,asset_current,pe_chart_template,False
chart27332,27332,Intangibles - Programas de computadora (software) - Revaluación,asset_current,pe_chart_template,False
chart27341,27341,Intangibles - Costos de exploración y desarrollo - Costo,asset_current,pe_chart_template,False
chart27342,27342,Intangibles - Costos de exploración y desarrollo - Revaluación,asset_current,pe_chart_template,False
chart27351,27351,"Intangibles - Fórmulas, diseños y prototipos - Costo",asset_current,pe_chart_template,False
chart27352,27352,"Intangibles - Fórmulas, diseños y prototipos - Revaluación",asset_current,pe_chart_template,False
chart27391,27391,Intangibles - Otros activos intangibles - Costo,asset_current,pe_chart_template,False
chart27392,27392,Intangibles - Otros activos intangibles - Revaluación,asset_current,pe_chart_template,False
chart27411,27411,Activos biológicos - Activos biológicos en producción - Costo,asset_current,pe_chart_template,False
chart27413,27413,Activos biológicos - Activos biológicos en producción - Costos de financiación,asset_current,pe_chart_template,False
chart27414,27414,Activos biológicos - Activos biológicos en producción - Valor razonable,asset_current,pe_chart_template,False
chart27421,27421,Activos biológicos - Activos biológicos en desarrollo - Costo,asset_current,pe_chart_template,False
chart27423,27423,Activos biológicos - Activos biológicos en desarrollo - Costos de financiación,asset_current,pe_chart_template,False
chart27424,27424,Activos biológicos - Activos biológicos en desarrollo - Valor razonable,asset_current,pe_chart_template,False
chart27521,27521,Depreciación acumulada – Propiedades de inversión - Edificaciones - Costo,asset_current,pe_chart_template,False
chart27522,27522,Depreciación acumulada – Propiedades de inversión - Edificaciones - Revaluación,asset_current,pe_chart_template,False
chart27523,27523,Depreciación acumulada – Propiedades de inversión - Edificaciones - Costo de financiación,asset_current,pe_chart_template,False
chart27601,27601,"Depreciación acumulada – Propiedad, planta y equipo - Planta productora en producción - Costo",asset_current,pe_chart_template,False
chart27602,27602,"Depreciación acumulada – Propiedad, planta y equipo - Planta productora en producción - Revaluación",asset_current,pe_chart_template,False
chart27603,27603,"Depreciación acumulada – Propiedad, planta y equipo - Planta productora en producción - Costo de financiación",asset_current,pe_chart_template,False
chart27604,27604,"Depreciación acumulada – Propiedad, planta y equipo - Planta productora en producción - Valor razonable",asset_current,pe_chart_template,False
chart27621,27621,"Depreciación acumulada – Propiedad, planta y equipo - Edificaciones - Costo",asset_current,pe_chart_template,False
chart27622,27622,"Depreciación acumulada – Propiedad, planta y equipo - Edificaciones - Revaluación",asset_current,pe_chart_template,False
chart27623,27623,"Depreciación acumulada – Propiedad, planta y equipo - Edificaciones - Costo de financiación",asset_current,pe_chart_template,False
chart27631,27631,"Depreciación acumulada – Propiedad, planta y equipo - Maquinarias y equipo de explotación - Costo",asset_current,pe_chart_template,False
chart27632,27632,"Depreciación acumulada – Propiedad, planta y equipo - Maquinarias y equipo de explotación - Revaluación",asset_current,pe_chart_template,False
chart27633,27633,"Depreciación acumulada – Propiedad, planta y equipo - Maquinarias y equipo de explotación - Costo de financiación",asset_current,pe_chart_template,False
chart27641,27641,"Depreciación acumulada – Propiedad, planta y equipo - Unidades de transporte - Costo",asset_current,pe_chart_template,False
chart27642,27642,"Depreciación acumulada – Propiedad, planta y equipo - Unidades de transporte - Revaluación",asset_current,pe_chart_template,False
chart27651,27651,"Depreciación acumulada – Propiedad, planta y equipo - Muebles y enseres - Costo",asset_current,pe_chart_template,False
chart27652,27652,"Depreciación acumulada – Propiedad, planta y equipo - Muebles y enseres - Revaluación",asset_current,pe_chart_template,False
chart27661,27661,"Depreciación acumulada – Propiedad, planta y equipo - Equipos diversos - Costo",asset_current,pe_chart_template,False
chart27662,27662,"Depreciación acumulada – Propiedad, planta y equipo - Equipos diversos - Revaluación",asset_current,pe_chart_template,False
chart27671,27671,"Depreciación acumulada – Propiedad, planta y equipo - Herramientas y unidades de reemplazo - Costo",asset_current,pe_chart_template,False
chart27672,27672,"Depreciación acumulada – Propiedad, planta y equipo - Herramientas y unidades de reemplazo - Revaluación",asset_current,pe_chart_template,False
chart27711,27711,"Amortización acumulada – Intangibles - Concesiones, licencias y derechos - Costo",asset_current,pe_chart_template,False
chart27712,27712,"Amortización acumulada – Intangibles - Concesiones, licencias y derechos - Revaluación",asset_current,pe_chart_template,False
chart27721,27721,Amortización acumulada – Intangibles - Patentes y propiedad industrial - Costo,asset_current,pe_chart_template,False
chart27722,27722,Amortización acumulada – Intangibles - Patentes y propiedad industrial - Revaluación,asset_current,pe_chart_template,False
chart27731,27731,Amortización acumulada – Intangibles - Programas de computadora (software) - Costo,asset_current,pe_chart_template,False
chart27732,27732,Amortización acumulada – Intangibles - Programas de computadora (software) - Revaluación,asset_current,pe_chart_template,False
chart27741,27741,Amortización acumulada – Intangibles - Costos de exploración y desarrollo - Costo,asset_current,pe_chart_template,False
chart27742,27742,Amortización acumulada – Intangibles - Costos de exploración y desarrollo - Revaluación,asset_current,pe_chart_template,False
chart27751,27751,"Amortización acumulada – Intangibles - Fórmulas, diseños y prototipos - Costo",asset_current,pe_chart_template,False
chart27752,27752,"Amortización acumulada – Intangibles - Fórmulas, diseños y prototipos - Revaluación",asset_current,pe_chart_template,False
chart27791,27791,Amortización acumulada – Intangibles - Otros activos intangibles - Costo,asset_current,pe_chart_template,False
chart27792,27792,Amortización acumulada – Intangibles - Otros activos intangibles - Revaluación,asset_current,pe_chart_template,False
chart27811,27811,Depreciación acumulada – Activos biológicos - Activos biológicos en producción - Costo,asset_current,pe_chart_template,False
chart27813,27813,Depreciación acumulada – Activos biológicos - Activos biológicos en producción - Costo de financiación,asset_current,pe_chart_template,False
chart27821,27821,Depreciación acumulada – Activos biológicos - Activos biológicos en desarrollo - Costo,asset_current,pe_chart_template,False
chart27823,27823,Depreciación acumulada – Activos biológicos - Activos biológicos en desarrollo - Costo de financiación,asset_current,pe_chart_template,False
chart27910,27910,Desvalorización acumulada - Propiedad de inversión - Planta productora en producción,asset_current,pe_chart_template,False
chart27911,27911,Desvalorización acumulada - Propiedad de inversión - Planta productora en desarrollo,asset_current,pe_chart_template,False
chart27912,27912,Desvalorización acumulada - Propiedad de inversión - Terrenos,asset_current,pe_chart_template,False
chart27913,27913,Desvalorización acumulada - Propiedad de inversión - Edificaciones,asset_current,pe_chart_template,False
chart27930,27930,"Desvalorización acumulada - Propiedad, planta y equipo - Plantas productoras en producción",asset_current,pe_chart_template,False
chart27931,27931,"Desvalorización acumulada - Propiedad, planta y equipo - Planta productora en desarrollo",asset_current,pe_chart_template,False
chart27932,27932,"Desvalorización acumulada - Propiedad, planta y equipo - Terrenos",asset_current,pe_chart_template,False
chart27933,27933,"Desvalorización acumulada - Propiedad, planta y equipo - Edificaciones",asset_current,pe_chart_template,False
chart27934,27934,"Desvalorización acumulada - Propiedad, planta y equipo - Maquinarias y equipos de explotación",asset_current,pe_chart_template,False
chart27935,27935,"Desvalorización acumulada - Propiedad, planta y equipo - Unidades de transporte",asset_current,pe_chart_template,False
chart27936,27936,"Desvalorización acumulada - Propiedad, planta y equipo - Muebles y enseres",asset_current,pe_chart_template,False
chart27937,27937,"Desvalorización acumulada - Propiedad, planta y equipo - Equipos diversos",asset_current,pe_chart_template,False
chart27938,27938,"Desvalorización acumulada - Propiedad, planta y equipo - Herramientas y unidades de reemplazo",asset_current,pe_chart_template,False
chart27941,27941,"Desvalorización acumulada - Intangibles - Concesiones, licencias y otros derechos",asset_current,pe_chart_template,False
chart27942,27942,Desvalorización acumulada - Intangibles - Patentes y propiedad industrial,asset_current,pe_chart_template,False
chart27943,27943,Desvalorización acumulada - Intangibles - Programas de computadora (software),asset_current,pe_chart_template,False
chart27944,27944,Desvalorización acumulada - Intangibles - Costos de exploración y desarrollo,asset_current,pe_chart_template,False
chart27945,27945,"Desvalorización acumulada - Intangibles - Fórmulas, diseños y prototipos",asset_current,pe_chart_template,False
chart27949,27949,Desvalorización acumulada - Intangibles - Otros activos intangibles,asset_current,pe_chart_template,False
chart27951,27951,Desvalorización acumulada - Activos biológicos - Activos biológicos en producción,asset_current,pe_chart_template,False
chart27952,27952,Desvalorización acumulada - Activos biológicos - Activos biológicos en desarrollo,asset_current,pe_chart_template,False
chart281,281,Mercaderías,asset_current,pe_chart_template,False
chart284,284,Materias primas,asset_current,pe_chart_template,False
chart285,285,"Materiales auxiliares, suministros y repuestos",asset_current,pe_chart_template,False
chart286,286,Envases y embalajes,asset_current,pe_chart_template,False
chart29111,29111,Mercaderías - Mercaderías - Costo,asset_current,pe_chart_template,False
chart29211,29211,Productos terminados - Productos terminados - Costo,asset_current,pe_chart_template,False
chart29213,29213,Productos terminados - Productos terminados - Costo de financiación,asset_current,pe_chart_template,False
chart29251,29251,Productos terminados - Inventario de servicios terminados - Costo,asset_current,pe_chart_template,False
chart2931,2931,"Subproductos, desechos y desperdicios - Subproductos",asset_current,pe_chart_template,False
chart2932,2932,"Subproductos, desechos y desperdicios - Desechos y desperdicios",asset_current,pe_chart_template,False
chart29411,29411,Productos en proceso - Productos en proceso - Costo,asset_current,pe_chart_template,False
chart29413,29413,Productos en proceso - Productos en proceso - Costo de financiación,asset_current,pe_chart_template,False
chart2945,2945,Productos en proceso - Inventario de servicios en proceso,asset_current,pe_chart_template,False
chart29511,29511,Materias primas - Materias primas - Costo,asset_current,pe_chart_template,False
chart2961,2961,"Materiales auxiliares, suministros y repuestos - Materiales auxiliares",asset_current,pe_chart_template,False
chart2962,2962,"Materiales auxiliares, suministros y repuestos - Suministros",asset_current,pe_chart_template,False
chart2963,2963,"Materiales auxiliares, suministros y repuestos - Repuestos",asset_current,pe_chart_template,False
chart2971,2971,Envases y embalajes - Envases,asset_current,pe_chart_template,False
chart2972,2972,Envases y embalajes - Embalajes,asset_current,pe_chart_template,False
chart2981,2981,Existencias por recibir - Mercaderías,asset_current,pe_chart_template,False
chart2982,2982,Existencias por recibir - Materias primas,asset_current,pe_chart_template,False
chart2983,2983,"Existencias por recibir - Materiales auxiliares, suministros y repuestos",asset_current,pe_chart_template,False
chart2984,2984,Existencias por recibir - Envases y embalajes,asset_current,pe_chart_template,False
chart30111,30111,Inversiones a ser mantenidas hasta el vencimiento - Instrumentos financieros representativos de deuda - Costo,asset_fixed,pe_chart_template,False
chart30114,30114,Inversiones a ser mantenidas hasta el vencimiento - Instrumentos financieros representativos de deuda - Valor razonable,asset_fixed,pe_chart_template,False
chart3021,3021,Instrumentos financieros representativos de derecho patrimonial - Certificados de suscripción preferente,asset_fixed,pe_chart_template,False
chart30221,30221,Instrumentos financieros representativos de derecho patrimonial - Acciones representativas de capital social – Comunes - Costo,asset_fixed,pe_chart_template,False
chart30224,30224,Instrumentos financieros representativos de derecho patrimonial - Acciones representativas de capital social – Comunes - Valor razonable,asset_fixed,pe_chart_template,False
chart30225,30225,Instrumentos financieros representativos de derecho patrimonial - Acciones representativas de capital social – Comunes - Participación patrimonial,asset_fixed,pe_chart_template,False
chart30231,30231,Instrumentos financieros representativos de derecho patrimonial - Acciones representativas de capital social – Preferentes - Costo,asset_fixed,pe_chart_template,False
chart30234,30234,Instrumentos financieros representativos de derecho patrimonial - Acciones representativas de capital social – Preferentes - Valor razonable,asset_fixed,pe_chart_template,False
chart30235,30235,Instrumentos financieros representativos de derecho patrimonial - Acciones representativas de capital social – Preferentes - Participación patrimonial,asset_fixed,pe_chart_template,False
chart30241,30241,Instrumentos financieros representativos de derecho patrimonial - Acciones de inversión - Costo,asset_fixed,pe_chart_template,False
chart30244,30244,Instrumentos financieros representativos de derecho patrimonial - Acciones de inversión - Valor razonable,asset_fixed,pe_chart_template,False
chart30245,30245,Instrumentos financieros representativos de derecho patrimonial - Acciones de inversión - Participación patrimonial,asset_fixed,pe_chart_template,False
chart30281,30281,Instrumentos financieros representativos de derecho patrimonial - Otros títulos representativos de patrimonio - Costo,asset_fixed,pe_chart_template,False
chart30284,30284,Instrumentos financieros representativos de derecho patrimonial - Otros títulos representativos de patrimonio - Valor razonable,asset_fixed,pe_chart_template,False
chart30285,30285,Instrumentos financieros representativos de derecho patrimonial - Otros títulos representativos de patrimonio - Participación patrimonial,asset_fixed,pe_chart_template,False
chart30311,30311,Certificados de participación en fondos - Cuotas - Fondos de inversión - Costo,asset_fixed,pe_chart_template,False
chart30314,30314,Certificados de participación en fondos - Cuotas - Fondos de inversión - Valor razonable,asset_fixed,pe_chart_template,False
chart30321,30321,Certificados de participación en fondos - Cuotas - Fondos mutuos - Costo,asset_fixed,pe_chart_template,False
chart30324,30324,Certificados de participación en fondos - Cuotas - Fondos mutuos - Valor razonable,asset_fixed,pe_chart_template,False
chart30411,30411,Participaciones en acuerdos conjuntos - Operaciones conjuntas - Costo,asset_fixed,pe_chart_template,False
chart30414,30414,Participaciones en acuerdos conjuntos - Operaciones conjuntas - Valor razonable,asset_fixed,pe_chart_template,False
chart30415,30415,Participaciones en acuerdos conjuntos - Operaciones conjuntas - Participación patrimonial,asset_fixed,pe_chart_template,False
chart30421,30421,Participaciones en acuerdos conjuntos - Negocios conjuntos - Costo,asset_fixed,pe_chart_template,False
chart30424,30424,Participaciones en acuerdos conjuntos - Negocios conjuntos - Valor razonable,asset_fixed,pe_chart_template,False
chart30425,30425,Participaciones en acuerdos conjuntos - Negocios conjuntos - Participación patrimonial,asset_fixed,pe_chart_template,False
chart30811,30811,Inversiones mobiliarias – Acuerdos de compra - Instrumentos financieros representativos de deuda – Acuerdo de compra - Costo,asset_fixed,pe_chart_template,False
chart30814,30814,Inversiones mobiliarias – Acuerdos de compra - Instrumentos financieros representativos de deuda – Acuerdo de compra - Valor razonable,asset_fixed,pe_chart_template,False
chart30821,30821,Inversiones mobiliarias – Acuerdos de compra - Instrumentos financieros representativos de derecho patrimonial – Acuerdo de compra - Costo,asset_fixed,pe_chart_template,False
chart30824,30824,Inversiones mobiliarias – Acuerdos de compra - Instrumentos financieros representativos de derecho patrimonial – Acuerdo de compra - Valor razonable,asset_fixed,pe_chart_template,False
chart31111,31111,Terrenos - Urbanos - Costo,asset_fixed,pe_chart_template,False
chart31112,31112,Terrenos - Urbanos - Revaluación,asset_fixed,pe_chart_template,False
chart31114,31114,Terrenos - Urbanos - Valor razonable,asset_fixed,pe_chart_template,False
chart31121,31121,Terrenos - Rurales - Costo,asset_fixed,pe_chart_template,False
chart31122,31122,Terrenos - Rurales - Revaluación,asset_fixed,pe_chart_template,False
chart31124,31124,Terrenos - Rurales - Valor razonable,asset_fixed,pe_chart_template,False
chart31211,31211,Edificaciones - Edificaciones - Costo,asset_fixed,pe_chart_template,False
chart31212,31212,Edificaciones - Edificaciones - Revaluación,asset_fixed,pe_chart_template,False
chart31213,31213,Edificaciones - Edificaciones - Costos de financiación,asset_fixed,pe_chart_template,False
chart31214,31214,Edificaciones - Edificaciones - Valor razonable,asset_fixed,pe_chart_template,False
chart31311,31311,Construcciones en curso - Edificaciones - Costo,asset_fixed,pe_chart_template,False
chart31312,31312,Construcciones en curso - Edificaciones - Revaluación,asset_fixed,pe_chart_template,False
chart31313,31313,Construcciones en curso - Edificaciones - Costos de financiación,asset_fixed,pe_chart_template,False
chart31314,31314,Construcciones en curso - Edificaciones - Valor razonable,asset_fixed,pe_chart_template,False
chart32111,32111,Propiedades de inversión - Arrendamiento financiero - Terrenos - Costo,asset_fixed,pe_chart_template,False
chart32112,32112,Propiedades de inversión - Arrendamiento financiero - Terrenos - Revaluación,asset_fixed,pe_chart_template,False
chart32114,32114,Propiedades de inversión - Arrendamiento financiero - Terrenos - Valor razonable,asset_fixed,pe_chart_template,False
chart32121,32121,Propiedades de inversión - Arrendamiento financiero - Edificaciones - Costo,asset_fixed,pe_chart_template,False
chart32122,32122,Propiedades de inversión - Arrendamiento financiero - Edificaciones - Revaluación,asset_fixed,pe_chart_template,False
chart32123,32123,Propiedades de inversión - Arrendamiento financiero - Edificaciones - Costo de financiación,asset_fixed,pe_chart_template,False
chart32124,32124,Propiedades de inversión - Arrendamiento financiero - Edificaciones - Valor razonable,asset_fixed,pe_chart_template,False
chart32201,32201,"Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en producción - Costo",asset_fixed,pe_chart_template,False
chart32202,32202,"Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en producción - Revaluación",asset_fixed,pe_chart_template,False
chart32203,32203,"Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en producción - Costo de financiación",asset_fixed,pe_chart_template,False
chart32211,32211,"Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en desarrollo - Costo",asset_fixed,pe_chart_template,False
chart32212,32212,"Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en desarrollo - Revaluación",asset_fixed,pe_chart_template,False
chart32213,32213,"Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en desarrollo - Costo de financiación",asset_fixed,pe_chart_template,False
chart32221,32221,"Propiedad, planta y equipo - Arrendamiento financiero - Terrenos - Costo",asset_fixed,pe_chart_template,False
chart32222,32222,"Propiedad, planta y equipo - Arrendamiento financiero - Terrenos - Revaluación",asset_fixed,pe_chart_template,False
chart32231,32231,"Propiedad, planta y equipo - Arrendamiento financiero - Edificaciones - Costo",asset_fixed,pe_chart_template,False
chart32232,32232,"Propiedad, planta y equipo - Arrendamiento financiero - Edificaciones - Revaluación",asset_fixed,pe_chart_template,False
chart32233,32233,"Propiedad, planta y equipo - Arrendamiento financiero - Edificaciones - Costo de financiación",asset_fixed,pe_chart_template,False
chart32241,32241,"Propiedad, planta y equipo - Arrendamiento financiero - Maquinaria y equipo de explotación - Costo",asset_fixed,pe_chart_template,False
chart32242,32242,"Propiedad, planta y equipo - Arrendamiento financiero - Maquinaria y equipo de explotación - Revaluación",asset_fixed,pe_chart_template,False
chart32243,32243,"Propiedad, planta y equipo - Arrendamiento financiero - Maquinaria y equipo de explotación - Costo de financiación",asset_fixed,pe_chart_template,False
chart32251,32251,"Propiedad, planta y equipo - Arrendamiento financiero - Unidades de transporte - Costo",asset_fixed,pe_chart_template,False
chart32252,32252,"Propiedad, planta y equipo - Arrendamiento financiero - Unidades de transporte - Revaluación",asset_fixed,pe_chart_template,False
chart32261,32261,"Propiedad, planta y equipo - Arrendamiento financiero - Muebles y enseres - Costo",asset_fixed,pe_chart_template,False
chart32262,32262,"Propiedad, planta y equipo - Arrendamiento financiero - Muebles y enseres - Revaluación",asset_fixed,pe_chart_template,False
chart32271,32271,"Propiedad, planta y equipo - Arrendamiento financiero - Equipos diversos - Costo",asset_fixed,pe_chart_template,False
chart32272,32272,"Propiedad, planta y equipo - Arrendamiento financiero - Equipos diversos - Revaluación",asset_fixed,pe_chart_template,False
chart32281,32281,"Propiedad, planta y equipo - Arrendamiento financiero - Herramientas y unidades de reemplazo - Costo",asset_fixed,pe_chart_template,False
chart32282,32282,"Propiedad, planta y equipo - Arrendamiento financiero - Herramientas y unidades de reemplazo - Revaluación",asset_fixed,pe_chart_template,False
chart32301,32301,"Propiedad, planta y equipo - Arrendamiento operativo - Planta productora en producción - Costo",asset_fixed,pe_chart_template,False
chart32302,32302,"Propiedad, planta y equipo - Arrendamiento operativo - Planta productora en producción - Revaluación",asset_fixed,pe_chart_template,False
chart32321,32321,"Propiedad, planta y equipo - Arrendamiento operativo - Terrenos - Costo",asset_fixed,pe_chart_template,False
chart32331,32331,"Propiedad, planta y equipo - Arrendamiento operativo - Edificaciones - Costo",asset_fixed,pe_chart_template,False
chart32332,32332,"Propiedad, planta y equipo - Arrendamiento operativo - Edificaciones - Revaluación",asset_fixed,pe_chart_template,False
chart32341,32341,"Propiedad, planta y equipo - Arrendamiento operativo - Maquinaria y equipo de explotación - Costo",asset_fixed,pe_chart_template,False
chart32342,32342,"Propiedad, planta y equipo - Arrendamiento operativo - Maquinaria y equipo de explotación - Revaluación",asset_fixed,pe_chart_template,False
chart32351,32351,"Propiedad, planta y equipo - Arrendamiento operativo - Unidades de transporte - Costo",asset_fixed,pe_chart_template,False
chart32352,32352,"Propiedad, planta y equipo - Arrendamiento operativo - Unidades de transporte - Revaluación",asset_fixed,pe_chart_template,False
chart32361,32361,"Propiedad, planta y equipo - Arrendamiento operativo - Equipos diversos - Costo",asset_fixed,pe_chart_template,False
chart32362,32362,"Propiedad, planta y equipo - Arrendamiento operativo - Equipos diversos - Revaluación",asset_fixed,pe_chart_template,False
chart33011,33011,Planta productora - Planta productora en producción - Costo,asset_fixed,pe_chart_template,False
chart33012,33012,Planta productora - Planta productora en producción - Revaluación,asset_fixed,pe_chart_template,False
chart33013,33013,Planta productora - Planta productora en producción - Costo de financiación,asset_fixed,pe_chart_template,False
chart33014,33014,Planta productora - Planta productora en producción - Valor razonable,asset_fixed,pe_chart_template,False
chart33021,33021,Planta productora - Planta productora en desarrollo - Costo,asset_fixed,pe_chart_template,False
chart33022,33022,Planta productora - Planta productora en desarrollo - Revaluación,asset_fixed,pe_chart_template,False
chart33023,33023,Planta productora - Planta productora en desarrollo - Costo de financiación,asset_fixed,pe_chart_template,False
chart33024,33024,Planta productora - Planta productora en desarrollo - Valor razonable,asset_fixed,pe_chart_template,False
chart33111,33111,Terrenos - Terrenos - Costo,asset_fixed,pe_chart_template,False
chart33112,33112,Terrenos - Terrenos - Revaluación,asset_fixed,pe_chart_template,False
chart33211,33211,Edificaciones - Edificaciones - Costo,asset_fixed,pe_chart_template,False
chart33212,33212,Edificaciones - Edificaciones - Revaluación,asset_fixed,pe_chart_template,False
chart33213,33213,Edificaciones - Edificaciones - Costo de financiación,asset_fixed,pe_chart_template,False
chart33241,33241,Edificaciones - Instalaciones - Costo,asset_fixed,pe_chart_template,False
chart33242,33242,Edificaciones - Instalaciones - Revaluación,asset_fixed,pe_chart_template,False
chart33243,33243,Edificaciones - Instalaciones - Costo de financiación,asset_fixed,pe_chart_template,False
chart33251,33251,Edificaciones - Mejoras en locales arrendados. - Costo,asset_fixed,pe_chart_template,False
chart33252,33252,Edificaciones - Mejoras en locales arrendados. - Revaluación,asset_fixed,pe_chart_template,False
chart33253,33253,Edificaciones - Mejoras en locales arrendados. - Costo de Financiación,asset_fixed,pe_chart_template,False
chart33311,33311,Maquinaria y equipo de explotación - Maquinaria y equipo de explotación - Costo,asset_fixed,pe_chart_template,False
chart33312,33312,Maquinaria y equipo de explotación - Maquinaria y equipo de explotación - Revaluación,asset_fixed,pe_chart_template,False
chart33313,33313,Maquinaria y equipo de explotación - Maquinaria y equipo de explotación - Costo de financiación,asset_fixed,pe_chart_template,False
chart33411,33411,Unidades de transporte - Vehículos motorizados - Costo,asset_fixed,pe_chart_template,False
chart33412,33412,Unidades de transporte - Vehículos motorizados - Revaluación,asset_fixed,pe_chart_template,False
chart33421,33421,Unidades de transporte - Vehículos no motorizados - Costo,asset_fixed,pe_chart_template,False
chart33422,33422,Unidades de transporte - Vehículos no motorizados - Revaluación,asset_fixed,pe_chart_template,False
chart33511,33511,Muebles y enseres - Muebles - Costo,asset_fixed,pe_chart_template,False
chart33512,33512,Muebles y enseres - Muebles - Revaluación,asset_fixed,pe_chart_template,False
chart33521,33521,Muebles y enseres - Enseres - Costo,asset_fixed,pe_chart_template,False
chart33522,33522,Muebles y enseres - Enseres - Revaluación,asset_fixed,pe_chart_template,False
chart33611,33611,Equipos diversos - Equipo para procesamiento de información - Costo,asset_fixed,pe_chart_template,False
chart33612,33612,Equipos diversos - Equipo para procesamiento de información - Revaluación,asset_fixed,pe_chart_template,False
chart33621,33621,Equipos diversos - Equipo de comunicación - Costo,asset_fixed,pe_chart_template,False
chart33622,33622,Equipos diversos - Equipo de comunicación - Revaluación,asset_fixed,pe_chart_template,False
chart33631,33631,Equipos diversos - Equipo de seguridad - Costo,asset_fixed,pe_chart_template,False
chart33632,33632,Equipos diversos - Equipo de seguridad - Revaluación,asset_fixed,pe_chart_template,False
chart33641,33641,Equipos diversos - Equipo de medio ambiente - Costo,asset_fixed,pe_chart_template,False
chart33642,33642,Equipos diversos - Equipo de medio ambiente - Revaluación,asset_fixed,pe_chart_template,False
chart33691,33691,Equipos diversos - Otros equipos - Costo,asset_fixed,pe_chart_template,False
chart33692,33692,Equipos diversos - Otros equipos - Revaluación,asset_fixed,pe_chart_template,False
chart33711,33711,Herramientas y unidades de reemplazo - Herramientas - Costo,asset_fixed,pe_chart_template,False
chart33712,33712,Herramientas y unidades de reemplazo - Herramientas - Revaluación,asset_fixed,pe_chart_template,False
chart33721,33721,Herramientas y unidades de reemplazo - Unidades de reemplazo - Costo,asset_fixed,pe_chart_template,False
chart33722,33722,Herramientas y unidades de reemplazo - Unidades de reemplazo - Revaluación,asset_fixed,pe_chart_template,False
chart3381,3381,Unidades por recibir - Maquinaria y equipo de explotación,asset_fixed,pe_chart_template,False
chart3382,3382,Unidades por recibir - Equipo de transporte,asset_fixed,pe_chart_template,False
chart3383,3383,Unidades por recibir - Muebles y enseres,asset_fixed,pe_chart_template,False
chart3386,3386,Unidades por recibir - Equipos diversos,asset_fixed,pe_chart_template,False
chart3387,3387,Unidades por recibir - Herramientas y unidades de reemplazo,asset_fixed,pe_chart_template,False
chart3391,3391,Obras en curso - Adecuación de terrenos,asset_fixed,pe_chart_template,False
chart33921,33921,Obras en curso - Edificaciones en curso - Costo,asset_fixed,pe_chart_template,False
chart33922,33922,Obras en curso - Edificaciones en curso - Costo de financiación,asset_fixed,pe_chart_template,False
chart33931,33931,Obras en curso - Maquinaria en montaje - Costo,asset_fixed,pe_chart_template,False
chart33932,33932,Obras en curso - Maquinaria en montaje - Costo de financiación,asset_fixed,pe_chart_template,False
chart34111,34111,"Concesiones, licencias y otros derechos - Derechos por concesiones - Costo",asset_fixed,pe_chart_template,False
chart34112,34112,"Concesiones, licencias y otros derechos - Derechos por concesiones - Revaluación",asset_fixed,pe_chart_template,False
chart34121,34121,"Concesiones, licencias y otros derechos - Licencias - Costo",asset_fixed,pe_chart_template,False
chart34122,34122,"Concesiones, licencias y otros derechos - Licencias - Revaluación",asset_fixed,pe_chart_template,False
chart34191,34191,"Concesiones, licencias y otros derechos - Otros derechos - Costo",asset_fixed,pe_chart_template,False
chart34192,34192,"Concesiones, licencias y otros derechos - Otros derechos - Revaluación",asset_fixed,pe_chart_template,False
chart34211,34211,Patentes y propiedad industrial - Patentes - Costo,asset_fixed,pe_chart_template,False
chart34212,34212,Patentes y propiedad industrial - Patentes - Revaluación,asset_fixed,pe_chart_template,False
chart34221,34221,Patentes y propiedad industrial - Marcas - Costo,asset_fixed,pe_chart_template,False
chart34222,34222,Patentes y propiedad industrial - Marcas - Revaluación,asset_fixed,pe_chart_template,False
chart34311,34311,Programas de computadora (software) - Aplicaciones informáticas - Costo,asset_fixed,pe_chart_template,False
chart34312,34312,Programas de computadora (software) - Aplicaciones informáticas - Revaluación,asset_fixed,pe_chart_template,False
chart34411,34411,Costos de exploración y desarrollo - Costos de exploración - Costo,asset_fixed,pe_chart_template,False
chart34412,34412,Costos de exploración y desarrollo - Costos de exploración - Revaluación,asset_fixed,pe_chart_template,False
chart34413,34413,Costos de exploración y desarrollo - Costos de exploración - Costo de financiación,asset_fixed,pe_chart_template,False
chart34421,34421,Costos de exploración y desarrollo - Costos de desarrollo - Costo,asset_fixed,pe_chart_template,False
chart34422,34422,Costos de exploración y desarrollo - Costos de desarrollo - Revaluación,asset_fixed,pe_chart_template,False
chart34423,34423,Costos de exploración y desarrollo - Costos de desarrollo - Costo de financiación,asset_fixed,pe_chart_template,False
chart34511,34511,"Fórmulas, diseños y prototipos - Fórmulas - Costo",asset_fixed,pe_chart_template,False
chart34512,34512,"Fórmulas, diseños y prototipos - Fórmulas - Revaluación",asset_fixed,pe_chart_template,False
chart34521,34521,"Fórmulas, diseños y prototipos - Diseños y prototipos - Costo",asset_fixed,pe_chart_template,False
chart34522,34522,"Fórmulas, diseños y prototipos - Diseños y prototipos - Revaluación",asset_fixed,pe_chart_template,False
chart3471,3471,Plusvalía mercantil - Plusvalía mercantil,asset_fixed,pe_chart_template,False
chart34911,34911,Otros activos intangibles - Otros activos intangibles - Costo,asset_fixed,pe_chart_template,False
chart34912,34912,Otros activos intangibles - Otros activos intangibles - Revaluación,asset_fixed,pe_chart_template,False
chart35111,35111,Activos biológicos en producción - De origen animal - Costo,asset_fixed,pe_chart_template,False
chart35113,35113,Activos biológicos en producción - De origen animal - Costo de financiación,asset_fixed,pe_chart_template,False
chart35114,35114,Activos biológicos en producción - De origen animal - Valor razonable,asset_fixed,pe_chart_template,False
chart35121,35121,Activos biológicos en producción - De origen vegetal - Costo,asset_fixed,pe_chart_template,False
chart35123,35123,Activos biológicos en producción - De origen vegetal - Costo de financiación,asset_fixed,pe_chart_template,False
chart35124,35124,Activos biológicos en producción - De origen vegetal - Valor razonable,asset_fixed,pe_chart_template,False
chart35211,35211,Activos biológicos en desarrollo - De origen animal - Costo,asset_fixed,pe_chart_template,False
chart35213,35213,Activos biológicos en desarrollo - De origen animal - Costo de financiación,asset_fixed,pe_chart_template,False
chart35214,35214,Activos biológicos en desarrollo - De origen animal - Valor razonable,asset_fixed,pe_chart_template,False
chart35221,35221,Activos biológicos en desarrollo - De origen vegetal - Costo,asset_fixed,pe_chart_template,False
chart35223,35223,Activos biológicos en desarrollo - De origen vegetal - Costo de financiación,asset_fixed,pe_chart_template,False
chart35224,35224,Activos biológicos en desarrollo - De origen vegetal - Valor razonable,asset_fixed,pe_chart_template,False
chart36111,36111,Terrenos - Costo,asset_fixed,pe_chart_template,False
chart36112,36112,Terrenos - Revaluación,asset_fixed,pe_chart_template,False
chart36121,36121,Edificaciones - Costo,asset_fixed,pe_chart_template,False
chart36122,36122,Edificaciones - Revaluación,asset_fixed,pe_chart_template,False
chart36123,36123,Edificaciones - Costo de financiación,asset_fixed,pe_chart_template,False
chart36131,36131,Construcciones en curso - edificaciones - Costo,asset_fixed,pe_chart_template,False
chart36132,36132,Construcciones en curso - edificaciones - Revaluación,asset_fixed,pe_chart_template,False
chart36133,36133,Construcciones en curso - edificaciones - Costo de financiación,asset_fixed,pe_chart_template,False
chart36211,36211,Arrendamiento financiero - Terrenos - Costo,asset_fixed,pe_chart_template,False
chart36212,36212,Arrendamiento financiero - Terrenos - Revaluación,asset_fixed,pe_chart_template,False
chart36221,36221,Arrendamiento financiero - Edificaciones - Costo,asset_fixed,pe_chart_template,False
chart36222,36222,Arrendamiento financiero - Edificaciones - Revaluación,asset_fixed,pe_chart_template,False
chart36223,36223,Arrendamiento financiero - Edificaciones - Costo de financiación,asset_fixed,pe_chart_template,False
chart36311,36311,Arrendamiento financiero - Terrenos - Costo,asset_fixed,pe_chart_template,False
chart36312,36312,Arrendamiento financiero - Terrenos - Revaluación,asset_fixed,pe_chart_template,False
chart36321,36321,Arrendamiento financiero - Edificaciones - Costo,asset_fixed,pe_chart_template,False
chart36322,36322,Arrendamiento financiero - Edificaciones - Revaluación,asset_fixed,pe_chart_template,False
chart36323,36323,Arrendamiento financiero - Edificaciones - Costo de financiación,asset_fixed,pe_chart_template,False
chart36331,36331,Arrendamiento financiero - Maquinaria y equipo de explotación - Costo,asset_fixed,pe_chart_template,False
chart36332,36332,Arrendamiento financiero - Maquinaria y equipo de explotación - Revaluación,asset_fixed,pe_chart_template,False
chart36333,36333,Arrendamiento financiero - Maquinaria y equipo de explotación - Costo de financiación,asset_fixed,pe_chart_template,False
chart36341,36341,Arrendamiento financiero - Unidades de transporte - Costo,asset_fixed,pe_chart_template,False
chart36342,36342,Arrendamiento financiero - Unidades de transporte - Revaluación,asset_fixed,pe_chart_template,False
chart36351,36351,Arrendamiento financiero - Muebles y enseres - Costo,asset_fixed,pe_chart_template,False
chart36352,36352,Arrendamiento financiero - Muebles y enseres - Revaluación,asset_fixed,pe_chart_template,False
chart36361,36361,Arrendamiento financiero - Equipos diversos - Costo,asset_fixed,pe_chart_template,False
chart36362,36362,Arrendamiento financiero - Equipos diversos - Revaluación,asset_fixed,pe_chart_template,False
chart36401,36401,Planta productora en producción - Costo,asset_fixed,pe_chart_template,False
chart36402,36402,Planta productora en producción - Planta productora en producción - Revaluación,asset_fixed,pe_chart_template,False
chart36403,36403,Planta productora en producción - Planta productora en producción - Costo de financiación,asset_fixed,pe_chart_template,False
chart36405,36405,Planta productora en producción - Planta productora en desarrollo - Costo,asset_fixed,pe_chart_template,False
chart36406,36406,Planta productora en producción - Planta productora en desarrollo - Revaluación,asset_fixed,pe_chart_template,False
chart36407,36407,Planta productora en producción - Planta productora en desarrollo - Costo de financiación,asset_fixed,pe_chart_template,False
chart36408,36408,Planta productora en producción - Planta productora en desarrollo - Valor razonable,asset_fixed,pe_chart_template,False
chart36411,36411,Terrenos - Costo,asset_fixed,pe_chart_template,False
chart36412,36412,Terrenos - Revaluación,asset_fixed,pe_chart_template,False
chart36421,36421,Edificaciones - Edificaciones - Costo,asset_fixed,pe_chart_template,False
chart36422,36422,Edificaciones - Edificaciones - Revaluación,asset_fixed,pe_chart_template,False
chart36423,36423,Edificaciones - Edificaciones - Costo de financiación,asset_fixed,pe_chart_template,False
chart36424,36424,Edificaciones - Instalaciones - Costo,asset_fixed,pe_chart_template,False
chart36425,36425,Edificaciones - Instalaciones - Revaluación,asset_fixed,pe_chart_template,False
chart36426,36426,Edificaciones - Instalaciones - Costo de financiación,asset_fixed,pe_chart_template,False
chart36427,36427,Edificaciones - Mejoras en locales arrendados - Costo,asset_fixed,pe_chart_template,False
chart36428,36428,Edificaciones - Mejoras en locales arrendados - Revaluación,asset_fixed,pe_chart_template,False
chart36429,36429,Edificaciones - Mejoras en locales arrendados - Costo de financiación,asset_fixed,pe_chart_template,False
chart36431,36431,Maquinaria y equipo de explotación - Costo,asset_fixed,pe_chart_template,False
chart36432,36432,Maquinaria y equipo de explotación - Revaluación,asset_fixed,pe_chart_template,False
chart36433,36433,Maquinaria y equipo de explotación - Costo de financiación,asset_fixed,pe_chart_template,False
chart36441,36441,Unidades de transporte - Costo,asset_fixed,pe_chart_template,False
chart36442,36442,Unidades de transporte - Revaluación,asset_fixed,pe_chart_template,False
chart36451,36451,Muebles y enseres - Costo,asset_fixed,pe_chart_template,False
chart36452,36452,Muebles y enseres - Revaluación,asset_fixed,pe_chart_template,False
chart36461,36461,Equipos diversos - Costo,asset_fixed,pe_chart_template,False
chart36462,36462,Equipos diversos - Revaluación,asset_fixed,pe_chart_template,False
chart36471,36471,Herramientas y unidades de reemplazo - Herramientas - Costo,asset_fixed,pe_chart_template,False
chart36491,36491,Obras en curso - Costo,asset_fixed,pe_chart_template,False
chart36492,36492,Obras en curso - Revaluación,asset_fixed,pe_chart_template,False
chart36511,36511,"Concesiones, licencias y otros derechos - Costo",asset_fixed,pe_chart_template,False
chart36512,36512,"Concesiones, licencias y otros derechos - Revaluación",asset_fixed,pe_chart_template,False
chart36521,36521,Patentes y propiedad industrial - Costo,asset_fixed,pe_chart_template,False
chart36522,36522,Patentes y propiedad industrial - Revaluación,asset_fixed,pe_chart_template,False
chart36531,36531,Programas de computadora (software) - Costo,asset_fixed,pe_chart_template,False
chart36532,36532,Programas de computadora (software) - Revaluación,asset_fixed,pe_chart_template,False
chart36541,36541,Costos de exploración y desarrollo - Costo,asset_fixed,pe_chart_template,False
chart36542,36542,Costos de exploración y desarrollo - Revaluación,asset_fixed,pe_chart_template,False
chart36543,36543,Costos de exploración y desarrollo - Costo de financiación,asset_fixed,pe_chart_template,False
chart36551,36551,"Fórmulas, diseños y prototipos - Costo",asset_fixed,pe_chart_template,False
chart36552,36552,"Fórmulas, diseños y prototipos - Revaluación",asset_fixed,pe_chart_template,False
chart3657,3657,Plusvalía mercantil,asset_fixed,pe_chart_template,False
chart36591,36591,Otros activos intangibles - Costo,asset_fixed,pe_chart_template,False
chart36592,36592,Otros activos intangibles - Revaluación,asset_fixed,pe_chart_template,False
chart36611,36611,Activos biológicos en producción - Costo,asset_fixed,pe_chart_template,False
chart36613,36613,Activos biológicos en producción - Costo de financiación,asset_fixed,pe_chart_template,False
chart36621,36621,Activos biológicos en desarrollo - Costo,asset_fixed,pe_chart_template,False
chart36622,36622,Activos biológicos en desarrollo - Costo de financiación,asset_fixed,pe_chart_template,False
chart36711,36711,Inversiones a ser mantenidas hasta el vencimiento - Costo,asset_fixed,pe_chart_template,False
chart36721,36721,Inversiones financieras representativas de derecho patrimonial - Costo,asset_fixed,pe_chart_template,False
chart36731,36731,Otras inversiones financieras - Costo,asset_fixed,pe_chart_template,False
chart3711,3711,Impuesto a la renta diferido - Impuesto a la renta diferido – Patrimonio,asset_fixed,pe_chart_template,False
chart3712,3712,Impuesto a la renta diferido - Impuesto a la renta diferido – Resultados,asset_fixed,pe_chart_template,False
chart3721,3721,Participaciones de los trabajadores diferidas - Participaciones de los trabajadores diferidas – Patrimonio,asset_fixed,pe_chart_template,False
chart3722,3722,Participaciones de los trabajadores diferidas - Participaciones de los trabajadores diferidas – Resultados,asset_fixed,pe_chart_template,False
chart3731,3731,Intereses diferidos - Intereses no devengados en transacciones con terceros,asset_fixed,pe_chart_template,False
chart3732,3732,Intereses diferidos - Intereses no devengados en medición a valor descontado,asset_fixed,pe_chart_template,False
chart3811,3811,Bienes de arte y cultura - Obras de arte,asset_fixed,pe_chart_template,False
chart3812,3812,Bienes de arte y cultura - Biblioteca,asset_fixed,pe_chart_template,False
chart3813,3813,Bienes de arte y cultura - Otros,asset_fixed,pe_chart_template,False
chart3821,3821,Diversos - Monedas y joyas,asset_fixed,pe_chart_template,False
chart3822,3822,Diversos - Bienes entregados en comodato,asset_fixed,pe_chart_template,False
chart3823,3823,Diversos - Bienes recibidos en pago (adjudicados y realizables),asset_fixed,pe_chart_template,False
chart3829,3829,Diversos - Otros,asset_fixed,pe_chart_template,False
chart39111,39111,Depreciación acumulada propiedades de inversión - Edificaciones - Costo,asset_fixed,pe_chart_template,False
chart39112,39112,Depreciación acumulada propiedades de inversión - Edificaciones - Revaluación,asset_fixed,pe_chart_template,False
chart39113,39113,Depreciación acumulada propiedades de inversión - Edificaciones - Costo de financiación,asset_fixed,pe_chart_template,False
chart39211,39211,Depreciación acumulada propiedades de inversión - Arrendamiento financiero - Edificaciones - Costo,asset_fixed,pe_chart_template,False
chart39212,39212,Depreciación acumulada propiedades de inversión - Arrendamiento financiero - Edificaciones - Revaluación,asset_fixed,pe_chart_template,False
chart39213,39213,Depreciación acumulada propiedades de inversión - Arrendamiento financiero - Edificaciones - Costo de financiación,asset_fixed,pe_chart_template,False
chart39321,39321,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Edificaciones - Costo",asset_fixed,pe_chart_template,False
chart39322,39322,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Edificaciones - Revaluación",asset_fixed,pe_chart_template,False
chart39323,39323,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Edificaciones - Costo de financiación",asset_fixed,pe_chart_template,False
chart39331,39331,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Maquinarias y equipos de explotación - Costo",asset_fixed,pe_chart_template,False
chart39332,39332,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Maquinarias y equipos de explotación - Revaluación",asset_fixed,pe_chart_template,False
chart39333,39333,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Maquinarias y equipos de explotación - Costo de financiación",asset_fixed,pe_chart_template,False
chart39341,39341,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Unidades de transporte - Costo",asset_fixed,pe_chart_template,False
chart39342,39342,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Unidades de transporte - Revaluación",asset_fixed,pe_chart_template,False
chart39351,39351,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Muebles y enseres - Costo",asset_fixed,pe_chart_template,False
chart39361,39361,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Equipos diversos - Costo",asset_fixed,pe_chart_template,False
chart39362,39362,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Equipos diversos - Revaluación",asset_fixed,pe_chart_template,False
chart39410,39410,Depreciación acumulada - Arrendamiento operativo - Activos por derecho de uso - arrendamiento operativo - Plantas productoras,asset_fixed,pe_chart_template,False
chart39411,39411,Depreciación acumulada - Arrendamiento operativo - Activos por derecho de uso - arrendamiento operativo - Terrenos,asset_fixed,pe_chart_template,False
chart39412,39412,Depreciación acumulada - Arrendamiento operativo - Activos por derecho de uso - arrendamiento operativo - Edificaciones,asset_fixed,pe_chart_template,False
chart39413,39413,Depreciación acumulada - Arrendamiento operativo - Activos por derecho de uso - arrendamiento operativo - Maquinarias y equipos de explotación,asset_fixed,pe_chart_template,False
chart39414,39414,Depreciación acumulada - Arrendamiento operativo - Activos por derecho de uso - arrendamiento operativo - Unidades de transporte,asset_fixed,pe_chart_template,False
chart39415,39415,Depreciación acumulada - Arrendamiento operativo - Activos por derecho de uso - arrendamiento operativo - Equipos diversos,asset_fixed,pe_chart_template,False
chart39520,39520,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Plantas productoras",asset_fixed,pe_chart_template,False
chart39521,39521,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Edificaciones",asset_fixed,pe_chart_template,False
chart39522,39522,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Instalaciones",asset_fixed,pe_chart_template,False
chart39523,39523,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Mejoras en locales arrendados",asset_fixed,pe_chart_template,False
chart39524,39524,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Maquinarias y equipos de explotación",asset_fixed,pe_chart_template,False
chart39525,39525,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Unidades de transporte",asset_fixed,pe_chart_template,False
chart39526,39526,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Muebles y enseres",asset_fixed,pe_chart_template,False
chart39527,39527,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Equipos diversos",asset_fixed,pe_chart_template,False
chart39528,39528,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Herramientas",asset_fixed,pe_chart_template,False
chart39529,39529,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Unidades de reemplazo",asset_fixed,pe_chart_template,False
chart39530,39530,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Plantas productoras",asset_fixed,pe_chart_template,False
chart39531,39531,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Edificaciones",asset_fixed,pe_chart_template,False
chart39532,39532,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Instalaciones",asset_fixed,pe_chart_template,False
chart39533,39533,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Mejoras en locales arrendados",asset_fixed,pe_chart_template,False
chart39534,39534,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Maquinarias y equipos de explotación",asset_fixed,pe_chart_template,False
chart39535,39535,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Unidades de transporte",asset_fixed,pe_chart_template,False
chart39536,39536,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Muebles y enseres",asset_fixed,pe_chart_template,False
chart39537,39537,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Equipos diversos",asset_fixed,pe_chart_template,False
chart39538,39538,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Herramientas y unidades de reemplazo",asset_fixed,pe_chart_template,False
chart39540,39540,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Costo de financiación - Plantas productoras",asset_fixed,pe_chart_template,False
chart39541,39541,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Costo de financiación - Edificaciones",asset_fixed,pe_chart_template,False
chart39542,39542,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Costo de financiación - Maquinarias y equipos de explotación",asset_fixed,pe_chart_template,False
chart39550,39550,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Valor razonable - Plantas productoras",asset_fixed,pe_chart_template,False
chart39611,39611,"Amortización acumulada - Intangibles – Costo - Concesiones, licencias y otros derechos",asset_fixed,pe_chart_template,False
chart39612,39612,Amortización acumulada - Intangibles – Costo - Patentes y propiedad industrial,asset_fixed,pe_chart_template,False
chart39613,39613,Amortización acumulada - Intangibles – Costo - Programas de computadora (software),asset_fixed,pe_chart_template,False
chart39614,39614,Amortización acumulada - Intangibles – Costo - Costos de exploración y desarrollo,asset_fixed,pe_chart_template,False
chart39615,39615,"Amortización acumulada - Intangibles – Costo - Fórmulas, diseños y prototipos",asset_fixed,pe_chart_template,False
chart39619,39619,Amortización acumulada - Intangibles – Costo - Otros activos intangibles,asset_fixed,pe_chart_template,False
chart39621,39621,"Amortización acumulada - Intangibles – Revaluación - Concesiones, licencias y otros derechos",asset_fixed,pe_chart_template,False
chart39622,39622,Amortización acumulada - Intangibles – Revaluación - Patentes y propiedad industrial,asset_fixed,pe_chart_template,False
chart39623,39623,Amortización acumulada - Intangibles – Revaluación - Programas de computadora (software),asset_fixed,pe_chart_template,False
chart39624,39624,Amortización acumulada - Intangibles – Revaluación - Costos de exploración y desarrollo,asset_fixed,pe_chart_template,False
chart39625,39625,"Amortización acumulada - Intangibles – Revaluación - Fórmulas, diseños y prototipos",asset_fixed,pe_chart_template,False
chart39629,39629,Amortización acumulada - Intangibles – Revaluación - Otros activos intangibles,asset_fixed,pe_chart_template,False
chart39633,39633,Amortización acumulada - Intangibles – Costos de financiación - Programas de computadora,asset_fixed,pe_chart_template,False
chart39634,39634,Amortización acumulada - Intangibles – Costos de financiación - Costos de exploración,asset_fixed,pe_chart_template,False
chart39635,39635,Amortización acumulada - Intangibles – Costos de financiación - Costos de desarrollo,asset_fixed,pe_chart_template,False
chart39811,39811,Depreciación acumulada - Activos biológicos en producción - Activos biológicos en producción - Costo - Activos biológicos en producción,asset_fixed,pe_chart_template,False
chart40111,40111,Gobierno nacional - Impuesto general a las ventas - IGV – Cuenta propia,liability_current,pe_chart_template,False
chart40112,40112,Gobierno nacional - Impuesto general a las ventas - IGV – Servicios prestados por no domiciliados,liability_current,pe_chart_template,False
chart40113,40113,Gobierno nacional - Impuesto general a las ventas - IGV – Régimen de percepciones,liability_current,pe_chart_template,False
chart40114,40114,Gobierno nacional - Impuesto general a las ventas - IGV – Régimen de retenciones,liability_current,pe_chart_template,False
chart40115,40115,Gobierno nacional - Impuesto general a las ventas - IGV – Importaciones,liability_current,pe_chart_template,False
chart40116,40116,Gobierno nacional - Impuesto general a las ventas - IGV – Destinado a operaciones gravadas,liability_current,pe_chart_template,False
chart40117,40117,Gobierno nacional - Impuesto general a las ventas - IGV -Destinado a operaciones comunes,liability_current,pe_chart_template,False
chart4012,4012,Gobierno nacional - Impuesto selectivo al consumo,liability_current,pe_chart_template,False
chart40151,40151,Gobierno nacional - Derechos aduaneros - Derechos arancelarios,liability_current,pe_chart_template,False
chart40152,40152,Gobierno nacional - Derechos aduaneros - Otros derechos arancelarios,liability_current,pe_chart_template,False
chart40171,40171,Gobierno nacional - Impuesto a la renta - Renta de tercera categoría,liability_current,pe_chart_template,False
chart40172,40172,Gobierno nacional - Impuesto a la renta - Renta de cuarta categoría,liability_current,pe_chart_template,False
chart40173,40173,Gobierno nacional - Impuesto a la renta - Renta de quinta categoría,liability_current,pe_chart_template,False
chart40174,40174,Gobierno nacional - Impuesto a la renta - Renta de no domiciliados,liability_current,pe_chart_template,False
chart40175,40175,Gobierno nacional - Impuesto a la renta - Otras retenciones,liability_current,pe_chart_template,False
chart40181,40181,Gobierno nacional - Otros impuestos y contraprestaciones - Impuesto a las transacciones financieras,liability_current,pe_chart_template,False
chart40182,40182,Gobierno nacional - Otros impuestos y contraprestaciones - Impuesto a los juegos de casino y tragamonedas,liability_current,pe_chart_template,False
chart40183,40183,Gobierno nacional - Otros impuestos y contraprestaciones - Tasas por la prestación de servicios públicos,liability_current,pe_chart_template,False
chart40184,40184,Gobierno nacional - Otros impuestos y contraprestaciones - Regalías,liability_current,pe_chart_template,False
chart40185,40185,Gobierno nacional - Otros impuestos y contraprestaciones - Impuesto a los dividendos,liability_current,pe_chart_template,False
chart40186,40186,Gobierno nacional - Otros impuestos y contraprestaciones - Impuesto temporal a los activos netos,liability_current,pe_chart_template,False
chart40189,40189,Gobierno nacional - Otros impuestos y contraprestaciones - Otros impuestos,liability_current,pe_chart_template,False
chart402,402,Certificados tributarios,liability_current,pe_chart_template,False
chart4031,4031,Instituciones públicas - ESSALUD,liability_current,pe_chart_template,False
chart4032,4032,Instituciones públicas - ONP,liability_current,pe_chart_template,False
chart4033,4033,Instituciones públicas - Contribución al SENATI,liability_current,pe_chart_template,False
chart4034,4034,Instituciones públicas - Contribución al SENCICO,liability_current,pe_chart_template,False
chart4039,4039,Instituciones públicas - Otras instituciones,liability_current,pe_chart_template,False
chart405,405,Gobiernos regionales,liability_current,pe_chart_template,False
chart40611,40611,Gobiernos locales - Impuestos - Impuesto al patrimonio vehicular,liability_current,pe_chart_template,False
chart40612,40612,Gobiernos locales - Impuestos - Impuesto a las apuestas,liability_current,pe_chart_template,False
chart40613,40613,Gobiernos locales - Impuestos - Impuesto a los juegos,liability_current,pe_chart_template,False
chart40614,40614,Gobiernos locales - Impuestos - Impuesto de alcabala,liability_current,pe_chart_template,False
chart40615,40615,Gobiernos locales - Impuestos - Impuesto predial,liability_current,pe_chart_template,False
chart40616,40616,Gobiernos locales - Impuestos - Impuesto a los espectáculos públicos no deportivos,liability_current,pe_chart_template,False
chart4062,4062,Gobiernos locales - Contribuciones,liability_current,pe_chart_template,False
chart40631,40631,Gobiernos locales - Tasas - Licencia de apertura de establecimientos,liability_current,pe_chart_template,False
chart40632,40632,Gobiernos locales - Tasas - Transporte público,liability_current,pe_chart_template,False
chart40633,40633,Gobiernos locales - Tasas - Estacionamiento de vehículos,liability_current,pe_chart_template,False
chart40634,40634,Gobiernos locales - Tasas - Servicios públicos o arbitrios,liability_current,pe_chart_template,False
chart40635,40635,Gobiernos locales - Tasas - Servicios administrativos o derechos,liability_current,pe_chart_template,False
chart409,409,Otros costos administrativos e intereses,liability_current,pe_chart_template,False
chart4111,4111,Remuneraciones por pagar - Sueldos y salarios por pagar,liability_payable,pe_chart_template,True
chart4112,4112,Remuneraciones por pagar - Comisiones por pagar,liability_payable,pe_chart_template,True
chart4113,4113,Remuneraciones por pagar - Remuneraciones en especie por pagar,liability_payable,pe_chart_template,True
chart4114,4114,Remuneraciones por pagar - Gratificaciones por pagar,liability_payable,pe_chart_template,True
chart4115,4115,Remuneraciones por pagar - Vacaciones por pagar,liability_payable,pe_chart_template,True
chart413,413,Participaciones de los trabajadores por pagar,liability_payable,pe_chart_template,True
chart4151,4151,Beneficios sociales de los trabajadores por pagar - Compensación por tiempo de servicios,liability_payable,pe_chart_template,True
chart4152,4152,Beneficios sociales de los trabajadores por pagar - Adelanto de compensación por tiempo de servicios,liability_payable,pe_chart_template,True
chart4153,4153,Beneficios sociales de los trabajadores por pagar - Pensiones y jubilaciones,liability_payable,pe_chart_template,True
chart417,417,Administradoras de fondos de pensiones,liability_payable,pe_chart_template,True
chart419,419,Otras remuneraciones y participaciones por pagar,liability_payable,pe_chart_template,True
chart4211,4211,"Facturas, boletas y otros comprobantes por pagar - No emitidas",liability_payable,pe_chart_template,True
chart4212,4212,"Facturas, boletas y otros comprobantes por pagar - Emitidas",liability_payable,pe_chart_template,True
chart422,422,Anticipos a proveedores,liability_payable,pe_chart_template,True
chart423,423,Letras por pagar,liability_payable,pe_chart_template,True
chart424,424,Honorarios por pagar,liability_payable,pe_chart_template,True
chart4311,4311,"Facturas, boletas y otros comprobantes por pagar - No emitidas",liability_payable,pe_chart_template,True
chart4312,4312,"Facturas, boletas y otros comprobantes por pagar - Emitidas",liability_payable,pe_chart_template,True
chart4321,4321,Anticipos otorgados - Anticipos otorgados,liability_payable,pe_chart_template,True
chart4331,4331,Letras por pagar - Letras por pagar,liability_payable,pe_chart_template,True
chart4341,4341,Honorarios por pagar - Honorarios por pagar,liability_payable,pe_chart_template,True
chart4411,4411,"Accionistas ( socios, partícipes) - Préstamos",liability_non_current,pe_chart_template,False
chart4412,4412,"Accionistas ( socios, partícipes) - Dividendos",liability_non_current,pe_chart_template,False
chart4419,4419,"Accionistas ( socios, partícipes) - Otras cuentas por pagar",liability_non_current,pe_chart_template,False
chart4421,4421,Directores - Dietas,liability_non_current,pe_chart_template,False
chart4429,4429,Directores - Otras cuentas por pagar,liability_non_current,pe_chart_template,False
chart4511,4511,Préstamos de instituciones financieras y otras entidades - Instituciones financieras,liability_current,pe_chart_template,False
chart4512,4512,Préstamos de instituciones financieras y otras entidades - Otras entidades,liability_current,pe_chart_template,False
chart452,452,Contratos de arrendamiento financiero,liability_current,pe_chart_template,False
chart4531,4531,Obligaciones emitidas - Bonos emitidos,liability_current,pe_chart_template,False
chart4532,4532,Obligaciones emitidas - Bonos titulizados,liability_current,pe_chart_template,False
chart4533,4533,Obligaciones emitidas - Papeles comerciales,liability_current,pe_chart_template,False
chart4539,4539,Obligaciones emitidas - Otras obligaciones,liability_current,pe_chart_template,False
chart4541,4541,Otros Instrumentos financieros por pagar - Letras,liability_current,pe_chart_template,False
chart4542,4542,Otros Instrumentos financieros por pagar - Papeles comerciales,liability_current,pe_chart_template,False
chart4543,4543,Otros Instrumentos financieros por pagar - Bonos,liability_current,pe_chart_template,False
chart4544,4544,Otros Instrumentos financieros por pagar - Pagarés,liability_current,pe_chart_template,False
chart4545,4545,Otros Instrumentos financieros por pagar - Facturas conformadas,liability_current,pe_chart_template,False
chart4549,4549,Otros Instrumentos financieros por pagar - Otras obligaciones financieras,liability_current,pe_chart_template,False
chart45511,45511,Costos de financiación por pagar - Préstamos de instituciones financieras y otras entidades - Instituciones financieras,liability_current,pe_chart_template,False
chart45512,45512,Costos de financiación por pagar - Préstamos de instituciones financieras y otras entidades - Otras entidades,liability_current,pe_chart_template,False
chart4552,4552,Costos de financiación por pagar - Contratos de arrendamiento financiero,liability_current,pe_chart_template,False
chart45531,45531,Costos de financiación por pagar - Obligaciones emitidas - Bonos emitidos,liability_current,pe_chart_template,False
chart45532,45532,Costos de financiación por pagar - Obligaciones emitidas - Bonos titulizados,liability_current,pe_chart_template,False
chart45533,45533,Costos de financiación por pagar - Obligaciones emitidas - Papeles comerciales,liability_current,pe_chart_template,False
chart45539,45539,Costos de financiación por pagar - Obligaciones emitidas - Otras obligaciones,liability_current,pe_chart_template,False
chart45541,45541,Costos de financiación por pagar - Otros instrumentos financieros por pagar - Letras,liability_current,pe_chart_template,False
chart45542,45542,Costos de financiación por pagar - Otros instrumentos financieros por pagar - Papeles comerciales,liability_current,pe_chart_template,False
chart45543,45543,Costos de financiación por pagar - Otros instrumentos financieros por pagar - Bonos,liability_current,pe_chart_template,False
chart45544,45544,Costos de financiación por pagar - Otros instrumentos financieros por pagar - Pagarés,liability_current,pe_chart_template,False
chart45545,45545,Costos de financiación por pagar - Otros instrumentos financieros por pagar - Facturas conformadas,liability_current,pe_chart_template,False
chart45549,45549,Costos de financiación por pagar - Otros instrumentos financieros por pagar - Otras obligaciones financieras,liability_current,pe_chart_template,False
chart456,456,Préstamos con compromisos de recompra,liability_current,pe_chart_template,False
chart461,461,Reclamaciones de terceros,liability_payable,pe_chart_template,True
chart4641,4641,Pasivos por instrumentos financieros - Instrumentos financieros primarios,liability_payable,pe_chart_template,True
chart46421,46421,Pasivos por instrumentos financieros - Instrumentos financieros derivados - Cartera de negociación,liability_payable,pe_chart_template,True
chart46422,46422,Pasivos por instrumentos financieros - Instrumentos financieros derivados - Instrumentos de cobertura,liability_payable,pe_chart_template,True
chart4651,4651,Pasivos por compra de activo inmovilizado - Inversiones mobiliarias,liability_payable,pe_chart_template,True
chart4652,4652,Pasivos por compra de activo inmovilizado - Propiedades de inversión,liability_payable,pe_chart_template,True
chart4653,4653,Pasivos por compra de activo inmovilizado - Activos adquiridos en arrendamiento financiero,liability_payable,pe_chart_template,True
chart4654,4654,"Pasivos por compra de activo inmovilizado - Propiedad, planta y equipo",liability_payable,pe_chart_template,True
chart4655,4655,Pasivos por compra de activo inmovilizado - Intangibles,liability_payable,pe_chart_template,True
chart4656,4656,Pasivos por compra de activo inmovilizado - Activos biológicos,liability_payable,pe_chart_template,True
chart466,466,Participación de terceros en acuerdos conjuntos,liability_payable,pe_chart_template,True
chart467,467,Depósitos recibidos en garantía,liability_payable,pe_chart_template,True
chart4691,4691,Otras cuentas por pagar diversas - Subsidios gubernamentales,liability_payable,pe_chart_template,True
chart4692,4692,Otras cuentas por pagar diversas - Donaciones condicionadas,liability_payable,pe_chart_template,True
chart4699,4699,Otras cuentas por pagar diversas - Otras cuentas por pagar,liability_payable,pe_chart_template,True
chart471,471,Préstamos,liability_payable,pe_chart_template,True
chart472,472,Costos de financiación,liability_payable,pe_chart_template,True
chart473,473,Anticipos recibidos,liability_payable,pe_chart_template,True
chart474,474,Regalías,liability_payable,pe_chart_template,True
chart475,475,Dividendos,liability_payable,pe_chart_template,True
chart476,476,Depósitos recibidos en garantía,liability_payable,pe_chart_template,True
chart4771,4771,Pasivo por compra de activo inmovilizado - Inversiones mobiliarias,liability_payable,pe_chart_template,True
chart4772,4772,Pasivo por compra de activo inmovilizado - Inversiones inmobiliarias,liability_payable,pe_chart_template,True
chart4773,4773,Pasivo por compra de activo inmovilizado - Activos adquiridos en arrendamiento financiero,liability_payable,pe_chart_template,True
chart4774,4774,"Pasivo por compra de activo inmovilizado - Propiedad, planta y equipo",liability_payable,pe_chart_template,True
chart4775,4775,Pasivo por compra de activo inmovilizado - Intangibles,liability_payable,pe_chart_template,True
chart4776,4776,Pasivo por compra de activo inmovilizado - Activos biológicos,liability_payable,pe_chart_template,True
chart4791,4791,Otras cuentas por pagar diversas - Otras cuentas por pagar diversas,liability_payable,pe_chart_template,True
chart481,481,Provisión para litigios,liability_non_current,pe_chart_template,False
chart482,482,"Provisión por desmantelamiento, retiro o rehabilitación del inmovilizado",liability_non_current,pe_chart_template,False
chart483,483,Provisión para reestructuraciones,liability_non_current,pe_chart_template,False
chart484,484,Provisión para protección y remediación del medio ambiente,liability_non_current,pe_chart_template,False
chart485,485,Provisión para gastos de responsabilidad social,liability_non_current,pe_chart_template,False
chart486,486,Provisión para garantías,liability_non_current,pe_chart_template,False
chart487,487,Provisión por activos por derecho de uso,liability_non_current,pe_chart_template,False
chart489,489,Otras provisiones,liability_non_current,pe_chart_template,False
chart4911,4911,Impuesto a la renta diferido - Impuesto a la renta diferido – Patrimonio,liability_current,pe_chart_template,False
chart4912,4912,Impuesto a la renta diferido - Impuesto a la renta diferido – Resultados,liability_current,pe_chart_template,False
chart4921,4921,Participaciones de los trabajadores diferidas - Participaciones de los trabajadores diferidas – Patrimonio,liability_current,pe_chart_template,False
chart4922,4922,Participaciones de los trabajadores diferidas - Participaciones de los trabajadores diferidas – Resultados,liability_current,pe_chart_template,False
chart4931,4931,Intereses diferidos - Intereses no devengados en transacciones con terceros,liability_current,pe_chart_template,False
chart4932,4932,Intereses diferidos - Intereses no devengados en medición a valor descontado,liability_current,pe_chart_template,False
chart494,494,Ganancia en venta con arrendamiento financiero paralelo,liability_current,pe_chart_template,False
chart495,495,Subsidios recibidos diferidos,liability_current,pe_chart_template,False
chart496,496,Ingresos diferidos,liability_current,pe_chart_template,False
chart497,497,Costos diferidos,liability_current,pe_chart_template,False
chart5011,5011,Capital social - Acciones,equity,pe_chart_template,False
chart5012,5012,Capital social - Participaciones,equity,pe_chart_template,False
chart502,502,Acciones en tesorería,equity,pe_chart_template,False
chart511,511,Acciones de inversión,equity,pe_chart_template,False
chart512,512,Acciones de inversión en tesorería,equity,pe_chart_template,False
chart521,521,Primas (descuento) de acciones,equity,pe_chart_template,False
chart5221,5221,Capitalizaciones en trámite - Aportes,equity,pe_chart_template,False
chart5222,5222,Capitalizaciones en trámite - Reservas,equity,pe_chart_template,False
chart5223,5223,Capitalizaciones en trámite - Acreencias,equity,pe_chart_template,False
chart5224,5224,Capitalizaciones en trámite - Utilidades,equity,pe_chart_template,False
chart523,523,Reducciones de capital pendientes de formalización,equity,pe_chart_template,False
chart561,561,Diferencia en cambio de inversiones permanentes en entidades extranjeras,equity,pe_chart_template,False
chart562,562,Instrumentos financieros – Coberturas,equity,pe_chart_template,False
chart5631,5631,Resultado en activos o pasivos financieros mantenidos para negociación - Ganancia,equity,pe_chart_template,False
chart5632,5632,Resultado en activos o pasivos financieros mantenidos para negociación - Pérdida,equity,pe_chart_template,False
chart5641,5641,Resultado en otros activos o pasivos por inversiones financieras - Ganancia,equity,pe_chart_template,False
chart5642,5642,Resultado en otros activos o pasivos por inversiones financieras - Pérdida,equity,pe_chart_template,False
chart5651,5651,Resultado en activos o pasivos financieros mantenidos para negociación – Compra o venta convencional fecha de liquidación - Ganancia,equity,pe_chart_template,False
chart5652,5652,Resultado en activos o pasivos financieros mantenidos para negociación – Compra o venta convencional fecha de liquidación - Pérdida,equity,pe_chart_template,False
chart57111,57111,Excedente de revaluación - Propiedad de inversión - Adquisición directa,equity,pe_chart_template,False
chart57112,57112,Excedente de revaluación - Propiedad de inversión - Arrendamiento financiero,equity,pe_chart_template,False
chart57121,57121,"Excedente de revaluación - Propiedad, planta y equipo - Adquisición directa",equity,pe_chart_template,False
chart57122,57122,"Excedente de revaluación - Propiedad, planta y equipo - Arrendamiento financiero",equity,pe_chart_template,False
chart5713,5713,Excedente de revaluación - Intangibles,equity,pe_chart_template,False
chart5714,5714,Excedente de revaluación - Activos por derecho de uso - arrendamiento operativo,equity,pe_chart_template,False
chart572,572,Excedente de revaluación – Acciones liberadas recibidas,equity,pe_chart_template,False
chart573,573,Participación en excedente de revaluación – Inversiones en entidades relacionadas,equity,pe_chart_template,False
chart581,581,Reinversión,equity,pe_chart_template,False
chart582,582,Legal,equity,pe_chart_template,False
chart583,583,Contractuales,equity,pe_chart_template,False
chart584,584,Estatutarias,equity,pe_chart_template,False
chart585,585,Facultativas,equity,pe_chart_template,False
chart589,589,Otras reservas,equity,pe_chart_template,False
chart5911,5911,Utilidades no distribuidas - Utilidades acumuladas,equity,pe_chart_template,False
chart5912,5912,Utilidades no distribuidas - Ingresos de años anteriores,equity,pe_chart_template,False
chart5921,5921,Pérdidas acumuladas - Pérdidas acumuladas,equity,pe_chart_template,False
chart5922,5922,Pérdidas acumuladas - Gastos de años anteriores,equity,pe_chart_template,False
chart6011,6011,Mercaderías - Mercaderías,expense,pe_chart_template,False
chart602,602,Materias primas,expense,pe_chart_template,False
chart6031,6031,"Materiales auxiliares, suministros y repuestos - Materiales auxiliares",expense,pe_chart_template,False
chart6032,6032,"Materiales auxiliares, suministros y repuestos - Suministros",expense,pe_chart_template,False
chart6033,6033,"Materiales auxiliares, suministros y repuestos - Repuestos",expense,pe_chart_template,False
chart6041,6041,Envases y embalajes - Envases,expense,pe_chart_template,False
chart6042,6042,Envases y embalajes - Embalajes,expense,pe_chart_template,False
chart60911,60911,Costos vinculados con las compras - Costos vinculados con las compras de mercaderías - Transporte,expense,pe_chart_template,False
chart60912,60912,Costos vinculados con las compras - Costos vinculados con las compras de mercaderías - Seguros,expense,pe_chart_template,False
chart60913,60913,Costos vinculados con las compras - Costos vinculados con las compras de mercaderías - Derechos aduaneros,expense,pe_chart_template,False
chart60914,60914,Costos vinculados con las compras - Costos vinculados con las compras de mercaderías - Comisiones,expense,pe_chart_template,False
chart60919,60919,Costos vinculados con las compras - Costos vinculados con las compras de mercaderías - Otros costos,expense,pe_chart_template,False
chart60921,60921,Costos vinculados con las compras - Costos vinculados con las compras de materias primas - Transporte,expense,pe_chart_template,False
chart60922,60922,Costos vinculados con las compras - Costos vinculados con las compras de materias primas - Seguros,expense,pe_chart_template,False
chart60923,60923,Costos vinculados con las compras - Costos vinculados con las compras de materias primas - Derechos aduaneros,expense,pe_chart_template,False
chart60924,60924,Costos vinculados con las compras - Costos vinculados con las compras de materias primas - Comisiones,expense,pe_chart_template,False
chart60925,60925,Costos vinculados con las compras - Costos vinculados con las compras de materias primas - Otros costos,expense,pe_chart_template,False
chart60931,60931,"Costos vinculados con las compras - Costos vinculados con las compras de materiales, suministros y repuestos - Transporte",expense,pe_chart_template,False
chart60932,60932,"Costos vinculados con las compras - Costos vinculados con las compras de materiales, suministros y repuestos - Seguros",expense,pe_chart_template,False
chart60933,60933,"Costos vinculados con las compras - Costos vinculados con las compras de materiales, suministros y repuestos - Derechos aduaneros",expense,pe_chart_template,False
chart60934,60934,"Costos vinculados con las compras - Costos vinculados con las compras de materiales, suministros y repuestos - Comisiones",expense,pe_chart_template,False
chart60935,60935,"Costos vinculados con las compras - Costos vinculados con las compras de materiales, suministros y repuestos - Otros costos",expense,pe_chart_template,False
chart60941,60941,Costos vinculados con las compras - Costos vinculados con las compras de envases y embalajes - Transporte,expense,pe_chart_template,False
chart60942,60942,Costos vinculados con las compras - Costos vinculados con las compras de envases y embalajes - Seguros,expense,pe_chart_template,False
chart60943,60943,Costos vinculados con las compras - Costos vinculados con las compras de envases y embalajes - Derechos aduaneros,expense,pe_chart_template,False
chart60944,60944,Costos vinculados con las compras - Costos vinculados con las compras de envases y embalajes - Comisiones,expense,pe_chart_template,False
chart60945,60945,Costos vinculados con las compras - Costos vinculados con las compras de envases y embalajes - Otros costos,expense,pe_chart_template,False
chart6111,6111,Mercaderías - Mercaderías,expense,pe_chart_template,False
chart6121,6121,Materias primas - Materias primas,expense,pe_chart_template,False
chart6131,6131,"Materiales auxiliares, suministros y repuestos - Materiales auxiliares",expense,pe_chart_template,False
chart6132,6132,"Materiales auxiliares, suministros y repuestos - Suministros",expense,pe_chart_template,False
chart6133,6133,"Materiales auxiliares, suministros y repuestos - Repuestos",expense,pe_chart_template,False
chart6141,6141,Envases y embalajes - Envases,expense,pe_chart_template,False
chart6142,6142,Envases y embalajes - Embalajes,expense,pe_chart_template,False
chart6211,6211,Remuneraciones - Sueldos y salarios,expense,pe_chart_template,False
chart6212,6212,Remuneraciones - Comisiones,expense,pe_chart_template,False
chart6213,6213,Remuneraciones - Remuneraciones en especie,expense,pe_chart_template,False
chart6214,6214,Remuneraciones - Gratificaciones,expense,pe_chart_template,False
chart6215,6215,Remuneraciones - Vacaciones,expense,pe_chart_template,False
chart622,622,Otras remuneraciones,expense,pe_chart_template,False
chart623,623,Indemnizaciones al personal,expense,pe_chart_template,False
chart624,624,Capacitación,expense,pe_chart_template,False
chart625,625,Atención al personal,expense,pe_chart_template,False
chart6271,6271,"Seguridad, previsión social y otras contribuciones - Régimen de prestaciones de salud",expense,pe_chart_template,False
chart6272,6272,"Seguridad, previsión social y otras contribuciones - Régimen de pensiones - Aporte de empresa",expense,pe_chart_template,False
chart6273,6273,"Seguridad, previsión social y otras contribuciones - Seguro complementario de trabajo de riesgo, accidentes de trabajo y enfermedades profesionales",expense,pe_chart_template,False
chart6274,6274,"Seguridad, previsión social y otras contribuciones - Seguro de vida",expense,pe_chart_template,False
chart6275,6275,"Seguridad, previsión social y otras contribuciones - Seguros particulares de prestaciones de salud – EPS y otros particulares",expense,pe_chart_template,False
chart6276,6276,"Seguridad, previsión social y otras contribuciones - Caja de beneficios de seguridad social del pescador",expense,pe_chart_template,False
chart6277,6277,"Seguridad, previsión social y otras contribuciones - Contribuciones al SENATI",expense,pe_chart_template,False
chart628,628,Retribuciones al directorio,expense,pe_chart_template,False
chart6291,6291,Beneficios sociales de los trabajadores - Compensación por tiempo de servicio,expense,pe_chart_template,False
chart6292,6292,Beneficios sociales de los trabajadores - Pensiones y jubilaciones,expense,pe_chart_template,False
chart6293,6293,Beneficios sociales de los trabajadores - Otros beneficios post-empleo,expense,pe_chart_template,False
chart62941,62941,Beneficios sociales de los trabajadores - Participación en las utilidades - Participación corriente,expense,pe_chart_template,False
chart62942,62942,Beneficios sociales de los trabajadores - Participación en las utilidades - Participación diferida,expense,pe_chart_template,False
chart63111,63111,"Transporte, correos y gastos de viaje - Transporte - De carga",expense,pe_chart_template,False
chart63112,63112,"Transporte, correos y gastos de viaje - Transporte - De pasajeros",expense,pe_chart_template,False
chart6312,6312,"Transporte, correos y gastos de viaje - Correos",expense,pe_chart_template,False
chart6313,6313,"Transporte, correos y gastos de viaje - Alojamiento",expense,pe_chart_template,False
chart6314,6314,"Transporte, correos y gastos de viaje - Alimentación",expense,pe_chart_template,False
chart6315,6315,"Transporte, correos y gastos de viaje - Otros gastos de viaje",expense,pe_chart_template,False
chart6321,6321,Asesoría y consultoría - Administrativa,expense,pe_chart_template,False
chart6322,6322,Asesoría y consultoría - Legal y tributaria,expense,pe_chart_template,False
chart6323,6323,Asesoría y consultoría - Auditoría y contable,expense,pe_chart_template,False
chart6324,6324,Asesoría y consultoría - Mercadotecnia,expense,pe_chart_template,False
chart6325,6325,Asesoría y consultoría - Medioambiental,expense,pe_chart_template,False
chart6326,6326,Asesoría y consultoría - Investigación y desarrollo,expense,pe_chart_template,False
chart6327,6327,Asesoría y consultoría - Producción,expense,pe_chart_template,False
chart6329,6329,Asesoría y consultoría - Otros,expense,pe_chart_template,False
chart633,633,Producción encargada a terceros,expense,pe_chart_template,False
chart6341,6341,Mantenimiento y reparaciones - Propiedad de inversión,expense,pe_chart_template,False
chart63421,63421,Mantenimiento y reparaciones - Activos por derecho de uso - Financiero,expense,pe_chart_template,False
chart63432,63432,"Mantenimiento y reparaciones - Propiedad, planta y equipo - Operativo",expense,pe_chart_template,False
chart6344,6344,Mantenimiento y reparaciones - Intangibles,expense,pe_chart_template,False
chart6345,6345,Mantenimiento y reparaciones - Activos biológicos,expense,pe_chart_template,False
chart6351,6351,Alquileres - Terrenos,expense,pe_chart_template,False
chart6352,6352,Alquileres - Edificaciones,expense,pe_chart_template,False
chart6353,6353,Alquileres - Maquinarias y equipos de explotación,expense,pe_chart_template,False
chart6354,6354,Alquileres - Equipo de transporte,expense,pe_chart_template,False
chart6355,6355,Alquileres - Muebles y enseres,expense,pe_chart_template,False
chart6356,6356,Alquileres - Equipos diversos,expense,pe_chart_template,False
chart6361,6361,Servicios básicos - Energía eléctrica,expense,pe_chart_template,False
chart6362,6362,Servicios básicos - Gas,expense,pe_chart_template,False
chart6363,6363,Servicios básicos - Agua,expense,pe_chart_template,False
chart6364,6364,Servicios básicos - Teléfono,expense,pe_chart_template,False
chart6365,6365,Servicios básicos - Internet,expense,pe_chart_template,False
chart6366,6366,Servicios básicos - Radio,expense,pe_chart_template,False
chart6367,6367,Servicios básicos - Cable,expense,pe_chart_template,False
chart6371,6371,"Publicidad, publicaciones, relaciones públicas - Publicidad",expense,pe_chart_template,False
chart6372,6372,"Publicidad, publicaciones, relaciones públicas - Publicaciones",expense,pe_chart_template,False
chart6373,6373,"Publicidad, publicaciones, relaciones públicas - Relaciones públicas",expense,pe_chart_template,False
chart638,638,Servicios de contratistas,expense,pe_chart_template,False
chart6391,6391,Otros servicios prestados por terceros - Gastos bancarios,expense,pe_chart_template,False
chart6392,6392,Otros servicios prestados por terceros - Gastos de laboratorio,expense,pe_chart_template,False
chart6411,6411,Gobierno nacional - Impuesto general a las ventas y selectivo al consumo,expense,pe_chart_template,False
chart6412,6412,Gobierno nacional - Impuesto a las transacciones financieras,expense,pe_chart_template,False
chart6413,6413,Gobierno nacional - Impuesto temporal a los activos netos,expense,pe_chart_template,False
chart6414,6414,Gobierno nacional - Impuesto a los juegos de casino y máquinas tragamonedas,expense,pe_chart_template,False
chart6415,6415,Gobierno nacional - Regalías mineras,expense,pe_chart_template,False
chart6416,6416,Gobierno nacional - Cánones,expense,pe_chart_template,False
chart6419,6419,Gobierno nacional - Otros,expense,pe_chart_template,False
chart642,642,Gobierno regional,expense,pe_chart_template,False
chart6431,6431,Gobierno local - Impuesto predial,expense,pe_chart_template,False
chart6432,6432,Gobierno local - Arbitrios municipales y seguridad ciudadana,expense,pe_chart_template,False
chart6433,6433,Gobierno local - Impuesto al patrimonio vehicular,expense,pe_chart_template,False
chart6434,6434,Gobierno local - Licencia de funcionamiento,expense,pe_chart_template,False
chart6439,6439,Gobierno local - Otros,expense,pe_chart_template,False
chart6442,6442,Otros gastos por tributos - Contribución al SENCICO,expense,pe_chart_template,False
chart6443,6443,Otros gastos por tributos - Otros,expense,pe_chart_template,False
chart6451,6451,Gastos en deuda tributaria - Intereses,expense,pe_chart_template,False
chart6452,6452,Gastos en deuda tributaria - intereses - fraccionamiento,expense,pe_chart_template,False
chart6453,6453,Gastos en deuda tributaria - Multas,expense,pe_chart_template,False
chart6454,6454,Gastos en deuda tributaria - Costas y otros,expense,pe_chart_template,False
chart651,651,Seguros,expense,pe_chart_template,False
chart652,652,Regalías,expense,pe_chart_template,False
chart653,653,Suscripciones,expense,pe_chart_template,False
chart654,654,Licencias y derechos de vigencia,expense,pe_chart_template,False
chart65511,65511,Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Costo neto de enajenación de activos inmovilizados - Inversiones mobiliarias,expense,pe_chart_template,False
chart65512,65512,Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Costo neto de enajenación de activos inmovilizados - Propiedades de inversión,expense,pe_chart_template,False
chart65513,65513,Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Costo neto de enajenación de activos inmovilizados - Activos por derecho de uso - arrendamiento financiero,expense,pe_chart_template,False
chart65514,65514,"Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Costo neto de enajenación de activos inmovilizados - Propiedad, planta y equipo",expense,pe_chart_template,False
chart65515,65515,Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Costo neto de enajenación de activos inmovilizados - Intangibles,expense,pe_chart_template,False
chart65516,65516,Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Costo neto de enajenación de activos inmovilizados - Activos biológicos,expense,pe_chart_template,False
chart65521,65521,Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Operaciones discontinuadas – Abandono de activos - Propiedades de inversión,expense,pe_chart_template,False
chart65522,65522,Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Operaciones discontinuadas – Abandono de activos - Activos por derecho de uso - Arrendamiento financiero,expense,pe_chart_template,False
chart65523,65523,"Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Operaciones discontinuadas – Abandono de activos - Propiedad, planta y equipo",expense,pe_chart_template,False
chart65524,65524,Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Operaciones discontinuadas – Abandono de activos - Intangibles,expense,pe_chart_template,False
chart65525,65525,Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Operaciones discontinuadas – Abandono de activos - Activos biológicos,expense,pe_chart_template,False
chart656,656,Suministros,expense,pe_chart_template,False
chart658,658,Gestión medioambiental,expense,pe_chart_template,False
chart6591,6591,Otros gastos de gestión - Donaciones,expense,pe_chart_template,False
chart6592,6592,Otros gastos de gestión - Sanciones administrativas,expense,pe_chart_template,False
chart6611,6611,Activo realizable - Mercaderías,expense,pe_chart_template,False
chart6612,6612,Activo realizable - Productos terminados,expense,pe_chart_template,False
chart66131,66131,Activo realizable - Activos no corrientes mantenidos para la venta - Propiedades de inversión,expense,pe_chart_template,False
chart66132,66132,"Activo realizable - Activos no corrientes mantenidos para la venta - Propiedad, planta y equipo",expense,pe_chart_template,False
chart66133,66133,Activo realizable - Activos no corrientes mantenidos para la venta - Intangibles,expense,pe_chart_template,False
chart66134,66134,Activo realizable - Activos no corrientes mantenidos para la venta - Activos biológicos,expense,pe_chart_template,False
chart6621,6621,Activo inmovilizado - Propiedades de inversión,expense,pe_chart_template,False
chart6622,6622,Activo inmovilizado - Activos biológicos,expense,pe_chart_template,False
chart6711,6711,Gastos en operaciones de endeudamiento y otros - Préstamos de instituciones financieras y otras entidades,expense,pe_chart_template,False
chart6712,6712,Gastos en operaciones de endeudamiento y otros - Contratos de arrendamiento financiero,expense,pe_chart_template,False
chart6713,6713,Gastos en operaciones de endeudamiento y otros - Emisión y colocación de instrumentos representativos de deuda y patrimonio,expense,pe_chart_template,False
chart6714,6714,Gastos en operaciones de endeudamiento y otros - Documentos vendidos o descontados,expense,pe_chart_template,False
chart672,672,Pérdida por instrumentos financieros derivados,expense,pe_chart_template,False
chart67311,67311,Intereses por préstamos y otras obligaciones - Préstamos de instituciones financieras y otras entidades - Instituciones financieras,expense,pe_chart_template,False
chart67312,67312,Intereses por préstamos y otras obligaciones - Préstamos de instituciones financieras y otras entidades - Otras entidades,expense,pe_chart_template,False
chart6732,6732,Intereses por préstamos y otras obligaciones - Contratos de arrendamiento financiero,expense,pe_chart_template,False
chart6733,6733,Intereses por préstamos y otras obligaciones - Otros instrumentos financieros por pagar,expense,pe_chart_template,False
chart6734,6734,Intereses por préstamos y otras obligaciones - Documentos vendidos o descontados,expense,pe_chart_template,False
chart6735,6735,Intereses por préstamos y otras obligaciones - Obligaciones emitidas,expense,pe_chart_template,False
chart6736,6736,Intereses por préstamos y otras obligaciones - Obligaciones comerciales,expense,pe_chart_template,False
chart6741,6741,Gastos en operaciones de factoraje (factoring) - Pérdida en instrumentos vendidos,expense,pe_chart_template,False
chart675,675,Descuentos concedidos por pronto pago,expense,pe_chart_template,False
chart676,676,Diferencia de cambio,expense,pe_chart_template,False
chart6771,6771,Pérdida por medición de activos y pasivos financieros al valor razonable - Inversiones mantenidas para negociación,expense,pe_chart_template,False
chart6772,6772,Pérdida por medición de activos y pasivos financieros al valor razonable - Otras inversiones financieras,expense,pe_chart_template,False
chart6773,6773,Pérdida por medición de activos y pasivos financieros al valor razonable - Otros,expense,pe_chart_template,False
chart6781,6781,Participación en resultados de entidades relacionadas - Participación en los resultados de subsidiarias y asociadas bajo el método del valor patrimonial,expense,pe_chart_template,False
chart6782,6782,Participación en resultados de entidades relacionadas - Participaciones en negocios conjuntos,expense,pe_chart_template,False
chart6791,6791,Otros gastos financieros - Primas por opciones,expense,pe_chart_template,False
chart6792,6792,Otros gastos financieros - Gastos financieros en medición a valor descontado,expense,pe_chart_template,False
chart6793,6793,Otros gastos financieros - Gastos financieros en actualización de activos por derecho de uso,expense,pe_chart_template,False
chart68111,68111,Depreciación de propiedades de inversión - Edificaciones - Costo,expense_depreciation,pe_chart_template,False
chart68112,68112,Depreciación de propiedades de inversión - Edificaciones - Revaluación,expense_depreciation,pe_chart_template,False
chart68113,68113,Depreciación de propiedades de inversión - Edificaciones - Costo de financiación,expense_depreciation,pe_chart_template,False
chart682111,682111,Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedades de inversión - Edificaciones - Costo,expense_depreciation,pe_chart_template,False
chart682112,682112,Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedades de inversión - Edificaciones - Revaluación,expense_depreciation,pe_chart_template,False
chart682113,682113,Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedades de inversión - Edificaciones - Costo de financiación,expense_depreciation,pe_chart_template,False
chart682211,682211,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Edificaciones - Costo",expense_depreciation,pe_chart_template,False
chart682212,682212,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Edificaciones - Revaluación",expense_depreciation,pe_chart_template,False
chart682213,682213,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Edificaciones - Costo de financiación",expense_depreciation,pe_chart_template,False
chart682221,682221,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Maquinarias y equipos de explotación - Costo",expense_depreciation,pe_chart_template,False
chart682222,682222,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Maquinarias y equipos de explotación - Revaluación",expense_depreciation,pe_chart_template,False
chart682223,682223,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Maquinarias y equipos de explotación - Costo de financiación",expense_depreciation,pe_chart_template,False
chart682231,682231,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Unidades de transporte - Costo",expense_depreciation,pe_chart_template,False
chart682232,682232,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Unidades de transporte - Revaluación",expense_depreciation,pe_chart_template,False
chart682251,682251,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Equipos diversos - Costo",expense_depreciation,pe_chart_template,False
chart682252,682252,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Equipos diversos - Revaluación",expense_depreciation,pe_chart_template,False
chart683111,683111,Depreciación de activos por derecho de uso - arrendamiento operativo - Depreciación de activos por derecho de uso - arrendamiento operativo - Edificaciones - Costo,expense_depreciation,pe_chart_template,False
chart683112,683112,Depreciación de activos por derecho de uso - arrendamiento operativo - Depreciación de activos por derecho de uso - arrendamiento operativo - Edificaciones - Revaluación,expense_depreciation,pe_chart_template,False
chart683121,683121,Depreciación de activos por derecho de uso - arrendamiento operativo - Depreciación de activos por derecho de uso - arrendamiento operativo - Maquinarias y equipos de explotación - Costo,expense_depreciation,pe_chart_template,False
chart683122,683122,Depreciación de activos por derecho de uso - arrendamiento operativo - Depreciación de activos por derecho de uso - arrendamiento operativo - Maquinarias y equipos de explotación - Revaluación,expense_depreciation,pe_chart_template,False
chart683131,683131,Depreciación de activos por derecho de uso - arrendamiento operativo - Depreciación de activos por derecho de uso - arrendamiento operativo - Unidades de transporte - Costo,expense_depreciation,pe_chart_template,False
chart683132,683132,Depreciación de activos por derecho de uso - arrendamiento operativo - Depreciación de activos por derecho de uso - arrendamiento operativo - Unidades de transporte - Revaluación,expense_depreciation,pe_chart_template,False
chart683152,683152,Depreciación de activos por derecho de uso - arrendamiento operativo - Depreciación de activos por derecho de uso - arrendamiento operativo - Equipos diversos - Revaluación,expense_depreciation,pe_chart_template,False
chart68410,68410,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Plantas productoras",expense_depreciation,pe_chart_template,False
chart68411,68411,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Edificaciones",expense_depreciation,pe_chart_template,False
chart68412,68412,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Maquinarias y equipos de explotación",expense_depreciation,pe_chart_template,False
chart68413,68413,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Unidades de transporte",expense_depreciation,pe_chart_template,False
chart68414,68414,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Muebles y enseres",expense_depreciation,pe_chart_template,False
chart68415,68415,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Equipos diversos",expense_depreciation,pe_chart_template,False
chart68416,68416,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Herramientas y unidades de reemplazo",expense_depreciation,pe_chart_template,False
chart68420,68420,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Plantas productoras",expense_depreciation,pe_chart_template,False
chart68421,68421,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Edificaciones",expense_depreciation,pe_chart_template,False
chart68422,68422,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Maquinarias y equipos de explotación",expense_depreciation,pe_chart_template,False
chart68423,68423,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Unidades de transporte",expense_depreciation,pe_chart_template,False
chart68424,68424,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Muebles y enseres",expense_depreciation,pe_chart_template,False
chart68425,68425,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Equipos diversos",expense_depreciation,pe_chart_template,False
chart68426,68426,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Herramientas y unidades de reemplazo",expense_depreciation,pe_chart_template,False
chart68430,68430,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costos de financiación - Plantas productoras",expense_depreciation,pe_chart_template,False
chart68431,68431,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costos de financiación - Edificaciones",expense_depreciation,pe_chart_template,False
chart68432,68432,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costos de financiación - Maquinarias y equipos de explotación",expense_depreciation,pe_chart_template,False
chart68511,68511,Depreciación de activos biológicos en producción - Depreciación de activos biológicos en producción - costo - Activos biológicos de origen animal,expense_depreciation,pe_chart_template,False
chart68512,68512,Depreciación de activos biológicos en producción - Depreciación de activos biológicos en producción - costo - Activos biológicos de origen vegetal,expense_depreciation,pe_chart_template,False
chart68521,68521,Depreciación de activos biológicos en producción - Depreciación de activos biológicos en producción - costo de financiación - Activos biológicos de origen animal,expense_depreciation,pe_chart_template,False
chart68522,68522,Depreciación de activos biológicos en producción - Depreciación de activos biológicos en producción - costo de financiación - Activos biológicos de origen vegetal,expense_depreciation,pe_chart_template,False
chart68611,68611,"Amortización de intangibles - Amortización de intangibles – Costo - Concesiones, licencias y otros derechos",expense_depreciation,pe_chart_template,False
chart68612,68612,Amortización de intangibles - Amortización de intangibles – Costo - Patentes y propiedad industrial,expense_depreciation,pe_chart_template,False
chart68613,68613,Amortización de intangibles - Amortización de intangibles – Costo - Programas de computadora (software),expense_depreciation,pe_chart_template,False
chart68614,68614,Amortización de intangibles - Amortización de intangibles – Costo - Costos de exploración y desarrollo,expense_depreciation,pe_chart_template,False
chart68615,68615,"Amortización de intangibles - Amortización de intangibles – Costo - Fórmulas, diseños y prototipos",expense_depreciation,pe_chart_template,False
chart68619,68619,Amortización de intangibles - Amortización de intangibles – Costo - Otros activos intangibles,expense_depreciation,pe_chart_template,False
chart68621,68621,"Amortización de intangibles - Amortización de intangibles – Revaluación - Concesiones, licencias y otros derechos",expense_depreciation,pe_chart_template,False
chart68622,68622,Amortización de intangibles - Amortización de intangibles – Revaluación - Patentes y propiedad industrial,expense_depreciation,pe_chart_template,False
chart68623,68623,Amortización de intangibles - Amortización de intangibles – Revaluación - Programas de computadora (software),expense_depreciation,pe_chart_template,False
chart68624,68624,Amortización de intangibles - Amortización de intangibles – Revaluación - Costos de exploración y desarrollo,expense_depreciation,pe_chart_template,False
chart68625,68625,"Amortización de intangibles - Amortización de intangibles – Revaluación - Fórmulas, diseños y prototipos",expense_depreciation,pe_chart_template,False
chart68629,68629,Amortización de intangibles - Amortización de intangibles – Revaluación - Otros activos intangibles,expense_depreciation,pe_chart_template,False
chart68711,68711,Valuación de activos - Estimación de cuentas de cobranza dudosa - Cuentas por cobrar comerciales – Terceros,expense_depreciation,pe_chart_template,False
chart68712,68712,Valuación de activos - Estimación de cuentas de cobranza dudosa - Cuentas por cobrar comerciales – Relacionadas,expense_depreciation,pe_chart_template,False
chart68713,68713,"Valuación de activos - Estimación de cuentas de cobranza dudosa - Cuentas por cobrar al personal, a los accionistas (socios) y directores",expense_depreciation,pe_chart_template,False
chart68714,68714,Valuación de activos - Estimación de cuentas de cobranza dudosa - Cuentas por cobrar diversas – Terceros,expense_depreciation,pe_chart_template,False
chart68715,68715,Valuación de activos - Estimación de cuentas de cobranza dudosa - Cuentas por cobrar diversas – Relacionadas,expense_depreciation,pe_chart_template,False
chart68731,68731,Valuación de activos - Desvalorización de inversiones mobiliarias - Inversiones a ser mantenidas hasta el vencimiento,expense_depreciation,pe_chart_template,False
chart68732,68732,Valuación de activos - Desvalorización de inversiones mobiliarias - Instrumentos financieros representativos de derecho patrimonial,expense_depreciation,pe_chart_template,False
chart68812,68812,Deterioro del valor de los activos - Desvalorización de propiedad de inversión - Edificaciones,expense_depreciation,pe_chart_template,False
chart68813,68813,Deterioro del valor de los activos - Desvalorización de propiedad de inversión - Construcciones en curso,expense_depreciation,pe_chart_template,False
chart68820,68820,Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - arrendamiento financiero - Planta productora en producción,expense_depreciation,pe_chart_template,False
chart68821,68821,Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - arrendamiento financiero - Planta productora en desarrollo,expense_depreciation,pe_chart_template,False
chart68822,68822,Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - arrendamiento financiero - Terrenos,expense_depreciation,pe_chart_template,False
chart68823,68823,Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - arrendamiento financiero - Edificaciones,expense_depreciation,pe_chart_template,False
chart68824,68824,Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - arrendamiento financiero - Maquinarias y equipos de explotación,expense_depreciation,pe_chart_template,False
chart68825,68825,Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - arrendamiento financiero - Unidades de transporte,expense_depreciation,pe_chart_template,False
chart68826,68826,Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - arrendamiento financiero - Muebles y enseres,expense_depreciation,pe_chart_template,False
chart68827,68827,Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - arrendamiento financiero - Equipos diversos,expense_depreciation,pe_chart_template,False
chart68828,68828,Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - arrendamiento financiero - Herramientas y unidades de reemplazo,expense_depreciation,pe_chart_template,False
chart68830,68830,"Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Planta productora en producción",expense_depreciation,pe_chart_template,False
chart68831,68831,"Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Planta productora en desarrollo",expense_depreciation,pe_chart_template,False
chart68832,68832,"Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Terrenos",expense_depreciation,pe_chart_template,False
chart68833,68833,"Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Edificaciones",expense_depreciation,pe_chart_template,False
chart68834,68834,"Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Maquinarias y equipos de explotación",expense_depreciation,pe_chart_template,False
chart68835,68835,"Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Unidades de transporte",expense_depreciation,pe_chart_template,False
chart68836,68836,"Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Muebles y enseres",expense_depreciation,pe_chart_template,False
chart68837,68837,"Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Equipos diversos",expense_depreciation,pe_chart_template,False
chart68838,68838,"Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Herramientas y unidades de reemplazo",expense_depreciation,pe_chart_template,False
chart68841,68841,"Deterioro del valor de los activos - Desvalorización de intangibles - Concesiones, licencias y otros derechos",expense_depreciation,pe_chart_template,False
chart68842,68842,Deterioro del valor de los activos - Desvalorización de intangibles - Patentes y propiedad industrial,expense_depreciation,pe_chart_template,False
chart68843,68843,Deterioro del valor de los activos - Desvalorización de intangibles - Programas de computadora (software),expense_depreciation,pe_chart_template,False
chart68844,68844,Deterioro del valor de los activos - Desvalorización de intangibles - Costos de exploración y desarrollo,expense_depreciation,pe_chart_template,False
chart68845,68845,"Deterioro del valor de los activos - Desvalorización de intangibles - Fórmulas, diseños y prototipos",expense_depreciation,pe_chart_template,False
chart68846,68846,Deterioro del valor de los activos - Desvalorización de intangibles - Otros activos intangibles,expense_depreciation,pe_chart_template,False
chart68847,68847,Deterioro del valor de los activos - Desvalorización de intangibles - Plusvalía mercantil,expense_depreciation,pe_chart_template,False
chart68891,68891,Deterioro del valor de los activos - Desvalorización de activos biológicos en producción - Activos biológicos de origen animal,expense_depreciation,pe_chart_template,False
chart68892,68892,Deterioro del valor de los activos - Desvalorización de activos biológicos en producción - Activos biológicos de origen vegetal,expense_depreciation,pe_chart_template,False
chart68911,68911,Provisiones - Provisión para litigios - Provisión para litigios – Costo,expense_depreciation,pe_chart_template,False
chart68912,68912,Provisiones - Provisión para litigios - Provisión para litigios – Actualización financiera,expense_depreciation,pe_chart_template,False
chart68921,68921,"Provisiones - Provisión por desmantelamiento, retiro o rehabilitación del inmovilizado - Provisión por desmantelamiento, retiro o rehabilitación del inmovilizado – Costo",expense_depreciation,pe_chart_template,False
chart68922,68922,"Provisiones - Provisión por desmantelamiento, retiro o rehabilitación del inmovilizado - Provisión por desmantelamiento, retiro o rehabilitación del inmovilizado – Actualización financiera",expense_depreciation,pe_chart_template,False
chart6893,6893,Provisiones - Provisión para reestructuraciones,expense_depreciation,pe_chart_template,False
chart68941,68941,Provisiones - Provisión para protección y remediación del medio ambiente - Provisión para protección y remediación del medio ambiente – Costo,expense_depreciation,pe_chart_template,False
chart68942,68942,Provisiones - Provisión para protección y remediación del medio ambiente - Provisión para protección y remediación del medio ambiente – Actualización financiera,expense_depreciation,pe_chart_template,False
chart68961,68961,Provisiones - Provisión para garantías - Provisión para garantías – Costo,expense_depreciation,pe_chart_template,False
chart68962,68962,Provisiones - Provisión para garantías - Provisión para garantías – Actualización financiera,expense_depreciation,pe_chart_template,False
chart68971,68971,Provisiones - Provisión por activos por derecho de uso - Provisión por activos por derecho de uso arrendamiento operativo,expense_depreciation,pe_chart_template,False
chart68972,68972,Provisiones - Provisión por activos por derecho de uso - Provisión por activos por derecho de uso arrendamiento operativo - actualización financiera,expense_depreciation,pe_chart_template,False
chart6899,6899,Provisiones - Otras provisiones,expense_depreciation,pe_chart_template,False
chart69111,69111,Mercaderías - Mercaderías - exportación - Terceros,expense_direct_cost,pe_chart_template,False
chart69112,69112,Mercaderías - Mercaderías - exportación - Relacionadas,expense_direct_cost,pe_chart_template,False
chart69121,69121,Mercaderías - Mercaderías - venta local - Terceros,expense_direct_cost,pe_chart_template,False
chart69122,69122,Mercaderías - Mercaderías - venta local - Relacionadas,expense_direct_cost,pe_chart_template,False
chart69211,69211,Productos terminados - Productos terminados - Exportación - Terceros,expense_direct_cost,pe_chart_template,False
chart69212,69212,Productos terminados - Productos terminados - Exportación - Relacionadas,expense_direct_cost,pe_chart_template,False
chart69221,69221,Productos terminados - Productos terminados - Venta local - Terceros,expense_direct_cost,pe_chart_template,False
chart69222,69222,Productos terminados - Productos terminados - Venta local - Relacionadas,expense_direct_cost,pe_chart_template,False
chart69231,69231,Productos terminados - Costos de financiación – Productos terminados - Terceros,expense_direct_cost,pe_chart_template,False
chart69232,69232,Productos terminados - Costos de financiación – Productos terminados - Relacionadas,expense_direct_cost,pe_chart_template,False
chart6924,6924,Productos terminados - Costos de producción no absorbido – Productos terminados,expense_direct_cost,pe_chart_template,False
chart6925,6925,Productos terminados - Costo de ineficiencia – Productos terminados,expense_direct_cost,pe_chart_template,False
chart69311,69311,Servicios terminados - Servicios – Exportación - Terceros,expense_direct_cost,pe_chart_template,False
chart69312,69312,Servicios terminados - Servicios – Exportación - Relacionadas,expense_direct_cost,pe_chart_template,False
chart69321,69321,Servicios terminados - Servicios – local - Terceros,expense_direct_cost,pe_chart_template,False
chart69322,69322,Servicios terminados - Servicios – local - Relacionadas,expense_direct_cost,pe_chart_template,False
chart69411,69411,"Subproductos, desechos y desperdicios - Subproductos - Terceros",expense_direct_cost,pe_chart_template,False
chart69412,69412,"Subproductos, desechos y desperdicios - Subproductos - Relacionadas",expense_direct_cost,pe_chart_template,False
chart69421,69421,"Subproductos, desechos y desperdicios - Desechos y desperdicios - Terceros",expense_direct_cost,pe_chart_template,False
chart69422,69422,"Subproductos, desechos y desperdicios - Desechos y desperdicios - Relacionadas",expense_direct_cost,pe_chart_template,False
chart6951,6951,Gastos por desvalorización de inventarios al costo - Mercaderías,expense_direct_cost,pe_chart_template,False
chart6952,6952,Gastos por desvalorización de inventarios al costo - Productos terminados,expense_direct_cost,pe_chart_template,False
chart6953,6953,"Gastos por desvalorización de inventarios al costo - Subproductos, desechos y desperdicios",expense_direct_cost,pe_chart_template,False
chart6954,6954,Gastos por desvalorización de inventarios al costo - Productos en proceso,expense_direct_cost,pe_chart_template,False
chart6955,6955,Gastos por desvalorización de inventarios al costo - Materias primas,expense_direct_cost,pe_chart_template,False
chart6956,6956,"Gastos por desvalorización de inventarios al costo - Materiales auxiliares, suministros y repuestos",expense_direct_cost,pe_chart_template,False
chart6957,6957,Gastos por desvalorización de inventarios al costo - Envases y embalajes,expense_direct_cost,pe_chart_template,False
chart6958,6958,Gastos por desvalorización de inventarios al costo - Inventarios por recibir,expense_direct_cost,pe_chart_template,False
chart70111,70111,Mercaderías - Mercaderías - venta de exportación - Terceros,income,pe_chart_template,False
chart70112,70112,Mercaderías - Mercaderías - venta de exportación - Relacionadas,income,pe_chart_template,False
chart70121,70121,Mercaderías - Mercaderías - venta local - Terceros,income,pe_chart_template,False
chart70122,70122,Mercaderías - Mercaderías - venta local - Relacionadas,income,pe_chart_template,False
chart70211,70211,Productos terminados - Productos terminados - venta de exportación - Terceros,income,pe_chart_template,False
chart70212,70212,Productos terminados - Productos terminados - venta de exportación - Relacionadas,income,pe_chart_template,False
chart70221,70221,Productos terminados - Productos terminados - venta local - Terceros,income,pe_chart_template,False
chart70222,70222,Productos terminados - Productos terminados - venta local - Relacionadas,income,pe_chart_template,False
chart70311,70311,Servicios terminados - Servicios – exportación - Terceros,income,pe_chart_template,False
chart70312,70312,Servicios terminados - Servicios – exportación - Relacionadas,income,pe_chart_template,False
chart70321,70321,Servicios terminados - Servicios – local - Terceros,income,pe_chart_template,False
chart70322,70322,Servicios terminados - Servicios – local - Relacionadas,income,pe_chart_template,False
chart70411,70411,"Subproductos, desechos y desperdicios - Subproductos - Terceros",income,pe_chart_template,False
chart70412,70412,"Subproductos, desechos y desperdicios - Subproductos - Relacionadas",income,pe_chart_template,False
chart70421,70421,"Subproductos, desechos y desperdicios - Desechos y desperdicios - Terceros",income,pe_chart_template,False
chart70422,70422,"Subproductos, desechos y desperdicios - Desechos y desperdicios - Relacionadas",income,pe_chart_template,False
chart70911,70911,Devoluciones sobre ventas - Mercaderías - Venta de exportación - Terceros,income,pe_chart_template,False
chart70912,70912,Devoluciones sobre ventas - Mercaderías - Venta de exportación - Relacionadas,income,pe_chart_template,False
chart70921,70921,Devoluciones sobre ventas - Mercaderías - Venta local - Terceros,income,pe_chart_template,False
chart70922,70922,Devoluciones sobre ventas - Mercaderías - Venta local - Relacionadas,income,pe_chart_template,False
chart70931,70931,Devoluciones sobre ventas - Productos terminados - Venta de exportación - Terceros,income,pe_chart_template,False
chart70932,70932,Devoluciones sobre ventas - Productos terminados - Venta de exportación - Relacionadas,income,pe_chart_template,False
chart70941,70941,Devoluciones sobre ventas - Productos terminados - Venta local - Terceros,income,pe_chart_template,False
chart70942,70942,Devoluciones sobre ventas - Productos terminados - Venta local - Relacionadas,income,pe_chart_template,False
chart70951,70951,Devoluciones sobre ventas - Inventarios de servicios rechazados - Terceros,income,pe_chart_template,False
chart70952,70952,Devoluciones sobre ventas - Inventarios de servicios rechazados - Relacionadas,income,pe_chart_template,False
chart70961,70961,"Devoluciones sobre ventas - Subproductos, desechos y desperdicios - Terceros",income,pe_chart_template,False
chart70962,70962,"Devoluciones sobre ventas - Subproductos, desechos y desperdicios - Relacionadas",income,pe_chart_template,False
chart7111,7111,Variación de productos terminados - Productos terminados,income_other,pe_chart_template,False
chart7121,7121,"Variación de subproductos, desechos y desperdicios - Subproductos",income_other,pe_chart_template,False
chart7122,7122,"Variación de subproductos, desechos y desperdicios - Desechos y desperdicios",income_other,pe_chart_template,False
chart7131,7131,Variación de productos en proceso - Productos en proceso de manufactura,income_other,pe_chart_template,False
chart7141,7141,Variación de envases y embalajes - Envases,income_other,pe_chart_template,False
chart7142,7142,Variación de envases y embalajes - Embalajes,income_other,pe_chart_template,False
chart7151,7151,Variación de inventarios de servicios - Inventarios de servicios en proceso,income_other,pe_chart_template,False
chart7211,7211,Propiedades de inversión - Edificaciones,income_other,pe_chart_template,False
chart7220,7220,"Propiedad, planta y equipo - Planta productora",income_other,pe_chart_template,False
chart7221,7221,"Propiedad, planta y equipo - Edificaciones",income_other,pe_chart_template,False
chart7222,7222,"Propiedad, planta y equipo - Maquinarias y otros equipos de explotación",income_other,pe_chart_template,False
chart7223,7223,"Propiedad, planta y equipo - Unidades de transporte",income_other,pe_chart_template,False
chart7224,7224,"Propiedad, planta y equipo - Muebles y enseres",income_other,pe_chart_template,False
chart7225,7225,"Propiedad, planta y equipo - Equipos diversos",income_other,pe_chart_template,False
chart7231,7231,Intangibles - Programas de computadora (software),income_other,pe_chart_template,False
chart7232,7232,Intangibles - Costos de exploración y desarrollo,income_other,pe_chart_template,False
chart7233,7233,"Intangibles - Fórmulas, diseños y prototipos",income_other,pe_chart_template,False
chart7241,7241,Activos biológicos - Activos biológicos en desarrollo de origen animal,income_other,pe_chart_template,False
chart7242,7242,Activos biológicos - Activos biológicos en desarrollo de origen vegetal,income_other,pe_chart_template,False
chart72511,72511,Costos de financiación capitalizados - Costos de financiación – Propiedades de inversión - Plantas productoras en desarrollo,income_other,pe_chart_template,False
chart72512,72512,Costos de financiación capitalizados - Costos de financiación – Propiedades de inversión - Edificaciones,income_other,pe_chart_template,False
chart72521,72521,"Costos de financiación capitalizados - Costos de financiación – Propiedad, planta y equipo - Plantas productoras en desarrollo",income_other,pe_chart_template,False
chart72522,72522,"Costos de financiación capitalizados - Costos de financiación – Propiedad, planta y equipo - Edificaciones",income_other,pe_chart_template,False
chart72523,72523,"Costos de financiación capitalizados - Costos de financiación – Propiedad, planta y equipo - Maquinarias y otros equipos de explotación",income_other,pe_chart_template,False
chart7253,7253,Costos de financiación capitalizados - Costos de financiación – Intangibles,income_other,pe_chart_template,False
chart72541,72541,Costos de financiación capitalizados - Costos de financiación – Activos biológicos en desarrollo - Activos biológicos de origen animal,income_other,pe_chart_template,False
chart72542,72542,Costos de financiación capitalizados - Costos de financiación – Activos biológicos en desarrollo - Activos biológicos de origen vegetal,income_other,pe_chart_template,False
chart7311,7311,"Descuentos, rebajas y bonificaciones obtenidos - Terceros",income,pe_chart_template,False
chart7312,7312,"Descuentos, rebajas y bonificaciones obtenidos - Relacionadas",income,pe_chart_template,False
chart7411,7411,"Descuentos, rebajas y bonificaciones concedidos - Terceros",income,pe_chart_template,False
chart7412,7412,"Descuentos, rebajas y bonificaciones concedidos - Relacionadas",income,pe_chart_template,False
chart751,751,Servicios en beneficio del personal,income_other,pe_chart_template,False
chart752,752,Comisiones y corretajes,income_other,pe_chart_template,False
chart753,753,Regalías,income_other,pe_chart_template,False
chart7540,7540,Alquileres - Plantas productoras,income_other,pe_chart_template,False
chart7541,7541,Alquileres - Terrenos,income_other,pe_chart_template,False
chart7542,7542,Alquileres - Edificaciones,income_other,pe_chart_template,False
chart7543,7543,Alquileres - Maquinarias y equipos de explotación,income_other,pe_chart_template,False
chart7544,7544,Alquileres - Unidades de transporte,income_other,pe_chart_template,False
chart7545,7545,Alquileres - Equipos diversos,income_other,pe_chart_template,False
chart7551,7551,Recuperación de cuentas de valuación - Recuperación – Cuentas de cobranza dudosa,income_other,pe_chart_template,False
chart7552,7552,Recuperación de cuentas de valuación - Recuperación – Desvalorización de inventarios,income_other,pe_chart_template,False
chart7553,7553,Recuperación de cuentas de valuación - Recuperación – Desvalorización de inversiones mobiliarias,income_other,pe_chart_template,False
chart7561,7561,Enajenación de activos inmovilizados - Inversiones mobiliarias,income_other,pe_chart_template,False
chart7562,7562,Enajenación de activos inmovilizados - Propiedades de inversión,income_other,pe_chart_template,False
chart7563,7563,Enajenación de activos inmovilizados - Activos adquiridos en arrendamiento financiero,income_other,pe_chart_template,False
chart7564,7564,"Enajenación de activos inmovilizados - Propiedad, planta y equipo",income_other,pe_chart_template,False
chart7565,7565,Enajenación de activos inmovilizados - Intangibles,income_other,pe_chart_template,False
chart7566,7566,Enajenación de activos inmovilizados - Activos biológicos,income_other,pe_chart_template,False
chart7571,7571,Recuperación de deterioro de cuentas de activos inmovilizados - Recuperación de deterioro de propiedades de inversión,income_other,pe_chart_template,False
chart7572,7572,"Recuperación de deterioro de cuentas de activos inmovilizados - Recuperación de deterioro de propiedad, planta y equipo",income_other,pe_chart_template,False
chart7573,7573,Recuperación de deterioro de cuentas de activos inmovilizados - Recuperación de deterioro de intangibles,income_other,pe_chart_template,False
chart7574,7574,Recuperación de deterioro de cuentas de activos inmovilizados - Recuperación de deterioro de activos biológicos,income_other,pe_chart_template,False
chart7591,7591,Otros ingresos de gestión - Subsidios gubernamentales,income_other,pe_chart_template,False
chart7592,7592,Otros ingresos de gestión - Reclamos al seguro,income_other,pe_chart_template,False
chart7593,7593,Otros ingresos de gestión - Donaciones,income_other,pe_chart_template,False
chart7594,7594,Otros ingresos de gestión - Devoluciones tributarias,income_other,pe_chart_template,False
chart7599,7599,Otros ingresos de gestión - Otros ingresos de gestión,income_other,pe_chart_template,False
chart7611,7611,Activo realizable - Mercaderías,income_other,pe_chart_template,False
chart7612,7612,Activo realizable - Productos terminados,income_other,pe_chart_template,False
chart76131,76131,Activo realizable - Activos no corrientes mantenidos para la venta - Propiedades de inversión,income_other,pe_chart_template,False
chart76132,76132,"Activo realizable - Activos no corrientes mantenidos para la venta - Propiedad, planta y equipo",income_other,pe_chart_template,False
chart76133,76133,Activo realizable - Activos no corrientes mantenidos para la venta - Intangibles,income_other,pe_chart_template,False
chart76134,76134,Activo realizable - Activos no corrientes mantenidos para la venta - Activos biológicos,income_other,pe_chart_template,False
chart7621,7621,Activo inmovilizado - Propiedades de inversión,income_other,pe_chart_template,False
chart7622,7622,Activo inmovilizado - Activos biológicos,income_other,pe_chart_template,False
chart771,771,Ganancia por instrumento financiero derivado,income_other,pe_chart_template,False
chart7721,7721,Rendimientos ganados - Depósitos en instituciones financieras,income_other,pe_chart_template,False
chart7722,7722,Rendimientos ganados - Cuentas por cobrar comerciales,income_other,pe_chart_template,False
chart7723,7723,Rendimientos ganados - Préstamos otorgados,income_other,pe_chart_template,False
chart7724,7724,Rendimientos ganados - Inversiones a ser mantenidas hasta el vencimiento,income_other,pe_chart_template,False
chart7725,7725,Rendimientos ganados - Instrumentos financieros representativos de derecho patrimonial,income_other,pe_chart_template,False
chart773,773,Dividendos,income_other,pe_chart_template,False
chart774,774,Ingresos en operaciones de factoraje (factoring),income_other,pe_chart_template,False
chart775,775,Descuentos obtenidos por pronto pago,income_other,pe_chart_template,False
chart776,776,Diferencia en cambio,income_other,pe_chart_template,False
chart7771,7771,Ganancia por medición de activos y pasivos financieros al valor razonable - Inversiones mantenidas para negociación,income_other,pe_chart_template,False
chart7772,7772,Ganancia por medición de activos y pasivos financieros al valor razonable - Otras inversiones,income_other,pe_chart_template,False
chart7773,7773,Ganancia por medición de activos y pasivos financieros al valor razonable - Otras,income_other,pe_chart_template,False
chart7781,7781,Participación en resultados de entidades relacionadas - Participación en los resultados de subsidiarias y asociadas bajo el método del valor patrimonial,income_other,pe_chart_template,False
chart7782,7782,Participación en resultados de entidades relacionadas - Ingresos por participaciones en negocios conjuntos,income_other,pe_chart_template,False
chart7792,7792,Otros ingresos financieros - Ingresos financieros en medición a valor descontado,income_other,pe_chart_template,False
chart781,781,Cargas cubiertas por provisiones,income_other,pe_chart_template,False
chart791,791,Cargas imputables a cuentas de costos y gastos,income,pe_chart_template,False
chart792,792,Gastos financieros imputables a cuentas de inventarios,income,pe_chart_template,False
chart801,801,Margen comercial,equity,pe_chart_template,False
chart811,811,Producción de bienes,equity,pe_chart_template,False
chart812,812,Producción de servicios,equity,pe_chart_template,False
chart813,813,Producción de activo inmovilizado,equity,pe_chart_template,False
chart821,821,Valor agregado,equity,pe_chart_template,False
chart831,831,Excedente bruto (insuficiencia bruta) de explotación,equity,pe_chart_template,False
chart841,841,Resultado de explotación,equity,pe_chart_template,False
chart851,851,Resultado antes del impuesto a las ganancias,equity,pe_chart_template,False
chart881,881,Impuesto a las ganancias – Corriente,equity,pe_chart_template,False
chart882,882,Impuesto a las ganancias – Diferido,equity,pe_chart_template,False
chart891,891,Utilidad,equity,pe_chart_template,False
chart892,892,Pérdida,equity,pe_chart_template,False

```

## File: data\account.group.template.csv

```csv
id,code_prefix_start,name,chart_template_id:id
group0,0,Cuentas de orden,pe_chart_template
group1,1,Activo disponible y exigible,pe_chart_template
group2,2,Activo realizable,pe_chart_template
group3,3,Activo inmovilizado,pe_chart_template
group4,4,Pasivo,pe_chart_template
group5,5,Patrimonio Neto,pe_chart_template
group6,6,Gastos por naturaleza,pe_chart_template
group7,7,Ingresos,pe_chart_template
group8,8,Saldos intrermediarios de gestión y determinación del resultado del ejercicio,pe_chart_template
group10,10,EFECTIVO Y EQUIVALENTES DE EFECTIVO,pe_chart_template
group11,11,INVERSIONES FINANCIERAS,pe_chart_template
group12,12,CUENTAS POR COBRAR COMERCIALES – TERCEROS,pe_chart_template
group13,13,CUENTAS POR COBRAR COMERCIALES – RELACIONADAS,pe_chart_template
group14,14,"CUENTAS POR COBRAR AL PERSONAL, A LOS ACCIONISTAS (SOCIOS) y DIRECTORES",pe_chart_template
group16,16,CUENTAS POR COBRAR DIVERSAS – TERCEROS,pe_chart_template
group17,17,CUENTAS POR COBRAR DIVERSAS – RELACIONADAS,pe_chart_template
group18,18,SERVICIOS Y OTROS CONTRATADOS POR ANTICIPADO,pe_chart_template
group19,19,ESTIMACIÓN DE CUENTAS DE COBRANZA DUDOSA,pe_chart_template
group20,20,MERCADERÍAS,pe_chart_template
group21,21,PRODUCTOS TERMINADOS,pe_chart_template
group22,22,"SUBPRODUCTOS, DESECHOS Y DESPERDICIOS",pe_chart_template
group23,23,PRODUCTOS EN PROCESO,pe_chart_template
group24,24,MATERIAS PRIMAS,pe_chart_template
group25,25,"MATERIALES AUXILIARES, SUMINISTROS Y REPUESTOS",pe_chart_template
group26,26,ENVASES Y EMBALAJES,pe_chart_template
group27,27,ACTIVOS NO CORRIENTES MANTENIDOS PARA LA VENTA,pe_chart_template
group28,28,INVENTARIOS POR RECIBIR,pe_chart_template
group29,29,DESVALORIZACIÓN DE INVENTARIOS,pe_chart_template
group30,30,INVERSIONES MOBILIARIAS,pe_chart_template
group31,31,PROPIEDADES DE INVERSIÓN,pe_chart_template
group32,32,ACTIVOS POR DERECHO DE USO,pe_chart_template
group33,33,"PROPIEDAD, PLANTA Y EQUIPO",pe_chart_template
group34,34,INTANGIBLES,pe_chart_template
group35,35,ACTIVOS BIOLÓGICOS,pe_chart_template
group36,36,DESVALORIZACIÓN DE ACTIVO INMOVILIZADO,pe_chart_template
group37,37,ACTIVO DIFERIDO,pe_chart_template
group38,38,OTROS ACTIVOS,pe_chart_template
group39,39,DEPRECIACIÓN y AMORTIZACIÓN ACUMULADOS,pe_chart_template
group40,40,"TRIBUTOS, CONTRAPRESTACIONES Y APORTES AL SISTEMA PÚBLICO DE PENSIONES Y DE SALUD POR PAGAR",pe_chart_template
group41,41,REMUNERACIONES Y PARTICIPACIONES POR PAGAR,pe_chart_template
group42,42,CUENTAS POR PAGAR COMERCIALES TERCEROS,pe_chart_template
group43,43,CUENTAS POR PAGAR COMERCIALES RELACIONADAS,pe_chart_template
group44,44,"CUENTAS POR PAGAR A LOS ACCIONISTAS (SOCIOS, PARTÍCIPES) Y DIRECTORES",pe_chart_template
group45,45,OBLIGACIONES FINANCIERAS,pe_chart_template
group46,46,CUENTAS POR PAGAR DIVERSAS – TERCEROS,pe_chart_template
group47,47,CUENTAS POR PAGAR DIVERSAS – RELACIONADAS,pe_chart_template
group48,48,PROVISIONES,pe_chart_template
group49,49,PASIVO DIFERIDO,pe_chart_template
group50,50,CAPITAL,pe_chart_template
group51,51,ACCIONES DE INVERSIÓN,pe_chart_template
group52,52,CAPITAL ADICIONAL,pe_chart_template
group56,56,RESULTADOS NO REALIZADOS,pe_chart_template
group57,57,EXCEDENTE DE REVALUACIÓN,pe_chart_template
group58,58,RESERVAS,pe_chart_template
group59,59,RESULTADOS ACUMULADOS,pe_chart_template
group60,60,COMPRAS,pe_chart_template
group61,61,VARIACIÓN DE INVENTARIOS,pe_chart_template
group62,62,GASTOS DE PERSONAL Y DIRECTORES,pe_chart_template
group63,63,GASTOS DE SERVICIOS PRESTADOS POR TERCEROS,pe_chart_template
group64,64,GASTOS POR TRIBUTOS,pe_chart_template
group65,65,OTROS GASTOS DE GESTION,pe_chart_template
group66,66,PERDIDA POR MEDICIÓN DE ACTIVOS NO FINANCIEROS AL VALOR RAZONABLE,pe_chart_template
group67,67,GASTOS FINANCIEROS,pe_chart_template
group68,68,VALUACIÓN Y DETERIORO DE ACTIVOS Y PROVISIONES,pe_chart_template
group69,69,COSTO DE VENTAS,pe_chart_template
group70,70,VENTAS,pe_chart_template
group71,71,VARIACIÓN DE LA PRODUCCIÓN ALMACENADA,pe_chart_template
group72,72,PRODUCCIÓN DE ACTIVO INMOVILIZADO,pe_chart_template
group73,73,"DESCUENTOS, REBAJAS Y BONIFICACIONES OBTENIDOS",pe_chart_template
group74,74,"DESCUENTOS, REBAJAS y BONIFICACIONES CONCEDIDOS",pe_chart_template
group75,75,OTROS INGRESOS DE GESTIÓN,pe_chart_template
group76,76,GANANCIA POR MEDICIÓN DE ACTIVOS NO FINANCIEROS AL VALOR RAZONABLE,pe_chart_template
group77,77,INGRESOS FINANCIEROS,pe_chart_template
group78,78,CARGAS CUBIERTAS POR PROVISIONES,pe_chart_template
group79,79,CARGAS IMPUTABLES A CUENTAS DE COSTOS Y GASTOS,pe_chart_template
group80,80,MARGEN COMERCIAL,pe_chart_template
group81,81,PRODUCCIÓN DEL EJERCICIO,pe_chart_template
group82,82,VALOR AGREGADO,pe_chart_template
group83,83,EXCEDENTE BRUTO (INSUFICIENCIA BRUTA) DE EXPLOTACIÓN,pe_chart_template
group84,84,RESULTADO DE EXPLOTACIÓN,pe_chart_template
group85,85,RESULTADO ANTES DE PARTICIPACIONES E IMPUESTOS,pe_chart_template
group88,88,IMPUESTO A LA RENTA,pe_chart_template
group89,89,DETERMINACIÓN DEL RESULTADO DEL EJERCICIO,pe_chart_template

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_pe.pe_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!--    Detraction can be understood here:-->
    <!--    http://orientacion.sunat.gob.pe/index.php/empresas-menu/regimen-de-detracciones-del-igv-empresas/como-funcionan-las-detracciones/3141-02-en-la-venta-de-bienes-empresas-->
    <!-- TODO AFFECT SUBSEQUENT -->
    <!-- VAT for sales -->
    <record id="sale_tax_igv_18" model="account.tax.template">
        <field name="chart_template_id" ref="pe_chart_template"/>
        <field name="name">18%</field>
        <field name="description">IGV</field>
        <field name="l10n_pe_edi_tax_code">1000</field>
        <field name="l10n_pe_edi_unece_category">S</field>
        <field name="amount">18.0</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence">1</field>
        <field name="include_base_amount">1</field>
        <field name="tax_group_id" ref="tax_group_igv"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
         ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
        ]"/>
    </record>
    <record id="sale_tax_igv_18_included" model="account.tax.template">
        <field name="chart_template_id" ref="pe_chart_template"/>
        <field name="name">18% (Included in price)</field>
        <field name="description">IGV</field>
        <field name="l10n_pe_edi_tax_code">1000</field>
        <field name="l10n_pe_edi_unece_category">S</field>
        <field name="amount">18.0</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence">1</field>
        <field name="price_include">1</field>
        <field name="include_base_amount">1</field>
        <field name="tax_group_id" ref="tax_group_igv"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
         ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
        ]"/>
    </record>
    <record id="sale_tax_exo" model="account.tax.template">
        <field name="chart_template_id" ref="pe_chart_template"/>
        <field name="name">0% Exonerated</field>
        <field name="description">EXO</field>
        <field name="l10n_pe_edi_tax_code">9997</field>
        <field name="l10n_pe_edi_unece_category">E</field>
        <field name="amount">0.0</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence">1</field>
        <field name="tax_group_id" ref="tax_group_exo"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
         ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
        ]"/>
    </record>
    <record id="sale_tax_ina" model="account.tax.template">
        <field name="chart_template_id" ref="pe_chart_template"/>
        <field name="name">0% Unaffected</field>
        <field name="description">INA</field>
        <field name="l10n_pe_edi_tax_code">9998</field>
        <field name="l10n_pe_edi_unece_category">Z</field>
        <field name="amount">0.0</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence">1</field>
        <field name="tax_group_id" ref="tax_group_ina"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
         ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
        ]"/>
    </record>
    <record id="sale_tax_gra" model="account.tax.template">
        <field name="chart_template_id" ref="pe_chart_template"/>
        <field name="name">0% Free</field>
        <field name="description">GRA</field>
        <field name="l10n_pe_edi_tax_code">9996</field>
        <field name="l10n_pe_edi_unece_category">E</field>
        <field name="amount">0.0</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence">1</field>
        <field name="tax_group_id" ref="tax_group_gra"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
         ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
        ]"/>
    </record>
    <record id="sale_tax_ics_0" model="account.tax.template">
        <field name="chart_template_id" ref="pe_chart_template"/>
        <field name="name">0% ISC</field>
        <field name="description">ISC</field>
        <field name="l10n_pe_edi_tax_code">2000</field>
        <field name="l10n_pe_edi_unece_category">S</field>
        <field name="amount">0.0</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence">1</field>
        <field name="include_base_amount">1</field>
        <field name="tax_group_id" ref="tax_group_isc"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart4012'),
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
                'account_id': ref('chart4012'),
            }),
        ]"/>
    </record>
    <!--    VAT for purchase-->
    <record id="purchase_tax_igv_18" model="account.tax.template">
        <field name="chart_template_id" ref="pe_chart_template"/>
        <field name="name">18%</field>
        <field name="description">IGV</field>
        <field name="l10n_pe_edi_tax_code">1000</field>
        <field name="l10n_pe_edi_unece_category">S</field>
        <field name="amount">18.0</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence">1</field>
        <field name="include_base_amount">1</field>
        <field name="tax_group_id" ref="tax_group_igv"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
         ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
        ]"/>
    </record>
    <record id="purchase_tax_igv_18_included" model="account.tax.template">
        <field name="chart_template_id" ref="pe_chart_template"/>
        <field name="name">18% (Included in price)</field>
        <field name="description">IGV</field>
        <field name="l10n_pe_edi_tax_code">1000</field>
        <field name="l10n_pe_edi_unece_category">S</field>
        <field name="amount">18.0</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence">1</field>
        <field name="price_include">1</field>
        <field name="include_base_amount">1</field>
        <field name="tax_group_id" ref="tax_group_igv"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
         ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
        ]"/>
    </record>
    <record id="sale_tax_igv_18g_ng" model="account.tax.template">
        <field name="chart_template_id" ref="pe_chart_template"/>
        <field name="name">18% Gravadas y No Gravadas</field>
        <field name="description">IGV</field>
        <field name="l10n_pe_edi_tax_code">1000</field>
        <field name="l10n_pe_edi_unece_category">S</field>
        <field name="amount">18.0</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence">1</field>
        <field name="include_base_amount">1</field>
        <field name="tax_group_id" ref="tax_group_igv_g_ng"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart40117'),
            }),
            (0,0, {
                'factor_percent': 0,
                'repartition_type': 'tax',
                'account_id': ref('chart6411'),
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
                'account_id': ref('chart40117'),
            }),
            (0,0, {
                'factor_percent': 0,
                'repartition_type': 'tax',
                'account_id': ref('chart6411'),
            }),
        ]"/>
    </record>
    <record id="sale_tax_igv_18_ng" model="account.tax.template">
        <field name="chart_template_id" ref="pe_chart_template"/>
        <field name="name">18% No Gravadas</field>
        <field name="description">IGV NG</field>
        <field name="l10n_pe_edi_tax_code">1000</field>
        <field name="l10n_pe_edi_unece_category">S</field>
        <field name="amount">18.0</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence">1</field>
        <field name="include_base_amount">1</field>
        <field name="tax_group_id" ref="tax_group_igv_ng"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart40116'),
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
                'account_id': ref('chart40116'),
            }),
        ]"/>
    </record>
    <record id="purchase_tax_exp_0" model="account.tax.template">
        <field name="chart_template_id" ref="pe_chart_template"/>
        <field name="name">0% EXP</field>
        <field name="description">EXP</field>
        <field name="l10n_pe_edi_tax_code">9995</field>
        <field name="l10n_pe_edi_unece_category">S</field>
        <field name="amount">0</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence">1</field>
        <field name="include_base_amount">1</field>
        <field name="tax_group_id" ref="tax_group_exp"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart40115'),
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
                'account_id': ref('chart40115'),
            }),
        ]"/>
    </record>
    <record id="purchase_tax_exo" model="account.tax.template">
        <field name="chart_template_id" ref="pe_chart_template"/>
        <field name="name">0% Exonerated</field>
        <field name="description">EXO</field>
        <field name="l10n_pe_edi_tax_code">9997</field>
        <field name="l10n_pe_edi_unece_category">E</field>
        <field name="amount">0.0</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence">1</field>
        <field name="tax_group_id" ref="tax_group_exo"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
         ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
        ]"/>
    </record>
    <record id="purchase_tax_ina" model="account.tax.template">
        <field name="chart_template_id" ref="pe_chart_template"/>
        <field name="name">0% Unaffected</field>
        <field name="description">INA</field>
        <field name="l10n_pe_edi_tax_code">9998</field>
        <field name="l10n_pe_edi_unece_category">Z</field>
        <field name="amount">0.0</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence">1</field>
        <field name="tax_group_id" ref="tax_group_ina"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
         ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
        ]"/>
    </record>
    <record id="purchase_tax_gra" model="account.tax.template">
        <field name="chart_template_id" ref="pe_chart_template"/>
        <field name="name">0% Free</field>
        <field name="description">GRA</field>
        <field name="l10n_pe_edi_tax_code">9996</field>
        <field name="l10n_pe_edi_unece_category">E</field>
        <field name="amount">0.0</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence">1</field>
        <field name="tax_group_id" ref="tax_group_gra"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
         ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
        ]"/>
    </record>
    <record id="sale_tax_exp" model="account.tax.template">
        <field name="chart_template_id" ref="pe_chart_template"/>
        <field name="name">0% EXP</field>
        <field name="description">EXP</field>
        <field name="l10n_pe_edi_tax_code">9995</field>
        <field name="l10n_pe_edi_unece_category">S</field>
        <field name="amount">0</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence">1</field>
        <field name="include_base_amount">1</field>
        <field name="tax_group_id" ref="tax_group_exp"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 0,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 0,
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
         ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 0,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 0,
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<odoo>
    <data noupdate="1">
        <record id="tax_group_igv" model="account.tax.group">
            <field name="name">IGV</field>
            <field name="sequence">0</field>
            <field name="country_id" ref="base.pe"/>
        </record>
        <record id="tax_group_igv_g_ng" model="account.tax.group">
            <field name="name">IGV GyNG</field>
            <field name="sequence">0</field>
            <field name="country_id" ref="base.pe"/>
        </record>
        <record id="tax_group_igv_ng" model="account.tax.group">
            <field name="name">IGV NG</field>
            <field name="sequence">0</field>
            <field name="country_id" ref="base.pe"/>
        </record>
        <record id="tax_group_ivap" model="account.tax.group">
            <field name="name">IVAP</field>
            <field name="sequence">0</field>
            <field name="country_id" ref="base.pe"/>
        </record>
        <record id="tax_group_isc" model="account.tax.group">
            <field name="name">ISC</field>
            <field name="sequence">0</field>
            <field name="country_id" ref="base.pe"/>
        </record>
        <record id="tax_group_exp" model="account.tax.group">
            <field name="name">EXP</field>
            <field name="sequence">0</field>
            <field name="country_id" ref="base.pe"/>
        </record>
        <record id="tax_group_gra" model="account.tax.group">
            <field name="name">GRA</field>
            <field name="sequence">0</field>
            <field name="country_id" ref="base.pe"/>
        </record>
        <record id="tax_group_exo" model="account.tax.group">
            <field name="name">EXO</field>
            <field name="sequence">0</field>
            <field name="country_id" ref="base.pe"/>
        </record>
        <record id="tax_group_ina" model="account.tax.group">
            <field name="name">INA</field>
            <field name="sequence">0</field>
            <field name="country_id" ref="base.pe"/>
        </record>
        <record id="tax_group_other" model="account.tax.group">
            <field name="name">OTROS</field>
            <field name="sequence">0</field>
            <field name="country_id" ref="base.pe"/>
        </record>
        <record id="tax_group_det" model="account.tax.group">
            <field name="name">DET</field>
            <field name="sequence">100</field>
            <field name="country_id" ref="base.pe"/>
        </record>
        <record id="tax_group_icbper" model="account.tax.group">
            <field name="name">ICBPER</field>
            <field name="sequence">0</field>
            <field name="country_id" ref="base.pe"/>
        </record>
        <record id="tax_group_ret" model="account.tax.group">
            <field name="name">RET</field>
            <field name="sequence">100</field>
        </record>
    </data>
</odoo>

```

## File: data\fiscal_position_data.xml

```xml
<odoo>
    <record id="local_peru" model="account.fiscal.position.template">
        <field name="name">LOCAL PERU</field>
        <field name="chart_template_id" ref="pe_chart_template"/>
        <field name="auto_apply">1</field>
        <field name="country_id" ref="base.pe"/>
        <field name="sequence">15</field>
    </record>
    <record id="exportation" model="account.fiscal.position.template">
        <field name="name">EXTRANJERO - EXPORTACIÓN</field>
        <field name="chart_template_id" ref="pe_chart_template"/>
        <field name="auto_apply">1</field>
        <field name="sequence">10</field>
    </record>
    <record id="exportation_sales_goods_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="exportation"/>
        <field name="tax_src_id" ref="sale_tax_igv_18"/>
        <field name="tax_dest_id" ref="sale_tax_exp"/>
    </record>
    <record id="exportation_sales_goods_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="exportation"/>
        <field name="tax_src_id" ref="sale_tax_igv_18_included"/>
        <field name="tax_dest_id" ref="sale_tax_exp"/>
    </record>
</odoo>

```

## File: data\l10n_latam_document_type_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model='l10n_latam.document.type' id='document_type01'>
        <field name='sequence'>10</field>
        <field name='code'>01</field>
        <field name='report_name'>Factura electrónica</field>
        <field name='name'>Factura</field>
        <field name='country_id' ref='base.pe'/>
        <field name='doc_code_prefix'>F</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type03'>
        <field name='sequence'>10</field>
        <field name='code'>02</field>
        <field name='report_name'>Recibo por Honorarios</field>
        <field name='name'>Recibo por Honorarios</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>R</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type02'>
        <field name='sequence'>20</field>
        <field name='code'>03</field>
        <field name='report_name'>Boleta de venta electrónica</field>
        <field name='name'>Boleta</field>
        <field name='country_id' ref='base.pe'/>
        <field name='doc_code_prefix'>B</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type04'>
        <field name='sequence'>20</field>
        <field name='code'>04</field>
        <field name='report_name'>Liquidación de compra</field>
        <field name='name'>Liquidación de compra</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>L</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type05'>
        <field name='sequence'>20</field>
        <field name='code'>05</field>
        <field name='report_name'>Boletos de Transporte Aéreo que emiten las Compañías de Aviación Comercial por el servicio de transporte aéreo regular de pasajeros, emitido de manera manual, mecanizada o por medios electrónicos (BME)</field>
        <field name='name'>Boletos de Transporte Aéreo que emiten las Compañías de Aviación Comercial por el servicio de transporte aéreo regular de pasajeros, emitido de manera manual, mecanizada o por medios electrónicos (BME)</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>B</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type06'>
        <field name='sequence'>20</field>
        <field name='code'>06</field>
        <field name='report_name'>Carta de porte aéreo por el servicio de transporte de carga aérea</field>
        <field name='name'>Carta de porte aéreo por el servicio de transporte de carga aérea</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>C</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type07'>
        <field name='sequence'>40</field>
        <field name='code'>07</field>
        <field name='report_name'>Nota de Crédito electrónica</field>
        <field name='name'>Nota de Crédito</field>
        <field name='country_id' ref='base.pe'/>
        <field name='doc_code_prefix'>F</field>
        <field name='internal_type'>credit_note</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type07b'>
        <field name='sequence'>41</field>
        <field name='code'>07</field>
        <field name='report_name'>Nota de Crédito Boleta electrónica</field>
        <field name='name'>Nota de Crédito Boleta</field>
        <field name='country_id' ref='base.pe'/>
        <field name='doc_code_prefix'>B</field>
        <field name='internal_type'>credit_note</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type08'>
        <field name='sequence'>50</field>
        <field name='code'>08</field>
        <field name='report_name'>Nota de Débito electrónica</field>
        <field name='name'>Nota de Débito</field>
        <field name='country_id' ref='base.pe'/>
        <field name='doc_code_prefix'>F</field>
        <field name='internal_type'>debit_note</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type08b'>
        <field name='sequence'>50</field>
        <field name='code'>08</field>
        <field name='report_name'>Nota de débito boleta electrónica </field>
        <field name='name'>Nota de Débito Boleta</field>
        <field name='country_id' ref='base.pe'/>
        <field name='doc_code_prefix'>B</field>
        <field name='internal_type'>debit_note</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type10'>
        <field name='sequence'>50</field>
        <field name='code'>10</field>
        <field name='report_name'>Recibo por Arrendamiento</field>
        <field name='name'>Recibo por Arrendamiento</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>R</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type11'>
        <field name='sequence'>50</field>
        <field name='code'>11</field>
        <field name='report_name'>Póliza emitida por las Bolsas de Valores, Bolsas de Productos o Agentes de Intermediación por operaciones realizadas en las Bolsas de Valores o Productos o fuera de las mismas, autorizadas por SMV</field>
        <field name='name'>Póliza emitida por las Bolsas de Valores, Bolsas de Productos o Agentes de Intermediación por operaciones realizadas en las Bolsas de Valores o Productos o fuera de las mismas, autorizadas por SMV</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>P</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type12'>
        <field name='sequence'>50</field>
        <field name='code'>12</field>
        <field name='report_name'>Ticket o cinta emitido por máquina registradora</field>
        <field name='name'>Ticket o cinta emitido por máquina registradora</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>T</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type13'>
        <field name='sequence'>50</field>
        <field name='code'>13</field>
        <field name='report_name'>Documentos emitidos por las empresas del sistema financiero y de seguros, y por las cooperativas de ahorro y crédito no autorizadas a captar recursos del público, que se encuentren bajo el control de la Superintendencia de Banca, Seguros y AFP.</field>
        <field name='name'>Documentos emitidos por las empresas del sistema financiero y de seguros, y por las cooperativas de ahorro y crédito no autorizadas a captar recursos del público, que se encuentren bajo el control de la Superintendencia de Banca, Seguros y AFP.</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>D</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type14'>
        <field name='sequence'>50</field>
        <field name='code'>14</field>
        <field name='report_name'>Recibo por servicios públicos de suministro de energía eléctrica, agua, teléfono, telex y telegráficos y otros servicios complementarios que se incluyan en el</field>
        <field name='name'>Recibo por servicios públicos de suministro de energía eléctrica, agua, teléfono, telex y telegráficos y otros servicios complementarios que se incluyan en el</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>R</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type15'>
        <field name='sequence'>50</field>
        <field name='code'>15</field>
        <field name='report_name'>Boletos emitidos por el servicio de transporte terrestre regular urbano de pasajeros y el ferroviario público de pasajeros prestado en vía férrea local.</field>
        <field name='name'>Boletos emitidos por el servicio de transporte terrestre regular urbano de pasajeros y el ferroviario público de pasajeros prestado en vía férrea local.</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>B</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type16'>
        <field name='sequence'>50</field>
        <field name='code'>16</field>
        <field name='report_name'>Boletos de viaje emitidos por las empresas de transporte nacional de pasajeros, siempre que cuenten con la autorización de la autoridad competente, en las rutas autorizadas. Vía terrestre o ferroviario público no emitido por medios electrónicos (BVME)</field>
        <field name='name'>Boletos de viaje emitidos por las empresas de transporte nacional de pasajeros, siempre que cuenten con la autorización de la autoridad competente, en las rutas autorizadas. Vía terrestre o ferroviario público no emitido por medios electrónicos (BVME)</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>B</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type17'>
        <field name='sequence'>50</field>
        <field name='code'>17</field>
        <field name='report_name'>Documento emitido por la Iglesia Católica por el arrendamiento de bienes inmuebles</field>
        <field name='name'>Documento emitido por la Iglesia Católica por el arrendamiento de bienes inmuebles</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>D</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type18'>
        <field name='sequence'>50</field>
        <field name='code'>18</field>
        <field name='report_name'>Documento emitido por las Administradoras Privadas de Fondo de Pensiones que se encuentran bajo la supervisión de la Superintendencia de Banca, Seguros y AFP</field>
        <field name='name'>Documento emitido por las Administradoras Privadas de Fondo de Pensiones que se encuentran bajo la supervisión de la Superintendencia de Banca, Seguros y AFP</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>D</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type19'>
        <field name='sequence'>50</field>
        <field name='code'>19</field>
        <field name='report_name'>Boleto o entrada por atracciones y espectáculos públicos</field>
        <field name='name'>Boleto o entrada por atracciones y espectáculos públicos</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>B</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type20'>
        <field name='sequence'>60</field>
        <field name='code'>20</field>
        <field name='report_name'>Comprobante de retención</field>
        <field name='name'>Comprobante de retención</field>
        <field name='country_id' ref='base.pe'/>
        <field name='doc_code_prefix'>R</field>
        <field name='internal_type'/>
    </record>
    <record model='l10n_latam.document.type' id='document_type21'>
        <field name='sequence'>60</field>
        <field name='code'>21</field>
        <field name='report_name'>Conocimiento de embarque por el servicio de transporte de carga marítima</field>
        <field name='name'>Conocimiento de embarque por el servicio de transporte de carga marítima</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>B</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type22'>
        <field name='sequence'>60</field>
        <field name='code'>22</field>
        <field name='report_name'>Comprobante por Operaciones No Habituales</field>
        <field name='name'>Comprobante por Operaciones No Habituales</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>C</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type23'>
        <field name='sequence'>60</field>
        <field name='code'>23</field>
        <field name='report_name'>Pólizas de Adjudicación emitidas con ocasión del remate o adjudicación de bienes por venta forzada, por los martilleros o las entidades que rematen o subasten bienes por cuenta de terceros</field>
        <field name='name'>Pólizas de Adjudicación emitidas con ocasión del remate o adjudicación de bienes por venta forzada, por los martilleros o las entidades que rematen o subasten bienes por cuenta de terceros</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>C</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type24'>
        <field name='sequence'>60</field>
        <field name='code'>24</field>
        <field name='report_name'>Certificado de pago de regalías emitidas por PERUPETRO S.A</field>
        <field name='name'>Certificado de pago de regalías emitidas por PERUPETRO S.A</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>C</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type25'>
        <field name='sequence'>60</field>
        <field name='code'>25</field>
        <field name='report_name'>Documento de Atribución (Ley del Impuesto General a las Ventas e Impuesto Selectivo al Consumo, Art. 19º, último párrafo, R.S. N° 022-98-SUNAT).</field>
        <field name='name'>Documento de Atribución (Ley del Impuesto General a las Ventas e Impuesto Selectivo al Consumo, Art. 19º, último párrafo, R.S. N° 022-98-SUNAT).</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>D</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type26'>
        <field name='sequence'>60</field>
        <field name='code'>26</field>
        <field name='report_name'>Recibo por el Pago de la Tarifa por Uso de Agua Superficial con fines agrarios y por el pago de la Cuota para la ejecución de una determinada obra o actividad acordada por la Asamblea General de la Comisión de Regantes o Resolución expedida por el Jefe de la Unidad de Aguas y de Riego (Decreto Supremo N° 003-90-AG, Arts. 28 y 48)</field>
        <field name='name'>Recibo por el Pago de la Tarifa por Uso de Agua Superficial con fines agrarios y por el pago de la Cuota para la ejecución de una determinada obra o actividad acordada por la Asamblea General de la Comisión de Regantes o Resolución expedida por el Jefe de la Unidad de Aguas y de Riego (Decreto Supremo N° 003-90-AG, Arts. 28 y 48)</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>R</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type27'>
        <field name='sequence'>60</field>
        <field name='code'>27</field>
        <field name='report_name'>Seguro Complementario de Trabajo de Riesgo</field>
        <field name='name'>Seguro Complementario de Trabajo de Riesgo</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>S</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type28'>
        <field name='sequence'>60</field>
        <field name='code'>28</field>
        <field name='report_name'>Documentos emitidos por los servicios aeroportuarios prestados a favor de los pasajeros, mediante mecanismo de etiquetas autoadhesivas.</field>
        <field name='name'>Documentos emitidos por los servicios aeroportuarios prestados a favor de los pasajeros, mediante mecanismo de etiquetas autoadhesivas.</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>D</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type29'>
        <field name='sequence'>60</field>
        <field name='code'>29</field>
        <field name='report_name'>Documentos emitidos por la COFOPRI en calidad de oferta de venta de terrenos, los correspondientes a las subastas públicas y a la retribución de los servicios que presta</field>
        <field name='name'>Documentos emitidos por la COFOPRI en calidad de oferta de venta de terrenos, los correspondientes a las subastas públicas y a la retribución de los servicios que presta</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>D</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type30'>
        <field name='sequence'>60</field>
        <field name='code'>30</field>
        <field name='report_name'>Documentos emitidos por las empresas que desempeñan el rol adquirente en los sistemas de pago mediante tarjetas de crédito y débito, emitidas por bancos e instituciones financieras o crediticias, domiciliados o no en el país.</field>
        <field name='name'>Documentos emitidos por las empresas que desempeñan el rol adquirente en los sistemas de pago mediante tarjetas de crédito y débito, emitidas por bancos e instituciones financieras o crediticias, domiciliados o no en el país.</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>D</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type32'>
        <field name='sequence'>60</field>
        <field name='code'>32</field>
        <field name='report_name'>Documentos emitidos por las empresas recaudadoras de la denominada Garantía de Red Principal a la que hace referencia el numeral 7.6 del artículo 7° de la Ley N° 27133 – Ley de Promoción del Desarrollo de la Industria del Gas Natural</field>
        <field name='name'>Documentos emitidos por las empresas recaudadoras de la denominada Garantía de Red Principal a la que hace referencia el numeral 7.6 del artículo 7° de la Ley N° 27133 – Ley de Promoción del Desarrollo de la Industria del Gas Natural</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>D</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type33'>
        <field name='sequence'>60</field>
        <field name='code'>33</field>
        <field name='report_name'>Manifiesto de Pasajeros</field>
        <field name='name'>Manifiesto de Pasajeros</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>M</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type34'>
        <field name='sequence'>60</field>
        <field name='code'>34</field>
        <field name='report_name'>Documento del Operador</field>
        <field name='name'>Documento del Operador</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>D</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type35'>
        <field name='sequence'>60</field>
        <field name='code'>35</field>
        <field name='report_name'>Documento del Partícipe</field>
        <field name='name'>Documento del Partícipe</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>D</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type36'>
        <field name='sequence'>60</field>
        <field name='code'>36</field>
        <field name='report_name'>Recibo de Distribución de Gas Natural</field>
        <field name='name'>Recibo de Distribución de Gas Natural</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>R</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type37'>
        <field name='sequence'>60</field>
        <field name='code'>37</field>
        <field name='report_name'>Documentos que emitan los concesionarios del servicio de revisiones técnicas vehiculares, por la prestación de dicho servicio</field>
        <field name='name'>Documentos que emitan los concesionarios del servicio de revisiones técnicas vehiculares, por la prestación de dicho servicio</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>D</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type40'>
        <field name='sequence'>60</field>
        <field name='code'>40</field>
        <field name='report_name'>Comprobante de percepción</field>
        <field name='name'>Comprobante de percepción</field>
        <field name='country_id' ref='base.pe'/>
        <field name='doc_code_prefix'>P</field>
        <field name='internal_type'/>
    </record>
    <record model='l10n_latam.document.type' id='document_type41'>
        <field name='sequence'>70</field>
        <field name='code'>41</field>
        <field name='report_name'>Comprobante de Percepción - Venta interna</field>
        <field name='name'>Comprobante de Percepción - Venta interna</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>C</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type42'>
        <field name='sequence'>70</field>
        <field name='code'>42</field>
        <field name='report_name'>Documentos emitidos por las empresas que desempeñan el rol adquiriente en los sistemas de pago mediante tarjetas de crédito emitidas por ellas mismas</field>
        <field name='name'>Documentos emitidos por las empresas que desempeñan el rol adquiriente en los sistemas de pago mediante tarjetas de crédito emitidas por ellas mismas</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>D</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type43'>
        <field name='sequence'>70</field>
        <field name='code'>43</field>
        <field name='report_name'>Boletos emitidos por las Compañías de Aviación Comercial que prestan servicios de transporte aéreo no regular de pasajeros y transporte aéreo especial de pasajeros.</field>
        <field name='name'>Boletos emitidos por las Compañías de Aviación Comercial que prestan servicios de transporte aéreo no regular de pasajeros y transporte aéreo especial de pasajeros.</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>B</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type44'>
        <field name='sequence'>70</field>
        <field name='code'>44</field>
        <field name='report_name'>Billetes de lotería, rifas y apuestas. </field>
        <field name='name'>Billetes de lotería, rifas y apuestas. </field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>B</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type45'>
        <field name='sequence'>70</field>
        <field name='code'>45</field>
        <field name='report_name'>Documentos emitidos por centros educativos y culturales, universidades, asociaciones y fundaciones, en lo referente a actividades no gravadas con tributos administrados por la SUNAT.</field>
        <field name='name'>Documentos emitidos por centros educativos y culturales, universidades, asociaciones y fundaciones, en lo referente a actividades no gravadas con tributos administrados por la SUNAT.</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>D</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type46'>
        <field name='sequence'>70</field>
        <field name='code'>46</field>
        <field name='report_name'>Formulario de Declaración - pago o Boleta de pago de tributos Internos</field>
        <field name='name'>Formulario de Declaración - pago o Boleta de pago de tributos Internos</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>F</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type48'>
        <field name='sequence'>70</field>
        <field name='code'>48</field>
        <field name='report_name'>Comprobante de Operaciones - Ley N° 29972</field>
        <field name='name'>Comprobante de Operaciones - Ley N° 29972</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>C</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type49'>
        <field name='sequence'>70</field>
        <field name='code'>49</field>
        <field name='report_name'>Constancia de Depósito - IVAP (Ley 28211)</field>
        <field name='name'>Constancia de Depósito - IVAP (Ley 28211)</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>C</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type50'>
        <field name='sequence'>70</field>
        <field name='code'>50</field>
        <field name='report_name'>Declaración Única de Aduanas - Importación definitiva</field>
        <field name='name'>Declaración Única de Aduanas - Importación definitiva</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>D</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type51'>
        <field name='sequence'>70</field>
        <field name='code'>51</field>
        <field name='report_name'>Póliza o DUI Fraccionada</field>
        <field name='name'>Póliza o DUI Fraccionada</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>P</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type52'>
        <field name='sequence'>70</field>
        <field name='code'>52</field>
        <field name='report_name'>Despacho Simplificado - Importación Simplificada</field>
        <field name='name'>Despacho Simplificado - Importación Simplificada</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>D</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type53'>
        <field name='sequence'>70</field>
        <field name='code'>53</field>
        <field name='report_name'>Declaración de Mensajería o Courier</field>
        <field name='name'>Declaración de Mensajería o Courier</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>D</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type54'>
        <field name='sequence'>70</field>
        <field name='code'>54</field>
        <field name='report_name'>Liquidación de Cobranza</field>
        <field name='name'>Liquidación de Cobranza</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>L</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type55'>
        <field name='sequence'>70</field>
        <field name='code'>55</field>
        <field name='report_name'>BVME para transporte ferroviario de pasajeros</field>
        <field name='name'>BVME para transporte ferroviario de pasajeros</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>B</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type56'>
        <field name='sequence'>70</field>
        <field name='code'>56</field>
        <field name='report_name'>Comprobante de pago SEAE</field>
        <field name='name'>Comprobante de pago SEAE</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>C</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type87'>
        <field name='sequence'>70</field>
        <field name='code'>87</field>
        <field name='report_name'>Nota de Crédito Especial</field>
        <field name='name'>Nota de Crédito Especial</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>N</field>
        <field name='internal_type'>credit_note</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type88'>
        <field name='sequence'>70</field>
        <field name='code'>88</field>
        <field name='report_name'>Nota de Débito Especial</field>
        <field name='name'>Nota de Débito Especial</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>N</field>
        <field name='internal_type'>debit_note</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type89'>
        <field name='sequence'>70</field>
        <field name='code'>89</field>
        <field name='report_name'>Nota de Ajuste de Operaciones - Ley N° 29972</field>
        <field name='name'>Nota de Ajuste de Operaciones - Ley N° 29972</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>N</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type91'>
        <field name='sequence'>70</field>
        <field name='code'>91</field>
        <field name='report_name'>Comprobante de No Domiciliado</field>
        <field name='name'>Comprobante de No Domiciliado</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>C</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type96'>
        <field name='sequence'>70</field>
        <field name='code'>96</field>
        <field name='report_name'>Exceso de crédito fiscal por retiro de bienes</field>
        <field name='name'>Exceso de crédito fiscal por retiro de bienes</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>E</field>
        <field name='internal_type'>invoice</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type97'>
        <field name='sequence'>70</field>
        <field name='code'>97</field>
        <field name='report_name'>Nota de Crédito - No Domiciliado</field>
        <field name='name'>Nota de Crédito - No Domiciliado</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>N</field>
        <field name='internal_type'>credit_note</field>
    </record>
    <record model='l10n_latam.document.type' id='document_type98'>
        <field name='sequence'>70</field>
        <field name='code'>98</field>
        <field name='report_name'>Nota de Débito - No Domiciliado</field>
        <field name='name'>Nota de Débito - No Domiciliado</field>
        <field name='country_id' ref='base.pe' />
        <field name='doc_code_prefix'>N</field>
        <field name='internal_type'>debit_note</field>
    </record>
</odoo>

```

## File: data\l10n_latam_identification_type_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model='l10n_latam.identification.type' id='l10n_latam_base.it_vat'>
        <field name='l10n_pe_vat_code'>0</field>
    </record>
    <record model='l10n_latam.identification.type' id='l10n_latam_base.it_pass'>
        <field name='l10n_pe_vat_code'>7</field>
    </record>
    <record model='l10n_latam.identification.type' id='l10n_latam_base.it_fid'>
        <field name='l10n_pe_vat_code'>4</field>
    </record>

    <record model='l10n_latam.identification.type' id='it_RUC'>
        <field name='name'>RUC</field>
        <field name='description'>Taxpayer Identification Number</field>
        <field name='country_id' ref='base.pe'/>
        <field name='is_vat' eval='True'/>
        <field name='l10n_pe_vat_code'>6</field>
        <field name='sequence'>10</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_DNI'>
        <field name='name'>DNI</field>
        <field name='description'>National Identity Document</field>
        <field name='country_id' ref='base.pe'/>
        <field name='l10n_pe_vat_code'>1</field>
        <field name='sequence'>82</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_NDTD'>
        <field name='name'>Non-Domiciled Tax Document</field>
        <field name='description'>Document without RUC from another country</field>
        <field name='country_id' ref='base.pe'/>
        <field name='l10n_pe_vat_code'>0</field>
        <field name='sequence'>86</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_DIC'>
        <field name='name'>Diplomatic Identity Card</field>
        <field name='country_id' ref='base.pe'/>
        <field name='l10n_pe_vat_code'>A</field>
        <field name='sequence'>105</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_IDCR'>
        <field name='name'>Identity document of the country of residence</field>
        <field name='country_id' ref='base.pe'/>
        <field name='l10n_pe_vat_code'>B</field>
        <field name='sequence'>110</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_TIN'>
        <field name='name'>Tax Identification Number</field>
        <field name='description'>TIN – Doc Trib PP.NN</field>
        <field name='country_id' ref='base.pe'/>
        <field name='l10n_pe_vat_code'>C</field>
        <field name='sequence'>115</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_IN'>
        <field name='name'>Identification Number</field>
        <field name='description'>IN - Doc Trib PP. JJ</field>
        <field name='country_id' ref='base.pe'/>
        <field name='l10n_pe_vat_code'>D</field>
        <field name='sequence'>120</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_TAM'>
        <field name='name'>TAM</field>
        <field name='description'>Andean Immigration Card</field>
        <field name='country_id' ref='base.pe'/>
        <field name='l10n_pe_vat_code'>E</field>
        <field name='sequence'>125</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_PTP'>
        <field name='name'>PTP</field>
        <field name='description'>Temporary Residence Permit</field>
        <field name='country_id' ref='base.pe'/>
        <field name='l10n_pe_vat_code'>F</field>
        <field name='sequence'>130</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_SP'>
        <field name='name'>Safe Passage</field>
        <field name='country_id' ref='base.pe'/>
        <field name='l10n_pe_vat_code'>G</field>
        <field name='sequence'>135</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CPP'>
        <field name='name'>License Permit Temp. Perman.</field>
        <field name='country_id' ref='base.pe'/>
        <field name='l10n_pe_vat_code'>H</field>
        <field name='sequence'>140</field>
    </record>
</odoo>

```

## File: data\l10n_pe.res.city.district.csv

```csv
"id","city_id:id","name","code"
district_pe_010101,city_pe_0101,"Chachapoyas","010101"
district_pe_010102,city_pe_0101,"Asunción","010102"
district_pe_010103,city_pe_0101,"Balsas","010103"
district_pe_010104,city_pe_0101,"Cheto","010104"
district_pe_010105,city_pe_0101,"Chiliquin","010105"
district_pe_010106,city_pe_0101,"Chuquibamba","010106"
district_pe_010107,city_pe_0101,"Granada","010107"
district_pe_010108,city_pe_0101,"Huancas","010108"
district_pe_010109,city_pe_0101,"La Jalca","010109"
district_pe_010110,city_pe_0101,"Leimebamba","010110"
district_pe_010111,city_pe_0101,"Levanto","010111"
district_pe_010112,city_pe_0101,"Magdalena","010112"
district_pe_010113,city_pe_0101,"Mariscal Castilla","010113"
district_pe_010114,city_pe_0101,"Molinopampa","010114"
district_pe_010115,city_pe_0101,"Montevideo","010115"
district_pe_010116,city_pe_0101,"Olleros","010116"
district_pe_010117,city_pe_0101,"Quinjalca","010117"
district_pe_010118,city_pe_0101,"San Francisco de Daguas","010118"
district_pe_010119,city_pe_0101,"San Isidro de Maino","010119"
district_pe_010120,city_pe_0101,"Soloco","010120"
district_pe_010121,city_pe_0101,"Sonche","010121"
district_pe_010201,city_pe_0102,"Bagua","010201"
district_pe_010202,city_pe_0102,"Aramango","010202"
district_pe_010203,city_pe_0102,"Copallin","010203"
district_pe_010204,city_pe_0102,"El Parco","010204"
district_pe_010205,city_pe_0102,"Imaza","010205"
district_pe_010206,city_pe_0102,"La Peca","010206"
district_pe_010301,city_pe_0103,"Jumbilla","010301"
district_pe_010302,city_pe_0103,"Chisquilla","010302"
district_pe_010303,city_pe_0103,"Churuja","010303"
district_pe_010304,city_pe_0103,"Corosha","010304"
district_pe_010305,city_pe_0103,"Cuispes","010305"
district_pe_010306,city_pe_0103,"Florida","010306"
district_pe_010307,city_pe_0103,"Jazan","010307"
district_pe_010308,city_pe_0103,"Recta","010308"
district_pe_010309,city_pe_0103,"San Carlos","010309"
district_pe_010310,city_pe_0103,"Shipasbamba","010310"
district_pe_010311,city_pe_0103,"Valera","010311"
district_pe_010312,city_pe_0103,"Yambrasbamba","010312"
district_pe_010401,city_pe_0104,"Nieva","010401"
district_pe_010402,city_pe_0104,"El Cenepa","010402"
district_pe_010403,city_pe_0104,"Río Santiago","010403"
district_pe_010501,city_pe_0105,"Lamud","010501"
district_pe_010502,city_pe_0105,"Camporredondo","010502"
district_pe_010503,city_pe_0105,"Cocabamba","010503"
district_pe_010504,city_pe_0105,"Colcamar","010504"
district_pe_010505,city_pe_0105,"Conila","010505"
district_pe_010506,city_pe_0105,"Inguilpata","010506"
district_pe_010507,city_pe_0105,"Longuita","010507"
district_pe_010508,city_pe_0105,"Lonya Chico","010508"
district_pe_010509,city_pe_0105,"Luya","010509"
district_pe_010510,city_pe_0105,"Luya Viejo","010510"
district_pe_010511,city_pe_0105,"María","010511"
district_pe_010512,city_pe_0105,"Ocalli","010512"
district_pe_010513,city_pe_0105,"Ocumal","010513"
district_pe_010514,city_pe_0105,"Pisuquia","010514"
district_pe_010515,city_pe_0105,"Providencia","010515"
district_pe_010516,city_pe_0105,"San Cristóbal","010516"
district_pe_010517,city_pe_0105,"San Francisco del Yeso","010517"
district_pe_010518,city_pe_0105,"San Jerónimo","010518"
district_pe_010519,city_pe_0105,"San Juan de Lopecancha","010519"
district_pe_010520,city_pe_0105,"Santa Catalina","010520"
district_pe_010521,city_pe_0105,"Santo Tomas","010521"
district_pe_010522,city_pe_0105,"Tingo","010522"
district_pe_010523,city_pe_0105,"Trita","010523"
district_pe_010601,city_pe_0106,"San Nicolás","010601"
district_pe_010602,city_pe_0106,"Chirimoto","010602"
district_pe_010603,city_pe_0106,"Cochamal","010603"
district_pe_010604,city_pe_0106,"Huambo","010604"
district_pe_010605,city_pe_0106,"Limabamba","010605"
district_pe_010606,city_pe_0106,"Longar","010606"
district_pe_010607,city_pe_0106,"Mariscal Benavides","010607"
district_pe_010608,city_pe_0106,"Milpuc","010608"
district_pe_010609,city_pe_0106,"Omia","010609"
district_pe_010610,city_pe_0106,"Santa Rosa","010610"
district_pe_010611,city_pe_0106,"Totora","010611"
district_pe_010612,city_pe_0106,"Vista Alegre","010612"
district_pe_010701,city_pe_0107,"Bagua Grande","010701"
district_pe_010702,city_pe_0107,"Cajaruro","010702"
district_pe_010703,city_pe_0107,"Cumba","010703"
district_pe_010704,city_pe_0107,"El Milagro","010704"
district_pe_010705,city_pe_0107,"Jamalca","010705"
district_pe_010706,city_pe_0107,"Lonya Grande","010706"
district_pe_010707,city_pe_0107,"Yamon","010707"
district_pe_020101,city_pe_0201,"Huaraz","020101"
district_pe_020102,city_pe_0201,"Cochabamba","020102"
district_pe_020103,city_pe_0201,"Colcabamba","020103"
district_pe_020104,city_pe_0201,"Huanchay","020104"
district_pe_020105,city_pe_0201,"Independencia","020105"
district_pe_020106,city_pe_0201,"Jangas","020106"
district_pe_020107,city_pe_0201,"La Libertad","020107"
district_pe_020108,city_pe_0201,"Olleros","020108"
district_pe_020109,city_pe_0201,"Pampas","020109"
district_pe_020110,city_pe_0201,"Pariacoto","020110"
district_pe_020111,city_pe_0201,"Pira","020111"
district_pe_020112,city_pe_0201,"Tarica","020112"
district_pe_020201,city_pe_0202,"Aija","020201"
district_pe_020202,city_pe_0202,"Coris","020202"
district_pe_020203,city_pe_0202,"Huacllan","020203"
district_pe_020204,city_pe_0202,"La Merced","020204"
district_pe_020205,city_pe_0202,"Succha","020205"
district_pe_020301,city_pe_0203,"Llamellin","020301"
district_pe_020302,city_pe_0203,"Aczo","020302"
district_pe_020303,city_pe_0203,"Chaccho","020303"
district_pe_020304,city_pe_0203,"Chingas","020304"
district_pe_020305,city_pe_0203,"Mirgas","020305"
district_pe_020306,city_pe_0203,"San Juan de Rontoy","020306"
district_pe_020401,city_pe_0204,"Chacas","020401"
district_pe_020402,city_pe_0204,"Acochaca","020402"
district_pe_020501,city_pe_0205,"Chiquian","020501"
district_pe_020502,city_pe_0205,"Abelardo Pardo Lezameta","020502"
district_pe_020503,city_pe_0205,"Antonio Raymondi","020503"
district_pe_020504,city_pe_0205,"Aquia","020504"
district_pe_020505,city_pe_0205,"Cajacay","020505"
district_pe_020506,city_pe_0205,"Canis","020506"
district_pe_020507,city_pe_0205,"Colquioc","020507"
district_pe_020508,city_pe_0205,"Huallanca","020508"
district_pe_020509,city_pe_0205,"Huasta","020509"
district_pe_020510,city_pe_0205,"Huayllacayan","020510"
district_pe_020511,city_pe_0205,"La Primavera","020511"
district_pe_020512,city_pe_0205,"Mangas","020512"
district_pe_020513,city_pe_0205,"Pacllon","020513"
district_pe_020514,city_pe_0205,"San Miguel de Corpanqui","020514"
district_pe_020515,city_pe_0205,"Ticllos","020515"
district_pe_020601,city_pe_0206,"Carhuaz","020601"
district_pe_020602,city_pe_0206,"Acopampa","020602"
district_pe_020603,city_pe_0206,"Amashca","020603"
district_pe_020604,city_pe_0206,"Anta","020604"
district_pe_020605,city_pe_0206,"Ataquero","020605"
district_pe_020606,city_pe_0206,"Marcara","020606"
district_pe_020607,city_pe_0206,"Pariahuanca","020607"
district_pe_020608,city_pe_0206,"San Miguel de Aco","020608"
district_pe_020609,city_pe_0206,"Shilla","020609"
district_pe_020610,city_pe_0206,"Tinco","020610"
district_pe_020611,city_pe_0206,"Yungar","020611"
district_pe_020701,city_pe_0207,"San Luis","020701"
district_pe_020702,city_pe_0207,"San Nicolás","020702"
district_pe_020703,city_pe_0207,"Yauya","020703"
district_pe_020801,city_pe_0208,"Casma","020801"
district_pe_020802,city_pe_0208,"Buena Vista Alta","020802"
district_pe_020803,city_pe_0208,"Comandante Noel","020803"
district_pe_020804,city_pe_0208,"Yautan","020804"
district_pe_020901,city_pe_0209,"Corongo","020901"
district_pe_020902,city_pe_0209,"Aco","020902"
district_pe_020903,city_pe_0209,"Bambas","020903"
district_pe_020904,city_pe_0209,"Cusca","020904"
district_pe_020905,city_pe_0209,"La Pampa","020905"
district_pe_020906,city_pe_0209,"Yanac","020906"
district_pe_020907,city_pe_0209,"Yupan","020907"
district_pe_021001,city_pe_0210,"Huari","021001"
district_pe_021002,city_pe_0210,"Anra","021002"
district_pe_021003,city_pe_0210,"Cajay","021003"
district_pe_021004,city_pe_0210,"Chavín de Huantar","021004"
district_pe_021005,city_pe_0210,"Huacachi","021005"
district_pe_021006,city_pe_0210,"Huacchis","021006"
district_pe_021007,city_pe_0210,"Huachis","021007"
district_pe_021008,city_pe_0210,"Huantar","021008"
district_pe_021009,city_pe_0210,"Masin","021009"
district_pe_021010,city_pe_0210,"Paucas","021010"
district_pe_021011,city_pe_0210,"Ponto","021011"
district_pe_021012,city_pe_0210,"Rahuapampa","021012"
district_pe_021013,city_pe_0210,"Rapayan","021013"
district_pe_021014,city_pe_0210,"San Marcos","021014"
district_pe_021015,city_pe_0210,"San Pedro de chana","021015"
district_pe_021016,city_pe_0210,"Uco","021016"
district_pe_021101,city_pe_0211,"Huarmey","021101"
district_pe_021102,city_pe_0211,"Cochapeti","021102"
district_pe_021103,city_pe_0211,"Culebras","021103"
district_pe_021104,city_pe_0211,"Huayan","021104"
district_pe_021105,city_pe_0211,"Malvas","021105"
district_pe_021201,city_pe_0212,"Caraz","021201"
district_pe_021202,city_pe_0212,"Huallanca","021202"
district_pe_021203,city_pe_0212,"Huata","021203"
district_pe_021204,city_pe_0212,"Huaylas","021204"
district_pe_021205,city_pe_0212,"Mato","021205"
district_pe_021206,city_pe_0212,"Pamparomas","021206"
district_pe_021207,city_pe_0212,"Pueblo Libre","021207"
district_pe_021208,city_pe_0212,"Santa Cruz","021208"
district_pe_021209,city_pe_0212,"Santo Toribio","021209"
district_pe_021210,city_pe_0212,"Yuracmarca","021210"
district_pe_021301,city_pe_0213,"Piscobamba","021301"
district_pe_021302,city_pe_0213,"Casca","021302"
district_pe_021303,city_pe_0213,"Eleazar Guzmán Barron","021303"
district_pe_021304,city_pe_0213,"Fidel Olivas Escudero","021304"
district_pe_021305,city_pe_0213,"Llama","021305"
district_pe_021306,city_pe_0213,"Llumpa","021306"
district_pe_021307,city_pe_0213,"Lucma","021307"
district_pe_021308,city_pe_0213,"Musga","021308"
district_pe_021401,city_pe_0214,"Ocros","021401"
district_pe_021402,city_pe_0214,"Acas","021402"
district_pe_021403,city_pe_0214,"Cajamarquilla","021403"
district_pe_021404,city_pe_0214,"Carhuapampa","021404"
district_pe_021405,city_pe_0214,"Cochas","021405"
district_pe_021406,city_pe_0214,"Congas","021406"
district_pe_021407,city_pe_0214,"Llipa","021407"
district_pe_021408,city_pe_0214,"San Cristóbal de Rajan","021408"
district_pe_021409,city_pe_0214,"San Pedro","021409"
district_pe_021410,city_pe_0214,"Santiago de chilcas","021410"
district_pe_021501,city_pe_0215,"Cabana","021501"
district_pe_021502,city_pe_0215,"Bolognesi","021502"
district_pe_021503,city_pe_0215,"Conchucos","021503"
district_pe_021504,city_pe_0215,"Huacaschuque","021504"
district_pe_021505,city_pe_0215,"Huandoval","021505"
district_pe_021506,city_pe_0215,"Lacabamba","021506"
district_pe_021507,city_pe_0215,"Llapo","021507"
district_pe_021508,city_pe_0215,"Pallasca","021508"
district_pe_021509,city_pe_0215,"Pampas","021509"
district_pe_021510,city_pe_0215,"Santa Rosa","021510"
district_pe_021511,city_pe_0215,"Tauca","021511"
district_pe_021601,city_pe_0216,"Pomabamba","021601"
district_pe_021602,city_pe_0216,"Huayllan","021602"
district_pe_021603,city_pe_0216,"Parobamba","021603"
district_pe_021604,city_pe_0216,"Quinuabamba","021604"
district_pe_021701,city_pe_0217,"Recuay","021701"
district_pe_021702,city_pe_0217,"Catac","021702"
district_pe_021703,city_pe_0217,"Cotaparaco","021703"
district_pe_021704,city_pe_0217,"Huayllapampa","021704"
district_pe_021705,city_pe_0217,"Llacllin","021705"
district_pe_021706,city_pe_0217,"Marca","021706"
district_pe_021707,city_pe_0217,"Pampas chico","021707"
district_pe_021708,city_pe_0217,"Pararin","021708"
district_pe_021709,city_pe_0217,"Tapacocha","021709"
district_pe_021710,city_pe_0217,"Ticapampa","021710"
district_pe_021801,city_pe_0218,"Chimbote","021801"
district_pe_021802,city_pe_0218,"Cáceres del Perú","021802"
district_pe_021803,city_pe_0218,"Coishco","021803"
district_pe_021804,city_pe_0218,"Macate","021804"
district_pe_021805,city_pe_0218,"Moro","021805"
district_pe_021806,city_pe_0218,"Nepeña","021806"
district_pe_021807,city_pe_0218,"Samanco","021807"
district_pe_021808,city_pe_0218,"Santa","021808"
district_pe_021809,city_pe_0218,"Nuevo chimbote","021809"
district_pe_021901,city_pe_0219,"Sihuas","021901"
district_pe_021902,city_pe_0219,"Acobamba","021902"
district_pe_021903,city_pe_0219,"Alfonso Ugarte","021903"
district_pe_021904,city_pe_0219,"Cashapampa","021904"
district_pe_021905,city_pe_0219,"Chingalpo","021905"
district_pe_021906,city_pe_0219,"Huayllabamba","021906"
district_pe_021907,city_pe_0219,"Quiches","021907"
district_pe_021908,city_pe_0219,"Ragash","021908"
district_pe_021909,city_pe_0219,"San Juan","021909"
district_pe_021910,city_pe_0219,"Sicsibamba","021910"
district_pe_022001,city_pe_0220,"Yungay","022001"
district_pe_022002,city_pe_0220,"Cascapara","022002"
district_pe_022003,city_pe_0220,"Mancos","022003"
district_pe_022004,city_pe_0220,"Matacoto","022004"
district_pe_022005,city_pe_0220,"Quillo","022005"
district_pe_022006,city_pe_0220,"Ranrahirca","022006"
district_pe_022007,city_pe_0220,"Shupluy","022007"
district_pe_022008,city_pe_0220,"Yanama","022008"
district_pe_030101,city_pe_0301,"Abancay","030101"
district_pe_030102,city_pe_0301,"Chacoche","030102"
district_pe_030103,city_pe_0301,"Circa","030103"
district_pe_030104,city_pe_0301,"Curahuasi","030104"
district_pe_030105,city_pe_0301,"Huanipaca","030105"
district_pe_030106,city_pe_0301,"Lambrama","030106"
district_pe_030107,city_pe_0301,"Pichirhua","030107"
district_pe_030108,city_pe_0301,"San Pedro de Cachora","030108"
district_pe_030109,city_pe_0301,"Tamburco","030109"
district_pe_030201,city_pe_0302,"Andahuaylas","030201"
district_pe_030202,city_pe_0302,"Andarapa","030202"
district_pe_030203,city_pe_0302,"Chiara","030203"
district_pe_030204,city_pe_0302,"Huancarama","030204"
district_pe_030205,city_pe_0302,"Huancaray","030205"
district_pe_030206,city_pe_0302,"Huayana","030206"
district_pe_030207,city_pe_0302,"Kishuara","030207"
district_pe_030208,city_pe_0302,"Pacobamba","030208"
district_pe_030209,city_pe_0302,"Pacucha","030209"
district_pe_030210,city_pe_0302,"Pampachiri","030210"
district_pe_030211,city_pe_0302,"Pomacocha","030211"
district_pe_030212,city_pe_0302,"San Antonio de Cachi","030212"
district_pe_030213,city_pe_0302,"San Jerónimo","030213"
district_pe_030214,city_pe_0302,"San Miguel de Chaccrampa","030214"
district_pe_030215,city_pe_0302,"Santa María de Chicmo","030215"
district_pe_030216,city_pe_0302,"Talavera","030216"
district_pe_030217,city_pe_0302,"Tumay Huaraca","030217"
district_pe_030218,city_pe_0302,"Turpo","030218"
district_pe_030219,city_pe_0302,"Kaquiabamba","030219"
district_pe_030220,city_pe_0302,"José María Arguedas","030220"
district_pe_030301,city_pe_0303,"Antabamba","030301"
district_pe_030302,city_pe_0303,"El Oro","030302"
district_pe_030303,city_pe_0303,"Huaquirca","030303"
district_pe_030304,city_pe_0303,"Juan Espinoza Medrano","030304"
district_pe_030305,city_pe_0303,"Oropesa","030305"
district_pe_030306,city_pe_0303,"Pachaconas","030306"
district_pe_030307,city_pe_0303,"Sabaino","030307"
district_pe_030401,city_pe_0304,"Chalhuanca","030401"
district_pe_030402,city_pe_0304,"Capaya","030402"
district_pe_030403,city_pe_0304,"Caraybamba","030403"
district_pe_030404,city_pe_0304,"Chapimarca","030404"
district_pe_030405,city_pe_0304,"Colcabamba","030405"
district_pe_030406,city_pe_0304,"Cotaruse","030406"
district_pe_030407,city_pe_0304,"Huayllo","030407"
district_pe_030408,city_pe_0304,"Justo Apu Sahuaraura","030408"
district_pe_030409,city_pe_0304,"Lucre","030409"
district_pe_030410,city_pe_0304,"Pocohuanca","030410"
district_pe_030411,city_pe_0304,"San Juan de Chacña","030411"
district_pe_030412,city_pe_0304,"Sañayca","030412"
district_pe_030413,city_pe_0304,"Soraya","030413"
district_pe_030414,city_pe_0304,"Tapairihua","030414"
district_pe_030415,city_pe_0304,"Tintay","030415"
district_pe_030416,city_pe_0304,"Toraya","030416"
district_pe_030417,city_pe_0304,"Yanaca","030417"
district_pe_030501,city_pe_0305,"Tambobamba","030501"
district_pe_030502,city_pe_0305,"Cotabambas","030502"
district_pe_030503,city_pe_0305,"Coyllurqui","030503"
district_pe_030504,city_pe_0305,"Haquira","030504"
district_pe_030505,city_pe_0305,"Mara","030505"
district_pe_030506,city_pe_0305,"Challhuahuacho","030506"
district_pe_030601,city_pe_0306,"Chincheros","030601"
district_pe_030602,city_pe_0306,"Anco_huallo","030602"
district_pe_030603,city_pe_0306,"Cocharcas","030603"
district_pe_030604,city_pe_0306,"Huaccana","030604"
district_pe_030605,city_pe_0306,"Ocobamba","030605"
district_pe_030606,city_pe_0306,"Ongoy","030606"
district_pe_030607,city_pe_0306,"Uranmarca","030607"
district_pe_030608,city_pe_0306,"Ranracancha","030608"
district_pe_030609,city_pe_0306,"Rocchacc","030609"
district_pe_030610,city_pe_0306,"El Porvenir","030610"
district_pe_030611,city_pe_0306,"Los Chankas","030611"
district_pe_030701,city_pe_0307,"Chuquibambilla","030701"
district_pe_030702,city_pe_0307,"Curpahuasi","030702"
district_pe_030703,city_pe_0307,"Gamarra","030703"
district_pe_030704,city_pe_0307,"Huayllati","030704"
district_pe_030705,city_pe_0307,"Mamara","030705"
district_pe_030706,city_pe_0307,"Micaela Bastidas","030706"
district_pe_030707,city_pe_0307,"Pataypampa","030707"
district_pe_030708,city_pe_0307,"Progreso","030708"
district_pe_030709,city_pe_0307,"San Antonio","030709"
district_pe_030710,city_pe_0307,"Santa Rosa","030710"
district_pe_030711,city_pe_0307,"Turpay","030711"
district_pe_030712,city_pe_0307,"Vilcabamba","030712"
district_pe_030713,city_pe_0307,"Virundo","030713"
district_pe_030714,city_pe_0307,"Curasco","030714"
district_pe_040101,city_pe_0401,"Arequipa","040101"
district_pe_040102,city_pe_0401,"Alto Selva Alegre","040102"
district_pe_040103,city_pe_0401,"Cayma","040103"
district_pe_040104,city_pe_0401,"Cerro colorado","040104"
district_pe_040105,city_pe_0401,"Characato","040105"
district_pe_040106,city_pe_0401,"Chiguata","040106"
district_pe_040107,city_pe_0401,"Jacobo Hunter","040107"
district_pe_040108,city_pe_0401,"La Joya","040108"
district_pe_040109,city_pe_0401,"Maríano Melgar","040109"
district_pe_040110,city_pe_0401,"Miraflores","040110"
district_pe_040111,city_pe_0401,"Mollebaya","040111"
district_pe_040112,city_pe_0401,"Paucarpata","040112"
district_pe_040113,city_pe_0401,"Pocsi","040113"
district_pe_040114,city_pe_0401,"Polobaya","040114"
district_pe_040115,city_pe_0401,"Quequeña","040115"
district_pe_040116,city_pe_0401,"Sabandia","040116"
district_pe_040117,city_pe_0401,"Sachaca","040117"
district_pe_040118,city_pe_0401,"San Juan de Siguas","040118"
district_pe_040119,city_pe_0401,"San Juan de Tarucani","040119"
district_pe_040120,city_pe_0401,"Santa Isabel de Siguas","040120"
district_pe_040121,city_pe_0401,"Santa Rita de Siguas","040121"
district_pe_040122,city_pe_0401,"Socabaya","040122"
district_pe_040123,city_pe_0401,"Tiabaya","040123"
district_pe_040124,city_pe_0401,"Uchumayo","040124"
district_pe_040125,city_pe_0401,"Vitor","040125"
district_pe_040126,city_pe_0401,"Yanahuara","040126"
district_pe_040127,city_pe_0401,"Yarabamba","040127"
district_pe_040128,city_pe_0401,"Yura","040128"
district_pe_040129,city_pe_0401,"José Luis Bustamante Y Rivero","040129"
district_pe_040201,city_pe_0402,"Camaná","040201"
district_pe_040202,city_pe_0402,"José María Quimper","040202"
district_pe_040203,city_pe_0402,"Maríano Nicolás Valcárcel","040203"
district_pe_040204,city_pe_0402,"Mariscal Cáceres","040204"
district_pe_040205,city_pe_0402,"Nicolás de Pierola","040205"
district_pe_040206,city_pe_0402,"Ocoña","040206"
district_pe_040207,city_pe_0402,"Quilca","040207"
district_pe_040208,city_pe_0402,"Samuel Pastor","040208"
district_pe_040301,city_pe_0403,"Caravelí","040301"
district_pe_040302,city_pe_0403,"Acarí","040302"
district_pe_040303,city_pe_0403,"Atico","040303"
district_pe_040304,city_pe_0403,"Atiquipa","040304"
district_pe_040305,city_pe_0403,"Bella Unión","040305"
district_pe_040306,city_pe_0403,"Cahuacho","040306"
district_pe_040307,city_pe_0403,"Chala","040307"
district_pe_040308,city_pe_0403,"Chaparra","040308"
district_pe_040309,city_pe_0403,"Huanuhuanu","040309"
district_pe_040310,city_pe_0403,"Jaqui","040310"
district_pe_040311,city_pe_0403,"Lomas","040311"
district_pe_040312,city_pe_0403,"Quicacha","040312"
district_pe_040313,city_pe_0403,"Yauca","040313"
district_pe_040401,city_pe_0404,"Aplao","040401"
district_pe_040402,city_pe_0404,"Andagua","040402"
district_pe_040403,city_pe_0404,"Ayo","040403"
district_pe_040404,city_pe_0404,"Chachas","040404"
district_pe_040405,city_pe_0404,"Chilcaymarca","040405"
district_pe_040406,city_pe_0404,"Choco","040406"
district_pe_040407,city_pe_0404,"Huancarqui","040407"
district_pe_040408,city_pe_0404,"Machaguay","040408"
district_pe_040409,city_pe_0404,"Orcopampa","040409"
district_pe_040410,city_pe_0404,"Pampacolca","040410"
district_pe_040411,city_pe_0404,"Tipan","040411"
district_pe_040412,city_pe_0404,"Uñon","040412"
district_pe_040413,city_pe_0404,"Uraca","040413"
district_pe_040414,city_pe_0404,"Viraco","040414"
district_pe_040501,city_pe_0405,"Chivay","040501"
district_pe_040502,city_pe_0405,"Achoma","040502"
district_pe_040503,city_pe_0405,"Cabanaconde","040503"
district_pe_040504,city_pe_0405,"Callalli","040504"
district_pe_040505,city_pe_0405,"Caylloma","040505"
district_pe_040506,city_pe_0405,"Coporaque","040506"
district_pe_040507,city_pe_0405,"Huambo","040507"
district_pe_040508,city_pe_0405,"Huanca","040508"
district_pe_040509,city_pe_0405,"Ichupampa","040509"
district_pe_040510,city_pe_0405,"Lari","040510"
district_pe_040511,city_pe_0405,"Lluta","040511"
district_pe_040512,city_pe_0405,"Maca","040512"
district_pe_040513,city_pe_0405,"Madrigal","040513"
district_pe_040514,city_pe_0405,"San Antonio de Ahuca","040514"
district_pe_040515,city_pe_0405,"Sibayo","040515"
district_pe_040516,city_pe_0405,"Tapay","040516"
district_pe_040517,city_pe_0405,"Tisco","040517"
district_pe_040518,city_pe_0405,"Tuti","040518"
district_pe_040519,city_pe_0405,"Yanque","040519"
district_pe_040520,city_pe_0405,"Majes","040520"
district_pe_040601,city_pe_0406,"Chuquibamba","040601"
district_pe_040602,city_pe_0406,"Andaray","040602"
district_pe_040603,city_pe_0406,"Cayarani","040603"
district_pe_040604,city_pe_0406,"Chichas","040604"
district_pe_040605,city_pe_0406,"Iray","040605"
district_pe_040606,city_pe_0406,"Río Grande","040606"
district_pe_040607,city_pe_0406,"Salamanca","040607"
district_pe_040608,city_pe_0406,"Yanaquihua","040608"
district_pe_040701,city_pe_0407,"Mollendo","040701"
district_pe_040702,city_pe_0407,"Cocachacra","040702"
district_pe_040703,city_pe_0407,"Dean Valdivia","040703"
district_pe_040704,city_pe_0407,"Islay","040704"
district_pe_040705,city_pe_0407,"Mejia","040705"
district_pe_040706,city_pe_0407,"Punta de Bombón","040706"
district_pe_040801,city_pe_0408,"Cotahuasi","040801"
district_pe_040802,city_pe_0408,"Alca","040802"
district_pe_040803,city_pe_0408,"Charcana","040803"
district_pe_040804,city_pe_0408,"Huaynacotas","040804"
district_pe_040805,city_pe_0408,"Pampamarca","040805"
district_pe_040806,city_pe_0408,"Puyca","040806"
district_pe_040807,city_pe_0408,"Quechualla","040807"
district_pe_040808,city_pe_0408,"Sayla","040808"
district_pe_040809,city_pe_0408,"Tauria","040809"
district_pe_040810,city_pe_0408,"Tomepampa","040810"
district_pe_040811,city_pe_0408,"Toro","040811"
district_pe_050101,city_pe_0501,"Ayacucho","050101"
district_pe_050102,city_pe_0501,"Acocro","050102"
district_pe_050103,city_pe_0501,"Acos Vinchos","050103"
district_pe_050104,city_pe_0501,"Carmen Alto","050104"
district_pe_050105,city_pe_0501,"Chiara","050105"
district_pe_050106,city_pe_0501,"Ocros","050106"
district_pe_050107,city_pe_0501,"Pacaycasa","050107"
district_pe_050108,city_pe_0501,"Quinua","050108"
district_pe_050109,city_pe_0501,"San José de Ticllas","050109"
district_pe_050110,city_pe_0501,"San Juan Bautista","050110"
district_pe_050111,city_pe_0501,"Santiago de Pischa","050111"
district_pe_050112,city_pe_0501,"Socos","050112"
district_pe_050113,city_pe_0501,"Tambillo","050113"
district_pe_050114,city_pe_0501,"Vinchos","050114"
district_pe_050115,city_pe_0501,"Jesús Nazareno","050115"
district_pe_050116,city_pe_0501,"Andrés Avelino Cáceres Dorregaray","050116"
district_pe_050201,city_pe_0502,"Cangallo","050201"
district_pe_050202,city_pe_0502,"Chuschi","050202"
district_pe_050203,city_pe_0502,"Los Morochucos","050203"
district_pe_050204,city_pe_0502,"María Parado de Bellido","050204"
district_pe_050205,city_pe_0502,"Paras","050205"
district_pe_050206,city_pe_0502,"Totos","050206"
district_pe_050301,city_pe_0503,"Sancos","050301"
district_pe_050302,city_pe_0503,"Carapo","050302"
district_pe_050303,city_pe_0503,"Sacsamarca","050303"
district_pe_050304,city_pe_0503,"Santiago de Lucanamarca","050304"
district_pe_050401,city_pe_0504,"Huanta","050401"
district_pe_050402,city_pe_0504,"Ayahuanco","050402"
district_pe_050403,city_pe_0504,"Huamanguilla","050403"
district_pe_050404,city_pe_0504,"Iguain","050404"
district_pe_050405,city_pe_0504,"Luricocha","050405"
district_pe_050406,city_pe_0504,"Santillana","050406"
district_pe_050407,city_pe_0504,"Sivia","050407"
district_pe_050408,city_pe_0504,"Llochegua","050408"
district_pe_050409,city_pe_0504,"Canayre","050409"
district_pe_050410,city_pe_0504,"Uchuraccay","050410"
district_pe_050411,city_pe_0504,"Pucacolpa","050411"
district_pe_050412,city_pe_0504,"Chaca","050412"
district_pe_050501,city_pe_0505,"San Miguel","050501"
district_pe_050502,city_pe_0505,"Anco","050502"
district_pe_050503,city_pe_0505,"Ayna","050503"
district_pe_050504,city_pe_0505,"Chilcas","050504"
district_pe_050505,city_pe_0505,"Chungui","050505"
district_pe_050506,city_pe_0505,"Luis Carranza","050506"
district_pe_050507,city_pe_0505,"Santa Rosa","050507"
district_pe_050508,city_pe_0505,"Tambo","050508"
district_pe_050509,city_pe_0505,"Samugari","050509"
district_pe_050510,city_pe_0505,"Anchihuay","050510"
district_pe_050511,city_pe_0505,"Oronccoy","050511"
district_pe_050601,city_pe_0506,"Puquio","050601"
district_pe_050602,city_pe_0506,"Aucara","050602"
district_pe_050603,city_pe_0506,"Cabana","050603"
district_pe_050604,city_pe_0506,"Carmen Salcedo","050604"
district_pe_050605,city_pe_0506,"Chaviña","050605"
district_pe_050606,city_pe_0506,"Chipao","050606"
district_pe_050607,city_pe_0506,"Huac-Huas","050607"
district_pe_050608,city_pe_0506,"Laramate","050608"
district_pe_050609,city_pe_0506,"Leoncio Prado","050609"
district_pe_050610,city_pe_0506,"Llauta","050610"
district_pe_050611,city_pe_0506,"Lucanas","050611"
district_pe_050612,city_pe_0506,"Ocaña","050612"
district_pe_050613,city_pe_0506,"Otoca","050613"
district_pe_050614,city_pe_0506,"Saisa","050614"
district_pe_050615,city_pe_0506,"San Cristóbal","050615"
district_pe_050616,city_pe_0506,"San Juan","050616"
district_pe_050617,city_pe_0506,"San Pedro","050617"
district_pe_050618,city_pe_0506,"San Pedro de Palco","050618"
district_pe_050619,city_pe_0506,"Sancos","050619"
district_pe_050620,city_pe_0506,"Santa Ana de Huaycahuacho","050620"
district_pe_050621,city_pe_0506,"Santa Lucia","050621"
district_pe_050701,city_pe_0507,"Coracora","050701"
district_pe_050702,city_pe_0507,"Chumpi","050702"
district_pe_050703,city_pe_0507,"Coronel Castañeda","050703"
district_pe_050704,city_pe_0507,"Pacapausa","050704"
district_pe_050705,city_pe_0507,"Pullo","050705"
district_pe_050706,city_pe_0507,"Puyusca","050706"
district_pe_050707,city_pe_0507,"San Francisco de Ravacayco","050707"
district_pe_050708,city_pe_0507,"Upahuacho","050708"
district_pe_050801,city_pe_0508,"Pausa","050801"
district_pe_050802,city_pe_0508,"Colta","050802"
district_pe_050803,city_pe_0508,"Corculla","050803"
district_pe_050804,city_pe_0508,"Lampa","050804"
district_pe_050805,city_pe_0508,"Marcabamba","050805"
district_pe_050806,city_pe_0508,"Oyolo","050806"
district_pe_050807,city_pe_0508,"Pararca","050807"
district_pe_050808,city_pe_0508,"San Javier de Alpabamba","050808"
district_pe_050809,city_pe_0508,"San José de ushua","050809"
district_pe_050810,city_pe_0508,"Sara Sara","050810"
district_pe_050901,city_pe_0509,"Querobamba","050901"
district_pe_050902,city_pe_0509,"Belén","050902"
district_pe_050903,city_pe_0509,"Chalcos","050903"
district_pe_050904,city_pe_0509,"Chilcayoc","050904"
district_pe_050905,city_pe_0509,"Huacaña","050905"
district_pe_050906,city_pe_0509,"Morcolla","050906"
district_pe_050907,city_pe_0509,"Paico","050907"
district_pe_050908,city_pe_0509,"San Pedro de Larcay","050908"
district_pe_050909,city_pe_0509,"San Salvador de Quije","050909"
district_pe_050910,city_pe_0509,"Santiago de Paucaray","050910"
district_pe_050911,city_pe_0509,"Soras","050911"
district_pe_051001,city_pe_0510,"Huancapi","051001"
district_pe_051002,city_pe_0510,"Alcamenca","051002"
district_pe_051003,city_pe_0510,"Apongo","051003"
district_pe_051004,city_pe_0510,"Asquipata","051004"
district_pe_051005,city_pe_0510,"Canaria","051005"
district_pe_051006,city_pe_0510,"Cayara","051006"
district_pe_051007,city_pe_0510,"Colca","051007"
district_pe_051008,city_pe_0510,"Huamanquiquia","051008"
district_pe_051009,city_pe_0510,"Huancaraylla","051009"
district_pe_051010,city_pe_0510,"Huaya","051010"
district_pe_051011,city_pe_0510,"Sarhua","051011"
district_pe_051012,city_pe_0510,"Vilcanchos","051012"
district_pe_051101,city_pe_0511,"Vilcas Huaman","051101"
district_pe_051102,city_pe_0511,"Accomarca","051102"
district_pe_051103,city_pe_0511,"Carhuanca","051103"
district_pe_051104,city_pe_0511,"Concepción","051104"
district_pe_051105,city_pe_0511,"Huambalpa","051105"
district_pe_051106,city_pe_0511,"Independencia","051106"
district_pe_051107,city_pe_0511,"Saurama","051107"
district_pe_051108,city_pe_0511,"Vischongo","051108"
district_pe_060101,city_pe_0601,"Cajamarca","060101"
district_pe_060102,city_pe_0601,"Asunción","060102"
district_pe_060103,city_pe_0601,"Chetilla","060103"
district_pe_060104,city_pe_0601,"Cospan","060104"
district_pe_060105,city_pe_0601,"Encañada","060105"
district_pe_060106,city_pe_0601,"Jesús","060106"
district_pe_060107,city_pe_0601,"Llacanora","060107"
district_pe_060108,city_pe_0601,"Los Baños del Inca","060108"
district_pe_060109,city_pe_0601,"Magdalena","060109"
district_pe_060110,city_pe_0601,"Matara","060110"
district_pe_060111,city_pe_0601,"Namora","060111"
district_pe_060112,city_pe_0601,"San Juan","060112"
district_pe_060201,city_pe_0602,"Cajabamba","060201"
district_pe_060202,city_pe_0602,"Cachachi","060202"
district_pe_060203,city_pe_0602,"Condebamba","060203"
district_pe_060204,city_pe_0602,"Sitacocha","060204"
district_pe_060301,city_pe_0603,"Celendín","060301"
district_pe_060302,city_pe_0603,"Chumuch","060302"
district_pe_060303,city_pe_0603,"Cortegana","060303"
district_pe_060304,city_pe_0603,"Huasmin","060304"
district_pe_060305,city_pe_0603,"Jorge Chávez","060305"
district_pe_060306,city_pe_0603,"José Gálvez","060306"
district_pe_060307,city_pe_0603,"Miguel Iglesias","060307"
district_pe_060308,city_pe_0603,"Oxamarca","060308"
district_pe_060309,city_pe_0603,"Sorochuco","060309"
district_pe_060310,city_pe_0603,"Sucre","060310"
district_pe_060311,city_pe_0603,"Utco","060311"
district_pe_060312,city_pe_0603,"La Libertad de Pallan","060312"
district_pe_060401,city_pe_0604,"Chota","060401"
district_pe_060402,city_pe_0604,"Anguia","060402"
district_pe_060403,city_pe_0604,"Chadin","060403"
district_pe_060404,city_pe_0604,"Chiguirip","060404"
district_pe_060405,city_pe_0604,"Chimban","060405"
district_pe_060406,city_pe_0604,"Choropampa","060406"
district_pe_060407,city_pe_0604,"Cochabamba","060407"
district_pe_060408,city_pe_0604,"Conchan","060408"
district_pe_060409,city_pe_0604,"Huambos","060409"
district_pe_060410,city_pe_0604,"Lajas","060410"
district_pe_060411,city_pe_0604,"Llama","060411"
district_pe_060412,city_pe_0604,"Miracosta","060412"
district_pe_060413,city_pe_0604,"Paccha","060413"
district_pe_060414,city_pe_0604,"Pion","060414"
district_pe_060415,city_pe_0604,"querocoto","060415"
district_pe_060416,city_pe_0604,"San Juan de Licupis","060416"
district_pe_060417,city_pe_0604,"Tacabamba","060417"
district_pe_060418,city_pe_0604,"Tocmoche","060418"
district_pe_060419,city_pe_0604,"Chalamarca","060419"
district_pe_060501,city_pe_0605,"Contumaza","060501"
district_pe_060502,city_pe_0605,"Chilete","060502"
district_pe_060503,city_pe_0605,"Cupisnique","060503"
district_pe_060504,city_pe_0605,"Guzmángo","060504"
district_pe_060505,city_pe_0605,"San Benito","060505"
district_pe_060506,city_pe_0605,"Santa Cruz de Toledo","060506"
district_pe_060507,city_pe_0605,"Tantarica","060507"
district_pe_060508,city_pe_0605,"Yonan","060508"
district_pe_060601,city_pe_0606,"Cutervo","060601"
district_pe_060602,city_pe_0606,"Callayuc","060602"
district_pe_060603,city_pe_0606,"Choros","060603"
district_pe_060604,city_pe_0606,"Cujillo","060604"
district_pe_060605,city_pe_0606,"La Ramada","060605"
district_pe_060606,city_pe_0606,"Pimpingos","060606"
district_pe_060607,city_pe_0606,"Querocotillo","060607"
district_pe_060608,city_pe_0606,"San Andrés de Cutervo","060608"
district_pe_060609,city_pe_0606,"San Juan de Cutervo","060609"
district_pe_060610,city_pe_0606,"San Luis de Lucma","060610"
district_pe_060611,city_pe_0606,"Santa Cruz","060611"
district_pe_060612,city_pe_0606,"Santo Domingo de la Capilla","060612"
district_pe_060613,city_pe_0606,"Santo Tomas","060613"
district_pe_060614,city_pe_0606,"Socota","060614"
district_pe_060615,city_pe_0606,"Toribio Casanova","060615"
district_pe_060701,city_pe_0607,"Bambamarca","060701"
district_pe_060702,city_pe_0607,"Chugur","060702"
district_pe_060703,city_pe_0607,"Hualgayoc","060703"
district_pe_060801,city_pe_0608,"Jaén","060801"
district_pe_060802,city_pe_0608,"Bellavista","060802"
district_pe_060803,city_pe_0608,"Chontali","060803"
district_pe_060804,city_pe_0608,"Colasay","060804"
district_pe_060805,city_pe_0608,"Huabal","060805"
district_pe_060806,city_pe_0608,"Las Pirias","060806"
district_pe_060807,city_pe_0608,"Pomahuaca","060807"
district_pe_060808,city_pe_0608,"Pucara","060808"
district_pe_060809,city_pe_0608,"Sallique","060809"
district_pe_060810,city_pe_0608,"San Felipe","060810"
district_pe_060811,city_pe_0608,"San José del Alto","060811"
district_pe_060812,city_pe_0608,"Santa Rosa","060812"
district_pe_060901,city_pe_0609,"San Ignacio","060901"
district_pe_060902,city_pe_0609,"Chirinos","060902"
district_pe_060903,city_pe_0609,"Huarango","060903"
district_pe_060904,city_pe_0609,"La coipa","060904"
district_pe_060905,city_pe_0609,"Namballe","060905"
district_pe_060906,city_pe_0609,"San José de Lourdes","060906"
district_pe_060907,city_pe_0609,"Tabaconas","060907"
district_pe_061001,city_pe_0610,"Pedro Gálvez","061001"
district_pe_061002,city_pe_0610,"Chancay","061002"
district_pe_061003,city_pe_0610,"Eduardo Villanueva","061003"
district_pe_061004,city_pe_0610,"Gregorio Pita","061004"
district_pe_061005,city_pe_0610,"Ichocan","061005"
district_pe_061006,city_pe_0610,"José Manuel Quiroz","061006"
district_pe_061007,city_pe_0610,"José Sabogal","061007"
district_pe_061101,city_pe_0611,"San Miguel","061101"
district_pe_061102,city_pe_0611,"Bolívar","061102"
district_pe_061103,city_pe_0611,"Calquis","061103"
district_pe_061104,city_pe_0611,"Catilluc","061104"
district_pe_061105,city_pe_0611,"El Prado","061105"
district_pe_061106,city_pe_0611,"La Florida","061106"
district_pe_061107,city_pe_0611,"Llapa","061107"
district_pe_061108,city_pe_0611,"Nanchoc","061108"
district_pe_061109,city_pe_0611,"Niepos","061109"
district_pe_061110,city_pe_0611,"San Gregorio","061110"
district_pe_061111,city_pe_0611,"San Silvestre de Cochan","061111"
district_pe_061112,city_pe_0611,"Tongod","061112"
district_pe_061113,city_pe_0611,"Unión Agua Blanca","061113"
district_pe_061201,city_pe_0612,"San Pablo","061201"
district_pe_061202,city_pe_0612,"San Bernardino","061202"
district_pe_061203,city_pe_0612,"San Luis","061203"
district_pe_061204,city_pe_0612,"Tumbaden","061204"
district_pe_061301,city_pe_0613,"Santa Cruz","061301"
district_pe_061302,city_pe_0613,"Andabamba","061302"
district_pe_061303,city_pe_0613,"Catache","061303"
district_pe_061304,city_pe_0613,"Chancaybaños","061304"
district_pe_061305,city_pe_0613,"La Esperanza","061305"
district_pe_061306,city_pe_0613,"Ninabamba","061306"
district_pe_061307,city_pe_0613,"Pulan","061307"
district_pe_061308,city_pe_0613,"Saucepampa","061308"
district_pe_061309,city_pe_0613,"Sexi","061309"
district_pe_061310,city_pe_0613,"Uticyacu","061310"
district_pe_061311,city_pe_0613,"Yauyucan","061311"
district_pe_070101,city_pe_0701,"Callao","070101"
district_pe_070102,city_pe_0701,"Bellavista","070102"
district_pe_070103,city_pe_0701,"Carmen de la Legua Reynoso","070103"
district_pe_070104,city_pe_0701,"La Perla","070104"
district_pe_070105,city_pe_0701,"La Punta","070105"
district_pe_070106,city_pe_0701,"Ventanilla","070106"
district_pe_070107,city_pe_0701,"Mi Perú","070107"
district_pe_080101,city_pe_0801,"Cusco","080101"
district_pe_080102,city_pe_0801,"Ccorca","080102"
district_pe_080103,city_pe_0801,"Poroy","080103"
district_pe_080104,city_pe_0801,"San Jerónimo","080104"
district_pe_080105,city_pe_0801,"San Sebastian","080105"
district_pe_080106,city_pe_0801,"Santiago","080106"
district_pe_080107,city_pe_0801,"Saylla","080107"
district_pe_080108,city_pe_0801,"Wanchaq","080108"
district_pe_080201,city_pe_0802,"Acomayo","080201"
district_pe_080202,city_pe_0802,"Acopia","080202"
district_pe_080203,city_pe_0802,"Acos","080203"
district_pe_080204,city_pe_0802,"Mosoc Llacta","080204"
district_pe_080205,city_pe_0802,"Pomacanchi","080205"
district_pe_080206,city_pe_0802,"Rondocan","080206"
district_pe_080207,city_pe_0802,"Sangarara","080207"
district_pe_080301,city_pe_0803,"Anta","080301"
district_pe_080302,city_pe_0803,"Ancahuasi","080302"
district_pe_080303,city_pe_0803,"Cachimayo","080303"
district_pe_080304,city_pe_0803,"Chinchaypujio","080304"
district_pe_080305,city_pe_0803,"Huarocondo","080305"
district_pe_080306,city_pe_0803,"Limatambo","080306"
district_pe_080307,city_pe_0803,"Mollepata","080307"
district_pe_080308,city_pe_0803,"Pucyura","080308"
district_pe_080309,city_pe_0803,"Zurite","080309"
district_pe_080401,city_pe_0804,"Calca","080401"
district_pe_080402,city_pe_0804,"Coya","080402"
district_pe_080403,city_pe_0804,"Lamay","080403"
district_pe_080404,city_pe_0804,"Lares","080404"
district_pe_080405,city_pe_0804,"Pisac","080405"
district_pe_080406,city_pe_0804,"San Salvador","080406"
district_pe_080407,city_pe_0804,"Taray","080407"
district_pe_080408,city_pe_0804,"Yanatile","080408"
district_pe_080501,city_pe_0805,"Yanaoca","080501"
district_pe_080502,city_pe_0805,"Checca","080502"
district_pe_080503,city_pe_0805,"Kunturkanki","080503"
district_pe_080504,city_pe_0805,"Langui","080504"
district_pe_080505,city_pe_0805,"Layo","080505"
district_pe_080506,city_pe_0805,"Pampamarca","080506"
district_pe_080507,city_pe_0805,"Quehue","080507"
district_pe_080508,city_pe_0805,"Tupac Amaru","080508"
district_pe_080601,city_pe_0806,"Sicuani","080601"
district_pe_080602,city_pe_0806,"Checacupe","080602"
district_pe_080603,city_pe_0806,"Combapata","080603"
district_pe_080604,city_pe_0806,"Marangani","080604"
district_pe_080605,city_pe_0806,"Pitumarca","080605"
district_pe_080606,city_pe_0806,"San Pablo","080606"
district_pe_080607,city_pe_0806,"San Pedro","080607"
district_pe_080608,city_pe_0806,"Tinta","080608"
district_pe_080701,city_pe_0807,"Santo Tomas","080701"
district_pe_080702,city_pe_0807,"Capacmarca","080702"
district_pe_080703,city_pe_0807,"Chamaca","080703"
district_pe_080704,city_pe_0807,"Colquemarca","080704"
district_pe_080705,city_pe_0807,"Livitaca","080705"
district_pe_080706,city_pe_0807,"Llusco","080706"
district_pe_080707,city_pe_0807,"Quiñota","080707"
district_pe_080708,city_pe_0807,"Velille","080708"
district_pe_080801,city_pe_0808,"Espinar","080801"
district_pe_080802,city_pe_0808,"Condoroma","080802"
district_pe_080803,city_pe_0808,"Coporaque","080803"
district_pe_080804,city_pe_0808,"Ocoruro","080804"
district_pe_080805,city_pe_0808,"Pallpata","080805"
district_pe_080806,city_pe_0808,"Pichigua","080806"
district_pe_080807,city_pe_0808,"Suyckutambo","080807"
district_pe_080808,city_pe_0808,"Alto Pichigua","080808"
district_pe_080901,city_pe_0809,"Santa Ana","080901"
district_pe_080902,city_pe_0809,"Echarate","080902"
district_pe_080903,city_pe_0809,"Huayopata","080903"
district_pe_080904,city_pe_0809,"Maranura","080904"
district_pe_080905,city_pe_0809,"Ocobamba","080905"
district_pe_080906,city_pe_0809,"Quellouno","080906"
district_pe_080907,city_pe_0809,"Kimbiri","080907"
district_pe_080908,city_pe_0809,"Santa Teresa","080908"
district_pe_080909,city_pe_0809,"Vilcabamba","080909"
district_pe_080910,city_pe_0809,"Pichari","080910"
district_pe_080911,city_pe_0809,"Inkawasi","080911"
district_pe_080912,city_pe_0809,"Villa Virgen","080912"
district_pe_080913,city_pe_0809,"Villa Kintiarina","080913"
district_pe_080914,city_pe_0809,"Megantoni","080914"
district_pe_081001,city_pe_0810,"Paruro","081001"
district_pe_081002,city_pe_0810,"Accha","081002"
district_pe_081003,city_pe_0810,"Ccapi","081003"
district_pe_081004,city_pe_0810,"Colcha","081004"
district_pe_081005,city_pe_0810,"Huanoquite","081005"
district_pe_081006,city_pe_0810,"Omacha","081006"
district_pe_081007,city_pe_0810,"Paccaritambo","081007"
district_pe_081008,city_pe_0810,"Pillpinto","081008"
district_pe_081009,city_pe_0810,"Yaurisque","081009"
district_pe_081101,city_pe_0811,"Paucartambo","081101"
district_pe_081102,city_pe_0811,"Caicay","081102"
district_pe_081103,city_pe_0811,"Challabamba","081103"
district_pe_081104,city_pe_0811,"Colquepata","081104"
district_pe_081105,city_pe_0811,"Huancarani","081105"
district_pe_081106,city_pe_0811,"Kosñipata","081106"
district_pe_081201,city_pe_0812,"Urcos","081201"
district_pe_081202,city_pe_0812,"Andahuaylillas","081202"
district_pe_081203,city_pe_0812,"Camanti","081203"
district_pe_081204,city_pe_0812,"Ccarhuayo","081204"
district_pe_081205,city_pe_0812,"Ccatca","081205"
district_pe_081206,city_pe_0812,"Cusipata","081206"
district_pe_081207,city_pe_0812,"Huaro","081207"
district_pe_081208,city_pe_0812,"Lucre","081208"
district_pe_081209,city_pe_0812,"Marcapata","081209"
district_pe_081210,city_pe_0812,"Ocongate","081210"
district_pe_081211,city_pe_0812,"Oropesa","081211"
district_pe_081212,city_pe_0812,"Quiquijana","081212"
district_pe_081301,city_pe_0813,"Urubamba","081301"
district_pe_081302,city_pe_0813,"Chinchero","081302"
district_pe_081303,city_pe_0813,"Huayllabamba","081303"
district_pe_081304,city_pe_0813,"Machupicchu","081304"
district_pe_081305,city_pe_0813,"Maras","081305"
district_pe_081306,city_pe_0813,"Ollantaytambo","081306"
district_pe_081307,city_pe_0813,"Yucay","081307"
district_pe_090101,city_pe_0901,"Huancavelica","090101"
district_pe_090102,city_pe_0901,"Acobambilla","090102"
district_pe_090103,city_pe_0901,"Acoria","090103"
district_pe_090104,city_pe_0901,"Conayca","090104"
district_pe_090105,city_pe_0901,"Cuenca","090105"
district_pe_090106,city_pe_0901,"Huachocolpa","090106"
district_pe_090107,city_pe_0901,"Huayllahuara","090107"
district_pe_090108,city_pe_0901,"Izcuchaca","090108"
district_pe_090109,city_pe_0901,"Laria","090109"
district_pe_090110,city_pe_0901,"Manta","090110"
district_pe_090111,city_pe_0901,"Mariscal Cáceres","090111"
district_pe_090112,city_pe_0901,"Moya","090112"
district_pe_090113,city_pe_0901,"Nuevo Occoro","090113"
district_pe_090114,city_pe_0901,"Palca","090114"
district_pe_090115,city_pe_0901,"Pilchaca","090115"
district_pe_090116,city_pe_0901,"Vilca","090116"
district_pe_090117,city_pe_0901,"Yauli","090117"
district_pe_090118,city_pe_0901,"Ascensión","090118"
district_pe_090119,city_pe_0901,"Huando","090119"
district_pe_090201,city_pe_0902,"Acobamba","090201"
district_pe_090202,city_pe_0902,"Andabamba","090202"
district_pe_090203,city_pe_0902,"Anta","090203"
district_pe_090204,city_pe_0902,"Caja","090204"
district_pe_090205,city_pe_0902,"Marcas","090205"
district_pe_090206,city_pe_0902,"Paucara","090206"
district_pe_090207,city_pe_0902,"Pomacocha","090207"
district_pe_090208,city_pe_0902,"Rosario","090208"
district_pe_090301,city_pe_0903,"Lircay","090301"
district_pe_090302,city_pe_0903,"Anchonga","090302"
district_pe_090303,city_pe_0903,"Callanmarca","090303"
district_pe_090304,city_pe_0903,"Ccochaccasa","090304"
district_pe_090305,city_pe_0903,"Chincho","090305"
district_pe_090306,city_pe_0903,"Congalla","090306"
district_pe_090307,city_pe_0903,"Huanca-Huanca","090307"
district_pe_090308,city_pe_0903,"Huayllay Grande","090308"
district_pe_090309,city_pe_0903,"Julcamarca","090309"
district_pe_090310,city_pe_0903,"San Antonio de Antaparco","090310"
district_pe_090311,city_pe_0903,"Santo Tomas de Pata","090311"
district_pe_090312,city_pe_0903,"Secclla","090312"
district_pe_090401,city_pe_0904,"Castrovirreyna","090401"
district_pe_090402,city_pe_0904,"Arma","090402"
district_pe_090403,city_pe_0904,"Aurahua","090403"
district_pe_090404,city_pe_0904,"Capillas","090404"
district_pe_090405,city_pe_0904,"Chupamarca","090405"
district_pe_090406,city_pe_0904,"Cocas","090406"
district_pe_090407,city_pe_0904,"Huachos","090407"
district_pe_090408,city_pe_0904,"Huamatambo","090408"
district_pe_090409,city_pe_0904,"Mollepampa","090409"
district_pe_090410,city_pe_0904,"San Juan","090410"
district_pe_090411,city_pe_0904,"Santa Ana","090411"
district_pe_090412,city_pe_0904,"Tantara","090412"
district_pe_090413,city_pe_0904,"Ticrapo","090413"
district_pe_090501,city_pe_0905,"Churcampa","090501"
district_pe_090502,city_pe_0905,"Anco","090502"
district_pe_090503,city_pe_0905,"Chinchihuasi","090503"
district_pe_090504,city_pe_0905,"El Carmen","090504"
district_pe_090505,city_pe_0905,"La Merced","090505"
district_pe_090506,city_pe_0905,"Locroja","090506"
district_pe_090507,city_pe_0905,"Paucarbamba","090507"
district_pe_090508,city_pe_0905,"San Miguel de Mayocc","090508"
district_pe_090509,city_pe_0905,"San Pedro de Coris","090509"
district_pe_090510,city_pe_0905,"Pachamarca","090510"
district_pe_090511,city_pe_0905,"Cosme","090511"
district_pe_090601,city_pe_0906,"Huaytara","090601"
district_pe_090602,city_pe_0906,"Ayavi","090602"
district_pe_090603,city_pe_0906,"Córdova","090603"
district_pe_090604,city_pe_0906,"Huayacundo Arma","090604"
district_pe_090605,city_pe_0906,"Laramarca","090605"
district_pe_090606,city_pe_0906,"Ocoyo","090606"
district_pe_090607,city_pe_0906,"Pilpichaca","090607"
district_pe_090608,city_pe_0906,"Querco","090608"
district_pe_090609,city_pe_0906,"Quito-Arma","090609"
district_pe_090610,city_pe_0906,"San Antonio de Cusicancha","090610"
district_pe_090611,city_pe_0906,"San Francisco de Sangayaico","090611"
district_pe_090612,city_pe_0906,"San Isidro","090612"
district_pe_090613,city_pe_0906,"Santiago de Chocorvos","090613"
district_pe_090614,city_pe_0906,"Santiago de Quirahuara","090614"
district_pe_090615,city_pe_0906,"Santo Domingo de Capillas","090615"
district_pe_090616,city_pe_0906,"Tambo","090616"
district_pe_090701,city_pe_0907,"Pampas","090701"
district_pe_090702,city_pe_0907,"Acostambo","090702"
district_pe_090703,city_pe_0907,"Acraquia","090703"
district_pe_090704,city_pe_0907,"Ahuaycha","090704"
district_pe_090705,city_pe_0907,"Colcabamba","090705"
district_pe_090706,city_pe_0907,"Daniel Hernández","090706"
district_pe_090707,city_pe_0907,"Huachocolpa","090707"
district_pe_090709,city_pe_0907,"Huaribamba","090709"
district_pe_090710,city_pe_0907,"Ñahuimpuquio","090710"
district_pe_090711,city_pe_0907,"Pazos","090711"
district_pe_090713,city_pe_0907,"Quishuar","090713"
district_pe_090714,city_pe_0907,"Salcabamba","090714"
district_pe_090715,city_pe_0907,"Salcahuasi","090715"
district_pe_090716,city_pe_0907,"San Marcos de Rocchac","090716"
district_pe_090717,city_pe_0907,"Surcubamba","090717"
district_pe_090718,city_pe_0907,"Tintay Puncu","090718"
district_pe_090719,city_pe_0907,"Quichuas","090719"
district_pe_090720,city_pe_0907,"Andaymarca","090720"
district_pe_090721,city_pe_0907,"Roble","090721"
district_pe_090722,city_pe_0907,"Pichos","090722"
district_pe_090723,city_pe_0907,"Santiago de Tucuma","090723"
district_pe_100101,city_pe_1001,"Huanuco","100101"
district_pe_100102,city_pe_1001,"Amarilis","100102"
district_pe_100103,city_pe_1001,"Chinchao","100103"
district_pe_100104,city_pe_1001,"Churubamba","100104"
district_pe_100105,city_pe_1001,"Margos","100105"
district_pe_100106,city_pe_1001,"Quisqui (Kichki)","100106"
district_pe_100107,city_pe_1001,"San Francisco de Cayran","100107"
district_pe_100108,city_pe_1001,"San Pedro de Chaulan","100108"
district_pe_100109,city_pe_1001,"Santa María del Valle","100109"
district_pe_100110,city_pe_1001,"Yarumayo","100110"
district_pe_100111,city_pe_1001,"Pillco Marca","100111"
district_pe_100112,city_pe_1001,"Yacus","100112"
district_pe_100113,city_pe_1001,"San Pablo de Pillao","100113"
district_pe_100201,city_pe_1002,"Ambo","100201"
district_pe_100202,city_pe_1002,"Cayna","100202"
district_pe_100203,city_pe_1002,"Colpas","100203"
district_pe_100204,city_pe_1002,"Conchamarca","100204"
district_pe_100205,city_pe_1002,"Huacar","100205"
district_pe_100206,city_pe_1002,"San Francisco","100206"
district_pe_100207,city_pe_1002,"San Rafael","100207"
district_pe_100208,city_pe_1002,"Tomay Kichwa","100208"
district_pe_100301,city_pe_1003,"La Unión","100301"
district_pe_100307,city_pe_1003,"Chuquis","100307"
district_pe_100311,city_pe_1003,"Marías","100311"
district_pe_100313,city_pe_1003,"Pachas","100313"
district_pe_100316,city_pe_1003,"Quivilla","100316"
district_pe_100317,city_pe_1003,"Ripan","100317"
district_pe_100321,city_pe_1003,"Shunqui","100321"
district_pe_100322,city_pe_1003,"Sillapata","100322"
district_pe_100323,city_pe_1003,"Yanas","100323"
district_pe_100401,city_pe_1004,"Huacaybamba","100401"
district_pe_100402,city_pe_1004,"Canchabamba","100402"
district_pe_100403,city_pe_1004,"Cochabamba","100403"
district_pe_100404,city_pe_1004,"Pinra","100404"
district_pe_100501,city_pe_1005,"Llata","100501"
district_pe_100502,city_pe_1005,"Arancay","100502"
district_pe_100503,city_pe_1005,"Chavín de Pariarca","100503"
district_pe_100504,city_pe_1005,"Jacas Grande","100504"
district_pe_100505,city_pe_1005,"Jircan","100505"
district_pe_100506,city_pe_1005,"Miraflores","100506"
district_pe_100507,city_pe_1005,"Monzón","100507"
district_pe_100508,city_pe_1005,"Punchao","100508"
district_pe_100509,city_pe_1005,"Puños","100509"
district_pe_100510,city_pe_1005,"Singa","100510"
district_pe_100511,city_pe_1005,"Tantamayo","100511"
district_pe_100601,city_pe_1006,"Rupa-Rupa","100601"
district_pe_100602,city_pe_1006,"Daniel Alomía Robles","100602"
district_pe_100603,city_pe_1006,"Hermílio Valdizan","100603"
district_pe_100604,city_pe_1006,"José Crespo y Castillo","100604"
district_pe_100605,city_pe_1006,"Luyando","100605"
district_pe_100606,city_pe_1006,"Mariano Damaso Beraun","100606"
district_pe_100607,city_pe_1006,"Pucayacu","100607"
district_pe_100608,city_pe_1006,"Castillo Grande","100608"
district_pe_100609,city_pe_1006,"Pueblo Nuevo","100609"
district_pe_100610,city_pe_1006,"Santo Domingo de Anda","100610"
district_pe_100701,city_pe_1007,"Huacrachuco","100701"
district_pe_100702,city_pe_1007,"Cholon","100702"
district_pe_100703,city_pe_1007,"San Buenaventura","100703"
district_pe_100704,city_pe_1007,"La Morada","100704"
district_pe_100705,city_pe_1007,"Santa Rosa de Alto Yanajanca","100705"
district_pe_100801,city_pe_1008,"Panao","100801"
district_pe_100802,city_pe_1008,"Chaglla","100802"
district_pe_100803,city_pe_1008,"Molino","100803"
district_pe_100804,city_pe_1008,"Umari","100804"
district_pe_100901,city_pe_1009,"Puerto Inca","100901"
district_pe_100902,city_pe_1009,"Codo del Pozuzo","100902"
district_pe_100903,city_pe_1009,"Honoria","100903"
district_pe_100904,city_pe_1009,"Tournavista","100904"
district_pe_100905,city_pe_1009,"Yuyapichis","100905"
district_pe_101001,city_pe_1010,"Jesús","101001"
district_pe_101002,city_pe_1010,"Baños","101002"
district_pe_101003,city_pe_1010,"Jivia","101003"
district_pe_101004,city_pe_1010,"Queropalca","101004"
district_pe_101005,city_pe_1010,"Rondos","101005"
district_pe_101006,city_pe_1010,"San Francisco de Asís","101006"
district_pe_101007,city_pe_1010,"San Miguel de Cauri","101007"
district_pe_101101,city_pe_1011,"Chavínillo","101101"
district_pe_101102,city_pe_1011,"Cahuac","101102"
district_pe_101103,city_pe_1011,"Chacabamba","101103"
district_pe_101104,city_pe_1011,"Aparicio Pomares","101104"
district_pe_101105,city_pe_1011,"Jacas Chico","101105"
district_pe_101106,city_pe_1011,"Obas","101106"
district_pe_101107,city_pe_1011,"Pampamarca","101107"
district_pe_101108,city_pe_1011,"Choras","101108"
district_pe_110101,city_pe_1101,"Ica","110101"
district_pe_110102,city_pe_1101,"La Tinguiña","110102"
district_pe_110103,city_pe_1101,"Los Aquijes","110103"
district_pe_110104,city_pe_1101,"Ocucaje","110104"
district_pe_110105,city_pe_1101,"Pachacutec","110105"
district_pe_110106,city_pe_1101,"Parcona","110106"
district_pe_110107,city_pe_1101,"Pueblo Nuevo","110107"
district_pe_110108,city_pe_1101,"Salas","110108"
district_pe_110109,city_pe_1101,"San José de Los Molinos","110109"
district_pe_110110,city_pe_1101,"San Juan Bautista","110110"
district_pe_110111,city_pe_1101,"Santiago","110111"
district_pe_110112,city_pe_1101,"Subtanjalla","110112"
district_pe_110113,city_pe_1101,"Tate","110113"
district_pe_110114,city_pe_1101,"Yauca del Rosario","110114"
district_pe_110201,city_pe_1102,"Chincha Alta","110201"
district_pe_110202,city_pe_1102,"Alto Laran","110202"
district_pe_110203,city_pe_1102,"Chavín","110203"
district_pe_110204,city_pe_1102,"Chincha Baja","110204"
district_pe_110205,city_pe_1102,"El Carmen","110205"
district_pe_110206,city_pe_1102,"Grocio Prado","110206"
district_pe_110207,city_pe_1102,"Pueblo Nuevo","110207"
district_pe_110208,city_pe_1102,"San Juan de Yanac","110208"
district_pe_110209,city_pe_1102,"San Pedro de Huacarpana","110209"
district_pe_110210,city_pe_1102,"Sunampe","110210"
district_pe_110211,city_pe_1102,"Tambo de Mora","110211"
district_pe_110301,city_pe_1103,"Nazca","110301"
district_pe_110302,city_pe_1103,"Changuillo","110302"
district_pe_110303,city_pe_1103,"El Ingenio","110303"
district_pe_110304,city_pe_1103,"Marcona","110304"
district_pe_110305,city_pe_1103,"Vista Alegre","110305"
district_pe_110401,city_pe_1104,"Palpa","110401"
district_pe_110402,city_pe_1104,"Llipata","110402"
district_pe_110403,city_pe_1104,"Río Grande","110403"
district_pe_110404,city_pe_1104,"Santa Cruz","110404"
district_pe_110405,city_pe_1104,"Tibillo","110405"
district_pe_110501,city_pe_1105,"Pisco","110501"
district_pe_110502,city_pe_1105,"Huancano","110502"
district_pe_110503,city_pe_1105,"Humay","110503"
district_pe_110504,city_pe_1105,"Independencia","110504"
district_pe_110505,city_pe_1105,"Paracas","110505"
district_pe_110506,city_pe_1105,"San Andrés","110506"
district_pe_110507,city_pe_1105,"San Clemente","110507"
district_pe_110508,city_pe_1105,"Tupac Amaru Inca","110508"
district_pe_120101,city_pe_1201,"Huancayo","120101"
district_pe_120104,city_pe_1201,"Carhuacallanga","120104"
district_pe_120105,city_pe_1201,"Chacapampa","120105"
district_pe_120106,city_pe_1201,"Chicche","120106"
district_pe_120107,city_pe_1201,"Chilca","120107"
district_pe_120108,city_pe_1201,"Chongos Alto","120108"
district_pe_120111,city_pe_1201,"Chupuro","120111"
district_pe_120112,city_pe_1201,"Colca","120112"
district_pe_120113,city_pe_1201,"Cullhuas","120113"
district_pe_120114,city_pe_1201,"El Tambo","120114"
district_pe_120116,city_pe_1201,"Huacrapuquio","120116"
district_pe_120117,city_pe_1201,"Hualhuas","120117"
district_pe_120119,city_pe_1201,"Huancan","120119"
district_pe_120120,city_pe_1201,"Huasicancha","120120"
district_pe_120121,city_pe_1201,"Huayucachi","120121"
district_pe_120122,city_pe_1201,"Ingenio","120122"
district_pe_120124,city_pe_1201,"Pariahuanca","120124"
district_pe_120125,city_pe_1201,"Pilcomayo","120125"
district_pe_120126,city_pe_1201,"Pucara","120126"
district_pe_120127,city_pe_1201,"Quichuay","120127"
district_pe_120128,city_pe_1201,"Quilcas","120128"
district_pe_120129,city_pe_1201,"San Agustín","120129"
district_pe_120130,city_pe_1201,"San Jerónimo de Tunan","120130"
district_pe_120132,city_pe_1201,"Saño","120132"
district_pe_120133,city_pe_1201,"Sapallanga","120133"
district_pe_120134,city_pe_1201,"Sicaya","120134"
district_pe_120135,city_pe_1201,"Santo Domingo de Acobamba","120135"
district_pe_120136,city_pe_1201,"Viques","120136"
district_pe_120201,city_pe_1202,"Concepción","120201"
district_pe_120202,city_pe_1202,"Aco","120202"
district_pe_120203,city_pe_1202,"Andamarca","120203"
district_pe_120204,city_pe_1202,"Chambara","120204"
district_pe_120205,city_pe_1202,"Cochas","120205"
district_pe_120206,city_pe_1202,"Comas","120206"
district_pe_120207,city_pe_1202,"Heroínas Toledo","120207"
district_pe_120208,city_pe_1202,"Manzanares","120208"
district_pe_120209,city_pe_1202,"Mariscal Castilla","120209"
district_pe_120210,city_pe_1202,"Matahuasi","120210"
district_pe_120211,city_pe_1202,"Mito","120211"
district_pe_120212,city_pe_1202,"Nueve de Julio","120212"
district_pe_120213,city_pe_1202,"Orcotuna","120213"
district_pe_120214,city_pe_1202,"San José de Quero","120214"
district_pe_120215,city_pe_1202,"Santa Rosa de Ocopa","120215"
district_pe_120301,city_pe_1203,"Chanchamayo","120301"
district_pe_120302,city_pe_1203,"Perene","120302"
district_pe_120303,city_pe_1203,"Pichanaqui","120303"
district_pe_120304,city_pe_1203,"San Luis de Shuaro","120304"
district_pe_120305,city_pe_1203,"San Ramón","120305"
district_pe_120306,city_pe_1203,"Vitoc","120306"
district_pe_120401,city_pe_1204,"Jauja","120401"
district_pe_120402,city_pe_1204,"Acolla","120402"
district_pe_120403,city_pe_1204,"Apata","120403"
district_pe_120404,city_pe_1204,"Ataura","120404"
district_pe_120405,city_pe_1204,"Canchayllo","120405"
district_pe_120406,city_pe_1204,"Curicaca","120406"
district_pe_120407,city_pe_1204,"El Mantaro","120407"
district_pe_120408,city_pe_1204,"Huamali","120408"
district_pe_120409,city_pe_1204,"Huaripampa","120409"
district_pe_120410,city_pe_1204,"Huertas","120410"
district_pe_120411,city_pe_1204,"Janjaillo","120411"
district_pe_120412,city_pe_1204,"Julcán","120412"
district_pe_120413,city_pe_1204,"Leonor Ordoñez","120413"
district_pe_120414,city_pe_1204,"Llocllapampa","120414"
district_pe_120415,city_pe_1204,"Marco","120415"
district_pe_120416,city_pe_1204,"Masma","120416"
district_pe_120417,city_pe_1204,"Masma Chicche","120417"
district_pe_120418,city_pe_1204,"Molinos","120418"
district_pe_120419,city_pe_1204,"Monobamba","120419"
district_pe_120420,city_pe_1204,"Muqui","120420"
district_pe_120421,city_pe_1204,"Muquiyauyo","120421"
district_pe_120422,city_pe_1204,"Paca","120422"
district_pe_120423,city_pe_1204,"Paccha","120423"
district_pe_120424,city_pe_1204,"Pancan","120424"
district_pe_120425,city_pe_1204,"Parco","120425"
district_pe_120426,city_pe_1204,"Pomacancha","120426"
district_pe_120427,city_pe_1204,"Ricran","120427"
district_pe_120428,city_pe_1204,"San Lorenzo","120428"
district_pe_120429,city_pe_1204,"San Pedro de Chunan","120429"
district_pe_120430,city_pe_1204,"Sausa","120430"
district_pe_120431,city_pe_1204,"Sincos","120431"
district_pe_120432,city_pe_1204,"Tunan Marca","120432"
district_pe_120433,city_pe_1204,"Yauli","120433"
district_pe_120434,city_pe_1204,"Yauyos","120434"
district_pe_120501,city_pe_1205,"Junin","120501"
district_pe_120502,city_pe_1205,"Carhuamayo","120502"
district_pe_120503,city_pe_1205,"Ondores","120503"
district_pe_120504,city_pe_1205,"Ulcumayo","120504"
district_pe_120601,city_pe_1206,"Satipo","120601"
district_pe_120602,city_pe_1206,"Coviriali","120602"
district_pe_120603,city_pe_1206,"Llaylla","120603"
district_pe_120604,city_pe_1206,"Mazamari","120604"
district_pe_120605,city_pe_1206,"Pampa Hermosa","120605"
district_pe_120606,city_pe_1206,"Pangoa","120606"
district_pe_120607,city_pe_1206,"Río Negro","120607"
district_pe_120608,city_pe_1206,"Río Tambo","120608"
district_pe_120609,city_pe_1206,"Vizcatan del Enengoa","120609"
district_pe_120701,city_pe_1207,"Tarma","120701"
district_pe_120702,city_pe_1207,"Acobamba","120702"
district_pe_120703,city_pe_1207,"Huaricolca","120703"
district_pe_120704,city_pe_1207,"Huasahuasi","120704"
district_pe_120705,city_pe_1207,"La Unión","120705"
district_pe_120706,city_pe_1207,"Palca","120706"
district_pe_120707,city_pe_1207,"Palcamayo","120707"
district_pe_120708,city_pe_1207,"San Pedro de Cajas","120708"
district_pe_120709,city_pe_1207,"Tapo","120709"
district_pe_120801,city_pe_1208,"La Oroya","120801"
district_pe_120802,city_pe_1208,"Chacapalpa","120802"
district_pe_120803,city_pe_1208,"Huay-Huay","120803"
district_pe_120804,city_pe_1208,"Marcapomacocha","120804"
district_pe_120805,city_pe_1208,"Morococha","120805"
district_pe_120806,city_pe_1208,"Paccha","120806"
district_pe_120807,city_pe_1208,"Santa Bárbara de Carhuacayan","120807"
district_pe_120808,city_pe_1208,"Santa Rosa de Sacco","120808"
district_pe_120809,city_pe_1208,"Suitucancha","120809"
district_pe_120810,city_pe_1208,"Yauli","120810"
district_pe_120901,city_pe_1209,"Chupaca","120901"
district_pe_120902,city_pe_1209,"Ahuac","120902"
district_pe_120903,city_pe_1209,"Chongos Bajo","120903"
district_pe_120904,city_pe_1209,"Huachac","120904"
district_pe_120905,city_pe_1209,"Huamancaca Chico","120905"
district_pe_120906,city_pe_1209,"San Juan de Iscos","120906"
district_pe_120907,city_pe_1209,"San Juan de Jarpa","120907"
district_pe_120908,city_pe_1209,"Tres de Diciembre","120908"
district_pe_120909,city_pe_1209,"Yanacancha","120909"
district_pe_130101,city_pe_1301,"Trujillo","130101"
district_pe_130102,city_pe_1301,"El Porvenir","130102"
district_pe_130103,city_pe_1301,"Florencia de Mora","130103"
district_pe_130104,city_pe_1301,"Huanchaco","130104"
district_pe_130105,city_pe_1301,"La Esperanza","130105"
district_pe_130106,city_pe_1301,"Laredo","130106"
district_pe_130107,city_pe_1301,"Moche","130107"
district_pe_130108,city_pe_1301,"Poroto","130108"
district_pe_130109,city_pe_1301,"Salaverry","130109"
district_pe_130110,city_pe_1301,"Simbal","130110"
district_pe_130111,city_pe_1301,"Victor Larco Herrera","130111"
district_pe_130201,city_pe_1302,"Ascope","130201"
district_pe_130202,city_pe_1302,"Chicama","130202"
district_pe_130203,city_pe_1302,"Chocope","130203"
district_pe_130204,city_pe_1302,"Magdalena de Cao","130204"
district_pe_130205,city_pe_1302,"Paijan","130205"
district_pe_130206,city_pe_1302,"Rázuri","130206"
district_pe_130207,city_pe_1302,"Santiago de Cao","130207"
district_pe_130208,city_pe_1302,"Casa Grande","130208"
district_pe_130301,city_pe_1303,"Bolívar","130301"
district_pe_130302,city_pe_1303,"Bambamarca","130302"
district_pe_130303,city_pe_1303,"Condormarca","130303"
district_pe_130304,city_pe_1303,"Longotea","130304"
district_pe_130305,city_pe_1303,"Uchumarca","130305"
district_pe_130306,city_pe_1303,"Ucuncha","130306"
district_pe_130401,city_pe_1304,"Chepen","130401"
district_pe_130402,city_pe_1304,"Pacanga","130402"
district_pe_130403,city_pe_1304,"Pueblo Nuevo","130403"
district_pe_130501,city_pe_1305,"Julcán","130501"
district_pe_130502,city_pe_1305,"Calamarca","130502"
district_pe_130503,city_pe_1305,"Carabamba","130503"
district_pe_130504,city_pe_1305,"Huaso","130504"
district_pe_130601,city_pe_1306,"Otuzco","130601"
district_pe_130602,city_pe_1306,"Agallpampa","130602"
district_pe_130604,city_pe_1306,"Charat","130604"
district_pe_130605,city_pe_1306,"Huaranchal","130605"
district_pe_130606,city_pe_1306,"La Cuesta","130606"
district_pe_130608,city_pe_1306,"Mache","130608"
district_pe_130610,city_pe_1306,"Paranday","130610"
district_pe_130611,city_pe_1306,"Salpo","130611"
district_pe_130613,city_pe_1306,"Sinsicap","130613"
district_pe_130614,city_pe_1306,"Usquil","130614"
district_pe_130701,city_pe_1307,"San Pedro de Lloc","130701"
district_pe_130702,city_pe_1307,"Guadalupe","130702"
district_pe_130703,city_pe_1307,"Jequetepeque","130703"
district_pe_130704,city_pe_1307,"Pacasmayo","130704"
district_pe_130705,city_pe_1307,"San José","130705"
district_pe_130801,city_pe_1308,"Tayabamba","130801"
district_pe_130802,city_pe_1308,"Buldibuyo","130802"
district_pe_130803,city_pe_1308,"Chillia","130803"
district_pe_130804,city_pe_1308,"Huancaspata","130804"
district_pe_130805,city_pe_1308,"Huaylillas","130805"
district_pe_130806,city_pe_1308,"Huayo","130806"
district_pe_130807,city_pe_1308,"Ongon","130807"
district_pe_130808,city_pe_1308,"Parcoy","130808"
district_pe_130809,city_pe_1308,"Pataz","130809"
district_pe_130810,city_pe_1308,"Pias","130810"
district_pe_130811,city_pe_1308,"Santiago de Challas","130811"
district_pe_130812,city_pe_1308,"Taurija","130812"
district_pe_130813,city_pe_1308,"Urpay","130813"
district_pe_130901,city_pe_1309,"Huamachuco","130901"
district_pe_130902,city_pe_1309,"Chugay","130902"
district_pe_130903,city_pe_1309,"Cochorco","130903"
district_pe_130904,city_pe_1309,"Curgos","130904"
district_pe_130905,city_pe_1309,"Marcabal","130905"
district_pe_130906,city_pe_1309,"Sanagoran","130906"
district_pe_130907,city_pe_1309,"Sarin","130907"
district_pe_130908,city_pe_1309,"Sartimbamba","130908"
district_pe_131001,city_pe_1310,"Santiago de Chuco","131001"
district_pe_131002,city_pe_1310,"Angasmarca","131002"
district_pe_131003,city_pe_1310,"Cachicadan","131003"
district_pe_131004,city_pe_1310,"Mollebamba","131004"
district_pe_131005,city_pe_1310,"Mollepata","131005"
district_pe_131006,city_pe_1310,"Quiruvilca","131006"
district_pe_131007,city_pe_1310,"Santa Cruz de Chuca","131007"
district_pe_131008,city_pe_1310,"Sitabamba","131008"
district_pe_131101,city_pe_1311,"Cascas","131101"
district_pe_131102,city_pe_1311,"Lucma","131102"
district_pe_131103,city_pe_1311,"Marmot","131103"
district_pe_131104,city_pe_1311,"Sayapullo","131104"
district_pe_131201,city_pe_1312,"Viru","131201"
district_pe_131202,city_pe_1312,"Chao","131202"
district_pe_131203,city_pe_1312,"Guadalupito","131203"
district_pe_140101,city_pe_1401,"Chiclayo","140101"
district_pe_140102,city_pe_1401,"Chongoyape","140102"
district_pe_140103,city_pe_1401,"Eten","140103"
district_pe_140104,city_pe_1401,"Eten Puerto","140104"
district_pe_140105,city_pe_1401,"José Leonardo Ortiz","140105"
district_pe_140106,city_pe_1401,"La Victoria","140106"
district_pe_140107,city_pe_1401,"Lagunas","140107"
district_pe_140108,city_pe_1401,"Monsefu","140108"
district_pe_140109,city_pe_1401,"Nueva Arica","140109"
district_pe_140110,city_pe_1401,"Oyotun","140110"
district_pe_140111,city_pe_1401,"Picsi","140111"
district_pe_140112,city_pe_1401,"Pimentel","140112"
district_pe_140113,city_pe_1401,"Reque","140113"
district_pe_140114,city_pe_1401,"Santa Rosa","140114"
district_pe_140115,city_pe_1401,"Saña","140115"
district_pe_140116,city_pe_1401,"Cayalti","140116"
district_pe_140117,city_pe_1401,"Patapo","140117"
district_pe_140118,city_pe_1401,"Pomalca","140118"
district_pe_140119,city_pe_1401,"Pucala","140119"
district_pe_140120,city_pe_1401,"Tuman","140120"
district_pe_140201,city_pe_1402,"Ferreñafe","140201"
district_pe_140202,city_pe_1402,"Cañaris","140202"
district_pe_140203,city_pe_1402,"Incahuasi","140203"
district_pe_140204,city_pe_1402,"Manuel Antonio Mesones Muro","140204"
district_pe_140205,city_pe_1402,"Pitipo","140205"
district_pe_140206,city_pe_1402,"Pueblo Nuevo","140206"
district_pe_140301,city_pe_1403,"Lambayeque","140301"
district_pe_140302,city_pe_1403,"Chochope","140302"
district_pe_140303,city_pe_1403,"Illimo","140303"
district_pe_140304,city_pe_1403,"Jayanca","140304"
district_pe_140305,city_pe_1403,"Mochumi","140305"
district_pe_140306,city_pe_1403,"Morrope","140306"
district_pe_140307,city_pe_1403,"Motupe","140307"
district_pe_140308,city_pe_1403,"Olmos","140308"
district_pe_140309,city_pe_1403,"Pacora","140309"
district_pe_140310,city_pe_1403,"Salas","140310"
district_pe_140311,city_pe_1403,"San José","140311"
district_pe_140312,city_pe_1403,"Tucume","140312"
district_pe_150101,city_pe_1501,"Lima","150101"
district_pe_150102,city_pe_1501,"Ancón","150102"
district_pe_150103,city_pe_1501,"Ate","150103"
district_pe_150104,city_pe_1501,"Barranco","150104"
district_pe_150105,city_pe_1501,"Breña","150105"
district_pe_150106,city_pe_1501,"Carabayllo","150106"
district_pe_150107,city_pe_1501,"Chaclacayo","150107"
district_pe_150108,city_pe_1501,"Chorrillos","150108"
district_pe_150109,city_pe_1501,"Cieneguilla","150109"
district_pe_150110,city_pe_1501,"Comas","150110"
district_pe_150111,city_pe_1501,"El Agustíno","150111"
district_pe_150112,city_pe_1501,"Independencia","150112"
district_pe_150113,city_pe_1501,"Jesús María","150113"
district_pe_150114,city_pe_1501,"La Molina","150114"
district_pe_150115,city_pe_1501,"La Victoria","150115"
district_pe_150116,city_pe_1501,"Lince","150116"
district_pe_150117,city_pe_1501,"Los Olivos","150117"
district_pe_150118,city_pe_1501,"Lurigancho","150118"
district_pe_150119,city_pe_1501,"Lurin","150119"
district_pe_150120,city_pe_1501,"Magdalena del Mar","150120"
district_pe_150121,city_pe_1501,"Pueblo Libre","150121"
district_pe_150122,city_pe_1501,"Miraflores","150122"
district_pe_150123,city_pe_1501,"Pachacamac","150123"
district_pe_150124,city_pe_1501,"Pucusana","150124"
district_pe_150125,city_pe_1501,"Puente Piedra","150125"
district_pe_150126,city_pe_1501,"Punta Hermosa","150126"
district_pe_150127,city_pe_1501,"Punta Negra","150127"
district_pe_150128,city_pe_1501,"Rimac","150128"
district_pe_150129,city_pe_1501,"San Bartolo","150129"
district_pe_150130,city_pe_1501,"San Borja","150130"
district_pe_150131,city_pe_1501,"San Isidro","150131"
district_pe_150132,city_pe_1501,"San Juan de Lurigancho","150132"
district_pe_150133,city_pe_1501,"San Juan de Miraflores","150133"
district_pe_150134,city_pe_1501,"San Luis","150134"
district_pe_150135,city_pe_1501,"San Martín de Porres","150135"
district_pe_150136,city_pe_1501,"San Miguel","150136"
district_pe_150137,city_pe_1501,"Santa Anita","150137"
district_pe_150138,city_pe_1501,"Santa María del Mar","150138"
district_pe_150139,city_pe_1501,"Santa Rosa","150139"
district_pe_150140,city_pe_1501,"Santiago de Surco","150140"
district_pe_150141,city_pe_1501,"Surquillo","150141"
district_pe_150142,city_pe_1501,"Villa el Salvador","150142"
district_pe_150143,city_pe_1501,"Villa María del Triunfo","150143"
district_pe_150201,city_pe_1502,"Barranca","150201"
district_pe_150202,city_pe_1502,"Paramonga","150202"
district_pe_150203,city_pe_1502,"Pativilca","150203"
district_pe_150204,city_pe_1502,"Supe","150204"
district_pe_150205,city_pe_1502,"Supe Puerto","150205"
district_pe_150301,city_pe_1503,"Cajatambo","150301"
district_pe_150302,city_pe_1503,"Copa","150302"
district_pe_150303,city_pe_1503,"Gorgor","150303"
district_pe_150304,city_pe_1503,"Huancapon","150304"
district_pe_150305,city_pe_1503,"Manas","150305"
district_pe_150401,city_pe_1504,"Canta","150401"
district_pe_150402,city_pe_1504,"Arahuay","150402"
district_pe_150403,city_pe_1504,"Huamantanga","150403"
district_pe_150404,city_pe_1504,"Huaros","150404"
district_pe_150405,city_pe_1504,"Lachaqui","150405"
district_pe_150406,city_pe_1504,"San Buenaventura","150406"
district_pe_150407,city_pe_1504,"Santa Rosa de Quives","150407"
district_pe_150501,city_pe_1505,"San Vicente de Cañete","150501"
district_pe_150502,city_pe_1505,"Asia","150502"
district_pe_150503,city_pe_1505,"Calango","150503"
district_pe_150504,city_pe_1505,"Cerro Azul","150504"
district_pe_150505,city_pe_1505,"Chilca","150505"
district_pe_150506,city_pe_1505,"Coayllo","150506"
district_pe_150507,city_pe_1505,"Imperial","150507"
district_pe_150508,city_pe_1505,"Lunahuana","150508"
district_pe_150509,city_pe_1505,"Mala","150509"
district_pe_150510,city_pe_1505,"Nuevo Imperial","150510"
district_pe_150511,city_pe_1505,"Pacaran","150511"
district_pe_150512,city_pe_1505,"Quilmana","150512"
district_pe_150513,city_pe_1505,"San Antonio","150513"
district_pe_150514,city_pe_1505,"San Luis","150514"
district_pe_150515,city_pe_1505,"Santa Cruz de Flores","150515"
district_pe_150516,city_pe_1505,"Zuñiga","150516"
district_pe_150601,city_pe_1506,"Huaral","150601"
district_pe_150602,city_pe_1506,"Atavillos Alto","150602"
district_pe_150603,city_pe_1506,"Atavillos Bajo","150603"
district_pe_150604,city_pe_1506,"Aucallama","150604"
district_pe_150605,city_pe_1506,"Chancay","150605"
district_pe_150606,city_pe_1506,"Ihuari","150606"
district_pe_150607,city_pe_1506,"Lampian","150607"
district_pe_150608,city_pe_1506,"Pacaraos","150608"
district_pe_150609,city_pe_1506,"San Miguel de Acos","150609"
district_pe_150610,city_pe_1506,"Santa Cruz de Andamarca","150610"
district_pe_150611,city_pe_1506,"Sumbilca","150611"
district_pe_150612,city_pe_1506,"Veintisiete de Noviembre","150612"
district_pe_150701,city_pe_1507,"Matucana","150701"
district_pe_150702,city_pe_1507,"Antioquia","150702"
district_pe_150703,city_pe_1507,"Callahuanca","150703"
district_pe_150704,city_pe_1507,"Carampoma","150704"
district_pe_150705,city_pe_1507,"Chicla","150705"
district_pe_150706,city_pe_1507,"Cuenca","150706"
district_pe_150707,city_pe_1507,"Huachupampa","150707"
district_pe_150708,city_pe_1507,"Huanza","150708"
district_pe_150709,city_pe_1507,"Huarochiri","150709"
district_pe_150710,city_pe_1507,"Lahuaytambo","150710"
district_pe_150711,city_pe_1507,"Langa","150711"
district_pe_150712,city_pe_1507,"Laraos","150712"
district_pe_150713,city_pe_1507,"Maríatana","150713"
district_pe_150714,city_pe_1507,"Ricardo Palma","150714"
district_pe_150715,city_pe_1507,"San Andrés de Tupicocha","150715"
district_pe_150716,city_pe_1507,"San Antonio","150716"
district_pe_150717,city_pe_1507,"San Bartolome","150717"
district_pe_150718,city_pe_1507,"San Damian","150718"
district_pe_150719,city_pe_1507,"San Juan de Iris","150719"
district_pe_150720,city_pe_1507,"San Juan de Tantaranche","150720"
district_pe_150721,city_pe_1507,"San Lorenzo de Quinti","150721"
district_pe_150722,city_pe_1507,"San Mateo","150722"
district_pe_150723,city_pe_1507,"San Mateo de Otao","150723"
district_pe_150724,city_pe_1507,"San Pedro de Casta","150724"
district_pe_150725,city_pe_1507,"San Pedro de Huancayre","150725"
district_pe_150726,city_pe_1507,"Sangallaya","150726"
district_pe_150727,city_pe_1507,"Santa Cruz de Cocachacra","150727"
district_pe_150728,city_pe_1507,"Santa Eulalia","150728"
district_pe_150729,city_pe_1507,"Santiago de Anchucaya","150729"
district_pe_150730,city_pe_1507,"Santiago de Tuna","150730"
district_pe_150731,city_pe_1507,"Santo Domingo de los Olleros","150731"
district_pe_150732,city_pe_1507,"Surco","150732"
district_pe_150801,city_pe_1508,"Huacho","150801"
district_pe_150802,city_pe_1508,"Ambar","150802"
district_pe_150803,city_pe_1508,"Caleta de Carquin","150803"
district_pe_150804,city_pe_1508,"Checras","150804"
district_pe_150805,city_pe_1508,"Hualmay","150805"
district_pe_150806,city_pe_1508,"Huaura","150806"
district_pe_150807,city_pe_1508,"Leoncio Prado","150807"
district_pe_150808,city_pe_1508,"Paccho","150808"
district_pe_150809,city_pe_1508,"Santa Leonor","150809"
district_pe_150810,city_pe_1508,"Santa María","150810"
district_pe_150811,city_pe_1508,"Sayan","150811"
district_pe_150812,city_pe_1508,"Vegueta","150812"
district_pe_150901,city_pe_1509,"Oyon","150901"
district_pe_150902,city_pe_1509,"Andajes","150902"
district_pe_150903,city_pe_1509,"Caujul","150903"
district_pe_150904,city_pe_1509,"Cochamarca","150904"
district_pe_150905,city_pe_1509,"Navan","150905"
district_pe_150906,city_pe_1509,"Pachangara","150906"
district_pe_151001,city_pe_1510,"Yauyos","151001"
district_pe_151002,city_pe_1510,"Alis","151002"
district_pe_151003,city_pe_1510,"Allauca","151003"
district_pe_151004,city_pe_1510,"Ayaviri","151004"
district_pe_151005,city_pe_1510,"Azángaro","151005"
district_pe_151006,city_pe_1510,"Cacra","151006"
district_pe_151007,city_pe_1510,"Carania","151007"
district_pe_151008,city_pe_1510,"Catahuasi","151008"
district_pe_151009,city_pe_1510,"Chocos","151009"
district_pe_151010,city_pe_1510,"Cochas","151010"
district_pe_151011,city_pe_1510,"Colonia","151011"
district_pe_151012,city_pe_1510,"Hongos","151012"
district_pe_151013,city_pe_1510,"Huampara","151013"
district_pe_151014,city_pe_1510,"Huancaya","151014"
district_pe_151015,city_pe_1510,"Huangascar","151015"
district_pe_151016,city_pe_1510,"Huantan","151016"
district_pe_151017,city_pe_1510,"Huañec","151017"
district_pe_151018,city_pe_1510,"Laraos","151018"
district_pe_151019,city_pe_1510,"Lincha","151019"
district_pe_151020,city_pe_1510,"Madean","151020"
district_pe_151021,city_pe_1510,"Miraflores","151021"
district_pe_151022,city_pe_1510,"Omas","151022"
district_pe_151023,city_pe_1510,"Putinza","151023"
district_pe_151024,city_pe_1510,"Quinches","151024"
district_pe_151025,city_pe_1510,"Quinocay","151025"
district_pe_151026,city_pe_1510,"San Joaquin","151026"
district_pe_151027,city_pe_1510,"San Pedro de Pilas","151027"
district_pe_151028,city_pe_1510,"Tanta","151028"
district_pe_151029,city_pe_1510,"Tauripampa","151029"
district_pe_151030,city_pe_1510,"Tomas","151030"
district_pe_151031,city_pe_1510,"Tupe","151031"
district_pe_151032,city_pe_1510,"Viñac","151032"
district_pe_151033,city_pe_1510,"Vitis","151033"
district_pe_160101,city_pe_1601,"Iquitos","160101"
district_pe_160102,city_pe_1601,"Alto Nanay","160102"
district_pe_160103,city_pe_1601,"Fernando Lores","160103"
district_pe_160104,city_pe_1601,"Indiana","160104"
district_pe_160105,city_pe_1601,"Las Amazonas","160105"
district_pe_160106,city_pe_1601,"Mazan","160106"
district_pe_160107,city_pe_1601,"Napo","160107"
district_pe_160108,city_pe_1601,"Punchana","160108"
district_pe_160109,city_pe_1601,"Putumayo","160109"
district_pe_160110,city_pe_1601,"Torres Causana","160110"
district_pe_160112,city_pe_1601,"Belén","160112"
district_pe_160113,city_pe_1601,"San Juan Bautista","160113"
district_pe_160201,city_pe_1602,"Yurimaguas","160201"
district_pe_160202,city_pe_1602,"Balsapuerto","160202"
district_pe_160205,city_pe_1602,"Jeberos","160205"
district_pe_160206,city_pe_1602,"Lagunas","160206"
district_pe_160210,city_pe_1602,"Santa Cruz","160210"
district_pe_160211,city_pe_1602,"Teniente Cesar López Rojas","160211"
district_pe_160301,city_pe_1603,"Nauta","160301"
district_pe_160302,city_pe_1603,"Parinari","160302"
district_pe_160303,city_pe_1603,"Tigre","160303"
district_pe_160304,city_pe_1603,"Trompeteros","160304"
district_pe_160305,city_pe_1603,"Urarinas","160305"
district_pe_160401,city_pe_1604,"Ramón Castilla","160401"
district_pe_160402,city_pe_1604,"Pebas","160402"
district_pe_160403,city_pe_1604,"Yavari","160403"
district_pe_160404,city_pe_1604,"San Pablo","160404"
district_pe_160501,city_pe_1605,"Requena","160501"
district_pe_160502,city_pe_1605,"Alto Tapiche","160502"
district_pe_160503,city_pe_1605,"Capelo","160503"
district_pe_160504,city_pe_1605,"Emilio San Martín","160504"
district_pe_160505,city_pe_1605,"Maquia","160505"
district_pe_160506,city_pe_1605,"Puinahua","160506"
district_pe_160507,city_pe_1605,"Saquena","160507"
district_pe_160508,city_pe_1605,"Soplin","160508"
district_pe_160509,city_pe_1605,"Tapiche","160509"
district_pe_160510,city_pe_1605,"Jenaro Herrera","160510"
district_pe_160511,city_pe_1605,"Yaquerana","160511"
district_pe_160601,city_pe_1606,"Contamana","160601"
district_pe_160602,city_pe_1606,"Inahuaya","160602"
district_pe_160603,city_pe_1606,"Padre Márquez","160603"
district_pe_160604,city_pe_1606,"Pampa Hermosa","160604"
district_pe_160605,city_pe_1606,"Sarayacu","160605"
district_pe_160606,city_pe_1606,"Vargas Guerra","160606"
district_pe_160701,city_pe_1607,"Barranca","160701"
district_pe_160702,city_pe_1607,"Cahuapanas","160702"
district_pe_160703,city_pe_1607,"Manseriche","160703"
district_pe_160704,city_pe_1607,"Morona","160704"
district_pe_160705,city_pe_1607,"Pastaza","160705"
district_pe_160706,city_pe_1607,"Andoas","160706"
district_pe_160801,city_pe_1608,"Putumayo","160801"
district_pe_160802,city_pe_1608,"Rosa Panduro","160802"
district_pe_160803,city_pe_1608,"Teniente Manuel Clavero","160803"
district_pe_160804,city_pe_1608,"Yaguas","160804"
district_pe_170101,city_pe_1701,"Tambopata","170101"
district_pe_170102,city_pe_1701,"Inambari","170102"
district_pe_170103,city_pe_1701,"Las Piedras","170103"
district_pe_170104,city_pe_1701,"Laberinto","170104"
district_pe_170201,city_pe_1702,"Manu","170201"
district_pe_170202,city_pe_1702,"Fitzcarrald","170202"
district_pe_170203,city_pe_1702,"Madre de Dios","170203"
district_pe_170204,city_pe_1702,"Huepetuhe","170204"
district_pe_170301,city_pe_1703,"Iñapari","170301"
district_pe_170302,city_pe_1703,"Iberia","170302"
district_pe_170303,city_pe_1703,"Tahuamanu","170303"
district_pe_180101,city_pe_1801,"Moquegua","180101"
district_pe_180102,city_pe_1801,"Carumas","180102"
district_pe_180103,city_pe_1801,"Cuchumbaya","180103"
district_pe_180104,city_pe_1801,"Samegua","180104"
district_pe_180105,city_pe_1801,"San Cristóbal","180105"
district_pe_180106,city_pe_1801,"Torata","180106"
district_pe_180201,city_pe_1802,"Omate","180201"
district_pe_180202,city_pe_1802,"Chojata","180202"
district_pe_180203,city_pe_1802,"Coalaque","180203"
district_pe_180204,city_pe_1802,"Ichuña","180204"
district_pe_180205,city_pe_1802,"La Capilla","180205"
district_pe_180206,city_pe_1802,"Lloque","180206"
district_pe_180207,city_pe_1802,"Matalaque","180207"
district_pe_180208,city_pe_1802,"Puquina","180208"
district_pe_180209,city_pe_1802,"Quinistaquillas","180209"
district_pe_180210,city_pe_1802,"Ubinas","180210"
district_pe_180211,city_pe_1802,"Yunga","180211"
district_pe_180301,city_pe_1803,"Ilo","180301"
district_pe_180302,city_pe_1803,"El Algarrobal","180302"
district_pe_180303,city_pe_1803,"Pacocha","180303"
district_pe_190101,city_pe_1901,"Chaupimarca","190101"
district_pe_190102,city_pe_1901,"Huachon","190102"
district_pe_190103,city_pe_1901,"Huariaca","190103"
district_pe_190104,city_pe_1901,"Huayllay","190104"
district_pe_190105,city_pe_1901,"Ninacaca","190105"
district_pe_190106,city_pe_1901,"Pallanchacra","190106"
district_pe_190107,city_pe_1901,"Paucartambo","190107"
district_pe_190108,city_pe_1901,"San Francisco de Asís de Yarusyacan","190108"
district_pe_190109,city_pe_1901,"Simon Bolívar","190109"
district_pe_190110,city_pe_1901,"Ticlacayan","190110"
district_pe_190111,city_pe_1901,"Tinyahuarco","190111"
district_pe_190112,city_pe_1901,"Vicco","190112"
district_pe_190113,city_pe_1901,"Yanacancha","190113"
district_pe_190201,city_pe_1902,"Yanahuanca","190201"
district_pe_190202,city_pe_1902,"Chacayan","190202"
district_pe_190203,city_pe_1902,"Goyllarisquizga","190203"
district_pe_190204,city_pe_1902,"Paucar","190204"
district_pe_190205,city_pe_1902,"San Pedro de Pillao","190205"
district_pe_190206,city_pe_1902,"Santa Ana de Tusi","190206"
district_pe_190207,city_pe_1902,"Tapuc","190207"
district_pe_190208,city_pe_1902,"Vilcabamba","190208"
district_pe_190301,city_pe_1903,"Oxapampa","190301"
district_pe_190302,city_pe_1903,"Chontabamba","190302"
district_pe_190303,city_pe_1903,"Huancabamba","190303"
district_pe_190304,city_pe_1903,"Palcazu","190304"
district_pe_190305,city_pe_1903,"Pozuzo","190305"
district_pe_190306,city_pe_1903,"Puerto Bermúdez","190306"
district_pe_190307,city_pe_1903,"Villa Rica","190307"
district_pe_190308,city_pe_1903,"Constitución","190308"
district_pe_200101,city_pe_2001,"Piura","200101"
district_pe_200104,city_pe_2001,"Castilla","200104"
district_pe_200105,city_pe_2001,"Catacaos","200105"
district_pe_200107,city_pe_2001,"Cura Mori","200107"
district_pe_200108,city_pe_2001,"El Tallan","200108"
district_pe_200109,city_pe_2001,"La Arena","200109"
district_pe_200110,city_pe_2001,"La Unión","200110"
district_pe_200111,city_pe_2001,"Las Lomas","200111"
district_pe_200114,city_pe_2001,"Tambo Grande","200114"
district_pe_200115,city_pe_2001,"Veintiseis de Octubre","200115"
district_pe_200201,city_pe_2002,"Ayabaca","200201"
district_pe_200202,city_pe_2002,"Frias","200202"
district_pe_200203,city_pe_2002,"Jilili","200203"
district_pe_200204,city_pe_2002,"Lagunas","200204"
district_pe_200205,city_pe_2002,"Montero","200205"
district_pe_200206,city_pe_2002,"Pacaipampa","200206"
district_pe_200207,city_pe_2002,"Paimas","200207"
district_pe_200208,city_pe_2002,"Sapillica","200208"
district_pe_200209,city_pe_2002,"Sicchez","200209"
district_pe_200210,city_pe_2002,"Suyo","200210"
district_pe_200301,city_pe_2003,"Huancabamba","200301"
district_pe_200330,city_pe_2003,"Canchaque","200302"
district_pe_200303,city_pe_2003,"El Carmen de la Frontera","200303"
district_pe_200305,city_pe_2003,"Lalaquiz","200305"
district_pe_200306,city_pe_2003,"San Miguel de El Faique","200306"
district_pe_200307,city_pe_2003,"Sondor","200307"
district_pe_200308,city_pe_2003,"Sondorillo","200308"
district_pe_200401,city_pe_2004,"Chulucanas","200401"
district_pe_200402,city_pe_2004,"Buenos Aires","200402"
district_pe_200403,city_pe_2004,"Chalaco","200403"
district_pe_200404,city_pe_2004,"La Matanza","200404"
district_pe_200405,city_pe_2004,"Morropon","200405"
district_pe_200406,city_pe_2004,"Salitral","200406"
district_pe_200407,city_pe_2004,"San Juan de Bigote","200407"
district_pe_200408,city_pe_2004,"Santa Catalina de Mossa","200408"
district_pe_200409,city_pe_2004,"Santo Domingo","200409"
district_pe_200410,city_pe_2004,"Yamango","200410"
district_pe_200501,city_pe_2005,"Paita","200501"
district_pe_200502,city_pe_2005,"Amotape","200502"
district_pe_200503,city_pe_2005,"Arenal","200503"
district_pe_200504,city_pe_2005,"Colan","200504"
district_pe_200505,city_pe_2005,"La Huaca","200505"
district_pe_200506,city_pe_2005,"Tamarindo","200506"
district_pe_200507,city_pe_2005,"Vichayal","200507"
district_pe_200601,city_pe_2006,"Sullana","200601"
district_pe_200602,city_pe_2006,"Bellavista","200602"
district_pe_200603,city_pe_2006,"Ignacio Escudero","200603"
district_pe_200604,city_pe_2006,"Lancones","200604"
district_pe_200605,city_pe_2006,"Marcavelica","200605"
district_pe_200606,city_pe_2006,"Miguel Checa","200606"
district_pe_200607,city_pe_2006,"Querecotillo","200607"
district_pe_200608,city_pe_2006,"Salitral","200608"
district_pe_200701,city_pe_2007,"Pariñas","200701"
district_pe_200702,city_pe_2007,"El Alto","200702"
district_pe_200703,city_pe_2007,"La Brea","200703"
district_pe_200704,city_pe_2007,"Lobitos","200704"
district_pe_200705,city_pe_2007,"Los Organos","200705"
district_pe_200706,city_pe_2007,"Mancora","200706"
district_pe_200801,city_pe_2008,"Sechura","200801"
district_pe_200802,city_pe_2008,"Bellavista de la Unión","200802"
district_pe_200803,city_pe_2008,"Bernal","200803"
district_pe_200804,city_pe_2008,"Cristo Nos Valga","200804"
district_pe_200805,city_pe_2008,"Vice","200805"
district_pe_200806,city_pe_2008,"Rinconada Llicuar","200806"
district_pe_210101,city_pe_2101,"Puno","210101"
district_pe_210102,city_pe_2101,"Acora","210102"
district_pe_210103,city_pe_2101,"Amantani","210103"
district_pe_210104,city_pe_2101,"Atuncolla","210104"
district_pe_210105,city_pe_2101,"Capachica","210105"
district_pe_210106,city_pe_2101,"Chucuito","210106"
district_pe_210107,city_pe_2101,"Coata","210107"
district_pe_210108,city_pe_2101,"Huata","210108"
district_pe_210109,city_pe_2101,"Mañazo","210109"
district_pe_210110,city_pe_2101,"Paucarcolla","210110"
district_pe_210111,city_pe_2101,"Pichacani","210111"
district_pe_210112,city_pe_2101,"Plateria","210112"
district_pe_210113,city_pe_2101,"San Antonio","210113"
district_pe_210114,city_pe_2101,"Tiquillaca","210114"
district_pe_210115,city_pe_2101,"Vilque","210115"
district_pe_210201,city_pe_2102,"Azángaro","210201"
district_pe_210202,city_pe_2102,"Achaya","210202"
district_pe_210203,city_pe_2102,"Arapa","210203"
district_pe_210204,city_pe_2102,"Asillo","210204"
district_pe_210205,city_pe_2102,"Caminaca","210205"
district_pe_210206,city_pe_2102,"Chupa","210206"
district_pe_210207,city_pe_2102,"José Domingo Choquehuanca","210207"
district_pe_210208,city_pe_2102,"Muñani","210208"
district_pe_210209,city_pe_2102,"Potoni","210209"
district_pe_210210,city_pe_2102,"Saman","210210"
district_pe_210211,city_pe_2102,"San Anton","210211"
district_pe_210212,city_pe_2102,"San José","210212"
district_pe_210213,city_pe_2102,"San Juan de Salinas","210213"
district_pe_210214,city_pe_2102,"Santiago de Pupuja","210214"
district_pe_210215,city_pe_2102,"Tirapata","210215"
district_pe_210301,city_pe_2103,"Macusani","210301"
district_pe_210302,city_pe_2103,"Ajoyani","210302"
district_pe_210303,city_pe_2103,"Ayapata","210303"
district_pe_210304,city_pe_2103,"Coasa","210304"
district_pe_210305,city_pe_2103,"Corani","210305"
district_pe_210306,city_pe_2103,"Crucero","210306"
district_pe_210307,city_pe_2103,"Ituata","210307"
district_pe_210308,city_pe_2103,"Ollachea","210308"
district_pe_210309,city_pe_2103,"San Gaban","210309"
district_pe_210310,city_pe_2103,"Usicayos","210310"
district_pe_210401,city_pe_2104,"Juli","210401"
district_pe_210402,city_pe_2104,"Desaguadero","210402"
district_pe_210403,city_pe_2104,"Huacullani","210403"
district_pe_210404,city_pe_2104,"Kelluyo","210404"
district_pe_210405,city_pe_2104,"Pisacoma","210405"
district_pe_210406,city_pe_2104,"Pomata","210406"
district_pe_210407,city_pe_2104,"Zepita","210407"
district_pe_210501,city_pe_2105,"Ilave","210501"
district_pe_210502,city_pe_2105,"Capazo","210502"
district_pe_210503,city_pe_2105,"Pilcuyo","210503"
district_pe_210504,city_pe_2105,"Santa Rosa","210504"
district_pe_210505,city_pe_2105,"Conduriri","210505"
district_pe_210601,city_pe_2106,"Huancane","210601"
district_pe_210602,city_pe_2106,"Cojata","210602"
district_pe_210603,city_pe_2106,"Huatasani","210603"
district_pe_210604,city_pe_2106,"Inchupalla","210604"
district_pe_210605,city_pe_2106,"Pusi","210605"
district_pe_210606,city_pe_2106,"Rosaspata","210606"
district_pe_210607,city_pe_2106,"Taraco","210607"
district_pe_210608,city_pe_2106,"Vilque Chico","210608"
district_pe_210701,city_pe_2107,"Lampa","210701"
district_pe_210702,city_pe_2107,"Cabanilla","210702"
district_pe_210703,city_pe_2107,"Calapuja","210703"
district_pe_210704,city_pe_2107,"Nicasio","210704"
district_pe_210705,city_pe_2107,"Ocuviri","210705"
district_pe_210706,city_pe_2107,"Palca","210706"
district_pe_210707,city_pe_2107,"Paratia","210707"
district_pe_210708,city_pe_2107,"Pucara","210708"
district_pe_210709,city_pe_2107,"Santa Lucia","210709"
district_pe_210710,city_pe_2107,"Vilavila","210710"
district_pe_210801,city_pe_2108,"Ayaviri","210801"
district_pe_210802,city_pe_2108,"Antauta","210802"
district_pe_210803,city_pe_2108,"Cupi","210803"
district_pe_210804,city_pe_2108,"Llalli","210804"
district_pe_210805,city_pe_2108,"Macari","210805"
district_pe_210806,city_pe_2108,"Nuñoa","210806"
district_pe_210807,city_pe_2108,"Orurillo","210807"
district_pe_210808,city_pe_2108,"Santa Rosa","210808"
district_pe_210809,city_pe_2108,"Umachiri","210809"
district_pe_210901,city_pe_2109,"Moho","210901"
district_pe_210902,city_pe_2109,"Conima","210902"
district_pe_210903,city_pe_2109,"Huayrapata","210903"
district_pe_210904,city_pe_2109,"Tilali","210904"
district_pe_211001,city_pe_2110,"Putina","211001"
district_pe_211002,city_pe_2110,"Ananea","211002"
district_pe_211003,city_pe_2110,"Pedro Vilca Apaza","211003"
district_pe_211004,city_pe_2110,"Quilcapuncu","211004"
district_pe_211005,city_pe_2110,"Sina","211005"
district_pe_211101,city_pe_2111,"Juliaca","211101"
district_pe_211102,city_pe_2111,"Cabana","211102"
district_pe_211103,city_pe_2111,"Cabanillas","211103"
district_pe_211104,city_pe_2111,"Caracoto","211104"
district_pe_211105,city_pe_2111,"San Miguel","211105"
district_pe_211201,city_pe_2112,"Sandia","211201"
district_pe_211202,city_pe_2112,"Cuyocuyo","211202"
district_pe_211203,city_pe_2112,"Limbani","211203"
district_pe_211204,city_pe_2112,"Patambuco","211204"
district_pe_211205,city_pe_2112,"Phara","211205"
district_pe_211206,city_pe_2112,"Quiaca","211206"
district_pe_211207,city_pe_2112,"San Juan del Oro","211207"
district_pe_211208,city_pe_2112,"Yanahuaya","211208"
district_pe_211209,city_pe_2112,"Alto Inambari","211209"
district_pe_211210,city_pe_2112,"San Pedro de Putina Punco","211210"
district_pe_211301,city_pe_2113,"Yunguyo","211301"
district_pe_211302,city_pe_2113,"Anapia","211302"
district_pe_211303,city_pe_2113,"Copani","211303"
district_pe_211304,city_pe_2113,"Cuturapi","211304"
district_pe_211305,city_pe_2113,"Ollaraya","211305"
district_pe_211306,city_pe_2113,"Tinicachi","211306"
district_pe_211307,city_pe_2113,"Unicachi","211307"
district_pe_220101,city_pe_2201,"Moyobamba","220101"
district_pe_220102,city_pe_2201,"Calzada","220102"
district_pe_220103,city_pe_2201,"Habana","220103"
district_pe_220104,city_pe_2201,"Jepelacio","220104"
district_pe_220105,city_pe_2201,"Soritor","220105"
district_pe_220106,city_pe_2201,"Yantalo","220106"
district_pe_220201,city_pe_2202,"Bellavista","220201"
district_pe_220202,city_pe_2202,"Alto Biavo","220202"
district_pe_220203,city_pe_2202,"Bajo Biavo","220203"
district_pe_220204,city_pe_2202,"Huallaga","220204"
district_pe_220205,city_pe_2202,"San Pablo","220205"
district_pe_220206,city_pe_2202,"San Rafael","220206"
district_pe_220301,city_pe_2203,"San José de Sisa","220301"
district_pe_220302,city_pe_2203,"Agua Blanca","220302"
district_pe_220303,city_pe_2203,"San Martín","220303"
district_pe_220304,city_pe_2203,"Santa Rosa","220304"
district_pe_220305,city_pe_2203,"Shatoja","220305"
district_pe_220401,city_pe_2204,"Saposoa","220401"
district_pe_220402,city_pe_2204,"Alto Saposoa","220402"
district_pe_220403,city_pe_2204,"El Eslabón","220403"
district_pe_220404,city_pe_2204,"Piscoyacu","220404"
district_pe_220405,city_pe_2204,"Sacanche","220405"
district_pe_220406,city_pe_2204,"Tingo de Saposoa","220406"
district_pe_220501,city_pe_2205,"Lamas","220501"
district_pe_220502,city_pe_2205,"Alonso de Alvarado","220502"
district_pe_220503,city_pe_2205,"Barranquita","220503"
district_pe_220504,city_pe_2205,"Caynarachi","220504"
district_pe_220505,city_pe_2205,"Cuñumbuqui","220505"
district_pe_220506,city_pe_2205,"Pinto Recodo","220506"
district_pe_220507,city_pe_2205,"Rumisapa","220507"
district_pe_220508,city_pe_2205,"San Roque de Cumbaza","220508"
district_pe_220509,city_pe_2205,"Shanao","220509"
district_pe_220510,city_pe_2205,"Tabalosos","220510"
district_pe_220511,city_pe_2205,"Zapatero","220511"
district_pe_220601,city_pe_2206,"Juanjuí","220601"
district_pe_220602,city_pe_2206,"Campanilla","220602"
district_pe_220603,city_pe_2206,"Huicungo","220603"
district_pe_220604,city_pe_2206,"Pachiza","220604"
district_pe_220605,city_pe_2206,"Pajarillo","220605"
district_pe_220701,city_pe_2207,"Picota","220701"
district_pe_220702,city_pe_2207,"Buenos Aires","220702"
district_pe_220703,city_pe_2207,"Caspisapa","220703"
district_pe_220704,city_pe_2207,"Pilluana","220704"
district_pe_220705,city_pe_2207,"Pucacaca","220705"
district_pe_220706,city_pe_2207,"San Cristóbal","220706"
district_pe_220707,city_pe_2207,"San Hilarión","220707"
district_pe_220708,city_pe_2207,"Shamboyacu","220708"
district_pe_220709,city_pe_2207,"Tingo de Ponasa","220709"
district_pe_220710,city_pe_2207,"Tres Unidos","220710"
district_pe_220801,city_pe_2208,"Ríoja","220801"
district_pe_220802,city_pe_2208,"Awajun","220802"
district_pe_220803,city_pe_2208,"Elias Soplin Vargas","220803"
district_pe_220804,city_pe_2208,"Nueva Cajamarca","220804"
district_pe_220805,city_pe_2208,"Pardo Miguel","220805"
district_pe_220806,city_pe_2208,"Posic","220806"
district_pe_220807,city_pe_2208,"San Fernando","220807"
district_pe_220808,city_pe_2208,"Yorongos","220808"
district_pe_220809,city_pe_2208,"Yuracyacu","220809"
district_pe_220901,city_pe_2209,"Tarapoto","220901"
district_pe_220902,city_pe_2209,"Alberto Leveau","220902"
district_pe_220903,city_pe_2209,"Cacatachi","220903"
district_pe_220904,city_pe_2209,"Chazuta","220904"
district_pe_220905,city_pe_2209,"Chipurana","220905"
district_pe_220906,city_pe_2209,"El Porvenir","220906"
district_pe_220907,city_pe_2209,"Huimbayoc","220907"
district_pe_220908,city_pe_2209,"Juan Guerra","220908"
district_pe_220909,city_pe_2209,"La Banda de Shilcayo","220909"
district_pe_220910,city_pe_2209,"Morales","220910"
district_pe_220911,city_pe_2209,"Papaplaya","220911"
district_pe_220912,city_pe_2209,"San Antonio","220912"
district_pe_220913,city_pe_2209,"Sauce","220913"
district_pe_220914,city_pe_2209,"Shapaja","220914"
district_pe_221001,city_pe_2210,"Tocache","221001"
district_pe_221002,city_pe_2210,"Nuevo Progreso","221002"
district_pe_221003,city_pe_2210,"Polvora","221003"
district_pe_221004,city_pe_2210,"Shunte","221004"
district_pe_221005,city_pe_2210,"Uchiza","221005"
district_pe_230101,city_pe_2301,"Tacna","230101"
district_pe_230102,city_pe_2301,"Alto de la Alianza","230102"
district_pe_230103,city_pe_2301,"Calana","230103"
district_pe_230104,city_pe_2301,"Ciudad Nueva","230104"
district_pe_230105,city_pe_2301,"Inclan","230105"
district_pe_230106,city_pe_2301,"Pachia","230106"
district_pe_230107,city_pe_2301,"Palca","230107"
district_pe_230108,city_pe_2301,"Pocollay","230108"
district_pe_230109,city_pe_2301,"Sama","230109"
district_pe_230110,city_pe_2301,"Coronel Gregorio Albarracín Lanchipa","230110"
district_pe_230111,city_pe_2301,"La Yarada los Palos","230111"
district_pe_230201,city_pe_2302,"Candarave","230201"
district_pe_230202,city_pe_2302,"Cairani","230202"
district_pe_230203,city_pe_2302,"Camilaca","230203"
district_pe_230204,city_pe_2302,"Curibaya","230204"
district_pe_230205,city_pe_2302,"Huanuara","230205"
district_pe_230206,city_pe_2302,"Quilahuani","230206"
district_pe_230301,city_pe_2303,"Locumba","230301"
district_pe_230302,city_pe_2303,"Ilabaya","230302"
district_pe_230303,city_pe_2303,"Ite","230303"
district_pe_230401,city_pe_2304,"Tarata","230401"
district_pe_230402,city_pe_2304,"Heroes Albarracín","230402"
district_pe_230403,city_pe_2304,"Estique","230403"
district_pe_230404,city_pe_2304,"Estique-Pampa","230404"
district_pe_230405,city_pe_2304,"Sitajara","230405"
district_pe_230406,city_pe_2304,"Susapaya","230406"
district_pe_230407,city_pe_2304,"Tarucachi","230407"
district_pe_230408,city_pe_2304,"Ticaco","230408"
district_pe_240101,city_pe_2401,"Tumbes","240101"
district_pe_240102,city_pe_2401,"Corrales","240102"
district_pe_240103,city_pe_2401,"La Cruz","240103"
district_pe_240104,city_pe_2401,"Pampas de Hospital","240104"
district_pe_240105,city_pe_2401,"San Jacinto","240105"
district_pe_240106,city_pe_2401,"San Juan de la Virgen","240106"
district_pe_240201,city_pe_2402,"Zorritos","240201"
district_pe_240202,city_pe_2402,"Casitas","240202"
district_pe_240203,city_pe_2402,"Canoas de Punta Sal","240203"
district_pe_240301,city_pe_2403,"Zarumilla","240301"
district_pe_240302,city_pe_2403,"Aguas Verdes","240302"
district_pe_240303,city_pe_2403,"Matapalo","240303"
district_pe_240304,city_pe_2403,"Papayal","240304"
district_pe_250101,city_pe_2501,"Calleria","250101"
district_pe_250102,city_pe_2501,"Campoverde","250102"
district_pe_250103,city_pe_2501,"Iparia","250103"
district_pe_250104,city_pe_2501,"Masisea","250104"
district_pe_250105,city_pe_2501,"Yarinacocha","250105"
district_pe_250106,city_pe_2501,"Nueva Requena","250106"
district_pe_250107,city_pe_2501,"Manantay","250107"
district_pe_250201,city_pe_2502,"Raymondi","250201"
district_pe_250202,city_pe_2502,"Sepahua","250202"
district_pe_250203,city_pe_2502,"Tahuania","250203"
district_pe_250204,city_pe_2502,"Yurua","250204"
district_pe_250301,city_pe_2503,"Padre Abad","250301"
district_pe_250302,city_pe_2503,"Irazola","250302"
district_pe_250303,city_pe_2503,"Curimana","250303"
district_pe_250304,city_pe_2503,"Neshuya","250304"
district_pe_250305,city_pe_2503,"Alexander Von Humboldt","250305"
district_pe_250401,city_pe_2504,"Purus","250401"

```

## File: data\l10n_pe_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pe_chart_template" model="account.chart.template">
        <field name="name">Peru - PCGE 2019</field>
        <field name="bank_account_code_prefix">1041</field>
        <field name="cash_account_code_prefix">1031</field>
        <field name="transfer_account_code_prefix">1051</field>
        <field name="code_digits">7</field>
        <field name="currency_id" ref="base.PEN"/>
        <field name="country_id" ref="base.pe"/>
    </record>
</odoo>

```

## File: data\l10n_pe_chart_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pe_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="chart1213"/>
        <field name="default_pos_receivable_account_id" ref="chart1215" />
        <field name="property_account_payable_id" ref="chart4212"/>
        <field name="property_account_expense_categ_id" ref="chart6329"/>
        <field name="property_account_expense_id" ref="chart6011"/>
        <field name="property_account_income_categ_id" ref="chart70121"/>
        <field name="property_stock_account_input_categ_id" ref="chart6111"/>
        <field name="property_stock_account_output_categ_id" ref="chart69111"/>
        <field name="property_stock_valuation_account_id" ref="chart20111"/>
        <field name="income_currency_exchange_account_id" ref="chart776"/>
        <field name="expense_currency_exchange_account_id" ref="chart676"/>
        <field name="account_journal_early_pay_discount_loss_account_id" ref="chart675"/>
        <field name="account_journal_early_pay_discount_gain_account_id" ref="chart775"/>
    </record>

<!--    transfer_account_code_prefix-->
</odoo>

```

## File: data\res.city.csv

```csv
"id","country_id:id","state_id:id","name","l10n_pe_code"
city_pe_0101,base.pe,base.state_pe_01,"Chachapoyas","0101"
city_pe_0102,base.pe,base.state_pe_01,"Bagua","0102"
city_pe_0103,base.pe,base.state_pe_01,"Bongara","0103"
city_pe_0104,base.pe,base.state_pe_01,"Condorcanqui","0104"
city_pe_0105,base.pe,base.state_pe_01,"Luya","0105"
city_pe_0106,base.pe,base.state_pe_01,"Rodriguez de Mendoza","0106"
city_pe_0107,base.pe,base.state_pe_01,"Utcubamba","0107"
city_pe_0201,base.pe,base.state_pe_02,"Huaraz","0201"
city_pe_0202,base.pe,base.state_pe_02,"Aija","0202"
city_pe_0203,base.pe,base.state_pe_02,"Antonio Raymondi","0203"
city_pe_0204,base.pe,base.state_pe_02,"Asunción","0204"
city_pe_0205,base.pe,base.state_pe_02,"Bolognesi","0205"
city_pe_0206,base.pe,base.state_pe_02,"Carhuaz","0206"
city_pe_0207,base.pe,base.state_pe_02,"Carlos Fermin Fitzcarrald","0207"
city_pe_0208,base.pe,base.state_pe_02,"Casma","0208"
city_pe_0209,base.pe,base.state_pe_02,"Corongo","0209"
city_pe_0210,base.pe,base.state_pe_02,"Huari","0210"
city_pe_0211,base.pe,base.state_pe_02,"Huarmey","0211"
city_pe_0212,base.pe,base.state_pe_02,"Huaylas","0212"
city_pe_0213,base.pe,base.state_pe_02,"Mariscal Luzuriaga","0213"
city_pe_0214,base.pe,base.state_pe_02,"Ocros","0214"
city_pe_0215,base.pe,base.state_pe_02,"Pallasca","0215"
city_pe_0216,base.pe,base.state_pe_02,"Pomabamba","0216"
city_pe_0217,base.pe,base.state_pe_02,"Recuay","0217"
city_pe_0218,base.pe,base.state_pe_02,"Santa","0218"
city_pe_0219,base.pe,base.state_pe_02,"Sihuas","0219"
city_pe_0220,base.pe,base.state_pe_02,"Yungay","0220"
city_pe_0301,base.pe,base.state_pe_03,"Abancay","0301"
city_pe_0302,base.pe,base.state_pe_03,"Andahuaylas","0302"
city_pe_0303,base.pe,base.state_pe_03,"Antabamba","0303"
city_pe_0304,base.pe,base.state_pe_03,"Aymaraes","0304"
city_pe_0305,base.pe,base.state_pe_03,"Cotabambas","0305"
city_pe_0306,base.pe,base.state_pe_03,"Chincheros","0306"
city_pe_0307,base.pe,base.state_pe_03,"Grau","0307"
city_pe_0401,base.pe,base.state_pe_04,"Arequipa","0401"
city_pe_0402,base.pe,base.state_pe_04,"Camana","0402"
city_pe_0403,base.pe,base.state_pe_04,"Caraveli","0403"
city_pe_0404,base.pe,base.state_pe_04,"Castilla","0404"
city_pe_0405,base.pe,base.state_pe_04,"Caylloma","0405"
city_pe_0406,base.pe,base.state_pe_04,"Condesuyos","0406"
city_pe_0407,base.pe,base.state_pe_04,"Islay","0407"
city_pe_0408,base.pe,base.state_pe_04,"La Union","0408"
city_pe_0501,base.pe,base.state_pe_05,"Huamanga","0501"
city_pe_0502,base.pe,base.state_pe_05,"Cangallo","0502"
city_pe_0503,base.pe,base.state_pe_05,"Huanca Sancos","0503"
city_pe_0504,base.pe,base.state_pe_05,"Huanta","0504"
city_pe_0505,base.pe,base.state_pe_05,"La mar","0505"
city_pe_0506,base.pe,base.state_pe_05,"Lucanas","0506"
city_pe_0507,base.pe,base.state_pe_05,"Parinacochas","0507"
city_pe_0508,base.pe,base.state_pe_05,"Paucar del Sara Sara","0508"
city_pe_0509,base.pe,base.state_pe_05,"Sucre","0509"
city_pe_0510,base.pe,base.state_pe_05,"Victor Fajardo","0510"
city_pe_0511,base.pe,base.state_pe_05,"Vilcas Huamán","0511"
city_pe_0601,base.pe,base.state_pe_06,"Cajamarca","0601"
city_pe_0602,base.pe,base.state_pe_06,"Cajabamba","0602"
city_pe_0603,base.pe,base.state_pe_06,"Celendin","0603"
city_pe_0604,base.pe,base.state_pe_06,"Chota","0604"
city_pe_0605,base.pe,base.state_pe_06,"Contumazá","0605"
city_pe_0606,base.pe,base.state_pe_06,"Cutervo","0606"
city_pe_0607,base.pe,base.state_pe_06,"Hualgayoc","0607"
city_pe_0608,base.pe,base.state_pe_06,"Jaén","0608"
city_pe_0609,base.pe,base.state_pe_06,"San Ignacio","0609"
city_pe_0610,base.pe,base.state_pe_06,"San Marcos","0610"
city_pe_0611,base.pe,base.state_pe_06,"San Miguel","0611"
city_pe_0612,base.pe,base.state_pe_06,"San Pablo","0612"
city_pe_0613,base.pe,base.state_pe_06,"Santa Cruz","0613"
city_pe_0701,base.pe,base.state_pe_07,"Callao","0701"
city_pe_0801,base.pe,base.state_pe_08,"Cusco","0801"
city_pe_0802,base.pe,base.state_pe_08,"Acomayo","0802"
city_pe_0803,base.pe,base.state_pe_08,"Anta","0803"
city_pe_0804,base.pe,base.state_pe_08,"Calca","0804"
city_pe_0805,base.pe,base.state_pe_08,"Canas","0805"
city_pe_0806,base.pe,base.state_pe_08,"Canchis","0806"
city_pe_0807,base.pe,base.state_pe_08,"Chumbivilcas","0807"
city_pe_0808,base.pe,base.state_pe_08,"Espinar","0808"
city_pe_0809,base.pe,base.state_pe_08,"La Convención","0809"
city_pe_0810,base.pe,base.state_pe_08,"Paruro","0810"
city_pe_0811,base.pe,base.state_pe_08,"Paucartambo","0811"
city_pe_0812,base.pe,base.state_pe_08,"Quispicanchi","0812"
city_pe_0813,base.pe,base.state_pe_08,"Urubamba","0813"
city_pe_0901,base.pe,base.state_pe_09,"Huancavelica","0901"
city_pe_0902,base.pe,base.state_pe_09,"Acobamba","0902"
city_pe_0903,base.pe,base.state_pe_09,"Angaraes","0903"
city_pe_0904,base.pe,base.state_pe_09,"Castrovirreyna","0904"
city_pe_0905,base.pe,base.state_pe_09,"Churcampa","0905"
city_pe_0906,base.pe,base.state_pe_09,"Huaytará","0906"
city_pe_0907,base.pe,base.state_pe_09,"Tayacaja","0907"
city_pe_1001,base.pe,base.state_pe_10,"Huánuco","1001"
city_pe_1002,base.pe,base.state_pe_10,"Ambo","1002"
city_pe_1003,base.pe,base.state_pe_10,"Dos de mayo","1003"
city_pe_1004,base.pe,base.state_pe_10,"Huacaybamba","1004"
city_pe_1005,base.pe,base.state_pe_10,"Huamalies","1005"
city_pe_1006,base.pe,base.state_pe_10,"Leoncio Prado","1006"
city_pe_1007,base.pe,base.state_pe_10,"Marañón","1007"
city_pe_1008,base.pe,base.state_pe_10,"Pachitea","1008"
city_pe_1009,base.pe,base.state_pe_10,"Puerto inca","1009"
city_pe_1010,base.pe,base.state_pe_10,"Lauricocha","1010"
city_pe_1011,base.pe,base.state_pe_10,"Yarowilca","1011"
city_pe_1101,base.pe,base.state_pe_11,"Ica","1101"
city_pe_1102,base.pe,base.state_pe_11,"Chincha","1102"
city_pe_1103,base.pe,base.state_pe_11,"Nazca","1103"
city_pe_1104,base.pe,base.state_pe_11,"Palpa","1104"
city_pe_1105,base.pe,base.state_pe_11,"Pisco","1105"
city_pe_1201,base.pe,base.state_pe_12,"Hua ncayo","1201"
city_pe_1202,base.pe,base.state_pe_12,"Concepción","1202"
city_pe_1203,base.pe,base.state_pe_12,"Chanchamayo","1203"
city_pe_1204,base.pe,base.state_pe_12,"Jauja","1204"
city_pe_1205,base.pe,base.state_pe_12,"Junin","1205"
city_pe_1206,base.pe,base.state_pe_12,"Satipo","1206"
city_pe_1207,base.pe,base.state_pe_12,"Tarma","1207"
city_pe_1208,base.pe,base.state_pe_12,"Yauli","1208"
city_pe_1209,base.pe,base.state_pe_12,"Chupaca","1209"
city_pe_1301,base.pe,base.state_pe_13,"Trujillo","1301"
city_pe_1302,base.pe,base.state_pe_13,"Ascope","1302"
city_pe_1303,base.pe,base.state_pe_13,"Bolivar","1303"
city_pe_1304,base.pe,base.state_pe_13,"Chepén","1304"
city_pe_1305,base.pe,base.state_pe_13,"Julcán","1305"
city_pe_1306,base.pe,base.state_pe_13,"Otuzco","1306"
city_pe_1307,base.pe,base.state_pe_13,"Pacasmayo","1307"
city_pe_1308,base.pe,base.state_pe_13,"Pataz","1308"
city_pe_1309,base.pe,base.state_pe_13,"Sánchez Carrión","1309"
city_pe_1310,base.pe,base.state_pe_13,"Santiago de Chuco","1310"
city_pe_1311,base.pe,base.state_pe_13,"Gran Chimú","1311"
city_pe_1312,base.pe,base.state_pe_13,"Virú","1312"
city_pe_1401,base.pe,base.state_pe_14,"Chiclayo","1401"
city_pe_1402,base.pe,base.state_pe_14,"Ferreñafe","1402"
city_pe_1403,base.pe,base.state_pe_14,"Lambayeque","1403"
city_pe_1501,base.pe,base.state_pe_15,"Lima","1501"
city_pe_1502,base.pe,base.state_pe_15,"Barranca","1502"
city_pe_1503,base.pe,base.state_pe_15,"Cajatambo","1503"
city_pe_1504,base.pe,base.state_pe_15,"Canta","1504"
city_pe_1505,base.pe,base.state_pe_15,"Cañete","1505"
city_pe_1506,base.pe,base.state_pe_15,"Huaral","1506"
city_pe_1507,base.pe,base.state_pe_15,"Huarochiri","1507"
city_pe_1508,base.pe,base.state_pe_15,"Huaura","1508"
city_pe_1509,base.pe,base.state_pe_15,"Oyón","1509"
city_pe_1510,base.pe,base.state_pe_15,"Yauyos","1510"
city_pe_1601,base.pe,base.state_pe_16,"Maynas","1601"
city_pe_1602,base.pe,base.state_pe_16,"Alto Amazonas","1602"
city_pe_1603,base.pe,base.state_pe_16,"Loreto","1603"
city_pe_1604,base.pe,base.state_pe_16,"Mariscal Ramón Castilla","1604"
city_pe_1605,base.pe,base.state_pe_16,"Requena","1605"
city_pe_1606,base.pe,base.state_pe_16,"Ucayali","1606"
city_pe_1607,base.pe,base.state_pe_16,"Datem del Marañón","1607"
city_pe_1608,base.pe,base.state_pe_16,"Putumayo","1608"
city_pe_1701,base.pe,base.state_pe_17,"Tambopata","1701"
city_pe_1702,base.pe,base.state_pe_17,"Manu","1702"
city_pe_1703,base.pe,base.state_pe_17,"Tahuamanu","1703"
city_pe_1801,base.pe,base.state_pe_18,"Mariscal Nieto","1801"
city_pe_1802,base.pe,base.state_pe_18,"General Sánchez Cerro","1802"
city_pe_1803,base.pe,base.state_pe_18,"Ilo","1803"
city_pe_1901,base.pe,base.state_pe_19,"Pasco","1901"
city_pe_1902,base.pe,base.state_pe_19,"Daniel Alcides Carrión","1902"
city_pe_1903,base.pe,base.state_pe_19,"Oxapampa","1903"
city_pe_2001,base.pe,base.state_pe_20,"Piura","2001"
city_pe_2002,base.pe,base.state_pe_20,"Ayabaca","2002"
city_pe_2003,base.pe,base.state_pe_20,"Huancabamba","2003"
city_pe_2004,base.pe,base.state_pe_20,"Morropón","2004"
city_pe_2005,base.pe,base.state_pe_20,"Paita","2005"
city_pe_2006,base.pe,base.state_pe_20,"Sullana","2006"
city_pe_2007,base.pe,base.state_pe_20,"Talara","2007"
city_pe_2008,base.pe,base.state_pe_20,"Sechura","2008"
city_pe_2101,base.pe,base.state_pe_21,"Puno","2101"
city_pe_2102,base.pe,base.state_pe_21,"Azángaro","2102"
city_pe_2103,base.pe,base.state_pe_21,"Carabaya","2103"
city_pe_2104,base.pe,base.state_pe_21,"Chucuito","2104"
city_pe_2105,base.pe,base.state_pe_21,"El Collao","2105"
city_pe_2106,base.pe,base.state_pe_21,"Huancané","2106"
city_pe_2107,base.pe,base.state_pe_21,"Lampa","2107"
city_pe_2108,base.pe,base.state_pe_21,"Melgar","2108"
city_pe_2109,base.pe,base.state_pe_21,"Moho","2109"
city_pe_2110,base.pe,base.state_pe_21,"San Antonio de Putina","2110"
city_pe_2111,base.pe,base.state_pe_21,"San Román","2111"
city_pe_2112,base.pe,base.state_pe_21,"Sandia","2112"
city_pe_2113,base.pe,base.state_pe_21,"Yunguyo","2113"
city_pe_2201,base.pe,base.state_pe_22,"Moyobamba","2201"
city_pe_2202,base.pe,base.state_pe_22,"Bellavista","2202"
city_pe_2203,base.pe,base.state_pe_22,"El Dorado","2203"
city_pe_2204,base.pe,base.state_pe_22,"Huallaga","2204"
city_pe_2205,base.pe,base.state_pe_22,"Lamas","2205"
city_pe_2206,base.pe,base.state_pe_22,"Mariscal Cáceres","2206"
city_pe_2207,base.pe,base.state_pe_22,"Picota","2207"
city_pe_2208,base.pe,base.state_pe_22,"Rioja","2208"
city_pe_2209,base.pe,base.state_pe_22,"San Martín","2209"
city_pe_2210,base.pe,base.state_pe_22,"Tocache","2210"
city_pe_2301,base.pe,base.state_pe_23,"Tacna","2301"
city_pe_2302,base.pe,base.state_pe_23,"Candarave","2302"
city_pe_2303,base.pe,base.state_pe_23,"Jorge Basadre","2303"
city_pe_2304,base.pe,base.state_pe_23,"Tarata","2304"
city_pe_2401,base.pe,base.state_pe_24,"Tumbes","2401"
city_pe_2402,base.pe,base.state_pe_24,"Contralmirante Villar","2402"
city_pe_2403,base.pe,base.state_pe_24,"Zarumilla","2403"
city_pe_2501,base.pe,base.state_pe_25,"Coronel Portillo","2501"
city_pe_2502,base.pe,base.state_pe_25,"Atalaya","2502"
city_pe_2503,base.pe,base.state_pe_25,"Padre Abad","2503"
city_pe_2504,base.pe,base.state_pe_25,"Purús","2504"

```

## File: data\res_country_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pe_partner_address_form" model="ir.ui.view">
        <field name="name">pe.partner.form.address</field>
        <field name="model">res.partner</field>
        <field name="priority" eval="900"/>
        <field name="arch" type="xml">
            <form>
                <div class="o_address_format">
                    <field name="country_enforce_cities" invisible="1"/>
                    <field name="parent_id" invisible="1"/>
                    <field name="type" invisible="1"/>
                    <field name="street" placeholder="Street..." class="o_address_street"
                           attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="street2" placeholder="Street 2..." class="o_address_street"
                           attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="l10n_pe_district" placeholder="District..." class="o_address_street"
                           attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="city_id"
                           placeholder="City"
                           class="o_address_city"
                           domain="[('country_id', '=', country_id)]"
                           attrs="{'invisible': [('country_enforce_cities', '=', False)], 'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"
                           context="{'default_country_id': country_id, 'default_state_id': state_id, 'default_zipcode': zip}"/>
                    <field name="city"
                           placeholder="City"
                           class="o_address_city"
                           attrs="{'invisible': [('country_enforce_cities', '=', True), '|', ('city_id', '!=', False), ('city', 'in', ['',False])],
                                   'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="state_id" class="o_address_state" placeholder="State" options="{'no_open': True, 'no_quick_create': True}"
                           context="{'default_country_id': country_id}"
                           attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="zip" placeholder="ZIP" class="o_address_zip"
                           attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="country_id" placeholder="Country" class="o_address_country" options='{"no_open": True, "no_create": True}'
                           attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                </div>
            </form>
        </field>
    </record>
    <record id="base.pe" model="res.country">
        <field name="enforce_cities" eval="1" />
        <field name="address_view_id" ref="pe_partner_address_form" />
        <field name="address_format" eval="'%(street)s\n%(l10n_pe_district_name)s\n%(zip)s%(city)s\n%(state_name)s\n%(country_name)s'"/>
    </record>
</odoo>

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, models, fields
from odoo.tools.sql import column_exists, create_column


class AccountMove(models.Model):
    _inherit = "account.move"

    def _get_l10n_latam_documents_domain(self):
        self.ensure_one()
        result = super()._get_l10n_latam_documents_domain()
        if self.company_id.country_id.code != "PE" or not self.journal_id.l10n_latam_use_documents:
            return result
        if self.journal_id.type == "sale":
            result.append(("code", "in", ("01", "03", "07", "08", "20", "40")))
        return result

    @api.onchange('l10n_latam_document_type_id', 'l10n_latam_document_number', 'partner_id')
    def _inverse_l10n_latam_document_number(self):
        """Inherit to complete the l10n_latam_document_number with the expected 8 characters after that a '-'
        Example: Change FFF-32 by FFF-00000032, to avoid incorrect values on the reports"""
        super()._inverse_l10n_latam_document_number()
        to_review = self.filtered(
            lambda x: x.journal_id.type == "purchase"
            and x.l10n_latam_document_type_id.code in ("01", "03", "07", "08")
            and x.l10n_latam_document_number
            and "-" in x.l10n_latam_document_number
            and x.l10n_latam_document_type_id.country_id.code == "PE"
        )
        for rec in to_review:
            number = rec.l10n_latam_document_number.split("-")
            rec.l10n_latam_document_number = "%s-%s" % (number[0], number[1].zfill(8))


class AccountMoveLine(models.Model):
    _inherit = "account.move.line"

    l10n_pe_group_id = fields.Many2one("account.group", related="account_id.group_id", store=True)

    def _auto_init(self):
        """
        Create column to stop ORM from computing it himself (too slow)
        """
        if not column_exists(self.env.cr, self._table, 'l10n_pe_group_id'):
            create_column(self.env.cr, self._table, 'l10n_pe_group_id', 'int4')
            self.env.cr.execute("""
                UPDATE account_move_line line
                SET l10n_pe_group_id = account.group_id
                FROM account_account account
                WHERE account.id = line.account_id
            """)
        return super()._auto_init()

```

## File: models\account_tax.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class AccountTax(models.Model):
    _inherit = "account.tax"

    l10n_pe_edi_tax_code = fields.Selection([
        ('1000', 'IGV - General Sales Tax'),
        ('1016', 'IVAP - Tax on Sale Paddy Rice'),
        ('2000', 'ISC - Selective Excise Tax'),
        ('7152', 'ICBPER - Plastic bag tax'),
        ('9995', 'EXP - Exportation'),
        ('9996', 'GRA - Free'),
        ('9997', 'EXO - Exonerated'),
        ('9998', 'INA - Unaffected'),
        ('9999', 'OTROS - Other taxes')
    ], 'EDI peruvian code')

    l10n_pe_edi_unece_category = fields.Selection([
        ('E', 'Exempt from tax'),
        ('G', 'Free export item, tax not charged'),
        ('O', 'Services outside scope of tax'),
        ('S', 'Standard rate'),
        ('Z', 'Zero rated goods')], 'EDI UNECE code',
        help="Follow the UN/ECE 5305 standard from the United Nations Economic Commission for Europe for more "
             "information http://www.unece.org/trade/untdid/d08a/tred/tred5305.htm"
    )
    l10n_pe_edi_isc_type = fields.Selection([
        ('01', 'System to value'),
        ('02', 'Application of the Fixed Amount'),
        ('03', 'Retail Price System'),
    ], 'ISC Type',
        help='Used in Selective Consumption Tax to indicate the type of calculation for the ISC.')


class AccountTaxTemplate(models.Model):
    _inherit = "account.tax.template"

    l10n_pe_edi_tax_code = fields.Selection([
        ('1000', 'IGV - General Sales Tax'),
        ('1016', 'IVAP - Tax on Sale Paddy Rice'),
        ('2000', 'ISC - Selective Excise Tax'),
        ('7152', 'ICBPER - Plastic bag tax'),
        ('9995', 'EXP - Exportation'),
        ('9996', 'GRA - Free'),
        ('9997', 'EXO - Exonerated'),
        ('9998', 'INA - Unaffected'),
        ('9999', 'OTROS - Other taxes')
    ], 'EDI peruvian code')

    l10n_pe_edi_unece_category = fields.Selection([
        ('E', 'Exempt from tax'),
        ('G', 'Free export item, tax not charged'),
        ('O', 'Services outside scope of tax'),
        ('S', 'Standard rate'),
        ('Z', 'Zero rated goods')], 'EDI UNECE code',
        help="Follow the UN/ECE 5305 standard from the United Nations Economic Commission for Europe for more "
             "information  http://www.unece.org/trade/untdid/d08a/tred/tred5305.htm"
    )
    l10n_pe_edi_isc_type = fields.Selection([
        ('01', 'System to value'),
        ('02', 'Application of the Fixed Amount'),
        ('03', 'Retail Price System'),
    ], 'ISC Type',
        help='Used in Selective Consumption Tax to indicate the type of calculation for the ISC.')

    def _get_tax_vals(self, company, tax_template_to_tax):
        val = super()._get_tax_vals(company, tax_template_to_tax)
        val.update({
            'l10n_pe_edi_tax_code': self.l10n_pe_edi_tax_code,
            'l10n_pe_edi_unece_category': self.l10n_pe_edi_unece_category,
            'l10n_pe_edi_isc_type': self.l10n_pe_edi_isc_type,
        })
        return val

```

## File: models\l10n_latam_identification_type.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields


class L10nLatamIdentificationType(models.Model):

    _inherit = "l10n_latam.identification.type"

    l10n_pe_vat_code = fields.Char()

```

## File: models\res_city.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class City(models.Model):
    _inherit = "res.city"

    l10n_pe_code = fields.Char('Code', help='This code will help with the '
                               'identification of each city in Peru.')

```

## File: models\res_city_district.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class L10nPeResCityDistrict(models.Model):
    _name = 'l10n_pe.res.city.district'
    _description = 'District'
    _order = 'name'

    name = fields.Char(translate=True)
    city_id = fields.Many2one('res.city', 'City')
    code = fields.Char(
        help='This code will help with the identification of each district '
        'in Peru.')

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class ResCompany(models.Model):
    _inherit = 'res.company'

    def _localization_use_documents(self):
        # OVERRIDE
        self.ensure_one()
        return self.account_fiscal_country_id.code == "PE" or super()._localization_use_documents()

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details
from odoo import fields, models, api


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_pe_district = fields.Many2one(
        'l10n_pe.res.city.district', string='District',
        help='Districts are part of a province or city.')
    l10n_pe_district_name = fields.Char(string='District name', related='l10n_pe_district.name')

    @api.onchange('l10n_pe_district')
    def _onchange_l10n_pe_district(self):
        if self.l10n_pe_district:
            self.city_id = self.l10n_pe_district.city_id

    @api.onchange('city_id')
    def _onchange_l10n_pe_city_id(self):
        if self.city_id and self.l10n_pe_district.city_id and self.l10n_pe_district.city_id != self.city_id:
            self.l10n_pe_district = False

    @api.model
    def _formatting_address_fields(self):
        """Returns the list of address fields usable to format addresses."""
        return super()._formatting_address_fields() + ['l10n_pe_district_name']

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_tax
from . import account_move
from . import l10n_latam_identification_type
from . import res_partner
from . import res_city_district
from . import res_city
from . import res_company

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
"access_l10n_pe_district_group_user","access_l10n_pe_district group_user","model_l10n_pe_res_city_district",base.group_user,1,1,1,1
"access_l10n_pe_district_group_all","access_l10n_pe_district group_all","model_l10n_pe_res_city_district",,1,0,0,0

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.8" y="5.8" width="50.4" height="36.38" maskUnits="userSpaceOnUse">
      <rect x="6.29" y="7.38" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="800" height="533" transform="translate(4.8 5.8) scale(0.06 0.07)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAyAAAAJCCAYAAAA4O6ONAAAACXBIWXMAAK/IAACvyAF3Om7hAAAgAElEQVR4Xuzdd3hddeHH8fe5M3vvpOnee++WQqGlgAoCArJRQOGHIAIyRAFBBJSNsreIsjeU0r3SPZK2aZKmzd7JzbrznN8fKEPaBhUP0H5ez9M/kvM953tu0ifJ+55lNOOxEBE5iJjTfoT3hQd7GiYih7nA6ZfQ9dfHehomIoc5R08DREREREREvioKEBERERERsY0CREREREREbKMAERERERER2yhARERERETENgoQERERERGxjQJERERERERsowARERERERHbKEBERERERMQ2ChAREREREbGNAkRERERERGyjABEREREREdsoQERERERExDYKEBERERERsY0CREREREREbKMAERERERER2yhARERERETENgoQERERERGxjQJERERERERsowARERERERHbKEBERERERMQ2ChAREREREbGNAkRERERERGyjABEREREREdsoQERERERExDYKEBERERERsY0CREREREREbKMAERERERER2yhARERERETENgoQERERERGxjQJERERERERsowARERERERHbKEBERERERMQ2ChAREREREbGNAkRERERERGyjABEREREREdsoQERERERExDYKEBERERERsY0CREREREREbKMAERERERER2yhARERERETENgoQERERERGxjQJERERERERsowARERERERHbKEBERERERMQ2ChAREREREbGNAkRERERERGyjABEREREREdsoQERERERExDYKEBERERERsY0CREREREREbKMAERERERER2yhARERERETENgoQERERERGxjQJERERERERsowARERERERHbKEBERERERMQ2ChAREREREbGNAkRERERERGyjABEREREREdsoQERERERExDYKEBERERERsY0CREREREREbKMAERERERER2yhARERERETENgoQERERERGxjQJERERERERsowARERERERHbKEBERERERMQ2ChAREREREbGNAkRERERERGyjABEREREREdsoQERERERExDYKEBERERERsY0CREREREREbKMAERERERER2yhARERERETENgoQERERERGxjQJERERERERsowARERERERHbKEBERERERMQ2ChAREREREbGNAkRERERERGyjABEREREREdsoQERERERExDYKEBERERERsY0CREREREREbKMAERERERER2yhARERERETENgoQERERERGxjQJERERERERsowARERERERHbKEBERERERMQ2ChAREREREbGNAkRERERERGyjABEREREREdsoQERERERExDYKEBERERERsY0CREREREREbKMAERERERER2yhARERERETENgoQERERERGxjQJERERERERsowARERERERHbKEBERERERMQ2ChAREREREbGNAkRERERERGyjABEREREREdsoQERERERExDYKEBERERERsY0CREREREREbKMAERERERER2yhARERERETENgoQERERERGxjQJERERERERsowARERERERHbKEBERERERMQ2ChAREREREbGNAkRERERERGyjABEREREREdsoQERERERExDYKEBERERERsY0CREREREREbKMAERERERER2yhARERERETENgoQERERERGxjQJERERERERsowARERERERHbKEBERERERMQ2ChAREREREbGNAkRERERERGyjABEREREREdsoQERERERExDYKEBERERERsY0CREREREREbKMAERERERER2yhARERERETENgoQERERERGxjQJERERERERsowARERERERHbKEBERERERMQ2ChAREREREbGNAkRERERERGyjABEREREREdsoQERERERExDYKEBERERERsY0CREREREREbKMAERERERER2yhARERERETENgoQERERERGxjQJERERERERsowARERERERHbKEBERERERMQ2ChAREREREbGNAkRERERERGyjABEREREREdsoQERERERExDYKEBERERERsY0CREREREREbKMAERERERER2yhARERERETENgoQERERERGxjQJERERERERsowARERERERHbKEBERERERMQ2ChAREREREbGNAkRERERERGyjABEREREREdsoQERERERExDYKEBERERERsY0CREREREREbOPqaYCIiBw6GhrXsWL7H5k9+hq2lzxLUsoYRvU/63Nj6uvXsqFmCbGY1NcvZPLgX9Arf8EBtvh5YX897yw9BW/yRExvJr1isxgx4PPbr61fzYadDzF37C18uPEa8nKPY/SAsw+wRREROdQoQEREDhNWJMjr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"/>
    </g>
  </g>
</svg>

```

## File: views\account_tax_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_tax_form" model="ir.ui.view">
        <field name="name">account.tax.form</field>
        <field name="model">account.tax</field>
        <field name="inherit_id" ref="account.view_tax_form"/>
        <field name="priority" eval="900"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='type_tax_use']" position="before">
                <field name="l10n_pe_edi_tax_code"
                       attrs="{'invisible': [('country_code', '!=', 'PE')]}"/>
                <field name="l10n_pe_edi_unece_category"
                       attrs="{'invisible': [('country_code', '!=', 'PE')]}"/>
                <field name="l10n_pe_edi_isc_type"
                       attrs="{'invisible': ['|', ('l10n_pe_edi_tax_code', '!=', '2000'), ('country_code', '!=', 'PE')]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

