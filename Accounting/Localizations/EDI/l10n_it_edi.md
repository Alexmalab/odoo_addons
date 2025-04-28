# Odoo Module: l10n_it_edi

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import tools
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Italy - E-invoicing',
    'countries': ['it'],
    'version': '0.4',
    'depends': [
        'l10n_it',
        'account_edi_proxy_client',
    ],
    'auto_install': ['l10n_it'],
    'description': """
E-invoice implementation
    """,
    'category': 'Accounting/Localizations/EDI',
    'website': 'http://www.odoo.com/',
    'data': [
        'security/ir.model.access.csv',
        'data/invoice_it_template.xml',
        'data/invoice_it_simplified_template.xml',
        'data/ir_cron.xml',
        'data/account.account.tag.csv',
        'views/res_config_settings_views.xml',
        'views/l10n_it_view.xml',
        'views/report_invoice.xml',
        'wizard/account_move_send_views.xml',
    ],
    'demo': [
        'data/account_invoice_demo.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.tag.csv

```csv
"id","name","applicability","country_id/id"
"l10n_it_edi_professional_fees_tag","Professional fees","accounts","base.it"

```

## File: data\account_invoice_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- add VAT, codice fiscal and tax system for main company -->
        <record id="l10n_it.demo_company_it" model="res.company">
            <field name="vat">IT01654010345</field>
            <field name="street">Test Street</field>
            <field name="city">Prova</field>
            <field name="zip">12345</field>
            <field name="l10n_it_codice_fiscale">01654010345</field>
            <field name="l10n_it_tax_system">RF01</field>
            <field name="zip">12345</field>
        </record>

        <record id="l10n_it.partner_demo_company_it" model="res.partner">
            <field name="l10n_it_pa_index">0803HR0</field>
        </record>

        <record id="partner_demo_it" model="res.partner">
            <field name="name">Palazzo dell'Arte</field>
            <field name="vat">IT00000010215</field>
            <field name="street">Piazza Marconi 5</field>
            <field name="city">Cremona</field>
            <field name="country_id" ref="base.it"/>
            <field name="state_id" ref="base.state_it_cr"/>
            <field name="zip">26000</field>
            <field name="email">info@partner.itexample.com</field>
            <field name="website">www.itexample.com</field>
        </record>

        <record id="demo_l10n_it_edi_bank" model="res.partner.bank">
            <field name="acc_type">iban</field>
            <field name="acc_number">BE71096123456769</field>
            <field name="bank_id" ref="base.bank_bnp"/>
            <field name="partner_id" ref="l10n_it.partner_demo_company_it"/>
            <field name="company_id" ref="l10n_it.demo_company_it"/>
        </record>

        <record id="demo_l10n_it_edi_partner_a" model="res.partner">
            <field name="name">Biscotti Oslenghi</field>
            <field name="company_type">company</field>
            <field name="country_id" ref="base.it"/>
            <field name="street">1234 Strada del Caffè</field>
            <field name="city">Milano</field>
            <field name="zip">20100</field>
            <field name="vat">IT06289781004</field>
            <field name="l10n_it_codice_fiscale">06289781004</field>
            <field name="l10n_it_pa_index">N8MIMM9</field>
        </record>

        <record id="demo_l10n_it_edi_partner_pa" model="res.partner">
            <field name="name">Agenzia Regionale Emergenza Urgenza</field>
            <field name="company_type">company</field>
            <field name="country_id" ref="base.it"/>
            <field name="street">Via Alfredo Campanini 6</field>
            <field name="city">Milano</field>
            <field name="zip">20124</field>
            <field name="vat">IT11513540960</field>
            <field name="l10n_it_codice_fiscale">11513540960</field>
            <field name="l10n_it_pa_index">SOOTJS</field>
        </record>

        <record id="demo_l10n_it_edi_proxy_user" model="account_edi_proxy_client.user">
            <field name="id_client">demo_id_client</field>
            <field name="company_id" ref="l10n_it.demo_company_it"/>
            <field name="proxy_type">l10n_it_edi</field>
            <field name="edi_mode">demo</field>
            <field name="edi_identification">01654010345</field>
            <field name="private_key">1234</field>
            <field name="refresh_token">demo</field>
        </record>

    </data>
</odoo>

```

## File: data\invoice_it_simplified_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="account_invoice_line_it_simplified_FatturaPA">
            <DatiBeniServizi>
                <Descrizione>
                    <t t-out="format_alphanumeric(line.name, 1000)"/>
                    <t t-if="not line.name" t-out="'NO NAME'"/>
                </Descrizione>
                <Importo t-out="format_monetary(line.price_total, currency)"/>
                <DatiIVA>
                    <Imposta t-out="format_monetary(line.price_total - line.price_subtotal, currency)"/>
                </DatiIVA>
                <Natura t-if="line.tax_ids.l10n_it_exempt_reason" t-out="line.tax_ids.l10n_it_exempt_reason"/>
            </DatiBeniServizi>
        </template>

        <template id="account_invoice_it_simplified_FatturaPA_export">
            <t t-set="currency" t-value="record.currency_id or record.company_currency_id"/>
            <t t-set="bank" t-value="record.partner_bank_id"/>
            <p:FatturaElettronicaSemplificata t-att-versione="formato_trasmissione" xmlns:ds="http://www.w3.org/2000/09/xmldsig#" xmlns:p="http://ivaservizi.agenziaentrate.gov.it/docs/xsd/fatture/v1.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://ivaservizi.agenziaentrate.gov.it/docs/xsd/fatture/v1.0 http://www.fatturapa.gov.it/export/fatturazione/sdi/fatturapa/v1.0/Schema_del_file_xml_FatturaPA_versione_1.0.xsd">
                <FatturaElettronicaHeader>
                    <DatiTrasmissione>
                        <IdTrasmittente>
                            <IdPaese t-out="sender_info['country_code']"/>
                            <IdCodice t-out="sender_info['codice_fiscale'] or sender_info['vat']"/>
                        </IdTrasmittente>
                        <ProgressivoInvio t-out="format_alphanumeric(record.name.replace('/',''), -10)"/>
                        <FormatoTrasmissione t-out="formato_trasmissione"/>
                        <CodiceDestinatario t-if="buyer_info['pa_index']" t-out="buyer_info['pa_index']"/>
                        <PECDestinatario t-if="buyer.l10n_it_pec_email" t-out="buyer.l10n_it_pec_email"/>
                    </DatiTrasmissione>
                    <CedentePrestatore>
                        <IdFiscaleIVA>
                            <IdPaese t-out="seller_info['country_code']"/>
                            <IdCodice t-out="seller_info['vat']"/>
                        </IdFiscaleIVA>
                        <CodiceFiscale t-if="seller_info['codice_fiscale']" t-out="seller_info['codice_fiscale']"/>
                        <t t-if="seller_info['is_company']">
                            <Denominazione t-out="format_alphanumeric(seller.display_name, 80)"/>
                        </t>
                        <t t-else="">
                            <Nome t-out="format_alphanumeric(seller_info['first_name'], 60)"/>
                            <Cognome t-out="format_alphanumeric(seller_info['last_name'], 60)"/>
                        </t>
                        <t t-call="l10n_it_edi.account_invoice_it_FatturaPA_sede">
                            <t t-set="partner" t-value="seller"/>
                            <t t-set="partner_info" t-value="seller_info"/>
                        </t>
                        <RappresentanteFiscale t-if="seller.l10n_it_has_tax_representative">
                            <IdFiscaleIVA>
                                <IdPaese t-out="representative_info['country_code']"/>
                                <IdCodice t-out="representative_info['vat']"/>
                            </IdFiscaleIVA>
                            <Anagrafica>
                                <t t-if="representative_info['is_company']">
                                    <Denominazione t-out="format_alphanumeric(representative.display_name, 80)"/>
                                </t>
                                <t t-else="">
                                    <Nome t-out="format_alphanumeric(representative_info['first_name'], 60)"/>
                                    <Cognome t-out="format_alphanumeric(representative_info['last_name'], 60)"/>
                                </t>
                            </Anagrafica>
                        </RappresentanteFiscale>
                        <IscrizioneREA t-if="seller.l10n_it_has_eco_index">
                            <Ufficio t-out="seller.l10n_it_eco_index_office.code"/>
                            <NumeroREA t-out="format_alphanumeric(seller.l10n_it_eco_index_number)"/>
                            <CapitaleSociale t-if="seller.l10n_it_eco_index_share_capital != 0" t-out="format_numbers_two(seller.l10n_it_eco_index_share_capital)"/>
                            <SocioUnico t-if="seller.l10n_it_eco_index_sole_shareholder != 'NO'" t-out="seller.l10n_it_eco_index_sole_shareholder"/>
                            <StatoLiquidazione t-out="seller.l10n_it_eco_index_liquidation_state"/>
                        </IscrizioneREA>
                        <RegimeFiscale t-out="seller.l10n_it_tax_system"/>
                    </CedentePrestatore>
                    <CessionarioCommittente>
                        <IdentificativiFiscali>
                            <IdFiscaleIVA t-if="buyer_info['vat'] and buyer_info['in_eu']">
                                <IdPaese t-out="buyer_info['country_code']"/>
                                <IdCodice t-out="buyer_info['vat']"/>
                            </IdFiscaleIVA>
                            <CodiceFiscale t-if="buyer_info['codice_fiscale']" t-out="buyer_info['codice_fiscale']"/>
                        </IdentificativiFiscali>
                    </CessionarioCommittente>
                </FatturaElettronicaHeader>
                <FatturaElettronicaBody>
                    <DatiGenerali>
                        <DatiGeneraliDocumento>
                            <TipoDocumento t-out="document_type"/>
                            <Divisa t-out="currency.name"/>
                            <Data t-out="format_date(record.invoice_date)"/>
                            <Numero t-out="format_alphanumeric(record.name, -20)"/>
                        </DatiGeneraliDocumento>
                        <DatiFatturaRettificata t-if="record.move_type == 'out_refund' and record.reversed_entry_id">
                            <NumeroFR t-out="format_alphanumeric(record.reversed_entry_id.name, -20)"/>
                            <DataFR t-out="format_date(record.reversed_entry_id.invoice_date)"/>
                            <ElementiRettificati t-out="format_alphanumeric(record.ref, 1000)"/>
                        </DatiFatturaRettificata>
                    </DatiGenerali>
                    <t t-foreach="record.invoice_line_ids.filtered(lambda l: l.display_type not in ('line_note', 'line_section'))" t-as="line">
                        <t t-call="l10n_it_edi.account_invoice_line_it_simplified_FatturaPA"/>
                    </t>
                    <Allegati t-if="pdf">
                        <NomeAttachment t-out="format_alphanumeric(pdf_name, 60)"/>
                        <FormatoAttachment t-translation="off">PDF</FormatoAttachment>
                        <Attachment t-out="pdf"/>
                    </Allegati>
                </FatturaElettronicaBody>
            </p:FatturaElettronicaSemplificata>
        </template>
    </data>
</odoo>

```

## File: data\invoice_it_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

<template id="account_invoice_line_it_FatturaPA">
                <DettaglioLinee>
                    <NumeroLinea t-out="line_dict['line_number']"/>
                    <CodiceArticolo t-if="line.product_id.barcode">
                        <CodiceTipo t-translation="off">EAN</CodiceTipo>
                        <CodiceValore t-out="format_alphanumeric(line.product_id.barcode, 35)"/>
                    </CodiceArticolo>
                    <CodiceArticolo t-elif="line.product_id.default_code">
                        <CodiceTipo t-translation="off">INTERNAL</CodiceTipo>
                        <CodiceValore t-out="format_alphanumeric(line.product_id.default_code, 35)"/>
                    </CodiceArticolo>
                    <Descrizione t-out="format_alphanumeric(line_dict['description'], 1000)"/>
                    <Quantita t-out="format_numbers(abs(line.quantity))"/>
                    <UnitaMisura t-if="line.product_uom_id and line.product_uom_id.category_id != env.ref('uom.product_uom_categ_unit')"
                                 t-out="format_alphanumeric(line.product_uom_id.name, 10)"/>
                    <PrezzoUnitario t-out="'%.06f' % (line_dict['unit_price'])"/>
                    <ScontoMaggiorazione t-if="line.discount != 0">
                        <Tipo t-out="format_alphanumeric(line_dict['discount_type'])"/>
                        <Percentuale t-out="format_numbers(abs(line.discount))"/>
                    </ScontoMaggiorazione>
                    <PrezzoTotale t-out="format_monetary(line_dict['subtotal_price_eur'], currency)"/>
                    <AliquotaIVA t-if="vat_tax.amount_type == 'percent'" t-out="format_numbers(vat_tax.amount)"/>
                    <AliquotaIVA t-elif="vat_tax.amount_type != 'percent'" t-out="'0.00'"/>
                    <Natura t-if="vat_tax.l10n_it_exempt_reason" t-out="vat_tax.l10n_it_exempt_reason"/>
                    <AltriDatiGestionali t-if="conversion_rate">
                        <TipoDato t-translation="off">DIVISA</TipoDato>
                        <RiferimentoTesto t-out="format_alphanumeric(record.currency_id.name)"/>
                        <RiferimentoNumero t-out="'%.06f' % line_dict['subtotal_price']"/>
                    </AltriDatiGestionali>
                    <AltriDatiGestionali t-if="conversion_rate">
                        <TipoDato t-translation="off">CAMBIO</TipoDato>
                        <RiferimentoNumero t-out="conversion_rate"/>
                        <RiferimentoData t-out="format_date(record.invoice_date)"/>
                    </AltriDatiGestionali>
                </DettaglioLinee>
</template>

<template id="account_invoice_it_FatturaPA_export">
<p:FatturaElettronica t-att-versione="formato_trasmissione" xmlns:ds="http://www.w3.org/2000/09/xmldsig#" xmlns:p="http://ivaservizi.agenziaentrate.gov.it/docs/xsd/fatture/v1.2" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://ivaservizi.agenziaentrate.gov.it/docs/xsd/fatture/v1.2 http://www.fatturapa.gov.it/export/fatturazione/sdi/fatturapa/v1.2/Schema_del_file_xml_FatturaPA_versione_1.2.xsd">
    <FatturaElettronicaHeader>
        <DatiTrasmissione>
            <IdTrasmittente>
                <IdPaese t-out="sender_info['country_code']"/>
                <IdCodice t-out="sender_info['codice_fiscale'] or sender_info['vat']"/>
            </IdTrasmittente>
            <ProgressivoInvio t-out="format_alphanumeric(record.name.replace('/',''), -10)"/>
            <FormatoTrasmissione t-out="formato_trasmissione"/>
            <CodiceDestinatario t-out="buyer_info['pa_index']"/>
            <ContattiTrasmittente>
                <Telefono t-if="sender.phone" t-out="format_phone(sender.phone)"/>
                <Email t-if="sender.email" t-out="format_alphanumeric(sender.email, 256)"/>
            </ContattiTrasmittente>
            <PECDestinatario t-if="not is_self_invoice and buyer.l10n_it_pec_email" t-out="format_alphanumeric(buyer.l10n_it_pec_email, 256)"/>
        </DatiTrasmissione>
        <CedentePrestatore>
            <DatiAnagrafici>
                <IdFiscaleIVA>
                    <IdPaese t-out="seller_info['country_code']"/>
                    <IdCodice t-out="seller_info['vat']"/>
                </IdFiscaleIVA>
                <CodiceFiscale t-if="seller_info['codice_fiscale']" t-out="seller_info['codice_fiscale']"/>
                <Anagrafica>
                <t t-if="seller_info['is_company']">
                    <Denominazione t-out="format_alphanumeric(seller.display_name, 80)"/>
                </t>
                <t t-else="">
                    <Nome t-out="format_alphanumeric(seller_info['first_name'], 60)"/>
                    <Cognome t-out="format_alphanumeric(seller_info['last_name'], 60)"/>
                </t>
                </Anagrafica>
                <RegimeFiscale t-out="regime_fiscale"/>
            </DatiAnagrafici>
            <t t-call="l10n_it_edi.account_invoice_it_FatturaPA_sede">
                <t t-set="partner" t-value="seller"/>
                <t t-set="partner_info" t-value="seller_info"/>
            </t>
            <IscrizioneREA t-if="not is_self_invoice and company.l10n_it_has_eco_index">
                <Ufficio t-out="company.l10n_it_eco_index_office.code"/>
                <NumeroREA t-out="format_alphanumeric(company.l10n_it_eco_index_number)"/>
                <CapitaleSociale t-if="company.l10n_it_eco_index_share_capital != 0" t-out="format_numbers_two(company.l10n_it_eco_index_share_capital)"/>
                <SocioUnico t-if="company.l10n_it_eco_index_sole_shareholder != 'NO'" t-out="company.l10n_it_eco_index_sole_shareholder"/>
                <StatoLiquidazione t-out="company.l10n_it_eco_index_liquidation_state"/>
            </IscrizioneREA>
        </CedentePrestatore>
        <RappresentanteFiscale t-if="not is_self_invoice and representative">
            <DatiAnagrafici>
                <IdFiscaleIVA>
                    <IdPaese t-out="representative_info['country_code']"/>
                    <IdCodice t-out="representative_info['vat']"/>
                </IdFiscaleIVA>
                <CodiceFiscale t-if="representative_info['codice_fiscale']" t-out="representative_info['codice_fiscale']"/>
                <Anagrafica>
                <t t-if="representative_info['is_company']">
                    <Denominazione t-out="format_alphanumeric(representative.display_name, 80)"/>
                </t>
                <t t-else="">
                    <Nome t-out="format_alphanumeric(representative_info['first_name'], 60)"/>
                    <Cognome t-out="format_alphanumeric(representative_info['last_name'], 60)"/>
                </t>
                </Anagrafica>
            </DatiAnagrafici>
        </RappresentanteFiscale>
        <CessionarioCommittente>
            <DatiAnagrafici>
                <IdFiscaleIVA t-if="buyer_info['vat']">
                    <IdPaese t-out="buyer_info['country_code']"/>
                    <IdCodice t-out="buyer_info['vat']"/>
                </IdFiscaleIVA>
                <CodiceFiscale t-if="buyer_info['codice_fiscale']" t-out="buyer_info['codice_fiscale']"/>
                <Anagrafica>
                    <t t-if="buyer_info['is_company']">
                        <Denominazione t-out="format_alphanumeric(buyer.display_name, 80)"/>
                    </t>
                    <t t-else="">
                        <Nome t-out="format_alphanumeric(buyer_info['first_name'], 60)"/>
                        <Cognome t-out="format_alphanumeric(buyer_info['last_name'], 60)"/>
                    </t>
                </Anagrafica>
            </DatiAnagrafici>
            <t t-call="l10n_it_edi.account_invoice_it_FatturaPA_sede">
                <t t-set="partner" t-value="buyer"/>
                <t t-set="partner_info" t-value="buyer_info"/>
            </t>
        </CessionarioCommittente>
    </FatturaElettronicaHeader>
    <FatturaElettronicaBody>
        <DatiGenerali>
            <DatiGeneraliDocumento>
                <TipoDocumento t-out="document_type"/>
                <Divisa t-out="currency.name"/>
                <Data t-out="format_date(record.invoice_date)"/>
                <Numero t-out="format_alphanumeric(record.name, -20)"/>
                <DatiBollo t-if="record.l10n_it_stamp_duty">
                    <BolloVirtuale t-translation="off">SI</BolloVirtuale>
                    <ImportoBollo t-out="format_numbers(record.l10n_it_stamp_duty)"/>
                </DatiBollo>
                <ImportoTotaleDocumento t-out="format_monetary(document_total, currency)"/>
            </DatiGeneraliDocumento>
            <DatiOrdineAcquisto t-if="origin_document_type == 'purchase_order'">
                <t t-call="l10n_it_edi.account_invoice_FatturaPA_origin_document"/>
            </DatiOrdineAcquisto>
            <DatiOrdineAcquisto t-elif="record.ref and not record.reversed_entry_id">
                <IdDocumento t-out="format_alphanumeric(record.ref, 20)"/>
            </DatiOrdineAcquisto>
            <DatiContratto t-if="origin_document_type == 'contract'">
                <t t-call="l10n_it_edi.account_invoice_FatturaPA_origin_document"/>
            </DatiContratto>
            <DatiConvenzione t-if="origin_document_type == 'agreement'">
                <t t-call="l10n_it_edi.account_invoice_FatturaPA_origin_document"/>
            </DatiConvenzione>
            <t t-if="reconciled_moves">
                <DatiFattureCollegate t-foreach="reconciled_moves" t-as="reconciled_move">
                    <IdDocumento t-out="format_alphanumeric(reconciled_move.name, -20)"/>
                    <Data t-out="format_date(reconciled_move.invoice_date)"/>
                </DatiFattureCollegate>
            </t>
            <t t-elif="record.reversed_entry_id">
                <DatiFattureCollegate>
                    <IdDocumento t-out="format_alphanumeric(record.reversed_entry_id.name, -20)"/>
                    <Data t-out="format_date(record.reversed_entry_id.invoice_date)"/>
                </DatiFattureCollegate>
            </t>
            <DatiFattureCollegate t-foreach="downpayment_moves" t-as="downpayment_move">
                <IdDocumento t-out="format_alphanumeric(downpayment_move.name, -20)"/>
                <Data t-out="format_date(downpayment_move.invoice_date)"/>
            </DatiFattureCollegate>
            <DatiDDT t-if="record.l10n_it_ddt_id">
                <NumeroDDT t-out="format_alphanumeric(record.l10n_it_ddt_id.name, -20)"/>
                <DataDDT t-out="format_date(record.l10n_it_ddt_id.date)"/>
            </DatiDDT>
        </DatiGenerali>
        <DatiBeniServizi>
            <t t-foreach="invoice_lines" t-as="line_dict">
                <t t-set="line" t-value="line_dict['line']"/>
                <t t-set="vat_tax" t-value="line_dict['vat_tax']"/>
                <t t-call="l10n_it_edi.account_invoice_line_it_FatturaPA"/>
            </t>
            <t t-foreach="tax_lines" t-as="tax_line">
                <t t-set="tax" t-value="tax_line['tax']"/>
                <t t-set="exempt_reason" t-value="tax.l10n_it_exempt_reason"/>
                <DatiRiepilogo>
                    <AliquotaIVA t-out="format_numbers(tax.amount)"/>
                    <Natura t-if="exempt_reason" t-out="exempt_reason"/>
                    <Arrotondamento t-if="tax_line.get('rounding')" t-out="format_numbers(-tax_line['rounding'])"/>
                    <t t-if="rc_refund">
                        <ImponibileImporto t-out="format_monetary(balance_multiplicator * tax_line['base_amount'], currency)"/>
                        <Imposta t-out="format_monetary(balance_multiplicator * tax_line['tax_amount'], currency)"/>
                    </t>
                    <t t-else="">
                        <ImponibileImporto t-out="format_monetary(tax_line['base_amount'], currency)"/>
                        <Imposta t-out="format_monetary(tax_line['tax_amount'], currency)"/>
                    </t>
                    <EsigibilitaIVA t-out="tax_line['exigibility_code']"/>
                    <RiferimentoNormativo t-if="tax.l10n_it_law_reference" t-out="format_alphanumeric(tax.l10n_it_law_reference[:100])"/>
                </DatiRiepilogo>
            </t>
        </DatiBeniServizi>
        <DatiPagamento t-if="partner_bank and record.move_type != 'out_refund'">
            <t t-set="payments" t-value="record.line_ids.filtered(lambda line: line.account_id.account_type in ('asset_receivable', 'liability_payable'))"/>
            <CondizioniPagamento t-translation="off"><t t-if="len(payments) == 1">TP02</t><t t-else="">TP01</t></CondizioniPagamento>
            <t t-foreach="payments" t-as="payment">
                <DettaglioPagamento>
                    <ModalitaPagamento t-translation="off" t-out="payment_method"/>
                    <DataScadenzaPagamento t-out="format_date(payment.date_maturity)"/>
                    <ImportoPagamento t-out="format_monetary(abs(payment.amount_currency), currency)"/>
                    <IstitutoFinanziario t-if="partner_bank.bank_id" t-out="format_alphanumeric(partner_bank.bank_id.name, 80)"/>
                    <IBAN t-if="partner_bank.acc_type == 'iban'" t-out="partner_bank.sanitized_acc_number"/>
                    <BIC t-elif="partner_bank.acc_type == 'bank' and partner_bank.bank_id.bic" t-out="partner_bank.bank_id.bic"/>
                    <CodicePagamento t-if="record.payment_reference" t-out="format_alphanumeric(record.payment_reference, 60)"/>
                </DettaglioPagamento>
            </t>
        </DatiPagamento>
        <Allegati t-if="pdf">
            <NomeAttachment t-out="format_alphanumeric(pdf_name, 60)"/>
            <FormatoAttachment t-translation="off">PDF</FormatoAttachment>
            <Attachment t-out="pdf"/>
        </Allegati>
    </FatturaElettronicaBody>
</p:FatturaElettronica>
</template>

<template id="account_invoice_it_FatturaPA_sede">
            <Sede>
                <Indirizzo><t t-if="partner.street or partner.street2" t-out="format_address(partner.street, partner.street2, 60)"/></Indirizzo>
                <CAP><t t-out="partner_info['zip']"/></CAP>
                <Comune t-out="format_alphanumeric(partner.city, 60)"/>
                <Provincia t-if="partner_info['state_code']" t-out="format_alphanumeric(partner_info['state_code'], 2)"/>
                <Nazione t-out="partner_info['country_code']"/>
            </Sede>
</template>

<template id="account_invoice_FatturaPA_origin_document">
                <IdDocumento t-if="origin_document_name" t-esc="format_alphanumeric(origin_document_name, 20)"/>
                <Data t-if="origin_document_date" t-esc="format_date(origin_document_date)"/>
                <CodiceCUP t-if="cup" t-esc="format_alphanumeric(cup, 15)"/>
                <CodiceCIG t-if="cig" t-esc="format_alphanumeric(cig, 15)"/>
</template>

    </data>
</odoo>

```

## File: data\ir_cron.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_cron_l10n_it_edi_download_and_update" model="ir.cron">
        <field name="name">IT EDI: Receive invoices from the SdI</field>
        <field name="interval_number">1</field>
        <field name="interval_type">days</field>
        <field name="numbercall">-1</field>
        <field name="model_id" ref="account.model_account_move"/>
        <field name="code">model.cron_l10n_it_edi_download_and_update()</field>
        <field name="doall" eval="False"/>
        <field name="state">code</field>
    </record>
</odoo>

```

## File: models\account_edi_proxy_user.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import _, fields, models
from odoo.exceptions import UserError

_logger = logging.getLogger(__name__)


class AccountEdiProxyClientUser(models.Model):
    _inherit = 'account_edi_proxy_client.user'

    proxy_type = fields.Selection(selection_add=[('l10n_it_edi', 'Italian EDI')], ondelete={'l10n_it_edi': 'cascade'})

    def _get_proxy_urls(self):
        urls = super()._get_proxy_urls()
        urls['l10n_it_edi'] = {
            'demo': False,
            'prod': 'https://l10n-it-edi.api.odoo.com',
            'test': 'https://iap-services-test.odoo.com',
        }
        return urls

    def _get_proxy_identification(self, company, proxy_type):
        if proxy_type == 'l10n_it_edi':
            if not company.l10n_it_codice_fiscale:
                raise UserError(_('Please fill your codice fiscale to be able to receive invoices from FatturaPA'))
            return company.partner_id._l10n_it_edi_normalized_codice_fiscale()
        return super()._get_proxy_identification(company, proxy_type)

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from base64 import b64encode
from datetime import datetime
import logging
from lxml import etree
from markupsafe import escape
import uuid
from operator import itemgetter

from odoo import _, api, Command, fields, models
from odoo.addons.base.models.ir_qweb_fields import Markup, nl2br, nl2br_enclose
from odoo.addons.account_edi_proxy_client.models.account_edi_proxy_user import AccountEdiProxyError
from odoo.exceptions import UserError
from odoo.tools import float_compare, float_repr, cleanup_xml_node

_logger = logging.getLogger(__name__)


WAITING_STATES = ('being_sent', 'processing', 'forward_attempt')


# -------------------------------------------------------------------------
# XML tool functions
# -------------------------------------------------------------------------

def get_text(tree, xpath, many=False):
    texts = [el.text.strip() for el in tree.xpath(xpath) if el.text]
    return texts if many else texts[0] if texts else ''

def get_float(tree, xpath):
    try:
        return float(get_text(tree, xpath))
    except ValueError:
        return 0.0

def get_date(tree, xpath):
    """ Dates in FatturaPA are ISO 8601 date format, pattern '[-]CCYY-MM-DD[Z|(+|-)hh:mm]' """
    dt = get_datetime(tree, xpath)
    return dt.date() if dt else False

def get_datetime(tree, xpath):
    """ Datetimes in FatturaPA are ISO 8601 date format, pattern '[-]CCYY-MM-DDThh:mm:ss[Z|(+|-)hh:mm]'
        Python 3.7 -> 3.11 doesn't support 'Z'.
    """
    if (datetime_str := get_text(tree, xpath)):
        try:
            return datetime.fromisoformat(datetime_str.replace('Z', '+00:00'))
        except (ValueError, TypeError):
            return False
    return False


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_it_edi_state = fields.Selection(
        string="SDI State",
        selection=[
            ('being_sent', 'Being Sent To SdI'),
            ('requires_user_signature', 'Requires user signature'),
            ('processing', 'SdI Processing'),
            ('rejected', 'SdI Rejected'),
            ('forwarded', 'SdI Accepted, Forwarded to Partner'),
            ('forward_failed', 'SdI Accepted, Forward to Partner Failed'),
            ('forward_attempt', 'SdI Accepted, Forwarding to Partner'),
            ('accepted_by_pa_partner', 'SdI Accepted, Accepted by the PA Partner'),
            ('rejected_by_pa_partner', 'SdI Accepted, Rejected by the PA Partner'),
            ('accepted_by_pa_partner_after_expiry', 'SdI Accepted, PA Partner Expired Terms'),
        ],
        copy=False, tracking=True,
        help="This state is updated by default, but you can force the value. ",
    )
    l10n_it_edi_header = fields.Html(
        help='User description of the current state, with hints to make the flow progress',
        readonly=True,
        copy=False,
    )
    l10n_it_edi_transaction = fields.Char(copy=False, string="FatturaPA Transaction")
    l10n_it_edi_attachment_file = fields.Binary(copy=False, attachment=True)
    l10n_it_edi_attachment_id = fields.Many2one(
        comodel_name='ir.attachment',
        string="FatturaPA Attachment",
        compute=lambda self: self._compute_linked_attachment_id('l10n_it_edi_attachment_id', 'l10n_it_edi_attachment_file'),
        depends=['l10n_it_edi_attachment_file'],
    )
    l10n_it_edi_is_self_invoice = fields.Boolean(compute="_compute_l10n_it_edi_is_self_invoice")
    l10n_it_stamp_duty = fields.Float(string="Dati Bollo")
    l10n_it_ddt_id = fields.Many2one('l10n_it.ddt', string='DDT', copy=False)

    l10n_it_origin_document_type = fields.Selection(
        string="Origin Document Type",
        selection=[('purchase_order', 'Purchase Order'), ('contract', 'Contract'), ('agreement', 'Agreement')],
        copy=False)
    l10n_it_origin_document_name = fields.Char(
        string="Origin Document Name",
        copy=False)
    l10n_it_origin_document_date = fields.Date(
        string="Origin Document Date",
        copy=False)
    l10n_it_cig = fields.Char(
        string="CIG",
        copy=False,
        help="Tender Unique Identifier")
    l10n_it_cup = fields.Char(
        string="CUP",
        copy=False,
        help="Public Investment Unique Identifier")
    # Technical field for showing the above fields or not
    l10n_it_partner_pa = fields.Boolean(compute='_compute_l10n_it_partner_pa')

    # -------------------------------------------------------------------------
    # Computes
    # -------------------------------------------------------------------------

    @api.depends('commercial_partner_id.l10n_it_pa_index', 'company_id')
    def _compute_l10n_it_partner_pa(self):
        for move in self:
            partner = move.commercial_partner_id
            move.l10n_it_partner_pa = partner and (partner._l10n_it_edi_is_public_administration() or len(partner.l10n_it_pa_index or '') == 7)

    @api.depends('move_type', 'line_ids.tax_tag_ids')
    def _compute_l10n_it_edi_is_self_invoice(self):
        """
            Italian EDI requires Vendor bills coming from EU countries to be sent as self-invoices.
            We recognize these cases based on the taxes that target the VJ tax grids, which imply
            the use of VAT External Reverse Charge.
        """
        purchases = self.filtered(lambda m: m.is_purchase_document())
        others = self - purchases
        for move in others:
            move.l10n_it_edi_is_self_invoice = False
        if purchases:
            it_tax_report_vj_lines = self.env['account.report.line'].sudo().search([
                ('report_id.country_id.code', '=', 'IT'),
                ('code', '=like', 'VJ%')
            ])
            vj_lines_tags = it_tax_report_vj_lines.expression_ids._get_matching_tags()
            for move in purchases:
                invoice_lines_tags = move.line_ids.tax_tag_ids
                ids_intersection = set(invoice_lines_tags.ids) & set(vj_lines_tags.ids)
                move.l10n_it_edi_is_self_invoice = bool(ids_intersection)

    def _l10n_it_edi_exempt_reason_tag_mapping(self):
        return {
            "N3.2": "VJ3",
            "N3.3": "VJ1",
            "N6.1": "VJ6",
            "N6.2": "VJ7",
            "N6.3": "VJ12",
            "N6.4": "VJ13",
            "N6.5": "VJ14",
            "N6.6": "VJ15",
            "N6.7": "VJ16",
            "N6.8": "VJ17",
        }

    # -------------------------------------------------------------------------
    # Overrides
    # -------------------------------------------------------------------------

    @api.depends('l10n_it_edi_transaction')
    def _compute_show_reset_to_draft_button(self):
        # EXTENDS 'account'
        super()._compute_show_reset_to_draft_button()
        for move in self:
            move.show_reset_to_draft_button = not move.l10n_it_edi_transaction and move.show_reset_to_draft_button

    def _get_edi_decoder(self, file_data, new=False):
        # EXTENDS 'account'
        if file_data['type'] == 'l10n_it_edi':
            return self._l10n_it_edi_import_invoice
        return super()._get_edi_decoder(file_data, new=new)

    def _post(self, soft=True):
        # EXTENDS 'account'
        self.write({'l10n_it_edi_header': False})
        return super()._post(soft)

    def _extend_with_attachments(self, attachments, new=False):
        result = False
        # Prediction is an enterprise feature.
        if self._is_prediction_enabled():
            # Italy needs a custom order in prediction, since prediction generally deduces taxes
            # from products, while in Italian EDI, taxes are generally explicited in the XML file
            # while the product may not be labelled exactly the same as in the database
            l10n_it_attachments = attachments.filtered(lambda rec: rec._is_l10n_it_edi_import_file())
            if l10n_it_attachments:
                attachments = attachments - l10n_it_attachments
                result = super(AccountMove, self.with_context(disable_onchange_name_predictive=True))._extend_with_attachments(l10n_it_attachments, new)
        return result or super()._extend_with_attachments(attachments, new)

    # -------------------------------------------------------------------------
    # Business actions
    # -------------------------------------------------------------------------

    def action_l10n_it_edi_send(self):
        """ Checks that the invoice data is coherent.
            Attaches the XML file to the invoice.
            Sends the invoice to the SdI.
        """
        self.ensure_one()

        if self._l10n_it_edi_export_data_check():
            raise UserError(_("The invoices you're trying to send have incomplete or incorrect data, please verify before sending."))

        attachment_vals = self._l10n_it_edi_get_attachment_values(pdf_values=None)
        self.env['ir.attachment'].create(attachment_vals)
        self.invalidate_recordset(fnames=['l10n_it_edi_attachment_id', 'l10n_it_edi_attachment_file'])
        self.message_post(attachment_ids=self.l10n_it_edi_attachment_id.ids)
        self._l10n_it_edi_send({self: attachment_vals})
        self.is_move_sent = True

    def action_check_l10n_it_edi(self):
        self.ensure_one()
        if not self.l10n_it_edi_transaction and self.l10n_it_edi_state not in WAITING_STATES:
            raise UserError(_("This move is not waiting for updates from the SdI."))
        if self.l10n_it_edi_state == 'being_sent':
            return {'type': 'ir.actions.client', 'tag': 'reload'}
        self._l10n_it_edi_update_send_state()

    def button_draft(self):
        # EXTENDS 'account'
        for move in self:
            move.l10n_it_edi_state = False
        return super().button_draft()

    # -------------------------------------------------------------------------
    # Helpers
    # -------------------------------------------------------------------------

    def _l10n_it_edi_ready_for_xml_export(self):
        self.ensure_one()
        return (
            self.state == 'posted'
            and self.company_id.account_fiscal_country_id.code == 'IT'
            and self.journal_id.type == 'sale'
            and self.l10n_it_edi_state in (False, 'rejected')
        )

    def _l10n_it_edi_get_line_values(self, reverse_charge_refund=False, is_downpayment=False, convert_to_euros=True):
        """ Returns a list of dictionaries passed to the template for the invoice lines (DettaglioLinee)
        """
        invoice_lines = []
        lines = self.invoice_line_ids.filtered(lambda l: l.display_type not in ('line_note', 'line_section'))
        base_lines = [invl._convert_to_tax_base_line_dict() for invl in lines]
        for num, line_dict in enumerate(base_lines):
            if reverse_charge_refund:
                line_dict['price_subtotal'] = -line_dict['price_subtotal']

            line_dict['subtotal_price_eur'] = line_dict['price_subtotal']/line_dict['rate'] if convert_to_euros else line_dict['price_subtotal']

            if line_dict['discount'] != 100.0 and line_dict['quantity']:
                line_dict['price_unit'] = line_dict['subtotal_price_eur'] / ((1 - (line_dict['discount'] or 0.0) / 100.0) * abs(line_dict['quantity']))
            else:
                line_dict['price_unit'] = line_dict['price_unit'] / line_dict['rate'] if convert_to_euros else line_dict['price_unit']

            line = line_dict['record']
            description = line.name

            # Down payment lines:
            # If there was a down paid amount that has been deducted from this move,
            # we need to put a reference to the down payment invoice in the DatiFattureCollegate tag
            downpayment_moves = self.env['account.move']
            if not is_downpayment and line.price_subtotal < 0:
                downpayment_moves = line._get_downpayment_lines().mapped("move_id")
                if downpayment_moves:
                    downpayment_moves_description = ', '.join(m.name for m in downpayment_moves)
                    sep = ', ' if description else ''
                    description = f"{description}{sep}{downpayment_moves_description}"

            invoice_lines.append({
                'line': line,
                'line_number': num + 1,
                'description': description or 'NO NAME',
                'subtotal_price_eur': line_dict['currency'].round(line_dict['subtotal_price_eur']),
                'subtotal_price': line_dict['currency'].round(line_dict['price_subtotal']),
                'unit_price': line_dict['price_unit'],
                'discount_amount': 0,  # kept because we didn't do a get in the line we removed from the template
                'vat_tax': line.tax_ids.flatten_taxes_hierarchy().filtered(lambda t: t._l10n_it_filter_kind('vat') and t.amount >= 0),
                'downpayment_moves': downpayment_moves,
                'discount_type': (
                    'SC' if line.discount > 0
                    else 'MG' if line.discount < 0
                    else False
                )
            })
        return invoice_lines

    def _l10n_it_edi_get_tax_values(self, tax_details):
        """ Returns a list of dictionaries passed to the template for the invoice lines (DatiRiepilogo)
        """
        tax_lines = []
        for _tax_name, tax_dict in tax_details['tax_details'].items():
            # The assumption is that the company currency is EUR.
            base_amount = tax_dict['base_amount']
            tax_amount = tax_dict['tax_amount']
            tax = tax_dict['tax']
            tax_rate = tax.amount
            tax_exigibility_code = (
                'S' if tax._l10n_it_is_split_payment()
                else 'D' if tax.tax_exigibility == 'on_payment'
                else 'I' if tax.tax_exigibility == 'on_invoice'
                else False
            )
            expected_base_amount = tax_amount * 100 / tax_rate if tax_rate else False
            tax = tax_dict['tax']
            # Constraints within the edi make local rounding on price included taxes a problem.
            # To solve this there is a <Arrotondamento> or 'rounding' field, such that:
            #   taxable base = sum(taxable base for each unit) + Arrotondamento
            if tax.price_include and tax.amount_type == 'percent':
                if expected_base_amount and float_compare(base_amount, expected_base_amount, 2):
                    tax_dict['rounding'] = base_amount - (tax_amount * 100 / tax_rate)
                    tax_dict['base_amount'] = base_amount - tax_dict['rounding']

            tax_line_dict = {
                'tax': tax,
                'rounding': tax_dict.get('rounding', False),
                'base_amount': tax_dict['base_amount'],
                'tax_amount': tax_dict['tax_amount'],
                'exigibility_code': tax_exigibility_code,
            }
            tax_lines.append(tax_line_dict)
        return tax_lines

    def _l10n_it_edi_filter_tax_details(self, line, tax_values):
        """Filters tax details to only include the positive amounted lines regarding VAT taxes."""
        repartition_line = tax_values['tax_repartition_line']
        return (repartition_line.factor_percent >= 0 and repartition_line.tax_id.amount >= 0)

    def _get_l10n_it_amount_split_payment(self):
        self.ensure_one()
        if not self.is_sale_document(False):
            return 0.0
        sign = -1 if self.move_type == "out_invoice" else 1
        return sum(sign * line.balance for line in self.line_ids.filtered(lambda l: l.tax_line_id and l.tax_line_id._l10n_it_is_split_payment()))

    def _l10n_it_edi_get_values(self, pdf_values=None):
        self.ensure_one()

        # Flags
        is_self_invoice = self.l10n_it_edi_is_self_invoice
        document_type = self._l10n_it_edi_get_document_type()

        # Represent if the document is a reverse charge refund in a single variable
        reverse_charge = document_type in ['TD16', 'TD17', 'TD18', 'TD19']
        is_downpayment = document_type in ['TD02']
        reverse_charge_refund = self.move_type == 'in_refund' and reverse_charge
        convert_to_euros = self.currency_id.name != 'EUR'

        tax_details = self._prepare_invoice_aggregated_taxes(filter_tax_values_to_apply=self._l10n_it_edi_filter_tax_details)

        company = self.company_id
        partner = self.commercial_partner_id
        sender = company
        buyer = partner if not is_self_invoice else company
        seller = company if not is_self_invoice else partner
        sender_info_values = company.partner_id._l10n_it_edi_get_values()
        buyer_info_values = (partner if not is_self_invoice else company.partner_id)._l10n_it_edi_get_values()
        seller_info_values = (company.partner_id if not is_self_invoice else partner)._l10n_it_edi_get_values()
        representative_info_values = company.l10n_it_tax_representative_partner_id._l10n_it_edi_get_values()

        if self._l10n_it_edi_is_simplified_document_type(document_type):
            formato_trasmissione = "FSM10"
        elif partner._l10n_it_edi_is_public_administration():
            formato_trasmissione = "FPA12"
        else:
            formato_trasmissione = "FPR12"

        # Self-invoices are technically -100%/+100% repartitioned
        # but functionally need to be exported as 100%
        document_total = self.amount_total
        if is_self_invoice:
            document_total += sum([abs(v['tax_amount_currency']) for k, v in tax_details['tax_details'].items()])
            if reverse_charge_refund:
                document_total = -abs(document_total)

        split_payment_amount = self._get_l10n_it_amount_split_payment()
        if split_payment_amount:
            document_total += split_payment_amount

        # Reference line for finding the conversion rate used in the document
        conversion_rate = float_repr(
            abs(self.amount_total / self.amount_total_signed), precision_digits=5,
        ) if convert_to_euros and self.invoice_line_ids else None

        invoice_lines = self._l10n_it_edi_get_line_values(reverse_charge_refund, is_downpayment, convert_to_euros)
        tax_lines = self._l10n_it_edi_get_tax_values(tax_details)

        # Reduce downpayment views to a single recordset
        downpayment_moves = [l.get('downpayment_moves', self.env['account.move']) for l in invoice_lines]
        downpayment_moves = self.browse(move.id for moves in downpayment_moves for move in moves)

        return {
            'record': self,
            'company': company,
            'partner': partner,
            'sender': sender,
            'buyer': buyer,
            'seller': seller,
            'representative': company.l10n_it_tax_representative_partner_id,
            'sender_info': sender_info_values,
            'buyer_info': buyer_info_values,
            'seller_info': seller_info_values,
            'representative_info': representative_info_values,
            'origin_document_type': self.l10n_it_origin_document_type,
            'origin_document_name': self.l10n_it_origin_document_name,
            'origin_document_date': self.l10n_it_origin_document_date,
            'cig': self.l10n_it_cig,
            'cup': self.l10n_it_cup,
            'currency': self.currency_id or self.company_currency_id if not convert_to_euros else self.env.ref('base.EUR'),
            'document_total': document_total,
            'regime_fiscale': company.l10n_it_tax_system if not is_self_invoice else 'RF18',
            'is_self_invoice': is_self_invoice,
            'partner_bank': self.partner_bank_id,
            'formato_trasmissione': formato_trasmissione,
            'document_type': document_type,
            'payment_method': 'MP05',
            'tax_details': tax_details,
            'downpayment_moves': downpayment_moves,
            'reconciled_moves': self._get_reconciled_invoices(),
            'rc_refund': reverse_charge_refund,
            'invoice_lines': invoice_lines,
            'tax_lines': tax_lines,
            'conversion_rate': conversion_rate,
            'balance_multiplicator': -1 if self.is_inbound() else 1,
            'abs': abs,
            'pdf_name': pdf_values['name'] if pdf_values else False,
            'pdf': b64encode(pdf_values['raw']).decode() if pdf_values else False,
        }

    def _l10n_it_edi_services_or_goods(self):
        """
            Services and goods have different tax grids when VAT is Reverse Charged, and they can't
            be mixed in the same invoice, because the TipoDocumento depends on which which kind
            of product is bought and it's unambiguous.
        """
        self.ensure_one()
        scopes = []
        for line in self.invoice_line_ids.filtered(lambda l: l.display_type not in ('line_note', 'line_section')):
            tax_ids_with_tax_scope = line.tax_ids.filtered(lambda x: x.tax_scope)
            if tax_ids_with_tax_scope:
                scopes += tax_ids_with_tax_scope.mapped('tax_scope')
            else:
                scopes.append(line.product_id and line.product_id.type == 'service' and 'service' or 'consu')

        if set(scopes) == {'consu', 'service'}:
            return "both"
        return scopes and scopes.pop()

    def _l10n_it_edi_goods_in_italy(self):
        """
            There is a specific TipoDocumento (Document Type TD19) and tax grid (VJ3) for goods
            that are phisically in Italy but are in a VAT deposit, meaning that the goods
            have not passed customs.
        """
        self.ensure_one()
        invoice_lines_tags = self.line_ids.tax_tag_ids
        it_tax_report_vj3_lines = self.env['account.report.line'].search([
            ('report_id.country_id.code', '=', 'IT'),
            ('code', '=', 'VJ3'),
        ])
        vj3_lines_tags = it_tax_report_vj3_lines.expression_ids._get_matching_tags()
        return bool(invoice_lines_tags & vj3_lines_tags)

    def _l10n_it_edi_is_simplified(self):
        """
            Simplified Invoices are a way for the invoice issuer to create an invoice with limited data.
            Example: a consultant goes to the restaurant and wants the invoice instead of the receipt,
            to be able to deduct the expense from his Taxes. The Italian State allows the restaurant
            to issue a Simplified Invoice with the VAT number only, to speed up times, instead of
            requiring the address and other informations about the buyer.
            Only invoices under the threshold of 400 Euroes are allowed, to avoid this tool
            be abused for bigger transactions, that would enable less transparency to tax institutions.
        """
        self.ensure_one()
        template_reference = self.env.ref('l10n_it_edi.account_invoice_it_simplified_FatturaPA_export', raise_if_not_found=False)
        buyer = self.commercial_partner_id
        checks = ['partner_address_missing', 'partner_vat_codice_fiscale_missing']
        return bool(
            template_reference
            and not self.l10n_it_edi_is_self_invoice
            and list(buyer._l10n_it_edi_export_check(checks).keys()) == ['partner_address_missing']
            and (not buyer.country_id or buyer.country_id.code == 'IT')
            and (buyer.l10n_it_codice_fiscale or (buyer.vat and (buyer.vat[:2].upper() == 'IT' or buyer.vat[:2].isdecimal())))
            and self.amount_total <= 400
        )

    def _l10n_it_edi_is_professional_fees(self):
        """
            This function returns a boolean value based on the comparison of the lines values with a product.
            If one line has the tag for professional fee then we return True
        """
        self.ensure_one()
        professional_fee_tag = self.env.ref('l10n_it_edi.l10n_it_edi_professional_fees_tag', raise_if_not_found=False)
        if not professional_fee_tag:
            return False

        return any(
            professional_fee_tag.id in line.account_id.tag_ids.ids
            for line in self.invoice_line_ids
            if line.display_type not in ('line_note', 'line_section')
        )

    def _l10n_it_edi_features_for_document_type_selection(self):
        """ Returns a dictionary of features to be compared with the TDxx FatturaPA
            document type requirements. """
        partner_values = self.commercial_partner_id._l10n_it_edi_get_values()
        services_or_goods = self._l10n_it_edi_services_or_goods()
        return {
            'move_types': self.move_type,
            'partner_in_eu': partner_values.get('in_eu', False),
            'partner_country_code': partner_values.get('country_code', False),
            'simplified': self._l10n_it_edi_is_simplified(),
            'self_invoice': self.l10n_it_edi_is_self_invoice,
            'tax_tags': {tag for tag in self.line_ids.tax_tag_ids.mapped(lambda x: (x.name or '').upper().replace("+", "").replace("-", "")) if tag},
            'downpayment': self._is_downpayment(),
            'services_or_goods': services_or_goods,
            'goods_in_italy': services_or_goods == 'consu' and self._l10n_it_edi_goods_in_italy(),
            'professional_fees': self._l10n_it_edi_is_professional_fees(),
        }

    def _l10n_it_edi_document_type_mapping(self):
        """ Returns a dictionary with the required features for every TDxx FatturaPA document type """
        return {
            'TD01': {'move_types': ['out_invoice'],
                     'import_type': 'in_invoice',
                     'self_invoice': False,
                     'simplified': False,
                     'downpayment': False,
                     'professional_fees': False},
            'TD02': {'move_types': ['out_invoice'],
                     'import_type': 'in_invoice',
                     'self_invoice': False,
                     'simplified': False,
                     'downpayment': True,
                     'professional_fees': False},
            'TD03': {'move_types': ['out_invoice'],
                     'import_type': 'in_invoice',
                     'self_invoice': False,
                     'simplified': False,
                     'downpayment': True,
                     'professional_fees': True},
            'TD04': {'move_types': ['out_refund'],
                     'import_type': 'in_refund',
                     'self_invoice': False,
                     'simplified': False},
            'TD05': {'move_types': ['out_refund'],
                     'import_type': 'in_refund',
                     'self_invoice': False,
                     'simplified': False},
            'TD06': {'move_types': ['out_invoice'],
                     'import_type': 'in_invoice',
                     'self_invoice': False,
                     'simplified': False,
                     'downpayment': False,
                     'professional_fees': True},
            'TD07': {'move_types': ['out_invoice'],
                     'import_type': 'in_invoice',
                     'self_invoice': False,
                     'simplified': True},
            'TD08': {'move_types': ['out_refund'],
                     'import_type': 'in_refund',
                     'self_invoice': False,
                     'simplified': True},
            'TD09': {'move_types': ['out_invoice'],
                     'import_type': 'in_invoice',
                     'self_invoice': False,
                     'simplified': True},
            'TD28': {'move_types': ['in_invoice', 'in_refund'],
                     'import_type': 'in_invoice',
                     'simplified': False,
                     'self_invoice': True,
                     'partner_country_code': "SM"},
            'TD16': {'move_types': ['in_invoice', 'in_refund'],
                     'import_type': 'in_invoice',
                     'simplified': False,
                     'self_invoice': True,
                     'tax_tags': {'VJ6', 'VJ7', 'VJ8', 'VJ12', 'VJ13', 'VJ14', 'VJ15', 'VJ16', 'VJ17'}},
            'TD17': {'move_types': ['in_invoice', 'in_refund'],
                     'import_type': 'in_invoice',
                     'simplified': False,
                     'self_invoice': True,
                     'services_or_goods': "service",
                     'tax_tags': {'VJ3'}},
            'TD18': {'move_types': ['in_invoice', 'in_refund'],
                     'import_type': 'in_invoice',
                     'simplified': False,
                     'self_invoice': True,
                     'services_or_goods': "consu",
                     'goods_in_italy': False,
                     'partner_in_eu': True,
                     'tax_tags': {'VJ9'}},
            'TD19': {'move_types': ['in_invoice', 'in_refund'],
                     'import_type': 'in_invoice',
                     'simplified': False,
                     'self_invoice': True,
                     'services_or_goods': "consu",
                     'goods_in_italy': True,
                     'tax_tags': {'VJ3'}},
        }

    def _l10n_it_edi_get_document_type(self):
        """ Compare the features of the invoice to the requirements of each Document Type (TDxx)
        FatturaPA until you find a valid one. """

        def compare(actual_values, expected_values):
            """ Compare a single entry from the invoice features with the one of the document_type """
            if isinstance(expected_values, set | list | tuple):
                # i.e. When we compare actual tax_tags from the invoice with expected tags, we see if there is at least one in common
                if isinstance(actual_values, set):
                    return actual_values & set(expected_values)
                # i.e. When we compare the move_type with the available ones, these can be more than one
                return actual_values in expected_values
            # We compare other features directly, one on one
            return actual_values == expected_values

        invoice_features = self._l10n_it_edi_features_for_document_type_selection()
        for document_type_code, document_type_features in self._l10n_it_edi_document_type_mapping().items():
            # By using a generator instead of a list, we can avoid some comparisons
            if all(compare(invoice_values, document_type_features[k]) for k, invoice_values in invoice_features.items() if k in document_type_features):
                return document_type_code
        return False

    def _l10n_it_edi_is_simplified_document_type(self, document_type):
        mapping = self._l10n_it_edi_document_type_mapping()
        return mapping.get(document_type, {}).get('simplified', False)

    @api.model
    def _l10n_it_buyer_seller_info(self):
        return {
            'buyer': {
                'role': 'buyer',
                'section_xpath': '//CessionarioCommittente',
                'vat_xpath': '//CessionarioCommittente//IdCodice',
                'codice_fiscale_xpath': '//CessionarioCommittente//CodiceFiscale',
                'type_tax_use_domain': [('type_tax_use', '=', 'purchase')],
            },
            'seller': {
                'role': 'seller',
                'section_xpath': '//CedentePrestatore',
                'vat_xpath': '//CedentePrestatore//IdCodice',
                'codice_fiscale_xpath': '//CedentePrestatore//CodiceFiscale',
                'type_tax_use_domain': [('type_tax_use', '=', 'sale')],
            },
        }

    # -------------------------------------------------------------------------
    # EDI: Import
    # -------------------------------------------------------------------------

    def cron_l10n_it_edi_download_and_update(self):
        """ Crons run with sudo(), with empty recordset. Remember that. """
        retrigger = False
        for proxy_user in self.env['account_edi_proxy_client.user'].search([('proxy_type', '=', 'l10n_it_edi')]):
            proxy_user = proxy_user.with_company(proxy_user.company_id)
            if proxy_user.edi_mode != 'demo':
                moves_to_check = self.search([
                    ('company_id', '=', proxy_user.company_id.id),
                    ('l10n_it_edi_transaction', '!=', False),
                    ('l10n_it_edi_state', 'in', WAITING_STATES)
                ])
                if moves_to_check:
                    moves_to_check._l10n_it_edi_update_send_state()
                retrigger = retrigger or self._l10n_it_edi_download_invoices(proxy_user)

        # Retrigger download if there are still some on the server
        if retrigger:
            _logger.info('Retriggering "Receive invoices from the SdI"...')
            self.env.ref('l10n_it_edi.ir_cron_l10n_it_edi_download_and_update')._trigger()

    def _l10n_it_edi_download_invoices(self, proxy_user):
        """ Check the proxy for incoming invoices for a specified proxy user.
            :return: True if there remain some invoices on the server to be downloaded, False otherwise.
        """
        server_url = proxy_user._get_server_url()

        # Download invoices
        invoices_data = {}
        try:
            invoices_data = proxy_user._make_request(f'{server_url}/api/l10n_it_edi/1/in/RicezioneInvoice',
                params={'recipient_codice_fiscale': proxy_user.company_id.l10n_it_codice_fiscale})
        except AccountEdiProxyError as e:
            _logger.error('Error while receiving invoices from the SdI: %s', e)
            return False

        # Process the downloaded invoices
        processed = self._l10n_it_edi_process_downloads(invoices_data, proxy_user)
        if processed['proxy_acks']:
            try:
                proxy_user._make_request(
                    f'{server_url}/api/l10n_it_edi/1/ack',
                    params={'transaction_ids': processed['proxy_acks']})
            except AccountEdiProxyError as e:
                _logger.error('Error while receiving file from the SdI: %s', e)

        return processed['retrigger']

    def _l10n_it_edi_process_downloads(self, invoices_data, proxy_user):
        """ Every attachment will be committed if stored succesfully.
            Also moves will be committed one by one, even if imported incorrectly.
        """
        proxy_acks = []
        retrigger = False
        moves = self.env['account.move']

        for id_transaction, invoice_data in invoices_data.items():

            # The IAP server has a maximum number of documents it can send.
            # If that maximum is reached, then we search for more
            # by re-triggering the download cron, avoiding the timeout.
            current_num = invoice_data.get('current_num', 0)
            max_num = invoice_data.get('max_num', 0)
            retrigger = retrigger or current_num == max_num > 0

            # `_l10n_it_edi_create_move_from_attachment` will create an empty move
            # then try and fill it with the content imported from the attachment.
            # Should the import fail, thanks to try..except and savepoint,
            # we will anyway end up with an empty `in_invoice` with the attachment posted on it.
            if move := self.with_company(proxy_user.company_id)._l10n_it_edi_create_move_with_attachment(
                invoice_data['filename'],
                invoice_data['file'],
                invoice_data['key'],
                proxy_user,
            ):
                self.env.cr.commit()
                moves |= move
            proxy_acks.append(id_transaction)

        # Extend created moves with the related attachments and commit
        for move in moves:
            move._extend_with_attachments(move.l10n_it_edi_attachment_id, new=True)
            self.env.cr.commit()

        return {"retrigger": retrigger, "proxy_acks": proxy_acks}

    def _l10n_it_edi_create_move_with_attachment(self, filename, content, key, proxy_user):
        """ Creates a move and save an incoming file from the SdI as its attachment.

            :param filename:       name of the file to be saved.
            :param content:        encrypted content of the file to be saved.
            :param key:            key to decrypt the file.
            :param proxy_user:     the AccountEdiProxyClientUser to use for decrypting the file
        """

        # Name should be unique per company, the invoice already exists
        Attachment = self.env['ir.attachment'].sudo().with_company(proxy_user.company_id)
        if Attachment.search_count([
            ('name', '=', filename),
            ('res_model', '=', 'account.move'),
            ('res_field', '=', 'l10n_it_edi_attachment_file'),
            ('company_id', '=', proxy_user.company_id.id),
        ], limit=1):
            _logger.warning('E-invoice already exists: %s', filename)
            return False

        # Decrypt with the server key
        try:
            decrypted_content = proxy_user._decrypt_data(content, key)
        except Exception as e: # noqa: BLE001
            _logger.warning("Cannot decrypt e-invoice: %s, %s", filename, e)
            return False

        # Create the attachment, an empty move, then attach the two and commit
        move = self.with_company(proxy_user.company_id).create({})
        attachment = Attachment.create({
            'name': filename,
            'raw': decrypted_content,
            'type': 'binary',
            'res_model': 'account.move',
            'res_id': move.id,
            'res_field': 'l10n_it_edi_attachment_file'
        })
        move.with_context(
            account_predictive_bills_disable_prediction=True,
            no_new_invoice=True,
        ).message_post(attachment_ids=attachment.ids)

        return move

    def _l10n_it_edi_search_partner(self, company, vat, codice_fiscale, email):
        for domain in [vat and [('vat', 'ilike', vat)],
                       codice_fiscale and [('l10n_it_codice_fiscale', 'in', ('IT' + codice_fiscale, codice_fiscale))],
                       email and ['|', ('email', '=', email), ('l10n_it_pec_email', '=', email)]]:
            if domain and (partner := self.env['res.partner'].search(
                    domain + self.env['res.partner']._check_company_domain(company), limit=1)):
                return partner
        return self.env['res.partner']

    def _l10n_it_edi_search_tax_for_import(self, company, percentage, extra_domain=None, l10n_it_exempt_reason=None):
        """ Returns the VAT, Withholding or Pension Fund tax that suits the conditions given
            and matches the percentage found in the XML for the company. """

        domain = [
            *self.env['account.tax']._check_company_domain(company),
            ('amount_type', '=', 'percent'),
        ] + (extra_domain or [])

        # We suppose we're importing a file that comes in as a customer invoice where the sale tax will be 0%.
        # To retrieve the correct purchase tax, we examine the sale tax's l10n_it_exempt_reason.
        # We determine whether the l10n_it_exempt_reason is specific to reverse charge.
        reversed_tax_tag = self._l10n_it_edi_exempt_reason_tag_mapping().get(l10n_it_exempt_reason, '')
        if not reversed_tax_tag:
            # Normal VAT taxes have a known percentage and generally have all positive repartition lines
            domain += [('amount', '=', percentage), ('l10n_it_exempt_reason', '=', l10n_it_exempt_reason)]
            taxes = self.env['account.tax'].search(domain).filtered(
                lambda tax: all(rep_line.factor_percent >= 0 for rep_line in tax.invoice_repartition_line_ids))
        else:
            # In case of reverse charge, the purchase tax has a negative repartition line.
            domain += [('invoice_repartition_line_ids.tag_ids.name', '=', f'+{reversed_tax_tag.lower()}')]
            taxes = self.env['account.tax'].search(domain, order="amount desc").filtered(
                lambda tax: any(rep_line.factor_percent < 0 for rep_line in tax.invoice_repartition_line_ids))

        return taxes[0] if taxes else taxes

    def _l10n_it_edi_get_extra_info(self, company, document_type, body_tree, incoming=True):
        """ This function is meant to collect other information that has to be inserted on the invoice lines by submodules.
            :return extra_info, messages_to_log"""
        return {
            'simplified': self.env['account.move']._l10n_it_edi_is_simplified_document_type(document_type),
            'type_tax_use_domain': [('type_tax_use', '=', 'purchase' if incoming else 'sale')],
        }, []

    def _l10n_it_edi_import_invoice(self, invoice, data, is_new):
        """ Decodes a l10n_it_edi move into an Odoo move.

        :param data:   the dictionary with the content to be imported
                       keys: 'filename', 'content', 'xml_tree', 'type', 'sort_weight'
        :param is_new: whether the move is newly created or to be updated
        :returns:      the imported move
        """
        with self._get_edi_creation() as self:
            buyer_seller_info = self._l10n_it_buyer_seller_info()

            tree = data['xml_tree']
            company = self.company_id

            # There are 2 cases:
            # - cron:
            #     * Move direction (incoming / outgoing) flexible (no 'default_move_type')
            #     * I.e. used for import from tax agency
            # - "Upload" button (invoices / bills view)
            #     * Fixed move direction; the button sets the 'default_move_type'
            default_move_type = self.env.context.get('default_move_type')
            if default_move_type is None:
                incoming_possibilities = [True, False]
            elif default_move_type in invoice.get_purchase_types(include_receipts=True):
                incoming_possibilities = [True]
            elif default_move_type in invoice.get_sale_types(include_receipts=True):
                incoming_possibilities = [False]
            else:
                _logger.warning("Cannot handle default_move_type '%s'.", default_move_type)
                return

            for incoming in incoming_possibilities:
                company_role, partner_role = ('buyer', 'seller') if incoming else ('seller', 'buyer')
                company_info = buyer_seller_info[company_role]
                vat = get_text(tree, company_info['vat_xpath'])
                if vat and vat .casefold() in (company.vat or '').casefold():
                    break
                codice_fiscale = get_text(tree, company_info['codice_fiscale_xpath'])
                if codice_fiscale and codice_fiscale.casefold() in (company.l10n_it_codice_fiscale or '').casefold():
                    break
            else:
                invoice.message_post(body=_("Your company's VAT number and Fiscal Code haven't been found in the buyer and/or seller sections inside the document."))
                return

            # For unsupported document types, just assume in_invoice, and log that the type is unsupported
            document_type = get_text(tree, '//DatiGeneraliDocumento/TipoDocumento')
            move_type = self._l10n_it_edi_document_type_mapping().get(document_type, {}).get('import_type')
            if not move_type:
                move_type = "in_invoice"
                _logger.info('Document type not managed: %s. Invoice type is set by default.', document_type)
            if not incoming and move_type.startswith('in_'):
                move_type = 'out' + move_type[2:]

            self.move_type = move_type

            if self.name and self.name != '/':
                # the journal might've changed, so we need to recompute the name in case it was set (first entry in journal)
                self.name = False
                self._compute_name()

            # Collect extra info from the XML that may be used by submodules to further put information on the invoice lines
            extra_info, message_to_log = self._l10n_it_edi_get_extra_info(company, document_type, tree, incoming=incoming)

            # Partner
            partner_info = buyer_seller_info[partner_role]
            vat = get_text(tree, partner_info['vat_xpath'])
            codice_fiscale = get_text(tree, partner_info['codice_fiscale_xpath'])
            email = get_text(tree, '//DatiTrasmissione//Email') if partner_info['role'] == 'seller' else ''
            if partner := self._l10n_it_edi_search_partner(company, vat, codice_fiscale, email):
                self.partner_id = partner
            else:
                message = Markup("<br/>").join((
                    _("Partner not found, useful informations from XML file:"),
                    self._compose_info_message(tree, partner_info['section_xpath'])
                ))
                message_to_log.append(message)

            # Numbering attributed by the transmitter
            if progressive_id := get_text(tree, '//ProgressivoInvio'):
                self.payment_reference = progressive_id

            # Document Number
            if number := get_text(tree, './/DatiGeneraliDocumento//Numero'):
                self.ref = number

            # Currency
            if currency_str := get_text(tree, './/DatiGeneraliDocumento/Divisa'):
                currency = self.env.ref('base.%s' % currency_str.upper(), raise_if_not_found=False)
                if currency != self.env.company.currency_id and currency.active:
                    self.currency_id = currency

            # Date
            if document_date := get_date(tree, './/DatiGeneraliDocumento/Data'):
                self.invoice_date = document_date
            else:
                message_to_log.append(_("Document date invalid in XML file: %s", document_date))

            # Stamp Duty
            if stamp_duty := get_text(tree, './/DatiGeneraliDocumento/DatiBollo/ImportoBollo'):
                self.l10n_it_stamp_duty = float(stamp_duty)

            # Comment
            for narration in get_text(tree, './/DatiGeneraliDocumento//Causale', many=True):
                self.narration = '%s%s<br/>' % (self.narration or '', narration)

            # Informations relative to the purchase order, the contract, the agreement,
            # the reception phase or invoices previously transmitted
            # <2.1.2> - <2.1.6>
            for document_type in ['DatiOrdineAcquisto', 'DatiContratto', 'DatiConvenzione', 'DatiRicezione', 'DatiFattureCollegate']:
                for element in tree.xpath('.//DatiGenerali/' + document_type):
                    message = Markup("{} {}<br/>{}").format(document_type, _("from XML file:"), self._compose_info_message(element, '.'))
                    message_to_log.append(message)

            #  Dati DDT. <2.1.8>
            if elements := tree.xpath('.//DatiGenerali/DatiDDT'):
                message = Markup("<br/>").join((
                    _("Transport informations from XML file:"),
                    self._compose_info_message(tree, './/DatiGenerali/DatiDDT')
                ))
                message_to_log.append(message)

            # Due date. <2.4.2.5>
            if due_date := get_date(tree, './/DatiPagamento/DettaglioPagamento/DataScadenzaPagamento'):
                self.invoice_date_due = fields.Date.to_string(due_date)
            else:
                message_to_log.append(_("Payment due date invalid in XML file: %s", str(due_date)))

            # Information related to the purchase order <2.1.2>
            if (po_refs := get_text(tree, '//DatiGenerali/DatiOrdineAcquisto/IdDocumento', many=True)):
                self.invoice_origin = ", ".join(po_refs)

            # Total amount. <2.4.2.6>
            if amount_total := sum(float(x) for x in get_text(tree, './/ImportoPagamento', many=True) if x):
                message_to_log.append(_("Total amount from the XML File: %s", amount_total))

            # Bank account. <2.4.2.13>
            if self.move_type not in ('out_invoice', 'in_refund'):
                if acc_number := get_text(tree, './/DatiPagamento/DettaglioPagamento/IBAN'):
                    if self.partner_id and self.partner_id.commercial_partner_id:
                        bank = self.env['res.partner.bank'].search([
                            ('acc_number', '=', acc_number),
                            ('partner_id', '=', self.partner_id.commercial_partner_id.id),
                            ('company_id', 'in', [self.company_id.id, False])
                        ], order='company_id', limit=1)
                    else:
                        bank = self.env['res.partner.bank'].search([
                            ('acc_number', '=', acc_number),
                            ('company_id', 'in', [self.company_id.id, False])
                        ], order='company_id', limit=1)
                    if bank:
                        self.partner_bank_id = bank
                    else:
                        message = Markup("<br/>").join((
                            _("Bank account not found, useful informations from XML file:"),
                            self._compose_info_message(tree, [
                                './/DatiPagamento//Beneficiario',
                                './/DatiPagamento//IstitutoFinanziario',
                                './/DatiPagamento//IBAN',
                                './/DatiPagamento//ABI',
                                './/DatiPagamento//CAB',
                                './/DatiPagamento//BIC',
                                './/DatiPagamento//ModalitaPagamento'
                            ])
                        ))
                        message_to_log.append(message)
            elif elements := tree.xpath('.//DatiPagamento/DettaglioPagamento'):
                message = Markup("<br/>").join((
                    _("Bank account not found, useful informations from XML file:"),
                    self._compose_info_message(tree, './/DatiPagamento')
                ))
                message_to_log.append(message)

            # Invoice lines. <2.2.1>
            tag_name = './/DettaglioLinee' if not extra_info['simplified'] else './/DatiBeniServizi'
            for element in tree.xpath(tag_name):
                move_line = self.invoice_line_ids.create({
                    'move_id': self.id,
                    'tax_ids': [fields.Command.clear()]})
                if move_line:
                    message_to_log += self._l10n_it_edi_import_line(element, move_line, extra_info)

            # Global discount summarized in 1 amount
            if discount_elements := tree.xpath('.//DatiGeneraliDocumento/ScontoMaggiorazione'):
                taxable_amount = float(self.tax_totals['amount_untaxed'])
                discounted_amount = taxable_amount
                for discount_element in discount_elements:
                    discount_sign = 1
                    if (discount_type := discount_element.xpath('.//Tipo')) and discount_type[0].text == 'MG':
                        discount_sign = -1
                    if discount_amount := get_text(discount_element, './/Importo'):
                        discounted_amount -= discount_sign * float(discount_amount)
                        continue
                    if discount_percentage := get_text(discount_element, './/Percentuale'):
                        discounted_amount *= 1 - discount_sign * float(discount_percentage) / 100

                general_discount = discounted_amount - taxable_amount
                sequence = len(elements) + 1

                self.invoice_line_ids = [Command.create({
                    'sequence': sequence,
                    'name': 'SCONTO' if general_discount < 0 else 'MAGGIORAZIONE',
                    'price_unit': general_discount,
                })]

            for element in tree.xpath('.//Allegati'):
                attachment_64 = self.env['ir.attachment'].create({
                    'name': get_text(element, './/NomeAttachment'),
                    'datas': str.encode(get_text(element, './/Attachment')),
                    'type': 'binary',
                    'res_model': 'account.move',
                    'res_id': self.id,
                })

                # no_new_invoice to prevent from looping on the.message_post that would create a new invoice without it
                self.with_context(no_new_invoice=True).sudo().message_post(
                    body=(_("Attachment from XML")),
                    attachment_ids=[attachment_64.id],
                )

            for message in message_to_log:
                self.sudo().message_post(body=message)
            return self

    @api.model
    def _is_prediction_enabled(self):
        return self.env['ir.module.module'].search([('name', '=', 'account_accountant'), ('state', '=', 'installed')])

    def _l10n_it_edi_import_line(self, element, move_line, extra_info=None):
        extra_info = extra_info or {}
        company = move_line.company_id
        partner = move_line.partner_id
        message_to_log = []
        predict_enabled = self._is_prediction_enabled()

        # Sequence.
        line_elements = element.xpath('.//NumeroLinea')
        if line_elements:
            move_line.sequence = int(line_elements[0].text)

        # Name.
        move_line.name = " ".join(get_text(element, './/Descrizione').split())

        # Product.
        if elements_code := element.xpath('.//CodiceArticolo'):
            for element_code in elements_code:
                type_code = element_code.xpath('.//CodiceTipo')[0]
                code = element_code.xpath('.//CodiceValore')[0]
                product = self.env['product.product'].search([('barcode', '=', code.text)])
                if (product and type_code.text == 'EAN'):
                    move_line.product_id = product
                    break
                if partner:
                    product_supplier = self.env['product.supplierinfo'].search([('partner_id', '=', partner.id), ('product_code', '=', code.text)], limit=2)
                    if product_supplier and len(product_supplier) == 1 and product_supplier.product_id:
                        move_line.product_id = product_supplier.product_id
                        break
            if not move_line.product_id:
                for element_code in elements_code:
                    code = element_code.xpath('.//CodiceValore')[0]
                    product = self.env['product.product'].search([('default_code', '=', code.text)], limit=2)
                    if product and len(product) == 1:
                        move_line.product_id = product
                        break

        # If no product is found, try to find a product that may be fitting
        if predict_enabled and not move_line.product_id:
            fitting_product = move_line._predict_product()
            if fitting_product:
                name = move_line.name
                move_line.product_id = fitting_product
                move_line.name = name

        if predict_enabled:
            # Fitting account for the line
            fitting_account = move_line._predict_account()
            if fitting_account:
                move_line.account_id = fitting_account

        # Quantity.
        move_line.quantity = float(get_text(element, './/Quantita') or '1')

        # Taxes
        percentage = None
        if not extra_info['simplified']:
            percentage = get_float(element, './/AliquotaIVA')
            if price_unit := get_float(element, './/PrezzoUnitario'):
                move_line.price_unit = price_unit
        elif amount := get_float(element, './/Importo'):
            percentage = get_float(element, './/Aliquota')
            if not percentage and (tax_amount := get_float(element, './/Imposta')):
                percentage = round(tax_amount / (amount - tax_amount) * 100)
            move_line.price_unit = amount / (1 + percentage / 100)

        move_line.tax_ids = []
        if percentage is not None:
            l10n_it_exempt_reason = get_text(element, './/Natura').upper() or False
            extra_domain = extra_info.get('type_tax_use_domain', [('type_tax_use', '=', 'purchase')])
            if tax := self._l10n_it_edi_search_tax_for_import(company, percentage, extra_domain, l10n_it_exempt_reason=l10n_it_exempt_reason):
                move_line.tax_ids += tax
            else:
                message = Markup("<br/>").join((
                    _("Tax not found for line with description '%s'", move_line.name),
                    self._compose_info_message(element, '.')
                ))
                message_to_log.append(message)

        # If no taxes were found, try to find taxes that may be fitting
        if predict_enabled and not move_line.tax_ids:
            fitting_taxes = move_line._predict_taxes()
            if fitting_taxes:
                move_line.tax_ids = [Command.set(fitting_taxes)]

        # Discounts
        if elements := element.xpath('.//ScontoMaggiorazione'):
            # Special case of only 1 percentage discount
            if len(elements) == 1:
                element = elements[0]
                if discount_percentage := get_float(element, './/Percentuale'):
                    discount_type = get_text(element, './/Tipo')
                    discount_sign = -1 if discount_type == 'MG' else 1
                    move_line.discount = discount_sign * discount_percentage
            # Discounts in cascade summarized in 1 percentage
            else:
                total = get_float(element, './/PrezzoTotale')
                discount = 100 - (100 * total) / (move_line.quantity * move_line.price_unit)
                move_line.discount = discount

        return message_to_log

    def _l10n_it_edi_format_errors(self, header, errors):
        return Markup('{}<ul class="mb-0">{}</ul>').format(
            nl2br_enclose(header, 'span') if header else '',
            Markup().join(nl2br_enclose(' '.join(error.split()), 'li') for error in errors)
        )

    def _compose_info_message(self, tree, tags):
        result = ""
        for tag in tags if isinstance(tags, list) else [tags]:
            for el in tree.xpath(tag):
                result += self._l10n_it_edi_format_errors("", [f'{subel.tag}: {subel.text}' for subel in el.iter()])
        return result

    # -------------------------------------------------------------------------
    # EDI: Export
    # -------------------------------------------------------------------------

    def _l10n_it_edi_export_data_check(self):
        """ This function checks the Settings, Company, Partners, Moves involved in the
            sending activity and returns an errors dictionary ready for the
            actionable_errors widget to display. """

        companies = self.mapped("company_id")
        companies_partners = companies.mapped("partner_id")
        moves_full = self.filtered(lambda m: not m._l10n_it_edi_is_simplified())
        moves_simplified = self.filtered(lambda m: m._l10n_it_edi_is_simplified())

        full = moves_full.mapped("commercial_partner_id").filtered(lambda p: p not in companies_partners)
        simplified = moves_simplified.mapped("commercial_partner_id").filtered(lambda p: p not in companies_partners | full)
        representatives = companies.mapped("l10n_it_tax_representative_partner_id").filtered(lambda p: p not in companies_partners | simplified | full)

        return {
            **companies._l10n_it_edi_export_check(),
            **full._l10n_it_edi_export_check(['partner_address_missing']),
            **simplified._l10n_it_edi_export_check(['partner_country_missing']),
            **(simplified | full)._l10n_it_edi_export_check(['partner_vat_codice_fiscale_missing']),
            **representatives._l10n_it_edi_export_check(['partner_vat_missing']),
            **self._l10n_it_edi_base_export_check(),
            **self._l10n_it_edi_export_taxes_check(),
        }

    def _l10n_it_edi_base_export_check(self):
        def build_error(message, records):
            return {
                'message': message,
                **({
                    'action_text': _("View invoice(s)"),
                    'action': records._get_records_action(name=_("Invoice(s) to check")),
                } if len(self) > 1 else {})
            }

        errors = {}
        if moves := self.filtered(lambda move: move.l10n_it_edi_is_self_invoice and move._l10n_it_edi_services_or_goods() == 'both'):
            errors['move_reverse_charge_with_mixed_services_and_goods'] = build_error(
                message=_("Cannot apply Reverse Charge to bills which contains both services and goods."),
                records=moves)
        if pa_moves := self.filtered(lambda move: move.commercial_partner_id._l10n_it_edi_is_public_administration()):
            if moves := pa_moves.filtered(lambda move: not move.l10n_it_origin_document_type):
                message = _("Partner(s) belongs to the Public Administration, please fill out Origin Document Type field in the Electronic Invoicing tab.")
                errors['move_missing_origin_document'] = build_error(message=message, records=moves)
            if moves := pa_moves.filtered(lambda move: move.l10n_it_origin_document_date and move.l10n_it_origin_document_date > fields.Date.today()):
                message = _("The Origin Document Date cannot be in the future.")
                errors['move_future_origin_document_date'] = build_error(message=message, records=moves)
        if pa_moves := self.filtered(lambda move: len(move.commercial_partner_id.l10n_it_pa_index or '') == 7):
            if moves := pa_moves.filtered(lambda move: not move.l10n_it_origin_document_type and (move.l10n_it_cig or move.l10n_it_cup)):
                message = _("CIG/CUP fields of partner(s) are present, please fill out Origin Document Type field in the Electronic Invoicing tab.")
                errors['move_missing_origin_document_field'] = build_error(message=message, records=moves)
        return errors

    def _l10n_it_edi_export_taxes_check(self):
        if move_lines := self.mapped("invoice_line_ids").filtered(lambda line:
            line.display_type == 'product'
            and len(line.tax_ids.flatten_taxes_hierarchy()._l10n_it_filter_kind('vat')) != 1
        ):
            return {
                'move_only_one_vat_tax_per_line': {
                    'message': _("Invoices must have exactly one VAT tax set per line."),
                    **({
                        'action_text': _("View invoice(s)"),
                        'action': move_lines.mapped("move_id")._get_records_action(name=_("Check taxes on invoice lines")),
                    } if len(self) > 1 else {})
                }}
        return {}

    def _l10n_it_edi_get_formatters(self):
        def format_alphanumeric(text, maxlen=None):
            if not text:
                return False
            text = text.encode('latin-1', 'replace').decode('latin-1')
            if maxlen and maxlen > 0:
                text = text[:maxlen]
            elif maxlen and maxlen < 0:
                text = text[maxlen:]
            return text

        def format_date(dt):
            # Format the date in the italian standard.
            dt = dt or datetime.now()
            return dt.strftime('%Y-%m-%d')

        def format_monetary(number, currency):
            # Format the monetary values to avoid trailing decimals (e.g. 90.85000000000001).
            return float_repr(number, min(2, currency.decimal_places))

        def format_numbers(number):
            #format number to str with between 2 and 8 decimals (event if it's .00)
            number_splited = str(number).split('.')
            if len(number_splited) == 1:
                return "%.02f" % number

            cents = number_splited[1]
            if len(cents) > 8:
                return "%.08f" % number
            return float_repr(number, max(2, len(cents)))

        def format_numbers_two(number):
            #format number to str with 2 (event if it's .00)
            return "%.02f" % number

        def format_phone(number):
            if not number:
                return False
            number = number.replace(' ', '').replace('/', '').replace('.', '')
            if len(number) > 4 and len(number) < 13:
                return format_alphanumeric(number)
            return False

        def format_address(street, street2, maxlen=60):
            street, street2 = street or '', street2 or ''
            if street and len(street) >= maxlen:
                street2 = ''
            sep = ' ' if street and street2 else ''
            return format_alphanumeric(f"{street}{sep}{street2}", maxlen)

        return {
            'format_date': format_date,
            'format_monetary': format_monetary,
            'format_numbers': format_numbers,
            'format_numbers_two': format_numbers_two,
            'format_phone': format_phone,
            'format_alphanumeric': format_alphanumeric,
            'format_address': format_address,
        }

    def _l10n_it_edi_render_xml(self, pdf_values=None):
        ''' Create the xml file content.
            :return:    The XML content as bytestring.
        '''
        qweb_template_name = (
            'l10n_it_edi.account_invoice_it_FatturaPA_export' if not self._l10n_it_edi_is_simplified()
            else 'l10n_it_edi.account_invoice_it_simplified_FatturaPA_export')
        xml_content = self.env['ir.qweb']._render(qweb_template_name, {
            **self._l10n_it_edi_get_values(pdf_values),
            **self._l10n_it_edi_get_formatters()})
        xml_node = cleanup_xml_node(xml_content, remove_blank_nodes=False)
        return etree.tostring(xml_node, xml_declaration=True, encoding='UTF-8')

    def _l10n_it_edi_get_attachment_values(self, pdf_values=None):
        self.ensure_one()
        return {
            'name': self._l10n_it_edi_generate_filename(),
            'type': 'binary',
            'mimetype': 'application/xml',
            'description': _('IT EDI e-move: %s', self.move_type),
            'company_id': self.company_id.id,
            'res_id': self.id,
            'res_model': self._name,
            'res_field': 'l10n_it_edi_attachment_file',
            'raw': self._l10n_it_edi_render_xml(pdf_values=pdf_values),
        }

    def _l10n_it_edi_generate_filename(self):
        '''Returns a name conform to the Fattura pa Specifications:
           See ES documentation 2.2
        '''
        a = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
        # Each company should have its own filename sequence. If it does not exist, create it
        n = self.env['ir.sequence'].with_company(self.company_id).next_by_code('l10n_it_edi.fattura_filename')
        if not n:
            # The offset is used to avoid conflicts with existing filenames
            offset = 62 ** 4
            sequence = self.env['ir.sequence'].sudo().create({
                'name': 'FatturaPA Filename Sequence',
                'code': 'l10n_it_edi.fattura_filename',
                'company_id': self.company_id.id,
                'number_next': offset,
            })
            n = sequence._next()
        # The n is returned as a string, but we require an int
        n = int(''.join(filter(lambda c: c.isdecimal(), n)))

        progressive_number = ""
        while n:
            (n, m) = divmod(n, len(a))
            progressive_number = a[m] + progressive_number

        return '%(country_code)s%(codice)s_%(progressive_number)s.xml' % {
            'country_code': self.company_id.country_id.code,
            'codice': self.company_id.partner_id._l10n_it_edi_normalized_codice_fiscale(),
            'progressive_number': progressive_number.zfill(5),
        }

    def _l10n_it_edi_send(self, attachments_vals):
        self.env['res.company']._with_locked_records(self)
        files_to_upload = []
        filename_move = {}

        # Setup moves for sending
        for move in self:
            attachment_vals = attachments_vals[move]
            filename = attachment_vals['name']
            content = b64encode(attachment_vals['raw']).decode()
            move.l10n_it_edi_header = False
            if move.commercial_partner_id._l10n_it_edi_is_public_administration():
                move.l10n_it_edi_state = 'requires_user_signature'
                move.l10n_it_edi_transaction = False
                move.sudo().message_post(body=nl2br(escape(_(
                    "Sending invoices to Public Administration partners is not supported.\n"
                    "The IT EDI XML file is generated, please sign the document and upload it "
                    "through the 'Fatture e Corrispettivi' portal of the Tax Agency."
                ))))
            else:
                move.l10n_it_edi_state = 'being_sent'
                files_to_upload.append({'filename': filename, 'xml': content})
                filename_move[filename] = move

        # Upload files
        try:
            results = self._l10n_it_edi_upload(files_to_upload)
        except AccountEdiProxyError as e:
            messages_to_log = []
            for filename in filename_move:
                unsent_move = filename_move[filename]
                unsent_move.l10n_it_edi_state = False
                text_message = _("Error uploading the e-invoice file %s.\n%s", filename, e.message)
                html_message = nl2br(escape(text_message))
                unsent_move.l10n_it_edi_header = text_message
                unsent_move.sudo().message_post(body=html_message)
                messages_to_log.append(text_message)
            raise UserError("\n".join(messages_to_log)) from e

        # Handle results
        for filename, vals in results.items():
            sent_move = filename_move[filename]
            if 'error' in vals:
                sent_move.l10n_it_edi_state = False
                sent_move.l10n_it_edi_transaction = False
                message = nl2br(escape(_("Error uploading the e-invoice file %s.\n%s", filename, vals['error'])))
            else:
                is_demo = vals['id_transaction'] == 'demo'
                sent_move.l10n_it_edi_state = 'processing'
                sent_move.l10n_it_edi_transaction = vals['id_transaction']
                message = (
                    _("We are simulating the sending of the e-invoice file %s, as we are in demo mode.", filename)
                    if is_demo else _("The e-invoice file %s was sent to the SdI for processing.", filename))
            sent_move.l10n_it_edi_header = message
            sent_move.sudo().message_post(body=message)

    def _l10n_it_edi_upload(self, files):
        '''Upload files to the SdI.

        :param files:    A list of dictionary {filename, base64_xml}.
        :returns:        A dictionary.
        * message:       Message from fatturapa.
        * transactionId: The fatturapa ID of this request.
        * error:         An eventual error.
        '''
        if not files:
            return {}
        proxy_user = self.company_id.l10n_it_edi_proxy_user_id
        proxy_user.ensure_one()
        if proxy_user.edi_mode == 'demo':
            return {file_data['filename']: {'id_transaction': 'demo'} for file_data in files}

        ERRORS = {'EI01': _('Attached file is empty'),
                  'EI02': _('Service momentarily unavailable'),
                  'EI03': _('Unauthorized user')}

        server_url = proxy_user._get_server_url()
        results = proxy_user._make_request(
            f'{server_url}/api/l10n_it_edi/1/out/SdiRiceviFile',
            params={'files': files})

        for filename, vals in results.items():
            if 'error' in vals:
                results[filename]['error'] = ERRORS.get(vals.get('error'), _("Unknown error"))

        return results

    # -------------------------------------------------------------------------
    # EDI: Update notifications
    # -------------------------------------------------------------------------

    def _l10n_it_edi_update_send_state(self):
        ''' Check if the current invoices have been processed by the SdI. '''
        proxy_user = self.company_id.l10n_it_edi_proxy_user_id
        if proxy_user.edi_mode == 'demo':
            for move in self:
                filename = move.l10n_it_edi_attachment_id and move.l10n_it_edi_attachment_id.name or '???'
                self._l10n_it_edi_write_send_state(
                    transformed_notification={
                        'l10n_it_edi_state': 'forwarded',
                        'l10n_it_edi_transaction': f'demo_{uuid.uuid4()}',
                        'send_ack_to_edi_proxy': False,
                        'date': fields.Date.today(),
                        'filename': filename},
                    message=_("The e-invoice file %s has been sent in Demo EDI mode.", filename))
            return

        server_url = proxy_user._get_server_url()
        try:
            notifications = proxy_user._make_request(
                f'{server_url}/api/l10n_it_edi/1/in/TrasmissioneFatture',
                params={'ids_transaction': self.mapped("l10n_it_edi_transaction")})
        except AccountEdiProxyError as pe:
            raise UserError(_("An error occurred while downloading updates from the Proxy Server: (%s) %s", pe.code, pe.message)) from pe

        for _id_transaction, notification in notifications.items():
            encrypted_update_content = notification.get('file')
            encryption_key = notification.get('key')
            if (encrypted_update_content and encryption_key):
                notification['xml_content'] = proxy_user._decrypt_data(encrypted_update_content, encryption_key)

        acks = {'transaction_ids': [], 'states': []}
        for move in self:
            notification = notifications[move.l10n_it_edi_transaction]
            parsed_notification = move._l10n_it_edi_parse_notification(notification)
            transformed_notification = move._l10n_it_edi_transform_notification(parsed_notification)
            message = move._l10n_it_edi_get_message(transformed_notification)
            move._l10n_it_edi_write_send_state(transformed_notification, message)
            if (
                transformed_notification.get('send_ack_to_edi_proxy')
                and (id_transaction_to_ack := transformed_notification.get('l10n_it_edi_transaction'))
                and (ack_state := transformed_notification.get('l10n_it_edi_state'))
            ):
                acks['transaction_ids'].append(id_transaction_to_ack)
                acks['states'].append(ack_state)

        if acks:
            transaction_ids = acks['transaction_ids']
            states = acks['states']
            try:
                proxy_user._make_request(
                    f'{server_url}/api/l10n_it_edi/1/ack',
                    params={'transaction_ids': transaction_ids, 'states': states})
            except AccountEdiProxyError as pe:
                raise UserError(_("An error occurred while downloading updates from the Proxy Server: (%s) %s", pe.code, pe.message)) from pe

    def _l10n_it_edi_parse_notification(self, notification):
        sdi_state = notification.get('state', '')
        if not (xml_content := notification.get('xml_content')):
            return {'sdi_state': sdi_state}

        decrypted_update_content = etree.fromstring(xml_content)
        outcome = get_text(decrypted_update_content, './/Esito')
        date_arrival = get_datetime(decrypted_update_content, './/DataOraRicezione') or fields.Date.today()
        errors = [(
            get_text(error_element, '//Codice'),
            get_text(error_element, '//Descrizione'),
        ) for error_element in decrypted_update_content.xpath('//Errore')]
        filename = get_text(decrypted_update_content, './/NomeFile')

        return {
            'sdi_state': sdi_state,
            'errors': errors,
            'outcome': outcome,
            'date': date_arrival,
            'filename': filename,
        }

    def _l10n_it_edi_transform_notification(self, parsed_notification):
        """ Reads the notification XML coming from the EDI Proxy Server
            Recovers information about the new state.
            Computes whether the EDI Proxy Server is to be acked,
            and whether the id_transaction has to be reset.
        """
        self.ensure_one()
        state_map = {
            'not_found': False,
            'awaiting_outcome': 'processing',
            'notificaScarto': 'rejected',
            'ricevutaConsegna': 'forwarded',
            'forward_attempt': 'forward_attempt',
            'notificaMancataConsegna': 'forward_failed',
            ('notificaEsito', 'EC01'): 'accepted_by_pa_partner',
            ('notificaEsito', 'EC02'): 'rejected_by_pa_partner',
            'notificaDecorrenzaTermini': 'accepted_by_pa_partner_after_expiry',
        }
        sdi_state = parsed_notification['sdi_state']
        filename = parsed_notification.get('filename')
        errors = parsed_notification.get('errors', [])
        date = parsed_notification.get('date', fields.Date.today())
        if not filename and self.l10n_it_edi_attachment_id:
            filename = self.l10n_it_edi_attachment_id.name
        outcome = parsed_notification.get('outcome', False)
        if not outcome:
            new_state = state_map.get(sdi_state, False)
        else:
            new_state = state_map.get((sdi_state, outcome), False)

        parsed_notification.update({
            'l10n_it_edi_state': new_state,
            'l10n_it_edi_transaction': False if new_state in (False, 'rejected') else self.l10n_it_edi_transaction,
            'send_ack_to_edi_proxy': bool(new_state),
            'date': date,
            'errors': errors,
            'filename': filename,
        })
        return parsed_notification

    def _l10n_it_edi_write_send_state(self, transformed_notification, message):
        """ Update the record with the data coming from the IAP server.
            Eventually post the message.
            Commit the transaction.
        """
        self.ensure_one()
        old_state = self.l10n_it_edi_state
        new_state = transformed_notification['l10n_it_edi_state']
        self.write({
            'l10n_it_edi_state': new_state,
            'l10n_it_edi_transaction': transformed_notification['l10n_it_edi_transaction'],
            'l10n_it_edi_header': message or False,
        })

        if message and old_state != new_state:
            self.with_context(no_new_invoice=True).sudo().message_post(body=message)

        if new_state == 'rejected':
            self.l10n_it_edi_attachment_file = False

        self.env.cr.commit()

    def _l10n_it_edi_get_message(self, transformed_notification):
        """ The status change will be notified in the chatter of the move.
            Compute the message from the notification information coming from the EDI Proxy Server
        """
        self.ensure_one()
        partner = self.commercial_partner_id
        partner_name = partner.display_name
        filename = transformed_notification['filename']
        new_state = transformed_notification['l10n_it_edi_state']
        if new_state == 'rejected':
            DUPLICATE_MOVE = '00404'
            DUPLICATE_FILENAME = '00002'
            error_descriptions = []
            for error_code, error_description in transformed_notification['errors']:
                error_description_copy = error_description
                if error_code == DUPLICATE_MOVE:
                    error_description_copy = _(
                        "The e-invoice file %s is duplicated.\n"
                        "Original message from the SdI: %s",
                        filename, error_description_copy)
                elif error_code == DUPLICATE_FILENAME:
                    error_description_copy = _(
                        "The e-invoice filename %s is duplicated. Please check the FatturaPA Filename sequence.\n"
                        "Original message from the SdI: %s",
                        filename, error_description_copy)
                error_descriptions.append(error_description_copy)

            return self._l10n_it_edi_format_errors(_('The e-invoice has been refused by the SdI.'), error_descriptions)

        elif partner._l10n_it_edi_is_public_administration():
            pa_specific_map = {
                'forwarded': nl2br(escape(_(
                    "The e-invoice file %s was succesfully sent to the SdI.\n"
                    "%s has 15 days to accept or reject it.",
                    filename, partner_name))),
                'forward_attempt': nl2br(escape(_(
                    "The e-invoice file %s can't be forward to %s (Public Administration) by the SdI at the moment.\n"
                    "It will try again for 10 days, after which it will be considered accepted, but "
                    "you will still have to send it by post or e-mail.",
                    filename, partner_name))),
                'accepted_by_pa_partner_after_expiry': nl2br(escape(_(
                    "The e-invoice file %s is succesfully sent to the SdI. The invoice is now considered fiscally relevant.\n"
                    "The %s (Public Administration) had 15 days to either accept or refused this document,"
                    "but since they did not reply, it's now considered accepted.",
                    filename, partner_name))),
                'rejected_by_pa_partner': nl2br(escape(_(
                    "The e-invoice file %s has been refused by %s (Public Administration).\n"
                    "You have 5 days from now to issue a full refund for this invoice, "
                    "then contact the PA partner to create a new one according to their "
                    "requests and submit it.",
                    filename, partner_name))),
                'accepted_by_pa_partner': _(
                    "The e-invoice file %s has been accepted by %s (Public Administration), a payment will be issued soon",
                    filename, partner_name),
            }
            if pa_specific_message := pa_specific_map.get(new_state):
                return pa_specific_message

        new_state_messages_map = {
            False: _(
                "The e-invoice file %s has not been found on the EDI Proxy server.", filename),
            'processing': nl2br(escape(_(
                "The e-invoice file %s was sent to the SdI for validation.\n"
                "It is not yet considered accepted, please wait further notifications.",
                filename))),
            'forwarded': _(
                "The e-invoice file %s was accepted and succesfully forwarded it to %s by the SdI.",
                filename, partner_name),
            'forward_attempt': nl2br(escape(_(
                "The e-invoice file %s has been accepted by the SdI.\n"
                "The SdI is trying to forward it to %s.\n"
                "It will try for up to 2 days, after which you'll eventually "
                "need to send it the invoice to the partner by post or e-mail.",
                filename, partner_name))),
            'forward_failed': nl2br(escape(_(
                "The e-invoice file %s couldn't be forwarded to %s.\n"
                "Please remember to send it via post or e-mail.",
                filename, partner_name)))
        }
        return new_state_messages_map.get(new_state)

```

## File: models\ddt.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api

class L10nItDdt(models.Model):
    _name = 'l10n_it.ddt'
    _description = 'Transport Document'

    invoice_id = fields.One2many('account.move', 'l10n_it_ddt_id', string='Invoice Reference')
    name = fields.Char(string="Numero DDT", size=20, help="Transport document number", required=True)
    date = fields.Date(string="Data DDT", help="Transport document date", required=True)

    @api.depends('date')
    def _compute_display_name(self):
        for ddt in self:
            ddt.display_name = f"{ddt.name} ({ddt.date})"

```

## File: models\ir_attachment.py

```python
# -*- coding: utf-8 -*-

from odoo import api, models
from odoo.addons.l10n_it_edi.tools.remove_signature import remove_signature

from lxml import etree
import logging
import re

_logger = logging.getLogger(__name__)

FATTURAPA_FILENAME_RE = "[A-Z]{2}[A-Za-z0-9]{2,28}_[A-Za-z0-9]{0,5}.((?i:xml.p7m|xml))"


class IrAttachment(models.Model):
    _inherit = 'ir.attachment'

    def _decode_edi_l10n_it_edi(self, name, content):
        """ Decodes a  into a list of one dictionary representing an attachment.
            :returns:           A list with a dictionary.
        """
        def parse_xml(parser, name, content):
            try:
                return etree.fromstring(content, parser)
            except (etree.ParseError, ValueError) as e:
                _logger.info("XML parsing of %s failed: %s", name, e)

        parser = etree.XMLParser(recover=True, resolve_entities=False)
        if (xml_tree := parse_xml(parser, name, content)) is None:
            # The file may have a Cades signature, trying to remove it
            if (xml_tree := parse_xml(parser, name, remove_signature(content))) is None:
                _logger.info("Italian EDI invoice file %s cannot be decoded.", name)
                return []

        return [{
            'filename': name,
            'content': content,
            'attachment': self,
            'xml_tree': xml_move_tree,
            'type': 'l10n_it_edi',
            'sort_weight': 11,
        } for xml_move_tree in xml_tree.xpath('//FatturaElettronicaBody')]

    def _is_l10n_it_edi_import_file(self):
        is_xml = (
            self.name.endswith('.xml')
            or self.mimetype.endswith('/xml')
            or 'text/plain' in self.mimetype
            and self.raw
            and self.raw.startswith(b'<?xml'))
        is_p7m = self.mimetype == 'application/pkcs7-mime'
        return (is_xml or is_p7m) and re.search(FATTURAPA_FILENAME_RE, self.name)

    @api.model
    def _get_edi_supported_formats(self):
        """ XML files could be l10n_it_edi related or not, so check it
            before demanding the decoding to the the standard XML methods.
        """
        # EXTENDS 'account'
        return [{
            'format': 'l10n_it_edi',
            'check': lambda a: a._is_l10n_it_edi_import_file(),
            'decoder': self._decode_edi_l10n_it_edi,
        }] + super()._get_edi_supported_formats()

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError

TAX_SYSTEM = [
    ("RF01", "[RF01] Ordinario"),
    ("RF02", "[RF02] Contribuenti minimi (art.1, c.96-117, L. 244/07)"),
    ("RF04", "[RF04] Agricoltura e attività connesse e pesca (artt.34 e 34-bis, DPR 633/72)"),
    ("RF05", "[RF05] Vendita sali e tabacchi (art.74, c.1, DPR. 633/72)"),
    ("RF06", "[RF06] Commercio fiammiferi (art.74, c.1, DPR  633/72)"),
    ("RF07", "[RF07] Editoria (art.74, c.1, DPR  633/72)"),
    ("RF08", "[RF08] Gestione servizi telefonia pubblica (art.74, c.1, DPR 633/72)"),
    ("RF09", "[RF09] Rivendita documenti di trasporto pubblico e di sosta (art.74, c.1, DPR  633/72)"),
    ("RF10", "[RF10] Intrattenimenti, giochi e altre attività di cui alla tariffa allegata al DPR 640/72 (art.74, c.6, DPR 633/72)"),
    ("RF11", "[RF11] Agenzie viaggi e turismo (art.74-ter, DPR 633/72)"),
    ("RF12", "[RF12] Agriturismo (art.5, c.2, L. 413/91)"),
    ("RF13", "[RF13] Vendite a domicilio (art.25-bis, c.6, DPR  600/73)"),
    ("RF14", "[RF14] Rivendita beni usati, oggetti d’arte, d’antiquariato o da collezione (art.36, DL 41/95)"),
    ("RF15", "[RF15] Agenzie di vendite all’asta di oggetti d’arte, antiquariato o da collezione (art.40-bis, DL 41/95)"),
    ("RF16", "[RF16] IVA per cassa P.A. (art.6, c.5, DPR 633/72)"),
    ("RF17", "[RF17] IVA per cassa (art. 32-bis, DL 83/2012)"),
    ("RF18", "[RF18] Altro"),
    ("RF19", "[RF19] Regime forfettario (art.1, c.54-89, L. 190/2014)"),
]

class ResCompany(models.Model):
    _name = 'res.company'
    _inherit = 'res.company'

    l10n_it_codice_fiscale = fields.Char(string="Codice Fiscale", size=16, related='partner_id.l10n_it_codice_fiscale',
        store=True, readonly=False, help="Fiscal code of your company")
    l10n_it_tax_system = fields.Selection(selection=TAX_SYSTEM, string="Tax System",
        help="Please select the Tax system to which you are subjected.")
    l10n_it_edi_proxy_user_id = fields.Many2one(
        comodel_name="account_edi_proxy_client.user",
        compute="_compute_l10n_it_edi_proxy_user_id",
    )

    # Economic and Administrative Index
    l10n_it_has_eco_index = fields.Boolean(
        help="The seller/provider is a company listed on the register of companies and as\
        such must also indicate the registration data on all documents (art. 2250, Italian\
        Civil Code)")
    l10n_it_eco_index_office = fields.Many2one('res.country.state', domain="[('country_id','=','IT')]",
        string="Province of the register-of-companies office")
    l10n_it_eco_index_number = fields.Char(string="Number in register of companies", size=20,
        help="This field must contain the number under which the\
        seller/provider is listed on the register of companies.")
    l10n_it_eco_index_share_capital = fields.Float(string="Share capital actually paid up",
        help="Mandatory if the seller/provider is a company with share\
        capital (SpA, SApA, Srl), this field must contain the amount\
        of share capital actually paid up as resulting from the last\
        financial statement")
    l10n_it_eco_index_sole_shareholder = fields.Selection(
        [
            ("NO", "Not a limited liability company"),
            ("SU", "Socio unico"),
            ("SM", "Più soci")],
        string="Shareholder")
    l10n_it_eco_index_liquidation_state = fields.Selection(
        [
            ("LS", "The company is in a state of liquidation"),
            ("LN", "The company is not in a state of liquidation")],
        string="Liquidation state")


    # Tax representative
    l10n_it_has_tax_representative = fields.Boolean(
        help="The seller/provider is a non-resident subject which\
        carries out transactions in Italy with relevance for VAT\
        purposes and which takes avail of a tax representative in\
        Italy")
    l10n_it_tax_representative_partner_id = fields.Many2one('res.partner', string='Tax representative partner')

    @api.constrains('l10n_it_has_eco_index',
                    'l10n_it_eco_index_office',
                    'l10n_it_eco_index_number',
                    'l10n_it_eco_index_liquidation_state')
    def _check_eco_admin_index(self):
        for record in self:
            if (record.l10n_it_has_eco_index
                and (not record.l10n_it_eco_index_office
                     or not record.l10n_it_eco_index_number
                     or not record.l10n_it_eco_index_liquidation_state)):
                raise ValidationError(_("All fields about the Economic and Administrative Index must be completed."))

    @api.constrains('l10n_it_has_eco_index',
                    'l10n_it_eco_index_share_capital',
                    'l10n_it_eco_index_sole_shareholder')
    def _check_eco_incorporated(self):
        """ If the business is incorporated, both these fields must be present.
            We don't know whether the business is incorporated, but in any case the fields
            must be both present or not present. """
        for record in self:
            if (record.l10n_it_has_eco_index
                and bool(record.l10n_it_eco_index_share_capital) ^ bool(record.l10n_it_eco_index_sole_shareholder)):
                raise ValidationError(_("If one of Share Capital or Sole Shareholder is present, "
                                        "then they must be both filled out."))

    @api.constrains('l10n_it_has_tax_representative',
                    'l10n_it_tax_representative_partner_id')
    def _check_tax_representative(self):
        for record in self:
            if not record.l10n_it_has_tax_representative:
                continue
            if not record.l10n_it_tax_representative_partner_id:
                raise ValidationError(_("You must select a tax representative."))
            if not record.l10n_it_tax_representative_partner_id.vat:
                raise ValidationError(_("Your tax representative partner must have a tax number."))
            if not record.l10n_it_tax_representative_partner_id.country_id:
                raise ValidationError(_("Your tax representative partner must have a country."))

    @api.depends("account_edi_proxy_client_ids")
    def _compute_l10n_it_edi_proxy_user_id(self):
        for company in self:
            company.l10n_it_edi_proxy_user_id = company.account_edi_proxy_client_ids.filtered(lambda x: x.proxy_type == 'l10n_it_edi')

    def _l10n_it_edi_export_check(self):
        checks = {
            'company_vat_codice_fiscale_missing': {
                'fields': [('vat', 'l10n_it_codice_fiscale')],
                'message': _("Company/ies should have a VAT number or Codice Fiscale."),
            },
            'company_address_missing': {
                'fields': [('street', 'street2'), ('zip',), ('city',), ('country_id',)],
                'message': _("Company/ies should have a complete address, verify their Street, City, Zipcode and Country."),
            },
            'company_l10n_it_tax_system_missing': {
                'fields': [('l10n_it_tax_system',)],
                'message': _("Company/ies should have a Tax System"),
            },
        }
        errors = {}
        for key, check in checks.items():
            for fields_tuple in check.pop('fields'):
                if invalid_records := self.filtered(lambda record: not any(record[field] for field in fields_tuple)):
                    errors[key] = {
                        'message': check['message'],
                        'action_text': _("View Company/ies"),
                        'action': invalid_records._get_records_action(name=_("Check Company Data")),
                    }
        if self.filtered(lambda x: not x.l10n_it_edi_proxy_user_id):
            new_context = {
                **self.env.context,
                'module': 'account',
                'default_search_setting': _("Italian Electronic Invoicing"),
                'bin_size': False,
            }
            errors['settings_l10n_it_edi_proxy_user_id'] = {
                'message': _("You must accept the terms and conditions in the Settings to use the IT EDI."),
                'action_text': _("View Settings"),
                'action': self.env['res.config.settings']._get_records_action(name=_("Settings"), context=new_context),
            }
        return errors

    @api.onchange("l10n_it_has_tax_representative")
    def _onchange_l10n_it_has_tax_represeentative(self):
        for company in self:
            if not company.l10n_it_has_tax_representative:
                company.l10n_it_tax_representative_partner_id = False

```

## File: models\res_config_settings.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields, _
from odoo.exceptions import UserError

class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    is_edi_proxy_active = fields.Boolean(compute='_compute_is_edi_proxy_active')
    l10n_it_edi_proxy_current_state = fields.Char(compute='_compute_l10n_it_edi_proxy_current_state')
    l10n_it_edi_register = fields.Boolean(compute='_compute_l10n_it_edi_register', inverse='_set_l10n_it_edi_register_demo_mode')
    l10n_it_edi_demo_mode = fields.Selection(
        [('demo', 'Demo'),
         ('test', 'Test (experimental)'),
         ('prod', 'Official')],
        compute='_compute_l10n_it_edi_demo_mode',
        inverse='_set_l10n_it_edi_register_demo_mode',
        readonly=False)

    def _create_proxy_user(self, company_id, edi_mode):
        self.env['account_edi_proxy_client.user']._register_proxy_user(company_id, 'l10n_it_edi', edi_mode)

    def button_create_proxy_user(self):
        self._create_proxy_user(self.company_id, self.l10n_it_edi_demo_mode)

    @api.depends('company_id.account_edi_proxy_client_ids', 'company_id.account_edi_proxy_client_ids.active')
    def _compute_l10n_it_edi_demo_mode(self):
        for config in self:
            edi_user = self.env['account_edi_proxy_client.user'].search([
                ('company_id', '=', config.company_id.id),
                ('proxy_type', '=', 'l10n_it_edi'),
            ], limit=1)
            config.l10n_it_edi_demo_mode = edi_user.edi_mode or 'demo'

    @api.depends('company_id.account_edi_proxy_client_ids', 'company_id.account_edi_proxy_client_ids.active')
    def _compute_is_edi_proxy_active(self):
        for config in self:
            config.is_edi_proxy_active = config.company_id.account_edi_proxy_client_ids

    @api.depends('company_id.account_edi_proxy_client_ids', 'company_id.account_edi_proxy_client_ids.active')
    def _compute_l10n_it_edi_proxy_current_state(self):
        for config in self:
            proxy_user = config.company_id.account_edi_proxy_client_ids.search([
                ('company_id', '=', config.company_id.id),
                ('proxy_type', '=', 'l10n_it_edi'),
            ], limit=1)

            config.l10n_it_edi_proxy_current_state = 'inactive' if not proxy_user else 'demo' if proxy_user.id_client[:4] == 'demo' else 'active'

    @api.depends('company_id')
    def _compute_l10n_it_edi_register(self):
        """Needed because it expects a compute"""
        self.l10n_it_edi_register = False

    def _set_l10n_it_edi_register_demo_mode(self):
        for config in self:

            proxy_user = self.env['account_edi_proxy_client.user'].search([
                ('company_id', '=', config.company_id.id),
                ('proxy_type', '=', 'l10n_it_edi'),
            ], limit=1)

            real_proxy_users = self.env['account_edi_proxy_client.user'].sudo().search([
                ('company_id', '=', config.company_id.id),
                ('proxy_type', '=', 'l10n_it_edi'),
                ('id_client', 'not like', 'demo'),
            ])

            # Update the config as per the selected radio button
            previous_demo_state = proxy_user.edi_mode
            edi_mode = config.l10n_it_edi_demo_mode

            # If the user is trying to change from a state in which they have a registered official or testing proxy client
            # to another state, we should stop them
            if real_proxy_users and previous_demo_state != edi_mode:
                raise UserError(_("The company has already registered with the service as 'Test' or 'Official', it cannot change."))

            if config.l10n_it_edi_register:
                # There should only be one user at a time, if there are no users, register one
                if not proxy_user:
                    self._create_proxy_user(config.company_id, edi_mode)
                    return

                # If there is a demo user, and we are transitioning from demo to test or production, we should
                # delete all demo users and then create the new user.
                elif proxy_user.id_client[:4] == 'demo' and edi_mode != 'demo':
                    self.env['account_edi_proxy_client.user'].search([
                        ('company_id', '=', config.company_id.id),
                        ('proxy_type', '=', 'l10n_it_edi'),
                        ('id_client', '=like', 'demo%'),
                    ]).sudo().unlink()
                    self._create_proxy_user(config.company_id, edi_mode)

```

## File: models\res_partner.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from stdnum.it import codicefiscale, iva

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class ResPartner(models.Model):
    _name = 'res.partner'
    _inherit = 'res.partner'

    l10n_it_pec_email = fields.Char(string="PEC e-mail")
    l10n_it_codice_fiscale = fields.Char(string="Codice Fiscale", size=16)
    l10n_it_pa_index = fields.Char(
        string="Destination Code",
        size=7,
        help="Must contain the 6-character (or 7) code, present in the PA Index "
             "in the information relative to the electronic invoicing service, "
             "associated with the office which, within the addressee administration, deals "
             "with receiving (and processing) the invoice.",
    )

    _sql_constraints = [
        ('l10n_it_codice_fiscale',
            "CHECK(l10n_it_codice_fiscale IS NULL OR l10n_it_codice_fiscale = '' OR LENGTH(l10n_it_codice_fiscale) >= 11)",
            "Codice fiscale must have between 11 and 16 characters."),

        ('l10n_it_pa_index',
            "CHECK(l10n_it_pa_index IS NULL OR l10n_it_pa_index = '' OR LENGTH(l10n_it_pa_index) >= 6)",
            "Destination Code must have between 6 and 7 characters."),
    ]

    def _l10n_it_edi_is_public_administration(self):
        """ Returns True if the destination of the FatturaPA belongs to the Public Administration. """
        self.ensure_one()
        return self.country_id.code == 'IT' and len(self.l10n_it_pa_index or '') == 6

    def _l10n_it_edi_get_values(self):
        """ Generates all partner values needed by l10n_it_edi XML export.

            VAT number:
            If there is a VAT number and the partner is not in EU, then the exported value is 'OO99999999999'
            If there is a VAT number and the partner is in EU, then remove the country prefix
            If there is no VAT and the partner is not in Italy, then the exported value is '0000000'
            If there is no VAT and the partner is in Italy, the VAT is not set and Codice Fiscale will be relevant in the XML.
            If there is no VAT and no Codice Fiscale, the invoice is not even exported, so this case is not handled.

            Country:
            First, try and deduct the country from the VAT number.
            If not, take the country directly from the partner.
            If there's a codice fiscale, the country is 'IT'.

            PA Index:
            If the partner is in Italy, then the l10n_it_pa_index is used, and '0000000' if missing.
            If the partner is not in Italy, the default 'XXXXXXX' is used.

            Codice Fiscale:
            If the Tax Code is equal to the Italian VAT, it may mistakenly have the country prefix,
            so we try and remove it if we can

            Zip(code):
            Non-italian countries are not mapped by the Tax Agency, so it's fixed at '00000'
        """
        if not self or len(self) > 1:
            return {}

        europe = self.env.ref('base.europe', raise_if_not_found=False)
        in_eu = not europe or not self.country_id or self.country_id in europe.country_ids
        is_sm = self.country_id and self.country_id.code == "SM"

        # VAT number and country code
        normalized_vat = self.vat
        normalized_country = self.country_code
        has_vat = self.vat and not self.vat in ['/', 'NA']
        if has_vat:
            normalized_vat = self.vat.replace(' ', '')
            if in_eu:
                # If there is no country-code prefix, it's domestic to Italy
                if normalized_vat[:2].isdecimal():
                    if not normalized_country:
                        normalized_country = 'IT'
                # If the partner is from the EU, the country-code prefix of the VAT must be taken away
                else:
                    if not normalized_country:
                        normalized_country = normalized_vat[:2].upper()
                    normalized_vat = normalized_vat[2:]
            # If customer is from San Marino
            elif is_sm:
                normalized_vat = normalized_vat if normalized_vat[:2].isdecimal() else normalized_vat[2:]
            # The Tax Agency arbitrarily decided that non-EU VAT are not interesting,
            # so this default code is used instead
            # Detect the country code from the partner country instead
            else:
                normalized_vat = 'OO99999999999'

        # If it has a codice fiscale (and no country), it's an Italian partner
        if not normalized_country and self.l10n_it_codice_fiscale:
            normalized_country = 'IT'
        elif not has_vat and self.country_id and self.country_id.code != 'IT':
            normalized_vat = '0000000'

        if normalized_country == 'IT':
            pa_index = (self.l10n_it_pa_index or '0000000').upper()
            zipcode = self.zip
            state_code = self.state_id and self.state_id.code
        else:
            # San Marino is externally integrated with the SdI.
            # The country as a whole has a single fixed Destination Code.
            # https://www.agenziaentrate.gov.it/portale/documents/20143/3788702/Modifiche+ProvvedimentonSanMarino+0248717-2021.pdf/429b5571-17b9-0cce-7f62-f79cf53086d7
            pa_index = '2R4GTO8' if is_sm else 'XXXXXXX'
            zipcode = '00000'
            state_code = False

        return {
            'codice_fiscale': self._l10n_it_edi_normalized_codice_fiscale(),
            'vat': normalized_vat,
            'country_code': normalized_country,
            'state_code': state_code,
            'pa_index': pa_index,
            'zip': zipcode,
            'in_eu': in_eu,
            'is_company': self.is_company,
            'first_name': ' '.join(self.name.split()[:1]),
            'last_name': ' '.join(self.name.split()[1:]),
        }

    def _l10n_it_edi_normalized_codice_fiscale(self, l10n_it_codice_fiscale=None):
        """ Normalize the Italian Tax Code for export.
            If the Tax Code is equal to the Italian VAT, it may mistakenly have the country prefix,
            so we try and remove it if we can
        """
        if l10n_it_codice_fiscale is None:
            self.ensure_one()
            l10n_it_codice_fiscale = self.l10n_it_codice_fiscale
        if l10n_it_codice_fiscale:
            if codicefiscale._code_re.match(l10n_it_codice_fiscale):
                # Personal codice
                return codicefiscale.compact(l10n_it_codice_fiscale)
            # Company codice
            return iva.compact(l10n_it_codice_fiscale)

    @api.onchange('vat', 'country_id')
    def _l10n_it_onchange_vat(self):
        if self.vat and (
            self.country_code == "IT"
            if self.country_code
            else self.vat.startswith("IT")
        ):
            self.l10n_it_codice_fiscale = self._l10n_it_edi_normalized_codice_fiscale(self.vat)
        else:
            self.l10n_it_codice_fiscale = False

    @api.constrains('l10n_it_codice_fiscale')
    def validate_codice_fiscale(self):
        for record in self:
            if record.l10n_it_codice_fiscale and (not codicefiscale.is_valid(record.l10n_it_codice_fiscale) and not iva.is_valid(record.l10n_it_codice_fiscale)):
                raise UserError(_("Invalid Codice Fiscale '%s': should be like 'MRTMTT91D08F205J' for physical person and '12345670546' for businesses.", record.l10n_it_codice_fiscale))

    def _l10n_it_edi_export_check(self, checks=None):
        checks = checks or ['partner_vat_codice_fiscale_missing', 'partner_address_missing']
        fields_to_check = {
            'partner_vat_missing': {
                'fields': [('vat',)],
                'message': _("Partner(s) should have a VAT number."),
            },
            'partner_vat_codice_fiscale_missing': {
                'fields': [('vat', 'l10n_it_codice_fiscale')],
                'message': _("Partner(s) should have a VAT number or Codice Fiscale."),
            },
            'partner_country_missing': {
                'fields': [('country_id',)],
                'message': _("Partner(s) should have a Country when used for simplified invoices."),
            },
            'partner_address_missing': {
                'fields': [('street', 'street2'), ('zip',), ('city',), ('country_id',)],
                'message': _("Partner(s) should have a complete address, verify their Street, City, Zipcode and Country."),
            },
        }
        selected_checks = {k: v for k, v in fields_to_check.items() if k in checks}
        single_views = [(False, 'form')]
        list_view = (self.env.ref('l10n_it_edi.res_partner_tree_l10n_it', raise_if_not_found=False))
        multi_views = [(list_view.id if list_view else False, 'list'), (False, 'form')]
        errors = {}
        for key, check in selected_checks.items():
            for fields_tuple in check['fields']:
                if invalid_records := self.filtered(lambda record: not any(record[field] for field in fields_tuple)):
                    views = single_views if len(invalid_records) == 1 else multi_views
                    errors[key] = {
                        'message': check['message'],
                        'action_text': _("View Partner(s)"),
                        'action': invalid_records._get_records_action(name=_("Check Partner(s)"), views=views),
                    }
        return errors

    def _deduce_country_code(self):
        if self.l10n_it_codice_fiscale:
            return 'IT'
        return super()._deduce_country_code()

    def _peppol_eas_endpoint_depends(self):
        # extends account_edi_ubl_cii
        return super()._peppol_eas_endpoint_depends() + ['l10n_it_codice_fiscale']

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_partner
from . import res_company
from . import res_config_settings
from . import account_move
from . import ddt
from . import ir_attachment
from . import account_edi_proxy_user

```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
"access_it_ddt_manager","it_ddt manager","model_l10n_it_ddt","account.group_account_invoice",1,1,1,1
```

## File: tools\remove_signature.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

"""
    Italian E-invoice signed files content extraction.

    There are two methods: OpenSSL and Fallback.
    Sometimes OpenSSL fail in reading signed invoices for some error in the signature itself.
    The Fallback method only has minimal code to extract the invoices' content without verifying the signature itself.
    It's only to be used as a no-requirements fallback for OpenSSL.
"""

import logging
import struct
import warnings
from contextlib import suppress

_logger = logging.getLogger(__name__)


def remove_signature(content, target=None):
    """ Takes a bytestring supposedly PKCS7 signed and returns its PKCS7-data only """
    for removal_strategy in (remove_signature_openssl, remove_signature_fallback):
        if target:
            target.remove_signature_method = removal_strategy.__name__
        with suppress(Exception):
            return removal_strategy(content)

# --------------------------------------------------------------------------------
# UTILS
# --------------------------------------------------------------------------------


def byte_to_bit_array(val):
    """ Convert a byte to an array of zeros and ones """
    return [((val & (1 << pos)) and 1) or 0 for pos in range(7, -1, -1)]


def bit_array_to_byte(val):
    """ Convert an array of zeros and ones to byte """
    value = 0
    max_idx = len(val) - 1
    for i in range(max_idx, -1, -1):
        value += val[i] << max_idx - i
    return value

# --------------------------------------------------------------------------------
# OPENSSL
# --------------------------------------------------------------------------------


try:
    from OpenSSL import crypto as ssl_crypto
    import OpenSSL._util as ssl_util
except ImportError:
    ssl_crypto = None
    _logger.warning("Cannot import library 'OpenSSL' for PKCS#7 envelope extraction.")


def remove_signature_openssl(content):
    """ Remove the PKCS#7 envelope from given content, making a '.xml.p7m' file content readable as it was '.xml'.
        As OpenSSL may not be installed, in that case a warning is issued and None is returned. """

    # Prevent using the library if it had import errors
    if not ssl_crypto:
        _logger.warning("Error reading the content, check if the OpenSSL library is installed for for PKCS#7 envelope extraction.")
        return None

    # Load some tools from the library
    null = ssl_util.ffi.NULL
    verify = ssl_util.lib.PKCS7_verify

    # By default ignore the validity of the certificates, just validate the structure
    flags = ssl_util.lib.PKCS7_NOVERIFY | ssl_util.lib.PKCS7_NOSIGS

    # Read the signed data fron the content
    out_buffer = ssl_crypto._new_mem_buf()

    # This method is deprecated, but there are actually no alternatives
    with warnings.catch_warnings():
        warnings.filterwarnings("ignore", category=DeprecationWarning)
        loaded_data = ssl_crypto.load_pkcs7_data(ssl_crypto.FILETYPE_ASN1, content)

    # Verify the signature
    if verify(loaded_data._pkcs7, null, null, null, out_buffer, flags) != 1:
        ssl_crypto._raise_current_error()

    # Get the content as a byte-string
    return ssl_crypto._bio_to_string(out_buffer)

# --------------------------------------------------------------------------------
# FALLBACK REMOVE SIGNATURE (ASN1 parse)
# --------------------------------------------------------------------------------


def remove_signature_fallback(content):
    """ The invoice content is inside an ASN1 node identified by PKCS7_DATA_OID (pkcs7-data).
        The node is defined as an OctectString, which can be composed of an arbitrary
        sequence of octects of string data.
        We visit in-order the ASN1 tree nodes until we find the pkcs7-data, then we look for content.
        Once we found it, we read all OctectString that get yielded by the in-order visit..
        When there are no more OctectStrings, then another object will follow
        with its header and identifier, so we stop exploring and just return the content.

        See also:
        https://datatracker.ietf.org/doc/html/rfc2315
        https://www.oss.com/asn1/resources/asn1-made-simple/asn1-quick-reference/octetstring.html
    """
    PKCS7_DATA_OID = '1.2.840.113549.1.7.1'
    result, header_found, data_found = None, False, False
    for node in Reader().build_from_stream(content):
        if node.kind == 'ObjectIdentifier' and node.content == PKCS7_DATA_OID:
            header_found = True
        if header_found and node.kind == 'OctetString':
            data_found = True
            result = (result or b'') + node.content
        elif data_found:
            break

    if not header_found:
        raise Exception("ASN1 Header not found")
    if not data_found:
        raise Exception("ASN1 Content not found")
    return result

# --------------------------------------------------------------------------------
# ASN1 DATA
# --------------------------------------------------------------------------------


universal_tags = {
    0: 'Zero',
    1: 'Boolean',
    2: 'Integer',
    3: 'BitString',
    4: 'OctetString',
    5: 'Null',
    6: 'ObjectIdentifier',
    7: 'ObjectDescriptor',
    8: 'External',
    9: 'Real',
    10: 'Enumerated',
    11: 'EmbeddedPDV',
    12: 'UTF8String',
    13: 'RelativeOid',
    16: 'Sequence',
    17: 'Set',
    18: 'NumericString',
    19: 'PrintableString',
    20: 'TeletexString',
    21: 'VideotexString',
    22: 'IA5String',
    23: 'UTCTime',
    24: 'GeneralizedTime',
    25: 'GraphicString',
    26: 'VisibleString',
    27: 'GeneralString',
    28: 'UniversalString',
    29: 'CharacterString',
    30: 'BMPString',
}

# --------------------------------------------------------------------------------
# NODES (ASN1 parse)
# --------------------------------------------------------------------------------


class Asn1Node:
    """ Base class for Asn1 nodes """
    _content = None

    def __init__(self, kind, start_offset, node_len, cls, parent=None):
        """ Initialization of the Asn1 node """

        if not (parent is None or issubclass(Asn1Node, parent.__class__)):
            raise TypeError("parent must be an Asn1Node or None")

        # Register to parent
        self.parent = parent
        if parent:
            parent.children.append(self)

        self.kind = kind
        self.start_offset = start_offset
        self.children = []
        self.cls = cls
        self.finalized = False
        self.name = self.__class__.__name__.replace('Node', '')
        self.length = node_len

    def finalize(self, end_offset, content=None):
        """ Closes the initialization of the Asn1 node, giving it content and finished length """
        self.content = content
        self.length = end_offset - self.start_offset
        self.end_offset = end_offset
        self.finalized = True

    def total_length(self):
        """ Get the total length of the node if defined. The definition and length bytes must be considered. """
        return self.length + 2 if self.length != "?" else "?"

    @property
    def content(self):
        return self._content

    @content.setter
    def content(self, content):
        if content is not None and not isinstance(content, bytes):
            raise TypeError("content must be bytes or None")
        self._content = content


class PrimitiveNode(Asn1Node):
    """ Primitive Asn1 nodes contain pure data """
    pass


class OctetStringNode(PrimitiveNode):
    """ Octet String Asn1 node """
    pass


class ObjectIdentifierNode(PrimitiveNode):
    """ Asn1 Object Identifier, i.e. 1.3.6.1.5.5.7.48.1 """
    @Asn1Node.content.setter
    def content(self, content):
        # Run through the content's bytes
        calc = 0
        result = ''
        for idx, octet in enumerate(content):
            # The first position is treated differently
            if idx == 0:
                result += "%s.%s" % (octet // 40, octet % 40)
                continue

            # Other positions value the less significant 7 bits,
            # but the most significant bit is only negative when there's a break
            calc = (calc << 7) + (octet % 0x80)
            break_it = not bool(octet // 0x80)
            if break_it:
                result += ".%s" % calc
                calc = 0

        self._content = result

# --------------------------------------------------------------------------------
# READER (ASN1 parse)
# --------------------------------------------------------------------------------


class Reader:

    def __init__(self, *args, **kwargs):
        self.clear()

    def clear(self):
        self.offset = 0
        self.root = None
        self.current_node = None
        self.parent_node = None
        self.open_nodes_stack = []
        self.last_open_node = None

    def finalize_last_open_node(self):
        """ Whenever a node is complete, it is finalized, and the references are updated """
        self.last_open_node = self.open_nodes_stack.pop()
        self.last_open_node.finalize(self.offset, None)
        self.parent_node = self.last_open_node.parent
        self.current_node = None
        finalized_node = self.last_open_node
        self.last_open_node = self.open_nodes_stack[-1] if self.open_nodes_stack else None
        return finalized_node

    def build_from_stream(self, stream):
        """ Build an Asn1 tree starting from a byte string from a p7m file """

        self.clear()
        while self.offset < len(stream):

            start_offset = self.offset
            self.last_open_node = self.open_nodes_stack[-1] if self.open_nodes_stack else None

            # Read the definition and length bytes
            definition_byte, self.offset = self.consume('B', stream, self.offset)
            node_len, _bytes_read, self.offset = self.read_length(stream, self.offset)

            if definition_byte == 0 and node_len == 0 and self.open_nodes_stack:
                yield self.finalize_last_open_node()
                continue

            # Create the current Node
            self.current_node = self.create_node(definition_byte, node_len, start_offset, parent=self.parent_node)
            if not self.root:
                self.root = self.current_node

            # If not primitive, add to the stack
            if not issubclass(self.current_node.__class__, PrimitiveNode):
                self.open_nodes_stack.append(self.current_node)
                self.last_open_node = self.current_node
                self.parent_node = self.current_node
            else:
                data, self.offset = self.consume('%ss' % self.current_node.length, stream, self.offset)
                self.current_node.finalize(self.offset, data)
                yield self.current_node

            # Clear the stack of all finished nodes
            while (
                self.last_open_node
                and not self.last_open_node.finalized
                and self.last_open_node.length != '?'
                and self.last_open_node.start_offset + self.last_open_node.total_length() <= self.offset
            ):
                yield self.finalize_last_open_node()

        return self.root

    def consume(self, _format, stream, offset):
        """ Read from a bytes stream to get data out """
        size = struct.calcsize(_format)
        value = struct.unpack_from(_format, stream, offset)[0]
        offset += size
        return value, offset

    def read_length(self, stream, offset):
        """ Returns: (length of the node, bytes read, updated offset) """

        # Read the first byte: if it is zero, it's a special entry.
        # Probably it's the second byte of a closing tag of a node (\x00 \x00 <--)
        first_byte, offset = self.consume('B', stream, offset)
        if first_byte == 0:
            return 0, 1, offset

        # Convert byte to bits
        bits = byte_to_bit_array(first_byte)

        # If the first bit of the first length byte is on
        if not bits[0]:
            return first_byte, 1, offset

        # If it's the only bit being set, the length is indefinite,
        # and the node will terminate with a double \x00
        if not any(bits[1:]):
            return '?', 1, offset

        # We turn off the first bit, and the rest is the number of bytes we have to read
        bytes_read = bit_array_to_byte([0] + bits[1:])

        # Each byte we read is less significant, so we increase the significance of the
        # value we already read and increment by the current byte
        node_len = 0
        for dummy in range(1, bytes_read + 1):
            current_byte, offset = self.consume('B', stream, offset)
            node_len = (node_len << 8) + current_byte

        return node_len, bytes_read, offset

    def create_node(self, definition_byte, node_len, start_offset, parent=None):
        """ Method to create new Asn1 nodes, given the definition bytes and the offset """

        target_class = Asn1Node
        kind = "Indefinite" if node_len == "?" else "Container"

        node_classes = {
            (0, 0): 'Universal',
            (0, 1): 'Application',
            (1, 0): 'Context-specific',
            (1, 1): 'Private'
        }
        bits = byte_to_bit_array(definition_byte)
        cls_bits = tuple(bits[0:2])
        cls = node_classes[cls_bits]
        if cls == 'Universal':
            is_primitive = not bool(bits[2])
            if is_primitive:
                tag = definition_byte % (1 << 5)
                kind = universal_tags.get(tag)
                if kind:
                    subclasses = PrimitiveNode.__subclasses__()
                    target_classes = {x.__name__: x for x in subclasses}
                    target_class = target_classes.get("%sNode" % kind, PrimitiveNode)

        return target_class(kind, start_offset, node_len, cls, parent)

```

## File: tools\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import remove_signature

```

## File: views\l10n_it_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_partner_tree_l10n_it" model="ir.ui.view">
        <field name="name">res.partner.tree.l10n.it</field>
        <field name="mode">primary</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="after">
                <field name="l10n_it_codice_fiscale"/>
                <field name="l10n_it_pa_index"/>
            </xpath>
        </field>
    </record>

    <record id="res_partner_form_l10n_it" model="ir.ui.view">
        <field name="name">res.partner.form.l10n.it</field>
        <field name="model">res.partner</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="arch" type="xml">
        <data>
            <xpath expr="//field[@name='category_id']" position="after">
                <field name="l10n_it_pec_email" invisible="'IT' not in fiscal_country_codes or parent_id"/>
                <field name="l10n_it_codice_fiscale" invisible="'IT' not in fiscal_country_codes or parent_id"/>
                <field name="l10n_it_pa_index" invisible="'IT' not in fiscal_country_codes or parent_id"/>
            </xpath>
        </data>
        </field>
    </record>

    <record id="res_company_form_l10n_it" model="ir.ui.view">
        <field name="name">res.company.form.l10n.it</field>
        <field name="model">res.company</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="base.view_company_form"/>
        <field name="arch" type="xml">
        <data>
            <xpath expr="//field[@name='vat']" position="after">
                <field name="l10n_it_codice_fiscale" invisible="country_code != 'IT'"/>
                <field name="l10n_it_tax_system" invisible="country_code != 'IT'"/>
            </xpath>
            <xpath expr="//page" position="after">
                <page string="Electronic Invoicing" name="electronic_invoicing" invisible="country_code != 'IT'">
                    <group>
                        <separator string="Economic and Administrative Index" colspan="4"/>
                        <div colspan="4">
                            The seller/provider is a company listed on the register of companies and as
                            such must also indicate the registration data on all documents (art. 2250, Italian
                            Civil Code)
                        </div>
                        <group>
                            <field name="l10n_it_has_eco_index" string="Company listed on the register of companies"/>
                            <field name="l10n_it_eco_index_office" invisible="not l10n_it_has_eco_index"/>
                            <field name="l10n_it_eco_index_number" invisible="not l10n_it_has_eco_index"/>
                            <field name="l10n_it_eco_index_share_capital" invisible="not l10n_it_has_eco_index"/>
                            <field name="l10n_it_eco_index_sole_shareholder" invisible="not l10n_it_has_eco_index"/>
                            <field name="l10n_it_eco_index_liquidation_state" invisible="not l10n_it_has_eco_index"/>
                        </group>
                    </group>
                    <group>
                        <separator string="Tax representative" colspan="4"/>
                        <div colspan="4">
                            The seller/provider is a non-resident subject which carries out transactions in Italy
                            with relevance for VAT purposes and which takes avail of a tax representative in Italy
                        </div>
                        <group>
                            <field name="l10n_it_has_tax_representative" string="Company have a tax representative"/>
                            <field name="l10n_it_tax_representative_partner_id" invisible="not l10n_it_has_tax_representative"/>
                        </group>
                    </group>
                </page>
            </xpath>
        </data>
        </field>
    </record>

    <record id="view_invoice_tree_inherit" model="ir.ui.view">
        <field name="name">account.move.tree.inherit</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_invoice_tree" />
        <field name="arch" type="xml">
            <field name="state" position="before">
                <field name="l10n_it_edi_transaction" optional="hide"/>
                <field name="l10n_it_edi_attachment_id" optional="hide"/>
                <field name="l10n_it_edi_state" optional="hide"/>
            </field>
        </field>
    </record>

    <record id="view_account_invoice_filter" model="ir.ui.view">
        <field name="name">account.invoice.select.inherit</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_account_invoice_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//search/field[@name='journal_id']" position="after">
                <field name="l10n_it_edi_transaction" groups="base.group_no_one"/>
                <field name="l10n_it_edi_attachment_id" groups="base.group_no_one"/>
                <field name="l10n_it_edi_state" groups="base.group_no_one"/>
            </xpath>
        </field>
    </record>

    <record id="account_invoice_form_l10n_it" model="ir.ui.view">
        <field name="name">account.move.form.l10n.it</field>
        <field name="model">account.move</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <data>
                <xpath expr="//header" position="inside">
                    <button name="action_l10n_it_edi_send"
                            type="object"
                            string="Send Tax Integration"
                            invisible="state != 'posted' or not l10n_it_edi_is_self_invoice or is_move_sent or country_code != 'IT'"
                            data-hotkey="y"/>
                </xpath>
                <xpath expr="//sheet" position="before">
                    <field name="l10n_it_edi_is_self_invoice" invisible="1"/>
                    <field name="l10n_it_edi_attachment_id" invisible="1"/>
                    <div class="alert alert-warning" role="alert"
                        invisible="not l10n_it_edi_header 
                                   or state == 'draft'
                                   or l10n_it_edi_state in ('forwarded', 'accepted_by_pa_partner', 'accepted_by_pa_partner_after_expiry', 'forward_failed')">
                        <div class="p-0 m-0"><i class='fa fa-warning' role="img" title="EDI (Italy)"/><span class="mx-1">E-invoicing (Italy)</span></div>
                        <field name="l10n_it_edi_header"/>
                    </div>
                </xpath>
                <xpath expr="//button[@name='action_invoice_sent']" position="before">
                    <button
                        name="action_check_l10n_it_edi"
                        type="object"
                        string="Check Sending"
                        class="oe_highlight"
                        data-hotkey="shift+K"
                        invisible="l10n_it_edi_state not in ('being_sent', 'processing', 'forward_attempt')"
                    />
                </xpath>
                <xpath expr="//page[@name='other_info']" position="after">
                    <page string="Electronic Invoicing"
                        name="electronic_invoicing"
                        invisible="move_type not in ('out_invoice', 'out_refund', 'in_invoice', 'in_refund') or country_code != 'IT'">
                        <group>
                            <group>
                                <field name="l10n_it_edi_transaction" groups="base.group_no_one" readonly="1"/>
                                <field name="l10n_it_edi_attachment_id" groups="base.group_no_one" readonly="1"/>
                                <field name="l10n_it_stamp_duty" readonly="state != 'draft'"/>
                                <field name="l10n_it_ddt_id" readonly="state != 'draft'" invisible="move_type not in ('out_invoice', 'out_refund')"/>
                            </group>
                            <field name="l10n_it_partner_pa" invisible="1"/>
                            <group invisible="not l10n_it_partner_pa">
                                <field name="l10n_it_origin_document_type" readonly="state != 'draft'"/>
                                <field name="l10n_it_origin_document_name" readonly="state != 'draft'"/>
                                <field name="l10n_it_origin_document_date" readonly="state != 'draft'"/>
                                <field name="l10n_it_cig" readonly="state != 'draft'"/>
                                <field name="l10n_it_cup" readonly="state != 'draft'"/>
                            </group>
                        </group>
                    </page>
                </xpath>
                <xpath expr="//div[@name='journal_div']" position="after">
                    <label for="l10n_it_edi_state" invisible="not l10n_it_edi_state"/>
                    <div name="l10n_it_edi_div" class="d-flex" invisible="not l10n_it_edi_state">
                        <field name="l10n_it_edi_state" class="oe_inline"/>
                    </div>
                </xpath>
            </data>
        </field>
    </record>

    <record id="l10n_it_ddt" model="ir.ui.view">
        <field name="name">ddt.form.l10n.it</field>
        <field name="model">l10n_it.ddt</field>
        <field name="arch" type="xml">
            <form>
                <group>
                    <field name="name"/>
                    <field name="date"/>
                </group>
            </form>
        </field>
    </record>

    <record id="l10n_it_ddt_list_view" model="ir.ui.view">
      <field name="name">l10n_it.ddt.list.view</field>
      <field name="model">l10n_it.ddt</field>
      <field name="arch" type="xml">
        <tree>
          <field name="name"/>
          <field name="date"/>
        </tree>
      </field>
    </record>

    <record id="action_ddt_account" model="ir.actions.act_window">
        <field name="name">Transport Document</field>
        <field name="res_model">l10n_it.ddt</field>
        <field name="view_mode">tree,form</field>
        <field name="view_id" ref="l10n_it_ddt_list_view"/>
    </record>

    <menuitem
            name="DDT"
            parent="account.account_account_menu"
            action="action_ddt_account"
            id="menu_action_ddt_account"
            sequence="15"
            groups="base.group_no_one"/>
</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document">
        <div name="comment" position="before">
            <t t-if="o.l10n_it_stamp_duty"><b>Stamp Duty: </b><span t-field="o.l10n_it_stamp_duty"></span><br/></t>
            <t t-if="o.l10n_it_ddt_id"><b>Transport Document: </b><span t-field="o.l10n_it_ddt_id"></span><br/></t>
            <div t-if="o.l10n_it_origin_document_type" name="pa_fields">
                <b><span t-field="o.l10n_it_origin_document_type"/></b>: <span t-field="o.l10n_it_origin_document_name"/><br/>
                <t t-if="o.l10n_it_origin_document_date"><b>Document Date: </b><span t-field="o.l10n_it_origin_document_date"/><br/></t>
                <t t-if="o.l10n_it_cig"><b>CIG: </b><span t-field="o.l10n_it_cig"/><br/></t>
                <t t-if="o.l10n_it_cup"><b>CUP: </b><span t-field="o.l10n_it_cup"/><br/></t>
            </div>
        </div>

        <p name="payment_communication" position="inside">
            <t t-if="o.country_code == 'IT'">
                <t t-set="term_lines" t-value="o.line_ids.filtered(lambda line: line.display_type == 'payment_term')"/>
                <br/><b>Payment Conditions:</b> <t t-if="len(term_lines) == 1">TP02 pagamento completo</t><t t-else="">TP01 pagamento a rate</t><br/>
            </t>
        </p>
    </template>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.proxy.user</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//block[@id='account_vendor_bills']" position="after">
                <block title="Italian Electronic Invoicing" invisible="country_code != 'IT'" id='account_edi'>
                    <setting>
                        <div class="group-content">
                            <field name="l10n_it_edi_proxy_current_state" invisible="1"/>
                            <span class="o_form_label">
                                Fattura Elettronica mode
                            </span>
                            <span class="fa fa-lg fa-building-o" title="Values set here are company-specific."/>
                            <div class="text-muted">
                                In demo mode Odoo will just simulate the sending of invoices to the government.<br/>
                                In test mode (experimental) Odoo will send the invoices to a non-production service.
                                Saving this change will direct all companies on this database to this use this configuration.
                                Once registered for testing or official, the mode cannot be changed.
                            </div>
                            <field name="l10n_it_edi_demo_mode"
                                    widget="radio"
                                    options="{'horizontal': true}"/>
                        </div>
                        <div class="mt8 content-group" invisible="l10n_it_edi_proxy_current_state == 'active' or l10n_it_edi_proxy_current_state == 'demo' and l10n_it_edi_demo_mode == 'demo'">
                            <span class="o_form_label">Allow Odoo to process invoices</span>
                            <div class="text-muted">
                                By checking this box, I accept that Odoo may process my invoices.
                            </div>
                            <div class="content-group">
                                <field name="l10n_it_edi_register"/>
                            </div>

                        </div>
                        <div class="text-success mt8" invisible="l10n_it_edi_proxy_current_state in ['inactive', 'demo']">
                            An Official or Test service has been registered.
                        </div>
                        <div class="text-success mt8" invisible="l10n_it_edi_proxy_current_state != 'demo' or l10n_it_edi_demo_mode != 'demo'">
                            A Demo service is in use.
                        </div>
                    </setting>
                </block>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: wizard\account_move_send.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from markupsafe import Markup, escape

from odoo import _, api, fields, models
from odoo.addons.base.models.ir_qweb_fields import nl2br


class AccountMoveSend(models.TransientModel):
    _inherit = 'account.move.send'

    l10n_it_edi_warning_message = fields.Html(compute='_compute_l10n_it_edi_warning_message')
    l10n_it_edi_actionable_errors = fields.Json(compute='_compute_l10n_it_edi_xml_export')
    l10n_it_edi_enable_xml_export = fields.Boolean(compute='_compute_l10n_it_edi_xml_export')
    l10n_it_edi_readonly_xml_export = fields.Boolean(compute='_compute_l10n_it_edi_xml_export')
    l10n_it_edi_checkbox_xml_export = fields.Boolean('E-invoice XML',
        compute='_compute_l10n_it_edi_checkbox_xml_export',
        store=True,
        readonly=False)

    l10n_it_edi_enable_send = fields.Boolean(compute='_compute_l10n_it_edi_enable_readonly_send')
    l10n_it_edi_readonly_send = fields.Boolean(compute='_compute_l10n_it_edi_enable_readonly_send')
    l10n_it_edi_checkbox_send = fields.Boolean('Send To Tax Agency',
        compute='_compute_l10n_it_edi_checkbox_send',
        store=True,
        readonly=False,
        help="Send the e-invoice XML to the Italian Tax Agency.")

    def _get_wizard_values(self):
        # EXTENDS 'account'
        values = super()._get_wizard_values()
        values['l10n_it_edi_checkbox_xml_export'] = self.l10n_it_edi_checkbox_xml_export
        values['l10n_it_edi_checkbox_send'] = self.l10n_it_edi_checkbox_send
        return values

    @api.model
    def _get_wizard_vals_restrict_to(self, only_options):
        # EXTENDS 'account'
        values = super()._get_wizard_vals_restrict_to(only_options)
        return {
            'l10n_it_edi_checkbox_xml_export': False,
            'l10n_it_edi_checkbox_send': False,
            **values,
        }

    # -------------------------------------------------------------------------
    # COMPUTE/CONSTRAINS METHODS
    # -------------------------------------------------------------------------

    @api.depends('l10n_it_edi_actionable_errors')
    def _compute_l10n_it_edi_warning_message(self):
        # To be removed -- Proxy feature to be replaced with actionable_errors as soon as the user updates the module
        for wizard in self:
            messages = []
            wizard.l10n_it_edi_warning_message = False
            if wizard.l10n_it_edi_actionable_errors:
                messages.append(_("Please upgrade the Italian EDI module to update this widget."))
                messages.append(_("Go to Applications page and update the 'Italia - Fatturazione Elettronica' module."))
                messages.append("")
                for error_key, error_data in wizard.l10n_it_edi_actionable_errors.items():
                    message = error_data['message']
                    split = error_key.split("_")
                    if len(split) > 1 and (model_id := {
                        'partner': 'res.partner',
                        'move': 'account.move',
                        'company': 'res.company'
                    }.get(split[0], None)):
                        if action := error_data.get('action'):
                            if 'res_id' in action:
                                record_ids = [action['res_id']]
                            else:
                                record_ids = action['domain'][0][2]
                            records = self.env[model_id].browse(record_ids)
                            message = f"{message} - {', '.join(records.mapped('display_name'))}"
                    messages.append(nl2br(escape(message)))
                wizard.l10n_it_edi_warning_message = Markup("<br/>").join(messages)

    @api.depends('move_ids')
    def _compute_l10n_it_edi_xml_export(self):
        for wizard in self:
            if wizard.company_id.account_fiscal_country_id.code == 'IT':
                has_pdf_but_no_xml = any(move.invoice_pdf_report_id and not move.l10n_it_edi_attachment_id for move in wizard.move_ids)
                all_have_xml = all(move.l10n_it_edi_attachment_id for move in wizard.move_ids)
                wizard.l10n_it_edi_actionable_errors = self.move_ids._l10n_it_edi_export_data_check()
                wizard.l10n_it_edi_enable_xml_export = any(m._l10n_it_edi_ready_for_xml_export() for m in wizard.move_ids)
                wizard.l10n_it_edi_readonly_xml_export = wizard.l10n_it_edi_actionable_errors or has_pdf_but_no_xml or all_have_xml
            else:
                wizard.l10n_it_edi_actionable_errors = False
                wizard.l10n_it_edi_enable_xml_export = False
                wizard.l10n_it_edi_readonly_xml_export = False

    @api.depends('move_ids', 'l10n_it_edi_checkbox_xml_export', 'l10n_it_edi_actionable_errors')
    def _compute_l10n_it_edi_enable_readonly_send(self):
        for wizard in self:
            if wizard.company_id.account_fiscal_country_id.code == 'IT':
                xml_already_sent = all(m.l10n_it_edi_state not in (False, 'rejected') for m in wizard.move_ids)
                wizard.l10n_it_edi_enable_send = wizard.l10n_it_edi_checkbox_xml_export
                wizard.l10n_it_edi_readonly_send = bool(wizard.l10n_it_edi_actionable_errors or xml_already_sent)
            else:
                wizard.l10n_it_edi_enable_send = False
                wizard.l10n_it_edi_readonly_send = False

    @api.depends('move_ids')
    def _compute_l10n_it_edi_checkbox_xml_export(self):
        for wizard in self:
            if wizard.company_id.account_fiscal_country_id.code == 'IT':
                all_have_xml = all(move.l10n_it_edi_attachment_id for move in wizard.move_ids)
                wizard.l10n_it_edi_checkbox_xml_export = all_have_xml or (wizard.l10n_it_edi_enable_xml_export and not wizard.l10n_it_edi_readonly_xml_export)
            else:
                wizard.l10n_it_edi_checkbox_xml_export = False

    @api.depends('move_ids', 'l10n_it_edi_checkbox_xml_export')
    def _compute_l10n_it_edi_checkbox_send(self):
        for wizard in self:
            if wizard.company_id.account_fiscal_country_id.code == 'IT':
                wizard.l10n_it_edi_checkbox_send = not wizard.l10n_it_edi_readonly_send and wizard.l10n_it_edi_checkbox_xml_export
            else:
                wizard.l10n_it_edi_checkbox_send = False

    # -------------------------------------------------------------------------
    # BUSINESS ACTIONS
    # -------------------------------------------------------------------------

    @api.model
    def _need_invoice_document(self, invoice):
        # EXTENDS 'account'
        return super()._need_invoice_document(invoice) and not invoice.l10n_it_edi_attachment_id

    @api.model
    def _get_invoice_extra_attachments(self, invoice):
        # EXTENDS 'account'
        return super()._get_invoice_extra_attachments(invoice) + invoice.l10n_it_edi_attachment_id

    @api.model
    def _hook_invoice_document_before_pdf_report_render(self, invoice, invoice_data):
        # EXTENDS 'account'
        super()._hook_invoice_document_before_pdf_report_render(invoice, invoice_data)
        if invoice_data.get('l10n_it_edi_checkbox_xml_export') and invoice._l10n_it_edi_ready_for_xml_export():
            if errors := invoice._l10n_it_edi_export_data_check():
                invoice_data['error'] = {
                    'error_title': _("Errors occurred while creating the e-invoice file:"),
                    'errors': errors,
                }

    @api.model
    def _hook_invoice_document_after_pdf_report_render(self, invoice, invoice_data):
        # EXTENDS 'account'
        super()._hook_invoice_document_after_pdf_report_render(invoice, invoice_data)
        if invoice_data.get('l10n_it_edi_checkbox_xml_export') and invoice._l10n_it_edi_ready_for_xml_export():
            invoice_data['l10n_it_edi_values'] = invoice._l10n_it_edi_get_attachment_values(
                pdf_values=invoice_data['pdf_attachment_values'])

    @api.model
    def _call_web_service_after_invoice_pdf_render(self, invoices_data):
        # EXTENDS 'account'
        super()._call_web_service_after_invoice_pdf_render(invoices_data)
        attachments_vals = {}
        moves = self.env['account.move']
        for move, move_data in invoices_data.items():
            if move_data.get('l10n_it_edi_checkbox_send') and move._l10n_it_edi_ready_for_xml_export():
                moves |= move
                if attachment := move.l10n_it_edi_attachment_id:
                    attachments_vals[move] = {'name': attachment.name, 'raw': attachment.raw}
                else:
                    attachments_vals[move] = invoices_data[move]['l10n_it_edi_values']
        moves._l10n_it_edi_send(attachments_vals)

    @api.model
    def _link_invoice_documents(self, invoice, invoice_data):
        # EXTENDS 'account'
        super()._link_invoice_documents(invoice, invoice_data)
        if attachment_vals := invoice_data.get('l10n_it_edi_values'):
            self.env['ir.attachment'].sudo().create(attachment_vals)
            invoice.invalidate_recordset(fnames=['l10n_it_edi_attachment_id', 'l10n_it_edi_attachment_file'])

```

## File: wizard\account_move_send_views.xml

```xml
<odoo>
    <data>
        <record model="ir.ui.view" id="account_move_send_inherit_l10n_it_edi">
            <field name="name">account.move.send.form.inherit.l10n_it_edi</field>
            <field name="model">account.move.send</field>
            <field name="inherit_id" ref="account.account_move_send_form"/>
            <field name="arch" type="xml">
                <xpath expr="//div[@name='warnings']" position="inside">
                    <field name="l10n_it_edi_readonly_xml_export" invisible="1"/>
                    <field name="l10n_it_edi_enable_xml_export" invisible="1"/>
                    <field name="l10n_it_edi_readonly_send" invisible="1"/>
                    <field name="l10n_it_edi_enable_send" invisible="1"/>
                    <field name="l10n_it_edi_actionable_errors" class="o_field_html" widget="actionable_errors"/>
                </xpath>
                <xpath expr="//div[@name='option_send_mail']" position='after'>
                    <div name="option_l10n_it_edi">
                        <div name="option_l10n_it_edi_xml_export" invisible="not l10n_it_edi_enable_xml_export">
                            <field name="l10n_it_edi_checkbox_xml_export" readonly="l10n_it_edi_readonly_xml_export"/>
                            <b><label for="l10n_it_edi_checkbox_xml_export"/></b>
                            <i class="fa fa-question-circle ml4"
                                role="img"
                                aria-label="Warning"
                                invisible="not l10n_it_edi_readonly_xml_export"
                                title="Create the e-invoice XML ready to be sent to the Italian Tax Agency. It is set as readonly if a report has already been created, to avoid inconsistencies. To re-enable it, delete the PDF attachment."/>
                        </div>
                        <div name="option_l10n_it_edi_send" invisible="not l10n_it_edi_enable_send">
                            <field name="l10n_it_edi_checkbox_send" readonly="l10n_it_edi_readonly_send"/>
                            <b><label for="l10n_it_edi_checkbox_send"/></b>
                        </div>
                    </div>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move_send

```

