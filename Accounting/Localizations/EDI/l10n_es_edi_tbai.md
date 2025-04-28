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
    'version': '1.0',
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
        'l10n_es_edi_sii',
    ],
    'data': [
        'data/account_edi_data.xml',
        'data/template_invoice.xml',
        'data/template_LROE_bizkaia.xml',

        'views/account_move_view.xml',
        'views/report_invoice.xml',
        'views/res_config_settings_views.xml',
        'views/res_company_views.xml',

        'wizards/account_move_reversal_views.xml',
    ],
    'demo': [
        'demo/demo_res_partner.xml',
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_edi_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="edi_es_tbai" model="account.edi.format">
            <field name="name">TicketBAI (ES)</field>
            <field name="code">es_tbai</field>
        </record>
    </data>
</odoo>

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
            <Sujetos t-if="is_emission">
                <t t-call="l10n_es_edi_tbai.template_invoice_sujetos"/>
            </Sujetos>
            <Factura t-if="is_emission">
                <t t-call="l10n_es_edi_tbai.template_invoice_factura"/>
            </Factura>
            <IDFactura t-if="not is_emission">
                <t t-call="l10n_es_edi_tbai.template_invoice_sujetos"/>
                <t t-call="l10n_es_edi_tbai.template_invoice_factura"/>
            </IDFactura>
            <HuellaTBAI>
                <EncadenamientoFacturaAnterior t-if="chain_prev_invoice">
                    <t t-set="seq_and_num" t-value="chain_prev_invoice._get_l10n_es_tbai_sequence_and_number()"/>
                    <SerieFacturaAnterior t-out="seq_and_num[0]"/>
                    <NumFacturaAnterior t-out="seq_and_num[1]"/>
                    <t t-set="sig_and_date" t-value="chain_prev_invoice._get_l10n_es_tbai_signature_and_date()"/>
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
            <t t-set="is_simplified" t-value="invoice.l10n_es_is_simplified"/>
            <CabeceraFactura>
                <t t-set="seq_and_num" t-value="invoice._get_l10n_es_tbai_sequence_and_number()"/>
                <SerieFactura t-out="seq_and_num[0]"/>
                <NumFactura t-out="seq_and_num[1]"/>
                <t t-if="is_emission">
                    <FechaExpedicionFactura t-out="format_date(datetime_now)"/>
                    <HoraExpedicionFactura t-out="format_time(datetime_now)"/>
                    <FacturaSimplificada t-out="'S' if is_simplified else 'N'"/>
                </t>
                <FechaExpedicionFactura t-else="" t-out="format_date(invoice._get_l10n_es_tbai_signature_and_date()[1])"/>
                <t t-if="is_refund">
                    <FacturaEmitidaSustitucionSimplificada t-out="'S' if (is_simplified and recipient) else 'N'"/>
                    <FacturaRectificativa>
                        <Codigo t-out="credit_note_code"/>
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
                            <t t-set="seq_and_num" t-value="credit_note_invoice._get_l10n_es_tbai_sequence_and_number()"/>
                            <SerieFactura t-out="seq_and_num[0]"/>
                            <NumFactura t-out="seq_and_num[1]"/>
                            <FechaExpedicionFactura t-out="format_date(credit_note_invoice.l10n_es_tbai_post_xml and credit_note_invoice._get_l10n_es_tbai_signature_and_date()[1] or credit_note_invoice.invoice_date)"/>
                        </IDFacturaRectificadaSustituida>
                    </FacturasRectificadasSustituidas>
                </t>
            </CabeceraFactura>
            <DatosFactura t-if="is_emission">
                <FechaOperacion t-if="invoice.delivery_date and format_date(invoice.invoice_date) != format_date(invoice.delivery_date)" t-out="format_date(invoice.delivery_date)"/>
                <DescripcionFactura t-out="invoice.invoice_origin and invoice.invoice_origin[:250] or 'manual'"/>
                <DetallesFactura>
                    <IDDetalleFactura t-foreach="invoice_lines" t-as="line_values">
                        <t t-set="line" t-value="line_values['line']"/>
                        <DescripcionDetalle t-out="line_values['description']"/>
                        <Cantidad t-out="format_float(line.quantity, precision_digits=8)"/>
                        <ImporteUnitario t-out="format_float(line_values['unit_price'], precision_digits=8)"/>
                        <Descuento t-out="format_float(line_values['discount'], precision_digits=8)"/>
                        <ImporteTotal t-out="format_float(line_values['total'], precision_digits=8)"/>
                    </IDDetalleFactura>
                </DetallesFactura>
                <ImporteTotalFactura t-out="format_float(amount_total)"/>
                <RetencionSoportada t-if="amount_retention != 0.0" t-out="format_float(amount_retention)"/>
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
                t-if="is_emission"
                t-call="l10n_es_edi_tbai.template_LROE_240_inner"/>
            <lrpjfecsgap:LROEPJ240FacturasEmitidasConSGAnulacionPeticion
                xmlns:lrpjfecsgap="https://www.batuz.eus/fitxategiak/batuz/LROE/esquemas/LROE_PJ_240_1_1_FacturasEmitidas_ConSG_AnulacionPeticion_V1_0_0.xsd"
                t-else=""
                t-call="l10n_es_edi_tbai.template_LROE_240_inner"/>
        </template>

        <template id="template_LROE_240_inner">
            <Cabecera>
                <Modelo>240</Modelo>
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
            <FacturasEmitidas>
                <FacturaEmitida t-foreach="tbai_b64_list" t-as="tbai_b64">
                    <TicketBai t-if="is_emission" t-out="tbai_b64"/>
                    <AnulacionTicketBai t-else="" t-out="tbai_b64"/>
                </FacturaEmitida>
            </FacturasEmitidas>
        </template>


        <template id="template_LROE_240_main_recibidas">
            <lrpjframp:LROEPJ240FacturasRecibidasAltaModifPeticion
                xmlns:lrpjframp="https://www.batuz.eus/fitxategiak/batuz/LROE/esquemas/LROE_PJ_240_2_FacturasRecibidas_AltaModifPeticion_V1_0_1.xsd"
                t-if="is_emission"
                t-call="l10n_es_edi_tbai.template_LROE_240_inner_recibidas"/>
            <lrpjfrap:LROEPJ240FacturasRecibidasAnulacionPeticion
                xmlns:lrpjfrap="https://www.batuz.eus/fitxategiak/batuz/LROE/esquemas/LROE_PJ_240_2_FacturasRecibidas_AnulacionPeticion_V1_0_0.xsd"
                t-else=""
                t-call="l10n_es_edi_tbai.template_LROE_240_inner_recibidas"/>
        </template>

        <template id="template_LROE_240_inner_recibidas">
            <Cabecera>
                <Modelo>240</Modelo>
                <Capitulo>2</Capitulo>
                <Operacion t-out="'A00' if is_emission else 'AN0'"/>
                <Version>1.0</Version>
                <Ejercicio t-out="fiscal_year"/>
                <ObligadoTributario>
                    <NIF t-out="sender_vat"/>
                    <ApellidosNombreRazonSocial t-out="sender.name"/>
                </ObligadoTributario>
            </Cabecera>
            <FacturasRecibidas>
                <FacturaRecibida>
                    <t t-if="not is_emission"> <!-- cancel case -->
                        <IDRecibida>
                            <t t-set="seq_and_num" t-value="invoice._get_l10n_es_tbai_sequence_and_number()"/>
                            <SerieFactura t-out="seq_and_num[0]"/>
                            <NumFactura t-out="seq_and_num[1]"/>
                            <FechaExpedicionFactura t-out="format_date(invoice.invoice_date)"/>
                            <EmisorFacturaRecibida>
                                <NIF t-if="recipient.get('nif')" t-out="recipient['nif']"/>
                                <IDOtro t-else="">
                                    <CodigoPais t-if="recipient.get('alt_id_country')" t-out="recipient['alt_id_country']"/>
                                    <IDType t-out="recipient['alt_id_type']"/>
                                    <ID t-out="recipient['alt_id_number']"/>
                                </IDOtro>
                            </EmisorFacturaRecibida>
                        </IDRecibida>
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
                            <t t-set="seq_and_num" t-value="invoice._get_l10n_es_tbai_sequence_and_number()"/>
                            <SerieFactura t-out="seq_and_num[0]"/>
                            <NumFactura t-out="seq_and_num[1]"/>
                            <FechaExpedicionFactura t-out="format_date(invoice.invoice_date)"/>
                            <FechaRecepcion t-out="format_date(invoice.date)"/>
                            <TipoFactura t-out="tipofactura"/>
                            <t t-if="is_refund">
                                <FacturaRectificativa>
                                    <Codigo t-out="credit_note_code"/>
                                    <Tipo>I</Tipo>
                                </FacturaRectificativa>
                                <FacturasRectificadasSustituidas t-if="credit_note_invoice">
                                    <IDFacturaRectificadaSustituida>
                                        <t t-set="seq_and_num" t-value="credit_note_invoice._get_l10n_es_tbai_sequence_and_number()"/>
                                        <SerieFactura t-out="seq_and_num[0]"/>
                                        <NumFactura t-out="seq_and_num[1]"/>
                                        <FechaExpedicionFactura t-out="format_date(credit_note_invoice.invoice_date)"/>
                                    </IDFacturaRectificadaSustituida>
                                </FacturasRectificadasSustituidas>
                            </t>
                        </CabeceraFactura>
                        <DatosFactura>
                            <DescripcionOperacion t-out="invoice.ref"/>

                            <Claves>
                                <IDClave t-foreach="regime_key" t-as="key">
                                    <ClaveRegimenIvaOpTrascendencia t-out="key"/>
                                </IDClave>
                            </Claves>
                            <ImporteTotalFactura t-out="format_float(amount_total)"/>
                        </DatosFactura>
                        <IVA>
                            <DetalleIVA t-foreach="iva_values" t-as="tax">
                                <CompraBienesCorrientesGastosBienesInversion t-out="tax['code']"/>
                                <InversionSujetoPasivo t-out="'N' if tax['rec'].l10n_es_type != 'sujeto_isp' else 'S'"/>
                                <BaseImponible t-out="format_float(tax['base'])"/>
                                <TipoImpositivo t-out="tax['rec'].amount"/>
                                <CuotaIVASoportada t-out="format_float(tax['tax'])"/>
                                <CuotaIVADeducible t-out="format_float(tax['tax']) if tax['rec'].l10n_es_type != 'no_deducible' else '0.00'"/>
                            </DetalleIVA>
                        </IVA>
                    </t>
                </FacturaRecibida>
            </FacturasRecibidas>
        </template>
    </data>
</odoo>

```

## File: models\account_edi_document.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models

class AccountEdiDocument(models.Model):
    _inherit = 'account.edi.document'

    def _prepare_jobs(self):
        """
        If there is a job to process that may already be part of the chain (posted invoice that timeout'ed),
        Re-places it at the beginning of the list.
        """
        # EXTENDS account_edi
        jobs = super()._prepare_jobs()
        if len(jobs) > 1:
            move_first_index = 0
            for index, job in enumerate(jobs):
                documents = job['documents']
                if any(d.edi_format_id.code == 'es_tbai' and d.state == 'to_send' and d.move_id.l10n_es_tbai_chain_index for d in documents):
                    move_first_index = index
                    break
            jobs = [jobs[move_first_index]] + jobs[:move_first_index] + jobs[move_first_index + 1:]

        return jobs

```

## File: models\account_edi_format.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import gzip
import json
from base64 import b64encode
from datetime import datetime
from re import sub as regex_sub
from uuid import uuid4
from markupsafe import Markup, escape

import requests
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.x509.oid import NameOID
from lxml import etree
from pytz import timezone
from requests.exceptions import RequestException

from odoo import _, models, release
from odoo.addons.l10n_es_edi_sii.models.account_edi_format import PatchedHTTPAdapter
from odoo.addons.l10n_es_edi_tbai.models.l10n_es_edi_tbai_agencies import get_key
from odoo.addons.l10n_es_edi_tbai.models.xml_utils import (
    NS_MAP, bytes_as_block, calculate_references_digests,
    cleanup_xml_signature, fill_signature, int_as_bytes)
from odoo.exceptions import UserError, ValidationError
from odoo.tools import get_lang
from odoo.tools.float_utils import float_repr, float_round
from odoo.tools.xml_utils import cleanup_xml_node, validate_xml_from_attachment


class AccountEdiFormat(models.Model):
    _inherit = 'account.edi.format'

    # -------------------------------------------------------------------------
    # OVERRIDES & EXTENSIONS
    # -------------------------------------------------------------------------

    def _needs_web_services(self):
        # EXTENDS account_edi
        return self.code == 'es_tbai' or super()._needs_web_services()

    def _is_enabled_by_default_on_journal(self, journal):
        """ Disable SII by default on a new journal when tbai is installed"""
        if self.code != 'es_sii':
            return super()._is_enabled_by_default_on_journal(journal)
        return False

    def _is_compatible_with_journal(self, journal):
        # EXTENDS account_edi
        if self.code != 'es_tbai':
            return super()._is_compatible_with_journal(journal)

        return journal.country_code == 'ES' and journal.type in ('sale', 'purchase')

    def _get_move_applicability(self, move):
        # EXTENDS account_edi
        self.ensure_one()
        if self.code != 'es_tbai' or move.country_code != 'ES' or not move.l10n_es_tbai_is_required:
            return super()._get_move_applicability(move)

        return {
            'post': self._l10n_es_tbai_post_invoice_edi,
            'cancel': self._l10n_es_tbai_cancel_invoice_edi,
            'edi_content': self._l10n_es_tbai_get_invoice_content_edi,
        }

    def _check_move_configuration(self, invoice):
        # EXTENDS account_edi
        errors = super()._check_move_configuration(invoice)

        if self.code != 'es_tbai' or invoice.country_code != 'ES':
            return errors

        if invoice.is_purchase_document() and not invoice.ref:
            errors.append(_("You need to fill in the Reference field as the invoice number from your vendor."))

        # Ensure a certificate is available.
        if not invoice.company_id.l10n_es_edi_certificate_id:
            errors.append(_("Please configure the certificate for TicketBAI/SII."))

        # Ensure a tax agency is available.
        if not invoice.company_id.mapped('l10n_es_tbai_tax_agency')[0]:
            errors.append(_("Please specify a tax agency on your company for TicketBAI."))

        # Ensure a vat is available.
        if not invoice.company_id.vat:
            errors.append(_("Please configure the Tax ID on your company for TicketBAI."))

        # Check the refund reason
        if invoice.move_type == 'out_refund':
            if not invoice.l10n_es_tbai_refund_reason:
                raise ValidationError(_('Refund reason must be specified (TicketBAI)'))
            if invoice.l10n_es_is_simplified:
                if invoice.l10n_es_tbai_refund_reason != 'R5':
                    raise ValidationError(_('Refund reason must be R5 for simplified invoices (TicketBAI)'))
            else:
                if invoice.l10n_es_tbai_refund_reason == 'R5':
                    raise ValidationError(_('Refund reason cannot be R5 for non-simplified invoices (TicketBAI)'))

        return errors

    def _l10n_es_tbai_refunded_invoices(self, invoice):
        return invoice.reversed_entry_id

    def _l10n_es_tbai_post_invoice_edi(self, invoice):
        # EXTENDS account_edi
        if self.code != 'es_tbai':
            return super()._post_invoice_edi(invoice)

        if invoice.is_purchase_document():
            inv_xml = False # For Ticketbai Batuz vendor bills, we get the values later as it does not need chaining, ...

        else:
            # Chain integrity check: chain head must have been REALLY posted (not timeout'ed)
            # - If called from a cron, then the re-ordering of jobs should prevent this from triggering
            # - If called manually, then the user will see this error pop up when it triggers
            chain_head = invoice.company_id._get_l10n_es_tbai_last_posted_invoice()
            error_msg = ''
            if chain_head and chain_head != invoice and not chain_head._l10n_es_tbai_is_in_chain():
                error_msg = _("TicketBAI: Cannot post invoice while chain head (%s) has not been posted", chain_head.name)
            if invoice.move_type == 'out_refund':
                refunded_invoices = self._l10n_es_tbai_refunded_invoices(invoice)
                if not refunded_invoices:
                    error_msg = _("TicketBAI: Cannot post a refund without source documents")
                else:
                    invalid_refunds = refunded_invoices.filtered(lambda inv:
                                                                 not inv._l10n_es_tbai_is_in_chain()
                                                                 and inv.edi_document_ids.filtered(lambda d: d.edi_format_id.code == 'es_tbai')  # avoid imported ones
                                                                 )
                    if invalid_refunds:
                        error_msg = _(
                            "TicketBAI: Cannot post a reversal move if its source documents (%s) have not been posted",
                            ', '.join(invalid_refunds.mapped('name'))
                        )

            # Tax configuration check: In case of foreign customer we need the tax scope to be set
            com_partner = invoice.commercial_partner_id
            if (com_partner.country_id.code not in ('ES', False) or (com_partner.vat or '').startswith("ESN")) and\
                    invoice.line_ids.tax_ids.filtered(lambda t: not t.tax_scope):
                error_msg = _(
                    "In case of a foreign customer, you need to configure the tax scope on taxes:\n%s",
                    "\n".join(invoice.line_ids.tax_ids.mapped('name'))
                )

            if error_msg:
                return {
                    invoice: {
                        'error': error_msg,
                        'blocking_level': 'error',
                    }
                }

            # Generate the XML values.
            inv_dict = self._get_l10n_es_tbai_invoice_xml(invoice)
            if 'error' in inv_dict[invoice]:
                return inv_dict  # XSD validation failed, return result dict

            # Store the XML as attachment to ensure it is never lost (even in case of timeout error)
            inv_xml = inv_dict[invoice]['xml_file']
            invoice._update_l10n_es_tbai_submitted_xml(xml_doc=inv_xml, cancel=False)

            # Assign unique 'chain index' from dedicated sequence
            if not invoice.l10n_es_tbai_chain_index:
                invoice.l10n_es_tbai_chain_index = invoice.company_id._get_l10n_es_tbai_next_chain_index()

        # Call the web service and get response
        res = self._l10n_es_tbai_post_to_web_service(invoice, inv_xml)

        # SUCCESS
        if res[invoice].get('success'):
            # Create attachment
            attachment = self.env['ir.attachment'].create({
                'name': invoice.name + '_post.xml',
                'datas': invoice.l10n_es_tbai_post_xml,
                'mimetype': 'application/xml',
                'res_id': invoice.id,
                'res_model': 'account.move',
            })

            # Post attachment to chatter and save it as EDI document
            test_suffix = '(test mode)' if invoice.company_id.l10n_es_edi_test_env else ''
            invoice.with_context(no_new_invoice=True).message_post(
                body=Markup("<pre>TicketBAI: posted emission XML {test_suffix}\n{message}</pre>").format(
                    test_suffix=test_suffix, message=res[invoice]['message']
                ),
                attachment_ids=[attachment.id],
            )
            res[invoice]['attachment'] = attachment

        # FAILURE
        # NOTE: 'warning' means timeout so absolutely keep the XML and chain index
        elif res[invoice].get('blocking_level') == 'error':
            invoice._update_l10n_es_tbai_submitted_xml(xml_doc=None, cancel=False)  # deletes XML
            # delete index (avoids re-trying same XML and chaining off of it)
            invoice.l10n_es_tbai_chain_index = False

        return res

    def _l10n_es_tbai_cancel_invoice_edi(self, invoice):
        # EXTENDS account_edi
        if self.code != 'es_tbai':
            return super()._cancel_invoice_edi(invoice)

        if invoice.is_purchase_document():
            cancel_xml = False # Batuz specific
        else:
            # Generate the XML values.
            cancel_dict = self._get_l10n_es_tbai_invoice_xml(invoice, cancel=True)
            if 'error' in cancel_dict[invoice]:
                return cancel_dict  # XSD validation failed, return result dict

            # Store the XML as attachment to ensure it is never lost (even in case of timeout error)
            cancel_xml = cancel_dict[invoice]['xml_file']
            invoice._update_l10n_es_tbai_submitted_xml(xml_doc=cancel_xml, cancel=True)

        # Call the web service and get response
        res = self._l10n_es_tbai_post_to_web_service(invoice, cancel_xml, cancel=True)

        # SUCCESS
        if res[invoice].get('success'):
            # Create attachment
            attachment = self.env['ir.attachment'].create({
                'name': invoice.name + '_cancel.xml',
                'datas': invoice.l10n_es_tbai_cancel_xml,
                'mimetype': 'application/xml',
                'res_id': invoice.id,
                'res_model': 'account.move',
            })

            # Post attachment to chatter
            test_suffix = '(test mode)' if invoice.company_id.l10n_es_edi_test_env else ''
            invoice.with_context(no_new_invoice=True).message_post(
                body=Markup("<pre>TicketBAI: posted cancellation XML {test_suffix}\n{message}</pre>").format(
                    test_suffix=test_suffix, message=res[invoice]['message']
                ),
                attachment_ids=[attachment.id],
            )

        # FAILURE
        # NOTE: 'warning' means timeout so absolutely keep the XML and chain index
        elif res[invoice].get('blocking_level') == 'error':
            invoice._update_l10n_es_tbai_submitted_xml(xml_doc=None, cancel=True)  # will need to be re-created

        return res

    # -------------------------------------------------------------------------
    # XML DOCUMENT
    # -------------------------------------------------------------------------

    def _l10n_es_tbai_validate_xml_with_xsd(self, xml_doc, cancel, tax_agency):
        xsd_name = get_key(tax_agency, 'xsd_name')['cancel' if cancel else 'post']
        try:
            validate_xml_from_attachment(self.env, xml_doc, xsd_name, prefix='l10n_es_edi_tbai')
        except UserError as e:
            return {'error': escape(str(e)), 'blocking_level': 'error'}
        return {}

    def _l10n_es_tbai_get_invoice_content_edi(self, invoice):
        cancel = invoice.edi_state in ('to_cancel', 'cancelled')
        if invoice.is_purchase_document():
            lroe_values = self._l10n_es_tbai_prepare_values_bi(invoice, False, cancel=cancel)
            xml_str = self.env['ir.qweb']._render('l10n_es_edi_tbai.template_LROE_240_main_recibidas', lroe_values).encode()
        else:
            xml_tree = self._get_l10n_es_tbai_invoice_xml(invoice, cancel)[invoice]['xml_file']
            xml_str = etree.tostring(xml_tree)
        return xml_str

    def _get_l10n_es_tbai_invoice_xml(self, invoice, cancel=False):
        def format_float(value, precision_digits=2):
            rounded_value = float_round(value, precision_digits=precision_digits)
            return float_repr(rounded_value, precision_digits=precision_digits)

        # If previously generated XML was posted and not rejected (success or timeout), reuse it
        doc = invoice._get_l10n_es_tbai_submitted_xml(cancel)
        if doc is not None:
            return {invoice: {'xml_file': doc}}

        # Otherwise, generate a new XML
        values = {
            **invoice.company_id._get_l10n_es_tbai_license_dict(),
            **self._l10n_es_tbai_get_header_values(invoice),
            **self._l10n_es_tbai_get_subject_values(invoice, cancel),
            **self._l10n_es_tbai_get_invoice_values(invoice, cancel),
            **self._l10n_es_tbai_get_trail_values(invoice, cancel),
            'is_emission': not cancel,
            'datetime_now': datetime.now(tz=timezone('Europe/Madrid')),
            'format_date': lambda d: datetime.strftime(d, '%d-%m-%Y'),
            'format_time': lambda d: datetime.strftime(d, '%H:%M:%S'),
            'format_float': format_float,
        }
        template_name = 'l10n_es_edi_tbai.template_invoice_main' + ('_cancel' if cancel else '_post')
        xml_str = self.env['ir.qweb']._render(template_name, values)
        xml_doc = cleanup_xml_node(xml_str, remove_blank_nodes=False)
        xml_doc = self._l10n_es_tbai_sign_invoice(invoice, xml_doc)
        res = {invoice: {'xml_file': xml_doc}}

        # Optional check using the XSD
        res[invoice].update(self._l10n_es_tbai_validate_xml_with_xsd(xml_doc, cancel, invoice.company_id.l10n_es_tbai_tax_agency))
        return res

    def _l10n_es_tbai_get_header_values(self, invoice):
        return {
            'tbai_version': self.L10N_ES_TBAI_VERSION,
            'odoo_version': release.version,
        }

    def _l10n_es_tbai_get_subject_values(self, invoice, cancel):
        # === SENDER (EMISOR) ===
        sender = invoice.company_id
        values = {
            'sender_vat': sender.vat[2:] if sender.vat.startswith('ES') else sender.vat,
            'sender': sender,
        }
        if cancel:
            return values  # cancellation invoices do not specify recipients (they stay the same)

        # NOTE: TicketBai supports simplified invoices WITH recipients but we don't for now (we should for POS)
        # NOTE: TicketBAI credit notes for simplified invoices are ALWAYS simplified BUT can have a recipient even if invoice doesn't
        if invoice.l10n_es_is_simplified:
            return values  # do not set 'recipient' unless there is an actual recipient (used as condition in template)

        # === RECIPIENTS (DESTINATARIOS) ===
        nif = False
        alt_id_country = False
        partner = invoice.commercial_partner_id
        alt_id_number = partner.vat or 'NO_DISPONIBLE'
        alt_id_type = ""
        if (not partner.country_id or partner.country_id.code == 'ES') and partner.vat:
            # ES partner with VAT.
            nif = partner.vat[2:] if partner.vat.startswith('ES') else partner.vat
        elif partner.country_id.code in self.env.ref('base.europe').country_ids.mapped('code'):
            # European partner
            alt_id_type = '02'
        else:
            # Non-european partner
            if partner.vat:
                alt_id_type = '04'
            else:
                alt_id_type = '06'
            if partner.country_id:
                alt_id_country = partner.country_id.code

        values_dest = {
            'nif': nif,
            'alt_id_country': alt_id_country,
            'alt_id_number': alt_id_number,
            'alt_id_type': alt_id_type,
            'partner': partner,
            'partner_address': ', '.join(filter(None, [partner.street, partner.street2, partner.city])),
        }

        values.update({
            'recipient': values_dest,
        })
        return values

    def _l10n_es_tbai_get_invoice_values(self, invoice, cancel):
        # Header
        values = {'invoice': invoice}
        if cancel:
            return values

        # Credit notes (factura rectificativa)
        # NOTE values below would have to be adapted for purchase invoices (Bizkaia LROE)
        values['is_refund'] = invoice.move_type == 'out_refund'
        if values['is_refund']:
            values['credit_note_code'] = invoice.l10n_es_tbai_refund_reason
            values['credit_note_invoice'] = invoice.reversed_entry_id

        # Lines (detalle)
        refund_sign = (1 if values['is_refund'] else -1)
        invoice_lines = []
        for line in invoice.invoice_line_ids.filtered(lambda line: line.display_type not in ('line_section', 'line_note')):
            if line.discount == 100.0:
                inverse_currency_rate = abs(line.move_id.amount_total_signed / line.move_id.amount_total) if line.move_id.amount_total else 1
                balance_before_discount = - line.price_unit * line.quantity * inverse_currency_rate
            else:
                balance_before_discount = line.balance / (1 - line.discount / 100)
            discount = (balance_before_discount - line.balance)
            line_price_total = self._l10n_es_tbai_get_invoice_line_price_total(line)

            if not any([t.l10n_es_type == 'sujeto_isp' for t in line.tax_ids]):
                total = line_price_total * abs(line.balance / line.amount_currency if line.amount_currency != 0 else 1) * -refund_sign
            else:
                total = abs(line.balance) * -refund_sign * (-1 if line_price_total < 0 else 1)
            invoice_lines.append({
                'line': line,
                'discount': discount * refund_sign,
                'unit_price': (line.balance + discount) / line.quantity * refund_sign if line.quantity > 0 else 0,
                'total': total,
                'description': regex_sub(r'[^0-9a-zA-Z ]', '', line.name or '')[:250]
            })
        values['invoice_lines'] = invoice_lines
        # Tax details (desglose)
        importe_total, desglose, amount_retention = self._l10n_es_tbai_get_importe_desglose(invoice)
        values['amount_total'] = importe_total
        values['invoice_info'] = desglose
        values['amount_retention'] = amount_retention * refund_sign if amount_retention != 0.0 else 0.0

        # Regime codes (ClaveRegimenEspecialOTrascendencia)
        # NOTE there's 11 more codes to implement, also there can be up to 3 in total
        # See https://www.gipuzkoa.eus/documents/2456431/13761128/Anexo+I.pdf/2ab0116c-25b4-f16a-440e-c299952d683d
        export_exempts = invoice.invoice_line_ids.tax_ids.filtered(lambda t: t.l10n_es_exempt_reason == 'E2')
        # If an invoice line contains an OSS tax, the invoice is considered as an OSS operation
        is_oss = self._has_oss_taxes(invoice)

        if is_oss:
            values['regime_key'] = ['17']
        elif export_exempts:
            values['regime_key'] = ['02']
        else:
            values['regime_key'] = ['01']

        values['nosujeto_causa'] = 'IE' if is_oss else 'RL'

        return values

    def _l10n_es_tbai_get_invoice_line_price_total(self, invoice_line):
        price_total = invoice_line.price_total
        retention_tax_lines = invoice_line.tax_ids.filtered(lambda t: t.l10n_es_type == "retencion")
        if retention_tax_lines:
            line_discount_price_unit = invoice_line.price_unit * (1 - (invoice_line.discount / 100.0))
            tax_lines_no_retention = invoice_line.tax_ids - retention_tax_lines
            if tax_lines_no_retention:
                taxes_res = tax_lines_no_retention.compute_all(line_discount_price_unit,
                                                               quantity=invoice_line.quantity,
                                                               currency=invoice_line.currency_id,
                                                               product=invoice_line.product_id,
                                                               partner=invoice_line.move_id.partner_id,
                                                               is_refund=invoice_line.is_refund)
                price_total = taxes_res['total_included']
        return price_total

    def _l10n_es_tbai_get_importe_desglose(self, invoice):
        com_partner = invoice.commercial_partner_id
        sign = -1 if invoice.move_type in ('out_refund', 'in_refund') else 1
        if (com_partner.country_id.code in ('ES', False) and not (com_partner.vat or '').startswith("ESN")) \
                or invoice.l10n_es_is_simplified:
            tax_details_info_vals = self._l10n_es_edi_get_invoices_tax_details_info(invoice)
            tax_amount_retention = tax_details_info_vals['tax_amount_retention']
            desglose = {'DesgloseFactura': tax_details_info_vals['tax_details_info']}
            desglose['DesgloseFactura'].update({'S1': tax_details_info_vals['S1_list'],
                                                'S2': tax_details_info_vals['S2_list']})
            importe_total = round(sign * (
                tax_details_info_vals['tax_details']['base_amount']
                + tax_details_info_vals['tax_details']['tax_amount']
                - tax_amount_retention
            ), 2)
        else:
            tax_details_info_service_vals = self._l10n_es_edi_get_invoices_tax_details_info(
                invoice,
                filter_invl_to_apply=lambda x: any(t.tax_scope == 'service' for t in x.tax_ids)
            )
            tax_details_info_consu_vals = self._l10n_es_edi_get_invoices_tax_details_info(
                invoice,
                filter_invl_to_apply=lambda x: any(t.tax_scope == 'consu' for t in x.tax_ids)
            )
            service_retention = tax_details_info_service_vals['tax_amount_retention']
            consu_retention = tax_details_info_consu_vals['tax_amount_retention']
            desglose = {}
            if tax_details_info_service_vals['tax_details_info']:
                desglose.setdefault('DesgloseTipoOperacion', {})
                desglose['DesgloseTipoOperacion']['PrestacionServicios'] = tax_details_info_service_vals['tax_details_info']
                desglose['DesgloseTipoOperacion']['PrestacionServicios'].update(
                    {'S1': tax_details_info_service_vals['S1_list'],
                     'S2': tax_details_info_service_vals['S2_list']})

            if tax_details_info_consu_vals['tax_details_info']:
                desglose.setdefault('DesgloseTipoOperacion', {})
                desglose['DesgloseTipoOperacion']['Entrega'] = tax_details_info_consu_vals['tax_details_info']
                desglose['DesgloseTipoOperacion']['Entrega'].update(
                    {'S1': tax_details_info_consu_vals['S1_list'],
                     'S2': tax_details_info_consu_vals['S2_list']})
            importe_total = round(sign * (
                tax_details_info_service_vals['tax_details']['base_amount']
                + tax_details_info_service_vals['tax_details']['tax_amount']
                - service_retention
                + tax_details_info_consu_vals['tax_details']['base_amount']
                + tax_details_info_consu_vals['tax_details']['tax_amount']
                - consu_retention
            ), 2)
            tax_amount_retention = service_retention + consu_retention
        return importe_total, desglose, tax_amount_retention

    def _l10n_es_tbai_get_trail_values(self, invoice, cancel):
        prev_invoice = invoice.company_id._get_l10n_es_tbai_last_posted_invoice(invoice)
        # NOTE: assumtion that last posted == previous works because XML is generated on post
        if prev_invoice and not cancel:
            return {
                'chain_prev_invoice': prev_invoice
            }
        else:
            return {}

    def _l10n_es_tbai_sign_invoice(self, invoice, xml_root):
        company = invoice.company_id
        cert_private, cert_public = (
            company.l10n_es_edi_certificate_id.sudo()._get_key_pair()
        )
        public_key = cert_public.public_key()

        # Identifiers
        document_id = "Document-" + str(uuid4())
        signature_id = "Signature-" + document_id
        keyinfo_id = "KeyInfo-" + document_id
        sigproperties_id = "SignatureProperties-" + document_id

        # Render digital signature scaffold from QWeb
        common_name = cert_public.issuer.get_attributes_for_oid(NameOID.COMMON_NAME)[0].value
        org_unit = cert_public.issuer.get_attributes_for_oid(NameOID.ORGANIZATIONAL_UNIT_NAME)[0].value
        org_name = cert_public.issuer.get_attributes_for_oid(NameOID.ORGANIZATION_NAME)[0].value
        country_name = cert_public.issuer.get_attributes_for_oid(NameOID.COUNTRY_NAME)[0].value
        values = {
            'dsig': {
                'document_id': document_id,
                'x509_certificate': bytes_as_block(cert_public.public_bytes(encoding=serialization.Encoding.DER)),
                'public_modulus': bytes_as_block(int_as_bytes(public_key.public_numbers().n)),
                'public_exponent': bytes_as_block(int_as_bytes(public_key.public_numbers().e)),
                'iso_now': datetime.now().isoformat(),
                'keyinfo_id': keyinfo_id,
                'signature_id': signature_id,
                'sigproperties_id': sigproperties_id,
                'reference_uri': "Reference-" + document_id,
                'sigpolicy_url': get_key(company.l10n_es_tbai_tax_agency, 'sigpolicy_url'),
                'sigpolicy_digest': get_key(company.l10n_es_tbai_tax_agency, 'sigpolicy_digest'),
                'sigcertif_digest': b64encode(cert_public.fingerprint(hashes.SHA256())).decode(),
                'x509_issuer_description': 'CN={}, OU={}, O={}, C={}'.format(common_name, org_unit, org_name, country_name),
                'x509_serial_number': cert_public.serial_number,
            }
        }
        xml_sig_str = self.env['ir.qweb']._render('l10n_es_edi_tbai.template_digital_signature', values)
        xml_sig = cleanup_xml_signature(xml_sig_str)

        # Complete document with signature template
        xml_root.append(xml_sig)

        # Compute digest values for references
        calculate_references_digests(xml_sig.find("SignedInfo", namespaces=NS_MAP))

        # Sign (writes into SignatureValue)
        fill_signature(xml_sig, cert_private)

        return xml_root

    # -------------------------------------------------------------------------
    # WEB SERVICE CALLS
    # -------------------------------------------------------------------------

    def _l10n_es_tbai_post_to_web_service(self, invoice, invoice_xml, cancel=False):
        company = invoice.company_id

        try:
            # Call the web service, retrieve and parse response
            success, message, response_xml = self._l10n_es_tbai_post_to_agency(
                self.env, company.l10n_es_tbai_tax_agency, invoice, invoice_xml, cancel)
        except (ValueError, RequestException) as e:
            # In case of timeout / request exception, return warning
            return {invoice: {
                'error': str(e),
                'blocking_level': 'warning',
                'response': None,
            }}

        if success:
            return {invoice: {
                'success': True,
                'message': message,
                'response': response_xml,
            }}
        else:
            return {invoice: {
                'error': message,
                'blocking_level': 'error',
                'response': response_xml,
            }}

    # -------------------------------------------------------------------------
    # WEB SERVICE METHODS
    # -------------------------------------------------------------------------
    # Provides helper methods for interacting with the Basque country's TicketBai servers.

    L10N_ES_TBAI_VERSION = 1.2

    def _l10n_es_tbai_post_to_agency(self, env, agency, invoice, invoice_xml, cancel=False):
        if agency in ('araba', 'gipuzkoa'):
            post_method, process_method = self._l10n_es_tbai_prepare_post_params_ar_gi, self._l10n_es_tbai_process_post_response_ar_gi
        elif agency == 'bizkaia':
            post_method, process_method = self._l10n_es_tbai_prepare_post_params_bi, self._l10n_es_tbai_process_post_response_bi
        params = post_method(env, agency, invoice, invoice_xml, cancel)
        response = self._l10n_es_tbai_send_request_to_agency(timeout=10, **params)
        return process_method(env, response)

    def _l10n_es_tbai_send_request_to_agency(self, *args, **kwargs):
        session = requests.Session()
        session.cert = kwargs.pop('pkcs12_data')
        session.mount("https://", PatchedHTTPAdapter())
        return session.request('post', *args, **kwargs)

    def _l10n_es_tbai_prepare_post_params_ar_gi(self, env, agency, invoice, invoice_xml, cancel=False):
        """Web service parameters for Araba and Gipuzkoa."""
        company = invoice.company_id
        return {
            'url': get_key(agency, 'cancel_url_' if cancel else 'post_url_', company.l10n_es_edi_test_env),
            'headers': {"Content-Type": "application/xml; charset=utf-8"},
            'pkcs12_data': company.l10n_es_edi_certificate_id,
            'data': etree.tostring(invoice_xml, encoding='UTF-8'),
        }

    def _l10n_es_tbai_process_post_response_ar_gi(self, env, response):
        """Government response processing for Araba and Gipuzkoa."""
        try:
            response_xml = etree.fromstring(response.content)
        except etree.XMLSyntaxError as e:
            return False, e, None

        # Error management
        message = ''
        already_received = False
        # Get message in basque if env is in basque
        msg_node_name = 'Azalpena' if get_lang(env).code == 'eu_ES' else 'Descripcion'
        for xml_res_node in response_xml.findall(r'.//ResultadosValidacion'):
            message_code = xml_res_node.find('Codigo').text
            message += message_code + ": " + xml_res_node.find(msg_node_name).text + "\n"
            if message_code in ('005', '019'):
                already_received = True  # error codes 5/19 mean XML was already received with that sequence
        response_code = int(response_xml.find(r'.//Estado').text)
        response_success = (response_code == 0) or already_received

        return response_success, message, response_xml

    def _l10n_es_tbai_get_in_invoice_values_batuz(self, invoice):
        """ For the vendor bills for Bizkaia, the structure is different than the regular Ticketbai XML (LROE)"""
        values = {
            **self._l10n_es_tbai_get_subject_values(invoice, False),
            **self._l10n_es_tbai_get_header_values(invoice),
             **invoice._get_vendor_bill_tax_values(),
            'invoice': invoice,
            'datetime_now': datetime.now(tz=timezone('Europe/Madrid')),
            'format_date': lambda d: datetime.strftime(d, '%d-%m-%Y'),
            'format_time': lambda d: datetime.strftime(d, '%H:%M:%S'),
            'format_float': lambda f: float_repr(f, precision_digits=2),
        }
        # Check if intracom
        mod_303_10 = self.env.ref('l10n_es.mod_303_casilla_10_balance')._get_matching_tags()
        mod_303_11 = self.env.ref('l10n_es.mod_303_casilla_11_balance')._get_matching_tags()
        tax_tags = invoice.invoice_line_ids.tax_ids.repartition_line_ids.tag_ids
        intracom = bool(tax_tags & (mod_303_10 + mod_303_11))
        values['regime_key'] = ['09'] if intracom else ['01']
        # Credit notes (factura rectificativa)
        values['is_refund'] = invoice.move_type == 'in_refund'
        if values['is_refund']:
            values['credit_note_code'] = invoice.l10n_es_tbai_refund_reason
            values['credit_note_invoice'] = invoice.reversed_entry_id
        values['tipofactura'] = 'F5' if invoice._l10n_es_is_dua() else 'F1'
        return values

    def _l10n_es_tbai_prepare_values_bi(self, invoice, invoice_xml, cancel=False):
        sender = invoice.company_id
        lroe_values = {
            'is_emission': not cancel,
            'sender': sender,
            'sender_vat': sender.vat[2:] if sender.vat.startswith('ES') else sender.vat,
            'fiscal_year': str(invoice.date.year),
        }
        if invoice.is_sale_document():
            lroe_values.update({'tbai_b64_list': [b64encode(etree.tostring(invoice_xml, encoding="UTF-8")).decode()]})
        else:
            lroe_values.update(self._l10n_es_tbai_get_in_invoice_values_batuz(invoice))
        return lroe_values

    def _l10n_es_tbai_prepare_post_params_bi(self, env, agency, invoice, invoice_xml, cancel=False):
        """Web service parameters for Bizkaia."""
        lroe_values = self._l10n_es_tbai_prepare_values_bi(invoice, invoice_xml, cancel=cancel)
        if invoice.is_purchase_document():
            lroe_str = env['ir.qweb']._render('l10n_es_edi_tbai.template_LROE_240_main_recibidas', lroe_values)
            if cancel:
                invoice.l10n_es_tbai_cancel_xml = b64encode(lroe_str.encode())
            else:
                invoice.l10n_es_tbai_post_xml = b64encode(lroe_str.encode())
        else:
            lroe_str = env['ir.qweb']._render('l10n_es_edi_tbai.template_LROE_240_main', lroe_values)

        lroe_xml = cleanup_xml_node(lroe_str)
        lroe_str = etree.tostring(lroe_xml, encoding="UTF-8")
        lroe_bytes = gzip.compress(lroe_str)

        company = invoice.company_id
        return {
            'url': get_key(agency, 'cancel_url_' if cancel else 'post_url_', company.l10n_es_edi_test_env),
            'headers': {
                'Accept-Encoding': 'gzip',
                'Content-Encoding': 'gzip',
                'Content-Length': str(len(lroe_str)),
                'Content-Type': 'application/octet-stream',
                'eus-bizkaia-n3-version': '1.0',
                'eus-bizkaia-n3-content-type': 'application/xml',
                'eus-bizkaia-n3-data': json.dumps({
                    'con': 'LROE',
                    'apa': '1.1' if invoice.is_sale_document() else '2',
                    'inte': {
                        'nif': lroe_values['sender_vat'],
                        'nrs': invoice.company_id.name,
                    },
                    'drs': {
                        'mode': '240',
                        # NOTE: modelo 140 for freelancers (in/out invoices)
                        # modelo 240 for legal entities (lots of account moves ?)
                        'ejer': str(invoice.date.year),
                    }
                }),
            },
            'pkcs12_data': invoice.company_id.l10n_es_edi_certificate_id,
            'data': lroe_bytes,
        }

    def _l10n_es_tbai_process_post_response_bi(self, env, response):
        """Government response processing for Bizkaia."""
        # GLOBAL STATUS (LROE)
        response_messages = []
        response_success = True
        if response.headers['eus-bizkaia-n3-tipo-respuesta'] != "Correcto":
            code = response.headers['eus-bizkaia-n3-codigo-respuesta']
            response_messages.append(code + ': ' + response.headers['eus-bizkaia-n3-mensaje-respuesta'])
            response_success = False

        response_data = response.content
        response_xml = None
        if response_data:
            try:
                response_xml = etree.fromstring(response_data)
            except etree.XMLSyntaxError as e:
                response_success = False
                response_messages.append(str(e))
        else:
            response_success = False
            response_messages.append(_('No XML response received from LROE.'))

        # INVOICE STATUS (only one in batch)
        # Get message in basque if env is in basque
        if response_xml is not None:
            msg_node_name = 'DescripcionErrorRegistro' + ('EU' if get_lang(env).code == 'eu_ES' else 'ES')
            invoice_success = response_xml.find(r'.//EstadoRegistro').text == "Correcto"
            if not invoice_success:
                invoice_code = response_xml.find(r'.//CodigoErrorRegistro').text
                if invoice_code == "B4_2000003":  # already received
                    invoice_success = True
                response_messages.append(invoice_code + ": " + (response_xml.find(rf'.//{msg_node_name}').text or ''))

        return response_success and invoice_success, '<br/>'.join(response_messages), response_xml

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from base64 import b64decode, b64encode
from datetime import datetime
from re import sub as regex_sub
from collections import defaultdict

from lxml import etree
from odoo import _, api, fields, models
from odoo.addons.l10n_es_edi_tbai.models.l10n_es_edi_tbai_agencies import get_key
from odoo.exceptions import UserError

L10N_ES_TBAI_CRC8_TABLE = [
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

class AccountMove(models.Model):
    _inherit = 'account.move'

    # Stored fields
    l10n_es_tbai_chain_index = fields.Integer(
        string="TicketBAI chain index",
        help="Invoice index in chain, set if and only if an in-chain XML was submitted and did not error",
        copy=False, readonly=True,
    )

    # Stored XML Binaries
    l10n_es_tbai_post_xml = fields.Binary(
        attachment=True, readonly=True, copy=False,
        string="Submission XML",
        help="Submission XML sent to TicketBAI. Kept if accepted or no response (timeout), cleared otherwise.",
    )
    l10n_es_tbai_cancel_xml = fields.Binary(
        attachment=True, readonly=True, copy=False,
        string="Cancellation XML",
        help="Cancellation XML sent to TicketBAI. Kept if accepted or no response (timeout), cleared otherwise.",
    )

    # Non-stored fields
    l10n_es_tbai_is_required = fields.Boolean(
        string="TicketBAI required",
        help="Is the Basque EDI (TicketBAI) needed ?",
        compute="_compute_l10n_es_tbai_is_required",
    )

    # Optional fields
    l10n_es_tbai_refund_reason = fields.Selection(
        selection=[
            ('R1', "R1: Art. 80.1, 80.2, 80.6 and rights founded error"),
            ('R2', "R2: Art. 80.3"),
            ('R3', "R3: Art. 80.4"),
            ('R4', "R4: Art. 80 - other"),
            ('R5', "R5: Factura rectificativa en facturas simplificadas"),
        ],
        string="Invoice Refund Reason Code (TicketBai)",
        help="BOE-A-1992-28740. Ley 37/1992, de 28 de diciembre, del Impuesto sobre el "
        "Valor Añadido. Artículo 80. Modificación de la base imponible.",
        copy=False,
    )

    # -------------------------------------------------------------------------
    # API-DECORATED & EXTENDED METHODS
    # -------------------------------------------------------------------------

    @api.depends('move_type', 'company_id')
    def _compute_l10n_es_tbai_is_required(self):
        for move in self:
            move.l10n_es_tbai_is_required = (move.is_sale_document() or move.is_purchase_document() and move.company_id.l10n_es_tbai_tax_agency == 'bizkaia'
                                             and not any(t.l10n_es_type == 'ignore' for t in move.invoice_line_ids.tax_ids))\
                and move.country_code == 'ES' \
                and move.company_id.l10n_es_tbai_tax_agency

    @api.depends('state', 'edi_document_ids.state')
    def _compute_show_reset_to_draft_button(self):
        # EXTENDS account_edi account.move
        super()._compute_show_reset_to_draft_button()

        for move in self:
            if move.l10n_es_tbai_chain_index:
                move.show_reset_to_draft_button = False

    def button_draft(self):
        # EXTENDS account account.move
        for move in self:
            if move.l10n_es_tbai_chain_index and not move.edi_state == 'cancelled':
                # NOTE this last condition (state is cancelled) is there because
                # _postprocess_cancel_edi_results calls button_draft before
                # calling button_cancel. Draft button does not appear for user.
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

    def _l10n_es_tbai_is_in_chain(self):
        """
        True iff invoice has been posted to the chain and confirmed by govt.
        Note that cancelled invoices remain part of the chain.
        """
        tbai_doc_ids = self.edi_document_ids.filtered(lambda d: d.edi_format_id.code == 'es_tbai')
        return self.l10n_es_tbai_is_required \
            and len(tbai_doc_ids) > 0 \
            and not any(tbai_doc_ids.filtered(lambda d: d.state == 'to_send'))

    def _get_l10n_es_tbai_sequence_and_number(self):
        """Get the TicketBAI sequence a number values for this invoice."""
        self.ensure_one()
        if self.is_purchase_document(): # Batuz
            # Check if we are cancelling or not
            doc = self.env['account.edi.document'].search([('state', '=', 'to_cancel'),
                                                           ('edi_format_id.code', '=', 'es_tbai')], limit=1)
            if doc and self.l10n_es_tbai_post_xml:
                vals = self._get_l10n_es_tbai_values_from_xml({
                    'sequence': './/CabeceraFactura/SerieFactura',
                    'number': './/CabeceraFactura/NumFactura',
                })
                if vals['sequence'] and vals['number']:
                    return vals['sequence'], vals['number']

            number = self.ref
            sequence = "TEST" if self.company_id.l10n_es_edi_test_env else ""
        else:
            sequence = self.sequence_prefix.rstrip('/')

            # NOTE non-decimal characters should not appear in the number
            seq_length = self._get_sequence_format_param(self.name)[1]['seq_length']
            number = f"{self.sequence_number:0{seq_length}d}"

            sequence = regex_sub(r"[^0-9A-Za-z.\_\-\/]", "", sequence)  # remove forbidden characters
            sequence = regex_sub(r"\s+", " ", sequence)  # no more than one consecutive whitespace allowed
            # NOTE (optional) not recommended to use chars out of ([0123456789ABCDEFGHJKLMNPQRSTUVXYZ.\_\-\/ ])
            sequence += "TEST" if self.company_id.l10n_es_edi_test_env else ""
        return sequence, number

    def _get_l10n_es_tbai_signature_and_date(self):
        """
        Get the TicketBAI signature and registration date for this invoice.
        Values are read directly from the 'post' XMLs submitted to the government \
            (the 'cancel' XML is ignored).
        The registration date is the date the invoice was registered into the govt's TicketBAI servers.
        """
        self.ensure_one()
        vals = self._get_l10n_es_tbai_values_from_xml({
            'signature': r'.//{http://www.w3.org/2000/09/xmldsig#}SignatureValue',
            'registration_date': r'.//CabeceraFactura//FechaExpedicionFactura'
        })
        # RFC2045 - Base64 Content-Transfer-Encoding (page 25)
        # Any characters outside of the base64 alphabet are to be ignored in base64-encoded data.
        signature = vals['signature'].replace("\n", "")
        registration_date = datetime.strptime(vals['registration_date'], '%d-%m-%Y')
        return signature, registration_date

    def _get_l10n_es_tbai_id(self):
        """Get the TicketBAI ID (TBAID) as defined in the TicketBAI doc."""
        self.ensure_one()
        if not self._l10n_es_tbai_is_in_chain():
            return ''

        signature, registration_date = self._get_l10n_es_tbai_signature_and_date()
        company = self.company_id
        tbai_id_no_crc = '-'.join([
            'TBAI',
            str(company.vat[2:] if company.vat.startswith('ES') else company.vat),
            datetime.strftime(registration_date, '%d%m%y'),
            signature[:13],
            ''  # CRC
        ])
        return tbai_id_no_crc + self._l10n_es_edi_tbai_crc8(tbai_id_no_crc)

    def _get_l10n_es_tbai_qr(self):
        """Returns the URL for the invoice's QR code.  We can not use url_encode because it escapes / e.g."""
        self.ensure_one()
        if not self._l10n_es_tbai_is_in_chain():
            return ''

        company = self.company_id
        sequence, number = self._get_l10n_es_tbai_sequence_and_number()
        tbai_qr_no_crc = get_key(company.l10n_es_tbai_tax_agency, 'qr_url_', company.l10n_es_edi_test_env) + '?' + '&'.join([
            'id=' + self._get_l10n_es_tbai_id(),
            's=' + sequence,
            'nf=' + number,
            'i=' + self._get_l10n_es_tbai_values_from_xml({'importe': r'.//ImporteTotalFactura'})['importe']
        ])
        qr_url = tbai_qr_no_crc + '&cr=' + self._l10n_es_edi_tbai_crc8(tbai_qr_no_crc)
        return qr_url

    def _l10n_es_edi_tbai_crc8(self, data):
        crc = 0x0
        for c in data:
            crc = L10N_ES_TBAI_CRC8_TABLE[(crc ^ ord(c)) & 0xFF]
        return '{:03d}'.format(crc & 0xFF)

    def _get_l10n_es_tbai_values_from_xml(self, xpaths):
        """
        This function reads values directly from the 'post' XML submitted to the government \
            (the 'cancel' XML is ignored).
        """
        res = dict.fromkeys(xpaths, '')
        doc_xml = self._get_l10n_es_tbai_submitted_xml()
        if doc_xml is None:
            return res
        for key, value in xpaths.items():
            res[key] = doc_xml.find(value).text
        return res

    def _get_l10n_es_tbai_submitted_xml(self, cancel=False):
        """Returns the XML object representing the post or cancel document."""
        self.ensure_one()
        self = self.with_context(bin_size=False)
        doc = self.l10n_es_tbai_cancel_xml if cancel else self.l10n_es_tbai_post_xml
        if not doc:
            return None
        return etree.fromstring(b64decode(doc))

    def _update_l10n_es_tbai_submitted_xml(self, xml_doc, cancel):
        """Updates the binary data of the post or cancel document, from its XML object."""
        self.ensure_one()
        b64_doc = b'' if xml_doc is None else b64encode(etree.tostring(xml_doc, encoding='UTF-8'))
        if cancel:
            self.l10n_es_tbai_cancel_xml = b64_doc
        else:
            self.l10n_es_tbai_post_xml = b64_doc

    def _get_vendor_bill_tax_values(self):
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
            code = "C" # Bienes Corrientes
            if tax.l10n_es_bien_inversion:
                code = "I" # Investment Goods
            if tax.tax_scope == 'service':
                code = 'G' # Gastos
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

## File: models\ir_attachment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api
from odoo.addons.l10n_es_edi_tbai.models.l10n_es_edi_tbai_agencies import get_key
from odoo.tools import xml_utils


class IrAttachment(models.Model):
    _inherit = 'ir.attachment'

    @api.model
    def action_download_xsd_files(self):
        """
        Downloads the TicketBAI XSD validation files if they don't already exist, for the active tax agency.
        """
        xml_utils.load_xsd_files_from_url(
            self.env, 'https://www.w3.org/TR/xmldsig-core/xmldsig-core-schema.xsd', 'xmldsig-core-schema.xsd',
            xsd_name_prefix='l10n_es_edi_tbai')

        for agency in ['gipuzkoa', 'araba', 'bizkaia']:
            urls = get_key(agency, 'xsd_url')
            names = get_key(agency, 'xsd_name')
            # For Bizkaia, one url per XSD (post/cancel)
            if isinstance(urls, dict):
                for move_type in ('post', 'cancel'):
                    xml_utils.load_xsd_files_from_url(
                        self.env, urls[move_type], names[move_type],
                        xsd_name_prefix='l10n_es_edi_tbai',
                    )
            # For other agencies, single url to zip file (only keep the desired names)
            else:
                xml_utils.load_xsd_files_from_url(
                    self.env, urls,  # NOTE: file_name discarded when XSDs bundled in ZIPs
                    xsd_name_prefix='l10n_es_edi_tbai',
                    xsd_names_filter=list(names.values()),
                )
        return super().action_download_xsd_files()

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

## File: models\l10n_es_edi_tbai_certificate.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from base64 import b64decode

from odoo import models
from odoo.addons.account.tools.certificate import load_key_and_certificates


class Certificate(models.Model):
    _inherit = 'l10n_es_edi.certificate'

    # -------------------------------------------------------------------------
    # HELPERS
    # -------------------------------------------------------------------------

    def _get_key_pair(self):
        self.ensure_one()

        if not self.password:
            return None, None

        private_key, certificate = load_key_and_certificates(
            b64decode(self.with_context(bin_size=False).content),  # Without bin_size=False, size is returned instead of content
            self.password.encode(),
        )

        return private_key, certificate

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import markupsafe
from odoo import _, api, fields, models, release


# === TBAI license values ===
L10N_ES_TBAI_LICENSE_DICT = {
    'production': {
        'license_name': _('Production license'),  # all agencies
        'license_number': 'TBAIGI5A266A7CCDE1EC',
        'license_nif': 'N0251909H',
        'software_name': 'Odoo SA',
        'software_version': release.version,
    },
    'araba': {
        'license_name': _('Test license (Araba)'),
        'license_number': 'TBAIARbjjMClHKH00849',
        'license_nif': 'N0251909H',
        'software_name': 'Odoo SA',
        'software_version': release.version,
    },
    'bizkaia': {
        'license_name': _('Test license (Bizkaia)'),
        'license_number': 'TBAIBI00000000PRUEBA',
        'license_nif': 'A99800005',
        'software_name': 'SOFTWARE GARANTE TICKETBAI PRUEBA',
        'software_version': '1.0',
    },
    'gipuzkoa': {
        'license_name': _('Test license (Gipuzkoa)'),
        'license_number': 'TBAIGIPRE00000000965',
        'license_nif': 'N0251909H',
        'software_name': 'Odoo SA',
        'software_version': release.version,
    },
}

class ResCompany(models.Model):
    _inherit = 'res.company'

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

    @api.depends('country_id', 'l10n_es_edi_test_env', 'l10n_es_tbai_tax_agency')
    def _compute_l10n_es_tbai_license_html(self):
        for company in self:
            license_dict = company._get_l10n_es_tbai_license_dict()
            if license_dict:
                license_dict.update({
                    'tr_nif': _('Licence NIF'),
                    'tr_number': _('Licence number'),
                    'tr_name': _('Software name'),
                    'tr_version': _('Software version')
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
<strong>{tr_no_license}</strong>''').format(tr_no_license=_('TicketBAI is not configured'))

    def _get_l10n_es_tbai_license_dict(self):
        self.ensure_one()
        if self.country_code == 'ES' and self.l10n_es_tbai_tax_agency:
            if self.l10n_es_edi_test_env:  # test env: each agency has its test license
                license_key = self.l10n_es_tbai_tax_agency
            else:  # production env: only one license
                license_key = 'production'
            return L10N_ES_TBAI_LICENSE_DICT[license_key]
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

    def _get_l10n_es_tbai_last_posted_invoice(self, being_posted=False):
        """
        Returns the last invoice posted to this company's chain.
        That invoice may have been received by the govt or not (eg. in case of a timeout).
        Only upon confirmed reception/refusal of that invoice can another one be posted.
        :param being_posted: next invoice to be posted on the chain, ignored in search domain
        """
        domain = [
            ('l10n_es_tbai_chain_index', '!=', 0),
            ('company_id', '=', self.id)
        ]
        if being_posted:
            domain.append(('l10n_es_tbai_chain_index', '!=', being_posted.l10n_es_tbai_chain_index))
            # NOTE: being_posted may not have a chain index at all (if being posted for the first time)
        return self.env['account.move'].search(domain, limit=1, order='l10n_es_tbai_chain_index desc')

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_es_tbai_tax_agency = fields.Selection(related='company_id.l10n_es_tbai_tax_agency', readonly=False)

```

## File: models\xml_utils.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import hashlib
import re
from base64 import b64encode, encodebytes

from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.asymmetric import padding
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

def fill_signature(node, private_key):
    """
    Uses private_key to sign the SignedInfo sub-node of `node`, as specified in:
    https://www.w3.org/TR/xmldsig-core/#sec-SignatureValue
    https://www.w3.org/TR/xmldsig-core/#sec-SignedInfo
    """
    signed_info_xml = node.find('SignedInfo', namespaces=NS_MAP)

    # During signature generation, the digest is computed over the canonical form of the document
    signature = private_key.sign(
        canonicalize_node(signed_info_xml),
        padding.PKCS1v15(),
        hashes.SHA256()
    )
    node.find('SignatureValue', namespaces=NS_MAP).text =\
        bytes_as_block(signature)

def int_as_bytes(number):
    """
    Converts an integer to an ASCII/UTF-8 byte string (with no leading zeroes).
    """
    return number.to_bytes((number.bit_length() + 7) // 8, byteorder='big')

def bytes_as_block(string):
    """
    Returns the passed string modified to include a line feed every `length` characters.
    It may be recommended to keep length under 76:
    https://www.w3.org/TR/2004/REC-xmlschema-2-20041028/#rf-maxLength
    https://www.ietf.org/rfc/rfc2045.txt
    """
    return encodebytes(string).decode()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_edi_document
from . import account_edi_format
from . import account_move
from . import ir_attachment
from . import l10n_es_edi_tbai_certificate
from . import res_company
from . import res_config_settings
from . import xml_utils
from . import l10n_es_edi_tbai_agencies

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
                <xpath expr="//group[@id='other_tab_group']/group[last()]" position='after'>
                    <group id="ticketbai_group" string="TicketBAI" invisible="not l10n_es_tbai_is_required">
                        <field name="l10n_es_tbai_is_required" invisible="1"/>
                        <field name="move_type" invisible="1"/>
                        <field name="l10n_es_tbai_chain_index" groups="base.group_no_one"/>
                        <field name="l10n_es_tbai_refund_reason"
                            invisible="move_type not in ('out_refund', 'in_refund')"
                            readonly="state != 'draft'"/>
                        <field name="l10n_es_tbai_post_xml" invisible="1"/>
                        <field name="reversed_entry_id"
                               invisible="move_type not in ('in_refund', 'out_refund') or not l10n_es_tbai_is_required"
                               readonly="l10n_es_tbai_post_xml"/>
                    </group>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <template id="l10n_es_tbai_external_layout_standard" inherit_id="account.report_invoice_document">
        <xpath expr="//div[@id='qrcode']" position="after">
            <div name="l10n_es_tbai_qrcode" t-if="o._l10n_es_tbai_is_in_chain()" style="page-break-inside: avoid">
                <div t-out="o._get_l10n_es_tbai_id()"/>
                <img
                    t-att-src="'/report/barcode/?barcode_type=%s&amp;value=%s&amp;width=%s&amp;height=%s&amp;barLevel=%s'%('QR', quote_plus(o._get_l10n_es_tbai_qr()), 125, 125, 'M')"/>

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
        name="Licenses (TicketBAI)"
        action="base.action_res_company_form"
        sequence="90"
        parent="l10n_es_edi_sii.menu_l10n_es_edi_root"
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
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//block[@name='spain_localization']/setting" position="attributes">
                <attribute name="string">Registro de Libros connection SII/TicketBAI</attribute>
            </xpath>
            <xpath expr="//block[@name='spain_localization']/setting/div[hasclass('content-group')]/div[hasclass('mt16')]/*" position="before">
                <label for="l10n_es_tbai_tax_agency" class="o_light_label"/>
                <field name="l10n_es_tbai_tax_agency"/>
                <div class="text-muted" invisible="l10n_es_tbai_tax_agency">
                    No tax agency selected: TicketBAI not activated.
                </div>
                <div class="text-muted" invisible="not l10n_es_tbai_tax_agency">
                    Tax agency selected: TicketBAI is activated.
                </div>
                <br/>
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
from odoo.exceptions import UserError


class AccountMoveReversal(models.TransientModel):
    _inherit = 'account.move.reversal'

    l10n_es_tbai_is_required = fields.Boolean(
        compute="_compute_l10n_es_tbai_is_required", readonly=True,
        string="Is TicketBai required for this reversal",
    )

    l10n_es_tbai_refund_reason = fields.Selection(
        selection=[
            ('R1', "R1: Art. 80.1, 80.2, 80.6 and rights founded error"),
            ('R2', "R2: Art. 80.3"),
            ('R3', "R3: Art. 80.4"),
            ('R4', "R4: Art. 80 - other"),
            ('R5', "R5: Factura rectificativa en facturas simplificadas"),
        ],
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
        if move.company_id.country_id.code == "ES" and move.l10n_es_tbai_is_required:
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
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move_reversal

```

