# Odoo Module: l10n_es_edi_tbai

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizards

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Thanks to Landoo and the Spanish community
# Specially among others Aritz Olea, Luis Salvatierra, Josean Soroa

{
    'name': "Spain - TicketBAI",
    'version': '1.1',
    'category': 'Accounting/Localizations/EDI',
    'description': """
This module sends invoices and vendor bills to the "Diputaciones
Forales" of Araba/Álava, Bizkaia and Gipuzkoa.

Invoices and bills get converted to XML and regularly sent to the
Basque government servers which provides them with a unique identifier.
A hash chain ensures the continuous nature of the invoice/bill
sequences. QR codes are added to emitted (sent/printed) invoices,
bills and tickets to allow anyone to check they have been declared.

You need to configure your certificate and the tax agency.
    """,
    'depends': [
        'l10n_es',
        'certificate',
    ],
    'data': [
        'data/template_invoice.xml',
        'data/template_LROE_bizkaia.xml',
        'data/ir_config_parameter.xml',

        'security/ir.model.access.csv',
        'security/l10n_es_edi_tbai_security.xml',

        'views/account_move_view.xml',
        'views/l10n_es_edi_tbai_certificate_views.xml',
        'views/report_invoice.xml',
        'views/res_config_settings_views.xml',
        'views/res_company_views.xml',

        'wizards/account_move_reversal_views.xml',
    ],
    'demo': [
        'demo/demo_certificate.xml',
        'demo/demo_res_partner.xml',
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\ir_config_parameter.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="epigrafe_config" model="ir.config_parameter">
            <field name="key">l10n_es_edi_tbai.epigrafe</field>
            <field name="value" eval=""/>
        </record>
    </data>
</odoo>

```

## File: data\neutralize.sql

```sql
-- disable_l10n_es_edi_integration
UPDATE res_company
   SET l10n_es_tbai_test_env = true;

```

## File: data\template_invoice.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <data>
        <template id="template_invoice_main_post">
            <T:TicketBai xmlns:T="urn:ticketbai:emision">
                <t t-call="l10n_es_edi_tbai.template_invoice_bundle"/>
            </T:TicketBai>
        </template>

        <template id="template_invoice_main_cancel">
            <T:AnulaTicketBai xmlns:T="urn:ticketbai:anulacion">
                <t t-call="l10n_es_edi_tbai.template_invoice_bundle"/>
            </T:AnulaTicketBai>
        </template>

        <template id="template_invoice_bundle">
            <Cabecera>
                <IDVersionTBAI t-out="tbai_version"/>
            </Cabecera>
            <t t-if="is_emission">
                <Sujetos>
                    <t t-call="l10n_es_edi_tbai.template_invoice_sujetos"/>
                </Sujetos>
                <Factura>
                    <t t-call="l10n_es_edi_tbai.template_invoice_factura"/>
                </Factura>
            </t>
            <t t-else="">
                <IDFactura>
                    <t t-call="l10n_es_edi_tbai.template_invoice_sujetos"/>
                    <t t-call="l10n_es_edi_tbai.template_invoice_factura"/>
                </IDFactura>
            </t>
            <HuellaTBAI>
                <EncadenamientoFacturaAnterior t-if="chain_prev_document">
                    <t t-set="seq_and_num" t-value="chain_prev_document._get_tbai_sequence_and_number()"/>
                    <SerieFacturaAnterior t-out="seq_and_num[0]"/>
                    <NumFacturaAnterior t-out="seq_and_num[1]"/>
                    <t t-set="sig_and_date" t-value="chain_prev_document._get_tbai_signature_and_date()"/>
                    <FechaExpedicionFacturaAnterior t-out="format_date(sig_and_date[1])"/>
                    <SignatureValueFirmaFacturaAnterior t-out="sig_and_date[0][:100]"/>
                </EncadenamientoFacturaAnterior>
                <Software>
                    <LicenciaTBAI t-out="license_number"/>
                    <EntidadDesarrolladora>
                        <NIF t-out="license_nif"/>
                    </EntidadDesarrolladora>
                    <Nombre t-out="software_name"/>
                    <Version t-out="software_version"/>
                </Software>
                <NumSerieDispositivo>TEST-DEVICE-001</NumSerieDispositivo>
            </HuellaTBAI>
        </template>

        <template id="template_invoice_sujetos">
            <Emisor>
                <NIF t-out="sender_vat"/>
                <ApellidosNombreRazonSocial t-out="sender.name"/>
            </Emisor>
            <Destinatarios t-if="is_emission and recipient">
                <IDDestinatario>
                    <NIF t-if="recipient.get('nif')" t-out="recipient['nif']"/>
                    <IDOtro t-else="">
                        <CodigoPais t-if="recipient.get('alt_id_country')" t-out="recipient['alt_id_country']"/>
                        <IDType t-out="recipient['alt_id_type']"/>
                        <ID t-out="recipient['alt_id_number']"/>
                    </IDOtro>
                    <t t-set="partner" t-value="recipient['partner']"/>
                    <ApellidosNombreRazonSocial t-out="partner.name"/>
                    <CodigoPostal t-out="partner.zip"/>
                    <Direccion t-out="recipient['partner_address']"/>
                </IDDestinatario>
            </Destinatarios>
            <VariosDestinatarios t-if="is_emission">N</VariosDestinatarios> <!-- Odoo does not support multi-recipient invoices (TBAI does)-->
            <EmitidaPorTercerosODestinatario t-if="is_emission">N</EmitidaPorTercerosODestinatario>
        </template>

        <template id="template_invoice_factura">
            <CabeceraFactura>
                <t t-set="seq_and_num" t-value="doc._get_tbai_sequence_and_number()"/>
                <SerieFactura t-out="seq_and_num[0]"/>
                <NumFactura t-out="seq_and_num[1]"/>
                <t t-if="is_emission">
                    <FechaExpedicionFactura t-out="format_date(datetime_now)"/>
                    <HoraExpedicionFactura t-out="format_time(datetime_now)"/>
                    <FacturaSimplificada t-out="'S' if is_simplified else 'N'"/>
                </t>
                <FechaExpedicionFactura t-else="" t-out="format_date(post_doc._get_tbai_signature_and_date()[1])"/>
                <t t-if="is_refund and is_emission">
                    <FacturaEmitidaSustitucionSimplificada t-out="'S' if (is_simplified and recipient) else 'N'"/>
                    <FacturaRectificativa>
                        <Codigo t-out="refund_reason"/>
                        <Tipo>I</Tipo>
                        <!-- NOTE: could also allow credit note Tipo 'S' (optional, tipo I already supported by SII)
                        <ImporteRectificacionSustitutiva>
                            <BaseRectificada>180.00</BaseRectificada>
                            <CuotaRectificada>20.21</CuotaRectificada>
                        </ImporteRectificacionSustitutiva> -->
                    </FacturaRectificativa>
                    <FacturasRectificadasSustituidas>
                        <IDFacturaRectificadaSustituida>
                            <!-- NOTE: could support issuing a single credit note for multiple invoices (optional) -->
                            <t t-set="seq_and_num" t-value="refunded_doc._get_tbai_sequence_and_number()"/>
                            <SerieFactura t-out="seq_and_num[0]"/>
                            <NumFactura t-out="seq_and_num[1]"/>
                            <FechaExpedicionFactura t-out="format_date(refunded_doc.xml_attachment_id and refunded_doc._get_tbai_signature_and_date()[1] or refunded_doc_invoice_date)"/>
                        </IDFacturaRectificadaSustituida>
                    </FacturasRectificadasSustituidas>
                </t>
            </CabeceraFactura>
            <DatosFactura t-if="is_emission">
                <FechaOperacion t-if="delivery_date" t-out="format_date(delivery_date)"/>
                <DescripcionFactura t-out="origin"/>
                <DetallesFactura>
                    <IDDetalleFactura t-foreach="base_lines" t-as="base_line">
                        <DescripcionDetalle t-out="base_line['description']"/>
                        <Cantidad t-out="format_float(base_line['quantity'], precision_digits=8)"/>
                        <ImporteUnitario t-out="format_float(base_line['gross_price_unit'], precision_digits=8)"/>
                        <Descuento t-out="format_float(base_line['discount_amount'], precision_digits=8)"/>
                        <ImporteTotal t-out="format_float(base_line['price_total'], precision_digits=8)"/>
                    </IDDetalleFactura>
                </DetallesFactura>
                <ImporteTotalFactura t-out="format_float(total_amount)"/>
                <RetencionSoportada t-if="total_retention" t-out="format_float(-total_retention)"/>
                <!-- <BaseImponibleACoste/> NOTE (only applicable with ClaveRegimenIvaOpTrascendencia 06, not supported yet) -->
                <Claves>
                    <IDClave t-foreach="regime_key" t-as="key">
                        <ClaveRegimenIvaOpTrascendencia t-out="key"/>
                    </IDClave>
                </Claves>
            </DatosFactura>
            <TipoDesglose t-if="is_emission">
                <DesgloseFactura t-if="'DesgloseFactura' in invoice_info">
                    <t t-call="l10n_es_edi_tbai.template_invoice_desglose">
                        <t t-set="desglose" t-value="invoice_info['DesgloseFactura']"/>
                    </t>
                </DesgloseFactura>
                <DesgloseTipoOperacion t-else="">
                    <t t-set="invoice_info" t-value="invoice_info['DesgloseTipoOperacion']"/>
                    <PrestacionServicios t-if="invoice_info.get('PrestacionServicios')">
                        <t t-call="l10n_es_edi_tbai.template_invoice_desglose">
                            <t t-set="desglose" t-value="invoice_info['PrestacionServicios']"/>
                        </t>
                    </PrestacionServicios>
                    <Entrega t-if="invoice_info.get('Entrega')">
                        <t t-call="l10n_es_edi_tbai.template_invoice_desglose">
                            <t t-set="desglose" t-value="invoice_info['Entrega']"/>
                        </t>
                    </Entrega>
                </DesgloseTipoOperacion>
            </TipoDesglose>
        </template>

        <template id="template_invoice_desglose">
            <Sujeta t-if="desglose.get('Sujeta')">
                <t t-set="sujeta" t-value="desglose['Sujeta']"/>
                <Exenta t-if="sujeta.get('Exenta')">
                    <DetalleExenta t-foreach="sujeta['Exenta']['DetalleExenta']" t-as="exenta">
                        <CausaExencion t-out="exenta['CausaExencion']"/>
                        <BaseImponible t-out="format_float(exenta['BaseImponible'])"/>
                    </DetalleExenta>
                </Exenta>
                <NoExenta t-if="sujeta.get('NoExenta')">
                    <DetalleNoExenta t-if="desglose['S1']">
                        <TipoNoExenta t-out="'S1'"/>
                        <DesgloseIVA>
                            <DetalleIVA t-foreach="desglose['S1']" t-as="detalle">
                                <BaseImponible t-out="format_float(detalle['BaseImponible'])"/>
                                <TipoImpositivo t-out="format_float(detalle['TipoImpositivo'])"/>
                                <CuotaImpuesto t-out="format_float(detalle['CuotaRepercutida'])"/>
                                <TipoRecargoEquivalencia t-if="detalle.get('TipoRecargoEquivalencia')" t-out="format_float(detalle['TipoRecargoEquivalencia'])"/>
                                <CuotaRecargoEquivalencia t-if="detalle.get('CuotaRecargoEquivalencia')" t-out="format_float(detalle['CuotaRecargoEquivalencia'])"/>
                                <OperacionEnRecargoDeEquivalenciaORegimenSimplificado t-out="'S' if is_simplified else 'N'"/>
                            </DetalleIVA>
                        </DesgloseIVA>
                    </DetalleNoExenta>
                    <DetalleNoExenta t-if="desglose['S2']">
                        <TipoNoExenta t-out="'S2'"/>
                        <DesgloseIVA>
                            <DetalleIVA t-foreach="desglose['S2']" t-as="detalle">
                                <BaseImponible t-out="format_float(detalle['BaseImponible'])"/>
                            </DetalleIVA>
                        </DesgloseIVA>
                    </DetalleNoExenta>
                </NoExenta>
            </Sujeta>
            <NoSujeta t-if="desglose.get('NoSujeta')">
                <t t-set="no_sujeta" t-value="desglose['NoSujeta']"/>
                <DetalleNoSujeta>
                    <Causa t-out="nosujeto_causa"/>
                    <!-- NOTE: Causa should be 
                        'OT' if 'the' ClaveRegimenIvaOpTrascendencia == 10
                        'RL' if 'some' ClaveRegimenIvaOpTrascendencia == 08
                    BUT those are not supported yet-->
                    <Importe t-out="no_sujeta.get('ImportePorArticulos7_14_Otros')"/>
                    <Importe t-out="no_sujeta.get('ImporteTAIReglasLocalizacion')"/>
                </DetalleNoSujeta>
            </NoSujeta>
        </template>

        <template id="template_digital_signature">
            <ds:Signature t-att-Id="dsig['signature_id']" xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
                <ds:SignedInfo>
                    <ds:CanonicalizationMethod Algorithm="http://www.w3.org/TR/2001/REC-xml-c14n-20010315"/>
                    <ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256"/>
                    <ds:Reference t-att-Id="dsig['reference_uri']" Type="http://www.w3.org/2000/09/xmldsig#Object" URI="">
                        <ds:Transforms>
                            <ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature"/>
                        </ds:Transforms>
                        <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
                        <ds:DigestValue></ds:DigestValue>
                    </ds:Reference>
                    <ds:Reference Type="http://uri.etsi.org/01903#SignedProperties" t-attf-URI="##{dsig['sigproperties_id']}">
                        <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
                        <ds:DigestValue></ds:DigestValue>
                    </ds:Reference>
                    <ds:Reference t-attf-URI="##{dsig['keyinfo_id']}">
                        <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
                        <ds:DigestValue></ds:DigestValue>
                    </ds:Reference>
                </ds:SignedInfo>
                <ds:SignatureValue></ds:SignatureValue>
                <ds:KeyInfo t-att-Id="dsig['keyinfo_id']">
                    <ds:X509Data>
                        <ds:X509Certificate t-out="dsig['x509_certificate']"/>
                    </ds:X509Data>
                    <ds:KeyValue>
                        <ds:RSAKeyValue>
                            <ds:Modulus t-out="dsig['public_modulus']"/>
                            <ds:Exponent t-out="dsig['public_exponent']"/>
                        </ds:RSAKeyValue>
                    </ds:KeyValue>
                </ds:KeyInfo>
                <ds:Object>
                    <xades:QualifyingProperties xmlns:xades="http://uri.etsi.org/01903/v1.3.2#" t-attf-Target="##{dsig['signature_id']}">
                        <xades:SignedProperties t-att-Id="dsig['sigproperties_id']">
                            <xades:SignedSignatureProperties>
                                <xades:SigningTime t-out="dsig['iso_now']"/>
                                <xades:SigningCertificateV2>
                                    <xades:Cert>
                                        <xades:CertDigest>
                                            <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
                                            <ds:DigestValue t-out="dsig['sigcertif_digest']"/>
                                        </xades:CertDigest>
                                        <xades:IssuerSerial>
                                            <ds:X509IssuerName t-out="dsig['x509_issuer_description']"/>
                                            <ds:X509SerialNumber t-out="dsig['x509_serial_number']"/>
                                        </xades:IssuerSerial>
                                    </xades:Cert>
                                </xades:SigningCertificateV2>
                                <xades:SignaturePolicyIdentifier>
                                    <xades:SignaturePolicyId>
                                        <xades:SigPolicyId>
                                            <xades:Identifier t-out="dsig['sigpolicy_url']"/>
                                        </xades:SigPolicyId>
                                        <xades:SigPolicyHash>
                                            <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
                                            <ds:DigestValue t-out="dsig['sigpolicy_digest']"/>
                                        </xades:SigPolicyHash>
                                    </xades:SignaturePolicyId>
                                </xades:SignaturePolicyIdentifier>
                            </xades:SignedSignatureProperties>
                        </xades:SignedProperties>
                    </xades:QualifyingProperties>
                </ds:Object>
            </ds:Signature>
        </template>
    </data>
</odoo>

```

## File: data\template_LROE_bizkaia.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<!--Bizkaia uses an extra layer to send TicketBAI invoices, called LROE 
    see https://www.batuz.eus/es/documentacion-tecnica -->
<odoo>
    <data>
        <template id="template_LROE_240_main">
            <lrpjfecsgap:LROEPJ240FacturasEmitidasConSGAltaPeticion
                xmlns:lrpjfecsgap="https://www.batuz.eus/fitxategiak/batuz/LROE/esquemas/LROE_PJ_240_1_1_FacturasEmitidas_ConSG_AltaPeticion_V1_0_2.xsd"
                t-if="is_emission and not freelancer"
                t-call="l10n_es_edi_tbai.template_LROE_240_inner"/>
            <lrpficfcsgap:LROEPF140IngresosConFacturaConSGAltaPeticion
                xmlns:lrpficfcsgap="https://www.batuz.eus/fitxategiak/batuz/LROE/esquemas/LROE_PF_140_1_1_Ingresos_ConfacturaConSG_AltaPeticion_V1_0_2.xsd"
                t-elif="is_emission and freelancer"
                t-call="l10n_es_edi_tbai.template_LROE_240_inner"/>
            <lrpjfecsgap:LROEPJ240FacturasEmitidasConSGAnulacionPeticion
                xmlns:lrpjfecsgap="https://www.batuz.eus/fitxategiak/batuz/LROE/esquemas/LROE_PJ_240_1_1_FacturasEmitidas_ConSG_AnulacionPeticion_V1_0_0.xsd"
                t-elif="not is_emission and not freelancer"
                t-call="l10n_es_edi_tbai.template_LROE_240_inner"/>
            <lrpficfcsgap:LROEPF140IngresosConFacturaConSGAnulacionPeticion
                xmlns:lrpficfcsgap="https://www.batuz.eus/fitxategiak/batuz/LROE/esquemas/LROE_PF_140_1_1_Ingresos_ConfacturaConSG_AnulacionPeticion_V1_0_0.xsd"
                t-elif="not is_emission and freelancer"
                t-call="l10n_es_edi_tbai.template_LROE_240_inner"/>
        </template>

        <template id="template_LROE_240_inner"> <!-- To be used for both 140 and 240 -->
            <Cabecera>
                <Modelo t-out="'140' if freelancer else '240'"/>
                <Capitulo>1</Capitulo>
                <Subcapitulo>1.1</Subcapitulo>
                <Operacion t-out="'A00' if is_emission else 'AN0'"/>
                <Version>1.0</Version>
                <Ejercicio t-out="fiscal_year"/>
                <ObligadoTributario>
                    <NIF t-out="sender_vat"/>
                    <ApellidosNombreRazonSocial t-out="sender.name"/>
                </ObligadoTributario>
            </Cabecera>
            <FacturasEmitidas t-if="not freelancer">
                <FacturaEmitida t-foreach="tbai_b64_list" t-as="tbai_b64">
                    <TicketBai t-if="is_emission" t-out="tbai_b64"/>
                    <AnulacionTicketBai t-else="" t-out="tbai_b64"/>
                </FacturaEmitida>
            </FacturasEmitidas>
            <Ingresos t-if="freelancer">
                <Ingreso t-foreach="tbai_b64_list" t-as="tbai_b64">
                    <TicketBai t-if="is_emission" t-out="tbai_b64"/>
                    <Renta><DetalleRenta><Epigrafe t-out="epigrafe"/></DetalleRenta></Renta>
                </Ingreso>
            </Ingresos>
        </template>


        <template id="template_LROE_240_main_recibidas">
            <lrpjframp:LROEPJ240FacturasRecibidasAltaModifPeticion
                xmlns:lrpjframp="https://www.batuz.eus/fitxategiak/batuz/LROE/esquemas/LROE_PJ_240_2_FacturasRecibidas_AltaModifPeticion_V1_0_1.xsd"
                t-if="is_emission and not freelancer"
                t-call="l10n_es_edi_tbai.template_LROE_240_inner_recibidas"/>
            <lrpfgcfamp:LROEPF140GastosConFacturaAltaModifPeticion
                xmlns:lrpfgcfamp="https://www.batuz.eus/fitxategiak/batuz/LROE/esquemas/LROE_PF_140_2_1_Gastos_Confactura_AltaModifPeticion_V1_0_2.xsd"
                t-elif="is_emission and freelancer"
                t-call="l10n_es_edi_tbai.template_LROE_240_inner_recibidas"/>
            <lrpjfrap:LROEPJ240FacturasRecibidasAnulacionPeticion
                xmlns:lrpjfrap="https://www.batuz.eus/fitxategiak/batuz/LROE/esquemas/LROE_PJ_240_2_FacturasRecibidas_AnulacionPeticion_V1_0_0.xsd"
                t-elif="not is_emission and not freelancer"
                t-call="l10n_es_edi_tbai.template_LROE_240_inner_recibidas"/>
            <lrpfgcfap:LROEPF140GastosConFacturaAnulacionPeticion
                xmlns:lrpfgcfap="https://www.batuz.eus/fitxategiak/batuz/LROE/esquemas/LROE_PF_140_2_1_Gastos_Confactura_AnulacionPeticion_V1_0_0.xsd"
                t-elif="not is_emission and freelancer"
                t-call="l10n_es_edi_tbai.template_LROE_240_inner_recibidas"/>
        </template>

        <template id="template_LROE_240_inner_recibidas">
            <Cabecera>
                <Modelo t-out="'140' if freelancer else '240'"/>
                <Capitulo>2</Capitulo>
                <Subcapitulo t-out="'2.1' if freelancer else None"/>
                <Operacion t-out="'A00' if is_emission else 'AN0'"/>
                <Version>1.0</Version>
                <Ejercicio t-out="fiscal_year"/>
                <ObligadoTributario>
                    <NIF t-out="sender_vat"/>
                    <ApellidosNombreRazonSocial t-out="sender.name"/>
                </ObligadoTributario>
            </Cabecera>
            <FacturasRecibidas t-if="not freelancer">
                <FacturaRecibida>
                    <t t-call="l10n_es_edi_tbai.template_LROE_recibidas_common"/>
                </FacturaRecibida>
            </FacturasRecibidas>
            <Gastos t-else="">
                <Gasto>
                    <t t-call="l10n_es_edi_tbai.template_LROE_recibidas_common"/>
                </Gasto>
            </Gastos>
        </template>

        <template id="template_LROE_recibidas_common">
            <t t-if="not is_emission"> <!-- cancel case -->
                        <IDRecibida t-if="not freelancer">
                            <t t-set="seq_and_num" t-value="doc._get_tbai_sequence_and_number()"/>
                            <SerieFactura t-out="seq_and_num[0]"/>
                            <NumFactura t-out="seq_and_num[1]"/>
                            <FechaExpedicionFactura t-out="format_date(invoice_date)"/>
                            <EmisorFacturaRecibida>
                                <NIF t-if="recipient.get('nif')" t-out="recipient['nif']"/>
                                <IDOtro t-else="">
                                    <CodigoPais t-if="recipient.get('alt_id_country')" t-out="recipient['alt_id_country']"/>
                                    <IDType t-out="recipient['alt_id_type']"/>
                                    <ID t-out="recipient['alt_id_number']"/>
                                </IDOtro>
                            </EmisorFacturaRecibida>
                        </IDRecibida>
                        <IDGasto t-else="">
                            <t t-set="seq_and_num" t-value="doc._get_tbai_sequence_and_number_purchase()"/>
                            <SerieFactura t-out="seq_and_num[0]"/>
                            <NumFactura t-out="seq_and_num[1]"/>
                            <FechaExpedicionFactura t-out="format_date(invoice_date)"/>
                            <EmisorFacturaRecibida>
                                <NIF t-if="recipient.get('nif')" t-out="recipient['nif']"/>
                                <IDOtro t-else="">
                                    <CodigoPais t-if="recipient.get('alt_id_country')" t-out="recipient['alt_id_country']"/>
                                    <IDType t-out="recipient['alt_id_type']"/>
                                    <ID t-out="recipient['alt_id_number']"/>
                                </IDOtro>
                            </EmisorFacturaRecibida>
                        </IDGasto>
                    </t>
                    <t t-else="">
                        <EmisorFacturaRecibida>
                            <NIF t-if="recipient.get('nif')" t-out="recipient['nif']"/>
                            <IDOtro t-else="">
                                <CodigoPais t-if="recipient.get('alt_id_country')" t-out="recipient['alt_id_country']"/>
                                <IDType t-out="recipient['alt_id_type']"/>
                                <ID t-out="recipient['alt_id_number']"/>
                            </IDOtro>
                            <t t-set="partner" t-value="recipient['partner']"/>
                            <ApellidosNombreRazonSocial t-out="partner.name"/>
                        </EmisorFacturaRecibida>
                        <CabeceraFactura>
                            <t t-set="seq_and_num" t-value="doc._get_tbai_sequence_and_number_purchase()"/>
                            <SerieFactura t-out="seq_and_num[0]"/>
                            <NumFactura t-out="seq_and_num[1]"/>
                            <FechaExpedicionFactura t-out="format_date(invoice_date)"/>
                            <FechaRecepcion t-out="format_date(doc.date)"/>
                            <TipoFactura t-out="tipofactura"/>
                            <t t-if="is_refund">
                                <FacturaRectificativa>
                                    <Codigo t-out="refund_reason"/>
                                    <Tipo>I</Tipo>
                                </FacturaRectificativa>
                                <FacturasRectificadasSustituidas t-if="credit_note_invoices">
                                    <IDFacturaRectificadaSustituida t-foreach="credit_note_invoices" t-as="credit_note_invoice">
                                        <t t-set="seq_and_num" t-value="credit_note_invoice.l10n_es_tbai_post_document_id._get_tbai_sequence_and_number_purchase()"/>
                                        <SerieFactura t-out="seq_and_num[0]"/>
                                        <NumFactura t-out="seq_and_num[1]"/>
                                        <FechaExpedicionFactura t-out="format_date(credit_note_invoice.invoice_date)"/>
                                    </IDFacturaRectificadaSustituida>
                                </FacturasRectificadasSustituidas>
                            </t>
                        </CabeceraFactura>
                        <DatosFactura>
                            <DescripcionOperacion t-out="ref"/>

                            <Claves>
                                <IDClave t-foreach="regime_key" t-as="key">
                                    <ClaveRegimenIvaOpTrascendencia t-out="key"/>
                                </IDClave>
                            </Claves>
                            <ImporteTotalFactura t-out="format_float(amount_total)"/>
                        </DatosFactura>
                        <IVA t-if="not freelancer">
                            <DetalleIVA t-foreach="iva_values" t-as="tax">
                                <CompraBienesCorrientesGastosBienesInversion t-out="tax['code']"/>
                                <InversionSujetoPasivo t-out="'N' if tax['rec'].l10n_es_type != 'sujeto_isp' else 'S'"/>
                                <BaseImponible t-out="format_float(tax['base'])"/>
                                <TipoImpositivo t-out="tax['rec'].amount"/>
                                <CuotaIVASoportada t-out="format_float(tax['tax'])"/>
                                <CuotaIVADeducible t-out="format_float(tax['tax']) if tax['rec'].l10n_es_type != 'no_deducible' else '0.00'"/>
                            </DetalleIVA>
                        </IVA>
                        <RentaIVA t-elif="freelancer">
                            <DetalleRentaIVA t-foreach="iva_values" t-as="tax">
                                <Epigrafe t-out="epigrafe"/>
                                <InversionSujetoPasivo t-out="'N' if tax['rec'].l10n_es_type != 'sujeto_isp' else 'S'"/>
                                <BaseImponible t-out="format_float(tax['base'])"/>
                                <TipoImpositivo t-out="tax['rec'].amount"/>
                                <CuotaIVASoportada t-out="format_float(tax['tax'])"/>
                                <CuotaIVADeducible t-out="format_float(tax['tax']) if tax['rec'].l10n_es_type != 'no_deducible' else '0.00'"/>
                            </DetalleRentaIVA>
                        </RentaIVA>
                    </t>
        </template>
    </data>
</odoo>

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from markupsafe import Markup
from psycopg2.errors import LockNotAvailable

from odoo import _, api, fields, models
from odoo.exceptions import UserError

TBAI_REFUND_REASONS = [
    ('R1', "R1: Art. 80.1, 80.2, 80.6 and rights founded error"),
    ('R2', "R2: Art. 80.3"),
    ('R3', "R3: Art. 80.4"),
    ('R4', "R4: Art. 80 - other"),
    ('R5', "R5: Factura rectificativa en facturas simplificadas"),
]


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_es_tbai_state = fields.Selection([
            ('to_send', 'To Send'),
            ('sent', 'Sent'),
            ('cancelled', 'Cancelled'),
        ],
        string='TicketBAI status',
        compute='_compute_l10n_es_tbai_state',
    )
    l10n_es_tbai_chain_index = fields.Integer(
        string="TicketBAI chain index",
        help="Invoice index in chain, set if and only if an in-chain XML was submitted and did not error",
        related='l10n_es_tbai_post_document_id.chain_index',
    )

    l10n_es_tbai_post_document_id = fields.Many2one(
        comodel_name='l10n_es_edi_tbai.document',
        readonly=True,
        copy=False,
    )
    l10n_es_tbai_cancel_document_id = fields.Many2one(
        comodel_name='l10n_es_edi_tbai.document',
        readonly=True,
        copy=False,
    )

    l10n_es_tbai_post_file = fields.Binary(
        string="TicketBAI Post File",
        related='l10n_es_tbai_post_document_id.xml_attachment_id.datas',
    )
    l10n_es_tbai_post_file_name = fields.Char(
        string="TicketBAI Post Attachment Name",
        related="l10n_es_tbai_post_document_id.xml_attachment_id.name",
    )
    l10n_es_tbai_cancel_file = fields.Binary(
        string="TicketBAI Cancel File",
        related='l10n_es_tbai_cancel_document_id.xml_attachment_id.datas',
    )
    l10n_es_tbai_cancel_file_name = fields.Char(
        string="TicketBAI Cancel File Name",
        related='l10n_es_tbai_cancel_document_id.xml_attachment_id.name',
    )

    l10n_es_tbai_is_required = fields.Boolean(
        string="TicketBAI required",
        help="Is the Basque EDI (TicketBAI) needed ?",
        compute='_compute_l10n_es_tbai_is_required',
    )

    l10n_es_tbai_refund_reason = fields.Selection(
        selection=TBAI_REFUND_REASONS,
        string="Invoice Refund Reason Code (TicketBai)",
        help="BOE-A-1992-28740. Ley 37/1992, de 28 de diciembre, del Impuesto sobre el "
        "Valor Añadido. Artículo 80. Modificación de la base imponible.",
        copy=False,
    )
    l10n_es_tbai_reversed_ids = fields.Many2many(
        'account.move', 'account_move_tbai_reversed_moves', 'refund_id', 'reversed_move_id',
        string="Refunded Vendor Bills",
        domain="[('move_type', '=', 'in_invoice'), ('commercial_partner_id', '=', commercial_partner_id)]",
        help="In the case where a vendor refund has multiple original invoices, you can set them here. ",
    )

    # -------------------------------------------------------------------------
    # API-DECORATED & EXTENDED METHODS
    # -------------------------------------------------------------------------

    @api.depends('l10n_es_tbai_post_document_id.state', 'l10n_es_tbai_cancel_document_id.state')
    def _compute_l10n_es_tbai_state(self):
        for move in self:
            state = 'to_send' if move.l10n_es_tbai_is_required else None
            if move.l10n_es_tbai_post_document_id and move.l10n_es_tbai_post_document_id.state == 'accepted':
                state = 'sent'
            if move.l10n_es_tbai_cancel_document_id and move.l10n_es_tbai_cancel_document_id.state == 'accepted':
                state = 'cancelled'

            move.l10n_es_tbai_state = state

    @api.depends('move_type', 'company_id')
    def _compute_l10n_es_tbai_is_required(self):
        for move in self:
            move.l10n_es_tbai_is_required = (
                move.company_id.l10n_es_tbai_is_enabled
                and (
                    move.is_sale_document()
                    or move.is_purchase_document() and move.company_id.l10n_es_tbai_tax_agency == 'bizkaia'
                )
                and any(not line._l10n_es_tbai_is_ignored() for line in move.invoice_line_ids)
            )

    @api.depends('l10n_es_tbai_post_document_id.chain_index')
    def _compute_show_reset_to_draft_button(self):
        # EXTENDS account_edi account.move
        super()._compute_show_reset_to_draft_button()

        for move in self:
            if move.l10n_es_tbai_chain_index:
                move.show_reset_to_draft_button = False

    def button_draft(self):
        # EXTENDS account account.move
        for move in self:
            if move.l10n_es_tbai_chain_index and move.l10n_es_tbai_state != 'cancelled':
                # NOTE this last condition (state is cancelled) is there because
                # button_cancel calls button_draft.
                # Draft button does not appear for user.
                raise UserError(_("You cannot reset to draft an entry that has been posted to TicketBAI's chain"))
        super().button_draft()

    @api.ondelete(at_uninstall=False)
    def _l10n_es_tbai_unlink_except_in_chain(self):
        # Prevent deleting moves that are part of the TicketBAI chain
        if not self._context.get('force_delete') and any(m.l10n_es_tbai_chain_index for m in self):
            raise UserError(_('You cannot delete a move that has a TicketBAI chain id.'))

    # -------------------------------------------------------------------------
    # HELPER METHODS
    # -------------------------------------------------------------------------

    def _l10n_es_tbai_check_can_send(self):
        # Ensure the move is posted
        if self.state != 'posted':
            return _("Cannot send an entry that is not posted to TicketBAI.")
        if self.l10n_es_tbai_state in ('sent', 'cancelled'):
            return _("This entry has already been posted.")
        if self.company_id.l10n_es_tbai_tax_agency == 'bizkaia' and self.is_purchase_document() and not self.ref:
            return _("You need to fill in the Reference field as the invoice number from your vendor.")


    def _l10n_es_tbai_get_attachment_name(self, cancel=False):
        return self.name + ('_post.xml' if not cancel else '_cancel.xml')

    def _l10n_es_tbai_create_edi_document(self, cancel=False):
        name = self.name
        if self.is_purchase_document():
            name = self.ref
        return self.env['l10n_es_edi_tbai.document'].sudo().create({
            'name': name,
            'date': self.date,
            'company_id': self.company_id.id,
            'is_cancel': cancel,
        })

    def _l10n_es_tbai_post_document_in_chatter(self, message, cancel=False):
        test_suffix = '(test mode)' if self.company_id.l10n_es_tbai_test_env else ''
        self.with_context(no_new_invoice=True).message_post(
            body=Markup("<pre>TicketBAI: posted {document_type} XML {test_suffix}\n{message}</pre>").format(
                document_type='emission' if not cancel else 'cancellation',
                test_suffix=test_suffix,
                message=message,
            ),
            attachment_ids=[self.l10n_es_tbai_post_document_id.xml_attachment_id.id] if not cancel else [self.l10n_es_tbai_cancel_document_id.xml_attachment_id.id],
        )

    def _l10n_es_tbai_lock_move(self):
        """ Acquire a write lock on the invoices in self. """
        self.ensure_one()

        try:
            with self.env.cr.savepoint(flush=False):
                self.env.cr.execute('SELECT * FROM account_move WHERE id = %s FOR UPDATE NOWAIT', [self.id])
        except LockNotAvailable:
            raise UserError(_('Cannot send this entry as it is already being processed.'))

    # -------------------------------------------------------------------------
    # WEB SERVICE CALLS
    # -------------------------------------------------------------------------

    def l10n_es_tbai_send_bill(self):
        for bill in self:
            error = bill._l10n_es_tbai_post()
            if self.env['account.move.send']._can_commit():
                self._cr.commit()
            if error:
                raise UserError(error)

    def l10n_es_tbai_cancel(self):
        for invoice in self:
            invoice._l10n_es_tbai_lock_move()

            if invoice.l10n_es_tbai_cancel_document_id and invoice.l10n_es_tbai_cancel_document_id.state == 'rejected':
                invoice.l10n_es_tbai_cancel_document_id.sudo().unlink()

            if not invoice.l10n_es_tbai_cancel_document_id:
                invoice.l10n_es_tbai_cancel_document_id = invoice._l10n_es_tbai_create_edi_document(cancel=True)

            edi_document = invoice.l10n_es_tbai_cancel_document_id

            error = edi_document._post_to_web_service(invoice._l10n_es_tbai_get_values(cancel=True))
            if error:
                raise UserError(error)

            if edi_document.state == 'accepted':
                invoice.button_cancel()
                invoice._l10n_es_tbai_post_document_in_chatter(edi_document.response_message, cancel=True)

            if self.env['account.move.send']._can_commit():
                self._cr.commit()

            if edi_document.state != 'accepted':
                raise UserError(edi_document.response_message)

    def _l10n_es_tbai_post(self):
        self.ensure_one()

        # Avoid the move to be sent if it is being modified by a parallel transaction (for example reset to draft)
        # It will also avoid the move to be sent by different parallel transactions
        self._l10n_es_tbai_lock_move()

        error = self._l10n_es_tbai_check_can_send()
        if error:
            return error

        if self.l10n_es_tbai_post_document_id and self.l10n_es_tbai_post_document_id.state == 'rejected':
            self.l10n_es_tbai_post_document_id.sudo().unlink()

        if not self.l10n_es_tbai_post_document_id:
            self.l10n_es_tbai_post_document_id = self._l10n_es_tbai_create_edi_document()

        edi_document = self.l10n_es_tbai_post_document_id

        error = edi_document._post_to_web_service(self._l10n_es_tbai_get_values())
        if error:
            return error

        if edi_document.state == 'accepted':
            self._l10n_es_tbai_post_document_in_chatter(edi_document.response_message)
            return

        # Return the error message if the xml document was not accepted
        return edi_document.response_message

    # -------------------------------------------------------------------------
    # XML DOCUMENT
    # -------------------------------------------------------------------------

    def _l10n_es_tbai_get_values(self, cancel=False):
        values = {
            'is_sale': self.is_sale_document(),
            'partner': self.commercial_partner_id,
            'is_simplified': self.l10n_es_is_simplified,
            'delivery_date': self.delivery_date if self.delivery_date and self.delivery_date != self.invoice_date else None,
            **self._l10n_es_tbai_get_attachment_values(cancel),
        }
        if values['is_sale']:
            values.update(self._l10n_es_tbai_get_invoice_values(cancel=cancel))

        elif self.company_id.l10n_es_tbai_tax_agency == 'bizkaia':
            values.update(self._l10n_es_tbai_get_vendor_bill_values_batuz())

        return values

    def _l10n_es_tbai_get_attachment_values(self, cancel=False):
        return {
            'attachment_name': self._l10n_es_tbai_get_attachment_name(cancel=cancel),
            'res_model': 'account.move',
            'res_id': self.id,
        }

    def _l10n_es_tbai_get_invoice_values(self, cancel=False):
        self.ensure_one()
        base_amls = self.line_ids.filtered(lambda x: x.display_type == 'product')
        base_lines = [self._prepare_product_base_line_for_taxes_computation(x) for x in base_amls]
        for base_line in base_lines:
            base_line['name'] = base_line['record'].name
        tax_amls = self.line_ids.filtered(lambda x: x.display_type == 'tax')
        tax_lines = [self._prepare_tax_line_for_taxes_computation(x) for x in tax_amls]
        self.env['l10n_es_edi_tbai.document']._add_base_lines_tax_amounts(base_lines, self.company_id, tax_lines=tax_lines)
        taxes = self.invoice_line_ids.tax_ids.flatten_taxes_hierarchy()
        is_oss = any(tax._l10n_es_get_regime_code() == '17' for tax in taxes)

        return {
            **self._l10n_es_tbai_get_credit_note_values(),
            'origin': self.invoice_origin and self.invoice_origin[:250] or 'manual',
            'taxes': taxes,
            'rate':  abs(self.amount_total / self.amount_total_signed) if self.amount_total else 1,
            'base_lines': base_lines,
            'nosujeto_causa': 'IE' if is_oss else 'RL',
            **({'post_doc': self.l10n_es_tbai_post_document_id} if cancel else {}),
        }

    def _l10n_es_tbai_get_credit_note_values(self):
        return {
            'is_refund': self.move_type == 'out_refund',
            'refund_reason': self.l10n_es_tbai_refund_reason,
            'refunded_doc': self.reversed_entry_id.l10n_es_tbai_post_document_id,
            'refunded_doc_invoice_date': self.reversed_entry_id.invoice_date if self.reversed_entry_id else False,
        }

    def _l10n_es_tbai_get_vendor_bill_values_batuz(self):
        """ For the vendor bills for Bizkaia, the structure is different than the regular Ticketbai XML (LROE)"""
        values = {
            'ref': self.ref,
            'is_refund': self.move_type == 'in_refund',
            'invoice_date': self.invoice_date,
            'tipofactura': 'F5' if self._l10n_es_is_dua() else 'F1',
             **self._l10n_es_tbai_get_vendor_bill_tax_values(),
        }
        # Check if intracom
        mod_303_10 = self.env.ref('l10n_es.mod_303_casilla_10_balance')._get_matching_tags()
        mod_303_11 = self.env.ref('l10n_es.mod_303_casilla_11_balance')._get_matching_tags()
        tax_tags = self.invoice_line_ids.tax_ids.flatten_taxes_hierarchy().repartition_line_ids.tag_ids
        intracom = bool(tax_tags & (mod_303_10 + mod_303_11))
        values['regime_key'] = ['09'] if intracom else ['01']
        # Credit notes (factura rectificativa)
        if values['is_refund']:
            values['refund_reason'] = self.l10n_es_tbai_refund_reason
            values['credit_note_invoices'] = self.reversed_entry_id | self.l10n_es_tbai_reversed_ids

        return values

    def _l10n_es_tbai_get_vendor_bill_tax_values(self):
        self.ensure_one()
        results = defaultdict(lambda: {'base_amount': 0.0, 'tax_amount': 0.0})
        amount_total = 0.0
        for line in self.line_ids.filtered(lambda l: l.display_type in ('product', 'tax')):
            if any(t.l10n_es_type == 'ignore' for t in line.tax_ids) or line.tax_line_id.l10n_es_type == 'ignore':
                continue
            if line.tax_line_id.l10n_es_type != 'retencion':
                amount_total += line.balance
            for tax in line.tax_ids.filtered(lambda t: t.l10n_es_type not in ('recargo', 'retencion')):
                results[tax]['base_amount'] += line.balance

            if ((tax := line.tax_line_id) and tax.l10n_es_type not in ('recargo', 'retencion') and
                line.tax_repartition_line_id.factor_percent != -100.0):
                results[tax]['tax_amount'] += line.balance
        iva_values = []
        for tax in results:
            code = "C"  # Bienes Corrientes
            if tax.l10n_es_bien_inversion:
                code = "I"  # Investment Goods
            if tax.tax_scope == 'service':
                code = 'G'  # Gastos
            iva_values.append({'base': results[tax]['base_amount'],
                               'code': code,
                               'tax': results[tax]['tax_amount'],
                               'rec': tax})
        return {'iva_values': iva_values,
                'amount_total': amount_total}

    def _refunds_origin_required(self):
        if self.l10n_es_tbai_is_required:
            return True
        return super()._refunds_origin_required()

```

## File: models\account_move_line.py

```python
from odoo import models


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    def _l10n_es_tbai_is_ignored(self):
        self.ensure_one()

        return 'ignore' in self.tax_ids.mapped('l10n_es_type')

```

## File: models\account_move_send.py

```python
from odoo import _, api, models


class AccountMoveSend(models.AbstractModel):
    _inherit = 'account.move.send'

    @api.model
    def _is_tbai_applicable(self, move):
        return move.l10n_es_tbai_is_required and move.l10n_es_tbai_state == 'to_send'

    def _get_all_extra_edis(self) -> dict:
        # EXTENDS 'account'
        res = super()._get_all_extra_edis()
        res.update({'es_tbai': {'label': _("TicketBAI"), 'is_applicable': self._is_tbai_applicable, 'help': _('Send the e-invoice to the Basque Government.')}})
        return res

    # -------------------------------------------------------------------------
    # ATTACHMENTS
    # -------------------------------------------------------------------------

    def _get_invoice_extra_attachments(self, move):
        # EXTENDS 'account'
        return super()._get_invoice_extra_attachments(move) + move.l10n_es_tbai_post_document_id.xml_attachment_id

    def _get_placeholder_mail_attachments_data(self, move, invoice_edi_format=None, extra_edis=None):
        # EXTENDS 'account'
        results = super()._get_placeholder_mail_attachments_data(move, invoice_edi_format=invoice_edi_format, extra_edis=extra_edis)

        if (
            not move.l10n_es_tbai_post_document_id.xml_attachment_id
            and 'es_tbai' in extra_edis
        ):
            filename = move._l10n_es_tbai_get_attachment_name()
            results.append({
                'id': f'placeholder_{filename}',
                'name': filename,
                'mimetype': 'application/xml',
                'placeholder': True,
            })

        return results

    # -------------------------------------------------------------------------
    # SENDING METHODS
    # -------------------------------------------------------------------------

    def _call_web_service_before_invoice_pdf_render(self, invoices_data):
        # EXTENDS 'account'
        super()._call_web_service_before_invoice_pdf_render(invoices_data)

        for invoice, invoice_data in invoices_data.items():

            if 'es_tbai' in invoice_data['extra_edis']:
                error = invoice._l10n_es_tbai_post()

                if error:
                    invoice_data['error'] = {
                        'error_title': _("Error when sending the invoice to TicketBAI:"),
                        'errors': [error],
                    }

                if self._can_commit():
                    self._cr.commit()

```

## File: models\certificate.py

```python
import base64

from cryptography import x509

from odoo import fields, models


class Certificate(models.Model):
    _inherit = 'certificate.certificate'

    scope = fields.Selection(
        selection_add=[
            ('tbai', 'TBAI')
        ],
    )

    def _l10n_es_edi_tbai_get_issuer(self):
        self.ensure_one()

        cert = x509.load_pem_x509_certificate(base64.b64decode(self.pem_certificate))

        common_name = cert.issuer.get_attributes_for_oid(x509.oid.NameOID.COMMON_NAME)[0].value
        org_unit = cert.issuer.get_attributes_for_oid(x509.oid.NameOID.ORGANIZATIONAL_UNIT_NAME)[0].value
        org_name = cert.issuer.get_attributes_for_oid(x509.oid.NameOID.ORGANIZATION_NAME)[0].value
        country_name = cert.issuer.get_attributes_for_oid(x509.oid.NameOID.COUNTRY_NAME)[0].value

        return f'CN={common_name}, OU={org_unit}, O={org_name}, C={country_name}'

```

## File: models\l10n_es_edi_tbai_agencies.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


# ===== TicketBAI TAX AGENCY METADATA =====

def get_key(agency, key, is_test_env=True):
    """
    Helper method to retrieve specific data about certain agencies.
    Notable differences in structure, by key
    - Any key ending with '_':
    These keys have two variants: 'test' and 'prod'. The parameter `is_test_env` matters for those keys only.
    - 'xsd_url':
    Araba and Gipuzkoa each have a single URL pointing to a zip file (which may contain many XSDs)
    Bizkaia has two URLs for post/cancel XSDs: in that case a dict of strings is returned (instead of a single string)
    """
    urls = {
        'araba': URLS_ARABA,
        'bizkaia': URLS_BIZKAIA,
        'gipuzkoa': URLS_GIPUZKOA,
    }[agency]
    if key.endswith('_'):
        key += 'test' if is_test_env else 'prod'
    return urls[key]


URLS_ARABA = {
    'sigpolicy_url': 'https://ticketbai.araba.eus/tbai/sinadura/',
    'sigpolicy_digest': '4Vk3uExj7tGn9DyUCPDsV9HRmK6KZfYdRiW3StOjcQA=',
    'xsd_url': 'https://web.araba.eus/documents/105044/5608600/TicketBai12+%282%29.zip',
    'xsd_name': {
        'post': 'ticketBaiV1-2.xsd',
        'cancel': 'Anula_ticketBaiV1-2.xsd',
    },
    'post_url_test': 'https://pruebas-ticketbai.araba.eus/TicketBAI/v1/facturas/',
    'post_url_prod': 'https://ticketbai.araba.eus/TicketBAI/v1/facturas/',
    'qr_url_test': 'https://pruebas-ticketbai.araba.eus/tbai/qrtbai/',
    'qr_url_prod': 'https://ticketbai.araba.eus/tbai/qrtbai/',
    'cancel_url_test': 'https://pruebas-ticketbai.araba.eus/TicketBAI/v1/anulaciones/',
    'cancel_url_prod': 'https://ticketbai.araba.eus/TicketBAI/v1/anulaciones/',
}

URLS_BIZKAIA = {
    'sigpolicy_url': 'https://www.batuz.eus/fitxategiak/batuz/ticketbai/sinadura_elektronikoaren_zehaztapenak_especificaciones_de_la_firma_electronica_v1_0.pdf',
    'sigpolicy_digest': 'Quzn98x3PMbSHwbUzaj5f5KOpiH0u8bvmwbbbNkO9Es=',
    'xsd_url': {
        'post': 'https://www.batuz.eus/fitxategiak/batuz/ticketbai/ticketBaiV1-2-1.xsd',
        'cancel': 'https://www.batuz.eus/fitxategiak/batuz/ticketbai/Anula_ticketBaiV1-2-1.xsd',
    },
    'xsd_name': {
        'post': 'ticketBaiV1-2-1.xsd',
        'cancel': 'Anula_ticketBaiV1-2-1.xsd',
    },
    'post_url_test': 'https://pruesarrerak.bizkaia.eus/N3B4000M/aurkezpena',
    'post_url_prod': 'https://sarrerak.bizkaia.eus/N3B4000M/aurkezpena',
    'qr_url_test': 'https://batuz.eus/QRTBAI/',
    'qr_url_prod': 'https://batuz.eus/QRTBAI/',
    'cancel_url_test': 'https://pruesarrerak.bizkaia.eus/N3B4000M/aurkezpena',
    'cancel_url_prod': 'https://sarrerak.bizkaia.eus/N3B4000M/aurkezpena',
}

URLS_GIPUZKOA = {
    'sigpolicy_url': 'https://www.gipuzkoa.eus/TicketBAI/signature',
    'sigpolicy_digest': '6NrKAm60o7u62FUQwzZew24ra2ve9PRQYwC21AM6In0=',
    'xsd_url': 'https://www.gipuzkoa.eus/documents/2456431/13761107/Esquemas+de+archivos+XSD+de+env%C3%ADo+y+anulaci%C3%B3n+de+factura_1_2.zip/2d116f8e-4d3a-bff0-7b03-df1cbb07ec52',
    'xsd_name': {
        'post': 'ticketBaiV1-2-1.xsd',
        'cancel': 'Anula_ticketBaiV1-2-1.xsd',
    },
    'post_url_test': 'https://tbai-prep.egoitza.gipuzkoa.eus/WAS/HACI/HTBRecepcionFacturasWEB/rest/recepcionFacturas/alta',
    'post_url_prod': 'https://tbai-z.egoitza.gipuzkoa.eus/sarrerak/alta',
    'qr_url_test': 'https://tbai.prep.gipuzkoa.eus/qr/',
    'qr_url_prod': 'https://tbai.egoitza.gipuzkoa.eus/qr/',
    'cancel_url_test': 'https://tbai-prep.egoitza.gipuzkoa.eus/WAS/HACI/HTBRecepcionFacturasWEB/rest/recepcionFacturas/anulacion',
    'cancel_url_prod': 'https://tbai-z.egoitza.gipuzkoa.eus/sarrerak/baja',
}

```

## File: models\l10n_es_edi_tbai_document.py

```python
import gzip
import json
import re
import base64
from datetime import datetime
from uuid import uuid4

import requests
from lxml import etree
from pytz import timezone
from requests.exceptions import RequestException

from odoo import _, api, fields, models, release
from odoo.addons.l10n_es_edi_sii.models.account_edi_format import PatchedHTTPAdapter
from odoo.addons.l10n_es_edi_tbai.models.l10n_es_edi_tbai_agencies import get_key
from odoo.addons.l10n_es_edi_tbai.models.xml_utils import (
    NS_MAP,
    calculate_references_digests,
    canonicalize_node,
    cleanup_xml_signature,
)
from odoo.exceptions import UserError
from odoo.tools import get_lang
from odoo.tools.float_utils import float_repr, float_round
from odoo.tools.xml_utils import cleanup_xml_node

CRC8_TABLE = [
    0x00, 0x07, 0x0E, 0x09, 0x1C, 0x1B, 0x12, 0x15, 0x38, 0x3F, 0x36, 0x31, 0x24, 0x23, 0x2A, 0x2D,
    0x70, 0x77, 0x7E, 0x79, 0x6C, 0x6B, 0x62, 0x65, 0x48, 0x4F, 0x46, 0x41, 0x54, 0x53, 0x5A, 0x5D,
    0xE0, 0xE7, 0xEE, 0xE9, 0xFC, 0xFB, 0xF2, 0xF5, 0xD8, 0xDF, 0xD6, 0xD1, 0xC4, 0xC3, 0xCA, 0xCD,
    0x90, 0x97, 0x9E, 0x99, 0x8C, 0x8B, 0x82, 0x85, 0xA8, 0xAF, 0xA6, 0xA1, 0xB4, 0xB3, 0xBA, 0xBD,
    0xC7, 0xC0, 0xC9, 0xCE, 0xDB, 0xDC, 0xD5, 0xD2, 0xFF, 0xF8, 0xF1, 0xF6, 0xE3, 0xE4, 0xED, 0xEA,
    0xB7, 0xB0, 0xB9, 0xBE, 0xAB, 0xAC, 0xA5, 0xA2, 0x8F, 0x88, 0x81, 0x86, 0x93, 0x94, 0x9D, 0x9A,
    0x27, 0x20, 0x29, 0x2E, 0x3B, 0x3C, 0x35, 0x32, 0x1F, 0x18, 0x11, 0x16, 0x03, 0x04, 0x0D, 0x0A,
    0x57, 0x50, 0x59, 0x5E, 0x4B, 0x4C, 0x45, 0x42, 0x6F, 0x68, 0x61, 0x66, 0x73, 0x74, 0x7D, 0x7A,
    0x89, 0x8E, 0x87, 0x80, 0x95, 0x92, 0x9B, 0x9C, 0xB1, 0xB6, 0xBF, 0xB8, 0xAD, 0xAA, 0xA3, 0xA4,
    0xF9, 0xFE, 0xF7, 0xF0, 0xE5, 0xE2, 0xEB, 0xEC, 0xC1, 0xC6, 0xCF, 0xC8, 0xDD, 0xDA, 0xD3, 0xD4,
    0x69, 0x6E, 0x67, 0x60, 0x75, 0x72, 0x7B, 0x7C, 0x51, 0x56, 0x5F, 0x58, 0x4D, 0x4A, 0x43, 0x44,
    0x19, 0x1E, 0x17, 0x10, 0x05, 0x02, 0x0B, 0x0C, 0x21, 0x26, 0x2F, 0x28, 0x3D, 0x3A, 0x33, 0x34,
    0x4E, 0x49, 0x40, 0x47, 0x52, 0x55, 0x5C, 0x5B, 0x76, 0x71, 0x78, 0x7F, 0x6A, 0x6D, 0x64, 0x63,
    0x3E, 0x39, 0x30, 0x37, 0x22, 0x25, 0x2C, 0x2B, 0x06, 0x01, 0x08, 0x0F, 0x1A, 0x1D, 0x14, 0x13,
    0xAE, 0xA9, 0xA0, 0xA7, 0xB2, 0xB5, 0xBC, 0xBB, 0x96, 0x91, 0x98, 0x9F, 0x8A, 0x8D, 0x84, 0x83,
    0xDE, 0xD9, 0xD0, 0xD7, 0xC2, 0xC5, 0xCC, 0xCB, 0xE6, 0xE1, 0xE8, 0xEF, 0xFA, 0xFD, 0xF4, 0xF3
]


class L10nEsEdiTbaiDocument(models.Model):
    _name = 'l10n_es_edi_tbai.document'
    _description = 'TicketBAI Document'

    name = fields.Char(
        required=True,
        readonly=True,
    )
    date = fields.Date(
        required=True,
        readonly=True,
    )
    xml_attachment_id = fields.Many2one(
        comodel_name='ir.attachment',
        string="XML Attachment",
        copy=False,
        readonly=True,
    )
    company_id = fields.Many2one(
        'res.company',
        required=True,
    )

    state = fields.Selection([
            ('to_send', "To Send"),
            ('accepted', "Accepted"),
            ('rejected', "Rejected"),
        ],
        string="status",
        default='to_send',
        copy=False,
        readonly=True,
    )
    chain_index = fields.Integer(
        copy=False,
        readonly=True,
    )
    response_message = fields.Text(
        copy=False,
        readonly=True,
    )

    is_cancel = fields.Boolean(
        default=False,
        readonly=True,
    )

    # -------------------------------------------------------------------------
    # HELPER METHODS
    # -------------------------------------------------------------------------

    def _is_in_chain(self):
        """True iff the document has been posted to the chain and confirmed by govt."""
        return self.chain_index and self.state == 'accepted'

    def _check_can_post(self, values):
        # Ensure a certificate is available.
        if not self.company_id.l10n_es_tbai_certificate_id:
            return _("Please configure the certificate for TicketBAI.")

        # Ensure a tax agency is available.
        if not self.company_id.l10n_es_tbai_tax_agency:
            return _("Please specify a tax agency on your company for TicketBAI.")

        # Ensure a vat is available.
        if not self.company_id.vat:
            return _("Please configure the Tax ID on your company for TicketBAI.")

        if self.company_id._l10n_es_freelancer() and not self.env['ir.config_parameter'].sudo().get_param('l10n_es_edi_tbai.epigrafe', False):
            return _("In order to use Ticketbai Batuz for freelancers, you will need to configure the "
                        "Epigrafe or Main Activity.  In this version, you need to go in debug mode to "
                        "Settings > Technical > System Parameters and set the parameter 'l10n_es_edi_tbai.epigrafe'"
                        "to your epigrafe number. You can find them in %s",
                        "https://www.batuz.eus/fitxategiak/batuz/lroe/batuz_lroe_lista_epigrafes_v1_0_3.xlsx")

        if values['is_sale'] and not self.is_cancel:
            if any(not base_line['tax_ids'] for base_line in values['base_lines']):
                return self.env._("There should be at least one tax set on each line in order to send to TicketBAI.")

            # Chain integrity check: chain head must have been REALLY posted
            chain_head_doc = self.company_id._get_l10n_es_tbai_last_chained_document()
            if chain_head_doc and chain_head_doc != self and chain_head_doc.state != 'accepted':
                return _("TicketBAI: Cannot post invoice while chain head (%s) has not been posted", chain_head_doc.name)

            # Tax configuration check: In case of foreign customer we need the tax scope to be set
            if values['partner'] and values['partner']._l10n_es_is_foreign() and values['taxes'].filtered(lambda t: not t.tax_scope):
                return _(
                    "In case of a foreign customer, you need to configure the tax scope on taxes:\n%s",
                    "\n".join(values['taxes'].mapped('name'))
                )
            if values['is_refund']:
                refunded_doc = values['refunded_doc']
                refund_reason = values['refund_reason']
                is_simplified = values['is_simplified']

                if not refunded_doc or refunded_doc.state == 'to_send':
                    return _("TicketBAI: Cannot post a reversal document while the source document has not been posted")
                if not refund_reason:
                    return _('Refund reason must be specified (TicketBAI)')
                if is_simplified and refund_reason != 'R5':
                    return _('Refund reason must be R5 for simplified invoices (TicketBAI)')
                if not is_simplified and refund_reason == 'R5':
                    return _('Refund reason cannot be R5 for non-simplified invoices (TicketBAI)')

    # -------------------------------------------------------------------------
    # WEB SERVICE CALLS
    # -------------------------------------------------------------------------

    def _post_to_web_service(self, values):
        self.ensure_one()

        error = self._check_can_post(values)
        if error:
            return error

        if not self.xml_attachment_id:
            self._generate_xml(values)

        if (
            not self.chain_index
            and not self.is_cancel
            and values['is_sale']
        ):
            # Assign unique 'chain index' from dedicated sequence
            self.sudo().chain_index = self.company_id._get_l10n_es_tbai_next_chain_index()

        try:
            # Call the web service, retrieve and parse response
            success, response_msgs = self._post_to_agency(self.env, values['is_sale'])
        except (RequestException) as e:
            # In case of timeout / request exception
            self.sudo().response_message = e
            return

        self.sudo().response_message = '\n'.join(response_msgs)
        if success:
            self.sudo().state = 'accepted'
        else:
            self.sudo().state = 'rejected'
            self.sudo().chain_index = 0

    def _post_to_agency(self, env, is_sale):

        def _send_request_to_agency(*args, **kwargs):
            session = requests.Session()
            session.cert = kwargs.pop('pkcs12_data')
            session.mount("https://", PatchedHTTPAdapter())
            response = session.request('post', *args, **kwargs)
            response.raise_for_status()
            response_xml = None
            error = None
            if response.content:
                try:
                    response_xml = etree.fromstring(response.content)
                except etree.XMLSyntaxError as e:
                    error = str(e)
            else:
                error = self.env._('No XML response received.')
            return response.headers, response_xml, [error] if error else []

        if self.company_id.l10n_es_tbai_tax_agency in ('araba', 'gipuzkoa'):
            params = self._prepare_post_params_ar_gi()
            _response_headers, response_xml, errors = _send_request_to_agency(timeout=10, **params)
            if errors:
                return False, errors
            return self._process_post_response_xml_ar_gi(env, response_xml)

        elif self.company_id.l10n_es_tbai_tax_agency == 'bizkaia':
            params = self._prepare_post_params_bi(is_sale)
            response_headers, response_xml, errors = _send_request_to_agency(timeout=10, **params)
            if response_headers['eus-bizkaia-n3-tipo-respuesta'] != "Correcto":
                error_code = response_headers['eus-bizkaia-n3-codigo-respuesta']
                error_msg = response_headers['eus-bizkaia-n3-mensaje-respuesta']
                errors.append(error_code + ": " + error_msg)
            success, errors_add = self._process_post_response_xml_bi(env, response_xml)
            errors += errors_add
            return success, errors

    def _prepare_post_params_ar_gi(self):
        """Web service parameters for Araba and Gipuzkoa."""
        company = self.company_id
        return {
            'url': get_key(self.company_id.l10n_es_tbai_tax_agency, 'cancel_url_' if self.is_cancel else 'post_url_', company.l10n_es_tbai_test_env),
            'headers': {"Content-Type": "application/xml; charset=utf-8"},
            'pkcs12_data': company.l10n_es_tbai_certificate_id,
            'data': self.xml_attachment_id.raw,
        }

    @api.model
    def _process_post_response_xml_ar_gi(self, env, response_xml):
        """Government response processing for Araba and Gipuzkoa."""
        success = int(response_xml.findtext('.//Estado')) == 0
        response_msgs = []

        # Get message in basque if env is in basque
        msg_node_name = 'Azalpena' if get_lang(env).code == 'eu_ES' else 'Descripcion'
        for res_node in response_xml.findall('.//ResultadosValidacion'):
            msg_code = res_node.findtext('Codigo')
            response_msgs.append(msg_code + ": " + res_node.findtext(msg_node_name))
            if msg_code in ('005', '019'):
                success = True  # error codes 5/19 mean XML was already received with that sequence

        return success, response_msgs

    def _prepare_post_params_bi(self, is_sale):
        """Web service parameters for Bizkaia."""
        company = self.company_id
        freelancer = company._l10n_es_freelancer()

        if is_sale:
            xml_to_send = self._generate_final_xml_bi(freelancer=freelancer)
            lroe_str = etree.tostring(xml_to_send)
        else:
            lroe_str = self.xml_attachment_id.raw

        lroe_bytes = gzip.compress(lroe_str)


        return {
            'url': get_key(company.l10n_es_tbai_tax_agency, 'cancel_url_' if self.is_cancel else 'post_url_', company.l10n_es_tbai_test_env),
            'headers': {
                'Accept-Encoding': 'gzip',
                'Content-Encoding': 'gzip',
                'Content-Length': str(len(lroe_str)),
                'Content-Type': 'application/octet-stream',
                'eus-bizkaia-n3-version': '1.0',
                'eus-bizkaia-n3-content-type': 'application/xml',
                'eus-bizkaia-n3-data': json.dumps({
                    'con': 'LROE',
                    'apa': '2.1' if freelancer and not is_sale else '1.1' if is_sale else '2',
                    'inte': {
                        'nif': company.vat[2:] if company.vat.startswith('ES') else company.vat,
                        'nrs': company.name,
                    },
                    'drs': {
                        'mode': '140' if freelancer else '240',
                        'ejer': str(self.date.year),
                    }
                }),
            },
            'pkcs12_data': company.l10n_es_tbai_certificate_id,
            'data': lroe_bytes,
        }

    def _generate_final_xml_bi(self, freelancer=False):
        sender = self.company_id
        lroe_values = {
            'is_emission': not self.is_cancel,
            'sender': sender,
            'sender_vat': sender.vat[2:] if sender.vat.startswith('ES') else sender.vat,
            'fiscal_year': str(self.date.year),
            'freelancer': freelancer,
            'is_freelancer': freelancer,  # For bugfix, will be removed in master
            'epigrafe': self.env['ir.config_parameter'].sudo().get_param('l10n_es_edi_tbai.epigrafe', '')
        }
        lroe_values.update({'tbai_b64_list': [base64.b64encode(self.xml_attachment_id.raw).decode()]})
        lroe_str = self.env['ir.qweb']._render('l10n_es_edi_tbai.template_LROE_240_main', lroe_values)
        lroe_xml = cleanup_xml_node(lroe_str)

        return lroe_xml

    @api.model
    def _process_post_response_xml_bi(self, env, response_xml):
        """Government response processing for Bizkaia."""
        if response_xml is None:
            return False, []
        success = response_xml.findtext('.//EstadoRegistro') == "Correcto"

        if success:
            return True, []

        error_code = response_xml.findtext('.//CodigoErrorRegistro')
        # Get message in basque if env is in basque
        error_msg_node_name = 'DescripcionErrorRegistro' + ('EU' if get_lang(env).code == 'eu_ES' else 'ES')
        error_msg = error_code + ": " + response_xml.findtext(f'.//{error_msg_node_name}', '')
        if error_code == "B4_2000003":  # already received
            success = True

        return success, [error_msg]

    # -------------------------------------------------------------------------
    # XML
    # -------------------------------------------------------------------------

    L10N_ES_TBAI_VERSION = 1.2

    def _generate_xml(self, values):
        self.ensure_one()

        def format_float(value, precision_digits=2):
            rounded_value = float_round(value, precision_digits=precision_digits)
            return float_repr(rounded_value, precision_digits=precision_digits)

        values.update({
            'doc': self,
            **self._get_header_values(),
            **self._get_sender_values(),
            **(self._get_recipient_values(values['partner'], values["is_simplified"]) if values['partner'] and not self.is_cancel or not values['is_sale'] else {}),
            'datetime_now': datetime.now(tz=timezone('Europe/Madrid')),
            'format_date': lambda d: datetime.strftime(d, '%d-%m-%Y'),
            'format_time': lambda d: datetime.strftime(d, '%H:%M:%S'),
            'format_float': format_float,
        })

        xml_doc = None

        if values['is_sale']:
            values.update({
                'is_emission': not self.is_cancel,
                **self.company_id._get_l10n_es_tbai_license_dict(),
                **(self._get_sale_values(values) if not self.is_cancel else {}),
            })
            xml_doc = self._generate_sale_document_xml(values)

        elif self.company_id.l10n_es_tbai_tax_agency == 'bizkaia':
            company = self.company_id
            freelancer = company._l10n_es_freelancer()
            values.update({'freelancer': freelancer})
            xml_doc = self._generate_purchase_document_xml_bi(values)

        if xml_doc is not None:
            self.sudo().xml_attachment_id = self.env['ir.attachment'].create({
                'name': values['attachment_name'],
                'raw': etree.tostring(xml_doc, encoding='UTF-8'),
                'type': 'binary',
                'res_model': values['res_model'],
                'res_id': values['res_id'],
            })

    @api.model
    def _get_header_values(self):
        return {
            'tbai_version': self.L10N_ES_TBAI_VERSION,
            'odoo_version': release.version,
        }

    @api.model
    def _get_sender_values(self):
        sender = self.company_id
        return {
            'sender_vat': sender.vat[2:] if sender.vat.startswith('ES') else sender.vat,
            'sender': sender,
        }

    def _get_recipient_values(self, partner, is_simplified=False):
        # TicketBAI accept recipient data for simplified invoices,
        # but only if the partner has a VAT number
        if is_simplified and not partner.vat:
            return {}
        recipient_values = {
            'partner': partner,
            'partner_address': ', '.join(filter(None, [partner.street, partner.street2, partner.city])),
            'alt_id_number': partner.vat or 'NO_DISPONIBLE',
        }

        if not partner._l10n_es_is_foreign() and partner.vat:
            recipient_values['nif'] = partner.vat[2:] if partner.vat.startswith('ES') else partner.vat

        elif partner.country_id in self.env.ref('base.europe').country_ids:
            recipient_values['alt_id_type'] = '02'

        else:
            recipient_values['alt_id_type'] = '04' if partner.vat else '06'
            recipient_values['alt_id_country'] = partner.country_id.code if partner.country_id else None

        return {'recipient': recipient_values}

    def _get_sale_values(self, values):
        sale_values = {
            'prev_doc': self.company_id._get_l10n_es_tbai_last_chained_document(),
            **self._get_regime_code_value(values['taxes'], values['is_simplified']),
        }

        if not values['partner'] or not values['partner']._l10n_es_is_foreign() or values["is_simplified"]:
            sale_values.update(**self._get_importe_desglose_es_partner(values['base_lines'], values['is_refund']))
        else:
            sale_values.update(**self._get_importe_desglose_foreign_partner(values['base_lines'], values['is_refund']))

        return sale_values

    def _get_regime_code_value(self, taxes, is_simplified):
        regime_key = []

        if is_simplified and self.company_id.l10n_es_tbai_tax_agency != 'bizkaia':
            regime_key.append('52')  # code for simplified invoices
        else:
            regime_key.append(taxes._l10n_es_get_regime_code())

        return {'regime_key': regime_key}

    @api.model
    def _add_base_lines_tax_amounts(self, base_lines, company, tax_lines=None):
        AccountTax = self.env['account.tax']
        AccountTax._add_tax_details_in_base_lines(base_lines, company)
        AccountTax._round_base_lines_tax_details(base_lines, company, tax_lines=tax_lines)

        for base_line in base_lines:
            discount = base_line['discount']
            price_unit = base_line['price_unit'] / base_line['rate'] if base_line['rate'] else 0.0
            quantity = base_line['quantity']
            price_subtotal = base_line['price_subtotal'] = base_line['tax_details']['raw_total_excluded']
            base_line['price_total'] = base_line['tax_details']['raw_total_included']
            for tax_data in base_line['tax_details']['taxes_data']:
                if tax_data['tax'].l10n_es_type == 'retencion':
                    base_line['price_total'] -= tax_data['tax_amount']

            if discount == 100.0:
                gross_price_subtotal_before_discount = price_unit * quantity
            else:
                gross_price_subtotal_before_discount = price_subtotal / (1 - discount / 100.0)

            base_line['gross_price_subtotal'] = gross_price_subtotal_before_discount
            base_line['discount_amount'] = gross_price_subtotal_before_discount - price_subtotal
            base_line['description'] = re.sub(r'[^0-9a-zA-Z ]+', '', base_line['name'] or base_line['product_id'].display_name or '')[:250]

            if quantity:
                base_line['gross_price_unit'] = gross_price_subtotal_before_discount / quantity
            else:
                base_line['gross_price_unit'] = 0.0

    @api.model
    def _build_tax_details_info(self, values_list):
        sujeta_no_sujeta = {}
        sujeto = []
        sujeto_isp = []
        encountered_l10n_es_type = set()
        for values in values_list:
            grouping_key = values['grouping_key']
            if not grouping_key:
                continue

            l10n_es_type = grouping_key['l10n_es_type']
            encountered_l10n_es_type.add(l10n_es_type)
            if l10n_es_type in ('sujeto', 'sujeto_isp'):
                tax_info = {
                    'TipoImpositivo': grouping_key['applied_tax_amount'],
                    'BaseImponible': float_round(values['base_amount'], 2),
                    'CuotaRepercutida': float_round(values['tax_amount'], 2),
                }
                sujeta_no_sujeta\
                    .setdefault('Sujeta', {})\
                    .setdefault('NoExenta', {})\
                    .setdefault('DesgloseIVA', {'DetalleIVA': []})['DetalleIVA']\
                    .append(tax_info)
                if l10n_es_type == 'sujeto':
                    sujeto.append(tax_info)
                else:
                    sujeto_isp.append(tax_info)
            elif l10n_es_type == 'exento':
                sujeta_no_sujeta\
                    .setdefault('Sujeta', {})\
                    .setdefault('Exenta', {'DetalleExenta': []})['DetalleExenta']\
                    .append({
                        'BaseImponible': float_round(values['base_amount'], 2),
                        'CausaExencion': grouping_key['l10n_es_exempt_reason'],
                    })
            elif l10n_es_type == 'recargo':
                detalle_iva = sujeta_no_sujeta\
                    .get('Sujeta', {})\
                    .get('NoExenta', {})\
                    .get('DesgloseIVA', {})\
                    .get('DetalleIVA')
                if detalle_iva:
                    detalle_iva[-1]['CuotaRecargoEquivalencia'] = float_round(values['tax_amount'], 2)
                    detalle_iva[-1]['TipoRecargoEquivalencia'] = grouping_key['applied_tax_amount']
            elif l10n_es_type == 'no_sujeto':
                no_sujeta = sujeta_no_sujeta.setdefault('NoSujeta', {})
                no_sujeta.setdefault('ImportePorArticulos7_14_Otros', 0.0)
                no_sujeta['ImportePorArticulos7_14_Otros'] += float_round(values['base_amount'], 2)
            elif l10n_es_type == 'no_sujeto_loc':
                no_sujeta = sujeta_no_sujeta.setdefault('NoSujeta', {})
                no_sujeta.setdefault('ImporteTAIReglasLocalizacion', 0.0)
                no_sujeta['ImporteTAIReglasLocalizacion'] += float_round(values['base_amount'], 2)

        if 'sujeto' in encountered_l10n_es_type and 'sujeto_isp' not in encountered_l10n_es_type:
            sujeta_no_sujeta['Sujeta']['NoExenta']['TipoNoExenta'] = 'S2'
        elif 'sujeto' not in encountered_l10n_es_type and 'sujeto_isp' in encountered_l10n_es_type:
            sujeta_no_sujeta['Sujeta']['NoExenta']['TipoNoExenta'] = 'S1'
        elif 'sujeto' in encountered_l10n_es_type and 'sujeto_isp' in encountered_l10n_es_type:
            sujeta_no_sujeta['Sujeta']['NoExenta']['TipoNoExenta'] = 'S3'

        return {
            'sujeta_no_sujeta': sujeta_no_sujeta,
            'sujeto': sujeto,
            'sujeto_isp': sujeto_isp,
        }

    @api.model
    def _get_importe_desglose_es_partner(self, base_lines, is_refund):
        AccountTax = self.env['account.tax']

        def tax_details_info_grouping_function(base_line, tax_data):
            tax = tax_data['tax']

            return {
                'applied_tax_amount': tax.amount,
                'l10n_es_type': tax.l10n_es_type,
                'l10n_es_exempt_reason': tax.l10n_es_exempt_reason if tax.l10n_es_type == 'exento' else False,
                'l10n_es_bien_inversion': tax.l10n_es_bien_inversion,
                'is_reverse_charge': tax_data['is_reverse_charge'],
                'tax_scope': tax.tax_scope,
            }

        base_lines_aggregated_values = AccountTax._aggregate_base_lines_tax_details(base_lines, tax_details_info_grouping_function)
        values_per_grouping_key = AccountTax._aggregate_base_lines_aggregated_values(base_lines_aggregated_values)

        tax_details_info = self._build_tax_details_info(values_per_grouping_key.values())
        invoice_info = {
            'DesgloseFactura': {
                **tax_details_info['sujeta_no_sujeta'],
                'S1': tax_details_info['sujeto'],
                'S2': tax_details_info['sujeto_isp'],
            },
        }

        total_amount = 0.0
        total_retention = 0.0
        for values in values_per_grouping_key.values():
            if values['grouping_key'] and values['grouping_key']['l10n_es_type'] == 'retencion':
                total_retention += values['tax_amount']
            else:
                total_amount += values['tax_amount']

        # Aggregate the base lines again (with no grouping) to add the base amount to the total.
        def totals_grouping_function(base_line, tax_data):
            return True

        base_lines_aggregated_values = AccountTax._aggregate_base_lines_tax_details(base_lines, totals_grouping_function)
        values_per_grouping_key = AccountTax._aggregate_base_lines_aggregated_values(base_lines_aggregated_values)

        for values in values_per_grouping_key.values():
            total_amount += values['base_amount']

        return {
            'invoice_info': invoice_info,
            'total_amount': total_amount,
            'total_retention': total_retention,
        }

    @api.model
    def _get_importe_desglose_foreign_partner(self, base_lines, is_refund):
        AccountTax = self.env['account.tax']

        def tax_details_info_grouping_function(base_line, tax_data):
            tax = tax_data['tax']

            return {
                'applied_tax_amount': tax.amount,
                'l10n_es_type': tax.l10n_es_type,
                'l10n_es_exempt_reason': tax.l10n_es_exempt_reason if tax.l10n_es_type == 'exento' else False,
                'l10n_es_bien_inversion': tax.l10n_es_bien_inversion,
                'is_reverse_charge': tax_data['is_reverse_charge'],
                'tax_scope': tax.tax_scope,
            }

        base_lines_aggregated_values = AccountTax._aggregate_base_lines_tax_details(base_lines, tax_details_info_grouping_function)
        values_per_grouping_key = AccountTax._aggregate_base_lines_aggregated_values(base_lines_aggregated_values)

        invoice_info = {}
        for scope, target_key in (('service', 'PrestacionServicios'), ('consu', 'Entrega')):
            service_values_list = [
                values
                for values in values_per_grouping_key.values()
                if values['grouping_key'] and values['grouping_key']['tax_scope'] == scope
            ]
            if service_values_list:
                tax_details_info = self._build_tax_details_info(service_values_list)
                invoice_info.setdefault('DesgloseTipoOperacion', {})[target_key] = {
                    **tax_details_info['sujeta_no_sujeta'],
                    'S1': tax_details_info['sujeto'],
                    'S2': tax_details_info['sujeto_isp'],
                }

        total_amount = 0.0
        total_retention = 0.0
        for values in values_per_grouping_key.values():
            if values['grouping_key'] and values['grouping_key']['l10n_es_type'] == 'retencion':
                total_retention += values['tax_amount']
            else:
                total_amount += values['tax_amount']

        # Aggregate the base lines again (with no grouping) to add the base amount to the total.
        def totals_grouping_function(base_line, tax_data):
            return True

        base_lines_aggregated_values = AccountTax._aggregate_base_lines_tax_details(base_lines, totals_grouping_function)
        values_per_grouping_key = AccountTax._aggregate_base_lines_aggregated_values(base_lines_aggregated_values)

        for values in values_per_grouping_key.values():
            total_amount += values['base_amount']

        return {
            'invoice_info': invoice_info,
            'total_amount': total_amount,
            'total_retention': total_retention,
        }

    def _generate_sale_document_xml(self, values):
        template_name = 'l10n_es_edi_tbai.template_invoice_main' + ('_cancel' if self.is_cancel else '_post')
        xml_str = self.env['ir.qweb']._render(template_name, values)
        xml_doc = cleanup_xml_node(xml_str, remove_blank_nodes=False)

        try:
            xml_doc = self._sign_sale_document(xml_doc)
        except ValueError:
            raise UserError(_('No valid certificate found for this company, TicketBAI file will not be signed.\n'))

        return xml_doc

    def _sign_sale_document(self, xml_root):
        self.ensure_one()

        company = self.company_id
        certificate_sudo = company.sudo().l10n_es_tbai_certificate_id
        if not certificate_sudo:
            raise UserError(_('No certificate found'))

        # Identifiers
        document_id = "Document-" + str(uuid4())
        signature_id = "Signature-" + document_id
        keyinfo_id = "KeyInfo-" + document_id
        sigproperties_id = "SignatureProperties-" + document_id

        # Render digital signature scaffold from QWeb

        e, n = certificate_sudo._get_public_key_numbers_bytes()
        issuer = certificate_sudo._l10n_es_edi_tbai_get_issuer()

        values = {
            'dsig': {
                'document_id': document_id,
                'x509_certificate': base64.encodebytes(base64.b64decode(certificate_sudo._get_der_certificate_bytes())).decode(),
                'public_modulus': n.decode(),
                'public_exponent': e.decode(),
                'iso_now': datetime.now().isoformat(),
                'keyinfo_id': keyinfo_id,
                'signature_id': signature_id,
                'sigproperties_id': sigproperties_id,
                'reference_uri': "Reference-" + document_id,
                'sigpolicy_url': get_key(company.l10n_es_tbai_tax_agency, 'sigpolicy_url'),
                'sigpolicy_digest': get_key(company.l10n_es_tbai_tax_agency, 'sigpolicy_digest'),
                'sigcertif_digest': certificate_sudo._get_fingerprint_bytes(formatting='base64').decode(),
                'x509_issuer_description': issuer,
                'x509_serial_number': int(certificate_sudo.serial_number),
            }
        }
        xml_sig_str = self.env['ir.qweb']._render('l10n_es_edi_tbai.template_digital_signature', values)
        xml_sig = cleanup_xml_signature(xml_sig_str)

        # Complete document with signature template
        xml_root.append(xml_sig)

        # Compute digest values for references
        calculate_references_digests(xml_sig.find("SignedInfo", namespaces=NS_MAP))

        # Sign (writes into SignatureValue)
        signed_info_xml = xml_sig.find('SignedInfo', namespaces=NS_MAP)
        xml_sig.find('SignatureValue', namespaces=NS_MAP).text = certificate_sudo._sign(canonicalize_node(signed_info_xml)).decode()

        return xml_root

    def _generate_purchase_document_xml_bi(self, values):
        sender = self.company_id
        lroe_values = {
            'is_emission': not self.is_cancel,
            'sender': sender,
            'sender_vat': sender.vat[2:] if sender.vat.startswith('ES') else sender.vat,
            'fiscal_year': str(self.date.year),
            'epigrafe': self.env['ir.config_parameter'].sudo().get_param('l10n_es_edi_tbai.epigrafe', '')

        }
        lroe_values.update(values)
        lroe_str = self.env['ir.qweb']._render('l10n_es_edi_tbai.template_LROE_240_main_recibidas', lroe_values)
        lroe_xml = cleanup_xml_node(lroe_str)

        return lroe_xml

    # -------------------------------------------------------------------------
    # SIGNATURE AND QR CODE
    # -------------------------------------------------------------------------

    @api.model
    def _get_tbai_sequence_and_number_purchase(self):
        ''' Get the numbers in the case of vendor bills of Bizkaia'''
        self.ensure_one()
        original_vendor_bill = self.env['account.move'].search([('l10n_es_tbai_post_document_id', '=', self.id)],
                                                               limit=1)
        if original_vendor_bill and self.is_cancel: # Normally it should be is_cancel in this case
            vals = original_vendor_bill.l10n_es_tbai_post_document_id._get_values_from_xml({
                'sequence': './/CabeceraFactura/SerieFactura',
                'number': './/CabeceraFactura/NumFactura',
            })
            if vals['sequence'] and vals['number']:
                return vals['sequence'], vals['number']

        sequence = "TEST" if self.company_id.l10n_es_tbai_test_env else ""
        return sequence, self.name

    @api.model
    def _get_tbai_sequence_and_number(self):
        """Get the TicketBAI sequence a number values for this invoice."""
        self.ensure_one()

        matching = list(re.finditer(r'\d+', self.name))[-1]
        sequence_prefix = self.name[:matching.start()]
        sequence_number = int(matching.group())

        # NOTE non-decimal characters should not appear in the number
        seq_length = self.env['sequence.mixin']._get_sequence_format_param(self.name)[1]['seq_length']
        number = f"{sequence_number:0{seq_length}d}"

        sequence = sequence_prefix.rstrip('/')
        sequence = re.sub(r"[^0-9A-Za-z.\_\-\/]+", "", sequence)  # remove forbidden characters
        sequence = re.sub(r"\s+", " ", sequence)  # no more than one consecutive whitespace allowed
        # NOTE (optional) not recommended to use chars out of ([0123456789ABCDEFGHJKLMNPQRSTUVXYZ.\_\-\/ ])
        sequence += "TEST" if self.company_id.l10n_es_tbai_test_env else ""
        return sequence, number

    def _get_tbai_signature_and_date(self):
        """
        Get the TicketBAI signature and registration date for this document.
        Should only be called for a "post" document (is_cancel==False).
        The registration date is the date the document was registered into the govt's TicketBAI servers.
        """
        self.ensure_one()
        vals = self._get_values_from_xml({
            'signature': './/{http://www.w3.org/2000/09/xmldsig#}SignatureValue',
            'registration_date': './/CabeceraFactura//FechaExpedicionFactura'
        })
        # RFC2045 - Base64 Content-Transfer-Encoding (page 25)
        # Any characters outside of the base64 alphabet are to be ignored in base64-encoded data.
        signature = vals['signature'].replace("\n", "")
        registration_date = datetime.strptime(vals['registration_date'], '%d-%m-%Y')
        return signature, registration_date

    def _get_tbai_id(self):
        """Get the TicketBAI ID (TBAID) as defined in the TicketBAI doc."""
        self.ensure_one()
        if not self._is_in_chain():
            return ''

        signature, registration_date = self._get_tbai_signature_and_date()
        company = self.company_id
        tbai_id_no_crc = '-'.join([
            'TBAI',
            str(company.vat[2:] if company.vat.startswith('ES') else company.vat),
            datetime.strftime(registration_date, '%d%m%y'),
            signature[:13],
            ''  # CRC
        ])
        return tbai_id_no_crc + self._get_crc8(tbai_id_no_crc)

    def _get_tbai_qr(self):
        """Returns the URL for the document's QR code.  We can not use url_encode because it escapes / e.g."""
        self.ensure_one()
        if not self._is_in_chain():
            return ''

        company = self.company_id
        sequence, number = self._get_tbai_sequence_and_number()
        tbai_qr_no_crc = get_key(company.l10n_es_tbai_tax_agency, 'qr_url_', company.l10n_es_tbai_test_env) + '?' + '&'.join([
            'id=' + self._get_tbai_id(),
            's=' + sequence,
            'nf=' + number,
            'i=' + self._get_values_from_xml({'importe': './/ImporteTotalFactura'})['importe']
        ])
        qr_url = tbai_qr_no_crc + '&cr=' + self._get_crc8(tbai_qr_no_crc)
        return qr_url

    def _get_crc8(self, data):
        crc = 0x0
        for c in data:
            crc = CRC8_TABLE[(crc ^ ord(c)) & 0xFF]
        return f'{crc & 0xFF:03d}'

    def _get_values_from_xml(self, xpaths):
        """This function reads values directly from the 'post' XML submitted to the government"""
        res = dict.fromkeys(xpaths, '')
        doc_xml = self._get_xml()
        if doc_xml is None:
            return res
        for key, value in xpaths.items():
            res[key] = doc_xml.find(value).text
        return res

    def _get_xml(self):
        """Returns the XML object representing the document."""
        self.ensure_one()
        doc = self.xml_attachment_id
        if not doc:
            return None
        return etree.fromstring(doc.raw.decode('utf-8'))

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import markupsafe
import re

from odoo import api, fields, models, release
from odoo.tools import LazyTranslate

_lt = LazyTranslate(__name__)

# === TBAI license values ===
L10N_ES_TBAI_LICENSE_DICT = {
    'production': {
        'license_name': _lt('Production license'),  # all agencies
        'license_number': 'TBAIGI5A266A7CCDE1EC',
        'license_nif': 'N0251909H',
        'software_name': 'Odoo SA',
        'software_version': release.version,
    },
    'araba': {
        'license_name': _lt('Test license (Araba)'),
        'license_number': 'TBAIARbjjMClHKH00849',
        'license_nif': 'N0251909H',
        'software_name': 'Odoo SA',
        'software_version': release.version,
    },
    'bizkaia': {
        'license_name': _lt('Test license (Bizkaia)'),
        'license_number': 'TBAIBI00000000PRUEBA',
        'license_nif': 'A99800005',
        'software_name': 'SOFTWARE GARANTE TICKETBAI PRUEBA',
        'software_version': '1.0',
    },
    'gipuzkoa': {
        'license_name': _lt('Test license (Gipuzkoa)'),
        'license_number': 'TBAIGIPRE00000000965',
        'license_nif': 'N0251909H',
        'software_name': 'Odoo SA',
        'software_version': release.version,
    },
}

class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_es_tbai_certificate_id = fields.Many2one(
        string="Certificate (TicketBAI)",
        store=True,
        readonly=False,
        comodel_name='certificate.certificate',
        compute="_compute_l10n_es_tbai_certificate",
    )
    l10n_es_tbai_certificate_ids = fields.One2many(
        comodel_name='certificate.certificate',
        inverse_name='company_id',
        domain=[('scope', '=', 'tbai')],
    )

    # === TBAI config ===
    l10n_es_tbai_tax_agency = fields.Selection(
        string="Tax Agency for TBAI",
        selection=[
            ('araba', "Hacienda Foral de Araba"),  # es-vi (region code)
            ('bizkaia', "Hacienda Foral de Bizkaia"),  # es-bi
            ('gipuzkoa', "Hacienda Foral de Gipuzkoa"),  # es-ss
        ],
    )
    l10n_es_tbai_license_html = fields.Html(
        string="TicketBAI license",
        compute='_compute_l10n_es_tbai_license_html',
    )

    # === TBAI CHAIN HEAD ===
    l10n_es_tbai_chain_sequence_id = fields.Many2one(
        comodel_name='ir.sequence',
        string='TicketBai account.move chain sequence',
        readonly=True,
        copy=False,
    )

    l10n_es_tbai_test_env = fields.Boolean(
        string="TBAI Test Mode",
        help="Use the test environment for TicketBAI",
        default=True,
    )

    l10n_es_tbai_is_enabled = fields.Boolean(compute='_compute_l10n_es_tbai_is_enabled')

    @api.depends('country_id', 'l10n_es_tbai_tax_agency')
    def _compute_l10n_es_tbai_is_enabled(self):
        for company in self:
            company.l10n_es_tbai_is_enabled = company.country_code == 'ES' and company.l10n_es_tbai_tax_agency

    @api.depends('country_id', 'l10n_es_tbai_certificate_ids')
    def _compute_l10n_es_tbai_certificate(self):
        for company in self:
            if company.country_code == 'ES':
                company.l10n_es_tbai_certificate_id = self.env['certificate.certificate'].search(
                    [('company_id', '=', company.id), ('is_valid', '=', True), ('scope', '=', 'tbai')],
                    order='date_end desc',
                    limit=1,
                )
            else:
                company.l10n_es_tbai_certificate_id = False

    @api.depends('country_id', 'l10n_es_tbai_test_env', 'l10n_es_tbai_tax_agency')
    def _compute_l10n_es_tbai_license_html(self):
        for company in self:
            license_dict = company._get_l10n_es_tbai_license_dict()
            if license_dict:
                license_dict.update({
                    'tr_nif': self.env._('Licence NIF'),
                    'tr_number': self.env._('Licence number'),
                    'tr_name': self.env._('Software name'),
                    'tr_version': self.env._('Software version')
                })
                company.l10n_es_tbai_license_html = markupsafe.Markup('''
<strong>{license_name}</strong><br/>
<p>
<strong>{tr_nif}: </strong>{license_nif}<br/>
<strong>{tr_number}: </strong>{license_number}<br/>
<strong>{tr_name}: </strong>{software_name}<br/>
<strong>{tr_version}: </strong>{software_version}<br/>
</p>''').format(**license_dict)
            else:
                company.l10n_es_tbai_license_html = markupsafe.Markup('''
<strong>{tr_no_license}</strong>''').format(tr_no_license=self.env._('TicketBAI is not configured'))

    def _get_l10n_es_tbai_license_dict(self):
        self.ensure_one()
        if self.l10n_es_tbai_is_enabled:
            if self.l10n_es_tbai_test_env:  # test env: each agency has its test license
                license_key = self.l10n_es_tbai_tax_agency
            else:  # production env: only one license
                license_key = 'production'
            license = L10N_ES_TBAI_LICENSE_DICT[license_key]
            return dict(license, license_name=str(license["license_name"]))  # force translation
        else:
            return {}

    def _get_l10n_es_tbai_next_chain_index(self):
        if not self.l10n_es_tbai_chain_sequence_id:
            self_sudo = self.sudo()
            self_sudo.l10n_es_tbai_chain_sequence_id = self_sudo.env['ir.sequence'].create({
                'name': f'TicketBAI account move sequence for {self.name} (id: {self.id})',
                'code': f'l10n_es.edi.tbai.account.move.{self.id}',
                'implementation': 'no_gap',
                'company_id': self.id,
            })
        return self.l10n_es_tbai_chain_sequence_id.next_by_id()

    def _get_l10n_es_tbai_last_chained_document(self):
        """
        Returns the last tbai document posted to this company's chain.
        That tbai document may have been received by the govt or not (eg. in case of a timeout).
        Only upon confirmed reception/refusal of that tbai document can another one be posted.
        """
        domain = [
            ('chain_index', '!=', 0),
            ('company_id', '=', self.id)
        ]
        return self.env['l10n_es_edi_tbai.document'].search(domain, limit=1, order='chain_index desc')

    def _l10n_es_freelancer(self):
        self.ensure_one()
        return self.vat and re.fullmatch(r"(ES)?(\d{8}[A-Z]|[X-Z].*)", self.vat) or False

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_es_tbai_certificate_ids = fields.One2many(related='company_id.l10n_es_tbai_certificate_ids', readonly=False)
    l10n_es_tbai_tax_agency = fields.Selection(related='company_id.l10n_es_tbai_tax_agency', readonly=False)
    l10n_es_tbai_test_env = fields.Boolean(related='company_id.l10n_es_tbai_test_env', readonly=False)

```

## File: models\xml_utils.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hashlib
import re
from base64 import b64encode

from lxml import etree
from odoo.tools.xml_utils import cleanup_xml_node


# Utility Methods for Basque Country's TicketBAI XML-related stuff.

NS_MAP = {'': 'http://www.w3.org/2000/09/xmldsig#'}  # default namespace matches signature's `ds:``


def canonicalize_node(node):
    """
    Returns the canonical (C14N 1.0, without comments, non exclusive) representation of node.
    Speficied in: https://www.w3.org/TR/2001/REC-xml-c14n-20010315
    Required for computing digests and signatures.
    Returns an UTF-8 encoded bytes string.
    """
    node = etree.fromstring(node) if isinstance(node, str) else node
    return etree.tostring(node, method='c14n', with_comments=False, exclusive=False)


def cleanup_xml_signature(xml_sig):
    """
    Cleanups the content of the provided string representation of an XML signature.
    In addition, removes all line feeds for the ds:Object element.
    Turns self-closing tags into regular tags (with an empty string content)
    as the former may not be supported by some signature validation implementations.
    Returns an etree._Element
    """
    sig_elem = cleanup_xml_node(xml_sig, remove_blank_nodes=False, indent_level=-1)
    etree.indent(sig_elem, space='')  # removes indentation
    for elem in sig_elem.find('Object', namespaces=NS_MAP).iter():
        if elem.text == '\n':
            elem.text = ''  # keeps the signature in one line, prevents self-closing tags
        elem.tail = ''  # removes line feed and whitespace after the tag
    return sig_elem


def get_uri(uri, reference, base_uri):
    """
    Returns the content within `reference` that is identified by `uri`.
    Canonicalization is used to convert node reference to an octet stream.
    - The base_uri points to the whole document tree, without the signature
    https://www.w3.org/TR/xmldsig-core/#sec-EnvelopedSignature

    - URIs starting with # are same-document references
    https://www.w3.org/TR/xmldsig-core/#sec-URI

    Returns an UTF-8 encoded bytes string.
    """
    node = reference.getroottree()
    if uri == base_uri:
        # Empty URI: whole document, without signature
        return canonicalize_node(
            re.sub(
                r'^[^\n]*<ds:Signature.*<\/ds:Signature>', r'',
                etree.tostring(node, encoding='unicode'),
                flags=re.DOTALL | re.MULTILINE)
        )

    if uri.startswith('#'):
        query = '//*[@*[local-name() = "Id" ]=$uri]'  # case-sensitive 'Id'
        results = node.xpath(query, uri=uri.lstrip('#'))
        if len(results) == 1:
            return canonicalize_node(results[0])
        if len(results) > 1:
            raise Exception("Ambiguous reference URI {} resolved to {} nodes".format(
                uri, len(results)))

    raise Exception(f"URI {uri!r} not found")


def calculate_references_digests(node, base_uri=''):
    """
    Processes the references from node and computes their digest values as specified in
    https://www.w3.org/TR/xmldsig-core/#sec-DigestMethod
    https://www.w3.org/TR/xmldsig-core/#sec-DigestValue
    """
    for reference in node.findall('Reference', namespaces=NS_MAP):
        ref_node = get_uri(reference.get('URI', ''), reference, base_uri)
        hash_digest = hashlib.new('sha256', ref_node).digest()
        reference.find('DigestValue', namespaces=NS_MAP).text = b64encode(hash_digest)

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import account_move_line
from . import certificate
from . import account_move_send
from . import l10n_es_edi_tbai_agencies
from . import l10n_es_edi_tbai_document
from . import res_company
from . import res_config_settings
from . import xml_utils

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_es_edi_tbai_document_readonly,access_l10n_es_edi_tbai_document,l10n_es_edi_tbai.model_l10n_es_edi_tbai_document,base.group_user,1,0,0,0

```

## File: security\l10n_es_edi_tbai_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="tbai_document_comp_rule" model="ir.rule">
        <field name="name">TicketBAI Document multi-company</field>
        <field name="model_id" ref="model_l10n_es_edi_tbai_document"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>
</odoo>

```

## File: views\account_move_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_move_form_inherit_l10n_es_edi_tbai" model="ir.ui.view">
            <field name="name">account.move.form.inherit.l10n_es_edi_tbai</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_move_form"/>
            <field name="arch" type="xml">
                <xpath expr="//header" position="inside">
                    <field name="l10n_es_tbai_is_required" invisible="1"/>
                    <field name="l10n_es_tbai_post_document_id" invisible="1"/>
                    <button
                        string="Send Bill to TicketBAI"
                        name="l10n_es_tbai_send_bill"
                        type="object"
                        invisible="not l10n_es_tbai_is_required or move_type not in ('in_invoice', 'in_refund') or state != 'posted' or l10n_es_tbai_state != 'to_send'"
                    />
                    <button
                        string="TicketBAI Cancel"
                        name="l10n_es_tbai_cancel"
                        type="object"
                        invisible="l10n_es_tbai_state != 'sent'"
                    />
                </xpath>
                <xpath expr="//group[@id='header_right_group']" position='inside'>
                    <field
                        name="l10n_es_tbai_refund_reason"
                        invisible="move_type not in ('in_refund', 'out_refund') or not l10n_es_tbai_is_required"
                        readonly="state != 'draft'"
                    />
                </xpath>
                <xpath expr="//page[@id='other_tab_entry']" position='after'>
                    <page
                        id="ticketbai_tab"
                        string="TicketBAI"
                        invisible="not l10n_es_tbai_is_required or not l10n_es_tbai_post_document_id"
                    >
                        <group>
                            <field name="l10n_es_tbai_state"/>
                            <field name="l10n_es_tbai_chain_index" groups="base.group_no_one"/>
                            <field name="l10n_es_tbai_post_file_name" invisible="1"/>
                            <field name="l10n_es_tbai_post_file" widget="binary" filename="l10n_es_tbai_post_file_name"/>
                            <field name="l10n_es_tbai_cancel_file_name" invisible="1"/>
                            <field name="l10n_es_tbai_cancel_file" widget="binary" filename="l10n_es_tbai_cancel_file_name"/>
                            <field name="l10n_es_tbai_reversed_ids" invisible="move_type != 'in_refund'" widget="many2many_tags"/>
                            <field name="reversed_entry_id" invisible="move_type != 'in_refund'"/>
                        </group>
                    </page>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\l10n_es_edi_tbai_certificate_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>

        <record id="certificate_certificate_view_search" model="ir.ui.view">
            <field name="name">certificate_certificate_view_search.inherit.l10n_es_edi_tbai</field>
            <field name="model">certificate.certificate</field>
            <field name="inherit_id" ref="certificate.certificate_certificate_view_search"/>
            <field name="arch" type="xml">
                <filter name="scope_general" position="after">
                    <filter string="TBAI" name="scope_tbai" domain="[('scope','=','tbai')]" help="TBAI certificates"/>
                </filter>
            </field>
        </record>
    
        <record id="l10n_es_edi_tbai_certificate_action" model="ir.actions.act_window">
            <field name="name">Certificates for EDI TicketBAI invoices on Spain</field>
            <field name="res_model">certificate.certificate</field>
            <field name="view_mode">list,form</field>
            <field name="context">{'scope': 'tbai', 'search_default_scope_tbai': 1}</field>
            <field name="help" type="html">
                <p class="oe_view_nocontent_create">Create the first certificate</p>
            </field>
        </record>
    
        <record id="certificate_certificate_view_form" model="ir.ui.view">
            <field name="name">certificate_certificate_view_form.inherit.l10n_es_edi_tbai</field>
            <field name="model">certificate.certificate</field>
            <field name="inherit_id" ref="certificate.certificate_certificate_view_form"/>
            <field name="arch" type="xml">
                <field name="scope" position="attributes">
                    <attribute name="invisible">False</attribute>
                </field>
            </field>
        </record>

        <menuitem id="menu_l10n_es_edi_tbai_root"
                  name="Spain TicketBAI"
                  sequence="110"
                  groups="account.group_account_manager"
                  parent="account.menu_finance_configuration">
            <menuitem id="menu_l10n_es_edi_tbai_certificates"
                      name="Certificates"
                      action="l10n_es_edi_tbai_certificate_action"
                      sequence="100"
                      groups="account.group_account_manager"/>
        </menuitem>

    </data>
</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <template id="l10n_es_tbai_external_layout_standard" inherit_id="account.report_invoice_document">
        <xpath expr="//div[@id='qrcode']" position="after">
            <div name="l10n_es_tbai_qrcode" t-if="o.l10n_es_tbai_state == 'sent'" style="page-break-inside: avoid">
                <div t-out="o.l10n_es_tbai_post_document_id._get_tbai_id()"/>
                <img
                    t-att-src="'/report/barcode/?barcode_type=%s&amp;value=%s&amp;width=%s&amp;height=%s&amp;barLevel=%s'%('QR', quote_plus(o.l10n_es_tbai_post_document_id._get_tbai_qr()), 125, 125, 'M')"/>

                <!-- NOTE: Sizes assume a 90 dpi resolution to meet requirements (between 30 and 40 mm) -->
            </div>
        </xpath>
    </template>
</odoo>

```

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="res_company_form_l10n_es_edi_tbai" model="ir.ui.view">
        <field name="name">res.company.form</field>
        <field name="model">res.company</field>
        <field name="inherit_id" ref="account.view_company_form"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='general_info']/group/group[last()]" position="after">
                <group>
                    <field name="l10n_es_tbai_license_html" type="object"
                        invisible="country_code != 'ES'"/>
                </group>
            </xpath>
        </field>
    </record>

    <!-- TicketBAI specifies there needs to be a menu link to display the license information -->
    <menuitem id="menu_l10n_es_edi_tbai_license"
        name="Licenses"
        action="base.action_res_company_form"
        sequence="90"
        parent="menu_l10n_es_edi_tbai_root"
        groups="account.group_account_manager">
    </menuitem>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.l10n.es</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="l10n_es.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//block[@name='spain_localization']" position="attributes">
                <attribute name="invisible">country_code != 'ES'</attribute>
            </xpath>
            <xpath expr="//block[@name='spain_localization']" position="inside">
                <!-- Invisible fields -->
                <field name="l10n_es_tbai_certificate_ids" invisible="1"/>
                <setting string="Registro de Libros connection TicketBAI" company_dependent="1">
                    <div class="content-group">
                        <div class="mt16">
                            <label for="l10n_es_tbai_tax_agency" class="o_light_label"/>
                            <field name="l10n_es_tbai_tax_agency"/>
                            <div class="text-muted" invisible="l10n_es_tbai_tax_agency">
                                No tax agency selected: TicketBAI not activated.
                            </div>
                            <div class="text-muted" invisible="not l10n_es_tbai_tax_agency">
                                Tax agency selected: invoices will be sent by TicketBAI.
                            </div>
                            <br/>
                            <div class="o_row">
                                <label for="l10n_es_tbai_test_env" class="o_light_label"/>
                                <field name="l10n_es_tbai_test_env"/>
                            </div>
                            <div class="text-muted" invisible="not l10n_es_tbai_test_env">
                                Test mode: EDI data is sent to separate test servers and is not considered official.
                            </div>
                            <div class="text-muted" invisible="l10n_es_tbai_test_env">
                                Production mode: EDI data is sent to the official agency servers.
                            </div>
                            <br/>
                            <div>
                                <button name="%(l10n_es_edi_tbai_certificate_action)d" type="action" class="oe_link">Manage certificates (TicketBAI)</button>
                            </div>
                        </div>
                    </div>
                </setting>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizards\account_move_reversal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models, api
from odoo.addons.l10n_es_edi_tbai.models.account_move import TBAI_REFUND_REASONS
from odoo.exceptions import UserError


class AccountMoveReversal(models.TransientModel):
    _inherit = 'account.move.reversal'

    l10n_es_tbai_is_required = fields.Boolean(
        compute="_compute_l10n_es_tbai_is_required", readonly=True,
        string="Is TicketBai required for this reversal",
    )

    l10n_es_tbai_refund_reason = fields.Selection(
        selection=TBAI_REFUND_REASONS,
        string="Invoice Refund Reason Code (TicketBai)",
        help="BOE-A-1992-28740. Ley 37/1992, de 28 de diciembre, del Impuesto sobre el "
        "Valor Añadido. Artículo 80. Modificación de la base imponible.",
    )

    @api.depends('move_ids')
    def _compute_l10n_es_tbai_is_required(self):
        for wizard in self:
            moves_tbai_required = set(m.l10n_es_tbai_is_required for m in wizard.move_ids)
            if len(moves_tbai_required) > 1:
                raise UserError("Reversals mixing invoices with and without TicketBAI are not allowed.")
            wizard.l10n_es_tbai_is_required = moves_tbai_required.pop()

    def _prepare_default_reversal(self, move):
        # OVERRIDE
        values = super()._prepare_default_reversal(move)
        if move.l10n_es_tbai_is_required:
            values.update({
                'l10n_es_tbai_refund_reason': self.l10n_es_tbai_refund_reason,
            })
        return values

```

## File: wizards\account_move_reversal_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_account_move_reversal" model="ir.ui.view">
        <field name="name">account.move.reversal.form.inherit.l10n_es_edi_tbai</field>
        <field name="model">account.move.reversal</field>
        <field name="inherit_id" ref="account.view_account_move_reversal"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='journal_id']" position="after">
                <field name="l10n_es_tbai_is_required" invisible="1"/>
                <field invisible="not l10n_es_tbai_is_required" name="l10n_es_tbai_refund_reason" widget="selection"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizards\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_move_reversal

```

