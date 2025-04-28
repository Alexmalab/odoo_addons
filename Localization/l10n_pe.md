# Odoo Module: l10n_pe

Category: Localization

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

The tree of the CoA is done using account groups, all the accounts with move
are available as groups but only the more common ones are available as actual
accounts, if you want to create a new one use the group of accounts as
reference.

# TODO: Image showing what I am talking about.

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

# TODO: Describe new fields.

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

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Peru - Accounting',
    "version": "2.0",
    'summary': "PCGE Simplified",
    'category': 'Localization',
    'author': 'Vauxoo, Odoo',
    'license': 'LGPL-3',
    'depends': [
        'base_vat',
        'base_address_extended',
        'base_address_city',
        'l10n_latam_base',
    ],
    'data': [
        'security/ir.model.access.csv',
        'views/account_tax_view.xml',
        'data/l10n_pe_chart_data.xml',
        'data/account.group.csv',
        'data/account.account.template.csv',
        'data/l10n_pe_chart_post_data.xml',
        'data/account_tax_data.xml',
        'data/account_chart_template_data.xml',
        'data/res.city.csv',
        'data/l10n_pe.res.city.district.csv',
        'data/res_country_data.xml',
        'data/l10n_latam_identification_type_data.xml',
    ],
}

```

## File: data\account.account.template.csv

```csv
id,code,name,group_id:id,user_type_id:id,chart_template_id:id,reconcile
chart0111,0111,Bienes y valores entregados,group01,account.data_account_off_sheet,pe_chart_template,False
chart0112,0112,Derechos sobre instrumentos financiero,group01,account.data_account_off_sheet,pe_chart_template,False
chart0113,0113,Otras cuentas de orden deudora,group01,account.data_account_off_sheet,pe_chart_template,False
chart0114,0114,Deudoras por contra,group01,account.data_account_off_sheet,pe_chart_template,False
chart0126,0126,Bienes y valores recibido,group01,account.data_account_off_sheet,pe_chart_template,False
chart0127,0127,Compromisos sobre instrumentos financieros,group01,account.data_account_off_sheet,pe_chart_template,False
chart0128,0128,Otras cuentas de orden acreedoras,group01,account.data_account_off_sheet,pe_chart_template,False
chart0129,0129,Acreedoras por contra,group01,account.data_account_off_sheet,pe_chart_template,False
chart0211,0211,Bienes y valores entregados,group02,account.data_account_off_sheet,pe_chart_template,False
chart0213,0213,Derechos sobre instrumentos financieros,group02,account.data_account_off_sheet,pe_chart_template,False
chart0214,0214,Otras cuentas de orden deudoras,group02,account.data_account_off_sheet,pe_chart_template,False
chart0215,0215,Contrapartida cuentas de orden deudora,group02,account.data_account_off_sheet,pe_chart_template,False
chart0226,0226,Bienes y valores recibidos,group02,account.data_account_off_sheet,pe_chart_template,False
chart0227,0227,Compromisos sobre instrumentos financieros,group02,account.data_account_off_sheet,pe_chart_template,False
chart0228,0228,Otras cuentas de orden acreedoras,group02,account.data_account_off_sheet,pe_chart_template,False
chart0229,0229,Contrapartidacuentas de orden acreedoras,group02,account.data_account_off_sheet,pe_chart_template,False
chart101,101,Caja,group10,account.data_account_type_liquidity,pe_chart_template,False
chart102,102,Fondos fijos,group10,account.data_account_type_liquidity,pe_chart_template,False
chart1031,1031,Fondos fijos - Efectivo en tránsito,group10,account.data_account_type_liquidity,pe_chart_template,False
chart1032,1032,Fondos fijos - Cheques en tránsito,group10,account.data_account_type_liquidity,pe_chart_template,False
chart1041,1041,Cuentas corrientes en instituciones financieras - Cuentas corrientes operativas,group10,account.data_account_type_liquidity,pe_chart_template,False
chart1042,1042,Cuentas corrientes en instituciones financieras - Cuentas corrientes para fines específicos,group10,account.data_account_type_liquidity,pe_chart_template,False
chart1051,1051,Otros equivalentes de efectivo - Otro equivalentes de efectivo,group10,account.data_account_type_liquidity,pe_chart_template,False
chart1061,1061,Depósitos en instituciones financieras - Depósitos de ahorro,group10,account.data_account_type_liquidity,pe_chart_template,False
chart1062,1062,Depósitos en instituciones financieras - Depósitos a plazo,group10,account.data_account_type_liquidity,pe_chart_template,False
chart1071,1071,Fondos sujetos a restricción - Fondos en garantía,group10,account.data_account_type_liquidity,pe_chart_template,False
chart1072,1072,Fondos sujetos a restricción - Fondos retenidos por mandato de la autoridad,group10,account.data_account_type_liquidity,pe_chart_template,False
chart1073,1073,Fondos sujetos a restricción - Otros fondos sujetos a restricción,group10,account.data_account_type_liquidity,pe_chart_template,False
chart11111,11111,Inversiones mantenidas para negociación - Valores emitidos o garantizados por el Estado - Costo,group11,account.data_account_type_receivable,pe_chart_template,True
chart11112,11112,Inversiones mantenidas para negociación - Valores emitidos o garantizados por el Estado - Valor Razonable,group11,account.data_account_type_receivable,pe_chart_template,True
chart11121,11121,Inversiones mantenidas para negociación - Valores emitidos por el sistema financiero - Costo,group11,account.data_account_type_receivable,pe_chart_template,True
chart11122,11122,Inversiones mantenidas para negociación - Valores emitidos por el sistema financiero - Valor Razonable,group11,account.data_account_type_receivable,pe_chart_template,True
chart11131,11131,Inversiones mantenidas para negociación - Valores emitidos por entidades - Costo,group11,account.data_account_type_receivable,pe_chart_template,True
chart11132,11132,Inversiones mantenidas para negociación - Valores emitidos por entidades - Valor Razonable,group11,account.data_account_type_receivable,pe_chart_template,True
chart11141,11141,Inversiones mantenidas para negociación - Otros títulos representativos de deuda - Costo,group11,account.data_account_type_receivable,pe_chart_template,True
chart11142,11142,Inversiones mantenidas para negociación - Otros títulos representativos de deuda - Valor Razonable,group11,account.data_account_type_receivable,pe_chart_template,True
chart11151,11151,Inversiones mantenidas para negociación - Participaciones en entidades - Costo,group11,account.data_account_type_receivable,pe_chart_template,True
chart11152,11152,Inversiones mantenidas para negociación - Participaciones en entidades - Valor Razonable,group11,account.data_account_type_receivable,pe_chart_template,True
chart11211,11211,Otras inversiones financieras - Otras inversiones financieras - Costo,group11,account.data_account_type_receivable,pe_chart_template,True
chart11212,11212,Otras inversiones financieras - Otras inversiones financieras - Valor Razonable,group11,account.data_account_type_receivable,pe_chart_template,True
chart11311,11311,Activos financieros – Acuerdo de compra - Inversiones mantenidas para negociación – Acuerdo de compra - Costo,group11,account.data_account_type_receivable,pe_chart_template,True
chart11312,11312,Activos financieros – Acuerdo de compra - Inversiones mantenidas para negociación – Acuerdo de compra - Valor Razonable,group11,account.data_account_type_receivable,pe_chart_template,True
chart11321,11321,Activos financieros – Acuerdo de compra - Otras inversiones financieras - Costo,group11,account.data_account_type_receivable,pe_chart_template,True
chart11322,11322,Activos financieros – Acuerdo de compra - Otras inversiones financieras - Valor Razonable,group11,account.data_account_type_receivable,pe_chart_template,True
chart1211,1211,"Facturas, boletas y otros comprobantes por cobrar - No emitidas",group12,account.data_account_type_receivable,pe_chart_template,True
chart1212,1212,"Facturas, boletas y otros comprobantes por cobrar - Emitidas en cartera",group12,account.data_account_type_receivable,pe_chart_template,True
chart1213,1213,"Facturas, boletas y otros comprobantes por cobrar - En cobranza",group12,account.data_account_type_receivable,pe_chart_template,True
chart1214,1214,"Facturas, boletas y otros comprobantes por cobrar - En descuento",group12,account.data_account_type_receivable,pe_chart_template,True
chart1215,1215,"Facturas, boletas y otros comprobantes por cobrar - En cobranza PoS",group12,account.data_account_type_receivable,pe_chart_template,True
chart122,122,Anticipos de clientes,group12,account.data_account_type_receivable,pe_chart_template,True
chart1232,1232,Letras por cobrar - En cartera,group12,account.data_account_type_receivable,pe_chart_template,True
chart1233,1233,Letras por cobrar - En cobranza,group12,account.data_account_type_receivable,pe_chart_template,True
chart1234,1234,Letras por cobrar - En descuento,group12,account.data_account_type_receivable,pe_chart_template,True
chart1311,1311,"Facturas, boletas y otros comprobantes por cobrar - No emitidas",group13,account.data_account_type_receivable,pe_chart_template,True
chart1312,1312,"Facturas, boletas y otros comprobantes por cobrar - En cartera",group13,account.data_account_type_receivable,pe_chart_template,True
chart1313,1313,"Facturas, boletas y otros comprobantes por cobrar - En cobranza",group13,account.data_account_type_receivable,pe_chart_template,True
chart1314,1314,"Facturas, boletas y otros comprobantes por cobrar - En descuento",group13,account.data_account_type_receivable,pe_chart_template,True
chart1321,1321,Anticipos recibidos - Anticipos recibidos,group13,account.data_account_type_receivable,pe_chart_template,True
chart1331,1331,Letras por cobrar - En cartera,group13,account.data_account_type_receivable,pe_chart_template,True
chart1332,1332,Letras por cobrar - En cobranza,group13,account.data_account_type_receivable,pe_chart_template,True
chart1333,1333,Letras por cobrar - En descuento,group13,account.data_account_type_receivable,pe_chart_template,True
chart1411,1411,Personal - Préstamos,group14,account.data_account_type_receivable,pe_chart_template,True
chart1412,1412,Personal - Adelanto de remuneraciones,group14,account.data_account_type_receivable,pe_chart_template,True
chart1413,1413,Personal - Entregas a rendir cuenta,group14,account.data_account_type_receivable,pe_chart_template,True
chart1419,1419,Personal - Otras cuentas por cobrar al personal,group14,account.data_account_type_receivable,pe_chart_template,True
chart1421,1421,Accionistas (o socios) - Suscripciones por cobrar a socios o accionistas,group14,account.data_account_type_receivable,pe_chart_template,True
chart1422,1422,Accionistas (o socios) - Préstamos,group14,account.data_account_type_receivable,pe_chart_template,True
chart1431,1431,Directores - Préstamos,group14,account.data_account_type_receivable,pe_chart_template,True
chart1432,1432,Directores - Adelanto de dietas,group14,account.data_account_type_receivable,pe_chart_template,True
chart1433,1433,Directores - Entregas a rendir cuenta,group14,account.data_account_type_receivable,pe_chart_template,True
chart149,149,Diversas,group14,account.data_account_type_receivable,pe_chart_template,True
chart1611,1611,Préstamos - Con garantía,group16,account.data_account_type_receivable,pe_chart_template,True
chart1612,1612,Préstamos - Sin garantía,group16,account.data_account_type_receivable,pe_chart_template,True
chart1621,1621,Reclamaciones a terceros - Compañías aseguradoras,group16,account.data_account_type_receivable,pe_chart_template,True
chart1622,1622,Reclamaciones a terceros - Transportadoras,group16,account.data_account_type_receivable,pe_chart_template,True
chart1623,1623,Reclamaciones a terceros - Servicios públicos,group16,account.data_account_type_receivable,pe_chart_template,True
chart1624,1624,Reclamaciones a terceros - Tributos,group16,account.data_account_type_receivable,pe_chart_template,True
chart1629,1629,Reclamaciones a terceros - Otras,group16,account.data_account_type_receivable,pe_chart_template,True
chart1631,1631,"Intereses, regalías y dividendos - Intereses",group16,account.data_account_type_receivable,pe_chart_template,True
chart1632,1632,"Intereses, regalías y dividendos - Regalías",group16,account.data_account_type_receivable,pe_chart_template,True
chart1633,1633,"Intereses, regalías y dividendos - Dividendos",group16,account.data_account_type_receivable,pe_chart_template,True
chart1641,1641,Depósitos otorgados en garantía - Préstamos de instituciones financieras,group16,account.data_account_type_receivable,pe_chart_template,True
chart1642,1642,Depósitos otorgados en garantía - Préstamos de instituciones no financieras,group16,account.data_account_type_receivable,pe_chart_template,True
chart1643,1643,Depósitos otorgados en garantía - Depósitos en garantía por alquileres,group16,account.data_account_type_receivable,pe_chart_template,True
chart1649,1649,Depósitos otorgados en garantía - Otros depósitos en garantía,group16,account.data_account_type_receivable,pe_chart_template,True
chart1651,1651,Venta de activo inmovilizado - Inversión mobiliaria,group16,account.data_account_type_receivable,pe_chart_template,True
chart1652,1652,Venta de activo inmovilizado - Propiedades de inversión,group16,account.data_account_type_receivable,pe_chart_template,True
chart1653,1653,"Venta de activo inmovilizado - Propiedad, planta y equipo",group16,account.data_account_type_receivable,pe_chart_template,True
chart1654,1654,Venta de activo inmovilizado - Intangibles,group16,account.data_account_type_receivable,pe_chart_template,True
chart1655,1655,Venta de activo inmovilizado - Activos biológicos,group16,account.data_account_type_receivable,pe_chart_template,True
chart1659,1659,Venta de activo inmovilizado - Otros activos inmovilizados,group16,account.data_account_type_receivable,pe_chart_template,True
chart16611,16611,Activos por instrumentos financieros - Instrumentos financieros primarios - Costo,group16,account.data_account_type_receivable,pe_chart_template,True
chart16612,16612,Activos por instrumentos financieros - Instrumentos financieros primarios - Valor razonable,group16,account.data_account_type_receivable,pe_chart_template,True
chart16621,16621,Activos por instrumentos financieros - Instrumentos financieros derivados - Costo,group16,account.data_account_type_receivable,pe_chart_template,True
chart16622,16622,Activos por instrumentos financieros - Instrumentos financieros derivados - Valor razonable,group16,account.data_account_type_receivable,pe_chart_template,True
chart1671,1671,Tributos por acreditar - Pagos a cuenta del impuesto a la renta,group16,account.data_account_type_receivable,pe_chart_template,True
chart1672,1672,Tributos por acreditar - Pagos a cuenta de ITAN,group16,account.data_account_type_receivable,pe_chart_template,True
chart1673,1673,Tributos por acreditar - IGV por acreditar en compras,group16,account.data_account_type_receivable,pe_chart_template,True
chart1674,1674,Tributos por acreditar - IGV por acreditar no domiciliados,group16,account.data_account_type_receivable,pe_chart_template,True
chart1675,1675,Tributos por acreditar - Obras por impuestos,group16,account.data_account_type_receivable,pe_chart_template,True
chart1691,1691,Otras cuentas por cobrar diversas - Entregas a rendir cuenta a terceros,group16,account.data_account_type_receivable,pe_chart_template,True
chart1699,1699,Otras cuentas por cobrar diversas - Otras cuentas por cobrar diversas,group16,account.data_account_type_receivable,pe_chart_template,True
chart1711,1711,Préstamos - Con garantía,group17,account.data_account_type_receivable,pe_chart_template,True
chart1712,1712,Préstamos - Sin garantía,group17,account.data_account_type_receivable,pe_chart_template,True
chart1731,1731,"Intereses, regalías y dividendos - Intereses",group17,account.data_account_type_receivable,pe_chart_template,True
chart1732,1732,"Intereses, regalías y dividendos - Regalías",group17,account.data_account_type_receivable,pe_chart_template,True
chart1733,1733,"Intereses, regalías y dividendos - Dividendos",group17,account.data_account_type_receivable,pe_chart_template,True
chart1741,1741,Depósitos otorgados en garantía - Préstamos de instituciones financieras,group17,account.data_account_type_receivable,pe_chart_template,True
chart1742,1742,Depósitos otorgados en garantía - Préstamos de instituciones no financieras,group17,account.data_account_type_receivable,pe_chart_template,True
chart1743,1743,Depósitos otorgados en garantía - Depósitos en garantía por alquileres,group17,account.data_account_type_receivable,pe_chart_template,True
chart1749,1749,Depósitos otorgados en garantía - Otros depósitos en garantía,group17,account.data_account_type_receivable,pe_chart_template,True
chart1751,1751,Venta de activo inmovilizado - Inversión mobiliaria,group17,account.data_account_type_receivable,pe_chart_template,True
chart1752,1752,Venta de activo inmovilizado - Propiedades de inversión,group17,account.data_account_type_receivable,pe_chart_template,True
chart1753,1753,"Venta de activo inmovilizado - Propiedad, planta y equipo",group17,account.data_account_type_receivable,pe_chart_template,True
chart1754,1754,Venta de activo inmovilizado - Intangibles,group17,account.data_account_type_receivable,pe_chart_template,True
chart1755,1755,Venta de activo inmovilizado - Activos biológicos,group17,account.data_account_type_receivable,pe_chart_template,True
chart1759,1759,Venta de activo inmovilizado - Otros activos inmovilizados,group17,account.data_account_type_receivable,pe_chart_template,True
chart17611,17611,Activos por instrumentos financieros - Instrumentos financieros primarios - Costo,group17,account.data_account_type_receivable,pe_chart_template,True
chart17612,17612,Activos por instrumentos financieros - Instrumentos financieros primarios - Valor razonable,group17,account.data_account_type_receivable,pe_chart_template,True
chart17621,17621,Activos por instrumentos financieros - Instrumentos financieros derivados - Costo,group17,account.data_account_type_receivable,pe_chart_template,True
chart17622,17622,Activos por instrumentos financieros - Instrumentos financieros derivados - Valor razonable,group17,account.data_account_type_receivable,pe_chart_template,True
chart179,179,Otras cuentas por cobrar diversas,group17,account.data_account_type_receivable,pe_chart_template,True
chart181,181,Costos financieros,group18,account.data_account_type_prepayments,pe_chart_template,False
chart182,182,Seguros,group18,account.data_account_type_prepayments,pe_chart_template,False
chart183,183,Alquileres,group18,account.data_account_type_prepayments,pe_chart_template,False
chart184,184,Primas pagadas por opciones,group18,account.data_account_type_prepayments,pe_chart_template,False
chart185,185,Mantenimiento de activos inmovilizados,group18,account.data_account_type_prepayments,pe_chart_template,False
chart189,189,Otros gastos contratados por anticipado,group18,account.data_account_type_prepayments,pe_chart_template,False
chart1911,1911,"Cuentas por cobrar comerciales – Terceros - Facturas, boletas y otros comprobantes por cobrar",group19,account.data_account_type_receivable,pe_chart_template,True
chart1913,1913,Cuentas por cobrar comerciales – Terceros - Letras por cobrar,group19,account.data_account_type_receivable,pe_chart_template,True
chart1921,1921,"Cuentas por cobrar comerciales – Relacionadas - Facturas, boletas y otros comprobantes por cobrar",group19,account.data_account_type_receivable,pe_chart_template,True
chart1923,1923,Cuentas por cobrar comerciales – Relacionadas - Letras por cobrar,group19,account.data_account_type_receivable,pe_chart_template,True
chart1931,1931,"Cuentas por cobrar al personal, a los accionistas (socios) y directores - Personal",group19,account.data_account_type_receivable,pe_chart_template,True
chart1932,1932,"Cuentas por cobrar al personal, a los accionistas (socios) y directores - Accionistas (o socios)",group19,account.data_account_type_receivable,pe_chart_template,True
chart1933,1933,"Cuentas por cobrar al personal, a los accionistas (socios) y directores - Directores",group19,account.data_account_type_receivable,pe_chart_template,True
chart1939,1939,"Cuentas por cobrar al personal, a los accionistas (socios) y directores - Diversas",group19,account.data_account_type_receivable,pe_chart_template,True
chart1941,1941,Cuentas por cobrar diversas – Terceros - Préstamos,group19,account.data_account_type_receivable,pe_chart_template,True
chart1942,1942,Cuentas por cobrar diversas – Terceros - Reclamaciones a terceros,group19,account.data_account_type_receivable,pe_chart_template,True
chart1943,1943,"Cuentas por cobrar diversas – Terceros - Intereses, regalías y dividendos",group19,account.data_account_type_receivable,pe_chart_template,True
chart1944,1944,Cuentas por cobrar diversas – Terceros - Depósitos otorgados en garantía,group19,account.data_account_type_receivable,pe_chart_template,True
chart1945,1945,Cuentas por cobrar diversas – Terceros - Venta de activo inmovilizado,group19,account.data_account_type_receivable,pe_chart_template,True
chart1946,1946,Cuentas por cobrar diversas – Terceros - Activos por instrumentos financieros,group19,account.data_account_type_receivable,pe_chart_template,True
chart1949,1949,Cuentas por cobrar diversas – Terceros - Otras cuentas por cobrar diversas,group19,account.data_account_type_receivable,pe_chart_template,True
chart1951,1951,Cuentas por cobrar diversas – Relacionadas - Préstamos,group19,account.data_account_type_receivable,pe_chart_template,True
chart1953,1953,"Cuentas por cobrar diversas – Relacionadas - Intereses, regalías y dividendos",group19,account.data_account_type_receivable,pe_chart_template,True
chart1954,1954,Cuentas por cobrar diversas – Relacionadas - Depósitos otorgados en garantía,group19,account.data_account_type_receivable,pe_chart_template,True
chart1955,1955,Cuentas por cobrar diversas – Relacionadas - Venta de activo inmovilizado,group19,account.data_account_type_receivable,pe_chart_template,True
chart1956,1956,Cuentas por cobrar diversas – Relacionadas - Activos por instrumentos financieros,group19,account.data_account_type_receivable,pe_chart_template,True
chart1959,1959,Cuentas por cobrar diversas – Relacionadas - Otras cuentas por cobrar diversas,group19,account.data_account_type_receivable,pe_chart_template,True
chart20111,20111,Mercaderías - Mercaderías - Costo,group20,account.data_account_type_current_assets,pe_chart_template,False
chart20114,20114,Mercaderías - Mercaderías - Valor razonable,group20,account.data_account_type_current_assets,pe_chart_template,False
chart21111,21111,Productos terminados - Productos terminados - Costo,group21,account.data_account_type_current_assets,pe_chart_template,False
chart21113,21113,Productos terminados - Productos terminados - Costo de financiación,group21,account.data_account_type_current_assets,pe_chart_template,False
chart21114,21114,Productos terminados - Productos terminados - Valor razonable,group21,account.data_account_type_current_assets,pe_chart_template,False
chart21511,21511,Inventario de servicios terminados - Servicios terminados - Costo,group21,account.data_account_type_current_assets,pe_chart_template,False
chart221,221,Subproductos,group22,account.data_account_type_current_assets,pe_chart_template,False
chart222,222,Desechos y desperdicios,group22,account.data_account_type_current_assets,pe_chart_template,False
chart23111,23111,Productos en proceso - Productos en proceso - Costo,group23,account.data_account_type_current_assets,pe_chart_template,False
chart23113,23113,Productos en proceso - Productos en proceso - Costo de financiación,group23,account.data_account_type_current_assets,pe_chart_template,False
chart23511,23511,Inventario de servicios en proceso - Servicios en proceso - Costo,group23,account.data_account_type_current_assets,pe_chart_template,False
chart24111,24111,Materias primas - Materias primas - Costo,group24,account.data_account_type_current_assets,pe_chart_template,False
chart24114,24114,Materias primas - Materias primas - Valor razonable,group24,account.data_account_type_current_assets,pe_chart_template,False
chart251,251,Materiales auxiliares,group25,account.data_account_type_current_assets,pe_chart_template,False
chart2521,2521,Suministros - Combustibles,group25,account.data_account_type_current_assets,pe_chart_template,False
chart2522,2522,Suministros - Lubricantes,group25,account.data_account_type_current_assets,pe_chart_template,False
chart2523,2523,Suministros - Energía,group25,account.data_account_type_current_assets,pe_chart_template,False
chart2524,2524,Suministros - Otros suministros,group25,account.data_account_type_current_assets,pe_chart_template,False
chart253,253,Repuestos,group25,account.data_account_type_current_assets,pe_chart_template,False
chart261,261,Envases,group26,account.data_account_type_current_assets,pe_chart_template,False
chart262,262,Embalajes,group26,account.data_account_type_current_assets,pe_chart_template,False
chart27111,27111,Propiedades de inversión - Terrenos - Costo,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27112,27112,Propiedades de inversión - Terrenos - Revaluación,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27114,27114,Propiedades de inversión - Terrenos - Valor razonable,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27121,27121,Propiedades de inversión - Edificaciones - Costo,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27122,27122,Propiedades de inversión - Edificaciones - Revaluación,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27123,27123,Propiedades de inversión - Edificaciones - Costos de financiación,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27124,27124,Propiedades de inversión - Edificaciones - Valor razonable,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27201,27201,"Propiedad, planta y equipo - Planta productora en producción - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27202,27202,"Propiedad, planta y equipo - Planta productora en producción - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27203,27203,"Propiedad, planta y equipo - Planta productora en producción - Costo de financiación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27204,27204,"Propiedad, planta y equipo - Planta productora en producción - Valor razonable",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27211,27211,"Propiedad, planta y equipo - Planta productora en desarrollo - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27212,27212,"Propiedad, planta y equipo - Planta productora en desarrollo - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27213,27213,"Propiedad, planta y equipo - Planta productora en desarrollo - Costo de financiación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27214,27214,"Propiedad, planta y equipo - Planta productora en desarrollo - Valor razonable",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27221,27221,"Propiedad, planta y equipo - Terrenos - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27222,27222,"Propiedad, planta y equipo - Terrenos - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27231,27231,"Propiedad, planta y equipo - Edificaciones - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27232,27232,"Propiedad, planta y equipo - Edificaciones - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27233,27233,"Propiedad, planta y equipo - Edificaciones - Costo de financiación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27241,27241,"Propiedad, planta y equipo - Maquinarias y equipos de explotación - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27242,27242,"Propiedad, planta y equipo - Maquinarias y equipos de explotación - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27243,27243,"Propiedad, planta y equipo - Maquinarias y equipos de explotación - Costo de financiación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27251,27251,"Propiedad, planta y equipo - Unidades de transporte - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27252,27252,"Propiedad, planta y equipo - Unidades de transporte - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27261,27261,"Propiedad, planta y equipo - Muebles y enseres - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27262,27262,"Propiedad, planta y equipo - Muebles y enseres - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27271,27271,"Propiedad, planta y equipo - Equipos diversos - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27272,27272,"Propiedad, planta y equipo - Equipos diversos - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27281,27281,"Propiedad, planta y equipo - Herramientas y unidades de reemplazo - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27282,27282,"Propiedad, planta y equipo - Herramientas y unidades de reemplazo - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27291,27291,"Propiedad, planta y equipo - Obras en curso - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27292,27292,"Propiedad, planta y equipo - Obras en curso - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27311,27311,"Intangibles - Concesiones, licencias y derechos - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27312,27312,"Intangibles - Concesiones, licencias y derechos - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27321,27321,Intangibles - Patentes y propiedad industrial - Costo,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27322,27322,Intangibles - Patentes y propiedad industrial - Revaluación,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27331,27331,Intangibles - Programas de computadora (software) - Costo,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27332,27332,Intangibles - Programas de computadora (software) - Revaluación,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27341,27341,Intangibles - Costos de exploración y desarrollo - Costo,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27342,27342,Intangibles - Costos de exploración y desarrollo - Revaluación,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27351,27351,"Intangibles - Fórmulas, diseños y prototipos - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27352,27352,"Intangibles - Fórmulas, diseños y prototipos - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27391,27391,Intangibles - Otros activos intangibles - Costo,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27392,27392,Intangibles - Otros activos intangibles - Revaluación,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27411,27411,Activos biológicos - Activos biológicos en producción - Costo,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27413,27413,Activos biológicos - Activos biológicos en producción - Costos de financiación,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27414,27414,Activos biológicos - Activos biológicos en producción - Valor razonable,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27421,27421,Activos biológicos - Activos biológicos en desarrollo - Costo,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27423,27423,Activos biológicos - Activos biológicos en desarrollo - Costos de financiación,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27424,27424,Activos biológicos - Activos biológicos en desarrollo - Valor razonable,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27521,27521,Depreciación acumulada – Propiedades de inversión - Edificaciones - Costo,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27522,27522,Depreciación acumulada – Propiedades de inversión - Edificaciones - Revaluación,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27523,27523,Depreciación acumulada – Propiedades de inversión - Edificaciones - Costo de financiación,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27601,27601,"Depreciación acumulada – Propiedad, planta y equipo - Planta productora en producción - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27602,27602,"Depreciación acumulada – Propiedad, planta y equipo - Planta productora en producción - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27603,27603,"Depreciación acumulada – Propiedad, planta y equipo - Planta productora en producción - Costo de financiación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27604,27604,"Depreciación acumulada – Propiedad, planta y equipo - Planta productora en producción - Valor razonable",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27621,27621,"Depreciación acumulada – Propiedad, planta y equipo - Edificaciones - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27622,27622,"Depreciación acumulada – Propiedad, planta y equipo - Edificaciones - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27623,27623,"Depreciación acumulada – Propiedad, planta y equipo - Edificaciones - Costo de financiación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27631,27631,"Depreciación acumulada – Propiedad, planta y equipo - Maquinarias y equipo de explotación - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27632,27632,"Depreciación acumulada – Propiedad, planta y equipo - Maquinarias y equipo de explotación - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27633,27633,"Depreciación acumulada – Propiedad, planta y equipo - Maquinarias y equipo de explotación - Costo de financiación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27641,27641,"Depreciación acumulada – Propiedad, planta y equipo - Unidades de transporte - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27642,27642,"Depreciación acumulada – Propiedad, planta y equipo - Unidades de transporte - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27651,27651,"Depreciación acumulada – Propiedad, planta y equipo - Muebles y enseres - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27652,27652,"Depreciación acumulada – Propiedad, planta y equipo - Muebles y enseres - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27661,27661,"Depreciación acumulada – Propiedad, planta y equipo - Equipos diversos - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27662,27662,"Depreciación acumulada – Propiedad, planta y equipo - Equipos diversos - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27671,27671,"Depreciación acumulada – Propiedad, planta y equipo - Herramientas y unidades de reemplazo - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27672,27672,"Depreciación acumulada – Propiedad, planta y equipo - Herramientas y unidades de reemplazo - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27711,27711,"Amortización acumulada – Intangibles - Concesiones, licencias y derechos - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27712,27712,"Amortización acumulada – Intangibles - Concesiones, licencias y derechos - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27721,27721,Amortización acumulada – Intangibles - Patentes y propiedad industrial - Costo,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27722,27722,Amortización acumulada – Intangibles - Patentes y propiedad industrial - Revaluación,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27731,27731,Amortización acumulada – Intangibles - Programas de computadora (software) - Costo,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27732,27732,Amortización acumulada – Intangibles - Programas de computadora (software) - Revaluación,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27741,27741,Amortización acumulada – Intangibles - Costos de exploración y desarrollo - Costo,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27742,27742,Amortización acumulada – Intangibles - Costos de exploración y desarrollo - Revaluación,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27751,27751,"Amortización acumulada – Intangibles - Fórmulas, diseños y prototipos - Costo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27752,27752,"Amortización acumulada – Intangibles - Fórmulas, diseños y prototipos - Revaluación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27791,27791,Amortización acumulada – Intangibles - Otros activos intangibles - Costo,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27792,27792,Amortización acumulada – Intangibles - Otros activos intangibles - Revaluación,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27811,27811,Depreciación acumulada – Activos biológicos - Activos biológicos en producción - Costo,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27813,27813,Depreciación acumulada – Activos biológicos - Activos biológicos en producción - Costo de financiación,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27821,27821,Depreciación acumulada – Activos biológicos - Activos biológicos en desarrollo - Costo,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27823,27823,Depreciación acumulada – Activos biológicos - Activos biológicos en desarrollo - Costo de financiación,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27910,27910,Desvalorización acumulada - Propiedad de inversión - Planta productora en producción,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27911,27911,Desvalorización acumulada - Propiedad de inversión - Planta productora en desarrollo,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27912,27912,Desvalorización acumulada - Propiedad de inversión - Terrenos,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27913,27913,Desvalorización acumulada - Propiedad de inversión - Edificaciones,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27930,27930,"Desvalorización acumulada - Propiedad, planta y equipo - Plantas productoras en producción",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27931,27931,"Desvalorización acumulada - Propiedad, planta y equipo - Planta productora en desarrollo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27932,27932,"Desvalorización acumulada - Propiedad, planta y equipo - Terrenos",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27933,27933,"Desvalorización acumulada - Propiedad, planta y equipo - Edificaciones",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27934,27934,"Desvalorización acumulada - Propiedad, planta y equipo - Maquinarias y equipos de explotación",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27935,27935,"Desvalorización acumulada - Propiedad, planta y equipo - Unidades de transporte",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27936,27936,"Desvalorización acumulada - Propiedad, planta y equipo - Muebles y enseres",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27937,27937,"Desvalorización acumulada - Propiedad, planta y equipo - Equipos diversos",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27938,27938,"Desvalorización acumulada - Propiedad, planta y equipo - Herramientas y unidades de reemplazo",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27941,27941,"Desvalorización acumulada - Intangibles - Concesiones, licencias y otros derechos",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27942,27942,Desvalorización acumulada - Intangibles - Patentes y propiedad industrial,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27943,27943,Desvalorización acumulada - Intangibles - Programas de computadora (software),group27,account.data_account_type_current_assets,pe_chart_template,False
chart27944,27944,Desvalorización acumulada - Intangibles - Costos de exploración y desarrollo,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27945,27945,"Desvalorización acumulada - Intangibles - Fórmulas, diseños y prototipos",group27,account.data_account_type_current_assets,pe_chart_template,False
chart27949,27949,Desvalorización acumulada - Intangibles - Otros activos intangibles,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27951,27951,Desvalorización acumulada - Activos biológicos - Activos biológicos en producción,group27,account.data_account_type_current_assets,pe_chart_template,False
chart27952,27952,Desvalorización acumulada - Activos biológicos - Activos biológicos en desarrollo,group27,account.data_account_type_current_assets,pe_chart_template,False
chart281,281,Mercaderías,group28,account.data_account_type_current_assets,pe_chart_template,False
chart284,284,Materias primas,group28,account.data_account_type_current_assets,pe_chart_template,False
chart285,285,"Materiales auxiliares, suministros y repuestos",group28,account.data_account_type_current_assets,pe_chart_template,False
chart286,286,Envases y embalajes,group28,account.data_account_type_current_assets,pe_chart_template,False
chart29111,29111,Mercaderías - Mercaderías - Costo,group29,account.data_account_type_current_assets,pe_chart_template,False
chart29211,29211,Productos terminados - Productos terminados - Costo,group29,account.data_account_type_current_assets,pe_chart_template,False
chart29213,29213,Productos terminados - Productos terminados - Costo de financiación,group29,account.data_account_type_current_assets,pe_chart_template,False
chart29251,29251,Productos terminados - Inventario de servicios terminados - Costo,group29,account.data_account_type_current_assets,pe_chart_template,False
chart2931,2931,"Subproductos, desechos y desperdicios - Subproductos",group29,account.data_account_type_current_assets,pe_chart_template,False
chart2932,2932,"Subproductos, desechos y desperdicios - Desechos y desperdicios",group29,account.data_account_type_current_assets,pe_chart_template,False
chart29411,29411,Productos en proceso - Productos en proceso - Costo,group29,account.data_account_type_current_assets,pe_chart_template,False
chart29413,29413,Productos en proceso - Productos en proceso - Costo de financiación,group29,account.data_account_type_current_assets,pe_chart_template,False
chart2945,2945,Productos en proceso - Inventario de servicios en proceso,group29,account.data_account_type_current_assets,pe_chart_template,False
chart29511,29511,Materias primas - Materias primas - Costo,group29,account.data_account_type_current_assets,pe_chart_template,False
chart2961,2961,"Materiales auxiliares, suministros y repuestos - Materiales auxiliares",group29,account.data_account_type_current_assets,pe_chart_template,False
chart2962,2962,"Materiales auxiliares, suministros y repuestos - Suministros",group29,account.data_account_type_current_assets,pe_chart_template,False
chart2963,2963,"Materiales auxiliares, suministros y repuestos - Repuestos",group29,account.data_account_type_current_assets,pe_chart_template,False
chart2971,2971,Envases y embalajes - Envases,group29,account.data_account_type_current_assets,pe_chart_template,False
chart2972,2972,Envases y embalajes - Embalajes,group29,account.data_account_type_current_assets,pe_chart_template,False
chart2981,2981,Existencias por recibir - Mercaderías,group29,account.data_account_type_current_assets,pe_chart_template,False
chart2982,2982,Existencias por recibir - Materias primas,group29,account.data_account_type_current_assets,pe_chart_template,False
chart2983,2983,"Existencias por recibir - Materiales auxiliares, suministros y repuestos",group29,account.data_account_type_current_assets,pe_chart_template,False
chart2984,2984,Existencias por recibir - Envases y embalajes,group29,account.data_account_type_current_assets,pe_chart_template,False
chart30111,30111,Inversiones a ser mantenidas hasta el vencimiento - Instrumentos financieros representativos de deuda - Costo,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30114,30114,Inversiones a ser mantenidas hasta el vencimiento - Instrumentos financieros representativos de deuda - Valor razonable,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart3021,3021,Instrumentos financieros representativos de derecho patrimonial - Certificados de suscripción preferente,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30221,30221,Instrumentos financieros representativos de derecho patrimonial - Acciones representativas de capital social – Comunes - Costo,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30224,30224,Instrumentos financieros representativos de derecho patrimonial - Acciones representativas de capital social – Comunes - Valor razonable,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30225,30225,Instrumentos financieros representativos de derecho patrimonial - Acciones representativas de capital social – Comunes - Participación patrimonial,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30231,30231,Instrumentos financieros representativos de derecho patrimonial - Acciones representativas de capital social – Preferentes - Costo,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30234,30234,Instrumentos financieros representativos de derecho patrimonial - Acciones representativas de capital social – Preferentes - Valor razonable,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30235,30235,Instrumentos financieros representativos de derecho patrimonial - Acciones representativas de capital social – Preferentes - Participación patrimonial,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30241,30241,Instrumentos financieros representativos de derecho patrimonial - Acciones de inversión - Costo,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30244,30244,Instrumentos financieros representativos de derecho patrimonial - Acciones de inversión - Valor razonable,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30245,30245,Instrumentos financieros representativos de derecho patrimonial - Acciones de inversión - Participación patrimonial,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30281,30281,Instrumentos financieros representativos de derecho patrimonial - Otros títulos representativos de patrimonio - Costo,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30284,30284,Instrumentos financieros representativos de derecho patrimonial - Otros títulos representativos de patrimonio - Valor razonable,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30285,30285,Instrumentos financieros representativos de derecho patrimonial - Otros títulos representativos de patrimonio - Participación patrimonial,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30311,30311,Certificados de participación en fondos - Cuotas - Fondos de inversión - Costo,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30314,30314,Certificados de participación en fondos - Cuotas - Fondos de inversión - Valor razonable,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30321,30321,Certificados de participación en fondos - Cuotas - Fondos mutuos - Costo,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30324,30324,Certificados de participación en fondos - Cuotas - Fondos mutuos - Valor razonable,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30411,30411,Participaciones en acuerdos conjuntos - Operaciones conjuntas - Costo,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30414,30414,Participaciones en acuerdos conjuntos - Operaciones conjuntas - Valor razonable,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30415,30415,Participaciones en acuerdos conjuntos - Operaciones conjuntas - Participación patrimonial,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30421,30421,Participaciones en acuerdos conjuntos - Negocios conjuntos - Costo,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30424,30424,Participaciones en acuerdos conjuntos - Negocios conjuntos - Valor razonable,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30425,30425,Participaciones en acuerdos conjuntos - Negocios conjuntos - Participación patrimonial,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30811,30811,Inversiones mobiliarias – Acuerdos de compra - Instrumentos financieros representativos de deuda – Acuerdo de compra - Costo,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30814,30814,Inversiones mobiliarias – Acuerdos de compra - Instrumentos financieros representativos de deuda – Acuerdo de compra - Valor razonable,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30821,30821,Inversiones mobiliarias – Acuerdos de compra - Instrumentos financieros representativos de derecho patrimonial – Acuerdo de compra - Costo,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart30824,30824,Inversiones mobiliarias – Acuerdos de compra - Instrumentos financieros representativos de derecho patrimonial – Acuerdo de compra - Valor razonable,group30,account.data_account_type_fixed_assets,pe_chart_template,False
chart31111,31111,Terrenos - Urbanos - Costo,group31,account.data_account_type_fixed_assets,pe_chart_template,False
chart31112,31112,Terrenos - Urbanos - Revaluación,group31,account.data_account_type_fixed_assets,pe_chart_template,False
chart31114,31114,Terrenos - Urbanos - Valor razonable,group31,account.data_account_type_fixed_assets,pe_chart_template,False
chart31121,31121,Terrenos - Rurales - Costo,group31,account.data_account_type_fixed_assets,pe_chart_template,False
chart31122,31122,Terrenos - Rurales - Revaluación,group31,account.data_account_type_fixed_assets,pe_chart_template,False
chart31124,31124,Terrenos - Rurales - Valor razonable,group31,account.data_account_type_fixed_assets,pe_chart_template,False
chart31211,31211,Edificaciones - Edificaciones - Costo,group31,account.data_account_type_fixed_assets,pe_chart_template,False
chart31212,31212,Edificaciones - Edificaciones - Revaluación,group31,account.data_account_type_fixed_assets,pe_chart_template,False
chart31213,31213,Edificaciones - Edificaciones - Costos de financiación,group31,account.data_account_type_fixed_assets,pe_chart_template,False
chart31214,31214,Edificaciones - Edificaciones - Valor razonable,group31,account.data_account_type_fixed_assets,pe_chart_template,False
chart31311,31311,Construcciones en curso - Edificaciones - Costo,group31,account.data_account_type_fixed_assets,pe_chart_template,False
chart31312,31312,Construcciones en curso - Edificaciones - Revaluación,group31,account.data_account_type_fixed_assets,pe_chart_template,False
chart31313,31313,Construcciones en curso - Edificaciones - Costos de financiación,group31,account.data_account_type_fixed_assets,pe_chart_template,False
chart31314,31314,Construcciones en curso - Edificaciones - Valor razonable,group31,account.data_account_type_fixed_assets,pe_chart_template,False
chart32111,32111,Propiedades de inversión - Arrendamiento financiero - Terrenos - Costo,group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32112,32112,Propiedades de inversión - Arrendamiento financiero - Terrenos - Revaluación,group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32114,32114,Propiedades de inversión - Arrendamiento financiero - Terrenos - Valor razonable,group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32121,32121,Propiedades de inversión - Arrendamiento financiero - Edificaciones - Costo,group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32122,32122,Propiedades de inversión - Arrendamiento financiero - Edificaciones - Revaluación,group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32123,32123,Propiedades de inversión - Arrendamiento financiero - Edificaciones - Costo de financiación,group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32124,32124,Propiedades de inversión - Arrendamiento financiero - Edificaciones - Valor razonable,group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32201,32201,"Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en producción - Costo",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32202,32202,"Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en producción - Revaluación",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32203,32203,"Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en producción - Costo de financiación",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32211,32211,"Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en desarrollo - Costo",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32212,32212,"Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en desarrollo - Revaluación",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32213,32213,"Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en desarrollo - Costo de financiación",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32221,32221,"Propiedad, planta y equipo - Arrendamiento financiero - Terrenos - Costo",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32222,32222,"Propiedad, planta y equipo - Arrendamiento financiero - Terrenos - Revaluación",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32231,32231,"Propiedad, planta y equipo - Arrendamiento financiero - Edificaciones - Costo",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32232,32232,"Propiedad, planta y equipo - Arrendamiento financiero - Edificaciones - Revaluación",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32233,32233,"Propiedad, planta y equipo - Arrendamiento financiero - Edificaciones - Costo de financiación",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32241,32241,"Propiedad, planta y equipo - Arrendamiento financiero - Maquinaria y equipo de explotación - Costo",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32242,32242,"Propiedad, planta y equipo - Arrendamiento financiero - Maquinaria y equipo de explotación - Revaluación",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32243,32243,"Propiedad, planta y equipo - Arrendamiento financiero - Maquinaria y equipo de explotación - Costo de financiación",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32251,32251,"Propiedad, planta y equipo - Arrendamiento financiero - Unidades de transporte - Costo",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32252,32252,"Propiedad, planta y equipo - Arrendamiento financiero - Unidades de transporte - Revaluación",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32261,32261,"Propiedad, planta y equipo - Arrendamiento financiero - Muebles y enseres - Costo",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32262,32262,"Propiedad, planta y equipo - Arrendamiento financiero - Muebles y enseres - Revaluación",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32271,32271,"Propiedad, planta y equipo - Arrendamiento financiero - Equipos diversos - Costo",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32272,32272,"Propiedad, planta y equipo - Arrendamiento financiero - Equipos diversos - Revaluación",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32281,32281,"Propiedad, planta y equipo - Arrendamiento financiero - Herramientas y unidades de reemplazo - Costo",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32282,32282,"Propiedad, planta y equipo - Arrendamiento financiero - Herramientas y unidades de reemplazo - Revaluación",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32301,32301,"Propiedad, planta y equipo - Arrendamiento operativo - Planta productora en producción - Costo",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32302,32302,"Propiedad, planta y equipo - Arrendamiento operativo - Planta productora en producción - Revaluación",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32321,32321,"Propiedad, planta y equipo - Arrendamiento operativo - Terrenos - Costo",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32331,32331,"Propiedad, planta y equipo - Arrendamiento operativo - Edificaciones - Costo",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32332,32332,"Propiedad, planta y equipo - Arrendamiento operativo - Edificaciones - Revaluación",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32341,32341,"Propiedad, planta y equipo - Arrendamiento operativo - Maquinaria y equipo de explotación - Costo",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32342,32342,"Propiedad, planta y equipo - Arrendamiento operativo - Maquinaria y equipo de explotación - Revaluación",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32351,32351,"Propiedad, planta y equipo - Arrendamiento operativo - Unidades de transporte - Costo",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32352,32352,"Propiedad, planta y equipo - Arrendamiento operativo - Unidades de transporte - Revaluación",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32361,32361,"Propiedad, planta y equipo - Arrendamiento operativo - Equipos diversos - Costo",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart32362,32362,"Propiedad, planta y equipo - Arrendamiento operativo - Equipos diversos - Revaluación",group32,account.data_account_type_fixed_assets,pe_chart_template,False
chart33011,33011,Planta productora - Planta productora en producción - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33012,33012,Planta productora - Planta productora en producción - Revaluación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33013,33013,Planta productora - Planta productora en producción - Costo de financiación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33014,33014,Planta productora - Planta productora en producción - Valor razonable,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33021,33021,Planta productora - Planta productora en desarrollo - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33022,33022,Planta productora - Planta productora en desarrollo - Revaluación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33023,33023,Planta productora - Planta productora en desarrollo - Costo de financiación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33024,33024,Planta productora - Planta productora en desarrollo - Valor razonable,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33111,33111,Terrenos - Terrenos - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33112,33112,Terrenos - Terrenos - Revaluación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33211,33211,Edificaciones - Edificaciones - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33212,33212,Edificaciones - Edificaciones - Revaluación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33213,33213,Edificaciones - Edificaciones - Costo de financiación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33241,33241,Edificaciones - Instalaciones - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33242,33242,Edificaciones - Instalaciones - Revaluación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33243,33243,Edificaciones - Instalaciones - Costo de financiación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33251,33251,Edificaciones - Mejoras en locales arrendados. - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33252,33252,Edificaciones - Mejoras en locales arrendados. - Revaluación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33253,33253,Edificaciones - Mejoras en locales arrendados. - Costo de Financiación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33311,33311,Maquinaria y equipo de explotación - Maquinaria y equipo de explotación - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33312,33312,Maquinaria y equipo de explotación - Maquinaria y equipo de explotación - Revaluación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33313,33313,Maquinaria y equipo de explotación - Maquinaria y equipo de explotación - Costo de financiación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33411,33411,Unidades de transporte - Vehículos motorizados - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33412,33412,Unidades de transporte - Vehículos motorizados - Revaluación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33421,33421,Unidades de transporte - Vehículos no motorizados - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33422,33422,Unidades de transporte - Vehículos no motorizados - Revaluación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33511,33511,Muebles y enseres - Muebles - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33512,33512,Muebles y enseres - Muebles - Revaluación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33521,33521,Muebles y enseres - Enseres - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33522,33522,Muebles y enseres - Enseres - Revaluación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33611,33611,Equipos diversos - Equipo para procesamiento de información - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33612,33612,Equipos diversos - Equipo para procesamiento de información - Revaluación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33621,33621,Equipos diversos - Equipo de comunicación - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33622,33622,Equipos diversos - Equipo de comunicación - Revaluación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33631,33631,Equipos diversos - Equipo de seguridad - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33632,33632,Equipos diversos - Equipo de seguridad - Revaluación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33641,33641,Equipos diversos - Equipo de medio ambiente - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33642,33642,Equipos diversos - Equipo de medio ambiente - Revaluación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33691,33691,Equipos diversos - Otros equipos - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33692,33692,Equipos diversos - Otros equipos - Revaluación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33711,33711,Herramientas y unidades de reemplazo - Herramientas - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33712,33712,Herramientas y unidades de reemplazo - Herramientas - Revaluación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33721,33721,Herramientas y unidades de reemplazo - Unidades de reemplazo - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33722,33722,Herramientas y unidades de reemplazo - Unidades de reemplazo - Revaluación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart3381,3381,Unidades por recibir - Maquinaria y equipo de explotación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart3382,3382,Unidades por recibir - Equipo de transporte,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart3383,3383,Unidades por recibir - Muebles y enseres,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart3386,3386,Unidades por recibir - Equipos diversos,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart3387,3387,Unidades por recibir - Herramientas y unidades de reemplazo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart3391,3391,Obras en curso - Adecuación de terrenos,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33921,33921,Obras en curso - Edificaciones en curso - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33922,33922,Obras en curso - Edificaciones en curso - Costo de financiación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33931,33931,Obras en curso - Maquinaria en montaje - Costo,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart33932,33932,Obras en curso - Maquinaria en montaje - Costo de financiación,group33,account.data_account_type_fixed_assets,pe_chart_template,False
chart34111,34111,"Concesiones, licencias y otros derechos - Derechos por concesiones - Costo",group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34112,34112,"Concesiones, licencias y otros derechos - Derechos por concesiones - Revaluación",group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34121,34121,"Concesiones, licencias y otros derechos - Licencias - Costo",group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34122,34122,"Concesiones, licencias y otros derechos - Licencias - Revaluación",group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34191,34191,"Concesiones, licencias y otros derechos - Otros derechos - Costo",group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34192,34192,"Concesiones, licencias y otros derechos - Otros derechos - Revaluación",group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34211,34211,Patentes y propiedad industrial - Patentes - Costo,group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34212,34212,Patentes y propiedad industrial - Patentes - Revaluación,group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34221,34221,Patentes y propiedad industrial - Marcas - Costo,group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34222,34222,Patentes y propiedad industrial - Marcas - Revaluación,group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34311,34311,Programas de computadora (software) - Aplicaciones informáticas - Costo,group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34312,34312,Programas de computadora (software) - Aplicaciones informáticas - Revaluación,group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34411,34411,Costos de exploración y desarrollo - Costos de exploración - Costo,group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34412,34412,Costos de exploración y desarrollo - Costos de exploración - Revaluación,group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34413,34413,Costos de exploración y desarrollo - Costos de exploración - Costo de financiación,group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34421,34421,Costos de exploración y desarrollo - Costos de desarrollo - Costo,group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34422,34422,Costos de exploración y desarrollo - Costos de desarrollo - Revaluación,group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34423,34423,Costos de exploración y desarrollo - Costos de desarrollo - Costo de financiación,group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34511,34511,"Fórmulas, diseños y prototipos - Fórmulas - Costo",group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34512,34512,"Fórmulas, diseños y prototipos - Fórmulas - Revaluación",group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34521,34521,"Fórmulas, diseños y prototipos - Diseños y prototipos - Costo",group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34522,34522,"Fórmulas, diseños y prototipos - Diseños y prototipos - Revaluación",group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart3471,3471,Plusvalía mercantil - Plusvalía mercantil,group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34911,34911,Otros activos intangibles - Otros activos intangibles - Costo,group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart34912,34912,Otros activos intangibles - Otros activos intangibles - Revaluación,group34,account.data_account_type_fixed_assets,pe_chart_template,False
chart35111,35111,Activos biológicos en producción - De origen animal - Costo,group35,account.data_account_type_fixed_assets,pe_chart_template,False
chart35113,35113,Activos biológicos en producción - De origen animal - Costo de financiación,group35,account.data_account_type_fixed_assets,pe_chart_template,False
chart35114,35114,Activos biológicos en producción - De origen animal - Valor razonable,group35,account.data_account_type_fixed_assets,pe_chart_template,False
chart35121,35121,Activos biológicos en producción - De origen vegetal - Costo,group35,account.data_account_type_fixed_assets,pe_chart_template,False
chart35123,35123,Activos biológicos en producción - De origen vegetal - Costo de financiación,group35,account.data_account_type_fixed_assets,pe_chart_template,False
chart35124,35124,Activos biológicos en producción - De origen vegetal - Valor razonable,group35,account.data_account_type_fixed_assets,pe_chart_template,False
chart35211,35211,Activos biológicos en desarrollo - De origen animal - Costo,group35,account.data_account_type_fixed_assets,pe_chart_template,False
chart35213,35213,Activos biológicos en desarrollo - De origen animal - Costo de financiación,group35,account.data_account_type_fixed_assets,pe_chart_template,False
chart35214,35214,Activos biológicos en desarrollo - De origen animal - Valor razonable,group35,account.data_account_type_fixed_assets,pe_chart_template,False
chart35221,35221,Activos biológicos en desarrollo - De origen vegetal - Costo,group35,account.data_account_type_fixed_assets,pe_chart_template,False
chart35223,35223,Activos biológicos en desarrollo - De origen vegetal - Costo de financiación,group35,account.data_account_type_fixed_assets,pe_chart_template,False
chart35224,35224,Activos biológicos en desarrollo - De origen vegetal - Valor razonable,group35,account.data_account_type_fixed_assets,pe_chart_template,False
chart36111,36111,Terrenos - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36112,36112,Terrenos - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36121,36121,Edificaciones - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36122,36122,Edificaciones - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36123,36123,Edificaciones - Costo de financiación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36131,36131,Construcciones en curso - edificaciones - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36132,36132,Construcciones en curso - edificaciones - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36133,36133,Construcciones en curso - edificaciones - Costo de financiación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36211,36211,Arrendamiento financiero - Terrenos - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36212,36212,Arrendamiento financiero - Terrenos - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36221,36221,Arrendamiento financiero - Edificaciones - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36222,36222,Arrendamiento financiero - Edificaciones - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36223,36223,Arrendamiento financiero - Edificaciones - Costo de financiación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36311,36311,Arrendamiento financiero - Terrenos - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36312,36312,Arrendamiento financiero - Terrenos - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36321,36321,Arrendamiento financiero - Edificaciones - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36322,36322,Arrendamiento financiero - Edificaciones - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36323,36323,Arrendamiento financiero - Edificaciones - Costo de financiación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36331,36331,Arrendamiento financiero - Maquinaria y equipo de explotación - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36332,36332,Arrendamiento financiero - Maquinaria y equipo de explotación - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36333,36333,Arrendamiento financiero - Maquinaria y equipo de explotación - Costo de financiación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36341,36341,Arrendamiento financiero - Unidades de transporte - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36342,36342,Arrendamiento financiero - Unidades de transporte - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36351,36351,Arrendamiento financiero - Muebles y enseres - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36352,36352,Arrendamiento financiero - Muebles y enseres - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36361,36361,Arrendamiento financiero - Equipos diversos - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36362,36362,Arrendamiento financiero - Equipos diversos - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36401,36401,Planta productora en producción - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36402,36402,Planta productora en producción - Planta productora en producción - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36403,36403,Planta productora en producción - Planta productora en producción - Costo de financiación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36405,36405,Planta productora en producción - Planta productora en desarrollo - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36406,36406,Planta productora en producción - Planta productora en desarrollo - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36407,36407,Planta productora en producción - Planta productora en desarrollo - Costo de financiación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36408,36408,Planta productora en producción - Planta productora en desarrollo - Valor razonable,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36411,36411,Terrenos - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36412,36412,Terrenos - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36421,36421,Edificaciones - Edificaciones - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36422,36422,Edificaciones - Edificaciones - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36423,36423,Edificaciones - Edificaciones - Costo de financiación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36424,36424,Edificaciones - Instalaciones - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36425,36425,Edificaciones - Instalaciones - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36426,36426,Edificaciones - Instalaciones - Costo de financiación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36427,36427,Edificaciones - Mejoras en locales arrendados - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36428,36428,Edificaciones - Mejoras en locales arrendados - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36429,36429,Edificaciones - Mejoras en locales arrendados - Costo de financiación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36431,36431,Maquinaria y equipo de explotación - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36432,36432,Maquinaria y equipo de explotación - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36433,36433,Maquinaria y equipo de explotación - Costo de financiación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36441,36441,Unidades de transporte - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36442,36442,Unidades de transporte - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36451,36451,Muebles y enseres - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36452,36452,Muebles y enseres - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36461,36461,Equipos diversos - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36462,36462,Equipos diversos - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36471,36471,Herramientas y unidades de reemplazo - Herramientas - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36491,36491,Obras en curso - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36492,36492,Obras en curso - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36511,36511,"Concesiones, licencias y otros derechos - Costo",group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36512,36512,"Concesiones, licencias y otros derechos - Revaluación",group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36521,36521,Patentes y propiedad industrial - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36522,36522,Patentes y propiedad industrial - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36531,36531,Programas de computadora (software) - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36532,36532,Programas de computadora (software) - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36541,36541,Costos de exploración y desarrollo - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36542,36542,Costos de exploración y desarrollo - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36543,36543,Costos de exploración y desarrollo - Costo de financiación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36551,36551,"Fórmulas, diseños y prototipos - Costo",group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36552,36552,"Fórmulas, diseños y prototipos - Revaluación",group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart3657,3657,Plusvalía mercantil,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36591,36591,Otros activos intangibles - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36592,36592,Otros activos intangibles - Revaluación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36611,36611,Activos biológicos en producción - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36613,36613,Activos biológicos en producción - Costo de financiación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36621,36621,Activos biológicos en desarrollo - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36622,36622,Activos biológicos en desarrollo - Costo de financiación,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36711,36711,Inversiones a ser mantenidas hasta el vencimiento - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36721,36721,Inversiones financieras representativas de derecho patrimonial - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart36731,36731,Otras inversiones financieras - Costo,group36,account.data_account_type_fixed_assets,pe_chart_template,False
chart3711,3711,Impuesto a la renta diferido - Impuesto a la renta diferido – Patrimonio,group37,account.data_account_type_fixed_assets,pe_chart_template,False
chart3712,3712,Impuesto a la renta diferido - Impuesto a la renta diferido – Resultados,group37,account.data_account_type_fixed_assets,pe_chart_template,False
chart3721,3721,Participaciones de los trabajadores diferidas - Participaciones de los trabajadores diferidas – Patrimonio,group37,account.data_account_type_fixed_assets,pe_chart_template,False
chart3722,3722,Participaciones de los trabajadores diferidas - Participaciones de los trabajadores diferidas – Resultados,group37,account.data_account_type_fixed_assets,pe_chart_template,False
chart3731,3731,Intereses diferidos - Intereses no devengados en transacciones con terceros,group37,account.data_account_type_fixed_assets,pe_chart_template,False
chart3732,3732,Intereses diferidos - Intereses no devengados en medición a valor descontado,group37,account.data_account_type_fixed_assets,pe_chart_template,False
chart3811,3811,Bienes de arte y cultura - Obras de arte,group38,account.data_account_type_fixed_assets,pe_chart_template,False
chart3812,3812,Bienes de arte y cultura - Biblioteca,group38,account.data_account_type_fixed_assets,pe_chart_template,False
chart3813,3813,Bienes de arte y cultura - Otros,group38,account.data_account_type_fixed_assets,pe_chart_template,False
chart3821,3821,Diversos - Monedas y joyas,group38,account.data_account_type_fixed_assets,pe_chart_template,False
chart3822,3822,Diversos - Bienes entregados en comodato,group38,account.data_account_type_fixed_assets,pe_chart_template,False
chart3823,3823,Diversos - Bienes recibidos en pago (adjudicados y realizables),group38,account.data_account_type_fixed_assets,pe_chart_template,False
chart3829,3829,Diversos - Otros,group38,account.data_account_type_fixed_assets,pe_chart_template,False
chart39111,39111,Depreciación acumulada propiedades de inversión - Edificaciones - Costo,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39112,39112,Depreciación acumulada propiedades de inversión - Edificaciones - Revaluación,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39113,39113,Depreciación acumulada propiedades de inversión - Edificaciones - Costo de financiación,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39211,39211,Depreciación acumulada propiedades de inversión - Arrendamiento financiero - Edificaciones - Costo,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39212,39212,Depreciación acumulada propiedades de inversión - Arrendamiento financiero - Edificaciones - Revaluación,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39213,39213,Depreciación acumulada propiedades de inversión - Arrendamiento financiero - Edificaciones - Costo de financiación,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39321,39321,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Edificaciones - Costo",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39322,39322,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Edificaciones - Revaluación",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39323,39323,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Edificaciones - Costo de financiación",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39331,39331,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Maquinarias y equipos de explotación - Costo",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39332,39332,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Maquinarias y equipos de explotación - Revaluación",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39333,39333,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Maquinarias y equipos de explotación - Costo de financiación",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39341,39341,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Unidades de transporte - Costo",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39342,39342,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Unidades de transporte - Revaluación",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39351,39351,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Muebles y enseres - Costo",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39361,39361,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Equipos diversos - Costo",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39362,39362,"Depreciación acumulada propiedad, planta y equipo - Arrendamiento financiero - Equipos diversos - Revaluación",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39410,39410,Depreciación acumulada - Arrendamiento operativo - Activos por derecho de uso - arrendamiento operativo - Plantas productoras,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39411,39411,Depreciación acumulada - Arrendamiento operativo - Activos por derecho de uso - arrendamiento operativo - Terrenos,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39412,39412,Depreciación acumulada - Arrendamiento operativo - Activos por derecho de uso - arrendamiento operativo - Edificaciones,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39413,39413,Depreciación acumulada - Arrendamiento operativo - Activos por derecho de uso - arrendamiento operativo - Maquinarias y equipos de explotación,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39414,39414,Depreciación acumulada - Arrendamiento operativo - Activos por derecho de uso - arrendamiento operativo - Unidades de transporte,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39415,39415,Depreciación acumulada - Arrendamiento operativo - Activos por derecho de uso - arrendamiento operativo - Equipos diversos,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39520,39520,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Plantas productoras",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39521,39521,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Edificaciones",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39522,39522,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Instalaciones",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39523,39523,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Mejoras en locales arrendados",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39524,39524,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Maquinarias y equipos de explotación",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39525,39525,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Unidades de transporte",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39526,39526,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Muebles y enseres",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39527,39527,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Equipos diversos",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39528,39528,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Herramientas",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39529,39529,"Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Unidades de reemplazo",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39530,39530,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Plantas productoras",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39531,39531,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Edificaciones",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39532,39532,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Instalaciones",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39533,39533,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Mejoras en locales arrendados",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39534,39534,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Maquinarias y equipos de explotación",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39535,39535,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Unidades de transporte",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39536,39536,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Muebles y enseres",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39537,39537,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Equipos diversos",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39538,39538,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Herramientas y unidades de reemplazo",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39540,39540,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Costo de financiación - Plantas productoras",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39541,39541,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Costo de financiación - Edificaciones",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39542,39542,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Costo de financiación - Maquinarias y equipos de explotación",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39550,39550,"Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Valor razonable - Plantas productoras",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39611,39611,"Amortización acumulada - Intangibles – Costo - Concesiones, licencias y otros derechos",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39612,39612,Amortización acumulada - Intangibles – Costo - Patentes y propiedad industrial,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39613,39613,Amortización acumulada - Intangibles – Costo - Programas de computadora (software),group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39614,39614,Amortización acumulada - Intangibles – Costo - Costos de exploración y desarrollo,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39615,39615,"Amortización acumulada - Intangibles – Costo - Fórmulas, diseños y prototipos",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39619,39619,Amortización acumulada - Intangibles – Costo - Otros activos intangibles,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39621,39621,"Amortización acumulada - Intangibles – Revaluación - Concesiones, licencias y otros derechos",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39622,39622,Amortización acumulada - Intangibles – Revaluación - Patentes y propiedad industrial,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39623,39623,Amortización acumulada - Intangibles – Revaluación - Programas de computadora (software),group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39624,39624,Amortización acumulada - Intangibles – Revaluación - Costos de exploración y desarrollo,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39625,39625,"Amortización acumulada - Intangibles – Revaluación - Fórmulas, diseños y prototipos",group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39629,39629,Amortización acumulada - Intangibles – Revaluación - Otros activos intangibles,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39633,39633,Amortización acumulada - Intangibles – Costos de financiación - Programas de computadora,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39634,39634,Amortización acumulada - Intangibles – Costos de financiación - Costos de exploración,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39635,39635,Amortización acumulada - Intangibles – Costos de financiación - Costos de desarrollo,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart39811,39811,Depreciación acumulada - Activos biológicos en producción - Activos biológicos en producción - Costo - Activos biológicos en producción,group39,account.data_account_type_fixed_assets,pe_chart_template,False
chart40111,40111,Gobierno nacional - Impuesto general a las ventas - IGV – Cuenta propia,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40112,40112,Gobierno nacional - Impuesto general a las ventas - IGV – Servicios prestados por no domiciliados,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40113,40113,Gobierno nacional - Impuesto general a las ventas - IGV – Régimen de percepciones,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40114,40114,Gobierno nacional - Impuesto general a las ventas - IGV – Régimen de retenciones,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40115,40115,Gobierno nacional - Impuesto general a las ventas - IGV – Importaciones,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40116,40116,Gobierno nacional - Impuesto general a las ventas - IGV – Destinado a operaciones gravadas,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40117,40117,Gobierno nacional - Impuesto general a las ventas - IGV -Destinado a operaciones comunes,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart4012,4012,Gobierno nacional - Impuesto selectivo al consumo,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40151,40151,Gobierno nacional - Derechos aduaneros - Derechos arancelarios,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40152,40152,Gobierno nacional - Derechos aduaneros - Otros derechos arancelarios,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40171,40171,Gobierno nacional - Impuesto a la renta - Renta de tercera categoría,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40172,40172,Gobierno nacional - Impuesto a la renta - Renta de cuarta categoría,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40173,40173,Gobierno nacional - Impuesto a la renta - Renta de quinta categoría,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40174,40174,Gobierno nacional - Impuesto a la renta - Renta de no domiciliados,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40175,40175,Gobierno nacional - Impuesto a la renta - Otras retenciones,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40181,40181,Gobierno nacional - Otros impuestos y contraprestaciones - Impuesto a las transacciones financieras,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40182,40182,Gobierno nacional - Otros impuestos y contraprestaciones - Impuesto a los juegos de casino y tragamonedas,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40183,40183,Gobierno nacional - Otros impuestos y contraprestaciones - Tasas por la prestación de servicios públicos,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40184,40184,Gobierno nacional - Otros impuestos y contraprestaciones - Regalías,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40185,40185,Gobierno nacional - Otros impuestos y contraprestaciones - Impuesto a los dividendos,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40186,40186,Gobierno nacional - Otros impuestos y contraprestaciones - Impuesto temporal a los activos netos,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40189,40189,Gobierno nacional - Otros impuestos y contraprestaciones - Otros impuestos,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart402,402,Certificados tributarios,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart4031,4031,Instituciones públicas - ESSALUD,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart4032,4032,Instituciones públicas - ONP,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart4033,4033,Instituciones públicas - Contribución al SENATI,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart4034,4034,Instituciones públicas - Contribución al SENCICO,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart4039,4039,Instituciones públicas - Otras instituciones,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart405,405,Gobiernos regionales,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40611,40611,Gobiernos locales - Impuestos - Impuesto al patrimonio vehicular,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40612,40612,Gobiernos locales - Impuestos - Impuesto a las apuestas,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40613,40613,Gobiernos locales - Impuestos - Impuesto a los juegos,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40614,40614,Gobiernos locales - Impuestos - Impuesto de alcabala,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40615,40615,Gobiernos locales - Impuestos - Impuesto predial,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40616,40616,Gobiernos locales - Impuestos - Impuesto a los espectáculos públicos no deportivos,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart4062,4062,Gobiernos locales - Contribuciones,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40631,40631,Gobiernos locales - Tasas - Licencia de apertura de establecimientos,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40632,40632,Gobiernos locales - Tasas - Transporte público,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40633,40633,Gobiernos locales - Tasas - Estacionamiento de vehículos,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40634,40634,Gobiernos locales - Tasas - Servicios públicos o arbitrios,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart40635,40635,Gobiernos locales - Tasas - Servicios administrativos o derechos,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart409,409,Otros costos administrativos e intereses,group40,account.data_account_type_current_liabilities,pe_chart_template,False
chart4111,4111,Remuneraciones por pagar - Sueldos y salarios por pagar,group41,account.data_account_type_payable,pe_chart_template,True
chart4112,4112,Remuneraciones por pagar - Comisiones por pagar,group41,account.data_account_type_payable,pe_chart_template,True
chart4113,4113,Remuneraciones por pagar - Remuneraciones en especie por pagar,group41,account.data_account_type_payable,pe_chart_template,True
chart4114,4114,Remuneraciones por pagar - Gratificaciones por pagar,group41,account.data_account_type_payable,pe_chart_template,True
chart4115,4115,Remuneraciones por pagar - Vacaciones por pagar,group41,account.data_account_type_payable,pe_chart_template,True
chart413,413,Participaciones de los trabajadores por pagar,group41,account.data_account_type_payable,pe_chart_template,True
chart4151,4151,Beneficios sociales de los trabajadores por pagar - Compensación por tiempo de servicios,group41,account.data_account_type_payable,pe_chart_template,True
chart4152,4152,Beneficios sociales de los trabajadores por pagar - Adelanto de compensación por tiempo de servicios,group41,account.data_account_type_payable,pe_chart_template,True
chart4153,4153,Beneficios sociales de los trabajadores por pagar - Pensiones y jubilaciones,group41,account.data_account_type_payable,pe_chart_template,True
chart417,417,Administradoras de fondos de pensiones,group41,account.data_account_type_payable,pe_chart_template,True
chart419,419,Otras remuneraciones y participaciones por pagar,group41,account.data_account_type_payable,pe_chart_template,True
chart4211,4211,"Facturas, boletas y otros comprobantes por pagar - No emitidas",group42,account.data_account_type_payable,pe_chart_template,True
chart4212,4212,"Facturas, boletas y otros comprobantes por pagar - Emitidas",group42,account.data_account_type_payable,pe_chart_template,True
chart422,422,Anticipos a proveedores,group42,account.data_account_type_payable,pe_chart_template,True
chart423,423,Letras por pagar,group42,account.data_account_type_payable,pe_chart_template,True
chart424,424,Honorarios por pagar,group42,account.data_account_type_payable,pe_chart_template,True
chart4311,4311,"Facturas, boletas y otros comprobantes por pagar - No emitidas",group43,account.data_account_type_payable,pe_chart_template,True
chart4312,4312,"Facturas, boletas y otros comprobantes por pagar - Emitidas",group43,account.data_account_type_payable,pe_chart_template,True
chart4321,4321,Anticipos otorgados - Anticipos otorgados,group43,account.data_account_type_payable,pe_chart_template,True
chart4331,4331,Letras por pagar - Letras por pagar,group43,account.data_account_type_payable,pe_chart_template,True
chart4341,4341,Honorarios por pagar - Honorarios por pagar,group43,account.data_account_type_payable,pe_chart_template,True
chart4411,4411,"Accionistas ( socios, partícipes) - Préstamos",group44,account.data_account_type_non_current_liabilities,pe_chart_template,False
chart4412,4412,"Accionistas ( socios, partícipes) - Dividendos",group44,account.data_account_type_non_current_liabilities,pe_chart_template,False
chart4419,4419,"Accionistas ( socios, partícipes) - Otras cuentas por pagar",group44,account.data_account_type_non_current_liabilities,pe_chart_template,False
chart4421,4421,Directores - Dietas,group44,account.data_account_type_non_current_liabilities,pe_chart_template,False
chart4429,4429,Directores - Otras cuentas por pagar,group44,account.data_account_type_non_current_liabilities,pe_chart_template,False
chart4511,4511,Préstamos de instituciones financieras y otras entidades - Instituciones financieras,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart4512,4512,Préstamos de instituciones financieras y otras entidades - Otras entidades,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart452,452,Contratos de arrendamiento financiero,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart4531,4531,Obligaciones emitidas - Bonos emitidos,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart4532,4532,Obligaciones emitidas - Bonos titulizados,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart4533,4533,Obligaciones emitidas - Papeles comerciales,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart4539,4539,Obligaciones emitidas - Otras obligaciones,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart4541,4541,Otros Instrumentos financieros por pagar - Letras,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart4542,4542,Otros Instrumentos financieros por pagar - Papeles comerciales,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart4543,4543,Otros Instrumentos financieros por pagar - Bonos,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart4544,4544,Otros Instrumentos financieros por pagar - Pagarés,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart4545,4545,Otros Instrumentos financieros por pagar - Facturas conformadas,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart4549,4549,Otros Instrumentos financieros por pagar - Otras obligaciones financieras,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart45511,45511,Costos de financiación por pagar - Préstamos de instituciones financieras y otras entidades - Instituciones financieras,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart45512,45512,Costos de financiación por pagar - Préstamos de instituciones financieras y otras entidades - Otras entidades,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart4552,4552,Costos de financiación por pagar - Contratos de arrendamiento financiero,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart45531,45531,Costos de financiación por pagar - Obligaciones emitidas - Bonos emitidos,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart45532,45532,Costos de financiación por pagar - Obligaciones emitidas - Bonos titulizados,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart45533,45533,Costos de financiación por pagar - Obligaciones emitidas - Papeles comerciales,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart45539,45539,Costos de financiación por pagar - Obligaciones emitidas - Otras obligaciones,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart45541,45541,Costos de financiación por pagar - Otros instrumentos financieros por pagar - Letras,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart45542,45542,Costos de financiación por pagar - Otros instrumentos financieros por pagar - Papeles comerciales,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart45543,45543,Costos de financiación por pagar - Otros instrumentos financieros por pagar - Bonos,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart45544,45544,Costos de financiación por pagar - Otros instrumentos financieros por pagar - Pagarés,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart45545,45545,Costos de financiación por pagar - Otros instrumentos financieros por pagar - Facturas conformadas,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart45549,45549,Costos de financiación por pagar - Otros instrumentos financieros por pagar - Otras obligaciones financieras,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart456,456,Préstamos con compromisos de recompra,group45,account.data_account_type_current_liabilities,pe_chart_template,False
chart461,461,Reclamaciones de terceros,group46,account.data_account_type_payable,pe_chart_template,True
chart4641,4641,Pasivos por instrumentos financieros - Instrumentos financieros primarios,group46,account.data_account_type_payable,pe_chart_template,True
chart46421,46421,Pasivos por instrumentos financieros - Instrumentos financieros derivados - Cartera de negociación,group46,account.data_account_type_payable,pe_chart_template,True
chart46422,46422,Pasivos por instrumentos financieros - Instrumentos financieros derivados - Instrumentos de cobertura,group46,account.data_account_type_payable,pe_chart_template,True
chart4651,4651,Pasivos por compra de activo inmovilizado - Inversiones mobiliarias,group46,account.data_account_type_payable,pe_chart_template,True
chart4652,4652,Pasivos por compra de activo inmovilizado - Propiedades de inversión,group46,account.data_account_type_payable,pe_chart_template,True
chart4653,4653,Pasivos por compra de activo inmovilizado - Activos adquiridos en arrendamiento financiero,group46,account.data_account_type_payable,pe_chart_template,True
chart4654,4654,"Pasivos por compra de activo inmovilizado - Propiedad, planta y equipo",group46,account.data_account_type_payable,pe_chart_template,True
chart4655,4655,Pasivos por compra de activo inmovilizado - Intangibles,group46,account.data_account_type_payable,pe_chart_template,True
chart4656,4656,Pasivos por compra de activo inmovilizado - Activos biológicos,group46,account.data_account_type_payable,pe_chart_template,True
chart466,466,Participación de terceros en acuerdos conjuntos,group46,account.data_account_type_payable,pe_chart_template,True
chart467,467,Depósitos recibidos en garantía,group46,account.data_account_type_payable,pe_chart_template,True
chart4691,4691,Otras cuentas por pagar diversas - Subsidios gubernamentales,group46,account.data_account_type_payable,pe_chart_template,True
chart4692,4692,Otras cuentas por pagar diversas - Donaciones condicionadas,group46,account.data_account_type_payable,pe_chart_template,True
chart4699,4699,Otras cuentas por pagar diversas - Otras cuentas por pagar,group46,account.data_account_type_payable,pe_chart_template,True
chart471,471,Préstamos,group47,account.data_account_type_payable,pe_chart_template,True
chart472,472,Costos de financiación,group47,account.data_account_type_payable,pe_chart_template,True
chart473,473,Anticipos recibidos,group47,account.data_account_type_payable,pe_chart_template,True
chart474,474,Regalías,group47,account.data_account_type_payable,pe_chart_template,True
chart475,475,Dividendos,group47,account.data_account_type_payable,pe_chart_template,True
chart476,476,Depósitos recibidos en garantía,group47,account.data_account_type_payable,pe_chart_template,True
chart4771,4771,Pasivo por compra de activo inmovilizado - Inversiones mobiliarias,group47,account.data_account_type_payable,pe_chart_template,True
chart4772,4772,Pasivo por compra de activo inmovilizado - Inversiones inmobiliarias,group47,account.data_account_type_payable,pe_chart_template,True
chart4773,4773,Pasivo por compra de activo inmovilizado - Activos adquiridos en arrendamiento financiero,group47,account.data_account_type_payable,pe_chart_template,True
chart4774,4774,"Pasivo por compra de activo inmovilizado - Propiedad, planta y equipo",group47,account.data_account_type_payable,pe_chart_template,True
chart4775,4775,Pasivo por compra de activo inmovilizado - Intangibles,group47,account.data_account_type_payable,pe_chart_template,True
chart4776,4776,Pasivo por compra de activo inmovilizado - Activos biológicos,group47,account.data_account_type_payable,pe_chart_template,True
chart4791,4791,Otras cuentas por pagar diversas - Otras cuentas por pagar diversas,group47,account.data_account_type_payable,pe_chart_template,True
chart481,481,Provisión para litigios,group48,account.data_account_type_non_current_liabilities,pe_chart_template,False
chart482,482,"Provisión por desmantelamiento, retiro o rehabilitación del inmovilizado",group48,account.data_account_type_non_current_liabilities,pe_chart_template,False
chart483,483,Provisión para reestructuraciones,group48,account.data_account_type_non_current_liabilities,pe_chart_template,False
chart484,484,Provisión para protección y remediación del medio ambiente,group48,account.data_account_type_non_current_liabilities,pe_chart_template,False
chart485,485,Provisión para gastos de responsabilidad social,group48,account.data_account_type_non_current_liabilities,pe_chart_template,False
chart486,486,Provisión para garantías,group48,account.data_account_type_non_current_liabilities,pe_chart_template,False
chart487,487,Provisión por activos por derecho de uso,group48,account.data_account_type_non_current_liabilities,pe_chart_template,False
chart489,489,Otras provisiones,group48,account.data_account_type_non_current_liabilities,pe_chart_template,False
chart4911,4911,Impuesto a la renta diferido - Impuesto a la renta diferido – Patrimonio,group49,account.data_account_type_current_liabilities,pe_chart_template,False
chart4912,4912,Impuesto a la renta diferido - Impuesto a la renta diferido – Resultados,group49,account.data_account_type_current_liabilities,pe_chart_template,False
chart4921,4921,Participaciones de los trabajadores diferidas - Participaciones de los trabajadores diferidas – Patrimonio,group49,account.data_account_type_current_liabilities,pe_chart_template,False
chart4922,4922,Participaciones de los trabajadores diferidas - Participaciones de los trabajadores diferidas – Resultados,group49,account.data_account_type_current_liabilities,pe_chart_template,False
chart4931,4931,Intereses diferidos - Intereses no devengados en transacciones con terceros,group49,account.data_account_type_current_liabilities,pe_chart_template,False
chart4932,4932,Intereses diferidos - Intereses no devengados en medición a valor descontado,group49,account.data_account_type_current_liabilities,pe_chart_template,False
chart494,494,Ganancia en venta con arrendamiento financiero paralelo,group49,account.data_account_type_current_liabilities,pe_chart_template,False
chart495,495,Subsidios recibidos diferidos,group49,account.data_account_type_current_liabilities,pe_chart_template,False
chart496,496,Ingresos diferidos,group49,account.data_account_type_current_liabilities,pe_chart_template,False
chart497,497,Costos diferidos,group49,account.data_account_type_current_liabilities,pe_chart_template,False
chart5011,5011,Capital social - Acciones,group50,account.data_account_type_equity,pe_chart_template,False
chart5012,5012,Capital social - Participaciones,group50,account.data_account_type_equity,pe_chart_template,False
chart502,502,Acciones en tesorería,group50,account.data_account_type_equity,pe_chart_template,False
chart511,511,Acciones de inversión,group51,account.data_account_type_equity,pe_chart_template,False
chart512,512,Acciones de inversión en tesorería,group51,account.data_account_type_equity,pe_chart_template,False
chart521,521,Primas (descuento) de acciones,group52,account.data_account_type_equity,pe_chart_template,False
chart5221,5221,Capitalizaciones en trámite - Aportes,group52,account.data_account_type_equity,pe_chart_template,False
chart5222,5222,Capitalizaciones en trámite - Reservas,group52,account.data_account_type_equity,pe_chart_template,False
chart5223,5223,Capitalizaciones en trámite - Acreencias,group52,account.data_account_type_equity,pe_chart_template,False
chart5224,5224,Capitalizaciones en trámite - Utilidades,group52,account.data_account_type_equity,pe_chart_template,False
chart523,523,Reducciones de capital pendientes de formalización,group52,account.data_account_type_equity,pe_chart_template,False
chart561,561,Diferencia en cambio de inversiones permanentes en entidades extranjeras,group56,account.data_account_type_equity,pe_chart_template,False
chart562,562,Instrumentos financieros – Coberturas,group56,account.data_account_type_equity,pe_chart_template,False
chart5631,5631,Resultado en activos o pasivos financieros mantenidos para negociación - Ganancia,group56,account.data_account_type_equity,pe_chart_template,False
chart5632,5632,Resultado en activos o pasivos financieros mantenidos para negociación - Pérdida,group56,account.data_account_type_equity,pe_chart_template,False
chart5641,5641,Resultado en otros activos o pasivos por inversiones financieras - Ganancia,group56,account.data_account_type_equity,pe_chart_template,False
chart5642,5642,Resultado en otros activos o pasivos por inversiones financieras - Pérdida,group56,account.data_account_type_equity,pe_chart_template,False
chart5651,5651,Resultado en activos o pasivos financieros mantenidos para negociación – Compra o venta convencional fecha de liquidación - Ganancia,group56,account.data_account_type_equity,pe_chart_template,False
chart5652,5652,Resultado en activos o pasivos financieros mantenidos para negociación – Compra o venta convencional fecha de liquidación - Pérdida,group56,account.data_account_type_equity,pe_chart_template,False
chart57111,57111,Excedente de revaluación - Propiedad de inversión - Adquisición directa,group57,account.data_account_type_equity,pe_chart_template,False
chart57112,57112,Excedente de revaluación - Propiedad de inversión - Arrendamiento financiero,group57,account.data_account_type_equity,pe_chart_template,False
chart57121,57121,"Excedente de revaluación - Propiedad, planta y equipo - Adquisición directa",group57,account.data_account_type_equity,pe_chart_template,False
chart57122,57122,"Excedente de revaluación - Propiedad, planta y equipo - Arrendamiento financiero",group57,account.data_account_type_equity,pe_chart_template,False
chart5713,5713,Excedente de revaluación - Intangibles,group57,account.data_account_type_equity,pe_chart_template,False
chart5714,5714,Excedente de revaluación - Activos por derecho de uso - arrendamiento operativo,group57,account.data_account_type_equity,pe_chart_template,False
chart572,572,Excedente de revaluación – Acciones liberadas recibidas,group57,account.data_account_type_equity,pe_chart_template,False
chart573,573,Participación en excedente de revaluación – Inversiones en entidades relacionadas,group57,account.data_account_type_equity,pe_chart_template,False
chart581,581,Reinversión,group58,account.data_account_type_equity,pe_chart_template,False
chart582,582,Legal,group58,account.data_account_type_equity,pe_chart_template,False
chart583,583,Contractuales,group58,account.data_account_type_equity,pe_chart_template,False
chart584,584,Estatutarias,group58,account.data_account_type_equity,pe_chart_template,False
chart585,585,Facultativas,group58,account.data_account_type_equity,pe_chart_template,False
chart589,589,Otras reservas,group58,account.data_account_type_equity,pe_chart_template,False
chart5911,5911,Utilidades no distribuidas - Utilidades acumuladas,group59,account.data_account_type_equity,pe_chart_template,False
chart5912,5912,Utilidades no distribuidas - Ingresos de años anteriores,group59,account.data_account_type_equity,pe_chart_template,False
chart5921,5921,Pérdidas acumuladas - Pérdidas acumuladas,group59,account.data_account_type_equity,pe_chart_template,False
chart5922,5922,Pérdidas acumuladas - Gastos de años anteriores,group59,account.data_account_type_equity,pe_chart_template,False
chart6011,6011,Mercaderías - Mercaderías,group60,account.data_account_type_current_assets,pe_chart_template,False
chart602,602,Materias primas,group60,account.data_account_type_current_assets,pe_chart_template,False
chart6031,6031,"Materiales auxiliares, suministros y repuestos - Materiales auxiliares",group60,account.data_account_type_current_assets,pe_chart_template,False
chart6032,6032,"Materiales auxiliares, suministros y repuestos - Suministros",group60,account.data_account_type_current_assets,pe_chart_template,False
chart6033,6033,"Materiales auxiliares, suministros y repuestos - Repuestos",group60,account.data_account_type_current_assets,pe_chart_template,False
chart6041,6041,Envases y embalajes - Envases,group60,account.data_account_type_current_assets,pe_chart_template,False
chart6042,6042,Envases y embalajes - Embalajes,group60,account.data_account_type_current_assets,pe_chart_template,False
chart60911,60911,Costos vinculados con las compras - Costos vinculados con las compras de mercaderías - Transporte,group60,account.data_account_type_current_assets,pe_chart_template,False
chart60912,60912,Costos vinculados con las compras - Costos vinculados con las compras de mercaderías - Seguros,group60,account.data_account_type_current_assets,pe_chart_template,False
chart60913,60913,Costos vinculados con las compras - Costos vinculados con las compras de mercaderías - Derechos aduaneros,group60,account.data_account_type_current_assets,pe_chart_template,False
chart60914,60914,Costos vinculados con las compras - Costos vinculados con las compras de mercaderías - Comisiones,group60,account.data_account_type_current_assets,pe_chart_template,False
chart60919,60919,Costos vinculados con las compras - Costos vinculados con las compras de mercaderías - Otros costos,group60,account.data_account_type_current_assets,pe_chart_template,False
chart60921,60921,Costos vinculados con las compras - Costos vinculados con las compras de materias primas - Transporte,group60,account.data_account_type_current_assets,pe_chart_template,False
chart60922,60922,Costos vinculados con las compras - Costos vinculados con las compras de materias primas - Seguros,group60,account.data_account_type_current_assets,pe_chart_template,False
chart60923,60923,Costos vinculados con las compras - Costos vinculados con las compras de materias primas - Derechos aduaneros,group60,account.data_account_type_current_assets,pe_chart_template,False
chart60924,60924,Costos vinculados con las compras - Costos vinculados con las compras de materias primas - Comisiones,group60,account.data_account_type_current_assets,pe_chart_template,False
chart60925,60925,Costos vinculados con las compras - Costos vinculados con las compras de materias primas - Otros costos,group60,account.data_account_type_current_assets,pe_chart_template,False
chart60931,60931,"Costos vinculados con las compras - Costos vinculados con las compras de materiales, suministros y repuestos - Transporte",group60,account.data_account_type_current_assets,pe_chart_template,False
chart60932,60932,"Costos vinculados con las compras - Costos vinculados con las compras de materiales, suministros y repuestos - Seguros",group60,account.data_account_type_current_assets,pe_chart_template,False
chart60933,60933,"Costos vinculados con las compras - Costos vinculados con las compras de materiales, suministros y repuestos - Derechos aduaneros",group60,account.data_account_type_current_assets,pe_chart_template,False
chart60934,60934,"Costos vinculados con las compras - Costos vinculados con las compras de materiales, suministros y repuestos - Comisiones",group60,account.data_account_type_current_assets,pe_chart_template,False
chart60935,60935,"Costos vinculados con las compras - Costos vinculados con las compras de materiales, suministros y repuestos - Otros costos",group60,account.data_account_type_current_assets,pe_chart_template,False
chart60941,60941,Costos vinculados con las compras - Costos vinculados con las compras de envases y embalajes - Transporte,group60,account.data_account_type_current_assets,pe_chart_template,False
chart60942,60942,Costos vinculados con las compras - Costos vinculados con las compras de envases y embalajes - Seguros,group60,account.data_account_type_current_assets,pe_chart_template,False
chart60943,60943,Costos vinculados con las compras - Costos vinculados con las compras de envases y embalajes - Derechos aduaneros,group60,account.data_account_type_current_assets,pe_chart_template,False
chart60944,60944,Costos vinculados con las compras - Costos vinculados con las compras de envases y embalajes - Comisiones,group60,account.data_account_type_current_assets,pe_chart_template,False
chart60945,60945,Costos vinculados con las compras - Costos vinculados con las compras de envases y embalajes - Otros costos,group60,account.data_account_type_current_assets,pe_chart_template,False
chart6111,6111,Mercaderías - Mercaderías,group61,account.data_account_type_expenses,pe_chart_template,False
chart6121,6121,Materias primas - Materias primas,group61,account.data_account_type_expenses,pe_chart_template,False
chart6131,6131,"Materiales auxiliares, suministros y repuestos - Materiales auxiliares",group61,account.data_account_type_expenses,pe_chart_template,False
chart6132,6132,"Materiales auxiliares, suministros y repuestos - Suministros",group61,account.data_account_type_expenses,pe_chart_template,False
chart6133,6133,"Materiales auxiliares, suministros y repuestos - Repuestos",group61,account.data_account_type_expenses,pe_chart_template,False
chart6141,6141,Envases y embalajes - Envases,group61,account.data_account_type_expenses,pe_chart_template,False
chart6142,6142,Envases y embalajes - Embalajes,group61,account.data_account_type_expenses,pe_chart_template,False
chart6211,6211,Remuneraciones - Sueldos y salarios,group62,account.data_account_type_expenses,pe_chart_template,False
chart6212,6212,Remuneraciones - Comisiones,group62,account.data_account_type_expenses,pe_chart_template,False
chart6213,6213,Remuneraciones - Remuneraciones en especie,group62,account.data_account_type_expenses,pe_chart_template,False
chart6214,6214,Remuneraciones - Gratificaciones,group62,account.data_account_type_expenses,pe_chart_template,False
chart6215,6215,Remuneraciones - Vacaciones,group62,account.data_account_type_expenses,pe_chart_template,False
chart622,622,Otras remuneraciones,group62,account.data_account_type_expenses,pe_chart_template,False
chart623,623,Indemnizaciones al personal,group62,account.data_account_type_expenses,pe_chart_template,False
chart624,624,Capacitación,group62,account.data_account_type_expenses,pe_chart_template,False
chart625,625,Atención al personal,group62,account.data_account_type_expenses,pe_chart_template,False
chart6271,6271,"Seguridad, previsión social y otras contribuciones - Régimen de prestaciones de salud",group62,account.data_account_type_expenses,pe_chart_template,False
chart6272,6272,"Seguridad, previsión social y otras contribuciones - Régimen de pensiones - Aporte de empresa",group62,account.data_account_type_expenses,pe_chart_template,False
chart6273,6273,"Seguridad, previsión social y otras contribuciones - Seguro complementario de trabajo de riesgo, accidentes de trabajo y enfermedades profesionales",group62,account.data_account_type_expenses,pe_chart_template,False
chart6274,6274,"Seguridad, previsión social y otras contribuciones - Seguro de vida",group62,account.data_account_type_expenses,pe_chart_template,False
chart6275,6275,"Seguridad, previsión social y otras contribuciones - Seguros particulares de prestaciones de salud – EPS y otros particulares",group62,account.data_account_type_expenses,pe_chart_template,False
chart6276,6276,"Seguridad, previsión social y otras contribuciones - Caja de beneficios de seguridad social del pescador",group62,account.data_account_type_expenses,pe_chart_template,False
chart6277,6277,"Seguridad, previsión social y otras contribuciones - Contribuciones al SENATI",group62,account.data_account_type_expenses,pe_chart_template,False
chart628,628,Retribuciones al directorio,group62,account.data_account_type_expenses,pe_chart_template,False
chart6291,6291,Beneficios sociales de los trabajadores - Compensación por tiempo de servicio,group62,account.data_account_type_expenses,pe_chart_template,False
chart6292,6292,Beneficios sociales de los trabajadores - Pensiones y jubilaciones,group62,account.data_account_type_expenses,pe_chart_template,False
chart6293,6293,Beneficios sociales de los trabajadores - Otros beneficios post-empleo,group62,account.data_account_type_expenses,pe_chart_template,False
chart62941,62941,Beneficios sociales de los trabajadores - Participación en las utilidades - Participación corriente,group62,account.data_account_type_expenses,pe_chart_template,False
chart62942,62942,Beneficios sociales de los trabajadores - Participación en las utilidades - Participación diferida,group62,account.data_account_type_expenses,pe_chart_template,False
chart63111,63111,"Transporte, correos y gastos de viaje - Transporte - De carga",group63,account.data_account_type_expenses,pe_chart_template,False
chart63112,63112,"Transporte, correos y gastos de viaje - Transporte - De pasajeros",group63,account.data_account_type_expenses,pe_chart_template,False
chart6312,6312,"Transporte, correos y gastos de viaje - Correos",group63,account.data_account_type_expenses,pe_chart_template,False
chart6313,6313,"Transporte, correos y gastos de viaje - Alojamiento",group63,account.data_account_type_expenses,pe_chart_template,False
chart6314,6314,"Transporte, correos y gastos de viaje - Alimentación",group63,account.data_account_type_expenses,pe_chart_template,False
chart6315,6315,"Transporte, correos y gastos de viaje - Otros gastos de viaje",group63,account.data_account_type_expenses,pe_chart_template,False
chart6321,6321,Asesoría y consultoría - Administrativa,group63,account.data_account_type_expenses,pe_chart_template,False
chart6322,6322,Asesoría y consultoría - Legal y tributaria,group63,account.data_account_type_expenses,pe_chart_template,False
chart6323,6323,Asesoría y consultoría - Auditoría y contable,group63,account.data_account_type_expenses,pe_chart_template,False
chart6324,6324,Asesoría y consultoría - Mercadotecnia,group63,account.data_account_type_expenses,pe_chart_template,False
chart6325,6325,Asesoría y consultoría - Medioambiental,group63,account.data_account_type_expenses,pe_chart_template,False
chart6326,6326,Asesoría y consultoría - Investigación y desarrollo,group63,account.data_account_type_expenses,pe_chart_template,False
chart6327,6327,Asesoría y consultoría - Producción,group63,account.data_account_type_expenses,pe_chart_template,False
chart6329,6329,Asesoría y consultoría - Otros,group63,account.data_account_type_expenses,pe_chart_template,False
chart633,633,Producción encargada a terceros,group63,account.data_account_type_expenses,pe_chart_template,False
chart6341,6341,Mantenimiento y reparaciones - Propiedad de inversión,group63,account.data_account_type_expenses,pe_chart_template,False
chart63421,63421,Mantenimiento y reparaciones - Activos por derecho de uso - Financiero,group63,account.data_account_type_expenses,pe_chart_template,False
chart63432,63432,"Mantenimiento y reparaciones - Propiedad, planta y equipo - Operativo",group63,account.data_account_type_expenses,pe_chart_template,False
chart6344,6344,Mantenimiento y reparaciones - Intangibles,group63,account.data_account_type_expenses,pe_chart_template,False
chart6345,6345,Mantenimiento y reparaciones - Activos biológicos,group63,account.data_account_type_expenses,pe_chart_template,False
chart6351,6351,Alquileres - Terrenos,group63,account.data_account_type_expenses,pe_chart_template,False
chart6352,6352,Alquileres - Edificaciones,group63,account.data_account_type_expenses,pe_chart_template,False
chart6353,6353,Alquileres - Maquinarias y equipos de explotación,group63,account.data_account_type_expenses,pe_chart_template,False
chart6354,6354,Alquileres - Equipo de transporte,group63,account.data_account_type_expenses,pe_chart_template,False
chart6355,6355,Alquileres - Muebles y enseres,group63,account.data_account_type_expenses,pe_chart_template,False
chart6356,6356,Alquileres - Equipos diversos,group63,account.data_account_type_expenses,pe_chart_template,False
chart6361,6361,Servicios básicos - Energía eléctrica,group63,account.data_account_type_expenses,pe_chart_template,False
chart6362,6362,Servicios básicos - Gas,group63,account.data_account_type_expenses,pe_chart_template,False
chart6363,6363,Servicios básicos - Agua,group63,account.data_account_type_expenses,pe_chart_template,False
chart6364,6364,Servicios básicos - Teléfono,group63,account.data_account_type_expenses,pe_chart_template,False
chart6365,6365,Servicios básicos - Internet,group63,account.data_account_type_expenses,pe_chart_template,False
chart6366,6366,Servicios básicos - Radio,group63,account.data_account_type_expenses,pe_chart_template,False
chart6367,6367,Servicios básicos - Cable,group63,account.data_account_type_expenses,pe_chart_template,False
chart6371,6371,"Publicidad, publicaciones, relaciones públicas - Publicidad",group63,account.data_account_type_expenses,pe_chart_template,False
chart6372,6372,"Publicidad, publicaciones, relaciones públicas - Publicaciones",group63,account.data_account_type_expenses,pe_chart_template,False
chart6373,6373,"Publicidad, publicaciones, relaciones públicas - Relaciones públicas",group63,account.data_account_type_expenses,pe_chart_template,False
chart638,638,Servicios de contratistas,group63,account.data_account_type_expenses,pe_chart_template,False
chart6391,6391,Otros servicios prestados por terceros - Gastos bancarios,group63,account.data_account_type_expenses,pe_chart_template,False
chart6392,6392,Otros servicios prestados por terceros - Gastos de laboratorio,group63,account.data_account_type_expenses,pe_chart_template,False
chart6411,6411,Gobierno nacional - Impuesto general a las ventas y selectivo al consumo,group64,account.data_account_type_expenses,pe_chart_template,False
chart6412,6412,Gobierno nacional - Impuesto a las transacciones financieras,group64,account.data_account_type_expenses,pe_chart_template,False
chart6413,6413,Gobierno nacional - Impuesto temporal a los activos netos,group64,account.data_account_type_expenses,pe_chart_template,False
chart6414,6414,Gobierno nacional - Impuesto a los juegos de casino y máquinas tragamonedas,group64,account.data_account_type_expenses,pe_chart_template,False
chart6415,6415,Gobierno nacional - Regalías mineras,group64,account.data_account_type_expenses,pe_chart_template,False
chart6416,6416,Gobierno nacional - Cánones,group64,account.data_account_type_expenses,pe_chart_template,False
chart6419,6419,Gobierno nacional - Otros,group64,account.data_account_type_expenses,pe_chart_template,False
chart642,642,Gobierno regional,group64,account.data_account_type_expenses,pe_chart_template,False
chart6431,6431,Gobierno local - Impuesto predial,group64,account.data_account_type_expenses,pe_chart_template,False
chart6432,6432,Gobierno local - Arbitrios municipales y seguridad ciudadana,group64,account.data_account_type_expenses,pe_chart_template,False
chart6433,6433,Gobierno local - Impuesto al patrimonio vehicular,group64,account.data_account_type_expenses,pe_chart_template,False
chart6434,6434,Gobierno local - Licencia de funcionamiento,group64,account.data_account_type_expenses,pe_chart_template,False
chart6439,6439,Gobierno local - Otros,group64,account.data_account_type_expenses,pe_chart_template,False
chart6442,6442,Otros gastos por tributos - Contribución al SENCICO,group64,account.data_account_type_expenses,pe_chart_template,False
chart6443,6443,Otros gastos por tributos - Otros,group64,account.data_account_type_expenses,pe_chart_template,False
chart6451,6451,Gastos en deuda tributaria - Intereses,group64,account.data_account_type_expenses,pe_chart_template,False
chart6452,6452,Gastos en deuda tributaria - intereses - fraccionamiento,group64,account.data_account_type_expenses,pe_chart_template,False
chart6453,6453,Gastos en deuda tributaria - Multas,group64,account.data_account_type_expenses,pe_chart_template,False
chart6454,6454,Gastos en deuda tributaria - Costas y otros,group64,account.data_account_type_expenses,pe_chart_template,False
chart651,651,Seguros,group65,account.data_account_type_expenses,pe_chart_template,False
chart652,652,Regalías,group65,account.data_account_type_expenses,pe_chart_template,False
chart653,653,Suscripciones,group65,account.data_account_type_expenses,pe_chart_template,False
chart654,654,Licencias y derechos de vigencia,group65,account.data_account_type_expenses,pe_chart_template,False
chart65511,65511,Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Costo neto de enajenación de activos inmovilizados - Inversiones mobiliarias,group65,account.data_account_type_expenses,pe_chart_template,False
chart65512,65512,Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Costo neto de enajenación de activos inmovilizados - Propiedades de inversión,group65,account.data_account_type_expenses,pe_chart_template,False
chart65513,65513,Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Costo neto de enajenación de activos inmovilizados - Activos por derecho de uso - arrendamiento financiero,group65,account.data_account_type_expenses,pe_chart_template,False
chart65514,65514,"Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Costo neto de enajenación de activos inmovilizados - Propiedad, planta y equipo",group65,account.data_account_type_expenses,pe_chart_template,False
chart65515,65515,Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Costo neto de enajenación de activos inmovilizados - Intangibles,group65,account.data_account_type_expenses,pe_chart_template,False
chart65516,65516,Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Costo neto de enajenación de activos inmovilizados - Activos biológicos,group65,account.data_account_type_expenses,pe_chart_template,False
chart65521,65521,Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Operaciones discontinuadas – Abandono de activos - Propiedades de inversión,group65,account.data_account_type_expenses,pe_chart_template,False
chart65522,65522,Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Operaciones discontinuadas – Abandono de activos - Activos por derecho de uso - Arrendamiento financiero,group65,account.data_account_type_expenses,pe_chart_template,False
chart65523,65523,"Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Operaciones discontinuadas – Abandono de activos - Propiedad, planta y equipo",group65,account.data_account_type_expenses,pe_chart_template,False
chart65524,65524,Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Operaciones discontinuadas – Abandono de activos - Intangibles,group65,account.data_account_type_expenses,pe_chart_template,False
chart65525,65525,Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Operaciones discontinuadas – Abandono de activos - Activos biológicos,group65,account.data_account_type_expenses,pe_chart_template,False
chart656,656,Suministros,group65,account.data_account_type_expenses,pe_chart_template,False
chart658,658,Gestión medioambiental,group65,account.data_account_type_expenses,pe_chart_template,False
chart6591,6591,Otros gastos de gestión - Donaciones,group65,account.data_account_type_expenses,pe_chart_template,False
chart6592,6592,Otros gastos de gestión - Sanciones administrativas,group65,account.data_account_type_expenses,pe_chart_template,False
chart6611,6611,Activo realizable - Mercaderías,group66,account.data_account_type_expenses,pe_chart_template,False
chart6612,6612,Activo realizable - Productos terminados,group66,account.data_account_type_expenses,pe_chart_template,False
chart66131,66131,Activo realizable - Activos no corrientes mantenidos para la venta - Propiedades de inversión,group66,account.data_account_type_expenses,pe_chart_template,False
chart66132,66132,"Activo realizable - Activos no corrientes mantenidos para la venta - Propiedad, planta y equipo",group66,account.data_account_type_expenses,pe_chart_template,False
chart66133,66133,Activo realizable - Activos no corrientes mantenidos para la venta - Intangibles,group66,account.data_account_type_expenses,pe_chart_template,False
chart66134,66134,Activo realizable - Activos no corrientes mantenidos para la venta - Activos biológicos,group66,account.data_account_type_expenses,pe_chart_template,False
chart6621,6621,Activo inmovilizado - Propiedades de inversión,group66,account.data_account_type_expenses,pe_chart_template,False
chart6622,6622,Activo inmovilizado - Activos biológicos,group66,account.data_account_type_expenses,pe_chart_template,False
chart6711,6711,Gastos en operaciones de endeudamiento y otros - Préstamos de instituciones financieras y otras entidades,group67,account.data_account_type_expenses,pe_chart_template,False
chart6712,6712,Gastos en operaciones de endeudamiento y otros - Contratos de arrendamiento financiero,group67,account.data_account_type_expenses,pe_chart_template,False
chart6713,6713,Gastos en operaciones de endeudamiento y otros - Emisión y colocación de instrumentos representativos de deuda y patrimonio,group67,account.data_account_type_expenses,pe_chart_template,False
chart6714,6714,Gastos en operaciones de endeudamiento y otros - Documentos vendidos o descontados,group67,account.data_account_type_expenses,pe_chart_template,False
chart672,672,Pérdida por instrumentos financieros derivados,group67,account.data_account_type_expenses,pe_chart_template,False
chart67311,67311,Intereses por préstamos y otras obligaciones - Préstamos de instituciones financieras y otras entidades - Instituciones financieras,group67,account.data_account_type_expenses,pe_chart_template,False
chart67312,67312,Intereses por préstamos y otras obligaciones - Préstamos de instituciones financieras y otras entidades - Otras entidades,group67,account.data_account_type_expenses,pe_chart_template,False
chart6732,6732,Intereses por préstamos y otras obligaciones - Contratos de arrendamiento financiero,group67,account.data_account_type_expenses,pe_chart_template,False
chart6733,6733,Intereses por préstamos y otras obligaciones - Otros instrumentos financieros por pagar,group67,account.data_account_type_expenses,pe_chart_template,False
chart6734,6734,Intereses por préstamos y otras obligaciones - Documentos vendidos o descontados,group67,account.data_account_type_expenses,pe_chart_template,False
chart6735,6735,Intereses por préstamos y otras obligaciones - Obligaciones emitidas,group67,account.data_account_type_expenses,pe_chart_template,False
chart6736,6736,Intereses por préstamos y otras obligaciones - Obligaciones comerciales,group67,account.data_account_type_expenses,pe_chart_template,False
chart6741,6741,Gastos en operaciones de factoraje (factoring) - Pérdida en instrumentos vendidos,group67,account.data_account_type_expenses,pe_chart_template,False
chart675,675,Descuentos concedidos por pronto pago,group67,account.data_account_type_expenses,pe_chart_template,False
chart676,676,Diferencia de cambio,group67,account.data_account_type_expenses,pe_chart_template,False
chart6771,6771,Pérdida por medición de activos y pasivos financieros al valor razonable - Inversiones mantenidas para negociación,group67,account.data_account_type_expenses,pe_chart_template,False
chart6772,6772,Pérdida por medición de activos y pasivos financieros al valor razonable - Otras inversiones financieras,group67,account.data_account_type_expenses,pe_chart_template,False
chart6773,6773,Pérdida por medición de activos y pasivos financieros al valor razonable - Otros,group67,account.data_account_type_expenses,pe_chart_template,False
chart6781,6781,Participación en resultados de entidades relacionadas - Participación en los resultados de subsidiarias y asociadas bajo el método del valor patrimonial,group67,account.data_account_type_expenses,pe_chart_template,False
chart6782,6782,Participación en resultados de entidades relacionadas - Participaciones en negocios conjuntos,group67,account.data_account_type_expenses,pe_chart_template,False
chart6791,6791,Otros gastos financieros - Primas por opciones,group67,account.data_account_type_expenses,pe_chart_template,False
chart6792,6792,Otros gastos financieros - Gastos financieros en medición a valor descontado,group67,account.data_account_type_expenses,pe_chart_template,False
chart6793,6793,Otros gastos financieros - Gastos financieros en actualización de activos por derecho de uso,group67,account.data_account_type_expenses,pe_chart_template,False
chart68111,68111,Depreciación de propiedades de inversión - Edificaciones - Costo,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68112,68112,Depreciación de propiedades de inversión - Edificaciones - Revaluación,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68113,68113,Depreciación de propiedades de inversión - Edificaciones - Costo de financiación,group68,account.data_account_type_depreciation,pe_chart_template,False
chart682111,682111,Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedades de inversión - Edificaciones - Costo,group68,account.data_account_type_depreciation,pe_chart_template,False
chart682112,682112,Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedades de inversión - Edificaciones - Revaluación,group68,account.data_account_type_depreciation,pe_chart_template,False
chart682113,682113,Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedades de inversión - Edificaciones - Costo de financiación,group68,account.data_account_type_depreciation,pe_chart_template,False
chart682211,682211,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Edificaciones - Costo",group68,account.data_account_type_depreciation,pe_chart_template,False
chart682212,682212,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Edificaciones - Revaluación",group68,account.data_account_type_depreciation,pe_chart_template,False
chart682213,682213,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Edificaciones - Costo de financiación",group68,account.data_account_type_depreciation,pe_chart_template,False
chart682221,682221,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Maquinarias y equipos de explotación - Costo",group68,account.data_account_type_depreciation,pe_chart_template,False
chart682222,682222,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Maquinarias y equipos de explotación - Revaluación",group68,account.data_account_type_depreciation,pe_chart_template,False
chart682223,682223,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Maquinarias y equipos de explotación - Costo de financiación",group68,account.data_account_type_depreciation,pe_chart_template,False
chart682231,682231,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Unidades de transporte - Costo",group68,account.data_account_type_depreciation,pe_chart_template,False
chart682232,682232,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Unidades de transporte - Revaluación",group68,account.data_account_type_depreciation,pe_chart_template,False
chart682251,682251,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Equipos diversos - Costo",group68,account.data_account_type_depreciation,pe_chart_template,False
chart682252,682252,"Depreciación de activos por derecho de uso - arrendamiento financiero - Propiedad, planta y equipo - Equipos diversos - Revaluación",group68,account.data_account_type_depreciation,pe_chart_template,False
chart683111,683111,Depreciación de activos por derecho de uso - arrendamiento operativo - Depreciación de activos por derecho de uso - arrendamiento operativo - Edificaciones - Costo,group68,account.data_account_type_depreciation,pe_chart_template,False
chart683112,683112,Depreciación de activos por derecho de uso - arrendamiento operativo - Depreciación de activos por derecho de uso - arrendamiento operativo - Edificaciones - Revaluación,group68,account.data_account_type_depreciation,pe_chart_template,False
chart683121,683121,Depreciación de activos por derecho de uso - arrendamiento operativo - Depreciación de activos por derecho de uso - arrendamiento operativo - Maquinarias y equipos de explotación - Costo,group68,account.data_account_type_depreciation,pe_chart_template,False
chart683122,683122,Depreciación de activos por derecho de uso - arrendamiento operativo - Depreciación de activos por derecho de uso - arrendamiento operativo - Maquinarias y equipos de explotación - Revaluación,group68,account.data_account_type_depreciation,pe_chart_template,False
chart683131,683131,Depreciación de activos por derecho de uso - arrendamiento operativo - Depreciación de activos por derecho de uso - arrendamiento operativo - Unidades de transporte - Costo,group68,account.data_account_type_depreciation,pe_chart_template,False
chart683132,683132,Depreciación de activos por derecho de uso - arrendamiento operativo - Depreciación de activos por derecho de uso - arrendamiento operativo - Unidades de transporte - Revaluación,group68,account.data_account_type_depreciation,pe_chart_template,False
chart683152,683152,Depreciación de activos por derecho de uso - arrendamiento operativo - Depreciación de activos por derecho de uso - arrendamiento operativo - Equipos diversos - Revaluación,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68410,68410,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Plantas productoras",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68411,68411,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Edificaciones",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68412,68412,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Maquinarias y equipos de explotación",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68413,68413,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Unidades de transporte",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68414,68414,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Muebles y enseres",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68415,68415,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Equipos diversos",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68416,68416,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Herramientas y unidades de reemplazo",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68420,68420,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Plantas productoras",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68421,68421,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Edificaciones",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68422,68422,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Maquinarias y equipos de explotación",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68423,68423,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Unidades de transporte",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68424,68424,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Muebles y enseres",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68425,68425,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Equipos diversos",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68426,68426,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Herramientas y unidades de reemplazo",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68430,68430,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costos de financiación - Plantas productoras",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68431,68431,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costos de financiación - Edificaciones",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68432,68432,"Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costos de financiación - Maquinarias y equipos de explotación",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68511,68511,Depreciación de activos biológicos en producción - Depreciación de activos biológicos en producción - costo - Activos biológicos de origen animal,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68512,68512,Depreciación de activos biológicos en producción - Depreciación de activos biológicos en producción - costo - Activos biológicos de origen vegetal,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68521,68521,Depreciación de activos biológicos en producción - Depreciación de activos biológicos en producción - costo de financiación - Activos biológicos de origen animal,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68522,68522,Depreciación de activos biológicos en producción - Depreciación de activos biológicos en producción - costo de financiación - Activos biológicos de origen vegetal,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68611,68611,"Amortización de intangibles - Amortización de intangibles – Costo - Concesiones, licencias y otros derechos",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68612,68612,Amortización de intangibles - Amortización de intangibles – Costo - Patentes y propiedad industrial,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68613,68613,Amortización de intangibles - Amortización de intangibles – Costo - Programas de computadora (software),group68,account.data_account_type_depreciation,pe_chart_template,False
chart68614,68614,Amortización de intangibles - Amortización de intangibles – Costo - Costos de exploración y desarrollo,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68615,68615,"Amortización de intangibles - Amortización de intangibles – Costo - Fórmulas, diseños y prototipos",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68619,68619,Amortización de intangibles - Amortización de intangibles – Costo - Otros activos intangibles,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68621,68621,"Amortización de intangibles - Amortización de intangibles – Revaluación - Concesiones, licencias y otros derechos",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68622,68622,Amortización de intangibles - Amortización de intangibles – Revaluación - Patentes y propiedad industrial,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68623,68623,Amortización de intangibles - Amortización de intangibles – Revaluación - Programas de computadora (software),group68,account.data_account_type_depreciation,pe_chart_template,False
chart68624,68624,Amortización de intangibles - Amortización de intangibles – Revaluación - Costos de exploración y desarrollo,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68625,68625,"Amortización de intangibles - Amortización de intangibles – Revaluación - Fórmulas, diseños y prototipos",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68629,68629,Amortización de intangibles - Amortización de intangibles – Revaluación - Otros activos intangibles,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68711,68711,Valuación de activos - Estimación de cuentas de cobranza dudosa - Cuentas por cobrar comerciales – Terceros,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68712,68712,Valuación de activos - Estimación de cuentas de cobranza dudosa - Cuentas por cobrar comerciales – Relacionadas,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68713,68713,"Valuación de activos - Estimación de cuentas de cobranza dudosa - Cuentas por cobrar al personal, a los accionistas (socios) y directores",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68714,68714,Valuación de activos - Estimación de cuentas de cobranza dudosa - Cuentas por cobrar diversas – Terceros,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68715,68715,Valuación de activos - Estimación de cuentas de cobranza dudosa - Cuentas por cobrar diversas – Relacionadas,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68731,68731,Valuación de activos - Desvalorización de inversiones mobiliarias - Inversiones a ser mantenidas hasta el vencimiento,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68732,68732,Valuación de activos - Desvalorización de inversiones mobiliarias - Instrumentos financieros representativos de derecho patrimonial,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68812,68812,Deterioro del valor de los activos - Desvalorización de propiedad de inversión - Edificaciones,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68813,68813,Deterioro del valor de los activos - Desvalorización de propiedad de inversión - Construcciones en curso,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68820,68820,Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - arrendamiento financiero - Planta productora en producción,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68821,68821,Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - arrendamiento financiero - Planta productora en desarrollo,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68822,68822,Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - arrendamiento financiero - Terrenos,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68823,68823,Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - arrendamiento financiero - Edificaciones,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68824,68824,Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - arrendamiento financiero - Maquinarias y equipos de explotación,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68825,68825,Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - arrendamiento financiero - Unidades de transporte,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68826,68826,Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - arrendamiento financiero - Muebles y enseres,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68827,68827,Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - arrendamiento financiero - Equipos diversos,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68828,68828,Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - arrendamiento financiero - Herramientas y unidades de reemplazo,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68830,68830,"Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Planta productora en producción",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68831,68831,"Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Planta productora en desarrollo",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68832,68832,"Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Terrenos",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68833,68833,"Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Edificaciones",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68834,68834,"Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Maquinarias y equipos de explotación",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68835,68835,"Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Unidades de transporte",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68836,68836,"Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Muebles y enseres",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68837,68837,"Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Equipos diversos",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68838,68838,"Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Herramientas y unidades de reemplazo",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68841,68841,"Deterioro del valor de los activos - Desvalorización de intangibles - Concesiones, licencias y otros derechos",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68842,68842,Deterioro del valor de los activos - Desvalorización de intangibles - Patentes y propiedad industrial,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68843,68843,Deterioro del valor de los activos - Desvalorización de intangibles - Programas de computadora (software),group68,account.data_account_type_depreciation,pe_chart_template,False
chart68844,68844,Deterioro del valor de los activos - Desvalorización de intangibles - Costos de exploración y desarrollo,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68845,68845,"Deterioro del valor de los activos - Desvalorización de intangibles - Fórmulas, diseños y prototipos",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68846,68846,Deterioro del valor de los activos - Desvalorización de intangibles - Otros activos intangibles,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68847,68847,Deterioro del valor de los activos - Desvalorización de intangibles - Plusvalía mercantil,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68891,68891,Deterioro del valor de los activos - Desvalorización de activos biológicos en producción - Activos biológicos de origen animal,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68892,68892,Deterioro del valor de los activos - Desvalorización de activos biológicos en producción - Activos biológicos de origen vegetal,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68911,68911,Provisiones - Provisión para litigios - Provisión para litigios – Costo,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68912,68912,Provisiones - Provisión para litigios - Provisión para litigios – Actualización financiera,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68921,68921,"Provisiones - Provisión por desmantelamiento, retiro o rehabilitación del inmovilizado - Provisión por desmantelamiento, retiro o rehabilitación del inmovilizado – Costo",group68,account.data_account_type_depreciation,pe_chart_template,False
chart68922,68922,"Provisiones - Provisión por desmantelamiento, retiro o rehabilitación del inmovilizado - Provisión por desmantelamiento, retiro o rehabilitación del inmovilizado – Actualización financiera",group68,account.data_account_type_depreciation,pe_chart_template,False
chart6893,6893,Provisiones - Provisión para reestructuraciones,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68941,68941,Provisiones - Provisión para protección y remediación del medio ambiente - Provisión para protección y remediación del medio ambiente – Costo,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68942,68942,Provisiones - Provisión para protección y remediación del medio ambiente - Provisión para protección y remediación del medio ambiente – Actualización financiera,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68961,68961,Provisiones - Provisión para garantías - Provisión para garantías – Costo,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68962,68962,Provisiones - Provisión para garantías - Provisión para garantías – Actualización financiera,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68971,68971,Provisiones - Provisión por activos por derecho de uso - Provisión por activos por derecho de uso arrendamiento operativo,group68,account.data_account_type_depreciation,pe_chart_template,False
chart68972,68972,Provisiones - Provisión por activos por derecho de uso - Provisión por activos por derecho de uso arrendamiento operativo - actualización financiera,group68,account.data_account_type_depreciation,pe_chart_template,False
chart6899,6899,Provisiones - Otras provisiones,group68,account.data_account_type_depreciation,pe_chart_template,False
chart69111,69111,Mercaderías - Mercaderías - exportación - Terceros,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart69112,69112,Mercaderías - Mercaderías - exportación - Relacionadas,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart69121,69121,Mercaderías - Mercaderías - venta local - Terceros,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart69122,69122,Mercaderías - Mercaderías - venta local - Relacionadas,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart69211,69211,Productos terminados - Productos terminados - Exportación - Terceros,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart69212,69212,Productos terminados - Productos terminados - Exportación - Relacionadas,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart69221,69221,Productos terminados - Productos terminados - Venta local - Terceros,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart69222,69222,Productos terminados - Productos terminados - Venta local - Relacionadas,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart69231,69231,Productos terminados - Costos de financiación – Productos terminados - Terceros,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart69232,69232,Productos terminados - Costos de financiación – Productos terminados - Relacionadas,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart6924,6924,Productos terminados - Costos de producción no absorbido – Productos terminados,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart6925,6925,Productos terminados - Costo de ineficiencia – Productos terminados,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart69311,69311,Servicios terminados - Servicios – Exportación - Terceros,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart69312,69312,Servicios terminados - Servicios – Exportación - Relacionadas,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart69321,69321,Servicios terminados - Servicios – local - Terceros,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart69322,69322,Servicios terminados - Servicios – local - Relacionadas,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart69411,69411,"Subproductos, desechos y desperdicios - Subproductos - Terceros",group69,account.data_account_type_direct_costs,pe_chart_template,False
chart69412,69412,"Subproductos, desechos y desperdicios - Subproductos - Relacionadas",group69,account.data_account_type_direct_costs,pe_chart_template,False
chart69421,69421,"Subproductos, desechos y desperdicios - Desechos y desperdicios - Terceros",group69,account.data_account_type_direct_costs,pe_chart_template,False
chart69422,69422,"Subproductos, desechos y desperdicios - Desechos y desperdicios - Relacionadas",group69,account.data_account_type_direct_costs,pe_chart_template,False
chart6951,6951,Gastos por desvalorización de inventarios al costo - Mercaderías,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart6952,6952,Gastos por desvalorización de inventarios al costo - Productos terminados,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart6953,6953,"Gastos por desvalorización de inventarios al costo - Subproductos, desechos y desperdicios",group69,account.data_account_type_direct_costs,pe_chart_template,False
chart6954,6954,Gastos por desvalorización de inventarios al costo - Productos en proceso,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart6955,6955,Gastos por desvalorización de inventarios al costo - Materias primas,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart6956,6956,"Gastos por desvalorización de inventarios al costo - Materiales auxiliares, suministros y repuestos",group69,account.data_account_type_direct_costs,pe_chart_template,False
chart6957,6957,Gastos por desvalorización de inventarios al costo - Envases y embalajes,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart6958,6958,Gastos por desvalorización de inventarios al costo - Inventarios por recibir,group69,account.data_account_type_direct_costs,pe_chart_template,False
chart70111,70111,Mercaderías - Mercaderías - venta de exportación - Terceros,group70,account.data_account_type_revenue,pe_chart_template,False
chart70112,70112,Mercaderías - Mercaderías - venta de exportación - Relacionadas,group70,account.data_account_type_revenue,pe_chart_template,False
chart70121,70121,Mercaderías - Mercaderías - venta local - Terceros,group70,account.data_account_type_revenue,pe_chart_template,False
chart70122,70122,Mercaderías - Mercaderías - venta local - Relacionadas,group70,account.data_account_type_revenue,pe_chart_template,False
chart70211,70211,Productos terminados - Productos terminados - venta de exportación - Terceros,group70,account.data_account_type_revenue,pe_chart_template,False
chart70212,70212,Productos terminados - Productos terminados - venta de exportación - Relacionadas,group70,account.data_account_type_revenue,pe_chart_template,False
chart70221,70221,Productos terminados - Productos terminados - venta local - Terceros,group70,account.data_account_type_revenue,pe_chart_template,False
chart70222,70222,Productos terminados - Productos terminados - venta local - Relacionadas,group70,account.data_account_type_revenue,pe_chart_template,False
chart70311,70311,Servicios terminados - Servicios – exportación - Terceros,group70,account.data_account_type_revenue,pe_chart_template,False
chart70312,70312,Servicios terminados - Servicios – exportación - Relacionadas,group70,account.data_account_type_revenue,pe_chart_template,False
chart70321,70321,Servicios terminados - Servicios – local - Terceros,group70,account.data_account_type_revenue,pe_chart_template,False
chart70322,70322,Servicios terminados - Servicios – local - Relacionadas,group70,account.data_account_type_revenue,pe_chart_template,False
chart70411,70411,"Subproductos, desechos y desperdicios - Subproductos - Terceros",group70,account.data_account_type_revenue,pe_chart_template,False
chart70412,70412,"Subproductos, desechos y desperdicios - Subproductos - Relacionadas",group70,account.data_account_type_revenue,pe_chart_template,False
chart70421,70421,"Subproductos, desechos y desperdicios - Desechos y desperdicios - Terceros",group70,account.data_account_type_revenue,pe_chart_template,False
chart70422,70422,"Subproductos, desechos y desperdicios - Desechos y desperdicios - Relacionadas",group70,account.data_account_type_revenue,pe_chart_template,False
chart70911,70911,Devoluciones sobre ventas - Mercaderías - Venta de exportación - Terceros,group70,account.data_account_type_revenue,pe_chart_template,False
chart70912,70912,Devoluciones sobre ventas - Mercaderías - Venta de exportación - Relacionadas,group70,account.data_account_type_revenue,pe_chart_template,False
chart70921,70921,Devoluciones sobre ventas - Mercaderías - Venta local - Terceros,group70,account.data_account_type_revenue,pe_chart_template,False
chart70922,70922,Devoluciones sobre ventas - Mercaderías - Venta local - Relacionadas,group70,account.data_account_type_revenue,pe_chart_template,False
chart70931,70931,Devoluciones sobre ventas - Productos terminados - Venta de exportación - Terceros,group70,account.data_account_type_revenue,pe_chart_template,False
chart70932,70932,Devoluciones sobre ventas - Productos terminados - Venta de exportación - Relacionadas,group70,account.data_account_type_revenue,pe_chart_template,False
chart70941,70941,Devoluciones sobre ventas - Productos terminados - Venta local - Terceros,group70,account.data_account_type_revenue,pe_chart_template,False
chart70942,70942,Devoluciones sobre ventas - Productos terminados - Venta local - Relacionadas,group70,account.data_account_type_revenue,pe_chart_template,False
chart70951,70951,Devoluciones sobre ventas - Inventarios de servicios rechazados - Terceros,group70,account.data_account_type_revenue,pe_chart_template,False
chart70952,70952,Devoluciones sobre ventas - Inventarios de servicios rechazados - Relacionadas,group70,account.data_account_type_revenue,pe_chart_template,False
chart70961,70961,"Devoluciones sobre ventas - Subproductos, desechos y desperdicios - Terceros",group70,account.data_account_type_revenue,pe_chart_template,False
chart70962,70962,"Devoluciones sobre ventas - Subproductos, desechos y desperdicios - Relacionadas",group70,account.data_account_type_revenue,pe_chart_template,False
chart7111,7111,Variación de productos terminados - Productos terminados,group71,account.data_account_type_other_income,pe_chart_template,False
chart7121,7121,"Variación de subproductos, desechos y desperdicios - Subproductos",group71,account.data_account_type_other_income,pe_chart_template,False
chart7122,7122,"Variación de subproductos, desechos y desperdicios - Desechos y desperdicios",group71,account.data_account_type_other_income,pe_chart_template,False
chart7131,7131,Variación de productos en proceso - Productos en proceso de manufactura,group71,account.data_account_type_other_income,pe_chart_template,False
chart7141,7141,Variación de envases y embalajes - Envases,group71,account.data_account_type_other_income,pe_chart_template,False
chart7142,7142,Variación de envases y embalajes - Embalajes,group71,account.data_account_type_other_income,pe_chart_template,False
chart7151,7151,Variación de inventarios de servicios - Inventarios de servicios en proceso,group71,account.data_account_type_other_income,pe_chart_template,False
chart7211,7211,Propiedades de inversión - Edificaciones,group72,account.data_account_type_other_income,pe_chart_template,False
chart7220,7220,"Propiedad, planta y equipo - Planta productora",group72,account.data_account_type_other_income,pe_chart_template,False
chart7221,7221,"Propiedad, planta y equipo - Edificaciones",group72,account.data_account_type_other_income,pe_chart_template,False
chart7222,7222,"Propiedad, planta y equipo - Maquinarias y otros equipos de explotación",group72,account.data_account_type_other_income,pe_chart_template,False
chart7223,7223,"Propiedad, planta y equipo - Unidades de transporte",group72,account.data_account_type_other_income,pe_chart_template,False
chart7224,7224,"Propiedad, planta y equipo - Muebles y enseres",group72,account.data_account_type_other_income,pe_chart_template,False
chart7225,7225,"Propiedad, planta y equipo - Equipos diversos",group72,account.data_account_type_other_income,pe_chart_template,False
chart7231,7231,Intangibles - Programas de computadora (software),group72,account.data_account_type_other_income,pe_chart_template,False
chart7232,7232,Intangibles - Costos de exploración y desarrollo,group72,account.data_account_type_other_income,pe_chart_template,False
chart7233,7233,"Intangibles - Fórmulas, diseños y prototipos",group72,account.data_account_type_other_income,pe_chart_template,False
chart7241,7241,Activos biológicos - Activos biológicos en desarrollo de origen animal,group72,account.data_account_type_other_income,pe_chart_template,False
chart7242,7242,Activos biológicos - Activos biológicos en desarrollo de origen vegetal,group72,account.data_account_type_other_income,pe_chart_template,False
chart72511,72511,Costos de financiación capitalizados - Costos de financiación – Propiedades de inversión - Plantas productoras en desarrollo,group72,account.data_account_type_other_income,pe_chart_template,False
chart72512,72512,Costos de financiación capitalizados - Costos de financiación – Propiedades de inversión - Edificaciones,group72,account.data_account_type_other_income,pe_chart_template,False
chart72521,72521,"Costos de financiación capitalizados - Costos de financiación – Propiedad, planta y equipo - Plantas productoras en desarrollo",group72,account.data_account_type_other_income,pe_chart_template,False
chart72522,72522,"Costos de financiación capitalizados - Costos de financiación – Propiedad, planta y equipo - Edificaciones",group72,account.data_account_type_other_income,pe_chart_template,False
chart72523,72523,"Costos de financiación capitalizados - Costos de financiación – Propiedad, planta y equipo - Maquinarias y otros equipos de explotación",group72,account.data_account_type_other_income,pe_chart_template,False
chart7253,7253,Costos de financiación capitalizados - Costos de financiación – Intangibles,group72,account.data_account_type_other_income,pe_chart_template,False
chart72541,72541,Costos de financiación capitalizados - Costos de financiación – Activos biológicos en desarrollo - Activos biológicos de origen animal,group72,account.data_account_type_other_income,pe_chart_template,False
chart72542,72542,Costos de financiación capitalizados - Costos de financiación – Activos biológicos en desarrollo - Activos biológicos de origen vegetal,group72,account.data_account_type_other_income,pe_chart_template,False
chart7311,7311,"Descuentos, rebajas y bonificaciones obtenidos - Terceros",group73,account.data_account_type_revenue,pe_chart_template,False
chart7312,7312,"Descuentos, rebajas y bonificaciones obtenidos - Relacionadas",group73,account.data_account_type_revenue,pe_chart_template,False
chart7411,7411,"Descuentos, rebajas y bonificaciones concedidos - Terceros",group74,account.data_account_type_revenue,pe_chart_template,False
chart7412,7412,"Descuentos, rebajas y bonificaciones concedidos - Relacionadas",group74,account.data_account_type_revenue,pe_chart_template,False
chart751,751,Servicios en beneficio del personal,group75,account.data_account_type_other_income,pe_chart_template,False
chart752,752,Comisiones y corretajes,group75,account.data_account_type_other_income,pe_chart_template,False
chart753,753,Regalías,group75,account.data_account_type_other_income,pe_chart_template,False
chart7540,7540,Alquileres - Plantas productoras,group75,account.data_account_type_other_income,pe_chart_template,False
chart7541,7541,Alquileres - Terrenos,group75,account.data_account_type_other_income,pe_chart_template,False
chart7542,7542,Alquileres - Edificaciones,group75,account.data_account_type_other_income,pe_chart_template,False
chart7543,7543,Alquileres - Maquinarias y equipos de explotación,group75,account.data_account_type_other_income,pe_chart_template,False
chart7544,7544,Alquileres - Unidades de transporte,group75,account.data_account_type_other_income,pe_chart_template,False
chart7545,7545,Alquileres - Equipos diversos,group75,account.data_account_type_other_income,pe_chart_template,False
chart7551,7551,Recuperación de cuentas de valuación - Recuperación – Cuentas de cobranza dudosa,group75,account.data_account_type_other_income,pe_chart_template,False
chart7552,7552,Recuperación de cuentas de valuación - Recuperación – Desvalorización de inventarios,group75,account.data_account_type_other_income,pe_chart_template,False
chart7553,7553,Recuperación de cuentas de valuación - Recuperación – Desvalorización de inversiones mobiliarias,group75,account.data_account_type_other_income,pe_chart_template,False
chart7561,7561,Enajenación de activos inmovilizados - Inversiones mobiliarias,group75,account.data_account_type_other_income,pe_chart_template,False
chart7562,7562,Enajenación de activos inmovilizados - Propiedades de inversión,group75,account.data_account_type_other_income,pe_chart_template,False
chart7563,7563,Enajenación de activos inmovilizados - Activos adquiridos en arrendamiento financiero,group75,account.data_account_type_other_income,pe_chart_template,False
chart7564,7564,"Enajenación de activos inmovilizados - Propiedad, planta y equipo",group75,account.data_account_type_other_income,pe_chart_template,False
chart7565,7565,Enajenación de activos inmovilizados - Intangibles,group75,account.data_account_type_other_income,pe_chart_template,False
chart7566,7566,Enajenación de activos inmovilizados - Activos biológicos,group75,account.data_account_type_other_income,pe_chart_template,False
chart7571,7571,Recuperación de deterioro de cuentas de activos inmovilizados - Recuperación de deterioro de propiedades de inversión,group75,account.data_account_type_other_income,pe_chart_template,False
chart7572,7572,"Recuperación de deterioro de cuentas de activos inmovilizados - Recuperación de deterioro de propiedad, planta y equipo",group75,account.data_account_type_other_income,pe_chart_template,False
chart7573,7573,Recuperación de deterioro de cuentas de activos inmovilizados - Recuperación de deterioro de intangibles,group75,account.data_account_type_other_income,pe_chart_template,False
chart7574,7574,Recuperación de deterioro de cuentas de activos inmovilizados - Recuperación de deterioro de activos biológicos,group75,account.data_account_type_other_income,pe_chart_template,False
chart7591,7591,Otros ingresos de gestión - Subsidios gubernamentales,group75,account.data_account_type_other_income,pe_chart_template,False
chart7592,7592,Otros ingresos de gestión - Reclamos al seguro,group75,account.data_account_type_other_income,pe_chart_template,False
chart7593,7593,Otros ingresos de gestión - Donaciones,group75,account.data_account_type_other_income,pe_chart_template,False
chart7594,7594,Otros ingresos de gestión - Devoluciones tributarias,group75,account.data_account_type_other_income,pe_chart_template,False
chart7599,7599,Otros ingresos de gestión - Otros ingresos de gestión,group75,account.data_account_type_other_income,pe_chart_template,False
chart7611,7611,Activo realizable - Mercaderías,group76,account.data_account_type_other_income,pe_chart_template,False
chart7612,7612,Activo realizable - Productos terminados,group76,account.data_account_type_other_income,pe_chart_template,False
chart76131,76131,Activo realizable - Activos no corrientes mantenidos para la venta - Propiedades de inversión,group76,account.data_account_type_other_income,pe_chart_template,False
chart76132,76132,"Activo realizable - Activos no corrientes mantenidos para la venta - Propiedad, planta y equipo",group76,account.data_account_type_other_income,pe_chart_template,False
chart76133,76133,Activo realizable - Activos no corrientes mantenidos para la venta - Intangibles,group76,account.data_account_type_other_income,pe_chart_template,False
chart76134,76134,Activo realizable - Activos no corrientes mantenidos para la venta - Activos biológicos,group76,account.data_account_type_other_income,pe_chart_template,False
chart7621,7621,Activo inmovilizado - Propiedades de inversión,group76,account.data_account_type_other_income,pe_chart_template,False
chart7622,7622,Activo inmovilizado - Activos biológicos,group76,account.data_account_type_other_income,pe_chart_template,False
chart771,771,Ganancia por instrumento financiero derivado,group77,account.data_account_type_other_income,pe_chart_template,False
chart7721,7721,Rendimientos ganados - Depósitos en instituciones financieras,group77,account.data_account_type_other_income,pe_chart_template,False
chart7722,7722,Rendimientos ganados - Cuentas por cobrar comerciales,group77,account.data_account_type_other_income,pe_chart_template,False
chart7723,7723,Rendimientos ganados - Préstamos otorgados,group77,account.data_account_type_other_income,pe_chart_template,False
chart7724,7724,Rendimientos ganados - Inversiones a ser mantenidas hasta el vencimiento,group77,account.data_account_type_other_income,pe_chart_template,False
chart7725,7725,Rendimientos ganados - Instrumentos financieros representativos de derecho patrimonial,group77,account.data_account_type_other_income,pe_chart_template,False
chart773,773,Dividendos,group77,account.data_account_type_other_income,pe_chart_template,False
chart774,774,Ingresos en operaciones de factoraje (factoring),group77,account.data_account_type_other_income,pe_chart_template,False
chart775,775,Descuentos obtenidos por pronto pago,group77,account.data_account_type_other_income,pe_chart_template,False
chart776,776,Diferencia en cambio,group77,account.data_account_type_other_income,pe_chart_template,False
chart7771,7771,Ganancia por medición de activos y pasivos financieros al valor razonable - Inversiones mantenidas para negociación,group77,account.data_account_type_other_income,pe_chart_template,False
chart7772,7772,Ganancia por medición de activos y pasivos financieros al valor razonable - Otras inversiones,group77,account.data_account_type_other_income,pe_chart_template,False
chart7773,7773,Ganancia por medición de activos y pasivos financieros al valor razonable - Otras,group77,account.data_account_type_other_income,pe_chart_template,False
chart7781,7781,Participación en resultados de entidades relacionadas - Participación en los resultados de subsidiarias y asociadas bajo el método del valor patrimonial,group77,account.data_account_type_other_income,pe_chart_template,False
chart7782,7782,Participación en resultados de entidades relacionadas - Ingresos por participaciones en negocios conjuntos,group77,account.data_account_type_other_income,pe_chart_template,False
chart7792,7792,Otros ingresos financieros - Ingresos financieros en medición a valor descontado,group77,account.data_account_type_other_income,pe_chart_template,False
chart781,781,Cargas cubiertas por provisiones,group78,account.data_account_type_other_income,pe_chart_template,False
chart791,791,Cargas imputables a cuentas de costos y gastos,group79,account.data_account_type_revenue,pe_chart_template,False
chart792,792,Gastos financieros imputables a cuentas de inventarios,group79,account.data_account_type_revenue,pe_chart_template,False
chart801,801,Margen comercial,group80,account.data_account_type_equity,pe_chart_template,False
chart811,811,Producción de bienes,group81,account.data_account_type_equity,pe_chart_template,False
chart812,812,Producción de servicios,group81,account.data_account_type_equity,pe_chart_template,False
chart813,813,Producción de activo inmovilizado,group81,account.data_account_type_equity,pe_chart_template,False
chart821,821,Valor agregado,group82,account.data_account_type_equity,pe_chart_template,False
chart831,831,Excedente bruto (insuficiencia bruta) de explotación,group83,account.data_account_type_equity,pe_chart_template,False
chart841,841,Resultado de explotación,group84,account.data_account_type_equity,pe_chart_template,False
chart851,851,Resultado antes del impuesto a las ganancias,group85,account.data_account_type_equity,pe_chart_template,False
chart881,881,Impuesto a las ganancias – Corriente,group88,account.data_account_type_equity,pe_chart_template,False
chart882,882,Impuesto a las ganancias – Diferido,group88,account.data_account_type_equity,pe_chart_template,False
chart891,891,Utilidad,group89,account.data_account_type_equity,pe_chart_template,False
chart892,892,Pérdida,group89,account.data_account_type_equity,pe_chart_template,False

```

## File: data\account.group.csv

```csv
id,code_prefix,name,parent_id:id
group0,0,Cuentas de orden,
group1,1,Activo disponible y exigible,
group2,2,Activo realizable,
group3,3,Activo inmovilizado,
group4,4,Pasivo,
group5,5,Patrimonio Neto,
group6,6,Gastos por naturaleza,
group7,7,Ingresos,
group8,8,Saldos intrermediarios de gestión y determinación del resultado del ejercicio,
group01,01,Cuentas de Orden,group0
group02,02,Cuentas de Orden,group0
group10,10,EFECTIVO Y EQUIVALENTES DE EFECTIVO,group1
group11,11,INVERSIONES FINANCIERAS,group1
group12,12,CUENTAS POR COBRAR COMERCIALES – TERCEROS,group1
group13,13,CUENTAS POR COBRAR COMERCIALES – RELACIONADAS,group1
group14,14,"CUENTAS POR COBRAR AL PERSONAL, A LOS ACCIONISTAS (SOCIOS) y
DIRECTORES",group1
group16,16,CUENTAS POR COBRAR DIVERSAS – TERCEROS,group1
group17,17,CUENTAS POR COBRAR DIVERSAS – RELACIONADAS,group1
group18,18,SERVICIOS Y OTROS CONTRATADOS POR ANTICIPADO,group1
group19,19,ESTIMACIÓN DE CUENTAS DE COBRANZA DUDOSA,group1
group20,20,MERCADERÍAS,group2
group21,21,PRODUCTOS TERMINADOS,group2
group22,22,"SUBPRODUCTOS, DESECHOS Y DESPERDICIOS",group2
group23,23,PRODUCTOS EN PROCESO,group2
group24,24,MATERIAS PRIMAS,group2
group25,25,"MATERIALES AUXILIARES, SUMINISTROS Y REPUESTOS",group2
group26,26,ENVASES Y EMBALAJES,group2
group27,27,ACTIVOS NO CORRIENTES MANTENIDOS PARA LA VENTA,group2
group28,28,INVENTARIOS POR RECIBIR,group2
group29,29,DESVALORIZACIÓN DE INVENTARIOS,group2
group30,30,INVERSIONES MOBILIARIAS,group3
group31,31,PROPIEDADES DE INVERSIÓN,group3
group32,32,ACTIVOS POR DERECHO DE USO,group3
group33,33,"PROPIEDAD, PLANTA Y EQUIPO",group3
group34,34,INTANGIBLES,group3
group35,35,ACTIVOS BIOLÓGICOS,group3
group36,36,DESVALORIZACIÓN DE ACTIVO INMOVILIZADO,group3
group37,37,ACTIVO DIFERIDO,group3
group38,38,OTROS ACTIVOS,group3
group39,39,DEPRECIACIÓN y AMORTIZACIÓN ACUMULADOS,group3
group40,40,"TRIBUTOS, CONTRAPRESTACIONES Y APORTES AL SISTEMA PÚBLICO DE
PENSIONES Y DE SALUD POR PAGAR",group4
group41,41,REMUNERACIONES Y PARTICIPACIONES POR PAGAR,group4
group42,42,CUENTAS POR PAGAR COMERCIALES TERCEROS,group4
group43,43,CUENTAS POR PAGAR COMERCIALES RELACIONADAS,group4
group44,44,"CUENTAS POR PAGAR A LOS ACCIONISTAS (SOCIOS, PARTÍCIPES) Y
DIRECTORES",group4
group45,45,OBLIGACIONES FINANCIERAS,group4
group46,46,CUENTAS POR PAGAR DIVERSAS – TERCEROS,group4
group47,47,CUENTAS POR PAGAR DIVERSAS – RELACIONADAS,group4
group48,48,PROVISIONES,group4
group49,49,PASIVO DIFERIDO,group4
group50,50,CAPITAL,group5
group51,51,ACCIONES DE INVERSIÓN,group5
group52,52,CAPITAL ADICIONAL,group5
group56,56,RESULTADOS NO REALIZADOS,group5
group57,57,EXCEDENTE DE REVALUACIÓN,group5
group58,58,RESERVAS,group5
group59,59,RESULTADOS ACUMULADOS,group5
group60,60,COMPRAS,group6
group61,61,VARIACIÓN DE INVENTARIOS,group6
group62,62,GASTOS DE PERSONAL Y DIRECTORES,group6
group63,63,GASTOS DE SERVICIOS PRESTADOS POR TERCEROS,group6
group64,64,GASTOS POR TRIBUTOS,group6
group65,65,OTROS GASTOS DE GESTION,group6
group66,66,PERDIDA POR MEDICIÓN DE ACTIVOS NO FINANCIEROS AL VALOR RAZONABLE,group6
group67,67,GASTOS FINANCIEROS,group6
group68,68,VALUACIÓN Y DETERIORO DE ACTIVOS Y PROVISIONES,group6
group69,69,COSTO DE VENTAS,group6
group70,70,VENTAS,group7
group71,71,VARIACIÓN DE LA PRODUCCIÓN ALMACENADA,group7
group72,72,PRODUCCIÓN DE ACTIVO INMOVILIZADO,group7
group73,73,"DESCUENTOS, REBAJAS Y BONIFICACIONES OBTENIDOS",group7
group74,74,"DESCUENTOS, REBAJAS y BONIFICACIONES CONCEDIDOS",group7
group75,75,OTROS INGRESOS DE GESTIÓN,group7
group76,76,GANANCIA POR MEDICIÓN DE ACTIVOS NO FINANCIEROS AL VALOR RAZONABLE,group7
group77,77,INGRESOS FINANCIEROS,group7
group78,78,CARGAS CUBIERTAS POR PROVISIONES,group7
group79,79,CARGAS IMPUTABLES A CUENTAS DE COSTOS Y GASTOS,group7
group80,80,MARGEN COMERCIAL,group8
group81,81,PRODUCCIÓN DEL EJERCICIO,group8
group82,82,VALOR AGREGADO,group8
group83,83,EXCEDENTE BRUTO (INSUFICIENCIA BRUTA) DE EXPLOTACIÓN,group8
group84,84,RESULTADO DE EXPLOTACIÓN,group8
group85,85,RESULTADO ANTES DE PARTICIPACIONES E IMPUESTOS,group8
group88,88,IMPUESTO A LA RENTA,group8
group89,89,DETERMINACIÓN DEL RESULTADO DEL EJERCICIO,group8

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
    <!-- ACCOUNT TAX GROUP -->
    <!-- TODO AFFECT SUBSEQUENT -->
    <record id="tax_group_igv" model="account.tax.group">
        <field name="name">IGV</field>
        <field name="sequence">0</field>
    </record>
    <record id="tax_group_ivap" model="account.tax.group">
        <field name="name">IVAP</field>
        <field name="sequence">0</field>
    </record>
    <record id="tax_group_isc" model="account.tax.group">
        <field name="name">ISC</field>
        <field name="sequence">0</field>
    </record>
    <record id="tax_group_exp" model="account.tax.group">
        <field name="name">EXP</field>
        <field name="sequence">0</field>
    </record>
    <record id="tax_group_gra" model="account.tax.group">
        <field name="name">GRA</field>
        <field name="sequence">0</field>
    </record>
    <record id="tax_group_exo" model="account.tax.group">
        <field name="name">EXO</field>
        <field name="sequence">0</field>
    </record>
    <record id="tax_group_ina" model="account.tax.group">
        <field name="name">INA</field>
        <field name="sequence">0</field>
    </record>
    <record id="tax_group_other" model="account.tax.group">
        <field name="name">OTROS</field>
        <field name="sequence">0</field>
    </record>
    <record id="tax_group_det" model="account.tax.group">
        <field name="name">DET</field>
        <field name="sequence">100</field>
    </record>
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
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
                'account_id': ref('chart40111'),
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
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
                'account_id': ref('chart40111'),
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart40111'),
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
                'account_id': ref('chart40111'),
            }),
        ]"/>
    </record>

</odoo>

```

## File: data\l10n_latam_identification_type_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model='l10n_latam.identification.type' id='l10n_latam_base.it_vat'>
        <field name='l10n_pe_vat_code'>6</field>
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
        <field name="income_currency_exchange_account_id" ref="chart676"/>
        <field name="expense_currency_exchange_account_id" ref="chart776"/>
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
                    <field name="parent_id" invisible="1"/>
                    <field name="type" invisible="1"/>
                    <field name="street_name" placeholder="Street Name..." attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}" class="oe_read_only"/>
                    <field name="street" placeholder="Street" class="oe_edit_only"/>
                    <field name="street2" placeholder="Street2" invisible="1"/>
                    <div class="o_row">
                        <label for="street_number" class="oe_edit_only"/>
                        <field name="street_number" attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                        <span> </span>
                        <label for="street_number2" class="oe_edit_only"/>
                        <field name="street_number2" attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    </div>
                    <field name="l10n_pe_district" placeholder="District..."/>
                    <field name="city" placeholder="Province..." invisible="1"/>
                    <field name="city_id" placeholder="Province..."/>
                    <field name="state_id" class="o_address_state" placeholder="State..." options='{"no_open": True}'/>
                    <field name="country_id" placeholder="Country" class="o_address_country" options='{"no_open": True, "no_create": True}'/>
                    <field name="zip" placeholder="ZIP" class="o_address_zip"/>
                </div>
            </form>
        </field>
    </record>
    <record id="base.pe" model="res.country">
        <field name="enforce_cities" eval="1" />
        <field name="address_view_id" ref="pe_partner_address_form" />
        <field name="address_format" eval="'%(street)s\n%(city)s\n%(state_name)s\n%(country_name)s'"/>
        <field name="street_format" eval="'%(street_name)s %(street_number)s, %(street_number2)s'"/>
    </record>
</odoo>

```

## File: models\account_journal.py

```python
from odoo import models, api


class AccountJournal(models.Model):
    _inherit = "account.journal"

    @api.model
    def _get_sequence_prefix(self, code, refund=False):
        """For peruvian companies we can not use sequences with **/** due to the edi generation which need in the
        sequence a plain text ended by a *-* and the length of this prefix"""
        if self.env.company.country_id != self.env.ref('base.pe'):
            return super()._get_sequence_prefix(code, refund=refund)
        prefix = code.upper()
        if len(prefix) > 3:
            prefix = prefix[:3]
        prefix = prefix.ljust(3, 'X')
        if refund:
            prefix = 'R' + prefix[:-1]
        return prefix + '-'

    @api.model
    def _create_sequence(self, vals, refund=False):
        """For Peruvian companies, a number reset by date do not make sense due to the fact that we can not use a free
        prefix the format does not have enough space to put a year on it or any other char, then with this approach we
        are avoiding the default behavior there."""
        res = super()._create_sequence(vals, refund=refund)
        # NOTE: the self element is coming filled just when write and not on create (which is Ok)
        journal_type = self.type if not vals.get('type') else vals.get('type')
        if self.env.company.country_id == self.env.ref('base.pe') or journal_type in ['sale', 'purchase']:
            res.write({'use_date_range': False})
        return res

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields


class AccountMoveLine(models.Model):
    _inherit = "account.move.line"

    l10n_pe_group_id = fields.Many2one("account.group", related="account_id.group_id", store=True)

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


class AccountTaxTemplate(models.Model):
    _inherit = "account.tax.template"

    l10n_pe_edi_tax_code = fields.Selection([
        ('1000', 'IGV - General Sales Tax'),
        ('1016', 'IVAP - Tax on Sale Paddy Rice'),
        ('2000', 'ISC - Selective Excise Tax'),
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

    def _get_tax_vals(self, company, tax_template_to_tax):
        val = super()._get_tax_vals(company, tax_template_to_tax)
        val.update({
            'l10n_pe_edi_tax_code': self.l10n_pe_edi_tax_code,
            'l10n_pe_edi_unece_category': self.l10n_pe_edi_unece_category,
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

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details
from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_pe_district = fields.Many2one(
        'l10n_pe.res.city.district', string='District',
        help='Districts are part of a province or city.')
```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_tax
from . import account_move
from . import account_journal
from . import l10n_latam_identification_type
from . import res_partner
from . import res_city_district
from . import res_city

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
"access_l10n_pe_district_group_user","access_l10n_pe_district group_user","model_l10n_pe_res_city_district",base.group_user,1,1,1,1
"access_l10n_pe_district_group_all","access_l10n_pe_district group_all","model_l10n_pe_res_city_district",,1,0,0,0

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
                <field name="l10n_pe_edi_tax_code"/>
                <field name="l10n_pe_edi_unece_category"/>
            </xpath>
        </field>
    </record>
</odoo>

```

