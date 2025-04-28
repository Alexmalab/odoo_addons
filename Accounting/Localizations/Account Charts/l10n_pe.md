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
'OTHERS': {'name': 'OTH', 'code': 'S'},

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
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import demo

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Peru - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['pe'],
    'version': '3.0',
    'category': 'Accounting/Localizations/Account Charts',
    'author': 'Vauxoo, Odoo S.A.',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations/peru.html',
    'license': 'LGPL-3',
    'depends': [
        'base_vat',
        'base_address_extended',
        'l10n_latam_base',
        'l10n_latam_invoice_document',
        'account_debit_note',
        'account',
    ],
    'data': [
        'security/ir.model.access.csv',
        'views/account_tax_view.xml',
        'data/l10n_latam_document_type_data.xml',
        'data/res.city.csv',
        'data/l10n_pe.res.city.district.csv',
        'data/res_country_data.xml',
        'data/l10n_latam_identification_type_data.xml',
        'data/res.bank.csv',
    ],
    'demo': [
        'demo/demo_company.xml',
        'demo/demo_partner.xml',
    ],
}

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

## File: data\res.bank.csv

```csv
id,name,bic,country
peruvian_national_bank,Banco de la nación,BANCPEPL,Peru
peruvian_baiopep1_bank,Banco Iberoamericano,BAIOPEP1,Peru
peruvian_bconpepl_bank,Banco Continental,BCONPEPL,Peru
peruvian_bcplpepl_bank,Banco De Credito Del Peru,BCPLPEPL,Peru
peruvian_bdcmpepl_bank,Banco De Comercio,BDCMPEPL,Peru
peruvian_belppepl_bank,Cetco S.a.,BELPPEPL,Peru
peruvian_bifspepl_bank,Banco Interamericano De Finanzas,BIFSPEPL,Peru
peruvian_binppepl_bank,Banco Internacional Del Peru,BINPPEPL,Peru
peruvian_bnpepep1_bank,Banco Nor Peru,BNPEPEP1,Peru
peruvian_bsappepl_bank,Banco Santander Peru S.a.,BSAPPEPL,Peru
peruvian_bsudpepl_bank,Scotiabank Peru,BSUDPEPL,Peru
peruvian_cbolpep1_bank,Credibolsa Sociedad Agente De Bolsa S.a.,CBOLPEP1,Peru
peruvian_citipep1_bank,Citibank Del Peru Sa,CITIPEP1,Peru
peruvian_citipepl_bank,Citibank Del Peru S.a.,CITIPEPL,Peru
peruvian_cjsipep1_bank,Caja Rural De Ahorro Y Credito Sipan S.a.,CJSIPEP1,Peru
peruvian_cofdpepl_bank,Corporacion Financiera De Desarrollo S.A.,COFDPEPL,Peru
peruvian_crhcpep1_bank,Caja Rural De Ahorro Y Credito Chavin Saa,CRHCPEP1,Peru
peruvian_crhcpep1001_bank,Caja Rural De Ahorro Y Credito Chavin Saa,CRHCPEP1001,Peru

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
                           readonly="type == 'contact' and parent_id"/>
                    <field name="street2" placeholder="Street 2..." class="o_address_street"
                           readonly="type == 'contact' and parent_id"/>
                    <field name="l10n_pe_district" placeholder="District..." class="o_address_street"
                           readonly="type == 'contact' and parent_id"/>
                    <field name="city_id"
                           placeholder="City"
                           class="o_address_city"
                           domain="[('country_id', '=', country_id)]"
                           invisible="not country_enforce_cities"
                           readonly="type == 'contact' and parent_id"
                           context="{'default_country_id': country_id, 'default_state_id': state_id, 'default_zipcode': zip}"/>
                    <field name="city"
                           placeholder="City"
                           class="o_address_city"
                           invisible="country_enforce_cities and (city_id or city in ['', False])"
                           readonly="type == 'contact' and parent_id"/>
                    <field name="state_id" class="o_address_state" placeholder="State" options="{'no_open': True, 'no_quick_create': True}"
                           context="{'default_country_id': country_id}"
                           readonly="type == 'contact' and parent_id"/>
                    <field name="zip" placeholder="ZIP" class="o_address_zip"
                           readonly="type == 'contact' and parent_id"/>
                    <field name="country_id" placeholder="Country" class="o_address_country" options='{"no_open": True, "no_create": True}'
                           readonly="type == 'contact' and parent_id"/>
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

## File: data\template\account.account-pe.csv

```csv
"id","code","name","account_type","reconcile","name@es"
"chart0111","0111","Goods and securities delivered","off_balance","False","Bienes y valores entregados"
"chart0112","0112","Rights on financial instruments","off_balance","False","Derechos sobre instrumentos financiero"
"chart0113","0113","Other debit memorandum accounts","off_balance","False","Otras cuentas de orden deudoras"
"chart0114","0114","Debitors offset","off_balance","False","Deudoras por contra"
"chart0126","0126","Goods and values received","off_balance","False","Bienes y valores recibidos"
"chart0127","0127","Commitments on financial instruments","off_balance","False","Compromisos sobre instrumentos financieros"
"chart0128","0128","Other credit memorandum accounts","off_balance","False","Otras cuentas de orden acreedoras"
"chart0129","0129","Creditors offset","off_balance","False","Acreedoras por contra"
"chart0211","0211","Goods and securities delivered","off_balance","False","Bienes y valores entregados"
"chart0213","0213","Rights on financial instruments","off_balance","False","Derechos sobre instrumentos financiero"
"chart0214","0214","Other debit memorandum accounts","off_balance","False","Otras cuentas de orden deudoras"
"chart0215","0215","Offsetting accounts debit memorandum","off_balance","False","Contrapartida cuentas de orden deudora"
"chart0226","0226","Goods and values received","off_balance","False","Bienes y valores recibidos"
"chart0227","0227","Commitments on financial instruments","off_balance","False","Compromisos sobre instrumentos financieros"
"chart0228","0228","Other credit memorandum accounts","off_balance","False","Otras cuentas de orden acreedoras"
"chart0229","0229","Counterpart creditor memorandum accounts","off_balance","False","Contrapartidacuentas de orden acreedoras"
"chart101","101","Cash","asset_cash","False","Caja"
"chart102","102","Fixed funds","asset_cash","False","Fondos fijos"
"chart1031","1031","Fixed funds - Cash in transit","asset_cash","False","Fondos fijos - Efectivo en tránsito"
"chart1032","1032","Fixed funds - Checks in transit","asset_cash","False","Fondos fijos - Cheques en tránsito"
"chart1041","1041","Current accounts in financial institutions - Operating checking accounts","asset_cash","False","Cuentas corrientes en instituciones financieras - Cuentas corrientes operativas"
"chart1042","1042","Current accounts in financial institutions - Current accounts for specific purposes","asset_cash","False","Cuentas corrientes en instituciones financieras - Cuentas corrientes para fines específicos"
"chart1051","1051","Other cash equivalents - Other cash equivalents","asset_cash","False","Otros equivalentes de efectivo - Otro equivalentes de efectivo"
"chart1061","1061","Deposits in financial institutions - Savings deposits","asset_cash","False","Depósitos en instituciones financieras - Depósitos de ahorro"
"chart1062","1062","Deposits in financial institutions - Time deposits","asset_cash","False","Depósitos en instituciones financieras - Depósitos a plazo"
"chart1071","1071","Restricted Funds - Funds in guarantee","asset_cash","False","Fondos sujetos a restricción - Fondos en garantía"
"chart1072","1072","Restricted Funds - Funds withheld by mandate of the authority","asset_cash","False","Fondos sujetos a restricción - Fondos retenidos por mandato de la autoridad"
"chart1073","1073","Restricted Funds - Other restricted funds","asset_cash","False","Fondos sujetos a restricción - Otros fondos sujetos a restricción"
"chart11111","11111","Investments held for trading - Securities issued or guaranteed by the State - Costs","asset_current","True","Inversiones mantenidas para negociación - Valores emitidos o garantizados por el Estado - Costo"
"chart11112","11112","Investments held for trading - Securities issued or guaranteed by the State - Fair value","asset_current","True","Inversiones mantenidas para negociación - Valores emitidos o garantizados por el Estado - Valor razonable"
"chart11121","11121","Investments held for trading - Securities issued by the financial system - Costs","asset_current","True","Inversiones mantenidas para negociación - Valores emitidos por el sistema financiero - Costo"
"chart11122","11122","Investments held for trading - Securities issued by the financial system - Fair value","asset_current","True","Inversiones mantenidas para negociación - Valores emitidos por el sistema financiero - Valor razonable"
"chart11131","11131","Investments held for trading - Securities issued by entities - Costs","asset_current","True","Inversiones mantenidas para negociación - Valores emitidos por entidades - Costo"
"chart11132","11132","Investments held for trading - Securities issued by entities - Fair value","asset_current","True","Inversiones mantenidas para negociación - Valores emitidos por entidades - Valor razonable"
"chart11141","11141","Investments held for trading - Other debt instruments - Costs","asset_current","True","Inversiones mantenidas para negociación - Otros títulos representativos de deuda - Costo"
"chart11142","11142","Investments held for trading - Other debt instruments - Fair value","asset_current","True","Inversiones mantenidas para negociación - Otros títulos representativos de deuda - Valor razonable"
"chart11151","11151","Investments held for trading - Holdings in entities - Costs","asset_current","True","Inversiones mantenidas para negociación - Participaciones en entidades - Costo"
"chart11152","11152","Investments held for trading - Holdings in entities - Fair value","asset_current","True","Inversiones mantenidas para negociación - Participaciones en entidades - Valor razonable"
"chart11211","11211","Other financial investments - Other financial investments - Costs","asset_current","True","Otras inversiones financieras - Otras inversiones financieras - Costo"
"chart11212","11212","Other financial investments - Other financial investments - Fair value","asset_current","True","Otras inversiones financieras - Otras inversiones financieras - Valor Razonable"
"chart11311","11311","Financial assets – Purchase agreements - Investments held for trading – Purchase agreements - Costs","asset_current","True","Activos financieros – Acuerdo de compra - Inversiones mantenidas para negociación – Acuerdo de compra - Costo"
"chart11312","11312","Financial assets – Purchase agreements - Investments held for trading – Purchase agreements - Fair value","asset_current","True","Activos financieros – Acuerdo de compra - Inversiones mantenidas para negociación – Acuerdo de compra - Valor Razonable"
"chart11321","11321","Financial assets – Purchase agreements - Other financial investments - Costs","asset_current","True","Activos financieros – Acuerdo de compra - Otras inversiones financieras - Costo"
"chart11322","11322","Financial assets – Purchase agreements - Other financial investments - Fair value","asset_current","True","Activos financieros – Acuerdo de compra - Otras inversiones financieras - Valor Razonable"
"chart1211","1211","Invoices, tickets and other receipts receivable - Not issued","asset_receivable","True","Facturas, boletas y otros comprobantes por cobrar - No emitidas"
"chart1212","1212","Invoices, tickets and other receipts receivable - Issued in portfolio","asset_receivable","True","Facturas, boletas y otros comprobantes por cobrar - Emitidas en cartera"
"chart1213","1213","Invoices, tickets and other receipts receivable - In collection","asset_receivable","True","Facturas, boletas y otros comprobantes por cobrar - En cobranza"
"chart1214","1214","Invoices, tickets and other receipts receivable - Off sale","asset_receivable","True","Facturas, boletas y otros comprobantes por cobrar - En descuento"
"chart1215","1215","Accounts receivable ... - Invoices, tickets and other receipts receivable not issued (PoS)","asset_receivable","True","Cuentas por cobrar ... - Facturas, boletas y otros comprobantes por cobrar / no emitidas (PoS)"
"chart122","122","Advances from customers","asset_receivable","True","Anticipos de clientes"
"chart1232","1232","Bills to be collected - In portfolio","asset_receivable","True","Letras por cobrar - En cartera"
"chart1233","1233","Bills to be collected - In collection","asset_receivable","True","Letras por cobrar - En cobranza"
"chart1234","1234","Bills to be collected - Off sale","asset_receivable","True","Letras por cobrar - En descuento"
"chart1311","1311","Invoices, tickets and other receipts receivable - Not issued","asset_receivable","True","Facturas, boletas y otros comprobantes por cobrar - No emitidas"
"chart1312","1312","Invoices, tickets and other receipts receivable - In portfolio","asset_receivable","True","Facturas, boletas y otros comprobantes por cobrar - En cartera"
"chart1313","1313","Invoices, tickets and other receipts receivable - In collection","asset_receivable","True","Facturas, boletas y otros comprobantes por cobrar - En cobranza"
"chart1314","1314","Invoices, tickets and other receipts receivable - Off sale","asset_receivable","True","Facturas, boletas y otros comprobantes por cobrar - En descuento"
"chart1321","1321","Advances received - Advances received","asset_receivable","True","Anticipos recibidos - Anticipos recibidos"
"chart1331","1331","Bills to be collected - In portfolio","asset_receivable","True","Letras por cobrar - En cartera"
"chart1332","1332","Bills to be collected - In collection","asset_receivable","True","Letras por cobrar - En cobranza"
"chart1333","1333","Bills to be collected - Off sale","asset_receivable","True","Letras por cobrar - En descuento"
"chart1411","1411","Staff - Loans","asset_receivable","True","Personal - Préstamos"
"chart1412","1412","Staff - Salary advance","asset_receivable","True","Personal - Adelanto de remuneraciones"
"chart1413","1413","Staff - Deliveries to account","asset_receivable","True","Personal - Entregas a rendir cuenta"
"chart1419","1419","Staff - Other accounts receivable from staff","asset_receivable","True","Personal - Otras cuentas por cobrar al personal"
"chart1421","1421","Shareholders (or partners) - Subscriptions receivable from partners or shareholders","asset_receivable","True","Accionistas (o socios) - Suscripciones por cobrar a socios o accionistas"
"chart1422","1422","Shareholders (or partners) - Loans","asset_receivable","True","Accionistas (o socios) - Préstamos"
"chart1431","1431","Directors - Loans","asset_receivable","True","Directores - Préstamos"
"chart1432","1432","Directors - Advance of daily subsistence allowance","asset_receivable","True","Directores - Adelanto de dietas"
"chart1433","1433","Directors - Deliveries to account","asset_receivable","True","Directores - Entregas a rendir cuenta"
"chart149","149","Various","asset_receivable","True","Diversas"
"chart1611","1611","Loans - With guarantee","asset_receivable","True","Préstamos - Con garantía"
"chart1612","1612","Loans - Whithout garantee","asset_receivable","True","Préstamos - Sin garantía"
"chart1621","1621","Claims to third parties - Insurance companies","asset_receivable","True","Reclamaciones a terceros - Compañías aseguradoras"
"chart1622","1622","Claims to third parties - Transportation","asset_receivable","True","Reclamaciones a terceros - Transportadoras"
"chart1623","1623","Claims to third parties - Public services","asset_receivable","True","Reclamaciones a terceros - Servicios públicos"
"chart1624","1624","Claims to third parties - Taxes","asset_receivable","True","Reclamaciones a terceros - Tributos"
"chart1629","1629","Claims to third parties - Others","asset_receivable","True","Reclamaciones a terceros - Otras"
"chart1631","1631","Interest, royalties and dividends - Interests","asset_receivable","True","Intereses, regalías y dividendos - Intereses"
"chart1632","1632","Interest, royalties and dividends - Royalties","asset_receivable","True","Intereses, regalías y dividendos - Regalías"
"chart1633","1633","Interest, royalties and dividends - Dividends","asset_receivable","True","Intereses, regalías y dividendos - Dividendos"
"chart1641","1641","Deposits granted in guarantee - Loans from financial institutions","asset_receivable","True","Depósitos otorgados en garantía - Préstamos de instituciones financieras"
"chart1642","1642","Deposits granted in guarantee - Loans from non-financial institutions","asset_receivable","True","Depósitos otorgados en garantía - Préstamos de instituciones no financieras"
"chart1643","1643","Deposits granted in guarantee - Security deposits for rentals","asset_receivable","True","Depósitos otorgados en garantía - Depósitos en garantía por alquileres"
"chart1649","1649","Deposits granted in guarantee - Other security deposits","asset_receivable","True","Depósitos otorgados en garantía - Otros depósitos en garantía"
"chart1651","1651","Sale of fixed assets - Property investment","asset_receivable","True","Venta de activo inmovilizado - Inversión mobiliaria"
"chart1652","1652","Sale of fixed assets - Investment property","asset_receivable","True","Venta de activo inmovilizado - Propiedades de inversión"
"chart1653","1653","Sale of fixed assets - Property, plant and equipment","asset_receivable","True","Venta de activo inmovilizado - Propiedad, planta y equipo"
"chart1654","1654","Sale of fixed assets - Intangibles","asset_receivable","True","Venta de activo inmovilizado - Intangibles"
"chart1655","1655","Sale of fixed assets - Biological assets","asset_receivable","True","Venta de activo inmovilizado - Activos biológicos"
"chart1659","1659","Sale of fixed assets - Otros activos inmovilizados","asset_receivable","True","Venta de activo inmovilizado - Otros activos inmovilizados"
"chart16611","16611","Assets for financial instruments - Primary financial instruments - Costs","asset_receivable","True","Activos por instrumentos financieros - Instrumentos financieros primarios - Costos"
"chart16612","16612","Assets for financial instruments - Primary financial instruments - Fair value","asset_receivable","True","Activos por instrumentos financieros - Instrumentos financieros primarios - Valor razonable"
"chart16621","16621","Assets for financial instruments - Derivative financial instruments - Costs","asset_receivable","True","Activos por instrumentos financieros - Instrumentos financieros derivados - Costo"
"chart16622","16622","Assets for financial instruments - Derivative financial instruments - Fair value","asset_receivable","True","Activos por instrumentos financieros - Instrumentos financieros derivados - Valor razonable"
"chart1671","1671","Taxes to be credited - Payments on account of income tax","asset_receivable","True","Tributos por acreditar - Pagos a cuenta del impuesto a la renta"
"chart1672","1672","Taxes to be credited - ITAN account payments","asset_receivable","True","Tributos por acreditar - Pagos a cuenta de ITAN"
"chart1673","1673","Taxes to be credited - IGV to be credited on purchases","asset_receivable","True","Tributos por acreditar - IGV por acreditar en compras"
"chart1674","1674","Taxes to be credited - IGV to credit non-residents","asset_receivable","True","Tributos por acreditar - IGV por acreditar no domiciliados"
"chart1675","1675","Taxes to be credited - Works for taxes","asset_receivable","True","Tributos por acreditar - Obras por impuestos"
"chart1691","1691","Other sundry accounts receivable - Deliveries to account to third parties","asset_receivable","True","Otras cuentas por cobrar diversas - Entregas a rendir cuenta a terceros"
"chart1699","1699","Other sundry accounts receivable - Other sundry accounts receivable","asset_receivable","True","Otras cuentas por cobrar diversas - Otras cuentas por cobrar diversas"
"chart1711","1711","Loans - With guarantee","asset_receivable","True","Préstamos - Con garantía"
"chart1712","1712","Loans - Whithout garantee","asset_receivable","True","Préstamos - Sin garantía"
"chart1731","1731","Interest, royalties and dividends - Interests","asset_receivable","True","Intereses, regalías y dividendos - Intereses"
"chart1732","1732","Interest, royalties and dividends - Royalties","asset_receivable","True","Intereses, regalías y dividendos - Regalías"
"chart1733","1733","Interest, royalties and dividends - Dividends","asset_receivable","True","Intereses, regalías y dividendos - Dividendos"
"chart1741","1741","Deposits granted in guarantee - Loans from financial institutions","asset_receivable","True","Depósitos otorgados en garantía - Préstamos de instituciones financieras"
"chart1742","1742","Deposits granted in guarantee - Loans from non-financial institutions","asset_receivable","True","Depósitos otorgados en garantía - Préstamos de instituciones no financieras"
"chart1743","1743","Deposits granted in guarantee - Security deposits for rentals","asset_receivable","True","Depósitos otorgados en garantía - Depósitos en garantía por alquileres"
"chart1749","1749","Deposits granted in guarantee - Other security deposits","asset_receivable","True","Depósitos otorgados en garantía - Otros depósitos en garantía"
"chart1751","1751","Sale of fixed assets - Property investment","asset_receivable","True","Venta de activo inmovilizado - Inversión mobiliaria"
"chart1752","1752","Sale of fixed assets - Investment property","asset_receivable","True","Venta de activo inmovilizado - Propiedades de inversión"
"chart1753","1753","Sale of fixed assets - Property, plant and equipment","asset_receivable","True","Venta de activo inmovilizado - Propiedad, planta y equipo"
"chart1754","1754","Sale of fixed assets - Intangibles","asset_receivable","True","Venta de activo inmovilizado - Intangibles"
"chart1755","1755","Sale of fixed assets - Biological assets","asset_receivable","True","Venta de activo inmovilizado - Activos biológicos"
"chart1759","1759","Sale of fixed assets - Otros activos inmovilizados","asset_receivable","True","Venta de activo inmovilizado - Otros activos inmovilizados"
"chart17611","17611","Assets for financial instruments - Primary financial instruments - Costs","asset_receivable","True","Activos por instrumentos financieros - Instrumentos financieros primarios - Costos"
"chart17612","17612","Assets for financial instruments - Primary financial instruments - Fair value","asset_receivable","True","Activos por instrumentos financieros - Instrumentos financieros primarios - Valor razonable"
"chart17621","17621","Assets for financial instruments - Derivative financial instruments - Costs","asset_receivable","True","Activos por instrumentos financieros - Instrumentos financieros derivados - Costo"
"chart17622","17622","Assets for financial instruments - Derivative financial instruments - Fair value","asset_receivable","True","Activos por instrumentos financieros - Instrumentos financieros derivados - Valor razonable"
"chart179","179","Other sundry accounts receivable","asset_receivable","True","Otras cuentas por cobrar diversas"
"chart181","181","Financial costs","asset_prepayments","False","Costos financieros"
"chart182","182","Insurance","asset_prepayments","False","Seguros"
"chart183","183","Leasings","asset_prepayments","False","Alquileres"
"chart184","184","Premiums paid for options","asset_prepayments","False","Primas pagadas por opciones"
"chart185","185","Maintenance of fixed assets","asset_prepayments","False","Mantenimiento de activos inmovilizados"
"chart189","189","Other expenses contracted in advance","asset_prepayments","False","Otros gastos contratados por anticipado"
"chart1911","1911","Trade accounts receivable – Third parties - Invoices, tickets and other receipts receivable","asset_receivable","True","Cuentas por cobrar comerciales – Terceros - Facturas, boletas y otros comprobantes por cobrar"
"chart1913","1913","Trade accounts receivable – Third parties - Bills to be collected","asset_receivable","True","Cuentas por cobrar comerciales – Terceros - Letras por cobrar"
"chart1921","1921","Trade accounts receivable – Associated - Invoices, tickets and other receipts receivable","asset_receivable","True","Cuentas por cobrar comerciales – Relacionadas - Facturas, boletas y otros comprobantes por cobrar"
"chart1923","1923","Trade accounts receivable – Associated - Bills to be collected","asset_receivable","True","Cuentas por cobrar comerciales – Relacionadas - Letras por cobrar"
"chart1931","1931","Accounts receivable from staff, shareholders (partners) and directors - Personal","asset_receivable","True","Cuentas por cobrar al personal, a los accionistas (socios) y directores - Personal"
"chart1932","1932","Accounts receivable from staff, shareholders (partners) and directors - Shareholders (or partners)","asset_receivable","True","Cuentas por cobrar al personal, a los accionistas (socios) y directores - Shareholders (or partners)"
"chart1933","1933","Accounts receivable from staff, shareholders (partners) and directors - Directors","asset_receivable","True","Cuentas por cobrar al personal, a los accionistas (socios) y directores - Directors"
"chart1939","1939","Accounts receivable from staff, shareholders (partners) and directors - Various","asset_receivable","True","Cuentas por cobrar al personal, a los accionistas (socios) y directores - Diversas"
"chart1941","1941","Various accounts receivable – Third parties - Loans","asset_receivable","True","Cuentas por cobrar diversas – Terceros - Préstamos"
"chart1942","1942","Various accounts receivable – Third parties - Claims to third parties","asset_receivable","True","Cuentas por cobrar diversas – Terceros - Reclamaciones a terceros"
"chart1943","1943","Various accounts receivable – Third parties - Interest, royalties and dividends","asset_receivable","True","Cuentas por cobrar diversas – Terceros - Intereses, regalías y dividendos"
"chart1944","1944","Various accounts receivable – Third parties - Deposits granted in guarantee","asset_receivable","True","Cuentas por cobrar diversas – Terceros - Depósitos otorgados en garantía"
"chart1945","1945","Various accounts receivable – Third parties - Sale of fixed assets","asset_receivable","True","Cuentas por cobrar diversas – Terceros - Venta de activo inmovilizado"
"chart1946","1946","Various accounts receivable – Third parties - Assets for financial instruments","asset_receivable","True","Cuentas por cobrar diversas – Terceros - Activos por instrumentos financieros"
"chart1949","1949","Various accounts receivable – Third parties - Other sundry accounts receivable","asset_receivable","True","Cuentas por cobrar diversas – Terceros - Otras cuentas por cobrar diversas"
"chart1951","1951","Various accounts receivable – Associated - Loans","asset_receivable","True","Cuentas por cobrar diversas – Relacionadas - Préstamos"
"chart1953","1953","Various accounts receivable – Associated - Interest, royalties and dividends","asset_receivable","True","Cuentas por cobrar diversas – Relacionadas - Intereses, regalías y dividendos"
"chart1954","1954","Various accounts receivable – Associated - Deposits granted in guarantee","asset_receivable","True","Cuentas por cobrar diversas – Relacionadas - Depósitos otorgados en garantía"
"chart1955","1955","Various accounts receivable – Associated - Sale of fixed assets","asset_receivable","True","Cuentas por cobrar diversas – Relacionadas - Venta de activo inmovilizado"
"chart1956","1956","Various accounts receivable – Associated - Assets for financial instruments","asset_receivable","True","Cuentas por cobrar diversas – Relacionadas - Activos por instrumentos financieros"
"chart1959","1959","Various accounts receivable – Associated - Other sundry accounts receivable","asset_receivable","True","Cuentas por cobrar diversas – Relacionadas - Otras cuentas por cobrar diversas"
"chart20111","20111","Merchandise - Merchandise - Costs","asset_current","False","Mercaderías - Mercaderías - Costo"
"chart20114","20114","Merchandise - Merchandise - Fair value","asset_current","False","Mercaderías - Mercaderías - Valor razonable"
"chart21111","21111","Finished products - Finished products - Costs","asset_current","False","Productos terminados - Productos terminados - Costo"
"chart21113","21113","Finished products - Finished products - Financing costs","asset_current","False","Productos terminados - Productos terminados - Costos de financiación"
"chart21114","21114","Finished products - Finished products - Fair value","asset_current","False","Productos terminados - Productos terminados - Fair value"
"chart21511","21511","Inventory of finished services - Finished services - Costs","asset_current","False","Inventario de servicios terminados - Servicios terminados - Costo"
"chart221","221","By-products","asset_current","False","Subproductos"
"chart222","222","Scrap and waste","asset_current","False","Desechos y desperdicios"
"chart23111","23111","Products in process - Products in process - Costs","asset_current","False","Productos en proceso - Productos en proceso - Costo"
"chart23113","23113","Products in process - Products in process - Financing costs","asset_current","False","Productos en proceso - Productos en proceso - Costos de financiación"
"chart23511","23511","Inventory of services in process - Services in process - Costs","asset_current","False","Inventario de servicios en proceso - Servicios en proceso - Costo"
"chart24111","24111","Raw Materials - Raw Materials - Costs","asset_current","False","Materias primas - Materias primas - Costo"
"chart24114","24114","Raw Materials - Raw Materials - Fair value","asset_current","False","Materias primas - Materias primas - Valor razonable"
"chart251","251","Auxiliary materials","asset_current","False","Materiales auxiliares"
"chart2521","2521","Supplies - Fuels","asset_current","False","Suministros - Combustibles"
"chart2522","2522","Supplies - Lubricants","asset_current","False","Suministros - Lubricantes"
"chart2523","2523","Supplies - Energy","asset_current","False","Suministros - Energía"
"chart2524","2524","Supplies - Other supplies","asset_current","False","Suministros - Otros suministros"
"chart253","253","Spare parts","asset_current","False","Repuestos"
"chart261","261","Containers","asset_current","False","Envases"
"chart262","262","Packaging","asset_current","False","Embalajes"
"chart27111","27111","Investment property - Land - Costs","asset_current","False","Propiedades de inversión - Terrenos - Costo"
"chart27112","27112","Investment property - Land - Revaluation","asset_current","False","Propiedades de inversión - Terrenos - Revaluación"
"chart27114","27114","Investment property - Land - Fair value","asset_current","False","Propiedades de inversión - Terrenos - Fair value"
"chart27121","27121","Investment property - Buildings - Costs","asset_current","False","Propiedades de inversión - Edificaciones - Costo"
"chart27122","27122","Investment property - Buildings - Revaluation","asset_current","False","Propiedades de inversión - Edificaciones - Revaluación"
"chart27123","27123","Investment property - Buildings - Financing costs","asset_current","False","Propiedades de inversión - Edificaciones - Costos de financiación"
"chart27124","27124","Investment property - Buildings - Fair value","asset_current","False","Propiedades de inversión - Edificaciones - Fair value"
"chart27201","27201","Property, plant and equipment - Production plant in production - Costs","asset_current","False","Propiedad, planta y equipo - Planta productora en producción - Costo"
"chart27202","27202","Property, plant and equipment - Production plant in production - Revaluation","asset_current","False","Propiedad, planta y equipo - Planta productora en producción - Revaluación"
"chart27203","27203","Property, plant and equipment - Production plant in production - Financing costs","asset_current","False","Propiedad, planta y equipo - Planta productora en producción - Costos de financiación"
"chart27204","27204","Property, plant and equipment - Production plant in production - Fair value","asset_current","False","Propiedad, planta y equipo - Planta productora en producción - Valor razonable"
"chart27211","27211","Property, plant and equipment - Production plant in development - Costs","asset_current","False","Propiedad, planta y equipo - Planta productora en desarrollo - Costo"
"chart27212","27212","Property, plant and equipment - Production plant in development - Revaluation","asset_current","False","Propiedad, planta y equipo - Planta productora en desarrollo - Revaluación"
"chart27213","27213","Property, plant and equipment - Production plant in development - Financing costs","asset_current","False","Propiedad, planta y equipo - Planta productora en desarrollo - Costos de financiación"
"chart27214","27214","Property, plant and equipment - Production plant in development - Fair value","asset_current","False","Propiedad, planta y equipo - Planta productora en desarrollo - Valor razonable"
"chart27221","27221","Property, plant and equipment - Land - Costs","asset_current","False","Propiedad, planta y equipo - Obras en curso - Costo"
"chart27222","27222","Property, plant and equipment - Land - Revaluation","asset_current","False","Propiedad, planta y equipo - Obras en curso - Revaluación"
"chart27231","27231","Property, plant and equipment - Buildings - Costs","asset_current","False","Propiedad, planta y equipo - Edificaciones - Costo"
"chart27232","27232","Property, plant and equipment - Buildings - Revaluation","asset_current","False","Propiedad, planta y equipo - Edificaciones - Revaluación"
"chart27233","27233","Property, plant and equipment - Buildings - Financing costs","asset_current","False","Propiedad, planta y equipo - Edificaciones - Costos de financiación"
"chart27241","27241","Property, plant and equipment - Machinery and exploitation equipment - Costs","asset_current","False","Propiedad, planta y equipo - Maquinarias y equipos de explotación - Costo"
"chart27242","27242","Property, plant and equipment - Machinery and exploitation equipment - Revaluation","asset_current","False",""
"chart27243","27243","Property, plant and equipment - Machinery and exploitation equipment - Financing costs","asset_current","False","Propiedad, planta y equipo - Maquinarias y equipos de explotación - Costo de financiación"
"chart27251","27251","Property, plant and equipment - Transport units - Costs","asset_current","False","Propiedad, planta y equipo - Unidades de transporte - Costo"
"chart27252","27252","Property, plant and equipment - Transport units - Revaluation","asset_current","False","Propiedad, planta y equipo - Unidades de transporte - Revaluación"
"chart27261","27261","Property, plant and equipment - Furniture and fixtures - Costs","asset_current","False","Propiedad, planta y equipo - Muebles y enseres - Costo"
"chart27262","27262","Property, plant and equipment - Furniture and fixtures - Revaluation","asset_current","False","Propiedad, planta y equipo - Muebles y enseres - Revaluación"
"chart27271","27271","Property, plant and equipment - Miscellaneous equipment - Costs","asset_current","False",""
"chart27272","27272","Property, plant and equipment - Miscellaneous equipment - Revaluation","asset_current","False",""
"chart27281","27281","Property, plant and equipment - Replacement tools and units - Costs","asset_current","False","Propiedad, planta y equipo - Herramientas y unidades de reemplazo - Costo"
"chart27282","27282","Property, plant and equipment - Replacement tools and units - Revaluation","asset_current","False","Propiedad, planta y equipo - Herramientas y unidades de reemplazo - Revaluación"
"chart27291","27291","Property, plant and equipment - Work in progress - Costs","asset_current","False","Propiedad, planta y equipo - Obras en curso - Costo"
"chart27292","27292","Property, plant and equipment - Work in progress - Revaluation","asset_current","False","Propiedad, planta y equipo - Obras en curso - Revaluación"
"chart27311","27311","Intangibles - Concessions, licenses and rights - Costs","asset_current","False","Intangibles - Concesiones, licencias y derechos - Costo"
"chart27312","27312","Intangibles - Concessions, licenses and rights - Revaluation","asset_current","False","Intangibles - Concesiones, licencias y derechos - Revaluación"
"chart27321","27321","Intangibles - Patents and industrial property - Costs","asset_current","False","Intangibles - Patentes y propiedad industrial - Costo"
"chart27322","27322","Intangibles - Patents and industrial property - Revaluation","asset_current","False","Intangibles - Patentes y propiedad industrial - Revaluación"
"chart27331","27331","Intangibles - Softwares - Costs","asset_current","False","Intangibles - Programas de computadora (software) - Costo"
"chart27332","27332","Intangibles - Softwares - Revaluation","asset_current","False","Intangibles - Programas de computadora (software) - Revaluación"
"chart27341","27341","Intangibles - Exploration and development costs - Costs","asset_current","False","Intangibles - Costos de exploración y desarrollo - Costo"
"chart27342","27342","Intangibles - Exploration and development costs - Revaluation","asset_current","False","Intangibles - Costos de exploración y desarrollo - Revaluación"
"chart27351","27351","Intangibles - Formulas, designs and prototypes - Costs","asset_current","False","Intangibles - Fórmulas, diseños y prototipos - Costo"
"chart27352","27352","Intangibles - Formulas, designs and prototypes - Revaluation","asset_current","False","Intangibles - Fórmulas, diseños y prototipos - Revaluación"
"chart27391","27391","Intangibles - Other intangible assets - Costs","asset_current","False","Intangibles - Otros activos intangibles - Costo"
"chart27392","27392","Intangibles - Other intangible assets - Revaluation","asset_current","False","Intangibles - Otros activos intangibles - Revaluación"
"chart27411","27411","Biological assets - Biological assets in production - Costs","asset_current","False","Activos biológicos - Activos biológicos en producción - Costo"
"chart27413","27413","Biological assets - Biological assets in production - Financing costs","asset_current","False","Activos biológicos - Activos biológicos en producción - Costos de financiación"
"chart27414","27414","Biological assets - Biological assets in production - Fair value","asset_current","False","Activos biológicos - Activos biológicos en producción - Valor razonable"
"chart27421","27421","Biological assets - Biological assets under development - Costs","asset_current","False","Activos biológicos - Activos biológicos en desarrollo - Costo"
"chart27423","27423","Biological assets - Biological assets under development - Financing costs","asset_current","False","Activos biológicos - Activos biológicos en desarrollo - Costos de financiación"
"chart27424","27424","Biological assets - Biological assets under development - Fair value","asset_current","False","Activos biológicos - Activos biológicos en desarrollo - Valor razonable"
"chart27521","27521","Accumulated depreciation – Investment property - Buildings - Costs","asset_current","False","Depreciación acumulada – Propiedades de inversión - Edificaciones - Costo"
"chart27522","27522","Accumulated depreciation – Investment property - Buildings - Revaluation","asset_current","False","Depreciación acumulada – Propiedades de inversión - Edificaciones - Revaluación"
"chart27523","27523","Accumulated depreciation – Investment property - Buildings - Financing costs","asset_current","False","Depreciación acumulada – Propiedades de inversión - Edificaciones - Costo de financiación"
"chart27601","27601","Accumulated depreciation – Property, plant and equipment - Production plant in production - Costs","asset_current","False","Depreciación acumulada – Propiedades, planta y equipo - Planta productora en producción - Costo"
"chart27602","27602","Accumulated depreciation – Property, plant and equipment - Production plant in production - Revaluation","asset_current","False","Depreciación acumulada – Propiedades, planta y equipo - Planta productora en producción - Revaluación"
"chart27603","27603","Accumulated depreciation – Property, plant and equipment - Production plant in production - Financing costs","asset_current","False","Depreciación acumulada – Propiedades, planta y equipo - Planta productora en producción - Costos de financiación"
"chart27604","27604","Accumulated depreciation – Property, plant and equipment - Production plant in production - Fair value","asset_current","False","Depreciación acumulada – Propiedades, planta y equipo - Planta productora en producción - Valor razonable"
"chart27621","27621","Accumulated depreciation – Property, plant and equipment - Buildings - Costs","asset_current","False","Depreciación acumulada – Propiedades, planta y equipo - Edificaciones - Costo"
"chart27622","27622","Accumulated depreciation – Property, plant and equipment - Buildings - Revaluation","asset_current","False","Depreciación acumulada – Propiedades, planta y equipo - Edificaciones - Revaluación"
"chart27623","27623","Accumulated depreciation – Property, plant and equipment - Buildings - Financing costs","asset_current","False","Depreciación acumulada – Propiedades, planta y equipo - Edificaciones - Costo de financiación"
"chart27631","27631","Accumulated depreciation – Property, plant and equipment - Machinery and operating equipment - Costs","asset_current","False","Depreciación acumulada – Propiedades, planta y equipo - Maquinarias y equipo de explotación - Costo"
"chart27632","27632","Accumulated depreciation – Property, plant and equipment - Machinery and operating equipment - Revaluation","asset_current","False","Depreciación acumulada – Propiedades, planta y equipo - Maquinarias y equipo de explotación - Revaluación"
"chart27633","27633","Accumulated depreciation – Property, plant and equipment - Machinery and operating equipment - Financing costs","asset_current","False","Depreciación acumulada – Propiedades, planta y equipo - Maquinarias y equipo de explotación - Costos de financiación"
"chart27641","27641","Accumulated depreciation – Property, plant and equipment - Transport units - Costs","asset_current","False","Depreciación acumulada – Propiedades, planta y equipo - Unidades de transporte - Costo"
"chart27642","27642","Accumulated depreciation – Property, plant and equipment - Transport units - Revaluation","asset_current","False","Depreciación acumulada – Propiedades, planta y equipo - Unidades de transporte - Revaluación"
"chart27651","27651","Accumulated depreciation – Property, plant and equipment - Furniture and fixtures - Costs","asset_current","False","Depreciación acumulada – Propiedades, planta y equipo - Muebles y enseres - Costo"
"chart27652","27652","Accumulated depreciation – Property, plant and equipment - Furniture and fixtures - Revaluation","asset_current","False","Depreciación acumulada – Propiedades, planta y equipo - Muebles y enseres - Revaluación"
"chart27661","27661","Accumulated depreciation – Property, plant and equipment - Miscellaneous equipment - Costs","asset_current","False","Depreciación acumulada – Propiedades, planta y equipo - Equipos diversos - Costo"
"chart27662","27662","Accumulated depreciation – Property, plant and equipment - Miscellaneous equipment - Revaluation","asset_current","False","Depreciación acumulada – Propiedades, planta y equipo - Equipos diversos - Revaluación"
"chart27671","27671","Accumulated depreciation – Property, plant and equipment - Replacement tools and units - Costs","asset_current","False","Depreciación acumulada – Propiedades, planta y equipo - Herramientas y unidades de reemplazo - Costo"
"chart27672","27672","Accumulated depreciation – Property, plant and equipment - Replacement tools and units - Revaluation","asset_current","False","Depreciación acumulada – Propiedades, planta y equipo - Herramientas y unidades de reemplazo - Revaluación"
"chart27711","27711","Accumulated amortization – Intangibles - Concessions, licenses and rights - Costs","asset_current","False","Amortización acumulada – Intangibles - Concesiones, licencias y derechos - Costo"
"chart27712","27712","Accumulated amortization – Intangibles - Concessions, licenses and rights - Revaluation","asset_current","False","Amortización acumulada – Intangibles - Concesiones, licencias y derechos - Revaluación"
"chart27721","27721","Accumulated amortization – Intangibles - Patents and industrial property - Costs","asset_current","False","Amortización acumulada – Intangibles - Patentes y propiedad industrial - Costo"
"chart27722","27722","Accumulated amortization – Intangibles - Patents and industrial property - Revaluation","asset_current","False","Amortización acumulada – Intangibles - Patentes y propiedad industrial - Revaluación"
"chart27731","27731","Accumulated amortization – Intangibles - Softwares - Costs","asset_current","False","Amortización acumulada – Intangibles - Programas de computadora (software) - Costo"
"chart27732","27732","Accumulated amortization – Intangibles - Softwares - Revaluation","asset_current","False","Amortización acumulada – Intangibles - Programas de computadora (software) - Revaluación"
"chart27741","27741","Accumulated amortization – Intangibles - Exploration and development costs - Costs","asset_current","False","Amortización acumulada – Intangibles - Costos de exploración y desarrollo - Costo"
"chart27742","27742","Accumulated amortization – Intangibles - Exploration and development costs - Revaluation","asset_current","False","Amortización acumulada – Intangibles - Costos de exploración y desarrollo - Revaluación"
"chart27751","27751","Accumulated amortization – Intangibles - Formulas, designs and prototypes - Costs","asset_current","False","Amortización acumulada – Intangibles - Fórmulas, diseños y prototipos - Costo"
"chart27752","27752","Accumulated amortization – Intangibles - Formulas, designs and prototypes - Revaluation","asset_current","False","Amortización acumulada – Intangibles - Fórmulas, diseños y prototipos - Revaluación"
"chart27791","27791","Accumulated amortization – Intangibles - Other intangible assets - Costs","asset_current","False","Amortización acumulada – Intangibles - Otros activos intangibles - Costo"
"chart27792","27792","Accumulated amortization – Intangibles - Other intangible assets - Revaluation","asset_current","False","Amortización acumulada – Intangibles - Otros activos intangibles - Revaluación"
"chart27811","27811","Accumulated depreciation – Biological assets - Biological assets in production - Costs","asset_current","False","Depreciación acumulada – Activos biológicos - Activos biológicos en producción - Costo"
"chart27813","27813","Accumulated depreciation – Biological assets - Biological assets in production - Financing costs","asset_current","False","Depreciación acumulada – Activos biológicos - Activos biológicos en producción - Costos de financiación"
"chart27821","27821","Accumulated depreciation – Biological assets - Biological assets under development - Costs","asset_current","False","Depreciación acumulada – Activos biológicos - Activos biológicos en desarrollo - Costo"
"chart27823","27823","Accumulated depreciation – Biological assets - Biological assets under development - Financing costs","asset_current","False","Depreciación acumulada – Activos biológicos - Activos biológicos en desarrollo - Costos de financiación"
"chart27910","27910","Accumulated impairment - Investment property - Production plant in production","asset_current","False","Desvalorización acumulada - Propiedad de inversión - Planta productora en producción"
"chart27911","27911","Accumulated impairment - Investment property - Production plant in development","asset_current","False","Desvalorización acumulada - Propiedad de inversión - Planta productora en desarrollo"
"chart27912","27912","Accumulated impairment - Investment property - Land","asset_current","False","Desvalorización acumulada - Propiedad de inversión - Terrenos"
"chart27913","27913","Accumulated impairment - Investment property - Buildings","asset_current","False","Desvalorización acumulada - Propiedad de inversión - Edificaciones"
"chart27930","27930","Accumulated impairment - Property, plant and equipment - Production plants in production","asset_current","False","Desvalorización acumulada - Propiedades, planta y equipo - Plantas productoras en producción"
"chart27931","27931","Accumulated impairment - Property, plant and equipment - Production plant in development","asset_current","False","Desvalorización acumulada - Propiedades, planta y equipo - Planta productora en desarrollo"
"chart27932","27932","Accumulated impairment - Property, plant and equipment - Land","asset_current","False","Desvalorización acumulada - Propiedades, planta y equipo - Terrenos"
"chart27933","27933","Accumulated impairment - Property, plant and equipment - Buildings","asset_current","False","Desvalorización acumulada - Propiedades, planta y equipo - Edificaciones"
"chart27934","27934","Accumulated impairment - Property, plant and equipment - Machinery and exploitation equipment","asset_current","False","Desvalorización acumulada - Propiedades, planta y equipo - Maquinarias y equipos de explotación"
"chart27935","27935","Accumulated impairment - Property, plant and equipment - Transport units","asset_current","False","Desvalorización acumulada - Propiedades, planta y equipo - Unidades de transporte"
"chart27936","27936","Accumulated impairment - Property, plant and equipment - Furniture and fixtures","asset_current","False","Desvalorización acumulada - Propiedades, planta y equipo - Muebles y enseres"
"chart27937","27937","Accumulated impairment - Property, plant and equipment - Miscellaneous equipment","asset_current","False","Desvalorización acumulada - Propiedades, planta y equipo - Equipos diversos"
"chart27938","27938","Accumulated impairment - Property, plant and equipment - Replacement tools and units","asset_current","False","Desvalorización acumulada - Propiedades, planta y equipo - Herramientas y unidades de reemplazo"
"chart27941","27941","Accumulated impairment - Intangibles - Concessions, licenses and other rights","asset_current","False","Desvalorización acumulada - Intangibles - Concesiones, licencias y otros derechos"
"chart27942","27942","Accumulated impairment - Intangibles - Patents and industrial property","asset_current","False","Desvalorización acumulada - Intangibles - Patentes y propiedad industrial"
"chart27943","27943","Accumulated impairment - Intangibles - Softwares","asset_current","False","Desvalorización acumulada - Intangibles - Programas de computadora (software)"
"chart27944","27944","Accumulated impairment - Intangibles - Exploration and development costs","asset_current","False","Desvalorización acumulada - Intangibles - Costos de exploración y desarrollo"
"chart27945","27945","Accumulated impairment - Intangibles - Formulas, designs and prototypes","asset_current","False","Desvalorización acumulada - Intangibles - Fórmulas, diseños y prototipos"
"chart27949","27949","Accumulated impairment - Intangibles - Other intangible assets","asset_current","False","Desvalorización acumulada - Intangibles - Otros activos intangibles"
"chart27951","27951","Accumulated impairment - Biological assets - Biological assets in production","asset_current","False","Desvalorización acumulada - Activos biológicos - Activos biológicos en producción"
"chart27952","27952","Accumulated impairment - Biological assets - Biological assets under development","asset_current","False","Desvalorización acumulada - Activos biológicos - Activos biológicos en desarrollo"
"chart281","281","Merchandise","asset_current","False","Mercaderías"
"chart284","284","Raw Materials","asset_current","False","Materias primas"
"chart285","285","Auxiliary materials, supplies and spare parts","asset_current","False","Materiales auxiliares, suministros y repuestos"
"chart286","286","Containers and packaging","asset_current","False","Envases y embalajes"
"chart29111","29111","Merchandise - Merchandise - Costs","asset_current","False","Mercaderías - Mercaderías - Costo"
"chart29211","29211","Finished products - Finished products - Costs","asset_current","False","Productos terminados - Productos terminados - Costo"
"chart29213","29213","Finished products - Finished products - Financing costs","asset_current","False","Productos terminados - Productos terminados - Costos de financiación"
"chart29251","29251","Finished products - Inventory of finished services - Costs","asset_current","False","Productos terminados - Inventario de servicios terminados - Costo"
"chart2931","2931","By-products, waste and scrap - By-products","asset_current","False","Subproductos, desechos y desperdicios - Subproductos"
"chart2932","2932","By-products, waste and scrap - Scrap and waste","asset_current","False","Subproductos, desechos y desperdicios - Desechos y desperdicios"
"chart29411","29411","Products in process - Products in process - Costs","asset_current","False","Productos en proceso - Productos en proceso - Costo"
"chart29413","29413","Products in process - Products in process - Financing costs","asset_current","False","Productos en proceso - Productos en proceso - Costos de financiación"
"chart2945","2945","Products in process - Inventory of services in process","asset_current","False","Productos en proceso - Inventario de servicios en proceso"
"chart29511","29511","Raw Materials - Raw Materials - Costs","asset_current","False","Materias primas - Materias primas - Costo"
"chart2961","2961","Auxiliary materials, supplies and spare parts - Auxiliary materials","asset_current","False","Materiales auxiliares, suministros y repuestos - Materiales auxiliares"
"chart2962","2962","Auxiliary materials, supplies and spare parts - Supplies","asset_current","False","Materiales auxiliares, suministros y repuestos - Suministros"
"chart2963","2963","Auxiliary materials, supplies and spare parts - Spare parts","asset_current","False","Materiales auxiliares, suministros y repuestos - Repuestos"
"chart2971","2971","Containers and packaging - Containers","asset_current","False","Envases y embalajes - Envases"
"chart2972","2972","Containers and packaging - Packaging","asset_current","False","Envases y embalajes - Embalajes"
"chart2981","2981","Stock to be received - Merchandise","asset_current","False","Existencias por recibir - Mercaderías"
"chart2982","2982","Stock to be received - Raw Materials","asset_current","False","Existencias por recibir - Materias primas"
"chart2983","2983","Stock to be received - Auxiliary materials, supplies and spare parts","asset_current","False","Existencias por recibir - Materiales auxiliares, suministros y repuestos"
"chart2984","2984","Stock to be received - Containers and packaging","asset_current","False","Existencias por recibir - Materias primas"
"chart30111","30111","Investments to be held until maturity - Debt financial instruments - Costs","asset_non_current","False","Inversiones a ser mantenidas hasta el vencimiento - Instrumentos financieros representativos de deuda - Costo"
"chart30114","30114","Investments to be held until maturity - Debt financial instruments - Fair value","asset_non_current","False","Inversiones a ser mantenidas hasta el vencimiento - Instrumentos financieros representativos de deuda - Valor razonable"
"chart3021","3021","Financial instruments representative of economic rights - Preferred Subscription Certificates","asset_non_current","False","Instrumentos financieros representativos de derecho patrimonial - Certificados de suscripción preferente"
"chart30221","30221","Financial instruments representative of economic rights - Shares representing capital stock – Common - Costs","asset_non_current","False","Instrumentos financieros representativos de derecho patrimonial - Acciones representativas de capital social – Comunes - Costo"
"chart30224","30224","Financial instruments representative of economic rights - Shares representing capital stock – Common - Fair value","asset_non_current","False","Instrumentos financieros representativos de derecho patrimonial - Acciones representativas de capital social – Comunes - Valor razonable"
"chart30225","30225","Financial instruments representative of economic rights - Shares representing capital stock – Common - Equity participation","asset_non_current","False","Instrumentos financieros representativos de derecho patrimonial - Acciones representativas de capital social – Comunes - Participación patrimonial"
"chart30231","30231","Financial instruments representative of economic rights - Shares representing capital stock – Preferences - Costs","asset_non_current","False","Instrumentos financieros representativos de derecho patrimonial - Acciones representativas de capital social – Preferentes - Costo"
"chart30234","30234","Financial instruments representative of economic rights - Shares representing capital stock – Preferences - Fair value","asset_non_current","False","Instrumentos financieros representativos de derecho patrimonial - Acciones representativas de capital social – Preferentes - Valor razonable"
"chart30235","30235","Financial instruments representative of economic rights - Shares representing capital stock – Preferences - Equity participation","asset_non_current","False","Instrumentos financieros representativos de derecho patrimonial - Acciones representativas de capital social – Preferentes - Participación patrimonial"
"chart30241","30241","Financial instruments representative of economic rights - Investment shares - Costs","asset_non_current","False","Financial instruments representative of economic rights - Acciones de inversión - Costo"
"chart30244","30244","Financial instruments representative of economic rights - Investment shares - Fair value","asset_non_current","False","Instrumentos financieros representativos de derecho patrimonial - Acciones de inversión - Valor razonable"
"chart30245","30245","Financial instruments representative of economic rights - Investment shares - Equity participation","asset_non_current","False","Instrumentos financieros representativos de derecho patrimonial - Acciones de inversión - Participación patrimonial"
"chart30281","30281","Financial instruments representative of economic rights - Other titles representing equity - Costs","asset_non_current","False","Instrumentos financieros representativos de derecho patrimonial - Otros títulos representativos de patrimonio - Costo"
"chart30284","30284","Financial instruments representative of economic rights - Other titles representing equity - Fair value","asset_non_current","False","Instrumentos financieros representativos de derecho patrimonial - Otros títulos representativos de patrimonio - Valor razonable"
"chart30285","30285","Financial instruments representative of economic rights - Other titles representing equity - Equity participation","asset_non_current","False","Instrumentos financieros representativos de derecho patrimonial - Otros títulos representativos de patrimonio - Participación patrimonial"
"chart30311","30311","Certificates of participation in funds - Dues - Investment funds - Costs","asset_non_current","False","Certificados de participación en fondos - Cuotas - Fondos de inversión - Costo"
"chart30314","30314","Certificates of participation in funds - Dues - Investment funds - Fair value","asset_non_current","False","Certificados de participación en fondos - Cuotas - Fondos de inversión - Fair value"
"chart30321","30321","Certificates of participation in funds - Dues - Mutual funds - Costs","asset_non_current","False","Certificados de participación en fondos - Cuotas - Fondos mutuos - Costo"
"chart30324","30324","Certificates of participation in funds - Dues - Mutual funds - Fair value","asset_non_current","False","Certificados de participación en fondos - Cuotas - Fondos mutuos - Valor razonable"
"chart30411","30411","Participations in joint agreements - Joint operations - Costs","asset_non_current","False","Participaciones en acuerdos conjuntos - Operaciones conjuntas - Costo"
"chart30414","30414","Participations in joint agreements - Joint operations - Fair value","asset_non_current","False","Participaciones en acuerdos conjuntos - Operaciones conjuntas - Valor razonable"
"chart30415","30415","Participations in joint agreements - Joint operations - Equity participation","asset_non_current","False","Participaciones en acuerdos conjuntos - Operaciones conjuntas - Participación patrimonial"
"chart30421","30421","Participations in joint agreements - Joint ventures - Costs","asset_non_current","False","Participaciones en acuerdos conjuntos - Negocios conjuntos - Costo"
"chart30424","30424","Participations in joint agreements - Joint ventures - Fair value","asset_non_current","False","Participaciones en acuerdos conjuntos - Negocios conjuntos - Valor razonable"
"chart30425","30425","Participations in joint agreements - Joint ventures - Equity participation","asset_non_current","False","Participaciones en acuerdos conjuntos - Negocios conjuntos - Participación patrimonial"
"chart30811","30811","Securities investments – Purchase agreements - Debt financial instruments – Purchase agreements - Costs","asset_non_current","False","Inversiones mobiliarias – Acuerdos de compra - Instrumentos financieros representativos de deuda – Acuerdos de compra - Costo"
"chart30814","30814","Securities investments – Purchase agreements - Debt financial instruments – Purchase agreements - Fair value","asset_non_current","False","Inversiones mobiliarias – Acuerdos de compra - Instrumentos financieros representativos de deuda – Acuerdos de compra - Valor razonable"
"chart30821","30821","Securities investments – Purchase agreements - Financial instruments representative of economic rights – Purchase agreements - Costs","asset_non_current","False","Inversiones mobiliarias – Acuerdos de compra - Instrumentos financieros representativos de derecho patrimonial – Acuerdos de compra - Costo"
"chart30824","30824","Securities investments – Purchase agreements - Financial instruments representative of economic rights – Purchase agreements - Fair value","asset_non_current","False","Inversiones mobiliarias – Acuerdos de compra - Instrumentos financieros representativos de derecho patrimonial – Acuerdos de compra - Valor razonable"
"chart31111","31111","Land - Urban - Costs","asset_fixed","False","Terrenos - Urbanos - Costo"
"chart31112","31112","Land - Urban - Revaluation","asset_fixed","False","Terrenos - Urbanos - Revaluación"
"chart31114","31114","Land - Urban - Fair value","asset_fixed","False","Terrenos - Urbanos - Fair value"
"chart31121","31121","Land - Rural - Costs","asset_fixed","False","Terrenos - Rurales - Costo"
"chart31122","31122","Land - Rural - Revaluation","asset_fixed","False","Terrenos - Rurales - Revaluación"
"chart31124","31124","Land - Rural - Fair value","asset_fixed","False","Terrenos - Rurales - Fair value"
"chart31211","31211","Buildings - Buildings - Costs","asset_fixed","False","Edificaciones - Edificaciones - Costo"
"chart31212","31212","Buildings - Buildings - Revaluation","asset_fixed","False","Edificaciones - Edificaciones - Revaluación"
"chart31213","31213","Buildings - Buildings - Financing costs","asset_fixed","False","Edificaciones - Edificaciones - Costos de financiación"
"chart31214","31214","Buildings - Buildings - Fair value","asset_fixed","False","Edificaciones - Edificaciones - Valor razonable"
"chart31311","31311","Constructions in progress - Buildings - Costs","asset_fixed","False","Construcciones en curso - Edificaciones - Costo"
"chart31312","31312","Constructions in progress - Buildings - Revaluation","asset_fixed","False","Construcciones en curso - Edificaciones - Revaluación"
"chart31313","31313","Constructions in progress - Buildings - Financing costs","asset_fixed","False","Construcciones en curso - Edificaciones - Costos de financiación"
"chart31314","31314","Constructions in progress - Buildings - Fair value","asset_fixed","False","Construcciones en curso - Edificaciones - Valor razonable"
"chart32111","32111","Investment property - Financial leasing - Land - Costs","asset_fixed","False","Propiedades de inversión - Arrendamiento financiero - Terrenos - Costo"
"chart32112","32112","Investment property - Financial leasing - Land - Revaluation","asset_fixed","False","Propiedades de inversión - Arrendamiento financiero - Terrenos - Revaluación"
"chart32114","32114","Investment property - Financial leasing - Land - Fair value","asset_fixed","False","Propiedades de inversión - Arrendamiento financiero - Terrenos - Valor razonable"
"chart32121","32121","Investment property - Financial leasing - Buildings - Costs","asset_fixed","False","Propiedades de inversión - Arrendamiento financiero - Edificaciones - Costo"
"chart32122","32122","Investment property - Financial leasing - Buildings - Revaluation","asset_fixed","False","Propiedades de inversión - Arrendamiento financiero - Edificaciones - Revaluación"
"chart32123","32123","Investment property - Financial leasing - Buildings - Financing costs","asset_fixed","False","Propiedades de inversión - Arrendamiento financiero - Edificaciones - Costo de financiación"
"chart32124","32124","Investment property - Financial leasing - Buildings - Fair value","asset_fixed","False","Propiedades de inversión - Arrendamiento financiero - Edificaciones - Valor razonable"
"chart32201","32201","Property, plant and equipment - Financial leasing - Production plant in production - Costs","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en producción - Costo"
"chart32202","32202","Property, plant and equipment - Financial leasing - Production plant in production - Revaluation","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en producción - Revaluación"
"chart32203","32203","Property, plant and equipment - Financial leasing - Production plant in production - Financing costs","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en producción - Costos de financiación"
"chart32211","32211","Property, plant and equipment - Financial leasing - Production plant in development - Costs","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en desarrollo - Costo"
"chart32212","32212","Property, plant and equipment - Financial leasing - Production plant in development - Revaluation","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en desarrollo - Revaluación"
"chart32213","32213","Property, plant and equipment - Financial leasing - Production plant in development - Financing costs","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en desarrollo - Costos de financiación"
"chart32221","32221","Property, plant and equipment - Financial leasing - Land - Costs","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento financiero - Planta productora en desarrollo - Costo"
"chart32222","32222","Property, plant and equipment - Financial leasing - Land - Revaluation","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento financiero - Terrenos - Revaluación"
"chart32231","32231","Property, plant and equipment - Financial leasing - Buildings - Costs","asset_fixed","False","Propiedades, planta y equipo - Arrendamiento financiero - Edificaciones - Costo"
"chart32232","32232","Property, plant and equipment - Financial leasing - Buildings - Revaluation","asset_fixed","False","Propiedades, planta y equipo - Arrendamiento financiero - Edificaciones - Revaluación"
"chart32233","32233","Property, plant and equipment - Financial leasing - Buildings - Financing costs","asset_fixed","False","Propiedades, planta y equipo - Arrendamiento financiero - Edificaciones - Costos de financiación"
"chart32241","32241","Property, plant and equipment - Financial leasing - Machinery and exploitation equipment - Costs","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento financiero - Maquinaria y equipo de explotación - Costo"
"chart32242","32242","Property, plant and equipment - Financial leasing - Machinery and exploitation equipment - Revaluation","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento financiero - Maquinaria y equipo de explotación - Revaluación"
"chart32243","32243","Property, plant and equipment - Financial leasing - Machinery and exploitation equipment - Financing costs","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento financiero - Maquinaria y equipo de explotación - Costos de financiación"
"chart32251","32251","Property, plant and equipment - Financial leasing - Transport units - Costs","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento financiero - Unidades de transporte - Costo"
"chart32252","32252","Property, plant and equipment - Financial leasing - Transport units - Revaluation","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento financiero - Unidades de transporte - Revaluación"
"chart32261","32261","Property, plant and equipment - Financial leasing - Furniture and fixtures - Costs","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento financiero - Muebles y enseres - Costo"
"chart32262","32262","Property, plant and equipment - Financial leasing - Furniture and fixtures - Revaluation","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento financiero - Muebles y enseres - Revaluación"
"chart32271","32271","Property, plant and equipment - Financial leasing - Miscellaneous equipment - Costs","asset_fixed","False","Propiedades, planta y equipo - Arrendamiento financiero - Equipos diversos - Costo"
"chart32272","32272","Property, plant and equipment - Financial leasing - Miscellaneous equipment - Revaluation","asset_fixed","False","Propiedades, planta y equipo - Arrendamiento financiero - Equipos diversos - Revaluación"
"chart32281","32281","Property, plant and equipment - Financial leasing - Replacement tools and units - Costs","asset_fixed","False","Propiedades, planta y equipo - Arrendamiento financiero - Herramientas y unidades de reemplazo - Costo"
"chart32282","32282","Property, plant and equipment - Financial leasing - Replacement tools and units - Revaluation","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento financiero - Herramientas y unidades de reemplazo - Revaluación"
"chart32301","32301","Property, plant and equipment - Operating lease - Production plant in production - Costs","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento operativo - Planta productora en producción - Costo"
"chart32302","32302","Property, plant and equipment - Operating lease - Production plant in production - Revaluation","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento operativo - Planta productora en producción - Revaluación"
"chart32321","32321","Property, plant and equipment - Operating lease - Land - Costs","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento operativo - Terrenos - Costo"
"chart32331","32331","Property, plant and equipment - Operating lease - Buildings - Costs","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento operativo - Edificaciones - Costo"
"chart32332","32332","Property, plant and equipment - Operating lease - Buildings - Revaluation","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento operativo - Edificaciones - Revaluación"
"chart32341","32341","Property, plant and equipment - Operating lease - Machinery and exploitation equipment - Costs","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento operativo - Maquinaria y equipo de explotación - Costo"
"chart32342","32342","Property, plant and equipment - Operating lease - Machinery and exploitation equipment - Revaluation","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento operativo - Maquinaria y equipo de explotación - Revaluación"
"chart32351","32351","Property, plant and equipment - Operating lease - Transport units - Costs","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento operativo - Unidades de transporte - Costo"
"chart32352","32352","Property, plant and equipment - Operating lease - Transport units - Revaluation","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento operativo - Unidades de transporte - Revaluación"
"chart32361","32361","Property, plant and equipment - Operating lease - Miscellaneous equipment - Costs","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento operativo - Equipos diversos - Costo"
"chart32362","32362","Property, plant and equipment - Operating lease - Miscellaneous equipment - Revaluation","asset_fixed","False","Propiedad, planta y equipo - Arrendamiento operativo - Equipos diversos - Revaluación"
"chart33011","33011","Producing plant - Production plant in production - Costs","asset_fixed","False","Planta productora - Planta productora en producción - Costo"
"chart33012","33012","Producing plant - Production plant in production - Revaluation","asset_fixed","False","Planta productora - Planta productora en producción - Revaluación"
"chart33013","33013","Producing plant - Production plant in production - Financing costs","asset_fixed","False","Planta productora - Planta productora en producción - Costos de financiación"
"chart33014","33014","Producing plant - Production plant in production - Fair value","asset_fixed","False","Planta productora - Planta productora en producción - Valor razonable"
"chart33021","33021","Producing plant - Production plant in development - Costs","asset_fixed","False","Planta productora - Planta productora en desarrollo - Costo"
"chart33022","33022","Producing plant - Production plant in development - Revaluation","asset_fixed","False","Planta productora - Planta productora en desarrollo - Revaluación"
"chart33023","33023","Producing plant - Production plant in development - Financing costs","asset_fixed","False","Planta productora - Planta productora en desarrollo - Costos de financiación"
"chart33024","33024","Producing plant - Production plant in development - Fair value","asset_fixed","False","Planta productora - Planta productora en desarrollo - Valor razonable"
"chart33111","33111","Land - Land - Costs","asset_fixed","False","Terrenos - Terrenos - Costo"
"chart33112","33112","Land - Land - Revaluation","asset_fixed","False","Terrenos - Terrenos - Revaluación"
"chart33211","33211","Buildings - Buildings - Costs","asset_fixed","False","Edificaciones - Edificaciones - Costo"
"chart33212","33212","Buildings - Buildings - Revaluation","asset_fixed","False","Edificaciones - Edificaciones - Revaluación"
"chart33213","33213","Buildings - Buildings - Financing costs","asset_fixed","False","Edificaciones - Edificaciones - Costos de financiación"
"chart33241","33241","Buildings - Installations - Costs","asset_fixed","False","Edificaciones - Instalaciones - Costo"
"chart33242","33242","Buildings - Installations - Revaluation","asset_fixed","False","Edificaciones - Instalaciones - Revaluación"
"chart33243","33243","Buildings - Installations - Financing costs","asset_fixed","False","Edificaciones - Instalaciones - Costos de financiación"
"chart33251","33251","Buildings - Improvements in leased premises. - Costs","asset_fixed","False","Edificaciones - Mejoras en locales arrendados. - Costo"
"chart33252","33252","Buildings - Improvements in leased premises. - Revaluation","asset_fixed","False","Edificaciones - Mejoras en locales arrendados. - Revaluación"
"chart33253","33253","Buildings - Improvements in leased premises. - Financing costs","asset_fixed","False","Edificaciones - Mejoras en locales arrendados. - Costo de Financiación"
"chart33311","33311","Machinery and exploitation equipment - Machinery and exploitation equipment - Costs","asset_fixed","False","Maquinarias y equipos de explotación - Maquinarias y equipos de explotación - Costo"
"chart33312","33312","Machinery and exploitation equipment - Machinery and exploitation equipment - Revaluation","asset_fixed","False","Maquinarias y equipos de explotación - Maquinarias y equipos de explotación - Revaluación"
"chart33313","33313","Machinery and exploitation equipment - Machinery and exploitation equipment - Financing costs","asset_fixed","False","Maquinarias y equipos de explotación - Maquinarias y equipos de explotación - Costos de financiación"
"chart33411","33411","Transport units - Motor vehicles - Costs","asset_fixed","False","Unidades de transporte - Vehículos motorizados - Costo"
"chart33412","33412","Transport units - Motor vehicles - Revaluation","asset_fixed","False",""
"chart33421","33421","Transport units - Non-motorized vehicles - Costs","asset_fixed","False","Unidades de transporte - Vehículos no motorizados - Costo"
"chart33422","33422","Transport units - Non-motorized vehicles - Revaluation","asset_fixed","False","Unidades de transporte - Vehículos no motorizados - Revaluación"
"chart33511","33511","Furniture and fixtures - Furnitures - Costs","asset_fixed","False","Muebles y enseres - Muebles - Costo"
"chart33512","33512","Furniture and fixtures - Furnitures - Revaluation","asset_fixed","False","Muebles y enseres - Muebles - Revaluación"
"chart33521","33521","Furniture and fixtures - Equipment - Costs","asset_fixed","False","Muebles y enseres - Enseres - Costo"
"chart33522","33522","Furniture and fixtures - Equipment - Revaluation","asset_fixed","False","Muebles y enseres - Enseres - Revaluación"
"chart33611","33611","Miscellaneous equipment - Information processing equipment - Costs","asset_fixed","False","Equipos diversos - Equipo para procesamiento de información - Costo"
"chart33612","33612","Miscellaneous equipment - Information processing equipment - Revaluation","asset_fixed","False","Equipos diversos - Equipo para procesamiento de información - Revaluación"
"chart33621","33621","Miscellaneous equipment - Communication equipment - Costs","asset_fixed","False","Equipos diversos - Equipo de comunicación - Costo"
"chart33622","33622","Miscellaneous equipment - Communication equipment - Revaluation","asset_fixed","False","Equipos diversos - Equipo de comunicación - Revaluación"
"chart33631","33631","Miscellaneous equipment - Security equipment - Costs","asset_fixed","False","Equipos diversos - Equipo de seguridad - Costo"
"chart33632","33632","Miscellaneous equipment - Security equipment - Revaluation","asset_fixed","False","Equipos diversos - Revaluación"
"chart33641","33641","Miscellaneous equipment - Environmental equipment - Costs","asset_fixed","False","Equipos diversos - Equipo de medio ambiente - Costo"
"chart33642","33642","Miscellaneous equipment - Environmental equipment - Revaluation","asset_fixed","False","Equipos diversos - Equipo de medio ambiente - Revaluación"
"chart33691","33691","Miscellaneous equipment - Other equipments - Costs","asset_fixed","False","Equipos diversos - Otros equipos - Costo"
"chart33692","33692","Miscellaneous equipment - Other equipments - Revaluation","asset_fixed","False","Equipos diversos - Otros equipos - Revaluación"
"chart33711","33711","Replacement tools and units - Tools - Costs","asset_fixed","False","Herramientas y unidades de reemplazo - Herramientas - Costo"
"chart33712","33712","Replacement tools and units - Tools - Revaluation","asset_fixed","False","Herramientas y unidades de reemplazo - Herramientas - Revaluación"
"chart33721","33721","Replacement tools and units - Replacement units - Costs","asset_fixed","False","Herramientas y unidades de reemplazo - Unidades de reemplazo - Costo"
"chart33722","33722","Replacement tools and units - Replacement units - Revaluation","asset_fixed","False","Herramientas y unidades de reemplazo - Unidades de reemplazo - Revaluación"
"chart3381","3381","Units to receive - Machinery and exploitation equipment","asset_fixed","False","Unidades por recibir - Maquinarias y equipos de explotación"
"chart3382","3382","Units to receive - Transportation equipment","asset_fixed","False","Unidades por recibir - Equipo de transporte"
"chart3383","3383","Units to receive - Furniture and fixtures","asset_fixed","False","Unidades por recibir - Muebles y enseres"
"chart3386","3386","Units to receive - Miscellaneous equipment","asset_fixed","False","Unidades por recibir - Equipos diversos"
"chart3387","3387","Units to receive - Replacement tools and units","asset_fixed","False","Unidades por recibir - Herramientas y unidades de reemplazo"
"chart3391","3391","Work in progress - Landscaping","asset_fixed","False","Obras en curso - Adecuación de terrenos"
"chart33921","33921","Work in progress - Buildings in progress - Costs","asset_fixed","False","Obras en curso - Edificaciones en curso - Costo"
"chart33922","33922","Work in progress - Buildings in progress - Financing costs","asset_fixed","False","Obras en curso - Edificaciones en curso - Costos de financiación"
"chart33931","33931","Work in progress - Assembly machinery - Costs","asset_fixed","False","Obras en curso - Maquinaria en montaje - Costo"
"chart33932","33932","Work in progress - Assembly machinery - Financing costs","asset_fixed","False","Obras en curso - Maquinaria en montaje - Costos de financiación"
"chart34111","34111","Concessions, licenses and other rights - Rights for concessions - Costs","asset_non_current","False","Concesiones, licencias y otros derechos - Derechos por concesiones - Costo"
"chart34112","34112","Concessions, licenses and other rights - Rights for concessions - Revaluation","asset_non_current","False","Concesiones, licencias y otros derechos - Derechos por concesiones - Revaluación"
"chart34121","34121","Concessions, licenses and other rights - License - Costs","asset_non_current","False","Concesiones, licencias y otros derechos - Licencias - Costo"
"chart34122","34122","Concessions, licenses and other rights - License - Revaluation","asset_non_current","False","Concesiones, licencias y otros derechos - Licencias - Revaluación"
"chart34191","34191","Concessions, licenses and other rights - Other rights - Costs","asset_non_current","False","Concesiones, licencias y otros derechos - Otros derechos - Costo"
"chart34192","34192","Concessions, licenses and other rights - Other rights - Revaluation","asset_non_current","False","Concesiones, licencias y otros derechos - Otros derechos - Revaluación"
"chart34211","34211","Patents and industrial property - Patents - Costs","asset_non_current","False","Patentes y propiedad industrial - Patentes - Costo"
"chart34212","34212","Patents and industrial property - Patents - Revaluation","asset_non_current","False","Patentes y propiedad industrial - Patentes - Revaluación"
"chart34221","34221","Patents and industrial property - Brands - Costs","asset_non_current","False","Patentes y propiedad industrial - Marcas - Costo"
"chart34222","34222","Patents and industrial property - Brands - Revaluation","asset_non_current","False","Patentes y propiedad industrial - Marcas - Revaluación"
"chart34311","34311","Softwares - Computer applications - Costs","asset_non_current","False","Programas de computadora (software) - Aplicaciones informáticas - Costo"
"chart34312","34312","Softwares - Computer applications - Revaluation","asset_non_current","False","Programas de computadora (software) - Aplicaciones informáticas - Revaluación"
"chart34411","34411","Exploration and development costs - Exploration costs - Costs","asset_non_current","False","Costos de exploración y desarrollo - Costos de exploración - Costo"
"chart34412","34412","Exploration and development costs - Exploration costs - Revaluation","asset_non_current","False","Costos de exploración y desarrollo - Costos de exploración - Revaluación"
"chart34413","34413","Exploration and development costs - Exploration costs - Financing costs","asset_non_current","False","Costos de exploración y desarrollo - Costos de exploración - Costos de financiación"
"chart34421","34421","Exploration and development costs - Development costs - Costs","asset_non_current","False","Costos de exploración y desarrollo - Costos de desarrollo - Costo"
"chart34422","34422","Exploration and development costs - Development costs - Revaluation","asset_non_current","False","Costos de exploración y desarrollo - Costos de desarrollo - Revaluación"
"chart34423","34423","Exploration and development costs - Development costs - Financing costs","asset_non_current","False","Costos de exploración y desarrollo - Costos de desarrollo - Costos de financiación"
"chart34511","34511","Formulas, designs and prototypes - Formulas - Costs","asset_non_current","False","Fórmulas, diseños y prototipos - Fórmulas - Costo"
"chart34512","34512","Formulas, designs and prototypes - Formulas - Revaluation","asset_non_current","False","Fórmulas, diseños y prototipos - Fórmulas - Revaluación"
"chart34521","34521","Formulas, designs and prototypes - Designs and prototypes - Costs","asset_non_current","False","Fórmulas, diseños y prototipos - Diseños y prototipos - Costo"
"chart34522","34522","Formulas, designs and prototypes - Designs and prototypes - Revaluation","asset_non_current","False","Fórmulas, diseños y prototipos - Diseños y prototipos - Revaluación"
"chart3471","3471","Goodwill - Goodwill","asset_non_current","False","Plusvalía mercantil - Plusvalía mercantil"
"chart34911","34911","Other intangible assets - Other intangible assets - Costs","asset_non_current","False","Otros activos intangibles - Otros activos intangibles - Costo"
"chart34912","34912","Other intangible assets - Other intangible assets - Revaluation","asset_non_current","False","Otros activos intangibles - Otros activos intangibles - Revaluación"
"chart35111","35111","Biological assets in production - Animal origin - Costs","asset_non_current","False","Activos biológicos en producción - De origen animal - Costo"
"chart35113","35113","Biological assets in production - Animal origin - Financing costs","asset_non_current","False","Activos biológicos en producción - Animal origin - Costos de financiación"
"chart35114","35114","Biological assets in production - Animal origin - Fair value","asset_non_current","False","Activos biológicos en producción - De origen animal - Valor razonable"
"chart35121","35121","Biological assets in production - Vegetal origin - Costs","asset_non_current","False","Activos biológicos en producción - De origen vegetal - Costo"
"chart35123","35123","Biological assets in production - Vegetal origin - Financing costs","asset_non_current","False","Activos biológicos en producción - De origen vegetal - Costos de financiación"
"chart35124","35124","Biological assets in production - Vegetal origin - Fair value","asset_non_current","False","Activos biológicos en producción - De origen vegetal - Valor razonable"
"chart35211","35211","Biological assets under development - Animal origin - Costs","asset_non_current","False","Activos biológicos en desarrollo - De origen animal - Costo"
"chart35213","35213","Biological assets under development - Animal origin - Financing costs","asset_non_current","False","Activos biológicos en desarrollo - De origen animal - Costos de financiación"
"chart35214","35214","Biological assets under development - Animal origin - Fair value","asset_non_current","False","Activos biológicos en desarrollo - De origen animal - Valor razonable"
"chart35221","35221","Biological assets under development - Vegetal origin - Costs","asset_non_current","False","Activos biológicos en desarrollo - De origen animal - Costo"
"chart35223","35223","Biological assets under development - Vegetal origin - Financing costs","asset_non_current","False","Activos biológicos under development - Vegetal origin - Costos de financiación"
"chart35224","35224","Biological assets under development - Vegetal origin - Fair value","asset_non_current","False","Activos biológicos en desarrollo - De origen vegetal - Valor razonable"
"chart36111","36111","Land - Costs","asset_fixed","False","Terrenos - Costo"
"chart36112","36112","Land - Revaluation","asset_fixed","False","Terrenos - Revaluación"
"chart36121","36121","Buildings - Costs","asset_fixed","False","Edificaciones - Costo"
"chart36122","36122","Buildings - Revaluation","asset_fixed","False","Edificaciones - Revaluación"
"chart36123","36123","Buildings - Financing costs","asset_fixed","False","Edificaciones - Costos de financiación"
"chart36131","36131","Constructions in progress - Buildings - Costs","asset_fixed","False","Construcciones en curso - Edificaciones - Costo"
"chart36132","36132","Constructions in progress - Buildings - Revaluation","asset_fixed","False","Construcciones en curso - Edificaciones - Revaluación"
"chart36133","36133","Constructions in progress - Buildings - Financing costs","asset_fixed","False","Construcciones en curso - Edificaciones - Costos de financiación"
"chart36211","36211","Financial leasing - Land - Costs","asset_non_current","False","Arrendamiento financiero - Terrenos - Costo"
"chart36212","36212","Financial leasing - Land - Revaluation","asset_non_current","False","Arrendamiento financiero - Terrenos - Revaluación"
"chart36221","36221","Financial leasing - Buildings - Costs","asset_non_current","False","Arrendamiento financiero - Edificaciones - Costo"
"chart36222","36222","Financial leasing - Buildings - Revaluation","asset_non_current","False","Arrendamiento financiero - Edificaciones - Revaluación"
"chart36223","36223","Financial leasing - Buildings - Financing costs","asset_non_current","False","Arrendamiento financiero - Edificaciones - Costos de financiación"
"chart36311","36311","Financial leasing - Land - Costs","asset_non_current","False","Arrendamiento financiero - Terrenos - Costo"
"chart36312","36312","Financial leasing - Land - Revaluation","asset_non_current","False","Arrendamiento financiero - Terrenos - Revaluación"
"chart36321","36321","Financial leasing - Buildings - Costs","asset_non_current","False","Arrendamiento financiero - Edificaciones - Costo"
"chart36322","36322","Financial leasing - Buildings - Revaluation","asset_non_current","False","Arrendamiento financiero - Edificaciones - Revaluación"
"chart36323","36323","Financial leasing - Buildings - Financing costs","asset_non_current","False","Arrendamiento financiero - Edificaciones - Costos de financiación"
"chart36331","36331","Financial leasing - Machinery and exploitation equipment - Costs","asset_non_current","False","Arrendamiento financiero - Maquinaria y equipo de explotación - Costo"
"chart36332","36332","Financial leasing - Machinery and exploitation equipment - Revaluation","asset_non_current","False","Arrendamiento financiero - Maquinaria y equipo de explotación - Revaluación"
"chart36333","36333","Financial leasing - Machinery and exploitation equipment - Financing costs","asset_non_current","False","Arrendamiento financiero - Maquinaria y equipo de explotación - Costo de financiación"
"chart36341","36341","Financial leasing - Transport units - Costs","asset_non_current","False","Arrendamiento financiero - Unidades de transporte - Costo"
"chart36342","36342","Financial leasing - Transport units - Revaluation","asset_non_current","False","Arrendamiento financiero - Unidades de transporte - Revaluación"
"chart36351","36351","Financial leasing - Furniture and fixtures - Costs","asset_non_current","False","Arrendamiento financiero - Muebles y enseres - Costo"
"chart36352","36352","Financial leasing - Furniture and fixtures - Revaluation","asset_non_current","False","Arrendamiento financiero - Muebles y enseres - Revaluación"
"chart36361","36361","Financial leasing - Miscellaneous equipment - Costs","asset_non_current","False","Arrendamiento financiero - Equipos diversos - Costo"
"chart36362","36362","Financial leasing - Miscellaneous equipment - Revaluation","asset_non_current","False","Arrendamiento financiero - Equipos diversos - Revaluación"
"chart36401","36401","Production plant in production - Costs","asset_fixed","False","Planta productora en producción - Costo"
"chart36402","36402","Production plant in production - Production plant in production - Revaluation","asset_fixed","False","Planta productora en producción - Planta productora en desarrollo - Revaluación"
"chart36403","36403","Production plant in production - Production plant in production - Financing costs","asset_fixed","False",""
"chart36405","36405","Production plant in production - Production plant in development - Costs","asset_fixed","False","Planta productora en producción - Planta productora en desarrollo - Costo"
"chart36406","36406","Production plant in production - Production plant in development - Revaluation","asset_fixed","False",""
"chart36407","36407","Production plant in production - Production plant in development - Financing costs","asset_fixed","False","Planta productora en producción - Planta productora en desarrollo - Costo de financiación"
"chart36408","36408","Production plant in production - Production plant in development - Fair value","asset_fixed","False","Planta productora en producción - Planta productora en desarrollo - Valor razonable"
"chart36411","36411","Land - Costs","asset_fixed","False","Terrenos - Costo"
"chart36412","36412","Land - Revaluation","asset_fixed","False","Terrenos - Revaluación"
"chart36421","36421","Buildings - Buildings - Costs","asset_fixed","False","Edificaciones - Edificaciones - Costo"
"chart36422","36422","Buildings - Buildings - Revaluation","asset_fixed","False","Edificaciones - Edificaciones - Revaluación"
"chart36423","36423","Buildings - Buildings - Financing costs","asset_fixed","False","Edificaciones - Edificaciones - Costos de financiación"
"chart36424","36424","Buildings - Installations - Costs","asset_fixed","False","Edificaciones - Instalaciones - Costo"
"chart36425","36425","Buildings - Installations - Revaluation","asset_fixed","False","Edificaciones - Instalaciones - Revaluación"
"chart36426","36426","Buildings - Installations - Financing costs","asset_fixed","False","Edificaciones - Instalaciones - Costos de financiación"
"chart36427","36427","Buildings - Improvements in leased premises - Costs","asset_fixed","False","Edificaciones - Mejoras en locales arrendados - Costo"
"chart36428","36428","Buildings - Improvements in leased premises - Revaluation","asset_fixed","False","Edificaciones - Mejoras en locales arrendados - Revaluación"
"chart36429","36429","Buildings - Improvements in leased premises - Financing costs","asset_fixed","False","Edificaciones - Mejoras en locales arrendados - Costos de financiación"
"chart36431","36431","Machinery and exploitation equipment - Costs","asset_fixed","False","Maquinarias y equipos de explotación - Costo"
"chart36432","36432","Machinery and exploitation equipment - Revaluation","asset_fixed","False","Maquinarias y equipos de explotación - Revaluación"
"chart36433","36433","Machinery and exploitation equipment - Financing costs","asset_fixed","False","Maquinarias y equipos de explotación - Costos de financiación"
"chart36441","36441","Transport units - Costs","asset_fixed","False","Unidades de transporte - Costo"
"chart36442","36442","Transport units - Revaluation","asset_fixed","False","Unidades de transporte - Revaluación"
"chart36451","36451","Furniture and fixtures - Costs","asset_fixed","False","Muebles y enseres - Costo"
"chart36452","36452","Furniture and fixtures - Revaluation","asset_fixed","False","Muebles y enseres - Revaluación"
"chart36461","36461","Miscellaneous equipment - Costs","asset_fixed","False","Equipos diversos - Costo"
"chart36462","36462","Miscellaneous equipment - Revaluation","asset_fixed","False","Equipos diversos - Revaluación"
"chart36471","36471","Replacement tools and units - Tools - Costs","asset_fixed","False","Herramientas y unidades de reemplazo - Herramientas - Costo"
"chart36491","36491","Work in progress - Costs","asset_fixed","False","Obras en curso - Costo"
"chart36492","36492","Work in progress - Revaluation","asset_fixed","False","Obras en curso - Revaluación"
"chart36511","36511","Concessions, licenses and other rights - Costs","asset_non_current","False","Concesiones, licencias y otros derechos - Costo"
"chart36512","36512","Concessions, licenses and other rights - Revaluation","asset_non_current","False","Concesiones, licencias y otros derechos - Revaluación"
"chart36521","36521","Patents and industrial property - Costs","asset_non_current","False","Patentes y propiedad industrial - Costo"
"chart36522","36522","Patents and industrial property - Revaluation","asset_non_current","False","Patentes y propiedad industrial - Revaluación"
"chart36531","36531","Softwares - Costs","asset_non_current","False","Programas de computadora (software) - Costo"
"chart36532","36532","Softwares - Revaluation","asset_non_current","False","Programas de computadora (software) - Revaluación"
"chart36541","36541","Exploration and development costs - Costs","asset_non_current","False","Costos de exploración y desarrollo - Costo"
"chart36542","36542","Exploration and development costs - Revaluation","asset_non_current","False","Costos de exploración y desarrollo - Revaluación"
"chart36543","36543","Exploration and development costs - Financing costs","asset_non_current","False","Costos de exploración y desarrollo - Costos de financiación"
"chart36551","36551","Formulas, designs and prototypes - Costs","asset_non_current","False","Fórmulas, diseños y prototipos - Costo"
"chart36552","36552","Formulas, designs and prototypes - Revaluation","asset_non_current","False","Fórmulas, diseños y prototipos - Revaluación"
"chart3657","3657","Goodwill","asset_non_current","False","Plusvalía mercantil"
"chart36591","36591","Other intangible assets - Costs","asset_non_current","False","Otros activos intangibles - Costo"
"chart36592","36592","Other intangible assets - Revaluation","asset_non_current","False","Otros activos intangibles - Revaluación"
"chart36611","36611","Biological assets in production - Costs","asset_non_current","False","Activos biológicos en producción - Costo"
"chart36613","36613","Biological assets in production - Financing costs","asset_non_current","False","Activos biológicos en producción - Costos de financiación"
"chart36621","36621","Biological assets under development - Costs","asset_non_current","False","Activos biológicos en desarrollo - Costo"
"chart36622","36622","Biological assets under development - Financing costs","asset_non_current","False","Activos biológicos en desarrollo - Costos de financiación"
"chart36711","36711","Investments to be held until maturity - Costs","asset_non_current","False","Inversiones a ser mantenidas hasta el vencimiento - Costo"
"chart36721","36721","Financial investments representative of patrimonial right - Costs","asset_non_current","False","Inversiones financieras representativas de derecho patrimonial - Costo"
"chart36731","36731","Other financial investments - Costs","asset_non_current","False","Otras inversiones financieras - Costo"
"chart3711","3711","Deferred income tax - Deferred income tax – Heritage","liability_non_current","False","Impuesto a la renta diferido - Impuesto a la renta diferido – Patrimonio"
"chart3712","3712","Deferred income tax - Deferred income tax – Results","liability_non_current","False","Impuesto a la renta diferido - Impuesto a la renta diferido – Resultados"
"chart3721","3721","Deferred employee shares - Deferred employee shares – Heritage","asset_non_current","False","Participaciones de los trabajadores diferidas - Participaciones de los trabajadores diferidas – Patrimonio"
"chart3722","3722","Deferred employee shares - Deferred employee shares – Results","asset_non_current","False","Participaciones de los trabajadores diferidas - Participaciones de los trabajadores diferidas – Resultados"
"chart3731","3731","Deferred interest - Unearned interest in transactions with third parties","asset_non_current","False","Intereses diferidos - Intereses no devengados en transacciones con terceros"
"chart3732","3732","Deferred interest - Unearned interest on discounted value measurement","asset_non_current","False","Intereses diferidos - Intereses no devengados en medición a valor descontado"
"chart3811","3811","Art and culture goods - Works of art","asset_non_current","False","Bienes de arte y cultura - Obras de arte"
"chart3812","3812","Art and culture goods - Library","asset_non_current","False","Bienes de arte y cultura - Biblioteca"
"chart3813","3813","Art and culture goods - Others","asset_non_current","False","Bienes de arte y cultura - Otros"
"chart3821","3821","Misc. - Coins and jewelry","asset_non_current","False","Diversos - Monedas y joyas"
"chart3822","3822","Misc. - Goods delivered on loan","asset_non_current","False","Diversos - Bienes entregados en comodato"
"chart3823","3823","Misc. - Assets received in payment (awarded and realizable)","asset_non_current","False","Diversos - Bienes recibidos en pago (adjudicados y realizables)"
"chart3829","3829","Misc. - Others","asset_non_current","False","Diversos - Otros"
"chart39111","39111","Accumulated depreciation investment properties - Buildings - Costs","asset_non_current","False","Depreciación acumulada propiedades de inversión - Edificaciones - Costo"
"chart39112","39112","Accumulated depreciation investment properties - Buildings - Revaluation","asset_non_current","False","Depreciación acumulada propiedades de inversión - Edificaciones - Revaluación"
"chart39113","39113","Accumulated depreciation investment properties - Buildings - Financing costs","asset_non_current","False","Depreciación acumulada propiedades de inversión - Edificaciones - Costos de financiación"
"chart39211","39211","Accumulated depreciation investment properties - Financial leasing - Buildings - Costs","asset_non_current","False","Depreciación acumulada propiedades de inversión - Arrendamiento financiero - Edificaciones - Costo"
"chart39212","39212","Accumulated depreciation investment properties - Financial leasing - Buildings - Revaluation","asset_non_current","False","Depreciación acumulada propiedades de inversión - Arrendamiento financiero - Edificaciones - Revaluación"
"chart39213","39213","Accumulated depreciation investment properties - Financial leasing - Buildings - Financing costs","asset_non_current","False","Depreciación acumulada propiedades de inversión - Arrendamiento financiero - Edificaciones - Costos de financiación"
"chart39321","39321","Accumulated depreciation property, plant and equipment - Financial leasing - Buildings - Costs","asset_fixed","False","Depreciación acumulada propiedad, planta y equipo - Financial leasing - Edificaciones - Costo"
"chart39322","39322","Accumulated depreciation property, plant and equipment - Financial leasing - Buildings - Revaluation","asset_fixed","False","Depreciación acumulada propiedad, planta y equipo - Financial leasing - Edificaciones - Revaluación"
"chart39323","39323","Accumulated depreciation property, plant and equipment - Financial leasing - Buildings - Financing costs","asset_fixed","False","Depreciación acumulada propiedad, planta y equipo - Financial leasing - Edificaciones - Costos de financiación"
"chart39331","39331","Accumulated depreciation property, plant and equipment - Financial leasing - Machinery and exploitation equipment - Costs","asset_fixed","False","Depreciación acumulada propiedad, planta y equipo - Financial leasing - Maquinarias y equipos de explotación - Costo"
"chart39332","39332","Accumulated depreciation property, plant and equipment - Financial leasing - Machinery and exploitation equipment - Revaluation","asset_fixed","False","Depreciación acumulada propiedad, planta y equipo - Financial leasing - Maquinarias y equipos de explotación - Revaluación"
"chart39333","39333","Accumulated depreciation property, plant and equipment - Financial leasing - Machinery and exploitation equipment - Financing costs","asset_fixed","False","Depreciación acumulada propiedad, planta y equipo - Financial leasing - Maquinarias y equipos de explotación - Costos de financiación"
"chart39341","39341","Accumulated depreciation property, plant and equipment - Financial leasing - Transport units - Costs","asset_fixed","False","Depreciación acumulada propiedad, planta y equipo - Financial leasing - Unidades de transporte - Costo"
"chart39342","39342","Accumulated depreciation property, plant and equipment - Financial leasing - Transport units - Revaluation","asset_fixed","False","Depreciación acumulada propiedad, planta y equipo - Financial leasing - Unidades de transporte - Revaluación"
"chart39351","39351","Accumulated depreciation property, plant and equipment - Financial leasing - Furniture and fixtures - Costs","asset_fixed","False","Depreciación acumulada propiedad, planta y equipo - Financial leasing - Muebles y enseres - Costo"
"chart39361","39361","Accumulated depreciation property, plant and equipment - Financial leasing - Miscellaneous equipment - Costs","asset_fixed","False","Depreciación acumulada propiedad, planta y equipo - Financial leasing - Equipos diversos - Costo"
"chart39362","39362","Accumulated depreciation property, plant and equipment - Financial leasing - Miscellaneous equipment - Revaluation","asset_fixed","False","Depreciación acumulada propiedad, planta y equipo - Financial leasing - Equipos diversos - Revaluación"
"chart39410","39410","Accumulated depreciation - Operating lease - Right-of-use assets - Operating lease - Production plants","asset_non_current","False","Depreciación acumulada - Arrendamiento operativo - Activos por derecho de uso - Arrendamiento operativo - Plantas productoras"
"chart39411","39411","Accumulated depreciation - Operating lease - Right-of-use assets - Operating lease - Land","asset_non_current","False","Depreciación acumulada - Arrendamiento operativo - Activos por derecho de uso - Arrendamiento operativo - Terrenos"
"chart39412","39412","Accumulated depreciation - Operating lease - Right-of-use assets - Operating lease - Buildings","asset_non_current","False","Depreciación acumulada - Arrendamiento operativo - Activos por derecho de uso - Arrendamiento operativo - Edificaciones"
"chart39413","39413","Accumulated depreciation - Operating lease - Right-of-use assets - Operating lease - Machinery and exploitation equipment","asset_non_current","False","Depreciación acumulada - Arrendamiento operativo - Activos por derecho de uso - Arrendamiento operativo - Maquinarias y equipos de explotación"
"chart39414","39414","Accumulated depreciation - Operating lease - Right-of-use assets - Operating lease - Transport units","asset_non_current","False","Depreciación acumulada - Arrendamiento operativo - Activos por derecho de uso - Arrendamiento operativo - Unidades de transporte"
"chart39415","39415","Accumulated depreciation - Operating lease - Right-of-use assets - Operating lease - Miscellaneous equipment","asset_non_current","False","Depreciación acumulada - Arrendamiento operativo - Activos por derecho de uso - Arrendamiento operativo - Equipos diversos"
"chart39520","39520","Accumulated depreciation of property, plant and equipment - Accumulated depreciation - Costs - Production plants","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Plantas productoras"
"chart39521","39521","Accumulated depreciation of property, plant and equipment - Accumulated depreciation - Costs - Buildings","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Edificaciones"
"chart39522","39522","Accumulated depreciation of property, plant and equipment - Accumulated depreciation - Costs - Installations","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Instalaciones"
"chart39523","39523","Accumulated depreciation of property, plant and equipment - Accumulated depreciation - Costs - Improvements in leased premises","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Mejoras en locales arrendados"
"chart39524","39524","Accumulated depreciation of property, plant and equipment - Accumulated depreciation - Costs - Machinery and exploitation equipment","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Maquinarias y equipos de explotación"
"chart39525","39525","Accumulated depreciation of property, plant and equipment - Accumulated depreciation - Costs - Transport units","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Unidades de transporte"
"chart39526","39526","Accumulated depreciation of property, plant and equipment - Accumulated depreciation - Costs - Furniture and fixtures","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Muebles y enseres"
"chart39527","39527","Accumulated depreciation of property, plant and equipment - Accumulated depreciation - Costs - Miscellaneous equipment","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Equipos diversos"
"chart39528","39528","Accumulated depreciation of property, plant and equipment - Accumulated depreciation - Costs - Tools","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Herramientas"
"chart39529","39529","Accumulated depreciation of property, plant and equipment - Accumulated depreciation - Costs - Replacement units","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Depreciación acumulada - Costo - Unidades de reemplazo"
"chart39530","39530","Accumulated depreciation of property, plant and equipment - Property, plant and equipment - Revaluation - Production plants","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Plantas productoras"
"chart39531","39531","Accumulated depreciation of property, plant and equipment - Property, plant and equipment - Revaluation - Buildings","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Edificaciones"
"chart39532","39532","Accumulated depreciation of property, plant and equipment - Property, plant and equipment - Revaluation - Installations","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Instalaciones"
"chart39533","39533","Accumulated depreciation of property, plant and equipment - Property, plant and equipment - Revaluation - Improvements in leased premises","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Mejoras en locales arrendados"
"chart39534","39534","Accumulated depreciation of property, plant and equipment - Property, plant and equipment - Revaluation - Machinery and exploitation equipment","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Maquinarias y equipos de explotación"
"chart39535","39535","Accumulated depreciation of property, plant and equipment - Property, plant and equipment - Revaluation - Transport units","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Unidades de transporte"
"chart39536","39536","Accumulated depreciation of property, plant and equipment - Property, plant and equipment - Revaluation - Furniture and fixtures","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Muebles y enseres"
"chart39537","39537","Accumulated depreciation of property, plant and equipment - Property, plant and equipment - Revaluation - Miscellaneous equipment","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Equipos diversos"
"chart39538","39538","Accumulated depreciation of property, plant and equipment - Property, plant and equipment - Revaluation - Replacement tools and units","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Revaluación - Herramientas y unidades de reemplazo"
"chart39540","39540","Accumulated depreciation of property, plant and equipment - Property, plant and equipment - Financing costs - Production plants","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Costos de financiación - Plantas productoras"
"chart39541","39541","Accumulated depreciation of property, plant and equipment - Property, plant and equipment - Financing costs - Buildings","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Costos de financiación - Edificaciones"
"chart39542","39542","Accumulated depreciation of property, plant and equipment - Property, plant and equipment - Financing costs - Machinery and exploitation equipment","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Costos de financiación - Maquinarias y equipos de explotación"
"chart39550","39550","Accumulated depreciation of property, plant and equipment - Property, plant and equipment - Fair value - Production plants","asset_fixed","False","Depreciación acumulada de propiedad, planta y equipo - Propiedad, planta y equipo - Valor razonable - Plantas productoras"
"chart39611","39611","Accumulated amortization - Intangibles – Costs - Concessions, licenses and other rights","asset_non_current","False","Amortización acumulada - Intangibles – Costo - Concesiones, licencias y otros derechos"
"chart39612","39612","Accumulated amortization - Intangibles – Costs - Patents and industrial property","asset_non_current","False","Amortización acumulada - Intangibles – Costo - Patentes y propiedad industrial"
"chart39613","39613","Accumulated amortization - Intangibles – Costs - Softwares","asset_non_current","False","Amortización acumulada - Intangibles – Costo - Programas de computadora (software)"
"chart39614","39614","Accumulated amortization - Intangibles – Costs - Exploration and development costs","asset_non_current","False","Amortización acumulada - Intangibles – Costo - Costos de exploración y desarrollo"
"chart39615","39615","Accumulated amortization - Intangibles – Costs - Formulas, designs and prototypes","asset_non_current","False","Amortización acumulada - Intangibles – Costo - Fórmulas, diseños y prototipos"
"chart39619","39619","Accumulated amortization - Intangibles – Costs - Other intangible assets","asset_non_current","False","Amortización acumulada - Intangibles – Costo - Otros activos intangibles"
"chart39621","39621","Accumulated amortization - Intangibles – Revaluation - Concessions, licenses and other rights","asset_non_current","False","Amortización acumulada - Intangibles – Revaluación - Concesiones, licencias y otros derechos"
"chart39622","39622","Accumulated amortization - Intangibles – Revaluation - Patents and industrial property","asset_non_current","False","Amortización acumulada - Intangibles – Revaluación - Patentes y propiedad industrial"
"chart39623","39623","Accumulated amortization - Intangibles – Revaluation - Softwares","asset_non_current","False","Amortización acumulada - Intangibles – Revaluación - Programas de computadora (software)"
"chart39624","39624","Accumulated amortization - Intangibles – Revaluation - Exploration and development costs","asset_non_current","False","Amortización acumulada - Intangibles – Revaluación - Costos de exploración y desarrollo"
"chart39625","39625","Accumulated amortization - Intangibles – Revaluation - Formulas, designs and prototypes","asset_non_current","False","Amortización acumulada - Intangibles – Revaluación - Fórmulas, diseños y prototipos"
"chart39629","39629","Accumulated amortization - Intangibles – Revaluation - Other intangible assets","asset_non_current","False","Amortización acumulada - Intangibles – Revaluación - Otros activos intangibles"
"chart39633","39633","Accumulated amortization - Intangibles – Financing costs - Softwares","asset_non_current","False","Amortización acumulada - Intangibles – Costos de financiación - Programas de computadora"
"chart39634","39634","Accumulated amortization - Intangibles – Financing costs - Exploration costs","asset_non_current","False","Amortización acumulada - Intangibles – Costos de financiación - Costos de exploración"
"chart39635","39635","Accumulated amortization - Intangibles – Financing costs - Development costs","asset_non_current","False","Amortización acumulada - Intangibles – Costos de financiación - Costos de desarrollo"
"chart39811","39811","Accumulated depreciation - Biological assets in production - Biological assets in production - Costs - Biological assets in production","asset_non_current","False","Depreciación acumulada - Activos biológicos en producción - Activos biológicos en producción - Costo - Activos biológicos en producción"
"chart40111","40111","National government - General sales tax - IGV – Own account","liability_current","False","Gobierno nacional - Impuesto general a las ventas - IGV – Cuenta propia"
"chart40112","40112","National government - General sales tax - IGV – Services provided by non-residents","liability_current","False","Gobierno nacional - Impuesto general a las ventas - IGV – Servicios prestados por no domiciliados"
"chart40113","40113","National government - General sales tax - IGV – Perception regime","liability_current","False","Gobierno nacional - Impuesto general a las ventas - IGV – Régimen de percepciones"
"chart40114","40114","National government - General sales tax - IGV – Withholding regime","liability_current","False","Gobierno nacional - Impuesto general a las ventas - IGV – Régimen de retenciones"
"chart40115","40115","National government - General sales tax - IGV – Imports","liability_current","False","Gobierno nacional - Impuesto general a las ventas - IGV – Importaciones"
"chart40116","40116","National government - General sales tax - IGV – Intended for taxed operations","liability_current","False","Gobierno nacional - Impuesto general a las ventas - IGV – Destinado a operaciones gravadas"
"chart40117","40117","National government - General sales tax - IGV - Intended for common operations","liability_current","False","Gobierno nacional - Impuesto general a las ventas - IGV - Destinado a operaciones comunes"
"chart4012","4012","National government - Excise tax","liability_current","False","Gobierno nacional - Impuesto selectivo al consumo"
"chart40151","40151","National government - Customs duties - Customs tariff","liability_current","False","Gobierno nacional - Derechos aduaneros - Derechos arancelarios"
"chart40152","40152","National government - Customs duties - Other customs duties","liability_current","False","Gobierno nacional - Derechos aduaneros - Otros derechos arancelarios"
"chart40171","40171","National government - Income tax - Third category income","liability_current","False","Gobierno nacional - Impuesto a la renta - Renta de tercera categoría"
"chart40172","40172","National government - Income tax - Fourth category rent","liability_current","False","Gobierno nacional - Impuesto a la renta - Renta de cuarta categoría"
"chart40173","40173","National government - Income tax - Fifth category income","liability_current","False","Gobierno nacional - Impuesto a la renta - Renta de quinta categoría"
"chart40174","40174","National government - Income tax - Non-domiciled income","liability_current","False","Gobierno nacional - Impuesto a la renta - Renta de no domiciliados"
"chart40175","40175","National government - Income tax - Other retentions","liability_current","False","Gobierno nacional - Impuesto a la renta - Otras retenciones"
"chart40181","40181","National government - Other taxes and considerations - Financial transaction tax","liability_current","False","Gobierno nacional - Otros impuestos y contraprestaciones - Impuesto a las transacciones financieras"
"chart40182","40182","National government - Other taxes and considerations - Tax on casino games and slots","liability_current","False","Gobierno nacional - Otros impuestos y contraprestaciones - Impuesto a los juegos de casino y tragamonedas"
"chart40183","40183","National government - Other taxes and considerations - Fees for the provision of public services","liability_current","False","Gobierno nacional - Otros impuestos y contraprestaciones - Tasas por la prestación de servicios públicos"
"chart40184","40184","National government - Other taxes and considerations - Royalties","liability_current","False","Gobierno nacional - Otros impuestos y contraprestaciones - Regalías"
"chart40185","40185","National government - Other taxes and considerations - Dividends tax","liability_current","False","Gobierno nacional - Otros impuestos y contraprestaciones - Impuesto a los dividendos"
"chart40186","40186","National government - Other taxes and considerations - Temporary tax on net assets","liability_current","False","Gobierno nacional - Otros impuestos y contraprestaciones - Impuesto temporal a los activos netos"
"chart40189","40189","National government - Other taxes and considerations - Other taxes","liability_current","False","Gobierno nacional - Otros impuestos y contraprestaciones - Otros impuestos"
"chart402","402","Tax certificates","liability_current","False","Certificados tributarios"
"chart4031","4031","Public institutions - ESSALUD","liability_current","False","Instituciones públicas - ESSALUD"
"chart4032","4032","Public institutions - ONP","liability_current","False","Instituciones públicas - ONP"
"chart4033","4033","Public institutions - Contribution to SENATI","liability_current","False","Instituciones públicas - Contribución al SENATI"
"chart4034","4034","Public institutions - Contribution to SENCICO","liability_current","False","Instituciones públicas - Contribución al SENCICO"
"chart4039","4039","Public institutions - Otras instituciones","liability_current","False","Instituciones públicas - Other institutions"
"chart405","405","Regional government","liability_current","False","Gobierno regional"
"chart40611","40611","Local governments - Taxes - Vehicle property tax","liability_current","False","Gobiernos locales - Impuestos - Impuesto al patrimonio vehicular"
"chart40612","40612","Local governments - Taxes - Gambling tax","liability_current","False","Gobiernos locales - Impuestos - Impuesto a las apuestas"
"chart40613","40613","Local governments - Taxes - Gaming taxes","liability_current","False","Gobiernos locales - Impuestos - Impuesto a los juegos"
"chart40614","40614","Local governments - Taxes - Impuesto de alcabala","liability_current","False","Gobiernos locales - Impuestos - Impuesto de alcabala"
"chart40615","40615","Local governments - Taxes - Property tax","liability_current","False","Gobiernos locales - Impuestos - Impuesto predial"
"chart40616","40616","Local governments - Taxes - Tax on non-sporting public shows","liability_current","False","Gobiernos locales - Impuestos - Impuesto a los espectáculos públicos no deportivos"
"chart4062","4062","Local governments - Contributions","liability_current","False","Gobiernos locales - Contribuciones"
"chart40631","40631","Local governments - Fees - Establishment opening license","liability_current","False","Gobiernos locales - Tasas - Licencia de apertura de establecimientos"
"chart40632","40632","Local governments - Fees - Public transportation","liability_current","False","Gobiernos locales - Tasas - Transporte público"
"chart40633","40633","Local governments - Fees - Vehicle parking","liability_current","False","Gobiernos locales - Tasas - Estacionamiento de vehículos"
"chart40634","40634","Local governments - Fees - Public or arbitrary services","liability_current","False","Gobiernos locales - Tasas - Servicios públicos o arbitrios"
"chart40635","40635","Local governments - Fees - Administrative services or rights","liability_current","False","Gobiernos locales - Tasas - Servicios administrativos o derechos"
"chart409","409","Other administrative costs and interest","liability_current","False","Otros costos administrativos e intereses"
"chart4111","4111","Remuneration payable - Wages and salaries payable","liability_payable","True","Remuneraciones por pagar - Sueldos y salarios por pagar"
"chart4112","4112","Remuneration payable - Commissions payable","liability_payable","True","Remuneraciones por pagar - Comisiones por pagar"
"chart4113","4113","Remuneration payable - Remuneration in kind payable","liability_payable","True","Remuneraciones por pagar - Remuneraciones en especie por pagar"
"chart4114","4114","Remuneration payable - Gratuities payable","liability_payable","True","Remuneraciones por pagar - Gratificaciones por pagar"
"chart4115","4115","Remuneration payable - Payable vacation","liability_payable","True","Remuneraciones por pagar - Vacaciones por pagar"
"chart413","413","Employee participations payable","liability_payable","True","Participaciones de los trabajadores por pagar"
"chart4151","4151","Employee social benefits payable - Compensation for time of services","liability_payable","True","Beneficios sociales de los trabajadores por pagar - Compensación por tiempo de servicios"
"chart4152","4152","Employee social benefits payable - Advance compensation for length of service","liability_payable","True","Beneficios sociales de los trabajadores por pagar - Adelanto de compensación por tiempo de servicios"
"chart4153","4153","Employee social benefits payable - Pensions and retirement","liability_payable","True","Beneficios sociales de los trabajadores por pagar - Pensiones y jubilaciones"
"chart417","417","Administrators of pension funds","liability_payable","True","Administradoras de fondos de pensiones"
"chart419","419","Other remuneration and shares payable","liability_payable","True","Otras remuneraciones y participaciones por pagar"
"chart4211","4211","Invoices, tickets and other payable vouchers - Not issued","liability_payable","True","Facturas, boletas y otros comprobantes por pagar - No emitidas"
"chart4212","4212","Invoices, tickets and other payable vouchers - Not issued","liability_payable","True","Facturas, boletas y otros comprobantes por pagar - No emitidas"
"chart422","422","Advances to suppliers","liability_payable","True","Anticipos a proveedores"
"chart423","423","Bills to pay","liability_payable","True","Letras por pagar"
"chart424","424","Honors to be payed","liability_payable","True","Honorarios por pagar"
"chart4311","4311","Invoices, tickets and other payable vouchers - Not issued","liability_payable","True","Facturas, boletas y otros comprobantes por pagar - No emitidas"
"chart4312","4312","Invoices, tickets and other payable vouchers - Not issued","liability_payable","True","Facturas, boletas y otros comprobantes por pagar - No emitidas"
"chart4321","4321","Advances granted - Advances granted","liability_payable","True","Anticipos otorgados - Anticipos otorgados"
"chart4331","4331","Bills to pay - Bills to pay","liability_payable","True","Letras por pagar - Letras por pagar"
"chart4341","4341","Honors to be payed - Honors to be payed","liability_payable","True","Honorarios por pagar - Honorarios por pagar"
"chart4411","4411","Shareholders (partners, participants) - Loans","liability_non_current","False","Accionistas (socios, partícipes) - Préstamos"
"chart4412","4412","Shareholders (partners, participants) - Dividends","liability_non_current","False","Accionistas (socios, partícipes) - Dividendos"
"chart4419","4419","Shareholders (partners, participants) - Other accounts payable","liability_non_current","False","Accionistas (socios, partícipes) - Otras cuentas por pagar"
"chart4421","4421","Directors - Subsistence allowance","liability_non_current","False","Directores - Dietas"
"chart4429","4429","Directors - Other accounts payable","liability_non_current","False","Directores - Other accounts payable"
"chart4511","4511","Loans from financial institutions and other entities - Financial institutions","liability_current","False","Préstamos de instituciones financieras y otras entidades - Instituciones financieras"
"chart4512","4512","Loans from financial institutions and other entities - Other entities","liability_current","False","Préstamos de instituciones financieras y otras entidades - Otras entidades"
"chart452","452","Financial lease contracts","liability_current","False","Contratos de arrendamiento financiero"
"chart4531","4531","Bonds issued - Issued bonds","liability_current","False","Obligaciones emitidas - Bonos emitidos"
"chart4532","4532","Bonds issued - Bonds securitized","liability_current","False","Obligaciones emitidas - Bonos titulizados"
"chart4533","4533","Bonds issued - Commercial papers","liability_current","False","Obligaciones emitidas - Papeles comerciales"
"chart4539","4539","Bonds issued - Other obligations","liability_current","False","Obligaciones emitidas - Otras obligaciones"
"chart4541","4541","Other financial instruments payable - Bills","liability_current","False","Otros Instrumentos financieros por pagar - Letras"
"chart4542","4542","Other financial instruments payable - Commercial papers","liability_current","False","Otros Instrumentos financieros por pagar - Papeles comerciales"
"chart4543","4543","Other financial instruments payable - Bonds","liability_current","False","Otros Instrumentos financieros por pagar - Bonos"
"chart4544","4544","Other financial instruments payable - Promissory notes","liability_current","False","Otros Instrumentos financieros por pagar - Pagarés"
"chart4545","4545","Other financial instruments payable - Conformed invoices","liability_current","False","Otros Instrumentos financieros por pagar - Facturas conformadas"
"chart4549","4549","Other financial instruments payable - Other financial obligations","liability_current","False","Otros Instrumentos financieros por pagar - Otras obligaciones financieras"
"chart45511","45511","Financing costs payable - Loans from financial institutions and other entities - Financial institutions","liability_current","False","Costos de financiación por pagar - Préstamos de instituciones financieras y otras entidades - Instituciones financieras"
"chart45512","45512","Financing costs payable - Loans from financial institutions and other entities - Other entities","liability_current","False","Costos de financiación por pagar - Préstamos de instituciones financieras y otras entidades - Otras entidades"
"chart4552","4552","Financing costs payable - Financial lease contracts","liability_current","False","Costos de financiación por pagar - Contratos de arrendamiento financiero"
"chart45531","45531","Financing costs payable - Bonds issued - Issued bonds","liability_current","False","Costos de financiación por pagar - Obligaciones emitidas - Bonos emitidos"
"chart45532","45532","Financing costs payable - Bonds issued - Bonds securitized","liability_current","False","Costos de financiación por pagar - Obligaciones emitidas - Bonos titulizados"
"chart45533","45533","Financing costs payable - Bonds issued - Commercial papers","liability_current","False","Costos de financiación por pagar - Obligaciones emitidas - Papeles comerciales"
"chart45539","45539","Financing costs payable - Bonds issued - Other obligations","liability_current","False","Costos de financiación por pagar - Obligaciones emitidas - Otras obligaciones"
"chart45541","45541","Financing costs payable - Other financial instruments payable - Bills","liability_current","False","Costos de financiación por pagar - Otros instrumentos financieros por pagar - Letras"
"chart45542","45542","Financing costs payable - Other financial instruments payable - Commercial papers","liability_current","False","Costos de financiación por pagar - Otros instrumentos financieros por pagar - Papeles comerciales"
"chart45543","45543","Financing costs payable - Other financial instruments payable - Bonds","liability_current","False","Costos de financiación por pagar - Otros instrumentos financieros por pagar - Bonos"
"chart45544","45544","Financing costs payable - Other financial instruments payable - Promissory notes","liability_current","False","Costos de financiación por pagar - Otros instrumentos financieros por pagar - Pagarés"
"chart45545","45545","Financing costs payable - Other financial instruments payable - Conformed invoices","liability_current","False","Costos de financiación por pagar - Otros instrumentos financieros por pagar - Facturas conformadas"
"chart45549","45549","Financing costs payable - Other financial instruments payable - Other financial obligations","liability_current","False","Costos de financiación por pagar - Otros instrumentos financieros por pagar - Otras obligaciones financieras"
"chart456","456","Loans with repurchase commitments","liability_current","False","Préstamos con compromisos de recompra"
"chart461","461","Third Party Claims","liability_payable","True","Reclamaciones de terceros"
"chart4641","4641","Liabilities for financial instruments - Primary financial instruments","liability_payable","True","Pasivos por instrumentos financieros - Instrumentos financieros primarios"
"chart46421","46421","Liabilities for financial instruments - Derivative financial instruments - Trading book","liability_payable","True","Pasivos por instrumentos financieros - Derivative financial instruments - Cartera de negociación"
"chart46422","46422","Liabilities for financial instruments - Derivative financial instruments - Hedging instruments","liability_payable","True","Pasivos por instrumentos financieros - Derivative financial instruments - Instrumentos de cobertura"
"chart4651","4651","Liabilities for purchase of fixed assets - Securities investments","liability_payable","True","Pasivo por compra de activo inmovilizado - Inversiones mobiliarias"
"chart4652","4652","Liabilities for purchase of fixed assets - Investment property","liability_payable","True","Pasivos por compra de activo inmovilizado - Propiedades de inversión"
"chart4653","4653","Liabilities for purchase of fixed assets - Assets acquired under finance lease","liability_payable","True",""
"chart4654","4654","Liabilities for purchase of fixed assets - Property, plant and equipment","liability_payable","True","Pasivo por compra de activo inmovilizado - Propiedades, planta y equipo"
"chart4655","4655","Liabilities for purchase of fixed assets - Intangibles","liability_payable","True","Pasivos por compra de activo inmovilizado - Intangibles"
"chart4656","4656","Liabilities for purchase of fixed assets - Biological assets","liability_payable","True","Pasivo por compra de activo inmovilizado - Activos biológicos"
"chart466","466","Participation of third parties in joint agreements","liability_payable","True","Participación de terceros en acuerdos conjuntos"
"chart467","467","Deposits received in guarantee","liability_payable","True","Depósitos recibidos en garantía"
"chart4691","4691","Other sundry accounts payable - Government subsidies","liability_payable","True","Otras cuentas por pagar diversas - Subsidios gubernamentales"
"chart4692","4692","Other sundry accounts payable - Conditional donations","liability_payable","True","Otras cuentas por pagar diversas - Donaciones condicionadas"
"chart4699","4699","Other sundry accounts payable - Other accounts payable","liability_payable","True","Otras cuentas por pagar diversas - Other accounts payable"
"chart471","471","Loans","liability_payable","True","Préstamos"
"chart472","472","Financing costs","liability_payable","True","Costos de financiación"
"chart473","473","Advances received","liability_payable","True","Anticipos recibidos"
"chart474","474","Royalties","liability_payable","True","Regalías"
"chart475","475","Dividends","liability_payable","True","Dividendos"
"chart476","476","Deposits received in guarantee","liability_payable","True","Depósitos recibidos en garantía"
"chart4771","4771","Liabilities for purchase of fixed assets - Securities investments","liability_payable","True","Pasivo por compra de activo inmovilizado - Inversiones mobiliarias"
"chart4772","4772","Liabilities for purchase of fixed assets - Investment property","liability_payable","True","Pasivos por compra de activo inmovilizado - Propiedades de inversión"
"chart4773","4773","Liabilities for purchase of fixed assets - Assets acquired under finance lease","liability_payable","True",""
"chart4774","4774","Liabilities for purchase of fixed assets - Property, plant and equipment","liability_payable","True","Pasivo por compra de activo inmovilizado - Propiedades, planta y equipo"
"chart4775","4775","Liabilities for purchase of fixed assets - Intangibles","liability_payable","True","Pasivos por compra de activo inmovilizado - Intangibles"
"chart4776","4776","Liabilities for purchase of fixed assets - Biological assets","liability_payable","True","Pasivo por compra de activo inmovilizado - Activos biológicos"
"chart4791","4791","Other sundry accounts payable - Other sundry accounts payable","liability_payable","True","Otras cuentas por pagar diversas - Otras cuentas por pagar diversas"
"chart481","481","Provision for litigation","liability_non_current","False","Provisión para litigios"
"chart482","482","Provision for dismantling, removal or rehabilitation of fixed assets","liability_non_current","False","Provisión por desmantelamiento, retiro o rehabilitación del inmovilizado"
"chart483","483","Provision for restructurings","liability_non_current","False","Provisión para reestructuraciones"
"chart484","484","Provision for environmental protection and remediation","liability_non_current","False","Provisión para protección y remediación del medio ambiente"
"chart485","485","Provision for social responsibility expenses","liability_non_current","False","Provisión para gastos de responsabilidad social"
"chart486","486","Provision for guarantees","liability_non_current","False","Provisión para garantías"
"chart487","487","Provision for right-of-use assets","liability_non_current","False","Provisión por activos por derecho de uso"
"chart489","489","Other provisions","liability_non_current","False","Otras provisiones"
"chart4911","4911","Deferred income tax - Deferred income tax – Heritage","liability_non_current","False","Impuesto a la renta diferido - Impuesto a la renta diferido – Patrimonio"
"chart4912","4912","Deferred income tax - Deferred income tax – Results","liability_non_current","False","Impuesto a la renta diferido - Impuesto a la renta diferido – Resultados"
"chart4921","4921","Deferred employee shares - Deferred employee shares – Heritage","liability_current","False","Participaciones de los trabajadores diferidas - Participaciones de los trabajadores diferidas – Patrimonio"
"chart4922","4922","Deferred employee shares - Deferred employee shares – Results","liability_current","False","Participaciones de los trabajadores diferidas - Participaciones de los trabajadores diferidas – Resultados"
"chart4931","4931","Deferred interest - Unearned interest in transactions with third parties","liability_current","False","Intereses diferidos - Intereses no devengados en transacciones con terceros"
"chart4932","4932","Deferred interest - Unearned interest on discounted value measurement","liability_current","False","Intereses diferidos - Intereses no devengados en medición a valor descontado"
"chart494","494","Profit on sale with parallel financial lease","liability_current","False","Ganancia en venta con arrendamiento financiero paralelo"
"chart495","495","Deferred subsidies received","liability_current","False","Subsidios recibidos diferidos"
"chart496","496","Deferred income","liability_current","False","Ingresos diferidos"
"chart497","497","Deferred costs","liability_current","False","Costos diferidos"
"chart5011","5011","Capital social - Actions","equity","False","Capital social - Acciones"
"chart5012","5012","Capital social - Participations","equity","False","Capital social - Participaciones"
"chart502","502","Treasury shares","equity","False","Acciones en tesorería"
"chart511","511","Investment shares","equity","False","Acciones de inversión"
"chart512","512","Investment shares in treasury","equity","False","Acciones de inversión en tesorería"
"chart521","521","Premiums (discount) of shares","equity","False","Primas (descuento) de acciones"
"chart5221","5221","Capitalizations in process - Contributions","equity","False","Capitalizaciones en trámite - Aportes"
"chart5222","5222","Capitalizations in process - Reserves","equity","False","Capitalizaciones en trámite - Reservas"
"chart5223","5223","Capitalizations in process - Credits","equity","False","Capitalizaciones en trámite - Acreencias"
"chart5224","5224","Capitalizations in process - Utilities","equity","False","Capitalizaciones en trámite - Utilidades"
"chart523","523","Capital reductions pending formalization","equity","False","Reducciones de capital pendientes de formalización"
"chart561","561","Exchange difference on permanent investments in foreign entities","equity","False","Diferencia en cambio de inversiones permanentes en entidades extranjeras"
"chart562","562","Financial instruments – Hedges","equity","False","Instrumentos financieros – Coberturas"
"chart5631","5631","Result on financial assets or liabilities held for trading - Profit","equity","False","Resultado en activos o pasivos financieros mantenidos para negociación - Ganancia"
"chart5632","5632","Result on financial assets or liabilities held for trading - Lost","equity","False","Resultado en activos o pasivos financieros mantenidos para negociación - Pérdida"
"chart5641","5641","Result in other assets or liabilities for financial investments - Profit","equity","False","Resultado en otros activos o pasivos por inversiones financieras - Ganancia"
"chart5642","5642","Result in other assets or liabilities for financial investments - Lost","equity","False","Resultado en otros activos o pasivos por inversiones financieras - Pérdida"
"chart5651","5651","Result on financial assets or liabilities held for trading – Conventional purchase or sale settlement date - Profit","equity","False","Resultado en activos o pasivos financieros mantenidos para negociación – Compra o venta convencional fecha de liquidación - Ganancia"
"chart5652","5652","Result on financial assets or liabilities held for trading – Conventional purchase or sale settlement date - Lost","equity","False","Resultado en activos o pasivos financieros mantenidos para negociación - Pérdida"
"chart57111","57111","Revaluation surplus - Investment property - Direct acquisition","equity","False","Excedente de revaluación - Propiedad de inversión - Adquisición directa"
"chart57112","57112","Revaluation surplus - Investment property - Financial leasing","equity","False","Excedente de revaluación - Propiedad de inversión - Financial leasing"
"chart57121","57121","Revaluation surplus - Property, plant and equipment - Direct acquisition","equity","False","Excedente de revaluación - Propiedades, planta y equipo - Adquisición directa"
"chart57122","57122","Revaluation surplus - Property, plant and equipment - Financial leasing","equity","False","Excedente de revaluación - Propiedades, planta y equipo - Arrendamiento financiero"
"chart5713","5713","Revaluation surplus - Intangibles","equity","False","Excedente de revaluación - Intangibles"
"chart5714","5714","Revaluation surplus - Right-of-use assets - Operating lease","equity","False","Excedente de revaluación - Activos por derecho de uso - Arrendamiento operativo"
"chart572","572","Revaluation surplus – Released shares received","equity","False","Excedente de revaluación – Acciones liberadas recibidas"
"chart573","573","Participation in revaluation surplus – Investments in related entities","equity","False","Participación en excedente de revaluación – Inversiones en entidades relacionadas"
"chart581","581","Reinvestment","equity","False","Reinversión"
"chart582","582","Legal","equity","False","Legal"
"chart583","583","Contractual","equity","False","Contractuales"
"chart584","584","By-laws","equity","False","Estatutarias"
"chart585","585","Facultative","equity","False","Facultativas"
"chart589","589","Other reserves","equity","False","Otras reservas"
"chart5911","5911","Undistributed profits - Acumulated utilities","equity","False","Utilidades no distribuidas - Utilidades acumuladas"
"chart5912","5912","Undistributed profits - Income from previous years","equity","False","Utilidades no distribuidas - Ingresos de años anteriores"
"chart5921","5921","Accumulated losses - Accumulated losses","equity","False","Pérdidas acumuladas - Pérdidas acumuladas"
"chart5922","5922","Accumulated losses - Expenses from previous years","equity","False","Pérdidas acumuladas - Gastos de años anteriores"
"chart6011","6011","Merchandise - Merchandise","expense","False","Mercaderías - Mercaderías"
"chart602","602","Raw Materials","expense","False","Materias primas"
"chart6031","6031","Auxiliary materials, supplies and spare parts - Auxiliary materials","expense","False","Materiales auxiliares, suministros y repuestos - Materiales auxiliares"
"chart6032","6032","Auxiliary materials, supplies and spare parts - Supplies","expense","False","Materiales auxiliares, suministros y repuestos - Suministros"
"chart6033","6033","Auxiliary materials, supplies and spare parts - Spare parts","expense","False","Materiales auxiliares, suministros y repuestos - Repuestos"
"chart6041","6041","Containers and packaging - Containers","expense","False","Envases y embalajes - Envases"
"chart6042","6042","Containers and packaging - Packaging","expense","False","Envases y embalajes - Embalajes"
"chart60911","60911","Costs linked to purchases - Costs associated with purchases of merchandise - Transport","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de mercaderías - Transporte"
"chart60912","60912","Costs linked to purchases - Costs associated with purchases of merchandise - Insurance","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de mercaderías - Seguros"
"chart60913","60913","Costs linked to purchases - Costs associated with purchases of merchandise - Customs duties","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de mercaderías - Derechos aduaneros"
"chart60914","60914","Costs linked to purchases - Costs associated with purchases of merchandise - Commissions","expense","False","Costos vinculados con las compras - Costs associated with purchases of merchandise - Comisiones"
"chart60919","60919","Costs linked to purchases - Costs associated with purchases of merchandise - Other costs","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de mercaderías - Otros costos"
"chart60921","60921","Costs linked to purchases - Costs linked to purchases of raw materials - Transport","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de materias primas - Transporte"
"chart60922","60922","Costs linked to purchases - Costs linked to purchases of raw materials - Insurance","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de materias primas - Seguros"
"chart60923","60923","Costs linked to purchases - Costs linked to purchases of raw materials - Customs duties","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de materias primas - Derechos aduaneros"
"chart60924","60924","Costs linked to purchases - Costs linked to purchases of raw materials - Commissions","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de materias primas - Comisiones"
"chart60925","60925","Costs linked to purchases - Costs linked to purchases of raw materials - Other costs","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de materias primas - Otros costos"
"chart60931","60931","Costos vinculados con las compras - Costs associated with purchases of materials, supplies and spare parts - Transport","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de materiales, suministros y repuestos - Transporte"
"chart60932","60932","Costos vinculados con las compras - Costs associated with purchases of materials, supplies and spare parts - Insurance","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de materiales, suministros y repuestos - Seguros"
"chart60933","60933","Costos vinculados con las compras - Costs associated with purchases of materials, supplies and spare parts - Customs duties","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de materiales, suministros y repuestos - Derechos aduaneros"
"chart60934","60934","Costos vinculados con las compras - Costs associated with purchases of materials, supplies and spare parts - Commissions","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de materiales, suministros y repuestos - Comisiones"
"chart60935","60935","Costos vinculados con las compras - Costs associated with purchases of materials, supplies and spare parts - Other costs","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de materiales, suministros y repuestos - Otros costos"
"chart60941","60941","Costos vinculados con las compras - Costs linked to purchases of containers and packaging - Transport","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de envases y embalajes - Transporte"
"chart60942","60942","Costos vinculados con las compras - Costs linked to purchases of containers and packaging - Insurance","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de envases y embalajes - Seguros"
"chart60943","60943","Costos vinculados con las compras - Costs linked to purchases of containers and packaging - Customs duties","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de envases y embalajes - Derechos aduaneros"
"chart60944","60944","Costos vinculados con las compras - Costs linked to purchases of containers and packaging - Commissions","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de envases y embalajes - Comisiones"
"chart60945","60945","Costos vinculados con las compras - Costs linked to purchases of containers and packaging - Other costs","expense","False","Costos vinculados con las compras - Costos vinculados con las compras de envases y embalajes - Otros costos"
"chart6111","6111","Merchandise - Merchandise","expense","False","Mercaderías - Mercaderías"
"chart6121","6121","Raw Materials - Raw Materials","expense","False","Materias primas - Materias primas"
"chart6131","6131","Auxiliary materials, supplies and spare parts - Auxiliary materials","expense","False","Materiales auxiliares, suministros y repuestos - Materiales auxiliares"
"chart6132","6132","Auxiliary materials, supplies and spare parts - Supplies","expense","False","Materiales auxiliares, suministros y repuestos - Suministros"
"chart6133","6133","Auxiliary materials, supplies and spare parts - Spare parts","expense","False","Materiales auxiliares, suministros y repuestos - Repuestos"
"chart6141","6141","Containers and packaging - Containers","expense","False","Envases y embalajes - Envases"
"chart6142","6142","Containers and packaging - Packaging","expense","False","Envases y embalajes - Embalajes"
"chart6211","6211","Remuneration - Wages and salaries","expense","False","Remuneraciones - Sueldos y salarios"
"chart6212","6212","Remuneration - Commissions","expense","False","Remuneraciones - Comisiones"
"chart6213","6213","Remuneration - Remuneration in kind","expense","False","Remuneraciones - Remuneraciones en especie"
"chart6214","6214","Remuneration - Bonuses","expense","False","Remuneraciones - Gratificaciones"
"chart6215","6215","Remuneration - Holidays","expense","False","Remuneraciones - Vacaciones"
"chart622","622","Other remunerations","expense","False","Otras remuneraciones"
"chart623","623","Staff compensation","expense","False","Indemnizaciones al personal"
"chart624","624","Training","expense","False","Capacitación"
"chart625","625","Attentions to staff","expense","False","Atención al personal"
"chart6271","6271","Security, social security and other contributions - Health benefits scheme","expense","False","Seguridad, previsión social y otras contribuciones - Régimen de prestaciones de salud"
"chart6272","6272","Security, social security and other contributions - Pension scheme - Company contribution","expense","False","Seguridad, previsión social y otras contribuciones - Régimen de pensiones - Aporte de empresa"
"chart6273","6273","Security, social security and other contributions - Complementary insurance for risky work, accidents at work and occupational diseases","expense","False","Seguridad, previsión social y otras contribuciones - Seguro complementario de trabajo de riesgo, accidentes de trabajo y enfermedades profesionales"
"chart6274","6274","Security, social security and other contributions - Life insurance","expense","False","Seguridad, previsión social y otras contribuciones - Seguro de vida"
"chart6275","6275","Security, social security and other contributions - Private insurance for health benefits – EPS and other individuals","expense","False","Seguridad, previsión social y otras contribuciones - Seguros particulares de prestaciones de salud – EPS y otros particulares"
"chart6276","6276","Security, social security and other contributions - Fisherman's Social Security Benefits Fund","expense","False","Seguridad, previsión social y otras contribuciones - Caja de beneficios de seguridad social del pescador"
"chart6277","6277","Security, social security and other contributions - Contributions to SENATI","expense","False","Seguridad, previsión social y otras contribuciones - Contribuciones al SENATI"
"chart628","628","Board remuneration","expense","False","Retribuciones al directorio"
"chart6291","6291","Employee social benefits - Compensation for time of service","expense","False","Beneficios sociales de los trabajadores - Compensación por tiempo de servicio"
"chart6292","6292","Employee social benefits - Pensions and retirement","expense","False","Beneficios sociales de los trabajadores - Pensiones y jubilaciones"
"chart6293","6293","Employee social benefits - Other post-employment benefits","expense","False","Beneficios sociales de los trabajadores - Otros beneficios post-empleo"
"chart62941","62941","Employee social benefits - Profit sharing - Current share","expense","False","Beneficios sociales de los trabajadores - Participación en las utilidades - Participación corriente"
"chart62942","62942","Employee social benefits - Profit sharing - Deferred share","expense","False","Beneficios sociales de los trabajadores - Participación en las utilidades - Participación diferida"
"chart63111","63111","Transport, mail and travel expenses - Transport - Charging","expense","False","Transporte, correos y gastos de viaje - Transporte - De carga"
"chart63112","63112","Transport, mail and travel expenses - Transport - Passengers","expense","False","Transporte, correos y gastos de viaje - Transporte - De pasajeros"
"chart6312","6312","Transport, mail and travel expenses - Post office","expense","False","Transporte, correos y gastos de viaje - Correos"
"chart6313","6313","Transport, mail and travel expenses - Lodgement","expense","False","Transporte, correos y gastos de viaje - Alojamiento"
"chart6314","6314","Transport, mail and travel expenses - Alimentation","expense","False","Transporte, correos y gastos de viaje - Alimentación"
"chart6315","6315","Transport, mail and travel expenses - Other travel expenses","expense","False","Transporte, correos y gastos de viaje - Otros gastos de viaje"
"chart6321","6321","Advice and consultancy - Administrative","expense","False","Asesoría y consultoría - Administrativa"
"chart6322","6322","Advice and consultancy - Legal and tax","expense","False","Asesoría y consultoría - Legal y tributaria"
"chart6323","6323","Advice and consultancy - Audit and accounting","expense","False","Asesoría y consultoría - Auditoría y contable"
"chart6324","6324","Advice and consultancy - Marketing","expense","False","Asesoría y consultoría - Mercadotecnia"
"chart6325","6325","Advice and consultancy - Environmental","expense","False","Asesoría y consultoría - Medioambiental"
"chart6326","6326","Advice and consultancy - Investigation and development","expense","False","Asesoría y consultoría - Investigación y desarrollo"
"chart6327","6327","Advice and consultancy - Production","expense","False","Asesoría y consultoría - Producción"
"chart6329","6329","Advice and consultancy - Others","expense","False","Asesoría y consultoría - Otros"
"chart633","633","Production commissioned to third parties","expense","False","Producción encargada a terceros"
"chart6341","6341","Maintenance and repairs - Investment property","expense","False","Mantenimiento y reparaciones - Propiedad de inversión"
"chart63421","63421","Maintenance and repairs - Right-of-use assets - Financial","expense","False","Mantenimiento y reparaciones - Activos por derecho de uso - Financiero"
"chart63432","63432","Maintenance and repairs - Property, plant and equipment - Operational","expense","False","Mantenimiento y reparaciones - Propiedades, planta y equipo - Operativo"
"chart6344","6344","Maintenance and repairs - Intangibles","expense","False","Mantenimiento y reparaciones - Intangibles"
"chart6345","6345","Maintenance and repairs - Biological assets","expense","False","Mantenimiento y reparaciones - Activos biológicos"
"chart6351","6351","Leasings - Land","expense","False","Alquileres - Terrenos"
"chart6352","6352","Leasings - Buildings","expense","False","Alquileres - Edificaciones"
"chart6353","6353","Leasings - Machinery and exploitation equipment","expense","False","Alquileres - Maquinarias y equipos de explotación"
"chart6354","6354","Leasings - Transportation equipment","expense","False","Alquileres - Equipo de transporte"
"chart6355","6355","Leasings - Furniture and fixtures","expense","False","Alquileres - Muebles y enseres"
"chart6356","6356","Leasings - Miscellaneous equipment","expense","False","Alquileres - Equipos diversos"
"chart6361","6361","Basic services - Electric power","expense","False","Servicios básicos - Energía eléctrica"
"chart6362","6362","Basic services - Gas","expense","False","Servicios básicos - Gas"
"chart6363","6363","Basic services - Water","expense","False","Servicios básicos - Agua"
"chart6364","6364","Basic services - Phone","expense","False",""
"chart6365","6365","Basic services - Internet","expense","False","Servicios básicos - Internet"
"chart6366","6366","Basic services - Radio","expense","False","Servicios básicos - Radio"
"chart6367","6367","Basic services - Cable","expense","False","Servicios básicos - Cable"
"chart6371","6371","Advertising, publications, public relations - Advertising","expense","False","Publicidad, publicaciones, relaciones públicas - Publicidad"
"chart6372","6372","Advertising, publications, public relations - Publications","expense","False","Publicidad, publicaciones, relaciones públicas - Publicaciones"
"chart6373","6373","Advertising, publications, public relations - Public relations","expense","False","Publicidad, publicaciones, relaciones públicas - Relaciones públicas"
"chart638","638","Contractor Services","expense","False","Servicios de contratistas"
"chart6391","6391","Other services provided by third parties - Banking expenses","expense","False","Otros servicios prestados por terceros - Gastos bancarios"
"chart6392","6392","Other services provided by third parties - Laboratory expenses","expense","False","Otros servicios prestados por terceros - Gastos de laboratorio"
"chart6411","6411","National government - General sales tax and selective consumption","expense","False","Gobierno nacional - Impuesto general a las ventas y selectivo al consumo"
"chart6412","6412","National government - Financial transaction tax","expense","False","Gobierno nacional - Impuesto a las transacciones financieras"
"chart6413","6413","National government - Temporary tax on net assets","expense","False","Gobierno nacional - Impuesto temporal a los activos netos"
"chart6414","6414","National government - Tax on casino games and slot machines","expense","False","Gobierno nacional - Impuesto a los juegos de casino y máquinas tragamonedas"
"chart6415","6415","National government - Mining royalties","expense","False","Gobierno nacional - Regalías mineras"
"chart6416","6416","National government - Royalties","expense","False","Gobierno nacional - Cánones"
"chart6419","6419","National government - Others","expense","False","Gobierno nacional - Otros"
"chart642","642","Regional government","expense","False","Gobierno regional"
"chart6431","6431","Local government - Property tax","expense","False","Gobierno local - Impuesto predial"
"chart6432","6432","Local government - Municipal taxes and citizen security","expense","False","Gobierno local - Arbitrios municipales y seguridad ciudadana"
"chart6433","6433","Local government - Vehicle property tax","expense","False","Gobierno local - Impuesto al patrimonio vehicular"
"chart6434","6434","Local government - Operating license","expense","False","Gobierno local - Licencia de funcionamiento"
"chart6439","6439","Local government - Others","expense","False","Gobierno local - Otros"
"chart6442","6442","Other tax expenses - Contribution to SENCICO","expense","False","Otros gastos por tributos - Contribución al SENCICO"
"chart6443","6443","Other tax expenses - Others","expense","False","Otros gastos por tributos - Otros"
"chart6451","6451","Tax debt expenses - Interests","expense","False","Gastos en deuda tributaria - Intereses"
"chart6452","6452","Tax debt expenses - Interests - Subdivision","expense","False","Gastos en deuda tributaria - Intereses - Fraccionamiento"
"chart6453","6453","Tax debt expenses - Fines","expense","False","Gastos en deuda tributaria - Multas"
"chart6454","6454","Tax debt expenses - Costs and others","expense","False","Gastos en deuda tributaria - Costas y otros"
"chart651","651","Insurance","expense","False","Seguros"
"chart652","652","Royalties","expense","False","Regalías"
"chart653","653","Subscriptions","expense","False","Suscripciones"
"chart654","654","Licenses and validity rights","expense","False","Licencias y derechos de vigencia"
"chart65511","65511","Net cost of disposal of fixed assets and operations discontinued - Net cost of disposal of fixed assets - Securities investments","expense","False","Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Costo neto de enajenación de activos inmovilizados - Inversiones mobiliarias"
"chart65512","65512","Net cost of disposal of fixed assets and operations discontinued - Net cost of disposal of fixed assets - Investment property","expense","False",""
"chart65513","65513","Net cost of disposal of fixed assets and operations discontinued - Net cost of disposal of fixed assets - Right-of-use assets - Financial leasing","expense","False","Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Costo neto de enajenación de activos inmovilizados - Activos por derecho de uso - Arrendamiento financiero"
"chart65514","65514","Net cost of disposal of fixed assets and operations discontinued - Net cost of disposal of fixed assets - Property, plant and equipment","expense","False","Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Costo neto de enajenación de activos inmovilizados - Propiedades, planta y equipo"
"chart65515","65515","Net cost of disposal of fixed assets and operations discontinued - Net cost of disposal of fixed assets - Intangibles","expense","False","Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Costo neto de enajenación de activos inmovilizados - Intangibles"
"chart65516","65516","Net cost of disposal of fixed assets and operations discontinued - Net cost of disposal of fixed assets - Biological assets","expense","False","Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Costo neto de enajenación de activos inmovilizados - Activos biológicos"
"chart65521","65521","Net cost of disposal of fixed assets and operations discontinued - Discontinued operations – Asset abandonment - Investment property","expense","False","Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Operaciones discontinuadas – Abandono de activos - Propiedades de inversión"
"chart65522","65522","Net cost of disposal of fixed assets and operations discontinued - Discontinued operations – Asset abandonment - Right-of-use assets - Financial leasing","expense","False","Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Operaciones discontinuadas – Abandono de activos - Activos por derecho de uso - Financial leasing"
"chart65523","65523","Net cost of disposal of fixed assets and operations discontinued - Discontinued operations – Asset abandonment - Property, plant and equipment","expense","False","Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Operaciones discontinuadas – Abandono de activos - Propiedades, planta y equipo"
"chart65524","65524","Net cost of disposal of fixed assets and operations discontinued - Discontinued operations – Asset abandonment - Intangibles","expense","False","Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Operaciones discontinuadas – Abandono de activos - Intangibles"
"chart65525","65525","Net cost of disposal of fixed assets and operations discontinued - Discontinued operations – Asset abandonment - Biological assets","expense","False","Costo neto de enajenación de activos inmovilizados y operaciones discontinuadas - Operaciones discontinuadas – Abandono de activos - Activos biológicos"
"chart656","656","Supplies","expense","False","Suministros"
"chart658","658","Environmental management","expense","False","Gestión medioambiental"
"chart6591","6591","Other management fees - Donations","expense","False","Otros gastos de gestión - Donaciones"
"chart6592","6592","Other management fees - Administrative sanctions","expense","False","Otros gastos de gestión - Sanciones administrativas"
"chart6611","6611","Realizable asset - Merchandise","expense","False","Activo realizable - Mercaderías"
"chart6612","6612","Realizable asset - Finished products","expense","False","Activo realizable - Productos terminados"
"chart66131","66131","Realizable asset - Non-current assets held for sale - Investment property","expense","False","Recuperación de cuentas de valuación - Recuperación – Desvalorización de inversiones mobiliarias"
"chart66132","66132","Realizable asset - Non-current assets held for sale - Property, plant and equipment","expense","False","Activo realizable - Activos no corrientes mantenidos para la venta - Propiedades, planta y equipo"
"chart66133","66133","Realizable asset - Non-current assets held for sale - Intangibles","expense","False","Activo realizable - Activos no corrientes mantenidos para la venta - Intangibles"
"chart66134","66134","Realizable asset - Non-current assets held for sale - Biological assets","expense","False","Activo realizable - Non-current assets held for sale - Activos biológicos"
"chart6621","6621","Fixed asset - Investment property","expense","False","Activo inmovilizado - Propiedades de inversión"
"chart6622","6622","Fixed asset - Biological assets","expense","False","Activo inmovilizado - Activos biológicos"
"chart6711","6711","Expenses in debt operations and others - Loans from financial institutions and other entities","expense","False","Gastos en operaciones de endeudamiento y otros - Préstamos de instituciones financieras y otras entidades"
"chart6712","6712","Expenses in debt operations and others - Financial lease contracts","expense","False","Gastos en operaciones de endeudamiento y otros - Contratos de arrendamiento financiero"
"chart6713","6713","Expenses in debt operations and others - Issuance and placement of instruments representing debt and equity","expense","False","Gastos en operaciones de endeudamiento y otros - Emisión y colocación de instrumentos representativos de deuda y patrimonio"
"chart6714","6714","Expenses in debt operations and others - Documents sold or discounted","expense","False","Gastos en operaciones de endeudamiento y otros - Documentos vendidos o descontados"
"chart672","672","Loss on derivative financial instruments","expense","False","Pérdida por instrumentos financieros derivados"
"chart67311","67311","Interest on loans and other obligations - Loans from financial institutions and other entities - Financial institutions","expense","False","Intereses por préstamos y otras obligaciones - Préstamos de instituciones financieras y otras entidades - Instituciones financieras"
"chart67312","67312","Interest on loans and other obligations - Loans from financial institutions and other entities - Other entities","expense","False","Intereses por préstamos y otras obligaciones - Préstamos de instituciones financieras y otras entidades - Otras entidades"
"chart6732","6732","Interest on loans and other obligations - Financial lease contracts","expense","False","Intereses por préstamos y otras obligaciones - Contratos de arrendamiento financiero"
"chart6733","6733","Interest on loans and other obligations - Other financial instruments payable","expense","False","Intereses por préstamos y otras obligaciones - Otros instrumentos financieros por pagar"
"chart6734","6734","Interest on loans and other obligations - Documents sold or discounted","expense","False","Intereses por préstamos y otras obligaciones - Documentos vendidos o descontados"
"chart6735","6735","Interest on loans and other obligations - Bonds issued","expense","False","Intereses por préstamos y otras obligaciones - Obligaciones emitidas"
"chart6736","6736","Interest on loans and other obligations - Commercial obligations","expense","False","Intereses por préstamos y otras obligaciones - Obligaciones comerciales"
"chart6741","6741","Expenses in factoring operations - Loss on instruments sold","expense","False","Gastos en operaciones de factoraje (factoring) - Pérdida en instrumentos vendidos"
"chart675","675","Discounts granted for early payment","expense","False","Descuentos concedidos por pronto pago"
"chart676","676","Exchange rate","expense","False","Diferencia de cambio"
"chart6771","6771","Loss from measurement of financial assets and liabilities at fair value - Investments held for trading","expense","False","Pérdida por medición de activos y pasivos financieros al valor razonable - Inversiones mantenidas para negociación"
"chart6772","6772","Loss from measurement of financial assets and liabilities at fair value - Other financial investments","expense","False","Pérdida por medición de activos y pasivos financieros al valor razonable - Otras inversiones financieras"
"chart6773","6773","Loss from measurement of financial assets and liabilities at fair value - Others","expense","False","Pérdida por medición de activos y pasivos financieros al valor razonable - Otros"
"chart6781","6781","Participation in results of related entities - Participation in the results of subsidiaries and associates under the equity value method","expense","False","Participación en resultados de entidades relacionadas - Participación en los resultados de subsidiarias y asociadas bajo el método del valor patrimonial"
"chart6782","6782","Participation in results of related entities - Shares in joint ventures","expense","False","Participación en resultados de entidades relacionadas - Participaciones en negocios conjuntos"
"chart6791","6791","Other financial expenses - Option premiums","expense","False","Otros gastos financieros - Primas por opciones"
"chart6792","6792","Other financial expenses - Financial expenses measured at discounted value","expense","False","Otros gastos financieros - Gastos financieros en medición a valor descontado"
"chart6793","6793","Other financial expenses - Financial expenses in updating assets for right of use","expense","False","Otros gastos financieros - Gastos financieros en actualización de activos por derecho de uso"
"chart68111","68111","Depreciation of investment properties - Buildings - Costs","expense_depreciation","False","Depreciación de propiedades de inversión - Edificaciones - Costo"
"chart68112","68112","Depreciation of investment properties - Buildings - Revaluation","expense_depreciation","False","Depreciación de propiedades de inversión - Edificaciones - Revaluación"
"chart68113","68113","Depreciation of investment properties - Buildings - Financing costs","expense_depreciation","False","Depreciación de propiedades de inversión - Edificaciones - Costos de financiación"
"chart682111","682111","Depreciation of assets for right of use - Financial leasing - Investment property - Buildings - Costs","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento financiero - Propiedades de inversión - Edificaciones - Costo"
"chart682112","682112","Depreciation of assets for right of use - Financial leasing - Investment property - Buildings - Revaluation","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento financiero - Propiedades de inversión - Edificaciones - Revaluación"
"chart682113","682113","Depreciation of assets for right of use - Financial leasing - Investment property - Buildings - Financing costs","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento financiero - Propiedades de inversión - Edificaciones - Costos de financiación"
"chart682211","682211","Depreciation of assets for right of use - Financial leasing - Property, plant and equipment - Buildings - Costs","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento financiero - Propiedades, planta y equipo - Edificaciones - Costo"
"chart682212","682212","Depreciation of assets for right of use - Financial leasing - Property, plant and equipment - Buildings - Revaluation","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento financiero - Propiedades, planta y equipo - Edificaciones - Revaluación"
"chart682213","682213","Depreciation of assets for right of use - Financial leasing - Property, plant and equipment - Buildings - Financing costs","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento financiero - Propiedades, planta y equipo - Edificaciones - Costos de financiación"
"chart682221","682221","Depreciation of assets for right of use - Financial leasing - Property, plant and equipment - Machinery and exploitation equipment - Costs","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento financiero - Propiedades, planta y equipo - Maquinarias y equipos de explotación - Costo"
"chart682222","682222","Depreciation of assets for right of use - Financial leasing - Property, plant and equipment - Machinery and exploitation equipment - Revaluation","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento financiero - Propiedades, planta y equipo - Maquinarias y equipos de explotación - Revaluación"
"chart682223","682223","Depreciation of assets for right of use - Financial leasing - Property, plant and equipment - Machinery and exploitation equipment - Financing costs","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento financiero - Propiedades, planta y equipo - Maquinarias y equipos de explotación - Costo de financiación"
"chart682231","682231","Depreciation of assets for right of use - Financial leasing - Property, plant and equipment - Transport units - Costs","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento financiero - Propiedades, planta y equipo - Unidades de transporte - Costo"
"chart682232","682232","Depreciation of assets for right of use - Financial leasing - Property, plant and equipment - Transport units - Revaluation","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento financiero - Propiedades, planta y equipo - Unidades de transporte - Revaluación"
"chart682251","682251","Depreciation of assets for right of use - Financial leasing - Property, plant and equipment - Miscellaneous equipment - Costs","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento financiero - Propiedades, planta y equipo - Equipos diversos - Costo"
"chart682252","682252","Depreciation of assets for right of use - Financial leasing - Property, plant and equipment - Miscellaneous equipment - Revaluation","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento financiero - Propiedades, planta y equipo - Equipos diversos - Revaluación"
"chart683111","683111","Depreciation of assets for right of use - Operating lease - Depreciation of assets for right of use - Operating lease - Buildings - Costs","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento operativo - Depreciación de activos por derecho de uso - Arrendamiento operativo - Edificaciones - Costo"
"chart683112","683112","Depreciation of assets for right of use - Operating lease - Depreciation of assets for right of use - Operating lease - Buildings - Revaluation","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento operativo - Depreciación de activos por derecho de uso - Arrendamiento operativo - Edificaciones - Revaluación"
"chart683121","683121","Depreciation of assets for right of use - Operating lease - Depreciation of assets for right of use - Operating lease - Machinery and exploitation equipment - Costs","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento operativo - Depreciación de activos por derecho de uso - Arrendamiento operativo - Maquinarias y equipos de explotación - Costo"
"chart683122","683122","Depreciation of assets for right of use - Operating lease - Depreciation of assets for right of use - Operating lease - Machinery and exploitation equipment - Revaluation","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento operativo - Depreciación de activos por derecho de uso - Arrendamiento operativo - Maquinarias y equipos de explotación - Revaluación"
"chart683131","683131","Depreciation of assets for right of use - Operating lease - Depreciation of assets for right of use - Operating lease - Transport units - Costs","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento operativo - Depreciación de activos por derecho de uso - Arrendamiento operativo - Unidades de transporte - Costo"
"chart683132","683132","Depreciation of assets for right of use - Operating lease - Depreciation of assets for right of use - Operating lease - Transport units - Revaluation","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento operativo - Depreciación de activos por derecho de uso - Arrendamiento operativo - Unidades de transporte - Revaluación"
"chart683152","683152","Depreciation of assets for right of use - Operating lease - Depreciation of assets for right of use - Operating lease - Miscellaneous equipment - Revaluation","expense_depreciation","False","Depreciación de activos por derecho de uso - Arrendamiento operativo - Equipos diversos - Revaluación"
"chart68410","68410","Depreciation of property, plant and equipment - Depreciation of property, plant and equipment - Costs - Production plants","expense_depreciation","False","Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Plantas productoras"
"chart68411","68411","Depreciation of property, plant and equipment - Depreciation of property, plant and equipment - Costs - Buildings","expense_depreciation","False","Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Edificaciones"
"chart68412","68412","Depreciation of property, plant and equipment - Depreciation of property, plant and equipment - Costs - Machinery and exploitation equipment","expense_depreciation","False","Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Maquinarias y equipos de explotación"
"chart68413","68413","Depreciation of property, plant and equipment - Depreciation of property, plant and equipment - Costs - Transport units","expense_depreciation","False","Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Unidades de transporte"
"chart68414","68414","Depreciation of property, plant and equipment - Depreciation of property, plant and equipment - Costs - Furniture and fixtures","expense_depreciation","False","Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Muebles y enseres"
"chart68415","68415","Depreciation of property, plant and equipment - Depreciation of property, plant and equipment - Costs - Miscellaneous equipment","expense_depreciation","False","Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Equipos diversos"
"chart68416","68416","Depreciation of property, plant and equipment - Depreciation of property, plant and equipment - Costs - Replacement tools and units","expense_depreciation","False","Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costo - Herramientas y unidades de reemplazo"
"chart68420","68420","Depreciation of property, plant and equipment - Depreciation of property, plant and equipment - Revaluation - Production plants","expense_depreciation","False","Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Plantas productoras"
"chart68421","68421","Depreciation of property, plant and equipment - Depreciation of property, plant and equipment - Revaluation - Buildings","expense_depreciation","False","Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Edificaciones"
"chart68422","68422","Depreciation of property, plant and equipment - Depreciation of property, plant and equipment - Revaluation - Machinery and exploitation equipment","expense_depreciation","False","Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Maquinarias y equipos de explotación"
"chart68423","68423","Depreciation of property, plant and equipment - Depreciation of property, plant and equipment - Revaluation - Transport units","expense_depreciation","False","Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Unidades de transporte"
"chart68424","68424","Depreciation of property, plant and equipment - Depreciation of property, plant and equipment - Revaluation - Furniture and fixtures","expense_depreciation","False","Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Muebles y enseres"
"chart68425","68425","Depreciation of property, plant and equipment - Depreciation of property, plant and equipment - Revaluation - Miscellaneous equipment","expense_depreciation","False","Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Equipos diversos"
"chart68426","68426","Depreciation of property, plant and equipment - Depreciation of property, plant and equipment - Revaluation - Replacement tools and units","expense_depreciation","False","Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Revaluación - Herramientas y unidades de reemplazo"
"chart68430","68430","Depreciation of property, plant and equipment - Depreciation of property, plant and equipment - Financing costs - Production plants","expense_depreciation","False","Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costos de financiación - Plantas productoras"
"chart68431","68431","Depreciation of property, plant and equipment - Depreciation of property, plant and equipment - Financing costs - Buildings","expense_depreciation","False","Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costos de financiación - Edificaciones"
"chart68432","68432","Depreciation of property, plant and equipment - Depreciation of property, plant and equipment - Financing costs - Machinery and exploitation equipment","expense_depreciation","False","Depreciación de propiedad, planta y equipo - Depreciación de propiedad, planta y equipo - Costos de financiación - Maquinarias y equipos de explotación"
"chart68511","68511","Depreciation of biological assets in production - Depreciation of biological assets in production - Costs - Biological assets of animal origin","expense_depreciation","False","Depreciación de activos biológicos en producción - Depreciación de activos biológicos en producción - Costo - Activos biológicos de origen animal"
"chart68512","68512","Depreciation of biological assets in production - Depreciation of biological assets in production - Costs - Biological asset of vegetal origin","expense_depreciation","False","Depreciación de activos biológicos en producción - Depreciación de activos biológicos en producción - Costo - Activos biológicos de origen vegetal"
"chart68521","68521","Depreciation of biological assets in production - Depreciation of biological assets in production - Financing costs - Biological assets of animal origin","expense_depreciation","False","Depreciación de activos biológicos en producción - Depreciación de activos biológicos en producción - Costo de financiación - Activos biológicos de origen animal"
"chart68522","68522","Depreciation of biological assets in production - Depreciation of biological assets in production - Financing costs - Biological asset of vegetal origin","expense_depreciation","False","Depreciación de activos biológicos en producción - Depreciación de activos biológicos en producción - Costo de financiación - Activos biológicos de origen vegetal"
"chart68611","68611","Amortization of intangibles - Amortization of intangibles – Costs - Concessions, licenses and other rights","expense_depreciation","False","Amortización de intangibles - Amortización de intangibles – Costo - Concesiones, licencias y otros derechos"
"chart68612","68612","Amortization of intangibles - Amortization of intangibles – Costs - Patents and industrial property","expense_depreciation","False","Amortización de intangibles - Amortización de intangibles – Costo - Patentes y propiedad industrial"
"chart68613","68613","Amortization of intangibles - Amortization of intangibles – Costs - Softwares","expense_depreciation","False","Amortización de intangibles - Amortización de intangibles – Costo - Programas de computadora (software)"
"chart68614","68614","Amortization of intangibles - Amortization of intangibles – Costs - Exploration and development costs","expense_depreciation","False","Amortización de intangibles - Amortización de intangibles – Costo - Costo de exploración y desarrollo"
"chart68615","68615","Amortization of intangibles - Amortization of intangibles – Costs - Formulas, designs and prototypes","expense_depreciation","False","Amortización de intangibles - Amortización de intangibles – Costo - Fórmulas, diseños y prototipos"
"chart68619","68619","Amortization of intangibles - Amortization of intangibles – Costs - Other intangible assets","expense_depreciation","False","Amortización de intangibles - Amortización de intangibles – Costo - Otros activos intangibles"
"chart68621","68621","Amortization of intangibles - Amortization of intangibles – Revaluation - Concessions, licenses and other rights","expense_depreciation","False","Amortización de intangibles - Amortización de intangibles – Revaluación - Concesiones, licencias y otros derechos"
"chart68622","68622","Amortization of intangibles - Amortization of intangibles – Revaluation - Patents and industrial property","expense_depreciation","False","Amortización de intangibles - Amortización de intangibles – Revaluación - Patentes y propiedad industrial"
"chart68623","68623","Amortization of intangibles - Amortization of intangibles – Revaluation - Softwares","expense_depreciation","False","Amortización de intangibles - Amortización de intangibles – Revaluación - Programas de computadora (software)"
"chart68624","68624","Amortization of intangibles - Amortization of intangibles – Revaluation - Exploration and development costs","expense_depreciation","False","Amortización de intangibles - Amortización de intangibles – Revaluación - Costos de exploración y desarrollo"
"chart68625","68625","Amortization of intangibles - Amortization of intangibles – Revaluation - Formulas, designs and prototypes","expense_depreciation","False","Amortización de intangibles - Amortización de intangibles – Revaluación - Fórmulas, diseños y prototipos"
"chart68629","68629","Amortization of intangibles - Amortization of intangibles – Revaluation - Other intangible assets","expense_depreciation","False","Amortización de intangibles - Amortización de intangibles – Revaluación - Otros activos intangibles"
"chart68711","68711","Asset valuation - Estimation of doubtful accounts receivable - Trade accounts receivable – Third parties","expense_depreciation","False","Valuación de activos - Estimación de cuentas de cobranza dudosa - Cuentas por cobrar comerciales – Terceros"
"chart68712","68712","Asset valuation - Estimation of doubtful accounts receivable - Trade accounts receivable – Associated","expense_depreciation","False","Valuación de activos - Estimación de cuentas de cobranza dudosa - Cuentas por cobrar comerciales – Relacionadas"
"chart68713","68713","Asset valuation - Estimation of doubtful accounts receivable - Accounts receivable from staff, shareholders (partners) and directors","expense_depreciation","False","Valuación de activos - Estimación de cuentas de cobranza dudosa - Cuentas por cobrar al personal, a los accionistas (socios) y directores"
"chart68714","68714","Asset valuation - Estimation of doubtful accounts receivable - Various accounts receivable – Third parties","expense_depreciation","False","Valuación de activos - Estimación de cuentas de cobranza dudosa - Cuentas por cobrar diversas – Terceros"
"chart68715","68715","Asset valuation - Estimation of doubtful accounts receivable - Various accounts receivable – Associated","expense_depreciation","False","Valuación de activos - Estimación de cuentas de cobranza dudosa - Cuentas por cobrar diversas – Relacionadas"
"chart68731","68731","Asset valuation - Depreciation of investment property - Investments to be held until maturity","expense_depreciation","False","Valuación de activos - Desvalorización de inversiones mobiliarias - Inversiones a ser mantenidas hasta el vencimiento"
"chart68732","68732","Asset valuation - Depreciation of investment property - Financial instruments representative of economic rights","expense_depreciation","False","Valuación de activos - Desvalorización de inversiones mobiliarias - Instrumentos financieros representativos de derecho patrimonial"
"chart68812","68812","Impairment of assets - Impairment of investment property - Buildings","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de propiedad de inversión - Edificaciones"
"chart68813","68813","Impairment of assets - Impairment of investment property - Constructions in progress","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de propiedad de inversión - Construcciones en curso"
"chart68820","68820","Impairment of assets - Impairment of assets for right of use - Financial leasing - Production plant in production","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - Arrendamiento financiero - Planta productora en producción"
"chart68821","68821","Impairment of assets - Impairment of assets for right of use - Financial leasing - Production plant in development","expense_depreciation","False",""
"chart68822","68822","Impairment of assets - Impairment of assets for right of use - Financial leasing - Land","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - Arrendamiento financiero - Terrenos"
"chart68823","68823","Impairment of assets - Impairment of assets for right of use - Financial leasing - Buildings","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - Arrendamiento financiero - Edificaciones"
"chart68824","68824","Impairment of assets - Impairment of assets for right of use - Financial leasing - Machinery and exploitation equipment","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - Arrendamiento financiero - Maquinarias y equipos de explotación"
"chart68825","68825","Impairment of assets - Impairment of assets for right of use - Financial leasing - Transport units","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - Arrendamiento financiero - Unidades de transporte"
"chart68826","68826","Impairment of assets - Impairment of assets for right of use - Financial leasing - Furniture and fixtures","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - Arrendamiento financiero - Muebles y enseres"
"chart68827","68827","Impairment of assets - Impairment of assets for right of use - Financial leasing - Miscellaneous equipment","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - Arrendamiento financiero - Equipos diversos"
"chart68828","68828","Impairment of assets - Impairment of assets for right of use - Financial leasing - Replacement tools and units","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de activos por derecho de uso - Arrendamiento financiero - Herramientas y unidades de reemplazo"
"chart68830","68830","Impairment of assets - Impairment of property, plant and equipment - Production plant in production","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Planta productora en producción"
"chart68831","68831","Impairment of assets - Impairment of property, plant and equipment - Production plant in development","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Planta productora en desarrollo"
"chart68832","68832","Impairment of assets - Impairment of property, plant and equipment - Land","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Terrenos"
"chart68833","68833","Impairment of assets - Impairment of property, plant and equipment - Buildings","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Edificaciones"
"chart68834","68834","Impairment of assets - Impairment of property, plant and equipment - Machinery and exploitation equipment","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Maquinarias y equipos de explotación"
"chart68835","68835","Impairment of assets - Impairment of property, plant and equipment - Transport units","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Unidades de transporte"
"chart68836","68836","Impairment of assets - Impairment of property, plant and equipment - Furniture and fixtures","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Muebles y enseres"
"chart68837","68837","Impairment of assets - Impairment of property, plant and equipment - Miscellaneous equipment","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Equipos diversos"
"chart68838","68838","Impairment of assets - Impairment of property, plant and equipment - Replacement tools and units","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de propiedad, planta y equipo - Herramientas y unidades de reemplazo"
"chart68841","68841","Impairment of assets - Impairment of intangibles - Concessions, licenses and other rights","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de intangibles - Concesiones, licencias y otros derechos"
"chart68842","68842","Impairment of assets - Impairment of intangibles - Patents and industrial property","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de intangibles - Patentes y propiedad industrial"
"chart68843","68843","Impairment of assets - Impairment of intangibles - Softwares","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de intangibles - Programas de computadora (software)"
"chart68844","68844","Impairment of assets - Impairment of intangibles - Exploration and development costs","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de intangibles - Costo de exploración y desarrollo"
"chart68845","68845","Impairment of assets - Impairment of intangibles - Formulas, designs and prototypes","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de intangibles - Fórmulas, diseños y prototipos"
"chart68846","68846","Impairment of assets - Impairment of intangibles - Other intangible assets","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de intangibles - Otros activos intangibles"
"chart68847","68847","Impairment of assets - Impairment of intangibles - Goodwill","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de intangibles - Plusvalía mercantil"
"chart68891","68891","Impairment of assets - Impairment of biological assets in production - Biological assets of animal origin","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de activos biológicos en producción - Activos biológicos de origen animal"
"chart68892","68892","Impairment of assets - Impairment of biological assets in production - Biological asset of vegetal origin","expense_depreciation","False","Deterioro del valor de los activos - Desvalorización de activos biológicos en producción - Activos biológicos de origen vegetal"
"chart68911","68911","Provisions - Provision for litigation - Provision for litigation - Costs","expense_depreciation","False","Provisiones - Provisión para litigios - Provisión para litigios – Costo"
"chart68912","68912","Provisions - Provision for litigation - Provision for litigation – Financial update","expense_depreciation","False","Provisiones - Provisión para litigios - Provisión para litigios – Actualización financiera"
"chart68921","68921","Provisions - Provision for dismantling, removal or rehabilitation of fixed assets - Provision for dismantling, removal or rehabilitation of fixed assets – Costs","expense_depreciation","False","Provisiones - Provisión por desmantelamiento, retiro o rehabilitación del inmovilizado - Provisión por desmantelamiento, retiro o rehabilitación del inmovilizado – Costo"
"chart68922","68922","Provisions - Provision for dismantling, removal or rehabilitation of fixed assets - Provision for dismantling, removal or rehabilitation of fixed assets – Financial update","expense_depreciation","False","Provisiones - Provisión por desmantelamiento, retiro o rehabilitación del inmovilizado - Provisión por desmantelamiento, retiro o rehabilitación del inmovilizado – Actualización financiera"
"chart6893","6893","Provisions - Provision for restructurings","expense_depreciation","False","Provisiones - Provisión para reestructuraciones"
"chart68941","68941","Provisions - Provision for environmental protection and remediation - Provision for environmental protection and remediation – Costs","expense_depreciation","False","Provisiones - Provisión para protección y remediación del medio ambiente - Provisión para protección y remediación del medio ambiente – Costo"
"chart68942","68942","Provisions - Provision for environmental protection and remediation - Provision for environmental protection and remediation – Financial update","expense_depreciation","False","Provisiones - Provisión para protección y remediación del medio ambiente - Provisión para protección y remediación del medio ambiente – Actualización financiera"
"chart68961","68961","Provisions - Provision for guarantees - Provision for guarantees – Costs","expense_depreciation","False","Provisiones - Provisión para garantías - Provisión para garantías – Costo"
"chart68962","68962","Provisions - Provision for guarantees - Provision for guarantees – Financial update","expense_depreciation","False","Provisiones - Provisión para garantías - Provisión para garantías – Actualización financiera"
"chart68971","68971","Provisions - Provision for right-of-use assets - Provision for right-of-use assets operating lease","expense_depreciation","False","Provisiones - Provisión por activos por derecho de uso - Provisión por activos por derecho de uso arrendamiento operativo"
"chart68972","68972","Provisions - Provision for right-of-use assets - Provision for right-of-use assets operating lease - Financial update","expense_depreciation","False","Provisiones - Provisión por activos por derecho de uso - Provisión por activos por derecho de uso arrendamiento operativo - Actualización financiera"
"chart6899","6899","Provisions - Other provisions","expense_depreciation","False","Provisiones - Otras provisiones"
"chart69111","69111","Merchandise - Merchandise - Exportation - Third parties","expense_direct_cost","False","Mercaderías - Mercaderías - Exportación - Terceros"
"chart69112","69112","Merchandise - Merchandise - Exportation - Associated","expense_direct_cost","False","Mercaderías - Mercaderías - Exportación - Relacionadas"
"chart69121","69121","Merchandise - Merchandise - Local sale - Third parties","expense_direct_cost","False","Mercaderías - Mercaderías - Venta local - Terceros"
"chart69122","69122","Merchandise - Merchandise - Local sale - Associated","expense_direct_cost","False","Mercaderías - Mercaderías - Venta local - Relacionadas"
"chart69211","69211","Finished products - Finished products - Exportation - Third parties","expense_direct_cost","False","Productos terminados - Productos terminados - Exportación - Terceros"
"chart69212","69212","Finished products - Finished products - Exportation - Associated","expense_direct_cost","False","Productos terminados - Productos terminados - Exportación - Relacionadas"
"chart69221","69221","Finished products - Finished products - Local sale - Third parties","expense_direct_cost","False","Productos terminados - Productos terminados - Venta local - Terceros"
"chart69222","69222","Finished products - Finished products - Local sale - Associated","expense_direct_cost","False","Productos terminados - Productos terminados - Venta local - Relacionadas"
"chart69231","69231","Finished products - Financing costs – Finished products - Third parties","expense_direct_cost","False","Productos terminados - Costos de financiación - Productos terminados - Terceros"
"chart69232","69232","Finished products - Financing costs – Finished products - Associated","expense_direct_cost","False","Productos terminados - Costos de financiación - Productos terminados - Relacionadas"
"chart6924","6924","Finished products - Unabsorbed production costs – Finished products","expense_direct_cost","False","Productos terminados - Costos de producción no absorbido – Productos terminados"
"chart6925","6925","Finished products - Cost of inefficiency – Finished products","expense_direct_cost","False","Productos terminados - Costo de ineficiencia – Finished products"
"chart69311","69311","Finished services - Services – Exportation - Third parties","expense_direct_cost","False","Servicios terminados - Servicios – exportación - Terceros"
"chart69312","69312","Finished services - Services – Exportation - Associated","expense_direct_cost","False","Servicios terminados - Servicios – Exportación - Relacionadas"
"chart69321","69321","Finished services - Services – Local - Third parties","expense_direct_cost","False","Servicios terminados - Servicios – Local - Terceros"
"chart69322","69322","Finished services - Services – Local - Associated","expense_direct_cost","False",""
"chart69411","69411","By-products, waste and scrap - By-products - Third parties","expense_direct_cost","False","Subproductos, desechos y desperdicios - Subproductos - Terceros"
"chart69412","69412","By-products, waste and scrap - By-products - Associated","expense_direct_cost","False","Subproductos, desechos y desperdicios - Subproductos - Relacionadas"
"chart69421","69421","By-products, waste and scrap - Scrap and waste - Third parties","expense_direct_cost","False","Subproductos, desechos y desperdicios - Desechos y desperdicios - Terceros"
"chart69422","69422","By-products, waste and scrap - Scrap and waste - Associated","expense_direct_cost","False","Subproductos, desechos y desperdicios - Desechos y desperdicios - Relacionadas"
"chart6951","6951","Expenses for impairment of inventories at cost - Merchandise","expense_direct_cost","False","Gastos por desvalorización de inventarios al costo - Mercaderías"
"chart6952","6952","Expenses for impairment of inventories at cost - Finished products","expense_direct_cost","False","Gastos por desvalorización de inventarios al costo - Productos terminados"
"chart6953","6953","Expenses for impairment of inventories at cost - By-products, waste and scrap","expense_direct_cost","False","Gastos por desvalorización de inventarios al costo - Subproductos, desechos y desperdicios"
"chart6954","6954","Expenses for impairment of inventories at cost - Products in process","expense_direct_cost","False","Gastos por desvalorización de inventarios al costo - Productos en proceso"
"chart6955","6955","Expenses for impairment of inventories at cost - Raw Materials","expense_direct_cost","False","Gastos por desvalorización de inventarios al costo - Materias primas"
"chart6956","6956","Expenses for impairment of inventories at cost - Auxiliary materials, supplies and spare parts","expense_direct_cost","False","Gastos por desvalorización de inventarios al costo - Materiales auxiliares, suministros y repuestos"
"chart6957","6957","Expenses for impairment of inventories at cost - Containers and packaging","expense_direct_cost","False","Gastos por desvalorización de inventarios al costo - Envases y embalajes"
"chart6958","6958","Expenses for impairment of inventories at cost - Inventories to be received","expense_direct_cost","False","Gastos por desvalorización de inventarios al costo - Inventarios por recibir"
"chart70111","70111","Merchandise - Merchandise - Export sale - Third parties","income","False","Mercaderías - Mercaderías - Venta de exportación - Terceros"
"chart70112","70112","Merchandise - Merchandise - Export sale - Associated","income","False","Mercaderías - Mercaderías - Venta de exportación - Relacionadas"
"chart70121","70121","Merchandise - Merchandise - Local sale - Third parties","income","False","Mercaderías - Mercaderías - Venta local - Terceros"
"chart70122","70122","Merchandise - Merchandise - Local sale - Associated","income","False","Mercaderías - Mercaderías - Venta local - Relacionadas"
"chart70211","70211","Finished products - Finished products - Export sale - Third parties","income","False","Productos terminados - Productos terminados - Venta de exportación - Terceros"
"chart70212","70212","Finished products - Finished products - Export sale - Associated","income","False","Productos terminados - Productos terminados - Venta de exportación - Relacionadas"
"chart70221","70221","Finished products - Finished products - Local sale - Third parties","income","False","Productos terminados - Productos terminados - Venta local - Terceros"
"chart70222","70222","Finished products - Finished products - Local sale - Associated","income","False","Productos terminados - Productos terminados - Venta local - Relacionadas"
"chart70311","70311","Finished services - Services – Exportation - Third parties","income","False","Servicios terminados - Servicios – exportación - Terceros"
"chart70312","70312","Finished services - Services – Exportation - Associated","income","False","Servicios terminados - Servicios – Exportación - Relacionadas"
"chart70321","70321","Finished services - Services – Local - Third parties","income","False","Servicios terminados - Servicios – Local - Terceros"
"chart70322","70322","Finished services - Services – Local - Associated","income","False",""
"chart70411","70411","By-products, waste and scrap - By-products - Third parties","income","False","Subproductos, desechos y desperdicios - Subproductos - Terceros"
"chart70412","70412","By-products, waste and scrap - By-products - Associated","income","False","Subproductos, desechos y desperdicios - Subproductos - Relacionadas"
"chart70421","70421","By-products, waste and scrap - Scrap and waste - Third parties","income","False","Subproductos, desechos y desperdicios - Desechos y desperdicios - Terceros"
"chart70422","70422","By-products, waste and scrap - Scrap and waste - Associated","income","False","Subproductos, desechos y desperdicios - Desechos y desperdicios - Relacionadas"
"chart70911","70911","Sales returns - Merchandise - Export sale - Third parties","income","False","Devoluciones sobre ventas - Mercaderías - Venta de exportación - Terceros"
"chart70912","70912","Sales returns - Merchandise - Export sale - Associated","income","False","Devoluciones sobre ventas - Mercaderías - Venta de exportación - Relacionadas"
"chart70921","70921","Sales returns - Merchandise - Local sale - Third parties","income","False","Devoluciones sobre ventas - Mercaderías - Venta local - Terceros"
"chart70922","70922","Sales returns - Merchandise - Local sale - Associated","income","False","Devoluciones sobre ventas - Mercaderías - Venta local - Relacionadas"
"chart70931","70931","Sales returns - Finished products - Export sale - Third parties","income","False","Devoluciones sobre ventas - Productos terminados - Venta de exportación - Terceros"
"chart70932","70932","Sales returns - Finished products - Export sale - Associated","income","False","Devoluciones sobre ventas - Productos terminados - Venta de exportación - Relacionadas"
"chart70941","70941","Sales returns - Finished products - Local sale - Third parties","income","False","Devoluciones sobre ventas - Productos terminados - Venta local - Terceros"
"chart70942","70942","Sales returns - Finished products - Local sale - Associated","income","False","Devoluciones sobre ventas - Productos terminados - Venta local - Relacionadas"
"chart70951","70951","Sales returns - Rejected service inventories - Third parties","income","False","Devoluciones sobre ventas - Inventarios de servicios rechazados - Terceros"
"chart70952","70952","Sales returns - Rejected service inventories - Associated","income","False","Devoluciones sobre ventas - Inventarios de servicios rechazados - Relacionadas"
"chart70961","70961","Sales returns - By-products, waste and scrap - Third parties","income","False","Devoluciones sobre ventas - Subproductos, desechos y desperdicios - Terceros"
"chart70962","70962","Sales returns - By-products, waste and scrap - Associated","income","False","Devoluciones sobre ventas - Subproductos, desechos y desperdicios - Relacionadas"
"chart7111","7111","Variation of finished products - Finished products","income_other","False","Variación de productos terminados - Productos terminados"
"chart7121","7121","Variation of by-products, waste and scrap - By-products","income_other","False","Variación de subproductos, desechos y desperdicios - Subproductos"
"chart7122","7122","Variation of by-products, waste and scrap - Scrap and waste","income_other","False","Variación de subproductos, desechos y desperdicios - Desechos y desperdicios"
"chart7131","7131","Variation of products in process - Products in manufacturing process","income_other","False","Variación de productos en proceso - Productos en proceso de manufactura"
"chart7141","7141","Variation of containers and packaging - Containers","income_other","False","Variación de envases y embalajes - Envases"
"chart7142","7142","Variation of containers and packaging - Packaging","income_other","False","Variación de envases y embalajes - Embalajes"
"chart7151","7151","Variation of services inventories - Inventories of services in process","income_other","False","Variación de inventarios de servicios - Inventarios de servicios en proceso"
"chart7211","7211","Investment property - Buildings","income_other","False","Propiedades de inversión - Edificaciones"
"chart7220","7220","Property, plant and equipment- Producing plant","income_other","False","Propiedad, planta y equipo - Planta productora"
"chart7221","7221","Property, plant and equipment- Buildings","income_other","False","Propiedad, planta y equipo - Edificaciones"
"chart7222","7222","Property, plant and equipment- Machinery and other operating equipment","income_other","False","Propiedad, planta y equipo - Maquinarias y otros equipos de explotación"
"chart7223","7223","Property, plant and equipment- Transport units","income_other","False","Propiedad, planta y equipo - Unidades de transporte"
"chart7224","7224","Property, plant and equipment- Furniture and fixtures","income_other","False","Propiedad, planta y equipo - Muebles y enseres"
"chart7225","7225","Property, plant and equipment- Miscellaneous equipment","income_other","False","Propiedad, planta y equipo - Equipos diversos"
"chart7231","7231","Intangibles - Softwares","income_other","False","Intangibles - Programas de computadora (software)"
"chart7232","7232","Intangibles - Exploration and development costs","income_other","False","Intangibles - Costos de exploración y desarrollo"
"chart7233","7233","Intangibles - Formulas, designs and prototypes","income_other","False","Intangibles - Fórmulas, diseños y prototipos"
"chart7241","7241","Biological assets - Biological assets in development of animal origin","income_other","False","Activos biológicos - Activos biológicos en desarrollo de origen animal"
"chart7242","7242","Biological assets - Biological assets in development of vegetable origin","income_other","False","Activos biológicos - Activos biológicos en desarrollo de origen vegetal"
"chart72511","72511","Capitalized financing costs - Financing costs – Investment property - Production plants under development","income_other","False","Costos de financiación capitalizados - Costos de financiación – Propiedades de inversión - Plantas productoras en desarrollo"
"chart72512","72512","Capitalized financing costs - Financing costs – Investment property - Buildings","income_other","False","Costos de financiación capitalizados - Costos de financiación – Propiedades de inversión - Edificaciones"
"chart72521","72521","Capitalized financing costs - Financing costs – Property, plant and equipment - Production plants in development","income_other","False","Costos de financiación capitalizados - Costos de financiación – Propiedad, planta y equipo - Plantas productoras en desarrollo"
"chart72522","72522","Capitalized financing costs - Financing costs – Property, plant and equipment - Buildings","income_other","False","Costos de financiación capitalizados - Costos de financiación – Propiedad, planta y equipo - Edificaciones"
"chart72523","72523","Capitalized financing costs - Financing costs – Property, plant and equipment - Machinery and other operating equipment","income_other","False","Costos de financiación capitalizados - Costos de financiación – Propiedad, planta y equipo - Maquinarias y otros equipos de explotación"
"chart7253","7253","Capitalized financing costs - Financing costs – Intangibles","income_other","False","Costos de financiación capitalizados - Costos de financiación – Intangibles"
"chart72541","72541","Capitalized financing costs - Financing costs – Biological assets under development - Biological assets of animal origin","income_other","False","Costos de financiación capitalizados - Costos de financiación – Activos biológicos en desarrollo - Activos biológicos de origen animal"
"chart72542","72542","Capitalized financing costs - Financing costs – Biological assets under development - Biological asset of vegetal origin","income_other","False","Costos de financiación capitalizados - Costos de financiación – Activos biológicos en desarrollo - Activos biológicos de origen vegetal"
"chart7311","7311","Discounts, rebates and bonuses obtained - Third parties","income","False","Descuentos, rebajas y bonificaciones obtenidos - Terceros"
"chart7312","7312","Discounts, rebates and bonuses obtained - Associated","income","False","Descuentos, rebajas y bonificaciones obtenidos - Relacionadas"
"chart7411","7411","Discounts, rebates and bonuses granted - Third parties","income","False","Descuentos, rebajas y bonificaciones concedidos - Terceros"
"chart7412","7412","Discounts, rebates and bonuses granted - Associated","income","False","Descuentos, rebajas y bonificaciones concedidos - Relacionadas"
"chart751","751","Services for the benefit of staff","income_other","False","Servicios en beneficio del personal"
"chart752","752","Commissions and brokerage","income_other","False","Comisiones y corretajes"
"chart753","753","Royalties","income_other","False","Regalías"
"chart7540","7540","Leasings - Production plants","income_other","False","Alquileres - Plantas productoras"
"chart7541","7541","Leasings - Land","income_other","False","Alquileres - Terrenos"
"chart7542","7542","Leasings - Buildings","income_other","False","Alquileres - Edificaciones"
"chart7543","7543","Leasings - Machinery and exploitation equipment","income_other","False","Alquileres - Maquinarias y equipos de explotación"
"chart7544","7544","Leasings - Transport units","income_other","False","Alquileres - Unidades de transporte"
"chart7545","7545","Leasings - Miscellaneous equipment","income_other","False","Alquileres - Equipos diversos"
"chart7551","7551","Recovery of valuation accounts - Recovery – Doubtful accounts","income_other","False","Recuperación de cuentas de valuación - Recuperación – Cuentas de cobranza dudosa"
"chart7552","7552","Recovery of valuation accounts - Recovery – Inventory impairment","income_other","False","Recuperación de cuentas de valuación - Recuperación – Desvalorización de inventarios"
"chart7553","7553","Recovery of valuation accounts - Recovery – Depreciation of investment property","income_other","False","Recuperación de cuentas de valuación - Recuperación – Desvalorización de inversiones mobiliarias"
"chart7561","7561","Disposal of fixed assets - Securities investments","income_other","False","Enajenación de activos inmovilizados - Inversiones mobiliarias"
"chart7562","7562","Disposal of fixed assets - Investment property","income_other","False","Enajenación de activos inmovilizados - Propiedades de inversión"
"chart7563","7563","Disposal of fixed assets - Assets acquired under finance lease","income_other","False","Enajenación de activos inmovilizados - Activos adquiridos en arrendamiento financiero"
"chart7564","7564","Disposal of fixed assets - Property, plant and equipment","income_other","False","Enajenación de activos inmovilizados - Propiedades, planta y equipo"
"chart7565","7565","Disposal of fixed assets - Intangibles","income_other","False","Enajenación de activos inmovilizados - Intangibles"
"chart7566","7566","Disposal of fixed assets - Biological assets","income_other","False","Enajenación de activos inmovilizados - Activos biológicos"
"chart7571","7571","Recovery of impairment of fixed asset accounts - Recovery of impairment of investment properties","income_other","False","Recuperación de deterioro de cuentas de activos inmovilizados - Recuperación de deterioro de propiedades de inversión"
"chart7572","7572","Recovery of impairment of fixed asset accounts - Recovery of impairment of property, plant and equipment","income_other","False","Recuperación de deterioro de cuentas de activos inmovilizados - Recuperación de deterioro de propiedad, planta y equipo"
"chart7573","7573","Recovery of impairment of fixed asset accounts - Recovery of impairment of intangibles","income_other","False","Recuperación de deterioro de cuentas de activos inmovilizados - Recuperación de deterioro de intangibles"
"chart7574","7574","Recovery of impairment of fixed asset accounts - Recovery of impairment of biological assets","income_other","False","Recuperación de deterioro de cuentas de activos inmovilizados - Recuperación de deterioro de activos biológicos"
"chart7591","7591","Other management income - Government subsidies","income_other","False","Otros ingresos de gestión - Subsidios gubernamentales"
"chart7592","7592","Other management income - Insurance claims","income_other","False","Otros ingresos de gestión - Reclamos al seguro"
"chart7593","7593","Other management income - Donations","income_other","False","Otros ingresos de gestión - Donaciones"
"chart7594","7594","Other management income - Tax refunds","income_other","False","Otros ingresos de gestión - Devoluciones tributarias"
"chart7599","7599","Other management income - Other management income","income_other","False","Otros ingresos de gestión - Otros ingresos de gestión"
"chart7611","7611","Realizable asset - Merchandise","income_other","False","Activo realizable - Mercaderías"
"chart7612","7612","Realizable asset - Finished products","income_other","False","Activo realizable - Productos terminados"
"chart76131","76131","Realizable asset - Non-current assets held for sale - Investment property","income_other","False","Recuperación de cuentas de valuación - Recuperación – Desvalorización de inversiones mobiliarias"
"chart76132","76132","Realizable asset - Non-current assets held for sale - Property, plant and equipment","income_other","False","Activo realizable - Activos no corrientes mantenidos para la venta - Propiedades, planta y equipo"
"chart76133","76133","Realizable asset - Non-current assets held for sale - Intangibles","income_other","False","Activo realizable - Activos no corrientes mantenidos para la venta - Intangibles"
"chart76134","76134","Realizable asset - Non-current assets held for sale - Biological assets","income_other","False","Activo realizable - Non-current assets held for sale - Activos biológicos"
"chart7621","7621","Fixed asset - Investment property","income_other","False","Activo inmovilizado - Propiedades de inversión"
"chart7622","7622","Fixed asset - Biological assets","income_other","False","Activo inmovilizado - Activos biológicos"
"chart771","771","Gain on derivative financial instrument","income_other","False","Ganancia por instrumento financiero derivado"
"chart7721","7721","Earned returns - Deposits in financial institutions","income_other","False","Rendimientos ganados - Depósitos en instituciones financieras"
"chart7722","7722","Earned returns - Trade accounts receivable","income_other","False","Rendimientos ganados - Cuentas por cobrar comerciales"
"chart7723","7723","Earned returns - Loans granted","income_other","False","Rendimientos ganados - Préstamos otorgados"
"chart7724","7724","Earned returns - Investments to be held until maturity","income_other","False","Rendimientos ganados - Inversiones a ser mantenidas hasta el vencimiento"
"chart7725","7725","Earned returns - Financial instruments representative of economic rights","income_other","False","Rendimientos ganados - Instrumentos financieros representativos de derecho patrimonial"
"chart773","773","Dividends","income_other","False","Dividendos"
"chart774","774","Income from factoring operations","income_other","False","Ingresos en operaciones de factoraje (factoring)"
"chart775","775","Discounts obtained for prompt payment","income_other","False","Descuentos obtenidos por pronto pago"
"chart776","776","Difference in change","income_other","False","Diferencia en cambio"
"chart7771","7771","Gain from measuring financial assets and liabilities at fair value - Investments held for trading","income_other","False","Ganancia por medición de activos y pasivos financieros al valor razonable - Inversiones mantenidas para negociación"
"chart7772","7772","Gain from measuring financial assets and liabilities at fair value - Other investments","income_other","False","Ganancia por medición de activos y pasivos financieros al valor razonable - Otras inversiones"
"chart7773","7773","Gain from measuring financial assets and liabilities at fair value - Others","income_other","False","Ganancia por medición de activos y pasivos financieros al valor razonable - Others"
"chart7781","7781","Participation in results of related entities - Participation in the results of subsidiaries and associates under the equity value method","income_other","False","Participación en resultados de entidades relacionadas - Participación en los resultados de subsidiarias y asociadas bajo el método del valor patrimonial"
"chart7782","7782","Participation in results of related entities - Income from participation in joint ventures","income_other","False","Participación en resultados de entidades relacionadas - Participaciones en negocios conjuntos"
"chart7792","7792","Other financial income - Financial income measured at discounted value","income_other","False","Otros ingresos financieros - Ingresos financieros en medición a valor descontado"
"chart781","781","Charges covered by provisions","income_other","False","Cargas cubiertas por provisiones"
"chart791","791","Charges attributable to cost and expense accounts","income","False","Cargas imputables a cuentas de costos y gastos"
"chart792","792","Financial expenses attributable to inventory accounts","income","False","Gastos financieros imputables a cuentas de inventarios"
"chart801","801","Margen comercial","equity","False","Margen comercial"
"chart811","811","Production of goods","equity","False","Producción de bienes"
"chart812","812","Production of services","equity","False","Producción de servicios"
"chart813","813","Production of fixed assets","equity","False","Producción de activo inmovilizado"
"chart821","821","Value added","equity","False","Valor agregado"
"chart831","831","Gross surplus (gross deficiency) of exploitation","equity","False","Excedente bruto (insuficiencia bruta) de explotación"
"chart841","841","Result of exploitation","equity","False","Resultado de explotación"
"chart851","851","Income before income tax","equity","False","Resultado antes del impuesto a las ganancias"
"chart881","881","Income tax – Current","expense","False","Impuesto a las ganancias – Corriente"
"chart882","882","Income tax – Deferred","liability_non_current","False","Impuesto a las ganancias – Diferido"
"chart891","891","Utility","equity","False","Utilidad"
"chart892","892","Lost","equity","False","Pérdida"

```

## File: data\template\account.fiscal.position-pe.csv

```csv
"id","name","auto_apply","country_id","sequence","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@es"
"local_peru","LOCAL PERU","1","base.pe","15","","","LOCAL PERÚ"
"exportation","FOREIGN - EXPORT","1","","10","sale_tax_igv_18","sale_tax_exp","EXTRANJERO - EXPORTACIÓN"
"","","","","","sale_tax_igv_18_included","sale_tax_exp",""

```

## File: data\template\account.group-pe.csv

```csv
"id","code_prefix_start","name","name@es"
"group0","0","Memorandum accounts","Cuentas de orden"
"group1","1","Assets available and payable","Activo disponible y exigible"
"group2","2","Realizable asset","Activo realizable"
"group3","3","Assets available and payable","Activo disponible y exigible"
"group4","4","Liabilities","Pasivo"
"group5","5","Net Worth","Patrimonio Neto"
"group6","6","Expenses by nature","Gastos por naturaleza"
"group7","7","Incomes","Ingresos"
"group8","8","Intermediate management balances and determination of profit for the year","Saldos intrermediarios de gestión y determinación del resultado del ejercicio"
"group10","10","CASH AND CASH EQUIVALENTS","EFECTIVO Y EQUIVALENTES DE EFECTIVO"
"group11","11","FINANCIAL INVESTMENTS","INVERSIONES FINANCIERAS"
"group12","12","TRADE ACCOUNTS RECEIVABLE – THIRD PARTIES","CUENTAS POR COBRAR COMERCIALES – TERCEROS"
"group13","13","TRADE ACCOUNTS RECEIVABLE – ASSOCIATED","CUENTAS POR COBRAR COMERCIALES – RELACIONADAS"
"group14","14","ACCOUNTS RECEIVABLE FROM STAFF, SHAREHOLDERS (PARTNERS) AND DIRECTORS","CUENTAS POR COBRAR AL PERSONAL, A LOS ACCIONISTAS (SOCIOS) y DIRECTORES"
"group16","16","MISCELLANEOUS ACCOUNTS RECEIVABLE – THIRD PARTIES","CUENTAS POR COBRAR DIVERSAS – TERCEROS"
"group17","17","MISCELLANEOUS ACCOUNTS RECEIVABLE – ASSOCIATED","CUENTAS POR COBRAR DIVERSAS – RELACIONADAS"
"group18","18","SERVICES AND OTHER CONTRACTED IN ADVANCE","SERVICIOS Y OTROS CONTRATADOS POR ANTICIPADO"
"group19","19","ESTIMATION OF DOUBTFUL ACCOUNTS","ESTIMACIÓN DE CUENTAS DE COBRANZA DUDOSA"
"group20","20","MERCHANDISE","MERCADERÍAS"
"group21","21","FINISHED PRODUCTS","PRODUCTOS TERMINADOS"
"group22","22","BY-PRODUCTS, WASTE AND WASTE","SUBPRODUCTOS, DESECHOS Y DESPERDICIOS"
"group23","23","PRODUCTS IN PROCESS","PRODUCTOS EN PROCESO"
"group24","24","RAW MATERIALS","MATERIAS PRIMAS"
"group25","25","AUXILIARY MATERIALS, SUPPLIES AND SPARE PARTS","MATERIALES AUXILIARES, SUMINISTROS Y REPUESTOS"
"group26","26","CONTAINERS AND PACKAGING","ENVASES Y EMBALAJES"
"group27","27","NON-CURRENT ASSETS HELD FOR SALE","ACTIVOS NO CORRIENTES MANTENIDOS PARA LA VENTA"
"group28","28","INVENTORIES TO RECEIVE","INVENTARIOS POR RECIBIR"
"group29","29","DEVALUATION OF INVENTORIES","DESVALORIZACIÓN DE INVENTARIOS"
"group30","30","PROPERTY INVESTMENTS","INVERSIONES MOBILIARIAS"
"group31","31","INVESTMENT PROPERTIES","PROPIEDADES DE INVERSIÓN"
"group32","32","ASSETS BY RIGHT OF USE","ACTIVOS POR DERECHO DE USO"
"group33","33","PROPERTY, PLANT AND EQUIPMENT","PROPIEDAD, PLANTA Y EQUIPO"
"group34","34","INTANGIBLES","INTANGIBLES"
"group35","35","BIOLOGICAL ASSETS","ACTIVOS BIOLÓGICOS"
"group36","36","DEVALUATION OF FIXED ASSETS","DESVALORIZACIÓN DE ACTIVO INMOVILIZADO"
"group37","37","DEFERRED ASSETS","ACTIVO DIFERIDO"
"group38","38","OTHER ASSETS","OTROS ACTIVOS"
"group39","39","ACCUMULATED DEPRECIATION AND AMORTIZATION","DEPRECIACIÓN y AMORTIZACIÓN ACUMULADOS"
"group40","40","TAXES, CONSIDERATIONS AND CONTRIBUTIONS TO THE PUBLIC PENSION AND HEALTH SYSTEM PAYABLE","TRIBUTOS, CONTRAPRESTACIONES Y APORTES AL SISTEMA PÚBLICO DE PENSIONES Y DE SALUD POR PAGAR"
"group41","41","REMUNERATIONS AND PARTICIPATIONS PAYABLE","REMUNERACIONES Y PARTICIPACIONES POR PAGAR"
"group42","42","THIRD-PARTY TRADE ACCOUNTS PAYABLE","CUENTAS POR PAGAR COMERCIALES TERCEROS"
"group43","43","RELATED TRADE ACCOUNTS PAYABLE","CUENTAS POR PAGAR COMERCIALES RELACIONADAS"
"group44","44","ACCOUNTS PAYABLE TO SHAREHOLDERS (PARTNERS, PARTICIPANTS) AND DIRECTORS","CUENTAS POR PAGAR A LOS ACCIONISTAS (SOCIOS, PARTÍCIPES) Y DIRECTORES"
"group45","45","FINANCIAL OBLIGATIONS","OBLIGACIONES FINANCIERAS"
"group46","46","OTHER ACCOUNTS PAYABLE – THIRD PARTIES","CUENTAS POR PAGAR DIVERSAS – TERCEROS"
"group47","47","OTHER ACCOUNTS PAYABLE – ASSOCIATED","CUENTAS POR PAGAR DIVERSAS – RELACIONADAS"
"group48","48","PROVISIONS","PROVISIONES"
"group49","49","DEFERRED LIABILITIES","PASIVO DIFERIDO"
"group50","50","CAPITAL","CAPITAL"
"group51","51","INVESTMENT STOCKS","ACCIONES DE INVERSIÓN"
"group52","52","ADDITIONAL CAPITAL","CAPITAL ADICIONAL"
"group56","56","UNREALIZED RESULTS","RESULTADOS NO REALIZADOS"
"group57","57","REVALUATION SURPLUS","EXCEDENTE DE REVALUACIÓN"
"group58","58","RESERVES","RESERVAS"
"group59","59","ACUMULATED RESULTS","RESULTADOS ACUMULADOS"
"group60","60","PURCHASES","COMPRAS"
"group61","61","VARIATION IN INVENTORIES","VARIACIÓN DE INVENTARIOS"
"group62","62","STAFF AND DIRECTORS EXPENSES","GASTOS DE PERSONAL Y DIRECTORES"
"group63","63","EXPENSES FOR SERVICES RENDERED BY THIRD PARTIES","GASTOS DE SERVICIOS PRESTADOS POR TERCEROS"
"group64","64","TAX EXPENSES","GASTOS POR TRIBUTOS"
"group65","65","OTHER MANAGEMENT EXPENSES","OTROS GASTOS DE GESTION"
"group66","66","LOSS DUE TO MEASUREMENT OF NON-FINANCIAL ASSETS AT FAIR VALUE","PERDIDA POR MEDICIÓN DE ACTIVOS NO FINANCIEROS AL VALOR RAZONABLE"
"group67","67","FINANCIAL EXPENSES","GASTOS FINANCIEROS"
"group68","68","VALUATION AND IMPAIRMENT OF ASSETS AND PROVISIONS","VALUACIÓN Y DETERIORO DE ACTIVOS Y PROVISIONES"
"group69","69","SALES COST","COSTO DE VENTAS"
"group70","70","SALES","VENTAS"
"group71","71","VARIATION IN STORED PRODUCTION","VARIACIÓN DE LA PRODUCCIÓN ALMACENADA"
"group72","72","PRODUCTION OF FIXED ASSETS","PRODUCCIÓN DE ACTIVO INMOVILIZADO"
"group73","73","DISCOUNTS, REBATES AND BONUSES OBTAINED","DESCUENTOS, REBAJAS Y BONIFICACIONES OBTENIDOS"
"group74","74","DISCOUNTS, DISCOUNTS AND BONUSES GRANTED","DESCUENTOS, REBAJAS y BONIFICACIONES CONCEDIDOS"
"group75","75","OTHER MANAGEMENT INCOME","OTROS INGRESOS DE GESTIÓN"
"group76","76","PROFIT FROM MEASUREMENT OF NON-FINANCIAL ASSETS AT FAIR VALUE","GANANCIA POR MEDICIÓN DE ACTIVOS NO FINANCIEROS AL VALOR RAZONABLE"
"group77","77","FINANCIAL INCOME","INGRESOS FINANCIEROS"
"group78","78","CHARGES COVERED BY PROVISIONS","CARGAS CUBIERTAS POR PROVISIONES"
"group79","79","CHARGES ATTRIBUTABLE TO COST AND EXPENSE ACCOUNTS","CARGAS IMPUTABLES A CUENTAS DE COSTOS Y GASTOS"
"group80","80","COMMERCIAL MARGIN","MARGEN COMERCIAL"
"group81","81","PRODUCTION OF THE YEAR","PRODUCCIÓN DEL EJERCICIO"
"group82","82","VALUE ADDED","VALOR AGREGADO"
"group83","83","GROSS SURPLUS (GROSS LACK) FROM OPERATING","EXCEDENTE BRUTO (INSUFICIENCIA BRUTA) DE EXPLOTACIÓN"
"group84","84","RESULT OF EXPLOITATION","RESULTADO DE EXPLOTACIÓN"
"group85","85","RESULT BEFORE PARTICIPATIONS AND TAXES","RESULTADO ANTES DE PARTICIPACIONES E IMPUESTOS"
"group88","88","INCOME TAX","IMPUESTO A LA RENTA"
"group89","89","DETERMINATION OF THE RESULT OF THE YEAR","DETERMINACIÓN DEL RESULTADO DEL EJERCICIO"

```

## File: data\template\account.tax-pe.csv

```csv
"id","name","description","invoice_label","l10n_pe_edi_tax_code","l10n_pe_edi_unece_category","amount","amount_type","children_tax_ids","type_tax_use","sequence","include_base_amount","tax_group_id","price_include","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@es"
"sale_tax_igv_18","18%","","IGV 18%","1000","S","18.0","percent","","sale","1","1","tax_group_igv","","base","invoice","","","18%"
"","","","","","","","","","","","","","","tax","invoice","chart40111","",""
"","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","tax","refund","chart40111","",""
"sale_tax_igv_18_included","18% TTC","","IGV 18%","1000","S","18.0","percent","","sale","1","1","tax_group_igv","True","base","invoice","","","18% (Incluido en precio)"
"","","","","","","","","","","","","","","tax","invoice","chart40111","",""
"","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","tax","refund","chart40111","",""
"sale_tax_exo","0% Exo","","EXO 0%","9997","E","0.0","percent","","sale","1","","tax_group_exo","","base","invoice","","","0% Exonerado"
"","","","","","","","","","","","","","","tax","invoice","chart40111","",""
"","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","tax","refund","chart40111","",""
"sale_tax_ina","0% Una","","INA 0%","9998","Z","0.0","percent","","sale","1","","tax_group_ina","","base","invoice","","","0% Inafectado"
"","","","","","","","","","","","","","","tax","invoice","chart40111","",""
"","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","tax","refund","chart40111","",""
"sale_tax_gra","0%","","GRA 0%","9996","E","0.0","percent","","sale","1","","tax_group_gra","","base","invoice","","","0% Gratis"
"","","","","","","","","","","","","","","tax","invoice","chart40111","",""
"","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","tax","refund","chart40111","",""
"sale_tax_ics_0","0% ISC","","ISC 0%","2000","S","0.0","percent","","sale","1","1","tax_group_isc","","base","invoice","","100",""
"","","","","","","","","","","","","","","tax","invoice","chart4012","100",""
"","","","","","","","","","","","","","","base","refund","","100",""
"","","","","","","","","","","","","","","tax","refund","chart4012","100",""
"purchase_tax_igv_18","18%","","IGV 18%","1000","S","18.0","percent","","purchase","1","1","tax_group_igv","","base","invoice","","","18%"
"","","","","","","","","","","","","","","tax","invoice","chart40111","",""
"","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","tax","refund","chart40111","",""
"purchase_tax_igv_18_included","18% TTC","","IGV 18%","1000","S","18.0","percent","","purchase","1","1","tax_group_igv","True","base","invoice","","","18% (Incluido en precio)"
"","","","","","","","","","","","","","","tax","invoice","chart40111","",""
"","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","tax","refund","chart40111","",""
"purchase_tax_igv_18g_ng","18% Levied and Not Taxed","","IGV 18%","1000","S","18.0","percent","","purchase","1","1","tax_group_igv_g_ng","True","base","invoice","","","18% Gravadas y No Gravada"
"","","","","","","","","","","","","","","tax","invoice","chart40117","",""
"","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","tax","refund","chart40117","",""
"purchase_tax_igv_18_ng","18% Not Levied","","IGV NG 18%","1000","S","18.0","percent","","purchase","1","1","tax_group_igv_ng","True","base","invoice","","","18% No Gravadas"
"","","","","","","","","","","","","","","tax","invoice","chart40116","",""
"","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","tax","refund","chart40116","",""
"purchase_tax_exp_0","0% EXP","","EXP 0%","9995","S","0.0","percent","","purchase","1","1","tax_group_exp","True","base","invoice","","","0% EXP"
"","","","","","","","","","","","","","","tax","invoice","chart40115","",""
"","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","tax","refund","chart40115","",""
"purchase_tax_exo","0% Exo","","EXO 0%","9997","E","0.0","percent","","purchase","1","","tax_group_exo","","base","invoice","","","0% Exonerado"
"","","","","","","","","","","","","","","tax","invoice","chart40111","",""
"","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","tax","refund","chart40111","",""
"purchase_tax_ina","0% Una","","INA 0%","9998","Z","0.0","percent","","purchase","1","","tax_group_ina","","base","invoice","","","0% Inafectado"
"","","","","","","","","","","","","","","tax","invoice","chart40111","",""
"","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","tax","refund","chart40111","",""
"purchase_tax_gra","0%","","GRA 0%","9996","E","0.0","percent","","purchase","1","","tax_group_gra","","base","invoice","","","0% Gratis"
"","","","","","","","","","","","","","","tax","invoice","chart40111","",""
"","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","tax","refund","chart40111","",""
"sale_tax_exp","0% EXP","","EXP 0%","9995","S","0.0","percent","","sale","1","1","tax_group_exp","","base","invoice","","0","0% EXP"
"","","","","","","","","","","","","","","tax","invoice","chart40111","0",""
"","","","","","","","","","","","","","","base","refund","","0",""
"","","","","","","","","","","","","","","tax","refund","chart40111","0",""
"tax_free_subtract_base","Subtract Base","","","","","-100","percent","","none","1","","tax_group_free_invoice","","base","invoice","","","Substraer base"
"","","","","","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","tax","refund","","",""
"tax_free_igv_18","18% Free","","","9996","E","18","percent","","none","1","","tax_group_gra","","base","invoice","","","18% Libre de Impuestos"
"","","","","","","","","","","","","","","tax","invoice","chart40111","",""
"","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","tax","refund","chart40111","",""
"tax_free_igv_18_expense","-18% Free Expense Tax","","","","","-18","percent","","none","1","","tax_group_free_invoice","","base","invoice","","","-18% Libre de Impuestos Gasto"
"","","","","","","","","","","","","","","tax","invoice","chart6411","",""
"","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","tax","refund","chart6411","",""
"tax_free_group","18% Free Final","","","9996","","","group","tax_free_igv_18,tax_free_subtract_base,tax_free_igv_18_expense","sale","1","","tax_group_igv","","","","","","18% Libre Final"

```

## File: data\template\account.tax.group-pe.csv

```csv
"id","name","sequence","country_id","name@es"
"tax_group_igv","IGV","0","base.pe","IGV"
"tax_group_igv_g_ng","IGV GyNG","0","base.pe","IGV GyNG"
"tax_group_igv_ng","IGV NG","0","base.pe","IGV NG"
"tax_group_ivap","IVAP","0","base.pe","IVAP"
"tax_group_isc","ISC","0","base.pe","ISC"
"tax_group_exp","EXP","0","base.pe","EXP"
"tax_group_gra","GRA","0","base.pe","GRA"
"tax_group_exo","EXO","0","base.pe","EXO"
"tax_group_ina","INA","0","base.pe","INA"
"tax_group_other","OTHERS","0","base.pe","OTROS"
"tax_group_det","DET","100","base.pe","DET"
"tax_group_icbper","ICBPER","0","base.pe",""
"tax_group_ret","RET","100","base.pe","RET"
"tax_group_free_invoice","Free Invoice","200","base.pe","Factura Gratuita"

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
        if self.company_id.country_id.code != "PE" or not self.journal_id.l10n_latam_use_documents or self.journal_id.type != "sale":
            return result
        result.append(("code", "in", ("01", "03", "07", "08", "20", "40")))
        if self.partner_id.l10n_latam_identification_type_id.l10n_pe_vat_code != '6' and self.move_type == 'out_invoice':
            result.append(('id', 'in', (
                self.env.ref('l10n_pe.document_type08b')
                | self.env.ref('l10n_pe.document_type02')
                | self.env.ref('l10n_pe.document_type07b')
            ).ids))
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

    l10n_pe_edi_tax_code = fields.Selection(
        [
            ('1000', 'IGV - General Sales Tax'),
            ('1016', 'IVAP - Tax on Sale Paddy Rice'),
            ('2000', 'ISC - Selective Excise Tax'),
            ('7152', 'ICBPER - Plastic bag tax'),
            ('9995', 'EXP - Exportation'),
            ('9996', 'GRA - Free'),
            ('9997', 'EXO - Exonerated'),
            ('9998', 'INA - Unaffected'),
            ('9999', 'OTHERS - Other taxes')
        ],
        string='Code',
        help="Peru: SUNAT tax code",
    )

    l10n_pe_edi_unece_category = fields.Selection(
        [
            ('E', 'Exempt from tax'),
            ('G', 'Free export item, tax not charged'),
            ('O', 'Services outside scope of tax'),
            ('S', 'Standard rate'),
            ('Z', 'Zero rated goods')
        ],
        string='UNECE Code',
        help="Peru: Follow the UN/ECE 5305 standard from the United Nations Economic Commission for Europe for more "
             "information http://www.unece.org/trade/untdid/d08a/tred/tred5305.html"
    )
    l10n_pe_edi_isc_type = fields.Selection([
        ('01', 'System to value'),
        ('02', 'Application of the Fixed Amount'),
        ('03', 'Retail Price System'),
    ], 'ISC Type',
        help='Used in Selective Consumption Tax to indicate the type of calculation for the ISC.')

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

## File: models\template_pe.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('pe')
    def _get_pe_template_data(self):
        return {
            'property_account_receivable_id': 'chart1213',
            'property_account_payable_id': 'chart4212',
            'property_account_expense_categ_id': 'chart6329',
            'property_account_income_categ_id': 'chart70121',
            'property_stock_account_input_categ_id': 'chart6111',
            'property_stock_account_output_categ_id': 'chart69111',
            'property_stock_valuation_account_id': 'chart20111',
            'code_digits': '7',
        }

    @template('pe', 'res.company')
    def _get_pe_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.pe',
                'bank_account_code_prefix': '1041',
                'cash_account_code_prefix': '1031',
                'transfer_account_code_prefix': '1051',
                'account_default_pos_receivable_account_id': 'chart1215',
                'income_currency_exchange_account_id': 'chart776',
                'expense_currency_exchange_account_id': 'chart676',
                'account_journal_early_pay_discount_loss_account_id': 'chart675',
                'account_journal_early_pay_discount_gain_account_id': 'chart775',
                'account_sale_tax_id': 'sale_tax_igv_18',
                'account_purchase_tax_id': 'purchase_tax_igv_18',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_pe
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
"access_l10n_pe_district_group_all","access_l10n_pe_district group_all","model_l10n_pe_res_city_district",,0,0,0,0

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
                       invisible="country_code != 'PE'"/>
                <field name="l10n_pe_edi_unece_category"
                       invisible="country_code != 'PE'"/>
                <field name="l10n_pe_edi_isc_type"
                       invisible="l10n_pe_edi_tax_code != '2000' or country_code != 'PE'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

